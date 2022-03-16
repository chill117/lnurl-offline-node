# lnurl-offline

![Build Status](https://github.com/samotari/lnurl-offline-node/actions/workflows/tests.yml/badge.svg)

Node.js module to support [LNURL](https://github.com/fiatjaf/lnurl-rfc) offline devices. Provides set of helper functions to verify signed payloads and decrypt encrypted payloads generated by LNURL offline devices such as the [Bleskomat ATM](https://www.bleskomat.com/), [bleskomay-diy](https://github.com/samotari/bleskomat-diy), and [LNPoS](https://github.com/arcbtc/LNPoS).

LNURL offline devices can be grouped by whether they use [lnurl-withdraw (LUD-03)](https://github.com/fiatjaf/lnurl-rfc/blob/luds/03.md) or [lnurl-pay (LUD-06)](https://github.com/fiatjaf/lnurl-rfc/blob/luds/06.md):
* __Point-of-sale terminals__ use lnurl-pay with an encrypted payload containing the amount and a random PIN. The PIN and amount are learned by the server by decrypting the payload using a shared secret key with the device. After decryption of the payload, the normal lnurl-pay flow is followed. The PIN is revealed to the customer's mobile wallet app once the associated Lightning invoice has been paid.
* __ATMs__ use lnurl-withdraw with a signature to ensure the integrity of the amount and fiat currency symbol. The server can verify the signature by using a shared secret key with the device. A Lightning payment is made by the server only if the signature can be validated.

* [Installation](#installation)
* [Usage](#usage)
	* [Decrypt XOR Payload](#decrypt-xor-payload)
	* [HMAC-SHA256 Signature Check](#hmac-sha256-signature-check)
* [Tests](#tests)
* [Changelog](#changelog)
* [License](#license)


## Installation

Add to your application via `npm`:
```bash
npm install lnurl-offline
```


## Usage

* [Decrypt XOR Payload](#decrypt-xor-payload)
* [HMAC-SHA256 Signature Check](#hmac-sha256-signature-check)


### Decrypt XOR Payload

Below is an example decrypting a XOR-encrypted payload generated by an LNPoS device:
```js
const { xor } = require('lnurl-offline');
const url = require('url');
const exampleUrl = 'http://localhost:3000/lnurlpos/device-id?p=AQhnxmlzUf9K7AWNqCRW3RbzzGjqckZA';
const parsedUrl = url.parse(exampleUrl, true);
const { query } = parsedUrl;
const { p } = query;
const key = Buffer.from('super-secret-key', 'utf8');
const payload = Buffer.from(p, 'base64');
const { pin, amount } = xor.decrypt(key, payload);
console.log({ pin, amount });// pin = 1234, amount = 21
```


### HMAC-SHA256 Signature Check

Below is an example HMAC-SHA256 signature check for a signed URL generated by a Bleskomat ATM or DIY device:
```js
const { isValidSignedQuery } = require('lnurl-offline');
const url = require('url');
const apiKey = {
	id: '7f6e30c87012dc3b7bf8',
	key: '5375baf547756f91225dbfcc40b18e7ddba478c4cdf6f18a668a8efa33a0e3b3',
	encoding: 'hex',
};
const exampleUrl = 'http://localhost:3000/lnurl?id=7f6e30c87012dc3b7bf8&n=ac0284a9560f9abe760c&s=bb183dcf8b17fd12641722e718fb75209816a43a60238ecb13db1e4748960dbb&t=w&pn=1000&px=1000&pd=';
const { query } = url.parse(exampleUrl, true);
const key = Buffer.from(apiKey.key, apiKey.encoding);
console.log(isValidSignedQuery(query, key) ? 'OK' : 'INVALID SIGNATURE');
```


## Tests

Run automated tests as follows:
```bash
npm test
```


## Changelog

See [CHANGELOG.md](https://github.com/samotari/lnurl-offline-node/blob/master/CHANGELOG.md)


## License

This software is [MIT licensed](https://tldrlegal.com/license/mit-license):
> A short, permissive software license. Basically, you can do whatever you want as long as you include the original copyright and license notice in any copy of the software/source.  There are many variations of this license in use.
