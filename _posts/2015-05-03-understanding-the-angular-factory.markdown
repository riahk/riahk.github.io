---
layout: post
title:  "Understanding the Angular Factory"
date:   2015-05-03 21:15:45
published: true
categories: javascript
---

Angular is an extremely popular MVC framework, and after my brief exposure to it I've happily jumped onto the bandwagon. Its intuitive interface and workflow make app design and coding an extremely fluid and intuitive process, and its wide userbase means you have a huge well of knowledge to draw from when you're learning and improving.

This post isn't about convincing you to use angular, although I do recommend it. Instead I want to explore one of the aspects of the framework that had me confused when I got started. That aspect is the factory.

Factory functions can be difficult to grasp for a number of reasons, the first being that we can easily build an angular application without factories (though hopefully after reading this you'll see the value in using them). Additionally, using a factory requires an understanding of angular's dependency injection strategy.

But wait, what's dependency injection anyway? Dependency injection is a software design pattern related to inversion of control that allows us to load outside libraries into our application, similar to a require statement (in node) or a script tag (in the browser). In angular, dependency injection can deliver the functionality of a library or module to specific parts of a file itself; this improves efficiency by only including dependencies in the exact places they're needed, such as a specific controller. 

In angular, dependency injection looks something like this:

```javascript
app.controller('MainCtrl', function($scope, $dependency1, dependency2) {
  //controller code here
});
```

Here we're creating a controller MainCtrl on app, and the list of arguments in our function (excluding the $scope variable) are our dependencies. The convention in angular is that anything preceded by a '$' is a dependency built directly into angular. But what about that second dependency? Where did it come from?

There are two possibilities: it's either an external library (probably included in our bower components) or it's a factory that we've written. That is what a factory is: a set of functions that can be injected into other parts of our angular app. In fact, to write a factory we barely have to change any of the above code:

```javascript
app.factory('FactName', function($dependency, dependency2) {
});
```

And we can even inject dependencies into our factories, which are then injected into other modules, and so on.

Why use a factory, as opposed to just writing this functionality into a controller? In short, factory functions allow us to keep our code more DRY and more modular. Factories are a fantastic place to write helper functions that you'll need to use throughout your application. Take the following code, for example:

```javascript
app.controller('UserCtrl', function($scope, $http) {
  $scope.getUsers = function() {
    $http.get('/url')
      .success() { ... }
      .error() { ... }
  };
});
```

Our controller here is making a simple http GET request to our server. Everything looks fine, and this code will work (assuming your routes are set up correctly). But we can easily imagine situations where it would be convenient to have this getUsers function on other controllers. This presents a fantastic opportunity to use factories, and in doing so make our code cleaner and more flexible.

```javascript
app.factory('Users', function($scope, $http) {
  return {
    getUsers: function() {
      $http.get('/url')
        .success() { ... }
        .error() { ... }
    }

    //we might also include some other useful functions
  } 
});
```

Now we can inject this factory into some controllers:

```javascript
app.controller('MainCtrl', function($scope, Users) {
  $scope.func = function() {
    Users.getUsers();
  };
});

app.controller('OtherCtrl', function($scope, Users) {
  $scope.func = function() {
    ...
    Users.getUsers();
  };
});
```

Ta-da! We have a function that we can easily use in multiple controllers.

Another important note on factories is that they are instantiated only once in the application. Thus, any variables contained in a factory will be constant across all the controllers it's injected into. This is fantastic for sharing data across multiple controllers. For example:

```
app.factory('Counter', function($scope) {
  return {
    count: 0,
    
    inc: function() {
      this.count++;
    }
  }
});

app.controller('MainCtrl', function($scope, Counter) {
  $scope.count = function() {
    console.log(Counter.count);
  };
});

app.controller('IncCtrl', function($scope, Counter) {
  $scope.inc = function() {
    Counter.inc();
  };
});
```

Both MainCtrl and IncCtrl have access to the Counter factory, and there is only one instance of Counter. Thus, if we run the inc() in IncCtrl, the change in Counter.count will be reflected in MainCtrl as well.

```
//in MainCtrl:
$scope.count(); //prints 0

//in IncCtrl:
$scope.inc();

//in MainCtrl:
$scope.count(); //prints 1
```

To recap: factories are sets of functions that can be injected as dependencies into controllers, allowing for DRYer, more modular code. Use factories for any functionality or data you expect to be shared across controllers in your angular application.
