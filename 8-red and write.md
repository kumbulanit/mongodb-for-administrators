### **Practical Working Example: Read and Write Scalability, Replication, Durability, and Handling Failures**

To showcase **Read and Write Scalability**, **Replication and Durability**, **Master-Slave Replication**, **Replica Sets**, **Using Write Concern for Durability**, and **Handling Replication Failures**, we’ll use a **Library Database** as a practical example. 

---

### **1. Setup the Library Database**

#### **a. Start MongoDB**
Ensure MongoDB is running on your local machine or a remote server.

#### **b. Create the Library Database**
1. Open the MongoDB Shell (`mongosh`).
2. Create a `library` database and insert sample data:
   ```javascript
   use library;

   db.books.insertMany([
     { title: "The Great Gatsby", author: "F. Scott Fitzgerald", copies: 5 },
     { title: "1984", author: "George Orwell", copies: 3 },
     { title: "To Kill a Mockingbird", author: "Harper Lee", copies: 7 }
   ]);

   db.members.insertMany([
     { name: "Alice", age: 25, borrowedBooks: [] },
     { name: "Bob", age: 30, borrowedBooks: [] },
     { name: "Charlie", age: 22, borrowedBooks: [] }
   ]);
   ```

---

### **2. Read and Write Scalability**

#### **a. Horizontal Scaling with Sharding**
1. Enable sharding for the `library` database:
   ```javascript
   sh.enableSharding("library");
   ```
2. Shard the `books` collection using a hashed shard key:
   ```javascript
   sh.shardCollection("library.books", { _id: "hashed" });
   ```

#### **b. Distribute Reads and Writes**
- Use secondary nodes for read scaling:
  ```javascript
  db.books.find().readPref("secondary");
  ```
- Write operations are automatically routed to the primary node.

---

### **3. Replication and Durability**

#### **a. Set Up a Replica Set**
1. Start three MongoDB instances as part of a replica set:
   ```bash
   mongod --replSet myReplicaSet --port 27017 --dbpath /data/rs1
   mongod --replSet myReplicaSet --port 27018 --dbpath /data/rs2
   mongod --replSet myReplicaSet --port 27019 --dbpath /data/rs3
   ```
2. Initialize the replica set in the MongoDB Shell:
   ```javascript
   rs.initiate({
     _id: "myReplicaSet",
     members: [
       { _id: 0, host: "localhost:27017" },
       { _id: 1, host: "localhost:27018" },
       { _id: 2, host: "localhost:27019" }
     ]
   });
   ```

#### **b. Verify Replication**
1. Insert a document into the `books` collection:
   ```javascript
   db.books.insert({ title: "Moby Dick", author: "Herman Melville", copies: 2 });
   ```
2. Check if the document is replicated to secondary nodes:
   ```javascript
   rs.slaveOk(); // Allow reads from secondaries
   db.books.find();
   ```

---

### **4. Master-Slave Replication (Legacy)**

#### **a. Set Up Master-Slave Replication**
1. Start two MongoDB instances:
   ```bash
   mongod --master --port 27020 --dbpath /data/master
   mongod --slave --source localhost:27020 --port 27021 --dbpath /data/slave
   ```
2. Insert data into the master:
   ```javascript
   use library;
   db.books.insert({ title: "Pride and Prejudice", author: "Jane Austen", copies: 4 });
   ```
3. Verify replication on the slave:
   ```javascript
   rs.slaveOk();
   db.books.find();
   ```

---

### **5. Using Write Concern for Durability**

#### **a. Write Concern Levels**
1. **Acknowledged** (`w: 1`):
   ```javascript
   db.books.insert(
     { title: "The Catcher in the Rye", author: "J.D. Salinger", copies: 6 },
     { writeConcern: { w: 1 } }
   );
   ```
2. **Majority** (`w: majority`):
   ```javascript
   db.books.insert(
     { title: "Brave New World", author: "Aldous Huxley", copies: 8 },
     { writeConcern: { w: "majority" } }
   );
   ```
3. **Journaled** (`j: true`):
   ```javascript
   db.books.insert(
     { title: "The Hobbit", author: "J.R.R. Tolkien", copies: 10 },
     { writeConcern: { j: true } }
   );
   ```

#### **b. Verify Durability**
- Check the logs or use `rs.status()` to confirm that writes are acknowledged by the required number of nodes.

---

### **6. Handling Replication Failures**

#### **a. Simulate a Failure**
1. Stop the primary node:
   ```bash
   mongod --shutdown --port 27017
   ```
2. Observe the replica set election process:
   ```javascript
   rs.status();
   ```

#### **b. Resolve the Failure**
1. Restart the stopped node:
   ```bash
   mongod --replSet myReplicaSet --port 27017 --dbpath /data/rs1
   ```
2. Verify that the node rejoins the replica set:
   ```javascript
   rs.status();
   ```

#### **c. Resync a Lagging Secondary**
1. Remove and re-add a lagging secondary:
   ```javascript
   rs.remove("localhost:27019");
   rs.add("localhost:27019");
   ```

---

### **7. Practical Exercise for Participants**

#### **a. Task 1: Set Up a Replica Set**
- Configure a 3-node replica set and verify replication.

#### **b. Task 2: Use Write Concern**
- Insert documents with different write concern levels and verify durability.

#### **c. Task 3: Simulate a Failure**
- Stop the primary node and observe the election process.

#### **d. Task 4: Resolve Replication Lag**
- Simulate replication lag and resync a secondary node.

#### **e. Task 5: Explore Sharding**
- Shard the `books` collection and distribute reads/writes across shards.

---

### **8. Summary**

- **Read and Write Scalability**: Use sharding and replica sets to distribute reads and writes.
- **Replication and Durability**: Configure replica sets and use write concern to ensure data durability.
- **Master-Slave Replication**: Understand the legacy replication model (not recommended for new deployments).
- **Handling Failures**: Simulate and resolve replication failures to ensure high availability.
