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



To test, connect to the `mongos` instance (`localhost:27017`) and interact with the cluster.

### **to remove shards and also configsvr****
To remove the shards (`shard1ReplSet`, `shard2ReplSet`, and `shard3ReplSet`) and their replica set members (including the secondary on port `27033`), follow these steps. This assumes you want to **completely remove the shards and their data** (development environment only).

---

### **Step 1: Remove Shards from the Cluster via `mongos`**
1. **Connect to the `mongos` router**:
   ```bash
   mongosh "<mongos-host:port>"
   ```

2. **Remove each shard** (repeat for all shards):
   ```javascript
   use admin

   // Remove shard1ReplSet
   db.adminCommand({ removeShard: "shard1ReplSet" });

   // Remove shard2ReplSet
   db.adminCommand({ removeShard: "shard2ReplSet" });

   // Remove shard3ReplSet
   db.adminCommand({ removeShard: "shard3ReplSet" });
   ```

3. **Check removal status** for each shard until `state: "completed"`:
   ```javascript
   db.adminCommand({ removeShard: "shard1ReplSet" });
   // Repeat for shard2ReplSet and shard3ReplSet
   ```

---

### **Step 2: Stop All MongoDB Instances for the Shards**
Stop the `mongod` processes for **all shard members** (primary and secondaries):

```bash
# Stop shard1ReplSet (port 27022)
mongosh --port 27022 --eval "db.adminCommand({ shutdown: 1 })"

# Stop shard2ReplSet (port 27023)
mongosh --port 27023 --eval "db.adminCommand({ shutdown: 1 })"

# Stop shard3ReplSet members (ports 27024 and 27033)
mongosh --port 27024 --eval "db.adminCommand({ shutdown: 1 })"
mongosh --port 27033 --eval "db.adminCommand({ shutdown: 1 })"
```

---

### **Step 3: Delete Data Directories**
Remove the data directories for all shards to erase all traces:
```bash
rm -rf ~/mongodb/shard1 ~/mongodb/shard2 ~/mongodb/shard3 ~/mongodb/secondary
```

---

### **Step 4: Clean Up Log Files (Optional)**
Delete the log files if no longer needed:
```bash
rm ~/mongodb/logs/shard1.log ~/mongodb/logs/shard2.log ~/mongodb/logs/shard3.log
```

---

### **Step 5: Verify Removal**
1. **Check the shard list** in `mongos`:
   ```javascript
   sh.status();
   ```
   The shards `shard1ReplSet`, `shard2ReplSet`, and `shard3ReplSet` should no longer appear.

2. **Confirm processes are stopped**:
   ```bash
   ps aux | grep mongod  # No processes for ports 27022, 27023, 27024, or 27033
   ```

---

### **Troubleshooting**
#### **If `removeShard` Fails**:
- Ensure the shard has no data left (use `sh.status()` to check).
- Force removal by first moving all databases off the shard:
  ```javascript
  use admin
  db.adminCommand({ movePrimary: "<database-name>", to: "other-shard" });
  ```

#### **If Processes Won’t Stop**:
- Kill them manually:
  ```bash
  sudo kill $(sudo lsof -t -i :27022)  # Repeat for 27023, 27024, 27033
  ```

To remove the config servers (`configReplSet`) and their data completely, follow these steps:

---

### **Step 1: Shut Down the Entire Sharded Cluster**
Config servers store metadata for the sharded cluster. To safely remove them, first shut down all components in this order:
1. **Stop all `mongos` routers**:
   ```bash
   # Find and kill mongos processes
   pkill -f "mongos"
   ```
2. **Stop all shards** (already done in previous steps).
3. **Stop the config servers**:
   ```bash
   # Stop config servers on ports 27019, 27020, and 27021
   mongosh --port 27019 --eval "db.adminCommand({ shutdown: 1 })"
   mongosh --port 27020 --eval "db.adminCommand({ shutdown: 1 })"
   mongosh --port 27021 --eval "db.adminCommand({ shutdown: 1 })"
   ```

---

### **Step 2: Delete Config Server Data Directories**
Remove the data directories for all config servers:
```bash
rm -rf ~/mongodb/config1 ~/mongodb/config2 ~/mongodb/config3
```

---

### **Step 3: Delete Config Server Log Files (Optional)**
```bash
rm ~/mongodb/logs/config1.log ~/mongodb/logs/config2.log ~/mongodb/logs/config3.log
```

---

### **Step 4: Verify All Processes Are Stopped**
```bash
ps aux | grep mongod  # No processes for ports 27019, 27020, or 27021 should appear
```

---

### **Important Notes**
- **Destructive Operation**: This permanently deletes the config servers and all cluster metadata.
- **Production Warning**: Never do this in production unless you intend to destroy the entire cluster.
- **Backup**: If you need to retain metadata, back up the `config` database first using `mongodump`.

---


---

### **Recreate a Fresh Cluster (Optional)**
To start over, reinitialize config servers, shards, and `mongos` with new directories.
---

### **Final Cleanup**
After completing these steps:
- The shards and their replica sets will be **fully removed**.
- Data directories and logs are deleted.
- The cluster will no longer reference these shards.