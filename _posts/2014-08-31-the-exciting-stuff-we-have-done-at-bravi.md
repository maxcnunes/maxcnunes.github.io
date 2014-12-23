---
layout: post
title: The exciting stuff we have done at Bravi
categories:
- Js
tags:
- Microservices
- Docker
- Bravo
- Git Deploy
- Logs everywhere
- Performance tuning
- Fancy prompt
- Indexed search
- React
- Webpack
---

Bravi has many projects and each one of them uses a proper language and technology for that specific scenario. The team I am currently working has done some pretty exciting stuff and now that the project has calm down (for few days) I think is the right time to share these things we have worked on.

The idea right now is not getting deep in each subject, just share the tools, patterns, ideas and other stuff we have used. Maybe  in a next post we could  explaining better each one.


### Microservices

> [**Microservices**](http://martinfowler.com/articles/microservices.html) are a style of software architecture that involves delivering systems as a set of very small, granular, independent collaborating services.

This approach came with a lot of questions and forced us to think deeply over many decisions (architectural and infrastructural).

However for this project was a pretty good decision use Microservices and we do not regret about that. Nowadays we have around 15 services and using Microservices in a scenario like this gives a lot of advantages isolating the services by responsibilities.

### Containers

As we are working with microservices some times a service can use a different language, technology and environment configuration from others. For this reason we needed to isolate each service in a physical level instead of only an project level. We could do that using a virtual machine, as we were used to do in previous projects. But this approach has high cost of memory and process usage. Then we  thought was time to try something more efficient as the Linux Containers with Docker.

[**Docker**](https://www.docker.com/) is an open-source application container engine and has been used for many big companies.

```bash
docker run -i -t ubuntu /bin/bash
```


### Bravo tool

Microservices brings up some concerns to an architectural level. For example how to run related projects, connect them allowing communicate with each other. Is possible to settup it using only docker (for simple scenarios). But to make this process easier we have built a tool to help us doing it.

**Bravo** is written in nodejs and basically reads an yaml configuration of the current project, where is specified how to run the service. For this it uses docker, nginx and dnsmasq (only in development environment).

```bash
bravo run -e development
```


### Git Deploy

To allow an push button deployment using the Bravo tool we also have built another tool **Git Deploy**.

![](/assets/gitdeployer-01.png)
![](/assets/gitdeployer-02.png)

### Logs everywhere

We used [winston](https://github.com/flatiron/winston) library to easily save logging in many ways (console, text and slack). Receive logs in channels on [slack](https://slack.com/) is pretty helpful. We are notified if something is going wrong right when this is happening and if you are not in the slack at the moment we are notified also in our email.

![](/assets/slack-logging.png)


### Fancy prompt

You can get lost using bashes everywhere over many containers. Therefore we have styled our prompts to show some important informations:

1. `host` - the container's host name
2. `env` -  in which environment the container is running (development, staging, production)
3. `git` - the current git branch
4. path of the current directory
5. a big `PRODUCTION` warning if the current environment is production to make you think twice before do anything

![](/assets/fancy-prompt.png)


### Performance tuning

One of ours main concern in this application was to support at least 100 concurrent requests per second. To measure it we used the [Apache AB tool](http://httpd.apache.org/docs/2.2/programs/ab.html). In our first test I think the application was supporting less than 20 requests per second. Basically we did:

1. Optimized linux networking
2. Optimized nodejs
3. Optimized nginx

At some point we thought we were having a low performance due we were using docker. But we were wrong and we didn't have to change anything related to docker configuration.


### Indexed search

This application has many content texts. So we needed a good way to search words an sentences in long texts. This approach should be fast and works ignoring case sensitive and accents. To solve these requirements we used the [Apache Solr](http://lucene.apache.org/solr/).


### React

For some front-end components were used the [**React**](http://facebook.github.io/react/).

> React is, in my opinion, the premier way to build big, fast Web apps with JavaScript. It's scaled very well for us at Facebook and Instagram.
> One of the many great parts of React is how it makes you think about apps as you build them. ([*reference*](http://facebook.github.io/react/blog/2013/11/05/thinking-in-react.html))

### Webpack

> [**webpack**](https://github.com/webpack/webpack) is a bundler for modules. The main purpose is to bundle javascript files for usage in a browser.

This allowed use move some front-end code to isolated components and use them as widgets on demand over different projects.