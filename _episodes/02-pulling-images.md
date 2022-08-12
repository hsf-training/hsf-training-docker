---
title: "Pulling Images"
teaching: 10
exercises: 5
questions:
- "How are images downloaded?"
- "How are images distinguished?"
objectives:
- "Pull images from Docker Hub image registry"
- "List local images"
- "Introduce image tags"
keypoints:
- "Pull images with `docker pull`"
- "List images with `docker images`"
- "Image tags distinguish releases or version and are appended to the image name with a colon"
---
<iframe width="427" height="251" src="https://www.youtube.com/embed/JihkukeoNVs?list=PLKZ9c4ONm-VnqD5oN2_8tXO0Yb1H_s0sj" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# Docker Hub

Much like GitHub allows for web hosting and searching for code, the [Docker Hub][docker-hub]
image registry allows the same for Docker images.
Hosting and building of images is [free for public repositories][docker-hub-billing] and
allows for downloading images as they are needed.
Additionally, through integrations with GitHub and Bitbucket, Docker Hub repositories can
be linked against Git repositories so that
[automated builds of Dockerfiles on Docker Hub][docker-hub-builds] will be triggered by
pushes to repositories.

# Pulling Images

To begin with we're going to [pull][docker-docs-pull] down the Docker image we're going
to be working in for the tutorial (note: if you did all the docker pulls in the setup instructions, this image will already be on your machine, in which case docker should notice it's there and not attempt to re-pull it unless it's changed in the meantime):

~~~bash
docker pull matthewfeickert/intro-to-docker
~~~
{: .source}

> ## Permission errors
> If you run into a permission error, use `sudo docker run ...` as a quick fix.
> To fix this for the future (**recommended**), see [the docker docs](https://docs.docker.com/install/linux/linux-postinstall/).
{: .callout}

and then [list the images][docker-docs-images] that we have available to us locally

~~~bash
docker images
~~~
{: .source}

If you have many images and want to get information on a particular one you can apply a
filter, such as the repository name

~~~bash
docker images matthewfeickert/intro-to-docker
~~~
{: .source}

~~~
REPOSITORY                        TAG                 IMAGE ID            CREATED             SIZE
matthewfeickert/intro-to-docker   latest              cf6508749ee0        3 months ago        1.49GB
~~~
{: .output}

or more explicitly

~~~bash
docker images --filter=reference="matthewfeickert/intro-to-docker"
~~~
{: .source}

~~~
REPOSITORY                        TAG                 IMAGE ID            CREATED             SIZE
matthewfeickert/intro-to-docker   latest              cf6508749ee0        3 months ago        1.49GB
~~~
{: .output}

You can see here that there is the `TAG` field associated with the
`matthewfeickert/intro-to-docker` image.
Tags are way of further specifying different versions of the same image.
As an example, let's pull the buster release tag of the
[Debian image](https://hub.docker.com/_/debian) (again, if it was already pulled during setup, docker won't attempt to re-pull it unless it's changed since last pulled).

~~~bash
docker pull debian:buster
docker images debian
~~~
{: .source}

~~~
buster: Pulling from library/debian
<some numbers>: Pull complete
Digest: sha256:<the relevant SHA hash>
Status: Downloaded newer image for debian:buster
docker.io/library/debian:buster

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
debian              buster              00bf7fdd8baf        5 weeks ago         114MB
~~~
{: .output}

> ## Pulling Python
>
> Pull the image python3.7-slim for Python 3.7 and then list all `python` images along with
> the `matthewfeickert/intro-to-docker` image
>
> > ## Solution
> >
> > ~~~bash
> > docker pull python:3.7-slim
> > docker images --filter=reference="matthewfeickert/intro-to-docker" --filter=reference="python"
> > ~~~
> > {: .source}
> >
> > ~~~
> > REPOSITORY                        TAG                 IMAGE ID            CREATED             SIZE
> > python                            3.7                 e440e2151380        23 hours ago        918MB
> > matthewfeickert/intro-to-docker   latest              cf6508749ee0        3 months ago        1.49GB
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

[docker-hub]: https://hub.docker.com/
[docker-hub-billing]: https://hub.docker.com/billing-plans/
[docker-hub-builds]: https://docs.docker.com/docker-hub/builds/
[docker-docs-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-docs-images]: https://docs.docker.com/engine/reference/commandline/images/

{% include links.md %}
