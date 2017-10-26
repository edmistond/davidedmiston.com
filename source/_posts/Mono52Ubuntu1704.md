---
title: Installing Mono 5.2 on Ubuntu 17.04
date: 2017-10-04 20:30
tags: 
  - linux
  - ubuntu
  - mono
  - .net
---

Just a quick post to fill a gap on the Internet that was frustrating me, and which hopefully someone will find helpful as well. I don't customarily work in Linux these days, but needed to set up a small Ubuntu 17.04 VM for some stuff I'm testing with F#. Since I wanted that fully working, I was trying to install Mono based on the [download page instructions](http://www.mono-project.com/download/#download-lin) and was dismayed to find there were no instructions there for Ubuntu 17.04, and extensive Googling didn't turn up anything.

After asking briefly in the Mono gitter.im chat - where, I'll add, folks were super helpful with a slightly stupid question - they informed me that everything past 16.04 should, currently, be able to use the 16.04 repositories and install instructions. I tested this out, they were right.

So, if you've been looking all over the place trying to figure that one out... there you go. Note that this may or may not change when Ubuntu 17.10 drops later this month.
