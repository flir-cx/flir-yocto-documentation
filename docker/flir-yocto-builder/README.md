flir-yocto-builder
===================

Overview
--------
This container is intended for building Yocto projects in a containerized
Ubuntu environment. The container works with both `docker` and
[`podman`](https://podman.io).

Build
-----
Start by choosing what Ubuntu version you want to use as the baseline. Projects
based on Yocto 2.5 or newer should use Ubuntu 16.04 (which is also the
default). Older projects should use 14.04. For more detailed information about
what products use what versions, consult the table below.

| Ubuntu Version | Yocto Version | Product(s)            |
|----------------|---------------|-----------------------|
| 14.04          | 2.0           | Rocky, Evander/Lennox |
| 16.04          | 2.5           | Sherlock              |
| 18.04\*        | 2.7           | FLIR One Gen 4        |

\* *Required because of a depency on* `xxd` *in meta-atmel*

Use the following command to build the container locally. Be sure to set the
FROM_TAG and container tag appropriately (16.04 is used in the example). Note
that there are extra arguments to ensure that the `yoctobuild` user inside the
container has the same user and group ID as the user who builds the image.
This in turn avoids permission problems with the build output.

~~~console
$ docker build \
    -t flir-yocto-builder:16.04 \
    --build-arg FROM_TAG=16.04 \
    --build-arg GID=$(id -g) \
    --build-arg UID=$(id -u) \
    .
~~~

Note: One can also use `buildah` to build the container by simply replacing
`docker build` with `buildah bud`. Also, **regardless of the tool, do not
forget the dot at the end of the command. It's important!**

Usage
-----
To run the container as an interactive session where it's possible to build
Yocto projects, use the following command:

~~~console
$ docker run --rm -it \
    -v "$PWD":/code \
    -v ~/.ssh:/home/yoctobuild/.ssh \
    -w /code \
    flir-yocto-builder:16.04
~~~

Note that depending on the exact setup of your Yocto project, it might be
necessary, or at least convenient, to add your ssh key to `ssh-agent`, such
that you only need to input the ssh key password once. This is especially
useful for projects relying on `repo` or similar tools. This container is
configured to automatically run the shell through `ssh-agent`, so all you need
to do is run `ssh-add` and input your ssh key password. After that the
(interactive) session will remember the key password for as long as it's
running. *NOTE: This can be a security risk if the container is left unattended
as someone else then can pose as you, using your ssh credentials.*

When using `podman`, `localhost/` needs to be prepended to the container name
and, assuming `podman` is being run as user and not root, `--userns=keep-id`
should be added to ensure correct ownership of files created by the container.
The `run` command should end up similar to this:

~~~console
$ podman run --rm -it \
    --userns=keep-id \
    -v $PWD:/code \
    -v ~/.ssh:/home/yoctobuild/.ssh \
    -w /code \
    localhost/flir-yocto-builder:16.04
~~~


Bash Alias examples
--------

Add to .bash_alias. Need to create .docker_bash_history file first for bash history to work

~~~console
alias docker16='docker run --rm -it --network=host \
    -v "$PWD":/code \
    -v ~/.ssh:/home/yoctobuild/.ssh \
    -v ~/.docker_bash_history:/home/yoctobuild/.bash_history \
    -w /code \
    flir-yocto-builder:16.04'

alias dockeraosp='docker run --rm -it --network=host \
    -v "$PWD":/code \
    -v ~/.ssh:/home/yoctobuild/.ssh \
    -v ~/.docker_bash_history:/home/yoctobuild/.bash_history \
    -w /code \
    aosp-builder'
~~~

Problems
--------

DNS
-----
The system may not use localhost as it's dns resolver, having this makes the
docker unable to resolve names, probably since the docker network is in a
different namespace.


e.g. /etc/resolv.conf should not contain
~~~/etc/resolv.conf
nameserver 127.0.0.1
~~~
