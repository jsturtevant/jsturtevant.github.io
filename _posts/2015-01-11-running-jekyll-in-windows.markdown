---
layout: post
title: Running Jekyll in Windows
date: "2015-01-11 13:28"
categories:
  - jekyll
---

<p class="message">I have an updated post <a href="/posts/Running-Jekyll-in-Windows-using-Docker">using Docker to run Jekyll in Windows</a>.  The method described below is still valid but you know have more choices.
</p>

Setting up [Jekyll](http://jekyllrb.com/) for Windows is painful.  To set it up you need to follow a multiple step setup process and cross your fingers.  [Julian Thilo](https://twitter.com/juthilo) has done a great job outlining all the steps and pitfalls on his [Run Jekyll on Windows](http://jekyll-windows.juthilo.com/) site.  This is how I initially setup Jekyll but the trouble with this process is it takes several steps and potentially requires a bit of debugging to figure out.

I recently switch development machines, which  meant I needed to setup Jekyll again and didn't want to repeat the error prone process.  At the same time I happened to learn [how use Vagrant](http://sciencevikinglabs.com/science/vagrant/2014/12/21/vagrant-getting-started.html) at the [user group](http://augusta-polyglot.github.io/) I organize.  

Combining the two ideas I created a [Vagrant Configuration for Jekyll](https://github.com/jsturtevant/jekyll-vagrant).  Now when I switch machines I have the same automated setup.  I can be up and running in a few minutes and it is the [same environment](https://github.com/github/pages-gem) that is used for [GitHub Pages](https://pages.github.com/).

## Setup
1. Make sure [Vagrant](https://www.vagrantup.com/) and your favorite virtual machine are installed. I use [Virtual Box](https://www.virtualbox.org/) and when setting up a new machine I already have these dependencies installed using [Chocolately and BoxStarter][8a792ea8].
2. Clone the [jekyll-vagrant](https://github.com/jsturtevant/jekyll-vagrant) repository

    ```git clone https://github.com/jsturtevant/jekyll-vagrant.git```
3. Open command prompt to location of the ```vagrantfile``` in the new cloned repository and run ```vagrant up```
4. Jekyll and all it's dependencies are installed!

## Existing Jekyll Projects
1. Copy the projects folder to the folder that contains the ```vagrantfile```.  
2. Login to the VM using ```vagrant ssh```

## New Jekyll Projects
1.  Open a command prompt to location of the ```vagrantfile``` and run ```vagrant ssh```
2.  Once in the VM prompt ```cd /vagrant```
3.  Create a new site with ```jekyll new <sitename>```

## Start the Site
1. In the VM prompt ```cd /vagrant/<YourProjectFolder>```
2. Start the Jekyll server ```jekyll serve --force_polling``` ([force polling is required](http://stackoverflow.com/a/23084706/697126) with vagrant because of share)
3. On your host machine you can open any browser and navigate to ```localhost:4000```
4. Work on you project on your host machine and see updates as they happen on ```localhost:4000```!

## Conclusion
[Jekyll](http://jekyllrb.com/) is a fun blogging platform but the support for Windows is limited.  There are ways to install Jekyll directly on the host machine but they are tedious and error prone.  By leveraging [Vagrant](https://www.vagrantup.com/) we can have a fast, repeatable Jekyll  setup that is in sync with [GitHub Pages](https://pages.github.com/) environment.


[8a792ea8]: http://www.aspenrootsdevelopment.com/posts/Chocolatey-And-Boxstarter "Chocolatey and Boxstarter"
