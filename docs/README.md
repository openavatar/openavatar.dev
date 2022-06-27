# Introduction

## What is Privateparty?

Privateparty is an open source framework that makes it super easy to build blockchain authenticated web apps.

You can build web apps with very sophisticated wallet authentication/authorization logic, with just a couple of lines of code.


![ppclient.png](ppclient.png)

The result:

![login.gif](login.gif)

1. **Authentication:** Login with a blockchain wallet signature
2. **Authorization:** Authorize access to your pages based on blockchain state or offchain query
3. **Simple:** Just one line of JavaScript code to set up. No convoluted steps.
1. **Open source:** It's a 100% open source framework with NO 3rd party api.


## What can you build?

Privateparty simply replaces traditional authentication methods with cryptography and in some cases blockchain state.

This means you can build ANYTHING you can imagine. You're simply replacing traditional user DB with Privateparty.

1. **Simple authentication:** Web apps that simply need to know who is currently signed in (allowing everyone to login with a wallet)
2. **Invite-only apps:** Only allow someone on the address list to login
3. **NFT based authorization:** Allow login based on onchain NFT ownership
4. **ERC20 based authorization:** Allow login based on ERC20 token balance
5. **Airdrops:** you can implement an airdrop by only authorizing addresses that have ever interacted with a certain contract.
6. **Many more:** Basically you can implement authorization based on ANY blockchain query.

## Community and Support

Ask questions or share feedback here:

1. Twitter: https://twitter.com/skogard
2. Discord: https://discord.gg/BZtp5F6QQM
3. GitHub: https://github.com/privatepart

---


# Quickstart

There are two things to do:

1. **Set up a backend:** The `privateparty` module lets you easily set up an [express.js](https://expressjs.com/) server protected by blockchain wallet signatures.
2. **Connect to the backend:** Once the backend is set up, you can connect to it from the browser using the `privatepartyjs` library.

## 1. Server

First install the dependencies

```
npm install privateparty
```

Now create a file named `index.js` and write initialization logic:

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

Now let's create a file named `index.html` that talks to the server:

```html
<html>
<head>
<script src="https://cdnjs.cloudflare.com/ajax/libs/web3/1.7.4/web3.min.js"></script>
<script src="https://unpkg.com/privatepartyjs/dist/privateparty.js"></script>
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
const web3 = new Web3(window.ethereum)
const party = new Privateparty({ web3 })
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
      await party.disconnect()      // if logged in, log out
    } else {
      await party.connect()         // if logged out, log in
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

## 3. Start the app

Now run:

```
node index
```

And open the browser at http://localhost:3000

---

# Examples

## 1. Invite only apps

Only allow certain addresses to login.

### Server

First install the dependencies

```
npm install privateparty
```

Now create a file named `index.js` and write initialization logic:

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

### Client

Now let's create a file named `index.html` that talks to the server:

```html
<html>
<head>
<script src="https://cdnjs.cloudflare.com/ajax/libs/web3/1.7.4/web3.min.js"></script>
<script src="https://unpkg.com/privatepartyjs/dist/privateparty.js"></script>
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
const web3 = new Web3(window.ethereum)
const party = new Privateparty({ web3 })
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


## 2. ERC20 gated apps

Sometimes you may want users to login with their wallet and store their last snapshot of their balance for a specific ERC20 token.

This may be useful when building airdrop websites or for many other purposes.

### Client

let's first build the frontend. It's the same as last example. Create a file named `index.html` that talks to the server:

```html
<html>
<head>
<script src="https://cdnjs.cloudflare.com/ajax/libs/web3/1.7.4/web3.min.js"></script>
<script src="https://unpkg.com/privatepartyjs/dist/privateparty.js"></script>
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
const web3 = new Web3(window.ethereum)
const party = new Privateparty({ web3 })
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

For this example we will:

1. Allow anyone to log in.
2. Query the blockchain for their UNISWAP token ($UNI) balance, and attach it to their session via cookies.

Since we will be querying the blockchain, we will need to use JSON-RPC endpoints.

#### 1. The hard way

First install the dependencies

```
npm install privateparty @alch/alchemy-web3
```

Now create a file named `index.js` and write initialization logic:

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


Since the `authorize()` function does not throw any error but only returns the balance object, this app will allow anyone to login, but use the balance information to identify users (for example, the holders will have a non-zero balance whereas non holders will have the balance of 0)


#### 2. The easy way

Because ERC20 and ERC721 are frequently used standards, Privateparty provides a built-in ABI you can access under `party.abi.erc20` and `party.abi.erc721` respectively.

The exact same code above can be re-written as follows:

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


## 3. NFT gated apps

Using the same principle, we can implement logins authorized by NFT ownership. For example, often you may want to only allow people to login when they own at least 1 (or more) NFTs from a collection.

You can use this feature to implement token gated communities and websites.

### Client

```javascript
///////////////////////////////////////////////////////////////////////////////////////////
//
//  When a user logs in, take a snapshot of an ERC20 token balance and store it in cookie
//
///////////////////////////////////////////////////////////////////////////////////////////
const Privateparty = require('privateparty')
const { createAlchemyWeb3 } = require("@alch/alchemy-web3");
const web3 = createAlchemyWeb3(<YOUR JSON-RPC ENDPOINT URL>)
const party = new Privateparty(web3)
party.add("user", {
  authorize: async (req, account) => {
    console.log("account", account)
    // take a snapshot of ERC721 NFT balance (End of Sartoshi)
    // ONLY allow login if the account holds AT LEAST 1
    const end_of_sartoshi = "0xf7d134224a66c6a4ddeb7dee714a280b99044805"
    let balance = await party.contract(web3, party.abi.erc721, end_of_sartoshi).balanceOf(account).call()
    console.log("balance", balance)
    if (balance > 0) {
      return { balance: balance }
    } else {
      // If the balance is 0, don't allow login
      throw new Error("must own at least one 'end of sartoshi'")
    }

  }
})
party.app.get("/", party.auth("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```

### Server

```html
<html>
<head>
<script src="https://cdnjs.cloudflare.com/ajax/libs/web3/1.7.4/web3.min.js"></script>
<script src="https://unpkg.com/privatepartyjs/dist/privateparty.js"></script>
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
const web3 = new Web3(window.ethereum)
const party = new Privateparty({ web3 })
const render = async () => {
  let session = await party.session("user")
  // if logged in (session exists), it's a logout button. if logged out, it's a login button.
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

## 4. Login with NFT

![loginwithnft.gif](loginwithnft.gif)

Sometimes you may want to literally "login with NFTs", and set the NFT image URL in the cookie directly.

That way, once a user logs into the app, the app can use the image throughout the app session.

But, how would the Privateparty server know which exact NFT you would like to sign in with?

To send additional payload to the server, you can simply pass additional attributes when calling the `connect()` method:


### Client

let's first create a file named `index.html` that talks to the server:

```html
<html>
<head>
<script src="https://cdnjs.cloudflare.com/ajax/libs/web3/1.7.4/web3.min.js"></script>
<script src="https://unpkg.com/privatepartyjs/dist/privateparty.js"></script>
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
const web3 = new Web3(window.ethereum)
const party = new Privateparty({ web3 })
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
    })         // if logged out, log in
    console.log("connection", connection)
  }
  await render()
})
render()
</script>
</body>
</html>
```

Note that the `await party.connect("user")` line is now passing an object with the attributes `collection` and `tokenId`:

```javascript
let connection = await party.connect("user", {
  collection: document.querySelector("#collection").value,
  tokenId: document.querySelector("#id").value
})
```

For example, the input value may look like this:

```json
{
  collection: "0x6866ed9a183f491024692971a8f78b048fb6b89b",
  tokenId: "55005454344647406361450320675654878134478584534017520891306338141495783002503"
}
```

This object will be passed to the Privateparty server `authorize()` handler as `req.body.payload`, which will be explained below:


### Server

First install the dependencies:

```
npm install privateparty @alch/alchemy-web3
```

Then, create a file named `index.js` and write initialization logic:

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
const party = new Privateparty(web3)
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

Now run `node index` and open http://localhost:3000

You will see a login screen where you can enter an NFT collection address and a tokenId.

You can only login if you actually own the NFT.


## 5. More examples

Check out the [demo folder](https://github.com/privatepart/privateparty/tree/main/demo) on GitHub for more examples.

---

# Install

You can implement a Privateparty web app with 2 libraries (server-side and client-side) that talk to each other:

1. `privateparty`: The server-side module
2. `privatepartyjs`: The client-side library

## 1. Client

Include in your frontend web app:

```html
<script src="https://unpkg.com/privatepartyjs/dist/privateparty.js"></script>
```

Then initialize with:

```javascript
const party = new Privateparty(config)
```

## 2. Server

To install:

```
npm install privateparty
```

Then, use the module in your app like this:

```javascript
const Privateparty = require('privateparty')
const party = new Privateparty()
```

---

# API

## Server

For the backend, you need to use the package `privateparty`. Simply instantiate a new `Privateparty` instance and it should give you everything you need to build a wallet protected web app backend.

### constructor

#### syntax

```javascript
const party = new Privateparty()
```

##### parameters

none

##### return value

- `party`: The initialized privateparty instance, which contains the following attributes:
  - `app`: an "app" instance created internally by calling `const app = express()`
  - `express`: the express module
  - `auth`: authentication & authorization function
  - `add`: a function to add authorization groups


### add()

Add a group to the party

#### syntax

```javascript
await party.add(name, config)
```

##### parameters

- `name`: group name (must be unique per group)
- `config`: configuration options for each group
  - `session`: The GET path to query the current session for this engine. (default: `/session`)
  - `connect`: The POST path to create a session for this engine (default: `/connect`)
  - `disconnect`: The POST path to destroy a session for this engine (default: `/disconnect`)
  - `authorize`: a function that takes two arguments `req` (The incoming request object passed from express) and `account` (The authenticated wallet address). 
    - To disallow a session based on the request, simply throw an error in the function.
    - To authorize the session, don't throw a function. Additionally, the return value of this function will be automatically set as the `auth` attribute of the session
  - `expire`: (optional) The session duration (how many seconds until a session expires). The default is `1000 * 60 * 60 * 24 * 30` (30 days).


##### return value

none


### auth()

The authorization middleware you can add to any route.

To add authorization logic to any route, you need to:

1. First define an authrorization group and its behavior through the `add()` method
2. Then make use of the group by calling `auth(name)`

#### syntax

```javascript
party.app.get(route1, auth(name), (req, res) => {
  ...
})
party.app.post(route2, auth(name), (req, res) => {
  ...
})
```

##### parameters

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
    - `POST /connect`
    - `POST /disconnect`
    - `GET /session`
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
  session: "/session",
  connect: "/connect",
  disconnect: "/disconnect",
  authorize: async (req, account) => {
    if (!allowed.includes(account) {
      throw new Error("not allowed")
    }
  }
})
party.app.get("/", auth("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```

Note that we are:

1. adding a group named `"user"`
2. and then using the group in the `party.app.get("/", auth("user", (req, res) => { . . . })` handler.


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
  session: "/session",
    connect: "/connect",
    disconnect: "/disconnect",
  },
})
party.add("admin", {
  session: "/admin/session",
  connect: "/admin/connect",
  disconnect: "/admin/disconnect",
  authorize: async (req, account) => {
    if (ADMINS.includes(account)) {
      return { admin: true }
    } else {
      throw new Error("not an admin")
    } 
  }
})
party.app.get("/", auth("user"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
party.app.get("/admin", auth("admin"), (req, res) => {
  console.log("session", req.session)
  res.sendFile(process.cwd() + "/index.html")
})
party.app.listen(3000)
```

Note that we have two `GET` route handlers here:

1. `GET /`: The normal route for normal users. Because we're using `auth("user")`, it will use the `user` group.
2. `GET /admin`: The page where the admins can login. Because we're using `auth("admin")`, it will use the `admin` group.

### contract()

Creates and returns a web3 contract methods object, which can be chained to call web3 methods.

#### syntax

```javascript
const methods = party.contract(web3, abi, contract_address)
```

##### parameters

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
  session: "/admin/session",
  connect: "/admin/connect",
  disconnect: "/admin/disconnect",
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



## Client

### constructor

#### syntax

```javascript
const party = new Privateparty(config)
```

##### parameters

- `config`: configuration
  - `web3`: an initialized web3 object

##### return value

- `party`: An instantiated privateparty client

#### examples

```javascript
const web3 = new Web3(window.ethereum)
const party = new Privateparty({ web3: web3 })
```

---

### connect()

#### syntax

```javascript
let session = await party.connect(name, payload)
```

##### parameters

- `name`: the name of a privateparty role. Automatically connects to the endpoints defined on the privateparty backend with the same name.
- `payload`: **(optional)** additional payload that will be passed to the Privateparty server. The Privateparty server will be able to inspect `req.body.payload` in its authorization logic.

##### return value

- `session`: The authenticated and authorized session object for this connection.
  - `account`: the authenticated account
  - `auth`: additional attributes set by the privateparty server if needed.

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

- `GET /session`
- `POST /connect`
- `POST /disconnect`

We can automatically connect to those endpoints simply by specifying the name of the role (`"user"`):


```javascript
// Browser code
const web3 = new Web3(window.ethereum)
const party = new Privateparty({ web3: web3 })
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
  session: "/admin/session",
  connect: "/admin/connect",
  disconnect: "/admin/disconnect",
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
const web3 = new Web3(window.ethereum)
const party = new Privateparty({ web3 })
let session = await party.connect("admin")
```

The `party.connect("admin")`:

1. automatically discovers the `/admin/connect` endpoint
2. makes a POST request to it
3. the `authorize()` callback in the backend takes care of the authorization for the admin role

##### 3. authenticate with custom payload

Often, the authorization logic may require more than just the user account. For example, the user may authenticate with a specific NFT (`tokenId` and `contract`), in which case the frontend needs to pass more data to the Privateparty server. Here's an example:

```javascript
const web3 = new Web3(window.ethereum)
const party = new Privateparty({ web3 })
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
const party = new Privateparty(web3)
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


---

### session()

The `session()` method is used to get the current session.

#### syntax

```javascript
let session = await party.session(name)
```

##### parameters

- `name`: The name of the role for the session

##### return value

- `session`: the global session object for the specified name


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

##### parameters

- `name`: The name of the session to disconnect from

##### return value

- none


---
