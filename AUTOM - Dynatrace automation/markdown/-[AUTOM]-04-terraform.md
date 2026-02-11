# Terraform Provider

> **Series:** AUTOM | **Notebook:** 4 of 8 | **Created:** January 2026 | **Last Updated:** 02/11/2026

The Dynatrace Terraform provider enables infrastructure-as-code management of Dynatrace configurations. It integrates with Terraform's ecosystem for state management, planning, and CI/CD integration.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Provider Configuration](#provider-configuration)
4. [Resource Types](#resource-types)
5. [State Management](#state-management)
6. [Advanced Patterns](#advanced-patterns)

---

## Prerequisites

Before starting this notebook, ensure you have:

| Requirement | Description |
|-------------|-------------|
| Terraform CLI | Version 1.0+ installed |
| Authentication | One of: **Access Token** (classic), **Platform Token**, or **OAuth Client** (see [Provider Configuration](#provider-configuration)) |
| Tenant URL | Your Dynatrace SaaS tenant URL |
| HCL Knowledge | Basic familiarity with Terraform syntax |

### Authentication Methods

The Dynatrace Terraform provider supports three authentication methods:

| Method | Use Case | Key |
|--------|----------|-----|
| **Access Token (Classic)** | Settings API, classic config resources | `dt_api_token` |
| **Platform Token** | User-friendly alternative; works within user's permissions | `dt_api_token` (same parameter) |
| **OAuth Client** | **Required** for Automation, Document, and Account Management resources | `dt_client_id` + `dt_client_secret` |

> **Important:** If you manage Workflows, Documents, or IAM resources via Terraform, you **must** configure OAuth client credentials in addition to (or instead of) an access/platform token.

---

## Learning Objectives

By the end of this notebook, you will:

- Understand the Dynatrace Terraform provider
- Know how to configure resources in HCL
- Be able to manage state and handle drift
- Implement multi-environment deployments

---

<a id="introduction"></a>
## 1. Introduction
### Why Terraform?

| Benefit | Description |
|---------|-------------|
| **State Management** | Track what exists vs. what's defined |
| **Drift Detection** | Identify manual changes |
| **Planning** | Preview changes before applying |
| **Ecosystem** | Integrate with other Terraform providers |
| **Modules** | Reusable configuration packages |

### Terraform vs Monaco

| Aspect | Terraform | Monaco |
|--------|-----------|--------|
| State file | Required | None |
| Drift detection | Built-in | Manual |
| Learning curve | Higher | Lower |
| Multi-cloud | Yes | Dynatrace only |
| Dependencies | Explicit | Implicit |

---

<a id="getting-started"></a>
## 2. Getting Started
### Installation

**Install Terraform:**

macOS:
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

Linux:
```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

**Verify installation:**
```bash
terraform version
```

---

<a id="provider-configuration"></a>
## 3. Provider Configuration

The Dynatrace Terraform provider supports three authentication methods. Which you need depends on the resources you manage.

### Method 1: Access Token or Platform Token

Use for **Settings API** and **classic configuration** resources (management zones, auto-tags, alerting, SLOs, synthetic monitors, etc.).

**Access tokens** (classic) require explicit scopes like `settings.read`, `settings.write`, `ReadConfig`, `WriteConfig`. **Platform tokens** are a newer, user-friendly alternative — they work within the assigned user's permissions, so a selected scope only grants access if that user has the respective permission.

```hcl
terraform {
  required_providers {
    dynatrace = {
      source  = "dynatrace-oss/dynatrace"
      version = "~> 1.0"
    }
  }
}

provider "dynatrace" {
  dt_env_url   = var.dynatrace_url
  dt_api_token = var.dynatrace_token  # Access token or platform token
}
```

### Method 2: OAuth Client Credentials

**Required** for Automation (Workflows), Document, and Account Management resources. The provider exchanges your client ID and secret for short-lived OAuth access tokens automatically.

```hcl
provider "dynatrace" {
  dt_env_url   = var.dynatrace_url
  dt_api_token = var.dynatrace_token  # For settings/classic resources

  # OAuth credentials — required for automation, document, and IAM resources
  automation_client_id     = var.automation_client_id
  automation_client_secret = var.automation_client_secret
}
```

> **Tip:** If you only manage automation/document resources, you can omit `dt_api_token` and use OAuth alone.

### Method 3: Environment Variables (Recommended for CI/CD)

Instead of hardcoding values, set environment variables:

| Variable | Purpose |
|----------|---------|
| `DYNATRACE_ENV_URL` or `DT_ENV_URL` | Tenant URL |
| `DYNATRACE_API_TOKEN` or `DT_API_TOKEN` | Access token or platform token |
| `DT_CLIENT_ID` | OAuth client ID (automation/documents) |
| `DT_CLIENT_SECRET` | OAuth client secret |
| `DT_ACCOUNT_ID` | Account UUID (for account-level IAM resources) |

```bash
export DT_ENV_URL="https://abc12345.live.dynatrace.com"
export DT_API_TOKEN="dt0c01.XXXX..."
export DT_CLIENT_ID="dt0s02.XXXX"
export DT_CLIENT_SECRET="dt0s02.XXXX.YYYY..."
```

When environment variables are set, the provider block can be minimal:

```hcl
provider "dynatrace" {}
```

### Variables File

Create a `variables.tf` file:

```hcl
variable "dynatrace_url" {
  description = "Dynatrace environment URL"
  type        = string
}

variable "dynatrace_token" {
  description = "Dynatrace access token or platform token"
  type        = string
  sensitive   = true
}

variable "automation_client_id" {
  description = "OAuth client ID for automation resources"
  type        = string
  default     = ""
}

variable "automation_client_secret" {
  description = "OAuth client secret for automation resources"
  type        = string
  sensitive   = true
  default     = ""
}
```

### Which Authentication for Which Resources?

| Resource Category | Access/Platform Token | OAuth Client |
|-------------------|-----------------------|--------------|
| Settings 2.0 (management zones, auto-tags, etc.) | Yes | No |
| Classic config (dashboards, alerting, etc.) | Yes | No |
| Synthetic monitors, SLOs | Yes | No |
| **Automation (Workflows, scheduling)** | No | **Required** |
| **Documents (notebooks, dashboards v2)** | No | **Required** |
| **Account Management (IAM, policies)** | No | **Required** (`DT_ACCOUNT_ID` also needed) |

### Creating an OAuth Client

1. Go to **Account Management** > **Identity & Access Management** > **OAuth clients**
2. Create a new client with the required scopes:

| Scope | Purpose |
|-------|---------|
| `automation:workflows:read` | Read workflow definitions |
| `automation:workflows:write` | Create/update workflows |
| `automation:rules:read` | Read scheduling rules |
| `automation:rules:write` | Create/update scheduling rules |
| `document:documents:read` | Read documents |
| `document:documents:write` | Create/update documents |
| `document:documents:delete` | Delete documents |

3. Copy the generated **client ID** and **client secret** immediately — the secret is only shown once
4. Store credentials securely (vault, CI/CD secrets, etc.)

> **Security:** Never commit OAuth client secrets or API tokens to version control. Use environment variables, HashiCorp Vault, AWS Secrets Manager, or your CI/CD platform's secret store.

---

### Initialize and Validate

```bash
# Initialize provider
terraform init

# Validate configuration
terraform validate

# Format code
terraform fmt
```

---

<a id="resource-types"></a>
## 4. Resource Types
### Management Zone

```hcl
resource "dynatrace_management_zone_v2" "production" {
  name = "Production"

  rules {
    type    = "SERVICE"
    enabled = true

    conditions {
      condition {
        key {
          type      = "STATIC"
          attribute = "SERVICE_TAGS"
        }
        tag {
          operator = "EQUALS"
          negate   = false
          value {
            context = "CONTEXTLESS"
            key     = "environment"
            value   = "production"
          }
        }
      }
    }
  }
}
```

### Auto-Tagging Rule

```hcl
resource "dynatrace_autotag_v2" "application" {
  name = "Application"

  rules {
    type         = "SERVICE"
    enabled      = true
    value_format = "{Service:DetectedName}"
    conditions   = []
  }
}
```

---

### Alerting Profile

```hcl
resource "dynatrace_alerting" "production_alerts" {
  name            = "Production Alerting"
  management_zone = dynatrace_management_zone_v2.production.id

  rules {
    delay_in_minutes = 0
    severity_level   = "AVAILABILITY"
    tag_filters      = []
  }

  rules {
    delay_in_minutes = 5
    severity_level   = "ERRORS"
    tag_filters      = []
  }

  rules {
    delay_in_minutes = 10
    severity_level   = "PERFORMANCE"
    tag_filters      = []
  }
}
```

### SLO

```hcl
resource "dynatrace_slo_v2" "availability" {
  name              = "Production Availability"
  enabled           = true
  evaluation_type   = "AGGREGATE"
  evaluation_window = "-1w"
  target_success    = 99.9
  target_warning    = 99.95
  
  metric_expression = "builtin:synthetic.http.availability.location.total:splitBy()"
  
  filter            = "type(SYNTHETIC_TEST)"
}
```

---

### HTTP Monitor (Synthetic)

```hcl
resource "dynatrace_http_monitor" "homepage" {
  name      = "Homepage Check"
  enabled   = true
  frequency = 5

  locations = ["GEOLOCATION-1234567890ABCDEF"]

  anomaly_detection {
    loading_time_thresholds {
      enabled = true
    }
    outage_handling {
      global_outage = true
      local_outage  = false
    }
  }

  script {
    request {
      description = "Homepage"
      method      = "GET"
      url         = "https://example.com"

      validation {
        rule {
          type  = "httpStatusesList"
          value = ">=400"
          pass_if_found = false
        }
      }
    }
  }
}
```

---

<a id="state-management"></a>
## 5. State Management
### Understanding Terraform State

Terraform tracks resources in a state file (`terraform.tfstate`):

| Concept | Description |
|---------|-------------|
| **State** | JSON file mapping config to real resources |
| **Plan** | Comparison of state vs. desired config |
| **Apply** | Execute changes to reach desired state |
| **Drift** | Difference between state and actual |

### Remote State Storage

For team collaboration, use remote state:

```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "dynatrace/production/terraform.tfstate"
    region = "us-east-1"
  }
}
```

Or using Terraform Cloud:

```hcl
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "dynatrace-production"
    }
  }
}
```

---

### Common State Commands

```bash
# Preview changes
terraform plan

# Apply changes
terraform apply

# Show current state
terraform show

# List resources in state
terraform state list

# Import existing resource
terraform import dynatrace_management_zone_v2.production "<object-id>"

# Remove resource from state (without deleting)
terraform state rm dynatrace_management_zone_v2.production
```

### Importing Existing Resources

To manage existing configurations:

1. Write the resource block in HCL
2. Find the object ID in Dynatrace
3. Import into state

```bash
terraform import dynatrace_management_zone_v2.production "vu9U3hXa3q0AAAABABhidWlsdGluOm1hbmFnZW1lbnQtem9uZXMABnRlbmFudAAGdGVuYW50ABp2dTlVM2hYYTNxMERUVF9fUHJvZHVjdGlvbr7vFJ4"
```

---

<a id="advanced-patterns"></a>
## 6. Advanced Patterns
### Modules for Reusability

Create reusable modules:

**modules/environment/main.tf:**
```hcl
variable "environment_name" {
  type = string
}

variable "environment_tag" {
  type = string
}

resource "dynatrace_management_zone_v2" "zone" {
  name = var.environment_name

  rules {
    type    = "SERVICE"
    enabled = true

    conditions {
      condition {
        key {
          type      = "STATIC"
          attribute = "SERVICE_TAGS"
        }
        tag {
          operator = "EQUALS"
          negate   = false
          value {
            context = "CONTEXTLESS"
            key     = "environment"
            value   = var.environment_tag
          }
        }
      }
    }
  }
}

output "management_zone_id" {
  value = dynatrace_management_zone_v2.zone.id
}
```

**Use the module:**
```hcl
module "production" {
  source           = "./modules/environment"
  environment_name = "Production"
  environment_tag  = "production"
}

module "staging" {
  source           = "./modules/environment"
  environment_name = "Staging"
  environment_tag  = "staging"
}
```

---

### Workspaces for Environments

Use workspaces to manage multiple environments:

```bash
# Create workspaces
terraform workspace new development
terraform workspace new staging
terraform workspace new production

# Switch workspace
terraform workspace select production

# List workspaces
terraform workspace list
```

**Use workspace in config:**
```hcl
locals {
  environment = terraform.workspace
  
  config = {
    development = {
      alert_delay = 30
    }
    staging = {
      alert_delay = 15
    }
    production = {
      alert_delay = 5
    }
  }
}

resource "dynatrace_alerting" "alerts" {
  name = "${local.environment} Alerting"
  
  rules {
    delay_in_minutes = local.config[local.environment].alert_delay
    severity_level   = "AVAILABILITY"
  }
}
```

---

### Best Practices

| Practice | Description |
|----------|-------------|
| **Remote state** | Never use local state for teams |
| **State locking** | Enable to prevent concurrent changes |
| **Modular design** | Use modules for reusable patterns |
| **Version pinning** | Pin provider versions |
| **Plan before apply** | Always review plan output |
| **Meaningful names** | Resource names should be descriptive |

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| "Resource already exists" | State out of sync | Import the resource |
| "Provider error" | Invalid token | Check credentials |
| "Drift detected" | Manual changes | Re-apply or update state |
| Slow plan/apply | Many resources | Use targets or modules |

---

<a id="next-steps"></a>
## 7. Next Steps

### Terraform Export Utility

To export existing Dynatrace configuration to Terraform:

```bash
# Install terraform-provider-dynatrace-export
# See: https://github.com/dynatrace-oss/terraform-provider-dynatrace

# Export all configurations
terraform-provider-dynatrace-export \
  -e https://{tenant}.live.dynatrace.com \
  -t {api-token} \
  -o ./exported-config
```

### Continue the Series

| Next Notebook | Focus |
|---------------|-------|
| **AUTOM-05: Dynatrace Workflows** | Event-driven automation |

### Additional Resources

- [Terraform Provider Documentation](https://registry.terraform.io/providers/dynatrace-oss/dynatrace/latest/docs)
- [Provider GitHub Repository](https://github.com/dynatrace-oss/terraform-provider-dynatrace)
- [Terraform Style Guide](https://developer.hashicorp.com/terraform/language/style)

---

## Summary

In this notebook, you learned:

- How to configure the Dynatrace Terraform provider
- Creating resources like management zones, auto-tags, and SLOs
- State management and drift detection
- Advanced patterns with modules and workspaces

> **Key Takeaway:** Terraform excels when you need state management, drift detection, or integration with other infrastructure. For Dynatrace-only config management, Monaco is often simpler.

---

*Continue to **AUTOM-05: Dynatrace Workflows** to learn event-driven automation.*

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
