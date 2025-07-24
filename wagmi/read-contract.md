# Wagmi 读取合约数据（Read from Contract）核心指南

## 核心概念

Wagmi 的「读取合约数据」功能用于从区块链合约中获取数据（如代币余额、合约状态等），无需消耗 gas（因为是只读操作）。核心是通过钩子简化合约交互流程，统一不同区块链网络的读取逻辑。

核心目标：

- 简化合约数据读取的参数配置
- 处理与区块链节点的通信
- 提供数据缓存和状态管理（加载、成功、错误）
- 统一不同合约标准（如 ERC20、ERC721）的读取方式

## 核心钩子与使用流程

### 1. 必要前提

- 合约 ABI（Application Binary Interface，描述合约方法和数据结构）
- 合约地址（部署在区块链上的具体地址）
- 配置正确的区块链网络（与合约部署网络一致）

### 2. 核心钩子：`useReadContract`

用于读取合约数据的核心钩子，支持调用任何合约的只读方法：

```tsx
import { useReadContract } from 'wagmi'
import { ERC20_ABI } from './abi' // 导入 ERC20 合约 ABI

function TokenBalance() {
  const { data, isLoading, error } = useReadContract({
    address: '0x5FbDB2315678afecb367f032d93F642f64180aa3', // 合约地址
    abi: ERC20_ABI, // 合约 ABI
    functionName: 'balanceOf', // 要调用的合约方法名
    args: ['0x70997970c51812dc3a010c7d01b50e0d17dc79c8'], // 方法参数（如查询的钱包地址）
  })

  if (isLoading) return <div>加载中...</div>
  if (error) return <div>错误: {error.message}</div>
  
  // 数据处理（根据合约返回值类型处理，此处为余额）
  return <div>代币余额: {data?.toString()}</div>
}
```

#### `useReadContract` 核心属性：

- `data`：合约方法返回的数据（类型取决于合约方法定义）
- `isLoading`：是否正在加载数据
- `error`：读取失败时的错误信息
- `refetch`：手动重新获取数据的函数

#### 关键参数说明：

- `address`：合约部署的区块链地址（`0x` 开头）
- `abi`：合约的 ABI 数组（描述方法和参数）
- `functionName`：要调用的合约方法名称（如 `balanceOf`、`symbol`）
- `args`：方法所需的参数数组（顺序与合约方法定义一致）
- `chainId`：可选，指定读取的区块链网络 ID

### 3. 常见合约读取场景

#### （1）读取 ERC20 代币信息

```tsx
// 读取代币符号（如 "USDC"）
const { data: symbol } = useReadContract({
  address: '0x...',
  abi: ERC20_ABI,
  functionName: 'symbol', // 无参数方法
})

// 读取代币小数位（如 6、18）
const { data: decimals } = useReadContract({
  address: '0x...',
  abi: ERC20_ABI,
  functionName: 'decimals',
})
```

#### （2）读取 ERC721 NFT 信息

```tsx
import { ERC721_ABI } from './abi'

// 读取 NFT 拥有者
const { data: owner } = useReadContract({
  address: '0x...', // NFT 合约地址
  abi: ERC721_ABI,
  functionName: 'ownerOf',
  args: [123n], // NFT ID（注意使用 BigInt 类型）
})

// 读取 NFT 元数据 URI
const { data: tokenUri } = useReadContract({
  address: '0x...',
  abi: ERC721_ABI,
  functionName: 'tokenURI',
  args: [123n],
})
```

## 完整读取流程

1. **准备合约信息**
   获取目标合约的 ABI 和部署地址，确认合约所在的区块链网络
2. **配置读取参数**
   指定要调用的方法名（`functionName`）和所需参数（`args`）
3. **调用钩子读取数据**
   通过 `useReadContract` 发起读取请求，自动处理与区块链节点的通信
4. **处理读取状态**
   - 加载中：`isLoading` 为 `true`，显示加载提示
   - 成功：`data` 包含返回结果，根据数据类型进行处理（如格式化余额）
   - 失败：`error` 包含错误信息，显示错误提示
5. **按需刷新数据**
   如需手动刷新，调用 `refetch` 方法重新读取

## 高级特性

### 1. 数据格式化

对合约返回的原始数据（通常为 BigInt）进行格式化：

```tsx
import { formatUnits } from 'viem'

// 读取代币余额并格式化
const { data: balance } = useReadContract({
  address: '0x...',
  abi: ERC20_ABI,
  functionName: 'balanceOf',
  args: ['0x...'],
})

// 假设代币小数位为 18，转换为可读格式
const formattedBalance = balance ? formatUnits(balance, 18) : '0'
```

### 2. 条件性读取

通过 `enabled` 参数控制是否发起读取请求：

```tsx
const { address } = useAccount() // 获取当前连接的钱包地址

const { data } = useReadContract({
  address: '0x...',
  abi: ERC20_ABI,
  functionName: 'balanceOf',
  args: [address],
  // 只有当地址有效时才发起读取
  enabled: !!address && isAddress(address),
})
```

### 3. 批量读取

同时读取多个合约方法（使用多个 `useReadContract` 钩子）：

```tsx
// 同时读取代币余额、符号和小数位
function TokenInfo() {
  const tokenAddress = '0x...'
  const userAddress = '0x...'

  const { data: balance } = useReadContract({/* 读取 balanceOf */})
  const { data: symbol } = useReadContract({/* 读取 symbol */})
  const { data: decimals } = useReadContract({/* 读取 decimals */})

  return (
    <div>
      <p>代币: {symbol}</p>
      <p>余额: {balance && formatUnits(balance, decimals)}</p>
    </div>
  )
}
```

## 核心总结

Wagmi 读取合约数据的核心是 `useReadContract` 钩子，它封装了以下关键能力：

- 通过合约 ABI 和地址调用任意只读方法
- 自动处理区块链节点通信和数据解析
- 提供完整的状态管理（加载、成功、错误）
- 支持条件读取和手动刷新