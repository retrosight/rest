# Hypermedia as the Engine of Application State Model Example

> Based upon an original flowchart by Stefan Tilkov https://twitter.com/stilkov

* Consider this model of an product order going through a system.
* The states shown are server state of the resource (an order): `received`, `accepted`, `rejected`, `cancelled`, `fulfilled`.

```
                                                                      -------------
            update          cancel                                    |           |
           -----------|   |-----------------------------------------> | cancelled | --------------|
           |          |   |                       |                   |           |               |
           |          |   |                       |                   -------------               |
           |          |   |                       |                                               |
           |      -------------             -------------             -------------               |
           V      |           |  acccept    |           |  ship       |           |               V
start ----------> | received  | ----------> | accepted  | ----------> | fulfilled | ---------> finish
                  |           |             |           |             |           |               ^
                  -------------             -------------             -------------               |
                          |                                                                       |
                          |                 -------------                                         |
                          |                 |           |                                         |
                          |---------------> | rejected  |-----------------------------------------|
                            reject          |           |
                                            -------------
```

## Start

* For the example the client code is authorized for all operations.
* Client code gets the service index URI which it knows from documentation.
* The examples which follow do not show the `href` values for any other operations because:
  * The server is always in control of building the entire URI.
  * URIs are completely opaque strings to the client code.
  * Client code follows links in `href` as the result of binding to the `rel` value.
* For completeness an endpoint for `"rel": "orders-all"` is provided to demonstrate the difference between the service itself and resources contained therein.

### Request

`GET https://example.com/orders`

### Response

```
200 OK
```

```json
{
	"operations": [
		{
			"rel": "order-post", "href": "https://...", "method": "post"
		},
    {
      "rel": "orders-all", "href": "https://..."
    }
	]
}
```

## State: `received`

* Client code extracts the `href` for the `order-post` link relation.
* Client builds the payload (aka data, representation).
* Client makes the call to `POST` the representation (an order).
* The server responds back to the client including the operations available from this state.
  * It would be acceptable for the server to also have a key value pair indicating the state of the resource on the server but it is not necessary.

### Request

`POST https://...`

```json
{
  "product": "widget",
  "quantity": 100
}
```

### Response

```
201 Created
Location: https://...
```

```json
{
  "product": "widget",
  "quantity": 100,
  "id": "https://...",
  "operations": [
    { "rel": "order-cancel", "href": "https://...", "method": "delete" },
    { "rel": "order-accept", "href": "https://...", "method": "post" },
    { "rel": "order-reject", "href": "https://...", "method": "post" },
    { "rel": "order-update", "href": "https://...", "method": "post" }
  ]
}
```

## State: `accepted`

* Time passes and order processing occurs which moves the resource state from `received` to `accepted`.
* The client code wishes to understand the current state of the order.
* The list of `operations` is smaller because the order is in an `accepted` state.

### Request

`GET https://.../order123`

### Response

```
200 OK
```

```json
{
  "product": "widget",
  "quantity": 100,
  "id": "https://...",
  "operations": [
    { "rel": "order-cancel", "href": "https://...", "method": "delete" },
    { "rel": "order-ship", "href": "https://...", "method": "post" }
  ]
}
```
