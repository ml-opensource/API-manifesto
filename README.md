# API-manifesto
Documents how to write APIs

## Response

Generally we have a few rules the response has to follow:

When we don’t have any data, we need to return in the following way:
Consistency of key types. e.g. always return IDs as an integer in all endpoints.
Date/timestamps should always be returned with a time zone.
Pagination data should be returned in a meta key.
When an endpoint doesn't have meaningful data to return (e.g. when deleting something), use a status key to communicate the status of the endpoint.

### Body

#### Object at the root level

A body should always return an object at the root level. This ensures flexibiliy if the response needs to change in the future. We recommend using `data` for succesful requests with meaningful response data and `error` for unsuccesful requests with error data being returned.

##### Do

Returning a collection should be encapsulated in a key. This makes it easy to add other keys in the future, e.g. for pagination or meta data:

```json
{
    "data": [
        {
            "email": "..."
        },
        {
            "email": "..."
        }
    ]
}
```

Returning an object (e.g. a user) should use the `data` key:

```json
{
    "data": {
        "email": "..."
    }
}
```

Returning an error should use the `error` key:

```json
{
    "error": {
        "description": "..."
    }
}
```

##### Don't

Avoid returning collections at the top level in the response:

```json
[
    {
        "email": "..."
    },
    {
        "email": "..."
    }
]
```

Avoid returning data that are not encapsulated in a root key (`data` or `error`):

```json
{
    "error": true,
    "description": "..."
}
```

#### Return an empty collection when there's no results

To make it easier for the API consumer, return HTTP status code `200` with an empty collection instead of e.g. `204` with no body

##### Do

Combine HTTP status code `200` with empty collections:

```json
[]
```

##### Don't

Avoid using HTTP status code `204` for empty collections.

#### Use `null` for keys that are not set

To make the API more explicit and to make it easier for the consumer, always return keys that might not have a value.

##### Do

Return a value as `null`:

```json
{
    "data": {
        "email": null,
        "name": "..."
    }
}
```

##### Don't

Avoid unsetting a key because the value is `null`:

```json
{
    "data": {
        "name": "..."
    }
}
```

## Inspiration

These guidelines has been made with inspiration from the following API guidelines:

- [Microsoft API Guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md)
- [Zalando API Guidelines](https://opensource.zalando.com/restful-api-guidelines/)
- [PayPal API Guidelines](https://github.com/paypal/api-standards/blob/master/api-style-guide.md)
- [Atlassian API Guidelines](https://developer.atlassian.com/server/framework/atlassian-sdk/atlassian-rest-api-design-guidelines-version-1/)