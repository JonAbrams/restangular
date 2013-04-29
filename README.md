#Restangular

[![Build Status](https://travis-ci.org/mgonto/restangular.png)](https://travis-ci.org/mgonto/restangular)


Restangular is an AngularJS service that will help you get, delete and update Restfull Resources with very few lines in the Client side. 
This service is a perfect fit for any WebApp that uses Restfull Resources as the API for your application.

#How do I add this to my project?

You can download this by:

* Using bower and running `bower install restangular`
* Using npm and running `npm install restangular`
* Downloading it manually by clicking [here to download development unminified version](https://raw.github.com/mgonto/restangular/master/dist/restangular.js) or [here to download minified production version](https://raw.github.com/mgonto/restangular/master/dist/restangular.min.js)
* Using [JsDelivr CDN files](https://github.com/jimaek/jsdelivr/tree/master/files/restangular):

````html
<!-- Use LATEST folder to always get the latest version-->
<script type="text/javascript" src="http://cdn.jsdelivr.net/restangular/latest/restangular.js"></script>
<script type="text/javascript" src="http://cdn.jsdelivr.net/restangular/latest/restangular.min.js"></script>

<!-- Or use TAG number for specific version -->
<script type="text/javascript" src="http://cdn.jsdelivr.net/restangular/0.5.3/restangular.js"></script>
<script type="text/javascript" src="http://cdn.jsdelivr.net/restangular/0.5.3/restangular.min.js"></script>
````


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

### Let's code!

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

//You can do RequestLess "connections" if you need as well

// Just ONE GET to /accounts/123/buildings/456
Restangular.one('accounts', 123).one('buildings', 456).get()

// Just ONE GET to /accounts/123/buildings
Restangular.one('accounts', 123).all('buildings').getList()

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

#### onElemRestangularized
This is a hook. After each element has been "restangularized" (Added the new methods from Restangular), this will be called. It means that if you receive a list of objects in one call, this method will be called first for the collection and then for each element of the collection.
This callback is a function that has 3 parameters:

* **elem**: The element that has just been restangularized. Can be a collection or a single element.
* **isCollection**: Boolean indicating if this is a collection or a single element.
* **what**: The model that is being modified. This is the "path" of this resource. For example `buildings`
 
This can be used together with `addRestangularMethod` (Explained later) to add custom methods to an element

#### responseInterceptor (or responseExtractor. It's an Alias)
The responseInterceptor is called after we get each response from the server. It's a function that receives 4 arguments:

* **response**: The response got from the server
* **operation**: The operation made. It'll be the HTTP method used except for a `GET` which returns a list of element which will return `getList` so that you can distinguish them.
* **what**: The model that's being requested. It can be for example: `accounts`, `buildings`, etc.
* **url**: The relative URL being requested. For example: `/api/v1/accounts/123`

Some of the use cases of the responseInterceptor are handling wrapped responses and enhancing response elements with more methods among others.

#### listTypeIsArray

You can set in this property wether the `getList` method will return an Array or not. Most of the times, it will return an array, as it returns a collection of values. However, sometimes this method returns first some metadata and inside it has the array. So this can be used together with `responseExtractor` to get the real array. The default value is true.

#### restangularFields

Restangular required 3 fields for every "Restangularized" element. This are:

* id: Id of the element. Default: id
* route: Name of the route of this element. Default: route
* parentResource: The reference to the parent resource. Default: parentResource
* restangularCollection: A boolean indicating if this is a collection or an element. Default: restangularCollection
* what: The name of the parameter to be used in inner $resource to handle the PATH of the url. For example, in `/users/123/messages`, `messages`represents the "what". Default: restangularWhat 

All of this fields except for `id` are handled by Restangular, so most of the time you won't change them. You can configure the name of the property that will be binded to all of this fields by setting restangularFields property.

#### requestSuffix

If all of your requests require to send some suffix to work, you can set it here. For example, if you need to send the format like `/users/123.json`you can add that `.json` to the suffix using the `setRequestSuffix`method

### How to configure them
You can configure this properties inside the config method of your app

````javascript
app.config(function(RestangularProvider) {
    RestangularProvider.setBaseUrl('/api/v1');
    RestangularProvider.setExtraFields(['name']);
    RestangularProvider.setResponseExtractor(function(response, operation) {
        return response.data;
    });
    
    RestangularProvider.setListTypeIsArray(true);
    
    // In this case we configure that the id of each element will be the __id field and we change the Restangular route. We leave the default value for parentResource
    RestangularProvider.setRestangularFields({
      id: "__id",
      route: "restangularRoute"
    });
    
    RestangularProvider.setRequestSuffix('.json');
});

````

## Methods description

There're 3 sets of methods. Collections have some methods and elements have others. There're are also some common methods for all of them

### Element methods
* **get([queryParams, headers])**: Gets the element. Query params and headers are optionals
* **getList(subElement, [queryParams, headers])**: Gets a nested resource. subElement is mandatory. **It's a string with the name of the nested resource (and URL)**. For example `buildings`
* **put([queryParams, headers])**: Does a put to the current element
* **post(subElement, elementToPost, [queryParams, headers])**: Does a POST and creates a subElement. Subelement is mandatory and is the nested resource. Element to post is the object to post to the server
* **remove([queryParams, headers])**: Does a DELETE
* **head([queryParams, headers])**: Does a HEAD
* **trace([queryParams, headers])**: Does a TRACE
* **options([queryParams, headers])**: Does a OPTIONS
* **patch([queryParams, headers])**: Does a PATCH
* **one(route, id)**: Used for RequestLess connections and URL Building. See section below.
* **all(route)**: Used for RequestLess connections and URL Building. See section below.

### Collection methods
* **getList([queryParams, headers]): Gets itself again (Remember this is a collection)**.
* **post(elementToPost, [queryParams, headers])**: Creates a new element of this collection.
* **head([queryParams, headers])**: Does a HEAD
* **trace: ([queryParams, headers])**: Does a TRACE
* **options: ([queryParams, headers])**: Does a OPTIONS
* **patch([queryParams, headers])**: Does a PATCH

### Custom methods
* **customGET(path, [params, headers])**: Does a GET to the specific path. Optionally you can set params and headers.
* **customGETLIST(path, [params, headers])**: Does a GET to the specific path. **In this case, you expect to get an array, not a single element**. Optionally you can set params and headers.
* **customDELETE(path, [params, headers])**: Does a DELETE to the specific path. Optionally you can set params and headers.
* **customPOST(path, [params, headers, elem])**: Does a POST to the specific path. Optionally you can set params and headers and elem. Elem is the element to post. If it's not set, it's assumed that it's the element itself from which you're calling this function.
* **customPUT(path, [params, headers, elem])**: Does a POST to the specific path. Optionally you can set params and headers and elem. Elem is the element to post. If it's not set, it's assumed that it's the element itself from which you're calling this function.
* **customOperation(operation, path, [params, headers, elem])**: This does a custom operation to the path that we specify. This method is actually used from all the others in this subsection. Operation can be one of: get, post, put, delete, head, options, patch, trace
* **addRestangularMethod(name, operation, [path, params, headers, elem])**: This will add a new restangular method to this object with the name `name` to the operation and path specified (or current path otherwise). There's a section on how to do this later. 
 
Let's see an example of this:

````javascript
//GET /accounts/123/messages
Restangular.one("accounts", 123).customGET("messages")

//GET /accounts/messages?param=param2
Restangular.all("accounts").customGET("messages", {param: "param2"})
````

## URL Building
Sometimes, we have a lot of entities names with their ids and we just want to fetch the later entity. In those cases, doing a request for everything to get the last entity is an overkill. For those cases, I've added the possibility to create URLs using the same API as creating a new Restangular object. This connections are created without doing any request. Let's see how to do this:

````javascript

var restangualrSpaces = Restangular.one("accounts",123).one("buildings", 456).all("spaces");

// This will do ONE get to /accounts/123/buildings/456/spaces
restangularSpaces.getList()

// This will do ONE get to /accounts/123/buildings/456/spaces/789
Restangular.one("accounts", 123).one("buildings", 456).one("spaces", 789).get()

// POST /accounts/123/buildings/456/spaces
Restangular.one("accounts", 123).one("buildings", 456).all("spaces").post({name: "New Space"});
````

## Creating new Restangular Methods

Let's assume that your API needs some custom methods to work. If that's the case, always calling customGET or customPOST for that method with all parameters is a pain in the ass. That's why every element has a `addRestangularMethod` method. This can be used together with the hook `setOnElemRestangularized` to do some neat stuff. Let's see an example to learn this:

````javascript
//In your app configuration (config method)
RestangularProvider.setOnElemRestangularized(function(elem, isCollection, route) {
    if (!isCollection && route === "buildings") {
        // This will add a method called evaluate that will do a get to path evaluate with NO default
        // query params and with some default header
        // signature is (name, operation, path, params, headers, elementToPost)
        elem.addRestangularMethod('evaluate', 'get', 'evaluate', undefined, {'myHeader': 'value'});
    }
    return elem;
})

// Then, later in your code you can do the following:

//GET to /buildings/123/evaluate?myParam=param with headers myHeader: value
//Signature for this "custom created" methods is (params, headers, elem)
// If something is set to any of this variables, the default set in the method creation will be overrided
// If nothing is set, then the defaults are sent
building.evaluate({myParam: 'param'});

//GET to /buildings/123/evaluate?myParam=param with headers myHeader: specialHeaderCase
building.evaluate({myParam: 'param'}, {'myHeader': 'specialHeaderCase'});

````
 
# FAQ

#### **How can I handle errors?**

Errors can be checked on the second argument of the then.

````javascript
Restangular.all("accounts").getList().then(function() {
  console.log("All ok");
}, function(response) {
  console.log("Error with status code", response.status);
});
````

#### **I need to send one header in EVERY Restangular request, how do I do this?**

Restangular uses $http inside, so you can actually set default headers by using $httpProvider. This also applies to XSRF headers as well

````javascript
app.config(["$httpProvider", function($httpProvider) {
  $httpProvider.defaults.headers.common['Tenant-id'] = 'X';
  $httpProvider.defaults.headers.get['Gonto-id'] = 'P';
}]);
````

#### **My response is actually wrapped with some metadata. How do I get the data in that case?**

So, let's assume that your data is the following:

````javascript
 // When getting the list, this is the response.
{
  "status":"success",
  "data": {
    "data": [{
      "id":1,
      // More data
    }],
    "meta": {
      "totalRecord":100
    }
  }
}

// When getting a single element, this is the response.
{
  "status":"success",
  "data": {
    "id" : 1
    // More data
  }
}
````

In this case, you'd need to configure Restangular's `responseExtractor`and `listTypeIsArray`. See the following:

````javascript
app.config(function(RestangularProvider) {
    // First let's set listTypeIsArray to false, as we have the array wrapped in some other object.
    RestangularProvider.setListTypeIsArray(false);
    
    // Now let's configure the response extractor for each request
    RestangularProvider.setResponseExtractor(function(response, operation) {
      // This is a get for a list
      var newResponse;
      if (operation === "getList") {
        // Here we're returning an Array which has one special property metadata with our extra information
        newResponse = response.data.data;
        newResponse.metadata = response.data.meta;
      } else {
        // This is an element
        newResponse = response.data;
      }
      return newResponse;
    });
});
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

## 0.5.5
* Changed by default from Underscore to Lodash. They both can be used anyway. (thanks @pauldijou)
* Added tests for both Underscore and Lodash to check it's working. (thanks @pauldijou)

## 0.5.4
* Added onElemRestangularized hook
* Added posibility to add your own Restangular methods

## 0.5.3
* Added the posibility to do URL Building and RequestLess tree navigations
* Added alias to `do[method]`. For example, Now you can do `customPOST` as well as `doPOST`

## 0.5.2
* responseExtractor renamed to responseInterceptor. Added alias from responseExtractor to responseInterceptor to mantain backwards compatibility
* responseExtractor now receives 4 parameters. Response, operation, what (path of current element) and URL
* Error function for any Restangular action now receives a response to get StatusCode and other interesting stuff

## 0.5.1
* Added listTypeIsArray property to set getList as not an array.

## 0.5.0
* Added `requestSuffix`configuration for requests ending en .json
* `what` field is now configurable and not hardcoded anymore
* All instance variables from `RestangularProvider` are now local variables to reduce visibility
* Fully functional version with all desired features

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

