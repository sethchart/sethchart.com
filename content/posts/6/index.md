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
