# 如何在 Solana 上获取代币持有者

在本指南中，我们将了解如何获取像 USDC 这样的可替代代币的所有持有者。这在您希望跟踪代币持有者或通过空投奖励持有者的情况下非常有用。

## 概述

首先，让我们来看看代币，特别是不可替代代币在 Solana 上是如何运作的。当开发者创建一个代币时，他们使用代币程序创建一个铸造账户（mint account）。这个铸造账户保存了关于特定代币的信息，比如名称、代币地址和图像。一旦创建了铸造账户，代币可以被铸造并存储在一个代币账户（token account）中。代币账户是一个持有关于特定代币信息的账户，且这些代币是由特定地址拥有的。这将包括铸造地址、所有者地址以及账户中的特定代币数量。例如，持有一些 USDC（一个 [SPL 代币](https://spl.solana.com/)）的地址将有一个 USDC 的代币账户。

<img src="https://cdn.prod.website-files.com/641ba798c17bb180d832b666/65d59a8c5f6bee1516ab8758_tokenholders-diagram.jpg" loading="lazy" alt="">

## 账户结构图

现在我们了解了代币和代币账户的工作原理，我们可以看看如何获取特定代币的所有持有者。每个持有特定代币的钱包将有一个该代币的代币账户。这意味着该代币将与所有持有该代币的钱包的代币账户相关联。这就是我们将如何确定所有持有者的方法。如果我们能够找到一种方法来获取与一个代币相关联的所有代币账户，然后获取这些账户的所有者，我们就会有一个所有持有者的列表！

## getTokenAccounts 方法

幸运的是，Helius 的 [getTokenAccounts API 方法](https://docs.helius.dev/compression-and-das-api/digital-asset-standard-das-api/get-token-accounts)允许我们做到这一点。我们可以在 API 调用的参数中包括任何代币的铸造地址，然后我们会得到一个返回的列表，其中包括该代币的所有代币账户。除此之外，API 还返回每个代币账户的所有者；这个所有者就是我们通常所说的代币持有者。需要注意的一点是，一个账户可以有多个相同代币的代币账户。这不是一个大问题；我们只需要设置一些逻辑来处理共享同一所有者的代币账户。

## 实现

现在让我们深入到代码中，看看如何实际操作。你需要一个 Helius API 密钥来进行操作。你可以通过访问 <https://dev.helius.xyz> 并注册一个账户来免费获取一个。首先，我们必须创建一个名为 getTokenHolders.js 的 JavaScript 文件。我们可以通过添加 Helius URL 并导入 fs 库来保存结果到 JSON 文件来开始。

```javascript
const url = `https://mainnet.helius-rpc.com/?api-key=`;
const fs = require("fs");
```

接下来，我们将创建一个方法来获取与特定代币相关的所有代币账户。我们可以通过创建一个名为 findHolders 的方法开始，该方法将使用 getTokenAccounts 方法来获取所需的数据。你可以在[这里](https://docs.helius.dev/compression-and-das-api/digital-asset-standard-das-api/get-token-accounts)阅读有关 getTokenAccounts 方法的更多信息。需要注意的重要一点是，每次调用 API 最多只能返回 1000 个代币账户。Solana 上的大多数大型代币都有超过 100,000 个代币账户。为了解决这个问题，我们将使用分页（pagination）来遍历所有的代币账户，并继续进行 API 调用，直到我们获取到所有现有代币账户的数据。在方法中，我们将在 getTokenAccounts 调用的参数中包含代币铸造地址。当我们遍历所有的代币账户时，我们将每个唯一的代币账户所有者添加到一个列表中。一旦方法运行完成，我们将这个持有者列表保存到包含所有代币持有者的 JSON 文件中。

```javascript
const findHolders = async () => {
  // 分页逻辑
  let page = 1;
  // allOwners 将存储所有持有该代币的地址
  let allOwners = new Set();

  while (true) {
    const response = await fetch(url, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        jsonrpc: "2.0",
        method: "getTokenAccounts",
        id: "helius-test",
        params: {
          page: page,
          limit: 1000,
          displayOptions: {},
          // 代币的铸造地址
          mint: "CKfatsPMUf8SkiURsDXs7eK6GWb4Jsd6UDbs7twMCWxo",
        },
      }),
    });
    const data = await response.json();
    // 分页逻辑
    if (!data.result || data.result.token_accounts.length === 0) {
      console.log(`没有更多结果。总页数：${page - 1}`);
      break;
    }
    console.log(`正在处理第 ${page} 页的结果`);
    // 将唯一的所有者添加到代币所有者列表中
    data.result.token_accounts.forEach((account) =>
      allOwners.add(account.owner)
    );
    page++;
  }

  fs.writeFileSync(
    "output.json",
    JSON.stringify(Array.from(allOwners), null, 2)
  );
};
```

在上面的示例中，getTokenAccounts 方法在分页遍历所有代币账户的过程中被多次调用。API 响应将为每个代币账户提供以下数据：

```json
{
  "address": "CVMR1nbxTcQ7Jpa1p137t5TyKFii3Y7Vazt9fFct3tk9",
  "mint": "SHDWyBxihqiCj6YekG2GUr7wqKLeLAMK1gHZck9pL6y",
  "owner": "CckxW6C1CjsxYcXSiDbk7NYfPLhfqAm3kSB5LEZunnSE",
  "amount": 100000000,
  "delegated_amount": 0,
  "frozen": false
}
```

我们从这些代币账户中提取了所有者并将其添加到我们的持有者列表中。如果我们愿意，我们还可以存储每个代币账户持有的代币数量，以找出最大的持有者。

现在，只需调用该方法即可：

```javascript
findHolders();
```

我们在 getTokenHolders.js 文件中的完整代码应如下所示：

```javascript
const url = `https://mainnet.helius-rpc.com/?api-key=`;
const fs = require("fs");

const findHolders = async () => {
  let page = 1;
  let allOwners = new Set();

  while (true) {
    const response = await fetch(url, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        jsonrpc: "2.0",
        method: "getTokenAccounts",
        id: "helius-test",
        params: {
          page: page,
          limit: 1000,
          displayOptions: {},
          mint: "DezXAZ8z7PnrnRJjz3wXBoRgixCa6xjnB7YaB1pPB263",
        },
      }),
    });
    const data = await response.json();

    if (!data.result || data.result.token_accounts.length === 0) {
      console.log(`没有更多结果。总页数：${page - 1}`);
      break;
    }
    console.log(`正在处理第 ${page} 页的结果`);
    data.result.token_accounts.forEach((account) =>
      allOwners.add(account.owner)
    );
    page++;
  }

  fs.writeFileSync(
    "output.json",
    JSON.stringify(Array.from(allOwners), null, 2)
  );
};

findHolders();
```

输出
代码的输出将是一个包含所有代币持有者的列表，类似于如下内容：

```json
[
  "111An9SVxuPpgjnuXW9Ub7hcVmZpYNrYZF4edsGwJEW",
  "11Mmng3DoMsq2Roq8LBcqdz6d4kw9oSD8oka9Pwfbj",
  "112uNfcC8iwX9P2TkRdJKyPatg6a4GNcr9NC5mTc2z3",
  "113uswn5HNgEfBUKfK4gVBmd2GpZYbxd1N6h1uUWReg",
  "11CyvpdYTqFmCVWbJJeKFNX8F8RSjNSYW5VVUi8eX4P",
  "11MANeaiHEy9S9pRQNu3nqKa2gpajzX2wrRJqWrf8dQ",
  …
]
```

你可以通过我们的 [Replit 示例](https://replit.com/@owen47/getTokenAccountsExample)自己进行测试。

结论
在本指南中，我们成功地介绍了通过 Helius getTokenAccounts API 来识别 Solana 代币持有者的过程。本教程应为您提供与您的代币社区直接互动的必要技能，无论是用于空投、获取见解，还是其他互动。如果你有任何问题，请随时通过 [Twitter](https://x.com/heliuslabs) 或 [Discord](https://discord.com/invite/FSUZxChanb) 与我们联系。
