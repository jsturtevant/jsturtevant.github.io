---
layout: post
title: Using the Visual Studio Team Services Agent Docker Images
date: "2017-05-06"
categories:
  - docker
  - VSTS
---

There are two ways to create Visual Studio Team Services (VSTS) agents: Hosted and Private.  [Creating your own private agent](https://www.visualstudio.com/en-us/docs/build/concepts/agents/agents#install) for VSTS has some advantages such as being able to install the specific software you need for your builds.  Another advantage that becomes important, particularly when you start to build docker images, is [the ability to do incremental builds](https://www.visualstudio.com/en-us/docs/build/concepts/agents/hosted#capabilities-and-limitations). Incremental builds lets you keep the source files, and in the case of the docker the images, on the machine between builds. 

If you choose the [VSTS Hosted Linux machine](https://www.visualstudio.com/en-us/docs/build/concepts/agents/agents#hosted-agents), each time you kick off a new build you will have to re-download the docker images your to the agent because the agent used is destroyed after the build is complete.  Whereas, if you use your own private agent the machine is not destroyed, so the docker image layers will be cached and builds can be very fast and you will minimize your network traffic as well.

> Note: This article ended up a bit long; If you are simply looking for how to run VSTS agents using docker [jump to the end where there is one command to run](#agent-that-supports-using-containers-to-build-source).

## Installing VSTS agent
VSTS provides you with an agent that is meant to get you started building your own agent quickly.  You can download it from your [agents page in VSTS](https://www.visualstudio.com/en-us/docs/build/actions/agents/v2-linux#download-and-configure-the-agent).  After it is installed and you can customize the software available on the machine.  VSTS has done a good job at making this pretty straightforward but if you [read the documentation](https://www.visualstudio.com/en-us/docs/build/actions/agents/v2-linux#learn-about-agents) you can see there are quite a few steps you have to take to get it configured properly.

Since we using docker, why not use the docker container as our agent?  It turns out that VSTS provides a docker image just for this purpose. **And it is only one command to configure and run an agent.**  You can find the source and the documentation on the [Microsoft VSTS Docker hub account](https://hub.docker.com/r/microsoft/vsts-agent/).  

There are many different agent images to choose from depending on your scenario (TFS, standard tools, or docker) but the one we are going to use is the image with Docker installed on it.

## Using the Docker VSTS Agent
All the way [at the bottom of the page](https://hub.docker.com/r/microsoft/vsts-agent/) there is a description of how to get started with the agent images that support docker.  One thing to note that it *uses the host instance of Docker to provide the Docker support*:

```
# note this doesn't work if you are building source with docker 
# containers from inside the agent
docker run \
  -e VSTS_ACCOUNT=<name> \
  -e VSTS_TOKEN=<pat> \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -it microsoft/vsts-agent:ubuntu-16.04-docker-1.11.2
```

This works for some things but if you using docker containers to build your source (and you should be considering it... it is even going to be [supported soon through multi-stage docker builds](http://blog.alexellis.io/mutli-stage-docker-builds/)), you may end up with some errors like:

```
2017-04-14T01:38:00.2586250Z �[36munit-tests_1     |�[0m MSBUILD : error MSB1009: Project file does not exist.
```

or 

```
2017-04-14T01:38:03.0135350Z Service 'build-image' failed to build: lstat obj/Docker/publish/: no such file or directory
```

You get these errors because the source code is downloaded into the VSTS agent docker container.  When you run a docker container from inside the VSTS agent, the new container runs in the context of the host machine because the VSTS agent uses docker on the host machine which doesn't have your files (that's what the line ```-v /var/run/docker.sock:/var/run/docker.sock``` does).  If this hurts your brain, you in good company.

## Agent that supports using Containers to build source
Finally, we come to the part you are most interested in.  How do we actually set up one of these agents and avoid the above errors?  

There is an environment variable called ```VSTS_WORK``` that specifies where the work should be done by the agent.  We can change the location of the directory and volume mount it so that when the docker container runs on the host it will have access to the files.

To create an agent that is capable of using docker in this way:

```
docker run -e VSTS_ACCOUNT=<youraccountname>  \
  -e VSTS_TOKEN=<your-account-Private-access-token> \
  -e VSTS_WORK=/var/vsts -v /var/run/docker.sock:/var/run/docker.sock \
  -v /var/vsts:/var/vsts -d \ microsoft/vsts-agent:ubuntu-16.04-docker-17.03.0-ce-standard
```

The important command here is ```-e VSTS_WORK=/var/vsts``` which tells the agent to do all the work in the ```/var/vsts``` folder.  Then volume mounting the folder with ```-v /var/vsts:/var/vsts``` enables you to run docker containers inside the VSTS agent and still see all the files.

> Note: you should change the image name above to the one most recent or the version you need

## Conclusion
Using the VSTS Docker Agents enables you to create and register agents with minimal configuration and ease.  If you are using Docker containers inside the VSTS Docker Agent to build source files you will need to add the ```VSTS_WORK``` environment variable to get it to build properly.
