# Murmurations Index API

> :construction: PROPOSED BASE URL
>
> https://index.murmurations.network/v1/{endpoint}

The Index API describes how nodes, using predefined schemas, add, update and delete their data in the index so that aggregators can discover them.

Schemas define collections of data fields that nodes can fill in to share information about themselves. Think of a schema as a form template and each instance of a completed form as a profile.

Nodes store their profiles on their website and then reference them in the index. The index validates that the profile meets the requirements of its associated schema(s). The index does not actually store the entire profile; it is only required to store the URL of the profile's location on the node's website (`profileUrl`) and the name(s) of the schema(s) that profile is/are based upon and must be validated against (`linkedSchemas`).

Using these two pieces of information, aggregators can then search the index for nodes with profiles that match specific schemas they want to use to create maps and directories, and the index returns a list of URLs (`profileUrl`s) for the matched nodes' profiles. Aggregators then download each node's profile data for use in their apps.

The index may (and usually will) also store additional data to enable aggregators to locate nodes based on other information in addition to the linked schemas. This additional information could/will include, for example:

- _Geolocation data_ - latitude/longitude of primary location of entity - enables searching for nodes within a geographical range (i.e., "100km from me")
- _Location data_ - town/city, country, etc. for searching based on map location
- _Entity type_ - Wikipedia/Wikidata URL that best describes the organization type - enables searching for specific types of entities (e.g., "food co-ops")


> :construction: INDEX SYNCING
>
> A `profileHash` is for future planning, and will also be stored for each node profile so that multiple indices can synchronize their lists of nodes. It will be a hash of the profile (the entire JSON object containing all node profile data) stored at the `profileUrl` that was submitted by the node. Nodes can compare `nodeId`s (the unique hash of a `profileUrl`) and then `profileHash`es between themselves. They will also store a Unix timestamp (`lastChecked`) of when they last accessed the `profileUrl` and created the `profileHash` for each `nodeId`. The index with the oldest timestamp should download the profile from the `profileUrl` and regenerate the `profileHash`. This is just a rough idea of index syncing; it will need to be thought through in a lot more detail, including thinking about edge cases.

## Node Endpoints

### `POST /nodes`

Node operators will call the `POST /nodes` endpoint to add their nodes to the index both when they first create a profile and to indicate when they have made changes to that profile.

They need to store their profile at a publicly accessible URL (`profileUrl`), and then submit the `profileUrl` along with the schema(s) (`linkedSchemas`) against which the profile must be validated.

> :warning: SCHEMA REQUIREMENT
>
> The `profileUrl` (string) and `linkedSchemas` (array of strings) properties must be included in every schema as required fields, so the library should never accept schemas and the index should never accept profiles without these two properties.

#### Input

- Profile that validates to a referenced schema (or list of schemas), available at a publicly accessible URL
    - The profile must be available at the `profileUrl` or it will not be recorded by the index
- One or more `schemaName`s (unique schema name within a namespace) against which the profile must be validated

```json
{
  "profileUrl": "https://node.site/optional-subdirectory/node-profile.json",
  "linkedSchemas": [
    "demo_schema-v1",
    "some_other_schema-v1"
  ]
}
```

#### Output

##### Success

- `nodeId` - hash of the `profileUrl`

```json
{
  "data": {
    "nodeId": "a55964aeaae9625dc2b8dbdb1c4ce0ed1e658483f44cf2be1a6479fe5e144d38"
  }
}
```

> :construction: HASH ALGORITHM
>
> The example above uses the SHA256 hashing algorithm which produces a 64-character output. The purpose of hashing the `profileUrl` is to make it easy to reference as a path parameter when requesting information about the node from the index (e.g., `GET /nodes/{nodeId}` as described below).

##### Error
- Error reason (e.g., `Failed validation with schema: {schemaName}`, `Profile not found at profileUrl: {profileUrl}`, etc.)

### `GET /nodes/{nodeId}`

The record of a node in the index's database can be in one of five possible states: `received`, `validated`, `validation_failed`, `posted` or `post_failed`. The node will only be discoverable in the index when it has the status of `posted`.

This endpoint enables the Node UI to get and present an update to the node operator as to the status of the node profile after it has been submitted to the index (e.g., when using `POST /nodes`).

#### Input

- the `nodeId` of the node's profile that is currently in the index

#### Output

##### Success

- Confirmation of status (e.g., `received`, `validated` or `posted`)

###### Received
```json
{
  "data": {
    "profileUrl": "https://node.site/optional-subdirectory/node-profile.json",
    "nodeId": "a55964aeaae9625dc2b8dbdb1c4ce0ed1e658483f44cf2be1a6479fe5e144d38",
    "status": "received"
  }
}
```

###### Validated

```json
{
  "data": {
    "profileUrl": "https://node.site/optional-subdirectory/node-profile.json",
    "nodeId": "a55964aeaae9625dc2b8dbdb1c4ce0ed1e658483f44cf2be1a6479fe5e144d38",
    "lastChecked": 1601979232403,
    "status": "validated"
  }
}
```

###### Posted
```json
{
  "data": {
    "profileUrl": "https://node.site/optional-subdirectory/node-profile.json",
    "nodeId": "a55964aeaae9625dc2b8dbdb1c4ce0ed1e658483f44cf2be1a6479fe5e144d38",
    "lastChecked": 1601979232403,
    "status": "posted"
  }
}
```

###### Validation Failed
```json
{
  "data": {
    "profileUrl": "https://node.site/optional-subdirectory/node-profile.json",
    "nodeId": "a55964aeaae9625dc2b8dbdb1c4ce0ed1e658483f44cf2be1a6479fe5e144d38",
    "lastChecked": 1601979232403,
    "status": "validation_failed"
  }
}
```

###### Post Failed
```json
{
  "data": {
    "profileUrl": "https://node.site/optional-subdirectory/node-profile.json",
    "nodeId": "a55964aeaae9625dc2b8dbdb1c4ce0ed1e658483f44cf2be1a6479fe5e144d38",
    "lastChecked": 1601979232403,
    "status": "post_failed"
  }
}
```

##### Error
- Error reason (e.g., `nodeId/profileUrl not found in index`, etc.)

### `DELETE /nodes/{nodeId}`

Node operators will use the `DELETE /nodes/{nodeId}` endpoint to remove their profile from the index when they no longer want it listed.

#### Input

- the `nodeId` of the profile that is currently in the index
    - The profile must no longer be available at the `profileUrl` (i.e., should return a `404 - Not Found` error when accessing the URL) or it will not be removed from the index

#### Output

##### Success

- Confirmation of removal (e.g., `Removed from index: https://node.site/optional-subdirectory/my-profile.json`)

##### Error
- Error reason (e.g., `profileUrl not in index`, `profile still available at profileUrl`, etc.)

## Aggregator Endpoints

### `GET /nodes`

The `GET /nodes` endpoint requires an API key in order to prevent unauthorized harvesting of the data in the index.

Aggregators can search for nodes based on the schema(s) nodes use, when the nodes were last validated by the index (i.e., for recent changes to node profiles), and by geolocating nodes within a certain (kilometer) range from a specific location or finding them based on the town/city, country, etc.

It is envisioned that other search parameters will be added to this endpoint as they are defined and deemed useful for aggregator searching.

> :construction: SEARCH BY ORGANIZATION TYPE
>
> We need to offer the ability to search by type of organization, but the methodology for defining and recording the organization type still needs to be determined ([see this issue for updates](https://github.com/MurmurationsNetwork/MurmurationsProtocol/issues/6)).

#### Input

- `schemaName` - unique schema name (only allow a single value per search)
- `lastChecked` - Unix timestamp (in milliseconds)
- `geolocation` - `latitude`, `longitude` & `range`
- `mapAddress` - `locality`, `region`, `country`

#### Output

- Array of nodes with:
    - `profileUrl`
    - `lastChecked` (in milliseconds)
    - `geolocation` - an object containing `latitude` & `longitude`
    - `mapAddress` - an object containing `locality`, `region` & `country`
