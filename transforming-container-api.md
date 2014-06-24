# Transforming Container API

For importing data to be transformed on import the resources to be transformed are added to a specially marked LDPC. When requesting the LDPC a triple with the LDPC as subject `fp:transformer`as property and an instance of `fp:ExctractionService` as object is returned in the returned graph. When a non-RDF resource is posted against that container the Transformation service will asynchronously be invoked. The results of the Transformation will be added to the same collection. A triple with predicate `fp:extractedFrom`point from the extracted resource to the non-RDF resource.

### Example 1

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
       dcterms:title "An extracting LDP Container using simple-transformer";
       ldp:membershipResource <http://example.org/container1/>;
       ldp:hasMemberRelation ldp:member;
       ldp:insertedContentRelation ldp:MemberSubject;
       fp:transformer <http://example.org/simple-transformer>.

The above container will process added resources using `http://example.org/simple-transformer` if their media-type matches one of the supported input formats of this transformer. If this is the case both the original resource as well as the Transformation results will be added to the container.
