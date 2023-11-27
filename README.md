# Document Retrieval Augmented Generation (RAG) Harness

The area of Retrieval Augmented Generation is rapidly evolving. There are many different ways to implement retrieval. 
Some people use embeddings and vector databases, some other use semantic graphs.
So, there are different designs and also there are different tasks and it is important to match a design to a task[^1].

[^1]: [Design Space for Graph Neural Networks](https://arxiv.org/pdf/2011.08843.pdf), [Lecture](https://www.youtube.com/watch?v=8OhnwzT9ypg&list=PLoROMvodv4rPLKxIpqhjhPgdQy7imNkDn&index=60) part of [Stanford CS224W: ML with Graphs](https://www.youtube.com/watch?v=JAB_plj2rbA&list=PLoROMvodv4rPLKxIpqhjhPgdQy7imNkDn), [Slides](http://snap.stanford.edu/class/cs224w-2021/slides/20-conclusion.pdf)

 
The goal of this harness to provide a collection definitions, abstractions, and building blocks to aid in understanding, benchmarking, comparing, and selecting a specific retrieval design which best matches a task at hand. 

The harness is intended to be somewhat similar to a Technology + [Technology Compatibility Kit (TCK)](https://en.wikipedia.org/wiki/Technology_Compatibility_Kit) - to provide:

* Java/EMF Ecore model/API for document storage and retrieval including "Design Provider Interface" to be implemented by candidate designs
* Testing framework for evaluate how different designs perform a specfic task.    

Java was selected as a dominant technology in the enterprise world, and EMF Ecore was selected because there are capabilities to load/store models from XMI and binary and generate HTML documentation from models and metamodels.

This page provides an introduction to core concepts and outlines several use cases (tasks) and designs (alternatives). 

## Concepts

The below diagram outlines the harness structure and context:

![overview](overview.png)

The following sections provide definitions and outline task/design dimensions for each definition. 
The [metamodel](https://doc-rag-harness.ai-deep-learning.us/) captures some of the definitions as model elements and elaborates them into features, operations, and subclasses.

### Document

Document is memorialized representation of thought or information. For the purposes of this harness documents:

* Are stored in a document source/repository (like books in a library)
* Can be categorized and categories can be nested (e.g. book genre - fiction / sci-fi)
* Can have internal structure (e.g. volume, chapter, section, paragraph, word)
* Can contain different types of contents - text, image, video, audio, structures like lists and tables
* Can reference other documents or external entities

"Physical" implementations:

* PDF. In Java can be loaded using [Apache PDFBox](https://pdfbox.apache.org/)
* OCR result in, say, JSON
* MS Office documents - in Java can be loaded with [Apache POI](https://poi.apache.org/). MS Excel files can be loaded as Ecore model with [Nasdanika Excel Model](https://github.com/Nasdanika-Models/excel) 
* HTML documents/pages

"Logical" implementations:

* For PDF/OCR - a scan of a business document. For example, a fax of a SWIFT message. In this case:
    * Header and footer added by the fax might be removed as irrelevant
    * Page bodies might be parsed into a SWIFT specific structure, e.g. [MT 700](https://www2.swift.com/knowledgecentre/publications/us7m_20210723/1.0?topic=mt700-format-spec.htm)
* For HTML - a documentation page. Say, [Spring Expression Language (SpEL)](https://docs.spring.io/spring-framework/reference/core/expressions.html) In this case header, left navigation, right sidebar, and footer might be discarded as not relevant or parsed into respective logical document features which might be ignored. Breadcrumb can be used for categorization.      

### Document Loader

Converts one document representation to another. E.g. PDF or OCR JSON to an object model of a SWIFT MT 700 message. 

### Documents Source 

Storage of documents in a specific format or formats. E.g. a file system with PDF documents. Documents sources may be converted/adapted. 

### Document Repository

A collection of documents providing storage and retrieval functionality. 
The primary interface of the DPI (see below) to be implemented by designs.

When storing a document the repository may perform tasks such as image recognition.  

There might be multiple retrieval modalities such as:

* Keyword search
* Semantic search
* Summarization - search and summarize top X results

Repositories can be assembled from other repositories and data loaders. E.g. a PDF repository may be assembled from a PDF -> Object model data loader and an object model repository.
Also document repositories may not have to store/recreate the source document - they may reference it and retrieve from a document store - the original from which the document was loaded, or a repository-specific document store.

It might also be possible to compose different designs of repositories. For example, a repository which supports keyword search and a repository which supports semantic search. In this case the keyword search repository query results would be necessary, but not sufficient and might be use to validate results of the semantic search repository.  

### User / Web UI

Users query a document repository via the Web UI. They can do it as part of their job function or to evaluate query functionality of a specific design and provide feedback. 
These two modalities may be combined - users may choose to use only the "champion" query engine/design, e.g. keyword search, or also select "challenger" engines/designs.

The Web UI might capture user context such as role/position in the organization and pass it to the design as part of a query.  

### Sponsor

A party interested in improving qualities of user work such as productivity by utilizing document retrieval augmented generation. 

Sponsors need to balance multiple criteria to minimize the "loss function":

* Retrieval speed
* Accuracy   
* Costs such as running costs, license costs etc.

### Design

Design is an instantiation/embodiment of technologies and their configuration parameters.

#### Design dimensions

Design variation points - what can be changed in different embodiments/instantiations and source of values. 
For example:

* Number of embedding dimensions
* ML model
* Model temperature
* Vector database 
* Vector database version

Design dimensions can form a tree or, more precisely, a directed graph. E.g. vector database versions would be nodes under a node for a specific vector database. 

### Design Provider Interface

Design Provider Interface (DPI) abstracts the harness from a particular design implementation. It is a set of interfaces and abstract classes which design has to implement.
E.g. ``DocumentRepository`` interface. DPI is defined in Java/Ecore and may provide adapters to different technologies. In particular:

* REST API 
* Language bindings and a runner which implements the REST API and calls components which implement the language binding interface. For example a Python binding can be implemented with [Flask](https://palletsprojects.com/p/flask/)
* Framework bindings/implementations under language bindings or directly under the DPI in Java. E.g. under the Python binding there might be a [LangChain](https://www.langchain.com/) binding and under Java there might be [OpenNLP](https://opennlp.apache.org/) binding

### Task

Task is a specific use of document retrieval. For example, semantic search in organizaiton-specific technical documentation "How do I deploy a Spring microservice to AKS?". 

### Test Data Set

A collection of test documents, queries, and evaluators of responses. 

### Runner Inputs

A collection of Test Data Set / Design combinations to be executed by the test runner.

### Test Runner

* Reads inputs
* Instantiates test data sets and designs
* Loads documents from a test data set into a design
* Executes queries and evaluates responses. Response evaluators may provide feedback to the design
* Stores test results for further analysis and report generation

Test runner may execute only parts of the above steps depending on inputs. 
For example:

* There might be already a design with pre-loaded documents and the test runner will execute only the querying part
* Or the test data set may contain only documents, but not queries and response evaluators because queries and responses are to be provided by users via the Web UI
* Test Runner may load documents to the design and save it as a new design. E.g. create a container from an image, load documents, and then stop container and create an image from the container.
* Similarly, the Test Runner may take a test data set, combine it with user provided feedback and create and create a new test data set.

Test runs can be distributed across multiple agents/machines.

### Test Results & User Feedback

Storage of test results and user feedback. 
Test results and user feedback shall reference test data sets and designs. 
As such, it is essentially a harness metadata repository containing design definition trees/graphs, test data set definitions, and results of test runs.

### Report Generator

Generates a report. The report might be in HTML format with visualizations. 
A possible report format:

* Left panel with the designs tree, tasks tree, and test data sets for tasks.
* Content panel - documentation for the selected item. E.g.
    * Home page - a summary of performed tests: filterable sortable table with design/test permutations (for relatively small spaces), visualizations, e.g. [ECharts](https://echarts.apache.org/examples/en/index.html) [3D Scatter](https://echarts.apache.org/examples/en/editor.html?c=scatter3d&gl=1&theme=dark)
    * Design page - configuration, tests and results - table, visualizations
    * Task page - description, tests, designs, visualizations
    
--- Work in progress ---    
    
## Tasks

Dimensions:

* Number of documents
* Number of users
* Frequency of changes
* Privacy
* Risk - cost of error

### Technical documentation

### Procedures

### Operational Documents

## Designs

### Embeddings, vector databases, LLM's

### Graphs

### Polymorphic graphs


