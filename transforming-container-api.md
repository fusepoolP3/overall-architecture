# Transforming Container API

## Conventions

In this document the following CURIE-Prefix shall be used for the following URIs:

 * eldp: http://vocab.fusepool.info/ldp#
 * trans: http://vocab.fusepool.info/transformer#

## Introduction

This document specifies an extension to the [LDP] specification to allow for container that execute a transformation task when a member resource is added via a POST request.

## General

This section is normative.

Implementations of this specification MUST also conform to [LDP].

If the representation contains a triple with the LDPC as subject and `eldp:transformer` as triple a complaint will invoke the `trans:Transformer` identified by the object of this triple whenever a non-RDF member resource is create following a POST request to the LDPC. The request against the `trans:Transformer` is executed asynchronously, the POST request will return according to section 5.2.3.1 of [LDP] without waiting for the Transformer to complete. If the transformationis successful The results of the Transformation will be added to the same collection. If the transformation result is an RDFS a triple with predicate `eldp:extractedFrom` the transformation result resource as subject and the original non-RDF resource as object is added to the transformation result LDPR.

### Example 1

This section in non-normative.

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
    @prefix eldp: <http://vocab.fusepool.info/ldp#>.

    <http://example.org/container1/>
       a ldp:DirectContainer;
       dcterms:title "An extracting LDP Container using simple-transformer";
       ldp:membershipResource <http://example.org/container1/>;
       ldp:hasMemberRelation ldp:member;
       ldp:insertedContentRelation ldp:MemberSubject;
       fp:transformer <http://example.org/simple-transformer>.

The above container will process added resources using `http://example.org/simple-transformer` if their media-type matches one of the supported input formats of this transformer. If this is the case both the original resource as well as the Transformation results will be added to the container.

[LDP]: http://www.w3.org/TR/ldp/
