# Encoding

---

The **encoding layer** defines the mapping between [the document layer](document.md), and its byte string encoding.

Core API currently defines JSON and HTML encoding schemes.

Servers implementations are free to support and respond with either of these schemes. Server implementations MAY choose to use the client `Accept` header to determine which of the two representations to respond with.

---

## JSON encoding

A document is represented using standard JSON, with the additional restriction that the object keys "\_type" and "\_meta" are reserved.

Any object with the key "\_type" may be considered as one of the Core API primitives described below. The keys "\_type" and "\_meta" are otherwise removed from the normal parsing flow.

The top level element in a Core API response MUST either be a Document or an Error.

The RECOMMENDED media type for this scheme is: `application/vnd.coreapi+json`, but `application/json` MAY also be used.

An example JSON encoded document is demonstrated below.

    {
        "_type": "document",
        "_meta": {
            "url": "/",
            "title": "Notes"
        },
        "notes": [
            {
                "_type": "document",
                "_meta": {
                    "url": "/1de153fe-6747-41d3-bc0e-d9d7d87e448a",
                    "title": "Note"
                },
                "complete": false,
                "description": "Email venue about conference dates",
                "delete": {
                    "_type": "link",
                    "trans": "delete"
                },
                "edit": {
                    "_type": "link",
                    "trans": "update",
                    "fields": [
                        "description",
                        "complete"
                    ]
                }
            }
        ],
        "add_note": {
            "_type": "link",
            "trans": "action",
            "fields": [
                {
                    "name": "description",
                    "required": true
                }
            ]
        }
    }

#### Escaping reserved keys during encoding

When any object key matching the regular expression `/[\_]+(type|meta)/` is encountered, it MUST be escaped by pre-pending an additional underscore character.

#### Unescaping reserved keys during decoding

When any object key matching the regular expression `/_[\_]+(type|meta)/` is encountered, it MUST be unescaped by removing the leading underscore character.

#### Handling unexpected types

A number of the Object structures described below indicate a required type for an element. When the value type is not as expected, the value SHOULD be ignored, and the indicated default value used instead.

---

### Document

**The Document primitive is represented using an object which includes a key-value pair of "_type": "document".**

* Documents MAY include a "\_meta" key. The value of this SHOULD be an object. If omitted, the default value is treated as an empty Object.

* The "\_meta" object MAY include keys named "url" and "title".

* Any other keys occurring in the Document "\_meta" section SHOULD be ignored by clients.

* The value of the "url" field SHOULD be a string. This value is treated as relative to the URL of any parent containing document. When a document is loaded from the network then the URL of the top level document is treated as relative to the URL that the content was fetched from. If omitted the default value is treated as the empty string, relative to the URL of the parent containing document or network URL, as above.

* The value of the "title" field SHOULD be a string. If omitted, the default value is treated as the empty string.

* All other key-value pairs are treated as the document content.

### Link

**The Link primitive is represented using an object which includes a key-value pair of "_type": "link".**

* Links MAY include keys named "url", "trans", and "fields".

* Any other keys occurring in an Link SHOULD be ignored by clients.

* The value of the "url" field SHOULD be a string, and is treated as relative to the URL of the parent containing document. If omitted the default value is treated as the empty string, relative to the URL of the parent containing document.

* The value of the "trans" field SHOULD be a string, and SHOULD be one of the valid transition types. These are "follow", "action", "create", "update" and "delete". If omitted, the link transition type defaults to "follow".

* The value of the "fields" field SHOULD be a list. Each element of the list SHOULD either be a string or an object. If omitted the default value is treated as the empty list.

* A string item in the "fields" list is to be interpreted as the name of an optional field.

* An object item in the "fields" list SHOULD contain a key "name". The value of this SHOULD be a string. The object MAY include a key "required". The value of this SHOULD be a boolean.

* Links SHOULD only be contained by parent a Document or Object. Any Link occurring in an Array SHOULD be ignored. This constraint ensures that Links are always named items, with the key indicating the link name.

### Error

**The Error primitive is represented using an object which includes a key-value pair of "_type": "error".**

* Errors MAY include a single key named "message". The value of this SHOULD be a list of strings. If omitted the default value is treated as the empty list.

* Any other keys occurring in an Error SHOULD be ignored by clients.

* Errors SHOULD only occur as the top level element. Any Error occurring as a child element inside a Document, Object or Array SHOULD be ignored.

### Data primitives

**The standard JSON primitives are represented as usual.**

* These are Object, Array, String, Number, `true`, `false` and `null`.

* Any Object with a "\_type" key that is not "document", "link" or "error" indicates an unknown type. The "\_type" key and any "\_meta" key are to be removed from the normal parse flow, and the element is to be treated by the client as a standard Object.

---

### Canonical style

The following canonical style indicates a set of guidelines that server implementations or client libraries MAY choose to follow, in order to generate a consistent output format for the JSON encoding.

#### Key ordering

Clients MAY choose to order any Object or Document keys in their output, as follows.

* "\_type"
* "\_meta" (With "url" occurring before "title" in the child Object.)
* All keys with a value *that is not* a Link, ordered alphabetically.
* All keys with a value *this is* a Link, ordered alphabetically.

#### Omitting default values

Clients MAY choose to omit any values that are the default when encoding a document.

For example, a link with a transition type of `"follow"` will typically be encoded without including the `"trans": "follow"` key-value pair.

#### Using relative links

Clients MAY take advantage of the relative URL transformations made when parsing Documents and Links, in order to encode minimal URL outputs.

* The top level Document MUST include its URL value, without transformation.
* If a child Document or Link has the same scheme, host and port portion as its parent, it MAY omit those portions of the URL, and include only the path, query string and fragment identifier portion of the URL.
* If a child Document or Link has the the same URL as its parent, it may omit the URL entirely.

#### Indentation and spacing

Client MAY choose to use a concise style as the default. Using this style will ensure that no spacing or indentation is used between tokens.

Clients MAY choose to allow an optional verbose style. Using this style will ensure that ":" delimiters have a following whitespace, "," delimiters have no following whitespace, and elements are newline indented, with four space character indentation level.

---

## HTML encoding

The HTML encoding for Core API allows servers to respond in a way that allows for direct Web browser based interaction with the API.

This encoding is primarily intended for usage in Web browsers, but MAY also be parsed as a machine readable format.

![HTML Encoding](../images/html-encoding.png)

Elements defined in the HTML encoding specification may include extra classes and attributes, and may be enclosed inside any parent HTML structure. This allows APIs to customize the style with which the browser based interaction is presented.

In order to to be properly supported the rendered HTML should include javascript and styling in order to allow the user to perform any available transitions included in the document.

The media type for this scheme may either be: `application/vnd.coreapi+html`, or `text/html`. To render in a Web browser, the `text/html` type should be used.

The [python client library](https://github.com/core-api/python-client) can be taken as the canonical example for implementing an HTML rendering.

### Document

**Documents are encoded as `table` elements with a `coreapi-document` class.**

The document name and URL are represented in a single row in the `thead` element.

The  `thead` element, SHOULD enclose a single `tr` element, which SHOULD enclose a single `th` element. This element contains the document name and URL. The name and URL SHOULD be included as an `a` element, with the `href` indicating the URL, and the element text indicating the document name.

The document content is represented as the `<tbody>` element enclosing multiple `<tr>` elements.

Each data item in the document is represented as a row, which SHOULD include a single `<th>` and `<td>` element. The `<th>` element SHOULD contain the key of the item, and the `<td>` element SHOULD contain the value of the item.

Each link item in the document is represented as a row, which SHOULD include a single `<th>` element, containing the link.

### Objects

**Objects are encoded as `<table>` elements with a `coreapi-object` class.**

The object content is represented as the `<tbody>` element enclosing multiple `<tr>` elements.

Each data item in the object is represented as a row, which SHOULD include a single `<th>` and `<td>` element. The `<th>` element SHOULD contain the key of the item, and the `<td>` element SHOULD contain the value of the item.

Each link item in the object is represented as a row, which SHOULD include a single `<th>` element, containing the link.

### Arrays

**Documents are encoded as `<table>` elements with a `coreapi-array` class.**

The array content is represented as the `<tbody>` element enclosing multiple `<tr>` elements.

Each row SHOULD containing a single `<th>` element, with a textual context indicating the array index for that row.

Each row SHOULD contain a single `<td>` element, containing the item at that point in the array.

### Links

**Links are encoded as `<a>` elements, with a `coreapi-link` class.**

The key under which the Link is contained by its parent Object or Document SHOULD be contained in the text content of the element.

The URL of the Link SHOULD be contained in the `href` value of the element.

The transition value of the Link SHOULD be include in a `data-trans` attribute.

The fields for the Link SHOULD be included in a `data-fields` attribute, which should be a whitespace separated list of the field names.

### Data primitives

**Data primitives are encoded as `<code>` and `<span>` elements.**

The `true`, `false` and `null` primitives SHOULD be enclosed within a `<code>` element, using their textual name as the content. For example `<code>true</code>`.

Number primitives SHOULD be enclosed within a `<code>` element.

String primitives, SHOULD be enclosed within a `<span>` element.

### Errors

**Errors are encoded as `<ul>` elements, with a `coreapi-error` class.**

Each message in the error SHOULD be included as a `<li>` element, with the text of the element containing the message value.
