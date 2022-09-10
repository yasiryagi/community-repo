
# Commands 
## Initial settings
### global settings
```
yarn joystream-cli api:setQueryNodeEndpoint
yarn joystream-cli account:import  --backupFilePath /path/to/lead-key.json
yarn joystream-cli working-groups:setDefaultGroup -g storageProviders
```
> Set "global" storage limits to 2000 GB and 200000 files:
```
yarn storage-node leader:update-voucher-limits -s 2000000000000 -o 20000 -k /root/keys/storage-role-key.json
```

> Update/set the dynamic bag policy:
```
yarn storage-node leader:update-dynamic-bag-policy -t Channel -n 5 -k /root/keys/storage-role-key.json
```

> Update the bag limit:
```
yarn storage-node leader:update-bag-limit -l 10 -k /root/keys/storage-role-key.json
```



### hiring
```
yarn joystream-cli working-groups:createOpening -o ~/joystream_working_dir/Strorage_WG_Worker.json
yarn joystream-cli working-groups:openings
yarn joystream-cli working-groups:opening --id 1
yarn joystream-cli working-groups:application 2
yarn joystream-cli working-groups:application 3
yarn joystream-cli working-groups:fillOpening --openingId 1 --applicationIds 2 --applicationIds 3
yarn joystream-cli working-groups:overview
yarn joystream-cli working-groups:createOpening -i ~/joystream_working_dir/Strorage_WG_Worker.json
```
### bucket mgmt
```
yarn storage-node leader:create-bucket -i 18 -n 20000 -s 1500000000000 -k /root/keys/storage-role-key.json -p xxxxxx
yarn storage-node leader:update-bucket-status -i 18 -s off -k /root/keys/storage-role-key.json -p xxxxxx
```

## Upload Test procedure

Needed : 
- URL : https://play.joystream.org/studio/videos
- test bag ID: YYAGI:2705,2706 
- test bag ID: 0x2bc:2222, 2223, 2227 
- Or create new channel in Atlas


Check buckets assigned to the bag 
```
{
  storageBags(limit: 3000, offset: 0, where: {id_eq: "dynamic:channel:2705"}) {
    storageBuckets {
      id
    }
    id
  }
}
```

Remove all buckets and leave only allow bucket to be tested 

```
yarn storage-node leader:update-bag -i dynamic:channel:2705 -k /root/keys/storage-role-key.json -r 17 -p xxxxxxx
yarn storage-node leader:update-bag -i dynamic:channel:2705 -k /root/keys/storage-role-key.json -r 14 -p xxxxxxx
yarn storage-node leader:update-bag -i dynamic:channel:2706 -k /root/keys/storage-role-key.json -a 16 -p xxxxxxx

```

## Overview 
```
yarn joystream-cli working-groups:overview -g=storageProviders
```
```
root@joystream2 ~/joystream # yarn joystream-cli working-groups:overview -g=storageProviders
yarn run v1.22.15
$ /root/joystream/node_modules/.bin/joystream-cli working-groups:overview -g=storageProviders
Initializing the query node connection (http://localhost:8081/graphql)...... done
Initializing the api connection (ws://localhost:9944)...... done
Current Group: storageProviders

___________________ Group lead ___________________

Member id:                    3098
Member handle:                yyagi
Role account:                 5CPhA9RkPnykLxGy1JVy1Y4x9YPSYq9iq48NAMzoKXX9mLG8

____________________ Members _____________________

Worker id     Member id     Member handle       Stake             Reward          Missed reward     Role account
18            2098          0x2bc               40.0000 kJOY      10.0000 JOY     0                 5DX9Vv...g2bDSx
16            3098          yyagi               100.0000 kJOY     21.0000 JOY     0                 5CPhA9...X9mLG8     ⭐ 🔑
15            2137          kalpakci            1.0000 kJOY       6.0000 JOY      0                 5FxJ7z...y35Gcw
14            3082          abramaria_          1.0000 kJOY       6.0000 JOY      0                 5EnCg3...GuNqK1
13            2141          plycho              1.0000 kJOY       6.0000 JOY      0                 5FHAEH...rCeVH8
11            458           sieemma             1.0000 kJOY       6.0000 JOY      0                 5FWcV6...B2uuSS
10            1019          razumv              1.0000 kJOY       6.0000 JOY      0                 5G27n1...VWaopF
9             3886          Craci_BwareLabs     1.0000 kJOY       6.0000 JOY      0                 5HDhUE...hAGoGZ
8             3085          mmx1916             1.0000 kJOY       6.0000 JOY      0                 5HR3fa...V7LAi1
5             1541          godshunter          1.0000 kJOY       6.0000 JOY      0                 5EPUmj...qDvGFv
4             515           l1dev               1.0000 kJOY       6.0000 JOY      0                 5DFb8X...HiJTe8
3             2130          maxlevush           1.0000 kJOY       6.0000 JOY      0                 5Gy9ei...rHQWPe
2             2574          alexznet            1.0000 kJOY       6.0000 JOY      0                 5FpdYU...hoRFjD
1             1305          joystreamstats      1.0000 kJOY       1.0000 JOY      0                 5C8BEb...vXhUP3

_____________________ Legend _____________________

⭐ - Leader
🔑 - Role key available in CLI
Done in 4.54s.
```

## Disable Bucket
```
yarn storage-node leader:update-bucket-status -i 8 -s off -k /root/keys/storage-role-key.json -p xxxxxx
```
## remove bags from Bucket
```
for i in $(seq 2000 2710) ; do
    yarn storage-node leader:update-bag -i dynamic:channel:$i -k /root/keys/storage-role-key.json -r 8 -p xxx
done

or 

curl 'https://joystream2.yyagi.cloud/graphql'  \
     -s \
     -H 'Accept-Encoding: gzip, deflate, br'  \
     -H 'Content-Type: application/json' \
     -H 'Accept: application/json'  \
     -H 'Connection: keep-alive'  \
     -H 'DNT: 1'  \
     -H 'Origin: https://joystream2.yyagi.cloud'  \
     --data-binary '{"query":"query MyQuery { storageBuckets(where: {id_eq: 2}) {  bags { id } } }\n"}'   2>&1\
     | jq . | grep dynamic | sed 's/"id"://g;s/"//g;s/ //g' > bags_file
     
for i in $(cat ~/bags_file) ; do
    yarn storage-node leader:update-bag -i $i -k /root/keys/storage-role-key.json -r 1 -p xxxxx
done
```

## Delete Bucket
Can only delete empty buckets
```
yarn storage-node leader:remove-operator -i 8 -k /root/keys/storage-role-key.json -p xxxxx
yarn storage-node leader:delete-bucket -i 8 -k /root/keys/storage-role-key.json -p xxxxx
```

## Evict worker 
Make sure the bucket is empty and deleted
```
 yarn joystream-cli working-groups:evictWorker 7 --group=storageProviders
 ```
 
 # GraphQL
 > URL: https://graphql-console.subsquid.io/?graphql_api=https://orion.joystream.org/graphql
 
 ## Failed uploads
 
 
 ```
 {
  storageDataObjects(limit: 3000, offset: 0, where: {isAccepted_eq: false, createdAt_gt: "2022-09-02T00:00:00.000Z", createdAt_lt: "2022-09-04T03:00:54.000Z"}) {
    createdAt
    id
    storageBag {
      storageBuckets {
        id
      }
      id
    }
  }
}
```

## Bucket

### Bags in Storage Bucket
```
{
  storageBuckets(where: {id_eq: "6"}) {
    id
    bags {
      id
    }
  }
}
```

### Bucket end point and worker 

```
{
  storageBuckets(where: {id_eq: "2"}) {
    id
    acceptingNewBags
    operatorMetadata {
      nodeEndpoint
      extra
    }
    operatorStatus {
      ... on StorageBucketOperatorStatusActive {
        __typename
        workerId
      }
    }
  }
}


```


## Bags

### Buckets asscoated with a bag
```
{
  storageBags(where: {id_eq: "dynamic:channel:2000"}) {
    storageBuckets {
      id
    }
  }
}
```
## Worker 

```
{
  workers(where: {groupId_eq: "storageWorkingGroup", id_eq: "storageWorkingGroup-16"}) {
    id
    groupId
    membershipId
    membership {
      handle
    }
    status {
      ... on WorkerStatusActive {
        phantom
      }
      ... on WorkerStatusLeaving {
        __typename
      }
      ... on WorkerStatusLeft {
        __typename
      }
      ... on WorkerStatusTerminated {
        __typename
      }
    }
  }
}
```

# Budget

Go [here](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc.joystream.org%3A9944#/explorer) and collect the hashes for begining and end of period.

![image](https://user-images.githubusercontent.com/4862448/189320726-3cd78bbf-ac5f-4c1a-9cdc-dd652c6449ba.png)


Go [here](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc.joystream.org%3A9944#/chainstate):
- StorageWorkingGroup
- Budget
- Paste the hash
- Press "+
![image](https://user-images.githubusercontent.com/4862448/189320458-ef1c163a-2c3a-449d-9937-ec585e8bcea7.png)


# Ref 
> Storage dir :/root/.joystream-storage/

> GraphQL URL : https://graphql-console.subsquid.io/?graphql_api=https://orion.joystream.org/graphql
