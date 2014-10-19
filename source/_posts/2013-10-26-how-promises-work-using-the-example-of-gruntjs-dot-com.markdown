---
layout: post
title: "How to handle a bunch of requests using JS promises"
date: 2013-11-01 14:00
comments: true
author: stefanjudis
categories:
- q
- javascript
- grunt
---

Yesterday I procrastinated the stuff I wanted (and should) do and spent a lot of time browsing Github and checking what is going on in the JS world. I discoverd a [discussion](https://github.com/gruntjs/grunt/issues/926) held by the grunt guys about how Grunt can be promoted better. That was quite a good read and it was really nice to see, that these people try to push Grunt forward to make tooling much better for everyone.

A lot of new issues were created at Github to push the project to the next level. It turns out that the [Gruntjs.com](http://gruntjs.com) website is a [seperate repository](https://github.com/gruntjs/gruntjs.com) whose code is available on Github (man, I really love that Open Source approach).

What else can I do than checking out the source? I mean the website of Grunt itself must include a lot of best practices and stuff to discover. I forked it and opened in my editor and there they were - a lot of JS promises...<!-- more -->

Basically I know how promises work, but I have to admit that they still irritate me a bit. It is just another way of thinking and sometimes it takes me a while to understand what is going on when the code in front of me is promises based.

The Grunt website makes usage of the framework [Q](https://npmjs.org/package/q). There are a lot of promises frameworks out there, but Q is with over 10000 downloads a day according to NPM stats not a bad choice. ;)

**Handling of requests at */plugins* **

There is not much functionality in the site (most of it is static content), so the most interessting part of it is probably the `plugins` site. It fetches all available Grunt plugins from "somewhere" - I will explain where it comes from, so keep reading ;) - and displays them. Let us check, what is going on there.

The fun part starts inside of a file called `server.js`.

```javascript
// plugin list route
app.get('/plugin-list', function (req, res, next) {
  // get the plugin list
  pluginListEntity.then(function (entity) {
    // Allow Cross-origin resource sharing
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.setHeader('Content-Type', 'application/json');
    res.setHeader('ETag', entity.etag);

    res.statusCode = 200;
    res.end(new Buffer(entity.json));
  }).fail(function () {
      next();
  });
});
```

What we see here is a route definition for the [Express Framework](http://expressjs.com/). If you are not familiar with Express it describes itself as follows:

> Express is a minimal and flexible node.js web application framework, providing a robust set of features for building single and multi-page, and hybrid web applications.

It is a quite handy framework and when you are into node.js it is definitely worth a try.

But anyway... The shown function will be called when a user enters the url `/plugin-list`. And there it is - the first promise `pluginListEntity`. You see that it is a promise, because an included function `then` gets called with an anonymous callback function. This is the basic pattern when dealing with promises.

**Comparing *callback* and *promise* way **

Particular functions return a promise object instead of the wished data. This becomes quite handy when you have to deal with asynchronous operations, because the "normal" way would be to implement a callback function that deals with wished data, when the operation succeeded.

```javascript
var result;

// callback way
asyncOperation( function( data ) {
  result = data;
} );

// promise way
result = asyncOperation();
result.then( function( data ) {
  result = data;
} );

```

**Creating *pluginListEntity* **

So far so good - let us check where this variable `pluginListEntity` seen in the first code block comes from. It is basically the result of a function called `getPluginListEntity`.

```javascript
function getPluginListEntity() {
  // create new promise
  var deferred = Q.defer();

  // oh, another promise...
  gruntPlugins.fetchPluginList().then(
    function ( pluginList ) {
      var entity = {
        json:JSON.stringify( pluginList )
      };
      var shasum = crypto.createHash( 'sha1' );
      shasum.update( entity.json );
      entity.etag = shasum.digest( 'hex' );
      deferred.resolve( entity );

      // update the entity
      pluginListEntity = deferred.promise;
    } ).fail( function ( e ) {
      deferred.reject( e );
    } );

  // return no values but rather a promise
  return deferred.promise;
}

// set pluginListEntitry at initial start
var pluginListEntity = getPluginListEntity();

// Update function to keep everything fresh
setInterval(function () {
  getPluginListEntity();
}, UPDATE_INTERVAL_IN_SECONDS * 1000);

```

What we see here is the actual "kick off" functionality for the plugin fetch process. It gets called at the initial start of the Express application. The most important thing is, that the variable `pluginListEntity` is set with a promise at initial start by calling `getPluginListEntity`.

`getPluginListEntity` does two things when called. First of all it creates a new promise and returns it. Additionally it refreshes `pluginListEntity` with the created promise. That is actually really smart, because this way it is possible to update it automatically using `setInterval` constantly which is absolutely necessary, because [gruntjs.com](http://gruntjs.com) should be up to date all the time.

But wait... you may have noticed that there is more promises stuff going on inside of `getPluginListEntity`. This line `gruntPlugins.fetchPluginList.then( ... )` hints to another object, that does the actual fetching job for the application and obviously return another promise.

Let us check it out.

**Fetching data from NPM**

```javascript
function fetchPluginList() {
  // return a promise to use it in 'getPluginListEntitry'
  return Q.fcall( function fetchPluginList() {
    var deferred = Q.defer();

    // fetch all grunt plugins

    return deferred.promise;
  } ).then( function getPlugin( list ) {
      var results = _.map( list, function( item ) {
        var deferred = Q.defer();

        // fetch plugin information for each plugin

        return deferred.promise;
      } );
      return Q.all( results );
  } ).then( function getDownloads( results ) {
      var resultsWithDownloads = _.map( results, function( result ) {
        var deferred = Q.defer();

        // fetch download statistics for each plugin

        return deferred.promise;
      } );

      return Q.all( resultsWithDownloads );
  } );
}
```
BAM!!! That is it - much more promises -, but let us break it into little pieces. :)

The needed functionality consists of really a lot of requests made to a [CouchDB](http://couchdb.apache.org/). NPM stores all its plugin inside of a CouchDB available at [http://isaacs.iriscouch.com](http://isaacs.iriscouch.com).

To wrap up what is needed to do:

- Fetch all plugins from database 'reqistry' that include keyword 'gruntplugin' - ** one call **
- Fetch plugin details of all fetched grunt plugins from database 'registry' - ** * calls **
- Fetch download statistics of all fetched plugin - ** * calls **

Just to make sure you understand to advantage of that, here is the callback way of doing it:

```javascript

fetchPluginList( function ( plugins ) {
  getPlugin( plugins, function( pluginsWithDetails ) {
    getDownloads( pluginsWithDetails, function( pluginsWithDownloadStats ) {
      // do something with 'pluginsWithDownladStats'
    } );
  } );
} );
```

Doing it like that is less readable and it is a perfect example of the so called 'callback hell'. Additionally dealing with a lot asynchronous requests can be really tricky and keeping it in sync to call the next callback is not so easy like it sounds.

So what is really going on? The function `fetchPluginList` makes usage of `Q.fcall`. This gives us the possibility to return a new promise by this function. Following is a simplified version of this approach to make it clearer.

**Example for *Q.fcall* **

```javascript
var Q = require( 'q' );

function getPromise () {
  return Q.fcall( function () {
    return 10;
  } ).then( function( value ) {
    return value * 2;
  } ).then( function( value ) {
    return value * 3;
  } );
}

var a = getPromise();

a.then( function( returnValue) {
  console.log( returnValue ); // will log '60'
});
```

This way we are able to stick a lot of functions together and avoid creating callback trees. The end result of a deep nested callback is then easily accessible using the `then` function of the particular promise.

**Example of *Q.defer* with one request**

Now starts the tricky part - in each step ( remember `fetchAll`, `fetchDetails`, `fetchDownloads`? ;) ) will be made an asynchronous request and this request is handled by ...? Yeah, you are right - another promise.

Here is the complete first step to fetch all grunt plugins including one call to the CouchDB:

```javascript
function fetchPluginList() {
  // get new deffered object
  var deferred = Q.defer();
  var keyword = 'gruntplugin';
  var url = 'http://isaacs.iriscouch.com/registry/_design/app/_view/byKeyword?startkey=[%22' +
    keyword + '%22]&endkey=[%22' + keyword + '%22,{}]&group_level=3';

  // fetch all plugins
  request(
    { url : url, json : true },
    function handlePluginList( error, response, body ) {
      if( !error && response.statusCode === 200 ) {
        // yeah - resolve it successfully
        deferred.resolve( body.rows );
      } else {
        // reject it in case of error
        deferred.reject( new Error( error ) );
      }
    }
  );

  // return no value but rather the deffered
  return deferred.promise;
}
```

A new promise is created by using `Q.defer`. `Q.defer` acts as an interface when you have to deal with callback based functions ( like `request` in this case ) and you want to do it the promise way. All you have to do is getting a new defer object by calling `Q.defer()` and then resolving/rejecting it inside of the asynchronous callback function. The response of the request is then easily accessible be calling `then`.

```javascript
var allPlugins = fetchPluginList();

allPlugins.then( function( plugins ) {
  doSomething( plugins );
} );
```

**Example of *Q.defer* with multiple request**

If the first step is clear to you ( if not feel free to comment or ping me on [Twitter](twitter.com/stefanjudis) ) let us check the second and third, because there is a bit more magic going on in it.

```javascript
function getPlugin( list ) {
  // create array full of promises
  var results = _.map( list, function( item ) {
    var deferred = Q.defer();
    var name = item.key[ 1 ];
    var url = 'http://isaacs.iriscouch.com/registry/' + name;

    // fetch plugin information
    request(
      { url : url, json: true },
      function handlePlugin( error, response, body ) {
        if( !error && response.statusCode === 200 ) {
          // yeah - resolve promise for this plugin
          deferred.resolve( condensePlugin( body ) );
        } else {
          // reject promise for this plugin
          deferred.reject( new Error( error ) );
        }
      }
    );
    return deferred.promise;
  } );
  // 'results' is an Array containing a lot of promises
  // -> returns a new promise that will be resolved
  // -> when all promises inside of results succeeded
  return Q.all(results);
}
```

This step does basically the same as the first step, but makes a lot of more requests. For each plugin a seperate request has to be made to fetch the plugin details. The function will be executed with an Array containing all information that was fetched in first call to get all grunt plugins.

This Array ( `list` ) will be transformed using the `map` function of [lo-dash](http://lodash.com/). Each item is replaced by a new deffered object which will be resolved, when the request for plugin information succeeded. Q provides the really nice function `Q.all` which gives us a lot of power to handle multiple requests.

`Q.all` will return a new promise which will be resolved when all promises inside of the handed in Array will be resolved and enriched with detail information.

```javascript
// response of first call to CouchDB
var list = [ {
  key : [ 'something', 'pluginName1' ]
}, {
  key : [ 'something', 'pluginName2' ]
} ];

var allPluginsWithDetails = getPlugin( list );

// do something with plugins when they were
// enriched with detail information
allPluginsWithDetails.then( function( plugins ) {
  doSomething( plugins );
} );
```

**Sticking promises together**

Now, we have created a lot of promises, so let us have another look where we started with a more detailed look and lots of comments. ;)

```javascript
function fetchPluginList() {
  // return a promise
  return Q.fcall( function fetchPluginList() {
    // fetch all grunt plugins with help of a deferred object
    var deferred = Q.defer();
    var keyword = 'gruntplugin';
    var url = 'http://isaacs.iriscouch.com/registry/_design/app/_view/byKeyword?startkey=[%22' +
      keyword + '%22]&endkey=[%22' + keyword + '%22,{}]&group_level=3';
    // fetch all plugins
    request(
      { url : url, json : true },
      function handlePluginList( error, response, body ) {
        if( !error && response.statusCode === 200 ) {
          // resolve deferred with response
          deferred.resolve( body.rows );
        } else {
          deferred.reject( new Error( error ) );
        }
      }
    );
    return deferred.promise;
  } )
  // executed when the promise returned by 'fetchPluginList' was resolved
  // argument 'list' === 'body.rows' in line 15
  .then( function getPlugin( list ) {
    // transform Array with data to Array with promises
    var results = _.map( list, function( item ) {
      var deferred = Q.defer();
      var name = item.key[ 1 ];
      var url = 'http://isaacs.iriscouch.com/registry/' + name;
      // fetch plugin information
      request(
        { url : url, json : true },
        function handlePlugin( error, response, body ) {
          if( !error && response.statusCode === 200 ) {
            // resolve deferred with response
            deferred.resolve( condensePlugin( body ) );
          } else {
            deferred.reject( new Error( error ) );
          }
        }
      );
      return deferred.promise;
    } );
    // return a promise that will be resolved
    // when all promises inside of 'results' are resolved
    return Q.all( results );
  })
  // executed when the promise returned by 'getPlugin was resolved'
  // argument 'results' is an Array which now consists of plugin data
  // with enriched plugin information
  .then( function getDownloads( results ) {
    // transform Array with data to Array with promises
    var resultsWithDownloads = _.map( results, function( result ) {
      var deferred = Q.defer();

      var today = Date.today();
      var oneMonthAgo = today.clone().add( { months : -1 } );

      var startKey = JSON.stringify( [ result.name, oneMonthAgo.toYMD() ] );
      var endKey = JSON.stringify( [ result.name, today.toYMD() ] );

      var url = 'http://isaacs.iriscouch.com/downloads/_design/app/_view/pkg?startkey=' + startKey + '&' + 'endkey=' + endKey;

      // fetch download information
      request(
        { url : url, json : true },
        function handlePlugin( error, response, body ) {
          if (!error && response.statusCode === 200) {
            if ( body.rows && body.rows.length ) {
              result.downloads = body.rows[ 0 ].value;
            } else {
              result.downloads = 'N/A';
            }

            // resolve deffered with enriched result
            deferred.resolve( result );
          } else {
            deferred.reject( new Error( error ) );
          }
        }
      );
      return deferred.promise;
    });

    // return a promise the will be resolved
    // when all promises inside of 'results' are resolved
    return Q.all( resultsWithDownloads );
  });
}

var plugins = fetchPluginList();

// working with result when all requests are down
// argument 'resultsWithDownloads' is result of 'Q.all' at line 89
plugins.then( function( resultsWithDownloads ) {
  doSomething( resultsWithDownloads );
} );
```

**Conclusion**

The `fetchPluginList` function looks first really heavy, but when you understood the principle ( looking at you [@TeixeiraPedro](https://twitter.com/TeixeiraPedro) ;) ) the code has much more structure and is much easier to read. Especially lots of asynchronous operations that has to be in sync at some point loose complexity by using `Q.all`, which absolutely blew my mind.

For me it is clear, that I will structure next projects with lots of requests definitely promises based like the Grunt guys do to make my life easier.

And that is it. I hope you enjoyed it and thanks. :)
