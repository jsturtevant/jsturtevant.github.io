---
layout: post
title: Azure Stream Analytics not processing data from Event Hubs to storage
date: '2015-06-22'
categories:
  - iot
  - woodstove
  - azure
---

The issue I ran into with  [Azure Stream Analytics](http://azure.microsoft.com/en-us/services/stream-analytics/) is a little bit embarrassing but I think we have all been [there before](/posts/the-amazing-fifteen-minute-break/).  You know the story...  it is late, you have that due date tomorrow and you can't figure out why it is not working.  At least not until you [start to explain it to someone else](http://blog.codinghorror.com/rubber-duck-problem-solving/).  


As I was doing the initial set up for Stream Analytics for my [woodstove project](/posts/wood-stove-project-introduction/) I found that I was able to send data to [Event Hubs](http://azure.microsoft.com/en-us/services/event-hubs/) but it was not being processed by the Stream Analytics job.  A couple hours later, after checking to make sure all my code was correct and the [Stream Analytics tests passed](http://blogs.msdn.com/b/streamanalytics/archive/2014/11/12/update-to-stream-analytics.aspx), I found the issue: I did not start the Stream Analytics Job.  All that was need was to press the start button at the bottom of the job:
![Start Stream Analytics Job]({{ site.url }}/assets/stream-anayltics-start-job.PNG)

I know it happens to all of us but hopefully this will short circuit your debugging process and get you on to developing awesome applications with [Azure Stream Analytics](http://azure.microsoft.com/en-us/services/stream-analytics/).
