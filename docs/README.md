# 简介

## 什么是 OpenAvatar-SDK ?

OpenAvatar 是全球第一个可视化 Web3 DID协议

OpenAvatar-SDK 是 OPENAVATAR 的一个开源组件，它是从Privateparty库派生的开源框架，可以非常容易地构建区块链认证的 Web3 应用程序。

只需几行代码，即可以搭建出具有复杂(钱包身份验证/授权)逻辑的 Web3 应用程序。

![ppclient.png](ppclient.png)

结果如下：

![login.gif](login.gif)


1. **身份验证：** 使用区块链钱包签名登录
2. **授权：** 根据区块链状态或链下查询授权访问您的页面
3. **简单：** 只需一行 JavaScript 代码即可设置。 没有复杂的步骤。
4. **开源：** 100% 开源框架，没有第三方API。

## OpenAvatar-SDK 能做什么?

OpenAvatar-SDK 利用密码学和区块链状态数据，取代传统的身份验证方法。

这意味着我们可以构建任何想象的应用。 您只是用 OpenAvatar-SDK 认证方式替换传统的用户数据库。

1. **简单身份验证：**  使用Web3钱包登录应用程序（允许任何人使用钱包登录）
2. **仅限邀请的应用程序：** 仅允许地址列表中的人登录
3. **基于 NFT 的授权：** 根据钱包拥有的 NFT 登录
4. **基于 ERC20 的授权：** 基于钱包拥有的 ERC20 代币登录
5. **空投：** 可以通过曾与某个合约交互过的地址来实施空投。
6. **More：** 根据任何区块链查询实现授权。


## 社群支持

提问或反馈：:

1. Twitter: https://twitter.com/@yielddao
2. GitHub: https://github.com/openavatar

---


# 快速入门

## 1. Server
服务端有两个步骤要做：

1. **设置后端：** `privateparty` 模块让您轻松设置受区块链钱包签名保护的 [express.js]服务器(https://expressjs.com/) 。
2. **连接后端：** 设置后端后，您可以使用 `partyconnect` 库从浏览器进行连接。

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


## 跨域登录

有时你的前端代码与后端可能运行在不同的域上。

在此种情况下，可以应用内置的CORS功能，仅允许指定域使用服务端进行身份验证。

需要如下设置：

1. OpenAvatar-SDK 服务端运行在 3007 端口
2. 前端运行在 8080 端口

前端网站（ 8080 ）将使用 OpenAvatar-SDK 服务器 http://localhost:3007 进行身份验证

### 服务端（Server)

将下面代码另存为 `index.js`：

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

### 客户端（Client）

将下面代码另存为 `index.html`：

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

唯一不同的部分是初始化步骤：

```javascript
const party = new Privateparty({
  host: "http://localhost:3000"
})
```

默认情况下， 使用客户端（partyconnect.js）向同一个域发出请求。 但是您可以在初始化 Privateparty 客户端时通过设置 `host` 属性来自定义端点。

### 运行

首先启动OpenAvatar-SDK 服务端：

```
node index
```

然后使用下面命令在 8080 端口启动 `index.html` 服务：

```
npx http-server
```

Now open the browser at http://localhost:8080 and it should work as intended.


## 跨平台登录

OpenAvatar 由 [JWT](https://en.wikipedia.org/wiki/JSON_Web_Token) 提供实现支持。

这意味着可以在**浏览器之内和外部**使用相同的生成令牌， “浏览器之外”可以是任何地方，包括：

- 移动应用
- 服务器
- 无服务器功能（AWS lambda、vercel、netlify 等）
- 物联网设备
- 等等..

让我们看看如何使用 node.js 应用程序进行身份验证和授权。

### 服务端（Server）

为了简单起见，我们创建一个允许任何人登录的OpenAvatar-SDK服务器（因此没有 `authorize()` 函数）：
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


### 客户端 （Client）

为此，我们使用一个名为 `partypass` 的 node.js 客户端,  和 `cross-fetch`（发出 fetch 请求）。

```
npm install partypass
```

然后创建一个名为 `client.js` 的文件：

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

运行命令： `node client` 

1. `client.js` 向 OpenAvatar 服务器的3000端口 发出请求以获取 Session 会话
2. 然后它将使用授权头中的`session.jwt`向`/api`端点发出经过身份验证的请求，并成功。

将打印下面信息：

```
{ status: 'Logged in!' }
```

接下来，为了确保在没有携带令牌时请求会失败，我们试试发出相同的请求，但没有带上授权头：

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

将打印下面信息：

```
{ error: 'not logged in' }
```

## 一行代码实现 登录/注销

我们已经研究了使用 JavaScript 进行身份验证的方法。 但是还有一种更简单的入门方法。 您需要做的就是将用户登录信息发送到 “gate” 页面。

### 服务端 （Server）

我们首先编写一个名为 `app.js` 的文件来设置一个简单的 OpenAvatar-SDK 服务端。


```javascript
const party = new Privateparty()
party.add("user")
party.app.get("/", (req, res) => {
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```

当用户进入`/`路由时，无论是否登录，都会显示`index.html`页面。 我们看一下 `index.html` ：

### 客户

以下代码不使用任何 JS 代码进行身份验证。 相反，只需添加一行代码`<a id='account' href='/privateparty/gate/user?callback=/'></a>`，用户就可以点击打开“gate”页面来登录或注销，然后返回（有点像为“facebook connect”打开一个 facebook 登录页面并在登录后重定向回来）：


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
运行 `node app.js` 启动服务器并访问 http://localhost:3000 - 会看到：

![gate.gif](gate.gif)

> 注意浏览器 URL 的变化。
>
> - 当用户点击“登录”时，它会转到 /privateparty/gate/user?callback=/
> - 登录后自动重定向回“/”，因为回调
> - 注销过程相同。注销后，它会自动重定向回“/”

让我们通过 HTML 来看看发生了什么。

1. 首先，`render()` 方法检查 `user` 会话，如果已经登录则显示帐户信息，否则就 “登录”。这部分我们很熟悉。
2. 接下来，注意 `<a>` 标签。它有一个 `/privateparty/gate/user?callback=/` 的 `href` 属性。让我们分解一下。
    - `/privateparty/gate/user` 是内置“gate”页面的默认路由，允许用户登录（如果已注销）或注销（如果已登录）。
      - 该路由是根据角色名称自动生成的。例如，要将用户发送到“admin”角色登录页面，链接将是“/privateparty/gate/admin”。
    - `?callback=/` 部分告诉网关页面在用户登录（或注销）后将其重定向到哪个 URL。
      - 在这种情况下，我们希望登录页面自动将用户送回`/`路由，所以`callback`是`/`。

### 移动端支持

要自动支持内置gate页面的移动钱包，您需要使用带有 `walletconnect` 属性（Infura ID）的自定义 `gate` 配置，来初始化 Privateparty ：

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


## 更多示例

查看 GitHub 上的 [demo 文件夹](https://github.com/openavatar/openavatar-server/tree/main/demo) 以获取更多示例。

## React 组件

如果使用 react? 可以试试 react 组件: [Partybutton](https://partybutton.papercorp.org/)

![partybutton.png](partybutton.png)

---

# 安装

可以使用2个库（服务器端和客户端）来实现 OpenAvatar Web3 应用程序：

1. `privateparty`：OpenAvatar 服务器端模块 
2. `partyconnect`：OpenAvatar 的浏览器客户端=>登录后自动使用浏览器钱包并设置cookies。
3. `partypass`：OpenAvatar 的 node.js 客户端 => 无状态客户端，用于向 OpenAvatar 服务器发出请求并在 JWT 中取回新会话。
4. 

## 服务端（Server）

安装服务端:

```
npm install privateparty
```

然后，在你的应用程序中使用该模块，如下所示：

```javascript
const Privateparty = require('privateparty')
const party = new Privateparty()
```

## 浏览器客户端

### 从CDN加载

如下所示，在前端APP里（从CDN）加载

```html
<script src="https://unpkg.com/partyconnect/dist/partyconnect.js"></script>
```

### 引入（Import）

```
npm install partyconnect
```

然后进行初始化：

```javascript
// CJS
const Privateparty = require('partyconnect')
const party = new Privateparty(config)
```

或者

```javascript
// ESM
import Privateparty from 'partyconnect'
const party = new Privateparty(config)
```

> 如果代码运行正常，但在尝试使用 webpack 打包生产时遇到问题，可能是因为 webpack 5 已开始去除 node.js core module（对于任何使用 node.js 核心模块的库）。 请参阅本文以了解如何解决此问题：https://www.alchemy.com/blog/how-to-polyfill-node-core-modules-in-webpack-5

## Node.js 客户端

### 导入（Import）

```
npm install partypass
```

然后初始化:

```javascript
// CJS
const Partypass = require('partypass')
const pass = new Partypass(config)
```

或者

```javascript
// ESM
import Partypass from 'partypass'
const pass = new Partypass(config)
```

### 从CDN加载

虽然partypass 是一个node.js客户端，但在某些情况下您可以在Web环境中使用它。

例如，服务器可以使用`pass.build()`创建一个pass创建请求，并将其发送到用户的浏览器，用户使用`pass.request()`发出请求。 在这种情况下，您还可以使用 CDN JS：

包括在web前端应用中：

```html
<script src="https://unpkg.com/partypass/dist/partypass.js"></script>
```

---

# API

## 服务端（Server）

对于后端需要使用包 `privateparty`。 只需实例化一个新 `Privateparty` 实例，它就会为您搭建一个Web3钱包保护的 Web 应用后端。

### 构造函数

#### 语法

```javascript
const party = new Privateparty(config)
```

#### 参数

- `config`: openavatar 服务端配置 
  - `secret`: **(可选)** 用于签署 cookie 的字符串
    - 参考 https://github.com/expressjs/cookie-parser#cookieparsersecret-options
    - 如果未指定，它将在每次服务器重新启动时使用uuid自动生成一个secret密钥 [uuid](https://github.com/uuidjs/uuid).
  - `cors`: **(可选)** 如果您想支持 CORS（跨源请求），请添加此属性。
    - 参考 https://github.com/expressjs/cors#configuration-options
  - `app`: **(可选)** 注入现有 express.js 应用实例
  - `express`: **(可选)** 注入一个现有的 express 模块
  - `gate`: **(可选)** 内置gate页面配置
    - `walletconnect`: Walletconnect Infura ID，用于支持移动钱包。
    - `fresh`: 使用默认登录页面时，登录是否要求用户（重新）连接钱包列表中的钱包，或者如果仍然连接则使用之前连接的钱包
       - 如果设置 `true`，登录总是显示列表中的所有钱包并让用户选择一个
       - 如果设置 `false`，跳过钱包选择步骤，使用之前选择的钱包（这是默认设置）

#### 返回值

- `party`: 经过初始化的 privateparty 实例，包含以下属性：
  - `app`: 通过调用 `const app = express()` 在内部创建的“app”实例
  - `express`: express 模块
  - `auth`: 认证授权功能
  - `protect`: 认证授权功能+错误处理
  - `add`: 添加授权组的功能

> the `auth` method only tells you if the authorization results in a legitimate session or not, whereas the `protect` method is used to do what `auth` does but also automatically redirect to a logged out page or display a logged out page.
> `auth` 方法仅告诉你授权是否生成合法会话，而 `protect` 方法用于执行 `auth` 所做功能，但会自动重定向到已注销的页面或显示已注销的页面 .

#### 示例

##### 1. 最小化服务端

```javascript
const party = new Privateparty()
```

##### 2. 具有固定签名密钥的服务器

```javascript
const party = new Privateparty({
  secret: "top secret"
})
```

##### 3. 跨域登录

In the following example, we have a privateparty server running at port 3001, and it allows requests from not just the port 3001 but also 3000, since we specified the origin http://localhost:3000

在下面的例子中，我们有一个运行在 3001 端口的 openavatar 服务器，它不仅允许来自端口 3001 的请求，还允许来自 3000 的请求，因为我们指定了源 http://localhost:3000

```javascript
const party = new Privateparty({
  cors: {
    credentials: true,
    origin: ["http://localhost:3000"]
  }
})
party.listen(3001)
```

##### 4. 具有动态源解析的跨域登录

使用 CORS 模块 (https://github.com/expressjs/cors#configuring-cors-w-dynamic-origin) 中的动态源配置选项，您可以动态解析请求源并授权：

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

##### 5. 与已有 express.js 应用集成

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

添加一个组到 OpenAvatar

#### 语法

```javascript
await party.add(name, config)
```

#### 参数

- `name`: 组名 (每个组必须唯一)
- `config`: 每个组的配置选项
  - `session`：（可选）查询此引擎当前会话的 GET 路径。 （默认：`/privateparty/session/${name}`）
  - `connect`：（可选）为此引擎创建会话的 POST 路径（默认值：`/privateparty/connect/${name}`）
  - `disconnect`：（可选）销毁此引擎会话的 POST 路径（默认值：`/privateparty/disconnect/${name}`）
  - `gate`：（可选）此引擎的网关页面路由（“登录/注销”页面）（默认：`privateparty/gate/${name}`）
  - `authorize`：一个接受两个或多个参数的函数`req`（从express传递的传入请求对象），`account`（经过身份验证的钱包地址）和可选的`contracts`（仅当您指定另一个属性`contracts `，解释如下）。
    - 要禁止基于请求的会话，只需在函数中抛出错误。
    - 要授权会话，请不要抛出函数。另外，这个函数的返回值会自动设置为会话的 `auth` 属性
  - `expire`：（可选）会话持续时间（会话过期的秒数）。默认值为“1000 * 60 * 60 * 24 * 30”（30 天）。
  - `tokens`：（可选）允许访问令牌的数组
  - `contracts`：（可选）用于定义一个或多个合约的声明性对象，它将被初始化并注入到 `authoirze()` 处理程序

#### 返回值

none

#### 示例

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

授权中间件，可以添加到任何路由中。

要将授权逻辑添加到任何路由，您需要：

1. 首先通过`add()`方法定义一个授权组及其行为
2. 然后通过调用 `party.auth(name)` 来使用该组

#### 语法

```javascript
party.app.get(route1, party.auth(name), (req, res) => {
  ...
})
party.app.post(route2, party.auth(name), (req, res) => {
  ...
})
```

#### 参数

- `name`: 用于路由处理程序的授权组名称


#### 示例

##### 1. 缺省身份验证

以下示例仅根据钱包签名对用户帐户进行身份验证。

```javascript
const party = new Privateparty()
party.add("user")
party.app.get("/", party.auth("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
```


1. `party.add("user"` 将创建一个名为 "user" 的组，它会自动创建以下路由：
     - `POST /privateparty/connect`
     - `POST /privateparty/disconnect`
     - `GET /privateparty/session`
2. 然后 express 应用实例（`party.app`）处理 `GET /` 请求。 但在此之前，它会通过 `party.auth("user")` 中间件。
3. 由于 `party.add("user")` 没有指定任何授权逻辑，所以允许所有请求。
4. 因此，当用户第一次访问`/`路由时，`req.session`将为空，但是...
5. 前端认证后，`req.session` 将包含 `{ "user": { "account": <user address> } }`

##### 2. 授权

默认情况下，Privateparty 让所有人都登录。但通常你想仅允许某些人登录。

可以使用 `authorize(req, account)` 函数实现此目的：

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

请注意：

1. 添加一个名为 `"user"`的组
2. 然后在 `party.app.get("/", party.auth("user", (req, res) => { . . . })` 句柄程序中使用该组。


##### 3. 多身份验证引擎

在下面的代码中，我们使用了 2 个引擎：

1. user：普通用户登录流程。 运行所有人登录
2. admin：管理员用户登录流程。 检查帐户是否包含在 ADMIN 数组中，如果没有，则抛出错误

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

请注意，我们在这里有两个 `GET` 路由处理程序：

1. `GET /`：普通用户的正常路由。 因为我们使用的是 `auth("user")`，所以它将使用 `user` 组。
2. `GET /admin`：管理员可以登录的页面。 因为我们使用的是 `auth("admin")`，所以它将使用 `admin` 组。

### protect()

与 auth() 类似，另外如果未授权，则会自动重定向到内置登录页面。

> `auth()` 方法在未授权时 `req.session` 返回一个`null` 值，仅此而已。 `protect()` 方法重定向到登录页面。

要将保护逻辑添加到任何路由，您需要：

1. 首先通过`add()`方法定义一个授权组及其行为
2. 然后通过调用 `party.protect(name)` 来使用该组

#### 语法


```javascript
party.protect(name, options)
```

示例用法:

```javascript
party.app.get(route1, party.protect(name), (req, res) => {
  ...
})
party.app.post(route2, party.protect(name), (req, res) => {
  ...
})
```



#### 参数

- `redirect`：注销时重定向的路由。例如，您可以设置一个额外的路由，在注销时显示一个页面。
  - `render`：注销时要渲染的 HTML 文件路径。
  - `json`：注销时返回的 JSON 对象，作为 API 请求发出（不是网站）
  - `walletconnect`：Walletconnect Infura ID，支持移动钱包。
  - `fresh`：使用默认登录页面时，登录是否要从钱包列表中（重新）连接一个钱包，如果仍然在连接，则使用之前连接的钱包
    - 如果设置 `true`，登录显示列表中的所有钱包，并让用户选择一个
    - 如果设置 `false`，跳过钱包选择步骤，使用之前选择的钱包（这是默认设置）

`redirect` 和 `render` 选项之间的区别在于，`redirect` 将用户发送到不同的指定路由（例如 `/login` 路由），而 `render` 不会将用户带到任何其他 URL，只显示提供的 HTML。

#### 示例

本节中的所有示例都与 `auth()` 示例相同，只是你使用的是 `party.protect()` 而不是 `party.auth()`。

##### 1. 缺省保护

以下示例仅基于钱包签名，来对用户帐户进行身份验证。

```javascript
const party = new Privateparty()
party.add("user")
party.app.get("/", party.protect("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
```

1. `party.add("user"` 将创建一个名为 "user" 的组，它会自动创建以下路由：
     - `POST /privateparty/connect`
     - `POST /privateparty/disconnect`
     - `GET /privateparty/session`
2. 然后 express app 实例（`party.app`）处理`GET /`请求。 在此之前，它会通过 `party.protect("user")` 中间件。
3. `party.add("user")` 未指定任何授权逻辑，所以允许所有请求。
4. 因此，当用户第一次访问`/`路由时，`req.session`将为空，但是...
5. 前端认证后，`req.session` 将包含 `{ "user": { "account": <user address> } }`

与 `auth()` 示例不同，当您第一次访问 `/` 路由时，Privateparty 会自动将您重定向到其内置的登录页面。

##### 2. 授权

缺省情况下，Privateparty 允许所有人登录。如果只允许某些人登录。

可以使用 `authorize(req, account)` 函数实现此目的：

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

注意：

1. 添加一个名为`"user"`的组
2. 然后在 `party.app.get("/", party.protect("user", (req, res) => { . . . })` 句柄程序中使用组。


##### 3. 多重保护引擎

如果需要为不同的角色提供不同的内容。 可以使用引擎创建角色。

在下面的代码中，我们使用了 2 个引擎：

1. user：普通用户登录流程。 让所有人登录
2. admin：管理员用户登录流程。 检查帐户是否包含在 ADMIN 数组中，如果没有，则抛出错误

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

请注意，我们在这里有两个 `GET` 路由处理程序：

1.`GET /`：普通用户的正常路由。 因为我们使用的是 `party.protect("user")`，所以它将使用 `user` 组。
2. `GET /admin`：管理员可以登录的页面。 因为我们使用的是 `party.protect("admin")`，所以它将使用 `admin` 组。

##### 4. 自定义注销流程

默认情况下，`protect()` 修饰符会自动将用户发送到登录页面，让用户登录该角色。

如果你想要一个自定义处理程序，可以这样做：


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

或者，如果您不想将用户发送到新路由，只想显示错误，可以使用 `render` 选项：

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

##### 5. Token 令牌认证

除了使用已登录用户的身份验证凭据进行授权之外，您还可以使用访问令牌的进行授权。

示例如下：


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


这将：

1. 允许所有经过身份验证的用户使用cookies登录浏览器（因为没有`authorize()`回调来限制访问
2. 仅允许拥有访问代币 “01127c36-32fa-4c85-b6da-f720796fe679” 和 “35161a5c-60f0-4809-8b49-1a662247f5b3” 的用户访问应用。

要请求令牌认证，您需要将 HTTP 请求头的 `Authorization` 字段设置为 `token <ACCESS_TOKEN>`。 例子：

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

##### 6. 手机钱包支持

以下示例根据钱包签名对用户帐户进行身份验证。

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

创建并返回一个 web3 合约方法对象，可以链接调用 web3 方法。

#### 语法

```javascript
const methods = party.contract(web3, abi, contract_address)
```

#### 参数

- `web3`: 初始化的 web3 对象
- `abi`: ABI 数组
- `contract_address`: 合约地址

#### 示例

##### 1. 代币余额合约调用

要根据钱包帐户的状态和数据授权用户，需要查询区块链。

这种情况下，需要实例化一个 web3 对象并使用内置的 `contract()` 方法来调用 web3 合约方法

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

或可以使用内置的 `party.abi.erc721` 而不是硬编码 `const abi` ，如下所示：

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

##### 2. 管理员登录

有时你想要构建一个只允许合约所有者登录的管理页面。

除了 erc20 和 erc721 ABI 之外，Privateparty 还有一个适用于合约所有者的 ABI 接口可以调用 [ownable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol) ：

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

常用 ABI 的内置常用模块：

#### 1. erc20

```javascript
// 使用 party.abi.erc20 而不是硬编码 ABI
const party = new Privateparty()
let balance = await party.contract(web3, party.abi.erc20, UNISWAP_ADDRESS).balanceOf(account).call()
```

#### 2. erc721

```javascript
//使用 party.abi.erc721 而不是硬编码 ABI
const party = new Privateparty()
let tokenURI = await party.contract(web3, party.abi.ownable, MFERS_NFT_ADDRESS).tokenURI(tokenId).call()
```

#### 3. ownable （所有者的）

```javascript
// use the party.abi.ownable instead of hardcoding the ownable ABI
const party = new Privateparty()
let owner = await party.contract(web3, party.abi.ownable, mfers).owner().call()
```

### app

使用 `app = express()` 在内部创建的 `app` 对象。

您可以像使用任何 express app 应用实例一样使用 `app` 对象。

#### 示例

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



## 浏览器客户端 

### constructor（构造函数）

#### 语法

```javascript
const party = new Privateparty(config)
```

#### 参数

- `config`：配置
   - `host`: **（可选）** 如果您希望向托管在另一个 domoain 上的openavatar服务器发出跨域请求，请指定主机。
   - `walletconnect`：**（可选）** 指定此字段以支持移动和桌面钱包。 `walletconnect` 属性是 [Walletconnect infuraId 属性](https://github.com/Web3Modal/web3modal/blob/master/docs/providers/walletconnect.md?plain=1#L22) (转到 [Infura]( https://infura.io/) 注册并获取 Infura 项目 ID)。

#### 返回值

- `party`: 实例化的OpenAvatar客户端

#### 示例

##### 1. 基础示例

```javascript
const party = new Privateparty()
```

##### 2. 跨域链接

假设您的openavatar服务器在 https://myprivatepartyserver.com 上运行 - 您可以使用 `host` 属性连接到它。

> **注意**
>
> 您必须在服务器端设置 CORS 支持才能使其正常工作 http://localhost:56503/#/?id=_3-cross-origin-login-support
> 
```javascript
const party = new Privateparty({
  host: "https://myprivatepartyserver.com"
})
```


##### 3. 移动和桌面钱包支持

为了支持手机和桌面钱包，我们需要使用[Walletconnect](https://walletconnect.com/)。 为此，我们需要从 [Infura](https://infura.io) 获取项目 ID 并将其设置为 `walletconnect` 属性。 例子：

```javascript
const party = new Privateparty({
  walletconnect: "27e484dcd9e3efcfd25a83a78777cdf1"   // 使用你自己的 INFURA ID!
})
```

---

### connect()

#### 语法

```javascript
let session = await party.connect(name, payload, options)
```

#### 参数

- `name`：openavatar 角色名称。 自动连接到 openavatar 后端定义的同名端点。
- `payload`：**（可选）** 将传递给 Privateparty 服务端的附加有效负载。 Privateparty 服务端将能够在其授权逻辑中检查“req.body.payload”。
- `options`: **（可选）** 一个对象描述了如何建立连接。 包括以下属性：
   - `fresh`：是否要求用户（重新）连接钱包列表中的钱包，如果仍然在连接，则使用先前连接的钱包
     - 如果设置 `true`，显示列表钱包，并让用户选择一个
     - 如果设置 `false`，跳过钱包选择步骤，使用之前选择的钱包（这是默认设置）
     - 
#### 返回值

- `session`：此连接的已验证和已授权会话对象。
   - `account`：已验证的帐户
   - `expiresIn`：自发布时间 (`iat`) 以来，会话有效时长，以秒为单位。 （默认：60 * 60 * 24 * 30，或 30 天）
   - `jwt`：完整的 JWT 字符串
   - `auth`: **（可选）** 如果需要，由openavatar服务器设置的附加属性。 仅当调用 `party.add()` 时从 `authorize()` 回调返回某些内容时才包括在内。

相同的 `session` 对象将存储在 cookie 中，随后可以通过 `party.session()` 访问

#### 示例

##### 1. 基本链接

假设 Privateparty对象 在后端添加了一个名为“user”的角色：

```javascript
const party = new Privateparty()
party.add("user")
party.app.get("/", party.auth("user"), (req, res) => {
  // public route.
  // no authentication and no authorization
})
party.app.listen(3000)
```

上面代码设置默认端点：

- `GET /privateparty/session`
- `POST /privateparty/connect`
- `POST /privateparty/disconnect`

可以简单地通过指定角色的名称（`"user"`）自动连接到这些端点：


```javascript
// Browser code
const party = new Privateparty()
let session = await party.connect("user")
if (session) document.write("logged in: " + session.account)
```

##### 2. 自定义引擎连接

如果在后端设置了多个引擎，可以通过使用自定义端点初始化 `Privateparty` 对象来连接到自定义端点。

例如，让我们以多个角色（“user”和“admin”）为例：

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

接下来，在浏览器中，我们尝试以admin管理员身份登录：

```javascript
const party = new Privateparty()
let session = await party.connect("admin")
```

The `party.connect("admin")`:

1. automatically discovers the `/admin/connect` endpoint
2. makes a POST request to it
3. the `authorize()` callback in the backend takes care of the authorization for the admin role

##### 3. 使用自定义有效负载进行身份验证

通常，授权逻辑需要的不仅是用户帐户。 例如，用户还可以使用特定的 NFT（`tokenId` 和 `contract`）进行身份验证，在这种情况下，前端将更多数据传递给服务器端 Privateparty 对象。 这是一个例子：

```javascript
const party = new Privateparty()
await party.connect("user", {
  contract: "0x6866ed9a183f491024692971a8f78b048fb6b89b",
  tokenId: "55005454344647406361450320675654878134478584534017520891306338141495783002503"
})
```

服务器可以在引擎中实现一个 `authorize(req, account)` 函数，如下所示：

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

请注意，附加的有效负载包含在属性“req.body.auth”下。

##### 4. 让用户从钱包中选择一个账户

默认情况下，Privateparty 自动使用缺省帐户。 如果允许用户从钱包中选择不同的帐户进行登录。

可以用下例方式连接：

```javascript
const party = new Privateparty()
await party.connect("user", null, { fresh: true })
```

注意:

- 第二个参数是 `null`（没有传递任何有效载荷）
- `connect()` 方法的第三个参数（`options`）是`{fresh: true}`。 这告诉 openavatar 建立一个新的连接，用户可以从钱包帐户中进行选择，而不是选择确认帐户。

当然，您也可以在传递有效负载的同时执行此操作：

```javascript
const party = new Privateparty()
await party.connect(
  "user",
  { contract: "0x6866ed9a183f491024692971a8f78b048fb6b89b", tokenId: "55005454344647406361450320675654878134478584534017520891306338141495783002503" },
  { fresh: true }
)
```

现在第二个参数是`payload`，第三个参数是`options`

---

### session()

`session()` 方法用于获取当前会话。

#### 语法

```javascript
let session = await party.session(name)
```

#### 参数

- `name`：会话的角色名称

#### 返回值

- `session`：指定名称的全局会话对象
   - `account`：经过身份验证的帐户
   - `expiresIn`：自发布时间 (`iat`) 以来，会话有效时长，以秒为单位。 （默认：60 * 60 * 24 * 30，或 30 天）
   - `iat`: 何时发出的会话
   - `auth`: **（可选）** 如果需要，由openavatar服务器设置的附加属性。 仅当调用 `party.add()` 时从 `authorize()` 回调返回某些内容时才包括在内。

#### 示例

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

清除 cookie 并退出所有会话

#### 语法

```javascript
await party.disconnect(name)
```

#### 参数

- `name`：要断开的会话的名称
- 
#### 返回值

- none

---

## Node.js 客户端

Privateparty 对象使用 [JWT](https://en.wikipedia.org/wiki/JSON_Web_Token)，这意味着您可以使用从浏览器创建的相同会话，并在服务器设置或任何其他非浏览器设置中使用它。

以下是 node.js 客户端与浏览器客户端的不同之处：

1. 没有cookie：与 partyconnect 浏览器客户端不同，它会在连接后自动设置cookie，而`partypass` node.js 客户端只是向  openavatar 服务端发出请求以创建JWT 令牌。从此时，如何处理 JWT 令牌由你决定。
2. 直接使用私钥：`partyconnect`浏览器客户端是为浏览器无缝应用而构建的，因此使用注入浏览器的任何钱包。但是 `partypass` node.js 客户端主要应该运行在 node.js 设置中，并且没有注入浏览器钱包。因此，要初始化 `partypass`，您必须传递一个私钥，该私钥将用于对消息进行签名。

基本上，“partyconnect”库主要用于浏览器内使用，而“partypass”是使用您提供的私钥创建 JWT 的最小库。

### 构造函数

#### 语法

```javascript
const pass = new Partypass(config)
```

#### 参数

- `config`
   - `host`：openavatar主机 URL
   - `key`： 十六进制形式的私钥（没有`0x`前缀）
   - 
#### 返回值

- `pass`: 初始化的partypass对象，可以用来创建会话

#### 示例

```javascript
const pass = new Partypass({
  host: "http://localhost:3000",
  key: "cd76c32ffeae94b725b40b1f58ffc793d5b0e96596f8d067f29d385894f16424" // private key
})
```

在大多数情况下，不应硬编码私钥，因此代码实际上可能如下：

```javascript
const pass = new Partypass({
  host: "http://localhost:3000",
  key: process.env.PRIVATE_KEY  // use environment variables
})
```

### create()

使用初始化的 Partypass 对象创建会话

> `await pass.create(name, payload)` 等价于 `await pass.request(await pass.build(name), payload)`
> 
#### 语法

```javascript
const session = await pass.create(name, payload)
```

#### 参数

- `name`：角色的名称。 自动连接到服务端定义的同名端点。
- `payload`：**（可选）** 传递给 Privateparty 服务器对象的附加有效负载。 Privateparty 服务器对象将能够在其授权逻辑中检查“req.body.payload”。

#### 返回值

- `session`：经过身份验证和授权的会话对象。
   - `account`：经过身份验证的帐户
   - `expiresIn`：自发布时间 (`iat`) 以来，会话有效时长，以秒为单位。 （默认：60 * 60 * 24 * 30，或 30 天）
   - `jwt`：完整的 JWT 字符串
   - `auth`: **（可选）** 如果需要，由服务器设置的附加属性。 仅当您在调用 `party.add()` 时从 `authorize()` 回调返回某些内容时才包括在内。

相同的 `session` 对象将存储在 cookie 中，随后可以通过 `party.session()` 访问

#### 示例

假设 Privateparty 服务端对象 添加了一个名为“user”的角色：

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

现在创建一个客户端，向上面的Privateparty服务器对象发出请求，作为`client.js`：

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

运行 “node 客户端”。 获得包含 JWT 的会话对象。


### session()

检查会话是否已过期

#### 语法

```javascript
const session = await pass.session(name)
```

#### 参数

- `name`: 角色的名称

#### 返回值

- `session`: 返回 `name` 角色的会话信息。 如果过期，则返回 `null`。

### build()

构建一个签名请求以发送到 openavatar 服务器。

>  `create()` 调用等效于调用 `build()` + 继续调用 `request()`
> 
#### 语法

```javascript
const req = await pass.build(name)
```

#### 参数

- `name`: 角色名称
- 
#### 返回值

- `req`：准备好的请求对象。 可以发送到相关的openavatar 服务器（即已连接主机）
   - `str`：已签名的 nonce 消息
   - `sig`：签名（个人签名）
   - `url`：请求发送到的端点


### request()

发送使用 `build()` 方法创建的构建请求，以从openavatar服务器获取相应的会话

#### 语法

```javascript
const session = await pass.request(req, payload)
```

#### 参数

- `req`：使用`pass.build()`创建的准备好的请求对象。 可以发送到相关的openavatar服务器（与已连接主机相同的主机）
   - `str`：已签名的 nonce 消息
   - `sig`：签名（个人签名）
   - `url`：将请求发送到的端点
- `payload`: **（可选）** 一个额外的有效载荷发送到服务端。

#### 返回值

- `session`：返回`name`角色创建的会话信息。
   - `account`：经过身份验证的帐户
   - `expiresIn`：自发布时间 (`iat`) 以来，会话有效时长，以秒为单位。 （默认：60 * 60 * 24 * 30，或 30 天）
   - `jwt`：完整的 JWT 字符串
   - `auth`: **（可选）** 如果需要，由openavatar设置的附加属性。 仅当调用 `party.add()` 时从 `authorize()` 回调返回某些内容时才包括在内。