# Masonite 3

Masonite 3 is a big change from Masonite 2. The most obvious change is the fact that we have dropped support for Orator and have created a new package called Masonite ORM that is intended to be a drop in replacement of Orator.

Hopefully many of you may not even tell we are no longer using Orator.

Below is a list of high level changes of things that are new or changed in Masonite 3 from Masonite 2. If you want to see how to upgrade from Masonite 2 to 3 then refer to the [Upgrade Guide]()

## Python Support

Since Python 3.9 came out we are dropping support for Python 3.5. Masonite adds some f string formats so Masonite will break on Python 3.5. Throughout Masonite 3.0 we will be upgrading the codebase to use features available to us that we were unable to do because of Python 3.5 support.

Masonite 3 supports all Python version 3.6 and above.

## Headers

In Masonite 2 the headers were largely controlled oddly. Internally they were just saved as dictionaries and then attached to the response later. Also strangely, the response headers were attached to the request class and not the response class, even though they really had nothing to do with the response class. This also presented an issue because a request header is one sent by a client and a response header is one set by your app but they were both being saved in the same place so it was impossible to be able to tell who set the header.

Now in Masonite 3 we refactored the headers to use a new `HeaderBag` class which is used to maintain `Header` classes. We put the exact same class on the response class as well so they can be managed separately.

Now if you want to set headers on the request or response and you can know which header is for which.

Also logically it makes more sense to set response headers on the response class.

This internal rewrite also negates the need to prefix headers using ` HTTP_`. 

## Cookies

Cookies were suffering the same fate as headers so we changed cookies to use the same class structure as the `HeaderBag` and there is now a `CookieJar` class that is used to maintain `Cookie` classes.

## Request Singleton

The request class has also been reworked. Before, the request class was a single class that was created when the framework booted up. This presented some challenges because the class needed to be maintained between requests. Meaning certain inputs and headers set on the request class needed to be reset once the request was over and set back to a state before the request. This obviously created some weird caching bugs between requests and instead of fixing the issues we actually just created hacky work arounds like resetting inputs. 

Now the request class is only created once a request is received. Because of this there are now certain places that the request class is no longer accessible. For example, you can no longer fetch the request class in the register method of service providers where the `wsgi` attribute is `False`. You mat also not be able to fetch the request class in some of your classes `__init__` method depending on when and where the class is being initialized.

If you are upgrading to Masonite 3 and run across an error like the request class is not found in the container then you will need to fetch the request class later down the code. 

## Class Responsibility

The request class was one of the first classes that was created for Masonite. There hasen't been much that changed on the class so the class slowly got larger and larger and took on more responsibility.

One of the things that the class was used for, like the headers, was the response status code. It did not make sense to set the response status code on the request class so now the response status is whatever status is set on the response class. Requests don't have status codes so we removed the status code on the request class all together.

## Start Method

Every Masonite project has had a bootstrap/start.py file. This is a low level method in the masonite app that really handles the entire application and responsible for the final response. It is the entry point everytime a request is received. Many of you were probably not even aware it was there or were confused on what it did.

Since the method was low level and should never be changed we moved this method into the Masonite codebase and instead of an import from the local `bootstrap.start import app` it is now an import from the masonite codebase `from masonite.wsgi import response_handler`

It is largely the same exact method but is now maintained by the masonite framework.

## Queue improvements

Masonite queueing had a simple table with not a lot of options on it. Because of this we had to make some decisions to prevent the duplication of some jobs. Like sleeping and only fetching 1 job per poll. This made the queue slower than what it should have been. So now we added more columns on the queue jobs table that will allow us to reserve some jobs to prevent duplication in other queue workers. If you still experience duplication of some jobs running via supervisor you may need to specify the option that offsets when supervisor commands are started.

## Dropping Orator

Obviously the biggest change is dropping Orator and picking up Masonite ORM. This new ORM is designed to be a drop in replacement for Orator. I have upgraded several projects to use the new ORM and I had to change very minimal code to get it to work. 

Theres a few resons we decided to drop support of Orator but the main one was that Sdispater (the creator of Orator) has his time occupied by other packages like Pendulum (which Masonite ORM still uses) as well as Poetry. These are great packages and more popular than Orator. Sdispater does not know if he will pick Orator back up but the issues and bugs have been piling up and the codebase was not up to my standard of being maintained. Myself and a few maintainers have taken the time to create a new ORM project called Masonite ORM. 

Another reason is that we now have completely creative control over the ORM side of Masonite. We don't have to go through someone who has complete control. Releases can now be scheduled whenever we want and we can add whatever features we want. This is a huge deal for Masonite.