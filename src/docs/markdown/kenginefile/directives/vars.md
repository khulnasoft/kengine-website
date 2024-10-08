---
title: vars (Kenginefile directive)
---

# vars

Sets one or more variables to a particular value, to be used later in the request handling chain.

The primary way to access variables is with placeholders, which have the form `{vars.variable_name}`, or with the [`vars`](/docs/kenginefile/matchers#vars) and [`vars_regexp`](/docs/kenginefile/matchers#vars_regexp) request matchers. You may use variables with the [`templates`](templates) directive using the `placeholder` function, for example: `{{placeholder "http.vars.variable_name"}}`

As a special case, it's possible to override the variable named `http.auth.user.id`, which is stored in the replacer, to update the `user_id` field in access logs.

## Syntax

```kengine-d
vars [<matcher>] [<name> <value>] {
    <name> <value>
    ...
}
```

-   **&lt;name&gt;** is the variable name to set.

-   **&lt;value&gt;** is the value of the variable.

    The value will be type converted if possible; `true` and `false` will be converted to boolean types, and numeric values will be converted to integer or float accordingly. To avoid this conversion, you may wrap the output with [quotes](/docs/kenginefile/concepts#tokens-and-quotes) and they will stay strings.

## Examples

To set a single variable, the value being conditional based on the request path, then responding with the value:

```kengine
example.com {
	vars /foo* isFoo "yep"
	vars isFoo "nope"

	respond {vars.isFoo}
}
```

To set multiple variables, each converted to the appropriate scalar type:

```kengine-d
vars {
	# boolean
	abc true

	# integer
	def 1

	# float
	ghi 2.3

	# string
	jkl "example"
}
```
