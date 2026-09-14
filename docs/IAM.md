# OCI IAM

Use a dedicated OCI API user for Zabbix monitoring rather than a personal administrator identity.

## Example group

```text
ZabbixMonitoring
```

## Example policy

```text
Allow group ZabbixMonitoring to read file-family in tenancy
Allow group ZabbixMonitoring to read metrics in tenancy
Allow group ZabbixMonitoring to read virtual-network-family in tenancy
```

The `virtual-network-family` permission is needed only for automatic private-IP enrichment of Mount Targets. If it is not granted, use:

```text
{$OCI.FSS.RESOLVE.PRIVATE.IP}=0
```

## Compartment-scoped alternative

If Zabbix should monitor only selected compartments, scope the policy to those compartments rather than granting tenancy-wide visibility. Automatic discovery will then return only the resources visible to the API user.

## API signing key

Generate an OCI API signing key for the dedicated monitoring user and configure these Zabbix macros:

```text
{$OCI.API.TENANCY}
{$OCI.API.USER}
{$OCI.API.FINGERPRINT}
{$OCI.API.PRIVATE.KEY}
```

Protect the private key as a secret. Avoid placing production private keys in source control, README files, issue descriptions or screenshots.
