- Viem 钱包操作（Wallet Actions）核心与全方法解析

  ## 核心定位

  Viem 的 **Wallet Actions** 是与以太坊账户进行**状态修改交互**的标准化接口，直接映射底层 “钱包相关” JSON-RPC 方法（如 `eth_sendTransaction`、`eth_sign` 等），需通过钱包客户端（Wallet Client）调用。其核心是**处理需要用户授权和私钥签名的操作**，实现区块链状态修改（如转账、合约交互）和安全验证（如消息签名），是连接用户账户与区块链的 “写操作” 入口。

  ## 全方法分类与详细说明

  ### 一、账户与权限管理

  | 方法名               | 核心作用                                         | 底层 RPC 映射               | 示例代码                                                     |
  | -------------------- | ------------------------------------------------ | --------------------------- | ------------------------------------------------------------ |
  | `getAddresses`       | 获取钱包中已授权的账户地址列表（只读）           | `eth_accounts`              | `const addresses = await walletClient.getAddresses()`        |
  | `requestAddresses`   | 请求用户授权访问账户（首次连接钱包时触发）       | `eth_requestAccounts`       | `const addresses = await walletClient.requestAddresses()`    |
  | `watchAddresses`     | 监听账户地址变化（如切换钱包账户），触发回调通知 | 自定义事件监听              | `const unwatch = walletClient.watchAddresses({ onAddresses: (addrs) => console.log(addrs) })` |
  | `getPermissions`     | 查询当前钱包已授予的权限（如交易权限、签名权限） | `wallet_getPermissions`     | `const permissions = await walletClient.getPermissions()`    |
  | `requestPermissions` | 向用户请求特定权限（如允许发送交易、签名消息）   | `wallet_requestPermissions` | `await walletClient.requestPermissions({ permissions: [{ eth_accounts: {} }] })` |

  ### 二、交易发送与签名

  | 方法名                      | 核心作用                                                     | 底层 RPC 映射                     | 示例代码                                                     |
  | --------------------------- | ------------------------------------------------------------ | --------------------------------- | ------------------------------------------------------------ |
  | `sendTransaction`           | 发送完整交易（包含签名和广播流程），最常用的转账 / 合约交互入口 | `eth_sendTransaction`             | `typescript const hash = await walletClient.sendTransaction({ account: '0x...', to: '0x...', value: parseEther('0.1') })` |
  | `signTransaction`           | 仅对交易签名（不广播），返回签名后的交易数据（支持离线签名场景） | `eth_signTransaction`             | `typescript const signedTx = await walletClient.signTransaction({ account: '0x...', to: '0x...', value: 100n })` |
  | `sendRawTransaction`        | 发送已签名的原始交易（需提前用 `signTransaction` 生成）      | `eth_sendRawTransaction`          | `typescript const hash = await walletClient.sendRawTransaction({ serializedTransaction: signedTx })` |
  | `prepareTransactionRequest` | 预构造交易参数（自动填充 nonce、gas 价格等），简化交易发送流程 | 组合 `eth_getTransactionCount` 等 | `typescript const request = await walletClient.prepareTransactionRequest({ to: '0x...', value: 100n })` |

  ### 三、合约交互

  | 方法名                | 核心作用                                                     | 底层逻辑                        | 示例代码                                                     |
  | --------------------- | ------------------------------------------------------------ | ------------------------------- | ------------------------------------------------------------ |
  | `writeContract`       | 调用合约写方法（修改链状态），自动封装签名和交易发送流程（最常用） | 基于 `eth_sendTransaction` 封装 | `typescript const hash = await walletClient.writeContract({ address: '0x合约地址', abi: erc20Abi, functionName: 'transfer', args: ['0x接收地址', 100n], account: '0x...' })` |
  | `simulateContract`    | 模拟合约调用（不实际发送交易），用于验证参数和执行逻辑（预检查） | 基于 `eth_call` 封装            | `typescript const { request } = await walletClient.simulateContract({ address: '0x合约地址', abi: erc20Abi, functionName: 'transfer', args: ['0x...', 100n] })` |
  | `estimateContractGas` | 估算合约方法执行所需的 Gas 用量（辅助交易参数配置）          | 基于 `eth_estimateGas` 封装     | `typescript const gas = await walletClient.estimateContractGas({ address: '0x合约地址', abi: erc20Abi, functionName: 'approve', args: ['0x...', 1000n] })` |

  ### 四、消息与数据签名

  | 方法名          | 核心作用                                                | 底层 RPC 映射                | 示例代码                                                     |
  | --------------- | ------------------------------------------------------- | ---------------------------- | ------------------------------------------------------------ |
  | `signMessage`   | 对普通文本消息签名（用于身份验证、授权证明等）          | `eth_sign` / `personal_sign` | `typescript const signature = await walletClient.signMessage({ account: '0x...', message: '请验证我的身份' })` |
  | `signTypedData` | 对结构化数据签名（符合 EIP-712 标准，支持复杂数据验证） | `eth_signTypedData_v4`       | `typescript const signature = await walletClient.signTypedData({ account: '0x...', domain: { name: 'MyDApp' }, types: { Message: [{ name: 'content', type: 'string' }] }, message: { content: 'Hello' } })` |
  | `verifyMessage` | 验证消息签名的有效性（通过签名反推签名者地址）          | 本地密码学验证               | `typescript const address = await walletClient.verifyMessage({ message: 'Hello', signature: '0x...' })` |

  ### 五、网络与资产管理

  | 方法名        | 核心作用                                                     | 底层 RPC 映射                | 示例代码                                                     |
  | ------------- | ------------------------------------------------------------ | ---------------------------- | ------------------------------------------------------------ |
  | `switchChain` | 切换钱包连接的区块链网络（如从以太坊主网切换到 Sepolia 测试网） | `wallet_switchEthereumChain` | `typescript await walletClient.switchChain({ chainId: 1 }) // 切换到主网` |
  | `addChain`    | 向钱包添加新的区块链网络（如添加 Polygon、Arbitrum 等）      | `wallet_addEthereumChain`    | `typescript await walletClient.addChain({ chainId: 137, name: 'Polygon', rpcUrls: ['https://polygon.llamarpc.com'] })` |
  | `watchAsset`  | 让钱包追踪特定资产（如 ERC20 代币、NFT），显示在钱包资产列表中 | `wallet_watchAsset`          | `typescript await walletClient.watchAsset({ type: 'ERC20', options: { address: '0x代币地址', symbol: 'TOKEN' } })` |

  ### 六、批量交易（EIP-5792 相关）

  | 方法名               | 核心作用                                                     | 底层标准     | 示例代码                                                     |
  | -------------------- | ------------------------------------------------------------ | ------------ | ------------------------------------------------------------ |
  | `sendCalls`          | 发送批量交易（Call Bundle），在单一区块中按顺序执行多笔交易（原子操作） | EIP-5792     | `typescript const bundleId = await walletClient.sendCalls({ calls: [ { to: '0x...', data: '0x...' }, { to: '0x...', value: 100n } ], blockNumber: 123456n })` |
  | `getCallsStatus`     | 查询批量交易的执行状态（是否被区块包含、各交易结果等）       | EIP-5792     | `typescript const status = await walletClient.getCallsStatus({ bundleId: '0x...' })` |
  | `waitForCallsStatus` | 监听批量交易状态变化，直到达到预期状态（如 “已上链” 或 “失败”） | 基于轮询封装 | `typescript const finalStatus = await walletClient.waitForCallsStatus({ bundleId: '0x...' })` |
  | `getCapabilities`    | 查询钱包对 EIP-5792 等标准的支持能力（如最大批量交易数量）   | EIP-5792     | `typescript const capabilities = await walletClient.getCapabilities()` |

  ## 核心设计逻辑

  1. **用户授权优先**：所有涉及资产和账户的操作（如 `sendTransaction`、`signMessage`）必须经过用户显式授权（钱包弹窗确认），确保私钥安全。
  2. **状态修改为核心**：区别于 Public Actions 的 “只读” 特性，Wallet Actions 全部用于 “写操作”，直接导致区块链状态变更（如余额变化、合约存储更新）。
  3. **标准化与兼容性**：严格映射以太坊 JSON-RPC 标准和 EIP 规范（如 EIP-712、EIP-5792），适配主流钱包（MetaMask、Coinbase Wallet 等）和区块链网络。