---
layout: post
title: Suspend vagrant before Mac sleep
---

Lately I was getting a lot of problems with my VM and even with the bluetooth on Mac.
Just because the vmware VM was not recovering well after my macbook waking up.

I use a VM everyday and I always forget to suspend/halt that before sleep my macbook.
So I find out is possible to configure scripts to be executed on wakeup/sleep the machine through [SleepWatcher](http://www.bernhard-baehr.de/).

This blog post [Mac OS X: Automating Tasks on Sleep](http://www.kodiakskorner.com/log/258/comment-page-1#comment-6777) by Kodiak explain really well how to setup the sleepwatcher.

Summarizing:

```bash
# 1. install
brew install sleepwatcher

# 2. create a script "~/.sleep" to be executed before the machine sleep
# in my case was "cd <vm-directory> && vagrant suspend"

# 3. set full permissions to the owner
chmod 700 ~/.sleep

# 4. start the watcher then put your machine to sleep to test the script is working
/usr/local/sbin/sleepwatcher --verbose --sleep ~/.sleep

# configure the deamon
sudo chown root:wheel /usr/local/Cellar/sleepwatcher/2.2/de.bernhard-baehr.sleepwatcher-20compatibility-localuser.plist

ln -sfv /usr/local/Cellar/sleepwatcher/2.2/de.bernhard-baehr.sleepwatcher-20compatibility-localuser.plist ~/Library/LaunchAgents/

launchctl load ~/Library/LaunchAgents/de.bernhard-baehr.sleepwatcher-20compatibility-localuser.plist
```
