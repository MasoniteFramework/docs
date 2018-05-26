# Introduction and Installaton

Masonite is the rapid application Python development framework that strives for: beautiful and elegant syntax, actual batteries included with a lot of out of the box functionality, and extremely extendable. Masonite works hard to be fast and easy from install to deployment so developers can go from concept to creation in as quick and efficiently as possible. Try it once and you’ll fall in love.

## Some Notable Features Shipped With Masonite

* Easily send emails with the Mail Provider and the SMTP and Mailgun drivers.
* Send websocket requests from your server with the Broadcast Provider and Pusher and Ably drivers.
* IOC container and auto resolving dependency injection.
* Service Providers to easily add functionality to the framework.
* Extremely simple static files configured and ready to go.
* Active Record style ORM called Orator.
* An extremely useful command line tool called craft commands.
* Extremely extendable.

These, among many other features, are all shipped out of the box and ready to go. Use what you need when you need it.

## Requirements

In order to use Masonite, you’ll need:

* Python 3.4+
* Pip

### Linux

If you are running on a Linux flavor, you’ll need the Python dev package. You can download this package by running:

```text
$ sudo apt-get install python-dev
```

Or you may need to specify your `python3.x-dev` version:

```text
$ sudo apt-get install python3.6-dev
```

## Installation

Masonite works at being simple to install and get going. We use a simple command line that will become your best friend. You’ll never want to develop again without it. We call it the `craft` command line tool.

We can download our `craft` command line tool by just running:

```text
$ pip install masonite-cli
```

{% hint style="danger" %}
You may have to use sudo if you are on a UNIX machine
{% endhint %}

Great! We are now ready to create our first project. We should have the new `craft` command. We can check this by running:

```text
$ craft
```

This should show a list of command options. If it doesn't then try closing your terminal and reopening it or running it with `sudo` if you are on a UNIX machine. We are currently only interested in the `craft new` command. To create a new project just run:

```text
$ craft new project_name
$ cd project_name
```

This will get the latest Masonite project template and unzip it for you. We just need to go into our new project directory and install the dependencies in our `requirements.txt` file.

You can optionally create a virtual environment if you don't want to install all of masonite's dependencies on your systems Python. If you use virtual environments then create your virtual environment by running:

```text
$ python -m venv venv
$ source venv/bin/activate
```

or if you are on Windows:

```text
$ python -m venv venv
$ ./venv/Scripts/activate
```

{% hint style="info" %}
The `python` command here is utilizing Python 3. Your machine may run Python 2 \(typically 2.7\) by default. You may set an alias on your machine for Python 3 or simply run `python3` anytime you see the `python` command. 

For example, you would run  `python3 -m venv venv` instead of  `python -m venv venv`
{% endhint %}

Now lets install our dependencies. We can do this simply by using a `craft` command:

```text
$ craft install
```

This command is just a wrapper around the `pip` command. This installs all the required dependencies of Masonite, creates a `.env` file for us, generates a new secret key, and puts that secret key in our `.env` file. After it’s done we can just run the server by using another `craft` command:

```text
$ craft serve
```

Congratulations! You’ve setup your first Masonite project! Keep going to learn more about how to use Masonite to build your applications. 

{% hint style="success" %}
You can learn more about craft by reading [The Craft Command](https://github.com/MasoniteFramework/docs/tree/ba9d9f8ac3e41d58b9d92d951f92c898fb16a2a4/the-craft-command.md) documentation or continue on to learning about how to create web application by first reading the [Routing ](../the-basics/routing.md)documentation
{% endhint %}



