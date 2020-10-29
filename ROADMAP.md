# Roadmap

## Release 1

- Further design and document the Murmurations Protocol
  - **DONE** - Define the library structure
  - **IN PROGRESS** - Define JSON Schema validation of fields and schemas
  - Document the protocol, APIs & DevOps
- Rebuild Index service
  - **IN PROGRESS** - Define REST API for Index service
  - **IN PROGRESS** - Use JSON Schema for one-to-one schema/profile validation
  - Use JSON Schema for many-to-one schemas/profile validation
  - Define and write tests for failure, validation & concurrency scenarios
- Rebuild Library service
  - Define REST API for Library service
  - Sync library git repo to serve schemas and fields directly to Index and Node UI
- Rebuild Node UI
  - **IN PROGRESS** - Use vanilla JS/HTML/CSS to enable wrapping in CMS plug-ins
  - **IN PROGRESS** - Leverage JSON Editor to generate forms from schemas
  - **IN PROGRESS** - Use JSON Schema for one-to-one schema/profile validation
  - Use JSON Schema for many-to-one schemas/profile validation
- Rebuild demo Aggregators (map, directory, RSS feed)
  - Adapt apps to consume new schema format

## Release 2

- Index service
  - Require authentication for aggregators
