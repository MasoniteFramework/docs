When contributing to Masonite, please first discuss the change you wish to make via a GitHub issue. Starting with a GitHub issue allows all contributors and maintainers to have a chance to give input on the issue. It creates a public forum to discuss the best way to implement the issue, fix, or improvement. It also creates an open discussion on if the issue will even be permitted to be in the project.

Please note we have a code of conduct, please follow it in all your interactions with the project. You can find it in this in this documentation as well as all of Masonite's repositories.

## Getting Started

The framework has 2 main parts: The "masonite" repo and the "cookie-cutter" repo.

The `MasoniteFramework/cookie-cutter` repository is the main repository that will install when creating new projects using the `project start` command. This is actually a full Masonite project. When you run `project start` it simply reaches out to this GitHub repo, fetches the zip and unzips it on your computer. Not much development will be done in this repository and won't be changed unless something requires changes in the default installation project structure.

The `MasoniteFramework/core` repository is deprecated and development has been moved into `MasoniteFramework/masonite`. This repository contains all the development of Masonite and contains all the releases for Masonite. If you need to fix a bug or add a new feature then this is the repository to fork and make your changes from.

The `MasoniteFramework/craft` is deprecated. This was where the craft CLI tool lived that has since been moved into the `masonite` repository.

## Getting the Masonite cookie-cutter repository up and running to be edited

[You can read about how the framework flows, works and architectural concepts here](https://masoniteframework.gitbooks.io/docs/content/request-lifecycle.html)

This repo is simple and will be able to be installed following the installation instruction in the README.

* Fork the `MasoniteFramework/cookie-cutter` repo.
* Clone that repo into your computer:
  * `git clone http://github.com/your-username/cookie-cutter.git`
* Checkout the current release branch \(example: `develop`\)
  * `git checkout -b develop`
* You should now be on a `develop` local branch.
* Run `git pull origin develop` to get the current release version.
* From there simply create your feature branches \(`<feature|fix>-<issue-number>`\) and make your desired changes.
* Push to your origin repository:
  * `git push origin change-default-orm`
* Open a pull request and follow the PR process below

## Editing the Masonite repository

The trick to this is that we need it to be pip installed and then quickly editable until we like it, and then pushed back to the repo for a PR. Do this only if you want to make changes to the core Masonite package

To do this just:

* Fork the `MasoniteFramework/masonite` repo,
* Clone that repo into your computer:
  * `git clone http://github.com/your-username/masonite.git`
* Activate your masonite virtual environment \(optional\)
  * Go to where you installed masonite and activate the environment
* While inside the virtual environment, `cd` into the directory you installed core.
* Run `pip install -e .` from inside the masonite directory. This will install masonite as a pip package in editable mode. Any changes you make to the codebase will immediately be available in your project. You may need to restart your development server.
* Any changes you make to this package just push to your feature branch on your fork and follow the PR process below.

## Comments

Comments are a vital part of any repository and should be used where needed. It is important not to overcomment something. If you find you need to constantly add comments, you're code may be too complex. Code should be self documenting \(with clearly defined variable and method names\)

## Types of comments to use

There are 3 main type of comments you should use when developing for Masonite:

### Module Docstrings

All modules should have a docstring at the top of every module file and should look something like:

```python
"""This is a module to add support for Billing users."""
from masonite.request import Request
...
```

Notice there are no spaces before and after the sentence.

### Method and Function Docstrings

All methods and functions should also contain a docstring with a brief description of what the module does

For example:

```python
def some_function(self):
    """This is a function that does x action. 

    Then give an exmaple of when to use it 
    """
    ... code ...
```

### Methods and Functions with Dependencies

Most methods will require some dependency or parameters. You must specify them like this:

```python
def route(self, route, output):
    """Load the route into the class. This also looks for the controller and attaches it to the route.

    Arguments:
        route {string} -- This is a URI to attach to the route (/dashboard/user).
        output {string|object} -- Controller to attach to the route.

    Returns:
        self
    """
```

And if your dependencies are objects it should give the path to the module:

```python
def __init__(self, request: Request, csrf: Csrf, view: View):
    """Initialize the CSRF Middleware

    Arguments:
        request {masonite.request.Request} -- The normal Masonite request class.
        csrf {masonite.auth.Csrf} -- CSRF auth class.
        view {masonite.view.View} -- The normal Masonite view class.

    Returns:
        self
    """
    pass
```

### Code Comments

Code comments should be left to a MINIMUM. If your code is complex to the point that it requires a comment then you should consider refactoring your code instead of adding a comment. If you're code MUST be complex enough that future developers may not understand it, add a `#` comment above it

For normal code this will look something like:

```python
# This code performs a complex task that may not be understood later on
# You can add a second line like this
complex_code = 'value'

perform_some_complex_task()
```

## Pull Request Process

**Please read this process carefully to prevent pull requests from being rejected.**

1. You should open an issue before making any pull requests. Not all features will be added to the framework and some may be better off as a third party package or not be added at all. It wouldn't be good if you worked on a feature for several days and the pull request gets rejected for reasons that could have been discussed in an issue for several minutes.
2. Ensure any changes are well commented and any configuration files that are added have docstring comments on the variables it's setting. See the comments section above.
3. Update the `MasoniteFramework/docs` repo \(and the README.md inside `MasoniteFramework/masonite` repo if applicable\) with details of changes to the UI. This includes new environment variables, new file locations, container parameters, new feature explanations etc.
4. Name your branches in the form of `<feature|fix>/<issue-number>`. For example if you are doing a bug fix and the issue number is `576` then name your branch `fix/576`. This will help us locate the branches on our computers at later dates. If it is a new feature name it `feature/576`.
5. You **must** add unit testing for any changes made before the PR will be merged. If you are unsure how to write unit tests of the repositories listed above, you may open the pull request anyway and we will add the tests together.
6. Increase the version numbers in any example files (like setup.py) and to the new version that this Pull Request would represent. The versioning scheme we use is [SEMVER](https://semver.org/).
7. The PR must pass the GitHub actions that run on pull requests. The Pull Request can be merged in once you have a successful review from two other collaborators, or one review from a maintainer. 

## Branching

Branching is also important. Depending on what fixes or features you are making you will need to branch from \(and back into\) the current branch. Branching for Masonite simple:

1\) All of Masonite repositories, packages, etc. follow the same basic branching flow. 

2\) Each repository has: a current release branch, previous release branches, a master branch and a develop branch. 

3\) The current release branch is what the current release is on. So for example, Masonite is on version 2.3 so the current release branch will be `2.3`. 

4\) Previous release branches are the same thing but instead are just previous versions. So if Masonite is currently on `4.0` then the previous release branches could be `3.0`, `2.1`, `2.2`, etc. 

5\) The master branch is a staging branch that will eventually be merged into the current release branch. Here is where all non breaking changes will be staged \(new non breaking features and bug fixes\). 

6\) The `develop` branch is a staging branch **to the next major version**. So for example, Masonite will be on version `4.0`. If you have an idea for a feature but it will break the existing feature then you will branch from \(and back into\) the `develop` branch. This branch will eventually be merged into `4.1` branch and become apart of the next major version when that is released.

### Examples:

For example, if you want to make a new feature and you know it will not break anything \(like adding the ability to queue something\) then you will branch from the master branch following the Pull Request flow from above. The PR will be open to the master branch. This feature will then be merged into the current release branch and be released as a new minor version bump \(`4.1.0`\).

Bug fixes that do not break anything is the same process as above.

New features will be the same process but be branched off of the `develop` branch. You will make your change and then open a pull request to the `develop` branch. This is a long running branch and will be merged once the next major version of Masonite is ready to be released.

## Code of Conduct

### Our Pledge

In the interest of fostering an open and welcoming environment, we as contributors and maintainers pledge to making participation in our project and our community a harassment-free experience for everyone, regardless of age, body size, disability, ethnicity, gender identity and expression, level of experience, nationality, personal appearance, race, religion, or sexual identity and orientation.

### Our Standards

Examples of behavior that contributes to creating a positive environment include:

* Using welcoming and inclusive language
* Being respectful of differing viewpoints and experiences
* Gracefully accepting constructive criticism
* Focusing on what is best for the community
* Showing empathy towards other community members

Examples of unacceptable behavior by participants include:

* The use of sexualized language or imagery and unwelcome sexual attention or advances
* Trolling, insulting/derogatory comments, and personal or political attacks
* Public or private harassment
* Publishing others' private information, such as a physical or electronic

  address, without explicit permission

* Other conduct which could reasonably be considered inappropriate in a

  professional setting

### Our Responsibilities

Project maintainers are responsible for clarifying the standards of acceptable behavior and are expected to take appropriate and fair corrective action in response to any instances of unacceptable behavior.

Project maintainers have the right and responsibility to remove, edit, or reject comments, commits, code, wiki edits, issues, and other contributions that are not aligned to this Code of Conduct, or to ban temporarily or permanently any contributor for other behaviors that they deem inappropriate, threatening, offensive, or harmful.

### Scope

This Code of Conduct applies both within project spaces and in public spaces when an individual is representing the project or its community. Examples of representing a project or community include using an official project e-mail address, posting via an official social media account, or acting as an appointed representative at an online or offline event. Representation of a project may be further defined and clarified by project maintainers.

### Enforcement

Instances of abusive, harassing, or otherwise unacceptable behavior may be reported by contacting the project team at joe@masoniteproject.com. All complaints will be reviewed and investigated and will result in a response that is deemed necessary and appropriate to the circumstances. The project team is obligated to maintain confidentiality with regard to the reporter of an incident. Further details of specific enforcement policies may be posted separately.

Project maintainers who do not follow or enforce the Code of Conduct in good faith may face temporary or permanent repercussions as determined by other members of the project's leadership.

### Attribution

This Code of Conduct is adapted from the [Contributor Covenant](http://contributor-covenant.org), version 1.4, available at [http://contributor-covenant.org/version/1/4](http://contributor-covenant.org/version/1/4/)

