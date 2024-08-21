## Solana 执行概览

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4309097325534b13d2290_66b42bf84f7fd7478dbc4534_Solana%2520Report%2520Visuals%2520for%2520Helius%2520Blog.pptx.png" loading="lazy" alt="">

我衷心感谢 0xIchigo、dubbelosix、Jacob Creech、Maël Bomane、Nagaprasad Vr 和 Rex St. John 阅读本报告的早期版本，并提供了宝贵的反馈。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b5494019c5527ba72b0aaf_66b5492cca2c05958ae970fa_Slide1%2520(1).jpeg" loading="lazy" alt="">
本报告的 PDF 版本可 [在此下载](https://drive.google.com/file/d/1puEThjO9rmnZmIj_05WvU99FPmUiZ4pb/view) 。

#### 简介


>  “我们比世界上任何人都更了解小巧、更快、更便宜的概念，现在我们将这些概念应用于区块链。” — Greg Fitzgerald，Solana 联合创始人


Solana 是一个高性能、低延迟的区块链，以其速度、效率和对用户体验的关注而闻名。其独特的集成架构使其能够在全球去中心化的网络中每秒处理数千笔交易。区块时间为 400 毫秒，交易费用仅为几分钱，使其在速度和成本效益上都表现出色。本报告将深入探讨 Solana 的设计和操作细节，探索其能力的关键机制和网络拓扑结构。

Solana 采用了集成化的区块链开发方法，利用了创始团队在构建分布式系统方面的几十年经验。Solana 的核心原则之一是软件不应妨碍硬件。这意味着软件要充分利用其运行的硬件，并随着硬件的扩展而扩展。作为一个统一的生态系统，所有在此单一区块链上构建的应用程序都继承了可组合性，使它们能够无缝互动和相互构建。这种架构还确保了简单直观的用户体验，无需桥接、独立链 ID 或流动性碎片化。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b425278d15bfcef59751d3_AD_4nXen99WstkKkSV_8yVPQmoSDC-s7FVUr3TUGAWMI9LdLaRG3lkUZh73B0P3RCE0jkzk23Tcgqn7MVoNQ2gLhHvAdeBPu5ZoiXghn-IYZhaWIHjRbqy8JU4HSsgMkhlBIZtc6V2bF9N--YUkiGiucU8T_dwtt.png" loading="lazy" alt="">

Solana 正在快速发展，最近的进展包括 SVM rollups 和 [ZK Compression](https://www.zkcompression.com/)，这些都是重要的扩展解决方案。虽然这些项目未来可能会影响我们对 Solana 的看法，但它们目前仍处于非常早期的开发或采用阶段，本报告将不会涉及这些内容。

#### 交易生命周期

本报告将通过典型交易的生命周期来理解 Solana。为了构建一个理解 Solana 交易的基本模型，我们可以概述以下过程：

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b541c4dbd67680ef5744cb_66b541b93d9f081de2547b00_New%2520Project.png" loading="lazy" alt="">

1. 用户发起交易，所有交易都发送到当前的主导区块生产者（称为领导者）。领导者将这些交易编译成一个区块，执行交易并更新本地状态。
2. 这个区块的交易随后在网络中传播，供其他验证者执行和确认。

本报告的后续部分将扩展这一模型，并更详细地探讨这一过程，从关键参与者——用户开始。

#### 记住

对 Solana 协议核心进行的重大更改需经过正式的透明过程，即提交 Solana 改进文档（SIMD），社区成员和核心工程团队将公开评论这些[文档](https://github.com/solana-foundation/solana-improvement-documents/blob/main/proposals/0001-simd-process.md)。SIMD 随后将由网络进行投票。

### 六个阶段

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4309097325534b13d2290_66b42bf84f7fd7478dbc4534_Solana%2520Report%2520Visuals%2520for%2520Helius%2520Blog.pptx.png" loading="lazy" alt="">

我们将在本报告中引用上述六个阶段的视觉图，因为它为理解 Solana 核心元素之间的关系提供了一致的框架。

前几个章节按照这六个阶段的顺序排列。最后几章——Gossip、Archive、Economics 和 Jito——则用来解决任何剩余的问题。需要注意的是，有些章节会跨越多个阶段，有些阶段也会出现在多个章节中。

这种重叠是不可避免的，因为六阶段框架有其局限性。实际上，Solana 是一个复杂的分布式系统，拥有许多相互依赖的元素。

#### 用户


> “Solana 有潜力成为加密货币领域的苹果公司” — Raj Gokal，Solana 联合创始人

用户的旅程通常始于设置和资助一个钱包应用程序。Solana 上有多个流行的钱包应用程序，可以是原生的移动应用程序或浏览器扩展。

钱包通过加密方式生成用户的密钥对，包括公钥和私钥。公钥作为其账户的唯一标识符，被网络中的所有参与者所知。Solana 上的用户账户可以看作是一个数据结构，包含与其在 Solana 区块链上的互动相关的信息和状态。这样，公钥类似于文件名：正如文件名在文件系统中唯一标识一个文件，Solana 公钥唯一标识区块链上的一个账户。Solana 上的公钥以 32 字节的 Base58 编码字符串表示。

例如：

FDKJvWcJNe6wecbgDYDFPCfgs14aJnVsUfWQRYWLn4Tn

私钥——也称为密钥——可以看作是一个密码或访问密钥，授予访问和修改账户的权限。通过私钥签名是区块链处理授权的方式。知道私钥就可以完全控制账户。Solana 的私钥也为 32 字节。密钥对是由公钥（前半部分）和私钥（后半部分）组合而成的 64 字节的密钥对。示例：

3j15jr41S9KmdfughusutvvqBjAeEDbU5sDQp8EbwQ3Hify2pfM1hiEsuFFAVq8bwGywnZpswrbDzPENbBZbd5nj

[63,107,47,255,141,135,58,142,191,245,78,18,90,162,107,197,8,33,211,15,228,235,250,30,185,122,105,23,147,115,115,86,8,155,67,155,110,51,117,0,19,150,143,217,132,205,122,91,167,61,6,246,107,39,51,110,185,81,13,81,16,182,30,71]

私钥也可以从助记词种子短语（通常由 12 或 24 个单词组成）中派生出来。这种格式通常用于钱包，方便备份和恢复。一个种子短语可以确定性地派生出多个密钥。

Solana 使用 [Ed25519](https://en.wikipedia.org/wiki/EdDSA#Ed25519)，这是一种广泛使用的椭圆曲线数字签名算法，满足其公钥加密需求。Ed25519 由于其小密钥尺寸、小签名尺寸、快速计算和对许多常见攻击的免疫性而受到青睐。每个 Solana 钱包地址代表 Ed25519 椭圆曲线上的一个点。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4308f97325534b13d2287_66b4301c70f691555c899730_Solana%2520Report%2520Visuals%2520for%2520Helius%2520Blog.pptx%2520(9).png" loading="lazy" alt="">

用户用其私钥对交易进行签名。这个签名会附加在交易数据中，其他参与者可以使用发送者的公钥进行验证。这个过程确保了交易未被篡改，并且由相应的私钥所有者授权。签名也作为交易的唯一标识符。

#### Solana 交易

在 Solana 上，发送交易是唯一可以改变状态的方式。任何写操作都通过交易执行，交易是原子的——要么交易尝试做的所有操作都发生，要么交易失败。交易，或更正式地称为“交易消息”，由四部分组成：头部(header)、账户地址列表(address)、最近区块哈希(recent blockhash)和指令(instructions)。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4349a0259ca4c7505bdaf_AD_4nXeTKh9LSDhziXt5D5i8YXOo6JNczhbY9nP1vfwgxi9ICunvKyY6t4V5po7X118WXGCkbPJXwRyEkIJA5kXSWby9XsBk9BIVxCEeX0jT6vOLPkRzx_M-Pymred9CLmQdP9VsfozpN13BGwTbKzWIpgV01uQ.png" loading="lazy" alt="">

- **头部:** 头部包含对账户地址列表的引用，指明哪些账户必须签署交易。
- **账户地址:** 这个列表包括在交易过程中将要读取或写入的所有账户。为每个交易构建这样的列表是 Solana 的一个独特要求，可能对开发者来说具有挑战性。**然而，提前知道交易将要交互的状态片段可以实现许多其他区块链上无法进行的优化**。
- **最近区块哈希:** 这用于防止重复和过时的交易。最近区块哈希在 150 个区块后（约 1 分钟）过期。默认情况下，RPC 每 2 秒尝试转发交易，直到交易被最终确定或最近区块哈希过期，过期后交易会被丢弃。
- **指令:** 这些是交易的核心部分。每条指令代表一个具体操作（如转账、铸造、销毁、创建账户、关闭账户）。每条指令指定要执行的程序、所需的账户以及指令执行所需的数据。

交易中的指令数量首先受限于其大小，最大可达 1,232 字节。还有限制可以引用的账户数量。最后，交易的复杂性有计算单位（CUs）限制。CUs 量化了处理交易时消耗的计算资源。

#### 记住

SOL 的最小单位称为“lamport”，相当于一个 SOL 的十亿分之一，类似于比特币中的 satoshi。lamport 以计算机科学家和数学家 [Leslie Lamport](https://en.wikipedia.org/wiki/Leslie_Lamport) 的名字命名，他的研究奠定了现代分布式系统的许多理论基础。

执行交易的 SOL 成本分为两个部分——基本费用和优先费用。基本费用是固定的，每个签名费用 5000 lamports，不论交易的复杂性如何——通常每笔交易有一个签名。

优先费用在技术上是可选的，但在区块空间需求高峰期变得必要。这些费用以微 lamports（万分之一的 lamport）每计算单位计价。其目的是作为价格信号，使交易对验证节点在其区块中包含变得更加经济有利。

```
总费用 = 优先费用 + 基本费用

优先费用 = 计算单位价格（微 lamports）x 计算单位限制
```

目前，所有与交易相关的费用中有 50% 被销毁，永久从流通中移除，而剩余的 50% 则归区块生产者所有。即将引入的新变化（SIMD 96）将允许 100% 的优先费用归区块生产者所有，基本费用保持不变。

### 发送交易

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4308f97325534b13d2284_66b4307e1b70b8398684582c_Solana%2520Report%2520Visuals%2520for%2520Helius%2520Blog.pptx%2520(2).png" loading="lazy" alt="">

用户将他们的钱包连接到应用程序，这样应用程序就可以读取用户的公钥。私钥则保持加密状态，并在一个与应用程序隔离的安全环境中。

应用程序根据用户的操作构建交易消息参数。例如，如果用户想要交换两种代币，他们会指定要购买的代币数量、要出售的对应代币以及可以接受的交易滑点。

一旦交易消息准备好，它会被发送到钱包进行用户私钥签名。在这时，用户会收到一个弹窗提示，确认他们是否愿意进行交易。这个弹窗可能会包括交易结果的模拟。签名完成后，交易消息和签名会返回到应用程序，应用程序随后可以将交易转发到他们选择的 RPC 提供商，可能是他们自己的，也可能是钱包提供商的。

RPC（远程过程调用）提供商充当应用程序和构建区块的验证者之间的中介。它们是一个关键服务，使应用程序能够[提交](https://github.com/solana-labs/solana/blob/cd6f931223181d5a1d47cba64e857785a175a760/rpc/src/rpc.rs#L3396)或模拟签名的交易，并有效地检索链上数据。希望与网络互动的应用程序通过 JSON-RPC 或 WebSocket 端点进行操作（[文档](https://solana.com/docs/rpc)）。

#### 记住

在 Solana 上，“失败的交易”这一术语可能会造成误解，并引发相当大的困惑。这些交易会产生费用，并且由运行时成功执行，正如签名者所预期的那样。它们之所以“失败”是由于交易自身的逻辑要求。+80% 的“失败”交易来自错误代码 0x1771，即超出滑点金额（[数据](https://flipsidecrypto.xyz/zapokorny/q/18zuYlugBdcm/solana-failed-txs-by-program)）。值得注意的是，95% 的这些交易由仅 0.1% 的活跃 Solana 地址提交，这些地址主要是试图利用时间敏感的价格套利机会的自动化机器人。

### Gulf Stream

> “Solana 的目标就是让交易传递速度像新闻传播一样快——所以是光速通过光纤。我们的竞争对手是 NASDAQ 和纽约证券交易所。” — Anatoly Yakovenko，Solana 联合创始人

RPC（远程过程调用）指的是 RPC 节点。这些节点可以被视为与网络交互和读取数据的网关。它们运行与全节点验证者相同的软件，但设置不同，使其能够准确模拟交易并保持当前状态的最新视图。截至本写作时，Solana 网络上已有超过 4,000 个 RPC 节点。

与全节点验证者不同，RPC 节点在网络中没有持有任何质押。没有质押，它们无法投票或构建区块。这种设置与大多数其他区块链不同，通常验证者和 RPC 节点是相同的。由于 RPC 节点不获得质押奖励，因此运行 RPC 节点的经济学与验证者有所不同，许多 RPC 节点作为为开发者提供服务的付费服务运营。

Solana 的突出之处在于，它从一开始就被设计为不使用内存池。与传统区块链使用 gossip 协议随机且广泛地传播交易不同，Solana 将所有交易转发给每个槽的预定领导验证者，即领导者。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b41fe65de8a0d019876ca9_66b41ed9148c9cd7c37237a0_Solana%2520Report%2520Visuals%2520for%2520Helius%2520Blog.pptx%2520(4).png" loading="lazy" alt="">

#### 说明框

Solana 运行四个集群：Localnet、Testnet、Devnet 和 Mainnet-Beta。当人们提到 Solana 或 Solana 网络时，他们几乎总是指 Mainnet-Beta。Mainnet-Beta 是唯一一个代币具有实际价值的集群，而其他集群则仅用于测试目的。

一旦 RPC 收到要包含在区块中的交易消息，它必须转发给领导者。在每个纪元（大约每两天）之前会产生一个领导者日程。即将到来的纪元被划分为槽，每个槽固定为 400 毫秒，并为每个槽选择一个领导者。持有较高质押的验证者将在每个纪元中更频繁地被选择为领导者。在每个槽期间，交易消息会被转发给领导者，领导者有机会生成一个区块。当轮到验证者时，他们会切换到“领导者模式”，开始积极处理交易并将区块广播到网络的其他部分。

### 质押加权服务质量 - SWQoS

在 2024 年初，Solana 引入了一种新的机制，旨在防止垃圾邮件和增强 Sybil 抵抗力，称为“质押加权服务质量”（SWQoS）。该系统使领导者能够优先处理通过其他质押验证者代理的交易消息。在这里，持有较高质押的验证者被授予按比例更高的能力，以将交易消息包传输到领导者。这种方法有效地缓解了网络中非质押节点的 Sybil 攻击。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b41fe65de8a0d019876cae_66b41fbd718582eb77853717_Solana%2520Report%2520Visuals%2520for%2520Helius%2520Blog.pptx%2520(1).png" loading="lazy" alt="">

在这种模型下，验证者还可以达成协议，将他们的质押加权能力租赁给 RPC 节点。作为回报，RPC 节点获得增加的带宽，使其在区块中实现更高的交易包含率。值得注意的是，领导者的 80% 的容量（2,000 个连接）保留用于 SWQoS。剩余的 20%（500 个连接）则分配给来自非质押节点的交易消息。这种分配策略类似于高速公路上的优先车道，司机支付通行费以避免交通拥堵。

SWQoS 通过提高将交易转发到领导者的要求，并减少垃圾邮件攻击的效果，从而影响了 Solana 生态系统。这一变化激励了高流量应用程序垂直整合其操作。通过运行自己的验证者节点或访问质押连接，应用程序可以确保优先访问领导者，从而增强其交易处理能力。

### QUIC 说明

在 2022 年底，Solana 采用了 [QUIC 网络协议](https://en.wikipedia.org/wiki/QUIC)来管理交易消息的传输到领导者。这个转变是由于机器人在链上 NFT 铸造时造成的网络中断。QUIC 促进了快速的异步通信。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b41fe65de8a0d019876ca6_66b41f7990bc020d3bc0e8af_Solana%2520Report%2520Visuals%2520for%2520Helius%2520Blog.pptx%2520(19).png" loading="lazy" alt="">

QUIC 最初由 [Google](https://research.google/pubs/the-quic-transport-protocol-design-and-internet-scale-deployment/) 于 2012 年开发，试图结合 UDP 的快速异步通信与 TCP 的安全会话和先进的流量控制策略。这允许对个别流量源进行限制，以便网络可以专注于处理真实的交易。它还具有独立流的概念，因此如果一个交易被丢弃，它不会阻塞其他交易。简而言之，QUIC 可以被视为试图结合 TCP 和 UDP 的最佳特性。

#### 说明框

质押加权是 Solana 系统中反复出现的原则，涵盖了投票奖励(voting rewards)、涡轮树(turbine trees)、领导者日程(leader schedules)、Gulf Stream 和 gossip 网络。持有更高质押的验证者在网络中享有更高的信任度和优先角色。

### 区块构建

> “我们认为 SVM（Solana 虚拟机）在虚拟机技术方面是目前最好的。” — Andre Cronje，Fantom 基金会

许多区块链网络在广播之前构建完整的区块，这种方法称为离散区块构建。与之不同，Solana 使用连续区块构建，这涉及到在分配的时间槽内动态地组装和流式传输区块，从而显著减少了延迟。

每个槽持续 400 毫秒，每个领导者被分配四个连续的槽（1.6 秒），然后轮换到下一个领导者。为了使区块获得接受，区块中的所有交易必须是有效的，并且能够被其他节点重复执行。

在接管领导权的前两个槽内，验证者会停止交易转发，以准备接下来的工作负载。在此期间，入站流量会激增，达到每秒超过一吉字节，因为整个网络将数据包指向即将到来的领导者。

一旦收到，交易消息会进入交易处理单元（TPU），这是验证者用于区块生成的核心逻辑。这里，交易处理序列从获取阶段开始，在此阶段，交易通过 QUIC 接收。随后，交易进入签名验证阶段，进行严格的验证检查。在这一阶段，验证者会验证签名的有效性，检查签名数量的正确性，并消除重复交易。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4349b387dfd5ed49eab98_AD_4nXc5Ka4ko5KjYScssIZ8m20itLom6iaur53g90WZ00g5oc05zGYWcpaH0isUPglVQDqhNgFKoGkepcOK0tV7c43oCmdJHIItCF0VGVu1VL8_S7RqJQbYZ6rBO8pBGCZwDU1fbmBklBnbyufd7yDfH05JkcNq.png" loading="lazy" alt="">

### 银行阶段

银行阶段可以描述为区块构建阶段。它是 TPU 中最重要的阶段，因其名称来源于“银行”。银行就是特定区块的状态。对于每个区块，Solana 有一个银行，用于访问该区块的状态。当区块在足够多的验证者投票后成为最终区块时，他们会将账户更新从银行刷新到磁盘，使其永久化。链的最终状态是所有确认交易的结果。这一状态可以从区块链历史中确定性地重建。

交易会并行处理，并打包成账本“条目”，即一批 64 个不冲突的交易。Solana 的并行交易处理之所以容易，是因为每个交易必须包含一个完整的账户列表，说明它将读取和写入哪些账户。这种设计选择对开发者提出了要求，但使验证者能够通过轻松选择不冲突的交易来避免竞争条件。交易会冲突，如果它们都尝试写入同一个账户（两个写入）或一个尝试读取而另一个写入同一个账户（读取 + 写入）。因此，冲突的交易会进入不同的条目，并按顺序执行，而不冲突的交易则并行执行。

有六个线程并行处理交易，其中四个专门处理普通交易，两个专门处理投票交易，这对于 Solana 的共识机制至关重要。所有的并行处理都是通过多个 CPU 核心实现的；验证者没有 GPU 要求（[文档](https://docs.solanalabs.com/operations/requirements)）。

一旦交易被分组到条目中，它们就可以由 Solana 虚拟机（SVM）执行。必要的账户被锁定；检查会运行以确认交易是最新的但尚未处理。账户被加载，交易逻辑被执行，更新账户状态。条目的哈希会发送到历史证明服务以进行记录（更多内容请见下一节）。如果记录过程成功，所有更改将被提交到银行，第一次步骤中对每个账户的锁定将被解除。执行由 SVM 完成，这是一个使用 Solana 分叉的 [rBPF](https://docs.rs/solana_rbpf/latest/solana_rbpf/) 库构建的虚拟机，rBPF 用于 JIT 编译和 eBPF 程序的虚拟机。请注意，Solana 不强制要求验证者在区块内如何排序交易。这种灵活性是一个关键点，我们将在本报告的经济学 + Jito 部分进一步讨论。

#### 记住

SVM 这一术语可能会造成歧义，因为它可能指代“Solana 虚拟机”或“Sealevel 虚拟机”。这两个术语描述的是同一个概念，其中 Sealevel 是 Solana 的运行时环境的名称。尽管最近有努力精确[定义其边界](https://www.anza.xyz/blog/anzas-new-svm-api)，但 SVM 这一术语仍然被松散地使用。

### 客户端

Solana 是一个由数千个独立运行的节点组成的网络，这些节点协作维护一个统一的账本。每个节点都是一个高性能的机器，运行相同的开源软件，称为“客户端”。

Solana 启动时有一个单一的验证者客户端软件——最初是 Solana Labs 客户端，现在被称为 [Agave 客户端](https://github.com/anza-xyz/agave)——用 Rust 编写。客户端多样化一直是一个优先事项，并且随着 [Firedancer 客户端](https://www.helius.dev/blog/what-is-firedancer)的推出，这一目标将真正实现。Firedancer 是用 C 语言从头编写的原始客户端的完全重写。由高频交易公司 Jump 的经验丰富团队开发，它承诺成为任何区块链上性能最强的验证者客户端。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4349b38e242661eae3acb_AD_4nXe6JNwYPIps4DC8blPEA_rDOczQycVfXAu4exdsmcdDsCC3OWZMm8CklLXhYMytI94K3-C703_-VP6VOhIdnTvl25e24cqB-QD0Lr7ojK3NTT20TbB32x1vyShhG4eZLUCb54o6XmY-_RuQP1TKwFusHTc.png" loading="lazy" alt="">

### 历史证明

> “我喝了两杯咖啡和一杯啤酒，熬夜到凌晨 4 点。我有了一个 Eureka 时刻，发现了类似工作量证明的难题，使用相同的 SHA-256 预映像抗性哈希函数……我知道我有了时间的箭头。” — Anatoly Yakovenko，Solana 联合创始人

历史证明（PoH）是 Solana 的秘密武器，像是每个验证者的特殊时钟，促进网络同步。PoH 确立了事件顺序和时间流逝的可靠源。最关键的是，它确保遵守领导者日程。尽管名字相似，历史证明并不是像工作量证明这样的共识算法。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4349a316838409e6b83cf_AD_4nXd1dPVasmXQc1GLdcNA8CzVjUcHMdnfcaL478TxJaLjN5ckqirsq7uJ0sxtwa0Fr0ksNbLU8nVN_h6ct2swNwM9fKpiNPjM-RVKgKoikMqTMH7NkEyzwrqQVtKsS0U9Mzl89WOWIn2gr9XOXXR9YA9nxjw.png" loading="lazy" alt="">

随着网络的扩展，节点之间的通信开销通常会增加，协调变得越来越复杂。Solana 通过用 PoH 本地计算替代节点间通信来缓解这一问题。这意味着验证者可以通过仅进行一次投票轮次来确认一个区块。消息中的受信时间戳确保验证者不能互相越过并提前开始他们的区块。

PoH 的基础是哈希算法的独特特性，特别是 [SHA256](https://en.wikipedia.org/wiki/SHA-2)：

- 确定性：相同的输入将始终产生相同的哈希。
- 固定大小：无论输入大小如何，输出哈希始终为 256 位。
- 高效：计算任何给定输入的哈希速度快。
- [预映像抗性](https://en.wikipedia.org/wiki/Preimage_attack)：从哈希输出中找到原始输入是计算上不可行的。
- 雪崩效应：输入的微小变化（甚至是一个比特）会导致哈希发生显著变化，这称为雪崩效应。
- [碰撞抗性](https://en.wikipedia.org/wiki/Collision_resistance)：找到两个不同的输入产生相同的哈希输出是不可行的。

在每个验证者客户端内，专用的“历史证明服务”持续运行 SHA256 哈希算法，创建哈希链。每个哈希的输入是前一个哈希的输出。这条链像是一个可验证延迟函数，因为哈希工作必须按顺序进行，未来哈希的结果不能提前知道。如果 PoH 服务创建了一千个哈希链，我们知道时间已经过去了，计算每个哈希是按顺序进行的——这可以被看作是一个“微型工作量证明”。然而，其他验证者可以并行验证这千个哈希的正确性，速度比它们生成的速度快得多，因为每个哈希的输入和输出已经广播到网络。因此，PoH 很难生成，但容易验证。

不同 CPU 计算 SHA-256 的性能范围惊人地狭窄，最快的机器之间只有小差异。尽管在优化这一功能上投入了大量时间和精力，但一个常见的上限已经被达到，主要是由于比特币对它的依赖。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4349be3faf1f9fdc68321_AD_4nXf_lUpxGKOc1qJ54MLg3jpp1JmxqTDPu6XsFFUi1-wJSDJxbQkbuHJ62eY-j-y43iEb--wY7kmUfpAhdVP5C6asrPRT8pwwYx_-xLtQ74RrjfF_tsQZ7qCaX7sPnlCcp1HfC2ukTylLXZBmyPTAQ66XzQIa.png" loading="lazy" alt="">

在领导者的时间槽内，PoH 服务将接收来自银行阶段的新处理条目。当前的 PoH 哈希与条目中所有交易的哈希组合成下一个 PoH 哈希。这作为时间戳，将条目插入哈希链中，证明交易处理的顺序。这个过程不仅确认了时间的流逝，还作为交易的加密记录。

在一个区块中，有 800,000 个哈希。PoH 流还包括“滴答”，这些是空条目，表示领导者的存活性和时间的流逝，大约为秒的一小部分。一个滴答每 6.25 毫秒发生一次，结果是每个区块有 64 个滴答，总区块时间为 400 毫秒。

即使在不是领导者时，验证者也会持续运行 PoH 时钟，因为它在节点之间的同步过程中扮演着关键角色。

#### 记住

PoH 的关键好处在于，它确保必须遵守正确的领导者日程，即使区块生产者处于离线状态——这种状态称为“失职”。PoH 防止恶意验证者在轮到他们之前生成区块。

### 账户模型

> “在 SVM 中分离代码和状态是最好的设计决策。愿那些将这个概念宗教般地灌输到我脑中的嵌入式系统开发人员得到祝福。” — Anatoly Yakovenko，Solana 联合创始人

在 Solana 验证者中，全局状态保存在名为 AccountsDB 的账户数据库中。这个数据库负责存储所有账户，无论是在内存中还是在磁盘上。账户索引中的主要数据结构是哈希表，因此 AccountsDB 本质上是一个庞大的[键值存储](https://en.wikipedia.org/wiki/Key%E2%80%93value_database)。在这里，键是账户地址，值是账户数据。

随着时间的推移，Solana 的账户数量激增，已达到数亿个。这部分是因为，正如 Solana 开发者常说的，“Solana 上的一切都是账户！”

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4349a688e3be8052f3275_AD_4nXe0QyHr7VANpouZrXlkMbgd2UajiLT2TdfkS1jRCabg6Gx9wSI6CUgKHqY7rRNTH_cNIMXotN0c9_AjB955Gi-p0o5tDVb_3Toq6r8rgB3lheV_Jfz4wgwYmwreLKsLWIDd_Gc-JvIUPNdV5CiEIZm1bC_o.png" loading="lazy" alt="">

#### Solana 账户

账户是持久保存数据的容器，类似于计算机上的文件。它们有不同的形式：

- 用户账户：这些账户具有私钥，通常由钱包软件为用户生成。
- 数据账户：这些账户存储状态信息，例如用户持有的特定代币的数量。
- 程序账户：这些账户包含可执行字节码，类似于 Windows 上的 .exe 文件或 Mac 上的 .app 文件。
- 原生程序账户：这些是预部署的特殊程序账户，执行网络的各种核心功能。[例如](https://github.com/solana-labs/solana/tree/master/programs)，包括投票程序和 BPF 加载程序。

所有账户都有以下字段：

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b53ffce5eb4a9e1ace857c_66b53fd88d01393355d5c2db_Solana%2520report%2520tabl%253Be.png" loading="lazy" alt="">

### 程序

Solana 程序账户仅包含可执行逻辑。这意味着当程序运行时，它会修改其他账户的状态，但自身保持不变。代码和状态的分离使 Solana 与其他区块链不同，并支持许多优化。开发者主要使用 Rust 编写这些程序，Rust 是一种通用编程语言，以其对安全性和性能的强烈关注而[闻名](https://www.whitehouse.gov/wp-content/uploads/2024/02/Final-ONCD-Technical-Report.pdf)。此外，还提供了 TypeScript 和 Python 的多个 SDK，以便于创建应用程序前端并使其能够与网络进行程序化交互。

许多常见的功能由原生程序提供。例如，Solana 不需要开发者部署代码来创建代币。相反，指令会发送到预部署的原生程序，该程序将设置一个账户来存储代币的元数据，从而有效地创建一个新代币。

### 租金

租金机制旨在激励用户关闭账户，从而减少状态膨胀。要创建一个新账户，账户必须持有最低余额的 SOL，这被称为“租金免除”金额。这可以视为为了保持账户在验证者内存中存活的存储成本。如果账户的数据大小增加，最低余额租金要求也会相应增加。当账户不再需要时，可以关闭它，租金将退还给账户所有者。

例如，如果用户持有一个美元计价的稳定币，这一状态存储在一个代币账户中。目前，代币账户的租金免除金额为 0.002 SOL。如果用户将其所有稳定币余额转移给朋友，则代币账户可以关闭，用户将收到回 0.002 SOL。程序通常会自动处理账户关闭。还提供了一些应用程序来帮助用户清理旧的、未使用的账户，并收回存储在其中的少量 SOL。

### 所有权

虽然读取账户数据是普遍允许的，但 Solana 的所有权模型通过限制谁可以修改（写入）账户的数据来增强安全性。这一概念对于在 Solana 区块链上强制执行规则和权限至关重要。每个账户都有一个程序“所有者”。账户的所有者负责治理它，确保只有授权程序可以更改账户的数据。一个显著的例外是转移 lamports（SOL 的最小单位）——增加账户的 lamports 余额是普遍允许的，无论所有权如何。

### 状态存储

Solana 程序作为只读的可执行文件，必须使用“程序派生地址”（PDAs）来存储状态。PDAs 是特殊类型的账户，关联并由程序而不是特定用户拥有。虽然普通的 Solana 用户地址是从 Ed25519 密钥对的公钥派生的，但 PDA 没有私钥。相反，它们的公钥是从一组参数（通常是关键字或其他账户地址）与拥有程序的程序 ID（地址）结合起来派生的。

PDA 地址存在“曲线外”，即它们不在 Ed25519 曲线上。只有拥有 PDA 的程序可以以编程方式生成对其的签名，确保只有该程序可以修改 PDA 的状态。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4349bb5eeca22c2023fa0_AD_4nXcTjhv7lTwH5TMTybh5z4OgD1QKLBLO3ZweCyiPRFVLte7yYAhQhpB7InXoMDAPNtB72_SA-1SLzReTTIrqqLV7-vpxDQ28lzJYCkgTtI7PfFWwoRxnAcyUeV1RHTTgIFs_0Knb7YoPPPSqKl_x_TlkDZA.png" loading="lazy" alt="">

上图：Solana 代币账户是程序派生地址（PDA）的具体例子。它们用于存储代币并且“曲线外”存在。相关代币账户程序确保每个钱包对于每种代币类型只有一个关联代币账户，提供了一种标准化的方式来管理代币账户。

### Turbine

> “Solana 最有趣的部分不是并行化、SVM 或 Toly 的推文。实际上，你可能没听说过的部分才最引人注目，那就是 Turbine。” — Mert Mumtaz，Helius

在银行阶段，事务被组织成条目并发送到历史证明（PoH）流中进行时间戳处理。区块的银行更新完成后，条目准备进入下一阶段——Turbine。

Turbine 是领导者向网络其余部分传播其区块的过程。受到 BitTorrent 启发，它旨在快速高效，减少通信开销并最小化领导者需要发送的数据量。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4349b00c1705cb7e7494c_AD_4nXfGhylK10fzEZY9xYJ1VaQk8xYQHvHLsNSPR9zh5dEFaOChTV0cARiTR0utQT476N3btzSPvR16TmT1uc2vR3QgHI7rHeZNoJYOGu6Bgpmm_yv6k1nH4pLPRucbAx6WRYhUykDxL2rCUWpIKh1KO68kLaU.png" loading="lazy" alt="">

Turbine 通过将事务数据分解成称为“shreds”（碎片）的“小数据包”来实现这一目标，碎片的大小可达 1280 字节，类似于视频流中的单独帧。重新组装这些碎片可以让验证者重播整个区块。这些碎片通过互联网在验证者之间使用 UDP 发送，并利用纠删码处理数据包丢失或恶意丢包。[纠删码](https://www.cs.cmu.edu/~guyb/realworld/reedsolomon/reed_solomon_codes.html#)是一种基于[多项式](https://en.wikipedia.org/wiki/Polynomial)的错误检测和修正方案，确保数据完整性。即使一些碎片丢失，区块仍然可以被重建。

碎片被分组到称为前向纠错（FEC）批次中的批次中。默认情况下，这些批次由 64 个碎片组成（32 个数据碎片 + 32 个恢复碎片）。数据恢复发生在每个 FEC 批次中，这意味着每个批次中最多可以丢失或损坏一半的数据包，所有数据仍然可以恢复。每个 64 个碎片的批次通过根 Merkle 树进行哈希，根由领导者签名，并链接到上一个批次。这一过程确保了碎片可以从网络中任何拥有它们的节点处安全地获取，因为 Merkle 根链提供了可验证的真实性和完整性路径。

领导者最初向单一的根节点广播，根节点将碎片传播到所有其他验证者节点。每个碎片的根节点都会发生变化。验证者被组织成层次结构，形成“涡轮树”。通常，持有较大股份的验证者位于树的上层，而持有较少股份的验证者则位于下层。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4349c8a31948c21bafcfa_AD_4nXey4cEhqVVk9sDCsPIbhXy4UveKwsQloo9IoS5MQsorEW9YKmYBpnqpzHzw4Bw7LZKsTXKEIidAJNrzeT4tOZzXcX_8JTJ4DshUKh_ZsK9RGjZmoy3gePKWdVRpV2NYZJPKUmmc7ie7aO6qPoHBWkL_NDA.png" loading="lazy" alt="">

树通常跨越两到三跳，这取决于活动验证者的数量。为了简化视觉效果，上图显示了一个 3 的扇出值，但 Solana 的实际扇出值目前设定为 200。出于安全原因，树的顺序会在每个新批次的碎片中旋转。

这种系统的主要目标是减轻领导者和根节点的外发数据压力。通过利用传输和重传系统，负载在领导者和重传者之间分布，从而减少了单个节点的压力。

### 共识

> “有些聪明人告诉我，Solana 上有一个真诚聪明的开发者社区……我希望这个社区能获得公平的机会繁荣。” — Vitalik Buterin，以太坊联合创始人

一旦验证者通过 Turbine 收到新区块，它必须验证每个条目中的所有事务。这涉及到重播整个区块，并行验证 PoH 哈希，按照 PoH 指定的顺序重新创建事务，并更新本地银行。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4349a8fb72180030d3d15_AD_4nXcdAsuya_fDbr_ut8ft6oyUwr-6BR1xI9gUGQXh8gX2ecr4mW7CzJkgYpYJ41NrXlgEh7ts2EvQFfieP424J-CE_R_xk4JH406tgdYYbY_O0WAHF6Y0bBlETU7J_wmNCTH7HrmEXJjgkM0v9fBVEE9dgD9e.png" loading="lazy" alt="">

这一过程由事务验证单元（TVU）处理，类似于领导者的事务处理单元（TPU），负责处理碎片和区块验证。与 TPU 一样，TVU 流程被分解为几个阶段，从接收碎片阶段开始，碎片通过 Turbine 接收。在随后的验证领导者签名阶段，碎片经过多个校验，特别是验证领导者的签名，以确保接收到的碎片确实来自领导者。

在重传阶段，验证者根据其在 Turbine 树中的位置，将碎片转发给下游的验证者。在重播阶段，验证者按照正确的顺序重新创建每个事务，同时更新本地的银行状态。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4349a8b98661ab0c8bff2_AD_4nXdj5vP0p919AsDuxFNFE3iemkoi4sbtWclHWft6Ksn783XA-jhDE--MTjUcXweQozeRJsKNsL-_UZWnlI6BCfQWQrtaWCRY2_VSpOPJLOH0vBQPUIWvmSueQ_YqIy5dnu2G33eKIg_A1xx37wY3UpIPfrKq.png" loading="lazy" alt="">

重播阶段类似于 TPU 中的银行阶段；它是最重要的阶段，可以更直接地描述为区块验证阶段。重播是一个单线程的过程循环，协调许多关键操作，包括投票、重置 PoH 时钟和切换银行。

### 共识机制

为了达成共识，Solana 使用 Tower BFT（TBFT），这是对著名的实用[拜占庭](https://en.wikipedia.org/wiki/Byzantine_fault)容错（PBFT）算法的自定义实现。PBFT 通常被大多数区块链用于对链的状态达成一致。像所有区块链一样，Solana 假设网络中存在恶意节点，因此系统必须承受节点故障和某些攻击。

Tower BFT 的区别在于它利用了历史证明提供的同步时钟。传统的 PBFT 需要多轮通信来达成交易顺序的共识，而 Solana 节点利用事先建立的事件顺序，大大减少了消息传递的开销。

### 投票

为了参与共识并获得奖励，验证者提交他们认为有效的区块的投票（即没有双重支出或错误签名），并应被视为标准区块。验证者为这些投票支付事务费用，这些费用由领导者处理并与常规用户事务一起包含在区块中。这就是为什么 Solana 的事务通常被分为投票事务和非投票事务。当验证者提交正确且成功的投票时，他们会获得信用。这一机制激励验证者投票支持他们认为最有可能被纳入的分叉，即“最重”的分叉。

### 分叉

Solana 的设计部分在于，网络不会等待所有验证者对新生产的区块达成一致才生产下一个区块。因此，两个不同的区块链接到同一个父区块是常见的，这会创建分叉。

Solana 验证者必须对这些分叉进行投票，并使用共识算法来确定采用哪个分叉。当存在竞争的分叉时，网络最终会确认一个分叉，而被丢弃的分叉中的区块将被放弃。

每个插槽都有一个预定的领导者，只有该领导者的区块会被接受；一个插槽不能有两个提议的区块。因此，潜在的分叉数量被限制为在领导者轮换插槽的边界处的“有/无”跳表。验证者一旦选择了一个分叉，就会坚持这个选择，直到锁定时间过期，这意味着它必须在最短时间内保持选择。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4349aaaa47c8e92151ccd_AD_4nXeHm-ZwJq6JwgzYBoN7KiFql4SF7WjVTruhaIwW8DXmypK7KZeO3lZsc65uUvRbx09cL1aQ6wgQrzVM4-APUoj-fsYHJ0pwJx7UxNpII9aM4aDolokJMkrFvcDmu2_0dBQYD8biYpVjwG6fXb1HdQNkIDTI.png" loading="lazy" alt="">

Solana 的“跳跃率”——区块未生产的插槽的百分比——从 2% 到 10% 不等，而分叉是这些跳过插槽的主要原因。其他可能导致插槽跳过的原因包括新的纪元的开始、领导者离线或生产了无效区块。

### 记住

Solana 上事务的状态根据其在共识过程中的当前阶段而异：

- 处理：事务已被包含在一个区块中。

- 确认：事务的区块已被三分之二的超大多数投票确认。

- 最终：在事务的区块上已建立超过 31 个区块。

截至目前，Solana 的历史上从未出现过一个（乐观地）确认的区块未能最终确定的情况。

对于每个区块，Solana 使用一个银行来访问该区块的状态。当一个银行被最终确定时，该银行及其祖先的账户更新被刷新到磁盘。此外，所有来自早期银行的账户更新，但不是最终银行的祖先，都被修剪。这一过程使 Solana 能够高效地维护多个潜在状态。

### Gossip + Archive

> “一个区块链需要巧妙地结合密码学、分布式系统、操作系统和编程语言。Solana 的超级能力是愿意逃离每个学科中最有趣的问题。” — Greg Fitzgerald，Solana 联合创始人

#### Gossip

Gossip 网络可以被视为 Solana 网络的[控制平面](https://en.wikipedia.org/wiki/Control_plane)。与处理事务流的数据平面不同，控制平面传播有关区块链状态的关键元数据，如联系信息、账本高度和投票信息。如果没有 Gossip，验证者和 RPC 将不知道哪些地址和端口开放以进行各种服务的通信。新的节点也依赖 Gossip 加入网络。

Solana 的 Gossip 协议使用非正式的点对点通信，采用一种改进的 PlumTree 算法的树广播方法。这种方法高效地传播信息，无需依赖任何中心源。

Gossip 操作有点像一个孤立的系统，独立于大多数其他验证者组件。验证者和 RPC 每 0.1 秒通过 UDP 通过 Gossip 共享签名的数据对象，以确保信息在网络中的可用性。所有 Gossip 消息必须小于或等于最大传输单元（MTU）1280 字节，称为代码库中的“数据包结构”。

Gossip 记录是节点之间共享的实际数据对象。大约有 10 种不同类型的记录，每种记录都有不同的用途。Gossip 

记录是签名的、版本化的和时间戳的，以确保完整性和时效性。

有四种类型的 Gossip 消息：

- **推送**：最常见的消息，与子集的“推送对等体”共享信息。
- **拉取 & 拉取响应**：定期检查错过的消息，拉取响应会返回节点没有的信息。
- **修剪**：允许节点选择性地减少它们维持的连接数量。
- **Ping & Pong**：节点的健康检查——如果发送了 Ping，则预期会收到 Pong，表示对等节点仍然活跃。

Gossip 数据存储在一个集群复制数据存储（CrdsTable）中。这个数据结构可以变得非常大，需要定期修剪。

#### Archive

Solana 与其他区块链的不同之处在于，它不需要整个历史来确定账户的当前状态。Solana 的账户模型确保在任何给定插槽上的状态是已知的，允许验证者仅存储每个账户的当前状态，而不需要处理所有历史区块。RPC 和验证者设计上不保留整个历史账本。相反，它们通常只存储 1 或 2 个纪元（2-4 天）的事务数据，这足以验证链的尖端。

归档由“仓库节点”管理，这些节点由专业的 RPC 服务提供商、Solana 基金会和其他对确保事务历史可用感兴趣的生态系统参与者运营。仓库节点通常维护以下一种或两种：

1. **账本归档**：上传原始账本和适用于从头开始重播的 AccountsDB 快照。
2. **Google Bigtable 实例**：存储从创世区块开始的区块数据，格式化以服务 RPC 请求。

### Economics + Jito

> “人们意识到 Solana 是今天唯一一个能够支持主流消费者应用的链。” — Ted Livingston，Code 创始人

Solana 通过通货膨胀来分配质押奖励，通过每个纪元生成新的 SOL 代币。这个过程导致非质押者的网络份额相对于质押者减少，从而导致财富从非质押者转移到质押者。通货膨胀从 2021 年初的 8% 开始，每年减少 15%，直到稳定在长期的 1.5% 的比率。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b4349c14ad9de571ca7532_AD_4nXe-oef2rXEYmBkbyp5S2BDkUicRlScGGaSTXHga4NRLx7o87JaPiRB6QR6sexDt8XZwR0DFj_YOAmaL1KljP3-r4EEij2YSURR-fRd63SVA1QPX59U8BAmHlyZTrsB5YOWuqJODhG6oNvXJbRy2Ghg95Ssu.png" loading="lazy" alt="">

任何 SOL 代币持有者都可以通过将代币质押到一个或多个验证者来赚取奖励并帮助保护网络。将代币委托给验证者被称为委托。将代币委托给验证者表示信任验证者，但并不会赋予验证者对代币的所有权或控制权。所有质押、撤质押和委托操作都在下一个新的纪元开始时执行。

#### 投票奖励

当验证者提交投票时，如果投票正确且成功，他们会获得信用。投票事务费用为 0.000005 SOL，并免于优先费用。投票费用约为每个验证者每天 1 SOL，这也是运行验证者的主要运营成本。在一个纪元内，验证者通过投票积累信用，这些信用可以在纪元结束时兑换为通货膨胀的份额。

表现最好的验证者通常成功投票约 90% 的插槽。值得注意的是，没有区块的插槽的百分比（跳过插槽率）范围从 2% 到超过 10%，这些插槽不能投票。平均验证者成功投票约 80% 的插槽，在一个 432,000 个插槽的纪元中获得 345,600 个信用。

总通货膨胀池首先根据纪元中获得的信用进行分配。验证者的信用份额（他们的信用除以所有验证者信用的总和）决定了他们的相应奖励。这进一步根据质押进行加权。

因此，一位拥有 1% 总质押的验证者应大致获得 1% 总通货膨胀的奖励，如果他们拥有平均数量的信用。如果他们的信用高于或低于平均水平，他们的奖励将相应波动。

投票表现的差异是验证者提供给质押者的回报（按年化收益率计算）不同的一个原因。另一个因素是验证者收取的佣金率，这是从他们的验证者中直接获取的通货膨胀奖励的一部分。此外，验证者离线或与区块链不同步（称为失职）会显著影响回报。

#### 区块奖励

被指定为特定区块的领导者的验证者会收到额外的区块奖励。这些奖励包括所有事务中的基础费用和优先费用的 50%，其余费用被销毁。只有生产区块的验证者可以获得这些奖励。与质押奖励不同，区块奖励在区块生成时会立即记入验证者的身份账户。

#### 流动质押

流动质押已成为传统质押的流行替代方案。参与者在质押其 SOL 时会收到一个称为流动质押代币（LST）或流动质押衍生品（LSD）的代币，通常在一个质押池中将他们的代币委托给多个验证者。新获得的 LST 代币代表用户在质押 SOL 中的份额。这些代币可以被交易、在应用程序中使用或转让，同时仍然获得质押奖励。这种系统的主要优势是显著提高了资本效率。

```
LST 的价格 = （池中总质押 SOL * SOL 价格）/ 总 LST 铸造数量
```

与传统的原生质押相比，随着时间的推移，质押者将直接增加更多的 SOL。而在流动质押中，奖励被再投资到池中，提高了 LST 的公平价值。只要有机制可以将 LST 兑换回基础的质押 SOL，套利交易者将确保代币价格保持合理。

#### Jito

截至撰写时，超过 80%（[来源](https://jito.retool.com/embedded/public/3557dd68-f772-4f4f-8a7b-f479941dba02)）的质押在 Solana 上使用 Jito 客户端验证器软件。这个客户端是原始 Agave 客户端的一个分支，引入了一个协议外的区块空间拍卖，为验证者提供额外的经济激励。这个额外的激励是 Jito 客户端在验证者中广泛采用的主要原因之一。

当领导者使用 Jito 验证器客户端时，他们的事务最初会被发送到 Jito-Relayer。这款开源软件充当事务代理路由器。其他网络节点并不知道 Jito-Relayer 的存在，因为它们只是将事务发送到领导者通过 Gossip 网络公布的地址和端口配置，假设这是领导者的。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66b41fe65de8a0d019876ca2_66b41fd8bd58bc7bf7d972c8_Solana%2520Report%2520Visuals%2520for%2520Helius%2520Blog.pptx%2520(15).png" loading="lazy" alt="">

Relayer 在转发事务到领导者之前，会持有所有事务 200 毫秒。这个“速度缓冲”机制延迟了传入事务消息，为拍卖提供了一个短暂的窗口。200 毫秒后，无论拍卖结果如何，Relayer 都会乐观地释放事务。

区块空间拍卖通过 Jito 区块引擎离线进行，允许搜索者和应用程序提交一组原子执行的事务，称为捆绑。这些捆绑通常包含时间敏感的事务，如套利或清算。Jito 对所有小费收取 5% 的费用，最低小费为 10,000 lamports。小费完全是在协议外运作，与协议内的优先和基础费用分开。之前，Jito 运行了一个标准的协议外内存池服务，但现在已经被弃用。