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
- "Pull images with `podman pull <image-id>`"
- "List all images on the computer and other information with `podman images`"
- "Image tags distinguish releases or version and are appended to the image name with a colon"
---
<iframe width="427" height="251" src="https://www.youtube.com/embed/Wkqt0eJihIA?si=qFVGhTygicu43JUm" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# Docker Hub

Much like how GitHub allows for web hosting and searching for code, the [Docker Hub][docker-hub]
image registry allows the same for Docker images.
Hosting images is [free for public repositories][docker-hub-billing] and
allows for downloading images as they are needed.

Additionally, through integrations with GitHub and Bitbucket, Docker Hub repositories can
be linked against Git repositories so that
[automated builds of Dockerfiles on Docker Hub][docker-hub-builds] will be triggered by
pushes to repositories. However, at this moment enabling such a feature requires a Pro (paid) account
or joining the [Docker-Sponsored Open Source Program](https://www.docker.com/community/open-source/application/).
There are other ways of doing this, such as using GitLab/GitHub CI/CD, but that's beyond the scope of this training module.

> ## Docker Hub and Podman
>
> Both Docker and Podman use OCI (Open Container Initiative) compliant images, so you can use the same images with both tools.
> It means Podman can pull and run images from Docker Hub.
>
> By default, `podman pull` pulls an image from Docker Hub if a registry is not specified in the command line argument.
{: .callout}

# Pulling Images

To begin with we're going to [pull][podman-docs-pull] down the image we're going
to be working in for the tutorial (note: if you did all the docker pulls in the setup instructions, this image will already be on your machine, in which case podman should notice it's there and not attempt to re-pull it unless it's changed in the meantime):

~~~bash
podman pull matthewfeickert/intro-to-docker
~~~
{: .source}

> ## No search registry defined
> Some installations of Podman may end with an error like `Error: unable to pull matthewfeickert/intro-to-docker:latest: unable to find registry in the system`.
> This is because the default registry is not defined. You can fix this by adding the `docker.io` registry at the command:
> ~~~bash
> podman pull docker.io/matthewfeickert/intro-to-docker
> ~~~
>
> Or, to pull images by default from Docker Hub, adding the following line to the `/etc/containers/registries.conf` file:
> ~~~bash
> unqualified-search-registries=["docker.io"]
> ~~~
>
{: .callout}

> ## Connection errors
> If using Podman or Docker in a non-Linux machine you run into an error like `Error: unable to connect to Podman`,
> make sure that the Podman or Docker desktop application is running.
>
> Remember that in such environments, Podman or Docker use a virtual machine to run the containers.
{: .callout}

and then [list the images][podman-docs-images] that we have available to us locally

~~~bash
podman images
~~~
{: .source}

If you have many images and want to get information on a particular one you can apply a
filter, such as the repository name

~~~bash
podman images matthewfeickert/intro-to-docker
~~~
{: .source}

~~~
REPOSITORY                        TAG                 IMAGE ID            CREATED             SIZE
matthewfeickert/intro-to-docker   latest              cf6508749ee0        3 months ago        1.49GB
~~~
{: .output}

or more explicitly

~~~bash
podman images --filter=reference="matthewfeickert/intro-to-docker"
~~~
{: .source}

~~~
REPOSITORY                        TAG                 IMAGE ID            CREATED             SIZE
matthewfeickert/intro-to-docker   latest              cf6508749ee0        3 months ago        1.49GB
~~~
{: .output}

You can see here that there is the `TAG` field associated with the
`matthewfeickert/intro-to-docker` image.
Tags are a way of further specifying different versions of the same image.
As an example, let's pull the buster release tag of the
[Debian image](https://hub.docker.com/_/debian) (again, if it was already pulled during setup, podman won't attempt to re-pull it unless it's changed since last pulled).

~~~bash
podman pull debian:buster
podman images debian
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

Check the documentation on [pull][podman-docs-pull] and [images][podman-docs-images] for more information on these commands.

> ## Pulling Python
>
> Pull the image python:3.9-slim for Python 3.9 and then list all `python` images on your computer.
>
> Browse [the official Python images][docker-hub-python] to find available tags and
> read about image variants. What does `-slim` mean?
>
> > ## Solution
> >
> > ~~~bash
> > podman pull python:3.9-slim
> > podman images --filter=reference="python"
> > ~~~
> > {: .source}
> >
> > ~~~
> > REPOSITORY                        TAG                 IMAGE ID            CREATED             SIZE
> > docker.io/library/python          3.9-slim            e440e2151380        2 weeks ago        131 MB
> > ~~~
> >
> >* `python:<version>-slim`: This image does not contain the common packages contained in the default
> >tag and only contains the minimal packages needed to run python
> > {: .output}
> {: .solution}
{: .challenge}

[docker-hub]: https://hub.docker.com/
[docker-hub-billing]: https://www.docker.com/pricing/
[docker-hub-python]: https://hub.docker.com/_/python
[docker-hub-builds]: https://docs.docker.com/docker-hub/builds/
[podman-docs-pull]: https://docs.podman.io/en/latest/markdown/podman-pull.1.html
[podman-docs-images]: https://docs.podman.io/en/stable/markdown/podman-images.1.html

{% include links.md %}
