# Service Index

> The service index design is based partially on [JSON Home](https://tools.ietf.org/html/draft-nottingham-json-home-06).

* [1 Introduction](#introduction)
  * [1.1 Notational Conventions](#notational-conventions)
* [2 Schema](#schema)
  * [2.1 Root](#schema-root)
  * [2.2 vars](#schema-vars)
  * [2.3 resources](#schema-resources)
  * [2.4 hints](#schema-hints)
  * [2.5 status](#schema-status)
  * [2.6 authSchemes](#schema-authSchemes)
* [3 Examples](#examples)
  * [3.1 Hello World - the simplest example](#examples-hello-world)
  * [3.2 Real World - a more realistic example an API would actually provide](#examples-real-world)
  * [3.3 Using variables](#examples-variables)
  * [3.4 Providing precondition details](#examples-precondition)
  * [3.5 Describing status](#examples-status)
  * [3.6 acceptRanges](#examples-accept-ranges)
  * [3.7 Describing authorization and authentication](#examples-auth-schemes)
  * [3.8 Expressing preferences](#examples-prefer)
* [4 Works Cited](#works-cited)

## <a name="introduction"></a>1 Introduction

One of the core architectural tenants of the internet is the use of [RFC 3986 Uniform Resource Identifier (URI): Generic Syntax](#RFC-3986) hyperlinks to navigate between resources. Traditionally, client code (including a human using a web browser) typically make use of documented static URLs (E.g., http://example.com) as the starting point which servers implement and any interaction outside of these bounds is uncharted territory.

Service index documents provide a fundamental starting place for the programmable internet like home pages do at root URLs for the World Wide Web used by humans with browsers.

* Humans use the service index document to learn about a server, write client code and help debug issues when they arise.
* Client code uses the service index document:
  * To interact with the server, navigate the resources and activities made available by the server.
  * To note and / or react to changes within the server.

Service index documents embrace and dovetail nicely with the HTTP protocol and REST principles:

* Server independence from client - Server interfaces can independently evolve without breaking client code through the addition of:
  * New link relations (Extensibility)
  * New metadata for existing link relations. (Evolvability)
* Abstraction of implementation details - The server can choose the URLs associated with a given link relation without breaking client code which is bound to relation descriptions which rarely, if ever, change.
* The service index aligns nicely with RESTful uniform interface constraints:
  * Identification of resources - Service index documents define the starter set of all of the resources available at a given root URL.
  * Hypermedia as the engine of application state (HATEOAS) - Home documents are the first place the server can define its state for client code, for example: emitting authorization boundaries by making only the resources / methods available as appropriate for the caller.
  * Self describing - The service index allows the server to be wholly self-describing starting at the root URL.
  * Manipulation of resources through representations - Service index documents provide excellent hints to client code on how to manipulate resources and reprentations to achieve objectives.

## <a name="notational-conventions"></a>1.1 Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119 Key words for use in RFCs to Indicate Requirement Levels](#RFC-2119).

## <a name="schema"></a>2 Schema

[JSON Schema](#json-schema) describing this schema can be found [here](./schema/com-example-serviceindex-2018-03-01.schema.json).

### <a name="schema-root"></a>2.1 Root

Name | Type | Format | Description
-----|------|--------|------------
`schema`|`string`|-|**Required** [RFC 3986 Uniform Resource Identifier (URI): Generic Syntax](#RFC-3986). Provides the self documentating feature for the resource / representation. Analogous to the 'describedby' relation type from [RFC 5988 Web Linking](#RFC-5988).
`href`|`string`|-|**Required** Identifier and hyperlink (aka 'self') for this resource using [RFC 3986 Uniform Resource Identifier (URI): Generic Syntax](#RFC-3986).
`title`|`string`|-|The human readable title of the API described by this service index resource.
`vars`|`array`|[`vars`](#schema-vars)|An array of variables to be used with hyperlink templates given in the document.
`resources`|`array`|[`resources`](#schema-resources)|**Required** An array of resources and hints about the resources used by humans writing client code and the client code itself.

### <a name="schema-vars"></a>2.2 vars

* `varName` + either `varDefinition` or `varValue` is required noting this either/or requirement cannot be described using [JSON Schema](#json-schema).
* Servers SHOULD avoid having client code build strings for URIs except in response to the service index document in order to preserve REST principles.

Name | Type | Format | Description
-----|------|--------|------------
`varName`|`string`|-|**Required** The variable name.
`varDefinition`|`string`|-|Link to a definition for the variable using [RFC 3986 Uniform Resource Identifier (URI): Generic Syntax](#RFC-3986).
`varValue`|`string`|-|The variable value.

### <a name="schema-resources"></a>2.3 resources

Name | Type | Format | Description
-----|------|--------|------------
`rel`|`string`|-|**Required** A [RFC 5988 Web Linking Link](#RFC-5988): link relation type. Examples: `author`, `describedBy`.
`href`|`string`|-|**Required** Link to a resource using [RFC 3986 Uniform Resource Identifier (URI): Generic Syntax](#RFC-3986) or [RFC 6570 URI Template](#RFC-6570).
`hints`|`array`|[hints](#schema-hints)|**Required** Hints provided to both humans writing the client code and the client code itself, describing the resource and activities available for a given link relation.

#### Versioning within the `rel` value

* Services SHOULD come up with a version scheme in the `rel` value. It is expected services will grow in functionality over time and with this growth comes the need for changes which can (or will) break existing client code. By versioning the `rel` value a service can more clearly hint the growth.
* Services MAY adopt the following pattern or some variation thereof:

```
  -{target}
    -{YYYY}
      -{MM}
        -{DD}

Examples:

  stores-2017-05-25
  products-2018-12-31
  product-2017-06-07
```

### <a name="schema-hints"></a>2.4 hints

Name | Type | Format | Description
-----|------|--------|------------
`method`|`string`|-|**Required** Valid method names are [RFC 7231 Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content](#RFC-7231) methods `GET`, `HEAD`, `POST`, `PUT`, `DELETE`, `CONNECT`, `OPTIONS` and `TRACE` plus the `PATCH` method from [RFC 5789 PATCH Method for HTTP](#RFC-5789).
`formats`|`array`|`string`|Hints the representation types that the resource produces and consumes, aligned with the ‘method’ hint.
`profiles`|`array`|`string`|Links to the schema(s) (typically [JSON Schema](#json-schema)) which define payloads the endpoint will generally accept. Based on [RFC 6096 The 'profile' Link Relation Type](#RFC-6096).
`preconditionRequired`|`array`|`string`|Hints that the resource requires state-changing requests (e.g., PUT, PATCH) using [RFC 7232 Hypertext Transfer Protocol (HTTP/1.1): Conditional Requests](#RFC-7232) to avoid conflicts due to concurrent updates. Examples: `etag`, `last-modified` and `If-Match`.
`acceptRanges`|`array`|`string`|Hints the range-specifiers available to the client for this resource; equivalent to the `Accept-Ranges` HTTP response header in [RFC 7233 Hypertext Transfer Protocol (HTTP/1.1): Range Requests](#RFC-7233).
`docs`|`string`|-|Hints the location for human-readable documentation for the relation type of the resource, a [RFC 3986 Uniform Resource Identifier (URI): Generic Syntax](#RFC-3986) referring to documentation that SHOULD be in HTML format.
`status`|`object`|[`status`](#schema-status)|Hints the status of the resource.
`authSchemes`|`array`|[`authschemes`](#schema-authSchemes)|Hints the resource requires authentication using [RFC 7235 Hypertext Transfer Protocol (HTTP/1.1): Authentication](#RFC-7235).
`prefer`|`array`|`string`|Hints the [RFC 7240 Prefer Header for HTTP](#RFC-7240) supported by the resource. Note that, as per the RFC, a preference can be ignored by the server.

### <a name="schema-status"></a>2.5 status

Name | Type | Format | Description
-----|------|--------|------------
`code`|`string`|-|The service index document uses existing HTTP status codes to hint the likely outcome of using an endpoint, for example: `410 Gone` (no longer available), `301 Moved Permanently` or `308 Permanent Redirect` (choices for deprecation of an endpoint).
`rels`|`array`|`string`|When the code is `301 Moved Permanently` or `308 Permanent Redirect` the `rel` points the consumer of this document to the alternative resource(s).

### <a name="schema-authSchemes"></a>2.6 authSchemes

Name | Type | Format | Description
-----|------|--------|------------
`scheme`|`string`|-|**Required** An HTTP authentication scheme according to [RFC 7235 Hypertext Transfer Protocol (HTTP/1.1): Authentication](#RFC-7235).
`realms`|`array`|`string`|An array of zero to many strings that identify [RFC 7235 Hypertext Transfer Protocol (HTTP/1.1): Authentication](#RFC-7235) protection spaces (realms) for the resource.

## <a name="examples"></a>3 Examples

### <a name="examples-hello-world"></a>3.1 Hello World - the simplest example

This example represents the bare minimum service index document.

* Reading through the example:
  * Self documentation is provided (`schema`) with a URI to the versioned service index schema to which the document conforms.
  * A self link (`href`) is noted.
  * Resources (`resources`) available are documented with:
    * A link relation (`rel`).
    * Associated hyperlink (`href`).
    * Hints (`hints`) with one method (`method`) which is an HTTP GET.

```json
{
  "schema": "http://example.com/schemas/com-example-serviceindex-2018-03-01.schema.json",
  "href": "http://example.com/apis",
  "resources": [
    {
      "rel": "hello-world-2017-05-25",
      "href": "http://example.com/api/helloworld",
      "hints": [
        {
          "method": "get"
        }
      ]
    }
  ]
}
```

### <a name="examples-real-world"></a>3.2 Real World - a more realistic example an API would actually provide

```json
{
  "schema": "http://example.com/schemas/com-example-serviceindex-2018-03-01.schema.json",
  "href": "http://example.com/apis",
  "title": "The API for Example",
  "resources": [
    {
      "rel": "accounts-2017-05-25",
      "href": "http://example.com/accounts",
      "hints": [
        {
          "method": "post",
          "formats": [ "application/json" ],
          "profiles": [ "http://example.com/schemas/com-example-accountinfo-2018-03-01.schema.json" ],
          "docs": "http://example.com/apis/creating-an-account.html"
        },
        {
          "method": "get",
          "formats": [ "application/json", "text/xml" ],
          "profiles": [ "http://example.com/schemas/com-example-accountinfo-2018-03-01.schema.json" ],
          "docs": "http://example.com/apis/listing-accounts.html"
        }
      ]
    }
  ]
}
```

### <a name="examples-variables"></a>3.3 Using variables

```json
{
  "schema": "http://example.com/schemas/com-example-serviceindex-2018-03-01.schema.json",
  "href": "http://example.com/apis",
  "vars": [
    {
      "varName": "specificPath",
      "varValue": "helloworld"
    },
    {
      "varName": "resourceIdentifier",
      "varDefinition": "http://example.com/apis/vars/types-of-accounts.html"
    }
  ],
  "resources": [
    {
      "rel": "var-name-and-value",
      "href": "http://example.com/api/{specificPath}",
      "hints": [
        {
          "method": "get"
        }
      ]
    },
    {
      "rel": "var-name-and-definition",
      "href": "http://example.com/api/{resourceIdentifier}",
      "hints": [
        {
          "method": "get"
        }
      ]
    },
    {
      "rel": "var-name-and-value-and-definition",
      "href": "http://example.com/api/{specificPath}/{resourceIdentifier}",
      "hints": [
        {
          "method": "get"
        }
      ]
    }
  ]
}
```

### <a name="examples-precondition"></a>3.4 Providing precondition details

```json
{
  "schema": "http://example.com/schemas/com-example-serviceindex-2018-03-01.schema.json",
  "href": "http://example.com/apis",
  "resources": [
    {
      "rel": "hello-world-2017-05-25",
      "href": "http://example.com/api/helloworld",
      "hints": [
        {
          "method": "put",
          "preconditionRequired": [ "If-Match", "If-Modified-Since" ]
        }
      ]
    }
  ]
}
```

### <a name="examples-status"></a>3.5 Describing status

```json
{
  "schema": "http://example.com/schemas/com-example-serviceindex-2018-03-01.schema.json",
  "href": "http://example.com/apis",
  "resources": [
    {
      "rel": "hello-world-2017-05-25",
      "href": "http://example.com/api/helloworld",
      "hints": [
        {
          "method": "get",
          "status": {
            "code": "200 OK"
          }
        }
      ]
    },
    {
      "rel": "hello-planet-2017-05-25",
      "href": "http://example.com/api/helloplanet",
      "hints": [
        {
          "method": "get",
          "status": {
            "code": "301 Moved Permanently",
            "rels": [ "http://example.com/api/hellojupiter" ]
          }
        }
      ]
    }
  ]
}
```

### <a name="examples-accept-ranges"></a>3.6 acceptRanges

```json
{
  "schema": "http://example.com/schemas/com-example-serviceindex-2018-03-01.schema.json",
  "href": "http://example.com/apis",
  "resources": [
    {
      "rel": "hello-world-2017-05-25",
      "href": "http://example.com/api/helloworld",
      "hints": [
        {
          "method": "get",
          "acceptRanges": "bytes"
        }
      ]
    }
  ]
}
```

### <a name="examples-auth-schemes"></a>3.7 Describing authorization and authentication

```json
{
  "schema": "http://example.com/schemas/com-example-serviceindex-2018-03-01.schema.json",
  "href": "http://example.com/apis",
  "resources": [
    {
      "rel": "hello-world-2017-05-25",
      "href": "http://example.com/api/helloworld",
      "hints": [
        {
          "method": "get",
          "authSchemes": [
            {
             "scheme": "Basic",
             "realms": [ "private", "example" ]
           }
          ]
        }
      ]
    }
  ]
}
```

### <a name="examples-prefer"></a>3.8 Expressing preferences

```json
{
  "schema": "http://example.com/schemas/com-example-serviceindex-2018-03-01.schema.json",
  "href": "http://example.com/apis",
  "resources": [
    {
      "rel": "hello-world-2017-05-25",
      "href": "http://example.com/api/helloworld",
      "hints": [
        {
          "method": "post",
          "prefer": [ "respond-async" "wait", "return" ]
        }
      ]
    }
  ]
}
```

## <a name="works-cited"></a>4 Works Cited

### <a name="RFC-2119"></a>RFC 2119 Key words for use in RFCs to Indicate Requirement Levels

* [https://www.ietf.org/rfc/rfc2119.txt](https://www.ietf.org/rfc/rfc2119.txt)

### <a name="RFC-3986"></a>RFC 3986 Uniform Resource Identifier (URI): Generic Syntax

* [https://www.ietf.org/rfc/rfc3986.txt](https://www.ietf.org/rfc/rfc3986.txt)

### <a name="RFC-5789"></a>RFC 5789 PATCH Method for HTTP

* [https://tools.ietf.org/html/rfc5789](https://tools.ietf.org/html/rfc5789)

### <a name="RFC-5988"></a>RFC 5988 Web Linking

* [https://tools.ietf.org/html/rfc5988](https://tools.ietf.org/html/rfc5988)
  * [Link Relation Types](https://tools.ietf.org/html/rfc5988#section-4)

### <a name="RFC-6096"></a>RFC 6096 The 'profile' Link Relation Type

* [https://tools.ietf.org/html/rfc6906](https://tools.ietf.org/html/rfc6906)

### <a name="RFC-6570"></a>RFC 6570 URI Template

* [https://tools.ietf.org/html/rfc6570](https://tools.ietf.org/html/rfc6570)

### <a name="RFC-7231"></a>RFC 7231 Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content

* [https://tools.ietf.org/html/rfc7231](https://tools.ietf.org/html/rfc7231)
  * [Method Definitions](https://tools.ietf.org/html/rfc7231#section-4.3)

### <a name="RFC-7232"></a>RFC 7232 Hypertext Transfer Protocol (HTTP/1.1): Conditional Requests

* [https://tools.ietf.org/html/rfc7232](https://tools.ietf.org/html/rfc7232)

### <a name="RFC-7233"></a>RFC 7233 Hypertext Transfer Protocol (HTTP/1.1): Range Requests

* [https://tools.ietf.org/html/rfc7233](https://tools.ietf.org/html/rfc7233)

### <a name="RFC-7235"></a>RFC 7235 Hypertext Transfer Protocol (HTTP/1.1): Authentication

* [https://tools.ietf.org/html/rfc7235](https://tools.ietf.org/html/rfc7235)
  * [Protection Space (Realm)](https://tools.ietf.org/html/rfc7235#section-2.2)
  * [Authentication Scheme Registry](https://tools.ietf.org/html/rfc7235#section-5.1)

### <a name="RFC-7240"></a>RFC 7240 Prefer Header for HTTP

* [https://tools.ietf.org/html/rfc7240](https://tools.ietf.org/html/rfc7240)

### <a name="json-schema"></a>JSON Schema

* [http://json-schema.org/](http://json-schema.org/)
