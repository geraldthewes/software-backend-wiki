# W3C SKOS Simple Knowledge Organization System Primer

**Title**: SKOS Simple Knowledge Organization System Primer  
**Authors**: Antoine Isaac (Vrije Universiteit Amsterdam), Ed Summers (Library of Congress)  
**Publisher**: W3C Semantic Web Deployment Working Group  
**Date**: 18 August 2009  
**Version**: W3C Working Group Note  
**URL**: http://www.w3.org/TR/skos-primer  

## Abstract

SKOS—Simple Knowledge Organization System—provides a model for expressing the basic structure and content of concept schemes such as thesauri, classification schemes, subject heading lists, taxonomies, folksonomies, and other similar types of controlled vocabulary. As an application of the Resource Description Framework (RDF), SKOS allows concepts to be composed and published on the World Wide Web, linked with data on the Web and integrated into other concept schemes.

This document is a user guide for those who would like to represent their concept scheme using SKOS.

In basic SKOS, conceptual resources (concepts) are identified with URIs, labeled with strings in one or more natural languages, documented with various types of note, semantically related to each other in informal hierarchies and association networks, and aggregated into concept schemes.

In advanced SKOS, conceptual resources can be mapped across concept schemes and grouped into labeled or ordered collections. Relationships can be specified between concept labels. Finally, the SKOS vocabulary itself can be extended to suit the needs of particular communities of practice or combined with other modeling vocabularies.

This document is a companion to the [SKOS Reference](https://www.w3.org/TR/skos-reference), which provides the normative reference on SKOS.

## Relevance to Software Engineering

SKOS is relevant to software engineering, particularly in the context of Domain-Driven Design (DDD) and semantic web technologies, as it provides a standardized way to:
1. Define mappings between disparate systems using properties like skos:broadMatch, skos:closeMatch, and skos:exactMatch
2. Organize vocabularies when different teams mean almost—but not exactly—the same thing
3. Bridge the gap between software engineering (object-oriented design) and semantic web technologies
4. Break down language silos across software development teams through corporate glossaries

SKOS supports the modern evolution of data mesh and knowledge graphs by providing interoperability standards that allow data from different domains to seamlessly link together.