# Solana 本地费用市场的真相

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Flocal-fee-markets.png&w=3840&q=90" alt="Local Fee Markets">

感谢 Eugene Chen 和 0xIchigo 审阅本文的早期版本。

## 可行性洞察

- 本地费用市场（Local Fee Markets，LFMs）使 Solana 能够根据状态的竞争程度为每个状态设置精细的费用。交易根据它们写入的具体状态支付费用，防止局部热点导致整个区块链的费用上涨。

- LFMs 对于实现 Solana 的愿景至关重要，即一个所有应用程序能无缝共存的可扩展统一基础层。如果没有 LFMs，链上某一部分的费用飙升会导致所有交易的费用增加——这是其他仅依赖全局费用市场进行区块空间定价的网络中常见的问题。

- 随着 Solana 上的经济活动在 2023 年底开始加速，LFMs 原始实现中的几个关键缺陷变得明显。最显著的问题是调度器优先级排序的不确定性。交易主要按照到达区块构建者的时间顺序排序，优先费用仅作为次要考虑因素。

- 在 2024 年 5 月的 Agave 客户端 v1.18 更新中，引入了新的交易调度器和改进的交易优先级公式。调度器构建依赖关系图以更好地管理跨线程的冲突交易的处理和优先级排序。这次重大更新显著提高了协议确定性排序交易的能力。

- 评估 LFMs 有效运作的一个重要指标是交易优先费用的中位数和平均值的比较。涉及无竞争状态的费用（50% 分位数中位数）预计会保持在较低水平。而竞争状态的费用应随需求增加而飙升，从而拉高平均值。最新数据证实了这一模式。2024 年 11 月，非投票交易的平均费用达到历史新高，超过 0.0003 SOL。然而，中位数费用保持稳定在 0.00000861 SOL，约低 35 倍。

- 如今，Solana 的本地费用市场虽然可以运作，但仍有很大的改进空间。Anza 工程师对银行阶段线程负载的分析表明，调度器存在一个 bug，导致验证节点客户端无法充分利用其全部容量。因此，Agave 客户端只能发挥其潜力的一小部分。此外，目前还没有关于如何对交易进行排序的正式规范。

- 当前的优先费用 API 缺乏为开发者提供确定性结果所需的复杂性。每个主要的 RPC 提供商都提供自己定制的优先费用 API，这可能会形成一种软性的供应商锁定。核心开源 RPC API 实现未能考虑关键的网络动态，如 Jito 的影响，导致费用估算不准确。

- 由于缺乏计算优先费用的确定性方法，开发者通常采取谨慎的方式，通过超额支付来确保其交易被处理。或者，他们可能过度使用 Jito 小费作为替代机制，即使对于不需要确保区块顶部位置的交易也是如此。

- 已经提出了多种策略来进一步改进 Solana 的费用结构。这些包括指数写锁费用和动态基础费用。网络尚未找到在保持真实用户低费用的同时，通过经济反压力来抑制垃圾交易的方法。

## 简介

费用市场是一种经济机制，通过动态调整交易费用，将稀缺的区块空间高效分配给最高价值的交易。交易愿意支付的费用可以作为其价值的代理指标。本地费用市场（LFMs）通过根据状态的竞争程度为每个状态设置精细的费用，进一步完善了这一概念。当两个交易访问相同的状态时（无论是两次写入还是一次读取和一次写入同一账户），它们就被视为存在竞争。

在 LFMs 中，交易根据它们写入的具体状态支付费用，防止局部热点导致整个区块链的费用上涨。访问高需求或竞争状态的交易会产生更高的费用，而与低需求状态交互的交易则支付较低的费用。这一点很重要，因为由于并行执行的特性，Solana 能更好地处理非竞争交易。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Flocal-vs-global-fee-markets.PNG&w=3840&q=90" alt="Local vs Global Fee Markets">

相比之下，全局费用市场对访问网络状态采用统一的成本，这意味着所有交易无论与哪些账户交互，都平等地竞争区块空间。[以太坊在 EIP-1559 中实现的费用模型](https://eips.ethereum.org/EIPS/eip-1559)就是全局费用市场的一个典型例子。EIP-1559 根据网络需求调整动态基础费用，以维持每个区块的最佳计算（gas）使用率。随着区块容量填满，所有交易的费用都会增加。钱包根据当前基础费用和交易的 gas 限制计算费用。这种方法在协议层面强制执行，提供可预测的费用计算；然而，它无法将高需求热点与更广泛的网络隔离开来。当费用飙升时，*所有*交易的费用都会上涨。

对特定状态的高需求问题并非区块链所独有。这个挑战类似于热点键问题，在 Web2 社交应用中通常被称为"名人问题"。

通过本文，我们旨在提供对 Solana 本地费用市场的易于理解的分析。本文分为以下几个部分：

- **Solana 费用基础：** 为读者建立对当前 Solana 交易处理方式的基本认识。
- **本地费用市场的早期问题：** 探讨本地费用市场早期实现中的初始问题及其不足。
- **中央调度器 v.1.18 更新：** 重点介绍 2024 年一个显著改善本地费用市场功能的关键更新。
- **衡量本地费用市场的有效性：** 提供相关数据以了解本地费用市场在当前 Solana 上的运行状况。
- **持续存在的问题和改进空间：** 本节讨论尚未解决的问题以及本地费用市场实现其全部潜力所需关注的领域。
- **建议的解决方案：** 回顾为完善本地费用市场和引入更细致的区块空间定价而提出的解决方案。

已经熟悉 Solana 交易费用结构的读者可以跳过下一节关于费用基础的内容。

## Solana 费用基础

[Solana 交易包含两种费用——基础费用和优先费用。](https://www.helius.dev/blog/priority-fees-understanding-solanas-transaction-fee-mechanics) 基础费用目前固定为每个签名 5,000 lamports。大多数 Solana 交易都有一个签名。优先费用以微 lamports（即 lamport 的百万分之一）每请求计算单元（CU）计价。费用从付费账户（签名者）中扣除。如果付款人没有足够的 lamports 支付交易费用，交易将被丢弃。在撰写本文时，基础费用和优先费用的 50% 由区块构建者保留作为将交易纳入区块的激励。另外 50% 被销毁。根据去年 5 月成功通过的[提案 SIMD-096，这将改变为 100% 的优先费用由区块构建者保留](https://www.helius.dev/blog/agave-v2-update#reward-full-priority-fee-to-validators)。例如：

一笔交易有一个签名并请求 500,000 CU。发送者设置每个请求 CU 的优先费用为 50,000 微 lamports。该交易的总费用为 5,000 lamports + (500,000 请求 CU \* 50,000 微 lamports 每请求 CU) = 25,000 lamports，即 0.000025 SOL。

验证者的计算资源是有限的，协议将每个区块的总计算资源限制在 4800 万 CU。这个数字是根据验证者在 400 毫秒区块时间内能够合理处理的量经验性选择的。每个账户每个区块的最大 CU 限制为 1200 万，每笔交易的最大计算量设置为 140 万 CU。交易消息也被限制在最大 1,232 字节的大小，这是 IPv6 的最小传输单元（1280 字节）减去头部。

为防止计算资源被滥用，Solana 上的每笔交易都被分配一个计算预算。默认情况下，网络为每条指令设置 200,000 计算单元（CU）的最大限制。但是，交易可以通过包含 **`SetComputeUnitLimit`** 指令来指定自定义计算单元限制，从而实现更高效的资源分配。[Agave 客户端代码库列出了各种操作的 CU 成本](https://github.com/anza-xyz/agave/blob/b7bbe36918f23d98e2e73502e3c4cba78d395ba9/program-runtime/src/compute_budget.rs#L135C1-L177C44)。

Solana 要求所有交易指定在交易期间将被读取或写入的完整账户地址列表。这个列表的最大大小是 35 个地址，可以[通过链上地址查找表扩展](https://www.helius.dev/blog/the-solana-programming-model-an-introduction-to-developing-on-solana#what-are-versioned-transactions)。构建地址列表为开发者创造了额外的开销，但这是解锁 Solana 许多优化的关键，包括并行交易执行和本地费用市场。

## Solana 本地费用市场的早期问题

随着 Solana 上的经济活动在 2023 年底开始加速，LFMs 原始实现中的几个关键缺陷变得明显。在此期间，Ellipsis Labs 的 Eugene Chen 在 [Umbra Research 文章《Solana 费用，第一部分》](https://www.umbraresearch.xyz/writings/solana-fees-part-1)中对这些挑战进行了全面分析。以下是 Chen 提出的关键要点总结。

### 缺乏准确请求计算单元的激励

Solana 的费用结构按签名收取基础费用，而不考虑使用或请求的计算单元（CUs）。同时，优先费用仅在网络拥堵期间提供有限的激励来减少 CU 使用。这种设计使交易发送者几乎没有动力去优化计算使用或使其 CU 请求与实际需求相匹配。因此，交易经常过度请求 CUs，导致网络调度过程效率低下。

### 使用协议外优先机制的激励

销毁 50% 的优先费用激励交易发送者通过与区块构建者串通并安排链下支付来绕过协议获取优先访问权。这种行为在 Jito 拍卖的日益增长的使用中表现得很明显。运行 Jito-Agave 客户端的验证者从更高的费用收入中受益，并可以通过 Jito MEV 佣金奖励有效地将这些利润分配给委托质押者。随着 Jito-Agave 客户端的采用不断增长，Jito 捆绑包在许多场景下已被证明是一种更优的交易传输服务。

### 非确定性调度器优先级

Solana 的共识和调度器都不会根据优先费用强制执行严格的交易排序。交易主要按照到达区块构建者的时间排序，优先费用仅作为次要考虑因素。更高的优先费用可以增加在竞争状态下被包含的可能性，但排序过程仍然是非确定性的。在到达交易处理单元（TPU）之前的网络抖动和调度器内部的抖动进一步增加了不可预测性。

这种确定性的缺乏降低了交易执行的可预测性和可靠性，驱使用户通过广播垃圾交易来提高更快被包含的机会。然而，在超过某个阈值后，提高优先费用的效果会递减，削弱了其作为改善交易位置机制的有效性。Solana 的共享区块空间最终成为了经典的"公地悲剧"的受害者。个体行为者为了自身利益，导致了这一公共资源的过度使用和低效。

## 中央调度器 v.1.18 更新

Agave 客户端调度器的初始实现仅提供了一个松散的保证：具有高优先费用的交易在给定区块中有更好的被包含机会。领导者的交易处理单元（TPU）使用六个并行线程运行：四个处理非投票交易，两个保留用于投票交易。四个非投票交易线程中的每一个都维护自己的队列，传入的交易在此等待分组以供执行。此前，交易被随机分配到这些线程，队列独立地对数据包进行优先级排序，而不了解其他线程处理的数据包。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fscheduler-banking-stage-TPU.PNG&w=3840&q=90" alt="Scheduler Banking Stage TPU">

在这个系统中，每个线程都会循环处理其队列，尝试锁定和执行交易。当一个线程完成当前循环后，它会收集更多的数据包并重新开始这个过程。这种结构给优先费用的有效使用带来了挑战。例如，当一个高优先级交易位于一个线程队列的顶部时，另一个线程可能同时从其队列末尾处理涉及相同账户的低优先级费用交易。优先费用仅影响单个线程内的交易排序（线程内），而不是跨所有线程（线程间）。因此，每个队列都采用了一种混合排序机制，将先进先出（FIFO）处理与优先费用考虑相结合。然而，线程之间并没有强制执行全局排序。

当线程准备执行交易时，它必须首先获取所需的账户锁。如果无法获得必要的写锁，交易将被重新排队。交易随机分配给线程的做法加剧了这个问题，因为相同类型的交易可能在多线程调度系统中出现在不同位置。调度器的这种随机性引入了抖动，造成交易在区块中的位置出现变化。

随着 2024 年 5 月 Agave 客户端更新 v1.18 的发布，一个新的交易调度器（又称中央调度器）出现了。在这个修订后的结构中，中央调度器构建了一个依赖图（dependency graph，称为优先级图，prio-graph），以更好地管理所有线程间冲突交易的处理和优先级排序。这个重大更新显著提高了 Solana 确定性排序交易的能力；具有更高优先费用的交易更有可能被包含在区块中。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fpriograph-central-scheduler.PNG&w=3840&q=90" alt="Priograph Central Scheduler">

要了解更多关于中央调度器的技术细节，请参阅 [Andrew Fitzgerald 的深入技术分析](https://apfitzge.github.io/posts/solana-scheduler/)。

优先级图是一个有向无环图（DAG），会随着新交易的添加而动态更新。交易在图中被组织成执行链，按时间优先级顺序处理。对于冲突的交易，优先费用决定插入顺序。这种方法最小化了锁竞争，使交易批次能够顺畅执行，减少了由资源冲突引起的延迟。交易预编译验证已被分配给工作线程处理，以提高性能并实现更高效的处理。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fpriograph-visulization.PNG&w=3840&q=90" alt="Priograph Visulization">

要了解更多关于优先级图的可视化，请参阅 [Andrew Fitzgerald 的深入技术分析](https://apfitzge.github.io/posts/solana-scheduler/)。

更新后的调度器设计显著提高了可扩展性和灵活性，允许在不增加锁冲突风险的情况下增加线程数量。此外，集中式调度方法改善了奖励生成，提高了许多验证者运营商的收益。

关于中央调度器的更详细分析，读者可以参考我们早期的 [Helius 博客文章，其中介绍了 Agave 1.18 更新](https://www.helius.dev/blog/all-you-need-to-know-about-solanas-v1-18-update)。

### 更有效的优先级计算

随着调度器的更新，交易优先级公式也进行了优化，为计算需求较低的交易提供优势，使开发者和资源使用最少的交易受益。

修订后的公式为：

$$
\text{优先级} = \frac{(\text{优先费用} \times \text{请求计算单元}) + \text{基础费用}}{1 + \text{请求执行 CUs} + \text{签名 CUs} + \text{写锁 CUs}}
$$

这个新计算方法考虑了与交易相关的所有计算和操作成本，确保优先级水平准确反映实际资源消耗。因此，不需要额外优先费用的简单代币转账或原生 SOL 交易在队列中保证有基准优先级水平。对于更复杂的交易，未使用 `SetComputeUnitLimit` 指令指定自定义 CU 限制的开发者在交易优先级排序中相比使用该指令的开发者处于劣势。

## 衡量本地费用市场有效性

本节将研究与 Solana 本地费用市场相关的数据。

### 中位数与平均交易费用

在有效运作的本地费用市场中，涉及无竞争状态的交易（如简单的稳定币转账）的费用预计将保持在较低水平。同时，访问竞争状态（如投机性低流动性代币）的交易费用应随需求增长而上涨。评估这种动态的一个重要指标是中位数和平均交易优先费用的比较。**中位数费用**代表第 50 百分位用户支付的费用，反映了典型成本，而**平均费用**则是所有费用除以交易总数，突显整体趋势。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fsolana-transaction-fee-median-vs-average.PNG&w=3840&q=90" alt="Solana Transaction Fee Median vs Average">

更多关于 Solana 交易费用中位数和平均值的历史数据可以在 [Blockworks Research](https://solana.blockworksresearch.com/?dashboard=sol-onchain-activity&currency=SOL&interval=weekly) 查看。

最新数据证实了这种预期模式。2024 年 11 月标志着迄今为止 Solana 经济活动的最高水平，非投票交易的平均费用达到历史新高，超过 0.0003 SOL。尽管如此，中位数费用仍保持稳定在 0.00000861 SOL，大约低 35 倍。这与 2024 年 4 月形成对比，当时类似的经济活动激增导致平均费用上升至 0.0002 SOL 以上，同时中位数费用相应增加到 0.00001862 SOL，大约低 10 倍。这种差异凸显了费用隔离在高需求期间保护普通用户免受成本飙升影响，并为非投机驱动的使用场景保持用户体验方面的有效性。

当分析类似的基于 EVM 的网络（如 Coinbase 运营的以太坊 L2 Base）的数据时，我们观察到中位数和平均交易费用之间存在强相关性，这是由于缺乏本地费用市场（LFMs）。由于需求增加时全局基础费用上涨，平均费用和中位数费用相对同步变动。此外，中位数和平均交易费用之间的差距明显较小。例如，在 2024 年 12 月 5 日，Base 上的平均交易费用激增至 0.1115 美元，而中位数费用也上升至 0.0228 美元——大约低 5 倍。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Ftransaction-fees-base.PNG&w=3840&q=90" alt="Transaction Fees on Base">

以太坊 L2 Base 链上交易费用中位数与平均值的历史数据可以在 [Blockworks Research](https://optimism.blockworksresearch.com/base?dashboard=base-onchain-activity) 查看。

### 交易回滚率

另一个值得研究的趋势是交易回滚率。在 2024 年 4 月和 5 月经济活动高峰期间，由于链遭受大量垃圾交易的影响，Solana 用户普遍反映用户体验下降。确定性的缺乏降低了交易执行的可预测性和可靠性，促使用户通过发送垃圾交易来提高更快被打包的机会。

搜索者经常在不考虑成功概率的情况下提交投机性交易。优先费用不合理偏低的套利交易仍然有效。协议会在处理完其他优先费用更高的交易后处理这些交易，而这些交易很可能会因为滑点逻辑而回滚。

交易回滚率在 2024 年 4 月达到峰值，占所有非投票交易的 75.7%。在推出包括 Agave 1.18 中央调度器在内的关键更新后，这一比例显著下降。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Freverted-transaction-rates.png&w=3840&q=90" alt="Reverted Transaction Rates">

Solana 交易回滚率的历史数据可以在 [Blockworks Research](https://solana.blockworksresearch.com/?dashboard=sol-onchain-activity&currency=SOL&interval=weekly) 查看。

[Blockworks Research 的群组分析](https://solana.blockworksresearch.com/?dashboard=sol-onchain-activity&currency=USD&interval=weekly)覆盖过去七天（2024 年 1 月 6-13 日）的数据显示，不同活动水平的地址有着不同的交易回滚率。每日进行 1-5 笔交易的地址（主要是零售用户）的回滚率为 1.4%，而每日进行 6-50 笔交易的地址回滚率增加到 4.6%。值得注意的是，每日进行超过 10,000 笔交易的地址的回滚率飙升至 66.7%。此外，每日进行超过 100,000 笔交易的高活动地址（机器人）占所有回滚交易的 95.2%。2024 年 12 月，所有非投票交易的整体回滚率为 41.2%，这表明网络的大量计算资源被用于处理失败的套利交易。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fnon-vote-transactions-reverted-successful.png&w=3840&q=90" alt="Non-Vote Transactions Rerverted vs Successful">

2024 年每个星期的交易回滚率可以在 [Blockworks Research](https://solana.blockworksresearch.com/?dashboard=sol-onchain-activity&currency=SOL&interval=weekly) 查看。

## 待解决问题和改进空间

尽管取得了显著进展，Agave 验证节点客户端调度器仍面临挑战。[Anza 工程师 Alessandro Decina 对银行阶段线程工作负载进行的以下分析](https://x.com/alessandrod/status/1872533928172769574)突显了现有的低效问题和需要改进的领域。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fbanking-stage-threads.png&w=3840&q=90" alt="Banking Stage Threads">

**调度线程:** 这是生产区块最关键的线程。调度器接收所有入站交易，然后对它们进行排序并安排执行。

**投票交易线程:** 两个专用线程处理投票交易，确保它们与用户交易分开处理。

**非投票交易线程:** 四个线程接收由调度器安排的交易并处理用户交易。

**QUIC 摄入线程:** 当验证节点是领导者时，Tokio 线程通过 QUIC 协议管理交易摄入。在 2024 年初的拥堵期间，这些线程成为重要瓶颈。

上述可视化显示，虽然在领导者第一个区块开始时所有工作线程都并行执行交易，但这种并行性很快就降级为顺序执行。具体来说，只有一个非投票交易线程（线程三）继续处理交易，而其余线程处于空闲状态。

这种行为表明调度器存在 bug，导致验证节点客户端无法充分利用其全部容量。因此，系统仅以其潜力的一小部分运行，这表明如果解决该问题，验证节点可以处理当前负载的四倍。

## 可观察性

当前用于估算可预测交易落地的费用 API 缺乏提供确定性结果所需的复杂性。每个主要的 RPC 提供商都提供自己定制的优先费用 API，而[核心开源 RPC API 实现仍然不够理想](https://github.com/anza-xyz/agave/issues/3332)。它未能考虑关键的网络动态，如 Jito 的影响，导致费用估算不够准确。

[Helius 提供 **`getPriorityFeeEstimate`** RPC 方法](https://docs.helius.dev/guides/priority-fee-api)，该方法基于全局和本地费用市场的历史数据提供费用建议。开发人员可以输入序列化的已签名交易或交易涉及的账户密钥列表。该方法支持自定义优先费用级别，分为六个百分位：*最低、低、中等、高、很高*和*不安全最高*。中等（第 50 百分位）被设置为默认建议。费用使用最近 50 个时隙的数据计算。

```json
{
  "jsonrpc": "2.0",
  "id": "helius-example",
  "method": "getPriorityFeeEstimate",
  "params": [
    {
      "transaction": "LxzhDW7T...", // Base58 encoded serialized transaction
      "options": {
        "recommended": true
      }
    }
  ]
}
```

上图：使用 base58 编码的序列化交易的 `getPriorityFeeEstimate` 示例负载。

由于缺乏确定性的优先费用计算方法，开发人员经常采取谨慎的方式，通过超额支付来确保其交易得到处理。或者，他们可能过度使用 Jito 小费，即使在不需要确保区块顶部位置的交易中也是如此。这些小费经常被用作优先费用的替代品。值得注意的是，[2024 年观察到的大多数小费并不与传统的 MEV 活动（如套利或夹子交易）相关](https://www.helius.dev/blog/solana-mev-report)，而是旨在实现更快的交易确认。验证节点通过收取更高的区块奖励和 MEV 佣金从这种低效中获益。

另一个挑战出现在开发人员没有实现根据波动的链上条件动态调整优先费用的逻辑时。在重大事件期间，如显著的市场波动，访问特定状态账户的费用可能会急剧上升。在这些情况下，缺乏动态费用机制的应用程序将会遇到困难，因为它们的静态费用设置不足以确保及时执行。

## 提议的解决方案

已经提出了多种策略来进一步改进 Solana 的费用结构。这些提案旨在优化网络资源分配并减少垃圾交易的动机。

### 指数写锁费用

[由 Tao Zhu (Anza) 和 Anatoly Yakavenko 于 2023 年 1 月提出的 SIMD-0110](https://github.com/solana-foundation/solana-improvement-documents/pull/110) 提出了一种通过对争用账户收取动态费用来管理拥堵的新机制。该机制跟踪写锁定账户的计算单元 (CU) 使用率的指数移动平均值 (EMA)，并增加持续表现出高使用率的写锁定账户的成本。

要实现这样的系统，Solana 运行时维护一个争用账户公钥及其对应计算单元定价器 (CUPs) 的 LRU（最近最少使用）缓存。CUPs 监控账户的 EMA CU 使用率，并在查询时提供更新的成本率。

该机制动态调整写锁费用。如果账户的 EMA CU 使用率超过目标阈值，写锁成本率就会增加。相反，如果使用率低于目标值，成本率就会降低。初始参数包括：

- 账户最大 CU 限制的 25% 的目标使用率。
- 每 CU 1,000 微 lamports 的初始写锁成本率。
- 每个区块 1% 的成本调整率。

账户的写锁费用通过将其成本率乘以交易请求的 CU 数来计算。在这个系统下，总交易费用是三个组成部分的总和：基本签名费用、优先费用和写锁费用。写锁费用 100% 销毁。

在发布时，SIMD-0110 在社区内引发了热烈讨论。然而，该提案目前处于非活动状态，并已被标记为关闭。

### 动态基础费用

改进 Solana LFM 的另一个长期解决方案是引入全局和每账户动态基础费用（DBFs）。[Ellipsis Labs 的 Jarry Xiao 和 Eugene Chen 是这种方法的主要支持者](https://www.umbraresearch.xyz/writings/toward-multidimensional-solana-fees)。

虽然优先费用是可选的，但基础费用是强制性的。目前，Solana 的基础费用固定为每个签名 5000 lamports。提交简单代币转账的用户与进行复杂多场所交换或尝试执行复杂 MEV 套利的搜索者支付相同的基础费用。基础费用并不能准确反映交易的计算使用情况。

使用动态基础费用，具有不适当基础费用的套利交易可以在到达调度器之前被视为无效并丢弃。提高基础费用可以鼓励垃圾邮件发送者减少发送交易。

基础费用最终将达到平衡，交易将根据区块空间市场的价值定价。由于基础费用在不断上涨，它最终将达到边际成本，此时发送交易的机会成本将不再值得。费用不能变得太高，否则将影响用户活动。对机器人来说太高但对用户来说普遍可接受的最高限额是理想的。在这样的系统下，为了包含而发送垃圾交易的账户将烧掉所有的 SOL。

Solana 的快速出块时间使得可以采用激进的算法来设置基础费用。在高需求期间，费用可以迅速调整——可能每个区块翻倍——以反映网络拥堵情况。相反，当需求减少时，费用可以更 gradually 地降低。得益于 Solana 的短区块时间，费用降低仍然发生得相对较快，确保网络能够快速适应不断变化的条件。

类似的经济反压机制的一个例子是 [Metaplex Candy Machine 程序](https://developers.metaplex.com/candy-machine/guards/bot-tax)，该程序在 2022 年引入了机器人税作为反垃圾邮件机制。机器人税是对无效交易的可选收费。通常，这个金额会相对较小，以避免影响做出真实错误的普通用户。这项税收证明是有效的；铸造狙击者很快就被耗尽，垃圾邮件也停止了。

## 结论

Solana 的 LFM 是可行的，但仍有很大的改进空间：

- **改进优先费用机制：** 优先费用 RPC 调用需要改进。理想情况下，开发人员应该有一个简单、确定性的方式来设置费用，以保证交易在随后的几个区块内被包含。
- **经济上抑制垃圾交易：** 网络必须找到方法，在高经济活动期间对机器人施加经济反压，同时为真实的人类用户保持低费用。
- **教育开发人员：** 开发人员需要停止设置静态应用程序交易费用，并减少对 Jito 等协议外机制的依赖来处理常规交易。
- **进一步优化调度器：** 交易调度器需要进一步优化，以确保在高需求期间充分利用所有工作线程。

正如 Solana 联合创始人 Anatoly Yakovenko 指出的那样，这些挑战主要是"仅仅是工程问题"——只要有适当的技术重点就可以解决。

## 附加资源

- [Solana 费用，第一部分](https://www.umbraresearch.xyz/writings/solana-fees-part-1) - Umbra Research
- [走向多维度的 Solana 费用](https://www.umbraresearch.xyz/writings/toward-multidimensional-solana-fees) - Umbra Research
- [本地费用市场对以太坊扩容的必要性](https://www.eclipse.xyz/articles/local-fee-markets-are-necessary-to-scale-ethereum) - Eclipse Labs
- [Solana 的本地费用市场并不真实 | Eugene Chen](https://www.youtube.com/watch?v=Bhh4chj-J0I) - Lightspeed Podcast
- [Solana 银行阶段和调度器](https://apfitzge.github.io/posts/solana-scheduler/) - A.Fitzgerald
