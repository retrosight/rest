##<a name="table-of-contents"></a>Table of Contents

* [1. Introduction](#introduction)
  * [1.1. Notational Conventions](#notational-conventions)
* [2. JSON Home Documents](#json-home-documents)
* [3. Schema](#schema)
* [4. Resource Objects](#resource-objects)
  * [4.1. Resolving Templated Links](#resolving-templated-links)
* [5. Resource Hints](#resource-hints)
  * [5.1. allow](#resource-hints-allow)
  * [5.2. formats](#resource-hints-formats)
  * [5.3. acceptPatch](#resource-hints-acceptpatch)
  * [5.4. acceptPost](#resource-hints-acceptpost)
  * [5.5. acceptRanges](#resource-hints-acceptranges)
  * [5.6. acceptPrefer](#resource-hints-acceptprefer)
  * [5.7. docs](#resource-hints-docs)
  * [5.8. preconditionRequired](#resource-hints-preconditionrequired)
  * [5.9. authSchemes](#resource-hints-authschemes)
  * [5.10. status](#resource-hints-status)
* [6. Security Considerations](#security-considerations)
* [7. IANA Considerations](#iana-considerations)
  * [7.1. HTTP Resource Hint Registry](#iana-http)
  * [7.2. Media Type Registration](#iana-mediatype)
* [8. References](#references)
  * [8.1. Normative References](#normative-references)
  * [8.2. Informative References](#informative-references)
* [Appendix A. Acknowledgements](#appendix-a)
* [Appendix B. Considerations for Creating and Serving Home Documents](#appendix-b)
  * [B.1.  Managing Change in Home Documents](#appendix-b-managing-change)
  * [B.2.  Evolving and Mixing APIs with Home Documents](#appendix-b-evolving-and-mixing)
* [Appendix C.  Considerations for Consuming Home Documents](#appendix-c)
* [Appendix D.  Frequently Asked Questions](#appendix-d)
  * [D.1.  Why doesn't the format allow references or inheritance?](#appendix-d-references-or-inheritance)
  * [D.2.  What about "Faults" (i.e., errors)?](#appendix-d-errors)
  * [D.3.  How Do I find the schema for a format?](#appendix-d-schema)
  * [D.4.  How do I express complex query arguments?](#appendix-d-complex-query)
* [Author's Address](#author-address)

## <a name="introduction"></a>1. Introduction

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

##<a name="notational-conventions"></a>1.1. Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119].

##<a name="json-home-documents"></a>2. JSON Home Documents

A JSON Home Document uses the format described in [RFC7159 The JavaScript Object Notation (JSON) Data Interchange Format](http://www.rfc-editor.org/info/rfc7159) and has the media type "application/json-home".

##<a name="schema"></a>2. Schema

[JSON Schema](http://json-schema.org/) describing this schema can be found [here](./json-home.v3.schema.json).

##<a name="schema-root"></a>2.1 Root

Name | Type | Format | Description
-----|------|--------|------------
`schema`|`string`|-|**Required** RFC 3986 Uniform Resource Identifier (URI). Provides the self documentating feature for the resource / representation. Analogous to the 'describedby' relation type from RFC 5988 Web Linking.
`href`|`string`|-|**Required** Identifier and hyperlink (aka 'self') for this JSON home resource using RFC 3986 Uniform Resource Identifier (URI): Generic Syntax https://www.ietf.org/rfc/rfc3986.txt.
`title`|`type`|-|The human readable title of the API described by this JSON Home resource. http://json-schema.org/latest/json-schema-hypermedia.html#rfc.section.5.3
`vars`|`array`|[`vars`](#schema-vars)|An array of variables to be used with hyperlink templates given in the document.
`resources`|`array`|[`resources`](#schema-resources)|An array of resources and hints about the resources used by humans writing client code and the client code itself.

###<a name="schema-vars"></a>2.2 vars

* `varName` + either `varDefinition` or `varValue` is required noting this cannot be described using JSON schema.
* Servers should avoid having client code build strings for URIs except in response to the JSON Home Document in order to maintain REST principles.

Name | Type | Format | Description
-----|------|--------|------------
`varName`|`string`|-|**Required** The variable name.
`varDefinition`|`string`|-|Link to a definition for the variable using RFC3986.
`varValue`|`string`|-|The variable value.

###<a name="schema-resources"></a>2.3 resources

Name | Type | Format | Description
-----|------|--------|------------
`rel`|`string`|-|RFC 5988 Web Linking Link Relation Types https://tools.ietf.org/html/rfc5988#section-5.3 Examples: author, describedBy
`href`|`string`|-|Link to a resource using RFC 3986 Uniform Resource Identifier (URI): Generic Syntax https://www.ietf.org/rfc/rfc3986.txt OR RFC6570 URI Template https://tools.ietf.org/html/rfc6570.
`hints`|`array`|[hints]()|Hints provided to both humans writing the client code and the client code itself, describing the resource and activities available for a given link relation.

###<a name="schema-resources"></a>2.4 hints

Name | Type | Format | Description
-----|------|--------|------------
`method`|`string`|-|Valid method names are RFC 7231 GET, HEAD, POST, PUT, DELETE, CONNECT, OPTIONS, TRACE + RFC 5789 PATCH.
`formats`|`array`|`string`|Hints the representation types that the resource produces and consumes, aligned with the ‘method’ hint.
`prefer`|`array`|`string`|Hints the RFC 7240 Prefer Header for HTTP https://tools.ietf.org/html/rfc7240 supported by the resource. Note that, as per the RFC, a preference can be ignored by the server.
`profiles`|`array`|`string`|Links to the schema(s) (typically JSON schema) which define payloads the endpoint will generally accept. Based on RFC 6096 The 'profile' Link Relation Type https://tools.ietf.org/html/rfc6906
`preconditionRequired`|`array`|`string`|Hints that the resource requires state-changing requests (e.g., PUT, PATCH) using RFC 7232 Hypertext Transfer Protocol (HTTP/1.1): Conditional Requests https://tools.ietf.org/html/rfc7232 to avoid conflicts due to concurrent updates. Examples: `etag`, `last-modified` and `If-Match`.
`acceptRanges`|`array`|`string`|Hints the range-specifiers available to the client for this resource; equivalent to the `Accept-Ranges` HTTP response header in RFC 7233 Hypertext Transfer Protocol (HTTP/1.1): Range Requests.
`docs`|`string`|-|Hints the location for human-readable documentation for the relation type of the resource, a RFC 3986 Uniform Resource Identifier (URI): Generic Syntax https://www.ietf.org/rfc/rfc3986.txt referring to documentation that SHOULD be in HTML format.
`status`|`object`|[`status`](#schema-status)|Hints the status of the resource.
`authSchemes`|`array`|`authschemes`|Hints that the resource requires authentication using RFC 7235 Hypertext Transfer Protocol (HTTP/1.1): Authentication https://tools.ietf.org/html/rfc7235

###<a name="schema-status"></a>2.5 status

Name | Type | Format | Description
-----|------|--------|------------
`code`|`string`|-|The document uses existing HTTP status codes to hint the likely outcome of using an endpoint, for example: `410 Gone` (no longer available), `301 Moved Permanently` or `308 Permanent Redirect` (choices for deprecation of an endpoint).
`rels`|`array`|`string`|When the code is `301 Moved Permanently` or `308 Permanent Redirect` the `rel` points the consumer of this document to the alternative resource(s)."

###<a name="schema-authSchemes"></a>2.6 authSchemes

Name | Type | Format | Description
-----|------|--------|------------
`scheme`|`string`|-|**Required** An HTTP authentication scheme
`realms`|`array`|`string`|An array of zero to many strings that identify protection spaces that the resource is a member of.

Its content consists of a root object with:

* A "resources" member, whose value is an object that describes the resources associated with the API.  Its member names are link relation types (as defined by [RFC5988](http://www.rfc-editor.org/info/rfc5988)), and their values are Resource Objects (Section 4).
* Optionally, a "api" member, whose value is an API Object (Section 3) that contains information about the API as a whole.

For example:

> Owen: Verbose HTTP request / response information probably doesn't add value here.

```
GET / HTTP/1.1
Host: example.org
Accept: application/json-home

HTTP/1.1 200 OK
Content-Type: application/json-home
Cache-Control: max-age=3600
Connection: close
```

> Owen: This is invalid JSON as there is a comma missing between the "api" and "resources" objects.

```json
{
 "api": {
   "title": "Example API",
   "links": {
     "author": "mailto:api-admin@example.com",
     "describedBy": "https://example.com/api-docs/"
   }
 }
 "resources": {
   "tag:me@example.com,2016:widgets": {
     "href": "/widgets/"
   },
   "tag:me@example.com,2016:widget": {
     "hrefTemplate": "/widgets/{widget_id}",
     "hrefVars": {
       "widget_id": "https://example.org/param/widget"
     },
     "hints": {
       "allow": ["GET", "PUT", "DELETE", "PATCH"],
       "formats": {
         "application/json": {}
       },
       "acceptPatch": ["application/json-patch+json"],
       "acceptRanges": ["bytes"]
     }
   }
 }
}
```

> Owen: Move this above the example. Also, there is likely value in talking about the schema of a JSON Home document in the entirety before any examples are given.

Here, we have a home document for the API "Example API", whose author can be contacted at the e-mail address "api-admin@example.com", and whose documentation is at "https://example.com/api-docs/".

> Owen: Having the object / key name be the relation name presents a rather large hurdle (blocker) for client code: It makes it difficult at best to create shared client libraries and tends to result in 'hard coding' of any client code to a specific API. Generally speaking, keys are static and values are dynamic. Suggest a `"rel"="tag:me@example.com,2016:widget"` approach instead.

It links to a resource "/widgets/" with the relation "tag:me@example.com,2016:widgets".  It also links to an unknown number of resources with the relation type "tag:me@example.com,2016:widget" using a URI Template [RFC6570](http://www.rfc-editor.org/info/rfc6570), along with a mapping of identifiers to a variable for use in that template.

It also gives several hints about interacting with the latter "widget" resources, including the HTTP methods usable with them, the PATCH and POST formats they accept, and the fact that they support partial requests [RFC7233] using the "bytes" range-specifier.

> Owen: It is perhaps interesting to note JSON Home documents can be pointers to other JSON Home documents -- so that likely should be called out as one of the examples.

It gives no such hints about the "widgets" resource.  This does not mean that it (for example) doesn't support any HTTP methods; it means that the client will need to discover this by interacting with the resource, and/or examining the documentation for its link relation type.

> Owen: Should probably investigate the use of n-number of key value pairs to allow for the URI templating to be explicity addressed. Also note the `base` keyword in JSON Schema: http://json-schema.org/latest/json-schema-hypermedia.html#rfc.section.4.1. In the current examples you can have only one common base URL which might be limiting in describing a graph of APIs.

Effectively, this names a set of behaviors, as described by a resource object, with a link relation type.  This means that several link relations might apply to a common base URL; e.g.:

```json
{
 "resources": {
   "tag:me@example.com,2016:search-by-id": {
     "hrefTemplate": "/search?id={widget_id}",
     "hrefVars": {
       "widget_id": "https://example.org/param/widget_id"
     }
   },
   "tag:me@example.com,2016:search-by-name": {
     "hrefTemplate": "/search?name={widget_name}",
     "hrefVars": {
       "widget_name": "https://example.org/param/widget_name"
     }
   }
 }
}
```

Note that the examples above use both tag [RFC4151](http://www.rfc-editor.org/info/rfc4151) and https [RFC7230](http://www.rfc-editor.org/info/rfc7230) URIs; any URI scheme can be used to identify link relations and other artefacts in home documents.

##<a name="api-objects"></a>3. API Objects

> Owen: In reality, links in the context of the examples are simply additional resources (perhaps more accurately, representations of resources). Suggest these simply be documented as such according to Section 4. Resource Objects and move `title` into the root of the JSON document, effectively eliminating this key from the vocabulary.

An API Object contains links to information about the API itself.

Two members are defined:

* "title" has a string value indicating the name of the API;
* "links" has an object value, whose member names are link relation types [RFC5988](http://www.rfc-editor.org/info/rfc5988), and values are URLs [RFC3986](http://www.rfc-editor.org/info/rfc3986). The context of these links is the API home document as a whole.

Future members MAY be defined by specifications that update this document.

##<a name="resource-objects"></a>4. Resource Objects

> Owen: It does not seem like a direct link will automatically imply exactly one resource, if for no other reason than from a data perspective, a resource can be an array of multiple resources / representations. An example following the POST to /widgets is a GET from /widgets which returns an array of widgets, each one of which can be referenced as a separate resource. Likewise, a templated link may not imply multiple resources. In summary, the RFC should remain agnostic to the definition of a resource, but simply explain how to get to resources for a given link relation.

A Resource Object links to resources of the defined type using one of two mechanisms; either a direct link (in which case there is exactly one resource of that relation type associated with the API), or a templated link, in which case there are zero to many such resources.

Direct links are indicated with an "href" property, whose value is a URI [RFC3986](http://www.rfc-editor.org/info/rfc3986).

> Owen: Suggest moving the `hrefVars` into the root of the document so they can be leveraged across mulitple relation types (aka end points).

Templated links are indicated with an "hrefTemplate" property, whose value is a URI Template [RFC6570](http://www.rfc-editor.org/info/rfc6570). When "hrefTemplate" is present, the Resource Object MUST have a "hrefVars" property; see "Resolving Templated Links".

> Owen: Suggest only having `href` which can be either a direct URI or a templated URI. I am a little confused about why they must have `exactly one` of the `href-vars` because variables are very powerful for templates and APIs may require multiple variables to identify resources.

Resource Objects MUST have exactly one of the "href" and "href-vars" properties.

In both forms, the links that "href" and "hrefTemplate" refer to are URI-references [RFC3986](http://www.rfc-editor.org/info/rfc3986) whose base URI is that of the API Home Document itself.

Resource Objects MAY also have a "hints" property, whose value is an object that uses named Resource Hints (see Section 5) as its properties.

###<a name="resolving-templated-links"></a>4.1. Resolving Templated Links

> Owen: Suggest RFC 6570 does a sufficient job at explaining templated URIs and it is probably best to allow that RFC to address the topic fully than oversimplify.

A URI can be derived from a Templated Link by treating the "hrefTemplate" value as a Level 3 URI Template [RFC6570](http://www.rfc-editor.org/info/rfc6570), using the "hrefVars" property to fill the template.

The "hrefVars" property, in turn, is an object that acts as a mapping between variable names available to the template and absolute URIs that are used as global identifiers for the semantics and syntax of those variables.

For example, given the following Resource Object:

```json
"https://example.org/rel/widget": {
 "hrefTemplate": "/widgets/{widget_id}",
 "hrefVars": {
   "widget_id": "https://example.org/param/widget"
 },
 "hints": {
   "allow": ["GET", "PUT", "DELETE", "PATCH"],
   "formats": {
     "application/json": {}
   },
   "acceptPatch": ["application/json-patch+json"],
   "acceptRanges": ["bytes"]
 }
}
  ```

If you understand that "https://example.org/param/widget" is an numeric identifier for a widget, you can then find the resource corresponding to widget number 12345 at "https://example.org/widgets/12345" (assuming that the Home Document is located at "https://example.org/").

##<a name="resource-hints"></a>5. Resource Hints

> Owen: Following an earlier comment, suggest making a clear distinction between human discovery and client code -- both will use hints, so it's a good idea to explicity document use cases.

Resource hints allow clients to find relevant information about interacting with a resource beforehand, as a means of optimizing communications, as well as advertising available behaviors (e.g., to aid in laying out a user interface for consuming the API).

> Owen: Hints not being a contract: client code will very likely bind to hints if for no other reason than to note when things change or are added to alert a human; If client code is meant to use them they are part of the runtime behavior. The runtime behavior of a resource should ALWAYS match the entire definition of a JSON Home document.

Hints are just that - they are not a "contract", and are to only be taken as advisory.  The runtime behavior of the resource always overrides hinted information.

> Owen: Is this describing hints at all or is it defining how the server is always in control? Feels like the latter so probably extraneous information best left out of the RFC.

For example, a resource might hint that the PUT method is allowed on all "widget" resources.  This means that generally, the user has the ability to PUT to a particular resource, but a specific resource might reject a PUT based upon access control or other considerations. More fine-grained information might be gathered by interacting with the resource (e.g., via a GET), or by another resource "containing" it (such as a "widgets" collection) or describing it (e.g., one linked to it with a "describedBy" link relation).

This specification defines a set of common hints, based upon information that's discoverable by directly interacting with resources.  See Section 7.1 for information on defining new hints.

###<a name="resource-hints-allow"></a>5.1. allow

> Owen: The challenge here is the actions can be vastly different between methods -- merely advertising the methods according to https://tools.ietf.org/html/rfc7231#section-7.4.1 does not appear to get granular enough for the human or client code to know what other hints can be used with each method; basically creating a cartesian product of sorts. Using this approach means hints can really only be used as documentation rather than during runtime.

* Resource Hint Name: allow
* Description: Hints the HTTP methods that the current client will be able to use to interact with the resource; equivalent to the Allow HTTP response header.
* Specification: [this document]

Content MUST be an array of strings, containing HTTP methods.

###<a name="resource-hints-formats"></a>5.2. formats

> Owen: Some APIs allow for multiple responses for GET yet limit for POST / PUT and vice versa -- in others words this isn't always 1:1 and often is not by design. I'd like to understand the approach where keys are media types and what objects are envisioned. I suspect they would eventually be links to schemas describing the documents the service accepts, which is currently missing from the design.

* Resource Hint Name: formats
* Description: Hints the representation types that the resource produces and consumes, using the GET and PUT methods respectively, subject to the 'allow' hint.
* Specification: [this document]

Content MUST be an object, whose keys are media types, and values are objects, currently empty.

###<a name="resource-hints-acceptpatch"></a>5.3. acceptPatch

> Owen: This design makes assumptions about the use of the different methods (`PUT, GET, POST...etc`) and effectively hard codes certain methods within the schema, such as `accept-Patch` and `acceptPost` yet these are functionally identical to the `formats` hint. Suggest a grouping by methods, each of which has `formats` and `prefers`.

* Resource Hint Name: accept-Patch
* Description: Hints the PATCH [RFC5789] request formats accepted by the resource for this client; equivalent to the Accept-Patch HTTP response header.
* Specification: [this document]

Content MUST be an array of strings, containing media types.

When this hint is present, "PATCH" SHOULD be listed in the "allow" hint.

###<a name="resource-hints-acceptpost"></a>5.4. acceptPost

> Owen: See note on acceptPatch.

* Resource Hint Name: acceptPost
* Description: Hints the POST request formats accepted by the resource for this client.
* Specification: [this document]

Content MUST be an array of strings, containing media types.

When this hint is present, "POST" SHOULD be listed in the "allow" hint.

###<a name="resource-hints-acceptranges"></a>5.5. acceptRanges

* Resource Hint Name: acceptRanges
* Description: Hints the range-specifiers available to the client for this resource; equivalent to the Accept-Ranges HTTP response header [RFC7233].
* Specification: [this document]

Content MUST be an array of strings, containing HTTP range-specifiers (typically, "bytes").

###<a name="resource-hints-acceptprefer"></a>5.6. acceptPrefer

> Owen: Suggest shortening to simply `prefer` to match with the RFC.

* Resource Hint Name: acceptPrefer
* Description: Hints the preferences [RFC7240] supported by the resource.  Note that, as per that specifications, a preference can be ignored by the server.
* Specification: [this document]

Content MUST be an array of strings, containing preferences.

###<a name="resource-hints-docs"></a>5.7. docs

* Resource Hint Name: docs
* Description: Hints the location for human-readable documentation for the relation type of the resource.
* Specification: [this document]

Content MUST be a string containing an absolute-URI [RFC3986](http://www.rfc-editor.org/info/rfc3986) referring to documentation that SHOULD be in HTML format.

###<a name="resource-hints-preconditionrequired"></a>5.8. preconditionRequired

* Resource Hint Name: preconditionRequired
* Description: Hints that the resource requires state-changing requests (e.g., PUT, PATCH) to include a precondition, as per [RFC7232](http://www.rfc-editor.org/info/rfc7232), to avoid conflicts due to concurrent updates.
* Specification: [this document]

Content MUST be an array of strings, with possible values "etag" and "last-modified" indicating type of precondition expected.

###<a name="resource-hints-authschemes"></a>5.9. authSchemes

* Resource Hint Name: authSchemes
* Description: Hints that the resource requires authentication using the HTTP Authentication Framework [RFC7235](http://www.rfc-editor.org/info/rfc7235).
* Specification: [this document]

Content MUST be an array of objects, each with a "scheme" property containing a string that corresponds to a HTTP authentication scheme, and optionally a "realms" property containing an array of zero to many strings that identify protection spaces that the resource is a member of.

For example, a Resource Object might contain the following hint:

```json
{
 "authSchemes": [
   {
     "scheme": "Basic",
     "realms": ["private"]
   }
 ]
}
```

###<a name="resource-hints-status"></a>5.10. status

* Resource Hint Name: status
* Description: Hints the status of the resource.
* Specification: [this document]

Content MUST be a string; possible values are:

* "deprecated" - indicates that use of the resource is not recommended, but it is still available.
* "gone" - indicates that the resource is no longer available; i.e., it will return a 404 (Not Found) or 410 (Gone) HTTP status code if accessed.

##<a name="security-considerations"></a>6. Security Considerations

Clients need to exercise care when using hints.  For example, a naive client might send credentials to a server that uses the auth-req hint, without checking to see if those credentials are appropriate for that server.

##<a name="iana-considerations"></a>7. IANA Considerations

###<a name="iana-http"></a>7.1. HTTP Resource Hint Registry

This specification defines the HTTP Resource Hint Registry.  See Section 5 for a general description of the function of resource hints.

In particular, resource hints are generic; that is, they are potentially applicable to any resource, not specific to one application of HTTP, nor to one particular format.  Generally, they ought to be information that would otherwise be discoverable by interacting with the resource.

Hint names MUST be composed of the lowercase letters (a-z), digits (0-9), underscores (`_`) and hyphens (`-`), and MUST begin with a lowercase letter.

Hint content SHOULD be described in terms of JSON [RFC7159](http://www.rfc-editor.org/info/rfc7159) constructs.

New hints are registered using the Expert Review process described in [RFC5226] to enforce the criteria above.  Requests for registration of new resource hints are to use the following template:

* Resource Hint Name: [hint name]
* Description: [a short description of the hint's semantics]
* Specification: [reference to specification document]

Initial registrations are enumerated in Section 5.

###<a name="iana-mediatype"></a>7.2. Media Type Registration

TBD

##<a name="references"></a>8. References

###<a name="normative-references"></a>8.1. Normative References

[RFC2119] Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, DOI 10.17487/ RFC2119, March 1997, <http://www.rfc-editor.org/info/rfc2119>.

[RFC3986]  Berners-Lee, T., Fielding, R., and L. Masinter, "Uniform Resource Identifier (URI): Generic Syntax", STD 66, RFC 3986, DOI 10.17487/RFC3986, January 2005, <http://www.rfc-editor.org/info/rfc3986>.

[RFC5226]  Narten, T. and H. Alvestrand, "Guidelines for Writing an IANA Considerations Section in RFCs", BCP 26, RFC 5226, DOI 10.17487/RFC5226, May 2008, <http://www.rfc-editor.org/info/rfc5226>.

[RFC5988]  Nottingham, M., "Web Linking", RFC 5988, DOI 10.17487/ RFC5988, October 2010, <http://www.rfc-editor.org/info/rfc5988>.

[RFC6570]  Gregorio, J., Fielding, R., Hadley, M., Nottingham, M., and D. Orchard, "URI Template", RFC 6570, DOI 10.17487/ RFC6570, March 2012, <http://www.rfc-editor.org/info/rfc6570>.

[RFC7159]  Bray, T., Ed., "The JavaScript Object Notation (JSON) Data Interchange Format", RFC 7159, DOI 10.17487/RFC7159, March 2014, <http://www.rfc-editor.org/info/rfc7159>.

[RFC7234]  Fielding, R., Ed., Nottingham, M., Ed., and J. Reschke, Ed., "Hypertext Transfer Protocol (HTTP/1.1): Caching", RFC 7234, DOI 10.17487/RFC7234, June 2014, <http://www.rfc-editor.org/info/rfc7234>.

###<a name="informative-references"></a>8.2. Informative References

[RFC4151]  Kindberg, T. and S. Hawke, "The 'tag' URI Scheme", RFC 4151, DOI 10.17487/RFC4151, October 2005, <http://www.rfc-editor.org/info/rfc4151>.

[RFC5789]  Dusseault, L. and J. Snell, "PATCH Method for HTTP", RFC 5789, DOI 10.17487/RFC5789, March 2010, <http://www.rfc-editor.org/info/rfc5789>.

[RFC6838]  Freed, N., Klensin, J., and T. Hansen, "Media Type Specifications and Registration Procedures", BCP 13, RFC 6838, DOI 10.17487/RFC6838, January 2013, <http://www.rfc-editor.org/info/rfc6838>.

[RFC7230]  Fielding, R., Ed. and J. Reschke, Ed., "Hypertext Transfer Protocol (HTTP/1.1): Message Syntax and Routing", RFC 7230, DOI 10.17487/RFC7230, June 2014, <http://www.rfc-editor.org/info/rfc7230>.

[RFC7232]  Fielding, R., Ed. and J. Reschke, Ed., "Hypertext Transfer Protocol (HTTP/1.1): Conditional Requests", RFC 7232, DOI 10.17487/RFC7232, June 2014, <http://www.rfc-editor.org/info/rfc7232>.

[RFC7233]  Fielding, R., Ed., Lafon, Y., Ed., and J. Reschke, Ed., "Hypertext Transfer Protocol (HTTP/1.1): Range Requests", RFC 7233, DOI 10.17487/RFC7233, June 2014, <http://www.rfc-editor.org/info/rfc7233>.

[RFC7235]  Fielding, R., Ed. and J. Reschke, Ed., "Hypertext Transfer Protocol (HTTP/1.1): Authentication", RFC 7235, DOI 10.17487/RFC7235, June 2014, <http://www.rfc-editor.org/info/rfc7235>.

[RFC7240]  Snell, J., "Prefer Header for HTTP", RFC 7240, DOI 10.17487/RFC7240, June 2014, <http://www.rfc-editor.org/info/rfc7240>.

##<a name="appendix-a"></a>Appendix A. Acknowledgements

Thanks to Jan Algermissen, Mike Amundsen, Bill Burke, Sven Dietze, Graham Klyne, Leif Hedstrom, Joe Hildebrand, Jeni Tennison, Erik Wilde and Jorge Williams for their suggestions and feedback.

##<a name="appendix-b"></a>Appendix B. Considerations for Creating and Serving Home Documents

When making an API home document available, there are a few things to keep in mind:

* A home document is best located at a memorable URI, because its URI will effectively become the URI for the API itself to clients.
* Home documents can be personalized, just as "normal" home pages can.  For example, you might advertise different URIs, and/or different kinds of link relations, depending on the client's identity.
* Home documents ought to be assigned a freshness lifetime (e.g., "Cache-Control: max-age=3600") so that clients can cache them, to avoid having to fetch it every time the client interacts with the service.
* Custom link relation types, as well as the URIs for variables, should lead to documentation for those constructs.

###<a name="appendix-b-managing-change"></a>B.1. Managing Change in Home Documents

The URIs used in API home documents MAY change over time.  However, changing them can cause issues for clients that are relying on cached home documents containing old links.

To mitigate the impact of such changes, servers ought to consider:

* Reducing the freshness lifetime of home documents before a link change, so that clients are less likely to refer to an "old" document.
* Regarding the "old" and "new" URIs as equally valid references for an "overlap" period.
* After that period, handling requests for the "old" URIs appropriately; e.g., with a 404 Not Found, or by redirecting the client to the new URI.

###<a name="appendix-b-evolving-and-mixing"></a>B.2. Evolving and Mixing APIs with Home Documents

Using home documents affords the opportunity to change the "shape" of the API over time, without breaking old clients.

This includes introducing new functions alongside the old ones - by adding new link relation types with corresponding resource objects - as well as adding new template variables, media types, and so on.

It's important to realise that a home document can serve more than one "API" at a time; by listing all relevant relation types, it can effectively "mix" different APIs, allowing clients to work with different resources as they see fit.

##<a name="appendix-c"></a>Appendix C. Considerations for Consuming Home Documents

Clients might use home documents in a variety of ways.

In the most common case - actually consuming the API - the client will scan the Resources Object for the link relation(s) that it is interested in, and then to interact with the resource(s) referred to. Resource Hints can be used to optimize communication with the client, as well as to inform as to the permissible actions (e.g., whether PUT is likely to be supported).

Note that the home document is a "living" document; it does not represent a "contract", but rather is expected to be inspected before each interaction.  In particular, links from the home document MUST NOT be assumed to be valid beyond the freshness lifetime of the home document, as per HTTP's caching model [RFC7234].

As a result, clients ought to cache the home document (as per [RFC7234]), to avoid fetching it before every interaction (which would otherwise be required).

Likewise, a client encountering a 404 Not Found on a link is encouraged obtain a fresh copy of the home document, to assure that it is up-to-date.

##<a name="appendix-d"></a>Appendix D. Frequently Asked Questions

###<a name="appendix-d-references-or-inheritance"></a>D.1. Why doesn't the format allow references or inheritance?

Adding inheritance or references would allow more modularity in the format and make it more compact, at the cost of considerable complexity and the associated potential for errors (both in the specification and by its users).

Since good tools and compression are effective ways to achieve the same ends, this specification doesn't attempt them.

###<a name="appendix-d-errors"></a>D.2. What about "Faults" (i.e., errors)?

In HTTP, errors are conveyed by HTTP status codes.  While this specification could (and even may) allow enumeration of possible error conditions, there's a concern that this will encourage applications to define many such "faults", leading to tight coupling between the application and its clients.

###<a name="appendix-d-schema"></a>D.3. How Do I find the schema for a format?

That isn't addressed by home documents. Ultimately, it's up to the media type accepted and generated by resources to define and constrain (or not) their syntax.

###<a name="appendix-d-complex-query"></a>D.4. How do I express complex query arguments?

Complex queries - i.e., those that exceed the expressive power of Link Templates or would require ambiguous properties of a "resources" object - aren't intended to be defined by a home document. The appropriate way to do this is with a "form" language, much as HTML defines.

Note that it is possible to support multiple query syntaxes on the same base URL, using more than one link relation type; see the example at the start of the document.

##<a name="author-address"></a>Author's Address

Mark Nottingham
Email: mnot@mnot.net
URI:   https://www.mnot.net/
