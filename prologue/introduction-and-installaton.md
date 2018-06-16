# Introduction and Installaton

The modern and developer centric Python web framework that strives for an actual batteries included developer tool with a lot of out of the box functionality with an extremely extendable architecture. Masonite is perfect for beginner developers getting into their first web applications as well as experienced devs that need to utilize the full expotential of Masonite to get their applications done.

Masonite works hard to be fast and easy from install to deployment so developers can go from concept to creation in as quick and efficiently as possible. Use it for your next SaaS! Try it once and you’ll fall in love.

## Some Notable Features Shipped With Masonite

* Easily send emails with the Mail Provider and the SMTP and Mailgun drivers.
* Send websocket requests from your server with the Broadcast Provider and Pusher and Ably drivers.
* IOC container and auto resolving dependency injection.
* Service Providers to easily add functionality to the framework.
* Extremely simple static files configured and ready to go.
* Active Record style ORM called Orator.
* Extremely useful command line tools called craft commands.
* Extremely extendable.

These, among many other features, are all shipped out of the box and ready to go. Use what you need when you need it.

## Requirements

In order to use Masonite, you’ll need:

* Python 3.4+
* Pip3

{% hint style="warning" %}
All commands of python and pip in this documentation is assuming they are pointing to the corresponding Python 3 versions. If you are having issues with any installation steps just be sure the commands are for Python 3.4+ and not 2.7 or below.
{% endhint %}

### Linux

If you are running on a Linux flavor, you’ll need the Python dev package and the libssl package. You can download these packages by running:

```text
$ sudo apt-get install python-dev libssl-dev
```

Or you may need to specify your `python3.x-dev` version:

```text
$ sudo apt-get install python3.6-dev libssl-dev
```

## Installation

Masonite excels at being simple to install and get going. We use a simple command line tool that will become your best friend. You’ll never want to develop again without it. We call them `craft` commands.

We can download our `craft` command line tool by just running:

```text
$ pip install masonite-cli
```

{% hint style="danger" %}
You may have to use sudo if you are on a UNIX machine
{% endhint %}

{% hint style="warning" %}
Be sure this pip command is pointing at your Python 3.4+ installation. If you are having installation issues, be sure to read the [Known Installation Issues](known-installation-issues.md) documentation.
{% endhint %}

## Creating Our Project

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

## Activation Our Virtual Environment

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
The `python`command here is utilizing Python 3. Your machine may run Python 2 \(typically 2.7\) by default. You may set an alias on your machine for Python 3 or simply run `python3`anytime you see the `python`command.

For example, you would run `python3 -m venv venv` instead of `python -m venv venv`
{% endhint %}

## Installation Our Dependencies

Now lets install our dependencies. We can do this simply by using a `craft` command:

```text
$ craft install
```

This command is just a wrapper around the `pip`command. This installs all the required dependencies of Masonite, creates a `.env` file for us, generates a new secret key, and puts that secret key in our `.env` file. After it’s done we can just run the server by using another `craft` command:

### Python 3.7

Two of the libraries that Masonite uses are currently not up to date with Python 3.7 installation. These libraries have old versions of `.pyc` files inside their distributions and need to be installed outside of the normal install workflow. Installing for Python 3.7 will currently be:

```text
$ pip install pycparser
$ pip install git+https://github.com/yaml/pyyaml.git
$ craft install
```

### Running The Server

After it’s done we can just run the server by using another `craft` command:

```text
$ craft serve
```

You can also run the server in auto-reload mode which will rerun the server when file changes are detected:

```text
$ craft serve -r
```

Congratulations! You’ve setup your first Masonite project! Keep going to learn more about how to use Masonite to build your applications.

{% hint style="info" %}
You can also add a auto reloading option to the serve command by running `craft serve -r` which will reload the server whenever you save a python file.
{% endhint %}

{% hint style="success" %}
You can learn more about craft by reading [The Craft Command](https://github.com/MasoniteFramework/docs/tree/ba9d9f8ac3e41d58b9d92d951f92c898fb16a2a4/the-craft-command.md) documentation or continue on to learning about how to create web application by first reading the [Routing ](../the-basics/routing.md)documentation
{% endhint %}

{% hint style="info" %}
Masonite has romantic versioning instead of semantic versioning. Because of this, all minor releases \(2.0.x\) will contain bug fixes and fully backwards compatible feature releases. Be sure to always keep your application up to date with the latest minor release to get the full benefit of Masonite's romantic versioning.
{% endhint %}

