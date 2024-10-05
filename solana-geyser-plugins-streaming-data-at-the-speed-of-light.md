# Solana Geyser 插件：以光速传输数据

## 这篇文章讲的是什么？

Geyser 插件是模块化组件，旨在将账户、时隙、区块和交易的数据传输到外部数据存储中，使开发者能够从验证节点中移除 RPC（远程过程调用）负载。Geyser 插件为希望自定义数据流和处理需求的开发者提供了灵活的解决方案。

在本文中，我们将深入探讨 Solana Geyser 插件的复杂性。我们将首先探索 AccountsDB 副本，这是一种提议的数据复制和负载管理方法，最终被 Geyser 插件所取代。然后，我们将分解 Geyser 插件的概念、功能和通过插件接口的结构。接下来，我们将讨论常见的 Geyser 插件，并指导您完成创建自己的插件的复杂过程。最后，我们将讨论 Helius，以及它如何简化 Solana 上的数据流传输。

## AccountsDB 副本：一种被放弃的数据复制和 RPC 负载方法

Solana 探索了多种方法来解决重 RPC 负载和数据复制的挑战。一种有前景的方法是使用 [AccountsDB 副本](https://docs.solana.com/proposals/accounts-db-replication)。这些副本旨在将账户扫描请求从主验证节点转移到 AccountsDB 副本。尽管前景看好，但该系统本质上很复杂，需要一套新的服务来确保主验证节点和副本之间的同步。最终，这个提议被放弃，转而采用 Geyser 插件系统——这是一个对验证节点客户端来说更简单的支持方案，同时也为开发者在实现应用时提供了更大的灵活性。那么，Solana Geyser 插件究竟是什么呢？

## Solana Geyser 插件是什么？

Solana Geyser 插件提供了对 Solana 数据的低延迟访问，可以为应用程序提供不需要在验证节点上进行 RPC 调用。例如，如果一个验证节点必须快速连续地处理大量的 getProgramAccounts 调用，由于这种密集的流量，验证节点可能会落后于网络。Geyser 插件通过将账户、区块、时隙和交易的信息重定向到外部数据存储（如关系数据库、NoSQL 数据库或 Kafka）来解决这个问题。这种数据重定向允许 RPC 服务为那些需要从这些外部存储获取数据的用户提供更灵活和有针对性的优化，如缓存和索引。

Geyser 插件充当 Solana 和外部数据存储解决方案之间的桥梁。它们使开发者能够将大部分数据管理任务从验证节点卸载出去，从而提高性能并降低潜在瓶颈的风险。无论 RPC 流量如何，Geyser 插件都能确保验证节点与网络保持同步。

## Geyser 插件接口

开发者可以使用 Solana Geyser 插件接口来构建 Geyser 插件。该接口提供了对账户、交易、时隙、区块元数据和条目的访问。它在 [solana-geyser-plugin-interface crate](https://docs.rs/solana-geyser-plugin-interface/latest/solana_geyser_plugin_interface/) 中声明，并由 [GeyserPlugin trait](https://docs.rs/solana-geyser-plugin-interface/latest/solana_geyser_plugin_interface/geyser_plugin_interface/trait.GeyserPlugin.html) 定义。这个 trait 定义了多个方法，每个方法都以 update\_ 为前缀，当新数据被创建或现有数据被更新时，这些方法会被调用。Geyser 插件还需要指定其在加载和卸载过程中的行为。该 trait 概述了 Geyser 插件应该实现的基本方法，以确保根据所需的插件行为进行高效的数据流传输。

### 源码

```rust
pub trait GeyserPlugin:Any +Send +Sync +Debug {
    // Required method
    fn name(&self) -> &'static str;

    // Provided methods
    fn on_load(&mut self, _config_file: &str) ->Result<()> { ... }
    fn on_unload(&mut self) { ... }
    fn update_account(
        &self,
        account:ReplicaAccountInfoVersions<'_>,
        slot: Slot,
        is_startup:bool
    ) ->Result<()> { ... }
    fn notify_end_of_startup(&self) ->Result<()> { ... }
    fn update_slot_status(
        &self,
        slot: Slot,
        parent:Option,
        status:SlotStatus
    ) ->Result<()> { ... }
    fn notify_transaction(
        &self,
        transaction:ReplicaTransactionInfoVersions<'_>,
        slot: Slot
    ) ->Result<()> { ... }
    fn notify_entry(&self, entry:ReplicaEntryInfoVersions<'_>) ->Result<()> { ... }
    fn notify_block_metadata(
        &self,
        blockinfo:ReplicaBlockInfoVersions<'_>
    ) ->Result<()> { ... }
    fn account_data_notifications_enabled(&self) ->bool { ... }
    fn transaction_notifications_enabled(&self) ->bool { ... }
    fn entry_notifications_enabled(&self) ->bool { ... }
}
```

### 特征描述

GeyserPlugin trait 是 Solana Geyser 插件生态系统中所有插件的基础接口。它被声明为一个公共 trait，带有 Rust 标准库中的 Any、Send、Sync 和 Debug trait 约束。这些 trait 约束的作用如下：

- Any 允许类型反射，这使得可以向下转换到具体类型
- Send 表示实现此 trait 的类型的所有权可以在线程之间转移
- Sync 意味着实现此 trait 的类型的引用可以在线程之间共享
- Debug 允许格式化类型以输出，特别是用于调试目的

Any 和 Debug 对我们来说并不是很重要。真正重要的是 GeyserPlugin 需要 Send 和 Sync 来使程序线程安全。

### 所需方法

```rust
fn name(&self) -> &'static str;
```

name 方法是任何实现 GeyserPlugin 的类型所必需的。这个方法作为 Geyser 插件的标识符。它返回一个静态字符串切片，代表 Geyser 插件的名称。

这个方法以及除了 on_load 和 on_unload 之外的所有其他方法使用 &self 而不是 &mut self 是 Solana 1.16 更新中的新特性。这大大提高了性能，因为不再需要将 Geyser 插件包装在读写锁中，并且每次调用其函数时都获取写锁。

### 提供的方法

该 trait 有许多提供的方法，它们包含默认实现，这些默认实现可以被 GeyserPlugin 的具体实现所覆盖。

```rust
fn on_load(&mut self, _config_file: &str) ->Result<()> { ... }
```

on_load 方法是当插件被系统加载时调用的回调函数，用于执行插件所需的任何初始化操作。它接受一个字符串引用，表示配置文件的路径。配置文件必须是 JSON5 格式，并且包含一个 libpath 字段，指示实现此接口的共享库的完整路径名。

```rust
fn on_unload(&mut self) { ... }
```

on_unload 方法是一个回调函数，在系统卸载插件之前调用，用于执行任何必要的清理工作。

```rust
fn update_account(
        &self,
        account:ReplicaAccountInfoVersions<'_>,
        slot: Slot,
        is_startup:bool
    ) ->Result<()> { ... }
```

update_account 方法在账户在已处理（processed）确认级别上更新时被调用，这在一个插槽内可能发生多次。在这里，跟踪已确认（confirmed）的插槽至关重要，以获取提交到规范链的账户更新。ReplicaAccountInfoVersions 结构包含了流式传输的账户的元数据和数据。slot 参数指向账户正在更新的插槽。当 is_startup 为 true 时，表示账户是在验证节点启动时从快照加载的。当 is_startup 为 false 时，账户是在交易处理过程中更新的。

```rust
fn notify_end_of_startup(&self) ->Result<()> { ... }
```

notify_end_of_startup 方法被调用以表示启动阶段的结束。这发生在验证节点已从快照恢复账户数据库，并且所有账户都已相应更新之后。

```rust
fn update_slot_status(
        &self,
        slot: Slot,
        parent:Option,
        status:SlotStatus
    ) ->Result<()> { ... }
```

update_slot_status 方法在插槽状态更新时被调用。它接受一个 Slot，一个 Option<u64> 类型的父插槽，以及一个 SlotStatus 枚举实例。

SlotStatus 概述了 Solana 中插槽的三种状态：

1. Processed（已处理） - 节点已处理的最高插槽。虽然该插槽既未确认也未最终确定，但它是验证者认为最有可能成为规范链的一部分。

2. Confirmed（已确认） - 该插槽已收到足够的投票，被认为是安全的，并且是链的一部分。这个插槽得到了 Solana 验证者超级多数的支持。

3. Rooted（已根化） - 该插槽现在是区块链的永久部分，所有其他版本或链的分叉都必须建立在这个插槽之上。这意味着网络上的所有分支都源自这个区块。

```rust
fn notify_transaction(
        &self,
        transaction:ReplicaTransactionInfoVersions<'_>,
        slot: Slot
    ) ->Result<()> { ... }
```

notify_transaction 方法在插槽中处理交易时被调用，向插件通知交易的详细信息。ReplicaTransactionInfoVersions 是一个枚举包装器，用于处理 ReplicaTransactionInfo。如果 ReplicaTransactionInfo 的结构发生变化，就会为新版本添加一个新的枚举条目。这将迫使插件实现通过适应新的枚举条目来处理变化。目前，该枚举包装了两个变体：V0_0_1(&'a ReplicaTransactionInfo<'a>) 和 V0_0_2(&'a ReplicaTransactionInfoV2<'a>)：

```rust
pub struct ReplicaTransactionInfo<'a> {
    pub signature: &'a Signature,
    pub is_vote: bool,
    pub transaction: &'a SanitizedTransaction,
    pub transaction_status_meta: &'a TransactionStatusMeta,
}

pub struct ReplicaTransactionInfoV2<'a> {
    pub signature: &'a Signature,
    pub is_vote: bool,
    pub transaction: &'a SanitizedTransaction,
    pub transaction_status_meta: &'a TransactionStatusMeta,
    pub index: usize,
}
```

这两个变体之间的主要区别在于第二个变体存储了交易在区块中的索引。

```rust
fn notify_entry(&self, entry:ReplicaEntryInfoVersions<'_>) ->Result<()> { ... }
```

notify_entry 方法在有新条目时通知插件。它接受一个 ReplicaEntryInfoVersions 实例，这是一个用于未来兼容 ReplicaEntryInfo 处理的包装器。目前，它包含 V0_0_1(&'a ReplicaEntryInfo<'a>) [变体](https://docs.rs/solana-geyser-plugin-interface/latest/solana_geyser_plugin_interface/geyser_plugin_interface/struct.ReplicaEntryInfo.html)。这个变体是一个结构体，包含了条目的插槽信息、在区块中的索引、自上一个条目以来的哈希数量、条目的 SHA-256 哈希值，以及条目中已执行的交易数量。

```rust
fn notify_block_metadata(
        &self,
        blockinfo:ReplicaBlockInfoVersions<'_>
    ) ->Result<()> { ... }
```

当区块的元数据更新时，会调用 notify_block_metadata 方法。它接受一个 ReplicaBlockInfoVersions 枚举实例作为其区块信息。这个枚举是各种 ReplicaBlockInfo 版本的包装器，包含了关于区块的信息，如插槽、哈希值、奖励、区块时间、区块高度等。

```rust
fn account_data_notifications_enabled(&self) ->bool { ... }
fn transaction_notifications_enabled(&self) ->bool { ... }
fn entry_notifications_enabled(&self) ->bool { ... }
```

这些方法分别返回布尔值，表示插件是否希望启用账户数据、交易和条目的通知。

### 关于确认级别的说明

Geyser 在处理账户数据和交易后立即发送更新。这对端到端索引速度有利，但存在处理过的插槽可能被跳过的风险。跳过的插槽指的是过去未产生区块的插槽，原因可能是领导节点离线或包含该插槽的分叉被放弃而选择了更好的替代方案。对于被流式传输的数据存储系统来说，认识到这种可能性并相应地管理更新至关重要。

## 常见的 Solana Geyser 插件

Solana 生态系统中有各种 Geyser 插件可供开发者使用，甚至可以根据特定需求进行分叉。以下是一些值得注意的插件：

- PostgreSQL 插件：用于使用 PostgreSQL 管理和查询数据
- gRPC 服务流式传输插件：用于将 Solana 账户更新流式传输到 gRPC 服务
- RabbitMQ 生产者插件：用于通过 RabbitMQ 实现消息队列
- Kafka 生产者插件：用于使用 Kafka 进行数据流式传输
- Amazon SQS 插件：利用 Amazon 的简单队列服务进行消息队列
- Google BigTable 插件：用于使用 Google BigTable 管理和查询数据

这些插件可以适应各种用例。例如，[Clockwork](https://www.clockwork.xyz/) 利用 Geyser 插件来调度交易并构建自动化的、事件驱动的 Solana 程序。尽管该项目已经结束，但其开源代码仍然是一个宝贵的资源，可以在他们的 [GitHub](https://github.com/clockwork-xyz) 上查看。其他可能的用例包括使用 Geyser 插件监控 DeFi 平台上的账户余额、提供网络健康指标或实时监控供应链事件。

## 创建您自己的 Solana Geyser 插件

### Solana Geyser 插件脚手架

[Solana Geyser 插件脚手架](https://github.com/JamesMDunn/solana-geyser-plugin-scaffold)是开始您的 Solana Geyser 插件开发之旅最简单的资源。这个脚手架作为一个最小化的模板，记录了[插件管理器](https://github.com/solana-labs/solana/tree/master/geyser-plugin-manager)和插件本身之间的交互。这是一个很好的起点，可以让您熟悉插件工作流程以及调试技术。

### 插件管理器

插件管理器是指导所有 Geyser 插件的生命周期和交互的核心组件。它能够在运行时动态加载和卸载插件，从而提供更大的灵活性和模块化。

在运行时，插件管理器将配置文件的路径传递给您的插件。这使得 Geyser 插件中的可自定义设置可以在不更改插件代码的情况下进行修改。要将插件集成到验证器中，您需要使用 --geyser-plugin-config 参数指定动态库路径。这告诉验证器在哪里找到插件及其相关配置。配置文件至少必须是 JSON 格式，并包含 Geyser 插件动态库的路径 - 在 Linux 上是 .so 文件。一个最小的配置文件看起来如下：

```
{
    "libpath": "/.so"
}
```

### 从头开始创建 Geyser 插件

如果您想走一条不同寻常的道路，不使用脚手架或修改现有插件来创建自己的 Geyser 插件，您需要使用 Geyser 插件接口来编写您的插件。插件必须实现 GeyserPlugin trait 才能与运行时一起工作。此外，动态库必须导出一个 "C" 函数 \_create_plugin，该函数创建插件的实现。一个例子是创建一个实现 GeyserPlugin trait 的 Webhook 插件：

```rust
#[no_mangle]
#[allow(improper_ctypes_definitions)]
/// # Safety
///
/// This function returns the WebhookPlugin pointer as trait GeyserPlugin.
pub unsafe extern "C" fn _create_plugin() -> *mut dyn GeyserPlugin {
    let plugin = WebhookPlugin::new();
    let plugin: Box = Box::new(plugin);
    Box::into_raw(plugin)
}
```

在这里，我们创建了一个不安全的公共函数，使用 C 调用约定 extern "C"，这使得它与 C 和其他语言兼容。函数本身 fn \_create_plugin() -> \*mut dyn GeyserPlugin 返回一个指向 dyn GeyserPlugin 的可变原始指针，其中 GeyserPlugin 是一个 trait。函数体创建了 WebhookPlugin 的新实例，将这个实例装箱为 trait 对象，然后将装箱的 trait 对象转换为原始指针，以便函数可以返回。

因此，创建自己的 Geyser 插件的步骤如下：

1. 构建实现 Solana Geyser 插件接口的插件
2. 从 target/release 或 target/debug 文件夹获取动态库（.so 文件）
3. 创建 geyser-config.json 文件，该文件必须在 "libpath" 字段下包含 Geyser 插件动态库的路径
4. 使用 --geyser-plugin-config geyser-config.json 标志启动验证器

这些步骤听起来相当简单，然而，实际运行和维护 Solana Geyser 插件的过程可能相当艰巨。

## Helius Geyser 流式传输

Helius 以在 Solana 上提供无与伦比的开发者体验而闻名。这种对 Solana 的专注使 Helius 积累了丰富的经验，成功应对了广泛的挑战并促成了众多大规模集成。Helius 独特地定位于解决开发者可能面临的任何问题。

在 Helius，我们为 Solana 生态系统中的几个高性能团队管理 Geyser 插件。我们运营专门的 Geyser 集群，具有额外的冗余和容错能力，确保您永远不必担心数据丢失或停机。我们的程序化 API 访问允许您动态修改 Geyser 插件，而无需担心可靠性问题。管理 Geyser 插件通常是一项艰巨的任务，因为您需要负责确保数据一致性、可靠性和可用性。为什么不让 Helius 为您做这些呢？

如果您对 Geyser 流式传输感兴趣，请立即在 [Discord](https://discord.com/invite/6GXdee3gBj) 上联系我们以开始使用。

## 结论

恭喜！在本文中，我们通过研究 Solana Geyser 插件，探讨了数据复制和 RPC 负载管理的复杂性。理解这个系统并非易事 - 它是一个几乎没有文档记录的复杂架构，但为 Solana 开发者提供了丰富的定制和性能优化机会。

本文中获得的知识是无价的，特别是对于那些希望在 Solana 上构建或管理高性能应用程序的开发者或团队。理解 Geyser 插件至关重要，因为它们为 Solana 生态系统提供了可扩展和可靠的解决方案。

如果你已经读到这里，匿名朋友，谢谢你！

## 额外资源 / 进一步阅读

- [Helius Discord](https://discord.com/invite/6GXdee3gBj)
- [Solana 关于 Geyser 插件的文档](https://docs.solana.com/developing/plugins/geyser-plugins)
- [Solana Geyser 插件脚手架](https://github.com/mwrites/solana-geyser-plugin-scaffold)
- [Solana Geyser 插件接口 Crate](https://docs.rs/solana-geyser-plugin-interface/latest/solana_geyser_plugin_interface/)
