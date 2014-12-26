---
layout: post
title: "NPM 2.0 and how it helps avoiding global dependencies"
date: 2014-10-18 20:50
comments: true
author : stefanjudis
categories:
- node
- npm
- grunt
- gulp
---

Today I was listening to the NodeUp [episode 70](http://nodeup.com/seventy), which is all about the [npm command line client](https://www.npmjs.org/package/npm). And there is tons of useful information in this episode. It is all about where npm is at the moment and what the plans are for the future. Especially the recent changes inside of the command line client are a heavy discussed topic and I highly recommend to listen to this episode, when you are dealing with [npm](https://www.npmjs.org/) on a daily basis.

One thing that is mentioned and really gets me excited is the change regarding the functionality to run scripts via npm which was introduced in the latest major version of npm - [npm@2.0.0](http://blog.npmjs.org/post/98131109725/npm-2-0-0).<!-- more -->

So let us reassess how to run scripts via npm, have a look at what has changed in version 2.0.0 and check why this is such a big deal.

### Running scripts via npm

The configuration file for any project based on node and npm is the `package.json`. This file includes meta information like name, version and author of the depending project, but also defines all dependencies, which need to be installed via calling `npm install`. If you are not familiar with this file, there is an excellent [interactive cheat sheet](http://browsenpm.org/package.json) out there and you may want to check it out.

One thing to notice is that you can also run scripts and execute commands via npm. To do so you can define an optional object as the `scripts` property inside of the `package.json` and define your wished commands. [@substack](https://twitter.com/substack) wrote a [great article](http://substack.net/task_automation_with_npm_run) about how to use this functionality extensively.

There is not much magic about this.

```json
{
  "name": "blog-npm-run-scripts",
  "version": "1.0.0",
  "description": "Show of the new npm run command",
  "scripts": {
    "echo" : "echo \"Hello world\""
  }
}
```

And then you can use `npm run` to kick it off - pretty straight forward.

```bash
> npm run echo

> blog-npm-run-scripts@1.0.0 echo /Users/stefan/Sites/blog-npm-run-scripts
> echo "Hello world"

Hello world
```

This functionality had one downside so far. It was not able to pass arguments to the `npm run` command. And that is why you had to hardcode the arguments, which made the whole thing less flexible and harder to use. The only solution for having similar commands with different arguments was to define specific named scripts inside of the `package.json` including different arguments.

```json
{
  "name": "blog-npm-run-scripts",
  "version": "1.0.0",
  "description": "Show of the new npm run command",
  "scripts": {
    "echo_helloWorld" : "echo \"Hello world\"",
    "echo_foo" : "echo \"Foo\""
  }
}
```

### Passing arguments to `npm run`

Since version 2.0.0 it is now possible to pass arguments to the scripts defined in the `package.json`. And this is a big improvement on flexibility and makes the whole thing much more powerful. The `package.json` above including two scripts running the `echo` command can be combined into one and can accept the wished arguments.

```json
{
  "name": "blog-npm-run-scripts",
  "version": "1.0.0",
  "description": "Show of the new npm run command",
  "scripts": {
    "echo" : "echo"
  }
}
```

The syntax to pass arguments to the defined scripts is as follows. You have to use `npm run` and then devided by two dashes(`--`) you can pass any arguments you like to the command.

```bash
> npm run echo -- "hello world"

> blog-npm-run-scripts@1.0.0 echo /Users/stefan/Sites/blog-npm-run-scripts
> echo "hello world"

hello world
```

### Setting up Grunt and gulp without the global dependency

Using the `echo` command might not seem really useful, but we will come to a much more useful example now. I am doing mostly frontend development and that is why in almost every project that I work on either [Grunt](http://gruntjs.com/) or [gulp](http://gulpjs.com/) is included. Grunt and gulp are task runners, that come with huge plugin registries to help automate any task you can think of.

When you check the getting started guide of both projects you will find the instruction to install them globally.

```bash
# install gulp globally
$ npm install -g gulp
# install grunt globally
$ npm install -g grunt-cli
```

This is absolutely fine when you are working alone and these tools are only supposed to be executed on your machine. But when you work together with other colleagues on a project or your process includes a [continuous integration](http://www.martinfowler.com/articles/continuousIntegration.html) system, then every global dependency can be quite troublesome. It simply moves the entry barrier a bit higher and increases the complexity to get everything up and running.

So let us have a look how to avoid that. First step is to install the needed modules in our project and not globally anymore.

```
# install gulp in the project
$ npm install gulp
# install grunt in the project
$ npm install grunt-cli
```

By calling `npm install` npm will install the module and depending, if it has the [`bin`](http://browsenpm.org/package.json#bin) property defined, it will create a `.bin` folder inside of the `node_modules` folder. This means that this folder will include all defined command line interfaces of your installed modules. In this case the `.bin` folder includes the binaries `gulp` and `grunt`.

```
node_modules
  |_  .bin
      |_ gulp
      |_ grunt
```

If you want to use either Grunt or gulp via the `npm run` command now, you can set them up inside of your `package.json`.

```json
{
  "name": "blog-npm-run-scripts",
  "version": "1.0.0",
  "description": "Show of the new npm run command",
  "scripts": {
    "gulp" : "./node_modules/.bin/gulp",
    "grunt" : "./node_modules/.bin/grunt"
  }
}
```

And then you can easily run your defined tasks with npm.

```bash
# run 'dev' task with in project install grunt
$ npm run grunt -- dev
# run 'dev' task with in project install gulp
$ npm run gulp -- dev
```

**But wait, it comes even better!**

To make it a bit nicer npm provides a nifty feature, when setting up custom scripts. It puts `./node_modules/.bin` in the `PATH` environment, when it executes the script.

This means, we can make the `package.json` a bit cleaner.

```json
{
  "name": "blog-npm-run-scripts",
  "version": "1.0.0",
  "description": "Show of the new npm run command",
  "scripts": {
    "gulp" : "gulp",
    "grunt" : "grunt"
  }
}
```

**For me this is pure awesomeness!**

It means not only dropping a global dependency but rather simplifying the whole work and setup flow.

Getting everything up and running is not

- installing node (which will install npm also)
- installing dependencies
- installing global dependencies
- and run e.g. Grunt

anymore.

It becomes

- installing node
- installing dependencies
- and run everything via npm scripts

only.

If you want to play around with this I set up an [example repository](https://github.com/stefanjudis/npm2-run-scripts-tryout), which includes Grunt and gulp ready to use without any global installation.

### Sum up

For me it is clear, that I will drop any project required global dependency that can be installed via npm in the future, because having less global dependencies just means less troubles and quicker setup.

And that is it for now and if you have any comments or ideas on that, please let me know. I hope you enjoyed it. :)
























