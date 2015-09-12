---
layout: post
title: Finding out to which docker container a process belongs to
categories:
- Docker
tags:
- docker
- linux
---

Linux containers run all over the same Linux Kernel. It means all process running inside each container will be visible in the host machine. Due that, how do we know if a process is running in the host or in a container? And how do we discover in which container that process is running? Lets solve that based in a real example.

This week I was running some tests in one of the Bravi's server and I realized there was a weird grunt process using 30% of CPU. That was weird because should not exists a grunt process running in a staging/production environment and neither should be using so much from the CPU.

```bash
top
```

![](/assets/top-grunt.png)

Running this command `ps -axfo pid,uname,cmd` I got the full list of process running in that machine. In this list I could find that process with id equals to `2973` and its top parent process.

![](/assets/grunt-parent-process.png)

With the top process id I could finally find the docker container running that grunt process.

![](/assets/grunt-docker-container.png)
