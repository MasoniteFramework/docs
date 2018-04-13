# Masonite 1.3

## Introduction

Masonite 1.3 comes with a plethora of improvements over previous versioning. This version brings new features such as Queue and Mail drivers as well as various bug fixes.

## Fixed infinite redirection

Previously when a you tried to redirect using the `Request.redirect()` method, Masonite would sometimes send the browser to an infinite redirection. This was because masonite was not resetting the redirection attributes of the `Request` class.

## Fixed browser content length

Previously the content length in the request header was not being set correctly which led to the gunicorn server showing a warning that the content length did not match the content of the output.

## Changed how request input data is retrieved

Previously the Request class simply got the input data on both `POST` and `GET` requests by converting the `wsgi.input` WSGI parameter into a string and parsing. All POST input data is now retrieved using `FieldStorage` which adds support for also getting files from `multipart/formdata` requests.

## Added Upload drivers

You may now simply upload images to both disk and Amazon S3 storage right out of the box. With the new `UploadProvider` service provider you can simply do something like:

```markup
<html>
  <body>
      <form action="/upload" method="POST" enctype="multipart/formdata">
        <input type="file" name="file">
      </form>
  </body>
</html>
```

```python
def show(self, Upload):
    Upload.store(Request.input('file'))
```

As well as support for Amazon S3 by setting the `DRIVER` to `s3`.

## Added several helper functions

These helper functions are added functions to the builtin Python functions which can be used by simply calling them as usual:

```python
def show(self):
    return view('welcome')
```

Notice how we never imported anything from the module or Service Container. See the [Helper Functions](../the-basics/helper-functions.md) documentation for a more exhaustive list

## Added a way to have global template variables

Very often you will want to have a single variable accessible in all of your views, such as the `Request` object or other class. We can use the new `View` class for this and put it in it's own service provider:

```python
def boot(self, View, Request):
    View.share({'request': Request})
```

## Middleware is now resolved by the container

You can now specify anything that is in the container in your middleware constructor and it will be resolved automatically from the container

## Added new domain method to the Get and Post classes

Specify the subdomain you want to target with your route. It's common to want to have separate routes for your public site and multi-tenant sites. This will now look something like:

```python
Get().domain('test').route('/dashboard', 'DashboardController@show')
```

Which will target `test.example.com/dashboard` and not `example.com/dashboard`. Read more about subdomains in the [Routing](https://github.com/MasoniteFramework/docs/tree/ba9d9f8ac3e41d58b9d92d951f92c898fb16a2a4/routing.md) documentation.

## Added new module method to Get and Post routes

By default, masonite will look for routes in the `app/http/controllers` namespace but you can change this for individual routes:

```python
Get().module('thirdpary.routes').route('/dashboard', 'DashboardController@show')
```

This will look for the controller in the `thirdparty.routes` module.

## Added Queues and Jobs

Masonite now ships with a `QueueManager` class which can be used to build queue drivers. Masonite ships with an `async` driver which sends jobs to a background thread. These queues can process Jobs which ca be created with the new `craft job` command. See the [Queues and Jobs](../useful-features/queues-and-jobs.md) documentation for more information.

