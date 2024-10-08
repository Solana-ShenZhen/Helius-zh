# Agave 2.0 过渡：开发者指南

## 简介

Agave v2.0 计划在 [Solana Breakpoint 2024 新加坡](https://solana.com/breakpoint) 之后的几周内在 Solana 主网测试版上 [首次亮相](https://github.com/anza-xyz/agave/wiki/v2.0-Release-Schedule)。这次重大更新包含了多个破坏性变更、API 端点移除和 crate 重命名，所有 Solana 开发者都应该了解这些变化。除了本文外，我们建议开发者查看 Anza 最近在 GitHub 上发布的 [过渡指南](https://github.com/anza-xyz/agave/wiki/Agave-v2.0-Transition-Guide)。

Agave v2.0 目前已在 Solana 的 Testnet 和 Devnet 集群上线。这次发布标志着 Solana 生态系统演进和向多客户端网络过渡的重要里程碑。Anza 团队预计将在正式发布前发布 Agave 2.0 的全面深入解析。本文将仅限于讨论过渡准备的实际方面。

## 移除的 RPC 端点和 SDK 调用

多个过时和已弃用的 v1 Agave RPC 端点将被移除。具体包括：

**getRecentBlockhash**、**getConfirmedSignatureForAddresses2**、**getConfirmedTransaction**、**getConfirmedBlock**、**getStakeActivation**、**getFees**、**confirmTransaction**、**getSignatureStatus**、**getSignatureConfirmation**、**getTotalSupply**、**getConfirmedSignaturesForAddress**、**getConfirmedBlocks**、**getConfirmedBlocksWithLimit**、**getFeeCalculatorForBlockhash**、**getFeeRateGovernor**、**getSnapshotSlot**

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66cec83a1a96f4775c32312e_AD_4nXdxz0cIGoVEv-O8WlwJr2U8t-7jZPT0OseawvWW57Yg9cFI84CKw3Obnd4LzAoniyFHZHwroM9luvgFc7indGq3P1vWKbJJQxi8RQV8ZQZtwol_Og-u83L5nqipcuaP-DhttKHrzm6Kd6Ka4q6zDOuy9wYe.png" loading="lazy" alt="">

根据 Helius 内部分析，我们认为大多数客户不会受到这些变更的影响。然而，仍有一小部分用户在积极使用以下端点：

**getRecentBlockhash**、**getConfirmedSignatureForAddresses2**、**getConfirmedTransaction**、**getConfirmedBlock**、**getStakeActivation**、**getFees**

**我们强烈建议所有开发者在 2.0 主网测试版客户端发布之前，检查这些调用的引用，并使用建议的替代方案进行适当更新。**

图中所示的 getAccountInfo 替代方法可以[在这里找到](https://solana.stackexchange.com/questions/15710/the-alternative-method-to-get-the-stake-account-status-since-getstakeactivation)。

SDK 破坏性变更包括：

- 移除了对 Borsh v0.9 的支持，请使用 v1 或 v0.10（[#1440](https://github.com/anza-xyz/agave/pull/1440)）
- Rent 和 EpochSchedule 不再[派生 Copy trait](https://github.com/solana-labs/solana/pull/32767)；请改用 **clone()**
- solana-sdk：移除了[已弃用的符号](https://github.com/anza-xyz/agave/releases/tag/v2.0.2#:~:text=v2.0%3A Remove deprecated,%231962)
- solana-program：移除了已弃用的符号

完整的破坏性变更列表可以在 [Agave 更新日志](https://github.com/anza-xyz/agave/blob/v2.0/CHANGELOG.md#200)中找到。

## 重命名的 Crates

为了适应多个 Solana 验证器客户端的引入（如由不同团队管理的 [Firedancer](https://www.helius.dev/blog/what-is-firedancer)），几个 crates 正在被重命名。通过在这些 crate 名称后附加"Agave"，这些 crates 被标识为专门由 [Anza](https://www.anza.xyz/) 为 Agave 验证器客户端维护的依赖项。这一变更确保了在区分 Agave 相关依赖项和其他团队管理的依赖项时有更大的清晰度。受影响的 crates 包括：

**solana-validator**、**solana-ledger-tool**、**solana-watchtower**、**solana-install**、**solana-geyser-plugin-interface**、**solana-cargo-registry**

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66cec83bb17ca6be7d3c312f_AD_4nXdsR9MQUNDpsL_JQs4sjefy6YNVG_XntMmjwWl7EhtXHEm47QQAZlS51Aakv1kNmW3EY6si2oLJ9DK3ShIbQy6kACKhdH4gZvhcHgA5pDeF1XWdCVFm4SkJiT6MmtPJRb28fIjpu4ZJ-bFvC_y02gpCiSg.png" loading="lazy" alt="">

开发者应检查任何自动化或脚本中对这些 crates 的引用，并适当更新它们。

## 移除的验证者节点参数

对于验证者节点运营商，以下已弃用的验证者节点参数将在 Agave v2.0 发布时被移除：

**--enable-rpc-obsolete_v1_7** ([#1886](https://github.com/anza-xyz/agave/pull/1886))

**--accounts-db-caching-enabled** ([#2063](https://github.com/anza-xyz/agave/pull/2063))

**--accounts-db-index-hashing** ([#2063](https://github.com/anza-xyz/agave/pull/2063))

**--no-accounts-db-index-hashing** ([#2063](https://github.com/anza-xyz/agave/pull/2063))

**--incremental-snapshots** ([#2148](https://github.com/anza-xyz/agave/pull/2148))

**--halt-on-known-validators-accounts-hash-mismatch** ([#2157](https://github.com/anza-xyz/agave/pull/2157))

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66cec83b0697e87b5c3fb540_AD_4nXdyWXxYadffu2rXZwNloLvC6SdFElMfToqtTdr0qsRUL9b0U8SkDAXh_6IJknqf7XVHyWGIHsnC1xdEXroavO0tiAO_2AMFfZEqFmfTTcUcEJKv4vgd1_A-TV4TzuoyAC0qEVIyp9Mzw994Dp7hU0EB2bmk.png" loading="lazy" alt="">

再次强调，我们建议验证者节点运营商仔细确保不再使用这些参数。

最后但同样重要的是，原始的 [Solana Labs GitHub](https://github.com/solana-labs/solana) 仓库将在 Agave v2.0 发布时被归档。开发者应将所有活动迁移到 [Anza Agave GitHub](https://github.com/anza-xyz/agave) 仓库。

最后，我们引用 Solana DevRel 团队的 [Nick Frostbutter](https://x.com/nickfrosty) 在[最近的更新日志视频](https://youtu.be/E480E-OUT5o?si=yKr2bMJfayL4kybu&t=27)中的话作为结束，**"如果你正在使用任何有弃用警告的内容，你需要确保升级你的代码。"**

以上就是这个简短的公告。希望在 Breakpoint 大会上见到你们中的许多人！
