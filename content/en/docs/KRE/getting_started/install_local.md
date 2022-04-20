---
title: "Run KAI Server"
linkTitle: "Run KAI Server"
weight: 30
description: >
  Run KAI Server in a local environment
---

## Requirements

In order to start development on this project you will need these tools:

- **gettext**: OS package to fill templates during deployment
- **minikube**: the local version of Kubernetes to deploy KAI Server
- **helm**: K8s package manager. Make sure you have v3+
- **yq**: YAML processor. Make sure you have v4+
- **hostctl**: Cli tool to manage entries in `/etc/hosts` file

## Local Environment

KAI Server has a tool called `krectl` to handle common actions you will need during development.

All the configuration needed to run KAI Server locally can be found in the `.krectl.conf` file. Usually, you'll be ok with the default values. Check Minikube's parameters if you need to tweak the resources assigned to it.

Run `help` to get info for each command:

```sh
$> krectl.sh [command] --help

// Outputs:

  krectl.sh -- a tool to manage KRE environment during development.

  syntax: krectl.sh <command> [options]

    commands:
      dev     creates a complete local environment and auto-login to frontend.
      start   starts minikube kre profile.
      stop    stops minikube kre profile.
      login   creates a login URL and open your browser automatically on the admin page.
      build   calls docker to build all images inside minikube.
      deploy  calls helm to create install/upgrade a kre release on minikube.
      delete  calls kubectl to remove runtimes or versions.

    global options:
      h     prints this help.
      v     verbose mode.
```

### Install local environment

To install KRE in your local environment:

```sh
$> ./krectl.sh dev [--hard]
```

This will install everything you need into the namespace specified in your development `.krectl.conf` file.  
The `Hard` flag will force to start from 0 all installation processes.

### Login to local environment

First, remember to edit your `/etc/hosts`, see `./krectl.sh dev` output for more details.

**NOTE**: If you have the [hostctl](https://github.com/guumaster/hostctl) tool installed, updating `/etc/hosts` will be done automatically too.

Now you can access the admin UI visiting the login URL that will be opened automatically by executing the following script:

```sh
$> ./krectl.sh login [--new]
```

You will see an output like this:

```sh
⏳ Calling Admin API...

 Login done. Open your browser at:

 🌎 http://admin.kre.local/signin/c7d024eb-ce35-4328-961a-7d2b79ee8988

✔️  Done.
```
