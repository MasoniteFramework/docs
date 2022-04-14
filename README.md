
{% hint style="warning" %}
You are browsing the `development` version of the documentation for an upcoming version of Masonite. It is subject to change.
{% endhint %}


Stop using old frameworks with just a few confusing features. Masonite is the developer focused dev tool with all the features you need for the rapid development you deserve. Masonite is perfect for beginners getting their first web app deployed or advanced developers and businesses that need to reach for the full fleet of features available.

Masonite works hard to be fast and easy from install to deployment so developers can go from concept to creation in as quick and efficiently as possible. Use it for your next SaaS! Try it once and you’ll fall in love.

{% hint style="success" %}
If you are more of a visual learner you can watch Masonite related tutorial videos at [MasoniteCasts.com](https://masonitecasts.com)
{% endhint %}

## Some Notable Features Shipped With Masonite

* Mail support for sending emails quickly.
* Queue support to speed your application up by sending jobs to run on a queue or asynchronously.
* Notifications for sending notifications to your users simply and effectively.
* Task scheduling to run your jobs on a schedule (like everyday at midnight) so you can set and forget your tasks.
* Events you can listen for to execute listeners that perform your tasks when certain events happen in your app.
* A BEAUTIFUL Active Record style ORM called Masonite ORM. Amazingness at your fingertips.
* Many more features you need which you can find in the docs!

These, among many other features, are all shipped out of the box and ready to go. Use what you need when you need it.

## Requirements

In order to use Masonite, you’ll need:

* Python 3.7+
* Latest version of OpenSSL
* Pip3

{% hint style="warning" %}
All commands of python and pip in this documentation is assuming they are pointing to the correct Python 3 versions. For example, anywhere you see the `python` command ran it is assuming that is a Python 376+ Python installation. If you are having issues with any installation steps just be sure the commands are for Python 3.7+ and not 2.7 or below.
{% endhint %}

### Linux

If you are running on a Linux flavor, you’ll need the Python dev package and the libssl package. You can download these packages by running:

#### Debian and Ubuntu based Linux distributions

{% code title="terminal" %}
```text
$ sudo apt install python3-dev python3-pip libssl-dev build-essential python3-venv
```
{% endcode %}

Or you may need to specify your `python3.x-dev` version:

{% code title="terminal" %}
```text
$ sudo apt-get install python3.7-dev python3-pip libssl-dev build-essential python3-venv
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
Be sure to join the [Discord Community](https://discord.gg/DWW5E9uM) for help or guidance.
{% endhint %}

Masonite excels at being simple to install and get going. If you are coming from previous versions of Masonite, the order of some of the installation steps have changed a bit.

Firstly, open a terminal and head to a directory you want to create your application in. You might want to create it in a programming directory for example:

```text
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
The `python` command here is utilizing Python 3. Your machine may run Python 2 \(typically 2.7\) by default for UNIX machines. You may set an alias on your machine for Python 3 or simply run `python3` anytime you see the `python` command.

For example, you would run `python3 -m venv venv` instead of `python -m venv venv`
{% endhint %}

# Installation

First install the Masonite package:

```
$ pip install masonite
```

Then start a new project:

```
$ project start .
```

This will create a new project in the current directory.

{% hint style="info" %}
If you want to create the project in a new directory (e.g. `my_project`) you must provide the directory name with `project start my_project`.
{% endhint %}

Then install Masonite dependencies:

```
$ project install
```

{% hint style="info" %}
If you have created the project in a new directory you must go to this directory before running `project install`.
{% endhint %}

Once installed you can run the development server:

```
$ python craft serve
```

Congratulations! You’ve setup your first Masonite project! Keep going to learn more about how to use Masonite to build your applications.
