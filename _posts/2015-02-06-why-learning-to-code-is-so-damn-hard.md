---
layout: post

title: CodeCatatlyst promise-as3 Library
subtitle: "The tutorial I wish I'd had"
cover_image: blog-cover.jpg

excerpt: "A quick overview of using the CodeCatalyst promise-as3 library and all of the tips I wish I'd had when I picked it up."

author:
  name: Eric Kenney
  bio: Web Developer
  image: logo.png
---
##CodeCatalyst's promise-as3 Help I Wish I'd Had

For a project at work recently, we found the need to adapt a promise-driven Javascript file into AS3. After a little bit of research we settled on CodeCatalyst's promise-as3 library. I say little bit of research because there's really not very many libraries to look into. And the ones that do exist have little to no documentation. Because of that, I decided to put together things I've found useful for my own and others' benefit.

###Starting with promise-as3

One thing to note about promise-as3's implementation of promises, is that you need to instantiate a Deferred object from which you will pull a Promise object. You can not create a promise directly.

{% highlight as3 %}
var deferred : Deferred = new Deferred();
var promise : Promise = deferred.promise;
{% endhighlight %}
This is different from JS's built-in promises, which bypass deferreds entirely. The reason for this separation of power is so that you are able to return the promise object and allow the promise to be passed around while maintaining control of when and how the `Deferred` is resolved/rejected. Whatever function receives the promise can add `then` and `otherwise` callbacks/errbacks and you maintain control of deferred. Which leads me to the next topic.

###Adding fulfill and reject functions (callback and errback)

In the new JS implementation of promises, the promise constructor takes a function as a parameter. That function itself takes `fulfill` and `reject` functions as arguments. It would look something like this:

{% highlight javascript %}
function doSomethingAsync(url) {
  var promise = new Promise( function (fulfill, reject) {
      setTimeout(function () {
          reject( new Error('Timeout' ));
        }, 10000);
    }
}
{% endhighlight %}

Here, `doSomethingAsync` creates a new promise. On construction, the promise is passed the fulfill and reject functions (defined elsewhere).

This is not how it is done with the promise-as3 library. Instead, you instantiate a `Deferred`, passing no arguments. You extract the promise associated with the deferred. That promise can then be returned and made available to outside functions/classes (the consumer).

{% highlight as3 %}
public function doSomethingAsync(url) : Promise {
  var deferred : Deferred = new Deferred();
  var promise : Promise = deferred.promise;

  setTimeout(function () {
            deferred.reject("Of course it timed out");
        }, 10000);

    return promise;
}
{% endhighlight %}

At that point, it is up to the consumer to add callbacks and errbacks.

{% highlight as3 %}
var myPromise = doSomethingAsync("www.url.com");

myPromise.then(callback, errback); // traces "Of course it timed out"

public function callback() : void {
  trace("That promise must have worked!")
}

public function errback(reason) : void {
  trace(reason)
}
{% endhighlight %}

In this example, the errback will always be called because `doSomethingAsync` just waits 10 seconds and then calls `deferred.reject()`.

I found the CodeCatalyst promise-as3 documentation a little light, especially with regards to extracting the promise from the deferred, yet still calling resolve/reject on the deferred object. The magic that passes this on to the associated promise is the `Resolver` built into the promise-as3 library. As the GitHub page notes...

  Each Deferred has an associated Resolver, and each Resolver has an associated Promise. A Deferred delegates resolve() and reject() calls to its Resolver's resolve() and reject() methods. A Promise delegates then() calls to its Resolver's then() method. In this way, access to Resolver operations are divided between producer (Deferred) and consumer (Promise) roles.

