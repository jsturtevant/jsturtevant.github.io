---
layout: post
title:  "AngularJS HTTP Service Success Handler 400 Response Code and Interceptors"
date:   2014-11-12
categories: AngularJS
---

I ran into a perplexing problem the other day while working with AngularJS.  All of my [400](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) (400, 401, 403, 404...) responses from my back end ASP.NET WebApi site were going to the success handler of of the [$http](https://docs.angularjs.org/api/ng/service/$http) service:

{% highlight javascript %}
$http.get('/api/resource').
  success(function(data, status, headers, config) {
    // 400/500 errors show up here
	if (status == 400)
	{
		console.log('Should never happen')
	}
  }).
  error(function(data, status, headers, config) {
    // never reached even for 400/500 status codes
  });
{% endhighlight %}

Luckily, the AngularJS team is tracking all of their [issues](https://github.com/angular/angular.js/issues) in the open and with a quick Google/Bing search I found an [issue](https://github.com/angular/angular.js/issues/2609) that pointed me in the correct direction.

If you take the time to read through the issue you find that a improperly implemented interceptor is likely the culprit.  And in my case this was indeed what was causing the strange behavior.  

When you handle the ```requestError``` or the ```responseError``` for a interceptor and you wish to pass the error on to the next handler you must use the [promise api](https://docs.angularjs.org/api/ng/service/$q) to reject the message:

{% highlight javascript %}
$httpProvider.interceptors.push(function($q) {
  return {
   'requestError': function(rejection) {
        // handle same as below
    },

   'responseError': function(rejection) {
      if (canRecover(rejection)) {
		 // if you can recover then don't call q.reject()
         // will go to success handlers
         return responseOrNewPromise;
      }

	  // !!Important Must use promise api's q.reject()
	  // to properly implement this interceptor
	  // or the response will go the success handler of the caller
      return $q.reject(rejection);
    }
  };
});
{% endhighlight %}

In my case the return ```$q.reject(rejection)``` was missing causing the message to go to the success handler.  The documentation does not make this explicitly clear.  If you are new to angular or not familiar with how the [promise api](https://docs.angularjs.org/api/ng/service/$q) works it is easily left off.  This is a quick fix but can be quite confusing when first encountered.
