---
layout: docs-en
title: CLI Tutorial
category: Manual
permalink: /manuals/1.0/en/tutorial3.html
---
# BEAR.Sunday CLI Tutorial

## Prerequisites

- PHP 8.2 or higher
- Composer
- Git

## Step 1: Project Creation

### 1.1 Create a New Project

```bash
VENDOR=MyVendor PACKAGE=Greet composer create-project bear/skeleton greet
cd greet
```

### 1.2 Verify Development Server

```bash
php -S 127.0.0.1:8080 -t public
```

Access [http://127.0.0.1:8080](http://127.0.0.1:8080) in your browser and confirm that "Hello BEAR.Sunday" is displayed.

```json
{
    "greeting": "Hello BEAR.Sunday",
    "_links": {
        "self": {
            "href": "/index"
        }
    }
}
```

## Step 2: Install BEAR.Cli

```bash
composer require bear/cli
```

## Step 3: Create Greeting Resource

Create `src/Resource/Page/Greeting.php`:

```php
<?php

namespace MyVendor\Greet\Resource\Page;

use BEAR\Cli\Attribute\Cli;
use BEAR\Cli\Attribute\Option;
use BEAR\Resource\ResourceObject;

class Greeting extends ResourceObject
{
    #[Cli(
        name: 'greet',
        description: 'Multilingual greeting command',
        output: 'greeting'
    )]
    public function onGet(
        #[Option(shortName: 'n', description: 'Name to greet')]
        string $name,
        #[Option(shortName: 'l', description: 'Language (en, ja, fr, es)')]
        string $lang = 'en'
    ): static {
        $greeting = match ($lang) {
            'ja' => 'こんにちは',
            'fr' => 'Bonjour',
            'es' => '¡Hola',
            default => 'Hello',
        };
        
        $this->body = [
            'greeting' => "{$greeting}, {$name}",
            'lang' => $lang
        ];

        return $this;
    }
}
```

## Step 4: Verify as Web Resource

Access the following URL in your browser to verify operation:

```
http://127.0.0.1:8080/greeting?name=World&lang=fr
```

You should see a JSON response like this:

```json
{
   "greeting": "Bonjour, World",
   "lang": "fr",
   "_links": {
      "self": {
         "href": "/greeting?name=World&lang=fr"
      }
   }
}
```

## Step 5: Generate CLI Command

```bash
vendor/bin/bear-cli-gen MyVendor.Greet
```

This generates the following files:
- `bin/cli/greet`: Executable CLI command

## Step 6: Test the Command

Test the generated command:

```bash
# Grant execute permission
chmod +x bin/cli/greet

# Display help
./bin/cli/greet --help

# Basic greeting
./bin/cli/greet -n "World"
# Output: Hello, World

# Japanese greeting
./bin/cli/greet -n "世界" -l ja
# Output: こんにちは, 世界

# JSON format output
./bin/cli/greet -n "World" -l ja --format json
# Output: {"greeting": "こんにちは, World", "lang": "ja"}
```

## Step 7: Test Homebrew Formula Locally

### 7.1 Generate Formula

To generate a formula, the Git repository must be initialized:

```bash
# Initialize Git repository (if not already done)
git init
git add .
git commit -m "Initial commit"
```

Generate the formula:

```bash
vendor/bin/bear-cli-gen MyVendor.Greet
```

This generates the following files:
- `bin/cli/greet`: Executable CLI command
- `var/homebrew/greet.rb`: Homebrew formula (if Git repository is configured)

### 7.2 Test Local Homebrew Installation

You can test the generated formula locally:

```bash
# Install locally using the formula
brew install --formula ./var/homebrew/greet.rb

# Test the installed command
greet -n "Homebrew" -l ja
# Output: こんにちは, Homebrew

# Uninstall
brew uninstall greet
```

## Optional: About Public Distribution

If you want to distribute your CLI tool to others, you can publish it as a Homebrew package:

1. Push your application to GitHub
2. Publish the generated formula (`var/homebrew/greet.rb`) in a GitHub repository with a `homebrew-` prefix
3. Users can install with `brew tap your-vendor/greet && brew install greet`

For detailed publishing procedures, see the [Homebrew official documentation](https://docs.brew.sh/How-to-Create-and-Maintain-a-Tap).

**Note**: The following conditions are required for formula generation:
- The application's Git repository must be initialized
- GitHub remote repository is not required for local testing

If these conditions are not met, formula generation will be skipped and the reason will be displayed.

## Summary

This tutorial demonstrated more than just CLI tool creation—it revealed the essential value of BEAR.Sunday:

### The True Value of Resource-Oriented Architecture

**Same Resource, Multiple Boundaries**
- The `Greeting` resource functions as Web API, CLI, and Homebrew package with a single implementation
- No duplication of business logic, maintenance in one place

### Boundary-Crossing Framework

BEAR.Sunday functions as a **boundary framework**, transparently handling:

- **Protocol boundaries**: HTTP ↔ Command line
- **Interface boundaries**: Web ↔ CLI ↔ Package distribution
- **Environment boundaries**: Development ↔ Production ↔ User environments

### Design Philosophy in Action

```php
// One resource
class Greeting extends ResourceObject {
    public function onGet(string $name, string $lang = 'en'): static
    {
        // Business logic in one place
    }
}
```

↓

```bash
# As Web API
curl "http://localhost/greeting?name=World&lang=ja"

# As CLI
./bin/cli/greet -n "World" -l ja

# As Homebrew package
brew install your-vendor/greet && greet -n "World" -l ja
```

### Long-term Maintainability and Productivity

- **DRY Principle**: Domain logic is not coupled with interfaces
- **Unified Testing**: Testing one resource covers all boundaries
- **Consistent API Design**: Same parameter structure for Web API and CLI
- **Future Extensibility**: New boundaries (gRPC, GraphQL, etc.) can use the same resource
- **PHP Version Independence**: Freedom to continue using what works

### Integration with Modern Distribution Systems

BEAR.Sunday resources integrate naturally with modern package systems. By leveraging package managers like Homebrew and the Composer ecosystem, users can utilize tools through unified interfaces without being aware of the execution environment.

BEAR.Sunday's "Because Everything is a Resource" is not just a slogan, but a design philosophy that realizes consistency and maintainability across boundaries. As experienced in this tutorial, resource-oriented architecture creates boundary-free software and brings new horizons to both development and user experiences.
