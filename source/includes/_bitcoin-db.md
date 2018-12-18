# BitcoinDb

<code>BitcoinDb</code> can read, store, and update data on the Bitcoin Cash blockchain.

````javascript
const { Db } = require('bitcointoken')
````

Bitcoin exposes two ways to store data: in [op_return](https://en.bitcoin.it/wiki/OP_RETURN) outputs and in output scripts via [p2sh](https://en.bitcoin.it/wiki/Pay_to_script_hash). Both are supported by BitcoinToken.

* To store flat files that do not need to be updated we recommended you use `db.putReturn` to store data in an op_return output.

* If your application requires data types, updates, or transactions it is recommended to store data in p2sh outputs via `db.put`. The trade off is that your data will only be stored on the blockchain once you update the data, before that only a hash of the data is stored in the blockchain.

<aside class="notice">You need to run a <a href="https://github.com/the-bitcoin-token/bitcoin-non-standard-server">non-standard server</a> to use BitcoinDb.</aside>

<!-- The storage model of BitcoinDb is designed to work as similar to Bitcoin as possible. In Bitcoin, the current state is stored in the unspent transaction output (utxo) set. The state is updated when a Bitcoin transaction is broadcast. The outputs representing the old state are removed from the utxo set and replaced by the outputs of the transaction that represent the new state.

Similarly, BitcoinDb stores data is unspent transaction outputs. When data is updated, the outputs representing the old state are spent and outputs representing the new state are created. The convention is that the i-th output represents the new state of the i-th input being spent. This leads to a natural notion of data ownership where the data is owned by the users(s) that can spend and thereby update the output that contains the data.

The location on the blockchain where data is stored is captured by an <code>OutputId</code> object, which contains two keys <code>txId</code> and <code>outputNumber</code> (that is an <code>OutputId</code> object has type <code>{| txId: string, outputNumber: number |})</code> in <a href="https://flow.org/">flow type</a> notation). -->


## constructor

> Create random BitcoinDb

````javascript
const randomDb = new Db()
````

A `BicoinDb` object stores a `BitcoinWallet` object to pay for storage space on the blockchain. To generate a `BitcoinDb` object from a new randomly generated `BitcoinWallet` object call the constructor without a parameter.

> Create BitcoinDb from a BitcoinWallet

````javascript
const wallet = new Wallet()
const db = new Db(wallet)
````

You can also create a new `BitcoinWallet` object from an existing `BitcoinWallet` object by passing it into the constructor.

## fromMnemonic

> Generate a BitcoinDb from a mnemonic

````javascript
const mnemonic = Wallet.getRandomMnemonic()
const db = Db.fromMnemonic(mnemonic)
````

Generates `BitcoinDb` object and initialize the embedded `BitcoinWallet` using the mnemonic.

### Type

<code>static fromMnemonic(mnemonic: string): BitcoinWallet</code>


<aside class="notice">The mnemonic of a `BitcoinDb` object can be accessed via `db.getWallet().getMnemonic()`.</aside>


## getWallet

> Return the wallet stored in the db

````javascript
const wallet = db.getWallet()

// wallet.constructor.name === 'BitcoinWallet`
````

Returns the `BitcoinWallet` stored in a `BitcoinDb` object.

### Type

<code>getWallet(): BitcoinWallet</code>



## putReturn

> Store a string on the blockchain

````javascript
const data = 'some string'
const outputId = await db.putReturn(data)

// outputId === {
//   txId: '3404832a17d996dffaa9ac1aea48a2fafc9d855e5fdca5100481d8a562523dfb',
//   outputNumber: 0
// }
````

Stores a string in an op_return output of a Bitcoin transaction. The string can have up to 110 characters.

### Type

<code>async putReturn(data: string): OutputId</code>


## put

> Store structured data

````javascript
const outputId = await db.put({
  text: 'Lorem ipsum',
  author: 'Alice'
})

// outputId === {
//   txId: '3404832a17d996dffaa9ac1aea48a2fafc9d855e5fdca5100481d8a562523dfb',
//   outputNumber: 0
// }
````

Stores json on the Bitcoin Cash blockchain.

The data is stored in a Bitcoin output script. Data stored in this way can be updated the same way that coin ownership is updated in Bitcoin: by spending the outputs containing the old data into outputs containing the new data.

> Script encoding the above json

````javascript
// 2-of-2 multisig script
OP_1 
33 0x03a70e0e5f7c300d9685195bbac132873cb5b68e9787aa7d63549cd0de5e4ace10
OP_1
OP_CHECKMULTISIG

// section encoding the data
 4 0x74657874 OP_DROP // text
11 0x4c6f72656d20697073756d OP_DROP // Lorem ipsum
 6 0x617574686f72 OP_DROP // author
13 0x416c69636520616e6420426f62 OP_DROP // Alice and Bob
````

The json data get's encoded into the script on the right. It consists of a 1-of-1 multisig script and a section that encodes the data. The data is stored by pushing a value onto the stack and popping it off immediately thereafter (OP_DROP). 

Data stored in this way comes with a very strong guaranty: only the person in possession of the private key for <code>0x03a70e0e5f7...</code> can spend the output containing the data. We call that user the owner of the data. By default the user to issue the command is the owner, to designate a different owner, pass in that users public key as the second argument. Multiple public keys can be passed in to model co-ownership of data.

> Store structured data and it's owner

````javascript
const publicKey = randomWallet.getPublicKey()
const outputId = await db.put({ key: 'string value'}, [publicKey])
````

The third argument is an amount of satoshi that will be stored in the output containing the data.

> Store a structured data, it's owner, and an amount

````javascript
const publicKey = randomWallet.getPublicKey()
const outputId = await db.put({ key: 'string value'}, [publicKey], 1 * 1e8)
````

### Type

<code>
type OutputId = {|<br />
&nbsp;&nbsp;txId: string,<br />
&nbsp;&nbsp;outputNumber: number<br />
|}<br />
<br />
async put(<br />
&nbsp;&nbsp;data: Object,<br />
&nbsp;&nbsp;owners?: Array&lt;string&gt;,<br />
&nbsp;&nbsp;amount?: number = MIN_NON_DUST_AMOUNT<br />
): Promise&lt;OutputId&gt;<br />
</code>


<aside class="notice">Note: We currently only support objects with string keys and string values. Also the strings much be relatively short (around 300 characters). Please let us know if your application needs to store bigger documents and we will add support for that.</aside>

## get

> Retrieve data from the blockchain


````javascript
const id = BitcoinDb.put({ value: 'a' })
const res = BitcoinDb.get(id)

// res === { 
//   data: { value: 'a' },
//   owners: [ '03223d34686d6f19d20519156a030f7216e5d5bd6daa9442572bbaa446d06c8dfe' ],
//   amount: 2750
// }
````

Retrieves a json object from the Bitcoin Cash blockchain given an <code>OutputId</code> object that specifies a location on the blockchain.

The return value is an object with three keys: <code>data</code> contains the data stored at the <code>OutputId</code>, <code>owners</code> contains the owners string encoded public keys, and <code>amount</code> is the number of satoshis stored in the output.

### Type

<code>
type OutputId = {|<br />
&nbsp;&nbsp;txId: string,<br />
&nbsp;&nbsp;outputNumber: number<br />
|}<br />
<br />
type OutputData = {|<br />
&nbsp;&nbsp;data: string,<br />
&nbsp;&nbsp;owners: Array&lt;string&gt;,<br />
&nbsp;&nbsp;amount: number<br />
|}<br />
<br />
async get(<br />
&nbsp;&nbsp;outputId: OutputId<br />
): Promise&lt;OutputData&gt;
</code>

## update

> Update a piece of data

````javascript
const db = new BitcoinDb()
const outputId1 = await db.put({ value: 'a' })
const outputId2 = await db.update(outputId1, { value: 'b'})

// outputId2 === {
//   txId: '3404832a17d996dffaa9ac1aea48a2fafc9d855e5fdca5100481d8a562523dfb',
//   outputNumber: 0
// }
````


Updates an object stored on the blockchain and returns the location of the new data. The first parameter is the <code>OutputId</code> of the output containing the data that is to be updated, the second parameter is the new data. There are two optional parameters "owners" and "amount" that act just like the optional parameters of <code>db.put</code>. 

> Only the owner can update data

````javascript
const db1 = new BitcoinDb()
const db2 = new BitcoinDb()
const outputId1 = await db1.put({ value: 'a' })

// works bc data at outputId1 is owned by db1
const outputId2 = await db1.update(outputId1, { value: 'b'}, [db2.getWallet().getPublicKey()])

// throws an error bc the data at outputId2 is owned by db2
const outputId3 = await db1.update(outputId2, { value: 'c'})
```

BitcoinDb enforces two properties with respect to updates:

* Only the owner of a piece of data can update it, `db.update()` with throw an error if a user that does not own the data is trying to update it.

* Data can only be updated once, an attempt to update the same data twice will throw an error


> Data can only be updated once

````javascript
const db = new BitcoinDb()
const outputId1 = await db.put({ value: 'a' })

// works 
const outputId2 = await db.update(outputId1, { value: 'b'})

// throws an error bc the data at outputId1 has been updated
const outputId3 = await db.update(outputId1, { value: 'c'})
```


### Type

<code>
type OutputId = {|<br />
&nbsp;&nbsp;txId: string,<br />
&nbsp;&nbsp;outputNumber: number<br />
|}<br />
<br />
async update(<br />
&nbsp;&nbsp;outputId: OutputId,<br />
&nbsp;&nbsp;data: object,<br />
&nbsp;&nbsp;owners?: Array&lt;string&gt;,<br />
&nbsp;&nbsp;amount?: number = MIN_NON_DUST_AMOUNT<br />
): Promise&lt;OutputId&gt;
</code>


## transaction

> Update multiple pieces of data simultaneously

````javascript
const db = new BitcoinDb()
const outputId1 = await db.put({ value: 'a' })
const outputId2 = await db.put({ value: 'b' })
const [outputId3, outputId4] = await db.transaction([
  {
    outputId: outputId1,
    value: 'aa'
  }, {
    outputId: outputId2,
    value: 'bb',
    owners: [<publicKey>]
  }
])
````

You can group multiple updates into a single transaction by calling `db.transaction`. A BitcoinDb transaction satisfies the [ACID](https://en.m.wikipedia.org/wiki/ACID_(computer_science)) properties of traditional database systems.

### Type

<code>
type OutputId = {|<br />
&nbsp;&nbsp;txId: string,<br />
&nbsp;&nbsp;outputNumber: number<br />
|}<br />
<br />
type Update = {|<br />
&nbsp;&nbsp;outputId: OutputId,<br />
&nbsp;&nbsp;data: Object,<br />
&nbsp;&nbsp;owners: Array&lt;string&gt;,<br />
&nbsp;&nbsp;amount?: number<br />
|}<br />
<br />
async transaction(<br />
&nbsp;&nbsp;update: Array&lt;Update&gt;<br />
): Promise&lt;Array&lt;OutputId&gt;&gt; {<br />
</code>