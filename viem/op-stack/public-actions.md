# Viem 中 OP-Stack 跨链操作方法解析

### 一、核心定位

Viem 中针对 OP - Stack 的跨链操作方法，主要围绕在 Optimism 技术栈（OP - Stack）生态下，实现 Layer 2（L2）网络（如 Optimism、Base 等）与 Layer 1（L1，以太坊主网）之间资产和数据的交互。涵盖了交易构建以及费用、Gas 估算等关键功能，为开发者简化了跨链操作的复杂流程，抽象了 OP - Stack 特有的底层逻辑，便于更高效地开发基于 OP - Stack 的应用。

### 二、跨链交易构建方法

#### 1. `buildDepositTransaction`

**核心作用**：构建从 L1 向 OP - Stack L2 网络存款的交易对象，用于将 ETH 或兼容的代币从 L1 跨链存入 L2。

**关键参数**：

- `to`：L2 接收地址，类型为 `Address`。
- `value`：存款金额，对于 ETH 以 `wei` 为单位，类型为 `bigint`。
- `l1Chain`：L1 的链配置信息。
- `l2Chain`：L2 的链配置信息。

**示例代码**：

```typescript
import { createPublicClient, http } from 'viem';
import { mainnet } from 'viem/chains';
import { optimism } from 'viem/op - stack/chains';
import { buildDepositTransaction } from 'viem/op - stack/actions';

// 创建 L1 公共客户端
const l1Client = createPublicClient({
    chain: mainnet,
    transport: http('https://mainnet.infura.io/v3/YOUR_API_KEY')
});

// 创建 L2 公共客户端
const l2Client = createPublicClient({
    chain: optimism,
    transport: http('https://optimism - mainnet.infura.io/v3/YOUR_API_KEY')
});

// 构建存款交易
const depositTx = buildDepositTransaction({
    to: '0xRecipientAddressInL2', // L2 接收地址
    value: 1000000000000000000n, // 1 ETH 以 wei 为单位
    l1Chain: mainnet,
    l2Chain: optimism
});

console.log(depositTx);
```

#### 2. `buildProveWithdrawal`

**核心作用**：构建用于在 L1 上证明 L2 提款有效性的交易数据，是 OP - Stack 提款流程中的关键步骤，用于验证提款的合法性，推进提款进入最终化阶段。

**关键参数**：

- `withdrawal`：提款对象，包含提款相关的关键信息。
- `output`：对应的 L2 输出根数据，用于验证 L2 状态。
- `l1Chain`：L1 的链配置信息。
- `l2Chain`：L2 的链配置信息。

**示例代码**：

```typescript
import { createPublicClient, http } from 'viem';
import { mainnet } from 'viem/chains';
import { optimism } from 'viem/op - stack/chains';
import { buildProveWithdrawal } from 'viem/op - stack/actions';

const l1Client = createPublicClient({
    chain: mainnet,
    transport: http('https://mainnet.infura.io/v3/YOUR_API_KEY')
});

const l2Client = createPublicClient({
    chain: optimism,
    transport: http('https://optimism - mainnet.infura.io/v3/YOUR_API_KEY')
});

// 假设已经获取到提款对象和 L2 输出根数据
const withdrawal = { /* 提款对象数据 */ };
const l2Output = { /* L2 输出根数据 */ };

const proveTx = buildProveWithdrawal({
    withdrawal,
    output: l2Output,
    l1Chain: mainnet,
    l2Chain: optimism
});

console.log(proveTx);
```

### 三、费用与 Gas 估算方法

#### 1. `estimateContractL1Fee`

**核心作用**：估算在 L2 上调用合约操作时，涉及到的 L1 数据费用。由于 OP - Stack 中 L2 交易数据需要同步到 L1，会产生相应的费用，该方法用于计算这部分成本。

**关键参数**：

- `address`：L2 合约地址，类型为 `Address`。
- `abi`：合约的 ABI，类型为 `Abi`。
- `functionName`：要调用的合约函数名，类型为 `string`。
- `args`：函数调用的参数，类型为 `any[]`（可选）。
- `l2Client`：L2 公共客户端实例。

**示例代码**：

```typescript
import { createPublicClient, http } from 'viem';
import { optimism } from 'viem/op - stack/chains';
import { estimateContractL1Fee } from 'viem/op - stack/actions';
import erc20Abi from './erc20Abi.json'; // 假设已经有 ERC20 合约 ABI 文件

const l2Client = createPublicClient({
    chain: optimism,
    transport: http('https://optimism - mainnet.infura.io/v3/YOUR_API_KEY')
});

const l1Fee = await estimateContractL1Fee({
    address: '0xERC20ContractAddressInL2',
    abi: erc20Abi,
    functionName: 'transfer',
    args: ['0xRecipientAddress', 100n], // 假设转账 100 个代币
    l2Client
});

console.log(`Estimated L1 fee: ${l1Fee} wei`);
```

#### 2. `estimateContractL1Gas`

**核心作用**：专门估算 L2 合约调用操作对应的 L1 Gas 消耗量，便于开发者从 Gas 维度进行成本分析和优化。

**关键参数**：与 `estimateContractL1Fee` 一致。

**示例代码**：

```typescript
import { createPublicClient, http } from 'viem';
import { optimism } from 'viem/op - stack/chains';
import { estimateContractL1Gas } from 'viem/op - stack/actions';
import erc20Abi from './erc20Abi.json';

const l2Client = createPublicClient({
    chain: optimism,
    transport: http('https://optimism - mainnet.infura.io/v3/YOUR_API_KEY')
});

const l1Gas = await estimateContractL1Gas({
    address: '0xERC20ContractAddressInL2',
    abi: erc20Abi,
    functionName: 'transfer',
    args: ['0xRecipientAddress', 100n],
    l2Client
});

console.log(`Estimated L1 Gas: ${l1Gas} units`);
```

#### 3. `estimateContractTotalFee`

**核心作用**：估算 L2 合约调用操作的总费用，包括 L2 执行费用以及 L1 数据费用，为开发者和用户提供完整的成本视角。

**关键参数**：与 `estimateContractL1Fee` 一致。

**示例代码**：

```typescript
import { createPublicClient, http } from 'viem';
import { optimism } from 'viem/op - stack/chains';
import { estimateContractTotalFee } from 'viem/op - stack/actions';
import erc20Abi from './erc20Abi.json';

const l2Client = createPublicClient({
    chain: optimism,
    transport: http('https://optimism - mainnet.infura.io/v3/YOUR_API_KEY')
});

const totalFee = await estimateContractTotalFee({
    address: '0xERC20ContractAddressInL2',
    abi: erc20Abi,
    functionName: 'transfer',
    args: ['0xRecipientAddress', 100n],
    l2Client
});

console.log(`Estimated total fee: ${totalFee} wei`);
```

#### 4. `estimateContractTotalGas`

**核心作用**：估算 L2 合约调用操作的总 Gas 消耗量，涵盖 L2 执行 Gas 以及 L1 数 Gas，帮助开发者分析和优化合约调用的整体 Gas 效率。

**关键参数**：与 `estimateContractL1Fee` 一致。

**示例代码**：

```typescript
import { createPublicClient, http } from 'viem';
import { optimism } from 'viem/op - stack/chains';
import { estimateContractTotalGas } from 'viem/op - stack/actions';
import erc20Abi from './erc20Abi.json';

const l2Client = createPublicClient({
    chain: optimism,
    transport: http('https://optimism - mainnet.infura.io/v3/YOUR_API_KEY')
});

const totalGas = await estimateContractTotalGas({
    address: '0xERC20ContractAddressInL2',
    abi: erc20Abi,
    functionName: 'transfer',
    args: ['0xRecipientAddress', 100n],
    l2Client
});

console.log(`Estimated total Gas: ${totalGas} units`);
```

#### 5. `estimateInitiateWithdrawalGas`

**核心作用**：估算在 L2 上发起提款操作所需的 L2 Gas 消耗量，便于用户和开发者在发起提款前，评估所需的 Gas 成本，确保钱包中有足够的余额支付 Gas。

**关键参数**：

- `to`：L1 接收地址，类型为 `Address`。
- `value`：提款金额，类型为 `bigint`。
- `l2Client`：L2 公共客户端实例。

**示例代码**：

```typescript
import { createPublicClient, http } from 'viem';
import { optimism } from 'viem/op - stack/chains';
import { estimateInitiateWithdrawalGas } from 'viem/op - stack/actions';

const l2Client = createPublicClient({
    chain: optimism,
    transport: http('https://optimism - mainnet.infura.io/v3/YOUR_API_KEY')
});

const initiateWithdrawalGas = await estimateInitiateWithdrawalGas({
    to: '0xRecipientAddressInL1',
    value: 1000000000000000000n, // 1 ETH 以 wei 为单位
    l2Client
});

console.log(`Estimated Gas for initiating withdrawal: ${initiateWithdrawalGas} units`);
```

#### 6. `estimateL1Fee`

**核心作用**：估算普通 L2 交易（非合约调用场景，如简单的 ETH 转账）所产生的 L1 数据费用，帮助用户和开发者了解跨链转账时在 L1 上产生的额外成本。

**关键参数**：

- `transaction`：L2 交易对象，包含 `to`、`value`、`data` 等信息，类型为 `TransactionRequest`。
- `l2Client`：L2 公共客户端实例。

**示例代码**：

```typescript
import { createPublicClient, http } from 'viem';
import { optimism } from 'viem/op - stack/chains';
import { estimateL1Fee } from 'viem/op - stack/actions';

const l2Client = createPublicClient({
    chain: optimism,
    transport: http('https://optimism - mainnet.infura.io/v3/YOUR_API_KEY')
});

const l2Transaction = {
    to: '0xRecipientAddressInL2',
    value: 1000000000000000000n, // 1 ETH 以 wei 为单位
    // 其他交易相关参数
};

const l1Fee = await estimateL1Fee({
    transaction: l2Transaction,
    l2Client
});

console.log(`Estimated L1 fee for L2 transaction: ${l1Fee} wei`);
```

#### 7. `estimateL1Gas`

**核心作用**：估算普通 L2 交易（非合约调用）所产生的 L1 Gas 消耗量，从 Gas 维度辅助开发者分析和优化 L2 普通交易在 L1 上的资源消耗情况。

**关键参数**：与 `estimateL1Fee` 一致。

**示例代码**：

```typescript
import { createPublicClient, http } from 'viem';
import { optimism } from 'viem/op - stack/chains';
import { estimateL1Gas } from 'viem/op - stack/actions';

const l2Client = createPublicClient({
    chain: optimism,
    transport: http('https://optimism - mainnet.infura.io/v3/YOUR_API_KEY')
});

const l2Transaction = {
    to: '0xRecipientAddressInL2',
    value: 1000000000000000000n, // 1 ETH 以 wei 为单位
    // 其他交易相关参数
};

const l1Gas = await estimateL1Gas({
    transaction: l2Transaction,
    l2Client
});

console.log(`Estimated L1 Gas for L2 transaction: ${l1Gas} units`);
```

#### 8. `estimateTotalFee`

**核心作用**：估算普通 L2 交易（非合约调用）的总费用，包括 L2 执行费用以及 L1 数据费用，为用户和开发者提供完整的交易成本信息，以便做出合理的决策。

**关键参数**：与 `estimateL1Fee` 一致。

**示例代码**：

```typescript
import { createPublicClient, http } from 'viem';
import { optimism } from 'viem/op - stack/chains';
import { estimateTotalFee } from 'viem/op - stack/actions';

const l2Client = createPublicClient({
    chain: optimism,
    transport: http('https://optimism - mainnet.infura.io/v3/YOUR_API_KEY')
});

const l2Transaction = {
    to: '0xRecipientAddressInL2',
    value: 1000000000000000000n, // 1 ETH 以 wei 为单位
    // 其他交易相关参数
};

const totalFee = await estimateTotalFee({
    transaction: l2Transaction,
    l2Client
});

console.log(`Estimated total fee for L2 transaction: ${totalFee} wei`);
```

#### 9. `estimateTotalGas`

**核心作用**：估算普通 L2 交易（非合约调用）的总 Gas 消耗量，涵盖 L2 执行 Gas 以及 L1 数据 Gas，帮助开发者全面了解交易的 Gas 使用情况，从而进行针对性的优化。

**关键参数**：与 `estimateL1Fee` 一致。

**示例代码**：

```typescript
import { createPublicClient, http } from 'viem';
import { optimism } from 'viem/op - stack/chains';
import { estimateTotalGas } from 'viem/op - stack/actions';

const l2Client = createPublicClient({
    chain: optimism,
    transport: http('https://optimism - mainnet.infura.io/v3/YOUR_API_KEY')
});

const l2Transaction = {
    to: '0xRecipientAddressInL2',
    value: 1000000000000000000n, // 1 ETH 以 wei 为单位
    // 其他交易相关参数
};

const totalGas = await estimateTotalGas({
    transaction: l2Transaction,
    l2Client
});

console.log(`Estimated total Gas for L2 transaction: ${totalGas} units`);
```

### 四、核心价值

- **降低开发门槛**：通过封装复杂的 OP - Stack 跨链底层逻辑，开发者无需深入了解 OP - Stack 的具体实现细节，就能快速实现跨链存款、提款等功能。
- **精准成本把控**：提供全面且细致的费用和 Gas 估算方法，使开发者和用户能够清晰地了解跨链操作的成本构成，有助于进行成本优化和预算管理。
- **兼容性强**：适用于所有基于 OP - Stack 的 L2 网络，开发者在不同的 OP - Stack 网络间切换时，能够复用这些方法，提高开发效率。