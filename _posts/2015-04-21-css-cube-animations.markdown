---
layout: post
title: CSS Cube Animations
date: "2015-04-21"
categories:
  - css
  - fun
---

Last year while studying for my [MCSD certification](https://www.microsoft.com/learning/en-us/mcsd-web-apps-certification.aspx) I came across a [comprehensive post on CSS animation](http://www.the-art-of-web.com/css/3d-transforms/).  Following along with the post, I created some spinning dice with the idea of turning the dice into some sort of game.  I never finished building the game but spinning dice are [fun enough to share](https://github.com/jsturtevant/jsturtevant.github.io/blob/master/_includes/dice.html):  

{% include dice.html %}

**Note:** If you are on IE 10/11 you will only see one side.  This is because IE [does not support]( http://caniuse.com/#feat=transforms3d) the ```preserve-3d property```.  But it works in the new [Edge](http://www.microsoft.com/en-us/windows/microsoft-edge) browser!
