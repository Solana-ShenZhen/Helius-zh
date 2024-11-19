# Agave v2.0 更新指南

感谢 Jacob Creech、Rex St.John、Brooks Prumo 和 0xIchigo 审阅本文的早期版本。

## Agave 2.0 概述

Agave 验证节点客户端 v2.0 的发布标志着 Solana 在构建更强大的多客户端生态系统道路上的重要里程碑。此次更新引入了多项关键改进，以提升网络性能、可靠性和效率。主要更新包括：

- 大规模代码库重构和优化
- 分区纪元奖励
- 向验证节点发放全额优先费用
- 新的中央调度器现已默认启用
- **ZK ElGamal 证明**程序
- **Get-Sysvar** 系统调用
- **GetEpochStake** 系统调用
- **MoveStake** 和 **MoveLamports**
- 移除已弃用的 RPC 方法
- 包名重命名

无论您是运行验证节点、在平台上开发，还是积极使用 Solana，这份 Agave 2.0 更新的全面概述都将为您提供理解和利用这些最新创新所需的见解。

### 为什么这是一个主要版本更新？

**不再只有一个"Solana 验证节点"。** Agave 2.0 拥抱 Solana 的新多客户端世界，并与旧的 [Solana Labs GitHub 仓库](https://github.com/solana-labs/solana) 彻底分离。Solana Labs 仓库将被归档，不再接受新的拉取请求或问题。此前，该仓库镜像了 Agave 仓库的活动。如果开发者还没有这样做，应该将所有活动迁移到 [Anza Agave GitHub 仓库](https://github.com/anza-xyz/agave)。[从 Solana Labs 到 Agave 的迁移过程](https://github.com/anza-xyz/agave/wiki/Agave-Transition)于 3 月 1 日开始，并在其 GitHub 上公开跟踪。

随着生态系统的发展，运营者必须适应运行一个或多个客户端。随着这一转变，几个代码包正在被重命名，清理命名空间以支持多个客户端——最显著的是由独立开发团队管理的 Firedancer。由 Anza 维护的代码包现在将以"agave"为前缀，使它们在多客户端环境中易于识别为 Anza 特定的依赖项。

![img](https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66cec83bb17ca6be7d3c312f_AD_4nXdsR9MQUNDpsL_JQs4sjefy6YNVG_XntMmjwWl7EhtXHEm47QQAZlS51Aakv1kNmW3EY6si2oLJ9DK3ShIbQy6kACKhdH4gZvhcHgA5pDeF1XWdCVFm4SkJiT6MmtPJRb28fIjpu4ZJ-bFvC_y02gpCiSg.png)

受影响的代码包包括：

**solana-validator、solana-ledger-tool、solana-watchtower、solana-install、solana-geyser-plugin-interface、solana-cargo-registry**

正如我们在[之前的过渡指南](https://www.helius.dev/blog/agave-2-0-transition#removed-validator-arguments)中详细说明的那样，2.0 更新引入了几个重大变更，最显著的是移除了几个过时和弃用的端点——这些都是所有 Solana 开发者现在应该知道的关键更新。RPC 变更的完整详情将在本文末尾提供。

### 功能发布

在撰写本文时，约 20.7% 的验证节点正在运行 2.0.14 版本。主网上的功能门控激活暂时暂停，以使 v2.0 的采用与测试网和开发网上的激活更加一致。一旦主网集群广泛采用 v2.0，功能门控激活预计将按照[预定的激活顺序](https://github.com/anza-xyz/agave/wiki/Feature-Gate-Tracker-Schedule)恢复。

以下各节讨论的新完整功能目前尚未上线，将在 2.0 生命周期内使用功能门控系统逐步推出。功能会根据相对优先级以及在测试网和开发网集群上的激活顺序，在特定的纪元被激活。

## 向验证节点发放全额优先费用

这项备受期待且[广受争议的经济更新](https://forum.solana.com/t/proposal-for-enabling-the-reward-full-priority-fee-to-validator-on-solana-mainnet-beta/1456/97)现已根据提案 [SIMD-0096](https://github.com/solana-foundation/solana-improvement-documents/pull/96) 开始实施。该提案在 5 月份经过验证节点治理投票，投票在第 620 个纪元结束时完成，51.17% 的质押参与了投票，其中 77.77% 投票支持。这项功能门控更新将从根本上改变网络对[优先费用](https://www.helius.dev/blog/priority-fees-understanding-solanas-transaction-fee-mechanics)的处理方式。不同于当前 50% 销毁、50% 奖励给验证节点的模式，新模式将把 100% 的优先费用直接分配给验证节点。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/672b9cf4aaf47f077bc22373_672b965570c7fdcb6b9d2896_Decentralization%2520(1).png" loading="lazy" alt="显示Solana每周优先费用趋势的图表">

上图：Solana 每周优先费用（美元价值）([来源](https://solana.blockworksresearch.com/?dashboard=sol-financial&currency=USD&interval=weekly))

虽然优先费用在技术上是可选的，但随着 Solana 上经济活动的增加，它已成为标准做法。这些费用使用以下公式以微 lamport（lamport 的百万分之一）每计算单元计算：

优先费用 = 计算单元价格（微 lamport）x 计算单元限制

今后，所有优先费用都将奖励给区块生产者。这创造了更强的激励机制对齐，并降低了验证节点参与协议外交易包含安排的可能性，这在过去一直是个问题。

尽管移除费用销毁会略微提高 SOL 的净通胀率，但通过质押奖励产生的新代币发行产生了更显著的影响。读者可以参考我们早期的 Helius 博客文章，了解有关 [Solana 的发行和通胀计划](https://www.helius.dev/blog/solana-issuance-inflation-schedule)的更详细分析。

## 分区纪元奖励

[分区纪元奖励](https://github.com/anza-xyz/agave/issues/426) 旨在将 [质押奖励](https://docs.solanalabs.com/consensus/stake-delegation-and-rewards#basics) 分配到多个区块中，缓解了将奖励分配集中在每个新纪元第一个区块所带来的性能问题。这个过程中的主要瓶颈是需要将更新写回网络上不断增长的活跃质押账户数量，目前总计约 140 万个。

在这种新方法下，纪元边界的质押奖励计算和分配将分为两个不同的阶段：

- **奖励计算阶段：** 在此阶段，计算所有活跃质押账户的纪元奖励，并将分配分成预定的块。
- **奖励分配阶段：** 按照预先计算的方式分配活跃质押账户的纪元奖励。

为了促进和监控这个过程，一个 Sysvar 账户 [**EpochRewards**](https://github.com/anza-xyz/agave/pull/2102) 将在整个分配阶段跟踪和验证奖励分配。EpochRewards Sysvar 记录奖励分配阶段是否正在进行，以及从快照开始时恢复分配所需的信息。

### 奖励计算

奖励计算将在纪元的第一个区块执行。一旦计算完成，奖励会被分割成分配块存储在银行中，这些奖励将在奖励分配阶段进行分发。

为了最小化奖励分配阶段对区块处理时间的影响，并确保每个区块以确定性的方式分配奖励的子集，每个区块将分配 4,096 个质押奖励作为目标。为了防止质押账户数量的剧烈增长，区块数量上限被设定为一个纪元总槽位数的 10%。只有在达到这个区块上限时，每个分区的账户数才允许超过 4,096 的目标值。
