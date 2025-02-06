# Aggregate Query
An **aggregation query** in MongoDB and Mongoose is a powerful way to process and analyze data stored in collections. It allows you to perform complex data transformations, computations, and groupings in a single query. Aggregation queries are executed using the **aggregation pipeline**, which is a framework for data processing in MongoDB.

The aggregation pipeline consists of **stages**, where each stage processes the input documents and passes the results to the next stage. Each stage performs a specific operation, such as filtering, grouping, sorting, or calculating values.


## Key Concepts of Aggregation Queries

1. **Pipeline**:
   - A pipeline is a sequence of stages that are applied to the data.
   - Each stage takes the output of the previous stage as input and processes it.

2. **Stages**:
   - Stages are the building blocks of an aggregation pipeline.
   - Each stage performs a specific operation, such as filtering, grouping, or sorting.
   - Common stages include `$match`, `$group`, `$sort`, `$project`, `$count`, and `$facet`.

3. **Documents**:
   - The data in MongoDB is stored as documents (similar to JSON objects).
   - Aggregation queries process these documents and produce results.

MongoDB's aggregation framework provides a variety of **stages** (commands) to process and transform data in a pipeline. Below is a comprehensive list of commonly used aggregation stages, along with examples of their syntax and usage.


## **Aggregation Stages**

#### 1. **`$match`**  
Filters documents to pass only those that match the specified condition(s).  
**Syntax**:  
```javascript
{ $match: { <query> } }
```
**Example**:  
```javascript
// Filter students with graduationType = "UG"
{ $match: { graduationType: "UG" } }
```

---

#### 2. **`$group`**  
Groups documents by a specified `_id` and performs aggregations (e.g., count, sum).  
**Syntax**:  
```javascript
{ $group: { _id: <expression>, <field>: { <accumulator> } } }
```
**Example**:  
```javascript
// Group by graduationType and count students
{ $group: { _id: "$graduationType", total: { $sum: 1 } } }
```

---

#### 3. **`$sort`**  
Sorts documents by specified fields.  
**Syntax**:  
```javascript
{ $sort: { <field1>: 1 | -1, <field2>: 1 | -1 } } // 1 = ascending, -1 = descending
```
**Example**:  
```javascript
// Sort by totalStudents in descending order
{ $sort: { totalStudents: -1 } }
```

---

#### 4. **`$project`**  
Selects or reshapes fields to include/exclude in the output.  
**Syntax**:  
```javascript
{ $project: { <field1>: 1 | 0, <newField>: <expression> } }
```
**Example**:  
```javascript
// Include name and graduationType, exclude _id
{ $project: { _id: 0, name: 1, graduationType: 1 } }
```

---

#### 5. **`$limit`**  
Limits the number of documents passed to the next stage.  
**Syntax**:  
```javascript
{ $limit: <number> }
```
**Example**:  
```javascript
// Pass only the first 10 documents
{ $limit: 10 }
```

---

#### 6. **`$skip`**  
Skips a specified number of documents.  
**Syntax**:  
```javascript
{ $skip: <number> }
```
**Example**:  
```javascript
// Skip the first 5 documents
{ $skip: 5 }
```

---

#### 7. **`$unwind`**  
Deconstructs an array field into multiple documents (one per array element).  
**Syntax**:  
```javascript
{ $unwind: "$<arrayField>" }
```
**Example**:  
```javascript
// Unwind the "courses" array
{ $unwind: "$courses" }
```

---

### **Advanced Aggregation Stages**

#### 8. **`$lookup`**  
Performs a left outer join with another collection.  
**Syntax**:  
```javascript
{
  $lookup: {
    from: "<collection>",
    localField: "<field>",
    foreignField: "<field>",
    as: "<outputArray>"
  }
}
```
**Example**:  
```javascript
// Join students with courses collection
{
  $lookup: {
    from: "courses",
    localField: "courseId",
    foreignField: "_id",
    as: "enrolledCourses"
  }
}
```

---

#### 9. **`$count`**  
Counts the number of documents at the current stage.  
**Syntax**:  
```javascript
{ $count: "<fieldName>" }
```
**Example**:  
```javascript
// Count total documents
{ $count: "totalStudents" }
```

---

#### 10. **`$facet`**  
Runs multiple independent pipelines within a single stage.  
**Syntax**:  
```javascript
{
  $facet: {
    "<output1>": [ <pipeline> ],
    "<output2>": [ <pipeline> ]
  }
}
```
**Example**:  
```javascript
// Compute total UG and PG students in one query
{
  $facet: {
    totalUG: [{ $match: { graduationType: "UG" } }, { $count: "count" }],
    totalPG: [{ $match: { graduationType: "PG" } }, { $count: "count" }]
  }
}
```

---

#### 11. **`$addFields`**  
Adds new fields to documents (similar to `$project` but retains existing fields).  
**Syntax**:  
```javascript
{ $addFields: { <newField>: <expression> } }
```
**Example**:  
```javascript
// Add a "fullName" field
{ $addFields: { fullName: { $concat: ["$firstName", " ", "$lastName"] } } }
```

---

#### 12. **`$replaceRoot`**  
Replaces the document with a new object (e.g., promote a nested field to the root).  
**Syntax**:  
```javascript
{ $replaceRoot: { newRoot: <expression> } }
```
**Example**:  
```javascript
// Replace the root with the "profile" subdocument
{ $replaceRoot: { newRoot: "$profile" } }
```

---

#### 13. **`$out`**  
Writes the result of the aggregation pipeline to a new collection.  
**Syntax**:  
```javascript
{ $out: "<collectionName>" }
```
**Example**:  
```javascript
// Save results to a "studentStats" collection
{ $out: "studentStats" }
```

---

#### 14. **`$merge`**  
Merges results into an existing collection (MongoDB 4.2+).  
**Syntax**:  
```javascript
{
  $merge: {
    into: "<collection>",
    on: "<field>", // Optional
    whenMatched: "replace" | "keepExisting" | "merge" | "pipeline",
    whenNotMatched: "insert" | "discard"
  }
}
```
**Example**:  
```javascript
// Merge results into "studentStats" collection
{ $merge: { into: "studentStats", whenMatched: "replace" } }
```

---

### **Specialized Stages**

#### 15. **`$bucket`**  
Groups documents into "buckets" based on a specified expression.  
**Example**:  
```javascript
{
  $bucket: {
    groupBy: "$age",
    boundaries: [18, 25, 35, 50],
    default: "Other",
    output: { count: { $sum: 1 } }
  }
}
```

---

#### 16. **`$geoNear`**  
Returns documents based on proximity to a geospatial point.  
**Example**:  
```javascript
{
  $geoNear: {
    near: { type: "Point", coordinates: [ -73.9667, 40.78 ] },
    distanceField: "distance",
    maxDistance: 5000,
    spherical: true
  }
}
```

---

### **Sample Aggregation Pipeline**

Here’s a pipeline that combines multiple stages to:
1. Filter UG students.
2. Group them by course.
3. Sort by count in descending order.
4. Limit to the top 3 courses.

```javascript
db.students.aggregate([
  { $match: { graduationType: "UG" } },
  { $group: { _id: "$course", count: { $sum: 1 } } },
  { $sort: { count: -1 } },
  { $limit: 3 }
]);
```

---

### **Best Practices**
1. **Order Matters**: Place `$match` early to reduce the number of documents processed.
2. **Use Indexes**: Add indexes to fields used in `$match`, `$sort`, or `$group`.
3. **Avoid Unnecessary Stages**: Remove unused stages (e.g., `$project`) to optimize performance.

---