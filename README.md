HTTP Message plugin for PHP Fat-Free Framework
---

This is a plugin to enable [PSR-7](https://www.php-fig.org/psr/psr-7/) HTTP message usage by the framework core.

In order to do so, this package can register any [PSR-17](https://www.php-fig.org/psr/psr-17/) compatible HTTP message factory to the frameworks' core dependency injection container, that can be used to create and hydrate a Request (or ServerRequest) object from current framework globals, so it can be consumed by the route-handling controller or [any other package](https://packagist.org/providers/psr/http-factory-implementation).

## Installation

This plugin requires:
- at least [`fatfree-core`](https://github.com/f3-factory/fatfree-core) version 4.0
- the DI `CONTAINER` enabled 
- an actual HTTP message implementation package to be installed, i.e.  
  - [`f3-factory/fatfree-psr7`](https://github.com/f3-factory/fatfree-psr7) 
  - or [`nyholm/psr7`](https://packagist.org/packages/nyholm/psr7) 
  - or any other [PSR7 package](https://packagist.org/providers/psr/http-message-implementation).
- a [PSR17](https://packagist.org/providers/psr/http-factory-implementation) HTTP message factory implementation (already included in `f3-factory/fatfree-psr7`)

```bash
composer require f3-factory/fatfree-psr7-factory
```

### Register factory service

The HTTP message objects (PSR-7) are created by a related PSR-17 factory.

At first, we need to register the PSR-17 factories for every PSR-7 interface.
Some PSR-17 packages have different factories for each interface, some share the same  for all of them.

This example uses the `Psr17Factory` from our own [fatfree-psr7](https://github.com/f3-factory/fatfree-psr7) package, which is very fast and of course fat-free.

To install it run `composer require f3-factory/fatfree-psr7` OR install any other PSR-7 and PSR-17 package you'd like to use instead.

In your bootstrap code or front controller (i.e. index.php):

```php
// create the PSR17 adapter
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
$psrAdapter->registerRequest(RequestInterface::class);
$psrAdapter->registerResponse(ResponseInterface::class);
$psrAdapter->registerServerRequest(ServerRequestInterface::class);
```

**NB:** Instead of the code above, you can use this included shortcut, which will execute this exact same sequence for you, but it only works with our own fatfree-psr7 package:

```php
MessageFactory::registerDefaults()
```


Registering the RequestInterface, ServerRequestInterface and ResponseInterface bindings will tell the dependency injection container which shortcut to use to resolve and hydrate the objects accordingly when you type-hinting them in your route controller.


## Usage

You can receive the Request and Response objects from any point now, i.e. from route handlers via auto-injection:

```php
$f3->route('GET /hallo-world', function(
    Base $f3, 
    RequestInterface $request, 
    ResponseInterface $response, 
    StreamFactoryInterface $streamFactory
) {
    $agent = $request->getHeaderLine('User-Agent');
    return $response->withBody($streamFactory
        ->createStream('Your user agent is: '.$agent));
});
```

Alternatively you can create a new request object anywhere manually. These are going to be hydrated from the currently available $_SERVER and hive data via:

```php
$request = $f3->make(RequestInterface::class);
// or 
$request = MessageFactory::instance()->makeRequest();

// for Server Request:
$serverRequest = $f3->make(ServerRequestInterface::class);
// or 
$serverRequest = MessageFactory::instance()->makeServerRequest();
```

`Response` and the other classes should be instantiated via the PSR17 factory directly and do not need any special treatment by the framework core.


### Move uploaded files

This is an example how to use the ServerRequest object to move an uploaded file.

```php
$f3->route('POST /upload', function(Base $app, ServerRequestInterface $request)
{
    $dir = './uploads/';
    // handle uploaded files
    $files = $request->getUploadedFiles();
    foreach ($files as $fieldName => $file) {
        if ($file instanceof UploadedFile) {
            $file->moveTo($dir.$file->getClientFilename());
        }
    }
};
```


For more usage examples see original [PSR Http message readme](https://github.com/php-fig/http-message):

* [`PSR-7 Interfaces Method List`](https://github.com/php-fig/http-message/blob/master/docs/PSR7-Interfaces.md)
* [`PSR-7 Usage Guide`](https://github.com/php-fig/http-message/blob/master/docs/PSR7-Usage.md)
