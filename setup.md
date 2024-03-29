---
title: Setup
---

## Installation

The training module can be followed using either Docker or Podman. We recommend to use **Podman** as it
does not require root privileges to use it out of the box. In addition, Docker has licensing restrictions that
may prevent you from using it in certain sites.

The installation of Podman requires sudo privileges. If you don't have them, check if Podman is already installed on your system
with
```bash
podman version
```
If it's not available, ask you system administrator to install it for you.

### Install Podman on Linux

Podman is available on the official repositories of most Linux distributions. Check the
[official documentation](https://podman.io/docs/installation#installing-on-linux)
to find out how to install it on your system.

For example, in Ubuntu you can install it by running the following command:
```bash
# Ubuntu 20.10 and newer
sudo apt-get update
sudo apt-get -y install podman
```

### Install Podman on MacOS

Running Podman or Docker on MacOS requires a virtual machine to run the containers.
In the case of Podman, it provides an installer at [https://podman.io/](https://podman.io/). Download the `.dmg` package for MacOS, extract
it and execute Podman Desktop.

The first time that Podman Desktop is executed it will require to
install Podman and a Podman machine to execute the containers. Click "Set up" and follow the instructions.


### Install Podman on Windows

Podman provides instructions to install it on Windows at the [GitHub repository](https://github.com/containers/podman/blob/main/docs/tutorials/podman-for-windows.md).

## Post Installation

Check that you can run Podman with the following command
```bash
podman run hello-world
```

> ## Installing Docker
>
> If you prefer to use Docker (check with the IT department of your institution before using Docker!),
> follow the official instructions for [Linux](https://docs.docker.com/engine/install/#server), [Mac](https://docs.docker.com/desktop/install/mac-install/), or [Windows](https://docs.docker.com/desktop/install/windows-install/).
>
> If you are using Linux, then please also follow these [post installation instructions](https://docs.docker.com/engine/install/linux-postinstall/).
>
> Across the tutorial, just replace `podman` by `docker` in the commands and you should be good to go.
>{: .source}
{: .callout}


### Optional: Fetch images in advance

Once you've got Podman up and running, do the following docker image pulls in advance to save time during the tutorial:

~~~bash
podman pull matthewfeickert/intro-to-docker
podman pull debian:buster-slim
podman pull python:2.7-slim
podman pull python:3.7-slim
podman pull rootproject/root:6.22.06-conda
~~~

## Analysis Code

Later in this tutorial, you will be asked to work with a simple analysis that utilizes the CMS OpenData to search for Higgs to 2 tau leptons.
The full analysis itself can be found [here](https://github.com/hsf-training/hsf-training-cms-analysis) - and there is a dedicated set of [training lessons](https://hsf-training.github.io/hsf-training-cms-analysis-webpage/index.html) ([videos available](https://www.youtube.com/watch?v=gplMywJAFDI&list=PLKZ9c4ONm-Vk0wnDKaaovoEkOk3PVdL0V)).

It is best if you work through these lessons before the tutorial on Containers, but not mandatory.

* **<font color="red">First and foremost:</font>** To fork these repos, open the  [GitLab project creation page](https://gitlab.cern.ch/projects/new) and then select _Import project_ -> _Repository by URL_. Please make sure you've forked the starter repos into your own namespace before cloning and making commits to them, otherwise you'll run into permissions issues when you try to push your commits! Also remember to set the visibility level to _Public_.
* Regarding authentication with ``kinit``:
  * If you are from CERN and use gitlab.cern.ch: Remember to add your CERN credentials as CI/CD variables to
    both repos for the `kinit` authentication in the `.gitlab-ci.yml` files to work.
    To do so, go to _Settings_ -> _CI/CD_ -> _Variables_ and create two new variables:
     * `CERN_USER` should contain your CERN username
     * `SERVICE_PASS` should contain your password.
  * Else, you can remove the ``kinit`` line from `.gitlab-ci.yml` and use the public EOS datasets:
    * `root://eospublic.cern.ch//eos/root-eos/HiggsTauTauReduced` for the skimming repo.
    * `root://eospublic.cern.ch//eos/opendata/cms/upload/apb2023/histograms.root` for the fitting repo.
* For the fitting code repo, the [fit_simple](https://github.com/hsf-training/hsf-training-cms-analysis-snapshot-stats/blob/master/.gitlab-ci.yml#L5) step in `.gitlab-ci.yml` expects to receive the file `histograms.root` produced by the skimming code. In case you haven't had a chance to produce this file yet, it can be downloaded from [here](https://eospublichttp.cern.ch//eos/opendata/cms/upload/apb2023/histograms.root). In any case you can:
  * use the public EOS datasets mentioned above.
  * If you are from CERN, you can copy the downloaded file to your personal eos user space (`root://eosuser.cern.ch//eos/user/[first_letter_of_username]/[username]`).

{% include links.md %}
