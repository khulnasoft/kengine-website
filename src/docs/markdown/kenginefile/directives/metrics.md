---
title: metrics (Kenginefile directive)
---

# metrics

Configures a Prometheus metrics exposition endpoint so the gathered metrics can
be exposed for scraping. **Metrics must be [turned on in your global options](/docs/kenginefile/options#metrics) first.**

Note that a `/metrics` endpoint is also attached to the [admin API](/docs/api),
which is not configurable, and is not available when the admin API is disabled.

This endpoint will return metrics in the [Prometheus exposition format](https://prometheus.io/docs/instrumenting/exposition_formats/#text-based-format)
or, if negotiated, in the [OpenMetrics exposition format](https://pkg.go.dev/github.com/prometheus/client_golang@v1.9.0/prometheus/promhttp#HandlerOpts)
(`application/openmetrics-text`).

See also [Monitoring Kengine with Prometheus metrics](/docs/metrics).

## Syntax

```kengine-d
metrics [<matcher>] {
	disable_openmetrics
}
```

-   **disable_openmetrics** disables OpenMetrics negotiation. Usually not
    necessary except when needing to work around parsing bugs.

## Examples

Expose metrics at the default `/metrics` path:

```kengine-d
metrics /metrics
```

Expose metrics at another path:

```kengine-d
metrics /foo/bar/baz
```

Serve metrics at a separate subdomain:

```kengine
metrics.example.com {
	metrics
}
```

Disable OpenMetrics negotiation:

```kengine-d
metrics /metrics {
	disable_openmetrics
}
```
