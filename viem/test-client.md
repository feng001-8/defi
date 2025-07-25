# Viem 测试客户端（Test Client）核心解析

## 核心定位

Viem 的 **Test Client** 是专为**本地以太坊测试节点**（如 Anvil、Hardhat、Ganache）设计的交互工具，提供测试环境专属的高级操作能力，用于简化区块链应用的开发、调试和测试流程。它是连接测试节点与应用代码的桥梁，聚焦于 “测试场景构建” 和 “链上状态控制”。

## 核心功能

1. **测试节点专属操作（Test Actions）**
   提供直接控制本地测试节点的功能，支持灵活构建测试场景：
   - **区块控制**：手动挖矿（`mine`）、设置区块时间（`setNextBlockTimestamp`），加速交易确认流程。
   - **账户模拟**：临时接管任意账户（`impersonateAccount`），无需私钥即可操作目标账户，方便测试权限逻辑。
   - **状态修改**：直接修改账户余额（`setBalance`）、合约代码（`setCode`）、存储数据（`setStorageAt`），快速构建特定测试状态。
   - **费用配置**：自定义基础 Gas 费（`setNextBlockBaseFeePerGas`），测试不同费用策略下的交易行为。
2. **集成读写功能**
   可通过扩展（`extend`）集成公共客户端（`publicActions`）和钱包客户端（`walletActions`）的功能，在同一客户端中完成：
   - 读操作（如查询区块、合约数据）；
   - 写操作（如发送交易、调用合约）；
   - 测试操作（如挖矿、模拟账户），避免多客户端配置冗余。

## 核心配置与使用

### 1. 初始化测试客户端

通过 `createTestClient` 创建实例，必须指定**测试节点类型（mode）** 和**传输层（Transport）**，通常搭配本地测试链（如 Foundry）使用：

```typescript
import { createTestClient, http } from 'viem'
import { foundry } from 'viem/chains'

// 连接 Anvil 本地测试节点
const testClient = createTestClient({
  chain: foundry, // Foundry 本地链（默认适配 Anvil）
  mode: 'anvil', // 测试节点类型（支持 anvil、hardhat、ganache）
  transport: http('http://localhost:8545'), // 本地节点 RPC 地址
})
```

### 2. 核心 API 示例

#### （1）基础测试操作

```typescript
// 挖矿 1 个区块（立即确认交易）
await testClient.mine({ blocks: 1 })

// 模拟控制目标账户（无需私钥）
await testClient.impersonateAccount({ address: '0x70997970C51812dc3A010C7d01b50e0d17dc79C8' })

// 修改账户余额（充值 10 ETH）
await testClient.setBalance({
  address: '0x70997970C51812dc3A010C7d01b50e0d17dc79C8',
  value: parseEther('10'), // 转换为 wei
})

// 设置下一个区块的时间
await testClient.setNextBlockTimestamp({ timestamp: 1678900000 })
```

#### （2）扩展读写功能

```typescript
import { createTestClient, http, publicActions, walletActions } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'

// 扩展公共和钱包功能，一站式处理读写和测试操作
const testClient = createTestClient({
  chain: foundry,
  mode: 'anvil',
  transport: http(),
})
.extend(publicActions) // 增加读操作（如 getBlockNumber）
.extend(walletActions) // 增加写操作（如 sendTransaction）

// 读操作：查询最新区块号
const blockNumber = await testClient.getBlockNumber()

// 写操作：发送交易（使用本地账户）
const account = privateKeyToAccount('0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690')
const hash = await testClient.sendTransaction({
  account,
  to: '0x0000000000000000000000000000000000000000',
  value: parseEther('1'),
})

// 测试操作：挖矿确认交易
await testClient.mine({ blocks: 1 })
```

## 关键参数说明

| 参数              | 类型                        | 说明                                                         |
| ----------------- | --------------------------- | ------------------------------------------------------------ |
| `mode`            | `anvil | hardhat | ganache` | 测试节点类型（必填），指定适配的本地节点（如 Anvil）。       |
| `transport`       | `Transport`                 | 传输层（必填），指定本地测试节点的 RPC 地址（如 `http('http://localhost:8545')`）。 |
| `chain`           | `Chain`                     | 区块链网络（可选），通常使用 `foundry`（本地测试链）。       |
| `account`         | `Account | Address`         | 默认账户（可选），用于需要签名的操作（如发送交易）。         |
| `pollingInterval` | `number`                    | 轮询频率（毫秒，默认 4000），用于监听链上状态更新。          |