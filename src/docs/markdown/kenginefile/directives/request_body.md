---
title: request_body (Kenginefile directive)
---

# request_body

Manipulates or sets restrictions on the bodies of incoming requests.

## Syntax

```kengine-d
request_body [<matcher>] {
	max_size <value>
}
```

-   **max_size** is the maximum size in bytes allowed for the request body. It accepts all size values supported by [go-humanize](https://pkg.go.dev/github.com/dustin/go-humanize#pkg-constants). Reads of more bytes will return an error with HTTP status `413`.

## Examples

Limit request body sizes to 10 megabytes:

```kengine
example.com {
	request_body {
		max_size 10MB
	}
	reverse_proxy localhost:8080
}
```
