# User Interaction Request API

This document defines an API for requesting an interaction with a user. It is typically used by components such as transformers that need some kind of interaction with a privileged user, for example to review some proposed annotations before the (asynchronous) transformer returns the results.

## Conventions

This section is normative.

In this document the following CURIE-Prefix shall be used for the following URIs:

 * rdf: http://www.w3.org/1999/02/22-rdf-syntax-ns#
 * fp3: http://vocab.fusepool.info/fp3#

Sections are non-normative unless otherwise specified. Unless otherwise specified subsections are normative if the containing section is normative.

The key words MUST, MUST NOT, REQUIRED, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL in this specification are to be interpreted as described in [RFC2119].

## Introduction

To request an interaction with a user a component adds a dereferenceable URI to a registry. Client tools will then present a link to the resource identified by this URI. If the user follows that link it is up to the component requesting the interaction to present the user appropriate resource representation so that they can perform the required interaction. Typical such a representation could be an HTML form. Once the interaction has been completed the component removes the URI from the registry.

## Terms

This section is normative.

 * **Interaction Resource**: A resource identified by an URI that dereferences to a representation allowing a user to perform the necessary interaction. This might be just an entry point for an actual interaction expanding over several resources identified by different URIs.
 * **Interaction Request**: A resource of type `fp3:InteractionRequest` representing an interaction request, its most important property `fp3:interactionResource` pointing to the *Interaction Resource*.
 * **Interaction Request Container (IR-LDPC)**: an LDP Basic Container (see [LDP]) to which *Interaction Request*s can be added.
 
## Interaction

This section is normative.

This specification does not define how clients discover an *IR-LDPC*.

An *IR-LDPC* is an LDP Basic container that MUST conform to the requirements of [LDP] along with the following restrictions. 

 * The server MUST support HTTP POST requests against the *IR-LDPC* to create LDPR
 * The server MUST support DELETE requests against the resources created by POST requests against the *IR-LDPC"

An agent requesting user interaction MUST send an HTTP POST request against an *IR-LDPC* with a request entity encoding a graph with at least one triple with `fp3:interactionResource` as predicate and the entity identified by a null relative URI as subject and the *interaction resource* as object, the graph MUST NOT contain any other triple with this combination of subject and predicate.

Once a user successfully performed the interaction by accessing the *interaction resource* the client that posted the interaction request MUST remove the *interaction request* by sending an HTTP DELETE request against the URI of the *interaction request*.

The client requesting user interaction MUST make ensure multiple concurrent requests against the *interaction request* do not cause server side errors or an inconsistent state.

Non-normative note: A client presenting *interaction request*s to users MAY hide an requests from other users after a user followed the link to the *interaction resource* for some time to reduce chances that multiple users attempt to perform the same interaction.

## Example

The following HTTP traces exemplify the process of submitting interaction requests to am *IR-LDPC* at `http://demo.fusepoolp3.eu/ir-ldpc`.


    POST /ir-ldpc
    Host: demo.fusepoolp3.eu
    Content-Type: text/turtle

    @prefix rdfs:   <http://www.w3.org/2000/01/rdf-schema#> .
    @prefix fp3: <http://vocab.fusepool.info/fp3#> .

    <> a fp3:InteractionRequest;
        fp3:interactionResource <http://example.org/annotator/disambiguation-request45>;
        rdfs:comment "Disambiguation-Request"@en.


Then, the response would look like:


    HTTP/1.1 201 Created
    Server: FusepoolP3/0.1.0 (build 2+)
    Last-Modified: Mon, 16 Jun 2014 08:54:14 GMT
    ETag: W/"1402908854000"
    Location: http://demo.fusepoolp3.eu/ir-ldpc/1234
    Content-Length: 0
    Date: Mon, 16 Jun 2014 08:54:14 GMT


An HTTP GET request to the container will return a container with at least the newly created *Interaction Request* as member:


    GET /ir-ldpc
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
       dcterms:title "An Interaction Request Container";
       ldp:contains <1234>.
       
As the response does not contain the needed property of the *interaction request* (even though it could) the client has to dereference the *interaction request* to get URI of the *interaction resource*.

    GET /ir-ldpc/1234
    Host: demo.fusepoolp3.eu
    Accept: text/turtle, application/rdf+xml;q=.8, \*/\*;q=.1


Will return something like:

    @prefix rdfs:   <http://www.w3.org/2000/01/rdf-schema#> .
    @prefix fp3: <http://vocab.fusepool.info/fp3#> .
    @prefix dcterms: <http://purl.org/dc/terms/>.

    <> a fp3:InteractionRequest;
        dcterms:created "2014-06-14T08:54:14Z"^^xsd:dateTime
        fp3:interactionResource <http://example.org/annotator/disambiguation-request45\>;
        rdfs:comment "Disambiguation-Request"@en.
               
        
[LDP]: http://www.w3.org/TR/ldp/