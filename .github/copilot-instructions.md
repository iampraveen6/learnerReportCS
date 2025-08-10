
# Copilot Instructions for learnerReportCS

## 1. Big Picture Architecture
This is a full-stack MERN (MongoDB, Express, React, Node.js) application, deployed on Kubernetes using Helm charts, with CI/CD managed by Jenkins. The main components are:
- **backend/**: Node.js REST API, MongoDB integration
- **frontend/**: React SPA
- **kubernetes/**: Raw Kubernetes manifests for deployments/services
- **learner-report/**: Helm chart for templated multi-service deployment

### Data Flow
1. **Frontend** (port 3000) communicates with **Backend** (port 3001) via HTTP.
2. **Backend** connects to **MongoDB** using the `MONGO_URI` environment variable injected by Kubernetes.
3. **Helm** templates all Kubernetes resources for multi-environment deployment.

## 2. File Reference Table
| File / Path | Description |
|-------------|-------------|
| `backend/Dockerfile` | Backend Dockerfile (Node.js, port 3001) |
| `frontend/Dockerfile` | Frontend Dockerfile (React, port 3000) |
| `kubernetes/backend-deployment.yaml` | Backend Deployment & Service manifest |
| `kubernetes/frontend-deployment.yaml` | Frontend Deployment & Service manifest |
| `learner-report/Chart.yaml` | Helm chart metadata |
| `learner-report/values.yaml` | Helm default values |
| `learner-report/templates/backend-deployment.yaml` | Helm backend Deployment template |
| `learner-report/templates/frontend-deployment.yaml` | Helm frontend Deployment template |
| `Jenkinsfile` | Jenkins pipeline for build/push/deploy |

## 3. Developer Workflows

## 3a. Docker Commands
Build and push images for backend and frontend:
```bash
# Build backend image
docker build -t <dockerhub_username>/backend:latest ./backend
# Push backend image
docker push <dockerhub_username>/backend:latest

# Build frontend image
docker build -t <dockerhub_username>/frontend:latest ./frontend
# Push frontend image
docker push <dockerhub_username>/frontend:latest

# (Optional) Remove local images
docker rmi <dockerhub_username>/backend:latest
docker rmi <dockerhub_username>/frontend:latest

# (Optional) View images
docker images
```

## 3b. Helm Chart Commands
Install, upgrade, and uninstall the Helm chart for multi-service deployment:
```bash
# Install chart (first time)
helm install learner-report ./learner-report
# Upgrade chart (after changes)
helm upgrade learner-report ./learner-report
# Uninstall chart
helm uninstall learner-report

# (Optional) Dry run to preview rendered manifests
helm install --dry-run --debug learner-report ./learner-report

# (Optional) List releases
helm list
```

### Kubernetes Manifests
Apply raw manifests directly (if not using Helm):
```bash
kubectl apply -f kubernetes/backend-deployment.yaml
kubectl apply -f kubernetes/frontend-deployment.yaml
```

### Jenkins Pipeline
Automates build, push, and deploy. See `Jenkinsfile` for details. Uses Docker Hub credentials and runs `docker login` in pipeline.

## 4. Deployment Details
- **Backend**: 2 replicas, ClusterIP service on port 3001
- **Frontend**: 2 replicas, NodePort service on port 3000   
- **MongoDB**: Deployed as a separate service, accessible via `MONGO_URI` in backend
- **Resource Requests/Limits**: Set for both backend and MongoDB containers to ensure stability.
- **Security Context**: Not explicitly defined, but can be added for enhanced security.
- **Health Checks**: Not included, but recommended for production readiness.
- **Logging**: Use `kubectl logs <pod_name>` to view logs for debugging.

## 5. Integration Points
- **MongoDB**: Cloud-hosted, accessed via `MONGO_URI`.
- **Docker Hub**: Images built/pushed for frontend and backend.
- **Jenkins**: CI/CD pipeline for automated builds/deployments.
- **Kubernetes**: ClusterIP for backend, NodePort for frontend (local access).
- **Helm**: Templated, multi-service deployment.

## 6. Troubleshooting & Useful Commands
| Issue | Solution |
|-------|----------|
| ImagePullBackOff | Check image name, tag, and Docker Hub push |
| Service not reachable | Use `minikube service frontend-service` |
| Large image size | Use Alpine base images & multi-stage builds |
| Jenkins Docker login fails | Configure credentials in Jenkins & reference in pipeline |

### Common kubectl commands
```bash
kubectl get pods
kubectl get svc
kubectl logs <pod_name>
kubectl describe pod <pod_name>
kubectl port-forward svc/frontend-service 3000:80
```
