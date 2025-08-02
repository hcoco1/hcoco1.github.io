---
title: Understanding .tf Files in Terraform
author: hcoco1
date: 2024-07-06 14:10:00 +0800
categories: [Programming, Dev Tools]
tags: [terraform]
render_with_liquid: false
---


Terraform configurations are written in files with the `.tf` extension. These files collectively define the desired state of your infrastructure. While there are no strict rules about the names of these files, certain conventions and best practices can help organize and manage your configurations effectively. Let's break down the purposes and uses of different `.tf` files.

## Commonly Used .tf Files

### `main.tf`

- **Purpose**: The `main.tf` file is typically used as the primary configuration file where the core resources and modules are defined.
- **Mandatory?**: No, `main.tf` is not mandatory. It is a convention to use `main.tf` as the main entry point, but you can name your files whatever you prefer.
- **Usage**: Define core infrastructure resources, provider configurations, and module calls.

```hcl
provider "aws" {
  region = var.region
}

module "vpc" {
  source = "./modules/vpc"
}

module "compute" {
  source = "./modules/compute"
}
```

### `variables.tf`

- **Purpose**: Define input variables that can be used throughout the Terraform configuration.
- **Mandatory?**: No, but it's a good practice to separate variable declarations for clarity.
- **Usage**: Declare variables with default values and descriptions.

```hcl
variable "region" {
  description = "The AWS region to deploy resources"
  default     = "us-west-2"
}

variable "instance_type" {
  description = "The type of EC2 instance"
  default     = "t2.micro"
}
```

### `outputs.tf`

- **Purpose**: Define output values that can be used to extract information from your resources.
- **Mandatory?**: No, but it is useful for exposing resource attributes.
- **Usage**: Specify the values you want to output after applying the configuration.

```hcl
output "instance_id" {
  value = aws_instance.this.id
}

output "instance_public_ip" {
  value = aws_instance.this.public_ip
}
```

### `terraform.tfvars`

- **Purpose**: Provide values for input variables defined in `variables.tf`.
- **Mandatory?**: No, but it's a common practice for setting variable values.
- **Usage**: Define variable values for different environments or contexts.

```hcl
region        = "us-west-2"
instance_type = "t2.micro"
ami           = "ami-0c55b159cbfafe1f0"
```

## Additional .tf Files for Organization

To maintain clarity and manageability, especially in larger projects, it is common to split configurations into multiple `.tf` files based on their functionality. Here are some examples:

### `compute.tf`

- **Purpose**: Define compute resources such as EC2 instances, ECS clusters, or Lambda functions.
- **Usage**: Separate compute-related configurations from other parts of the infrastructure.

```hcl
resource "aws_instance" "web" {
  ami           = var.ami
  instance_type = var.instance_type
}
```

### `network.tf`

- **Purpose**: Define network resources like VPCs, subnets, security groups, and route tables.
- **Usage**: Isolate network configurations to simplify management and enhance clarity.

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "subnet" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
}
```

### `data.tf`

- **Purpose**: Define data sources to query information about existing infrastructure.
- **Usage**: Separate data sources from resource declarations.

```hcl
data "aws_ami" "latest" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}
```

### `providers.tf`

- **Purpose**: Define provider configurations.
- **Usage**: Isolate provider setup to enhance organization and clarity.

```hcl
provider "aws" {
  region = var.region
}
```

## Best Practices for File Organization

1. **Logical Separation**: Organize resources into different files based on their type and function (e.g., compute, network).
2. **Clarity and Readability**: Use descriptive file names to indicate their purpose, making it easier for others to understand and navigate the configuration.
3. **Modularity**: Use modules to encapsulate reusable components and separate them from the main configuration.
4. **Environment-specific Configurations**: Use `terraform.tfvars` or environment-specific variable files (e.g., `production.tfvars`, `staging.tfvars`) to manage configurations across different environments.

## Example Directory Structure

```plaintext
.
├── main.tf
├── variables.tf
├── outputs.tf
├── compute.tf
├── network.tf
├── data.tf
├── providers.tf
├── terraform.tfvars
├── modules
│   └── vpc
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
│   └── compute
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
```

By following these conventions and best practices, you can create Terraform configurations that are easy to manage, understand, and scale as your infrastructure grows.