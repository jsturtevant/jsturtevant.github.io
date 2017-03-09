---
layout: post
title: Running Jekyll in Windows Using Docker
date: "2016-03-04"
categories:
  - jekyll
  - docker
---

It has been just over a year since I wrote a post on [running Jekyll in Windows using Vagrant](/posts/running-jekyll-in-windows/).  The world has relentlessly moved on... we now have [Jekyll 3](http://jekyllrb.com/) and [Docker](http://www.docker.com/) has become popular to the point that you [might want to check it out](http://www.zdnet.com/article/what-is-docker-and-why-is-it-so-darn-popular/). So I revisited the way I was running my local version of my Jekyll blog.  The [previous way I outlined using Vargrant to run Jekyll on Windows](/posts/running-jekyll-in-windows/) has no issues but I needed to test my Jekyll blog against the [upgrade that GitHub Pages](https://github.com/blog/2100-github-pages-now-faster-and-simpler-with-jekyll-3-0) is doing and so I decided to also try running Jekyll in Docker just for fun.

There are a couple of different Docker containers that will work for testing your Jekyll site locally on Windows.  The first option is the [official Jekyll Docker container](https://github.com/jekyll/docker).  With this container you can get up and going with Jekyll by its self or  by using the provided pages container.  Since [GitHub Pages supports Jekyll](https://help.github.com/articles/about-github-pages-and-jekyll/) and has extra packages/components that are available when your Jekyll site is built for GitHub Pages, it is a good idea to use the pages container provided to be sure you are getting the closest match to the GitHub Pages environment.

The second option for Docker containers is one created by [Hans Kristian Flaatten](https://github.com/Starefossen).  [This container](https://github.com/Starefossen/docker-github-pages) uses the [github-pages gem](https://github.com/github/pages-gem) that is maintained by GitHub themselves.  Using this container will mimic the GitHub Pages enviroment and is very similar to the solution I created [using Vagrant to Run Jekyll on Windows](/posts/running-jekyll-in-windows/) because they both use the github-pages gem.  *This is definitely my preferred way of running my blog locally* because I know the environment will be closely mimicking the GitHub Pages environment.

## Setting up Jekyll using Jekyll Docker Container on Windows
1. [Install Docker](https://docs.docker.com/windows/)
2. Open the Docker Quickstart terminal
3. Clone or Create your Jekyll Project and move into the root folder of your project where you ```_config.yml``` is located.
4. Run the following command in the console: 

   ```
   $ docker run --rm --label=jekyll --volume=%CD%:/srv/jekyll  -it -p 4000:4000 jekyll/jekyll jekyll serve
   ```

5. Visit the local address to view your site! (typically ```192.168.99.100:4000``` if you are using Docker on Mac or Windows)

> note that the above command uses the defualt Jekyll docker container.  If you want to use the container desgined for GitHub Pages change the image name to ```jekyll/jekyll:pages```

## Setting up Jekyll using the GitHub Pages Container on Windows (recommended if you deploy on GitHub Pages)
1. [Install Docker](https://docs.docker.com/windows/)
2. Open the Docker Quickstart terminal
3. Clone or Create your Jekyll Project and move into the root folder of your project where you ```_config.yml``` is located.
4. Run the following command in the console: 

   ```
   $ docker run -v "$PWD":/usr/src/app -p "4000:4000" starefossen/github-pages
   ```

5. Visit the local address to view your site! (typically ```192.168.99.100:4000``` if you are using Docker on Mac or Windows)

## Considerations

### Project location for Docker on Windows
Where you create/clone your project on you machine matters other wise you may get permission errors or a note that your configuration file is not found.  I found [this issue](https://github.com/jekyll/docker/issues/91) on the Jekyll Docker repository where the project was in the wrong location.  On Windows and Mac the Virtual Machine that is created to run Docker mounts the user's home directory (```c:\users\foo```) by default.  If your Jekyll project does not live in that path then you maybe get an error similar to:

```
Configuration file: none
       Source: /srv/jekyll
       Destination: /srv/jekyll/_site
Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 0.017 seconds.
Auto-regeneration: enabled for '/srv/jekyll'
Configuration file: none
jekyll 3.0.2 | Error:  Permission denied @ dir_s_mkdir - /srv/jekyll/_site
ok: down: /etc/startup3.d/nginx: 1s, normally up
sh: can't kill pid 29: No such process
```

### Upgrading Jekyll
If you have an existing project that is upgrading to Jekyll 3.0 you may need to make a few updates to your Jekyll project ```_config.yml``` file to get the project to run properly using GitHub Pages Container.  See they details in [Jekyll's upgrade documentation](https://jekyllrb.com/docs/upgrading/2-to-3/).

## Conclusion
It wasn't strictly necessary to switch to using Docker to run Jekyll and my [Jekyll Vagrant box]((/posts/running-jekyll-in-windows/) ) will still work.  Choosing to explorer Docker as an alternative was for fun and a learning experience.  The world is moving to containerized solutions so it was a good experience running Jekyll in Docker and will make it easy to potentially deploy my blog to another location in the future.  As always, I would love to hear your feedback and let me know if you are successful (or not) using this or other solutions to run Jekyll on Windows.

