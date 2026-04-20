---
layout: guide
title: "uv Guide"
permalink: /guides/uv
---

# uv 
## Package Manager for Python
Written 2026 | Updated 2026

***

Once an interest in programming spreads, project ideation (theoretically!) tends to follow. When projects begin to increase in complexity, a sprawling world of virtual environments, dependencies and more may rear its confounding head. For an individual in the midst of wrapping his/her head around a relatively new programming language, this discovery can be confusing and discourage further learning. Thankfully for python programmers, `uv` exists, a tool released by [Astral](https://astral.sh/) in February 2024 which serves as a significant upgrade over the previous package and project management options.

When I was first getting into building python projects of my own, I knew next to nothing about development environments. Packages were being installed globally on my machine, leading to unorganization and future confusion. Over time as I began working on group projects in school, conflict issues involving python versions, package versions and more arose, rapidly forcing my workflow to evolve. 

Research led me to the existence of virtual environments (venv) -- how cool! `venv` allowed for each project to be isolated, having its own set of packages installed in the respective directory via `pip`. In tandem these tools offered a fairly user-friendly way to set up isolated environments for each of my projects and install the necessary packages. Nevertheless, I'd constantly forget how to create an environment and I struggled remembering to actually **activate** it. Once I caught wind of `uv`, I learned the ropes, simplified my project management workflow, and haven't looked back. The entire process feels more streamlined and more pleasant.

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
There are other installation methods, but the command above is recommended by Astral. To verify installation/check the version of your `uv` package, open a new terminal window and run `uv --version`.

Running `uv` in the terminal yields a help menu with commands and flags that are helpful to see together in one place.

## Project Initialization
Once `uv` has been successfully installed and the list of commands has been perused, a new project can be initialized. Navigate to the appropriate directory and run the following command:
```bash
uv init <project folder name>
```
For example, running `uv init new_project` will create a folder called new_project and initialize the project there. If you navigate to some directory and want the project to live there (not creating another dedicated folder), simply run `uv init` without the project folder name argument.

There now exist some files in the directory where the project was initialized.

```text
.
├── .git/
├── .gitignore
├── main.py
├── pyproject.toml
├── .python-version
└── README.md
```

Running `uv init` created a git repository and the additional five files seen above. Once a project command (we're getting to those) is executed, a virtual environment will be created (`.venv/`) in the directory as well as a `uv.lock` file. `main.py` is just a simple "Hello from {directory}" program serving as a placeholder.


## Adding/Removing Packages (Dependencies)
With an initialized project ready for development, dependencies can be managed using a few key commands. To add numpy to your project and then remove it (just for example's sake), run the following:

```bash
uv add numpy
uv remove numpy
```

Specifying the package version is also possible:
```bash
uv add 'numpy==2.4.0'
```

uv also provides a method to upgrade packages. Rather than removing a package and re-adding an updated version of it, the following command can be run:
```bash
uv lock --upgrade-package numpy
```

When packages are being added, removed, or upgraded, you'll notice that a few files update accordingly within the project directory. The `pyproject.toml` metadata file has a dependency field that will reflect whatever change took place, and the `uv.lock` file will update as well (recall that this file doesn't appear until a project command is run).


## Project Commands

Wow, you made it far! The rest is still under development. Thanks for your patience.
<!-- what happens in uv run, compare that to running uv sync + source venv chenanigans -->
<!--  -->


<!-- ## Syncing Your Environment, Other Helpful Commands, maybe include a full code block display of commands used to run something -->



<!-- uv python pin <version> to update version of python in the environment, then run uv sync to update packages -->
<!-- or when you initialize the project, uv init {project name} --python {version} -->