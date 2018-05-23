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

Within all resources:

```
// TODO:
// Add search schema.
// Add product schema.
```

```
GET https:/example.com/search?product=milk
```

```json
{
  "schema": "http://example.com/schema/com-example-search-2018-03-01.schema.json",
  "href": "https:/example.com/search?product=milk",
  "id": "product=milk",
  "template": "https:/example.com/search?{id}",
  "results": [
    {
      "schema": "https://example.com/schemas/com-example-store-2018-03-01.schema.json",
      "items":
      [
        "https://example.com/stores/76cc758e256c438b8e49546e0102b8c8",
        "https://example.com/stores/5ad81b6a25e347899c6335aa46fe1097",
        "https://example.com/stores/51873c3a2f654a2680fef0c3b926630d"
      ]
    },
    {
      "schema": "https://example.com/schemas/com-example-aisle-2018-03-01.schema.json",
      "items":
      [
        "https://example.com/aisles/6fec299cafb942bb9dfcacb053cfedae",
        "https://example.com/aisles/745b5f0064db41018194f0621e14f86e"
      ]
    },
    {
      "schema": "https://example.com/schemas/com-example-product-2018-03-01.schema.json",
      "items":
      [
        "https://example.com/products/5cd482b2a6cd4c13a9ed64310bbe961c"
      ]
    }
  ]
}
```

Within a specific set of resources:

```
GET https:/example.com/search?product=milk&resources=stores
```

```json
[
  "https://example.com/stores/76cc758e256c438b8e49546e0102b8c8",
  "https://example.com/stores/5ad81b6a25e347899c6335aa46fe1097",
  "https://example.com/stores/51873c3a2f654a2680fef0c3b926630d",
]
```

Having a resource for search also preserves the use of the query component within other URIs, for example:

```
Template: /{resource}/{identifier}?{query}
URI:      /stores    /abc123de    ?city=Seattle
```

## Compute vs. storage in search

Modeling search as a resource........
