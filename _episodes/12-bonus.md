---
title: "Bonus Episode: Building and deploying a Docker container to Github Packages"
teaching: 40
exercises: 0
questions:
- How to build a Docker container for python packages?
- How to share Docker images?
objectives:
- To be able to build a Docker container and share it via GitHub packages
keypoints:
- Python packages can be installed in Docker images along with ubuntu packages.
- It is possible to publish and share Docker images over github packages.
---

> ## Prerequisites
> For this lesson, you will need,
> * Knowledge of Git [SW Carpentry Git-Novice Lesson](https://swcarpentry.github.io/git-novice/)
> * Knowledge of GitHub CI/CD [HSF Github CI/CD Lesson](https://hsf-training.github.io/hsf-training-cicd-github/)
{: .prereq}

## Docker Container for python packages

Python packages can be installed using a Docker image. The following example illustrates how to write a Dockerfile for building an image containing python packages.

```docker
FROM ubuntu:20.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
 && apt-get install wget -y \
 && apt-get install dpkg-dev cmake g++ gcc binutils libx11-dev libxpm-dev \
  libxft-dev libxext-dev python3 libssl-dev libgsl0-dev libtiff-dev \
  python3-pip -y

 RUN pip3 install numpy \
  && pip3 install awkward \
  && pip3 install uproot4 \
  && pip3 install particle \
  && pip3 install hepunits \
  && pip3 install matplotlib \
  && pip3 install mplhep \
  && pip3 install vector \
  && pip3 install fastjet \
  && pip3 install iminuit
```


As we see, several packages are installed.


## Publish Docker images with GitHub Packages and share them!

It is possible to publish Docker images with [GitHub packages](https://github.com/features/packages).
To do so, one needs to use GitHub CI/CD. A step-by-step guide is presented here.

* **Step 1**: Create a GitHub repository and clone it locally.
* **Step 2**: In the empty repository, make a folder called `.github/workflows`. In this folder we will store the file containing the YAML script for a GitHub workflow, named `Docker-build-deploy.yml` (the name doesn't really matter).
* **Step 3**: In the top directory of your GitHub repository, create a file named `Dockerfile`.
* **Step 4**: Copy-paste the content above and add it to the Dockerfile. (In principle it is possible to build this image locally, but we will not do that here, as we wish to build it with GitHub CI/CD).
* **Step 5**: In the `Docker-build-deploy.yml` file, add the following content:

{% raw %}
```yaml
name: Create and publish a Docker image

on:
  push:
    branches:
      - main
      - master

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push-image:
    runs-on: ubuntu-latest

    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Log in to the Container registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Docker Metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```
{% endraw %}


The above script is designed to build and publish a Docker image with [GitHub packages](https://github.com/features/packages).


* **Step 6**: Add LICENSE and README as recommended in the [SW Carpentry Git-Novice Lesson](https://swcarpentry.github.io/git-novice/), and then the repository is good to go.
