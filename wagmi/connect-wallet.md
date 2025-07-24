# Wagmi 连接钱包（Connect Wallet）核心指南

## 目录

- 核心概念
- 初始化配置
- 核心钩子解析
- 完整连接流程
- 高级特性

## 核心概念

Wagmi 的「连接钱包」功能通过一系列钩子（hooks）实现，旨在简化多种钱包（如 MetaMask、WalletConnect 等）与 DApp 的集成过程，统一不同钱包的交互方式，让开发者无需关注底层实现差异。

核心目标：

- 提供一致的钱包连接 API
- 支持多种主流钱包
- 简化连接状态管理
- 处理连接生命周期（连接、断开、自动重连）

## 初始化配置

使用 Wagmi 连接钱包前，需要先配置客户端和钱包连接器，并通过 WagmiProvider 包裹应用：

```tsx
import { WagmiProvider, createConfig, configureChains } from 'wagmi'
import { publicProvider } from 'wagmi/providers/public'
import { mainnet, sepolia } from 'wagmi/chains'
import { metaMask, walletConnect } from 'wagmi/connectors'

// 配置区块链网络
const { chains, publicClient, webSocketPublicClient } = configureChains(
  [mainnet, sepolia], // 支持的网络列表
  [publicProvider()], // 节点提供者
)

// 创建 Wagmi 配置
const config = createConfig({
  autoConnect: true, // 自动连接上次使用的钱包
  connectors: [
    metaMask(), // MetaMask 连接器
    walletConnect({ projectId: 'YOUR_PROJECT_ID' }), // WalletConnect 连接器
  ],
  publicClient,
  webSocketPublicClient,
})

// 应用入口
function App() {
  return (
    <WagmiProvider config={config}>
      {/* 应用内容 */}
    </WagmiProvider>
  )
}
```

## 核心钩子解析

### 1. useConnect - 处理钱包连接

用于触发钱包连接，获取可用钱包列表和连接状态：

```tsx
import { useConnect } from 'wagmi'

function ConnectButton() {
  const { connect, connectors, isPending } = useConnect()
  
  return (
    <div>
      {connectors.map((connector) => (
        <button
          key={connector.id}
          onClick={() => connect({ connector })} // 连接指定钱包
          disabled={isPending} // 连接中禁用按钮
        >
          {isPending ? '连接中...' : `连接 ${connector.name}`}
        </button>
      ))}
    </div>
  )
}
```

核心属性：

- connectors：可用钱包连接器数组（包含钱包 ID、名称等信息）
- connect：连接钱包的函数（需传入连接器对象）
- isPending：连接状态（布尔值，标识是否正在连接）
- error：连接错误信息（连接失败时可用）

### 2. useAccount - 获取账户信息

用于获取当前已连接的钱包信息和状态：

```tsx
import { useAccount } from 'wagmi'

function AccountInfo() {
  const { address, isConnected, isConnecting, chainId } = useAccount()
  
  if (isConnecting) return <div>连接中...</div>
  if (!isConnected) return <div>未连接钱包</div>
  
  return (
    <div>
      <p>地址：{address}</p>
      <p>当前网络：{chainId}</p>
    </div>
  )
}
```

核心属性：

- address：钱包地址（0x 开头的字符串，连接后可用）
- isConnected：是否已成功连接（布尔值）
- isConnecting：是否处于连接过程中（布尔值）
- chainId：当前连接的区块链网络 ID

### 3. useDisconnect - 断开连接

用于断开当前已连接的钱包：

```tsx
import { useDisconnect } from 'wagmi'

function DisconnectButton() {
  const { disconnect, isPending } = useDisconnect()
  
  return (
    <button onClick={disconnect} disabled={isPending}>
      {isPending ? '断开中...' : '断开连接'}
    </button>
  )
}
```

核心属性：

- disconnect：断开连接的函数
- isPending：断开过程状态（布尔值）

## 完整连接流程

1. **显示连接按钮**
   通过 useConnect 获取可用钱包列表，为每个钱包渲染连接按钮
2. **用户触发连接**
   点击按钮后调用 connect ({connector})，触发钱包连接流程（如 MetaMask 弹窗授权）
3. **获取连接状态**
   通过 useAccount 实时获取连接状态：
   - 连接中：isConnecting = true
   - 连接成功：isConnected = true 并获取 address
   - 连接失败：error 包含错误信息
4. **断开连接**
   用户点击断开按钮时，调用 useDisconnect 的 disconnect 方法

## 高级特性

### 1. 自动连接

配置 autoConnect: true 后，用户下次打开 DApp 时会自动连接上次使用的钱包：

```tsx
const config = createConfig({
  autoConnect: true, // 启用自动连接
  // ...其他配置
})
```

### 2. 连接事件监听

通过 useConnect 的回调函数处理连接成功 / 失败事件：

```tsx
const { connect } = useConnect({
  onSuccess: (data) => {
    console.log('连接成功', data.address)
    // 可执行跳转、显示提示等操作
  },
  onError: (error) => {
    console.error('连接失败', error.message)
  },
})
```

### 3. 过滤可用钱包

根据需求筛选显示的钱包：

```tsx
const { connectors } = useConnect()
// 只显示非 MetaMask 的钱包
const filteredConnectors = connectors.filter(connector => connector.id !== 'metaMask')
```

### 4. 指定连接网络

连接时指定钱包必须切换到特定网络：

```tsx
connect({
  connector,
  chainId: 11155111, // Sepolia 测试网
})
```

## 总结

Wagmi 连接钱包的核心是通过三个钩子实现完整的钱包生命周期管理：

- useConnect：管理连接触发和可用钱包列表
- useAccount：获取连接状态和账户信息
- useDisconnect：处理断开连接操作