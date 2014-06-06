OpenAnnotation
--------------

The Open Annotation specifies an RDF based model to (formally) describe - annotate - associations between related resources. An Annotation is considered to be a set of connected resources, typically including a body and target, where the body is somehow about the target. The following figure depicts this basic model.

![Basic Annotation](http://www.openannotation.org/spec/core/images/anno_model1.png)

Based on those three basic concepts Open Annotation defines extensions that do support rich semantic annotations, embedding content, selecting segments of resources, choosing the appropriate representation of a resource, multiplicity constructs and providing styling hints for consuming clients. The following subsections do provide more detailed information on those extensions relevant to Fusepool.

[Open Annotation Specification](http://www.openannotation.org/spec/core/) is an W3C working draft developed by the [Open Annotation Community Group](http://www.w3.org/community/openannotation/).

### Specifiers and Specific Resources

Specifiers are used by OpenAnnotation to further specify the target of an annotation. Specifiers include _Content Selectors_ - constructs that describe the annotated part of an document, _Content State_ - constructs that describe the state of the annotated document (e.g. the version or the content type) and the _Resource Scope_ - constructs that describe the context of the annotation.

#### Specific Resource

The Specific Resource provides identity to the state described by Specifiers. For that the Specific Resource takes the role of the target of an annotation and mediates to the full resource. The following figure shows this design. 

![Specific Annotated Resource](http://www.openannotation.org/spec/core/images/specificresource.png)

The _Specific Resource_ is the `oa:hasTarget` of the annotation. The full resource is linked by the _Specific Resource_ by the `oa:hasSource` property. Specifiers - as described in the following sections - are attached to the _Specific Resource_. 

#### Content Selectors

Content Selectors allow to formally specify the part of a document targeted by an annotation. Selectors are an alternative to the use of fragment URIs that aim to provide more freedom and expressiveness in the definition of selections. As `oa:Selector` are _Specifiers_ they are attached to _Specific Resources_ as shown by the following figure. 

![Open Annotation Selectors](http://www.openannotation.org/spec/core/images/selector.png)

Open Annotation defines the following selector types for textual information.

* [Text Position Selector](http://www.openannotation.org/spec/core/specific.html#TextPositionSelector) are specified based on start/end character indexes
* [Text Quote Selector](http://www.openannotation.org/spec/core/specific.html#TextQuoteSelector) are defined by using a prefix, suffix and a copy of the selected text
* [Data Position Selector](http://www.openannotation.org/spec/core/specific.html#DataPositionSelector) are based on the start/end byte offsets.

Open Annotation also defines a [SVG Selector](http://www.openannotation.org/spec/core/specific.html#SvgSelector) that can be used to select `path`, `rect`, `circle`, `ellipse`, `polyline`, `polygon` or `g` shape elements for selecting parts of images.

#### Content State

States are used to describe the state of the annotated resource. The Open Annotation specification include two content state classes: 

* [Time State](http://www.openannotation.org/spec/core/specific.html#TimeState) specifying the time stamp of the resource used for annotation.
* [Request Header State](http://www.openannotation.org/spec/core/specific.html#HttpRequestState) specifying HTTP headers required to retrieve the annotated resource. Typically things like the `Accept` or the `Accept-Language` header.

#### Resource Scope

Resource Scopes can be used to specify that an Annotation of an resource was created in the scope of an other one (e.g. If an annotation of an image was done based on surrounding text in an webpage the webpage can be defined as an resource scope). Scopes are attached to the _Specific Resources_ node.

#### Selectors vs. Media Fragment URIs

As noted in the introduction Open Annotation does allow to use fragment URIs as alternative to selectors. Therefore the use of [Media Fragment URI](http://www.w3.org/TR/media-frags/) is possible.

However the use a Media Fragment URIs as target of an annotation prevents the use of a _Specific Resource_ and therefore prevents the use of _Content Selectors_, _Content States_ and _Resource Scopes_. Because of that this model is only suitable for simple use cases. 

For details and additional considerations please not this [Blog Post](http://www.w3.org/community/openannotation/fragment-uris/) by Robert Sanderson a Chair of the Open Annotation Community Group.


### Annotation Provenance


![Open Annotation Provenance Information](http://www.openannotation.org/spec/core/images/anno_prov.png)

### Multiplicity Constructs

Multiplicity Constructs constructs allow to have multiple annotation bodies and/or targets. They can e.g. be used to provide users with different [choices](http://www.openannotation.org/spec/core/multiplicity.html#Choice) to disambiguate from.

![Open Annotation Choice](http://www.openannotation.org/spec/core/images/choice.png)

This option is especially interesting in use cases where one wants to allow users to select between different options as suggested by a SoftwareAgent.

Open Annotation also defines [Composite](http://www.openannotation.org/spec/core/multiplicity.html#Composite) - all included bodies or targets are required for the annotation to be valid - and [List](http://www.openannotation.org/spec/core/multiplicity.html#List) - a special composite where the order of the included items is of importance.

