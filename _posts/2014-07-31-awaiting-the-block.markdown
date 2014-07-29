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
 Take a look at the code below from a simple WinForms app.  
