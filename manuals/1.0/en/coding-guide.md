---
layout: docs-en
title: Coding Guide
category: Manual
permalink: /manuals/1.0/en/coding-guide.html
---

# Coding Guide

## Project

`vendor` should be the company name, team name, or owner name (`excite`, `koriym`, etc.). `package` is the name of the application or service (`blog`, `news`, etc.). Projects are created on a per-application basis. Even when serving Web API and HTML from different hosts, they are considered one project.

## Style

Follow [PSR1](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md), [PSR2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md), [PSR4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md).

```php
<?php
namespace Koriym\Blog\Resource\App;

use BEAR\RepositoryModule\Annotation\Cacheable;
use BEAR\Resource\Annotation\Embed;
use BEAR\Resource\Annotation\Link;
use BEAR\Resource\Code;
use BEAR\Resource\ResourceObject;

#[CacheableResponse]
class Entry extends ResourceObject
{
    public function __construct(
        private readonly ExtendPdoInterface $pdo,
        private readonly ResourceInterface $resource
    ) {}

    #[Embed(rel: "author", src: "/author{?author_id}")]
    public function onGet(string $author_id, string $slug): static
    {
        // ...
        return $this;
    }

    #[Link(rel: "next_action1", href: "/next_action1")]
    public function onPost(
        string $title,
        string $body,
        string $uid,
        string $slug
    ): static {
        // ...
        $this->code = Code::CREATED;
        return $this;
    }
}
```

Resource [DocBlock comments](https://phpdoc.org/docs/latest/getting-started/your-first-set-of-documentation.html) are optional. Add a method summary (one line), description (multiple lines allowed), and `@params` when the resource URI or argument names alone are insufficient explanation.

```php
/**
 * A summary informing the user what the associated element does.
 *
 * A *description*, that can span multiple lines, to go _in-depth_ into the details of this element
 * and to provide some background information or textual references.
 *
 * @param string $arg1 *description*
 * @param string $arg2 *description*
*/
```

## Resources

See [Resource Best Practices](resource_bp.html) for resource best practices.

### Code

Return appropriate status codes. This makes testing easier and conveys correct information to bots and crawlers.

* `100` Continue - Continuation of multiple requests
* `200` OK
* `201` Created - Resource creation
* `202` Accepted - Queue/batch acceptance
* `204` No Content - When there is no body
* `304` Not Modified - No update
* `400` Bad Request - Request has issues
* `401` Unauthorized - Authentication required
* `403` Forbidden - Prohibited
* `404` Not Found
* `405` Method Not Allowed
* `503` Service Unavailable - Temporary server-side error

`304` is automatically set when using the `#[Cacheable]` attribute. `404` is automatically set when the resource class doesn't exist, and `405` when the resource method doesn't exist. Also, always return `503` for DB connection errors to notify crawlers.

### HTML Form Methods

BEAR.Sunday can override methods using the `X-HTTP-Method-Override` header or `_method` query in `POST` requests from HTML web forms, but this is not necessarily recommended. For Page resources, implementing only `onGet` and `onPost` is acceptable.

### Hyperlinks

* Resources with links are recommended to indicate them with `#[Link]`.
* Resources are recommended to be embedded as semantic graphs using `#[Embed]`.

## Globals

Referencing global values in resources or application classes is not recommended. (Use only in Modules)

* Do not reference [superglobal](http://php.net/manual/en/language.variables.superglobals.php) values
* Do not use [define](http://php.net/manual/en/function.define.php)
* Do not create `Config` classes to hold configuration values
* Do not use global object containers (service locators)
* Directly getting current time with [date](http://php.net/manual/en/function.date.php) function or [DateTime](http://php.net/manual/en/class.datetime.php) class is not recommended[^now]. Inject time from outside.
* Global method calls such as static methods are also not recommended.
* All values needed by application code should be injected, not retrieved from configuration files.[^inject-all]

[^now]: [koriym/now](https://github.com/koriym/Koriym.Now)
[^inject-all]: When using values from external systems like Web APIs, centralize them in client classes or Web API access resources to make mocking easy with DI or AOP.

## Classes and Objects

* [Traits](http://php.net/manual/en/language.oop5.traits.php) are not recommended.[^no-trait]
* Child classes using parent class methods is not recommended. Common functions should be injected as dedicated classes rather than shared through inheritance or traits. [Composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance).

[^no-trait]: Injection traits like `ResourceInject` existed to reduce boilerplate code for injection, but lost their purpose with PHP8's [constructor property promotion](https://www.php.net/manual/en/language.oop5.decon.php#language.oop5.decon.constructor.promotion). Use constructor injection.

## DI

* Do not inject the execution context value itself (prod, dev, etc.). Instead, inject instances according to the context. The application should be unaware of which context it's running in.
* Setter injection is not recommended for library code.
* Prefer `toConstructor` bindings over `Provider` bindings where possible.
* Avoid conditional bindings in `Module`. ([AvoidConditionalLogicInModules](https://github.com/google/guice/wiki/AvoidConditionalLogicInModules))
* Do not reference environment variables from module's `configure()`. Use constructor injection.

## AOP

* Do not make interceptors mandatory. For example, logging and DB transactions should not change the essential behavior of the program even without interceptors.
* Avoid having interceptors inject dependencies into methods. Values that can only be determined at implementation time should be injected into arguments via `@Assisted` injection.
* When there are multiple interceptors, avoid depending on execution order as much as possible.
* For interceptors that are unconditionally applied to all methods, consider describing them in `bootstrap.php`.
* AOP is used to separate cross-cutting concerns from core concerns. Using interceptors to hack specific methods is not recommended.

## Script Commands

* It is recommended that `composer setup` command completes application setup. This script includes database initialization and required library checks. If manual operations like `.env` settings are needed, it is recommended to display the procedure on screen.

## Environment

* Applications that only work on the Web are not recommended. Make them work on console as well for testability.
* It is recommended not to include `.env` files in the project repository.
* Consider using [Koriym.EnvJson](https://github.com/koriym/Koriym.EnvJson) which describes schema instead of `.env`.

## Testing

* Focus on resource testing using resource clients, adding resource representation testing (e.g., HTML) if needed.
* Hypermedia tests can preserve use cases as tests.
* `prod` is the context for production. Use of `prod` context in tests should be minimal, preferably none.

## HTML Templates

* Avoid large loop statements. Consider replacing if statements in loops with [Generator](https://www.php.net/manual/en/language.generators.overview.php).
