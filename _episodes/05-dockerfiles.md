---
title: "Writing Dockerfiles and Building Images"
teaching: 20
exercises: 10
questions:
- "How are Dockerfiles written?"
- "How are Docker images built?"
objectives:
- "Write simple Dockerfiles"
- "Build a Docker image from a Dockerfile"
keypoints:
- "Dockerfiles are written as text file commands to the Docker engine"
- "Docker images are built with `podman build`"
- "Docker images can have multiple tags associated to them"
- "Docker images can use `COPY` to copy files into them during build"
---
<iframe width="427" height="251" src="https://www.youtube.com/embed/8BhkS2rZQ6E?list=PLKZ9c4ONm-VnqD5oN2_8tXO0Yb1H_s0sj" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Container images are static files that contain a template to create containers on machines.
Container engines like Podman or Docker pull the images from repositories or local storage and then create containers from them.
Container engines can also build and save to a repository new container images, interactively or following a set of instructions, starting from scratch or modifying an existing image.

A common way of defining the instructions to build a container image is through a [Dockerfile][docker-docs-builder].
These text based documents provide the instructions through an API similar to the Linux
operating system commands to execute commands during the build.
The [`Dockerfile` for the example image][example-Dockerfile] being used is an example of
some simple extensions of the [official Python 3.9 Docker image][python-docker-image] based on Debian Bullseye (`python:3.9-bullseye`).

Like Docker, Podman also uses `Dockerfile`s to build images, so the same instructions can be used for both tools.
We will continue with Podman throughout this lesson but the same commands can be used with Docker.

As a very simple example of extending the example image into a new image create a `Dockerfile`
on your local machine

~~~bash
touch Dockerfile
~~~
{: .source}

and then write in it the Docker engine instructions to add [`cowsay`][cowsay] and
[`scikit-learn`][scikit-learn] to the environment

~~~yaml
# Dockerfile

# Specify the base image that we're building the image on top of
FROM matthewfeickert/intro-to-docker:latest

# Build the image as root user
USER root

# Run some bash commands to install packages
RUN apt-get -y update && \
    apt-get -y upgrade && \
    apt-get -y install cowsay && \
    apt-get -y autoclean && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt-get/lists/* && \
    ln -s /usr/games/cowsay /usr/bin/cowsay
RUN pip install --no-cache-dir -q scikit-learn

# This sets the default working directory when a container is launched from the image
WORKDIR /home/docker

# Run as docker user by default when the container starts up
USER docker
~~~
{: .source}

> ## Dockerfile layers (or: why all these '&&'s??)
>
>Each `RUN` command in a Dockerfile creates a new layer to the image.
>In general, each layer should try to do one job and the fewer layers in an image
> the easier it is compress.
>
> This is why you see all these '&& \'s in the `RUN` command, so that all the shell commands will run in a pipeline and will take place in a single layer
> When trying to upload and download images on demand the smaller the size the better.
>
> Another thing to keep in mind is that each `RUN` command occurs in its own shell, so any environment variables, etc. set in one `RUN` command will not persist to the next.
{: .callout}

> ## Garbage cleanup
> Notice that the last few lines of the `RUN` command clean up and remove unneeded files that get produced during the installation process. This is important for keeping image sizes small, since files produced during each image-building layer will persist into the final image and add unnecessary bulk.
{: .callout}

> ## Don't run as `root`
>
>By default Docker containers will run as `root`. This is a bad idea and a security concern.
>Instead, setup a default user (like `docker` in the example) and if needed give the user
>greater privileges.
{: .callout}

Then [`build`][podman-docs-build] an image from the `Dockerfile` with Podman and tag it with a
human-readable name

~~~bash
podman build -f Dockerfile -t extend-example:latest .
~~~
{: .source}

You can now run the image as a container and verify for yourself that your additions exist

~~~bash
podman run --rm -it extend-example:latest /bin/bash
which cowsay
cowsay "Hello from inside the container"
pip list | grep scikit
python3 -c "import sklearn as sk; print(sk)"
~~~
{: .source}

~~~
/usr/bin/cowsay
 ___________________
< Hello from inside the container >
 -------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

scikit-learn        1.3.1
<module 'sklearn' from '/usr/local/lib/python3.9/site-packages/sklearn/__init__.py'>
~~~
{: .output}

You can list all images available on your local machine with `podman images`:
~~~bash
podman images
~~~
{: .source}

~~~
REPOSITORY                                 TAG            IMAGE ID      CREATED       SIZE
localhost/extend-example                   latest         c24a757fabe7  8 hours ago   2.2 GB
docker.io/matthewfeickert/intro-to-docker  latest         64708e04f3a9  2 years ago   1.62 GB
...
~~~
{: .output}

`docker.io` indicates that the image was pulled from the Docker Hub,
while `localhost` indicates that the image was built locally.

## Tags

In the examples so far the built image has been tagged with a single tag (e.g. `latest`).
However, tags are simply arbitrary labels meant to help identify images and images can
have multiple tags.
New tags can be specified in the `podman build` (or `docker build`) command by giving the `-t` flag multiple
times or they can be specified after an image is built by using
[`podman tag`][podman-docs-tag].

~~~bash
podman tag <SOURCE_IMAGE[:TAG]> <TARGET_IMAGE[:TAG]>
~~~
{: .source}

> ## Add your own tag
>
> Using `podman tag` add a new tag to the image you built.
>
> > ## Solution
> >
> > ~~~
> >podman images extend-example
> >podman tag extend-example:latest extend-example:my-tag
> >podman images extend-example
> > ~~~
> > {: .source}
> >
> > ~~~
> >REPOSITORY                TAG         IMAGE ID      CREATED      SIZE
> >localhost/extend-example  latest      c24a757fabe7  9 hours ago  2.2 GB
> >
> >REPOSITORY                TAG         IMAGE ID      CREATED      SIZE
> >localhost/extend-example  my-tag      c24a757fabe7  9 hours ago  2.2 GB
> >localhost/extend-example  latest      c24a757fabe7  9 hours ago  2.2 GB
>>
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

> ## Tags are labels
>
>Note how the image ID didn't change for the two tags: they are the same object.
>Tags are simply convenient human-readable labels.
{: .callout}

## `COPY`

Podman also gives you the ability to copy external files into a container image during the
build with the [`COPY`][docker-docs-COPY] Dockerfile command.
Which allows copying a target file from a host file system into the image
file system

~~~yaml
COPY <path on host> <path in container image>
~~~
{: .source}

For example, if there is a file called `install_python_deps.sh` in the same directory as
the build is executed from

~~~bash
touch install_python_deps.sh
~~~
{: .source}

with contents

~~~bash
cat install_python_deps.sh
~~~
{: .source}

~~~
#!/usr/bin/env bash

set -e

pip install --upgrade --no-cache-dir pip setuptools wheel
pip install --no-cache-dir -q scikit-learn
~~~
{: .output}

then this could be copied into the container image of the previous example during the build
and then used (and then removed as it is no longer needed).

Create a new file called `Dockerfile.copy`:

~~~bash
touch Dockerfile.copy
~~~
{: .source}

and fill it with a modified version of the above Dockerfile, where we now copy `install_python_deps.sh` from the local working directory into the container and use it to install the specified python dependencies:

~~~yaml
# Dockerfile.copy
FROM matthewfeickert/intro-to-docker:latest
USER root
RUN apt-get -qq -y update && \
    apt-get -qq -y upgrade && \
    apt-get -qq -y install cowsay && \
    apt-get -y autoclean && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt-get/lists/* && \
    ln -s /usr/games/cowsay /usr/bin/cowsay
COPY install_python_deps.sh install_python_deps.sh
RUN bash install_python_deps.sh && \
    rm install_python_deps.sh
WORKDIR /home/data
USER docker
~~~
{: .source}

~~~bash
podman build -f Dockerfile.copy -t copy-example:latest .
~~~
{: .source}

For very complex scripts or files that are on some remote, `COPY` offers a straightforward
way to bring them into the container image build.


[docker-docs-builder]: https://docs.docker.com/engine/reference/builder/
[example-Dockerfile]: https://github.com/matthewfeickert/intro-to-docker/blob/feat/update-2021-bootcamp/docker/Dockerfile
[python-docker-image]: https://hub.docker.com/_/python
[cowsay]: https://packages.debian.org/bullseye/cowsay
[scikit-learn]: https://scikit-learn.org
[podman-docs-build]: https://docs.podman.io/en/stable/markdown/podman-build.1.html
[podman-docs-tag]: https://docs.podman.io/en/latest/markdown/podman-tag.1.html
[docker-docs-COPY]: https://docs.docker.com/engine/reference/builder/#copy

{% include links.md %}
