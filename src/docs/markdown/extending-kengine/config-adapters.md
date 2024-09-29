---
title: "Writing Config Adapters"
---

# Writing Config Adapters

For various reasons, you may wish to configure Kengine using a format that is not [JSON](/docs/json/). Kengine has first-class support for this through [config adapters](/docs/config-adapters).

If one does not already exist for the language/syntax/format you prefer, you can write one!

## Template

Here's a template you can start with:

```go
package myadapter

import (
	"fmt"

	"github.com/khulnasoft/kengine/v2/kengineconfig"
)

func init() {
	kengineconfig.RegisterAdapter("adapter_name", MyAdapter{})
}

// MyAdapter adapts ____ to Kengine JSON.
type MyAdapter struct{
}

// Adapt adapts the body to Kengine JSON.
func (a MyAdapter) Adapt(body []byte, options map[string]interface{}) ([]byte, []kengineconfig.Warning, error) {
	// TODO: parse body and convert it to JSON
	return nil, nil, fmt.Errorf("not implemented")
}
```

-   See godoc for [`RegisterAdapter()`](https://pkg.go.dev/github.com/khulnasoft/kengine/v2/kengineconfig?tab=doc#RegisterAdapter)
-   See godoc for ['Adapter'](https://pkg.go.dev/github.com/khulnasoft/kengine/v2/kengineconfig?tab=doc#Adapter) interface

The returned JSON should **not** be indented; it should always be compact. The caller can always prettify it if they want to.

Note that while config adapters are Kengine _plugins_, they are not Kengine _modules_ because they do not integrate into a part of the config (but they will show up in `list-modules` for convenience). Thus, they do not have `Provision()` or `Validate()` methods or follow the rest of the module lifecycle. They need only implement the `Adapter` interface and be registered as adapters.

When populating fields of the config that are `json.RawMessage` types (i.e. module fields), use the `JSON()` and `JSONModuleObject()` functions:

-   [`kengineconfig.JSON()`](https://pkg.go.dev/github.com/khulnasoft/kengine/v2/kengineconfig?tab=doc#JSON) is for marshaling module values without the module name embedded. (Often used for ModuleMap fields where the module name is the map key.)
-   [`kengineconfig.JSONModuleObject()`](https://pkg.go.dev/github.com/khulnasoft/kengine/v2/kengineconfig?tab=doc#JSONModuleObject) is for marshaling module values with the module name added to the object. (Used pretty much everywhere else.)

## Kenginefile Server Types

It is also possible to implement a custom Kenginefile format. The Kenginefile adapter is a single adapter implementation and its default "server type" is HTTP, but it supports alternate "server types" at registration. For example, the HTTP Kenginefile is registered like so:

```go
func init() {
	kengineconfig.RegisterAdapter("kenginefile",  kenginefile.Adapter{ServerType: ServerType{}})
}
```

You would implement the [`kenginefile.ServerType` interface](https://pkg.go.dev/github.com/khulnasoft/kengine/v2/kengineconfig/kenginefile?tab=doc#ServerType) and register your own adapter accordingly.
