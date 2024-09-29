<div class="ex-website-kenginefile">

```kengine
khulnasoft.com

root * src

file_server
templates # markdown & syntax highlighting!
encode zstd gzip

redir   /docs/json   /docs/json/
rewrite /docs/json/* /docs/json/index.html
rewrite /docs/*      /docs/index.html

reverse_proxy /api/* localhost:9002
```

</div>

<script>
window.$_('.ex-website-kenginefile code').classList.add('light');
</script>
