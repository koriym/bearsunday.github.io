---
layout: docs-en
title: Validation
category: Manual
permalink: /manuals/1.0/en/validation.html
---

# Validation

* BEAR.Sunday validates using JSON Schema.
* For web form validation, see [Form](form.html).

## JSON Schema Validation

### Overview

You can define and validate resource API input/output specifications using [JSON Schema](http://json-schema.org/). This allows managing API specifications in a format that both humans and machines can understand. You can also output API documentation as ApiDoc.

### Setup

#### Module Configuration

Configure according to the scope of validation:

- To validate in all environments: Configure in `AppModule`
- To validate only in development environment: Configure in `DevModule`

```php
use BEAR\Resource\Module\JsonSchemaModule;
use BEAR\Package\AbstractAppModule;

class AppModule extends AbstractAppModule
{
    protected function configure(): void
    {
        $this->install(
            new JsonSchemaModule(
                $appDir . '/var/json_schema',  // For schema definitions
                $appDir . '/var/json_validate' // For validation
            )
        );
    }
}
```

#### 2. Create Required Directories

```bash
mkdir -p var/json_schema
mkdir -p var/json_validate
```

### Basic Usage

#### 1. Resource Class Definition

```php
use BEAR\Resource\Annotation\JsonSchema;

class User extends ResourceObject
{
    #[JsonSchema('user.json')]
    public function onGet(): static
    {
        $this->body = [
            'firstName' => 'mucha',
            'lastName' => 'alfons',
            'age' => 12
        ];
        return $this;
    }
}
```

#### 2. JSON Schema Definition

`var/json_schema/user.json`:

```json
{
    "type": "object",
    "properties": {
        "firstName": {
            "type": "string",
            "maxLength": 30,
            "pattern": "[a-z\\d~+-]+"
        },
        "lastName": {
            "type": "string",
            "maxLength": 30,
            "pattern": "[a-z\\d~+-]+"
        }
    },
    "required": ["firstName", "lastName"]
}
```

### Advanced Usage

#### Specifying Index Key

When the response body has an index key, specify it with the `key` parameter:

```php
class User extends ResourceObject
{
    #[JsonSchema(key: 'user', schema: 'user.json')]
    public function onGet(): static
    {
        $this->body = [
            'user' => [
                'firstName' => 'mucha',
                'lastName' => 'alfons',
                'age' => 12
            ]
        ];
        
        return $this;
    }
}
```

#### Argument Validation

To validate method arguments, specify the schema with the `params` parameter:

```php
class Todo extends ResourceObject
{
    #[JsonSchema(
        key: 'user',
        schema: 'user.json',
        params: 'todo.post.json'
    )]
    public function onPost(string $title)
    {
        // Method processing
    }
}
```

`var/json_validate/todo.post.json`:
```json
{
    "$schema": "http://json-schema.org/draft-04/schema#",
    "title": "/todo POST request validation",
    "properties": {
        "title": {
            "type": "string",
            "minLength": 1,
            "maxLength": 40
        }
    }
}
```

### target

To apply schema validation to the representation of the resource object (the rendered result) rather than to the body of the ResourceObject, specify the `target='view'` option. This allows describing `_link` schema in HAL format.

```php
#[JsonSchema(schema: 'user.json', target: 'view')]
```

### Schema Creation Tools

The following tools are helpful for creating JSON schemas:

- [JSON Schema Generator](https://jsonschema.net/#/editor)
- [Understanding JSON Schema](https://spacetelescope.github.io/understanding-json-schema/)
