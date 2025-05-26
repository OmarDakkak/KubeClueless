# KubeClueless

Lost in the world of Kubernetes? KubeClueless is your friendly guide from newbie to K8s pro! With simple tutorials, cheat sheets, and practical examples, this repo helps you navigate pods, services, and beyond. Perfect for those just starting or looking to avoid 'kubectl' chaos. Let's learn Kubernetes together, one step at a time!

## Repository Structure

This repository is organized into key Kubernetes topic areas to help you master Kubernetes concepts and prepare for certifications like the Certified Kubernetes Application Developer (CKAD) exam.

### Core Concepts
- **Pods**: The smallest deployable units in Kubernetes
- **ReplicaSets**: Ensuring a specified number of pod replicas are running
- **Deployments**: Declarative updates for pods and ReplicaSets
- **Namespaces**: Virtual clusters within a physical cluster
- **Imperative Commands**: Quick kubectl commands for resource management
- **Helm**: The package manager for Kubernetes

### Configuration
- **Commands and Arguments**: Customizing container startup behaviors
- **ConfigMaps**: External configuration data for applications
- **Secrets**: Sensitive information management
- **Resource Requirements**: Specifying CPU and memory requirements
- **Security Contexts**: Pod and container security settings
- **Service Accounts**: Identities for processes running in pods

### Multi-Container Pods
- **Design Patterns**: Ambassador, adapter, sidecar patterns
- **Init Containers**: Specialized containers that run before app containers
- **Sidecar Patterns**: Enhancing the main container functionality

### Pod Design
- **Labels and Selectors**: Organizing and selecting Kubernetes objects
- **Annotations**: Non-identifying metadata for objects
- **Deployments Strategies**: Rolling updates, blue/green, canary deployments
- **Jobs**: One-time task execution
- **CronJobs**: Scheduled task execution

### State Persistence
- **Volumes**: Container storage that persists beyond container lifecycle
- **PersistentVolumes**: Cluster storage resources
- **PersistentVolumeClaims**: Storage requests by users

### Services and Networking
- **Services**: Exposing applications running on pods
- **NetworkPolicies**: Controlling traffic flow between pods
- **Ingress**: Managing external access to services

### Observability
- **Probes**: Application health checking mechanisms
- **Logging**: Capturing and accessing container logs
- **Debugging**: Troubleshooting Kubernetes applications

### CKAD Practice
- **Kubectl Efficiency**: Tips to use kubectl effectively
- **Time Management**: Strategies for time management during the exam
- **Mock Exams**: Practice scenarios for certification preparation

## Getting Started

Browse through the folders based on the topic you're interested in learning. Each markdown file contains:
- Core concepts explanation
- YAML examples
- Best practices
- Common troubleshooting tips
- Exam-relevant information (for certification preparation)

## Contributing

Feel free to contribute by:
1. Adding new examples
2. Improving explanations
3. Fixing errors
4. Suggesting additional topics

## License

This repository is available under the [MIT License](LICENSE).
