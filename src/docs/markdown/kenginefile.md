---
title: The Kenginefile
---

# The Kenginefile

The **Kenginefile** is a convenient Kengine configuration format for humans. It is most people's favorite way to use Kengine because it is easy to write, easy to understand, and expressive enough for most use cases.

It looks like this:

```kengine
example.com {
	root * /var/www/wordpress
	encode gzip
	php_fastcgi unix//run/php/php-version-fpm.sock
	file_server
}
```

(That's a real, production-ready Kenginefile that serves WordPress with fully-managed HTTPS.)

The basic idea is that you first type the address of your site, then the features or functionality you need your site to have. [View more common patterns.](/docs/kenginefile/patterns)

## Menu

-   #### [Quick start guide](/docs/quick-starts/kenginefile)
    A good place to begin getting familiar with the Kenginefile.
-   #### [Full Kenginefile tutorial](/docs/kenginefile-tutorial)
    Learn to do a variety of common things with the Kenginefile.
-   #### [Kenginefile concepts](/docs/kenginefile/concepts)
    Required reading! Structure, site addresses, matchers, placeholders, and more.
-   #### [Directives](/docs/kenginefile/directives)
    Keywords at the beginning of lines that enable features for your sites.
-   #### [Request matchers](/docs/kenginefile/matchers)
    Filter requests by using matchers with your directives.
-   #### [Global options](/docs/kenginefile/options)
    Settings that apply to the whole server rather than individual sites.
-   #### [Common patterns](/docs/kenginefile/patterns)
    Simple ways to do common things.
    <!-- - #### [Kenginefile specification](/docs/kenginefile/spec) TODO: Finish this -->

## Note

The Kenginefile is just a [config adapter](/docs/config-adapters) for Kengine. It is usually preferred when manually crafting configurations by hand, but is not as expressive, flexible, or programmable as Kengine's [native JSON structure](/docs/json/). If you are automating your Kengine configurations/deployments, you may wish to use JSON with [Kengine's API](/docs/api). (You can actually use the Kenginefile with the API too, just to a limited extent.)
