# Terraform - Introduction & Basics (Interview Ready Notes)

---

## 1. What is Terraform?

**One-liner:** Open-source IaC tool by HashiCorp to create, change, and manage infrastructure using declarative config files (HCL).

```
You write CODE → Terraform creates INFRASTRUCTURE
```

**3 Core Components:**
- **Providers** → Connect to cloud APIs (AWS, Azure, GCP)
- **Resources** → Actual infra objects (EC2, VPC, RDS)
- **State** → Tracks what's currently deployed

---

## 2. Why IaC?

| Benefit | Explanation |
|---------|-------------|
| Consistency | Same infra every time, no manual drift |
| Automation | No human errors, repeatable |
| Version Control | Git tracks infra changes like code |
| Collaboration | Team can review PRs for infra |
| Documentation | Code IS the documentation |

---

## 3. Terraform vs Others (Interview Comparison)

```
┌─────────────────┬────────────┬────────────────┬──────────┬─────────┐
│ Feature         │ Terraform  │ CloudFormation │ Pulumi   │ Ansible │
├─────────────────┼────────────┼────────────────┼──────────┼─────────┤
│ Language        │ HCL        │ JSON/YAML      │ JS/Py/Go │ YAML    │
│ Multi-Cloud     │ ✅ Yes     │ ❌ AWS only    │ ✅ Yes   │ ✅ Yes  │
│ State Mgmt      │ Built-in   │ AWS Managed    │ Built-in │ Limited │
│ Style           │ Declarative│ Declarative    │Imperative│ Mixed   │
│ Best For        │ Multi-cloud│ AWS-only shops │ Devs     │ Config  │
└─────────────────┴────────────┴────────────────┴──────────┴─────────┘
```

**🎯 Interview Point:** "Terraform is cloud-agnostic and declarative. CloudFormation locks you into AWS. Pulumi uses real languages but has a steeper learning curve for ops teams. Ansible is better for config management, not provisioning."

---

## 4. Terraform Workflow

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│   init   │───▶│   plan   │───▶│  apply   │───▶│ destroy  │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
 Download         Preview         Execute         Tear down
 providers        changes         changes         infra
```

| Command | What it does |
|---------|-------------|
| `terraform init` | Downloads providers/modules |
| `terraform plan` | Dry run — shows what WILL change |
| `terraform apply` | Actually creates/updates infra |
| `terraform destroy` | Deletes everything Terraform manages |
| `terraform validate` | Checks syntax errors |
| `terraform fmt` | Auto-formats HCL files |

---

## 5. Providers

**What:** Plugins that let Terraform talk to APIs.

```hcl
provider "aws" {
  region = "us-east-1"
}
```

**🎯 Interview Point:** Providers are downloaded during `terraform init`. You can pin versions. Multiple providers can coexist in one config (e.g., AWS + Kubernetes + Datadog).

---

## 6. Resources

**What:** The actual infrastructure objects you create.

```hcl
resource "<type>" "<local_name>" {
  property = "value"
}
```

**Example:**
```hcl
resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = { Name = "MyWebServer" }
}
```

---

## 7. Data Sources

**What:** Read-only. Fetches info about EXISTING resources (not managed by your Terraform).

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-ebs"]
  }
  owners = ["amazon"]
}

# Use it:
resource "aws_instance" "web" {
  ami = data.aws_ami.amazon_linux.id  # Dynamic lookup!
}
```

**🎯 Interview Point:** Data sources are for READING, resources are for CREATING. Data sources help avoid hardcoding values (like AMI IDs) that change over time.

---

## 8. Variables & Outputs

### Input Variables
```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

# Usage:
instance_type = var.instance_type
```

**Ways to pass variables (priority order):**
1. CLI: `-var="instance_type=t3.medium"`
2. `.tfvars` file
3. Environment variable: `TF_VAR_instance_type`
4. Default value in variable block

### Output Variables
```hcl
output "public_ip" {
  value       = aws_instance.web_server.public_ip
  description = "The public IP of the web server"
}
```

**🎯 Interview Point:** Outputs are used to pass data between modules and to display results after apply.

---

## 9. Terraform State

**What:** A JSON file (`terraform.tfstate`) that maps your config to real-world resources.

```
┌──────────────────┐         ┌──────────────────┐
│  Your HCL Code   │◀───────▶│  terraform.tfstate│
│  (desired state) │         │  (actual state)   │
└──────────────────┘         └──────────────────┘
         │                            │
         └──────────┬─────────────────┘
                    ▼
         ┌──────────────────┐
         │  Real Cloud Infra │
         │  (AWS/Azure/GCP)  │
         └──────────────────┘
```

### Local vs Remote State

| Aspect | Local | Remote (S3 + DynamoDB) |
|--------|-------|------------------------|
| Storage | Local disk | S3 bucket |
| Team use | ❌ Single user | ✅ Multi-user |
| Locking | ❌ No | ✅ DynamoDB lock |
| Security | ❌ Risk of exposure | ✅ Encrypted, versioned |

### Remote State Config (AWS — Most Common in Interviews):
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-lock-table"  # Prevents concurrent edits
    encrypt        = true
  }
}
```

**🎯 Interview Point:** "Never store state locally in production. Use S3 + DynamoDB for locking. State contains sensitive data (passwords, IPs) — always encrypt it."

---

## 10. Resource Lifecycle Meta-Arguments

```hcl
lifecycle {
  create_before_destroy = true   # Zero-downtime replacements
  prevent_destroy       = true   # Protect critical resources (DB)
  ignore_changes        = [tags] # Don't trigger update for tag changes
}
```

**🎯 Interview Point:**
- `create_before_destroy` → Used for blue-green style replacements
- `prevent_destroy` → Safety net for databases/critical infra
- `ignore_changes` → When external tools modify tags/metadata

---

## 11. Recommended Project Structure

```
terraform-project/
├── modules/              # Reusable components
│   └── vpc/
│   └── ec2/
├── environments/         # Per-env configs
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── prod/
└── README.md
```

---

## 🔥 Top Interview Questions from This Section

1. **What is Terraform and how is it different from Ansible?**
   → Terraform = provisioning (create infra). Ansible = configuration (configure existing infra).

2. **What is Terraform state and why is it important?**
   → Maps config to real resources. Without it, Terraform can't know what exists.

3. **How do you handle state in a team?**
   → Remote backend (S3 + DynamoDB locking). Never commit state to git.

4. **What's the difference between a resource and a data source?**
   → Resource = Terraform manages it. Data source = Read-only lookup of existing infra.

5. **Explain the Terraform workflow.**
   → init → plan → apply → destroy. Plan is a dry run, apply executes.

6. **How do you prevent accidental deletion of critical resources?**
   → `lifecycle { prevent_destroy = true }`

7. **Why is Terraform cloud-agnostic?**
   → Uses providers as plugins. Same workflow regardless of cloud.

---
# Terraform - HCL Syntax & State Management (Interview Ready Notes)

---

## 1. HCL Syntax — The Basics

**One-liner:** HCL (HashiCorp Configuration Language) is Terraform's declarative language. Files end in `.tf`.

### Block Structure:
```hcl
block_type "label1" "label2" {
  attribute = "value"

  nested_block {
    attribute = "value"
  }
}
```

**Example:**
```hcl
resource "aws_instance" "web" {    # block_type "type" "name"
  ami           = "ami-0c55b159"   # attribute = value
  instance_type = "t3.medium"
}
```

---

## 2. Data Types

```
┌────────────┬──────────────────────────────────┐
│ Type       │ Example                          │
├────────────┼──────────────────────────────────┤
│ string     │ "hello"                          │
│ number     │ 42                               │
│ bool       │ true / false                     │
│ list       │ ["us-east-1a", "us-east-1b"]     │
│ map        │ { Env = "prod", Owner = "devops"}│
│ set        │ toset(["a", "b", "c"])           │
│ object     │ { name = "db", type = "t3.med" } │
└────────────┴──────────────────────────────────┘
```

**🎯 Interview Point:** List = ordered, duplicates allowed. Set = unordered, no duplicates. Map = key-value pairs.

---

## 3. Variables vs Locals

```
┌─────────────────┬──────────────────────────────────────────┐
│ Variables (var.) │ Locals (local.)                          │
├─────────────────┼──────────────────────────────────────────┤
│ External input   │ Internal computed values                 │
│ Passed by user   │ Defined inside config                   │
│ Reusable across  │ Simplify repeated expressions           │
│ modules          │ Not exposed outside                     │
└─────────────────┴──────────────────────────────────────────┘
```

### Variable:
```hcl
variable "region" {
  type        = string
  default     = "us-east-1"
}
# Usage: var.region
```

### Local:
```hcl
locals {
  common_tags = {
    Environment = "prod"
    ManagedBy   = "Terraform"
  }
}
# Usage: local.common_tags
```

**🎯 Interview Point:** Use `locals` when you need to compute/combine values or avoid repeating logic. Use `variables` for values that change per environment or module call.

---

## 4. Conditionals

```hcl
# Syntax: condition ? true_value : false_value

instance_type = var.is_production ? "t3.large" : "t3.micro"
```

**Real-world use:** Toggle features, pick AMIs, set instance sizes by environment.

---

## 5. Built-in Functions (Most Asked in Interviews)

| Category | Function | Example | Result |
|----------|----------|---------|--------|
| String | `upper()` | `upper("hello")` | `"HELLO"` |
| String | `format()` | `format("Hi %s", "TF")` | `"Hi TF"` |
| Collection | `length()` | `length(["a","b"])` | `2` |
| Collection | `merge()` | `merge({a=1},{b=2})` | `{a=1,b=2}` |
| Collection | `contains()` | `contains(["a","b"],"b")` | `true` |
| Numeric | `min/max()` | `min(10,5,3)` | `3` |
| Encoding | `jsonencode()` | `jsonencode({k="v"})` | `{"k":"v"}` |

**🎯 Interview Point:** Know `lookup()`, `merge()`, `concat()`, `flatten()`, `templatefile()` — these come up in real projects constantly.

---

## 6. Formatting & Comments

```bash
terraform fmt          # Auto-format all .tf files
terraform fmt -check   # CI check — fails if not formatted
```

```hcl
# Single-line comment

/* Multi-line
   comment */
```

**Best Practice:** Run `terraform fmt` in CI pipeline pre-commit hook.

---

## 7. HCL Best Practices

| Practice | Why |
|----------|-----|
| Don't hardcode | Use variables/locals/data sources |
| Use locals for repetition | DRY principle |
| Use functions | Cleaner logic |
| Run `terraform fmt` | Consistent style |
| Comment decisions | Why, not what |

---

---

# Terraform State Management (Deep Dive)

---

## 8. What is State?

```
┌─────────────────┐
│  main.tf (Code) │ ← Desired State (what you WANT)
└────────┬────────┘
         │ terraform plan (compares)
         ▼
┌─────────────────────┐
│ terraform.tfstate    │ ← Known State (what Terraform KNOWS)
└────────┬────────────┘
         │ terraform apply (syncs)
         ▼
┌─────────────────────┐
│ Real Cloud Resources │ ← Actual State (what EXISTS)
└─────────────────────┘
```

**State file:** JSON file mapping your HCL resources → real-world resource IDs.

**⚠️ Golden Rule:** NEVER edit state manually. Use CLI commands only.

---

## 9. Local vs Remote State

```
┌──────────────┬──────────────────────┬────────────────────────────┐
│              │ Local State          │ Remote State               │
├──────────────┼──────────────────────┼────────────────────────────┤
│ Storage      │ Local file           │ S3 / Azure Blob / TF Cloud│
│ Team Use     │ ❌ Single user       │ ✅ Multi-user              │
│ Locking      │ ❌ No                │ ✅ DynamoDB / built-in     │
│ Encryption   │ ❌ Plaintext on disk │ ✅ Server-side encryption  │
│ Backup       │ ❌ Manual            │ ✅ Versioning enabled      │
│ Use Case     │ Learning / testing   │ Production / teams         │
└──────────────┴──────────────────────┴────────────────────────────┘
```

---

## 10. Remote Backend Setup (S3 + DynamoDB)

### Step 1: Create infra for state
```bash
aws s3 mb s3://my-terraform-state-bucket
aws dynamodb create-table \
  --table-name terraform-lock-table \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

### Step 2: Configure backend
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-lock-table"
    encrypt        = true
  }
}
```

### Step 3: Initialize
```bash
terraform init   # Migrates local state to S3
```

**🎯 Interview Point:**
- `bucket` = where state lives
- `key` = path inside bucket (use env prefix: `prod/`, `dev/`)
- `dynamodb_table` = prevents two people running `apply` simultaneously
- `encrypt` = state contains secrets (DB passwords, IPs)

---

## 11. State Locking

```
Developer A: terraform apply ──┐
                               │ DynamoDB Lock ──▶ ✅ Acquired
Developer B: terraform apply ──┘                 ──▶ ❌ BLOCKED
                                                    "Error acquiring state lock"
```

**Force unlock (emergency only):**
```bash
terraform force-unlock LOCK_ID
```

**🎯 Interview Point:** Locking prevents state corruption from concurrent operations. DynamoDB uses `LockID` as partition key.

---

## 12. Importing Existing Resources

**Scenario:** You have manually-created infra, want Terraform to manage it.

```
Step 1: Write the resource block in .tf file
Step 2: Run import command
Step 3: Run plan to align config with reality
```

```bash
terraform import aws_instance.existing_web i-0abcd1234ef5678gh
```

**🎯 Interview Point:** Import only updates STATE, not your .tf code. You must write the resource block yourself and run `plan` to fill in missing attributes.

---

## 13. State Management Commands

| Command | Purpose |
|---------|---------|
| `terraform state list` | List all resources in state |
| `terraform state show <resource>` | Show details of one resource |
| `terraform state mv old new` | Rename/move resource in state |
| `terraform state rm <resource>` | Remove from state (doesn't destroy) |
| `terraform state pull` | Download remote state to stdout |
| `terraform state push` | Upload state to remote backend |
| `terraform apply -refresh-only` | Sync state with real infra (replaces deprecated `refresh`) |

**🎯 Interview Point:** `state rm` is used when you want to "un-manage" a resource without destroying it. Common during refactoring or module migration.

---

## 14. State Recovery & Corruption

```
┌─────────────────────────────────────────────┐
│ Prevention:                                  │
│ • Enable S3 versioning                       │
│ • Use DynamoDB locking                       │
│ • Never edit state manually                  │
│                                              │
│ Recovery:                                    │
│ • terraform state pull > backup.tfstate      │
│ • terraform state push backup.tfstate        │
│ • Restore from S3 version history            │
└─────────────────────────────────────────────┘
```

---

## 🔥 Top Interview Questions from This Section

1. **What is HCL?**
   → HashiCorp Configuration Language. Declarative, human-readable. Files = `.tf`.

2. **Difference between variables and locals?**
   → Variables = external inputs (passed in). Locals = internal computed values (not exposed).

3. **How do you handle multiple environments with state?**
   → Separate state files per env using different `key` paths: `dev/terraform.tfstate`, `prod/terraform.tfstate`.

4. **What happens if two people run `terraform apply` simultaneously?**
   → With locking (DynamoDB), second person gets blocked. Without locking, state corruption.

5. **How do you import existing resources?**
   → Write resource block → `terraform import <resource> <id>` → run `plan` to align.

6. **How do you remove a resource from state without destroying it?**
   → `terraform state rm <resource_address>`

7. **What's in the state file? Is it sensitive?**
   → YES. Contains resource IDs, IPs, passwords, connection strings. Always encrypt + restrict access.

8. **How do you recover corrupted state?**
   → S3 versioning to restore previous version, or `terraform state push` from a backup.

---
# Terraform - Modules, Workspaces & CLI Commands (Interview Ready Notes)

---

## 1. Terraform Modules

**One-liner:** A module is a reusable, self-contained package of `.tf` files that encapsulates a logical infrastructure component.

### Module Structure:
```
modules/ec2/
├── main.tf          # Resources
├── variables.tf     # Inputs
├── outputs.tf       # Outputs
├── versions.tf      # Provider version constraints
└── README.md        # Documentation
```

### How Modules Work (Diagram):
```
┌─────────────────────────────┐
│  Root Module (your main.tf) │
│                             │
│  module "vpc" {             │
│    source = "./modules/vpc" │──────▶ ┌──────────────────┐
│    cidr   = "10.0.0.0/16"  │        │ modules/vpc/     │
│  }                          │        │  main.tf         │
│                             │        │  variables.tf    │
│  module "ec2" {             │        │  outputs.tf      │
│    source = "./modules/ec2" │──────▶ └──────────────────┘
│    ami    = "ami-123"       │
│  }                          │
└─────────────────────────────┘
```

---

### Module Sources:

| Source | Syntax |
|--------|--------|
| Local path | `source = "../modules/ec2"` |
| Terraform Registry | `source = "terraform-aws-modules/vpc/aws"` |
| Git repo | `source = "git::https://github.com/org/repo.git?ref=v1.2.0"` |
| S3 bucket | `source = "s3::https://bucket.s3.amazonaws.com/module.zip"` |

### Example — Using a Registry Module:
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"              # Always pin version!

  name = "my-vpc"
  cidr = "10.0.0.0/16"
}
```

### Example — Creating Your Own Module:

**modules/ec2/main.tf:**
```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
  tags          = var.tags
}
```

**modules/ec2/variables.tf:**
```hcl
variable "ami_id"        { type = string }
variable "instance_type" { type = string; default = "t3.micro" }
variable "tags"          { type = map(string); default = {} }
```

**modules/ec2/outputs.tf:**
```hcl
output "instance_id" { value = aws_instance.web.id }
output "public_ip"   { value = aws_instance.web.public_ip }
```

**Calling it:**
```hcl
module "my_server" {
  source        = "./modules/ec2"
  ami_id        = "ami-12345678"
  instance_type = "t3.medium"
  tags          = { Environment = "dev" }
}
```

---

### Module Best Practices (🎯 Interview Points):

| Practice | Why |
|----------|-----|
| **Always version modules** | `version = "~> 5.0"` prevents breaking changes |
| **Small, focused modules** | One module = one concern (VPC, EC2, RDS) |
| **Sensible defaults** | Make it easy to use, hard to misuse |
| **Document inputs/outputs** | README + description in variables |
| **Test with Terratest** | Automated Go tests for reliability |
| **Semantic versioning** | v1.0.0 → v1.1.0 (minor) → v2.0.0 (breaking) |

---

### Testing Modules (Terratest):
```go
func TestTerraformModule(t *testing.T) {
  opts := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
    TerraformDir: "../",
  })
  defer terraform.Destroy(t, opts)
  terraform.InitAndApply(t, opts)

  instanceID := terraform.Output(t, opts, "instance_id")
  assert.NotEmpty(t, instanceID)
}
```

**🎯 Interview Point:** Terratest is the industry standard for testing Terraform modules. It deploys real infra, validates, then destroys. Alternative: `terraform test` (native, newer).

---

## 2. Terraform Workspaces

**One-liner:** Workspaces = isolated state files within the same config, used to manage multiple environments.

### How Workspaces Work:
```
┌─────────────────────────────────────────────┐
│            Same .tf Configuration            │
├──────────┬──────────────┬───────────────────┤
│  dev     │  staging     │  prod             │
│  state   │  state       │  state            │
└──────────┴──────────────┴───────────────────┘

State path: terraform.tfstate.d/<workspace>/terraform.tfstate
```

### Commands:
```bash
terraform workspace new dev        # Create + switch
terraform workspace select prod    # Switch
terraform workspace list           # List all
terraform workspace show           # Current workspace
terraform workspace delete staging # Delete
```

### Using Workspace in Config:
```hcl
variable "instance_sizes" {
  default = {
    dev     = "t2.micro"
    staging = "t3.small"
    prod    = "t3.large"
  }
}

resource "aws_instance" "app" {
  ami           = var.ami
  instance_type = var.instance_sizes[terraform.workspace]  # Dynamic!

  tags = {
    Environment = terraform.workspace
    Name        = "${terraform.workspace}-app-server"
  }
}
```

---

### Workspaces vs Separate Directories:

```
┌──────────────────────┬────────────────────────────────────┐
│ Workspaces           │ Separate Directories               │
├──────────────────────┼────────────────────────────────────┤
│ Same code, diff state│ Different code per environment     │
│ Simple, less files   │ More flexibility, more duplication │
│ Good for small/medium│ Good for large/complex setups      │
│ Risk: wrong workspace│ Risk: code drift between envs      │
└──────────────────────┴────────────────────────────────────┘
```

**🎯 Interview Point:** "For 12 YOE level — I prefer separate directories (or Terragrunt) for production because:
1. Environments often diverge (prod has DR, dev doesn't)
2. Blast radius is smaller (can't accidentally apply to wrong env)
3. Workspaces are fine for simple setups but don't scale well for complex orgs"

---

### Workspace Best Practices:

- ✅ Use for small-medium projects with similar infra across envs
- ✅ Always check `terraform workspace show` before `apply`
- ❌ Don't overload with complex conditionals
- ❌ Don't use for vastly different environments

---

## 3. Terraform CLI Commands Cheat Sheet

### Init & Setup:
| Command | Purpose |
|---------|---------|
| `terraform init` | Download providers/modules |
| `terraform init -upgrade` | Upgrade providers to latest |
| `terraform version` | Show installed version |
| `terraform providers` | List providers in use |

### Plan & Apply:
| Command | Purpose |
|---------|---------|
| `terraform plan` | Preview changes (dry run) |
| `terraform plan -out=plan.tfplan` | Save plan to file |
| `terraform apply` | Apply changes |
| `terraform apply plan.tfplan` | Apply saved plan (no re-prompt) |
| `terraform apply -auto-approve` | Skip confirmation (CI/CD) |
| `terraform destroy` | Delete all managed resources |
| `terraform apply -refresh-only` | Sync state with real infra |

### State:
| Command | Purpose |
|---------|---------|
| `terraform state list` | List all resources |
| `terraform state show <res>` | Details of one resource |
| `terraform state mv old new` | Rename in state |
| `terraform state rm <res>` | Remove from state (keep real) |
| `terraform import <res> <id>` | Import existing resource |
| `terraform force-unlock <ID>` | Break a stuck lock |

### Validation & Format:
| Command | Purpose |
|---------|---------|
| `terraform validate` | Check syntax/config errors |
| `terraform fmt` | Auto-format files |
| `terraform fmt -check` | CI check (fails if unformatted) |

### Output & Debug:
| Command | Purpose |
|---------|---------|
| `terraform output` | Show all outputs |
| `terraform output <name>` | Show specific output |
| `TF_LOG=DEBUG terraform plan` | Verbose debug logging |
| `terraform console` | Interactive expression testing |

---

## 4. State Best Practices (Quick Checklist)

```
✅ Remote backend (S3 + DynamoDB)
✅ Encrypt state (encrypt = true)
✅ Enable S3 versioning (recovery)
✅ State locking (DynamoDB)
✅ Never edit state manually
✅ Never commit state to git
✅ Run refresh-only periodically
✅ Restrict IAM access to state bucket
```

---

## 🔥 Top Interview Questions from This Section

1. **What is a Terraform module and why use it?**
   → Reusable, self-contained config package. DRY principle, encapsulation, team sharing.

2. **How do you version modules?**
   → Git tags (v1.0.0). Pin in consumer: `version = "~> 5.0"`. Follow semver.

3. **Local module vs Registry module?**
   → Local = your own code, `source = "./path"`. Registry = community/official, `source = "org/module/provider"`.

4. **How do you test Terraform modules?**
   → Terratest (Go). Deploy real infra → validate → destroy. Also `terraform validate` for syntax.

5. **Explain workspaces. When would you NOT use them?**
   → Isolated state per env. Don't use when environments are vastly different or when blast radius matters (use separate dirs instead).

6. **What is `terraform.workspace` variable?**
   → Built-in variable returning current workspace name. Used for dynamic config (instance sizes, tags, names).

7. **How do you handle a locked state?**
   → Wait for current op to finish. If stuck: `terraform force-unlock <LOCK_ID>`. Investigate why it was locked.

8. **What's the difference between `terraform plan -out` and just `terraform apply`?**
   → `-out` saves the exact plan. `apply plan.tfplan` executes THAT plan without re-evaluating. Critical for CI/CD — ensures what was reviewed is what gets deployed.

---
# Terraform - Provisioners, Cloud Providers, Advanced Concepts & CI/CD (Interview Ready Notes)

---

## 1. Terraform Provisioners

**One-liner:** Provisioners run scripts/commands on resources AFTER creation (or destruction). Use sparingly — they're a last resort.

### Three Types:

```
┌──────────────┬────────────────────────────┬──────────────────────────┐
│ Type         │ What it does               │ Use case                 │
├──────────────┼────────────────────────────┼──────────────────────────┤
│ remote-exec  │ Run commands on REMOTE box │ Install packages via SSH │
│ local-exec   │ Run commands LOCALLY       │ Trigger notifications    │
│ file         │ Copy files to remote box   │ Upload configs/scripts   │
└──────────────┴────────────────────────────┴──────────────────────────┘
```

### Example — Full Flow:
```hcl
resource "aws_instance" "app" {
  ami           = "ami-0abcdef123"
  instance_type = "t2.micro"
  key_name      = "my_key"

  # 1. Upload script
  provisioner "file" {
    source      = "setup.sh"
    destination = "/tmp/setup.sh"
    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/my_key.pem")
      host        = self.public_ip
    }
  }

  # 2. Execute remotely
  provisioner "remote-exec" {
    inline = ["chmod +x /tmp/setup.sh", "sudo /tmp/setup.sh"]
    connection { /* same as above */ }
  }

  # 3. Log locally
  provisioner "local-exec" {
    command = "echo ${self.public_ip} >> deploy.log"
  }
}
```

### Run on Destroy:
```hcl
provisioner "local-exec" {
  when    = destroy
  command = "echo 'Resource destroyed!' >> cleanup.log"
}
```

### 🎯 Interview Point — Why NOT to Use Provisioners:

```
┌────────────────────────────────────────────────────────────────┐
│ Problem with Provisioners:                                      │
│ • NOT tracked in state (no drift detection)                     │
│ • NOT idempotent by default                                     │
│ • Fail = entire resource tainted                                │
│ • Can't re-run without recreating resource                      │
│                                                                  │
│ Better Alternatives:                                             │
│ • cloud-init / user_data  → Bootstrap at launch                 │
│ • Packer                  → Pre-bake AMIs                       │
│ • Ansible/Chef/Puppet     → Configuration management            │
│ • AWS SSM Run Command     → Post-deploy without SSH             │
└────────────────────────────────────────────────────────────────┘
```

**Senior answer:** "I avoid provisioners in production. I use Packer to bake AMIs and cloud-init for bootstrap. If I need post-deploy config, I use Ansible triggered by local-exec or a separate pipeline step."

---

## 2. Terraform with Cloud Providers (Quick Reference)

### Provider Setup Pattern:
```
┌──────────┬──────────────────────────────────────────────────┐
│ Provider │ Auth Method                                       │
├──────────┼──────────────────────────────────────────────────┤
│ AWS      │ AWS_ACCESS_KEY_ID + AWS_SECRET_ACCESS_KEY (env)  │
│ Azure    │ ARM_CLIENT_ID + ARM_CLIENT_SECRET (env)          │
│ GCP      │ GOOGLE_CREDENTIALS=path/to/json (env)           │
│ DO       │ DIGITALOCEAN_TOKEN (env)                         │
└──────────┴──────────────────────────────────────────────────┘
```

### Minimal Examples:

**AWS EC2:**
```hcl
provider "aws" { region = "us-east-1" }

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  tags          = { Name = "web-server" }
}
```

**GCP Compute:**
```hcl
provider "google" { project = "my-project"; region = "us-central1" }

resource "google_compute_instance" "vm" {
  name         = "terraform-vm"
  machine_type = "f1-micro"
  zone         = "us-central1-a"
  boot_disk { initialize_params { image = "debian-cloud/debian-11" } }
  network_interface { network = "default"; access_config {} }
}
```

**🎯 Interview Point:** Always use environment variables or IAM roles for credentials. NEVER hardcode secrets in `.tf` files. For AWS in CI/CD, prefer OIDC federation over static keys.

---

## 3. Advanced Terraform Concepts

### Dynamic Blocks + for_each

**Problem:** Repeating nested blocks (security group rules, tags, etc.)
**Solution:** `dynamic` block

```hcl
variable "ingress_rules" {
  default = [
    { port = 80,  cidr = "0.0.0.0/0" },
    { port = 443, cidr = "0.0.0.0/0" },
  ]
}

resource "aws_security_group" "web" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = [ingress.value.cidr]
    }
  }
}
```

**🎯 Interview Point:** `for_each` > `count` when resources have unique identifiers. `count` uses index (fragile — removing item 2 recreates 3,4,5...). `for_each` uses keys (stable).

---

### Terraform Cloud & Enterprise

```
┌────────────────────────────────────────────────────────┐
│                  Terraform Cloud                        │
├────────────────────────────────────────────────────────┤
│ • Remote state storage (encrypted, versioned)          │
│ • Remote execution (plan/apply in cloud)               │
│ • Team collaboration (RBAC)                            │
│ • VCS integration (auto-plan on PR)                    │
│ • Sentinel policies (governance)                       │
│ • Private module registry                              │
│ • Cost estimation                                      │
└────────────────────────────────────────────────────────┘
```

---

### Sentinel — Policy as Code

**What:** Enforce governance rules BEFORE apply.

```python
# Block public S3 buckets
import "tfplan/v2"

public_buckets = filter tfplan.resource_changes as rc {
  rc.type is "aws_s3_bucket" and
  rc.change.after.acl is "public-read"
}

main = rule { length(public_buckets) is 0 }
```

**Real-world policies:**
- All resources must have tags
- No public ingress on port 22
- Only approved instance types
- Only specific regions allowed

**🎯 Interview Point:** Sentinel = enterprise governance. Open-source alternative = **OPA (Open Policy Agent)** with `conftest`.

---

### CDKTF (CDK for Terraform)

```
Traditional:  HCL → terraform plan → apply
CDKTF:        TypeScript/Python → cdktf synth → terraform JSON → apply
```

**When to use:** Teams that prefer imperative languages, complex logic, or want to share libraries via npm/pip.

**When NOT to use:** Simple infra, ops-heavy teams comfortable with HCL.

---

### Custom Providers

- Written in **Go** using Terraform Plugin SDK
- For internal APIs with no existing provider
- Published to private or public registry

---

## 4. Terraform CI/CD Integration

### Why CI/CD + Terraform?
```
┌─────────────────────────────────────────────────────┐
│  Developer pushes code                               │
│       ↓                                              │
│  CI: fmt → validate → plan (on PR)                  │
│       ↓                                              │
│  Review: Team reviews plan output in PR              │
│       ↓                                              │
│  CD: apply (on merge to main, with approval gate)   │
└─────────────────────────────────────────────────────┘
```

### GitHub Actions Example:
```yaml
name: Terraform CI
on:
  push: { branches: [main] }
  pull_request:

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: hashicorp/setup-terraform@v3
        with: { terraform_version: "1.6.0" }

      - run: terraform fmt -check
      - run: terraform init
      - run: terraform validate
      - run: terraform plan -out=plan.tfplan

      - if: github.ref == 'refs/heads/main'
        run: terraform apply -auto-approve plan.tfplan
```

### GitLab CI Example:
```yaml
stages: [validate, plan, apply]

plan:
  stage: plan
  script:
    - terraform init
    - terraform plan -out=tfplan
  artifacts:
    paths: [tfplan]

apply:
  stage: apply
  script:
    - terraform apply -auto-approve tfplan
  when: manual          # ← Manual approval gate!
  only: [main]
```

### Jenkins Pipeline:
```groovy
pipeline {
  environment {
    AWS_ACCESS_KEY_ID     = credentials('aws-access-key')
    AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
  }
  stages {
    stage('Plan')  { steps { sh 'terraform plan -out=tfplan' } }
    stage('Apply') { when { branch 'main' }
                     steps { sh 'terraform apply -auto-approve tfplan' } }
  }
}
```

---

### CI/CD Best Practices:

| Practice | Why |
|----------|-----|
| Separate plan & apply stages | Review before deploy |
| Save plan as artifact | Ensures reviewed plan = applied plan |
| Manual approval for prod | Prevent accidental deployments |
| Use OIDC (not static keys) | Short-lived creds, more secure |
| `fmt -check` in CI | Enforce formatting standards |
| Use remote state | Shared state across CI runners |
| Pin Terraform version | Reproducible builds |

**🎯 Interview Point:** "In production, I use: PR → auto-plan → comment plan on PR → manual approve → apply on merge. I never use `-auto-approve` for prod without a gate."

---

## 5. CLI Debugging & Environment Variables

```bash
# Debug levels: TRACE > DEBUG > INFO > WARN > ERROR
export TF_LOG=DEBUG
export TF_LOG_PATH=terraform.log
terraform plan

# Useful CLI flags
terraform apply -auto-approve          # Skip prompt (CI only!)
terraform apply -target=aws_instance.x # Apply single resource
terraform apply -var-file=prod.tfvars  # Load env-specific vars
terraform plan -out=plan.tfplan        # Save plan for later
```

---

## 🔥 Top Interview Questions from This Section

1. **When would you use provisioners vs alternatives?**
   → Almost never in production. Use Packer (AMI baking), cloud-init (bootstrap), or Ansible (config mgmt). Provisioners = last resort, not tracked in state.

2. **Explain `for_each` vs `count`.**
   → `count` = index-based (fragile). `for_each` = key-based (stable). Removing middle item in count shifts all subsequent indexes → recreation. for_each doesn't.

3. **What is Sentinel?**
   → Policy-as-code for Terraform Enterprise/Cloud. Enforces rules before apply (e.g., no public S3, mandatory tags). Open-source alternative = OPA.

4. **How do you integrate Terraform into CI/CD?**
   → PR triggers plan → plan output posted as PR comment → manual approval → merge triggers apply. Always save plan as artifact.

5. **How do you manage secrets in Terraform CI/CD?**
   → GitHub Secrets / GitLab CI Variables / Jenkins Credentials. Never in code. Prefer OIDC federation for cloud auth. Use Vault for runtime secrets.

6. **What is CDKTF?**
   → Write Terraform using TypeScript/Python/Go instead of HCL. Compiles to Terraform JSON. Good for dev teams, complex logic.

7. **Dynamic blocks — when and why?**
   → When you need to generate repeated nested blocks (security group rules, tags) from a variable list. Keeps code DRY.

8. **What's the difference between Terraform Cloud and self-managed?**
   → Cloud = managed remote state, remote runs, Sentinel, RBAC, VCS integration. Self-managed = you handle state backend, CI/CD, and policies yourself.

---
# Terraform - Security, Troubleshooting & Quick Reference (Interview Ready Notes)

---

## 1. Terraform Security Best Practices

### Security Layers (Diagram):

```
┌─────────────────────────────────────────────────────────────┐
│                    TERRAFORM SECURITY                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────┐   ┌──────────┐   ┌────────────┐   ┌────────┐ │
│  │  STATE  │   │ SECRETS  │   │    IAM     │   │ POLICY │ │
│  │         │   │          │   │            │   │        │ │
│  │Encrypted│   │Vault/SM  │   │Least Priv  │   │Sentinel│ │
│  │Remote   │   │Env vars  │   │Temp creds  │   │OPA     │ │
│  │Versioned│   │sensitive │   │OIDC        │   │tfsec   │ │
│  └─────────┘   └──────────┘   └────────────┘   └────────┘ │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

### A. Secure State Files

| Do | Don't |
|----|-------|
| Remote backend (S3/TF Cloud) | Store locally in prod |
| Enable encryption (SSE-KMS) | Commit to Git |
| Enable versioning (rollback) | Share state file manually |
| Restrict IAM access to bucket | Give broad S3 permissions |
| Enable DynamoDB locking | Run without locking |

```hcl
# .gitignore — ALWAYS include:
*.tfstate
*.tfstate.backup
*.tfvars        # If contains secrets
.terraform/
```

---

### B. Secrets Management

```
❌ BAD:
  variable "db_pass" { default = "MyP@ssw0rd!" }

✅ GOOD:
  - Environment variable:  export TF_VAR_db_pass="..."
  - AWS Secrets Manager:   data "aws_secretsmanager_secret_version" ...
  - HashiCorp Vault:       data "vault_generic_secret" ...
  - CI/CD secrets:         GitHub Secrets / GitLab CI Variables
```

**Mark sensitive outputs:**
```hcl
output "db_password" {
  value     = var.db_password
  sensitive = true              # Hidden in plan/apply output
}
```

---

### C. Least-Privilege IAM

| Cloud | Best Practice |
|-------|---------------|
| AWS | Separate IAM role for Terraform. Use STS AssumeRole. Prefer OIDC in CI/CD |
| Azure | Service Principal with Contributor (not Owner) |
| GCP | Workload Identity Federation. Scoped Service Account |

**🎯 Interview Point:** "I create a dedicated Terraform IAM role per environment with only the permissions needed for that env's resources. I use OIDC federation from GitHub Actions — no static keys."

---

### D. Security Scanning Tools

```
┌──────────┬──────────────────────────────────┬─────────────────┐
│ Tool     │ Purpose                          │ Command         │
├──────────┼──────────────────────────────────┼─────────────────┤
│ tfsec    │ Static security analysis         │ tfsec .         │
│ checkov  │ IaC security + compliance        │ checkov -d .    │
│ terrascan│ Policy-as-code scanner           │ terrascan scan  │
│ TFLint   │ Linting + best practices         │ tflint          │
│ Sentinel │ Enterprise policy enforcement    │ (TF Cloud)      │
│ OPA      │ Open-source policy engine        │ conftest test   │
└──────────┴──────────────────────────────────┴─────────────────┘
```

**CI/CD Integration:**
```yaml
# Run in PR pipeline:
- run: tfsec .
- run: checkov -d . --framework terraform
- run: tflint
```

---

### E. Module Security

```hcl
# Always pin versions:
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.2.0"    # Exact pin, not "~> 5.0" for prod
}
```

**Audit third-party modules for:**
- Public exposure (0.0.0.0/0 in SGs)
- Hardcoded secrets
- Overly permissive IAM policies
- Unencrypted resources

---

### F. Security Cheat Sheet

| Concern | Solution |
|---------|----------|
| Secrets in code | Env vars / Vault / Secrets Manager |
| State exposure | Encrypted remote backend + restricted access |
| Over-privileged IAM | Least privilege + separate role per env |
| Unapproved changes | Manual approval gates in CI/CD |
| Misconfiguration | tfsec + checkov in pipeline |
| Module supply chain | Pin versions + audit source code |
| Blast radius | Separate state per environment |
| Audit trail | Git history + CloudTrail + TF Cloud logs |

---

## 2. Common Errors & Troubleshooting

### Error Resolution Matrix:

```
┌─────────────────────────────────┬──────────────────────────────────────────┐
│ Error                           │ Fix                                      │
├─────────────────────────────────┼──────────────────────────────────────────┤
│ Provider not found              │ terraform init -upgrade                  │
│ Unsupported attribute           │ Check var name, run terraform validate   │
│ Reference to undeclared resource│ Ensure resource exists, use depends_on   │
│ Timeout waiting for instance    │ Retry, add timeouts block, check quotas  │
│ remote-exec fails               │ Check SSH key, SG port 22, username      │
│ State is locked                 │ Wait or: terraform force-unlock <ID>     │
│ Resource already exists         │ terraform import <res> <id>              │
│ DependencyViolation on destroy  │ destroy dependents first, or use -target │
│ Unexpected plan changes (drift) │ terraform apply -refresh-only            │
│ Circular dependency             │ Refactor, use depends_on explicitly      │
└─────────────────────────────────┴──────────────────────────────────────────┘
```

---

### Debug Mode:

```bash
# Enable verbose logging
export TF_LOG=DEBUG          # Levels: TRACE > DEBUG > INFO > WARN > ERROR
export TF_LOG_PATH=terraform.log
terraform plan

# Interactive testing
terraform console
> var.instance_type
"t3.micro"
> cidrsubnet("10.0.0.0/16", 8, 1)
"10.0.1.0/24"
```

---

### State Recovery Steps:

```
Problem: State is corrupted or lost

Step 1: Check S3 versioning → restore previous version
Step 2: If no backup:
        - terraform state pull > backup.tfstate (if partial state exists)
        - terraform import each resource manually
Step 3: terraform plan (verify alignment)
Step 4: terraform apply -refresh-only (sync)
```

---

### 🎯 Interview Point — Troubleshooting Approach:

"When Terraform fails, my approach is:
1. Read the error message carefully (it's usually precise)
2. Check `terraform validate` for syntax
3. Enable `TF_LOG=DEBUG` for API-level issues
4. Use `terraform console` to test expressions
5. Check state with `terraform state list/show`
6. For drift: `terraform apply -refresh-only`
7. For locks: verify no running ops, then `force-unlock`"

---

## 3. Terraform Quick Reference

### File Structure:

```
project/
├── main.tf              # Core resources
├── variables.tf         # Input variable declarations
├── outputs.tf           # Output values
├── terraform.tfvars     # Variable values (gitignore if secrets)
├── backend.tf           # Remote state config
├── versions.tf          # Provider + Terraform version constraints
├── locals.tf            # Local computed values
└── .terraform.lock.hcl  # Provider version lock (commit this!)
```

---

### Block Patterns (One-Page Reference):

```hcl
# RESOURCE
resource "aws_instance" "web" {
  ami           = "ami-123"
  instance_type = var.type
  tags          = local.common_tags
}

# VARIABLE
variable "type" {
  type    = string
  default = "t2.micro"
}

# OUTPUT
output "ip" {
  value     = aws_instance.web.public_ip
  sensitive = false
}

# LOCALS
locals {
  common_tags = { Env = terraform.workspace, ManagedBy = "Terraform" }
}

# DATA SOURCE
data "aws_ami" "latest" {
  most_recent = true
  owners      = ["amazon"]
}

# MODULE
module "vpc" {
  source  = "./modules/vpc"
  cidr    = "10.0.0.0/16"
}
```

---

### Loops & Conditionals:

```hcl
# for_each (preferred — key-based, stable)
resource "aws_s3_bucket" "env" {
  for_each = toset(["dev", "staging", "prod"])
  bucket   = "app-${each.key}"
}

# count (index-based — fragile)
resource "aws_instance" "web" {
  count         = var.create_instance ? 1 : 0   # Conditional creation
  instance_type = "t2.micro"
}

# Conditional expression
instance_type = var.env == "prod" ? "t3.large" : "t3.micro"

# Dynamic block
dynamic "ingress" {
  for_each = var.rules
  content {
    from_port   = ingress.value.port
    to_port     = ingress.value.port
    protocol    = "tcp"
    cidr_blocks = [ingress.value.cidr]
  }
}
```

---

### CLI Commands (Complete):

| Category | Command | Purpose |
|----------|---------|---------|
| **Init** | `terraform init` | Download providers/modules |
| | `terraform init -upgrade` | Upgrade to latest providers |
| **Plan** | `terraform plan` | Dry run |
| | `terraform plan -out=p.tfplan` | Save plan |
| | `terraform plan -target=res` | Plan single resource |
| **Apply** | `terraform apply` | Execute changes |
| | `terraform apply p.tfplan` | Apply saved plan |
| | `terraform apply -auto-approve` | Skip prompt (CI) |
| **Destroy** | `terraform destroy` | Delete all |
| | `terraform destroy -target=res` | Delete one resource |
| **State** | `terraform state list` | List managed resources |
| | `terraform state show <res>` | Resource details |
| | `terraform state mv old new` | Rename |
| | `terraform state rm <res>` | Unmanage (don't delete) |
| | `terraform import <res> <id>` | Import existing |
| **Validate** | `terraform validate` | Syntax check |
| | `terraform fmt` | Auto-format |
| | `terraform fmt -check` | CI format check |
| **Debug** | `terraform console` | Interactive REPL |
| | `terraform graph` | Dependency graph (DOT) |
| | `terraform show` | Human-readable state |
| **Workspace** | `terraform workspace new X` | Create env |
| | `terraform workspace select X` | Switch env |

---

## 🔥 Top Interview Questions from This Section

1. **How do you secure Terraform state?**
   → Encrypted remote backend (S3+KMS), restricted IAM, versioning enabled, DynamoDB locking, never in Git.

2. **How do you manage secrets in Terraform?**
   → Never in code. Use env vars (`TF_VAR_*`), Vault, AWS Secrets Manager, or CI/CD secret injection. Mark outputs `sensitive = true`.

3. **What security tools do you use with Terraform?**
   → tfsec (static analysis), checkov (compliance), TFLint (linting). All integrated in CI pipeline on every PR.

4. **How do you handle drift?**
   → `terraform apply -refresh-only` to detect. Investigate manual changes. Re-import if needed. Prevent with "Terraform-only" policy.

5. **What do you do when state is locked?**
   → First check if another operation is running. If genuinely stuck: `terraform force-unlock <LOCK_ID>`. Always investigate root cause.

6. **How do you handle "resource already exists" error?**
   → `terraform import` to bring it under management. Then `plan` to align config with reality.

7. **How do you limit blast radius?**
   → Separate state per environment. Use `-target` for risky changes. Require manual approval for prod. Use Sentinel/OPA policies.

8. **What's your approach to least-privilege for Terraform?**
   → Dedicated IAM role per env, scoped to only needed services. OIDC federation in CI (no static keys). STS AssumeRole with session duration limits.

---
# Terraform - Real-World Architecture, Multi-Cloud, Tool Integration & K8s/Helm (Interview Ready Notes)

---

## 1. Real-World Architecture Patterns

### 3-Tier Web App on AWS (Most Common Interview Scenario):

```
┌─────────────────────────────────────────────────────────┐
│                        INTERNET                          │
└────────────────────────────┬────────────────────────────┘
                             │
                    ┌────────▼────────┐
                    │   ALB (Public)   │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
      ┌───────▼──┐   ┌──────▼───┐   ┌─────▼────┐
      │ EC2 (AZ1)│   │ EC2 (AZ2)│   │ EC2 (AZ3)│  ← Private Subnet
      └───────┬──┘   └──────┬───┘   └─────┬────┘
              │              │              │
              └──────────────┼──────────────┘
                             │
                    ┌────────▼────────┐
                    │   RDS (Multi-AZ) │  ← Private Subnet
                    └─────────────────┘
```

**Terraform Resources:**
```
VPC + Subnets + NAT → aws_vpc, aws_subnet, aws_nat_gateway
ALB                  → aws_lb, aws_lb_target_group, aws_lb_listener
EC2/ASG              → aws_launch_template, aws_autoscaling_group
Security Groups      → aws_security_group (web, app, db tiers)
RDS                  → aws_db_instance (Multi-AZ)
```

### Folder Layout (Production):
```
project/
├── modules/
│   ├── vpc/          # Network layer
│   ├── ec2/          # Compute layer
│   ├── rds/          # Database layer
│   └── alb/          # Load balancer
├── environments/
│   ├── dev/          # Separate state
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   └── prod/         # Separate state
└── backend.tf
```

---

### Other Architecture Patterns (Quick Reference):

| Pattern | Key Resources |
|---------|---------------|
| **EKS Cluster** | aws_eks_cluster, aws_eks_node_group, IAM roles |
| **Azure Serverless** | azurerm_app_service, azurerm_cosmosdb_account |
| **GCP Auto-Scaled** | google_compute_instance_template, MIG, LB |
| **Serverless (AWS)** | aws_lambda_function, aws_api_gateway, aws_dynamodb_table |

---

## 2. Terraform + Ansible + Packer + Docker

### Tool Responsibilities:

```
┌──────────────────────────────────────────────────────────────┐
│                    DevOps Tool Chain                           │
├──────────┬───────────────────────────────────────────────────┤
│ Packer   │ Build immutable AMIs (pre-bake software)          │
│ Terraform│ Provision infrastructure (VPC, EC2, RDS, K8s)     │
│ Ansible  │ Configure software on running instances           │
│ Docker   │ Package apps as containers                        │
└──────────┴───────────────────────────────────────────────────┘
```

### End-to-End Flow:

```
Packer ──▶ Bake AMI (with NGINX, Docker, monitoring agent)
   │
   ▼
Terraform ──▶ Provision VPC + EC2 (using Packer AMI) + RDS + ALB
   │
   ▼
Ansible ──▶ Configure app (deploy code, SSL certs, env vars)
   │
   ▼
Docker ──▶ Run containers on provisioned infra (or use ECS/K8s)
```

### Terraform + Packer:
```hcl
# Packer builds AMI → Terraform uses it
variable "ami_id" {}  # Passed from Packer output

resource "aws_instance" "web" {
  ami           = var.ami_id    # Pre-baked image!
  instance_type = "t3.micro"
}
```

### Terraform + Ansible:
```hcl
# Option 1: Trigger Ansible from Terraform
resource "null_resource" "ansible" {
  provisioner "local-exec" {
    command = "ansible-playbook -i '${aws_instance.web.public_ip},' playbook.yml"
  }
  depends_on = [aws_instance.web]
}

# Option 2 (Better): Run Ansible separately in CI/CD after TF apply
```

### Terraform + Docker Provider:
```hcl
provider "docker" {}

resource "docker_container" "nginx" {
  name  = "nginx"
  image = "nginx:latest"
  ports { internal = 80; external = 8080 }
}
```

**🎯 Interview Point:** "In production, I use Packer for immutable AMIs (fast boot, no config drift), Terraform for provisioning, and Ansible only for things that can't be baked (secrets injection, dynamic config). Docker runs inside ECS/EKS, not managed directly by Terraform's Docker provider."

---

## 3. Multi-Cloud Deployments

### When to Use Multi-Cloud:
- Avoid vendor lock-in
- Compliance (data residency)
- Best-of-breed (GCP ML + AWS DB)
- DR/redundancy across clouds

### Provider Aliases:
```hcl
provider "aws" {
  region = "us-east-1"
  alias  = "aws_primary"
}

provider "google" {
  project = "my-project"
  region  = "us-central1"
  alias   = "gcp_secondary"
}

# Pass to modules:
module "aws_web" {
  source    = "./modules/aws_webapp"
  providers = { aws = aws.aws_primary }
}

module "gcp_compute" {
  source    = "./modules/gcp_compute"
  providers = { google = google.gcp_secondary }
}
```

### Cross-Cloud Dependencies:
```hcl
# GCP frontend depends on AWS database
resource "google_compute_instance" "frontend" {
  ...
  metadata = {
    db_endpoint = aws_db_instance.backend.endpoint  # Cross-cloud reference
  }
  depends_on = [aws_db_instance.backend]
}
```

### Multi-Cloud Best Practices:

| Practice | Why |
|----------|-----|
| Use provider aliases | Avoid conflicts |
| Separate state per cloud | Limit blast radius |
| Modularize per cloud | Each cloud = own module |
| Env vars for creds | Never hardcode |
| Output sharing between modules | Pass IPs, endpoints |

**🎯 Interview Point:** "Multi-cloud adds complexity. I use it when there's a genuine business requirement (compliance, redundancy). I keep separate state per cloud, separate modules, and orchestrate via CI/CD pipeline stages."

---

## 4. Terraform + Kubernetes + Helm

### Full Stack Diagram:

```
┌────────────────────────────────────────────────────────┐
│                   Terraform                              │
├────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────┐ │
│  │ AWS Infra   │  │ EKS Cluster  │  │ K8s Resources │ │
│  │ VPC/Subnets │  │ Node Groups  │  │ Namespaces    │ │
│  │ IAM/SGs     │  │ IRSA         │  │ Deployments   │ │
│  └─────────────┘  └──────────────┘  └───────────────┘ │
│                                                          │
│  ┌─────────────────────────────────────────────────┐   │
│  │              Helm Releases                        │   │
│  │  • NGINX Ingress    • Cert-Manager               │   │
│  │  • Prometheus/Grafana • ArgoCD                    │   │
│  │  • App Charts                                     │   │
│  └─────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────┘
```

### Provider Chain (EKS → K8s → Helm):
```hcl
# 1. Provision EKS
module "eks" {
  source          = "terraform-aws-modules/eks/aws"
  cluster_name    = "platform-eks"
  cluster_version = "1.27"
  subnet_ids      = module.vpc.private_subnets
  vpc_id          = module.vpc.vpc_id
}

# 2. Get cluster credentials
data "aws_eks_cluster" "eks"      { name = module.eks.cluster_name }
data "aws_eks_cluster_auth" "eks" { name = module.eks.cluster_name }

# 3. Configure K8s provider
provider "kubernetes" {
  host                   = data.aws_eks_cluster.eks.endpoint
  token                  = data.aws_eks_cluster_auth.eks.token
  cluster_ca_certificate = base64decode(data.aws_eks_cluster.eks.certificate_authority[0].data)
}

# 4. Configure Helm provider
provider "helm" {
  kubernetes {
    host                   = data.aws_eks_cluster.eks.endpoint
    token                  = data.aws_eks_cluster_auth.eks.token
    cluster_ca_certificate = base64decode(data.aws_eks_cluster.eks.certificate_authority[0].data)
  }
}
```

### Deploy K8s Resources:
```hcl
resource "kubernetes_namespace" "app" {
  metadata { name = "my-app" }
}

resource "kubernetes_deployment" "nginx" {
  metadata {
    name      = "nginx"
    namespace = kubernetes_namespace.app.metadata[0].name
  }
  spec {
    replicas = 3
    selector { match_labels = { app = "nginx" } }
    template {
      metadata { labels = { app = "nginx" } }
      spec {
        container {
          name  = "nginx"
          image = "nginx:1.21"
          port  { container_port = 80 }
        }
      }
    }
  }
}
```

### Install Helm Charts:
```hcl
resource "helm_release" "ingress" {
  name             = "nginx-ingress"
  namespace        = "ingress-nginx"
  repository       = "https://kubernetes.github.io/ingress-nginx"
  chart            = "ingress-nginx"
  version          = "4.9.1"
  create_namespace = true
}

resource "helm_release" "prometheus" {
  name             = "kube-prometheus-stack"
  namespace        = "monitoring"
  repository       = "https://prometheus-community.github.io/helm-charts"
  chart            = "kube-prometheus-stack"
  version          = "48.0.1"
  create_namespace = true
}

resource "helm_release" "argocd" {
  name             = "argocd"
  namespace        = "argocd"
  repository       = "https://argoproj.github.io/argo-helm"
  chart            = "argo-cd"
  version          = "5.46.5"
  create_namespace = true
}
```

---

### 🎯 Interview Point — Terraform vs ArgoCD for K8s:

```
┌────────────────────┬─────────────────────────────────────────┐
│ Terraform          │ ArgoCD                                   │
├────────────────────┼─────────────────────────────────────────┤
│ Infra provisioning │ App deployment (GitOps)                  │
│ One-time setup     │ Continuous reconciliation                │
│ State-based        │ Git-based (desired state in repo)        │
│ Good for: cluster, │ Good for: app manifests, Helm values,   │
│ namespaces, RBAC   │ rolling updates, canary deployments      │
└────────────────────┴─────────────────────────────────────────┘

Best Practice: Terraform for INFRA + ArgoCD for APPS
```

---

## 5. Production Safeguards Summary

| Layer | Safeguard |
|-------|-----------|
| State | S3 + KMS encryption + DynamoDB lock + versioning |
| Secrets | AWS SSM / Vault → Terraform → Helm values |
| Environments | Workspaces or separate dirs, isolated state |
| CI/CD | Plan → review → manual approve → apply |
| Monitoring | Prometheus + Grafana via Helm |
| GitOps | ArgoCD syncs app state from Git |
| TLS | Cert-Manager + Let's Encrypt |
| Audit | Git history + CloudTrail + TF Cloud logs |

---

## 6. Final Workflow (Production-Grade):

```bash
# Daily workflow
terraform init
terraform fmt -recursive
terraform validate
terraform plan -out=plan.tfplan
terraform apply plan.tfplan
terraform output
```

---

## 🔥 Top Interview Questions from This Section

1. **Describe a production Terraform architecture you've built.**
   → "3-tier on AWS: modular (VPC/EC2/RDS modules), separate state per env, S3+DynamoDB backend, CI/CD with plan review, Prometheus monitoring via Helm on EKS."

2. **How do you combine Terraform with other tools?**
   → "Packer bakes AMIs (immutable infra), Terraform provisions, Ansible handles dynamic config that can't be baked. In K8s world: Terraform for cluster + Helm charts, ArgoCD for app deployments."

3. **Terraform vs ArgoCD — when to use which?**
   → "Terraform for infra (VPC, EKS, RDS, namespaces). ArgoCD for app manifests (continuous reconciliation from Git). They complement each other."

4. **How do you handle multi-cloud with Terraform?**
   → "Provider aliases, separate modules per cloud, separate state. Cross-cloud deps via outputs + depends_on. Orchestrate in CI/CD."

5. **How do you deploy Helm charts with Terraform?**
   → "`helm_release` resource. Pin chart versions. Pass values via `set` blocks or `values` file. `create_namespace = true`."

6. **What's your approach to Terraform + EKS?**
   → "Use official EKS module. Chain providers: EKS data source → kubernetes provider → helm provider. Separate infra apply from app deploy."

7. **Immutable infra vs mutable — which do you prefer?**
   → "Immutable (Packer + Terraform). No config drift, fast rollback (just switch AMI), reproducible. Mutable (Ansible) only for things that MUST be dynamic."

8. **How do you manage secrets in Terraform + K8s?**
   → "Secrets in AWS SSM or Vault. Terraform reads them via data source, passes to Helm values or kubernetes_secret. `sensitive = true` everywhere. Never in code."

---
# Terraform - Senior-Level Deep Dives (Part 7 - Interview Differentiators)

---

## 1. Terragrunt (DRY Terraform at Scale)

**One-liner:** Terragrunt is a thin wrapper around Terraform that keeps configurations DRY, manages dependencies between modules, and simplifies multi-environment setups.

### Why Terragrunt?

```
┌─────────────────────────────────────────────────────────────┐
│ Problem with vanilla Terraform at scale:                     │
│                                                              │
│ • Repeated backend config in every environment               │
│ • Duplicated provider blocks                                 │
│ • No native dependency between root modules                  │
│ • Copy-paste variables across dev/staging/prod               │
│                                                              │
│ Terragrunt solves ALL of these.                              │
└─────────────────────────────────────────────────────────────┘
```

### Terragrunt Folder Structure:

```
live/
├── terragrunt.hcl              # Root — common backend/provider config
├── dev/
│   ├── vpc/
│   │   └── terragrunt.hcl     # Points to modules/vpc
│   ├── eks/
│   │   └── terragrunt.hcl     # Points to modules/eks, depends on vpc
│   └── rds/
│       └── terragrunt.hcl
├── prod/
│   ├── vpc/
│   │   └── terragrunt.hcl
│   ├── eks/
│   │   └── terragrunt.hcl
│   └── rds/
│       └── terragrunt.hcl
└── modules/                    # Reusable Terraform modules
    ├── vpc/
    ├── eks/
    └── rds/
```

### Root terragrunt.hcl (DRY backend):
```hcl
remote_state {
  backend = "s3"
  generate = {
    path      = "backend.tf"
    if_exists = "overwrite"
  }
  config = {
    bucket         = "my-company-terraform-state"
    key            = "${path_relative_to_include()}/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

### Module terragrunt.hcl (e.g., dev/eks/):
```hcl
include "root" {
  path = find_in_parent_folders()
}

terraform {
  source = "../../modules/eks"
}

dependency "vpc" {
  config_path = "../vpc"
}

inputs = {
  vpc_id     = dependency.vpc.outputs.vpc_id
  subnet_ids = dependency.vpc.outputs.private_subnets
  env        = "dev"
}
```

### Key Terragrunt Features:

| Feature | What it does |
|---------|-------------|
| `include` | Inherit parent config (DRY backend/provider) |
| `dependency` | Reference outputs from other modules |
| `run-all` | Apply/plan all modules in dependency order |
| `generate` | Auto-generate backend.tf, provider.tf |
| `inputs` | Pass variables without .tfvars files |

### Commands:
```bash
terragrunt run-all plan    # Plan ALL modules in order
terragrunt run-all apply   # Apply ALL with dependency resolution
terragrunt run-all destroy # Destroy in reverse dependency order
```

### 🎯 Interview Point:
"I use Terragrunt when managing 10+ environments or 20+ modules. It eliminates backend duplication, handles cross-module dependencies via `dependency` blocks, and `run-all` applies everything in the correct order. For smaller setups, vanilla Terraform with separate dirs is fine."

---

## 2. Reading `terraform plan` Output

### Symbol Reference:

```
┌────────┬───────────────────────────────────────────┐
│ Symbol │ Meaning                                    │
├────────┼───────────────────────────────────────────┤
│   +    │ Resource will be CREATED                   │
│   -    │ Resource will be DESTROYED                 │
│   ~    │ Resource will be UPDATED in-place          │
│  -/+   │ Resource will be DESTROYED then RECREATED  │
│  <=    │ Data source will be READ                   │
└────────┴───────────────────────────────────────────┘
```

### Example Plan Output:
```
Terraform will perform the following actions:

  # aws_instance.web will be updated in-place
  ~ resource "aws_instance" "web" {
      ~ instance_type = "t2.micro" -> "t3.medium"    # Changed
        tags          = { Name = "web" }              # Unchanged (no ~)
    }

  # aws_security_group.db will be destroyed
  - resource "aws_security_group" "db" { ... }

  # aws_s3_bucket.logs will be created
  + resource "aws_s3_bucket" "logs" {
      + bucket = "my-app-logs"
      + acl    = "private"
    }

Plan: 1 to add, 1 to change, 1 to destroy.
```

### Forces Replacement (-/+):
```
  # aws_instance.web must be replaced
  -/+ resource "aws_instance" "web" {
      ~ ami = "ami-old123" -> "ami-new456"   # Forces new resource!
    }
```

**🎯 Interview Point:** "When I see `-/+` (replacement), I check if it's expected. Changing AMI, instance type in some cases, or subnet forces recreation. I use `lifecycle { create_before_destroy = true }` to avoid downtime during replacements."

---

## 3. State File Internals

### What's Inside terraform.tfstate:

```json
{
  "version": 4,
  "terraform_version": "1.6.0",
  "serial": 42,              ← Increments on every change
  "lineage": "abc-123-...",  ← Unique ID for this state (prevents mixing)
  "outputs": { ... },
  "resources": [
    {
      "mode": "managed",     ← "managed" (resource) or "data" (data source)
      "type": "aws_instance",
      "name": "web",
      "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
      "instances": [
        {
          "attributes": {
            "id": "i-0abc123def456",
            "ami": "ami-0c55b159",
            "instance_type": "t3.micro",
            "public_ip": "54.23.45.67",
            "tags": { "Name": "web" }
          }
        }
      ]
    }
  ]
}
```

### Key Fields:

| Field | Purpose |
|-------|---------|
| `serial` | Version counter — detects concurrent changes |
| `lineage` | UUID — prevents pushing state to wrong backend |
| `resources[].instances[].attributes` | Actual cloud resource properties |
| `outputs` | All defined output values |

**🎯 Interview Point:** "State contains sensitive data in plaintext (passwords, IPs, keys). That's why encryption + access control is non-negotiable. The `serial` field is how Terraform detects if someone else modified state between your plan and apply."

---

## 4. `moved` Blocks (Refactoring Without State Surgery)

**Problem:** You renamed a resource or moved it into a module. Without `moved`, Terraform destroys old + creates new.

### Before (Terraform < 1.1):
```bash
# Manual state surgery required:
terraform state mv aws_instance.old aws_instance.new
```

### After (Terraform 1.1+):
```hcl
# In your .tf file — declarative, reviewable, version-controlled:
moved {
  from = aws_instance.old_name
  to   = aws_instance.new_name
}

moved {
  from = aws_instance.web
  to   = module.compute.aws_instance.web
}
```

### When to Use:
- Renaming resources
- Moving resources into/out of modules
- Refactoring module structure

**🎯 Interview Point:** "`moved` blocks are superior to `terraform state mv` because they're declarative, in code, reviewable in PRs, and work across team members without manual steps. I leave them in for one release cycle, then remove."

---

## 5. `import` Blocks (Terraform 1.5+ — Declarative Import)

### Before (CLI import):
```bash
terraform import aws_instance.web i-0abc123
# Then manually write the resource block
```

### After (Terraform 1.5+):
```hcl
# import.tf — declarative, plannable!
import {
  to = aws_instance.web
  id = "i-0abc123def456"
}

# resource block (can be auto-generated):
resource "aws_instance" "web" {
  ami           = "ami-0c55b159"
  instance_type = "t3.micro"
}
```

### Generate Config Automatically:
```bash
terraform plan -generate-config-out=generated.tf
```
This creates the resource block FOR you based on the imported resource!

**🎯 Interview Point:** "Declarative import + config generation is a game changer for brownfield adoption. I can import 100 resources in a plan, review them, and apply — no manual state surgery."

---

## 6. Terraform Native Testing (`terraform test`)

### Available since Terraform 1.6+

```
tests/
├── vpc_test.tftest.hcl
└── ec2_test.tftest.hcl
```

### Test File Example (tests/ec2_test.tftest.hcl):
```hcl
run "create_instance" {
  command = apply

  variables {
    instance_type = "t3.micro"
    ami           = "ami-0c55b159"
  }

  assert {
    condition     = aws_instance.web.instance_type == "t3.micro"
    error_message = "Instance type mismatch"
  }

  assert {
    condition     = length(aws_instance.web.tags) > 0
    error_message = "Tags must not be empty"
  }
}

run "validate_outputs" {
  command = plan

  assert {
    condition     = output.public_ip != ""
    error_message = "Public IP must be set"
  }
}
```

### Run Tests:
```bash
terraform test
```

### Terraform Test vs Terratest:

| Aspect | `terraform test` | Terratest |
|--------|-------------------|-----------|
| Language | HCL (native) | Go |
| Setup | Zero — built-in | Go environment needed |
| Speed | Fast (can mock) | Slow (real infra) |
| Maturity | Newer (1.6+) | Battle-tested |
| Best for | Unit tests, validation | Integration tests, E2E |

**🎯 Interview Point:** "I use `terraform test` for quick validation (type checks, output assertions, mock runs). For full integration tests that deploy real infra, I still use Terratest."

---

## 7. Dependency Graph & Execution Order

### How Terraform Determines Order:

```
Terraform builds a DAG (Directed Acyclic Graph):

     aws_vpc
       │
       ├──────────────┐
       ▼              ▼
  aws_subnet_a    aws_subnet_b
       │              │
       └──────┬───────┘
              ▼
        aws_instance
              │
              ▼
        aws_eip
```

### Visualize:
```bash
terraform graph | dot -Tpng > graph.png
# Or view in browser:
terraform graph | dot -Tsvg > graph.svg
```

### Implicit vs Explicit Dependencies:

```hcl
# IMPLICIT — Terraform auto-detects via references:
resource "aws_instance" "web" {
  subnet_id = aws_subnet.main.id  # TF knows: subnet must exist first
}

# EXPLICIT — When there's no direct reference but order matters:
resource "aws_instance" "web" {
  depends_on = [aws_iam_role_policy.s3_access]
  # No direct reference, but instance needs the policy to exist
}
```

**🎯 Interview Point:** "Terraform parallelizes everything it can. It only serializes when there's a dependency (implicit via reference or explicit via `depends_on`). Understanding the graph helps debug slow plans and circular dependency errors."

---

## 8. Common Scenario Questions (Interview Favorites)

### Q: How do you achieve zero-downtime deployments with Terraform?

```hcl
resource "aws_instance" "web" {
  ami           = var.new_ami
  instance_type = "t3.medium"

  lifecycle {
    create_before_destroy = true  # New instance UP before old one DOWN
  }
}
```

**Better answer:** "For true zero-downtime, I use:
1. Blue-green via ASG: Terraform creates new ASG → ALB shifts traffic → destroy old ASG
2. Rolling updates via EKS: Update deployment image, K8s handles rolling
3. `create_before_destroy` for individual resources"

---

### Q: How do you handle Terraform in a large team (50+ engineers)?

```
┌─────────────────────────────────────────────────────┐
│ Large Team Strategy:                                 │
│                                                      │
│ 1. Terragrunt for DRY configs + dependencies        │
│ 2. Separate state per service/team (blast radius)   │
│ 3. CODEOWNERS on Terraform directories              │
│ 4. Atlantis or TF Cloud for plan-on-PR              │
│ 5. Sentinel/OPA for policy guardrails               │
│ 6. Module registry (private) for standardization    │
│ 7. PR-based workflow: plan comment → approve → apply│
│ 8. Separate "platform" team owns base modules       │
└─────────────────────────────────────────────────────┘
```

---

### Q: How do you migrate from one Terraform structure to another?

**Steps:**
1. Write new structure (modules, files)
2. Use `moved` blocks for renames/relocations
3. `terraform plan` — verify 0 destroy, 0 create (just moves)
4. If moving between state files: `terraform state mv -state-out=new.tfstate`
5. Test in lower env first

---

### Q: How do you handle Terraform state split/merge?

**Split (monolith → microservices):**
```bash
# Move resources to new state:
terraform state mv -state-out=service-a.tfstate aws_instance.service_a
terraform state mv -state-out=service-b.tfstate aws_instance.service_b
```

**Merge (rare, careful):**
```bash
terraform state pull > combined.tfstate
# Manually merge or use terraform import in target project
```

---

### Q: What is Atlantis?

```
┌─────────────────────────────────────────────┐
│ Atlantis = Self-hosted Terraform automation  │
│                                              │
│ PR opened → Atlantis runs `plan`            │
│ Plan posted as PR comment                    │
│ Reviewer approves → comment `atlantis apply` │
│ Atlantis applies and comments result         │
└─────────────────────────────────────────────┘
```

Alternative to Terraform Cloud for teams wanting self-hosted control.

---

### Q: How do you handle breaking changes in provider upgrades?

```
1. Pin provider versions strictly in prod:
   version = "= 5.30.0"  (exact pin)

2. Upgrade process:
   - Read changelog
   - Bump in dev first
   - Run plan — check for unexpected changes
   - Fix deprecated resources/attributes
   - Apply in dev → staging → prod

3. Use .terraform.lock.hcl (commit this file!)
   - Locks exact provider hashes
   - Ensures all team members use same version
```

---

## 9. Terraform Anti-Patterns (What NOT to Do)

| Anti-Pattern | Why It's Bad | Instead |
|-------------|-------------|---------|
| Monolithic state (all infra in one state) | One bad apply = everything breaks | Split by service/layer |
| `terraform apply` without plan review | Could destroy production | Always plan → review → apply |
| Hardcoded values | Not reusable, error-prone | Variables + locals + data sources |
| Using `count` for distinct resources | Index shift = recreation | Use `for_each` with keys |
| Manual state edits | Corruption risk | Use `state mv`, `state rm`, `moved` blocks |
| Secrets in .tf files | Security breach | Env vars / Vault / SSM |
| Not pinning versions | Breaking changes surprise | Pin providers + modules |
| Overly complex modules | Hard to debug/maintain | Small, focused, composable |
| Skipping `-out` in CI/CD | Plan drift between plan and apply | Always `plan -out` → `apply plan` |
| Using default workspace for prod | Accidental apply to wrong env | Named workspaces or separate dirs |

---

## 🔥 Senior Interview Questions (Differentiators)

1. **What is Terragrunt and when would you use it?**
   → "Wrapper for DRY configs. Use when you have 10+ envs, need cross-module dependencies, or want auto-generated backends. Overkill for small projects."

2. **Explain the Terraform dependency graph.**
   → "DAG of all resources. Terraform parallelizes independent resources and serializes dependencies. `terraform graph` visualizes it. Circular deps = error."

3. **How do you refactor Terraform without destroying resources?**
   → "`moved` blocks (declarative, in code). For cross-state moves: `terraform state mv -state-out`. Always plan first to verify 0 destroy."

4. **What are `import` blocks and why are they better than CLI import?**
   → "Declarative, plannable, config-generatable. Can import multiple resources in one plan. No manual state surgery."

5. **How do you test Terraform code?**
   → "Three layers: `terraform validate` (syntax), `terraform test` (unit/assertion), Terratest (full integration). Plus tfsec/checkov for security."

6. **What's your strategy for Terraform at scale (50+ engineers)?**
   → "Terragrunt + Atlantis + private module registry + CODEOWNERS + Sentinel policies + separate state per service."

7. **How do you handle zero-downtime with Terraform?**
   → "`create_before_destroy` for simple cases. Blue-green ASG swap for EC2. For K8s, let the orchestrator handle rolling updates."

8. **What anti-patterns have you seen and fixed?**
   → "Monolithic state (split it), count for distinct resources (switched to for_each), no plan review (added Atlantis), secrets in code (moved to Vault)."

---
