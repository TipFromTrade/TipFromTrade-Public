# TipFromTrade v2.5 自定义金库透明说明

更新日期：2026-09-25

## 当前状态

v2.5 是 TipFromTrade 为 Flap Tax Token V3 准备的自定义 BNB 税收金库候选版本。它已经完成 BSC 公共测试网验证、固定版本单元测试和只读 BSC 主网分叉测试，但目前：

- **尚未部署为 TipFromTrade 正式主网工厂；**
- **尚未进入 Flap 官方登记列表，Flap 页面会显示“未验证”；**
- **尚未完成独立第三方合约审计；**
- 不代表 Flap、币安或币安广场对本项目的认可或背书。

在完成真实主网部署后，工厂地址、部署交易、编译器设置和 BscScan 源码验证链接会补充到本页。

## 金库结构

TipFromTrade 使用“一个固定工厂、每个代币一个独立金库”的结构。每次通过该工厂发射社区代币时，Flap VaultPortal 都会为该代币创建新的专属金库。

不同代币的余额和分账记录不会混在同一个金库地址中。公开看板可以按代币读取金库余额、结算事件和释放记录。

## 固定发行规则

- 报价与支付资产：原生 BNB
- 买入税：1%
- 卖出税：1%
- 进入金库比例：100%
- 创作者奖励：70%
- TFT 回购金库：20%
- 相关内容奖励：10%
- 达标门槛：0.1 BNB
- 达标结算：在当前五分钟周期结束时执行
- 闲置结算：连续 30 分钟没有新增金库收入后，在对应五分钟周期末检查并结算
- 平台额外抽成：0%

## BSC 主网固定依赖与收款地址

| 用途 | 地址 |
| --- | --- |
| 运营地址 | `0xe14cCCC958b241eE5cF047BD9d25147FC2b907dA` |
| 打赏托管地址（接收 70% 与 10%） | `0xa1aBBdE591c9a2D801D41E94d4183a2Ee06E24F1` |
| 统一 TFT 回购金库（接收 20%） | `0x5AeFFC634e43dd8766cD0E54eFEFb695DE59e9bf` |
| Flap VaultPortal | `0x90497450f2a706f1951b5bdda52B4E5d16f34C06` |
| Flap Trigger Service | `0xcf4EE25035CF883895110f367F5BA8172416a7F9` |

这些地址会写入工厂构造参数。工厂部署后不能修改；部署前必须再次核对控制权和官方依赖地址。

## 可公开核对的数据

每个金库公开保存或提供以下状态：

- 已识别但尚未分配的 BNB；
- 待释放的创作者奖励；
- 待释放的内容奖励；
- 下一五分钟周期边界；
- 30 分钟闲置结算边界；
- 当前 Flap 触发任务编号与状态；
- 代币地址、发射者、币安广场主页、运营地址和三个固定依赖地址。

主要链上事件包括：

- `RevenueRecognized`：识别新增 BNB；
- `CycleSettled`：记录本轮总额和 70/20/10 三项金额；
- `BuybackTreasuryFunded`：记录 20% 回购金库入账；
- `CustodyTransfer`：记录 70% 或 10% 份额转入固定托管地址；
- `SettlementTriggerScheduled` 与 `SettlementTriggerConsumed`：记录 Flap 定时任务；
- `EmergencyWithdrawNative` 与 `EmergencyWithdrawToken`：记录紧急恢复操作。

币安广场打赏和 TFT 回购销毁属于后续链下/链上操作，必须分别提供付款凭证和交易哈希。金库完成分账不等于打赏或销毁已经完成。

## 权限和风险边界

- 运营地址只能把已经结算的 70% 或 10% 份额释放到固定托管地址，不能在释放时更换收款地址。
- 20% 在结算时直接发送到固定回购金库。
- Flap Guardian 保留紧急恢复原生 BNB 和误入 ERC-20 的能力。该操作会产生公开事件，但属于重要管理权限。
- 安排自动结算前，需要有人调用 `scheduleSettlement` 并支付当时的 Flap Trigger Service 费用；已验证的是“安排后由官方触发服务自动回调”，不是无人参与、永久免费运行。
- 合约按金库余额增量识别收入，无法仅凭合约区分 Flap 下发的交易税与他人直接转入的 BNB。公开报表应结合 Flap 下发交易和金库事件进行对账。
- BNB 转入托管地址后，币安广场打赏属于运营流程。链上金库无法证明币安广场收款人已经收到打赏。
- 主网分叉测试、公共测试网测试和代码审查都不能代替独立第三方审计。

## 固定构建标识

| 项目 | 值 |
| --- | --- |
| 合约 | `TipFromTradeVaultFactoryV25` |
| Solidity | `0.8.26` |
| EVM | `cancun` |
| Optimizer runs | `200` |
| 创建字节码 Keccak-256 | `0x928844c34cc8a9236a05573096cba59a3e814f0c65def0d27d9c9dbcff62af52` |
| 运行时代码模板 Keccak-256 | `0x936986a482b02168e89b45c559d4b2989a0f47074822692d9c33f331bbce6aef` |
| 候选源码 SHA-256 | `fd7dab0f46fc35e54b34d1511bee8a3a32650e0920f3793d1256785658e9766e` |

运行时代码包含不可变量；正式部署后仍应通过构造参数读取、BscScan 源码验证和链上字节码共同核对。

## 已完成验证

### BSC 公共测试网

v2.5 候选版本通过 Flap 测试网 VaultPortal 发射代币，并由 Flap 官方测试网 Trigger Service 自动执行结算回调：

- 工厂：`0x893b67daad953738f7e431e199570cb1ac18ccac`
- 代币：`0x4B345b50bcFEd73f19eBfA337924dfc74Ee87777`
- 金库：`0xE1f7F94fB88A3FeF886a16Ad28705666855087a6`
- [工厂部署](https://testnet.bscscan.com/tx/0x44955377fbfa2baef6316062e966ce3d6d7f4819ce907f50689cd14c97015c52)
- [Flap 发射](https://testnet.bscscan.com/tx/0x3a28a6d85f730b46336b3c7a957a709ae012d024c0618401cb884968217c0546)
- [0.1 tBNB 入金](https://testnet.bscscan.com/tx/0xc40805908c53eb6d01be14aa04d4a561f58cf87d9f374a3fc0ca6be328afe521)
- [安排触发任务](https://testnet.bscscan.com/tx/0x383ee96e352ae18f4abae9d5609a933274aee18658f463020db1f483b74c0215)
- [官方服务自动回调](https://testnet.bscscan.com/tx/0x48bd7279d7afaebc49f18db3df1aee4c7ebf52824f77cd13f322fcbd97970937)

0.1 tBNB 最终记账为 0.07 tBNB 创作者奖励、0.02 tBNB 回购金库和 0.01 tBNB 内容奖励。

### 固定版本自动化测试

2026-09-25 使用 Foundry v1.8.3、Solidity 0.8.26 对固定候选版本执行：

- 9 个单元测试：全部通过；
- 1 个 BSC 主网分叉端到端测试：通过；
- 主网分叉覆盖 Flap 发射、独立金库创建、0.1 BNB 入账、官方 Trigger Service 调度与回调、70/20/10 分账和零未分配余额；
- TypeScript 检查与独立 Cloudflare 生产构建：通过。

这些结果证明固定候选版本在测试条件下按设计运行，不构成对未来漏洞或运营风险的保证。
