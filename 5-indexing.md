

## **Step 1: Verify Existing Data**
1. **Check the `students` Collection**:
   - Verify that the `students` collection exists and contains data:
     ```javascript
     use university;
     db.students.find().limit(5);
     ```

2. **Check Existing Indexes**:
   - View the current indexes on the `students` collection:
     ```javascript
     db.students.getIndexes();
     ```

---

## **Step 2: Managing Indexes**
### **2.1 Create a Single Field Index**
- Create an index on the `email` field:
  ```javascript
  db.students.createIndex({ email: 1 });
  ```

- Verify the index:
  ```javascript
  db.students.getIndexes();
  ```

### **2.2 Create a Compound Index**
- Create a compound index on the `course` and `year` fields:
  ```javascript
  db.students.createIndex({ course: 1, year: -1 });
  ```

- Verify the index:
  ```javascript
  db.students.getIndexes();
  ```

### **2.3 Create a Geospatial Index**
- Create a `2dsphere` index on the `location` field:
  ```javascript
  db.students.createIndex({ location: "2dsphere" });
  ```

- Verify the index:
  ```javascript
  db.students.getIndexes();
  ```

---

## **Step 3: Query Optimization**
### **3.1 Query Using a Single Field Index**
- Find a student by email:
  ```javascript
  db.students.find({ email: "alice@university.edu" }).explain("executionStats");
  ```

- **Analysis**:
  - Check the `executionStats` to ensure the query uses the index (`IXSCAN`).

### **3.2 Query Using a Compound Index**
- Find students in the "Computer Science" course in year 3:
  ```javascript
  db.students.find({ course: "Computer Science", year: 3 }).explain("executionStats");
  ```

- **Analysis**:
  - Check the `executionStats` to ensure the query uses the compound index (`IXSCAN`).

### **3.3 Query Using a Geospatial Index**
- Find students near a specific location:
  ```javascript
  db.students.find({
    location: {
      $near: {
        $geometry: { type: "Point", coordinates: [-73.98, 40.76] },
        $maxDistance: 1000 // Distance in meters
      }
    }
  }).explain("executionStats");
  ```

- **Analysis**:
  - Check the `executionStats` to ensure the query uses the geospatial index (`IXSCAN`).

---

## **Step 4: Identifying Sub-Optimal Queries**
### **4.1 Enable the Query Profiler**
- Set the profiling level to log slow queries (e.g., >100ms):
  ```javascript
  db.setProfilingLevel(1, 100);
  ```

### **4.2 Run Sub-Optimal Queries**
- Run a query without an index (full collection scan):
  ```javascript
  db.students.find({ gpa: { $gt: 3.5 } });
  ```

- Run a query with an inefficient index:
  ```javascript
  db.students.find({ name: "Alice", age: { $gt: 20 } });
  ```

  - run efficient index after indexing email and age 
  ```javascript
  db.students.find({ email: "alicebrown@university.edu", age: { $gt: 19 } })
  ```

### **4.3 Analyze Profiler Data**
- View the profiler logs:
  ```javascript
  db.system.profile.find().sort({ ts: -1 }).limit(10);
  ```

- **Example Output**:
  ```json
  {
    "op": "query",
    "ns": "university.students",
    "millis": 250,
    "planSummary": "COLLSCAN",
    "keysExamined": 0,
    "docsExamined": 2000,
    "nreturned": 500
  }
  ```

- **Analysis**:
  - The query performs a full collection scan (`COLLSCAN`).
  - It examines 2000 documents but returns only 500.

### **4.4 Optimize the Query**
- Create an index on the `gpa` field:
  ```javascript
  db.students.createIndex({ gpa: 1 });
  ```

- Re-run the query and check the profiler:
  ```javascript
  db.students.find({ gpa: { $gt: 3.5 } }).explain("executionStats");
  ```

- **Analysis**:
  - The query now uses the index (`IXSCAN`).

---

## **Step 5: Advanced Techniques**
### **5.1 Covered Queries**
- Create a compound index for a covered query:
  ```javascript
  db.students.createIndex({ name: 1, age: 1 });
  ```

- Run a covered query:
  ```javascript
  db.students.find({ name: "Alice" }, { _id: 0, name: 1, age: 1 }).explain("executionStats");
  ```

- **Analysis**:
  - The query uses the index and does not fetch documents from disk.

### **5.2 Index Intersection**
- MongoDB can use multiple indexes for a single query.
- Example:
  ```javascript
  db.students.find({ name: "Alice", gpa: { $gt: 3.5 } }).explain("executionStats");
  ```

- **Analysis**:
  - Check if MongoDB uses index intersection.

### **5.3 Use `$indexStats` to Monitor Index Usage**
- View index usage statistics:
  ```javascript
  db.students.aggregate([{ $indexStats: {} }]);
  ```

---

## **Step 6: Summary**
This practical example demonstrates:
1. **Managing Indexes**:
   - Creating single field, compound, and geospatial indexes.
2. **Query Optimization**:
   - Using indexes to speed up queries.
3. **Identifying Sub-Optimal Queries**:
   - Using the query profiler to analyze and optimize slow queries.
4. **Advanced Techniques**:
   - Covered queries, index intersection, and monitoring index usage.

