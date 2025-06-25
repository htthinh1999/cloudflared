# Cloudflared Helm Chart

A Kubernetes Helm chart for deploying [Cloudflare Tunnel (cloudflared)](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/) with automated CI/CD pipeline and semantic versioning.

## Overview

This repository provides a production-ready Helm chart for deploying cloudflared in Kubernetes clusters. Cloudflare Tunnel creates secure, outbound-only connections from your infrastructure to Cloudflare's network, eliminating the need to open inbound firewall ports.

## Features

- 🚀 **Production-ready Helm chart** with configurable values
- 🔄 **Automated CI/CD pipeline** with GitHub Actions
- 📦 **Semantic versioning** with automatic tag increment
- 🔧 **Configurable ingress rules** for multiple services
- 📊 **Built-in metrics and health checks** on port 2000
- 🔄 **Horizontal Pod Autoscaling** support
- 🔐 **Secure credential management** via Kubernetes secrets

## Quick Start

### Prerequisites

- Kubernetes cluster (1.19+)
- Helm 3.x
- Cloudflare account with Tunnel configured
- Tunnel credentials file

### Installation

1. **Create tunnel credentials secret:**
   ```bash
   kubectl create secret generic tunnel-credentials \
     --from-file=credentials.json=/path/to/your/tunnel/credentials.json
   ```

2. **Add the Helm repository:**
   ```bash
   helm repo add cloudflared ghcr.io/htthinh1999
   helm repo update
   ```

3. **Install the chart:**
   ```bash
   helm install my-cloudflared cloudflared/cloudflared \
     --set cloudflaredConfig.tunnel=your-tunnel-name
   ```

## Configuration

### Basic Configuration

The chart can be configured via `values.yaml`. Key configuration options:

```yaml
# Replica count
replicaCount: 1

# Cloudflared image
image:
  repository: cloudflare/cloudflared
  tag: 2025.5.0

# Tunnel configuration
cloudflaredConfig:
  tunnel: your-tunnel-name
  credentials-file: /etc/cloudflared/creds/credentials.json
  metrics: 0.0.0.0:2000
  no-autoupdate: true
  
  # Ingress rules for routing traffic
  ingress:
    - hostname: example.com
      service: http://my-service.default.svc.cluster.local:80
    - service: http_status:404

# Secret containing tunnel credentials
cloudflaredTunnelSecretName: tunnel-credentials
```

### Advanced Configuration

#### Horizontal Pod Autoscaling
```yaml
autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80
```

#### WARP Routing
```yaml
cloudflaredConfig:
  warp_routing:
    enabled: true
```

## Project Structure

```
cloudflared/
├── .github/workflows/ci.yaml   # CI/CD pipeline configuration
├── charts/cloudflared/         # Helm chart files
│   ├── Chart.yaml              # Chart metadata
│   ├── values.yaml             # Default configuration values
│   └── templates/              # Kubernetes manifests
│       ├── deployment.yaml     # Main cloudflared deployment
│       ├── configmap.yaml      # Configuration file
│       ├── hpa.yaml            # Horizontal Pod Autoscaler
│       └── _helpers.tpl        # Template helpers
```

## CI/CD Pipeline

The project includes a GitHub CI/CD pipeline that:

1. **Validates** the chart on every commit
2. **Auto-increments** semantic version tags on main branch
3. **Builds and pushes** Helm chart to registry
4. **Updates** chart version automatically

### Pipeline Stages

- `validate`: Validates Helm chart syntax
- `build-push-docker-image`: Creates and pushes git tags
- `push-helm-chart`: Packages and publishes Helm chart

## Development

### Local Development

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd cloudflared
   ```

2. **Modify chart values:**
   Edit `charts/cloudflared/values.yaml` with your configuration

3. **Test the chart:**
   ```bash
   helm template test charts/cloudflared --values charts/cloudflared/values.yaml
   ```

4. **Install locally:**
   ```bash
   helm install test-cloudflared charts/cloudflared
   ```

### Scripts

- `./auto-increment-git-tag.sh` - Automatically increment patch version
- `./current-git-tag.sh` - Get the current git tag version
- `./update-helm-chart.sh <version>` - Update chart version in Chart.yaml and values.yaml

## Monitoring

The deployment includes built-in monitoring capabilities:

- **Metrics endpoint**: Available at `:2000/metrics`
- **Health check**: Available at `:2000/ready`
- **Liveness probe**: Configured to check tunnel connectivity

## Troubleshooting

### Common Issues

1. **Tunnel not connecting:**
   - Verify credentials secret is correctly mounted
   - Check tunnel name matches your Cloudflare configuration
   - Review logs: `kubectl logs -l app=cloudflared`

2. **Service not accessible:**
   - Verify ingress rules in `values.yaml`
   - Check DNS configuration in Cloudflare dashboard
   - Ensure target services are running and accessible

3. **Pod startup issues:**
   - Check resource limits and requests
   - Verify image pull permissions
   - Review security context settings

### Viewing Logs

```bash
# View all cloudflared pods
kubectl get pods -l app=cloudflared

# View logs for specific pod
kubectl logs <pod-name>

# Follow logs in real-time
kubectl logs -f deployment/cloudflared
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a merge request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

For issues and questions:
- Create an issue in this repository
- Check [Cloudflare Tunnel documentation](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)
- Review [Kubernetes documentation](https://kubernetes.io/docs/)
