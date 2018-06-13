# Known Installation Issues

## I ran pip install masonite-cli but it didn't gave me a permission error

You are likely running this command on a UNIX based machine like Mac or Linux. In that case you should either run it again with a sudo command or a user command flag:  


```text
$ sudo pip install masonite-cli
```

or

```text
$ pip install --user masonite-cli
```

## I'm getting this weird ModuleNotFound idna issue when running the craft new command

You may get a strange error like:

```text
pkg_resources.DistributionNotFound: The 'idna<2.7,>=2.5' distribution was not found and is required by requests
```

The simple fix may just be to run:

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
You may have to run sudo to remove it and you may need to close your terminal to get it work
{% endhint %}

## I installed masonite-cli successfully but the craft command is not showing up

If you installed everything successfully and running:

```text
$ craft
```

Shows an error that it can't be found then try closing your terminal and opening it again. This should refresh any commands that were recently installed

