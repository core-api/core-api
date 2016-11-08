# [Core API](http://www.coreapi.org/)

[![Join the chat at https://gitter.im/core-api/core-api](https://badges.gitter.im/core-api/core-api.svg)](https://gitter.im/core-api/core-api?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

---

Core API is a format-independent **Document Object Model** for representing Web APIs.

It can be used to represent **either Schema or Hypermedia responses**, and allows you
to interact with an API at the layer of an application interface, rather than a network interface.

Core API currently has implementations available for **Core JSON**, **Open API/Swagger**,
**HAL**, and **JSON Hyper-Schema**.

There is a **command line tool** that you can use to interact with APIs exposing
any of these formats, as well as a **Python client library**.

Using a Core API client is a more **robust** and **meaningful** way to interact with
your API than constructing HTTP requests and decoding responses. The dynamic client library
is always up to date with the API, and client code focuses solely on the interface being provided,
rather than dealing with network details and encodings.

---

## Examples

Core API can be used to interact with any API that exposes a supported Schema or Hypermedia format.

There are various examples of [using Core API as a client against existing APIs](/docs/tools-and-resources/example-services.md).

#### Hypermedia services

Below are two hypermedia style services available for demonstrating Core API.

You can interact with these example services either directly through your browser, by installing the command-line client, or by using one of the client libraries.

* **Notes** - Create, update and delete items from a list of notes. [http://notes.coreapi.org/](http://notes.coreapi.org/)
* **Game** - Find the treasure in 5 turns or less. [http://game.coreapi.org/](http://game.coreapi.org/)

Note that because Core API can be used to provide multiple different output formats,
it can also be used to provide **APIs that can be interacted with directly in the
browser**, by returning an HTML rendered format of the API response.

#### Using a Core API client

Let's try interacting with the "Game" service using the command line client.

In this example, you have have 5 guesses to try to find the position of the hidden treasure in a 3x3 grid.

First make sure to [install Python](https://www.python.org/downloads/), then...

```bash
$ pip install coreapi-cli  # Use Python's package manager `pip` to install the command-line client.
$ coreapi get http://game.coreapi.org/
<Home "http://game.coreapi.org/">
    new_game()
$ coreapi action new_game
<Game "http://game.coreapi.org/dee07521-6862-4744-95ec-812db90135bd">
    board: "...a
            ...b
            ...c
            123"
    description: "5 turns remaining."
    new_game()
    play([position])
$ coreapi action play --param position=b3
<Game "http://game.coreapi.org/dee07521-6862-4744-95ec-812db90135bd">
    board: "...a
            ..xb
            ...c
            123"
    description: "4 turns remaining."
    new_game()
    play([position])
```

---

## Overview

There are three layers to the Core API specification.

Name               |   Description
------------------- | ---------------------
[Document layer](http://www.coreapi.org/specification/document/) | The abstract object interface that clients interact with.
[Encoding layer](http://www.coreapi.org/specification/encoding/) | The mapping between a Document and a byte string.
[Transport layer](http://www.coreapi.org/specification/transport/) | How document interactions are mapped to network requests.

The following is an brief overview of the document layer, demonstrating how the client interacts with a Core API interface.

#### Document

Documents are the basic building blocks of Core API.

Documents are key-value pairs that contain the data and actions presented by the interface. Documents always have an associated URL, and should also have a title.

The top level element in any Core API interface is always a Document.

Let's take a look at a Core API document by using the command line client.

```bash
$ pip install coreapi-cli
$ coreapi get http://notes.coreapi.org/
<Notes "http://notes.coreapi.org/">
    notes: [
        <Note "http://notes.coreapi.org/123d4e35-cb09-40c3-98d3-d119e9079fca">
            complete: true
            description: "Do the weekly shopping"
            delete()
            edit([description], [complete]),
        <Note "http://notes.coreapi.org/0ace0b3b-9db2-4c10-989e-76c5c61265e7">
            complete: false
            description: "Fix the kitchen door"
            delete()
            edit([description], [complete])
    ]
    add_note(description)
```

We've got a document here that contains a couple of other nested documents. We can also see the actions and data that the interface exposes.

#### Links

Links are the available points of interaction that the interface presents.

Links have an associated URL and action, and may accept a set of named parameters.

Core API also has a concept of in-place transformations. Links that are marked as
in-place effect a partial transformation on the document, modifying or removing the nested document from the document tree. The 'put', 'patch' and 'delete' actions default to being in-place.

Let's return to the command line client, and take a look at calling some links.
We'll start by removing an existing note:

```bash
$ coreapi action notes 0 delete
<Notes "http://notes.coreapi.org/">
    notes: [
        <Note "http://notes.coreapi.org/123d4e35-cb09-40c3-98d3-d119e9079fca">
            complete: true
            description: "Do the weekly shopping"
            delete()
            edit([description], [complete])
    ]
    add_note(description)
```

Let's remove the final remaining note.

```bash
$ coreapi action notes 0 delete
<Notes "http://notes.coreapi.org/">
    notes: []
    add_note(description)
```

There should now be no notes remaining.

Okay, let's create a new note. In this case we'll want to include a named parameter
when acting on the link.

```bash
$ coreapi action add_note --param description="Email venue about conference dates"
<Notes "http://notes.coreapi.org/">
    notes: [
        <Note "http://notes.coreapi.org/e7785f34-2b74-41d2-ab3f-f754f688987c/">
            complete: false
            description: "Email venue about conference dates"
            delete()
            edit([description], [complete])
    ]
    add_note(description)
```

Finally we'll update the state of the note we've just created:

```bash
$ coreapi action notes 0 edit --param complete=true
<Notes "http://notes.coreapi.org/">
    notes: [
        <Note "http://notes.coreapi.org/e7785f34-2b74-41d2-ab3f-f754f688987c/">
            complete: true
            description: "Email venue about conference dates"
            delete()
            edit([description], [complete])
    ]
    add_note(description)
```

#### Data primitives

Data primitives are the set of basic datatypes that may be used to represent data in the interface.

Core API supports the same subset of data primitives as JSON. These are Object, Array, String, Integer, Number, `true`, `false`, and `null`.

```bash
$ coreapi show notes 0 description
"Email venue about conference dates"
$ coreapi show notes 0 complete
true
```

#### Errors

Following a link may result in an error. An error is a set of key-value pairs, which is used to represent any error information associated with the failed transition.

Encountering an error prevents any transition from taking place, and will normally be represented by an exception or other error status by the client library.

For example, in this case the server responds with an error when we fail to include a parameter:

```bash
$ coreapi action add_note
<Error: Invalid parameters>
    description: [
        "This field is required."
    ]
```

In this case the server responds with an error when we include an invalid parameter:

```bash
$ coreapi action add_note -p description='xxxxxxxxx-xxxxxxxxx-xxxxxxxxx-xxxxxxxxx-xxxxxxxxx-xxxxxxxxx-xxxxxxxxx-xxxxxxxxx-xxxxxxxxx-xxxxxxxxx-xxx'
<Error: Invalid parameters>
description: [
    "Ensure this field has no more than 100 characters."
]
```

---

## Get involved

For news and updates follow [@core-api](https://twitter.com/core_api), or [@_tomchristie](https://twitter.com/_tomchristie).

For discussion of the tools and specification, use the [Hypermedia Web mailing list](https://groups.google.com/forum/#!forum/hypermedia-web).


[command-line-client]: http://www.coreapi.org/tools-and-resources/command-line-client/
[python-client]: https://github.com/core-api/python-client
[javascript-client]: https://github.com/core-api/javascript-client
[example-server]: https://github.com/core-api/example-server
[corejson-encoding]: http://www.coreapi.org/specification/encoding/#core-json-encoding
[hal-encoding]: http://www.coreapi.org/specification/encoding/#hal-encoding
[openapi-encoding]: http://www.coreapi.org/specification/encoding/#openapi-swagger
[hyperschema-encoding]: http://www.coreapi.org/specification/encoding/#json-hyper-schema
[html-encoding]: http://www.coreapi.org/specification/encoding/#html-encoding
