# Qdrant

Nodejs library for the Qdrant vector search engine (https://qdrant.tech)

## Install

`npm install @davmixcool/qdrant`

Then you can use it in your project:

```javascript
const { Qdrant } = require("@davmixcool/qdrant")
const qdrant = new Qdrant("http://localhost:6333/");
```

## Quick Start

Here is a basic example that creates a client connection and adds a new collection `pretty_colors` to Qdrant.  It assumes the Qdrant docker is running at localhost:6333.  This quick start is also in the examples folder in this repository.

```javascript
const { Qdrant } = require("@davmixcool/qdrant")

const qdrant = new Qdrant("http://localhost:6333/");

const name = "pretty_colors";

/// -------------------------------------------------------------------------
/// Create the new collection with the name and schema
const schema = {
    "name":name,
    "vector_size": 3,
    "distance": "Cosine"
};
let create_result = await qdrant.create_collection(name,schema);
if (create_result.err) {
    console.error(`ERROR:  Couldn't create collection "${name}"!`);
    console.error(create_result.err);
} else {
    console.log(`Success! Collection "${name} created!"`);
    console.log(create_result.response);
}

/// -------------------------------------------------------------------------
/// Show the collection info as it exists in the Qdrant engine
let collection_result = await qdrant.get_collection(name);
if (collection_result.err) {
    console.error(`ERROR:  Couldn't access collection "${name}"!`);
    console.error(collection_result.err);
} else {
    console.log(`Collection "${name} found!"`);
    console.log(collection_result.response);
}

/// -------------------------------------------------------------------------
/// Upload some points - just five RGB colors
let points = [
    { "id": 1, "payload": {"color": "red"}, "vector": [0.9, 0.1, 0.1] },
    { "id": 2, "payload": {"color": "green"}, "vector": [0.1, 0.9, 0.1] },
    { "id": 3, "payload": {"color": "blue"}, "vector": [0.1, 0.1, 0.9] },
    { "id": 4, "payload": {"color": "purple"}, "vector": [1.0, 0.1, 0.9] },
    { "id": 5, "payload": {"color": "cyan"}, "vector": [0.1, 0.9, 0.8] }
]
let upload_result = await qdrant.upload_points(name,points);
if (upload_result.err) {
    console.error(`ERROR:  Couldn't upload to "${name}"!`);
    console.error(upload_result.err);
} else {
    console.log(`Uploaded to "${name} successfully!"`);
    console.log(upload_result.response);
}

/// -------------------------------------------------------------------------
/// Search the closest color (k=1)
let vector = [0.8,0.1,0.7];
let search_result = await qdrant.search_collection({name,vector,1});
if (search_result.err) {
    console.error(`ERROR: Couldn't search ${vector}`);
    console.error(search_result.err);
} else {
    console.log(`Search results for ${vector}`);
    console.log(search_result.response);
}


/// -------------------------------------------------------------------------
/// Filtered search the closest color
let filter = {
    "must": [
        { "key": "color", "match": { "keyword": "cyan" } }
    ]
}
let filtered_result = await qdrant.search_collection({name,vector,k:1,ef:128,filter});
if (filtered_result.err) {
    console.error(`ERROR: Couldn't search ${vector} with ${filter}`);
    console.error(filtered_result.err);
} else {
    console.log(`Search results for filtered ${vector}`);
    console.log(filtered_result.response);
}

/// -------------------------------------------------------------------------
/// Delete the collection
let delete_result = await qdrant.delete_collection(name);
if (delete_result.err) {
    console.error(`ERROR:  Couldn't delete "${name}"!`);
    console.error(delete_result.err);
} else {
    console.log(`Deleted "${name} successfully!"`);
    console.log(delete_result.response);
}

```

## Conventions

All methods must be awaited, and return a QdrantResponse object - which only has two properties: `err` and `response`.

Always check for presence of `err`.  If `err` is not null, then the response might not be valid.

## Methods

With an qdrant object, just await one of the following methods to interact with the engine and its collections:

### `create_collection(collection_name,body)`

Creates a new collection with `collection_name` and the schema specified in `body`

### `get_collection(collection_name)`

Gets the collection information for `collection_name`

### `delete_collection(collection_name)`

Deletes a collection with `collection_name`

### `create_collection_index(collection_name,body)`

Creates a new collection index with `collection_name` and the schema specified in `body`

### `delete_collection_index(collection_name,field_name)`

Deletes a collection index with `collection_name` and `field_name`

### `upload_points(collection_name,points)`

Uploads vectors and payloads in `points` to the collection `collection_name`

### `delete_points(collection_name,points)`

Delete `points` in a collection `collection_name`

### `update_points(collection_name,points,payload)`

Update `points` payload in a collection `collection_name`

### `search_collection({name,vector,k,ef,filter,exact,indexed_only})`

Searches the collection with a `vector`, to get the top `k` most similar points (default 5), using HNSW `ef` (default is 128), and an optional payload `filter`, `exact`, and `indexed_only` params.

`exact` - option to not use the approximate search (ANN). If set to true, the search may run for a long as it performs a full scan to retrieve exact results.

`indexed_only` - With this option you can disable the search in those segments where vector index is not built yet. This may be useful if you want to minimize the impact to the search performance whilst the collection is also being updated. Using this option may lead to a partial result if the collection is not fully indexed yet, consider using it only if eventual consistency is acceptable for your use case.

### `query_collection(collection_name,query)`

Searches the collection with a `query` that must be fully defined by the caller.

### `retrieve_points(collection_name,query)`

Gets all the points by the array of ids provided