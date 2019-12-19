#  Surface  Manual



# Introduction





# Part 1 Deployment And Restoring Data

**## Install MongoDB**

***\*MongoDB\**** is a NoSQL database used for storage of single-cell datasets. `Official guide` of MongoDB installation can be found at: https://docs.mongodb.com/manual/administration/install-on-linux/. Select the appropriate version for your system.

Installation using the `binary version` is recommended to avoid using `sudo`.



Create a directory where the MongoDB instance stores its data. For example:

```
sudo mkdir -p /var/lib/mongo
```

Create a directory where the MongoDB instance stores its log. For example:

```
sudo mkdir -p /var/log/mongodb
```

To run MongoDB, run the [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) process at the system prompt.

```
mongod --dbpath /var/lib/mongo --logpath /var/log/mongodb/mongod.log --fork
```

Verify that MongoDB has started successfully by checking the process output for the following line in the log file `/var/log/mongodb/mongod.log`:

```
[initandlisten] waiting for connections on port 27017
```

Start a [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo) shell on the same host machine as the [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod). You can run the [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo) shell without any command-line options to connect to a [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) that is running on your localhost with default port 27017:

```
mongo
```



Before restoring,make sure that MongoDB has started successfully. Otherwise,it will failed and you need to restart the mongodb.



**# Restore dumped database into the mongodb**

- Required package : `os , sys`.

- You will get a file named as `XXX.tar.gz`.

- Unzip the file.

  ```
  tar -zxvf XXX.tar.gz
  ```

- There will be two parts, a folder named as `Pfizer_data` and a Python source file named `restore.py`.

- Just use `restore.py` to restore the datasets into the mongodb.

  path : the absolute path of `"mongorestore"`   For example : `"User/database/mongodb-linux-x86_64-ubuntu1804-4.0.11/bin/mongorestore"`

  ```
  python3 restore.py  "User/database/mongodb-linux-x86_64-ubuntu1804-4.0.11/bin/mongorestore"
  ```



**# Omnibrowser_web_backend**



**## Introduction**



\* This repo contains the backend(server-side) code for the omnibrowser web portal.

\* The web portal takes a ***\*MVC(model-view-controller)\**** architecture, with the backend responsible for model(database-interaction) and controller (server-client communiations) and the frontend responsible for client-side rendering.

\* The backend is developed using the python ***\*Django\**** framework, with customized APIs to interact with ***\*MongoDB\**** where single-cell datasets are stored, and ***\*Sqlite3\**** where user data and user comments are stored. The backend interacts with the databases and provides a ***\*JSON-API\**** where the frontend access its data.



**## Function**



\* single-cell dataset database interaction

  \* query the index of all datasets

  \* query metadata or obs of a dataset

  \* compute differential expression of a dataset

  \* compute correlation of gene expression

  \* compute marker genes of a dataset

  \* gene ontology analysis

  \* query the statistical summary of all datasets

\* user system data interaction

  \* user system database interaction

  \* user sign up, login and logout

  \* Email address authenciation

  \* query user information

  \* delete a user

  \* reset the password, user type, access authority for datasets, expiration time of a user

  \* generate a new VIP registration code

  \* comment system database interaction

  \* post a new comment

  \* query all existing comments



**## Deployment**



**### 1. Deployment of the backend**



The backend can be deployed on any Linux server, preferably Ubuntu.



**#### Step 1 - Clone the repo** 



Clone the repo to your server. Download file ```gene_reference``` from ? (Sorry, this file is too large to be put on github).



**#### Step 2 - Set up the environment**



Under the repo clone's directory, within a python virtualenv, run



\```

pip install -r requirement.txt

\```



**#### Step 4 - Sqlite3 database configuration**



Under the repo clone's directory, within a python virtualenv, run



\```

sqlite3 user_info.db

\```



A ```user_info.db``` file will be generated under the current directory.



Run these commands to create tables in the Sqlite3 database.



\```

python manage.py makemigrations UserSystem

python manage.py migrate

\```



**#### Step 3 - Customize the backend settings**



In ```/Abio/settings.py/```, customize following fields for SMTP service (sending emails).



\* EMAIL_HOST

\* EMAIL_PORT

\* EMAIL_HOST_USER

\* EMAIL_HOST_PASSWORD

\* EMAIL_USE_TLS

\* EMAIL_USE_SSL



Email is sent using the SMTP host and port specified in the `EMAIL_HOST` and `EMAIL_PORT` settings. The`EMAIL_HOST_USER` and `EMAIL_HOST_PASSWORD` settings are used to authenticate to the SMTP server, and the `EMAIL_USE_TLS` and `EMAIL_USE_SSL` settings control whether a secure connection is used. 



**#### Step 4 -Run server using Supervisor**



Open ``` supervisord.conf ``` using VIM, edit following items:



\* Under ```[inet_http_server]```, edit ```port``` as ```127.0.0.1:<port1>``` .

\* Under ```[program:mongodb]```, edit ```command``` as ```mongod --dbpath /home/web/data/database/mongodb/db --logpath /home/web/data/database/mongodb/log/mongod.log```



\* Under ```[program:web_backend]```, edit ```command``` as ```<absolute path of virtualenv python> <absolute path of manage.py> runserver 0:<port2>```.

\* Under ```[program:website_monitor]```, edit ```command``` as ```<absolute path of virtualenv python> <absolute path of website_monitor.py>```.

\* Under [program:clear_expired_common_accounts], edit command as ```<absolute path of virtualenv python> <absolute path of clear_expired_common_accounts.py>```.



Run this command to start supervisor service.



\```

supervisord -c supervisord.conf

\```



Open a browser and go to ```http://127.0.0.1:<port2>``` to see whether these 5 programs are running normally.



**### 2. Configure the JSON API**



**#### Step 1 Check JSON API**



After deploying the backend and loading data into the DB, dataset info should be available on the JSON API. Simply open a browser and go to 

```
http://<server_IP>:<port2>/market/index_of_all_datasets
```

to see whether data is available via Http GET request.



**#### Step 2 Configure the server IP in the frontend framework**



As the front end sends requests to the JSON API for data, the `Root Server IP` should be configured in the frontend source code to adapt to your deployment server. 

Details please see instructions on frontend deployment.



**# Analytical Biosciences deployment**

**### This repo contains the frontend(browser-side) code for the genentech single-cell web portal.**



The fronted is developed by using the Vue.js framework which is a progressive framework for building user interfaces.At the same time, we use a lot of visualization tools(eg. plotly.js ) to improve the user experience.

**### 1.deployment of the frontend**

The frontend can run on all current mainstream browsers.

**#### step 1-clone the repo**

\- clone the repo to your computer

**#### step 2-set up the environment**

\- 1.download and install Node.js

 \- Download the installation kit on the website(https://nodejs.org/en/).

\- 2.intsall cnpm

  \- Since npm is built into Node.js, we do not need to install npm again.But because some npm resources are blocked or foreign,

it often leads to failure to install dependency packages with npm, so I also need the npm's domestic mirror -- cnpm.Open your command,run 

npm install -g cnpm --registry=http://registry.npm.taobao.org

\- 3.install webpack

 \- npm install webpack -g

\- 4.install vue-cli

 \- cnpm install vue-cli -g

**#### step 3-install dependency in project path**

\- 1.go to project folder

 \- Open your command,run 

cd project path

\- 2.install dependency

 \- npm install or(cnpm install)

 \- Note:If you run the npm install report an error, you can delete the node_module folder and run the "npm install" command again.

**#### step 4-run project**

 \- Open your command,run 

 \- npm run dev or(cnpm run dev)

**#### step 5-pack and deploy servers**

\- 1.pack the frontend

 \- Open your command,run 

 \- npm run build or(cnpm run build)

\- 2.deploy the frontend on the server

 \- After the frontend is packaged, there is a static directory and index.html file under the dist directory. You can upload these two files directly to the server.

**### 2.configure the JSON API**

**#### modify the JSON API**

\- The Root Server IP and Port should be modified in the frontend source code to adapt to your deployment server.

\- To modify the the Root Server IP and Port:

  \- 1.open global.vue file

file path:project folder/src/page

  \- 2.modify the the Root Server IP and Port

Pay attention to serverSrc. 

It says “const serverSrc = 'http://111.100.100.111:8000';” in the code.you can replace '111.100.100.111' with your Root Server IP and modify the '8000' to adapt the right port for the server you're deploying.



  \- 3.access this item on the browser

Contents of the website can be accessed on http://<server_IP>/path(static directory and index.html stored on the server)

# Part 2 Load Own Data Into MongoDB

## Introduction

The API provided is used for you to load your own data into MongoDB. There are  ten collections which are independent of each other. You can use the API according to your existing data  or the results of further calculation. You can just pass some function if you do not have the data as input . Each function has a fixed input format. We provided an example in the following introduction.

## Writing data into MongoDB

### Required packages

- System module : os , typing

- Third-party packages : pymongo , numpy , pandas

- Own module : algorithm



### 1. Import the `API`

```
>>> import database_API
```

   

### 2. Use the class `DatabaseAPI`

```
>>> my_write = database_API.DatabaseAPI("No_1_1")
```

 "No_1_1" is your database name

### 3. Create different collections

- ① collection name :'X_obs_by_var'(the matrix of cell_by_gene)

​            If the matrix is too large , you need to downsample firstly.

​             format : numpy array

```
>>> obs_by_var = [[  8.81681845,  1.86101436,  0.    , ..., 1075.3386941 ,

​     13.86479859,  4.19443915],

​    [  4.79722197,  1.85728085,  0.    , ..., 543.88052284,

​      7.10723373,  1.85909992],

​    [  8.72143929,  2.51605233,  0.    , ..., 633.50072299,

​      8.14137582,  1.58583213],

​    ...,

​    [  5.89335936,  2.87069351,  0.    , ..., 1038.53328297,

​     19.17375322,  4.95795392],

​    [  3.18538671,  3.29554885,  0.    , ..., 389.23097883,

​     10.03333826,  2.90586704],

​    [  4.15034603,  2.94515593,  0.    , ..., 443.73716544,

​     11.41476906,  2.01794376]]
```

```
>>> my_write.write_collection_X_obs_by_var(obs_by_var_matrix = obs_by_var,overwrite = True)
```

set "overwrite = True" if you want to overwrite the collection

- ② collection name :'X_var_by_obs'(the matrix of gene_by_cell)

​            If the matrix is too large , you need to downsample firstly.

​			format : numpy array

```
>>> var_by_obs = [[  8.81681845,  4.79722197,            8.72143929, ...,  5.89335936,3.18538671,       4.15034603],[  1.86101436,  1.85728085,     2.51605233, ...,  2.87069351,3.29554885,  2.94515593],[    0.    ,  0.    ,  0.    , ...,  0.    ,0.    ,  0.    ],

​    ...,[1075.3386941 , 543.88052284, 633.50072299, ..., 1038.53328297,

​     389.23097883, 443.73716544],

​    [ 13.86479859,  7.10723373,  8.14137582, ...,  19.17375322,

​     10.03333826,  11.41476906],

​    [  4.19443915,  1.85909992,  1.58583213, ...,  4.95795392,

​      2.90586704,  2.01794376]]
```

```
>>> my_write.write_collection_X_var_by_obs(var_by_obs_matrix = var_by_obs,overwrite = False)
```

 set "overwrite = True" if you want to overwrite the collection

- ③ collection name : 'var'(gene annotation)

​            The number of genes must match the matrix

​			format : Dict[str, list]

```
>>> var = {'geneSymbol': ['A1BG', 'A1BG-AS1', 'A1CF', 'A2M', 'A2M-AS1', 'A2ML1', 'A2MP1', 'A4GALT', 'A4GNT', 'AA06',...],'ensemblID': ['ENSG00000121410', 'ENSG00000268895', 'ENSG00000148584', 'ENSG00000175899', 'ENSG00000245105', 'ENSG00000166535', 'ENSG00000256069', 'ENSG00000128274', 'ENSG00000118017', 'ENSG00000265544',...],...}
```

```
>>> my_write.write_collection_var(var = var,overwrite = False)
```

set "overwrite = True" if you want to overwrite the collection

- ④ collection name : 'obs'(cell annotation)

​            The number of cells must match the matrix

​			format : Dict[str, list]

```
>>> obs = {'cellID': ['PTC141_0205', 'PTC142_0205', 'PTC143_0205', 'PTC144_0205', 'PTC17_0205', 'PTC24_0205', 'PTC25_0205', 'PTC27_0205', 'PTC29_0205', 'PTC38_0205', ...],'clusterID': ['cluster_1', 'cluster_1', 'cluster_1', 'cluster_1', 'cluster_1', 'cluster_1', 'cluster_1', 'cluster_1', 'cluster_1', 'cluster_1',...],'clusterName': ['CD8_SLC4A10_P', 'CD8_CX3CR1', 'CD8_SELL_Grp2', 'CD8_CX3CR1', 'CD8_CX3CR1', 'CD8_SLC4A10_P', 'CD8_SLC4A10_P', 'CD8_CX3CR1', 'CD8_SLC4A10_P', 'CD8_CX3CR1', ...],...]
```

```
>>> my_write.write_collection_obs(obs = obs,overwrite = False)
```

set "overwrite = True" if you want to overwrite the collection

- ⑤ collection name : 'uns_metadata'(metadata about the database)

   format : Dict[str, Any]

```
>>> metadata = {'datasetID': 'No_1_part_1', 'title': '', 'accessionNumber': '', 'abstract': '', 'source': '', 'sourceID': '', 'numberOfCells': 5063, 'sequencingPlatform': '', 'pubmedID': , 'clusteringMethod': '', 'biomarkerDerivationMethod': '', 'fastqURL': '', 'figureURL': '', 'taxonomyID': 9606, 'subDataset': '', 'description': '', 'libraryPreparationMethod': '', 'genomeBuild': '', 'annotation': '', 'journal': 'Cell', 'publicationDate': '', 'citation': , 'tissue': ['liver'], 'disease': 'True', 'methodology': 'False', 'nonAdult': 'False', 'cancer': 'True', 'neuroscience': 'False', 'developmentalBiology': 'False', 'immunology': 'True', 'isFigurePublic': 'True', 'cellAtlas': 'False', 'clusterAvailability': 'True', 'isBadtSNE': 'False', 'authors': [{'lastname': '', 'firstname': '', 'initials': 'C'}, {'lastname': '', 'firstname': '', 'initials': ''}, {'lastname': '', 'firstname': '', 'initials': ''}, {'lastname': '', 'firstname': '', 'initials': ''}, {'lastname': '', 'firstname': '', 'initials': ''}, {'lastname': '', 'firstname': '', 'initials': ''}, {'lastname': '', 'firstname': '', 'initials': ''}, {'lastname': '', 'firstname': '', 'initials': ''}, ...], 'copyright': '', 'tissueOntology': [''], 'down_numberOfcells': }
```

```
>>> my_write.write_collection_uns_metadata(metadata = metadata,overwrite = False)
```

set "overwrite = True" if you want to overwrite the collection

- ⑥ collection name : 't-test' or 'wilcoxon'(calculation about marker gene)

​            In this collection, genes are used as keys. 

​			format : Dict[str, Any]

```
>>> marker = {"cluster_4": {"geneSymbol": ["STMN2", "DCX", "TUBB3", "DPYSL3", "NCAM1",...],"logFC": [8.83183479309082, 8.738519668579102, 6.778541088104248, 7.197245121002197, 6.992650508880615,...],"pValue": [1.2880483081674684e-128, 4.686245140046686e-115, 3.0909383321304324e-82, 3.0851563608915224e-62, 1.6698326424573794e-60,...],"qValue": [3.40663016544132e-124, 6.197090573197737e-111, 2.7249712336061894e-78, 1.6319243086571796e-58, 7.360622287952129e-57, ...],"cluster_1": {...},...}
```

```
>>> my_write.write_collection_marker(marker = marker,method = 't-test',overwrite = False)
```

set "overwrite = True" if you want to overwrite the collection

```
>>> my_write.write_collection_marker(marker = marker,method = 'wilcoxon',overwrite = False)
```

set "overwrite = True" if you want to overwrite the collection

- ⑦ collection name : 't-test_cluster' or 'wilcoxon_cluster'(calculation about marker gene)

​            In this collections, clusters are used as keys.

​			format : Dict[str, Any]

```
>>> marker = {"cluster_4": {"geneSymbol": ["STMN2", "DCX", "TUBB3", "DPYSL3", "NCAM1",...],"logFC": [8.83183479309082, 8.738519668579102, 6.778541088104248, 7.197245121002197, 6.992650508880615,...],"pValue": [1.2880483081674684e-128, 4.686245140046686e-115, 3.0909383321304324e-82, 3.0851563608915224e-62, 1.6698326424573794e-60,...],"qValue": [3.40663016544132e-124, 6.197090573197737e-111, 2.7249712336061894e-78, 1.6319243086571796e-58, 7.360622287952129e-57, ...],"cluster_1": {...},...}
```

```
>>> my_write.write_collection_marker_cluster(marker = marker,method = 't-test',overwrite = False)
```

set "overwrite = True" if you want to overwrite the collection

```
my_write.write_collection_marker_cluster(marker = marker,method = 'wilcoxon',overwrite = False)
```

set "overwrite = True" if you want to overwrite the collection

- ⑧ collection name : 't-test_gene_set_analysis' or 'wilcoxon_gene_set_analysis'(calulation of GO terms by the marker genes)

​        This calculation needs the file algorithm.py and gene reference.

​		format : Dict[str, Any]

```
>>> marker = {"cluster_4": {"geneSymbol": ["STMN2", "DCX", "TUBB3", "DPYSL3", "NCAM1",...],"logFC": [8.83183479309082, 8.738519668579102, 6.778541088104248, 7.197245121002197, 6.992650508880615,...],"pValue": [1.2880483081674684e-128, 4.686245140046686e-115, 3.0909383321304324e-82, 3.0851563608915224e-62, 1.6698326424573794e-60,...],"qValue": [3.40663016544132e-124, 6.197090573197737e-111, 2.7249712336061894e-78, 1.6319243086571796e-58, 7.360622287952129e-57, ...],"cluster_1": {...},...}
```

```
>>> my_write.write_collection_gene_set_analysis(marker = marker,method = 't-test',overwrite = False)
```

set "overwrite = True" if you want to overwrite the collection

```
>>> my_write.write_collection_gene_set_analysis(marker = marker,method = 'wilcoxon',overwrite = False)
```

set "overwrite = True" if you want to overwrite the collection

- ⑨ collection name : 'scibet'(calculation of scibet)

  format : numpy array

```
>>> scibet_npy = [[1.91600967e-05 2.21634790e-04 1.48657706e-02 ... 1.92950301e-05

 2.97740121e-04 1.81086929e-04]

 [2.84816458e-02 1.48412703e-05 8.86187122e-03 ... 4.19355388e-05

 3.49396460e-04 1.45434702e-05]

 [1.88606399e-05 5.00889319e-04 1.12634651e-02 ... 1.55657809e-05

 1.52463129e-03 2.86294360e-05]

 ...

 [4.60650358e-04 1.91860743e-05 2.30086704e-05 ... 2.68337343e-04

 2.00495644e-05 5.11610682e-05]

 [1.76884126e-05 9.85777618e-05 1.59182631e-05 ... 6.39648547e-05

 1.37833742e-04 2.63434845e-05]

 [4.13793897e-04 1.55425917e-05 1.75096532e-05 ... 1.55425917e-05

 9.34204131e-05 2.46767253e-05]]
```

format : numpy array

```
>>> genes = ['CD4_FOXP3-CTLA4' 'CD4_GZMA_cytotoxic' 'CD4_GZMA_exhausted'

 'CD4_GZMA_main' 'CD4_P_PTR' 'CD4_P_other' 'CD8_CX3CR1' 'CD8_RGS1_GZMK'

 'CD8_RGS1_HAVCR2_ITGAE_FOXP3' 'CD8_RGS1_HAVCR2_ITGAE_HOPX'

 'CD8_RGS1_HAVCR2_SLAMF7' 'CD8_SELL_CCL5' 'CD8_SELL_Grp2' 'CD8_SLC4A10_NT'

 'CD8_SLC4A10_P']
```

format : numpy array

```
>>> cell_types = ['FGFBP2' 'KRT86' 'CD4' ... 'ARMCX2' 'CDCA7' 'RASAL1']
```

```
my_write.write_collection_scibet(scibet_npy = scibet_npy,genes = genes,cell_types = cell_types,overwrite: bool = False)
```

set "overwrite = True" if you want to overwrite the collection

- ⑩ collection name : 'paga'(calculation of paga)

```
>>> paga = {"node_name": ["CD4_FOXP3-CTLA4", "CD4_GZMA_cytotoxic", "CD4_GZMA_exhausted", "CD4_GZMA_main", "CD4_P_PTR", "CD4_P_other", "CD8_CX3CR1", "CD8_RGS1_GZMK", "CD8_RGS1_HAVCR2_ITGAE_FOXP3", "CD8_RGS1_HAVCR2_ITGAE_HOPX", "CD8_RGS1_HAVCR2_SLAMF7", "CD8_SELL_CCL5", "CD8_SELL_Grp2", "CD8_SLC4A10_NT", "CD8_SLC4A10_P"], "node_size": [582, 167, 146, 689, 261, 646, 288, 463, 13, 43, 194, 69, 84, 241, 122], "connectivities": [[0, 1, 0.025766765438195772], [0, 2, 1.0], [0, 3, 0.527107990563544], [0, 4, 0.5836320456610183], [0, 5, 0.1332213569095571], [0, 6, 0.017929374284077895], [0, 7, 0.102232285334699], [0, 9, 0.24017022296811316], [0, 10, 0.14195628299146207], [0, 11, 0.012472608197619403], [0, 13, 0.08213290128473856], [0, 14, 0.021162540138583742], [1, 2, 0.12325691083586252], [1, 3, 0.31777265498031515], [1, 4, 0.09193108036799963], [1, 5, 0.06964206262397805], [1, 7, 0.45345055030328113], [1, 10, 0.1855207111550096], [1, 11, 0.08693482600017356], [1, 13, 0.22401048525355927], [1, 14, 0.8112729459114557], [2, 4, 0.5520587309085183], [2, 5, 0.31332568175071035], [2, 6, 0.011911981544901066], [2, 7, 0.42234792449480757], [2, 8, 0.26389620653319285], [2, 10, 0.5658805253495269], [2, 11, 0.04971957514393488], [2, 13, 0.15658571590973683], [2, 14, 0.05624017516281159], [3, 4, 0.5904804008252285], [3, 5, 0.6808210737507133], [3, 6, 0.08077326237703596], [3, 7, 0.5165650753745216], [3, 8, 0.055919950876409504], [3, 9, 0.2535904749046478], [3, 10, 0.13864688851315968], [3, 11, 0.5794603605309101], [3, 12, 0.008654278111825281], [3, 14, 0.9474306431273645], [4, 7, 0.02901388578568887], [4, 8, 0.14762010020630711], [4, 11, 0.30593730912321615], [4, 12, 0.6396871008939975], [4, 13, 0.06370327975707858], [4, 14, 0.0943800640663275], [5, 6, 0.021537452700378396], [5, 7, 0.03349236705026446], [5, 10, 0.007993281414573426], [5, 11, 0.5955573877148113], [5, 13, 0.09651638554526419], [5, 14, 0.14617221996650256], [6, 8, 0.26756143162393164], [6, 9, 0.20222666343669252], [6, 10, 0.6902809099369989], [6, 11, 0.3528708735909823], [6, 13, 0.14432774319963118], [6, 14, 0.6842554644808744], [7, 12, 0.15454335081764886], [7, 13, 0.8259412276063558], [8, 13, 0.4796121927864666], [9, 13, 0.43499710508539996], [9, 14, 0.2864325200152497], [10, 11, 0.710938667264306], [10, 12, 0.06147214040255277], [10, 13, 0.44994545921204604], [10, 14, 0.2116254013858374], [11, 13, 0.5120497323952132], [12, 13, 0.024741898834222488]]}
```

```
>>> my_write.write_collection_paga(paga = paga,overwrite = False)
```

set "overwrite = True" if you want to overwrite the collection

## Getting the collection of some database

### Required packages



- System module : os, typing

- Third-party packages : pymongo, numpy, pandas

- Own module : algorithm



### 1. Import the `API`

```
>>> import database_API
```

### 2. Use the class `DatabaseAPI`

```
>>> my_read = database_API.DatabaseAPI("No_1_1")
```

"No_1_1" is the database name that you need

### 3. Getting the collection

- ① collection name : 'X_obs_by_var'(the matrix of cell_by_gene)

```
>>> my_read.read_collection_X_obs_by_var(obs_id = 1)
```

obs_id : `int`(The specific row) return a `list`

```
>>> my_read.query_collection_X_obs_by_var(obs_ids = [1,3,5,7])
```

obs_ids : `List[int]` (The specific rows) return a `numpy array`

```
>>> my_read.get_collection_X_obs_by_var()
```

(the whole matrix) return a `numpy array`

- ② collection name :'X_var_by_obs'(the matrix of gene_by_cell)

```
>>> my_read.read_collection_X_var_by_obs(var_id = 1)
```

var_id : `int`(The specific row) return a `list`

```
>>> my_read.query_collection_X_var_by_obs(var_ids = [1,3,5,7])
```

var_ids : `List[int]` (The specific rows) return a `numpy array`

```
>>> my_read.get_collection_X_var_by_obs()
```

(the whole matrix) return a `numpy array`

- ③ collection name : 'var'(gene annotation)

```
>>> my_read.read_collection_var(key = "geneSymbol")
```

key : `str` return the `value(list)` of a given key in var 

```
>>> my_read.query_collection_var()
```

return the `value([Dict[str, List]])` of all keys in var

- ④ collection name : 'obs'(cell annotation)

```
>>> my_read.read_collection_obs(key = "cellID")
```

key : `str` returns the `value(list)` of a given key in obs 

```
>>> my_read.query_collection_obs()
```

return the `value([Dict[str, List]])` of all keys in obs

- ⑤ collection name : 'uns_metadata'(metadata about the database)

```
my_read.read_collection_uns_metadata()
```

return a metadata   `dictionary` with keys being metadata fields

```
>>> my_read.query_collection_uns_metadata()
```

return a metadata   `dictionary` with keys being metadata fields

- ⑥ collection name : 't-test' or 'wilcoxon'(calculation about marker gene)

```
>>> my_read.query_collection_marker(method = 't-test')
```

return a marker gene `dictionary` with genes as keys 

```
>>> my_read.query_collection_marker(method = 'wilcoxon')
```

return a marker gene `dictionary` with genes as keys 

- ⑦ collection name : 't-test_cluster' or 'wilcoxon_cluster'(calculation about marker gene)

```
>>> my_read.query_collection_marker_cluster(method = 't-test')
```

return a marker gene `dictionary` with clusters as keys 

```
>>> my_read.query_collection_marker_cluster(method = 'wilcoxon')
```

return a marker gene `dictionary` with clusters as keys 

- ⑧ collection name : 't-test_gene_set_analysis' or 'wilcoxon_gene_set_analysis'(calulation of GO terms by the marker genes)

``` 
>>> my_read.read_collection_gene_set_analysis(method = 't-test',cell_type = 'CD4_FOXP3-CTLA4')
```

return a `dictionary` of specific cell type

```
>>> my_read.read_collection_gene_set_analysis(method = 'wilcoxon',cell_type = 'CD4_FOXP3-CTLA4')
```

return a `dictionary` of specific cell type

```
>>> my_read.query_collection_gene_set_analysis(method = 't-test')
```

return `dictionary`

```
>>> my_read.query_collection_gene_set_analysis(method = 'wilcoxon')
```

return `dictionary`

- ⑨ collection name : 'scibet'(calculation of scibet)

```
>>> my_read.query_collection_scibet()
```

return scibet_npy,genes,cell_types in the format of `numpy array`

- ⑩ collection name : 'paga'(calculation of paga)

```
>>> my_read.query_collection_paga()
```

return node_name,node_size,connectivities in the format of `numpy array`



