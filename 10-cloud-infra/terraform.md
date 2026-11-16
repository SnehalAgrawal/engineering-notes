# Cloud Architecture: Terraform (Infrastructure as Code)

### 1. Overview
Terraform describes infrastructure (networks, databases, DNS, IAM) in HCL files. It compares the files with what exists and makes the API calls to close the gap. Changes then go through code review instead of console clicks. Terraform has used the Business Source License since 2023. OpenTofu is the open-source fork with a mostly compatible CLI.

### 2. Key Concepts
*   **Provider**: A plugin for one API, such as AWS, GCP, Kubernetes, or Cloudflare.
*   **Resource**: Something Terraform creates and manages, such as a bucket.
*   **Data source**: Something Terraform only reads, such as the latest AMI.
*   **Module**: A reusable group of resources with inputs and outputs.
*   **State**: A file that maps each block in the code to a real object ID. The real cloud is the truth. State is how Terraform knows which object a block means.
*   **Plan and apply**: `plan` shows what will be created, changed, or destroyed. `apply` does it.

### 3. Real-World Usage
*   **One workflow for many APIs**: AWS, DNS, and monitoring in the same tool. The code is not portable between clouds; each provider has its own resources.
*   **History**: Infrastructure changes are reviewed in pull requests and tracked in Git.
*   **New environments**: The same modules with different variables create a new environment or region. Data must be restored separately.
*   **Drift detection**: A scheduled `terraform plan -detailed-exitcode` returns exit code 2 when someone changed things by hand.

### 4. Tradeoffs
*   **Declarative**: You write the end state and Terraform works out the order. Some attribute changes force a replace (destroy and recreate), and the plan shows this.
*   **State risk**: State must be stored remotely, locked, and backed up. If it is lost, Terraform forgets what it manages and resources must be imported again.
*   **HCL limits**: Loops and conditions (`for_each`, `count`) are less flexible than a real programming language. Pulumi and CDK for Terraform use general-purpose languages.

### 5. When NOT to Use
*   **Quick experiments**: The console is faster for a short test. Delete it afterwards.
*   **Runtime objects**: Pods, autoscaled instances, and per-user resources belong to the app or autoscaler, not Terraform.
*   **App releases**: Deploy app versions with Helm, Argo CD, or a serverless tool. Terraform manages the platform under them.

### 6. Interview Focus
*   **State locking**: "Why use a remote backend with locking?"
    *   Two applies at once can overwrite each other's state. A lock makes the second one wait or fail.
    *   A remote backend also shares state with the team and CI, encrypts it, and keeps old versions.
    *   Examples: S3 (recent versions lock with `use_lockfile = true`; DynamoDB locking is the older way), GCS, Azure Blob, HCP Terraform.
*   **Lifecycle**: "What do `prevent_destroy` and `create_before_destroy` do?"
    *   `prevent_destroy` makes any plan that destroys the resource fail. Deleting the block removes the protection too.
    *   `create_before_destroy` builds the replacement first, then deletes the old one. It fails if the name must be unique.
*   **Refactoring**: "How do you move a resource without recreating it?"
    *   Add a `moved` block (`from` and `to`). `terraform state mv` does the same by hand.
    *   Related: `import` blocks adopt existing resources. `removed` blocks stop managing one without deleting it.

### 7. Common Mistakes
*   **Secrets in state**: State stores values in plain text, including generated passwords. Never commit it. `sensitive = true` hides a value from output, not from state.
*   **One giant state**: Plans get slow and one mistake can hit everything. Split state by environment and component.
*   **Unpinned versions**: Pin provider versions and commit `.terraform.lock.hcl`, or a new major version can change behavior.
*   **Not reading the plan**: Missing a "must be replaced" on a database.

### Example
Pinned provider, remote state with locking, and a web server whose AMI is looked up instead of hardcoded (AMI IDs differ by region and go stale).

```terraform
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }

  backend "s3" {
    bucket       = "example-terraform-state"
    key          = "web/terraform.tfstate"
    region       = "us-east-1"
    encrypt      = true
    use_lockfile = true
  }
}

provider "aws" {
  region = "us-east-1"
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"]
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  tags          = { Name = "example-server" }
}

output "instance_public_ip" {
  value = aws_instance.web.public_ip
}

resource "aws_s3_bucket" "assets" {
  bucket = "example-unique-bucket-name" # bucket names are global across AWS
}

output "bucket_domain_name" {
  value = aws_s3_bucket.assets.bucket_domain_name
}
```
