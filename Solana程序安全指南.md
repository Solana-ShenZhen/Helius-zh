# Solana程序安全指南

本文由bl0ckpain共同撰写，他是一名安全研究员和智能合约开发人员，曾在Kudelski Security和Halborn工作。

## 引言

Solana程序安全不仅仅是防止黑客窃取项目资金——更重要的是确保程序按预期运行，遵循项目规范和用户期望。Solana程序安全还会影响dApp的性能、可扩展性和互操作性。因此，开发人员在构建面向消费者的应用程序之前，必须了解潜在的攻击向量和常见漏洞。

本文探讨了开发人员在创建Solana程序时会遇到的常见漏洞。首先，我们介绍了利用Solana程序的攻击者思维，包括Solana的编程模型、Solana的设计本质上如何受攻击者控制、潜在的攻击向量以及常见的缓解策略。随后，我们介绍了各种不同的漏洞，解释了漏洞的原理，并在适用的情况下给出了不安全和安全代码的示例。

请注意，本文章面向的是中高级读者，假设读者已具备Solana编程模型和程序开发的知识。我们不会详细讲解如何构建程序或Solana特定的概念——重点在于分析常见漏洞并学习如何缓解这些漏洞。如果你是Solana新手，建议先阅读以下博客文章：

- [Solana编程模型：Solana开发入门]()
- [Anchor入门：Solana程序开发者指南]()

## 攻击者利用Solana程序的思维

### Solana的编程模型

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/654d46c12245f98f47d174d7_program.jpg" loading="lazy" alt="">

[Solana的编程模型]()塑造了其网络上应用程序的安全格局。在Solana中，账户相当于数据的容器，类似于计算机上的文件。我们可以将账户大致分为两类：可执行和不可执行账户。可执行账户（或程序）是能够运行代码的账户。不可执行账户用于数据存储，但无法执行代码（因为它们不存储任何代码）。这种代码与数据的分离意味着程序是无状态的——它们通过交易时传递的引用与存储在其他账户中的数据进行交互。
j
## Solana受攻击者控制

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/654d46c12245f98f47d174d7_program.jpg" loading="lazy" alt="">
交易会指定要调用的程序、账户列表和指令数据的字节数组。这个模型依赖程序解析并解释交易提供的账户和指令。允许任何账户传递到程序的函数中，这为攻击者提供了对程序将操作的数据的显著控制。理解Solana本质上受攻击者控制的编程模型是开发安全程序的关键。

由于攻击者能够将任何账户传递到程序函数中，数据验证成为Solana程序安全的核心支柱。开发人员必须确保他们的程序能够区分合法和恶意输入。这包括验证账户的所有权、确保账户是预期类型以及是否为签名者账户。

## 潜在攻击向量
Solana独特的编程模型和执行环境带来了特定的攻击向量。理解这些向量对于开发人员保护程序免受潜在漏洞的威胁至关重要。这些攻击向量包括：

- 逻辑漏洞：程序逻辑中的缺陷可能被操纵导致意外行为，如资产丢失或未经授权的访问。这也包括未能正确实现项目规范——如果程序声称做某件事，那么它应做到所有相关细节。
- 数据验证漏洞：不充分的数据验证可能允许攻击者传递恶意数据，操纵程序状态或执行。
- Rust特定问题：尽管Rust有安全特性，但不安全代码块、并发问题和程序崩溃可能引入漏洞。
- 访问控制漏洞：未能正确实施访问控制检查（如验证账户所有者）可能导致恶意行为者进行未经授权的操作。
- 算术和精度错误：溢出/下溢和精度错误可能被利用获取经济利益或导致程序故障。
- 跨程序调用（CPI）问题：处理CPI中的缺陷可能导致状态变化或错误，尤其是在调用的程序表现恶意或意外时。
- 程序派生地址（PDA）误用：不正确地生成或处理PDA可能导致漏洞，使攻击者可以劫持或伪造PDA，进而获取未经授权的访问或操控程序控制的账户。


值得注意的是，由于Solana的执行模型，重入攻击被天然限制。Solana运行时限制CPI的深度为4，并强制执行严格的账户规则，例如只允许账户的所有者修改其数据。这些限制通过限制直接自递归和确保程序无法在中间状态被强制调用来防止重入攻击。

## 缓解策略
为了缓解这些潜在的攻击，开发人员应采用一系列严格的测试、代码审计和遵循最佳实践的方式：

- 实现全面的输入验证和访问控制检查
- 充分利用Rust的类型系统和安全特性，避免使用不安全代码，除非必要
- 遵循Solana和Rust的安全最佳实践，并保持对最新发展的关注
进行内部代码审查，并使用自动化工具在程序开发过程中识别常见漏洞和逻辑错误
- 请可信的第三方，如安全公司和独立安全研究人员，审计代码库
- 为程序创建漏洞奖励平台，以激励报告漏洞，而非依赖灰帽黑客
  
接下来的部分将按字母顺序探索不同的漏洞。每一部分将描述潜在的漏洞，解释如何缓解该漏洞，并尽可能给出示例场景。

## 账户数据匹配

### 漏洞

账户数据匹配漏洞出现在开发人员未能检查账户中存储的数据是否符合预期值集合时。如果缺少适当的数据验证检查，程序可能会在不正确或被恶意替换的账户下操作。这种漏洞在涉及权限相关的检查时尤为严重。

### 示例场景

假设某个程序具有管理其设置的功能。该程序包含一个指令，用于更新当前的管理配置（如功能标志或操作参数）。该指令必须验证请求来自授权管理员。然而，该程序未能验证请求更改的账户是否与配置数据中存储的管理员账户匹配：

``` rust
pub fn update_admin_settings(ctx: Context<UpdateAdminSettings>, new_settings: AdminSettings) -> Result<()> {
  ctx.accounts.config_data.settings = new_settings;
  
  Ok(())
}

# [derive(Accounts)]
pub struct UpdateAdminSettings<'info> {
  #[account(mut)]
  pub config_data: Account<'info, ConfigData>,
  pub admin: Signer<'info>,
}

# [account]
pub struct ConfigData {
  admin: Pubkey,
  settings: AdminSettings
}
```

## 推荐的缓解措施

为了缓解这一漏洞，开发人员可以实现显式检查，将账户密钥和存储数据与预期值进行比较。例如，验证存款人的公钥是否与正在使用的存款代币账户的所有者字段匹配：

``` rust
pub fn update_admin_settings(ctx: Context<UpdateAdminSettings>, new_settings: AdminSettings) -> Result<()> {
  if ctx.accounts.admin.key() != ctx.accounts.config_data.admin {
    return Err(ProgramError::Unauthorized);
  }  

  ctx.accounts.config_data.settings = new_settings;
  
  Ok(())
}
```

开发人员还可以使用Anchor的has_one和constraint属性，以声明的方式实施数据验证检查。使用上面的示例，我们可以使用constraint属性检查存款人的公钥与存款代币账户的所有者是否相同：

``` rust
pub struct UpdateAdminSettings<'info> {
  #[account(
    mut,
    constraint = config_data.admin == admin.key()
  )]
  pub config_data: Account<'info, ConfigData>,
  pub admin: Signer<'info>,
}
```

## 账户数据重新分配

### 漏洞
在 Anchor 中，AccountInfo 结构体提供的 realloc 函数引入了与内存管理相关的漏洞。该函数允许重新分配账户的数据大小，这对于程序中的动态数据处理非常有用。然而，不当使用 realloc 可能会导致意外后果，包括浪费计算单元或可能暴露旧数据。

realloc 方法有两个参数：

- new_len：一个 usize 值，指定账户数据的新长度。
- zero_init：一个布尔值，决定新分配的内存空间是否需要被初始化为零。
realloc 的定义如下：

``` rust
pub fn realloc(
    &self,
    new_len: usize,
    zero_init: bool
) -> Result<(), ProgramError>
```

账户数据的分配在程序入口时已经被初始化为零。这意味着当数据在单次交易中重新分配为更大的尺寸时，新分配的内存空间已经被置为零，因此再次置零是不必要的，并且会导致额外的计算单元消耗。相反，如果在同一交易中将数据尺寸减少后再增大，并且 zero_init 被设置为 false，则可能会暴露旧数据。

## 示例场景

假设有一个动态的待办事项列表程序，用户可以在单次交易中添加、删除或修改待办事项。该程序需要根据用户的操作动态调整数据大小：

``` rust
pub fn modify_todo_list(ctx: Context<ModifyTodoList>, modifications: Vec<TodoModification>) -> ProgramResult {
    // 处理修改的逻辑
    for modification in modifications {
        match modification {
            TodoModification::Add(entry) => {
                // 添加逻辑
            },
            TodoModification::Remove(index) => {
                // 删除逻辑，可能需要重新分配数据
            },
            TodoModification::Edit(index, new_entry) => {
                // 编辑逻辑
            },
        }
    }

    // 根据修改调整数据大小的重新分配逻辑
    let required_data_len = calculate_required_data_len(&modifications);
    ctx.accounts.todo_list_data.realloc(required_data_len, false)?;

    Ok(())
}

# [derive(Accounts)]
pub struct ModifyTodoList<'info> {
    #[account(mut)]
    todo_list_data: AccountInfo<'info>,
    // 其他相关账户
}
```

在这个场景中，modify_todo_list 函数可能会多次重新分配 todo_list_data 以适应修改所需的大小。如果数据大小在删除待办事项时减小，然后在同一交易中因为添加新的事项而增大，设置 zero_init 为 false 可能会暴露旧数据。

## 建议的缓解措施

合理使用 zero_init 参数至关重要：

- 在同一交易调用中，先减小数据大小后再增大时，设置 zero_init 为 true，以确保任何新分配的内存空间被初始化为零，防止暴露旧数据。
- 在同一交易调用中，如果没有先减小数据大小而直接增大，则设置 zero_init 为 false，因为内存已经初始化为零。
开发者还可以使用地址查找表（ALT）来避免频繁的内存重新分配。ALT 允许开发者将多达 256 个地址存储在一个链上账户中，并通过 1 字节的索引来引用表中的地址，从而减少每个交易中对地址引用所需的数据量。

## 账户重新加载

### 漏洞

账户重新加载漏洞发生在开发者在执行 CPI（跨程序调用）后未更新反序列化的账户状态时。Anchor 不会在 CPI 后自动刷新反序列化账户的状态，这可能导致程序逻辑使用了陈旧的数据，进而引发逻辑错误或计算错误。

### 示例场景

假设有一个协议，用户可以质押代币以赚取奖励。程序根据某些条件或外部触发器来更新用户的质押奖励。用户的奖励通过 CPI 调用奖励分发程序来计算和更新。然而，程序在 CPI 之后没有更新原始质押账户，导致未反映新的奖励余额：

``` rust
pub fn update_rewards(ctx: Context<UpdateStakingRewards>, amount: u64) -> Result<()> {
    let staking_seeds = &[b"stake", ctx.accounts.staker.key().as_ref(), &[ctx.accounts.staking_account.bump]];

    let cpi_accounts = UpdateRewards {
        staking_account: ctx.accounts.staking_account.to_account_info(),
    };
    let cpi_program = ctx.accounts.rewards_distribution_program.to_account_info();
    let cpi_ctx = CpiContext::new_with_signer(cpi_program, cpi_accounts, staking_seeds);

    rewards_distribution::cpi::update_rewards(cpi_ctx, amount)?;

    // 记录“更新”的奖励余额
    msg!("Rewards: {}", ctx.accounts.staking_account.rewards);
    
    // 使用陈旧的 ctx.accounts.staking_account.rewards 的逻辑

    Ok(())
}

# [derive(Accounts)]
pub struct UpdateStakingRewards<'info> {
    #[account(mut)]
    pub staker: Signer<'info>,
    #[account(
        mut,
        seeds = [b"stake", staker.key().as_ref()],
        bump,
    )]
    pub staking_account: Account<'info, StakingAccount>,
    pub rewards_distribution_program: Program<'info, RewardsDistribution>,
}

# [account]
pub struct StakingAccount {
    pub staker: Pubkey,
    pub stake_amount: u64,
    pub rewards: u64,
    pub bump: u8,
}
```

在这个例子中，update_rewards 函数通过 CPI 调用奖励分发程序来更新用户质押账户的奖励。程序尝试在 CPI 之后记录 ctx.accounts.staking_account.rewards（即奖励余额），但由于质押账户的状态没有自动更新，导致使用的是陈旧的数据。

建议的缓解措施
为防止这个问题，应该在 CPI 后显式调用 Anchor 的 reload 方法来重新从存储中加载账户，确保账户状态是最新的：

``` rust
pub fn update_rewards(ctx: Context<UpdateStakingRewards>, amount: u64) -> Result<()> {
    let staking_seeds = &[b"stake", ctx.accounts.staker.key().as_ref(), &[ctx.accounts.staking_account.bump]];

    let cpi_accounts = UpdateRewards {
        staking_account: ctx.accounts.staking_account.to_account_info(),
    };
    let cpi_program = ctx.accounts.rewards_distribution_program.to_account_info();
    let cpi_ctx = CpiContext::new_with_signer(cpi_program, cpi_accounts, staking_seeds);

    rewards_distribution::cpi::update_rewards(cpi_ctx, amount)?;

    // 重新加载质押账户以反映更新的奖励余额
    ctx.accounts.staking_account.reload()?;

    // 记录更新后的奖励余额
    msg!("Rewards: {}", ctx.accounts.staking_account.rewards);
    
    // 使用 ctx.accounts.staking_account.rewards 的逻辑

    Ok(())
}
```

## 任意 CPI

### 漏洞

任意 CPI 发生在程序调用另一个程序时，未验证目标程序的身份。由于 Solana 运行时允许任何程序调用另一个程序（只要调用者有被调用程序的程序 ID 并遵循其接口），如果程序根据用户输入进行 CPI，而不验证被调用程序的程序 ID，可能会导致调用攻击者控制的程序。

### 示例场景

假设一个程序根据项目参与者的贡献来分发奖励。分发完奖励后，程序将相关信息记录在一个单独的分类账程序中以便审核。然而，函数没有验证传入的 ledger_program，这就可能让攻击者传入恶意程序的 ID，从而导致意外的后果：

``` rust
pub fn distribute_and_record_rewards(ctx: Context<DistributeAndRecord>, reward_amount: u64) -> ProgramResult {
    // 奖励分发逻辑

    let instruction = custom_ledger_program::instruction::record_transaction(
        &ctx.accounts.ledger_program.key(),
        &ctx.accounts.reward_account.key(),
        reward_amount,
    )?;

    invoke(
        &instruction,
        &[
            ctx.accounts.reward_account.clone(),
            ctx.accounts.ledger_program.clone(),
        ],
    )
}

# [derive(Accounts)]
pub struct DistributeAndRecord<'info> {
    reward_account: AccountInfo<'info>,
    ledger_program: AccountInfo<'info>,
}
```

攻击者可以通过传递恶意程序的 ID 作为 **ledger_program** 来利用这个漏洞。

### 建议的缓解措施

为防止这种情况发生，开发者应在执行 CPI 之前检查目标程序的身份。这样可以确保 CPI 调用发送到预期的程序：

``` rust
pub fn distribute_and_record_rewards(ctx: Context<DistributeAndRecord>, reward_amount: u64) -> ProgramResult {
    // 奖励分发逻辑

    // 验证 ledger_program 是否为预期的自定义分类账程序
    if ctx.accounts.ledger_program.key() != &custom_ledger_program::ID {
        return Err(ProgramError::IncorrectProgramId.into());
    }
    
    let instruction = custom_ledger_program::instruction::record_transaction(
        &ctx.accounts.ledger_program.key(),
        &ctx.accounts.reward_account.key(),
        reward_amount,
    )?;

    invoke(
        &instruction,
        &[
            ctx.accounts.reward_account.clone(),
            ctx.accounts.ledger_program.clone(),
        ],
    )
}

# [derive(Accounts)]
pub struct DistributeAndRecord<'info> {
    reward_account: AccountInfo<'info>,
    ledger_program: AccountInfo<'info>,
}
```

通过检查程序 ID，确保 CPI 调用仅限于可信的程序，从而防止任意 CPI。

## 权限转移功能
### 漏洞
在 Solana 程序中，通常会为关键功能（如更新程序参数或提取资金）指定某些公钥作为权限。然而，缺乏将此权限转移到另一个地址的功能可能会带来风险，例如在团队变更、协议出售或权限被泄露的情况下。

### 示例场景
假设某个程序中，全球管理员负责通过 set_params 函数设置特定协议参数。然而，该程序没有包含将全球管理员权限转移到新地址的机制：

``` rust
pub fn set_params(ctx: Context<SetParams>, /*parameters to be set*/) -> Result<()> {
    require_keys_eq!(
        ctx.accounts.current_admin.key(),
        ctx.accounts.global_admin.authority,
    );

    // 设置参数的逻辑
}
```

此处，权限被静态定义，没有办法更新为新地址。

### 建议的缓解措施
一个安全的缓解方法是创建一个两步的权限转移过程，允许当前权限所有者提名一个新的 pending_authority，然后由新的 pending_authority 明确接受角色。这种流程不仅提供了权限转移功能，还能防止意外转移或恶意接管。流程如下：

- 当前权限所有者提名新权限：当前权限所有者调用 nominate_new_authority，将新的 pending_authority 设置为程序状态中的待定权限。
- 新权限接受：提名的 pending_authority 调用 accept_authority 接受角色，将权限从当前权限所有者转移到 pending_authority。
示例代码如下：

``` rust
pub fn nominate_new_authority(ctx: Context<NominateAuthority>, new_authority: Pubkey) -> Result<()> {
    let state = &mut ctx.accounts.state;
    require_keys_eq!(
        state.authority,
        ctx.accounts.current_authority.key()
    );

    state.pending_authority = Some(new_authority);
    Ok(())
}

pub fn accept_authority(ctx: Context<AcceptAuthority>) -> Result<()> {
    let state = &mut ctx.accounts.state;
    require_keys_eq!(
        Some(ctx.accounts.new_authority.key()),
        state.pending_authority
    );

    state.authority = ctx.accounts.new_authority.key();
    state.pending_authority = None;
    Ok(())
}

# [derive(Accounts)]
pub struct NominateAuthority<'info> {
    #[account(
        mut,
        has_one = authority,
    )]
    pub state: Account<'info, ProgramState>,
    pub current_authority: Signer<'info>,
    pub system_program: Program<'info, System>,
}

# [derive(Accounts)]
pub struct AcceptAuthority<'info> {
    #[account(
        mut,
        constraint = state.pending_authority == Some(new_authority.key())
    )]
    pub state: Account<'info, ProgramState>,
    pub new_authority: Signer<'info>,
}

# [account]
pub struct ProgramState {
    pub authority: Pubkey,
    pub pending_authority: Option<Pubkey>,
    // 其他相关的程序状态字段
}
```

在这个例子中，**ProgramState** 账户结构保存当前的权限以及一个可选的 **pending_authority**。**NominateAuthority** 上下文确保当前权限所有者签署交易，允许他们提名一个新的权限。**AcceptAuthority** 上下文检查 **pending_authority** 是否与交易的签署者匹配，允许他们接受并成为新的权限所有者。通过这个流程，确保了程序内的权限转移是安全且受控的。

## Bump Seed 标准化
### 漏洞
Bump seed 标准化指的是在推导 PDA（程序派生地址，Program Derived Addresses）时使用最高有效的 bump seed（即，标准 bump）。使用标准 bump 是一种确定性且安全的方法，可以根据一组种子找到一个地址。未能使用标准 bump 可能导致漏洞，例如恶意行为者创建或操纵 PDA，进而破坏程序逻辑或数据完整性。

### 示例场景
考虑一个程序用于创建唯一的用户档案，每个档案都与一个明确使用 create_program_address 推导的 PDA 关联。该程序允许用户提供的 bump 来创建档案。然而，这存在问题，因为这引入了使用非标准 bump 的风险：

``` rust
pub fn create_profile(ctx: Context<CreateProfile>, user_id: u64, attributes: Vec<u8>, bump: u8) -> Result<()> {
    // 显式使用 create_program_address 和用户提供的 bump 来推导 PDA
    let seeds: &[&[u8]] = &[b"profile", &user_id.to_le_bytes(), &[bump]];
    let (derived_address, _bump) = Pubkey::create_program_address(seeds, &ctx.program_id)?;

    if derived_address != ctx.accounts.profile.key() {
        return Err(ProgramError::InvalidSeeds);
    }

    let profile_pda = &mut ctx.accounts.profile;
    profile_pda.user_id = user_id;
    profile_pda.attributes = attributes;

    Ok(())
}

# [derive(Accounts)]
pub struct CreateProfile<'info> {
    #[account(mut)]
    pub user: Signer<'info>,
    /// 期望是通过 user_id 和用户提供的 bump seed 推导的 PDA 档案账户
    #[account(mut)]
    pub profile: Account<'info, UserProfile>,
    pub system_program: Program<'info, System>,
}

# [account]
pub struct UserProfile {
    pub user_id: u64,
    pub attributes: Vec<u8>,
}
```

在这个场景中，程序使用 create_program_address，结合包含用户提供的 bump 的种子来推导 UserProfile 的 PDA。使用用户提供的 bump 是有问题的，因为它未确保使用标准 bump。这可能让恶意行为者为同一用户ID创建多个具有不同 bump 的 PDA。

### 推荐的缓解措施
为缓解这个问题，我们可以重构示例代码，通过 find_program_address 来推导 PDA，并显式验证 bump seed：

``` rust
pub fn create_profile(ctx: Context<CreateProfile>, user_id: u64, attributes: Vec<u8>) -> Result<()> {
    // 安全地使用 find_program_address 推导 PDA，以确保使用标准 bump
    let seeds: &[&[u8]] = &[b"profile", user_id.to_le_bytes()];
    let (derived_address, bump) = Pubkey::find_program_address(seeds, &ctx.program_id);

    // 将标准 bump 存储在 profile 中，以供将来验证
    let profile_pda = &mut ctx.accounts.profile;
    profile_pda.user_id = user_id;
    profile_pda.attributes = attributes;
    profile_pda.bump = bump;

    Ok(())
}

# [derive(Accounts)]
# [instruction(user_id: u64)]
pub struct CreateProfile<'info> {
    #[account(
        init,
        payer = user,
        space = 8 + 1024 + 1,
        seeds = [b"profile", user_id.to_le_bytes().as_ref()],
        bump
    )]
    pub profile: Account<'info, UserProfile>,
    #[account(mut)]
    pub user: Signer<'info>,
    pub system_program: Program<'info, System>,
}

# [account]
pub struct UserProfile {
    pub user_id: u64,
    pub attributes: Vec<u8>,
    pub bump: u8,
}
```

在这里，使用 **find_program_address** 来推导 PDA，确保使用标准 bump seed 以确保 PDA 的创建是确定且安全的。标准 bump 存储在 **UserProfile** 账户中，允许在后续操作中进行有效和安全的验证。相比 **create_program_address**，我们更倾向于使用 **find_program_address**，因为后者总是使用标准 bump 来创建 PDA。原因是 **find_program_address** 会从 bump 为 255 开始，通过不断递减来调用 **create_program_address**，直到找到有效地址。一旦找到有效地址，函数返回该 PDA 和用于推导它的标准 bump。

Anchor 框架通过其 **seeds** 和 **bump** 约束强制标准化 bump，用于简化 PDA 推导和验证的整个过程，以确保其安全性和确定性。

## 关闭账户
### 漏洞
在程序中不正确地关闭账户会导致多个漏洞，包括 "关闭" 账户被重新初始化或滥用的风险。问题在于未能正确标记账户已关闭或未能防止其在后续交易中被重复使用。此类疏忽可能使恶意行为者利用该账户，从而在程序中执行未授权的操作或访问。

### 示例场景
考虑一个允许用户创建和关闭数据存储账户的程序。该程序通过转移账户中的 lamports 来关闭账户：

``` rust
pub fn close_account(ctx: Context<CloseAccount>) -> ProgramResult {
    let account = ctx.accounts.data_account.to_account_info();
    let destination = ctx.accounts.destination.to_account_info();

    **destination.lamports.borrow_mut() = destination
        .lamports()
        .checked_add(account.lamports())
        .unwrap();
    **account.lamports.borrow_mut() = 0;
    
    Ok(())
}

# [derive(Accounts)]
pub struct CloseAccount<'info> {
    #[account(mut)]
    pub data_account: Account<'info, Data>,
    #[account(mut)]
    pub destination: AccountInfo<'info>,
}

# [account]
pub struct Data {
    data: u64,
}
```

这是有问题的，因为该程序未能清除账户的数据或标记其已关闭。仅仅转移账户的 lamports 并不能真正关闭账户。

### 推荐的缓解措施

为了缓解这个问题，程序不仅应该转移所有 lamports，还应将账户的数据清零，并使用识别符（例如 **"CLOSED_ACCOUNT_DISCRIMINATOR"**）标记其已关闭。程序还应该实现检查，防止关闭后的账户在将来的交易中被重新使用：

``` rust
use anchor_lang::__private::CLOSED_ACCOUNT_DISCRIMINATOR;
use anchor_lang::prelude::*;
use std::io::Cursor;
use std::ops::DerefMut;

// 其他代码

pub fn close_account(ctx: Context<CloseAccount>) -> ProgramResult {
    let account = ctx.accounts.data_account.to_account_info();
    let destination = ctx.accounts.destination.to_account_info();

    **destination.lamports.borrow_mut() = destination
        .lamports()
        .checked_add(account.lamports())
        .unwrap();
    **account.lamports.borrow_mut() = 0;

    // 清除账户数据
    let mut data = account.try_borrow_mut_data()?;
    for byte in data.deref_mut().iter_mut() {
        *byte = 0;
    }

    // 标记账户为已关闭
    let dst: &mut [u8] = &mut data;
    let mut cursor = Cursor::new(dst);
    cursor.write_all(&CLOSED_ACCOUNT_DISCRIMINATOR).unwrap();

    Ok(())
}

pub fn force_defund(ctx: Context<ForceDefund>) -> ProgramResult {
    let account = &ctx.accounts.account;
    let data = account.try_borrow_data()?;

    if data.len() < 8 || data[0..8] != CLOSED_ACCOUNT_DISCRIMINATOR {
        return Err(ProgramError::InvalidAccountData);
    }

    let destination = ctx.accounts.destination.to_account_info();

    **destination.lamports.borrow_mut() = destination
        .lamports()
        .checked_add(account.lamports())
        .unwrap();
    **account.lamports.borrow_mut() = 0;

    Ok(())
}

# [derive(Accounts)]
pub struct ForceDefund<'info> {
    #[account(mut)]
    pub account: AccountInfo<'info>,
    #[account(mut)]
    pub destination: AccountInfo<'info>,
}

# [derive(Accounts)]
pub struct CloseAccount<'info> {
    #[account(mut)]
    pub data_account: Account<'info, Data>,
    #[account(mut)]
    pub destination: AccountInfo<'info>,
}

# [account]
pub struct Data {
    data: u64,
}
```

然而，清零数据并添加关闭的识别符还不够。用户可以在指令结束前为账户重新注入 lamports，从而使该账户处于一个无法使用但也无法被垃圾回收的“怪异状态”。因此，我们添加了 force_defund 函数，以解决这种边缘情况。现在，任何人都可以为关闭的账户进行退款。

Anchor 框架通过 **#[account(close = destination)]** 约束简化了这一过程，自动完成 lamports 转移、数据清零以及账户关闭标记操作。

## 重复的可变账户

### 漏洞

重复的可变账户是指在指令中同一账户作为可变参数被多次传递。这种情况发生在一个指令需要两个相同类型的可变账户时。恶意行为者可以传递相同的账户两次，导致账户被意外地重复修改（例如，覆盖数据）。这种漏洞的严重性取决于具体的场景。

### 示例场景
考虑一个设计用于根据用户的链上活动奖励用户的程序。该程序有一个指令，用于更新两个账户的余额：一个是奖励账户，一个是奖金账户。用户应该在一个账户中收到标准奖励，在另一个账户中根据特定标准获得奖金：

``` rust
pub fn distribute_rewards(ctx: Context<DistributeRewards>, reward_amount: u64, bonus_amount: u64) -> Result<()> {
    let reward_account = &mut ctx.accounts.reward_account;
    let bonus_reward = &mut ctx.accounts.bonus_account;

    // 计划分别增加奖励和奖金账户的余额
    reward_account.balance += reward_amount;
    bonus_account.balance += bonus_amount;

    Ok(())
}

# [derive(Accounts)]
pub struct DistributeRewards<'info> {
    #[account(mut)]
    reward_account: Account<'info, RewardAccount>,
    #[account(mut)]
    bonus_account: Account<'info, RewardAccount>,
}

# [account]
pub struct RewardAccount {
    pub balance: u64,
}
```

如果恶意行为者为 reward_account 和 bonus_account 提供相同的账户，则该账户的余额将被错误地更新两次。

推荐的缓解措施
为缓解此问题，应在指令逻辑中添加检查，以确保两个账户的公钥不相同：

```
pub fn distribute_rewards(ctx: Context<DistributeRewards>, reward_amount: u64, bonus_amount: u64) -> Result<()> {
    if ctx.accounts.reward_account.key() == ctx.accounts.bonus_account.key() {
        return Err(ProgramError::InvalidArgument.into())
    }

    let reward_account = &mut ctx.accounts.reward_account;
    let bonus_reward = &mut ctx.accounts.bonus_account;

    // 计划分别增加奖励和奖金账户的余额
    reward_account.balance += reward_amount;
    bonus_account.balance += bonus_amount;

    Ok(())
}
```
开发人员可以使用 Anchor 的账户约束功能，使用更明确的检查方法来防止此类问题。这可以通过 #[account] 属性和 constraint 关键字实现：

```
pub fn distribute_rewards(ctx: Context<DistributeRewards>, reward_amount: u64, bonus_amount: u64) -> Result<()> {
    let reward_account = &mut ctx.accounts.reward_account;
    let bonus_reward = &mut ctx.accounts.bonus_account;

    // 计划分别增加奖励和奖金账户的余额
    reward_account.balance += reward_amount;
    bonus_account.balance += bonus_amount;

    Ok(())
}

# [derive(Accounts)]
pub struct DistributeRewards<'info> {
    #[account(
        mut,
        constraint = reward_account.key() != bonus_account.key()
    )]
    reward_account: Account<'info, RewardAccount>,
    #[account(mut)]
    bonus_account: Account<'info, RewardAccount>,
}

# [account]
pub struct RewardAccount {
    pub balance: u64,
}
```

## 抢跑（Frontrunning）

### 漏洞

随着交易捆绑器的流行，抢跑成为了 Solana 上构建的协议应认真对待的问题。抢跑是指恶意行为者通过精心构造的交易，操纵预期值和实际值。

### 示例场景

假设一个协议处理产品的购买和竞标，卖家的定价信息存储在一个名为 SellInfo 的账户中：

``` rust
# [derive(Accounts)]
pub struct SellProduct<'info> {
  product_listing: Account<'info, ProductListing>,
  sale_token_mint: Account<'info, Mint>,
  sale_token_destination: Account<'info, TokenAccount>,
  product_owner: Signer<'info>,
  purchaser_token_source: Account<'info, TokenAccount>,
  product: Account<info, Product>
}

# [derive(Accounts)]
pub struct PurchaseProduct<'info> {
  product_listing: Account<'info, ProductListing>,
  token_destination: Account<'info, TokenAccount>,
  token_source: Account<'info, TokenAccount>,
  buyer: Signer<'info>,
  product_account: Account<'info, Product>,
  token_mint_sale: Account<'info, Mint>,
}

# [account]
pub struct ProductListing {
  sale_price: u64,
  token_mint: Pubkey,
  destination_token_account: Pubkey,
  product_owner: Pubkey,
  product: Pubkey,
}
```

为了购买列出的产品，买家必须传递与产品相关的 ProductListing 账户。但如果卖家可以更改其列表的 sale_price 会怎样呢？

``` rust
pub fn change_sale_price(ctx: Context<ChangeSalePrice>, new_price: u64) -> Result<()> {...}
```

这将为卖家提供一个抢跑的机会，特别是如果买方的购买交易中没有包含 **expected_price** 检查，以确保他们不会支付超过预期的价格。如果买家提交了购买产品的交易，卖家可以调用 change_sale_price，并通过 Jito 确保该交易在买方的交易之前包含在内。恶意卖家可以将 ProductListing 账户中的价格更改为极高的金额，而买家在不知情的情况下支付了远高于预期的价格。

### 推荐的缓解措施

一个简单的解决方案是在购买过程中包括 expected_price 检查，防止买家为其想要购买的产品支付超过预期的价格：

``` rust
pub fn purchase_product(ctx: Context<PurchaseProduct>, expected_price: u64) -> Result<()> {
  assert!(ctx.accounts.product_listing.sale_price <= expected_price);
  ...
}
```

## 不安全的初始化

与部署到 EVM 的合约不同，Solana 程序在部署时不会自动使用构造函数来设置状态变量。相反，它们需要手动初始化（通常通过一个名为 **initialize** 的函数）。初始化函数通常会设置诸如程序的权限或创建账户等数据，这些账户构成了程序的基础（即一个中央状态账户或类似的东西）。

由于初始化函数是手动调用的，而不是在程序部署时自动执行的，必须由程序开发团队控制的已知地址调用此指令。否则，攻击者可能会抢先执行初始化，将程序使用的账户设为攻击者控制的账户。

一个常见的做法是使用程序的 upgrade_authority 作为调用 initialize 函数的授权地址，如果该程序有 upgrade_authority。

### 不安全示例及缓解措施

``` rust
pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
  ctx.accounts.central_state.authority = authority.key();
  ...  
}

# [derive(Accounts)]
pub struct Initialize<'info> {
  authority: Signer<'info>,
  #[account(mut,
    init,
    payer = authority,
    space = CentralState::SIZE,
    seeds = [b"central_state"],
    bump
  )]
  central_state: Account<'info, CentralState>,
  ...
}

# [account]
pub struct CentralState {
  authority: Pubkey,
  ...
}
```

上面的示例是一个简化的 initialize 函数，用于设置 CentralState 账户的权限为调用指令者的账户。然而，任何账户都可以调用 initialize！如前所述，安全初始化函数的常见方式是使用程序的 upgrade_authority，在程序部署时就已知。

下面是 Anchor 文档中的一个示例，使用 constraint 确保只有程序的 upgrade_authority 可以调用 initialize：

``` rust
use anchor_lang::prelude::*;
use crate::program::MyProgram;

declare_id!("Cum9tTyj5HwcEiAmhgaS7Bbj4UczCwsucrCkxRECzM4e");

# [program]
pub mod my_program {
    use super::*;

    pub fn set_initial_admin(
        ctx: Context<SetInitialAdmin>,
        admin_key: Pubkey
    ) -> Result<()> {
        ctx.accounts.admin_settings.admin_key = admin_key;
        Ok(())
    }

    pub fn set_admin(...){...}

    pub fn set_settings(...){...}
}

# [account]
# [derive(Default, Debug)]
pub struct AdminSettings {
    admin_key: Pubkey
}

# [derive(Accounts)]
pub struct SetInitialAdmin<'info> {
    #[account(init, payer = authority, seeds = [b"admin"], bump)]
    pub admin_settings: Account<'info, AdminSettings>,
    #[account(mut)]
    pub authority: Signer<'info>,
    #[account(constraint = program.programdata_address()? == Some(program_data.key()))]
    pub program: Program<'info, MyProgram>,
    #[account(constraint = program_data.upgrade_authority_address == Some(authority.key()))]
    pub program_data: Account<'info, ProgramData>,
    pub system_program: Program<'info, System>,
}
```

## 精度丧失

### 漏洞

尽管精度丧失看似微不足道，但它可能对程序构成重大威胁。它可能导致计算错误、套利机会以及意外的程序行为。

在算术运算中，精度丧失是常见的错误来源。在 Solana 程序中，尽可能推荐使用定点运算。这是因为程序仅支持 Rust 浮点运算的有限子集。如果程序尝试使用不支持的浮点运算，运行时将返回一个未解析的符号错误。此外，浮点运算需要比整数运算更多的指令。由于需要精确处理大量代币和小数金额，使用定点运算可能会加剧精度丧失的问题。

### 除法后的乘法

虽然结合律对大多数数学运算都适用，但其在计算机算术中的应用可能导致意外的精度丧失。一个经典的例子是在执行除法后的乘法时会导致精度丧失，这与先进行乘法再除法的结果不同。例如，考虑以下表达式：**(a / c) * b** 和 **(a * b) / c**。从数学上看，这些表达式是结合的——它们应该产生相同的结果。然而，在 Solana 和定点运算的上下文中，运算顺序非常重要。如果首先执行除法 **(a / c)**，商可能会在乘以 b 之前被向下舍入，从而导致较小的结果。相反，先进行乘法 **(a * b)** 再除以 **c** 可能会保留更多的原始精度。这种差异可能导致计算错误，进而导致程序行为异常和/或套利机会。

### 饱和算术函数

**饱和算术函数（saturating_*）**通过将值限制在其最大或最小值来防止溢出和下溢，但如果意外达到该限制，它们可能会导致细微的错误和精度丧失。这种情况发生在程序的逻辑假设饱和处理可以保证准确结果时，但忽略了可能的精度或准确性损失。

例如，假设有一个程序根据用户在特定时间段内交易的代币数量来计算并分配奖励：

``` rust
pub fn calculate_reward(transaction_amount: u64, reward_multiplier: u64) -> u64 {
    transaction_amount.saturating_mul(reward_multiplier)
}
```

假设 transaction_amount 是 100,000 个代币，reward_multiplier 是每笔交易 100 个代币。将两者相乘会超过 u64 类型的最大值。这意味着它们的乘积将被限制在 u64 的上限，从而导致严重的精度丧失，最终对用户的奖励计算不足。

### 舍入错误

在编程中，舍入操作是常见的精度丧失来源。舍入方法的选择会显著影响计算的准确性以及 Solana 程序的行为。try_round_u64() 函数会将小数值四舍五入到最接近的整数。向上舍入的问题在于它可能人为地增加值，导致实际计算和预期计算之间出现差异。

假设一个 Solana 程序根据市场情况将抵押品转换为流动性。程序使用 try_round_u64() 来舍入除法运算的结果：

``` rust
pub fn collateral_to_liquidity(&self, collateral_amount: u64) -> Result<u64, ProgramError> {
    Decimal::from(collateral_amount)
        .try_div(self.0)?
        .try_round_u64()
}
```

在这种情况下，向上舍入可能会导致发行的流动性代币超过抵押品数量。这种差异可能被恶意用户利用，通过有利的舍入结果进行套利攻击，从协议中提取价值。为了缓解这一问题，使用 try_floor_u64() 将结果舍入到最接近的整数。这种方法可以最大程度地减少人为增加值的风险，确保舍入不会让用户获得系统的额外好处。此外，还可以实现逻辑来处理舍入可能影响结果的场景。这可能包括为舍入决策设置特定阈值，或者根据值的大小应用不同的逻辑。

## 缺少所有权检查

### 漏洞

所有权检查对于验证某个账户在交易或操作中属于预期的程序非常重要。账户包括一个 owner 字段，该字段指示有权对账户数据进行写入的程序。这个字段可以确保只有授权的程序可以修改账户的状态。此外，这个字段还可以用于确保传入指令的账户属于预期的程序。缺少所有权检查可能导致严重漏洞，包括未经授权的资金转移和特权操作的执行。

### 示例场景

假设一个程序功能允许管理员从一个金库中提取资金。该功能接受一个配置账户（config），并使用其 admin 字段检查提供的管理员账户的公钥是否与配置账户中存储的公钥相同。然而，它没有验证配置账户的所有权，默认认为它是可信的：

``` rust
pub fn admin_token_withdraw(program_id: &Pubkey, accounts: &[AccountInfo], amount: u64) -> ProgramResult {
    // 账户设置

    if config.admin != admin.pubkey() {
        return Err(ProgramError::InvalidAdminAccount)
    }

    // 转账逻辑
}
```

恶意用户可以利用这一点，提供一个他们控制的配置账户，并使其 admin 字段匹配，从而欺骗程序执行提款操作。

### 推荐的缓解措施

为了缓解这一问题，需要进行所有权检查以验证账户的 owner 字段：

``` rust
pub fn admin_token_withdraw(program_id: &Pubkey, accounts: &[AccountInfo], amount: u64) -> ProgramResult {
    // 账户设置

    if config.admin != admin.pubkey() {
        return Err(ProgramError::InvalidAdminAccount)
    }

    if config.owner != program_id {
        return Err(ProgramError::InvalidConfigAccount)
    }

    // 转账逻辑
}
```

Anchor 简化了这一检查，使用 **Account** 类型。**Account<'info, T>** 是 **AccountInfo** 的包装，它验证程序的所有权并将底层数据反序列化为 T（即指定的账户类型）。这使得开发者可以轻松使用 Account<'info, T> 来验证账户的所有权。开发者还可以使用 #[account] 属性为给定账户添加 Owner 特性。该特性定义了预期拥有账户的地址。此外，开发者还可以使用 owner 限制来定义应拥有某个账户的程序，特别是在编写期望某个账户来自不同程序派生的 PDA 指令时。**owner** 限制的定义格式为 **#[account(owner = <expr>)]**，其中 **<expr>** 是任意表达式。

### 只读账户

确保程序执行上下文中指定为只读的账户是合法的也同样重要。恶意用户可能传递包含任意或伪造数据的账户，取代合法账户。这可能导致意外或有害的程序行为。开发者仍然应执行检查，以确保程序需要读取的账户是真实的、未被篡改。这可以通过验证账户的地址是否与已知值匹配，或确认账户的所有者是否符合预期，尤其是对于 sysvars（即只读系统账户，如 Clock 或 EpochSchedule）。可以通过 get() 方法访问 sysvars，这不需要任何手动地址或所有权检查。这是一种更安全的访问这些账户的方法；然而，并非所有 sysvars 都支持 get() 方法。在这种情况下，可以使用它们的公共地址访问它们。

## 缺少签名检查

### 漏洞

交易通过钱包的私钥签名，以确保身份验证、完整性、不可否认性以及对特定交易的授权。通过要求交易使用发送者的私钥进行签名，Solana 的运行时可以验证交易是由正确的账户发起的，并且没有被篡改。该机制是去中心化网络信任机制的基础。如果没有此验证，任何提供正确账户作为参数的账户都可以执行交易。这可能导致未经授权的访问特权信息、资金或功能。该漏洞源于未能在执行某些特权功能之前验证操作是否由适当账户的私钥签名。

### 示例场景

假设有以下函数：
``` rust
pub fn update_admin(program_id: &Pubkey, accounts &[AccountInfo]) -> ProgramResult {
    let account_iter = &mut accounts.iter();
    let config = ConfigAccount::unpack(next_account_info(account_iter)?)?;
    let admin = next_account_info(account_iter)?;
    let new_admin = next_account_info(account_iter)?;

    if admin.pubkey() != config.admin {
        return Err(ProgramError::InvalidAdminAccount);
    }

    config.admin = new_admin.pubkey();

    Ok(())
}
```

该函数的意图是更新程序的管理员。它包括一个检查，以确保当前管理员发起了该操作，这是一个良好的访问控制措施。然而，该函数未能验证当前管理员的私钥是否签署了交易。因此，任何人都可以调用该函数并传递正确的管理员账户，使 admin.pubkey() = config.admin，无论调用该函数的账户是否真的是当前管理员。这允许恶意用户通过将其账户传递为新的管理员，直接绕过当前管理员的授权。

### 推荐的缓解措施

程序必须包含检查，以验证账户是否由适当的钱包签名。这可以通过检查交易中涉及的账户的 AccountInfo::is_signer 字段来实现。通过检查执行特权操作的账户是否设置了 is_signer 标志为 true，程序可以强制只有授权账户才能执行某些操作。

更新后的代码示例如下：

``` rust
pub fn update_admin(program_id: &Pubkey, accounts &[AccountInfo]) -> ProgramResult {
    let account_iter = &mut accounts.iter();
    let config = ConfigAccount::unpack(next_account_info(account_iter)?)?;
    let admin = next_account_info(account_iter)?;
    let new_admin = next_account_info(account_iter)?;

    if admin.pubkey() != config.admin {
        return Err(ProgramError::InvalidAdminAccount);
    }

    // 添加管理员签名检查
    if !admin.is_signer {
        return Err(ProgramError::NotSigner);
    }

    config.admin = new_admin.pubkey();

    Ok(())
}
```
Anchor 通过 Signer<'info> 账户类型简化了整个过程。

## 溢出和下溢

### 漏洞

整数是没有小数部分的数字。Rust 将整数存储为固定大小的变量。这些变量通过它们的符号（有符号或无符号）以及它们在内存中占据的空间来定义。例如，u8 类型表示一个无符号整数，它占据 8 位空间，能够保存从 0 到 255 的值。存储超出该范围的值将导致整数溢出或下溢。整数溢出是指变量超过其最大容量并回绕到最小值。整数下溢是指变量跌破其最小容量并回绕到最大值。

在调试模式下编译时，Rust 包含对整数溢出和下溢的检查。如果检测到这种情况，这些检查将在运行时导致程序 panic。然而，当使用 --release 标志在发布模式下编译时，Rust 不包括这些检查。这种行为可能引入细微的漏洞，因为溢出或下溢会在不被检测的情况下发生。伯克利数据包过滤器（BPF）工具链是 Solana 开发环境的重要组成部分，因为它将 Solana 程序编译为 BPF 字节码以进行部署。问题在于它默认以发布模式编译程序。因此，Solana 程序容易受到整数溢出和下溢的影响。

### 示例场景

攻击者可以利用发布模式下的静默溢出/下溢行为，尤其是处理代币余额的函数。考虑以下示例：

``` rust
pub fn process_instruction(
    _program_id: & Pubkey,
    accounts: [&AccountInfo],
    _instruction_data: &[u8],
) -> ProgramResult {
    let account_info_iter = &mut accounts.iter();
    let account = next_account_info(account_info_iter)?;

    let mut balance: u8 = account.data.borrow()[0];
    let tokens_to_subtract: u8 = 100;

    balance = balance - tokens_to_subtract;

    account.data.borrow_mut()[0] = balance;
    msg!("Updated balance to {}", balance);
    
    Ok(())
}
```

该函数假设余额存储在第一个字节中，以简化代码。它获取账户的余额，并从中减去 tokens_to_subtract。如果用户的余额小于 tokens_to_subtract，则会导致下溢。例如，余额为 10 个代币的用户将下溢为 165 个代币的总余额。

## 推荐的缓解措施

### overflow-checks

最简单的缓解方法是在项目的 Cargo.toml 文件中将 overflow-checks 设置为 true。此时，Rust 将在编译器中添加溢出和下溢检查。然而，添加溢出和下溢检查会增加交易的计算成本。在需要优化计算的情况下，可能更有利于将 overflow-checks 设置为 false。

### checked_ 算术*

在整个程序中，使用 Rust 的 checked_* 算术函数对每个整数类型进行战略性检查，以防止溢出和下溢。这些函数在发生溢出或下溢时将返回 None，从而允许程序优雅地处理错误。例如，您可以将之前的代码重构为：

``` rust
pub fn process_instruction(
    _program_id: & Pubkey,
    accounts: [&AccountInfo],
    _instruction_data: &[u8],
) -> ProgramResult {
    let account_info_iter = &mut accounts.iter();
    let account = next_account_info(account_info_iter)?;

    let mut balance: u8 = account.data.borrow()[0];
    let tokens_to_subtract: u8 = 100;

    match balance.checked_sub(tokens_to_subtract) {
        Some(new_balance) => {
            account.data.borrow_mut()[0] = new_balance;
            msg!("Updated balance to {}", new_balance);
        },
        None => {
            return Err(ProgramError::InsufficientFunds);
        }
    }

    Ok(())
}
```

在修订后的示例中，checked_sub 用于从 balance 中减去 tokens_to_subtract。因此，如果余额足够覆盖减法运算，checked_sub 将返回 Some(new_balance)。程序继续安全地更新账户余额并记录下来。然而，如果减法会导致下溢，checked_sub 将返回 None，我们可以通过返回错误来处理这一情况。

### Checked Math 宏

Checked Math 是一种过程宏，用于检查数学表达式的属性，而无需更改这些表达式的大部分内容。`checked_*`算术函数的问题在于失去了数学符号。相反，必须使用像 `a.checked_add(b).unwrap()` 这样麻烦的方法来代替 `a + b`。例如，如果我们想写 `(x * y) + z`，使用 checked 算术函数，我们会写 **x.checked_mul(y).unwrap().checked_add(z).unwrap()**。

相反，使用 Checked Math 宏的表达式如下所示：

``` rust
use checked_math::checked_math as cm;

cm!((x * y) + z).unwrap()
```
这种写法更方便，保留了表达式的数学符号，并且只需要一个 .unwrap()。这是因为宏将普通的数学表达式转换为在任意一个步骤返回 None 时返回 None 的表达式。如果成功，则返回 Some(_)，这就是我们在最后解包表达式的原因。

### 类型转换

类似地，使用 **as** 关键字在整数类型之间进行类型转换而不进行适当检查，可能会引入整数溢出或下溢漏洞。这是因为转换可能会以非预期的方式截断或扩展值。当从较大的整数类型转换为较小的整数类型（例如，从 u64 转换为 u32）时，Rust 会截断不适合目标类型的原始值的较高位。当从较小的整数类型转换为较大的整数类型（例如从 i16 转换为 i32）时，Rust 会扩展该值。这对无符号类型是直接的，但对有符号整数来说，可能会导致符号扩展，进而引入意外的负值。

### 推荐的缓解措施

使用 Rust 的安全类型转换方法来缓解此漏洞。这包括 try_from 和 from 方法。使用 try_from 返回一个 Result 类型，允许显式处理无法适应目标类型的情况。Rust 的 from 方法可用于安全的隐式转换，这种转换在转换时不会丢失数据（例如，从 u8 转换为 u32）。例如，假设程序需要安全地将 u64 代币金额转换为 u32 类型进行处理，代码可以这样编写：

``` rust
pub fn convert_token_amount(amount: u64) -> Result<u32, ProgramError> {
    u32::try_from(amount).map_err(|_| ProgramError::InvalidArgument)
}
```

在这个示例中，如果 amount 超过了 u32 可以保存的最大值（即 4,294,967,295），转换将失败，程序会返回错误。这可以防止潜在的溢出/下溢问题。

## PDA 共享

### 漏洞

PDA 共享是指在多个权限域或角色之间使用相同的 PDA（程序派生地址），这是常见的漏洞。这可能会让恶意用户通过滥用作为签名者的 PDA 而没有适当的访问控制检查，来访问不属于他们的数据或资金。

### 示例场景

假设有一个程序用于促进代币的质押和奖励分发。该程序使用单一 PDA 将代币转入指定池并提取奖励。PDA 是通过静态种子（例如质押池的名称）派生的，这使其在所有操作中是通用的：

``` rust
pub fn stake_tokens(ctx: Context<StakeTokens>, amount: u64) -> ProgramResult {
    // 质押代币逻辑
    Ok(())
}

pub fn withdraw_rewards(ctx: Context<WithdrawRewards>, amount: u64) -> ProgramResult {
    // 提取奖励逻辑
    Ok(())
}

# [derive(Accounts)]
pub struct StakeTokens<'info> {
    #[account(
        mut,
        seeds = [b"staking_pool_pda"],
        bump
    )]
    staking_pool: AccountInfo<'info>,
    // 其他与质押相关的账户
}

# [derive(Accounts)]
pub struct WithdrawRewards<'info> {
    #[account(
        mut,
        seeds = [b"staking_pool_pda"],
        bump
    )]
    rewards_pool: AccountInfo<'info>,
    // 其他与奖励提取相关的账户
}
```

这是有问题的，因为质押代币和奖励提取功能都依赖于相同的由 staking_pool_pda 派生的 PDA。这可能让用户操纵合同，进行未经授权的奖励提取或质押操控。

### 推荐的缓解措施

为了缓解这一漏洞，应该为不同的功能使用不同的 PDA。确保每个 PDA 仅服务于特定的上下文，并且使用唯一的、与操作相关的种子派生 PDA：

``` rust
pub fn stake_tokens(ctx: Context<StakeTokens>, amount: u64) -> ProgramResult {
    // 质押代币逻辑
    Ok(())
}

pub fn withdraw_rewards(ctx: Context<WithdrawRewards>, amount: u64) -> ProgramResult {
    // 提取奖励逻辑
    Ok(())
}

# [derive(Accounts)]
pub struct StakeTokens<'info> {
    #[account(
        mut,
        seeds = [b"staking_pool", &staking_pool.key().as_ref()],
        bump
    )]
    staking_pool: AccountInfo<'info>,
    // 其他与质押相关的账户
}

# [derive(Accounts)]
pub struct WithdrawRewards<'info> {
    #[account(
        mut,
        seeds = [b"rewards_pool", &rewards_pool.key().as_ref()],
        bump
    )]
    rewards_pool: AccountInfo<'info>,
    // 其他与奖励提取相关的账户
}
```

在上述示例中，质押代币和提取奖励的 PDA 是使用不同的种子（分别为 **staking_pool** 和 **rewards_pool**），并结合特定账户的公钥派生的。这确保了 PDA 与其预期功能唯一绑定，减少了未经授权操作的风险。

## 剩余账户

### 漏洞

ctx.remaining_accounts 提供了一种将未在 Accounts 结构中初始指定的附加账户传递给函数的方法。这为开发者提供了更大的灵活性，允许他们处理需要动态数量账户的场景（例如，处理可变数量的用户或与不同的程序交互）。然而，这种增加的灵活性带来了一个问题：通过 ctx.remaining_accounts 传递的账户不会经过与初始定义的账户相同的验证。由于 ctx.remaining_accounts 不会验证传递的账户，恶意用户可以利用这一点传递程序不打算与之交互的账户，导致未经授权的操作或访问。

### 示例场景

假设一个奖励程序使用 ctx.remaining_accounts 动态接收用户 PDA 并计算奖励：

``` rust
pub fn calculate_rewards(ctx: Context<CalculateRewards>) -> Result<()> {
    let rewards_account = &ctx.accounts.rewards_account;
    let authority = &ctx.accounts.authority;

    // 遍历通过 ctx.remaining_accounts 传递的账户
    for user_pda_info in ctx.remaining_accounts.iter() {
        // 检查用户活动并计算奖励的逻辑
    }

    // 分配计算的奖励的逻辑

    Ok(())
}

# [derive(Accounts)]
pub struct CalculateRewards<'info> {
    #[account(mut)]
    pub rewards_account: Account<'info, RewardsAccount>,
    pub authority : Signer<'info>,
}

# [account]
pub struct RewardsAccount {
    pub total_rewards: u64,
    // 其他相关字段
}
```

问题在于，这里没有对通过 ctx.remaining_accounts 传递的账户进行显式检查，也就是说，无法确保只有合法且符合资格的用户账户才能参与奖励的计算和分配。恶意用户可以通过传递自己创建的账户或不属于他们的账户，获取超出他们应得的奖励。

### 推荐的缓解措施

为了缓解这一漏洞，开发者应在函数内手动验证每个账户的有效性。这包括检查账户的所有者，以确保其与预期用户匹配，并验证账户中的任何相关数据。通过加入这些手动检查，开发者可以利用 ctx.remaining_accounts 的灵活性，同时减少未经授权的访问或操控的风险。

## Rust 特定错误

Rust 是 Solana 程序开发的主要语言。在 Rust 中开发带来了独特的挑战和考虑因素，特别是在处理不安全代码和 Rust 特有的错误时。了解 Rust 的注意事项有助于开发安全、高效且可靠的程序。

### Unsafe Rust

Rust 以其内存安全保证而闻名，这种安全性通过严格的所有权和借用系统来实现。然而，这些保证有时会给开发者带来限制，因此 Rust 提供了 unsafe 关键字，允许绕过这些安全检查。unsafe Rust 主要用于四个上下文：

- 不安全函数：执行可能违反 Rust 安全保证的操作的函数必须标记为 unsafe。例如：unsafe fn dangerous_function() {}。
- 不安全代码块：允许不安全操作的代码块。例如：unsafe { // 不安全操作 }。
- 不安全特性：包含某些编译器无法验证的不变量特性的特性。例如：unsafe trait BadTrait {}。
- 不安全特性实现：实现这些特性时也必须标记为 unsafe。例如：unsafe impl UnsafeTrait for UnsafeType {}。

Unsafe Rust 的存在是因为静态分析具有保守性。当编译器尝试确定代码是否保持某些保证时，它宁可拒绝一些有效的代码，也不愿接受一些无效的代码。尽管代码可能运行得非常正常，如果编译器没有足够的信息来确信代码是否符合 Rust 的安全保证，它将拒绝这些代码。Unsafe 代码允许开发者在自担风险的情况下绕过这些检查。此外，计算机硬件本质上是不安全的。为了进行低级编程，开发者必须能够进行不安全的操作。

使用 unsafe 关键字，开发者可以：

- 解引用原始指针：允许直接访问内存中的原始指针，这些指针可能指向任意内存位置，可能不包含有效数据。
- 调用不安全函数：这些函数可能不遵守 Rust 的安全保证，可能导致未定义行为。
- 访问可变的静态变量：全局可变状态可能导致数据竞争。
缓解 unsafe Rust 的最佳方式是尽量减少使用 unsafe 代码块。如果确实需要使用 unsafe 代码，确保其有充分的文档说明，定期审计，并尽可能将其封装在一个安全的抽象层中，供程序的其他部分使用。

### Panic 和错误管理

当 Rust 程序遇到不可恢复的错误并终止执行时，就会发生 panic。Panic 通常用于那些不应该被捕获的意外错误。在 Solana 程序的上下文中，panic 可能会导致意外行为，因为运行时期望程序能够优雅地处理错误，而不是崩溃。

当 panic 发生时，Rust 会开始展开堆栈并逐步清理。这会返回一个堆栈跟踪，包含有关错误的详细信息。这可能会向攻击者提供有关底层文件结构的信息。虽然这不直接适用于 Solana 程序，但程序使用的依赖项可能会受到此类攻击的影响。确保依赖项保持最新，并使用不包含已知漏洞的版本。

常见的 panic 场景包括：

- 除以零：Rust 在尝试除以零时会发生 panic。因此，在执行除法之前，始终检查除数是否为零。
- 数组索引越界：访问超出数组边界的索引会导致 panic。为了缓解此问题，使用返回 Option 类型的方法（如 get）来安全地访问数组元素。
- 解包 None 值：在持有 None 值的 Option 上调用 .unwrap() 会导致 panic。始终使用模式匹配或方法（如 unwrap_or、unwrap_or_else 或在返回 Result 的函数中使用 ? 运算符）来避免 panic。
为了缓解与 panic 相关的问题，必须避免会引发 panic 的操作，验证所有输入和可能导致问题操作的条件，并使用 Result 和 Option 类型进行错误处理。此外，编写全面的程序测试有助于在部署之前发现和解决潜在的 panic 场景。

## 种子冲突

### 漏洞 
种子冲突发生在生成 PDA（程序派生地址）时使用了不同的输入（即种子和程序 ID），但结果是相同的 PDA 地址。当 PDA 在程序中用于不同用途时，这可能会导致意外行为，包括拒绝服务攻击或完全妥协。

### 示例场景

假设有一个用于各种提案和倡议的去中心化投票平台程序。每个提案或倡议的投票会话都有一个唯一的标识符，用户提交投票。该程序使用 PDA 处理投票会话和单个投票：

``` rust
// 创建投票会话 PDA
# [derive(Accounts)]
# [instruction(session_id: String)]
pub struct CreateVotingSession<'info> {
    #[account(mut)]
    pub organizer: Signer<'info>,
    #[account(
        init,
        payer = organizer,
        space = 8 + Product::SIZE,
        seeds = [b"session", session_id.as_bytes()],
    )]
    pub voting_session: Account<'info, VotingSession>,
    pub system_program: Program<'info, System>,
}

// 提交投票 PDA
# [derive(Accounts)]
# [instruction(session_id: String)]
pub struct SubmitVote<'info> {
    #[account(mut)]
    pub voter: Signer<'info>,
    #[account(
        init,
        payer = voter,
        space = 8 + Vote::SIZE,
        seeds = [session_id.as_bytes(), voter.key().as_ref()]
    )]
    pub vote: Account<'info, Vote>,
    pub system_program: Program<'info, System>,
}
```

在这种情况下，攻击者可能会尝试巧妙地创建一个投票会话，当与静态种子 "session" 组合时，结果会与另一个投票会话生成的 PDA 地址相同。通过故意制造 PDA 冲突，可能会破坏平台的操作，例如阻止合法的投票或阻止新提案被添加到平台上，因为 Solana 运行时无法区分冲突的 PDA。

### 推荐的缓解措施

为了缓解种子冲突的风险，开发者可以：

为同一程序中的不同 PDA 使用唯一的种子前缀。这种方法可以帮助确保 PDA 保持唯一性。
使用唯一标识符（例如时间戳、用户 ID、随机数值），以保证每次生成唯一的 PDA。
编程验证生成的 PDA 不会与现有的 PDA 冲突。

## 类型伪装

### 漏洞

类型伪装是指由于在反序列化过程中缺乏类型检查，导致一个账户类型被误认为是另一个类型。这可能导致未经授权的操作或数据损坏，因为程序会根据错误的账户角色或权限执行操作。始终在反序列化时显式检查账户的预期类型。

### 示例场景

假设有一个程序根据用户的角色管理对管理员操作的访问。每个用户账户都包含一个角色鉴别符，用于区分普通用户和管理员。该程序包含一个仅供管理员使用的更新管理员设置的功能。然而，该程序没有检查账户的鉴别符，而是在没有确认账户是否为管理员的情况下反序列化用户账户数据：

``` rust
pub fn update_admin_settings(ctx: Context<UpdateSettings>) -> ProgramResult {
    // 未检查鉴别符直接反序列化
    let user = User::try_from_slice(&ctx.accounts.user.data.borrow()).unwrap();

    // 敏感更新逻辑

    msg!("Admin settings updated by: {}", user.authority)
    Ok(())
}

# [derive(Accounts)]
pub struct UpdateSettings<'info> {
    user: AccountInfo<'info>
}

# [derive(BorshSerialize, BorshDeserialize)]
pub struct User {
    authority: Pubkey,
}
```

问题在于，update_admin_settings 在未检查账户角色鉴别符的情况下反序列化了传递的用户账户，部分原因是 User 结构体缺少鉴别符字段！

### 推荐的缓解措施

为了缓解此问题，开发者可以在 User 结构体中引入一个鉴别符字段，并在反序列化过程中进行验证：

``` rust
pub fn update_admin_settings(ctx: Context<UpdateSettings>) -> ProgramResult {
    let user = User::try_from_slice(&ctx.accounts.user.data.borrow()).unwrap();

    // 验证用户的鉴别符
    if user.discriminant != AccountDiscriminant::Admin {
        return Err(ProgramError::InvalidAccountData.into())
    }
    
    // 敏感更新逻辑

    msg!("Admin settings updated by: {}", user.authority)
    Ok(())
}

# [derive(Accounts)]
pub struct UpdateSettings<'info> {
    user: AccountInfo<'info>
}

# [derive(BorshSerialize, BorshDeserialize)]
pub struct User {
    discriminant: AccountDiscriminant,
    authority: Pubkey,
}

# [derive(BorshSerialize, BorshDeserialize, PartialEq)]
pub enum AccountDiscriminant {
    Admin,
    // 其他账户类型
}
```

Anchor 简化了类型伪装漏洞的缓解措施，通过自动管理账户类型的鉴别符。通过 Account<'info, T> 包装器，Anchor 在反序列化过程中自动检查鉴别符，从而确保类型安全。这样，开发者可以更多地关注程序的业务逻辑，而不必手动实现各种类型检查。

## 结论

程序安全性的重要性不容忽视。本文涵盖了常见漏洞的各个方面，从 Rust 特定错误到 Anchor 的 realloc 方法的复杂性。掌握这些漏洞及程序安全的过程是一个持续学习、适应和协作的过程。作为开发者，我们对安全的承诺不仅仅是保护资产，更是培养信任，确保我们应用程序的完整性，并为 Solana 的发展和稳定性做出贡献。

如果你读到这里，非常感谢！请务必在下方输入你的电子邮件地址，以便你不会错过有关 Solana 新动态的更新。准备好深入了解了吗？探索 Helius 博客上的最新文章，继续你的 Solana 之旅吧。

## 附加资源

- 如何成为智能合约审计员
- Immunefi
- Neodyme 的 Solana 安全研讨会
- Sealevel 攻击
- Solana: 审计员的简介
- Unsafe Rust