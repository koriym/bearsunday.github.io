---
layout: docs-en
title: PHPDoc Types
category: Manual
permalink: /manuals/1.0/en/types.html
---
# PHPDoc Types

PHP is a dynamically typed language, but by using static analysis tools such as Psalm and PHPStan together with PHPDoc, you can express advanced type concepts and benefit from type checking at static analysis time. This reference describes the types available in PHPDoc and other related concepts.

## Table of Contents

1. [Atomic Types](#atomic-types)
   - [Scalar Types](#scalar-types)
   - [Object Types](#object-types)
   - [Array Types](#array-types)
   - [Callable Types](#callable-types)
   - [Value Types](#value-types)
   - [Special Types](#special-types)
2. [Composite Types](#composite-types)
   - [Union Types](#union-types)
   - [Intersection Types](#intersection-types)
3. [Advanced Type System](#advanced-type-system)
   - [Generic Types](#generic-types)
   - [Template Types](#template-types)
   - [Conditional Types](#conditional-types)
   - [Type Aliases](#type-aliases)
   - [Type Constraints](#type-constraints)
   - [Covariance and Contravariance](#covariance-and-contravariance)
4. [Type Operators (Utility Types)](#type-operators)
  - [Key-of and Value-of Types](#key-of-and-value-of-types)
  - [Properties-of Type](#properties-of-type)
  - [Class String Map (class-string-map<T of Foo, T>)](#class-string-map)
  - [Index Access Type (T[K])](#index-access-type)
5. [Functional Programming Concepts](#functional-programming-concepts)
   - [Pure Functions](#pure-functions)
   - [Immutable Objects](#immutable-objects)
   - [Side Effect Annotations](#side-effect-annotations)
   - [Higher-Order Functions](#higher-order-functions)
6. [Assert Annotations](#assert-annotations)
7. [Security Annotations](#security-annotations)
8. [Examples: Using Types in Design Patterns](#examples-using-types-in-design-patterns)

---

## Atomic Types

These are basic types that cannot be subdivided further.

### Scalar Types

```php
/** @param int $i */
/** @param float $f */
/** @param string $str */
/** @param lowercase-string $lowercaseStr */
/** @param non-empty-string $nonEmptyStr */
/** @param non-empty-lowercase-string $nonEmptyLowercaseStr */
/** @param class-string $class */
/** @param class-string<AbstractFoo> $fooClass */
/** @param callable-string $callable */
/** @param numeric-string $num */ 
/** @param bool $isSet */
/** @param array-key $key */
/** @param numeric $num */
/** @param scalar $a */
/** @param positive-int $positiveInt */
/** @param negative-int $negativeInt */
/** @param int-range<0, 100> $percentage */
/** @param int-mask<1, 2, 4> $flags */
/** @param int-mask-of<MyClass::CLASS_CONSTANT_*> $classFlags */
/** @param trait-string $trait */
/** @param enum-string $enum */
/** @param literal-string $literalStr */
/** @param literal-int $literalInt */
```

These types can be combined with [Composite Types](#composite-types) and the [Advanced Type System](#advanced-type-system).

### Object Types

```php
/** @param object $obj */
/** @param stdClass $std */
/** @param Foo\Bar $fooBar */
/** @param object{foo: string, bar?: int} $objWithProperties */
/** @return ArrayObject<int, string> */
/** @param Collection<User> $users */
/** @return Generator<int, string, mixed, void> */
```

Object types can be used in combination with [Generic Types](#generic-types).

### Array Types

#### Generic Arrays

```php
/** @return array<TKey, TValue> */
/** @return array<int, Foo> */
/** @return array<string, int|string> */
/** @return non-empty-array<string, int> */
```

Generic arrays use the concept of [Generic Types](#generic-types).

#### Object-Like Arrays

```php
/** @return array{0: string, 1: string, foo: stdClass, 28: false} */
/** @return array{foo: string, bar: int} */
/** @return array{optional?: string, bar: int} */
```

#### Lists

```php
/** @param list<string> $stringList */
/** @param non-empty-list<int> $nonEmptyIntList */
```

#### PHPDoc Arrays (Legacy Notation)

```php
/** @param string[] $strings */
/** @param int[][] $nestedInts */
```

### Callable Types

```php
/** @return callable(Type1, OptionalType2=, SpreadType3...): ReturnType */
/** @return Closure(bool):int */
/** @param callable(int): string $callback */
```

Callable types are particularly important in [Higher-Order Functions](#higher-order-functions).

### Value Types

```php
/** @return null */
/** @return true */
/** @return false */
/** @return 42 */
/** @return 3.14 */
/** @return "specific string" */
/** @param Foo\Bar::MY_SCALAR_CONST $const */
/** @param A::class|B::class $classNames */
```

### Special Types

```php
/** @return void */
/** @return never */
/** @return empty */
/** @return mixed */
/** @return resource */
/** @return closed-resource */
/** @return iterable<TKey, TValue> */
```

## Composite Types

These are types created by combining multiple [Atomic Types](#atomic-types).

### Union Types

```php
/** @param int|string $id */
/** @return string|null */
/** @var array<string|int> $mixedArray */
/** @return 'success'|'error'|'pending' */
```

### Intersection Types

```php
/** @param Countable&Traversable $collection */
/** @param Renderable&Serializable $object */
```

Intersection types can be useful in implementing [Design Patterns](#examples-using-types-in-design-patterns).

## Advanced Type System

Advanced features that enable more complex and flexible type expressions.

### Generic Types

```php
/**
 * @template T
 * @param array<T> $items
 * @param callable(T): bool $predicate
 * @return array<T>
 */
function filter(array $items, callable $predicate): array {
    return array_filter($items, $predicate);
}
```

Generic types are often used in combination with [Higher-Order Functions](#higher-order-functions).

### Template Types

```php
/**
 * @template T of object
 * @param class-string<T> $className
 * @return T
 */
function create(string $className)
{
    return new $className();
}
```

Template types can be used together with [Type Constraints](#type-constraints).

### Conditional Types

```php
/**
 * @template T
 * @param T $value
 * @return (T is string ? int : string)
 */
function processValue($value) {
    return is_string($value) ? strlen($value) : strval($value);
}
```

Conditional types are sometimes used together with [Union Types](#union-types).

### Type Aliases

```php
/**
 * @psalm-type UserId = positive-int
 * @psalm-type UserData = array{id: UserId, name: string, email: string}
 */

/**
 * @param UserData $userData
 * @return UserId
 */
function createUser(array $userData): int {
    // user creation logic
    return $userData['id'];
}
```

Type aliases help simplify complex type definitions.

### Type Constraints

You can specify more specific type requirements by adding constraints to type parameters.

```php
/**
 * @template T of \DateTimeInterface
 * @param T $date
 * @return T
 */
function cloneDate($date) {
    return clone $date;
}

// Usage example
$dateTime = new DateTime();
$clonedDateTime = cloneDate($dateTime);
```

In this example, `T` is constrained to classes that implement `\DateTimeInterface`.

### Covariance and Contravariance

When working with generic types, the concepts of [covariance and contravariance](https://www.php.net/manual/en/language.oop5.variance.php) become important.

```php
/**
 * @template-covariant T
 */
interface Producer {
    /** @return T */
    public function produce();
}

/**
 * @template-contravariant T
 */
interface Consumer {
    /** @param T $item */
    public function consume($item);
}

// Usage example
/** @var Producer<Dog> $dogProducer */
/** @var Consumer<Animal> $animalConsumer */
```

Covariance means that more derived types (subtypes) can be used, while contravariance means that more general types (supertypes) can be used.

## Type Operators

Type operators allow you to generate new types from existing types. In Psalm these are referred to as utility types.


### Key-of and Value-of Types

- `key-of` retrieves the types of all keys of a given array or object, while `value-of` retrieves the types of its values.

```php
/**
 * @param key-of<UserData> $key
 * @return value-of<UserData>
 */
function getUserData(string $key) {
    $userData = ['id' => 1, 'name' => 'John', 'email' => 'john@example.com'];
    return $userData[$key] ?? null;
}

/**
 * @return ArrayIterator<key-of<UserData>, value-of<UserData>>
 */
function getUserDataIterator() {
    $userData = ['id' => 1, 'name' => 'John', 'email' => 'john@example.com'];
    return new ArrayIterator($userData);
}
```

### Properties-of Type

`properties-of` represents the types of all properties of a class. This is useful when handling class properties dynamically.

```php
class User {
    public int $id;
    public string $name;
    public ?string $email;
}

/**
 * @param User $user
 * @param key-of<properties-of<User>> $property
 * @return value-of<properties-of<User>>
 */
function getUserProperty(User $user, string $property) {
    return $user->$property;
}

// Usage example
$user = new User();
$propertyValue = getUserProperty($user, 'name'); // $propertyValue is of type string
```

`properties-of` has the following variants:

- `public-properties-of<T>`: Targets public properties only.
- `protected-properties-of<T>`: Targets protected properties only.
- `private-properties-of<T>`: Targets private properties only.

By using these variants, you can target only properties with specific access modifiers.

### Class String Map

`class-string-map` represents an array whose keys are class names and whose values are instances of those classes. This is useful when implementing dependency injection containers or factory patterns.

```php
/**
 * @template T of object
 * @param class-string-map<T, T> $map
 * @param class-string<T> $className
 * @return T
 */
function getInstance(array $map, string $className) {
    return $map[$className] ?? new $className();
}

// Usage example
$container = [
    UserRepository::class => new UserRepository(),
    ProductRepository::class => new ProductRepository(),
];

$userRepo = getInstance($container, UserRepository::class);
```

### Index Access Type

The index access type (`T[K]`) represents the element of type `T` at index `K`. This is useful for accurately expressing the types of accessing array or object properties.

```php
/**
 * @template T of array
 * @template K of key-of<T>
 * @param T $data
 * @param K $key
 * @return T[K]
 */
function getArrayValue(array $data, $key) {
    return $data[$key];
}

// Usage example
$config = ['debug' => true, 'version' => '1.0.0'];
$debugMode = getArrayValue($config, 'debug'); // $debugMode is of type bool
```

These utility types are specific to Psalm and can be considered part of the [Advanced Type System](#advanced-type-system).

## Functional Programming Concepts

PHPDoc supports important concepts influenced by functional programming. By using these concepts, you can improve the predictability and reliability of your code.

### Pure Functions

A pure function is one that has no side effects and always returns the same output for the same input.

```php
/**
 * @pure
 */
function add(int $a, int $b): int 
{
    return $a + $b;
}
```

This makes it explicit that the function has no side effects and that its result depends only on its inputs.

### Immutable Objects

Immutable objects are objects whose state does not change after they are created.

```php
/**
 * @immutable
 * - All properties are effectively treated as `readonly`.
 * - All methods are implicitly treated as `@psalm-mutation-free`.
 */
class Point {
    public function __construct(
        private float $x, 
        private float $y
    ) {}

    public function withX(float $x): static 
    {
        return new self($x, $this->y);
    }

    public function withY(float $y): static
    {
        return new self($this->x, $y);
    }
}
```

#### @psalm-mutation-free

This annotation indicates that a method modifies neither the internal state of the class nor any external state. Methods of an `@immutable` class implicitly have this property, but it can also be applied to specific methods of non-immutable classes.

```php
class Calculator {
    private float $lastResult = 0;

    /**
     * @psalm-mutation-free
     */
    public function add(float $a, float $b): float {
        return $a + $b;
    }

    public function addAndStore(float $a, float $b): float {
        $this->lastResult = $a + $b; // This is not allowed under @psalm-mutation-free
        return $this->lastResult;
    }
}
```

#### @psalm-external-mutation-free

This annotation indicates that the method does not modify state outside of the class. Internal state changes are allowed.

```php
class Logger {
    private array $logs = [];

    /**
     * @psalm-external-mutation-free
     */
    public function log(string $message): void {
        $this->logs[] = $message; // Modifying internal state is allowed
    }

    public function writeToFile(string $filename): void {
        file_put_contents($filename, implode("\n", $this->logs)); // This modifies external state, so it cannot use @psalm-external-mutation-free
    }
}
```

#### Guidelines for Using Immutability Annotations

1. Use `@immutable` when the entire class is immutable.
2. Use `@psalm-mutation-free` when a specific method does not change state.
3. Use `@psalm-external-mutation-free` when a method may modify internal state but does not modify external state.

By expressing immutability appropriately, you gain many benefits, such as improved safety in concurrent processing, reduction of side effects, and improved code readability.

### Side Effect Annotations

When a function has side effects, you can explicitly annotate them to alert callers to use the function with care.

```php
/**
 * @side-effect This function writes to the database
 */
function logMessage(string $message): void {
    // Process to write the message to the database
}
```

### Higher-Order Functions

Higher-order functions are functions that either take functions as arguments or return functions. PHPDoc lets you express the types of higher-order functions accurately.

```php
/**
 * @param callable(int): bool $predicate
 * @param list<int>           $numbers
 * @return list<int>
 */
function filter(callable $predicate, array $numbers): array {
    return array_filter($numbers, $predicate);
}
```

Higher-order functions are closely related to [Callable Types](#callable-types).

## Assert Annotations

Assert annotations are used to communicate to static analysis tools that certain conditions are met.

```php
/**
 * @psalm-assert string $value
 * @psalm-assert-if-true string $value
 * @psalm-assert-if-false null $value
 */
function isString($value): bool {
    return is_string($value);
}

/**
 * @psalm-assert !null $value
 */
function assertNotNull($value): void {
    if ($value === null) {
        throw new \InvalidArgumentException('Value must not be null');
    }
}

/**
 * @psalm-assert-if-true positive-int $number
 */
function isPositiveInteger($number): bool {
    return is_int($number) && $number > 0;
}
```

These assert annotations are used as follows:

- `@psalm-assert`: Indicates that the assertion is true if the function returns normally (without throwing an exception).
- `@psalm-assert-if-true`: Indicates that the assertion is true if the function returns `true`.
- `@psalm-assert-if-false`: Indicates that the assertion is true if the function returns `false`.

Assert annotations are sometimes used in combination with [Type Constraints](#type-constraints).

## Security Annotations

Security annotations are used to highlight security-relevant parts of code and to track potential vulnerabilities. There are mainly three annotations:

1. `@psalm-taint-source`: Indicates an untrusted input source.
2. `@psalm-taint-sink`: Indicates a place where security-sensitive operations are performed.
3. `@psalm-taint-escape`: Indicates a place where data has been safely escaped or sanitized.

Below are examples of using these annotations:

```php
/**
 * @psalm-taint-source input
 */
function getUserInput(): string {
    return $_GET['user_input'] ?? '';
}

/**
 * @psalm-taint-sink sql
 */
function executeQuery(string $query): void {
    // Execute SQL query
}

/**
 * @psalm-taint-escape sql
 */
function escapeForSql(string $input): string {
    return addslashes($input);
}

// Usage example
$userInput = getUserInput();
$safeSqlInput = escapeForSql($userInput);
executeQuery("SELECT * FROM users WHERE name = '$safeSqlInput'");
```

By using these annotations, static analysis tools can trace the flow of untrusted input and detect potential security issues such as SQL injection.

## Examples: Using Types in Design Patterns

By leveraging the type system, you can implement common design patterns in a more type-safe way.

#### Builder Pattern

```php
/**
 * @template T
 */
interface BuilderInterface {
    /**
     * @return T
     */
    public function build();
}

/**
 * @template T
 * @template-implements BuilderInterface<T>
 */
abstract class AbstractBuilder implements BuilderInterface {
    /** @var array<string, mixed> */
    protected $data = [];

    /** @param mixed $value */
    public function set(string $name, $value): static {
        $this->data[$name] = $value;
        return $this;
    }
}

/**
 * @extends AbstractBuilder<User>
 */
class UserBuilder extends AbstractBuilder {
    public function build(): User {
        return new User($this->data);
    }
}

// Usage example
$user = (new UserBuilder())
    ->set('name', 'John Doe')
    ->set('email', 'john@example.com')
    ->build();
```

#### Repository Pattern

```php
/**
 * @template T
 */
interface RepositoryInterface {
    /**
     * @param int $id
     * @return T|null
     */
    public function find(int $id);

    /**
     * @param T $entity
     */
    public function save($entity): void;
}

/**
 * @implements RepositoryInterface<User>
 */
class UserRepository implements RepositoryInterface {
    public function find(int $id): ?User {
        // Logic to fetch a user from the database
    }

    public function save(User $user): void {
        // Logic to save a user to the database
    }
}
```

## Summary

By deeply understanding and properly using PHPDoc's type system, you gain benefits such as self-documenting code, early bug detection through static analysis, powerful code completion and assistance from IDEs, clearer expression of code intent and structure, and reduction of security risks—enabling you to write more robust and maintainable PHP code. The following is a comprehensive example covering the available types.

```php
<?php

namespace App\Comprehensive\Types;

/**
 * A class covering atomic types, scalar types, union types, intersection types, and generic types
 * 
 * @psalm-type UserId = int
 * @psalm-type HtmlContent = string
 * @psalm-type PositiveFloat = float&positive
 * @psalm-type Numeric = int|float
 * @psalm-type QueryResult = array<string, mixed>
 */
class TypeExamples {
    /**
     * @param UserId|non-empty-string $id
     * @return HtmlContent
     */
    public function getUserContent(int|string $id): string {
        return "<p>User ID: {$id}</p>";
    }

    /**
     * @param PositiveFloat $amount
     * @return bool
     */
    public function processPositiveAmount(float $amount): bool {
        return $amount > 0;
    }
}

/**
 * Example of immutable class, functional programming, and pure functions
 * 
 * @immutable
 */
class ImmutableUser {
    /** @var non-empty-string */
    private string $name;

    /** @var positive-int */
    private int $age;

    /**
     * @param non-empty-string $name
     * @param positive-int $age
     */
    public function __construct(string $name, int $age) {
        $this->name = $name;
        $this->age = $age;
    }

    /**
     * @psalm-pure
     * @return ImmutableUser
     */
    public function withAdditionalYears(int $additionalYears): self {
        return new self($this->name, $this->age + $additionalYears);
    }
}

/**
 * Example of template types, generic types, conditional types, and covariance/contravariance
 * 
 * @template T
 * @template-covariant U
 */
class StorageContainer {
    /** @var array<T, U> */
    private array $items = [];

    /**
     * @param T $key
     * @param U $value
     */
    public function add(mixed $key, mixed $value): void {
        $this->items[$key] = $value;
    }

    /**
     * @param T $key
     * @return U|null
     */
    public function get(mixed $key): mixed {
        return $this->items[$key] ?? null;
    }
    
    /**
     * @template V
     * @param T $key
     * @return (T is string ? string : U|null)
     */
    public function get(mixed $key): mixed {
        return is_string($key) ? "default_string_value" : ($this->items[$key] ?? null);
    }
}

/**
 * Example of type constraints, utility types, functional programming, and assert annotations
 * 
 * @template T of array-key
 */
class UtilityExamples {
    /**
     * @template T of array-key
     * @psalm-param array<T, mixed> $array
     * @psalm-return list<T>
     * @psalm-assert array<string, mixed> $array
     */
    public function getKeys(array $array): array {
        return array_keys($array);
    }

    /**
     * @template T of object
     * @psalm-param class-string-map<T, array-key> $classes
     * @psalm-return list<T>
     */
    public function mapClasses(array $classes): array {
        return array_map(fn(string $className): object => new $className(), array_keys($classes));
    }
}

/**
 * Example of higher-order functions, type aliases, and index access types
 * 
 * @template T
 * @psalm-type Predicate = callable(T): bool
 */
class FunctionalExamples {
    /**
     * @param list<T> $items
     * @param Predicate<T> $predicate
     * @return list<T>
     */
    public function filter(array $items, callable $predicate): array {
        return array_filter($items, $predicate);
    }

    /**
     * @param array<string, T> $map
     * @param key-of<$map> $key
     * @return T|null
     */
    public function getValue(array $map, string $key): mixed {
        return $map[$key] ?? null;
    }
}

/**
 * Example of security annotations, type constraints, index access types, properties-of, key-of, and value-of
 * 
 * @template T
 */
class SecureAccess {
    /**
     * @psalm-type UserProfile = array{
     *   id: int,
     *   name: non-empty-string,
     *   email: non-empty-string,
     *   roles: list<non-empty-string>
     * }
     * @psalm-param UserProfile $profile
     * @psalm-param key-of<UserProfile> $property
     * @return value-of<UserProfile>
     * @psalm-taint-escape system
     */
    public function getUserProperty(array $profile, string $property): mixed {
        return $profile[$property];
    }
}

/**
 * Example of a very complex type structure, security annotations, and pure-function implementation
 * 
 * @template T of object
 * @template-covariant U of array-key
 * @psalm-type ErrorResponse = array{error: non-empty-string, code: positive-int}
 */
class ComplexExample {
    /** @var array<U, T> */
    private array $registry = [];

    /**
     * @param U $key
     * @param T $value
     */
    public function register(mixed $key, object $value): void {
        $this->registry[$key] = $value;
    }

    /**
     * @param U $key
     * @return T|null
     * @psalm-pure
     * @psalm-assert-if-true ErrorResponse $this->registry[$key]
     */
    public function getRegistered(mixed $key): ?object {
        return $this->registry[$key] ?? null;
    }
}

<?php

namespace App\Additional\Types;

/**
 * Example of template type constraints and contravariance
 * 
 * @template-contravariant T of \Throwable
 */
interface ErrorHandlerInterface {
    /**
     * @param T $error
     * @return void
     */
    public function handle(\Throwable $error): void;
}

/**
 * Example of an implementation for a more specific type
 * 
 * @implements ErrorHandlerInterface<\RuntimeException>
 */
class RuntimeErrorHandler implements ErrorHandlerInterface {
    public function handle(\Throwable $error): void {
        // Handling RuntimeException
    }
}

/**
 * Example of complex type combinations and conditional branching
 * 
 * @psalm-type JsonPrimitive = string|int|float|bool|null
 * @psalm-type JsonArray = array<array-key, JsonValue>
 * @psalm-type JsonObject = array<string, JsonValue>
 * @psalm-type JsonValue = JsonPrimitive|JsonArray|JsonObject
 */
class JsonProcessor {
    /**
     * @param JsonValue $value
     * @return (JsonValue is JsonObject ? array<string, mixed> : (JsonValue is JsonArray ? list<mixed> : scalar|null))
     */
    public function process(mixed $value): mixed {
        if (is_array($value)) {
            return array_keys($value) === range(0, count($value) - 1) 
                ? array_values($value)
                : $value;
        }
        return $value;
    }
}

/**
 * Example of more advanced tuple and record types
 */
class AdvancedTypes {
    /**
     * @return array{0: int, 1: string, 2: bool}
     */
    public function getTuple(): array {
        return [42, "hello", true];
    }

    /**
     * @param array{id: int, name: string, meta: array{created: string, modified?: string}} $record
     * @return void
     */
    public function processRecord(array $record): void {
        // Processing of record types
    }

    /**
     * @template T of object
     * @param class-string<T> $className
     * @param array<string, mixed> $properties
     * @return T
     */
    public function createInstance(string $className, array $properties): object {
        $instance = new $className();
        foreach ($properties as $key => $value) {
            $instance->$key = $value;
        }
        return $instance;
    }
}

/**
 * Example of custom type guards and assertions
 */
class TypeGuards {
    /**
     * @psalm-assert-if-true non-empty-string $value
     */
    public function isNonEmptyString(mixed $value): bool {
        return is_string($value) && $value !== '';
    }

    /**
     * @template T of object
     * @param mixed $value
     * @param class-string<T> $className
     * @psalm-assert-if-true T $value
     */
    public function isInstanceOf(mixed $value, string $className): bool {
        return $value instanceof $className;
    }
}

/**
 * Example of test-related type annotations for PHPUnit
 */
class TestTypes {
    /**
     * @param class-string<\Exception> $expectedClass
     * @param callable(): mixed $callback
     */
    public function expectException(string $expectedClass, callable $callback): void {
        try {
            $callback();
            $this->fail('Exception was not thrown');
        } catch (\Exception $e) {
            $this->assertInstanceOf($expectedClass, $e);
        }
    }

    /**
     * @template T
     * @param T $expected
     * @param T $actual
     * @param non-empty-string $message
     */
    public function assertEquals(mixed $expected, mixed $actual, string $message = ''): void {
        // Type-safe comparison logic
    }
}

/**
 * Advanced examples of collection types and iterators
 * 
 * @template-covariant TKey of array-key
 * @template-covariant TValue
 * @template-implements \IteratorAggregate<TKey, TValue>
 */
class TypedCollection implements \IteratorAggregate {
    /** @var array<TKey, TValue> */
    private array $items = [];

    /**
     * @return \Traversable<TKey, TValue>
     */
    public function getIterator(): \Traversable {
        yield from $this->items;
    }

    /**
     * @param TValue $item
     * @return void
     */
    public function add(mixed $item): void {
        $this->items[] = $item;
    }

    /**
     * @template TCallback
     * @param callable(TValue): TCallback $callback
     * @return TypedCollection<TKey, TCallback>
     */
    public function map(callable $callback): self {
        $result = new self();
        foreach ($this->items as $key => $value) {
            $result->items[$key] = $callback($value);
        }
        return $result;
    }
}

/**
 * Example of conditional methods
 */
interface ConditionalInterface {
    /**
     * @template T
     * @param T $value
     * @return (T is numeric ? float : string)
     */
    public function process(mixed $value): mixed;
}

```

## Reference

To make full use of PHPDoc types, you need static analysis tools such as Psalm or PHPStan. For more details, refer to the following resources:

- [Psalm - Typing in Psalm](https://psalm.dev/docs/annotating_code/typing_in_psalm/)
  - [Templating](https://psalm.dev/docs/annotating_code/templated_annotations/)
  - [Assertions](https://psalm.dev/docs/annotating_code/adding_assertions/)
  - [Security Analysis](https://psalm.dev/docs/security_analysis/)
- [PHPStan - PHPDoc Types](https://phpstan.org/writing-php-code/phpdoc-types)
