# Introduction and Installation

{% hint style="success" %}
Any developers that are using or have used Masonite

If you can take 5 minutes to complete this survey it would be very much appreciated. Just very simple questions that we can use to make Masonite an even better framework then it already is!

[Click here to start the survey](https://joemancuso.typeform.com/to/mOh1ox)
{% endhint %}

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

{% code title="terminal" %}
```text
$ sudo apt-get install python-dev libssl-dev
```
{% endcode %}

Or you may need to specify your `python3.x-dev` version:

{% code title="terminal" %}
```text
$ sudo apt-get install python3.6-dev libssl-dev
```
{% endcode %}

#### Enterprise Linux based distributions \(Fedora, CentOS, RHEL, ...\)

{% code title="terminal" %}
```text
# dnf install python-devel openssl-devel
```
{% endcode %}

## Installation

{% hint style="success" %}
Be sure to join the [Slack Channel](http://slack.masoniteproject.com) for help or guidance.
{% endhint %}

Masonite excels at being simple to install and get going. We use a simple command line tool that will become your best friend. You’ll never want to develop again without it. We call them `craft` commands.

We can download our `craft` command line tool by just running:

{% code title="terminal" %}
```text
$ pip install masonite-cli
```
{% endcode %}

If you already have craft installed, Masonite 2.1 requires `masonite-cli>=2.1.0` so you may have to run with the upgrade flag to.

{% code title="terminal" %}
```text
$ pip install masonite-cli --upgrade
```
{% endcode %}

{% hint style="warning" %}
If you are having installation issues, be sure to read the [Known Installation Issues](prologue/known-installation-issues.md) documentation.
{% endhint %}

## Creating Our Project

Great! We are now ready to create our first project. We should have the new `craft` command. We can check this by running:

{% code title="terminal" %}
```text
$ craft
```
{% endcode %}

This should show a list of command options. If it doesn't then try closing your terminal and reopening it or running it with `sudo` if you are on a UNIX machine. We are currently only interested in the `craft new` command. To create a new project just run:

{% code title="terminal" %}
```text
$ craft new project_name

#Crafting Application ...

#Application Created Successfully!

#Now just cd into your project and run

#    $ craft install

#to install the project dependencies.

#Create Something Amazing!
$ cd project_name
```
{% endcode %}

This will get the latest Masonite project template and unzip it for you. We just need to go into our new project directory and install the dependencies in our `requirements.txt` file.

## Activating Our Virtual Environment \(optional\)

You can optionally create a virtual environment if you don't want to install all of masonite's dependencies on your systems Python. If you use virtual environments then create your virtual environment by running:

{% code title="terminal" %}
```text
$ python -m venv venv
$ source venv/bin/activate
```
{% endcode %}

or if you are on Windows:

{% code title="terminal" %}
```text
$ python -m venv venv
$ ./venv/Scripts/activate
```
{% endcode %}

{% hint style="info" %}
The `python`command here is utilizing Python 3. Your machine may run Python 2 \(typically 2.7\) by default for UNIX machines. You may set an alias on your machine for Python 3 or simply run `python3`anytime you see the `python`command.

For example, you would run `python3 -m venv venv` instead of `python -m venv venv`
{% endhint %}

## Installing Our Dependencies

Now lets install our dependencies. We can do this simply by using a `craft` command:

{% code title="terminal" %}
```text
$ craft install
```
{% endcode %}

After install you are ready to create Something Amazing! Masonite folder structure is like this.

. ├── LICENSE ├── README.md ├── app │ ├── User.py │ ├── http │ │ ├── controllers [controllers](https://docs.masoniteproject.com/the-basics/controllers) │ │ │ └── WelcomeController.py │ │ └── middleware [middleware](https://docs.masoniteproject.com/advanced/middleware) │ │ ├── AuthenticationMiddleware.py │ │ ├── CsrfMiddleware.py │ │ ├── LoadUserMiddleware.py │ │ └── VerifyEmailMiddleware.py │ └── providers [service-providers](https://docs.masoniteproject.com/architectural-concepts/service-providers) ├── bootstrap │ ├── cache [caching](https://docs.masoniteproject.com/useful-features/caching) │ └── start.py ├── config │ ├── **init**.py │ ├── application.py │ ├── auth.py │ ├── broadcast.py │ ├── cache.py │ ├── database.py │ ├── mail.py │ ├── middleware.py │ ├── packages.py │ ├── providers.py │ ├── queue.py │ ├── session.py │ └── storage.py ├── craft ├── databases │ ├── migrations [database-migrations](https://docs.masoniteproject.com/orator-orm/database-migrations) │ │ ├── 2018_0109043202createuserstable.py │ │ └── **init**.py │ └── seeds │ ├── **init**.py │ ├── databaseseeder.py │ └── usertable\_seeder.py ├── requirements.txt ├── resources │ ├── \_\_init.py │ └── templates │ ├── \_\_init.py │ └── welcome.html ├── routes_ [_routing_](https://docs.masoniteproject.com/the-basics/routing) _│ └── web.py ├── storage_ [_static-files_](https://docs.masoniteproject.com/the-basics/static-files) _│ ├── compiled │ │ └── style.css │ ├── public │ │ ├── favicon.ico │ │ └── robots.txt │ ├── static │ │ ├── \_\_init.py │ │ └── sass │ │ └── style.scss │ └── uploads │ └── \_\_init_.py ├── tests [testing](https://docs.masoniteproject.com/useful-features/testing) │ ├── feature &lt;-- Add tests to your single fetures or functions. │ │ └── test\_feature\_works.py │ ├── framework │ │ ├── test\_file\_locations.py │ │ └── test\_imports.py │ └── unit &lt;-- Add tests to your unit test. │ └── test\_works.py └── wsgi.py

This command is just a wrapper around the `pip`command. This installs all the required dependencies of Masonite, creates a `.env` file for us, generates a new secret key, and puts that secret key in our `.env` file. After it’s done we can just run the server by using another `craft` command:

## Running The Server

After it’s done we can just run the server by using another `craft` command:

{% code title="terminal" %}
```text
$ craft serve
```
{% endcode %}

You can also run the server in auto-reload mode which will rerun the server when file changes are detected:

{% code title="terminal" %}
```text
$ craft serve -r
```
{% endcode %}

Congratulations! You’ve setup your first Masonite project! Keep going to learn more about how to use Masonite to build your applications.

The Masonite CLI \(also known as craft\) will try to find all the commands in your project but may not be able to. In this case you will need to call craft directly using something like:

{% code title="terminal" %}
```text
$ python craft serve -r
```
{% endcode %}

{% hint style="info" %}
You can also add a auto reloading option to the serve command by running `craft serve -r` which will reload the server whenever you save a python file.
{% endhint %}

{% hint style="success" %}
You can learn more about craft by reading [The Craft Command](https://github.com/MasoniteFramework/docs/tree/ba9d9f8ac3e41d58b9d92d951f92c898fb16a2a4/the-craft-command.md) documentation or continue on to learning about how to create web application by first reading the [Routing ](the-basics/routing.md)documentation
{% endhint %}

{% hint style="info" %}
Masonite has romantic versioning instead of semantic versioning. Because of this, all minor releases \(2.0.x\) will contain bug fixes and fully backwards compatible feature releases. Be sure to always keep your application up to date with the latest minor release to get the full benefit of Masonite's romantic versioning.
{% endhint %}

