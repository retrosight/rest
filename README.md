# Representational State Transfer (REST) Guidelines

> [Find the website at https://retrosight.github.io/rest/](https://retrosight.github.io/rest/)

Welcome...!

This is a set of guidelines and resources for building a server that can participate in a REST paradigm.

* The [Guidelines](./guidelines.md) leverage the Representational State Transfer (REST) dissertation by Fielding to craft an API accounting for the Null Style, Client-Server (CS), Stateless, Cache and Uniform Interface constraints.
* [Service Index](./service-index.md) documents provide a fundamental starting place for the programmable internet like home pages do at root URLs for the World Wide Web used by humans with browsers.
* The [Hypermedia as the Engine of Application State Model](./hateoas-model-example.md) builds out the work of [Stefan Tilkov](https://twitter.com/stilkov) from his excellent talk [GOTO 2014 • REST: I don't Think it Means What You Think it Does](https://www.youtube.com/watch?v=pspy1H6A3FM). You can find my summary of his slides [here](https://github.com/retrosight/learning/blob/master/REST-I-dont-think-it-means-what-you-think-it-does-stefan-tilkov.md).
* [Resources and Representations](./resource-and-representation.md) provides an example of how there can be one resource with many representations as well as how to bring together data from two distinct resources into a single representation for client code to use.
* Hyperlinks are rather ubiquitous in REST and it can get confusing so [Comparing Hyperlinks: Identifiers, Related Data and Hypermedia As The Engine Of Application State](./id-related-data-hateoas.md) attempts to disambiguate for the reader.
* [Pagination](./pagination-design.md) elaborates on how the server controls pagination rather than putting it in the hands of the client.
* [Search](./search.md) is an important way an API allows client code to discover resources based on criteria.
* [JSON Schema](/schema) resources which are used in the design -- see http://json-schema.org/ for more information on this emerging standard.
* Also in the repository:
  * [A Day of REST](./A-Day-Of-REST.pdf) is a set of slides used in a five hour course I teach. It's very useful as an introduction to the key concepts and constraints of REST design.
  * [Design Thoughts](./design-thoughts.md) - A collection of mostly readings from folks who influenced the overall design you find here.
  * [Documentation Template](./documentation-template.md) - Lists out all the basics an API should cover within its documentation and it includes all of the HTTP headers and status codes as hyperlinks to the various RFCs that cover same.
  * JSON Web Tokens are becoming more commonplace for access tokens included in an Authorization Header -- find a `TL;dr` explanation within [JWT Summary](./jwt-summary.md).

---

Please note that this project is released with a [Contributor Code of Conduct](./code-of-conduct.md). By participating in this project you agree to abide by its terms. Learn more at [https://www.contributor-covenant.org/](https://www.contributor-covenant.org/).

---

MIT License

Copyright (c) 2018 Charlie Owen

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
