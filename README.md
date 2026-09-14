# Zabbix OCI File Storage by HTTP

![Zabbix 7.0](https://img.shields.io/badge/Zabbix-7.0-D40000?logo=zabbix&logoColor=white)
![Oracle Cloud](https://img.shields.io/badge/Oracle%20Cloud-OCI-F80000?logo=oracle&logoColor=white)
![File Storage](https://img.shields.io/badge/OCI-File%20Storage-312D2A)
![HTTP API](https://img.shields.io/badge/Monitoring-REST%20API-0078D4)
![Auto Discovery](https://img.shields.io/badge/Resources-Auto%20Discovery-success)

Zabbix 7.0 template for monitoring **Oracle Cloud Infrastructure File Storage (FSS)** directly through OCI REST APIs.

The template automatically discovers File Systems and Mount Targets in the configured OCI region. No File System OCID, Mount Target OCID or compartment list is required.

## What it monitors

- File System and Mount Target lifecycle state
- capacity and metered usage
- hard and soft quotas
- quota usage and violations
- read/write throughput
- read/write latency
- metadata operations and latency
- Mount Target connections and health
- exports and resource metadata
- private IP enrichment when permitted by IAM
- OCI API collector health

The template monitors the **provider side** of OCI File Storage. For client-side NFS monitoring, use [`zabbix-linux-nfs-enterprise`](https://github.com/kevinmagni/zabbix-linux-nfs-enterprise).

## Requirements

- Zabbix 7.0
- OCI API signing key
- dedicated OCI monitoring user recommended
- HTTPS access from the Zabbix server/proxy to OCI API endpoints
- one Zabbix host per OCI region

No OCI CLI or OCI SDK is required.

## Installation

1. Import `zabbix_template_oci_file_storage_by_http_7.0.yaml` into Zabbix.
2. Create a host representing the OCI region, for example:

```text
OCI File Storage - eu-frankfurt-1
```

3. Link the template `Oracle OCI File Storage by HTTP`.
4. Configure the OCI authentication macros.
5. Set `{$OCI.FSS.REGION}` to the region monitored by that host.

## Required authentication macros

| Macro | Description |
|---|---|
| `{$OCI.API.TENANCY}` | OCI Tenancy OCID |
| `{$OCI.API.USER}` | OCI API user OCID |
| `{$OCI.API.FINGERPRINT}` | API signing key fingerprint |
| `{$OCI.API.PRIVATE.KEY}` | PEM private key used to sign API requests |
| `{$OCI.FSS.REGION}` | OCI region to monitor |

Store `{$OCI.API.PRIVATE.KEY}` as a Zabbix secret macro or in an external secret store where possible.

## OCI IAM

A dedicated monitoring group can use the following read-only policy:

```text
Allow group ZabbixMonitoring to read file-family in tenancy
Allow group ZabbixMonitoring to read metrics in tenancy
Allow group ZabbixMonitoring to read virtual-network-family in tenancy
```

`virtual-network-family` is required only for private IP enrichment.

If that permission is not available:

```text
{$OCI.FSS.RESOLVE.PRIVATE.IP}=0
```

Policies can be scoped to specific compartments instead of the whole tenancy. Discovery only returns resources visible to the API user.

See [`docs/IAM.md`](./docs/IAM.md) for more details.

## Main thresholds

| Macro | Default | Description |
|---|---:|---|
| `{$OCI.FSS.QUOTA.WARN}` | `85` | Quota usage warning threshold (%) |
| `{$OCI.FSS.QUOTA.CRIT}` | `95` | Quota usage critical threshold (%) |
| `{$OCI.FSS.LATENCY.WARN}` | `0.05` | File Storage latency warning threshold (seconds) |
| `{$OCI.FSS.LATENCY.CRIT}` | `0.1` | File Storage latency critical threshold (seconds) |
| `{$OCI.FSS.METADATA.LATENCY.WARN}` | `0.05` | Metadata latency warning threshold (seconds) |
| `{$OCI.FSS.MT.HEALTH.WARN}` | `99` | Mount Target health warning threshold (%) |
| `{$OCI.FSS.MT.HEALTH.CRIT}` | `95` | Mount Target health critical threshold (%) |

## Resource filters

All File Systems and Mount Targets are discovered by default.

Use these macros only when filtering is required:

```text
{$OCI.FSS.FS.NAME.MATCHES}=.*
{$OCI.FSS.FS.NAME.NOT_MATCHES}=CHANGE_IF_NEEDED
{$OCI.FSS.MT.NAME.MATCHES}=.*
{$OCI.FSS.MT.NAME.NOT_MATCHES}=CHANGE_IF_NEEDED
```

## Optional settings

Commercial OCI regions use:

```text
{$OCI.FSS.REALM.DOMAIN}=oraclecloud.com
```

TLS verification is enabled by default:

```text
{$OCI.FSS.TLS.VERIFY}=full
```

An HTTP proxy can be configured with:

```text
{$OCI.HTTP.PROXY}
```

Individual OCI API hosts can also be overridden when required by the target realm or network architecture.

## Discovery model

The template automatically discovers:

**File Systems**
- display name and OCID
- compartment and Availability Domain
- lifecycle state
- exports
- quota configuration
- capacity and performance metrics

**Mount Targets**
- display name and OCID
- lifecycle state
- private IP information
- Export Set
- NSGs
- connection and health metrics

## API health

The template monitors its own OCI API collectors. Authentication failures, missing IAM permissions, endpoint errors and request failures are reported explicitly instead of appearing as zero metrics.

## Troubleshooting

See [`docs/TROUBLESHOOTING.md`](./docs/TROUBLESHOOTING.md).

Useful checks:

- verify the API key fingerprint and private key
- verify the configured OCI region
- verify IAM policies assigned to the monitoring user/group
- verify HTTPS connectivity from the Zabbix server/proxy to OCI API endpoints

## Version

Current template: **1.0**  
Target: **Zabbix 7.0**
