# 关于 Solana v1.18 更新你需要知道的一切

感谢 [Rex St. John](https://x.com/rexstjohn) 和 [Mike MacCana](https://x.com/mikemaccana) 审阅本文。

## 引言

Solana 1.18 更新获得绝大多数节点采用是一个重要的里程碑。它带来了一系列旨在提升网络性能、可靠性和效率的改进和新功能。最显著的变化之一是引入了中央调度器。这个新的调度器旨在简化交易处理并确保更准确和高效的优先级计算。例如，对运行时环境和程序部署的其他改进有助于在网络负载高峰期也能提供更可靠的性能。

本文探讨了 1.18 版本带来的更新和改进。我们将探讨这些变化背后的动机、这些新功能的具体细节，以及它们对改善网络的预期影响。无论您是验证者运营商、开发者还是普通的 Solana 用户，这篇关于 1.18 更新的全面概述都将为您提供理解和利用这些新改进的必要信息。

我们首先需要讨论 Anza，这家推动这些变革的新成立开发公司，以及它在 Solana 持续发展中的角色。

## 什么是 Anza?

[Anza](https://www.anza.xyz/) 是一家由前 [Solana Labs](https://solanalabs.com/) 高管和核心工程师创建的新软件开发公司。它的成立代表了加强 Solana 生态系统的战略举措，旨在提高其可靠性、去中心化和网络强度。Anza 的成立目的是通过开发关键基础设施、为核心协议做出贡献以及促进新工具的创新来增强 Solana 生态系统。

创始团队包括 [Jeff Washington](https://twitter.com/JWashAnza)、[Stephen Akridge](https://twitter.com/stephenakridge)、[Jed Halfon](https://twitter.com/jed)、Amber Christiansen、Pankaj Garg、Jon Cinque 以及多位来自 Solana Labs 的核心工程师。

Anza 专注于通过创建 [Agave](https://github.com/anza-xyz/agave)（[Solana Labs 验证者客户端](https://github.com/solana-labs/solana)的一个分支）来开发和完善 Solana 的验证者客户端。Anza 的抱负不仅限于开发其验证者客户端，还致力于整个生态系统的改进。这包括开发[代币扩展（Token Extensions）](https://solana.com/solutions/token-extensions)和[定制的 Rust / Clang 工具链](https://github.com/anza-xyz/platform-tools)。通过培养协作和开放的开发方式，Anza 致力于加速和改进 Solana 生态系统。

## 什么是 Agave?

如前一节简要提到的，Agave 是由 Anza 主导的 Solana Labs 验证者客户端的一个分支。在这里，"分支"指的是 Anza 的开发团队从 Solana Labs 代码库中获取现有代码，并开始一条独立于原始代码库的新开发路径。这使得 Anza 能够对 Solana Labs 客户端实施自己的改进、功能和优化。

### 迁移过程

客户端向 Anza 的 GitHub 组织的迁移[于 3 月 1 日开始](https://x.com/anza_xyz/status/1763667506785587248)。最初，Agave 将镜像 Solana Labs 的代码库，以给社区时间适应。在此期间，Anza 将处理关闭拉取请求（PRs）并将相关问题迁移到 Agave 的代码库。Agave 和 Solana Labs 客户端的 1.17 和 1.18 版本在功能上将完全相同。Anza 计划在今年夏天发布 Agave v2.0，届时将归档 Solana Labs 客户端，并建议网络 100% 迁移到新的 Agave 客户端。

Solana Labs 到 Agave 的[迁移过程在他们的 GitHub 上公开追踪](https://github.com/anza-xyz/agave/wiki/Agave-Transition)。

### Agave 运行时

Agave 运行时继承了 [Solana 虚拟机（SVM）](https://squads.so/blog/solana-svm-sealevel-virtual-machine)的基础架构，是执行 [Sealevel 运行时](https://medium.com/solana-labs/sealevel-parallel-processing-thousands-of-smart-contracts-d814b378192)定义的核心功能的支柱。

Solana 协议将运行时定义为处理交易和更新账户数据库状态的关键组件。这一规范已被 Agave 和 [Firedancer](https://www.helius.dev/blog/what-is-firedancer) 客户端采用并进一步完善。SVM 的本质是其能够并行执行所有 Solana 程序并修改账户状态。

银行（bank）的概念对于处理交易和理解 1.18 版本的变化至关重要。[银行（bank）](https://github.com/solana-labs/solana/blob/master/runtime/src/bank.rs)既是一段逻辑，也是特定时间点[账本](https://solana.com/docs/terminology#ledger)状态的表示。它作为一个复杂的控制器，管理账户数据库、监督客户账户跟踪、管理程序执行，并维护 Solana 账本的完整性和进展。银行封装了给定区块中包含的交易产生的状态，作为该时间点账本的快照。

每个银行都配备了执行交易所需的缓存和引用，允许它们从先前的快照或创世区块初始化。在[银行阶段（banking stage）](https://github.com/solana-labs/solana/blob/27eff8408b7223bb3c4ab70523f8a8dca3ca6645/core/src/banking_stage.rs#L322)，即验证者处理交易的阶段，银行用于组装区块并随后验证其完整性。这个生命周期包括加载账户、处理交易、冻结银行以完成状态，最终使其成为根，确保其永久性。

总的来说，Agave 运行时中的交易处理引擎负责加载、编译和执行程序。它使用[即时（Just-In-Time）编译](https://en.wikipedia.org/wiki/Just-in-time_compilation)，缓存已编译的程序以优化执行效率并减少不必要的重新编译。程序在部署前被编译为 [eBPF 格式](https://ebpf.io/what-is-ebpf/)。然后，运行时使用 [rBPF 工具包](https://github.com/solana-labs/rbpf)创建一个 eBPF 虚拟机，该虚拟机将 eBPF 代码即时编译为 x86_64 机器代码指令，充分利用可用硬件。这确保了程序的高效执行。

1.18 更新引入了中央交易调度器，它与 Agave 运行时引入的运营效率深度交织在一起。通过改进交易的编译、执行和通过银行的管理方式，1.18 更新实现了更精简和高效的调度过程。这反过来又导致更快的交易处理时间和增强的吞吐量。新的 Agave 运行时及其客户端作为这些增强功能的基石，因此在深入研究新调度器的复杂性之前，我们必须对其有某种程度的总体了解。

如果你想了解更多关于 Agave 运行时的信息，我建议阅读 [Joe Caulfield](https://twitter.com/realbuffalojoe) 的[文章](https://fluff-ranunculus-275.notion.site/The-Agave-Runtime-d1f8d3608e5d4529b120e09e80b48887)。它提供了大量细节和有用的代码片段。

## 更高效的交易调度器

### 当前实现

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fsolana-banking-stage-and-scheduler.webp&w=3840&q=90" alt="Solana Banking Stage and Scheduler" />

图片来源: [Andrew Fitzgerald](https://twitter.com/apfitzge) 的 [Solana 银行阶段和调度器](https://apfitzge.github.io/posts/solana-scheduler/) 文章

在交易处理流水线中，交易数据包首先通过[入口（ingress）](https://medium.com/@dipan.saha/understanding-ingress-and-egress-in-networking-how-data-flows-in-and-out-of-networks-152e816472fe)进入系统。这些数据包随后在[签名验证阶段（SigVerify）](https://github.com/solana-labs/solana/blob/27eff8408b7223bb3c4ab70523f8a8dca3ca6645/core/src/tpu.rs#L191)进行签名验证。这一步确保每笔交易都是有效的且经过发送者授权的。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fbanking-stages.webp&w=3840&q=90" alt="Solana Banking Stages" />

在签名验证之后，交易被发送到银行阶段。银行阶段有六个线程——两个专门处理来自[交易处理单元（TPU）](https://www.helius.dev/blog/stake-weighted-quality-of-service-everything-you-need-to-know#how-solana-processes-transactions)或 Gossip 的投票交易，四个专注于非投票交易。每个线程都是相互独立的，从共享通道接收数据包。也就是说，签名验证阶段会以数据包批次发送数据包，每个线程会从该共享通道拉取交易并将其存储在本地缓冲区中。

本地缓冲区接收交易，确定其优先级，并相应地对其进行排序。这个队列是动态的，不断更新以反映交易状态和网络需求的实时变化。当交易被添加到队列时，会重新评估其顺序，以确保最高优先级的交易首先准备处理。

这个过程持续进行，这些交易数据包的处理取决于验证者在[领导者调度](https://solana.com/docs/terminology#leader-schedule)中的位置。如果验证者在不久的将来没有被调度为[领导者](https://solana.com/docs/terminology#leader)，它们会将数据包转发给即将到来的领导者并丢弃它们。当验证者接近其预定的领导时段（约 20 个时隙之前），它会继续转发数据包但不再丢弃它们。这样做是为了确保如果其他领导者没有处理这些数据包，它们可以被包含在自己的区块中。当验证者距离成为领导者还有 2 个时隙时，它开始保留数据包——接受它们但不做任何处理，以便在验证者成为领导者时进行处理。

在区块生产期间，每个线程从其本地队列中取出前 128 个交易，尝试获取锁，然后检查、加载、执行、记录和提交交易。如果锁获取失败，交易将稍后重试。让我们详细说明每个步骤：

- **锁定（Lock）**：这一步检查线程可以为哪些交易获取锁。每个交易都会读取和写入一定数量的账户，因此验证者需要确保没有任何冲突
- **检查（Check）**：这一步检查交易是否太旧或是否已经被处理。注意银行有一个状态缓存，用于跟踪最近 150-300 个时段的交易
- **加载（Loads）**：这一步加载执行给定交易所需的账户。这一步还检查付费方是否真的能够支付费用，以及调用的程序是否有效。基本上，这一步加载账户并执行一些初始设置
- **执行（Execute）**：这一步[执行每个交易](https://github.com/solana-labs/solana/blob/cd6f931223181d5a1d47cba64e857785a175a760/core/src/banking_stage.rs#L845)
- **记录（Record）**：执行的交易结果被[发送到工作量证明服务进行哈希](https://github.com/solana-labs/solana/blob/cd6f931223181d5a1d47cba64e857785a175a760/core/src/banking_stage.rs#L787)。这是发送交易签名的地方
- **提交（Commit）**：如果记录步骤成功，交易就会被提交。这一步还将更改传播回账户系统，以便本时段或后续时段的未来交易将看到每个账户的更新视图
- **解锁（Unlock）**：第一步中为每个账户设置的锁被解除

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fmulti-iterator-approach.webp&w=3840&q=90" alt="Multi Iterator Approach" />

银行阶段使用多迭代器方法（multi-iterator approach）来创建这些交易批次。多迭代器是一种编程模式，允许同时以多个序列遍历数据集。可以将其想象为有几个读者在读同一本书，每个人从不同的章节开始，相互协调以确保如果他们对内容的理解可能相互干扰，就不会同时读同一页。在银行阶段，这些"读者"就是迭代器，而"书"就是等待处理的交易集合。多迭代器的目标是高效地筛选交易，将它们分组成可以在没有任何锁冲突的情况下处理的批次。

最初，交易根据优先级被序列化到一个向量中。这为多迭代器提供了一个结构化的序列，以将这些交易分割成非冲突的批次。多迭代器从序列化向量的开始处开始，在交易之间不相互冲突的位置放置迭代器。通过这种方式，它创建了 128 个没有任何读写或写写冲突的交易批次。如果一个交易与当前正在形成的批次发生冲突，它会被跳过并保持未标记状态，允许它被包含在冲突不再存在的后续批次中。这个迭代过程会随着交易继续处理而动态调整。

在成功形成批次后，交易被执行，如果成功，它们会被记录在工作量证明服务中并广播到网络。

### 当前实现的问题

当前的实现在几个方面可能会影响性能，导致交易处理的潜在瓶颈和不一致的优先级排序。这些挑战主要源于银行阶段的架构和系统内交易处理的性质。

一个基本问题是，_处理非投票交易的四个独立线程在各自的线程内对交易优先级有自己的看法_。这种差异可能导致交易排序的抖动或不一致。当所有高优先级交易发生冲突时，这些差异会变得更加明显。由于数据包基本上是由每个线程从签名验证的共享通道中随机拉取的，每个线程都会有一个随机的交易集合。在竞争事件期间，比如热门 NFT 铸造，很可能许多高优先级交易会出现在多个银行阶段线程中。这是有问题的，因为它可能导致线程间的锁定冲突。这些线程使用不同的优先级集合，可能会相互竞争处理这些高优先级交易，无意中由于未成功的锁定尝试而导致处理时间的浪费。

可以将银行阶段想象成一个管弦乐队，每个线程就像不同的乐器组——弦乐、铜管、木管和打击乐。理想情况下，指挥应该协调这些乐器组以确保和谐的演奏。然而，当前的系统就像一个没有指挥的管弦乐队试图演奏一首复杂的乐曲。每个乐器组都在演奏自己的曲调，经常与其他组发生冲突。高优先级交易就像所有乐器组试图同时演奏的独奏部分，造成混乱。这种缺乏协调突显了 Solana 交易处理需要一个中央"指挥"来确保效率和和谐，就像指挥带领管弦乐队一样。

### 新的交易调度器

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fbanking-stage-central-scheduler.webp&w=3840&q=90" alt="Banking Stage Central Scheduler" />

1.18 更新引入了一个中央调度线程，取代了之前由四个独立银行线程各自管理交易优先级和处理的模型。在这个修订后的结构中，中央调度器是从签名验证阶段接收交易的唯一接收者。它构建优先级队列并部署依赖图（dependency graph）来管理交易的优先级排序和处理。

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Ftransaction-dependency-graph.webp&w=3840&q=90" alt="Transaction Dependency Graph" />

这个依赖图被称为[优先级图（prio-graph）](https://crates.io/crates/prio-graph)。它是一个[有向无环图（directed acyclic graph）](https://en.wikipedia.org/wiki/Directed_acyclic_graph)，在添加新交易时会[延迟求值（lazily evaluated）](https://en.wikipedia.org/wiki/Lazy_evaluation)。交易被插入图中以创建执行链，然后按时间优先级顺序弹出。在处理冲突交易时，先插入的交易总是具有更高的优先级。在上面的例子中，我们有从 A 到 H 的交易。注意交易 A 和 E 在各自的执行链中具有最高优先级且不存在冲突。调度器从左向右移动，分批处理这些交易：

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Ftransaction-process.webp&w=3840&q=90" alt="Transaction Process" />

交易 A 和 E 作为第一批被处理；然后是 B 和 F；接着是 C、D、G；最后一批是 H。如你所见，最高优先级的交易位于图的顶部（即最左侧）。当调度器按降序检查交易时，它会识别冲突。如果一个交易与更高优先级的交易发生冲突，就会在图中创建一条边来表示这种依赖关系（例如，C 和 D 与 B 发生冲突）。

新的调度器模型解决了多迭代器方法固有的几个关键问题：

- **优先级处理的一致性**：通过集中交易的接收和调度，新系统确保所有交易都按照一致的优先级顺序处理。这消除了此前由多个线程对交易优先级有不同看法而导致的抖动
- **减少处理延迟**：优先级图确保准备执行的批次很可能在没有锁冲突的情况下成功，从而简化处理时间并减少因锁竞争可能造成的延迟。注意"很可能成功"这个措辞——严格来说，优先级图创建的批次并非不可能出现锁失败，因为它们可能与投票线程发生冲突，尽管这是一个非常罕见的边缘情况
- **可扩展性和灵活性**：这种新的调度器设计允许增加线程数量，而不会出现之前那样的锁冲突问题。这要归功于对锁的集中视图以及对工作线程之间更受控制的交易分配

  1.18 中引入的中央调度器预计将显著改善交易处理，降低与之前系统相关的复杂性和开销。这可能会带来更快的交易处理时间、更高的吞吐量和更稳定的网络。由于 1.18 发布延迟，调度器自推出以来已有所改进。例如，[交易的预编译验证已移至工作线程以提高效率](https://github.com/anza-xyz/agave/pull/1535)。此外，与旧调度器相比，CU 限制现在更加合理，估计/实际比率更低。[新调度器现在可以使用 CU 来限制调度工作队列，防止由于账户冲突而导致过多工作排队](https://github.com/anza-xyz/agave/pull/1220)。

请注意，[中央调度器默认不启用，启动验证节点时必须使用新的 **--block-production-method central-scheduler** 标志来启用](https://github.com/solana-labs/solana/pull/33890)。目前它仅为可选功能，但在未来的版本中将成为默认调度器。另外请注意，可以使用 **--block-production-method thread-local-multi-iterator** 标志启用旧调度器（这是默认启用的，但请不要在未来的版本中这样做 — 中央调度器效率更高，并解决了旧调度器存在的问题）。

## 更有效的优先级计算

1.18 还改进了交易优先级的确定方式，使这个过程在资源使用和成本回收方面更加公平和高效。此前，交易优先级主要基于计算预算优先级，有时会导致计算单元定价不理想。这是因为优先级排序没有充分考虑收取的基础费用，导致资源可能定价过低并影响网络的运营效率。

[新方法](https://github.com/solana-labs/solana/pull/34888)调整了交易优先级计算，使用公式 **优先级 = 费用 / (成本 + 1)** 来考虑交易费用和相关成本。这里，费用（Fees）代表与给定交易相关的交易费用，而成本（Cost）代表由 Solana 成本模型确定的计算和资源消耗。在分母中加入"1"是为了防止除以零的安全措施。

我们可以进一步分解公式，使**费用**和**成本**更加明确：

<img src="https://www.helius.dev/_next/image?url=https%3A%2F%2Fwww.helius.dev%2Fapi%2Fmedia%2Ffile%2Fsolana-priority-fee-calculation.jpg&w=3840&q=90" alt="Solana Priority Fee Calculation" />

现在交易成本的计算更加全面，[考虑了所有相关的计算和操作成本](https://github.com/apfitzge/agave/blob/c02021173db45aa9bc5ce64af0ee50097318913a/cost-model/src/cost_model.rs#L34-L48)。这确保了优先级计算能够反映交易的真实资源消耗。这意味着如果开发者和用户请求较少的计算单元，他们将获得更高的优先级。这也意味着简单的转账交易，即使没有优先费用，在队列中也会有一定的优先级。

## 改进程序部署

1.18 在[部署可靠性](https://github.com/anza-xyz/agave/pull/387)和[执行效率](https://github.com/anza-xyz/agave/pull/392)方面显著改进了程序部署。

新的更新解决了一个问题：在一个 epoch 的最后一个 slot 部署的程序无法正确应用为下一个 epoch 计划的运行时环境变更。因此，在这个过渡期间部署的程序会错误地使用旧的运行时环境。1.18 调整了部署流程，确保在 epoch 末部署的任何程序的运行时环境都与即将到来的 epoch 的环境保持一致。

1.18 还通过[在 CLI 程序部署命令中添加 **--with-compute-unit-price** 标志](https://github.com/anza-xyz/agave/pull/392)解决了无法在部署交易中设置计算单元价格或限制的问题。这个标志可以与 **solana program deploy** 和 **solana program write-buffer** 命令一起使用。计算单元限制是通过模拟每种类型的部署交易并将其设置为消耗的计算单元数来确定的。

[另一个重要的改进涉及大型程序部署中区块哈希的处理方式](https://github.com/anza-xyz/agave/pull/1074)。在 1.18 之前，使用 `sign_all_messages_and_send` 发送的交易被限制在每秒 100 笔交易(TPS)。对于较大的程序，部署交易的数量可能达到数千笔。这意味着交易可能会延迟，并且由于许多交易会延迟超过 10 秒，存在使用过期区块哈希的风险。1.18 将使用最新区块哈希签署部署交易的时间推迟到限流延迟之后。区块哈希现在每 5 秒刷新一次，因此超过 500 笔交易的部署将受益于使用更新的区块哈希。

此外，[1.18 改进了网络处理程序部署和验证交易的方式](https://github.com/anza-xyz/agave/pull/419)。此前，由于识别账户状态时出现错误，一些程序被错误地标记为 **FailedVerification**。这可能会错误标记那些实际上并未失败任何检查的程序。如果这些程序不应该处于活动状态，现在会被正确识别为 **Closed**。这一改变确保只有存在问题的程序才会被标记为需要重新检查，并有助于防止不必要的重新验证。

更新程序状态的流程也得到了改进。程序现在可以在部署的同一时隙内从 **Closed** 状态转换为活动状态。这意味着程序变得可操作的速度更快、更可靠，这在高需求时期尤为重要。但是，需要注意的是，这项改进仍然受制于一个时隙的取消/重新/部署冷却时间和一个时隙的可见性延迟。因此，虽然这些调整有助于更有效地管理网络负载并防止某些类型的拥塞，但它们并没有显著改变 dApp 开发者的工作流程。

## "拥塞补丁" — 更好地处理拥塞

[测试网版本 1.18.11](https://github.com/anza-xyz/agave/releases/tag/v1.18.11)被称为["拥塞补丁"](https://twitter.com/rexstjohn/status/1778614349709677004)，提出了解决 Solana 最近拥塞问题的变更。需要注意的是，这个版本并不是 1.18 特有的，它已经[向后移植到 1.17.31](https://github.com/anza-xyz/agave/releases/tag/v1.17.31)。不过，我们有必要讨论一下这个问题。

最大的变化是在[基于质押加权的服务质量(SWQoS)](https://www.helius.dev/blog/stake-weighted-quality-of-service-everything-you-need-to-know)中，[QUIC](https://www.helius.dev/blog/all-you-need-to-know-about-solana-and-quic#how-solana-implements-quic)现在[将超低质押的节点视为无质押节点](https://github.com/anza-xyz/agave/pull/701)。这是为了解决质押非常少的节点可能滥用系统获取不成比例带宽的问题。此外，当前的指标无法显示来自有质押节点和无质押节点的数据包发送和限制的比例。因此，[添加了这些指标](https://github.com/anza-xyz/agave/pull/614)以提高可见性。[数据包块的处理方式也得到了优化](https://github.com/anza-xyz/agave/pull/735)，通过用 [**smallvec**](https://docs.rs/smallvec/latest/smallvec/) 替换 **vec** 实例来节省每个数据包的分配。这是可行的，因为流是按数据包大小的，预期数量很少。

在银行阶段，之前所有数据包都会转发到下一个节点。然而，**1.18 改变了这一点，[现在只转发来自有质押节点的数据包](https://github.com/anza-xyz/agave/pull/697)**。这个更新使得有质押连接在计算优先级和转发交易时变得比以往任何时候都更加重要。

## 改进文档

1.18 更新还[显著改进了官方 Solana 文档的翻译支持](https://github.com/solana-labs/solana/pull/35169)，以确保全球用户能更好地访问。更新包括升级 [Crowdin](https://crowdin.com/) CLI 和配置（简化了跨语言文档的同步），并通过 [Docusaurus](https://docusaurus.io/) 引入新的 **serve** 命令以实现更好的本地测试。文档还改进了静态内容的处理方式，通过将 PDF 文件直接链接到 GitHub blob 来避免翻译构建中相对路径的问题。

对于开发者来说，[更新后的 README](https://github.com/solana-labs/solana/pull/35169/files#diff-0b5ca119d2be595aa307d34512d9679e49186307ef94201e4b3dfa079aa89938) 明确了翻译贡献的流程，包括如何处理常见问题，如必要的环境变量和典型的构建错误。这与持续集成流程的改进相辅相成，现在只在稳定版本构建中包含翻译。这确保只有经过审核的稳定文档才能到达最终用户。这些变更旨在简化贡献流程，提高官方文档质量，并让所有用户都能获取可靠准确的信息。

## 总结

在 Anza 的推动下，1.18 版本显著改进了交易处理、优先级计算、程序部署、官方文档和整体网络性能。通过引入中央调度器以及针对近期拥塞问题的各种修复，Solana 现在能够更好地处理峰值负载，确保高效可靠的网络运行。Solana 是可扩展区块链最有希望的选择，这次更新再次证实了它的潜力。

如果你已经读到这里，感谢你的关注！请务必在下方输入你的电子邮箱地址，这样你就不会错过任何关于 Solana 最新动态的更新。准备深入了解更多？立即访问 [Helius 博客](https://www.helius.dev/blog)探索最新文章，继续你的 Solana 之旅。

## 其他资源

- [Agave 客户端](https://github.com/anza-xyz/agave)
- [Agave 过渡计划](https://github.com/anza-xyz/agave/wiki/Agave-Transition)
- [Anza](https://www.anza.xyz/)
- [Prio-graph Crate](https://crates.io/crates/prio-graph/0.1.0)
- [Solana 银行阶段和调度器](https://apfitzge.github.io/posts/solana-scheduler/)
- [聚焦：Solana 的调度器](https://youtu.be/R7hq8ampBio?si=zVpJ0HaQzaiP3pYj)
