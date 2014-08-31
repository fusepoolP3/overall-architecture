# Transforming Container API

## Conventions

This section is normative.

In this document the following CURIE-Prefix shall be used for the following URIs:

 * eldp: http://vocab.fusepool.info/eldp#
 * trans: http://vocab.fusepool.info/transformer#

Sections are non-normative unless otherwise specified. Unless otherwise specified subsections are normative if the containing section is normative.

The key words MUST, MUST NOT, REQUIRED, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this specification are to be interpreted as described in [RFC2119].

## Introduction

This document specifies an extension to the [LDP] specification to allow for container that execute a transformation task when a member resource is added via a POST request.

## General

This section is normative.

Implementations of this specification MUST also conform to [LDP].

If the representation contains a triple with the LDPC as subject and `eldp:transformer` as predicate an implementation MUST invoke the `trans:Transformer` identified by the object of this triple whenever a non-RDF member resource is create following a POST request to the LDPC. The request against the `trans:Transformer` SHOULD be executed asynchronously: the POST request SHOULD return according to section 5.2.3.1 of [LDP] without waiting for the Transformer to complete. If the transformationis successful the results of the Transformation will SHOULD be added to the same collection. If the transformation result is an RDF graph a triple with predicate `eldp:transformedFrom` the transformation result resource as subject and the original non-RDF resource as object SHOULD be added to the transformation result LDPR.

### Example

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
    @prefix eldp: <http://vocab.fusepool.info/eldp#>.

    <http://example.org/container1/>
       a ldp:DirectContainer;
       dcterms:title "A transforming LDP Container using simple-transformer";
       ldp:membershipResource <http://example.org/container1/>;
       ldp:hasMemberRelation ldp:member;
       ldp:insertedContentRelation ldp:MemberSubject;
       fp:transformer <http://example.org/simple-transformer>.

The above container will process added resources using `http://example.org/simple-transformer` if their media-type matches one of the supported input formats of this transformer. If this is the case both the original resource as well as the Transformation results will be added to the container.

[RFC2119]: http://www.ietf.org/rfc/rfc2119
[LDP]: http://www.w3.org/TR/ldp/
