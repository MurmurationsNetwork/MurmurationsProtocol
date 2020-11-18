# Roadmap

## Release 1

- Further design and document the Murmurations Protocol
  - **DONE** - Define the library structure
  - **DONE** - Define JSON Schema validation of fields and schemas
  - **IN PROGRESS** - Document the protocol, APIs & DevOps
- Rebuild Index service
  - **DONE** - Draft REST API for Index service
  - **IN PROGRESS** - Write service in Go
  - Define and write tests for failure, validation & concurrency scenarios
- Build Library service (sync with library git repo to serve schemas and fields directly to Index and Node UI)
  - **IN PROGRESS** - Define REST API for Library service
  - Write service in Go
  - Define and write tests for failure, validation & concurrency scenarios
- Rebuild Node UI
  - **IN PROGRESS** - Use JS/HTML/CSS to enable wrapping in CMS plug-ins (using React for prototype)
  - **IN PROGRESS** - Leverage JSON Forms to generate forms from schemas
- Rebuild demo Aggregators (map, directory, RSS feed)
  - Adapt apps to consume new schema format

## Release 2

- Index service
  - Require authentication for aggregators
