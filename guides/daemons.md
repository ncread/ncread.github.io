--- 
layout: guide
title: "Daemons Guide"
permalink: /guides/daemons
--- 

# Daemons
Written 20260219 | Updated 20260223

***

My journey into the world of Arch Linux has introduced me to the topics of systemd, services and daemons. The init system **systemd** is a suite of building blocks for Linux systems, primarily providing a system and service manager that runs as PID (process ID) 1 and starts the rest of the system. As PID 1, systemd is responsible for a host of other functions, including daemon management.

At its core, a **daemon** refers to a "process running in the background." This encapsulates processes that automatically run without requiring any user interaction. Daemons can be socket-activated, timer-activated, path-activated, and device-activated, among others. As far as pronunciation of the word goes, some pronounce it as "demon" (which is awesome), yet it's primarily pronounced as "damon" (think Matt Damon, Johnny Damon, Damon Stoudamire).

Many existing Unix daemons end with the letter *d*, making them fairly straightforward to spot (occasionally the entire word "daemon" is present instead of just *d*, but again, very easy to spot). The *d* is simply a convention, not a requirement. Astute observers may have correctly realized that systemd *itself* is a daemon, albeit a much more vital one as it boots the system (among numerous other functions). Other daemon examples include **sshd**, an SSH daemon that stealthily listens for incoming SSH requests, and **crond**, a daemon that executes scheduled commands.

There are two main scopes of daemons that will be covered, **system daemons and user daemons**, with the main differences involving when they are run, by whom they are controlled, and their environment.
<br><br>


## System Daemon (system unit)
* Unit files located in `/etc/systemd/system/` or `/usr/lib/systemd/system/`
    * add your own user overrides and custom units to the former path, `/etc/systemd/system/`
* Runs as root by default
* managed with `sudo systemctl...`
* Starts at system boot, independent of user log in activity
* Include services that involve the entire system, not a specific user

## User Daemon (user unit)
* Unit files located in `~/.config/systemd/user/` or `/usr/lib/systemd/user/`
* managed with `systemctl --user...` (no sudo necessary)
* Runs as logged-in user (not root)
* Starts when user logs in and isolated to user's session

The `/usr/lib/systemd/system/` and `/usr/lib/systemd/user/` paths hold package-provided units, so it's wise to stay away from those.
<br><br>



## Command Line Control
Countless methods exist to manipulate both system and user daemons. The keywords used below can be used for both daemon types; however, keep in mind that system unit manipulation requires root access (sudo) and user unit manipulation requires the --user flag. Syntax for each is shown below:

##### System Daemon Syntax
```bash
sudo systemctl enable <unit>
```

##### User Daemon Syntax
```bash
systemctl --user enable <unit>
```

### Managing Daemons with `systemctl`:
The following commands are listed to show the umbrella architecture, with `<unit>` as a placeholder for the service unit of interest. The list is not comprehensive, but includes useful, common commands. Use sudo or --user appropriately. 

| Command | Purpose |
| :--- | :--- |
| `systemctl enable <unit>` | Enable unit instance (notable flags include `--system`, `--user`, `--runtime`, `--global`, or `--now` to both enable AND run now) |
| `systemctl disable <unit>` | Disable unit instance (same scope flags as in previous row) |
| `systemctl start <unit>` | Activate the unit now |
| `systemctl stop <unit>` | Deactivate the unit now |
| `systemctl reload <unit>` | Reload the unit config (if altered) without stopping the process (NOT the same as `daemon-reload` command) |
| `systemctl restart <unit>` | Stop the unit (if running) and restart it |
| `systemctl reload-or-restart <unit>` | Reload the unit if supported, otherwise restart. If unit isn't running, start it. |
| `systemctl is-enabled <unit>` | Check if the unit file is enabled. Typically returns 0 if enabled, non-zero otherwise. |
| `systemctl status <unit>` | Check current state of the unit and its recent logs |
| `systemctl daemon-reload` | Allow systemd to re-read unit files from disk after creating, editing, or deleting `.service`, `.timer`, or `.socket` files. Only considers systemd unit files. |

Notes: Units may be provided with or without their suffix (if missing, systemctl will append .service to the entered parameter). Enabling and starting units are not the same. Enabling "activates" the unit and is persistent, whereas starting runs the daemon process *now* and is temporary. When disabling, ensure the correct scope (--user, --global, etc) is taken into account. For example, a unit may be disabled for `--user` but could still be enabled in the `--global` scope. If a unit file is edited, `daemon-reload` followed by `restart`. If a service's config file is edited, `reload` or `restart`.


### Viewing Logs with `journalctl`:
The ability to print log entries from systemd's logging service (journald) is fulfilled by the `journalctl` utility, which serves as the standard method to read log messages in linux. A non-comprehensive list of useful commands is shown below:

| Command | Purpose |
| :--- | :--- |
| `journalctl` | Display journal contents accessible to calling user, beginning with oldest collected entry. `--system`, `--user` flags can be specified for more specific logs. |
| `journalctl -u <unit>` | Get logs for a specific unit |
| `journalctl -u <unit> -f` | Get logs for a specific unit in real time (follows) |
| `journalctl -b <unit>` | Get logs from current boot |
| `journalctl --list-boots` | List system boots by boot ID, date & timestamp |
| `journalctl --since "<time period>"` | Display logged messages within given time period ("1 hour ago", "2 days ago", "2026-02-01 10:00:00", etc.). Optional `--until "< >"` flag can be added to end. |



## Daemon Composition
It is now understood how to work with and manipulate daemons from the command line, but how are they even constructed in the first place? What *is* `<unit>` in the commands above? The manner in which a daemon is represented and controlled often spans across a few different files working together. Beyond the daemon scope explained above, there are numerous manners in which a corresponding service unit can be activated. 

One daemon example mentioned previously is sshd, which is a socket-activated service. Systemd binds to a specific port on the machine, holding the socket open and waiting for SSH connections. Once a connection is initiated, sshd is started by systemd and takes over the socket. Such a daemon requires a `.socket` file which defines the socket, and a `.service` file which defines the daemon to start when an SSH connection is initiated. This is advantageous since sshd doesn't have to be persistently running and taking up disk space when it's not in use.

A much simpler process involves scheduling tasks, which falls under the category of timer-activated services. In many cases cron is perfectly suitable for such a scheduling job, but the ability of a daemon to constantly run gives it plenty of viablity. Rather than a socket connection triggering the daemon, a schedule defined by the user does the job. Here, a `.service` file defines the daemon process and a `.timer` file controls when it gets started. 

So, depending on the type of service being manipulated, `<unit>` will have a different suffix (for `systemctl` commands, at least). To enable a socket-activated service (sshd, for example), run it now, and check the live logs, the following two commands would be run:

```bash
sudo systemctl enable --now sshd.socket

journalctl -u sshd.service -f
```

Similarly for a timer-activated service (using my own get_matches service, which snags tennis match statistics), enabling the service, running it now, and checking its logs from the current boot are executed as follows:

```bash
systemctl --user enable --now get_matches.timer

journalctl --user -u get_matches.service -b
```

Notice that `systemctl` deals with the unit's lifecycle, so it's manipulating the `.socket` and `.timer` files. `journalctl` on the other hand deals with the `.service` file directly since the output produced from the daemon actually running is of interest.
<br><br>



## Unit File Contents 
There are countless unit file configurations for each service type, but regardless of whether a `.service`, `.socket`, `.device`, `.mount`, `.automount`, `.swap`, `.target`, `.path`, `.timer`, `.slice`, or `.scope` config file is being constructed, the [Unit] and [Install] sections are considered generic and are typically present.

Beyond the [Unit] and [Install] sections, a unit file may also contain a type-specific section, such as a [Service] section for a `.service` file, a [Socket] section for a `.socket` file, and so on. 

From here on out the focus will be placed on the baseline structure for a `.service` file and an associated `.timer` file. Consider a shell script, `sp500.sh`, that fetches the price of the S&P 500 every 10 seconds. To schedule the execution of this file at some cadence, we could use the following unit files to create a daemon:

`sp500.service`:
```bash
[Unit]
Description=Price of S&P500
After=network-online.target

[Service]
Type=oneshot
ExecStart=/path/to/shell/script/sp500.sh
StandardOutput=journal+console
```
[Install] section is unnecessary in the `.service` file here since there is an associated `.timer` file that is being enabled. If the `.service` file were being enabled directly, it would need an [Install] section.

`sp500.timer`:
```bash
[Unit]
Description=Price of S&P500 Timer
Requires=sp500.service

[Timer]
OnBootSec=1min
OnUnitActiveSec=10sec
AccuracySec=1sec

[Install]
WantedBy=timers.target
```
Note: Any time specification (every second, daily, weekly, monthly, etc) can be defined in the [Timer] section via the `OnCalendar=` directive. Find the full manual outlining every configuration option for each section [here](https://man7.org/linux/man-pages/man5/systemd.unit.5.html).

As this would be categorized as a user daemon, `sp500.service` and `sp500.timer` would both be placed in `~/.config/systemd/user/`. To activate them, the following commands would be run:

```bash
cd /path/to/script/sp500.sh
chmod +x sp500.sh

systemctl --user daemon-reload
systemctl --user enable sp500.timer
```

It is worth noting that the newly formed existence of `sp500.service` and `sp500.timer` and the subsequent enabling of `sp500.timer` doesn't magically create a "daemon file" on disk. Enabling `sp500.timer` creates a symlink which is monitored by systemd, and at runtime the actual process is spawned with a PID. The process then is completed, culminating in updated logs or in whatever output method that has been specified. The symlink will be forever present until `systemctl --user disable sp500.timer` is run.
<br><br>



## Conclusion
Daemons are powerful tools, and in the spirit of this guide, they inarguably have their time and place for purposes such as scheduling tasks, but they're absolutely vital in many other applications. My first daemon involved task scheduling (which is why the guide includes that type of example), but I'm excited to shift over and explore path-activated daemons that monitor directories, as well as socket-activated daemons. It seems like those arenas are where daemons really shine. If there is any interest in learning more about daemons, check out the references below -- there is a ton to pore over. 

Thanks for reading. For any thoughts or questions, shoot me a message somewhere. I'd love to hear from you!

## References
* [https://man7.org/linux/man-pages/man5/systemd.unit.5.html](https://man7.org/linux/man-pages/man5/systemd.unit.5.html)
* [https://man7.org/linux/man-pages/man7/daemon.7.html](https://man7.org/linux/man-pages/man7/daemon.7.html)
* [https://www.linfo.org/daemon.html](https://www.linfo.org/daemon.html)
* [https://wiki.archlinux.org/title/Systemd](https://wiki.archlinux.org/title/Systemd)