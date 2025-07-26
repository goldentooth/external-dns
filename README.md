# ExternalDNS

GitOps configuration for ExternalDNS in the Goldentooth Kubernetes cluster, enabling automatic DNS record management for Services and Ingresses.

## Overview

ExternalDNS automatically creates DNS records in AWS Route53 for Kubernetes Services and Ingresses, eliminating the need for manual DNS management. This is crucial for the Goldentooth cluster's public services and load balancer integration.

## Components

### Helm Chart Configuration
- **Chart**: Custom Helm chart for ExternalDNS deployment
- **Version**: v0.14.2 (configurable via `values.yaml`)
- **Namespace**: `external-dns`
- **Domain**: `goldentooth.net` (filtered)

### Kubernetes Resources

#### `ServiceAccount.yaml`
Creates the `external-dns` service account with appropriate annotations for AWS IAM role binding.

#### `ClusterRole.yaml` & `ClusterRoleBinding.yaml`
Provides necessary permissions for ExternalDNS to:
- Read Services and Ingresses cluster-wide
- Manage DNS records based on annotations
- Access AWS Route53 via IAM roles

#### `Deployment.yaml`
Deploys ExternalDNS controller with:
- AWS Route53 provider configuration
- Domain filtering for `goldentooth.net`
- Resource limits and health checks
- Security context and pod security policies

## AWS Integration

ExternalDNS integrates with AWS Route53 through:
- **IAM Roles**: AWS IAM role for service accounts (IRSA)
- **Route53 Zones**: Manages records in the `goldentooth.net` hosted zone
- **Credential Management**: Uses Kubernetes service account annotations for AWS authentication

## Deployment

This chart is deployed via Argo CD as part of the Goldentooth GitOps workflow:

```yaml
# Deployed by Argo CD
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: external-dns
spec:
  source:
    repoURL: https://github.com/goldentooth/external-dns
    path: .
```

## Configuration

### Domain Filtering
ExternalDNS is configured to only manage DNS records for:
- `*.goldentooth.net`
- Root domain: `goldentooth.net`

### Service Annotations
To expose a service via ExternalDNS, add annotations:
```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    external-dns.alpha.kubernetes.io/hostname: "service.goldentooth.net"
spec:
  type: LoadBalancer
```

### Ingress Integration
For Ingress resources:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    external-dns.alpha.kubernetes.io/hostname: "app.goldentooth.net"
spec:
  rules:
  - host: app.goldentooth.net
```

## Monitoring

ExternalDNS provides metrics for:
- DNS record creation/deletion operations
- AWS API call rates and errors
- Sync loop performance
- Route53 zone health

These metrics are scraped by Prometheus and displayed in Grafana dashboards.

## Security

- **IAM Least Privilege**: Limited to specific Route53 zone access
- **Network Policies**: Restricted egress to AWS Route53 endpoints
- **Pod Security**: Non-root execution with read-only filesystem
- **RBAC**: Minimal cluster permissions for DNS operations only
