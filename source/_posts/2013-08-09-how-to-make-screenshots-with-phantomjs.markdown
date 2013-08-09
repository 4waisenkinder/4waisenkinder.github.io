---
layout: post
title: "How to make screenshots with phantomjs"
date: 2013-08-09 10:40
comments: true
author: stefanjudis
categories:
- phantomjs 
---

Recently I started developing my first grunt plugin called [grunt-photobox](https://npmjs.org/package/grunt-photobox). The goal was to make the layout QA-process before shipping a new feature much easier. It should take screenshots of every relevant site and compare it to the last one - [post about how to use it here](http://4waisenkinder.de/blog/2013/07/26/grunt-photobox-secure-yourself-against-broken-layout/).

The base for that should be [phantomjs](http://phantomjs.org/). It's a headless webkit browser, that gives you all functionality a "real" browser has plus a few features for making screenshots or reading files from disk for example. You can run it via command line and pass in a script to execute inside the scope of phantomjs.

```
$ phantomjs script.js
```

<!-- more -->

Inside of this script you have got a few more global variables available than usual - check them out [here](https://github.com/ariya/phantomjs/wiki/_pages). For making screenshots we will use the webpage- and system-api ( will be explained later on ). Phantomjs provides a ```require``` function to get the functionality we need. When you do a lot of nodejs you are probably familiar with that principle already.

[The script](https://github.com/stefanjudis/phantomjs-screenshot) for making screenshots has 50 lines and is not really complicated, so lets have a look:

```js
var system = require( 'system' ),
    webpage = require( 'webpage' ),
    page = webpage.create(),
    url = system.args[ 1 ] || 'http://4waisenkinder.de', // set a default value if argument was not set
    path = system.args[ 2 ] || './', // set a default value if argument was not set
    width = +system.args[ 3 ] || 1000, // set a default value if argument was not set
    height = +system.args[ 4 ] || 1200; // set a default value if argument was not set

```

The variable declaration at top of the page ( yes, I know, Crockford-Style ;) ) shows us all we need. ```system``` will take the responsibiliy of getting arguments with which we fired up phantomjs. This way the script is more generic and we will be able to pass in arguments like the url we want to make a picture of, the path of the picture, and width and height of the headless browser window ( this becomes handy when you want to check your responsive web design ). ```webpage``` includes the actual browser functionality for opening a page and rendering it later on.

```js
page.onError = function ( msg ) {
    system.stderr.writeLine( 'ERROR:' + msg );
};

page.onConsoleMessage = function( msg, lineNum, sourceId ) {
    system.stdout.writeLine( 'CONSOLE: ' + msg, lineNum, sourceId );
};
```

These seven lines are quite important for debugging purposes, without it you can rarely know what is going on. Phantomjs needs to know how to handle any output inside of its JavaScript console. So let's define to listen to messages and errors to write it to ```stderr```and ```stdout``` later on.

```js
page.viewportSize = {
    height : height,
    width  : width
};

page.clipRect = {
  height : height,
  width  : width
};
```

To set up the size of the window we need to set the view port size of the ```page``` object, which was initialized at the beginning. In this case ```viewportSize``` means the actual size of the simulated window. But to make proper screenshots we have to define the property ```clipRect``` as well to make the image fitting to the actual window view size. According to phantomjs documentation ```clipRect``` defines the size of the area that should be rasterized when calling the ```render``` function.

So let's do the last step:

Photosession-Time!!! :)

```js
page.open( url, function( status ) {
  console.log( 'Opened url with status: ' + status );

  window.setTimeout( function() {
    var imgPath = path +
                    'img/' +
                    url.replace( /(http:\/\/|https:\/\/)/, '').replace( /\//g, '-') +
                    '-' + width + 'x' + height +
                    '.png';

    console.log( 'Rendering ' + imgPath );
    page.render( imgPath );

    phantom.exit();
  }, 500 );
} );
```

And that is it. :)

The ```page``` object opens the defined url and provides a callback, when it is done. The image path and name to save the image will be built up and the ```page.render()``` command then renders the image. 

The console output works here because we defined the listeners to write it to the system on logs and errors. 

The timeout is defined to provide some time to fetch images and execute included scripts. Many people are doing it this way. It is not 100% clean, but it does its job. If you don't like it, there is also the possibility to define a callback at ```page.onLoadFinished```. But it is fine for me now. :)

And the last thing is to stop phantom with the ```exit``` command and return with a given status code. Per default it returns the status code 0, which means that everything was fine.

That's all the "magic" you need to take pictures of any site. The command to start the script is:

```
$ phantomjs script.js http://google.com ./ 1000 1200
Opened url with status: success
Rendering ./img/google.com-1000x1200.png
```

The script probably needs some improvements for error handling ( errors on page opening or missing system arguments ), but it has just the purpose to get started. :)


If you are interested in more stuff, how I build it into a grunt plugin, let me know. I am really excited about this whole phantomjs stuff and would like to share it.

Thanks for reading and if you have any comments or thoughts leave a comment or ping me on [Twitter](https://twitter.com/stefanjudis).