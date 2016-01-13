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
* A Link has an associated action which MUST be a string. The empty string is valid, and MAY be considered a default value by client libraries.
* A Link has an associated in-place marker which MUST be a boolean or null. The null value MAY be considered a default value by client libraries.
* A Link has an associated list of parameters. The empty list is valid, and MAY be considered a default value by client libraries.
* Each element in the parameter list is associated with a name, which MUST be a string.
* Each element in the parameter list is associated as either being *required* or being *optional*. The *optional* state MAY be considered a default value by client libraries.
* Each element in the parameter list is associated with a location which MUST be a string. The empty string is valid, and MAY be considered a default value by client libraries.

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

#### Link actions

Links MAY optionally include an action, which MUST be a string.

The set of valid actions and their meaning is determined by the transport layer.
For HTTP URLs the valid actions will be the HTTP methods, such as 'get', 'put', 'post', and 'delete'.

When no action is specified the transport layer is free to determine a default.

In the case of HTTP, the default method is 'get'.

#### Link parameters

Clients making a link transition may include parameters to the transition, as follows:

* Link transitions MAY optionally include a set of parameterized key-value pairs.
* All parameter values MUST be valid data primitives, as described above.
* Any *required* field items associated with the Link MUST be included in the set of parameters.
* Any *optional* field items associated with the Link MAY be included in the set of parameters.
* The client MAY include additional parameters.

#### In-place transitions

The in-place marker on a Link determines how the document is transformed.

This is relevant when the Link belongs to a nested document,
and allows for links to effect partial transformations of the document.

Transitions that are in-place should replace or remove the nested document
from the existing document tree, and return the new root document.

When no transition type is specified the transport layer is free to determine a default.

In the case of HTTP, the default transition depends on the method. The 'put',
'patch', and 'delete' methods default to being in-place.

#### The resulting network request

When undertaking a transition, the resulting transform is applied as follows.

* An [appropriate transport scheme should be selected](transport.md), based on the scheme of the Link URL. If the URL scheme is unsupported the transition should fail and raise an exception or other error indicator.
* The [transport layer](transport.md) is then responsible for handling the transition URL, action, and parameters.
* The transition MUST return a Document, an Error, or no content.
* A returned Error element should result in an exception or other error state, and no transformation is effected.

#### The resulting object transformation

In all other cases a transition is effected.

* If the Link is not an in-place transition: The value is returned to the client.
* If the Link is an in-place transition and content has been returned: The parent Document of the Link is replaced with the new value, and the new root Document is returned to the client.
* If the Link is an in-place transition and content has been returned: The parent Document of the Link is removed from the Document tree, and the new root Document is returned to the client.
