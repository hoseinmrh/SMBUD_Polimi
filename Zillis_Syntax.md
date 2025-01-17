# Zilliz Cheatsheet

This cheatsheet covers the essential syntax and operations for working with Zilliz using the `pymilvus` library. It includes collection management, data insertion, search operations, and more.

---

## 1. **Collection Management**

### 1.1. **Create a Collection**

```python
from pymilvus import MilvusClient

client = MilvusClient(
    uri="CLUSTER_ENDPOINT",  # Cluster endpoint
    token="TOKEN"            # API key or username:password
)

# Create a collection with a simple setup
client.create_collection(
    collection_name="test_collection",
    dimension=5  # Dimension of the vector
)
```

### 1.2. **Drop a Collection**

```python
client.drop_collection(
    collection_name="test_collection"
)
```

### 1.3. **List Collections**

```python
collection_list = client.list_collections()
print(collection_list)
```

### 1.4. **Describe a Collection**

```python
collection_description = client.describe_collection(
    collection_name="test_collection"
)
print(collection_description)
```

### 1.5. **Load and Release a Collection**

```python
# Load a collection
client.load_collection(
    collection_name="test_collection"
)

# Release a collection
client.release_collection(
    collection_name="test_collection"
)
```

---

## 2. **Data Insertion**

### 2.1. **Insert Data**

```python
data = [
    {"id": 1, "vector": [0.1, 0.2, 0.3, 0.4, 0.5], "color": "red"},
    {"id": 2, "vector": [0.6, 0.7, 0.8, 0.9, 1.0], "color": "blue"}
]

insert_status = client.insert(
    collection_name="test_collection",
    data=data
)
print(insert_status)
```

### 2.2. **Upsert Data**

```python
upsert_status = client.upsert(
    collection_name="test_collection",
    data=data  # Data to upsert
)
print(upsert_status)
```

### 2.3. **Delete Data**

```python
delete_status = client.delete(
    collection_name="test_collection",
    filter="id in [1, 2]"  # Filter to delete specific entities
)
print(delete_status)
```

---

## 3. **Search Operations**

### 3.1. **Single Vector Search**

```python
query_vector = [0.1, 0.2, 0.3, 0.4, 0.5]

search_result = client.search(
    collection_name="test_collection",
    data=[query_vector],
    limit=3,  # Number of results to return
    search_params={"metric_type": "IP", "params": {"level": 1}}
)
print(search_result)
```

### 3.2. **Batch Vector Search**

```python
bulk_query_vectors = [
    [0.1, 0.2, 0.3, 0.4, 0.5],
    [0.6, 0.7, 0.8, 0.9, 1.0]
]

bulk_search_result = client.search(
    collection_name="test_collection",
    data=bulk_query_vectors,
    limit=2,
    search_params={"metric_type": "IP", "params": {"level": 1}}
)
print(bulk_search_result)
```

### 3.3. **Partition Search**

```python
partition_search_result = client.search(
    collection_name="test_collection",
    data=[query_vector],
    limit=5,
    search_params={"metric_type": "IP", "params": {"level": 1}},
    partition_names=["partition_1"]
)
print(partition_search_result)
```

### 3.4. **Filtered Search**

```python
filtered_search_result = client.search(
    collection_name="test_collection",
    data=[query_vector],
    limit=5,
    search_params={"metric_type": "IP", "params": {"level": 1}},
    filter='color == "red"'
)
print(filtered_search_result)
```

---

## 4. **Partition Management**

### 4.1. **Create a Partition**

```python
client.create_partition(
    collection_name="test_collection",
    partition_name="partition_1"
)
```

### 4.2. **List Partitions**

```python
partitions = client.list_partitions(
    collection_name="test_collection"
)
print(partitions)
```

### 4.3. **Drop a Partition**

```python
client.drop_partition(
    collection_name="test_collection",
    partition_name="partition_1"
)
```

### 4.4. **Load and Release a Partition**

```python
# Load a partition
client.load_partitions(
    collection_name="test_collection",
    partition_names=["partition_1"]
)

# Release a partition
client.release_partitions(
    collection_name="test_collection",
    partition_names=["partition_1"]
)
```

---

## 5. **Advanced Search Operations**

### 5.1. **Hybrid Search**

```python
from pymilvus import AnnSearchRequest, WeightedRanker

# Create ANN search requests
request_film = AnnSearchRequest(
    data=[[0.1, 0.2, 0.3, 0.4, 0.5]],
    anns_field="filmVector",
    param={"metric_type": "L2", "params": {"nprobe": 10}},
    limit=2
)

request_poster = AnnSearchRequest(
    data=[[0.6, 0.7, 0.8, 0.9, 1.0]],
    anns_field="posterVector",
    param={"metric_type": "L2", "params": {"nprobe": 10}},
    limit=2
)

# Perform hybrid search
rerank = WeightedRanker(0.8, 0.2)  # Weighted ranker
search_results = collection.hybrid_search(
    requests=[request_film, request_poster],
    rerank=rerank,
    limit=2
)
print(search_results)
```

---

## 6. **Query Operations**

### 6.1. **Get Entities by ID**

```python
entities = client.get(
    collection_name="test_collection",
    ids=[1, 2]
)
print(entities)
```

### 6.2. **Basic Filtering Queries**

```python
query_result = client.query(
    collection_name="test_collection",
    filter='color == "red"',
    output_fields=["color"],
    limit=3
)
print(query_result)
```

### 6.3. **Advanced Queries**

```python
advanced_query_result = client.query(
    collection_name="test_collection",
    filter='color == "red" and tag > 1000',
    output_fields=["color_tag"],
    limit=3
)
print(advanced_query_result)
```

---

## 7. **Index Management**

### 7.1. **Create an Index**

```python
index_params = client.prepare_index_params()
index_params.add_index(
    field_name="vector",
    index_type="AUTOINDEX",
    metric_type="IP",
    params={"nlist": 128}
)

client.create_index(
    collection_name="test_collection",
    index_params=index_params
)
```

### 7.2. **List Indexes**

```python
index_list = client.list_indexes(
    collection_name="test_collection"
)
print(index_list)
```

### 7.3. **Drop an Index**

```python
client.drop_index(
    collection_name="test_collection",
    index_name="vector_index"
)
```
