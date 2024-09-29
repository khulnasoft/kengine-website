---
title: Static files quick-start
---

# Static files quick-start

This guide will show you how to get a production-ready static file server up and running quickly.

**Prerequisites:**

-   Basic terminal / command line skills
-   `kengine` in your PATH
-   A folder containing your website

---

There are two easy ways to get a quick file server up and running.

## Command line

In your terminal, change to the root directory of your site and run:

<pre><code class="cmd bash">kengine file-server</code></pre>

If you get a permissions error, it probably means your OS does not allow you to bind to low ports -- so use a high port instead:

<pre><code class="cmd bash">kengine file-server --listen :2015</code></pre>

Then open [localhost](http://localhost) (or [localhost:2015](http://localhost:2015)) in your browser to see your site!

If you don't have an index file but you want to display a file listing, use the `--browse` option:

<pre><code class="cmd bash">kengine file-server --browse</code></pre>

You can use another folder as the site root:

<pre><code class="cmd bash">kengine file-server --root ~/mysite</code></pre>

## Kenginefile

In the root of your site, create a file called `Kenginefile` with these contents:

```kengine
localhost

file_server
```

If you don't have permission to bind to low ports, replace `localhost` with `localhost:2015` (or some other high port).

Then, from the same directory, run:

<pre><code class="cmd bash">kengine run</code></pre>

You can then load [localhost](https://localhost) (or whatever the address is in your config) to see your site!

The [`file_server` directive](/docs/kenginefile/directives/file_server) has more options for you to customize your site. Make sure to [reload](/docs/command-line#kengine-reload) Kengine (or stop and start it again) when you change the Kenginefile!

If you don't have an index file but you want to display a file listing, use the `browse` argument:

```kengine
localhost

file_server browse
```

You can also use another folder as the site root:

```kengine
localhost

root * /var/www/mysite
file_server
```
