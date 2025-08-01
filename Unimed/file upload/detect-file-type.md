---
state: "[[Rascunhos]]"
---

# Link
https://www.npmjs.com/package/detect-file-type

# Types

jpg, png, gif, webp, flif, cr2, tif, bmp, jxr, psd, zip, epub, xpi, tar, rar, gz, bz2, 7z, dmg, mov, mp4, m4v, m4a, 3g2, 3gp, avi, wav, qcp, mid, mkv, webm, wasm, asf, wmv, wma, mpg, mp3, opus, ogg, ogv, oga, ogm, ogx, spx, flac, ape, wv, amr, pdf, exe, swf, rtf, woff, woff2, eot, ttf, otf, ico, cur, flv, ps, xz, sqlite, nes, dex, crx, elf, cab, deb, ar, rpm, Z, lz, msi, mxf, mts, blend, bpg, jp2, jpx, jpm, mj2, aif, xml, svg, mobi, heic, ktx, dcm, mpc, ics, glb, pcap, html

# Example

```javascript
  var detect = require('detect-file-type');
  detect.fromFile('./image.jpg', function(err, result) {

    if (err) {
      return console.log(err);
    }

    console.log(result); // { ext: 'jpg', mime: 'image/jpeg' }
  });
```

# Check withou saving


[fromBuffer(buffer, callback)](https://github.com/dimapaloskin/detect-file-type#frombufferbuffer-callback)

Detect file type from buffer

- `buffer` - uint8array/Buffer
- `callback`

# Custom

[addCustomFunction(fn)](https://github.com/dimapaloskin/detect-file-type?tab=readme-ov-file#addcustomfunctionfn)

Add custom function which receive buffer and trying to detect file type.

- `fn` - function which receive buffer

This method needed for more complicated cases like html or xml for example. Truly uncomfortable to detect html via signatures because html format has a lot of "magic numbers" in the different places. So you can install [is-html](https://www.npmjs.com/package/is-html) package for example and use its functionality.

```js
const detect = require('detect-file-type');
const isHtml = require('is-html');

detect.addCustomFunction((buffer) => {

  const str = buffer.toString();
  if (isHtml(str)) {
    return {
      ext: 'html',
      mime: 'text/html'
    }
  }

  return false;
});

detect.fromFile('./some.html', (err, result) => {
  
  if (err) {
    return console.log(err);
  }
  
  console.log(result); // { ext: 'html', mime: 'text/html' }
});
```

**Note**: custom function should be pure (without any async operations)


[Signature and creating your own signatures](https://github.com/dimapaloskin/detect-file-type?tab=readme-ov-file#signature-and-creating-your-own-signatures)

Detecting of file type work via signatures. The simplest signature in JSON format looks like:

```json
{
  "type": "jpg",
  "ext": "jpg",
  "mime": "image/jpeg",
  "rules": [
    { "type": "equal", "start": 0, "end": 2, "bytes": "ffd8"  }
  ]
}
```

params:

- `type` - signature type, mostly it's the same as param 'ext'
- `ext` - file extension
- `iana` - optional iana registered mime type - will be added when actual used mime differs from iana, or when the old mime type we used was wrong
- `mime` - mime type of file
- `rules` - list of rules for detecting

More details about param `rules`:

**This param have to be array of objects**

- `type` - a rule type. There are available a few types: `equal`, `notEqual`, `contains`, `notContains`, `or`, `and`, `default`
- `search` - a searching rule, that keeps the offset of the searched bytes in a special id.
- `search_ref` - a reference to a previously performed search, `start` and `end` will be offset by it.

#### More details about: equal, notEqual, contains & notContains.

[](https://github.com/dimapaloskin/detect-[[[[file-type]]]]?tab=readme-ov-file#more-details-about-equal-notequal-contains--notcontains)

- `equal` - here is required field `bytes`. We get a dump of buffer from `start` (equals 0 by default) to `end` (equals buffer.length by default). After that we compare the dump with value in param `bytes`. If values are equal then this rule is correct.
- `notEqual` - here is required field `bytes`. We get a dump of buffer from `start` (equals 0 by default) to `end` (equals buffer.length by default). After that we compare the dump with value in param `bytes`. If values aren't equal then this rule is correct.
- `contains` - here is required field `bytes`. We get a dump of buffer from `start` (equals 0 by default) to `end` (equals buffer.length by default). After that we try to find the sequence from `bytes` in this dump. If the dump contains `bytes` then rules is correct.
- `notContains` - here is required field `bytes`. We get a dump of buffer from `start` (equals 0 by default) to `end` (equals buffer.length by default). After that we try to find the sequence from `bytes` in this dump. If the dump contains `bytes` then rules is incorrect.