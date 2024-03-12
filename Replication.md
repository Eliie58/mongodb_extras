# MongoDB Replication

## 1 - Setting up a standalone MongoDB instance

MongoDB standalone instances, are a single instance of MongoDB running as a database. They are quick to setup, but are not recommended for production environments.

### 1.1 - Setup

To create the standalone instance of MongoDB, we will use a docker Ubuntu image.

Let's go ahead and start and container:

```
docker run --name mongo-standalone -it ubuntu
```

This command will create a docker container, running the latest ubuntu image, combined with the `-i` and `-t` flags, to start an interactive shell in the container.

Next setp, we need to install a MongoDB instance, we will follow the steps from the [tutorial](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/):

- Reload local package database:<br>
  ```
  apt-get update
  ```
- Import the public key used by the package management system:<br>
  ```
  apt-get install -y gnupg curl
  curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
  ```
- Create a list file for MongoDB:<br>
  ```
  echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-7.0.list
  ```
- Reload local package database:<br>
  ```
  apt-get update
  ```
- Install the MongoDB packages:<br>
  ```
  apt-get install -y mongodb-org
  ```
- Configure region when prompted.
- Start the mongod instance:<br>
  ```
  mongod --config /etc/mongod.conf
  ```

### 1.2 - Connect to the cluster

From a new terminal window, execute the following command:

```
docker exec -it mongo-standalone mongosh
```

### 1.3 - Configure Security

Let's create a new user. Using mongosh:

1. Switch to the `admin` database
2. Add the `admin` user with the `userAdminAnyDatabase` and `readWriteAnyDatabase` roles:

   ```
   use admin
   db.createUser(
   {
       user: "admin",
       pwd: passwordPrompt(), // or cleartext password
       roles: [
       { role: "userAdminAnyDatabase", db: "admin" },
       { role: "readWriteAnyDatabase", db: "admin" }
       ]
   }
   )
   ```

Now, we have to enable authentication on the database, to do that:

1.  Shutdown the `mongod` instance:

    ```
    db.adminCommand({ shutdown: 1 })
    ```

2.  Add the `security.authorization` configuration file setting:

    Install `nano` using:

        apt-get install -y nano

    To modify the config file:

        nano /etc/mongod.conf

3.  Restart the mongod instance

    ```
    mongod --config /etc/mongod.conf
    ```

4.  To connect to the mongod instance, try :

    ```
    docker exec -it mongo-standalone mongosh --username admin
    ```

### 1.4 - Cleanup

To stop and remove the container, run the following commands:

```
docker kill mongo-standalone
docker rm mongo-standalone
```

## 2 - Setting up a 3 node MongoDB Replica Set

### 2.1 - Setup

First step is to create a new docker network:

```
docker network create mongodbCluster
```

Now, we have to launch 3 different mongo docker containers:

```
docker run -d -p 27017:27017 --name mongo1 --network mongodbCluster mongo:7 mongod --replSet myReplicaSet --bind_ip localhost,mongo1
docker run -d -p 27018:27017 --name mongo2 --network mongodbCluster mongo:7 mongod --replSet myReplicaSet --bind_ip localhost,mongo2
docker run -d -p 27019:27017 --name mongo3 --network mongodbCluster mongo:7 mongod --replSet myReplicaSet --bind_ip localhost,mongo3
```

Next step is to connect to a MongoDB instance, and inistiate the replica set:

```
docker exec -it mongo1 bash
mongosh
```

Now, we initiate the replica set:

```
rs.initiate()
rs.add('mongo2:27017')
rs.add('mongo3:27017')
```

To see the status of the RS:

```
rs.status()
```

## 3 - Setting up a 4 node MongoDB Replica Set

### 3.1 - Setup

To add a 4th node, run the following command from a new terminal:

```
docker run -d -p 27020:27017 --name mongo4 --network mongodbCluster mongo:7 mongod --replSet myReplicaSet --bind_ip localhost,mongo4
```

From mongosh:

```
rs.add('mongo4:27017')
```

### 3.2 - Cleanup

To stop and remove the created containers:

```
docker kill mongo1
docker kill mongo2
docker kill mongo3
docker kill mongo4

docker rm mongo1
docker rm mongo2
docker rm mongo3
docker rm mongo4

docker network rm mongodbCluster
```

## 4 - Setting up a sharded cluster

[Guide](https://github.com/minhhungit/mongodb-cluster-docker-compose/blob/master/readme.md#mongodb-601-sharded-cluster-with-docker-compose)
