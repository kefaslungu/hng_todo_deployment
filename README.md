# Infrastructure Repository for DevOps Stage 4

This repository contains all the Infrastructure-as-Code (IAC) and configuration management files required to provision and deploy the containerized application in Azure. It uses Terraform to provision the Azure resources (e.g., Resource Group, Virtual Network, Linux VM, DNS zone) and Ansible to configure the VM and deploy the application via Docker Compose with Traefik.

---

## Requirements

Before you begin, ensure you have the following installed and configured:

- **Terraform:** Version 1.x or later.  
- **Ansible:** Version 2.9 or later.
- **Azure CLI:** Latest version (for authentication and verification).
- **Git:** To clone this repository and the application repository.
- **SSH Key Pair:** Both a public key (e.g., `HNG_key.pub`) and its corresponding private key (e.g., `HNG_key`) for accessing the Azure VM.
- **Azure Subscription:** Ensure you have an active Azure account with the necessary permissions.
- **DNS Management:** Access to your DNS provider (e.g., HostAfrica) to point your domain to the public IP of the deployed VM.

---

## Setup Instructions

### 1. Clone the Repository

Clone this infrastructure repository to your local machine:

```bash
git clone https://github.com/yourusername/infra-stage4.git
cd infra-stage4
```

### 2. Configure Azure Credentials

Set up your Azure authentication. You can either export the credentials as environment variables or supply them in a `terraform.tfvars` file.

**Using Environment Variables:**

```bash
export ARM_SUBSCRIPTION_ID="your-subscription-id"
export ARM_TENANT_ID="your-tenant-id"
export ARM_CLIENT_ID="your-client-id"
export ARM_CLIENT_SECRET="your-client-secret"
```

Alternatively, add these to your `terraform.tfvars` (see [Configuration](#configuration-and-environment-variables)).

### 3. Review and Edit Configuration Files

- **Terraform Variables:**  
  Check the `variables.tf` file in the `terraform/` directory. Update any values if necessary (e.g., region, resource group name, SSH key paths, domain name, etc.).

- **Terraform.tfvars:**  
  Create or update `terraform.tfvars` with your actual values. For example:

  ```hcl
  domain_name          = "kefaslungu.name.ng"
  resource_group_name  = "HNG_stage4_rg"
  location             = "eastus"
  instance_size        = "Standard_B1s"
  admin_username       = "HNG_kefas"
  ssh_public_key_path  = "../keys/HNG_key.pub"
  ssh_private_key_path = "../keys/HNG_key"
  admin_email          = "jameskefaslungu@gmail.com"
  git_repo_url         = "https://github.com/kefaslungu/DevOps-Stage-4.git"
  git_branch           = "main"
  
  # Azure credentials (if not provided via env variables)
  subscription_id      = "your-subscription-id"
  tenant_id            = "your-tenant-id"
  client_id            = "your-client-id"
  client_secret        = "your-client-secret"
  ```

- **Ansible Configuration:**  
  Verify the Ansible playbook and roles in the `ansible/` directory. The playbook (`playbook.yml`) should reference the roles that install Docker, clone the application repository, and execute `docker-compose up -d --build`.

### 4. Provision and Deploy

Run the following command from the root of your infrastructure repository:

```bash
terraform init
terraform apply -auto-approve
```

Terraform will:
- Provision the Azure infrastructure (Resource Group, Virtual Network, Subnet, NSG, Public IP, Linux VM, DNS zone, etc.).
- Generate Ansible inventory and variables files locally.
- Trigger a null_resource that executes your Ansible playbook to configure the VM and deploy your application via Docker Compose.

---

## Infrastructure and Deployment Process

1. **Terraform Provisioning:**
   - **Resource Group & Virtual Network:**  
     A Resource Group and Virtual Network are created in the specified Azure region.
   - **Linux VM:**  
     A Linux VM is provisioned with the specified instance size. The VM is configured to use your SSH public key for authentication.
   - **Network Security Group (NSG):**  
     NSG rules are configured to allow inbound traffic on required ports (e.g., SSH, HTTP, HTTPS, Traefik).
   - **DNS Zone:**  
     A DNS Zone is created (or updated) to point your domain (`kefaslungu.name.ng`) to the VM's public IP.

2. **Ansible Deployment:**
   - **Dependency Installation:**  
     Ansible installs Docker, Docker Compose, and Git on the VM.
   - **Application Deployment:**  
     The application repository is cloned, and Docker Compose is executed to deploy the application containers (frontend, APIs, etc.) in detached mode.
   - **Traefik Configuration:**  
     If configured, Traefik routes the incoming domain traffic to the correct containers based on host rules.

3. **Accessing the Application:**
   - Once deployed, your application should be accessible at `https://kefaslungu.name.ng`.

---

## Configuration and Environment Variables

The following environment variables and configuration values are required:

- **Domain and DNS:**
  - `domain_name`: Your domain name (e.g., kefaslungu.name.ng).
- **Azure Specific:**
  - `location`: Azure region (e.g., eastus).
  - `resource_group_name`: The name of the Resource Group.
  - `instance_size`: The VM size (e.g., Standard_B1s).
- **Authentication & SSH:**
  - `admin_username`: The username for SSH (e.g., HNG_kefas).
  - `ssh_public_key_path`: Relative path to your SSH public key file.
  - `ssh_private_key_path`: Relative path to your SSH private key file.
- **Git Repository:**
  - `git_repo_url`: URL of the application repository.
  - `git_branch`: Git branch to checkout.
- **Azure Credentials (if not using environment variables):**
  - `subscription_id`
  - `tenant_id`
  - `client_id`
  - `client_secret`
- **Notification:**
  - `admin_email`: Email used for notifications and Let's Encrypt certificates.

You can set these in your `terraform.tfvars` or via your CI/CD pipeline's secret management system.

---

