---
layout: docs-en
title: Swoole
category: Manual
permalink: /manuals/1.0/en/swoole.html
---

# Swoole

Swoole is a PHP extension written in C/C++, an event-driven asynchronous and coroutine-based concurrent networking communication engine.
You can execute your BEAR.Sunday web application directly from the command line using Swoole. It dramatically improves performance.

## Install

### Swoole Install

Via PECL:

```bash
pecl install swoole
```

From source:

```bash
git clone https://github.com/swoole/swoole-src.git && \
cd swoole-src && \
phpize && \
./configure && \
make && make install
```

Add `extension=swoole.so` to your `php.ini`.

### BEAR.Swoole Install

```bash
composer require bear/swoole ^0.4
```

No installation in `AppModule` is required.

Place the bootstrap script at `bin/swoole.php`.

```php
<?php
require dirname(__DIR__) . '/autoload.php';
exit((require dirname(__DIR__) . '/vendor/bear/swoole/bootstrap.php')(
    'prod-hal-app',       // context
    'MyVendor\MyProject', // application name
    '127.0.0.1',          // IP
    8080                  // port
));
```

## Execute

Start the server.

```bash
php bin/swoole.php
```

When executed, the following message is displayed:

```
Swoole http server is started at http://127.0.0.1:8088
```

## Benchmarking site

[BEAR.HelloworldBenchmark](https://github.com/bearsunday/BEAR.HelloworldBenchmark) is provided for running benchmark tests in a specific environment.

* [The benchmark result](https://github.com/bearsunday/BEAR.HelloworldBenchmark/wiki)

[<img src="https://github.com/swoole/swoole-src/raw/master/mascot.png">](https://github.com/swoole/swoole-src)
