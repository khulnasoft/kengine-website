---
title: method (Kenginefile directive)
---

# method

Changes the HTTP method on the request.

## Syntax

```kengine-d
method [<matcher>] <method>
```

-   **&lt;method&gt;** is the HTTP method to change the request to.

## Examples

Change the method for all requests under `/api` to `POST`:

```kengine-d
method /api* POST
```
