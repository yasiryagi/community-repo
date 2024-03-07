###  Missing objects fix for Storage provider
1) Upload data to the pending folder
2) Restart colossus-1 container

```
# Example:
cd /home/data/joystream-storage/pending
wget <link_to_the_file>
cd $HOME/joystream
docker-compose up --detach --force-recreate --remove-orphans colossus-1
```

* if that didn't help:
3) Go to the console using the next link: https://graphql-console.subsquid.io/?graphql_api=https://joystream.yyagi.cloud/graphql) and find the necessary storage bag by request
```
# Example:
query MyQuery {
  storageDataObjects(where: {id_eq: "985358"}) {   <-------------------------- replace 985358 with your object
    storageBag {
      id
      storageBuckets {
        id
      }
    }
    id
  }
}
```

4) You can now explicitly process an object via Polkadot.js (screenshot below)


![img.png](img.png)