# 如何在 Solana 上成功提交交易

最近，Solana 网络经历了前所未有的交易量，这导致了大量交易失败或被丢弃。Solana 的每秒处理交易数（TPS）约为 [2000-3000](https://solscan.io/analytics#networks)，其中大约 800-900 是非投票交易。[网络层](https://www.helius.dev/blog/all-you-need-to-know-about-solana-and-quic)的 Rust 实现（[QUIC](https://quinn-rs.github.io/quinn/networking-introduction.html)）在处理高需求场景中的垃圾交易时存在局限性，这可能导致区块领导者不得不选择性地丢弃连接。在所有失败的交易中，[约有 8% 是由真实用户发起的，其余的则是由机器人生成的任意交易](https://x.com/_nishil_/status/1777048361729994777)。

理解交易是如何在 Solana 上提交和处理的，对于解决交易失败问题至关重要。本文深入探讨了交易失败的可能原因，并推荐了提高交易吞吐量的最佳实践。本文假设您对 [Solana 的编程模型](https://www.helius.dev/blog/the-solana-programming-model-an-introduction-to-developing-on-solana)以及创建和[发送交易](https://solanacookbook.com/references/basic-transactions.html#how-to-send-sol)有基本的了解。

## 交易

程序的执行始于提交到[集群](https://www.helius.dev/blog/the-solana-programming-model-an-introduction-to-developing-on-solana#what-are-solana-clusters)的[交易](https://www.helius.dev/blog/the-solana-programming-model-an-introduction-to-developing-on-solana#what-are-transactions)。一个交易包含以下内容：

- 它打算读取或写入的所有账户的数组
- 一个或多个指令（即最小的执行单元）
- 最近的区块哈希
- 一个或多个签名

运行时会按顺序处理交易中包含的每个指令，并且是原子性的。如果任何指令部分失败，整个交易将失败。

## 什么是区块哈希？

"区块哈希" 是某一[槽位](https://solana.com/docs/terminology#slot)最新的历史证明（[PoH](https://solana.com/news/proof-of-history)）哈希。由于 Solana 依赖 PoH 作为可信的时钟，一个交易的最近区块哈希可以被视为时间戳。区块哈希可以防止重复，并为交易提供有效期。如果一个交易的区块哈希过旧，它将被拒绝。区块哈希的最大寿命为 150 个区块或大约 1 分钟 19 秒。

## 交易是如何提交的？

Solana 由一群[验证者](https://solana.com/docs/terminology#validator)维护，他们验证添加到[分类账](https://solana.com/docs/terminology#ledger)的交易。从这群验证者中选择一个领导验证者来将条目追加到分类账中。分类账中的一个条目可以是一个[滴答条](https://solana.com/docs/terminology#tick)目或一个[交易条目](https://solana.com/docs/terminology#transactions-entry)。分类账持有一个包含客户端签名交易的条目列表。[创世区块](https://solana.com/docs/terminology#genesis-block)从概念上追溯到分类账，但为了减少存储，实际验证者的分类账可能只保留较新的区块，因为设计上不需要旧的区块来验证未来的区块。

领导验证者在每个[槽位](https://solana.com/docs/terminology#slot)上只能生成一个[区块](https://solana.com/docs/terminology#block)，区块哈希是用于标识每个区块的唯一标识符。它是区块中所有条目的哈希，包括上一个区块的哈希。领导计划在每个纪元之前确定，通常是在两天左右，以决定在任何给定时间哪个验证者将担任当前的领导者。当交易发起时，它会被转发给当前和下一个[领导验证者](https://solana.com/docs/terminology#leader-schedule)。

## 交易可以通过以下方式提交给领导者

- **RPC 服务器**：可以通过 RPC 提供商使用 [`sendTransaction`](https://solana.com/docs/rpc/http/sendtransaction) JSON-RPC 方法提交交易。接收的 RPC 节点将尝试每两秒将其作为 [UDP](https://www.helius.dev/blog/all-you-need-to-know-about-solana-and-quic#what%E2%80%99s-udp) 包发送给当前和下一个领导者，直到交易被最终确定或交易的区块哈希过期（150 个区块后或大约 1 分钟 19 秒）。在此之前，除了客户端和中继 RPC 节点之外，不会有任何交易记录。
- **TPU 客户端**：[TPU 客户端](https://crates.io/crates/solana-tpu-client/1.17.28)仅提交交易。客户端软件需要处理重广播和领导者转发。

要使用 [`sendTransaction`](https://solana.com/docs/rpc/http/sendtransaction) 方法，需要传递编码为字符串的交易对象。其他可选参数包括：

- **encoding**：用于交易数据的编码，通常是 base58 或 base64。
- **skipPreflight**：Preflight 检查包括验证交易签名并对预定的银行槽位模拟交易。如果 Preflight 检查失败，将返回错误。此功能的默认设置为 `false`，即不跳过 Preflight 检查。
- **preflightCommitment**：指定执行 Preflight 检查时的[承诺级别](https://docs.solanalabs.com/consensus/commitments)。默认情况下，承诺级别设置为 `finalized`，但可以通过指定字符串更改。建议将承诺级别和 Preflight 承诺设置为相同，以避免产生混淆行为。
- **maxRetries**：`maxRetries` 参数确定 RPC 节点需要重试将交易发送给领导者的最大次数。如果未提供此参数，RPC 节点将重试交易，直到交易被最终确定或区块哈希过期。
- **minContextSlot**：`minContextSlot` 参数指定执行 Preflight 交易检查的最小槽位。

## 交易是如何处理的？

验证者的交易处理单元（[TPU](https://docs.solanalabs.com/validator/tpu)）接收交易，验证签名，执行交易，并与网络中的其他验证者共享。

TPU 以五个不同的阶段处理交易：

1. **Fetch Stage**：负责[接收交易](https://github.com/solana-labs/solana/blob/cd6f931223181d5a1d47cba64e857785a175a760/core/src/tpu.rs#L79)。它将传入的交易根据三个端口进行分类：
   - `tpu`：处理常规交易，例如代币转移、NFT 铸造和程序指令。
   - `tpu_vote`：专注于投票交易。
   - `tpu_forwards`：如果当前领导者无法处理所有交易，它会将未处理的包转发给下一个领导者。
   包将分批（每组 128 个）并转发到签名验证阶段。

2. **SigVerify Stage**：[签名验证阶段](https://github.com/solana-labs/solana/blob/cd6f931223181d5a1d47cba64e857785a175a760/core/src/tpu.rs#L91)验证包上的签名，如果验证失败则将其删除。投票和常规包在两个独立的流水线中运行。从软件的角度来看，它接收到的包包含一些元数据，但尚不清楚这些包是否是交易。如果安装了 GPU，它将用于签名验证。此外，在高流量情况下，会有处理过多包的逻辑，使用 IP 地址来丢弃包。

3. **Banking Stage**：[这个阶段](https://github.com/solana-labs/solana/blob/cd6f931223181d5a1d47cba64e857785a175a760/core/src/tpu.rs#L125)负责筛选和处理交易。目前，它由 6 个独立的工作线程组成，其中 2 个是投票线程，4 个是非投票线程。常规交易被添加到非投票线程中。每个线程都有一个本地缓冲区，可以在优先队列中容纳多达 64 个非冲突交易。然后这些交易并行处理，通过 [Sealevel](https://medium.com/solana-labs/sealevel-parallel-processing-thousands-of-smart-contracts-d814b378192) 实现。您可以参考[此视频](https://www.youtube.com/watch?v=R7hq8ampBio)以了解更多有关银行阶段的信息。

4. **Proof of History Service**：[PoH 服务](https://github.com/solana-labs/solana/blob/cd6f931223181d5a1d47cba64e857785a175a760/poh/src/poh_service.rs)模块记录滴答时间的流逝。每个滴答代表一个时间单位，一个槽位中有 64 个滴答。哈希将不断生成，直到银行阶段接收到一个记录：

   ```text
   next_hash = hash(prev_hash, hash(transaction_ids))
   ```

   这些记录然后转换为[条目](https://solana.com/docs/terminology#entry)，并通过广播阶段广播到网络。

5. **Broadcast Stage**：PoH 服务中的条目将转换为碎片，代表区块的最小单位，然后使用称为 [Turbine](https://docs.solanalabs.com/consensus/turbine-block-propagation) 的区块传播技术发送到网络的其他部分。在较高层次上，Turbine 将区块分成更小的部分，并通过分层结构的节点进行分发。节点无需与每个其他节点通信。它们只需与选定的少数节点进行通信。您可以参考此文章以了解更多关于 [Turbine](https://www.helius.dev/blog/turbine-block-propagation-on-solana) 的工作原理。

## 交易如何失败？

排除由于错误指令或自定义程序错误导致的失败，交易失败的可能原因包括：

1. **网络丢包**：
   网络层可能在领导者甚至处理之前就丢弃了一个交易。[UDP 包丢失](https://www.baeldung.com/cs/udp-packet-loss)是最简单的原因。另一个原因与 TPU 的获取阶段有关。当网络负载高时，验证者可能会因为需要处理的交易数量过多而不堪重负。验证者可以将多余的交易转发到下一个验证者的 `tpu_forward` 端口。然而，可以转发的数据量有限，每次转发仅限于验证者之间的一跳。这意味着在 `tpu_forwards` 端口收到的交易不会转发给其他验证者。如果未处理的重广播队列大小超过 10,000 个交易，新提交的交易将被丢弃。

2. **过期或不正确的区块哈希**：
   每个交易都有一个 "最近区块哈希"，用于作为 [PoH 时钟](https://www.helius.dev/blog/proof-of-history-proof-of-stake-proof-of-work-explained#what%E2%80%99s-proof-of-history)的时间戳。这个区块哈希帮助验证者避免两次处理相同的交易，并跟踪交易的处理时间和顺序。如果验证者在处理过程中由于无效的区块哈希而拒绝交易，该交易将失败。

3. **区块哈希过期**：
   当一个交易的区块哈希不再被视为“最近”时，它就会过期。为了处理一个交易，Solana 验证者会在一个区块中搜索相应区块哈希的槽位号。如果验证者找不到区块哈希的槽位号，或者查找的槽位号比正在处理的区块的槽位号低 151 个以上，交易将被拒绝。默认情况下，如果交易在一定时间内未提交到区块中，Solana 交易将过期（大约 1 分钟 19 秒）。

4. **滞后的 RPC 节点**：
   当您通过 RPC 提交交易时，可能会出现 RPC 池超前于其余部分的情况。这可能会在池中的节点需要协同工作时引发问题。例如，如果从池的超前部分查询并提交了交易的 `recentBlockhash`，而滞后部分的节点无法识别该超前的区块哈希，则该交易将被拒绝。您可以通过在提交交易时启用 Preflight 检查来检测这一点。

5. **临时网络分叉**：
   临时网络分叉也会导致交易被丢弃。如果一个验证者在银行阶段慢于重放其区块，它可能会创建一个少数分叉。当客户端构建交易时，可能会引用仅存在于少数分叉中的 `recentBlockhash`。在交易提交后，集群可能会在交易处理之前切换到非少数分叉。在这种情况下，由于未找到区块哈希，交易会被丢弃。

## 如何确保交易成功提交？

要诊断确认问题，了解交易过期非常重要。按照以下步骤提高成功提交交易的机会：

**简要说明：**

- 使用 `confirmed` 或 `finalized` 承诺级别获取最新的区块哈希。
- 将 `skipPreflight` 设置为 `true`。
- 优化请求的计算单元数量。
- 动态添加和计算优先费用。
- 将 `maxRetries` 设置为 `0`，并添加自定义重试逻辑来发送交易。
- 探索质押连接。
- 如果交易不敏感于时间，使用[耐用的 Nonces](https://www.helius.dev/blog/solana-transactions) 。

## 区块哈希

交易有有限的时间可以由验证者处理。如果与交易关联的区块哈希在验证者处理交易之前过期，则交易将被取消。为了确保您的交易顺利通过，重要的是使用最近的区块哈希发送它。如果区块哈希在验证者处理交易之前过期，您可以重新尝试使用新的区块哈希来确保成功处理交易。这可以通过以下两种方式完成：

1. **设置新的承诺级别**：
   获取最新区块哈希的推荐 RPC API 方法是 [`getLatestBlockhash`](https://solana.com/docs/rpc/http/getlatestblockhash)。默认情况下，该方法使用 `finalized` [承诺级别](https://solana.com/docs/rpc/http/getlatestblockhash)返回最近最终确定的区块的区块哈希。此承诺级别表明该区块上方已至少添加了 31 个确认的区块。这消除了使用属于已丢弃分叉的区块哈希的风险。然而，通常最近的确认区块和最终确定的区块之间至少有 32 槽位的差距。这个权衡大约减少了 13 秒的交易过期时间，在集群不稳定的情况下可能会更长。

   您可以通过设置承诺参数为不同的级别来覆盖区块哈希的承诺级别。推荐为 RPC 请求设置 `confirmed` 承诺级别，因为它通常仅比 `processed` 承诺级别落后几个槽位，并且属于丢弃分叉的可能性较低。虽然 `processed` 承诺级别相比其他承诺级别获取最最近的区块哈希，但由于 Solana 协议中的分叉[约有 5% 的区块](https://solana.com/docs/advanced/confirmation#fetch-blockhashes-with-the-appropriate-commitment-level)未最终确定，因此不推荐使用。如果您的交易使用属于丢弃分叉的区块哈希，则该交易在最终确定的区块链中不会被视为最近。

2. **频繁轮询新的最近区块哈希**：
   添加一个脚本，通过 `getLatestBlockhash` 方法频繁获取并存储最新的区块哈希（每 60 秒）。因此，每当用户触发交易时，应用程序就会有一个新鲜的区块哈希准备就绪。钱包也应该频繁轮询新的区块哈希，并在签署交易前替换交易的最近区块哈希，以确保其尽可能的最近。

## 跳过 Preflight

在提交交易之前，会执行以下 Preflight 检查：

- 验证交易的签名。
- 根据指定的银行槽位模拟交易。如果失败，将返回错误。

如果选择的模拟区块比您的交易的区块哈希所用区块旧，模拟将因“找不到区块哈希”错误而失败。

如果您确信交易签名已验证且没有其他错误，可以跳过 Preflight 检查。即使使用 `skipPreflight` 参数，仍应将 `preflightCommitment` 参数设置为用于获取交易区块哈希的相同承诺级别，用于 `sendTransaction` 和 [`simulateTransaction`]((https://solana.com/docs/rpc/http/simulatetransaction))请求。

## 计算单元

当一个交易在网络上确认时，它会消耗区块中可用的总计算单元（CU）。目前，一个区块的总计算限制为 [48M CU](https://github.com/solana-labs/solana/blob/master/cost-model/src/block_cost_limits.rs#L64-L68)。开发者可以为其交易指定计算单元预算。如果未设置预算，将使用[默认值 1,400,000 CU](https://github.com/solana-labs/solana/blob/27eff8408b7223bb3c4ab70523f8a8dca3ca6645/program-runtime/src/compute_budget_processor.rs#L19)，通常比大多数交易所需的 CU 多。许多交易不会使用整个 CU 预算，因为请求比实际需要更多的预算不会受到惩罚。然而，过多的 CU 请求会使调度程序在调度交易时更加困难，因为调度程序在交易执行前不知道区块中剩余多少计算能力。为避免这种情况，开发者应设置与交易需求相匹配的合理 CU 请求。您可以参考[本指南](https://solana.com/developers/guides/advanced/how-to-optimize-compute)以优化计算单元预算。在即将发布的 Solana 客户端 v1.18 更新中，[计算单元需求较少的交易将获得更高的优先级](https://github.com/solana-labs/solana/pull/34888)。

优化计算单元（CU）使用有以下好处：

- 较小的交易更有可能被包含在区块中。
- 更便宜的指令使您的程序更具可组合性。
- 降低整体区块使用率，使更多的交易能被包含在区块中。

## 实施优先费用

可以在基础交易费用的基础上添加优先费用，以使交易在验证者中优先处理。这些费用以每计算单元的微 lamports（例如少量 SOL）为单位计算。它们被添加到交易中，使其在网络中的区块内更具经济吸引力，验证者节点会优先处理这些交易。

然而，重要的是要注意，有一个应该支付优先费用的上限。支付超出典型费用的优先费用不会提高交易成功的概率。因此，建议动态计算优先费用，以确保支付适当的费用以保持竞争力，同时避免过度支付。此集成非常简单，您可以参考官方文档了解优先费用，或使用现成的 Helius API。

## 实施强大的重试逻辑

在网络拥塞的情况下，在代码中实现自定义逻辑以处理交易失败并手动重试这些交易。为此，在使用 `sendTransaction` 提交交易时将 `maxRetries` 参数设置为 `0`。您可以使用不同的方法来重试交易：

- 使用不同的承诺级别轮询交易状态，并持续使用相同的签名交易，直到它通过指数回退机制得到确认，以避免垃圾消息。或者，您可以以恒定间隔提交交易，直到发生超时。
- 存储 `getLatestBlockhash` 方法返回的 `lastValidBlockHeight`。然后，轮询集群的区块高度，并在当前区块高度超过 `lastValidBlockHeight` 后手动重试交易。当通过 `getLatestBlockhash` 轮询时，建议指定您意图的承诺级别。通过将承诺设置为 `confirmed`（已投票）或 `finalized`（确认后大约 30 个区块），可以避免轮询少数分叉的区块哈希。

## 质押连接

领导者的网络带宽容量是有限的。为了有效利用它，质押加权是必要的，以避免盲目地按先到先得的原则接受交易，而不考虑其来源。Solana 作为一个权益证明（PoS）网络，使得扩展质押加权以提高交易服务质量变得很自然。这意味着拥有 0.5% 质押的节点可以发送至少 0.5% 的包给领导者。其余的网络，或剩余质押的组合，将无法完全将其冲刷掉。这种机制称为质押加权服务质量（SWQoS）。

Helius 提供两种形式的质押连接：

- **共享质押连接**：推荐给日常用户、企业和初创公司。如果其优先费用达到或超过 Helius 优先费用 API 提供的推荐值，则所有付费计划均可访问。
- **专用质押连接**：保证质押连接带宽，推荐给企业、量化机构和交易公司。对于有兴趣使用专用质押连接的用户，我们提供了以下表格。

## 耐用的 Nonces

[耐用 Nonces](https://docs.solanalabs.com/implemented-proposals/durable-tx-nonces) 允许创建和签署一个交易，该交易可以在未来任何时间点提交。它们用于需要更多时间生成交易签名的[情况](https://solana.com/developers/guides/advanced/introduction-to-durable-nonces#durable-nonce-applications)，例如托管服务。如果您的交易对时间不敏感，可以使用此方法绕过交易最近区块哈希的短寿命。

要开始使用耐用交易，您需要提交一个调用指令的交易，以在链上创建一个特殊的 "nonce" 账户，并在其中存储一个耐用的区块哈希。Nonce 账户存储 nonce 的值。只要 nonce 账户未被使用，您可以通过遵循以下两条规则创建耐用交易：

1. 指令列表必须以 [`advance nonce` 系统指令开头](https://docs.rs/solana-program/latest/solana_program/system_instruction/fn.advance_nonce_account.html)，该指令会加载链上的 nonce 账户。
2. 交易的区块哈希必须等于存储在链上 nonce 账户中的耐用区块哈希。

您可以参考[本文](https://solana.com/developers/guides/advanced/introduction-to-durable-nonces)学习如何通过 CLI 和 Web3.js 实现耐用 Nonces。

## Helius 提交交易的方法

`sendTransaction` 请求会自动路由到我们最近的 RPC 节点。如果未指定 `maxRetries`，则它每 2 秒重试交易，直到区块哈希过期。我们建议将 `maxRetries` 设置为 `0`，并每 2 秒重新广播交易，直到确认。

为了应对当前的网络拥堵，我们日以继夜地努力提高用户的交易着陆率。我们已经降低了 `sendTransaction` 请求的速率限制。此措施旨在管理拥堵并防止向验证者发送垃圾信息。您可以在[此处](https://docs.helius.dev/welcome/pricing-and-rate-limits)查看限制。

接下来，我们通过质押连接为付费计划路由高质量流量。我们利用我们的[验证者](https://x.com/heliuslabs/status/1785317488265179555)进行这些质押连接。如果总优先费用达到至少 10,000 Lamports（集群中位数），则流量被视为高质量。

我们要求总费用超过 10,000 Lamports，以确保我们向验证者提供高质量流量。验证者已经开始对发送低费用交易的流量源进行速率限制（甚至完全阻止）。

使用质押连接可以显著提高您的交易着陆率。请参考[此文](https://solana.com/developers/guides/advanced/how-to-use-priority-fees#how-do-i-implement-priority-fees)以了解更多关于设置优先费用以创建交易的信息。

## 我们的建议

- **初学者/中级用户**：我们建议将 `skipPreflight` 设置为 `false`。Preflight 检查包括验证交易签名并根据预定的银行槽位模拟交易。如果 Preflight 检查失败，将返回错误。如果跳过 Preflight 检查，交易可能会因为配置错误而被丢弃。

- **高级用户**：对于需要最低延迟的高级用户，您可以将 `skipPreflight` 设置为 `true`。然而，您需要确保交易配置正确。

## 结论

在网络拥堵期间成功提交交易需要对 Solana 网络架构和交易处理机制有深入了解。通过掌握区块哈希在交易唯一性和及时性中的作用，了解通过 RPC 服务器或 TPU 客户端提交交易的过程，以及设置正确参数（如 `skipPreflight`、`preflightCommitment` 和 `maxRetries`）的重要性，用户可以显著提高交易性能。实施自定义重试机制和利用质押连接可以帮助增加成功率。

此外，了解网络的当前局限性和 Anza 为解决这些问题所做的努力，如[即将发布的 v1.18 客户端版本](https://github.com/anza-xyz/agave/wiki/v1.18-Release-Schedule)，也至关重要。随着网络的发展和扩展，保持信息更新和适应性将是有效与之互动的关键。

如果您需要任何帮助或支持，请随时通过 Discord 联系我们。确保在下方输入您的电子邮件地址，以便您不会错过有关 Solana 最新动态的更新。准备深入了解更多？立即探索 [Helius 博客](https://www.helius.dev/blog)上的最新文章，继续您的 Solana 之旅。

## 资源

- [交易 – Solana Cookbook](https://solanacookbook.com/core-concepts/transactions.html#facts)
- RPC 方法：[`sendTransaction`](https://solana.com/docs/rpc/http/sendtransaction)、[`simulateTransaction`](https://solana.com/docs/rpc/http/simulatetransaction)、[`getLatestBlockhash`](https://solana.com/docs/rpc/http/getlatestblockhash)
- [Solana 上的区块优化](https://solana.com/news/block-optimization-on-the-solana-network)
- [Solana 验证者 101：Jito Labs 的交易处理](https://www.jito.wtf/blog/solana-validator-101-transaction-processing/)
- [Solana 验证者中的交易处理单元](https://docs.solanalabs.com/validator/tpu)
- [Solana 的调度程序](https://www.youtube.com/watch?v=R7hq8ampBio)
- [Solana 的交易确认](https://solana.com/docs/advanced/confirmation)
- [重试交易 – Solana](https://solana.com/docs/advanced/retry)
- [如何使用优先费用](https://solana.com/developers/guides/advanced/how-to-use-priority-fees)
- [如何优化计算](https://solana.com/developers/guides/advanced/how-to-optimize-compute)
- [优先费用和计算单元](https://solana.com/docs/more/exchange#prioritization-fees-and-compute-units)
- [耐用交易 Nonces](https://docs.solanalabs.com/implemented-proposals/durable-tx-nonces) 
- [使用 Nonces 的耐用和离线交易签名](https://solana.com/developers/guides/advanced/introduction-to-durable-nonces#durable-nonces-with-solana-object-object)
- [Helius 的 Solana 交易：耐用 Nonces](https://www.helius.dev/blog/solana-transactions)

