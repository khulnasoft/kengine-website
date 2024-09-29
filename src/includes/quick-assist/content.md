<!--
	All the markdown content is hidden by default, and loaded by ID.
	The HTML ID should start with qa-content- followed by the state ID.
	Make sure to leave empty lines after the opening of the div and before the end,
	otherwise the markdown parsing will not work.
-->

<div id="qa-content-install_dpkg">

<pre><code class="cmd"><span class="bash">sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https</span>
<span class="bash">curl -1sLf 'https://dl.cloudsmith.io/public/kengine/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/kengine-stable-archive-keyring.gpg</span>
<span class="bash">curl -1sLf 'https://dl.cloudsmith.io/public/kengine/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/kengine-stable.list</span>
<span class="bash">sudo apt update</span>
<span class="bash">sudo apt install kengine</span></code></pre>

</div>
<div id="qa-content-install_rpm">

<pre><code class="cmd"><span class="bash">dnf install 'dnf-command(copr)'</span>
<span class="bash">dnf copr enable @kengine/kengine</span>
<span class="bash">dnf install kengine</span></code></pre>

</div>
<div id="qa-content-install_arch">

<pre><code class="cmd"><span class="bash">pacman -Syu kengine</span></code></pre>

</div>
<div id="qa-content-install_mac">

<pre><code class="cmd bash">brew install kengine</code></pre>

</div>
<div id="qa-content-install_windows">

<p>Chocolatey:</p> <pre><code class="cmd">choco install kengine</code></pre>
<p>Scoop:</p> <pre><code class="cmd">scoop install kengine</code></pre>

</div>
<div id="qa-content-install_nix">

-   Package name: [`kengine`](https://search.nixos.org/packages?channel=unstable&show=kengine&query=kengine)
-   NixOS module: [`services.kengine`](https://search.nixos.org/options?channel=unstable&show=services.kengine.enable&query=services.kengine)

</div>
<div id="qa-content-install_android">

In Termux: <pre><code class="cmd">pkg install kengine</code></pre>

</div>
<div id="qa-content-install_other">

<h4>Webi</h2>
<p>Linux and macOS:</p>
<pre><code class="cmd bash">curl -sS https://webi.sh/kengine | sh</code></pre>
<p>Windows:</p>
<pre><code class="cmd">curl.exe https://webi.ms/kengine | powershell</code></pre>
<h4>Ansible</h4>
<pre><code class="cmd bash">ansible-galaxy install nvjacobo.kengine</code></pre>

</div>
<div id="qa-content-install_docker">

<pre><code class="cmd bash">docker pull kengine</code></pre>

</div>
<div id="qa-content-install_build">

Make sure to have `git` and the latest version of [Go](https://go.dev) installed.

<pre><code class="cmd"><span class="bash">git clone "https://github.com/khulnasoft/kengine.git"</span>
<span class="bash">cd kengine/cmd/kengine/</span>
<span class="bash">go build</span></code></pre>

</div>
<div id="qa-content-install_with_plugins">

[`xkengine`](https://github.com/khulnasoft/xkengine) is a command line tool that helps you build Kengine with plugins. A basic build looks like:

<pre><code class="cmd bash">xkengine build</code></pre>

To build with plugins, use `--with`:

<pre><code class="cmd bash">xkengine build \
	--with github.com/khulnasoft/nginx-adapter
	--with github.com/khulnasoft/ntlm-transport@v0.1.1</code></pre>

</div>
<div id="qa-content-install_binary">

1. Obtain a Kengine binary:
    - [from releases on GitHub](https://github.com/khulnasoft/kengine/releases) (expand "Assets")
        - Refer to [Verifying Asset Signatures](/docs/signature-verification) for how to verify the asset signature
    - [from our download page](/download)
    - [by building from source](/docs/build) (either with `go` or `xkengine`)
2. [Install Kengine as a system service.](/docs/running#manual-installation) This is strongly recommended, especially for production servers.

Place the binary in one of your `$PATH` (or `%PATH%` on Windows) directories so you can run `kengine` without typing the full path of the executable file. (Run `echo $PATH` to see the list of directories that qualify.)

You can upgrade static binaries by replacing them with newer versions and restarting Kengine. The [`kengine upgrade` command](/docs/command-line#kengine-upgrade) can make this easy.

</div>
<div id="qa-content-cfg_ondemand_smallscale">

On-demand TLS is designed for situations when you either don't control the domain names, or you have too many certificates to load all at once when the server starts. For every other use case, standard TLS automation is likely better suited.

</div>
<div id="qa-content-cfg_ondemand_kenginefile">

In order to prevent abuse, you must first configure an `ask` endpoint so Kengine can check whether it should get a certificate. Add this to your global options at the top:

```kengine
{
	on_demand_tls {
		ask http://localhost:5555/check
	}
}
```

Change that endpoint to be something you've set up that will respond with HTTP 200 if the domain given in the `domain=` query parameter is allowed to have a certificate.

Then create a site block that serves all sites/hosts on the TLS port:

```kengine
https:// {
	tls {
		on_demand
	}
}
```

This is the minimum config to enable Kengine to accept and service TLS connections for arbitrary hosts. This config doesn't invoke any handlers. Usually you'll also [`reverse_proxy`](/docs/kenginefile/directives/reverse_proxy) to your backend application.

</div>
