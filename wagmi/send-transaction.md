# Wagmi 发送交易（Send Transaction Transaction）核心指南

## 核心概念

Wagmi 的「发送交易」功能用于在区块链上执行转账（发送原生代币如 ETH）或调用合约方法，核心是通过钩子简化交易的创建、签名和上链流程，统一不同钱包的交易处理方式。

核心目标：

- 简化交易参数配置
- 处理交易签名和上链
- 提供交易状态跟踪（ pending /success/error）
- 统一不同钱包的交易交互逻辑

## 核心钩子与使用流程

### 1. 必要前提

- 钱包已连接（通过 `useAccount` 确认 `isConnected` 为 `true`）
- 配置正确的区块链网络（与钱包当前网络一致）

### 2. 核心钩子：`useSendTransaction`

用于创建和发送交易的核心钩子，处理交易的整个生命周期：

```tsx
import { useSendTransaction, useWaitForTransactionReceipt } from 'wagmi'
import { parseEther } from 'viem'

function SendTransaction() {
  // 发送交易钩子
  const { data: hash, sendTransaction, isPending } = useSendTransaction()
  
  // 等待交易确认钩子
  const { isLoading: isConfirming, isSuccess: isConfirmed } = useWaitForTransactionReceipt({
    hash, // 传入交易哈希
  })

  const handleSend = () => {
    // 调用发送交易方法
    sendTransaction({
      to: '0x70997970c51812dc3a010c7d01b50e0d17dc79c8', // 接收地址
      value: parseEther('0.001'), // 发送金额（转换为 wei）
    })
  }

  return (
    <div>
      <button 
        onClick={handleSend} 
        disabled={isPending || isConfirming}
      >
        {isPending ? '处理中...' : 
         isConfirming ? '等待确认...' : '发送 0.001 ETH'}
      </button>
      
      {hash && (
        <div>
          交易哈希: {hash}
          {isConfirmed && <p>交易已确认！</p>}
        </div>
      )}
    </div>
  )
}
```

#### `useSendTransaction` 核心属性：

- `sendTransaction`：触发交易的函数（需传入交易参数）
- `data: hash`：交易成功提交到网络后返回的交易哈希
- `isPending`：交易是否正在处理中（钱包签名阶段）
- `error`：交易失败时的错误信息

#### 交易参数说明：

- `to`：接收地址（`0x` 开头的字符串）
- `value`：发送的原生代币数量（需转换为最小单位 wei，使用 `parseEther` 处理）
- `gas`：可选，指定 gas 限制
- `gasPrice`：可选，指定 gas 价格
- `data`：可选，合约调用数据（用于执行合约方法）

### 3. 交易确认跟踪：`useWaitForTransactionReceipt`

用于跟踪交易是否被区块链确认，获取最终结果：

```tsx
const { 
  isLoading: isConfirming,  // 等待确认中
  isSuccess: isConfirmed,   // 交易已确认
  data: receipt             // 交易收据（包含区块号、gas 用量等）
} = useWaitForTransactionReceipt({
  hash,  // 交易哈希（来自 useSendTransaction 的 data）
})
```

交易收据（`receipt`）包含的关键信息：

- `status`：交易状态（1 成功，0 失败）
- `blockNumber`：交易所在区块号
- `gasUsed`：实际消耗的 gas
- `logs`：交易产生的日志（如代币转账记录）

## 完整交易流程

1. **准备交易参数**
   确定接收地址、发送金额等信息，注意单位转换（ETH → wei）
2. **触发交易**
   调用 `sendTransaction` 方法，钱包会弹出签名确认窗口
3. **处理签名阶段**
   通过 `isPending` 状态显示 "处理中"，等待用户在钱包中确认
4. **跟踪确认状态**
   交易提交到网络后，使用 `useWaitForTransactionReceipt` 跟踪确认状态：
   - `isConfirming`：交易已提交但未被区块包含
   - `isConfirmed`：交易被区块确认（成功完成）
5. **处理结果**
   - 成功：通过 `receipt` 获取交易详情
   - 失败：通过 `error` 获取失败原因

## 高级特性

### 1. 自定义 gas 设置

手动指定 gas 限制或价格（通常由钱包自动估算）：

```tsx
sendTransaction({
  to: '0x...',
  value: parseEther('0.001'),
  gas: 21000n, // 21000 单位 gas（原生转账固定值）
  gasPrice: parseGwei('30'), // 30 Gwei
})
```

### 2. 调用合约方法

通过 `data` 参数发送合约调用交易（非转账）：

```tsx
import { encodeFunctionData } from 'viem'
import { ERC20_ABI } from './abi'

// 编码转账方法调用数据
const data = encodeFunctionData({
  abi: ERC20_ABI,
  functionName: 'transfer',
  args: ['0x接收地址', parseUnits('100', 18)], // 转账 100 代币（18位小数）
})

// 发送合约交易（to 为合约地址，value 为 0）
sendTransaction({
  to: '0x合约地址',
  value: 0n,
  data,
})
```

### 3. 错误处理

捕获和处理交易可能出现的错误：

```tsx
const { sendTransaction, error } = useSendTransaction()

// 显示错误信息
if (error) {
  return <div>交易失败: {error.message}</div>
}
```

常见错误类型：

- 用户拒绝交易（User rejected the transaction）
- 余额不足（Insufficient funds）
- gas 不足（Out of gas）

## 核心总结

Wagmi 发送交易的核心是通过两个钩子实现完整的交易生命周期管理：

- `useSendTransaction`：创建交易、处理签名、获取交易哈希
- `useWaitForTransactionReceipt`：跟踪交易确认状态、获取最终结果