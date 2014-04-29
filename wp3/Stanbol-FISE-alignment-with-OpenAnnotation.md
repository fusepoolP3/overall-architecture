# Fusepool P3: Stanbol's FISE alignment with Open Annotation

* **Authors**: Rupert Westenthaler, Sergio Fernández
* **Last modification date**: April 29, 2014
* **Status**: This documents is just an internal **draft** from the [Fusepool P3](http://www.fusepool.eu/p3) consortium

# Motivation

The project would require to integrate different system producing (automatic) annotations.
So it is required to find a common model that would allow us to easily do it. And, since Stanbol 
will be part of our technology strack, it is a could be good to approach this anlysis from
the experience there.

# Introduction

[Stanbol](http://stanbol.apache.org) currently uses 
[FISE](http://stanbol.apache.org/docs/trunk/components/enhancer/enhancementstructure),
a custom ontology defining all required concepts and properties for producing annotations
from content. Since Stanbol is currently [working in the 1.0 release](http://markmail.org/message/6dwwkwv7elmo6454),
now it is the right time to address some major changes that will break backward compatibility. 
An the annotation model is one of the obvious candidates. The project would be happy to get
contributions from Fusepool on such aspect.

This document is part of the outcomes of DSS23 from the [backlog](https://easybacklog.com/accounts/4748/backlogs/54217).

# Analysis

There are several options that we should take a look to:

## Open Annotation

[Open Annotation](http://www.openannotation.org/spec/core/) specifies an interoperable 
framework for creating associations between related resources, annotations, using a 
methodology that conforms to the Architecture of the World Wide Web. 

The Open Annotation data model provides an extensible, interoperable framework for expressing annotations such that they can easily be shared between platforms, with sufficient richness of expression to satisfy complex requirements while remaining simple enough to also allow for the most common use cases, such as attaching a piece of text to a single web resource.

An `oa:Annotation` is considered to be a set of connected resources, typically including a `oa:hasBody` and `oa:hasTarget`, and conveys that the body is related to the target. From there, one can enhance different structures of the annotation by adding specific classes and relationships (see the [core Documentation](http://www.openannotation.org/spec/core/core.html) for details and examples). Annotation [Provenance](http://www.openannotation.org/spec/core/core.html#Provenance) is modeled by information added to the oa:Annotation resource. This also includes information about the person or software that created the annotation.

This core model is extended by three optional modules: The Specifiers and Specific Resources module allows the annotation target to select a specific part of the annotated media resource (see the [module documentation](http://www.openannotation.org/spec/core/specific.html)) for details and examples). Specific Resource allows to identify the resource the annotations are about. This is important for resources that do change are may have different states. This module also defines a range of Selectors used to selects parts of the annotated resource. Open Annotation allows to use W3C Media Fragment URIs as selectors but also defines its own selectors for text, data and SVG. The [Multiplicity Constructs](http://www.openannotation.org/spec/core/multiplicity.html) that allows to describe choices, composites. Finally there is also a [Publishing](http://www.openannotation.org/spec/core/publishing.html) module.

Unlike previous attempts at annotation interoperability, the Open Annotation system does not prescribe a transport protocol for creating, managing and retrieving annotations. Instead it describes a web-centric method, promoting discovery and sharing of annotations without clients or servers having to agree on a particular set of network transactions to communicate those annotations.

## NIF

[NIF](http://nlp2rdf.org/nif-1-0) (Nlp Interchange Format, [paper](http://svn.aksw.org/papers/2013/ISWC_NIF/public.pdf)) is an RDF based format that aims to achieve interoperability between Natural Language Processing (NLP) tools, language resources and annotations. To formally represent NLP annotations NIF first defines an URI scheme based on [RFC 5147](http://tools.ietf.org/html/rfc5147) that allows to reference Strings (words, phrases, sentences ...) as RDF resources and second it defines/uses a set OWL ontology that allows to formally describe such Strings.
The String Ontology is the core vocabulary to describe a String as part of a document and possible a sub-String of an other one. Structured Sentence Ontology (SSO) allows to define Strings as Sentences, Phrases or Words and its relationships.

## OLiA

The Ontologies of Linguistic Annotation ([OLiA](http://purl.org/olia) - [paper](http://www.semantic-web-journal.net/system/files/swj518.pdf)) defines a Reference Model with classes for linguistic categories (e.g. Noun, Determiner ...). Multiple Annotation Models formalize annotation schemes and tag sets used by different corpora and languages. Finally for every Annotation Model there is also a Linking Model that aligns it with the OLiA Reference Model. This allows to use the OLiA Reference Model to access/query NLP annotations using different annotation models (e.g. NLP annotations generated by different NLP tools/framework). It also supports queries for Words with a specific lexical category (e.g. ProperNouns) in texts with different languages. NIF uses OLiA for representing the POS annotations of words.

## NERD

[NERD](http://nerd.eurecom.fr/ontology/) (Named Entity Recognition and Disambiguation - papers: [1](http://nerd.eurecom.fr/ui/paper/Rizzo_Troncy-eacl2012demo.pdf), [2](http://events.linkeddata.org/ldow2012/papers/ldow2012-paper-02.pdf)) defines both a data model for describing detected Named Entities as well as an ontology for the types of Named Entities. This hierarchy of Named Entity types is also aligned with several other ontologies and common datasets. NIF uses NER to link Strings with entities as well as with the NER type.

# Working Plan

1. Looking at the available standards
2. Definition of the aligned annotation model
3. Implementation in in the Enhancer core modules
4. Adaptations of all Enhancement Engines
5. Adaptations of client libraries

# Related Information

* There was already a [discussion](http://markmail.org/message/mwnsz7lznq3g77el) about a similar toic on the Stanbol Mailing list.
* [STANBOL-351](https://issues.apache.org/jira/browse/STANBOL-351) is about the definition of a Stanbol Enhancement Structure. This work should contribute to this issue.

# Open issues

* 

# Conclusions

@@TODO

