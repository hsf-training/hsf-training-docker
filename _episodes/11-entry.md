---
title: "Using CMD and ENTRYPOINT in Dockerfiles"
teaching: 15
exercises: 10
questions:
- "How are default commands set in Dockerfiles?"
objectives:
- "Learn how and when to use `CMD`"
- "Learn how and when to use `ENTRYPOINT`"
keypoints:
- "`CMD` provide defaults for an executing container"
- "`CMD` can provide options for `ENTRYPOINT`"
- "`ENTRYPOINT` allows you to configure commands that will always run for an executing container"
---

So far every time we've run the containers we've typed

~~~bash
podman run --rm -it <IMAGE>:<TAG> <command>
~~~
{: .source}

like

~~~bash
podman run --rm -it python:3.9-slim /bin/bash
~~~
{: .source}

Running this dumps us into a Bash session

~~~bash
echo $SHELL
~~~
{: .source}

~~~bash
SHELL=/bin/bash
~~~
{: .output}

However, if no `/bin/bash` is given then you are placed inside the Python 3.7 REPL.

~~~bash
podman run --rm -it python:3.9-slim
~~~
{: .source}

~~~
Python 3.9.18 (main, Feb 13 2024, 10:56:47)
[GCC 12.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
~~~
{: .output}

These are very different behaviors, so let's understand what is happening.

The Python 3.9 image has a default command that runs when the container is executed,
which is specified in the Dockerfile with [`CMD`][docker-docs-CMD].

Create a file named `Dockerfile.defaults`

~~~bash
touch Dockerfile.defaults
~~~
{: .source}

~~~yaml
# Dockerfile.defaults
# Make the base image configurable
ARG BASE_IMAGE=python:3.9-slim
FROM ${BASE_IMAGE}
USER root
RUN apt-get -qq -y update && \
    apt-get -qq -y upgrade && \
    apt-get -y autoclean && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt-get/lists/*
# Create user "docker"
RUN useradd -m docker && \
    cp /root/.bashrc /home/docker/ && \
    mkdir /home/docker/data && \
    chown -R --from=root docker /home/docker
ENV HOME /home/docker
WORKDIR ${HOME}/data
USER docker

CMD ["/bin/bash"]
~~~
{: .source}

Now build the dockerfile, specifying its name with the `-f` argument since the engine will otherwise look for a file named `Dockerfile` by default.

~~~
podman build -f Dockerfile.defaults -t defaults-example:latest .
~~~
{: .source}

Now running

~~~
podman run --rm -it defaults-example:latest
~~~
{: .source}

again drops you into a Bash shell as specified by `CMD`.
As has already been seen, `CMD` can be overridden by giving a command after the image

~~~
podman run --rm -it defaults-example:latest python3
~~~
{: .source}

The [`ENTRYPOINT`][docker-docs-ENTRYPOINT] builder command allows to define a command or
commands that are **always** run at the "entry" to the container.
If an `ENTRYPOINT` has been defined then `CMD` provides optional inputs to the `ENTRYPOINT`.

Create a file named `entrypoint.sh`
~~~bash
# entrypoint.sh
#!/usr/bin/env bash

set -e

function main() {
    if [[ $# -eq 0 ]]; then
        printf "\nHello, World!\n"
    else
        printf "\nHello %s\n" "${1}"
    fi
}

main "$@"

/bin/bash
~~~
{: .bash}

And now modify the `Dockerfile.defaults` to use the `entrypoint.sh` script
~~~
# Dockerfile.defaults
# Make the base image configurable
ARG BASE_IMAGE=python:3.9-slim
FROM ${BASE_IMAGE}
USER root
RUN apt-get -qq -y update && \
    apt-get -qq -y upgrade && \
    apt-get -y autoclean && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt-get/lists/*
# Create user "docker"
RUN useradd -m docker && \
    cp /root/.bashrc /home/docker/ && \
    mkdir /home/docker/data && \
    chown -R --from=root docker /home/docker
ENV HOME /home/docker
WORKDIR ${HOME}/data
USER docker

COPY entrypoint.sh $HOME/entrypoint.sh
ENTRYPOINT ["/bin/bash", "/home/docker/entrypoint.sh"]
CMD ["there"]
~~~
{: .source}
Note how `CMD` provides an optional input to `entrypoint.sh`.

~~~
podman build -f Dockerfile.defaults -t defaults-example:latest --compress .
~~~
{: .source}

So now try
~~~
podman run --rm -it defaults-example:latest
~~~
{: .source}

> ## Applied `ENTRYPOINT` and `CMD`
>
> What will be the output of
>~~~
>podman run --rm -it defaults-example:latest $USER
>~~~
>{: .source}
> and why?
>
> > ## Solution
> >
> >~~~
> >
> >Hello <your user name>
> >docker@2a99ffabb512:~/data$
> >~~~
> >{: .output}
> `$USER` is evaluated and then overrides the default `CMD` to be passed to `entrypoint.sh`
> {: .solution}
{: .challenge}

[docker-docs-CMD]: https://docs.docker.com/engine/reference/builder/#cmd
[docker-docs-ENTRYPOINT]: https://docs.docker.com/engine/reference/builder/#entrypoint

{% include links.md %}
