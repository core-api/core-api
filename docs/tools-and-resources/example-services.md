# Example Services

Below are some examples of using Core API to interact with existing APIs.

## FoxyCart

FoxyCart expose a hypermedia API, using HAL.

Let's try following [their Getting Started example][foxycart-example], using the Core API command line client.

```bash
# Install the command line client, if not already done.
$ pip install coreapi
[...]
Successfully installed coreapi

# Attempt to get the sandbox API entry point.
$ coreapi get https://api-sandbox.foxycart.com
<Error>
    errors: [
        {
            logref: "id-1453305072"
            message: "The FOXY-API-VERSION request header is required in order to use the API. Please include the following header with all requests: FOXY-API-VERSION: 1"
        }
    ]
    total: 1

# Include the FoxyCart required version header.
$ coreapi headers add FOXY-API-VERSION 1  # Add a custom header to all outgoing requests.
Added header
Foxy-Api-Version: 1

# Get the sandbox API entry point.
$ coreapi get https://api-sandbox.foxycart.com
<Your API starting point. "https://api-sandbox.foxycart.com/">
    message: "Welcome to the FoxyCart API! Our hope is to be as self-documenting and RESTful as possible. Please let us know if you have any questions by posting in our forum http://forum.foxycart.com or emailing us at helpdesk@foxycart.com. As a last resort, you could read the documentation at http://wiki.foxycart.com. Your first action should be to create an OAuth Client, then a user, followed by a store."
    create_client()  # Note that HAL APIs won't include the parameter listings here.
- hide quoted text -
    property_helpers()
    rels()
    token()

# Let's add a bookmark so it's easier to get back to.
$ coreapi bookmarks add foxycart
Added bookmark
foxycart

# Try to follow this link.
$ coreapi action create_client
<Error>
    errors: [
        {
            logref: "id-1453305175"
            message: "No route found for \"GET /clients\": Method Not Allowed (Allow: OPTIONS, POST)"
        }
    ]
    total: 1

# HAL doesn't include the action information, so we specify it explicitly.
$ coreapi action create_client --action post
<Error>
    errors: [
        {
            logref: "id-1453305183"
            message: "redirect_uri can't be blank, redirect_uri is too short (minimum is 1 characters)"
        },
        {
            logref: "id-1453305183"
            message: "project_name can't be blank, project_name is too short (minimum is 1 characters)"
        },
        {
            logref: "id-1453305183"
            message: "company_name can't be blank, company_name is too short (minimum is 1 characters)"
        },
        {
            logref: "id-1453305183"
            message: "contact_name can't be blank, contact_name is too short (minimum is 1 characters)"
        },
        {
            logref: "id-1453305183"
            message: "contact_email can't be blank, contact_email is too short (minimum is 1 characters), contact_email is not a valid email address."
        },
        {
            logref: "id-1453305183"
            message: "contact_phone can't be blank, contact_phone is too short (minimum is 1 characters)"
        }
    ]
    total: 6

# Specify several parameters. Use --param for more verbose style.
$ coreapi action create_client -a post -p redirect_uri=http://example.com/redirect -p project_name=example -p company_name=example -p contact_name=tom -p contact_email=tom@example.com -p contact_phone=123
<This Client "https://api-sandbox.foxycart.com/clients/3731">
    message: "client 3731 created successfully."
    token: {
        access_token: "***"
        expires_in: 7200
        refresh_token: "***"
        scope: "client_full_access"
        token_type: "Bearer"
    }
    attributes()

# Add an Authorization header, for all requests to this domain.
$ coreapi credentials add api-sandbox.foxycart.com "Bearer ***"
Added credentials
api-sandbox.foxycart.com "Bearer ***"

# Go back to our bookmarked URL.
$ coreapi bookmarks get foxycart
<Your API starting point. "https://api-sandbox.foxycart.com/">
    message: "Welcome to the FoxyCart API! Our hope is to be as self-documenting and RESTful as possible. Please let us know if you have any questions by posting in our forum http://forum.foxycart.com or emailing us at helpdesk@foxycart.com. As a last resort, you could read the documentation at http://wiki.foxycart.com. Your first action should be to create an OAuth Client, then a user, followed by a store."
    client()
    create_user()
    property_helpers()
    rels()
    reporting()
    token()
```

## Heroku

Heroku publish a JSON Hyper-Schema document for their API, which the Core API
command line client can interact with.

Let's follow [their QuickStart example][heroku-example], using the command line client.

First make sure you have [a Heroku account][heroku-account] and have [obtained an auth token][heroku-auth-token].


```bash
# Install the command line client, if not already done.
$ pip install coreapi
[...]
Successfully installed coreapi

# Attempt to retrieve the API schema.
$ coreapi get https://api.heroku.com/schema
<Error: Not Found>
    message: "The requested API endpoint was not found. Are you using the right HTTP verb (i.e. `GET` vs. `POST`), and did you specify your intended version with the `Accept` header?"

# Use Heroku's versioned accept header.
$ coreapi headers add accept "application/vnd.heroku+json; version=3"
Added header
Accept: application/vnd.heroku+json; version=3

# Retrieve the API schema.
$ coreapi get https://api.heroku.com/schema
<Heroku Platform API "https://api.heroku.com/schema">
    account: {
        destroy(account_identity)
        read(account_identity)
        update(account_identity, new_password, password)
    }
    account-feature: {
        instances()
        read(account_feature_identity)
        update(account_feature_identity, enabled)
    }
    ...

# Note: If you've setup the Heroku client, its possible that you may not need
# to add credentials to your command line client, as it may be able to obtain
# existing Heroku credentials from your `.netrc` file.
$ coreapi action app create
<Error: Unauthorized>
    id: "unauthorized"
    message: "There were no credentials in your `Authorization` header. Try `Authorization: Bearer <OAuth access token>` or `Authorization: Basic <base64-encoded email + \":\" + password>`."

# Add our credentials.
$ coreapi credentials add api.heroku.com "Bearer 7b365743b4fdf2cdb1620538a064eebb532d5ba7"
Added credentials
api.heroku.com "Bearer 7b365743b4fdf2cdb1620538a064eebb532d5ba7"

# Create a new app.
$ coreapi action app create
{
    "repo_size": null,
    "name": "radiant-woodland-74673",
    "created_at": "2016-01-25T14:00:11Z",
    ...
}

# Rename the app.
$ coreapi action app update -p app_identity='radiant-woodland-74673' -p name=coreapi-kicks-ass
{
    "repo_size": null,
    "name": "coreapi-kicks-ass",
    "created_at": "2016-01-25T14:00:11Z",
    ...
}

# Remove the app.
$ coreapi action app destroy -p app_identity=coreapi-kicks-ass
```

[foxycart-example]: https://api.foxycart.com/docs/getting-started
[heroku-example]: https://devcenter.heroku.com/articles/platform-api-quickstart
[heroku-account]: https://devcenter.heroku.com/articles/platform-api-quickstart#prerequisites
[heroku-auth-token]: https://devcenter.heroku.com/articles/platform-api-quickstart#authentication
