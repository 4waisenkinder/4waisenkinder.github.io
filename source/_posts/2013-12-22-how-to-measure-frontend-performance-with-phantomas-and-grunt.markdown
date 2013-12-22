---
layout: post
title: "How to measure frontend performance with Grunt"
date: 2013-12-22 23:28
comments: true
author : stefanjudis
categories: 
- phantomas
- performance
- metrics
---
When you are a frontend developer, you are on the constant journey to find the right tool to measure the performance of your site. Sure, there are the Developer Tools of your choice, [Google Pagespeed](https://developers.google.com/speed/pagespeed/) or [webpagespeed.org](http://webpagetest.org) available. And those tools are great (if you do not know them, you should definitely check them out), but for me it was always "just" nice to use these tools. They provide lots of useful information and I check them on a regular basis, but as far as I know, these tools do not provide a kind of timeline view of my daily work. They only give useful information to a given time - that is it. What I want to know, is how the site evolved after my latest deploy to production ... <!-- more --> 

Did the size of CSS / JavaScript decrease or increase? How many Ajax requests are made on page load? How many DOM queries are executed? And how did my latest changes effect the overall performance of the site?

Sure, I could answer all these questions manually and could keep track of them in some way, but that is not in the nature of a developer, right? ;)

That is definitely a task that belongs to an automated process and should be done right after the deployment to production, so that I can check right away, what happened and how the production site changed.

Well, ( **attention shameless self promotion** ) - I wrote a [Grunt](http://gruntjs.com) plugin called [grunt-phantomas](https://npmjs.org/package/grunt-phantomas) to solve exactly that problem (preview of generated example results [here](http://stefanjudis.github.io/grunt-phantomas/gruntjs/)).

Two weeks ago I discovered a NPM module called [Phantomas](https://github.com/macbre/phantomas) and it absolutely blew my mind. It is a command line tool, which scans any site you like via [PhantomJS](http://phantomjs.org/). You can easily install it via `npm install -g phantomas` and after that, you are ready to go. To show how powerful this tool is, let us have a look at the by Phantomas given statistics of the [Grunt site](http://gruntjs.com) as an example.

```
$ phantomas --url http://gruntjs.com

phantomas v0.9.0 metrics for <http://gruntjs.com>:

* requests: 29
* gzipRequests: 3
* postRequests: 0
* httpsRequests: 0
* redirects: 0
* notFound: 0
* timeToFirstByte: 807
* timeToLastByte: 833
* bodySize: 179021
* contentLength: 311187
* ajaxRequests: 0
* htmlCount: 1
* htmlSize: 1151
* cssCount: 2
* cssSize: 1630
* jsCount: 5
* jsSize: 41272
* jsonCount: 0
* jsonSize: 0
* imageCount: 21
* imageSize: 134968
* webfontCount: 0
* webfontSize: 0
* base64Count: 0
* base64Size: 0
* otherCount: 0
* otherSize: 0
* cacheHits: 3
* cacheMisses: 0
* cachingNotSpecified: 4
* cachingTooShort: 2
* cachingDisabled: 20
* domains: 6
* maxRequestsPerDomain: 21
* medianRequestsPerDomain: 2
* DOMqueries: 7
* DOMqueriesById: 4
* DOMqueriesByClassName: 0
* DOMqueriesByTagName: 3
* DOMqueriesByQuerySelectorAll: 0
* DOMinserts: 4
* DOMqueriesDuplicated: 2
* eventsBound: 0
* headersCount: 368
* headersSentCount: 86
* headersRecvCount: 282
* headersSize: 12025
* headersSentSize: 2989
* headersRecvSize: 9036
* documentWriteCalls: 1
* evalCalls: 0
* jQueryVersion:
* jQueryOnDOMReadyFunctions: 0
* jQuerySizzleCalls: 0
* jsErrors: 0
* assetsNotGzipped: 5
* assetsWithQueryString: 4
* smallImages: 3
* multipleRequests: 0
* timeToFirstCss: 878
* timeToFirstJs: 1054
* timeToFirstImage: 1860
* onDOMReadyTime: 1339
* onDOMReadyTimeEnd: 1340
* windowOnLoadTime: 2386
* windowOnLoadTimeEnd: 2387
* httpTrafficCompleted: 3285
* windowAlerts: 0
* windowConfirms: 0
* windowPrompts: 0
* consoleMessages: 0
* cookiesSent: 0
* cookiesRecv: 354
* domainsWithCookies: 1
* documentCookiesLength: 193
* documentCookiesCount: 4
* bodyHTMLSize: 10809
* iframesCount: 0
* imagesWithoutDimensions: 16
* commentsSize: 53
* hiddenContentSize: 61
* whiteSpacesSize: 4
* DOMelementsCount: 197
* DOMelementMaxDepth: 10
* nodesWithInlineCSS: 0
* globalVariables: 57
* localStorageEntries: 0
* smallestResponse: 35
* biggestResponse: 29652
* fastestResponse: 27
* slowestResponse: 1057
* medianResponse: 517

First css received in 878 ms: <http://fonts.googleapis.com/css?family=Lato:400,700>
First js received in 1054 ms: <http://gruntjs.com/js/vendor/lib/modernizr.min.js>
First image received in 1860 ms: <http://gruntjs.com/img/grunt-logo.svg>
Requests per domain:
 gruntjs.com: 21 request(s)
 www.google-analytics.com: 2 request(s)
 static.adzerk.net: 2 request(s)
 engine.adzerk.net: 2 request(s)
 fonts.googleapis.com: 1 request(s)
 c.jslogger.com: 1 request(s)

Duplicated DOM queries (2):
 id "#azk48893": 4 queries
 tag name "script": 3 queries

JavaScript globals (57): JSLogger, Modernizr, _gaq, _gat, ados, adosResults, adosRun, ados_addInlinePlacement, ados_addPlacement, ados_addPlacementObject, ados_add_placement, ados_async_load, ados_execPassback, ados_findPassback, ados_frameLoaded, ados_load, ados_loadDiv, ados_loadFIframe, ados_loadInline, ados_loadPassback, ados_loadResults, ados_log, ados_passback, ados_passbackFilled, ados_passbackWritePixel, ados_passback_next, ados_passback_receiveMessage, ados_refresh, ados_setDomain, ados_setKeywordCookie, ados_setKeywords, ados_setNoTrack, ados_setWriteResults, ados_timeoutExpired, ados_writePixel, azHtmlLoad, azInitExtension, azLoad, azRegisterExtension, azScriptExtensionLoad, azScriptInlineLoad, azScriptSRCLoad, azk48893_html_98280, azk48893_html_command_17656, azk48893_pixel_64108, azk48893_pixel_command_60057, cssLinkLoad, cssLoad, d, gaGlobal, html5, jslogger, p, s, z, zItems, zshow

The smallest response (0.03 kB): <http://www.google-analytics.com/__utm.gif?utmwv=5.4.6&utms=1&utmn=265814781&utmhn=gruntjs.com&utmcs=UTF-8&utmsr=1920x1080&utmvp=1024x1280&utmsc=32-bit&utmul=de-de&utmje=0&utmfl=-&utmdt=Grunt%3A%20The%20JavaScript%20Task%20Runner&utmhid=319494337&utmr=-&utmp=%2F&utmht=1387753136010&utmac=UA-34623937-1&utmcc=__utma%3D221136078.1297690710.1387753136.1387753136.1387753136.1%3B%2B__utmz%3D221136078.1387753136.1.1.utmcsr%3D(direct)%7Cutmccn%3D(direct)%7Cutmcmd%3D(none)%3B&utmu=q~>
The biggest response (28.96 kB): <http://gruntjs.com/img/grunt-logo.svg>

The fastest response (27 ms): <http://www.google-analytics.com/__utm.gif?utmwv=5.4.6&utms=1&utmn=265814781&utmhn=gruntjs.com&utmcs=UTF-8&utmsr=1920x1080&utmvp=1024x1280&utmsc=32-bit&utmul=de-de&utmje=0&utmfl=-&utmdt=Grunt%3A%20The%20JavaScript%20Task%20Runner&utmhid=319494337&utmr=-&utmp=%2F&utmht=1387753136010&utmac=UA-34623937-1&utmcc=__utma%3D221136078.1297690710.1387753136.1387753136.1387753136.1%3B%2B__utmz%3D221136078.1387753136.1.1.utmcsr%3D(direct)%7Cutmccn%3D(direct)%7Cutmcmd%3D(none)%3B&utmu=q~>
The slowest response (1057 ms): <http://gruntjs.com/img/logo-bitovi.jpg>

Time spent on backend / frontend: 25% / 75%
```

When I saw these metrics for the first time I really had to dance for joy. They are by far the most powerful metrics I have seen. I thought about these metrics a bit more and descided that Phantomas is perfect for an automated process to keep track of all that after every deployment. Grunt is included in most of the projects I do, so it was not a big question how to create an automated task. That was the moment when [grunt-phantomas](https://npmjs.org/package/grunt-phantomas) was born and that is what it looks like today:

{% img left /images/blog/stefanjudis/grunt-phantomas.png 1892 676 'Example stats of grunt-phantomas' 'Example stats of grunt-phantomas' %}

All you have to do is to integrate the plugin in your Grunt process and then you are ready to go. You just have to define a folder, where the generated `index.html` and needed files will take place, as well as the wished url to get metrics for. Any more needed information can be found in the [official documentation](https://npmjs.org/package/grunt-phantomas).

Additional possible options are by now the following:

- number of runs -> Phantomas will be executed multiple times to make the data more reliable
- raw phantomas arguments -> Phantomas has lots of options - it is for example possible to block external scripts, what can become extremely handy when you have to deal with lots of marketing stuff (check [Phantomas Docs](https://github.com/macbre/phantomas) for more a complete list of options)

We integrated it into our CI system in my company already, so that I can really check all that data right after deployment and I'm really excited about that. In my opinion this can be an extremely valuable tool and any feedback on it is more than welcome.

And that is it. Thanks for reading and merry Christmas. :)

