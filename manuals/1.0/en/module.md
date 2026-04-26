---
layout: docs-en
title: Module
category: Manual
permalink: /manuals/1.0/en/module.html
---
# Modules

A Module is a set of DI and AOP bindings. BEAR.Sunday does not have general configuration files, Config classes, or execution modes. Values needed by each component are provided through dependency injection. Modules are responsible for these dependency bindings.

The entry point module for the application is `AppModule` (src/Module/AppModule.php). Other required modules are `install`ed within `AppModule`.

Values needed by modules (values needed at compile time, not runtime) are bound through manual constructor injection.

```php
class AppModule extends AbstractAppModule
{
    /**
     * {@inheritdoc}
     */
    protected function configure()
    {
        // Additional modules
        $this->install(new AuraSqlModule('mysql:host=localhost;dbname=test', getenv('db_username'), getenv('db_password')));
        $this->install(new TwigModule());
        // Package standard module
        $this->install(new PackageModule());
    }
}
```

## DI Bindings

Here are the representative binding patterns:

```php
// Class binding
$this->bind($interface)->to($class);

// Provider (factory) binding
$this->bind($interface)->toProvider($provider);

// Instance binding
$this->bind($interface)->toInstance($instance);

// Named binding
$this->bind($interface)->annotatedWith($annotation)->to($class);

// Singleton
$this->bind($interface)->to($class)->in(Scope::SINGLETON);

// Constructor binding
$this->bind($interface)->toConstructor($class, $named);
```

For details, see [DI](di.html).

## AOP Configuration

AOP "searches" for classes and methods using `Matcher` and binds interceptors to matching methods.

```php
// Example 1: Binding by method name
$this->bindInterceptor(
    $this->matcher->any(),                   // In any class
    $this->matcher->startsWith('delete'),    // Methods starting with "delete"
    [Logger::class]                          // Bind Logger interceptor
);

// Example 2: Binding by class and annotation
$this->bindInterceptor(
    $this->matcher->SubclassesOf(AdminPage::class),  // Classes inheriting or implementing AdminPage
    $this->matcher->annotatedWith(Auth::class),      // Methods annotated with @Auth
    [AdminAuthentication::class]                     // Bind AdminAuthentication interceptor
);
```

For details, see [AOP](aop.html).

## Binding Priority

### Priority Within the Same Module

Within the same module, earlier bindings take priority. In the following example, Foo1 takes priority:

```php
$this->bind(FooInterface::class)->to(Foo1::class);
$this->bind(FooInterface::class)->to(Foo2::class);
```

### Priority in Module Installation

Earlier installed modules take priority. In the following example, Foo1Module takes priority:

```php
$this->install(new Foo1Module);
$this->install(new Foo2Module);
```

To give later modules priority, use `override`. In the following example, Foo2Module takes priority:

```php
$this->install(new Foo1Module);
$this->override(new Foo2Module);
```

### Priority in Context String

In context strings, left modules have priority in bindings. For example, with `prod-hal-app`:

- HalModule has priority over AppModule
- ProdModule has priority over HalModule

They are installed in this priority order.
