

To showcase **Shard/Chunk Migration**, **Backup and Restore Plans**, **Filesystem-Based Strategies**, and tools like **mongodump/mongorestore**, **rsync**, and **mongoimport/mongoexport**, we’ll use a **Library Database** as a practical example. This example will allow participants to simulate chunk migration, perform backups and restores, and use various tools for data management.

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

### **2. Shard/Chunk Migration**

#### **a. Set Up a Sharded Cluster**
1. Start config servers, shards, and a `mongos` instance (refer to the previous example for setup steps).
2. Enable sharding for the `library` database:
   ```javascript
   sh.enableSharding("library");
   ```
3. Shard the `books` collection using the `title` field as the shard key:
   ```javascript
   sh.shardCollection("library.books", { title: 1 });
   ```

#### **b. Simulate Chunk Migration**
1. Insert a large number of documents into the `books` collection to create multiple chunks:
   ```javascript
   for (let i = 0; i < 1000; i++) {
     db.books.insert({ title: `Book ${i}`, author: `Author ${i}`, copies: 1 });
   }
   ```
2. Check chunk distribution:
   ```javascript
   use config;
   db.chunks.find({ ns: "library.books" }).forEach(printjson);
   ```
3. Manually move a chunk to another shard:
   ```javascript
   sh.moveChunk("library.books", { title: "Book 500" }, "shard1ReplSet");
   ```

---

### **3. Backup and Restore Plans**

#### **a. Filesystem-Based Backup**
1. Stop the MongoDB instance or lock the database:
   ```javascript
   db.fsyncLock();
   ```
2. Take a snapshot of the data files using tools like **LVM** or **rsync**:
   ```bash
   rsync -av /data/db /backup
   ```
3. Unlock the database:
   ```javascript
   db.fsyncUnlock();
   ```

#### **b. Filesystem-Based Restore**
1. Stop the MongoDB instance.
2. Replace the data files with the snapshot:
   ```bash
   rsync -av /backup /data/db
   ```
3. Start the MongoDB instance.

---

### **4. mongodump / mongorestore**

#### **a. Backup with mongodump**
1. Export the `library` database to a BSON file:
   ```bash
   mongodump --host localhost --port 27017 --out /backup
   ```

#### **b. Restore with mongorestore**
1. Restore the `library` database from the BSON file:
   ```bash
   mongorestore --host localhost --port 27017 /backup
   ```

---

### **5. rsync for Backup**

#### **a. Backup Data Files**
1. Use `rsync` to copy MongoDB data files to a backup location:
   ```bash
   rsync -av /data/db /backup
   ```

#### **b. Restore Data Files**
1. Use `rsync` to restore data files from the backup location:
   ```bash
   rsync -av /backup /data/db
   ```

---

### **6. mongoimport / mongoexport**

#### **a. Export Data with mongoexport**
1. Export the `books` collection to a JSON file:
   ```bash
   mongoexport --host localhost --port 27017 --db library --collection books --out /backup/books.json
   ```

#### **b. Import Data with mongoimport**
1. Import the `books` collection from the JSON file:
   ```bash
   mongoimport --host localhost --port 27017 --db library --collection books --file /backup/books.json
   ```

---

### **7. Practical Exercise for Participants**

#### **a. Task 1: Simulate Chunk Migration**
- Set up a sharded cluster and simulate chunk migration by inserting a large number of documents and moving chunks manually.

#### **b. Task 2: Perform a Filesystem-Based Backup**
- Use `rsync` to back up MongoDB data files and restore them.

#### **c. Task 3: Use mongodump/mongorestore**
- Export the `library` database using `mongodump` and restore it using `mongorestore`.

#### **d. Task 4: Use mongoimport/mongoexport**
- Export the `books` collection to a JSON file using `mongoexport` and import it using `mongoimport`.

#### **e. Task 5: Monitor Backup and Restore**
- Verify the integrity of the data after performing backups and restores.

---

### **8. Summary**

- **Shard/Chunk Migration**: Simulate chunk migration in a sharded cluster to understand data distribution.
- **Backup and Restore Plans**: Use filesystem-based strategies, `mongodump/mongorestore`, and `rsync` for backups and restores.
- **Filesystem-Based Strategies**: Take snapshots of MongoDB data files for large datasets.
- **mongodump/mongorestore**: Export and import data in BSON format for logical backups.
- **mongoimport/mongoexport**: Export and import data in JSON format for specific collections.

