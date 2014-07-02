NLP Interchange Format (NIF)
----------------------------

The NLP Interchange Format (NIF) is a RDF/OWL-based format that aims to improve interoperability between Natural Language Processing (NLP) tools as well as language resources. To facilitate this NIF defines an annotation model for NLP annotations including words, lemmas, stemms, part of speech tags, phrases, named entities and entity mentions. As such NIF can be compared to other annotation standards such as [Open Annotation](http://www.openannotation.org/spec/core/).

This section describes [NIF version 2.0](http://persistence.uni-leipzig.org/nlp2rdf/) with a focus on the defined Annotation Model leaving other important aspects - such as the interoperability - of the standard nearly untouched. 

NIF was specified with the following principles in mind. RDF was chosen as data formt as it provides both structural and conceptual interoperability. NIF aims for a big coverage of different NLP annotations going from coarse grained - document level - annotations down to annotations on word level. As fined grained - word level - annotations create a lot information scalability a simple structures with a low triple count was an importnat requirement. Provenance and Confidence information are also defined by NIF.


The remaining sections of this chapter first describe the URI Schemes used by NIF to assign annotated parts of the text unique URIs, second the annotation model as defined by the NIF core ontology and finally two sections describing Ontologies NIF is integrated with. First the Ontologies of Linguistic Annotation (OLiA) the ontology providing stable identifier for morpho-syntactical annotations and second the RDF version of the Internationalization Tag Set used by NIF to describe entity mentions.

### URI Schemes

The URI scheme of NIF combines two things. First and mot importnat it ensures to generate unique identifiers for annotated parts of the text. Second it also allows to encode the actual selection within the URI itself. That means that with NIF a single URI can be used to replace rich selectors as e.g. defined by [Open Annotation Selectors](http://www.openannotation.org/spec/core/specific.html#Selectors). In that aspect NIF is very much in line with the W3C [Media Fragment](http://www.w3.org/TR/media-frags/) specification.

While NIF allows to use different URI Schemes the preferred URI Scheme of NIF 2.0 is based on [RFC 5147](http://tools.ietf.org/html/rfc5147). It is based on start/end character indexes. To give an example lets assume a document with the URI `http://example.org/doc/demo1` containing a text with 2345 characters. Based on this URI Scheme the whole text is uniquely identified by `http://example.org/doc/demo1#char=0,2345`. Assuming the the word "Fusepool" is mentioned at position [1234..1245] it will use the identifier `http://example.org/doc/demo1#char=1234,1245`.

So the URI scheme ensure that we have three unique identifier (1) for the document (2) for the text contained in the document and (3) for the word "Fusepool" contained in the text. Those identifier now allow to make formal statements about the document, the text contained in the document and any arbitrary selection of characters in that text. In addition the URI scheme also - if desirable - to reduce the triple count of annotation as a lot of information are already encoded in the URI.

As mentioned above NIF allows to use different URI schemes. The most interesting alternative on is the Context-Hash-based URI Scheme as defined in section 2.1 (page 7-8) of [Towards an Ontology for Representing Strings](http://svn.aksw.org/papers/2012/WWW_NIF/public/string_ontology.pdf). This URI Scheme has two unique features: First it provides stable identifiers even if other parts of the content change and second it also works for rich text documents where character offsets can not be used. 

The used URI Scheme can be indicated by adding the according `nif:UriSchme` type to `nif:String` instances. As in most application the same URI Scheme will be used for all NIF annotations it should be sufficient to define the URIScheme for the `nif:Context` instance - this is the annotation selecting the content as a whole (see the next section for details).

### NIF Core Ontology

The NIF Core Ontology define an text annotation model focused the description of relations between substrings, text, documents and their URI schemes. the following figure shows the concepts and relations defined by the core ontology

![NIF Core Ontology](http://persistence.uni-leipzig.org/nlp2rdf/ontologies/nif-core/nif-core-ontology_web.png)

The main class in the ontology is `nif:String`, which is the class of all words over the alphabet of Unicode characters. `nif:String` defines data type properties for the selection (start/end index as well as before/anchorOf/after), properties for NLP related information (stemm, lemma, POS, sentiment) as well as relations to other `nif:String` instances. 

`nif:Structure` a subclass of `nif:String` is the parent of a collection of concepts defining the structure of texts including `nif:Title`, `nif:Paragraph`, `nif:Sentence`, `nif:Phrase` and `nif:Word`. A collection of properties between those classes allow to describe things like that a `nif:Word` is the first word of a `nif:Sentence` or the sequence of `nif:Words` within a text. 

The `nif:Context` type is assigned to instance selecting the whole text. This instance is used as the context for all `nif:String` instances created for the text. This means that all `nif:beginIndex` and `nif:endIndex` values are relative to the `nif:Context` instance referenced by the `nif:referenceContext` property. `nif:Context` defines to more importnat properties: First the `nif:sourceURL` property links to the unique identifier of the document and second the `nif:isString` property can be used to include the actual text as `rdf:Literal` in the RDF Graph.

`nif:URIScheme` is used to define the URI Scheme used to create unique identifier for `nif:String` instances. For more details about URI Schemes see the previous section.

When parsing NIF annotated documents a typical workflow looks like:

1. Query for a `nif:Context` instance that has a `nif:sourceURL` relation to the URI of the document in question. This `nif:Context` instance is in the following referenced by `{context}`
    * in cases where one wants to parse information from the used URI Scheme one needs to validate if the expected `nif:URIScheme` types is present for the `{context}`
    * the `nif:isString` property - if present - can be used to obtain the text. 
2. Query for all interesting `nif:String` instances that do have the `{context}` as `nif:referenceContext`
    * For parsing the text sentence by sentence one can first query for `nif:Sentence` instances and later get the words of the sentence by using the filter `null, {nif:sentence}, {sentence}`. While there are also properties like `nif:firstWord`, `nif:nextWord` ... in most cases it will be easier to parse the start/end offsets of the words and later order them base on those offsets.
    * For parsing headers and paragraphs queries for `nif:Title` and `nif:Paragraph` can be used.
3. Fia the properties defined by `NIF:String` one can now access all NLP annotations of selected sentences, phrases and words.
    * The POS tag is available by the `nif:oliaLink` property. Note also that the lexical category is directly referenced by the `nif:oliaCategory` property. See the following section for mor information about the OLiA Ontology.
    * The anchor text of the `nif:String` instance is available by the `nif:anchorOf` property. For full text searches the `nif:stem` value is often more interesting. For building Tag Clouds the `nif:lemma` (base form) is very useful.
    * The `nif:sentimentValue` provides the sentiment of the current word, phrase, sentence or if it is present on the `{context}` for the whole document.

### Ontologies of Linguistic Annotation (OLiA)

The [Ontologies of Linguistic Annotation](http://purl.org/olia) (OLiA) provide stable identiers for morpho-syntactical annotation tag sets. This allows NLP applications to be implemented based on stable identifiers instead of depending on the often different tag sets as used by different NLP processing tools. Additionally OLiA is multi lingual meaning the if an NLP application is interested in proper nouns it can just use `olia:ProperNoun` and will be able to process proper nouns of about 70 different languages.

Internally OLiA is build up by three different types of Models:

1. the __Reference Model__ defines the reference terminology intended to be used by NLP Applications. In other words the application programmer interface of OLiA.
2. a set of __Annotation Models__. Those formally describe morpho-syntactical annotation tag sets used by corpora and/or NLP frameworks. A good example is the [annotation model](http://purl.org/olia/penn.owl) for [Penn Treebank](http://www.cis.upenn.edu/~treebank/) tag set.
3. for every annotation model there is also a __Linking Model__ that establishes subClassOf relationships between the _Annotation Model_ and the _Reference Model_

Based on this design about 35 tag sets supporting about 70 languages are integrated with the _Reference Model_.

The __Reference Model__ contains several types of Linguistic Concepts. Most important is the `olia:MorphoSyntacticCategory` hierarchy. It defines an hierarchy over Part-of-Speech categories. On the first level OLiA defines categories like `olia:Noun`, `olia:Verb`, `olia:Adjective`, ... further down the hierarchy one can find categories like `olia:ProperNoun`, `olia:Gerund` or `PastParticipleAdjective`. But one can also find some other useful identifiers such as `olia:NamedEntity` defined as subclass of `DiscourseEntity` and as sibling of `olia:headline`; inflection types like `olia:BaseForm`, `olia:StrongInflection` ...; Features like Mood, Gender, Tense, ... ; Semantic Roles like `olia:AgentRole`, `olia:CauseRole`, `GoalRole`, ...; and many more.

The integration of OLiA to NIF is done by two properties both defined for the `nif:String` class.

1. the `nif:oliaLink` property intended to link to the instance of the _Annotation Model_. This is typically provided by processing results of an NLP tool. To map this to the according instance of the _Reference Model_ a reasoner is required. As this might not be the case for every application case NIF also defines
2. the `nif:oliaCategory` property that directly links to the _Reference Model_ instance. This property is redundant to the `nif:oliaLink` but extremely useful for simple queries of OLiA based NLP applications.

### Entity Linking Support

NIF uses the `itsrdf:taIdentRef` property to link an Entity with an `nif:String` instance. This property is defined by the [Internationalization Tag Set 2.0](http://www.w3.org/TR/its20/) W3C Recommendation that is integrated with NIF for providing support for RDF.

So an annotation linking the phrase `White House` with the entity `http://dbpedia.org/resource/White_House` could look like the following listing

    <http://example.org/doc/demo1#char=0,2345> a nif:Context, nif:RFC6147String
    
    <http://example.org/doc/demo1#char=127,138> a nif:Phrase ;
        nif:referenceContext <http://example.org/doc/demo1#char=0,2345> ;
        nif:beginIndex "127"^^xsd:integer ;
        nif:endIndex "138"^^xsd:integer ;
        nif:anchorOf "White House"@en ;
        nif:oliaCategory olia:NamedEntity, plia:NounPhrase ;
        nif:oliaConf "0.91"^^xsd:decimal ;
        itsrdf:taIdentRef <http://dbpedia.org/resource/White_House> ;
        


