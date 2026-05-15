---
layout: docs-en
title: PHPDoc Utility Types
category: Manual
permalink: /manuals/1.0/en/types-utility.html
---
# PHPDoc Utility Types

Utility types are used to manipulate existing types or dynamically generate new types. Using these types enables more flexible and expressive type definitions.

## Table of Contents

1. [key-of<T>](#key-oft)
2. [value-of<T>](#value-oft)
3. [properties-of<T>](#properties-oft)
4. [class-string-map<T of Foo, T>](#class-string-mapt-of-foo-t)
5. [T[K]](#tk)

## key-of<T>

`key-of<T>` represents the type of all possible keys of type `T`.

```php
/**
 * @template T of array
 * @param T $data
 * @param key-of<T> $key
 * @return mixed
 */
function getValueByKey(array $data, $key) {
    return $data[$key];
}

// Usage example
$userData = ['id' => 1, 'name' => 'John'];
$name = getValueByKey($userData, 'name'); // OK
$age = getValueByKey($userData, 'age'); // Psalm will warn
```

## value-of<T>

`value-of<T>` represents the type of all possible values of type `T`.

```php
/**
 * @template T of array
 * @param T $data
 * @return value-of<T>
 */
function getRandomValue(array $data) {
    return $data[array_rand($data)];
}

// Usage example
$numbers = [1, 2, 3, 4, 5];
$randomNumber = getRandomValue($numbers); // int type
```

## properties-of<T>

`properties-of<T>` represents the type of all properties of type `T`.

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
$name = getUserProperty($user, 'name'); // string type
$id = getUserProperty($user, 'id'); // int type
$unknown = getUserProperty($user, 'unknown'); // Psalm will warn
```

## class-string-map<T of Foo, T>

`class-string-map` represents an array with class names as keys and their instances as values.

```php
interface Repository {}
class UserRepository implements Repository {}
class ProductRepository implements Repository {}

/**
 * @template T of Repository
 * @param class-string-map<T, T> $repositories
 * @param class-string<T> $className
 * @return T
 */
function getRepository(array $repositories, string $className): Repository {
    return $repositories[$className];
}

// Usage example
$repositories = [
    UserRepository::class => new UserRepository(),
    ProductRepository::class => new ProductRepository(),
];

$userRepo = getRepository($repositories, UserRepository::class);
```

## T[K]

`T[K]` represents the element at index `K` of type `T`.

```php
/**
 * @template T of array
 * @template K of array-key
 * @param T $data
 * @param K $key
 * @return T[K]
 */
function getArrayElement(array $data, $key) {
    return $data[$key];
}

// Usage example
$config = ['debug' => true, 'version' => '1.0.0'];
$debugMode = getArrayElement($config, 'debug'); // bool type
```
