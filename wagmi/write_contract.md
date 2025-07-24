# Wagmi 写入合约合约写入（Write to Contract）核心指南

## 核心概念

Wagmi 的「合约写入」功能用于向区块链合约发送写入交易（如转账代币、执行状态变更操作），这类操作会修改区块链状态并消耗 gas。核心是通过钩子简化合约写入的参数配置、签名流程和状态跟踪，统一不同钱包的交互逻辑。

核心目标：

- 简化合约写入交易的参数配置
- 处理交易签名、上链和状态跟踪
- 统一不同合约标准的写入操作
- 提供完整的交易生命周期管理（发起、签名、确认、失败）

## 核心钩子与使用流程

### 1. 必要前提

- 钱包已连接（通过 `useAccount` 确认 `isConnected` 为 `true`）
- 合约 ABI（描述合约可写入方法）
- 合约地址（部署在区块链上的地址）
- 足够的 gas 费用（用于支付交易成本）

### 2. 核心钩子：`useWriteContract`

用于向合约发送写入交易的核心钩子，处理交易创建、签名和上链：

```tsx
import { useWriteContract, useWaitForTransactionReceipt } from 'wagmi'
import { parseUnits } from 'viem'
import { ERC20_ABI } from './abi'

function TokenTransfer() {
  // 写入合约钩子
  const { data: hash, writeContract, isPending } = useWriteContract()
  
  // 等待交易确认钩子
  const { isLoading: isConfirming, isSuccess: isConfirmed } = useWaitForTransactionReceipt({
    hash,
  })

  const handleTransfer = () => {
    // 调用合约写入方法
    writeContract({
      address: '0x5FbDB2315678afecb367f032d93F642f64180aa3', // 合约地址
      abi: ERC20_ABI, // 合约ABI
      functionName: 'transfer', // 要调用的写入方法
      args: [
        '0x70997970c51812dc3a010c7d01b50e0d17dc79c8', // 接收地址
        parseUnits('100', 18), // 转账数量（根据代币小数位转换）
      ],
    })
  }

  return (
    <div>
      <button 
        onClick={handleTransfer} 
        disabled={isPending || isConfirming}
      >
        {isPending ? '签名中...' : 
         isConfirming ? '等待确认...' : '转账 100 代币'}
      </button>
      
      {hash && (
        <div>
          交易哈希: {hash}
          {isConfirmed && <p>转账成功！</p>}
        </div>
      )}
    </div>
  )
}
```

#### `useWriteContract` 核心属性：

- `writeContract`：触发合约写入的函数（需传入交易参数）
- `data: hash`：交易提交到网络后返回的交易哈希
- `isPending`：交易是否处于签名阶段（等待用户在钱包中确认）
- `error`：交易失败时的错误信息

#### 关键参数说明：

- `address`：合约部署地址（`0x` 开头）
- `abi`：合约的 ABI 数组
- `functionName`：要调用的合约写入方法（如 `transfer`、`approve`）
- `args`：方法所需的参数数组（顺序与合约定义一致）
- `value`：可选，发送的原生代币数量（用于需要支付 ETH 的合约方法）
- `gas`：可选，指定 gas 限制

### 3. 交易确认跟踪：`useWaitForTransactionReceipt`

与发送原生代币相同，用于跟踪合约写入交易的确认状态：

```tsx
const { 
  isLoading: isConfirming,  // 等待区块确认中
  isSuccess: isConfirmed,   // 交易已确认
  data: receipt             // 交易收据
} = useWaitForTransactionReceipt({
  hash,  // 来自 useWriteContract 的交易哈希
})
```

交易收据包含的关键信息：

- `status`：交易状态（1 成功，0 失败）
- `gasUsed`：实际消耗的 gas 数量
- `logs`：交易产生的事件日志（可用于验证合约状态变更）

## 完整写入流程

1. **准备合约信息**
   确定合约地址、ABI、要调用的方法名和参数
2. **触发写入交易**
   调用 `writeContract` 方法，钱包会弹出签名确认窗口（显示交易详情和 gas 费用）
3. **处理签名阶段**
   通过 `isPending` 状态显示 "签名中"，等待用户在钱包中确认
4. **跟踪确认状态**
   交易提交到网络后，使用 `useWaitForTransactionReceipt` 跟踪：
   - `isConfirming`：交易已提交但未被区块包含
   - `isConfirmed`：交易被区块确认（合约状态已更新）
5. **处理结果**
   - 成功：通过 `receipt` 验证交易状态，可读取事件日志确认合约变更
   - 失败：通过 `error` 获取失败原因（如用户拒绝、gas 不足等）

## 常见合约写入场景

### 1. ERC20 代币授权（Approve）

```tsx
// 授权给某个地址使用代币
writeContract({
  address: '0x...', // 代币合约地址
  abi: ERC20_ABI,
  functionName: 'approve',
  args: [
    '0x授权地址', // 被授权地址
    parseUnits('1000', 18), // 授权数量
  ],
})
```

### 2. 调用自定义合约方法

```tsx
// 调用自定义合约的 mint 方法铸造代币
writeContract({
  address: '0x...', // 自定义合约地址
  abi: CUSTOM_ABI, // 自定义合约ABI
  functionName: 'mint',
  args: [
    '0x接收地址', // 接收者
    10n, // 铸造数量（BigInt类型）
  ],
  value: parseEther('0.01'), // 如果铸造需要支付ETH
})
```

## 高级特性

### 1. 处理事件日志

通过交易收据中的 `logs` 解析合约事件，验证状态变更：

```tsx
import { decodeEventLog } from 'viem'

// 交易确认后解析事件
if (isConfirmed && receipt) {
  // 遍历所有日志
  for (const log of receipt.logs) {
    // 尝试解码 Transfer 事件
    try {
      const event = decodeEventLog({
        abi: ERC20_ABI,
        data: log.data,
        topics: log.topics,
      })
      if (event.eventName === 'Transfer') {
        console.log('转账成功:', event.args)
      }
    } catch (error) {
      continue
    }
  }
}
```

### 2. 自定义 gas 配置

手动设置 gas 限制或使用估算的 gas：

```tsx
import { useEstimateGas } from 'wagmi'

// 估算 gas
const { data: gas } = useEstimateGas({
  // 与 writeContract 相同的参数
  address: '0x...',
  abi: ERC20_ABI,
  functionName: 'transfer',
  args: ['0x...', parseUnits('100', 18)],
})

// 使用估算的 gas 发送交易
writeContract({
  // ...其他参数
  gas: gas ? gas + 10000n : undefined, // 增加一点缓冲
})
```

### 3. 错误处理

捕获常见的合约写入错误：

```tsx
const { writeContract, error } = useWriteContract()

// 显示错误信息
if (error) {
  if (error.message.includes('User rejected')) {
    return <div>用户已拒绝交易</div>
  } else if (error.message.includes('Insufficient funds')) {
    return <div>余额不足，无法支付 gas 费用</div>
  } else {
    return <div>交易失败: {error.message}</div>
  }
}
```

## 核心总结

Wagmi 合约写入的核心是通过两个钩子实现完整的交互流程：

- `useWriteContract`：创建合约写入交易、处理钱包签名、获取交易哈希
- `useWaitForTransactionReceipt`：跟踪交易确认状态、获取最终结果