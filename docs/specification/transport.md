# Transport

---

The **transport layer** defines how link transitions made at [the document layer](document.md) are mapped to network requests.

Core API currently defines a single HTTP and HTTPS transport scheme.

---

## The HTTP transport

Link transition types are mapped to HTTP methods as follows.

Transition type | HTTP method
----| ----
"follow" | `GET`
"action" | `POST`
"create" | `POST`
"update" | `PUT`
"delete" | `DELETE`

A link transition is effected by making an HTTP request to the link URL, with the appropriate method. Any link parameters provided are to be encoded as described below.

Clients MAY choose to include an appropriate `Accept` header in the request, indicating the encodings that the client supports.

The resulting content is decoded by selecting [an available codec](encoding.md) based on the response `Content-Type`. If no content is present in the response then the transport layer should return a `null` indicator.

The decoded object or `null` is then presented to [the document layer](document.md), which will effect the transform on the document.

---

## Encoding link parameters

#### Encoding for `GET` and `DELETE` requests

If the request method is `GET` or `DELETE` and link parameters are included, then the parameters MUST be included as additional query parameters in the request URL.

Because query parameters can only handle string encodings a simple mapping of the parameter values is required.

* String and Number values are encoded as-is, without any additional quoting.
* The `true`, `false` and `null` values SHOULD be encoded as `"1"`, `"0"`, and `""` respectively.
* Array and Object parameter values are not supported and their usage SHOULD raise an error.

#### Encoding for `POST` and `PUT` requests

If the request method is `POST` or `PUT` and link parameters are included, then the parameters MUST be `JSON` encoded. The encoded parameters MUST then be included in the request body and the `Content-Type` header of the request SHOULD be set to `application/json`.

---

## Constraints of the HTTP transport

The HTTP transport layer includes some constraints on the transitions that may be effected.

* As described above "follow" and "delete" links SHOULD only include String, Number, `true`, `false` and `null` parameter values. Object and Array values SHOULD NOT be supported, and their usage MAY raise an error.
* It is RECOMMENDED that URL lengths greater than 2,048 characters are not supported in requests. When they do occur, they MAY raise an error.
