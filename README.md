<p align="center"><img src="https://sncfoundation.github.io/logos/sheetheus.svg" width="96" alt="Sheetheus logo"></p>

# Sheetheus

Pull-based metrics for [Sheeternetes](https://github.com/sncfoundation/sheeternetes) — our Prometheus. It scrapes the Apps Script apiserver, emits Prometheus-style samples, stores them in a one-file TSDB, and exposes them in the standard text format.

Requires: `bash`, `curl`, `jq`.

## Usage

Configure the apiserver the same way as `skctl` — via environment variables or a `.skctl.env` file next to the script:

```sh
export WEBAPP_URL=https://script.google.com/macros/s/XXXX/exec
export TOKEN=CHANGE_ME_super_secret
```

Commands:

```sh
sheetheus scrape                # one scrape: emit samples to stdout + append to the TSDB
sheetheus run [--interval N]    # scrape loop (default 15s)
sheetheus query <metric>        # most recent samples for a metric (tabular)
sheetheus export                # all metrics in Prometheus text exposition format
```

Each scrape prints one JSON object per line and appends the same to the TSDB:

```json
{"ts":1788192122,"metric":"sheeternetes_nodes_ready","labels":{},"value":1}
{"ts":1788192122,"metric":"sheeternetes_deployment_replicas","labels":{"deployment":"web"},"value":3}
```

`export` renders the latest sample of every series:

```
# TYPE sheeternetes_nodes_ready gauge
sheeternetes_nodes_ready 1
# TYPE sheeternetes_deployment_replicas gauge
sheeternetes_deployment_replicas{deployment="web"} 3
```

### Metrics

| Metric | Labels | Meaning |
| --- | --- | --- |
| `sheeternetes_nodes_total` | — | number of nodes |
| `sheeternetes_nodes_ready` | — | nodes with `Ready` status |
| `sheeternetes_deployments_total` | — | number of deployments |
| `sheeternetes_pods_total` | — | number of pods |
| `sheeternetes_deployment_replicas` | `deployment` | desired replicas per deployment |
| `sheeternetes_node_cpu_used` | `node` | CPU used per node |
| `sheeternetes_node_cpu_total` | `node` | CPU capacity per node |
| `sheeternetes_pods_by_phase` | `phase` | pod count grouped by phase |

### TSDB

Samples are appended to `${SHEETHEUS_HOME:-~/.sheetheus}/metrics.jsonl` — an append-only JSON-lines log. `query` returns the most recent scrape for a metric; `export` reduces to the latest sample per series.

### Offline testing

Skip the network entirely by feeding fixture JSON through env overrides (same shape as the apiserver's `{items:[...]}` responses):

```sh
export SHEET_NODES_JSON='{"items":[{"name":"node-a","cpu_total":4,"cpu_used":2,"status":"Ready"}]}'
export SHEET_DEPLOYMENTS_JSON='{"items":[{"name":"web","replicas":3}]}'
export SHEET_PODS_JSON='{"items":[{"name":"web-1","phase":"Running"}]}'
sheetheus scrape
```

## License

Apache License 2.0.
