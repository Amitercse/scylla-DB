# scylla-DB
Repo has some important points about scyllaDB, frequently used commands and connectivity through spring boot.

### Some important points of scylla DB
#### Advantages of scylla
* Wide column datastore (Data is stored in columns and related columns are grouped together as table or column family)
* High Availability (Scylla prefers availability over consistency)
* High Performance (Millions of requests per second per node)
* Scalability
* Low latency
* Replacement of cassandra

#### Replication strategies
Simple Strategy: It is used for single datacenter or one rack.<br>
NetworkTopologyStrategy: When you have cluster deployed across multiple data centers. Here we can set replication factor for different data centers. For ex- if DC1 has replication factor as 3 and DC2 has 2 respectively then it has replication factor as 5.

#### Consistency level
In scylla consistency level determines how many nodes must acknowledge read and write operations before it is considered as successful. Below are different consistency levels. Coordinator node sends write or read operations to multiple replicas based on replication factor. But it is not necessary that data should be written to all replicas and all replicas should respond to read operation. Once consistency level is matched then request is honoured. 

* Any - Write should be written to at least one replica and read must be received from at least one replica. It provides highest availability and lowest consistency.
* Quorum - Majority of the replicas should respond. It is calculated as (n/2+1). If RF=3 then 2 nodes must respond.
* One - If one replica respond that is enough.
* Local_One - At least one replica in local data center responds.
* Local_Quorum - A quorum of replicas from local data center should respond.
* Each_Quorum (unsupported for reads) - A quorum of replicas in all data centers must be written to.
* All - All replicas should respond. It maintains least availability and highest consistency.

We can tune consistency level per query as well.

#### Scylla Architecture
* Scylla doesn't work on master slave model. Each node is treated as equal.
* Within scylla cluster internode communication is peer to peer hence no single point of failure. 
* For communication outside of cluster, scylla client will communicate with a single server node called coordinator.
* Data will be written to which node will be decided based on partition key using consistent hash function.
* Each node is assigned with a range based. For each partition key has code is computed and then data is placed in the node. Scylla uses murmur hash function to generate hash of partition key which is of 64-bit length in range from -2^63 to 2^63-1.
* Nodes use gossipping protocol to exchange information with each other.
* Scylla uses vnodes concept where each node is divided in multiple vnodes. Before vnode each physical node was having only one range assigned. But after vnode concept, each vnode will have a separate range assigned. It helps in rebuilding the dead node or adding any new node etc.
* Scylla uses snitches to identify to which data center node belongs to, or to identify where requests should be rerouted based on network topology.

#### Data modeling in scylla
* Data modeling in scylla is query based. So first think about application and data and then model your tables.
* Too small or too large partitions should be avoided. Too large partitions can be checked by - system.large_partitions.
* Scylla supports three types of collection - list, set and map. List elements are stored by passing inside bracket, set inside curly braces and map also inside curley braces but in key-value pair. Check below link for detail. But we should store small amount of data only in collections as to read one data of collection we need to scan entire collection.
https://university.scylladb.com/courses/data-modeling/lessons/advanced-data-modeling/topic/common-data-types-and-collections/
* Scylla provides ttl functionality to remove data automatically once ttl is expired. TTL is measured in seconds and is defined at column level. If column is not updated within defined ttl then it is removed.
* TTL can de defined at table creation time or insert command or via update command. But ttl is applied at column level.
* TTL of a column can be retrived using ttl(column_name). Check below doc for more details.
https://docs.scylladb.com/getting-started/time-to-live/

#### UDT in scylla
* Scylla supports UDT and UDT is created using type.
* We should have UDT as frozen to make sure that entire value is updated for any update and no partial update.
https://docs.scylladb.com/getting-started/types/#user-defined-types

#### Materialized views
* Materialized views are global indexes. When materialized view is declared then a new table gets created.
* Reads from materialized view is as efficient as in regular tables but writes are slower because data needs to be committed to regular table as well as to materialzed view and also consistency between them needs to be maintained.
* Materlized views must have all columns of primary key of base table. Also it can have at most one column which is not part of primary key of base table.
* If from base table or materialzed views, we want to search based on column which is not part of primary key then add 'allow filtering' with the query. It will have performance impact. 
* While creating materialized views we can select particular columns as well rather than choosing all columns.

### Frequently used scylla DB command
#### Create keyspace

CREATE KEYSPACE test_keyspace WITH replication = {'class': 'SimpleStrategy', 'replication_factor' : 3};<br>

CREATE KEYSPACE test_keyspace WITH replication = {'class': 'NetworkTopologyStrategy', 'replication_factor' : 3}

#### Create table
CREATE TABLE user_details (user_id text, name text, email text, mobile_no text, dob text, created_at bigint, created_by text, updated_at bigint, updated_by text, PRIMARY KEY (user_id) WITH cdc={'enabled':'true', 'delta':'full'};

#### Create global index
CREATE INDEX user_by_mobile ON user_details (mobile_no);


