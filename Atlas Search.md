# Atlas Search

MongoDB's Atlas Search allows fine-grained text indexing and querying of data on your Atlas cluster. It enables advanced search functionality for your applications without any additional management or separate search system alongside your database.

Atlas Search provides options for several kinds of text analyzers, a rich query language that uses Atlas Search aggregation pipeline stages like `$search` and `$searchMeta` in conjunction with other MongoDB aggregation pipeline stages, and score-based results ranking.

## Using Atlas Search

### Step 1: Create an Atlas Search Index

Atlas Search index is a data structure that categorizes data in an easily searchable format. It is a mapping between terms and the documents that contain those terms. Atlas Search indexes enable faster retrieval of documents using certain identifiers. You must configure an Atlas Search index to query data in your Atlas cluster using Atlas Search.

1. Navigate to the Atlas Search page for your project.
2. Click <b>Create Search Index</b>
3. Select <b>Visual Editor</b> Method and click Next.
4. Enter the <b>Index Name</b>, and set the Database and Collection.
5. Specify an index definition.
6. Click Save Changes.
7. Click Create Search Index.
8. Close the You're All Set! Modal Window.
9. Wait for the index to finish building.

### Step 2: Run Search Queries

Try to execute the following pipelines:

```json
[
  {
    "$search": {
      "text": {
        "query": "baseball",
        "path": "plot"
      }
    }
  },
  {
    "$limit": 5
  },
  {
    "$project": {
      "_id": 0,
      "title": 1,
      "plot": 1
    }
  }
]
```

```json
[
  {
    "$search": {
      "compound": {
        "must": [
          {
            "text": {
              "query": ["Hawaii", "Alaska"],
              "path": "plot"
            }
          },
          {
            "regex": {
              "query": "([0-9]{4})",
              "path": "plot",
              "allowAnalyzedField": true
            }
          }
        ],
        "mustNot": [
          {
            "text": {
              "query": ["Comedy", "Romance"],
              "path": "genres"
            }
          },
          {
            "text": {
              "query": ["Beach", "Snow"],
              "path": "title"
            }
          }
        ]
      }
    }
  },
  {
    "$project": {
      "title": 1,
      "plot": 1,
      "genres": 1,
      "_id": 0
    }
  }
]
```

## Exercise

Follow this [Tutorial](https://www.mongodb.com/docs/atlas/atlas-search/tutorials/)

## Sources

- https://www.mongodb.com/docs/atlas/atlas-search/
- https://www.mongodb.com/docs/atlas/atlas-search/tutorial/create-index/
- https://www.mongodb.com/docs/atlas/atlas-search/tutorial/run-query/
- https://www.mongodb.com/docs/atlas/atlas-search/tutorials/
