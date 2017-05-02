## <a name="table-of-contents"></a>Table of Contents

* [1 Introduction](#introduction)
  * [1.1 Notational Conventions](#notational-conventions)
* [2 Media Type](#mediatype)
* [3 Schema](#schema)
  * [3.1 Root](#schema-root)
  * [3.2 vars](#schema-vars)
  * [3.3 resources](#schema-resources)
  * [3.4 hints](#schema-hints)
  * [3.5 status](#schema-status)
  * [3.6 authSchemes](#schema-authSchemes)
* [4 Examples](#examples)
  * [4.1 Hello World - the simplest example](#examples-hello-world)
  * [4.2 Real World - a more realistic example an API would actually provide](#examples-real-world)
* [8. References](#references)
  * [8.1. Normative References](#normative-references)
  * [8.2. Informative References](#informative-references)
* [Author's Address](#author-address)

## <a name="introduction"></a>1 Introduction

One of the core architectural tenants of the internet is the use of [RFC 3986 Uniform Resource Identifier (URI): Generic Syntax](http://www.rfc-editor.org/info/rfc3986) hyperlinks to navigate between resources. Traditionally, client code (including a human using a web browser) typically make use of documented static URLs (E.g., http://example.com) as the starting point which servers implement and any interaction outside of these bounds is uncharted territory.

JSON-Home documents provide a fundamental starting place for the programmable internet like home pages at root URLs for the World Wide Web, used by humand with browsers.

* Humans use the server's JSON Home document to learn about a server, write client code and help debug issues when they arise.
* Client code uses the JSON Home document:
  * To interact with the server, navigate the resources and activities made available by the server.
  * To note and / or react to changes within the server.

JSON-Home documents embrace and dovetail nicely with the HTTP protocol and REST principles:

* Server independence from client - Server interfaces can independently evolve without breaking client code through the addition of:
  * New link relations (Extensibility)
  * New metadata for existing link relations. (Evolvability)
* Abstraction of implementation details - The server can choose the URLs associated with a given link relation without breaking client code which is bound to relation descriptions which rarely, if ever, change.
* JSON Home aligns nicely with RESTful uniform interface constraints:
  * Identification of resources - JSON Home documents define the starter set of all of the resources available at a given root URL.
  * Hypermedia as the engine of application state (HATEOAS) - Home documents are the first place the server can define its state for the client, for example: emitting authorization boundaries by making only the resources / methods available as appropriate for the caller.
  * Self describing - JSON Home allows the server to be wholly self-describing starting at the root URL.
  * Manipulation of resources through representations - JSON Home documents provide excellent hints to client code on how to manipulate resources and reprentations to achieve objectives.

## <a name="notational-conventions"></a>1.1 Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119].

## <a name="mediatype"></a>2 Media Type

A JSON Home Document uses the format described in [RFC7159 The JavaScript Object Notation (JSON) Data Interchange Format](http://www.rfc-editor.org/info/rfc7159) and has the media type "application/json-home".

## <a name="schema"></a>3 Schema

[JSON Schema](http://json-schema.org/) describing this schema can be found [here](./json-home.v3.schema.json).

### <a name="schema-root"></a>3.1 Root

Name | Type | Format | Description
-----|------|--------|------------
`schema`|`string`|-|**Required** RFC 3986 Uniform Resource Identifier (URI). Provides the self documentating feature for the resource / representation. Analogous to the 'describedby' relation type from RFC 5988 Web Linking.
`href`|`string`|-|**Required** Identifier and hyperlink (aka 'self') for this JSON home resource using RFC 3986 Uniform Resource Identifier (URI): Generic Syntax https://www.ietf.org/rfc/rfc3986.txt.
`title`|`type`|-|The human readable title of the API described by this JSON Home resource. http://json-schema.org/latest/json-schema-hypermedia.html#rfc.section.5.3
`vars`|`array`|[`vars`](#schema-vars)|An array of variables to be used with hyperlink templates given in the document.
`resources`|`array`|[`resources`](#schema-resources)|**Required** An array of resources and hints about the resources used by humans writing client code and the client code itself.

### <a name="schema-vars"></a>3.2 vars

* `varName` + either `varDefinition` or `varValue` is required noting this either/or requirement cannot be described using JSON schema.
* Servers should avoid having client code build strings for URIs except in response to the JSON Home Document in order to preserve REST principles.

Name | Type | Format | Description
-----|------|--------|------------
`varName`|`string`|-|**Required** The variable name.
`varDefinition`|`string`|-|Link to a definition for the variable using RFC3986.
`varValue`|`string`|-|The variable value.

### <a name="schema-resources"></a>3.3 resources

Name | Type | Format | Description
-----|------|--------|------------
`rel`|`string`|-|**Required** RFC 5988 Web Linking Link Relation Types https://tools.ietf.org/html/rfc5988#section-5.3 Examples: author, describedBy
`href`|`string`|-|**Required** Link to a resource using RFC 3986 Uniform Resource Identifier (URI): Generic Syntax https://www.ietf.org/rfc/rfc3986.txt OR RFC6570 URI Template https://tools.ietf.org/html/rfc6570.
`hints`|`array`|[hints](#schema-hints)|**Required** Hints provided to both humans writing the client code and the client code itself, describing the resource and activities available for a given link relation.

### <a name="schema-hints"></a>3.4 hints

Name | Type | Format | Description
-----|------|--------|------------
`method`|`string`|-|**Required** Valid method names are RFC 7231 GET, HEAD, POST, PUT, DELETE, CONNECT, OPTIONS, TRACE + RFC 5789 PATCH.
`formats`|`array`|`string`|Hints the representation types that the resource produces and consumes, aligned with the ‘method’ hint.
`prefer`|`array`|`string`|Hints the RFC 7240 Prefer Header for HTTP https://tools.ietf.org/html/rfc7240 supported by the resource. Note that, as per the RFC, a preference can be ignored by the server.
`profiles`|`array`|`string`|Links to the schema(s) (typically JSON schema) which define payloads the endpoint will generally accept. Based on RFC 6096 The 'profile' Link Relation Type https://tools.ietf.org/html/rfc6906
`preconditionRequired`|`array`|`string`|Hints that the resource requires state-changing requests (e.g., PUT, PATCH) using RFC 7232 Hypertext Transfer Protocol (HTTP/1.1): Conditional Requests https://tools.ietf.org/html/rfc7232 to avoid conflicts due to concurrent updates. Examples: `etag`, `last-modified` and `If-Match`.
`acceptRanges`|`array`|`string`|Hints the range-specifiers available to the client for this resource; equivalent to the `Accept-Ranges` HTTP response header in RFC 7233 Hypertext Transfer Protocol (HTTP/1.1): Range Requests.
`docs`|`string`|-|Hints the location for human-readable documentation for the relation type of the resource, a RFC 3986 Uniform Resource Identifier (URI): Generic Syntax https://www.ietf.org/rfc/rfc3986.txt referring to documentation that SHOULD be in HTML format.
`status`|`object`|[`status`](#schema-status)|Hints the status of the resource.
`authSchemes`|`array`|[`authschemes`](#schema-authSchemes)|Hints that the resource requires authentication using RFC 7235 Hypertext Transfer Protocol (HTTP/1.1): Authentication https://tools.ietf.org/html/rfc7235

### <a name="schema-status"></a>3.5 status

Name | Type | Format | Description
-----|------|--------|------------
`code`|`string`|-|The document uses existing HTTP status codes to hint the likely outcome of using an endpoint, for example: `410 Gone` (no longer available), `301 Moved Permanently` or `308 Permanent Redirect` (choices for deprecation of an endpoint).
`rels`|`array`|`string`|When the code is `301 Moved Permanently` or `308 Permanent Redirect` the `rel` points the consumer of this document to the alternative resource(s)."

### <a name="schema-authSchemes"></a>3.6 authSchemes

Name | Type | Format | Description
-----|------|--------|------------
`scheme`|`string`|-|**Required** An HTTP authentication scheme
`realms`|`array`|`string`|An array of zero to many strings that identify protection spaces that the resource is a member of.

## <a name="examples"></a>4 Examples

### <a name="examples-hello-world"></a>4.1 Hello World - the simplest example

This example represents the bare minimum JSON Home document.

* Reading through the example:
  * Self documentation is provided (`schema`) with a URI to the versioned JSON home schema to which the document conforms.
  * A self link (`href`) is noted.
  * Resources (`resources`) available are documented with:
    * A link relation (`rel`).
    * Associated hyperlink (`href`).
    * Hints (`hints`) with one method (`method`) which is an HTTP GET.

```json
{
  "schema": "http://example.com/schemas/json-home.v3.schema.json",
  "href": "http://example.com/apis",
  "resources": [
    {
      "rel": "hello-world",
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

### <a name="examples-real-world"></a>4.2 Real World - a more realistic example an API would actually provide

```json
{
  "schema": "http://example.com/schemas/json-home.v3.schema.json",
  "href": "http://example.com/apis",
  "title": "The API for Example",
  "resources": [
    {
      "rel": "create-an-account",
      "href": "http://example.com/accounts",
      "hints": [
        {
          "method": "post",
          "formats": [ "application/json" ],
          "profiles": [ "http://example.com/schemas/account-info.v1.schema.json" ],
          "docs": "http://example.com/apis/creating-an-account.html"
        }
      ]
    },
    {
      "rel": "list-accounts",
      "href": "http://example.com/accounts",
      "hints": [
        {
          "method": "get",
          "formats": [ "application/json", "text/xml" ],
          "profiles": [ "http://example.com/schemas/account-info.v1.schema.json" ],
          "docs": "http://example.com/apis/listing-accounts.html"
        }
      ]
    }
  ]
}
```

### <a name="examples-variables"></a>4.3 Using variables



### <a name="examples-precondition"></a>4.4 Providing precondition details



### <a name="examples-status"></a>4.5 Describing status



### <a name="examples-accept-ranges"></a>4.6 acceptRanges



### <a name="examples-prefer"></a>4.7 prefer



### <a name="examples-auth-schemes"></a>4.8 Describing authorization and authentication





## <a name="references"></a>8. References

### <a name="normative-references"></a>8.1. Normative References

[RFC2119] Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, DOI 10.17487/ RFC2119, March 1997, <http://www.rfc-editor.org/info/rfc2119>.

[RFC3986]  Berners-Lee, T., Fielding, R., and L. Masinter, "Uniform Resource Identifier (URI): Generic Syntax", STD 66, RFC 3986, DOI 10.17487/RFC3986, January 2005, <http://www.rfc-editor.org/info/rfc3986>.

[RFC5226]  Narten, T. and H. Alvestrand, "Guidelines for Writing an IANA Considerations Section in RFCs", BCP 26, RFC 5226, DOI 10.17487/RFC5226, May 2008, <http://www.rfc-editor.org/info/rfc5226>.

[RFC5988]  Nottingham, M., "Web Linking", RFC 5988, DOI 10.17487/ RFC5988, October 2010, <http://www.rfc-editor.org/info/rfc5988>.

[RFC6570]  Gregorio, J., Fielding, R., Hadley, M., Nottingham, M., and D. Orchard, "URI Template", RFC 6570, DOI 10.17487/ RFC6570, March 2012, <http://www.rfc-editor.org/info/rfc6570>.

[RFC7159]  Bray, T., Ed., "The JavaScript Object Notation (JSON) Data Interchange Format", RFC 7159, DOI 10.17487/RFC7159, March 2014, <http://www.rfc-editor.org/info/rfc7159>.

[RFC7234]  Fielding, R., Ed., Nottingham, M., Ed., and J. Reschke, Ed., "Hypertext Transfer Protocol (HTTP/1.1): Caching", RFC 7234, DOI 10.17487/RFC7234, June 2014, <http://www.rfc-editor.org/info/rfc7234>.

### <a name="informative-references"></a>8.2. Informative References

[RFC4151]  Kindberg, T. and S. Hawke, "The 'tag' URI Scheme", RFC 4151, DOI 10.17487/RFC4151, October 2005, <http://www.rfc-editor.org/info/rfc4151>.

[RFC5789]  Dusseault, L. and J. Snell, "PATCH Method for HTTP", RFC 5789, DOI 10.17487/RFC5789, March 2010, <http://www.rfc-editor.org/info/rfc5789>.

[RFC6838]  Freed, N., Klensin, J., and T. Hansen, "Media Type Specifications and Registration Procedures", BCP 13, RFC 6838, DOI 10.17487/RFC6838, January 2013, <http://www.rfc-editor.org/info/rfc6838>.

[RFC7230]  Fielding, R., Ed. and J. Reschke, Ed., "Hypertext Transfer Protocol (HTTP/1.1): Message Syntax and Routing", RFC 7230, DOI 10.17487/RFC7230, June 2014, <http://www.rfc-editor.org/info/rfc7230>.

[RFC7232]  Fielding, R., Ed. and J. Reschke, Ed., "Hypertext Transfer Protocol (HTTP/1.1): Conditional Requests", RFC 7232, DOI 10.17487/RFC7232, June 2014, <http://www.rfc-editor.org/info/rfc7232>.

[RFC7233]  Fielding, R., Ed., Lafon, Y., Ed., and J. Reschke, Ed., "Hypertext Transfer Protocol (HTTP/1.1): Range Requests", RFC 7233, DOI 10.17487/RFC7233, June 2014, <http://www.rfc-editor.org/info/rfc7233>.

[RFC7235]  Fielding, R., Ed. and J. Reschke, Ed., "Hypertext Transfer Protocol (HTTP/1.1): Authentication", RFC 7235, DOI 10.17487/RFC7235, June 2014, <http://www.rfc-editor.org/info/rfc7235>.

[RFC7240]  Snell, J., "Prefer Header for HTTP", RFC 7240, DOI 10.17487/RFC7240, June 2014, <http://www.rfc-editor.org/info/rfc7240>.

## <a name="author-address"></a>Author's Address

Mark Nottingham
Email: mnot@mnot.net
URI:   https://www.mnot.net/
