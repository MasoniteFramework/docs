Masonite Tinker is a powerful REPL (Read, Evaluate, Print and Loop) environment for the Masonite
framework. It's a supercharged Python interactive shell with access to the container, models and
helpers.

Tinker allows you to interact with your entire Masonite project on the command line, including
models, jobs, events, and more. To enter the Tinker environment, run the tinker command:

```
python craft tinker
```

This will open a Python shell with the application container (under the `app` variable), the application models and
some helpers imported for you.

Finally you can get an enhanced experience by using the Tinker IPython shell.
[IPython](https://ipython.org/) is an improved Python shell offering some interesting features:

- Syntax highlighting
- Tab completion of python variables and keywords, filenames and function keywords
- Input history, persistent across sessions
- Integrated access to the pdb debugger and the Python profiler
- and much more...

You just need to use `-i` option and install IPython if not installed yet (`pip install IPython`):

```
python craft tinker -i
```

# Configuration

## Auto-loading Models

By default your app models are loaded from the location configured in your project Kernel.
You can override the directory to load models from with the `-d` flag. It should
be a path relative to your project root. For example you can run the following command if your
models are located in a `models/` folder located at your project root:

```
python craft tinker -d models/
```

## Startup script

You can use `PYTHONSTARTUP` environment variable to add a script that you want to run at the
beginning of the shell session.

With IPython you can use this variable or put some Python scripts
in `~/.ipython/profile_default/startup/`. IPython will run those scripts for you at the beginning
of the shell session.
