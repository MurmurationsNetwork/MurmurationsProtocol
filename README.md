# MurmurationsProtocol

The Murmurations Protocol enables **nodes** (organizations and individuals) to easily distribute information about themselves to **aggregators**, who create **schemas** to define the data they need to create maps and directories.

A **library** records each schema and the **fields** (data points) it is composed of. A node pulls a schema from the library to determine the data needed to create a **profile**.

An **index** keeps track of nodes and the schemas linked to their profiles. Whenever a node updates its profile it tells the index. Aggregators regularly query the index for profile changes by nodes using their schemas, enabling them to provide accurate and timely information in their maps and directories.
