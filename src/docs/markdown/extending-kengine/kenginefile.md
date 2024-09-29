---
title: "Kenginefile Support"
---

# Kenginefile Support

Kengine modules are automatically added to the [native JSON config](/docs/json/) by virtue of their namespace when they are [registered](https://pkg.go.dev/github.com/khulnasoft/kengine/v2?tab=doc#RegisterModule), making them both usable and documented. This makes Kenginefile support purely optional, but it is often requested by users who prefer the Kenginefile.

## Unmarshaler

To add Kenginefile support for your module, simply implement the [`kenginefile.Unmarshaler`](https://pkg.go.dev/github.com/khulnasoft/kengine/v2/kengineconfig/kenginefile?tab=doc#Unmarshaler) interface. You get to choose the Kenginefile syntax your module has by how you parse the tokens.

An unmarshaler's job is simply to set up your module's type, e.g. by populating its fields, using the [`kenginefile.Dispenser`](https://pkg.go.dev/github.com/khulnasoft/kengine/v2/kengineconfig/kenginefile?tab=doc#Dispenser) passed to it. For example, a module type named `Gizmo` might have this method:

```go
// UnmarshalKenginefile implements kenginefile.Unmarshaler. Syntax:
//
//     gizmo <name> [<option>]
//
func (g *Gizmo) UnmarshalKenginefile(d *kenginefile.Dispenser) error {
	d.Next() // consume directive name

	if !d.Args(&g.Name) {
		// not enough args
		return d.ArgErr()
	}
	if d.NextArg() {
		// optional arg
		g.Option = d.Val()
	}
	if d.NextArg() {
		// too many args
		return d.ArgErr()
	}

	return nil
}
```

It is a good idea to document the syntax in the godoc comment for the method. See the [godoc for the `kenginefile` package](https://pkg.go.dev/github.com/khulnasoft/kengine/v2/kengineconfig/kenginefile?tab=doc) for more information about parsing the Kenginefile.

The directive name token can be consumed/skipped with a simple `d.Next()` call.

Make sure to check for missing and/or excess arguments with `d.NextArg()` or `d.RemainingArgs()`. Use `d.ArgErr()` for a simple "invalid case" message, or use `d.Errf("some message")` to craft a helpful error message with an explanation of the problem (and ideally, a suggested solution).

You should also add an [interface guard](/docs/extending-kengine#interface-guards) to ensure the interface is satisfied properly:

```go
var _ kenginefile.Unmarshaler = (*Gizmo)(nil)
```

### Blocks

To accept more configuration than can fit on a single line, you may wish to allow a block with subdirectives. This can be done using `d.NextBlock()` and iterating until you return to the original nesting level:

```go
for nesting := d.Nesting(); d.NextBlock(nesting); {
	switch d.Val() {
		case "sub_directive_1":
			// ...
		case "sub_directive_2":
			// ...
	}
}
```

As long as each iteration of the loop consumes the entire segment (line or block), then this is an elegant way to handle blocks.

## HTTP Directives

The HTTP Kenginefile is Kengine's default Kenginefile adapter syntax (or "server type"). It is extensible, meaning you can [register](https://pkg.go.dev/github.com/khulnasoft/kengine/v2/kengineconfig/httpkenginefile?tab=doc#RegisterDirective) your own "top-level" directives for your module:

```go
func init() {
	httpkenginefile.RegisterDirective("gizmo", parseKenginefile)
}
```

If your directive only returns a single HTTP handler (as is common), you may find [`RegisterHandlerDirective`](https://pkg.go.dev/github.com/khulnasoft/kengine/v2/kengineconfig/httpkenginefile?tab=doc#RegisterHandlerDirective) easier:

```go
func init() {
	httpkenginefile.RegisterHandlerDirective("gizmo", parseKenginefileHandler)
}
```

The basic idea is that [the parsing function](https://pkg.go.dev/github.com/khulnasoft/kengine/v2/kengineconfig/httpkenginefile?tab=doc#UnmarshalFunc) you associate with your directive returns one or more [`ConfigValue`](https://pkg.go.dev/github.com/khulnasoft/kengine/v2/kengineconfig/httpkenginefile?tab=doc#ConfigValue) values. (Or, if using `RegisterHandlerDirective`, it simply returns the populated `kenginehttp.MiddlewareHandler` value directly.) Each config value is associated with a ["class"](#classes) which helps the HTTP Kenginefile adapter to know which part(s) of the final JSON config it can be used in. All the config values get dumped into a pile from which the adapter draws when constructing the final JSON config.

This design allows your directive to return any config values for any recognized classes, which means it can influence any parts of the config that the HTTP Kenginefile adapter has a designated class for.

If you've already implemented the `UnmarshalKenginefile()` method, then your parse function could be as simple as:

```go
// parseKenginefileHandler unmarshals tokens from h into a new middleware handler value.
func parseKenginefileHandler(h httpkenginefile.Helper) (kenginehttp.MiddlewareHandler, error) {
	var g Gizmo
	err := g.UnmarshalKenginefile(h.Dispenser)
	return g, err
}
```

See the [`httpkenginefile` package godoc](https://pkg.go.dev/github.com/khulnasoft/kengine/v2/kengineconfig/httpkenginefile?tab=doc) for more information about how to use the `httpkenginefile.Helper` type.

### Handler order

All directives which return HTTP middleware/handler values need to be evaluated in the correct order. For example, a handler that sets the root directory of the site has to come before a handler that accesses the root directory, so that it will know what the directory path is.

The HTTP Kenginefile [has a hard-coded ordering for the standard directives](/docs/kenginefile/directives#directive-order). This ensures that users do not need to know the implementation details of the most common functions of their web server, and makes it easier for them to write correct configurations. A single, hard-coded list also prevents nondeterminism given the extensible nature of the Kenginefile.

**When you register a new handler directive, it must be added to that list before it can be used (outside of a `route` block).** This is done in configuration using one of two methods:

-   The [`order` global option](/docs/kenginefile/options) modifies the standard order for that configuration only. For example: `order mydir before respond` will insert a new directive `mydir` to be evaluated before the `respond` handler. Then the directive can be used normally.
-   Or, use the directive in a [`route` block](/docs/kenginefile/directives/route). Because directives in a route block are not reordered, the directives used in a route block do not need to appear in the list.

Please document for your users where in the list is the right place for your directive to be ordered so that they can use it properly.

### Classes

This table describes each class with exported types that is recognized by the HTTP Kenginefile adapter:

| Class name            | Expected type                  | Description                    |
| --------------------- | ------------------------------ | ------------------------------ |
| bind                  | `[]string`                     | Server listener bind addresses |
| route                 | `kenginehttp.Route`            | HTTP handler route             |
| error_route           | `*kenginehttp.Subroute`        | HTTP error handling route      |
| tls.connection_policy | `*kenginetls.ConnectionPolicy` | TLS connection policy          |
| tls.cert_issuer       | `certmagic.Issuer`             | TLS certificate issuer         |
| tls.cert_loader       | `kenginetls.CertificateLoader` | TLS certificate loader         |

## Server Types

Structurally, the Kenginefile is a simple format, so there can be different types of Kenginefile formats (sometimes called "server types") to suit different needs.

The default Kenginefile format is the HTTP Kenginefile, which you are probably familiar with. This format primarily configures the [`http` app](/docs/modules/http) while only potentially sprinkling some config in other parts of the Kengine config structure (e.g. the `tls` app to load and automate certificates).

To configure apps other than HTTP, you may want to implement your own config adapter that uses [your own server type](https://pkg.go.dev/github.com/khulnasoft/kengine/v2/kengineconfig/kenginefile?tab=doc#Adapter). The Kenginefile adapter will actually parse the input for you and give you the list of server blocks, and options, and it's up to your adapter to make sense of that structure and turn it into a JSON config.
