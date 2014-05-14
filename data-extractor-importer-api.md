# Data extractor and importer with extraction

D2.1) Data importer with data extraction tools implemented: PDF Report and software. The prototype deliverables will always be accompanied by appropriate documentation for their development and for their usage. In addition, prototype deliverables need to ensure integration with other prototype deliverables in order to maximise the value of that prototype. [OGL, M12, T2.1+2.2] [month 12]


##Data-Extraction tool

An Extraction Service is represented by an dereferenceable URI representing a resource of type fp:ExctractionService

A GET request of the resource will return a description of the service. At least text/turle must be supported as format to describe the resource. A POST request does the actual transformation of the data.

This is a very generic menchanism designed to interface simple services that can do a very specific transformation as well as services that interface a complex pipe or routing mechanism to handle many input and output formats. Such a more complex service might well delegate to more simple services that also expose the interface.

### Example 1
Retrieving the description of the extractor `http://example.org/simple-extractor`

    GET /simple-extractor
    Host: example.org
    Accept: text/turte, application/rdf+xml;q=.8, */*;q=.1
    User-Agent: my-client/0.1

Response:

    HTTP/1.1 200 OK
    Date: Fri, 21 March 2014 10:55:12 GMT
    Connection: close
    Content-Type: text/turtle

	@prefix dct: <http://purl.org/dc/terms/>.
    @prefix fp: <http://fusepool.eu/ontology/p3#>.
    <http://example.org/simple-extractor> a fp:ExctractionService;
		dct:title "A simple rdf extraction"@en;
		dct:description "transforms vcards to RDF";
		fp:supportedInputFormat "text/vcard";
		fp:supportedOutputFormat "text/turtle";
		fp:supportedOutputFormat "text/ld+json".

This response tells the client that this is an extraction service accepting text/vcard and able to produce text/turtle or text/ld+json.

Parameters of the media type might further narrow the format using media type parmeters, e.g. application/json;app=foobar%version=2.1. It the input format is specified with parameters the submitted data format must be qualified with all these parameters, the submitted data format may be qualified with additional parameters.

The media type might also contain wildcars, analogously to the accept header in HTTP.

#### Open issues:

- Namespace of fp ontology
- Using hydra?
- Should it be possible to use wildcars when specifying the input format (e.g. for a service routing the requests based on their content type)?

### Example 2
Transorming data using the extractor `http://example.org/simple-extractor`

    POST /simple-extractor
    Host: example.org
    Accept: text/turte, application/rdf+xml;q=.8, */*;q=.1
    User-Agent: my-client/0.1
    Content-Type: text/vcard
    Content-Length: 164

    BEGIN:VCARD
    VERSION:4.0
    FN:Corky Crystal
    NICKNAME:Corks
    TEL;TYPE=home,voice;VALUE=uri:tel:+61755555555
    EMAIL:corky@example.com
    REV:20080424T195243Z
    END:VCARD

Response:

    HTTP/1.1 200 OK
    Date: Fri, 21 March 20014 10:59:12 GMT
    Connection: close
    Content-Type: text/turtle

    @prefix vcard: <http://www.w3.org/2006/vcard/ns#> .
    @prefix rdfa: <http://www.w3.org/ns/rdfa#> .
    [] a vcard:Individual;
      vcard:hasEmail <mailto:corky@example.com>;
      vcard:fn "Corky Crystal";
      vcard:hasTelephone [ a vcard:Home,
        vcard:Voice;
        vcard:hasValue "tel:+61755555555" ];
      vcard:nickname "Corks" .

The response is an RDF representation of the submitted VCard content.


### Example 3
Transforming data using the asynchronous extractor `http://example.org/asynchronous-extractor`

Request 1

    POST /asynchronous-extractor
    Host: example.org
    Accept: text/turte, application/rdf+xml;q=.8, */*;q=.1
    User-Agent: my-client/0.1
    Content-Type: text/vcard
    Content-Length: 164

    BEGIN:VCARD
    VERSION:4.0
    FN:Corky Crystal
    NICKNAME:Corks
    TEL;TYPE=home,voice;VALUE=uri:tel:+61755555555
    EMAIL:corky@example.com
    REV:20080424T195243Z
    END:VCARD

Response 1:

    HTTP/1.1 202 ACCEPTED
    Date: Fri, 21 March 2014 10:59:12 GMT
    Location: /asynchronous-extractor/status/765
    Connection: close

The response tells the client that the request seems to be correct and gives the client a URI from which the status and eventually the result can be retrieved.

Request 2

    GET /asynchronous-extractor/status/765
    Host: example.org
    Accept: text/turte, application/rdf+xml;q=.8, */*;q=.1
    User-Agent: my-client/0.1

Response 2:

    HTTP/1.1 202 ACCEPTED
    Date: Fri, 21 March 2014 10:59:18 GMT
    Connection: close
    Content-Type: text/turtle

    @prefix fp: <http://fusepool.eu/ontology/p3#>.
    <> fp:status fp:Processing.

The request was accepted and the body of the message tells the client that the status if still `fp:Processing`.

Request 3

    GET /asynchronous-extractor/status/765
    Host: example.org
    Accept: text/turte, application/rdf+xml;q=.8, */*;q=.1
    User-Agent: my-client/0.1

Response 3:

    HTTP/1.1 200 OK
    Date: Fri, 21 March 2014 11:00:12 GMT
    Connection: close
    Content-Type: text/turtle

    @prefix vcard: <http://www.w3.org/2006/vcard/ns#> .
    @prefix rdfa: <http://www.w3.org/ns/rdfa#> .
    [] a vcard:Individual;
      vcard:hasEmail <mailto:corky@example.com>;
      vcard:fn "Corky Crystal";
      vcard:hasTelephone [ a vcard:Home,
        vcard:Voice;
        vcard:hasValue "tel:+61755555555" ];
      vcard:nickname "Corks" .

Eventually the client gets a 200 Response code with the RDF representation of the submitted VCard content as body.


## Data-Importer tool

For importing data to be transformed on import the resources to be transformed are added to a specially marked LDPC. When requesting the LDPC a triple with the LDPC as subject `fp:extractor`as property and an instance of `fp:ExctractionService` as object is returned in the returned graph. When a non-RDF resource is posted against that container the extraction service will asynchronously be invoked. The results of the extraction will be added to the same collection. A triple with predicate `fp:extractedFrom`point from the extracted resource to the non-RDF resource.

### Example 4

    GET /container1 HTTP/1.1
    Host: example.org
    Accept: text/turtle; charset=UTF-8
    Prefer: return=representation; include="http://www.w3.org/ns/ldp#PreferEmptyContainer"

Response

    HTTP/1.1 200 OK
    Content-Type: text/turtle; charset=UTF-8
    ETag: "_87e52ce2917987"
    Content-Length: 477
    Link: <http://www.w3.org/ns/ldp#Container>; rel="type"
    Preference-Applied: return=representation

    @prefix dcterms: <http://purl.org/dc/terms/>.
    @prefix ldp: <http://www.w3.org/ns/ldp#>.
    @prefix fp: <http://fusepool.eu/ontology/p3#>.

    <http://example.org/container1/>
       a ldp:DirectContainer;
       dcterms:title "An extracting LDP Container using simple-extractor";
       ldp:membershipResource <http://example.org/container1/>;
       ldp:hasMemberRelation ldp:member;
       ldp:insertedContentRelation ldp:MemberSubject;
       fp:extractor <http://example.org/simple-extractor>.

The above container will process added resources using `http://example.org/simple-extractor` if their media-type matches one of the supported input formats of this extractor. If this is the case both the original resource as well as the extraction results will be added to the container.

### Open issues

- Name hints: can one give name hint for the extracted resource?
