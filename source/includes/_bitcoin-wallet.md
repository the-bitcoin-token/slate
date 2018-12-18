# BitcoinWallet

`BitcoinWallet` allows you to receive, store, and send Bitcoin Cash.

```javascript
const { Wallet } = require('bitcointoken')
```

A BitcoinWallet stores a secrete passphrase called menmonic. The mnemonic represents an identity, for example every user in an application would have their own mnemonic. The mnemonic can be used to send Bitcoin to another user, or to generate public addresses that can receive Bitcoin.


## constructor

> Generate a random BitcoinWallet

```javascript
const randomWallet = new Wallet()
```

Generates a new `BitcoinWallet` with a random mnemonic.


<!-- A <code>BitcoinWallet</code> object stores a <a href="https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki">BIP32</a> extended private key. This key can be used to generate a private public key pair and a Bitcoin Cash address. Users can send Bitcoin Cash to the address to fund the wallet. The wallet object can compute the balance of the address as well as send bitcoin from it.
  
  
In addition, a <code>BitcoinWallet</code> can generate 2<sup>32</sup> (>4 billion) child <code>BitcoinWallet</code> objects. The child derivation process is recursive and can generate an infinite number of identities. As the process is deterministic, all identities can be recomputed from a single <code>HdPrivateKey</code>.

The functions of a <code>BitcoinWallet</code> object are listed below. -->


## getRandomMnemonic

> Generate a mnemonic

```javascript
const mnemonic = Wallet.getRandomMnemonic()

// mnemonic === 'rail install size scorpion orchard kingdom vacuum collect pencil element fall enhance media island medal'
```

Generates a new mnemonic from a secure random source.

### Type

<code>static getHdPrivateKey(): string</code>

## fromMnemonic

> Generate a BitcoinWallet from a mnemonic

````javascript
const mnemonic = Wallet.getRandomMnemonic()
const wallet = Wallet.fromMnemonic(mnemonic)
````

Generates `BitcoinWallet` from a the mnemonic.

### Type

<code>static getRandomMnemonic(mnemonic: string): BitcoinWallet</code>

## getMnemonic

> Return the mnemonic

````javascript
const wallet = Wallet.fromMnemonic('rail install size scorpion orchard kingdom vacuum collect pencil element fall enhance media island medal')
const mnemonic = wallet.getMnemonic()

// menominc === 'rail install size scorpion orchard kingdom vacuum collect pencil element fall enhance media island medal'
````

Returns the mnemonic of the wallet

### Type

<code>getMnemonic(): string</code>

## getPrivateKey

> Return the private key

````javascript
const privateKey = wallet.getPrivateKey()
````
Every BitcoinWallet is associated with a Bitcoin account consisting of a private-public keypair and an address. These can be accessed using `getPrivateKey`, `getPublicKey` and `getAddress`.

`getPrivateKey` return the private key stored in the wallet.

### Type
<code>getPrivateKey(): string</code>


## getPublicKey

> Return the public key

````javascript
const publicKey = wallet.getPublicKey()
````

Returns the public key.

### Type
<code>getPublicKey(): string</code>


## getAddress

> Return the address

````javascript
const address = wallet.getAddress()
const CashAddress = wallet.getAddress('cashaddr')
````

Returns the address in <code>legacy</code> format. To return the address in a different format pass in <code>legacy</code>, <code>bitpay</code>, or <code>cashaddr</code> as a parameter.

### Type
<code>
type Format = 'legacy' | 'bitpay' | 'cashaddr'<br />
getAddress(format?: Format = 'legacy'): string
</code>

## getBalance

> Return the current balance

````javascript
const satoshi = await wallet.getBalance()
const bitcoin = satoshi / 1e8
````

Returns the current balance in satoshi. Divide by `1e8` to compute the balance in Bitcoin.


### Type

<code>async getBalance(): number</code>

<aside class="notice">Note that `getBalance` returns the balance of the current account and not the balance of derived accounts.</aside>

## send

> Send Bitcoin

````javascript
const address = randomWallet.getAddress()
const outputId = await wallet.send(15000, address)

// outputId === {
//   txId: '3404832a17d996dffaa9ac1aea48a2fafc9d855e5fdca5100481d8a562523dfb',
//   outputNumber: 0
// }
````

Sends bitcoin. The first parameter is the amount to send in satoshi, the second is the recipients address, and the third (optional) paramater is the change address. If no change address is provided, the address of the wallet is used.

### Type

<code>type OutputId = {|<br />
&nbsp;&nbsp;txId: string,<br />
&nbsp;&nbsp;outputNumber: number<br />
}<br />
<br />
async send(<br />
&nbsp;&nbsp;amount: number,<br />
&nbsp;&nbsp;address: string,<br />
&nbsp;&nbsp;changeAddress: ?string<br />
): Promise&lt;OutputId&gt;<br />
</code>


## transaction

> Send Bitcoin to multiple recipients

````javascript
const address = randomWallet.getAddress()
const data1 = { amount: 2000, address: 'mvQPGnzRT6gMWASZBMg7NcT3vmvsSKSQtf' }
const data2 = { amount: 5000, address: 'n1xqzrckuoohZPj3XoMn1mRgJZ3nkCLSq3' }
const outputIds = await wallet.transaction([data1, data2])

// outputIds === [{
//   txId: '3404832a17d996dffaa9ac1aea48a2fafc9d855e5fdca5100481d8a562523dfb',
//   outputNumber: 0
// }, {
//   txId: '3404832a17d996dffaa9ac1aea48a2fafc9d855e5fdca5100481d8a562523dfb',
//   outputNumber: 1
// }]
````

Use `transaction` to pay multiple parties at once. 

### Type

<code>type OutputId = {|<br />
&nbsp;&nbsp;txId: string,<br />
&nbsp;&nbsp;outputNumber: number<br />
}<br />
<br />
type PkhData = {|<br />
&nbsp;&nbsp;amount: number,<br />
&nbsp;&nbsp;address: string<br />
|}<br />
<br />
async transaction(<br />
&nbsp;&nbsp;outputs: Array&lt;PkhData&gt;,<br />
&nbsp;&nbsp;changeAddress?: string<br />
): Promise&lt;Array&lt;OutputId&gt;&gt;<br />
</code>

## derive

> Derive a child wallet

````javascript
const derivedWallet = wallet.derive(4)
````

A BitcoinWallet can deterministically derive <em>2<sup>32</sup></em> child wallets. To derive a child wallet call `derive` with a number <em>&le; 2<sup>32</sup></em>. The second optional parameter determines if the wallet if hardened or not. Read more about hardened wallets in <a href="https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki">BIP32</a>.

> Derive wallet with path `m/44'/0'/0'`

````javascript
const wallet = wallet.derive('m', false).derive(44, true).derive(0, true).derive(0, true)
````

There exists a standard for how to use derivation paths to generate interoperable wallets (<a href="https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki">BIP44</a>). Two popular derivation paths are

* `m/44'/0'/0'` used eg by Yours.org, Bitcoin.com, or Coinbase.com
* `m/44'/145'/0'` used eg by Bitpay. 

### Type

<code>
derive(<br />
&nbsp;&nbsp;index: ?number = 0,<br />
&nbsp;&nbsp;hardened: ?boolean = false<br />
): BitcoinWallet
</code>

## getPath

> Return the current derivation path

````javascript
const wallet = wallet.derive('m', false).derive(44, true).derive(0, true).derive(0, true)
const path = wallet.getPath()

// path === `m/44'/0'/0'`
````

Returns the current derivation path.

### Type

<code>
getPath(): string
</code>
