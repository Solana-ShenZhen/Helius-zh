# 质押加权服务质量：您需要了解的一切

## 介绍

Solana 作为一个高吞吐量、低延迟的网络，处于区块链技术的前沿，不断推动着去中心化网络可以实现的边界。然而，这也带来了重大挑战。Solana 发展过程中的一个关键时刻是 [2022 年 4 月 30 日的网络中断](https://solana.com/news/04-30-22-solana-mainnet-beta-outage-report-mitigation)，这凸显了需要更强大的机制来处理高交易量并在高负载下维持网络性能。

质押加权服务质量（Stake-Weighted Quality of Service，SWQoS）就是为了应对这一事件而创建的。这种机制根据验证者持有的质押量来优先处理网络流量，确保拥有更多质押的验证者可以发送更高优先级的交易。SWQoS 旨在防止低质押验证者压垮网络，从而增强 Solana 的弹性和效率。

本文探讨了 SWQoS、Solana 的交易处理模型以及 SWQoS 如何影响它。文章还探讨了质押连接、它们的设置以及 SWQoS 与优先费用之间的区别。此外，本文还讨论了未来的影响，特别是验证者和流动性质押代币（LSTs）日益增长的重要性、准入门槛以及这个系统固有的信任假设。

本文假设读者已了解 Solana 的编程模型和 QUIC 协议。如果您对这些主题不熟悉，建议在深入阅读本文之前先阅读以下文章：

1. [Solana 编程模型：Solana 开发入门](https://www.helius.dev/blog/the-solana-programming-model-an-introduction-to-developing-on-solana)
2. [如何快速缓解垃圾信息：关于 Solana 和 QUIC 的所有必知信息](https://www.helius.dev/blog/all-you-need-to-know-about-solana-and-quic)

尽管如此，本文在必要时也会提供相关背景信息。

## 什么是质押加权服务质量（SWQoS）？

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66562ea9a6640bfbad5081ff_SWQoS-Diagram.jpg" alt="质押加权服务质量（SWQoS）示意图">

<p align="center"><em>图片来源：改编自 <a href="https://x.com/rexstjohn/status/1770859892967960907/photo/1">Rex St. John 在 Twitter 上发布的图表</a></em></p>

质押加权服务质量（Stake-weighted Quality of Service，SWQoS）是一种基于验证者持有的质押量来优先处理网络流量的机制。这种机制确保拥有更多质押的验证者能够更有效地发送交易，从而提高他们的服务质量。

鉴于 Solana 是一个权益证明网络，将质押权重扩展到交易性能是很自然的。简单来说，Solana 使用质押（即锁定在验证者处以保护网络安全的资金）来表示验证者的可信度。验证者持有的质押越多，他们在网络的安全性和可靠性方面就有越大的利益。此外，[服务质量](https://en.wikipedia.org/wiki/Quality_of_service)是一个网络概念，其中某些数据包被优先处理，以便在网络上获得更可靠的性能。因此，Solana 通过将交易路由到优先连接，为质押的验证者在发送交易时提供更可靠的性能。

SWQoS 的主要目的是防止低质押验证者用大量交易淹没网络，从而冲掉来自高质量或高质押验证者的交易。例如，如果一个验证者持有总质押量的 5%，他们就可以向领导者发送总数据包的 5%。SWQoS 可以被视为一种 [Sybil 攻击防御机制](https://en.wikipedia.org/wiki/Sybil_attack)，使恶意行为者更难用"低质量"交易淹没网络。

想象一下，你正在参加一个热门活动，比如音乐会或游乐园，门票数量有限。有普通的售票队伍，任何人都可以排队购票，但这些队伍可能会很长且缓慢。然而，还有为高付费顾客预留的 VIP 队伍。这些 VIP 队伍要短得多，也快得多，因为只有满足特定条件的人才能进入，比如拥有特定会员资格，而且每次都保证一定数量的人能入场。在 Solana 的背景下，SWQoS 就像这些 VIP 队伍，准入由验证者持有的质押量决定。你拥有的质押越多，就能获得越多的 VIP 访问权（即优先连接），确保更快速、更可靠的票务（交易）处理。

那么，这在实践中是如何运作的呢？首先，重要的是要理解 Solana 如何处理交易。

## Solana 如何处理交易

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66562ed76e815e5073fcf856_TPU-Diagram.jpg" alt="Solana 交易处理单元 (TPU) 示意图">

<p align="center"><em>图片来源：改编自 <a href="https://docs.solanalabs.com/validator/tpu">Solana 官方文档</a></em></p>

Solana 的交易处理单元（Transaction Processing Unit，TPU）高效地处理和执行交易。处理过程通过几个不同的阶段进行，以确保交易被验证、执行并在网络中传播。这些阶段包括获取阶段（Fetch Stage）、签名验证阶段（SigVerify Stage）、银行阶段（Banking Stage）、历史证明服务（Proof of History，PoH Service）和广播阶段（Broadcast Stage）。

### 获取阶段

[获取阶段](https://github.com/solana-labs/solana/blob/master/core/src/fetch_stage.rs)通过网络接收来自客户端的传入交易。它从 UDP 套接字批量接收输入，并将其分类为三个主要套接字：

• tpu：用于常规交易，如代币转账、NFT 铸造和程序交互

• tpu_vote：用于投票交易

• tpu_forwards：当当前领导者无法处理所有交易时，将未处理的数据包转发给下一个领导者

这些套接字在[ Gossip 中创建](https://github.com/solana-labs/solana/blob/27eff8408b7223bb3c4ab70523f8a8dca3ca6645/gossip/src/cluster_info.rs#L2962)，并存储在[ ContactInfo 结构体中，由其对应的套接字标记](https://github.com/solana-labs/solana/blob/27eff8408b7223bb3c4ab70523f8a8dca3ca6645/gossip/src/contact_info.rs#L28-L39)。

获取阶段使用一种机制来合并同时接收的数据包，减少单个数据包处理操作的数量，同时提高吞吐量。当它接收到转发的数据包时，会用[ FORWARDED 标志](https://github.com/solana-labs/solana/blob/12fe7d39e89dd9a0377adbdded5f36c0e5d9a3ac/core/src/fetch_stage.rs#L104)标记它们，以确保在后续阶段中能够相应地识别它们。如果节点不是当前的领导者，这些转发的数据包会被丢弃以避免不必要的处理。然而，如果节点是领导者，这些数据包会被接受并相应地处理。

获取阶段创建无界通道（例如，packet_sender、packet_receiver）来将交易传递到下一个阶段，即签名验证阶段。这些通道将 TPU 的各个阶段解耦，使它们能够并发运行而不会相互阻塞。无界函数创建了一个具有无限容量的通道，以确保数据包不会因通道溢出而被丢弃。

通过高效管理交易的接收和分类，获取阶段为 Solana 交易处理单元（TPU）中所有后续处理阶段奠定了基础。

### 签名验证阶段

签名验证阶段是 Solana 交易处理流程中的第二个阶段。它对于确保交易的完整性和真实性至关重要。

在这个阶段，TPU 通过无界通道从获取阶段接收批量交易。这里的主要任务是使用[ Ed25519 签名方案](https://en.wikipedia.org/wiki/EdDSA#Ed25519)验证交易签名。这种加密验证确认了涉及的账户的正确所有者签署了交易。

签名验证阶段被设计为高性能的，利用现代 CPU 和 GPU 的并行处理能力来验证签名。默认情况下，CPU 执行所有处理。然而，由于其并行化的特性，当有性能库可用时，[GPU 卸载可以显著加快处理速度](https://github.com/solana-labs/solana/blob/12fe7d39e89dd9a0377adbdded5f36c0e5d9a3ac/core/src/sigverify_stage.rs#L5-L6)。

该过程从[ new ](https://github.com/solana-labs/solana/blob/12fe7d39e89dd9a0377adbdded5f36c0e5d9a3ac/core/src/sigverify_stage.rs#L238)函数开始，该函数初始化签名验证阶段并设置必要的接收通道以从获取阶段接收数据包。[verifier ](https://github.com/solana-labs/solana/blob/12fe7d39e89dd9a0377adbdded5f36c0e5d9a3ac/core/src/sigverify_stage.rs#L293)函数负责接收括号并验证签名。它处理重复数据删除，丢弃多余的数据包，并验证剩余的数据包。该函数使用[ ed25519_verify 方法进行签名验证](https://github.com/solana-labs/solana/blob/12fe7d39e89dd9a0377adbdded5f36c0e5d9a3ac/core/src/sigverify.rs#L141-L147)。在高交易量期间，会发生负载削减。这是签名验证阶段丢弃多余数据包的地方。[它通过按源 IP 地址对数据包进行分组，并为每个地址分配最大处理数据包数来实现这一点](https://github.com/solana-labs/solana/blob/12fe7d39e89dd9a0377adbdded5f36c0e5d9a3ac/core/src/sigverify_stage.rs#L247-L275)。

签名无效的交易会被标记并丢弃。这个丢弃过程确保只有有效的交易才能继续前进，防止欺诈或不正确的交易进入下一个阶段。

在签名验证之后，有效的交易通过另一组无界通道传递到下一个阶段（即银行阶段）。这确保了各个阶段保持解耦，并能够并发运行。值得注意的是，这个阶段在多个线程中运行，每个线程处理一部分交易，验证签名，并执行重复数据删除和负载削减。

### 银行阶段

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66562f3b8f5834fd13e09d16_SharedChannel-Diagram.jpg" alt="">

[银行阶段](https://github.com/solana-labs/solana/blob/v1.18/core/src/banking_stage.rs)是 Solana 交易处理流程中的第三个阶段。这个阶段至关重要，因为它是交易被执行并应用到当前账本状态的地方。银行阶段利用 Solana 独特的 Sealevel 运行时来实现高吞吐量的并行交易处理。

银行阶段有六个线程——两个专门用于处理来自 TPU 或 Gossip 的投票交易，四个专门用于非投票交易。每个线程独立运行，从一个共享通道接收数据包（注意这将在 1.18 版本后发生变化），SigVerify 在这个通道中批量发送数据包。每个线程从这个共享通道中提取交易并将它们存储在本地缓冲区中。本地缓冲区充当优先队列，动态更新以反映交易状态和网络需求的实时变化。

这些交易的处理取决于验证节点是否在领导者调度中。如果验证节点距离成为领导者还很远，它会将数据包转发给即将到来的领导者并丢弃它们。当验证节点接近其轮次（约 20 个槽位之前），它会继续转发数据包，但会保留它们，以防即将到来的领导者未能处理它们。当验证节点距离成为领导者还有两个槽位时，它会保留这些数据包，以确保在成为领导者时能够处理它们。

在区块生产期间，每个线程通过从其本地队列中取出前 128 个交易来处理交易。这个过程涉及锁定、检查、加载、执行、记录、提交和解锁等步骤。

银行阶段使用多迭代器方法来批处理交易。这种方法允许同时遍历数据集，将交易分组为非冲突的批次。首先，交易被序列化到一个基于优先级的向量中。然后，多迭代器在交易不冲突的点放置迭代器，创建 128 个交易的批次。冲突的交易被跳过，并在解决后包含在后续批次中。形成批次后，交易被执行。成功的交易被记录在历史证明服务中，并通过[ Turbine ](https://www.helius.dev/blog/turbine-block-propagation-on-solana)广播到网络。

### 历史证明服务

历史证明服务（Proof of History，PoH）是 Solana 交易处理流程中的一个基本组件。它提供了一种可验证的方式来跟踪网络内的时间和事件顺序，确保交易的高效和安全排序。它通过哈希链生成一个加密序列，作为交易的时间戳。这个连续的哈希链创建了一个历史记录，证明了事件之间时间的流逝。

PoH 服务确保所有网络参与者可以就交易顺序达成一致，而无需中央时间管理器。它还有助于同步整个网络的验证节点。通过提供可靠的时间戳来确定验证节点何时应成为领导者并生成下一个区块，PoH 服务支持 Solana 的领导者选举过程。

PoH 服务首先初始化一个种子值，该值生成一系列哈希。当接收到交易时，它们会被标记上 PoH 序列中的当前哈希，为它们提供一个唯一的时间戳。然后，验证者验证这个哈希序列，以确认交易的顺序和时间。

要了解更多关于历史证明的信息，请阅读我们的文章《[历史证明、权益证明、工作量证明 — 解释](https://www.helius.dev/blog/proof-of-history-proof-of-stake-proof-of-work-explained)》。如果密码学对你来说听起来像是外语，我们还建议阅读我们的文章《[密码学工具 101 — 哈希函数和默克尔树解释](https://www.helius.dev/blog/cryptographic-tools-101-hash-functions-and-merkle-trees-explained)》。

### 广播阶段

广播阶段是 Solana 交易处理流程的最后一步。它负责将已验证和确认的交易分发到网络的其他部分。

一旦交易在银行阶段被处理和提交，它们就会被组成条目（entries）。这些条目随后被打包成称为碎片（shreds）的数据结构。广播阶段对这些碎片进行序列化，签名，并生成纠删码以增强数据完整性和恢复能力。碎片通过一种称为 Turbine 的结构化、树状传播过程发送给对等节点。这是一个高效且冗余的分发过程，纠删码允许验证者重构丢失或损坏的数据。

要更全面地了解 Turbine，请阅读我们的文章《[Turbine：Solana 上的区块传播](https://www.helius.dev/blog/turbine-block-propagation-on-solana)》。

### SWQoS 交易的生命周期

与其他区块链不同，Solana 没有交易在处理前等待的内存池。相反，交易直接路由到当前的领导者并由其 TPU 处理。用户创建交易，无论是直接还是间接地通过钱包或应用程序，并通过[ JSON RPC API ](https://solana.com/docs/rpc)将其提交给[ RPC 节点](https://www.helius.dev/blog/solana-nodes-a-primer-on-solana-rpcs-validators-and-rpc-providers#rpc-nodes)。这些节点充当用户和 Solana 验证者之间的中介。重要的是，它们不应在网络中拥有任何权益。也就是说，RPC 节点是无权益的、不投票的，因此是非共识的。

现在与领导者的连接是通过 QUIC 进行的。QUIC 已被添加到接收用户交易的端口，以替代 Solana 的 TPU 中的 UDP。由于 QUIC 需要握手，可以对参与者的流量设置限制，使网络能够专注于处理真实交易，同时过滤掉垃圾交易。当然，这是实施 QUIC 的初衷，本文不会讨论其当前的有效性。需要注意的重要一点是，与领导者的连接是通过 QUIC 进行的。

有两种类型的连接：

• 500 个开放连接，任何 RPC 节点都可以访问。
• 2000 个质押加权连接，仅限于有质押的验证者。验证者根据其质押获得这些连接的比例份额。

为了使 RPC 能够有效地中继交易，它必须与一个有质押的验证者建立对等连接。由于 RPC 在网络中没有任何质押，验证者需要虚拟地扩展他们的质押。验证者可以使用 --staked-nodes-overrides 标志将他们的部分质押连接分配给特定的 RPC 节点。

验证者必须使用 --staked-nodes-overrides 标志指定一个 YAML 文件路径来配置他们的质押加权连接。YAML 文件包含以下形式的映射：

```
staked_map_id:
	<pubkey_of_RPC>: 80000000000000000
```

给定 RPC 身份的每个公钥都需要一个以 lamports 为单位的值。这个值指定了你想给予 RPC 身份的质押权重。例如，如果你指定了一百万 SOL，你实际分配给它们的是一百万 SOL 除以总活跃质押量。本质上，你是在给 RPC 节点分配质押连接，就好像它是一个在网络中拥有那么多质押的验证者。你相当于在说："在我的本地视图中，当与我的验证者通信时，将这个 RPC 身份视为拥有 x 数量的质押。"注意，这种设置不需要验证者重启 — 可以对文件进行更改并即时重新加载。

此外，[Jito 中继器支持质押节点覆盖标志](https://github.com/jito-foundation/jito-relayer/pull/119)。建议将中继器与质押节点运行在同一台机器上以优化性能。

要使用这些质押连接，RPC 运营商必须使用 --rpc-send-transaction-tpu-peer 标志，该标志需要质押验证者的 TPU 的 IP 和端口。TPU 端口通常从动态端口范围加三开始，可以在 Gossip 中找到。对于 Jito，流量将转到运营商运行的中继器，因为无法使用公共 Jito 中继器。RPC 运营商应该检查他们的日志中是否有 solana_quic_client 和 warm 等条目来验证他们的连接。注意，这种设置要求 RPC 节点运行 v1.17.28 或更高版本的 Agave 客户端以支持必要的标志。

总的来说，使用 SWQoS 的交易生命周期与普通交易基本相同。用户通过 RPC 节点创建和提交交易，然后将其发送给领导者。然而，当 RPC 节点与有质押的验证者建立对等连接并使用 --rpc-send-transaction-tpu-peer 标志时，交易会通过质押连接发送。总结如下：

1. 交易创建：用户使用钱包、应用程序或以编程方式创建交易
2. 提交到 RPC 节点：通过 JSON RPC API 将交易提交到 RPC 节点
3. QUIC 连接：RPC 节点与领导者建立 QUIC 连接，根据其配置利用开放或质押加权连接
4. 质押加权 QoS：如果 RPC 节点与有质押的验证者建立了对等连接，它会使用验证者的质押连接，从而提高交易性能
5. 转发给领导者：交易通过这些质押连接发送给领导者，被延迟或丢弃的可能性较低
6. 交易处理：交易通过 TPU（如前所述），并由领导者处理

SWQoS 通过确保有质押的验证者及其对等的 RPC 节点能更好地访问领导者，从而增强了交易生命周期，减少了因网络拥塞导致延迟的可能性。这种机制与优先费用一起工作，以提高交易性能。

## SWQoS 与优先费用：明确的区别

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/665630413e633874591cb8ef_highway.jpg" alt="高速公路收费车道和普通车道">

<p align="center"><em>图片来源：Nathaniel Minor 为<a href="https://www.cpr.org/news/">CPR News</a>撰写的《<a href="https://www.cpr.org/2019/11/07/whether-you-love-toll-lanes-or-hate-em-you-can-expect-them-pretty-much-everywhere-on-the-front-range/">无论你喜欢还是讨厌收费车道，你都可以在前沿地区的几乎所有地方看到它们</a>》</em></p>

在网络拥塞的情况下，SWQoS 确保高质押验证者的交易不太可能被延迟或丢弃。这个系统通常被比作收费公路，其中质押更多的验证者可以访问拥堵较少的路径，类似于高速公路上的更多车道。以上图为例。在科罗拉多州，司机可以选择在交通拥堵中等待，或者多付几美元使用收费车道。[快速车道](https://www.codot.gov/programs/expresslanes)是科罗拉多州的高级车道网络，其价格不断调整，以保持足够高的水平来确保交通流畅。因此，我们可以将 Solana 的质押连接比作科罗拉多州的快速车道，因为它们都是旨在为高级用户减少拥堵的优先通道。

然而，区分 SWQoS 和优先费用是至关重要的，因为常见的收费公路类比可能会模糊这两个概念之间的区别：

• 优先费用在银行阶段发挥作用，领导者根据支付的费用对交易进行优先排序。这个想法是，支付更高费用的交易将被更早处理，确保愿意支付更多的用户能获得更快的执行。

• SWQoS 改善了连接访问，但不影响领导者交易队列中的交易优先级。SWQoS 确保有质押的验证者能更好地访问网络，减少了交易因网络拥塞而被延迟或丢弃的可能性。

虽然优先费用影响交易在领导者队列中的排序和处理方式，但 SWQoS 确保从有质押的验证者发送的交易有一个优先通道到达领导者。这两种机制都旨在提高网络性能，但在交易生命周期的不同阶段发挥作用。

## SWQoS 战争

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/6656306ddb77c3c26b942560_Untitled.png" alt="">

SWQoS 将成为 Solana 网络基础设施未来发展的一个基本方面。它将通过优化交易处理和优先处理来自有质押验证者到领导者的连接，从而显著影响生态系统。我怎么强调这一事实都不为过。

可以说，当网络不拥堵且交易不时间敏感时，使用高质押验证者的优势开始减弱。这是因为网络有足够的容量快速处理交易，无论其背后的质押权重如何。然而，随着 Solana 需求的增加，大规模采用即将到来，网络可能并不总是有足够的容量来处理每一笔交易而不出现延迟或偶尔的丢弃。这并不是说 Solana 无法扩展，因为它可以说是最有可能实现可扩展区块链的机会之一，如果不是最好的话。关键是，随着需求的增长，SWQoS 将继续对卓越的用户体验至关重要。

反过来，验证者的角色变得更加至关重要，这导致了更多的竞争和创新，以提供尽可能最好的服务。自然而然，这是否意味着每个人都会想要启动自己的验证者节点呢？

## 验证者和流动性质押代币（LSTs）

趋势很明显：每个严肃的 Solana 协议都将运行一个验证者。这是因为他们需要支持自己的应用程序。如果你有兴趣运行自己的验证者，我们在 Helius 博客上有以下[指南](https://www.helius.dev/blog/how-to-set-up-a-solana-validator)，介绍如何开始。

我们正处于 Solana 上 LSTs 大规模寒武纪爆发的边缘，这种情况因 SWQoS 而加剧。仅在本月，总质押量（即原生 + LST）就[增加](https://dune.com/queries/3240537/5421270)了约 320 万。仅 LSTs 的市场价值目前在本月达到[ 60 亿至 90 亿美元](https://dune.com/queries/3133856/5227408)的范围。像 [Sanctum](https://www.sanctum.so/) 这样的协议正在成为重要参与者，因为其平台使验证者和应用程序能够创建自己的 LSTs。此外，[Picasso Network](https://x.com/Picasso_Network) 在 Solana 上实现了重新质押，充当 LSTs 获得更大效用和收益的中心。因此，创建具有附加效用的 LST 的能力已经存在，并且正在全力发展。

然而，必须考虑一些潜在的进入障碍。

## 进入障碍

尽管 SWQoS 具有潜在的好处，但它也引入了一些进入障碍。具体来说，在[ Agave 客户端的 v1.17.31 ](https://github.com/anza-xyz/agave/releases/tag/v1.17.31)版本中引入了[最低质押要求](https://github.com/anza-xyz/agave/pull/701)，将低质押验证者视为未质押对等节点。问题在于，低质押的节点可能会滥用质押连接，获得不成比例的带宽。现在，质押比例低于以下公式的节点将被视为未质押：

```
stake / total_stake < 1 / (max packet per 100ms)
```

这意味着质押少于约 15,000 SOL 的客户端现在被归类为未质押的验证者。这一要求可能会进一步增加运行 Solana 验证者已经很高的财务和技术要求，可能会排除较小的参与者和独立验证者参与网络。这一要求相当于大约 300 万美元，乍看之下是一笔巨款。

然而，正如[ Austin Federa 指出的](https://x.com/Austin_Federa/status/1778704360618213511)，这个门槛只是总质押量的 1/25,000，可以说是相当低的。此外，如果大多数竞争质押的验证者不断改进他们的性能以提供最佳服务并最大化自己的回报，这将有利于整个网络。这正是[ Toly 所描述的 SWQoS 的总体目标](https://x.com/aeyakovenko/status/1784273069353426972)。

在 Helius，我们正在降低所有使用 Helius 推荐费用发送交易的付费计划的准入门槛。也就是说，任何通过付费共享计划发送交易的用户，如果发送的费用达到或超过我们的[优先费用 API ](https://docs.helius.dev/solana-rpc-nodes/priority-fee-api#helius-priority-fee-api)提供的推荐值，现在都将通过我们的质押连接进行路由。我们还通过在[ Node.js ](https://github.com/helius-labs/helius-sdk)和[ Rust SDK ](https://github.com/helius-labs/helius-rust-sdk)中添加[新的智能交易](https://docs.helius.dev/solana-rpc-nodes/new-sending-solana-transactions)功能简化了这个过程。在最基本的层面上，无论使用哪种 SDK，用户只需提供他们的密钥对和希望执行的指令，我们就会处理其余的部分。现在，用户只需[每月 50 美元的开发者计划](https://www.helius.dev/pricing)就可以访问共享的质押连接。请注意，我们还提供专用的质押连接，这些连接保证质押连接带宽，推荐给企业、量化交易者和交易公司使用。我们为所有对专用质押连接感兴趣的人提供了[以下表格](https://form.typeform.com/to/STE2XEDw)。

重要的是要注意，质押可能会开始集中在由大型协议和可以收取 0% 佣金的 RPC 提供商运行的验证者中。以 Helius 为例；我们可以在其他地方赚钱来抵消运行验证者的成本。我们这样做是为了通过让一个 Solana 原生团队运行的顶级验证者来改善网络的去中心化，同时获得更多对质押连接的访问权限，以改善我们用户的体验。

## 信任假设

如果质押可能集中在由大型协议和 RPC 提供商运行的验证者中，那么选择一个以 Solana 最佳利益为重的验证者进行质押就变得非常重要。SWQoS 引入了几个信任假设。

其中一个主要的信任假设是验证者和 RPC 节点之间需要高度信任。正如前文所述，SWQoS 允许领导者识别并优先处理来自有质押验证者的交易。由于 RPC 节点是未质押、非投票和非共识的，它们无法像有质押的验证者那样直接从优先交易中受益。因此，验证者和 RPC 节点之间必须建立信任关系，以利用 SWQoS 的优势。

这种信任关系至关重要，因为启用 SWQoS 涉及共享敏感的网络配置，并允许 RPC 节点影响交易优先级。验证者必须确保与之对等的 RPC 节点会以网络的最佳利益行事，而不会滥用扩展的质押权用于恶意目的。验证者和 RPC 节点应该事先就如何使用质押连接达成协议和相互理解。理想情况下，这种设置应该在高度信任的实体之间进行，比如长期合作伙伴或同一组织内部。

目前，这些关系通常是隐藏的。展望未来，我不认为会出现 RPC 运营商不与验证者就质押覆盖进行协商的情况。需要更大的透明度，以便普通用户知道他们支持哪些 RPC 和验证者。随着时间的推移，对此的需求只会增加。

## 总结

SWQoS 有望彻底改变 Solana 的网络基础设施。虽然它提供了许多好处，包括增强的交易性能和改善的 Sybil 抵抗能力，但它也引入了新的挑战和信任假设，需要谨慎应对。尽管 SWQoS 旨在优先处理由有质押验证者发送的交易，但目前没有任何机制强制执行这种优先级。严肃的验证者经常会覆盖默认设置，甚至可能阻止某些参与者。这突显了验证者和 RPC 节点之间关系中对信任和透明度的需求，以确保 SWQoS 的公平和有效使用。无论如何，其实施标志着 Solana 作为一个高效和有弹性的网络在演进过程中迈出了重要一步。

在本文中，我们探讨了 SWQoS 及其如何影响交易处理。我们还涵盖了重要的区别，比如 SWQoS 和优先费用之间的差异。此外，我们讨论了验证者和 LSTs 的兴起、潜在的进入障碍以及涉及的信任假设，为未来的讨论提供了一个关键的起点。理解所有这些元素对于有效利用 SWQoS 和整体改善 Solana 至关重要。

如果你已经读到这里，谢谢你，匿名朋友！请务必在下方输入你的电子邮件地址，这样你就永远不会错过关于 Solana 最新动态的更新。准备深入探索了吗？立即浏览[ Helius 博客上的最新文章](https://www.helius.dev/blog)，继续你的 Solana 之旅。

## 额外资源

- [04-30-22 Solana 主网 Beta 中断报告和缓解措施](https://solana.com/news/04-30-22-solana-mainnet-beta-outage-report-mitigation)
- [Solana 上的质押加权服务质量指南](https://solana.com/developers/guides/advanced/stake-weighted-qos)
- [如何设置 Solana 验证者](https://www.helius.dev/blog/how-to-set-up-a-solana-validator)
- [发送智能交易](https://docs.helius.dev/solana-rpc-nodes/sending-transactions-on-solana)
- [Solana 验证者教育 — 质押加权 QoS](https://youtu.be/yeV2i8bfSMs?si=oUvHph6GDETtVSF6)
