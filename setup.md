---
title: Setup
---

## Installation

The training module can be followed using either Docker or Podman. We recommend using **Podman** as it
does not require root privileges to use it out of the box. In addition, Docker has licensing restrictions that
may prevent you from using it in certain sites.

> ## Installing Docker
>
> If you prefer to use Docker (check with the IT department of your institution before using Docker!), follow the official instructions for [Linux](https://docs.docker.com/engine/install/#server), [Mac](https://docs.docker.com/desktop/install/mac-install/), or [Windows](https://docs.docker.com/desktop/install/windows-install/).
>
> If you are using Linux, then please also follow these [post installation instructions](https://docs.docker.com/engine/install/linux-postinstall/).
>
> Across the tutorial, just replace `podman` by `docker` in the commands and you should be good to go.
>{: .source}
{: .callout}

The installation of Podman requires sudo privileges. If you don't have them, check if Podman is already installed on your system
with:
```bash
podman version
```
If it's not available, ask your system administrator to install it for you.

### Install Podman on Linux

Podman is available on the official repositories of most Linux distributions. Check the
[official documentation](https://podman.io/docs/installation#installing-on-linux)
to find out how to install it on your system.

For example, in Ubuntu, you can install it by running the following command:
```bash
# Ubuntu 20.10 and newer
sudo apt-get update
sudo apt-get -y install podman
```

### Install Podman on MacOS

Running Podman or Docker on MacOS requires a virtual machine to run the containers.
In the case of Podman, it provides an installer at [https://podman.io/](https://podman.io/). Download the `.dmg` package for MacOS, extract it, and execute Podman Desktop.

The first time that Podman Desktop is executed it will be required to install Podman and a Podman machine to execute the containers. Click "Set up" and follow the instructions.

### Install Podman on Windows

Podman provides instructions to install it on Windows at the [GitHub repository](https://github.com/containers/podman/blob/main/docs/tutorials/podman-for-windows.md).


## Configuration

We will now configure Podman or Docker to download (pull) container images from the correct source.

If you do not have a CERN account, we will set things up so that images are pulled from [Docker Hub](https://hub.docker.com/) (`docker.io`).

If you do have a CERN account, we will set things up to run from a [CERN-hosted repository](https://registry.cern.ch/harbor/projects/9/repositories) (`registry.cern.ch/docker.io`) instead.
This is preferred, as there is a usage-limit for `docker.io` that can come into effect if multiple people are using it from the same IP address.

To use this CERN registry, regardless of if you use Podman or Docker, you will need to log in with
~~~
podman login registry.cern.ch
# or
docker login registry.cern.ch
~~~
{: .source}

It will then prompt you for a username and password.
Use your CERN username.
The password is the CLI token found at https://registry.cern.ch/ in your account under "User Profile".

### Podman

> ## MacOS and Windows users
> 
> If you're not on Linux, you will need to edit these configuration files from within the podman virtual linux machine.
> You can do this by connectng the the VM with 
> ~~~bash
> podman machine ssh
> ~~~
> {: .source}
>
> and editing the configuration files there.
> 
> Once you are done, exit the VM
> ~~~bash
> exit
> ~~~
> {: .source}
{: .callout}

If you do not have a CERN account , add the following lines to `/etc/containers/registries.conf` to use `docker.io`.
~~~toml
# /etc/containers/registries.conf

unqualified-search-registries=["docker.io"]
~~~
{: .source}


If you have a CERN account, add these instead to use `registry.cern.ch/docker.io`
```toml
# /etc/containers/registries.conf

unqualified-search-registries=["docker.io"]

[[registry]]
prefix = "docker.io"
location = "registry.cern.ch/docker.io"
```
{: .source}

This will first set the default registry to `docker.io`, then it will map `docker.io` to `registry.cern.ch/docker.io`.
This means that images pulled from `registry.cern.ch/docker.io` appear as if they're coming from `docker.io`.

This mapping can have some unintended side effects, and so if it is causing any issues you can just clear the `/etc/containers/registries.conf` file and prepend `registry.cern.ch/docker.io/` to image names manually as we show to do with Docker below. This is, however, more tedious than using the automatic mapping.


### Docker
Docker uses `docker.io` by default.

If you are a CERN user and want/need to use the CERN registry, you can prepend `registry.cern.ch/docker.io/` to each image name 

So, for example, the command in the next section (with Docker) is 
~~~bash
docker run hello-world
~~~
{: .source}
this will, by default, be equivalent to
~~~bash
docker run docker.io/hello-world
~~~
{: .source}
To pull from the CERN regsitry, you can instead do
~~~bash
docker run registry.cern.ch/docker.io/hello-world
~~~
{: .source}
after logging in, and similarly prepend `registry.cern.ch/docker.io/` for all the commands in this tutorial.



## Post Installation

Check that you can run Podman with the following command:
```bash
podman run hello-world
```



### Optional: Fetch images in advance

Once you've got Podman up and running, do the following docker image pulls in advance to save time during the tutorial:

~~~bash
podman pull almalinux:9
podman pull debian:buster-slim
podman pull python:2.7-slim
podman pull python:3.7-slim
~~~

## Analysis Code

Later in this tutorial, you will be asked to work with a simple analysis that utilizes the CMS OpenData to search for Higgs to 2 tau leptons.
The full analysis itself can be found [here](https://github.com/hsf-training/hsf-training-cms-analysis) &mdash; and there is a dedicated set of [training lessons](https://hsf-training.github.io/hsf-training-cms-analysis-webpage/index.html) ([videos available](https://www.youtube.com/watch?v=gplMywJAFDI&list=PLKZ9c4ONm-Vk0wnDKaaovoEkOk3PVdL0V)).

It is best if you work through these lessons before the tutorial on Containers, but not mandatory.

* **<font color="red">First and foremost:</font>** To fork these repos, open the  [GitLab project creation page](https://gitlab.cern.ch/projects/new) and then select _Import project_ -> _Repository by URL_. Please make sure you've forked the starter repos into your own namespace before cloning and making commits to them, otherwise you'll run into permissions issues when you try to push your commits! Also, remember to set the visibility level to _Public_.
* Regarding authentication with ``kinit``:
  * If you are from CERN and use gitlab.cern.ch: Remember to add your CERN credentials as CI/CD variables to
    both repos for the `kinit` authentication in the `.gitlab-ci.yml` files to work.
    To do so, go to _Settings_ -> _CI/CD_ -> _Variables_ and create two new variables:
     * `CERN_USER` should contain your CERN username
     * `SERVICE_PASS` should contain your password.
  * Else, you can remove the ``kinit`` line from `.gitlab-ci.yml` and use the public EOS datasets:
    * `root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced` for the skimming repo.
    * `root://eospublic.cern.ch//eos/opendata/cms/upload/apb2023/histograms.root` for the fitting repo.
* For the fitting code repo, the [fit_simple](https://github.com/hsf-training/hsf-training-cms-analysis-snapshot-stats/blob/master/.gitlab-ci.yml#L5) step in `.gitlab-ci.yml` expects to receive the file `histograms.root` produced by the skimming code. In case you haven't had a chance to produce this file yet, it can be downloaded from [here](https://eospublichttp.cern.ch//eos/opendata/cms/upload/apb2023/histograms.root). In any case, you can:
  * Use the public EOS datasets mentioned above.
  * If you are from CERN, you can copy the downloaded file to your personal EOS user space (`root://eosuser.cern.ch//eos/user/[first_letter_of_username]/[username]`).

{% include links.md %}
