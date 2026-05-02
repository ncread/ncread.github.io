---
layout: guide
title: "uv Guide"
permalink: /guides/uv
---

# uv
## Package Manager for Python
Written 2026/05/01

***

Once an interest in programming spreads, project ideation (theoretically!) tends to follow. When projects begin to increase in complexity, a sprawling world of virtual environments, dependencies and more may rear its confounding head. For an individual in the midst of wrapping his/her head around a relatively new programming language, this discovery can be confusing and discourage further learning. Thankfully for python programmers, `uv` exists, a tool released by [Astral](https://astral.sh/) in February 2024 which serves as a significant upgrade over the previous package and project management options.

When I was first getting into building python projects of my own, I knew next to nothing about development environments. Packages were being installed globally on my machine, leading to unorganization and future confusion. Over time as I began working on group projects in school, conflict issues involving python versions, package versions and more arose, rapidly forcing my workflow to evolve. 

Research led me to the existence of virtual environments (venv) -- how cool! `venv` allowed for each project to be isolated, having its own set of packages installed in the respective directory via `pip`. In tandem these tools offered a fairly user-friendly way to set up isolated environments for each of my projects and install the necessary packages. Nevertheless, I'd constantly forget how to create an environment and I struggled remembering to actually **activate** it. Once I caught wind of `uv`, I learned the ropes, simplified my project management workflow, and haven't looked back. `uv` is written in Rust and employs caching and parallelization methods to accelerate depedency resolution and installation. The entire process feels simpler and utimately more pleasant.

This guide will run through leveraging `uv` on your machine for your project management needs, including tool installation, best practices, command line commands, and more. Let's dive in.
<br><br>


## Installation
On Linux/MacOS:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```
On Windows:
```powershell
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```
There are other installation methods, but the commands above are recommended by Astral. To verify installation/check the version of your `uv` package, open a new terminal window and run `uv --version`.

Running `uv` in the terminal yields a help menu with commands and flags that are helpful to see together in one place.

## Project Initialization
Once `uv` has been successfully installed and the list of available commands has been perused, a new project can be initialized. Navigate to the appropriate directory and run the following command:
```bash
uv init <project folder name>
```
For example, running `uv init new_project` will create a folder called *new_project* in your current directory and initialize the project there. If you navigate to some directory and want the project to live there (not creating another dedicated folder), simply run `uv init` without the project folder name argument.

The version of python used within the environment can also be specified during initialization:
```bash
uv init <project folder name> --python <version>
```

Within the directory where the project was initialized, some files should now exist:

```text
.
├── .git/
├── .gitignore
├── main.py
├── pyproject.toml
├── .python-version
└── README.md
```

Running `uv init` will create a git repository and the additional five files seen above. Once a project command is executed, a virtual environment will be created (`.venv/`) in the directory as well as a `uv.lock` file. `main.py` is just a simple "Hello from {directory}" program serving as a placeholder. 

If a specific python version is not specified in the `uv init` command, `.python-version` by default holds the version of python used on your machine, but this can be specified by running the following:

```bash
uv python pin <version>
```

Whatever python version is specified will then be included within the virtual environment on the machine.

## Working with Dependencies
With an initialized project ready for development, dependencies can be managed using a few key commands. Just for example's sake, to add numpy to your project and then remove it, run the following:

```bash
uv add numpy
uv remove numpy
```

Specifying the package version is also possible, as is declaring a version constraint:
```bash
uv add 'numpy==2.4.0'
uv add 'numpy>=2.2.0'
```

Dev-only dependencies may also be added by using the dev flag, which will create a *dev* dependency group within the `pyproject.toml` file:
```bash
# using ruff as an example
uv add ruff --dev
uv run ruff check <file> #applying ruff to a file
```

uv also provides a method to upgrade packages. Rather than removing a package and re-adding an updated version of it, the following command can be run:
```bash
uv lock --upgrade-package numpy
```

When packages are being added, removed, or upgraded, you'll notice that a few files update accordingly within the project directory. The `pyproject.toml` metadata file has a dependency field that will reflect whatever change took place, and the `uv.lock` file will update as well.

## Syncing Your Environment
If you're strictly working on a single machine and adjusting dependencies from there, syncing is taken care of via `uv add` and `uv remove` (so don't worry too much about it). If `pyproject.toml` is edited manually for whatever reason or `/.venv` is deleted, action must be taken to ensure the development environment is synced. `uv sync` checks out the `uv.lock` file and appropriately syncs the environment with the file's contents:

```bash
uv sync
```

Syncing is also necessary if a project repository that is using `uv` is cloned. Since `/.venv` is not committed to git, the environment must be recreated from the lockfile. There are some legacy methods of working with pip commands and freezing`requirements.txt` files, but sidestepping those and leveraging `pyprpject.toml` is cleaner.

## Running a File
Whether or not the project was cloned or was born locally, after applying changes to the dependencies installed within the project, `/.venv` and `uv.lock` should now both be present. Running a python file within the project environment is performed as such:
```bash
uv run <python file>
```
`uv run` is outwardly simple, but numerous steps are taken on the backend prior to .py file execution:

1. `uv` locates the `pyproject.toml` file within the directory
2. ensures that `/.venv` is present (and constructs it, if not)
3. verifies that the environment is synced with `uv.lock` (and syncs it, if not)
4. executes the file within the environment

Previous workflows required multiple commands to do what `uv run` does. This is where the utility of `uv` really shines.


## Miscellanous Helpful Commands
* `uv tree` reveals the project's dependency tree (just like the `tree` command in linux)
* `uv tool install <tool>` rather than add the tool to a specific project environment, this will add the tool globally
    * ex. `uv tool install ruff`, then `ruff check <file>` to apply it
* `uv tool list` lists installed tools
* `uv python list` lists installed python versions
* `uv venv` creates `/.venv` (this is taken care of by other commands as laid out previously, but the abillity to manually create it does exist)

## References
* [https://docs.astral.sh/uv/](https://docs.astral.sh/uv/)

