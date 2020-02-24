---
title: "Windows Dev Machine Pt I"
date: 2020-02-24T17:22:02Z
draft: false
---

This isn't a story about how macOS is suddenly horrible or how Linux makes for
a terrible desktop environment. This is a story about a guy who spent a ton
of cash on a gaming PC...and then realised that gaming is largely a pointless
waste of time. Unless there's an awesome new release. Or he goes into a bi-annual
cycle of madness and decides to replay Fallout 4 with a hundred mods in pursuit of the
playthrough to end all playthroughs. Or there's a rainy weekend that makes
several hours of Anno 1800 a candidate for entertainment over Netflix. Or he
falls into a sprial of self-loathing and resubs to Eve Online. Plus, I have
an amazing Drevo BladeMaster Pro keyboard with CherryMX black switches attached
to the machine. Who wouldn't want to use that?

The long and the short of it is that having Windows 10 Pro as the default OS
on my shiny desktop is the desirable base config, as the freedom to flip between
games, photo editing, gaming, working and side projects for a best of all worlds
box is the end goal.

There are a ton of ways to proceed into building a development machine without
banging another OS onto the primary drive, either overwriting Windows or
in a dual-boot config. Windows 10 Pro gives you access to HyperV, so you could
pick your Linux of choice and spin up a powerful VM that will run in full screen
mode. Easy. You could run the Windows Subsystem for Linux (WSL) and have a
command line environment with Atom running within Windows and your dev tools
running in WSL. You could run [multipass](https://multipass.run/) and have some
fun with cloud-init and Ansible (everything resolves to multipass-name.mshome.net).
Or, you could decide to leave your Bash/ZSH world behind and run your tools
natively on Windows, using Microsoft products and apps/language installers
written for the OS.

I've played with the WSL, runnning a desktop in HyperV and having some multipass
VM's running provisioned with cloud-init and Ansible. I decided to go for the
native route.

Something that I don't necessarily consider native is [chocolately](https://chocolatey.org/),
which feels like Brew for Windows machines. I won't dwell on this, but I wanted
Terraform and Hugo, and I opted to use it to get those two packages installed.

Seeing as I needed a terminal, I opted for
[Windows Terminal (Preview)](https://devblogs.microsoft.com/commandline/introducing-windows-terminal/).
It does what it says on the tin. It's a terminal. It understands what `ls` is
(and does `ll` by default). Mostly it just needs to understand what `hugo server -D`
or `hugo deploy` need to do, as well as how to run `terraform` commands.

Next up was Git. As I use GitHub everywhere, it was easy enough to find and
download [GitHub Desktop](https://desktop.github.com/). It just works, whether
cloning existing repos, pushing new repos or rebasing, it makes these processes
pretty quick and easy.

An editor was also easy: [Atom](https://atom.io/) with minimal language plugins
to support what I use and enjoy.

To complement this, I have support for the following languages installed using
their native Windows installers:

* [Go 1.13.8](https://golang.org/)
* [Python 3.8.1](https://www.python.org)
* [Ruby 2.6.5](https://www.ruby-lang.org/en/)

I've also installed [Docker](https://www.docker.com/), and I have
[multipass](https://multipass.run) running for the purpose of banging up any
supporting services that I don't want to run locally. Most stuff will likely
slot neatly into a Docker container however. My `awscli` tool is also native,
with the necessary variables set up under my user's environment.

I do have WSL running with an Ubuntu 18.04 distro hooked into it. `snap` doesn't
work, but it does give me access to some tools that I can't live without, such
as nmap and dig (I cannot for the life of me remember how `nslookup` works). I also
have `kubectl` available there as well as `doctl`.

I'm looking forward to learning more about Windows by using it for something more
than firing up a game and spending hours in lalaland pretending to be king of the
world (or a cowboy). After years on macOS and Linux, it's fun to tinker with
something that is both so familiar and yet so different to what I remember. Now
that I have my personal site set up, I can spend some time doing actual work on this
box. Hopefully, once I climb into building Docker images and some Kubernetes
deployments, I can hopefully convince myself that an upgrade to an AMD Ryzen 3 3950X
(from my 3700X) will be worth it for a boatload of threads on a development desktop.
