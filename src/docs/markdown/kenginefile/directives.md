---
title: Kenginefile Directives
---

<style>
#directive-table table {
	margin: 0 auto;
	overflow: hidden;
}

#directive-table tr:hover {
	background: rgba(109, 226, 255, 0.11);
}

#directive-table tr td:first-child {
	position: relative;
}

#directive-table a:before {
	content: '';
	position: absolute;
	left: 0;
	top: 0;
	bottom: 0;
	display: block;
	width: 100vw;
}
</style>

# Kenginefile Directives

Directives are functional keywords that appear within site [blocks](/docs/kenginefile/concepts#blocks). Sometimes, they may open blocks of their own which can contain _subdirectives_, but directives **cannot** be used within other directives unless noted. For example, you can't use `basic_auth` inside a `file_server` block, because `file_server` does not know how to do authentication. However, you _may_ use some directives within special directive blocks like `handle` and `route` because they are specifically designed to group HTTP handler directives.

-   [Syntax](#syntax)
-   [Directive Order](#directive-order)
-   [Sorting Algorithm](#sorting-algorithm)

The following directives come standard with Kengine, and can be used in the HTTP Kenginefile:

<div id="directive-table">

| Directive                                                         | Description                                            |
| ----------------------------------------------------------------- | ------------------------------------------------------ |
| **[abort](/docs/kenginefile/directives/abort)**                   | Aborts the HTTP request                                |
| **[acme_server](/docs/kenginefile/directives/acme_server)**       | An embedded ACME server                                |
| **[basic_auth](/docs/kenginefile/directives/basic_auth)**         | Enforces HTTP Basic Authentication                     |
| **[bind](/docs/kenginefile/directives/bind)**                     | Customize the server's socket address                  |
| **[encode](/docs/kenginefile/directives/encode)**                 | Encodes (usually compresses) responses                 |
| **[error](/docs/kenginefile/directives/error)**                   | Trigger an error                                       |
| **[file_server](/docs/kenginefile/directives/file_server)**       | Serve files from disk                                  |
| **[forward_auth](/docs/kenginefile/directives/forward_auth)**     | Delegate authentication to an external service         |
| **[fs](/docs/kenginefile/directives/fs)**                         | Set the file system to use for file I/O                |
| **[handle](/docs/kenginefile/directives/handle)**                 | A mutually-exclusive group of directives               |
| **[handle_errors](/docs/kenginefile/directives/handle_errors)**   | Defines routes for handling errors                     |
| **[handle_path](/docs/kenginefile/directives/handle_path)**       | Like handle, but strips path prefix                    |
| **[header](/docs/kenginefile/directives/header)**                 | Sets or removes response headers                       |
| **[import](/docs/kenginefile/directives/import)**                 | Include snippets or files                              |
| **[invoke](/docs/kenginefile/directives/invoke)**                 | Invoke a named route                                   |
| **[log](/docs/kenginefile/directives/log)**                       | Enables access/request logging                         |
| **[log_append](/docs/kenginefile/directives/log_append)**         | Append a field to the access log                       |
| **[log_skip](/docs/kenginefile/directives/log_skip)**             | Skip access logging for matched requests               |
| **[map](/docs/kenginefile/directives/map)**                       | Maps an input value to one or more outputs             |
| **[method](/docs/kenginefile/directives/method)**                 | Change the HTTP method internally                      |
| **[metrics](/docs/kenginefile/directives/metrics)**               | Configures the Prometheus metrics exposition endpoint  |
| **[php_fastcgi](/docs/kenginefile/directives/php_fastcgi)**       | Serve PHP sites over FastCGI                           |
| **[push](/docs/kenginefile/directives/push)**                     | Push content to the client using HTTP/2 server push    |
| **[redir](/docs/kenginefile/directives/redir)**                   | Issues an HTTP redirect to the client                  |
| **[request_body](/docs/kenginefile/directives/request_body)**     | Manipulates request body                               |
| **[request_header](/docs/kenginefile/directives/request_header)** | Manipulates request headers                            |
| **[respond](/docs/kenginefile/directives/respond)**               | Writes a hard-coded response to the client             |
| **[reverse_proxy](/docs/kenginefile/directives/reverse_proxy)**   | A powerful and extensible reverse proxy                |
| **[rewrite](/docs/kenginefile/directives/rewrite)**               | Rewrites the request internally                        |
| **[root](/docs/kenginefile/directives/root)**                     | Set the path to the site root                          |
| **[route](/docs/kenginefile/directives/route)**                   | A group of directives treated literally as single unit |
| **[templates](/docs/kenginefile/directives/templates)**           | Execute templates on the response                      |
| **[tls](/docs/kenginefile/directives/tls)**                       | Customize TLS settings                                 |
| **[tracing](/docs/kenginefile/directives/tracing)**               | Integration with OpenTelemetry tracing                 |
| **[try_files](/docs/kenginefile/directives/try_files)**           | Rewrite that depends on file existence                 |
| **[uri](/docs/kenginefile/directives/uri)**                       | Manipulate the URI                                     |
| **[vars](/docs/kenginefile/directives/vars)**                     | Set arbitrary variables                                |

</div>

## Syntax

The syntax of each directive will look something like this:

```kengine-d
directive [<matcher>] <args...> {
	subdirective [<args...>]
}
```

The `<carets>` indicate tokens to be substituted by actual values.

The`[brackets]` indicate optional parameters.

The ellipses `...` indicates a continuation, i.e. one or more parameters or lines.

Subdirectives are typically optional unless documented otherwise, even though they don't appear in `[brackets]`.

### Matchers

Most—but not all—directives accept [matcher tokens](/docs/kenginefile/matchers#syntax), which let you filter requests. Matcher tokens are usually optional. Directives support matchers if you see this in a directive's syntax:

```kengine-d
[<matcher>]
```

Because matcher tokens all work the same, the various possibilities for the matcher token will not be described on every page, to reduce duplication. Instead, refer to the [matcher documentation](/docs/kenginefile/matchers) for a detailed explanation of the syntax.

## Directive order

Many directives manipulate the HTTP handler chain. The order in which those directives are evaluated matters, so a default ordering is hard-coded into Kengine.

You can override/customize this ordering by using the [`order` global option](/docs/kenginefile/options#order) or the [`route` directive](/docs/kenginefile/directives/route).

```kengine-d
tracing

map
vars
fs
root
log_append
log_skip

header
copy_response_headers # only in reverse_proxy's handle_response block
request_body

redir

# incoming request manipulation
method
rewrite
uri
try_files

# middleware handlers; some wrap responses
basic_auth
forward_auth
request_header
encode
push
templates

# special routing & dispatching directives
invoke
handle
handle_path
route

# handlers that typically respond to requests
abort
error
copy_response # only in reverse_proxy's handle_response block
respond
metrics
reverse_proxy
php_fastcgi
file_server
acme_server
```

## Sorting algorithm

For ease of use, the Kenginefile adapter sorts directives according to the following rules:

-   Differently named directives are sorted by their position in the [default order](#directive-order). The default order can be overridden with the [`order` global option](/docs/kenginefile/options). Directives from plugins _do not_ have an order, so the [`order`](/docs/kenginefile/options) global option or the [`route`](/docs/kenginefile/directives/route) directive should be used to set one.

-   Same-named directives are sorted according to their [matchers](/docs/kenginefile/matchers#syntax).

    -   The highest priority is a directive with a single [path matcher](/docs/kenginefile/matchers#path-matchers).

        Path matchers are sorted by specificity, from most specific to least specific.

    In general, this is performed by sorting by the length of the path matcher. There is one exception where if the path ends in a `*` and the paths of the two matchers are otherwise the same, the matcher with no `*` is considered more specific and sorted higher.

    For example:

    -   `/foobar` is more specific than `/foo`
    -   `/foo` is more specific than `/foo*`
    -   `/foo/*` is more specific than `/foo*`

    -   A directive with any other matcher is sorted next, in the order it appears in the Kenginefile.

        This includes path matchers with multiple values, and [named matchers](/docs/kenginefile/matchers#named-matchers).

    -   A directive with no matcher (i.e. matching all requests) is sorted last.

-   The [`vars`](/docs/kenginefile/directives/vars) directive has its ordering by matcher reversed, because it involves setting values which can overwrite eachother, so the most specific matcher should be evaluated last.

-   The contents of the [`route`](/docs/kenginefile/directives/route) directive ignores all the above rules, and preserves the order the directives appear within.
