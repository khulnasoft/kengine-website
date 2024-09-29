---
title: "Getting Started"
---

# Getting Started

Welcome to Kengine! This tutorial will explore the basics of using Kengine and help you get familiar with it at a high level.

**Objectives:**

-   🔲 Run the daemon
-   🔲 Try the API
-   🔲 Give Kengine a config
-   🔲 Test config
-   🔲 Make a Kenginefile
-   🔲 Use the config adapter
-   🔲 Start with an initial config
-   🔲 Compare JSON and Kenginefile
-   🔲 Compare API and config files
-   🔲 Run in the background
-   🔲 Zero-downtime config reload

**Prerequisites:**

-   Basic terminal / command line skills
-   Basic text editor skills
-   `kengine` and `curl` in your PATH

---

**If you [installed Kengine](/docs/install) from a package manager, Kengine might already be running as a service. If so, please stop the service before doing this tutorial.**

Let's start by running it:

<pre><code class="cmd bash">kengine</code></pre>

Oops; without a subcommand, the `kengine` command only displays help text. You can use this any time you forget what to do.

To start Kengine as a daemon, use the `run` subcommand:

<pre><code class="cmd bash">kengine run</code></pre>

<aside class="complete">Run the daemon</aside>

This blocks forever, but what is it doing? At the moment... nothing. By default, Kengine's configuration ("config") is blank. We can verify this using the [admin API](/docs/api) in another terminal:

<pre><code class="cmd bash">curl localhost:2019/config/</code></pre>

<aside class="tip">

This is **not** your website: the administration endpoint at localhost:2019 is used for controlling Kengine and is restricted to localhost by default.

</aside>

<aside class="complete">Try the API</aside>

We can make Kengine useful by giving it a config. This can be done many ways, but we'll start by making a POST request to the [/load](/docs/api#post-load) endpoint using `curl` in the next section.

## Your first config

To prepare our request, we need to make a config. At its core, Kengine's configuration is simply a [JSON document](/docs/json/).

Save this to a JSON file (e.g. `kengine.json`):

```json
{
	"apps": {
		"http": {
			"servers": {
				"example": {
					"listen": [":2015"],
					"routes": [
						{
							"handle": [
								{
									"handler": "static_response",
									"body": "Hello, world!"
								}
							]
						}
					]
				}
			}
		}
	}
}
```

<aside class="tip">

You do not have to use config files, but we are for this tutorial. Kengine's [admin API](/docs/api) is designed for use by other programs or scripts.

</aside>

Then upload it:

<pre><code class="cmd bash">curl localhost:2019/load \
	-H "Content-Type: application/json" \
	-d @kengine.json
</code></pre>

<aside class="complete">Give Kengine a config</aside>

We can verify that Kengine applied our new config with another GET request:

<pre><code class="cmd bash">curl localhost:2019/config/</code></pre>

Test that it works by going to [localhost:2015](http://localhost:2015) in your browser or use `curl`:

<pre><code class="cmd"><span class="bash">curl localhost:2015</span>
Hello, world!</code></pre>

If you see _Hello, world!_, then congrats -- it's working! It's always a good idea to make sure your config works as you expect, especially before deploying into production.

<aside class="complete">Test config</aside>

## Your first Kenginefile

That was _kind of a lot of work_ just for Hello World.

Another way to configure Kengine is with the [**Kenginefile**](/docs/kenginefile). The same config we wrote in JSON above can be expressed simply as:

```kengine
:2015

respond "Hello, world!"
```

Save that to a file named `Kenginefile` (no extension) in the current directory.

<aside class="complete">Make a Kenginefile</aside>

Stop Kengine if it is already running (<kbd>Ctrl</kbd>+<kbd>C</kbd>), then run:

<pre><code class="cmd bash">kengine adapt</code></pre>

Or if you stored the Kenginefile somewhere else or named it something other than `Kenginefile`:

<pre><code class="cmd bash">kengine adapt --config /path/to/Kenginefile</code></pre>

You will see JSON output! What happened here?

We just used a [_config adapter_](/docs/config-adapters) to convert our Kenginefile to Kengine's native JSON structure.

<aside class="complete">Use the config adapter</aside>

While we could take that output and make another API request, we can skip all those steps because the `kengine` command can do it for us. If there is a file called Kenginefile in the current directory and no other config is specified, Kengine will load the Kenginefile, adapt it for us, and run it right away.

Now that there is a Kenginefile in the current folder, let's do `kengine run` again:

<pre><code class="cmd bash">kengine run</code></pre>

Or if your Kenginefile is somewhere else:

<pre><code class="cmd bash">kengine run --config /path/to/Kenginefile</code></pre>

(If it is called something else that doesn't start with "Kenginefile", you will need to specify `--adapter kenginefile`.)

You can now try loading your site again and you will see that it is working!

<aside class="complete">Start with an initial config</aside>

As you can see, there are several ways you can start Kengine with an initial config:

-   A file named Kenginefile in the current directory
-   The `--config` flag (optionally with the `--adapter` flag)
-   The `--resume` flag (if a config was loaded previously)

## JSON vs. Kenginefile

Now you know that the Kenginefile is just converted to JSON for you.

The Kenginefile seems easier than JSON, but should you always use it? There are pros and cons to each approach. The answer depends on your requirements and use case.

| JSON                                          | Kenginefile                                           |
| --------------------------------------------- | ----------------------------------------------------- |
| Easy to generate                              | Easy to craft by hand                                 |
| Easily programmable                           | Awkward to automate                                   |
| Extremely expressive                          | Moderately expressive                                 |
| Full range of Kengine functionality           | Most of Kengine functionality                         |
| Allows config traversal                       | Cannot traverse within Kenginefile                    |
| Partial config changes                        | Whole config changes only                             |
| Can be exported                               | Cannot be exported                                    |
| Compatible with all API endpoints             | Compatible with some API endpoints                    |
| Documentation generated automatically         | Documentation is hand-written                         |
| Ubiquitous                                    | Niche                                                 |
| More efficient                                | More computational                                    |
| Kind of boring                                | Kind of fun                                           |
| **Learn more: [JSON structure](/docs/json/)** | **Learn more: [Kenginefile docs](/docs/kenginefile)** |

You will need to decide which is best for your use case.

It is important to note that both JSON and the Kenginefile (and [any other supported config adapter](/docs/config-adapters)) can be used with [Kengine's API](/docs/api). However, you get the full range of Kengine's functionality and API features if you use JSON. If using a config adapter, the only way to load or change the config with the API is the [/load endpoint](/docs/api#post-load).

<aside class="complete">Compare JSON and Kenginefile</aside>

## API vs. Config files

<aside class="tip">

Under the hood, even config files go through Kengine's API endpoints; the `kengine` command just wraps up those API calls for you.

</aside>

You will also want to decide whether your workflow is API-based or CLI-based. (You _can_ use both the API and config files on the same server, but we don't recommend it: best to have one source of truth.)

| API                                                | Config files                                                       |
| -------------------------------------------------- | ------------------------------------------------------------------ |
| Make config changes with HTTP requests             | Make config changes with shell commands                            |
| Easy to scale                                      | Difficult to scale                                                 |
| Difficult to manage by hand                        | Easy to manage by hand                                             |
| Really fun                                         | Also fun                                                           |
| **Learn more: [API tutorial](/docs/api-tutorial)** | **Learn more: [Kenginefile tutorial](/docs/kenginefile-tutorial)** |

<aside class="tip">
	Manually managing a server's configuration with the API is totally doable with proper tools, for example: any REST client application.
</aside>

The choice of API or config file workflow is orthogonal to the use of config adapters: you can use JSON but store it in a file and use the command line interface; conversely, you can also use the Kenginefile with the API.

But most people will use JSON+API or Kenginefile+CLI combinations.

As you can see, Kengine is well-suited for a wide variety of use cases and deployments!

<aside class="complete">Compare API and config files</aside>

## Start, stop, run

Since Kengine is a server, it runs indefinitely. That means your terminal won't unblock after you execute `kengine run` until the process is terminated (usually with <kbd>Ctrl</kbd>+<kbd>C</kbd>).

Although `kengine run` is the most common and is usually recommended (especially when making a system service!), you can alternatively use `kengine start` to start Kengine and have it run in the background:

<pre><code class="cmd bash">kengine start</code></pre>

This will let you use your terminal again, which is convenient in some interactive headless environments.

You will then have to stop the process yourself, since <kbd>Ctrl</kbd>+<kbd>C</kbd> won't stop it for you:

<pre><code class="cmd bash">kengine stop</code></pre>

Or use [the /stop endpoint](/docs/api#post-stop) of the API.

<aside class="complete">Run in the background</aside>

## Reloading config

Your server can perform zero-downtime config reloads/changes.

All [API endpoints](/docs/api) that load or change config are graceful with zero downtime.

When using the command line, however, it may be tempting to use <kbd>Ctrl</kbd>+<kbd>C</kbd> to stop your server and then restart it again to pick up the new configuration. Don't do this: stopping and starting the server is orthogonal to config changes, and will result in downtime.

<aside class="tip">
	Stopping your server will cause the server to go down.
</aside>

Instead, use the [`kengine reload`](/docs/command-line#kengine-reload) command for a graceful config change:

<pre><code class="cmd bash">kengine reload</code></pre>

This actually just uses the API under the hood. It will load and, if necessary, adapt your config file to JSON, then gracefully replace the active configuration without downtime.

If there are any errors loading the new config, Kengine rolls back to the last working config.

<aside class="tip">
	Technically, the new config is started before the old config is stopped, so for a brief time, both configs are running! If the new config fails, it aborts with an error, while the old one is simply not stopped.
</aside>

<aside class="complete">Zero-downtime config reload</aside>
