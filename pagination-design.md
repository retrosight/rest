#Pagination Design

##Allowing a client to determine server behavior undermines the independent evolvability paradigm

For example, let us examine a server accepting a `pageSize` query parameter where the maximum size is 100.

```
http://example.com/stores?pageSize=100
```

* Should the server later determine for optimization purposes the number should be no greater than 50 it is a breaking change.
* Should the client code enter a value outside the `pageSize` parameter maximum (example" `http://example.com/stores?pageSize=1000`) the server has to write code to handle the out of bounds value.
* Should the server leave the `pageSize` parameter unbounded (allowing for example `http://example.com/stores?pageSize=1000000000000`) opens up holes to get all of the data at one time, which could be a potential DDOS or at the least a data leak issue).

This paradigm breaks the client-server constraint of the REST model by allowing the client to control server behavior.

##Leveraging Web Linking

The pagination approach leverages the relation names for navigating an ordered series of resources from [RFC 5988 Web Linking](https://tools.ietf.org/html/rfc5988) which puts the server in charge (always) of how the data is offered to the client code and client code can maintain a durable cursor for itself to page through that data, following the provided hyperlinks to do so.

Following the prior example of a `pageSize` parameter:

* The client code doesn't have to worry about crafting URLs to get pages of data -- it follows links given to it by the server.
* The server can change the page size from 50 to 100 then to 10 and client code happily keeps working.
* The client code cannot specify server behavior which means the server and client can both evolve independently.

##Pagination in `Link` Header vs. JSON data
One of the features of [RFC 5988 Web Linking](https://tools.ietf.org/html/rfc5988) is the defined [`Link`](https://tools.ietf.org/html/rfc5988#section-5) header field which allows for the creation of pagination links via HTTP header as in this example:

```
Link: <https://example.com/people/page4>; rel="next",
	  <https://example.com/people/page2>; rel="prev",
	  <https://example.com/people/first>; rel="first",
	  <https://example.com/people/last>; rel="last";
```

The reason the guidelines place the links in the payload body is:

* So there can be multiple paged results within a single representation. Using the [`Link`](https://tools.ietf.org/html/rfc5988#section-5) header field means:
	* The server is limited to one set of links per payload body OR
	* The server has to mint disambiguated relation types OR
	* The server has to mint additional schemas to account for nested data.
	* See next section.
* It is generally easier for client code to bind to the JSON in the payload body (lots if not all of the JSON oriented libraries do so) rather than parse out headers.
* [RFC 5988 Web Linking](https://tools.ietf.org/html/rfc5988) does not require the Link header to be used for all links, from the abstract: `"This document specifies relation types for Web links, and defines a registry for them. It also defines the use of such links in HTTP headers with the Link header field."`
* Basically, the guidelines are using the relation types from this standard but not the `Link` header.

##Nested data with pagination example

* This is one paradigm that is difficult to achieve with pagination in the `Link` header.
* In the example the client can page through the `data` and the `relateddata`.

```json
{
	"data": [
		{
			"id": "Alpha",
			"relateddata": [
				{ "id": "Delta"	},
				{ "id": "Echo" },
				{ "id": "Foxtrot" }
			],
			"operations": [
				{
					"rel": "next",
					"href": "https://example.com/service/relateddata/page2"
				}
			]
		},
		{ "id": "Bravo" },
		{ "id": "Charlie" }
	],
	"operations": [
		{
			"rel": "next",
			"href": "https://example.com/service/data/page4"
		}
	]

}
```
