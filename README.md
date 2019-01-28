# Introduction and Installation

The modern and developer centric Python web framework that strives for an actual batteries included developer tool with a lot of out of the box functionality with an extremely extendable architecture. Masonite is perfect for beginner developers getting into their first web applications as well as experienced devs that need to utilize the full potential of Masonite to get their applications done.

Masonite works hard to be fast and easy from install to deployment so developers can go from concept to creation in as quick and efficiently as possible. Use it for your next SaaS! Try it once and you’ll fall in love.

{% hint style="success" %}
![](.gitbook/assets/go_arrow_next_up_green_forward-512.png) If you are more of a visual learner you can watch[ Masonite 2.1 tutorial videos here](https://www.youtube.com/playlist?list=PLdR9bD5hyZigsHxhJ-hLGYIGXUne2JwsL).  
{% endhint %}

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
* Pip3

{% hint style="warning" %}
All commands of python and pip in this documentation is assuming they are pointing to the corresponding Python 3 versions. If you are having issues with any installation steps just be sure the commands are for Python 3.4+ and not 2.7 or below.
{% endhint %}

{% hint style="danger" %}
NOTE: Masonite does not support the Anaconda virtual environment system. In order for the craft command to work, something you will be using often, Masonite needs to be able to locate the Masonite package even in a virtual environment. If you follow the installation steps here and use Python 3's builtin virtualenv package \(`python3 -m venv venv`\) then this will work best. If you would like to add Anaconda support to Masonite then feel free to open an issue in GitHub :\)
{% endhint %}

### Linux

If you are running on a Linux flavor, you’ll need the Python dev package and the libssl package. You can download these packages by running:

#### Debian and Ubuntu based Linux distributions

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ sudo apt-get install python-dev libssl-dev
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Or you may need to specify your `python3.x-dev` version:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ sudo apt-get install python3.6-dev libssl-dev
```
{% endcode-tabs-item %}
{% endcode-tabs %}

#### Enterprise Linux based distributions \(Fedora, CentOS, RHEL, ...\)

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
# dnf install python-devel openssl-devel
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Installation

{% hint style="success" %}
Be sure to join the [Slack Channel](http://slack.masoniteproject.com) for help or guidance.
{% endhint %}

Masonite excels at being simple to install and get going. We use a simple command line tool that will become your best friend. You’ll never want to develop again without it. We call them `craft` commands.

We can download our `craft` command line tool by just running:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ pip install masonite-cli
```
{% endcode-tabs-item %}
{% endcode-tabs %}

If you already have craft installed, Masonite 2.1 requires `masonite-cli>=2.1.0` so you may have to run with the upgrade flag to.

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ pip install masonite-cli --upgrade
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="warning" %}
If you are having installation issues, be sure to read the [Known Installation Issues](prologue/known-installation-issues.md) documentation.
{% endhint %}

## Creating Our Project

Great! We are now ready to create our first project. We should have the new `craft` command. We can check this by running:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This should show a list of command options. If it doesn't then try closing your terminal and reopening it or running it with `sudo` if you are on a UNIX machine. We are currently only interested in the `craft new` command. To create a new project just run:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft new project_name
$ cd project_name
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will get the latest Masonite project template and unzip it for you. We just need to go into our new project directory and install the dependencies in our `requirements.txt` file.

## Activating Our Virtual Environment \(optional\)

You can optionally create a virtual environment if you don't want to install all of masonite's dependencies on your systems Python. If you use virtual environments then create your virtual environment by running:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ python -m venv venv
$ source venv/bin/activate
```
{% endcode-tabs-item %}
{% endcode-tabs %}

or if you are on Windows:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ python -m venv venv
$ ./venv/Scripts/activate
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
The `python`command here is utilizing Python 3. Your machine may run Python 2 \(typically 2.7\) by default for UNIX machines. You may set an alias on your machine for Python 3 or simply run `python3`anytime you see the `python`command.

For example, you would run `python3 -m venv venv` instead of `python -m venv venv`
{% endhint %}

## Installing Our Dependencies

Now lets install our dependencies. We can do this simply by using a `craft` command:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft install
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This command is just a wrapper around the `pip`command. This installs all the required dependencies of Masonite, creates a `.env` file for us, generates a new secret key, and puts that secret key in our `.env` file. After it’s done we can just run the server by using another `craft` command:

## Running The Server

After it’s done we can just run the server by using another `craft` command:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft serve
```
{% endcode-tabs-item %}
{% endcode-tabs %}

You can also run the server in auto-reload mode which will rerun the server when file changes are detected:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ craft serve -r
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Congratulations! You’ve setup your first Masonite project! Keep going to learn more about how to use Masonite to build your applications.

The Masonite CLI \(also known as craft\) will try to find all the commands in your project but may not be able to. In this case you will need to call craft directly using something like:

{% code-tabs %}
{% code-tabs-item title="terminal" %}
```text
$ python craft serve -r
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
You can also add a auto reloading option to the serve command by running `craft serve -r` which will reload the server whenever you save a python file.
{% endhint %}

{% hint style="success" %}
You can learn more about craft by reading [The Craft Command](https://github.com/MasoniteFramework/docs/tree/ba9d9f8ac3e41d58b9d92d951f92c898fb16a2a4/the-craft-command.md) documentation or continue on to learning about how to create web application by first reading the [Routing ](the-basics/routing.md)documentation
{% endhint %}

{% hint style="info" %}
Masonite has romantic versioning instead of semantic versioning. Because of this, all minor releases \(2.0.x\) will contain bug fixes and fully backwards compatible feature releases. Be sure to always keep your application up to date with the latest minor release to get the full benefit of Masonite's romantic versioning.
{% endhint %}

