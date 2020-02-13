---
title: "Introduction"
teaching: 5
exercises: 5
questions:
- "What are containers?"
objectives:
- "First learning objective."
keypoints:
- "First key point. Brief Answer to questions."
---

# Documentation

The [official Docker documentation and tutorial][docker-tutorial] can be found on the
Docker website.
It is quite thorough and useful.
It is an excellent guide that should be routinely visited, but the emphasis of this
introduction is on using Docker, not how Docker itself works.

A note up front, Docker has very similar syntax to Git and Linux, so if you are familiar
with the command line tools for them then most of Docker should seem somewhat natural
(though you should still read the docs!).

[![Docker logo](https://www.docker.com/sites/default/files/social/docker_twitter_share_new.png)](https://www.docker.com/)

# Docker Images and Containers

It is still important to know what Docker _is_ and what the components of it _are_.
Docker images are executables that bundle together all necessary components for an
application or an environment.
[Docker containers][docker-containers] are the runtime instances of images &mdash; they
are images with a state.

Importantly, containers share the host machine's OS system kernel and so don't require an
OS per application.
As discrete processes containers take up only as much memory as necessary, making them
very lightweight and fast to spin up to run.

[![Docker structure](https://www.docker.com/sites/default/files/styles/large/public/container-what-is-container.png)](https://www.docker.com/resources/what-container)

> ## Singularity
> Docker is the most popular containerization tool these days, particularly in industry, but it's not the only one. There are other kids on the block including Rocket and Singularity which are in use, but just haven't gained as much large-scale traction. 
>
> Singularity in particular is used widely in HPC, and particularly by CMS, so you may have need to familiarize yourself with it at some point. 
> 
> The RECAST FAQ includes a brief intro to the important differences between docker and singularity. 
{: .callout}

[docker-tutorial]: https://docs.docker.com/get-started
[docker-containers]: https://www.docker.com/resources/what-container

{% include links.md %}
