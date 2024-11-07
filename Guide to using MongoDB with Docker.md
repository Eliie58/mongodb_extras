# MongoDB with Docker Guide

## Table of Contents

1. Standalone MongoDB Node

   - Running a Standalone MongoDB Node
   - Connecting, Reading, and Writing Data

2. MongoDB Replica Set
   - Setting Up a 3-Node Replica Set

- Initializing the Replica Set
- Testing Replication
- Testing Failover and Syncing Scenarios

## Prerequisites

- Ensure you have Docker and Docker Compose installed.
- Install MongoDB shell (mongosh) to interact with the MongoDB containers.

## 1. Standalone MongoDB Node

### Running a Standalone MongoDB Node

Run a standalone MongoDB instance on port 27000 in a Docker container.

1. Open your terminal and run the following command to start the MongoDB container:

```bash
docker run -d --name mongodb-standalone -p 27000:27017 mongo
```

This command:

- Runs MongoDB in a detached state (-d).
- Names the container mongodb-standalone.
- Exposes MongoDB on port 27000 on the host.

### Connecting, Reading, and Writing Data

1. Connect to the MongoDB instance using mongosh:

```bash
mongosh "mongodb://localhost:27000"
```

2. Once connected, create a database, insert data, and query it.

```javascript
// Switch to a new database
use testDB

// Insert sample data
db.users.insertOne({ name: "Alice", age: 25 })

// Query the data
db.users.find()
```

## 2. MongoDB Replica Set

This section covers setting up a 3-node MongoDB replica set and testing various replication and failover scenarios.

### Setting Up a 3-Node Replica Set

1. Create a custom Docker network to allow the MongoDB containers to communicate.

```bash
docker network create mongo-replica-network
```

2. Start three MongoDB containers, each representing a node in the replica set, with unique ports.

```bash
# Start MongoDB node 1

docker run -d --name mongo1 --hostname mongo1 --net mongo-replica-network -p 27001:27017 mongo --replSet "rs0"

# Start MongoDB node 2

docker run -d --name mongo2 --hostname mongo2 --net mongo-replica-network -p 27002:27017 mongo --replSet "rs0"

# Start MongoDB node 3

docker run -d --name mongo3 --hostname mongo3 --net mongo-replica-network -p 27003:27017 mongo --replSet "rs0"
```

### Initializing the Replica Set

1. Connect to one of the nodes to initialize the replica set configuration.

```bash
mongosh "mongodb://localhost:27001"
```

2. Once connected, run the following commands to initiate the replica set:

```javascript
rs.initiate({
    _id: "rs0",
    members: [
        { _id: 0, host: "mongo1:27017" },
        { _id: 1, host: "mongo2:27017" },
        { _id: 2, host: "mongo3:27017" }
    ]
})
```

3.Verify the status of the replica set:

```javascript
rs.status();
```

### Testing Replication

1. Connect to the primary node

```bash
mongosh "mongodb://localhost:27001"
```

2. Insert some data into the primary:

```javascript
db.users.insertOne({ name: "Bob", age: 30 })
```

3. Connect to a secondary node to verify data replication.

```bash
mongosh "mongodb://localhost:27002"
```

4. Read data from the secondary node:

```javascript
db.users.find()
```

### Testing Failover and Syncing Scenarios

1. Pausing and Resuming a Secondary Node

   - Pause a secondary node to simulate a temporary outage:

   ```bash
   docker pause mongo2
   ```

   - Insert additional data into the primary:

   ```javascript
   db.users.insertOne({ name: "Charlie", age: 28 });
   ```

   - Resume the paused node:

   ```bash
   docker unpause mongo2
   ```

   - Connect back to the resumed node and verify it catches up with the latest data:

   ```javascript
   use userDB
   db.users.find()
   ```

2. Stopping the Primary Node and Testing Failover

   - Stop the current primary node:

   ```bash
   docker stop mongo1
   ```

   - MongoDB will elect a new primary. Connect to one of the other nodes and check the replica set status to see the new primary:

   ```javascript
   rs.status();
   ```

   - Insert new data into the new primary to confirm it’s functional:

   ```javascript
   db.users.insertOne({ name: "David", age: 32 });
   ```

   - Restart the original primary node:

   ```bash
   docker start mongo1
   ```

   - Verify that the restarted node re-syncs data from the new primary.

### Connecting to All Replica Set Nodes

To connect to all nodes in the replica set from mongosh:

```bash
mongosh "mongodb://localhost:27001,localhost:27002,localhost:27003/?replicaSet=rs0"
```

This connection string allows mongosh to automatically detect the primary and direct writes accordingly.
