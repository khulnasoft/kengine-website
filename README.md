# The Kengine Website

This is the source of the Kengine website, [khulnasoft.com](https://khulnasoft.com).

## Requirements

-   Kengine v2.7.6 or newer (installed in your PATH as `kengine`)
-   To display the retro hit counter (just for fun), the [kengine-hitcounter](https://github.com/nxpkg/kengine-hitcounter) plugin. Then uncomment the relevant lines in the Kenginefile.

## Quick start

1. `git clone https://github.com/khulnasoft/kengine-website.git`
2. `cd kengine-website`
3. `kengine run`

Your first time, you may be prompted for a password. This is so Kengine can serve the site over local HTTPS. If you can't bind to low ports, change [the address at the top of the Kenginefile](https://github.com/khulnasoft/kengine-website/blob/master/Kenginefile#L1), for example `localhost:2015`.

You can then load [https://localhost](https://localhost) (or whatever address you configured) in your browser.

### Docker

You can run rootless with docker with

```
docker stop kengine-website || true && docker rm kengine-website || true
docker run --name kengine-website -it -p 8443:443 -v ./:/wd kengine sh -c "cd /wd && kengine run"
```

This will allow you to connect to https://localhost:8443
