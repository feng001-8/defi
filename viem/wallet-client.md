# Viem 钱包客户端（Wallet Client）核心解析

## 核心定位

Viem 的 **Wallet Client** 是用于与以太坊账户交互的核心工具，专注于**区块链状态修改操作**（如发送交易、签名消息、调用合约写方法等）。它支持两种账户类型：**JSON-RPC 账户**（如 MetaMask 等浏览器钱包）和**本地账户**（如私钥、助记词管理的账户），是处理 “写操作” 的核心组件。

## 核心功能

1. **账户管理**
   - 支持获取账户地址（`getAddresses`）、请求账户授权（`requestAddresses`）。
   - 兼容 JSON-RPC 账户（依赖外部钱包签名）和本地账户（直接用私钥签名）。
2. **交易与签名**
   - 发送原生代币转账（`sendTransaction`）。
   - 调用合约写方法（`writeContract`），支持 ABI 编码和签名。
   - 签名消息（`signMessage`）、签名交易（`signTransaction`）等安全操作。
3. **与公共客户端集成**
   可扩展公共客户端（Public Client）的功能（如 `publicActions`），同时处理读写操作，避免重复配置。

## 核心配置与使用

### 1. 初始化钱包客户端

通过 `createWalletClient` 创建实例，需指定**传输层（Transport）**，可选配**账户（Account）** 和**区块链网络（Chain）**。

#### （1）对接 JSON-RPC 账户（如 MetaMask）

```typescript
import { createWalletClient, custom } from 'viem'
import { mainnet } from 'viem/chains'

// 连接 MetaMask 等浏览器钱包
const walletClient = createWalletClient({
  chain: mainnet, // 目标网络
  transport: custom(window.ethereum!), // 使用钱包提供的 provider
})

// 获取钱包地址（需用户授权）
const [address] = await walletClient.requestAddresses()
```

#### （2）使用本地账户（如私钥）

```typescript
import { createWalletClient, http } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'
import { mainnet } from 'viem/chains'

// 从私钥创建本地账户
const account = privateKeyToAccount('0x你的私钥...')

// 初始化钱包客户端
const walletClient = createWalletClient({
  account, // 挂载本地账户（可选，避免每次操作传账户）
  chain: mainnet,
  transport: http('https://mainnet.infura.io/v3/你的API密钥'), // 直接连接节点
})
```

### 2. 核心 API 示例

#### （1）发送原生代币转账

```typescript
import { parseEther } from 'viem'

// 发送 0.001 ETH 到目标地址
const hash = await walletClient.sendTransaction({
  account: address, // 发送账户（JSON-RPC 账户需显式传入）
  to: '0xa5cc3c03994DB5b0d9A5eEdD10CabaB0813678AC', // 接收地址
  value: parseEther('0.001'), // 金额（转换为 wei）
})
```

#### （2）调用合约写方法

```typescript
// 调用 ERC20 合约的 transfer 方法
const hash = await walletClient.writeContract({
  address: '0x合约地址', // 合约地址
  abi: erc20Abi, // 合约 ABI
  functionName: 'transfer', // 方法名
  args: ['0x接收地址', parseEther('1')], // 方法参数（接收地址和金额）
  account: address, // 签名账户
})
```

#### （3）扩展公共客户端功能

```typescript
import { createWalletClient, http, publicActions } from 'viem'

// 扩展公共客户端功能，同时处理读写操作
const walletClient = createWalletClient({
  account,
  chain: mainnet,
  transport: http(),
}).extend(publicActions) // 扩展公共操作

// 先用公共客户端模拟交易（读操作）
const { request } = await walletClient.simulateContract({
  address: '0x合约地址',
  abi: erc20Abi,
  functionName: 'transfer',
  args: ['0x接收地址', 100n],
})

// 再用钱包客户端发送交易（写操作）
const hash = await walletClient.writeContract(request)
```

## 关键参数说明

| 参数              | 类型                | 说明                                                         |
| ----------------- | ------------------- | ------------------------------------------------------------ |
| `account`         | `Account | Address` | 默认账户，用于签名和发送交易，支持 JSON-RPC 账户或本地账户。 |
| `chain`           | `Chain`             | 目标区块链网络，用于校验交易网络是否匹配钱包当前网络。       |
| `transport`       | `Transport`         | 传输层，JSON-RPC 账户用 `custom(window.ethereum)`，本地账户用 `http()`。 |
| `pollingInterval` | `number`            | 轮询频率（毫秒），用于监听交易状态等实时更新（默认 4000ms）。 |
| `cacheTime`       | `number`            | 缓存时长（毫秒），减少重复请求（默认与 `pollingInterval` 一致）。 |

## 核心价值

- **统一接口**：无论对接外部钱包还是本地账户，均通过相同 API 处理写操作。
- **安全签名**：JSON-RPC 账户的签名在外部钱包完成，本地账户支持私钥安全管理。
- **功能扩展**：可集成公共客户端功能，一站式处理读写操作，简化代码。