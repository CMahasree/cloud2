# Typing Speed Test - DevOps Project

## A. Project Overview
This project is a modern Typing Speed Test web application. It calculates WPM (Words Per Minute), accuracy, and correct/incorrect characters in real-time. The project demonstrates a complete DevOps lifecycle using modern tools.

## B. Architecture
*   **Frontend**: HTML, CSS, JavaScript (Vanilla, no framework)
*   **Backend**: Python Flask
*   **Containerization**: Docker
*   **Automation**: Ansible
*   **Orchestration**: Kubernetes (Kind - local cluster)
*   **Infrastructure as Code**: Terraform
*   **CI/CD**: Jenkins

## C. Prerequisites
Before running this project, ensure you have the following installed on your Windows machine (with WSL2/PowerShell):
1.  [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2.  [Python 3.9+](https://www.python.org/downloads/)
3.  [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) (Best run in WSL or Linux environment)
4.  [kubectl](https://kubernetes.io/docs/tasks/tools/)
5.  [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
6.  [Terraform](https://developer.hashicorp.com/terraform/downloads)

## D. Installation
Clone the repository and navigate into it:
```bash
cd typing-speed-test
```

## E. Docker Commands
To build and run the application locally using Docker:
```bash
# Build the Docker image
docker build -t typing-speed-test:latest .

# Run the container locally
docker run -d -p 5000:5000 --name typing-app typing-speed-test:latest

# Or use docker-compose
docker-compose up -d
```
Access the application at: `http://localhost:5000`

## F. Ansible Commands
We use Ansible to verify dependencies, create directories, build the Docker image, and set up the Kind cluster.
```bash
# Run the Ansible playbook
ansible-playbook -i ansible/inventory.ini ansible/playbook.yml
```

## G. Kind Cluster Creation
If not using Ansible, you can create the Kind cluster manually:
```bash
# Create cluster
kind create cluster --name typing-cluster

# Load Docker image into the cluster
kind load docker-image typing-speed-test:latest --name typing-cluster
```

## H. Kubernetes Deployment
To deploy using raw Kubernetes manifests (Optional, we recommend Terraform):
```bash
kubectl apply -f kubernetes/
```

## I. Terraform Initialization and Apply
We use Terraform to manage the Kubernetes resources inside our Kind cluster.
*Note: Terraform here is managing Kubernetes resources (Pods/Deployments/Services) on the existing local Kind cluster, not creating physical nodes.*
```bash
cd terraform
terraform init
terraform plan
terraform apply
```

## J. Terraform Scaling
To scale the application pods using Terraform:
```bash
# Scale to 3 replicas
terraform apply -var="replicas=3"

# Verify pods
kubectl get pods -n typing-speed-test
```

## K. Jenkins Setup
A Jenkinsfile is provided for CI/CD automation. It includes stages for:
1.  Checkout
2.  Install/Validate Dependencies
3.  Run Tests (Pytest)
4.  Build Docker Image
5.  Docker Image Test
6.  Deploy to Kubernetes
7.  Terraform Apply
8.  Deployment Verification

To use it, create a new Pipeline job in your Jenkins server and point it to this repository.

## L. How to Access the Application
Once deployed via Kubernetes (NodePort):
1.  Find the NodePort (it is configured to `30005` in `service.yaml`).
2.  If using Docker Desktop on Windows, you can access it via: `http://localhost:30005`
3.  Alternatively, use port forwarding:
    ```bash
    kubectl port-forward service/typing-app-service -n typing-speed-test 5000:5000
    ```
    Then visit `http://localhost:5000`.

## M. Troubleshooting
*   **Docker not found**: Ensure Docker Desktop is running.
*   **ImagePullBackOff**: Ensure you loaded the Docker image into the Kind cluster (`kind load docker-image ...`).
*   **Terraform cannot connect to Kubernetes**: Ensure your `~/.kube/config` is pointing to the `kind-typing-cluster` context (`kubectl config use-context kind-typing-cluster`).

## N. How to stop/delete everything
```bash
# Destroy Terraform resources
cd terraform
terraform destroy

# Delete Kind cluster
kind delete cluster --name typing-cluster

# Remove local Docker containers (if any)
docker rm -f typing-app
```
