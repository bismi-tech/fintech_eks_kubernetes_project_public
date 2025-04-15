# fintech_eks_kubernetes_project_public

# AWS EKS Secure Deployment Project

## Overview

This project demonstrates a production-ready deployment of a containerized application on AWS Elastic Kubernetes Service (EKS) using Terraform for infrastructure as code. The implementation follows AWS best practices for security, scalability, and automation.

![AWS Architecture Diagram](./architecture-diagram.png)

## Key Features

- **Infrastructure as Code**: Complete Terraform implementation for reproducible deployments
- **Security-First Design**: Comprehensive security controls at network, IAM, and application levels
- **High Availability**: Multi-AZ deployment with resilient architecture
- **Cross-Account Communication**: Secure service-to-service communication across AWS accounts
- **Scalability**: Automatic scaling at pod and node levels based on workload

## Architecture Components

### Networking
- VPC with public and private subnets across 3 availability zones
- NAT Gateways for outbound connectivity from private subnets
- Network ACLs and Security Groups for defense-in-depth
- Application Load Balancer for secure service exposure

### Kubernetes Infrastructure
- EKS cluster with managed node groups
- AWS Load Balancer Controller for integration with ALB/NLB
- Kubernetes service and ingress resources for application exposure
- Pod-level security controls and resource management

### Security
- Least privilege IAM roles with fine-grained permissions
- IAM Roles for Service Accounts (IRSA) for pod-level access control
- KMS encryption for EKS secrets
- Private API endpoint with controlled access
- Network isolation using security groups and network policies

### Multi-Account Strategy
- AWS PrivateLink for secure cross-account service communication
- IAM role assumption with strict conditions
- Endpoint policies for granular access control

## Prerequisites

- AWS CLI configured with appropriate credentials
- Terraform v1.0.0+
- kubectl v1.21.0+
- Docker (for local testing)

## Deployment Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/your-repo/eks-kubernetes-project.git
cd eks-kubernetes-project
```

### 2. Configure AWS Credentials

```bash
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-east-1"
```

### 3. Initialize Terraform

```bash
terraform init
```

This command initializes your working directory containing Terraform configuration files, installs providers defined in the configuration, and sets up the backend.

### 4. Format and Validate Terraform Code

```bash
terraform fmt
terraform validate
```

These commands format your Terraform code for consistency and readability, and validate the syntax and configuration of your Terraform files.

### 5. Create an Execution Plan

```bash
terraform plan -out=tfplan
```

This creates an execution plan showing what changes Terraform will make. The `-out` flag saves the plan to a file that can be used in the apply step.

### 6. Apply the Changes

```bash
terraform apply tfplan
```

This applies the changes required to reach the desired state defined in your configuration. Using the saved plan file ensures you apply exactly what you reviewed.

### 7. Configure kubectl to Work with the New Cluster

```bash
aws eks update-kubeconfig --name amex-eks-demo --region us-east-1
```

This updates your kubeconfig file with the necessary credentials to interact with your EKS cluster.

### 8. Verify the Deployment

```bash
# Verify nodes are running
kubectl get nodes

# Verify all pods are running
kubectl get pods -A

# Check service endpoints
kubectl get svc -n default

# Get the ALB endpoint
kubectl get ingress -n default
```

## Project Structure

```
eks-kubernetes-project/
├── README.md                       # This file
├── main.tf                         # Main Terraform file
├── variables.tf                    # Variable definitions
├── outputs.tf                      # Output definitions
├── backend.tf                      # Backend configuration
├── modules/
│   ├── vpc/                        # VPC module
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── eks/                        # EKS module
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── iam/                        # IAM module
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── alb/                        # ALB and Ingress Controller module
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── app/                        # Application deployment module
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── policies/                       # IAM policy documents
│   ├── s3-read-only.json
│   └── cross-account-access.json
├── kubernetes/                     # Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── docs/                           # Documentation
    ├── architecture-diagram.png
    └── cross-account-communication.md
```

## Module Details

### VPC Module (`vpc.tf`)

The VPC module sets up the networking foundation with:
- CIDR block `10.0.0.0/16`
- Public and private subnets across 3 AZs
- NAT Gateways in each public subnet
- Network ACLs for subnet isolation
- DNS support for EKS service discovery

### EKS Module (`eks.tf`)

The EKS module provisions the Kubernetes cluster with:
- EKS version 1.28
- T3.medium instances for worker nodes
- Auto-scaling configuration (2-5 nodes)
- KMS encryption for secrets
- Comprehensive logging to CloudWatch
- IRSA enabled for pod-level permissions

### IAM Module (`iam.tf`)

The IAM module creates security roles with:
- KMS key for EKS secrets encryption
- S3 read-only policy for application pods
- Cross-account role for secure communication
- Least privilege permissions throughout

### ALB Module (`alb.tf`) 

The ALB module sets up external access with:
- AWS Load Balancer Controller via Helm
- ALB with SSL termination
- HTTP to HTTPS redirection
- Health checks and connection draining
- Security group rules for controlled access

### Application Module (`app.tf`)

The application module deploys the containerized application with:
- Kubernetes Deployment with 3 replicas
- Service Account with IAM role annotations
- Resource requests and limits
- Liveness and readiness probes
- Horizontal Pod Autoscaler for scaling

## Cross-Account Communication

This project implements a secure cross-account communication strategy using AWS PrivateLink. For detailed information, see [Cross-Account Communication Strategy](./docs/cross-account-communication.md).

Key benefits of this approach:
- Traffic never traverses the public internet
- Service-level exposure rather than network-level
- Combined network and identity-based security
- Simplified management compared to VPC peering
- Compliance with regulatory requirements

## Security Best Practices

This implementation follows these security best practices:
1. **Least Privilege**: IAM roles with minimal required permissions
2. **Defense in Depth**: Multiple security layers (VPC, NACL, SG, IAM, K8s policies)
3. **Encryption**: Data encrypted both in transit and at rest
4. **Private Networking**: No direct internet access to worker nodes
5. **Secrets Management**: Secure handling of credentials and sensitive information
6. **Logging and Monitoring**: Comprehensive audit trail of all activities

## Scalability and Maintainability

The architecture is designed for both scalability and maintainability:
- Horizontal Pod Autoscaler adjusts pod count based on CPU/memory metrics
- Cluster Autoscaler adds/removes nodes based on pod scheduling needs
- Infrastructure as Code enables version-controlled changes
- Modular design allows for component updates without system-wide changes
- Consistent tagging strategy for resource management

## Monitoring and Logging

The implementation includes:
- CloudWatch Container Insights for metrics
- Fluent Bit for container log collection
- Prometheus and Grafana for advanced monitoring (optional)
- Kubernetes Event Exporter for cluster event tracking




# Cross-Account Communication Strategy

## Problem Statement

Our application in Account A needs to communicate with another application in Account B within the same AWS Organization without exposing services to the public internet.

## Chosen Solution: AWS PrivateLink with VPC Endpoints

I've implemented an AWS PrivateLink solution for secure cross-account communication. This approach uses VPC endpoints to create private connections between VPCs in different accounts without traffic traversing the public internet.

### Implementation Details

#### In Account B (Service Provider):

1. **Network Load Balancer Configuration**:
   - Created a Network Load Balancer (NLB) in front of the application service
   - Configured the NLB in private subnets
   - Set up target groups with the appropriate health checks

2. **VPC Endpoint Service Setup**:
   - Created a VPC Endpoint Service (AWS PrivateLink) attached to the NLB
   - Configured allowlist to accept connections only from Account A's VPC
   - Applied service-level endpoint policies

3. **Service Authentication**:
   - Implemented IAM policies for service authentication
   - Added resource-based policies to restrict access

#### In Account A (Service Consumer):

1. **Interface VPC Endpoint Creation**:
   - Created an Interface VPC Endpoint to connect to Account B's VPC Endpoint Service
   - Placed endpoint in private subnets
   - Applied security group rules to restrict access to authorized sources

2. **IAM Configuration**:
   - Created IAM roles with STS AssumeRole for cross-account authentication
   - Applied least privilege permissions
   - Set up conditions based on organization ID and source VPC

3. **DNS Resolution**:
   - Used private hosted zones for DNS resolution
   - Ensured service discovery for applications

### Key Security Features

- **Network Isolation**: All traffic stays within the AWS network, never traversing the public internet
- **Granular Access Control**: Endpoint policies restrict which VPCs can establish connections
- **Authentication**: IAM roles with cross-account trust relationships ensure proper authorization
- **Encryption**: Traffic is automatically encrypted in transit
- **Audit Trail**: CloudTrail logs all API calls related to cross-account access
- **Principle of Least Privilege**: Services only have access to exactly what they need

## Implementation Code Example

Here's a sample Terraform configuration for setting up the PrivateLink connection:

```hcl
# In Account B (Service Provider)

# Network Load Balancer
resource "aws_lb" "service_nlb" {
  name               = "service-nlb"
  internal           = true
  load_balancer_type = "network"
  subnets            = module.vpc.private_subnets
}

# VPC Endpoint Service
resource "aws_vpc_endpoint_service" "service" {
  acceptance_required        = false
  network_load_balancer_arns = [aws_lb.service_nlb.arn]
  allowed_principals         = ["arn:aws:iam::${var.account_a_id}:root"]
}

# In Account A (Service Consumer)

# Interface VPC Endpoint
resource "aws_vpc_endpoint" "service_endpoint" {
  vpc_id             = module.vpc.vpc_id
  service_name       = "com.amazonaws.vpce.${var.region}.${var.account_b_id}.${aws_vpc_endpoint_service.service.service_name}"
  vpc_endpoint_type  = "Interface"
  subnet_ids         = module.vpc.private_subnets
  security_group_ids = [aws_security_group.endpoint_sg.id]
  private_dns_enabled = true
}

# Security Group for Endpoint
resource "aws_security_group" "endpoint_sg" {
  name        = "service-endpoint-sg"
  description = "Security group for service VPC endpoint"
  vpc_id      = module.vpc.vpc_id

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = [module.vpc.vpc_cidr_block]
  }
}
```

## Alternatives Considered

### 1. VPC Peering

**Pros**:
- Simple to set up for a small number of VPCs
- Low latency as traffic stays within AWS network

**Cons**:
- Exposes the entire network rather than just specific services
- Requires careful CIDR planning to avoid IP overlap
- Can become complex to manage with many VPCs
- Less granular access control compared to PrivateLink

### 2. Transit Gateway

**Pros**:
- Centralized hub for connecting multiple VPCs
- Simplified routing for complex networks
- Regional resource with inter-region peering options

**Cons**:
- Higher cost for our specific use case
- More complex than needed for connecting just two services
- Still exposes more of the network than necessary

### 3. API Gateway with Resource Policies

**Pros**:
- Managed service with built-in caching, throttling, and monitoring
- API versioning capabilities

**Cons**:
- Services would be exposed to the public internet
- Larger attack surface even with resource policies in place
- Additional hop in the network path affecting latency

## Why PrivateLink is Superior for This Use Case

PrivateLink was chosen because it provides the most secure and targeted solution:

1. **Service-Level Exposure**: Only exposes the specific application service, not the entire network
2. **Simplified Network Management**: No need to manage overlapping CIDRs or complex routing tables
3. **Reduced Attack Surface**: No public internet exposure at any point
4. **Scalable**: Can easily expose additional services as needed
5. **Performance**: Low-latency, high-throughput connections
6. **Compliance**: Helps meet regulatory requirements for data never traversing the public internet

This approach aligns perfectly with the principle of least privilege and provides the most secure communication channel between our accounts while maintaining high performance.

## Integration with Pod Identity

To enable pods in the EKS cluster to securely communicate with the service in Account B:

1. We use IAM Roles for Service Accounts (IRSA) to provide pod-level credentials
2. The pod assumes the IAM role with permissions to access the VPC endpoint
3. The role has permissions to assume the cross-account role in Account B
4. This creates a secure identity chain from pod to service

## Monitoring and Audit

The implementation includes:

1. CloudTrail logging for all API calls
2. VPC Flow Logs for network traffic analysis
3. CloudWatch metrics for endpoint usage and performance
4. Custom alarms for unexpected access patterns

## Disaster Recovery Considerations

The cross-account communication architecture supports disaster recovery by:

1. Enabling replication of the same setup across regions
2. Supporting failover mechanisms through DNS routing policies
3. Allowing for redundant endpoints in multiple availability zones




















## License

[MIT License](LICENSE)

## Contact

For questions or support, please contact:
- Your Name: your.email@example.com
- GitHub: [Your GitHub Profile](https://github.com/bismi-tech)