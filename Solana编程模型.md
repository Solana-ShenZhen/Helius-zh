# Solana 编程模型：Solana 开发入门

## 这篇文章讲的是什么？

Solana 的去中心化计算方式基于一个简单的原则：所有东西都存储在自己的内存区域中，这个区域称为账户（Account）。Solana 可看作一个全局键/值（key/value）存储系统运作，其中地址（Address）作为相应账户的唯一标识符。账户是 Solana 的支柱，因为它们存储状态；它们持有从程序到代币余额的所有内容。交易（Transactions）用于更新账户并反映状态的变化。

在本文中，我们将探讨 Solana 架构的复杂性。我们首先概述集群（clusters）和状态（state）的概念，然后讨论账户和程序作为 Solana 基础组件的作用。接着，我们将研究交易如何实现账户和程序之间的动态交互。

到本文结束时，您将对 Solana 的编程模型有一个透彻的理解。您将熟悉集群的架构、账户在数据存储中的关键作用以及交易更新账户数据的过程。此外，您还将探索 Solana 独有的特性，如租金系统（rent system）和版本化交易（versioned transactions）。

## 什么是 Solana 集群？

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/654d4507281aa742c8e5ccae_sys-arc.jpg" loading="lazy" alt="">

Solana 架构的核心是[集群](https://solana.com/docs/terminology#cluster)（clusters）——一组验证者共同处理交易并维护单一[账本](https://solana.com/docs/terminology#ledger)（ledger）。Solana 有几个不同的集群，每个集群都有特定的用途：

- **[Localhost](https://solana.com/developers/guides/getstarted/setup-local-development)**：一个本地开发集群，位于默认端口 8899。[Solana 命令行界面（CLI）](https://docs.solanalabs.com/clusters#devnet)带有内置的[测试验证器](https://docs.solanalabs.com/clusters#mainnet-beta)（test validator），可以根据个人开发者的需求进行自定义，没有任何空投或访问速率限制。
- **[Devnet](https://docs.solanalabs.com/clusters#devnet)**：一个无后果（上面的代币没有实际价值）的沙箱（sandbox）环境，用于在 Solana 上进行测试和实验。
- **[Testnet](https://docs.solanalabs.com/clusters#testnet)**：一个 Solana 核心贡献者用来测试新更新和功能的试验场，在这些功能上线主网之前使用。它也是开发者用于进行性能测试的测试环境。
- **[Mainnet Beta](https://docs.solanalabs.com/clusters#mainnet-beta)**：实际发生交易的实时无许可集群。这是“真实”的 Solana，用户、开发者、代币持有者和验证者每天在此进行互动。

每个集群独立运行，完全不互相感知。发送到错误集群的交易将被拒绝，以确保每个操作环境的完整性。

可以将集群想象成一个单一的数据堆。在计算机科学中，堆指的是一个可以动态存储和修改数据的内存区域。然而，需要注意的是，**集群并不是真的使用堆数据结构**。这个类比作为一种概念工具，有助于理解集群由各种内存区域组成，可以根据需要分配和释放内存。将集群理解为一个动态堆是理解网络中数据如何管理、访问和保护的关键。

你也可以把这个单一的数据堆想象成某种数字仓库。在这里，数据就像货架上的盒子，每个盒子都有独特的标签，还有用于移动和更改其内容特定规则。这确保了一个安全有序的系统，其中只有授权的移动或更改才被允许。

智能合约（smart contracts，在 Solana 上称为程序，program）被分配到仓库或堆的自己部分，他们可以管理这些部分。虽然程序可以读取仓库中任何部分的数据，但它们需要某些权限才能更改它们不拥有的部分内容。唯一普遍允许的操作是将 Solana 的本地加密货币 lamports 转移到其它仓库中。

所有状态都存在于这个堆中，甚至包括程序。每个区域都有一个程序来拥有和管理它。例如，程序由 [BPFLoader](https://solana.com/docs/programs/deploying#overview-of-the-upgradeable-bpf-loader) 程序拥有，它负责加载、部署和升级链上程序。我们将这些内存区域——我们数字仓库中的盒子——称为账户。

## 什么是账户？

在 Solana 上，一切都是账户（**Everything on Solana is an account.** ）。可以把账户想象成持久保存数据的容器，就像计算机上的文件一样。它们是 Solana 程序模型的基本构件，用于存储状态（即账户的余额、所有权信息、账户是否持有程序以及租金信息）。

Solana 上有三种类型的账户：

- 存储数据的账户
- 存储可执行程序的账户
- 存储原生程序（native programs）的账户

这些类型的账户可以根据其功能进一步区分为：

- **可执行账户**——能够运行代码的账户
- **不可执行账户**——用于数据存储的账户，不能执行代码（因为它们不包含任何代码！）

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/654e519c9bf0390c9c9a55d2_exec-non%20(1).jpg" loading="lazy" alt="">

在上图中，我们有几个可执行和不可执行账户的例子。对于可执行账户，[Bubblegum](https://github.com/metaplex-foundation/mpl-bubblegum) 是一个程序账户的例子，它是 Metaplex 用来创建和管理压缩 NFT 的程序。[投票程序(vote program)](https://docs.solanalabs.com/runtime/programs#vote-program) 是一个原生程序账户的例子，用于创建和管理跟踪验证者投票状态和奖励的账户。我们将在“[什么是程序](#什么是程序)？”部分中详细讨论程序账户和原生程序账户之间的区别。目前，重要的是要知道 Solana 上有不同类型的可执行账户。

此外，每个不可执行账户都可以分类为数据账户（Data Account）。数据账户的例子包括：

- **[关联代币账户](https://spl.solana.com/associated-token-account#motivation)（Associated Token Account）**——一个账户，包含有关特定代币的信息、余额及其所有者（例如，Alice 拥有 10 个 USDC）。
- **[系统账户](https://solana.com/docs/core/accounts#ownership-and-assignment-to-programs)（System Account）**——由 [系统程序(System Program) ](https://docs.solanalabs.com/runtime/programs#system-program)创建并拥有的账户。
- **[质押账户](https://solana.com/docs/economics/staking/stake-accounts)（Stake Account）**——一个用于将代币委托给验证者以潜在获得奖励的账户。

## 账户结构

账户按照 AccountInfo 结构体进行组织：

```rust
pub struct AccountInfo<'a> {
    pub key: &'a Pubkey,
    pub lamports: Rc<RefCell<&'a mut u64>>,
    pub data: Rc<RefCell<&'a mut [u8]>>,
    pub owner: &'a Pubkey,
    pub rent_epoch: Epoch,
    pub is_signer: bool,
    pub is_writable: bool,
    pub executable: bool,
}
```

账户由其地址（密钥）标识，这是一个唯一的 32 字节数组。

lamports 字段表示此账户拥有的[lamports](https://solana.com/docs/terminology#lamport)数量。一个 lamport 等于十亿分之一的 SOL，SOL 是 Solana 的[原生代币](https://solana.com/docs/terminology#native-token)。

**data**字段指的是该账户存储的原始数据字节数组。它可以存储从数字资产的元数据到代币余额的任何内容，并且可以由程序修改。

**owner** 字段包含账户的所有者，由程序账户的地址表示。关于账户所有权有一些规则：

- 只有账户的所有者可以更改其数据并提取 lamports。
- 任何人都可以向账户存入 lamports。
- 账户的所有者可以将所有权转移给新所有者，前提是账户的数据被重置为零。

**is_signer** 字段是一个布尔值，指示交易是否已由相关账户的所有者签名。换句话说，它告诉参与交易的程序该账户是否是签名者。作为签名者意味着账户持有公钥对应的私钥，并有权批准拟议的交易。

**is_writable** 字段是一个布尔值，指示账户的数据是否可以修改。Solana 允许交易指定账户为只读，以便于并行处理。虽然运行时允许只读账户同时被不同的程序访问，但它通过交易处理顺序处理潜在的写入冲突，以确保只有不冲突的交易可以并行处理。

**executable** 字段是一个布尔值，指示账户是否可以运行指令。是的，这意味着程序被存储在账户中，我们将在下一节中深入探讨。首先，我们需要讨论租金的概念。

**rent_epoch** 字段指示该账户将在下一个 epoch（时期）欠租金的时间点。[epoch ](https://solana.com/docs/terminology#epoch)是一个领导者调度表（leader schedult）有效的插槽数。与操作系统中的传统文件不同，Solana 上的账户有一个以 lamports 数量表示的生命周期。账户的持续存在取决于其 lamport 余额，这引出了租金的概念。

## 租金

租金（rent）是维持 Solana 上账户存活并确保账户存储在验证者内存中的存储成本。租金收取基于 epoch 进行，epoch 是一段由插槽（slot）定义的时间单位，在此期间领导者调度表有效。以下是租金的运作方式：

- **租金收取**——租金每个 epoch 收取一次。也可以在账户被交易引用时收取。
- **租金分配**——部分收取的租金被销毁，这意味着它被永久移出流通。剩余部分在每个插槽后分配给投票账户。
- **租金支付**——如果账户没有足够的 lamports 支付租金，则其数据将被删除，账户将在一个称为垃圾回收的过程中被释放。
- **租金豁免**——如果账户保持相当于两年租金支付的最低余额，则可以免租（rent-exempt）。所有新账户必须达到此租金豁免门槛，具体取决于账户的大小。
- **租金取回**——用户可以关闭账户以回收其剩余的 lamports。这允许用户取回账户中存储的租金。

租金可以通过 `getMinimumBalanceForRentExemption ` RPC 端点根据特定账户大小进行估算。[Test Drive ](https://testdrive.helius.xyz/)简化了这一过程，只需输入账户的数据长度（usize）。Solana 租金 CLI 子命令也可用于估算账户达到租金豁免所需的最低 SOL 数量。例如，在撰写本文时，运行命令`solana rent 20000`会返回`Rent-exempt minimum: 0.14009088 SOL`。

## Solana 上的地址

在 Solana 上，实际上有两种类型的地址。Solana 使用 ed25519，它是一种基于 [SHA-512（SHA-2）](https://en.wikipedia.org/wiki/SHA-2)和 [Curve25519](https://en.wikipedia.org/wiki/Curve25519) 椭圆曲线的 EdDSA 签名方案，用于地址的创建。这会生成 32 字节的公钥，作为主要的地址格式。它们可以直接使用，因为它们不是通过哈希计算得到的。

一个地址要有效，它必须是 ed25519 曲线上的一个点。然而，并非所有地址都需要从这条曲线派生出来。程序派生地址（PDAs）是从曲线外生成的，这意味着它们没有对应的私钥，无法用于签名。PDAs 通过系统程序创建，用于在程序需要管理账户时使用。这里仅是为读者简单介绍 Solana 上不同类型的地址，PDAs 将在未来的文章中详细介绍。

## Solana 上的账户与 Ethereum 上的账户有何不同？

以太坊（Ethereum）有两种主要的账户类型：外部拥有的账户（EOAs）和合约账户。EOAs 由私钥控制，而合约账户由其合约代码管理，无法自行发起交易。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/654d452c020fe54c3f62fa68_soleth.jpg" loading="lazy" alt="">

EOAs 和合约账户都遵循相同的账户结构：

- **余额** - 每个账户都有一个以太币（Ether）的余额。
- **Nonce** - 对于 EOAs，这是账户发送的交易计数。对于合约，这是账户创建的合约数量。
- **存储根** - 一个 256 位的哈希值，对应一个 [Merkle Patricia Trie](https://ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie/) 的根节点，这是账户存储内容的编码。
- **代码哈希** - 以太坊虚拟机（EVM）代码的哈希值。这个值是不可变的，意味着代码在创建后不会改变，尽管其状态可能会改变。需要注意的是，以太坊上可以通过代理模式等方式升级合约，但这超出了本文的讨论范围。对于 EOAs，这是一个空字符串的哈希值，因为 EOAs 不包含代码。

Solana 采用了一种更统一的账户模型，其中任何账户都有可能成为一个程序。代码和数据的分离促进了更高效、更灵活的环境。Solana 程序是无状态的，与各种数据账户交互而无需重复部署。这对于去中心化金融（DeFi）应用尤其有利，因为用户希望与多个协议进行交互，而无需在不同程序之间转移资产。相比之下，以太坊的编程模型将代码和状态结合成一个实体。这使得交互更加复杂，并可能因为状态更改的燃气费用而导致更高的成本。

Solana 的账户曾经需要支付租金，要求它们保持一个最低余额以保持活跃。这确保了未使用或资金不足的账户最终会被网络回收，从而减少状态膨胀。最近的更新使得主网上不再有支付租金的账户——账户必须免租金。相比之下，以太坊使用燃气管理资源分配。在这种模式下，合约存储将无限期保留，除非明确清除。Solana 的方法提供了一种更可预测的状态存储成本结构，而以太坊的成本可能会波动，并在网络拥堵期间变得过高。

在接下来的部分中，我们将研究 Solana 如何将其程序逻辑与状态分离。与以太坊的编程模型相比，你会看到这种模块化方法如何促进更高效的链上操作，同时为开发者提供一个透明且可预测的成本结构。

## 什么是程序？

程序是由[ BPF 加载器 ](https://docs.solanalabs.com/runtime/programs#bpf-loader)（BPF Loader）拥有的可执行账户。它们由 [Solana 运行时](https://solana.com/docs/core/fees)（Solana Runtime） 执行，Solana 运行时旨在处理交易和程序逻辑。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/654d46c12245f98f47d174d7_program.jpg" loading="lazy" alt="">

Solana 编程模型的一个显著特征是代码和数据的分离。程序是无状态的，意味着它们不会在内部存储任何状态。相反，所有它们需要操作的数据都存储在单独的账户中，通过交易引用传递给程序。这种设计允许单个通用程序的部署与不同的账户进行交互。

Solana 上的程序具有以下能力：

- 拥有额外的账户
- 读取或入账其他账户
- 修改数据或出账其拥有的账户

有两种类型的程序：

- **链上程序** - 这些是用户编写并部署在 Solana 上的程序。它们可以由其升级权限进行升级，这通常是部署该程序的账户。
- **原生程序** - 这些是集成到 Solana 核心中的程序。它们提供验证者操作所需的基本功能。原生程序只能通过全网的软件更新进行升级。常见的例子包括 [系统程序（System Program）](https://docs.solanalabs.com/runtime/programs#system-program)、[BPF 加载器程序](https://docs.solanalabs.com/runtime/programs#bpf-loader) 和[投票程序](https://docs.solanalabs.com/runtime/programs#vote-program)。

用户和其他程序都可以调用链上程序和原生程序。主要区别在于它们的升级机制：链上程序可以由其升级权限进行升级，而原生程序只能作为集群更新的一部分进行升级。

Solana Labs 策划了一组精选的链上程序，称为 [Solana 程序库（Solana Program Library）](https://spl.solana.com/) 。该库支持各种链上操作，包括代币借贷和质押池的创建。例如，[关联代币账户程序](https://spl.solana.com/associated-token-account)（Associated Token Account Program）为将用户的钱包链接到其相应的代币账户设置了标准和机制。此外，SPL 是动态的。像[Token-2022](https://spl.solana.com/token-2022)这样的程序在原[Token 程序](https://spl.solana.com/token)的基础上构建并扩展了其功能。

在 Solana 上，程序开发通常使用 [Rust 语言](https://www.rust-lang.org/)，并借助 [Anchor 框架](https://www.anchor-lang.com/)。Anchor 是一个优化框架，通过减少样板代码并简化序列化和反序列化来简化程序的创建。虽然 Rust 是首选，但开发者并不限于它——C、C++ 以及任何可以编译为 [BPF 字节码](https://en.wikipedia.org/wiki/Berkeley_Packet_Filter) 的语言都可以使用。[Solang](https://solang.readthedocs.io/en/latest/) 和 [Neon Labs](https://neon-labs.org/) 的最新开发已经允许开发者在程序开发中使用 [Solidity 语言](https://soliditylang.org/)。

程序通常在 Localhost 和 Devnet 上开发和测试，然后部署到 Testnet 或 Mainnet Beta。开发者可以通过 Solana CLI 使用命令`solana program deploy <path to program>`部署他们的程序。程序一旦编译为包含 BPF 字节码的 [ELF 共享对象](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) 后，就会上传到指定的 Solana 集群。部署的程序存在于标记为可执行的账户中，账户的地址作为`program_id`。

最初，Solana 上的程序是以两倍于程序大小的账户进行部署的。[Solana 的 1.16 更新引入了支持可调整大小的账户](https://www.helius.dev/blog/all-you-need-to-know-about-solanas-v1-16-update#support-for-resizable-accounts)，为开发者提供了更多的灵活性和资源分配。现在，开发者可以使用较小的账户大小部署他们的程序，并在以后扩展其大小。

如上所述，程序被认为是无状态的，因为它们交互的任何数据都存储在单独的账户中，并作为引用传递给它们。所有程序都有一个单一的入口点进行指令处理，该入口点接收`program_id`、账户数组和指令数据作为字节数组。一旦由交易调用，Solana 运行时将执行程序。

## 什么是交易？

交易是链上活动的核心。它们是调用程序和执行状态更改的机制。Solana 上的交易是指示验证者应执行哪些操作、在哪些账户上执行以及是否有必要权限执行这些操作的指令集合。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/654d4705a387b49455c060d2_transaction.jpg" loading="lazy" alt="">

一个交易由三个主要部分组成：

- 读取或写入的账户数组
- 一个或多个指令
- 一个或多个签名

Solana 上的交易遵循 Transaction 结构体。它提供了网络处理和验证操作所需的信息。其定义如下：

```rust
pub struct Transaction {
    pub signatures: Vec<Signature>,
    pub message: Message,
}
```

`signatures`字段包含与序列化的 Message 对应的一组签名。每个签名与 Message 的`account_keys`列表中的一个账户密钥相关联，从支付费用的账户开始。支付费用的账户是负责支付处理交易时产生的交易费用的账户。这通常是发起交易的账户。所需的签名数量等于消息头中的`num_required_signatures`。

消息本身是一个类型为 Message 的结构体。其定义如下：

```rust
pub struct Message {
    pub header: MessageHeader,
    pub account_keys: Vec<Pubkey>,
    pub recent_blockhash: Hash,
    pub instructions: Vec<CompiledInstruction>,
}
```

消息的头部包含三个无符号 8 位整数：所需的签名数量（即`num_required_signatures`）、只读签名者数量和只读非签名者数量。

`account_keys`字段列出了交易中涉及的所有账户地址。请求读写访问的账户首先列出，然后是只读账户。

`recent_blockhash`是一个最近的区块哈希，包含一个 32 字节的 SHA-256 哈希值。这是为了指示客户端上次观察账本的时间，并作为最近交易的生命周期。如果交易的区块哈希过旧，验证者将拒绝该交易。此外，包含一个最近的区块哈希有助于防止重复交易，因为任何与之前完全相同的交易都会被拒绝。如果由于某种原因需要在提交到网络之前很久签署交易，可以使用[持久交易随机数](https://www.helius.dev/blog/solana-transactions)（durable nonce）来代替最近的区块哈希，以确保这是一个独特的交易。

`instructions`字段包含一个或多个`CompiledInstruction`结构体，每个结构体指示网络验证者应采取的特定行动。

## 指令

指令是一次 Solana 程序调用的指示。它是程序中最小的执行逻辑单元，也是 Solana 上最基本的操作单元。程序解释从指令传递的数据并对指定的账户进行操作。Instruction 结构体定义如下：

```rust
pub struct Instruction {
    pub program_id: Pubkey,
    pub accounts: Vec<AccountMeta>,
    pub data: Vec<u8>,
}
```

`program_id`字段指定要执行的程序的公钥。这是将处理指令的程序的地址。该程序账户的所有者由这个公钥指示，指定了负责初始化和执行程序的加载器。加载器将链上的 Solana 字节码格式（SBF）程序标记为可执行。一旦部署，Solana 运行时将拒绝任何尝试调用未标记为可执行账户的交易。

`accounts`字段列出了指令可能读取或写入的账户。这些账户必须以`AccountMeta`值提供。任何数据可能被指令修改的账户都必须指定为可写，否则交易将失败。这是因为程序无法写入它们不拥有或没有相应权限的账户。这也适用于修改账户的 lamports：从程序不拥有的账户中扣除 lamports 将导致交易失败，而向任何账户添加 lamports 是允许的。`accounts`字段还可以指定程序不读取或写入的账户。这是为了影响运行时的程序执行调度，但除此之外，这些账户将被忽略。

`data`是一个 8 位无符号整数的通用向量，作为传递给程序的输入。该字段非常重要，因为它包含程序将要执行的指令的编码内容。

Solana 对指令数据的格式不拘一格。然而，它内置了对 bincode 和 borsh（用于哈希的二进制对象表示序列化器）的支持。序列化是将复杂数据结构转换为可传输或存储的平面字节序列的过程。数据编码的选择应考虑解码的开销，因为这一切都发生在链上。[Borsh](https://borsh.io/) 序列化通常比 [bincode](https://github.com/bincode-org/bincode) 更受欢迎，因为它有一个稳定的规范，有 [JavaScript 实现](https://github.com/near/borsh-js)，并且通常更高效。

程序使用辅助函数来简化支持指令的构造。例如，系统程序提供了一个辅助函数来构造`SystemInstruction::Assign`指令：

```rust
pub fn assign(pubkey: &Pubkey, owner: &Pubkey) -> Instruction {
    let account_metas = vec![AccountMeta::new(*pubkey, true)];
    Instruction::new(
        system_program::id(),
        &SystemInstruction::Assign { owner: *owner },
        account_metas,
    )
}
```

此函数构造一个指令，当处理时，它将把指定账户的所有者更改为提供的新所有者。

一个交易可以包含多个指令，它们按照列出的顺序顺序执行并且原子性执行。这意味着所有指令要么全部成功，要么全部失败。这也意味着指令的顺序可能至关重要。程序必须经过强化，以安全地处理任何可能的指令顺序，以防止潜在的漏洞。

例如，在反初始化过程中，程序可能试图通过将账户的 lamport 余额设置为零来反初始化一个账户。这假设 Solana 运行时将删除该账户。这个假设在交易之间是有效的，但在指令或跨程序调用之间则无效（我们将在未来的文章中介绍跨程序调用）。程序应该明确将账户的数据清零，以防范这种反初始化过程中的潜在漏洞。否则，攻击者可能会发出后续指令，以利用假定的删除，例如在交易完成前重新使用该账户。

## 什么是版本化交易？

Solana 的交易使用 [IPv6 最大传输单元（MTU，Maximum Transmission Unit）](https://en.wikipedia.org/wiki/Maximum_transmission_unit) 标准，以确保数据在集群中的快速和可靠传输。Solana 的网络栈采用了保守的 1280 字节 MTU 大小。预留头部空间后，剩余 1232 字节可用于数据包数据。因此，Solana 的交易大小受限于这个值。

这种大小限制促进了一系列网络增强功能，但也限制了在单个交易中可以执行的操作复杂性。考虑到每个账户地址占用 32 字节的存储空间，交易最多可以存储 35 个不含指令的账户。这一限制对那些需要在单个交易中包含超过 35 个无需签名账户的用例构成了挑战。

为了解决这个问题，引入了一种新交易格式，支持多种交易格式版本。目前，Solana 运行时支持两种交易版本：

- **legacy** - 原始交易格式
- **0（版本 0）** - 最新的交易格式，支持地址查找表（Address Lookup Tables）

版本 0 的发布是为了支持[地址查找表（ALTs）](https://solana.com/docs/advanced/lookup-tables)。这些表本质上是将账户地址存储在链上的类似于表格的数据结构中。这些表是存储账户地址的独立账户，并允许在交易中使用 1 字节的 u8 索引引用它们。这大大减少了交易的大小，因为每个包含的账户只需要使用 1 字节而不是 32 字节。ALTs 对于涉及许多账户的复杂操作特别有用，例如在去中心化金融（DeFi）应用中常见的那些操作。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/654d4896dc72540225aa3af3_alts.jpg" loading="lazy" alt="">

“版本化交易”一词指的是 Solana 支持 legacy 和版本 0 交易格式的方式。这种方式确保了组合性，同时也迎合了运行时的增强功能。

## 版本化交易的结构

一个`VersionedTransaction`的定义如下：

```rust
pub struct VersionedTransaction {
    pub signatures: Vec<Signature>,
    pub message: VersionedMessage,
}
```

`signatures`字段是来自交易签名者的签名列表，它们用于认证并保持交易的完整性。`message`是交易的实际内容。这由`VersionedMessage`类型封装，这是一个处理 legacy 和版本 0 消息的轻量级枚举包装器：

```rust
pub enum VersionedMessage {
    Legacy(Message),
    V0(Message),
}
```


消息版本由序列化过程中第一个比特位决定。如果第一个比特位被设置，那么剩下的 7 位用于确定序列化的消息版本，从版本 0 开始。如果第一个比特位没有被设置，那么所有字节用于编码 legacy 消息格式。这是因为有两个同名的消息结构体`Message`，但它们分属不同的模块—— [legacy](https://docs.rs/solana-sdk/latest/solana_sdk/message/legacy/index.html) 和[v0](https://docs.rs/solana-sdk/latest/solana_sdk/message/v0/index.html)。


`Message`表示交易的简化内部格式，用于网络传输和运行时处理。它包含了交易指令中使用的所有账户的线性列表、描述账户数组结构的`MessageHeader`、一个最近的区块哈希以及消息指令的紧凑编码。v0 消息结构体的结构如下：

```rust
pub struct Message {
    pub header: MessageHeader,
    pub account_keys: Vec<Pubkey>,
    pub recent_blockhash: Hash,
    pub instructions: Vec<CompiledInstruction>,
    pub address_table_lookups: Vec<MessageAddressTableLookup>,
}
```

legacy 消息和 v0 消息之间的区别在于`address_table_lookups`字段的存在。

## 将编程模型与 Solana 的交易流程集成

Solana 的编程模型与其账户和交易系统紧密集成。以下是这些概念如何联系在一起：

- **账户作为状态** - Solana 上的账户充当程序的状态容器。编程模型围绕着响应指令修改这些容器中存储的数据。
- **指令** - 程序定义了处理交易中包含的指令的逻辑。这些指令是与账户数据交互的可执行组件。
- **序列化和处理** - 当交易被序列化时，程序的指令决定了账户状态的变化。序列化过程遵循程序的设计，无论它使用的是 legacy 还是版本 0 的交易格式。
- **原子性** - Solana 的编程模型确保指令处理的原子性。程序必须设计为能够安全高效地处理并发交易。
- **可扩展性** - Solana 的编程模型通过地址查找表（ALTs）等功能支持可扩展性。这些表减少了交易的大小并增加了交易可以引用的账户数量。

Solana 的编程模型不仅仅是编写代码——它还涉及理解代码在更广泛生态系统中的交互方式。账户是这个模型的核心，它们是数据在网络上存储和修改的主要方式。交易通过告知验证者需要创建、更新或删除哪些数据来实现链上活动。对这些方面的深刻理解对于开发者来说至关重要，以构建充分利用 Solana 能力的应用程序。

## 结论

恭喜你！在这篇文章中，我们深入探讨了 Solana 系统架构的复杂性，研究了集群作为单一数据堆的概念。我们发现这个堆是如何组织成称为账户的独立内存区域的，它们构成了 Solana 编程模型的支柱。账户存储了从用户代币到定义网络行为的程序的所有内容，所有这些内容都通过交易进行修改。

对于开发者来说，理解 Solana 的去中心化计算方法至关重要。掌握账户、程序和交易的复杂性是构建充分利用 Solana 功能的应用程序所必需的。这涉及理解一个代码与状态解耦的系统，结果是无状态的程序在一个前所未有的可组合性和可升级性规模上与数据交互。

对于投资者和普通用户来说，理解 Solana 的设计如何创建一个稳健、灵活且高效的生态系统，对于理解平台的可行性及其培育只有在 Solana 上可能的创新应用的能力至关重要。

如果你读到这里，感谢你！准备深入研究？加入我们的 [Discord 社区](https://discord.com/invite/6GXdee3gBj)，今天就开始在 Solana 上编程吧。

### 其他资源/进一步阅读

- [Solana 命令行指南](https://docs.solanalabs.com/cli/)
- [Solana Wiki - 账户模型](https://solana.wiki/zh-cn/docs/account-model/)
- [账户抽象简介](https://squads.so/blog/what-is-account-abstraction-ethereum-vs-solana)
- [垃圾回收](https://docs.solanalabs.com/implemented-proposals/persistent-account-storage#garbage-collection)
- [地址查找表（ALTs）](https://solana.com/docs/advanced/lookup-tables)
- [版本化交易](https://solanacookbook.com/guides/versioned-transactions.html#facts)
- [Crate solana_sdk](https://docs.rs/solana-sdk/latest/solana_sdk/index.html)
