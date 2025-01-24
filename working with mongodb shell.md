## **1. CRUD Operations**

### **1.1 Insert Documents**
Add new students to the `students` collection.

```javascript
// Insert a single document
db.students.insertOne({
  name: "John Doe",
  age: 22,
  email: "johndoe@university.edu",
  course: "Computer Science",
  subjects: ["Algorithms", "Data Structures"],
  year: 3,
  gpa: 3.8
});

// Insert multiple documents
db.students.insertMany([
  {
    name: "Jane Smith",
    age: 21,
    email: "janesmith@university.edu",
    course: "Mathematics",
    subjects: ["Calculus", "Linear Algebra"],
    year: 2,
    gpa: 3.9
  },
  {
    name: "Alice Brown",
    age: 20,
    email: "alicebrown@university.edu",
    course: "Physics",
    subjects: ["Quantum Mechanics", "Thermodynamics"],
    year: 1,
    gpa: 3.5
  }
]);
```

---

### **1.2 Query Documents**
Retrieve documents from the `students` collection.

```javascript
// Find all students
db.students.find();

// Find students in the "Computer Science" course
db.students.find({ course: "Computer Science" });

// Find students with a GPA greater than 3.5
db.students.find({ gpa: { $gt: 3.5 } });

// Find students in year 2 studying "Mathematics"
db.students.find({ year: 2, course: "Mathematics" });

// Find students with "Algorithms" in their subjects
db.students.find({ subjects: "Algorithms" });

// Limit results to 5 students
db.students.find().limit(5);

// Sort students by GPA in descending order
db.students.find().sort({ gpa: -1 });
```

---

### **1.3 Update Documents**
Modify existing documents in the `students` collection.

```javascript
// Update a single document
db.students.updateOne(
  { name: "John Doe" }, // Query
  { $set: { gpa: 4.0 } } // Update
);

// Update multiple documents
db.students.updateMany(
  { course: "Computer Science" }, // Query
  { $set: { year: 4 } } // Update
);

// Increment the age of all students by 1
db.students.updateMany({}, { $inc: { age: 1 } });

// Add a new subject to a student's subjects array
db.students.updateOne(
  { name: "Jane Smith" },
  { $push: { subjects: "Probability" } }
);
```

---

### **1.4 Delete Documents**
Remove documents from the `students` collection.

```javascript
// Delete a single document
db.students.deleteOne({ name: "Alice Brown" });

// Delete multiple documents
db.students.deleteMany({ course: "Physics" });

// Delete all documents in the collection
db.students.deleteMany({});
```

---

## **2. System Commands**
Use MongoDB system commands to manage the database and collections.

### **2.1 Database Commands**
```javascript
// Show all databases
show dbs;

// Switch to the `university` database
use university;

// Show current database
db;

// Drop the `university` database
db.dropDatabase();
```

### **2.2 Collection Commands**
```javascript
// Show all collections in the current database
show collections;

// Get statistics about the `students` collection
db.students.stats();

// Get the total size of the `students` collection
db.students.totalSize();

// Get the number of documents in the `students` collection
db.students.countDocuments();

// Rename the `students` collection to `learners`
db.students.renameCollection("learners");

// Drop the `learners` collection
db.learners.drop();
```

### **2.3 Index Commands**
```javascript
// Create an index on the `course` field
db.students.createIndex({ course: 1 });

// List all indexes on the `students` collection
db.students.getIndexes();

// Drop the index on the `course` field
db.students.dropIndex("course_1");
```

### **2.4 User Management Commands**
```javascript
// Create a new user with read/write access to the `university` database
db.createUser({
  user: "universityAdmin",
  pwd: "admin123",
  roles: [{ role: "readWrite", db: "university" }]
});

// List all users in the current database
db.getUsers();

// Delete a user
db.dropUser("universityAdmin");
```

---

## **3. Example Exercises**
Here are some exercises to practice using the data:

### **Exercise 1: Querying**
- Find all students in the "Mathematics" course with a GPA greater than 3.7.
- Find students who are in year 3 and study "Algorithms".

### **Exercise 2: Aggregation**
- Calculate the average GPA for each course.
- Count the number of students in each year.

### **Exercise 3: Updates**
- Increase the GPA of all students in the "Computer Science" course by 0.1.
- Add a new subject, "Machine Learning", to all students in the "Computer Science" course.

### **Exercise 4: Indexing**
- Create an index on the `year` field and measure query performance.
- Drop the index and compare query performance.

### **Exercise 5: System Commands**
- Backup the `university` database using `mongodump`.
- Restore the database using `mongorestore`.

---

## **4. Backup and Restore**
### **4.1 Backup the Database**
```bash
mongodump --db university --out /backup
```

### **4.2 Restore the Database**
```bash
mongorestore --db university /backup/university
```

---
