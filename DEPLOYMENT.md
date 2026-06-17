# ClusterInstance Deployment Guide
# Documentation for deploying and maintaining this Helm chart

## Prerequisites

### Secrets
Before deploying, create the following secrets in the target namespace:

```bash
# Create pull secret
kubectl create secret docker-registry pullsecret-cluster-adc-cluster \
  --docker-server=quay.io \
  --docker-username=<username> \
  --docker-password=<password> \
  -n open-cluster-management
```

### Network Requirements
- All node IPs must be within the machine network CIDR
- Virtual IPs (apiVIP, ingressVIP) must not be in DHCP range
- Hostnames must be FQDNs matching `*.baseDomain`

### DNS
- All node hostnames must be resolvable
- Ensure DNS servers in `networkDefaults.dns.servers` are accessible from nodes

## Deployment

### Validate Schema
```bash
helm lint . --strict
```

### Install
```bash
helm install cluster-deployment . -f values.yaml \
  -n open-cluster-management
```

### Upgrade
```bash
helm upgrade cluster-deployment . -f values.yaml \
  -n open-cluster-management
```

## Configuration Options

### Per-Node Network Overrides
Override network defaults for specific nodes:

```yaml
masterNodes:
  - name: "master-1"
    hostname: "master-1.adc.example.com"
    # ... other fields ...
    networkConfig:
      ipv4:
        prefixLength: 24  # Override default 22
```

### Boot Mode Override
```yaml
masterNodes:
  - name: "master-1"
    # ... other fields ...
    bootMode: "BIOS"  # Override default UEFI
```

## Important Notes

1. **SSH Public Key**: Currently embedded in values.yaml. Consider moving to a Kubernetes Secret for better security.

2. **Pull Secret**: Template constructs the secret name as `{pullSecretName}-{cluster.name}`. Ensure this secret exists before deployment.

3. **Node Count Requirements**:
   - Master nodes: Minimum 3 (required for High Availability)
   - Worker nodes: Minimum 1

4. **Template References**: These must reference existing templates in the OCM namespace.

5. **Network Configuration**: Verify all IP addresses are correctly assigned and within network CIDRs.

## Troubleshooting

### Cluster Installation Hangs
- Check `holdInstallation: false` is set
- Verify pull secret exists with correct name
- Confirm all node hostnames are resolvable

### Network Connectivity Issues
- Verify gateway address and interface names
- Check MAC addresses match actual hardware
- Ensure DNS servers are accessible

### Invalid YAML Generation
- Validate `values.schema.json` using: `helm lint`
- Check for special characters in string fields
- Ensure all required fields are populated

## Best Practices

1. **Use per-node overrides** instead of modifying networkDefaults directly
2. **Store SSH keys** in Kubernetes Secrets, not in values.yaml
3. **Enable holdInstallation** when troubleshooting, then set to false
4. **Validate** before each deployment: `helm lint`
5. **Monitor node status** after deployment via `oc get nodes`
