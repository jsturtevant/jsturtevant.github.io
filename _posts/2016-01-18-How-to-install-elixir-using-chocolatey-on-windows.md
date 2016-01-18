---
layout: post
title: 'How to install Elixir using Chocolatey on Windows'
date: '2016-01-18'
categories:
  - elixir
  - chocolatey
---

Installing [Elixir](http://elixir-lang.org/) is very easy using [Chocolatey](https://chocolatey.org/) for Windows.  If you haven't heard of Chocolately before, it is the missing package manager for Windows (similar to [apt-get](https://en.wikipedia.org/wiki/Advanced_Packaging_Tool) or [homebrew](http://brew.sh/)).  When combined with [Boxstarter](http://www.boxstarter.org/), it can be used to [automatically set up your development environment](/posts/Chocolatey-And-Boxstarter/).

## Installing Elixir
To install Elixir using Chocolaty:

1. Make sure you have (Chocolaty installed)[https://chocolatey.org/]. 
2. Open an administrative ```cmd``` prompt
3. Type ```choco install elixir``` (add ```-y``` to  skip install prompts)
4. Restart your ```cmd``` prompt.

That is it! The last command will not only install the [latest version of Elixir](https://chocolatey.org/packages/Elixir) but it will install [Erlang](http://www.erlang.org/) which is a [prerequisite for Elixir](http://elixir-lang.org/install.html#installing-erlang).  It will also add Elixir to your PATH (if not you can find the commands at C:\ProgramData\chocolatey\lib\Elixir\bin and manually add it) so you can open up a command prompt and type ```iex``` to start [Elixir's Interactive Mode](http://elixir-lang.org/getting-started/introduction.html#interactive-mode).  

Have fun [getting started with Elixir](http://elixir-lang.org/getting-started/introduction.html)!