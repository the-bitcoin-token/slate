# Install and run

You need <a href="https://docs.npmjs.com/downloading-and-installing-node-js-and-npm">node.js and npm</a> installed.

## Install

### Create a BitcoinToken project

> Create a BitcoinToken project

````terminal
npm i -y
npm i bitcointoken
````

Create a project by running <code>npm i -y</code> and install BitcoinToken using <code>npm i bitcointoken</code>.

If you only want to use BitcoinWallet you are done. If you want to use BitcoinDb or BitcoinToken you need to install and run a non-standard server.


### Install and run the non-standard server

> Install and run the non-standard server

````terminal
git clone https://github.com/BitcoinDB/bitcoin-non-standard-server.git
cd bitcoin-non-standard-server/
docker-compose up
````

We recommend to use <a href="https://www.docker.com/">Docker</a> and <a href="https://docs.docker.com/compose/">Docker Compose</a> to build and run the server using the command on the right. If you do not use docker you can find instructions in the <a href="https://github.com/the-bitcoin-token/bitcoin-non-standard-server">Github repo</a>.

## Run in Node

> File index.js

````javascript
const Bitcoin = require('bitcointoken')

const wallet = new Bitcoin.Wallet()
console.log(`address: ${wallet.getAddress()}`)
````

After installing BitcoinToken, create a file <code>index.js</code> with the content shown on the right. Run the code using

`node --experimental-repl-await index.js`

The output should be similar to

<code>address: mwZd1bgRYhxY4JLyx1sdEGBYFZnXVDvgmp</code>

<!--
## Run in the browser

> File index.js

````javascript
const Bitcoin = require('bitcointoken')

const wallet = new Bitcoin.Wallet()
document.body.innerHTML = `address: ${wallet.getAddress()}`
````

> File index.html

````html
<html>
  <body>
    <script src="./index.js"></script>
  </body>
</html>
````

Install <a href="https://parceljs.org">Parcel</a> using `npm install -g parcel-bundler`.

Create a files `index.js` and `index.html` in an empty folder.

Start the server using `parcel index.html`. The website will be rendered at <a href="http://localhost:1234/">http://localhost:1234/</a>.
-->

## Run in the browser

> File index.js

````javascript
const Bitcoin = require('bitcointoken')

const wallet = new Bitcoin.Wallet()
document.body.innerHTML = `address: ${wallet.getAddress()}`
````

> File index.html

````html
<html>
  <body>
    <script src="./bundle.js"></script>
  </body>
</html>
````

After installing BitcoinToken, create files <code>index.js</code> and <code>index.html</code> as shown.

Install and run browserify

<code>
npm install -g browserify<br />
browserify index.js > bundle.js
</code>

Open <code>index.html</code> in a web browser


## Troubleshooting

If you have a problem that is not discussed here, please let us know in the <a href="https://t.me/joinchat/FMrjOUWRuUkNuIt7zJL8tg">Telegram group</a>.

### "Communication error: Service unavailable"

The problem is most likely that you are not running the non-standard server.

### "Error: Insufficient balance in address ..."

> File generate-mnemonic.js

````javascript
const Bitcoin = require('bitcointoken')
console.log(Bitcoin.Wallet.getRandomMnemonic())
````

You have to fund your wallet. First check that the same wallet is generated every time you run your code: Run your code multiple times and check if the  same address is logged every time.


If so you can fund that address using a [testnet faucet](https://coinfaucet.eu/en/bch-testnet/).

Otherwise you need to initialize your object from a mnemonic. To generate a mnemonic, create the file <code>generate-mnemonic.js</code> and run it using <code> node --experimental-repl-await generate-mnemonic.js</code>. Now you can generate your object using the logged mnemonic using <code>.fromMnemonic()</code> and fund it using the [testnet faucet](https://coinfaucet.eu/en/bch-testnet/)