# TechPathway Enterprise Capstone

## Project Overview
This project containerizes and deploys three TechPathway applications to AWS EKS using Docker, Amazon ECR, Kubernetes, Helm, Prometheus, and Grafana.

## Applications
- Action Messages - Port 5001
- TechPathway Warehouse - Port 5002
- Weekly Call - Port 5111

## Technologies
- Docker
- Amazon ECR
- Amazon EKS
- Kubernetes
- Helm
- Prometheus
- Grafana

## Kubernetes Resources
The kubernetes folder contains:
- Deployments for all three applications
- ClusterIP Services
- Namespace
- ConfigMap
- Secret example
- Horizontal Pod Autoscaler

## Scaling
Manual scaling was tested from 1 to 5 replicas and back to 2 replicas.

Kubernetes self-healing was validated by deleting a Pod and confirming that the Deployment automatically created a replacement.

## Horizontal Pod Autoscaling
HPA was configured for Action Messages with:
- Minimum replicas: 1
- Maximum replicas: 5
- CPU target: 50%

Traffic generation increased CPU utilization and automatically scaled the application up to 5 replicas. After traffic stopped, the HPA automatically scaled the application back down.

## Monitoring
Prometheus and Grafana were installed using the kube-prometheus-stack Helm chart.

Grafana was used to visualize Kubernetes cluster CPU, memory, workloads, namespaces, and Pod metrics.

## Troubleshooting
The following Kubernetes failure scenarios were tested and investigated:
- ImagePullBackOff
- CrashLoopBackOff
- Pending Pod caused by insufficient CPU
- Incorrect Service target port
- Readiness probe failure
- Liveness probe failure

## Cleanup
At the end of testing:
- Monitoring resources were removed
- Application namespace was removed
- EKS cluster and worker nodes were deleted
- ECR repositories were deleted

## Security
Sensitive credentials are not committed to source control. The repository contains secret-example.yaml as a template only.
