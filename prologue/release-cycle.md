# Release Cycle

## Introduction

The Masonite framework itself follows the RomVer versioning schema which is PARADIGM.MAJOR.MINOR although all Masonite packages follow the SemVer versioning schema which is MAJOR.MINOR.FEATURE/BUGFIX.

This means that a framework version of 1.3.20 will have breaking changes with version 1.4.0.

## Reasoning for RomVer over SemVer

The Masonite main repository \(the `MiraFramework/masonite` repository\) contains only the basic file structure of the application. All of the core framework functionality is inside the `MasoniteFramework/core` repository which can be updated as much as every day or once per month and therefore will follow normal SemVer. Because `MiraFramework/masonite` does not require major updates, we can follow RomVer nicely and keep the versioning number artificially lower. Any major updates to this repsository will likely just be file structure changes which should rarely happen unless there are major architectural changes.

## Releases and Release Cycles

Masonite is currently on a 1 month major release cycle. This means that once every month will be a new 1.x release. This 1 month release cycle will continue until Masonite has reached a release that is stable enough to move far into the future with that releases architecture.

Once this stable release has been achieved, Masonite will move to a 6 month major release cycle with minor releases as often as every few days to as much as every few months.

Releases are planned to be upgradable from the previous release in 10 minutes or less. If they break that requirement then they should be considered for the next Paradigm release \(the `x.0.0` release\)

### Scheduled Releases

* [x] v1.3 - February 2018
* [x] v1.4 - March 2018 
* [x] v1.5 - April 2018
* v1.6 - May 2018
* v2.0 - June 2018
* v2.1 - December 2018

### Creating Releases

Masonite is made up of three different repositories. There is

* The main repository where development is done on the repo that installs on developer's systems.
* The core repository which is where the main Masonite pip package is located.
* The craft repository where the craft command tool is located.

Major 1 month releases will be released on or after the release date when all repositories are able to be released at the same time, as well as passing all tests.

Whenever the `MasoniteFramework/craft` and `MasoniteFramework/core` repositories are released on Github, Travis CI will run tests and automatically deploy to PyPi. These major version numbers should correspond to the version of Masonite they support. For example, if the `MasoniteFramework/masonite` releases to version 1.4, `MasoniteFramework/core` should bump up to 1.4.x regardless of changes.

## Main Repository and New Projects

The main repository which is `MasoniteFramework/masonite` does not have a corresponding PyPi package and is only for installing new Masonite projects. See the `craft new` command under [The Craft Command](https://github.com/MasoniteFramework/docs/tree/ba9d9f8ac3e41d58b9d92d951f92c898fb16a2a4/the-craft-command.md) documentation. The `craft new` command will download a zip of the latest release of Masonite, unzip it and rename the folder. Once the next release for this repository is ready, it will be released but marked as a `Pre-release` and therefore will not be installable by the default `craft new` command.

This is so developers and maintainers will be able to test the new pre release with their applications. Once all QA tests have passed, it will be marked as a normal release and therefore all new applications created from there on out will be the new release.

Developers will still have the option of doing something like: `craft new project_name --version 1.5` and installing that version of Masonite.

Once all three repositories are ready for release, they will all be released on GitHub under the respective new version numbers.

