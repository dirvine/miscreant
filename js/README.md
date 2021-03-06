# miscreant.js [![Latest Version][npm-shield]][npm-link] [![Build Status][build-image]][build-link] [![Known Vulnerabilities][snyk-image]][snyk-link] [![MIT licensed][license-image]][license-link] [![Gitter Chat][gitter-image]][gitter-link]

[npm-shield]: https://img.shields.io/npm/v/miscreant.svg
[npm-link]: https://www.npmjs.com/package/miscreant
[build-image]: https://secure.travis-ci.org/miscreant/miscreant.svg?branch=master
[build-link]: http://travis-ci.org/miscreant/miscreant
[snyk-image]: https://snyk.io/test/github/miscreant/miscreant/badge.svg?targetFile=js%2Fpackage.json
[snyk-link]: https://snyk.io/test/github/miscreant/miscreant?targetFile=js%2Fpackage.json
[license-image]: https://img.shields.io/badge/license-MIT-blue.svg
[license-link]: https://github.com/miscreant/miscreant/blob/master/LICENSE.txt
[gitter-image]: https://badges.gitter.im/badge.svg
[gitter-link]: https://gitter.im/miscreant/Lobby

> The best crypto you've never heard of, brought to you by [Phil Rogaway]

JavaScript-compatible TypeScript implementation of **Miscreant**:
Advanced symmetric encryption using the [AES-SIV] ([RFC 5297]) and [CHAIN/STREAM]
constructions, providing easy-to-use (or rather, hard-to-misuse) encryption of
individual messages or message streams.

**AES-SIV** provides [nonce-reuse misuse-resistance] (NRMR): accidentally
reusing a nonce with this construction is not a security catastrophe,
unlike it is with more popular AES encryption modes like [AES-GCM].
With **AES-SIV**, the worst outcome of reusing a nonce is an attacker
can see you've sent the same plaintext twice, as opposed to almost all other
AES modes where it can facilitate [chosen ciphertext attacks] and/or
full plaintext recovery.

For more information, see the [toplevel README.md].

[Phil Rogaway]: https://en.wikipedia.org/wiki/Phillip_Rogaway
[AES-SIV]: https://www.iacr.org/archive/eurocrypt2006/40040377/40040377.pdf
[RFC 5297]: https://tools.ietf.org/html/rfc5297
[CHAIN/STREAM]: http://web.cs.ucdavis.edu/~rogaway/papers/oae.pdf
[nonce-reuse misuse-resistance]: https://www.lvh.io/posts/nonce-misuse-resistance-101.html
[AES-GCM]: https://en.wikipedia.org/wiki/Galois/Counter_Mode
[chosen ciphertext attacks]: https://en.wikipedia.org/wiki/Chosen-ciphertext_attack
[toplevel README.md]: https://github.com/miscreant/miscreant/blob/master/README.md

## Help and Discussion

Have questions? Want to suggest a feature or change?

* [Gitter]: web-based chat about miscreant projects including **miscreant.js**
* [Google Group]: join via web or email ([miscreant-crypto+subscribe@googlegroups.com])

[Gitter]: https://gitter.im/miscreant/Lobby
[Google Group]: https://groups.google.com/forum/#!forum/miscreant-crypto
[miscreant-crypto+subscribe@googlegroups.com]: mailto:miscreant-crypto+subscribe@googlegroups.com?subject=subscribe

## Security Notice

Though this library is written by cryptographic professionals, it has not
undergone a thorough security audit, and cryptographic professionals are still
humans that make mistakes. Use this library at your own risk.

This library contains two implementations of the cryptographic primitives
which underlie its implementation: ones based on the [Web Cryptography API],
(a.k.a. Web Crypto) and a set of pure JavaScript polyfills.

By default only the Web Crypto versions will be used, and an exception raised
if Web Crypto is not available. Users of this library may opt into using the
polyfills in environments where Web Crypto is unavailable, but see the security
notes below and understand the potential risks before doing so.

### Web Crypto Security Notes

The Web Crypto API should provide access to high-quality implementations of
the underlying cryptographic primitive functions used by this library in
most modern browsers, implemented in optimized native code.

On Node.js, you will need a native WebCrypto provider such as
[node-webcrypto-ossl] to utilize native code implementations of the underlying
ciphers instead of the polyfills. However, please see the security warning
on this package before using it.

[node-webcrypto-ossl]: https://github.com/PeculiarVentures/node-webcrypto-ossl

### Polyfill Security Warning

The AES polyfill implementation (off by default, see above) uses table lookups
and is therefore not constant time. This means there's potential that
co-tenant or even remote attackers may be able to measure minute timing
variations and use them to recover AES keys.

If at all possible, use the Web Crypto implementation instead of the polyfills.

[Web Cryptography API]: https://www.w3.org/TR/WebCryptoAPI/

## Installation

Via [npm](https://www.npmjs.com/):

```bash
npm install miscreant
```

Via [Yarn](https://yarnpkg.com/):

```bash
yarn install miscreant
```

Import **miscreant.js** into your project with:

```js
import Miscreant from "miscreant";
```

## API

### Miscreant.importKey()

The **Miscreant.importKey()** method creates a new instance of an **AES-SIV**
encryptor/decryptor.

#### Syntax

```
Miscreant.importKey(keyData, algorithm[, crypto = window.crypto])
```

#### Parameters

* **keyData**: a [Uint8Array] containing the encryption key to use.
  Key must be 32-bytes (for AES-128) or 64-bytes (for AES-256), as
  SIV uses two distinct AES keys to perform its operations.
* **algorithm**: a string describing the algorithm to use. The only algorithm
  presently supported is `"AES-SIV"`.
* **crypto**: a cryptography provider that implements the WebCrypto API's
  [Crypto] interface.

#### Return Value

The **Miscreant.importKey()** method returns a [Promise] that, when fulfilled,
returns a SIV encryptor/decryptor.

#### Exceptions

The **Miscreant.importKey()** method will throw an error if it's attempting to use
the default `window.crypto` provider either doesn't exist (e.g. `window` is
not defined because we're on Node.js) or if that provider does not provide
native implementations of the cryptographic primitives **AES-SIV** is built
on top of.

In these cases, you may choose to use `PolyfillCrypto`, but be aware this may
decrease security.

#### Example

```
// Assuming window.crypto.getRandomValues is available

let keyData = new Uint32Array(32);
window.crypto.getRandomValues(keyData);

let key = await Miscreant.importKey(keyData, "AES-SIV");
```

### seal()

The **seal()** method encrypts a message along with a set of message headers
known as *associated data*.

#### Syntax

```
key.seal(associatedData, plaintext)
```

#### Parameters

* **associatedData**: array of [Uint8Array] values containing data which won't
  be encrypted, but will be *authenticated* along with the message. This is
  useful for including a *nonce* for the message, ensuring that if the same
  message is encrypted twice, the ciphertext will not repeat.
* **plaintext**: a [Uint8Array] of data to be encrypted.

#### Return Value

The **seal()** method returns a [Promise] that, when fulfilled, returns a
[Uint8Array] containing the resulting ciphertext.

#### Example

```
// Assuming window.crypto.getRandomValues is available

let keyData = new Uint8Array(32);
window.crypto.getRandomValues(keyData);

let key = await Miscreant.importKey(keyData, "AES-SIV");

// Encrypt plaintext

let plaintext = new Uint8Array([2,3,5,7,11,13,17,19,23,29]);
let nonce = new Uint8Array(16);
window.crypto.getRandomValues(nonce);

let ciphertext = await key.seal([nonce], plaintext);
```

### open()

The **open()** method decrypts a message which has been encrypted using **AES-SIV**.

#### Syntax

```
key.open(associatedData, ciphertext)
```

#### Parameters

* **associatedData**: array of [Uint8Array] values supplied as associated data
  when the message was originally encrypted.
* **ciphertext**: a [Uint8Array] containing an encrypted message.

#### Return Value

The **open()** method returns a [Promise] that, when fulfilled,
returns a [Uint8Array] containing the decrypted plaintext.

If the message has been tampered with or is otherwise corrupted, the promise
will be rejected with an **IntegrityError**.

#### Example

```
// Assuming window.crypto.getRandomValues is available

let keyData = new Uint8Array(32);
window.crypto.getRandomValues(keyData);

let key = await Miscreant.importKey(keyData, "AES-SIV");

// Encrypt plaintext

let plaintext = new Uint8Array([2,3,5,7,11,13,17,19,23,29]);
let nonce = new Uint8Array(16);
window.crypto.getRandomValues(nonce);

let ciphertext = await key.seal([nonce], plaintext);

// Decrypt ciphertext
var decrypted = await key.open([nonce], ciphertext);
```

[Promise]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
[Uint8Array]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array
[Crypto]: https://developer.mozilla.org/en-US/docs/Web/API/Crypto

## Polyfill Support

**WARNING:** The polyfill implementation is not constant time! Please read
the [Polyfill Security Warning](#polyfill-security-warning) before proceeding!

By default, this library uses a WebCrypto-based implementation of **AES-SIV** and
will throw an exception if WebCrypto is unavailable.

However, this library also contains a `PolyfillCrypto` implementation which
can be passed as the second parameter to `Miscreant.importKey()`. This implementation
uses pure JavaScript, however is not provided by default because there are
security concerns around its implementation.

This implementation should only be used in environments which have no support
for WebCrypto whatsoever. WebCrypto should be available on most modern browsers.
On Node.js, we would suggest you consider [node-webcrypto-ossl] before using
the polyfill implementations, although please see that project's security
warning before using it.

If you have already read the [Polyfill Security Warning](#polyfill-security-warning),
understand the security concerns, and would like to use it anyway, call the
following to obtain a `PolyfillCrypto` instance:

```
Miscreant.getCryptoProvider("polyfill")
```

You can pass it to `Miscreant.importKey()` like so:

```
const polyfillCrypto = Miscreant.getCryptoProvider("polyfill");
const key = Miscreant.importKey(keyData, "AES-SIV", polyfillCrypto);
```

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/miscreant/miscreant

## Copyright

Copyright (c) 2017 [The Miscreant Developers][AUTHORS].

AES polyfill implementation derived from the Go standard library:
Copyright (c) 2012 The Go Authors. All rights reserved.

See [LICENSE.txt] for further details.

[AUTHORS]: https://github.com/miscreant/miscreant/blob/master/AUTHORS.md
[LICENSE.txt]: https://github.com/miscreant/miscreant/blob/master/js/LICENSE.txt
