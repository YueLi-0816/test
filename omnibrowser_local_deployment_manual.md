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
    ├── database_API.py
    ├── gene_reference
    ├── human_gene.txt
    ├── manage.py
    ├── market
    ├── mus_gene.txt
    ├── requirements.txt
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
├── database_API.py
├── gene_reference
├── human_gene.txt
├── manage.py
├── market
├── mus_gene.txt
├── requirements.txt
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
To always have database running, we suggest using `supervisord` to run **MongoDB** as a sub-process.
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

Now that we have configured the client-side, server-side and have had the database running in the background. The only thing left is to restore pre-formatted data into the database and thereby have data in the client-side.

Let's walk through the following simple process:

#### restore data into mongodb
A data package delivered by Analytical Biosciences will have the following structure.
on the top level will be a `restore.py`, python script for restoring data. The data are all pre-processed into database-compatible formats (MongoDB-BSON, technically) to smooth the data-loading process.

```bash
data
├── restore.py
└── dataset
    ├── No_1_1.tar.gz
    ├── No_2_1.tar.gz
    ├── No_3_1.tar.gz
    ├── No_4_1.tar.gz
    └── ...
```

Use the code restore.py to restore data into MongoDB, no additional python package is required except the built-in modules.
```
python3 restore.py <absolute_path_to_mongorestore>
```
A restore.log file will be generated in the sibling directory and you can learn if any dataset goes wrong.
To remove test or mistakenly-loaded data in the database, you can use the `db.dropDatabase()` command from `mongo shell`: <https://docs.mongodb.com/manual/reference/method/db.dropDatabase/>

#### updating database summary
The `server_side/summary.py` script is used to calculate the summary information of the whole datasets in the MongoDB, including cell number, cell type number, dataset number and artical number.The corresponding numbers will be placed on the front page of the website.
```
python3 summary.py
```
Because the corresponding numbers will change as the datasets change, you are recommended to run it after each update.

#### explore your data
After restoring data into MongoDB, we would be able to access the datasets from the browser via <http://server_ip/omnibrowser/client_side>, as illustrated in **3.1 deployment of client-side step-1** 

<p align="center">
<br><br><br>
### Local deployment guide ends here
<br><br><br>
</p>

<div style="page-break-after: always;"></div>



# 4 Database API walk-through

## 4.1 database architecture
As introduced in chapter `1.3 The database`, for each dataset we store it as a `database` and for each piece of information we store it as a `collection` (i.e. matrix, cell annotations, gene annotations are stored as independent collections). 
Below is the clarified schema and you are encouraged to explore it via `mongo shell`

```bash
├──No_1_1/database
  ├── X_obs_by_var/collection
      ├──{'key':'value'}/document
      ├──{'key':'value'}/document
  ├── X_var_by_obs
  ├── obs
  ├── var
  ├── t-test
  ├── t-test_cluster
  ├── t-test_gene_set_analysis
  ├── wilcoxon
  ├── wilcoxon_cluster
  ├── wilcoxon_gene_set_analysis
  ├── scibet
  └── paga
```

## 4.2 introduction of the API

to handle the I/O of the above-mentioned schema, we wrote a customized Python API, wrapped around Pymongo, an official release of MongoDB API. The idea behind our API is to have a central class binding I/O methods, with each method corresponding to the writing or reading of a single collection. <br>
The input of the API are standard Python data structures (i.e. dict(), numpy.ndarray(), etc) while the output will be database files able to be displayed at the client side. Since each collection works independently, you may simply pass some if not interested.

To use the API we simply:

1. read on-disk data into Python memory
e.g.
```Python
# from tsv
import pandas as pd
obs_by_var_matrix = pd.read_tsv('obs_by_var_matrix.tsv').to_numpy()

# from mtx
from scipy.sparse import mmread
obs_by_var_matrix = mmread('obs_by_var_matrix.mtx')
```

2. write data into MongoDB
```Python
# run the API definition codes first

my_writer = DatabaseAPI('No_1_1')
write_collection_X_obs_by_var(obs_by_var_matrix,overwrite=True)
```

## 4.3 introduction to the central class

class  database_API.DatabaseAPI(target_db)

   Parameters :  target_db(str) - Name of the datasets



   - write_collection_X_obs_by_var(obs_by_var_matrix,overwrite)

         Parameters : obs_by_var(numpy array) - the matrix of cell_by_gene

         overwrite(True or False) - set "overwrite = True" if you want to overwrite the collection

   - write_collection_X_var_by_obs(var_by_obs_matrix,overwrite)

         Parameters : var_by_obs(numpy array) - the matrix of gene_by_cell

         overwrite(True or False) - set "overwrite = True" if you want to overwrite the collection

   - write_collection_var(var,overwrite)

         Parameters : var(Dict[str, list]) - gene annotation("geneSymbol","ensemblID")
		 
         overwrite(True or False) - set "overwrite = True" if you want to overwrite the collection

   - write_collection_obs(obs,overwrite)

         Parameters : obs(Dict[str, list]) - cell annotation("cellID","clusterID","clusterName","sampleID","cellOntologyName",	"cellOntologyID","meta_sourceName","FACSMarker","tSNE1","tSNE2","UMAP1","UMAP2","meta_cellType","clusteringMethod","clusterName_scibet")
		 
         overwrite(True or False) - set "overwrite = True" if you want to overwrite the collecti

   - write_collection_uns_metadata(metadata,overwrite)

         Parameters : metadata(Dict[str, Any]) - metadata information

    	 overwrite(True or False) - set "overwrite = True" if you want to overwrite the collection

   - write_collection_marker(marker,method,overwrite)
   
        genes are used as keys for storage

         Parameters : marker(Dict[str, Any]) - marker gene of each cell type({cluster_1 : {"geneSymbol":[],"statistics":[],"logFC":[],"pValue":[],"qValue":[]},...})

         method(str) - 't-test' or 'wilcoxon'

         overwrite(True or False) - set "overwrite = True" if you want to overwrite the collection

   - write_collection_marker_cluster(marker,method,overwrite)
   
       clusters are used as key for storage

         Parameters : marker(Dict[str, Any]) - marker gene of each cell type({cluster_1 : {"geneSymbol":[],"statistics":[],"logFC":[],"pValue":[],"qValue":[]},...})

         method(str) - 't-test' or 'wilcoxon'

         overwrite(True or False) - set "overwrite = True" if you want to overwrite the collection

   - write_collection_gene_set_analysis(marker,method,overwrite)
   
       calculate  GO terms and save the results,the input is the marker gene

         Parameters : marker(Dict[str, Any]) - marker gene of each cell type({cluster_1 : {"geneSymbol":[],"statistics":[],"logFC":[],"pValue":[],"qValue":[]},...})

         method(str) - 't-test' or 'wilcoxon'

         overwrite(True or False) - set "overwrite = True" if you want to overwrite the collection

   - write_collection_scibet(scibet_npy,genes,cell_types,overwrite)
       
       save the calculation of scibet(calculated result)

         Parameters : scibet_npy(numpy array) - scibet calculation

         gene(numpy array) - gene list

         cell_types(numpyarray) - cell type list

         overwrite(True or False) - set "overwrite = True" if you want to overwrite the collection

   - write_collection_paga(paga,overwrite)
   
       save the calculation of paga(calculated result)

         Parameters : paga(Dict[str, list]) - paga calculation

         overwrite(True or False) - set "overwrite = True" if you want to overwrite the collection
	       
   - read_collection_X_obs_by_var(obs_id)

         Parameters : obs_id(int) - the row you needed

         Returns: obs_value

         Return type: list

   - read_collection_X_var_by_obs(var_id)

         Parameters : var_id(int) - the row you needed

         Returns: var_value

         Return type: list

   - read_collection_var(key)

         Parameters : key(str) - gene key("geneSymbol","ensemblID")

         Returns: gene_value

         Return type: list

   - read_collection_obs(key)

         Parameters : key(str) - cell annotation key("cellID","clusterID","clusterName",...)

         Returns: cell_annotation_value

         Return type: list

   - read_collection_uns_metadata()

         Returns: metadata

         Return type: dictionary

   - read_collection_gene_set_analysis(method,cell_type)

         Parameters : method(str) - "t-test" or "wilcoxon"

         cell_type(str) - the cell type you needed

         Returns: GO terms calculation of a specific cell type

         Return type: dictionary

   - query_collection_X_obs_by_var(obs_ids)

         Parameters : obs_ids(List[int]) - the rows you needed

         Returns: obs_values

         Return type: numpy array

   - query_collection_X_var_by_obs(var_ids)

         Parameters : var_ids(List[int]) - the rows you needed

         Returns: var_values

         Return type: numpy array

   - query_collection_obs()

         Returns: cell annotation

         Return type: Dict[str,list]

   - query_collection_var()

         Returns: gene annotation

         Return type: Dict[str,list]

   - query_collection_uns_metadata()

         Returns: metadata information

         Return type: Dict[str,Any]

   - query_collection_marker(method)

         Parameters : method(str) - "t-test" or "wilcoxon"

         Returns: marker gene and relevant calculation

         Return type: dictionary

   - query_collection_marker_cluster(method)

         Parameters : method(str) - "t-test" or "wilcoxon

         Returns: marker gene and relevant calculation

         Return type: dictionary

   - query_collection_scibet()

         Returns: matrix，genes，cell types

         Return type: numpy array

   - query_collection_paga()
 
         Returns: node_name,node_size,connectivities

         Return type: numpy array

   - query_collection_gene_set_analysis(method)

         Parameters : method(str) - "t-test" or "wilcoxon"

         Returns: gene set

         Return type: dictionary

   - get_collection_X_obs_by_var()

         Returns: obs_by_var_matrix

         Return type: numpy array

   - get_collection_X_var_by_obs()

         Returns:var_by_obs_matrix

         Return type:numpy array

   - get_collection_geneExpression(gene)

         Parameters : gene(str) - the gene you needed

         Returns: value,message

         Return type: list ,str
	
class database_API.Databases()
    
   - get_public_datasets()
      
       get all the datasets
        
         Returns : datasets
	
         Return type : list

  
<div style="page-break-after: always;"></div>



# 5 Contact

Any potential issues, please do not hesitate to contact our technical support at <omnibrowser@abiosciences.com>

