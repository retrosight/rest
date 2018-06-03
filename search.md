# Search

Search is a powerful tool which helps client code find resources it is interested in when it does not have direct links to those resources but does have data that might be within those resources.

Excerpt from https://www.ietf.org/rfc/rfc3986.txt

```
The query component contains non-hierarchical data that, along with
data in the path component (Section 3.3), serves to identify a
resource within the scope of the URI's scheme and naming authority
(if any).
```

The phrase `serves to identify` from the RFC is a bit ambiguous and can be interpreted to mean it may or may not return only a single item -- hence search should always return an array.

Search is another resource within the service.

> This is how Google formats the search URI: `https://www.google.com/search?q=api`

## Examples of search

### Within all resources

```
GET https:/example.com/search?product=milk
```

```json
{
  "schema": "http://example.com/schema/com-example-search-results-2018-03-01.schema.json",
  "href": "https:/example.com/search?product=milk",
  "id": "product=milk",
  "template": "https:/example.com/search?{id}",
  "results": [
    {
      "schema": "https://example.com/schemas/com-example-store-2018-03-01.schema.json",
      "hrefs":
      [
        "https://example.com/stores/76cc758e256c438b8e49546e0102b8c8",
        "https://example.com/stores/5ad81b6a25e347899c6335aa46fe1097",
        "https://example.com/stores/51873c3a2f654a2680fef0c3b926630d"
      ]
    },
    {
      "schema": "https://example.com/schemas/com-example-aisle-2018-03-01.schema.json",
      "hrefs":
      [
        "https://example.com/aisles/6fec299cafb942bb9dfcacb053cfedae",
        "https://example.com/aisles/745b5f0064db41018194f0621e14f86e"
      ]
    },
    {
      "schema": "https://example.com/schemas/com-example-product-2018-03-01.schema.json",
      "hrefs":
      [
        "https://example.com/products/5cd482b2a6cd4c13a9ed64310bbe961c"
      ]
    }
  ]
}
```

### Within a specific set of resources:

```
GET https:/example.com/search?product=milk&resources=stores
```

```json
{
  "schema": "http://example.com/schema/com-example-search-results-2018-03-01.schema.json",
  "href": "https:/example.com/search?product=milk&resources=stores",
  "id": "product=milk&resources=stores",
  "template": "https:/example.com/search?{id}",
  "results": [
    {
      "schema": "https://example.com/schemas/com-example-store-2018-03-01.schema.json",
      "hrefs":
      [
        "https://example.com/stores/76cc758e256c438b8e49546e0102b8c8",
        "https://example.com/stores/5ad81b6a25e347899c6335aa46fe1097",
        "https://example.com/stores/51873c3a2f654a2680fef0c3b926630d"
      ]
    }
  ]
}
```

### Parent + child search results

In this scenario there are results that are related in a hierarchal relationship where the search term isn't in the child but in the parent and is related via a hyperlink.

Note the use of a different schema at the top level: `search-results-nested`.

```
GET https:/example.com/search?product=milk
```

```json
{
  "schema": "https://example.com/schemas/com-example-search-results-nested-2018-03-01.schema.json",
  "href": "https:/example.com/search?product=milk",
  "id": "product=milk",
  "template": "https:/example.com/search?{id}",
  "results": [
    {
      "schema": "https://example.com/schemas/com-example-store-2018-03-01.schema.json",
      "hrefs": [
        {
          "href": "https://example.com/stores/76cc758e256c438b8e49546e0102b8c8",
          "results": [
            {
              "schema": "https://example.com/schemas/com-example-aisle-2018-03-01.schema.json",
              "hrefs": [
                {
                  "href": "https://example.com/aisles/6fec299cafb942bb9dfcacb053cfedae"
                },
                {
                  "href": "https://example.com/aisles/745b5f0064db41018194f0621e14f86e"
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

## Preserving use of query in other resources

Having a resource for search also preserves the use of the query component within other URIs, for example:

```
Template: /{resource}/{identifier}?{query}
URI:      /stores    /abc123de    ?city=Seattle
```

## Compute vs. storage in search

Modeling search as a resource abstracts away the actual implementation in such a way it can be static (using storage) or dynamic (using compute).

Using a static approach means the search results resource will get updated each time there is an operation which changes a resource which will affect the search result.

Modeling search as a resource also allows it to fully participate as any other resource, including in the service index.

```
title Search Authoring - Resource Exists
person->search: GET /search?nameFirst=Alpha
search->person: 200 OK [results]
person->person: Edit [results]
person->search: PUT /search?nameFirst=Alpha
search->person: 200 OK [results]
```

```
title Search Authoring - Resource Does Not Exist
person->search: GET /search?nameFirst=Alpha
search->person: 404 NOT FOUND
person->person: Create [results]
person->search: PUT /search?nameFirst=Alpha [results]
search->person: 200 OK [results]
```

**Example**

**NOT YET DONE**

```
POST https://example.com/api
```

```json
{
  "schema": "http://example.com/schema/person.schema.json",
  "nameFirst": "Alpha",
  "nameLast": "Bravo"
}
```

```
201 Created
```

```json
{
  "schema": "http://example.com/schema/person.schema.json",
  "nameFirst": "Alpha",
  "nameLast": "Bravo",
  "href": "https://example.com/api/1234",
  "id": "1234",
  "template": "https://example.com/api/{id}"
}
```
Upon completion of the post the search results resource is written as follows by the service:

```
GET https://example.com/api/search?nameFirst=Alpha
```


When both of those are POSTed the search results are written as follows;

```
GET https://example.com/api/search?nameFirst=Alpha
```

```json
{
  "schema": "http://example.com/schema/com-example-search-results-2018-03-01.schema.json",
  "href": "https://example.com/api/search?nameFirst=Alpha",
  "id": "nameFirst=Alpha",
  "template": "https://example.com/api/search?{id}",
  "results": [
    {
      "schema": "http://example.com/schema/person.schema.json",
      "hrefs":
      [
        "https://example.com/api/1234"
      ]
    }
  ]
}
```

```
GET https://example.com/api/5678
```

```json
{
  "schema": "http://example.com/schema/person.schema.json",
  "nameLast": "Delta"
}
```
