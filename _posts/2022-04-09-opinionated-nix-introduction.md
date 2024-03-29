---
hide: false
layout: post
title: An opinionated introduction to the Nix world
description: What's what in the Nix world?
categories: [nix]
toc: false
comments: true
---

> What is "_nix_" ?

When people talk about "Nix", they usually refer to a whole ecosystem, mainly:

- `nix`, the tool
- `nixpkgs`, the package repository
- Nix, the language
- NixOS, the operating system

The starting point is `nix`, the tool, but it's important to have an idea of what all the other things are.

## `nix`, the tool

`nix` is a **command line tool** to run applications, package applications and manage all about it.
It can be compared to `apt` or `pacman`.

But _why_ use Nix?

Using `apt` or any common package manager, one can't install multiple versions of a same package.
Nix makes it possible.

This is useful at any level of usage.

As an end user, I can run any software without caring about my system configuration.
Just use 

```
nix run nixpkgs#blender
```

and Blender starts, even if it was not installed on your machine!

As a developer, I can work on multiple projects at the same time, each with different dependencies.
I can work on a NodeJS 12, a NodeJS 14 project, a Java 8 project , Java 17 project, ... all side by side, while nothing collides!

As a DevOps, I can upgrade packages of a machine without any risk of breaking it.
If any build fails while upgrading, the upgrade hasn't touched anything of the current setup, so it rollbacks for free since nothing was changed!
If the upgraded system is not working as intended, it's easy to downgrade since all the packages are still there!

### How is this possible?

What sets `nix` apart from other package managers is this very important feature:

> With Nix, libraries and applications are isolated components stored in a central place, identified by a hash of everything required for such a component.

Basically, every piece of software, libs, executables, assets, or anything, is each stored in a subfolder of `/nix/store/`.
For instance, an installation of Python could be stored in a folder `/nix/store/hb1lzaisgx2m9n29hqhh6yp6hasplq1v-python3-3.9.10`.

The first consequence is that there's no software "globally installed" in places like `/usr/bin/` or `/etc/` or `/user/lib/` anymore.
This means that you don't have to manage multiple possible sources of truth about your system packages.

The second consequence is that you can have the same libraries in different versions at the same time on your computer.
For example, to take the earlier example of CUDA, you can have CUDA in version 8.0 at path `/nix/store/someHash1-cuda-8.0/`, and CUDA in version 9.0 at path `/nix/store/someHash2-cuda-9.0/`.
This would not be possible without Nix, because you would have the CUDA files of one version installed in `/usr/lib/`, mixed with plenty of other libraries, which would make it impossible to have any other version installed as well.

Basically, it's a package manager with super powers!

If you are curious about the technicalities behind this idea, you should read [the PhD thesis of its creator](https://edolstra.github.io/pubs/phd-thesis.pdf), Eelco Dolstra.

## `nixpkgs`, the package repository

Like any package manager, in order for the end users to be able to install packages and run programs, some people (usually known as "maintainers") have to describe how those packages are made.
After it has been done by those people, anybody can refer to it, just like any package manager.

These maintainers have decided to concentrate all their efforts in one place, called `nixpkgs`, which can be found on GitHub.
In this repository, the Nix community is building a **giant tree** with all the possible packages.
Most of them are defined at the root of the tree (e.g. `cowsay` or `gcc`) but some can be found nested in some attributes (e.g. `python39Packages.pep8`).

If you are looking for a package, you can find it thanks to the [search web page](https://search.nixos.org/).

### The CI and cache of `nixpkgs` 

Note that almost all packages are built and tested on a system called Hydra, ensuring a high level of quality.

As a user, using a package built by Hydra allows Nix to download it from the Hydra cache instead of building it.
If the package is not available in the cache, worry not: Nix will run the build on your laptop, it will just take some more time.

## Nix, the language

`nixpkgs` holds all the definitions of the packages that can be installed or used via the `nix` tool.
Each package is called a **derivation**, which is basically the set of derivations it depends on and the commands it has to run to build it.
In order to write those definitions, one can use the Nix language, that the `nix` command line tool can parse and evaluate.

Nix is a _configuration language_, akin to a programming language but specifically designed for the sole purpose of defining derivations.
Basically, it's JSON with functions!

`nix` can evaluate things written in Nix:

```bash
$ echo 'builtins.mapAttrs (_: v: v + 1) {a=1; b=10;}' > test.nix
$ nix eval --file test.nix
{ a = 2; b = 11; }
```

Since Nix is functional, we can say that we evaluate **Nix expressions**.

### Builtins and libraries

`nix` provides a set of built-in functions that you can use in any Nix expression, such as `builtins.derivation` or `builtins.toString`.

`nixpkgs` defines and uses a lot of functions, such as `stdenv.mkDerivation` or `lib.lists.sort`, to help you do the most common operations.

From there, you can define and use your own functions.

You can even share it as a library of your making (we'll talk about that much later).

### Exploring with the REPL

You can play with Nix in an interactive session:

```
$ nix repl
nix-repl> builtins.mapAttrs (_: v: v + 1) { c=11;d=22; }
{ c = 12; d = 23; }
```

We will use that later to learn more about how the language works.

## NixOS

Using `nix` can be really helpful.
Instead of installing user packages, how about using it to define the whole system of your machine?
This would really helpful, because you'd define your whole system as code which is more easy to track and reproduce: you can write your configuration once, install it on any machine and you'll get the same system every time!

This would fundamentally change how we approach system administration, right?

This is the idea behind NixOS: a fully reproducible system management through code.

No more `/usr/lib`, no more `/etc/`: everything is stored and managed in `/nix/store`, even your operating system!

## Summary

We use a tool in the command line, `nix`, to run or install softwares.
Those softwares can be built thanks to the build instructions and dependency graphs in `nixpkgs`, which is written in Nix, the language.
This allows for an amazing reproducibility, which can be achieved at the level of the entire operating system of your machine using NixOS.
