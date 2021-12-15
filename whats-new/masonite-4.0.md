# Masonite 4

The primary goal of Masonite was to restructure the Masonite internals so that I can make sure Masonite is around for the next 5-10 years. Masonite started as a learning project and went through version 1-4 in 4 years. In the very beginning, there really was no clear vision for the direction of the project so the foundation was not well thought out. It wasn't until version 2 that there was a clear vision and major structural changes. **Version 1 looked incredibly different than version 3 yet they were built on the same foundation. You can see the issue ..** There would be no way we would be able to get to version 6+ on the same foundation as version 1. Masonite needed a complete foundational rewrite. **Meet Version 4.**

Masonite originally started as a learning project in December of 2017. Over the years it has been a giant learning experience and Masonite has turned into something I never could have imagined. Because of the success that Masonite had, I found it was important to take a step back and make sure that the foundation of Masonite was built in a way that allowed Masonite to be as successful as it can be. Masonite 4 was written from complete scratch with everything I've learned over 4 years of maintaining the framework.

We will attempt here to go through everything new in Masonite 4. This documentation section will be a mix of both **what** is new and **why** it is new. Most of Masonite 4 changes are actually internal. Much of the interface and features of Masonite are identical to previous versions.

# Foundation

The foundation of Masonite has changed a lot. The IOC container, service providers and features are all the same but how they are registered and how features work together has changed.

As many of you are familiar, when you maintain any application for several years and you go through requirement changes and scope creap it leads to technical debt. 

With Masonite, there were times where I would come into contact with a very large company that wanted to use Masonite that maybe didn't have a feature they needed and I would have to build the feature over the weekend or even during the night. I did this to try to capture the company to use Masonite. Many of those companies did go along with using Masonite but creating features this rapidly tends to lead to poor feature structure. Rapid development of features naturally tends to lead to high coupling of code. High coupling of code tends to lead to more difficult maintanance.

One of the goals of the Masonite 4 release is to have very very low coupled code. So one of the things I did with Masonite 4 was to make  sure each feature's foundation was identical. So each feature has a main manager class which acts as the main interface for each feature and each feature is built around drivers. This makes expanding features simple. New service? New driver.

# Internal Project Structure

One of the downsides to Masonite development was that in the beginning, I had no clue what I was doing. Masonite 1 and Masonite 2 were completely different frameworks but they were built on the same foundation. See the issue?

One of the problems with this is that I thought it would be a great idea to simply make a project inside the Masonite core's code so I can more easily test certain features. The problem with this is that you can't use one of the PIP's most powerful features. Development mode. This is because the Masonite config directory would override the import inheritance. So the alternative with Masonite was to have to uninstall and install it everytime we made a change. This made developing with Masonite a lot longer and harder. 

The tricky part is the only way to solve this issue is making everything in the project configurable. This is where the new Kernel file comes in. So previously we had controllers inside `app/http/controllers`, middleware inside `app/middleware`, etc. 

Now with the Kernel file we register all of our paths in this file. We can then use those registrations to do what we need to do.

# Project Structure

The project structure slightly changed. Masonite 4 gets rid of the `app/http` directory as there was no real benefit to the directory. The `http` directory was removed and we now just have `app/controllers` and `app/middleware`.

The `resources/templates` directory has been removed. This has been replaced with a `templates` directory.

# Features

Surpringly, the features in Masonite 4 look almost identical to Masonite 3. There are some new or improved features such as: 
* A new Notifications feature
* Improved broadcasting feature to include a new features like private and presence channels as well as easy authentication support.
* More powerful email support based around Mailable classes.
* Several packages like Validation and task scheduling have been moved into the core repository from being separate packages.

# Facades

Masonite 4 brings a new concept called facades. Facades are a simple proxy class that call a deeper implementation through Masonite's service container.

Here would be the same code, one using auto resolving and one using facades:

```python
from masonite.response import Response

def show(self, response: Response):
    return response.redirect('/')
```

and using a facade:

```python
from masonite.facades import Response

def show(self):
    return Response.redirect('/')
```

You can see its a bit cleaner and also allows us to use these facades internally to reference other implementations without having code be too rigid.

# Service Providers

Service providers have changed slightly. In Masonite 3, providers were:

* Register each provider to the container
* Then looped through to call the register method on each of them
* If the WSGI attribute was false it would call the boot method on those providers
* The server would then boot up
* On a request the service providers where WSGI was true have their boot method ran in order

The new service provider flow looks like this:
* Register each provider to the container
* Provider register methods are called as each one registers to the container
* The server will then boot up
* On each request, all the boot methods are ran in order

The end goals are the same but the order the flow happens has been modified. This is largely an internal change.


