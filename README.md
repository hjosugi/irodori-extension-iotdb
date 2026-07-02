# IoTDB Connector

Adds Apache IoTDB connectivity as an installable connector extension.

This connector is listed in the public Irodori extension marketplace.

## Connector

- Extension ID: `irodori.iotdb`
- Engine ID: `iotdb`
- Wire: `timeSeries`
- Default port: `6667`
- Native ABI: `irodori.connector.native.v1`
- Driver linked: `true`

No desktop adapter source exists yet; this package starts from the refactored ABI shim and connector metadata.

Connector metadata lives in `connector.config.json` and `irodori.extension.json`.
The Rust code keeps native ABI exports in `src/lib.rs`, shared buffer/JSON helpers in `src/abi.rs`, and Apache IoTDB Thrift session behavior in `src/driver.rs`.

## Connection Metadata

- Endpoint modes: `hostPort`, `connectionString`
- Transport modes: `direct`, `sshTunnel`, `socks5Proxy`, `httpConnectProxy`, `proxyChain`
- TLS supported: `true`
- Custom driver options: `true`

| Auth method | Label | Secret purposes |
|---|---|---|
| `none` | No authentication | none |
| `connectionString` | Connection string / DSN | none |
| `userPassword` | User/password | `password` |
| `bearerToken` | Bearer token | `token` |
| `kerberos` | Kerberos / GSSAPI | `token` |
| `clientCertificate` | Client certificate / mTLS | `privateKey`, `privateKeyPassphrase` |
| `customDriverOptions` | Custom driver options | `password`, `token`, `privateKey`, `privateKeyPassphrase` |

## Experience Metadata

- Domains: `timeSeries`
- Result views: `timeChart`, `table`, `heatmap`
- Inspired by: `Apache IoTDB time-series hierarchy`, `GROUP BY time`, `ALIGN BY DEVICE`, `FILL`

| Workflow | Result view | Templates |
|---|---|---|
| Device aligned query | timeChart | time-iotdb-group-by-device |
| Gap fill | timeChart | time-iotdb-fill |
| Latest telemetry | table | time-iotdb-latest |

| Template | Label | Language | Result view |
|---|---|---|---|
| `time-iotdb-group-by-device` | GROUP BY time aligned by device | `sql` | `timeChart` |
| `time-iotdb-fill` | Fill missing windows | `sql` | `timeChart` |
| `time-iotdb-latest` | Latest values | `sql` | `table` |

## ABI Calls

The driver handles these JSON requests today:

| Method | Response |
|---|---|
| `health` / `ping` | Connector health, engine id, ABI version, and driver link status. |
| `describe` / `capabilities` | Embedded manifest and connector config. |
| `manifest` | Raw `irodori.extension.json`. |
| `config` | Raw `connector.config.json`. |
| `connect` | Opens an Apache IoTDB RPC session. |
| `query` | Runs SQL through the IoTDB session API and drains result sets. |
| `metadata` | Reads time-series metadata with `SHOW TIMESERIES`. |
| `close` | Closes the IoTDB session and removes the cached native connection. |

## Development


Generated extension repositories share `../target` across sibling repositories so Rust dependencies are compiled once per checkout. DuckDB and MotherDuck are driver-linked by default; set `IRODORI_CONNECTOR_LINK_DUCKDB=0` only when you need metadata-only DuckDB-compatible scaffolds.


```sh
make check
make build
```

Release packages place platform-specific native artifacts under `dist/native`.
