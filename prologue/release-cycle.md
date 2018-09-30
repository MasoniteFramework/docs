# Release Cycle

## Introduction

The Masonite framework itself follows the RomVer versioning schema which is `PARADIGM.MAJOR.MINOR` although all Masonite packages follow the SemVer versioning schema which is `MAJOR.MINOR.BUGFIX`.

This means that a framework version of 1.3.20 may have breaking changes with version 1.4.0. Masonite uses a 6 month major release cycle in order to maintain it's state as a modern Python web framework. Each release is a solid and stable release and upgrading typically takes minimal time.

## Reasoning for RomVer over SemVer

The Masonite main repository \(the `MiraFramework/masonite` repository\) contains only the basic file structure of the application. All of the core framework functionality is inside the `MasoniteFramework/core` repository which can be updated as much as every day or once per month.

Because `MiraFramework/masonite` does not require major updates, we can follow RomVer nicely and keep the versioning number artificially lower. Any major updates to this repsository will likely just be file structure changes which should rarely happen unless there are major architectural changes.

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

Masonite is made up of three different repositories. There is

* The main repository where development is done on the repo that installs on developer's systems.
* The core repository which is where the main Masonite pip package is located.
* The craft repository where the craft command tool is located.

Major 6 month releases will be released on or after the release date when all repositories are able to be released at the same time, as well as passing all tests.

The core repository is uploaded to PyPi under the new release. At this time, everyone could theoretically install the package. Once core is released, craft is updated for any minor changes it may require. Once these two repositories are completed, the masonite repository is updated and finally released. Once this repo is released to the latest version, all future projects will default to the latest version of Masonite.

After that we will bump the version of Masonite in the documentation and verify and update any documentation that is not up to standard.

Whenever the `MasoniteFramework/craft` and `MasoniteFramework/core` repositories are released on Github, Travis CI will run tests and automatically deploy to PyPi. These major version numbers should correspond to the version of Masonite they support. For example, if the `MasoniteFramework/masonite` releases to version 1.4, `MasoniteFramework/core` should bump up to 1.4.x regardless of changes.

## Main Repository and New Projects

The main repository which is `MasoniteFramework/masonite` does not have a corresponding PyPi package and is only for installing new Masonite projects. See the `craft new` command under [The Craft Command](https://github.com/MasoniteFramework/docs/tree/ba9d9f8ac3e41d58b9d92d951f92c898fb16a2a4/the-craft-command.md) documentation. The `craft new` command will download a zip of the latest release of Masonite, unzip it and rename the folder. Once the next release for this repository is ready, it will be released but marked as a `Pre-release` and therefore will not be installable by the default `craft new` command.

This is so developers and maintainers will be able to test the new pre release with their applications. Once all QA tests have passed, it will be marked as a normal release and therefore all new applications created from there on out will be the new release.

Developers will still have the option of doing something like: `craft new project_name --version 1.6` and installing that version of Masonite.

Once all three repositories are ready for release, they will all be released on GitHub under the respective new version numbers.

