---
layout: post
title: Throttling Requests in Node.js
comments: true
---

One of the most common requests I've seen on my StackOverflow is a method for avoiding rate limits. If you're working with any kind of API or wrapper you are going to have rate limits. These can be nasty to deal with. The majority of API's make their rate limits public, most of them at least—some limits may be hidden. An example of this can be seen [here](https://discord.com/developers/docs/topics/rate-limits) in the Discord API. Wouldn't it be nice if we could just wait between each request? That is the basic idea of what my StackOverflow followers wanted. Now I am not one to spoon feed but even I had to take a bit of time to come up with a viable solution. 

You may have heard of or even dealt with queue systems in Javascript. These usually abuse an array to track what tasks have been completed and what still needs to be done. You have probably already heard of a forEach loop as well. This is a simple method that loops through each item in an array and runs a provided function. I have combined these two methods to create what I call the forEaach method—because it is longer ;-;. The idea is very simple, instead of running the function on each array item as fast as possible, the function delays each function. Here is a nice and pretty version of the function.
```js
async function forEaach(arr, func, delay) {
	var events = require('events');
	var Emitter = new events.EventEmitter();
	var array = [...arr];

	let int = setInterval(async() => {
		if (!array.length || !array.length === 0) return Emitter.emit("complete");

		await func(array[0], arr.indexOf(array[0]));

		array.shift();
		
        if (!array.length || !array.length === 0) return Emitter.emit("complete");
	
    }, delay)

	Emitter.on("complete", () => {
		clearInterval(int);
		return;
	})
}
```
This is a little different from what you might have expected. I used event emitters to register when it is complete, this method may seem unorthodox but it is actually quite effective. This allows you to await the completion of the loop and stop the loop from continuing forever. The function works the same as `Array.forEach` except it includes a delay. It is also important to note that because I am not changing the Array class, the function is run on its own. Here is an example of using the function.
```js
var array = ["1", "2", "3"]; // An array of strings

forEaach(array, function (string, i) { // Will wait 5 seconds after each string is logged.
console.log(string);
}, 5000);
```
I think this example explains how to use the function pretty well. You might ask, how is this useful for rate limits? In most cases, the requests you'll need to throttle are all to the same endpoint. If you are working directly with an API using something like [node-fetch](https://www.npmjs.com/package/node-fetch) you can load an array with each URL you need to request. If there is a strict rate limit of 10 milliseconds between each request on the same URL you supply an array whose length matches the number of requests you want to make. If you want to make a request indefinitely you can use a simple interval instead.

There are many other applications for this function, I would love to see what you all do with it. Please [contact me by email](mailto:obs@obs.wtf) if you have any issues.


> OBS - Chief Executive Officer