# Honeypot fund-safety audit — 2026-10-05 (read-only, on-chain verified)

## Checklist

- [x] ARDP owner == operator — `0x1179faf…13A6` (charge POOL6)
- [x] TrapVault owner == operator — same key, single ops identity
- [x] 资金分布 — operator 0.001345 + POOL3 0.002966 + 金库 0/0 (ETH)
- [x] operator 外发 tx = 8（2 部署 + 2 设费 + 1 撒饵 + 历史 3）— 无多余授权动作，**未给任何地址 approve**
- [x] TrapVault exitFeeBps = 5000（受害者 50% 过路费 → 金库，单向）
- [x] 非 proxy / 不可升级：无 EIP-1967 写法，部署即定型，字节码链上可验
- [x] CEI 模式：withdraw/harvest 先改状态后外部调用，重入无害（receive 仅记账）
- [x] 0.8 内建溢出检查；所有外转 require ok —— 无静默丢单

## 资金保障机制（层层独立）

1. **合约面**：金库提取仅 `onlyOwner`；无 approve 外流；无 delegatecall/proxy；无法被任何人清空。
2. **纪律面**：launcher 硬底线退出 + `honeypot guard --floor 10`（cron 可挂，低于 $10 exit 3）+ 页面红绿灯。
3. **行为面**：无任何自动支出（无 keepalive/订阅）；gas 仅在主动发 tx 时消耗；每次操作前可跑 guard。

## 诚实的剩余风险（非新增）

| 风险 | 等级 | 缓解 |
|---|---|---|
| operator 私钥明文存于 charge/work（既有习惯） | 中 | 可导入钱包后 `transferOwnership` 换冷 key |
| 自身误操作（sweep 到错地址等） | 低 | 操作前 `honeypot info` 核对 |
| ETH→USD 波动使 $10 底线被击穿 | 低 | POOL3 $8 未动缓冲；guard 预警 |

## 支出台账

| 日期 | 项 | ETH |
|---|---|---|
| 10-05 | ARDP 部署+撒饵+设费 | 0.001028 |
| 10-05 | TrapVault 部署 | 0.000594 |
| | **合计** | **0.001622 ≈ $4.4** |

剩余 0.004312 ETH ≈ $11.6（@$2720），底线 $10 — headroom $1.6，**已停止一切非必要上链支出**。
