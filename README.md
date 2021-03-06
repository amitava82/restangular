#Restangular

[![Build Status](https://travis-ci.org/mgonto/restangular.png)](https://travis-ci.org/mgonto/restangular)


Restangular is an AngularJS service that will help you get, delete and update Restfull Resources with very few lines in the Client side. 
This service is a perfect fit for any WebApp that uses Restfull Resources as the API for your application.

#How do I add this to my project?

You can do this by manually downloading the file from dist folder in Github or by using bower and running `bower install restangular`

You can click [here to download development unminified version](https://raw.github.com/mgonto/restangular/master/dist/restangular.js) or you can click [here to download minified production version](https://raw.github.com/mgonto/restangular/master/dist/restangular.min.js)

#Dependencies

Restangular depends on Angular, Angular-Resources and (Underscore or Lodash).

#Starter Guide

## Adding dependency to Restangular module in your app

The first thing you need to do after adding link to script file, is mentioning in your app that you'll use Restangular.

````javascript
var app = angular.module('angularjs-starter', ['restangular']);
````

## Using Restangular

Now that you have everything configured, you can just inject this Service to any Controller or Directive like any other :)

### Creating Main Restangular object

There're 2 ways of creating a main Restangular object. 
The first one and most common one is by stating the main route of all requests. 
The second one is by stating the main route and object of all requests.

````javascript
// Only stating main route
Restangular.all('accounts')

// Stating main object
Restangular.one('accounts', 1234)
````

### Fetching elements from Main object

Now that we have our main Object lets's start playing with it.

````javascript
// First way of creating a Restangular object. Just saying the base URL
var baseAccounts = Restangular.all('accounts');

// This will query /accounts and return a promise. As Angular supports setting promises to scope variables
// as soon as we get the information from the server, it will be shown in our template :)
$scope.allAccounts = baseAccounts.getList();

var newAccount = {name: "Gonto's account"};

// POST /accounts
baseAccounts.post(newAccount);

//Here we use Promises then 
// GET /accounts
baseAccounts.getList().then(function (accounts) {
  // Here we can continue fetching the tree :).

  var firstAccount = accounts[0];
  // This will query /accounts/123/buildings considering 123 is the id of the firstAccount
  $scope.buildings = firstAccount.getList("buildings");
  
  // GET /accounts/123/places?query=param with request header: x-user:mgonto
  $scope.loggedInPlaces = firstAccount.getList("places", {query: param}, {'x-user': 'mgonto'})

  // This is a regular JS object, we can change anything we want :) 
  firstAccount.name = "Gonto"

  // PUT /accounts/123. The name of this account will be Gonto from now on
  firstAccount.put();

  // DELETE /accounts/123 We don't have first account anymore :(
  firstAccount.remove();
  
  var myBuilding = {
    name: "Gonto's Building",
    place: "Argentina"
  };
  
  // POST /accounts/123/buildings with MyBuilding information
  firstAccount.post("Buildings", myBuilding).then(function() {
    console.log("Object saved OK");
  }, function() {
    console.log("There was an error saving");
  });

  // GET /accounts/123/users?query=params
  firstAccount.getList("users", {query: params}).then(function(users) {
    // Instead of posting nested element, a collection can post to itself
    // POST /accounts/123/users
    users.post({userName: 'unknown'});
    
    // Custom methods are available now :).
    // GET /accounts/123/users/messages?param=myParam
    users.customGET("messages", {param: "myParam"})
    
    var firstUser = users[0];

    // GET /accounts/123/users/456. Just in case we want to update one user :)
    $scope.userFromServer = firstUser.get();
    
    // ALL http methods are available :)
    // HEAD /accounts/123/users/456
    firstUser.head()

  });

}, function errorCallback() {
  alert("Oops error from server :(");
})

// Second way of creating Restangular object. URL and ID :)
var account = Restangular.one("accounts", 123);

// GET /accounts/123?single=true
$scope.account = account.get({single: true});

// POST /accounts/123/messages?param=myParam with the body of name: "My Message"
account.customPOST("messages", {param: "myParam"}, {}, {name: "My Message"})
````

## Configuring Restangular

### Properties
Restangular comes with some defaults for all of it's properties but you can configure them. All of this properties have setters so that you can change them.

#### baseUrl
The base URL for all calls to your API. For example if your URL for fetching accounts is http://example.com/api/v1/accounts, then your baseUrl is `/api/v1`. The default baseUrl is an empty string which resolves to the same url that AngularJS is running, so you can also set an absolute url like `http://api.example.com/api/v1` if you need do set another domain.

#### extraFields
This are the fields that you want to save from your parent resources if you need to display them. By default this is an Empty Array which will suit most cases

#### urlCreator
This is the factory that will create URLs based on the resources. For the time being, only Path UrlCreator is implemented. This means that if you have a resource names Building which is a child of Account, the URL to fetch this will be `/accounts/123/buildings`. In the future, I'll implement more UrlCreator like QueryParams UrlCreator.

#### responseExtractor
There are sometimes when the data from the response isn't the whole response, but it's wrapped in another object. For example, if your response is the following, you'd extract the data object:

````javascript
{
    status: 200,
    data: {
        name: "Gonto"
    }
}
````

The responseExtractor is a function that will be called with every response from the server that receives 2 arguments: The first one is the response itself, the second one is the HTTP method being run.

#### restangularFields

Restangular required 3 fields for every "Restangularized" element. This are:

* id: Id of the element
* route: name of the route of this element
* parentResource: the reference to the parent resource

All of this fields except for `id` are handled by Restangular. You can configure the name of the property that will be binded to all of this fields by setting restangularFields property. By default, the values are id, route and parentResource respectively.

### How to configure them
You can configure this properties inside the config method of your app

````javascript
app.config(function(RestangularProvider) {
    RestangularProvider.setBaseUrl('/api/v1');
    RestangularProvider.setExtraFields(['name']);
    RestangularProvider.setResponseExtractor(function(response, operation) {
        return response.data;
    });
    // In this case we configure that the id of each element will be the __id field and we change the Restangular route. We leave the default value for parentResource
    RestangularProvider.setRestangularFields({
      id: "__id",
      route: "restangularRoute"
    });
});

````

# Methods description

There're 3 sets of methods. Collections have some methods and elements have others. There're are also some common methods for all of them

## Element methods
* **get([queryParams, headers])**: Gets the element. Query params and headers are optionals
* **getList(subElement, [queryParams, headers])**: Gets a nested resource. subElement is mandatory. It's a st**ring with the name of the nested resource (and URL)**. For example `buildings`
* **put([queryParams, headers])**: Does a put to the current element
* **post(subElement, elementToPost, [queryParams, headers])**: Does a POST and creates a subElement. Subelement is mandatory and is the nested resource. Element to post is the object to post to the server
* **remove([queryParams, headers])**: Does a DELETE
* **head([queryParams, headers])**: Does a HEAD
* **trace: ([queryParams, headers])**: Does a TRACE
* **options: ([queryParams, headers])**: Does a OPTIONS
* **patch([queryParams, headers])**: Does a PATCH

## Collection methods
* **getList([queryParams, headers]): Gets itself again (Remember this is a collection)**.
* **post(elementToPost, [queryParams, headers])**: Creates a new element of this collection.
* **head([queryParams, headers])**: Does a HEAD
* **trace: ([queryParams, headers])**: Does a TRACE
* **options: ([queryParams, headers])**: Does a OPTIONS
* **patch([queryParams, headers])**: Does a PATCH

## Custom methods
* **customGET(path, [params, headers])**: Does a GET to the specific path. Optionally you can set params and headers.
* **customGETLIST(path, [params, headers])**: Does a GET to the specific path. **In this case, you expect to get an array, not a single element**. Optionally you can set params and headers.
* **customDELETE(path, [params, headers])**: Does a DELETE to the specific path. Optionally you can set params and headers.
* **customPOST(path, [params, headers, elem])**: Does a POST to the specific path. Optionally you can set params and headers and elem. Elem is the element to post. If it's not set, it's assumed that it's the element itself from which you're calling this function.
* **customPUT(path, [params, headers, elem])**: Does a POST to the specific path. Optionally you can set params and headers and elem. Elem is the element to post. If it's not set, it's assumed that it's the element itself from which you're calling this function.
* **customOperation(operation, path, [params, headers, elem])**: This does a custom operation to the path that we specify. This method is actually used from all the others in this subsection. Operation can be one of: get, post, put, delete, head, options, patch, trace
 
Let's see an example of this:

````javascript
//GET /accounts/123/messages
Restangular.one("accounts", 123).customGET("messages")

//GET /accounts/messages?param=param2
Restangular.all("accounts").customGET("messages", {param: "param2"})
````
 
# FAQ

#### **I need to send one header in EVERY Restangular request, how do I do this?**

Restangular uses $http inside, so you can actually set default headers by using $httpProvider. This also applies to XSRF headers as well

````javascript
app.config(["$httpProvider", function($httpProvider) {
  $httpProvider.defaults.headers.common['Tenant-id'] = 'X';
  $httpProvider.defaults.headers.get['Gonto-id'] = 'P';
}]);
````

#### **After getting some List, when I want to add a new element to the array, it gives me an error saying that push method doesn't exist**

When you actually get some list by doing

````javascript
$scope.owners = house.getList('owners')
````

You're actually assigning a Promise to the owners value of the $scope. You can actually iterate over owners and display its values in the template because AngularJS "understands" promises, and while the promise isn't resolved, it will return an empty array/value and once the value is returned, it will use that returned value.
If you actually want to do something with the list of owners afterwards you can do the following:

````javascript
// Option 1
$scope.owners = house.getList('owners')

// Later in the code
$scope.owners.then(function(owners) {
  owners.push({name: "Gonto"});
});

// Option 2. I like this one better
$scope.owners = [];
house.getList('owners').then(function(owners) {
  $scope.owners = owners;
});

// Later in the code
$scope.owners.push({name: "Gonto"});

````

#### Why does this depend on Lodash / Underscore?

This is a very good question. I could've done the code so that I don't depend on Underscore nor Lodash, but I think both libraries make your life SO much easier. They have all of the "functional" stuff like map, reduce, filter, find, etc. 
With these libraries, you always work with Inmutable stuff, you get compatibility for browsers which don't implement ECMA5 nor some of these cool methods, and they're actually quicker.
So, why not use it? If you've never heard of them, by using Restangular, you could start using them. Trust me, you're never going to give them up after this!



# Contribute

In order to Contribute just git clone the repository and then run:

````
npm install grunt-cli --global
npm install
grunt
````

Be sure to have PhantomJS installed as Karma tests use it. Otherwise, in mac just run `brew install phantomjs`

All changes must be done in `src/restangular.js`and then after running `grunt`all changes will be submited to `dist/`

Please submit a Pull Request or create issues for anything you want :).

# Server Frameworks

This server frameworks play real nice with Restangular, as they let you create a Nested Restful Resources API easily:

* Ruby on Rails
* CakePHP for PHP
* Play1 & 2 for Java & scala
* Restify and Express for NodeJS


# Releases Notes

## 0.4.6
* Added Custom methods to all Restangular objects. Check it out in the README

## 0.4.5
* Fixed but that didn't let ID to be 0.
* Added different Collection methods and Element methods
* Added posibility po do a post in a collection to post an element to itself
* Added Travis CI for build
* Fixed bug with parentResource after a post of a new element
* When doing a post, if no element is returned, we enhance the object received as a parameter


## 0.3.4
* Added new HTTP methods to use: Patch, Head, Trace and Options (thanks @pauldijou)
* Added tests with Karma for all functionality.

## 0.3.3
* Restangular fields can now be configured. You can set the id, route and parentResource fields. They're not hardcoded anymore

## 0.3.2
* Added ResponseExtractor for when the real data is wrapped in an envelope in the WebServer response.

## 0.3.1

* Now all methods accept Headers. You can query `account.getList('buildings', {query: 'param'}, {'header': 'mine'})`

## 0.2.1

* Added query params to all methods. getList, post, put, get and delete accept query params now.

## 0.2.0
* Added post method to all elements. Now you can also create new elements by calling `account.post('buildings', {name: "gonto"})`. 

## 0.1.1
* Changed `elem.delete()` to `elem.remove()` due to errors with Closure Compiler in Play 2 


# License

The MIT License

Copyright (c) 2013 Martin Gontovnikas http://www.gon.to/

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

