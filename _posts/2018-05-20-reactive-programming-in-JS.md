---
tags: [programming]
---
## What is Reactive Programming

Reactive Programming is a programming paradigm for asynchronous propagation of change - it's often described as "event driven programming". Instead of "execute function B after function A" or "execute function A after 5 seconds" you declare "execute function A whenever X happens". This is already common practice in almost every UI framework where you write event handler. You attach a function to a "button clicked" event for example. But reactive programming takes it even further by making it possible to react to any data like you would to an event. And with a much more advanced API to manipulate and combine these streams of events.

A great embodiment of reactive programming is ReactiveX - a language agnostic specification for reactive programming libraries. We will focus mainly on ReactiveX and RxJS (the JavaScript implementation of ReactiveX) but will also show you a bit of Socket.io.

*“ReactiveX is a library for composing asynchronous and event-based programs by using observable sequences.”* [ReactiveX](http://reactivex.io/intro.html)

What does that mean? Let's compare it with other stuff we already know

| type of execution   |  single items | multiple items
|------ |  --------  |  ----------
| synchronous | ``T getData() ``| ``Iterable<T> getData()``
| asynchronous |`` Future<T> getData() ``|`` Observable<T> getData()``  


Working synchronously is pretty standard. You call `getData()` and get your data ``T``. Or if it's multiple items you get an `Iterable<T>` like a list or an array.  
When you work asynchronously you might get something like a `Future<T>` or `Promise<T>`. There you can register a callback, for example `myPromise.then(myCallbackFunction)` - that's a typical way of handling for example HTTP calls. The call might take a while, so we do it asynchronously and when it's done we want to execute a function with the returned value of the HTTP call.  
Reactive programming fills the last slot in this matrix - multiple items that are returned asynchronously (so not all items will be returned at the same time). The type for this is an `Observable<T>`. This is basically a `Promise<T>` that can be fulfilled multiple times. This is great for either a continuous stream of data (like a sensor, sending it's readings every couple of seconds) or updated versions of the same data (like changing the address of a customer object and sending the update as an event). 

The 2 most common elements of ReactiveX are observables and operators. These 2 are enough to start creating reactive systems - no need for any in-depth knowledge about things like the scheduler (for now).

**Observable**

An *observer* subscribes to an *observable* so whenever the *observable* emits a new item, the observer can react on that.
The following image by [ReactiveX](http://reactivex.io) shows the flow of items emitted by an observable and processed into a new observable.
![Observables](http://reactivex.io/assets/operators/legend.png)

In short: an observable is a source of items which can be subscribed to. You can subscribe with 3 types of callback functions 
- ``onNext`` get's called with the next item
- ``onError`` get's called when an error occurred  
- ``onCompleted`` get's called when the stream ends

**Operators**

Operators are functions that can be called on an observable. They transform, combine or otherwise operate with the items and usually return a new observable. 
The most common operators are probably ``filter`` and ``map`` - which behave pretty much like the ``filter`` and ``map`` operator from synchronous functional programming. They return new ``Observables`` without any side effects. And since most of them return new ``Observables`` they can be chained.



## Reactive Programming in JavaScript

### Double Clicks

So what does reactive Programming look like in JavaScript? Let's start with something common: double clicks. Lot's of applications use them. How would you do it with traditional programming? Saving the time of every click and when another click occurs compare the current time to the previous time? But what if you want to react to single clicks and double clicks. After every single click you would need to wait a few milliseconds to see if another click has occurred. It's obviously possible but it's going to be some ugly code. With streams this can be done with a few simple and easy to read operations.

 Every browser offers event handlers for button clicks. We can create a stream of click events from that button with only 1 line (and the RxJS library). At first we would want to group clicks by proximity in time. We want to group all clicks that are within 250ms of each other. Then map this list to the number of clicks in the list. So a single click within our specified 250ms will becomes a 1, double clicks a 2, triple click a 3 and so on. Then simply filter the stream to only return values equal to or greater than 2 and you have a stream that only contains events where the user clicks more than once in quick succession. This is what the streams for our double click detection would look like:
 
![DoubleClickStream](/static/img/double_click_stream.png)

And this is how the code looks

```javascript
const button = document.getElementById('multiclickButton');

const clickStream = Rx.Observable.fromEvent(button, 'click');

const multiClicks = clickStream
  .buffer(clickStream.debounce(250))
  .map(x => x.length)
  .filter(x => x>=2);

multiClicks.subscribe(clicks => console.log('You made ${clicks} clicks!'))
```

Pretty neat, right?   
First we get a reference to the button and then use RxJS to create a Observable for the click events.   
Then the magic happens in just 3 lines. Buffer, map and filter.  
For different buffer strategies check out [the official documentation](http://reactivex.io/documentation/operators/buffer.html)

### Autocomplete

Let's look at another example that is perfect for reactive programming: Autocomplete functionality. You type and you get suggestions in real time about what you might be typing. Let's start of naively and work out the errors until we arrive at a working solution. We want to do something like this:

````javascript
Rx.Observable.fromEvent(myInputField, 'input')
             .map(searchWikipedia)
             .subscribe(renderData, renderError);
  
function searchWikipedia(searchTerm) { /* ... */} // returns a promise
function renderData(data) { /* ... */}
function renderError(error) { /* ... */}
````

We get a stream from the users input in our ``myInputField``, make a call to the wikipedia server and then render the results beneath the input field.

Why won't this work? Well first of all the ``fromEvent`` functions creates a stream of ``events``. We need the input as a string, so we add a 

````javascript
.map(event => event.target.value)
````

Now the ``searchWikipedia`` function receives proper string input. But there will probably be an error in our ``renderData`` function now. We would expect a list of suggestions or something similar, but what we end up with is a ``Promise``! Why is that? ``searchWikipedia`` makes an HTTP call and returns a promise synchronously. So our ``.map(searchWikipedia)`` function will create a new Observable that emits a ``Promises``. That's not what we want! We want a stream that emits the results of the HTTP call. That's what the ``flatMapLatest`` (see [docs](http://reactivex.io/documentation/operators/flatmap.html)) is for. This will "unpack" our promises and create a stream that only emits the values once the promises have been fulfilled.

So now we have this:

````javascript
Rx.Observable.fromEvent(myInputField, 'input')
             .map(event => event.target.value)
             .flatMapLatest(searchWikipedia)
             .subscribe(renderData, renderError);
  
function searchWikipedia(searchTerm) { /* ... */} // returns a promise
function renderData(data) { /* ... */}
function renderError(error) { /* ... */}
````

This works but we're not done yet - this can be improved. We should only suggest something to the user if there is at least some input. A search of 1 or 2 characters is pretty pointless since the user could be writing anything when all we have is the input "A". So let's ``filter`` out these kinds of things.  
And if a user types really really fast we will send A LOT of requests. Basically every keystroke creates a new requests. It's probably enough to only check ever half second or so. This way our system is still suggesting in "real time" but without going overboard on HTTP requests. This can easily accomplished by throttling the stream.

```javascript
.filter(input => input.length > 2)
.throttle(500)
````

This will leave us with this:

````javascript
Rx.Observable.fromEvent(myInputField, 'input')
             .map(event => event.target.value)
             .filter(input => input.length > 2)
             .throttle(500)
             .flatMapLatest(searchWikipedia)
             .subscribe(renderData, renderError);
  
function searchWikipedia(searchTerm) { /* ... */} // returns a promise
function renderData(data) { /* ... */}
function renderError(error) { /* ... */}
````

This code will query the wikipedia server every time the user enters something with 3 or more characters (but not send more than 1 requests per .5 seconds) and show the suggestions to the user. All that's left to do is clean up. Instead of having one long chain of inputs we can break it apart to make it more obvious what the individual steps are. 

````javascript
const rawInput = Rx.Observable.fromEvent(myInputField, 'input')
                            .map(e => e.target.value);
  
const newSearchTerm = rawInput.filter(text => text.length > 2)
                              .throttle(500);
  
newSearchTerm.flatMapLatest(searchWikipedia)
             .subscribe(renderData, renderError);
  
function searchWikipedia(searchTerm) { /* ... */} // returns a promise
function renderData(data) { /* ... */}
function renderError(error) { /* ... */}
````

### Observable as core data structure

In case you're not familiar with UI development, let me tell you something: Keeping everybody up to date can be a huge problem. What's the issue?   
Let's say your application has a proper dependency structure and good data flow. The services on top get data from servers. The UI components down below get services injected and get all of their data from them. The services don't know the UI components and the UI components don't know each other.   
You have a ``TweetDisplayer`` component. It gets a reference to the ``TweetService`` via your dependency injection and then calls the ``getNewTweets`` function of the service to fetch the newest tweets. If you press the "update" button in the ``TweetDisplayer`` it will call ``getNewTweets`` again and receive a promise. When the promise is fulfilled it will display the new tweets - this works fine.  
But what if there is another component now: The ``CategoryPicker``. A UI component that let's you decided you only want to see tweets about "JavaScript", "Cats" or some other topic. The ``TweetDisplayer`` and the ``CategoryPicker`` don't know each other and they shouldn't. So what happens when the ``CategoryPicker`` tells the ``TweetService`` to ``getNewTweets``? It will receive a promise and once that promise is fulfilled the tweets are here. Here in the ``CategoryPicker``. The ``TweetDisplayer`` was never involved and has no idea that the ``CategoryPicker`` chose a category or that the ``TweetService`` fetched new tweets! So what do you do?  
 **Use observables!**  
 Instead of returning one promise to the one component that called for a tweet update your ``TweetService`` will now offer a ``getTweetObservable`` function. This will return the observable that will deliver our tweets to the UI components. And unlike previously every UI component will receive the same observable! What happens when you want the service to get new tweets? First of all, the ``getNewTweets`` function will no longer return a promise. In fact it doesn't need to do anything! Why? Because everybody that wants the newest tweets is already subscribed to the tweet observable. When the service receives new tweets it will simply publish them via the observable and everybody who subscribed will be updated. It doesn't matter who, when or why called for new tweets, the observable delivers the newest, freshest and hottest tweets to every component in your application!
 
 This is especially useful if you need to display the same data in multiple locations. As long as every component is subscribed to the same observable they will always have the same data!
 
 There's a minimalistic example in the [plunker](http://embed.plnkr.co/4GeB5mRIIskWgqj75QPq/) (`3_1-TweetUI.js` & `3_1-TweetService.js`)

### Socket.io
Lastly I want to show you something other than RxJS because while it is a great library it's not the only way to program reactively.  
So far all of our reactions and events where user based. Button clicks, text input and so on. But what about the other side - the server.  
There many use cases where a client would want to react to events from a server. The first thing that springs to my mind is a chat room.  
While REST interfaces are really good for a lot of applications it's not useful for a real time chat. You don't want to ``GET /chatroom/messages`` every seconds and see if there are new messages. Wouldn't it be nice if the server could just tell us when there are new messages?  
Well guess what - the server CAN do that!
It's called websockets and I'm not going into the technical details. Let's just say it's like an open channel between two applications where both can send messages through the channel. The perfect setup for reactive programming!  
We can easily use this technology with the [Socket.io](https://socket.io/) library.  
Note: To run this example at home you need Node.js and NPM but you can understand the code without it.  
We need some libraries to set up a working web server (that's what the first three and very last line do) and then we're ready to programm reactively. 
 
 ````javascript
 const app = require('express')();
 const http = require('http').Server(app);
 const io = require('socket.io')(http);
   
 // Our code will go here
   
 http.listen(3000)
````
 
 We will be handling our application ``io`` which contains all current connections and we will work with ``sockets`` which are the single connections. They offer us ``on`` and ``emit`` functions and work like this:  
````javascript
myConnection.on(
    // The name of the event - some (like "connection") are predefined, but otherwise these are our own names 
    'my event', 
    // a function that receives the data of the event
    (data) => {
        // do something with the data
    });
````


````javascript
myConnection.emit(
    // The name of the event
    'my event',
    // The data for the event - this will be (de-)serialized with JSON so make sure that's ok for your data
    someData)
````
 
 
 Let's make a chat server! Start by installing the dependencies  (``npm i --save express socket.io``) and setup ``yourFile.js`` with the stub from above. Then we implement the predefined "connection" event that will pass a socket into our fuction. In there we can declare all kinds of reactions to events. Like the 'chat message' event. When the socket sends us a "chat message" event, we simply emit it to every current socket (because we used ``io.emit`` - had we wrote ``socket.emit`` we would send the message only back to the sender. Remember: The socket is a single connection to a single other application).

````javascript
const app = require('express')();
const http = require('http').Server(app);
const io = require('socket.io')(http);
  
io.on('connection', function (socket) {
  
    socket.on('chat message', function (msg) {
        io.emit('chat message', message);
    });
  
});
  
http.listen(3000);
````

Now start the file with node (``node yourFile.js`` in your console of choice) and the server is up and running!

And our client could look something like this:

```javascript
const socket = io.connect(
    'http://localhost:3000', 
    { transports: ['websocket']}
    );
  
socket.on('chat message', renderMessage);
  
function renderMessage(message) { /* ... */}
```

We connect to the localhost on port 3000 (as we defined in our server code) and choose websockets instead of polling.

Then we can just like on the server side react to events with ``.on`` and emit events with ``.emit``

Go check out the plunker to use as a chat client (and our server code). There's also code examples of what we talked about here, some basic exercises and one advanced exercise (Typing of the Dead):  
[http://embed.plnkr.co/4GeB5mRIIskWgqj75QPq/](http://embed.plnkr.co/4GeB5mRIIskWgqj75QPq/)
