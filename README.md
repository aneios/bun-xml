# parse-xml

A fast, safe, compliant XML parser for Node.js and browsers.

[![npm version](https://badge.fury.io/js/%40rgrove%2Fparse-xml.svg)](https://badge.fury.io/js/%40rgrove%2Fparse-xml) [![Bundle size](https://badgen.net/bundlephobia/minzip/@rgrove/parse-xml)](https://bundlephobia.com/result?p=@rgrove/parse-xml) [![CI](https://github.com/rgrove/parse-xml/actions/workflows/ci.yml/badge.svg)](https://github.com/rgrove/parse-xml/actions/workflows/ci.yml)

## Links

- [API Docs](https://rgrove.github.io/parse-xml/)
- [GitHub](https://github.com/rgrove/parse-xml)
- [npm](https://www.npmjs.com/package/@rgrove/parse-xml)

## Installation

```
npm install @rgrove/parse-xml
```

Or, if you like living dangerously, you can load [the minified bundle](https://unpkg.com/@rgrove/parse-xml/dist/global.min.js) in a browser via [Unpkg](https://unpkg.com/) and use the `parseXml` global.

## Features

-   Returns a convenient [object tree](#basic-usage) representing an XML document.

-   Works great in Node.js and browsers.

-   Provides [helpful, detailed error messages](#friendly-errors) with context when a document is not well-formed.

-   Mostly conforms to [XML 1.0 (Fifth Edition)](https://www.w3.org/TR/2008/REC-xml-20081126/) as a non-validating parser (see [below](#not-features) for details).

-   Passes all relevant tests in the [XML Conformance Test Suite](https://www.w3.org/XML/Test/).

-   Written in TypeScript and compiled to ES2020 JavaScript for Node.js and ES2017 JavaScript for browsers. The browser build is also optimized for minification.

-   Extremely [fast](#benchmark) and surprisingly [small](https://bundlephobia.com/result?p=@rgrove/parse-xml).

-   Zero dependencies.

## Not Features

This parser currently discards document type declarations (`<!DOCTYPE ... >`) and all their contents, because they're rarely useful and some of their features aren't safe when the XML being parsed comes from an untrusted source.

In addition, the only supported character encoding is UTF-8 because it's not feasible (or useful) to support other character encodings in JavaScript.

## Examples

### Basic Usage

**ESM**

```js
import { parseXml } from '@rgrove/parse-xml';
parseXml('<kittens fuzzy="yes">I like fuzzy kittens.</kittens>');
```

**CommonJS**

```js
const { parseXml } = require('@rgrove/parse-xml');
parseXml('<kittens fuzzy="yes">I like fuzzy kittens.</kittens>');
```

The result is an [`XmlDocument`](https://rgrove.github.io/parse-xml/classes/XmlDocument.html) instance containing the parsed document, with a structure that looks like this (some properties and methods are excluded for clarity; see the [API docs](https://rgrove.github.io/parse-xml/) for details):

```js
{
  type: 'document',
  children: [
    {
      type: 'element',
      name: 'kittens',
      attributes: {
        fuzzy: 'yes'
      },
      children: [
        {
          type: 'text',
          text: 'I like fuzzy kittens.'
        }
      ],
      parent: { ... },
      isRootNode: true
    }
  ]
}
```

All parse-xml objects have `toJSON()` methods that return JSON-serializable objects, so you can easily convert an XML document to JSON:

```js
let json = JSON.stringify(parseXml(xml));
```

### Friendly Errors

When something goes wrong, parse-xml throws an error that tells you exactly what happened and shows you where the problem is so you can fix it.

```js
parseXml('<foo><bar>baz</foo>');
```

**Output**

```
Error: Missing end tag for element bar (line 1, column 14)
  <foo><bar>baz</foo>
               ^
```

In addition to a helpful message, error objects have the following properties:

-   **column** _Number_

    Column where the error occurred (1-based).

-   **excerpt** _String_

    Excerpt from the input string that contains the problem.

-   **line** _Number_

    Line where the error occurred (1-based).

-   **pos** _Number_

    Character position where the error occurred relative to the beginning of the input (0-based).

## Why another XML parser?

There are many XML parsers for Node, and some of them are good. However, most of them suffer from one or more of the following shortcomings:

-   Native dependencies.

-   Loose, non-standard parsing behavior that can lead to unexpected or even unsafe results when given input the author didn't anticipate.

-   Kitchen sink APIs that tightly couple a parser with DOM manipulation functions, a stringifier, or other tooling that isn't directly related to parsing and consuming XML.

-   Stream-based parsing. This is great in the rare case that you need to parse truly enormous documents, but can be a pain to work with when all you want is a node tree.

-   Poor error handling.

-   Too big or too Node-specific to work well in browsers.

parse-xml's goal is to be a small, fast, safe, compliant, non-streaming, non-validating, browser-friendly parser, because I think this is an under-served niche.

I think parse-xml demonstrates that it's not necessary to jettison the spec entirely or to write complex code in order to implement a small, fast XML parser.

Also, it was fun.

## Benchmark

Here's how parse-xml stacks up against two comparable libraries, [libxmljs2](https://github.com/marudor/libxmljs2) (which is based on the native libxml library) and [xmldoc](https://github.com/nfarina/xmldoc) (which is based on [sax-js](https://github.com/isaacs/sax-js)).

While libxmljs2 is faster at parsing medium and large documents, its performance comes at the expense of a large C dependency, no browser support, and a [history of security vulnerabilities](https://www.cvedetails.com/vulnerability-list/vendor_id-1962/product_id-3311/Xmlsoft-Libxml2.html) in the underlying libxml2 library.

```
Node.js v18.9.1 / Darwin arm64
Apple M1 Max

Running "Small document (291 bytes)" suite...
Progress: 100%

  @rgrove/parse-xml 4.0.0:
    189 074 ops/s, ±0.10%   | fastest

  libxmljs2 0.30.1 (native):
    74 006 ops/s, ±0.32%    | 60.86% slower

  xmldoc 1.2.0 (sax-js):
    68 045 ops/s, ±0.08%    | slowest, 64.01% slower

Finished 3 cases!
  Fastest: @rgrove/parse-xml 4.0.0
  Slowest: xmldoc 1.2.0 (sax-js)

Running "Medium document (72081 bytes)" suite...
Progress: 100%

  @rgrove/parse-xml 4.0.0:
    1 066 ops/s, ±0.11%   | 49.12% slower

  libxmljs2 0.30.1 (native):
    2 095 ops/s, ±2.68%   | fastest

  xmldoc 1.2.0 (sax-js):
    459 ops/s, ±0.10%     | slowest, 78.09% slower

Finished 3 cases!
  Fastest: libxmljs2 0.30.1 (native)
  Slowest: xmldoc 1.2.0 (sax-js)

Running "Large document (1162464 bytes)" suite...
Progress: 100%

  @rgrove/parse-xml 4.0.0:
    91 ops/s, ±0.11%    | 51.85% slower

  libxmljs2 0.30.1 (native):
    189 ops/s, ±0.99%   | fastest

  xmldoc 1.2.0 (sax-js):
    39 ops/s, ±0.08%    | slowest, 79.37% slower

Finished 3 cases!
  Fastest: libxmljs2 0.30.1 (native)
  Slowest: xmldoc 1.2.0 (sax-js)
```

See the [parse-xml-benchmark](https://github.com/rgrove/parse-xml-benchmark) repo for instructions on running this benchmark yourself.

## License

[ISC License](LICENSE)
