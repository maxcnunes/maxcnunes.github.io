---
layout: post
title: Docker Build Error
categories:
- Docker
tags:
- Docker
- Dockerfile
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  _headspace_page_title: Docker Build Error
  _headspace_description: 'Fix docker error: no such file or directory - The command [] returned a non-zero code: 1'
author:
  login: maxcnunes
  email: maxcnunes@gmail.com
  display_name: maxcnunes
  first_name: ''
  last_name: ''
---

I was playing around Docker this week using this Dockerfile to build my container:



{% gist maxcnunes/9956342 Dockerfile %}

But during the build I was getting this error message:

{% gist maxcnunes/9956342 docker_build_error.sh %}


I was using a Linux Mint 16 and after some research to solve the problem I figured out was missing install the [LXC](https://linuxcontainers.org/).

```bash
sudo apt-get install lxc
```
