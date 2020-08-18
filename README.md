<!--
Table of contents created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

use the following command: `gh-md-toc README.md | grep -v g-emoji` to filter out lines with emoji
-->

<!--
---
title: API Manifesto
tags: [api,manifesto,best practice]
link: https://github.com/monstar-lab-oss/API-manifesto
---
-->

<!--
New section template:

## Section

### TODO
<details>
<summary>Click to see examples</summary>

#### ✅

#### ⛔️

</details>
-->

# API Manifesto
Documents how to write APIs

## Introduction

This API Manifesto is a fast and easy overview of the most important elements for building a rock-solid API, which both the backend and frontend team enjoys working with.
APIs are suppose to be very strict, it's a contract between Backend & Frontend. In the same time, there is no reason for APIs to be different depending on which language / framework was used in the Backend. 

This is not a full blown manifesto, check the [Inspiration section](#inspiration) for references to other manifestos with more details.

Table of Contents
=================

   * [API Manifesto](#api-manifesto)
      * [Introduction](#introduction)
   * [Table of Contents](#table-of-contents)
   * [Requests](#requests)
      * [URLs](#urls)
         * [Prefix API endpoints](#prefix-api-endpoints)
         * [API versioning](#api-versioning)
      * [Request Headers](#request-headers)
         * [Content Negotiation](#content-negotiation)
         * [Protected endpoints](#protected-endpoints)
         * [Supporting localization](#supporting-localization)
         * [Making debugging easier](#making-debugging-easier)
   * [Responses](#responses)
      * [Response Body](#response-body)
         * [Object at the root level](#object-at-the-root-level)
         * [Return an empty collection when there are no results](#return-an-empty-collection-when-there-are-no-results)
      * [Response Headers](#response-headers)
         * [Timestamp](#timestamp)
         * [Content Negotiation](#content-negotiation)
      * [Use null or unset keys that are not set](#use-null-or-unset-keys-that-are-not-set)
   * [Status Codes](#status-codes)
   * [Auth](#auth)
      * [TODO](#todo-1)
   * [Error Handling](#error-handling)
   * [Localization](#localization)
      * [TODO](#todo-3)
   * [Timeouts](#timeouts)
      * [Client to Server](#client-to-server)
      * [Server to Server](#client-to-server)
   * [Pagination](#pagination)
      * [TODO](#todo-5)
   * [Inspiration](#inspiration)
   
# Requests

## URLs

### Prefix API endpoints

Prefix API endpoints with `/api/` to separate them from other URLs like HTML views served on the same server.

<details>
<summary>Click to see examples</summary>

#### ✅

Make sure to prefix your API endpoints with `/api/`:

```bash
www.example.com/api/v1/products/1
```

</details>

### API versioning

Versioning your API allows you to make non-backwards compatible changes to your API for newer clients by introducing new versions of endpoints while not breaking existing clients.

Include the API version in the URL. Versions start with 1 and are prefixed with `v`. The version path component should come right after the [`api`](#prefix-api-endpoints) path component.

In case of an existing API that doesn't have this versioning scheme but needs a new version, skip `v1` and go straight to `v2`.

> We're not recommending to use the headers (typically the Accept header) for versioning. The URL based approach is more obvious, usually simpler to implement, and testing URLs can be done in the browser.

<details>
<summary>Click to see examples</summary>

#### ✅

Include the API's version in the URL:

```bash
www.example.com/api/v2/auth/login
```

#### ⛔️

Don't depend on a header like "Accept" for versioning:

```bash
Accept = "application/vnd.example.v1+json"
```

</details>

### REST resources
After the API prefix and the version comes the part of the URL path that identifies the resource -- the piece of data you are interested in. Refer to a type of resource with a plural noun (eg. "users"). Directly following such a noun can be an identifier that points to a single instance.
A resource can also be nested, usually if there some sort of parent/child relationship. This can be expressed by appending another plural noun to the URL.

<details>
<summary>Click to see examples</summary>

#### ✅

Refer to a resource with a plural noun:

```bash
/api/v1/shops
```

#### ✅

Use an identifier following a noun to refer to a single entity:

```bash
/api/v1/products/42
```

#### ✅

Refer to a nested resource like so:

```bash
/api/v1/posts/1/comments
```

In some cases it can be ok to simplify and have the child object at the beginning as long as the child object's id is globally unique:

```bash
/api/v1/comments/87
```

Be careful with this since this approach lack the extra safety of asserting that the resource you are referring to belongs to the parent resource you think it does.

</details>

### Query parameters

Query parameters are like meta data to the (usually GET) URL request. They can be used when you need more control over what data should be returned. Good use cases include filters and sorting. Some things are better suited for headers, such as providing authentication and indicating the preferred encoding type.

<details>
<summary>Click to see examples</summary>

#### ✅

Use query parameters for a paginated endpoint to define which page and with how many results per page you want to retrieve:

```bash
/api/v1/posts?page=2&perPage=10
```

#### ⛔️

Do not use query parameters for authentication:

```bash
/api/v1/posts?apiKey=a7dhas8u
```

</details>

### HTTP Methods

HTTP methods are used to indicate what action to perform with the resource.

#### GET

A GET call is used to retrieve data and should not result in changes to the accessed resource. Multiple identical requests should have the same effect as a single request (idempotency).

#### POST

POST is used to create new resources.

#### PATCH

PATCH requests modify existing resources. Only fields that need to be updated need to be included - all others will be left as they are. In order to "unset" optional properties use `null` for the value.

#### PUT

With PUT calls we can replace entire objects. Only the database identifier should not be changed.

#### DELETE

To delete a resource, use the DELETE method.

#### HEAD

A HEAD call must never return a body. It can be used to see if an object exists and to see if a cached value is still up to date.

## Request Headers

This section goes through a couple of standard HTTP headers that we have found overselves using across various projects. A complete list of approved headers can be found in the [IANA Header Registry](https://www.iana.org/assignments/message-headers/message-headers.xhtml) with references to their respective RFC's.

#### Content negotiation

Using content negotiation, representations of a ressource are served differently at the same URI so the user agent can specify which format or content encoding suits best.

<details>
<summary>Click to see examples</summary>

##### ✅

Use `Accept` header to define the mime type the client is able to understand.

```bash
Accept = "application/json"
```

##### ⛔️

Avoid using the general default for all types.

```bash
Accept = "*/*"
```

##### Encoding

##### ✅

Use `Accept-Encoding` header to inform the server which encoding(s) the client supports.

```bash
Accept-Encoding = "gzip, deflate, br"
```

##### ⛔️

Avoid using the general default for all encoding types.

```bash
Accept-Encoding = "*"
```

</details>


#### Protected endpoints

Use the `Authorization` header to consume protected endpoints. See the [Auth](#auth) section for more information on how to handle authorization and authentication.

<details>
<summary>Click to see examples</summary>

##### ✅

Use `Authorization` to authorize:

```bash
Authorization = "Basic QWxhZGRpbjpPcGVuU2VzYW1l"
```

##### ⛔️

Avoid using custom headers for authorization:

```bash
UserToken = "QWxhZGRpbjpPcGVuU2VzYW1l"
```

</details>

#### Supporting localization

In order to support localization now and in the future, the `Accept-Language` should be used to indicate the client's language towards the API. 

<details>
<summary>Click to see examples</summary>

##### ✅

Use [ISO 639-1](http://www.loc.gov/standards/iso639-2/php/code_list.php) codes to indicate the preferred language of the response.

```bash
Accept-Language = "da"
```

Use a prioritized list of languages to influence the fallback language:

```bash
Accept-Language = "da, en"
```

##### ⛔️

Avoid using other standards than ISO 639-1 for specifying the preferred language:

```bash
Accept-Language = "danish"
```

</details>

#### Making debugging easier

Use headers to give the API information about the consumer to ease debugging. There's no industry standard, so feel free to make your own convention, just remember to use it consistently.

<details>
<summary>Click to see examples</summary>

##### ✅

```bash
Client-Meta-Information = iOS;staging;v1.2;iOS12;iPhone13
```

</details>

See:
 - [N-Meta-Vapor](https://github.com/nodes-vapor/n-meta)    
 - [N-Meta-PHP](https://github.com/monstar-lab-oss/n-meta-php)
 - [N-Meta-Laravel](https://github.com/monstar-lab-oss/n-meta-laravel)

# Responses

## Response Body

### Object at the root level

A body should always return an object at the root level. This enables including additional data about the response such as metadata separate from the object(s). We recommend using `data` for successful requests with meaningful response data and `error` for unsuccessful requests with error data being returned.

<details>
<summary>Click to see examples</summary>

#### ✅

Returning a collection should be encapsulated in a key:

```json
{
    "data": [
        {
            "username": "..."
        },
        {
            "username": "..."
        }
    ]
}
```

Returning an object (e.g. a user) should also use the `data` key:

```json
{
    "data": {
        "username": "..."
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

Please see the [error section](#errors) for more information.

#### ⛔️

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

</details>

### Return an empty collection when there are no results

To make it easier for the API consumer, return HTTP status code `200` with an empty collection instead of e.g. `204` with no body.

<details>
<summary>Click to see examples</summary>

#### ✅

Combine HTTP status code `200` with empty collections:

```json
{
    "data": []
}
```

#### ⛔️

Avoid using HTTP status code `204` for empty collections.

</details>

### Use `null` or unset keys that are not set

In case of missing values return them as `null` or don't include them. Do not use empty objects or empty strings.

<details>
<summary>Click to see examples</summary>

#### ✅

Return a value as `null`:

```json
{
    "data": {
        "email": null,
        "name": "..."
    }
}
```

Unset a key without a value:

```json
{
    "data": {
        "name": "..."
    }
}
```

#### ⛔️

Avoid including a key without a meaningful value:

```json
{
    "data": {
        "name": ""
    }
}
```
</details>

## Response Headers

This section goes through a couple of standard HTTP headers that we have found overselves using across various projects. A complete list of approved headers can be found in the [IANA Header Registry](https://www.iana.org/assignments/message-headers/message-headers.xhtml) with references to their respective RFC's.

#### Timestamp

Use the `Date` header to timestamp the processed response based on the server's date and time format. This header **MUST** be included in the response.

<details>
<summary>Click to see examples</summary>
<br/>

##### ✅

```bash
Date = "Tue, 18 Aug 2020 12:53:03 GMT"
```

##### ⛔️

Do not send back an empty value to the `Date` key.

```bash
Date = ""
```

</details>

#### Content negotiation

Using content negotiation, representations of a ressource are served differently at the same URI. The headers below describe the processed content of the body in the response.

<details>
<summary>Click to see examples</summary>

##### ✅

Use the `Content-Type` header to indicate the media type of the body content.

```bash
Content-Type = "application/json"
```

##### ⛔️

Avoid using the general default for all types.

```bash
Content-Type = "*/*"
```

##### Encoding

##### ✅

Use the `Content-Encoding` to indicate compression or encryption algorithms applied to the content.

```bash
Content-Encoding = "gzip, deflate, br"
```

##### ⛔️

Avoid using the general default for all encoding types

```bash
Content-Encoding = "*"
```

</details>

## Status Codes

<details>
<summary>Click to see examples</summary>

#### ✅

It's ok to use all available response codes, [See list](https://www.restapitutorial.com/httpstatuscodes.html)

Here is a list of the commonly used

#### 2xx
 - `200` -> **OK**, used when on `GET` request with successful response 
 - `201` -> **Created**, used on `POST` creating a record in DB
 - `202` -> **Accepted**, used when request has been received, but processed async
 - `204` -> **No Content**, used when no response is send, e.g. on `DELETE`

#### 3xx
 - `301`-> **Moved Permanently**, used if the resource has been moved to another `URI`
 - `304` -> **Not Modified**, used if `If-Modified-Since` header is send and nothing has changed since
 
#### 4xx

 - `400` -> **Bad Request**, used when request cannot be processed, remember to give more info
 - `401` -> **Unauthorized**, used when authorization session is invalid or missing
 - `403` -> **Forbidden**, used when a route / entity was requested, but users access level does not permit it 
 - `404` -> **Not Found**, used when a route / entity was not found
 - `405` -> **Method Not Allowed**, used when a route was hit with wrong method
 - `409` -> **Conflict**, used when an entity conflicts with another entity, e.g. duplicate entities / IDs
 - `422` -> **Unprocessable Entity**, used when validation rules on `POST/PATCH/PUT` are not followed
 - `429` -> **Too Many Requests**, used when you want to rate limit your API
 
#### 5xx 

- `500` -> **Internal Server Error**, used for undefined server errors, should store a record in an bug tracking tool like Bugsnag, Crashlytics, Rollbar, New relic
- `501` -> **Not Implemented**, used when you want to indicate that the feature/functionality is not implemented (yet)
- `502` -> **Bad Gateway**, used when an internal service was not reachable, e.g. in micro service architecture
- `503` -> **Service Unavailable**, used when an external service was not reachable, e.g. twilio.com 
- `504` -> **Gateway Timeout**, used to indicate that a request timed out (e.g. third party service took too long)
   
#### ⛔️

Custom response codes, eg:

- 490 
- 205
- 512

Use response body for the message instead

</details>

# Auth

## TODO

<details>
<summary>Click to see examples</summary>

### ✅

### ⛔️

</details>

# Error Handling

<details>
<summary>Click to see examples</summary>

### ✅

The error object needs to have the following: 
 - Be consistent
 - Have all required info
 - Easily parsable
 - Should be possible to build a solid UI on top, guiding the user what happened, and how to move on

```js
{
    "error": {
        "localizedTitle": "Title goes here", // Optional title localized for end user
        "localizedMessage": "Message goes here", // Optional message localized for end user
        "message": "Invalid format, digits required", // Message for developer
        "isRecoverable": true, // Is the error handled in the UI is fatal or can it be recovered, eg: try again
        "identifier": "PASSWORD_NOT_FOLLOWING_PATTERN", // Identifier which the consumer of the API can parse and switch case on
        "source": "LoginService" // In micro services architecture, you might want to understand what service
    },
    "payload": {
        "validationErrors": [{
            "field": "password",
            "errors": [{
                    "type": "required",
                    "localizedMessage": "Please enter a password"
                },
                {
                    "type": "regex",
                    "localizedMessage": "Password format should have following: 8 characters, 1 small letter, 1 big letter & 1 number"
                },
            ]
        }]
    },
    "metadata": {
        "errorID": "1234-ABC" // Optional ID for if the error is stored in DB, APM, Bug tracking tools like Bugsnag, Sentry, Rollbar, New Relic etc.
    }
}
```

### ⛔️

```json
{
  "error": "Internal server error"
}
```

</details>

# Localization

## TODO

<details>
<summary>Click to see examples</summary>

### ✅

### ⛔️

</details>

# Timeouts

Generally APIs should respond in less than 250ms on a wired connection. There needs to be a special reason for exceeding that. Further, it is important to understand that it will be harder and more expensive to scale the backend if response times are high.

It's common that the webserver configuration will timeout the request after 30 or 60 seconds. 

## Client to server

Since you never know what network the client is on, if they are in a metro or 100mBit wifi. Response time can vary a lot for several reason.
Therefore set timeouts to:

<details>
<summary>Click to see examples</summary>

### ✅

__Default:__ 15 sec

__File upload APIs:__ Align with web server timeout (eg 30 or 60 sec)

_If you are going to upload files above 5mb, consider having client upload directly to AWS S3, Dropbox etc. And sending path to server._

### ⛔️

__+ 30sec__

If API requests are taking more than 2 sec on a wired connection, consider changing the API design. 
Eg: Put the operation in a queue system like SQS, Redis, Beanstalkd and inform the client about operation is complete by push notification, web socket, email etc.

</details>

## Server to server

Server to service APIs should always be very stable due to connection being wired and stable. Therefore we can be much more aggressive about timeouts.
<details>
<summary>Click to see examples</summary>

### ✅

Depending on service: 1-5 sec timeouts.

Implementing a retry system is strongly advised. If the server is not responding in 1-5 sec, there is a high chance they never respond. Just fail and retry, up to a max of 3-5 retries, and then throw an exception.

### ⛔️

__+10 sec__

</details>

# Pagination

## TODO

<details>
<summary>Click to see examples</summary>

### ✅

### ⛔️

</details>

# Inspiration

These guidelines have been made with inspiration from the following API guidelines:

- [Microsoft API Guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md)
- [Zalando API Guidelines](https://opensource.zalando.com/restful-api-guidelines/)
- [PayPal API Guidelines](https://github.com/paypal/api-standards/blob/master/api-style-guide.md)
- [Atlassian API Guidelines](https://developer.atlassian.com/server/framework/atlassian-sdk/atlassian-rest-api-design-guidelines-version-1)
- [JSON:API](https://jsonapi.org)
