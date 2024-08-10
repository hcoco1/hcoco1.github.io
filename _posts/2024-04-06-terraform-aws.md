---
title: Terraform -- AWS
author: hcoco1
date: 2024-04-06 14:10:00 +0800
categories: [Programming, Tools]
tags: [terraform, aws]
render_with_liquid: false
---


This example demonstrates best practices in using Terraform to manage AWS infrastructure, including remote state management, module organization, version control, and planning/review processes.

## Directory Structure

```plaintext
.
├── main.tf
├── variables.tf
├── outputs.tf
├── modules
│   └── ec2-instance
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
└── terraform.tfvars
```

## 1. Remote State Management

First, configure Terraform to store the state file in an S3 bucket to ensure consistency and enable collaboration.

### `main.tf`

```hcl
provider "aws" {
  region = var.region
}

terraform {
  backend "s3" {
    bucket         = "your-terraform-state-bucket"
    key            = "path/to/terraform.tfstate"
    region         = var.region
    encrypt        = true
    dynamodb_table = "your-lock-table"
  }
}

module "ec2_instance" {
  source = "./modules/ec2-instance"

  instance_type = var.instance_type
  ami           = var.ami
}
```

### `variables.tf`

```hcl
variable "region" {
  description = "AWS region"
  default     = "us-west-2"
}

variable "instance_type" {
  description = "Type of the instance"
  default     = "t2.micro"
}

variable "ami" {
  description = "AMI ID"
  default     = "ami-0c55b159cbfafe1f0"
}
```

### `outputs.tf`

```hcl
output "instance_id" {
  value = module.ec2_instance.instance_id
}

output "instance_public_ip" {
  value = module.ec2_instance.instance_public_ip
}
```

## 2. Module Organization

Organize your Terraform configurations into modules for better manageability.

### `modules/ec2-instance/main.tf`

```hcl
resource "aws_instance" "this" {
  ami           = var.ami
  instance_type = var.instance_type

  tags = {
    Name = "ExampleInstance"
  }
}
```

### `modules/ec2-instance/variables.tf`

```hcl
variable "instance_type" {
  description = "Type of the instance"
}

variable "ami" {
  description = "AMI ID"
}
```

### `modules/ec2-instance/outputs.tf`

```hcl
output "instance_id" {
  value = aws_instance.this.id
}

output "instance_public_ip" {
  value = aws_instance.this.public_ip
}
```

## 3. Version Control

Keep your Terraform configurations in a version control system like Git.

### `.gitignore`

Create a `.gitignore` file to ignore sensitive files and directories.

```plaintext
*.tfstate
*.tfstate.backup
.terraform/
.terraform.lock.hcl
```

Initialize a Git repository and commit your Terraform files.

```sh
git init
git add .
git commit -m "Initial commit of Terraform configurations"
```

## 4. Plan and Review

Always run `terraform plan` before applying changes to review what will be modified.

### Commands

1. **Initialize Terraform**:

   ```sh
   terraform init
   ```

2. **Plan Changes**:

   ```sh
   terraform plan
   ```

3. **Apply Changes**:

   ```sh
   terraform apply
   ```

## `terraform.tfvars`

Use a `terraform.tfvars` file to set variable values.

```hcl
region        = "us-west-2"
instance_type = "t2.micro"
ami           = "ami-0c55b159cbfafe1f0"
```


## Explanation of "this"

In Terraform, the second string in the resource block declaration (in this case, `"this"`) is the name or identifier of the resource within the Terraform configuration. This identifier is used to reference this specific resource elsewhere in the configuration or outputs.



The general syntax for defining a resource in Terraform is:

```hcl
resource "<PROVIDER>_<RESOURCE_TYPE>" "<NAME>" {
  # Configuration options
}
```

- **`<PROVIDER>`**: The name of the provider, such as `aws`, `google`, `azurerm`, etc.
- **`<RESOURCE_TYPE>`**: The type of resource being defined, such as `instance`, `s3_bucket`, `vpc`, etc.
- **`<NAME>`**: A unique identifier for this resource within the configuration.

### Example: `aws_instance.this`

```hcl
resource "aws_instance" "this" {
  ami           = var.ami
  instance_type = var.instance_type

  tags = {
    Name = "ExampleInstance"
  }
}
```

- **`aws_instance`**: Specifies that the resource type is an AWS EC2 instance.
- **`this`**: Is a unique name for this specific EC2 instance within the Terraform configuration. You can use any valid identifier here.

### Usage

The name `"this"` (or any chosen identifier) is used to reference this resource in other parts of the configuration. For example, if you want to output the instance ID and public IP, you would reference it like this:

#### `outputs.tf`

```hcl
output "instance_id" {
  value = aws_instance.this.id
}

output "instance_public_ip" {
  value = aws_instance.this.public_ip
}
```

In this example:
- `aws_instance.this.id` refers to the `id` attribute of the `aws_instance` resource named `this`.
- `aws_instance.this.public_ip` refers to the `public_ip` attribute of the same resource.

### Naming Convention

Using `"this"` as the identifier is a common convention when there is only one resource of that type, but you can choose more descriptive names, especially in more complex configurations with multiple similar resources. For example:

```hcl
resource "aws_instance" "web_server" {
  ami           = var.ami
  instance_type = var.instance_type

  tags = {
    Name = "WebServerInstance"
  }
}

resource "aws_instance" "db_server" {
  ami           = var.ami
  instance_type = var.instance_type

  tags = {
    Name = "DBServerInstance"
  }
}
```

In this case, `"web_server"` and `"db_server"` are the identifiers, making it clear what each resource represents.

Using meaningful names helps improve the readability and maintainability of your Terraform configurations, especially in larger projects.