# Posting a node to an index

The following sequence diagram illustrates the series of steps taken between various services when posting a node to an index.

_Index Service_, _Validation Service_, _Library Service_ and _Elasticsearch_ are all services that are orchestrated inside of the [Murmurations Services](https://github.com/MurmurationsNetwork/MurmurationsServices) repository when it is instantiated.

The _Node Owner_ is the individual or organization operating the _Node Server_ that hosts the node's [profile](https://docs.murmurations.network/about/common-terms.html#profile).

```mermaid
sequenceDiagram
    participant Node Owner
    participant Node Server
    participant Index Service
    participant Validation Service
    participant Library Service
    participant Elasticsearch

    Node Owner->>Node Server: Post profile to profile_url
    Node Owner->>+Index Service: Submit profile_url (POST /nodes)
    Index Service->>Index Service: Add node record to index DB and <br> create node_id (hash of profile_url)
    Note over Index Service: status: received
    Index Service->>-Node Owner: Confirm node recorded in DB by returning node_id
    Index Service-x+Validation Service: node:created
    Validation Service->>Node Server: Check if profile at profile_url
    alt check failed
    Node Server-->>Validation Service: Profile not found or node unreachable
    Note over Index Service: status: validation_failed
    else check success
    Node Server-->>Validation Service: Return JSON profile data
    end
    Validation Service->>Library Service: Get schema(s) & fields
    Library Service-->>Validation Service: Return schema(s) & fields
    Validation Service->>Validation Service: Validate profile to schema(s) & fields
    alt validate failed
    Validation Service-xIndex Service: node:validation_failed
    Note over Index Service: status: validation_failed
    else validate success
    Validation Service-x-Index Service: node:validated
    Index Service->>Index Service: Set last_updated (Unix timestamp) and <br> profile_hash (hash of JSON profile data) in DB
    Note over Index Service: status: validated
    end
    Index Service->>Elasticsearch: Post data to Elasticsearch
    alt post failed
    Note over Elasticsearch: No response
    Note over Index Service: status: post_failed
    else post success
    Elasticsearch-->>Index Service: Return confirmation
    Note over Index Service: status: posted
    end
    Node Owner->>+Index Service: Poll for status
    Index Service->>Index Service: Get status from node record
    Index Service->>-Node Owner: Return profile_url, node_id, last_updated, status
```
