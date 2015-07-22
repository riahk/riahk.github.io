---
layout: post
title:  "Asynchronous Functions and You"
date:   2015-04-29 21:15:45
published: true
categories: javascript
---
If you've been coding in javascript for a while, odds are you've heard of callback hell. If you haven't, you probably will soon (perhaps while learning the ever-popular node), and you'll probably hear varying levels of anxiety around it. Regardless of your exposure to asynchronous functions, I'm here to tell you that they're really nothing to be afraid of. Async is your friend!

####What the heck does asynchronous even mean, though?####
As its name would suggest, asynchronous code runs outside the normal flow of a program (AKA all the 'synchronous' functions). Generally an async function is one that may take a while to complete, such as reading an external file or making a request to a database/server. Because these processes are time-consuming, synchronous code doesn't wait for asynchronous code to finish before executing. Take the following snippet of code as an example:

```javascript
setTimeout(function() { console.log('Hello!') }, 2000);
console.log('Goodbye!');
```

The setTimeout function waits two seconds before executing, so one might expect the following output:

```javascript
//two seconds pass
Hello!
Goodbye!
```

But this is not so! The **actual** output is something more like this:

```javascript
Goodbye!
//approximately two seconds later
Hello!
```

Seems a bit backwards, right? But this is simply something we have to accept, because this is how javascript handles asynchronous functions like setTimeout. And in the long run it's more efficient; a program would run much slower if it waited for each asynchronous function to finish.

While this handling solves the problem of long execution times, it creates another problem: what do we do when we want to return the result of an asynchronous call? The short story is we don't; instead, we pass a callback into our asynchronous function.

Callbacks sound mysterious and weird but in actuality a callback is simply a function, which will be executed on the result of an asynchronous process. In the above example, console.log is the callback passed into setTimeout and it gets executed after the interpreter recognizes that two seconds have passed. (The exact system behind how this happens is somewhat complex, but for our purposes we don't need to dive into that. For those interested, [here](http://blog.carbonfive.com/2013/10/27/the-javascript-event-loop-explained/) is a great post on the javascript event loop).

Let's take a look at a more practical example of asynchronous code: reading and serving a file on node. File-reading can be a very long process: the computer has to read the file path, go to that location in memory, and send the data in that file back to the server. This can take a while, especially for large files. So javascript can let this process run in the background, without delaying the rest of a program. Here's how we might read a text file:

```javascript
fs.readFile('example.txt', function(err, data) {
  console.log(data);
});
```

These few lines of code read a file and then log it to the console, and it does it asynchronously; the function where we call console.log is a callback! The readFile function goes through all the steps to find and read a file, and returns it as a data string. This data is passed into a user-specified callback.

We can have more fun with this, though, by appending some text to our file...

```javascript
fs.readFile('example.txt', function(err, data) {
  data += 'some more text';
  
  fs.writeFile('example.txt', data, function(err, data) {
    console.log('we saved it!');
  });
});
```

Inside our readFile function we called writeFile, another asynchronous function, and thus passed another callback into it. We've created a chain of asynchronous processes, triggered by the first call to readFile: a file is read, edited, and then saved. What if we were to add a couple more console.logs into this code?

```javascript
fs.readFile('example.txt', function(err, data) {
  console.log('read file.');
  data += 'some more text';
  console.log('edited file.');
  
  fs.writeFile('example.txt', data, function(err, data) {
    console.log('we saved it!');
  });
});

console.log('When will this log???');
```

Take a second to make a hypothesis about the output of this code, then check your expectations down at the bottom of this post.

####Even More Fun with Async Functions! (+The PYRAMID OF DOOM)####

The above example is a simple, yet very useful usage of asynchronous functions. As you write more complex code, you may find yourself writing longer chains of callbacks. And your code might start looking a bit like this:

```javascript
asyncFunc1(stuff, function(err, data) {
  ...
  asyncFunc2(moreStuff, function(err, data) {
    ...
    asyncFunc3(evenMoreStuff, function(err, data) {
      ...
      asyncFunc4(almostDone, function(err, data) {
        ...
      });
    });
  });
});
```

And it can get longer! That's a lot of indentation, and over time your code becomes less and less pretty/easy to read. As programmers we have a name for this phenomenon: the pyramid of doom. It's unavoidable! Or is it?

In this kind of situation, we can use Promises, a mechanic designed to simplify asynchronous code. I won't be going into great detail on Promises in this post, but if you're curious you can dive into a great Promise library like [Bluebird](https://github.com/petkaantonov/bluebird) or [Q](https://github.com/kriskowal/q).

Hopefully you now have a better idea of asynchronous functions and how to use them. Go forth and code!

(Oh yeah, and here's the output of that code from earlier:)

```javascript
'When will this log?'
'read file.'
'edited file.'
'we saved it!'
```

