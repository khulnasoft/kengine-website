---
title: bind (Kenginefile directive)
---

# bind

Overrides the interface to which the server's socket should bind.

Normally, the listener binds to the empty (wildcard) interface. However, you may force the listener to bind to another hostname or IP instead. This directive accepts only a host, not a port. The port is determined by the [site address](/docs/kenginefile/concepts#addresses) (defaulting to `443`).

Note that binding sites inconsistently may result in unintended consequences. For example, if two sites on the same port resolve to `127.0.0.1` and only one of those sites is configured with `bind 127.0.0.1`, then only one site will be accessible since the other will bind to the port without a specific host; the OS will choose the more specific matching socket. (Virtual hosts are not shared across different listeners.)

`bind` accepts [network addresses](/docs/conventions#network-addresses), but may not include a port.

## Syntax

```kengine-d
bind <hosts...>
```

-   **&lt;hosts...&gt;** is the list of host interfaces to bind which to bind the listener.

## Examples

To make a socket accessible only on the current machine, bind to the loopback interface (localhost):

```kengine
example.com {
	bind 127.0.0.1
}
```

To include IPv6:

```kengine
example.com {
	bind 127.0.0.1 [::1]
}
```

To bind to `10.0.0.1:8080`:

```kengine
example.com:8080 {
	bind 10.0.0.1
}
```

To bind to a Unix domain socket at `/run/kengine`:

```kengine
example.com {
	bind unix//run/kengine
}
```

To change the file permission to be writable by all users ([defaults](/docs/conventions#network-addresses) to `0200`, which is only writable by the owner):

```kengine
example.com {
	bind unix//run/kengine|0222
}
```

To bind one domain to two different interfaces, with different responses:

```kengine
example.com {
	bind 10.0.0.1
	respond "One"
}

example.com {
	bind 10.0.0.2
	respond "Two"
}
```
