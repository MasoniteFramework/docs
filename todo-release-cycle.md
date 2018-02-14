# Release Cycle

# Introduction

The Masonite framework itself follows the RomVer versioning schema which is PARADIGM.MAJOR.MINOR although all Masonite packages follow the SemVer versioning schema which is MAJOR.MINOR.FEATURE/BUGFIX.

This means that a framework version of 1.3.20 will have breaking changes with version 1.4.0.

## Cycle

Masonite is currently on a 1 month major release cycle. This means that on the first of every month will be a new 1.x release. This 1 month release cycle will continue until Masonite has reached a release that is stable enough to move far into the future with that releases architecture.

Once this stable release has been achieved, Masonite will move to a 6 month major release cycle with minor releases as often as every few days to as much as every few months.

## Scheduled Releases

* v1.3 - March 2018
* v1.4 - April 2018
* v1.5 - May 2018
* v2.0 - June 2018
* v2.1 - December 2018

## Creating Releases

Masonite is made up of three different repositories. There is 

* The main repository where development is done on the repo that installs on peoples systems.
* The core repository which is where the main Masonite pip package is located.
* The cli repository where the craft command tool is located.

Major releases will be released on after the release date when all repositories are able to be released at the same time, as well as passing all tests.

Whenever the `MasoniteFramework/cli` and `MasoniteFramework/core` repositories are released on Github, Travis CI will run tests and automatically deploy to PyPi.

The main repository which is `MasoniteFramework/masonite` which does not have a corresponding PyPi package and is only for installing new Masonite projects. See the `craft new` command under "The Craft Command" documentation.

Once all three repositories are ready for release, they will all be released on GitHub under the new version number.




