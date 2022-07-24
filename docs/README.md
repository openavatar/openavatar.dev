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
<meta name="viewport" content="width=device-width, initial-scale=1">
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

## 3. Start the app

Now run:

```
node index
```

And open the browser at http://localhost:3000

You will see it working, like this:

![desktoplogin.gif](desktoplogin.gif)

## 4. Mobile wallet support

So far we've just used the default wallet installed on your desktop browser. Now let's see how we can support mobile wallets.

To support mobile wallets we're going to use [Walletconnect](https://walletconnect.com/). You don't have to learn how to use Walletconnect. You simply need to pass an attribute named `walletconnect` when creating the `Privateparty` instance, like this:

```javascript
const party = new Privateparty({
  walletconnect: <Your Infura Key>
})
```

First sign up to [Infura](https://infura.io) and create a project and get the project ID.

Then let's go back to the frontend example above and just pass the `walletconnect` attribute when initializing the `Privateparty` instance:

```html
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
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

One more thing, Walletconnect does not seem to work on localhost so you will need to get an HTTPS URL for testing. You can do this by:

1. Deploying the site publicly to an HTTPS domain (all web hosting providers support this by default)
2. Testing locally with things like [ngrok](https://ngrok.com/) or [localtunnel](https://theboroer.github.io/localtunnel-www/)

Let's try the second approach and test this locally using localtunnel. Here are the steps to follow:

1. Start the privateparty server: `node index`
2. Start a localtunnel that points to the privateparty server: `npx lt --port 3000`

This will give you a public HTTPS url you can test with. Copy and paste it into your browser. You'll see something similar to the following animation (Here I'm demonstrating logging in with two different mobile wallets Metamask mobile and Rainbow wallet):

![mobilelogin.gif](mobilelogin.gif)


## 5. Drop-in react component

Using react? Try the react component: [Partybutton](https://partybutton.papercorp.org/)

[!partybutton.png](partybutton.png)

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
<meta name="viewport" content="width=device-width, initial-scale=1">
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


## 2. ERC20 gated apps

Sometimes you may want users to login with their wallet and store their last snapshot of their balance for a specific ERC20 token.

This may be useful when building airdrop websites or for many other purposes.

### Client

let's first build the frontend. It's the same as last example. Create a file named `index.html` that talks to the server:

```html
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
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
    let balance = await contracts.sartoshi.balanceOf(account).call()
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
<script src="https://unpkg.com/privatepartyjs/dist/privateparty.js"></script>
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
<meta name="viewport" content="width=device-width, initial-scale=1">
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

Now run `node index` and open http://localhost:3000

You will see a login screen where you can enter an NFT collection address and a tokenId.

You can only login if you actually own the NFT.


## 5. Multiple roles

Sometimes you may want to support multiple roles for a single account.

For example, Alice may be a "user" in a web app, but she may also be the "admin" who can have an admin interface. Only those with an "admin" role can access the admin interface, while the rest of the users can only have the "user" role and access the user interface.

Let's try building a minimal app that does that. We will build:

1. Privateparty server
2. User interface
3. Admin inteface

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

### User interface

The user interface is accessible at http://localhost:3000 (route "/") and any account will be able to login.

```html
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<script src="https://unpkg.com/privatepartyjs@0.0.29/dist/privateparty.js"></script>
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

### Admin interface

The admin interface is accessible at http://localhost:3000/admin (route "/admin"). You'll be able to login as "admin". The privateparty server code above allows anyone to login as admin, but you can update the `authorize()` part to authorize only a limited set of addresses to login as admin.

```html
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<script src="https://unpkg.com/privatepartyjs@0.0.29/dist/privateparty.js"></script>
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

## 6. Cross origin login

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
<script src="https://unpkg.com/privatepartyjs@0.0.29/dist/privateparty.js"></script>
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

By default, the privateparty.js client makes requests to the same domain. But you can customize the endpoint by setting the `host` attribute when initializing a Privateparty client.

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

## 7. More examples

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
const party = new Privateparty(config)
```

##### parameters

- `config`: privateparty server configuration
  - `secret`: **(optional)** a string used for signing cookies
    - See https://github.com/expressjs/cookie-parser#cookieparsersecret-options
    - If not specified, it will autogenerate a secret everytime the server restarts using [uuid](https://github.com/uuidjs/uuid).
  - `cors`: **(optional)** If you want to support CORS (cross origin requests) pass this attribute.
    - See https://github.com/expressjs/cors#configuration-options
  - `app`: **(optional)** Inject an existing instantiated express.js app instance
  - `express`: **(optional)** Inject an existin express module

##### return value

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

##### parameters

- `name`: group name (must be unique per group)
- `config`: configuration options for each group
  - `session`: (optional) The GET path to query the current session for this engine. (default: `/privateparty/session/${name}`)
  - `connect`: (optional) The POST path to create a session for this engine (default: `/privateparty/connect/${name}`)
  - `disconnect`: (optional) The POST path to destroy a session for this engine (default: `/privateparty/disconnect/${name}`)
  - `authorize`: a function that takes two or more arguments `req` (The incoming request object passed from express), `account` (The authenticated wallet address), and optionally `contracts` (only when you specify another attribute `contract`, explained below). 
    - To disallow a session based on the request, simply throw an error in the function.
    - To authorize the session, don't throw a function. Additionally, the return value of this function will be automatically set as the `auth` attribute of the session
  - `expire`: (optional) The session duration (how many seconds until a session expires). The default is `1000 * 60 * 60 * 24 * 30` (30 days).
  - `tokens`: (optional) an array of access tokens to allow
  - `contracts`: (optional) a declarative object for defining one or more contracts, which will be initialized and injected in to the `authoirze()` handler


##### return value

none

##### example

```javascript
const Privateparty = require('privateparty')
const party = new Privateparty()
party.add("user", {

  session: "/privateparty/session/user",            // custom path for the session route
  connect: "/privateparty/connect/user",            // custom path for the connect route 
  disconnect: "/privateparty/disconnect/user",      // custom path for the disconnect route

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



##### parameters

- `name`: The authorization group name to use for the route
- `options`: additional information about the protection
  - `redirect`: The web route to redirect to when logged out. For example you can set up an additional route that displays a web page when logged out.
  - `render`: The HTML file path to render when logged out.
  - `json`: The JSON object to return when logged out, and the request was made as an API request (not a website)
  - `walletconnect`: The Walletconnect Infura ID, to support mobile wallets.
  - `fresh`: `true` to display all accounts from the wallet and let the user select, or `false` to use the default account (The default is `false` if not specified. `false` takes two fewer clicks from the user than the `true` option)

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



## Client

### constructor

#### syntax

```javascript
const party = new Privateparty(config)
```

##### parameters

- `config`: configuration
  - `host`: **(optional)** specify the host in case you wish to make a cross-origin request to a privateparty server hosted on another domoain.
  - `walletconnect`: **(optional)** Specify this field to support mobile and desktop wallets. The `walletconnect` attribute is the [Walletconnect infuraId attribute](https://github.com/Web3Modal/web3modal/blob/master/docs/providers/walletconnect.md?plain=1#L22) (Go to [Infura](https://infura.io/) to sign up and get the Infura project ID).

##### return value

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

##### parameters

- `name`: the name of a privateparty role. Automatically connects to the endpoints defined on the privateparty backend with the same name.
- `payload`: **(optional)** additional payload that will be passed to the Privateparty server. The Privateparty server will be able to inspect `req.body.payload` in its authorization logic.
- `options`: **(optional)** an object describes how the connection shall be made. includes the following attributes:
  - `fresh`: whether the login should ask the user to select an account from the wallet, or to use the default account. The default value is `false`
    - if `true`, the login attempt always displays all the account from the wallet and lets the user select an account from the list.
    - if `false`, just uses the default account. Takes 2 fewer steps than `true` (This is the default)

##### return value

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

##### parameters

- `name`: The name of the role for the session

##### return value

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

##### parameters

- `name`: The name of the session to disconnect from

##### return value

- none


---
