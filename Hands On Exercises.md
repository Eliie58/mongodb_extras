## Hands-On Exercises

### Setup

Run these commands in `mongosh` to prepare the dataset:

```javascript
use ecommerce

// Drop existing collection if needed
db.orders.drop()

// Insert sample data
db.orders.insertMany([
  { customerId: "C-01", status: "shipped",   orderDate: ISODate("2025-01-05"), total: 120.00, region: "EU",   tags: ["sale", "express"] },
  { customerId: "C-42", status: "pending",   orderDate: ISODate("2025-02-28"), total: 45.50,  region: "US",   tags: ["standard"] },
  { customerId: "C-42", status: "cancelled", orderDate: ISODate("2024-12-01"), total: 200.00, region: "US",   tags: ["sale"] },
  { customerId: "C-10", status: "shipped",   orderDate: ISODate("2025-01-20"), total: 88.00,  region: "EU",   tags: ["express"] },
  { customerId: "C-10", status: "cancelled", orderDate: ISODate("2024-11-01"), total: 15.00,  region: "EU",   tags: [] },
  { customerId: "C-05", status: "shipped",   orderDate: ISODate("2025-01-05"), total: 310.50, region: "US",   tags: ["sale", "express", "vip"] },
  { customerId: "C-07", status: "pending",   orderDate: ISODate("2025-03-01"), total: 55.00,  region: "APAC", tags: ["standard"] },
  { customerId: "C-99", status: "refunded",  orderDate: ISODate("2025-02-14"), total: 77.00,  region: "EU",   tags: ["sale"] },
  { customerId: "C-01", status: "pending",   orderDate: ISODate("2025-03-10"), total: 430.00, region: "EU",   tags: ["vip", "express"] },
  { customerId: "C-05", status: "pending",   orderDate: ISODate("2025-03-12"), total: 22.00,  region: "US",   tags: ["standard"] },
])
```

---

### Exercise 1 — Observe a COLLSCAN

**Goal:** See what a query looks like without any index.

```javascript
// Step 1: Make sure there are no extra indexes
db.orders.dropIndexes()  // This drops all non-_id indexes

// Step 2: Run this query and examine the output
db.orders.find({ status: "shipped" }).explain("executionStats")
```

**Questions to answer:**
1. What is the `stage` of the winning plan?
2. What are the values of `nReturned`, `totalDocsExamined`, `totalKeysExamined`?
3. How many more documents did MongoDB examine than it returned?

---

### Exercise 2 — Create a Single Field Index and Compare

**Goal:** Observe the difference an index makes.

```javascript
// Step 1: Create the index
db.orders.createIndex({ status: 1 })

// Step 2: Run the same query
db.orders.find({ status: "shipped" }).explain("executionStats")
```

**Questions to answer:**
1. What is the `stage` now?
2. What index name appears in the plan?
3. Compare `totalDocsExamined` before and after. What changed?
4. Does `nReturned` equal `totalKeysExamined`? What does that tell you?

---

### Exercise 3 — Detect an In-Memory Sort

**Goal:** See when MongoDB performs an expensive in-memory sort.

```javascript
// Step 1: Drop all non-_id indexes
db.orders.dropIndexes()

// Step 2: Run a query with a sort
db.orders.find({ status: "shipped" }).sort({ orderDate: -1 }).explain("executionStats")
```

**Questions to answer:**
1. Do you see a `SORT` stage in the plan? What does that mean?
2. Now create this index and run the same query again:

```javascript
db.orders.createIndex({ status: 1, orderDate: -1 })
db.orders.find({ status: "shipped" }).sort({ orderDate: -1 }).explain("executionStats")
```

3. Is there still a `SORT` stage? Why or why not?

---

### Exercise 4 — Apply the ESR Rule

**Goal:** Practice ordering index fields correctly.

You have this query:
```javascript
db.orders.find(
  {
    region: "EU",
    total: { $gte: 50 }
  }
).sort({ orderDate: -1 })
```

**Task:**

1. Identify which field is **E** (Equality), **S** (Sort), and **R** (Range).
2. Write the `createIndex` command using the correct ESR order.
3. Run the query with `.explain("executionStats")` both before and after creating your index.
4. Check that `totalDocsExamined` equals or is very close to `nReturned`.

**Expected answer:**
```javascript
db.orders.createIndex({ region: 1, orderDate: -1, total: 1 })
//                       E            S               R
```

---

### Exercise 5 — Compound Index and Prefix Rule

**Goal:** Understand which queries a compound index can and cannot serve.

```javascript
// Create this compound index
db.orders.createIndex({ region: 1, status: 1, customerId: 1 })

// Now test each of these queries and note the stage (IXSCAN or COLLSCAN)
// Query A
db.orders.find({ region: "EU" }).explain("executionStats")

// Query B
db.orders.find({ region: "EU", status: "shipped" }).explain("executionStats")

// Query C
db.orders.find({ region: "EU", status: "shipped", customerId: "C-10" }).explain("executionStats")

// Query D — does this use the index?
db.orders.find({ status: "shipped" }).explain("executionStats")

// Query E — does this use the index?
db.orders.find({ customerId: "C-10" }).explain("executionStats")
```

**Questions to answer:**
1. Which queries used the index? Which did not?
2. Can you explain why using the prefix rule?
3. What would you need to do to make Query D and E efficient?

---

### Exercise 6 — TTL Index

**Goal:** Create a TTL index and observe document expiry.

```javascript
// Step 1: Create a sessions collection
use ecommerce
db.sessions.drop()

// Step 2: Insert a session that expires 10 seconds from now
db.sessions.insertOne({
  userId: "U-001",
  token: "tok_abc",
  createdAt: new Date()
})

// Step 3: Create TTL index — expire after 10 seconds
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 10 })

// Step 4: Immediately count documents — should be 1
db.sessions.countDocuments()

// Step 5: Wait 60-70 seconds (TTL monitor runs every 60s), then check again
// In mongosh you can wait with a loop or just check after a minute
db.sessions.countDocuments()  // Should now be 0
```

**Questions to answer:**
1. Why doesn't the document disappear exactly after 10 seconds?
2. What would happen if the `createdAt` field contained a string instead of a Date?
3. Can you add a second field to the TTL index to make it a compound index?

---

### Exercise 7 — Unique Index

**Goal:** Enforce uniqueness and handle violations.

```javascript
// Step 1: Create a users collection with some data
db.users.drop()
db.users.insertMany([
  { email: "alice@example.com", name: "Alice" },
  { email: "bob@example.com",   name: "Bob" },
])

// Step 2: Create a unique index on email
db.users.createIndex({ email: 1 }, { unique: true })

// Step 3: Try inserting a duplicate
db.users.insertOne({ email: "alice@example.com", name: "Alice Again" })
// Observe the error message

// Step 4: Try inserting a document without an email field
db.users.insertOne({ name: "Charlie" })   // First time — OK
db.users.insertOne({ name: "Diana" })     // Second time — what happens?
```

**Questions to answer:**
1. What error code appears on a duplicate key violation?
2. Why does the second document without an email fail?
3. How would you create a unique index that allows multiple documents with a missing email?

---

### Exercise 8 — Partial Index

**Goal:** Create an index that only covers a subset of documents.

```javascript
// Step 1: Drop existing indexes
db.orders.dropIndexes()

// Step 2: Create a partial index — only for pending orders with total > 40
db.orders.createIndex(
  { customerId: 1 },
  {
    partialFilterExpression: {
      status: "pending",
      total: { $gt: 40 }
    }
  }
)

// Step 3: This query WILL use the partial index
db.orders.find(
  { customerId: "C-42", status: "pending", total: { $gt: 40 } }
).explain("executionStats")

// Step 4: This query will NOT use the partial index — why?
db.orders.find(
  { customerId: "C-42" }
).explain("executionStats")
```

**Questions to answer:**
1. What stage appears when the partial index is used?
2. What stage appears when the partial index is not used? How can you verify this?
3. How many documents does the partial index contain? Use `db.orders.getIndexes()` to find the index and think about it.
4. Why might a partial index be preferable to a full index in a large real-world collection?

---

### Bonus Exercise — Covered Query

**Goal:** Achieve zero `totalDocsExamined`.

```javascript
// Create this index
db.orders.createIndex({ customerId: 1, status: 1, total: 1 })

// Try to write a query that is "covered" — totalDocsExamined should be 0
// Hint: your projection must only include fields in the index, and exclude _id
db.orders.find(
  { customerId: "C-42" },
  { _id: 0, customerId: 1, status: 1, total: 1 }
).explain("executionStats")
```

**Questions to answer:**
1. What is `totalDocsExamined`? Did you achieve 0?
2. What happens if you add `orderDate: 1` to the projection?
3. What is the significance of `_id: 0` in achieving a covered query?

---

## 11. Summary Cheat Sheet

### Index Types at a Glance

| Index Type | Command | Use Case |
|---|---|---|
| Single field | `createIndex({ field: 1 })` | Simple lookups on one field |
| Compound | `createIndex({ a: 1, b: -1, c: 1 })` | Multi-field filter/sort |
| TTL | `createIndex({ date: 1 }, { expireAfterSeconds: N })` | Auto-expiring documents |
| Unique | `createIndex({ field: 1 }, { unique: true })` | Enforce uniqueness |
| Partial | `createIndex({ field: 1 }, { partialFilterExpression: {...} })` | Index subset of docs |
| Sparse | `createIndex({ field: 1 }, { sparse: true })` | Skip docs where field is missing |
| Multikey | `createIndex({ arrayField: 1 })` | Index array elements (auto) |
| Text | `createIndex({ field: "text" })` | Full-text search |

### The ESR Rule

```
{ Equality fields } → { Sort fields } → { Range fields }
```

### explain() Quick Reference

```javascript
.explain("executionStats")   // Always use this mode

// Look for:
stage: "IXSCAN"              // ✅ Index is being used
stage: "COLLSCAN"            // ❌ No index — check your query/indexes
stage: "SORT"                // ⚠️ In-memory sort — may need index adjustment

nReturned ≈ totalKeysExamined ≈ totalDocsExamined   // ✅ Good ratio
totalDocsExamined >> nReturned                       // ❌ Index is too broad
```

### Key Commands

```javascript
db.collection.createIndex({ field: 1 }, { options })   // Create
db.collection.getIndexes()                              // List all
db.collection.dropIndex("index_name")                   // Drop one
db.collection.dropIndexes()                             // Drop all (except _id)
db.collection.find({...}).explain("executionStats")     // Analyse
db.collection.find({...}).hint({ field: 1 })            // Force index
```

### When to Use Each Index Type

```
Frequent equality + sort + range queries   → Compound (ESR order)
Data that should auto-delete               → TTL
Enforce business uniqueness rules          → Unique
Large collections with conditional queries → Partial
Very large arrays or sparse fields         → Sparse
```

---

*End of course material. Students are encouraged to experiment further by loading a larger dataset (e.g., the MongoDB Atlas sample datasets) and repeating these exercises at scale to see the performance differences more dramatically.*
