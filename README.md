# 🔄 Stakater Reloader - CleanStart Container

This project provides a CleanStart container image for **Stakater Reloader**, a Kubernetes controller that automatically reloads pods when ConfigMaps or Secrets change. Stakater Reloader watches for changes in ConfigMaps and Secrets that are referenced by Deployments, StatefulSets, or DaemonSets, and triggers rolling updates to restart pods with the updated configuration, eliminating the need for manual pod restarts. The CleanStart Stakater-Reloader image provides a production-ready, security-hardened container optimized for enterprise environments. Built on a minimal base OS with comprehensive security hardening, this image delivers reliable application execution with advanced security features.

**📌 CleanStart Foundation:** Security-hardened, minimal base OS designed for enterprise containerized environments.

**Image Path:** `ghcr.io/cleanstart-containers/stakater-reloader`

**Registry:** CleanStart Registry

---

## Overview

Stakater Reloader is a Kubernetes controller that automatically reloads pods when ConfigMaps or Secrets change. It operates as a Kubernetes-native controller, continuously monitoring ConfigMaps and Secrets referenced by workloads and triggering rolling updates when changes are detected. This CleanStart Stakater-Reloader container is part of the CleanStart application suite, featuring enterprise-grade security hardening, automated vulnerability management, and compliance with industry standards.

---

## About CleanStart

CleanStart is a comprehensive container registry providing security-hardened, enterprise-ready container images. Our images are designed with security-first principles, featuring minimal attack surfaces, regular security updates, and compliance with industry standards.

### About CleanStart Images

CleanStart images are built on secure, minimal base operating systems and optimized for production environments. Each image undergoes rigorous security testing, vulnerability scanning, and compliance validation to ensure enterprise-grade security and reliability.

---

## Project Structure

The Stakater Reloader container project is organized into the following structure:

- **Root Directory**: Contains the main project README and container build configuration
- **`kubernetes/`**: Complete Kubernetes deployment manifests and documentation for testing the Stakater Reloader container in a Kubernetes environment, including a test web application that demonstrates the reloader functionality

---

## Container Image

**Image:** `ghcr.io/cleanstart-containers/stakater-reloader:latest-dev`

The container image includes:

- Stakater Reloader binary located at `/usr/bin/Reloader`
- Non-root user execution (`clnstrt` user)
- Pre-configured SSL certificates at `/etc/ssl/certs/ca-certificates.crt`
- Linux/amd64 architecture support
- Security-hardened configuration with minimal privileges

---

## Key Features

- **Security-First Design**: Built with minimal attack surfaces and security hardening
- **Enterprise Compliance**: Meets industry standards including FIPS, STIG, and CIS benchmarks
- **Regular Updates**: Automated security patches and vulnerability management
- **Multi-Architecture Support**: Available for AMD64 and ARM64 architectures
- **Production Ready**: Optimized for enterprise deployment and scaling
- **Comprehensive Documentation**: Detailed guides and best practices for each image
- **Automatic Pod Reloading**: Automatically triggers rolling updates when ConfigMaps or Secrets change
- **Annotation-Based**: Uses Kubernetes annotations to identify which deployments should be reloaded
- **Multi-Namespace Support**: Watches all namespaces by default (configurable)
- **ConfigMap & Secret Support**: Monitors both ConfigMaps and Secrets for changes
- **Multiple Workload Types**: Supports Deployments, StatefulSets, and DaemonSets
- **Security**: Runs as non-root user with dropped capabilities and no privilege escalation
- **Resource Management**: Configured with CPU and memory requests/limits
- **Leader Election**: Enabled to prevent multiple controller instances from running simultaneously

---

## Use Cases

Typical scenarios where this container excels:

- **Image Validation**: Testing the CleanStart Stakater Reloader container image
- **Development Testing**: Validating Reloader functionality in Kubernetes environments
- **Configuration Management**: Demonstrating automatic pod reloading when configuration changes
- **CI/CD Integration**: Automated testing of the container image and reloader functionality
- **Learning & Training**: Understanding how automatic configuration reloading works in Kubernetes
- Zero downtime configuration updates
- Automated application restarts on config changes
- Production configuration management

---

## How It Works

1. **Configuration Monitoring**: The Reloader controller watches all ConfigMaps and Secrets in the cluster (or specific namespaces if configured)

2. **Change Detection**: When a ConfigMap or Secret that is referenced by a Deployment (via environment variables or volume mounts) is updated, the reloader detects the change

3. **Annotation Update**: The reloader updates an annotation on the Deployment, which triggers a rolling restart of the pods

4. **Pod Restart**: Kubernetes performs a rolling update, creating new pods with the updated configuration while gracefully terminating old pods

5. **Annotation Configuration**: Deployments with the annotation `reloader.stakater.com/auto: "true"` automatically reload when any referenced ConfigMap or Secret changes

---

## Prerequisites

Before deploying, ensure you have:

1. **Kubernetes Cluster**: Any Kubernetes cluster (Kind, minikube, k3s, GKE, EKS, AKS, etc.)
2. **kubectl**: Installed and configured to connect to your cluster
3. **Image Access**: Ability to pull `ghcr.io/cleanstart-containers/stakater-reloader:latest-dev` and `python:3.11-alpine` images

---

## Quick Start

### Pull Commands
```bash
docker pull ghcr.io/cleanstart-containers/stakater-reloader:latest
docker pull ghcr.io/cleanstart-containers/stakater-reloader:latest-dev
```

### Run Commands

Basic test:
```bash
docker run -it --name stakater-reloader-test ghcr.io/cleanstart-containers/stakater-reloader:latest-dev
```

Production deployment:
```bash
docker run -d --name stakater-reloader-prod \
  --read-only \
  --security-opt=no-new-privileges \
  --user 1000:1000 \
  ghcr.io/cleanstart-containers/stakater-reloader:latest
```

---

## Kubernetes Deployment

The `kubernetes/` directory contains a complete deployment setup for testing the Stakater Reloader container functionality in Kubernetes clusters, including a test web application that demonstrates automatic pod reloading when ConfigMaps or Secrets change.

### Deployment Components

The Kubernetes deployment includes:

**Stakater Reloader Controller** (in `reloader-test` namespace):
- Namespace: `reloader-test`
- ServiceAccount: `reloader` with appropriate RBAC permissions
- ClusterRole & ClusterRoleBinding: Permissions for watching ConfigMaps, Secrets, and updating Deployments, StatefulSets, and DaemonSets
- Deployment: Single-replica deployment running the Reloader controller
- Leader Election: Enabled for high availability

**Test Web Application** (in `reloader-test` namespace):
- Deployment: Python-based web server that displays configuration values from ConfigMaps and Secrets
- Service: ClusterIP service exposing port 80 (targeting port 8000)
- ConfigMap: Contains application configuration (APP_TITLE, APP_COLOR, SITE_NAME)
- Secret: Contains sensitive configuration (APP_NAME, ENV, VERSION, MESSAGE)
- Annotations: Configured with `reloader.stakater.com/auto: "true"` to enable automatic reloading

### Current Configuration

The deployment is configured for **testing and validation purposes**. The reloader controller runs with:

- Default configuration (watches all namespaces)
- Annotation-based reload strategy
- Automatic detection of ConfigMaps and Secrets referenced by deployments
- Test web application that demonstrates the reloader functionality

For detailed deployment instructions, testing procedures, and configuration options, refer to the `kubernetes/README.md` file in the `kubernetes/` directory.

---

## Test Application

The deployment includes a test web application that:

- Displays configuration values from ConfigMaps and Secrets in a web browser
- Shows pod information (pod name, timestamp) to verify restarts
- Demonstrates the reloader functionality by automatically restarting when configuration changes
- Provides a visual interface for testing configuration updates

---

## Benefits

- **Zero Downtime Updates**: Rolling updates ensure applications remain available during configuration changes
- **Automated Management**: No manual intervention required to restart pods after configuration changes
- **Configuration Consistency**: Ensures all pods use the latest configuration without manual restarts
- **DevOps Friendly**: Simplifies configuration management and reduces operational overhead
- **Production Ready**: Suitable for production environments with proper RBAC and resource limits

---

## Architecture Support

CleanStart images support multiple architectures to ensure compatibility across different deployment environments:

- **AMD64**: Intel and AMD x86-64 processors
- **ARM64**: ARM-based processors including Apple Silicon and ARM servers

### Architecture-based Pull Commands
```bash
docker pull --platform linux/amd64 ghcr.io/cleanstart-containers/stakater-reloader:latest
docker pull --platform linux/arm64 ghcr.io/cleanstart-containers/stakater-reloader:latest
```

---

## Notes

The current Kubernetes deployment runs Stakater Reloader with default configuration suitable for testing and validation. For production use, you would typically:

- Configure namespace-specific watching if needed
- Set up monitoring and alerting for the reloader controller
- Configure appropriate resource limits based on cluster size
- Use specific annotations for fine-grained control over which ConfigMaps/Secrets trigger reloads
- Implement proper logging and observability for configuration changes
- Consider using reloader with GitOps tools for configuration management

---

## Resources

- **Official Documentation:** https://github.com/stakater/Reloader
- **Stakater GitHub Repository:** https://github.com/stakater/Reloader
- **Provenance / SBOM / Signature:** https://images.cleanstart.com/images/stakater-reloader
- **Docker Hub:** https://hub.docker.com/r/cleanstart/stakater-reloader
- **CleanStart All Images:** https://images.cleanstart.com/images/stakater-reloader/details
- **CleanStart Community Images:** https://hub.docker.com/u/cleanstart
---

## Vulnerability Disclaimer

CleanStart offers Docker images that include third-party open-source libraries and packages maintained by independent contributors. While CleanStart maintains these images and applies industry-standard security practices, it cannot guarantee the security or integrity of upstream components beyond its control.

Users acknowledge and agree that open-source software may contain undiscovered vulnerabilities or introduce new risks through updates. CleanStart shall not be liable for security issues originating from third-party libraries, including but not limited to zero-day exploits, supply chain attacks, or contributor-introduced risks.

**Security remains a shared responsibility:** CleanStart provides updated images and guidance where possible, while users are responsible for evaluating deployments and implementing appropriate controls.
