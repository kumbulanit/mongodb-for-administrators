Below is an **extended and progressive practical exercise** designed to take participants from **simple to advanced MongoDB concepts**. This exercise uses the `university` database and data to cover a wide range of topics, starting with basic operations and gradually moving to advanced configurations, security, and monitoring.

---

## **Exercise: Managing a University Database**

---

### **Part 1: Basic Operations**
#### **Step 1: Set Up the Environment**
1. **Install MongoDB**:
   - Follow the installation steps for your operating system (e.g., Ubuntu, Windows, macOS).
   - Start the MongoDB server:
     ```bash
     mongod --dbpath /data/db
     ```

2. **Access the MongoDB Shell**:
   - Open a new terminal and start the MongoDB shell:
     ```bash
     mongosh
     ```

3. **Switch to the `university` Database**:
   - Create and switch to the `university` database:
     ```javascript
     use university
     ```

---

#### **Step 2: Working with Documents and Data Types**
1. **Insert Documents**:
   - Insert student documents with different data types:
     ```javascript
     db.students.insertMany([
       {
         name: "Alice",
         age: 22,
         email: "alice@university.edu",
         course: "Computer Science",
         subjects: ["Algorithms", "Data Structures"],
         year: 3,
         gpa: 3.8
       },
       {
         name: "Bob",
         age: 21,
         email: "bob@university.edu",
         course: "Mathematics",
         subjects: ["Calculus", "Linear Algebra"],
         year: 2,
         gpa: 3.9
       },
       {
         name: "Charlie",
         age: 23,
         email: "charlie@university.edu",
         course: "Physics",
         subjects: ["Quantum Mechanics", "Thermodynamics"],
         year: 4,
         gpa: 3.7
       }
     ]);
     ```

2. **Query Documents**:
   - Find all students in the "Computer Science" course:
     ```javascript
     db.students.find({ course: "Computer Science" });
     ```

   - Find students with a GPA greater than 3.5:
     ```javascript
     db.students.find({ gpa: { $gt: 3.5 } });
     ```

---

#### **Step 3: CRUD Operations**
1. **Update Documents**:
   - Update Alice's GPA to 4.0:
     ```javascript
     db.students.updateOne({ name: "Alice" }, { $set: { gpa: 4.0 } });
     ```

2. **Delete Documents**:
   - Delete Bob's record:
     ```javascript
     db.students.deleteOne({ name: "Bob" });
     ```

---

### **Part 2: Intermediate Operations**
#### **Step 4: System Commands**
1. **Show Databases**:
   ```javascript
   show dbs;
   ```

2. **Show Collections**:
   ```javascript
   show collections;
   ```

3. **Get Collection Statistics**:
   ```javascript
   db.students.stats();
   ```

---

#### **Step 5: Indexing**
1. **Create an Index**:
   - Create an index on the `course` field:
     ```javascript
     db.students.createIndex({ course: 1 });
     ```

2. **Explain Query Execution**:
   - Use `explain()` to analyze query performance:
     ```javascript
     db.students.find({ course: "Computer Science" }).explain("executionStats");
     ```

---

#### **Step 6: Aggregation**
1. **Calculate Average GPA by Course**:
   - Use the aggregation framework to calculate the average GPA for each course:
     ```javascript
     db.students.aggregate([
       { $group: { _id: "$course", averageGPA: { $avg: "$gpa" } } }
     ]);
     ```

2. **Count Students by Year**:
   - Count the number of students in each year:
     ```javascript
     db.students.aggregate([
       { $group: { _id: "$year", count: { $sum: 1 } } }
     ]);
     ```

---

### **Part 3: Advanced Operations**
#### **Step 7: Single-Server Configuration and Deployment**
1. **Create a Configuration File**:
   - Create a `mongod.conf` file:
     ```yaml
     storage:
       dbPath: /data/db
     systemLog:
       path: /var/log/mongodb/mongod.log
     net:
       bindIp: 127.0.0.1
       port: 27017
     security:
       authorization: enabled
     ```

2. **Start MongoDB with the Configuration File**:
   ```bash
   mongod --config /path/to/mongod.conf
   ```

---

#### **Step 8: Data Files and Allocation**
1. **Check Data File Location**:
   - Verify the `dbPath` in the `mongod.conf` file:
     ```yaml
     storage:
       dbPath: /data/db
     ```

2. **View Data Files**:
   - Navigate to the `dbPath` directory and list the files:
     ```bash
     ls /data/db
     ```

---

#### **Step 9: Log Files**
1. **View Log File**:
   - Check the log file specified in the `mongod.conf` file:
     ```bash
     tail -f /var/log/mongodb/mongod.log
     ```

---

#### **Step 10: Hardware and File-System Recommendations**
1. **Use SSDs**:
   - Ensure MongoDB data files are stored on SSDs for better performance.

2. **Enable Journaling**:
   - Journaling is enabled by default in the `mongod.conf` file:
     ```yaml
     storage:
       journal:
         enabled: true
     ```

---

### **Part 4: Security**
#### **Step 11: Built-in Authentication**
1. **Enable Authentication**:
   - Create an admin user:
     ```javascript
     use admin;
     db.createUser({
       user: "admin",
       pwd: "admin123",
       roles: [{ role: "userAdminAnyDatabase", db: "admin" }]
     });
     ```

2. **Restart MongoDB with Authentication**:
   - Update the `mongod.conf` file:
     ```yaml
     security:
       authorization: enabled
     ```

---

#### **Step 12: Recommendations for Secure Deployment**
1. **Encrypt Data in Transit**:
   - Use TLS/SSL to encrypt communication between clients and MongoDB.
   - Example configuration:
     ```yaml
     net:
       ssl:
         mode: requireSSL
         PEMKeyFile: /etc/ssl/mongodb.pem
     ```

2. **Encrypt Data at Rest**:
   - Use MongoDB's **Encrypted Storage Engine** or file-system encryption to protect data at rest.
   - Example:
     ```yaml
     security:
       enableEncryption: true
       encryptionKeyFile: /etc/mongodb/keyfile
     ```

---

### **Part 5: Monitoring MongoDB**
#### **Step 13: Use `mongostat`**
1. **Monitor Real-Time Statistics**:
   - Run `mongostat` to monitor real-time statistics:
     ```bash
     mongostat
     ```

---

#### **Step 14: Analyze Memory and IO Performance**
1. **Check Memory Usage**:
   - Use `db.serverStatus()` to analyze memory usage:
     ```javascript
     db.serverStatus().wiredTiger.cache;
     ```

2. **Check IO Performance**:
   - Use operating system tools like `iostat` (Linux) or Performance Monitor (Windows).

---

#### **Step 15: Integration with Monitoring Tools**
1. **Install Munin**:
   - Follow the installation instructions for Munin.
   - Configure Munin to monitor MongoDB metrics.

2. **View Metrics**:
   - Access the Munin web interface to view MongoDB performance metrics.

---

#### **Step 16: MongoDB's Web Console**
1. **Install MongoDB Ops Manager**:
   - Follow the installation steps for MongoDB Ops Manager.

2. **Monitor MongoDB**:
   - Use the Ops Manager web interface to monitor your MongoDB deployment.

---
