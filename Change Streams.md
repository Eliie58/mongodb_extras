# Change Streams

Change streams allow applications to access real-time data changes without the prior complexity and risk of manually tailing the oplog. Applications can use change streams to subscribe to all data changes on a single collection, a database, or an entire deployment, and immediately react to them. Because change streams use the aggregation framework, applications can also filter for specific changes or transform the notifications at will.

## Watch a collection, Database, or Deployment

You can open change streams against:

- A collection
- A database
- A deployment

## Open a change Stream

The following example opens a change stream for a collection and iterates over the cursor to retrieve the change stream documents:

```
const collection = db.inventory;
const changeStream = collection.watch();
while(changeStream.hasNext()) {
    print(changeStream.next())
}
```

## Modify Change Stream Output

You can control change stream output by providing an array of one or more of the following pipeline stages when configuring the change stream:

- $addFields
- $match
- $project
- $replaceRoot
- $replaceWith
- $redact
- $set
- $unset

The following example uses stream to process the change events.

```
const pipeline = [
  { $match: { 'fullDocument.username': 'alice' } },
  { $addFields: { newField: 'this is an added field!' } }
];
const collection = db.inventory;
const changeStream = collection.watch(pipeline);
while(changeStream.hasNext()) {
    print(changeStream.next())
}
```

## Use Cases

Change streams can benefit architectures with reliant business systems, informing downstream systems once data changes are durable. For example, change streams can save time for developers when implementing Extract, Transform, and Load (ETL) services, cross-platform synchronization, collaboration functionality, and notification services.

## Exercise

Python exercise using Python Notebook

### Setup

```
!pip install "pymongo[srv]"
!curl ipecho.net/plain
```

### Open Change Stream

```
import pymongo
from bson.json_util import dumps

client = pymongo.MongoClient(......)
change_stream = client.changestream.collection.watch()
for change in change_stream:
    print(dumps(change))
    print('') # for readability only
```

## Sources

- https://www.mongodb.com/docs/manual/changeStreams/
- https://www.mongodb.com/developer/languages/python/python-change-streams/
