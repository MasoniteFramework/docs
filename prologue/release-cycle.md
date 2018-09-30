# Release Cycle

## Introduction

The Masonite framework itself follows the RomVer versioning schema which is `PARADIGM.MAJOR.MINOR` although all Masonite packages follow the SemVer versioning schema which is `MAJOR.MINOR.BUGFIX`.

This means that a framework version of 1.3.20 may have breaking changes with version 1.4.0. Masonite uses a 6 month major release cycle in order to maintain it's state as a modern Python web framework. Each release is a solid and stable release and upgrading typically takes minimal time.

## Reasoning for RomVer over SemVer

The Masonite main repository \(the `MasoniteFramework/masonite` repository\) contains only the basic file structure of the application. All of the core framework functionality is inside the `MasoniteFramework/core` repository which can be updated as much as every day or once per month.

Because `MasoniteFramework/masonite` does not require major updates, we can follow RomVer nicely and keep the versioning number artificially lower. Any major updates to this repository will likely just be file structure changes which should rarely happen unless there are major architectural changes.

## Releases and Release Cycles

**TL;DR: Minor release every 2 weeks. Major release every 6 months. Beta releases every 1 month starting 4 months out from major release \(i.e: September, October, November is beta 1, 2 and 3 and then feature freeze for beta 3\). Then the major release on the 4th month \(December\). Similar concept for June releases.**

Masonite is currently on a 6 month major release cycle. This means that once six months will be a new 2.x release.

## Releases and Release Cycles

Masonite is currently on a 6 month major release cycle. This means that once six months will be a new 2.x release. 


Releases are planned to be upgradable from the previous release in 30 minutes or less. If they break that requirement then they should be considered for the next Paradigm release \(the `x.0.0` release\)

### Scheduled Releases

* [x] v1.3 - February 2018
* [x] v1.4 - March 2018
* [x] v1.5 - April 2018
* [x] v1.6 - May 2018
* [x] v2.0 - June 2018
* v2.1 - December 2018

### Creating Releases

Masonite is currently on a 6 month release cycle and is made up of three different repositories:

* The main repository where development is done on the repo that installs on developer's systems.

* The "core" repository which is where the main Masonite pip package is located.
* The craft repository where the craft command tool is located.

### Development Releases

When developing new releases there will be 3 beta releases before the final release. So for example if Masonite 2.1 is being released in December, there will be a beta 1 release in September, beta 2 in October, beta 3 in November and then the final 2.1 release in December.

These releases will be called something like:

* v2.1.0b1
* v2.1.0b2
* v2.1.0b3
* v2.1.0

The core repository is uploaded to PyPi under the new release number. At this time, everyone could theoretically install the package. Once core is released, craft is updated for any minor changes it may require. Once these two repositories are completed, the masonite repository is updated and finally released. Once this repo is released to the latest version, all future projects will default to the latest version of Masonite.

{% hint style="warning" %}
The third beta release will be a complete feature freeze and not introduce any new features.
{% endhint %}

### Development Fixes

Sometimes we might come out with a beta release and something is broke on it that was not caught in tests and we need to "hotfix" them. These will be called "post" releases since they will append a `postN` to the beta release and will be numbered like so:

* v2.1.0b1.post1

A full release structure from 2.0 to 2.1 might be something like:

* v2.0.16 - `Normal minor release`
* v2.1.0b1 - `First beta release`
* v2.1.0b1.post1 - `Bugfix on beta 1 release`
* v2.1.0b2 - `Second beta release`
* v2.1.0b2.post1 - `Bugfix on beta 2 release`
* v2.1.0b2.post2 - `Bugfix on beta 2 release`
* v2.1.0b3 - `Third beta release`
* v2.1.0 - `Final release`

After that we will bump the version of Masonite in the documentation and verify and update any documentation that is not up to date or standard.

Whenever the `MasoniteFramework/craft` and `MasoniteFramework/core` repositories are released on Github, Travis CI will run tests and automatically deploy to PyPi. These major version numbers should correspond to the version of Masonite they support. For example, if the `MasoniteFramework/masonite` releases to version 1.4, `MasoniteFramework/core` should bump up to 1.4.x regardless of changes.

## Main Repository and New Projects

The main repository which is `MasoniteFramework/masonite` does not have a corresponding PyPi package and is only for installing new Masonite projects. See the `craft new` command under [The Craft Command](https://github.com/MasoniteFramework/docs/tree/ba9d9f8ac3e41d58b9d92d951f92c898fb16a2a4/the-craft-command.md) documentation. The `craft new` command will download a zip of the latest release of Masonite, unzip it and rename the folder. Once the next release for this repository is ready, it will be released but marked as a `Pre-release` and therefore will not be installable by the default `craft new` command.

This is so developers and maintainers will be able to test the new pre release with their applications. Once all QA tests have passed, it will be marked as a normal release and therefore all new applications created from there on out will be the new release.

Developers will still have the option of doing something like: `craft new project_name --version 1.6` and installing that version of Masonite.

Once all three repositories are ready for release, they will all be released on GitHub under the respective new version numbers.

## Testing New Beta Releases

You may want to test new beta releases to help with any existing bugs. Upgrade guides will be release between the beta 3 release and the final release so it might not be possible to upgrade your existing application within a beta 1 or beta 2 release.

You may still create new applications though since those should work pretty much the same as the previous release in terms of installation. Here are the installation instructions to test new beta releases:

### Craft New

You'll need to first craft new the project but use the develop branch:

```text
$ craft new project_name --branch develop
```

### Requirements.txt

You may need to update this file as well to change the masonite version to the latest beta version. You can find the latest beta version on [GitHub](https://github.com/MasoniteFramework/core/releases).

```text
waitress==1.1.0
masonite==2.1.0b1.post3
```

### Install

Now we just need to install the application like normal. Here we will first create and activate the virtual environment:

```text
$ python3 -m venv venv
$ source venv/bin/activate
$ craft install
```

{% hint style="warning" %}
If you're on Windows you'll need to activate the virtual environment with the `./venv/Scripts/activate` command.
{% endhint %}

Great! You now have the latest release of Masonite. Documentation may or may not be up to date. Check the latest version number in the documentation

