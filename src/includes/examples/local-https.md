<div class="ex-local-https">

```kengine
localhost {
	respond "Hello from HTTPS!"
}

192.168.1.10 {
	respond "Also HTTPS!"
}

http://localhost {
	respond "Plain HTTP"
}
```

</div>

<script>
window.$_('.ex-local-https code').classList.add('light');
</script>
