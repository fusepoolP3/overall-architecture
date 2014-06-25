# Transformer API

This document defines an API for data transforming components. The term "transform" and the derived terms are used very broadly here and they include processes auch as annotating and lifting contents.

## Conventions

This section is normative.

In this document the following CURIE-Prefix shall be used for the following URIs:

 * trans: http://vocab.fusepool.info/transformer#
 * rdf: http://www.w3.org/1999/02/22-rdf-syntax-ns#

Sections are non-normative unless otherwise specified. Unless otherwise specified subsections are normative if the containing section is normative.

The key words MUST, MUST NOT, REQUIRED, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this specification are to be interpreted as described in [RFC2119].

## Terminology

This section is normative.

 * Transformer: A transformer is transformation service identified and accessible via an HTTP(S) URI.
 * Implementation: An implemenation is the server processing requests against the URI of Transformers.

## Introduction

A Transformation Service (aka Transformer) is represented by an dereferenceable URI representing a resource of type `trans:Transformer`

A GET request of the resource will return a description of the service. At least text/turle must be supported as format to describe the resource. A POST request does the actual transformation of the data.

This is a very generic menchanism designed to interface simple services that can do a very specific transformation as well as services that interface a complex pipe or routing mechanism to handle many input and output formats. Such a more complex service might well delegate to more simple services that also expose the interface.


## Transformers

This section is normative.

Implementation MUST support the GET method for Transformers. Implementations MUST accept at least requests for `text/turtle` representations. Implementations MAY support other RDF and non-RDF formats. If an acceptable request is answered with a response entity in a format serializing an RDF graph this graph must contain at least the following triples:

* A triple with the Transformer as subject, `rdf:type` as predicate and `trans:Transformer` as object.
* A triple with the Transformer as subject and `trans:supportedInputFormat` as predicate.
* A triple with the Transformer as subject and `trans:supportedOutputFormat` as predicate.

Implementations SHOULD return a triple with the Transformer as subject and `trans:supportedOutputFormat` as predicate and a media type as `xsd:String`value of the object for any media-type that might be the format of the result of a successful transformation.

Implementations must support POST requests. Implementations SHOULD accept requests entities of all media-types matching a value of one of the `trans:supportedInputFormat` properties of the Extractor contained in the RDF representation returned on GET requests when interpreing this value as `media-range` the same way as the `media-range` is for accept header values as per section 14.1 of [RFC2616].

Implementations handle the request synchronously or asynchronously. If the implementations chooses to handle the request synchonously and the transformation succeeds it MUST respond with status code 200. The result of the transformation MUST be returned as the response entity. If the request fails because of an error in the POSTed entity implementation SHOULD answer the request with status code 400 and a response entity explaining the error.

If the implementation chooses to handle the request asynchronously and the request is acceptable in that the value of the Accept-headers as the Media-Type of request entity is acceptable the implementations MUST respond with status code 202 and a Location header with a URI as value for which requests are handled by the implementation, in the follwing this URI will be referred to as JOB-URI.

As long as the transformation has not completed implemenations MUST respond with status code 202 to GET requests against the JOB-URI. The response entity should be in preferred RDF serialization in the request accept header supported by the implementation, if multiple supported RDF serializations are equally preferred the response entity SHOULD by of type `text/turtle`. The graph serialized by the response entity SHOULD contain as least a triple with JOB-URI as subject, trans:status as property and trans:Processing as object.

After the request completed successfully implementatins MUST respond to request to the JOB-URI with status code 200 for some time. The response entity MUST be the result of the transformation. If the transformation failed implemenation should respod to requests to the JOB URI with status code 500 and a response entity explaining the error. In both cases after some time implementations MAY respond with status code 404 to request to the JOB-URI.

Implementations MAY honor a `Prefer` header with value `respond-async` as well as the "Wait" preference both specified in [RFC7240]. Implementations SHOULD NOT assume that a client sending a request without specifying a respond-async preference prefer a snychrnous response.

### Examples

This section is non-normative.

#### Example 1
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


#### Example 2
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


#### Example 3
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
- Using hydra?

[RFC2119]: http://www.ietf.org/rfc/rfc2119
[RFC2616]: http://www.ietf.org/rfc/rfc2616
[RFC7240]: http://www.ietf.org/rfc/rfc2740
