# Transport

---

The **transport layer** defines how link transitions made at [the document layer](document.md) are mapped to network requests.

When effecting a link transition, an appropriate transport should be selected based
on the [scheme portion of the URL][scheme].

Core API currently defines a single HTTP transport.

---

## The HTTP transport

The HTTP transport supports the `http:` and `https:` schemes.

Link transitions for this transport are effected via an HTTP request/response, as determined below.

### Determining the request method

The request method is determined based on the link `action` property.

* The link `action` should be uppercased to give the resulting HTTP method.
* When no link `action` is specified, the default method is `GET`.

### Encoding link parameters

Link parameters are encoded into the request in different ways, depending on the parameter `location` property.

* `location="path"` - The parameter is included in the URL, using [URL templating][rfc6570].
* `location="query"` - The parameter is included in the URL, as a query parameter.
* `location="form"` - The parameter is included in request body.

When no parameter location is specified, the default is to use `location="query"` for `GET` and `DELETE` actions,
and `location="form"` for all other actions.

#### Encoding `path` and `query` parameters.

Because path and query parameters can only handle string encodings a simple mapping of the parameter values is required.

* String values are encoded as-is.
* Integer and Number values are encoded as their equivalent string representation.
* The `true` and `false` values SHOULD be encoded as the literal strings `"true"` and `"false"`.
* The `null` value SHOULD be encoded as the empty string, `""`.
* Array and Object parameter values are not supported and their usage SHOULD raise an error.

#### Encoding `form` parameters.

If form parameters are included, then the parameters are treated as a mapping of key-value pairs.
The resulting map MUST be `JSON` encoded. The encoded parameters MUST then be included in the request body and the `Content-Type` header of the request SHOULD be set to `application/json`.

### Determining the request headers

Requests are free to include any standard HTTP request headers, in particular:

* An appropriate `Accept` header MAY be included in the request, indicating the encodings that the client supports.
* Clients MAY choose to include additional headers to support authentication, request signing, or other use cases.

---

### Decoding the response

The result of following a link transition is to either return a Document, or raise an Error condition.

* The resulting content is decoded by selecting [an available codec](encoding.md) based on the response `Content-Type`.
* Either a Document or an Error condition MUST be returned by the codec.
* If no content is present in the response then the transport layer MUST return an empty document.

### Handling in-place transformations

An in-place transformation takes place when a link has `inplace=true`, and the
link is contained by a nested document.

When the `inplace` property is `null`, an appropriate default is used:

* The `PUT`, `PATCH` and `DELETE` methods default to `inplace=true`.
* All other methods default to `inplace=false`.

In the case of an in-place transformation, a *partial transformation* is effected on
the document tree, as follows:

* If a Document is returned in the HTTP response: The nested document is replaced with
the returned Document, and the new document tree is returned to the client.
* If no content is returned in HTTP response: The nested document is removed from
the document tree, and the new document tree is returned to the client.

### Coercing 4xx and 5xx responses to errors

When a 4xx or 5xx response is received the transport layer SHOULD coerce any
Document returned into an Error. This allows media types that do not support
an error primitive to be handled gracefully.

When a 4xx or 5xx response is received that contains a media type that is
not a supported hypermedia format, the transport MAY attempt to use fallback
media types to decode the content and return an Error. For example, a transport
MAY support graceful handling of error responses that have been returned in
plain `application/json` and/or `text/html` formats.


[scheme]: https://en.wikipedia.org/wiki/Uniform_Resource_Identifier#Syntax
[rfc6570]: http://tools.ietf.org/html/rfc6570
