# Running Containers

:::{admonition} Overview
:class: note
**Teaching:** 15 min | **Exercises:** 5 min

**Questions**
- How are containers run?
- How do you monitor containers?
- How are containers exited?
- How are containers restarted?

**Objectives**
- Run containers
- Understand container state
- Stop and restart containers
:::

<iframe width="427" height="251" src="https://www.youtube.com/embed/sebWDiHp9jA?si=R-U1GEVf-afqTIgw" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

To use an image as a particular instance on a host machine, you [run][podman-docs-run]
it as a container.
You can run in either a detached or foreground (interactive) mode.

Run the image we pulled as a container with an interactive bash terminal:

```bash
podman run -it almalinux:9 /bin/bash
```

The `-i` option here enables the interactive session, the `-t` option gives access to a terminal and the `/bin/bash` command makes the container start up in a bash session.

You are now inside the container in an interactive bash session. Try listing the files

```bash
ls
```

```text
afs  bin  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run	sbin  srv  sys	tmp  usr  var
```

and check the host to see that you are not in your local host system

```bash
hostname
```

```text
<generated hostname>
```

Further, check the `os-release` to see that you are actually inside a release of AlmaLinux

```bash
cat /etc/os-release
```

```text
NAME="AlmaLinux"
VERSION="9.7 (Moss Jungle Cat)"
ID="almalinux"
ID_LIKE="rhel centos fedora"
VERSION_ID="9.7"
PLATFORM_ID="platform:el9"
PRETTY_NAME="AlmaLinux 9.7 (Moss Jungle Cat)"
ANSI_COLOR="0;34"
LOGO="fedora-logo-icon"
CPE_NAME="cpe:/o:almalinux:almalinux:9::baseos"
HOME_URL="https://almalinux.org/"
DOCUMENTATION_URL="https://wiki.almalinux.org/"
BUG_REPORT_URL="https://bugs.almalinux.org/"

ALMALINUX_MANTISBT_PROJECT="AlmaLinux-9"
ALMALINUX_MANTISBT_PROJECT_VERSION="9.7"
REDHAT_SUPPORT_PRODUCT="AlmaLinux"
REDHAT_SUPPORT_PRODUCT_VERSION="9.7"
SUPPORT_END=2032-06-01
```

## Monitoring Containers

Open up a new terminal tab on the host machine and
[list the containers that are currently running][podman-docs-ps]:

```bash
podman ps
```

```text
CONTAINER ID        IMAGE         COMMAND             CREATED             STATUS              PORTS               NAMES
<generated id>      <image:tag>   "/bin/bash"         n minutes ago       Up n minutes                            <generated name>
```

Notice that the name of your container is some randomly generated name.
To make the name more helpful, [rename][podman-docs-rename] the running container

```bash
podman rename <CONTAINER ID> my-example
```

and then verify it has been renamed

```bash
podman ps
```

```text
CONTAINER ID        IMAGE         COMMAND             CREATED             STATUS              PORTS               NAMES
<generated id>      <image:tag>   "/bin/bash"         n minutes ago       Up n minutes                            my-example
```

:::{admonition} Renaming by name
:class: tip
You can also identify containers to rename by their current name

```bash
podman rename <NAME> my-example
```
:::

Alternatively, you can also give the container a name at creation, using the `--name ` option:

```bash
podman run -it --name my-fancy-name almalinux:9 /bin/bash
```

This way, it has a custom chosen name to start with, which you can use later on to interact with it.

## Exiting and restarting containers

As a test, go back into the terminal used for your container, and create a file in the container

```bash
touch test.txt
```

In the container exit at the command line

```bash
exit
```

You are returned to your shell.
If you list the containers you will notice that none are running

```bash
podman ps
```

```text
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

but you can see all containers that have been run and not removed with

```bash
podman ps -a
```

```text
CONTAINER ID        IMAGE         COMMAND             CREATED            STATUS                     PORTS               NAMES
<generated id>      <image:tag>   "/bin/bash"         n minutes ago      Exited (0) t seconds ago                       my-example
```

To restart your exited container [start][podman-docs-start] it again and then
[attach][podman-docs-attach] it interactively to your shell

```bash
podman start <CONTAINER ID>
podman attach <CONTAINER ID>
```

:::{admonition} `exec` command
:class: tip
The [attach][podman-docs-attach] command used here is a handy shortcut to interactively access a running container with the same start command (in this case `/bin/bash`) that it was originally run with.

In case you'd like some more flexibility, the [exec][podman-docs-exec] command lets you run any command in the container, with options similar to the run command to enable an interactive (`-i`) session, etc.

For example, the `exec` equivalent to `attach`ing in our case would look like:
```bash
podman start <CONTAINER ID>
podman exec -it <CONTAINER ID> /bin/bash
```
:::

:::{admonition} Starting and attaching by name
:class: tip
You can also start and attach containers by their name

```bash
podman start <NAME>
podman attach <NAME>
```
:::

Check that `test.txt` still exists

```bash
ls
```

```text
afs  bin  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run	sbin  srv  sys	test.txt  tmp  usr  var
```

So this shows us that we can exit containers for arbitrary lengths of time and then
return to our working environment inside of them as desired.

:::{admonition} Clean up a container
:class: tip
If you want a container to be [cleaned up][podman-docs-run-clean-up] &mdash; that is,
deleted &mdash; after you exit it then run with the `--rm` option flag

```bash
podman run --rm -it <IMAGE> /bin/bash
```
:::

[podman-docs-run]: https://docs.podman.io/en/stable/markdown/podman-run.1.html
[docker-hub-python]: https://github.com/docker-library/python
[podman-docs-ps]: https://docs.podman.io/en/stable/markdown/podman-ps.1.html
[podman-docs-rename]: https://docs.docker.com/engine/reference/commandline/rename/
[podman-docs-start]: https://docs.docker.com/engine/reference/commandline/start/
[podman-docs-attach]: https://docs.docker.com/engine/reference/commandline/attach/
[podman-docs-exec]: https://docs.docker.com/engine/reference/commandline/exec/
[podman-docs-run-clean-up]: https://docs.podman.io/en/stable/markdown/podman-run.1.html#rm

:::{admonition} Key Points
:class: note
- Run containers with `podman run <image-id>`
- Monitor containers with `podman ps`
- Exit interactive sessions using the `exit` command
- Restart stopped containers with `podman start`
:::
