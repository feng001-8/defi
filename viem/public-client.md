# Viem 公共客户端（Public Client）核心解析

## 核心定位

Viem 的 **Public Client** 是专门用于与区块链进行**只读交互**的核心工具，封装了区块链节点的公共 JSON-RPC 接口（如查询区块、交易、合约数据等），无需涉及钱包签名或状态修改操作。它是连接前端应用与区块链节点的桥梁，抽象了底层 RPC 通信细节，提供简洁的 API 接口。

## 核心功能

1. **区块链数据查询**
   提供一系列 “公共操作（Public Actions）”，支持查询链上基础数据：
   - 区块信息（`getBlock`、`getBlockNumber`）
   - 交易详情（`getTransaction`、`getTransactionReceipt`）
   - 账户数据（`getBalance`、`getCode`）
   - 合约只读方法调用（`readContract`）
2. **批量请求优化**
   支持 `eth_call` 聚合（通过 Multicall 合约），将多个合约读取请求合并为一个 RPC 调用，大幅减少网络请求次数，降低 RPC 服务的计算单元（CU）消耗。
3. **数据缓存与 polling**
   内置缓存机制（可通过 `cacheTime` 配置），减少重复请求；支持定时轮询（`pollingInterval`），实时获取链上数据更新。

## 核心配置与使用

### 1. 初始化公共客户端

通过 `createPublicClient` 创建实例，需指定**传输层（Transport）** 和**区块链网络（Chain）**：

```typescript
import { createPublicClient, http } from 'viem'
import { mainnet } from 'viem/chains'

// 连接以太坊主网的公共客户端
const publicClient = createPublicClient({
  chain: mainnet, // 目标区块链网络（内置主流网络配置）
  transport: http('https://mainnet.infura.io/v3/YOUR_API_KEY'), // 数据传输方式（HTTP/WS）
})
```

#### 关键配置项：

- **`transport`**（必选）：指定与区块链节点通信的方式，支持 `http()`、`webSocket()` 等，负责底层 RPC 请求发送。

- **`chain`**（可选）：指定区块链网络（如 `mainnet`、`sepolia`），内置网络信息（链 ID、节点 URL 等）。

- `batch.multicall`（可选）：启用批量请求优化（默认false），合并多个 `readContract`调用为一个请求。

  ```typescript
  // 启用批量请求
  const publicClient = createPublicClient({
    chain: mainnet,
    transport: http(),
    batch: { multicall: true }, // 开启聚合
  })
  ```

### 2. 核心 API 示例

#### （1）查询区块与交易

```typescript
// 获取最新区块号
const blockNumber = await publicClient.getBlockNumber()

// 获取交易详情
const transaction = await publicClient.getTransaction({
  hash: '0x...', // 交易哈希
})
```

#### （2）读取合约数据

```typescript
// 调用 ERC20 合约的 read 方法
const balance = await publicClient.readContract({
  address: '0x...', // 合约地址
  abi: erc20Abi, // 合约 ABI
  functionName: 'balanceOf', // 方法名
  args: ['0x...'], // 方法参数（查询地址）
})
```

#### （3）批量读取合约数据（启用 `multicall` 后）

```typescript
// 同时调用多个合约方法，仅发送一个 RPC 请求
const [name, symbol, totalSupply] = await Promise.all([
  publicClient.readContract({ address, abi, functionName: 'name' }),
  publicClient.readContract({ address, abi, functionName: 'symbol' }),
  publicClient.readContract({ address, abi, functionName: 'totalSupply' }),
])
```

## 高级特性

### 1. 批量请求优化（Multicall）

- **原理**：将多个 `eth_call` 请求打包为一个 `aggregate3` 调用，通过 Multicall 合约执行，减少网络往返。

- 配置

  ```typescript
  const publicClient = createPublicClient({
    batch: {
      multicall: {
        wait: 16, // 等待 16ms 再发送批量请求，收集更多调用
        batchSize: 512, // 每个批量请求的最大字节数
      },
    },
    // ...其他配置
  })
  ```

### 2. 数据缓存

- 通过`cacheTime`配置缓存时长（默认与`pollingInterval`一致），避免重复请求相同数据：

  ```typescript
  const publicClient = createPublicClient({
    cacheTime: 60_000, // 缓存 1 分钟
    // ...其他配置
  })
  ```

### 3. 自定义 RPC 方法

通过 `rpcSchema` 扩展自定义 JSON-RPC 方法，支持类型安全调用：

```typescript
import { rpcSchema } from 'viem'

// 定义自定义 RPC 方法类型
type CustomRpcSchema = [{
  Method: 'eth_customMethod',
  Parameters: [string],
  ReturnType: string
}]

// 注册自定义方法
const publicClient = createPublicClient({
  rpcSchema: rpcSchema<CustomRpcSchema>(),
  // ...其他配置
})

// 调用自定义方法
const result = await publicClient.request({
  method: 'eth_customMethod',
  params: ['param1'],
})
```

## 核心价值

- **简化只读交互**：封装底层 JSON-RPC 细节，提供直观的 API（如 `readContract`、`getBalance`）。
- **性能优化**：批量请求减少网络开销，缓存机制降低重复请求。
- **多网络支持**：内置主流区块链网络配置，切换网络只需修改 `chain` 参数。
- **类型安全**：基于 TypeScript 设计，自动推断参数和返回值类型，减少开发错误。