# Install, build and run anHALytics backend

The anHALytics backend can be run with the anHALytics-core project. It covers the document harvesting, annotating and indexing, together with the creation of a Knowledge Base over the whole document collection. 

## Prerequisites

### Current development version

The current development version can be downloaded from GitHub and built as follow:

Clone source code from github:

	git clone https://github.com/anHALytics/anHALytics-core.git

### Java/Maven

A Java development environment is necessary: [JAVA JDK 1.7 minimum](https://java.com/en/download/manual_java7.jsp) and [Maven](https://maven.apache.org).

### GROBID (GeneRation Of BIbliographic Data)

[Grobid](https://github.com/kermitt2/grobid) is used as entry point for document digestion, which means extracting metadata and structured full text in ([TEI](http://www.tei-c.org/Guidelines/)). [Grobid](https://github.com/kermitt2/grobid) is a machine learning library for extracting bibliographical information and structured full texts from technical and scientific documents, in particular from PDF. It is distributed under Apache 2 license.

Clone the current project from github (as of November 2016, GROBID version 0.4.2-SNAPSHOT):

	git clone https://github.com/kermitt2/grobid.git

AnHALytics uses GROBID as a service which allows to distribute the process and exploit multithreading easily. See the [GROBID documentation](http://grobid.readthedocs.org) on how to start the RESTful service.

### (N)ERD - Entity Recognition and Disambiguisation

Our (N)ERD service annotates the text by recognizing and disambiguating terms in context. Entities are currently identified against Wikipedia. In this project, the (N)ERD is using the free disambiguation service - the recognition and disambiguation is not constrained by a preliminary Named Entity Recognition. 

At the present date, only the NER part of the NERD is available in open source (see [grobid-ner](https://github.com/kermitt2/grobid-ner). The full NERD repo will be made publicly available on GitHub soon under Apache 2 license. 

### Keyterm extraction and disambiguation 

This keyphrase, key concept and category extraction service is based on the [keyphrase extraction tool](http://www.aclweb.org/anthology/S10-1055) developed and ranked first at [SemEval-2010, task 5 - Automatic Keyphrase Extraction from Scientific Articles](http://www.aclweb.org/anthology/S10-1004). _Term_ in this context has to be understood as a complex technical term, e.g. a phrase having a specialized meaning given a technical or scientific field. In addition to key term extraction, the weighted vector of terms is disambiguated by the above (N)ERD service, resulting in a weighted list of Wikipedia concepts (i.e. Wikipedia articles) and a list of Wikipedia categories is provided.  

The Keyterm extraction repo will be made publicly available on GitHub soon under Apache 2 license. 

### ElasticSearch

[Elasticsearch](https://github.com/elastic/elasticsearch) is a distributed RESTful search engine. Specify (and adjust, if you want more shards and replicas) the following in the ElasticSearch config file (```elasticsearch.yml```):

    cluster.name: anhalytics_cluster2711
    index.number_of_shards: 1
    index.number_of_replicas: 0
    cluster.routing.allocation.disk.threshold_enabled: false
    http.jsonp.enable: true

    http.max_initial_line_length: 8kb

    http.cors.enabled : true
    http.cors.allow-origin : "*"
    http.cors.allow-methods : OPTIONS, HEAD, GET, POST, PUT, DELETE
    http.cors.allow-headers : X-Requested-With,X-Auth-Token,Content-Type, Content-Length

Note that CORS must be allowed.

AnHALytics supports currently (November 2016) a version __2.4__ of ElasticSearch, we plan to upgrade to a version 5.* in the next weeks. 

### MongoDB

A running instance of [MongoDB](https://www.mongodb.org) is required for document persistence and document provision.

### Mysql

A running instance of [Mysql](https://www.mysql.fr/) is required for building the knowledge base (relational database).

### Web server

A web application server, such as Tomcat, JBoss or Jetty, is necessary to deploy the complementary front demos which are packaged in a war file.

## Project design

anHALytics-core performs the document ingestion, from external harvesting of documents to indexing. It has (so far) six components corresponding to six sub-projects:

0. __common__ contains methods and resources shared by several other components. 
1. __harvest__ performs the document harvesting (PDF and metadata) and the transformations into common TEI representations. 
2. __annotate__ realises document enrichment, more precisely it disambiguates and annotates entities and key-concepts into the TEI structures.
3. __kb__ build and update the Knowledge Base (KB) of anHALytics.
4. __index__ performs indexing in ElasticSearch for the final TEI, the annotations and the KB.
5. __test__ is dedicated to integration tests.

### Build with maven

After cloning the repository, under the main directory `anHALytics-core/`make sur the paths are correct in the configuration properties files `conf/anhalytics.test.properties` then compile and build using maven :

    mvn clean install

After the compilation you'll find the jar produced for each module under ``MODULE/target``.

You need first to customize the configuration:

    ./anhalytics.sh --configure

This it will create a configuration file ```anHALytics-core/config/anhalytics.properties```.

Finally, to create the knowledge base database using MySQL :

    ./anhalytics.sh --prepare

Make sure all the required service are up.

## Components

### Harvesting

    > cd anhalytics-harvest

Currently only OAI-PMH is supported as harvesting protocol (for HAL archive)

An executable jar file is produced under the directory ``anhalytics-harvest/target``.

The following command displays the help:

    > java -jar target/anhalytics-harvest-<current version>.one-jar.jar -h

For a large harvesting task, use -Xmx2048m to set the JVM memory to avoid OutOfMemoryException.

#### HarvestAll / HarvestDaily

To start harvesting all the documents of HAL based on [OAI-PMH](http://www.openarchives.org/pmh) v2, use:

    > java -Xmx2048m -jar target/anhalytics-harvest-<current version>.one-jar.jar -exe harvestAll

Harvesting is done through a reverse chronological order, here is a sample of the OAI-PMH request:
http://api.archives-ouvertes.fr/oai/hal/?verb=ListRecords&metadataPrefix=xml-tei&from=2015-01-14&until=2015-01-14

To perform an harvesting on a daily basis, use:

    > java -Xmx2048m -jar target/anhalytics-harvest-<current version>.one-jar.jar -exe harvestDaily

For instance, the process can be configured on a cron table.

#### Grobid processing

Once the document are downloaded, the TEI extrating threads will run automatically. You can also run the process manually with

    > java -Xmx2048m -jar target/anhalytics-harvest-<current version>.one-jar.jar -exe processGrobid

#### TEI building

The working TEI is generated following this struture

```xml
    <teiCorpus>
        <teiHeader>
            <!-- Consolidated harvested metadata, from HAL for example, with entity 
				(author, affiliation, etc.) disambiguation -->
        </teiHeader>
        <TEI>
            <!-- GROBID automatically extracted data -->
        </TEI>
    </teiCorpus>
```

At least the harvested Metadata TEI is necessary to produce the final TEI(the PDF is not always available :( ), it's done using :

    > java -Xmx2048m -jar target/anhalytics-harvest-<current version>.one-jar.jar -exe generateTei

#### Document storage and provision

We use MongoDB GridFS layer for document file support. Each type of files are stored in a different collection. hal tei => hal-tei-collection , binaries => binaries-collection,..., 

<!-- documentation of the collections here !! -->

### Annotating

Once the working TEI collection is set, we can start to enrich the documents with our text mining components: extraction of named entities and computation of keyterms (aka free keyphrase extraction), key concepts (Wikipedia articles) and key categories (Wikipedia catgories) from the downloaded documents. This is the purpose of the anhalytics-annotate sub-project: 

    > cd anhalytics-annotate

The documents are enriched with semantic annotations. This is realized with the NERD service.

An executable jar file is produced under the directory ``anhalytics-annotate/target``.

The following command displays the help:

    > java -Xmx2048m -jar target/anhalytics-annotate-<current version>.one-jar.jar -h

For launching the full annotation of all the documents using all the available annotators: 

    > java -Xmx2048m -jar target/anhalytics-annotate-<current version>.one-jar.jar -multiThread -exe annotateAll

(```-multiThread``` option is recommended in general in order to activate parallel processing for document annotation, due to the size of the document repository)

#### Annotation of the HAL collection with the (N)ERD service

The annotation of the sub-project ``anhalytics-annotate/``:

    > java -Xmx2048m -jar target/anhalytics-annotate-<current version>.one-jar.jar -multiThread -exe annotateAllNerd

(```-multiThread``` option is recommended)

#### Annotation of the HAL collection with the KeyTerm extraction and disambiguation service

The annotation on the HAL collection can be launch with the command in the main directory of the sub-project ``anhalytics-annotate/``:

    > java -Xmx2048m -jar target/anhalytics-annotate-<current version>.one-jar.jar -multiThread -exe annotateAllKeyTerm

(```-multiThread``` option is recommended)

#### Storage of annotations

Annotations are persistently stored in a MongoDB collection and available for indexing by ElasticSearch. 

### Knowledge Base Builder

The knowledge base is built using MySQL. All the metadata at our disposal are extracted and saved in a relational database following this [data model](https://github.com/anHALytics/documentation/blob/master/model/anhalyticsDB.png).

    > cd anhalytics-kb

To see all available options:

    > java -Xmx2048m -jar target/anhalytics-kb-<current version>.one-jar.jar -h

To create the metadata database from the working TEI use:

    > java -Xmx2048m -jar target/anhalytics-kb-<current version>.one-jar.jar -exe initKnowledgeBase

To build a database for the bibliographic references :

    > java -Xmx2048m -jar target/anhalytics-kb-<current version>.one-jar.jar -exe initCitationKnowledgeBase

### Indexing

Move to the subproject: 

    > cd anhalytics-index

#### Set-up the indexes

The indexes will be initalized with the following command:

    > java -Xmx2048m -jar target/anhalytics-index-0.1-SNAPSHOT.one-jar.jar -exe setup

#### Build all the indexes 

For building all the indexes required by the different frontend applications, use the following command:

    > java -Xmx2048m -jar target/anhalytics-index-<current version>.one-jar.jar -exe indexAll

For indexing only the data corresponding on a daily basis:

    > java -Xmx2048m -jar target/anhalytics-index-<current version>.one-jar.jar -exe indexDaily

The following commands make possible to index separately only certain type of data. 

#### Indexing TEI

In the indexing process, the working TEI documents have to be indexed first: 

    > java -Xmx2048m -jar target/anhalytics-index-<current version>.one-jar.jar -exe indexFulltext

#### Indexing annotations

For indexing then the annotations, in the main directory of the sub-project ``anhalytics-index/`` use:

    > java -Xmx2048m -jar target/anhalytics-index-<current version>.one-jar.jar -exe indexAnnotations

#### Indexing the Knowledge Base

Third, for indexing the content of the Knowlkedge Base, in the main directory of the sub-project ``anhalytics-index/``:

    > java -Xmx2048m -jar target/anhalytics-index-<current version>.one-jar.jar -exe indexKB


### Test

This subproject is dedicated to integration and end-to-end tests - in contrast to unit tests which come with each specific sub-project.

    > cd anhalytics-test

Work in progress...


