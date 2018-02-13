# Controllers
#controllers

# Introduction
Controllers are a vital part of Masonite and is mainly what differs it from other Python frameworks that implement the MVC structure differently. Controllers are simply classes with methods. These methods take a `self` and a `request` parameters which are injected when the route is executed.  Controller methods can be looked at as function based views as they are simply methods inside a class and function the exact same way.

Controllers have an added benefit over straight function based views as they have a level of abstraction of creating methods are the view and inside the class which the developer can leverage to separate out logic but still contain it in a the same class.

## Defining a Controller
Its very easy to create a controller with Masonite with the help of our `craft` command tool. We can simply create a new file inside `app/http/controllers`, name the class the same name as the file and then create a class with methods. We can also use the `craft controller` command which is:

    $ craft controller DashboardController

This command will scaffold the application for us as well as import the `view` function for us which we can use to render a template.

When we run this command we now have a new class under `app/http/controllers.DashboardController` called `DashboardController`. By convention, Masonite expects that all controllers have their own file since it’s an extremely easy way to keep track of all your classes since the class name is the same name as the file but you can obviously but this class wherever you like.

## Defining a Controller Method
Controller methods are very similar to function based views in a Django application. Our controller methods should look like:

```python
def show(self, request):
    passs
```

All controller methods must have two parameters. The `self` parameter is the normal python `self` object which is just an instance of the current class and `request` is an instance of the `Request` class which is created on every call and will contain information specific to the request as well as some helper methods you’ll use when developing your applications. For more information about the `Request` class, read the `Request` class documentation.

It’s important to note that unlike other frameworks, we do not have to specify our route parameters as parameters in our controller method. We can retrieve the parameters using the `Request` class.














