# rest

Welcome...!

This is a set of guidelines for building a RESTful server.

* **Start Here -->** The [Guidelines](/guidelines.md) attempt to address the first five constraints from the Representational State Transfer (REST) dissertation by Fielding: Starting with the Null Style, Client-Server (CS), Stateless, Cache and Uniform Interface. Included in the guidelines are pointers to additional items:
  * The [Hypermedia as the Engine of Application State model](/hateoas-model-example.md) builds out the work of [Stefan Tilkov](https://twitter.com/stilkov) from his excellent talk [GOTO 2014 • REST: I don't Think it Means What You Think it Does](https://www.youtube.com/watch?v=pspy1H6A3FM). You can find my summary of his slides [here](https://github.com/retrosight/learning/blob/master/REST-I-dont-think-it-means-what-you-think-it-does-stefan-tilkov.md).
  * [Resources and Representations](/resource-and-representation.md) provides an example of how there can be one resource with many representations as well as how to bring together data from two distinct resources into a single representation for client code to use.
  * Hyperlinks are rather ubiquitous in REST and it can get confusing so [Comparing Hyperlinks: Identifiers, Related Data and Hypermedia As The Engine Of Application State](/d-related-data-hateoas.md) attempts to disambiguate for the reader.
  * [Pagination Design] elaborates on how the server controls pagination rather than putting it in the hands of the client.
  * [JSON Schema](/schema) resources which are used in the design.
* Also in the repository:
  * [Design Thoughts](/design-thoughts.md) - A collection of mostly readings from folks who influenced the overall design you find here.
  * [Documentation Template](/documentation-template.md) - Lists out all the basics an API should cover within its documentation and it includes all of the HTTP headers and status codes as hyperlinks to the various RFCs that cover same.
  * JSON Web Tokens are becoming more commonplace for access tokens included in an Authorization Header -- this is a TL;dr summary of a JWT.

MIT License

Copyright (c) 2016 Charlie Owen

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
