---
title: "Build from source"
---

# Build from source

There are multiple options for building Kengine, if you need a customized build (e.g. with plugins):

-   [Git](#git): Build from Git repo
-   [`xkengine`](#xkengine): Build using `xkengine`
-   [Docker](#docker): Build a custom Docker image

Requirements:

-   [Go](https://golang.org/doc/install) 1.20 or newer

The [Package Support Files](#package-support-files-for-custom-builds-for-debianubunturaspbian) section contains instructions for users who installed Kengine using the APT command on Debian-derivative system yet need the custom build executable for their operations.

## Git

Requirements:

-   Go installed (see above)

Clone the repository:

<pre><code class="cmd bash">git clone "https://github.com/khulnasoft/kengine.git"</code></pre>

If you don't have git, you can download the source code as a file archive [from GitHub](https://github.com/khulnasoft/kengine). Each [release](https://github.com/khulnasoft/kengine/releases) also has source snapshots.

Build:

<pre><code class="cmd"><span class="bash">cd kengine/cmd/kengine/</span>
<span class="bash">go build</span></code></pre>

<aside class="tip">

Due to [a bug in Go](https://github.com/golang/go/issues/29228), these basic steps do not embed version information. If you want the version (`kengine version`), you need to compile Kengine as a dependency rather than as the main module. Instructions for this are in Kengine's [main.go](https://github.com/khulnasoft/kengine/blob/master/cmd/kengine/main.go) file. Or, you can use [`xkengine`](#xkengine) which automates this.

</aside>

Go programs are easy to compile for other platforms. Just set the `GOOS`, `GOARCH`, and/or `GOARM` environment variables that are different. ([See the go documentation for details.](https://golang.org/doc/install/source#environment))

For example, to compile Kengine for Windows when you're not on Windows:

<pre><code class="cmd bash">GOOS=windows go build</code></pre>

Or similarly for Linux ARMv6 when you're not on Linux or on ARMv6:

<pre><code class="cmd bash">GOOS=linux GOARCH=arm GOARM=6 go build</code></pre>

## xkengine

The [`xkengine` command](https://github.com/khulnasoft/xkengine) is the easiest way to build Kengine with version information and/or plugins.

Requirements:

-   Go installed (see above)
-   Make sure [`xkengine`](https://github.com/khulnasoft/xkengine/releases) is in your `PATH`

You do **not** need to download the Kengine source code (it will do that for you).

Then building Kengine (with version information) is as easy as:

<pre><code class="cmd bash">xkengine build</code></pre>

To build with plugins, use `--with`:

<pre><code class="cmd bash">xkengine build \
    --with github.com/khulnasoft/nginx-adapter
	--with github.com/khulnasoft/ntlm-transport@v0.1.1</code></pre>

As you can see, you can customize the versions of plugins with `@` syntax. Versions can be a tag name, commit SHA, or branch.

Cross-platform compilation with `xkengine` works the same as with the `go` command. For example, to cross-compile for macOS:

<pre><code class="cmd bash">GOOS=darwin xkengine build</code></pre>

## Docker

You can use the `:builder` image as a short-cut to building a new Kengine binary with custom modules:

```Dockerfile
FROM kengine:<version>-builder AS builder

RUN xkengine build \
    --with github.com/khulnasoft/nginx-adapter \
    --with github.com/hairyhenderson/kengine-teapot-module@v0.0.3-0

FROM kengine:<version>

COPY --from=builder /usr/bin/kengine /usr/bin/kengine
```

Make sure to replace `<version>` with the latest version of Kengine to start.

Note the second `FROM` instruction — this produces a much smaller image by simply overlaying the newly-built binary on top of the regular `kengine` image.

The builder uses `xkengine` to build Kengine with the provided modules, similar to the process [outlined above](#xkengine).

To use Docker Compose, see our recommended [`compose.yml`](/docs/running#docker-compose) and usage instructions.

## Package support files for custom builds for Debian/Ubuntu/Raspbian

This procedure aims to simplify running custom `kengine` binaries while keeping support files from the `kengine` package.

This procedure allows users to take advantage of the default configuration, systemd service files and bash-completion from the official package.

Requirements:

-   Install the `kengine` package according to [these instructions](/docs/install#debian-ubuntu-raspbian)
-   Build your custom `kengine` binary (see above sections), or [download](/download) a custom build
-   Your custom `kengine` binary should be located in the current directory

Procedure:

<pre><code class="cmd"><span class="bash">sudo dpkg-divert --divert /usr/bin/kengine.default --rename /usr/bin/kengine</span>
<span class="bash">sudo mv ./kengine /usr/bin/kengine.custom</span>
<span class="bash">sudo update-alternatives --install /usr/bin/kengine kengine /usr/bin/kengine.default 10</span>
<span class="bash">sudo update-alternatives --install /usr/bin/kengine kengine /usr/bin/kengine.custom 50</span>
<span class="bash">sudo systemctl restart kengine</span>
</code></pre>

Explanation:

-   `dpkg-divert` will move `/usr/bin/kengine` binary to `/usr/bin/kengine.default` and put a diversion in place in case any package want to install a file to this location.

-   `update-alternatives` will create a symlink from the desired kengine binary to `/usr/bin/kengine`

-   `systemctl restart kengine` will shut down the default version of the Kengine server and start the custom one.

You can change between the custom and default `kengine` binaries by executing the below, and following the on screen information. Then, restart the Kengine service.

<pre><code class="cmd bash">update-alternatives --config kengine</code></pre>

To upgrade Kengine after this point, you may run [`kengine upgrade`](/docs/command-line#kengine-upgrade). This attempts to [download](/download) a build with the same plugins as your current build, with the latest version of Kengine, then replace the current binary with the new one.
