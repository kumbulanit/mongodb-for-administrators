---

### 1. **Mongo Express** (Recommended)  
**Best for**: Lightweight, modern, and Docker-friendly.  
**Installation**:
```bash
# Install Node.js 18.x (default in Ubuntu 24.04)
sudo apt update && sudo apt install -y nodejs npm

# Install Mongo Express globally
sudo npm install -g mongo-express

# Configure (update MongoDB URL if remote/authenticated)
ME_CONFIG_MONGODB_URL="mongodb://localhost:27017" mongo-express
```
**Access**: `http://localhost:8081`  
**Docker (No Dependencies)**:  
```bash
docker run -p 8081:8081 -e ME_CONFIG_MONGODB_URL=mongodb://host.docker.internal:27017 mongo-express
```

---

### 2. **AdminMongo**  
**Best for**: Modern UI with user/role management.  
**Installation**:
```bash
# Install Node.js
sudo apt install -y nodejs npm

# Clone and run AdminMongo
git clone https://github.com/mrvautin/adminMongo.git
cd adminMongo
npm install && npm start
```
**Access**: `http://localhost:1234`  
**Docker**:  
```bash
docker run -p 1234:1234 --name adminmongo mrvautin/adminmongo
```

---

### 3. **phpMoAdmin** (Quick PHP-Based)  
**Best for**: Ultra-fast setup (single PHP file).  
**Installation**:
```bash
# Install PHP 8.3 (default in Ubuntu 24.04) and Apache
sudo apt install -y apache2 php libapache2-mod-php php-mongodb

# Download phpMoAdmin
sudo wget -O /var/www/html/moadmin.php https://github.com/clouddueling/moadmin-php/raw/master/moadmin.php

# Access via browser
http://localhost/moadmin.php
```
**Note**: Works with PHP 8.3 and MongoDB PHP driver 1.17+.

---

### 4. **RockMongo** (Avoid – Deprecated)  
**Not Recommended** for Ubuntu 24.04 due to PHP 8.3 incompatibility. Use alternatives above.

---

### 5. **MongoDB Compass** (Non-Open-Source)  
**For Reference**: Official GUI (free but proprietary).  
```bash
# Download .deb for Ubuntu 24.04
wget https://downloads.mongodb.com/compass/mongodb-compass_1.40.0_amd64.deb
sudo dpkg -i mongodb-compass_*.deb
```

---

### Key Notes for Ubuntu 24.04:  
- **Node.js 18.x** and **PHP 8.3** are preinstalled, simplifying setups.  
- **Docker** avoids dependency conflicts (use `docker compose` for production).  
- Avoid deprecated tools like RockMongo (not PHP 8.x compatible).  

For **easiest setup**: Use **Mongo Express** (Node.js) or **phpMoAdmin** (PHP). Both work out-of-the-box on Ubuntu 24.04.