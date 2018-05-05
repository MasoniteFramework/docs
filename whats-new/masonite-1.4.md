# Masonite 1.4

## Introduction

Masonite 1.4 brings several new features to Masonite. These features include caching, template caching, websocket support with Masonite calls Broadcasting and much more testing to make Masonite as stable as possible. If you would like to contribute to Masonite, please read the [Contributing Guide](../prologue/contributing-guide.md) and the [How To Contribute](../prologue/how-to-contribute.md) documentation.

If you are upgrading from Masonite 1.3 then please read the [Masonite 1.3 to 1.4](../upgrade-guide/masonite-1.3-to-1.4.md) documentation.

## Broadcast Support

We recognize that in order for frameworks to keep up with modern web application, they require real time broadcasting. Masonite 1.4 brings basic broadcasting of events to masonite and comes with two drivers out of the box: `pusher` and `ably`. If you'd like to create more drivers then you can do so easily by reading the [About Drivers](../managers-and-drivers/about-drivers.md) documentation. If you do create a driver, please consider making it available on PyPi so others can install it into their projects or open an issue on GitHub and make to add it to the built in drivers.

## Caching

Masonite now has a built in caching class that you can use to either cache forever or cache for a specific amount of time.

## Template Caching

Templates may have a lot of logic that are only updated every few minutes or even every few months. With template caching you can now cache your templates anywhere from every few seconds to every few years. This is an extremely powerful caching technique that will allow your servers to run less intensively and easily increase the performance of your application.

If a page gets hit 100 times every second then you can cache for 5, 10 or 15 seconds at a time to lessen the load on your server.

This feature only activates if you have the `CacheProvider` loaded in your `PROVIDERS` list. If you try to use these features without that provider then you will be hit with a `RequiredContainerBindingNotFound` exception letting you know you are missing a required binding from a service provider. This provider comes out of the box in Masonite 1.4.

## PEP 8 Standards

We have also updated the code to closely conform to PEP 8 standards.

## Added a New Folder and Configuration File

Because of the caching features, we have added a `bootstrap/cache` folder where all caching will be put but you can change this in the new `config/cache.py` file.

## Added Contracts

Masonite 1.4 brings the idea of contracts which are very similar to interfaces in other languages. Contracts ensure that a driver or manager inherits has the same functionality across all classes of the same type.

## Added CSRF Protection

Cross-Site Request Forgery is a crucial security milestone to hit and Masonite 1.4 brings that ability. With a new Service Provider and middleware, we can now add a simple `{{ csrf_field|safe }}` to our forms and ensure we are protected from CSRF attacks.

## Changed Managers

Managers were very redundant before this release so we made it much easier to create managers with 2 simple class attributes instead of the redundant method. Managers are used to manage features and drivers to Masonite.

## Middleware is now resolved by the container

Now the constructor of all middleware is resolved by the container. This means you may use the IOC dependency injection techniques like controller methods and drivers.

## Fixed unused imports

There were two unused imports in the models that Masonite created. These have been removed completely.

