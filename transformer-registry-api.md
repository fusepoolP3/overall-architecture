# Transformer Registry API

This document defines an API for maintaining a registry of Transformers. It bases on [LDP], it is typically used by UI components that want to present a list of transformers to the user.

## Conventions

This section is normative.

In this document the following CURIE-Prefix shall be used for the following URIs:

 * rdf: http://www.w3.org/1999/02/22-rdf-syntax-ns#
 * trldpc: http://vocab.fusepool.info/trldpc#

Sections are non-normative unless otherwise specified. Unless otherwise specified subsections are normative if the containing section is normative.

The key words MUST, MUST NOT, REQUIRED, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this specification are to be interpreted as described in [RFC2119].

## Introduction

It is not required for a transfomer to be registered in a registry for it to be usable as a transformer. Howver registering a transformer to one or several transformer registry advertises the transformer and UI tools that allow for example the creation of pipelines of of transforming containers can offer a better user experience thanks to these registries.

## Terms

This section is normative.

 * **Transformer**: A transformer as defined by the [TRANSFORMER API] identified by an URI.
 * **Transformer Registration**: A resource of type `trldpc:TransformerRegistration` representing an entry in a transformer registry, its most important property `trldpc:transformer` pointing to the *Transformer*.
 * **Transformer Registry Container (TR-LDPC)**: an LDP Basic Container (see [LDP]) to which *Transformer Registration*s can be added.
 
## Interaction

This section is normative.

This specification does not define how clients discover a *TR-LDPC*.

An *TR-LDPC* is an LDP Basic container that MUST conform to the requirements of [LDP] along with the following restrictions. 

 * The server MUST support HTTP POST requests against the *TR-LDPC* to create LDPR
 * The server MUST support DELETE requests against the resources created by POST requests against the *TR-LDPC"

An agent registering a transformer MUST send an HTTP POST request against a *TR-LDPC* with a request entity encoding a graph with at least one triple with `trldpc:transformer` as predicate and the entity identified by a null relative URI as subject and the *transformer* as object, the graph MUST NOT contain any other triple with this combination of subject and predicate. The newly created resource is the  *transformer registration*.

To remove a registration an agent sends an HTTP DELETE request against the URI of the *transformer registration*. When the server receives an HTTP DELETE request against the URI of the *transformer registration* the resource MUST be removed from the *TR-LDPC*.

## Example

The following HTTP traces exemplify the process of submitting transformer registrations to a *TR-LDPC* at `http://demo.fusepoolp3.eu/tr-ldpc`.


    POST /tr-ldpc
    Host: demo.fusepoolp3.eu
    Content-Type: text/turtle

    @prefix dct: <http://purl.org/dc/terms/>.
    @prefix trldpc: <http://vocab.fusepool.info/trldpc#> .

    <> a trldpc:TransformerRegistration;
        trldpc:transformer <http://example.org/simple-transformer>;
        dct:title "A simple RDF Transformation"@en;
        dct:description "transforms vcards to RDF".


Then, the response would look like:


    HTTP/1.1 201 Created
    Server: FusepoolP3/0.1.0 (build 2+)
    Last-Modified: Mon, 16 Jun 2014 08:54:14 GMT
    ETag: W/"1402908854000"
    Location: http://demo.fusepoolp3.eu/tr-ldpc/1234
    Content-Length: 0
    Date: Mon, 16 Jun 2014 08:54:14 GMT


An HTTP GET request to the container will return a container with at least the newly created *Transformer Registration* as member:


    GET /tr-ldpc
    Host: demo.fusepoolp3.eu
    Accept: text/turtle, application/rdf+xml;q=.8, \*/\*;q=.1


Will return something like:


    HTTP/1.1 200 OK
    Date: Mon, 16 Jun 2014 08:57:32 GMT
    Connection: close
    Content-Type: text/turtle
    Accept-Post: text/turtle, application/ld+json
    Allow: POST,GET,OPTIONS,HEAD,PUT
    ETag: W/"7868768696996"

    @prefix dcterms: <http://purl.org/dc/terms/>.
    @prefix ldp: <http://www.w3.org/ns/ldp#>.

    <>
       a ldp:BasicContainer;
       dcterms:title "A Transformer Registry Container";
       ldp:contains <1234>.
       
As the response does not contain the needed property of the *transformer registration* (even though it could) the client has to dereference the *transformer registration* to get URI of the *transformer*.

    GET /tr-ldpc/1234
    Host: demo.fusepoolp3.eu
    Accept: text/turtle, application/rdf+xml;q=.8, \*/\*;q=.1


Will return something like:

    @prefix rdfs:   <http://www.w3.org/2000/01/rdf-schema#> .
    @prefix trldpc: <http://vocab.fusepool.info/trldpc#> .
    @prefix dcterms: <http://purl.org/dc/terms/>.

    <> a trldpc:TransformerRegistration;
        trldpc:transformer <http://example.org/simple-transformer>;
        dct:title "A simple RDF Transformation"@en;
        dct:description "transforms vcards to RDF".
               
        
[LDP]: http://www.w3.org/TR/ldp/
[TRANSFORMER API]: https://github.com/fusepoolP3/overall-architecture/blob/master/transformer-api.md