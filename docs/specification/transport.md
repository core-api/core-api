# Transport

---

The **transport layer** defines how link transitions made at [the document layer](document.md) are mapped to network requests.

Core API currently defines a single HTTP and HTTPS transport scheme.

---

## The HTTP transport

A link transition is effected by making an HTTP request to the link URL.

The HTTP method is determined by uppercasing the link action, and defaults to `GET`.

Any link parameters provided are to be encoded as described below.

Clients MAY choose to include an appropriate `Accept` header in the request, indicating the encodings that the client supports.

The resulting content is decoded by selecting [an available codec](encoding.md) based on the response `Content-Type`. If no content is present in the response then the transport layer should return an indicator value, such as `null`.

The decoded object or a "no-content" indicator is then presented to [the document layer](document.md), which will effect the transform on the document.

---

## Encoding link parameters

Link parameters are encoded into the request in different ways, depending on their associated location.

* `path` - The parameter is included in the URL, using URL templating.
* `query` - The parameter is as a URL query parameter.
* `form` - The parameter is included in request body.

When no parameter location is specified, the default is `query` for `GET` and `DELETE` actions,
and `form` for all other actions.

#### Encoding `path` and `query` parameters.

Because path and query parameters can only handle string encodings a simple mapping of the parameter values is required.

* String and Number values are encoded as-is, without any additional quoting.
* The `true`, `false` and `null` values SHOULD be encoded as `"1"`, `"0"`, and `""` respectively.
* Array and Object parameter values are not supported and their usage SHOULD raise an error.

#### Encoding `form` parameters.

If form parameters are included, then the parameters are treated as a mapping of key-value pairs.
The resulting map MUST be `JSON` encoded. The encoded parameters MUST then be included in the request body and the `Content-Type` header of the request SHOULD be set to `application/json`.

---

## Constraints of the HTTP transport

The HTTP transport layer includes some constraints on the transitions that may be effected.

* As described above "GET" and "DELETE" links SHOULD only include String, Integer, Number, `true`, `false` and `null` parameter values. Object and Array values SHOULD NOT be supported, and their usage MAY raise an error.
* It is RECOMMENDED that URL lengths greater than 2,048 characters are not supported in requests. When they do occur, they MAY raise an error.
