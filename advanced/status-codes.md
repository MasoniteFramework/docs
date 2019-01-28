# Status Codes

## Introduction

Status codes are a crucial part of any application. They allow your users to identify exactly what went wrong with your site without showing too much information. By default, Masonite will show generic 404 pages when a url is missed.

When APP\_DEBUG is True in your .env file, an exception view is shown to help you debug your application. When it is False then it will show a generic 500 error page.

This behavior is through the `StatusCodeProvider` in your `PROVIDERS` list. In addition to this behavior we can also show our own error pages.

## How It Works

Masonite will first look for the error that is being thrown in your `resources/templates/errors` directory and render that template. If one does not exist then it will return a generic view from the Masonite package itself.

## Usage

For example if a 404 Not Found error will be thrown then it will first check in `resources/templates/errors/404.html` and render that template. This is the same behavior for `500 Server Not Found` errors and other errors thrown.

