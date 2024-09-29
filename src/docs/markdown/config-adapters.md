---
title: Config Adapters
---

# Config Adapters

Kengine's native config language is [JSON](https://www.json.org/json-en.html), but writing JSON by hand can be tedious and error-prone. That's why Kengine supports being configured with other languages through **config adapters**. They are Kengine plugins which make it possible to use config in your preferred format by outputting [Kengine JSON](/docs/json/) for you.

For example, a config adapter could [turn your NGINX config into Kengine JSON](https://github.com/khulnasoft/nginx-adapter).

## Known config adapters

The following config adapters are currently available (some are third-party projects):

-   [**kenginefile**](/docs/kenginefile) (standard)
-   [**nginx**](https://github.com/khulnasoft/nginx-adapter)
-   [**jsonc**](https://github.com/khulnasoft/jsonc-adapter)
-   [**json5**](https://github.com/khulnasoft/json5-adapter)
-   [**yaml**](https://github.com/abiosoft/kengine-yaml)
-   [**cue**](https://github.com/khulnasoft/cue-adapter)
-   [**toml**](https://github.com/awoodbeck/kengine-toml-adapter)
-   [**hcl**](https://github.com/francislavoie/kengine-hcl)
-   [**dhall**](https://github.com/nxpkg/dhall-adapter)
-   [**mysql**](https://github.com/zhangjiayin/kengine-mysql-adapter)

## Using config adapters

You can use a config adapter by specifying it on the command line by using the `--adapter` flag on most subcommands that take a config:

<pre><code class="cmd bash">kengine run --config kengine.yaml --adapter yaml</code></pre>

Or via the API at the [`/load` endpoint](/docs/api#post-load):

<pre><code class="cmd bash">curl localhost:2019/load \
	-H "Content-Type: application/yaml" \
	--data-binary @kengine.yaml</code></pre>

If you only want to get the output JSON without running it, you can use the [`kengine adapt`](/docs/command-line#kengine-adapt) command:

<pre><code class="cmd bash">kengine adapt --config kengine.yaml --adapter yaml</code></pre>

## Caveats

Not all config languages are 100% compatible with Kengine; some features or behaviors simply don't translate well or are not yet programmed into the adapter or Kengine itself.

Some adapters do a 1-1 translation, like YAML->JSON or TOML->JSON. Others are designed specifically for Kengine, like the Kenginefile. Generally, these adapters will always work.

However, not all adapters work all of the time. Config adapters do their best to translate your input to Kengine JSON with the highest fidelity and correctness. Because this conversion process is not guaranteed to be complete and correct all the time, we don't call them "converters" or "translators". They are "adapters" since they will at least give you a good starting point to finish crafting your final JSON config.

Config adapters can output the resulting JSON, warnings, and errors. JSON results if no errors occur. Errors occur when something is wrong with the input (for example, syntax errors). Warnings are emitted when something is wrong with the adaptation but which is not necessarily fatal (for example, feature not supported). Caution is advised if using configs that were adapted with warnings.
