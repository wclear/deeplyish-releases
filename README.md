# deeplyish releases

The signed list of what [deeplyish.com](https://deeplyish.com) serves to browsers, for the
**deeplyish verifier** browser extension (Chrome and Firefox).

- `releases.json` names the newest 20 releases of the site. For each one it gives the SHA-384 hash of
  every script and stylesheet, in Subresource Integrity form, and the `Content-Security-Policy` the
  site sends with its pages.
- `releases.json.sig` is an Ed25519 signature over the exact bytes of `releases.json`, in base64url.

On every deeplyish page, the extension hashes each script and stylesheet the page loads and reads the
page's policy. It then compares them with a release in this list, and accepts only a list signed by
the key it ships with. A page whose code is in no signed release is flagged as **not verified**.

The list lives here, not on the site, so that the site cannot change what it is checked against.
The history of this repository is a public record of every release. A workflow in the site's
(private) repository adds a release and signs the list on every deploy that changes what browsers
receive. Nobody edits these files by hand.

## Checking it yourself

With the public key from the extension's `config.js` (`TRUSTED_KEYS`, a base64url Ed25519 key) and
Node.js 22:

```js
// node check.mjs <public key>
import { readFileSync } from 'node:fs';
const key = await crypto.subtle.importKey('jwk', { kty: 'OKP', crv: 'Ed25519', x: process.argv[2] }, 'Ed25519', false, ['verify']);
const sig = Buffer.from(readFileSync('releases.json.sig', 'utf8').trim(), 'base64url');
console.log(await crypto.subtle.verify('Ed25519', key, sig, readFileSync('releases.json')) ? 'signed' : 'NOT signed');
```

To compare a file the site serves with the list:

```
curl -s https://deeplyish.com/lib/crypto.js | openssl dgst -sha384 -binary | base64
```
