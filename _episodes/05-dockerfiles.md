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
- "Docker images are built with `docker build`"
- "Built time variables can be defined with `ARG` and set with `--build-arg`"
- "Docker images can have multiple tags associated to them"
- "Docker images can use `COPY` to copy files into them during build"
---

Docker images are built through the Docker engine by reading the instructions from a
[`Dockerfile`][docker-docs-builder].
These text based documents provide the instructions though an API similar to the Linux
operating system commands to execute commands during the build.
The [`Dockerfile` for the example image][example-Dockerfile] being used is an example of
some simple extensions of the [official Python 3.6.8 Docker image][python-docker-image].

As a very simple of extending the example image into a new image create a `Dockerfile`
on your local machine

~~~
touch Dockerfile
~~~
{: .source}

and then write in it the Docker engine instructions to add [`cowsay`][cowsay] and
[`scikit-learn`][scikit-learn] to the environment

~~~
# Dockerfile
FROM matthewfeickert/intro-to-docker:latest
USER root
RUN apt-get -qq -y update && \
    apt-get -qq -y upgrade && \
    apt-get -qq -y install cowsay && \
    apt-get -y autoclean && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt-get/lists/* && \
    ln -s /usr/games/cowsay /usr/bin/cowsay
RUN pip install --no-cache-dir -q scikit-learn
USER docker
~~~
{: .source}

> ## Dockerfile layers
>
>Each `RUN` command in a Dockerfile creates a new layer to the Docker image.
>In general, each layer should try to do one job and the fewer layers in an image
> the easier it is compress. When trying to upload and download images on demand the
> smaller the size the better.
{: .callout}

> ## Don't run as `root`
>
>By default Docker containers will run as `root`. This is a bad idea and a security concern.
>Instead, setup a default user (like `docker` in the example) and if needed give the user
>greater privileges.
{: .callout}

Then [`build`][docker-docs-build] an image from the `Dockerfile` and tag it with a human
readable name

~~~
docker build -f Dockerfile -t extend-example:latest .
~~~
{: .source}

You can now run the image as a container and verify for yourself that your additions exist

~~~
docker run --rm -it extend-example:latest /bin/bash
which cowsay
cowsay "Hello from Docker"
pip list | grep scikit
python3 -c "import sklearn as sk; print(sk)"
~~~
{: .source}

~~~
/usr/bin/cowsay
 ___________________
< Hello from Docker >
 -------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

scikit-learn       0.21.3
<module 'sklearn' from '/usr/local/lib/python3.6/site-packages/sklearn/__init__.py'>
~~~
{: .output}

## Tags

In the examples so far the built image has been tagged with a single tag (e.g. `latest`).
However, tags are simply arbitrary labels meant to help identify images and images can
have multiple tags.
New tags can be specified in the `docker build` command by giving the `-t` flag multiple
times or they can be specified after an image is built by using
[`docker tag`][docker-docs-tag].

~~~
docker tag <SOURCE_IMAGE[:TAG]> <TARGET_IMAGE[:TAG]>
~~~
{: .source}

> ## Add your own tag
>
> Using `docker tag` add a new tag to the image you built.
>
> > ## Solution
> >
> > ~~~
> >docker images extend-example
> >docker tag extend-example:latest extend-example:my-tag
> >docker images extend-example
> > ~~~
> > {: .source}
> >
> > ~~~
> >REPOSITORY          TAG                 IMAGE ID            CREATED            SIZE
> >extend-example      latest              b571a34f63b9        t seconds ago      1.59GB
> >
> >REPOSITORY          TAG                 IMAGE ID            CREATED            SIZE
> >extend-example      latest              b571a34f63b9        t seconds ago      1.59GB
> >extend-example      my-tag              b571a34f63b9        t seconds ago      1.59GB
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

> ## Tags are labels
>
>Note how the image ID didn't change for the two tags: they are the same object.
>Tags are simply convenient human readable labels.
{: .callout}

## `COPY`

Docker also gives you the ability to copy external files into a Docker image during the
build with the [`COPY`][docker-docs-COPY] Dockerfile command.
Which allows copying a target file on the from a host file system into the Docker image
file system

~~~
COPY <path on host> <path in Docker image>
~~~
{: .source}

For example, if there is a file called `install_python_deps.sh` in the same directory as
the build is executed from

~~~
touch install_python_deps.sh
~~~
{: .source}

with contents

~~~
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

then this could be copied into the Docker image of the previous example during the build
and then used (and then removed as it is no longer needed) with the following

~~~
# Dockerfile.copy
FROM python:3.7
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
# Create user "docker"
RUN useradd -m docker && \
    cp /root/.bashrc /home/docker/ && \
    mkdir /home/docker/data && \
    chown -R --from=root docker /home/docker
WORKDIR /home/data
USER docker
~~~
{: .source}

~~~
docker build -f Dockerfile.copy -t copy-example:latest .
~~~
{: .source}

For very complex scripts or files that are on some remote, `COPY` offers a straightforward
way to bring them into the Docker build.


[docker-docs-builder]: https://docs.docker.com/engine/reference/builder/
[example-Dockerfile]: https://github.com/matthewfeickert/Intro-to-Docker/blob/master/Dockerfile
[python-docker-image]: https://hub.docker.com/_/python
[cowsay]: https://packages.debian.org/jessie/cowsay
[scikit-learn]: https://scikit-learn.org
[docker-docs-build]: https://docs.docker.com/engine/reference/commandline/build/
[docker-docs-ARG]: https://docs.docker.com/engine/reference/builder/#arg
[docker-docs-FROM]: https://docs.docker.com/engine/reference/builder/#from
[docker-docs-build-arg]: https://docs.docker.com/engine/reference/commandline/build/#set-build-time-variables---build-arg
[docker-docs-ENV]: https://docs.docker.com/engine/reference/builder/#env
[docker-docs-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-docs-COPY]: https://docs.docker.com/engine/reference/builder/#copy

{% include links.md %}
