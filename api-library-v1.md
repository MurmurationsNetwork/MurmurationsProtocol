# Murmurations Library API

> :construction: PROPOSED BASE URL
>
> https://library.murmurations.network/v1/{endpoint}

The purpose of the Library API is to replicate the collection of fields and schemas so that they are stored in multiple locations (i.e., removes the GitHub library repo as a single point of failure), and ideally very close (from a networking point of view) to the indices and node UIs that are accessing them.

The service running the Library API will regularly synchronize with the GitHub library repo to ensure it is up-to-date with the list of fields and schemas on the master branch. It will respond to requests from indices and node UIs for schemas and field definitions so that they can validate profiles, and so that the node UI can build a form to create a profile for a node.

## Index & Node UI Endpoints

### `GET /fields`

#### Input

- A standard GET request with no additional parameters

> :construction: OPTIMIZATION WITH PARAMETERS
>
> When the list of fields starts to become large, we can add parameters for searching by field name, fields included in a specific schema, date/time last updated, etc.

#### Output

- JSON array of field names and their definitions

```json
[
  {
    "name": "mission-v1",
    "definition": {
      "title": "Mission Statement",
      "description": "A short statement of why the entity exists, what its overall goal is: what kind of product or service it provides, its primary customers or market, and its geographical region of operation.",
      "type": "string",
      "maxLength": 1000,
      "context": "https://en.wikipedia.org/wiki/Mission_statement"
    }
  },
  {
    "name": "tags-v1",
    "definition": {
      "title": "Descriptive Tags",
      "description": "The most common tags used to describe the entity",
      "type": "array",
      "items": {
        "type": "string"
      },
      "minItems": 1,
      "maxItems": 50,
      "uniqueItems": true,
      "context": "https://schema.org/keywords"
    }
  }
]
```

### `GET /schemas`

#### Input

- A standard GET request with no additional parameters

> :construction: OPTIMIZATION WITH PARAMETERS
>
> When the list of schemas starts to become large, we can add parameters for searching by schema name or ID, schemas using a specific field, date/time last updated, etc.

#### Output

- JSON array of schema names, their `schemaId`s and their definitions

```json
{
  "data": [
    {
      "name": "demo-v1.json",
      "id": "9FB93DC4C2556F7579AC2F1900DDA5ABD67905D3DA7293D89F90F01BF8188075",
      "definition": {
        "$schema": "https://json-schema.org/draft-04/schema#",
        "id": "https://raw.githubusercontent.com/MurmurationsNetwork/MurmurationsLibrary/master/",
        "title": "Demo Schema",
        "description": "A demo schema that is to be used only for testing purposes.",
        "type": "object",
        "properties": {
          "profileUrl": {
            "$ref": "fields/profileurl-v1.json"
          },
          "linkedSchemas": {
            "$ref": "fields/linkedschemas-v1.json"
          },
          "name": {
            "$ref": "fields/name-v1.json"
          },
          "url": {
            "$ref": "fields/url-v1.json"
          }
        },
        "required": [
          "profileUrl",
          "linkedSchemas",
          "name",
          "url"
        ],
        "metadata": {
          "owner": {
            "name": "Mumurmations Network",
            "url": "https://murmurations.network"
          }
        }
      }
    }
  ]
}
```