---
layout: post
title: "See how your project performs at Travis CI"
date: 2013-07-07 14:50
author : stefanjudis
comments: true
categories:
- travis
- data analysis
- open source
- performance
---
Many people run their test suite at Travis CI these days. At Travis you got the possibility to run your test suite against different programming languages and different versions of these ( e.g. node 0.8 / 0.9 / … ). Figuring out if there will be any problems or exceptions before switching to a new language version are easily done this way ( always assuming you are following the principles of test driven development ). **Test your stuff at Travis first, then switch your server to the new version of language XY**.

<!-- more -->

Tests that run in different languages are called 'jobs' in travis context. You can define them in your .travis.yml.

```
language: node_js
  node_js:
    - "0.8"
    - "0.9"
    - "0.10"
```

Later on you can see this configuration inside of the so called build-matrix at [Travis](http://travis-ci.org).

{% img center /images/blog/stefanjudis/buildMatrix.png 1225 184 'build matrix cushion-cli' 'build matrix for my project cushion-cli' %}

Via Travalizit ( [further explanation and introduction](http://4waisenkinder.de/blog/2013/06/21/help-me-analyze-travis-build-data/) ) you got the possibility to analyse your Travis build data. It is still in early stage, but I think you can already get some information, you did not know before. To have always the latest Travis information at Travalizit you only have to define a wekhook that sends the build data to Travalizit each time you finished a build ( checkout out explanation and introduction ).

Yesterday night I implemented a new chart to compare different builds with including jobs in term of state and duration. With the help of this chart you can easily check if a new language version will increase performance or not.

To check your project just go to [Travalizit](http://travalizit.org), add a new chart and choose type "Travis job details".

{% img center /images/blog/stefanjudis/jobDetails.png 603 377 'add job details chart action' 'push the button ;)' %}

After that you will probably get a message, that your build data is not included in the Travalizit database ( there is the small chance that it is, because I tested a lot ). Right now you got the possibility to fetch the latest builds from Travis. 

{% img center /images/blog/stefanjudis/fetchData.png 867 655 'fetch data action' 'fetch data dialog if no build data is available' %}

Press the 'fetch' - button and you are done. This works only **once**, because the architecture is actual set up to receive recent build data via webhook. That means, if you want to come back and to see the latest builds, you have implement the hook. ;)

And that is the result:

{% img center /images/blog/stefanjudis/jobDetailsChart.png 882 670 'job details for cushion-cli' 'Travis job details chart for cushion-cli' %}
 
Right now you can check the duration and state ( passed / errored / failed ) of the different jobs. If you want to see more, let me know.

I would love to hear wishes, improvements and feedback. So, if it is not working for you and your project or you have something to say about that ping me on [Twitter](https://twitter.com/stefanjudis), write me an [email](mailto:stefanjudis@gmail.com) or leave a comment.

THX. 

PS. I started developing a [Nodes.js Travis-Api](https://npmjs.org/package/travalizit) for this project. If someone is interested in that, give a sign. That keeps motivation high. ;)