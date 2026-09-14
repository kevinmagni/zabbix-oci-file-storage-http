# Troubleshooting

## No resources are discovered

Verify:

1. the configured region;
2. OCI API credentials;
3. Resource Search permissions;
4. File Storage read permissions;
5. discovery filters.

Default region in this repository version:

```text
{$OCI.FSS.REGION}=eu-milan-1
```

Default discovery filters include every resource:

```text
{$OCI.FSS.FS.NAME.MATCHES}=.*
{$OCI.FSS.MT.NAME.MATCHES}=.*
```

## HTTP 401 / signature failures

Verify all four API authentication macros:

```text
{$OCI.API.TENANCY}
{$OCI.API.USER}
{$OCI.API.FINGERPRINT}
{$OCI.API.PRIVATE.KEY}
```

Make sure the private key corresponds to the public key registered for the OCI user and that the fingerprint is correct.

Also verify time synchronization on the Zabbix server/proxy because signed API requests are time-sensitive.

## HTTP 403

A 403 normally indicates that the request was authenticated but the API user lacks permission for the requested resource.

Review the OCI IAM policies described in [`IAM.md`](./IAM.md).

## File Systems appear but private IPs do not

Mount Target private-IP enrichment requires Core/VNIC read permissions.

Either grant the required read access or disable enrichment:

```text
{$OCI.FSS.RESOLVE.PRIVATE.IP}=0
```

## Metrics are empty for a discovered resource

Confirm that the resource is in the same region represented by the Zabbix host and that OCI Monitoring is publishing the expected `oci_filestorage` metrics for that resource.

Also check the API collector items for explicit OCI errors rather than assuming an empty result means zero activity.

## Custom realm or endpoint

Commercial OC1 regions normally use:

```text
{$OCI.FSS.REALM.DOMAIN}=oraclecloud.com
```

For another realm or custom endpoint, override the relevant host macros:

```text
{$OCI.FSS.SEARCH.HOST}
{$OCI.FSS.FILE.HOST}
{$OCI.FSS.TELEMETRY.HOST}
{$OCI.FSS.CORE.HOST}
```
