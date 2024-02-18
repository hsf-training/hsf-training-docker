---
title: "Github and Dockerhub for Automated Environment Preservation"
teaching: 20
exercises: 25
questions:
- "What do I need to do to enable this automated environment preservation on github?"
objectives:
- "Learn how to write a Dockerfile to containerize your analysis code and environment."
- "Understand how to use github + dockerhub to enable automatic environment preservation."
keypoints:
- "Combination of github and dockerhub allows you to automatically build the docker containers every time you push to a repository."
---
<iframe width="427" height="251" src="https://www.youtube.com/embed/YmLmWm3RNwg?list=PLKZ9c4ONm-VnqD5oN2_8tXO0Yb1H_s0sj" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Introduction
In this section, we learn how to combine the forces of dockerhub and github to automatically keep your analysis environment up-to-date.

We will be doing this using the [CMS OpenData HTauTau Analysis Payload](https://hsf-training.github.io/hsf-training-cms-analysis-webpage/). Specifically, we will be using two "snapshots" of this code which are the repositories described on the [setup page](https://hsf-training.github.io/hsf-training-docker/setup.html) of this training.  A walkthrough of how to setup those repositories can also be found [on this video](https://www.youtube.com/watch?v=krsBupoxoNI&list=PLKZ9c4ONm-VnqD5oN2_8tXO0Yb1H_s0sj&index=7). The "snapshot" repositories are available on GitHub ([skimmer repository](https://github.com/hsf-training/hsf-training-cms-analysis-snapshot) and [statistics repository](https://github.com/hsf-training/hsf-training-cms-analysis-snapshot-stats) ). If you don't already have this setup, take a detour now and watch that video and revisit the setup page.


### Writing your Dockerfile

The goal of automated environment preservation is to create a docker image that you can **immediately** start executing your analysis code inside upon startup. Let's review the needed components for this.

 * Set up the OS, system libraries, and other dependencies that your code depends on,
 * Add your analysis code to the container, and
 * Build the code so that it can just be executed trivially inside the container.

As we've seen, all these components can be encoded in a Dockerfile. So the first step to set up automated image building is to add a Dockerfile to the repo specifying these components.

> ## The `rootproject/root` docker image
> In this tutorial, we build our analysis environments on top of the `rootproject/root` base image ([link to project area on docker hub](https://hub.docker.com/r/rootproject/root)) with conda. This image comes with root 6.22 and python 3.8 pre-installed. It also comes with XrootD for downloading files from eos.
> The `rootproject/root` is itself built with a [Dockerfile](https://github.com/root-project/root-docker/blob/6.22.06-conda/conda/Dockerfile), which uses conda to install root and python on top of another base image (`condaforge/miniforge3`).
{: .callout}

> ## Exercise (15 min)
> Working from your bash shell, `cd` into the top level of the repo you use for skimming, that being the "event selection" snapshot of the CMS HTauTau analysis payload. Create an empty file named `Dockerfile`.
>
> ~~~bash
> touch Dockerfile
> ~~~
> {: .source}
>
> Now open the Dockerfile with a text editor and, starting with the following skeleton, fill in the FIXMEs to make a Dockerfile that fully specifies your analysis environment in this repo.
>
> ~~~yaml
> # Start from the rootproject/root:6.22.06-conda base image
> [FIXME]
>
> # Put the current repo (the one in which this Dockerfile resides) in the /analysis/skim directory
> # Note that this directory is created on the fly and does not need to reside in the repo already
> [FIXME]
>
> # Make /analysis/skim the default working directory (again, it will create the directory if it doesn't already exist)
> [FIXME]
>
> # Compile an executable named 'skim' from the skim.cxx source file
> RUN echo ">>> Compile skimming executable ..." &&  \
>     COMPILER=[FIXME] && \
>     FLAGS=[FIXME] && \
>     [FIXME]
> ~~~
> {: .source}
>
> Hint: have a look at `skim.sh` if you are unsure about how to complete the last `RUN` statement!
> > ## Solution
> > ~~~yaml
> > # Start from the rootproject/root base image with conda
> > FROM rootproject/root:6.22.06-conda
> >
> > # Put the current repo (the one in which this Dockerfile resides) in the /analysis/skim directory
> > # Note that this directory is created on the fly and does not need to reside in the repo already
> > COPY . /analysis/skim
> >
> > # Make /analysis/skim the default working directory (again, it will create the directory if it doesn't already exist)
> > WORKDIR /analysis/skim
> >
> > # Compile an executable named 'skim' from the skim.cxx source file
> > RUN echo ">>> Compile skimming executable ..." &&  \
> > COMPILER=$(root-config --cxx) &&  \
> > FLAGS=$(root-config --cflags --libs) &&  \
> > $COMPILER -g -std=c++11 -O3 -Wall -Wextra -Wpedantic -o skim skim.cxx $FLAGS
> > ~~~
> > {: .source}
> {: .solution}
>
> Once you're happy with your Dockerfile, you can commit it to your repo and push it to github.
{: .challenge}

> ## Hints
> As you're working, you can test whether the Dockerfile builds successfully using the `docker build` command. Eg.
> ~~~bash
> docker build -t payload_analysis .
> ~~~
> {: .source}
>
> When your image builds successfully, you can `run` it and poke around to make sure it's set up exactly as you want, and that you can successfully run the executable you built:
> ~~~bash
> docker run -it --rm payload_analysis /bin/bash
> ~~~
> {: .source}
{: .callout}

## Automatic image building with github + dockerhub

Now, you can proceed with updating your `.gitlab-ci.yml` to actually build the container during the CI/CD pipeline and store it in the gitlab registry. You can later pull it from the gitlab registry just as you would any other container, but in this case using your CERN credentials.

> ## Not from CERN?
> If you do not have a CERN computing account with access to [gitlab.cern.ch](https://gitlab.cern.ch), then everything discussed here is also available on [gitlab.com](https://gitlab.com), which offers CI/CD tools, including the docker builder.
> Furthermore, you can achieve the same with GitHub + Github Container Registry.
> To learn more about these methods, see the next subsections.
{: .callout}

Add the following lines at the end of the `.gitlab-ci.yml` file to build the image with Kaniko and save it to the docker registry.
For more details about building docker images on CERN's GitLab, see the [Building docker images](https://gitlab.docs.cern.ch/docs/Build%20your%20application/Packages%20&%20Registries/using-gitlab-container-registry#building-docker-images) docs page.

~~~yaml
build_image:
  stage: build
  variables:
    IMAGE_DESTINATION: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG-$CI_COMMIT_SHORT_SHA
  image:
    # The kaniko debug image is recommended because it has a shell, and a shell is required for an image to be used with GitLab CI/CD.
    name: gcr.io/kaniko-project/executor:debug
    entrypoint: [""]
  script:
    # Prepare Kaniko configuration file
    - echo "{\"auths\":{\"$CI_REGISTRY\":{\"username\":\"$CI_REGISTRY_USER\",\"password\":\"$CI_REGISTRY_PASSWORD\"}}}" > /kaniko/.docker/config.json
    # Build and push the image from the Dockerfile at the root of the project.
    - /kaniko/executor --context $CI_PROJECT_DIR --dockerfile $CI_PROJECT_DIR/Dockerfile --destination $IMAGE_DESTINATION
    # Print the full registry path of the pushed image
    - echo "Image pushed successfully to ${IMAGE_DESTINATION}"
~~~
{: .source}

<!--Now, remove the line `image: rootproject/root` underneath the stages, since the image-building stage uses its own dedicated image for automated image building. You'll then need to explicitly specify that the other stages use this image by adding the line `image: rootproject/root` to the stages, since it's no longer a global specification.-->

Once this is done, you can commit and push the updated `.gitlab-ci.yml` file to your gitlab repo and check to make sure the pipeline passed. If it passed, the repo image built by the pipeline should now be stored on the docker registry, and be accessible as follows:

~~~bash
docker login gitlab-registry.cern.ch
docker pull gitlab-registry.cern.ch/[repo owner's username]/[skimming repo name]:[branch name]-[shortened commit SHA]
~~~
{: .source}

You can also go to the container registry on the gitlab UI to see all the images you've built:

<img src="../fig/ContainerRegistry.png" alt="ContainerRegistry" style="width:900px">

Notice that the script to run is just a dummy 'ignore' command. This is because using the docker-image-build tag, the jobs always land on special runners that are managed by CERN IT which run a custom script in the background. You can safely ignore the details.

> ## Recommended Tag Structure
> You'll notice the environment variable `IMAGE_DESTINATION` in the `.gitlab-ci.yml` script above. This controls the name of the Docker image that is produced in the CI step. Here, the image name will be `<reponame>:<branch or tagname>-<short commit SHA>`. The shortened 8-character commit SHA ensures that each image created from a different commit will be unique, and you can easily go back and find images from previous commits for debugging, etc.
>
> As you'll see tomorrow, it's recommended when using your images as part of a REANA workflow to make a unique image for each gitlab commit, because REANA will only attempt to update an image that it's already pulled if it sees that there's a new tag associated with the image.
>
> If you feel it's overkill for your specific use case to save a unique image for every commit, the `-$CI_COMMIT_SHORT_SHA` can be removed. Then the `$CI_COMMIT_REF_SLUG` will at least ensure that images built from different branches will not overwrite each other, and tagged commits will correspond to tagged images.
{: .callout}

### Alternative: GitLab.com

This training module is rather CERN-centric and assumes you have a CERN computing account with access to [gitlab.cern.ch](https://gitlab.cern.ch).  If this is not the case, then as with the [CICD training module](https://hsf-training.github.io/hsf-training-cicd/), everything can be carried out using [gitlab.com](https://gitlab.com) with a few slight modifications.
In particular, you will have to specify that your pipeline job that builds the image is executed on a special type of runner with the appropriate `services`.  However, unlike at CERN, you can use the docker commands that you have seen in the previous episodes to build and push the docker images.

Add the following lines at the end of the `.gitlab-ci.yml` file to build the image and save it to the docker registry.

~~~yaml
build_image:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  variables:
    IMAGE_DESTINATION: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG-$CI_COMMIT_SHORT_SHA
  script:
    - docker build -t $IMAGE_DESTINATION .
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker push $IMAGE_DESTINATION
~~~
{: .source}

In this job, the specific `image: docker:latest`, along with specifying the `services` to contain `docker:dind` are needed to be able to execute docker commands. If you are curious to read about this in detail, refer to the [official gitlab documentation](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html) or [this example](https://gitlab.com/gitlab-examples/docker).

In the `script` of this job there are three components :
  - [`docker build`](https://docs.docker.com/engine/reference/commandline/build/) : This is performing the same build of our docker image to the tagged image which we will call `<reponame>:<branch or tagname>-<short commit SHA>`
  - [`docker login`](https://docs.docker.com/engine/reference/commandline/login/) : This call is performing [an authentication of the user to the gitlab registry](https://docs.gitlab.com/ee/user/packages/container_registry/#authenticating-to-the-gitlab-container-registry) using a set of [predefined environment variables](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html) that are automatically available in any gitlab repository.
  - [`docker push`](https://docs.docker.com/engine/reference/commandline/push/) : This call is pushing the docker image which exists locally on the runner to the gitlab.com registry associated with the repository against which we have performed the authentication in the previous step.

If the job runs successfully, then in the same way as described for [gitlab.cern.ch](https://gitlab.cern.ch) in the previous section, you will be able to find the `Container Registry` on the left hand icon menu of your gitlab.com web browser and navigate to the image that was pushed to the registry.  Et voila, c'est fini, exactement comme au CERN!

You can also build Docker images on [github.com](https://github.com) and push them to the GitHub Container Registry ([ghcr.io](https://ghcr.io)) with the help of [GitHub Actions](https://github.com/features/actions).
The bonus episode [Building and deploying a Docker container to Github Packages](/hsf-training-docker/12-bonus/index.html) explains how to do so.

> ## Tag your docker image
> Notice that the command above had a ``<tag>`` specified. A tag uniquely identifies a docker image and is usually used to identify different versions of the same image. The tag name has to be written with ASCII symbols.



## An updated version of `skim.sh`

> ## Exercise (10 mins)
> Since we're now taking care of building the skimming executable during image building, let's make an updated version of `skim.sh` that excludes the step of building the `skim` executable.
>
> The updated script should just directly run the pre-existing `skim` executable on the input samples. You could call it eg. `skim_prebuilt.sh`. We'll be using this updated script in an exercise later on in which we'll be going through the full analysis in containers launched from the images.
>
> Once you're happy with the script, you can commit and push it to the repo.
>
> > ## Solution
> > ~~~bash
> > #!/bin/bash
> >
> > INPUT_DIR=$1
> > OUTPUT_DIR=$2
> >
> > # Sanitize input path, XRootD breaks if we double accidentally a slash
> > if [ "${INPUT_DIR: -1}" = "/" ];
> > then
> > INPUT_DIR=${INPUT_DIR::-1}
> > fi
> >
> > # Skim samples
> > while IFS=, read -r SAMPLE XSEC
> > do
> > echo ">>> Skim sample ${SAMPLE}"
> > INPUT=${INPUT_DIR}/${SAMPLE}.root
> > OUTPUT=${OUTPUT_DIR}/${SAMPLE}Skim.root
> > LUMI=11467.0 # Integrated luminosity of the unscaled dataset
> > SCALE=0.1 # Same fraction as used to down-size the analysis
> > ./skim $INPUT $OUTPUT $XSEC $LUMI $SCALE
> > done < skim.csv
> > ~~~
> > {: .source}
> {: .solution}
{: .challenge}

{% include links.md %}
