# OmniBrowser Local Deployment Manual


# 0. Introduction

OmniBrowser is a web portal developed by Analytical Biosciences for exploration
of sc-RNAseq datasets. In addition to the official copy maintained by the
licensor, the portal can be deployed on any local server maintained by the
licensee. This manual works through the local deployment process in the
following order:

1.  Introduction on software architecture

2.  System & software requirements

3.  Local deployment guide

4.  Database API walk-through





# 1. Introduction on software architecture

In this section we offer a brief introduction on the architecture of the web
portal which serves as a prior knowledge to the deployment.

The web portal takes a traditional MVC (model-view-controller) architecture,
with each module developed independently while connected with standard API
interfaces. We’ll start from the client-side, via the server-side, to the
database.

## 1.1 The client-side

The client-side is developed using the Vue.js framework. Delivered code in
production are serialized html & javascript files able to run without any
additional software requirements (i.e. node.js is not required). While delivered
packages are self-sufficient in static files, it accesses database information
via a standardized JSON-API provided by the server-side code.

## 1.2 The server-side

The server-side is developed using the Python Django framework. It accesses
the database via a customized API (see below, 1.3 the database) and presents data to
the client-side via a JSON-API.

## 1.3 The database

scRNA-seq data is stored in MongoDB, a NoSQL database, with a customized API
for database I/O operations.

We suggest a very brief tour to understand the database-collection-document
architecture of MongoDB to better understand the system:
<https://docs.mongodb.com/v4.0/core/databases-and-collections/>






# 2. System & software requirements

## 2.1 Operation system requirements

The web portal can be deployed on any Linux server (preferably Ubuntu), as long as the software
requirements can be satisfied.

## 2.2 Software requirements

1.  Internet access configuration  
    Nginx, Apache or other utilities of the same function should be
    pre-installed for internet access configuration.

2.  Python 3.5 +  
    Python 3.5 + is required to run the database API & server-side codes. Any
    previous Python version will cause error since the typing module we include is
    only available after Python 3.5. (<https://docs.python.org/3/library/typing.html>)

3.  Python environments  
    All necessary packages to deploy & run the system is tracked within the
    requirements.txt available within the delivered code package. The environment is 
    sufficient for the deployment & production of all server-side & database codes. We 
    suggest setting up a saperate Python environment to continue the following operations.






# 3 Local deployment guide

**lack general intro here**

## 3.0 setting up Python environment

The delivered package will look like:

```bash
.
├── client_side
│   ├── index.html
│   └── static
└── server_side
    ├── Abio
    ├── algorithm.py
    ├── cell_by_gene.py
    ├── database_API.py
    ├── gene_reference
    ├── human_gene.txt
    ├── manage.py
    ├── market
    ├── mus_gene.txt
    ├── requirements.txt
    ├── restore.py
    ├── summary.py
    └── supervisord.conf
```

<br>

Follow the instructions to set up the environment:

1.  set up a isolated python environment using conda or virtualenv (<https://virtualenv.pypa.io/en/stable/userguide/>)

2.  install required packages listed in server_side/requirements.txt

3.  activate and perform all following operations within the environment


## 3.1 client-side deployment

make sure before continue:

- Apache or Nginx has been installed for internet access configuration.

The client-side package will look like:

```bash
client_side
├── index.html
└── static
    ├── configure.js
    ├── css
    ├── fonts
    ├── img
    ├── js
    ├── scss
    └── vendor
```

Follow the instructions to deploy the client-side:

### step 1 - cp the entire client_side package to a internet-accessible directory

e.g. (when the server is configured using Apache)

```bash
cp -r client_side /var/www/html/omnibrowser/
```

open an Internet browser and go to this directory (e.g. 123.456.789.111/omnibrowser/client_side) to validate internet access.
Display of OmniBrowser welcome page indicates success.

### step 2 - configure JSON-API

We will skip and go back to this part after completing **3.2 server-side deployment step 1**

open /client_side/static/configure.js, which reads like:

```javascript
window.Global1 = {
  serverSrc: 'http://<server_ip>:<port>',
};
```

replace <server_ip> & <port> to your specifications.





## 3.2 server-side deployment

make sure before continue:

- Apache or Nginx has been installed for internet access configuration, we'll use it to configure the <port> of JSON-API.
- We're within the isolated Python environment.
    
    
### step 1 - run server

the server-side code will look like

```bash
server_side/
├── Abio
├── algorithm.py
├── cell_by_gene.py
├── database_API.py
├── gene_reference
├── human_gene.txt
├── manage.py
├── market
├── mus_gene.txt
├── requirements.txt
├── restore.py
├── summary.py
└── supervisord.conf
```

The server_side code can be copied to and run within any directory.
To run server, within the Python environment:

```bash
cd path_to_server_side/server_side/
```

```bash
nohup python3 manage.py runserver 0:<port>
```

Since we used the nohup command, the server will be running in the background no matter whether we close the ssh connection, unless there's operation system reboot.
To shutdown the server, use htop to kill the process:

```bash
htop
```

To check server availability, open any Internet browser, go to 

```bash
http://<server_ip>:<port>/market/monitor
```

on successful deployment, the browser will display a json string:

```json
{"running": "true"}
```

### step 2 - configure the client-side code

Now that we have had the server running, we should go back to **3.1 client-side deployment - step -2** 
to configure the JSON API for the client side.





## 3.3 database deployment

### step 1 - install MongoDB

`Official guide` of MongoDB installation is available at: <https://docs.mongodb.com/manual/administration/install-on-linux/>. Please select the appropriate version for your system.

Installation using the `binary version` is recommended to avoid using `sudo`.
