# Data transformation

This document defines an API for data transforming components. The term "transform" and the derived terms are sued very broadly here and they include processes auch as annotating and lifting content.

##Transformers

An Transformation Service is represented by an dereferenceable URI representing a resource of type trans:Transformer

A GET request of the resource will return a description of the service. At least text/turle must be supported as format to describe the resource. A POST request does the actual transformation of the data.

This is a very generic menchanism designed to interface simple services that can do a very specific transformation as well as services that interface a complex pipe or routing mechanism to handle many input and output formats. Such a more complex service might well delegate to more simple services that also expose the interface.

### Example 1
Retrieving the description of the transformer `http://example.org/simple-transformer`

    GET /simple-transformer
    Host: example.org
    Accept: text/turte, application/rdf+xml;q=.8, */*;q=.1
    User-Agent: my-client/0.1

Response:

    HTTP/1.1 200 OK
    Date: Fri, 21 March 2014 10:55:12 GMT
    Connection: close
    Content-Type: text/turtle

	@prefix dct: <http://purl.org/dc/terms/>.
    @prefix trans: <http://vocab.fusepool.info/transformer#>.
    <http://example.org/simple-transformer> a trans:Transformer;
		dct:title "A simple rdf Transformation"@en;
		dct:description "transforms vcards to RDF";
		trans:supportedInputFormat "text/vcard";
		trans:supportedOutputFormat "text/turtle";
		trans:supportedOutputFormat "text/ld+json".

This response tells the client that this is an Transformation service accepting text/vcard and able to produce text/turtle or text/ld+json.

Parameters of the media type might further narrow the format using media type parmeters, e.g. application/json;app=foobar%version=2.1. It the input format is specified with parameters the submitted data format must be qualified with all these parameters, the submitted data format may be qualified with additional parameters.

The media type might also contain wildcars, analogously to the accept header in HTTP.

#### Open issues:

- Using hydra?
- Should it be possible to use wildcars when specifying the input format (e.g. for a service routing the requests based on their content type)?

### Example 2
Transorming data using the transformer `http://example.org/simple-transformer`

    POST /simple-transformer
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
Transforming data using the asynchronous transformer `http://example.org/asynchronous-transformer`

Request 1

    POST /asynchronous-transformer
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
    Location: /asynchronous-transformer/status/765
    Connection: close

The response tells the client that the request seems to be correct and gives the client a URI from which the status and eventually the result can be retrieved.

Request 2

    GET /asynchronous-transformer/status/765
    Host: example.org
    Accept: text/turte, application/rdf+xml;q=.8, */*;q=.1
    User-Agent: my-client/0.1

Response 2:

    HTTP/1.1 202 ACCEPTED
    Date: Fri, 21 March 2014 10:59:18 GMT
    Connection: close
    Content-Type: text/turtle

    @prefix trans: <http://vocab.fusepool.info/transformer#>.
    <> trans:status trans:Processing.

The request was accepted and the body of the message tells the client that the status if still `trans:Processing`.

Request 3

    GET /asynchronous-transformer/status/765
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

### Remarks

The asynchronous transformer does not need to be a separate transformer from its synchronous counterpart. It can be the same transformer from the user perspective, i.e. have the same URI. This transformer can then support both sync and async behaviour. The exact behaviour can be triggered based on the lenght of the POSTed content, for example.

It is undefined by this specification how long an Transformation result shall remain available for retrieval. The transformer should not delete the Transformation result on the first GET request, repeating the GET request shortly after should yield to the same results. Typical the time a result is available depends on the required processing time. The result of a job that took only a few seconds might remain available for several minutes, the result of a job that took several hours might remain available for several days. When a result is no longer available the server returns status code 404. A restart of the transformer will typically cause the Transformation results to become unavailable. Even though some transformers might keep the result available permantently, clients should never rely on this.


### Open issues

- Name hints: can one give name hint for the extracted resource?
