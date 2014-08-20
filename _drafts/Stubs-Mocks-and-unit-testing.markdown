---
layout: post
title:  "Stubs, Mocks and Unit Testing"
date:   2014-08-20
categories: testing
---

I have always been a fan of unit testing since the moment I was introduced to it.  I was shown how to use [Moq](https://github.com/Moq/moq4) and I never really looked back.  It was a couple years later that I was formally introduced to the various concepts and definitions of stubs and mocks.  There are many smarter people than I who give detailed differences and do comparisons but what I often need is a simple reminder of the definitions, so here they are:

 Test Double - defines any test object that is used 
 Fake
 Mock - Provides a way to do behavior verification.  
 Stub - Provides a predetermined value.  Usually used in state verification.
 Shims

[martin fowler](http://martinfowler.com/articles/mocksArentStubs.html)