# Fusepool P3: Platform Architectural Proposal (by SRFG)

* **Authors**: Jakob Frank, Rupert Westenthaler, Sergio Fernández
* **Last modification date**: March 27, 2014
* **Status**: This documents is just a proposal, a **draft** to discuss with the [Fusepool P3](http://www.fusepool.eu/p3) consortium; most of the points described in this document are just ideas that need to be proven.

## Motivation

The proposal is build around the _data value chain_ concept. Where it aims to provide a 
set of integrated software components that provide a dynamic data value chain capable of 
process, enhance and publish the data. Let's try to arrive to a common understanding of it.

The [architectural principles](https://github.com/fusepoolP3/overall-architecture/blob/master/architectural-principles.md)
outlined by Reto envisions a complete interaction model based fully based on HTTP, 
either LDP, SPARQL or custom REST APIs. But there a several aspects on how component 
would interact in the project, both with the platform and between themselves, 
that requires a more detailed analysis. The motivation is to try to get closer to Reto's 
idea, and to assess some of the concepts.

This document aims to provide a suitable proposal to implement the platform. In 
this document we are trying our best to anticipate the functional requirements 
from the use cases, as well of trying to be a much compatible with the technological 
stack of the consortium. Probably few points will be actually right, and many other 
will require further discussion and agreement, but at least is good to continue 
discussing

In addition, this document should also provide the fundamentals for the Deliverable 
D5.1, scheduled to be delivered by M6 of the project (June 2014).

## Components
**TODO:** What are "Components"
### Components implementation

The intermediate results from the [current sprint](https://easybacklog.com/accounts/4748/backlogs/54217#2) 
arise that the simple concepts proposed to 
[interact with the extractors](https://github.com/fusepoolP3/overall-architecture/blob/master/data-extractor-importer-api.md)
wouldn't be enough: OpenRefine [requires several calls](https://docs.google.com/a/spaziodati.eu/document/d/18Dup7hT2DMMCK6MP8IpnKSEM9YjKSY-cVTjC36P9_Kw), 
even pull or callbacks; Virtuoso Sponger implements something completely different,
where data is stasteful stored. Therefore the interaction will be more complex
in many scenarios.

\[OpenLink\]
_The current Sponger works statefully. Given a resource URL, the generated RDF is typically saved in a graph with a URL matching the resource URL. However we are working on an alternative stateless Sponger service to return the generated RDF directly without persisting it in a graph._

From the platform perspective, each component would needs to be managed somehow. 
So we propose that, based on the Adapter design pattern, each component shall 
implement a simple Java API (TBD) that allows to wrap the actual implementation 
behind the interaction with the actual business logic. We expect most of the components 
would be simple wrappers over other REST web services; but at the same time we allow 
the implementation of more complex interaction models without increasing the complexity 
of the platform.

\[OpenLink\]
_With Virtuoso not being Java-based, the platform would need to provide Java adapters to serve as proxies wrapping HTTP Virtuoso Sponger service endpoints_

On the contrary to the [RESTful interaction proposed](https://github.com/fusepoolP3/overall-architecture/blob/master/data-extractor-importer-api.md),
this approach is a compromise between the functionality and effectiveness. It tries
to handle in a more effective way the components registration by removing some 
unnecessary overhead, that at the same time would potentially introduce points of 
failure.

### Components types

In principle we would have two types of components managed by the platform:

* transformers (WP2)
* enhancers (WP3)

but this list can be extended if necessary.

### Components registration

From our point of view these management operations do not need to be done following
Linked Data principles. Linked Data was never design for such tasks, so it offers
a poor expressiveness with an elevated overhead. Instead we propose to use any native 
mechanism available for solve this non-functional requirement in a effective way.

In ([Marmotta](http://marmotta.apache.org)), we will provide a simple 
Java API for the components, that will be (auto-)registered in the platform via common mechanism, 
e.g. [ServiceLoader](http://docs.oracle.com/javase/7/docs/api/index.html?java/util/ServiceLoader.html). This would provide a components' runtime with a very small effort,
that could be refined in future iterations.

Internally, based on the defined Java API, the platform can optional create
RDF descriptions that would be published following the Linked Data principles. 
This meta-data can be used for further steps of the workflow (see the details in the description 
of the REST API). 

### Components configuration

Each component is free to accept or reject the submitted configuration. See further details 
in the description of the REST API.

### Components orchestration

Now that the platform provides, in theory, a set of components usable from a clean API,
a remarkable question comes: who (which WP) is responsible of the components orchestration?
For instance, which transformed call, which enhancer fits better for this concrete data, 
and so on. This issue has appear several times in the last meetings, but the DoW does not
provide a adequate answer. 

## APIs

### Java API

TBD (e.g. a concept similar to
[what we have](https://git-wip-us.apache.org/repos/asf?p=marmotta.git;a=blob;f=libraries/ldclient/ldclient-api/src/main/java/org/apache/marmotta/ldclient/api/provider/DataProvider.java;h=5d0fda3d01dce4b5349c16e8df6097165bb873cb;hb=develop#l35) in [LDClient](http://marmotta.apache.org/ldclient))

### REST API

Here an initial draft how the REST API could look like:

| Scope                  | Endpoint                      | Method  | Status | Description
| ---------------------- | ----------------------------- | ------- | ------ | -----------------------------------------------------------------------------------------------------
| Global                 | /p3                           |         |        | TBD
| Transformers           | /p3/transformers              | `GET`   | `200`  | List all available transformers.  
|                        | /p3/transformers/_t1_         | `GET`   | `200`  | Get RDF description (vocab to be defined), including all available configurations. \[OPL1\]              
|                        |                               | `POST`  | `201`  | Creates a instance of the transformer with the configuration in the body (in the case of BatchRefine the JSON script, XSLT for the Sponger, etc). The `Slug` header indicates name preference. `Location` returns the created endpoint. \[OPL2\] \[OPL3\] \[OPL4\]
|                        |                               |         | `400`  | The transformer does not accept the configuration submitted.
|                        | /p3/transformers/_t1_/_c1_    | `GET`   | `200`  | Return the raw configuration of the instance. `Link` header to the transformer it belongs to.
|                        |                               | `POST`  | `200`  | **Synchronous**: with header `Prefer: return="representation"` will return the transformed data, no server-side storage (see [Prefer Header for HTTP](http://tools.ietf.org/html/draft-snell-http-prefer-18#section-4)). \[OPL5\] \[OPL6\] \[OPL7\]
|                        |                               |         | `202`  | **Asynchronous**: with header `Prefer: respond-async` the transformation will be executed in background, returning a `Location` header to the job.
|                        |                               |         | `400`  | The submitted data cannot be transformed.
|                        |                               |         | `500`  | An error has occurred during transformation (when synchronous).   
| Enhancers              | /p3/enhancers                 |         |        | TBD 
| Jobs                   | /p3/jobs                      | `GET`   | `200`  | List all running jobs (TBD). 
|                        | /p3/jobs/_j1_                 | `GET`   | `200`  | Current status while the job is running. 
|                        |                               |         | `303`  | Once the job has been complete, it redirects to the result (`Location` header). 
|                        |                               |         | `500`  | Job execution error.
| Results                | /ldp/...                      | _*_     | _*_    | Result (Regular LDP interaction). 

\[OPL1\]
_Each Sponger extractor (i.e. transformer) cartridge could be made to return an RDF description of itself._
 
 \[OPL2\]
_The Sponger cartridges are not configurable to this degree. Whilst certain configuration options, either generic or specific to a particular extractor, may be supported, having the ability to set the XSLT for the extractor is not practicable. Each cartridge comprises logic (written in Virtuoso PL) and an XSLT stylesheet. The name of the stylesheet is currently hardcoded in the cartridge logic. The XSLT stylesheets are handcrafted to capture the RDF mapping for a particular data source. Is it likely that users will want to craft and dynamically instantiate their own stylesheets?_

 \[OPL3\]
 _Dynamically creating extractor service endpoints on request at scale in Virtuoso may not be practicable. Similarly with the suggestion to create separate per-request instances of a transformer. From OpenLink's side, at best we could provide a REST API in which each extractor is individually addressable, but redirect from the extractor URI to a generic extractor service endpoint where the extractor to be used is named by a query string parameter. Given that we have a large number of extractors, this would be our preferred solution._
 
 \[OPL4\]
 _Because multiple instances of a transformer are not practicable in Virtuoso, nor is the proposal to have individually configured instances. An alternative would be to provide extractor/transformer options through a POST query string._ 
 
 _A POST to a Virtuoso-hosted REST API endpoint /p3/transformers/opl-csv?baseUri=http://example.org/ could be redirected through Virtuoso's URL rewriting to a generic HTTP RDF/Linked Data generation endpoint - /ldgeneration?extractor=opl-csv&baseUri=http://example.org/
(/p3/transformers/opl-csv has been used in this example, rather than /p3/transformers/opl-csv/c1, because of the issues with creating individual instances of extractors/transformers in Virtuoso). The baseUri parameter in this example provides the base URI which Sponger extractor cartridges would use as the basis for creating entity URIs.
Similarly, a format parameter could indicate the required RDF serialization as an alternative to using an Accept: header._

_An RDF description of the CSV cartridge returned by /p3/transformers/opl-csv would include a description of the baseUri and other such transformer-specific parameters._
 
  \[OPL5\]
  _Virtuoso extractor cartridges operate synchronously. We're unlikely to support asynchronous operation._
  
  \[OPL6\]
  _Could header `Prefer: return=minimal` be used to support a stateful option in which, rather than returning the generated RDF directly, the transformer saves the generated RDF, e.g. to a graph? The Content-Location header could then be used to identify the URI of the resource containing the generated RDF, or a proxy URI which returns the graph contents. return=representation and return=minimal would then allow support of both stateless and stateful transformation._
  
  \[OPL7\]
_Rather than only allowing the POSTing of content to be transformed, it may also be desirable to allow an agent to POST the URL of a resource/document to be transformed. (This is primarily how the Virtuoso Sponger works at present.) The document URL and other transformer options could, for instance, be encoded as application/x-www-form-urlencoded._
  

# Open issues

* Assess if exactly the same model proposed for the transformers could fit with the enhancers too.
* How to represent transformers which do not require configuration (i.e., direct mapping).
* Compatible alternatives for the Java API for other runtimes.
* How transformer which require user interaction would fit in this model?
* Responsible for components orchestration is nor clearly defined in the proposal. Taking the previous point as context, it could be a simple UI (then WP4 is responsible). But could be something else.

