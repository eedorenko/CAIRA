---
# Core Metadata
title: "Kata: Devcontainer & Foundry Basic Deployment"
description: Deploy CAIRA foundry_basic architecture using devcontainer workflow with AI-assisted Terraform deployment and validation
author: Edge AI Team
ms.date: 2025-12-02
ms.topic: how-to-guide

# Kata Identity
kata_id: caira-fundamentals-200-devcontainer-foundry-basic-deployment
kata_category:
  - ai-assisted-engineering
kata_difficulty: 2
estimated_time_minutes: 35

# Learning Content
learning_objectives:
  - Develop inside CAIRA devcontainer with all tools pre-configured
  - Authenticate Azure CLI within containerized development environment
  - Use caira-assistant chatmode for guided Terraform deployment workflow
  - Execute complete Terraform lifecycle (init, plan, apply) safely
  - Make informed configuration decisions for variable management
  - Validate successful Azure AI Foundry infrastructure deployment
prerequisite_katas:
  - caira-fundamentals-150-understanding-architecture-patterns
technologies:
  - Docker
  - VS Code Dev Containers
  - CAIRA
  - Terraform
  - Azure CLI
  - Azure AI Foundry
  - caira-assistant chatmode
success_criteria:
  - Devcontainer opened successfully with all tools available
  - Azure CLI authenticated and subscription verified
  - Terraform initialized without errors in foundry_basic directory
  - Plan reviewed showing only expected resource creations
  - Infrastructure deployed successfully to Azure
  - All expected resources visible in Azure Portal
  - Can explain devcontainer benefits and deployment workflow

# AI Coaching
ai_coaching_level: guided
scaffolding_level: medium-heavy
hint_strategy: progressive
common_pitfalls:
  - Docker Desktop not running before opening devcontainer
  - Skipping Azure CLI login step inside container (authentication scoped to container)
  - Applying Terraform without reviewing plan first
  - Insufficient Azure permissions (need Contributor + User Access Administrator)
  - Not setting ARM_SUBSCRIPTION_ID environment variable before Terraform commands

# Requirements
requires_azure_subscription: true
requires_local_environment: true
requires_github_account: false

# SEO & Discoverability
tags:
  - ai-assisted-engineering
search_keywords:
  - CAIRA devcontainer deployment
  - Azure AI Foundry terraform
  - caira-assistant guided deployment
  - foundry_basic architecture
  - containerized development workflow
real_world_application: New team members onboard to CAIRA platform by deploying foundry_basic architecture using standardized devcontainer environment, following AI-assisted workflow that ensures safe configuration and deployment practices aligned with team standards
---

<!--
AI_COACH: This kata introduces hands-on Terraform deployment within devcontainer.
Guide learners to ALWAYS review terraform plan output before apply. If they rush
to apply without reviewing, remind them this is a critical safety practice.
The caira-assistant chatmode should be used throughout for validation checkpoints.
-->

## Quick Context

**You'll Learn**: How to deploy CAIRA's foundry_basic architecture using the built-in devcontainer environment and AI-assisted deployment workflow for safe, guided infrastructure provisioning.

**Real Challenge**: Your team needs a standardized way to onboard new engineers to the CAIRA platform. Everyone should use the same development environment with pre-configured tools, follow consistent deployment workflows, and leverage AI assistance to avoid common configuration mistakes. You're the first to validate this workflow.

**Your Task**: Open CAIRA in VS Code devcontainer, authenticate with Azure, use caira-assistant to guide your deployment decisions, and successfully deploy the foundry_basic architecture with proper validation at each step.

## Essential Setup

**Required** (check these first):

- [ ] Docker Desktop installed and **running** (check status bar/tray icon)
- [ ] VS Code installed with Remote - Containers extension (ms-vscode-remote.remote-containers)
- [ ] CAIRA repository cloned locally (`git clone https://github.com/microsoft/CAIRA.git`)
- [ ] Azure subscription with these permissions: **Contributor** + **User Access Administrator** (or **Owner**)
- [ ] Azure CLI authentication capability (browser or device code flow available)
- [ ] Completed Kata 150 (Understanding CAIRA Architecture Patterns)

**Quick Validation**: Open Docker Desktop and confirm it shows "Engine running". In VS Code, verify the Remote - Containers extension is installed and enabled.

> **🤖 Want Interactive AI Coaching?**
>
> Load the **CAIRA Assistant** chat mode for deployment-specific guidance and validation!
>
> In GitHub Copilot Chat, select **caira-assistant** mode and say:
>
> ```text
> I'm working on the Devcontainer & Foundry Basic Deployment kata and want guidance for deploying foundry_basic architecture.
> ```

## Practice Tasks

### Task 1: Launch Devcontainer & Verify Environment (10 minutes)

**What You'll Do**: Open the CAIRA repository in a VS Code devcontainer and verify all development tools are available and properly configured.

**Steps**:

1. **Open** CAIRA repository in VS Code
   - [ ] Launch VS Code
   - [ ] Open the folder containing your cloned CAIRA repository
   - [ ] Wait for VS Code to detect the `.devcontainer` configuration
   - **Pro tip**: VS Code should show a notification "Folder contains a Dev Container configuration file"

2. **Reopen** in devcontainer
   - [ ] Click "Reopen in Container" when prompted (or use Command Palette: "Dev Containers: Reopen in Container")
   - [ ] Wait for container build and initialization (first time: 3-5 minutes; subsequent: 30-60 seconds)
   - [ ] Verify status bar shows "Dev Container: CAIRA"
   - **Expected result**: Terminal opens inside container with vscode@[container-id] prompt

3. **Verify** development tools inside container
   - [ ] Open terminal in VS Code (should be inside container)
   - [ ] Run: `terraform version` (should show v1.13 or later)
   - [ ] Run: `az version` (should show Azure CLI version info)
   - [ ] Run: `which docker` (should show path, confirming Docker-outside-of-Docker works)
   - **Success check**: All three commands return version information without errors

4. **Authenticate** with Azure CLI inside container
   - [ ] Run: `az login` in terminal
   - [ ] Complete authentication flow (browser or device code)
   - [ ] Verify authentication: `az account show`
   - **Validation checkpoint**: Output shows your subscription details in JSON format
   - **Expected result**: You are now authenticated to Azure from within the container

### Task 2: Configure Deployment with AI Assistance (12 minutes)

**What You'll Do**: Use caira-assistant chatmode to determine the right configuration approach and prepare your Terraform variables for foundry_basic deployment.

**Steps**:

1. **Activate** caira-assistant chatmode
   - [ ] Open GitHub Copilot Chat in VS Code
   - [ ] Select **caira-assistant** mode from the dropdown
   - [ ] Ask: "I want to deploy foundry_basic. What are the required variables and recommended configuration approach?"
   - **Pro tip**: The assistant will explain terraform.tfvars vs main.tf variable approaches

2. **Navigate** to foundry_basic directory
   - [ ] In terminal: `cd reference_architectures/foundry_basic`
   - [ ] List contents: `ls -la`
   - [ ] Verify you see: main.tf, variables.tf, outputs.tf, README.md
   - **Expected result**: You're in the correct deployment directory

3. **Review** variable requirements
   - [ ] Open `variables.tf` and scan for required variables (those without defaults)
   - [ ] Note the `subscription_id` variable (required)
   - [ ] Ask caira-assistant: "Which variables are required vs optional for minimal foundry_basic deployment?"
   - **Validation checkpoint**: Understand that subscription_id is mandatory, location and resource_group_name are optional but recommended

4. **Choose** configuration method
   - [ ] Ask caira-assistant: "Should I create terraform.tfvars or set variables via command line for this deployment?"
   - [ ] Review the assistant's recommendation (tfvars is cleaner for multiple variables)
   - [ ] Decision: Create `terraform.tfvars` file for configuration
   - **Pro tip**: Using tfvars keeps your configuration reusable and version-controllable

5. **Create** terraform.tfvars file
   - [ ] Create file: `touch terraform.tfvars`
   - [ ] Open in editor and add minimum required configuration:

   ```hcl
   subscription_id     = "YOUR_SUBSCRIPTION_ID_HERE"
   location            = "eastus"
   resource_group_name = "rg-caira-foundry-basic-dev"
   ```

   - [ ] Replace `YOUR_SUBSCRIPTION_ID_HERE` with your actual subscription ID (from `az account show --query id -o tsv`)
   - [ ] Save the file
   - **Success check**: terraform.tfvars exists with valid subscription ID and location

### Task 3: Execute Terraform Deployment Workflow (13 minutes)

**What You'll Do**: Run the complete Terraform deployment lifecycle (init → plan → apply) with AI-assisted validation at each checkpoint to safely provision Azure AI Foundry infrastructure.

**Steps**:

1. **Set** ARM_SUBSCRIPTION_ID environment variable
   - [ ] Run: `export ARM_SUBSCRIPTION_ID=$(az account show --query id -o tsv)`
   - [ ] Verify: `echo $ARM_SUBSCRIPTION_ID` (should show your subscription ID)
   - **Pro tip**: Terraform AzureRM provider needs this environment variable for authentication
   - **Expected result**: Environment variable set correctly for current terminal session

2. **Initialize** Terraform
   - [ ] Run: `terraform init`
   - [ ] Observe provider downloads (azurerm, azapi)
   - [ ] Look for: "Terraform has been successfully initialized!"
   - **Validation checkpoint**: No errors about provider configuration or backend initialization
   - **Expected result**: .terraform directory created with providers installed

3. **Generate** and review plan
   - [ ] Run: `terraform plan -out=tfplan`
   - [ ] **CRITICAL**: Read the plan output carefully
   - [ ] Count resources to be created (should be ~7-10 resources for foundry_basic)
   - [ ] Verify resource types include: azurerm_resource_group, azurerm_cognitive_account, azurerm_application_insights
   - [ ] Ask caira-assistant: "Review this terraform plan output - does it look correct for foundry_basic minimal deployment?"
   - **Success check**: Plan shows only "create" actions (no updates or deletes), resource count matches expectation

4. **Apply** the deployment
   - [ ] Run: `terraform apply tfplan`
   - [ ] Observe resource creation progress (takes 3-5 minutes)
   - [ ] Watch for: "Apply complete! Resources: X added, 0 changed, 0 destroyed."
   - **Pro tip**: Don't interrupt the apply process - let it complete fully
   - **Expected result**: Successful deployment with no errors

5. **Capture** deployment outputs
   - [ ] Run: `terraform output`
   - [ ] Note the resource_group_name, ai_project_id, and other outputs
   - [ ] Copy resource_group_name for next task validation
   - **Validation checkpoint**: Outputs show deployed resource identifiers

6. **Verify** in Azure Portal
   - [ ] Open Azure Portal in browser
   - [ ] Navigate to Resource Groups
   - [ ] Find your deployed resource group (e.g., rg-caira-foundry-basic-dev)
   - [ ] Confirm resources exist: Cognitive Services, Log Analytics Workspace, Application Insights
   - **Success check**: All expected resources visible and show "Succeeded" deployment state

## Completion Check

**You've Succeeded When**:

- [ ] Devcontainer opened successfully and all development tools verified
- [ ] Azure CLI authenticated within container environment
- [ ] caira-assistant provided guidance on configuration approach
- [ ] terraform.tfvars created with correct subscription and location settings
- [ ] Terraform initialization completed without errors
- [ ] Terraform plan reviewed and validated before applying
- [ ] Infrastructure deployed successfully (apply completed with 0 errors)
- [ ] All expected resources visible and healthy in Azure Portal
- [ ] Can explain benefits of devcontainer for team standardization
- [ ] Can describe the Terraform deployment workflow (init → plan → apply)

**Next Steps**: Ready to learn troubleshooting? Continue to **Kata 300: Troubleshooting CAIRA Deployments** to handle common deployment issues. Or customize your deployment in **Kata 400: Customizing CAIRA for Your Requirements**.

---

## Reference Appendix

### Help Resources

- **CAIRA Assistant Chatmode**: Use throughout deployment for decision validation and troubleshooting
- **foundry_basic README**: `external/caira/reference_architectures/foundry_basic/README.md` - detailed architecture documentation
- **Terraform Documentation**: Official Terraform CLI command reference
- **Azure AI Foundry Docs**: Microsoft Learn documentation for Azure AI services
- **Dev Containers Documentation**: VS Code development container configuration reference

### Professional Tips

**Devcontainer Benefits for Teams**:

- **Consistent Environment**: Everyone uses identical tool versions (Terraform, Azure CLI, Python)
- **Fast Onboarding**: New team members productive in minutes, not hours
- **Isolated Dependencies**: Container keeps project tools separate from local machine
- **Reproducible Builds**: Same container configuration ensures identical behavior across team

**Terraform Workflow Best Practices**:

```bash
# Always follow this safe deployment pattern:
terraform init      # Install providers and initialize backend
terraform plan      # Preview changes BEFORE applying
terraform apply     # Execute changes after review

# For ongoing changes:
terraform plan -out=tfplan    # Save plan to file
# Review the plan output carefully
terraform apply tfplan        # Apply the reviewed plan
```

**Configuration Management Decision Tree**:

- **Single deployment, few variables** → Command line: `terraform apply -var="subscription_id=..."`
- **Repeatable deployment, multiple variables** → terraform.tfvars file (recommended)
- **Multiple environments** → Separate .tfvars files per environment (dev.tfvars, prod.tfvars)
- **Sensitive values** → Environment variables + .tfvars for non-sensitive
- **Team collaboration** → terraform.tfvars.example (template) + gitignored terraform.tfvars (local)

### Troubleshooting

**Issue**: "Reopen in Container" doesn't appear or fails

- **Check Docker**: Ensure Docker Desktop is running (status bar shows "Engine running")
- **Extension Installed**: Verify Remote - Containers extension is installed in VS Code
- **Docker Resources**: Check Docker Desktop has sufficient memory (8GB+ recommended)
- **Rebuild**: Try Command Palette → "Dev Containers: Rebuild Container"

**Issue**: `az login` fails inside container

- **Browser Access**: Ensure container can open browser for authentication flow
- **Device Code**: Try `az login --use-device-code` for alternate authentication method
- **Corporate Network**: Check if proxy or firewall blocks Azure authentication endpoints
- **Manual Token**: Use `az login --use-device-code` and follow the URL instructions

**Issue**: `terraform init` fails with provider download errors

- **Network Connectivity**: Container needs internet access to download providers from registry.terraform.io
- **Proxy Configuration**: Set HTTP_PROXY and HTTPS_PROXY environment variables if behind proxy
- **Registry Mirror**: Configure Terraform registry mirror if direct access blocked
- **Manual Download**: Download providers manually and place in .terraform directory as last resort

**Issue**: `terraform plan` shows errors about insufficient permissions

- **RBAC Roles**: Verify you have both Contributor AND User Access Administrator roles
- **Subscription Scope**: Ensure roles are assigned at subscription or resource group scope
- **Cached Credentials**: Run `az account clear` then `az login` to refresh authentication
- **ARM_SUBSCRIPTION_ID**: Verify environment variable is set: `echo $ARM_SUBSCRIPTION_ID`

**Issue**: `terraform apply` fails with quota exceeded errors

- **Check Quotas**: Run `az vm list-usage --location eastus -o table` to see current quotas
- **Request Increase**: Submit quota increase request through Azure Portal (Support → Quotas)
- **Different Region**: Try deploying to alternate region with available quota
- **Resource SKUs**: Some regions have limited SKU availability for AI services

**Issue**: Resources deployed but not visible in Azure Portal

- **Subscription Filter**: Check Portal subscription filter includes your deployment subscription
- **Resource Group Name**: Verify you're looking at correct resource group (check terraform output)
- **Refresh Portal**: Click refresh in Portal - newly created resources may take 30-60 seconds
- **Deployment Status**: Check resource group → Deployments to see creation status

---

<!-- markdownlint-disable MD036 -->
*🤖 Crafted with precision by ✨Copilot following brilliant human instruction,
then carefully refined by our team of discerning human reviewers.*
<!-- markdownlint-enable MD036 -->

<!-- Reference Links -->
