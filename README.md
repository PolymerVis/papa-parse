papa-parse
[![GitHub release](https://img.shields.io/github/release/PolymerVis/papa-parse.svg)](https://github.com/PolymerVis/papa-parse/releases)
[![Published on webcomponents.org](https://img.shields.io/badge/webcomponents.org-published-blue.svg)](https://www.webcomponents.org/element/PolymerVis/papa-parse)
[![styled with prettier](https://img.shields.io/badge/styled_with-prettier-ff69b4.svg)](https://github.com/prettier/prettier)
==========

<!---
```
<custom-element-demo>
  <template>
    <link rel="import" href="../polymer/lib/elements/dom-bind.html">
    <link rel="import" href="../polymer/lib/elements/dom-repeat.html">
    <link rel="import" href="papa-parse.html">
    <link rel="import" href="papa-unparse.html">
    <dom-bind>
      <template is="dom-bind">
        <next-code-block></next-code-block>
      </template>
    </dom-bind>
  </template>
</custom-element-demo>
```
-->
```html
<!-- download and parse into JSON -->
<papa-parse auto
  url="https://rawgit.com/PolymerVis/papa-parse/master/demo/MOCK_DATA.csv"
  rows="{{rows}}"></papa-parse>

<!-- render as a table -->
<h4>Parsed CSV table:</h4>
<table>
  <template is="dom-repeat" items="[[rows]]" as="row">
    <tr>
      <template is="dom-repeat" items="[[row]]">
        <td>[[item]]</td>
      </template>
    </tr>
  </template>
</table>

<!-- unparse JSON into string -->
<papa-unparse header
  json-array="[[rows]]"
  csv-str="{{csvStr}}"></papa-unparse>

<!-- csv string output -->
<h4>Csv string</h4>
<pre>[[csvStr]]</pre>

```

## Installation
```
bower install --save PolymerVis/papa-parse
```

## Documentation and demos
More details @  [webcomponents.org](https://www.webcomponents.org/element/PolymerVis/papa-parse).

## Disclaimers
PolymerVis is a personal project and is NOT in any way affliated with Papaparse, Polymer or Google.

# papa-parse
`papa-parse` is a Polymer 2.0 element to parse CSV files into JSON object(s)
with [Papa parse](http://papaparse.com/).

`papa-parse` can download and parse a csv file via `url`, or from raw csv strings via `raw`, and File object via `file`. If the `auto` flag is set, `papa-parse` will automatically start the job, otherwise a manual call to the function `parse` will be needed.

Parse from URL.
```html
<papa-parse auto url="MOCK_DATA.csv" rows="{{rows}}"></papa-parse>
```

Parse from raw csv String.
```html
<papa-parse auto raw="[[SomeCsvString]]" rows="{{rows}}"></papa-parse>
```

Parse from FileReader
```html
<input id="selectfile" type="file"></input>
<papa-parse auto file="[[file]]" rows="{{rows}}"></papa-parse>
```
```javascript
this.$.selectfile
  .addEventListener('change', function(e) {
    this.file = e.target.files[0];
  });
```

There are 2 possible outputs for `rows` - an *array of array of values*, or an
*array of objects*. An array of array is returned if the `header` flag is not
set, and an array of objects otherwise, where each object is a map comprising
of the column name and its corresponding value for the row
(e.g. {col1: value1, col2: value2}).

### Handling big files
Parsing is synchronous and may block the UI thread (i.e. freeze the screen) if
the csv file is big. Parsing with a web-worker via `worker` flag is recommended
for big file. However, note that `papaparse.min.js` *should not be bundled*
during the build process if web-worker is required as the web-worker needs to
load the script. You will also need to correctly reference the location of the
script via the `script-path` attribute.

You should not use relative path if parsing from `url`, as the web-worker may
load from an incorrect path.

```html
<papa-parse auto header worker stream
  script-path="../../papaparse/papaparse.min.js"
  fields="{{fields}}"
  url="https://some.domain/some.csv"
  on-stream="handleRecord"></papa-parse>
```
```javascript
function handleRecord(e, {data, meta, errors}) {
  console.log(data);
}
```

Streaming via `stream` flag is also recommended for big file (`stream` and
`worker` are not dependent on each other). However, when `stream` flag is set,
`rows` attribute will only return an empty array as none of the records will be
persisted. Instead, `papa-parse` will emit an `stream` event of the form
{{data: Array[]|Object[], meta: Object, errors: Array, parser: Object}} for each
record parsed.

```html
<button onclick="javascript:start()">[[buttonLabel]]</button>

<papa-parse id="parser"
  stream
  url="MOCK_DATA.csv"
  rows="{{rows}}"
  on-stream="handleRecord"></papa-parse>
```
```javascript
function start() {
  app.$.parser.start();
}

function handleRecord(e, {data, meta, errors}) {
  alert(JSON.stringify(data));
  app.$.parser.pause();
}
```

# papa-unparse
`papa-unparse` is a Polymer 2.0 element to convert JSON arrays or objects into
CSV strings with [Papa parse](http://papaparse.com/).

**Basic usage**  
For array of objects, e.g. [{col1: 1, col2: 2}, {col1: 3, col2: 4}]
```html
<papa-unparse header json-array="[[rows]]"></papa-unparse>
```

For array of arrays, e.g. [[1,2], [3,4]]
```html
<papa-unparse json-array="[[rows]]" fields="[[fields]]"></papa-unparse>
```

### Download as CSV file
```html
<papa-unparse header json-array="[[rows]]"></papa-unparse>
<button onclick="javascript:download">Download</button>
```
```javascript
function download() {
  document.querySelector('papa-unparse').downloadCsv('somefile.csv');
}
```

### Other options
```html
<papa-unparse
  header
  quotes
  quoteChar="'"
  delimiter="|"
  newline="\n"
  json-array="[[rows]]"></papa-unparse>
```
