If you're working with an **on-premises MongoDB deployment** and want to use a **web-based console** for monitoring and management, you can use **MongoDB Ops Manager** or open-source alternatives like **MongoDB Express** or **RockMongo**. 

---

## **1. MongoDB Ops Manager (On-Premises)**
MongoDB Ops Manager is a comprehensive tool for managing and monitoring MongoDB deployments. It provides a web-based interface for real-time monitoring, backup, and automation.

### **1.1 Key Features**
- **Real-Time Monitoring**: Track performance metrics like CPU, memory, and disk usage.
- **Automation**: Automate deployment, scaling, and upgrades.
- **Backup and Restore**: Manage backups and restore data.
- **Alerts**: Set up alerts for critical metrics.

### **1.2 Installation Steps**
1. **Download Ops Manager**:
   - Download MongoDB Ops Manager from the [MongoDB website](https://www.mongodb.com/try/download/ops-manager).

2. **Install Ops Manager**:
   - Follow the installation instructions for your operating system.
   - Example for Linux:
     ```bash
     tar -xvzf mongodb-mms-<version>.tar.gz
     cd mongodb-mms-<version>
     ./bin/mongodb-mms start
     ```

3. **Configure Ops Manager**:
   - Access the Ops Manager web interface at `http://<ops-manager-host>:8080`.
   - Follow the setup wizard to configure Ops Manager.

4. **Deploy MongoDB Agents**:
   - Install MongoDB Agents on each MongoDB instance to enable monitoring and automation.
   - Example:
     ```bash
     curl -OL https://downloads.mongodb.com/on-prem-mms/rpm/mongodb-mms-monitoring-agent-<version>.x86_64.rpm
     sudo rpm -i mongodb-mms-monitoring-agent-<version>.x86_64.rpm
     ```

5. **Monitor MongoDB**:
   - Use the Ops Manager web interface to monitor your MongoDB deployment.

---

## **2. MongoDB Express (Open Source)**
MongoDB Express is a lightweight, open-source web-based MongoDB admin interface.

### **2.1 Key Features**
- **CRUD Operations**: Perform insert, update, delete, and query operations.
- **Database Management**: View and manage databases and collections.
- **User-Friendly Interface**: Simple and intuitive web interface.

### **2.2 Installation Steps**
1. **Install Node.js**:
   - MongoDB Express requires Node.js. Install it using:
     ```bash
     sudo apt update
     sudo apt install nodejs npm
     ```

2. **Install MongoDB Express**:
   - Install MongoDB Express globally using npm:
     ```bash
     npm install -g mongo-express
     ```

3. **Configure MongoDB Express**:
   - Create a configuration file (`config.js`) with the following content:
     ```javascript
     module.exports = {
       mongodb: {
         connectionString: 'mongodb://localhost:27017'
       },
       site: {
         port: 8081
       }
     };
     ```

4. **Start MongoDB Express**:
   - Run MongoDB Express:
     ```bash
     mongo-express -c config.js
     ```

5. **Access MongoDB Express**:
   - Open a browser and navigate to `http://localhost:8081`.

---

## **3. RockMongo (Open Source)**
RockMongo is another open-source MongoDB admin tool with a PHP-based web interface.

### **3.1 Key Features**
- **CRUD Operations**: Perform insert, update, delete, and query operations.
- **Database Management**: View and manage databases and collections.
- **Query Execution**: Execute MongoDB queries directly from the web interface.

### **3.2 Installation Steps**
1. **Install PHP and Apache**:
   - Install PHP and Apache on your server:
     ```bash
     sudo apt update
     sudo apt install apache2 php libapache2-mod-php
     ```

2. **Download RockMongo**:
   - Download RockMongo from the [official website](https://rockmongo.com/).

3. **Extract and Configure**:
   - Extract the RockMongo files to the Apache web directory:
     ```bash
     sudo tar -xvzf rockmongo-<version>.tar.gz -C /var/www/html/
     sudo mv /var/www/html/rockmongo-<version> /var/www/html/rockmongo
     ```

4. **Configure RockMongo**:
   - Edit the configuration file (`config.php`):
     ```php
     $MONGO["servers"][0]["host"] = "localhost";
     $MONGO["servers"][0]["port"] = "27017";
     ```

5. **Access RockMongo**:
   - Open a browser and navigate to `http://<server-ip>/rockmongo`.

---

## **4. Example Workflow**
### **Step 1: Install MongoDB Express**
```bash
npm install -g mongo-express
```

### **Step 2: Configure MongoDB Express**
Create `config.js`:
```javascript
module.exports = {
  mongodb: {
    connectionString: 'mongodb://localhost:27017'
  },
  site: {
    port: 8081
  }
};
```

### **Step 3: Start MongoDB Express**
```bash
mongo-express -c config.js
```

### **Step 4: Access MongoDB Express**
- Open a browser and navigate to `http://localhost:8081`.

---

## **5. Summary**
- **MongoDB Ops Manager**: A comprehensive tool for on-premises MongoDB management and monitoring.
- **MongoDB Express**: A lightweight, open-source web interface for MongoDB.
- **RockMongo**: A PHP-based open-source MongoDB admin tool.

