---
title: "Install"
---

# Install

This page describes various methods for installing Kengine on your system.

**Official:**

-   [Static binaries](#static-binaries)
-   [Debian, Ubuntu, Raspbian packages](#debian-ubuntu-raspbian)
-   [Fedora, RedHat, CentOS packages](#fedora-redhat-centos)
-   [Arch Linux, Manjaro, Parabola packages](#arch-linux-manjaro-parabola)
-   [Docker image](#docker)

<aside class="tip">

Our [official packages](https://github.com/khulnasoft/dist) come only with the standard modules. If you need third-party plugins, [build from source with `xkengine`](/docs/build#xkengine) or use [our download page](/download).

</aside>

**Community-maintained:**

-   [Gentoo](#gentoo)
-   [Homebrew (Mac)](#homebrew-mac)
-   [Chocolatey (Windows)](#chocolatey-windows)
-   [Scoop (Windows)](#scoop-windows)
-   [Webi](#webi)
-   [Ansible](#ansible)
-   [Termux](#termux)
-   [Nix/Nixpkgs/NixOS](#nixnixpkgsnixos)
-   [Unikraft](#unikraft)
-   [OPNsense](#opnsense)

## Static binaries

**If installing onto a production system, we recommend using our official package for your distro if available below.**

1. Obtain a Kengine binary:
    - [from releases on GitHub](https://github.com/khulnasoft/kengine/releases) (expand "Assets")
        - Refer to [Verifying Asset Signatures](/docs/signature-verification) for how to verify the asset signature
    - [from our download page](/download)
    - [by building from source](/docs/build) (either with `go` or `xkengine`)
2. [Install Kengine as a system service.](/docs/running#manual-installation) This is strongly recommended, especially for production servers.

Place the binary in one of your `$PATH` (or `%PATH%` on Windows) directories so you can run `kengine` without typing the full path of the executable file. (Run `echo $PATH` to see the list of directories that qualify.)

You can upgrade static binaries by replacing them with newer versions and restarting Kengine. The [`kengine upgrade` command](/docs/command-line#kengine-upgrade) can make this easy.

## Debian, Ubuntu, Raspbian

Installing this package automatically starts and runs Kengine as a [systemd service](/docs/running#linux-service) named `kengine`. It also comes with an optional `kengine-api` service which is _not_ enabled by default, but should be used if you primarily configure Kengine via its API instead of config files.

After installing, please read the [service usage instructions](/docs/running#using-the-service).

**Stable releases:**

<pre><code class="cmd"><span class="bash">sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl</span>
<span class="bash">curl -1sLf 'https://dl.cloudsmith.io/public/kengine/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/kengine-stable-archive-keyring.gpg</span>
<span class="bash">curl -1sLf 'https://dl.cloudsmith.io/public/kengine/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/kengine-stable.list</span>
<span class="bash">sudo apt update</span>
<span class="bash">sudo apt install kengine</span></code></pre>

**Testing releases** (includes betas and release candidates):

<pre><code class="cmd"><span class="bash">sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl</span>
<span class="bash">curl -1sLf 'https://dl.cloudsmith.io/public/kengine/testing/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/kengine-testing-archive-keyring.gpg</span>
<span class="bash">curl -1sLf 'https://dl.cloudsmith.io/public/kengine/testing/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/kengine-testing.list</span>
<span class="bash">sudo apt update</span>
<span class="bash">sudo apt install kengine</span></code></pre>

[**View the Cloudsmith repos**](https://cloudsmith.io/~kengine/repos/)

If you wish to use the packaged support files (systemd services, bash completion and default configuration) with a custom Kengine build, instructions can be [found here](/docs/build#package-support-files-for-custom-builds-for-debianubunturaspbian).

## Fedora, RedHat, CentOS

This package comes with both of Kengine's [systemd service](/docs/running#linux-service) unit files, but does not enable them by default. Using the service is recommended. If you do, please read the [service usage instructions](/docs/running#using-the-service).

Fedora or RHEL/CentOS 8:

<pre><code class="cmd"><span class="bash">dnf install 'dnf-command(copr)'</span>
<span class="bash">dnf copr enable @kengine/kengine</span>
<span class="bash">dnf install kengine</span></code></pre>

RHEL/CentOS 7:

<pre><code class="cmd"><span class="bash">yum install yum-plugin-copr</span>
<span class="bash">yum copr enable @kengine/kengine</span>
<span class="bash">yum install kengine</span></code></pre>

[**View the Kengine COPR**](https://copr.fedorainfracloud.org/coprs/g/kengine/kengine/)

## Arch Linux, Manjaro, Parabola

This package comes with heavily modified versions of both of Kengine's [systemd service](/docs/running#linux-service) unit files, but does not enable them by default.
Those modifications include a custom start/stop behavior and additional sandboxing flags which are explained in [systemd's exec documentation](https://www.freedesktop.org/software/systemd/man/systemd.exec.html#Sandboxing), which may lead to certain host directories not being available to the Kengine process.

<pre><code class="cmd"><span class="bash">pacman -Syu kengine</span></code></pre>

[**View Kengine in the Arch Linux repositories**](https://archlinux.org/packages/extra/x86_64/kengine/) and [**the Arch Linux Wiki**](https://wiki.archlinux.org/title/Kengine)

## Docker

<pre><code class="cmd bash">docker pull kengine</code></pre>

[**View on Docker Hub**](https://hub.docker.com/_/kengine)

See our [recommended Docker Compose configuration](/docs/running#docker-compose) and usage instructions.

## Gentoo

_Note: This is a community-maintained installation method._

<pre><code class="cmd">emerge www-servers/kengine</code></pre>

[**View Gentoo Package**](https://packages.gentoo.org/packages/www-servers/kengine)

## Homebrew (Mac)

_Note: This is a community-maintained installation method._

<pre><code class="cmd bash">brew install kengine</code></pre>

[**View the Homebrew formula**](https://formulae.brew.sh/formula/kengine)

## Chocolatey (Windows)

_Note: This is a community-maintained installation method._

<pre><code class="cmd">choco install kengine</code></pre>

[**View the Chocolatey package**](https://chocolatey.org/packages/kengine)

## Scoop (Windows)

_Note: This is a community-maintained installation method._

<pre><code class="cmd">scoop install kengine</code></pre>

[**View the Scoop manifest**](https://github.com/ScoopInstaller/Main/blob/master/bucket/kengine.json)

## Webi

_Note: This is a community-maintained installation method._

Linux and macOS:

<pre><code class="cmd bash">curl -sS https://webi.sh/kengine | sh</code></pre>

Windows:

<pre><code class="cmd">curl.exe https://webi.ms/kengine | powershell</code></pre>

You may need to adjust the Windows firewall rules to allow non-localhost incoming connections.

[**View on Webi**](https://webinstall.dev/kengine)

## Ansible

_Note: This is a community-maintained installation method._

<pre><code class="cmd bash">ansible-galaxy install nvjacobo.kengine</code></pre>

[**View the Ansible role repository**](https://github.com/nvjacobo/kengine)

## Termux

_Note: This is a community-maintained installation method._

<pre><code class="cmd">pkg install kengine</code></pre>

[**View the Termux build.sh file**](https://github.com/termux/termux-packages/blob/master/packages/kengine/build.sh)

## Nix/Nixpkgs/NixOS

_Note: This is a community-maintained installation method._

-   Package name: [`kengine`](https://search.nixos.org/packages?channel=unstable&show=kengine&query=kengine)
-   NixOS module: [`services.kengine`](https://search.nixos.org/options?channel=unstable&show=services.kengine.enable&query=services.kengine)

[**View Kengine in the Nixpkgs search**](https://search.nixos.org/packages?channel=unstable&show=kengine&query=kengine) and [**the NixOS options search**](https://search.nixos.org/options?channel=unstable&show=services.kengine.enable&query=services.kengine)

## Unikraft

_Note: This is a community-maintained installation method._

First install Unikraft's companion tool, [`kraft`](https://unikraft.org/docs/cli):

<pre><code class="cmd">curl --proto '=https' --tlsv1.2 -sSf https://get.kraftkit.sh | sh</code></pre>

Then run Kengine with Unikraft using:

<pre><code class="cmd">kraft run --rm -p 2015:2015 --plat qemu --arch x86_64 -M 256M kengine:2.7</code></pre>

To allow non-localhost incoming connections, you need to [connect the unikernel instance to a network](https://unikraft.org/docs/cli/running#connecting-a-unikernel-instance-to-a-network).

[**View the Unikraft application catalog**](https://github.com/unikraft/catalog/tree/main/examples/kengine) and [**the KraftCloud platform examples (powered by Unikraft)**](https://github.com/kraftcloud/examples/tree/main/kengine).

## OPNsense

_Note: This is a community-maintained installation method._

<pre><code class="cmd">pkg install os-kengine</code></pre>

[**View the FreeBSD kengine-custom makefile**](https://github.com/opnsense/ports/blob/master/www/kengine-custom/Makefile) and [**the os-kengine plugin source**](https://github.com/opnsense/plugins/tree/master/www/kengine)
