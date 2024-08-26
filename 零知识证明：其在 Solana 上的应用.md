# **零知识证明：其在 Solana 上的应用**

特别感谢 [Matt](https://x.com/__lostin__)、[Porter](https://x.com/portport255)、[Nick](https://x.com/nick_pennie)、[Swen](https://x.com/swen_sjn) 以及 [bl0ckpain](https://x.com/bl0ckpain) 对本系列文章的审阅。

**介绍**

这篇文章是零知识证明入门系列的第二篇。在阅读本文之前，我强烈建议先阅读《零知识证明：基础介绍》，因为它提供了本文分析所需的基础理论、数学和密码学背景。本文假设读者已具备相关知识，因此，如果您对零知识证明不太熟悉，我建议您在阅读本文后再回去阅读那篇文章。

本文的目标是让读者具备必要的知识，能够参与 Solana 上零知识证明的讨论，甚至可能引导您走上开发新的、创新的零知识原语的道路。

现在，凭借这些新的知识，我们可以终于问到：什么是零知识证明？它们如何在 Solana 上应用？

那么，什么是零知识证明呢？

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66a3ec2f34f1d32f99252ecd_65cbcbd9e8cd50601750bd2b_secretsolution-banner.jpeg" loading="lazy" alt="">

现在，我们终于掌握了必要的理论、数学和密码学知识，可以适时地提出一个问题：什么是零知识证明？

零知识证明是一种密码学过程，其中一方能够向另一方证明某个陈述是真实的，而无需透露除了该陈述真实之外的任何附加信息。这些证明必须在统计上是可靠的、完整的，并且不会泄露信息。

在零知识证明中，有两种类型的陈述需要证明。总体而言：

* 关于事实的陈述（例如，这个特定的图具有三色可染性）
* 关于知识的陈述（例如，我知c道 N 的因式分解）

第一类陈述最终涉及宇宙的一些固有属性——即某些根本真实的东西，比如 1 + 1 = 2。第二类陈述被称为知识证明。这意味着它不仅仅证明某件事是正确的，还依赖于证明者所掌握的知识。

如前所述，为了证明这些陈述，我们的证明可以是交互式的或非交互式的。在交互式零知识证明中，证明者和验证者会通过一系列回合互动，直到验证者毫无疑问地被说服。<a href="https://z.cash/" target="_blank">Zcash</a>使用非交互式零知识证明允许用户进行匿名交易。

相比之下，在非交互式零知识证明中，证明会在离线状态下完成，证明者和验证者之间没有直接沟通。在这种情况下，证明者生成一个包含所有必要信息的证明，验证者可以独立验证该证明，无需进一步的互动。<a href="https://filecoin.io/" target="_blank">Filecoin</a>使用非交互式零知识证明来证明用户已存储数据，而无需透露数据本身。

### **zk-SNARKs 和电路**

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66a3c3209015820b3dc0f7e2_66a3c221ac63ab22f110a957_SNARKs.jpeg" loading="lazy" alt="">
<center><small>改编自<a href="https://midnight.network/blog/zero-knowledge-demystified" target="_blank"><em> Mauricio Magaldi 和 Olga Hryniuk 的文章 零知识揭秘 - 这不是魔法，而是技术。</em> </a></small></center>

zk-SNARK 是“简洁的非交互式知识论证”（Succinct Non-interactive ARgument of Knowledge）的缩写。这种零知识证明特别高效且紧凑，或称“简洁”。一个证明被认为是简洁的，如果证明的大小和验证所需的时间都比需要验证的计算量增长得更慢。因此，如果我们希望获得一个简洁的证明，就不能让验证者在每轮哈希中做一些工作，否则验证时间将与计算量成正比。简洁的证明之所以成为可能，要归功于多项式和 Fiat-Shamir 启发式方法。

无论要证明的陈述有多复杂，zk-SNARKs 都能保持证明的体积小且验证速度相对较快。这使得 zk-SNARKs 对于汇总（rollups）非常有吸引力；在这里，我们有一种方法可以证明给定 L2 的所有交易都是有效的，并且证明足够小，可以在 L1 上验证。像 <a href="https://minaprotocol.com/" target="_blank"> Mina </a>这样的协议更进一步，使用递归证明，这使得它们能够通过一个固定大小的证明来验证链的整个历史。
<a href="https://minaprotocol.com/blog/meet-pickles-snark-enabling-smart-contracts-on-coda-protocol" target="_blank"> Pickles 是 Mina 的新证明系统及其相关工具包</a>，它是第一个无需可信设置就能实现递归组合的 zk-SNARK。需要注意的是，所谓递归组合，我们指的是电路。

电路描述了你想要证明的计算。它们是一系列数学运算，接受一些输入并生成输出。在零知识证明中，电路用于表示正确执行的计算，而无需透露计算的输入。一般的工作流程如下：

* **编写和编译电路** — 我们需要编写并编译电路。这可以像在素数 p = 7 所定义的有限域上创建一个算术电路，进行计算 x * y = z 一样简单。这里的一般思路是将需要证明的计算表示为变量之间的一组约束条件，将这些约束条件简化为多项式方程，并编写代码，将所有值映射到一个新的代数结构（i.e., [homomorphism](https://en.wikipedia.org/wiki/Homomorphism)）。编译过程会生成多个工件，包括电路定义的一组约束条件以及在后续步骤中使用的脚本或二进制文件。
* **可信设置仪式** — 根据所使用的 zk-SNARK 类型，可能需要运行一个仪式来生成证明密钥和验证密钥。
* **执行电路** — 电路必须使用编译过程中生成的脚本或二进制文件来执行，就像执行某种类型的程序一样。用户输入公共和私有输入，然后计算所有中间变量和输出变量的值。见证（也称为轨迹）是所有计算步骤的记录。
* **生成证明** — 在第二步中获得证明密钥，并在第三步中获得见证后，证明者可以生成一个零知识证明，以证明电路中定义的所有约束条件都成立，同时只揭示输出值。该证明会被发送给验证者。
* **验证证明** — 验证者使用提交的证明和验证密钥来验证该证明对于其公共输出是否正确。

  某些类型的 zk-SNARKs（如 <a href="https://www.zeroknowledgeblog.com/index.php/groth16" target="_blank">Groth16</a>）需要为每个电路进行可信设置。这可能会成为一个障碍，因为每次创建新程序时都需要运行一个新的仪式。其他 zk-SNARKs（如 PlonK）只需进行一次通用的可信设置，简化了整个过程。而其他类型的零知识证明（如 zk-STARKs）则完全消除了可信设置的需求。

### **zk-STARKs**

zk-STARK 是“可扩展透明知识论证”（Scalable Transparent ARgument of Knowledge）的缩写。zk-STARKs 由 <a href="https://starkware.co/" target="_blank">StarkWare</a> 发明，并在<a href="https://starkware.co/wp-content/uploads/2022/05/STARK-paper.pdf" target="_blank">2018 年的论文中首次提出</a> ，作为 zk-SNARKs 的替代方案。它们基本上允许区块链将计算转移到一个离链的 STARK 证明者，并使用链上的 STARK 验证器来验证这些计算的完整性。

zk-STARKs 被认为是零知识的，因为离链证明者使用的输入不会暴露给区块链，从而保持用户的隐私。zk-STARKs 具有可扩展性，因为将计算移到链外大大降低了 L1 验证成本。它们的证明也呈线性扩展，而 zk-SNARKs 仅呈准线性扩展。此外，zk-STARKs 不依赖于复杂的可信设置仪式，这些仪式可能受到“有毒废料”的影响——如果仪式没有正确进行，可能会接受无效证明。相反，zk-STARKs 使用公开可验证的随机性来设置证明者和验证者之间的交互。而且，这些证明只能由实际执行计算的离链证明者生成，同时需要所需的辅助输入。

zk-STARKs 通过提供可扩展的透明知识论证，解决了 zk-SNARKs 的局限性。它们还具有更简单的加密假设，完全避免了椭圆曲线等元素的需求。相反，它们纯粹依赖于哈希和<a href="https://en.wikipedia.org/wiki/Information_theory" target="_blank">信息理论</a>，使它们具有<a href="https://en.wikipedia.org/wiki/Post-quantum_cryptography" target="_blank">量子抗性</a>。然而，证明的大小通常在几百千字节的范围内，这可能限制了它们在带宽或存储有限的环境（如区块链）中的可行性。

需要注意的是，一些更复杂的证明设置和 zkVM 会将 zk-STARKs 递归地证明到 zk-SNARK 中，只用于最终验证步骤。

## **ZK 压缩**

### **状态增长问题**

Solana 面临的一个紧迫问题是<a href="https://x.com/aeyakovenko/status/1796569211273445619" target="_blank">状态增长问题</a>。为了使这个问题有上下文，Solana 的状态存储在完整节点的 <a href="https://solana.com/docs/core/accounts" target="_blank">Accounts DB</a> 磁盘上。这是一个键值存储，其中每个数据库条目称为<a href="https://www.helius.dev/blog/the-solana-programming-model-an-introduction-to-developing-on-solana#what-are-accounts" target="_blank">账户</a>。账户有 32 字节的地址，账户可以存储的数据量在 0 到 10 MB 之间。目前，无论是一个 10 MB 的账户还是一千个 10 KB 的账户，存储 10 MB 的数据的成本大约为 70 SOL。根据 Toly 关于状态增长问题的帖子，每天大约会有一百万个新账户被添加到链上，这使得总状态超过 5 亿个账户。随着 Solana 的持续增长，这将带来几个挑战，即无限的快照大小、PCI 带宽限制、账户索引以及昂贵的内存和磁盘管理。

当前的完整快照大小大约为 70 GB，这在现有硬件下是可以管理的。然而，持续增长将不可避免地导致状态管理效率低下和潜在瓶颈。随着快照大小的增加，硬件故障后<a href="https://nordvpn.com/cybersecurity/glossary/cold-boot/" target="_blank">冷启动</a>新系统所需的时间显著增加，这在网络重启时可能会造成不利影响。

<a href="https://en.wikipedia.org/wiki/Peripheral_Component_Interconnect" target="_blank">外围组件互连（PCI）</a>带宽指的是 CPU 与外围设备之间的数据传输速率，例如显卡、网络卡和存储设备。<a href="https://en.wikipedia.org/wiki/PCI_Express" target="_blank">PCI Express (PCIe)</a>是一个高速接口标准，旨在取代较旧的 PCI 标准并提供更高的数据传输速率。最新的 PCI 带宽可以达到 1 TB，或 <a href="https://www.rambus.com/blogs/pcie-6/" target="_blank">128 GB/s</a>。尽管这听起来很多，但在 Solana 的上下文中并不算多。如果一个交易读取或写入 128 MB，那么 128 GB/s 的 PCI 带宽将把 Solana 限制在每秒 1000 个交易（TPS）。然而，大多数交易访问的是已经加载并缓存到验证者 RAM 中的最近内存。尽管如此，高效的状态内存对于确保 Solana 扩展时能够保持高吞吐量至关重要，否则带宽可能很快成为限制因素。

每个验证者需要维护所有现有账户的索引。这是因为创建新账户需要证明该账户不存在。考虑到总状态超过 5 亿个账户，即使是最小的索引（即每个条目 32 字节的键和 32 字节的数据哈希）也需要大约 32 GB 的 RAM。这种状态存储成本昂贵，需要仔细管理以防止性能下降。随着 Solana 状态的持续增长，使用快速、昂贵的内存（即 RAM）与较慢、便宜的内存（即磁盘）的区别变得至关重要。

### **交易和状态增长**

每笔 Solana 交易都需要指定其读取和写入的所有账户。交易目前被限制为 1232 字节，并且必须包含以下内容：

* 头部信息 (3 字节)
* 签名 (每个 64 字节)
* 账户地址 (每个 32 字节)
* 指令数据 (大小可变)
* 最近的区块哈希 (32 字节)

当一笔交易被执行时，发生以下过程：

* **合理性检查** — 仅最近的交易有效，并对交易进行重复检查、结构验证、费用和签名检查。
* **程序加载** — 根据程序地址加载程序字节码，并实例化 Solana 虚拟机 (SVM)。
* **账户加载** — 检查并从存储中加载交易所引用的所有账户，将其传递给 SVM。
* **执行** — 执行程序字节码。
* **同步** — 所有被修改的账户将被同步回存储。

随着状态的持续增长，这一生命周期带来了几个挑战。首先，链上状态是昂贵的，将更多账户存储在磁盘上会导致更大的快照和索引。此外，并非所有账户都被频繁访问，因此在这些账户上持续消耗资源是低效的。

### **通过 ZK 压缩简化状态管理**

与其将所有账户存储在磁盘上并在需要时读取，不如让交易将账户数据作为交易负载的一部分。我们可以使用默克尔树来确保提交交易的用户提供正确的状态。默克尔证明是一种数据提交方式。通过这种方式，可以通过将证明与提交进行验证来确认已传递正确的状态，并且用户没有对所提供的状态撒谎。

虽然安全，但这些证明可能非常大。例如，如果一棵树包含 10 万个账户，证明大小将为 544 字节。为多个账户提供证明可能会很快超过 1232 字节的交易大小限制。幸运的是，我们可以通过使用更高效的证明系统来规避这一问题。使用恒定证明大小的提交方式，如 KZG 或 Pedersen 提交，可以减小证明大小，使得在交易大小限制内包含证明更加可行。

ZK 压缩就是解决默克尔证明大小问题的机制——它是一种通过利用 Solana <a href="https://solana.com/docs/terminology#ledger" target="_blank"><em>账本</em></a>，在不增加链上存储成本的情况下，证明某些计算已正确完成的方法。

## **什么是 ZK 压缩？**

ZK 压缩是一种新型原语，允许开发者压缩链上状态，从而在保持安全性、性能和可组合性的同时，大幅降低状态成本。例如，当前创建 100 个代币账户的费用大约为 0.2 SOL，而使用 ZK 压缩后，成本减少 5000 倍，仅为约 0.00004 SOL。

ZK 压缩利用零知识证明来验证状态转换，而无需暴露底层数据。它通过将多个账户分组为单个可验证的默克尔根存储在链上，而将底层数据存储在账本上来实现这一点。有效性证明是简洁的零知识证明，用于证明 n 个账户作为 m 个状态树中的叶子节点存在，同时保持恒定的 128 字节证明大小。这些证明是在链下生成并在链上验证的，从而减少了 Solana 的整体计算负担。ZK 压缩使用 <a href="https://docs.rs/groth16-solana/latest/groth16_solana/" target="_blank">Groth16</a>，这是一种著名的<a href="https://en.wikipedia.org/wiki/Pairing-based_cryptography" target="_blank">基于配对的</a> zk-SNARK，作为其证明系统。

然而，这些账户并不是常规的 Solana 账户，而是压缩账户。

### **压缩账户模型**

ZK压缩状态存储在压缩账户中。这些账户与常规的Solana账户类似，但在效率和可扩展性方面有几个关键区别：

* **哈希标识** — 每个压缩账户都可以通过其哈希进行识别。
* **写入操作导致哈希变化** — 任何对压缩账户的写操作都会更改其哈希值。
* **可选地址** — 地址可以选择性地设置为压缩账户的永久唯一ID。这在某些使用场景中非常有用，例如NFT。这个字段是可选的，以避免计算开销，因为压缩账户可以通过其哈希引用。
* **稀疏状态树** — 所有压缩账户都存储在Merkle树中，只有树的状态根（即Merkle根）存储在链上的账户空间中。更具体地说，<a href="https://github.com/Lightprotocol/light-protocol/blob/main/merkle-tree/concurrent/tests/tests.rs" target="_blank">状态树是基于Poseidon哈希的并发Merkle树。</a>

  <img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66a3c406550ff30970c897f0_66a3c3e1b47ec73deac9d26e_compressed-pda.jpeg" loading="lazy" alt="">

<center><small> 摘自 <a href="https://www.zkcompression.com/learn/core-concepts/compressed-account-model" target="_blank"><em> ZK Compression 文档的压缩账户模型部分</em> </a></small></center>

压缩程序派生地址（PDA）可以通过其唯一且持久的地址进行识别。它们的布局与常规 PDA 账户类似，包含数据、Lamports、所有者和地址字段。然而，与常规 PDA 不同的是，数据字段内置了 AccountData 结构，该结构包括 Discriminator、Data 和 DataHash 字段。

### **节点**

不同类型的节点在支持 ZK 压缩中起着关键作用。任何人都可以运行 Photon RPC 节点、Prover 节点或 Light Forester 节点，以连接到 Devnet 和 Mainnet-Beta。对于本地开发，<a href="https://github.com/Lightprotocol/light-protocol/tree/main/cli" target="_blank">ZK 压缩 CLI </a>的 `test-validator` 命令会启动一个包含所有相关节点（即 Photon RPC 和 Prover），以及系统程序、账户和运行时功能的单节点 Solana 集群。

<a href="https://www.zkcompression.com/node-operators/run-a-node#photon-rpc-node" target="_blank">Photon RPC 节点</a>负责索引压缩程序。这使得客户端能够读取并构建与压缩状态交互的交易。规范的压缩索引器名为 <a href="https://github.com/helius-labs/photon" target="_blank">Photon</a>，由 Helius 提供。这种类型的节点可以通过最少的设置在本地运行，并需要指向现有的 RPC。

<a href="https://www.zkcompression.com/node-operators/run-a-node#prover-node" target="_blank">Prover 节点</a>用于生成状态包含的有效性证明。可以使用<a href="https://www.zkcompression.com/developers/json-rpc-methods" target="_blank">ZK 压缩 RPC API 规范</a>的 <a href="https://www.zkcompression.com/developers/json-rpc-methods#getvalidityproof" target="_blank"><strong>getValidityProof </strong>endpoint</a>来获取证明。Prover 节点可以作为独立节点运行，也可以与其他 RPC 捆绑运行。需要注意的是，规范的 Photon RPC 实现中包括一个 Prover 节点。

<a href="https://www.zkcompression.com/node-operators/run-a-node#light-forester-node" target="_blank">Light Forester 节点</a>负责管理共享和程序拥有的状态树的创建、轮替和更新。这适用于那些希望由 Light Forester 节点网络服务的开发人员，他们可以选择拥有自己的程序拥有的状态树。

### **信任假设**

任何人都可以运行上述节点之一，并存储生成证明和提交交易所需的原始数据。这引入了一种信任假设，影响压缩状态的活跃性。具体来说，如果数据丢失或延迟，除非个人存储了数据，否则交易将无法提交。由于只需一个诚实的节点提供数据，并且证明是自验证的，因此问题在于活跃性和潜在的审查，而非安全性。

此外，验证压缩账户的程序目前是可升级的，这引入了另一个信任假设。这是为了能够修改程序以修复任何问题或适应新需求。然而，一旦程序达到稳定和安全的状态，它可以变得不可更改或被冻结。

另一个活跃性信任假设是使用 Forester 节点。这些节点维护状态根的推进，并通过异步清空并推进状态根来管理空白队列。此处，账户哈希被替换为零以使其无效。推进和无效化的分离确保了压缩状态转换的即时终结，同时保持交易在 Solana 尺寸限制内。由于空白队列的大小是恒定的，Forester 节点对于协议的活跃性至关重要。满队列将导致相关状态树的活跃性失败。幸运的是，Forester 节点通过清空队列来防止这种情况发生。然而，人们仍需要运行这些节点以帮助维持协议的完整性和活跃性。没有这些节点，ZK Compression 只能支持大约两千个账户/地址。

### **限制**

即使在不需要隐藏某些信息的情况下，零知识证明仍然会将需要多个计算步骤的问题转化为只需要验证单个证明即可确定计算正确性的问题。而且这些计算不需要局限于特定的叶节点是否属于给定的树——它们可以是任何任意的计算。然而，这会带来一定的代价。

在使用 ZK 压缩之前，请考虑以下几点：

* **更大的交易大小** — ZK 压缩需要 128 字节用于有效性证明，并且需要在链上读写的数据
* **更高的计算单元使用量** — ZK 压缩显著增加了计算单元 (CU) 的使用量，验证有效性证明大约需要 10 万 CU，系统使用大约需要 10 万 CU，每次压缩账户的读写大约需要 6 千 CU
* **每笔交易的状态成本** — 每次写操作都会产生少量网络成本，因为它必须使前一个压缩账户状态无效，并将新压缩状态追加到状态树中。因此，如果某个压缩账户需要进行大量状态更新，其生命周期成本可能会超过其未压缩的等价物

在以下情况下，使用常规账户可能更可取：

* 账户被频繁更新
* 账户的生命周期写操作次数非常多（即，大于 1000 次）
* 账户存储了大量必须在链上交易中访问的数据

### **优势**

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66a3eb119bd072ff4977f5a0_66a3e9309710d78dcbe9f466_image%2520(3).png" loading="lazy" alt="">

ZK 压缩是一种可扩展、安全、高效且灵活的原语，直接解决了 Solana 的状态增长问题，并支持广泛的应用和用例。可以说，其最显著的优势是其状态成本的降低。ZK 压缩通过在更便宜的分类账空间中安全地存储状态，同时使用状态指纹技术将链上存储量保持在最低限度，从而使应用程序能够轻松扩展到数百万用户。以铸造 10,000 个代币账户为例，假设 SOL 的价格为 130 美元，则大约需要花费 2,600 美元。而 ZK 压缩将此成本降低到不到 50 美分。

ZK 压缩还很好地适应了当前的 Solana 规范。例如，压缩账户的结构与普通 Solana 账户几乎相同。它还支持 Solana 的创新特性，如并行处理。也就是说，在同一状态树（即承诺）下访问不同压缩账户的任何两个交易都可以并行执行。此外，ZK 压缩还加强了同步原子组合性。例如，列出 n 个压缩账户和 m 个普通账户的交易是一种完全有效的配置。引用压缩账户的指令可以调用另一个引用普通账户的指令或程序。即使这些账户压缩在不同的状态树下也是如此。如果一个指令失败，则整个交易回滚，并且从一个指令到下一个指令的更改都是可见的。这与 ZK 汇总不同，ZK 汇总不能同步或原子地相互调用，除非它们采取锁定措施。自然，这引发了 ZK 压缩与汇总之间的比较。

### **ZK Compression 不是 Rollup**

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66a3eb119bd072ff4977f5a4_66a3e945ecb6a7a9ccb81678_vitalik-masterclass.jpeg" loading="lazy" alt="">

<center><small> 来源：<a href="https://warpcast.com/vitalik.eth/0xf986f71e" target="_blank"><em> Vitalik 在 Warpcast 上的回复</em> </a></small></center>


ZK Compression 不是一个 Rollup。尽管两者都依赖于相同的技术，但它们的实现方式有所不同。Rollup 分为两种类型：

* **Optimistic Rollups** — 在给定的时间段内，所有交易都被假定为有效，且在此时间框架内使用欺诈证明来证明虚假交易。
* **Zero-Knowledge Rollups** — 交易通过有效性证明即时证明有效或无效。

零知识 Rollup 的整个状态作为单个根节点存在于基础层（即 Ethereum）上。这导致了多个声明，认为 ZK Compression 实际上是一个 Rollup。然而，存在一些关键差异。

考虑一个涉及 500 笔交易的 ZK Rollup 场景。在这种情况下，整个 Rollup 被视为一个单一的电路。所有 500 笔交易一起被验证，生成一个证明，确认状态根节点从 A 变为 B。在此证明经过验证后，管理 L1 和 L2 交互的智能合约相应地更新状态根节点。相比之下，在 ZK Compression 中，每笔交易都生成其自己的证明来验证账户数据的正确性。这些交易由 SVM 本身执行，一旦每个证明被验证，账户会被视为“常规”账户。

如果我们将 ZK Compression 分类为 Rollup，那么这意味着任何存储在 Solana 上的 Merkle 根节点都可以被视为基于有效性的 Rollup。如果我们查看 Solana 上所有压缩的 cNFT 根节点，<a href="https://x.com/ledger_top/status/1804248077735256527" target="_blank">那么大约有 4-5k 个基于有效性的 Rollup，具体取决于我们是否计算了零 mint 的 Merkle 树</a>。

因此，可以清楚地看出，ZK Compression 是一种量身定制的解决方案，适用于 Solana 的架构。它是一种新型的原语，区别于 ZK Rollups，提升了可扩展性和效率，而不涉及 Rollups 的复杂性和分离。

## **ZK 在 Solana 和跨链互操作性的未来**

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66a3eb119bd072ff4977f59b_66a3ea4283f9fdbf4b1f4da6_hey.png" loading="lazy" alt="">

<center><small><figcaption>来源: <a href="https://x.com/aeyakovenko/status/1739485437545296183" target="_blank">https://x.com/aeyakovenko/status/1739485437545296183</a></figcaption></small></center>

### **Solana 上 ZK 技术的现状**

我在 Helius 的第一项写作任务之一是覆盖 <a href="https://www.helius.dev/blog/all-you-need-to-know-about-solanas-v1-16-update" target="_blank">Solana 的 v1.16 更新</a>。我对零知识证明的更好运行时支持感到非常兴奋，并在文章中进行了详细介绍。然而，这些改进的实施被推迟了。我在<a href="https://www.helius.dev/blog/all-you-need-to-know-about-solanas-v1-17-update" target="_blank">v1.17 更新文章</a> 中再次详细介绍了这些改进，但它们再次被推迟。最终，我在<a href="https://www.helius.dev/blog/all-you-need-to-know-about-solanas-v1-18-update" target="_blank">v1.18 更新文章</a>中甚至没有提及这些改进。这自然让我感到失望，<a href="https://x.com/0xNIC0/status/1800457297530961958" target="_blank">其他人也表达了他们的挫败感 </a>。

尽管如此，在 Solana 上仍然有一个正在成长的小型 ZK 开发者社区。最初，<a href="https://lightprotocol.com/" target="_blank">Light Protocol</a> 关注于以 <a href="https://www.helius.dev/blog/privacy-on-solana-with-elusiv-and-light#what-is-the-light-protocol" target="_blank">PSP（Private Solana Programs）形式执行私密程序</a>，然后将焦点转向了 ZK Compression。<a href="https://x.com/privatelp" target="_blank">Dark protocol</a> 也是一个建立在 Solana 上的 <a href="https://www.helius.dev/blog/futarchy-and-governance-prediction-markets-meet-daos-on-solana" target="_blank">futarchic </a>隐私协议，没有中心化团队，通过提案进行贡献。Arcium，前身为 <a href="https://medium.com/elusiv-privacy/sunsetting-elusiv-transitioning-towards-the-future-of-privacy-and-confidentiality-0b078e9bcfac" target="_blank">Elusiv</a>，使用多方计算执行环境（MXEs）来推动其并行化、保密计算网络。<a href="https://github.com/anagrambuild/bonsol" target="_blank">Bonsol</a>是一个零知识“协处理器”，允许开发者在 Solana 上运行任何 <a href="https://github.com/risc0/risc0" target="_blank">risc0 镜像</a>并验证（即可验证的离链计算）。有<a href="https://github.com/anagrambuild/tour-de-zk" target="_blank">关零知识证明的链接</a>和<a href="https://dev.to/zeref101/building-your-first-zk-program-on-solana-2nhn" target="_blank">教程</a>也在流传。

最值得注意的是，<a href="https://docs.solanalabs.com/runtime/zk-token-proof" target="_blank">ZK Token Proof Program</a> 验证了几个适用于 <a href="https://web.getmonero.org/resources/moneropedia/pedersen-commitment.html" target="_blank">Pedersen 承诺</a>和基于 <a href="https://www.rfc-editor.org/rfc/rfc7748#section-4.1" target="_blank">curve25519</a> 的 <a href="https://spl.solana.com/confidential-token/deep-dive/encryption#twisted-elgamal-encryption" target="_blank">Twisted ElGamal 加密</a>的零知识证明。它支持 <a href="https://www.helius.dev/blog/all-you-need-to-know-about-solanas-v1-16-update#confidential-transfers" target="_blank">Confidential Transfers</a>，这些转账使用零知识证明来加密 SPL 代币的余额和交易金额。其目标是保密而非匿名。 <a href="https://en.wikipedia.org/wiki/Homomorphic_encryption" target="_blank">同态加密</a>允许在加密数据上进行计算，而无需解密。为此，Confidential Transfers 使用 Twisted ElGamal 加密对密文进行隐藏的数学操作，并使用 <a href="https://spl.solana.com/confidential-token/deep-dive/zkps#sigma-protocols" target="_blank">Sigma 协议</a>验证这些转账而不泄露敏感信息。只有持有解密密钥的账户持有者才能查看其加密余额。然而，<a href="https://spl.solana.com/confidential-token/deep-dive/overview#global-auditor" target="_blank">Global Auditor System</a> 通过单独的解密密钥允许合规性和审计的选择性读取访问。

ZK Token Proof Program 当前是<a href="https://github.com/anza-xyz/agave/wiki/Feature-Gate-Activation-Schedule" target="_blank">一个被阻塞的功能</a>，原因是<a href="https://github.com/solana-foundation/solana-improvement-documents/pull/153" target="_blank"> SIMD-0153: ZK ElGamal Proof Program</a> 的通过。这项新的 SIMD 旨在弃用现有的 ZK Token Proof 程序，该程序专门为 SPL Token 程序设计，并用一个更通用的零知识证明程序取而代之，该程序不依赖于任何特定应用。该 SIMD 已经在 <a href="https://www.anza.xyz/" target="_blank">Anza</a>和 <a href="https://www.helius.dev/blog/what-is-firedancer" target="_blank">the Firedancer team</a>。

然而，局势开始发生变化。**随着这些改进最终来到 Devnet 和 Mainnet-Beta，Solana 正在成为一个 ZK 巨头**。目前，<a href="https://x.com/swen_sjn/status/1816890875600941131" target="_blank"><strong>已经有 <em>三个</em>ZK 系统调用在Solana上线 </strong></a> 。

### **Poseidon 系统调用**

<a href="https://www.poseidon-hash.info/" target="_blank">Poseidon</a>是一组专门为零知识证明设计的哈希函数系列，用于 Zcash、Mina 和 Light Protocol 等项目。与传统的通用哈希函数（如<a href="https://en.wikipedia.org/wiki/SHA-2" target="_blank"> SHA-256</a>）相比，Poseidon 在零知识证明中具有更高的计算效率。

Poseidon 哈希函数之所以适合零知识证明，主要因为它们：

* 高效执行算术操作
* 生成证明所需的步骤较少（即电路复杂度较低），这是由于其算术友好的设计、优化的 <a href="https://en.wikipedia.org/wiki/S-box" target="_blank">S-box</a> 和较少的轮数
* 使用能够处理任意长度位序列的算法，使其非常通用

以前，在单个事务中计算 Poseidon 哈希非常昂贵。然而，随着 644 纪元的到来以及 <a href="https://github.com/solana-labs/solana/pull/32680" target="_blank">Poseidon syscall</a> 的激活，这一情况发生了变化。Poseidon syscall 是一个系统调用，接受一个二维字节切片作为输入，并计算相应的 Poseidon 哈希作为输出。这一点令人兴奋，因为 ZK Compression 依赖于 Poseidon 哈希来处理其状态树。

Poseidon syscall 使用 <a href="https://hackmd.io/@jpw/bn254" target="_blank">BN254 curve曲线</a> 计算哈希，具有以下参数：

* S-boxes — x5 替代盒
* 输入 — 1 ≤ n ≤ 12
* 宽度 — 2 ≤ t ≤ 13
* 轮次 — 8 个完整轮次和根据 t 变化的部分轮次：[56, 57, 56, 60, 60, 63, 64, 63, 60, 66, 60, 65]

其输出将是编码为 32 字节的 Poseidon 哈希结果，具体的字节序由指定的<a href="https://en.wikipedia.org/wiki/Endianness" target="_blank">字节序</a>规定。

需要注意的是，这里用于 syscall 的特定变体是 Poseidon，其中使用了 x5 S-box 和针对 BN254 曲线定制的参数。<a href="https://crates.io/crates/light-poseidon" target="_blank"><strong>light-poseidon </strong>crate</a> 将用于计算这些哈希。该 crate 本身<a href="https://github.com/Lightprotocol/light-poseidon/blob/main/assets/audit.pdf" target="_blank">经过审计</a>，并与 <a href="https://docs.circom.io/" target="_blank">Circom</a> 兼容。

“alt_bn128 系统调用”
是指 Barreto-Naehrig (BN-128) 椭圆曲线的实现，这是一种支持配对友好的曲线，可用于高效的 zk-SNARKs 证明和计算。这种曲线在各种零知识证明系统中起着重要作用，包括 Groth16，ZK Compression 依赖它来验证状态转换。这个系统调用大大减少了每个证明所需的空间，提供了关键的空间和时间优化，以实现高效的链上证明。

<a href="https://github.com/solana-labs/solana/pull/27961" target="_blank"><strong>`sol_alt_bn128_group_op` </strong>syscalls</a> 系统调用在 alt_bn128 曲线上计算操作，包括 G1 中的点加法（G1 指给定椭圆曲线上的点群）、G1 中的标量乘法和配对操作：

* **输入** ：序列化的点和以大端格式表示的标量
* **操作** ：G1 中的点加法、G1 中的标量乘法、配对（G1 中的一个点和 G2 中的一个点）
* **输出** ：G1 中的点或配对结果，序列化为 256 位整数

<a href="https://github.com/solana-labs/solana/pull/32870" target="_blank"><strong>`sol_alt_bn128_compression` </strong>syscalls</a>调用压缩或解压缩 G1 或 G2 组中的点，并以标准的大端格式返回这些点。

这些系统调用目前已经在测试网中上线，<a href="https://x.com/trentdotsol/status/1794068150150902034" target="_blank">并有望在解决加载程序重新编译阶段中的错误后，在 Devnet 上提供</a>。这些改进正是 <a href="https://github.com/iceomatic/solana-improvement-documents/blob/patch-1/proposals/0129-alt-bn128-simplified-error-code.md" target="_blank">SIMD-0075（Secp256r1 预编译）</a>提案的一部分，该提案旨在简化 alt_bn128 系统调用以及 Poseidon 系统调用的错误代码，确保一致性，并减少因验证者返回不同错误代码而导致的共识失败风险。

alt_bn128 系统调用可以用于常量证明大小的常规向量承诺，例如 KZG 承诺。因此，它们将适用于任何支持配对的曲线，并且不需要 ZK 证明电路。

### **互通性**

Solana 是一个 ZK 链。它是一个高性能的 Layer 1 区块链，具有低费用和对椭圆曲线操作的运行时支持。对零知识证明相关系统调用的实现和支持促进了创新，使得诸如 ZK 压缩等新颖的原语和应用程序能够在 Solana 上构建。

引入 alt_bn128 系统调用缩小了 Solana 与基于 Solidity 的合约之间的组合差距，这些合约依赖于 <a href="https://eips.ethereum.org/EIPS/eip-196" target="_blank">EIP-196</a>、<a href="https://eips.ethereum.org/EIPS/eip-197" target="_blank">EIP-197</a> 和 EIP-198 中规定的椭圆曲线操作的预编译合约。这些操作使得 zk-SNARK 证明能够在以太坊的 Gas 限制内进行验证。因此，依赖这些椭圆曲线操作的 Solidity 合约现在可能更容易过渡到或甚至与 Solana 进行互操作。

SIMD-0075 对互操作性解决方案至关重要。一旦全面实施，这个 SIMD 将使即将推出的 <a href="https://x.com/MrPkar/status/1807466781520646342" target="_blank">blobsream-solana</a> 等项目能够使用链下生成的证明和链上验证来存储 Merkle 承诺。没有这些系统调用，在 Solana 上验证 Groth16 证明将是不可能的。此外，这些系统调用增强了信任最小化的跨链桥接和互操作性，使其他区块链能够与 Solana 无缝且安全地互动。

Toly 说得对——随着 Solana 运行时的所有这些增强，Solana 实际上成为了一个以太坊 L2。很快，将没有任何东西阻止你将所有的 Solana 区块提交到以太坊上的某个数据验证桥接合约中。反之， 也不会有任何东西阻止你将所有以太坊区块提交到 Solana 上的某个数据验证桥接程序中。在零知识证明的加速下，取代过时的跨链桥接，双向互操作性将成为 Solana 光明而萌芽的未来。

## **结论**

零知识证明无疑是由密码学家开发的最强大、最具影响力的原语之一。通过在第一篇文章中审视这一概念的理论、数学和密码学，从基础原理出发，这一点变得显而易见。它的潜在应用无穷无尽，从链上游戏的真正战争迷雾到证明一组交易在 L2 上导致了特定的状态转换。

这两篇文章完全可以扩展到五十页以上，涵盖同态加密的复杂性、在 Circom 中编写电路以及分析不同的承诺方案。然而，这些文章的目标是让读者了解零知识证明的基本原理，以便他们可以将这些新获得的知识应用于 Solana。

Solana 正在成为一个 ZK 巨头。从 ZK 压缩的发布到即将上线的各种系统调用，其重要性不可低估。尽管像 ZK 压缩这样的原语为普通开发者抽象了零知识证明的所有复杂性，但对基本原理的理解对于推动这一话题及其在 Solana 上的发展至关重要。

如果你读到了这里，感谢你，匿名者！请在下方输入你的电子邮件地址，以便随时获取关于 Solana 新动态的更新。准备深入了解吗？立即探索 <a href="https://www.helius.dev/blog" target="_blank"> Helius 博客</a>上的最新文章，继续你的 Solana 之旅。

## **附加资源**

* <a href="https://vitalik.eth.limo/general/2021/01/26/snarks.html" target="_blank"> 这是 Vitalik Buterin 提供的一篇关于 zk-SNARKs 的介绍，解释了其可能性的基本概念。</a>
* <a href="https://eprint.iacr.org/2019/458.pdf" target="_blank"> 了解 Poseidon 哈希函数，这是一种专门为零知识证明系统设计的哈希函数。</a>
* <a href="https://github.com/iceomatic/solana-improvement-documents/blob/patch-1/proposals/0129-alt-bn128-simplified-error-code.md" target="_blank">了解 SIMD-0075 提案，它旨在简化 secp256r1 预编译的错误代码，提升 Solana 的兼容性</a>
* <a href="https://chain.link/education-hub/zk-snarks-vs-zk-starks" target="_blank"> 这篇文章解释了 zk-SNARKs 和 zk-STARKs 之间的主要区别，包括各自的优势和应用场景。</a>
* <a href="https://www.zkcompression.com/" target="_blank">查看关于 ZK 压缩技术的官方文档，了解其在 Solana 上的应用和实现细节。</a>
