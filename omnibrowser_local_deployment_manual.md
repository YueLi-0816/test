# OmniBrowser Local Deployment Manual

<div style="page-break-after: always;"></div>

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

5.  Contact

<div style="page-break-after: always;"></div>



# 1. Introduction on software architecture

In this section we offer a brief introduction on the architecture of the web
portal which serves as a prior knowledge to the deployment.

The web portal takes a traditional MVC (model-view-controller) architecture,
with each module developed independently while connected with standard API
interfaces. We’ll start from the client-side, via the server-side, to the
database.

## 1.1 The client-side

The client-side is developed using the **Vue.js** framework. Delivered code in
production are serialized html & javascript files able to run without any
additional software requirements (i.e. node.js is not required). While delivered
packages are self-sufficient in static files, it accesses database information
via a standardized JSON-API provided by the server-side code.

## 1.2 The server-side

The server-side is developed using the **Python Django** framework. It accesses
the database via a customized API (see below, 1.3 the database) and presents data to
the client-side via a JSON-API.

## 1.3 The database

scRNA-seq data is stored in MongoDB, a NoSQL database, with a customized API
for database I/O operations.

We suggest a very brief tour to understand the database-collection-document
architecture of MongoDB to better understand the system:
<https://docs.mongodb.com/v4.0/core/databases-and-collections/>

<div style="page-break-after: always;"></div>




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
    suggest setting up a separate Python environment to continue the following operations.

<div style="page-break-after: always;"></div>




# 3 Local deployment guide

## 3.0 Setting up Python environment

The delivered package will look like:

```bash
.
├── client_side
│   ├── index.html
│   └── static
└── server_side
    ├── Abio
    ├── algorithm.py
    ├── cellType_by_gene.py
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

2.  install required packages listed in `server_side/requirements.txt`

3.  activate and perform all following operations within the environment


## 3.1 Client-side deployment

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

### Step 1 - Copy the entire client_side package to a internet-accessible directory

e.g. (when the server is configured using Apache)

```bash
cp -r client_side /var/www/html/omnibrowser/
```

open an Internet browser and go to this directory (e.g. `123.456.789.111/omnibrowser/client_side`) to validate internet access.
Display of OmniBrowser welcome page indicates success.

### Step 2 - Configure JSON-API

We will skip and go back to this part after completing **3.2 server-side deployment step 1**

open `/client_side/static/configure.js`, which reads like:

```javascript
window.Global1 = {
  serverSrc: 'http://<server_ip>:<port>',
};
```

replace `<server_ip>` and `<port>` to your specifications.





## 3.2 Server-side deployment

make sure before continue:

- Apache or Nginx has been installed for internet access configuration, we'll use it to configure the `<port>` of JSON-API.
- We're within the isolated Python environment.
    
    
### Step 1 - Run server

the server-side code will look like

```bash
server_side/
├── Abio
├── algorithm.py
├── cellType_by_gene.py
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

### Step 2 - Configure the client-side code

Now that we have had the server running, we should go back to **3.1 client-side deployment - step -2** 
to configure the JSON API for the client side.





## 3.3 Database deployment

### Step 1 - Install MongoDB

`Official guide` of **MongoDB** installation is available at: <https://docs.mongodb.com/manual/administration/install-on-linux/>. Please select the appropriate version for your system.

Installation using the `binary tar ball` is recommended to avoid using `sudo` and we suggest a thorough go-through of the following manual <https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu-tarball/>

### Step 2 - Configure supervisord

The built-in Cache functions of **MongoDB** may usually trigger `Linux OOM-killer` which in turn kills the database process.
To always have database running, we suggest using `supervisord` to run **MongoDB** as a run-process.
`Supervisor` is a client/server system that allows its users to monitor and control a number of processes on UNIX-like operating systems. Learn more at: http://supervisord.org/.

Make sure before continue:

- mongod is not running in the background.
to stop mongod process, You may use either simply `htop` or follow the instructions here:
    <https://docs.mongodb.com/manual/tutorial/manage-mongodb-processes/#stop-mongod-processes>

To configure supervisord:

1. Open `/server_side/supervisord.conf`
    
2. Locate to `[program:mongodb]`

3. Edit `command`
   replace the 3 quoted paths with your specifications
   
   ```bash
   <absolute_path_to_mongod> 
   --dbpath <absolute_path_to_db_path> 
   --logpath <absolute_path_to_log>
   ```
   
   e.g. 
   ```bash
   command=/home/web/data/database/mongodb
   /mongodb-linux-x86_64-ubuntu1804-4.2.1/bin/mongod 
   --dbpath 
   /home/web/data/database/mongodb/db 
   --logpath /home/web/data/database/mongodb/log/mongod.log
   ```
4. Edit `user` 
   
   replace <user> with you Linux user account
   
   ```bash
   user=<user>
   ```
   
5. Save changes

### Step 3 - Run MongoDB from supervisord

Run this command (within Python environment) to start supervisor service.

```
supervisord -c supervisord.conf
```

Run this command to see whether the program is running normally.

```
supervisorctl status
```

If your Linux system is disconnected , you need restart supervisor service when it is connected again.


### Step 4 - Restore data into MongoDB
The data will look like 
```bash
dataset/
├── No_1_1.tar.gz
├── No_2_1.tar.gz
├── No_3_1.tar.gz
├── No_4_1.tar.gz
└── ...
```

Use the code restore.py in **3.2 Run server - step -2** to restore data into MongoDB.
```
python3 restore.py <absolute_path_to_mongorestore>
```

Here we will introduce another two codes in **3.2 Run server - step -2**,summary.py and cellType_by_gene.py.

The summary.py is used to calculate the summary information of the whole datasets in the MongoDB, including cell number, cell type number, dataset number and artical number.The corresponding numbers will be placed on the front page of the website.
```
python3 summary.py
```
Because the corresponding numbers will change as the datasets change, you are recommended to run it regularly for example using crontab.


Through the cellType_by_gene.py, you will get two matrixes of cellType_by_gene.One is for human and another one is for mouse.It will aggregate all of the datasets in the MongoDB to determine whether some gene is the marker gene of some cell type and save the result as '0' or '1'.It is used to find all datasets with a specific gene as marker gene and the corresponding cell type from the website.
```
python3 cellType_by_gene.py
```

*<wenjie Please edit within this range>* <br>


After restoring data into MongoDB, we would be able to access the datasets from the browser via <http://server_ip/omnibrowser/client_side>, as illustrated in **3.1 deployment of client-side step-1** 

<br><br><br>
### Local deployment guide ends here
<br><br><br>

<div style="page-break-after: always;"></div>



# 4 Database API walk-through

*Wenjie would you mind completing this? since I'm not familiar with your final-delivery format.* <br>
*1. please get some mongo shell view of the 10 collections to clarify it* <br>
*2. the direct import may cause confusion, what about we just see this is an API introduction and you may just copy paste the definitions?* <br>
*3. I think we can make things simpler, we just introduce (1) the database architecture (the collections), (2)what is pymongo and how do we use it as an API (3)the central class in the API (4) a very brief introduction on class-method parameters. 
maybe sth like https://geoparse.readthedocs.io/en/latest/GEOparse.html and we just have themselves read the codes* <br>

<div style="page-break-after: always;"></div>



# 5 Contact

Any potential issues, please do not hesitate to contact our technical support at <omnibrowser@abiosciences.com>

