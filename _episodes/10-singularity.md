---
title: "Optional: Running Containers on LXPLUS Using Singularity"
teaching: 10
exercises: 5
questions:
- "How can I run a container on LXPLUS?"
objectives:
- "Understand some of the differences between Singularity and Docker."
- "Successfully run a custom analysis container on LXPLUS."
keypoints:
- "Singularity needs to be used for running containers on LXPLUS."
- "To run your own container, you need to run Singularity manually."
---
<iframe width="427" height="251" src="https://www.youtube.com/embed/DxUQIBK5on4?list=PLKZ9c4ONm-VnqD5oN2_8tXO0Yb1H_s0sj" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Introduction

Analysis containers allow software development to occur on a local computer        
with the only requirement being that Docker is installed. The same containers 
can be run in GitLab CI/CD via Docker. These containers can also be run on 
LXPLUS, but this requires some additional steps.

When dealing with Analysis Containers, *privileged* containers are often needed. 
These are not available to you on LXPLUS (nor is the `docker`command). 
On LXPLUS, the tool to run containers is Singularity.
**The following commands will therefore all be run on LXPLUS**
(`lxplus7.cern.ch` or later specifically).

## Running custom images with Singularity

Some of the LHC experiments have scripts for running singularity that hide the complexity.
For the purpose of running a custom analysis image, Singularity must be run manually.

As an example, we are going to run a container using the `matthewfeickert/intro-to-docker`
image. Before running Singularity, you should set the cache directory (i.e.
the directory to which the images are being pulled) to a
place outside your AFS space (here we use the `tmp` directory):

~~~
export SINGULARITY_CACHEDIR="/tmp/$(whoami)/singularity"
singularity shell -B /afs -B /eos -B /cvmfs docker://matthewfeickert/intro-to-docker:latest
# try accessing cvmfs inside of the container
source /cvmfs/cms.cern.ch/cmsset_default.sh
~~~
{: .language-bash}

If you are asked for a docker username and password, just hit
enter twice.

One particular difference from Docker is that the image
name needs to be prepended by `docker://` to tell Singularity that this
is a Docker image.
As you can see from the output, Singularity first downloads the layers
from the registry, and is then unpacking the layers into a format that
can be read by Singularity. This is somewhat a technical detail, but 
is different from Docker.

In the next example, we are executing a script with singularity using a Docker CERN Cent0S 7 image.

~~~
export SINGULARITY_CACHEDIR="/tmp/$(whoami)/singularity"
singularity exec -B /afs -B /eos -B /cvmfs docker://cmssw/cc7:latest bash .gitlab/build.sh
~~~
{: .language-bash}

> ## `exec` vs. `shell`
>
> Singularity differentiates between providing you with an
> interactive shell (`singularity shell`) and executing scripts
> non-interactively (`singularity exec`).
>
{: .callout}

> ## `-B` (bind strings)
>
> The -B option allows the user to specify paths to bind to the Singularity container.
> This option is similar to '-v' in docker. By default paths are mounted as rw (read/write),
> but can also be specified as ro (read-only).
>
{: .callout}

## Authentication with Singularity

In case your image is not public, you can authenticate to
the registry in two different ways: either you append the
option `--docker-login` to the `singularity` command, which
makes sense when running interactively, or via environment
variables (e.g. on GitLab):

~~~
export SINGULARITY_DOCKER_USERNAME=${CERNUSER}
export SINGULARITY_DOCKER_PASSWORD='mysecretpass'
~~~
{: .language-bash}

> ## Exercise (5 min)
> Working from lxplus, use Singularity to pull the Docker image created in lesson 8 for the skimming repository and use this image to start an interactive Singularity container.
>
> > ## Solution
> > ~~~bash
> > export SINGULARITY_CACHEDIR="/tmp/$(whoami)/singularity"
> > singularity shell -B /afs -B /eos -B /cvmfs docker://gitlab-registry.cern.ch/[repo owner's username]/[skimming repo name]:[branch name]-[shortened commit SHA] --docker-login
> > ~~~
> > {: .source}
> {: .solution}
> 
{: .challenge}




