# Solana 交易优先级机制深度解析

## 1. 优先级计算概述

Solana 在 1.18 版本中引入了新的交易优先级机制，通过经济模型来计算交易优先级，实现区块空间的高效分配。该机制将区块空间视为稀缺资源，通过竞价机制分配给出价最高的用户。

与传统的优先费机制不同，新机制不再是简单的矿工激励，而是一个完整的经济模型。中央调度器会综合考虑交易成本（cost）和用户支付的费用，通过优先级公式对冲突的交易进行排序，从而实现区块空间的最优分配。

## 2. 优先级计算核心

### 2.1 计算公式

优先级通过以下公式计算：

$$
Priority = \frac{Reward \times MULTIPLIER}{Cost + 1}
$$

其中：

$$
\begin{align*}
& Reward：&& \text{交易的奖励（主要基于交易费用）} \\
& Cost：&& \text{交易的执行成本} \\
& MULTIPLIER：&& \text{乘数因子（}1,000,000\text{），用于调整奖励和成本的比例关系}
\end{align*}
$$

### 2.2 核心实现

```rust
fn calculate_priority_and_cost(
    transaction: &impl TransactionWithMeta,
    fee_budget_limits: &FeeBudgetLimits,
    bank: &Bank,
) -> (u64, u64) {
    // 计算交易成本
    let cost = CostModel::calculate_cost(transaction, &bank.feature_set).sum();
    // 计算交易奖励
    let reward = bank.calculate_reward_for_transaction(transaction, fee_budget_limits);

    const MULTIPLIER: u64 = 1_000_000;
    (
        reward
            .saturating_mul(MULTIPLIER)
            .saturating_div(cost.saturating_add(1)),
        cost,
    )
}
```

### 2.3 设计考虑

1. **成本与奖励平衡**
   - 优先级与交易奖励（费用）成正比
   - 与交易成本成反比
   - 使用乘数因子确保小额交易也能获得合理优先级
2. **数值处理**
   - 分母加 1 避免除零错误
3. **资源效率**
   - 成本计算考虑了交易的资源消耗
   - 通过优先级机制实现资源的高效分配

### 2.4 成本计算组成

交易的总执行成本（Cost）由以下几个部分组成：

1. **签名验证成本 (signature_cost)**

   - 普通签名：30 \* 24 units/signature
   - Secp256k1：30 \* 223 units/signature
   - Ed25519：30 \* 76 units/signature
   - Secp256r1：30 \* 160 units/signature

2. **写锁成本 (write_lock_cost)**

   - 每个写锁：30 \* 10 = 300 units/lock

3. **程序执行成本 (programs_execution_cost)**
   有两种计算路径：

   - 通过计算预算指令设置
     - 使用 compute_budget_limits.compute_unit_limit
     - 由 compute budget program 显式申请计算单元
   - 通过指令执行成本计算
     - 内置程序：使用固定成本
     - 用户程序：默认 200,000 单元/指令
     - 总限制：1,400,000 单元/交易

4. **数据成本**
   - 指令数据成本 (data_bytes_cost)：基于数据字节大小
   - 账户数据加载成本 (loaded_accounts_data_size_cost)：基于内存使用
   - 账户数据分配成本 (allocated_accounts_data_size)：基于新分配的账户数据，system program 的 create account 和 alloc 相关，1 byte = 1 unit

### 2.5 交易奖励计算

交易奖励（Reward）是优先级计算公式中的另一个关键组成部分。系统通过以下方式计算交易奖励：

```rust
pub fn calculate_reward_for_transaction(
    &self,
    transaction: &impl TransactionWithMeta,
    fee_budget_limits: &FeeBudgetLimits,
) -> u64 {
    let fee_details = solana_fee::calculate_fee_details(
        transaction,
        self.get_lamports_per_signature() == 0,
        self.fee_structure().lamports_per_signature,
        fee_budget_limits.prioritization_fee,
        FeeFeatures::from(self.feature_set.as_ref()),
    );

    let (reward, _burn) = if self.feature_set.is_active(&reward_full_priority_fee::id()) {
        self.calculate_reward_and_burn_fee_details(&CollectorFeeDetails::from(fee_details))
    } else {
        let fee = fee_details.total_fee();
        self.calculate_reward_and_burn_fees(fee)
    };
    reward
}
```

#### 2.5.1 奖励计算机制

1. **新版本计算（启用 reward_full_priority_fee 特性）**

```rust
fn calculate_reward_and_burn_fee_details(
    &self,
    fee_details: &CollectorFeeDetails,
) -> (u64, u64) {
    // 计算基础交易费用的分配
    let (deposit, burn) = if fee_details.transaction_fee != 0 {
        self.fee_rate_governor.burn(fee_details.transaction_fee)
    } else {
        (0, 0)
    };
    // 优先级费用全部作为奖励
    (deposit.saturating_add(fee_details.priority_fee), burn)
}
```

2. **传统计算方式**

```rust
fn calculate_reward_and_burn_fees(&self, fee: u64) -> (u64, u64) {
    self.fee_rate_governor.burn(fee)
}
```

#### 2.5.2 关键特点

1. **费用组成**

   - 基础交易费用（transaction_fee，lamports_per_signature）
   - 优先级费用（priority_fee）

2. **奖励分配机制**

   - 新版本：优先级费用全部作为奖励
   - 基础交易费用部分销毁，部分作为奖励
   - 使用 saturating_add 防止数值溢出

3. **特性依赖**
   - reward_full_priority_fee 特性决定计算方式
   - 新版本对优先级费用和基础费用分别处理
   - 传统版本对所有费用统一处理

## 3. 提高优先级的建议

1. 在相同交易的情况下，提高 reward，即通过提高 compute unit price 来增加优先费
2. 在相同交易的情况下，降低 cost，由于 cost 的主要成本是程序执行成本，通过申请合适的 compute budget 即可提高优先级
