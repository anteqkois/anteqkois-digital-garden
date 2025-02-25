---
title: MongoDb Replica Set
tags:
  - dev
  - database
  - mongodb
  - replicaset
  - devops
---

## Commands to create local mongo replica set

Run docker compose (first time, run with environments MONGO_INITDB_ROOT_PASSWORD etc to create admin. On the linux server they are not always properly readed for no reason.... After initializing mongo, remove environments from compose file)

```
docker compose up -d
```

It problems with auth file access, check: [Keyfile Access](https://www.mongodb.com/docs/manual/tutorial/enforce-keyfile-access-control-in-existing-replica-set/#create-a-keyfile)
or use trick with entrypoint:
```
entrypoint:
  - bash
  - -c
  - |
      chmod 400 /etc/mongo/authKey
      chown 999:999 /etc/mongo/authKey
      exec docker-entrypoint.sh "$@"
```

## Login as a admin
```
docker exec -it mongodb mongo --host mongodb:27017 -u admin -p <password> --authenticationDatabase admin
```

## Setup replica set config (add secondary database)
```
config = {
      "_id" : "rs0",
      "members" : [
        {
          "_id" : 0,
          "host" : "mongodb:27017",
					"priority" : 2
        },
        {
          "_id" : 1,
          "host" : "mongodb2:27017",
					"priority" : 1
        },
      ]
      };
```

## Initialize cluster
```
rs.initiate(config);
```

## Add new node to existing replica set
```
rs.add("mongodb2:27017")
```

## Check current config
```
rs.conf()
```

## Change replica set config by just assign new config (would be better the hosts to be DNS host then ip addresses)
```
use local;

config = {
      "_id" : "rs0",
      "members" : [
        {
          "_id" : 0,
          "host" : "<ip/host>:27017",
					"priority" : 2
        },
        {
          "_id" : 1,
          "host" : "<ip/host>:27018",
					"priority" : 1
        },
      ]
      };

rs.reconfig(config);
```

## How to connect:
You must use `directConnection=true` connection option

```
mongodb://order-book-alts:<password>@mongodb:27017/on_chain_data?replicaSet=rs0&authSource=admin&directConnection=true
```
