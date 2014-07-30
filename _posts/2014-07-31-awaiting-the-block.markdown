---
layout: post
title:  "Awaiting the Block"
date:   2014-07-31
categories: csharp
---

The **await** keyword in C# is magical.  It seems to just work.  Throw an **await** in the front of the offending code
 and slap an **async** on the method.  Do this on all the methods in the chaing and it just works. Right?  Although it 
 does *just* work most of the times there are quite a few *gotcha's*.  This guy talks about this one and 
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
{% endhighlight %}

Did you guess that it would be responsive?  Good guess, but this was a trick question and the *gotcha* I would like 
to share with you.  To understand why this is not going to keep the UI responsive we have see the implementation of 
the ```primenumberAsync()``` and have basic understanding of what happening when 
the code is compiled.  

Let's see what is happening in the implementation of ```primenumberAsync()```:

{% highlight csharp %}
private async Task<long> primenumberAsync()
{
    long prime = this.FindPrimeNumber(PRIME_TO_FIND);

    return await Task.FromResult(prime);
}
{% endhighlight %}

We are simply calling ```FindPrimeNumber```  and creating a ```Task``` from the result.  Now if you are like me, 
you might be saying "Doesn't **await** and **async** create new threads when ```primenumberAsync()``` is called?  

## Under the Covers
To understand why the code above blocks we have to have a understand of how **await** and **async** gets compiled.
The key words **await** and **async** actually get compiled into a [state machine](http://en.wikipedia
.org/wiki/Finite-state_machine).  The compiled state machine is how the asynchronous behavior is achieved.  I would 
recommend watching the [pluralsight] video or reading [this](http://www.codeproject
.com/Articles/535635/Async-Await-and-the-Generated-StateMachine)
article to get a better understanding of the generated state machine. 

The key take away here is that the **await** and **async** keywords do not create threads.  This is important to 
consider because now we can understand why the code in ```primenumberAsync()``` causes the UI to block.  Since the 
work is not being done on a separate thread and requires the use of the raw computing power the UI thread gets 
blocked.  You can see this being executed on the main thread if you download the sample code from the github repository 
and set a break point  inside of the ```FindPrimeNumber``` routine.  

## Unblocked
To create a responsive UI we can leave our call in the button handler the same but re-implement ```primenumberAsync()```
to create a new thread:

{% highlight csharp %}
private async Task<long> primenumberAsync2()
{
    var prime = await Task.Run(() => this.FindPrimeNumber(PRIME_TO_FIND));

    return prime;
}
{% endhighlight %}

## Conclusion
**Await** and **async** are power additions to the C# language but with great power comes great responsibility.  In 
this post we took a look at one scenario where **await** and **async** are not magic bullets and a little bit of care
 needs to be taken to make sure that the behavior we are expecting at runtime is properly implemented.
 
 
 