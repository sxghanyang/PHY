# cuphycontroller_scf — main() 入口调用关系分析

## 1. 总体执行流程

```
main()
 ├─ 1. 基础环境初始化
 │    ├─ signal(SIGINT/SIGTERM, sig_handler)   // 信号注册
 │    ├─ AppConfig::instance().load(yaml)       // YAML 配置加载 (单例)
 │    └─ nvlog_open(...)                        // 日志系统初始化
 │
 ├─ 2. CUDA / 硬件初始化
 │    ├─ cuInit(0)
 │    ├─ cudaSetDevice(gpu_id)
 │    └─ cudaGetDeviceProperties(...)
 │
 ├─ 3. PHY Driver 初始化  ★ 核心路径
 │    └─ pc_init_phydriver(...)                 // cuphycontroller/src/cuphydriver.cpp
 │         └─ l1_init(cfg)                      // cuphydriver/src/common/cuphydriver_api.cpp
 │              └─ PhyDriverCtx 构造            // cuphydriver/src/common/context.cpp
 │                   ├─ MpsCtx (MPS/GreenContext GPU分区)
 │                   ├─ CUDA Streams (per-cell, 优先级 -5 ~ -1)
 │                   ├─ PhyXxxAggr 对象池 (PDSCH/PUSCH/PRACH/PUCCH/SRS...)
 │                   ├─ FhProxy (Fronthaul 代理)
 │                   └─ Worker threads
 │
 ├─ 4. 运行模式分支
 │    ├─ [Standalone 模式] → 直接 cell_create / cell_start → 事件循环
 │    └─ [FAPI 模式] ★ 主流路径
 │         ├─ PHY_group 创建                    // cuphyl2adapter/lib/nvPHY/nv_phy_group.hpp
 │         │    └─ PHY_module 创建              // nv_phy_module.hpp
 │         │         └─ PHYDriverProxy::instance()  // 单例, 包装 l1_cell_create/start/stop
 │         └─ scf_5g_fapi::phy 创建            // scfl2adapter/lib/scf_5g_fapi/scf_5g_fapi_phy.cpp
 │              ├─ FAPI P5/P7 消息处理
 │              ├─ nvIPC 通信通道 → L2 (testMAC / 外部 L2)
 │              └─ Slot indication 驱动调度
 │
 ├─ 5. 主循环
 │    └─ while(!g_quit) { sleep / event_wait }
 │
 └─ 6. 清理
      ├─ scf_5g_fapi::phy 销毁
      ├─ PHY_group 销毁
      ├─ pc_close_phydriver() → l1_shutdown()
      ├─ nvlog_close()
      └─ CUDA 资源释放
```

## 2. 关键调用链详解

### 2.1 PHY Driver 初始化链

```
pc_init_phydriver(cuphycontroller_scf.cpp)
  │
  ├─ 解析 YAML 配置 → PhyDriverConfig
  │
  └─ l1_init(PhyDriverConfig)                    [cuphydriver_api.cpp]
       │
       ├─ PhyDriverCtx::PhyDriverCtx(cfg)        [context.cpp]
       │    │
       │    ├─ MpsCtx 初始化                      [mps.cpp]
       │    │    ├─ cuDeviceGet()
       │    │    ├─ cuDeviceGetAttribute(MULTIPROCESSOR_COUNT)
       │    │    ├─ 若启用 MPS:
       │    │    │    ├─ cuGreenCtxCreate() × N   // 每 cell 一个 Green Context
       │    │    │    └─ SM 分区 (per-cell SM count 配置)
       │    │    └─ 若未启用: 使用默认 CUDA context
       │    │
       │    ├─ CUDA Stream 创建 (per-cell)
       │    │    ├─ DL stream  (priority -5, 最高)
       │    │    ├─ UL stream  (priority -3)
       │    │    ├─ PRACH stream (priority -1)
       │    │    └─ Aux streams
       │    │
       │    ├─ PhyXxxAggr 聚合对象构造
       │    │    ├─ PhyPdschAggr → cuPHY PDSCH TX pipeline
       │    │    ├─ PhyPuschAggr → cuPHY PUSCH RX pipeline
       │    │    ├─ PhyPrachAggr → cuPHY PRACH RX pipeline
       │    │    ├─ PhyPucchAggr → cuPHY PUCCH RX pipeline
       │    │    └─ PhySrsAggr  → cuPHY SRS pipeline
       │    │
       │    ├─ FhProxy 初始化
       │    │    ├─ aerial-fh-driver 绑定
       │    │    └─ DPDK/DOCA hugepage 缓冲区映射
       │    │
       │    └─ Worker 线程启动
       │         ├─ DL worker (slot-based, FH 发送)
       │         └─ UL worker (接收 + 解码)
       │
       └─ 返回 phy_handle (不透明句柄)
```

### 2.2 FAPI 模式运行时调用链

```
scf_5g_fapi::phy
  │
  ├─ P5 接口 (配置)
  │    ├─ PARAM.request / PARAM.response
  │    ├─ CONFIG.request → PHYDriverProxy::cell_config()
  │    │    └─ l1_cell_create(phy_handle, cell_cfg)    [cuphydriver_api.cpp]
  │    │         └─ PhyDriverCtx::cellCreate(cell_id)
  │    │              ├─ 分配 cell-specific GPU 资源
  │    │              ├─ 绑定 MPS Green Context
  │    │              └─ 初始化 per-cell pipeline state
  │    └─ START.request → PHYDriverProxy::cell_start()
  │         └─ l1_cell_start(phy_handle, cell_id)
  │              └─ PhyDriverCtx::cellStart(cell_id)
  │                   ├─ 启动 FH 收发
  │                   └─ 启动 slot timer
  │
  ├─ P7 接口 (实时数据) — 每 slot 调用
  │    ├─ DL_TTI.request → 组装 DL 任务
  │    │    ├─ TX_DATA.request (传输块)
  │    │    └─ → PhyPdschAggr::enqueue() → CUDA kernel launch
  │    │         └─ FhProxy::send() → RU
  │    │
  │    ├─ UL_TTI.request → 配置 UL 接收窗口
  │    │    └─ FhProxy::recv() → PhyPuschAggr::decode()
  │    │         └─ CRC.indication / RX_DATA.indication → L2
  │    │
  │    ├─ UL_DCI.request → DCI format 组装
  │    └─ SLOT.indication → 触发下一 slot 调度
  │
  └─ nvIPC 通信
       ├─ SHM 模式: 共享内存 + eventfd
       ├─ DPDK 模式: rte_ring
       └─ GPU 模式: GDRCopy / CUDA IPC
```

### 2.3 Cell 生命周期

```
l1_init()          → PhyDriverCtx 创建 (全局, 一次性)
l1_cell_create()   → Per-cell 资源分配 (GPU mem, streams, pipelines)
l1_cell_start()    → 激活 FH, 开始 slot 处理
  ↓ 运行中...
l1_cell_stop()     → 停止 slot 处理, 停止 FH
l1_cell_delete()   → 释放 per-cell 资源
l1_shutdown()      → PhyDriverCtx 销毁 (全局清理)
```

## 3. 线程模型

| 线程 | 职责 | 绑核 |
|------|------|------|
| Main thread | 初始化 + 信号等待 | 可配置 |
| FAPI RX thread | nvIPC 接收 L2 消息 | 可配置 |
| DL Worker | PDSCH 编码 + FH 发送 | 可配置 (isolcpus) |
| UL Worker | FH 接收 + PUSCH/PUCCH/PRACH 解码 | 可配置 (isolcpus) |
| FH TX thread | 以太网帧发送 (DPDK poll) | 可配置 |
| FH RX thread | 以太网帧接收 (DPDK poll) | 可配置 |
| OAM thread | gRPC 管理接口 | — |

## 4. 关键设计模式

### 4.1 单例模式
- **AppConfig::instance()** — 全局 YAML 配置访问
- **PHYDriverProxy::instance()** — L1 驱动的唯一入口, 封装 `l1_*()` C API

### 4.2 不透明句柄模式
- `l1_init()` 返回 `phy_handle_t` (即 `void*` 指向 `PhyDriverCtx`)
- 所有后续 API (`l1_cell_create/start/stop/delete/shutdown`) 传入此句柄
- 隐藏内部实现, C ABI 兼容

### 4.3 聚合器模式 (PhyXxxAggr)
- 每个物理信道有一个 Aggr 对象
- 负责: 参数收集 → buffer 分配 → cuPHY pipeline 调用 → 结果回收
- Pre-allocated buffer pools, 避免运行时 malloc

### 4.4 代理/适配器层
```
L2 (testMAC / 外部)
  ↕ nvIPC
scf_5g_fapi::phy        ← SCF FAPI 标准接口适配
  ↕
PHY_module / PHY_group   ← 内部抽象层
  ↕
PHYDriverProxy           ← 单例, C++ 包装
  ↕
l1_*() C API             ← cuphydriver 公开接口
  ↕
PhyDriverCtx             ← 实际实现 (CUDA, FH, pipelines)
```

## 5. 配置流转

```
YAML 文件
  → AppConfig (全局单例)
  → PhyDriverConfig (结构体, 传入 l1_init)
  → PhyDriverCtx 成员变量
  → Per-cell CellConfig (传入 l1_cell_create)
  → cuPHY pipeline 参数
```

关键配置项:
- `gpu_id` — GPU 设备号
- `mps.enabled` / `mps.sm_partition` — MPS/GreenContext 分区
- `cells[]` — 多 cell 配置 (频点、带宽、天线数等)
- `fh.*` — Fronthaul 网口、VLAN、RU 地址
- `dpdk.*` — DPDK EAL 参数、hugepage
- `nvipc.*` — L2 通信模式 (SHM/DPDK/GPU)
- `threads.cpu_affinity.*` — 各线程绑核

---

## 6. 全部线程清单

> 线程名 = `pthread_setname_np` 设置的名称（Linux `ps -T` / `top -H` 可见），最长 15 字符。
> 
cuPHYDriver（核心模块）—— 专用 Worker 线程，非线程池
每个 Worker 对象持有且拥有一个专属 pthread，线程在系统初始化时创建，持续运行直至销毁。
// worker.cpp: Worker::run()
ret = pthread_create(&wid, NULL, worker_init, (void*)(&wargs));
// 设置 CPU 亲和性
ret = pthread_setaffinity_np(wid, sizeof(wcpuset), &wcpuset);
// 设置实时调度优先级
pthread_setschedparam(wid, SCHED_FIFO, &schedprm);

Worker 分为三种固定类型，分别注册在 PhyDriverCtx 的不同 map 中：

|类型	          |map 字段	             |配置来源
|WORKER_UL     |	worker_ul_map	      |ul_worker_threads: [2,3]（YAML）
|WORKER_DL	   |worker_dl_map	        |dl_worker_threads: [4,5,6]（YAML）
|WORKER_GENERIC|	worker_generic_map	|l1_worker_start_generic() 动态注册

每个 Worker 绑定到固定 CPU 核心，不共享，不复用。

### 6.1 cuphycontroller_scf — 主控进程

| 线程名 | 创建位置 | 职责 | 调度 / 绑核 |
|--------|----------|------|------------|
| `phy_init` | `cuphycontroller_scf.cpp:91` | 进程入口初始化阶段标识 | — |
| `phy_drv_init` | `cuphycontroller_scf.cpp:251` | PHY 驱动初始化阶段标识 | — |
| `phy_drv_proxy` | `cuphycontroller_scf.cpp:473` | PHYDriverProxy 启动阶段标识 | — |
| `phy_main` | `cuphycontroller_scf.cpp:556` | 主事件循环 | `low_priority_core` 可配置 |
| *(nvlog bg)* | `nvlog_fmtlog_init()` 返回 | fmtlog 异步日志后台刷写 | — |

### 6.2 cuphydriver — PHY 驱动核心线程 (PhyDriverCtx)

| 线程名 | 创建位置 | 职责 | 调度 / 绑核 |
|--------|----------|------|------------|
| `UlPhyDriver%02d` × N | `context.cpp` — `worker_ul_map` | **UL 实时处理 Worker**：PUSCH/PUCCH/PRACH/SRS 解码，每核一个 | `SCHED_FIFO`，`ul_cores[]` 隔离核 |
| `DlPhyDriver%02d` × N | `context.cpp` — `worker_dl_map` | **DL 实时处理 Worker**：PDSCH 编码 + FH 发送，每核一个 | `SCHED_FIFO`，`dl_cores[]` 隔离核 |
| `DlValPhyDrv%02d` × N | `context.cpp` — `worker_dl_validation_map` | DL 输出验证 Worker（调试 / CI 用） | `SCHED_FIFO`，`dl_validation_cores[]` |
| `DebugWorker%02d` | `context.cpp` — `addGenericWorker()` | 通用调试 Worker（可选，`debug_worker` 配置） | `SCHED_FIFO`，`debug_worker` 核 |
| `h2dcpy_thread` | `context.cpp:996` | **H2D 预拷贝线程**：提前将 TB 从 Host 搬到 GPU，降低 DL 关键路径延迟（可选） | 可配置核 + 优先级 |
| `datalake_thread` | `context.cpp:367` | 等待 Data Lake 数据就绪并转发 | 可配置核 |
| `ulpcap_thread` | `ul_pcap_capture_thread.cpp:129` | UL IQ 数据 PCAP 捕获（调试用，可选） | 可配置核 + 优先级 |
| `deletePrachObj` | `context.cpp:2570` | 异步释放 PRACH 对象（一次性临时线程） | — |
| `PtpMonitoring` | `cuphydriver_api.cpp:2247` | PTP 时间戳监控 | — |
| `RhocpPtpMon` | `cuphydriver_api.cpp:2334` | Rhocp PTP 事件监控 | — |
| `cell_update_cfg` | `cuphydriver_api.cpp:2401` | Cell 参数热更新（一次性临时线程） | — |
| `queue_log` | `task_function_dl_aggr.cpp:1406` | DL 聚合任务队列日志异步记录 | — |
| `pcap_log_thread` | `pcap_logger.cpp:122` | PCAP 日志文件写入（异步消费环形缓冲区） | 可配置核 + 优先级 |

### 6.3 aerial-fh-driver — Fronthaul 收发线程 (fh_gen::Worker)

| 线程名 / 函数 | 端点 | 职责 | 调度 / 绑核 |
|--------------|------|------|------------|
| `fronthaul_generator_dl_tx_worker` | DU 侧 `DL_TX_U` | DL eCPRI U-Plane 帧封装 + 以太网发送 | `SCHED_FIFO`，可配置核 |
| `fronthaul_generator_dl_tx_c_worker` | DU 侧 `DL_TX_C` | DL eCPRI C-Plane 控制帧发送 | `SCHED_FIFO`，可配置核 |
| `fronthaul_generator_ul_rx_worker` | DU 侧 `UL_RX` | UL eCPRI 帧接收 + 解封装 → GPU buffer | `SCHED_FIFO`，可配置核 |
| `fronthaul_generator_dl_rx_worker` | RU 侧 `DL_RX` | RU 模拟：接收 DL 帧 | `SCHED_FIFO`，可配置核 |
| `fronthaul_generator_ul_tx_worker` | RU 侧 `UL_TX` | RU 模拟：发送 UL IQ 帧 | `SCHED_FIFO`，可配置核 |
| `du_sync` | DU 侧 | DU SFN/Slot 时间同步命令处理（gRPC OAM） | 低优先级核 |
| `ru_sync` | RU 侧 | RU SFN/Slot 时间同步命令处理（gRPC OAM） | 低优先级核 |
| `fh_stats` | DU/RU | FH 统计轮询 + 周期打印 | — |
| `pcap_client_worker` | DU 侧 | pdump PCAP 客户端，发送 IQ 数据到外部抓包工具 | — |

### 6.4 cuphyl2adapter — L2 适配层线程

| 线程名 | 创建位置 | 职责 | 备注 |
|--------|----------|------|------|
| `msg_processing` | `nv_phy_module.cpp:497` — `PHY_module::thread_func` | **FAPI P5/P7 消息处理主线程**：从 nvIPC 读取消息，分发到各处理器 | 每 PHY_module 一个 |
| `oam_cell_update` | `nv_phy_module.cpp:807` | OAM 触发的 Cell 配置更新线程 | — |
| `sfn_slot_sync_cmd` | `nv_phy_module.cpp:852` | SFN/Slot 同步命令处理线程 | — |
| `timer_thread` | `nv_tick_generator.cpp:41` — `tti_gen` | **Slot Tick 生成线程**：产生 SLOT.indication，驱动整个调度循环（3 种模式：poll / sleep / timerfd） | — |
| *(mac thread)* | `nv_mac.cpp:46` — `mac::thread_func` | MAC 内部消息循环 | — |
| *(uephy thread)* | `nv_ue_phy_module.cpp:30` — `UEPHY_Module::thread_func` | UE PHY 模拟模块消息处理 | 仅 UE 侧模拟场景 |
| `AerialCUSConnMgr` | `cus_conn_mgr.cpp:371` | CU-S 连接管理（O-RAN CUS 接口） | — |

### 6.5 testMAC — L2 模拟器线程

| 线程名 | 创建位置 | 职责 | 调度 / 绑核 |
|--------|----------|------|------------|
| `mac_main` | `main.cpp:356` | testMAC 主循环 | 可配置 |
| *(mac_recv)* | `test_mac.cpp:323` — `mac_recv_thread_func` | **nvIPC 接收主线程**：接收 SLOT.indication / UL indication，驱动调度 | `recv_thread` 配置项 |
| *(scheduler)* | `test_mac.cpp:318` — `scheduler_thread_func` | **调度线程**：根据 launch_pattern 构建 DL/UL TTI 请求 | `sched_thread` 配置项 |
| *(builder)* | `test_mac.cpp:228` — `builder_thread_func` | FAPI 消息预构建线程（可选，`builder_thread_enable`） | `builder_thread` 配置项 |
| *(worker)* × N | `test_mac.cpp:246` — `worker_thread_func` | 多核并行 Worker（可选，`worker_cores[]`） | 可配置核 |
| `oam_thread` | `test_mac.cpp:235` | OAM 命令轮询线程：处理 cell start/stop/reconfig | 100 ms 轮询 |

### 6.6 cuMAC-CP — cuMAC 处理线程

| 线程名 | 创建位置 | 职责 |
|--------|----------|------|
| *(cumac_recv)* | `cumac_handler.cpp:359` — `cumac_receiver_thread_func` | cuMAC IPC 接收线程，内部再创建 tick 线程 |
| *(cumac_sched)* | `cumac_handler.cpp:364` — `cumac_scheduler_thread_func` | cuMAC 调度线程 |
| *(cumac_builder)* | `cumac_handler.cpp:382` — `cumac_builder_thread_func` | cuMAC 消息构建线程 |
| *(cumac_worker)* × N | `cumac_handler.cpp:374` — `cumac_worker_thread_func` | cuMAC 并行 Worker 线程 |
| *(cumac_tick)* | `cumac_handler.cpp:1237` — `cumac_tick_thread_func` | cuMAC Slot Tick 线程 |
| *(parsing)* × N | `cumac_pattern.cpp:764` / `launch_pattern.cpp:2214` | launch_pattern 文件并行解析线程 |

### 6.7 data_lake / E3Agent — 数据湖 & O1/E2 接口线程

| 线程名 | 创建位置 | 职责 |
|--------|----------|------|
| `datalake_task%d` × N | `data_lake.cpp:54` | Data Lake 任务处理线程（多实例） |
| *(e3_data)* | `e3_agent.cpp:98` — `E3Agent::dataServerThread` | E3 数据服务线程：处理 dApp 数据请求 |
| *(e3_reaper)* | `e3_agent.cpp:101` — `E3Agent::reaperThread` | E3 清理线程：超时 / 断开 dApp 订阅清理 |
| *(e3_sub)* | `e3_agent.cpp:104` — `E3Agent::managerSubscriptionThread` | E3 Manager 订阅管理线程：接收订阅 / 取消请求 |

### 6.8 ru-emulator — RU 模拟器线程

| 线程名 | 创建位置 | 职责 |
|--------|----------|------|
| `ru_oam_func` | `ru_emulator.cpp:66` | RU OAM 功能线程：处理 OAM 命令 |
| *(standalone_core)* | `ru_emulator.cpp:547` — `standalone_core_wrapper` | RU 独立运行核心处理线程 |

### 6.9 线程汇总统计

| 组件 | 固定线程 | 可扩展线程 | 典型总数 |
|------|---------|-----------|---------|
| cuphycontroller_scf | 4+1(nvlog) | — | ~5 |
| cuphydriver Workers | — | UL×N + DL×N + Val×N | 4~20（N=2~8）|
| cuphydriver 辅助 | 5 固定 | h2dcpy/ulpcap 可选 | 5~7 |
| aerial-fh-driver | 3(DU)+2(RU) | sync/stats | 5~9 |
| cuphyl2adapter | 3~4 固定 | timer per cell | ~5 |
| testMAC | 3 固定 | worker×N | 5~10 |
| cuMAC-CP | 4~5 固定 | worker×N + parsing×N | 5~8 |
| data_lake / E3 | 3 固定 | task×N | ~5 |
| ru-emulator | 2 | — | 2 |
| **合计（典型单 cell）** | | | **~40~70** |

### 6.10 线程间通信与同步

```
Slot 驱动链:
  timer_thread (tti_gen)
    → sem / nvIPC
      → msg_processing (PHY_module)
          → fapi_handler::slot_indication_handler
              → sem_post(slot_sem) → builder_thread (可选)
              → static_ul_dl_scheduler → nvIPC → PHY

DL 关键路径 (slot N, 截止时间 <1ms):
  DlPhyDriver%02d
    ← slot_indication
    → h2dcpy_thread: Host TB → GPU (提前搬运, 可选)
    → PhyPdschAggr::enqueue() → CUDA kernel (DL stream, priority -5)
    → fronthaul_generator_dl_tx_worker: eCPRI 帧 → RU

UL 关键路径 (slot N+k):
  fronthaul_generator_ul_rx_worker: eCPRI 帧 ← RU
    → GPU DMA (GDRCopy / DOCA 零拷贝)
    → UlPhyDriver%02d
        → PhyPuschAggr::decode() (UL stream, priority -3)
        → CRC.indication / RX_DATA.indication            → nvIPC → msg_processing → L2
```

### 6.11 线程间消息交互时序图

#### 6.11.1 完整 Slot 生命周期时序（DL + UL）

```
时间轴 →   T=0          T+50µs       T+100µs      T+300µs      T+500µs      T+1ms(下一slot)
           │            │            │            │            │            │

[timer_thread]
(tti_gen)  ├─timerfd/──►│            │            │            │            ├─timerfd/──►
           │  sleep到期  │            │            │            │            │  sleep到期

           │SLOT.indication(nvIPC)    │            │            │            │
           │────────────►│            │            │            │            │

[msg_processing]         │            │            │            │            │
(PHY_module)             ├─解析FAPI──►│            │            │            │
                         │ P7消息     │            │            │            │

                         │sem_post(slot_sem)        │            │            │
                         │────────────────────────►│            │            │

[scheduler_thread]                                 │            │            │
(testMAC)                                          ├─构建──────►│            │
                                                   │ DL_TTI.req │            │
                                                   │ TX_DATA.req│            │
                                                   │ UL_TTI.req │            │

                                                   │DL/UL_TTI.request(nvIPC) │
                                                   │────────────────────────►│

[msg_processing]                                                 │            │
(PHY_module)                                                     ├─分发──────►│
                                                                 │ DL任务入队 │
                                                                 │ UL窗口配置 │

                                                                 │enqueue(DlTask)
                                                                 │────────────►

[DlPhyDriver]                                                                │
(cuphydriver)                                                                ├─PhyPdschAggr
                                                                             │ ::setup()
                                                                             │ ::run()
                                                                             │ CUDA kernels
                                                                             │ (DL stream
                                                                             │  priority -5)
                                                                             │────────────►

[h2dcpy_thread]  ←──────────────────────────────────────────────────────────│
(可选)            提前 Host TB → GPU cudaMemcpyAsync                         │

[fh_dl_tx_worker]                                                            │
(aerial-fh-drv)                                                             ◄├─FH TX触发
                 eCPRI U-Plane帧 → RU (以太网)                               │
```

---

#### 6.11.2 UL 接收处理时序（slot N+k）

```
时间轴 →   T=0(UL slot开始)   T+100µs      T+400µs      T+500µs      T+700µs

[fh_ul_rx_worker]
(aerial-fh-drv)  ├─以太网帧接收──►│            │            │            │
                 │ eCPRI解封装    │            │            │            │
                 │ GDRCopy→GPU buf│            │            │            │
                 │                │            │            │            │
                 │Order Kernel启动(cudaLaunchKernel)         │            │
                 │────────────────────────────►│            │            │
                 │                             │            │            │
                 │orderLaunched信号(sem_post)  │            │            │
                 │────────────────────────────────────────►│            │

[UlPhyDriver]                                               │            │
(cuphydriver)                                               ├─waitULC────►│
                                                            │ TasksComplete│
                                                            │            │
                                                            │PUSCH setup Phase1
                                                            │ cuphySetupPuschRx()
                                                            │ 查询HARQ buf尺寸
                                                            │────────────►│
                                                                          │
                                                            │HARQ buf分配 │
                                                            │ hb_pool->   │
                                                            │ bucketAlloc │
                                                            │────────────►│
                                                                          │
                                                            │PUSCH setup Phase2
                                                            │ cuphySetupPuschRx()
                                                            │ 写GPU descriptor
                                                            │────────────►│
                                                                          │
                                                            │waitOrderLaunched()
                                                            │(等Order Kernel入队)
                                                            │            │
                                                            │PUSCH RUN   │
                                                            │ subSlotProc│
                                                            │ (phase1_stream
                                                            │  priority -3)
                                                            │ ChEst/ChEq │
                                                            │ EarlyHARQ  │
                                                            │────────────►│
                                                                          │
                                                     GPU完成early kernels │
                                                     subSlotCompletedEvent↑
                                                                          │
                                                            │PUSCH RUN   │
                                                            │ fullSlotProc│
                                                            │ LDPC/CRC   │
                                                            │────────────►│
                                                                          │
                                                            │phase1Complete↑
                                                            │            │
                                                            │PUSCH RUN   │
                                                            │ D2H copy   │
                                                            │(phase2_stream)
                                                            │ waitEvent  │
                                                            │ cudaMemcpyAsync
                                                            │────────────►│

[UlPhyDriver]
 task_ul_aggr_2                                                          ◄├─D2H完成
                                                                          │
                                                            │pusch->callback()
                                                            │────────────►│

[msg_processing]◄──────────────────────────────────────────────────────────│
(PHY_module)     CRC.indication + RX_DATA.indication (nvIPC)              │
                 → L2 (testMAC / 外部 L2)
```

---

#### 6.11.3 多线程并发交互总览时序图

```
组件/线程          消息 / 事件
─────────────────────────────────────────────────────────────────────────────────
                  ┌──────────────┐   ┌─────────────┐   ┌──────────────────────┐
                  │  testMAC     │   │cuphyl2adapter│   │   cuphydriver        │
                  │              │   │             │   │                      │
L2层              │mac_recv_thrd │   │msg_processing│   │ DlPhyDriver          │
                  │scheduler_thrd│   │timer_thread  │   │ UlPhyDriver          │
                  │builder_thread│   │             │   │ h2dcpy_thread        │
                  │oam_thread    │   │             │   │ datalake_thread      │
                  └──────┬───────┘   └──────┬──────┘   └──────────┬───────────┘
                         │                  │                      │
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 初始化阶段:            │                  │                      │
  oam_thread ──────────►│ cell_start cmd   │                      │
                         │──PARAM.req──────►│                      │
                         │  CONFIG.req      │──l1_cell_create()───►│
                         │  START.req       │──l1_cell_start()────►│ FH启动
                         │◄─responses───────│                      │ slot timer启动
                         │                  │                      │
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 每 Slot 调度循环:       │                  │                      │
                         │                  │◄──SLOT.indication────│ timer_thread
                         │                  │   (nvIPC)            │  触发
                         │◄─SLOT.indication─│                      │
                         │  (nvIPC)         │                      │
  mac_recv_thrd 收到     │                  │                      │
  ──sem_post──►scheduler │                  │                      │
  scheduler构建 DL/UL 请求                  │                      │
                         │──DL_TTI.request─►│                      │
                         │  TX_DATA.request │──enqueue(DL task)───►│ DlPhyDriver
                         │──UL_TTI.request─►│                      │  PDSCH encode
                         │                  │──schedule UL window─►│  + FH TX
                         │                  │                      │
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 UL 结果返回:            │                  │                      │
                         │                  │◄─CRC.indication──────│ UlPhyDriver
                         │                  │  RX_DATA.indication  │  callback
                         │                  │  (nvIPC)             │
                         │◄─CRC.indication──│                      │
                         │  RX_DATA.ind     │                      │
  mac_recv_thrd 处理     │                  │                      │
  UL结果/ACK/NACK        │                  │                      │
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

                  ┌──────────────────────┐   ┌──────────────────────────────────┐
                  │  aerial-fh-driver    │   │  cuphydriver (GPU/CUDA)          │
FH层              │  fh_dl_tx_worker     │   │  DL stream (priority -5)         │
                  │  fh_dl_c_worker      │   │  UL stream (priority -3)         │
                  │  fh_ul_rx_worker     │   │  pucch_stream                    │
                  └──────────┬───────────┘   └──────────────┬───────────────────┘
                             │                              │
  DlPhyDriver触发FH发送:     │◄─FhProxy::send()─────────────│ DL CUDA kernels完成
                             │  传递GPU buffer地址           │ (eCPRI帧封装)
                             │ eCPRI U-Plane帧 → RU         │
                             │                              │
  FH接收UL IQ:               │ eCPRI U-Plane帧 ← RU         │
                             │──GDRCopy/DOCA──────────────►│ GPU buffer
                             │  零拷贝DMA                    │
                             │──orderLaunched(sem)─────────►│ Order Kernel
                             │                              │  IQ重排
                             │                              │──waitOrderLaunched
                             │                              │  PUSCH setup/run
```

---

#### 6.11.4 同步原语汇总

| 同步原语 | 发送方 | 接收方 | 传递内容 |
|---------|-------|-------|---------|
| `sem_post(slot_sem)` | `msg_processing` | `scheduler_thread` | Slot indication 触发信号 |
| `nvIPC ring (SHM/DPDK)` | `timer_thread` | `mac_recv_thrd` | FAPI SLOT.indication 消息 |
| `nvIPC ring` | `scheduler_thread` | `msg_processing` | DL/UL_TTI.request 消息 |
| `nvIPC ring` | `UlPhyDriver callback` | `mac_recv_thrd` | CRC/RX_DATA.indication |
| `sem_post(orderLaunched)` | `fh_ul_rx_worker` | `UlPhyDriver` | Order Kernel 已入队通知 |
| `waitULCTasksComplete()` | `fh_ul_rx_worker` | `UlPhyDriver` | ULC (FH→GPU DMA) 完成同步 |
| `cudaEventRecord(subSlotCompletedEvent)` | `phase1_stream` | CPU 轮询 | Early-HARQ GPU kernel 完成 |
| `cudaEventRecord(phase1Complete)` | `phase1_stream` | `phase2_stream` | Phase1 全部 kernel 完成 |
| `cudaStreamWaitEvent(phase2_stream, phase1Complete)` | GPU 驱动 | `phase2_stream` | D2H 拷贝解锁 |
| `FhProxy::notifyTxReady()` | `DlPhyDriver` | `fh_dl_tx_worker` | DL GPU buffer 就绪，可封帧发送 |
| `DataLake::notify()` | `UlPhyDriver callback` | `datalake_thread` | UL 结果数据入湖通知 |
| `gRPC (OAM)` | `oam_thread` | `msg_processing` | Cell start/stop/reconfig 命令 |

---

## 7. PUSCH 解调完整调用关系分析

### 7.1 PUSCH 端到端处理总览

```
RU (天线)
  → eCPRI U-Plane 帧 (IQ 数据, 14 OFDM 符号/slot)
  → fronthaul_generator_ul_rx_worker (DU 侧 FH 接收线程)
  → GPU Buffer (GDRCopy 零拷贝 / DOCA DMA)
  → Order Kernel (UL 排序核，按 PRB 重排)
  → PhyPuschAggr::setup() × 2 phases
  → PhyPuschAggr::run()   × 1~3 phases
  → cuphyRunPuschRx() → 一批 CUDA kernel
  → D2H 拷贝 (pinned host buffer)
  → PhyPuschAggr::callback()
  → ul_cb.callback_fn() → L2 (CRC.indication + RX_DATA.indication)
```

---

### 7.2 调用链详解（从 UL Worker 到 cuPHY kernel）

```
UlPhyDriver%02d (Worker 线程, SCHED_FIFO, UL stream priority -3)
  │
  └─ task_work_function_ul_aggr_1_pucch_pusch()
       │                    [task_function_ul_aggr.cpp]
       │
       ├─ [等待 ULC 任务完成] slot_map->waitULCTasksComplete()
       │    └─ ULC = UL Copy 任务 (FH 数据搬运到 GPU)
       │
       ├─ [选流] 根据 TDD slot 编号选择 phase1/phase2/pucch CUDA stream
       │    ├─ phase1_stream: pusch_streams[PHASE1_SPLIT_STREAM1/2]
       │    └─ phase2_stream: pusch_streams[PHASE2_SPLIT_STREAM1/2]
       │
       ├─ ① PUCCH Setup (并行, pucch_stream)
       │    └─ pucch->setup(aggr_cell_list, aggr_ulbuf_st1, pucch_stream)
       │
       ├─ ② PUSCH Setup (phase1_stream + phase2_stream)
       │    └─ pusch->setup(aggr_cell_list, aggr_ulbuf_st1,
       │                    phase1_stream, phase2_stream)
       │         [phypusch_aggr.cpp]
       │         │
       │         ├─ 填充 dyn_params.pCellGrpDynPrm (UE 数量/RNTI/HARQ PID...)
       │         ├─ 设置 DataIn.pTDataRx[].pAddr = ulbuf_st1->getBufD() (GPU ptr)
       │         │
       │         ├─ [PHASE 1] dyn_params.setupPhase = PUSCH_SETUP_PHASE_1
       │         │    └─ cuphySetupPuschRx(puschRxHndl, &dyn_params, batchPrmHndl)
       │         │         → 查询每个 UE TB 所需 HARQ buffer 大小
       │         │         → 输出: DataOut.h_harqBufferSizeInBytes[]
       │         │
       │         ├─ [HARQ buffer 分配] 按 Phase 1 返回的尺寸
       │         │    └─ hb_pool_m->bucketAllocateBuffer() × nUes
       │         │         → 从 HARQ 池分配 pinned GPU 内存
       │         │         → DataInOut.pHarqBuffersInOut[k] = hb_ptr->getAddr()
       │         │
       │         └─ [PHASE 2] dyn_params.setupPhase = PUSCH_SETUP_PHASE_2
       │              └─ cuphySetupPuschRx(puschRxHndl, &dyn_params, batchPrmHndl)
       │                   → 将 UE 参数/HARQ buffer 地址写入 GPU descriptor
       │                   → 准备 CUDA kernel launch 配置
       │
       ├─ [等待 Order Kernel 启动] oentity->waitOrderLaunched()
       │    └─ Order Kernel: 重排 FH IQ 数据 → 按信道/UE 顺序排列
       │
       ├─ ③ PUSCH RUN 1 (早期 HARQ 模式 / 前置 DMRS 模式)
       │    └─ pusch->run(PUSCH_RUN_SUB_SLOT_PROC)  ← sub-slot 4 符号
       │         └─ cuphyRunPuschRx(puschRxHndl, PUSCH_RUN_SUB_SLOT_PROC)
       │              └─ PuschRx::run()  [pusch_rx.cpp]
       │                   └─ subSlotKernelLaunch()  ← 非 Graph 模式
       │                        (见 §7.3)
       │
       ├─ ④ PUCCH RUN (pucch_stream, 可与 PUSCH 并行)
       │
       └─ ⑤ PUSCH RUN 2 (完整 slot)
            ├─ [有 EH/FL-DMRS] pusch->run(PUSCH_RUN_FULL_SLOT_PROC)
            │    └─ cuphyRunPuschRx → PuschRx::run → fullSlotKernelLaunch()
            │         (见 §7.4)
            ├─ [有 EH/FL-DMRS] pusch->run(PUSCH_RUN_FULL_SLOT_COPY)
            │    └─ phase2Stream 等待 phase1 完成事件 → copyOutputToCPU()
            │         → cudaMemcpyAsync(GPU→pinned_host, phase2_stream)
            └─ [无 EH/FL-DMRS] pusch->run(PUSCH_RUN_ALL_PHASES)
                 └─ subSlotKernelLaunch + fullSlotKernelLaunch + copyOutputToCPU
```

---

### 7.3 subSlotKernelLaunch — 早期 HARQ (前 4 符号) CUDA kernel 序列

```
subSlotKernelLaunch()  [pusch_rx.cpp:5447]
  (在 phase1_stream 上，或借助 G1/G2 streamPool 并行)
  │
  ├─ m_chest->chestStream().launchKernels0Slot(phase1_stream)
  │    └─ 信道估计 kernel (DMRS 符号 → 信道响应 Ĥ)
  │         cuphyPuschRxChEst (chest_factory 创建)
  │
  ├─ noiseIntfEst kernel (launch_kernel, 按 hetCfg 遍历)
  │    └─ 噪声/干扰方差估计 (MMSE 均衡需要)
  │
  ├─ chEqCoefComp kernel  ← 均衡系数计算 W = Ĥ^H / (|Ĥ|² + σ²)
  │
  ├─ chEqSoftDemap kernel ← 均衡 + 软解映射
  │    └─ LLR 生成 (每个比特的对数似然比)
  │    [若 DFT-s-OFDM: + IDFT kernel + SoftDemapAfterDft kernel]
  │
  ├─ [G1/G2 streamPool fork — 多流并行 UCI 解码]
  │    ├─ G1 流: uciOnPuschEarlySegLLRs0 kernel (UCI LLR 提取)
  │    ├─ G2 流1: simplexDecoder_early kernel   (HARQ-ACK simplex)
  │    ├─ G2 流2: rmDecoder_early kernel        (Rate-match 解码, RM)
  │    └─ G2 流3: compCwTreeTypes → polSegDeRmDeItl → polarDecoder
  │         └─ Polar 编码 UCI 解码 (CSI Part1/HARQ)
  │
  ├─ [可选] rssi kernel  (RSSI 测量)
  ├─ [可选] rsrp kernel  (RSRP/SINR 测量)
  │
  └─ G1/G2 streamPool join(phase1_stream)
       └─ cudaEventRecord(subSlotCompletedEvent, phase1_stream)
            → 通知 cuPHY-CP 早期 HARQ 处理完成
```

---

### 7.4 fullSlotKernelLaunch — 全 slot (14 符号) CUDA kernel 序列

```
fullSlotKernelLaunch()  [pusch_rx.cpp:5648]
  (在 phase1_stream 上)
  │
  ├─ m_chest->chestStream().launchKernels(phase1_stream)
  │    └─ 全 slot 信道估计 kernel
  │         时频二维插值: DMRS → 全 PRB × 全符号 Ĥ(f,t)
  │
  ├─ noiseIntfEst kernel (全 slot 噪声方差)
  │
  ├─ [若 enableWeightedAverageCfo] rsrp kernel (预先测量用于 CFO 加权)
  │
  ├─ cfoTaEst kernel — CFO/TA 估计
  │    └─ 利用 DMRS 相位旋转估计载波频偏 & 时延
  │
  ├─ chEqCoefComp kernel × CUPHY_PUSCH_RX_MAX_N_TIME_CH_EQ
  │    └─ MMSE 均衡系数 (时变信道多次计算)
  │
  ├─ chEqSoftDemap kernel — 均衡 + 软解映射 → LLR
  │    [若 DFT-s-OFDM: + IDFT + SoftDemapAfterDft]
  │
  ├─ [G1/G2 streamPool fork — UCI 并行]
  │    ├─ G1 流: uciOnPuschSegLLRs0 kernel (HARQ-ACK/CSI1 LLR)
  │    ├─ G2 流1: simplexDecoder kernel  (单工 UCI 解码)
  │    ├─ G2 流2: rmDecoder kernel       (RM 解码)
  │    └─ G2 流3: compCwTreeTypes → polSegDeRmDeItl → polarDecoder
  │
  ├─ [若有 CSI Part2] uciOnPuschCsi2Ctrl + SegLLRs2 → rmDecoder_csi2
  │    → simplexDecoder_csi2 → compCwTreeTypes_csi2
  │    → polSegDeRmDeItl_csi2 → polarDecoder_csi2
  │
  ├─ [SCH 数据 UE (TB 传输)]
  │    ├─ de_rate_matching_global2 (reset + main + clamp) 3 个 kernel
  │    │    └─ LDPC 码率匹配逆处理 (软比特解速率匹配)
  │    │
  │    ├─ LDPC Decoder kernel × nCBs
  │    │    ├─ 模式1 (DRIVER_LDPC_LAUNCH): launch_kernel × DESC_COUNT
  │    │    ├─ 模式2 (SEQUENTIAL): launchLDPCStreamsTensor()
  │    │    └─ 模式3 (STREAM_POOL/SINGLE_STREAM): launchLDPCStreamsTB()
  │    │    → 迭代 LDPC 解码 (最大 ldpcMaxNumItrs 次, 早终止)
  │    │
  │    └─ CRC kernel × 2
  │         ├─ CB-level CRC 校验 (24B)
  │         └─ TB-level CRC 校验 (24A)
  │              → 输出 pTbCrcs[]/pCbCrcs[] (0=pass, 非0=fail)
  │
  ├─ rssi kernel (RSSI 测量, 可选)
  └─ rsrp kernel (RSRP/SINR 测量, 可选)
```

---

### 7.5 D2H 拷贝与结果回调

```
copyOutputToCPU(phase2_stream)
  [PuschRx::run, PUSCH_RUN_FULL_SLOT_COPY 或 ALL_PHASES]
  │
  ├─ cudaStreamWaitEvent(phase2Stream, m_phase1Complete, 0)
  │    └─ 等待 phase1 全部 kernel 完成
  │
  └─ cudaMemcpyAsync(host_pinned ← GPU, phase2_stream) × 多次
       ├─ pTbPayloads  (解码后的 TB payload 字节)
       ├─ pTbCrcs      (TB CRC 校验结果)
       ├─ pCbCrcs      (CB CRC 校验结果)
       ├─ pTaEsts      (TA 估计值, 用于 TAC 上报)
       ├─ pRsrp/pSinr  (信号质量测量)
       └─ pUciPayloads (UCI: HARQ-ACK/CSI 解码结果)

↓ (CPU 等待 phase2_stream 完成)

task_work_function_ul_aggr_2_result()
  [task_function_ul_aggr.cpp:2504]
  │
  └─ pusch->callback(cell_timeout_list, gpu_early_harq_timeout)
       [phypusch_aggr.cpp:1352]
       │
       ├─ pdctx->getDataLake()->notify(...)   // Data Lake 异步通知
       │
       ├─ ul_cb.callback_fn(context, nTbCrcErrors, msg, *si, *pusch,
       │                    &DataOut, &static_params)
       │    └─ → scf_5g_fapi → nvIPC → L2
       │         ├─ CRC.indication   (每 UE 一条: TB CRC pass/fail + TA)
       │         └─ RX_DATA.indication (TB payload 数据)
       │
       └─ HARQ buffer 回收
            ├─ CRC==0 (成功): bucketReleaseBuffer() 立即释放
            └─ CRC!=0 (失败): 保留 HARQ buffer (软合并 Chase/IR)
                 → 下次重传时 pHarqBuffersInOut 指向同一 buffer
                 → LDPC 解码器累加 LLR (增量冗余合并)
```

---

### 7.6 CUDA 内存布局与 buffer 对应关系

| Buffer | 类型 | 分配方式 | 说明 |
|--------|------|---------|------|
| `DataIn.pTDataRx[cell_idx].pAddr` | GPU Global | GDRCopy 映射 / DOCA DMA | FH 接收 IQ 数据，直接指向 FH buffer |
| HARQ buffer (`hb_ptr->getAddr()`) | GPU Pinned (GDR) | `hb_pool_m->bucketAllocate` | LDPC 软比特中间态，跨 slot 保留 |
| `DataOut.pTbPayloads` | Host Pinned | `cuphy::buffer<uint8_t, pinned_alloc>` | D2H 目标，解码后 TB 字节 |
| `DataOut.pTbCrcs` | Host Pinned | `cuphy::buffer<uint32_t, pinned_alloc>` | CRC 结果 (0=pass) |
| `DataOut.pUciPayloads` | Host Pinned | `cuphy::buffer<uint8_t, pinned_alloc>` | UCI 内容 (HARQ-ACK / CSI) |
| `DataOut.pTaEsts` | Host Pinned | `cuphy::buffer<float, pinned_alloc>` | TA 估计 (ns) |
| `DataOut.pSinrPostEq` | Host Pinned | `cuphy::buffer<float, pinned_alloc>` | 后均衡 SINR (dB) |
| CUDA descriptor (静/动) | GPU Global | `cuphy::device_alloc` | kernel 参数描述符 (H2D 拷贝) |
| `sym_ord_done_sig_arr` | GDR Pinned | `gDev->newGDRbuf()` | Order Kernel 完成信号 (每符号一个) |

---

### 7.7 CUDA Stream / Event 协调机制

```
时序图 (单 slot, EH 模式)：

phase1_stream: ──[OrderKernel]──[ChEst0Slot]──[NoiseEst]──[ChEqCoef]──[SoftDemap]──
                                                           ──[EarlyHARQ kernels]──[subSlotCompletedEvent↑]──
                                                                                  ──[PostSubSlotWait]──[ChEst全slot]──
                                                                                                      ──[ChEqCoef]──[SoftDemap]──
                                                                                                      ──[DeRateMatch]──[LDPC]──[CRC]──[phase1Complete↑]──

phase2_stream: ──────────────────────────────────────────────────────────────────────────────────────────────────────────
               wait(phase1Complete) ──[cudaMemcpyAsync D2H]──

pucch_stream:  ──wait(OrderKernelDone) ──[PUCCH kernels]──

关键事件：
  waitCompletedSubSlotEvent  → 通知 CPU: sub-slot kernel 已入队
  subSlotCompletedEvent      → 通知 CPU: early-HARQ GPU 完成
  waitCompletedFullSlotEvent → 通知 CPU: full-slot kernel 已入队
  phase1Complete             → phase2_stream 开始 D2H 拷贝的触发点
```

---

### 7.8 cuPHY API 层级关系（从 cuphydriver 到底层 kernel）

```
cuphydriver (C++ 层)
  PhyPuschAggr::setup()
    └─ cuphySetupPuschRx()          ← cuphy.h C API
         └─ PuschRx::setup()        [pusch_rx.cpp]
              ├─ expandBackEndParameters()   // UE参数展开
              ├─ allocateDeviceMemory()      // GPU descriptor H2D
              └─ allocateDescr()            // 各子模块 descriptor 空间

  PhyPuschAggr::run()
    └─ cuphyRunPuschRx()            ← cuphy.h C API
         └─ PuschRx::run()          [pusch_rx.cpp]
              ├─ [Graph 模式]  cudaGraphLaunch(m_fullSlotGraphExec, stream)
              │                └─ CUDA Graph: 所有 kernel 依赖关系预编译
              └─ [Stream 模式] fullSlotKernelLaunch() / subSlotKernelLaunch()
                   ├─ launch_kernel(kernelNodeParamsDriver, stream)
                   │    └─ cuLaunchKernel() (CUDA Driver API)
                   │         各子模块 kernel:
                   │         ├─ cuphyPuschRxChEst      (信道估计)
                   │         ├─ cuphyPuschRxNoiseIntfEst (噪声估计)
                   │         ├─ cuphyPuschRxCfoTaEst   (CFO/TA 估计)
                   │         ├─ cuphyPuschRxChEq       (均衡+软解映射)
                   │         ├─ de_rate_matching_global2 (解速率匹配)
                   │         ├─ ldpc_decoder            (LDPC 迭代解码)
                   │         ├─ crc_decode              (CRC 校验)
                   │         ├─ rm_decoder              (RM 解码, UCI)
                   │         └─ polar_decoder           (Polar 解码, UCI)
                   └─ G1/G2 streamPool
                        └─ 多 CUDA stream 并行: UCI 与 SCH 数据并行解码
```

---

### 7.9 关键数据结构

| 结构体 | 定义文件 | 作用 |
|--------|---------|------|
| `cuphyPuschStatPrms_t` | `cuphy.h` | 静态参数：最大 PRB、天线数、LDPC 配置、stream 优先级 |
| `cuphyPuschDynPrms_t` | `cuphy.h` | 动态参数：每 slot 更新，包含 setupPhase、stream、procMode |
| `cuphyPuschCellGrpDynPrm_t` | `cuphy.h` | Cell 组动态参数：nCells、nUeGrps、nUes |
| `cuphyPuschUePrm_t` | `cuphy.h` | 单 UE 参数：RNTI、MCS、layers、HARQ PID、ndi、pduBitmap |
| `cuphyPuschDataOut_t` | `cuphy.h` | 输出指针集合：pTbPayloads、pTbCrcs、pUciPayloads 等 |
| `cuphyPuschDataInOut_t` | `cuphy.h` | HARQ buffer 指针数组（GPU 地址，跨 slot 保留） |
| `SlotMapUl` | `slot_map_ul.hpp` | UL slot 上下文：聚合所有 Aggr 对象 + FH buffer + 任务列表 |
| `HarqBuffer` | `harq_pool.hpp` | HARQ 软比特 buffer：GDR pinned，含 RNTI/PID/NDI/refCount |

---


## 8 Graph 总体架构与类型
### 8.1 Graph 总体架构与类型

PUSCH RX 在初始化时创建 **5 类 CUDA Graph**，均存储于 `PuschRx` 对象中：

| Graph 成员变量 | GraphExec | 用途 | 对应处理路径 |
|---------------|-----------|------|------------|
| `m_fullSlotGraph` | `m_fullSlotGraphExec` | Legacy full-slot 处理 (无前置 DMRS) | `PUSCH_LEGACY_FULL_SLOT_PROC` |
| `m_frontLoadedDmrsFullSlotGraph` | `m_frontLoadedDmrsFullSlotGraphExec` | 前置 DMRS full-slot 处理 | `PUSCH_FRONT_LOADED_DMRS_FULL_SLOT_PROC` |
| `m_ehqGraph` | `m_ehqGraphExec` | Early-HARQ 子图 (前 4 符号) | `CUPHY_PUSCH_SUB_SLOT_PATH` |
| `m_frontLoadedDmrsGraph` | `m_frontLoadedDmrsGraphExec` | 前置 DMRS 子图 | `CUPHY_PUSCH_FULL_SLOT_PATH` |
| `m_preSubSlotGraph` / `m_preFullSlotGraph` | `m_preSubSlotGraphExec` / `m_preFullSlotGraphExec` | **Launch Graph**：同步等待 + Event 记录 + Device Graph Launch 触发 | 两条路径各一个 |

**Work Cancel 模式**（控制 `condGraphInfo` 结构）：

| 枚举值 | 含义 | CUDA 最低版本 |
|--------|------|--------------|
| `PUSCH_NO_WORK_CANCEL` | 无条件执行，所有节点按使能位决定 | 任意 |
| `PUSCH_DEVICE_GRAPHS` | Device Graph Launch：主图通过 kernel node 触发子图 | CUDA 12.0+ |
| `PUSCH_COND_IF_NODES_W_KERNEL` | 条件节点 (Conditional IF) + 触发 kernel | CUDA 12.4+ |

---

### 8.2 Graph 初始化流程（`PuschRx` 构造阶段）

```
PuschRx::PuschRx(cuphyPuschStatPrms_t*, cuStream)  [pusch_rx.cpp:~line 950]
  │
  ├─ 1. 静态参数解析 & Work-Cancel 模式协商
  │    ├─ 读取 m_cuphyPuschStatPrms.workCancelMode
  │    ├─ 若 CUDA < 12.4 且请求 PUSCH_COND_IF_NODES_W_KERNEL
  │    │    → 降级为 PUSCH_DEVICE_GRAPHS
  │    └─ 若 CUDA < 12.0 且请求 PUSCH_DEVICE_GRAPHS
  │         → 降级为 PUSCH_NO_WORK_CANCEL
  │
  ├─ 2. 组件创建 createComponents(cuStream, ...)
  │    ├─ ChannelEstimateGraphMgr (ChEst/ChEq nodes 管理)
  │    ├─ LdpcDecodeDescSet (LDPC descriptor 集合)
  │    ├─ RateMatchDescSet
  │    └─ 各子模块 launch config 填充
  │         (kernelNodeParamsDriver: func/gridDim/blockDim/sharedMem/kernelParams)
  │
  ├─ 3. 静态参数 H2D 拷贝
  │    └─ cudaMemcpyAsync(m_puschCellStatPrmBufGpu ← m_puschCellStatPrmCpu, cuStream)
  │
  ├─ 4. Full-Slot Graph 建立 × 2 种处理模式
  │    ├─ createFullSlotGraph(PUSCH_LEGACY_FULL_SLOT_PROC,
  │    │                      m_fullSlotGraph, m_emptyFullSlotRootNode,
  │    │                      m_fullSlotGraphCondInfo)
  │    ├─ createFullSlotGraph(PUSCH_FRONT_LOADED_DMRS_FULL_SLOT_PROC,
  │    │                      m_frontLoadedDmrsFullSlotGraph, ...)
  │    ├─ cuGraphInstantiate(&m_fullSlotGraphExec, m_fullSlotGraph,
  │    │                     [CUDA_GRAPH_INSTANTIATE_FLAG_DEVICE_LAUNCH 若 DGL])
  │    ├─ cuGraphInstantiate(&m_frontLoadedDmrsFullSlotGraphExec, ...)
  │    ├─ updateFullSlotGraph(disableAllNodes=true, ...)   // 初始全部禁用
  │    └─ updateFullSlotGraph(disableAllNodes=true, ...)
  │
  ├─ 5. Early-HARQ / Front-Loaded DMRS Sub-Graph 建立
  │    ├─ createEarlyHarqGraph()          → m_ehqGraph
  │    ├─ createFrontLoadedDmrsGraph()    → m_frontLoadedDmrsGraph
  │    ├─ cuGraphInstantiate(&m_ehqGraphExec, m_ehqGraph, [...])
  │    ├─ cuGraphInstantiate(&m_frontLoadedDmrsGraphExec, ...)
  │    ├─ updateEarlyHarqGraph(true)      // 初始全部禁用
  │    └─ updateFrontLoadedDmrsGraph(true)
  │
  └─ 6. Launch Graph 建立 × 2 条路径
       ├─ createLaunchGraph(m_preSubSlotGraph, m_preSubSlotWaitNode,
       │                    m_preSubSlotEventNode, m_preSubSlotDglNode,
       │                    m_preSubSlotWaitCfgs, m_preSubSlotDglCfgs,
       │                    CUPHY_PUSCH_SUB_SLOT_PATH,
       │                    m_waitCompletedSubSlotEvent, m_deviceGraphLaunchEnabled)
       ├─ cuGraphInstantiate(&m_preSubSlotGraphExec, m_preSubSlotGraph, 0)
       ├─ createLaunchGraph(m_preFullSlotGraph, ..., CUPHY_PUSCH_FULL_SLOT_PATH, ...)
       └─ cuGraphInstantiate(&m_preFullSlotGraphExec, m_preFullSlotGraph, 0)
```

---

### 8.3 `createFullSlotGraph` 内部节点构建流程

```
PuschRx::createFullSlotGraph(fullSlotProcMode, &fullSlotGraph, &emptyRootNode, condInfo)
  [pusch_rx.cpp:3868]
  │
  ├─ cuGraphCreate(&fullSlotGraph, 0)
  │
  ├─ 依赖向量初始化 (std::vector<CUgraphNode>)
  │    ├─ rootParents      ← 图的入口依赖
  │    ├─ softDemapParents ← ChEst/NoiseEst 的出口
  │    ├─ uciP1Parents     ← SoftDemap 的出口
  │    ├─ csiP2Parents     ← UCI-P1 解码器的出口
  │    └─ schParents       ← SegLLRs2 的出口
  │
  ├─ [条件节点 Stage 0 — C0] enterConditionalStage0(...)
  │    ├─ 模式 PUSCH_COND_IF_NODES_W_KERNEL (CUDA >= 12.4):
  │    │    ├─ cuGraphConditionalHandleCreate(&m_conditional_node_C0_handle,
  │    │    │                                fullSlotGraph, ctx,
  │    │    │                                DEFAULT_COND_VAL, flags)
  │    │    ├─ cuGraphAddNode(&m_graph_G0_cond_node, ...)  // 条件 IF 节点
  │    │    └─ condStage.pGraph = &(条件节点内的子图)
  │    └─ 模式 PUSCH_DEVICE_GRAPHS:
  │         ├─ cuGraphCreate(&condInfo.m_graph[0], 0)     // 独立 C0 子图
  │         └─ condStage.pGraph = &condInfo.m_graph[0]
  │
  ├─ [前端 (FE) Stage] buildFrontEndStage(fullSlotProcMode, condStage.pGraph, rootParents)
  │    ├─ ChEst 节点  m_chest->chestStream().addKernelNodeToGraph(graph, deps, ...)
  │    │    → cuGraphAddKernelNode(&chEstNodes[], graph, parents, ...)
  │    ├─ NoiseIntfEst 节点 (遍历 hetCfg)
  │    │    → cuGraphAddKernelNode(&noiseIntfEstNodes[], ...)
  │    ├─ CfoTaEst 节点 (若启用)
  │    │    → cuGraphAddKernelNode(&cfoTaEstNode, ...)
  │    ├─ ChEqCoef 节点 × CUPHY_PUSCH_RX_MAX_N_TIME_CH_EQ
  │    │    → cuGraphAddKernelNode(&chEqCoefNodes[], ...)
  │    └─ SoftDemap 节点 (含可选 IDFT / SoftDemapAfterDft for DFT-s-OFDM)
  │         → cuGraphAddKernelNode(&softDemapNode, ...)
  │         输出: softDemapParents
  │
  ├─ [条件节点 Stage 1 — C1] enterConditionalStage1(condInfo, ctx, flags,
  │    softDemapParents, dglParentsC1)
  │    ├─ UCI-P1 (HARQ-ACK/CSI1) 分支独立子图 G1
  │    └─ 输出: condStage1 (指向 G1 子图或条件节点内 G1)
  │
  ├─ [UCI-P1 Backend Stage] buildUciP1BackendStage(fullSlotProcMode, condStage1.pGraph, ...)
  │    ├─ uciOnPuschSegLLRs0 节点  → cuGraphAddKernelNode(...)
  │    ├─ simplexDecoder 节点      → cuGraphAddKernelNode(...)
  │    ├─ rmDecoder 节点           → cuGraphAddKernelNode(...)
  │    ├─ compCwTreeTypes 节点     → cuGraphAddKernelNode(...)
  │    ├─ polSegDeRmDeItl 节点     → cuGraphAddKernelNode(...)
  │    └─ polarDecoder 节点        → cuGraphAddKernelNode(...)
  │         输出: uciP1Parents (UCI-P1 全部解码器的终止节点)
  │
  ├─ [条件节点 Stage 2 — C2] enterConditionalStage2(condInfo, ctx, flags,
  │    use_cond_if_node_c2, parentsForC2, dglParentsC2)
  │    └─ CSI-P2 + SCH 数据分支独立子图 G2
  │
  ├─ [CSI-P2 Backend Stage] buildCsiP2BackendStage(fullSlotProcMode, condStage2.pGraph, ...)
  │    ├─ uciOnPuschCsi2Ctrl 节点
  │    ├─ uciOnPuschCsi2SegLLRs2 节点
  │    ├─ rmDecoder_csi2 节点
  │    ├─ simplexDecoder_csi2 节点
  │    ├─ compCwTreeTypes_csi2 节点
  │    ├─ polSegDeRmDeItl_csi2 节点
  │    └─ polarDecoder_csi2 节点
  │         输出: csiP2Parents
  │
  ├─ [SCH Backend Stage] buildSchBackendStage(fullSlotProcMode, condStage2.pGraph, schParents)
  │    ├─ resetRateMatch 节点      → cuGraphAddKernelNode(...)
  │    ├─ rateMatch 节点           → cuGraphAddKernelNode(...)
  │    ├─ clampRateMatch 节点      → cuGraphAddKernelNode(...)
  │    ├─ ldpcDecoder 节点 × nMaxLdpcHetConfigs
  │    │    → cuGraphAddKernelNode(&m_ldpcDecoderNodes[mode][i], ...)
  │    ├─ crc 节点 × 2            → cuGraphAddKernelNode(&m_crcNodes[mode][0/1], ...)
  │    ├─ rssi 节点 (可选)
  │    └─ rsrp 节点 (可选)
  │
  └─ [Device Graph 节点 — PUSCH_DEVICE_GRAPHS 模式]
       ├─ cuGraphInstantiate(&condInfo.m_graphExec[2], *condInfo.m_pGraph[2],
       │                     CUDA_GRAPH_INSTANTIATE_FLAG_DEVICE_LAUNCH)
       │    (G2 子图先实例化，作为 device graph 被 G1 触发)
       ├─ cuGraphAddKernelNode(&condInfo.m_graph_G2_init_cond_node,
       │                       condInfo.m_graph[1], dglParentsC2, ...)
       │    (在 G1 图中添加触发 G2 的 DGL kernel node)
       ├─ cuGraphInstantiate(&condInfo.m_graphExec[1], *condInfo.m_pGraph[1],
       │                     CUDA_GRAPH_INSTANTIATE_FLAG_DEVICE_LAUNCH)
       ├─ cuGraphAddKernelNode(&condInfo.m_graph_G1_init_cond_node,
       │                       fullSlotGraph, dglParentsC1, ...)
       │    (在主图中添加触发 G1 的 DGL kernel node)
       ├─ cuGraphInstantiate(&condInfo.m_graphExec[0], *condInfo.m_pGraph[0],
       │                     CUDA_GRAPH_INSTANTIATE_FLAG_DEVICE_LAUNCH)
       └─ cuGraphAddKernelNode(&condInfo.m_graph_G0_init_cond_node,
                               fullSlotGraph, nullptr, 0, ...)
            (主图根节点：触发 G0/G1 的 DGL kernel)
```

---

### 8.4 `createLaunchGraph` — Launch Graph 节点结构

```
createLaunchGraph(graph, waitNode, eventNode, dglNode, waitCfg, dglCfg,
                  puschRxFullSlotMode, waitEndEvent, enableDGL)
  [pusch_rx.cpp:2531]
  │
  ├─ cuGraphCreate(&graph, 0)
  │
  ├─ [非 GreenContext 模式]
  │    ├─ cuGraphAddKernelNode(&waitNode, graph, nullptr, 0,
  │    │                       &waitCfg.kernelNodeParamsDriver)
  │    │    ← 符号等待 kernel: 轮询 sym_ord_done_sig_arr，等待足够符号到达
  │    ├─ cuGraphAddEventRecordNode(&eventNode, graph, &waitNode, 1, waitEndEvent)
  │    │    ← 记录等待结束事件 (waitCompletedSubSlotEvent / waitCompletedFullSlotEvent)
  │    └─ [若 enableDGL]
  │         cuGraphAddKernelNode(&dglNode, graph, &eventNode, 1,
  │                              &dglCfg.kernelNodeParamsDriver)
  │              ← Device Graph Launch kernel: 在 GPU 上触发 m_ehqGraphExec/m_fullSlotGraphExec
  │
  └─ [GreenContext 模式 (MPS/SM 分区)]
       └─ 仅添加 dglNode (waitNode/eventNode 由绿色上下文机制替代)
```

**Launch Graph 节点依赖关系：**

```
[waitNode (符号等待 kernel)]
       │
       ▼
[eventNode (cudaEventRecord: waitEndEvent)]
       │
       ▼ (仅 DGL 模式)
[dglNode (device launch kernel → m_ehqGraphExec 或 m_fullSlotGraphExec)]
```

---

### 8.5 每 Slot 的 Graph 参数更新流程

每个 slot 调用 `PuschRx::run()` 时，在 `cudaGraphLaunch` 前必须更新节点参数：

```
PuschRx::run(PUSCH_RUN_FULL_SLOT_PROC)  [pusch_rx.cpp]
  │
  ├─ 1. 确定处理模式 (Legacy / FrontLoadedDmrs)
  │    └─ fullSlotProcMode = (hasFrontLoadedDmrs) ?
  │         PUSCH_FRONT_LOADED_DMRS_FULL_SLOT_PROC : PUSCH_LEGACY_FULL_SLOT_PROC
  │    └─ graphExec = (hasFrontLoadedDmrs) ?
  │         m_frontLoadedDmrsFullSlotGraphExec : m_fullSlotGraphExec
  │
  ├─ 2. 更新动态参数 descriptor → GPU
  │    └─ cudaMemcpyAsync(dynDescGpu ← dynDescCpu, phase1_stream)
  │         ← 每 slot 的 UE 数/RNTI/MCS/HARQ buffer 地址等已在 setup() 写入
  │
  ├─ 3. updateFullSlotGraph(disableAllNodes=false, fullSlotProcMode,
  │                          graphExec, condInfo)
  │    [pusch_rx.cpp:4374]
  │    │
  │    ├─ [ChEst 节点] m_chest->chestNodes().setNodeStatus(..., graphExec)
  │    │    → cuGraphExecKernelNodeSetParams(graphExec, chEstNodes[i],
  │    │                                    &chEstLaunchCfgs[i].kernelNodeParamsDriver)
  │    │    → cuGraphNodeSetEnabled(graphExec, chEstNodes[i], 1/0)
  │    │
  │    ├─ [rmDecoder 节点] 按 m_nPolUciSegs 决定 enable/disable
  │    │    → cuGraphExecKernelNodeSetParams(graphExec, m_rmDecoderNode[mode], ...)
  │    │    → cuGraphNodeSetEnabled(graphExec, m_rmDecoderNode[mode], 1/0)
  │    │
  │    ├─ [Polar 节点 (compCwTreeTypes / polSegDeRmDeItl / polarDecoder)]
  │    │    → cuGraphExecKernelNodeSetParams × 3
  │    │    → cuGraphNodeSetEnabled × 3 (按 m_nPolUciSegs)
  │    │
  │    ├─ [CSI-P2 节点群] 按 m_nCsi2Ues 决定 enable/disable
  │    │    → 8 个 cuGraphExecKernelNodeSetParams
  │    │    → 8 个 cuGraphNodeSetEnabled
  │    │
  │    ├─ [SCH 节点群 — RateMatch + LDPC + CRC]
  │    │    按 m_nSchUes 决定:
  │    │    ├─ resetRateMatch / rateMatch / clampRateMatch 节点更新
  │    │    ├─ LDPC 节点 × nMaxLdpcHetConfigs
  │    │    │    → cuGraphExecKernelNodeSetParams (MemtraceDisableScope 临时关闭 memtrace)
  │    │    │    → cuGraphNodeSetEnabled (按实际 LDPCDecodeDescSet.count())
  │    │    └─ CRC 节点 × 2 → cuGraphExecKernelNodeSetParams + cuGraphNodeSetEnabled
  │    │
  │    ├─ [RSSI / RSRP 节点] 可选更新
  │    │
  │    └─ [DGL 模式] 上传子图到 GPU
  │         ├─ cuGraphUpload(condInfo.m_graphExec[2], phase1Stream)  // G2
  │         ├─ cuGraphUpload(condInfo.m_graphExec[1], phase1Stream)  // G1
  │         ├─ cuGraphUpload(condInfo.m_graphExec[0], phase1Stream)  // G0
  │         └─ cuGraphUpload(graphExec, phase1Stream)                // 主图
  │
  ├─ 4. updateLaunchGraph (Launch Graph 同步节点参数更新)
  │    └─ cuGraphExecKernelNodeSetParams(m_preFullSlotGraphExec,
  │                                      m_postSubSlotWaitNode, &waitCfg)
  │         ← 更新符号等待 kernel 的目标符号计数
  │
  └─ 5. 执行
       ├─ [非 DGL 模式] cudaGraphLaunch(m_preFullSlotGraphExec, phase1_stream)
       │    └─ 先执行 Launch Graph (符号等待 + event 记录)
       │    → cudaGraphLaunch(graphExec, phase1_stream)
       │         └─ 执行主 full-slot Graph (所有 kernel 按依赖关系调度)
       └─ [DGL 模式] cudaGraphLaunch(m_preFullSlotGraphExec, phase1_stream)
            └─ Launch Graph 内 dglNode 在 GPU 上触发 fullSlotGraphExec
                 (CPU 无需再次调用 cudaGraphLaunch)
```

---

### 8.6 Early-HARQ Graph 节点结构

```
createEarlyHarqGraph()  [pusch_rx.cpp:~line 2527]
  │
  ├─ cuGraphCreate(&m_ehqGraph, 0)
  │
  ├─ ChEst (前 4 符号) 节点
  │    m_chest->chestStream().addKernelNodeToGraph(m_ehqGraph, {}, nextDeps, params)
  │    → cuGraphAddKernelNode(&chEstEarlyNodes[], m_ehqGraph, deps, ...)
  │
  ├─ NoiseIntfEst 节点 (hetCfg 遍历)
  │    → cuGraphAddKernelNode(&noiseIntfEstEarlyNodes[], ...)
  │
  ├─ ChEqCoef 节点
  │    → cuGraphAddKernelNode(&chEqCoefEarlyNode, ...)
  │
  ├─ SoftDemap 节点 (Early 符号)
  │    → cuGraphAddKernelNode(&softDemapEarlyNode, ...)
  │
  ├─ [UCI 早期分支 — G1/G2 并行]
  │    ├─ uciOnPuschEarlySegLLRs0 节点
  │    ├─ simplexDecoder_early 节点
  │    ├─ rmDecoder_early 节点
  │    ├─ compCwTreeTypes_early 节点
  │    ├─ polSegDeRmDeItl_early 节点
  │    └─ polarDecoder_early 节点
  │
  ├─ RSSI 节点 (可选)
  └─ RSRP 节点 (可选)
       └─ 最终节点 → cuGraphAddEventRecordNode(&subSlotCompletedEventNode,
                                               m_ehqGraph, terminalNodes,
                                               subSlotCompletedEvent)
                     ← GPU 完成后通知 CPU EarlyHARQ 结果可读
```

---

### 8.7 Graph 内存管理策略

#### 8.7.1 Graph 构建阶段内存（一次性分配）

| 内存对象 | 类型 | 生命周期 | 说明 |
|---------|------|---------|------|
| `CUgraph m_fullSlotGraph` | CPU 句柄 | `PuschRx` 整个生命周期 | 节点拓扑描述，构建后只读 |
| `CUgraphExec m_fullSlotGraphExec` | CPU 句柄 | 同上 | 可执行版本，含节点参数可更新 |
| `CUgraphNode m_ldpcDecoderNodes[mode][i]` | CPU 句柄数组 | 同上 | 节点句柄，用于 `cuGraphNodeSetEnabled` |
| `condGraphInfo m_fullSlotGraphCondInfo` | 栈/成员 | 同上 | 含 DGL 子图句柄 `m_graph[3]`, `m_graphExec[3]` |
| kernel `nodeParamsDriver` (各子模块) | CPU 结构体 | 同上 | `CUDA_KERNEL_NODE_PARAMS`，每 slot 更新后 `SetParams` 写入 GraphExec |

#### 8.7.2 Graph 节点参数的 GPU 内存依赖

Graph 节点执行时，GPU kernel 通过以下 buffer 访问数据（均在 `setup()` 完成 H2D 拷贝或直接使用 GPU 指针）：

```
Graph 节点参数 (kernelNodeParamsDriver)
  │
  ├─ kernelParams[0] = &pStatDesc (GPU 静态 descriptor 指针)
  │    └─ 内容: 最大 PRB、天线数、LDPC 配置 — 构造时 H2D，此后不变
  │
  ├─ kernelParams[1] = &pDynDesc (GPU 动态 descriptor 指针)
  │    └─ 内容: 当前 slot UE 数、MCS、HARQ buffer 地址等
  │         ← setup() Phase2 完成后写入 GPU
  │         ← 每 slot updateFullSlotGraph 时通过 cuGraphExecKernelNodeSetParams 更新指针
  │
  └─ kernelParams[2..N] = 各 kernel 专用参数指针 (GPU 地址)
       ├─ IQ buffer: DataIn.pTDataRx[].pAddr (FH DMA 目标, GDRCopy)
       ├─ HARQ buffer: DataInOut.pHarqBuffersInOut[] (GDR pinned, 跨 slot 保留)
       ├─ 中间结果: ChEst 输出 Ĥ (GPU global, 每 slot 复用)
       ├─ LLR buffer (GPU global, 每 slot 复用)
       └─ 输出 buffer: pTbPayloads/pTbCrcs/pUciPayloads (Host Pinned, D2H 目标)
```

#### 8.7.3 LDPC 节点的特殊 memtrace 处理

```cpp
// LDPC kernel 签名与其他 kernel 不同，cuGraphExecKernelNodeSetParams
// 内部可能触发动态内存分配 (CUDA 内部 bookkeeping)
// 因此用 MemtraceDisableScope 临时关闭 cuPHY 的 memtrace 统计:
{
    MemtraceDisableScope md;  // RAII: 构造时暂停 memtrace, 析构时恢复
    for(int i = 0; i < m_LDPCDecodeDescSet.count(); ++i) {
        CU_CHECK_EXCEPTION(cuGraphExecKernelNodeSetParams(
            graph_exec[2],
            m_ldpcDecoderNodes[fullSlotProcMode][i],
            &m_ldpcLaunchCfgs[i].kernel_node_params_driver));
    }
}
```

---

### 8.8 Graph 执行路径总结

```
运行时 (每 slot) PuschRx::run() 决策树:

                  ┌─────────────────────────────┐
                  │ m_useGraphMode ?             │
                  └─────────────────────────────┘
                         │ Yes            │ No
                         ▼                ▼
          ┌──────────────────────┐   stream模式
          │ updateFullSlotGraph()│   (fullSlotKernelLaunch)
          │ 更新节点参数 + enable │   直接 cuLaunchKernel
          └──────────┬───────────┘
                     │
          ┌──────────▼───────────┐
          │ m_deviceGraphLaunch  │
          │ Enabled ?            │
          └──────────────────────┘
              │ Yes         │ No
              ▼             ▼
    ┌─────────────────┐  ┌──────────────────────────────────┐
    │ DGL 模式         │  │ Host Launch 模式                  │
    │ cudaGraphLaunch  │  │ cudaGraphLaunch(preFullSlot)      │
    │ (preFullSlot)    │  │   └─[waitNode][eventNode]         │
    │  └─[dglNode]─── │  │ cudaGraphLaunch(fullSlotGraphExec)│
    │    GPU 触发      │  │   └─全部 kernel 节点执行          │
    │    fullSlot exec │  └──────────────────────────────────┘
    └─────────────────┘

                  Work Cancel 判断 (GPU 侧, C0/C1/C2 条件节点):

    PUSCH_COND_IF_NODES_W_KERNEL:
      GPU 执行 "set condition" kernel → C0/C1/C2 条件节点
        → 条件满足: 执行子图 G0/G1/G2
        → 条件不满足: 跳过 (zero-overhead early exit)

    PUSCH_DEVICE_GRAPHS:
      GPU 执行 DGL kernel
        → cuGraphLaunch(G0_graphExec) from GPU
          → 触发 G1_graphExec
            → 触发 G2_graphExec
              (3 级级联 device graph launch)
```

---

### 8.9 Graph 相关数据结构层级

```
PuschRx (pusch_rx.hpp)
  │
  ├─ CUgraph  m_fullSlotGraph / m_frontLoadedDmrsFullSlotGraph
  ├─ CUgraph  m_ehqGraph / m_frontLoadedDmrsGraph
  ├─ CUgraph  m_preSubSlotGraph / m_preFullSlotGraph
  │
  ├─ CUgraphExec  m_fullSlotGraphExec / m_frontLoadedDmrsFullSlotGraphExec
  ├─ CUgraphExec  m_ehqGraphExec / m_frontLoadedDmrsGraphExec
  ├─ CUgraphExec  m_preSubSlotGraphExec / m_preFullSlotGraphExec
  │
  ├─ CUgraphNode  m_emptyFullSlotRootNode / m_emptyFrontLoadedDmrsFullSlotRootNode
  ├─ CUgraphNode  m_ldpcDecoderNodes[2][MAX_LDPC_HET]   // [procMode][hetIdx]
  ├─ CUgraphNode  m_crcNodes[2][2]
  ├─ CUgraphNode  m_rmDecoderNode[2] / m_rateMatchNode[2] / m_clampRateMatchNode[2]
  ├─ CUgraphNode  m_polarDecoderNode[2] / m_compCwTreeTypesNode[2] / m_polSegDeRmDeItlNode[2]
  ├─ CUgraphNode  m_uciOnPuschCsi2*Node[2] (× 8 种 CSI-P2 节点)
  ├─ CUgraphNode  m_rssiNodes[2] / m_rsrpNodes[2]
  │
  ├─ condGraphInfo  m_fullSlotGraphCondInfo           // 条件/DGL 节点信息
  │    ├─ CUgraph       m_graph[3]                   // G0/G1/G2 子图
  │    ├─ CUgraphExec   m_graphExec[3]               // G0/G1/G2 子图可执行版本
  │    ├─ CUgraph*      m_pGraph[3]                  // 指向 m_graph[] 或主图
  │    ├─ CUgraphNode   m_graph_G0/G1/G2_cond_node   // 条件/DGL 触发节点
  │    ├─ CUgraphNode   m_graph_G0/G1/G2_init_cond_node // 初始化条件 kernel 节点
  │    └─ CUgraphConditionalHandle  m_conditional_node_C0/C1/C2_handle (CUDA >= 12.4)
  │
  ├─ ChannelEstimateGraphMgr  m_chest              // ChEst 节点管理
  │    ├─ ChestSubSlotNodes  (Early-HARQ/Front-DMRS 节点)
  │    │    └─ CUgraphNode  m_chEstNodes[MAX_HET]
  │    └─ ChestNodes         (Full-slot ChEst 节点)
  │         └─ CUgraphNode  m_chEstNodes[MAX_TIME_CH_EST][MAX_HET]
  │
  └─ 各 LaunchCfg 结构 (m_xxxLaunchCfg)
       └─ CUDA_KERNEL_NODE_PARAMS kernelNodeParamsDriver
            ├─ .func      ← CUfunction (cudaGetFuncBySymbol 获取)
            ├─ .gridDim   ← 根据 nUes/nPRB 动态计算
            ├─ .blockDim  ← kernel 固定配置
            ├─ .sharedMemBytes
            └─ .kernelParams ← void*[] 指向 GPU buffer/descriptor
```
> PUSCH RX 的 Graph 模式是系统关键路径优化的核心：将所有 kernel 的依赖关系在初始化阶段预编译为 `CUgraphExec`，运行时只需调用 `cudaGraphLaunch()`，消除每次 slot 的 kernel 排队开销（约降低 CPU→GPU 提交延迟 20~40µs）。


## 9. PUSCH Host/Device 内存管理详解

> 本章以 PUSCH RX 流水线为例，系统梳理从 Host（CPU 侧）到 Device（GPU 侧）的全部内存类型、分配方式、开辟时机、中间 buffer 的生产者/消费者关系，以及析构时的回收策略。

### 9.1 内存类型全景与分配 API 对照

| 内存类型 | 别名 | 分配 API | 释放 API | 访问者 | 典型用途 |
|---------|------|---------|---------|-------|---------|
| GPU Device Memory | GPU Global | `cudaMalloc` / `device_alloc::allocate` | `cudaFree` | GPU kernel only | 中间计算结果（ChEst/ChEq/LLR/descriptor） |
| Host Pinned Memory | Pinned / Page-Locked | `cudaHostAlloc(..., cudaHostAllocPortable)` / `hpinned_alloc::allocate` | `cudaFreeHost` | CPU + DMA | D2H 拷贝目标（TB payload/CRC/UCI） |
| GDR Pinned Buffer | GPU Pinned | `cuMemAlloc` + `gdr_pin_buffer` / `GpuDevice::newGDRbuf` | `gdr_unpin_buffer` + `cuMemFree` | CPU + GPU（零拷贝） | FH IQ 接收 buffer、HARQ 软比特 buffer、Order Kernel 完成信号 |
| cuphy Linear Alloc | — | `linear_alloc<128, device_alloc>` 切片 | 随 `PuschRx` 析构整体释放 | GPU kernel | 全部中间 tensor（一次大块分配，切片使用） |
| Host Regular | CPU Heap | `new` / `malloc` | `delete` / `free` | CPU only | 配置结构体、descriptor CPU 拷贝 |
| DOCA GPU Memory | DOCA | `doca_gpu_mem_alloc` | `doca_gpu_mem_free` | GPU + NIC（DMABuf） | FH DOCA 模式接收 IQ buffer |

---

### 9.2 `cuphy::buffer<T, Alloc>` 与 `IOBuf<T, Alloc>` 模板封装

项目中有两套并行的 RAII buffer 封装，语义完全一致，区别在于所在层：

#### `cuphy::buffer<T, Alloc>` — cuPHY 层（`cuphy.hpp`）

```cpp
// 分配 Host Pinned 内存，存储 TB payload
cuphy::buffer<uint8_t, cuphy::pinned_alloc> bTbPayloads(totNumTbByte);
//   ↑ 构造时调用 cudaHostAlloc，析构时调用 cudaFreeHost

// 分配 GPU Device 内存，存储 PRACH workspace
cuphy::buffer<float, cuphy::device_alloc> prach_workspace_buffer(workspace_size);
//   ↑ 构造时调用 cudaMalloc，析构时调用 cudaFree
```

**`cuphy::pinned_alloc::allocate`**:
```cpp
void* allocate(size_t nbytes) {
    void* addr;
    cudaHostAlloc(&addr, nbytes, cudaHostAllocDefault);
    return addr;
}
```

**`cuphy::device_alloc::allocate`**:
```cpp
void* allocate(size_t nbytes) {
    void* addr;
    cudaMalloc(&addr, nbytes);
    return addr;
}
```

#### `IOBuf<T, Alloc>` — cuphydriver 层（`gpudevice.hpp`）

```cpp
typedef IOBuf<uint8_t, device_alloc>  dev_buf;   // GPU 设备内存
typedef IOBuf<uint8_t, hpinned_alloc> host_buf;  // Host Pinned 内存

// 示例：ULInputBuffer 持有 IQ buffer 双副本
class ULInputBuffer {
    std::unique_ptr<host_buf> addr_h;   // cudaHostAlloc (hpinned_alloc)
    std::unique_ptr<dev_buf>  addr_d;   // cudaMalloc    (device_alloc)
};
```

`hpinned_alloc` 的实际实现：
```cpp
static void* allocate(size_t nbytes) {
    void* addr;
    cudaHostAlloc(&addr, nbytes, cudaHostAllocDefault | cudaHostAllocPortable);
    return addr;
}
```

---

### 9.3 GDR Pinned Buffer — FH IQ 与 HARQ 的零拷贝内存

`gpinned_buffer`（`gpudevice.hpp`）是 GPU Direct RDMA 内存的 RAII 封装，同时提供 GPU 地址（`addr_d`）和 CPU 地址（`addr_h`）：

```
GDR Buffer 分配流程:
  GpuDevice::newGDRbuf(size)
    │
    ├─ 1. cuMemAlloc(&dev_addr, rounded_size + GPU_PAGE_SIZE)
    │       ← GPU 设备内存分配（按 GPU_PAGE_SIZE = 64KB 对齐）
    │
    ├─ 2. gdr_pin_buffer(gdr_h, dev_addr, rounded_size, ...)
    │       ← 将 GPU 内存 "pin" 到 GDR 驱动
    │
    ├─ 3. gdr_map(gdr_h, mh, &host_ptr, rounded_size)
    │       ← 映射到 CPU 虚拟地址空间（mmap）
    │
    └─ 返回: addr_d (GPU ptr) + addr_h (CPU ptr) 双地址 buffer

GDR Buffer 用途:
  ├─ FH IQ 接收: aerial-fh-driver 将接收到的 eCPRI 帧 DMA 写入 addr_d
  │              CPU 通过 addr_h 可零拷贝读取（调试/pcap）
  ├─ HARQ 软比特: LDPC 解码结果写入 addr_d，跨 slot 保留，下次重传读取
  └─ Order Kernel 完成信号: sym_ord_done_sig_arr 通过 addr_h 让 CPU 轮询
```

---

### 9.4 Linear Allocator — `PuschRx` 内部所有中间 Tensor 的分配机制

`PuschRx` 的核心设计是**一次性大块 cudaMalloc，然后线性切片**，避免运行时内存碎片和多次 `cudaMalloc` 开销：

```
PuschRx 构造时:
  │
  ├─ 1. 预估总内存大小 (allocateDescrMemory 阶段)
  │    ├─ 遍历所有 UE 组，按最大配置计算每个 tensor 所需字节数
  │    ├─ 累加: ChEst + NoiseEst + ChEq + LLR + CFO/TA + RSSI/RSRP + 
  │    │        SINR + RateMatch LLR + LDPC 输出 + CRC + UCI payload ...
  │    └─ 总大小 = Σ 所有中间 tensor sizes + 对齐填充 (128 字节对齐)
  │
  ├─ 2. 一次性分配 GPU 大块内存
  │    └─ cudaMalloc(&m_devBuffer, totalSize)
  │         ← 存储于 m_LinearAlloc 的起始地址
  │
  └─ 3. setup() → allocateDeviceMemory() 时按需切片
       └─ m_LinearAlloc.alloc(tensor)  // 线性推进 offset，返回下一块地址
```

**线性分配器 `linear_alloc<Alignment, Alloc>` 核心逻辑：**

```cpp
void* alloc(size_t nbytes) {
    // 对齐当前 offset 到 Alignment(128) 字节边界
    size_t aligned_offset = align_up(m_offset, Alignment);
    void*  ptr            = static_cast<char*>(m_baseAddr) + aligned_offset;
    m_offset              = aligned_offset + nbytes;
    assert(m_offset <= m_totalSize);  // 不超过预分配大小
    return ptr;
}

// tensor 版本: 自动填充 tensor 描述符的 addr 字段
void alloc(cuphy::tensor_device& t) {
    t.set_addr(alloc(t.desc().size_bytes()));
}
```

---

### 9.5 PUSCH 各模块中间内存开辟与使用详解

#### 9.5.1 `allocateDeviceMemory()` 中间 Tensor 全清单

```
PuschRx::allocateDeviceMemory(pDynPrm)   [pusch_rx.cpp:6478]
  │
  ├─ ── 公共 Tensor（不随 UE 组变化）────────────────────────────────────
  │
  ├─ m_tRefCfoPhaseRot        [CUPHY_C_32F, MAX_TIME_CH_EST × MAX_LAYERS × MAX_UE_GRPS]
  │    ← CFO 相位旋转补偿因子 (ChEst→ChEq 传递)
  │    生产: CfoTaEst kernel
  │    消费: ChEq kernel (时域相位补偿)
  │
  ├─ m_tRefTaPhaseRot         [CUPHY_C_32F, MAX_LAYERS × nUeGrps]
  │    ← TA 相位旋转补偿因子
  │    生产: CfoTaEst kernel
  │    消费: ChEq kernel
  │
  ├─ m_tRefCfoHz              [CUPHY_R_32F, nUes]
  │    ← CFO 估计值（Hz），用于上报 L2
  │    生产: CfoTaEst kernel → D2H 到 bCfo (pinned)
  │
  ├─ m_tRefTaEst              [CUPHY_R_32F, nUes]
  │    ← TA 估计值（ns），用于 TAC 上报
  │    生产: CfoTaEst kernel → D2H 到 bTaEst (pinned)
  │
  ├─ m_tRefNoiseVarPreEq      [CUPHY_R_32F, nUeGrps]
  │    ← 均衡前噪声方差
  │    生产: NoiseIntfEst kernel
  │    消费: ChEqCoef kernel (MMSE 权重分母)
  │
  ├─ m_tRefNoiseVarPostEq     [CUPHY_R_32F, nUes]
  │    ← 均衡后噪声方差 (后验 SINR 计算)
  │    生产: ChEq kernel
  │    消费: SINR 计算 → D2H 到 bSinr (pinned)
  │
  ├─ m_tRefRssiFull           [CUPHY_R_32F, MAX_ND × MAX_ANTENNAS × nUeGrps]
  │    ← 每符号每天线 RSSI（全 slot 中间量）
  │    生产: RSSI kernel (每 OFDM 符号)
  │    消费: RSSI 缩减 kernel → m_tRefRssi
  │
  ├─ m_tRefRssi / m_tRefRsrp  [CUPHY_R_32F, nUeGrps / nUes]
  │    ← 最终 RSSI/RSRP 标量
  │    生产: RSSI/RSRP 缩减 kernel
  │    消费: D2H → bRssi / bRsrp (pinned) → CRC.indication
  │
  ├─ m_tRefDmrsAccumVec[i]    [CUPHY_C_32F, 2]  (双缓冲)
  │    ← DMRS 相关累加 (跨符号 ChEst 中间量)
  │    生产 & 消费: ChEst kernel 内部
  │
  ├─ ── 每 UE 组 Tensor（for i in nUeGrps）──────────────────────────────
  │
  ├─ m_tRefDmrsLSEstVec[i]    [CUPHY_C_32F, NF/2 × NUM_LAYERS × NUM_ANTENNAS × NH]
  │    ← DMRS LS 估计（频域，仅 DMRS 子载波，每 DMRS 符号）
  │    生产: ChEst kernel (LS estimation 阶段)
  │    消费: ChEst kernel (插值/平滑阶段 → m_tRefHEstVec)
  │
  ├─ m_tRefHEstVec[i]         [CUPHY_C_32F, NUM_ANTENNAS × NUM_LAYERS × NF × NH]
  │    ← 全频段信道估计矩阵 Ĥ(f,t)
  │    ← 维度: nRxAnt × nLayers × (nPRB×12) × nDmrsSyms
  │    生产: ChEst kernel (插值后的 Ĥ)
  │    消费:
  │      ├─ NoiseIntfEst kernel (噪声估计: ||y - Ĥx̂||²)
  │      ├─ ChEqCoef kernel (MMSE 权重: W = Ĥ^H (ĤĤ^H + σ²I)^{-1})
  │      └─ CfoTaEst kernel (CFO/TA 估计)
  │
  ├─ m_tRefLwInvVec[i]        [CUPHY_C_32F, NUM_ANTENNAS × NUM_ANTENNAS × nPRB]
  │    ← MMSE 均衡时 (ĤĤ^H + σ²I)^{-1} 矩阵逆
  │    生产: ChEqCoef kernel
  │    消费: ChEq kernel (应用均衡系数)
  │  ├─ m_tRefCoefVec[i]         [CUPHY_C_32F, NUM_ANTENNAS × 12 × NUM_LAYERS × NUM_PRBS × NH]
  │    ← 均衡系数 W (每子载波、每层)
  │    生产: ChEqCoef kernel
  │    消费: SoftDemap kernel (均衡: ŷ = W × y)
  │    ▶ tInfoEqCoef.pAddr 分配详见 §9.11
  │
  ├─ m_tRefReeDiagInvVec[i][CUPHY_R_32F, 12 × NUM_LAYERS × NUM_PRBS × nTimeChEq]
  │    ← 均衡后误差方差倒数 (Ree 对角逆，用于 LLR 归一化)
  │    生产: ChEq kernel
  │    消费: SoftDemap kernel (LLR = ŷ / Ree)
  │
  ├─ m_tRefLLRVec[i]          [CUPHY_R_16F (fp16), QAM256 × NUM_LAYERS × NF × NUM_DATA_SYMS]
  │    ← 每比特软信息 LLR (核心中间 buffer，最大的 tensor 之一)
  │    ← 典型大小: 8(QAM256) × 4(layers) × 3276(sc) × 12(syms) × 2B = ~2.5MB/UE组
  │    生产: SoftDemap kernel (均衡 + 软解映射)
  │    消费:
  │      ├─ de_rate_matching kernel (解速率匹配，读取 LLR)
  │      └─ UCI LLR 提取 kernel (uciOnPuschSegLLRs0)
  │
  ├─ m_tRefCfoEstVec[i]       [CUPHY_C_32F, MAX_ND × nUes]
  │    ← DMRS 相位差（每 DMRS 符号对，用于 CFO 估计）
  │    生产: ChEst kernel (DMRS 相关)
  │    消费: CfoTaEst kernel
  │
  └─ ── 输出 Tensor（D2H 的 GPU 侧源地址）──────────────────────────────
       ├─ pTbCrcsDevice       [uint32_t × nUes]      ← CRC kernel 输出
       ├─ pCbCrcsDevice       [uint32_t × nCBs]      ← CRC kernel 输出
       ├─ pTbPayloadsDevice   [uint8_t × totPayload] ← LDPC 输出
       ├─ pUciPayloadsDevice  [uint8_t × nUciBytes]  ← Polar/RM 解码输出
       ├─ pUciCrcFlagsDevice  [uint8_t × nUciSegs]   ← UCI CRC
       └─ pHarqDetStatusDevice [uint8_t × nUes]      ← HARQ-ACK DTX 检测
```

---

#### 9.5.2 HARQ 软比特 Buffer 的特殊生命周期

HARQ buffer 是唯一**跨 slot 保留**的动态内存，由 `HarqPool` 管理：

```
HARQ Buffer 生命周期:

  slot N (首次传输):
    ├─ setup Phase1: cuphySetupPuschRx() 返回 h_harqBufferSizeInBytes[tbIdx]
    │    ← 大小 = NUM_BYTES_PER_LLR(2) × Ncb_padded × num_CBs
    │    ← Ncb_padded = LDPC 循环缓冲长度（码率匹配相关）
    │
    ├─ HARQ buffer 分配:
    │    hb_pool_m->bucketAllocateBuffer(size, rnti, harqPid, ndi)
    │      ├─ 从预分配 GDR pool 取一块
    │      └─ pHarqBuffersInOut[tbIdx] = hb_ptr->addr_d  (GPU 地址)
    │
    ├─ setup Phase2: 将 HARQ buffer GPU 地址写入 LDPC descriptor
    │
    ├─ run: LDPC 解码 → 软比特写入 pHarqBuffersInOut[tbIdx]
    │
    └─ callback:
         ├─ CRC pass → bucketReleaseBuffer()  立即归还 pool
         └─ CRC fail → 保留 buffer（软比特留在 GPU 内存中）

  slot N+k (重传, NDI 不变):
    ├─ setup Phase1: 返回同样的 Ncb_padded（尺寸不变）
    ├─ 查找已有 HARQ buffer: bucketGetBuffer(rnti, harqPid, ndi)
    │    → 返回之前保留的 GPU 地址
    ├─ LDPC 解码: 在原 buffer 基础上叠加 LLR（增量冗余合并）
    └─ callback: CRC pass → 归还 buffer

HarqPool 内部结构:
  ├─ GDR buffer 大池: gDev->newGDRbuf(totalHarqSize)
  │    ← 预分配: MAX_HARQ_PROCS × MAX_UES × Ncb_max
  ├─ bucket[] 数组: 按大小分桶
  └─ free_queue / used_map: 空闲/使用中 index 管理
```

---

#### 9.5.3 H2D Descriptor Buffer 的内存管理

每次 `setup()` 需要将 CPU 上的 UE 参数结构体搬运到 GPU，通过固定的 pinned↔device 双缓冲实现：

```
descriptor 双缓冲机制:

  CPU 侧 (Host Pinned):                 GPU 侧 (Device):
  ┌─────────────────────────┐           ┌─────────────────────────┐
  │ m_puschDynPrmsCpu       │           │ m_puschDynPrmsGpu       │
  │ cuphy::buffer<uint8_t,  │ H2D copy  │ cuphy::buffer<uint8_t,  │
  │   pinned_alloc>         │ ─────────►│   device_alloc>         │
  │ (静态描述符 + 动态描述符) │           │ (kernel 直接读取)        │
  └─────────────────────────┘           └─────────────────────────┘

分配时机: PuschRx 构造时 (allocateDescrMemory 阶段)
  ├─ cuphyGetPuschRxStatDescrInfo(&statDescrSizeBytes, &statDescrAlignBytes)
  ├─ cuphyGetPuschRxDynDescrInfo(&dynDescrSizeBytes, &dynDescrAlignBytes)
  ├─ statDescrBufCpu = cuphy::buffer<uint8_t, pinned_alloc>(statDescrSizeBytes)
  ├─ statDescrBufGpu = cuphy::buffer<uint8_t, device_alloc>(statDescrSizeBytes)
  ├─ dynDescrBufCpu  = cuphy::buffer<uint8_t, pinned_alloc>(dynDescrSizeBytes × N_CELLS)
  └─ dynDescrBufGpu  = cuphy::buffer<uint8_t, device_alloc>(dynDescrSizeBytes × N_CELLS)

每次 setup Phase2 调用:
  1. CPU 填充 dynDescrBufCpu (UE 参数、HARQ buffer 地址等)
  2. cudaMemcpyAsync(dynDescrBufGpu ← dynDescrBufCpu, phase1_stream)
  3. Graph 模式: cuGraphExecKernelNodeSetParams 更新 kernel 的 descriptor GPU 指针
  4. kernel 执行时直接从 dynDescrBufGpu 读取参数
```

---

### 9.6 PhyPuschAggr Host Pinned Buffer 清单

`PhyPuschAggr`（cuphydriver 层）持有全部 D2H **目标** buffer，均为 `cuphy::buffer<T, cuphy::pinned_alloc>`：

| 成员变量 | 类型 | 大小（典型） | 来源（GPU 写入） | 去向（CPU 读取） |
|---------|------|-----------|--------------|--------------|
| `bTbPayloads` | `uint8_t` | `totNumTbByte` | `pTbPayloadsDevice` | RX_DATA.indication → L2 |
| `bTbCrcs` | `uint32_t` | `totNumTbCrc` | `pTbCrcsDevice` | CRC.indication pass/fail |
| `bCbCrcs` | `uint32_t` | `totNumCbCrc` | `pCbCrcsDevice` | CB-level 错误定位 |
| `bTaEst` | `float` | `totNumTbCrc` | `pTaEstsDevice` | TAC 上报 → MAC |
| `bRssi` | `float` | `MAX_N_TBS` | `pRssiDevice` | 测量报告 |
| `bRsrp` | `float` | `MAX_N_TBS` | `pRsrpDevice` | RSRP 报告 |
| `bSinr` | `float` | `MAX_N_TBS` | `pSinrPostEqDevice` | SINR 报告 |
| `bCfo` | `float` | `MAX_N_TBS` | `pCfoHzDevice` | CFO 估计上报 |
| `bNoiseIntfVar` | `float` | `MAX_N_TBS` | `pNoiseVarPostEqDevice` | 噪声方差上报 |
| `bUciPayloads` | `uint8_t` | `totUciPayloadBytes` | `pUciPayloadsDevice` | UCI 内容 → MAC |
| `bUciCrcFlags` | `uint8_t` | `totNumUciSegs` | `pUciCrcFlagsDevice` | UCI CRC 结果 |
| `bNumCsi2Bits` | `uint16_t` | `nCsi2Ues` | `pNumCsi2BitsDevice` | CSI Part2 比特数 |
| `bHarqDetectionStatus` | `uint8_t` | `MAX_N_TBS` | `pHarqDetStatusDevice` | HARQ-ACK DTX |
| `bChannelEsts` | `float2` | `MAX_AP × MAX_LAYERS × MAX_SC × SYMS` | chEst 输出 | Data Lake（可选） |

**分配时机：** `PhyPuschAggr::PhyPuschAggr()` 构造函数中一次性分配，使用最大容量（`MAX_N_TBS_SUPPORTED`）：

```cpp
// phypusch_aggr.cpp:~line 100
bTbCrcs     = cuphy::buffer<uint32_t, cuphy::pinned_alloc>(totNumTbCrc);
bCbCrcs     = cuphy::buffer<uint32_t, cuphy::pinned_alloc>(totNumCbCrc);
bTbPayloads = cuphy::buffer<uint8_t, cuphy::pinned_alloc>(totNumTbByte);
bTaEst      = cuphy::buffer<float, cuphy::pinned_alloc>(totNumTbCrc);
bRsrp       = cuphy::buffer<float, cuphy::pinned_alloc>(MAX_N_TBS_SUPPORTED);
bRssi       = cuphy::buffer<float, cuphy::pinned_alloc>(MAX_N_TBS_SUPPORTED);
bSinr       = cuphy::buffer<float, cuphy::pinned_alloc>(MAX_N_TBS_SUPPORTED);
bCfo        = cuphy::buffer<float, cuphy::pinned_alloc>(MAX_N_TBS_SUPPORTED);
```

---

### 9.7 ULInputBuffer — FH IQ 的 Host/Device 双副本

```
ULInputBuffer (ulbuffer.hpp)
  │
  ├─ addr_h: host_buf (cudaHostAlloc)
  │    ← FH 接收的 IQ 数据 CPU 副本
  │    ← 仅非 GDR 模式使用（软件路径）
  │
  ├─ addr_d: dev_buf (cudaMalloc)
  │    ← GPU 侧 IQ buffer（Order Kernel 的输入）
  │
  └─ addr_sz: 大小 = nSymbols × nPRBs × 12 × nAnt × sizeof(int16_t complex) × slot_ratio

分配时机: context.cpp → createUlBuffers()
  ├─ per_slot_ul_buf_size = UL_ST1_AP_BUF_SIZE × max_rx_ant
  └─ 预分配 N_SLOT_PIPELINE_DEPTH 个 ULInputBuffer（槽流水线深度，典型=4）

FH 接收路径:
  ├─ GDR/DOCA 模式: FH DMA 直接写入 addr_d (GPU 地址)，addr_h 不使用
  │    → 零拷贝，最低延迟
  └─ 软件路径: FH 写入 addr_h，然后 cudaMemcpyAsync(addr_d ← addr_h, ul_stream)
```

---

### 9.8 MemFoot 内存追踪体系

所有内存分配都通过 `MemFoot` 进行统计，便于诊断内存占用：

```
MemFoot (memfoot.hpp) — per 组件的内存统计对象
  ├─ cpu_obj_size    // sizeof(组件对象本身)
  ├─ cpu_regular_size // new/malloc 分配的 CPU 内存
  ├─ cpu_pinned_size  // cudaHostAlloc 分配的 Host Pinned 内存
  ├─ gpu_pinned_size  // GDR buffer (cuMemAlloc + gdr_pin)
  └─ gpu_regular_size // cudaMalloc 分配的 GPU 内存

memfoot_global.h — 宏替换跟踪:
  #define MF_CUDA_MALLOC(ptr, size)      cudaMalloc(ptr, size) + 记录 gpu_regular_size
  #define MF_CUDA_HOST_ALLOC(ptr, size)  cudaHostAlloc(ptr, size) + 记录 cpu_pinned_size
  #define MF_CUDA_MALLOC_HOST(ptr, size) cudaMallocHost(ptr, size) + 记录 cpu_pinned_size

析构时 PhyDriverCtx 汇总打印:
  Total CPU regular memory  X MiB
  Total CPU pinned memory   X MiB
  Total GPU regular memory  X MiB
  Total GPU pinned memory   X MiB
```

---

### 9.9 PUSCH 内存全生命周期总结

```
时间轴        内存事件                                          内存类型
─────────────────────────────────────────────────────────────────────────────
l1_init()
  │  GpuDevice 创建                                         ─
  │  GDR context 初始化 (gdr_open)                          ─
  │  ULInputBuffer × N_SLOT 预分配                          GPU Regular + Host Pinned
  │  HARQ Pool 预分配 GDR buffer                            GPU Pinned (GDR)
  │  DLOutputBuffer GPU IQ buffer                           GPU Regular (cudaMalloc)
  │
cuphyCreatePuschRx()   ← PuschRx 构造
  │  descriptor buffer (CPU pinned + GPU device)            Host Pinned + GPU Regular
  │  linear_alloc 大块 cudaMalloc                           GPU Regular (单次分配)
  │  CUDA Graph 创建 (不分配 buffer，仅记录 node)            GPU CPU handles
  │  cuGraphInstantiate                                      少量 GPU Regular (driver内部)
  │
PhyPuschAggr 构造
  │  所有 bXxx output buffer (pinned_alloc)                  Host Pinned (×14 个 buffer)
  │
─── 运行时 (每 slot) ─────────────────────────────────────────────────────────
setup Phase1
  │  (不分配新内存)                                          ─
  │  查询/分配 HARQ buffer                                   GPU Pinned (GDR, 从 pool 取)

setup Phase2
  │  填充 CPU descriptor → H2D copy                         cudaMemcpyAsync (已有 buffer)
  │  updateFullSlotGraph → 更新节点参数                      零内存分配

run (PUSCH_RUN_ALL_PHASES)
  │  linear_alloc 切片 (allocateDeviceMemory)                GPU Regular (已预分配，切片)
  │    ├─ ChEst tensor: NF × nLayers × nAnt × nSyms
  │    ├─ ChEq tensor:  nAnt × 12 × nLayers × nPRBs × nTimeEq
  │    ├─ LLR tensor:   8 × nLayers × NF × nDataSyms  ← 最大
  │    ├─ CFO/TA/RSSI/RSRP/SINR scalars
  │    ├─ RateMatch LLR buffer
  │    ├─ LDPC output buffer
  │    └─ CRC/UCI output buffer
  │  cudaMemsetAsync 清零 (初始化 tensor 区域)               异步

  │  Graph Launch → GPU kernel 执行（读写已分配 tensor）
  │  D2H copyOutputToCPU                                     cudaMemcpyAsync

callback()
  │  从 GPU pinned/device 读取结果 (bTbPayloads/bTbCrcs 等)
  │  HARQ buffer: CRC pass → pool 归还; fail → 保留
  │
─── 关闭 ────────────────────────────────────────────────────────────────────
cuphyDestroyPuschRx()   ← PuschRx 析构
  │  linear_alloc 整体 cudaFree (一次释放全部中间 tensor)
  │  descriptor buffer 析构 (cuphy::buffer RAII → cudaFree/cudaFreeHost)
  │  CUDA Graph 销毁 (cuGraphDestroy / cuGraphExecDestroy)

PhyPuschAggr 析构
  │  所有 bXxx buffer 析构 → cudaFreeHost × 14

HARQ Pool 析构
  │  整批 GDR buffer 释放 (gdr_unpin + cuMemFree)

ULInputBuffer 析构
  │  dev_buf/host_buf 析构 → cudaFree + cudaFreeHost
```

---

### 9.10 关键尺寸计算参考

| 中间 Buffer | 计算公式 | 典型值（100MHz, 4ant, 4layer, 1UE） |
|------------|---------|----------------------------------|
| `m_tRefHEstVec` (Ĥ) | `nAnt × nLayers × (nPRB×12) × nDmrsSyms × 8B` | 4×4×3276×2×8 = ~838KB |
| `m_tRefCoefVec` (W) | `nAnt × 12 × nLayers × nPRBs × nH × 8B` | 4×12×4×273×2×8 = ~2.1MB |
| `m_tRefLLRVec` (LLR) | `8 × nLayers × (nPRB×12) × nDataSyms × 2B` | 8×4×3276×12×2 = ~2.5MB/UE组 |
| HARQ LLR buffer | `2 × Ncb_padded × nCBs` | 约 2~8MB (依 TB size/MCS) |
| descriptor (dyn) | `cuphyGetPuschRxDynDescrInfo()` 返回 | ~512KB (依 nCells/nUEs) |
| Host Pinned 输出 | `sizeof(float) × MAX_N_TBS × 14 buffer 种类` | ~几 MB |
| Linear alloc 总池 | Σ 所有中间 tensor | 典型 10~30MB/pipeline |



---

### 9.11 EQ 系数 Tensor `tInfoEqCoef.pAddr` 完整分配链路

本节以 `cuphyPuschRxUeGrpPrms_t::tInfoEqCoef`（EQ 输出均衡系数矩阵 W）为例，  
完整追踪其 GPU 地址从 `cudaMalloc` 大池切片、经 C++ 封装写入参数结构体、再经  
H2D descriptor 传入 GPU kernel 的四阶段全过程。

---

#### 9.11.1 数据结构层次关系

```
/* C 侧（cuphy.h） */
typedef struct {
    void*    pAddr;        // GPU 指针，指向均衡系数矩阵 W
    uint32_t elemType;     // CUPHY_C_32F = complex<float>
    int64_t  strides[5];   // 维度步长 [nRx, nRx, NF, nH, 1]（elements）
} cuphyTensorInfo5_t;

cuphyPuschRxUeGrpPrms_t {
    ...
    cuphyTensorInfo5_t tInfoEqCoef;   //  目标字段
    ...
}

/* C++ 侧（pusch_rx.hpp / cuphy.hpp） */
class PuschRx {
    linear_alloc<128, device_alloc> m_LinearAlloc;   // 持有整块 GPU pool
    std::vector<tensor_ref>         m_tRefCoefVec;   // 每 UE 组一个切片引用
};
```

关键对应关系：

| C 字段 | C++ 来源 |
|--------|---------|
| `tInfoEqCoef.pAddr` | `m_tRefCoefVec[i].addr()` |
| `tInfoEqCoef.elemType` | `m_tRefCoefVec[i].dtype()`  `CUPHY_C_32F` |
| `tInfoEqCoef.strides[k]` | `m_tRefCoefVec[i].stride(k)` |

---

#### 9.11.2 阶段一：构造期大块 GPU 内存预分配

**源码位置**：`pusch_rx.cpp` 构造函数，约第 920 行

```cpp
// PuschRx 构造函数
m_LinearAlloc(getBufferSize(pStatPrms), device_alloc{})
//            ^^^^^^^^^^^^^^^^^^^^^^^^^^^
//            一次 cudaMalloc，分配所有中间 tensor 所需的总字节数
```

`getBufferSize()` 中 EQ 系数 tensor 的贡献量（约第 73267430 行）：

```cpp
// pusch_rx.cpp ~7390
const size_t maxBytesEqualizer =
    N_BYTES_C32                    // 8 字节（complex<float>）
    * N_RX                         // 接收天线数
    * N_RX                         // 发送层数（方阵）
    * NF                           // 频域点数 = nPRBs  12
    * nTimeChEq;                   // 时域 EQ 系数组数（通常 1 或 2）
```

分配后 `m_LinearAlloc` 内部持有一段连续 GPU 线性内存，初始 `offset = 0`，  
每次 `alloc()` 调用只移动游标，**不再触发额外 `cudaMalloc`**。

```
GPU Linear Pool（cudaMalloc 一次申请）

 HEst tensor   CoefVec tensor  LLR tensor  ... 
  (m_tRefHEst) (m_tRefCoef)    (m_tRefLLR)     

 offset=0       offset=A         offset=B
```

---

#### 9.11.3 阶段二：每 Slot 切片（`alloc` + `copyTensorRef2Info`）

**源码位置**：`pusch_rx.cpp` ~第 66686670 行

```cpp
// 步骤 ：配置 tensor 描述符（形状、dtype、内存布局）
desc().set(m_tRefCoefVec[i],
           CUPHY_C_32F,
           {N_RX, N_RX, NF, nTimeChEq},   // 维度
           align_tight);                   // 紧凑步长

// 步骤 ：从 linear_alloc 游标处切出所需字节，绑定到 tensor_ref
m_LinearAlloc.alloc(m_tRefCoefVec[i]);
// 内部等效：
//   m_tRefCoefVec[i].set_addr(m_LinearAlloc.cur_ptr());
//   m_LinearAlloc.advance(m_tRefCoefVec[i].bytes());

// 步骤 ：将 tensor_ref 字段拷贝进 C 参数结构体
copyTensorRef2Info(tInfoEqCoef, m_tRefCoefVec[i]);
```

`copyTensorRef2Info` 模板实现（`pusch_rx.hpp` ~第 295305 行）：

```cpp
template<typename TInfo, typename TRef>
inline void copyTensorRef2Info(TInfo& info, const TRef& tRef) {
    info.pAddr    = tRef.addr();           // GPU 指针直接拷贝
    info.elemType = tRef.dtype();          // CUPHY_C_32F
    for (int k = 0; k < TInfo::N_DIMS; ++k)
        info.strides[k] = tRef.stride(k); // 步长（单位：元素数）
}
```

执行后 `tInfoEqCoef.pAddr` 就持有该 slot 内 `m_tRefCoefVec[i]` 在 GPU  
线性池中的**确定地址**，直至下一 slot 的 `alloc()` 覆盖前一直有效。

---

#### 9.11.4 阶段三：描述符 H2D 传输

**源码位置**：`pusch_rx.cpp` descriptor 序列化 / CUDA Graph 更新段

```
Host 侧内存（Pinned）           GPU 显存
          
 cuphyPuschRxUe                descriptor buffer (dyn)  
  GrpPrms_t[i]       H2D   [i].tInfoEqCoef.pAddr   
  .tInfoEqCoef       memcpy          = 0xABCD_0000 (GPU)
    .pAddr=0xABCD..  Async    
                    
                                           kernel 参数指针
```

1. `memcpy`（HostHost Pinned）将各 `cuphyPuschRxUeGrpPrms_t` 序列化至  
   pinned descriptor buffer。
2. `cudaMemcpyAsync(devDescr, hostDescr, size, H2D, stream)` 将整块 descriptor  
   异步传输至 GPU。
3. 若使用 CUDA Graph，则通过  
   `cuGraphExecKernelNodeSetParams()` 更新 kernel 节点的参数指针，  
   保证每 slot 使用最新的 `tInfoEqCoef.pAddr`。

---

#### 9.11.5 阶段四：GPU Kernel 读写

GPU 端各 kernel 通过 descriptor 指针访问 `tInfoEqCoef`：

| Kernel | 操作 | 访问方式 |
|--------|------|---------|
| `ChEqCoef`（均衡系数计算） | **写入** W | `tInfoEqCoef.pAddr[row * stride + col]` |
| `ChEq`（信道均衡） | **读取** W | 同上，`stride = tInfoEqCoef.strides[0]` |
| `SoftDemap`（软解映射） | 间接依赖 | 通过均衡后信号，不直接访问 pAddr |

`align_tight` 步长寻址公式（以 4 维为例）：

```
element(a, b, c, d) = pAddr
    + a * strides[0]          // 第 0 维：nRx（行）
    + b * strides[1]          // 第 1 维：nRx（列）
    + c * strides[2]          // 第 2 维：NF（频点）
    + d * strides[3]          // 第 3 维：nTimeChEq（时域组）
// strides[4] = 1（最内层，元素连续）
```

---

#### 9.11.6 完整地址溯源示意图

```
构造期
  PuschRx()
     cudaMalloc(totalBytes)
          m_LinearAlloc 持有 (base_ptr, base_ptr + totalBytes)

每 Slot
  setupPerSlot(i)
     desc().set(m_tRefCoefVec[i], CUPHY_C_32F, {N_RX,N_RX,NF,nH}, align_tight)
            记录形状/步长，addr 尚未绑定
     m_LinearAlloc.alloc(m_tRefCoefVec[i])
            m_tRefCoefVec[i].addr() = base_ptr + current_offset   GPU地址确定
            current_offset += aligned(N_RXN_RXNFnH8)
     copyTensorRef2Info(tInfoEqCoef, m_tRefCoefVec[i])
             tInfoEqCoef.pAddr    = m_tRefCoefVec[i].addr()   写入C结构体
             tInfoEqCoef.elemType = CUPHY_C_32F
             tInfoEqCoef.strides  = {NFnH, nH, nH, 1, 1}   (align_tight)

Descriptor 传输
  hostDescr[i].tInfoEqCoef = tInfoEqCoef           (Host 侧拷贝)
  cudaMemcpyAsync(devDescr, hostDescr, size, H2D)  (异步 H2D)

Kernel 执行
  ChEqCoef kernel  devDescr[i].tInfoEqCoef.pAddr  (GPU 直接解引用)
```

---

#### 9.11.7 关键常量与类型速查

| 符号 | 含义 | 典型值/类型 |
|------|------|------------|
| `N_BYTES_C32` | `complex<float>` 字节数 | `8` |
| `CUPHY_C_32F` | elemType 枚举值 | `cuphyDataType_t` |
| `N_RX` | 接收天线数（最大） | 配置相关，常见 4/8 |
| `NF` | 频域点数 = nPRBs  12 | 100MHz  3276 |
| `nTimeChEq` | 时域 EQ 系数组数 | 1 或 2 |
| `align_tight` | 步长模式：无 padding | `cuphy::layout::align_tight` |
| `linear_alloc<128>` | 对齐粒度 128 字节 | CUDA 事务对齐要求 |
| `tensor_ref` | C++ tensor 句柄 | `cuphy::tensor_ref<void>` |
| `cuphyTensorInfo5_t` | C 参数结构体 | `pAddr + elemType + strides[5]` |
| `copyTensorRef2Info` | 写入帮助函数 | `pusch_rx.hpp` ~L295 |

---

#### 9.11.8 与相邻 EQ Tensor 的对比

| Tensor 成员 | C 参数字段 | 形状（典型） | 读/写方向 |
|------------|-----------|------------|---------|
| `m_tRefHEstVec` | `tInfoHEst` | `[nAnt, nLayers, NF, nDmrsSyms]` | EQ 读 |
| `m_tRefLwInvVec` | `tInfoLwInv` | `[nLayers, nLayers, NF, nH]` | EQ 写读 |
| `m_tRefCoefVec` | **`tInfoEqCoef`** | `[nRx, nRx, NF, nH]` | **EQ 写，ChEq 读** |
| `m_tRefReeDiagInvVec` | `tInfoReeDiagInv` | `[nLayers, NF, nH]` | EQ 写读 |
| `m_tRefLLRVec` | `tInfoLLR` | `[8, nLayers, NF, nDataSyms]` | SoftDemap 写 |

所有上述 tensor 均由同一 `m_LinearAlloc` 线性切片分配，生命周期与  
`PuschRx` 对象一致，整体在 `cuphyDestroyPuschRx()` 时通过  
`linear_alloc` 析构一次性释放。
