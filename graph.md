# PUSCH RX CUDA Graph 结构分析

## 目录
1. [总体架构](#1-总体架构)
2. [图的种类与生命周期](#2-图的种类与生命周期)
3. [节点创建](#3-节点创建)
4. [子图创建（Graph 分层与 Device Graph Launch）](#4-子图创建)
5. [节点更新](#5-节点更新)
6. [信道估计图节点管理](#6-信道估计图节点管理)
7. [条件图节点（Conditional Graph Node）](#7-条件图节点)
8. [完整调用流程图](#8-完整调用流程图)
9. [各图节点完整加载顺序 DAG](#9-各图节点完整加载顺序-dag)

---

## 1. 总体架构

PUSCH RX 采用 **CUDA Graph** 来管理 GPU 端流水线，将各处理阶段（信道估计、均衡、软解映射、LDPC 解码、CRC 等）组织为有向无环图（DAG），从而消除每个 Kernel 的 CPU 启动开销，并显式表达阶段间的依赖关系。



### 1.1 处理路径

```
PUSCH RX 整体处理路径：
                                              ┌──── Early-HARQ 路径 ──────────────┐
输入帧                                        │  m_ehqGraph                        │
   │                                         │  (信道估计pre + 均衡 + 极化解码)     │
   ▼                                         └────────────────────────────────────┘
[Launch Graph]                                              │
   │  m_preSubSlotGraph (wait+event+DGL)                    │
   ▼                                                        ▼
[Full-Slot Graph]        ┌──── Front-Loaded DMRS 路径 ────────────────────────────┐
   │  m_fullSlotGraph    │  m_frontLoadedDmrsGraph                                 │
   │  m_frontLoaded...   │  m_frontLoadedDmrsFullSlotGraph                        │
   │                     └────────────────────────────────────────────────────────┘
   ▼
[输出到 CPU]
```

### 1.2 图对象一览（`pusch_rx.hpp`）

| 成员变量 | 类型 | 说明 |
|---------|------|------|
| `m_fullSlotGraph` | `CUgraph` | 标准全时隙处理图 |
| `m_fullSlotGraphExec` | `CUgraphExec` | 全时隙图执行实例 |
| `m_frontLoadedDmrsFullSlotGraph` | `CUgraph` | 前置 DMRS 全时隙图 |
| `m_ehqGraph` | `CUgraph` | Early-HARQ 子图 |
| `m_ehqGraphExec` | `CUgraphExec` | Early-HARQ 执行实例 |
| `m_frontLoadedDmrsGraph` | `CUgraph` | 前置 DMRS 子图 |
| `m_preSubSlotGraph` | `CUgraph` | Sub-slot 启动同步图 |
| `m_preFullSlotGraph` | `CUgraph` | Full-slot 启动同步图 |
| `condGraphInfo::m_graph[3]` | `CUgraph[3]` | 条件图/设备图（G0/G1/G2） |

---

## 2. 图的种类与生命周期

### 2.1 创建顺序（构造函数内）

```cpp
// pusch_rx.cpp ~line 1184
createComponents(cuStream, ...);

// Full-slot 图（两种模式各一个）
createFullSlotGraph(PUSCH_LEGACY_FULL_SLOT_PROC, m_fullSlotGraph, ...);
createFullSlotGraph(PUSCH_FRONT_LOADED_DMRS_FULL_SLOT_PROC, m_frontLoadedDmrsFullSlotGraph, ...);

// 实例化（Instantiate）→ 生成可执行图
cuGraphInstantiate(&m_fullSlotGraphExec, m_fullSlotGraph, 0);
cuGraphInstantiate(&m_frontLoadedDmrsFullSlotGraphExec, ...);

// Early-HARQ 与前置 DMRS 子图
createEarlyHarqGraph();
createFrontLoadedDmrsGraph();
cuGraphInstantiate(&m_ehqGraphExec, m_ehqGraph, 0);
cuGraphInstantiate(&m_frontLoadedDmrsGraphExec, ...);

// Launch 同步图（等待时序信号用）
createLaunchGraph(m_preSubSlotGraph, ...);
createLaunchGraph(m_preFullSlotGraph, ...);
cuGraphInstantiate(&m_preSubSlotGraphExec, m_preSubSlotGraph, 0);
cuGraphInstantiate(&m_preFullSlotGraphExec, m_preFullSlotGraph, 0);

// 初始禁用所有节点（首次 setup 之前所有节点均 disabled）
updateEarlyHarqGraph(true);
updateFrontLoadedDmrsGraph(true);
updateFullSlotGraph(true, PUSCH_LEGACY_FULL_SLOT_PROC, ...);
```

### 2.2 运行阶段（每时隙）

```
setup() → updateXxxGraph()  更新节点参数 + 按条件 enable/disable 节点
run()   → subSlotKernelLaunch() / fullSlotKernelLaunch()
           cuGraphLaunch(m_ehqGraphExec, stream)
           cuGraphLaunch(m_fullSlotGraphExec, stream)
```

---

## 3. 节点创建

### 3.1 Launch Graph 节点（`createLaunchGraph`）

`createLaunchGraph` 创建一个仅有 2~3 个节点的轻量图，用于实现 **时序等待 → 事件通知 → （可选）设备图启动**：

```
[waitKernelNode] ──► [eventRecordNode] ──► [deviceGraphLaunchNode]（可选）
```

```cpp
// 创建空图
cuGraphCreate(&graph, 0);

// 1. Wait 节点：自旋等待指定数量的 OFDM 符号到达
cuGraphAddKernelNode(&graphWaitNode, graph,
                     nullptr, 0,          // 无依赖（根节点）
                     &waitCfg.kernelNodeParamsDriver);

// 2. Event 节点：wait 完成后记录事件，通知后续图可以启动
cuGraphAddEventRecordNode(&graphEventNode, graph,
                          &graphWaitNode, 1,
                          waitEndEvent);

// 3. DGL 节点（Device Graph Launch）：由 Event 节点触发
cuGraphAddKernelNode(&graphDglNode, graph,
                     &graphEventNode, 1,
                     &dglCfg.kernelNodeParamsDriver);
```

### 3.2 Full-Slot Graph 节点（`createFullSlotGraph`）

Full-Slot Graph 由以下 **五个阶段** 的 Helper 函数拼接而成，每个阶段返回 `StageResult`（包含该阶段末尾节点，作为下一阶段的父节点）：

#### 阶段依赖关系

```
[C0 根节点]
     │
     ▼
[Front-End 阶段]  buildFrontEndStage()
  ├─ ChEst 节点 (m_chest->chestGraph().addKernelNodeToGraph)
  ├─ NoiseIntfEst 节点
  ├─ CfoTaEst 节点
  ├─ RSRP 节点
  └─ ChEqCoefComp 节点
     │
     ▼
[SoftDemap 阶段]  buildSoftDemapStage()
  ├─ m_chEqSoftDemapNodes (per hetCfg)
  ├─ m_chEqIdftNodes（DFT-S-OFDM）
  └─ m_uciOnPuschSegLLRs0Nodes
     │
     ▼
[C1 可选条件节点]
     │
     ▼
[UCI-P1 Backend]  buildUciP1BackendStage()
  ├─ m_simplexDecoderNode
  ├─ m_rmDecoderNode
  └─ m_polarDecoderNode (+ compCwTree + polSegDeRm)
     │
     ▼
[C2 可选条件节点]
     │
     ▼
[CSI-P2 Backend]  buildCsiP2BackendStage()
  ├─ m_uciOnPuschCsi2CtrlNode
  ├─ m_uciOnPuschCsi2SegLLRs2Node
  ├─ m_uciOnPuschCsi2rmDecoderNode
  ├─ m_uciOnPuschCsi2simplexDecoderNode
  ├─ m_uciOnPuschCsi2CompCwTreeTypesNode
  ├─ m_uciOnPuschCsi2PolSegDeRmDeItlNode
  └─ m_uciOnPuschCsi2PolarDecoderNode
     │
     ▼
[SCH Backend]     buildSchBackendStage()
  ├─ m_resetRateMatchNode / m_rateMatchNode / m_clampRateMatchNode
  ├─ m_ldpcDecoderNodes (per hetCfg)
  └─ m_crcNodes[0/1]
```

#### 关键 API 调用模式

```cpp
// 批量添加多个异构配置的 Kernel 节点
for (int hetCfgIdx = 0; hetCfgIdx < CUPHY_PUSCH_RX_CH_EQ_N_MAX_HET_CFGS; ++hetCfgIdx)
{
    CU_CHECK_EXCEPTION(cuGraphAddKernelNode(
        &m_chEqSoftDemapNodes[fullSlotProcMode][hetCfgIdx],
        *pGraph,
        currDeps.data(), currDeps.size(),          // 父节点列表
        &m_chEqSoftDemapLaunchCfgs[...].kernelNodeParamsDriver));

    nextDeps.push_back(m_chEqSoftDemapNodes[fullSlotProcMode][hetCfgIdx]);
}
currDeps = nextDeps;  // 滚动依赖链
```

### 3.3 Early-HARQ Graph 节点（`createEarlyHarqGraph`）

```cpp
cuGraphCreate(&m_ehqGraph, 0);

// 根节点：空 Kernel，用于统一 fanout
cuGraphAddKernelNode(&m_ehqRootNode, m_ehqGraph,
                     nullptr, 0, &m_emptyNode0paramDriver);

// 信道估计（Early HARQ 路径）
m_chest->earlyHarqGraph().addKernelNodeToGraph(
    m_ehqGraph, currNodeDeps[0], nextNodeDeps[0], nodeParams);

// 噪声/干扰估计节点（并行添加，共同依赖 ChEst）
for(int i = 0; i < CUPHY_PUSCH_RX_NOISE_INTF_EST_N_MAX_HET_CFGS; ++i)
    cuGraphAddKernelNode(&m_ehqNoiseIntfEstNodes[i], ...);

// 均衡系数计算节点
for(int i = 0; i < CUPHY_PUSCH_RX_CH_EQ_N_MAX_HET_CFGS; ++i)
    cuGraphAddKernelNode(&m_ehqChEqCoefCompNodes[i], ...);

// Polar 解码族：compCwTree → polSegDeRmDeItl → polarDecoder
cuGraphAddKernelNode(&m_ehqCompCwTreeTypesNode, ...);
cuGraphAddKernelNode(&m_ehqPolSegDeRmDeItlNode, ...);
cuGraphAddKernelNode(&m_ehqPolarDecoderNode, ...);

// RSRP / RSSI
for(int i = 0; i < ...; ++i)
    cuGraphAddKernelNode(&m_ehqRsrpNodes[i], ...);
    cuGraphAddKernelNode(&m_ehqRssiNodes[i], ...);
```

### 3.4 `addGraphNodeHelper`（CUDA 版本兼容性封装）

```cpp
CUresult PuschRx::addGraphNodeHelper(CUgraphNode* phGraphNode, CUgraph hGraph,
    const CUgraphNode* dependencies, size_t numDependencies, CUgraphNodeParams* nodeParams)
{
#if CUDA_VERSION >= 13000
    // CUDA 13 要求 edge data 参数，传 nullptr
    return cuGraphAddNode(phGraphNode, hGraph, dependencies,
                          nullptr /*edge data*/, numDependencies, nodeParams);
#else
    return cuGraphAddNode(phGraphNode, hGraph, dependencies,
                          numDependencies, nodeParams);
#endif
}
```

---

## 4. 子图创建

### 4.1 Device Graph Launch（DGL）模式

当 `m_deviceGraphLaunchEnabled == true` 时，Full-Slot Graph 被拆分为 **三级设备图（G0 / G1 / G2）**，由 GPU 端 Kernel 负责触发下一级：

```
Host GPU Stream
    │
    ▼
[preFullSlotGraph]  (Host 图，含 DGL 节点)
    │  cuGraphLaunch → G0 device graph
    ▼
G0 (condInfo.m_graph[0])
  Front-End 阶段 → 尾部 DGL Kernel 触发 G1
    │
    ▼
G1 (condInfo.m_graph[1])
  SoftDemap + UCI-P1 → 尾部 DGL Kernel 触发 G2
    │
    ▼
G2 (condInfo.m_graph[2])
  CSI-P2 + SCH Backend (LDPC + CRC)
```

#### 关键代码（`createFullSlotGraph` DGL 部分）

```cpp
if (m_workCancelMode == PUSCH_DEVICE_GRAPHS)
{
    // 实例化 G2 为 Device-launchable
    cuGraphInstantiate(&condInfo.m_graphExec[2],
                       *condInfo.m_pGraph[2],
                       CUDA_GRAPH_INSTANTIATE_FLAG_DEVICE_LAUNCH);

    // 在 G1 中添加"启动 G2"的 Kernel 节点
    void* pKArgs_c2[2] = {&m_workCancelPtr, &condInfo.m_graphExec[2]};
    cuphySetWorkCancelKernelNodeParams(&condInfo.m_init_C2_node_params, pKArgs_c2, 1);
    cuGraphAddKernelNode(&condInfo.m_graph_G2_init_cond_node,
                         condInfo.m_graph[1],
                         dglParentsC2.data(), dglParentsC2.size(),
                         &condInfo.m_init_C2_node_params);

    // 同理实例化 G1，在 G0 中添加"启动 G1"的 Kernel 节点
    cuGraphInstantiate(&condInfo.m_graphExec[1], ...,
                       CUDA_GRAPH_INSTANTIATE_FLAG_DEVICE_LAUNCH);
    cuGraphAddKernelNode(&condInfo.m_graph_G1_init_cond_node, condInfo.m_graph[0], ...);

    // G0 在 Host Full-Slot Graph 中作为设备图启动
    cuGraphInstantiate(&condInfo.m_graphExec[0], ...,
                       CUDA_GRAPH_INSTANTIATE_FLAG_DEVICE_LAUNCH);
}
```

### 4.2 信道估计子图（ChestGraph / EarlyHarqGraph / FrontDmrsGraph）

ChEst 组件内部通过 `ch_est::ChannelEstimateGraphMgr` 管理三种图接口：

| 接口 | 实现类 | 对应 Graph |
|------|--------|-----------|
| `IChestGraphNodes` | `ChestNodes` | Full-slot ChEst 节点 |
| `IChestSubSlotNodes`（earlyHarq） | `ChestSubSlotNodes` | EarlyHarq ChEst 节点 |
| `IChestSubSlotNodes`（frontDmrs） | `ChestSubSlotNodes` | FrontDmrs ChEst 节点 |

`addKernelNodeToGraph` 实现：

```cpp
void ChestSubSlotNodes::addKernelNodeToGraph(CUgraph graph, ...)
{
    // 添加主 Kernel（Pre-Kernel：LS估计 + OCC解扩）
    addKernelImpl(m_nodes, m_nMaxChEstHetCfgs, graph, currNodeDeps, nextNodeDeps, nodeParams);

    // 若算法为多阶段 MMSE（带延迟估计），追加 Filter-Kernel
    if(m_chEstAlgo == PUSCH_CH_EST_ALGO_TYPE_MULTISTAGE_MMSE_WITH_DELAY_EST)
    {
        currNodeDeps = nextNodeDeps;
        nextNodeDeps.clear();
        addSecondaryKernelNodeToGraph(graph, currNodeDeps, nextNodeDeps, nodeParams);
    }
}
```

`addKernelImpl` 核心循环（迭代所有 HetCfg）：

```cpp
void addKernelImpl(CUgraphNode nodes[], std::size_t nMaxChEstHetCfgs,
                   CUgraph graph, const std::vector<CUgraphNode>& currNodeDeps,
                   std::vector<CUgraphNode>& nextNodeDeps, ...)
{
    for(int i = 0; i < nMaxChEstHetCfgs; ++i)
    {
        cuGraphAddKernelNode(&nodes[i], graph,
                             currNodeDeps.data(), currNodeDeps.size(),
                             &nodeParams);
        nextNodeDeps.push_back(nodes[i]);
    }
}
```

---

## 5. 节点更新

### 5.1 更新策略

每个时隙调用 `setup()` → `updateXxxGraph()`，执行两类操作：
1. **更新参数**：`cuGraphExecKernelNodeSetParams` — 更新 Kernel 的动态描述符指针
2. **启用/禁用节点**：`cuGraphNodeSetEnabled` — 根据本时隙是否有该业务，动态开关节点

### 5.2 `updateFullSlotGraph`

```cpp
void PuschRx::updateFullSlotGraph(bool disableAllNodes,
    cuphyPuschFullSlotProcMode_t fullSlotProcMode,
    CUgraphExec graphExec, condGraphInfo& condInfo)
{
    // 对于 DGL 模式，graph_exec 有 3 个（G0/G1/G2）；否则只用 graphExec
    CUgraphExec graph_exec[3] = {graphExec, ...};

    // --- 更新 ChEst 节点参数 ---
    m_chest->chestGraph().setNodeStatus(..., graph_exec[0]);

    // --- UCI 相关节点（Simplex / RM / Polar）---
    if(m_nRmCws > 0 && !disableAllNodes)
    {
        // 更新参数
        cuGraphExecKernelNodeSetParams(graph_exec[1], m_rmDecoderNode[mode], &rmCfg);
        // 按需启用
        if(m_rmDecoderNodeEnabled[mode] != 1) {
            m_rmDecoderNodeEnabled[mode] = 1;
            cuGraphNodeSetEnabled(graph_exec[1], m_rmDecoderNode[mode], 1);
        }
    }
    else
    {
        if(m_rmDecoderNodeEnabled[mode] != 0) {
            m_rmDecoderNodeEnabled[mode] = 0;
            cuGraphNodeSetEnabled(graph_exec[1], m_rmDecoderNode[mode], 0);
        }
    }

    // --- Polar 解码族 ---
    if(m_nPolUciSegs > 0 && !disableAllNodes)
    {
        cuGraphExecKernelNodeSetParams(graph_exec[1], m_compCwTreeTypesNode[mode], ...);
        cuGraphExecKernelNodeSetParams(graph_exec[1], m_polSegDeRmDeItlNode[mode], ...);
        cuGraphExecKernelNodeSetParams(graph_exec[1], m_polarDecoderNode[mode], ...);
        if(m_polarNodeEnabled[mode] != 1) {
            m_polarNodeEnabled[mode] = 1;
            cuGraphNodeSetEnabled(graph_exec[1], m_compCwTreeTypesNode[mode], 1);
            cuGraphNodeSetEnabled(graph_exec[1], m_polSegDeRmDeItlNode[mode], 1);
            cuGraphNodeSetEnabled(graph_exec[1], m_polarDecoderNode[mode], 1);
        }
    }

    // --- CSI-P2 节点 ---
    // --- LDPC + CRC 节点 ---
    // --- RSSI / RSRP 节点 ---

    // 最后上传执行图（Upload）
    cuGraphUpload(graphExec, phase1Stream);
}
```

### 5.3 `updateEarlyHarqGraph`

```cpp
void PuschRx::updateEarlyHarqGraph(bool disableAllNodes)
{
    // ChEst Early-HARQ 节点
    m_chest->earlyHarqGraph().setNodeStatus(disableAllNodes ? DISABLE_ALL : ENABLE, m_ehqGraphExec);

    // Polar 解码族（compCwTree + polSegDeRmDeItl + polarDecoder）
    if(m_nPolUciSegs_early > 0 && !disableAllNodes)
    {
        cuGraphExecKernelNodeSetParams(m_ehqGraphExec, m_ehqPolSegDeRmDeItlNode, ...);
        cuGraphExecKernelNodeSetParams(m_ehqGraphExec, m_ehqPolarDecoderNode, ...);
        if(m_ehqPolarNodeEnabled != 1) {
            m_ehqPolarNodeEnabled = 1;
            cuGraphNodeSetEnabled(m_ehqGraphExec, m_ehqCompCwTreeTypesNode, 1);
            cuGraphNodeSetEnabled(m_ehqGraphExec, m_ehqPolSegDeRmDeItlNode, 1);
            cuGraphNodeSetEnabled(m_ehqGraphExec, m_ehqPolarDecoderNode, 1);
        }
    }
    else
    {
        if(m_ehqPolarNodeEnabled != 0) {
            m_ehqPolarNodeEnabled = 0;
            cuGraphNodeSetEnabled(m_ehqGraphExec, m_ehqCompCwTreeTypesNode, 0);
            cuGraphNodeSetEnabled(m_ehqGraphExec, m_ehqPolSegDeRmDeItlNode, 0);
            cuGraphNodeSetEnabled(m_ehqGraphExec, m_ehqPolarDecoderNode, 0);
        }
    }

    // 上传执行图
    if(!disableAllNodes)
        cuGraphUpload(m_ehqGraphExec, phase1Stream);
}
```

### 5.4 `updateLaunchGraph`

```cpp
void PuschRx::updateLaunchGraph(CUgraphExec& graphExec,
    CUgraphNode& graphWaitNode, CUgraphNode& graphDglNode,
    cuphyPuschRxWaitLaunchCfg_t& waitCfg,
    cuphyPuschRxDglLaunchCfg_t&  dglCfg,
    bool enableDeviceGraphLaunch)
{
    // 更新 wait kernel 参数（包含当前时隙的等待配置）
    cuGraphExecKernelNodeSetParams(graphExec, graphWaitNode, &waitCfg.kernelNodeParamsDriver);

    // 按需启/停 DGL 节点
    if(enableDeviceGraphLaunch)
    {
        if(m_DeviceGraphLaunchNodeEnabled != 1) {
            m_DeviceGraphLaunchNodeEnabled = 1;
            cuGraphNodeSetEnabled(graphExec, graphDglNode, 1);
        }
        cuGraphExecKernelNodeSetParams(graphExec, graphDglNode, &dglCfg.kernelNodeParamsDriver);
    }
    else
    {
        if(m_DeviceGraphLaunchNodeEnabled != 0) {
            m_DeviceGraphLaunchNodeEnabled = 0;
            cuGraphNodeSetEnabled(graphExec, graphDglNode, 0);
        }
    }
}
```

---

## 6. 信道估计图节点管理

### 6.1 类层次

```
IChestGraphNodes (纯虚接口)
├── ChestNodes               ← Full-Slot ChEst 节点管理
└── NullChestGraph           ← 空实现（Null Object Pattern）

IChestSubSlotNodes (纯虚接口)
├── ChestSubSlotNodes        ← EarlyHarq / FrontDmrs ChEst 节点管理
└── NullChestSubSlotNodes    ← 空实现
```

### 6.2 `ChestNodes` 节点状态管理

```cpp
// 节点内部存储（每个时间实例 × 每个 HetCfg）
CUgraphNode m_nodes[CUPHY_PUSCH_RX_MAX_N_TIME_CH_EST][CUPHY_PUSCH_RX_CH_EST_ALL_ALGS_N_MAX_HET_CFGS];
CUgraphNode m_secondNodes[...];  // Filter Kernel（延迟估计路径）

// 状态更新
void ChestNodes::setNodeStatus(DisableAllNodes disableAllNodes, CUgraphExec graphExec)
{
    for(int timeInst = 0; timeInst < CUPHY_PUSCH_RX_MAX_N_TIME_CH_EST; ++timeInst)
        for(int cfg = 0; cfg < m_nMaxChEstHetCfgs; ++cfg)
            cuGraphNodeSetEnabled(graphExec, m_nodes[timeInst][cfg], enabled ? 1 : 0);
}

// disableNodes0Slot：禁用 slot0 的节点（用于 EarlyHarq 模式）
void ChestNodes::disableNodes0Slot(CUgraphExec graphExec)
{
    // 只禁用 timeInst == 0 的节点
}
```

---

## 7. 条件图节点（Conditional Graph Node）

### 7.1 Work Cancel 模式

| 模式 | 说明 |
|------|------|
| `PUSCH_NO_WORK_CANCEL` | 无工作取消，全图顺序执行 |
| `PUSCH_COND_IF_NODES_W_KERNEL` | 使用 CUDA 12.4+ `CU_GRAPH_NODE_TYPE_CONDITIONAL` |
| `PUSCH_DEVICE_GRAPHS` | 使用 Device Graph Launch（GPU 端触发子图） |

### 7.2 条件图结构（CUDA 12.4+）

```cpp
struct condGraphInfo {
    // 条件 Handle（绑定到 Full-Slot Graph）
    CUgraphConditionalHandle m_conditional_node_C0_handle;
    CUgraphConditionalHandle m_conditional_node_C1_handle;
    CUgraphConditionalHandle m_conditional_node_C2_handle;

    // 条件节点本身
    CUgraphNode m_graph_G0_cond_node;  // 在 fullSlotGraph 中
    CUgraphNode m_graph_G1_cond_node;  // 在 G0 子图中
    CUgraphNode m_graph_G2_cond_node;  // 在 G1 子图中

    // 每个条件节点内嵌一个独立子图
    CUgraph  m_graph[3];    // G0、G1、G2 的 CUgraph
    CUgraph* m_pGraph[3];   // 指向各自 graph 的指针
    CUgraphExec m_graphExec[3];
};
```

### 7.3 C0 条件节点创建流程（`enterConditionalStage0`）

```cpp
// 在 Full-Slot Graph 上创建条件 Handle
cuGraphConditionalHandleCreate(
    &condInfo.m_conditional_node_C0_handle,
    fullSlotGraph,
    current_context,
    DEFAULT_COND_VAL,       // 默认条件值（通常 1 = 执行）
    cond_handle_flags);

// 添加设置条件值的 Kernel 节点（PUSCH_COND_IF_NODES_W_KERNEL 模式）
cuGraphAddKernelNode(&condInfo.m_graph_G0_init_cond_node,
                     fullSlotGraph,
                     initialParents.data(), initialParents.size(),
                     &condInfo.m_init_C0_node_params);

// 创建条件节点（内嵌子图 G0）
CUgraphNodeParams condNodeParams = { CU_GRAPH_NODE_TYPE_CONDITIONAL };
condNodeParams.conditional.handle = condInfo.m_conditional_node_C0_handle;
condNodeParams.conditional.type   = CU_GRAPH_COND_TYPE_IF;
addGraphNodeHelper(&condInfo.m_graph_G0_cond_node,
                   fullSlotGraph,
                   deps.data(), deps.size(),
                   &condNodeParams);

// G0 内嵌子图由 condNodeParams 自动创建，通过 handle 获取
cuGraphConditionalHandleGetGraph(handle, &condInfo.m_graph[0]);
```

---

## 8. 完整调用流程图

```
PuschRx 构造
│
├── allocateDescr()                 各阶段描述符内存规划
├── createComponents()              创建 ChEst、NoiseIntfEst、CfoTaEst 等组件
│
├── createFullSlotGraph(LEGACY)     ─────────────────────────────────────────┐
│     cuGraphCreate()                                                         │
│     enterConditionalStage0()  → C0 根节点 / 子图 G0                        │
│     buildFrontEndStage()      → ChEst + NoiseEst + CfoTa + EqCoef 节点     │
│     buildSoftDemapStage()     → SoftDemap + SegLLRs0 节点                  │
│     enterConditionalStage1()  → C1 节点 / 子图 G1                          │
│     buildUciP1BackendStage()  → Simplex + RM + Polar 节点                  │
│     enterConditionalStage2()  → C2 节点 / 子图 G2                          │
│     buildCsiP2BackendStage()  → CSI-P2 控制 + 解码节点                     │
│     buildSchBackendStage()    → RateMatch + LDPC + CRC 节点                │
│     [DGL 模式] cuGraphInstantiate(G2/G1/G0) + 添加 DGL Kernel 节点         │
│   cuGraphInstantiate(&m_fullSlotGraphExec)                                  │
│   updateFullSlotGraph(true)   → 初始禁用所有节点                           │
│                                                                             │
├── createEarlyHarqGraph()          ─────────────────────────────────────────┘
│     cuGraphCreate()
│     添加 Root 空节点
│     ChEst Early Harq 节点（addKernelNodeToGraph）
│     NoiseIntfEst 节点
│     ChEqCoefComp 节点
│     Polar 解码族节点
│     RSRP/RSSI 节点
│   cuGraphInstantiate(&m_ehqGraphExec)
│   updateEarlyHarqGraph(true)
│
├── createLaunchGraph(preSubSlot)   → WaitKernel + EventRecord + DGL 节点
├── createLaunchGraph(preFullSlot)
│   cuGraphInstantiate(x2)
│
└── 构造完成
         │
         ▼ 每时隙
    setup(pDynPrm)
      ├── setupComponents()         更新各组件动态描述符
      ├── updateEarlyHarqGraph()    按需 enable/disable + 更新参数 + Upload
      ├── updateFrontLoadedDmrsGraph()
      ├── updateFullSlotGraph()     按需 enable/disable + 更新参数 + Upload
      └── updateLaunchGraph(x2)    更新 Wait/DGL 参数

    run(phase)
      ├── subSlotKernelLaunch()    cuGraphLaunch(m_ehqGraphExec / m_frontLoadedDmrsGraphExec)
      └── fullSlotKernelLaunch()   cuGraphLaunch(m_preSubSlotGraphExec)
                                   cuGraphLaunch(m_preFullSlotGraphExec)
                                   cuGraphLaunch(m_fullSlotGraphExec)
```

---

## 9. 各图节点完整加载顺序 DAG

本章以源码中 `cuGraphAddKernelNode` / `cuGraphAddEventRecordNode` / `addGraphNodeHelper` 的实际调用顺序，逐节点画出每张图的完整 DAG。括号内标注节点变量名，`×N` 表示循环添加 N 个并行节点（HetCfg 维度），箭头 `──►` 表示依赖（父→子）。

---

### 9.1 Launch Graph（`m_preSubSlotGraph` / `m_preFullSlotGraph`）

两张 Launch 图结构相同，创建函数 `createLaunchGraph()`。

```
加载顺序：①②③ 严格串行

cuGraphCreate()
    │
    ① cuGraphAddKernelNode
    │     [graphWaitNode]   ← 无父节点（根节点）
    │     自旋等待指定数量 OFDM 符号到达
    │
    ▼
    ② cuGraphAddEventRecordNode
    │     [graphEventNode]  ← 父: graphWaitNode
    │     通知 PUSCH 流水线可以开始
    │
    ▼
    ③ cuGraphAddKernelNode
          [graphDglNode]    ← 父: graphEventNode
          Device Graph Launch（可被 disable）

  Green-Context 模式下只有 ③，跳过 ①②
```

---

### 9.2 Early-HARQ Graph（`m_ehqGraph`）

创建函数 `createEarlyHarqGraph()`，节点按以下顺序加入图：

```
cuGraphCreate()
    │
    ① cuGraphAddKernelNode
    │     [m_ehqRootNode]                      ← 无父节点（空 Kernel，统一 fanout 根）
    │
    ▼ (父: m_ehqRootNode)
    ┌──────────────────────────────────────────────────────────────────┐
    │  ②  ChEst 子图节点（earlyHarqGraph().addKernelNodeToGraph）      │
    │      [m_ehqChEstNodes[0..N-1]]            ← 父: m_ehqRootNode    │
    │      若算法含 Filter-Kernel：                                     │
    │      [m_ehqChEstSecondNodes[0..N-1]]      ← 父: m_ehqChEstNodes  │
    └──────────────────────────────────────────────────────────────────┘
    │  输出节点集合记为 ChEstOut
    │
    ├──────────────────────────── 并行分支 ────────────────────────────┤
    │                                                                   │
    ▼ (父: ChEstOut)                                                   │
    ③ cuGraphAddKernelNode ×NOISE_INTF_N_MAX_HET_CFGS                 │
    │     [m_ehqNoiseIntfEstNodes[0..N-1]]                             │
    │     （条件：enableSinrMeasurement || MMSE 均衡）                  │
    │                                                                   │
    ▼ (父: ChEstOut ∪ m_ehqNoiseIntfEstNodes)                         │
    ④ cuGraphAddKernelNode ×CH_EQ_N_MAX_HET_CFGS                      │
    │     [m_ehqChEqCoefCompNodes[0..N-1]]                             │
    │                                                                   │
    ▼ (父: m_ehqChEqCoefCompNodes)                                     │
    ⑤ cuGraphAddKernelNode ×CH_EQ_N_MAX_HET_CFGS                      │
    │     [m_ehqChEqSoftDemapNodes[0..N-1]]                            │
    │                                                                   │
    │  DFT-S-OFDM 可选路径（enableDftSOfdm==1）：                       │
    │  ⑤a cuGraphAddKernelNode ×CH_EQ_N_MAX_HET_CFGS                  │
    │        [m_ehqChEqSoftDemapIdftNodes[0..N-1]]                     │
    │        ← 父: m_ehqChEqSoftDemapNodes                             │
    │  ⑤b cuGraphAddKernelNode ×CH_EQ_N_MAX_HET_CFGS                  │
    │        [m_ehqChEqSoftDemapAfterDftNodes[0..N-1]]                 │
    │        ← 父: m_ehqChEqSoftDemapIdftNodes                         │
    │                                                                   │
    │  SoftDemapOut = 末端 SoftDemap 节点集合                           │
    │                                                                   │
    ├──── 以下均以 SoftDemapOut 为父，互相并行 ────────────────────────┤
    │                  │                  │                │            │
    ▼                  ▼                  ▼                ▼            │
    ⑥                 ⑦                  ⑧                ⑨            │
[m_ehqCompCwTreeTypesNode] [m_ehqRsrpNodes×N] [m_ehqRssiNodes×N] [m_ehqUciSegLLRs0Node]
    │                                                        │
    └───────────────────────────┬────────────────────────────┘
                                │ 父: m_ehqUciSegLLRs0Node
            ┌───────────────────┼───────────────────────────────────────┐
            ▼                   ▼                                       │
    ⑩ [m_ehqSimplexDecoderNode]  ⑪ [m_ehqRmDecoderNode]                │
       父: m_ehqUciSegLLRs0Node     父: m_ehqUciSegLLRs0Node             │
                                                                        │
            ▼ (父: m_ehqUciSegLLRs0Node + m_ehqCompCwTreeTypesNode)     │
    ⑫ [m_ehqPolSegDeRmDeItlNode]  ←──────────────────────────────────┘
            │
            ▼ (父: m_ehqPolSegDeRmDeItlNode)
    ⑬ [m_ehqPolarDecoderNode]
```

**加载顺序汇总（按 `cuGraphAddKernelNode` 调用次序）：**

| 序号 | 节点 | 父节点 |
|------|------|--------|
| ① | `m_ehqRootNode` | 无（根） |
| ② | `m_ehqChEstNodes[0..N]`（+SecondNodes） | `m_ehqRootNode` |
| ③ | `m_ehqNoiseIntfEstNodes[0..N]` | ChEstOut |
| ④ | `m_ehqChEqCoefCompNodes[0..N]` | ChEstOut ∪ NoiseIntfOut |
| ⑤ | `m_ehqChEqSoftDemapNodes[0..N]` | CoefCompOut |
| ⑤a | `m_ehqChEqSoftDemapIdftNodes[0..N]`（可选） | SoftDemapOut |
| ⑤b | `m_ehqChEqSoftDemapAfterDftNodes[0..N]`（可选） | IdftOut |
| ⑥ | `m_ehqCompCwTreeTypesNode` | SoftDemapOut |
| ⑦ | `m_ehqRsrpNodes[0..N]`（可选） | SoftDemapOut |
| ⑧ | `m_ehqRssiNodes[0..N]`（可选） | SoftDemapOut |
| ⑨ | `m_ehqUciSegLLRs0Node` | SoftDemapOut |
| ⑩ | `m_ehqSimplexDecoderNode` | `m_ehqUciSegLLRs0Node` |
| ⑪ | `m_ehqRmDecoderNode` | `m_ehqUciSegLLRs0Node` |
| ⑫ | `m_ehqPolSegDeRmDeItlNode` | `m_ehqUciSegLLRs0Node` + `m_ehqCompCwTreeTypesNode` |
| ⑬ | `m_ehqPolarDecoderNode` | `m_ehqPolSegDeRmDeItlNode` |

---

### 9.3 Front-Loaded DMRS Graph（`m_frontLoadedDmrsGraph`）

创建函数 `createFrontLoadedDmrsGraph()`，是 Early-HARQ 图的前几个阶段（只到均衡系数计算，不含 SoftDemap 及之后）：

```
cuGraphCreate()
    │
    ① cuGraphAddKernelNode
    │     [m_frontLoadedDmrsRootNode]           ← 无父节点（根）
    │
    ▼ (父: m_frontLoadedDmrsRootNode)
    ② ChEst 子图节点（frontDmrsGraph().addKernelNodeToGraph）
    │     [m_frontLoadedDmrsChEstNodes[0..N]]
    │     （若 Filter-Kernel：m_frontLoadedDmrsChEstSecondNodes[0..N]）
    │     ChEstOut = 末端 ChEst 节点集合
    │
    ├──────────────── 并行 ─────────────────────┤
    │                                           │
    ▼ (父: ChEstOut)                            │
    ③ cuGraphAddKernelNode ×NOISE_INTF_N         │
    │     [m_frontLoadedDmrsNoiseIntfEstNodes[0..N]]
    │     （条件：enableSinrMeasurement || MMSE）│
    │                                           │
    ▼ (父: ChEstOut ∪ NoiseIntfOut)             │
    ④ cuGraphAddKernelNode ×CH_EQ_N              │
          [m_frontLoadedDmrsChEqCoefCompNodes[0..N]]

  ← 图在此结束，不含 SoftDemap 及后续节点 →
```

| 序号 | 节点 | 父节点 |
|------|------|--------|
| ① | `m_frontLoadedDmrsRootNode` | 无（根） |
| ② | `m_frontLoadedDmrsChEstNodes[0..N]`（+Second） | `m_frontLoadedDmrsRootNode` |
| ③ | `m_frontLoadedDmrsNoiseIntfEstNodes[0..N]`（可选） | ChEstOut |
| ④ | `m_frontLoadedDmrsChEqCoefCompNodes[0..N]` | ChEstOut ∪ NoiseIntfOut |

---

### 9.4 Full-Slot Graph（`m_fullSlotGraph` / `m_frontLoadedDmrsFullSlotGraph`）

创建函数 `createFullSlotGraph()`，由五个 Stage 构建器串联，中间可插入条件/设备子图节点。以下按**三种工作取消模式**分别展示。

#### 9.4.1 模式 A：`PUSCH_NO_WORK_CANCEL`（无条件节点，单图）

全部节点加入同一张 `fullSlotGraph`，G0/G1/G2 三段共用同一图指针。

```
cuGraphCreate(&fullSlotGraph)
    │
    ══════════════════ G0 段（前端 + 软解映射）══════════════════
    │
    ① cuGraphAddKernelNode
    │     [emptyRootNode]                        ← 无父（根）
    │
    ▼ (父: emptyRootNode)
    ②─── buildFrontEndStage() ─────────────────────────────────┐
    │                                                            │
    │  Path A（enableWeightedAverageCfo == 0）：                 │
    │                                                            │
    │  ②a ChEst（chestGraph().addKernelNodeToGraph）             │
    │       [m_chEstNodes[t][0..N]]  ← 父: emptyRootNode        │
    │       [m_chEstSecondNodes[t][0..N]]（Filter-Kernel）       │
    │       ChEstOut = 所有末端 ChEst 节点                       │
    │                                                            │
    │  以下三组以 ChEstOut 为父，互相并行：                       │
    │                                                            │
    │  ②b cuGraphAddKernelNode ×NOISE_INTF_N                    │
    │       [m_noiseIntfEstNodes[0..N]]   ← 父: ChEstOut        │
    │       （条件：SINR || MMSE）                               │
    │                                                            │
    │  ②c cuGraphAddKernelNode ×CFO_EST_N                       │
    │       [m_cfoTaEstNodes[0..N]]       ← 父: ChEstOut        │
    │       （条件：CfoCorrection || ToEst）                     │
    │                                                            │
    │  eqParents = ChEstOut ∪ NoiseIntfOut ∪ CfoTaOut           │
    │                                                            │
    │  ②d cuGraphAddKernelNode ×TIME×CH_EQ_N                    │
    │       [m_chEqCoefCompNodes[t][0..N]] ← 父: eqParents      │
    │       （enablePuschTdi 时 TIME=MAX_N_TIME_CH_EQ，否则1）   │
    │                                                            │
    │  Path B（enableWeightedAverageCfo == 1）：                 │
    │   串行链：ChEst → NoiseIntfEst → RSRP → CfoTaEst → EqCoef│
    │   ②b' [m_noiseIntfEstNodes]  ← 父: ChEstOut               │
    │   ②c' [m_rsrpNodes[mode][0..N]] ← 父: NoiseIntfOut        │
    │   ②d' [m_cfoTaEstNodes[0..N]]   ← 父: RsrpOut             │
    │   ②e' [m_chEqCoefCompNodes[t][0..N]] ← 父: CfoTaOut       │
    │                                                            │
    └─────── frontEndStage.terminalNodes = EqCoefCompOut ───────┘
    │
    ▼ (父: EqCoefCompOut)
    ③─── buildSoftDemapStage() ────────────────────────────────┐
    │                                                            │
    │  ③a cuGraphAddKernelNode ×CH_EQ_N                         │
    │       [m_chEqSoftDemapNodes[mode][0..N]]                  │
    │                                                            │
    │  DFT-S-OFDM 可选：                                         │
    │  ③b [m_chEqSoftDemapIdftNodes[mode][0..N]]                │
    │       ← 父: SoftDemapNodes                                │
    │  ③c [m_chEqSoftDemapAfterDftNodes[mode][0..N]]            │
    │       ← 父: IdftNodes                                     │
    │                                                            │
    └─────── softDemapStage.terminalNodes = SoftDemapOut ───────┘
    │
    ══════════════════ G1 段（UCI-P1 后端）══════════════════════
    │  （PUSCH_NO_WORK_CANCEL 时仍在同一图，无 C1 节点）
    │
    ▼ (父: SoftDemapOut)
    ④─── buildUciP1BackendStage() ─────────────────────────────┐
    │                                                            │
    │  以 SoftDemapOut 为父，以下 4 节点并行：                   │
    │                                                            │
    │  ④a [m_compCwTreeTypesNode[mode]]  ← 父: SoftDemapOut     │
    │                                                            │
    │  ④b [m_rsrpNodes[mode][0..N]]      ← 父: SoftDemapOut     │
    │      （条件：SINR && !WeightedAvgCfo，或 FrontLoaded 模式）│
    │                                                            │
    │  ④c [m_rssiNodes[mode][0..N]]      ← 父: SoftDemapOut     │
    │      （条件：enableRssiMeasurement）                       │
    │                                                            │
    │  ④d [m_uciSegLLRs0Node[mode]]      ← 父: SoftDemapOut     │
    │                                                            │
    │  以 m_uciSegLLRs0Node 为父（串行或并行）：                 │
    │                                                            │
    │  ④e [m_simplexDecoderNode[mode]]   ← 父: uciSegLLRs0      │
    │  ④f [m_rmDecoderNode[mode]]        ← 父: uciSegLLRs0      │
    │  ④g [m_polSegDeRmDeItlNode[mode]]  ← 父: uciSegLLRs0      │
    │                                                │              │
    │                               + compCwTreeTypesNode ─────────┘
    │  ④h [m_polarDecoderNode[mode]]     ← 父: polSegDeRmDeItl  │
    │                                                            │
    └─────── uciP1Stage.terminalNodes =                         │
             {simplex, rm, polar}  ─────────────────────────────┘
    │
    ══════════════════ G2 段（CSI-P2 + SCH）════════════════════
    │  （PUSCH_NO_WORK_CANCEL 时仍在同一图，无 C2 节点）
    │
    ▼ (父: {simplex, rm, polar})
    ⑤─── buildCsiP2BackendStage() ─────────────────────────────┐
    │                                                            │
    │  ⑤a [m_uciOnPuschCsi2CtrlNode[mode]]   ← 父: UCI-P1 out  │
    │                                                            │
    │  以 Csi2Ctrl 为父：                                        │
    │  ⑤b [m_uciOnPuschCsi2CompCwTreeTypesNode[mode]]           │
    │  ⑤c [m_uciOnPuschCsi2SegLLRs2Node[mode]]                  │
    │      （两者并行，父均为 Csi2Ctrl）                          │
    │                                                            │
    │  以 SegLLRs2 为父（三解码并行）：                          │
    │  ⑤d [m_uciOnPuschCsi2rmDecoderNode[mode]]                 │
    │  ⑤e [m_uciOnPuschCsi2simplexDecoderNode[mode]]            │
    │  ⑤f [m_uciOnPuschCsi2PolSegDeRmDeItlNode[mode]]           │
    │       ← 父: SegLLRs2 + CompCw2                             │
    │  ⑤g [m_uciOnPuschCsi2PolarDecoderNode[mode]]              │
    │       ← 父: PolSegDeRmDeItl2                               │
    │                                                            │
    └─────── csi2Stage.terminalNodes = {SegLLRs2} ──────────────┘
    │
    ▼ (父: SegLLRs2)
    ⑥─── buildSchBackendStage() ────────────────────────────────┐
    │                                                            │
    │  ⑥a [m_resetRateMatchNode[mode]]   ← 父: SegLLRs2         │
    │  ⑥b [m_rateMatchNode[mode]]        ← 父: resetRateMatch   │
    │  ⑥c [m_clampRateMatchNode[mode]]   ← 父: rateMatch        │
    │                                                            │
    │  以 clampRateMatch 为父（并行 LDPC）：                     │
    │  ⑥d [m_ldpcDecoderNodes[mode][0..nMaxLdpcHetConfigs-1]]   │
    │                                                            │
    │  以 LDPC 输出为父：                                        │
    │  ⑥e [m_crcNodes[mode][0]]          ← 父: all LDPC         │
    │  ⑥f [m_crcNodes[mode][1]]          ← 父: crcNodes[0]      │
    │                                                            │
    └─────── schStage.terminalNodes = {crcNodes[1]} ────────────┘
```

**Full-Slot Graph 完整节点加载顺序表（PUSCH_NO_WORK_CANCEL 模式）：**

| 序号 | 节点 | 函数 | 父节点 |
|------|------|------|--------|
| ① | `emptyRootNode` | `enterConditionalStage0` | 无 |
| ②a | `m_chEstNodes[t][0..N]`（+Second） | `buildFrontEndStage` | `emptyRootNode` |
| ②b | `m_noiseIntfEstNodes[0..N]`（可选） | `buildFrontEndStage` | ChEstOut |
| ②c | `m_cfoTaEstNodes[0..N]`（可选） | `buildFrontEndStage` | ChEstOut |
| ②d | `m_chEqCoefCompNodes[t][0..N]` | `buildFrontEndStage` | ChEstOut∪NoiseIntf∪CfoTa |
| ③a | `m_chEqSoftDemapNodes[mode][0..N]` | `buildSoftDemapStage` | EqCoefCompOut |
| ③b | `m_chEqSoftDemapIdftNodes[mode][0..N]`（可选） | `buildSoftDemapStage` | SoftDemapNodes |
| ③c | `m_chEqSoftDemapAfterDftNodes[mode][0..N]`（可选） | `buildSoftDemapStage` | IdftNodes |
| ④a | `m_compCwTreeTypesNode[mode]` | `buildUciP1BackendStage` | SoftDemapOut |
| ④b | `m_rsrpNodes[mode][0..N]`（可选） | `buildUciP1BackendStage` | SoftDemapOut |
| ④c | `m_rssiNodes[mode][0..N]`（可选） | `buildUciP1BackendStage` | SoftDemapOut |
| ④d | `m_uciSegLLRs0Node[mode]` | `buildUciP1BackendStage` | SoftDemapOut |
| ④e | `m_simplexDecoderNode[mode]` | `buildUciP1BackendStage` | uciSegLLRs0 |
| ④f | `m_rmDecoderNode[mode]` | `buildUciP1BackendStage` | uciSegLLRs0 |
| ④g | `m_polSegDeRmDeItlNode[mode]` | `buildUciP1BackendStage` | uciSegLLRs0 + compCwTree |
| ④h | `m_polarDecoderNode[mode]` | `buildUciP1BackendStage` | polSegDeRmDeItl |
| ⑤a | `m_uciOnPuschCsi2CtrlNode[mode]` | `buildCsiP2BackendStage` | {simplex,rm,polar} |
| ⑤b | `m_uciOnPuschCsi2CompCwTreeTypesNode[mode]` | `buildCsiP2BackendStage` | Csi2Ctrl |
| ⑤c | `m_uciOnPuschCsi2SegLLRs2Node[mode]` | `buildCsiP2BackendStage` | Csi2Ctrl |
| ⑤d | `m_uciOnPuschCsi2rmDecoderNode[mode]` | `buildCsiP2BackendStage` | SegLLRs2 |
| ⑤e | `m_uciOnPuschCsi2simplexDecoderNode[mode]` | `buildCsiP2BackendStage` | SegLLRs2 |
| ⑤f | `m_uciOnPuschCsi2PolSegDeRmDeItlNode[mode]` | `buildCsiP2BackendStage` | SegLLRs2+CompCw2 |
| ⑤g | `m_uciOnPuschCsi2PolarDecoderNode[mode]` | `buildCsiP2BackendStage` | PolSegDeRmDeItl2 |
| ⑥a | `m_resetRateMatchNode[mode]` | `buildSchBackendStage` | SegLLRs2 |
| ⑥b | `m_rateMatchNode[mode]` | `buildSchBackendStage` | resetRateMatch |
| ⑥c | `m_clampRateMatchNode[mode]` | `buildSchBackendStage` | rateMatch |
| ⑥d | `m_ldpcDecoderNodes[mode][0..K-1]` | `buildSchBackendStage` | clampRateMatch |
| ⑥e | `m_crcNodes[mode][0]` | `buildSchBackendStage` | all LDPC |
| ⑥f | `m_crcNodes[mode][1]` | `buildSchBackendStage` | crcNodes[0] |

---

#### 9.4.2 模式 B：`PUSCH_COND_IF_NODES_W_KERNEL`（条件 IF 节点，CUDA 12.4+）

在模式 A 的基础上，三个切割点各插入一对节点（`initCondKernel` + `condNode`）：

```
fullSlotGraph（主图）
    │
    ┌──── C0 段（当 USE_COND_GRAPH_NODE_C0==1）────────────────────┐
    │  [m_graph_G0_init_cond_node]   ← 父: 无（根节点，无额外父）  │
    │         ↓                                                    │
    │  [m_graph_G0_cond_node]        ← 父: init_C0               │
    │    （CU_GRAPH_COND_TYPE_IF，内嵌子图 = G0）                  │
    └───────────────────────────────────────────────────────────── ┘
    │  以下节点全部加入 G0 内嵌子图（condInfo.m_pGraph[0]）
    │
    ②a~②d  ChEst / NoiseIntf / CfoTa / EqCoefComp   （同 9.4.1）
    ③a~③c  SoftDemap（+可选 IDFT）                   （同 9.4.1）
    │
    ┌──── C1 段（当 USE_COND_GRAPH_NODE_C1==1）────────────────────┐
    │  [m_graph_G1_init_cond_node]   ← 父: SoftDemapOut（加入G0）  │
    │         ↓                                                    │
    │  [m_graph_G1_cond_node]        ← 父: init_C1（加入G0）       │
    │    （CU_GRAPH_COND_TYPE_IF，内嵌子图 = G1）                  │
    └──────────────────────────────────────────────────────────────┘
    │  以下节点全部加入 G1 内嵌子图（condInfo.m_pGraph[1]）
    │
    ④a~④h  UCI-P1 backend                            （同 9.4.1）
    │
    ┌──── C2 段（当 USE_COND_GRAPH_NODE_C2==1）────────────────────┐
    │  [m_graph_G2_init_cond_node]   ← 父: UCI-P1 out（加入G1）    │
    │         ↓                                                    │
    │  [m_graph_G2_cond_node]        ← 父: init_C2（加入G1）       │
    │    （CU_GRAPH_COND_TYPE_IF，内嵌子图 = G2）                  │
    └──────────────────────────────────────────────────────────────┘
    │  以下节点全部加入 G2 内嵌子图（condInfo.m_pGraph[2]）
    │
    ⑤a~⑤g  CSI-P2 backend                            （同 9.4.1）
    ⑥a~⑥f  SCH backend（LDPC + CRC）                  （同 9.4.1）
```

---

#### 9.4.3 模式 C：`PUSCH_DEVICE_GRAPHS`（设备图三级链）

三张独立子图分别创建，通过 DGL Kernel 链式触发：

```
fullSlotGraph（Host 图）
    │
    ① [m_graph_G0_init_cond_node]  ← 父: 无（根节点）
       DGL Kernel：在 GPU 端触发 G0（condInfo.m_graphExec[0]）

condInfo.m_graph[0] = G0（Device-launchable 图）
    │
    ②a~②d  ChEst / NoiseIntf / CfoTa / EqCoefComp
    ③a~③c  SoftDemap（+可选 IDFT）
    │
    末尾追加（在 G0 中，父为 SoftDemapOut）：
    [m_graph_G1_init_cond_node]   DGL Kernel：触发 G1

condInfo.m_graph[1] = G1（Device-launchable 图）
    │
    ④a~④h  UCI-P1 backend
    │
    末尾追加（在 G1 中，父为 UCI-P1 Out）：
    [m_graph_G2_init_cond_node]   DGL Kernel：触发 G2

condInfo.m_graph[2] = G2（Device-launchable 图）
    │
    ⑤a~⑤g  CSI-P2 backend
    ⑥a~⑥f  SCH backend（LDPC + CRC）

DGL 节点实际添加顺序（反向实例化）：
  1. 先实例化 G2 → 在 G1 末尾加 DGL-G2 节点
  2. 再实例化 G1 → 在 G0 末尾加 DGL-G1 节点
  3. 最后实例化 G0 → 在 fullSlotGraph 加 DGL-G0 节点
```

---

### 9.5 四图节点总览对照表

| 节点 | LaunchGraph | EarlyHarqGraph | FrontLoadedDmrsGraph | FullSlotGraph |
|------|:-----------:|:--------------:|:--------------------:|:-------------:|
| WaitKernel | ✓ | — | — | — |
| EventRecord | ✓ | — | — | — |
| DGL Kernel | ✓ | — | — | ✓（DGL模式） |
| 空根节点 | — | ✓ | ✓ | ✓ |
| ChEstNodes（PreKernel） | — | ✓ | ✓ | ✓ |
| ChEstSecondNodes（FilterKernel） | — | 可选 | 可选 | 可选 |
| NoiseIntfEstNodes | — | ✓ | ✓ | ✓ |
| CfoTaEstNodes | — | — | — | ✓ |
| RsrpNodes | — | ✓ | — | ✓ |
| RssiNodes | — | ✓ | — | ✓ |
| ChEqCoefCompNodes | — | ✓ | ✓ | ✓ |
| ChEqSoftDemapNodes | — | ✓ | — | ✓ |
| ChEqSoftDemapIdftNodes | — | 可选 | — | 可选 |
| ChEqSoftDemapAfterDftNodes | — | 可选 | — | 可选 |
| CompCwTreeTypesNode | — | ✓ | — | ✓ |
| UciSegLLRs0Node | — | ✓ | — | ✓ |
| SimplexDecoderNode | — | ✓ | — | ✓ |
| RmDecoderNode | — | ✓ | — | ✓ |
| PolSegDeRmDeItlNode | — | ✓ | — | ✓ |
| PolarDecoderNode | — | ✓ | — | ✓ |
| Csi2CtrlNode | — | — | — | ✓ |
| Csi2CompCwTreeTypesNode | — | — | — | ✓ |
| Csi2SegLLRs2Node | — | — | — | ✓ |
| Csi2RmDecoderNode | — | — | — | ✓ |
| Csi2SimplexDecoderNode | — | — | — | ✓ |
| Csi2PolSegDeRmDeItlNode | — | — | — | ✓ |
| Csi2PolarDecoderNode | — | — | — | ✓ |
| ResetRateMatchNode | — | — | — | ✓ |
| RateMatchNode | — | — | — | ✓ |
| ClampRateMatchNode | — | — | — | ✓ |
| LdpcDecoderNodes | — | — | — | ✓ |
| CrcNodes[0] | — | — | — | ✓ |
| CrcNodes[1] | — | — | — | ✓ |
| C0/C1/C2 initCondKernel | — | — | — | 可选 |
| C0/C1/C2 condNode | — | — | — | 可选 |

---

## 总结

| 特性 | 实现方式 |
|------|---------|
| 图拆分粒度 | 4 类图（Launch / EarlyHarq / FrontLoadedDmrs / FullSlot） |
| 异构配置并行 | 每阶段按 `nMaxHetCfgs` 循环添加多个并行 Kernel 节点 |
| 动态开关节点 | `cuGraphNodeSetEnabled` + 状态标志位（避免重复调用） |
| 参数更新 | `cuGraphExecKernelNodeSetParams` 直接修改 Exec 图 |
| 工作取消 | 三模式：无取消 / Conditional-IF 节点 / Device Graph Launch |
| 信道估计节点 | 通过 `IChestGraphNodes` 接口抽象，支持 Null Object 模式 |
| CUDA 版本兼容 | `addGraphNodeHelper` 封装 CUDA 13 的 edge data 参数差异 |
| 图上传 | `cuGraphUpload` 在 `update` 阶段末尾显式上传，减少首次执行延迟 |
