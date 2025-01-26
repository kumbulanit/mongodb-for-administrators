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

#### **a. Start Config Servers**
1. Start three MongoDB instances as config servers:
   ```bash
   mongod --configsvr --replSet configReplSet --port 27019 --dbpath /data/config1
   mongod --configsvr --replSet configReplSet --port 27020 --dbpath /data/config2
   mongod --configsvr --replSet configReplSet --port 27021 --dbpath /data/config3
   ```
2. Initialize the config server replica set:
   ```javascript
   rs.initiate({
     _id: "configReplSet",
     configsvr: true,
     members: [
       { _id: 0, host: "localhost:27019" },
       { _id: 1, host: "localhost:27020" },
       { _id: 2, host: "localhost:27021" }
     ]
   });
   ```

#### **b. Start Shards**
1. Start two MongoDB instances as shards:
   ```bash
   mongod --shardsvr --replSet shard1ReplSet --port 27022 --dbpath /data/shard1
   mongod --shardsvr --replSet shard1ReplSet --port 27023 --dbpath /data/shard2
   ```
2. Initialize the shard replica sets:
   ```javascript
   rs.initiate({
     _id: "shard1ReplSet",
     members: [
       { _id: 0, host: "localhost:27022" },
       { _id: 1, host: "localhost:27023" }
     ]
   });
   ```

#### **c. Start Query Routers (Mongos)**
1. Start a `mongos` instance:
   ```bash
   mongos --configdb configReplSet/localhost:27019,localhost:27020,localhost:27021 --port 27024
   ```

#### **d. Add Shards to the Cluster**
1. Connect to the `mongos` instance:
   ```bash
   mongosh --port 27024
   ```
2. Add shards to the cluster:
   ```javascript
   sh.addShard("shard1ReplSet/localhost:27022,localhost:27023");
   ```

#### **e. Enable Sharding for the Database**
1. Enable sharding for the `sales` database:
   ```javascript
   sh.enableSharding("sales");
   ```

#### **f. Shard a Collection**
1. Shard the `orders` collection using the `region` field as the shard key:
   ```javascript
   sh.shardCollection("sales.orders", { region: 1 });
   ```

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
