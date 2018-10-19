# Optimization

## Introduction

Masonite can handle most of the application that will be built with it. Python is clearly not the fastest language so if it is brute speed to manage 100,000 simultaneous connections than no application written in Python will suit your needs. Those types of application will typically require some Node.js server and asynchronous tasks on multiple servers but if you are going with a Python framework than it is likely you will not need that much juice.

## WSGI Server

Out of the box, Masonite is designed to run as a great development environment. There are a few trade off's that need to be made in order to stably run Masonite on all operating systems. Masonite defaults to running as the WSGI server with Waitress. This was chosen because waitress is a WSGI server that runs on both Windows and Mac and is powerful enough to run most application during development.

As your application is entering the deployment phase you should look away from running the server with Waitress and look to more production ready options such as uWSGI and Gunicorn. Gunicorn has been tested to run faster than uWSGI both using their default options but uWSGI may be able to be tweaked in a way that is faster dependent on your application.

Because Masonite is simply a WSGI application, any deployment tutorials you research should simply be how to setup a WSGI application on the platform you are trying to deploy to. 

So remember, Waitress is great for development but a better WSGI server like Gunicorn should be used when deploying to production.

