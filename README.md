# Core API

**Hypermedia driven object interfaces.**

Core API is a general purpose system for exposing service interfaces.

It provides the following benefits:

* **Robust** - Clients interacting with a Core API service always have the available interactions presented to them as part of the interface.
* **Composable** - Documents may be nested, allowing you to fully express complex interfaces without having to make multiple network calls.
* **Explorable** - Client libraries allow you to inspect and interact with a Core API interface.
* **Evolvable** - Core API draws a proper separation between the object interface and the encoding and transport layers. This allows future iterations of a client library to add support for new and more efficient protocols, without needing to modify the client application.

There are three layers to the Core API specification.

----------------------|
Document layer | The abstract object interface that clients interact with.
Encoding layer | The mapping between a Document and a byte string.
Transport layer | How document interactions are mapped to network requests.

---

#### Document

Documents are the basic building blocks of Core API.

Documents are key-value pairs that contain the data and actions presented by the interface. Documents always have an associated URL, and should also have a title.

The top level element in any Core API interface is always a Document.

*In object-oriented terms a document can be thought of as an object.*

Let's take a look at a Core API document by using the Python client library.

    $ pip install coreapi
    $ python
    >>> import coreapi
    >>> doc = coreapi.get('http://coreapi.heroku.com/')
    >>> print(doc)
    <Notes 'http://coreapi.heroku.com/'>
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
* Update the current document.
* Remove the current document.

Let's return to the python client library, and take a look at calling some links. We'll start by removing all the existing notes:

    >>> while doc['notes']:
    >>>     doc = doc.action(['notes', 0, 'delete'])

The python client always treats transitions as returning a completely 
new document, so we're always re-assigning the updated state to the `doc` variable.

There should now be no notes remaining:

    >>> print(doc)
    <Notes 'http://coreapi.heroku.com/'>
        'notes': [],
        'add_note': link(description)

Okay, let's create a new note:

    >>> doc = doc.action(['add_note'], description='Email venue about conference dates')
    >>> print(doc)
    <Notes 'http://coreapi.heroku.com/'>
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
    <Notes 'http://coreapi.heroku.com/'>
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

**TODO**

---

* Media links.
* File uploads.
* Documents as parameters.
* Generator primitives.
* Field errors.
* Authentication.
* Realtime interfaces.
* Command line client.
* Javascript client.
* HTML encoding.
* Django test client.