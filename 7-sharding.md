### **Practical Working Example: Auto-Sharding, Setting Up a MongoDB Shard Cluster, and Monitoring**
---

### **1. Setup the Sales Database**

#### **a. Start MongoDB**
Ensure MongoDB is running on your local machine or a remote server.

#### **b. Create the Sales Database**
1. Open the MongoDB Shell (`mongosh`).
2. Create a `sales` database and insert sample data:
   ```javascript
   use sales;

   db.orders.insertMany([
     { orderId: 1, customer: "Alice", amount: 100, region: "North" },
     { orderId: 2, customer: "Bob", amount: 200, region: "South" },
     { orderId: 3, customer: "Charlie", amount: 150, region: "East" }
   ]);

   db.products.insertMany([
     { productId: 101, name: "Laptop", price: 1200, stock: 10 },
     { productId: 102, name: "Smartphone", price: 800, stock: 20 },
     { productId: 103, name: "Tablet", price: 500, stock: 15 }
   ]);
   ```

---

### **2. Auto-Sharding and How Sharding Works**

#### **a. What is Auto-Sharding?**
- Auto-sharding is MongoDB’s mechanism for distributing data across multiple servers (shards) to achieve horizontal scaling.
- MongoDB automatically manages data distribution, query routing, and balancing.

#### **b. Key Concepts**
1. **Shard Key**:
   - A field or set of fields used to distribute data across shards.
   - Example: `region` or `orderId`.

2. **Chunks**:
   - Subsets of data within a sharded collection.
   - MongoDB splits data into chunks based on the shard key.

3. **Balancer**:
   - Automatically redistributes chunks across shards to ensure even data distribution.

---

### **3. Setting Up a MongoDB Shard Cluster**
Create directories for **config servers**, **shards**, and **logs**. For example:

```bash
mkdir -p ~/mongodb/shard1 ~/mongodb/shard2 ~/mongodb/shard3 ~/mongodb/secondary
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
   mongosh --port 27019
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
mongod --shardsvr --replSet shard1ReplSet --port 27023 --dbpath ~/mongodb/shard2 --logpath ~/mongodb/logs/shard2.log --fork
mongod --shardsvr --replSet shard1ReplSet --port 27024 --dbpath ~/mongodb/shard3 --logpath ~/mongodb/logs/shard3.log --fork 
mongod --shardsvr --replSet shard1ReplSet --port 27033 --dbpath ~/mongodb/secondary --logpath ~/mongodb/logs/shard3.log --fork
```

2. Connect to each shard and initiate their replica sets:

   For `shard1`:
   ```bash
   mongosh --port 27022
   ```
   ```javascript
   rs.initiate({
     _id: "shard1ReplSet",
     members: [{ _id: 0, host: "localhost:27022" }]
   })
   ```

   add secondary  shards:
 
   ```javascript
   rs.add("localhost:27023");
   rs.add("localhost:27024");
   rs.add("localhost:27033");
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
   mongos --configdb configReplSet/localhost:27019,localhost:27020,localhost:27021 --logpath ~/mongodb/logs/mongos.log --fork --port 27010 #make sure mongos port is not used
   ```

2. Connect to the **mongos** instance:
   ```bash
   mongosh --port 27010
   ```

---

### **Step 6: Add Shards to the Cluster**

1. Add the shards to the cluster via the `mongos` router:
   ```javascript
   sh.addShard("shard1ReplSet/localhost:27022")
   sh.addShard("shard1ReplSet/localhost:27023")
   sh.addShard("shard1ReplSet/localhost:27024")
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

---

### **4. Monitoring the Sharded Cluster**

#### **a. Check Cluster Status**
1. Use `sh.status()` to view the cluster status:
   ```javascript
   sh.status();
   ```

#### **b. Monitor Chunk Distribution**
1. View chunk distribution across shards:
   ```javascript
   use config;
   db.chunks.find().forEach(printjson);
   ```

#### **c. Monitor Query Routing**
1. Use `mongostat` to monitor query routing and performance:
   ```bash
   mongostat --host localhost:27024
   ```

#### **d. Monitor Balancer Activity**
1. Check balancer status:
   ```javascript
   sh.isBalancerRunning();
   ```
2. View balancer logs:
   ```javascript
   use config;
   db.changelog.find({ what: "balancer.round" }).forEach(printjson);
   ```

---

### **5. Practical Exercise for Participants**

#### **a. Task 1: Set Up a Sharded Cluster**
- Configure config servers, shards, and a `mongos` instance.
- Add shards to the cluster and enable sharding for the `sales` database.

#### **b. Task 2: Shard a Collection**
- Shard the `orders` collection using the `region` field as the shard key.

#### **c. Task 3: Insert and Query Data**
- Insert data into the `orders` collection and query it through the `mongos` instance.

#### **d. Task 4: Monitor the Cluster**
- Use `sh.status()`, `mongostat`, and balancer logs to monitor the cluster.

#### **e. Task 5: Simulate Data Growth**
- Insert a large number of documents into the `orders` collection and observe how MongoDB distributes data across shards.

---

### **6. Summary**

- **Auto-Sharding**: MongoDB automatically distributes data across shards for horizontal scaling.
- **Shard Cluster Setup**: Configure config servers, shards, and `mongos` instances to create a sharded cluster.
- **Monitoring**: Use tools like `sh.status()`, `mongostat`, and balancer logs to monitor the cluster’s performance and data distribution.
