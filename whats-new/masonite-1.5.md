# Masonite 1.5

## Introduction

Masonite 1.5 is focused on a few bug fixes and changes to several core classes in order to expand on the support of third party package integration.

## Masonite Entry

Masonite 1.5 is releasing also with a new official package for adding RESTful API's to your project. This package used API Resources which are classes that can add a plethora of capabilities to your API endpoints. You can read more about it at the [Masonite Entry](https://masoniteframework.gitbook.io/masonite-entry) documentation.

## HTTP Methods

Masonite 1.5 also adds additional support HTTP methods like `PUT`, `PATCH` and `DELETE`. More information about these new routes can be found under the [Routing](../the-basics/routing.md) documentation

## Moving Dependencies

Nearly all dependencies have been moved to the core Masonite package. The only thing inside the project that is installed using `craft new` is the WSGI server \(which is waitress by default\) and Masonite itself. This will improve the ability to change and update dependencies.

## Changing Form Request Methods

HTML forms only support GET and POST so there is no the ability to add any other HTTP methods like `PUT` and `PATCH` which will change the actual request method on submission. You can read more about this in the [Requests](../the-basics/requests.md) documentation.

## Added Shorthand Route Helpers

You can now use functions for routes instead of the classes. These new functions are just wrappers around the classes itself. Read more about this under the [Routing](../the-basics/routing.md) documentation.

## Removed API from core

In Masonite 1.4 and below was the `Api()` route which added some very basic API endpoints. All references to API's have been removed from core and added to the new Masonite Entry official package.

If you would like to use API's you will have to use the Masonite Entry package instead.

## Headers

You can now get and set any header information using the new `Request.header` method. This also allows third party packages to manipulate header information. Read more about this in the [Requests](../the-basics/requests.md) documentation.

## Cookies

You can now delete cookies using the `delete_cookie` method as well as set expiration dates for them. See the [Requests](../the-basics/requests.md) documentation for more information.

## Caching Driver Update

The Cache driver now has an update method which can update a cache value by key. This is useful if you want to change a key value or increment it. Storing a cache file also now auto creates that directory. Read more about this in the [Caching](../useful-features/caching.md) documentation.

## An All New Craft Command

Craft commands have been built from the ground up with the cleo package. It's an excellent package that is built around the extendability of commands by using primarily classes \(instead of decorator functions\). Read more under [The Craft Command](../the-craft-command/introduction.md) documentation

## Adding Commands

It is now possible to add craft commands to craft. You can read more about how under [The Craft Command](../the-craft-command/introduction.md) documentation

## Adding Migration Directories

You can now add more migration directories by adding it to the container with a key ending in `MigrationDirectory`. This will add the directory to the list of directory that run when migrate commands are ran. You can read more about this in the [Creating Packages](../advanced/creating-packages.md) documentation.

## Added Sessions

You can now add data to sessions using the new Sessions feature which comes with a memory and cookie driver for storing data.

## Craft Auto Adds Site Packages

In previous versions, Masonite has not been able to fetch the site packages directory if you were in a virtual environment because virtual environment directories are dynamically named depending on who created it. We have found a way to detect the virtual environment and the site packages directory so now there is no need to add the site packages directory manually to the packages configuration file

