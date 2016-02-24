# [Core API](http://www.coreapi.org/)

**Hypermedia driven Web APIs.**

---

**Core API allows you to interact with you API in a more meaningful way.**

It can be used either:

* On the client side, to interact with APIs over a wide range of schema and hypermedia formats.
* On the server side, to build APIs that make themselves available in a number of encodings.

Core API currently supports **Open API/Swagger**, **HAL** and **JSON Hyper-Schema**.

It has a **command line tool** that you can use to interact with APIs exposing any of these formats, as well as a **Python** client library.

Using a Core API client is a more **robust** and **meaningful** way to interact with
you API than constructing HTTP requests and decoding responses. The dynamic client library
is always up to date with the API, and client code focuses solely on the interface being provided,
rather that dealing with network details and encodings.

When used on the server side, Core API lets you build **explorable** and **expressive** APIs.
Documents may be nested, and support in-place transitions, allowing you to express rich and complex interfaces without having to make multiple network calls, and the HTML encoding allows for fully web-browsable APIs.

---

#### Tooling

The following tooling is currently available for Core API.

* A [command line client][command-line-client] for interacting with services from the console.
* There is a complete [Python client library][python-client] for Core API.
* A [Javascript client library][javascript-client] is currently planned.
* We have an [example server implementation][example-server], for demonstration purposes.

#### Example services

You can interact with these example services either directly through your browser, by installing the command-line client, or by using one of the client libraries.

* **Notes** - Create, update and delete items from a list of notes. [http://notes.coreapi.org/](http://notes.coreapi.org/)
* **Game** - Find the treasure in 5 turns or less. [http://game.coreapi.org/](http://game.coreapi.org/)

For example, let's try interacting with the "Game" service using the command line client.

In this example, you have have 5 guesses to try to find the position of the hidden treasure in a 3x3 grid.

First make sure to [install Python](https://www.python.org/downloads/), then...

```bash
$ pip install coreapi  # Use Python's package manager `pip` to install the command-line client.
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

#### What does it look like?

Core API decouples the binary encoding from the document model, meaning that clients
and servers are able to interact with any one of several different formats.

The following are currently supported:

* [Core JSON][corejson-encoding] (A JSON based encoding designed specifically for Core API).
* [HAL][hal-encoding].
* [JSON HyperSchema][hyperschema-encoding].
* [OpenAPI / Swagger][openapi-encoding].
* An [HTML based encoding][html-encoding].

The HTML based encoding allows servers to present APIs that can be interacted with directly from a Web browser, for example:

![HTML encoding example](http://www.coreapi.org/images/html-encoding.png)

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
$ pip install coreapi
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
$ coreapi action add_note --params description="Email venue about conference dates"
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
$ coreapi action notes 0 edit --params complete=true
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
