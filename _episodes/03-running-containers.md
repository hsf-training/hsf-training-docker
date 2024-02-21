---
title: "Running Containers"
teaching: 15
exercises: 5
questions:
- "How are containers run?"
- "How do you monitor containers?"
- "How are containers exited?"
- "How are containers restarted?"
objectives:
- "Run containers"
- "Understand container state"
- "Stop and restart containers"
keypoints:
- "Run containers with `podman run <image-id>`"
- "Monitor containers with `podman ps`"
- "Exit interactive sessions using the `exit` command"
- "Restart stopped containers with `podman start`"
---
<iframe width="427" height="251" src="https://www.youtube.com/embed/UejGBfppmZY?list=PLKZ9c4ONm-VnqD5oN2_8tXO0Yb1H_s0sj" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

To use an image as a particular instance on a host machine, you [run][podman-docs-run]
it as a container.
You can run in either a detached or foreground (interactive) mode.

Run the image we pulled as a container with an interactive bash terminal:

~~~bash
podman run -it matthewfeickert/intro-to-docker:latest /bin/bash
~~~
{: .source}

The `-i` option here enables the interactive session, the `-t` option gives access to a terminal and the `/bin/bash` command makes the container start up in a bash session.

You are now inside the container in an interactive bash session. Check the file directory

~~~bash
pwd
~~~
{: .source}

~~~
/home/docker/data
~~~
{: .output}

and check the host to see that you are not in your local host system

~~~bash
hostname
~~~
{: .source}

~~~
<generated hostname>
~~~
{: .output}

Further, check the `os-release` to see that you are actually inside a release of Debian
(given the [Docker Library's Python image][docker-hub-python] Dockerfile choices)

~~~bash
cat /etc/os-release
~~~
{: .source}

~~~
PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
~~~
{: .output}

> ## Working directory
>
>You may be wondering why you are at `/home/docker/data` inside the container.
>This is the working directory that was set for the image.
>
> In the next chapters we will see how to build your own images
> and set parameters such as the working directory.
>{: .source}
{: .callout}

## Monitoring Containers

Open up a new terminal tab on the host machine and
[list the containers that are currently running][podman-docs-ps]:

~~~bash
podman ps
~~~
{: .source}

~~~
CONTAINER ID        IMAGE         COMMAND             CREATED             STATUS              PORTS               NAMES
<generated id>      <image:tag>   "/bin/bash"         n minutes ago       Up n minutes                            <generated name>
~~~
{: .output}

Notice that the name of your container is some randomly generated name.
To make the name more helpful, [rename][podman-docs-rename] the running container

~~~bash
podman rename <CONTAINER ID> my-example
~~~
{: .source}

and then verify it has been renamed

~~~bash
podman ps
~~~
{: .source}

~~~
CONTAINER ID        IMAGE         COMMAND             CREATED             STATUS              PORTS               NAMES
<generated id>      <image:tag>   "/bin/bash"         n minutes ago       Up n minutes                            my-example
~~~
{: .output}

> ## Renaming by name
>
>You can also identify containers to rename by their current name
>
>~~~bash
>podman rename <NAME> my-example
>~~~
>{: .source}
{: .callout}

Alternatively, you can also give the container a name at creation, using the `--name ` option:

~~~bash
podman run -it --name my-fancy-name matthewfeickert/intro-to-docker:latest /bin/bash
~~~
{: .source}

This way, it has a custom chosen name to start with, which you can use later on to interact with it.


# Exiting and restarting containers

As a test, go back into the terminal used for your container, and create a file in the container

~~~bash
touch test.txt
~~~
{: .source}

In the container exit at the command line

~~~bash
exit
~~~
{: .source}

You are returned to your shell.
If you list the containers you will notice that none are running

~~~bash
podman ps
~~~
{: .source}

~~~
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
~~~
{: .output}

but you can see all containers that have been run and not removed with

~~~bash
podman ps -a
~~~
{: .source}

~~~
CONTAINER ID        IMAGE         COMMAND             CREATED            STATUS                     PORTS               NAMES
<generated id>      <image:tag>   "/bin/bash"         n minutes ago      Exited (0) t seconds ago                       my-example
~~~
{: .output}

To restart your exited container [start][podman-docs-start] it again and then
[attach][podman-docs-attach] it interactively to your shell

~~~bash
podman start <CONTAINER ID>
podman attach <CONTAINER ID>
~~~
{: .source}

> ## `exec` command
> The [attach][podman-docs-attach] command used here is a handy shortcut to interactively access a running container with the same start command (in this case `/bin/bash`) that it was originally run with.
>
> In case you'd like some more flexibility, the [exec][podman-docs-exec] command lets you run any command in the container, with options similar to the run command to enable an interactive (`-i`) session, etc.
>
> For example, the `exec` equivalent to `attach`ing in our case would look like:
> ~~~bash
> podman start <CONTAINER ID>
> podman exec -it <CONTAINER ID> /bin/bash
> ~~~
{: .callout}

> ## Starting and attaching by name
>
>You can also start and attach containers by their name
>
>~~~bash
>podman start <NAME>
>podman attach <NAME>
>~~~
>{: .source}
{: .callout}


Notice that your entry point is still `/home/docker/data` and then check that your
`test.txt` still exists

~~~bash
ls
~~~
{: .source}

~~~
test.txt
~~~
{: .output}

So this shows us that we can exit containers for arbitrary lengths of time and then
return to our working environment inside of them as desired.

>## Clean up a container
>
>If you want a container to be [cleaned up][podman-docs-run-clean-up] &mdash; that is,
>deleted &mdash; after you exit it then run with the `--rm` option flag
>
>~~~bash
>podman run --rm -it <IMAGE> /bin/bash
>~~~
>{: .source}
{: .callout}

[podman-docs-run]: https://docs.podman.io/en/stable/markdown/podman-run.1.html
[docker-hub-python]: https://github.com/docker-library/python
[podman-docs-ps]: https://docs.podman.io/en/stable/markdown/podman-ps.1.html
[podman-docs-rename]: https://docs.docker.com/engine/reference/commandline/rename/
[podman-docs-start]: https://docs.docker.com/engine/reference/commandline/start/
[podman-docs-attach]: https://docs.docker.com/engine/reference/commandline/attach/
[podman-docs-exec]: https://docs.docker.com/engine/reference/commandline/exec/
[podman-docs-run-clean-up]: https://docs.podman.io/en/stable/markdown/podman-run.1.html#rm

{% include links.md %}
