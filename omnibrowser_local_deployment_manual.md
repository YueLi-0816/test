OmniBrowser Local Deployment Manual
===================================

Introduction
============

OmniBrowser is a web portal developed by Analytical Biosciences for exploration
of sc-RNAseq datasets. In addition to the official copy maintained by the
licensor, the portal can be deployed on any local server maintained by the
licensee. This manual works through the local deployment process in the
following order:

1.  Introduction on software architecture

2.  System & software requirements

3.  Local deployment guide

4.  Database API walk-through

Introduction on software architecture
=====================================

In this section we offer a brief introduction on the architecture of the web
portal which serves as a prior knowledge to the deployment.

The web portal takes a traditional MVC (model-view-controller) architecture,
with each module developed independently while connected with standard API
interfaces. We’ll start from the client-side, via the server-side, to the
database.

The client-side
===============

The client-side is developed using the Vue.js framework. Delivered code in
production are serialized html & javascript files able to run without any
additional software requirements (i.e. node.js is not required). While delivered
packages are self-sufficient in static files, it accesses database information
via a standardized JSON-API provided by the server-side code.

The server-side
===============

The server-side is developed using the python Django framework which accesses
the database via a customized API (see below, the database) and presents data to
the client-side via a JSON-API.

The database
============

scRNA-seq data is stored in MongoDB, a NoSQL database, with a customized API
developed to handle database I/O operations.

We suggest a very brief tour to understand the database-collection-document
architecture of MongoDB to better understand the system:

<https://docs.mongodb.com/v4.0/core/databases-and-collections/>

System & software requirements
==============================

Operation system requirements
=============================

The web portal can be deployed on any Linux server as long as the software
requirements can be satisfied.

Software requirements
=====================

1.  Internet access configuration  
    Nginx, Apache or other utilities of the same function should be
    pre-installed for internet access configuration

2.  Python 3.5 +  
    Python 3.5 + is required to run the database API & server-side codes. Any
    older Python version will cause error since the typing module we include is
    only available after Python3.5.
    (<https://docs.python.org/3/library/typing.html)>

3.  Python environments  
    All necessary packages to deploy & run the system is tracked within the
    requirements.txt available within the delivered code package. We suggest
    performing all following operations within a separated Python environment.

Local deployment guide
======================

Operation system requirements
=============================

The web portal can be deployed on any Linux server as long as the software

Database API walkthrough
========================
