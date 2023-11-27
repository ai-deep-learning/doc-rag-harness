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

### Design

### Design Provider Interface

### Task