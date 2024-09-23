# Solana 基金会委托计划（SFDP）以及长尾验证者面临的挑战

原文[链接](https://www.helius.dev/blog/solana-foundation-delegation-program-sfdp)

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ea18e2a7986d80380e_AD_4nXe7evDsZY-Q10PZJIIYKBkoSrMyPEMvU4Lhc13bzvLTlSfD6wsUciifz6EDC2Tb6eLuzx4ieSPE1apUcWe9k_CqxtF8S2pE8oukG_Ehmv8BYvid7M0DsuxKzLguQBIFQB8xgOBvRKOlHd0xO7Nq4mYPr9M.png" loading="lazy" alt="">

衷心感谢 Shinobi Systems 的 [Zantetsu](https://x.com/ShinobiSystems) 、Laine 的[Michael](https://x.com/michaelh_laine?lang=en) | Stakewiz 和 [Ben Hawkins](https://x.com/B3nHawkins) 审阅了这项工作的早期版本。

## 名词解释

- **纪元（epoch）**：在区块链网络中，纪元是一个时间段，通常用于定义验证者节点的工作周期。在每个纪元结束时，网络会重新分配质押和奖励，并更新验证者节点的状态。纪元的长度和定义可能因区块链协议的不同而有所差异。在 Solana 中，纪元是由时隙（slot）组成的，平均一个纪元是 2 天，这个时间内会保持一个有效的领导者调度表（leader schedule）。
- **超级少数（Superminority）**：在区块链网络中，超级少数是指控制网络中超过三分之一（33.3%）质押量的验证者集合。由于共识机制要求超过三分之二（66.7%）的验证者同意才能确认交易，因此超级少数验证者可以阻止网络达成共识，冻结网络的运行。超级少数的存在对网络的去中心化和安全性构成潜在威胁，因为它们集中控制了大量的质押权重。
- **超级多数（Supermajority）**：在区块链网络中，超级多数是指控制网络中超过三分之二（66.7%）质押量的验证者集合。由于共识机制要求超过三分之二的验证者同意才能确认交易，因此超级多数验证者可以控制网络的共识过程，决定哪些交易被确认。超级多数的存在对网络的去中心化和安全性构成潜在威胁，因为它们集中控制了大量的质押权重。
- **ISP（Internet Service Provider，互联网服务提供商）**：ISP 是指向用户提供互联网接入服务的公司或组织。ISP 提供的服务包括互联网连接、域名注册、虚拟主机、电子邮件服务等。对于运行 Solana 验证者节点的运营商来说，选择合适的 ISP 是至关重要的，因为验证者节点需要稳定、高速的互联网连接来确保网络的正常运行和数据的及时传输。常见的 ISP 包括 Teraswitch、Constant、Interserver 和 OVH 等。

## 可行的见解

- 运行 Solana 验证者节点是无需许可的。没有强制性的最低质押数量要求。包括 Solana 基金会在内的任何一个实体都无法控制谁在网络上运行。
- 验证者节点赚取以 SOL 计价的收入，该收入与他们的质押权重直接相关。相反，许多运营成本是固定的并以法定方式支付。这些动态不利于新的低质押权重的长尾验证者。
- SFDP 在质押中授权 5100 万个 sol，占 Solana [总质押](https://dune.com/queries/3229774/5401861)（3.85 亿）的 13%。这比 2020 年 12 月该计划启动时的 1 亿有所下降。
- SFDP 的活动高度透明，提供详细的授权[标准](https://solana.org/delegation-criteria)、精细的[仪表板](https://solana.org/validators-search?o=&d=&q=&f=&flagged=)信息、自动[纪元更新](https://discord.gg/solana)和公共支持渠道。所有委托均通过一个公开且带有标签的[帐户](https://solscan.io/account/mpa4abUkjQoAvPzREkh5Mo75hZhPFQ2FSH6w7dWKuQ5#stakeAccounts)进行管理。
- [72%](https://docs.google.com/spreadsheets/d/1f_aczNyRHbHDBFEn5yF2SPSXYmYNZruttDfJAT1yFNA/edit?usp=sharing)的 Solana 验证者节点获得 SFDP 的支持，占 19% 的质押。不依赖 SFDP 的 420 名验证者节点控制着 81% 的质押。
- 由于外部质押有 100 万 SOL 的硬性上限，无论是三分之一的超级少数还是三分之二的超级多数，验证者节点都没有资格获得 SFDP 支持。
- SFDP 委托质押分为 2 类：质押匹配（最多 100k SOL）和剩余委托（约 30k SOL）。该计划第一年的投票费用也会帮忙部分承担。平均授权数为 45,652 SOL。
- [73%](https://docs.google.com/spreadsheets/d/1f_aczNyRHbHDBFEn5yF2SPSXYmYNZruttDfJAT1yFNA/edit?usp=sharing) 的 SFDP 计划参与者从计划之外吸引了不到 10,000 质押量，51% 的参与者成功吸引了不到 1,000 质押量。
- 我们估计，如果 SFDP 立即终止，大约 897 名该计划参与者（占所有 Solana 验证者的 57%）将难以维持盈利运营。
- 为了占领市场份额，长尾独立验证者必须竞争链上的质押量，并积极管理的、对 APY 率等因素做出反应，或者从少数有竞争力的大型质押池中安全分配。
- 在多种情况下，运行验证器无利可图是有意义的，特别是如果此类操作转化为其他业务线的优势。对于运营 dApp、钱包或基础设施的生态系统团队来说尤其如此。

## 介绍

今年 6 月， [Solana 基金会委托计划](https://solana.org/delegation-program)(SFDP) 因多名参与者被强制退出该计划而受到广泛关注。这些验证者被发现参与了私人内存池，这种行为明显违反了该计划的条款。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eb875e91095b340aed_AD_4nXdIteAvEIGl6M-AmqIEWZxBlrn_Ce8bTPALZho9Q14lRSbNnSNozfp6zEh5bRPK5NY-7pWGSuOuekJ_33RVb6NqaTUS6TwKMGiVw8p1ob5et_0jF_tQv9YjoKZI1Jg1S-x_5S925xMdZ-V-GTy8WpsPscRP.png" loading="lazy" alt="">

<div style="text-align:center">上图：SFDP 6 月份宣布删除运营商</div>

这一行为[得到了](https://www.theblock.co/post/299244/solana-foundation-removes-certain-operators-from-delegation-program-over-malicious-sandwich-attacks)行业媒体的[广泛](https://solanafloor.com/news/solana-foundation-defends-users-against-sandwich-attacks-removes-malicious-validators-from-delegation-program)的[报道](https://cointelegraph.com/news/solana-removes-validators-sandwich-attacks)，随后，大量错误的批评和彻头彻尾的错误信息在网上传播。这一事件暴露了对 SFDP 缺乏正式分析以及该计划对 Solana 验证者环境的积极和消极影响——这是我们希望通过这项工作解决的问题。

首先，我们将介绍 Solana 质押的、出块的验证者节点的经济学，因为扎实掌握验证者经济学是后续分析的关键前提。已经熟悉最新进展（包括[SIMD-96](https://github.com/solana-foundation/solana-improvement-documents/pull/96) ）的读者可能希望跳过本节。其次，我们提供了 SFDP 的完整分解，通过示例和简单分析剖析了该计划的关键组成部分。然后，我们将具体研究 Solana 的质押分配以及 SFDP 对这种分配的影响。根据验证者社区的意见，我们最后讨论了低风险长尾验证者在建立可行、长期、可持续业务方面的可行性。

免责声明：Solana 前进很快！这项工作应该被视为时间的快照。引用的许多值每天都在变化，尤其是 SOL 的价格。请参阅链接的数据源以获取最新值。

## Solana 验证者经济学

运行 Solana 验证者节点是无需许可的。具有适当技术知识、访问开放互联网和足够财务资源的任何人或任何组织都可以[操作 Solana 验证者节点](https://www.helius.dev/blog/how-to-set-up-a-solana-validator)（如果他们愿意）。没有强制性的最低质押要求。撇开外部因素不谈，可以合理地假设，理性的个人和组织只有在预期获得正的经济回报（收入超过成本）时才会投入时间和资源来运行验证者节点。为了进一步理解这一点，我们首先考虑一下相关费用。

### 开支

运行 Solana 验证者节点所涉及的费用可分为三个主要部分：硬件、运营成本以及资本和劳动力的机会成本。

#### 硬件（固定成本，以法定货币计价）

Solana 比大多数行业同行网络具有更高的节点要求，并[为 Solana 验证者节点提供了完整的官方硬件推荐](https://docs.solanalabs.com/operations/requirements)，包括以下关键组件：

- **CPU** ：12 核/24 线程或更多，2.8GHz 基本时钟速度或更快
- **内存**：256 GB 或以上
- **磁盘**：PCIe Gen3 x4 NVME SSD，或更好，组合容量为 2 TB 或更大。高总重量
- **无 GPU 要求**

硬件是最容易衡量的验证者节点成本之一，但即使在这里，也存在很多细微差别。运营商有两个主要选择：直接购买硬件并在设备的使用寿命内分摊此成本，或者从提供裸机专用服务器的数据中心租用硬件。大多数验证器运营商选择后者，因为它具有灵活性，而且由于大量的带宽要求，家庭操作将需要专用的互联网线路。

满足这些要求的一个经常被提及的更便宜的选择是 Latitude 的裸机服务器产品，其[起价](https://www.latitude.sh/pricing#metal)为每月 350 美元。该定价处于运营商通常支付价格的低端。成本因地理区域、服务提供商和具体硬件而异。被 100 个或更多验证者选择的热门 ISP 包括[Teraswitch](https://teraswitch.com/bare-metal/) 、 [Constant](https://www.constant.com/products/bare-metal/) 、 [Interserver](https://www.interserver.net/dedicated/)和[OVH](https://www.ovhcloud.com/en/bare-metal/) 。可以[在此处](https://app.marinade.finance/network/isps/?direction=descending&sorting=stake)找到详细信息。

Solana 基金会长期运行的[服务器计划](https://solana.org/server-program)(SFSP) 旨在降低新验证者运营商的进入门槛，通过开箱即用的解决方案快速加入网络。该计划本质上是一份批量购买协议，基金会不从中抽取任何费用。 SFSP 目前为 Edgevana 和 Equinix 提供灵活的租赁合同选项，[价格](https://srv.edgevana.com/solana-validator-servers)在每月 499 美元至 599 美元之间。设置最快一天即可完成，并且需要 KYC。

#### **投票费**（固定成本，以 SOL 计价）

_每个纪元的投票费用 = 0.000005 SOL \* 投票数_

运营成本分为两种：链上投票费用和数据带宽成本。链上投票费用是运行验证者节点的主要成本。之所以会出现这种情况，是因为验证者非常有动力对他们认为应该规范的区块进行投票，并且这些共识投票被视为交易。投票交易统一定价为 0.000005 SOL。它们是 [banking 阶段内由专用通道处理的](https://www.helius.dev/blog/all-you-need-to-know-about-solanas-v1-18-update#the-new-transaction-scheduler)特权交易，因此不需要支付优先费用。每个纪元由 432,000 个时隙（slots）组成，但并非每个时隙都有关联的区块。由于无法投票的[跳过时隙](https://solana.com/docs/terminology#skip-rate)，实际数字会略低。根据网络条件，跳过时隙率通常在 2% 到 6% 之间变化。自从今年早些时候在主网测试版上引入及时[投票](https://forum.solana.com/t/proposal-for-enabling-the-timely-vote-credits-mechanism-on-solana-mainnet/1179)积分以来，验证者现在可以通过一次投票为多个区块投票。这将平均投票费用降低至每周期 1.4 SOL（约 2 天）。

#### **带宽**（可变成本，以法定货币计价）

验证者之间的带宽要求各不相同，因为那些拥有较高的验质押证者将更频繁地被选为领导者（leader）。领导者在其分配的时隙内会经历活动的大幅高峰。当整个网络将数据包定向到入站流量时，入站流量可以达到[每秒 1 GB](https://open.spotify.com/episode/49DbzAxITQUUc42dGD9hq3)以上。因此，验证者带宽建议至少为 1 GB，建议为 10。典型的验证者每月可以轻松处理[100 TB](https://apfikunmi.medium.com/running-a-solana-validator-a95cdfd6488a)的数据，[其他验证者](https://discord.com/channels/428295358100013066/1187805174803210341/1267977719354888263)报告仅数据出口就接近 150 TB。

数据中心对带宽的收费方式差异很大。 Latitude 提供免费入口和 20 GB 免费出口，之后带宽将每 TB 花费[0.64 至 3.60 美元](https://www.latitude.sh/network/pricing)，具体取决于所在地区。

#### **机会成本**（固定成本，以法定货币计价）

机会成本有两部分，一是资本，二是劳动力。资本的机会成本是持有美元短期[国债](https://home.treasury.gov/resource-center/data-chart-center/interest-rates/TextView?type=daily_treasury_bill_rates&field_tdr_date_value=2024)的回报，目前为 5.25-5.5%。如果我们选择不投资验证器运营，这就是我们期望获得的回报。劳动力的机会成本是有资格操作验证者的人员的市场工资，验证者是一项要求很高的技术工作。运营商必须牢牢掌握区块链架构、硬件、后端开发和 DevOps。拖欠行为会受到惩罚，这意味着验证节点应该在 100% 的时间内处于活动状态；它们需要持续监控和定期软件更新。这些人的公平市场工资根据地理位置差异很大。

<div style="text-align: center;">
  <img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eadc1e813f5ed73560_AD_4nXenx7i340pBJb8gTpCkwE4RmAGIyd1prA_aCIPlznFYgLBSfIZCP14S4VfL31apvcUP0dnzwFIhAaxHqIONz25AdE5Imyg_MQAUi7rOfUOGF8ofjHxF4Q2rEONih4IwDe7mYYr7k46SosdOQItrvpvIxTg1.png" alt="">
</div>
<div style="text-align:center">上图：SFDP 6 月份宣布删除运营商</div>

### 收入

验证者有多种收入来源，包括区块奖励、佣金和 MEV。

#### **质押佣金**（可变成本，以 SOL 计价）

_质押佣金 =（委托质押奖励\*佣金率）_

Solana 通过根据[通胀时间表](https://solana.com/docs/economics/inflation/inflation-schedule)生成新的 SOL 代币来分配每个时期的质押奖励。质押奖励首先根据该纪元获得的积分进行分配。对后来最终确定并成为规范的区块进行正确投票将为验证者赢得一个积分（这可能会随着[及时投票积分](https://forum.solana.com/t/proposal-for-enabling-the-timely-vote-credits-mechanism-on-solana-mainnet/1179)的引入而改变）。

验证者在总积分中所占的份额（即，他们的积分除以所有验证者积分的总和）决定了他们的奖励比例。这进一步按质押加权。如果拥有平均积分数，拥有总质押 1% 的验证者应该赚取大约 1% 的总通胀率。如果超过平均积分数，奖励也会相应波动。投票积分是验证者在共识过程中的参与度和正确性的定量衡量标准。

质押奖励分配给委托人而不是验证者节点。但是，验证者可以为其服务收取佣金，该佣金是委托者通胀奖励的一定百分比。该佣金通常是个位数百分比，但技术上可以是 0% 到 100% 之间的任何整数。

注意：有超过 200 个“私人” Solana 验证者，其质押大概由运营实体全资拥有并委托给他们自己。这些完全自我质押的 SOL 验证者将佣金设置为 100%。

#### **区块奖励**（可变成本，以 SOL 计价）

_每个最终生成的区块的区块奖励（SIMD-96 后）=（基本交易费用 \* 0.5）+ 优先费用_

被指定为特定区块领导者的验证者将获得额外的区块奖励。只有生成该块的验证者才能收到这些奖励。当区块产生时，区块奖励立即记入验证者的身份账户。

此前，这些奖励包括区块内所有交易的 50% [基本费用](https://solana.com/docs/core/fees#transaction-fees)和 50% 的[优先费用](https://solana.com/docs/core/fees#prioritization-fees)，其余费用被烧毁。然而，随着 SIMD-96 的通过，这种结构将发生变化， [SIMD-96](https://github.com/solana-foundation/solana-improvement-documents/pull/96) 应该在 2024 年 Breakpoint 会议之后实施，并按照当前的功能门激活[时间表](https://github.com/anza-xyz/agave/wiki/Feature-Gate-Activation-Schedule)使用 Agave 2.0。展望未来，100% 的优先费将流向区块生产者。这一变化消除了区块生产者参与协议外附带交易的动力。

#### **MEV** （可变成本，以 SOL 计价）

_MEV 佣金 =（MEV 总额 \* MEV 佣金率）_

[最大可提取价值（MEV）](https://www.helius.dev/blog/solana-mev-an-introduction)是指区块生产者在其生产的区块内任意包含、排除或重新排序交易所能获得的利润。 Solana 不强制验证器如何对区块内的交易进行排序，这为协议外 MEV 机制提供了广泛的范围。

超过 80% 的质押运营着 [Jito 客户端验证者节点](https://docs.google.com/document/d/1PNSpqR-bmQpRp-Vq5wmy-wwp9g8Ni4ikoWmRYnWO-CY/edit#heading=h.wiz900r6t3t6)，这是原始 Agave 客户端的一个分支，引入了协议外的区块空间拍卖，为验证者提供了额外的小费收入流。区块空间拍卖通过 Jito 区块引擎（Jito Block Engine）在链外进行，允许搜索者（searchers）和应用程序提交原子执行的交易组（称为捆绑包，bundles）。这些捆绑包通常包含对时间敏感的交易，例如套利或清算。此前，Jito 运营着规范的协议外内存池服务，但现已弃用。

Jito 对所有小费收取 5% 的费用，最低小费为 10,000 lamport。小费的运作完全是协议之外，与优先费和基本费分开。它们是通过八个静态分配的帐户地址从搜索者那里收集的，并由[小费分发计划](https://jito-foundation.gitbook.io/mev/mev-payment-and-distribution/tip-distribution-program#tip-distribution)在每个纪元结束时分发。

2024 年，Jito MEV 已从不起眼的收入，增长为验证者的[重要](https://explorer.jito.wtf/api/getTipsV4?period=Month)收入来源。该收入按质押分配并收取佣金，与通货膨胀奖励的分配方式相匹配。 MEV 佣金是委托人 MEV 奖励的百分比，范围从 0% 到 100%。该百分比的设置与验证者通胀奖励佣金率无关。

Solana 的 MEV 格局正在迅速发展。如今，一些验证者继续参与私人内存池，从对用户的三明治攻击中获利，尽管此类活动带来了社会耻辱和对用户体验的负面影响。我们预计将继续开发旨在阻止此类做法的机制，包括[基于声誉的](https://x.com/trustless_matt/status/1823533692154773614) QoS、完全链上[评分系统](https://www.jito.network/blog/a-deep-dive-into-stakenet/)以及捕获“良好”MEV 的新[验证者客户端](https://medium.com/@uri_61495/introducing-paladin-9ec10ac49a31)。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ec16b516bdc251df93_AD_4nXfrBMFg9iDt9IYlQT8F4cWRhaKVRXQOeamne1qLXlI8Bje070lBOdX5WiqHCdu6N6Yk7cvRXYjUkghNaWlZoqfl-TzBZrJKj_hRjkUpFAx7FDve2odoLFvR0kfU0qf2-vDHeQj0VDmA038kom-FkslSn_pE.png" loading="lazy" alt="">

<div style="text-align: center;">
  上图：2024 年 Jito 每日提示的增长（<a href="https://dune.com/queries/3034383/5044505" target="_blank">仪表板</a>）
</div>

MEV 是一个复杂的话题。请阅读我们之前的 Helius 博客[文章](https://www.helius.dev/blog/solana-mev-an-introduction)，了解有关 Jito 和 Solana MEV 的更多信息。

#### **其他的**

自从部分采用[质押加权服务质量](https://www.helius.dev/blog/stake-weighted-quality-of-service-everything-you-need-to-know)(SWQoS) 以来，验证者可以自由签订协议，将其质押加权容量出租给 RPC 节点。作为回报，RPC 节点获得了更多的带宽，使它们能够实现更高的块包含率。交易带宽拍卖[市场](https://www.triton.one/cascade-marketplace/)有助于促进寻求购买带宽的项目和寻求出售多余带宽的验证器运营商之间的协议。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ea241e3c15ea0c7827_AD_4nXcGhLWWtDpzjpfo8qg8-ybvDMIh3HHafZKiut55aJi6lmb_KHeXhjCHmYyH2J4IYtDUuMtd6ANvNLR7nVx-wR7enjcWaAK-2qZfErYBe35Yx212gmCYjOJK1ycHvpuw-X-3lTJnHc-6s2Vx6An8dzDK-nw4.png" loading="lazy" alt="">

<div style="text-align:center">上图：所有验证者运营收入来源都根据权益而变化。不包括“其他的”。</div>

### **间接动机**

在分析了运行验证者节点的所有直接成本和收入流之后，是时候考虑更多的间接动机了。

运营验证者节点可能具有正外部性或转化为其他业务线的优势。 Helius 通过运行验证者节点来证明这一点，该验证者节点与我们的核心 RPC 业务协同作用，增强客户交易包容性。自从采用 SWQoS 以来，所有流行的生态系统应用程序团队都有额外的动力来运行验证者节点，以确保交易服务质量。

此外，运行验证者节点标志着对生态系统的良性承诺，这可以对行业同行产生积极影响。质押比例非常高的验证者也赋予其运营商一定程度的声望。相反，运行低质押的验证者节点可能会让新的、不太成熟的团队受益，培养信任并充当“技术能力的证明”。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eb41f8fb3aa67cd5f2_AD_4nXeFq9rGu2SLrY4-TA_3AhOUX92j2XjNNlPM4XqWX52Q4ImlZFXQM9zDCAWeVhDuavPuNRK3E4ojOAj-U-tfLytUjX3R8DlVzyrtfqN1qgzGnplhtV-xu9tI_5ktP_7WuiLxg1rizRurVpbFBzu_sVXoKN7E.png" loading="lazy" alt="">

中心化交易所 (CEX) 是另一个需要考虑的案例，因为有几个交易所跻身质押比例最高的 Solana 验证者之列。对于他们来说，Solana 质押代表了更大产品矩阵中相对较小的单一业务线。规模较小的交易所可能仍希望运营无利可图的质押服务，以维持完整的产品供应。 CEX 质押服务部分不受竞争影响——零售用户处于链外，寻求便利，并且对低于平均水平的质押回报相对不敏感。这使得 CEX 在希望传递给用户的回报方面具有很大的灵活性。

另一个值得考虑的类别是机构（institutional）质押解决方案提供商。这些企业提供跨多个链的质押服务，其中一些服务严格来说可能无法单独盈利，但作为公司更广泛战略的一部分进行分析时是有意义的。

这里要讲的关键点是，**在多种情况下，运行无利可图的验证者节点仍然有意义**。此外，SOL 的价格（决定以法定货币计价的盈利能力的关键变量）会受到加密货币市场波动的影响。 2022 年 12 月，FTX 崩盘后 SOL 价格为 15 美元，[估计](https://medium.com/@ultimateapp/solana-on-its-own-two-legs-5969bc0fc30b)只有 183 个 Solana 验证者实现盈利。价格波动使可靠的预测变得复杂，但会提高对亏损周期的接收程度，特别是对于对生态系统长期前景充满信心的运营商而言。

### 小结

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eb049486c1469f4aa0_AD_4nXfxuZO-kb4fCKQY8pyNFXoseZayJGdP4adFxcBNQDpTNkEO14NHHHjbdlT6SfQeFF0tcNrKRXnwyVOT6L1hzi4aRVUmsBojCUG0vBumNDFzkYWLM2IkLnKFbcIMdMQdnnmyq3N7tLMrar5pTaH_jZF_Iac.png" loading="lazy" alt="">

$成本 = 硬件(固定，法币) + 宽带(浮动，法币) + 投票(固定，SOL) + 机会成本(固定，法币)$

$收入 = 质押佣金(可变，SOL) + 区块奖励(可变，SOL) + MEV佣金(可变，SOL)$

所有收入均以 SOL 形式赚取，并直接与质押水平成比例。许多成本是固定的并以法币方式支付。

要更深入地探讨这一重要主题，请参阅我们之前的 Helius 博客文章[Solana 验证者经济学：入门](https://www.helius.dev/blog/solana-validator-economics-a-primer)。

## Solana 基金会委托计划 (SFDP)

_“从长远来看，我认为 SFDP 应该是一个看起来更像孵化器的系统，我们可以在开始时帮助降低风险，但不能无限期地维持下去。我认为，基金会在保持健康的验证者数量方面越不重要越好。” - Ben Hawkins，Solana 基金会 Stake 生态系统主管（_[来源](https://youtu.be/59uY4z9u4-A?si=UtESlcfolv90-MaW&t=2764)_）_

### 计划起源

Solana 早期并没有 SFDP。大多数主网 Beta 验证者直接从 Solana 基金会获得高达 550,000 SOL 的委托——这个值的选择有些随意。

SFDP 于 2020 年 12 月正式[宣布](https://medium.com/solana-labs/announcing-the-solana-foundation-delegation-strategy-5bcccf9104ab)。部署最初是通过一个脚本实现自动化的，该脚本动态且均匀地划分 1 亿个 SOL 池，以最大化质押总占比在占三分之一头部的超级少数（superminority）节点数量，避免少数验证者掌握过多的控制权。超级少数是一个关键的门槛，代表最小的验证者群体，如果他们串通一气，将持有足够的质押来破坏共识过程、审查区块甚至停止区块链——这是一种理论上可能但实际上不太可能的情况，因为这些实体处于因网络中断而遭受最大的经济损失。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ec0689ffce0adcebd8_AD_4nXeRVP22xbckUoQNCcID6gLp-qxAn5LwF8-JvhlRBqI9IBSpENdpdmm1RVuZPJYdGm0mCEDKatT5m7Uq9naS6qD6n7EVsjsl5P6k1mr6Dn21ky-2RhNUBDORkYyvEuyDAzWyO0U8Pf0CWr5AgyXQ6w2gQc8.png" loading="lazy" alt="原始SFDP策略的理论分布图" title="基于原始SFDP策略的理论分布">

<figure>
  <figcaption style="text-align:center">图：基于原始SFDP策略的理论分布</figcaption>
</figure>

这种原始的 SFDP 设置积极激励超级少数中的运营商将其质押分配给多个验证者，从而在超级少数之外创建新节点，并自动获得计划委托资格。与现行标准相比，最初的计划要求相对宽松，且委托池规模更大，为 1 亿 SOL。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ec8328010b549a7b1a_AD_4nXcTEWjuJmOdw8Vx3riXGQMLL7AAK0vIO2Hds95W5AiiTPGUEV90P5NuP7n47yc8_eszmZsDtSpVq5lLf1uPlh052Nc2yOZM9x688FKJFVXv8jAtS8rJE9O7-rDsr7QF7C7LcNtYxhnPCK85sbTa4F_R7ZVb.png" loading="lazy" alt="">

<div style="text-align:center">上图：在最初的 SFDP 分配机制下，超级少数边界附近的代表团逐渐减少，以防止出现可能引入边缘情况激励（edge-case incentives）或经济攻击向量（economic attack vectors）的情况。
</div>

这个版本的 SFDP 充当了一个生硬的自动化工具，有助于在低质押验证者的长尾中均匀分配质押。

当前的 SFDP 于 [2024 年 4 月](https://solana.com/news/solana-foundation-delegation-program-updates)开始实施。更新后的计划[提出了三个目标](https://solana.org/delegation-program)：

- 最大限度地提高 Solana 的去中心化、可靠性和性能
- 最大化拥有不同质押来源的验证者数量
- 维持一个大型且具有代表性的测试网

### 测试网

我们将首先检查 Testnet，它是 Solana 的四个标准集群之一 - Localnet、Testnet、Devnet 和 Mainnet-Beta。其目的是对即将发布的功能进行压力测试，同时监控实时集群上的网络性能、稳定性和验证者节点行为。 Localnet 不应与 Devnet 混淆，后者是开发人员的游乐场，允许安全、零成本的程序部署和应用测试。活跃的大型测试网对于 Solana 至关重要，但所有测试网代币本质上都是毫无价值的，因此必须找到替代方法来激励和补偿验证者。因此，测试网构成了 SFDP 的重要组成部分。

加入 Testnet SFDP 是参与 Mainnet-Beta 计划的先决条件，因为所有参与者都必须被 Testnet 计划接受并具有良好的信誉。通过这种方式，SFDP 可以审查运营商，运营商必须通过满足一组[测试网要求](https://solana.org/delegation-criteria)来证明其运营能力。此外，测试网计划还有一小部分奖励付款——每月 250 美元，最多六个月——部分抵消了节点运营的固定硬件成本，并且取决于是否满足性能要求。

### 计划上线

虽然测试网计划可以立即访问，但由于需求量很大，加入主网测试版上的核心 SFDP 需要等待至少一年。目前，每周通过[标准优先队列](https://solana.org/faq#onboarding_questions)添加到主网计划的新参与者数量只有低个位数。此外，还有一个名为“生态系统贡献者优先队列（Ecosystem Contributor Priority Queue，ECPQ）”的替代上线途径，专为开发人员和生态系统贡献者设计。这种“工作量证明”抗 Sybil 队列允许那些能够对 Solana 做出有意义贡献的人快速加入，经过验证的生态系统贡献者将在[一到两周](https://youtu.be/59uY4z9u4-A?si=VGabzCNpfXqwOL8c&t=2703)内加入。对于什么构成“有意义的贡献”，没有提供正式的标准。

### 质押委托

SFDP 委托质押分为两类：质押匹配（stake matching）和剩余委托（residual delegation）。通过质押匹配，SFDP 将一对一匹配参与者的非计划质押，上限为 100,000 SOL。凭借剩余质押，指定用于该计划但尚未通过匹配分配的 SOL 将平均分配给所有符合条件的验证者。目前，每个验证者的剩余质押[约为](https://solana.org/sfdp-validators/simpRo1FrQYGa1moicfgnPDp6KyE38d4gYrZzhjXYJb)30,000 SOL。目前该计划的最大质押门槛为[100 万 SOL](https://forum.solana.com/t/upcoming-sfdp-changes/772/9) 。持有超过此金额的非 SFDP 股份的参与者不再有资格参加该计划。以下是基于不同数量的非 SFDP 股份的一些场景：

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eb735501ed8f7269ad_AD_4nXclcgrKWUlCCcdrxK1xRSLRpjZ0kR7NA1TeticTjeNe-lHu0UNYv4-LU1VY93B7_pQm592gCIz78OwFsOYACOuj8FkgzCfb935CCEMWv9kyyGKYmRFtA37F9pMPq96LsPndH6mTOhZqaGUVEatR4n9Zqnvg.png" loading="lazy" alt="">

SFDP 极大地有利于低质押验证者。只有当验证者超过 130k 非 SFDP SOL 时，SFDP 部分才会开始占其质押的一半以下。此外，该计划的硬截止边界意味着拥有 110 万个非 SFDP 质押 SOL 的验证者的总体质押将少于那些仍从该计划中受益的不到 100 万个的验证者。毫不奇怪，一些人呼吁采取更温和的逐渐减少方法，而不是单一的截止方式，在某些极端情况下，单一截止方式可能会导致验证者减少质押以留在该计划中。

### **投票成本覆盖范围**

该计划的另一个主要好处是覆盖投票成本。如前所述，投票是验证者的主要运营成本，对于新的、质押较低的验证者来说可能会令人望而却步，从而阻止许多人尝试运行节点。为了缓解这一问题，SFDP 为运营商在该计划的第一年提供投票成本补贴，涵盖前三个月的 100% 投票成本。每个季度（45 个纪元）减少 25%，12 个月后结束。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eccc671157990954b5_AD_4nXf3t_2HSJs2358SxX04We_-S5t5U2osamkabi-wkcBFMUbs-agWxzGDWnY0karKV4zxYO6eLtWZHbtU9FwE8W-h9E3YxH-1k3EzW3uRhOva-vZi3OcQWiQOq_hVrR4matJ6v-2E8qDn5v8von5PYE5w8wLn.png" loading="lazy" alt="">

### **计划标准**

潜在参与者必须通过 KYC/AML 流程才能进入该计划。他们还必须满足该计划官方网站上详细列出的[性能标准](https://solana.org/delegation-criteria)。目前，接收剩余质押有十项要求，而接收质押匹配则有一项要求。投票成本覆盖率与剩余成本具有相同的要求。

### **剩余授权要求**

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ebc1e2f48372ea46d8_AD_4nXcbQZMA62TkWcFMmapY7jGi0lWfC7gdTI9P9jYVp1S5nqvR1NP_nDsLiEXM2-dJD2hgX-9fOCgCQEY2d7bGMSkafwo3xsxSD-_CnI2xIv0ZZT_h07QOb8if-6g4cOKBFsNs67POUmTdbPNcR0_-xC1LRrk1.png" loading="lazy" alt="">

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eb6c6be2f908dad183_AD_4nXfGlEQMdGjCrxnN20g-5BQ2Sy5SvMglsYoe31NE2dvTEZ0xvuyl_Rgrr_o34HGGZtS0xpsozwNqY7_ZVAQu1x_xZ0BZJOEkltJhssfOtZw-xGUsQ6HiR1p2rOl4Yazw060o0V5GvHWb6Y8dcs2KDOTSxgA.png" loading="lazy" alt="">

### **质押匹配要求**

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eb9483080f7c8fccf6_AD_4nXdrwizOhiGGmXgJJReKamKm6v0BhgL5cUKD6tpm9LkBeuCnqhIjkjXeOadzwP01TUaqA64qu7SYSAqDGWM71R4vh7PfCperHVvr3jXwus_K0Q9RS4f4ARwcMxg4uhhHwlkpVRXlGsbFmYtffLKjDfwILVU.png" loading="lazy" alt="">

SFDP 参与者可以运行 Jito 验证器客户端并出售 SWQoS 带宽。然而，他们被[明确禁止](https://discord.com/channels/428295358100013066/895740485140906054/1237454455079964693)加入对用户三明治攻击的私人内存池程序。

官方网站提供了参与 SFDP 计划的个人验证者的详细信息。可以通过[此处的](https://stakeview.app/sfdp_stakes/0647.json) Stakeview 以原始 JSON 格式访问计划质押。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eb741813035257f575_AD_4nXfAvvf5rvtwXi6ZIbNmpXQ7SQ5cJcSN7kODo_-bCCUJAMMrvMW5rq6JVJuh1I0YDj_JH7q-VXTaOvQ2TjqEpy66nT8KLzTlpQlB5Mbz9D6iQaDw4jF6f31joDJISVvk_YMQwSJgAmsJSW7JezVTGwdyQI4.png" loading="lazy" alt="">

<div style="text-align:center">上图：示例计划参与者信息（验证者选择 ̶a̶t̶ ̶r̶a̶n̶d̶o̶m̶ 因为他们的可爱图片）</div>

详细的委托计划质押信息在每个纪元都会通过 Solana Tech [Discord 服务器](https://discord.gg/solana)“sfdp-mb-stake”通道公开发布。 SFDP 之前提供了[CLI 工具](https://docs.rs/crate/solana-foundation-delegation-program-cli/latest)，但已弃用。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ec741dae3cb21c6ef4_AD_4nXd9OvrNgFhoUtZG7P5e1bnAvMLC1LAHpvWvvzIR60ZZ5wrYhFm8g62U4e3VYgqC7zu1vsmuANFY-kqhtlImPwjTuxc5PtkFdmkLZnlLALyKkgX3SC_2tmfSVvb_HwPmj1kYeKE7CTqhjvieUL-Aqt22AkM.png" loading="lazy" alt="">

<div style="text-align:center">上图：SFDP Discord 对 Epoch 653 的总结</div>

SFDP 并不是唯一的此类计划。其他区块链上的类似举措包括 Celestia 基金会的[委托计划](https://docs.celestia.org/community/foundation-delegation-program?)（[据估计](https://chainflow.io/independent-validators-are-reliant-on-foundation-delegations/)该计划使所有活跃 Celestia 验证者的一半以上受益）、Sui 基金会的[委托计划](https://blog.sui.io/token-delegation/)和 Near 基金会的[委托计划](https://near.org/blog/near-foundation-launches-new-validator-program-to-further-decentralize-the-network)（不再活跃）。

## 验证者格局和 SFDP 的影响

在讨论 SFDP 的影响之前，我们首先需要对 Solana 的验证者和质押环境建立基本的共同理解。

尽管 Solana 联合创始人 Anatoly Yakovenko 此前曾表示长期[目标](https://docs.google.com/document/d/1fQp2G-W0fFN19nRZVRbAwxD7Qa8H8cn5VVUvPKOF1Pg/)是拥有超过 10,000 个全节点，但 Solana 目前的验证者总数为[1525 人](https://solanabeach.io/validators)，低于 2023 年 3 月 2,564 人的高点，与大多数业内同行相比这个数字仍然相对较高。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ebcd21138bc275e814_AD_4nXfdXMhlZuLQCJ3LuC6RsNGb9_j0d-BgNqGnq7M4Z9FG1Ia0tPpZvXQ54bLUnXBHwBQBi5fVuk1g_t_6bxyv1CjQybzPPnMwLEU_VpiViO-UdsOGuFsRi0Z6WyinP99-BeYTOVw8t3-8Nuax2jb0clpwk6mp.png" loading="lazy" alt="">

然而，正如区块链上的地址并不等于用户一样，验证者数量也不等于运行验证者节点的实体的真实数量。这个数字可能会略低，因为较大的实体通常会将其质押分配给多个验证者节点；示例包括 Jito ( [1](https://solanabeach.io/validator/J1to1yufRnoWn81KYg1XkTWzmKjnYSnmE2VY8DGUJ9Qv) , [2](https://solanabeach.io/validator/J1to2NAwajc8hD6E6kujdQiPn1Bbt2mGKKZLY9kSQKdB) )、Coinbase ( [1](https://solanabeach.io/validator/beefKGBWeSpHzYBHZXwp5So7wdQGX6mu4ZHCsH3uTar) , [2](https://solanabeach.io/validator/6D2jqw9hyVCpppZexquxa74Fn33rJzzBx38T58VucHx9) ) 和 Mrgn ( [1](https://stakewiz.com/validator/7emL18Bnve7wbYE9Az7vYJjikxN6YPU81igf6rVU5FN8) , [2](https://stakewiz.com/validator/mrgn2vsZ5EJ8YEfAMNPXmRux7th9cNfBasQ1JJvVwPn) , [3](https://stakewiz.com/validator/mrgn6ETrBDM8mjjYN8rbVwFqVwF8z6rtmvGLbdGuVUU) , [4](https://stakewiz.com/validator/mrgn4t2JabSgvGnrCaHXMvz8ocr4F52scsxJnkQMQsQ) )。单个实体运行多个验证者节点并没有本质上的错误。假设验证者节点不在同一位置运行，它可能会通过增加地理和 ISP 多样性而使网络受益。另请注意，并非所有验证者节点都与实体公开关联，这是无许可系统的本质。

目前 SOL 的总供应量为[5.82 亿枚](https://explorer.solana.com/supply)，质押的 SOL 数量为[3.82 亿枚](https://stakeview.app/stakes)，约占总供应量的 66%。即使在通货膨胀的情况下，这一质押水平多年来仍保持相对稳定。高质押率部分归因于质押过程的简单性。委托权益证明 (Delegated Proof of Stake，DPOS) 共识原生内置于 Solana 网络中。可以直接通过钱包、生态系统 dApp 和各种比较平台进行质押。代币持有者可以在每个纪元边界轻松地将 SOL 质押或取消质押给验证者。此外，他们可以委托给质押池或购买流动质押代币，这相当于质押。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eb1347ac9af5a7fbc6_AD_4nXdIdXIhD24DLRfuA2pjtPzOZ05mUO3r5Q9x8xn4nLfs-iETHPwyeVKa7RMvKUxrluQveWjLfBIgm9sL8oa0NeJoiviU7hjVGaS2E5s2P4frfpVrxqIdBksWznbxUmR3-wEf8vPkx_3eGOyRV35nJUmEz4o.png" loading="lazy" alt="">

接下来，让我们检查 Mainnet-Beta 验证者群体当前的质押分布。直接在线性比例图表上绘制这个长尾质押分布有点尴尬：

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ebddf0be2659c1857a_AD_4nXddmQLZIYOgqKyFNEuqQVoCDYiRX9RvN5ZZEpd0J0hUmhR6-KmWez6SMQas6DmD4ffJeJsixz-k5r4eO2O4n0f6YkQ_ayqqARqv7nCis7F_tM-7Tq3av_i_O03cidIqZhNIDFgvZQ05VN1-7lOpCXbG9JLa.png" loading="lazy" alt="">

为了便于分析，下面是使用对数刻度绘制的相同数据：

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ea54c570d11a922a62_AD_4nXczwW4mDPwkk4kX8VmldmWUO1rL4eEhlwqu-xMqfyIukzapU19LN9d-F9NBN1lr7Zrv9rHN507g73kk5McGgBpMFR7cnBdsiv1oehGjO5hKATp5AVKY5CSACHZM5uDnCUB4uFRMlkJOLHE_bNJhGMal8WXb.png" loading="lazy" alt="">

正在质押的地址总数目前为 43.7 万个，多年来一直稳步增长。目前质押账户总数为 127 万，超过了这一增幅。

每个质押地址平均有 2.9 个质押账户。单个用户控制多个地址有很多正当理由，所以质押地址不能直接等同于质押用户。然而，可以合理地预期用户数量将随着地址数量的增加而呈定向上升趋势。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ebc672439f94dd41cb_AD_4nXcRG_lMuhz7qxveda9f3uaLm4ZMs3idhtIH_inr4D7ZEpsa1byIKcM_iHbGoE4ClpR7W3HI-zWEM-kYXw_DeJiWkccYvmT9rpu08mYX-7etvm6aMOGrAkOafbMTCVuU8n481UVAkXiRaQ3BMEHyZ2Q9_j8.png" loading="lazy" alt="">

锁定质押长期处于缓慢稳定下降的状态。目前为 4750 万 SOL，占总质押的 12%。由于 FTX Estate/Alameda Research 破产程序中的代币[销售](https://cointelegraph.com/news/pantera-fundraise-buy-250-million-sol-ftx)，今年锁定的质押账户数量跃升至 5,194 个历史新高。

锁定质押是指账户中持有的代币，其条件是在预定日期之前无法提取。这些[锁定参数](https://medium.com/web3-builders-alliance/unlocking-the-secrets-of-staking-a-deep-dive-into-the-stake-program-on-solana-1b9721c92d37)由指定的托管人在创建帐户时设置，基于特定的 UNIX 时间戳或纪元。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ec44038d5aa35ea865_AD_4nXfBkZCHw_Hj7ntIy8SkNQOb24SDQERZEzsyUepWQg1bz-unhdk01v-cWx7nP2PRfL72ZNp3cTxjezcPZttQjA0iQA-vZ1GVtvEi9XOerF9cgz2pfLlLkyod9ML7G7NK1rfwn7LP_Qt2qD_R4_238YdJRxo_.png" loading="lazy" alt="">

令人惊讶的是，所有质押 SOL 中的[94%](https://dune.com/ilemi/solana-staking)是原生质押，只有 6%（2420 万 SOL）利用流动性质押，这可以提高资本效率并轻松参与整个 Solana 的 DeFi 生态系统。流动质押数量从一年前的 1240 万 SOL 增加到 2024 年初的 1700 万 SOL （年增长率为 95%）。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eccc671157990954b2_AD_4nXeXqm_z517vBOb4keGWk-tLTYorcUZYu2mcw1S5Ot7_IoZzYcwoHZqfLv32WSzQgQWmt9BplCoR884qun_JSQgKKa4sHMDSjkMJuk63SwOpcLCus9snWRanWOtrTXqjTZCb3xHq6oeV8zN5P-eIQhU2GK9M.png" loading="lazy" alt="">

最后，Solana 经常被引用的[中本聪系数](https://nakaflow.io/)目前为 19，低于 2023 年 8 月 13 日的峰值 34。中本聪系数是去中心化的粗略衡量标准，代表可以合谋停止区块链的最小节点集，也称为作为超级少数（superminority），占前三分之一的质押的节点。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ea4df8d85c60f722db_AD_4nXfY0FwN5zyRq8ndxoNv_Z9hYIaX1wMSCrn0lVpYOenNGJgBq2b5PLgL6qU6LZ9H8hFTLE5iyMpz5IUKCj3yki4MlHIBti-8pMwsd9v6grfWm_-Qf0emSUKBgWsXn_1A7SmNeKDumsjjNko2Wg2LKpvlYI48.png" loading="lazy" alt="">

### SFDP

下面我们具体分析一下 SFDP。本节中提供的所有图表和值均基于 658 纪元（2024 年 8 月中旬）。可以[在此处](https://docs.google.com/spreadsheets/d/1f_aczNyRHbHDBFEn5yF2SPSXYmYNZruttDfJAT1yFNA/edit?usp=sharing)访问这些图表和值的原始数据。

SFDP 的委托由单一[质押授权账户](https://solscan.io/account/mpa4abUkjQoAvPzREkh5Mo75hZhPFQ2FSH6w7dWKuQ5#stakeAccounts)进行管理，该账户目前委托了超过 5100 万个 SOL，占 Solana[总质押](https://dune.com/queries/3229774/5401861)的 13%，在撰写本文时价值为 73 亿美元。在过去 3.5 年的运营中，管理的质押比最初的[1 亿 SOL](https://medium.com/solana-labs/announcing-the-solana-foundation-delegation-strategy-5bcccf9104ab)减少了大约一半。

获得 SFDP 支持的验证者总数为 1105 人，占 1525 名验证人总数的 72%。受益于 SFDP 的验证者占 7400 万个 SOL（SFDP + 非 SFDP 质押），即 Solana 总质押的 19%。不依赖 SFDP 的 420 名验证者拥有 81% 的活跃质押。平均委托数量为 45,652 SOL，中等平均委托数量为 31,513 SOL。

将 SFDP 质押叠加在总质押对数图上可以看出，尽管 SFDP 质押仅占总质押的 13%，但它却对长尾验证者产生了不成比例的影响。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ed1894477d0ee1cb2e_AD_4nXc4zE1FmYx0AzalYHOZEEJ1iwSaOGBlqvxH7_Hj1h-om65cY6IvDEbYKv_crauqi27dHyOAeOlqmkjMkoekDm9zdD3UfWFDI5YI1fdixIkFvIA_H5HEouGJWKfhrdpYrL6E8iM6E3suFaCWuFUtiT9pVRPl.png" loading="lazy" alt="">

从上图中删除 SFDP 质押进一步揭示了该计划在三分之二超级多数（supermajority）之外的相当大的影响。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ebd33e8596ae7d9f17_AD_4nXcfprpOyWxztqIXKdn2QsR9zrT2Lxt2_QUBWmrLkgECRL5KDnDJJCOe9AIT6IXUAGKwT98Uk8qEcKs2YUQvOfprSBRIHWYkaxQtIvrZ-GDw9zTpgZo0KOIoIYQfpw8EjPCXGaz7FGbyIbkAQ91nhLMF-pM.png" loading="lazy" alt="">

就 SFDP 参与者的总质押（SFDP + 非 SFDP 质押）而言，13 个验证者（参与者的 1%）拥有超过 300k SOL，205 个验证者（19%）拥有超过 100k。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eca64ae6a162109088_AD_4nXdjnzqh5J9kfFle-t0St1vCheAbAWz1wf9p1o0F5VfvXj3EELRb_XhU_-DecXDbbTGFpHt6PKz3qzhcwhrmyWXIULiz8WIQY3fXCpS8PEW2aFlIJ_TwKVDDSMWetWsJ9ao7h36os4JLIqrb0j0TEQdyd4qB.png" loading="lazy" alt="">

当我们删除 SFDP 质押并关注计划参与者能够吸引自己的非 SFDP 质押时，我们看到 50 个验证者 (4.5%) 拥有超过 10 万质押，184 个验证者 (17%) 拥有超过 5 万质押， 810 名验证者 (73%) 拥有不到 10,000 质押，567 名验证者 (51%) 已成功从计划之外吸引了不到 1,000 质押。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eb289210dc27e004ec_AD_4nXcSfrHn7GAgXnpl818czXUL6icRPGOoHM9xRLE63H7Js_76UPP4CbNA0lupz8GkW1cDFKlfHjqDXJV-B5A1QeAPkecXQ6u12-MxtEH-61atrEI1IVgO9jfiPed_f-CLrwMSvw9mC-cVX52HniZv4kooCMMb.png" loading="lazy" alt="">

新的 SFDP 参与者上线率显示出多年来一致的趋势线，表明参与者是缓慢地进入该计划而不是分批加入该计划。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ebdf8918039670810f_AD_4nXd3tSkytzhJMeBxVu2KgBaDNWRf1smDqfYqsWXxvSyZWiijwmyinUsNayV1V7mz93vxoRs_GOlF43Gby2GIjm7L5VBErNygL0S6IHaQ5ZjhHCkgZRK-bWQCp2wu_3xejX5Ke51GRNJWpsJDjqyPFrHeVuTd.png" loading="lazy" alt="">

763 名 SFDP 验证者 (69%) 将佣金率设置为 7%，这是该纪元计划允许的最高佣金率。 295 个验证者 (27%) 将佣金率设置为 0%。

有趣的是，**所有 SFDP 验证者中有 531 个 (48%) 的佣金率最高设置为 7%，并且吸引了不到 1k SOL 的**非 SFDP 权益。在按其从计划之外吸引的非 SFDP 质押水平排名的前 100 名计划验证者中，有 18 个的佣金率为 7%。在排名最后的 100 个验证者中，这个数字是 97 个。

Solana 验证者社区的长期成员[此前估计](https://x.com/laine_sa_/status/1800424217432690897)，需要大约质押 35,000 SOL 才能覆盖投票费用，这是验证者最重要的支出。目前，只有 208 个 SFDP 验证者 (19%) 的非 SFDP 权益达到这一基准。**据估计，如果 SFDP 立即终止，该计划的 897 名参与者（占所有 Solana 验证者的 57%）将无利可图，**达到无法再支付投票费用的水平，更不用说诸如此类的额外费用了。如硬件、带宽和机会成本。

由于 SFDP 的上限为 100 万个非程序委托 SOL，**目前超级少数或超级多数中的验证者都无法获得 SFDP 支持**。考虑到这一硬性限制，只有在超级多数之外的最后 32% 质押的长尾验证者才有资格。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eb49a5fc9d95e46c7e_AD_4nXfOU3lzmm-uQFYme78iIZjdamMXzvhm1hHa2IQ1VZkNm5DyCddSGIG2QHMrmc-XHZ-0zkFkPKjDh00UzVx8Bn_G_q2q9kIrQbDz8RHF50AMqJWQvC4bLlpwZ3Wb224G9V3z_fVKoAgxYhRzJDft4q1xcGKB.png" loading="lazy" alt="">

## 验证者的可持续发展之路

本报告的这一部分将从一个较新的、质押较少的独立 Solana 验证者的角度来审视问题——这正是 SFDP 旨在帮助的实体类型。他们有哪些选择可以在 SFDP 之外增加他们的质押？他们如何在市场中获得竞争优势？这一群体的长期前景如何？

总的来说，一个独立的长尾验证者有三个主要渠道来增加质押：SFDP、质押池和自有渠道。我们已经讨论过 SFDP，现在让我们来看看其他可用的选择。

### 质押池

"在商业中只有两种赚钱的方式。一种是捆绑，另一种是解绑。" — Jim Barksdale，Netscape 公司

[质押池](https://solana.org/stake-pools)通常被[认为](https://docs.solanalabs.com/operations/validator-initiatives)是独立验证者获得质押的最佳方式之一。它们是传统原生质押的替代方案，通过允许轻松地将质押分配给多个验证节点来帮助去中心化网络。作为对质押池进行质押的回报，委托人通常会收到一种流动性质押代币（LST）。每个质押池都有自己独特的验证者标准和委托策略。一些池强调地理位置的去中心化；几乎所有池都要求验证者满足最低性能标准才能被添加到验证者集合中。单个验证者可以同时参与多个质押池。

Solana 上参与验证者数量最多的质押池是 [Blaze](https://www.jito.network/stakepool/stk9ApL5HeVAwPLr3TLhDXdZS8ptVu7zp6ov8HFDuMi/)（351 个）、[Marinade](https://www.jito.network/stakepool/marinade/)（337 个）、[Jito](https://www.jito.network/stakepool/Jito4APyf642JPZPx3hGc6WWJ8zPKtRbRs4P815Awbb/)（283 个）和 [JPool](https://www.jito.network/stakepool/CtMyWsrUtAwXWiGr9WjHT5fC3p3fgV8cyGpLTo2LJzG1/)（263 个）。所有这些池都有严格的标准，验证者必须满足这些标准才能加入池中。一些常见的要求是验证者处于超级少数之外、在最近几个纪元中有较高的 APY，以及较低的跳过时隙率。详细的委托标准可以在这里找到（[Jito](https://www.jito.network/docs/jitosol/stake-delegation/)、[Blaze](https://stake-docs.solblaze.org/protocol/delegation-strategy#what-is-our-goal-for-selecting-validators)、[Marinade](https://docs.marinade.finance/marinade-protocol/validators-1)、[JPool](https://docs.jpool.one/for-validator-operators/how-to-join-jpool-delegation-program)）。管理费用分别为 mSOL 6%、bSOL 5%、JitoSOL 4% 和 JPool 5%。每个纪元池都会重新平衡，上限约为总质押的 5% 到 7.5%。

质押池是经典的在线聚合平台，用于匹配供需双方。在这种情况下，供给方是无差异化的独立验证者，你可以将其视为类似于优步司机或电商平台上的独立小商户。需求方是成千上万寻求质押相对较小或中等数量 SOL 的用户。用户寻求便利性和以具有竞争力的 APY 形式的回报。聚合平台则致力于扩大市场份额并提高抽成率，以促进双方之间的交易。独立验证者主要希望增加他们的质押量。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ea4991ae486c181d1e_AD_4nXfx3yzDraoZXSaCY7wsoF3OT1mVM8zSNqMQLoI7uVfcDw1Op6QWehxY7RCDSHKqVj8VyFkytOmvbFhO5lUy4pdhRToQ7EFAt7_ckRdwPkNZb7y2CtbFqGUO6HzPA41UQIt6jKcG6CEeZ7vbZFntV2rondGG.png" loading="lazy" alt="">

基于互联网的聚合平台已经被广泛理解。通常在这种价值链中，价值会流向能够对供应方施加定价权的聚合平台。对供应方来说，原本是免费的午餐随着时间的推移演变成了一个高度竞争的付费参与系统。我们看到这种趋势正在 Marinade Finance 的新[质押拍卖市场](https://docs.marinade.finance/marinade-protocol/validators-1)（Stake Auction Marketplace，SAM）委托策略中显现——这是一个竞争性的价格拍卖，验证者在"付费获得质押"的系统中直接相互竞价以获得质押分配。验证者被促使到他们认为仍能盈利的最高出价，如果他们相信 SFDP 会匹配他们从 SAM 获得的质押，甚至可能超过这个比率。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eaf76d36fb3f201a33_AD_4nXfeEdS7SzrWyJWTyPyNTap1GxqF-8inUwemh2uxy2nhFF-2NKL6gDGkg8CX8yol31AgwEZ0fkC4PgPbRia7Bsf3s8mUgnDTDPDzJVz5VUyg2td9QLfD9USuIyA8KServPFUj3dQIsxbRcHPlz9Z_pCXMkW-.png" loading="lazy" alt="">

<p align="center">数据来源：<a href="https://www.jito.network/stakepools/">Jito</a>, <a href="https://dune.com/ilemi/solana-staking">Dune</a></p>

Jito 是质押量最大的池，超过 1200 万 SOL。平均分配到该池的 283 个验证者中，每个验证者约有 42.5k SOL——这是一个相当可观的分配，单凭这一点就能使验证者达到盈亏平衡点。其他池的平均值较低，考虑到它们之间的重叠，仅依靠这些池就能为几百个验证者提供足够的质押支持。

### Sanctum 和验证者流动性质押代币

今年一个[备受关注](https://www.helius.dev/blog/lsts-on-solana)且日益流行的质押池模型补充是 [Sanctum](https://www.sanctum.so/)。Sanctum 消除了发行流动性质押代币（LST）的许多关键障碍，为希望推出独立品牌 LST 以吸引质押的长尾验证者解决了流动性供应和 DeFi 可组合性的问题。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eb4eec7217a3ddc9cd_AD_4nXe5precCeZdcWGC4V-q5RB26TU-QCLnCT15P0fAYNvcjPs-7Xz7q9s4dgk4GZieyUIU7KLA3ABZHQG2vsnY4HqDR9PccyEqO3R9Q1vCLL1X6biOpZY9xTKst0r9GIbCftHzAKxBIDtLqvm4ZTrJjUyOPzgC.png" loading="lazy" alt="Sanctum 验证者流动性质押代币模型图">

Sanctum 可以被视为一个类似 [Shopify](https://www.shopify.com/) 的平台，为验证者提供了直接面向消费者（Direct-to-Consumer，DTC）所需的基础设施。然而，它也面临着与 Shopify 相同的关键缺点——无法提供有意义的分销渠道。Sanctum 只是一个工具包，尽管是一个重要的工具包，它在很大程度上平衡了最大、最具流动性的 LST 提供商和小型参与者之间的用户体验差距。对于那些已经实现产品差异化和/或拥有自有分销渠道的验证者来说，Sanctum 是一个重要的突破。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ea1148c6e64abd205f_AD_4nXehDjmBdhnJsjVHDHQ-N7fMWUgmYkbTkT6Fu7pH9snr2mUEB5S1SnI21TquuWkudcLGxMeic4pApnrKRjJGn_7sshYN_TnSAyePigHSt-ZWNWX2epLRY9pIpOfetAOhBWlnmJqN8Lh6eUWXhpDOzKV6oIU.jpeg" loading="lazy" alt="Sanctum 验证者申请表单列出的优先验证者类别">

<p align="center">上图：Sanctum 的验证者申请表单列出了优先考虑的验证者类别</p>

这突出了一个对于质押较少的纯独立验证者来说的关键问题——他们缺乏有效推广服务的渠道。创建内容、管理社交媒体和社区建设都可能是有效的方法，但这些技能并不适合小型的、专注于技术的验证者运营团队。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852ea0c37ef8cdda46976_AD_4nXfB9mHaRRJBFRJMTt1GR7KHRSJ2Pr5DRRI4ul8TBEBvYp2LM16Br8GiYoF8Afa6rZmx9J26aZV0gg_AUvrvprOqofClxs_FAd4Dy2Zsdz4WmBi0s5dQo0gfSsxqgU2m6UwwvhMP8llUDHpVfRABDl2Qq1XM.jpeg" loading="lazy" alt="">

<p align="center">上图：验证者的推广/分销渠道示例</p>

回顾我们之前的验证者格局概述，我们看到其他群体，即生态系统团队、交易所和面向机构的质押服务提供商，有更明确定义的渠道来接触质押者。生态系统团队运营 dApp、钱包和基础设施，这自然会为质押服务产生高质量的潜在客户。如果精心规划，将用户转化为质押者的追加销售或促销活动可以相对无缝地进行。面向机构的质押服务自然会采取不同的多管齐下的方法，包括高层关系建立、招聘 B2B 销售团队和积极参与行业活动。可以说，中心化交易所的地位是最好的，能够利用其现有的庞大客户群。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eb41f8fb3aa67cd5f2_AD_4nXeFq9rGu2SLrY4-TA_3AhOUX92j2XjNNlPM4XqWX52Q4ImlZFXQM9zDCAWeVhDuavPuNRK3E4ojOAj-U-tfLytUjX3R8DlVzyrtfqN1qgzGnplhtV-xu9tI_5ktP_7WuiLxg1rizRurVpbFBzu_sVXoKN7E.png" loading="lazy" alt="">

为了获得市场份额，长尾独立验证者必须竞争非被动质押的部分——这部分质押在链上被积极管理，并对诸如 APY 利率等因素做出响应。然而，这个数量相对于总质押量来说相当小；例如，所有 LST 加起来只占 6%。此外，通过提供 0% 的通胀和 MEV 佣金，验证者可能无意中吸引了最敏感于价格的质押者——当验证者试图提高佣金率到更可持续的水平时，这些质押者最有可能撤走质押。

### 投票费用

另一个不利于长尾验证者的因素是投票费用的动态变化。投票费用是运行验证者的主要成本；由于以 SOL 计价，随着网络价值的增加，这一成本将进一步上升。Solana 当前的投票费用机制主要有两个效果。首先，它强制执行了一个相对较小的 SOL 代币通缩性销毁，这有利于所有代币持有者。其次，它将财富从质押较少的验证者转移到质押较多的验证者，如下面的高层次示例所概述的那样。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/66c852eb412a8060a9d66d6c_AD_4nXdfSXGI47_ncHOFVsk2cHh9T4wMm8555ITo4rj_B1RSNom7k5N2_LXAkZXsRh3iOfSxVYOxIiXqrukxr9DMwG0eOYZdkRrk32E8bznP3FBkPsfzPVt-cl-aJvL6mrdCkfX2xNwvMrfjmnzzYM7THFIYT8BY.png" loading="lazy" alt="">

<p align="center">上图：仅包含五个验证者的投票费用重新分配简化模型</p>

无论质押多少，所有验证者都必须投票并支付相同的投票费用。这些费用的 50%被销毁，50%分配给区块构建者。然而，领导者槽位是根据质押分配的，这意味着质押较高的验证者在每个纪元中构建更多的区块，从而从其他验证者那里赚回更多的投票费用。因此，投票费用可以被视为一个有利于高质押验证者的循环经济。

在 2024 年期间，优先费用和 MEV 佣金的显著增加降低了投票费用的相对重要性。然而，该机制的固定成本和可变收入动态仍然构成了一种持续的中心化力量，使长尾验证者处于不利地位。

## 未来研究领域

基于本文讨论的挑战，未来研究有几个潜在方向。以下是一些值得考虑的方向：

• 可以开发哪些激励机制来抵消中心化力量并更好地支持低质押验证者？

• Solana 验证者的理想数量是多少？什么样的中本聪系数和质押分布可以被认为是"健康"的？

• 如何修改 SFDP 以更好地使参与者的激励与计划的预期目标保持一致？

• 是否有策略可以鼓励大型私人、交易所和机构验证者更均匀地分配他们的质押，可能利用现有的质押池基础设施？

## 总结

在本文中，我们研究了 SFDP 以及 Solana 长尾独立验证者面临的相关挑战。独立验证者的运营是一个高度竞争和开放的市场。这种商业模式相对透明，进入门槛中等偏低。没有自有分销渠道的新兴独立验证者是可替代的。如果没有差异化、协同业务或自有推广渠道，他们可能难以产生长期超额利润。即使有 SFDP 的支持，吸引质押也是一项挑战。大多数 SOL 被大型私人、机构和交易所验证者被动质押和持有，这意味着新验证者无法获得这些质押。SFDP 在其当前形式下，对解决这些更广泛的问题几乎没有帮助。
