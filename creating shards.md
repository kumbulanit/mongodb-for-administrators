Setting up a **sharded cluster** in MongoDB on your localhost involves configuring the following components:

1. **Config servers (csrs)**: Store metadata for the sharded cluster.
2. **Shard servers**: Hold actual data, divided into chunks.
3. **Mongos router**: Routes client queries to the appropriate shard.



---

### **Step 1: Download and Install MongoDB**

1. **Download MongoDB** from the [official website](https://www.mongodb.com/try/download/community) if it’s not already installed.
2. Extract and add `mongod` and `mongo` to your PATH.

---

### **Step 2: Prepare Directories for Data Storage**

Create directories for **config servers**, **shards**, and **logs**. For example:

```bash
mkdir -p ~/mongodb/shard1 ~/mongodb/shard2 ~/mongodb/shard3
mkdir -p ~/mongodb/config1 ~/mongodb/config2 ~/mongodb/config3
mkdir -p ~/mongodb/logs
```

---

### **Step 3: Start Config Server Replica Set**

Config servers are critical for managing metadata. Start a **config server replica set**:

1. Start 3 config server instances (one per terminal):

```bash
mongod --configsvr --replSet configReplSet --port 27019 --dbpath ~/mongodb/config1 --logpath ~/mongodb/logs/config1.log --fork
mongod --configsvr --replSet configReplSet --port 27020 --dbpath ~/mongodb/config2 --logpath ~/mongodb/logs/config2.log --fork
mongod --configsvr --replSet configReplSet --port 27021 --dbpath ~/mongodb/config3 --logpath ~/mongodb/logs/config3.log --fork
```

2. Connect to one of the config servers:
   ```bash
   mongo --port 27019
   ```

3. Initiate the replica set:
   ```javascript
   rs.initiate({
     _id: "configReplSet",
     configsvr: true,
     members: [
       { _id: 0, host: "localhost:27019" },
       { _id: 1, host: "localhost:27020" },
       { _id: 2, host: "localhost:27021" }
     ]
   })
   ```

4. Verify the status:
   ```javascript
   rs.status()
   ```

---

### **Step 4: Start Shard Servers**

Start multiple shards. Each shard can also be a replica set for high availability. Here, we'll create 3 shards (one per terminal):

1. Start each shard instance:

```bash
mongod --shardsvr --replSet shard1ReplSet --port 27022 --dbpath ~/mongodb/shard1 --logpath ~/mongodb/logs/shard1.log --fork
mongod --shardsvr --replSet shard2ReplSet --port 27023 --dbpath ~/mongodb/shard2 --logpath ~/mongodb/logs/shard2.log --fork
mongod --shardsvr --replSet shard3ReplSet --port 27024 --dbpath ~/mongodb/shard3 --logpath ~/mongodb/logs/shard3.log --fork
```

2. Connect to each shard and initiate their replica sets:

   For `shard1`:
   ```bash
   mongo --port 27022
   ```
   ```javascript
   rs.initiate({
     _id: "shard1ReplSet",
     members: [{ _id: 0, host: "localhost:27022" }]
   })
   ```

   For `shard2`:
   ```bash
   mongo --port 27023
   ```
   ```javascript
   rs.initiate({
     _id: "shard2ReplSet",
     members: [{ _id: 0, host: "localhost:27023" }]
   })
   ```

   For `shard3`:
   ```bash
   mongo --port 27024
   ```
   ```javascript
   rs.initiate({
     _id: "shard3ReplSet",
     members: [{ _id: 0, host: "localhost:27024" }]
   })
   ```

3. Verify each replica set’s status:
   ```javascript
   rs.status()
   ```

---

### **Step 5: Start the Mongos Router**

The **mongos** process routes queries to the appropriate shard.

1. Start the **mongos** instance:
   ```bash
   mongos --configdb configReplSet/localhost:27019,localhost:27020,localhost:27021 --logpath ~/mongodb/logs/mongos.log --fork --port 27017
   ```

2. Connect to the **mongos** instance:
   ```bash
   mongo --port 27017
   ```

---

### **Step 6: Add Shards to the Cluster**

1. Add the shards to the cluster via the `mongos` router:
   ```javascript
   sh.addShard("shard1ReplSet/localhost:27022")
   sh.addShard("shard2ReplSet/localhost:27023")
   sh.addShard("shard3ReplSet/localhost:27024")
   ```

2. Verify the shards are added:
   ```javascript
   sh.status()
   ```

---

### **Step 7: Enable Sharding for a Database**

1. Enable sharding for a specific database (e.g., `myDatabase`):
   ```javascript
   sh.enableSharding("myDatabase")
   ```

2. Create a sharded collection (e.g., `myCollection`) with a shard key:
   ```javascript
   sh.shardCollection("myDatabase.myCollection", { keyField: 1 })
   ```

---

### **Step 8: Verify the Cluster**

1. Insert data into the sharded collection:
   ```javascript
   use myDatabase
   db.myCollection.insertMany([
     { keyField: 1, value: "A" },
     { keyField: 2, value: "B" },
     { keyField: 3, value: "C" }
   ])
   ```

2. Check the distribution of chunks:
   ```javascript
   sh.status()
   ```

---

### **Summary**

You’ve now set up a sharded MongoDB cluster with:
- 3 Config Servers.
- 3 Shards (each can be a replica set).
- 1 Mongos Router.

To test, connect to the `mongos` instance (`localhost:27017`) and interact with the cluster.