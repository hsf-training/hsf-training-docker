---
title: "Writing Dockerfiles and Building Images"
teaching: 30
exercises: 20
questions:
- "How are Dockerfiles written?"
- "How are images built?"
objectives:
- "Write simple Dockerfiles"
- "Build a container image from a Dockerfile"
keypoints:
- "Dockerfiles are written as text file commands to the container engine"
- "Images are built with `podman build`"
- "Images can have multiple tags associated to them"
- "Images can use `COPY` to copy files into them during build"
- "Images can use `ADD` to copy remote files and extract compressed files"
- "Images can use multi-stage builds to reduce their final size"
---
<iframe width="427" height="251" src="https://www.youtube.com/embed/NSVXBgYSkBY?si=pAZsMxfkZ2imcL52" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Container images are static files that contain a template to create containers on machines.
Container engines like Podman or Docker pull the images from repositories or local storage and then create containers from them.
Container engines can also build and save to a repository new container images, interactively or following a set of instructions, starting from scratch or modifying an existing image.

A common way of defining the instructions to build a container image is through a [Dockerfile][docker-docs-builder].
These text-based documents provide the instructions through an API similar to the Linux
operating system commands to execute commands during the build.

Like Docker, Podman also uses `Dockerfile`s to build images, so the same instructions can be used for both tools.
We will continue with Podman throughout this lesson, but the same commands can be used with Docker.

As a very simple example of extending the example image, into a new image create a `Dockerfile`
on your local machine

~~~bash
touch Dockerfile
~~~
{: .source}

and then write in it the Docker engine instructions to add [`cowsay`][cowsay] and
[`scikit-learn`][scikit-learn] to the environment

~~~dockerfile
# Dockerfile

# Specify the base image that we're building the image on top of
FROM almalinux:9

# Build the image as root user
USER root

# Run some bash commands to install packages
RUN dnf -y update && \
    dnf -y upgrade && \
    dnf -y install epel-release && \
    dnf -y install pip && \
    dnf -y install cowsay && \
    dnf clean all && \
    rm -rf /var/cache/dnf

RUN pip install --no-cache-dir -q scikit-learn

# Create a new user
RUN useradd -ms /bin/bash docker

# This sets the default working directory when a container is launched from the image
WORKDIR /home/docker

# Run as docker user by default when the container starts up
USER docker
~~~
{: .source}

> ## Dockerfile layers (or: why all these '&&'s??)
>
>Each `RUN` command in a Dockerfile creates a new layer to the image.
>In general, each layer should try to do one job, and the fewer layers in an image,
> the easier it is to compress.
>
> This is why you see all these '&& \'s in the `RUN` command, so that all the shell commands will run in a pipeline and will take place in a single layer
> When trying to upload and download images on demand, the smaller the size, the better.
>
> Another thing to keep in mind is that each `RUN` command occurs in its own shell, so any environment variables, etc., set in one `RUN` command will not persist to the next.
{: .callout}

> ## Garbage cleanup
> Notice that the last few lines of the `RUN` command clean up and remove unneeded files that get produced during the installation process. This is important for keeping image sizes small, since files produced during each image-building layer will persist into the final image and add unnecessary bulk.
{: .callout}

> ## Don't run as `root`
>
>By default, Docker containers will run as `root`. This is a bad idea and a security concern.
>Instead, set up a default user (like `docker` in the example) and, if needed, give the user
>greater privileges.
{: .callout}

Then, [`build`][podman-docs-build] an image from the `Dockerfile` with Podman and tag it with a
human-readable name

~~~bash
podman build -f Dockerfile -t extend-example:latest .
~~~
{: .source}

You can now run the image as a container and verify for yourself that your additions exist

~~~bash
podman run --rm -it extend-example:latest /bin/bash
cowsay "Hello from inside the container"
pip list | grep scikit
python3 -c "import sklearn as sk; print(sk)"
~~~
{: .source}

~~~
 _________________________________
< Hello from inside the container >
 ---------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

scikit-learn    1.6.1
<module 'sklearn' from '/usr/local/lib64/python3.9/site-packages/sklearn/__init__.py'>
~~~
{: .output}

You can list all images available on your local machine with `podman images`:
~~~bash
podman images
~~~
{: .source}

~~~
REPOSITORY                   TAG         IMAGE ID      CREATED        SIZE
localhost/extend-example     latest      c8b76717b954  2 minutes ago  550 MB
docker.io/library/almalinux  9           b894a52b4112  5 weeks ago    196 MB
...
~~~
{: .output}

`docker.io` indicates that the image was pulled from the Docker Hub,
while `localhost` indicates that the image was built locally.

## Tags

In the examples so far, the built image has been tagged with a single tag (e.g., `latest`).
However, tags are simply arbitrary labels meant to help identify images, and images can
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
> REPOSITORY                TAG         IMAGE ID      CREATED        SIZE
> localhost/extend-example  latest      c8b76717b954  5 minutes ago  550 MB
> >
> REPOSITORY                TAG         IMAGE ID      CREATED        SIZE
> localhost/extend-example  my-tag      c8b76717b954  5 minutes ago  550 MB
> localhost/extend-example  latest      c8b76717b954  5 minutes ago  550 MB
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
This allows copying a target file from a host file system into the image
file system

~~~dockerfile
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

Then, this could be copied into the container image of the previous example during the build
and then used (and then removed as it is no longer needed).

Create a new file called `Dockerfile.copy`:

~~~bash
touch Dockerfile.copy
~~~
{: .source}

and fill it with a modified version of the above Dockerfile, where we now copy `install_python_deps.sh` from the local working directory into the container and use it to install the specified Python dependencies:

~~~dockerfile
# Dockerfile.copy

# Specify the base image that we're building the image on top of
FROM almalinux:9

# Build the image as root user
USER root

# Run some bash commands to install packages
RUN dnf -y update && \
    dnf -y upgrade && \
    dnf -y install epel-release && \
    dnf -y install pip && \
    dnf -y install cowsay && \
    dnf clean all && \
    rm -rf /var/cache/dnf

COPY install_python_deps.sh install_python_deps.sh
RUN bash install_python_deps.sh && \
    rm install_python_deps.sh

# Create a new user
RUN useradd -ms /bin/bash docker

# This sets the default working directory when a container is launched from the image
WORKDIR /home/docker

# Run as docker user by default when the container starts up
USER docker
~~~
{: .source}

~~~bash
podman build -f Dockerfile.copy -t copy-example:latest .
~~~
{: .source}

For very complex scripts or files that are on some remote, `COPY` offers a straightforward
way to bring them into the container image build.


## `ADD`

The `ADD` command is very similar to the `COPY` command, except that the `ADD` command supports two additional features:
1. Automatic decompression of compressed files.
2. Automatic fetching of remote URLs (starting with `http://` or `https://`) and cloning of git repositories (starting with `git@`).

When these features are not required, [`COPY` is preferred][add-or-copy].

Note that
- local compressed files are unpacked by default
- remote compressed files are not unpacked by default

This behaviour can be changed by adding a `--unpack=true` or `--unpack=false` flag immediately after the `ADD` command:
~~~dockerfile
ADD --unpack=true <src> <dest>
~~~
{: .source}

As an example, let's compile a simple [`main.c`][c-file] file from a remote url:
~~~dockerfile
# Dockerfile.add
FROM almalinux:9
ADD https://raw.githubusercontent.com/oer-particle-physics/hsf-training-docker/refs/heads/gh-pages/examples/main.c .
RUN dnf -y update && \
    dnf -y upgrade && \
    dnf -y install clang && \
    dnf clean all && \
    rm -rf /var/cache/dnf
RUN clang main.c -o main
~~~
{: .source}

~~~bash
podman build -f Dockerfile.add -t add-example
~~~
{: .source}

Then, you can run the compiled executable with
~~~bash
podman run --rm add-example ./main
~~~
{: .source}

~~~
hello world
~~~
{: .output}

## Multi-Stage Builds
The tools you use to build your image are often not necessary for a user of the image.

To dramatically reduce the final image size, we can separate the build process into multiple stages by using multiple `FROM` statements.
Each `FROM` statement specifies an independent image up until the next `FROM` statement.
By default, nothing is copied between images, and only the image specified by the final `FROM` statement is saved with the tag that you provide.

Files can be copied between stages using the [`COPY --from=<stage>`][copy-from] syntax.

Let's improve on the `Dockerfile.add` example by only copying over the compiled executable:

~~~dockerfile
# Dockerfile.multistage
FROM almalinux:9 AS build
ADD https://raw.githubusercontent.com/oer-particle-physics/hsf-training-docker/refs/heads/gh-pages/examples/main.c .
RUN dnf -y update && \
    dnf -y upgrade && \
    dnf -y install clang && \
    dnf clean all && \
    rm -rf /var/cache/dnf
RUN clang main.c -o main

FROM almalinux:9
COPY --from=build main .
~~~
{: .source}

> ## Build compatibility
>
> Docker [recommends][from-alpine] using the simple and small Alpine Linux image when possible.
> However, programs compiled with one image may not run on another, so in this example, I'm using almalinux for both the build stage and the final stage.
{: .callout}


~~~bash
podman build -f Dockerfile.multistage -t multistage-example
~~~
{: .source}

Podman will cache the build stage for further use, so this multi-staged method has the added benefit that making changes to the second stage won't require rebuilding of the first stage.

The `FROM <image> AS <name>` syntax lets us reference the build stage by its `<name>` with the `COPY --from=<name>` command. Without this, we would have to reference the build stages in the order they appear (`COPY --from=<0,1,2,...>`).

Now, let's look at the sizes of the `Dockerfile.multistage` image versus the `Dockerfile.add` image:

~~~bash
podman images --filter reference=multistage* --filter reference=add*
~~~
{: .source}

~~~
REPOSITORY                    TAG         IMAGE ID      CREATED        SIZE
localhost/multistage-example  latest      ac9640ee042b  4 minutes ago  190 MB
localhost/add-example         latest      6d2f891efd09  4 minutes ago  777 MB
~~~
{: .output}

Our multi-stage build saves 570 MB and is 1/4 the size of the single-stage build, while still producing the same results for someone using the image:

~~~bash
podman run --rm multistage-example ./main
~~~
{: .source}

~~~
hello world
~~~
{: .output}



[docker-docs-builder]: https://docs.docker.com/engine/reference/builder/
[example-Dockerfile]: https://github.com/matthewfeickert/intro-to-docker/blob/feat/update-2021-bootcamp/docker/Dockerfile
[python-docker-image]: https://hub.docker.com/_/python
[cowsay]: https://packages.debian.org/bullseye/cowsay
[scikit-learn]: https://scikit-learn.org
[podman-docs-build]: https://docs.podman.io/en/stable/markdown/podman-build.1.html
[podman-docs-tag]: https://docs.podman.io/en/latest/markdown/podman-tag.1.html
[docker-docs-COPY]: https://docs.docker.com/engine/reference/builder/#copy
[add-or-copy]: https://docs.docker.com/build/building/best-practices/#add-or-copy
[c-file]: https://raw.githubusercontent.com/oer-particle-physics/hsf-training-docker/refs/heads/gh-pages/examples/main.c
[copy-from]: https://docs.docker.com/reference/dockerfile/#copy---from
[from-alpine]: https://docs.docker.com/build/building/best-practices/#from


{% include links.md %}
