# Description of the annotations/extractions in anHALytics

The anHALytics backend produces currently different types of annotations. We describe here these annotations, the terminology used to refer to these annotations and how they differ from the available metadada. 

## Semantic descriptions

The following types of descriptions must be carefully distinguished:

### Keywords

The **keywords** are phrases usually selected by the author and present on the paper usually in the header/front section. In anHALytics the keywords come from two sources:

- HAL keywords: present in the HAL TEI metadata record, normally highly reliable because selected via a manual process. 

- GROBID keywords: those are the keywords that grobid service extracts automatically from the PDF in the header section (together with the title, authors, abstract, etc.). They are not fully reliable because of structure extraction errors. 

**Keywords** present in anHALytics are multilingual by nature. 

### Keyterms

The **keyterms** are the most discriminant/important phrases extracted from the textual content of a paper by grobid-keyterm in order to characterize the publication given a corpus of publication in background. The phrase are minimally normalized and matches the mentions in the text. Each keyterm is associated to a relevance score, corresponding to the probability that a reader of a paper would select the keyterm as key phrase.   

Currently **Keyterms** are only supported for the English language in anHALytics.


### Key categories

The key categories are the categories from the set of Wikipedia categories that are the most discriminant/important concepts to charcaterize the publication given a corpus of publication in background. Categories are general purposes semantic classes usually organised in a taxonomy. 

**Key categories** are currently only supported for the English language in anHALytics.

### Concepts

**Concept** annotations are produced by the NERD component. NERD annotates mentions in the text with the concept that are realized. Concepts are expressed in the Wikipedia semantic space (where a concept is a Wikipedia article). All concepts realized in a text are identified, there is no "ranking" of the concepts for these annotations. 

Concepts, here, are abstract representations used to describe and capture the reality of the world we observe and understand. 

**Concepts** annotations currently cover three languages in anHALytics: English, French and German. 

### Key concepts

The **key concepts** are the concepts expressed the Wikipedia space (where a concept is a Wikipedia article) that are the most discriminant/important concepts to characterize the publication given a corpus of publication in background. For each represented language, a concept is associated to a preferred term, a set of synonyms, one or several definitions, a set of attributes, a set of relations. 

**Key concepts** currently cover three languages in anHALytics: English, French and German. 

## Purposes

Keywords, keyterms, key-categories and key-concepts are extractions that aim at capturing the most important aspects characterizing a document against the whole set of background documents. They are thus appropriate for facettings in search, organising a set of documents, classification, etc. 

Concept annotations will identify all the realizations of a concept in a text, they give an additional layer of abstract description on top of the words. They are appropriate to semantic search, where the search query can use concepts, or for improving the ranking given a query expressed in words and disambiguated. 