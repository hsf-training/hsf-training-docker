---
title: "File I/O with Containers"
teaching: 15
exercises: 5
questions:
- "How do containers interact with my local file system?"
objectives:
- "First learning objective. (FIXME)"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

# Copying

[Copying](https://docs.docker.com/engine/reference/commandline/cp/) files between the local host and Docker containers is possible. On your local host find a file that you want to transfer to the container and then

~~~
docker cp <file path> <CONTAINER ID>:/
~~~
{: .source}

and then from the container check and modify it in some way

~~~
echo "This was written inside Docker" >> example_file.txt
~~~
{: .source}

and then on the local host copy the file out of the container

~~~
docker cp <CONTAINER ID>:/example_file.txt .
~~~
{: .source}

and verify if you want that the file has been modified as you wanted

~~~
tail example_file.txt
~~~
{: .source}

# Volume mounting

What is more common and arguably more useful is to [mount volumes](https://docs.docker.com/storage/volumes/) to containers with the `-v` flag. This allows for direct access to the host file system inside of the container and for container processes to write directly to the host file system.

~~~
docker run -v <path on host>:<path in container> <image>
~~~
{: .source}

For example, to mount your current working directory on your local machine to the `data` directory in the example container

~~~
docker run --rm -it -v $PWD:/home/docker/data matthewfeickert/intro-to-docker
~~~
{: .source}

From inside the container you can `ls` to see the contents of your directory on your local machine

~~~
ls
~~~
{: .source}

and yet you are still inside the container

~~~
pwd
~~~
{: .source}

You can also see that any files created in this path in the container persist upon exit

~~~
touch created_inside.txt
exit
ls *.txt
~~~
{: .source}

This I/O allows for Docker images to be used for specific tasks that may be difficult to do with the tools or software installed on only the local host machine. For example, debugging problems with software that arise on cross-platform software, or even just having a specific version of software perform a task (e.g., using Python 2 when you don't want it on your machine, or using a specific release of [TeX Live](https://hub.docker.com/r/matthewfeickert/latex-docker/) when you aren't ready to update your system release).

# Running Jupyter from a Docker Container

You can run a Jupyter server from inside of your Docker container. First run a container while [exposing](https://docs.docker.com/engine/reference/run/#expose-incoming-ports) the container's internal port `8888` with the `-p` flag

~~~
docker run --rm -it -p 8888:8888 matthewfeickert/intro-to-docker /bin/bash
~~~
{: .source}

Then [start a Jupyter server](https://jupyter.readthedocs.io/en/latest/running.html#starting-the-notebook-server) with the server listening on all IPs

~~~
jupyter notebook --allow-root --no-browser --ip 0.0.0.0
~~~
{: .source}

though for your convince the example container has been configured with these default settings so you can just run

~~~
jupyter notebook
~~~
{: .source}

Finally, copy and paste the following with the generated token from the server as `<token>` into your web browser on your local host machine

~~~
http://localhost:8888/?token=<token>
~~~
{: .source}

You now have access to Jupyter running on your Docker container.