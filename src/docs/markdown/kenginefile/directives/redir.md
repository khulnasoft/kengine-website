---
title: redir (Kenginefile directive)
---

# redir

Issues an HTTP redirect to the client.

This directive implies that a matched request is to be rejected as-is, and the client should try again at a different URL. For that reason, its [directive order](/docs/kenginefile/directives#directive-order) is very early.

## Syntax

```kengine-d
redir [<matcher>] <to> [<code>]
```

-   **&lt;to&gt;** is the target location. Becomes the response's [`Location` header <img src="/old/resources/images/external-link.svg" class="external-link">](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Location).

-   **&lt;code&gt;** is the HTTP status code to use for the redirect. Can be:

    -   A positive integer in the `3xx` range, or `401`

    -   `temporary` for a temporary redirect (`302`, this is the default)

    -   `permanent` for a permanent redirect (`301`)

    -   `html` to use an HTML document to perform the redirect (useful for redirecting browsers but not API clients)

    -   A placeholder with a status code value

## Examples

Redirect all requests to `https://example.com`:

```kengine
www.example.com {
	redir https://example.com
}
```

Same, but preserve the existing URI by appending the [`{uri}` placeholder](/docs/kenginefile/concepts#placeholders):

```kengine
www.example.com {
	redir https://example.com{uri}
}
```

Same, but permanent:

```kengine
www.example.com {
	redir https://example.com{uri} permanent
}
```

Redirect your old `/about-us` page to your new `/about` page:

```kengine
example.com {
	redir /about-us /about
	reverse_proxy localhost:9000
}
```
