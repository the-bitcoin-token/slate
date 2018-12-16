
# BitcoinToken

<p><code>BitcoinToken</code> objects can issue a token and send them to another user. It can also return the current balance in tokens.</p>

<aside class="warning">You need to run a <a href="https://github.com/the-bitcoin-token/bitcoin-non-standard-server">non-standard server</a> to use BitcoinToken.</aside>


## constructor

> Create random BitcoinToken

````javascript
const BitcoinToken = Bitcoin.Token
const randomToken = new BitcoinToken()
````

A `BitcoinToken` object contains a `BitcoinDb` object to store data on the blockchain. Recall that each `BitcoinDb` object stores a mnemonic that serves as it's identity. To generate a `BitcoinToken` object from a new randomly generated `BitcoinWallet` object call the constructor without a parameter.

> Create BitcoinToken from a BitcoinDb

````javascript
const db = new Bitcoin.Db()
const token = new Bitcoin.Token(db)
````

You can also create a new `BitcoinToken` object from an existing `BitcoinDb` object by passing it into the constructor.

## fromMnemonic

> Generate a BitcoinToken from a mnemonic

````javascript
const mnemonic = BitcoinWallet.getRandomMnemonic()
const token = BitcoinToken.fromMnemonic(mnemonic)
````

Generates `BitcoinToken` object and initialize the embedded `BitcoinWallet` using the mnemonic.

### Type

<code>static fromMnemonic(mnemonic: string): BitcoinToken</code>


<aside class="notice">The mnemonic and other properties of the wallet contained in a <code>BitcoinToken</code> object can be accessed via <code>db.getWallet().getMnemonic()</code>.</aside>


## getWallet

> Return the wallet

````javascript
const token = new BitcoinToken()
const wallet = token.getWallet()

// wallet.constructor.name === 'BitcoinWallet`
````

Returns the wallet stored in the db contained in the `BitcoinToken` object.

### Type

<code>getWallet(): BitcoinWallet</code>


## getDb

> Return the db

````javascript
const token = new BitcoinToken()
const db = token.getDb()

// db.constructor.name === 'BitcoinDb`
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

### Type

<code>async create(data: Object): Promise&lt;OutputId&gt;</code>


<aside class="warning"><strong>Known issue</strong> The token implementation is currently 100% trustless but not very efficient. In particular, it does not store intermediary results and re-does many computations. This will be fixed in an upcoming release.<br /><br />

If developing with BitcoinToken and the system becomes slow, we recommend to delete both tables in the non-standard server.
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
const tokenAlice = new BitcoinToken()
const tokenId = await tokenAlice.create({ balance: '10' })

const tokenBob = new BitcoinToken()
tokenBob.join(tokenId)

const publicKeyBob = tokenBob.getWallet().getPublicKey()
const outputIds = const tokenAlice.send(1, publicKeyBob)

// outputIds === [{
//   txId: '3404832a17d996dffaa9ac1aea48a2fafc9d855e5fdca5100481d8a562523dfb',
//   outputNumber: 0
// }]
````

Sends tokens to another user. The first parameter is the number of tokens to send and the second is the public key of the recipient. Sending a token might involve a change output, just like the change output used when sending Bitcoin. The function returns a promise that resolves to an array of OutputIds that containing the newly created outputs that reflect the new state.

### Type

<code>
type OutputId = {|<br />
&nbsp;&nbsp;txId: string,<br />
&nbsp;&nbsp;outputNumber: number<br />
|}<br />
<br />
send(<br />
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

Returns the number of tokens owned by the current `BitcoinToken` object.

### Type

<code>getBalance(): Promise&lt;number&gt;</code>
