# TTL Index

TTL indexes are special single-field indexes that MongoDB can use to automatically remove documents from a collection after a certain amount of time or at a specific clock time. Data expiration is useful for certain types of information like machine generated event data, logs, and session information that only need to persist in a database for a finite amount of time.

## Create a TTL Index

To create a TTL index, use [createIndex()](https://www.mongodb.com/docs/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex). Specify an index field that is either a date type or an array that contains date type values. Use the `expireAfterSeconds` option to specify a TTL value in seconds.

For example, to create a TTL index on the `lastModifiedDate` field of the `eventlog` collection with a TTL value of `3600` seconds, use the following operation:

```
db.eventlog.createIndex(
    { "lastModifiedDate": 1 },
    { expireAfterSeconds: 3600 }
)
```

## Change the expireAfterSeconds value for a TTL Index

To change the `expireAfterSeconds` value for a TTL Index, use the [collMod](https://www.mongodb.com/docs/manual/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod) database command.

The following example changes the `expireAfterSeconds` value for an index with the pattern `{ "lastModifiedDate": 1 }` on the `tickets` collection:

```
db.runCommand({
  "collMod": "tickets",
  "index": {
    "keyPattern": { "lastModifiedDate": 1 },
    "expireAfterSeconds": 100
  }
})
```

## Expire Data from Collections by Setting TTL

There are 2 appraoches for expiring documents from a Collection using TTL indexes:

### Expire Documents after a Specified Number of Seconds

To expire data after a specified number of seconds has passed since the indexed field, create a TTL index on a field that holds values of BSON date type or an array of BSON date-typed objects and specify a positive non-zero value in the expireAfterSeconds field.

For example, the following operation creates an index on the `log_events` collection's `createdAt` field and specifies the `expireAfterSeconds` value of `10` to set the expiration time to be ten seconds after the time specified by `createdAt`.

```
db.log_events.createIndex(
    { "createdAt": 1 },
    { expireAfterSeconds: 10 }
)
```

### Expire Documents at a Specific Clock Time

To expire documents at a specific clock time, begin by creating a TTL index on a field that holds values of BSON date type or an array of BSON date-typed objects and specify an expireAfterSeconds value of 0. For each document in the collection, set the indexed date field to a value corresponding to the time the document should expire. If the indexed date field contains a date in the past, MongoDB considers the document expired.

For example, the following operation creates an index on the `log_events` collection's `expireAt` field and specifies the `expireAfterSeconds` value of `0`:

```
db.log_events.createIndex(
    { "expireAt": 1 },
    { expireAfterSeconds: 0 }
)
```

## Sources

- https://www.mongodb.com/docs/manual/core/index-ttl/
- https://www.mongodb.com/docs/manual/tutorial/expire-data/
