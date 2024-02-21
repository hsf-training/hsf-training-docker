---
title: "Removal of Containers and Images"
teaching: 5
exercises: 5
questions:
- "How do you cleanup old containers?"
- "How do you delete images?"
objectives:
- "Learn how to cleanup after working with containers"
keypoints:
- "Remove containers with `podman rm <CONTAINER NAME>`"
- "Remove images with `podman rmi <IMAGE ID>`"
- "Perform faster cleanup with `podman container prune`, `podman image prune`, and `podman system prune`"
---
<iframe width="427" height="251" src="https://www.youtube.com/embed/Gsp6EapBcoo?list=PLKZ9c4ONm-VnqD5oN2_8tXO0Yb1H_s0sj" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

You can cleanup/remove a container with [`podman rm`][podman-docs-rm]
~~~bash
podman rm <CONTAINER NAME>
~~~
{: .source}

> ## Remove old containers
>
> Start an instance of the tutorial container, exit it, and then remove it with
> `podman rm`
>
> > ## Solution
> >
> > ~~~bash
> > podman run matthewfeickert/intro-to-docker:latest
> > podman ps -a
> > podman rm <CONTAINER NAME>
> > podman ps -a
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

You can remove an image from your computer entirely with [`podman rmi`][podman-docs-rmi]
~~~bash
podman rmi <IMAGE ID>
~~~
{: .source}

> ## Remove an image
>
> Pull down the Python 2.7 image (2.7-slim tag) from Docker Hub and then delete it.
>
> > ## Solution
> >
> > ~~~bash
> > podman pull python:2.7-slim
> > podman images python
> > podman rmi <IMAGE ID>
> > podman images python
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
> >python              3.9-slim            e440e2151380        23 hours ago        918MB
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
> >python              3.9-slim            e440e2151380        23 hours ago        918MB
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

> ## Helpful cleanup commands
> What is helpful is to have a command to detect and remove unwanted images and containers for you.
> This can be done with `prune`, which depending on the context will remove different things.
> - [`podman container prune`](https://docs.podman.io/en/latest/markdown/podman-container-prune.1.html) removes all stopped containers, which is helpful to clean up forgotten stopped containers.
> - [`podman image prune`](https://docs.podman.io/en/latest/markdown/podman-image-prune.1.html) removes all unused or dangling images (images that do not have a tag). This is helpful for cleaning up after builds.
> - [`podman system prune`](https://docs.podman.io/en/stable/markdown/podman-system-prune.1.html) removes all stopped containers, dangling images, and dangling build caches. This is very helpful for cleaning up everything all at once.
{: .callout}

[podman-docs-rm]: https://docs.podman.io/en/stable/markdown/podman-rm.1.html
[podman-docs-rmi]: https://docs.podman.io/en/latest/markdown/podman-rmi.1.html

{% include links.md %}
