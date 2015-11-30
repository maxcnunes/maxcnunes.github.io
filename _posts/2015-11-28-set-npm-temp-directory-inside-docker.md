---
layout: post
title: Set temp directory for NPM scripts running with root user (Docker)
categories:
- docker
- nodejs
tags:
- docker
- nodejs
---

Running scripts through `npm` with root user will not respect the `TMPDIR` or any other environment variable used to set the temp directory.

Even this looking like a bug. Actually it is a expected behavior as mentioned in this [issue](https://github.com/npm/npm/issues/4531).

So the solution is enable the `unsafe-perm` flag in the NPM configuration. Since
doing this for the global NPM configuration or even in the project scope is quite
unsafe because would run anything with root permissions. I think the safest solution
is running those commands that depend on the temp directory with the `--unsafe-perm`
argument.

```bash
npm run --unsafe-perm <my-npm-script>
```
