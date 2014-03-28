# Fusepool P3: Platform Architectural Proposal (by SRFG)

* **Authors**: Jakob Frank, Rupert Westenthaler, Sergio Fernández
* **Last modification date**: March 28, 2014
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

From the platform perspective, each component would needs to be managed somehow. 
So we propose that, based on the Adapter design pattern, each component shall 
implement a simple Java API (TBD) that allows to wrap the actual implementation 
behind the interaction with the actual business logic. We expect most of the components 
would be simple wrappers over other REST web services; but at the same time we allow 
the implementation of more complex interaction models without increasing the complexity 
of the platform.

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
|                        | /p3/transformers/_t1_         | `GET`   | `200`  | Get RDF description (vocab to be defined), including all available configurations.
|                        |                               | `POST`  | `201`  | Creates a instance of the transformer with the configuration in the body (in the case of BatchRefine the JSON script, XSLT for the Sponger, etc). The `Slug` header indicates name preference. `Location` returns the created endpoint.
|                        |                               |         | `400`  | The transformer does not accept the configuration submitted.
|                        | /p3/transformers/_t1_/_c1_    | `GET`   | `200`  | Return the raw configuration of the instance. `Link` header to the transformer it belongs to.
|                        |                               | `POST`  |        | Sending the original data in the request body, or by reference using [RFC2017](http://tools.ietf.org/html/rfc2017) (using the header `Content-type: message/external-body; access-type=URL; URL="http://example.org/file.csv"`). Two processing options would be offered:
|                        |                               |         | `200`  | a) **Synchronous**: with header `Prefer: return="representation"` will return the transformed data, no server-side storage (see [Prefer Header for HTTP](http://tools.ietf.org/html/draft-snell-http-prefer-18#section-4)).
|                        |                               |         | `202`  | b) **Asynchronous**: with header `Prefer: respond-async` the transformation will be executed in background, returning a `Location` header to the job.
|                        |                               |         | `400`  | The submitted data cannot be transformed.
|                        |                               |         | `500`  | An error has occurred during transformation (when synchronous).   
| Enhancers              | /p3/enhancers                 |         |        | TBD 
| Jobs                   | /p3/jobs                      | `GET`   | `200`  | List all running jobs (TBD). 
|                        | /p3/jobs/_j1_                 | `GET`   | `200`  | Current status while the job is running. 
|                        |                               |         | `303`  | Once the job has been complete, it redirects to the result (`Location` header). 
|                        |                               |         | `500`  | Job execution error.
| Results                | /ldp/...                      | _*_     | _*_    | Result (Regular LDP interaction). 

# Open issues

* Assess if exactly the same model proposed for the transformers could fit with the enhancers too.
* How to represent transformers which do not require configuration (i.e., direct mapping).
* Compatible alternatives for the Java API for other runtimes.
* How transformer which require user interaction would fit in this model?
* Responsible for components orchestration is nor clearly defined in the proposal. Taking the previous point as context, it could be a simple UI (then WP4 is responsible). But could be something else.

