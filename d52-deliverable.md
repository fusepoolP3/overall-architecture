# Data retrieval for semantic enrichment/processing and semantic indexes for domain specific data retrieval

Deliverable D5.2

## Documentation Information


* *Deliverable Nr Title*: D5.2 Data retrieval for semantic enrichment/processing and semantic indexes for domain specific data retrieval
* *Lead*: Reto Gmür (BUAS)
* *Authors*: Reto Gmür (BUAS)
* *Publication Level*: Public

### Document Context Information

* *Project (Title/Number)*: Fusepool P3 (609696)
* *Work Package / Task*: WP5 / T5.3 & T5.4
* *Responsible person and project partner*: Reto Gmür (BUAS)

### Quality Assurance / Review

[TBD]


### Official Citation

Fusepool-P3-D5.2

### Copyright

This document contains material copyright by certain
Fusepool P3 consortium parties.

This work is licensed under the Creative Commons Attribution 4.0 International License. To view a copy of this license, visit http://creativecommons.org/licenses/by/4.0/.

## Executive Summary

This deliverable covers the interaction of Fusepool P3 with external non-human resources and actors. It 
is thematically and technologically adjacent on one side to D4.1 which covers the interaction with human users and which uses the services provided by this deliverable and on the other side with D5.3 which provides the generic data storage/access RDF API and is a constituent part of the P3 Platform Architecture (D5.1).

The data retrieval for semantic enrichment supports both components delivering the data to the platform, notably GUI components allowing to upload files as well as components harvesting data from external sites. The data retrieval is tightly integrated with the data transformation components provided by Work Package 2 which uses the same RESTfull RDF API as the components performing the semantic enrichment provided by Work Package 3.

The components involved are tightly integrated in that they provide a single entry points hiding away all the complexity of the various processing steps. On the other hand the components are loosely coupled which does not just improve stability and flexibility but also greatly enhance debugging possibilities by having a very small set of well defined and monitorable points of interactions. Throughout the refinement, enrichment and processing lifecycle the data passes interfaces defined by the P3 Transformer API that allow to monitor the state of the processing so that corrective measures can be taken if needed.

With T5.4 this deliverable comprises the provision of application level retrieval and query services. As specified by the Platform Architecture wherever possible the interaction between data consuming and the data-storing components shall follow standard protocols and interaction patterns, most notably the ones defined by the Linked Data Platform Specification [LDP]. The architecture foresees T5.4 components to allow some application specific access interfaces provided by services interacting directly with the underlying triple store as opposed to using the standard interactions APIs. As shown by D4.1 the generic API did not needed to be extended with non-standard custom interfaces to accommodate the requirements of the client components. In one case however it was necessary to switch from the originally envisaged LDP interaction model to the SPARQL query language and protocol as this offered a far better performance. The fact that no specific interfaces needed to be introduced proves the versatility of the chosen standard protocols and greatly increases flexibility as well as protection of the investments of the organizations deploying the P3 platform as it allows them to freely choose between standard compliant backends.

## Abbreviations

| Acronym/Abbreviation | Description |
|----------------------|-------------|
| DoW | Description of Work |
| API     | Application Programming Interface            |
| LDP                 | Linked data platform |
| LDPC                | Linked data platform Container |
| PHP |  PHP hypertext Preprocessor |
| RDF     | Resource Description Framework               |
| REST    | Representational State Transfer              |
| SPARQL        | SPARQL Protocol And RDF Query Language |                                                                                                                                                                                                              


## Introduction

The purpose of this deliverable it to provide both for the data retrieval, i.e. allow to retrieve the data that shall be semantically enriched, as well as for the services providing domain/application level views on datasets. 

The data retrieval for semantic enrichment supports both components delivering the data to the platform, notably GUI components allowing to upload files as well as components harvesting data from external sites. The components delivering content to the platform use standard interaction mechanisms to add contents to a Transforming LDPC as this has been defined in the Platform Architecture D5.1. While D5.1 describes the functional role as well as the protocols supported by the P3 Proxy this report shall focus on the implementation of this crucial mediator between data retrieval and the data processing leading to queryable semantic content. The Transforming LDPC architecture presupposes an agent adding data to the container. Such an agent can be a GUI component allowing the user to add files to an LDPC of the workspace or the crawler that retrieves remote resources at regular intervals. 

On the side of the application level retrieval and query service the architecture of the platform foresees the possibilities of services providing a custom HTTP API that interact directly with the triple store. As written in the D5.1 report, such services are only implemented if the development of the UI shows that an interaction via the standard mechanism SPARQL and LDP is not feasible or not providing an adequate level of performance. We will see that a change of the standard protocol could avoid the introduction of such service, increasing the portability of the platform across storage backends. 

## Positioning within the project life span

The DoW has foressen this deliverables for project month 9. However the chosen architecture described in D5.1/5.3 emphasizises on using pre-existing standards wherever possible and writes about the domain specific query API and services foreseen by T5.4 (one of the two tasks comprising this deliverable) that "such services are only implemented if the development of the UI shows that an interaction via the standard mechanism SPARQL and LDP is not feasible or not providing an adequate level of performance". Defining and implementing such service at an early project stage would be a typical example of premature optimization. It was thus beneficial to postpone this task to a late stage in the project so that the application specific services could be defined to address empirically observed rather than be based on merely hypothized problems.
As for the other task of this deliverable T5.3 recognizing that the chosen modular achitecture offers plenty of interfaces suitable for adding the required debugging and backtracing possibilities we opted to develop the required tools based on the concrete requirements that arise during the product development. As an unforseen dividend of the standard compliance we found that standard tools can be used and no development was necessary. This shows that it was a good decision not to develop tools beforehand, which would have not only been futile but also an unecessary addition of complexity to our software corpus.
As postponenment of both task of this deliverable was indicated in our views we consequently decided to postpone this deliverable.

## Data retrieval

### Proxy

The functional role and relevant APIs for the LDP Transforming Proxy are defined in D5.1. This deliverable report shall provided some more details on how the proxy has been implemented and how it is used.

### Crawler

The integration of the Virtuoso Crawler is described in detail in D2.1 and shall not be repeated here. The proxy is a service to efficiently retrieve data for storage and semantic enrichment with the P3 platforrm.

### Monitoring data

Originally we planned to implement a logging no-operation transformer for debugging as part of T5.3. This no-op transformer could have been added at any position of a transforming pipeline to log the data passed from one transformer to the next. The no-op transformer was thought to be useful to monitor the data at each of its transformation steps. Experience has shown however that standard TCP/IP network monitoring tools where sufficient in most cases. The only case we encountered where standard TCP/IP monitoring tool were not powerful enough was when debugging a scenario where different pipelines that are run concurrently access the same transformer, but even in this case we did not need the planned logging no-operation transformer but could use a standard logging HTTP Proxy such as The Grinder [Grinder], given that the transformer API is based on HTTP and the RDF data exchange bases on standard serialization formats no further tool was needed. This finding clearly exemplifies the benefits of the chose standards based architecture with loosely coupled components communicating vie HTTP and standard format.

### Dashboard

The Dashboard allows the user to visually interact with transforming containers (defined in D5.1). The dashboard supports multiple configurations effectively allowing the user to have multiple visual workbenches to serve different usage scenario of one or of several users. With the Dashboard the user can to drag-and-drop files to the platform, as defined by the Transforming Container API these files are not only added to the container but a transformation job is also started in the background. The user will immediately see and be able to access the uploaded data (i.e. the file drag-and-dropped to the visual representation of the container and shortly after the transformation compleeted also the results of the transformation. The dashboard is described in detail in D4.1.

## Application level retrieval

With T5.4 the DoW foresees the development of application level retrieval and query services. The platform architecture described in D5.1 stipulates that interaction with the platform by non-human agents as well as interactions between platform components shall be based on either established protocols and standards for interacting with linked data, specifically:

 * LDP
 * SPARQL

As well as protocls defined by the P3 project. The protocols defined within the project shall follow the REST design principles. Notably the following protocols where defined within P3:

 * Transformer API
 * Transforming Container API
 * Transformer Registry API
 * User Interaction Request API
 
Apart from there adherence to the REST principles and (except for the first) relying on the LDP specification what these protocols have in common is that the payload of the messages (the HTTP message body) is expressed using a media type encoding an RDF graph (or, where the LDP specification foresees this, a serialization that will become an RDF graph once the LDP instance assigns the necessary base IRI for parsing). To foster interoperability at the semantic level the platform architecture also defines which ontologies shall be used for the purpose of describing semantic enrichments, notably:

 * Open Annotation [Sanderson2013]
 * Fusepool Annotation Model (FAM)
 
These standards and API describe generic and domain independent data retreival and processing mechanism. The use of standard ontologies for describing annotations allows for text/label based retrieval using the standard SPARQL protocol. Where the backend supports GeoSPARQL [Perry] it is also possible to run more advanced topological queries using the SPARQL protocol. It should however be noted that simple bounding-box based spatial/temporal queries can be run efficiently against any SPARQL endpoint without requiring specific extensions.

The expected price of genericy is a degradation of performance as well as a higher complexity required by the client software. While the services developed within T5.4 should if possible also follow the overall design patterns, i.e. provide RESTfull interfaces with RDF payloads they should be designed for specific domains and applications. This would allow to optimize performance as well as to provide the simplest possible API for the use case at hands. As written earlier we would be guilty of premature optimization if we had defined such APIs before having concrete empirical facts on shortcomings of the generic APIs. The necessity and the content of the services implemented withing T5.4 thus depends on the results of T4.1 which shall identify possible optimizations of the genric APIs based on the development of clients.

### Findings from T4.1

As described in report for D4.1 the generic APIs could readily be used and tested in the creation of the UI components. In some cases however the UI implementation was changes to use the platform's SPARQL endpoint rather than accessing the data via LDP as originally foreseen and implemented, this is discussed in some more details in the next section. Only some minor adaptations to the generic APIs where conducted based on the expirience with the UI development.

 * Because of a limitation in PHP is is not possible to send nn HTTP response with status code 202 and a `location` header, therefore it was not possible to implement spec comformant implementation of asynchronous transformers using the PHP programming language. Because of this the Transfomer API was modified to allow the use of a 201 status code in place of the semantically more accurate 202 status code. The client libary was adapted to support this novelty. As the novetlty describes only an optional and its usage discourage no adpatation of the transfomer implementation library for Java was needed.
 * As many clients are implemented in JavaScript and running in the browser the same-origin policy [SPO] applies. This policy is a security mechanism implemented by all modern browsers preventing scripts in a web page to access data and services on servers other than the "origin" server which server the web page. Luckily with the Cross-Origin Resource Sharing (CORS) Specification [CORS] there is a standard mechanism that allows services to override the default policy and be accessible even from scripts running in pages served from different hosts. As with CORS an existing standards is available and nothing in the RESTfull APIs we defined is incompatible with this specification, no adaptation in our specification was needed. We simply changed the transfomer library and all transformer to add support for CORS. We also addressed some issues with the CORS support in Marmotta. The P3 Proxy needed no adaptation as it forwards any CORS support the proxied service provides.

### LDP vs. SPARQL

### CORS
 
-> D4.1
-> Issue against marmotta
-> Added support in transformers by support in the library


| Ref.             | Description |                                                                                                                                                                                                              
|------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| CORS | [http://www.w3.org/TR/cors/](http://www.w3.org/TR/cors/) |
| LDP | [http://www.w3.org/TR/ldp/](http://www.w3.org/TR/ldp/) |
| Grinder | [http://grinder.sourceforge.net/g3/tcpproxy.html](http://grinder.sourceforge.net/g3/tcpproxy.html) |
| Perry | Perry, M., Herring, J. (2012). OGC GeoSPARQL - A Geographic Query Language for RDF Data. Open Geospatial Consortium. http://www.opengeospatial.org/standards/geosparql |
| Sanderson2013    | Sanderson, R., Ciccarese, P., Van deSompel, H. (2013). Open Annotation Data Model. Community Draft, W3C. http://www.openannotation.org/spec/core/                                                            |
|SPO | [http://www.w3.org/Security/wiki/Same_Origin_Policy](http://www.w3.org/Security/wiki/Same_Origin_Policy) |



T5.3 – Data retrieval for semantic enrichment: The Fusepool P3 platform will provide workspaces for the
semantic enrichment process. This ensures that the original state of the data is kept throughout the enrichment
process and enables storage and retrieval of meta-information from the semantic enrichment process (e.g.
to backtrace processing steps leading to (un)wanted linking results). [SRFG, BUAS & OGL support, BUAS
coordinates integration]
T5.4 – Application level retrieval and query services: provide domain/application level views on datasets
including specialised query services. This includes spatial/temporal queries as applied by YAGO2 as well
as text/label based retrieval methods. For the later open-source search technologies like Apache Solr and
elasticsearch customize for RDF dataset will be exploited. [SRFG, OGL support, BUAS coordinates integration]