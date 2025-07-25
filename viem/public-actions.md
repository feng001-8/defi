# Viem 公共操作（Public Actions）核心解析与全方法列表

## 核心定位

Viem 中的 **Public Actions** 是与以太坊区块链进行**只读交互**的标准化接口，直接映射底层 JSON-RPC 方法（如 `eth_blockNumber`、`eth_getBalance` 等），通过 ** 公共客户端（Public Client）** 调用，无需权限验证或签名操作，是获取区块链公开数据的核心手段。

## 核心特性

1. **只读性**
   所有操作均为查询，不修改链状态，无需消耗 Gas。
2. **无权限要求**
   任何应用均可调用，适合获取公开链上数据（如区块、交易、合约状态）。
3. **一一映射 RPC 方法**
   每个方法对应一个以太坊标准 RPC 接口，Viem 提供类型安全的封装。

## 全方法分类与说明

### 一、区块相关方法

| 方法名                     | 用途与 RPC 映射                                              | 示例代码                                                     |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `getBlockNumber`           | 获取最新区块号（对应 `eth_blockNumber`）                     | `const blockNumber = await publicClient.getBlockNumber()`    |
| `getBlock`                 | 获取区块详情（对应 `eth_getBlockByNumber` 或 `eth_getBlockByHash`） | `const block = await publicClient.getBlock({ hash: '0x...' })` |
| `getBlockTransactionCount` | 获取区块中的交易数量（对应 `eth_getBlockTransactionCountByNumber`） | `const count = await publicClient.getBlockTransactionCount({ number: 123 })` |
| `getBlobBaseFee`           | 获取 Blob 基础费用（对应 `eth_blobBaseFee`，需支持 EIP-4844 的链） | `const fee = await publicClient.getBlobBaseFee()`            |

### 二、交易相关方法

| 方法名                        | 用途与 RPC 映射                                      | 示例代码                                                     |
| ----------------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| `getTransaction`              | 获取交易详情（对应 `eth_getTransactionByHash`）      | `const tx = await publicClient.getTransaction({ hash: '0x...' })` |
| `getTransactionReceipt`       | 获取交易收据（对应 `eth_getTransactionReceipt`）     | `const receipt = await publicClient.getTransactionReceipt({ hash: '0x...' })` |
| `getTransactionCount`         | 获取账户的交易计数（对应 `eth_getTransactionCount`） | `const count = await publicClient.getTransactionCount({ address: '0x...' })` |
| `getTransactionConfirmations` | 获取交易确认数（基于当前区块号计算）                 | `const confirmations = await publicClient.getTransactionConfirmations({ hash: '0x...' })` |
| `estimateGas`                 | 估计交易所需的 Gas（对应 `eth_estimateGas`）         | `const gas = await publicClient.estimateGas({ to: '0x...', value: 1n })` |

### 三、账户相关方法

| 方法名         | 用途与 RPC 映射                                              | 示例代码                                                     |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `getBalance`   | 获取账户余额（对应 `eth_getBalance`）                        | `const balance = await publicClient.getBalance({ address: '0x...' })` |
| `getCode`      | 获取合约代码（对应 `eth_getCode`）                           | `const code = await publicClient.getCode({ address: '0x...' })` |
| `getStorageAt` | 获取账户存储位置的值（对应 `eth_getStorageAt`）              | `const value = await publicClient.getStorageAt({ address: '0x...', position: 0n })` |
| `getProof`     | 获取账户证明（包括余额、代码、存储等）（对应 `eth_getProof`） | `const proof = await publicClient.getProof({ address: '0x...' })` |

### 四、合约相关方法

| 方法名                | 用途与 RPC 映射                                           | 示例代码                                                     |
| --------------------- | --------------------------------------------------------- | ------------------------------------------------------------ |
| `readContract`        | 调用合约只读方法（对应 `eth_call`）                       | `const balance = await publicClient.readContract({ address: '0x...', abi: erc20Abi, functionName: 'balanceOf', args: ['0x...'] })` |
| `simulateContract`    | 模拟合约调用（对应 `eth_call`，用于验证逻辑但不实际执行） | `const result = await publicClient.simulateContract({ address: '0x...', abi: erc20Abi, functionName: 'transfer', args: ['0x...', 100n] })` |
| `estimateContractGas` | 估计合约方法执行所需的 Gas（对应 `eth_estimateGas`）      | `const gas = await publicClient.estimateContractGas({ address: '0x...', abi: erc20Abi, functionName: 'approve', args: ['0x...', 1000n] })` |

### 五、网络与配置方法

| 方法名          | 用途与 RPC 映射                            | 示例代码                                                     |
| --------------- | ------------------------------------------ | ------------------------------------------------------------ |
| `getChainId`    | 获取链 ID（对应 `eth_chainId`）            | `const chainId = await publicClient.getChainId()`            |
| `getGasPrice`   | 获取当前 Gas 价格（对应 `eth_gasPrice`）   | `const gasPrice = await publicClient.getGasPrice()`          |
| `getFeeHistory` | 获取 Gas 费用历史（对应 `eth_feeHistory`） | `const history = await publicClient.getFeeHistory({ oldestBlock: 123 })` |

### 六、事件与过滤器方法

| 方法名              | 用途与 RPC 映射                               | 示例代码                                                     |
| ------------------- | --------------------------------------------- | ------------------------------------------------------------ |
| `getLogs`           | 获取日志（对应 `eth_getLogs`）                | `const logs = await publicClient.getLogs({ address: '0x...', topics: ['0x...'] })` |
| `createBlockFilter` | 创建区块过滤器（对应 `eth_newBlockFilter`）   | `const filterId = await publicClient.createBlockFilter()`    |
| `createEventFilter` | 创建事件过滤器（对应 `eth_newFilter`）        | `const filterId = await publicClient.createEventFilter({ address: '0x...', topics: ['0x...'] })` |
| `getFilterChanges`  | 获取过滤器变化（对应 `eth_getFilterChanges`） | `const changes = await publicClient.getFilterChanges({ filterId: '0x...' })` |
| `uninstallFilter`   | 卸载过滤器（对应 `eth_uninstallFilter`）      | `const success = await publicClient.uninstallFilter({ filterId: '0x...' })` |

### 七、高级与批量操作

| 方法名           | 用途与 RPC 映射                               | 示例代码                                                     |
| ---------------- | --------------------------------------------- | ------------------------------------------------------------ |
| `multicall`      | 批量执行多个 Public Actions（支持并行或串行） | `const results = await publicClient.multicall([{ method: 'getBlockNumber' }, { method: 'getBalance', args: [{ address: '0x...' }] }])` |
| `simulateCalls`  | 模拟多个调用（用于测试或验证批量操作）        | `const outcomes = await publicClient.simulateCalls([{ to: '0x...', value: 1n }, { to: '0x...', data: '0x...' }])` |
| `simulateBlocks` | 模拟多个区块的执行（用于链状态预测）          | `const state = await publicClient.simulateBlocks([{ transactions: [...] }])` |

## 核心价值

- **简化开发**：通过类型安全的封装避免手动处理 RPC 细节，降低开发门槛。
- **高效查询**：支持批量操作（如 `multicall`）和缓存机制（如 `getBlockNumber` 的 `cacheTime` 参数），提升性能。
- **灵活扩展**：可通过 `call` 方法直接发送任意 JSON-RPC 请求，满足自定义需求。