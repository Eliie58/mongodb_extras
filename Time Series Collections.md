# Time Series Collections

## Introduction

Time series data is a sequence of data points in which insights are gained by analyzing changes over time.

Time series data is generally composed of these components:

- <b>Time</b> when the data point was recorded.
- <b>Metadata</b> (sometimes referred to as source), which is a label or tag that uniquely identifies a series and rarely changes.
- <b>Measurements</b> (sometimes referred to as metrics or values), which are the data points tracked at increments in time. Generally these are key-value pairs that change over time.

## Create a Time Series Collection

Create the collection using either the [db.createCollection()](https://www.mongodb.com/docs/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection) method or the create command. For example:

```
db.createCollection(
    "weather",
    {
      timeseries: {
          timeField: "timestamp",
          metaField: "metadata"
      }
  }
)
```

You can set the `timeField` to the field that contains time data, and the `metaField` to the field that contains metadata.

### Optional

You can also define:

- `granuality`: `seconds` (default), `minutes`, or `hours`.
- `expireAfterSeconds`: Enable the automatic deletion of documents in a time series collection by specifying the number of seconds after which documents expire
  ```
  db.createCollection(
    "weather",
    {
        timeseries: {
            timeField: "timestamp",
            metaField: "metadata"
        },
        expireAfterSeconds: 86400
    }
  )
  ```

## Insert Measurments into a Time Series Collection

Each document you insert should contain a single measurement. To insert multiple documents at once, issue the following command:

```
db.weather.insertMany( [
   {
      "metadata": { "sensorId": 5578, "type": "temperature" },
      "timestamp": ISODate("2021-05-18T00:00:00.000Z"),
      "temp": 12
   },
   {
      "metadata": { "sensorId": 5578, "type": "temperature" },
      "timestamp": ISODate("2021-05-18T04:00:00.000Z"),
      "temp": 11
   },
   {
      "metadata": { "sensorId": 5578, "type": "temperature" },
      "timestamp": ISODate("2021-05-18T08:00:00.000Z"),
      "temp": 11
   },
   {
      "metadata": { "sensorId": 5578, "type": "temperature" },
      "timestamp": ISODate("2021-05-18T12:00:00.000Z"),
      "temp": 12
   },
   {
      "metadata": { "sensorId": 5578, "type": "temperature" },
      "timestamp": ISODate("2021-05-18T16:00:00.000Z"),
      "temp": 16
   },
   {
      "metadata": { "sensorId": 5578, "type": "temperature" },
      "timestamp": ISODate("2021-05-18T20:00:00.000Z"),
      "temp": 15
   }, {
      "metadata": { "sensorId": 5578, "type": "temperature" },
      "timestamp": ISODate("2021-05-19T00:00:00.000Z"),
      "temp": 13
   },
   {
      "metadata": { "sensorId": 5578, "type": "temperature" },
      "timestamp": ISODate("2021-05-19T04:00:00.000Z"),
      "temp": 12
   },
   {
      "metadata": { "sensorId": 5578, "type": "temperature" },
      "timestamp": ISODate("2021-05-19T08:00:00.000Z"),
      "temp": 11
   },
   {
      "metadata": { "sensorId": 5578, "type": "temperature" },
      "timestamp": ISODate("2021-05-19T12:00:00.000Z"),
      "temp": 12
   },
   {
      "metadata": { "sensorId": 5578, "type": "temperature" },
      "timestamp": ISODate("2021-05-19T16:00:00.000Z"),
      "temp": 17
   },
   {
      "metadata": { "sensorId": 5578, "type": "temperature" },
      "timestamp": ISODate("2021-05-19T20:00:00.000Z"),
      "temp": 12
   }
] )
```

## Query a Time Series Collection

You query a time series collection the same way you query a standard MongoDB collection.

To return one document from a time series collection, run:

```
db.weather.findOne({
   "timestamp": ISODate("2021-05-18T00:00:00.000Z")
})
```

## Run Aggregations on a Time Series Collection

For additional query functionality, use an [aggregation pipeline](https://www.mongodb.com/docs/manual/core/aggregation-pipeline/#std-label-aggregation-pipeline) such as:

```
db.weather.aggregate( [
   {
      $project: {
         date: {
            $dateToParts: { date: "$timestamp" }
         },
         temp: 1
      }
   },
   {
      $group: {
         _id: {
            date: {
               year: "$date.year",
               month: "$date.month",
               day: "$date.day"
            }
         },
         avgTmp: { $avg: "$temp" }
      }
   }
] )
```

## Deal with missing data

Time series data frequently exhibit gaps for a number of reasons, including measurement failures, formatting problems, human errors or a lack of information to record. On the other hand, time series data must be continuous to perform analytics and guarantee accurate outcome.

### [$densify](https://www.mongodb.com/docs/manual/reference/operator/aggregation/densify/)

Creates new documents in a sequence of documents where certain values in a field are missing.

You can use $densify to:

- Fill gaps in time series data.
- Add missing values between groups of data.
- Populate your data with a specified range of values.

Example:

```
db.weather.deleteMany({})

db.weather.insertMany( [
   {
       "metadata": { "sensorId": 5578, "type": "temperature" },
       "timestamp": ISODate("2021-05-18T00:00:00.000Z"),
       "temp": 12
   },
   {
       "metadata": { "sensorId": 5578, "type": "temperature" },
       "timestamp": ISODate("2021-05-18T04:00:00.000Z"),
       "temp": 11
   },
   {
       "metadata": { "sensorId": 5578, "type": "temperature" },
       "timestamp": ISODate("2021-05-18T08:00:00.000Z"),
       "temp": 11
   },
   {
       "metadata": { "sensorId": 5578, "type": "temperature" },
       "timestamp": ISODate("2021-05-18T12:00:00.000Z"),
       "temp": 12
   }
] )

db.weather.aggregate( [
   {
      $densify: {
         field: "timestamp",
         range: {
            step: 1,
            unit: "hour",
            bounds:[ ISODate("2021-05-18T00:00:00.000Z"), ISODate("2021-05-18T08:00:00.000Z") ]
         }
      }
   }
] )
```

More [Examples](https://www.mongodb.com/docs/manual/reference/operator/aggregation/densify/#examples)

### $fill

Populates `null` and missing field values within documents.

You can use $fill to populate missing data points:

- In a sequence based on surrounding values.
- With a fixed value.

Example:

```
db.weather.aggregate( [
   {
      $densify: {
         field: "timestamp",
         range: {
            step: 1,
            unit: "hour",
            bounds:[ ISODate("2021-05-18T00:00:00.000Z"), ISODate("2021-05-18T08:00:00.000Z") ]
         }
      }
   },
   {
      $fill: {
         sortBy : { timestamp : 1 },
         output : {
            "temp" : {
                method : "linear"
            },
            "metadata" : {
                method : "locf"
            }
         }
      }
   }
] )
```

More [Examples](https://www.mongodb.com/docs/manual/reference/operator/aggregation/fill/#examples)

## Sources

- https://medium.com/data-reply-it-datatech/time-series-with-mongodb-dd60f8d6acd6
- https://www.mongodb.com/docs/manual/core/timeseries-collections/
- https://www.mongodb.com/docs/manual/core/timeseries/timeseries-procedures/#std-label-timeseries-create-query-procedures
- https://www.mongodb.com/docs/manual/reference/operator/aggregation/fill
- https://www.mongodb.com/docs/manual/reference/operator/aggregation/densify
