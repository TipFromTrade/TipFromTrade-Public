# TipFromTrade v2.5 自定义金库透明说明

更新日期：2026-09-25

## 当前状态

TipFromTrade v2.5 已按照 Flap 官方 `FlapVaultExample` 的
UpgradeableBeacon + BeaconProxy 架构重写，并保留本项目固定的 1% / 1%
税率和 70 / 20 / 10 分配规则。该候选版本已通过 12 项单元测试与 3 项
只读 BSC 主网分叉集成测试。

当前准确状态是：

- **官方示例架构适配和自动化验证已完成；**
- **v2.5 Beacon 版本尚未部署为正式 BSC 主网工厂；**
- **正在准备提交 Flap 安排最终第三方审计；**
- **第三方审计尚未完成，因此 Flap 页面仍会显示默认警告；**
- 不代表 Flap、币安或币安广场对本项目的认可或背书。

Flap V2 发币完全无许可，不需要工厂登记或白名单。项目目标是完成
Flap 安排的最终第三方审计，由 Flap 移除前台默认警告。

## 为什么使用官方示例架构

Flap 官方示例采用“一个工厂、一个实现合约、一个 Beacon、每个代币
一个 BeaconProxy 金库”的模式。TipFromTrade v2.5 采用同一协议骨架：

1. 工厂部署一个锁定初始化入口的金库实现合约；
2. 工厂部署一个由 Flap Guardian 控制的 UpgradeableBeacon；
3. 每次发币时，Flap VaultPortal 创建并初始化新的 BeaconProxy 金库；
4. 每个代币拥有独立地址、余额和分账记录；
5. 只有 Flap Guardian 可以升级 Beacon；
6. Guardian 可以通过放弃 Beacon 所有权永久关闭升级能力；
7. Proxy 金库不提供任意提走 BNB 或代币的紧急提款函数。

正式上线前，升级权限何时永久锁定，将结合第三方审计结果与 Flap
安全团队意见确定并公开。

## 固定发行与分账规则

- 报价与支付资产：原生 BNB
- 买入税：1%
- 卖出税：1%
- 进入对应代币金库：100%
- 创作者奖励：70%
- TFT 回购金库：20%
- 相关内容奖励：10%
- 达标门槛：0.1 BNB
- 达标结算：在当前五分钟周期结束时执行
- 闲置结算：连续 30 分钟没有新增金库收入后，在对应五分钟周期末检查并结算
- 平台额外抽成：0%

工厂在发币前校验这些参数。非 BNB 报价、非 Tax Token V3、非
1% / 1% 税率或非 100% 金库分配都会被拒绝。

## BSC 主网固定依赖与收款地址

| 用途 | 地址 |
| --- | --- |
| 运营地址 | `0xe14cCCC958b241eE5cF047BD9d25147FC2b907dA` |
| 打赏托管地址（接收 70% 与 10%） | `0xa1aBBdE591c9a2D801D41E94d4183a2Ee06E24F1` |
| 统一 TFT 回购金库（接收 20%） | `0x5AeFFC634e43dd8766cD0E54eFEFb695DE59e9bf` |
| Flap VaultPortal | `0x90497450f2a706f1951b5bdda52B4E5d16f34C06` |
| Flap Trigger Service | `0xcf4EE25035CF883895110f367F5BA8172416a7F9` |

运营、托管、回购和 Trigger Service 地址写入工厂且不能修改。Beacon
实现代码在永久锁定前只能由 Flap Guardian 升级，这项权限与资金收款
地址是两类不同的权限。

## 可公开核对的数据

每个代币金库均可公开读取：

- 已识别但尚未分配的 BNB；
- 待释放的创作者奖励；
- 待释放的内容奖励；
- 下一五分钟周期边界；
- 30 分钟闲置结算边界；
- 当前 Flap 触发任务编号与状态；
- 代币地址、发射者、币安广场主页和固定收款地址。

主要链上事件包括：

- `RevenueRecognized`：识别新增 BNB；
- `CycleSettled`：记录本轮总额和 70 / 20 / 10 三项金额；
- `BuybackTreasuryFunded`：记录 20% 回购金库入账；
- `CustodyTransfer`：记录 70% 或 10% 份额转入固定托管地址；
- `SettlementTriggerScheduled` 与 `SettlementTriggerConsumed`：记录 Flap 定时任务；
- `VaultCreated`：记录代币、独立 Proxy 金库和发射者。

合约没有 `EmergencyWithdrawNative` 或 `EmergencyWithdrawToken`
之类的任意紧急提款入口。

币安广场打赏和 TFT 回购销毁属于后续操作，必须分别提供付款凭证和
交易哈希。金库完成分账不等于打赏或销毁已经完成。

## 权限与风险边界

- 运营地址只能把已经结算的 70% 或 10% 份额释放到固定托管地址，
  不能临时更换收款人。
- 20% 在结算时直接发送到固定回购金库。
- Flap Guardian 可以释放已结算份额到同一个固定托管地址。
- Flap Guardian 在永久锁定前拥有 Beacon 实现升级权；永久锁定后
  任何人都不能再次升级。
- 自动结算需要先调用 `scheduleSettlement` 并支付 Flap Trigger
  Service 当时的费用。
- 合约按余额增量识别收入，无法只凭金库合约区分 Flap 下发的交易税
  与第三方直接转入的 BNB，公开报表需结合 TaxProcessor 与金库事件对账。
- 主网分叉测试、测试网测试和内部规则核对都不能替代独立第三方审计。

## 可复现构建标识

| 项目 | 值 |
| --- | --- |
| 合约 | `TipFromTradeVaultFactoryV25` |
| Solidity | `0.8.26` |
| EVM | `cancun` |
| Optimizer runs | `200` |
| 测试源码提交 | `fab4c9ad237b44f4f90dffc493236f543b9213d7` |
| 候选源码 SHA-256 | `5257837e28fb3511539b791ab5432f0a67b7d41cf762d231a6cbf54cd725c76b` |
| 工厂创建字节码 Keccak-256 | `0x47a350658d8e3e730349138c04667e40ec3f910862b35acfdabbb4091b2ea606` |
| 工厂运行时代码 Keccak-256 | `0x030bdc36bff62ad4ccf5b3469c0ec36db3875d191cdfef4ee19f3e5b101d0d8b` |

正式部署后还需公开并核对工厂、Beacon、实现合约与 Proxy 金库地址，
完成 BscScan 源码验证。

## 已完成的自动化验证

2026-09-25 的官方架构候选版本完成：

- 12 项单元测试：全部通过；
- 3 项只读 BSC 主网分叉集成测试：全部通过；
- 真实 Flap VaultPortal 发币和独立 Proxy 金库创建；
- TaxProcessor 的 marketAddress 与金库绑定；
- 联合曲线买入税进入 TaxProcessor 并下发金库；
- 代币毕业到 DEX 后卖出税收集、兑换并下发同一金库；
- 官方 Trigger Service 回调将 0.1 BNB 分为
  0.07 / 0.02 / 0.01 BNB；
- Trigger 身份、执行时间、重放防护和 2,000,000 Gas 上限；
- `receive()` 路径低于 1,000,000 Gas；
- Guardian 升级权限和不可逆升级锁；
- 固定托管地址、初始化锁、双语界面描述及无任意提款入口。

这些测试在本地主网分叉上执行，不向 BSC 主网广播交易。

## 历史公共测试网证据

重构为 Beacon 架构之前的 v2.5 候选版本曾完成 Flap BSC 测试网发射和
官方 Trigger Service 自动回调：

- 工厂：`0x893b67daad953738f7e431e199570cb1ac18ccac`
- 代币：`0x4B345b50bcFEd73f19eBfA337924dfc74Ee87777`
- 金库：`0xE1f7F94fB88A3FeF886a16Ad28705666855087a6`
- [工厂部署](https://testnet.bscscan.com/tx/0x44955377fbfa2baef6316062e966ce3d6d7f4819ce907f50689cd14c97015c52)
- [Flap 发射](https://testnet.bscscan.com/tx/0x3a28a6d85f730b46336b3c7a957a709ae012d024c0618401cb884968217c0546)
- [0.1 tBNB 入金](https://testnet.bscscan.com/tx/0xc40805908c53eb6d01be14aa04d4a561f58cf87d9f374a3fc0ca6be328afe521)
- [安排触发任务](https://testnet.bscscan.com/tx/0x383ee96e352ae18f4abae9d5609a933274aee18658f463020db1f483b74c0215)
- [官方服务自动回调](https://testnet.bscscan.com/tx/0x48bd7279d7afaebc49f18db3df1aee4c7ebf52824f77cd13f322fcbd97970937)

这组记录证明历史候选版本的 Flap 测试网接入与分账结果，不应被描述
为新 Beacon 版本的公共测试网部署证据。新版本正式部署前仍需完成
独立审计、Flap 登记和必要的测试网复核。
