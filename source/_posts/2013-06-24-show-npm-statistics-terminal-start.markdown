---
layout: post
title: "Show npm statistics at terminal start"
date: 2013-06-24 22:21
author : stefanjudis
comments: true
categories:
- npm
- cli
- terminal
---

Two month ago I released my [first npm module](https://npmjs.org/package/cushion-cli). I got really excited (and I am still) about releasing something and showing it to the world. Since then the statistics written on top of the module page at NPM are checked by me on a daily basis. Unfortunately these statistics do not seem to be really precise. They are jumping from day to day and should not be taken to serious in my mind (No offense meant. I really like NPM and I think it is a great service).

That is the reason why a friend of mine wrote a [tiny script](https://raw.github.com/Zoddy/geektool-desk/master/os_downloads.js) to read the statistics directly from where the data is stored - a [CouchDB](http://isaacs.iriscouch.com/) available for everyone. The script will be part of a [library for geeky stuff](https://github.com/Zoddy/geektool-desk) and tools later on, but that is another story.

Today I had the idea about showing this kind of information whenever I open a new terminal window to save some time and "pimp my terminal".

<!-- more -->

I googled a bit and had to notice that it is not to easy to execute scripts on MacOS in the ```/etc/motd``` (message of the day). It seems to be much easier in the linux world - correct me when I'm wrong please. ;)

So I decided to execute the whole stuff inside of the shell configuration files placed in the given user root (information about NPM modules are probably user related, so after a bit of thinking I like that solution even more).

To start you have to clone the Github project with the mentioned script (in my case ```~/Sites/```).

```
git clone https://github.com/Zoddy/geektool-desk.git
```

After that we start editing the .bash_profile, .bashrc, .zshrc or whatever shell file your system and shell gives you.

I prefer using ZSH. Adding the following lines does all the magic for me - look at this [readme](https://github.com/Zoddy/geektool-desk) for further explanation how to use the script. This example is made for my 'cushion-cli' project. I think you are not interested in this data, so please feel free to change it to your project names.

```
### show cushion-cli downloads

echo "\nCUSHION-CLI DOWNLOADS\n"

echo "TODAY / YESTERDAY:"
today=$(node /Users/stefan/Sites/geektool-desk/os_downloads.js cushion-cli today)
yesterday=$(node /Users/stefan/Sites/geektool-desk/os_downloads.js cushion-cli yesterday)
printf "%s / %s downloads\n\n" "$today" "$yesterday"

echo 'CURRENT WEEK / LAST WEEK:'
week=$(node /Users/stefan/Sites/geektool-desk/os_downloads.js cushion-cli current-week)
last_week=$(node /Users/stefan/Sites/geektool-desk/os_downloads.js cushion-cli last-week)
printf "%s / %s downloads\n\n" "$week" "$last_week"

echo 'CURRENT MONTH / LAST MONTH:'
month=$(node /Users/stefan/Sites/geektool-desk/os_downloads.js cushion-cli current-month)
last_month=$(node /Users/stefan/Sites/geektool-desk/os_downloads.js cushion-cli last-month)
printf "%s / %s downloads\n\n" "$month" "$last_month"
```

I am not a shell script wizard. If something is not correct with these lines let me know. 

And that is the result. Hope you enjoy it.

{% img right /images/blog/stefanjudis/npmPrompt.png 254 233 'prompt with npm statistic' 'prompt with npm statistic for cushion-cli' %}


