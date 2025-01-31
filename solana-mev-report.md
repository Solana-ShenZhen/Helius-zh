# Solana MEV 报告：趋势、洞察与挑战

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fsolana-mev-report-2025-helius.png&w=3840&q=90" alt="Solana MEV Report">

感谢 Lucas Bruder、Max Resnick、Eugene Chen、Mert Mumtaz、0xIchigo、Uri Klarman 和 Nitesh Nath 审阅本文的早期版本。

## 可行性洞察

- 由于 Solana 独特的架构和缺乏全局内存池，其 MEV 的运作方式与其他区块链网络不同。协议外的内存池必须独立开发，需要网络中大部分质押者的采用才能有效运作，这带来了较高的技术和社会壁垒。

- Jito 在 2024 年 3 月暂停了其公共内存池，导致收入大幅下降。这一举措立即减少了有害的 MEV 行为。然而，这一决定也促使缺乏透明度的替代内存池兴起，这些内存池主要让少数拥有独家访问权的群体受益。

- 迷因币交易者特别容易受到三明治攻击，因为他们在交易流动性低且波动性高的资产时会设置较高的滑点容忍度。这类用户倾向于使用 Telegram 交易机器人以获得更快的执行速度和实时通知。迷因币交易者对其交易被抢跑的情况相对不敏感。

- Marinade Finance 的质押拍卖市场（Stake Auction Marketplace，SAM）采用竞争性拍卖机制，验证者在"付费质押"（pay-for-stake）系统中直接相互竞价以获得质押分配。该程序受到批评，因为它使参与用户三明治攻击的验证者能够通过出价超过其他验证者，从而获得更多质押并增加其在网络中的影响力。

- Solana 上的大部分三明治攻击来自单一实体 DeezNode 运营的私有内存池。DeezNode 运营的一个关键验证者（地址为 [HM5H6 …jdMRA](https://stakewiz.com/validator/HM5H6FAYWEMcm9PCXFbbiUFfFVLTN9UGy9AqmMQjdMRA)）目前持有 811,604.73 SOL 的委托质押，价值约 1.685 亿美元。该验证者的委托质押出现大幅增长，从 11 月 13 日（第 697 个 epoch）的 307.9k SOL 增长到 12 月 9 日（第 709 个 epoch）的 802.5k SOL。此后，增长趋于稳定。值得注意的是，其中 19.89% 的质押来自 Marinade 的 mSOL 流动性质押池和原生委托。

- 多个 Solana 验证者运营商已[公开报告收到参与私有内存池的诱人提议](https://x.com/durdenwannabe/status/1770787171688923225)，包括详细说明利润分成和预期收益的文件。

- Jito 捆绑包是搜索者确保获利交易排序的主要方法。然而，Jito 数据并未捕获 MEV 活动的全貌；特别是，它无法捕获搜索者的利润或通过替代内存池发生的活动。此外，许多应用程序出于非 MEV 目的使用 Jito，绕过优先费用以确保及时的交易包含。

- 在过去一年中，处理了超过 30 亿个 Jito 捆绑包，总计产生了 375 万 SOL 的小费。这一活动呈现明显的上升趋势，从 1 月 11 日的 781 SOL 小费低点上升到 11 月 19 日的 60,801 SOL 高点。

- Jito 的套利检测算法分析了所有 Solana 交易（包括 Jito 捆绑包之外的交易），在过去一年中识别出 90,445,905 笔成功的套利交易。每笔套利的平均利润为 1.58 美元，单笔最高利润达到 370 万美元。这些套利总共产生了 1.428 亿美元的利润，其中 1.267 亿美元（88.7%）以 SOL 计价。

- DeezNode 内存池运营着一个地址为 [vpeNAL..oax38b](https://xray.helius.dev/address/vpeNALD89BZ4KxNUFjdLmFXBCwtyqBDQ85ouNoax38b/history?cluster=mainnet-beta&page=1) 的三明治机器人。Jito 的内部分析表明，Solana 上近一半的三明治攻击都可以归因于这个单一程序。在 30 天内（12 月 7 日至 1 月 5 日），该程序执行了 155 万笔三明治交易，获利 65,880 SOL（1,343 万美元）。每次攻击的平均获利为 0.0425 SOL（8.67 美元）。将这些数据年化，该程序每年将产生 801,540 SOL 的利润。在网络中心化的最坏情况下，如果 100% 的利润都被重新投资，他们在网络中的质押份额将增加 0.2%。

- 这个机器人只是执行三明治攻击的众多链上程序之一。要实时查看 Solana 上检测到的三明治攻击，请访问 [sandwiched.me](http://sandwiched.me/)。

- 验证者白名单通常被视为对抗恶意行为者的最后手段。它们可能会创造一个半许可和审查的环境，这直接与行业去中心化的理念相冲突。在某些情况下，这种方法还可能延迟交易处理，造成次优的用户体验。

- 防三明治攻击的自动做市商（Sandwich-resistant AMMs，简称 sr-AMMs）是在传统恒定乘积 AMM 基础上的实验性设计。在防三明治 AMM 中，所有交易的执行价格都不会优于槽位窗口开始时的池子价格。这种机制有效地消除了三明治攻击的获利空间。Ellipsis Labs 发布了 [Plasma，一个经过审计的防三明治 AMM 设计参考实现](https://www.ellipsislabs.xyz/blog-posts/introducing-plasma)。

- 多并发领导者（Multiple Concurrent Leaders，MCL）通过允许用户在不产生延迟的情况下在领导者之间进行选择，为缓解有害的 MEV 提供了一个有前景的长期解决方案。如果领导者 A 表现恶意，用户可以将其交易重定向到诚实的领导者 B。然而，MCL 的实施预计需要数年的开发时间。

## 简介

最大可提取价值（Maximal Extractable Value，MEV）是通过操纵交易排序可以提取的价值。这包括在区块内添加、删除或重新排序交易。MEV 以各种形式出现，但都有一个共同点：它们依赖于交易排序。搜索者（监控链上活动的交易者）试图在其他交易之前或之后策略性地下单以提取价值。

在 Solana 上，由于其独特的架构和缺乏全局内存池，MEV 的运作方式与其他区块链网络不同。诸如[传播状态更新的 Turbine](https://www.helius.dev/blog/turbine-block-propagation-on-solana) 和[用于交易转发的质押加权服务质量（SWQoS）](https://www.helius.dev/blog/stake-weighted-quality-of-service-everything-you-need-to-know)等特性塑造了其处理 MEV 的方式。Solana 的快速流式区块生产不依赖外部附加组件或协议外拍卖机制，这缩小了某些类型 MEV（如抢跑）的传统方法的适用范围。为了获得优势，搜索者运营自己的节点或与高质押验证者合作以访问最新的区块链状态。

MEV 已成为一个含义过载的术语，对其精确定义存在不同观点。与普遍认知相反，并非所有 MEV 都是有害的。由于区块链的分布式和透明特性，完全消除 MEV 被广泛认为是不太可能的。声称已消除 MEV 的网络要么缺乏足够的用户活动来吸引搜索者，要么采用随机区块打包等技术，这些技术虽然表面上缓解了 MEV，但可能会激励垃圾交易。

三明治攻击是最受关注且对用户有害的 MEV 形式。在这种策略中，搜索者在目标交易之前和之后各放置一笔交易以提取价值。虽然对搜索者有利可图，但三明治攻击会增加交易成本并恶化普通用户的执行价格。我们将在后面的章节详细讨论三明治攻击。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fsolana-mev-sandwich-attack-visualization.png&w=3840&q=90" alt="Solana MEV Sandwich Attack Visualization">

本报告将分析 Solana 当前的 MEV 格局。报告分为四个部分：

1. **Solana MEV 时间线**：按时间顺序列出关键事件，为不太熟悉 Solana 上 MEV 快速发展的读者提供有价值的背景信息。
2. **MEV 形式**：探讨目前在 Solana 上观察到的各种 MEV 形式，并提供具体详细的示例。
3. **Solana MEV 数据**：本节提供相关的、可量化的和具有上下文的数据，以说明 MEV 在 Solana 中的当前范围和影响。
4. **MEV 缓解机制**：研究正在考虑用于减少或消除有害 MEV 形式的策略和机制。

虽然建议按顺序阅读这些章节，但每个章节都可以独立阅读。

## Solana MEV 时间线

以下是与 Solana MEV 格局相关的重要事件时间线。

### 2021 年 9 月至 2022 年 4 月 - 垃圾交易和 DDoS 攻击

NFT 是 Solana 上首个获得显著发展的领域。NFT 领域的 MEV 主要出现在公开铸造活动期间，参与者竞相获取稀有或有价值的资产。这些活动为搜索者创造了突然且极端的机会，在铸造前的区块中没有 MEV 潜力，而在铸造后的区块中则有巨大的 MEV 潜力。NFT 铸造机制是导致 Solana 出现大规模拥堵峰值的最早原因之一，机器人发送的垃圾交易使网络不堪重负，导致区块生产暂时停止。

### 2022 年中期 - 优先费用的引入

Solana 实现了[可选的优先费用，用户可以在计算预算指令（compute budget instruction，一种用于设置交易资源限制的系统指令）中指定该费用以优先处理其交易](https://www.helius.dev/blog/priority-fees-understanding-solanas-transaction-fee-mechanics)。这一机制通过允许用户在高活动期间支付加急处理费用来帮助缓解网络拥堵。它还为费用市场建立了一个更高效的框架，增强了网络的经济模型。

优先费用通过改变竞争格局来帮助阻止垃圾交易。以前依靠暴力交易量来获取优势的机器人不能再仅仅通过发送垃圾交易来主导。相反，优先级还取决于用户愿意支付的费用。

### 2022 年 8 月 - Jito-Solana 客户端发布

[Jito 已成为 Solana MEV 的默认基础设施](https://www.helius.dev/blog/solana-mev-an-introduction#solana's-mev-structure)。该客户端旨在实现 MEV 捕获的民主化，确保网络中奖励的更公平分配。当领导者使用 Jito 客户端验证器时，他们的交易最初会被定向到 Jito-Relayer，它充当交易代理路由器。这个中继器在将交易转发给领导者之前会持有交易 200 毫秒。这个减速带延迟了传入的交易消息，通过 Jito Block Engine 为链下拍卖提供了一个时间窗口。搜索者和应用程序提交原子执行的交易捆绑包，并附带以 SOL 为主的小费。Jito 对所有小费收取 5% 的费用，最低小费为 10,000 lamports。[可以通过 Jito 捆绑包浏览器查看捆绑包](https://explorer.jito.wtf/)。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fjito-block-engine-architecture-diagram.png&w=3840&q=90" alt="Jito Block Engine Architecture Diagram">

这种方法通过在链下运行拍卖并仅将单个获胜者写入区块，减少了垃圾交易并提高了 Solana 计算资源的效率。考虑到未成功的交易会消耗网络大量的计算资源，这一点很重要。

在最初的九个月里，由于网络活动较低且 MEV 奖励很少，Jito-Solana 客户端的采用率保持在 10% 以下。从 2023 年底开始，采用率显著加快，到 2024 年 1 月达到 50%。如今，按质押权重计算，超过 92% 的 Solana 验证者使用 Jito-Solana 客户端。

### 2024 年 1 月：迷因币季节的到来

2024 年初，网络活动激增。Bonk 和 DogWifHat 等迷因币获得了关注，引发了搜索者的浓厚兴趣，并显著增加了 MEV 活动。这一时期标志着用户行为的显著转变：迷因币交易者更倾向于使用 BonkBot、Trojan 和 Photon 等 Telegram 交易机器人，而不是传统的去中心化交易所或聚合器。这些机器人提供更快的速度、实时通知和直观的基于文本的界面，吸引了散户投机者。由于设置了高滑点率以优先处理时间敏感的交易，这些交易者对其交易被抢跑相对不敏感。

### 2024 年 3 月：Jito 暂停其旗舰内存池功能

Jito 的内存池为搜索者提供了 200 毫秒的窗口来预览所有发送给领导者的交易。在运行期间，这个系统经常被用于三明治攻击，显著降低了用户体验。为了优先考虑网络的长期增长和稳定性，Jito 做出了有争议的决定暂停其内存池，在此过程中牺牲了大量收入。虽然这一举措获得了广泛支持，但也面临来自一些重要人物的批评，包括 Mert Mumtaz 和 Jon Charbonneau。

这一决定的主要风险是可能会出现复制 Jito 功能的替代内存池，从而使更有害的 MEV 形式得以提取。与促进 MEV 机会更公平分配并缓解网络中权力不平衡的公共内存池不同，私有许可内存池在缺乏透明度的情况下运行，只让少数能够访问它们的人受益。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fdeeznode-mev-mempool-proposal-to-validators.png&w=3840&q=90" alt="DeezNode MEV Mempool Proposal to Validators">

多个 Solana [验证节点运营商报告收到了参与私有内存池的丰厚报价](https://x.com/durdenwannabe/status/1770787171688923225)。

### 2024 年 5 月：新的交易调度器

作为 [Agave-Solana 1.18 更新](https://www.helius.dev/blog/all-you-need-to-know-about-solanas-v1-18-update)的一部分，新的调度器显著提高了 Solana 确定性排序交易的能力。改进后的调度器更好地优先处理手续费较高的交易，提高了它们被纳入区块的可能性。中央调度器构建了一个称为"优先级图"（prio-graph）的依赖关系图，以优化跨多个线程的冲突交易的处理和优先级排序。

此前，参与套利和其他 MEV 活动的机器人为了提高执行成功的机会，会倾向于向领导者发送垃圾交易。旧调度器的随机性引入了抖动，导致区块内交易位置的可变性。新的确定性方法减少了这种随机性，抑制了垃圾交易，提高了网络的整体效率。

### 2024 年 6 月：Marinade 推出质押拍卖市场（Stake Auction Marketplace，SAM）

Marinade Finance 的质押拍卖市场（Stake Auction Marketplace，SAM）采用竞价拍卖机制，验证者在"付费质押"系统中直接相互竞价以获得质押分配。这种结构激励验证者出价到他们认为有利可图的最高水平。该计划面临批评，因为它使参与用户三明治攻击的验证者能够出价超过其他验证者，从而获得更多质押并增加他们在网络中的影响力。[Marinade Labs 最近提议建立一个公共委员会来监督委托](https://forum.marinade.finance/t/tackling-malicious-validators-and-democratizing-mev-on-solana/1686)。Marinade Finance 的 mSOL 是 Solana 仅次于 Jito 的第二大流动性质押代币和质押池。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fmarinade-finance-stake-auction-bids.png&w=3840&q=90" alt="Marinade Finance Stake Auction Bids">

您可以在 [PSR 浏览器](https://psr.marinade.finance/) 中查看实时的质押拍卖出价。

截至第 717 个 epoch，0% 质押佣金和 0% MEV 佣金的验证者通常为质押者提供约 9.4% 的年化收益率（APY）。使用协议外方法重新分配区块奖励的验证者一般提供 10% 或更低的 APY。相比之下，Marinade 的 SAM 拍卖显示获胜者的 APY 为 13.73%，前十名验证者的出价达到 18.27% APY。

这种差异表明，这些验证者要么在非理性竞价并承受损失，要么通过[来自 Solana 基金会的质押委托](https://www.helius.dev/blog/solana-foundation-delegation-program-sfdp)补贴他们的出价，要么通过其他收入来源（如从三明治攻击用户中获取的 MEV）来补充收入。

### 2024 年 12 月：对新私有内存池的担忧加剧

在专注于 Solana 的[研究公司 Temporal 公开提出对网络质押可能集中化的担忧](https://tempxyz.substack.com/p/solanas-largest-existential-risk)后，Solana MEV 成为一个有争议的话题。这引发了广泛的讨论，并重新激发了解决 Solana MEV 挑战的努力。

参与有害 MEV 提取的验证者获取了不成比例的价值，导致他们的质押增长速度快于其他验证者。这使得验证者能够随着时间的推移积累更大的网络影响力，给 Solana 的验证者经济带来了集中化风险。收益较高的验证者可以为质押者提供更高的回报，吸引更多的质押，从而进一步巩固其地位。

Solana 上的大部分三明治攻击来自由单一实体 DeezNode 运营的私有内存池。DeezNode 运营的一个关键验证节点（地址为 [HM5H6 …jdMRA](https://stakewiz.com/validator/HM5H6FAYWEMcm9PCXFbbiUFfFVLTN9UGy9AqmMQjdMRA)）目前持有 811,604.73 SOL 的委托质押，价值约 1.685 亿美元。该验证节点的委托质押出现大幅增长，从 11 月 13 日（第 697 个 epoch）的 307.9k SOL 增长到 12 月 9 日（第 709 个 epoch）的 802.5k SOL。此后，增长趋于稳定。[值得注意的是，其中 19.89% 的质押来自 Marinade 的 mSOL 流动性质押池和 Marinade 原生委托](https://flipsidecrypto.xyz/heliusresearch/q/6T6RDRx_n8cl/hm5h6-stakers-epoch-722/visualizations/ebef73b8-57d1-4e28-9698-fb0d1d8e0606)。该验证节点占总质押量（目前为 3.925 亿 SOL）的 0.2%，在更广泛的验证节点集合中按质押量排名第 93 位，处于超级多数子集之外。

Jito 的内部分析显示，在 Jito 拍卖机制之外发生的三明治攻击数量不断增加，这表明存在其他区块引擎或修改过的验证节点客户端在进行三明治攻击活动。

## MEV 的形式

让我们来研究 Solana 上各种类型的 MEV，并用真实交易的具体示例来说明每种类型。以下是目前在 Solana 上观察到的最普遍的 MEV 交易类型。

### 清算

当借款人在借贷协议上无法维持所需的抵押率时，他们的头寸就会面临清算风险。搜索者会监控区块链上这些抵押不足的头寸，并通过偿还部分或全部债务来执行清算，以换取部分抵押品作为奖励。清算被认为是一种良性的 MEV。它们对于维持协议的偿付能力至关重要，并有助于更广泛的链上去中心化金融生态系统的稳定性。

#### 清算交易示例

这次清算发生在 12 月 10 日，通过 Kamino（Solana 按流动性和用户基数计算最大的借贷协议）进行。该交易包含三个步骤：

- 搜索者通过向 Kamino 储备金转入 10.642 USDC 来启动清算，以覆盖用户的债务头寸。
- 作为交换，Kamino 储备金将用户的 0.05479 SOL 抵押品转给了搜索者。
- 搜索者支付了 0.0013 SOL 的协议费用。

此外，搜索者为该交易支付了 0.001317 SOL 的优先费，最终获得了 0.0492 美元的净利润。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fexample-solana-mev-liquidation-transaction.png&w=3840&q=90" alt="Example Solana MEV Liquidation Transaction">

[查看 Solscan 上的清算交易](https://solscan.io/tx/5gcWCkHxdYKJCHxrADX8KKkRN6cfwh4XkUDpH4QJdKtiatZdCGeceAFHdwDK7TqxSt2JBBykFu5c6gxcAZEN9ZFg)

### 套利

套利通过在不同交易场所之间对齐价格来提高市场效率，利用同一资产的价格差异获利。这些机会可能发生在链内、跨链或中心化与去中心化交易所（CEX/DEX 套利）之间。其中，链内套利保证了原子性，因为交易的两个部分可以在单个 Solana 交易中一起执行。相比之下，跨链和跨平台套利会引入额外的信任假设。

原子套利是 Solana 上最主要的 MEV 形式。原子套利最简单的例子是当两个 DEX 对同一交易对列出不同价格时出现。这通常涉及利用恒定乘积（xy=k）自动做市商（AMM）上的过时报价，并在链上限价订单簿上进行对冲交易，其中做市商已经调整了他们的报价以反映链下价格变动。

套利交易示例

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fexample-solana-mev-arbitrage-transaction.png&w=3840&q=90" alt="Example Solana MEV Arbitrage Transaction">

[查看 Solscan 上的套利交易](https://solscan.io/tx/5UkU17qi6THzr4By3uoF99TktLAjhJNyuYMr8mmLBWXf1XrkdNrxRBPR8pCire8xiKAxUKLuQUQXcJnY8vuU5NxK)

要了解更多关于 Solana MEV 的信息，请参阅 [Umbra Research 的研究报告](https://www.umbraresearch.xyz/writings/mev-on-solana)。

在这个场景中，SOL/USDC 交易对的价格在链下发生了变化，促使 Phoenix 做市商相应地更新了他们的报价。与此同时，Orca AMM 仍然基于过时的价格进行报价，为搜索者创造了套利机会。搜索者在 Orca 上用 45 USDC 购买了 2.11513 SOL，然后在 Phoenix 上以 45.0045 USDC 的价格卖出 2.115 SOL，获得了 0.00013 SOL（约 0.026 美元）的利润。套利交易以原子方式执行，消除了搜索者持有库存的需求。主要风险在于为失败的交易尝试支付的费用。

### 抢跑

抢跑是指 MEV 搜索者在内存池中识别出其他交易者的买入或卖出订单，并在该交易者之前下达相同的订单，从受害者交易的价格影响中获利。

当观察者注意到一笔未确认的交易可能会影响代币价格时，就会发生这种情况。观察者在原始交易处理之前利用这个信息采取行动。这种抢跑策略很直接，不像三明治攻击等其他方法那样复杂。

搜索者发现一笔待处理的买入交易会对目标代币价格产生积极影响。搜索者将自己的买入交易与目标交易打包在一起。他们的订单会在目标之前以较低的价格处理，一旦目标的交易完成，他们就能获利。在这个过程中，由于 MEV 搜索者的买入交易的影响，目标方不得不以更高的价格买入，从而遭受损失。

### 尾随交易 (Back-running)

尾随交易是抢跑的对应策略，是一种特定的 MEV 策略，利用其他交易（通常由于路由不当）造成的临时价格不平衡。一旦用户的交易执行完成，尾随搜索者通过交易相同的资产来平衡各个池子之间的价格并获取利润。从理论上讲，如果用户能够更有效地执行交易，这部分利润本可以被用户获得。

#### 尾随交易示例

这个著名的尾随交易发生在 2024 年 1 月 10 日，当时一个用户在单笔交易中购买了价值 890 万美元的 DogWifHat (WIF) 代币。当时，WIF 代币的交易价格为 0.2 美元，在所有链上交易场所的流动性总额仅有几百万美元。Jupiter 聚合器通过三个流动性有限的池子执行了这笔交易，导致价格一度飙升至 3 美元。

搜索者[使用 Jito Bundle 执行了尾随交易](https://explorer.jito.wtf/bundle/7d2f02a0542fd3950d90c9bd8ca84d233e28f0298d9f002c7e3cc0959b72b24f)，支付了高达 890.42 SOL（91,621 美元）的 Jito 小费。他们首先通过 Raydium 集中流动性池将 703.31 SOL（72,368 美元）兑换成 490,143.90 个 WIF 代币。接着，他们通过 Raydium V4 流动性池将这些 WIF 代币兑换成 19,035.97 SOL（1,958,733 美元）。这一系列操作在单笔交易中产生了 17,442.24 SOL（1,794,746 美元）的净利润。所有美元价值均反映交易时的价格。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fexample-solana-backrun-transaction.png&w=3840&q=90" alt="Example Solana Backrun Transaction">

[查看 Jito Explorer 上的尾随交易](https://explorer.jito.wtf/bundle/7d2f02a0542fd3950d90c9bd8ca84d233e28f0298d9f002c7e3cc0959b72b24f)

### 三明治攻击

三明治攻击是最臭名昭著的有害 MEV 形式。它们利用在 AMM 或 bonding curves（债券曲线）上设置高滑点容忍度的交易者。交易者设置高滑点不是为了接受更差的价格，而是为了确保快速执行订单。迷因币交易者（寻找那些 100 倍宝石）特别容易受到三明治攻击，因为他们在交易流动性不足、波动性高的资产时倾向于设置高滑点容忍度。三明治攻击对最终交易者造成严格的负面外部性——这些用户以最差的可能价格成交。

典型的三明治攻击涉及三个原子化打包在一起的交易。首先，攻击者执行一个不盈利的抢跑交易，买入资产以将其价格推至受害者滑点设置允许的最差执行水平。接着，受害者的交易发生，由于在这个不利水平执行而进一步推高价格。最后，攻击者完成一个盈利的尾随交易，在膨胀的价格下卖出资产，抵消他们最初的损失，并获得净利润。

#### 三明治攻击交易示例

这次攻击发生在 2024 年 12 月 16 日，通过一个知名的三明治攻击程序（vpeNALD… Noax38b）执行。[搜索者将这些交易作为原子 Jito 包提交](https://explorer.jito.wtf/bundle/17232d2f4dd1164648bd70f31be26dbfbf9561a899dfd031fa568cf31f7435fd)，支付了 0.000148 SOL（0.03 美元）的小费。

- [**抢跑交易**](https://xray.helius.dev/tx/3k35bPRTDZssneWhpbEptK27JjXVGvsv9ZWWnCJvn8PNA5pgKSNpMACPnmh3ogPbgpHKuFDo9ZKzLipM9kXianVY?cluster=mainnet-beta)**：**搜索者支付 14.63 SOL 购买了 3290 万个 Komeko 代币，这是 Pump Fun 平台上新发布的迷因币。
- [**受害者交易**](https://xray.helius.dev/tx/3jwXgoXkeLNPzunqGGjF9s9J4xwSSXjG98oPx7xrKaeX86eVxxpXrDkvL3M2JqnrH44otyvA3DSm9Hk7KUcaiyEY?cluster=mainnet-beta)**：**兑换 0.33 SOL 购买了 62.4 万个 Komeko 代币。
- [**尾随交易**](https://xray.helius.dev/tx/JajVyCLkukZ3jgsZMbq35SD7GcgkfYHEQLNoaoSGw3DaPfGmHAe2W7rfSiA1EMT8AhnCKM9pX6nMkCvt9eu11Yh?cluster=mainnet-beta)**：**搜索者将 3290 万个 Komeko 代币卖出换取 14.65 SOL。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fsolana-sandwich-attack-diagram.png&w=3840&q=90" alt="Solana Sandwich Attack Diagram">

表明这是三明治攻击的特征：

- 中间交易的签名者与第一笔和最后一笔交易的签名者不同。
- 前两笔交易购买的代币与第三笔交易卖出的是同一种代币。
- 交易的代币是新铸造的、流动性不足且波动性很高的 Pump Fun 代币。

搜索者获得了 0.01678 SOL 的净利润，在交易时约合 3.35 美元。

## Solana MEV 数据

本节通过可获得的公开数据评估当前 Solana MEV 的格局。我们首先分析 Jito 的性能指标，然后深入研究交易回滚数量以及套利盈利能力的细分情况。最后以一个案例研究收尾，详细介绍一个著名三明治机器人的行为和盈利能力。

### Jito

Jito 捆绑包是搜索者用来确保交易排序获利的主要方法。大多数 Jito 小费来自于想要第一个购买代币或抓住机会的用户对区块顶部位置的需求。然而，**Jito 数据并未捕获 MEV 活动的全貌**；特别是，它没有捕获搜索者的利润或通过其他内存池发生的活动。此外，许多应用程序出于非 MEV 目的使用 Jito，绕过优先费用以确保及时的交易打包。

来自八个指定 Jito 小费账户的转账数据显示，在过去一年中，处理了超过 30 亿个捆绑包，总计产生了 375 万 SOL 的小费。这一活动呈现明显的上升趋势，从 1 月 11 日的 781 SOL 小费低点，上升到 11 月 19 日和 20 日分别达到 60,801 SOL 和 60,636 SOL 的高点。第三季度出现明显放缓，小费在 9 月 7 日降至 1,661 SOL 的低点。与 2024 年全年看到的大幅增长相比，2023 年 12 月之前的小费数值可以忽略不计。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2F2024-daily-jito-tips.png&w=3840&q=90" alt="2024 Daily Jito Tips">

[查看 Dune Analytics 上的 Jito 小费数据](https://dune.com/queries/3272756/5525700)

通过 Jito 处理的捆绑包数量在 2024 年持续增长，在 12 月 21 日达到 2440 万个捆绑包的峰值。这一增长包括两次显著的激增。第一次发生在 5 月至 7 月初期间，每日捆绑包数量从约 300 万增长至 1200 万，增长了四倍，这可能是对网络拥堵问题的响应。第二次激增发生在 11 月至 12 月期间，每日捆绑包数量从约 1200 万翻倍至 2400 万的峰值。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2F2024-jito-bundles-daily.png&w=3840&q=90" alt="2024 Jito Bundles Daily">

[查看 Dune Analytics 上的 Jito 捆绑包数据](https://dune.com/queries/3034383/5044506)

使用 Jito 的账户数量也呈现出平行的上升趋势，年初每日支付小费的账户约为 20,000 个，在 12 月 10 日达到近 938,000 个的峰值。重要的增长期包括从 3 月初的 21,000 个增长到 4 月中旬的 135,000 个（增长了 6 倍），以及从 10 月的 208,000 个急剧上升到月底的 703,000 个（增长了 3.4 倍）。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fgraph-of-daily-jito-tippers-in-2024.png&w=3840&q=90" alt="2024 Jito Tippers Daily">

[查看 Dune Analytics 上的 Jito 小费支付者数据](https://dune.com/queries/3034383/5046648)

验证节点对 Jito-Solana 客户端的采用在 2024 年全年稳步增长，提高了 Jito 捆绑包快速打包交易的有效性。在年初，使用 Jito-Solana 客户端的验证节点代表了 1.895 亿质押 SOL，占网络总质押量的 48%。到 2025 年初，这一数字已上升至 3.738 亿质押 SOL，占总质押量的 92%。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fgraph-of-jito-solana-stake-growth-in-2024.png&w=3840&q=90" alt="2024 Jito Solana Stake Growth">

[查看 Jito 费用统计数据](https://explorer.jito.wtf/fee-stats)

### 回滚交易

Solana 上相当大一部分交易可归因于与 MEV 提取相关的垃圾交易。通过检查回滚交易与成功交易的比率，我们可以识别出表明 MEV 机器人在竞争套利机会的模式。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fgraph-of-non-vote-solana-transactions-by-week-in-2024.png&w=3840&q=90" alt="2024 Non-Vote Solana Transactions by Week">

[查看 Blockworks Research 上的 Solana 链上活动数据](https://solana.blockworksresearch.com/?dashboard=sol-onchain-activity&currency=USD&interval=weekly)

垃圾交易带来了重大挑战，因为它会导致大量交易回滚。在 MEV 的赢家通吃性质中，只有一笔交易能够利用给定的机会。然而，即使在这个机会被抓住之后，领导者通常仍会处理其他试图利用同一机会的交易。这些回滚的交易仍然会消耗宝贵的计算资源和网络带宽。搜索者之间的延迟竞争进一步加剧了这个问题，用重复的交易淹没网络，在极端情况下导致拥堵和用户体验下降。由于 Solana 的交易成本较低，回滚的套利垃圾交易仍然保持着正的期望值。随着时间推移，尽管个别交易失败，交易者通过大规模执行这些交易仍能实现盈利。

回滚交易在 2024 年 4 月达到峰值，占所有非投票交易的 75.7%。在推出包括 Agave 1.18 中央调度器在内的关键更新后，这一比例显著下降。新的调度器改进了银行阶段内的确定性交易排序，抑制了垃圾交易的有效性。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fgraph-of-reverted-transactions-on-solana-as-a-percentage-of-non-vote-transactions.png&w=3840&q=90" alt="2024 Reverted Transactions on Solana as a Percentage of Non-Vote Transactions">

[查看 Dune Analytics 上的 Solana 回滚交易数据](https://dune.com/queries/3537204/5951285)

### 套利盈利能力

Jito 的套利检测算法分析了所有 Solana 交易（包括 Jito 捆绑包之外的交易），在过去一年中识别出 90,445,905 笔成功的套利交易。每笔套利的平均利润为 1.58 美元，其中最赚钱的单笔套利获得了 370 万美元的收益。这些套利总共产生了 1.428 亿美元的利润，其中 1.267 亿美元（88.7%）以 SOL 计价。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Farbitrage-profits-by-token-on-solana-in-2024.png&w=3840&q=90" alt="2024 Arbitrage Profits by Token on Solana">

[查看 Jito 套利概览数据](https://explorer.jito.wtf/arbitrage-overview)

### 案例研究：Vpe 三明治程序

DeezNode 在地址 [vpeNAL..oax38b](https://xray.helius.dev/address/vpeNALD89BZ4KxNUFjdLmFXBCwtyqBDQ85ouNoax38b/history?cluster=mainnet-beta&page=1) 运营着一个链上三明治机器人，作为其替代内存池操作的一部分。这个高度活跃的程序最近因执行大规模用户三明治攻击而声名狼藉。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fgraph-of-hourly-sandwich-bundles-by-vpe-sandwich-bot-december-2024.png&w=3840&q=90" alt="2024 Hourly Sandwich Bundles by Vpe Sandwich Bot">

[查看 Flipside Crypto 上的 Vpe 三明治机器人数据](https://flipsidecrypto.xyz/marqu/solana-mev---sandwich-bot-vpe-Sv-6sy)

Jito 的内部分析表明，Solana 上近一半的三明治攻击都可以归因于这个单一程序。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fpie-chart-diagram-of-sandwich-attacks-by-program.png&w=3840&q=90" alt="Pie Chart Diagram of Sandwich Attacks by Program">

在一个 30 天的时期内（12 月 7 日至 1 月 5 日），该程序执行了 155 万笔三明治交易，平均每天约 51,600 笔交易，成功率为 88.9%。该程序产生了 65,880 SOL（1,343 万美元）的利润，相当于每天约 2,200 SOL。程序支付的 Jito 小费总计 22,760 SOL（463 万美元），平均每天约 758 SOL。每笔三明治交易的平均盈利为 0.0425 SOL（8.67 美元）。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fpie-chart-of-vpe-sandwich-bot-extracted-value-december-2024-to-jan-2025.png&w=3840&q=90" alt="Pie Chart of Vpe Sandwich Bot Extracted Value December 2024 to Jan 2025">

受害交易中大部分涉及通过 Raydium 进行的兑换交易。在前 20 个被三明治攻击的代币中，16 个是在 Pump Fun 上创建的，可以通过以"pump"结尾的虚荣代币铸造地址识别。

Vpe 三明治机器人是执行三明治攻击的众多链上程序之一。访问 [sandwiched.me 查看 Solana 上检测到的三明治攻击实时视图](http://sandwiched.me/)。

根据 12 月的利润数据年化计算，该程序预计将产生 801,540 SOL 的年度利润。在网络中心化的最坏情况下，如果将 100% 的利润重新投资到替代内存池的验证节点中，假设整体网络质押量保持不变，他们在网络中的质押份额将增加 0.2%。

这种最坏情况不太可能发生，原因有几个。首先，网络目前正经历着接近历史记录的活跃水平。其次，合理假设搜索者和运营者会将部分利润兑现，而不是将所有收益重新投资。

## MEV 缓解机制

大量资源已投入到研究和探索各种缓解或重新分配 MEV 的机制中。通用的、协议外的解决方案越来越多地集成到应用程序和基础设施中，以最小化链上 MEV 的攻击面。这些机制包括：

### 验证节点白名单

一个提议的想法是，质押者、RPC 提供商和其他验证节点可以通过忽略被发现进行三明治攻击的验证节点的领导时段来对其进行社会性孤立。然而，白名单普遍被视为最后的手段。由于领导者被分配四个连续的时段，这种方法可能会导致交易处理延迟数秒，这是一种次优的用户体验。更为关键的是，白名单可能会创造一个半许可和审查的环境，这直接与区块链行业的去中心化理念相冲突。此外，这类系统还存在错误排除诚实验证节点的固有风险，可能会破坏网络信任和参与度。

顺便说一下，独立开发者和应用程序可以自由建立自己的验证节点允许或拒绝列表，这一功能由 [**Helius Node.js SDK 中的 sendTransaction 方法**](https://github.com/helius-labs/helius-sdk?tab=readme-ov-file#sendtransaction)支持。

### 动态滑点 + MEV 保护

管理滑点一直是用户面临的一个具有挑战性和繁琐的过程，需要根据交易代币进行手动调整。在处理波动性或流动性较低的代币时，这种方法尤其繁重，因为适用于流动性质押代币或稳定币等稳定资产的滑点设置与迷因币所需的设置有很大不同。

2024 年 8 月，Solana 最受欢迎的零售交易平台 Jupiter Aggregator 引入了动态滑点来解决这个复杂问题。这种算法机制实时优化滑点设置，利用一组启发式方法计算每笔交易的理想滑点阈值。这些启发式方法考虑了以下因素：

- 当前市场状况
- 交易代币的类型（例如，稳定币对与波动性迷因币）
- 交易路由经过的流动性池或订单簿
- 用户设置的最大滑点容忍度

这些启发式方法确保交易在最小滑点的情况下得到优化执行，从而减少了 MEV 提取的空间。

MEV 保护模式是去中心化交易所和 Telegram 交易机器人中越来越常见的功能。启用后，用户交易将专门路由到 Jito 区块引擎，显著降低三明治攻击的风险。然而，这种保护需要支付略高的交易费用。轶事证据表明，即使提供了 MEV 保护，许多 Telegram 机器人用户也不会启用它。他们主要关注的是快速交易确认，并且在降低三明治攻击风险和交易速度之间，他们优先选择速度。

### 询价系统

询价系统在 Solana 上正在获得关注，它使订单能够由专业做市商而不是链上自动做市商(AMM)或订单簿来完成。这些系统使用基于签名的定价，允许链下计算，价格发现在链下进行，只有最终交易记录在链上。示例包括：

#### Kamino 兑换

一个基于意图的交易平台，旨在消除滑点和 MEV。Kamino 利用 [Pyth Express Relay](https://www.pyth.network/blog/express-relay-priority-auctions) 向搜索者网络广播兑换请求，搜索者通过拍卖竞争完成交易。获胜的搜索者提供最佳执行价格并向用户支付小费。在出现套利机会的情况下，搜索者可能会以比要求更好的价格执行交易，产生交易"盈余"。用户通过保留交易中的任何盈余来获益，从而提高其整体执行价值。

#### ‍JupiterZ (Jupiter RFQ)

从 12 月开始，JupiterZ 在 Jupiter 上的所有兑换交易中默认启用。该功能允许兑换交易在 Jupiter 的标准链上路由引擎和其 RFQ 系统之间自动选择最佳价格。通过 RFQ，用户可以受益于零滑点和零 MEV，因为交易直接与链下做市商执行。此外，做市商承担交易优先费用，交易在计算单元使用上也很高效，无需复杂的路由逻辑。

‍RFQ 系统在中心化交易所上市的广泛交易代币方面表现出色。然而，对于较新的、流动性低的和高度波动的链上资产，它们的效果较差。不幸的是，这些恰恰是最容易受到 MEV 利用的交易。另一个缺点是流动性转移到链下，降低了可组合性。

### 防三明治攻击的自动做市商

[防三明治攻击的自动做市商（Sandwich-resistant AMMs, sr-AMMs）](https://www.umbraresearch.xyz/writings/sandwich-resistant-amm)是在传统恒定乘积（xy=k）AMM 基础上的实验性设计。在防三明治 AMM 中，所有交易的执行价格都不会优于槽位窗口开始时的池子价格。这种机制有效地消除了三明治攻击的获利空间。

防三明治 AMM 使用槽位窗口来管理交易。在一个槽位窗口内的交易会对买卖订单产生不对称的影响：

- 当执行买入订单时，池子的报价沿着 xy=k 曲线上升，而买价保持不变，实际上为买方增加了流动性。
- 相反，卖出订单会消耗这种买方流动性，降低由 xy=k 曲线决定的报价。

在每个新槽位窗口开始时，防三明治 AMM 会重置到其等效的 xy=k 状态，重新校准买卖价格。通过将这些重置与单个交易解耦，并在每个槽位窗口内保持一致的定价，防三明治 AMM 打破了三明治攻击获利所需的原子执行，使其失效。

三明治攻击在槽位边界处仍然可能发生。如果一个领导者控制连续的槽位窗口，他们可以在第一个槽位结束时执行抢跑交易和目标交易，然后在下一个槽位开始时执行回跑交易。

Ellipsis Labs 于今年 11 月发布了 [Plasma，一个经过审计的防三明治 AMM 设计参考实现](https://www.ellipsislabs.xyz/blog-posts/introducing-plasma)。

### 条件性流动性与订单流分割

去中心化交易所（DEX）目前缺乏针对不同类型市场参与者应用差异化定价的机制。这一限制的产生是因为 DEX 无法准确识别订单流对 DEX 协议造成的成本。DEX 为了吸引订单流而收窄点差，无意中增加了来自老练交易者的逆向选择风险。

[条件性流动性](https://pond.dflow.net/blog/2024-12-19-intro-cl)引入了一种新机制，使 DEX 能够根据预期的订单流毒性动态调整点差。这使 DEX 能够表达更广泛的链上、即时偏好。与其向所有参与者提供单一点差，条件性流动性使 DEX 能够根据特定交易者的预期逆向选择可能性，提供一个梯度化的点差范围。

这个过程依赖于一类新的市场参与者，称为分割者（Segmenters）。分割者专门评估订单流的毒性并相应调整点差。他们获取调整后点差的一部分作为补偿，同时将剩余部分传递给钱包或交易者。通过管理点差设置职责，分割者使 DEX 能够更好地竞争非毒性订单流。分割者相互竞争以最小化流动性提供者的逆向选择风险。最紧凑的报价保留给被认为最不可能损害流动性提供者的订单流。在最简单的形式中，钱包或应用程序可以作为其自身订单流的分割者。或者，它也可以将这种流量分割的职责委托给市场。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Forderflow-segmentation-diagram-conditional-liquidity-using-segmenters.png&w=3840&q=90" alt="Orderflow Segmentation Diagram - Conditional Liquidity Using Segmenters">

用户通过"声明式兑换"来利用这一机制，它使用户能够声明其兑换意图并利用分割者来执行。这些兑换同时与现有的 Solana 流动性来源和支持条件性流动性的 DEX 进行交互。声明式兑换基于 Jito 捆绑包构建，在签名时为交易者提供保证报价，同时在交易落地网络之前重新计算最优路由——确保遵守初始报价。

这种方法显著减少了路由计算和交易完成之间的延迟，从而减少滑点。此外，当通过条件性流动性 DEX 进行路由时，声明式兑换最大限度地降低了三明治攻击的可能性。通过为非毒性流提供更紧凑的点差，这些 DEX 改善了 Solana 用户的交易条件。因此，声明式兑换为交易者提供了更低的滑点、更低的延迟和更强的防三明治攻击保护，带来更高效和安全的交易体验。

### Paladin

Paladin-Solana 是 Jito-Solana 验证者客户端的修改版本，引入了一个最小代码补丁（约 2000 行），以在捆绑包阶段包含 Paladin 优先端口（P3）交易。Paladin 优先端口（P3）用于处理高优先级费用交易。验证者作为领导者开放这条快速通道，使其能够及时处理有价值的交易。每笔 P3 交易都需要满足最低费用阈值（每计算单元 10 lamports），并按接收顺序直接传递到捆绑包阶段进行处理。

Paladin 优先处理高优先级费用交易，并根据交易模式主动识别和丢弃三明治捆绑包。虽然这最初可能看起来会损害验证者的奖励，但 Paladin 验证者通过基于信任的机制获得补偿。避免三明治攻击的验证者可以吸引直接交易，从而创建一个基于信任和提高收益的生态系统。

验证者通过获得额外奖励和依赖 P3 快速通道的用户信任而获得激励。然而，如果他们在区块中包含三明治捆绑包，就会面临失去 P3 交易收入的风险。这种信任由 PAL 代币作为抵押。

PAL 代币旨在协调验证者、用户和更广泛的 Solana 社区的利益。它的固定供应量为 10 亿代币，其中 65% 将分配给验证者和质押者，其余部分将在 Solana 开发者、Paladin 团队和开发基金之间分配。验证者可以锁定 PAL 以在其节点上启用 P3 交易，从而创建一个去中心化、无需许可和基于代币的 MEV 提取和交易优先级机制。

该项目仍处于早期阶段，尚未达到大规模采用。目前有 80 个验证者运行 Paladin，占网络质押的 6%。Paladin 声称可以将区块奖励提高 12.5%。

### 多并发领导者

区块生产者在其分配的槽位内对交易包含具有垄断权。即使当前领导者已知会恶意进行三明治攻击，用户也不会意识到这一点，仍会提交交易，期望它们能够不延迟地处理。这种对哪个节点处理和排序交易缺乏选择权的情况使用户容易受到操纵。

多并发领导者（MCL）系统在同一槽位内引入了区块生产者之间的竞争。用户获得了在不引入延迟的情况下在领导者之间进行选择的能力。如果领导者 A 表现恶意且已知会进行三明治攻击，用户或应用程序可以选择将交易提交给诚实行事的领导者 B。

长期最大化领导者之间的竞争涉及减少槽位持续时间、限制分配给单个领导者的连续槽位数量，以及增加每个槽位的并发领导者数量。通过每秒安排更多的领导者，用户获得更大的灵活性，能够从可用的领导者中选择最有利的交易包含方案。

尽管 MCL 为缓解 MEV 提供了一个有前景的长期解决方案，但其实施很复杂，可能需要数年的开发时间。

[异步执行（Asynchronous Execution，AE）](https://www.helius.dev/blog/asynchronous-program-execution)提供了另一种可能的减少 MEV 的方法。在 AE 下，区块的构建不需要执行或评估每笔交易的结果。这种速度对算法在计算获利机会和及时执行有效的三明治策略方面带来了重大挑战。

## 结论

Solana 的 MEV 格局正在快速演变，距离达到稳定的竞争均衡还很遥远。搜索者不断开发更复杂的策略来提取价值，而生态系统则采用多管齐下的基础设施和机制来缓解有害的 MEV。前瞻性的生态系统投资者（如 Multicoin Capital）正在进行资本配置，他们相信生态系统团队从 Solana MEV 中获取的价值将显著增长，并且这种价值获取的分配在未来几年将呈现出非常不同的面貌。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fvalue-capture-distribution-of-mev-diagram.png&w=3840&q=90" alt="Value Capture Distribution of MEV Diagram">

想要深入了解 Solana MEV 的未来发展，可以观看 [Multicoin Capital 的 Kyle Samani 在 Breakpoint 2023 上的精彩演讲](https://youtu.be/NY_4W03VmFI?si=t-o0hKucRuax1syE)。

MEV 是任何有重大金融活动的去中心化区块链都无法避免的挑战。应对和管理这个"MEV 恶魔"对网络的长期成功至关重要。在经历了 2023 年的种种困难后，Solana 现已变得更加强大，作为一个活跃度高且采用率不断增长的区块链蓬勃发展。然而，新的挑战就在前方。要实现下一阶段的采用，生态系统必须直面这些挑战。这是 Solana 发展历程中的关键时刻，也是定义其未来的重要机遇。

## [更多资源](https://www.helius.dev/blog/solana-mev-report#further-resources)

- [MEV 简介](https://www.helius.dev/blog/solana-mev-an-introduction) - Helius 博客
- [套利作为凸优化问题](https://chorusone.notion.site/Arbitrage-as-a-Convex-optimization-problem-f2490665033f41b6b6d41cfd5196acae) - Umberto Natale，Chorus One
- [以太坊与 Solana：MEV 及其他](https://www.youtube.com/watch?v=PLq-75nM3dE) - Uncommon Core
