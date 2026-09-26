# v2.6 统一工厂审计候选说明

状态：**公共测试网验证完成，等待独立第三方审计；尚未部署主网统一工厂，
尚未获得 Flap 登记。**

## 为什么改成一个工厂

Flap 的未上架模板入口一次读取一个工厂地址。v2.6 因此用一个工厂暴露
两种配置，同时在链上保持规则隔离：

- 社区模式：普通用户可创建，固定 1% 买入税和 1% 卖出税，按 70% 创作者、
  20% TFT 回购金库、10% 相关内容奖励记账；
- 官方 TFT 模式：仅固定官方钱包可使用一次，固定按 80% 内容奖励、20%
  TFT 回购金库记账，不采用社区模式的 0.1 BNB 门槛、五分钟周期或
  30 分钟闲置规则；内容资金释放必须携带非零的 12 小时审核批次哈希。

两种模式使用独立 `UpgradeableBeacon`、实现合约和金库代理。工厂会为
代币与金库记录所选模式，两个 Beacon 也可以由 Flap Guardian 分别审核、
升级和最终锁定。

## 固定审计范围

编译条件：Solidity `0.8.26`、优化器 200 runs、Cancun EVM。

| 合约源文件 | SHA-256 |
| --- | --- |
| 统一工厂 | `f96299b4f4c16d59d5cc9e5eb897ad661f69b11318c1d20ced27d7de0947841f` |
| 社区金库实现 | `5257837e28fb3511539b791ab5432f0a67b7d41cf762d231a6cbf54cd725c76b` |
| 官方金库实现 | `5a98b4d3625fa83f86519f494bf88e60d6d36f43c2ee162445fe9bf6d9ded671` |

## BSC 公共测试网地址

| 组件 | 地址 |
| --- | --- |
| 统一工厂 | `0x48a97197f1aa74b1fe8bc965f906c93cd3ec5058` |
| 社区 Beacon | `0xc9b22153192F95a1675D7E314cbBFC76Be045E6d` |
| 社区实现 | `0x6FBFC241626DA1e50E2dedAD3d5048607F606a72` |
| 官方 Beacon | `0xDA2fe9077D480Af1D927baa48c7f420E01757aE6` |
| 官方实现 | `0x9c6a094edcE2E0f6a18d8D9Bccee806614bbB486` |
| 社区测试代币 | `0x89D482438557b771a4D5AAcE85db40cd8F0e7777` |
| 社区金库 | `0xfcDfC11862F62EfB43f2910357681eE888D106f9` |
| 官方测试代币 | `0x5B4e23556fd179F51e8579CE81e483a6fDbD7777` |
| 官方金库 | `0x7e8438c6a1139d1Ef37e8e608D26B736350A738B` |

## 链上证据

- [统一工厂部署](https://testnet.bscscan.com/tx/0xf3c106b3f02851c76f527ae72eea4ac2f6f8650e16aca8159562b2e079d72701)
- [官方模式通过 Flap 发射](https://testnet.bscscan.com/tx/0x71b9825f79ffbabc1a15e6f7ec48885963c0a596f41deddbd0d743700f410d6b)
- [官方 0.01 tBNB 入金](https://testnet.bscscan.com/tx/0x5c2b40939d64147de78f3be9b02485d747c4656b2a49b5ae8a6f3c36c71f9326)
- [官方 80/20 立即结算](https://testnet.bscscan.com/tx/0x19d7e3a6d628cdf15e932f852d41b60a2f709aa0719ea6155906a2814d758955)
- [官方审核批次哈希释放](https://testnet.bscscan.com/tx/0x5efc89ffdc23909da99925b345520fb1c6e951732c1953ad97dd8bfe0ba4daa6)
- [社区模式通过同一个工厂发射](https://testnet.bscscan.com/tx/0xd9b21dfac7d90b689b231ceadd0ad415c527ec2448d321349b9a8fa3694862c1)
- [社区 0.1 tBNB 入金](https://testnet.bscscan.com/tx/0xe9d020ed833a9d9caade8a945c6f35855279900636c1f68078a6dc282119b588)
- [社区 70/20/10 轮末结算](https://testnet.bscscan.com/tx/0xfc31df71a5b6e40746e064834eba7718ddf08fc30dc0cbd251f512a0725520d5)

测试结果：社区 0.1 tBNB 分为 0.07 / 0.02 / 0.01；官方 0.01 tBNB
立即分为 0.008 / 0.002。第二次官方发行被只读模拟拒绝。测试使用直接
存入金库的 tBNB，用于验证创建、代理绑定、时间与分账逻辑，不代表自然
交易税、币安广场打赏、TFT 市场回购或销毁已经执行。

## 正确发布顺序

1. 由独立第三方审计上述固定源码范围；
2. 修复被接受的问题并重新执行完整测试；
3. 部署一个新的 BSC 主网统一工厂并核对字节码与固定参数；
4. 向 Flap 提交这一工厂地址、两个 Beacon、两个实现合约、公共测试网
   证据和最终审计报告；
5. 等待 Flap 登记后再将其称为“已验证”。
