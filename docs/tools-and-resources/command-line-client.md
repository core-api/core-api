# Command line client

The command line client allows you to interact with Core API services from the console.

The client includes commands for retrieving, displaying and interacting with various services, including HAL, JSON HyperSchema, and Core JSON.

It includes a rich set of functionality, including:

* History. Navigate backwards and forwards through a document history.
* Credentials. Associate Authorization headers with given domains.
* Bookmarks. Save and reload bookmarked documents.

## Installation

First you'll need to make sure you've got the Python programming language available on your system. If you need to install it, you can [do so here](https://www.python.org/downloads/). Either of the 2.7 or 3.x versions will do fine.

Once you've got Python you can use its package manager to install the `coreapi` command.

```bash
    $ pip install coreapi-cli
```

Finally, make sure that the client has been successfully installed.

```bash
    $ coreapi
    Usage: coreapi [OPTIONS] COMMAND [ARGS]
    [...]
```

### Installing additional codecs

The command line client uses a plugin system to allow users to install and
configure which media types are supported. Core JSON support is included
by default, but other codecs may also be installed, allowing support for
other schema and hypermedia types.

```bash
    $ pip install openapi-codec jsonhyperschema-codec hal-codec
    [...]
    Successfully installed
    $ coreapi codecs show
    Codecs
    corejson "application/vnd.coreapi+json"
    openapi "application/openapi+json"
    jsonhyperschema "application/schema+json"
    hal "application/hal+json"
```

## Fetching documents

You can test the command line client against one of the example services.

* **Notes** - Create, update and delete items from a list of notes. [http://notes.coreapi.org/](http://notes.coreapi.org/)
* **Game** - Find the treasure in 5 turns or less. [http://game.coreapi.org/](http://game.coreapi.org/)

To retrieve a document use the `get` subcommand.

```bash
    $ coreapi get http://notes.coreapi.org/
    <Notes "http://notes.coreapi.org/">
        notes: [
            <Note "http://notes.coreapi.org/123d4e35-cb09-40c3-98d3-d119e9079fca">
                complete: false
                description: "Write the great American novel"
                delete()
                edit([description], [complete]),
            <Note "http://notes.coreapi.org/0ace0b3b-9db2-4c10-989e-76c5c61265e7">
                complete: true
                description: "Fix the kitchen door"
                delete()
                edit([description], [complete])
        ]
        add_note(description)
```

## Inspecting documents

The `show` command will display the current active document.

```bash
    $ coreapi show
    <Notes "http://notes.coreapi.org/">
        notes: [
            <Note "http://notes.coreapi.org/123d4e35-cb09-40c3-98d3-d119e9079fca">
                complete: false
                description: "Write the great American novel."
                delete()
                edit([description], [complete]),
            <Note "http://notes.coreapi.org/0ace0b3b-9db2-4c10-989e-76c5c61265e7">
                complete: true
                description: "Fix the kitchen door."
                delete()
                edit([description], [complete])
        ]
        add_note(description)
```

You can also include a path argument to display only a part of the active document.

```bash
    $ coreapi show notes 0 description
    Write the great American novel.
```

## Interacting with documents

The `action` command is used to navigate and interact with a document.

```bash
    $ coreapi action notes 1 delete
    <Notes "http://notes.coreapi.org/">
        notes: [
            <Note "http://notes.coreapi.org/123d4e35-cb09-40c3-98d3-d119e9079fca">
                complete: false
                description: "Write the great American novel."
                delete()
                edit([description], [complete])
        ]
        add_note(description)
```

Some actions may take parameters which can be either optional or required. To include parameters, use the `--param key value` syntax.

```bash
    $ coreapi action notes 0 edit --param complete=true
    <Notes "http://notes.coreapi.org/">
        notes: [
            <Note "http://notes.coreapi.org/123d4e35-cb09-40c3-98d3-d119e9079fca">
                complete: true
                description: "Write the great American novel."
                delete()
                edit([description], [complete])
        ]
        add_note(description)
    $ coreapi action add_note --param description="Try Core API."
    <Notes "http://notes.coreapi.org/">
        notes: [
            <Note "http://notes.coreapi.org/1fc3f188-0051-43e4-9e43-5bcefd6b0ada">
                complete: false
                description: "Try Core API."
                delete()
                edit([description], [complete]),
            <Note "http://notes.coreapi.org/123d4e35-cb09-40c3-98d3-d119e9079fca">
                complete: true
                description: "Write the great American novel."
                delete()
                edit([description], [complete])
        ]
        add_note(description)
```

When using `--param`, the type of the input will be determined automatically.
If you want to be more explicit about the parameter type then use `--data` for
any null, numeric, boolean, list, or object inputs, and use `--string` for string inputs.

For example:

```bash
    coreapi action notes 0 edit --data complete=true --string description="Write the great American novel."
```
