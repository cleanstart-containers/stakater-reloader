# Stakater Reloader Test Deployment

This deployment demonstrates the functionality of the Stakater Reloader controller, which automatically reloads pods when ConfigMaps or Secrets change.

## Overview

The deployment includes:
1. **Stakater Reloader Controller** - Watches for changes in ConfigMaps and Secrets
2. **Test Web Application** - A simple Python web server that displays configuration values from ConfigMaps and Secrets
3. **ConfigMap** - Contains application configuration (APP_TITLE, APP_COLOR, SITE_NAME)
4. **Secret** - Contains sensitive configuration (APP_NAME, ENV, VERSION, MESSAGE)

## Deployment

Deploy all resources:

```bash
kubectl apply -f deployment.yaml
```

Wait for all pods to be ready:

```bash
kubectl get pods -n reloader-test -w
```

You should see:
- `reloader-*` - The reloader controller pod
- `web-app-*` - The test web application pod

## Accessing the Web Application

### Option 1: Port Forward

```bash
kubectl port-forward -n reloader-test svc/web-app 8080:80
```

Then open your browser and navigate to: `http://localhost:8080`

### Option 2: Using kubectl proxy

```bash
kubectl proxy
```

Then access: `http://localhost:8001/api/v1/namespaces/reloader-test/services/web-app:80/proxy/`

### Option 3: Ingress (if configured)

If you have an ingress controller, uncomment the Ingress section in the deployment.yaml file and configure your ingress controller accordingly.

## Testing the Reloader Functionality

### Test 1: Update ConfigMap

1. Edit the ConfigMap:
```bash
kubectl edit configmap web-app-config -n reloader-test
```

2. Change one of the values, for example:
```yaml
data:
  APP_TITLE: "Stakater Reloader Demo - UPDATED"
  APP_COLOR: "#ff6b6b"
  SITE_NAME: "Updated Test Site"
```

3. Save and exit. The reloader will detect the change and restart the web-app pod.

4. Watch the pod restart:
```bash
kubectl get pods -n reloader-test -w
```
After the pod restarts, refresh your browser to see the updated configuration.

5. Port Forward and check changes in website
```bash
kubectl port-forward -n reloader-test svc/web-app 8080:80
```
Then open your browser and navigate to: `http://localhost:8080`


### Test 2: Update Secret

1. Update the Secret using kubectl:
```bash
kubectl create secret generic web-app-secret \
  --from-literal=APP_NAME="Updated App Name" \
  --from-literal=ENV="staging" \
  --from-literal=VERSION="2.0.0" \
  --from-literal=MESSAGE="Secret was updated! Reloader triggered a restart." \
  -n reloader-test \
  --dry-run=client -o yaml | kubectl apply -f -
```

Or edit it directly (values are base64 encoded):
```bash
kubectl edit secret web-app-secret -n reloader-test
```
- use this configuration:

APP_NAME=Updated App Name

ENV=staging

VERSION=2.0.0

MESSAGE=Secret was updated! Reloader triggered a restart.

2. Watch the pod restart:
```bash
kubectl get pods -n reloader-test -w
```

3. Port Forward and check changes in website
```bash
kubectl port-forward -n reloader-test svc/web-app 8080:80
```
Then open your browser and navigate to: `http://localhost:8080`

### Test 3: Verify Reloader Logs

Check the reloader logs to see it detecting changes:

```bash
kubectl logs -n reloader-test -l app=reloader -f
```

You should see logs indicating that it detected changes in ConfigMaps or Secrets and triggered a reload.

## How It Works

1. The reloader controller watches all ConfigMaps and Secrets in the cluster (or specific namespaces if configured).

2. When a ConfigMap or Secret that is referenced by a Deployment (via environment variables or volume mounts) is updated, the reloader detects the change.

3. The reloader updates an annotation on the Deployment, which triggers a rolling restart of the pods.

4. The annotation `reloader.stakater.com/auto: "true"` on the Deployment enables automatic reloading for all ConfigMaps and Secrets referenced by that Deployment.

## Configuration Options

### Reloader Annotations

- `reloader.stakater.com/auto: "true"` - Automatically reload when any ConfigMap or Secret referenced by the Deployment changes
- `reloader.stakater.com/search: "true"` - Search for ConfigMaps and Secrets with matching labels
- `reloader.stakater.com/match: "true"` - Match ConfigMaps and Secrets with specific annotations

### Reloader Configuration

The reloader in this deployment uses default configuration:
- Watches all namespaces by default
- Uses annotation-based reload strategy (default)
- Automatic detection of ConfigMap and Secret changes

## Cleanup

To remove all resources:

```bash
kubectl delete -f deployment.yaml
```

Or delete the namespace (which will delete all resources):

```bash
kubectl delete namespace reloader-test
```

