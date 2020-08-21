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
Default used ubuntu version is 14.04. FLIR flir-cx build is successfully tested
with this configuration. (also tested with ubuntu 16.04) 

Use the following command to build the container locally. 
Note that there are extra arguments to ensure that the `yoctobuild` user 
inside the container has the same user and group ID as the user who builds 
the image.
This in turn avoids permission problems with the build output.

~~~console
$ docker build \
    -t flir-yocto-builder \
    --build-arg GID=$(id -g) \
    --build-arg UID=$(id -u) \
    .

(OR

$ docker build \
    -t flir-yocto-builder:16.04 \
    --build-arg FROM_TAG=16.04 \
    --build-arg GID=$(id -g) \
    --build-arg UID=$(id -u) \
    .
)
~~~

(It should be possible to use other Ubuntu versions 14,16,18 when using
2:nd syntax. Refer to Dockerfile content for details)

Note: One can also use `buildah` to build the container by simply replacing
`docker build` with `buildah bud`. Also, **regardless of the tool, do not
forget the dot at the end of the command. It's important!**

Usage
-----
To run the container as an interactive session where it's possible to build
Yocto projects, use the following command:

~~~console
cd <your yocto project root>
docker run --rm -it \
    -v "$PWD":/"$PWD" \
    -w "$PWD" \
    flir-yocto-builder

OR:

cd <your yocto project root>
docker run --rm -it --user $(id -u):$(id -g) \
    -v "$PWD":/"$PWD" \
    -v ~/.ssh:/home/yoctobuild/.ssh \
    -w "$PWD" \
    flir-yocto-builder
~~~

(note the arguments; 
--user argument maps generated files owner to host owner. Not needed if 
the docker image was built on the same computer
first "-v" makes docker build and host built result compatible 
second -v is optional (if you like to fetch code inside container
)

When run like this, yocto build could be managed from the shell prompt
For a flir-cx opensource yocto environment, typically as;
~~~console
MACHINE=ec201 source ./flir-setup-release.sh -b build_ec201
bitbake flir-image-sherlock
~~~
(bitbake could be done for any present recipes)

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
$ podman run --rm -it --user $(id -u):$(id -g) \
    --userns=keep-id \
    -v $PWD:/"$PWD" \
    -v ~/.ssh:/home/yoctobuild/.ssh \
    -w "$PWD" \
    localhost/flir-yocto-builder
~~~

