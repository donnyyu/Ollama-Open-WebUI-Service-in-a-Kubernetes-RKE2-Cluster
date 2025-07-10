# Ollama + Open WebUI on RKE2

This project provides Kubernetes manifests to deploy Ollama (Large Language Model inference server) along with Open WebUI (web interface) on an RKE2 cluster with GPU support.

## Features

- **Ollama Server**: Self-hosted LLM inference server
- **Open WebUI**: Modern web interface for interacting with language models
- **GPU Support**: Configured for NVIDIA GeForce RTX 3060
- **Persistent Storage**: Data persistence for models and configurations
- **Ingress Access**: External access via NGINX ingress controller

## Prerequisites

### Hardware Requirements
- RKE2 cluster with at least one node containing an NVIDIA GeForce RTX 3060 GPU
- Minimum 20GB storage for Ollama models
- Additional 1GB storage for Open WebUI data

### Software Requirements
- RKE2 cluster with kubectl access
- NVIDIA Container Runtime configured on GPU nodes
- NGINX Ingress Controller installed
- Local storage provisioner or manual PV creation

### Node Labels
Ensure your GPU node has the following labels:
```bash
kubectl label nodes <node-name> nvidia.com/gpu.present=true
kubectl label nodes <node-name> nvidia.com/gpu.product=NVIDIA-GeForce-RTX-3060
```

## Installation

### 1. Create Namespace
```bash
kubectl create namespace ollama
```

### 2. Prepare Storage Directories
Create the required directories on your GPU node:
```bash
sudo mkdir -p /opt/ollama-data
sudo mkdir -p /opt/open-webui-data
sudo chmod 755 /opt/ollama-data /opt/open-webui-data
```

### 3. Deploy the Stack
```bash
kubectl apply -f ollama-openwebui.yaml
```

### 4. Verify Deployment
```bash
# Check if pods are running
kubectl get pods -n ollama

# Check services
kubectl get svc -n ollama

# Check ingress
kubectl get ingress -n ollama
```

## Configuration

### Storage Configuration
- **Ollama Models**: 20GB persistent volume mounted at `/root/.ollama/models`
- **Open WebUI Data**: 1GB persistent volume mounted at `/app/backend/data`
- **Storage Class**: `local-storage` (modify as needed for your cluster)

### Network Access

#### Internal Access
- **Ollama Service**: `http://ollama-service:11434` (ClusterIP)
- **Open WebUI Service**: `http://open-webui-service:8080` (NodePort: 30088)

#### External Access
Configure your DNS or `/etc/hosts` file:
```
<your-cluster-ip> ollama.local
<your-cluster-ip> open-webui.local
```

Access the applications:
- **Ollama API**: `http://ollama.local`
- **Open WebUI**: `http://open-webui.local`

## Usage

### Accessing Open WebUI
1. Navigate to `http://open-webui.local` in your browser
2. Create an account (first user becomes admin)
3. Start chatting with AI models

### Managing Models
You can pull models directly through the Open WebUI interface or via kubectl exec:

```bash
# Execute into the ollama container
kubectl exec -it deployment/ollama-deployment -n ollama -- /bin/bash

# Pull a model (e.g., llama2)
ollama pull llama2

# List available models
ollama list
```

### Common Models to Try
- `llama2`: General-purpose conversational model
- `codellama`: Code-focused model
- `mistral`: Efficient general-purpose model
- `phi`: Compact model for resource-constrained environments

## Troubleshooting

### Pod Scheduling Issues
If pods are not scheduling, check:
```bash
# Verify node labels
kubectl get nodes --show-labels | grep nvidia

# Check node affinity
kubectl describe pod <pod-name> -n ollama
```

### Storage Issues
```bash
# Check PV/PVC status
kubectl get pv,pvc -n ollama

# Verify storage directories exist on the node
ls -la /opt/ollama-data /opt/open-webui-data
```

### GPU Access Issues
```bash
# Check if NVIDIA runtime is available
kubectl describe node <gpu-node> | grep -i nvidia

# Verify runtimeClassName is supported
kubectl get runtimeclasses
```

### Service Connection Issues
```bash
# Test internal connectivity
kubectl exec -it deployment/open-webui-deployment -n ollama -- curl -I http://ollama-service:11434

# Check service endpoints
kubectl get endpoints -n ollama
```

## Customization

### Different GPU Models
To use a different GPU, update the node selectors and affinity rules:
```yaml
nodeSelector:
  nvidia.com/gpu.present: "true"
  nvidia.com/gpu.product: "YOUR-GPU-MODEL"
```

### Storage Configuration
For different storage solutions, modify the PV specifications:
```yaml
spec:
  storageClassName: your-storage-class
  # Remove local volume configuration if using dynamic provisioning
```

### Resource Limits
Add resource limits to prevent resource contention:
```yaml
resources:
  requests:
    memory: "4Gi"
    cpu: "2"
    nvidia.com/gpu: 1
  limits:
    memory: "8Gi"
    cpu: "4"
    nvidia.com/gpu: 1
```

## Security Considerations

- The current configuration runs containers as root for GPU access
- Consider implementing network policies for production deployments
- Use proper authentication mechanisms for Open WebUI in production
- Secure ingress with TLS certificates

## Monitoring

### Health Checks
```bash
# Check Ollama health
curl http://ollama.local/api/tags

# Check Open WebUI health
curl http://open-webui.local/health
```

### Logs
```bash
# View Ollama logs
kubectl logs deployment/ollama-deployment -n ollama

# View Open WebUI logs
kubectl logs deployment/open-webui-deployment -n ollama
```

## Uninstallation

To remove the deployment:
```bash
kubectl delete -f ollama-openwebui.yaml
kubectl delete namespace ollama

# Clean up storage directories (optional)
sudo rm -rf /opt/ollama-data /opt/open-webui-data
```

## Acknowledgments

- [Ollama](https://ollama.ai/) - Self-hosted LLM inference server
- [Open WebUI](https://github.com/open-webui/open-webui) - Web interface for LLMs
- [RKE2](https://docs.rke2.io/) - Kubernetes distribution