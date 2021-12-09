# scylla-DB
Repo has some important points about scyllaDB, frequently used commands and connectivity through spring boot.

### Some important points of scylla DB


### Frequently used scylla DB command
#### Create keyspace
CREATE KEYSPACE test_keyspace WITH replication = {'class': 'SimpleStrategy', 'replication_factor' : 3};<br>

CREATE KEYSPACE test_keyspace WITH replication = {'class': 'NetworkTopologyStrategy', 'replication_factor' : 3}

#### Create table
CREATE TABLE user_details (user_id text, name text, email text, mobile_no text, dob text, created_at bigint, created_by text, updated_at bigint, updated_by text, PRIMARY KEY (user_id) WITH cdc={'enabled':'true', 'delta':'full'};

#### Create global index
CREATE INDEX user_by_mobile ON user_details (mobile_no);
