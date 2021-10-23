# VictoriaMetrics

Tested on: Debian 11

This role sets up VictoriaMetrics in standalone mode. It also sets up HAProxy to make
it accessible with TLS client certificates.

## Note
This role assumes that TLS certificates will be provisioned at:
 * `/etc/haproxy/victoriametrics.pem` (pem with both cert and key)
 * `/etc/haproxy/victoriametrics-ca.pem`
 * `/etc/haproxy/victoriametrics-crl.pem`

## Configuration
|Var|Default value|Description|
|---|-------------|-----------|
|victoriametrics_enable_graphite|`False`|Whether to turn on the Graphite API|
|victoriametrics_graphite_addr|`127.0.0.1:2003`|Where to bind for Graphite, if enabled|
|victoriametrics_enable_influx|`False`|Whether to enable the InfluxDB API|
|victoriametrics_influx_url|`http://127.0.0.1:8428/write`|Where to bind for Influx, if enabled|
|victoriametrics_http_addr|`127.0.0.1:8428`|Where to bind for the normal HTTP API|
|victoriametrics_mem_pct|`60`|Memory target (in percent of total memory) to use for data|
|victoriametrics_retention_period|`1`|How long to retain data (in months)|
|victoriametrics_haproxy_port|`443`|Which port to bind the HAProxy frontend on (public!)|
|victoriametrics_haproxy_vmport|`55042`|Which port to bind the HAProxy internal frontend on (private)|
|victoriametrics_haproxy_domain|`ship.example.org`|Which domain to use for TLS SNI proxying (see below)|
|victoriametrics_haproxy_default_backend|`127.0.0.1:444`|Address of default backend to use on public frontend (see below)|

## Features

### HAProxy TLS SNI
This role ships a HAProxy configuration (compatible with SOSETH/haproxy) that does SNI-based proxying:
 * All requests that do not match `victoriametrics_haproxy_domain` get forwarded to `victoriametrics_haproxy_default_backend` (with proxy protocol v2)
 * All requests that _do_ match are forwarded to the VictoriaMetrics HTTP API after presenting a client certificate