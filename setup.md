---
title: Setup
---

## Docker installations

To install Docker Community Edition on your Linux, Mac, or Windows machine follow the [instructions in the Docker docs](https://docs.docker.com/install/).

Once you've got docker up and running, do the following docker image pulls in advance to save waiting during the tutorial:

~~~bash
docker pull matthewfeickert/intro-to-docker
docker pull debian:buster
docker pull python:2.7
docker pull python:3.7
docker pull rootproject/root-conda
~~~

## Analysis Code

Having completed yesterday's gitlab-CI tutorial, we assume you now have two gitlab repos in your namespace: one containing code to skim the NanoAOD-like samples and convert them to histograms, and the other containing the code needed to do the final fit. 

In case you'd like to start fresh, we provide two such 'starter' repos that you can fork into your namespace and work directly from:

**Skimming code:** https://gitlab.cern.ch/awesome-workshop/Payload-Stage2-Analysis

**Fitting code:** https://gitlab.cern.ch/awesome-workshop/Payload-Stage2-Stats

**WARNING:** Please make sure you've forked the starter repos into your own namespace before cloning and making commits to them, otherwise you'll run into permissions issues when you try to push your commits!!

{% include links.md %}
