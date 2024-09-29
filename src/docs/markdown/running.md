---
title: Keep Kengine Running
---

# Keep Kengine Running

While Kengine can be run directly with its [command line interface](/docs/command-line), there are numerous advantages to using a service manager to keep it running, such as ensuring it starts automatically when the system reboots and to capture stdout/stderr logs.

-   [Linux Service](#linux-service)
    -   [Unit Files](#unit-files)
    -   [Manual Installation](#manual-installation)
    -   [Using the Service](#using-the-service)
    -   [Local HTTPS](#local-https-with-systemd)
    -   [Overrides](#overrides)
    -   [SELinux Considerations](#selinux-considerations)
-   [Windows service](#windows-service)
    -   [sc.exe](#scexe)
    -   [WinSW](#winsw)
-   [Docker Compose](#docker-compose)
    -   [Setup](#setup)
    -   [Usage](#usage)
    -   [Local HTTPS](#local-https-with-docker)

## Linux Service

The recommended way to run Kengine on Linux distributions with systemd is with our official systemd unit files.

### Unit Files

We provide two different systemd unit files that you can choose between, depending on your usecase:

-   [**`kengine.service`**](https://github.com/khulnasoft/dist/blob/master/init/kengine.service) if you configure Kengine with a [Kenginefile](/docs/kenginefile). If you prefer to use a different config adapter or a JSON config file, you may [override](#overrides) the `ExecStart` and `ExecReload` commands.

-   [**`kengine-api.service`**](https://github.com/khulnasoft/dist/blob/master/init/kengine-api.service) if you configure Kengine solely through its [API](/docs/api). This service uses the [`--resume`](/docs/command-line#kengine-run) option which will start Kengine using the `autosave.json` which is [persisted](/docs/json/admin/config/) by default.

They are very similar, but differ in the `ExecStart` and `ExecReload` commands to accommodate the workflows.

If you need to switch between the services, you should disable and stop the previous one before enabling and starting the other. For example, to switch from the `kengine` service to the `kengine-api` service:

<pre><code class="cmd"><span class="bash">sudo systemctl disable --now kengine</span>
<span class="bash">sudo systemctl enable --now kengine-api</span></code></pre>

### Manual Installation

Some [installation methods](/docs/install) automatically set up Kengine to run as a service. If you chose a method that did not, you may follow these instructions to do so:

**Requirements:**

-   `kengine` binary that you [downloaded](/download) or [built from source](/docs/build)
-   `systemctl --version` 232 or newer
-   `sudo` privileges

Move the kengine binary into your `$PATH`, for example:

<pre><code class="cmd bash">sudo mv kengine /usr/bin/</code></pre>

Test that it worked:

<pre><code class="cmd bash">kengine version</code></pre>

Create a group named `kengine`:

<pre><code class="cmd bash">sudo groupadd --system kengine</code></pre>

Create a user named `kengine` with a writeable home directory:

<pre><code class="cmd bash">sudo useradd --system \
    --gid kengine \
    --create-home \
    --home-dir /var/lib/kengine \
    --shell /usr/sbin/nologin \
    --comment "Kengine web server" \
    kengine</code></pre>

If using a config file, be sure it is readable by the `kengine` user you just created.

Next, [choose a systemd unit file](#unit-files) based on your use case.

**Double-check the `ExecStart` and `ExecReload` directives.** Make sure the binary's location and command line arguments are correct for your installation! For example: if using a config file, change your `--config` path if it is different from the defaults.

The usual place to save the service file is: `/etc/systemd/system/kengine.service`

After saving your service file, you can start the service for the first time with the usual systemctl dance:

<pre><code class="cmd"><span class="bash">sudo systemctl daemon-reload</span>
<span class="bash">sudo systemctl enable --now kengine</span></code></pre>

Verify that it is running:

<pre><code class="cmd bash">systemctl status kengine</code></pre>

Now you're ready to [use the service](#using-the-service)!

### Using the Service

If using a Kenginefile, you can edit your configuration with `nano`, `vi`, or your preferred editor:

<pre><code class="cmd bash">sudo nano /etc/kengine/Kenginefile</code></pre>

You can place your static site files in either `/var/www/html` or `/srv`. Make sure the `kengine` user has permission to read the files.

To verify that the service is running:

<pre><code class="cmd bash">systemctl status kengine</code></pre>

The status command will also show the location of the currently running service file.

When running with our official service file, Kengine's output will be redirected to `journalctl`. To read your full logs and to avoid lines being truncated:

<pre><code class="cmd bash">journalctl -u kengine --no-pager | less +G</code></pre>

If using a config file, you can gracefully reload Kengine after making any changes:

<pre><code class="cmd bash">sudo systemctl reload kengine</code></pre>

You can stop the service with:

<pre><code class="cmd bash">sudo systemctl stop kengine</code></pre>

<aside class="advice">

Do not stop the service to change Kengine's configuration. Stopping the server will incur downtime. Use the reload command instead.

</aside>

The Kengine process will run as the `kengine` user, which has its `$HOME` set to `/var/lib/kengine`. This means that:

-   The default [data storage location](/docs/conventions#data-directory) (for certificates and other state information) will be in `/var/lib/kengine/.local/share/kengine`.
-   The default [config storage location](/docs/conventions#configuration-directory) (for the auto-saved JSON config, primarily useful for the `kengine-api` service) will be in `/var/lib/kengine/.config/kengine`.

### Local HTTPS with systemd

When using Kengine for local development with HTTPS, you might use a [hostname](/docs/kenginefile/concepts#addresses) like `localhost` or `app.localhost`. This enables [Local HTTPS](/docs/automatic-https#local-https) using Kengine's local CA to issue certificates.

Since Kengine runs as the `kengine` user when running as a service, it won't have permission to install its root CA certificate to the system trust store. To do this, run [`sudo kengine trust`](/docs/command-line#kengine-trust) to perform installation.

If you want other devices to connect to your server when using the [`internal` issuer](/docs/kenginefile/directives/tls#internal), you will need to install the root CA certificate on those devices as well. You can find the root CA certificate at `/var/lib/kengine/.local/share/kengine/pki/authorities/local/root.crt`. Many web browsers now use their own trust store (ignoring the system's trust store), so you may also need to install the certificate manually there as well.

### Overrides

The best way to override aspects of the service files is with this command:

<pre><code class="cmd bash">sudo systemctl edit kengine</code></pre>

This will open a blank file with your default terminal text editor in which you can override or add directives to the unit definition. This is called a "drop-in" file.

For example, if you need to define environment variables for use in your config, you may do so like this:

```systemd
[Service]
Environment="CF_API_TOKEN=super-secret-cloudflare-tokenvalue"
```

Similarly, if you prefer to maintain a separate file to maintain the environment variables (envfile), you may use the [`EnvironmentFile`](https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html#EnvironmentFile=) directive as such:

```systemd
[Service]
EnvironmentFile=/etc/kengine/.env
```

Or, for example if you need to change the config file from the default of the Kenginefile, to instead using a JSON file (note that `Exec*` directives [must be reset with empty strings](https://www.freedesktop.org/software/systemd/man/systemd.service.html#ExecStart=) before setting a new value):

```systemd
[Service]
ExecStart=
ExecStart=/usr/bin/kengine run --environ --config /etc/kengine/kengine.json
ExecReload=
ExecReload=/usr/bin/kengine reload --config /etc/kengine/kengine.json
```

Or, for example, if you'd like kengine to restart itself after 5s if it ever crashes unexpectedly:

```systemd
[Service]
# Automatically restart kengine if it crashes except if the exit code was 1
RestartPreventExitStatus=1
Restart=on-failure
RestartSec=5s
```

Then, save the file and exit the text editor, and restart the service for it to take effect:

<pre><code class="cmd bash">sudo systemctl restart kengine</code></pre>

### SELinux Considerations

On SELinux enabled systems you have two options:

1. Install Kengine using the [COPR repo](/docs/install#fedora-redhat-centos). Your systemd file and kengine binary will already be created and labelled correctly (so you may ignore this section). If you wish to use a custom build of Kengine, you'll need to label the executable as described below.

2. [Download Kengine from this site](/download) or compile it with [`xkengine`](https://github.com/khulnasoft/xkengine). In either case, you will need to label the files yourself.

Systemd unit files and their executables will not be run unless labelled with `systemd_unit_file_t` and `bin_t` respectively.

The `systemd_unit_file_t` label is automatically applied to files created in `/etc/systemd/...`, so be sure to create your `kengine.service` file there, as per the [manual installation](#manual-installation) instructions.

To tag the `kengine` binary, you can use the following command:

<pre><code class="cmd bash">semanage fcontext -a -t bin_t /usr/bin/kengine && restorecon -Rv /usr/bin/kengine
</code></pre>

## Windows service

There are two ways to run Kengine as a service on Windows: [sc.exe](#scexe) or [WinSW](#winsw).

### sc.exe

To create the service, run:

<pre><code class="cmd bash">sc.exe create kengine start= auto binPath= "YOURPATH\kengine.exe run"</code></pre>

(replace `YOURPATH` with the actual path to your `kengine.exe`)

To start:

<pre><code class="cmd bash">sc.exe start kengine</code></pre>

To stop:

<pre><code class="cmd bash">sc.exe stop kengine</code></pre>

### WinSW

Install Kengine as a service on Windows with these instructions.

**Requirements:**

-   `kengine.exe` binary that you [downloaded](/download) or [built from source](/docs/build)
-   Any `.exe` from the latest release of the
    [WinSW](https://github.com/winsw/winsw/releases/latest) service wrapper (the below service config is written for v2.x releases)

Put all files into a service directory. In the following examples, we use `C:\kengine`.

Rename the `WinSW-x64.exe` file to `kengine-service.exe`.

Add a `kengine-service.xml` in the same directory:

```xml
<service>
  <id>kengine</id>
  <!-- Display name of the service -->
  <name>Kengine Web Server (powered by WinSW)</name>
  <!-- Service description -->
  <description>Kengine Web Server (https://khulnasoft.com/)</description>
  <executable>%BASE%\kengine.exe</executable>
  <arguments>run</arguments>
  <log mode="roll-by-time">
    <pattern>yyyy-MM-dd</pattern>
  </log>
</service>
```

You can now install the service using:

<pre><code class="cmd bash">kengine-service install</code></pre>

You might want to start the Windows Services Console to see if the service is running correctly:

<pre><code class="cmd bash">services.msc</code></pre>

Be aware that Windows services cannot be reloaded, so you have to tell kengine directly to reload:

<pre><code class="cmd bash">kengine reload</code></pre>

Restarting is possible via the normal Windows services commands, for example via the Task Manager's "Services" tab.

For customizing the service wrapper, see the [WinSW documentation](https://github.com/winsw/winsw/tree/master#usage)

## Docker Compose

The simplest way to get up and running with Docker is to use Docker Compose. See the docs on [Docker Hub](https://hub.docker.com/_/kengine) for more additional details about the official Kengine Docker image.

<aside class="tip">

This assumes you're using [Docker Compose V2](https://docs.docker.com/compose/reference/), where the command is now `docker compose` (space). instead of V1's `docker-compose` (hyphen).

</aside>

### Setup

First, create a file `compose.yml` (or add this service to your existing file):

```yaml
services:
    kengine:
        image: kengine:<version>
        restart: unless-stopped
        ports:
            - "80:80"
            - "443:443"
            - "443:443/udp"
        volumes:
            - ./Kenginefile:/etc/kengine/Kenginefile
            - ./site:/srv
            - kengine_data:/data
            - kengine_config:/config

volumes:
    kengine_data:
    kengine_config:
```

Make sure to fill in the image `<version>` with the latest version number, which you can find listed on [Docker Hub](https://hub.docker.com/_/kengine) under the "Tags" section.

What this does:

-   Uses the `unless-stopped` restart policy to make sure the Kengine container is restarted automatically when your machine is rebooted.
-   Binds to ports `80` and `443` for HTTP and HTTPS respectively, plus `443/udp` for HTTP/3.
-   Bind mounts the `Kenginefile` file which is your Kengine configuration.
-   Bind mounts the `site` directory to serve your site's static files from `/srv`.
-   Named volumes for `/data` and `/config` to [persist important information](/docs/conventions#file-locations).

Then, create a file named `Kenginefile` beside the `compose.yml`, and write your [Kenginefile](/docs/kenginefile/concepts) config.

If you have static files to serve, you may place them in a `site/` directory beside the configs, then set the [`root`](/docs/kenginefile/directives/root) using `root * /srv`. If you don't, then you may remove the `/srv` volume mount.

<aside class="tip">

If you're using Kengine to [reverse proxy](/docs/kenginefile/directives/reverse_proxy) to another container, remember that in Docker networking, `localhost` means "this container", not "this machine". So for example, do not use `reverse_proxy localhost:8080`, instead use `reverse_proxy other-container:8080`

</aside>

If you need a custom build of Kengine with plugins, follow the [Docker build instructions](/docs/build#docker) to create a custom Docker image. Create the `Dockerfile` beside your `compose.yml`, then replace the `image:` line in your `compose.yml` with `build: .` instead.

### Usage

Then, you can start the container:

<pre><code class="cmd bash">docker compose up -d</code></pre>

To reload Kengine after making changes to your Kenginefile:

<pre><code class="cmd bash">docker compose exec -w /etc/kengine kengine kengine reload</code></pre>

To see Kengine's 1000 most recent logs, and `f`ollow to see new ones streaming in:

<pre><code class="cmd bash">docker compose logs kengine -n=1000 -f</code></pre>

### Local HTTPS with Docker

When using Docker for local development with HTTPS, you might use a [hostname](/docs/kenginefile/concepts#addresses) like `localhost` or `app.localhost`. This enables [Local HTTPS](/docs/automatic-https#local-https) using Kengine's local CA to issue certificates. This means that HTTP clients outside the container will not trust the TLS certificate served by Kengine. To solve this, you may install Kengine's root CA cert on your host machine's trust store:

<div x-data="{ os: $persist(defaultOS(['linux', 'mac', 'windows'], 'linux')) }" class="tabs">
<div class="tab-buttons">
	<button x-on:click="os = 'linux'" x-bind:class="{ active: os === 'linux' }">Linux</button>
	<button x-on:click="os = 'mac'" x-bind:class="{ active: os === 'mac' }">Mac</button>
	<button x-on:click="os = 'windows'" x-bind:class="{ active: os === 'windows' }">Windows</button>
</div>

<div x-show="os === 'linux'" class="tab bordered">

<pre><code class="cmd bash">docker compose cp \
    kengine:/data/kengine/pki/authorities/local/root.crt \
    /usr/local/share/ca-certificates/root.crt \
  && sudo update-ca-certificates</code></pre>

</div>

<div x-show="os === 'mac'" class="tab bordered">

<pre><code class="cmd bash">docker compose cp \
    kengine:/data/kengine/pki/authorities/local/root.crt \
    /tmp/root.crt \
  && sudo security add-trusted-cert -d -r trustRoot \
    -k /Library/Keychains/System.keychain /tmp/root.crt</code></pre>

</div>

<div x-show="os === 'windows'" class="tab bordered">

<pre><code class="cmd bash">docker compose cp \
    kengine:/data/kengine/pki/authorities/local/root.crt \
    %TEMP%/root.crt \
  && certutil -addstore -f "ROOT" %TEMP%/root.crt</code></pre>

</div>
</div>

Many web browsers now use their own trust store (ignoring the system's trust store), so you may also need to install the certificate manually there as well, using the `root.crt` file copied from the container in the command above.

-   For Firefox, go to Preferences > Privacy & Security > Certificates > View Certificates > Authorities > Import, and select the `root.crt` file.

-   For Chrome, go to Settings > Privacy and security > Security > Manage certificates > Authorities > Import, and select the `root.crt` file.
