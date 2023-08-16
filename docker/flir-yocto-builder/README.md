flir-yocto-builder
===================

Overview
--------
This container is intended for building Yocto projects in a containerized
Ubuntu environment. The container should work with both `docker` and
[`podman`](https://podman.io).
(poorly tested with podman)

Build
-----
Start by choosing what Ubuntu version you want to use as the baseline. Projects
based on Yocto 2.5 or newer should use Ubuntu 16.04 or 14.04. Older projects should use 14.04 (which is also the default).<br>
Newer projects based upon Yocto 3.3 should use Ubuntu 20.04 (or 18.04)<br>
For more detailed information about what products use what versions, consult the table below.

| Ubuntu Version | Yocto Version | Product(s)            | Dockerfile     |
|----------------|---------------|-----------------------|----------------|
| 14.04 or 16.04 | 2.5           | Sherlock              | Dockerfile     |
| 20.04 or 18.04 | 3.3           | All new               | Dockerfile     |

Use the following command to build the container locally. Be sure to set the
FROM_TAG and container tag (*16.04, 18.04, 20.04*) appropriately (20.04 is used in the example). Note
that there are extra arguments to ensure that the `yoctobuild` user inside the
container has the same user and group ID as the user who builds the image.
This in turn avoids permission problems with the build output.

~~~console
$ docker build \
    -t flir-yocto-builder \
    --build-arg GID=$(id -g) \
    --build-arg UID=$(id -u) \
    .

OR (ubuntu 20 based):
$ docker build \
    -t flir-yocto-builder:20.04 \
    --build-arg FROM_TAG=20.04 \
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
$ cd <your yocto tree root>
$ docker run --rm -it \
    -v "$PWD":"$PWD" \
    -v ~/.ssh:/home/yoctobuild/.ssh \
    -w "$PWD" \
    flir-yocto-builder

(note the arguments; 
--user argument maps generated files owner to host owner. Not needed if 
the docker image was built on the same computer
first "-v" makes docker build and host built result compatible 
second -v is optional (if you like to fetch code inside container
)

(you may want to use a specific tag, i.e. flir-yocto-builder:20.04)
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
    -v $PWD:"$PWD" \
    -v ~/.ssh:/home/yoctobuild/.ssh \
    -w "$PWD" \
    localhost/flir-yocto-builder
~~~


Bash Alias examples
--------

Add to .bash_alias. Need to create .docker_bash_history file first for bash history to work

~~~console
alias docker20='docker run --rm -it \
    -v "$PWD":"$PWD" \
    -v ~/.ssh:/home/yoctobuild/.ssh \
    -v ~/.docker_bash_history:/home/yoctobuild/.bash_history \
    -w "$PWD" \
    flir-yocto-builder:20.04'

alias docker16='docker run --rm -it \
    -v "$PWD":"$PWD" \
    -v ~/.ssh:/home/yoctobuild/.ssh \
    -v ~/.docker_bash_history:/home/yoctobuild/.bash_history \
    -w "$PWD" \
    flir-yocto-builder:16.04'
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

You may want to add "--network=host" as argument when starting container
