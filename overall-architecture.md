# Overall Architecture

The P3 Platform allows storing and retrieving RDF and non-RDF data as well extracting RDF and transforming unstructured data into RDF. It supports the Linked Data Platform (LDP) Standard and defines an extensible mechanism so that transformation to RDF and automatic metadata generation can be perfomed automatically when content is added to the platform.

![Platform Diagramn](p3-platform-diagram.svg "Platform Diagram")

## Components as applications

By default each component including all extractors are indiviual processes intraction via HTTP. An exception to this are the storage related components LDP, RDF Triple Store, SPARQL and the possible custom backend services which may be more tightly coupled. For these components the reference implementation will be based on Marmotta however it will also be possible to use Virtuoso.


## LDP Proxy

A crucial role is held by the P3 LDP Proxy. The Proxy allows to use any compliant LDP implementations and adds the P3 specific features. The proxy delegates the actual storing and Retrieving of RDF data to the proxied LDP instance but perfomas some additional actions on specially marked LDP Collections. Refer to the [Data extractor and importer with extraction API](data-extractor-importer-api) for the interaction with the extractors as well as the definition of data importing LDP Container.

## Deployment Infrastructure

Basically a component can run on any platform that can expose HTTP services. The components of the reference implementation may require some common runtimes readlily available on windows as well as on the most popular UNIX derivates, e.g. Java or nodejs. Scripts are provided for startup on Unix Platforms.
