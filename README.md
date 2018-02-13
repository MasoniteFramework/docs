# Introduction

The Masonite framework is an amazing Python web framework inspired by Laravel.  Masonite strives on beautifully elegant syntax, out of the box functionality and easily extendable. Masonite works hard to be fast and easy from install to deployment so developers can work as efficient as possible. Try it once and you’ll fall in love.

## Requirements

In order to use Masonite, you’ll need:

* Python 3.4+
* Pip

### Linux

If you are running on a Linux flavor, you’ll need a few extra packages. You can download these packages by running:

```
$ sudo apt-get install python-dev
```

Or you may need to specify your python version

```
$ sudo apt-get install python3.6-dev
```

## Installation

Masonite works at being simple to install and get going. We use a simple command line that will become your best friend.  You’ll never want to develop again without it. We call it the `craft` command line tool.

We can download our `craft` command line tool by just running:

```
$ pip install masonite-cli
```

**You may have to use sudo if you are on a UNIX machine**

Great! We are now ready to create our first project. We should have the new `craft` command. We can check this by running:

```
$ craft
```

This should show a list of command options. We are currently only interested in the `craft new` command. To create a new project just run:

```
$ craft new project_name
```

This will get the latest Masonite project template and unzip it for you. We just need to go into our new project directory and install the dependencies in our `requirements.txt` file.  If you use virtual environments then create your virtual environment. We can do this simply by using a `craft` command:

```
$ cd project_name
$ craft install
```

Let this install all the required dependencies of Masonite. After it’s done we can just run the server by using another `craft` command:

```
$ craft serve
```

Congratulations! You’ve setup your first Masonite project! Keep going to learn more about how to use Masonite to build your applications.