# Compiling Assets

## Introduction

Understanding that modern frameworks need to handle modern web applications. Many developers are starting to use third party packages, like Sass, to write CSS. Normally, many people who write Sass in other frameworks will need to run other third party services like Webpack or grunt. Masonite tries to make this as simple as possible and comes with Sass built in. So you just need to write Sass and it will compile into CSS when you run the server.

## Getting Started

Now although Masonite comes with the ability to compile Sass, it is deliberately missing the `libsass` dependency. This dependency takes several minutes to install and therefore was left out of the requirements to speed up the process of creating a new project.

Masonite can compile all of your Sass into CSS files when you run the server. The code needed to compile Sass is already inside the framework although it does not execute without `libsass` .

In order to activate this feature we can run:

```text
pip install libsass
```

Awesome! We're good to go. All Sass put inside `resources/static` will compile into `resources/compiled` .

## Configuration

Masonite comes with a configuration file that will allow you to tweak the locations of files that Sass will be looked for, as well as where it will compile into. This setting page can be found in `config/storage.py` . The configuration constant looks something like:

```python
SASSFILES = {
    'importFrom': [
        'storage/static'
    ],
    'includePaths': [
        'storage/static/sass'
    ],
    'compileTo': 'storage/compiled'
}
```

#### ImportFrom Setting

This setting will look for base `.sass` and `.scss` files. Base Sass files are files without a preceding underscore. So `style.scss` is a base Sass file but `_dashboard.scss` is not. Once all the base Sass files are found, it will compile them into CSS and put them in the location specifed in the `compileTo` setting.

#### IncludePaths Setting

This setting is where Masonite will look for files anytime you want to include using the `@include` keyword in your sass files. Without the correct location here, Masonite will not find any files you include in your Sass files. This setting can be a list of directory locations.

#### CompileTo Setting

This setting specifies a single directory you want all of your Sass compiled down into.

This is all setup by default for you and works as soon as you install the `libsass` dependency.

