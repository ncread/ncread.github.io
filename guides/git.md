---
layout: guide
title: "Git Guide"
permalink: /guides/git
---

# Git
## Open-Source Version Control System
Written 2026/05/08

***
## Installation
Whether you use Linux, Windows, or macOS, Git can be installed.

Linux: see [install page](https://git-scm.com/install/linux) for other kernels:
```bash
# Debian/Ubuntu
apt-get install git

# Arch
pacman -S git
```

macOS:
```bash
# must have homebrew installed
brew install git 
```

For Windows, it's honestly simplest to just navigate to the [installation page for Git](https://git-scm.com/install/windows) and download from there. I used Windows for a long time and used Git Bash, I highly recommend it if you're tied to Windows as it'll help you get comfortable with Bash. Using Powershell isn't very fun when everyone around you is using something else.


## Setup
With Git present on your machine, you'll want to set a few things before proceeding. All of your future commits will be tied to a username and email address, so those must be specified early. 

**Note**: Moving forward, it is assumed that you're using Bash or Zsh


```bash
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```
Use `--global` so that you just have to do this the one time on this specific machine. I have historically set the user.name as my github username, but your first and last name are probably better suited.

**Much more to come...**

## References
https://git-scm.com/book/ms/v2/Getting-Started-First-Time-Git-Setup