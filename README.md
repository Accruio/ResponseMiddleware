# Relay Response Middleware

[![Build Status](https://travis-ci.org/Accruio/ResponseMiddleware.svg?branch=1.x)](https://travis-ci.org/Accruio/ResponseMiddleware)
[![Latest Stable Version](https://poser.pugx.org/accruio/response-middleware/v/stable)](https://packagist.org/packages/accruio/response-middleware)
[![Total Downloads](https://poser.pugx.org/accruio/response-middleware/downloads)](https://packagist.org/packages/accruio/response-middleware)
[![Latest Unstable Version](https://poser.pugx.org/accruio/response-middleware/v/unstable)](https://packagist.org/packages/accruio/response-middleware)
[![License](https://poser.pugx.org/accruio/response-middleware/license)](https://packagist.org/packages/accruio/response-middleware)


This package include the following Relay-compatible response middleware:

- _ResponseSender_ to send a PSR-7 response
- _ExceptionHandler_ to handle exceptions from subsequent middleware
- _FormContentHandler_ to deserialize the URL-encoded payload of a PSR-7 request
- _JsonContentHandler_ to deserialize the JSON payload of a PSR-7 request
- _JsonDecoder_ to deserialize the JSON payload of a PSR-7 request (**deprecated**)

This package is installable and PSR-4 autoloadable via Composer as `relay/middleware`.

## ResponseSender

The _ResponseSender_ does just what it sounds like: it sends the PSR-7 response object.

The _ResponseSender_ does nothing with the `$request` or `$response`, passing them immediately to `$next`. Afterwards, it takes the returned `$response` and sends it using `header()` and `echo`, and returns the sent `$response`.

The _ResponseSender_ is intended to go at the top of the Relay queue, so that it is the middleware with the last opportunity to do something with the returned response.

To add the _ResponseSender_ to your Relay queue, instantiate it directly ...

```php
$queue[] = new \Relay\Middleware\ResponseSender();
```

... or use a `$resolver` of your choice to instantiate it from the `$queue`.

## ExceptionHandler

Similarly, the _ExceptionHandler_ does what it sound like: it catches any exceptions that bubble up through the subsequent middleware decorators.

The _ExceptionHandler_ does nothing with the `$request` or `$response`, and passes them directly to `$next` inside a `try/catch` block. If no exception bubbles up, it returns the `$response` from `$next`.  However, if it catches an exception, it returns an entirely new `$response` object with the exception message and an HTTP 500 status code. It then returns the new `$response` object.

The _ExceptionHandler_ is intended to go near the top of the Relay queue, but after the _ResponseSender_, so that the _ResponseSender_ can then send the returned `$response`.

To add the _ExceptionHandler_ to your queue, instantiate it directly with an empty $response implementation object ...

```php
$queue = new \Relay\Middleware\ExceptionHandler(new ResponseImplementation());
```

... or use a `$resolver` of your choice to instantiate it from the `$queue`.

## JsonContentHandler

Again, the _JsonContentHandler_ does what it sounds like: it deserializes the JSON
payload of a PSR-7 request object and makes the parameters available in
subsequent middleware decorators.

The _JsonContentHandler_ checks the incoming request for a method other than `GET`
and for an `application/json` or `application/vnd.api+json` `Content-Type` header.
If it finds both of these, it parses the JSON and makes it available as the
_parsed body_ of the `$request` before passing it and the `$response` to `$next`.
If the method is `GET` or the `Content-Type` header defines a different mime type,
the _JsonContentHandler_ ignores the `$request` and continues the chain.

To add the _JsonContentHandler_ to your queue, instantiate it directly...

```php
$queue = new \Relay\Middleware\JsonContentHandler();
```

... or use a `$resolver` of your choice to instantiate it from the `$queue`.

To access the decoded parameters in subsequent middleware, use the
`getParsedBody()` method of the `$request`

```php
$decodedJsonData = $request->getParsedBody();
```

## FormContentHandler

_FormContentHandler_ works almost identically to _JsonContentHandler_, but parses
payloads of requests that have `application/x-www-form-urlencoded` as the `Content-Type`.


## JsonDecoder

**NOTE: This handler has been deprecated in favor of _JsonContentHandler_!**

Again, the _JsonDecoder_ does what it sounds like: it deserializes the JSON
payload of a PSR-7 request object and makes the parameters available in
subsequent middleware decorators.

The _JsonDecoder_ checks the incoming request for a method other than `GET` and
for an `application/json` `Content-Type` header. If it finds both of these, it
decodes the JSON and makes it available as the _parsed body_ of the `$request`
before passing it and the `$response` to `$next`. If the method is `GET` or the
`Content-Type` header does not specify `application/json`, the _JsonDecoder_
does nothing with the `$request` and passes it and the `$response` to `$next`.


To add the _JsonDecoder_ to your queue, instantiate it directly...

```php
$queue = new \Relay\Middleware\JsonDecoder();
```

... or use a `$resolver` of your choice to instantiate it from the `$queue`.

To access the decoded parameters in subsequent middleware, use the
`getParsedBody()` method of the `$request`

```php
$decodedJsonData = $request->getParsedBody();
```
