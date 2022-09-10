---
title: "Removal of Containers and Images"
teaching: 5
exercises: 5
questions:
- "How do you cleanup old containers?"
- "How do you delete images?"
objectives:
- "Learn how to cleanup after Docker"
keypoints:
- "Remove containers with `docker rm <CONTAINER NAME>`"
- "Remove images with `docker rmi <IMAGE ID>`"
- "Perform faster cleanup with `docker container prune`, `docker image prune`, and `docker system prune`"
---
<iframe width="427" height="251" src="https://www.youtube.com/embed/Gsp6EapBcoo?list=PLKZ9c4ONm-VnqD5oN2_8tXO0Yb1H_s0sj" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

You can cleanup/remove a container [`docker rm`][docker-docs-rm]
~~~bash
docker rm <CONTAINER NAME>
~~~
{: .source}

> ## Remove old containers
>
> Start an instance of the tutorial container, exit it, and then remove it with
> `docker rm`
>
> > ## Solution
> >
> > ~~~bash
> > docker run matthewfeickert/intro-to-docker:latest
> > docker ps -a
> > docker rm <CONTAINER NAME>
> > docker ps -a
> > ~~~
> > {: .source}
> >
> > ~~~
> >CONTAINER ID        IMAGE         COMMAND             CREATED            STATUS                     PORTS               NAMES
> ><generated id>      <image:tag>   "/bin/bash"         n seconds ago      Exited (0) t seconds ago                       <name>
> >
> ><generated id>
> >
> >CONTAINER ID        IMAGE         COMMAND             CREATED            STATUS                     PORTS               NAMES
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

You can remove an image from your computer entirely with [`docker rmi`][docker-docs-rmi]
~~~bash
docker rmi <IMAGE ID>
~~~
{: .source}

> ## Remove an image
>
> Pull down the Python 2.7 image (2.7-slim tag) from Docker Hub and then delete it.
>
> > ## Solution
> >
> > ~~~bash
> > docker pull python:2.7-slim
> > docker images python
> > docker rmi <IMAGE ID>
> > docker images python
> > ~~~
> > {: .source}
> >
> > ~~~
> >2.7: Pulling from library/python
> ><some numbers>: Pull complete
> ><some numbers>: Pull complete
> ><some numbers>: Pull complete
> ><some numbers>: Pull complete
> ><some numbers>: Pull complete
> ><some numbers>: Pull complete
> ><some numbers>: Pull complete
> ><some numbers>: Pull complete
> >Digest: sha256:<the relevant SHA hash>
> >Status: Downloaded newer image for python:2.7-slim
> >docker.io/library/python:2.7-slim
> >
> >REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
> >python              2.7-slim            d75b4eed9ada        14 hours ago        886MB
> >python              3.7-slim            e440e2151380        23 hours ago        918MB
> >
> >Untagged: python@sha256:<the relevant SHA hash>
> >Deleted: sha256:<layer SHA hash>
> >Deleted: sha256:<layer SHA hash>
> >Deleted: sha256:<layer SHA hash>
> >Deleted: sha256:<layer SHA hash>
> >Deleted: sha256:<layer SHA hash>
> >Deleted: sha256:<layer SHA hash>
> >Deleted: sha256:<layer SHA hash>
> >Deleted: sha256:<layer SHA hash>
> >Deleted: sha256:<layer SHA hash>
> >Deleted: sha256:<layer SHA hash>
> >
> >REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
> >python              3.7-slim            e440e2151380        23 hours ago        918MB
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

> ## Helpful cleanup commands
> What is helpful is to have Docker detect and remove unwanted images and containers for you.
> This can be done with `prune`, which depending on the context will remove different things.
> - [`docker container prune`](https://docs.docker.com/engine/reference/commandline/container_prune/) removes all stopped containers, which is helpful to clean up forgotten stopped containers.
> - [`docker image prune`](https://docs.docker.com/engine/reference/commandline/image_prune/) removes all unused or dangling images (images that do not have a tag). This is helpful for cleaning up after builds.
> - [`docker system prune`](https://docs.docker.com/engine/reference/commandline/system_prune/) removes all stopped containers, dangling images, and dangling build caches. This is very helpful for cleaning up everything all at once.
{: .callout}

[docker-docs-rm]: https://docs.docker.com/engine/reference/commandline/rm/
[docker-docs-rmi]: https://docs.docker.com/engine/reference/commandline/rmi/

{% include links.md %}
