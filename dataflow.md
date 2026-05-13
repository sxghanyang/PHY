## 0. PUSCH RX 完整数据流图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        输入数据                                              │
│                                                                              │
│   tInfoDataRx                                                                │
│   (NF, ND, N_BS_ANTS)  FP16复数                                             │
│   [频域子载波 × 数据符号 × 基站天线]                                          │
└───────────────────────────────┬─────────────────────────────────────────────┘
                                │
                ┌───────────────┴───────────────┐
                ▼                               ▼
┌───────────────────────────┐   ┌───────────────────────────────────────────┐
│   1. 信道估计              │   │   (DMRS导频子载波提取)                      │
│  cuphySetupPuschRxChEst   │   │   tInfoDmrsLSEst                          │
│                           │   │   (NF/2, N_LAYERS, N_BS_ANTS, NH)         │
│  LS估计 → 插值/滤波        │   │   FP32复数  [LS初步估计]                   │
└───────────────┬───────────┘   └───────────────────────────────────────────┘
                │
                │  tInfoHEst
                │  (N_BS_ANTS, N_LAYERS, NF, NH)  FP32复数
                │  [天线 × 层 × 子载波 × 时域DMRS符号]
                │
        ┌───────┴────────────────────────┐
        │                                │
        ▼                                ▼
┌───────────────────────┐   ┌────────────────────────────────┐
│  2. 噪声/干扰估计      │   │   3. CFO/TA估计                 │
│  cuphySetupPuschRx    │   │   cuphySetupPuschRxCfoTaEst    │
│  NoiseIntfEst         │   │                                │
│                       │   │  累积相位旋转:                   │
│  输入:                │   │  tInfoCfoPhaseRot              │
│  ├─ tInfoDataRx       │   │  (MAX_N_TIME_CH_EST,           │
│  └─ tInfoHEst         │   │   N_MAX_LAYERS, N_UE_GRPS)     │
│                       │   │  FP32复数 [需预置零]            │
│  输出:                │   │                                │
│  ├─ tInfoNoiseVar     │   │  tInfoTaPhaseRot               │
│  │  PreEq             │   │  (N_MAX_LAYERS, N_UE_GRPS)     │
│  │  (N_UE_GRP) FP32   │   │  FP32复数 [需预置零]            │
│  │  [噪声方差/dB]      │   └──────────────┬─────────────────┘
│  │                    │                  │
│  └─ tInfoLwInv        │      tInfoCfoEst │  (MAX_ND_SUPPORTED, N_LAYERS)
│     (N_BS_ANTS,       │      tInfoTaEst  │  (N_UE)  FP32实数 [秒]
│      N_BS_ANTS,       │      tInfoCfoHz  │  (N_UE)  FP32实数 [Hz]
│      N_PRB)  FP32复数 │                  │
│     [Cholesky逆-IRC]  │                  │
└──────────┬────────────┘                  │
           │                               │
           │ tInfoLwInv                    │ tInfoCfoEst
           └────────────────┬──────────────┘
                            │
                            │  + tInfoHEst
                            ▼
            ┌───────────────────────────────────────────────────────────┐
            │  4. 均衡系数计算                                           │
            │  cuphySetupPuschRxChEqCoefCompute                        │
            │                                                           │
            │  [标准 MMSE]                  [MMSE-IRC]                  │
            │  M = H' · σ⁻²·I              N = LwwInv · H             │
            │  G = M·H + I                 G = N'·N + I               │
            │                              M = N'·LwwInv               │
            │         ↓                           ↓                    │
            │   A = [G | I | M]             A = [I | M]               │
            │         ↓                           ↓                    │
            │  LU分解 luFactorize*()        LDL分解 computeLDL()       │
            │         ↓                           ↓                    │
            │  回代 backSub()               前代+回代求解               │
            │         ↓                           ↓                    │
            │         偏置校正 λ = 1/(1 - Ree[i][i])                   │
            │         ↓                           ↓                    │
            │   tCoef = C × λ            tReeDiagInv = 1/(λ·Ree[i][i])│
            └──────────────┬────────────────────────┬──────────────────┘
                           │                        │
                           ▼                        ▼
               ┌─────────────────────┐   ┌───────────────────────┐
               │  tInfoEqCoef        │   │  tInfoReeDiagInv       │
               │  (N_SC, N_LAYERS,   │   │  (N_SC, N_LAYERS,      │
               │   N_BS_ANTS, N_PRB, │   │   N_PRB, NH)           │
               │   NH)               │   │  FP32实数               │
               │  FP32/FP16复数      │   │  [残差误差方差逆 Ree⁻¹] │
               │  [均衡权值矩阵W]     │   └─────────┬─────────────┘
               └──────────┬──────────┘             │
                          │                        │
                          └────────────┬───────────┘
                                       │  + tInfoDataRx
                                       ▼
                       ┌───────────────────────────────────┐
                       │  5. 软解调（均衡应用）              │
                       │  cuphySetupPuschRxChEqSoftDemap   │
                       │                                   │
                       │  ŝ = Σ C[l][a]·Y[a]  →  LLR计算  │
                       │  noiseInv = ReeInv[layer]         │
                       │                                   │
                       │  → simplified_soft_demapper(默认) │
                       │  → texture_soft_demapper(TEX=1)   │
                       │  → legacy PAM-LLR(TEX=0)          │
                       └──────────────┬────────────────────┘
                                      │
                                      ▼
                       ┌───────────────────────────────────┐
                       │  tInfoLLR                         │
                       │  (N_LLR, N_LAYERS, NF, ND)        │
                       │  FP16实数                          │
                       │  N_LLR: QPSK→2, 16QAM→4,         │
                       │         64QAM→6, 256QAM→8         │
                       └──────────────┬────────────────────┘
                                      │
                      ┌───────────────┼───────────────┐
                      ▼               ▼               ▼
               ┌────────────┐  ┌────────────┐  ┌────────────┐
               │ 6. 速率去  │  │ 7. LDPC    │  │ 8. UCI解码 │
               │    匹配    │  │    解码    │  │  (Polar/RM)│
               │ RateMatch  │  │ LDPCDecode │  │            │
               └────────────┘  └────────────┘  └────────────┘
                      │               │               │
                      └───────────────┼───────────────┘
                                      ▼
                       ┌───────────────────────────────────┐
                       │           最终输出                 │
                       │  TB Payload / CRC / TA / CFO_Hz   │
                       │  NoiseVar / SINR / RSSI / RSRP    │
                       └───────────────────────────────────┘
```

### 关键张量维度速查

| 张量 | 维度 | 类型 | 流图位置 |
|------|------|------|---------|
| `tInfoDataRx` | `(NF, ND, N_BS_ANTS)` | FP16复数 | 全局输入 |
| `tInfoHEst` | `(N_BS_ANTS, N_LAYERS, NF, NH)` | FP32复数 | 步骤1→2,3,4 |
| `tInfoLwInv` | `(N_BS_ANTS, N_BS_ANTS, N_PRB)` | FP32复数 | 步骤2→4 |
| `tInfoNoiseVarPreEq` | `(N_UE_GRP)` | FP32实数 | 步骤2输出 |
| `tInfoCfoEst` | `(MAX_ND_SUPPORTED, N_LAYERS)` | FP32复数 | 步骤3→4 |
| `tInfoEqCoef` | `(N_SC, N_LAYERS, N_BS_ANTS, N_PRB, NH)` | FP32/FP16复数 | 步骤4→5 |
| `tInfoReeDiagInv` | `(N_SC, N_LAYERS, N_PRB, NH)` | FP32实数 | 步骤4→5 |
| `tInfoLLR` | `(N_LLR, N_LAYERS, NF, ND)` | FP16实数 | 步骤5→6,7,8 |
