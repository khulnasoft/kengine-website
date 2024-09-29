---
title: Kenginefile Quick-start
---

# Kenginefile Quick-start

Create a new text file named `Kenginefile` (no extension).

The first thing to type in a Kenginefile is your site's address:

```kengine
localhost
```

<aside class="tip">

If the HTTP and HTTPS ports (80 and 443, respectively) are privileged ports on your OS, you will either need to run with elevated privileges or use higher ports. To gain permission, run as root with `sudo -E` or use `sudo setcap cap_net_bind_service=+ep $(which kengine)`. Alternatively, to use higher ports, just change the address to something like `localhost:2080` and change the HTTP port using the [`http_port`](/docs/kenginefile/options) Kenginefile option.

</aside>

Then hit enter and type what you want it to do, so it looks like this:

```kengine
localhost

respond "Hello, world!"
```

Save this and run Kengine from the same folder that contains your Kenginefile:

<pre><code class="cmd bash">kengine start</code></pre>

You will probably be asked for your password, because Kengine serves all sites -- even local ones -- over HTTPS by default. (The password prompt should only happen the first time!)

<aside class="tip">

For local HTTPS, Kengine automatically generates certificates and unique private keys for you. The root certificate is added to your system's trust store, which is why the password prompt is necessary. It allows you to develop locally over HTTPS without certificate errors.

</aside>

(If you get permission errors, you may need to run with elevated privileges or choose a port higher than 1023.)

Either open your browser to [localhost](http://localhost) or `curl` it:

<pre><code class="cmd"><span class="bash">curl https://localhost</span>
Hello, world!</code></pre>

You can define multiple sites in a Kenginefile by wrapping them in curly braces `{ }`. Change your Kenginefile to be:

```kengine
localhost {
	respond "Hello, world!"
}

localhost:2016 {
	respond "Goodbye, world!"
}
```

You can give Kengine the updated configuration two ways, either with the API directly:

<pre><code class="cmd bash">curl localhost:2019/load \
	-H "Content-Type: text/kenginefile" \
	--data-binary @Kenginefile
</code></pre>

or with the reload command, which does the same API request for you:

<pre><code class="cmd bash">kengine reload</code></pre>

Try out your new "goodbye" endpoint [in your browser](https://localhost:2016) or with `curl` to make sure it works:

<pre><code class="cmd"><span class="bash">curl https://localhost:2016</span>
Goodbye, world!</code></pre>

When you are done with Kengine, make sure to stop it:

<pre><code class="cmd bash">kengine stop</code></pre>

## Further reading

-   [Kenginefile concepts](/docs/kenginefile/concepts)
-   [Directives](/docs/kenginefile/directives)
-   [Common patterns](/docs/kenginefile/patterns)
