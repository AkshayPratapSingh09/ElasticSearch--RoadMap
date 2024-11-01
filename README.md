# Elastic Search

Elasticsearch is a distributed, open-source search and analytics engine designed for handling large volumes of data in real-time. It provides powerful full-text search, filtering, and aggregation capabilities, making it ideal for use cases like log analysis, data indexing, and enabling fast, relevant search across large datasets.


## Elastic Stack

- Elastic Search is a part of a bigger elastic Stack
- Elastic Stack Includes - `Elastic Search`, `Kibana`,`Logstash`,``

- **LogStash** - Data pipeline used to handle data from one app to another (Kafka/Elastic Search etc)
- **Elastic Search** - Full text searh Engine
- **Kibana** - Dashboard for visualization and Analytics for Elastic Search
- ***X Pack*** - Addon features for Elastic Search/ Kibana

### Key Add-on Features

- **Security**
  - Authentication: Supports multiple authentication mechanisms (e.g., LDAP, Active Directory, OAuth).
  - Role-based access control.
 
- **Monitoring**
  - Track cluster health: Provides real-time monitoring of Elasticsearch clusters and Kibana.
  - Performance metrics: Detailed metrics on node performance, indexing, and search operations.
 
- **Alerting**
  - Rule-based alerts: Configure rules that trigger alerts when conditions are met (e.g., when disk space is low).
  - Multiple alert channels: Send notifications via email, Slack, or webhooks.

- **Machine Learning**
  - Anomaly detection: Detect anomalies in time-series data automatically.
  - Forecasting: Predict future trends based on historical data.

- **Graph**
  - Explore relationships: Visualize and explore relationships between documents in Elasticsearch.
  - Discover patterns: Identify hidden connections in your data.

- **Graph**
  - Explore relationships: Visualize and explore relationships between documents in Elasticsearch.
  - Discover patterns: Identify hidden connections in your data.
## FAQ

#### Question 1

Answer 1

#### Question 2

Answer 2


## 🚀 About Me
I'm a full stack developer...


## Roadmap

- Additional browser support

- Add more integrations


## Screenshots

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)


## Related

Here are some related projects

[Awesome README](https://github.com/matiassingers/awesome-readme)


## Run Locally

Run Elasticsearch & Kibana
[Elasticsearch Doc.](https://hub.docker.com/_/elasticsearch)
[Kibana Doc.](https://hub.docker.com/_/kibana)

- Just need to run them and on a single network

- Create Network
```
docker network create elasticnetwork
```
- Now Run Elasticsearch
```bash
  docker run -d --name elasticsearch --net elasticnetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:8.15.2
```
- Now Run Kibana (For visualization)
```bash
  docker run -d --name kibana --net elasticnetwork -p 5601:5601 kibana:8.15.2
```
- Now we will need to run Kibana on web

```
http://localhost:5601/
```

- Now It will ask for token (In the elasticsearch terminal)
```
bin/elasticsearch-create-enrollment-token --scope kibana
```
- Now It will ask for Username/Password
- By default (username - `elastic`)
- password is given in the terminal when elasticsearch service is started ! if Missed  
```
bin/elasticsearch-setup-passwords interactive
```
**Note : It will ask to reset all passwords for all service (Not username)**


- Now On this webpage `Management\Dev Tools` can be used to try querys on the service





****

##  **Note - Internally the query is calling the RESTApi**

## Queries❓

- All the internal query starts with "_" like "_cat" , "_cluster" 
- Can check info about the Default Cluster created by the *Elasticsearch Instance* 

```
GET _cluster/health
```
- Get Info About the Nodes 

```
GET _cat/nodes?v
```
- Get Info About the Tables(indices) already Present  

```
GET _cat/indices?v&expand_wildcards=all
```
* 🟩 --> healthy
* 🔴 --> unhealthy

## **Note - For Running These Queries on CMD will have to push CURL Request**

 - `-k` - It will bypass the license check
- `-u` - It will supply the user authentication

```
curl -X GET -k -u {username}:{password} {elasticsearch Endpoint}
```
```
curl -X GET -k -u elastic:123456 https://localhost:9200
```

- Accessing above query using CMD
```
curl -X GET -k -u elastic:123456 https://localhost:9200/_cluster/health
```

## #Sharding

* Term for splitting/diving (Not Copy) the data
* Divides indices into smaller pieces
* Each peice is **Shard**
* It is done at index level
* Improves performance by Parallelization of Queries
**`1 Shard` limited to 2 Billion documents**

- To split the data into Shard - `**Split Api**`
- To reduce the data from multiple shard into less Shard - `**Shrink Api**`

![Alt text](https://static.pingcap.com/files/2024/02/14133212/Range-Based-Sharding.png)
![Alt text](https://codingexplained.com/wp-content/uploads/1-1024x591.png)



## #Replication
- Making Copies of Shard to ensure **availability** 
- Groups of Shards with Same Indexes are **Replication Group** 
- `Primary Shard` - to make copies of 
- `Replica Shard` - Copies of the primary Shard
- The Replica Shard are made and assigned on **Different Nodes**
- They will remain unassigned until assigned **Another Node**

- Make new index in  the Cluster (It will make fruits index) 
```
PUT /fruits
```
- Check for the Indices Now
```
GET /_cat/indices?v
```

| Health Status | Index  | UUID                           | Pri | Rep | Docs Count | Docs Deleted | Store Size | Pri Store Size | Dataset Size |
|---------------|--------|--------------------------------|-----|-----|------------|--------------|------------|----------------|--------------|
| yellow        | open   | fruits                         | 1   | 1   | 0          | 0            | 227b       | 227b           | 227b         |

- 🟡 - unhealthy (Only 1 Node **No Fault Tolerance**)
- Already there is 1 Rep(Replica) & 1 Pri(Primary) shard
- **1 Rep(Replica)** is not assigned

- Get the Info of Shards
```
GET _cat/shards?v
```

| Index  | Shard | Pri/Rep | State     | Docs | Store | Dataset | IP         | Node        |
|--------|-------|---------|-----------|------|-------|---------|------------|-------------|
| fruits | 0     | p       | STARTED   | 0    | 249b  | 249b    | 172.19.0.2 | 42aa3631c480 |
| fruits | 0     | r       | UNASSIGNED|      |       |         |            |             |

- Get the Cluster Health (🟡 Now)

```json
{
  "cluster_name": "docker-cluster",
  "status": "yellow",
  "timed_out": false,
  "number_of_nodes": 1,
  "number_of_data_nodes": 1,
  "active_primary_shards": 32,
  "active_shards": 32,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 1,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 96.96969696969697
}
```

## #Roles in Nodes

 ### Master Node Role
It is responsible For
- **Cluster wide Actions**
- **Creating and deleting Index**
- **Tracking which Nodes are part of clusters**
- **Deciding which shards to allocate to which nodes**


*Config For Master Role*

- `Node.master = true` - It will convert Node to Master Node
- `Node.data = false` - No Data will be saved on Master Node

### Data Node Role
- It holds the shards that contains the document you have indexed

*Config For Data Role*

- `Node.master = false` - No Conversion Needed
- `Node.data = true` - Data will be saved on Data Node

### Ingest Role
- Manipualte documents before adding index to them
- like removing useless fields 
- Adding some more Info

*Config For Ingest Role*

- `Node.ingest = true` - Runs the ingest pipeline

### ML Role

*Config For ML Role*

- `Node.ml = true` - ML Jobs will take place
- `xpack.ml.enabled = true` - ML Apis will get enabled

### Coordination Role

- Route queries to other Nodes
- Load Balancer
- Doesn't Do anything itself

*Config For ML Role*

- `Node.master = false` 
- `Node.data = false` 
- `Node.ingest = false` 
- `Node.ml = false` 
- `xpack.ml.enabled = false` 

## #Queries❓

- Delete Index
```
DELETE student
```
- Create Index with custom no. of Shards(Primary) and Replica
```
PUT student
{
    "settings":{
        "number_of_shards":2,
        "number_of_replicas":2,
    }
}
```

| Index   | Shard | Pri/Rep | State      | Docs | Store | Dataset | IP         | Node        |
|---------|-------|---------|------------|------|-------|---------|------------|-------------|
| student | 0     | p       | STARTED    | 0    | 227b  | 227b    | 172.19.0.2 | 42aa3631c480 |
| student | 0     | r       | UNASSIGNED |      |       |         |            |             |
| student | 0     | r       | UNASSIGNED |      |       |         |            |             |
| student | 1     | p       | STARTED    | 0    | 227b  | 227b    | 172.19.0.2 | 42aa3631c480 |
| student | 1     | r       | UNASSIGNED |      |       |         |            |             |
| student | 1     | r       | UNASSIGNED |      |       |         |            |             |


- Create Data in an Index

```
POST student/_doc
{
    "name":"Rahul",
    "age":35
}
```

returns 
```json
{
  "_index": "student",
  "_id": "whu2tpIBLXx4PSZlMBug",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 3,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```
- Create Data in an Index with Custom ID

```
POST student/_doc/43465645745
{
    "name":"Rahul",
    "age":35
}
```
returns 
```json
{
  "_index": "student",
  "_id": "43465645745",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 3,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

- Search in the Index (this will return all the data)

```
GET student/_search
{
    "query":{
        "match_all":{}
    }
}
```

- Update an existing data in the index (Custom Id)
```
POST /student/_update/43465645745
{
    "doc":{
        "age":5,
        "lastname":"tiwari"
    }
} 
``` 

**NOTE📌📌 : The Schema here is flexible, can add or remove fields**

- Update with a custom script
```
POST /student/_update/43465645745
{
    "script":{
        "source":"ctx._source.age=50"
    }
}

```
- Update with a custom script with Params provided
```
POST /student/_update/43465645745
{
    "script":{
        "source":"ctx._source.age=params.new_age",
        "params":{
            "new_age":5
        }
    }
}
```
- Update with a Multiline(""") - custom script
- It will descrease the age until it will become 0
- And then it will stop as "**noop**"
```
POST /student/_update/43465645745
{
    "script":{
        "source":"""
        if(ctx._source.age <0){
            ctx.op ="noop";
        }else{
            ctx._source.age--;
        }
        """
    }
}
```

- Another same script which will delete the record when age will be equal to 0
```
POST /student/_update/43465645745
{
    "script":{
        "source":"""
        if(ctx._source.age <0){
            ctx.op ="delete";
        }else{
            ctx._source.age--;
        }
        """
    }
}
```

- **UPSERT** (handle case for not found) 
- By default Update the data acc. to source
```
POST /student/_update/43465645745
{
    "script":{
        "source":"ctx._source.age++"
    },
    "upsert":{
        "name":"Billu",
        "age":46
    }
}
```


- **Replace** the Data with Custom Id

```
PUT /student/_update/43465645745
{
        "age":5,
        "lastname":"tiwari"
}
```
