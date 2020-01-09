# Introduction and Installation


The modern and developer centric Python web framework that strives for an actual batteries included developer tool with a lot of out of the box functionality with an extremely extendable architecture. Masonite is perfect for beginner developers getting into their first web applications as well as experienced devs that need to utilize the full potential of Masonite to get their applications done.

Masonite works hard to be fast and easy from install to deployment so developers can go from concept to creation in as quick and efficiently as possible. Use it for your next SaaS! Try it once and you’ll fall in love.

{% hint style="success" %}
If you are more of a visual learner you can watch Masonite related tutorial videos at [MasoniteCasts.com](https://masonitecasts.com)
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

* Python 3.5+
* Latest version of OpenSSL
* Pip3

{% hint style="warning" %}
All commands of python and pip in this documentation is assuming they are pointing to the correct Python 3 versions. For example, anywhere you see the `python` command ran it is assuming that is a Python 3.5+ Python installation. If you are having issues with any installation steps just be sure the commands are for Python 3.5+ and not 2.7 or below.
{% endhint %}

### Linux

If you are running on a Linux flavor, you’ll need the Python dev package and the libssl package. You can download these packages by running:

#### Debian and Ubuntu based Linux distributions

{% code title="terminal" %}
```text
$ sudo apt-get install python-dev libssl-dev python3-pip
```
{% endcode %}

Or you may need to specify your `python3.x-dev` version:

{% code title="terminal" %}
```text
$ sudo apt-get install python3.6-dev libssl-dev python3-pip
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

Masonite excels at being simple to install and get going. If you are coming from previous versions of Masonite, the order of some of the installation steps have changed a bit.

Firstly, open a terminal and head to a directory you want to create your application in. You might want to create it in a programming directory for example:

```
$ cd ~/programming
$ mkdir myapp
$ cd myapp
```

If you are on windows you can just create a directory and open the directory in the Powershell.

## Activating Our Virtual Environment \(optional\)

Although this step is technically optional, it is highly recommended. You can create a virtual environment if you don't want to install all of masonite's dependencies on your systems Python. If you use virtual environments then create your virtual environment by running:

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

## Installing Masonite

Now we can install Masonite. This will give us access to a craft command we can use to finish the install steps for us:

```
$ pip install masonite
```

Once Masonite installs you will now have access to the `craft` command line tool. Craft will become your best friend during your development. You will learn to love it very quickly :).

You can ensure Masonite and craft installed correctly by running:

```
$ craft
```

You should see a list of a few commands like `install` and `new`

## Creating Our Project

Great! We are now ready to create our first project. We should have the new `craft` command. We can check this by running:

{% code title="terminal" %}
```text
$ craft
```
{% endcode %}

We are currently only interested in the `craft new` command. To create a new project just run:

{% code title="terminal" %}
```text
$ craft new
```
{% endcode %}

This will get the latest Masonite project template and unzip it for you. We just need to go into our new project directory and install the dependencies in our `requirements.txt` file.

## Installing Our Dependencies

Now lets install our dependencies. We can do this simply by using a `craft` command. Remember that other craft command called `craft install`? 

{% code title="terminal" %}
```text
$ craft install
```
{% endcode %}

This command is just a wrapper around the `pip` or `pipenv` command. This installs all the required dependencies of Masonite, creates a `.env` file for us, generates a new secret key, and puts that secret key in our `.env` file. After it’s done we can just run the server by using another `craft` command:

## Additional Commands

Now that Masonite installed fully we can check all the new commands we have available. There are many :).

```
$ craft
```

We should see many more commands now.

## Running The Server

After it’s done we can just run the server by using another `craft` command:

{% code title="terminal" %}
```text
$ craft serve
```
{% endcode %}

Congratulations! You’ve setup your first Masonite project! Keep going to learn more about how to use Masonite to build your applications.

{% hint style="success" %}
You can learn more about craft by reading [The Craft Command](https://github.com/MasoniteFramework/docs/tree/ba9d9f8ac3e41d58b9d92d951f92c898fb16a2a4/the-craft-command.md) documentation or continue on to learning about how to create web application by first reading the [Routing ](the-basics/routing.md)documentation
{% endhint %}

{% hint style="info" %}
Masonite uses romantic versioning instead of semantic versioning. Because of this, all minor releases \(2.0.x\) will contain bug fixes and fully backwards compatible feature releases. Be sure to always keep your application up to date with the latest minor release to get the full benefit of Masonite's romantic versioning.
{% endhint %}

