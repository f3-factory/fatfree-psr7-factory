HTTP Message plugin for PHP Fat-Free Framework
---

This is a plugin to enable HTTP Message usage by the framework core.  

The goal of this package is to register any [PSR-17](https://www.php-fig.org/psr/psr-17/) compatible http message factory to the frameworks core DI container and use those factories to create and hydrate a Request (or ServerRequest) object with all available data from the hive, so it can be consumed by the route-handling controller or any other package.

## Installation

This plugin requires fatfree-core ^4.0, the DI Container enabled and a package of a PSR-7 implementation installed.

```bash
composer require f3-factory/fatfree-psr7-factory
```

## Usage

The Http message objects (PSR-7) are created by a related (PSR-17) factory.

At first, we need to register these factories for every message objects.
Some PSR-17 factory packages have different classes for each object, some share the same class for all object types.

This example uses the `Psr17Factory` from our own [fatfree-psr7](https://github.com/f3-factory/fatfree-psr7) package, which is very fast and fat-free of course.

To install it run `composer require f3-factory/fatfree-psr7-factory` OR install any other PSR-7 and PSR-17 package you'd like to use instead.

In your front controller (i.e. index.php):

```php
// create the plugin adapter
$psrAdapter = F3\Http\MessageFactory::instance();
// register the factories:
$psrAdapter->register(
    requestFactory:       Psr17Factory::class,
    responseFactory:      Psr17Factory::class,
    serverRequestFactory: Psr17Factory::class,
    uploadedFileFactory:  Psr17Factory::class,
    uriFactory:           Psr17Factory::class,
    streamFactory:        Psr17Factory::class,
);
// register the concrete Request / Response objects 
$psrAdapter->registerRequest(\F3\Http\Request::class);
$psrAdapter->registerResponse(\F3\Http\Response::class);
$psrAdapter->registerServerRequest(\F3\Http\ServerRequest::class);
```

Registering the Request, ServerRequest and Response objects will tell the dependency injection container which shortcut it should use to resolve and hydrate the objects accordingly when you're type-hinting them in your route controller.

// TODO: add example of DI usage here

Alternatively you can create a new request object from the currently available $_SERVER and hive data via:

```php
MessageFactory::instance()->makeRequest();
// or 
MessageFactory::instance()->makeServerRequest();
```

`Response` and the other classes should be instantiated via the PSR17 factory directly and do not need any special treatment by the framework core.

---
