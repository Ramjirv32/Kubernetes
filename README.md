# Kubernetes

A comprehensive Kubernetes learning and deployment project featuring a Go-based microservice with various Kubernetes resources including Pods, Deployments, Services, ConfigMaps, and Ingress configurations.

## Project Structure

```
.
├── README.md
├── Development/
└── Initial/
    ├── 1.yaml                    # Basic nginx Pod
    ├── deployment.yaml           # nginx Deployment with ConfigMap
    ├── pod.yaml                  # MongoDB Pod
    └── go-server/               # Go Gin API microservice
        ├── .env
        ├── cm.yaml              # ConfigMap for environment variables
        ├── deployment.yaml      # Go server Deployment
        ├── Dockerfile           # Container image definition
        ├── go.mod               # Go module dependencies
        ├── ingress.yaml         # Ingress routing configuration
        ├── main.go              # Go application entry point
        └── service.yaml         # NodePort Service
```

## Components

### Go Server Microservice

A lightweight REST API built with the Gin framework.

- **Port**: 14000
- **Image**: `ramjirv3217/go-gin-api:latest`
- **Replicas**: 3
- **Service Type**: NodePort (30007)

#### Endpoints

- `GET /` - Returns a welcome message

### Kubernetes Resources

#### ConfigMap ([Initial/go-server/cm.yaml](Initial/go-server/cm.yaml))
Stores configuration data:
- `PORT`: 14000
- `db-port`: 27017

#### Deployment ([Initial/go-server/deployment.yaml](Initial/go-server/deployment.yaml))
- 3 replicas of the Go server
- Environment variables injected from ConfigMap
- Volume mount for configuration files

#### Service ([Initial/go-server/service.yaml](Initial/go-server/service.yaml))
- Type: NodePort
- Internal port: 80
- Target port: 14000
- Node port: 30007

#### Ingress ([Initial/go-server/ingress.yaml](Initial/go-server/ingress.yaml))
- Host: `foo.bar.com`
- Path: `/bar`
- Backend: go-server-service:80

## Prerequisites

- Kubernetes cluster (Minikube, kind, or cloud provider)
- kubectl CLI tool
- Docker (for building images)
- Go 1.24+ (for local development)

## Setup Instructions

### 1. Build and Push Docker Image (Optional)

```bash
cd Initial/go-server
docker build -t ramjirv3217/go-gin-api:latest .
docker push ramjirv3217/go-gin-api:latest
```

### 2. Deploy ConfigMap

```bash
kubectl apply -f Initial/go-server/cm.yaml
```

### 3. Deploy the Application

```bash
kubectl apply -f Initial/go-server/deployment.yaml
```

### 4. Expose the Service

```bash
kubectl apply -f Initial/go-server/service.yaml
```

### 5. Configure Ingress (Optional)

```bash
# Enable ingress controller (Minikube example)
minikube addons enable ingress

# Apply ingress configuration
kubectl apply -f Initial/go-server/ingress.yaml

# Update /etc/hosts for local testing
echo "$(minikube ip) foo.bar.com" | sudo tee -a /etc/hosts
```

## Accessing the Application

### Via NodePort

```bash
# Get the node IP
kubectl get nodes -o wide

# Access the service
curl http://<NODE_IP>:30007/
```

### Via Minikube

```bash
minikube service go-server-service
```

### Via Ingress

```bash
curl http://foo.bar.com/bar/
```

## Local Development

### Run the Go Server Locally

```bash
cd Initial/go-server
go mod download
go run main.go
```

The server will start on port 14000 (configured in [.env](Initial/go-server/.env)).

### Test the API

```bash
curl http://localhost:14000/
```

Expected response:
```json
{
  "message": "Hi, Gin server is running!"
}
```

## Kubernetes Commands

### View Resources

```bash
# Check deployments
kubectl get deployments

# Check pods
kubectl get pods

# Check services
kubectl get services

# Check configmaps
kubectl get configmaps

# Check ingress
kubectl get ingress
```

### View Logs

```bash
# Get pod name
kubectl get pods

# View logs
kubectl logs <pod-name>

# Follow logs
kubectl logs -f <pod-name>
```

### Scaling

```bash
# Scale deployment
kubectl scale deployment go-server-v1 --replicas=5
```

### Debugging

```bash
# Describe resources
kubectl describe deployment go-server-v1
kubectl describe service go-server-service
kubectl describe configmap test-cm

# Execute commands in pod
kubectl exec -it <pod-name> -- /bin/sh

# Port forwarding for debugging
kubectl port-forward service/go-server-service 8080:80
```

## Additional Examples

### Basic nginx Pod ([Initial/1.yaml](Initial/1.yaml))
Simple Pod running nginx 1.14.2

### nginx Deployment ([Initial/deployment.yaml](Initial/deployment.yaml))
Deployment with 10 replicas using ConfigMap for environment variables

### MongoDB Pod ([Initial/pod.yaml](Initial/pod.yaml))
Single Pod running MongoDB 7.0

## Technologies Used

- **Kubernetes**: Container orchestration
- **Go**: Backend programming language
- **Gin**: Web framework for Go
- **Docker**: Containerization
- **nginx**: Web server (examples)
- **MongoDB**: Database (pod example)

## Environment Variables

| Variable | Source | Description |
|----------|--------|-------------|
| `PORT` | ConfigMap | Application port (14000) |
| `DB-PORT` | ConfigMap | Database port (27017) |

## Troubleshooting

### Pods not starting
```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

### Service not accessible
```bash
kubectl get endpoints go-server-service
kubectl describe service go-server-service
```

### ConfigMap not loading
```bash
kubectl get configmap test-cm -o yaml
```

## Contributing

Feel free to submit issues and enhancement requests!

## License

This project is open source and available for educational purposes.