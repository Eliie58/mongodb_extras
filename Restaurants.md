# MongoDB Data Analysis & Optimization Lab (1 Hour)

## Dataset: Restaurants (Native JSON)

We will use a JSON dataset of restaurants.

Download:
https://raw.githubusercontent.com/ozlerhakan/mongodb-json-files/refs/heads/master/datasets/restaurant.json

Import into MongoDB:
```bash
mongoimport <connection_string> --db lab --collection restaurants --file restaurants.json
```

---

## Objective

Practice:
- Filtering and sorting efficiently
- Writing aggregation pipelines
- Understanding and applying indexes
- Optimizing queries using explain()

---

## 1. Warm-up (10 minutes)

1. Find one restaurant located in Manhattan.
2. Find restaurants that serves "Italian" cuisine.
3. Count how many restaurants exist.
4. Find distinct boroughs.
5. Find restaurants with a grade of "A".

---

## 2. Filtering & Sorting (15 minutes)

1. Find the restaurant with the best rating (hint: grades).
2. Identify restaurants that are likely struggling (low grades or poor scores).
3. Find the most reviewed restaurants (based on number of grades entries).
4. Identify restaurants that consistently perform well (multiple "A" grades).
5. Find restaurants that recently improved their score.

---

## 3. Aggregation (15 minutes)

1. Which borough has the highest average score?
2. Which cuisine is the most common?
3. Which restaurant has the most inspections?
4. Find the average score per cuisine.
5. Identify cuisines with consistently high ratings.
6. Identify the most consistent restaurants.
   Find restaurants that have multiple inspections with similar scores (low variance).
   (Hint: think about measuring variation across grades.score)
7. Detect outlier restaurants.
   Find restaurants whose scores are significantly worse than others in the same borough.
   (Hint: compare each restaurant’s average score vs borough average)
---

## 4. Query Optimization & Indexes (20 minutes)

### Step 1: Analyze Queries

Take 2–3 queries from above and run:
```js
db.restaurants.find(...).explain("executionStats")
```

Questions:
1. Is MongoDB using a COLLSCAN or IXSCAN?
2. What is the execution time?
3. How many documents are examined?

---

### Step 2: Add Indexes

Create indexes to improve performance:

```js
db.restaurants.createIndex({ borough: 1 })
db.restaurants.createIndex({ cuisine: 1 })
db.restaurants.createIndex({ "grades.grade": 1 })
db.restaurants.createIndex({ borough: 1, cuisine: 1 })
```

---

### Step 3: Re-run Analysis

1. Re-run your queries with explain()
2. Compare:
   - documents examined
   - execution time
   - winning plan

---

### Step 4: Optimization Challenges

1. Optimize a query filtering by borough and cuisine.
2. Optimize a query filtering on grades.
3. Design a compound index for a query that:
   - filters by borough
   - sorts by cuisine=

---

## Bonus Challenges

1. Create a partial index (e.g., only restaurants with grade "A").
2. Create a multikey index on grades.
3. Compare performance before and after indexing.

---
