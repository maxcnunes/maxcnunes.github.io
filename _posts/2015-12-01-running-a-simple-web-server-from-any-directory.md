---
layout: post
title: Running a simple web server from any directory
tags:
- mac
- web server
---

You just need [Python](https://www.python.org/) installed to run a simple web server from any directory.
On Mac Python is already installed. So you just need follow these steps below:

Run these commands in the terminal:

```bash
# access the directory
cd /path/to/directory

# start the web server in the port 8000
python -m SimpleHTTPServer 8000

# execute Ctrl+C to kill the web server
```

Open the browser and access the web server through http://localhost:8000.

### Demo

<img src="/assets/running-a-simple-web-server-from-any-directory.gif" style="width: 100%" />
<a href="/assets/running-a-simple-web-server-from-any-directory.gif">See the demo in the original scale</a>
