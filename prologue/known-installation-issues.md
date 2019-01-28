# Known Installation Issues

## Introduction

There are no known Masonite specific issues known and if there are then there should be an issue open for them on GitHub. With that being said, some users may experience some difficulties with installing Masonite simply because their computer environment is not the norm, they have never setup Python and may have configured it incorrectly or they have not used Python in a while and have an old version.

Before you get started reading through this FAQ make sure you have:

* Python 3.4+
* Pip3

Ensure you are installing masonite-cli with `pip3` and not `pip`.

## I ran pip install masonite-cli but it gave me a permission error?

You are likely running this command on a UNIX based machine like Mac or Linux. In that case you should either run it again with a sudo command or a user command flag:  


```text
$ sudo pip install masonite-cli
```

or

```text
$ pip install --user masonite-cli
```

## I pip installed masonite-cli but running craft doesn't work?

If you ran:

```text
$ pip install masonite-cli
```

and then run:

```text
$ craft
```

and get something like:

```text
-bash: craft: command not found
```

then try closing your terminal and reopening it. If that doesn't work then you may be running a pip version connecting to Python 2.7. Try uninstalling it and reinstalling it using pip3:

```text
$ pip uninstall masonite-cli
$ pip3 install masonite-cli
```

If that does not work then you may have to run sudo:

```text
$ pip3 uninstall masonite-cli
$ sudo pip3 install masonite-cli
```

## I'm getting this weird ModuleNotFound idna issue when running the craft new command

You may get a strange error like:

```text
pkg_resources.DistributionNotFound: The 'idna<2.7,>=2.5' distribution was not found and is required by requests
```

The simple fix may just be to run:

```text
pip install --upgrade requests
```

If that doesn't work we can just go back to the lower idna version:

```text
pip install idna==2.6
```

If that does not fix the issue then continue reading. 

If the above fix did not work then this likely means you installed masonite-cli using the Python 2.7 pip command. Out of the box, all Mac and Linux based machines have Python 2.7. If you run:

```text
$ python --version
```

you should get a return value of:

```text
Python 2.7.14
```

But if you run:

```text
$ python3 --version
```

you should get a return value of:

```text
Python 3.6.5
```

Now pip commands are similar:

```text
$ pip --version
pip 10.0.1 /location/of/installation (python 2.7)

$ pip3 --version
pip 10.0.1 /location/of/installation (python 3.6)
```

Notice here we are using 2 different Python installations.

So if you are getting this error you should uninstall masonite-cli from pip and reinstall it using pip3:

```text
$ pip uninstall masonite-cli
$ pip3 install masonite-cli
```

{% hint style="info" %}
You may have to run sudo to remove and install it and you may need to close your terminal to get it work if you are using a UNIX machine.
{% endhint %}

## I installed masonite-cli successfully but the craft command is not showing up

If you installed everything successfully and running:

```text
$ craft
```

Shows an error that it can't be found then try closing your terminal and opening it again. This should refresh any commands that were recently installed

If you still have errors and on a UNIX based machine try running:

```text
$ sudo pip3 install masonite-cli
```

## I'm getting a module urlib has no attribute urlopen

You likely ran:

```text
$ craft new project_name
```

and hit this weird snag that throws this ambiguous error. You might think this is because of a Python version issue but craft is designed to work on Python 2.7 and 3.4+ \(although 2.7 and not thoroughly tested\) and you're years of Python experience would make you right but this is special. If you are getting this error then that means you are likely on a UNIX machine, Mac right?

The problem is that your machine does not have sufficient permissions to access these external calls from the command line because your machine does not have permission to do so. You will have to give you machine the command to do so by running:

```text
$ /Applications/Python\ 3.6/Install\ Certificates.command
```

or whatever your Python 3 version is in the middle. Now try running:

```text
$ craft new project_name
```

and it should work great!



