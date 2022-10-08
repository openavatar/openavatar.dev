# 简介

## 什么是 OpenAvatar-SDK ?

OpenAvatar 是全球第一个可视化 Web3 DID协议

OpenAvatar-SDK 是 OPENAVATAR 的一个开源组件，它是从Privateparty库派生的，是一个开源框架，可以非常容易地构建区块链认证的 Web3 应用程序。

只需几行代码，您就可以构建具有非常复杂的钱包身份验证/授权逻辑的 Web 应用程序。

![ppclient.png](ppclient.png)

The result:

![login.gif](login.gif)


1. **身份验证：** 使用区块链钱包签名登录
2. **授权：** 根据区块链状态或链下查询授权访问您的页面
3. **简单：** 只需一行 JavaScript 代码即可设置。 没有复杂的步骤。
4. **开源：** 它是一个 100% 开源框架，没有第三方API。

## 你能用 OpenAvatar-SDK 做什么?

OpenAvatar-SDK 只是用密码学和区块链状态取代了传统的身份验证方法。

这意味着您可以构建任何您可以想象的应用。 您只是用 OpenAvatar-SDK 替换传统的用户数据库。

1. **简单的身份验证：** 适合知道当前登录钱包账户的的Web3应用程序（允许任何人使用钱包登录）
2. **仅限邀请的应用程序：** 仅允许地址列表中的人登录
3. **基于 NFT 的授权：** 仅允许基于钱包拥有的 NFT 所有权登录
4. **基于 ERC20 的授权：** 仅允许基于钱包拥有的 ERC20 代币登录
5. **空投：** 可以通过仅授权曾经与某个合约交互过的地址来实施空投。
6. **More：** 基本上可以根据任何区块链查询实现授权。


## 社群支持

提问或反馈：:

1. Twitter: https://twitter.com/skogard
2. Discord: https://discord.gg/BZtp5F6QQM
3. GitHub: https://github.com/privatepart

---


# 快速入门

## 1. Server
服务端有两个步骤要做：

1. **设置后端：** `privateparty` 模块让您轻松设置受区块链钱包签名保护的 [express.js]服务器(https://expressjs.com/) 。
2. **连接到后端：** 设置后端后，您可以使用 `partyconnect` 库从浏览器连接到它。

首先安装依赖

```
npm install privateparty
```

接着创建一个名为 `index.js` 的文件并编写初始化逻辑：

```javascript
const Privateparty = require('privateparty')
const party = new Privateparty()

// create a "user" group
party.add("user")

// authenticate using the "user" group.
party.app.get("/", party.auth("user"), (req, res) => {

  // the 'req.session' will be an empty object before the user authenticates
  console.log("session", req.session)

  // serve the index.html file to the public
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```

## 2. Client

现在让我们创建一个与服务器端通信的文件 `index.html`：

```html
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<script src="https://unpkg.com/partyconnect/dist/partyconnect.js"></script>
<style>
.hidden { display: none; }
</style>
</head>
<body>
<nav>
  <button></button>
  <pre class='session'></pre>
</nav>
<script>
const party = new Privateparty()
const render = async () => {
  // Get the "user" session
  let session = await party.session("user")
  // if logged in (session.user exists), it's a logout button. if logged out, it's a login button.
  document.querySelector("button").innerHTML = (session ? "logout" : "login")
  // print the current session
  document.querySelector(".session").innerHTML = JSON.stringify(session, null, 2)
}
document.querySelector("button").addEventListener("click", async (e) => {
  try {
    // Get the "user" session
    let session = await party.session("user")
    if (session) {
      await party.disconnect("user")      // if logged in, log out
    } else {
      await party.connect("user")         // if logged out, log in
    }
    await render()
  } catch (e) {
    // display error if something went wrong
    document.querySelector(".session").innerHTML = e.message
  }
})
render()
</script>
</body>
</html>
```

## 3. 启动 App

运行命令:

```
node index
```

打开浏览器链接 http://localhost:3000 

就会看到下面的结果，：

![desktoplogin.gif](desktoplogin.gif)

## 4. 支持手机移动钱包

到目前为止，我们刚刚使用了桌面浏览器上安装的默认钱包（Metamask）。 现在让我们看看我们如何支持移动钱包。

为了支持移动钱包，我们将使用 [Walletconnect](https://walletconnect.com/)。 您无需学习如何使用 Walletconnect。 在创建 `Privateparty` 实例时，您只需要传递一个名为 `walletconnect` 的属性，如下所示：

```javascript
const party = new Privateparty({
  walletconnect: <Your Infura Key>
})
```

首先注册 [Infura](https://infura.io) ，创建一个项目并获取项目 ID。

然后让我们回到上面的前端例子，在初始化 `Privateparty` 实例时只传递 `walletconnect` 属性：

```html
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<script src="https://unpkg.com/partyconnect/dist/partyconnect.js"></script>
<style>
.hidden { display: none; }
</style>
</head>
<body>
<nav>
  <button></button>
  <pre class='session'></pre>
</nav>
<script>
const party = new Privateparty({
  walletconnect: <USE YOUR OWN INFURA KEY>
})
const render = async () => {
  // Get the "user" session
  let session = await party.session("user")
  // if logged in (session.user exists), it's a logout button. if logged out, it's a login button.
  document.querySelector("button").innerHTML = (session ? "logout" : "login")
  // print the current session
  document.querySelector(".session").innerHTML = JSON.stringify(session, null, 2)
}
document.querySelector("button").addEventListener("click", async (e) => {
  try {
    // Get the "user" session
    let session = await party.session("user")
    if (session) {
      await party.disconnect("user")      // if logged in, log out
    } else {
      await party.connect("user")         // if logged out, log in
    }
    await render()
  } catch (e) {
    // display error if something went wrong
    document.querySelector(".session").innerHTML = e.message
  }
})
render()
</script>
</body>
</html>
```
注意 Walletconnect 无法在 localhost 上运行，因此您需要获取 HTTPS URL 进行测试。 您可以通过以下方式做到这一点：

1. 将站点公开部署到 HTTPS 域（所有 Web 托管服务提供商默认支持此功能）
  
2. 使用 [ngrok](https://ngrok.com/) 或 [localtunnel](https://theboroer.github.io/localtunnel-www/) 等进行本地测试

让我们尝试第二种方法并使用 localtunnel 在本地进行测试。 需要遵循以下步骤：

1.启动privateparty服务器：`node index`

2.启动一个指向privateparty服务器的localtunnel：`npx lt --port 3000`

这将为您提供一个可以测试的公共 HTTPS 网址。 将其复制并粘贴到浏览器中。 您将看到类似于以下动画的内容（这里演示了使用两个不同的移动钱包 Metamask mobile 和 Rainbow 钱包登录）：

![mobilelogin.gif](mobilelogin.gif)

## 5. 更简单的身份验证

上面示例演示了如何使用 JavaScript 以编程方式登录或注销。

还有一种更简单的方法来实现登录：只需包含一个链接（👇🏻下面示意）就可以工作：

```
<a href="/privateparty/gate/user">Login</a>
```

它将用户发送到内置的 **"gate"** 页面，该页面会自动处理登录和注销，并在处理完登录/注销后将用户返回。

要了解如何执行此操作，请查看“[One-liner login/logout](#one-liner-loginlogout)”部分。

![gate.gif](gate.gif)

---

# 示例

## 仅邀请应用

仅允许指定地址登录。

### 服务器

先安装依赖

```
npm install privateparty
```

接着创建一个名为 `index.js` 的文件并编写初始化逻辑：

```javascript
const Privateparty = require('privateparty')
const party = new Privateparty()
const MEMBERS = [
  "0xfb7b2717f7a2a30b42e21cef03dd0fc76ef761e9",
  "0x502b2fe7cc3488fcff2e16158615af87b4ab5c41"
]
party.add("user", {
  authorize: async (req, account) => {
    if (MEMBERS.includes(account)) {
      // if the account is part of the MEMBERS array, the account is authorized.
      // when authorized, the session will look like:
      //
      //  {
      //    account: <account>,
      //    auth: {
      //      member: true
      //    }
      //  }
      return { member: true }
    } else {
      throw new Error("not on the list!") 
    }
  }
})
party.app.get("/", party.auth("user"), (req, res) => {
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```

### 客户端

现在让我们创建一个与服务器通信的文件：`index.html`

```html
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<script src="https://unpkg.com/partyconnect/dist/partyconnect.js"></script>
<style>
.hidden { display: none; }
</style>
</head>
<body>
<nav>
  <button></button>
  <pre class='session'></pre>
</nav>
<script>
const party = new Privateparty()
const render = async () => {
  let session = await party.session("user")
  // if logged in (session.user exists), it's a logout button. if logged out, it's a login button.
  document.querySelector("button").innerHTML = (session ? "logout" : "login")
  // print the current session
  document.querySelector(".session").innerHTML = JSON.stringify(session, null, 2)
}
document.querySelector("button").addEventListener("click", async (e) => {
  try {
    let session = await party.session("user")
    if (session) {
      await party.disconnect("user")      // if logged in, log out
    } else {
      await party.connect("user")         // if logged out, log in
    }
    await render()
  } catch (e) {
    document.querySelector(".session").innerHTML = e.message
  }
})
render()
</script>
</body>
</html>
```


## ERC20 门控应用

有时，您可能希望用户使用钱包登录，并存储指定 ERC20 代币余额。

这对于搭建空投网站，或许多其他目的的应用时，可能很有用。

### 客户

让我们首先搭建前端。 先创建一个与服务器通信的文件： `index.html` 

```html
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<script src="https://unpkg.com/partyconnect/dist/partyconnect.js"></script>
<style>
.hidden { display: none; }
</style>
</head>
<body>
<nav>
  <button></button>
  <pre class='session'></pre>
</nav>
<script>
const party = new Privateparty()
const render = async () => {
  let session = await party.session("user")
  // if logged in (session.user exists), it's a logout button. if logged out, it's a login button.
  document.querySelector("button").innerHTML = (session ? "logout" : "login")
  // print the current session
  document.querySelector(".session").innerHTML = JSON.stringify(session, null, 2)
}
document.querySelector("button").addEventListener("click", async (e) => {
  try {
    let session = await party.session("user")
    if (session) {
      await party.disconnect("user")      // if logged in, log out
    } else {
      await party.connect("user")         // if logged out, log in
    }
    await render()
  } catch (e) {
    document.querySelector(".session").innerHTML = e.message
  }
})
render()
</script>
</body>
</html>
```


### Server

对于这个例子，我们将：

1. 允许任何人登录。
2. 在区块链中查询他们的 UNISWAP 代币 ($UNI) 余额，并通过 cookie 将其保存用于后继访问。

由于我们将查询区块链，因此我们需要使用 JSON-RPC 端点。


#### 1. 复杂的方式

首先安装依赖

```
npm install privateparty @alch/alchemy-web3
```

现在创建一个名为 `index.js` 的文件并编写初始化逻辑：

```javascript
///////////////////////////////////////////////////////////////////////////////////////////
//
//  When a user logs in, take a snapshot of an ERC20 token balance and store it in cookie
//
///////////////////////////////////////////////////////////////////////////////////////////
const Privateparty = require('privateparty')
const { createAlchemyWeb3 } = require("@alch/alchemy-web3");
const web3 = createAlchemyWeb3(<YOUR JSON-RPC ENDPOINT URL>)
const party = new Privateparty()
party.add("user", {
  authorize: async (req, account) => {
    const UNI = "0x1f9840a85d5af5bf1d1762f925bdaddc4201f984"
    let contract = new web3.eth.Contract([{
      "constant": true,
      "inputs": [ { "internalType": "address", "name": "owner", "type": "address" } ],
      "name": "balanceOf",
      "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ],
      "payable": false,
      "stateMutability": "view",
      "type": "function"
    }], UNI)
    let balance = await contract.methods.balanceOf(account).call()

    // Returning the additional attribute 'balance' will automatically set the value on the cookie
    // under the "auth" attribute
    return {
      balance: balance
    }
  }
})
party.app.get("/", party.auth("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```


由于 `authorize()` 函数不会抛出任何错误，而只会返回余额对象，因此此应用程序将允许任何人登录，但使用余额信息来识别用户（例如，持有人将拥有非零余额，而 非持有者的余额为0）

#### 2. 简单的方式

由于 ERC20 和 ERC721 是常用的标准，Privateparty 提供了一个内置的 ABI，您可以分别在 `party.abi.erc20` 和 `party.abi.erc721` 下访问。

上面完全相同的代码可以重写如下：

```javascript
///////////////////////////////////////////////////////////////////////////////////////////
//
//  当用户登录时，获取ERC20代币余额，并将其存储在 cookie 中
//
///////////////////////////////////////////////////////////////////////////////////////////
const Privateparty = require('privateparty')
const { createAlchemyWeb3 } = require("@alch/alchemy-web3");
const web3 = createAlchemyWeb3(<YOUR JSON-RPC ENDPOINT URL>)
const party = new Privateparty()
party.add("user", {
  authorize: async (req, account) => {
    const UNI = "0x1f9840a85d5af5bf1d1762f925bdaddc4201f984"
    let balance = await party.contract(web3, party.abi.erc20, UNI).balanceOf(account).call()
    return {
      balance: balance
    }
  }
})
party.app.get("/", party.auth("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```


## NFT 控制应用

使用相同的原理，我们可以实现通过 NFT 授权的登录。 例如，通常您可能希望只允许人们在拥有至少 1 个（或更多）集合中的 NFT 时登录。

您可以使用此功能来实现通过 NFT 认证社区和网站。

### Server

```javascript
///////////////////////////////////////////////////////////////////////////////////////////
//
//  When a user logs in, take a snapshot of an ERC20 token balance and store it in cookie
//
///////////////////////////////////////////////////////////////////////////////////////////
const Privateparty = require('privateparty')
const party = new Privateparty()
party.add("mfer", {
  contracts: {
    sartoshi: {
      address: "0xf7d134224a66c6a4ddeb7dee714a280b99044805",
      rpc: "https://eth-mainnet.alchemyapi.io/v2/NgVL3BEuBntBU4cbzjh3FxBIDO8dZM4y",
      abi: party.abi.erc721
    }
  },
  authorize: async (req, account, contracts) => {
    let balance = await contracts.sartoshi.methods.balanceOf(account).call()
    if (balance > 0) return { balance: balance }
    else throw new Error("must own at least one 'end of sartoshi'")
  }
})
party.app.get("/", party.protect("mfer"), (req, res) => {
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```

### Client

```html
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<script src="https://unpkg.com/partyconnect/dist/partyconnect.js"></script>
<style>
.hidden { display: none; }
</style>
</head>
<body>
<h1>Some exclusive content!</h1>
<div>This page is protected by Privateparty "mfers" role</div>
</body>
</html>
```

## 用 NFT 登录

![loginwithnft.gif](loginwithnft.gif)

有时你可能想直接“使用 NFT 登录”，并直接在 cookie 中设置 NFT 图像的URL。

这样，一旦用户登录应用程序，应用程序就可以在整个应用程序会话中使用该NFT的相关数据（例如编号、图像...）。

但是，Privateparty 服务器如何知道您想使用哪个确切的 NFT 登录？

需要向服务器发送额外的数据，可以在调用 `connect()` 方法时简单地传递其他属性：

### Client

让我们首先创建一个与服务器通信的文件： `index.html` 

```html
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<script src="https://unpkg.com/partyconnect/dist/partyconnect.js"></script>
<style>
.hidden { display: none; }
img { width: 50px; height: 50px; flex-shrink: 0; margin-right: 10px; border-radius: 50px; }
nav { display: flex; align-items: center; }
</style>
</head>
<body>
<nav>
  <img class='hidden'>
  <input type='text' placeholder='collection address' id='collection'>
  <input type='text' placeholder='tokenId' id='id'>
  <button></button>
</nav>
<pre class='session'></pre>
<script>
const party = new Privateparty()
const render = async () => {
  let session = await party.session("user")
  console.log("session", session)
  // if logged in (session exists), it's a logout button. if logged out, it's a login button.
  document.querySelector("button").innerHTML = (session ? "logout" : "login")
  // print the current session
  document.querySelector(".session").innerHTML = JSON.stringify(session, null, 2)
  if (session && session.auth && session.auth.image) {
    document.querySelector("img").src = session.auth.image 
    document.querySelector("img").classList.remove("hidden")
  } else {
    document.querySelector("img").classList.add("hidden")
  }
}
document.querySelector("button").addEventListener("click", async (e) => {
  let session = await party.session("user")
  if (session) {
    await party.disconnect("user")      // if logged in, log out
  } else {
    // Pass additional payload
    // These will be accessible as req.body.payload
    let connection = await party.connect("user", {
      collection: document.querySelector("#collection").value,
      tokenId: document.querySelector("#id").value
    })
    console.log("connection", connection)
  }
  await render()
})
render()
</script>
</body>
</html>
```

请注意第一行 `await party.connect("user")` , 它传递了一个具有 `collection` 和 `tokenId` 属性的对象：

```javascript
let connection = await party.connect("user", {
  collection: document.querySelector("#collection").value,
  tokenId: document.querySelector("#id").value
})
```

例如，输入值可能如下所示：

```json
{
  collection: "0x6866ed9a183f491024692971a8f78b048fb6b89b",
  tokenId: "55005454344647406361450320675654878134478584534017520891306338141495783002503"
}
```

该对象将作为 `req.body.payload` 传递给 OpenAvatar-SDK 服务器 `authorize()` 程序，下面对此进行解释：

### Server

首先安装依赖库:

```
npm install privateparty @alch/alchemy-web3
```

然后，创建一个名为 `index.js` 的文件并编写初始化逻辑：

```javascript
///////////////////////////////////////////////////////////////////////////////////////////
//
//  When a user logs in, take a snapshot of an ERC20 token balance and store it in cookie
//
///////////////////////////////////////////////////////////////////////////////////////////
const fetch = require('cross-fetch')
const Privateparty = require('privateparty')
const { createAlchemyWeb3 } = require("@alch/alchemy-web3");
const web3 = createAlchemyWeb3(<YOUR JSON-RPC ENDPOINT URL>)
const party = new Privateparty()
party.add("user", {
  authorize: async (req, account) => {
    //  req.body.payload := {
    //    collection: "0x6866ed9a183f491024692971a8f78b048fb6b89b",
    //    tokenId: "55005454344647406361450320675654878134478584534017520891306338141495783002503"
    //  }
    console.log("req.body", req.body)
    console.log("account", account)

    // Query the blockchain to get the ERC721 tokenURI
    let tokenURI = await party.contract(web3, party.abi.erc721, req.body.payload.collection).tokenURI(req.body.payload.tokenId).call()
    // Get the image URL and turn it into IPFS gateway URL.
    let image = await fetch("https://ipfs.io/ipfs/" + tokenURI.replace("ipfs://", "")).then(r => r.json()).then(r => r.image)
    return {
      tokenURI,
      image: "https://ipfs.io/ipfs/" + image.replace("ipfs://", "")
    }
  }
})
party.app.get("/", party.auth("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```

现在运行命令 `node index` 并打开浏览器链接 http://localhost:3000

您将看到一个登录界面，可以在其中输入 NFT 合约地址和 tokenId。

只有当你的钱包拥有 NFT 时，才能登录。


## 多角色

Sometimes you may want to support multiple roles for a single account.

For example, Alice may be a "user" in a web app, but she may also be the "admin" who can have an admin interface. Only those with an "admin" role can access the admin interface, while the rest of the users can only have the "user" role and access the user interface.

Let's try building a minimal app that does that. We will build:

1. Privateparty server
2. User interface
3. Admin inteface

有时你可能希望为单个帐户支持多角色。

例如，Alice 可能是 Web3 应用中的“user”角色，但她也可以拥有管理界面的“admin”角色。 只有拥有“admin”管理员角色的用户才能访问管理界面，而其余拥有“user”角色的用户只能并访问普通用户界面。

让我们搭建一个最小应用示例。 我们将建立：

1. OpenAvatar-SDK 服务器
2. 用户页面
3. 管理员页面

### Server

```javascript
const Privateparty = require('privateparty')
const party = new Privateparty()

// Add a "user" role => will automatically create the default the following endpoints:
//
//  session: "/privateparty/admin/session",
//  connect: "/privateparty/admin/connect",
//  disconnect: "/privateparty/admin/disconnect",
//
party.add("user")

// Add an "admin" role with custom endpoints:
party.add("admin", {
  session: "/privateparty/admin/session",
  connect: "/privateparty/admin/connect",
  disconnect: "/privateparty/admin/disconnect",
  authorize: (req, account) => {
    // Currently anyone can login as admin, but you can add a logic to only allow certain addresses to login
    return { admin: true }
  }
})

// "user" interface => will display the index.html file
party.app.get("/", party.auth("user"), (req, res) => {
  res.sendFile(process.cwd() + "/index.html")
})

// "admin" interface => will display the admin.html file
party.app.get("/admin", party.auth("admin"), (req, res) => {
  res.sendFile(process.cwd() + "/admin.html")
})
party.app.listen(3000)
```

### 用户页面

通过 http://localhost:3000 访问用户界面（路由“/”），任何帐户都可以登录。

```html
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<script src="https://unpkg.com/partyconnect/dist/partyconnect.js"></script>
<style>
.hidden { display: none; }
</style>
</head>
<body>
<nav>
  <h1>User page</h1>
  <button></button>
  <pre class='session'></pre>
  <a href="/admin">go to admin dashboard</a>
</nav>
<script>
const party = new Privateparty()
const render = async () => {
  let session = await party.session("user")
  // if logged in (session.user exists), it's a logout button. if logged out, it's a login button.
  document.querySelector("button").innerHTML = (session ? "logout" : "login")
  // print the current session
  document.querySelector(".session").innerHTML = JSON.stringify(session, null, 2)
}
document.querySelector("button").addEventListener("click", async (e) => {
  let session = await party.session("user")
  try {
    if (session) {
      await party.disconnect("user")      // if logged in, log out
    } else {
      await party.connect("user")         // if logged out, log in
    }
    await render()
  } catch (e) {
    document.querySelector(".session").innerHTML = e.message
  }
})
render()
</script>
</body>
</html>
```

### 管理员页面

通过 http://localhost:3000/admin 访问管理界面（路由“/admin”）。 您可以用“管理员”身份登录。 上面的OpenAvatar-SDK 服务器代码允许任何人以管理员身份登录，但您可以更新 `authorize()` 部分仅授权白名单（名单中的地址可以登录）以管理员身份登录。

```html
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<script src="https://unpkg.com/partyconnect/dist/partyconnect.js"></script>
<style>
.hidden { display: none; }
</style>
</head>
<body>
<nav>
  <h1>Admin page</h1>
  <button></button>
  <pre class='session'></pre>
  <a href="/">go to user dashboard</a>
</nav>
<script>
const party = new Privateparty()
const render = async () => {
  let session = await party.session("admin")
  // if logged in (session.user exists), it's a logout button. if logged out, it's a login button.
  document.querySelector("button").innerHTML = (session ? "logout" : "login")
  // print the current session
  document.querySelector(".session").innerHTML = JSON.stringify(session, null, 2)
}
document.querySelector("button").addEventListener("click", async (e) => {
  let session = await party.session("admin")
  try {
    if (session) {
      await party.disconnect("admin")      // if logged in, log out
    } else {
      await party.connect("admin")         // if logged out, log in
    }
    await render()
  } catch (e) {
    document.querySelector(".session").innerHTML = e.message
  }
})
render()
</script>
</body>
</html>
```

## Cross origin login

Sometimes your frontend code may be hosted on a different domain than the backend.

In this case you can use the built-in CORS support to allow ONLY the domain you specify to authenticate using your privateparty server.

Let's set up:

1. A privateparty server at port 3007
2. A frontend website running at port 8080

The frontend website at 8080 will try to authenticate against the privateparty server at http://localhost:3007

### Server

Save the following code as `index.js`:

```javascript
const Privateparty = require('privateparty')
const party = new Privateparty({
  cors: {
    credentials: true,
    origin: ["http://localhost:8080"] // Allow port 8080 to access the server cross origin
  }
})
party.add("user")
party.app.listen(3000)
```

### Client

Save the following code as `index.html`:


```html
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<script src="https://unpkg.com/partyconnect/dist/partyconnect.js"></script>
<style>
.hidden { display: none; }
</style>
</head>
<body>
<nav>
  <button></button>
  <pre class='session'></pre>
</nav>
<script>
const party = new Privateparty({
  host: "http://localhost:3000"
})
const render = async () => {
  let session = await party.session("user")
  // if logged in (session.user exists), it's a logout button. if logged out, it's a login button.
  document.querySelector("button").innerHTML = (session ? "logout" : "login")
  // print the current session
  document.querySelector(".session").innerHTML = JSON.stringify(session, null, 2)
}
document.querySelector("button").addEventListener("click", async (e) => {
  let session = await party.session("user")
  try {
    if (session) {
      await party.disconnect("user")      // if logged in, log out
    } else {
      await party.connect("user")         // if logged out, log in
    }
    await render()
  } catch (e) {
    document.querySelector(".session").innerHTML = e.message
  }
})
render()
</script>
</body>
</html>
```

The only differnt part here is the initialization step:

```javascript
const party = new Privateparty({
  host: "http://localhost:3000"
})
```

By default, the partyconnect.js client makes requests to the same domain. But you can customize the endpoint by setting the `host` attribute when initializing a Privateparty client.

### Run

First start the privateparty server:

```
node index
```

Now let's launch the `index.html` at port 8080 using:

```
npx http-server
```

Now open the browser at http://localhost:8080 and it should work as intended.

## Cross platform login

Privateparty is powered by [JWT](https://en.wikipedia.org/wiki/JSON_Web_Token).

This means you can use the same generated token **both inside and outside the browser**. "outside the browser" can be anywhere, including:

- mobile app
- server
- serverless function (AWS lambda, vercel, netlify, etc.)
- IoT device
- and so on..

Let's check out how to authenticate and authorize using a node.js app.

### Server

For the sake of simplicity, we will just create a privateparty server that lets anyone log in (therefore there is no `authorize()` function:

```javascript
///////////////////////////////////////////////////////////////////////////////////////////
//
//  When a user logs in, take a snapshot of an ERC20 token balance and store it in cookie
//
///////////////////////////////////////////////////////////////////////////////////////////
const Privateparty = require('privateparty')
const party = new Privateparty()
party.add("user")
party.app.get("/api", party.protect("user", { json: { error: "not logged in" } }), (req, res) => {
  res.json({ status: "Logged in!" })
})
party.app.listen(3000)
```

If you want to t


### Client

For this we will use a node.js client named `partypass`, and `cross-fetch` (to make fetch requests).

```
npm install partypass
```

Now create a file named `client.js`:

```javascript
const Partypass = require('partypass')
const fetch = require('cross-fetch')
const run = async () => {
  const pass = new Partypass({
    host: "http://localhost:3000",
    key: "cd76c32ffeae94b725b40b1f58ffc793d5b0e96596f8d067f29d385894f16424" // replace with your private key
  });
  const session = await pass.create("user"); // The session object will contain { jwt, account, ... }

  // Now let's make a request with the jwt
  let response = await fetch("http://localhost:3000/api", {
    headers: {
      authorization: `token ${session.jwt}`
    }
  }).then((r) => {
    return r.json()
  })
  console.log(response)
}
run()   // run it!
```

Run it with `node client`.

1. The `client.js` will make a request to the privateparty server at port 3000 to get a session
2. Then it will use the `session.jwt` in the authorization header to make an authenticated request to the `/api` endpoint, which will succeed.

It will print:

```
{ status: 'Logged in!' }
```

Now, just to make sure that it fails when there's no token, let's try making the same request, but without the authorization header:

```javascript
const Partypass = require('partypass')
const fetch = require('cross-fetch')
const run = async () => {
  let response = await fetch("http://localhost:3000/api").then((r) => {
    return r.json()
  })
  console.log(response)
}
run()   // run it!
```

It will print:

```
{ error: 'not logged in' }
```

## One-liner login/logout

We've looked at ways to authenticate using JavaScript. But there's an even simpler way to get started. All you need to do is send users to a "gate" page.

### Server

Let's first set up a simple Privateparty server by writing a file named `app.js`.

```javascript
const party = new Privateparty()
party.add("user")
party.app.get("/", (req, res) => {
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```

When a user goes to the `/` route, it will display the `index.html` page regardless of whether logged in or not. Let's take a look at the `index.html` file:

### Client

The following code does not use any Javascript code to authenticate. Instead, just by adding one line `<a id='account' href='/privateparty/gate/user?callback=/'></a>`, the user can click to open a "gate" page to login or logout, and come back (kind of like opening a facebook login page for "facebook connect" and redirecting back after logging in):

```html
<html>
<head>
<script src="https://unpkg.com/partyconnect@0.0.42/dist/partyconnect.js"></script>
</head>
<body>
<nav>
  <a id='account' href="/privateparty/gate/user?callback=/"></a>
  <pre class='session'></pre>
</nav>
<script>
const party = new Privateparty()
const render = async () => {
  let session = await party.session("user")

  // if logged in, display the account. Otherwise display "login"
  document.querySelector("a").innerHTML = (session ? session.account : "login")

  // print the current session
  document.querySelector(".session").innerHTML = JSON.stringify(session, null, 2)
}
render()
</script>
</body>
</html>
```

Run the server with `node app.js` and go to http://localhost:3000 - you will see:

![gate.gif](gate.gif)

> Pay attention to how the browser URL changes.
> 
> - When the user clicks "login" it goes to /privateparty/gate/user?callback=/
> - Then after logging in, it automatically redirects back to "/" because of the callback
> - Same process for the logout. After logging out, it automatically redirects back to "/"

Let's go through the HTML to see what's going on.

1. First of all, the `render()` method checks the `user` session and displays the account if logged in, otherwise "login". This part we are familiar with.
2. Next, notice the `<a>` tag. It has an `href` attribute of `/privateparty/gate/user?callback=/`. Let's break this down.
    - The `/privateparty/gate/user` is the default route for the built-in "gate" page, which lets the user login (if logged out) or log out (if logged in).
      - This route is automatically generated from the role name. For example, to send a user to an `admin` role login page, the link would be `/privateparty/gate/admin`.
    - The `?callback=/` part tells the gate page which URL to redirect the user to after logging in (or logging out).
      - In this case, we want the login page to automatically send the user back to the `/` route, so the `callback` is `/`.

### Mobile support

To automatically support mobile wallets for the built-in gate page, you need to initialize the Privateparty with a custom `gate` config with the `walletconnect` attribute (the Infura ID):


```javascript
const party = new Privateparty({
  gate: {
    walletconnect: "667750972a89441ea5d276ed16d7eef0"
  }
})
party.add("user")
party.app.get("/", (req, res) => {
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```


## More examples

Check out the [demo folder](https://github.com/privatepart/privateparty/tree/main/demo) on GitHub for more examples.

## React component

Using react? Try the react component: [Partybutton](https://partybutton.papercorp.org/)

![partybutton.png](partybutton.png)

---

# Install

You can implement a Privateparty web app with 2 libraries (server-side and client-side) that talk to each other:

1. `privateparty`: The server-side module
2. `partyconnect`: The browser client for privateparty => automatically uses the browser wallets and sets the cookies after logging in.
3. `partypass`: The node.js client for privateparty => stateless client for making a request to a privateparty server and getting back a new session in a JWT.

## Server

To install:

```
npm install privateparty
```

Then, use the module in your app like this:

```javascript
const Privateparty = require('privateparty')
const party = new Privateparty()
```

## Browser client

### Load from CDN

Include in your frontend web app:

```html
<script src="https://unpkg.com/partyconnect/dist/partyconnect.js"></script>
```

### Import

```
npm install partyconnect
```

Then initialize with:

```javascript
// CJS
const Privateparty = require('partyconnect')
const party = new Privateparty(config)
```

or

```javascript
// ESM
import Privateparty from 'partyconnect'
const party = new Privateparty(config)
```

> If the code works fine but you're having trouble when you're trying to package for production with webpack, it's probably because webpack 5 has started excluding node.js core modules (for any library that uses node.js core modules). See this article to learn how to resolve this issue: https://www.alchemy.com/blog/how-to-polyfill-node-core-modules-in-webpack-5

## Node.js client

### Import

```
npm install partypass
```

Then initialize with:

```javascript
// CJS
const Partypass = require('partypass')
const pass = new Partypass(config)
```

or

```javascript
// ESM
import Partypass from 'partypass'
const pass = new Partypass(config)
```

### Load from CDN

Although partypass is a node.js client, you can use it in the web in certain cases.

For example, a server may use `pass.build()` to create a pass creation request, and send it to the user's browser, where the user makes the request using `pass.request()`. In this case you can also use the CDN JS:

Include in your frontend web app:

```html
<script src="https://unpkg.com/partypass/dist/partypass.js"></script>
```

---

# API

## Server

For the backend, you need to use the package `privateparty`. Simply instantiate a new `Privateparty` instance and it should give you everything you need to build a wallet protected web app backend.

### constructor

#### syntax

```javascript
const party = new Privateparty(config)
```

#### parameters

- `config`: privateparty server configuration
  - `secret`: **(optional)** a string used for signing cookies
    - See https://github.com/expressjs/cookie-parser#cookieparsersecret-options
    - If not specified, it will autogenerate a secret everytime the server restarts using [uuid](https://github.com/uuidjs/uuid).
  - `cors`: **(optional)** If you want to support CORS (cross origin requests) pass this attribute.
    - See https://github.com/expressjs/cors#configuration-options
  - `app`: **(optional)** Inject an existing instantiated express.js app instance
  - `express`: **(optional)** Inject an existin express module
  - `gate`: **(optional)** The built-in gate page config
    - `walletconnect`: The Walletconnect Infura ID, to support mobile wallets.
    - `fresh`: when using the default login page, whether the login should ask the user to (re-)connect a wallet from the wallet list, or to use the previously connected wallet if still connected
      - if `true`, the login attempt always displays all the wallets from the list and lets the user select one
      - if `false`, tries to immediately use a previously selected wallet to skip the wallet selection step (This is the default)

#### return value

- `party`: The initialized privateparty instance, which contains the following attributes:
  - `app`: an "app" instance created internally by calling `const app = express()`
  - `express`: the express module
  - `auth`: authentication & authorization function
  - `protect`: authentication & authorization function + error handling
  - `add`: a function to add authorization groups

> the `auth` method only tells you if the authorization results in a legitimate session or not, whereas the `protect` method is used to do what `auth` does but also automatically redirect to a logged out page or display a logged out page.

#### examples

##### 1. minimal server

```javascript
const party = new Privateparty()
```

##### 2. server with a fixed signing secret

```javascript
const party = new Privateparty({
  secret: "top secret"
})
```

##### 3. cross origin login support

In the following example, we have a privateparty server running at port 3001, and it allows requests from not just the port 3001 but also 3000, since we specified the origin http://localhost:3000

```javascript
const party = new Privateparty({
  cors: {
    credentials: true,
    origin: ["http://localhost:3000"]
  }
})
party.listen(3001)
```

##### 4. cross origin login with dynamic origin parsing

Using the dynamic origin configuration option from the CORS module (https://github.com/expressjs/cors#configuring-cors-w-dynamic-origin), you can dynamically parse request origins and authorize:

```javascript
const party = new Privateparty({
  cors: {
    credentials: true,
    origin: (origin, callback) => {
      // allow ALL localhost connections
      if (/localhost:[0-9]+/i.test(origin)) {
        callback(null, true)
      } else {
        callback(new Error())
      }
    }
  }
})
party.listen(3001)
```

##### 5. integrate with an existing express.js app

```javascript
const express = require('express')
const app = express()
const port = 3000

// Inject express app to Privateparty!
const party = new Privateparty({
  app: app,
})
// Define the authorization logic
party.add("user", {
  authorize: (req, account) => {
    // only allow 0xf7d134224a66c6a4ddeb7dee714a280b99044805 to log in
    if (account === "0xf7d134224a66c6a4ddeb7dee714a280b99044805") {
      return { authorized: true }
    } else {
      throw new Error("not allowed")
    }
  }
})
// Protect the app with the authorization role!
app.get('/', party.protect("user"), (req, res) => {
  res.send('Hello World!')
})
app.listen(port, () => {
  console.log(`Example app listening on port ${port}`)
})
```

### add()

Add a group to the party

#### syntax

```javascript
await party.add(name, config)
```

#### parameters

- `name`: group name (must be unique per group)
- `config`: configuration options for each group
  - `session`: (optional) The GET path to query the current session for this engine. (default: `/privateparty/session/${name}`)
  - `connect`: (optional) The POST path to create a session for this engine (default: `/privateparty/connect/${name}`)
  - `disconnect`: (optional) The POST path to destroy a session for this engine (default: `/privateparty/disconnect/${name}`)
  - `gate`: (optional) The gate page route for this engine (The "login/logout" page) (default: `privateparty/gate/${name}`)
  - `authorize`: a function that takes two or more arguments `req` (The incoming request object passed from express), `account` (The authenticated wallet address), and optionally `contracts` (only when you specify another attribute `contract`, explained below). 
    - To disallow a session based on the request, simply throw an error in the function.
    - To authorize the session, don't throw a function. Additionally, the return value of this function will be automatically set as the `auth` attribute of the session
  - `expire`: (optional) The session duration (how many seconds until a session expires). The default is `1000 * 60 * 60 * 24 * 30` (30 days).
  - `tokens`: (optional) an array of access tokens to allow
  - `contracts`: (optional) a declarative object for defining one or more contracts, which will be initialized and injected in to the `authoirze()` handler


#### return value

none

#### example

```javascript
const Privateparty = require('privateparty')
const party = new Privateparty()
party.add("user", {

  session: "/privateparty/session/user",            // custom path for the session route
  connect: "/privateparty/connect/user",            // custom path for the connect route 
  disconnect: "/privateparty/disconnect/user",      // custom path for the disconnect route
  gate: "/privateparty/gate/user",                  // custom path for the gate page route

  // Define as many contracts as you want, using <name>: <description object>
  contracts: {
    sartoshi: {
      address: "0xf7d134224a66c6a4ddeb7dee714a280b99044805",
      rpc: process.env.RPC,
      abi: party.abi.erc721
    }
  },

  authorize: async (req, account, contracts) => {

    // The "request" is the full HTTP request object (express.js)
    // The "account" is the account derived from the incoming signature
    // The "contracts" is an object made up of instantiated Web3.js contract objects, determined by the "contracts" attribute above

    let balance = await contracts.sartoshi.methods.balanceOf(account).call()
    console.log("balance", balance)
    if (balance > 0) {
      return { balance: balance }
    } else {
      throw new Error("must own at least one 'end of sartoshi'")
    }
  },

  expire: 1000 * 60 * 60 * 24,   // expire after 1 day

  // access tokens for API access
  tokens: [
    "01127c36-32fa-4c85-b6da-f720796fe679",
    "35161a5c-60f0-4809-8b49-1a662247f5b3"
  ]
  

})
party.app.get("/", party.auth("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```

### auth()

The authorization middleware you can add to any route.

To add authorization logic to any route, you need to:

1. First define an authrorization group and its behavior through the `add()` method
2. Then make use of the group by calling `party.auth(name)`

#### syntax

```javascript
party.app.get(route1, party.auth(name), (req, res) => {
  ...
})
party.app.post(route2, party.auth(name), (req, res) => {
  ...
})
```

#### parameters

- `name`: The authorization group name to use for the route handler


#### examples

##### 1. Default authentication

The following example simply authenticates a user's account based on the wallet signature.

```javascript
const party = new Privateparty()
party.add("user")
party.app.get("/", party.auth("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
```

1. The `party.add("user"` line will create a group named "user", which automatically creates the following routes:
    - `POST /privateparty/connect`
    - `POST /privateparty/disconnect`
    - `GET /privateparty/session`
2. Then the express app instance (`party.app`) handles the `GET /` request. But before that, it goes through the `party.auth("user")` middleware.
3. Since the `party.add("user")` did not specify any authorization logic, it will just allow all requests.
4. Therefore, when a user first visits the `/` route, the `req.session` will be null but...
3. After authenticating from the frontend, the `req.session` will contain `{ "user": { "account": <user address> } }`

##### 2. Authorization

By default, Privateparty logs everyone in. But often you will want to only allow certain people in.

You can achieve this with an `authorize(req, account)` function:

```javascript
const Privateparty = require('privateparty')
const allowed = [
  "0xab3b229eb4bcff881275e7ea2f0fd24eeac8c83a",
  "0x1ad91ee08f21be3de0ba2ba6918e714da6b45836",
  "0x829bd824b016326a401d083b33d092293333a830"
]
const party = new Privateparty()
party.add("user", {
  session: "/privateparty/session",
  connect: "/privateparty/connect",
  disconnect: "/privateparty/disconnect",
  authorize: async (req, account) => {
    if (!allowed.includes(account) {
      throw new Error("not allowed")
    }
  }
})
party.app.get("/", party.auth("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```

Note that we are:

1. adding a group named `"user"`
2. and then using the group in the `party.app.get("/", party.auth("user", (req, res) => { . . . })` handler.


##### 3. Multiple auth engines

Sometimes you may want to serve different content based on different roles. You can create roles with engines.

In the following code, we are using 2 engines:

1. user: normal user login flow. sign everyone in
2. admin: admin user login flow. check if the account is included in the ADMIN array, and if not, throw an error

```javascript
const Privateparty = require('privateparty')
const ADMINS = ["0x502b2fe7cc3488fcff2e16158615af87b4ab5c41"]
const party = new Privateparty()
party.add("user", {
  session: "/privateparty/session",
  connect: "/privateparty/connect",
  disconnect: "/privateparty/disconnect",
})
party.add("admin", {
  session: "/privateparty/admin/session",
  connect: "/privateparty/admin/connect",
  disconnect: "/privateparty/admin/disconnect",
  authorize: async (req, account) => {
    if (ADMINS.includes(account)) {
      return { admin: true }
    } else {
      throw new Error("not an admin")
    } 
  }
})
party.app.get("/", party.auth("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
party.app.get("/admin", party.auth("admin"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```

Note that we have two `GET` route handlers here:

1. `GET /`: The normal route for normal users. Because we're using `auth("user")`, it will use the `user` group.
2. `GET /admin`: The page where the admins can login. Because we're using `auth("admin")`, it will use the `admin` group.


### protect()

Like `auth()`, but automatically redirects to the built-in login page if not authorized.

> The `auth()` method returns a `null` value for `req.session` when not authorized, and that's all. The `protect()` method actually redirects to the login page.

To add the protection logic to any route, you need to:

1. First define an authrorization group and its behavior through the `add()` method
2. Then make use of the group by calling `party.protect(name)`

#### syntax


```javascript
party.protect(name, options)
```

Example usage:

```javascript
party.app.get(route1, party.protect(name), (req, res) => {
  ...
})
party.app.post(route2, party.protect(name), (req, res) => {
  ...
})
```



#### parameters

- `name`: The authorization group name to use for the route
- `options`: additional information about the protection
  - `redirect`: The web route to redirect to when logged out. For example you can set up an additional route that displays a web page when logged out.
  - `render`: The HTML file path to render when logged out.
  - `json`: The JSON object to return when logged out, and the request was made as an API request (not a website)
  - `walletconnect`: The Walletconnect Infura ID, to support mobile wallets.
  - `fresh`: when using the default login page, whether the login should ask the user to (re-)connect a wallet from the wallet list, or to use the previously connected wallet if still connected
    - if `true`, the login attempt always displays all the wallets from the list and lets the user select one
    - if `false`, tries to immediately use a previously selected wallet to skip the wallet selection step (This is the default)

The difference between the `redirect` and the `render` option is that, the `redirect` sends the user to a different designated route (for example a `/login` route), whereas `render` DOES NOT take the user to any other URL but just displays the supplied HTML.

#### examples

All examples in this section are the same as the `auth()` examples, except that you're using `party.protect()` instead of `party.auth()`.

##### 1. Default protection

The following example simply authenticates a user's account based on the wallet signature.

```javascript
const party = new Privateparty()
party.add("user")
party.app.get("/", party.protect("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
```

1. The `party.add("user"` line will create a group named "user", which automatically creates the following routes:
    - `POST /privateparty/connect`
    - `POST /privateparty/disconnect`
    - `GET /privateparty/session`
2. Then the express app instance (`party.app`) handles the `GET /` request. But before that, it goes through the `party.protect("user")` middleware.
3. Since the `party.add("user")` did not specify any authorization logic, it will just allow all requests.
4. Therefore, when a user first visits the `/` route, the `req.session` will be null but...
3. After authenticating from the frontend, the `req.session` will contain `{ "user": { "account": <user address> } }`

Unlike the `auth()` example, when you first visit the `/` route, Privateparty will automatically redirect you to its built-in login page.

##### 2. Authorization

By default, Privateparty logs everyone in. But often you will want to only allow certain people in.

You can achieve this with an `authorize(req, account)` function:

```javascript
const Privateparty = require('privateparty')
const allowed = [
  "0xab3b229eb4bcff881275e7ea2f0fd24eeac8c83a",
  "0x1ad91ee08f21be3de0ba2ba6918e714da6b45836",
  "0x829bd824b016326a401d083b33d092293333a830"
]
const party = new Privateparty()
party.add("user", {
  session: "/privateparty/session",
  connect: "/privateparty/connect",
  disconnect: "/privateparty/disconnect",
  authorize: async (req, account) => {
    if (!allowed.includes(account) {
      throw new Error("not allowed")
    }
  }
})
party.app.get("/", party.protect("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```

Note that we are:

1. adding a group named `"user"`
2. and then using the group in the `party.app.get("/", party.protect("user", (req, res) => { . . . })` handler.


##### 3. Multiple protection engines

Sometimes you may want to serve different content based on different roles. You can create roles with engines.

In the following code, we are using 2 engines:

1. user: normal user login flow. sign everyone in
2. admin: admin user login flow. check if the account is included in the ADMIN array, and if not, throw an error

```javascript
const Privateparty = require('privateparty')
const ADMINS = ["0x502b2fe7cc3488fcff2e16158615af87b4ab5c41"]
const party = new Privateparty()
party.add("user", {
  session: "/privateparty/session",
    connect: "/privateparty/connect",
    disconnect: "/privateparty/disconnect",
  },
})
party.add("admin", {
  session: "/privateparty/admin/session",
  connect: "/privateparty/admin/connect",
  disconnect: "/privateparty/admin/disconnect",
  authorize: async (req, account) => {
    if (ADMINS.includes(account)) {
      return { admin: true }
    } else {
      throw new Error("not an admin")
    } 
  }
})
party.app.get("/", party.protect("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
party.app.get("/admin", party.protect("admin"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```

Note that we have two `GET` route handlers here:

1. `GET /`: The normal route for normal users. Because we're using `party.protect("user")`, it will use the `user` group.
2. `GET /admin`: The page where the admins can login. Because we're using `party.protect("admin")`, it will use the `admin` group.


##### 4. Custom logged out handling

By default the `protect()` modifier automatically sends the users to the built-in login page where the user can log in for that role.

But if you want a custom handler, you can do something like this:


```javascript
const Privateparty = require('privateparty')
const allowed = [
  "0xab3b229eb4bcff881275e7ea2f0fd24eeac8c83a",
  "0x1ad91ee08f21be3de0ba2ba6918e714da6b45836",
  "0x829bd824b016326a401d083b33d092293333a830"
]
const party = new Privateparty()
party.add("user", {
  authorize: async (req, account) => {
    if (!allowed.includes(account) {
      throw new Error("not allowed")
    }
  }
})
party.app.get("/login", (req, res) => {
  res.sendFile(__dirname + "/login.html")
})
party.app.get("/", party.protect("user", { redirect: "/login" } ), (req, res) => {
  console.log("session", req.session)
  res.sendFile(__dirname + "/index.html")
})
party.app.listen(3000)
```

Or if you DO NOT want to send the user to a new route but just display the error, you can use the `render` option:


```javascript
const Privateparty = require('privateparty')
const allowed = [
  "0xab3b229eb4bcff881275e7ea2f0fd24eeac8c83a",
  "0x1ad91ee08f21be3de0ba2ba6918e714da6b45836",
  "0x829bd824b016326a401d083b33d092293333a830"
]
const party = new Privateparty()
party.add("user", {
  authorize: async (req, account) => {
    if (!allowed.includes(account) {
      throw new Error("not allowed")
    }
  }
})
party.app.get("/", party.protect("user", { render: __dirname + "/login.html" } ), (req, res) => {
  console.log("session", req.session)
  res.sendFile(__dirname + "/index.html")
})
party.app.listen(3000)
```

##### 5. Token authentication

In addition to using the authenticted user's credentials for authorizing, you can use access token based authorization.

Here's an example:


```javascript
const party = new Privateparty()
party.add("user", {
  tokens: [
    "01127c36-32fa-4c85-b6da-f720796fe679",
    "35161a5c-60f0-4809-8b49-1a662247f5b3"
  ]
})
// Our JSON API endpoint
party.app.get("/api", party.protect("user"), (req, res) => {
  res.json({
    people: ["alice", "bob", "carol"]
  })
})
party.app.get("/", party.protect("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
```


This will:

1. Allow ALL authenticated user to log in in the browser using cookies (since there is no `authorize()` callback to restrict access
2. Allow only those who have access to the access tokens `01127c36-32fa-4c85-b6da-f720796fe679` and `35161a5c-60f0-4809-8b49-1a662247f5b3` to make request to the app.

To make an access token authenticated request, you need to set the HTTP request header's `Authorization` field as `token <ACCESS_TOKEN>`. Example:

```javascript
fetch("https://protectedendpoint.com/api", {
  headers: {
    "Authorization": "token 01127c36-32fa-4c85-b6da-f720796fe679"
  }
}).then((r) => {
  return r.json()
}).then((r) => {
  console.log(r)
})
```

##### 6. Mobile wallet support

The following example simply authenticates a user's account based on the wallet signature.

```javascript
const party = new Privateparty()
party.add("user")

// Use the walletconnect Inufra ID to support mobile wallets on the default login page
party.app.get("/", party.protect("user", { walletconnect: "667750972a89441ea5d276ed16d7eef0" }), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
```


### contract()

Creates and returns a web3 contract methods object, which can be chained to call web3 methods.

#### syntax

```javascript
const methods = party.contract(web3, abi, contract_address)
```

#### parameters

- `web3`: an initialized web3 object
- `abi`: an ABI array
- `contract_address`: the contract address

#### examples

##### 1. Token balance contract calls

To authorize users based on the blockchain state associated with their authenticated wallet accounts, you will need to query the blockchain.

In this case you will need to instantiate a web3 object and use the built-in `contract()` convenience method to call web3 contract methods

```javascript
// Example. Use your own RPC URL
const Privateparty = require('privateparty')
const { createAlchemyWeb3 } = require("@alch/alchemy-web3");
// Use your own JSON-RPC ENDPOINT instead of the URL below!
const web3 = createAlchemyWeb3("https://eth-mainnet.alchemyapi.io/v2/YAB7qBnOb0pceNn29u1v_PATpqKUN623")
let party = new Privateparty()
party.add("user", {
  authorize: async (req, account) => {
    const UNI = "0x1f9840a85d5af5bf1d1762f925bdaddc4201f984"
    const abi = [{
      "constant": true,
      "inputs": [ { "internalType": "address", "name": "owner", "type": "address" } ],
      "name": "balanceOf",
      "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ],
      "payable": false,
      "stateMutability": "view",
      "type": "function"
    }], UNI)
    const balance = await party.contract(web3, abi, UNI).balanceOf(account)
    if (balance >= 100) {
      return { balance } 
    } else {
      throw new Error("you need to own at least 100 $UNI)
    }
  }
})
```

Or equivalently, you can use the built-in `party.abi.erc721` instead of hardcoding the `const abi` part, like this:

```javascript
// Example. Use your own RPC URL
const Privateparty = require('privateparty')
const { createAlchemyWeb3 } = require("@alch/alchemy-web3");
// Use your own JSON-RPC ENDPOINT instead of the URL below!
const web3 = createAlchemyWeb3("https://eth-mainnet.alchemyapi.io/v2/YAB7qBnOb0pceNn29u1v_PATpqKUN623")
let party = new Privateparty()
party.add("user", {
  authorize: async (req, account) => {
    const UNI = "0x1f9840a85d5af5bf1d1762f925bdaddc4201f984"
    const balance = await party.contract(web3, party.abi.erc721, UNI).balanceOf(account)
    if (balance >= 100) {
      return { balance } 
    } else {
      throw new Error("you need to own at least 100 $UNI)
    }
  }
})
```

##### 2. Admin login

Sometimes you may want to build an adming interface that ONLY allows the owner of the contract to login.

Privateparty includes an abi interface for [ownable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol) in addition to erc20 and erc721 ABIs, so you can take advantage of this as well:

```javascript
const Privateparty = require('privateparty')
const { createAlchemyWeb3 } = require("@alch/alchemy-web3");
const web3 = createAlchemyWeb3(<YOUR JSON-RPC ENDPOINT URL>)
const party = new Privateparty()
party.add("admin", {
  session: "/privateparty/admin/session",
  connect: "/privateparty/admin/connect",
  disconnect: "/privateparty/admin/disconnect",
  authorize: async (req, account) => {
    const mfers = "0x79fcdef22feed20eddacbb2587640e45491b757f"
    let owner = await party.contract(web3, party.abi.ownable, mfers).owner().call()
    if (owner.toLowerCase() === account) {
      return { admin: true }
    } else {
      throw new Error("not an admin")
    }
  }
})
```

### abi

Built-in convenience module for frequently used ABIs:

#### 1. erc20

```javascript
// use the party.abi.erc20 instead of hardcoding the ABI
const party = new Privateparty()
let balance = await party.contract(web3, party.abi.erc20, UNISWAP_ADDRESS).balanceOf(account).call()
```

#### 2. erc721

```javascript
// use the party.abi.erc721 instead of hardcoding the ABI
const party = new Privateparty()
let tokenURI = await party.contract(web3, party.abi.ownable, MFERS_NFT_ADDRESS).tokenURI(tokenId).call()
```

#### 3. ownable

```javascript
// use the party.abi.ownable instead of hardcoding the ownable ABI
const party = new Privateparty()
let owner = await party.contract(web3, party.abi.ownable, mfers).owner().call()
```

### app

The `app` object internally created with `app = express()`.

You can use the `app` object just like you would with any express app instance.

#### examples

```javascript
const party = new Privateparty()
party.add("members", {
  authorize: async (req, account) => {
    // some membership checking logic
  }
})
party.app.get("/", (req, res) => {
  // public route.
  // no authentication and no authorization
})
party.app.get("/members", party("members"), (req, res) => {
  // members only logic
})
// Don't forget to start the app by listening to a port!
party.app.liten(3000)
```

### express

The expressjs module.



## Browser client

### constructor

#### syntax

```javascript
const party = new Privateparty(config)
```

#### parameters

- `config`: configuration
  - `host`: **(optional)** specify the host in case you wish to make a cross-origin request to a privateparty server hosted on another domoain.
  - `walletconnect`: **(optional)** Specify this field to support mobile and desktop wallets. The `walletconnect` attribute is the [Walletconnect infuraId attribute](https://github.com/Web3Modal/web3modal/blob/master/docs/providers/walletconnect.md?plain=1#L22) (Go to [Infura](https://infura.io/) to sign up and get the Infura project ID).

#### return value

- `party`: An instantiated privateparty client

#### examples

##### 1. basic

```javascript
const party = new Privateparty()
```

##### 2. cross origin connection

Let's say your privateparty server is running at https://myprivatepartyserver.com - You can connect to it using the `host` attribute.

> **NOTE**
>
> You MUST set the CORS support on the server side to make this work http://localhost:56503/#/?id=_3-cross-origin-login-support

```javascript
const party = new Privateparty({
  host: "https://myprivatepartyserver.com"
})
```


##### 3. mobile & desktop wallet support

To support mobile and desktop wallets, we need to use [Walletconnect](https://walletconnect.com/). For this, we need to get a project ID from [Infura](https://infura.io) and set it as the `walletconnect` attribute. Example:

```javascript
const party = new Privateparty({
  walletconnect: "27e484dcd9e3efcfd25a83a78777cdf1"   // USE YOUR OWN INFURA ID!
})
```

---

### connect()

#### syntax

```javascript
let session = await party.connect(name, payload, options)
```

#### parameters

- `name`: the name of a privateparty role. Automatically connects to the endpoints defined on the privateparty backend with the same name.
- `payload`: **(optional)** additional payload that will be passed to the Privateparty server. The Privateparty server will be able to inspect `req.body.payload` in its authorization logic.
- `options`: **(optional)** an object describes how the connection shall be made. includes the following attributes:
  - `fresh`: whether the login should ask the user to (re-)connect a wallet from the wallet list, or to use the previously connected wallet if still connected
    - if `true`, the login attempt always displays all the wallets from the list and lets the user select one
    - if `false`, tries to immediately use a previously selected wallet to skip the wallet selection step (This is the default)

#### return value

- `session`: The authenticated and authorized session object for this connection.
  - `account`: the authenticated account
  - `expiresIn`: how long this session will be valid for since the issued time (`iat`), in seconds. (default: 60 * 60 * 24 * 30, or 30 days)
  - `jwt`: the full JWT string
  - `auth`: **(optional)** additional attributes set by the privateparty server if needed. Only included when you return something from the `authorize()` callback when calling `party.add()`.

The same `session` object will be stored inside the cookie and will be accessible subsequently via `party.session()`

#### examples

##### 1. basic connection

Let's assume the Privateparty backend has added a role named "user":

```javascript
const party = new Privateparty()
party.add("user")
party.app.get("/", party.auth("user"), (req, res) => {
  // public route.
  // no authentication and no authorization
})
party.app.listen(3000)
```

Above code will set up the default endpoints:

- `GET /privateparty/session`
- `POST /privateparty/connect`
- `POST /privateparty/disconnect`

We can automatically connect to those endpoints simply by specifying the name of the role (`"user"`):


```javascript
// Browser code
const party = new Privateparty()
let session = await party.connect("user")
if (session) document.write("logged in: " + session.account)
```

##### 2. custom engine connection

if you've set up multiple engines on the backend side, you can connect to the custom endpoints by initializing the `Privateparty` object with custom endpoints.

For example let's take an example with multiple roles ("user" and "admin"):

```javascript
const party = new Privateparty()

// "user" role
party.add("user")

// "admin" role
party.add("admin", {
  session: "/privateparty/admin/session",
  connect: "/privateparty/admin/connect",
  disconnect: "/privateparty/admin/disconnect",
  authorize: (req, account) => {
    const ADMIN = "0x502b2FE7Cc3488fcfF2E16158615AF87b4Ab5C41"
    if (account === ADMIN) {
      return { admin: true }
    } else {
      throw new Error("Not an admin")
    }
  }
})

// User UI
party.app.get("/", party.auth("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})

// Admin UI
party.app.get("/admin", party.auth("admin"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/admin.html")
})
party.app.listen(3000)
```

Now, from the browser, lets try to login as admin:

```javascript
const party = new Privateparty()
let session = await party.connect("admin")
```

The `party.connect("admin")`:

1. automatically discovers the `/admin/connect` endpoint
2. makes a POST request to it
3. the `authorize()` callback in the backend takes care of the authorization for the admin role

##### 3. authenticate with custom payload

Often, the authorization logic may require more than just the user account. For example, the user may authenticate with a specific NFT (`tokenId` and `contract`), in which case the frontend needs to pass more data to the Privateparty server. Here's an example:

```javascript
const party = new Privateparty()
await party.connect("user", {
  contract: "0x6866ed9a183f491024692971a8f78b048fb6b89b",
  tokenId: "55005454344647406361450320675654878134478584534017520891306338141495783002503"
})
```

The server may implement an `authorize(req, account)` function in the engine that looks like this:

```javascript
const Privateparty = require('privateparty')
const { createAlchemyWeb3 } = require("@alch/alchemy-web3");
const web3 = createAlchemyWeb3(<YOUR JSON-RPC ENDPOINT URL>)
const party = new Privateparty()
const { app, express, auth } = new Privateparty({
  engines: {
    user: {
      authorize: async (req, account) => {
        const collection = req.body.auth.contract
        const tokenId = req.body.auth.tokenId
        const abi = [{
          "inputs": [{ "internalType": "uint256", "name": "tokenId", "type": "uint256" }],
          "name": "ownerOf",
          "outputs": [{ "internalType": "address", "name": "", "type": "address" }],
          "stateMutability": "view",
          "type": "function"
        }]
        let contract = new web3.eth.Contract(abi, collection)

        // allow login ONLY if the current account owns the tokenId
        let owner = await contract.methods.ownerOf(tokenId).call()
        if (owner.toLowerCase() === account.toLowerCase()) {
          return { tokenId, collection }
        } else {
          throw new Error("the user is not the token owner")
        }
      }
    }
  }
})
```

Note that the additional payload is included under the attribute `req.body.auth`.

##### 4. let users select an account from the wallet

By default, Privateparty automatically uses the default account. But sometimes you may want to let the user select another account from the wallet.

To connect this way, you can:

```javascript
const party = new Privateparty()
await party.connect("user", null, { fresh: true })
```

Note:

- The second argument is `null` (we are not passing any payload)
- The third argument (`options`) of the `connect()` method is `{fresh: true}`. This tells privateparty to make a fresh connection, which lets the user select from all the accounts in the wallet instead of the default one.

Of course, you can do this while passing a payload too:

```javascript
const party = new Privateparty()
await party.connect(
  "user",
  { contract: "0x6866ed9a183f491024692971a8f78b048fb6b89b", tokenId: "55005454344647406361450320675654878134478584534017520891306338141495783002503" },
  { fresh: true }
)
```

Now the second argument is the `payload`, and the third argument is the `options`

---

### session()

The `session()` method is used to get the current session.

#### syntax

```javascript
let session = await party.session(name)
```

#### parameters

- `name`: The name of the role for the session

#### return value

- `session`: the global session object for the specified name
  - `account`: the authenticated account
  - `expiresIn`: how long this session will be valid for since the issued time (`iat`), in seconds. (default: 60 * 60 * 24 * 30, or 30 days)
  - `iat`: when this session was issued
  - `auth`: **(optional)** additional attributes set by the privateparty server if needed. Only included when you return something from the `authorize()` callback when calling `party.add()`.


#### examples

```javascript
let session = await party.session("user")
console.log(session)
// The session may look something like:
//
//  {
//    account: "0x502b2FE7Cc3488fcfF2E16158615AF87b4Ab5C41",
//    auth: {
//      collection: "0x6866ed9a183f491024692971a8f78b048fb6b89b",
//      tokenId: "55005454344647406361450320675654878134478584534017520891306338141495783002503"
//    }
//  }


let admin_session = await party.session("admin")
console.log(admin_session)
// The admin_session may look something like:
//
//  {
//    account: "0x502b2FE7Cc3488fcfF2E16158615AF87b4Ab5C41",
//    auth: {
//      admin: true
//    }
//  }
//
```

---

### disconnect()

Clears the cookies and logs out of all sessions

#### syntax

```javascript
await party.disconnect(name)
```

#### parameters

- `name`: The name of the session to disconnect from

#### return value

- none


---

## Node.js client

Privateparty utilizes [JWT](https://en.wikipedia.org/wiki/JSON_Web_Token), which means you can take the same session created from the browser, and use it in a server setting, or any other non-browser setting.

Here's how the node.js client differ from the browser client:

1. No cookie: Unlike the partyconnect browser client, which takes care of setting cookies automatically after the connection, the `partypass` node.js client simply makes a request to the privateparty server to create a JWT token. From that point on, it's up to you what to do with the JWT token.
2. Use private key directly: The `partyconnect` browser client was built for a seamless in-browser usage, therefore uses whatever wallet is injected into the browser. But the `partypass` node.js client is primarily supposed to run in node.js setting, and there is no injected browser wallet. Therefore, to initialize `partypass` you must pass a private key, which will be used to sign messages.

Basically, the `partyconnect` library is primarily for in-browser usage, whereas `partypass` is a super minimal library for creating JWTs, using a private key you provided.

### constructor

#### syntax

```javascript
const pass = new Partypass(config)
```

#### parameters

- `config`
  - `host`: The privateparty host URL
  - `key`: The private key in hex form (without a `0x` prefix)

#### return value

- `pass`: an initialized partypass object, which can be used to create sessions

#### example

```javascript
const pass = new Partypass({
  host: "http://localhost:3000",
  key: "cd76c32ffeae94b725b40b1f58ffc793d5b0e96596f8d067f29d385894f16424" // private key
})
```

In most cases you do NOT want to hardcode private keys, so the code may actually look something like:

```javascript
const pass = new Partypass({
  host: "http://localhost:3000",
  key: process.env.PRIVATE_KEY  // use environment variables
})
```

### create()

Create a session using the initialized Partypass object

> `await pass.create(name, payload)` is equivalent to `await pass.request(await pass.build(name), payload)`

#### syntax

```javascript
const session = await pass.create(name, payload)
```

#### parameters

- `name`: the name of a privateparty role. Automatically connects to the endpoints defined on the privateparty backend with the same name.
- `payload`: **(optional)** additional payload that will be passed to the Privateparty server. The Privateparty server will be able to inspect `req.body.payload` in its authorization logic.

#### return value

- `session`: The authenticated and authorized session object for this connection.
  - `account`: the authenticated account
  - `expiresIn`: how long this session will be valid for since the issued time (`iat`), in seconds. (default: 60 * 60 * 24 * 30, or 30 days)
  - `jwt`: the full JWT string
  - `auth`: **(optional)** additional attributes set by the privateparty server if needed. Only included when you return something from the `authorize()` callback when calling `party.add()`.

The same `session` object will be stored inside the cookie and will be accessible subsequently via `party.session()`

#### examples

Let's assume the Privateparty backend has added a role named "user":

```javascript
// server.js
const party = new Privateparty()
party.add("user")
party.app.get("/", party.auth("user"), (req, res) => {
  // public route.
  // no authentication and no authorization
})
party.app.listen(3000)
```

Now create a client that will make a request to above privateparty server, as `client.js`:

```javascript
// client.js
const Partypass = require('partypass')
const pass = new Partypass({
  host: "http://localhost:3000",
  key: "cd76c32ffeae94b725b40b1f58ffc793d5b0e96596f8d067f29d385894f16424" // replace with your private key
})
let session = await pass.create("user")
console.log(session)
```

and run `node client`. You will get a session object with a JWT included.


### session()

check if the session has expired or not

#### syntax

```javascript
const session = await pass.session(name)
```

#### parameters

- `name`: the name of the role

#### return value

- `session`: returns the session info for the `name` role. If expired, returns `null`.


### build()

build a signed request to send to a privateparty server.

> a `create()` call is equivalent to calling `build()` and then calling `request()` 

#### syntax

```javascript
const req = await pass.build(name)
```

#### parameters

- `name`: the name of the role

#### return value

- `req`: the prepared request object. This can be sent to the associated privateparty server (must be the same host as the connected `host`)
  - `str`: the nonce message that was signed
  - `sig`: the signature (personal signature)
  - `url`: the endpoint to send the request to


### request()

send a built request created with the `build()` method to get the corresponding session back from the privateparty server

#### syntax

```javascript
const session = await pass.request(req, payload)
```

#### parameters

- `req`: the prepared request object created with `pass.build()`. This can be sent to the associated privateparty server (must be the same host as the connected `host`)
  - `str`: the nonce message that was signed
  - `sig`: the signature (personal signature)
  - `url`: the endpoint to send the request to
- `payload`: **(optional)** an extra payload to send to the privateparty server.

#### return value

- `session`: returns the created session info for the `name` role.
  - `account`: the authenticated account
  - `expiresIn`: how long this session will be valid for since the issued time (`iat`), in seconds. (default: 60 * 60 * 24 * 30, or 30 days)
  - `jwt`: the full JWT string
  - `auth`: **(optional)** additional attributes set by the privateparty server if needed. Only included when you return something from the `authorize()` callback when calling `party.add()`.
