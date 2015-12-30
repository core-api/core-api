# Document

---

The **document layer** is the set of primitives that a Core API client interacts with.

The available primitives include both data and actions, and may be composed in nested structures.

This layer is a solely abstract object representation, and is the only point of contact with the interface that the client makes.

The specification for mapping documents onto byte strings is documented by [the encoding layer](encoding.md). The specification for how network requests are made is documented by [the transport layer](transport.md).

---

## The document primitives

#### Documents

A document is used as a container for the data and actions provided by the interface.

* The top level element in a Core API interface MUST be a Document.
* A Document is a set of key-value pairs.
* The keys in a Document MUST be string values.
* The values in a Document can be any Core API primitive.
* A Document has an associated URL, which MUST be a string. The empty string is valid, and MAY be considered a default value by client libraries.
* A Document has an associated title, which MUST be a string. The empty string is valid, and MAY be considered a default value by client libraries.

#### Links

A link is used to represent a possible transition that the client may take.

* A Link has an associated URL which MUST be a string. The empty string is valid, and MAY be considered a default value by client libraries.
* A Link has an associated transition type which MUST be one of "follow", "action", "create", "update" or "delete". The "follow" value MAY be considered a default value by client libraries.
* A Link has an associated list of parameters. The empty list is valid, and MAY be considered a default value by client libraries.
* Each element in the parameter list is associated with a name, which MUST be a string.
* Each element in the parameter list is associated as either being *required* or being *optional*. The *optional* state MAY be considered a default value by client libraries.

#### Data primitives

Data primitives are the set of basic datatypes that may be used to represent data in the interface.

* Core API supports the same subset of data primitives as JSON. These are Object, Array, String, Integer, Number, `true`, `false`, and `null`.

#### Errors

Errors are exception states that may occur when a transition fails. This element allows the server to respond with a message or messages indicating why the transition could not be effected.

* An Error has an associated set of messages, which MUST be a list of strings. The empty list is valid and MAY be considered a default value by client libraries.
* An Error is an exception case that may be encountered when effecting a transition, and occurs outside of the normal context of a Document element.
* An Error MUST NOT not be contained at any point in a Document.

---

## Effecting link transitions

When client acts on a Link, a transition is effected, and the resulting state is returned to the client.

#### Link parameters

Clients making a link transition may include parameters to the transition, as follows:

* Link transitions MAY optionally include a set of parameterized key-value pairs.
* Any *required* field items associated with the Link MUST be included in the set of parameters.
* Any *optional* field items associated with the Link MAY be included in the set of parameters.
* Any other key values MUST NOT be included in the set of parameters.
* All parameter values MUST be valid Data primitives, as described above.

#### Link transition types

The transition type of a Link determines the link behavior, as follows.

Name | Document transition | Safe | Idempotent
----| ---- | ---- | ----
"follow" | Return a new document, or other media. (\*) | ✓ | ✓
"action" | Replace the document. | ✗ | ✗
"create" | Return a new document, or other media. (\*) | ✗ | ✗
"update" | Replace the document. | ✗ | ✓
"delete" | Remove the document. | ✗ | ✓

(\*): **TODO** Link transitions returning other media are not yet fully described in the specification.

#### The resulting network request

When undertaking a transition, the resulting transform is applied as follows.

* An [appropriate transport scheme should be selected](transport.md), based on the scheme of the Link URL. If the URL scheme is unsupported the transition should fail and raise an exception or other error indicator.
* The [transport layer](transport.md) is then responsible for handling the transition URL and parameters.
* The "follow", "action", "create" and "update" transition types MUST return a Document or an Error.
* The "delete" transition type MUST return a `null` value or an Error.
* A returned Error element should result in an exception or other error state, and no transformation is effected.

#### The resulting object transformation

In all other cases a transition is effected.

* If the Link transition type is "follow" or "new" then the value is returned to the client.
* If the Link transition type is "action" or "updates", then the parent Document of the Link is replaced with the new value, and the result returned to the client.
* If the Link transition type is "delete", then the parent Document of the Link is removed from the Document tree, and the result returned to the client.
