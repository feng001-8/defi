### 一、op-stack/client（Optimism 客户端工具）

#### 核心作用

提供与 **OP Stack（Optimism 技术栈）** 交互的客户端工具，简化以太坊 Layer 2（Optimism 等基于 OP Stack 的网络）与 Layer 1（以太坊主网）之间的跨链操作（如存款、提款、状态同步等），封装了 OP Stack 特有的通信逻辑（如跨链消息、输出根验证等）。

#### 全部关键方法

1. **createOPStackClient**
   - 作用：创建 OP Stack 客户端实例，整合 L1（以太坊主网）和 L2（Optimism 等）的 Provider，用于跨链交互。
   - 参数：`{ l1: PublicClient, l2: PublicClient }`（L1 和 L2 的公共客户端）。
   - 返回：OP Stack 客户端实例，包含所有跨链操作方法。
2. **deposit**
   - 作用：从 L1 向 L2 存款（将 ETH 或 ERC20 从以太坊主网转入 OP Stack L2 网络）。
   - 参数：
     - `args: { to: Address, value: bigint }`（接收地址、存款金额，ETH 场景）；
     - （ERC20 场景）额外需传入代币合约地址等参数。
   - 说明：触发 L1 的存款预编译合约，资金将通过跨链消息同步到 L2。
3. **withdraw**
   - 作用：从 L2 向 L1 发起提款（启动提款流程，需后续验证和最终化）。
   - 参数：`args: { to: Address, value: bigint }`（L1 接收地址、提款金额）。
   - 说明：在 L2 上触发提款交易，生成提款凭证，进入挑战期。
4. **getL2Output**
   - 作用：获取 L2 网络的输出根（Output Root），用于验证 L2 状态（提款时需验证该根）。
   - 参数：`args: { blockNumber: bigint }`（L2 区块号）。
   - 返回：输出根哈希，包含 L2 该区块的状态摘要。
5. **proveWithdrawal**
   - 作用：在 L1 上证明提款的有效性（基于 L2 输出根），完成提款的验证步骤。
   - 参数：`args: { withdrawal: Withdrawal, output: L2Output }`（提款对象、对应的 L2 输出根）。
   - 说明：需在 L2 输出根被提交到 L1 后调用，进入提款最终化阶段。
6. **finalizeWithdrawal**
   - 作用：在 L1 上最终化提款，将资金从 L1 的跨链合约转移到接收地址。
   - 参数：`args: { withdrawal: Withdrawal }`（提款对象）。
   - 说明：需在 proveWithdrawal 成功后，且挑战期结束后调用。
7. **getWithdrawalStatus**
   - 作用：查询提款的当前状态（如 “已发起”“已证明”“已最终化” 等）。
   - 参数：`args: { withdrawalHash: Hash }`（提款哈希）。
   - 返回：提款状态枚举（如`WithdrawalStatus.Initiated`）。
8. **waitForL2Output**
   - 作用：监听并等待 L2 输出根被提交到 L1（用于提款验证前的状态确认）。
   - 参数：`args: { blockNumber: bigint, pollingInterval?: number }`（L2 区块号、轮询间隔）。
   - 返回：当输出根提交到 L1 后返回该输出根信息。

### 二、op-stack/chains（OP Stack 链配置）

#### 核心作用

提供 **OP Stack 兼容链**（如 Optimism 主网、测试网等）的预定义配置，简化开发者连接和交互 OP Stack 网络的流程，同时支持自定义 OP Stack 链配置。

#### 全部关键方法及配置

1. **预定义 OP Stack 链对象**
   提供主流 OP Stack 网络的现成配置（包含链 ID、RPC 地址、区块浏览器等信息），例如：
   - `optimism`：Optimism 主网配置（链 ID：10）。
   - `optimismSepolia`：Optimism Sepolia 测试网（链 ID：11155420）。
   - `optimismGoerli`（已废弃，由 Sepolia 替代）。
   - `base`：Base（基于 OP Stack 的 L2）主网（链 ID：8453）。
   - `baseSepolia`：Base Sepolia 测试网（链 ID：84532）。
2. **getOPStackChain**
   - 作用：根据链 ID 或名称获取预定义的 OP Stack 链配置。
   - 参数：`chainId: number | string`（链 ID 或名称，如 10、'optimism'）。
   - 返回：链配置对象（包含`id`、`name`、`rpcUrls`、`blockExplorers`等）。
3. **defineOPStackChain**
   - 作用：定义自定义 OP Stack 链配置（适用于自建的 OP Stack 网络）。
   - 参数：`config: OPStackChainConfig`（链配置，需包含`id`、`name`、`l1Chain`（对应的 L1 链）、`rpcUrls`等）。
   - 返回：自定义 OP Stack 链对象，可用于客户端连接。
4. **isOPStackChain**
   - 作用：判断一个链配置是否为 OP Stack 兼容链。
   - 参数：`chain: Chain`（链配置对象）。
   - 返回：`boolean`（是否为 OP Stack 链）。

### 总结核心

- **op-stack/client**：核心是封装 OP Stack 跨链交互逻辑，提供存款、提款全流程（发起→证明→最终化）及状态查询工具，简化 L1 与 L2 的资产和消息交互。
- **op-stack/chains**：核心是提供 OP Stack 链的预定义配置和自定义能力，让开发者快速连接主流或自建 OP Stack 网络，无需手动配置链参数。