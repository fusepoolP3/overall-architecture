# Overall Architecture

The P3 Platform allows storing and retrieving RDF and non-RDF data as well extracting RDF and transforming unstructured data into RDF. It supports the Linked Data Platform (LDP) Standard and defines an extensible mechanism so that transformation to RDF and automatic metadata generation can be perfomed automatically when content is added to the platform.

## LDP Proxy

A crucial role is held by the P3 LDP Proxy. The Proxy allows to use any compliant LDP implementations and adds the P3 specifci features. The proxydelegates the actual storing and Retrieving of RDF data to the proxied LDP instance but perfomas some additional actions on specially marked LDP Collections.
