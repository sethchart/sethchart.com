---
title: "Index"
date: 2021-03-07T11:37:13-05:00
slug: ""
description: "How to use Use Jupyter Notebook in a Docker Container."
keywords: ["Docker", "Jupyter"]
draft: true
tags: ["Docker", "Jupyter"]
math: false
toc: false
---

## What You Will Learn

In this post I will show you how to set up a Jupyter notebook server in a Docker container.
There are three major benefits that you can expect from this approach.

1. Setting up a new environment is fast!
2. Resetting your environment is easy!
3. Tools that you install while working on your project are separated from you computer so cleaning up after a project is easy.

## Prerequisites

To follow along with this blog post you will need to install Docker on your computer.
Here are the official instructions about how to [get Docker](https://docs.docker.com/get-docker/).
We are going to use the command line to set up our container, so open up your terminal.
To verify that you have docker installed properly, type the command below into your terminal and hit enter.
```
docker --version
```

You should get output that looks something like the line below.

```
Docker version 20.10.3, build 48d30b5
```

## Selecting an Image

Once you have Docker installed on your computer, you will need to decide which Jupyter image will fit your needs best.
Check out the guide to [selecting an image](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html) for more information.
I am going to use the `jupyter/scipy-notebook` image for this post.

## Pulling the Image

A docker image is a file with all of the information that you need to create a Docker container.
Before we can set up our container, we need to download the image that we chose in the Selecting an Image section.
The `docker pull` command is used to download Docker images.
The line below will download the latest version of the `jupyter/scipy-notebook` image from [Docker Hub](https://hub.docker.com/r/jupyter/scipy-notebook/tags/?page=1&ordering=last_updated).

```
docker pull jupyter/scipy-notebook
```

## Creating the Container


The command below will create a Docker container that will serve a Jupyter notebook server on your computer.
Note that you do not need to include the slashes and hard returns, I have done this to improve readability.
That said, if you copy the command below into your terminal it will run.
Docker commands can get pretty long.

```
docker create \
--name notebook-server \
-p `8888`:`8888` \
-v ${PWD}:/home/jovyan/work \
jupyter/scipy-notebook
```

Let’s break down the parts of this command so we understand what we are doing here.
The main command is `docker create` which unsurprisingly is used to create a docker container.
In the sections below we will investigate each of the options that we are using with the `docker create` command.

### Naming the Container

If you leave the line below out, docker will assign a random name to your container for you, but if you want to remember the name of your container and what it is for, I would recommend naming it yourself by adding the `--name <your container name>` option to your `docker create` command.
As you can see, I used the option below to name my container notebook-server.

```
--name notebook-server

### Mapping the Notebook Server Port

Whenever you connect to a Jupyter notebook server that is running on your computer, you connect to your computers [localhost](https://en.wikipedia.org/wiki/Localhost) through [port](https://en.wikipedia.org/wiki/Port_(computer_networking)) `8888`.

If not, you probably already know what we are doing here, so bear with me.

By default, Jupyter is going to try to communicate on port `8888`.
Inside the container, Jupyter will serve notebooks on port `8888`, but in order to access the notebook using the browser on our computer, we need to map the container port to one of our computer’s localhost ports.

The easy thing to do is just map the container’s port `8888` to our computers localhost port `8888` which we can do by passing the option below to the `docker create` command.

```
-p 8888:8888

```

If we are feeling fancy, we can chose a different port on our computer’s localhost.
The port mapping option used the `host:container` convention, so if we wanted to access the container on port `8889`, then we would use `-p 8889:8888`.
Remapping the default port is only necessary if you need to have multiple services running, so further discussion is beyond the scope of this post.

### Mounting a Local Folder

If you want to be able to access files that you have saved on your computer from your notebook, you will need to mount that folder to a folder inside the container.
To do this we pass the option below to the docker create command.

```
-v ${PWD}:/home/jovyan/work
```

This folder mapping also follows the convention `host:container`.

The string `${PWD}` tells your computer to print the working directory.
That means that whatever folder you are currently in when you run the `docker create` command gets connected to your docker container.

Alternatively, you could just type out the path to the folder that you want to mount in place of `${PWD}`.
Either way, we only get one shot at this.
The folder that we mount to our container is the one and only folder that is mounted for the life of the container.
If we want to access a different folder on our computer, we will need to create a new container.

### Selecting the Image

Finally, we need to tell `docker create` which image to use for our container, so we end our command with the name of the image that we pulled.

```
jupyter/scipy-notebook
```

### Starting the Container


Once we have created the container, we still need to start it with the `docker start` command.
We need to tell `docker start` the name of the container that we want to start, so to start the container we just created we use the command below.

```
docker start notebook-server
```


