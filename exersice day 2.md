### **Step-by-Step MongoDB Exercise: Hospital Database Setup & Optimization**  
**Duration:** ~60 minutes  
**Objective:** Build a hospital database with 5000 patient records, configure MongoDB for security, scalability, and durability, and troubleshoot common issues.  

---

### **1. Install MongoDB & Tools**  
**Topic:** Installation and Basic Setup  
**Steps:**  
1. **Install MongoDB (Ubuntu):**  
   ```bash 
   # Add MongoDB repository 
   sudo apt-get install -y gnupg 
   curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb.gpg 
   echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list 
   sudo apt-get update 
   sudo apt-get install -y mongodb-org 

   # Start MongoDB 
   sudo systemctl start mongod 
   sudo systemctl enable mongod 
   ```  
2. **Install MongoDB Shell (`mongosh`):**  
   ```bash 
   curl -O https://downloads.mongodb.com/compass/mongosh-2.0.1-linux-x64.tgz 
   tar -zxvf mongosh-2.0.1-linux-x64.tgz 
   sudo mv mongosh-2.0.1-linux-x64/bin/mongosh /usr/local/bin/ 
   ```  

**Check:**  
```bash 
mongosh --version  # Verify installation 
sudo systemctl status mongod  # Ensure MongoDB is running 
```

---

### **2. Create the Hospital Database**  
**Topic:** CRUD Operations, Data Modeling  
**Steps:**  
1. **Generate 5000 Patient Records:**  
   ```bash 
   mongosh 
   ```  
   ```javascript 
   // In MongoDB Shell 
   use hospital 

   for (let i=1; i<=5000; i++) { 
     db.patients.insertOne({ 
       patientId: i, 
       name: `Patient ${i}`, 
       age: Math.floor(Math.random() * 80) + 18, 
       ailment: ["Diabetes", "Hypertension", "Asthma", "Arthritis"][Math.floor(Math.random() * 4)], 
       medications: ["Metformin", "Lisinopril", "Albuterol", "Ibuprofen"][Math.floor(Math.random() * 4)], 
       procedures: ["Blood Test", "X-Ray", "MRI", "Surgery"][Math.floor(Math.random() * 4)], 
       referrals: ["Cardiologist", "Orthopedist", "Pulmonologist", "Endocrinologist"][Math.floor(Math.random() * 4)], 
       appointments: [ 
         { date: new Date(), specialist: "Cardiologist" }, 
         { date: new Date(), specialist: "Orthopedist" } 
       ] 
     }) 
   } 
   ```  

**Check:**  
```javascript 
db.patients.countDocuments()  // Should return 5000 
```

---

### **3. Configure Security**  
**Topic:** Authentication, Secure Deployment  
**Steps:**  
1. **Enable Authentication:**  
   ```bash 
   sudo nano /etc/mongod.conf 
   ```  
   Add:  
   ```yaml 
   security: 
     authorization: enabled 
   ```  
   Restart MongoDB:  
   ```bash 
   sudo systemctl restart mongod 
   ```  

2. **Create Admin User:**  
   ```javascript 
   use admin 
   db.createUser({ 
     user: "admin", 
     pwd: "admin123", 
     roles: ["root"] 
   }) 
   ```  

3. **Reconnect with Authentication:**  
   ```bash 
   mongosh -u admin -p admin123 --authenticationDatabase admin 
   ```  

**Troubleshooting:**  
- If MongoDB fails to start, check logs at `/var/log/mongodb/mongod.log`.  
- Fix syntax errors in `/etc/mongod.conf`.  

---

### **4. Indexing & Query Optimization**  
**Topic:** Indexing, Query Profiler  
**Steps:**  
1. **Create an Index on `patientId`:**  
   ```javascript 
   db.patients.createIndex({ patientId: 1 }) 
   ```  

2. **Run a Query Without Index:**  
   ```javascript 
   db.patients.find({ age: { $gt: 60 } }).explain("executionStats")  // Note "COLLSCAN" 
   ```  

3. **Create Index on `age`:**  
   ```javascript 
   db.patients.createIndex({ age: 1 }) 
   ```  

4. **Re-run Query with Index:**  
   ```javascript 
   db.patients.find({ age: { $gt: 60 } }).explain("executionStats")  // Note "IXSCAN" 
   ```  

**Check:**  
- Compare `totalDocsExamined` and `executionTimeMillis` before/after indexing.  

---

### **5. Replication & Write Concern**  
**Topic:** Replica Sets, Durability  
**Steps:**  
1. **Initialize a Replica Set:**  
   Stop MongoDB and restart with replication:  
   ```bash 
   mongod --replSet hospitalRepl --port 27017 --dbpath /data/db --logpath /var/log/mongodb/mongod.log --fork 
   ```  
   In `mongosh`:  
   ```javascript 
   rs.initiate() 
   rs.add("localhost:27018")  # Add second node (if available) 
   ```  

2. **Insert with Write Concern:**  
   ```javascript 
   db.patients.insertOne( 
     { name: "Emergency Patient", age: 35, ailment: "Fracture" }, 
     { writeConcern: { w: "majority", wtimeout: 5000 } } 
   ) 
   ```  

**Troubleshooting:**  
- If replication fails, check node connectivity with `rs.status()`.  

---

### **6. Backup & Restore**  
**Topic:** Backup Strategies, `mongodump`  
**Steps:**  
1. **Backup with `mongodump`:**  
   ```bash 
   mongodump -u admin -p admin123 --authenticationDatabase admin --db hospital --out /backup 
   ```  

2. **Simulate Data Loss:**  
   ```javascript 
   db.patients.deleteMany({}) 
   ```  

3. **Restore with `mongorestore`:**  
   ```bash 
   mongorestore -u admin -p admin123 --authenticationDatabase admin --db hospital /backup/hospital 
   ```  

**Check:**  
```javascript 
db.patients.countDocuments()  // Should return 5001 
```

---

### **7. Troubleshooting Scenarios (Final Step)**  
**Common Issues & Solutions:**  
1. **Connection Refused:**  
   - **Fix:** Ensure MongoDB is running (`sudo systemctl start mongod`).  

2. **Authentication Failed:**  
   - **Fix:** Verify username/password and `authenticationDatabase`.  

3. **Replication Lag:**  
   - **Fix:** Check network latency or resync secondary nodes with `rs.syncFrom()`.  

4. **Slow Queries:**  
   - **Fix:** Create appropriate indexes or optimize queries.  

---

**Exercise Complete!**  
