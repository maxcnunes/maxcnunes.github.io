---
layout: post
title: Debug nodejs v0.10 with 100% cpu usage
tags:
- nodejs
---

Last week a worker at my job written in Nodejs v0.10 started to fail. At some point the worker would consume 100% of CPU and gradually would increase the memory usage as well. That job was been forced to stop once all the memory available was consumed.

Basically I had two approaches available to find out what was causing the problem.

1. Put `console.log` every place and find where in my code was the last time executed before getting stuck with 100% of CPU usage.
1. Do it right and profile the application to find out properly based in statistics about the running process how much CPU each part was consuming.

Even `console.log` been a common approach for many simple cases. The profile is the best option for this kind of problem.

So this is the flow I followed to find out the problem:

1. Start the app with enabled profile `node --prof index.js`.
1. In other shell window run `top` to monitor when the process get 100% CPU usage.
1. Once the process get in high CPU usage stop it.
1. The profile will have created `v8.log` file.
1. In node v0.10 has not a built in tool to process this file. Because of that install one with `npm -g install tick`.
1. Execute `node-tick-processor` and it will print the explained results about the profile log [v8-profile-cpu-100-usage.txt](https://gist.github.com/maxcnunes/66bb591c1f62f922254597353cd757b3).

Looking the **[Bottom up (heavy) profile]** section I could find out the module causing the 100% CPU usage.

```
[Bottom up (heavy) profile]:
Note: percentage shows a share of a particular caller in the total
amount of its parent calls.
Callers occupying less than 2.0% are not shown.

ticks parent  name
15924   71.2%  /opt/node-v0.10.31-linux-x64/bin/node
7742   48.6%    Function: ~derez /src/node_modules/cycle/cycle.js:46
7739  100.0%      Function: ~derez /src/node_modules/cycle/cycle.js:46
7739  100.0%        Function: ~derez /src/node_modules/cycle/cycle.js:46
7739  100.0%          Function: ~derez /src/node_modules/cycle/cycle.js:46
7739  100.0%            Function: ~derez /src/node_modules/cycle/cycle.js:46
```

As we can see the [cycle](https://github.com/dscape/cycle) is the guilty one. Cycle is a module used to encode and decode JSON data to Javascript. So we know already probably is a JS object been parsed to JSON string. And I would guess this JS object has circular reference and that is why is getting 100% CPU usage without stop until consume all the memory available.

But that was part of the problem. We know who is the responsible. But who is calling that function?

Looking into the **[Top down (heavy) profile]** I find out that [winston](https://github.com/winstonjs/winston) module is the one using the cycle. Winston is a library for logging. So are suspicion was right. Probably is a JS object been saved to the log. 

```
[Top down (heavy) profile]:
Note: callees occupying less than 0.1% are not shown.

inclusive      self           name
ticks   total  ticks   total
18799   84.0%      0    0.0%  LazyCompile: ~processImmediate timers.js:335
18767   83.9%      0    0.0%    LazyCompile: ~lib$rsvp$asap$$flush /src/node_modules/rsvp/dist/rsvp.js:1193
18747   83.8%      0    0.0%      LazyCompile: ~lib$rsvp$$internal$$publishRejection /src/node_modules/rsvp/dist/rsvp.js:957
18747   83.8%      0    0.0%        LazyCompile: ~lib$rsvp$$internal$$publish /src/node_modules/rsvp/dist/rsvp.js:1002
18747   83.8%      0    0.0%          LazyCompile: ~lib$rsvp$$internal$$invokeCallback /src/node_modules/rsvp/dist/rsvp.js:1043
18747   83.8%      0    0.0%            LazyCompile: lib$rsvp$$internal$$tryCatch /src/node_modules/rsvp/dist/rsvp.js:1034
18746   83.8%      0    0.0%              LazyCompile: ~<anonymous> /src/lib/promised-queue.js:29
18736   83.8%      0    0.0%                LazyCompile: ~logError /src/lib/promised-queue.js:80
18736   83.8%      0    0.0%                  LazyCompile: ~target.(anonymous function) /src/node_modules/winston/lib/winston/common.js:41
18736   83.8%      0    0.0%                    LazyCompile: ~Logger.log /src/node_modules/winston/lib/winston/logger.js:123
18736   83.8%      0    0.0%                      LazyCompile: ~async.each /src/node_modules/winston/node_modules/async/lib/async.js:104
18736   83.8%      0    0.0%                        LazyCompile: ~_each /src/node_modules/winston/node_modules/async/lib/async.js:30
18736   83.8%      0    0.0%                          LazyCompile: *forEach native array.js:1087
18736   83.8%      0    0.0%                            LazyCompile: ~<anonymous> /src/node_modules/winston/node_modules/async/lib/async.js:110
18736   83.8%      0    0.0%                              LazyCompile: ~emit /src/node_modules/winston/lib/winston/logger.js:175
18736   83.8%      0    0.0%                                LazyCompile: ~Console.log /src/node_modules/winston/lib/winston/transports/console.js:56
18736   83.8%      0    0.0%                                  LazyCompile: ~exports.log /src/node_modules/winston/lib/winston/common.js:112
18736   83.8%      0    0.0%                                    LazyCompile: ~decycle /src/node_modules/cycle/cycle.js:24
18736   83.8%      0    0.0%                                      Function: ~derez /src/node_modules/cycle/cycle.js:46
18736   83.8%      0    0.0%                                        Function: ~derez /src/node_modules/cycle/cycle.js:46
18736   83.8%      0    0.0%                                          Function: ~derez /src/node_modules/cycle/cycle.js:46
18736   83.8%      0    0.0%                                            Function: ~derez /src/node_modules/cycle/cycle.js:46
18736   83.8%      0    0.0%                                              Function: ~derez /src/node_modules/cycle/cycle.js:46
18736   83.8%      0    0.0%                                                Function: ~derez /src/node_modules/cycle/cycle.js:46
18736   83.8%      0    0.0%                                                  Function: ~derez /src/node_modules/cycle/cycle.js:46
18551   82.9%    998    4.5%                                                    Function: ~derez /src/node_modules/cycle/cycle.js:46
4754   21.3%   4754   21.3%                                                      /opt/node-v0.10.31-linux-x64/bin/node
3805   17.0%    296    1.3%                                                      Function: ~derez /src/node_modules/cycle/cycle.js:46
```

Looking above the winston tick the last part of my code before calling the winston lib is:

    LazyCompile: ~logError /src/lib/promised-queue.js:80

And that is the end. Looking into my code `/src/lib/promised-queue.js` at line `80`. The function `logError` really was calling the `winston` library. Based on this knowledge my approach was simplest replacing `winston` with `console.log` because at that part a simple print of the error into the console was enough.
