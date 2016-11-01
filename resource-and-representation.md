# Resources and Representations

There are a number of challenges when there are large resource definitions and a polyglot of microservices in terms of making all of the data (to include related data) available as needed by client code.

The following design and examples are intended to provide a starting point for services as they investigate data design and end points for client code.

* [Schema](#schema)
	* [Base](#schema-base)
	* [List](#schema-list)
	* [Operation](#schema-operation)
	* [Store](#schema-store)
	* [Aisle](#schema-aisle)
	* [Error](#schema-error)
* [Service](#service)
	* [Retrieve the service index](#service-retrieve-index)
	* [Create a store](#service-create-store)
	* [Create some aisles](#service-create-aisles)
	* [Retrieve the store](#service-retrieve-store)
	* [Get a store and list of aisles](#service-retrieve-store-aisle-list)
	* [Get a store with aisles data](service-retrieve-store-aisle-data)

##<a name="schema"></a>Schema

* This overall data model describes stores and their aisles.

###<a name="schema-base"></a>Base

* [base.schema.json](./schema/base.schema.json)
* The base schema provides a starting point for all representations.
* It is the default schema for all representations.
* It is the schema used for: metadata, HATEOAS operations and errors (should the occur).

Name | Type | Format | Description
-----|------|--------|------------
`schema`|`string`|[RFC 3986 URI](https://www.ietf.org/rfc/rfc3986.txt)|**Required** Hyperlink to the schema describing this representation to fulfill the self-describing portion of the uniform interface.
`href`|`string`|[RFC 3986 URI](https://www.ietf.org/rfc/rfc3986.txt)|Hyperlink to this representation, matches the URI the client code used.
`id`|`string`|-|For relational database systems to save the identifier separate from the URI template. When combined with the `template` key results in the `href` key value.
`template`|`string`|[URI Template](https://tools.ietf.org/html/rfc6570)|For relational database systems to save the template separate from the identifier.
`relatedData`|`array`|-|Inline representations of related data.
`relatedLists`|`array`|[`list`](#schema-list)|List of hyperlinks to related data representations. There can be multiple lists, each with their own schema.
`alternates`|`array`|[`operation`](#schema-base-operation)|Alternate representations of the same resource.
`operations`|`array`|[`operation`](#schema-base-operation)|The hypermedia as the engine of application state (HATEOAS) information.
`errors`|`array`|[`error`](#schema-base-error)|The errors collection for when the response code is 4xx.

###<a name="schema-list"></a>List

* [list.schema.json](./schema/list.schema.json)
* A list allows related data to be provided to the client which can optionally `GET` same.

Name | Type | Format | Description
-----|------|--------|------------
`schema`|`string`|-|**Required** The schema used to describe all of the items in the list, from [base](#schema-base).
`listItems`|`array` of `string`|[RFC 3986 URI](https://www.ietf.org/rfc/rfc3986.txt)|**Required** Hyperlinks to the related representations which all match the `schema`.

###<a name="schema-base-operation"></a>Operation

* [operation.schema.json](./schema/operation.schema.json)

Name | Type | Format | Description
-----|------|--------|------------
`rel`|`string`|[RFC 5988 Relation Type](https://tools.ietf.org/html/rfc5988#section-4)|**Required** Relation type as defined by the server. There are registered relation types listed in [RFC 5988 6.2.2. Initial Registry Contents](https://tools.ietf.org/html/rfc5988#section-6.2.2) including pagination relation types of `next`, `prev`, `first` and `last`.
`href`|`string`|[RFC 3986 URI](https://www.ietf.org/rfc/rfc3986.txt)|**Required** Hyperlink to the resource. This key name is borrowed from the HTML `href` element and behaves similarly.
`method`|`string`|[RFC 7231 Methods](https://tools.ietf.org/html/rfc7231#section-4.3)|Default is GET -  (GET, HEAD, POST, PUT, DELETE, CONNECT, OPTIONS, TRACE) + RFC 5789 PATCH method.

###<a name="schema-store"></a>Store

* [store.v1.schema.json](./schema/store.v1.schema.json)
* Versioning in the schema name allows the service to revise schema with versioning.
	* Generally speaking, you can rather easily add new, non-required keys to existing schema without breaking clients as they should be coded in such a way as to ignore keys they don't understand. However, this type of change can be considered a breaking change.
	* This makes it relatively easy to introduce new schema features or breaking changes (such as new and required keys) without breaking existing clients.

Name | Type | Format | Description
-----|------|--------|------------
`schema`|`string`|[RFC 3986 URI](https://www.ietf.org/rfc/rfc3986.txt)|**Required** - Hyperlink to the schema describing this representation, from [base](#schema-base).
`name`|`string`|-|**Required** - Store name. Examples: `Piggly Wiggly`, `Safeway` and `Albertsons`.
`phone`|`string`|-|Phone number of the store.

###<a name="schema-aisle"></a>Aisle

* [aisle.v1.schema.json](./schema/aisle.v1.schema.json)
* See notes on versioning schema in the [store schema](#schema-store).

Name | Type | Format | Description
-----|------|--------|------------
`schema`|`string`|[RFC 3986 URI](https://www.ietf.org/rfc/rfc3986.txt)|**Required** - Hyperlink to the schema describing this representation, from [base](#schema-base).
`store`|`string`|[RFC 3986 URI](https://www.ietf.org/rfc/rfc3986.txt)|**Required** - Hyperlink to the store where this aisle is located.
`name`|`string`|-|**Required** - Aisle name. Examples: `Baking` and `Dairy`.
`number`|`integer`|-|Aisle number. Examples: `1` and `2`.

###<a name="schema-error"></a>Error

* [error.schema.json](./schema/error.schema.json)
* Included here for completeness of the resource model but not currently used in the examples.

Name | Type | Format | Description
-----|------|--------|------------
`errorCode`|`string`|-|**Required** Machine readable code associated with the error. Examples: `dateTimeMissing`, `OutOfMem`, `invalidUser`. Contextual strings are recommended over numbers or UUIDs.
`errorMessage`|`string`|-|**Required** Message associated with the error.
`dataPath`|`string`|-|Relative data path.
`schemaPath`|`string`|-|Relative schema path.
`errors`|`array`|[`error`](#schema-base-error)|An array of errors. Note: this points to this schema as errors can nest.

##<a name="service"></a>Service

###<a name="service-retrieve-index"></a>Retrieve the service index

* This is the only URI the client code must know about ahead of time.
* The llustration starts with no data.
* The example shows two different services (`stores` and `aisles`).
	* This is for human readability -- they could be the same exact service.
* There isn't an `id` or `template` here because they aren't needed for the index.
* The server can make additional data available here, like links to schema(s).

### Request

`GET https://example.com/stores`

### Response

```
200 OK
```

```json
{
	"href": "https://example.com/stores",
	"schema": "https://example.com/schemas/base.schema.json",
	"alternates": [
		{
			"rel": "all-stores",
			"href": "https://example.com/stores/all"
		}
	],
	"operations": [
		{
			"rel": "create-store",
			"href": "https://example.com/stores",
			"method": "POST"
		}
	]
}
```

###<a name="service-create-store"></a>Create a store

* Client code is creating the data.
* The URI for where to POST comes from the service index: `create-store`.
* The schema is defined by the server and is used by the client code to specify the data being sent by the client.
	* `schema` key comes from `base.schema.json`.
	* `name` and `phone` comes from `store.v1.schema.json`

### Request

`POST https://example.com/stores`

```json
{
	"schema": "https://example.com/schemas/store.v1.schema.json",
	"name": "Alpha",
	"phone": "(425) 555-1212"
}
```

### Response

* Note the additional metadata leveraging `base.schema.json`: `href`, `id` and `template`.
* The client is given the hyperllink for an `operation` on the store for adding an aisle.
	* This hyperlink is at the heart of Hypermedia as the Engine of Application State.
	* `operations` is part of `base.schema.json` and defined in `operation.schema.json`.

```
201 Created
Location: https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b
```

```json
{
	"schema": "https://example.com/schemas/store.v1.schema.json",
	"name": "Alpha",
	"phone": "(425) 555-1212",
	"href": "https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b",
	"id": "97b83a73-5620-465c-b8a0-1bf82392336b",
	"template": "https://example.com/stores/{id}",
	"operations": [
		{
			"rel": "add-aisle",
			"href": "https://example.com/aisles",
			"method": "POST"
		}
	]
}
```

###<a name="service-create-aisles"></a>Create some aisles

* Relationships between resources are established.
* The URI to accomplish the task was provided by the response to creating a store with `"rel": "add-aisle"`.
* The response contains an operation link to update the aisle data using a `POST`.

### Request

`POST https://example.com/aisles`

```json
{
	"schema": "https://example.com/schemas/aisle.v1.schema.json",
	"store": "https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b",
	"name": "Dairy",
	"number": "10"
}
```

### Response

```
201 Created
Location: https://example.com/aisles/b29e4945-2ebf-4625-b44d-e3034ad99e4d
```

```json
{
	"schema": "https://example.com/schemas/aisle.v1.schema.json",
	"store": "https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b",
	"name": "Dairy",
	"number": "10",
	"href": "https://example.com/aisles/b29e4945-2ebf-4625-b44d-e3034ad99e4d",
	"id": "b29e4945-2ebf-4625-b44d-e3034ad99e4d",
	"template": "https://example.com/aisles/{id}",
	"operations": [
		{
			"rel": "update-aisle",
			"href": "https://example.com/aisles/b29e4945-2ebf-4625-b44d-e3034ad99e4d",
			"method": "POST"
		}
	]
}
```

### Request

`POST https://example.com/aisles`

```json
{
	"schema": "https://example.com/schemas/aisle.v1.schema.json",
	"store": "https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b",
	"name": "Baking",
	"number": "11"
}
```

### Response

```
201 Created
Location: https://example.com/aisles/a256aabd-e988-4b37-9f1f-99222ebdfe3d
```

```json
{
	"schema": "https://example.com/schemas/aisle.v1.schema.json",
	"store": "https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b",
	"name": "Baking",
	"number": "11",
	"href": "https://example.com/aisles/a256aabd-e988-4b37-9f1f-99222ebdfe3d",
	"id": "a256aabd-e988-4b37-9f1f-99222ebdfe3d",
	"template": "https://example.com/aisles/{id}",
	"operations": [
		{
			"rel": "update-aisle",
			"href": "https://example.com/aisles/a256aabd-e988-4b37-9f1f-99222ebdfe3d",
			"method": "POST"
		}
	]
}
```

###<a name="service-retrieve-store"></a>Retrieve the store

* Using the value in the `Location` header or the data in the response payload from creating a store perform a `GET` operation.
* The server has made additional `alternates` available because now there are aisles:
	* GET the store with a list of aisles using `"rel": "store-aisle-list"`
	* GET the store with aisles data using `"rel": "store-aisle-data"`.
	* Not listed is the relation type for the store itself (because that's what we have here).

### Request

`GET https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b`

### Response

```
200 OK
```

```json
{
	"schema": "https://example.com/schemas/store.v1.schema.json",
	"name": "Alpha",
	"phone": "(425) 555-1212",
	"href": "https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b",
	"id": "a",
	"template": "https://example.com/stores/{id}",
	"alternates": [
		{
			"rel": "store-aisle-list",
			"href": "https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b/aislelist"
		},
		{
			"rel": "store-aisle-data",
			"href": "https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b/aisledata"
		}
	],
	"operations": [
		{
			"rel": "add-aisle",
			"href": "https://example.com/aisles",
			"method": "POST"
		}
	]
}
```

###<a name="service-retrieve-store-aisle-list"></a>Get a store and list of aisles

* This representation can be helpful if the related data for each item in the array is really large, infrequently accessed or accessed individually.
* This representation may also help mobile clients potentially keep data usage levels lower.
* Note the change within `alternates`:
	* GET the store using `"rel": "store"`.
	* GET the store with aisles data using `"rel": "store-aisle-data"`.
	* There is no store with list of aisles (because that is the representation we now have).

### Request

`GET https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b/aislelist`

### Response

```
200 OK
```

```json
{
	"schema": "https://example.com/schemas/store.v1.schema.json",
	"name": "Alpha",
	"phone": "(425) 555-1212",
	"href": "https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b/aislelist",
	"id": "97b83a73-5620-465c-b8a0-1bf82392336b",
	"template": "https://example.com/stores/{id}/aislelist",
	"lists": [
		{
			"schema": "https://example.com/schemas/aisle.v1.schema.json",
			"listItems": [
				"https://example.com/aisles/b29e4945-2ebf-4625-b44d-e3034ad99e4d",
				"https://example.com/aisles/a256aabd-e988-4b37-9f1f-99222ebdfe3d"
			]
		}
	],
	"alternates": [
		{
			"rel": "store",
			"href": "https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b"
		},
		{
			"rel": "store-aisle-data",
			"href": "https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b/aisledata"
		}
	],
	"operations": [
		{
			"rel": "add-aisle",
			"href": "https://example.com/aisles",
			"method": "POST"
		}
	]
}
```

###<a name="service-retrieve-store-aisle-data"></a>Get a store with aisles data

* Note the full aisle data is included inline with the stores, complete with operations.
* Note the change within `alternates`:
	* GET the store using `"rel": "store"`.
	* GET the store with a list of aisles using `"rel": "store-aisle-list"`
	* There is no store with aisles data (because that is the representation we now have).

### Request

`GET https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b/aisledata`

```json
{
	"schema": "https://example.com/schemas/store.v1.schema.json",
	"name": "Alpha",
	"phone": "(425) 555-1212",
	"href": "https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b/aisledata",
	"id": "97b83a73-5620-465c-b8a0-1bf82392336b",
	"template": "https://example.com/stores/{id}/aisledata",
	"data": [
		{
			"schema": "https://example.com/schemas/aisles.v1.schema.json",
			"store": "https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b",
			"name": "Dairy",
			"number": "10",
			"href": "https://example.com/aisles/b29e4945-2ebf-4625-b44d-e3034ad99e4d",
			"id": "b29e4945-2ebf-4625-b44d-e3034ad99e4d",
			"template": "https://example.com/aisles/{id}",
			"operations": [
				{
					"rel": "update",
					"href": "https://example.com/aisles/b29e4945-2ebf-4625-b44d-e3034ad99e4d",
					"method": "PUT"
				}
			]
		},
		{
			"schema": "https://example.com/schemas/aisles.v1.schema.json",
			"store": "https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b",
			"name": "Baking",
			"number": "11",
			"href": "https://example.com/aisles/a256aabd-e988-4b37-9f1f-99222ebdfe3d",
			"id": "a256aabd-e988-4b37-9f1f-99222ebdfe3d",
			"template": "https://example.com/aisles/{id}",
			"operations": [
				{
					"rel": "update",
					"href": "https://example.com/aisles/a256aabd-e988-4b37-9f1f-99222ebdfe3d",
					"method": "PUT"
				}
			]
		}
	],
	"alternates": [
		{
			"rel": "store",
			"href": "https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b"
		},
		{
			"rel": "store-aisle-list",
			"href": "https://example.com/stores/97b83a73-5620-465c-b8a0-1bf82392336b/aislelist"
		}
	],
  "operations": [
    {
      "rel": "add-aisle",
      "href": "https://example.com/aisles",
      "method": "POST"
    }
  ]
}
```
