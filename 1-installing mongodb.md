Below is a **step-by-step guide** for installing MongoDB on **Ubuntu 24**, creating a database for a university college, and populating it with student data. This setup will include 2000 students with different study paths and a combination of subjects and courses. The data will be structured in a way that allows for various exercises.

---

## **Step 1: Install MongoDB on Ubuntu 24**
### **1.1 Update the System**
```bash
sudo apt update
sudo apt upgrade -y
```
Install MongoDB Community Edition
Follow these steps to install MongoDB Community Edition using the apt package manager.

1
Import the public key.

From a terminal, install gnupg and curl if they are not already available:
```bash
sudo apt-get install gnupg curl
```
To import the MongoDB public GPG key, run the following command:
```bash
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg \
   --dearmor
```
2
Create the list file.

Create the list file /etc/apt/sources.list.d/mongodb-org-8.0.list for your version of Ubuntu.

```bash
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```
3
Reload the package database.

Issue the following command to reload the local package database:
```bash
sudo apt-get update
```

4
Install MongoDB Community Server.

You can install either the latest stable version of MongoDB or a specific version of MongoDB.


Latest Release

Specific Release
To install the latest stable version, issue the following
```bash
sudo apt-get install -y mongodb-org
```

5. Start and enable MongoDB:
   ```bash
   sudo systemctl start mongod
   sudo systemctl enable mongod
   ```

6. Verify MongoDB is running:
   ```bash
   sudo systemctl status mongod
   ```

---

## **Step 2: Access MongoDB Shell**
1. Open the MongoDB shell:
   ```bash
   mongosh
   ```

2. Switch to the `admin` database:
   ```bash
   use admin
   ```

3. Create an admin user (for database management):
   ```bash
   db.createUser({
     user: "admin",
     pwd: "admin123",
     roles: [{ role: "userAdminAnyDatabase", db: "admin" }]
   })
   ```

4. Authenticate as the admin user:
   ```bash
   db.auth("admin", "admin123")
   ```

---

## **Step 3: Create a Database for the University**
1. Switch to a new database called `university`:
   ```bash
   use university
   ```

2. Create a collection called `students`:
   ```bash
   db.createCollection("students")
   ```

---

## **Step 4: Populate the Database with Student Data**
We will create 2000 student documents with the following fields:
- `_id`: Unique identifier (auto-generated).
- `name`: Student's full name.
- `age`: Student's age.
- `email`: Student's email.
- `course`: The course the student is enrolled in.
- `subjects`: An array of subjects the student is studying.
- `year`: The academic year (e.g., 1, 2, 3, 4).
- `gpa`: The student's GPA (randomized for realism).

### **4.1 Generate Student Data**
Use a script to generate 2000 student documents. Below is an example using JavaScript in the MongoDB shell:

```javascript
// Function to generate random data
function getRandomElement(array) {
  return array[Math.floor(Math.random() * array.length)];
}

// Define possible values
const courses = ["Computer Science", "Mathematics", "Physics", "Biology", "Chemistry", "Engineering"];
const subjects = ["Algorithms", "Calculus", "Quantum Mechanics", "Genetics", "Organic Chemistry", "Thermodynamics"];
const years = [1, 2, 3, 4];

// Generate 2000 students
for (let i = 1; i <= 2000; i++) {
  const student = {
    name: `Student ${i}`,
    age: Math.floor(Math.random() * 10) + 18, // Random age between 18 and 27
    email: `student${i}@university.edu`,
    course: getRandomElement(courses),
    subjects: [getRandomElement(subjects), getRandomElement(subjects)], // Random 2 subjects
    year: getRandomElement(years),
    gpa: (Math.random() * 4).toFixed(2) // Random GPA between 0 and 4
  };
  db.students.insertOne(student);
}
```

### **4.2 Verify Data Insertion**
1. Check the number of documents in the `students` collection:
   ```bash
   db.students.countDocuments()
   ```

2. View a sample document:
   ```bash
   db.students.findOne()
   ```

---

## **Step 5: Add Different Types of Documents**
To make the dataset more versatile for exercises, add additional documents with different structures. For example:

### **5.1 Add Faculty Data**
1. Create a `faculty` collection:
   ```bash
   db.createCollection("faculty")
   ```

2. Insert faculty documents:
   ```javascript
   db.faculty.insertMany([
     { name: "Dr. Smith", department: "Computer Science", email: "smith@university.edu" },
     { name: "Dr. Johnson", department: "Mathematics", email: "johnson@university.edu" },
     { name: "Dr. Brown", department: "Physics", email: "brown@university.edu" }
   ]);
   ```

### **5.2 Add Course Data**
1. Create a `courses` collection:
   ```bash
   db.createCollection("courses")
   ```

2. Insert course documents:
   ```javascript
   db.courses.insertMany([
     { courseName: "Computer Science", credits: 120, duration: "4 years" },
     { courseName: "Mathematics", credits: 100, duration: "3 years" },
     { courseName: "Physics", credits: 110, duration: "4 years" }
   ]);
   ```

---

## **Step 6: Use the Data for Exercises**
The dataset can now be used for various exercises, such as:
1. **Querying**:
   - Find all students in the "Computer Science" course.
   - Find students with a GPA greater than 3.5.
2. **Aggregation**:
   - Calculate the average GPA by course.
   - Count the number of students in each year.
3. **Indexing**:
   - Create an index on the `course` field and measure query performance.
4. **Updates**:
   - Update the GPA of all students in the "Mathematics" course.
5. **Joins** (using `$lookup`):
   - Join the `students` and `courses` collections to display student details with course information.

---

## **Step 7: Backup the Database**
1. Export the `university` database to a JSON file:
   ```bash
   mongodump --db university --out /backup
   ```

2. Restore the database (if needed):
   ```bash
   mongorestore --db university /backup/university
   ```

---

This step-by-step guide provides a complete setup for MongoDB on Ubuntu 24, including creating a realistic dataset for a university college. Let me know if you need further assistance!