##Identifier

* An identifier is part of the resource itself.
* The identifier is usually determined by the server with the usage of the HTTP POST method.
  * It can be determined by the client with the usage of the HTTP PUT method.

Here is an example of a POST to a people service. The service accepts and stores the data and responds with the data itself + metadata which is the identifier: `href`, `id` and `template`.

**Request**

POST https://example.com/people

```
{
  "nameFirst": "John",
  "nameLast": "Doe"
  "gender": "male",
  "colorHair": "brown",
  "colorEyes": "brown",
  "ssn": "123-45-6789"
}
```

**Response**

```
{
  "nameFirst": "John",
  "nameLast": "Doe",
  "gender": "male",
  "colorHair": "brown",
  "colorEyes": "brown",
  "ssn": "123-45-6789",

  // Identifier
  "href": "https://example.com/people/11fa754d-d5b8-45e3-a2e7-d821351dc345",
  "id": "11fa754d-d5b8-45e3-a2e7-d821351dc345",
  "template": "https://example.com/people/{id}"
}
```

##Related Data

* Related data are values which point to something that is NOT the resource but is related to the resource in some way.

Following on the example, let's see what a family and where they live might look like if we referenced them in our people service. You'll note the `spouse` is a single data point, the `children` are in an array and both within the domain of the service. The `house` is in a domain outside of the service.

```
{
  "nameFirst": "John",
  "nameLast": "Doe",
  "gender": "male",
  "colorHair": "brown",
  "colorEyes": "brown",
  "ssn": "123-45-6789",

  // Identifier
  "href": "https://example.com/people/11fa754d-d5b8-45e3-a2e7-d821351dc345",
  "id": "11fa754d-d5b8-45e3-a2e7-d821351dc345",
  "template": "https://example.com/people/{id}",

  // Related Data
  "spouse": "https://example.com/people/3ba141c1-1aaf-47f2-8939-1709f0263008",
  "children": [
      "https://example.com/people/516287fa-dd48-49c9-bc9c-dddaae46a41b",
      "https://example.com/people/9ad2bff7-54e9-480e-bbe9-28f5a970f613",
  ],
  "house": "https://www.google.com/maps/place/1820+Wensley+Dr,+Charlotte,+NC+28210/@35.1541824,-80.8694037,17z"
}
```

##Hypermedia As The Engine Of Application State (HATEOAS)

Continuing the example, the server lets the client know it can update and remove the data.

```
{
  "nameFirst": "John",
  "nameLast": "Doe",
  "gender": "male",
  "colorHair": "brown",
  "colorEyes": "brown",
  "ssn": "123-45-6789",

  // Identifier
  "href": "https://example.com/people/11fa754d-d5b8-45e3-a2e7-d821351dc345",
  "id": "11fa754d-d5b8-45e3-a2e7-d821351dc345",
  "template": "https://example.com/people/{id}",

  // Related Data
  "spouse": "https://example.com/people/3ba141c1-1aaf-47f2-8939-1709f0263008",
  "children": [
      "https://example.com/people/516287fa-dd48-49c9-bc9c-dddaae46a41b",
      "https://example.com/people/9ad2bff7-54e9-480e-bbe9-28f5a970f613",
  ],
  "house": "https://www.google.com/maps/place/1820+Wensley+Dr,+Charlotte,+NC+28210/@35.1541824,-80.8694037,17z",

  // HATEOAS
  "operations": [
    {
      "rel": "update-person",
      "href": "https://example.com/people/11fa754d-d5b8-45e3-a2e7-d821351dc345",
      "method": "PUT"
    },
    {
      "rel": "remove-person",
      "href": "https://example.com/people/11fa754d-d5b8-45e3-a2e7-d821351dc345",
      "method": "DELETE"
    }
  ]
}
```
