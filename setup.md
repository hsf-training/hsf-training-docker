---
title: Setup
---

## Docker installations

To install Docker on your machine follow the official instructions for [Linux](https://docs.docker.com/engine/install/#server), [Mac](https://docs.docker.com/desktop/install/mac-install/), or [Windows](https://docs.docker.com/desktop/install/windows-install/).

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

You can either do this yourself by following the aforementioned lessons, or if you'd like to start fresh you can use the following  'starter' repos that you can directly fork into your namespace and work directly from:

**Skimming code:** [https://github.com/hsf-training/hsf-training-cms-analysis-snapshot](https://github.com/hsf-training/hsf-training-cms-analysis-snapshot)

**Fitting code:** [https://github.com/hsf-training/hsf-training-cms-analysis-snapshot-stats](https://github.com/hsf-training/hsf-training-cms-analysis-snapshot-stats)

A few things to keep in mind if starting from these 'starter repos':

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
