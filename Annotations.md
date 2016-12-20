# Description of the annotations/extractions in anHALytics

The anHALytics backend produces currently different types of annotations. We describe here these annotations, the terminology used to refer to these annotations and how they differ from the available metadada. 

## Semantic descriptions

The following types of descriptions must be carefully distinguished:

### Keywords

The **keywords** are phrases usually selected by the author and present on the paper usually in the header/front section. In anHALytics the keywords come from two sources:

- HAL keywords: present in the HAL TEI metadata record, normally highly reliable because selected via a manual process. 

- GROBID keywords: those are the keywords that grobid service extracts automatically from the PDF in the header section (together with the title, authors, abstract, etc.). They are not fully reliable because of structure extraction errors. 

**Keywords** present in anHALytics are multilingual by nature. 

They are encoded in the TEI document model as metadata in the TEI header. Path in TEI document model: ```teiCorpus/teiHeader/profileDesc/textClass/keywords[@type='author']/term```

JSON path in ElasticSearch: ```$teiCorpus.$teiHeader.$profileDesc.$textClass.$keywords.$type_author.$term.notAnalyzed```

### Keyterms

The **keyterms** are the most discriminant/important phrases extracted from the textual content of a paper by grobid-keyterm in order to characterize the publication given a corpus of publication in background. The phrase are minimally normalized and matches the mentions in the text. Each keyterm is associated to a relevance score, corresponding to the probability that a reader of a paper would select the keyterm as key phrase.   

Currently **Keyterms** are only supported for the English language in anHALytics. See [inria-00493437](https://hal.inria.fr/inria-00493437) for the grobid-keyterm extraction algorithm. 

They are encoded in the TEI document model as standoff annotations.  Path in TEI document model: ```teiCorpus/standoff/keyterm/keyterm```

JSON path in ElasticSearch: ```$teiCorpus.$standoff.$keyterm.keyterm```

### Key categories

The key categories are the categories from the set of Wikipedia categories that are the most discriminant/important concepts to charcaterize the publication given a corpus of publication in background. Categories are general purposes semantic classes usually organised in a taxonomy. 

**Key categories** are currently only supported for the English language in anHALytics. These categories are the distributions of the categories associated to the concepts obtained by disambiguating the weighted vector of terms extracted by grobid-keyterm. The disambiguation is performed by the NERD component. 

They are encoded in the TEI document model as standoff annotations.  Path in TEI document model: ```teiCorpus/standoff/category/category```
Wikipedia category identifier under: ```teiCorpus/standoff/category/wikipediaExternalRef```

JSON path in ElasticSearch: ```$teiCorpus.$standoff.$category.category```
```$teiCorpus.$standoff.$category.wikipediaExternalRef```

### Concepts

**Concept** annotations are produced by the NERD component. NERD annotates mentions in the text with the concept that are realized. Concepts are expressed in the Wikipedia semantic space (where a concept is a Wikipedia article). For each represented language, a concept is associated to a preferred term, a set of synonyms, one or several definitions, a set of attributes, a set of relations. 

All concepts realized in a text are identified, there is no "ranking" of the concepts for these annotations. 

Concepts, here, are abstract representations used to describe and capture the reality of the world we observe and understand. 

As we cannot represent a concept for human manipulation (a concept is represented by default by the identifier of the corresponding Wikipedia page), we use the preferred term of the concept. 

**Concepts** annotations currently cover three languages in anHALytics: English, French and German. Concepts are produced by the NERD component using a classifier exploiting various statistical features, in particular an adaptation of the relatedness score from [(Milne & Witten, 2008)](http://researchcommons.waikato.ac.nz/handle/10289/1777). 

They are encoded in the TEI document model as standoff annotations.  Path in TEI document model: ```teiCorpus/standoff/nerd/preferredTerm```
Wikipedia identifier corresponding to the concept is given at ```teiCorpus/standoff/nerd/wikipediaExternalRef```

JSON path/query field in ElasticSearch: ```$teiCorpus.$standoff.$nerd.preferredTerm```
```$teiCorpus.$standoff.$nerd.wikipediaExternalRef```

### Key concepts

The **key concepts** are the concepts expressed the Wikipedia space (where a concept is a Wikipedia article) that are the most discriminant/important concepts to characterize the publication given a corpus of publication in background. 

As we cannot represent a concept for human manipulation (a concept is represented by default by the identifier of the corresponding Wikipedia page), we use the preferred term of the concept. 

**Key concepts** currently cover three languages in anHALytics: English, French and German. These concepts are obtained by disambiguating the weighted vector of terms extracted by grobid-keyterm. The disambiguation is performed by the NERD component. 

They are encoded in the TEI document model as standoff annotations.  Path in TEI document model of the preferred term of a concept: ```teiCorpus/standoff/keyterm/preferredTerm```. Wikipedia identifier corresponding to the concept is given at ```teiCorpus/standoff/keyterm/wikipediaExternalRef```

JSON path/query field in ElasticSearch: ```$teiCorpus.$standoff.$keyterm.preferredTerm```
```$teiCorpus.$standoff.$nerd.preferredTerm```


## Purposes

Keywords, keyterms, key-categories and key-concepts are extractions that aim at capturing the most important aspects characterizing a document against the whole set of background documents. They are thus appropriate for facettings in search, organising a set of documents, classification, etc. 

Concept annotations will identify all the realizations of a concept in a text, they give an additional layer of abstract description on top of the words. They are appropriate to semantic search, where the search query can use concepts, or for improving the ranking given a query expressed in words and disambiguated. 