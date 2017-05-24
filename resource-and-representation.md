# Resources and Representations

There are a number of challenges when there are large resource definitions and a polyglot of microservices in terms of making all of the data (to include related data) available as needed by client code.

The following design and examples are intended to provide a starting point for services as they investigate data design and end points for client code.

* [Schema](#schema)
	* [Base](#schema-base)
	* [Links](#schema-links)
	* [Operation](#schema-operation)
	* [Store](#schema-store)
	* [Aisle](#schema-aisle)
	* [Error](#schema-error)
* [Service](#service)
	* [Retrieve the service index](#service-retrieve-index)
	* [Create a store](#service-create-store)
	* [Create some aisles](#service-create-aisles)
	* [Retrieve the store](#service-retrieve-store)
	* [Retrieve a store and list of links to aisles](#service-retrieve-store-aisle-data-links)
	* [Retrieve a store with aisles data](#service-retrieve-store-aisle-data)

## <a name="schema"></a>Schema

* This overall data model describes stores and their aisles.

***

### <a name="schema-base"></a>Base

* [com-example-base-v1](./schema/com-example-base-v1.schema.json)
* The base schema provides a starting point for all representations.
* It is the default schema for all representations.
* It is the schema used for:
  * Metadata
  * Hypermedia as the Engine of Application State (HATEOAS) operations
  * Related data
  * Related data links
  * Alternative representations
  * Conveying errors (should they occur).

Name | Type | Format | Description
-----|------|--------|------------
`schema`|`string`|[RFC 3986 URI](https://www.ietf.org/rfc/rfc3986.txt)|**Required** Hyperlink to the schema describing this resource to fulfill the self-describing portion of the uniform interface.
`href`|`string`|[RFC 3986 URI](https://www.ietf.org/rfc/rfc3986.txt)|Hyperlink to this representation, matches the URI the client code used.
`id`|`string`|-|For relational database systems to save the identifier separate from the URI template. When combined with the `template` key results in the `href` key value.
`template`|`string`|[URI Template](https://tools.ietf.org/html/rfc6570)|For relational database systems to save the template separate from the identifier.
`relatedData`|`array`|-|Inline representations of related data.
`relatedDataLinks`|`array`|[`links`](#schema-links)|List of hyperlinks to related data representations. There can be multiple lists, each with their own schema.
`alternates`|`array`|[`operation`](#schema-base-operation)|Alternate representations of the same resource.
`operations`|`array`|[`operation`](#schema-base-operation)|The hypermedia as the engine of application state (HATEOAS) information.
`errors`|`array`|[`error`](#schema-base-error)|The errors collection for when the response code is 4xx.

***

### <a name="schema-links"></a>Links

* [com-example-links-v1](./schema/com-example-links-v1.schema.json)
* A list of links allows related data to be provided to the client which can optionally `GET` each item as so desired.

Name | Type | Format | Description
-----|------|--------|------------
`schema`|`string`|-|**Required** The schema used to describe all of the items in the array of `hrefs`. From [base](#schema-base).
`hrefs`|`array` of `string`|[RFC 3986 URI](https://www.ietf.org/rfc/rfc3986.txt)|**Required** Hyperlinks to the related representations which all match the `schema`.

***

### <a name="schema-operation"></a>Operation

* [com-example-operation-v1](./schema/com-example-operation-v1.schema.json)

Name | Type | Format | Description
-----|------|--------|------------
`rel`|`string`|[RFC 5988 Relation Type](https://tools.ietf.org/html/rfc5988#section-4)|**Required** Relation type as defined by the server. There are registered relation types listed in [RFC 5988 6.2.2. Initial Registry Contents](https://tools.ietf.org/html/rfc5988#section-6.2.2) including pagination relation types of `next`, `prev`, `first` and `last`.
`href`|`string`|[RFC 3986 URI](https://www.ietf.org/rfc/rfc3986.txt)|**Required** Hyperlink to the resource. This key name is borrowed from the HTML `href` element and behaves similarly.
`method`|`string`|[RFC 7231 Methods](https://tools.ietf.org/html/rfc7231#section-4.3)|Default is GET -  (GET, HEAD, POST, PUT, DELETE, CONNECT, OPTIONS, TRACE) + RFC 5789 PATCH method.

***

### <a name="schema-store"></a>Store

* [com-example-store-v1](./schema/com-example-store-v1.schema.json)
* Versioning in the schema name allows the service to revise schema with versioning.
	* Generally speaking, you can rather easily add new, non-required keys to existing schema without breaking clients as they should be coded in such a way as to ignore keys they don't understand. However, this type of change can be considered a breaking change.
	* This makes it relatively easy to introduce new schema features or breaking changes (such as new and required keys) without breaking existing clients.

Name | Type | Format | Description
-----|------|--------|------------
`schema`|`string`|[RFC 3986 URI](https://www.ietf.org/rfc/rfc3986.txt)|**Required** - Hyperlink to the schema describing this representation. From [base](#schema-base).
`storeName`|`string`|-|**Required** - Store name. Examples: `Piggly Wiggly`, `Safeway` and `Albertsons`.
`storePhone`|`string`|-|Phone number of the store.

***

### <a name="schema-aisle"></a>Aisle

* [com-example-aisle-v1](./schema/com-example-aisle-v1.schema.json)
* See notes on versioning schema in the [store schema](#schema-store).

Name | Type | Format | Description
-----|------|--------|------------
`schema`|`string`|[RFC 3986 URI](https://www.ietf.org/rfc/rfc3986.txt)|**Required** - Hyperlink to the schema describing this representation, from [base](#schema-base).
`storeId`|`string`|[RFC 3986 URI](https://www.ietf.org/rfc/rfc3986.txt)|**Required** - Hyperlink to the store where this aisle is located.
`aisleName`|`string`|-|**Required** - Aisle name. Examples: `Baking` and `Dairy`.
`aisleNumber`|`integer`|-|Aisle number. Examples: `1` and `2`.

***

### <a name="schema-error"></a>Error

* [com-example-error-v1](./schema/com-example-error-v1.schema.json)
* Included here for completeness of the resource model but not currently used in the examples.

Name | Type | Format | Description
-----|------|--------|------------
`errorCode`|`string`|-|**Required** Machine readable code associated with the error. Examples: `dateTimeMissing`, `OutOfMem`, `invalidUser`. Contextual strings are recommended over numbers or UUIDs.
`errorMessage`|`string`|-|**Required** Message associated with the error.
`dataPath`|`string`|-|Relative data path.
`schemaPath`|`string`|-|Relative schema path.
`errors`|`array`|[`error`](#schema-base-error)|An array of errors. Note: this points to this schema as errors can nest.

***

## <a name="service"></a>Service

### <a name="service-retrieve-index"></a>Retrieve the service index

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
	"schema": "https://example.com/schemas/com-example-base-v1",
	"operations": [
    {
      "rel": "all-stores",
      "href": "https://example.com/stores/all"
    },
		{
			"rel": "create-store",
			"href": "https://example.com/stores",
			"method": "POST"
		}
	]
}
```

***

### <a name="service-create-store"></a>Create a store

* Client code is creating the data.
* The URI for where to POST comes from the service index: `create-store`.
* The schema is defined by the server and is used by the client code to specify the data being sent by the client.
	* `schema` key comes from `com-example-base-v1`.
	* `storeName` and `storePhone` comes from `com-example-store-v1`.

### Request

`POST https://example.com/stores`

```json
{
	"schema": "https://example.com/schemas/com-example-store-v1",
	"storeName": "Alpha",
	"storePhone": "(425) 555-1212"
}
```

### Response

* Note the additional metadata leveraging `com-example-base-v1`: `href`, `id` and `template`.
* The client is given the hyperlink for an `operation` on the store for adding an aisle.
	* This hyperlink is at the heart of Hypermedia as the Engine of Application State.
	* `operations` is part of `com-example-base-v1` and defined in `com-example-operation-v1`.
* The `POST` occurs on the collection (plural: `stores`).
* The location of a store is individual (singular: `store`).

```
201 Created
Location: https://example.com/store/97b83a735620465cb8a01bf82392336b
```

```json
{
	"schema": "https://example.com/schemas/com-example-store-v1",
	"storeName": "Alpha",
	"storePhone": "(425) 555-1212",
	"href": "https://example.com/store/97b83a735620465cb8a01bf82392336b",
	"id": "97b83a735620465cb8a01bf82392336b",
	"template": "https://example.com/store/{id}",
	"operations": [
		{
			"rel": "add-aisle",
			"href": "https://example.com/aisles",
			"method": "POST"
		}
	]
}
```

***

### <a name="service-create-aisles"></a>Create some aisles

* Relationships between resources are established.
* The URI to accomplish the task was provided by the response to creating a store with `"rel": "add-aisle"`.
* The response contains an operation link to update the aisle data using a `POST`.
* Same collection (plural) and item (singluar) pattern as a store.

### Request

`POST https://example.com/aisles`

```json
{
	"schema": "https://example.com/schemas/com-example-aisle-v1",
	"storeId": "https://example.com/store/97b83a735620465cb8a01bf82392336b",
	"aisleName": "Dairy",
	"aisleNumber": 10
}
```

### Response

```
201 Created
Location: https://example.com/aisle/b29e49452ebf4625b44de3034ad99e4d
```

```json
{
	"schema": "https://example.com/schemas/com-example-aisle-v1",
	"storeId": "https://example.com/store/97b83a735620465cb8a01bf82392336b",
	"aisleName": "Dairy",
	"aisleNumber": 10,
	"href": "https://example.com/aisle/b29e49452ebf4625b44de3034ad99e4d",
	"id": "b29e49452ebf4625b44de3034ad99e4d",
	"template": "https://example.com/aisle/{id}",
	"operations": [
		{
			"rel": "update-aisle",
			"href": "https://example.com/aisle/b29e49452ebf4625b44de3034ad99e4d",
			"method": "POST"
		}
	]
}
```

### Request

`POST https://example.com/aisles`

```json
{
	"schema": "https://example.com/schemas/com-example-aisle-v1",
	"storeId": "https://example.com/store/97b83a735620465cb8a01bf82392336b",
	"aisleName": "Baking",
	"aisleNumber": 11
}
```

### Response

```
201 Created
Location: https://example.com/aisle/a256aabde9884b379f1f99222ebdfe3d
```

```json
{
	"schema": "https://example.com/schemas/com-example-aisle-v1",
	"storeId": "https://example.com/store/97b83a735620465cb8a01bf82392336b",
	"aisleName": "Baking",
	"aisleNumber": 11,
	"href": "https://example.com/aisle/a256aabde9884b379f1f99222ebdfe3d",
	"id": "a256aabde9884b379f1f99222ebdfe3d",
	"template": "https://example.com/aisle/{id}",
	"operations": [
		{
			"rel": "update-aisle",
			"href": "https://example.com/aisle/a256aabde9884b379f1f99222ebdfe3d",
			"method": "POST"
		}
	]
}
```

***

### <a name="service-retrieve-store"></a>Retrieve the store

* Using the value in the `Location` header or the data in the response payload from creating a store perform a `GET` operation.
* The server has made additional `alternates` available because now there are aisles:
	* GET the store with a list of links to aisles using `"rel": "store-aisle-data-links"`
	* GET the store with aisles data using `"rel": "store-aisle-data"`.
	* Not listed is the relation type for the store itself (because that's what we have here).

### Request

`GET https://example.com/store/97b83a735620465cb8a01bf82392336b`

### Response

```
200 OK
```

```json
{
	"schema": "https://example.com/schemas/com-example-store-v1",
	"storeName": "Alpha",
	"storePhone": "(425) 555-1212",
	"href": "https://example.com/store/97b83a735620465cb8a01bf82392336b",
	"id": "97b83a735620465cb8a01bf82392336b",
	"template": "https://example.com/store/{id}",
	"alternates": [
		{
			"rel": "store-aisle-data-links",
			"href": "https://example.com/store/97b83a735620465cb8a01bf82392336b/aisledatalinks"
		},
		{
			"rel": "store-aisle-data",
			"href": "https://example.com/store/97b83a735620465cb8a01bf82392336b/aisledata"
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

***

### <a name="service-retrieve-store-aisle-data-links"></a>Retrieve a store and list of links to aisles

* This representation can be helpful if the related data for each item in the array is really large, infrequently accessed or accessed individually.
* This representation may also help mobile clients potentially keep data usage levels lower.
* Note the change within `alternates`:
	* GET the store using `"rel": "store"`.
	* GET the store with aisles data using `"rel": "store-aisle-data"`.
	* There is no store with list of links to aisles (because that is the representation we now have).

### Request

`GET https://example.com/store/97b83a735620465cb8a01bf82392336b/aisledatalinks`

### Response

```
200 OK
```

```json
{
	"schema": "https://example.com/schemas/com-example-store-v1",
	"storeName": "Alpha",
	"storePhone": "(425) 555-1212",
	"href": "https://example.com/store/97b83a735620465cb8a01bf82392336b/aisledatalinks",
	"id": "97b83a735620465cb8a01bf82392336b",
	"template": "https://example.com/store/{id}/aisledatalinks",
	"relatedDataLinks": [
		{
			"schema": "https://example.com/schemas/com-example-aisle-v1",
			"hrefs": [
				"https://example.com/aisle/b29e49452ebf4625b44de3034ad99e4d",
				"https://example.com/aisle/a256aabde9884b379f1f99222ebdfe3d"
			]
		}
	],
	"alternates": [
		{
			"rel": "store",
			"href": "https://example.com/store/97b83a735620465cb8a01bf82392336b"
		},
		{
			"rel": "store-aisle-data",
			"href": "https://example.com/store/97b83a735620465cb8a01bf82392336b/aisledata"
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

***

### <a name="service-retrieve-store-aisle-data"></a>Retrieve a store with aisles data

* Note the full aisle data is included inline with the stores, complete with operations.
* Note the change within `alternates`:
	* GET the store using `"rel": "store"`.
	* GET the store with a list of links to aisles using `"rel": "store-aisle-data-links"`
	* There is no store with aisles data (because that is the representation we now have).

### Request

`GET https://example.com/store/97b83a735620465cb8a01bf82392336b/aisledata`

```json
{
	"schema": "https://example.com/schemas/com-example-store-v1",
	"storeName": "Alpha",
	"storePhone": "(425) 555-1212",
	"href": "https://example.com/store/97b83a735620465cb8a01bf82392336b/aisledata",
	"id": "97b83a735620465cb8a01bf82392336b",
	"template": "https://example.com/store/{id}/aisledata",
	"relatedData": [
		{
			"schema": "https://example.com/schemas/com-example-aisle-v1",
			"storeId": "https://example.com/store/97b83a735620465cb8a01bf82392336b",
			"aisleName": "Dairy",
			"aisleNumber": 10,
			"href": "https://example.com/aisle/b29e49452ebf4625b44de3034ad99e4d",
			"id": "b29e49452ebf4625b44de3034ad99e4d",
			"template": "https://example.com/aisle/{id}",
			"operations": [
				{
					"rel": "update",
					"href": "https://example.com/aisle/b29e49452ebf4625b44de3034ad99e4d",
					"method": "PUT"
				}
			]
		},
		{
			"schema": "https://example.com/schemas/com-example-aisle-v1",
			"storeId": "https://example.com/store/97b83a735620465cb8a01bf82392336b",
			"aisleName": "Baking",
			"aisleNumber": 11,
			"href": "https://example.com/aisle/a256aabde9884b379f1f99222ebdfe3d",
			"id": "a256aabde9884b379f1f99222ebdfe3d",
			"template": "https://example.com/aisle/{id}",
			"operations": [
				{
					"rel": "update",
					"href": "https://example.com/aisle/a256aabde9884b379f1f99222ebdfe3d",
					"method": "PUT"
				}
			]
		}
	],
	"alternates": [
		{
			"rel": "store",
			"href": "https://example.com/store/97b83a735620465cb8a01bf82392336b"
		},
		{
			"rel": "store-aisle-data-links",
			"href": "https://example.com/store/97b83a735620465cb8a01bf82392336b/aisledatalinks"
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
***
