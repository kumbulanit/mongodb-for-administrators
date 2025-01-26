### **Practical Exercise: MongoDB Security & Monitoring**  
**Objective:** Secure a MongoDB deployment, monitor performance with `mongostat`, and analyze memory/I/O.  

---

### **1. Setup MongoDB (Ubuntu)**  
**Install MongoDB:**  
```bash 
# Import MongoDB GPG key 
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add - 

# Add repository 
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list 
sudo apt update 
sudo apt install -y mongodb-org 

# Start MongoDB 
sudo systemctl start mongod 
sudo systemctl enable mongod 
```  
**Verify Installation:**  
```bash 
mongosh --eval "db.runCommand({ping: 1})" 
```  

---

### **2. Enable Built-in Authentication**  
**Topic:** Security & Authentication  
**Steps:**  
1. **Create Admin User:**  
   ```bash 
   mongosh 
   ```  
   ```javascript 
   use admin 
   db.createUser({ 
     user: "admin", 
     pwd: "SecureAdmin123!", 
     roles: ["root"] 
   }) 
   exit 
   ```  

2. **Enable Authentication:**  
   Edit `/etc/mongod.conf`:  
   ```yaml 
   security: 
     authorization: enabled 
   ```  
   Restart MongoDB:  
   ```bash 
   sudo systemctl restart mongod 
   ```  

3. **Authenticate as Admin:**  
   ```bash 
   mongosh -u admin -p SecureAdmin123! --authenticationDatabase admin 
   ```  

**Troubleshooting:**  
- If authentication fails, check `/var/log/mongodb/mongod.log` for errors.  

---

### **3. Secure Deployment Recommendations**  
**Topic:** Network & Encryption  
**Steps:**  
1. **Bind to Local Network:**  
   Edit `/etc/mongod.conf`:  
   ```yaml 
   net: 
     bindIp: 127.0.0.1  # Restrict to localhost 
     port: 27017 
   ```  

2. **Enable TLS/SSL (Advanced):**  
   Generate a self-signed certificate:  
   ```bash 
   openssl req -newkey rsa:2048 -nodes -keyout mongodb.key -x509 -days 365 -out mongodb.crt 
   cat mongodb.key mongodb.crt > mongodb.pem 
   sudo mv mongodb.pem /etc/ssl/ 
   ```  
   Edit `/etc/mongod.conf`:  
   ```yaml 
   net: 
     tls: 
       mode: requireTLS 
       certificateKeyFile: /etc/ssl/mongodb.pem 
   ```  
   Restart MongoDB:  
   ```bash 
   sudo systemctl restart mongod 
   ```  

3. **Firewall Rules:**  
   ```bash 
   sudo ufw allow from 192.168.1.0/24 to any port 27017  # Allow internal network 
   sudo ufw enable 
   ```  

---

### **4. Monitoring with `mongostat`**  
**Topic:** Real-time Monitoring  
**Steps:**  
1. **Run `mongostat`:**  
   ```bash 
   mongostat -u admin -p SecureAdmin123! --authenticationDatabase admin 
   ```  
   **Output Example:**  
   ``` 
   inserts  queries update  delete getmore command dirty used flushes vsize  res qrw arw net_in net_out conn 
       1      0       0       0       0     1|0  0.0% 0.0%       0 1.01G 50M 0|0 0|0   1.02k   58.5k    1 
   ```  

2. **Key Metrics:**  
   - **inserts/queries/update/delete**: Operations per second.  
   - **vsize/res**: Virtual and resident memory usage.  
   - **net_in/net_out**: Network traffic.  

---

### **5. Analyzing Memory & I/O Performance**  
**Topic:** System Resource Monitoring  
**Steps:**  
1. **Check Memory Usage:**  
   ```bash 
   top -p $(pgrep mongod) 
   ```  
   - **RES**: Physical memory used by MongoDB.  

2. **Check I/O Statistics:**  
   ```bash 
   iostat -dx 2  # Update every 2 seconds 
   ```  
   - **%util**: Disk utilization.  
   - **await**: Average I/O wait time.  

3. **MongoDB Internal Stats:**  
   ```javascript 
   use admin 
   db.serverStatus().mem     // Memory details 
   db.serverStatus().wiredTiger.cache  // Cache usage 
   ```  

---

### **6. Advanced Monitoring**  
**Topic:** Integration with Prometheus & Grafana  
**Steps:**  
1. **Install Prometheus MongoDB Exporter:**  
   ```bash 
   docker run -d --name mongo-exporter -p 9216:9216 \ 
     -e MONGODB_URI="mongodb://admin:SecureAdmin123!@localhost:27017" \ 
     bitnami/mongodb-exporter:latest 
   ```  

2. **Configure Prometheus:**  
   Edit `/etc/prometheus/prometheus.yml`:  
   ```yaml 
   scrape_configs: 
     - job_name: 'mongodb' 
       static_configs: 
         - targets: ['localhost:9216'] 
   ```  

3. **Visualize in Grafana:**  
   - Import MongoDB dashboard ID `2583` from Grafana Labs.  

---

### **7. Troubleshooting**  
**Issue:** High Memory Usage  
- **Solution:**  
  - Check slow queries with `db.currentOp({"secs_running": {$gte: 5}})`  
  - Increase memory limits in `/etc/mongod.conf`:  
    ```yaml 
    storage: 
      wiredTiger: 
        engineConfig: 
          cacheSizeGB: 2  # Limit to 2GB 
    ```  

**Issue:** Authentication Errors  
- **Solution:**  
  - Verify user roles with `db.getUser("admin")`  
  - Reset password:  
    ```javascript 
    db.changeUserPassword("admin", "NewSecurePassword123!") 
    ```  

---

### **Exercise Summary**  
**Outcome:**  
- Secured MongoDB with authentication, TLS, and firewall rules.  
- Monitored real-time performance using `mongostat` and system tools.  
- Integrated advanced monitoring with Prometheus/Grafana.  

