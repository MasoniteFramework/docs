# How To Contribute

## Introduction

There are plenty of ways to contribute to open source. Many of which don't even rely on writing code. A great open source project should have excellent documentation and have as little bugs as possible. Below I will explain how to contribute to this project in different ways.

This is not an exhaustive list and not the only ways to contribute but they are the most common. If you know of other ways to contribute then please let us know.

## Contributing to Development

Of course the project requires contributions to the main development aspects but it's not the only way. But if you would like to contribute to development then a great way to get started is to simply read through this documentation. Get acquainted with how the framework works, how [Controllers](../the-basics/controllers.md) and [Routing](../the-basics/routing.md) work and read the [Architectural Concepts](../architectural-concepts/request-lifecycle.md) documentation starting with the [Request Lifecycle](../architectural-concepts/request-lifecycle.md), then the [Service Providers](../architectural-concepts/service-providers.md) and finally the [Service Container](../architectural-concepts/service-container.md).

It would also be good to read about the [Release Cycle](release-cycle.md) to get familiar with how Masonite does releases \(SemVer and RomVer\).

## Become a Feature Maintainer

Feature Maintainers are people who are in charge of specific features \(such as [Caching](../useful-features/caching.md) or [Creating Packages](../advanced/creating-packages.md)\). These developers will be in charge of reviewing PR's and merging them into the development branch and also have direct contact with the repository owner to discuss.

Feature maintainers must already have significant contributions to development of the repository they are trying to be a Feature Maintainer for. Although they do not have to be contributors to the actual feature they plan to maintain.

## Comment the Code

If you don't want to touch the code and instead want to just look at it and figure it out, contribute some comments! Comments are an excellent way for future developers to read and understand the framework. Masonite strives on being extremely commented. Although most of the code itself does not need to be commented, some of the classes, modules, methods and functions do \(although a lot of them already are\).

Comments don't affect the working code so if you want to get used to contributing to open source or you just don't quite understand what a class method is doing or you are afraid of contributing and breaking the project \(there are tests\) then contributing comments is right for you!

## Write Tests

The [Masonite pip packages](https://pypi.org/search/?q=masonite) require testing \(The main repository does not\). If you want to search through all the tests in the tests directories of those repositories and write additional tests and use cases then that will be great! There are already over 100 tests but you can always write more. With more testing comes more stability. Especially as people start to contribute to the project. Check the tests that are already there and write any use cases that are missing. These tests can be things such as special characters in a url or other oddities that may not have been thought of when using TDD for that feature.

## Contribute to Tutorials

Once familiar with the project \(by either contributing or by building application using the framework\) it would be excellent if you could write or record tutorials and put them on [Medium](http://medium.com) or [YouTube](http://youtube.com). In order for the framework to be successful, it needs to have a plethora of documentation even outside of this documentation. It needs to have notoriety and if people are seeing the framework pop up in their favorite locations they will be more inclined to use the framework and contribute to it as well.

Plus there will be fantastic tutorials out there for beginners to find and watch and you could also build a following off the back of Masonite.

## Fix the Documentation

This documentation is fantastic but there are spots where it could be improved. Maybe we haven't explained something fully or something just doesn't make sense to you. Masonite uses [Gitbook.com](http://gitbook.com) to host it's documentation and with that you are able to comment directly on the documentation which will start a discussion between you and the documentation collaborators. So if you want to cycle through the documentation page by page and get acquainted with the framework but at the same time contribute to the documentation, this is perfect for you.

## Report Bugs

If you just don't want to contribute code to the main project you may instead simply report bugs or improvements. You can go ahead and build any of your applications as usual and report any bugs you encounter to the [GitHub.com](https://github.com/MasoniteFramework/masonite/issues) issues page.

## Squash Bugs

Look at the issues page on [GitHub.com](https://github.com/MasoniteFramework/masonite/issues) for any issues, bugs or enhancements that you are willing to fix. If you don't know how to work on them, just comment on the issue and Joseph Mancuso or other core contributors will be more than happy explaining step by step on how you can go about fixing or developing that issue.

## Build a Community

If you have a large following on any social media or no following at all, you can contribute by trying to build up a following around Masonite. Any open source project requires an amazing community around the framework. You can either build up a community personally and be the leader of that community or you can simply send them to Masonite's GitHub repository where we can build up a community around there.

## Build Community Software Around Masonite

Another idea is to use Masonite to build applications such as a screencast website like [LaraCasts.com](https://laracasts.com) or an official Masonite website or even a social network around Masonite. Every great framework needs it's "ecosystem" so you may be apart of that buy building these applications with the Masonite branding and logos. Although copying the branding requires an OK from Joseph Mancuso, as long as the website was built with Masonite and looks clean it shouldn't be a problem at all.

## Answer Questions from the Community

Questions will come in eventually either through the GitHub issues or through websites like StackOverflow. You could make it a priority to be the first to answer these peoples questions or if you don't know the answer you can redirect one of the core maintainers or contributors to the question so we can answer it further.

## Review Code on Pull Requests

Most pull requests will sit inside GitHub for a few days while it gets quality tested. The main `develop` branch pull requests could sit there for as long as 6 months and will only be merged in on releases. With that being said, you can look at the file changes of these pull requests and ensure they meet the community guidelines, the API is similar to other aspects of the project and that they are being respectful and following pull requests rules in accordance with the [Contributing Guide](contributing-guide.md) documentation.

## Discuss Issues

Every now and then will be a `requires discussion` label on an issue or pull request. If you see this label then be sure to add your thoughts on an issue. All issues are open for discussion and Masonite strives off of developer input so feel free to enter a discussion.

