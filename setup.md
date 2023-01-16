---
title: Setup
---

## Docker installations

To install Docker Community Edition on your Linux, Mac, or Windows machine follow the [instructions in the Docker docs](https://docs.docker.com/install/).

If you are using Linux, then please also follow these [post installation instructions](https://docs.docker.com/engine/install/linux-postinstall/).

To test your docker set up, run the following command:

~~~bash
docker run hello-world
~~~

### Optional: Fetch docker images in advance

Once you've got docker up and running, do the following docker image pulls in advance to save waiting during the tutorial:

~~~bash
docker pull matthewfeickert/intro-to-docker
docker pull debian:buster-slim
docker pull python:2.7-slim
docker pull python:3.7-slim
docker pull rootproject/root:6.22.06-conda
~~~

## Analysis Code

Later in this tutorial, you will be asked to work with a simple analysis that utilizes the CMS OpenData to search for Higgs to 2 tau leptons.
The full analysis itself can be found [here](https://github.com/hsf-training/hsf-training-cms-analysis) - and there is a dedicated set of [training lessons](https://hsf-training.github.io/hsf-training-cms-analysis-webpage/index.html) ([videos available](https://www.youtube.com/watch?v=gplMywJAFDI&list=PLKZ9c4ONm-Vk0wnDKaaovoEkOk3PVdL0V)).
These are then used as input to the [continuous integration training](https://hsf-training.github.io/hsf-training-cicd/) ([videos available](https://www.youtube.com/watch?v=NcVGX8zWzQY&list=PLKZ9c4ONm-VmmTObyNWpz4hB3Hgx8ZWSb)).

Though it is best if you work through these lessons, the key point is that we will assume you have two GitLab repos in your namespace :

* one containing code to skim the NanoAOD-like samples and convert them to histograms,
* one  containing the code needed to do the final fit.

You can either do this yourself, or if you'd like to start fresh (by following the above training) or use the following  'starter' repos that you can directly fork into your namespace and work directly from:

**Skimming code:** [https://github.com/hsf-training/hsf-training-cms-analysis-snapshot](https://github.com/hsf-training/hsf-training-cms-analysis-snapshot)

**Fitting code:** [https://github.com/hsf-training/hsf-training-cms-analysis-snapshot-stats](https://github.com/hsf-training/hsf-training-cms-analysis-snapshot-stats)

A few things to keep in mind if starting from these 'starter repos':

* **<font color="red">First and foremost:</font>** Please make sure you've forked the starter repos into your own namespace before cloning and making commits to them, otherwise you'll run into permissions issues when you try to push your commits!!
* Regarding authentication with ``kinit``:
  * If you are from CERN and use gitlab.cern.ch: Remember to add your CERN credentials as CI/CD variables to
    both repos for the `kinit` authentication in the `.gitlab-ci.yml` files to work.
  * Else, you can remove the ``kinit`` line and use the public EOS dataset
* For the fitting code repo, the [fit_simple](https://github.com/hsf-training/hsf-training-cms-analysis-snapshot-stats/blob/master/.gitlab-ci.yml#L5) gitlab-ci.yml file expects to receive the file `histograms.root` produced by the skimming code. In case you haven't had a chance to produce this file yet, it can be downloaded from [here](https://cernbox.cern.ch/index.php/s/LADW94G9fjY7hjF).
  * If you are from CERN, you'll need to copy this file to your personal eos user space (`root://eosuser.cern.ch//eos/user/[first_letter_of_username]/[username]`)
  * else: use the public eos link.

{% include links.md %}
