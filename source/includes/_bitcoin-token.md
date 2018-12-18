
# BitcoinToken

<code>BitcoinToken</code> can issue, send, and store tokens on the Bitcoin Cash blockchain.

````javascript
const { Token } = require('bitcointoken')
````

It uses <code>BitcoinDb</code> to build and broadcast transactions that encode meta information about token issuances and transfers.

<aside class="notice">You need to run a <a href="https://github.com/the-bitcoin-token/bitcoin-non-standard-server">non-standard server</a> to use BitcoinToken.</aside>


## constructor

> Create random BitcoinToken

````javascript
const { Token } = require('bitcointoken')
const randomToken = new Token()
````

A `BitcoinToken` object contains a `BitcoinDb` object to store data on the blockchain. Recall that each `BitcoinDb` object stores a mnemonic that serves as it's identity. To generate a `BitcoinToken` object from a new randomly generated `BitcoinWallet` object call the constructor without a parameter.

> Create BitcoinToken from a BitcoinDb

````javascript
const { Token, Db } = require('bitcointoken')
const db = new Db()
const token = new Token(db)
````

You can also create a new `BitcoinToken` object from an existing `BitcoinDb` object by passing it into the constructor.

## fromMnemonic

> Generate a BitcoinToken from a mnemonic

````javascript
const mnemonic = Wallet.getRandomMnemonic()
const token = Token.fromMnemonic(mnemonic)
````

Generates `BitcoinToken` object and initialize the embedded `BitcoinWallet` using the mnemonic.

### Type

<code>static fromMnemonic(mnemonic: string): BitcoinToken</code>


<aside class="notice">The mnemonic and other properties of the wallet contained in a <code>BitcoinToken</code> object can be accessed via <code>db.getWallet().getMnemonic()</code>.</aside>


## getWallet

> Return the wallet

````javascript
const wallet = token.getWallet()

// wallet.constructor.name === 'Wallet`
````

Returns the wallet stored in the db contained in the `BitcoinToken` object.

### Type

<code>getWallet(): BitcoinWallet</code>


## getDb

> Return the db

````javascript
const db = token.getDb()

// db.constructor.name === 'Db`
````

Returns the `BitcoinDb` stored in the `BitcoinToken` object.

### Type

<code>getDb(): BitcoinDb</code>



## create

> Issue a token

````javascript
const tokenId = await token.create({
  balance: '10',
  name: 'my-token',
  url: 'www.mytoken.com',
})
````

Issues a new fungible token, similar to ERC20 tokens on Ethereum. The parameter must be an object that can be passes into <code>BitcoinDb.put</code>, that is a json object with string keys and string values. It must have one key <code>balance</code> whose value indicates the initial number of tokens.

The create command calls <code>db.put</code> to store the token meta data in the blockchain. The return value is the id of the output that stores the data.

### Type

<code>async create(data: Object): Promise&lt;OutputId&gt;</code>


<aside class="warning"><strong>Known issue</strong> In order for the token implementation to be trustless it must verify the history of every token when <code>send</code> or <code>getBalance</code> are called. The current implementation is trustless, but it does not store intermediary results and is consequently not efficient. This will be addressed in an upcoming release.<br /><br />

In development we recommend to delete both tables in data base of the non-standard server from time to time.
</aside>


## join

> Connect to an issued token

````javascript
const tokenAlice = new Token()
const tokenBob = new Token()
const tokenId = await tokenAlice.create({ balance: '10' })
tokenBob.join(tokenId)
````

Connects a `BitcoinToken` object a token that has been issued using `token.create`. The balance will be calculated with respect to the token being joined.

### Type
<code>join(tokenId: OutputId): void</code>

## send

> Send a token

````javascript
const tokenAlice = new Token()
const tokenId = await tokenAlice.create({ balance: '10' })

const tokenBob = new Token()
tokenBob.join(tokenId)

const publicKeyBob = tokenBob.getWallet().getPublicKey()
const outputIds = const tokenAlice.send(1, publicKeyBob)

// outputIds === [{
//   txId: '3404832a17d996dffaa9ac1aea48a2fafc9d855e5fdca5100481d8a562523dfb',
//   outputNumber: 0
// }]
````

Sends tokens to another user. The first parameter is the number of tokens to send and the second is the public key of the recipient. Sending a token might involve a change output, just like the change output used when sending Bitcoin. The function returns a promise that resolves to an array of OutputIds that containing the newly created outputs that reflect the new state.

<code>token.send</code> calls <code>db.transaction</code> to broadcast a transaction that stores meta data about associated with the token transfer.

### Type

<code>
type OutputId = {|<br />
&nbsp;&nbsp;txId: string,<br />
&nbsp;&nbsp;outputNumber: number<br />
|}<br />
<br />
async send(<br />
&nbsp;&nbsp;amount: number,<br />
&nbsp;&nbsp;publicKey: string<br />
): Promise&lt;Array&lt;OutputId&gt;&gt;<br />
</code>


## getBalance

> Return the balance in tokens

````javascript
const token = new BitcoinToken()
const tokenId = await token.create({
  balance: 10
})

const balance1 = await token.getBalance()
// balance1 === 10

const randomPublicKey  = new BitcoinWallet().getPublicKey()
await token.send(1, randomPublicKey)

const balance2 = await token.getBalance()
// balance2 === 9
````

Returns the number of tokens owned by the current `BitcoinToken` object. The balance will be computed with respect to the token that was created by the object, or with respect to the token that was joined. <code>token.getBalance()</code> will throw an error if the <code>token</code> object has not created or joined a token.

### Type

<code>async getBalance(): Promise&lt;number&gt;</code>

# BitcoinSource

<code>BitcoinSource</code> is a readable Bitcoin implementation in modern Javascript.

````javascript
const { Source } = require('bitcointoken')
````

It is a fork of Bitcore and shares it's [api](https://bitcore.io/api/). You can find the source code [here](https://github.com/the-bitcoin-token/BitcoinSource)