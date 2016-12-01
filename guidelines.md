#RESTful API Server Design

* [Guidelines](#guidelines)
	* [Headers](#headers)
		* [Content-Type](#header-content-type)
		* [Accept](#header-accept)
		* [Accept-Language](#header-accept-language)
		* [Date](#header-date)
		* [Cache Headers](#header-cache)
		* [Last-Modified](#header-last-modified)
		* [ETag](#header-etag)
		* [Precondition Headers](#header-precondition)
	* [Resource Naming](#resource-naming)
		* [Collection and Item Pattern](#collection-item-pattern)
		* [Creation of Resources and Representations](#creating-resources)
		* [Resource Naming Syntax](#resource-naming-syntax)
	* [Hypermedia as the Engine of Application State](#hypermedia)
	* [Related Data](#related-data)
	* [Custom Data](#custom-data)
	* [Index](#index)
	* [Pagination](#pagination)
	* [Data Design](#data)
		* [Identifiers](#data-identifiers)
		* [Dates and Times](#data-date-time)
		* [Currency](#data-currency)
		* [Key-Value Pair Names](#data-key-names)
	* [HTTP Status Codes](#http-status-codes)
		* [Errors when HTTP Status Code is 4xx](#errors-when-4xx)
* [Works Cited](#works-cited)

##<a name="guidelines"></a>Guidelines

###<a name="headers"></a>Headers

* Services SHOULD limit themselves to standards based HTTP headers as defined in the [Internet Assigned Numbers Authority (IANA) Message Headers](http://www.iana.org/assignments/message-headers/message-headers.xhtml) (Protocol=HTTP, Status=Standard)

####<a name="header-content-type"></a>Content-Type

* Services SHOULD always provide the [RFC 7231 Content-Type](https://tools.ietf.org/html/rfc7231#section-3.1.1.5) header field in all responses.

####<a name="header-accept"></a>Accept

* Services SHOULD respect the [RFC 7231 Accept](https://tools.ietf.org/html/rfc7231#section-5.3.2) request header field whenever possible.
* Services SHOULD provide a [406 Not Acceptable](https://tools.ietf.org/html/rfc7231#section-6.5.6) HTTP status code when unable to respect the Accept header field.
* Services MAY disregard the Accept header field and treat the response as if it is not subject to content negotiation provided the [Content-Type](#header-content-type) header is present in the response.

####<a name="header-accept-language"></a>Accept-Language

* Services SHOULD respect the [RFC 7231 Accept-Language](https://tools.ietf.org/html/rfc7231#section-5.3.5) request header field and respond with the appropriately localized data when applicable.
* The Accept-Language request header field uses [RFC 5646 Tags for Identifying Languages](#RFC-5646). Simplest examples are:
	* en-US
	* de-DE
* Services MAY default to US English (en-US) in the absence of the Accept-Language request header field.

####<a name="header-date"></a>Date

* Services SHOULD provide the [RFC 7231 Date](https://tools.ietf.org/html/rfc7231#section-7.1.1.2) response header field.

####<a name="header-location"></a>Location

* Services SHOULD provide the [RFC 7231 7.1.2. Location](https://tools.ietf.org/html/rfc7231#section-7.1.2) response header field with the value expressed as a fully qualified domain name (FQDN) [RFC 3986 Uniform Resource Identifier (URI): Generic Syntax](#RFC-3986) for all operations which use the `POST` HTTP method.
* Services MAY provide the Location header as a relative value noting this requires client code to build URIs which should generally be avoided.

####<a name="header-cache"></a>Cache Headers

For a full understanding of caching see [RFC 7234 Hypertext Transfer Protocol (HTTP/1.1): Caching](https://tools.ietf.org/html/rfc7234). This section along with the [Last-Modified](#header-last-modified), [ETag](#header-etag) and [Precondition](#header-precondition) sections cover the caching responsibilities for a service.

* Services SHOULD help clients with cacheability of responses though the use of headers.
	* [RFC 7234 Age](https://tools.ietf.org/html/rfc7234#section-5.1)
	* [RFC 7234 Cache-Control](https://tools.ietf.org/html/rfc7234#section-5.2)
		* [RFC 7234 Response Cache-Control Directives](https://tools.ietf.org/html/rfc7234#section-5.2.2)
	* [RFC 7234 Expires](https://tools.ietf.org/html/rfc7234#section-5.3)
	* [RFC 7234 Pragma](https://tools.ietf.org/html/rfc7234#section-5.4)
	* [RFC 7234 Warning](https://tools.ietf.org/html/rfc7234#section-5.5)

####<a name="header-last-modified"></a>Last-Modified

* Services SHOULD provide the [RFC 7232 Last-Modified](https://tools.ietf.org/html/rfc7232#section-2.2) header in all responses.

####<a name="header-etag"></a>ETag

* Services SHOULD provide the [RFC 7232 ETag](https://tools.ietf.org/html/rfc7232#section-2.3) header in all responses.

####<a name="header-precondition"></a>Precondition Headers

* Services SHOULD support the following request header fields:
	* [RFC 7232 If-Match](https://tools.ietf.org/html/rfc7232#section-3.1)
	* [RFC 7232 If-None-Match](https://tools.ietf.org/html/rfc7232#section-3.2)
	* [RFC 7232 If-Modified-Since](https://tools.ietf.org/html/rfc7232#section-3.3)
	* [RFC 7232 If-Unmodified-Since](https://tools.ietf.org/html/rfc7232#section-3.4)
	* [RFC 7232 If-Range](https://tools.ietf.org/html/rfc7232#section-3.5)
* Services SHOULD respond with the following HTTP status codes to conditional requests:
	* [304 Not Modified](https://tools.ietf.org/html/rfc7232#section-4.1)
	* [412 Precondition Failed](https://tools.ietf.org/html/rfc7232#section-4.2)

###<a name="resource-naming"></a>Resource Naming

The following is a direct copy+paste from [RFC 3986 Uniform Resource Identifier (URI): Generic Syntax](#RFC-3986) as a baseline for the balance of this section.

```
  foo://example.com:8042/over/there?name=ferret#nose
  \_/   \______________/\_________/ \_________/ \__/
   |           |            |            |        |
scheme     authority       path        query   fragment
```

* All portions of a URI SHOULD be in lowercase.
* Included in the path in the following order:
	* Services SHOULD include their name in the first URI path segment.
		* Service names SHOULD be alphanumeric characters only.
		* Service names SHOULD NOT include `service` in their name.
		* Service names SHOULD NOT include punctuation.
		* Services SHOULD provide an [index](#index) at this root path.
	* Services MAY include a version number in the URI path using the form of vX where X is a positive integer: `v1`, `v2`, ... , `v10` in the second URI path segment.
		* Generally speaking, well designed services should never need a version number. It is included here for completeness.
		* For an excellent talk on why "no versioning" is a good thing fast forward to 27:50 in [GOTO 2014 • REST: I don't Think it Means What You Think it Does](https://youtu.be/pspy1H6A3FM?t=1670).
		* Versioning overload makes it difficult to independently evolve services.
	* Service MAY include a resource name.
	* Services MAY include a resource identifier.
	* Services MAY include a representation name.
* Services SHOULD have unique names.
	* `/stores`
	* `/aisles`
* Services SHOULD NOT share paths.
	* Examples:
		* `/stores/aisles` and `/stores/aisles`
	* Reasons:
		* Service(s) would have to implement a reverse proxy at each hop.
		* It breaks the consistency model of service/resource/representation.

####<a name="collection-item-pattern"></a>Collection and Item Pattern

* Services SHOULD arrange and name resources according to a `/collection/item` paradigm.
  * `collection` name is plural.
  * `item` name is singular.
* Services SHOULD have a base collection, the root of which is the [index](#index) resource.
* The base collection MAY contain individual items.
* The base collection MAY contain additional collections.

```
https://example.com/stores                                            // The base collection.
https://example.com/stores/76cc758e256c438b8e49546e0102b8c8           // A single item within the collection.

https://example.com/stores/aisles                                     // A related collection within the same service.
https://example.com/stores/aisles/c66f06fdb31b4882ad995e4d19ca7aed    // A single item within the collection.

https://example.com/stores/schema                                     // A schema collection within the service.
https://example.com/stores/schema/store.v1.schema.json                // A single item within the collection.

```

* Services MAY use paths which describe a parent-child hierarchy of resources available within the domain of the service.
	* Example: `https://example.com/stores/76cc758e256c438b8e49546e0102b8c8/aisles/c66f06fdb31b4882ad995e4d19ca7aed`
	* This pattern can cause resources to be named in an overly complex way, including the revealing of data storage paradigms, business logic, business unit seams or embracing of Remote Procedure Call (RPC) architecture style.
	* This pattern usually duplicates data which should be found in the resource itself.
	* This pattern MUST NOT be a substitute for proper handling of [Hypermedia as the Engine of Application State](#links-hateoas) or [Related Data](#related-data).

####<a name="creating-resources"></a>Creation of Resources and Representations

> One example (of many approaches) for resources, representations and related data can be found in [Resources and Representations](./resource-and-representation.md).

* Services MAY create as many representations as is needed.
	* Services are encouraged to do so in order to logically order the resources and representations.
	* Services are encouraged to do so to avoid overcomplicating paths.
* Services MAY make the same resource or representation available via multiple paths.
* Services MAY make subsets of resources available in representations.

#####<a name="creating-resources-example"></a>Examples

Item: `https://example.com/stores/76cc758e256c438b8e49546e0102b8c8`

```json
{
  "href": "https://example.com/stores/76cc758e256c438b8e49546e0102b8c8",
  "id": "76cc758e256c438b8e49546e0102b8c8",
  "template": "https://example.com/stores/{id}",
  "schema": "https://example.com/schemas/store.v1.schema.json",
  "name": "Alpha",
  "phone": "(425) 555-1212",
  "operations": [
    {
      "rel": "add-aisle",
      "href": "https://example.com/stores/aisles",
      "method": "POST"
    }
  ]
}
```

* A representation containing only the metadata and one key value pair, excluding all other data and the HATEOAS operations.
* A pattern of `resource/representation` has been used in naming: `76cc758e256c438b8e49546e0102b8c8/phone`

Representation: `https://example.com/stores/76cc758e256c438b8e49546e0102b8c8/phone`

```json
{
  "href": "https://example.com/stores/76cc758e256c438b8e49546e0102b8c8/phone",
  "id": "76cc758e256c438b8e49546e0102b8c8",
  "template": "https://example.com/stores/{id}/phone",
  "schema": "https://example.com/schemas/store.v1.schema.json",
  "phone": "(425) 555-1212"
}
```

* A representation which provides all items in the collection, excluding the HATEOAS operations.
* A naming pattern of `resource/representation` is still present: `/all`; The `resource` is implied and is the base collection.

Collection: `https://example.com/stores/all`

```json
[
  {
  	"href": "https://example.com/stores/76cc758e256c438b8e49546e0102b8c8",
  	"id": "76cc758e256c438b8e49546e0102b8c8",
  	"template": "https://example.com/stores/{id}",
    "schema": "https://example.com/schemas/store.v1.schema.json",
    "name": "Alpha",
    "phone": "(425) 555-1212"
  },
  {
    "comment": "...another store..."
  }
]
```

####<a name="resource-naming-syntax"></a>Resource Naming Syntax

Building upon everything in this section the following illustrates how teams should think of URIs when naming resources:

```
                  service  version   resource        identifier         representation
                        |    |       |                   |                    |
                       _|__  |   ____|__   ______________|_______________   __|_
                      /    \ /\ /       \ /                              \ /    \
  https://example.com/stores/v1/inventory/71b1d7acbb254e05b7f9060b0a29efab/digest?name=ferret#nose
  \___/   \_________/ \________________________________________________________/ \_________/ \__/
    |          |                                  |                                   |        |
 scheme    authority                             path                               query   fragment
```

* Naming paradigms and patterns for the `path` are optional and flexible.
* All of the following are valid:

```
https://example.com/stores/                                            // Base collection
https://example.com/stores/{collection}                                // Collection within the service.
https://example.com/stores/{identifier}                                // Single item within the base collection.
https://example.com/stores/{representation}                            // Representation of the base collection.
https://example.com/stores/{collection}/{identifier}                   // Single item within a collection.
https://example.com/stores/{identifier}/{representation}               // Representation of a single item within the base collection.
https://example.com/stores/{collection}/{identifier}/{representation}  // Representation of a single item within a collection.
```

###<a name="hypermedia"></a>Hypermedia as the Engine of Application State

> A primer on Hypermedia as the Engine of Application State can be found [here](./hateoas-model-example.md).

> The differences between [Hypermedia as the Engine of Application State](#hypermedia), [Related Data](#related-data) and [Identifiers](#data-identifiers) are explained [here](./id-related-data-hateoas.md).

* Services SHOULD make links available only when they are valid for the state of the service at the time of the response.
* Hypermedia as the Engine of Application State describes what client code can do with the server.
* The "application state" portion of the concept refers to the server (not the client).
* It is encapsulated within an `operations` array.
* Client code binds to the `rel` (relation) key and follows the link in the `href` key using the HTTP method in the `method` key.
  * This insulates against breaking changes as it allows the server to change the `href` value at any time and the client code will still work.

####<a name="hypermedia-operations"></a>Operations Schema

Name | Type | Format | Description
-----|------|--------|------------
`operations`|`array`|[`operation`](#hypermedia-operations-operation)|The hypermedia as the engine of application state (HATEOAS) information.

####<a name="hypermedia-operations-operation"></a>Operation Schema

Name | Type | Format | Description
-----|------|--------|------------
`rel`|`string`|[RFC 5988 Relation Type](https://tools.ietf.org/html/rfc5988#section-4)|**Required** Relation type as defined by the server. There are registered relation types listed in [RFC 5988 6.2.2. Initial Registry Contents](https://tools.ietf.org/html/rfc5988#section-6.2.2) including pagination relation types of `next`, `prev`, `first` and `last`.
`href`|`string`|[RFC 3986 Uniform Resource Identifier (URI)](#RFC-3986)|**Required** Hyperlink to the resource. This key name is borrowed from the HTML5 `href` element and behaves similarly.
`method`|`string`|[RFC 7231 Methods](https://tools.ietf.org/html/rfc7231#section-4.3)|Default is GET when `method` is not specified. Valid method names are RFC 7231 GET, HEAD, POST, PUT, DELETE, CONNECT, OPTIONS, TRACE + RFC 5789 PATCH.

####Example

* See the [Index](#index) or [Pagination](#pagination) sections for examples.

###<a name="related-data"></a>Related Data

> The differences between [Hypermedia as the Engine of Application State](#hypermedia), [Related Data](#related-data) and [Identifiers](#data-identifiers) are explained [here](./id-related-data-hateoas.md).

> One example (of many approaches) for resources, representations and related data can be found in [Resources and Representations](./resource-and-representation.md).

* All relationships to data outside of the document SHOULD be described by fully qualified domain name (FQDN) `URI` resource links.
* It is acceptable for data to reference other data which does not have a corresponding reference back; In the example given it would be fine if the inventory did not have a link to the store.
* Related data can reference resources outside of the domain.

####Example

In the following example:
* `inventory` is the inventory for a store.
* `aisles` is an array of aisles for the store.
* `map` is a hyperlink to the location for the store within a external domain.

Contents of https://example.com/stores/abc:

```json
{
	"name":"Example",
	"inventory": "https://example.com/inventory/123",
	"aisles": [
		"https://example.com/aisles/1",
		"https://example.com/aisles/2"
	],
	"map": "https://www.google.com/maps/place/1820+Wensley+Dr,+Charlotte,+NC+28210/@35.1541824,-80.8694037,17z"
}
```

###<a name="custom-data"></a>Custom Data

* Services SHOULD NOT use ambiguous key names such as `custom1`.
* Services SHOULD use a [RFC 3986 Uniform Resource Identifier (URI)](#RFC-3986) to disambiguate data and avoid name collisions.

Name | Type | Format | Description
-----|------|--------|------------
`customDataUri`|`string`|-|[RFC 3986 Uniform Resource Identifier (URI)](#RFC-3986)
-|`object`|-|Any valid JSON.

```json
{
	"https://example.com": {
		"alpha": {
			"bravo": "charlie"
		}
	}
}
```

###<a name="index"></a>Index

* Services SHOULD provide an index of links at the base URI which will be interesting to callers of the service.
* The links SHOULD follow the [RFC 5988 Web Linking](#RFC-5988) standard.
* The links MAY use [RFC 6570 URI Template](#RFC-6570).
* The index SHOULD be made available at the root of the service: `https://example.com/servicename`.

The service index is one of the keys to evolvability of services, allowing client code to:

* Reference an abstracted name rather than a link directly which allows service links to change without requiring client code changes.
* Simply follow given links rather than using `string` builders or concatenation to craft service links.

####Schema

An index leverages the Hypermedia as the Engine of Application State [`operations`](#hypermedia-operations) schema.

#### Example

```json
{
	"operations": [
		{
			"rel": "data-post",
			"href": "https://example.com/service/data/",
			"method": "post"
		},
		{
			"rel": "data-get",
			"href": "https://example.com/service/data/"
		},
		{
			"rel": "data-get-specific",
			"href": "https://example.com/service/data/{id}"
		}
	]
}
```

###<a name="pagination"></a>Pagination

* Services SHOULD always determine the pagination scheme rather than allowing client code to define (for example, with a query string parameter) to allow for service design and performance tuning independent of client code.
* Services SHOULD provide pagination operations in all responses using the following relation names according to the [RFC 5988 Web Linking](#RFC-5988) standard:
	* `next` - refers to the next resource in a ordered series of resources.
	* `prev` - refers to the previous resource in an ordered series of resources.
* Services MAY also use the following relation names as desired:
	* `first` - refers to the furthest preceding resource in a series of resources..
	* `last` - refers to the furthest following resource in a series of resources.
* Services MAY use any sort of name to denote pages of data. `page` is acceptable as would be any other name which makes sense internally to the service itself. Naming is of less importance here because client code simply follows the links rather than crafting URIs.
* For more information see [Pagination Design](/pagination-design.md)

####Schema

Pagination leverages the [Hypermedia as the Engine of Application State](#hypermedia) [`operations`](#hypermedia-operations) schema.

#### Example

```json
{
	"data": [
		{ "id": "Alpha" },
		{ "id": "Bravo" },
		{ "id": "Charlie" }
	],
	"operations": [
		{
			"rel": "next",
			"href": "https://example.com/service/page4"
		},
		{
			"rel": "prev",
			"href": "https://example.com/service/page2"
		},
		{
			"rel": "first",
			"href": "https://example.com/service/first"
		}
		{
			"rel": "last",
			"href": "https://example.com/service/last"
		}
	]
}
```

###<a name="data"></a>Data Design

####<a name="data-identifiers"></a>Identifiers

> The differences between [Hypermedia as the Engine of Application State](#hypermedia), [Related Data](#related-data) and [Identifiers](#data-identifiers) are explained [here](./id-related-data-hateoas.md).

* Services SHOULD identify unique resources with a string in the format of a [RFC 4122 A Universally Unique IDentifier (UUID) URN Namespace](#RFC-4122) UUID4 without dashes in a key named `id`.
	* This value SHOULD be separate from database key values (i.e., primary key) to avoid leaking implementation details.
* Services SHOULD provide a URI as the identifier (including the UUID4 value stored in `id`) for the resource in a key named `href`.
	* The key name `href` is based on the [HTML5 concept of links](https://www.w3.org/TR/html5/links.html), [RFC 5988 Web Linking Appendix A Notes on Using the Link Header with the HTML4 Format](https://tools.ietf.org/html/rfc5988#appendix-A) and is consistent with other sections of the guidelines where hyperlinks are provided using the Hypermedia as the Engine of Application State [Operations Schema](#hypermedia-operations-operation) such as [Related Data](#related-data), [Index](#index) and [Pagination](#pagination).
* Services SHOULD provide a [RFC 6570 URI Template](#RFC-6570) in a key named `template`.
	* The combination of `template` + `id` = `href` allows legacy relational database systems the flexibility to store URI values minimally to avoid database bloat (mainly due to indexing) and therefore storage costs.

#####Example

```json
{
	"href": "http://example.com/service/ae7d9679708f48e2951bbefd478b3d16",
	"id": "ae7d9679708f48e2951bbefd478b3d16",
	"template": "http://example.com/service/{id}"
}
```

####<a name="data-self-describing"></a>Self-describing Data

* Representations SHOULD be self-describing through the use of a `schema` key in the root of all payloads.
* The `schema` value SHOULD be in the form of a [RFC 3986 Uniform Resource Identifier (URI)](#RFC-3986).
* Examples of schema illustrating the paradigms found in this document can be found in [schema](./schema).

```json
{
	"schema": "http://example.com/schema/base.schema.json"
}
```

####<a name="data-date-time"></a>Dates and Times

* Services SHOULD choose from the following formats to accept and represent dates and timestamps:
  * **Preferred**: [RFC 3339 Date and Time on the Internet: Timestamps](#RFC-3339): `YYYY-MM-DDThh:mm:ss.nnn-hh:mmZ`
  * [ISO 8601:2004](#ISO-8601) UTC + Offset: `YYYY-MM-DDThh:mm:ss.nnn-hhmmZ`
  * [Unix Time](https://en.wikipedia.org/wiki/Unix_time) also known as `POSIX` and `Epoch` time.

#####Examples

```json
RFC 3339
{
  "keyName": "2016-10-18T03:56:06.798-07:00",
  "keyName": "2016-10-18T03:56:06.798Z"
}

ISO 8601
{
  "keyName": "2016-10-18T03:56:06.798-0700",
  "keyName": "2016-10-18T03:56:06.798Z"
}

Unix
{
  "keyName": 1477246370
}
```

####<a name="data-currency"></a>Currency

* All currency values SHOULD follow [ISO 4217:2015 Codes for the representation of currencies](#ISO-4217).
* A currency value SHOULD be an JSON object with the following keys:
 	* `amount` - Number with decimal and an optional negative (-) sign. The value SHOULD NOT contain formatting:
		* No monetary symbols.
		* No thousands separator.
	* `currency` - The [ISO 3166](#ISO-3166) country code.

```json
{
	"subtotal": {
		"amount": -1234.56,
		"currency": "USD"
	},
	"tax": {
		"amount": 1.00,
		"currency": "CHF"
	},
	"total": {
		"amount": 100,
		"currency": "JPY"
	}
}
```

####<a name="data-key-names"></a>Key-Value Pair Names

* Services SHOULD use [Camel Case](https://en.wikipedia.org/wiki/CamelCase) with the first letter in lower case for all key-value pair names. Example: `total`, `currencyCode` and `createDate`.

###<a name="http-status-codes"></a>HTTP Status Codes

* Services SHOULD always use the appropriate HTTP status codes as defined in [RFC 7231 Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content](https://tools.ietf.org/html/rfc7231).

The following table summarizes the status code ranges and is a direct copy from the aforementioned RFC.

Status Code Range|Definition
-----------------|----------
1xx|The 1xx (Informational) class of status code indicates an interim response for communicating connection status or request progress prior to completing the requested action and sending a final response.
2xx|The 2xx (Successful) class of status code indicates that the client's request was successfully received, understood, and accepted.
3xx|The 3xx (Redirection) class of status code indicates that further action needs to be taken by the user agent in order to fulfill the request.
4xx|The 4xx (Client Error) class of status code indicates that the client seems to have erred.
5xx|The 5xx (Server Error) class of status code indicates that the server is aware that it has erred or is incapable of performing the requested method.

####<a name="errors-when-4xx"></a>Errors when HTTP Status Code is 4xx

* Services SHOULD provide additional context via JSON entity document when 4xx HTTP status code is provided in the response.
* Generally speaking, most 4xx errors occur occur during a `PUT` or `POST` operation.
* There may not be a need for returning 4xx errors for a service that is primarily `GET` operations noting there are exceptions for `GET` operations, for example:
	* A `GET` should return a 4xx when there are invalid parameters in the query string.
	* A 404 Not Found should be returned when the `GET` URI is dynamic and a value is wrongly formatted, or does not exist, or the user does not have permission.
* The schema starts with an array which allows the services to expand the items within the errors without breaking the contract of the service itself. It is expected many services will only ever return a single item in the array.

#####<a name="errors-when-4xx-errors"></a>Errors Schema

Name | Type | Format | Description
-----|------|--------|------------
`errors`|`array`|[`error`](#errors-when-4xx-error)|**Required** An array of errors.

#####<a name="errors-when-4xx-error"></a>Error Schema
Name | Type | Format | Description
-----|------|--------|------------
`errorCode`|`string`|-|**Required** Machine readable code associated with the error. Examples: `dateTimeMissing`, `OutOfMem`, `invalidUser`. Contextual strings are recommended over numbers or UUID4 values.
`errorMessage`|`string`|-|**Required** Message associated with the error.
`dataPath`|`string`|-|Relative data path.
`schemaPath`|`string`|-|Relative schema path.
`errors`|`array`|[`error`](#errors-when-4xx-error)|An array of errors. Note: this points to this schema as errors can nest.

#####Examples

**Minimum**

```
HTTP/1.1 400 Bad Request
Date: Tue, 19 Jul 2016 18:23:16 GMT
```

```json
{
	"errors": [
		{
			"errorCode": "missingRequiredProp",
			"errorMessage": "Missing required property: dateTime"
		}
	]
}
```

**Non-nested**

```
HTTP/1.1 400 Bad Request
Date: Tue, 19 Jul 2016 18:23:16 GMT
```

```json
{
	"errors": [
		{
			"errorCode": "missingRequiredProp",
			"errorMessage": "Missing required property: dateTime"
		},
		{
			"errorCode": "parseDataFailed",
			"errorMessage": "Unable to parse data."
		}
	]
}
```

**Nested**

* Also included in this example are the optional keys.

```
HTTP/1.1 400 Bad Request
Date: Tue, 19 Jul 2016 18:23:16 GMT
```

```json
{
	"errors": [
		{
			"errorCode": "missingRequiredProp",
			"errorMessage": "Missing required property: dateTime",
			"dataPath": "/merchant/location/address",
			"schemaPath": "/allOf/0/required/0",
			"errors": [
				{
					"errorCode": "parseDataFailed",
					"errorMessage": "Unable to parse data.",
					"dataPath": "/merchant/location/address/state",
					"schemaPath": "/allOf/0/required/0/0"
				}    
			]
		}    
	]
}
```

##<a name="works-cited"></a>Works Cited

###<a name="rest"></a>Representational State Transfer (REST) Dissertation by Roy Fielding, 2000

* [https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)

Also included inline are the references to the parts of the dissertation outside of chapter 5.

* [5.1 Deriving REST](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1)
	* [5.1.1 Starting with the Null Style](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_1)
	* [5.1.2 Client-Server](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_2)
		* Reference: [3.4.1 Client-Server (CS)](https://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_4_1)
	* [5.1.3 Stateless](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_3)
		* Reference: [3.4.3 Client-Stateless-Server (CSS)](https://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_4_3)
	* [5.1.4 Cache](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_4)
		* Reference: [3.4.4 Client-Cache-Stateless-Server (C$SS)](https://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_4_4)
	* [5.1.5 Uniform Interface](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_5)
	* [5.1.6 Layered System](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_6)
		* Reference: [3.4.2 Layered System (LS) and Layered-Client-Server (LCS)](https://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_4_2)
	* [5.1.7 Code-On-Demand](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_7)
		* Reference: [3.5.3 Code on Demand (COD)](https://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_5_3)
	* [5.1.8 Style Derivation Summary](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_8)
* [5.2 REST Architectural Elements](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2)
	* [5.2.1 Data Elements](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2_1)
		* [5.2.1.1 Resources and Resource Identifiers](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2_1_1)
		* [5.2.1.2 Representations](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2_1_2)
	* [5.2.2 Connectors](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2_2)
	* [5.2.3 Components](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2_3)
* [5.3 REST Architectural Views](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_3)
	* [5.3.1 Process View](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_3_1)
	* [5.3.2 Connector View](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_3_2)
	* [5.3.3 Data View](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_3_3)
* [5.4 Related Work](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_4)
* [5.5 Summary](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_5)

###<a name="hateoas"></a>Hypermedia as the Engine of Application State

* [https://en.wikipedia.org/wiki/HATEOAS](https://en.wikipedia.org/wiki/HATEOAS)

###<a name="RFC-7230-7237"></a>RFC 7230-7237 Hypertext Transfer Protocol (HTTP/1.1)

* [RFC 7230 Hypertext Transfer Protocol (HTTP/1.1): Message Syntax and Routing](https://tools.ietf.org/html/rfc7230)
* [RFC 7231 Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content](https://tools.ietf.org/html/rfc7231)
	* [GET](https://tools.ietf.org/html/rfc7231#section-4.3.1)
	* [HEAD](https://tools.ietf.org/html/rfc7231#section-4.3.2)
	* [POST](https://tools.ietf.org/html/rfc7231#section-4.3.3)
	* [PUT](https://tools.ietf.org/html/rfc7231#section-4.3.4)
	* [DELETE](https://tools.ietf.org/html/rfc7231#section-4.3.5)
	* [CONNECT](https://tools.ietf.org/html/rfc7231#section-4.3.6)
	* [OPTIONS](https://tools.ietf.org/html/rfc7231#section-4.3.7)
	* [TRACE](https://tools.ietf.org/html/rfc7231#section-4.3.8)
	* [Request Header Fields](https://tools.ietf.org/html/rfc7231#section-5)
	* [Response Status Codes](https://tools.ietf.org/html/rfc7231#section-6)
	* [Response Header Fields](https://tools.ietf.org/html/rfc7231#section-7)
* [RFC 7232 Hypertext Transfer Protocol (HTTP/1.1): Conditional Requests](https://tools.ietf.org/html/rfc7232)
* [RFC 7233 Hypertext Transfer Protocol (HTTP/1.1): Range Requests](https://tools.ietf.org/html/rfc7233)
* [RFC 7234 Hypertext Transfer Protocol (HTTP/1.1): Caching](https://tools.ietf.org/html/rfc7234)
* [RFC 7235 Hypertext Transfer Protocol (HTTP/1.1): Authentication](https://tools.ietf.org/html/rfc7235)
* [RFC 7236 Initial Hypertext Transfer Protocol (HTTP) Authentication Scheme Registrations](https://tools.ietf.org/html/rfc7236)
* [RFC 7237 Initial Hypertext Transfer Protocol (HTTP) Method Registrations](https://tools.ietf.org/html/rfc7237)

###<a name="RFC-5789"></a>RFC 5789 PATCH Method for HTTP

* [https://tools.ietf.org/html/rfc5789](https://tools.ietf.org/html/rfc5789)

###<a name="RFC-5646"></a>RFC 5646 Tags for Identifying Languages

* [https://tools.ietf.org/html/rfc5646](https://tools.ietf.org/html/rfc5646)

###<a name="json"></a>Javascript Object Notation (JSON)

* [http://www.json.org](http://www.json.org)
* [http://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf](http://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf)
* [https://tools.ietf.org/html/rfc7159](https://tools.ietf.org/html/rfc7159)

###<a name="json=patch"></a>JavaScript Object Notation (JSON) Patch

* [https://tools.ietf.org/html/rfc6902](https://tools.ietf.org/html/rfc6902)

###<a name="json-schema"></a>JSON Schema

* [http://json-schema.org](http://json-schema.org)

###<a name="RFC-7519"></a>RFC 7519 JSON Web Token (JWT)

* [https://tools.ietf.org/html/rfc7519](https://tools.ietf.org/html/rfc7519)

Links to claims within the RFC:

* [4.1 Registered Claim Names](https://tools.ietf.org/html/rfc7519#section-4.1)
	* [4.1.1 "iss" (Issuer) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.1)
	* [4.1.2 "sub" (Subject) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.2)
	* [4.1.3 "aud" (Audience) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.3)
	* [4.1.4 "exp" (Expiration Time) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.4)
	* [4.1.5 "nbf" (Not Before) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.5)
	* [4.1.6 "iat" (Issued At) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.6)
	* [4.1.7 "jti" (JWT ID) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.7)

####Additional Resources

* [JWT Summary](./jwt-summary.md)
* [https://jwt.io](https://jwt.io)
* [Public Claim Names](http://www.iana.org/assignments/jwt/jwt.txt)

###<a name="RFC-3339"></a>RFC 3339 Date and Time on the Internet: Timestamps

* [https://tools.ietf.org/html/rfc3339](https://tools.ietf.org/html/rfc3339)

###<a name="RFC-5280"></a>RFC 5280 Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile

* [https://tools.ietf.org/html/rfc5280](https://tools.ietf.org/html/rfc5280)

###<a name="RFC-6749"></a>RFC 6749 OAuth 2.0 Authorization Framework

* [https://tools.ietf.org/html/rfc6749](https://tools.ietf.org/html/rfc6749)

###<a name="RFC-1034"></a>RFC 1034 Domain Names - Concepts and Facilities

* [https://tools.ietf.org/html/rfc1034](https://tools.ietf.org/html/rfc1034)

###<a name="RFC-3986"></a>RFC 3986 Uniform Resource Identifier (URI): Generic Syntax

* [https://www.ietf.org/rfc/rfc3986.txt](https://www.ietf.org/rfc/rfc3986.txt)

###<a name="RFC-4122"></a>RFC 4122 A Universally Unique IDentifier (UUID) URN Namespace

* [https://tools.ietf.org/html/rfc4122](https://tools.ietf.org/html/rfc4122)
* Services SHOULD use all lowercase with dashes preserved.
	* This RFC generally recommends a lower case when producing values (while allowing for uppercase) and requires case-insensitivity when parsing.
* For more information on UUID4 see [4.4. Algorithms for Creating a UUID from Truly Random or Pseudo-Random Numbers](https://tools.ietf.org/html/rfc4122#section-4.4).

###<a name="RFC-5988"></a>RFC 5988 Web Linking

* [https://tools.ietf.org/html/rfc5988](https://tools.ietf.org/html/rfc5988)

###<a name="RFC-6570"></a>RFC 6570 URI Template

* [https://tools.ietf.org/html/rfc6570](https://tools.ietf.org/html/rfc6570)

###<a name="IANA-headers"></a>Internet Assigned Numbers Authority (IANA) Message Headers

* [http://www.iana.org/assignments/message-headers/message-headers.xhtml](http://www.iana.org/assignments/message-headers/message-headers.xhtml)

###<a name="ISO-3166"></a>ISO 3166-1 Country Codes

* Online Browsing Platform: [https://www.iso.org/obp/ui/#search/code/](https://www.iso.org/obp/ui/#search/code/)
* [http://www.iso.org/iso/home/standards/country_codes.htm](http://www.iso.org/iso/home/standards/country_codes.htm)

###<a name="ISO-4217"></a>ISO 4217:2015 Codes for the representation of currencies

* [ISO 4217:2015 Codes for the representation of currencies](https://github..com/developer/api/tree/master/standards/ISO_4217_2015_en.PDF)

###<a name="ISO-8601"></a>ISO 8601:2004 Data elements and interchange formats - Information interchange - Representation of dates and times

* [http://www.iso.org/iso/catalogue_detail?csnumber=40874](http://www.iso.org/iso/catalogue_detail?csnumber=40874)

###<a name="RFC-2119"></a>RFC 2119 Key words for use in RFCs to Indicate Requirement Levels

* [https://www.ietf.org/rfc/rfc2119.txt](https://www.ietf.org/rfc/rfc2119.txt)
* All requirements are noted with uppercase (MUST, SHOULD) in bulleted lists to disambiguate from narrative prose using lower case (should, must).

The following table has been added for ease of understanding and is a direct copy of the definitions contained in RFC 2119.

Keyword|Synonyms|Definition
-------|--------|----------
MUST|Required, Shall|Absolute requirement.
MUST NOT|Shall Not|Absolute prohibition.
SHOULD|Recommended|There may exist valid reasons in particular circumstances to **ignore a particular item**, but the full implications must be understood and carefully weighed before choosing a different course.
SHOULD NOT|Not Recommended|There may exist valid reasons in particular circumstances when the particular **behavior is acceptable** or even useful, but the full implications should be understood and the case carefully weighed before implementing any behavior described with this label.
MAY|Optional|An item is truly optional
