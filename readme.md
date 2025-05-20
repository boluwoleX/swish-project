# Dockerfile Deploy workflow kube-action.yml
This repo includes an automated CI/CD pipeline defined in `.github/workflows/kube-action.yml` that:

1. **Builds a Docker image** with Python2, Python3, and R, then pushes it to Docker Hub (`boluwole/deploy-env:latest`).
2. **Logs build time** to an artifact and suggests improvements (layer caching, pinning versions).
3. **Scans for CVEs** using Trivy and uploads a vulnerability report.
4. **Deploys the image** to a local Kubernetes cluster via `kind`, with a long-running `tail -f /dev/null` command.
5. **Exposes the deployment** via a Kubernetes `NodePort` service on port 8080.
6. **Automates all steps** using GitHub Actions (triggered on push or manual dispatch).
7. **Adds monitoring hooks** (Prometheus `ServiceMonitor` + alert rules) for pod health and memory usage.

All logic lives in `kube-action.yml`. This setup covers secure build, deploy, and observability in a minimal dev cluster.


# Kube Deploy Workflow test.yml

This GitHub Action workflow automates the deployment of a Docker-based Kubernetes environment, provisioning Prometheus monitoring, and exposing essential services for both application and SSH access.

---

##  **Workflow Overview**
The workflow performs the following steps:
1. **Checkout Repository** - Retrieves the project code.
2. **Install Dependencies** - Installs `kind`, `kubectl`, and `helm`.
3. **Generate Dockerfile** - Creates a Dockerfile based on user inputs.
4. **Build and Push Docker Image** - Builds the image and pushes it to Docker Hub.
5. **Create Kubernetes Cluster** - Spins up a `kind` Kubernetes cluster.
6. **Install Prometheus Operator** - Deploys monitoring tools using Helm.
7. **Apply Kubernetes Manifests** - Deploys services and applications to the cluster.
8. **Expose Services** - Opens up NodePorts for application access.
9. **Install Metrics Server** - Deploys the Kubernetes Metrics Server.
10. **Monitor Resource Usage** - Collects usage data from the running pod.
11. **Upload Usage Data** - Artifacts are uploaded for later analysis.

---

## ⚙️ **Inputs**
| Input           | Description                                     | Default Value       |
|------------------|-------------------------------------------------|---------------------|
| `base_image`    | The base Docker image to use.                   | `ubuntu:20.04`     |
| `packages`      | Comma-separated list of packages to install.    | `curl,vim,git`     |
| `memory_request`| Memory request for Kubernetes deployment.       | `8Gi`              |
| `cpu_request`   | CPU request for Kubernetes deployment.          | `2`                |

---

##  **Run the Workflow**
Trigger the workflow manually with the required inputs:
```bash
gh workflow run kube-deploy-workflow.yml \
  --field base_image=ubuntu:20.04 \
  --field packages="curl,vim,git" \
  --field memory_request=8Gi \
  --field cpu_request=2
```

---

##  **Generated Dockerfile**
Below is a sample Dockerfile that gets generated during the workflow:

```dockerfile
FROM ubuntu:20.04

ENV DEBIAN_FRONTEND=noninteractive

# Install specified packages
RUN apt-get update && apt-get install -y curl vim git && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install OpenSSH server
RUN apt-get update && apt-get install -y openssh-server
RUN mkdir /var/run/sshd && \
    echo 'root:root' | chpasswd && \
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config

# Expose the default SSH port
EXPOSE 22

# Start the SSH server
CMD ["/usr/sbin/sshd", "-D"]
```

---

##  **Services Exposed**
| Service Name                | Type      | Port |
|----------------------------- |-----------|------|
| `dev-environment-service`   | NodePort  | 8080 |
| `dev-environment-ssh`       | NodePort  | 2222 |

---

##  **Monitoring Setup**
Prometheus Operator is deployed for real-time monitoring. Access it by running:
```bash
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090
```
Access Prometheus at: [http://localhost:9090](http://localhost:9090)

---

## **Resource Monitoring**
Resource consumption is collected during the workflow and uploaded as an artifact:
- **Usage Data:** Viewable in the GitHub Actions artifacts section.

---

##  **To-Do / Improvements**
- [ ] Integrate Grafana for better visualization.
- [ ] Automate port-forwarding for Prometheus and Grafana.
- [ ] Implement cleanup for stale `kind` clusters.

---

##  **Contributing**
Feel free to open a PR for improvements or report issues in the repository.

---

##  **License**
MIT License - see [LICENSE](./LICENSE) for more details.

---

##  **DevSecOps Test Solutions**

### **1. Docker Image Creation**
The image was created with `python2`, `python3`, and `R`. It includes all necessary dependencies and is uploaded to Docker Hub. The Dockerfile includes multi-stage builds to optimize image size.

### **2. Build Times and Improvements**
- Current build time: Approximately 4 minutes.
- Improvements:
  - Use Docker layer caching for faster rebuilds.
  - Optimize `apt-get` commands to reduce layers.
  - Leverage Docker BuildKit for parallel builds.

### **3. CVE Scanning and Mitigation**
- The image is scanned with `trivy` for vulnerabilities.
- A report is generated and uploaded as an artifact.
- CVE mitigations include:
  - Upgrading vulnerable packages.
  - Implementing `distroless` images to reduce the attack surface.

### **4. Kubernetes Deployment and Exposure**
- The Docker image is used to create a Kubernetes deployment with a `NodePort` service for external access.

### **5. CI/CD Automation**
- GitHub Actions is used for full CI/CD.
- On code changes, Docker images are built, scanned for CVEs, pushed to Docker Hub, and deployed to Kubernetes.

### **6. Monitoring of the Deployment**
- Prometheus Operator and Metrics Server are installed.
- Resource usage is collected using `kubectl top` and Prometheus.
- Alerts can be set up in Prometheus for over-utilization or under-utilization of resources.

### **7. Scaling and Multi-Environment Strategy**
- Kubernetes `HorizontalPodAutoscaler` is applied for automatic scaling.
- Namespaces are segmented per team to isolate resources.
- Taints and tolerations are applied for resource segregation.

### **8. SFTP and SSH Access Strategy**
- SFTP and SSH are exposed through NodePort services.
- DNS entries are dynamically updated via Kubernetes External DNS.
- Access is managed with RBAC policies for secure, isolated access.

### **9. Handling Large Data Loads (100-250GB)**
- StatefulSets are used for persistent storage.
- EFS or Amazon FSx is mounted for large memory requirements.
- Prometheus monitoring is configured for memory utilization tracking.

---

