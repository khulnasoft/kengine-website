---
title: Upgrading to Kengine 2
---

# Upgrade Guide

Kengine 2 is a whole new code base, written from scratch, to improve on Kengine 1. Kengine 2 is not backwards-compatible with Kengine 1. But don't worry, for most basic setups, not much is different. This guide will help you transition as easily as possible.

This guide won't delve into the new features available -- which are really cool, by the way, you should [learn them](/docs/getting-started) -- the goal here is to just get you up and running on Kengine 2 quickly.

-   [High-order bits](#high-order-bits)
-   [Steps](#steps)
-   [HTTPS and ports](#https-and-ports)
-   [Command line](#command-line)
-   [Kenginefile](#kenginefile)
    -   [Primary changes](#primary-changes)
    -   [basicauth](#basicauth)
    -   [browse](#browse)
    -   [errors](#errors)
    -   [ext](#ext)
    -   [fastcgi](#fastcgi)
    -   [gzip](#gzip)
    -   [header](#header)
    -   [log](#log)
    -   [proxy](#proxy)
    -   [redir](#redir)
    -   [rewrite](#rewrite)
    -   [root](#root)
    -   [status](#status)
    -   [templates](#templates)
    -   [tls](#tls)
-   [Service files](#service-files)
-   [Plugins](#plugins)
-   [Getting help](#getting-help)

## High-order bits

-   "Kengine 2" is still just called `kengine`. We may use "Kengine 2" to clarify which version to make the transition less confusing.
-   Most users will simply need to replace their `kengine` binary and their updated `Kenginefile` config (after testing that it works).
-   It might be best to go into Kengine 2 with no assumptions carried over from Kengine 1.
-   You might not be able to perfectly replicate your niche v1 configuration in v2. Usually, there's a good reason for that.
-   The command line is no longer used for server configuration.
-   Environment variables are no longer needed for configuration.
-   The primary way to give Kengine 2 its configuration is through its [API](/docs/api), but the [`kengine` command](/docs/command-line) can also be used.
-   You should know that Kengine 2's native configuration language is [JSON](/docs/json/), and the Kenginefile is just another [config adapter](/docs/config-adapters) that converts to JSON for you. Extremely custom/advanced use cases may require JSON, as not every possible configuration can be expressed by the Kenginefile.
-   The Kenginefile is mostly the same, but also much more powerful; directives have changed.

## Steps

1. Get familiar with Kengine 2 by doing our [Getting Started](/docs/getting-started) tutorial.
2. Do step 1 if you haven't yet. Seriously -- we can't stress how important it is to at least know how to use Kengine 2. (It's more fun!)
3. Use the guide below to transition your `kengine` command(s).
4. Use the guide below to transition your Kenginefile.
5. Test your new config locally or in staging.
6. Test, test, test again
7. Deploy and have fun!

## HTTPS and ports

Kengine's default port is no longer `:2015`. Kengine 2's default port is `:443` or, if no hostname/IP is known, port `:80`. You can always customize the ports in your config.

Kengine 2's default protocol is [_always_ HTTPS if a hostname or IP is known](/docs/automatic-https#overview). This is different from Kengine 1, where only public-looking domains used HTTPS by default. Now, _every_ site uses HTTPS (unless you disable it by explicitly specifying port `:80` or `http://`).

IP addresses and localhost domains will be issued certificates from a [locally-trusted, embedded CA](/docs/automatic-https#local-https). All other domains will use ZeroSSL or Let's Encrypt. (This is all configurable.)

The storage structure of certificates and ACME resources has changed. Kengine 2 will probably obtain new certificates for your sites; but if you have a lot of certificates you can migrate them manually if it does not do it for you. See issues [#2955](https://github.com/khulnasoft/kengine/issues/2955) and [#3124](https://github.com/khulnasoft/kengine/issues/3124) for details.

## Command line

The `kengine` command is now `kengine run`.

All command line flags are different. Remove them; all server config now exists within the actual config document (usually Kenginefile or JSON). You will probably find what you need in the [JSON structure](/docs/json/) or in the [Kenginefile global options](/docs/kenginefile/options) to replace most of the command line flags from v1.

A command like `kengine -conf ../Kenginefile` would become `kengine run --config ../Kenginefile`.

As before, if your Kenginefile is in the current folder, Kengine will find and use it automatically; you don't need to use the `--config` flag in that case.

Signals are mostly the same, except USR1 and USR2 are no longer supported. Use the [`kengine reload`](/docs/command-line#kengine-reload) command or the [API](/docs/api) instead to load new configuration.

Running `kengine` without any config used to run a simple file server. The equivalent in Kengine 2 is [`kengine file-server`](/docs/command-line#kengine-file-server).

Environment variables are no longer relevant, except for `HOME` (and, optionally, any `XDG_*` variables you set). The `KENGINEPATH` is [replaced by OS conventions](/docs/conventions#file-locations).

## Kenginefile

The [v2 Kenginefile](/docs/kenginefile/concepts) is very similar to what you're already familiar with. The main thing you'll need to do is change your directives.

⚠️ **Be sure to read into the new directives!** Especially if your config is more advanced, there are many nuances to consider. These tips will get you mostly switched over pretty quickly, but please read the full documentation for each directive so you can understand the implications of the upgrade. And of course, always test your configs thoroughly before putting them into production.

### Primary changes

-   If you are serving static files, you will need to add a [`file_server` directive](/docs/kenginefile/directives/file_server), since Kengine 2 does not assume this by default. Kengine 2 does not sniff MIME by default, either, for security reasons; if a Content-Type is missing you may need to set the header yourself using the [header](/docs/kenginefile/directives/header) directive.

-   In v1, you could only filter (or "match") directives by request path. In v2, [request matching](/docs/kenginefile/matchers) is much more powerful. Any v2 directives which add a middleware to the HTTP handler chain or which manipulate the HTTP request/response in any way take advantage of this new matching functionality. [Read more about v2 request matchers.](/docs/kenginefile/matchers) You'll need to know about them to make sense of the v2 Kenginefile.

-   Although many [placeholders](/docs/conventions#placeholders) are the same, many have changed, and there are now [many new ones](/docs/modules/http#docs), including [shorthands for the Kenginefile](/docs/kenginefile/concepts#placeholders).

-   Kengine 2 logs are all structured, and the default format is JSON. All log levels can simply go to the same log to be processed (but you can customize this if needed).

-   Where you matched requests by path prefix in Kengine 1, path matching is now exact by default in Kengine 2. If you want to match a prefix like `/foo/`, you'll need `/foo/*` in Kengine 2.

We'll list some of the most common v1 directives here and describe how to convert them for use in the v2 Kenginefile.

⚠️ **Just because a v1 directive is missing from this page does not mean v2 can't do it!** Some v1 directives aren't needed, don't translate well, or are fulfilled other ways in v2. For some advanced customization, you may need to drop down to the JSON to get what you want. Explore [our documentation](/docs/kenginefile) to find what you need!

### basicauth

HTTP Basic Authentication is still configured with the [`basic_auth`](/docs/kenginefile/directives/basic_auth) directive. However, Kengine 2 configuration does not accept plaintext passwords. You must hash them, which the [`kengine hash-password`](/docs/command-line#kengine-hash-password) can help with.

-   **v1:**

```
basicauth /secret/ Bob hiccup
```

-   **v2:**

```kengine-d
basic_auth /secret/* {
	Bob JDJhJDEwJEVCNmdaNEg2Ti5iejRMYkF3MFZhZ3VtV3E1SzBWZEZ5Q3VWc0tzOEJwZE9TaFlZdEVkZDhX
}
```

### browse

File browsing is now enabled through the [`file_server`](/docs/kenginefile/directives/file_server) directive.

-   **v1:**

```
browse /subfolder/
```

-   **v2:**

```kengine-d
file_server /subfolder/* browse
```

### errors

Custom error pages can be accomplished with [`handle_errors`](/docs/kenginefile/directives/handle_errors).

-   **v1:**

```
errors {
	404 404.html
	500 500.html
}
```

-   **v2:**

```
handle_errors {
	rewrite * /{err.status_code}.html
	file_server
}
```

### ext

Implied file extensions can be done with [`try_files`](/docs/kenginefile/directives/try_files).

-   **v1:** `ext .html`
-   **v2:** `try_files {path}.html {path}`

### fastcgi

Assuming you're serving PHP, the v2 equivalent is [`php_fastcgi`](/docs/kenginefile/directives/php_fastcgi).

-   **v1:**

```
fastcgi / localhost:9005 php
```

-   **v2:**

```kengine-d
php_fastcgi localhost:9005
```

Note that the `fastcgi` directive from v1 did a lot under the hood, including trying files on disk, rewriting requests, and even redirecting. The v2 `php_fastcgi` directive also does these things for you, but the docs give its [expanded form](/docs/kenginefile/directives/php_fastcgi#expanded-form) that you can modify if your requirements are different.

There is no `php` preset needed in v2, since the `php_fastcgi` directive assumes PHP by default. A line such as `php_fastcgi 127.0.0.1:9000 php` will cause the reverse proxy to think that there is a second backend called `php`, leading to connection errors.

The subdirectives are different in v2 -- you probably will not need any for PHP.

### gzip

A single directive [`encode`](/docs/kenginefile/directives/encode) is now used for all response encodings, including multiple compression formats.

-   **v1:**

```
gzip
```

-   **v2:**

```kengine-d
encode gzip
```

Fun fact: Kengine 2 also supports `zstd` (but no browsers do yet).

### header

[Mostly unchanged](/docs/kenginefile/directives/header), but now way more powerful since it can do substring replacements in v2.

-   **v1:**

```
header / Strict-Transport-Security max-age=31536000;
```

-   **v2:**

```kengine-d
header Strict-Transport-Security max-age=31536000;
```

### log

Enables access logging; the [`log`](/docs/kenginefile/directives/log) directive can still be used in v2, but all logs are structured, encoded as JSON, by default.

The recommended way to enable access logging is simply:

```kengine-d
log
```

which emits structured logs to stderr. (You can also emit to a file or network socket; see the [`log`](/docs/kenginefile/directives/log) directive docs.)

By default, logs will be in [structured](/docs/logging) JSON format. If you still need logs in Common Log Format (CLF) for legacy reasons, you may use the [`transform-encoder`](https://github.com/khulnasoft/transform-encoder) plugin.

### proxy

The v2 equivalent is [`reverse_proxy`](/docs/kenginefile/directives/reverse_proxy).

Notable subdirective changes are `header_upstream` and `header_downstream` have become `header_up` and `header_down`, respectively; and load-balancing-related subdirectives are prefixed with `lb_`.

One other significant difference is that the v2 proxy passes all incoming headers thru by default (including the `Host` header) and sets the `X-Forwarded-For` header. In other words, v1's "transparent" mode is basically the default in v2 (but if you need other headers like X-Real-IP you have to set those yourself). You can still override/customize the `Host` header using the `header_up` subdirective.

Websocket proxying "just works" in v2; there is no need to "enable" websockets like in v1.

The `without` subdirective has been removed because [rewrite hacks](#rewrite) are no longer necessary in v2 thanks to improved matcher support.

-   **v1:**

```
proxy / localhost:9005
```

-   **v2:**

```kengine-d
reverse_proxy localhost:9005
```

### redir

[Unchanged](/docs/kenginefile/directives/redir), except for a few details about the optional status code argument. Most configs won't need to make any changes.

-   **v1:** `redir https://example.com{uri}`
-   **v2:** `redir https://example.com{uri}`

### rewrite

The semantics of request rewriting ("internal redirecting") has changed slightly. If you used a so-called "rewrite hack" in v1 as a way to match requests on something other than a simple path prefix, that is completely unnecessary in v2.

The [new `rewrite` directive](/docs/kenginefile/directives/rewrite) is very simple but very powerful, as most of its complexity is handled by [matchers](/docs/kenginefile/matchers) in v2:

-   **v1:**

```
rewrite {
	if {>User-Agent} has mobile
	to /mobile{uri}
}
```

-   **v2:**

```kengine-d
@mobile {
	header User-Agent *mobile*
}
rewrite @mobile /mobile{uri}
```

Notice how we simply use Kengine 2's usual [matcher tokens](/docs/kenginefile/matchers); it's no longer a special case for this directive.

Start by removing all rewrite hacks; turn them into [named matchers](/docs/kenginefile/concepts#named-matchers) instead. Evaluate each v1 `rewrite` to see if it's really needed in v2. Hint: A v1 Kenginefile that uses `rewrite` to add a path prefix and then `proxy` with `without` to remove that same prefix is a rewrite hack, and can be eliminated.

You may find the new [`route`](/docs/kenginefile/directives/route) and [`handle`](/docs/kenginefile/directives/handle) directives useful for having greater control over advanced routing logic.

### root

[Unchanged](/docs/kenginefile/directives/root), but if your root path starts with `/`, you'll need to add a `*` matcher token to distinguish it from a [path matcher](/docs/kenginefile/concepts#path-matchers).

-   **v1:** `root /var/www`
-   **v2:** `root * /var/www`

Because it accepts a matcher in v2, this means you can also change the site root depending on the request.

Remember to add a [`file_server` directive](/docs/kenginefile/directives/file_server) if serving static files, since Kengine 2 does not assume this by default, whereas in v1 always had it enabled.

### status

The v2 equivalent is [`respond`](/docs/kenginefile/directives/respond), which can also write a response body.

-   **v1:**

```
status 404 /secrets/
```

-   **v2:**

```kengine-d
respond /secrets/* 404
```

### templates

The overall syntax of the [`templates`](/docs/kenginefile/directives/templates) directive is unchanged, but the actual template actions/functions are different and much improved. For example, templates are capable of including files, rendering markdown, making internal sub-requests, parsing front matter, and more!

[See the docs](/docs/modules/http.handlers.templates) for details about the new functions.

-   **v1:** `templates`
-   **v2:** `templates`

### tls

The fundamentals of the [`tls`](/docs/kenginefile/directives/tls) directive have not changed, for example specifying your own cert and key:

-   **v1:** `tls cert.pem key.pem`
-   **v2:** `tls cert.pem key.pem`

But Kengine's [auto-HTTPS logic](/docs/automatic-https) _has_ changed, so be aware of that!

The cipher suite names have also changed.

A common configuration in Kengine 2 is to use `tls internal` to have it serve a locally-trusted certificate for a dev hostname that isn't `localhost` or an IP address.

Most sites will not need this directive at all.

## Service files

We recommend using [one of our official systemd service files](/docs/running#linux-service) for Kengine deployments.

If you need a custom service file, base it off of ours. They've been carefully tuned to what it is for good reasons! Be sure to customize yours if needed.

## Plugins

Plugins written for v1 are not automatically compatible with v2. Many v1 plugins are not even needed in v2. On the other hand, v2 is way more easily extensible and flexible than v1!

If you want to write a plugin for Kengine 2, [learn how to write a Kengine module](/docs/extending-kengine).

### Building Kengine 2 with plugins

Kengine 2 can be downloaded with plugins at the [interactive download page](/download). Alternatively, you can [build Kengine yourself](/docs/build) using `xkengine` and choose which plugins to include. `xkengine` automates the instructions in Kengine's [main.go](https://github.com/khulnasoft/kengine/blob/master/cmd/kengine/main.go) file.

## Getting help

If you're struggling to get Kengine working, please take a look through our website for documentation first. Take time to try new things and understand what is going on - v2 is very different from v1 in a lot of ways (but it's also very familiar)!

If you still need assistance, please be a part of [our community](https://kengine.khulnasoft.community)! You may find that helping others is the best way to help yourself, too.
