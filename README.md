# url-metadata

Request an http(s) url and scrape its metadata. Many of the metadata fields returned are [Open Graph Protocol (og:)](http://ogp.me/) so far.

Support also added for [JSON-LD](https://moz.com/blog/json-ld-for-beginners).

Under the hood, this package does some post-request processing on top of the [request](https://www.npmjs.com/package/request) module.

If you want a new feature, please open an issue or pull request in [GitHub](https://github.com/laurengarcia/url-metadata).


## Usage

To use in an npm/ Node.js project, install from your CLI:
```
$ npm install url-metadata --save
```

Then in your project file (from example/basic.js):
```javascript
const urlMetadata = require('url-metadata')
urlMetadata('http://bit.ly/2ePIrDy').then(
  function (metadata) { // success handler
    console.log(metadata)
  },
  function (error) { // failure handler
    console.log(error)
  })
```

If you'd like to override the default options (see below), pass in a second argument:
```javascript
const urlMetadata = require('urlMetadata')
urlMetadata('http://bit.ly/2ePIrDy', {fromEmail: 'me@myexample.com'}).then(...)
```

### Options & Defaults
This package's default options are the values below that you may want to override:
```javascript
{
  // custom name for the user agent and email that will make url request:
  userAgent: 'MetadataScraper',
  fromEmail: 'example@example.com',

  // module will follow a maximum of 10 redirects
  maxRedirects: 10,

  // timeout in milliseconds, default below is 10 seconds:
  timeout: 10000,

  // number of characters to truncate description to:
  descriptionLength: 750,

  // force image urls in selected tags to use https,
  // valid for 'image', 'og:image' and 'og:image:secure_url' tags:
  ensureSecureImageRequest: true,

  // object containing key/value pairs used for `source` attribution;
  // defaults to empty object, see usage details below:
  sourceMap: {},

  // custom function to decode special-case encodings;
  // defaults to undefined:
  decode: undefined,

  // custom function to encode the metadata fields before they are returned;
  // defaults to undefined:
  encode: undefined
}
```

#### Option: Source Map
This module introduces and supports a metadata field called `source`. More details about `source` in the `Returns` section below. Example usage can be found in `example/source-map.js`

`sourceMap` is used to override the default `source` attribution behavior, which derives `source` from the web host that the url resolves to.

`sourceMap` is a simple object containing key/value pairs where the key is a YouTube username and the value is the source you'd like to attribute the content to (such as a domain, as in example below).
```javascript
const options = {
  sourceMap: { 'the guardian': 'theguardian.com' }
}
```

If you'd like to extend this functionality beyond YouTube attribution, create an issue or pull request in [GitHub](https://github.com/LevelNewsOrg/url-metadata).

#### Option: Decode
You can supply a custom function to decode the metadata scraped from the url. Example decoding of EUC-JP (Japanese) metadata can be found in `example/decode.js`.

If you pass in an options.decode() function, this module will force the [request](https://www.npmjs.com/package/request) module to return the scraped metadata as a buffer to decode(). This module is not opinionated about what you do in the decode() function, only that it accepts a buffer as its argument and returns a string.

#### Option: Encode
You can supply a custom function to encode the metadata fields before they are returned from this module, see `example/encode.js`:
```javascript
const options = {
  encode: function (value) {
    return encodeURIComponent(value).replace(/['*]/g, escape)
  }
}
```

### Returns
Returns a promise that gets resolved with the following url metadata if the url request response returns successfully. Note that the `url` field returned below will be the last hop in the request chain. So if you passed in a url that was generated by a link shortener, for example, you'll get back the final destination of the link as the `url`.
```javascript
{
    'url': '',
    'canonical': '',
    'title': '',
    'image': '',
    'author': '',
    'description': '',
    'keywords': '',
    'source': '',
    'price': '',
    'priceCurrency': '',
    'availability': '',
    'robots': '',
    'og:url': '',
    'og:locale': '',
    'og:locale:alternate': '',
    'og:title': '',
    'og:type': '',
    'og:description': '',
    'og:determiner': '',
    'og:site_name': '',
    'og:image': '',
    'og:image:secure_url': '',
    'og:image:type': '',
    'og:image:width': '',
    'og:image:height': '',
    'twitter:title': '',
    'twitter:image': '',
    'twitter:image:alt': '',
    'twitter:card': '',
    'twitter:site': '',
    'twitter:site:id': '',
    'twitter:account_id': '',
    'twitter:creator': '',
    'twitter:creator:id': '',
    'twitter:player': '',
    'twitter:player:width': '',
    'twitter:player:height': '',
    'twitter:player:stream': '',
    'jsonld': {}
}
```

Additional fields are also returned if the url has an `og:type` set to `article`. These fields are:
```javascript
{
  'article:published_time'     : '',
  'article:modified_time'      : '',
  'article:expiration_time'    : '',
  'article:author'             : '',
  'article:section'            : '',
  'article:tag'                : '',
  'og:article:published_time'  : '',
  'og:article:modified_time'   : '',
  'og:article:expiration_time' : '',
  'og:article:author'          : '',
  'og:article:section'         : '',
  'og:article:tag'             : ''
}
```

#### The `source` field
This module introduces and supports a metadata field called `source` which may be useful for attributing content hosted elsewhere to an original source.

The default behavior of this module is to derive the `source` field from the url's host (in cases where redirects take place before the last hop, the host is the last hop in the request chain):
```javascript
metadata.set({ source: url.split('://')[1].split('/')[0] })
```

You may be able to override this default behavior with the `sourceMap` option.
