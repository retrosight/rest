# Search

Search is a powerful tool which helps client code find resources it is interested in when it does not have direct links to those resources but does have data that might be within those resources.

Excerpt from https://www.ietf.org/rfc/rfc3986.txt

```
The query component contains non-hierarchical data that, along with
data in the path component (Section 3.3), serves to identify a
resource within the scope of the URI's scheme and naming authority
(if any).
```

The phrase `serves to identify` from the RFC can be interpreted to mean it may or may not return only a single item -- hence search should always return an array.

Search is another collection within the service.

> This is how Google approaches search, as a collection: `https://www.google.com/search?q=api`

## Examples of search

Within all collections:

```
GET https:/example.com/search?product=milk
```

```
[
  "https://example.com/stores/76cc758e256c438b8e49546e0102b8c8",
  "https://example.com/stores/5ad81b6a25e347899c6335aa46fe1097",
  "https://example.com/stores/51873c3a2f654a2680fef0c3b926630d",
  "https://example.com/aisles/6fec299cafb942bb9dfcacb053cfedae",
  "https://example.com/aisles/745b5f0064db41018194f0621e14f86e",
  "https://example.com/products/5cd482b2a6cd4c13a9ed64310bbe961c",
]
```

Within a specific collection:

```
GET https:/example.com/search?product=milk&collection=stores
```

```
[
  "https://example.com/stores/76cc758e256c438b8e49546e0102b8c8",
  "https://example.com/stores/5ad81b6a25e347899c6335aa46fe1097",
  "https://example.com/stores/51873c3a2f654a2680fef0c3b926630d",
]
```

Having a collection for search also preserves the use of the query component within other URIs, for example:

```
Template: /{collection}  /{identifier}  ?{query}
URI:      /stores        /abc123def456  ?city=Seattle
```

---

```
/profile?type=companyid&name=208602     // identify within all of profile.
/profile/v1?type=companyid&name=208602  // identify within all of profile v1
/profile/v1/companies?name=208602       // identify within all of the companies collection within profile v1
```

You'll note the last one doesn't need `type=company` because presumably you are within the collection that only serves up a company. (edited)
In any event, I tend to think of any use of the `?` within the URI as a search which returns an array.
Even if it's an array of one.

```
[
  "/profile/e41ab369-f1ba-4790-b685-fc04cb3387c2"
]
```
