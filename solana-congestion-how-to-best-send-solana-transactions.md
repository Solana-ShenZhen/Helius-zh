# Solana 网络拥堵：如何最好地发送 Solana 交易

## 简介

"Transaction simulation failed: Blockhash not found." "Failed to send transaction." 这些错误信息已变得太过熟悉。网络拥堵已经让最基本的交易都变成了一场碰运气的游戏。但这种痛苦是不必要的。

本指南探讨了在高流量时期有效导航以确保交易成功的简单策略。我们将介绍处理网络拥堵的综合方法，包括[优先费用](https://www.helius.dev/blog/priority-fees-understanding-solanas-transaction-fee-mechanics)、[计算单元优化](https://www.helius.dev/blog/optimizing-solana-programs)，以及 Helius 独家秘诀，确保您的交易始终能够成功。

在本指南中，我们将探讨：

- 使用序列化交易设置优先费用
- 使用账户密钥设置优先费用
- 高级优先费用策略
- 交易优化技术
- 利用质押节点端点确保交易送达

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/672e143f54073df03b6f7bf9_AD_4nXfMAyHeyvyFY63BqJ2ckgpIuc2haFDdjS2txEeTTLPd4nNPSyzHzaytp-fWRXvcE9fWnrxQ4oO5Ai9KJ1Uce9hH0T4-VzKCmZ9eBPuia4UXZNzzl2TTmW7Ala8Xrbx2y_YWbO7DFc8rE-2CRrzcfpQjE0dA.png" loading="lazy" alt="">

## 优先费用

优先费用作为一种竞价机制，让您向验证节点表明交易的重要性。这些费用以每[计算单元（compute unit）](https://solana.com/docs/terminology#compute-units)[微 lamport（micro-lamport）](https://solana.com/docs/terminology#lamport) 为单位定价，由您的交易与之交互的特定账户决定，为每个账户创建独立的费用市场。通过基于账户特定拥堵情况战略性地设置这些费用，您可以显著提高交易被纳入下一个区块的机会。

接下来，我们将看看如何使用 Helius 的[优先费用 API](https://docs.helius.dev/solana-rpc-nodes/priority-fee-api)通过各种方法正确设置优先费用：

### 序列化交易

设置优先费用最直接的方法是使用序列化交易。序列化交易是交易的二进制表示，转换为可以在网络上传输的线格式缓冲区。

1. 序列化您的交易并将其传递给 API：

```ts
const serializedTx = transaction.serialize();

const response = await fetch(heliusEndpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    id: "helius-example",
    method: "getPriorityFeeEstimate",
    params: [
      {
        transaction: serializedTx,
        options: { recommended: true },
      },
    ],
  }),
});

const { result } = await response.json();
const priorityFee = result.priorityFeeEstimate;
```

2. 将推荐的费用添加到您的交易中 — 指令的顺序并不重要：

```ts
transaction.add(
  ComputeBudgetProgram.setComputeUnitPrice({
    microLamports: priorityFee,
  })
);
```

### 账户公钥

另一种方法是向优先费用 API 提供交易涉及的账户公钥列表：

1. 从交易中提取账户公钥：

```ts
const transaction = new Transaction();

// Add your instructions to the transaction

// Extract all account keys from the transaction
const accountKeys = transaction.compileMessage().accountKeys;

// Convert PublicKeys to base58 strings
const publicKeys = accountKeys.map((key) => key.toBase58());
```

2. 将账户公钥传递给优先费用 API：

```ts
const response = await fetch(heliusEndpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    id: "helius-example",
    method: "getPriorityFeeEstimate",
    params: [
      {
        accountKeys: publicKeys,
        options: { recommended: true },
      },
    ],
  }),
});

const { result } = await response.json();
const priorityFee = result.priorityFeeEstimate;
```

3. 将推荐的费用添加到您的交易中：

```ts
transaction.add(
  ComputeBudgetProgram.setComputeUnitPrice({
    microLamports: priorityFee,
  })
);
```

虽然两种方法都可行，但推荐使用序列化交易方法。使用账户公钥需要手动跟踪和包含所有相关账户，如果遗漏任何账户可能导致费用估算不准确。

序列化交易方法通过自动包含所有必要账户并基于完整的交易上下文提供更准确的费用估算，消除了这种风险。

### 高级优先费用策略

对于大多数场景，使用带有推荐优先费用的序列化交易就足够了。但是，如果需要更精细的控制，优先费用 API 允许您请求不同优先级的费用估算并启用[空槽位评估](https://docs.helius.dev/solana-rpc-nodes/priority-fee-api#empty-slot-evaluation)。

#### 优先级别

以下是如何[请求所有优先费用级别](https://docs.helius.dev/solana-rpc-nodes/priority-fee-api#examples)的示例：

```ts
const response = await fetch(heliusEndpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    id: "helius-example",
    method: "getPriorityFeeEstimate",
    params: [
      {
        transaction: serializedTx,
        options: {
          includeAllPriorityFeeLevels: true,
        },
      },
    ],
  }),
});

const { result } = await response.json();
console.log(result.priorityFeeLevels);
```

这将返回一个带有不同优先级级别的对象：

```sh
{
  "priorityFeeLevels": {
    "min": 10000.0,
    "low": 10000.0,
    "medium": 10000.0,
    "high": 100000.0,
    "veryHigh": 5483924.800000011,
    "unsafeMax": 8698904817.0
  }
}
```

优先级别对应于最近交易费用的不同百分位数：

- **min**：第 0 百分位
- **low**：第 25 百分位
- **medium**：第 50 百分位
- **high**：第 75 百分位
- **veryHigh**：第 95 百分位
- **unsafeMax**：第 100 百分位（谨慎使用）

为了进一步提高交易落地率，可以考虑使用 **high** 或 **very high** 优先级别。但是，使用较高的优先级别时要谨慎，特别是 **veryHigh** 和 **unsafeMax**，因为它们会显著增加交易成本。

注意：当使用 **recommended: true** 选项时，如果中等（第 50 百分位）优先费用超过 10,000，API 将返回该中等费用。如果中等费用低于 10,000，API 将返回 10,000。这确保了交易有一个基准优先费用。

#### 空槽评估

优先费用 API 的最新更新通过考虑空槽实现了更智能的费用估算。当区块中不包含特定账户的任何交易时，您可以配置 API [将这些槽评估为零费用](https://docs.helius.dev/solana-rpc-nodes/priority-fee-api#empty-slot-evaluation)，从而在网络活动较低时提供更准确的估算：

```ts
const response = await fetch(heliusEndpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    id: "helius-example",
    method: "getPriorityFeeEstimate",
    params: [
      {
        transaction: serializedTx,
        options: {
          recommended: true,
          evaluateEmptySlotAsZero: true,
        },
      },
    ],
  }),
});
```

当启用 **evaluateEmptySlotAsZero** 时，API 会：

- 识别不包含您账户交易的区块
- 将这些空槽计为零费用
- 在网络拥堵较低时下调费用估算
- 防止在网络活动较低时支付过高费用

## 交易优化策略

除了优先费用外，优化交易还涉及几个关键组件，它们共同作用以提高交易落地率：

### 计算单元管理

1. 使用 Helius SDK [获取交易的最优计算单元](https://github.com/helius-labs/helius-sdk?tab=readme-ov-file#getcomputeunits)：

```ts
const response = await helius.rpc.getComputeUnits({
  transaction: serializedTransaction,
});

const computeUnits = response.computeUnits;
```

2. 将计算单元限制添加到您的交易中：

```ts
transaction.add(
  ComputeBudgetProgram.setComputeUnitLimit({
    units: computeUnits,
  })
);
```

### 确认级别

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/672e143f5e051c143f25e09a_AD_4nXdjT1Kqn2JXq-xi9YXHtT64E4hW_YJHg4AOjpJkre0GPWgVoXuzNkOdB__5eE9Ldem2F4ZT-ygbxuShxIx6zj6sI70TaSjxg9Pq7yh8e0zk5cc6HVodvlkmfRU8RQ0B7QqSgSBN-7hBEK43Vj4sDYVBdup_.png" loading="lazy" alt="">

对于大多数应用程序来说，使用 **processed** 确认级别能提供最佳平衡。它：

- 提供最快的确认时间
- 对大多数用例来说安全性足够
- 降低区块哈希过期的风险
- 在网络拥堵时最大限度减少交易丢失

在发送和确认交易时始终使用相同的确认级别，以保持应用程序行为的一致性。

### RPC 配置

将 **maxRetries** 设置为 0 并实现您自己的重试逻辑，以更好地控制交易重新提交。使用 **skipPreflight: true** 来减少延迟。这在网络拥堵期间特别有用：

```ts
const signature = await connection.sendTransaction(transaction, {
  maxRetries: 0,
  skipPreflight: true,
});
```

## 使用质押端点

Helius [质押端点](https://docs.helius.dev/solana-rpc-nodes/sending-transactions-on-solana) 提供直接且有保障的质押连接访问，无需手动优化优先费用：

```ts
const connection = new Connection(
  "https://staked.helius-rpc.com?api-key=YOUR_API_KEY"
);

// Transaction will be processed with optimal priority
const signature = await connection.sendTransaction(transaction);
```

质押端点：

- 提供有保障的质押带宽
- 减少网络拥堵期间的交易失败
- 通过专用基础设施提供更快的交易传播

## 总结

有效使用优先费用对于在 Solana 上成功完成交易至关重要，尤其是在网络拥堵期间。通过利用 Helius 优先费用 API 并实施我们讨论的策略，您可以在平衡成本的同时显著提高交易成功率。保持关注并根据网络状况调整以获得最佳结果。

如果您已经读到这里，感谢您的阅读！我们希望本指南帮助您了解如何优化您在 Solana 上的交易。请务必订阅我们的新闻通讯，这样您就不会错过任何关于 Solana 最新动态的更新。

准备深入了解更多？

立即探索 [Helius 博客](https://www.helius.dev/blog)上的最新文章，继续您的 Solana 之旅。
