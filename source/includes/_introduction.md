
# Introduction

```javascript
const Bitcoin = require('bitcointoken')

// Bitcoin === { Wallet, Db, Token, Source }
```

The BitcoinToken library consists of four classes

 * **BitcoinWallet** send, store, and receive Bitcoin Cash
 * **BitcoinDb** store data on the Bitcoin Cash blockchain
 * **BitcoinToken** issue tokens on Bitcoin Cash
 * **BitcoinSource** a JS implementation of Bitcoin Cash

The classes can be used independently from one another, but are designed to work well together.