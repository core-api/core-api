# Document

---

The **document layer** is the set of primitives that a Core API client interacts with.

The available primitives include both data and actions, and may be composed in nested structures.

Clients interact with a Core API service solely through interacting with the document layer.
In doing so they do not require any information about the underlying transport or encodings used.

The specification for mapping documents onto byte strings is documented by [the encoding layer](encoding.md). The specification for how network requests are made is documented by [the transport layer](transport.md).

---

## The document primitives

The top level element in a Core API interface MUST be a Document or Error.

#### Documents

A document is used as a container for the data and actions provided by the interface.

* A Document is a set of key-value pairs.
* The keys in a Document MUST be string values.
* The values in a Document can be any Core API primitive.
* A Document has an associated `"url"` property, which MUST be a string. The empty string is valid, and MAY be considered a default value by client libraries.
* A Document has an associated `"title"` property, which MUST be a string. The empty string is valid, and MAY be considered a default value by client libraries.

#### Links

A link is used to represent a possible transition that the client may take.

* A Link has an associated `"url"` property which MUST be a string. The empty string is valid, and MAY be considered a default value by client libraries.
* A Link has an associated `"action"` property which MUST be a string. The empty string is valid, and MAY be considered a default value by client libraries. For HTTP transports, this property [indicates the request method][http-action].
* A Link has an associated `"inplace"` property which MUST be either `true`, `false` or `null`. The `null` value MAY be considered a default value by client libraries. This property [indicates if nested documents should be modified in-place or not][http-inplace].
* A Link has an associated `"fields"` property, which is list of link parameters. The empty list is valid, and MAY be considered a default value by client libraries.

#### Link parameters

Links parameters allow a client to include additional information as part of a
transition, and may affect the resulting URL, query parameters or request body.

* A link parameter has an associated `"name"` property, which MUST be a string.
* A link parameter has an associated `"required"` property, which MUST be `true` or `false`. The `false` value MAY be considered a default value by client libraries.
* A link parameter has an associated `"location"` property which MUST be a string. The empty string is valid, and MAY be considered a default value by client libraries. This property [indicates how the parameter is encoded in the outgoing request][http-location].

#### Data primitives

Data primitives are the set of basic datatypes that may be used to represent data in the interface.

* Core API supports an equivalent subset of data primitives to JSON. These are Object, Array, String, Integer, Number, `true`, `false`, and `null`.

#### Errors

Errors are exception states that may occur when a transition fails. This element allows the server to respond with a message or messages indicating why the transition could not be effected.

Errors are similar to Documents, but because they do not represent a successful transition they are not
associated with a URL.

* An error is a set of key-value pairs.
* The keys in a Error MUST be string values.
* The values in a Error can be any Core API primitive.
* A Error has an associated `"title"` property, which MUST be a string. The empty string is valid, and MAY be considered a default value by client libraries.
* An Error is an exception case that may be encountered when effecting a transition, and occurs outside of the normal context of a Document element.
* An Error MUST NOT not be contained at any point in a Document.

---

## Interacting with documents

Documents provide both the data and available actions to the client, allowing clients to both inspect the available data, and to interact with the service.

A link transition is effected as follows:

* The client MUST provide the set of keys that index to the Link.
* The client MAY provide a set of parameters, which is a map of key-value pairs.
* The client MAY optionally explicitly override aspects of the link properties, such as the `action` property. This allows Core API to support hypermedia formats that are not able to fully express the full set of Core API primitives.

When a link is followed, an appropriate [transport](transport.md) should be selected based on the scheme of the URL.

The [transport](transport.md) is then responsible for returning either a new Document, or an Error condition.

[http-action]: /specification/transport/#determining-the-request-method
[http-inplace]: /specification/transport/#handling-in-place-transformations
[http-location]: /specification/transport/#encoding-link-parameters
