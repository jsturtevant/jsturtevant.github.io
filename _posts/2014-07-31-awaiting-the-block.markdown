---
layout: post
title:  "Awaiting the Block"
date:   2014-07-31
categories: csharp
---

The **await** keyword in C# is magical.  It seems to just work.  Throw an **await** in the front of the offending code
 and slap an **async** on the method.  Do this all the way up and it just works. Right?  Although it does *just* work 
 alot of the times there are quite a few gotcha's.  This guy talks about this one and 
 that guy talks about that one. 
 
 I recently watch an awesome pluralsight video by Skeet that went into depth on how **await** works.  I recommend the 
 video as it goes into great depth on how async works under the covers, which is out of scope for this article.  
 Here I am going to cover a topic that I had to play with a bit more to understand.
 
## Blocked
 Take a look at the code below from a click even on a simple WinForms app created for testing.  What do you think will 
 happen to the responsiveness of the form?
 
 {% highlight csharp %}
  private void noAwaitClick(object sender, EventArgs e)
 {
     label1.Text = "starting...";
     long prime = this.FindPrimeNumber(PRIME_TO_FIND);
     label1.Text = prime.ToString();
 }
 {% endhighlight %}

If you guessed that the form will freeze up until the prime number is calculated then you are correct.  Using the 
well known [methods](http://stackoverflow.com/questions/709187/accessing-ui-in-a-thread) to spin off worker threads 
and then calling invoke on the UI thread is one way to do it.  This is a bit tedious and difficult for beginners to 
understand but now with **async** and **await** we have a new, simpler way to stay responsive.

## Async to the Rescue
Let's take a look at the code block below.  What do you think will happen to the responsiveness of the form this time?

{% highlight csharp %}
private async void async1Click(object sender, EventArgs e)
{
    label1.Text = "starting...";

    var prime=  await this.primenumberAsync();

    label1.Text = prime.ToString();
}
{% endhighlight %

