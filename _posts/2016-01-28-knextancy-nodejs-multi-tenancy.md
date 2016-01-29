---
layout: post
title: Knextancy - Node JS Multi Tenancy
tags:
- nodejs
---

At Bravi every project has multi tenant support since the beginning of the development.
This because we have built a solution that is so easy to use that is almost imperceptible we are developing a multi tenant application. The only thing we have to care about is:

* pass ahead in the HTTP header the tenant id for every consumed API/application
* use a prefix in the table on executing SQL queries `select * from "$_mytable"`

And that is it. You are already building a multi tenant application.

## How does that work?

Well. The library [knextancy](https://github.com/bravi-software/knextancy) takes care of everything for us.

Knextancy is built on top of [knex](knexjs.org) and implements multi tenant using table prefixes. So all the clients share the same database but each one has it own table. For example:

![](https://raw.githubusercontent.com/bravi-software/knextancy-example/master/screeshot.png)

Every time a request like this one below is done:

```bash
curl -X GET -H "x-client-id: 11" "http://localhost:3000/"
```

Knextancy will identify the tenant associated to that request based in the `x-client-id`. In case is the first request for that server, knextancy also will execute the migration and seed tasks. So there is no manual configuration required for bringing up a database for a new client.

> You may ask yourself. How is that secure if the tenant is based in HTTP header? Then the user could change manually the tenant he is accessing.
>
> Well. This is not a problem for services running inside a private network. For the public app/services, we run everything behind a proxy, that injects the x-client-id based in the domain.

After that knextancy injects in the request an instance of [knex](knexjs.org) with the context of the current client (e.g. `req.knex`).

Then to execute any SQL query with multi tenant support is just required to use the `req.knex` and any table must have prefix `$_` to allow knextancy resolve the table for the current client.

> Simple example of how to build a multi tenant REST API. Using knextancy with express and PostgreSQL database.
>  https://github.com/bravi-software/knextancy-example
