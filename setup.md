---
title: Setup
---

## Docker installations

To install Docker Community Edition on your Linux, Mac, or Windows machine follow the [instructions in the Docker docs](https://docs.docker.com/install/).

Once you've got docker up and running, do the following docker image pulls in advance to save waiting during the tutorial:

~~~bash
docker pull matthewfeickert/intro-to-docker
docker pull debian:buster-slim
docker pull python:2.7-slim
docker pull python:3.7-slim
docker pull rootproject/root-conda:6.18.04
~~~

## Analysis Code

Having completed yesterday's gitlab-CI tutorial, we assume you now have two gitlab repos in your namespace: one containing code to skim the NanoAOD-like samples and convert them to histograms, and the other containing the code needed to do the final fit.

In case you'd like to start fresh, we provide two such 'starter' repos that you can fork into your namespace and work directly from:

**Skimming code:** [repo on gitlab.cern.ch](https://gitlab.cern.ch/awesome-workshop/awesome-analysis-eventselection-stage2)

**Fitting code:** [repo on gitlab.cern.ch](https://gitlab.cern.ch/awesome-workshop/awesome-analysis-statistics-stage2)

A few things to keep in mind if starting from these 'starter repos':

* **<font color="red">First and foremost:</font>** Please make sure you've forked the starter repos into your own namespace before cloning and making commits to them, otherwise you'll run into permissions issues when you try to push your commits!!
* Remember to add your CERN credentials as CI/CD variables to both repos for the `kinit` authentication in the `.gitlab-ci.yml` files to work. 
* For the fitting code repo, the [fit_simple](https://gitlab.cern.ch/awesome-workshop/awesome-analysis-statistics-stage2/blob/master/.gitlab-ci.yml#L5) gitlab-ci.yml file expects to receive the file `histograms.root` produced by the skimming code. You'll need to copy this file to your personal eos user space (`root://eosuser.cern.ch//eos/user/[first_letter_of_username]/[username]`). In case you haven't had a chance to produce this file yet, it can be downloaded from [here](https://cernbox.cern.ch/index.php/s/krURAx8AkmnlTGX).

{% include links.md %}
