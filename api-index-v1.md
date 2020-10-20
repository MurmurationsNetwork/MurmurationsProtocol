# Murmurations Index API

> :construction: PROPOSED BASE URL
>
> https://index.murmurations.network/v1/{endpoint}

The Index API describes how nodes, using predefined schemas, add, update and delete their data in the index so that aggregators can discover them.

Schemas define collections of data fields that nodes can fill in to share information about themselves. Think of a schema as a form template and each instance of a completed form as a profile.

Nodes store their profiles on their website and then reference them in the index. The index validates that the profile meets the requirements of its schema(s). The index does not actually store the entire profile; it is only required to store the URL of the profile's location on the node's website (`profileUrl`) and the schema(s) that profile is/are based upon and must be validated against (`linkedSchemas`).

Using these two pieces of information, aggregators can then search the index for nodes with profiles that match certain schemas (`linkedSchemas`) they want to use to create maps and directories, and the index returns a list of URLs (`profileUrl`s) for the matched nodes' profiles. Aggregators then download each node's data from the `profileUrl` for use in their apps.

The index may (and usually will - future plans include multiple Indices; see Index Syncing below) also store additional data to enable aggregators to locate nodes based on other information in addition to the linked schemas. This additional information could/will include, for example:

- _Geolocation data_ - latitude/longitude of primary location of entity - enables searching for nodes within a geographical range (i.e., "100km from me")
- _Entity type_ - Wikipedia/Wikidata URL that best describes the organization type - enables searching for specific types of entities (e.g., "food co-ops")

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
  "profileUrl": "https://node.site/optional-subdirectory/my-profile.json",
  "linkedSchemas": [
    "demo_schema-v1",
    "some_other_schema-v1"
  ]
}
```

> :construction: HASH ALGORITHM
>
> The example above uses the SHA256 hashing algorithm which produces a 64-character output. We should research if this is the best algorithm to use (e.g., should we use SHA512 instead?).

#### Output

##### Success

- `nodeId` - hash of profile located at `profileUrl`
- `lastChecked` - Unix timestamp (in milliseconds)

TODO: Add example

##### Error
- Error reason (e.g., `Failed validation with schema: {schemaName}`, `Profile not found at profileUrl: {profileUrl}`, etc.)

> :construction: INDEX SYNCING
>
> A `nodeId` is for future planning, so that multiple indices can synchronize their lists of nodes. It will be a hash of the profile stored at the `profileUrl` that was submitted by the node. Nodes can compare `profileUrl`s and then `nodeId`s between themselves. They will also store a Unix timestamp (`lastChecked`) of when they obtained the `nodeId` for each `profileUrl`. The index with the oldest timestamp should download the profile from the `profileUrl` and regenerate the `nodeId`. This is just a rough idea of index syncing; it will need to be thought through in a lot more detail, including thinking about edge cases.
> 

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
