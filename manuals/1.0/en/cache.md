---
layout: docs-en
title: Cache
category: Manual
permalink: /manuals/1.0/en/cache.html
---

# Cache

> There are only two hard things in Computer Science: cache invalidation and naming things.
>
> -- Phil Karlton

## Overview

A good caching system fundamentally improves the quality of user experience and reduces resource utilization costs and environmental impact. BEAR.Sunday supports the following caching features in addition to traditional simple TTL-based caching:

* Event-driven cache invalidation
* Cache dependency resolution
* Donut cache and donut hole cache
* CDN control
* Conditional requests

### Distributed Cache Framework

A distributed caching system that follows REST constraints saves not only computational resources but also network resources. BEAR.Sunday provides a caching framework that integrates **server-side caches** (such as Redis and APC handled directly by PHP), **shared caches** (known as content delivery networks - CDNs), and **client-side caches** (cached by web browsers and API clients) with modern CDNs.

<img src="https://user-images.githubusercontent.com/529021/137062427-c733c832-0631-4a43-a6ee-4204e6be007c.png" alt="distributed cache">

## Tag-based Cache Invalidation

<img width="369" alt="dependency graph 2021-10-19 21 38 02" src="https://user-images.githubusercontent.com/529021/137910748-b6e95839-eeb7-4ade-a564-3cdcd5fdc09e.png">

Content caching has dependency issues. If content A depends on content B, and B depends on C, then when C is updated, not only must C's cache and ETag be updated, but also B's cache and ETag (which depends on C), and A's cache and ETag (which depends on B).

BEAR.Sunday solves this problem by having each resource hold the URI of dependent resources as tags. When a resource embedded with `#[Embed]` is modified, the cache and ETag of all related resources are invalidated, and cache regeneration occurs for the next request.

## Donut Cache

<img width="200" alt="donut caching" src="https://user-images.githubusercontent.com/529021/137097856-f9428918-5b76-4c0e-8cea-2472c15d82e9.png">

Donut caching is a partial caching technique for cache optimization. It separates content into cacheable and non-cacheable parts and combines them for output.

For example, consider content containing a non-cacheable resource like "`Welcome to $name`". The non-cacheable (do-not-cache) part is combined with other cacheable parts for output.

<img width="557" alt="image" src="https://user-images.githubusercontent.com/529021/139617102-1f7f436c-a1f4-4c6c-b90b-de24491e4c8c.png">

In this case, since the entire content is dynamic, the whole donut is not cached. Therefore, no ETag is output either.

## Donut Hole Cache

<img width="544" alt="image" src="https://user-images.githubusercontent.com/529021/139617571-31aea99a-533f-4b95-b3f3-6c613407d377.png">

When the donut hole part is cacheable, it can be handled the same way as donut cache. In the example above, a weather forecast resource that changes once per hour is cached and included in the news resource.

In this case, since the donut content as a whole (news) is static, the entire content is also cached and given an ETag. This creates cache dependency. When the donut hole content is updated, the entire cached donut needs to be regenerated.

This dependency resolution happens automatically. To minimize computational resources, the donut part computation is reused. When the hole part (weather resource) is updated, the cache and ETag of the entire content are also automatically updated.

### Recursive Donut

<img width="191" alt="recursive donut 2021-10-19 21 27 06" src="https://user-images.githubusercontent.com/529021/137909083-2c5176f7-edb7-422b-bccc-1db90460fc15.png">

The donut structure is applied recursively. For example, if A contains B and B contains C, when C is modified, A's cache and B's cache are reused except for the modified C part. A's and B's caches and ETags are regenerated, but database access for A and B content retrieval and view rendering are not performed.

The optimized partial cache structure performs content regeneration with minimal cost. Clients don't need to know about the content cache structure.

## Event-Driven Content

Traditionally, CDNs considered content requiring application logic as "dynamic" and therefore not cacheable by CDNs. However, some CDNs like Fastly and Akamai now support immediate or tag-based cache invalidation within seconds, making [this thinking obsolete](https://www.fastly.com/blog/leveraging-your-cdn-cache-uncacheable-content).

BEAR.Sunday dependency resolution works not only server-side but also on shared caches. When AOP detects changes and makes PURGE requests to shared caches, related cache invalidation occurs on shared caches just like server-side.

## Conditional Requests

<img width="468" alt="conditional request" src="https://user-images.githubusercontent.com/529021/137151061-8d7a5605-3aa3-494c-91c5-c1deddd987dd.png">

Content changes are managed by AOP, and content entity tags (ETags) are automatically updated. HTTP conditional requests using ETags not only minimize computational resource usage, but responses returning only `304 Not Modified` also minimize network resource usage.

# Usage

For classes to be cached, use the `#[DonutCache]` attribute for donut cache (when embedded content is not cacheable), and `#[CacheableResponse]` for other cases:

```php
use BEAR\RepositoryModule\Annotation\CacheableResponse;

#[CacheableResponse]
class BlogPosting extends ResourceObject
{
    public $headers = [
        RequestHeader::CACHE_CONTROL => CacheControl::NO_CACHE
    ];

    #[Embed(rel: "comment", src: "page://self/html/comment")]
    public function onGet(int $id = 0): static
    {
        $this->body['article'] = 'hello world';

        return $this;
    }

    public function onDelete(int $id = 0): static
    {
        return $this;
    }
}
```

If you want to specify which methods to cache, add the attribute to the method instead of the class. In that case, add the `#[RefreshCache]` attribute to methods that modify the cache:

```php
class Todo extends ResourceObject
{
    #[CacheableResponse]
    public function onPut(int $id = 0, string $todo): static
    {
    }

    #[RefreshCache]
    public function onDelete(int $id = 0): static
    {
    }
}
```

Whichever way you add attributes, all the features introduced in the overview will apply. By default, cache invalidation by time (TTL) is not performed, assuming event-driven content. Note that with `#[DonutCache]` the entire content is not cached, but with `#[CacheableResponse]` it is.

## TTL

TTL is specified with `DonutRepositoryInterface::put()`. `ttl` is the cache time for non-donut holes, `sMaxAge` is the cache time for CDNs:

```php
use BEAR\RepositoryModule\Annotation\CacheableResponse;

#[CacheableResponse]
class BlogPosting extends ResourceObject
{
    public function __construct(private DonutRepositoryInterface $repository)
    {
    }

    #[Embed(rel: "comment", src: "page://self/html/comment")]
    public function onGet(): static
    {
        // process ...
        $this->repository->put($this, ttl: 10, sMaxAge: 100);
        return $this;
    }
}
```

### Default TTL Value

For event-driven content, content changes must be reflected in the cache immediately. Therefore, the default TTL varies depending on the installed CDN module.

If the CDN supports tag-based cache invalidation, the TTL is indefinite (1 year). If not supported, it is 10 seconds. The expected cache reflection time is immediate for Fastly, a few seconds for Akamai, and 10 seconds for others.

To customize this, implement `CdnCacheControlHeaderSetterInterface` referring to `CdnCacheControlHeader` and bind it.

## Cache Invalidation

Use the methods of `DonutRepositoryInterface` to manually invalidate the cache. Not only the specified cache, but also its ETag, caches of other resources that depend on it, and their ETags will be invalidated on both server-side and CDN (if possible):

```php
interface DonutRepositoryInterface
{
    public function purge(AbstractUri $uri): void;
    public function invalidateTags(array $tags): void;
}
```

### Invalidate by URI

```php
// example
$this->repository->purge(new Uri('app://self/blog/comment'));
```

### Invalidate by Tag

```php
$this->repository->invalidateTags(['template_a', 'campaign_b']);
```

### Tag Invalidation in CDN

To enable tag-based cache invalidation in CDN, you need to implement and bind `PurgerInterface`:

```php
use BEAR\QueryRepository\PurgerInterface;

interface PurgerInterface
{
    public function __invoke(string $tag): void;
}
```

### Specifying Dependency Tags

Use the `SURROGATE_KEY` header to specify the key for PURGE. Use a space as separator for multiple strings:

```php
use BEAR\QueryRepository\Header;

class Foo
{
    public $headers = [
        Header::SURROGATE_KEY => 'template_a campaign_b'
    ];
}
```

If the cache is invalidated by `template_a` or `campaign_b` tags, Foo's cache and ETag will be invalidated on both server-side and CDN.

### Resource Dependencies

Use `UriTagInterface` to convert a URI into a dependency tag string:

```php
public function __construct(private UriTagInterface $uriTag)
{
}
```

```php
$this->headers[Header::SURROGATE_KEY] = ($this->uriTag)(new Uri('app://self/foo'));
```

This cache will be invalidated on both server-side and CDN when `app://self/foo` is modified.

### Making Associative Array a Resource Dependency

```php
// body contents
[
    ['id' => '1', 'name' => 'a'],
    ['id' => '2', 'name' => 'b'],
]
```

To generate a list of dependent URI tags from a `body` associative array like above, specify the URI template with the `fromAssoc()` method:

```php
$this->headers[Header::SURROGATE_KEY] = $this->uriTag->fromAssoc(
    uriTemplate: 'app://self/item{?id}',
    assoc: $this->body
);
```

In this case, this cache will be invalidated on both server-side and CDN when `app://self/item?id=1` or `app://self/item?id=2` is changed.

## Configuration

### Redis Marshaller

The Redis cache adapter allows you to configure data compression and serialization methods.

A marshaller handles the serialization of PHP objects and arrays when storing them in Redis, and deserialization when retrieving them.

```php
use BEAR\QueryRepository\StorageRedisDsnModule;

$this->install(
    new StorageRedisDsnModule(
        dsn: 'redis://localhost:6379',
        marshallingOptions: [
            'enabled' => true,
            'type' => 'deflate',      // 'default' or 'deflate'
            'use_igbinary' => true    // requires ext-igbinary
        ]
    )
);
```

**Marshaller types:**

- `default`: Uses PHP's standard serialization (enabling `use_igbinary` uses a more efficient binary format)
- `deflate`: Compresses data before storing (uses zlib)

Use `deflate` if you want to reduce Redis memory usage. This is a trade-off with CPU usage.

## CDN Specific

If you install a module that supports a specific CDN, vendor-specific headers will be output:

```php
$this->install(new FastlyModule());
$this->install(new AkamaiModule());
```

## Multi-CDN

You can configure a multi-tier CDN and set TTL according to the role. For example, in the diagram below, a multi-functional CDN is placed upstream and a conventional CDN is placed downstream. Content invalidation is done for the upstream CDN, and the downstream CDN uses it.

<img width="344" alt="multi cdn diagram" src="https://user-images.githubusercontent.com/529021/137098809-ec949a15-8efb-4d03-9808-3be15523ade7.png">

# Response Headers

BEAR.Sunday automatically handles cache control for CDN and outputs headers for CDN. Client cache control should be specified in the `$header` of ResourceObject according to the content.

This section is important from security and maintenance perspectives. Make sure to specify `Cache-Control` in all ResourceObjects.

### Cannot Cache

Always specify content that cannot be cached:

```php
ResponseHeader::CACHE_CONTROL => CacheControl::NO_STORE
```

### Conditional Requests

Check the server for content changes before using the cache. Server-side content changes will be detected and reflected:

```php
ResponseHeader::CACHE_CONTROL => CacheControl::NO_CACHE
```

### Specifying Client Cache Time

The client caches content. This is the most efficient cache, but server-side content changes will not be reflected during the specified time. Also, this cache is not used when the browser reloads. The cache is used when navigating with `<a>` tags or entering URLs:

```php
ResponseHeader::CACHE_CONTROL => 'max-age=60'
```

If response time is important, consider specifying SWR:

```php
ResponseHeader::CACHE_CONTROL => 'max-age=30 stale-while-revalidate=10'
```

In this case, when max-age of 30 seconds is exceeded, the old cached (stale) response is returned for up to 10 seconds as specified by SWR, until a fresh response is obtained from the origin server. This means the cache is updated sometime between 30 and 40 seconds after the last update, but every request gets a cached response and is fast.

#### RFC7234 Compliant Clients

To use client cache with APIs, use an RFC7234 compliant API client:

* iOS: [NSURLCache](https://nshipster.com/nsurlcache/)
* Android: [HttpResponseCache](https://developer.android.com/reference/android/net/http/HttpResponseCache)
* PHP: [guzzle-cache-middleware](https://github.com/Kevinrob/guzzle-cache-middleware)
* JavaScript(Node): [cacheable-request](https://www.npmjs.com/package/cacheable-request)
* Go: [lox/httpcache](https://github.com/lox/httpcache)
* Ruby: [faraday-http-cache](https://github.com/plataformatec/faraday-http-cache)
* Python: [requests-cache](https://pypi.org/project/requests-cache/)

### Private Cache

Specify `private` if you do not want to share the cache with other clients. The cache is saved only on the client side. In this case, do not specify cache on the server side.

```php
ResponseHeader::CACHE_CONTROL => 'private, max-age=30'
```

## Cache Design

APIs (or content) can be divided into two categories: **Information APIs** and **Computation APIs**. Computation APIs have content that is difficult to reproduce and truly dynamic, making it unsuitable for caching. Information APIs, on the other hand, are APIs for content that is essentially static, even if read from a DB and processed by PHP.

Analyze the content to apply appropriate caching:

* Information API or Computation API?
* What are the dependencies?
* What are the containment relationships?
* Is invalidation triggered by event or TTL?
* Is the event detectable by the application or does it need monitoring?
* Is the TTL predictable or unpredictable?

Consider making cache design part of the application design process and include it in specifications. It will also contribute to the safety of your project throughout its lifecycle.

### Adaptive TTL

Adaptive TTL predicts the lifetime of content and correctly tells the client or CDN when it will not be updated during that time. For example, when dealing with a stock API, if it's Friday night, we know the information won't be updated until trading starts on Monday. We calculate the seconds until that time, specify it as TTL, and then specify the appropriate TTL during trading hours.

The client doesn't need to request a resource it knows won't be updated.

## #[Cacheable]

Traditional TTL caching with `#[Cacheable]` is also supported.

Example: 30 seconds cache on server side, 30 seconds cache on client. Since it's specified on server side, the same duration is cached on client side:

```php
use BEAR\RepositoryModule\Annotation\Cacheable;

#[Cacheable(expirySecond: 30)]
class CachedResource extends ResourceObject
{
```

Example: Cache the resource on server and client until the specified expiration date (the date in `$body['expiry_at']`):

```php
use BEAR\RepositoryModule\Annotation\Cacheable;

#[Cacheable(expiryAt: 'expiry_at')]
class CachedResource extends ResourceObject
{
```

See the [HTTP Cache](https://bearsunday.github.io/manuals/1.0/en/http-cache.html) page for more information.

## Conclusion

Web content can be of the information (data) type or the computation (process) type. Although the former is essentially static, it was difficult to treat as completely static content due to problems managing content changes and dependencies, so cache was invalidated by TTL even when no content changes occurred.

BEAR.Sunday's caching framework treats information type content as static as possible, maximizing the power of caching.

## Terminology

* [Conditional Requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests)
* [ETag (Version Identifier)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag)
* [Event-Driven Content](https://www.fastly.com/blog/rise-event-driven-content-or-how-cache-more-edge)
* [Donut Cache / Partial Cache](https://www.infoq.com/news/2011/12/MvcDonutCaching/)
* [Surrogate Key / Tag-Based Invalidation](https://docs.fastly.com/en/guides/getting-started-with-surrogate-keys)
* Headers
  * [Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)
  * [CDN-Cache-Control](https://blog.cloudflare.com/cdn-cache-control/)
  * [Vary](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Vary)
  * [Stale-While-Revalidate (SWR)](https://web.dev/stale-while-revalidate/)
