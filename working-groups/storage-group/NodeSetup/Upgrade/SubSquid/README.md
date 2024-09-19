


## 0- Make sure you are in the right directory 
```
 cd /your/joystream/directory
```


## 1- Backup exisitng files 
```
cp .env .env.bk
cp docker-compose.yml docker-compose.yml.bk
``` 


## 2- Create new .env and update variables:
[New .env](./.env)
```
wget wget -O /your/joystream/directory/.env https://raw.githubusercontent.com/Joystream/community-repo/master/working-groups/storage-group/NodeSteup/Upgrade/SubSquid/.env
```
Update the below variables in your new .env

```
JOYSTREAM_NODE_WS
KEY_FILE
COLOSSUS_1_WORKER_ID
JOYSTREAM_ES_USERNAME
JOYSTREAM_ES_PASSWORD
DATA_FOLDER
KEY_FOLDER
LOG_FOLDER
ENDPOINT
STORAGESQUIDENDPOINT
```



## 3- Update docker-compose.yml 

[docker-compose.yml](./docker-compose.yml)

Copy directories 'database and entrypoints' in this guide into your joystream directory 

```
wget -O /your/joystream/directory/docker-compose.yml https://raw.githubusercontent.com/Joystream/community-repo/master/working-groups/storage-group/NodeSteup/Upgrade/SubSquid/docker-compose.yml

# Make folder database
mkdir /your/joystream/directory/database
wget -O /your/joystream/directory/database/pg_hba.conf https://raw.githubusercontent.com/Joystream/community-repo/master/working-groups/storage-group/NodeSteup/Upgrade/SubSquid/database/pg_hba.conf
wget -O /your/joystream/directory/database/postgres.conf https://raw.githubusercontent.com/Joystream/community-repo/master/working-groups/storage-group/NodeSteup/Upgrade/SubSquid/database/postgres.conf
```

## 4- Bring up SubSquid 

(be careful you do not want to stop your current storage container, the storage container in the new docker-compose.yml is pointing to SubSquid)
```
docker-compose up --detach squid-db squid-processor squid-graphql-server

```
## 5- Check and monitor 
```
# are all containers up and healthy
docker ps
docker logs -f squid-db --tail 100
docker logs -f squid-processor --tail 100
docker logs -f squid-graphql-server --tail 100
```


## 6- Restart your Colossus server (After Squid is  synced)
```
# Create folder entrypoints
mkdir /your/joystream/directory/entrypoints
wget -O /your/joystream/directory/entrypoints/storage.sh https://raw.githubusercontent.com/Joystream/community-repo/master/working-groups/storage-group/NodeSteup/Upgrade/SubSquid/entrypoints/storage.sh
```


```
docker stop colossus-1
docker rm colossus-1
docker-compose up --detach storage
```

## 7- Stop old query node and storage

```
docker stop graphql-server processor hydra-indexer-gateway indexer redis db 
docker rm graphql-server processor hydra-indexer-gateway indexer redis db 
```

## 8- Check and monitor 
```
# are all containers up and healthy
docker ps
docker logs -f storage --tail 100
```

## 9- Update caddy

```
cd caddy
vim Caddyfile
# disable access to the joystream node and QN (QN should be accessed with the docker name not public URL)
#wss://YOUR.URL/rpc {
#        reverse_proxy joystream-node:9944
#}
#https://YOUR.UR {
#        log {
#                output stdout
#        }
#        route /server/* {
#                uri strip_prefix /server
#                reverse_proxy graphql-server:8081
#        }
#        route /graphql {
#                reverse_proxy graphql-server:8081
#        }
#        route /graphql/* {
#                reverse_proxy graphql-server:8081
#        }
#        route /gateway/* {
#                uri strip_prefix /gateway
#                reverse_proxy graphql-server:4000
#        }
#        route /@apollographql/* {
#                reverse_proxy graphql-server:8081
#        }
#}

#Replace 
reverse_proxy colossus-1:3333
#With
reverse_proxy storage:3333


docker-compose up --detach --force-recreate --remove-orphans caddy
```
## 10- Update prometheus 
If using prometheus prometheus/alerts update  alert.rules with the correct containers names and restart prometheus 
```
docker-compose up --detach --force-recreate --remove-orphans prometheus
```