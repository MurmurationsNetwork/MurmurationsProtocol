# Murmurations Library API

The purpose of the Library API is to:

1. replicate the collection of fields and schemas so that they are stored in at least one more location (i.e., removes the GitHub library repo as a single point of failure), and ideally very close (from a networking point of view) to the index and Murmurations profile generators (MPGs) that are accessing them (via a CDN), and
2. provide a list of schemas and fields along with their version numbers (to find the latest version of a schema) and other metadata that will be useful for presenting them to users creating node profiles in the MPG.

The service running the Library API will regularly synchronize with the GitHub library repo to ensure it is up-to-date with the list of fields and schemas on the master branch. It will respond to requests from the index and MPGs for schemas and field definitions so that they can validate profiles, and so that the MPG can build forms to create profiles for a node.

## Raw Data Endpoints

Schemas and fields can be downloaded directly from the GitHub library repo by using GitHub's `raw.githubusercontent.com` CDN. For example, the demo schema can be accessed at:

`https://raw.githubusercontent.com/MurmurationsNetwork/MurmurationsLibrary/master/schemas/demo-v1.json`

And the `name` field referenced in the above schema can be accessed at:

`https://raw.githubusercontent.com/MurmurationsNetwork/MurmurationsLibrary/master/fields/name-v1.json`

The library service will mimic this functionality by serving these schemas and fields from the following URLs over HTTPS, for example:

```
https://cdn.murmurations.network/schemas/demo-v1.json
https://cdn.murmurations.network/fields/name-v1.json
```


## Index & Node UI Endpoints

> :construction: PROPOSED API BASE URL
>
> https://library.murmurations.network/v1/{endpoint}

### `GET /schemas`

#### Input

- A standard GET request with no additional parameters

> :construction: OPTIMIZATION WITH PARAMETERS
>
> When the list of schemas starts to become large, we can add parameters for searching by schema name or ID, schemas using a specific field, date/time last updated, etc.

#### Output

- JSON array of schemas with some basic information about them

```json
{
  "data": [
    {
      "title": "Demo Schema",
      "description": "A demo schema that is to be used only for testing purposes.",
      "name": "demo-v1",
      "version": 1,
      "url": "http://schema-creator.site/page-about/schema.html"
    }
  ]
}
```

- `title` is the same as defined in the schema
- `description` is the same as defined in the schema
- `version` is an integer and must match the version number (`v1`, `v2`, etc.) in the filename (`name`) of the schema
- `name` is the filename (without the `.json` extension) of the schema as stored in the library
- `url` is a link to a web page where one can learn more about the schema from its creators. The web page should include information about the schema's purpose, a CHANGELOG describing changes made between versions and any other relevant information that could be useful to the node operator who is considering to use the schema.

All of the information above is pulled from the schema as it is recorded in the library, either from its JSON Schema-related contents (`title` & `description`) or from its `metadata.schema` (`name`, `version`, `url`):

```json
{
  "$schema": "https://json-schema.org/draft-04/schema#",
  "id": "https://cdn.murmurations.network/schemas/demo-v1.json",
  "title": "Demo Schema",
  "description": "A demo schema that is to be used only for testing purposes.",
  "type": "object",
  "properties": {
    "linked_schemas": {
      "$ref": "../fields/linked_schemas-v1.json"
    },
    "name": {
      "$ref": "../fields/name-v1.json"
    },
    "url": {
      "$ref": "../fields/url-v1.json"
    },
    "mission": {
      "$ref": "../fields/mission-v1.json"
    },
    "geolocation": {
      "$ref": "../fields/geolocation-v1.json"
    },
    "keywords": {
      "$ref": "../fields/keywords-v1.json"
    }
  },
  "required": [
    "linked_schemas",
    "name",
    "url"
  ],
  "metadata": {
    "creator": {
      "name": "Murmurations Network",
      "url": "https://murmurations.network"
    },
    "schema": {
      "name": "demo-v1",
      "version": 1,
      "url": "https://murmurations.network/schemas/demo.html"
    }
  }
}
```

### `GET /fields`

#### Input

- A standard GET request with no additional parameters

> :construction: OPTIMIZATION WITH PARAMETERS
>
> When the list of fields starts to become large, we can add parameters for searching by field name, fields included in a specific schema, date/time last updated, etc.

#### Output

- JSON array of field names and their definitions

```json
{
  "data": [
    {
      "title": "Mission Statement",
      "description": "A short statement of why the entity exists, what its overall goal is: what kind of product or service it provides, its primary customers or market, and its geographical region of operation.",
      "type": "string",
      "minLength": 1,
      "maxLength": 1000,
      "metadata": {
        "creator": {
          "name": "Murmurations Network",
          "url": "https://murmurations.network"
        },
        "field": {
          "name": "mission-v1",
          "version": 1
        },
        "context": ["https://en.wikipedia.org/wiki/Mission_statement"]
      }
    },
    {
      "title": "Descriptive Keywords",
      "description": "The most common keywords used to describe the entity",
      "type": "array",
      "items": {
        "type": "string",
        "minLength": 1,
        "maxLength": 100
      },
      "minItems": 1,
      "maxItems": 50,
      "uniqueItems": true,
      "metadata": {
        "creator": {
          "name": "Murmurations Network",
          "url": "https://murmurations.network"
        },
        "field": {
          "name": "keywords-v1",
          "version": 1
        },
        "context": ["https://schema.org/keywords"]
      }
    },
    {
      "title": "Geolocation Coordinates",
      "description": "The geo-coordinates (latitude & longitude) of the primary location of the entity",
      "type": "object",
      "properties": {
        "lat": {
          "type": "number",
          "minimum": -90,
          "maximum": 90
        },
        "lon": {
          "type": "number",
          "minimum": -180,
          "maximum": 180
        }
      },
      "required": ["lat", "lon"],
      "metadata": {
        "creator": {
          "name": "Murmurations Network",
          "url": "https://murmurations.network"
        },
        "field": {
          "name": "geolocation-v1",
          "version": 1
        },
        "context": [
          "https://schema.org/latitude",
          "https://schema.org/longitude",
          "https://schema.org/GeoCoordinates"
        ]
      }
    }
  ]
}

```
