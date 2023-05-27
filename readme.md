## Run the Construction Executive DB Locally

The database copy we received is a [.bak file of a MSSQL 2022 database](https://thor-studio.box.com/s/9w1bj7a5avhs7ekrmnqmbxgfguknudyd). To examine the DB, the best move is to download it and run it locally using Docker and Azure Data Studio. You can download [Docker](https://www.docker.com/) and [Azure Data Studio](https://azure.microsoft.com/en-us/products/data-studio) - ADS is basically a VS Code instance customized for databases so you can skip it if you have a preferred tool for SQL.

Below is a step-by-step guide to get the database up and running.

## Establish container to get database running

- Download [Docker](https://www.docker.com/) and [Azure Data Studio](https://azure.microsoft.com/en-us/products/data-studio).
- Now open a CLI to more quickly get going with Docker and follow the instructions below:


```
# Downloads official MSSQL Server 2022 Docker Image
docker pull mcr.microsoft.com/mssql/server:2022-latest

# Runs the MSSQL Server 2022 Docker Image that was just downloaded
# Note - you need to change the pw and can customize --name and -h flags
# CLI feedback should be the hash of the current docker image that is now running
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=xyz" -p 1433:1433 --name ce_legacy_mssql_db_container -h ce_legacy_mssql_db_container -d mcr.microsoft.com/mssql/server:2022-latest
> a505efe16812e5ddda4b000aa34b46e89ae4592e65456f083af49fb02404deba (for example)

# Shows current docker images running (you should see the hash returned in the last step as the container ID w/ status 'UP')
docker ps

# Gets you into bash prompt for querying new DB
docker exec -it ce_legacy_mssql_db_container bash

# Once inside bash environment, use sqlcmd to connect / query (should open `1> ` prompt, as cryptic as that is - `exit` to close prompt). 
# You need to create the database inside of the container w/ sqlcmd (your user will automatically be `sa` - system admin)
# Note `mssql@ce_legacy_mssql_db_container:/$` is the prompt showing you the container name you chose above:
mssql@ce_legacy_mssql_db_container:/$ /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "xyz" -Q "CREATE DATABASE ce_legacy_mssql_db"

# Allows you to directly query the Docker container, verify if you actually got the database installed in the docker image correctly
docker exec -it ce_legacy_mssql_db_container /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'xyz'

# Some basic docker commands if you need them (you can control the containers in the desktop GUI as well)
docker stop <your-container-name-here>
docker ps 


```


### Copy .bak into your container (this is an example path I used)
```
docker cp /Users/johnserrao/desktop/ss_constructionexec_com_05152023.bak ce_legacy_mssql_db_container:/media/ss_constructionexec_com_05152023.bak
```

### You need to then query the database you made with Azure Data Studio and map the legacy structure to the new database
```
USE master;

RESTORE DATABASE ce_legacy_mssql_db
FROM DISK = '/media/ss_constructionexec_com_05152023.bak'
WITH
   MOVE 'SmartSite' TO '/var/opt/mssql/data/ss_constructionexec_com.mdf',
   MOVE 'ftrow_index' TO '/var/opt/mssql/data/ftrow_index{26As0daD4dB1-4648-4669-9AF6-B737826855E7}.ndf',
   MOVE 'SmartSite_log' TO '/var/opt/mssql/data/ss_constructionexec_com_1.LDF',
   REPLACE;
```

You should get a dialogue about the migration working. Now you can connect to the DB and look around the tables.









SmartSite	C:\Program Files\Microsoft SQL Server\MSSQL13.SQLEXPRESS\MSSQL\DATA\ss_constructionexec_com.mdf	D	PRIMARY	923992064	35184372080640	1	0	0	a2a7809b-33ca-49c8-a891-d71ec2b6b741	0	0	919601152	4096	1	NULL	11040000000012800055	28024c25-2b04-4699-914d-cd49ca8620ba	0	1	NULL	NULL

ftrow_index	C:\Program Files\Microsoft SQL Server\MSSQL13.SQLEXPRESS\MSSQL\DATA\ftrow_index{26As0daD4dB1-4648-4669-9AF6-B737826855E7}.ndf	D	ftfg_index	1048576	35184372080640	3	2538000000049300001	0	42f3f0f7-eb8b-4996-952a-e58fdcaa9e53	0	0	65536	4096	2	NULL	11040000000012800055	28024c25-2b04-4699-914d-cd49ca8620ba	0	1	NULL	NULL

SmartSite_log	C:\Program Files\Microsoft SQL Server\MSSQL13.SQLEXPRESS\MSSQL\DATA\ss_constructionexec_com_1.LDF	L	NULL	153288704	2199023255552	2	0	0	1a741b35-3537-445a-8f1b-17b7839041ef	0	0	0	4096	0	NULL	0	00000000-0000-0000-0000-000000000000	0	1	NULL	NULL


C:\Program Files\Microsoft SQL Server\MSSQL13.SQLEXPRESS\MSSQL\DATA\ss_constructionexec_com.mdf