# Core API

**Hypermedia driven object interfaces.**

Core API is a general purpose system for exposing service interfaces.

It allows you to build Web APIs that describe their available interface, and provides the following benefits:

* **Robust** - Clients interacting with a Core API service always have the available interactions presented to them as part of the interface.
* **Composable** - Documents may be nested, allowing you to fully express complex interfaces without having to make multiple network calls.
* **Explorable** - Client libraries allow you to inspect and interact with a Core API interface.
* **Evolvable** - Core API draws a proper separation between the object interface and the encoding and transport layers. This allows future iterations of a client library to add support for new and more efficient protocols, without needing to modify the client application.

Existing tooling and resources:

* There is a complete [Python client library][python-client] for Core API.
* A [Javascript client library][javascript-client] is currently planned.
* We have an [example server implementation][example-server], for demonstration purposes.
* A live service available at http://coreapi.herokuapp.com for testing.

---

## Overview

There are three layers to the Core API specification.

Name               |   Description
------------------- | ---------------------
Document layer | The abstract object interface that clients interact with.
Encoding layer | The mapping between a Document and a byte string.
Transport layer | How document interactions are mapped to network requests.

The following is an overview of the document layer, describing how the client interacts with a Core API interface.

The design work for a JSON-based encoding, and an HTTP transport layer specification have both been completed. These are properly expressed in the [Python client library][python-client], but they have not yet been written up as part of this documentation.

#### Document

Documents are the basic building blocks of Core API.

Documents are key-value pairs that contain the data and actions presented by the interface. Documents always have an associated URL, and should also have a title.

The top level element in any Core API interface is always a Document.

*In object-oriented terms a document can be thought of as an object.*

Let's take a look at a Core API document by using the Python client library.

    $ pip install coreapi
    $ python
    >>> import coreapi
    >>> doc = coreapi.get('http://coreapi.herokuapp.com/')
    >>> print(doc)
    <Notes 'http://coreapi.herokuapp.com/'>
        'notes': [
            <Note '/e7785f34-2b74-41d2-ab3f-f754f688987c/'>
                'complete': False,
                'description': 'Schedule design meeting',
                'delete': link(),
                'edit': link([description], [complete]),
            <Note '/626579bd-b2ba-40d0-92af-9ff0bfa5f276/'>
                'complete: True,
                'description': 'Write release notes',
                'delete': link(),
                'edit': link([description], [complete])
        ],
        'add_note': link(description)

We've got a document here that contains a couple of other nested documents. We can also see the actions and data that the interface exposes.

#### Links

Links are the available actions that the interface presents.

Links have an associated URL and transition type, and may accept a set of named parameters.

*In object-oriented terms a link can be thought of as a method.*

Calling a link will perform one of the following actions:

* Return a new document, or other media.
* Replace the document.
* Remove the document.

The transition types are defined as follows:

Name | Document transition | Safe | Idempotent
----| ---- | ---- | ----
"follow" | Return a new document, or other media. | ✓ | ✓
"action" | Replace the document. | ✗ | ✗
"create" | Return a new document, or other media. | ✗ | ✗
"update" | Replace the document. | ✗ | ✓
"delete" | Remove the document. | ✗ | ✓

When interacting with a nested document, the replace and removal transitions apply only to the child document. The new state is the existing document with the child transformation applied.

Let's return to the python client library, and take a look at calling some links. We'll start by removing all the existing notes:

    >>> while doc['notes']:
    >>>     doc = doc.action(['notes', 0, 'delete'])

The python client treats documents as immutable objects. Transitions return completely 
new documents, so note that we're always re-assigning the updated state to the `doc` variable.

There should now be no notes remaining:

    >>> print(doc)
    <Notes 'http://coreapi.herokuapp.com/'>
        'notes': [],
        'add_note': link(description)

Okay, let's create a new note:

    >>> doc = doc.action(['add_note'], description='Email venue about conference dates')
    >>> print(doc)
    <Notes 'http://coreapi.herokuapp.com/'>
        'notes: [
            <Note '/e7785f34-2b74-41d2-ab3f-f754f688987c/'>
                'complete': False,
                'description': 'Email venue about conference dates',
                'delete': link(),
                'edit': link([description], [complete])
        ],
        'add_note': link(description)

Finally we'll update the state of the note we've just created:

    >>> doc = doc.action(['notes', 0, 'edit'], complete=True)
    >>> print(doc)
    <Notes 'http://coreapi.herokuapp.com/'>
        'notes': [
            <Note '/e7785f34-2b74-41d2-ab3f-f754f688987c/'>
                'complete': True,
                'description': 'Email venue about conference dates',
                'delete': link(),
                'edit': link([description], [complete])
        ],
        'add_note': link(description)

#### Primitives

Primitives are the set of basic datatypes that may be used to represent data in the interface.

Core API supports the same subset of data primitives as JSON. These are Object, Array, String, Number, `true`, `false`, and `null`.

    >>> doc['notes'][0]['description']
    'Email venue about conference dates'
    >>> doc['notes'][0]['complete']
    True

#### Errors

Following a link may result in an error. An error is defined as having a list of strings, which represent any error message associated with the failed transition.

Encountering an error prevents any transition from taking place, and will normally be represented by an exception or other error status by the client library.

    >>> doc.add_note(description = 'x' * 999999)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    LinkError: ['description - Ensure this parameter has no more than 100 characters.']

---

## Design considerations

Because Core API documents are composable there are a few design decisions you'll want to be aware of, that may not exist in other API systems.

#### Rich interfaces

The status quo with many Web APIs today is to have a close relationship between items in the storage backend and API endpoints. While Core API *does* allow you to build collection type APIs it can also express richer interfaces.

You probably want to think of a Core API interface as being similar to a web page, but without any layout or style information. You can embed multiple controls and related elements inside a single document.

#### Actions should effect child elements

When in-place link transitions are followed they will only ever update the part of the document that the link is contained by. You should make sure that links are always included at the topmost level of any elements they might effect.

Failing to follow this constraint will mean that clients may need to have some implicit knowledge about which other parts of a document to reload once a transition takes place. This introduces coupling, and increases the number of required network requests.

#### Interlinking vs nesting

When building a Core API interface you'll need to decide on when to link to associated document, and when to nest an associated document. The former implies a necessary network request to retrieve the document, while the later embeds the document directly.

Normally this decision should be self-evident, and will depend on the type of the relationship that the link expresses.

---

## Plans for the future

I'm currently planning a significant amount of time into building the Core API specification, tooling and ecosystem. Possible future work includes the following:

* **Media links** - Links to images, videos and other downloadable media will be supported.
* **File uploads** - The HTTP transport should support files as parameters. When files are present multipart encoding will be used.
* **Documents as parameters** - We should support passing documents as parameters to a link. They can be encoded using the URL, and allow for operations such as `.swap_position(child_document_1, child_document_2)`.
* **Generator primitives** - We plan to add support for a generator primitive, that transparently deals with paginated result sets.
* **Field errors** - Currently errors only support a list of messages. We will also support messages being mapped to the fields that they relate too.
* **Primitive return values** - As well as document transitions and media downloads, we might want to support links that return primitive values.
* **Richer types** - We could consider expanding the primitives to include a richer set of types. A date-time primitive is one obvious candidate. We'll need to balance this consideration against maximizing language portability.
* **Authentication** - The HTTP transport needs to discuss what authentication schemes are supported, and how this works between differing domains.
* **Javascript client** - Currently we only have a Python client library. Having a fully supported Javascript library is a priority.
* **HTML encoding** - In addition to JSON, we will define an HTML encoding, and provide an API browser.
* **Command line client** - A command line client for interfacing with Core API, including history, forwards/backwards operations etc.
* **Realtime interfaces** - We need to specify at the document layer how realtime interfaces can be supported. At that point we can consider adding either or both of HTTP polling and web sockets as supported transports.
* **Server tooling** - We should introduce some server tooling support. In particular a package including renderers, parsers and a test client library for Django REST framework.
* **Timestamps and validation** - There may be scope for design work on timestamp and validation information being associated at the document layer. We could then build on this with better caching support, and support for conditional updates.
* **Other media types** - The separation of document and encoding concerns in Core API means that we could add support for other encodings such as `application/hal+json`. These encodings might have restrictions on what parts of the document interface they can support. For example some formats might not support parameterized links, or composable documents.

[python-client]: https://github.com/core-api/python-client
[javascript-client]: https://github.com/core-api/javascript-client
[example-server]: https://github.com/core-api/example-server
