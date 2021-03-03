# Roadmap

## Release 1

- Further design and document the Murmurations Protocol
  - **DONE** - Define the library structure
  - **DONE** - Define JSON Schema validation of fields and schemas
  - **DONE** - Document the protocol, APIs & DevOps
  - **DONE** - Implement production environment for Index, Library & Profile Generator
- Rebuild Index service
  - **DONE** - Draft REST API for Index service
  - **DONE** - Write service in Go
- Build Library service (sync with library git repo to serve schemas and fields directly to Index and Node UI)
  - **DONE** - Define REST API for Library service
  - **DONE** - Write service in Go
- Rebuild Node UI
  - **DONE** - Leverage JSON Forms to generate forms from schemas
  - **DONE** - Implement Profile Generator to enable users to create/update/delete node profiles

## Release 2

- Rebuild demo Aggregators (map, directory, RSS feed)
  - **IN PROGRESS** - Adapt apps to consume new schema format
- Rebuild WordPress plugin
  - Wrap JSON Forms React app
- Index service
  - Require authentication for aggregators
  - Define and write tests for failure, validation & concurrency scenarios
- Library service
  - Define and write tests for failure, validation & concurrency scenarios
