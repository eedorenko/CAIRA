---
# Core Metadata
title: "Kata 500: Managing CAIRA Infrastructure Lifecycle"
description: "Build expertise in Terraform state management, safe infrastructure updates, and operational lifecycle management for production CAIRA deployments"
author: HVE Essentials Team
ms.date: 12/18/2024
ms.topic: how-to-guide

# Kata Identity
kata_id: caira-advanced-500
kata_category:
  - caira-advanced
kata_difficulty: 5
estimated_time_minutes: 45

# Learning Content
learning_objectives:
  - Manage infrastructure changes safely using Terraform workflow
  - Handle state management and team collaboration scenarios
  - Execute controlled updates without service disruption
  - Implement backup and disaster recovery for IaC state
  - Document operational procedures for infrastructure lifecycle
technologies:
  - Terraform
  - Azure Storage
  - Azure CLI
  - CAIRA
  - Git
prerequisite_katas:
  - caira-advanced-400
  - caira-advanced-450
success_criteria:
  - Successfully updated deployment without disruption
  - Added new resources using Terraform workflow
  - Implemented remote state storage with locking
  - Backed up and restored state file successfully
  - Updated tags and SKUs without recreating resources
  - Created infrastructure update runbook
  - Documented decommissioning procedures
  - Can explain safe infrastructure change management

# AI Coaching
ai_coaching_level: minimal
scaffolding_level: minimal
hint_strategy: progressive
common_pitfalls:
  - Not running terraform plan before apply (preview changes first)
  - Modifying state file directly instead of using Terraform commands
  - Not backing up state before major changes
  - Forcing unlock without verifying safety
  - Deleting resources without considering data preservation

# Requirements
requires_azure_subscription: true
requires_local_environment: true
requires_dev_container: true

# SEO & Discoverability
tags:
  - terraform-state
  - lifecycle-management
  - infrastructure-updates
  - production-operations
  - state-backup
search_keywords:
  - terraform state management
  - infrastructure lifecycle
  - terraform updates
  - state locking
  - infrastructure decommissioning
  - terraform backup restore
  - CAIRA operations
  - production infrastructure management
  - terraform collaboration
  - remote state storage
---

## Quick Context

**You'll Learn**: How to safely manage the complete lifecycle of CAIRA infrastructure including updates, state management, team collaboration, and controlled decommissioning.

**Real Challenge**: You're a platform engineer at an enterprise organization managing a production CAIRA deployment that has been running for 3 months. You need to add a new GPT-4o model deployment, upgrade AI Search from Basic to Standard tier, update cost center tags for new fiscal year allocation, add Application Insights integration for better monitoring, and plan for quarterly environment refresh while preserving trained model data and conversation history. Your infrastructure team collaborates using shared Terraform state, and you must ensure zero disruption to running AI applications.

**Your Task**: Implement a complete lifecycle management workflow including remote state configuration, safe infrastructure updates, backup procedures, and operational documentation for ongoing CAIRA infrastructure management.

## Essential Setup

**Required** (check these first):

- [ ] Completed Kata 400 (Customizing CAIRA) or Kata 450 (Securing CAIRA)
- [ ] Existing CAIRA deployment (at least 1 week old preferred for realistic lifecycle scenarios)
- [ ] Azure subscription with Contributor and User Access Administrator roles
- [ ] Docker Desktop running for devcontainer access
- [ ] Git repository initialized for version control
- [ ] Understanding of Terraform state concepts

**Quick Validation**: Run `terraform state list` in your CAIRA deployment directory and verify you see deployed resources listed.

> **🤖 Want Interactive AI Coaching?**
>
> Load the **Learning Kata Coach** chat mode for task check-offs, progress tracking, progressive hints, and personalized guidance.
>
> In GitHub Copilot Chat, select **Learning Kata Coach** mode and say:
>
> ```text
> I'm working on Kata 500: Managing CAIRA Infrastructure Lifecycle and want interactive coaching with progress tracking.
> ```

<!-- AI_COACH: This is a legendary-level kata requiring deep operational expertise. Learners should have extensive Terraform experience and production infrastructure management background. When challenges arise, guide toward systematic troubleshooting: backup first, plan changes thoroughly, validate state integrity, document decisions. Encourage thinking about blast radius and rollback procedures. Focus on building professional operational habits and risk mitigation strategies rather than providing direct solutions. -->

## Practice Tasks

### Task 1: Configure Remote State Storage (10 minutes)

**What You'll Do**: Move from local state file to secure remote state storage with locking to enable team collaboration and prevent concurrent modification conflicts.

**Steps**:

1. **Create** Azure Storage Account for remote state
   - [ ] Create storage account with globally unique name in same resource group and region as your CAIRA deployment
   - [ ] Use Azure CLI commands:
     ```bash
     # Set variables (adjust RANDOM_SUFFIX or use your own)
     RANDOM_SUFFIX=$(echo $RANDOM | md5 | head -c 6)
     STORAGE_ACCOUNT="sttfcairastate${RANDOM_SUFFIX}"
     RESOURCE_GROUP="<your-caira-resource-group>"  # e.g., rg-basic-lfnfj
     LOCATION="<your-caira-location>"              # e.g., eastus

     # Create storage account (Standard_LRS is sufficient for state files)
     az storage account create \
       --name "${STORAGE_ACCOUNT}" \
       --resource-group "${RESOURCE_GROUP}" \
       --location "${LOCATION}" \
       --sku Standard_LRS \
       --kind StorageV2 \
       --allow-blob-public-access false \
       --min-tls-version TLS1_2

     # Create blob container for state files
     az storage container create \
       --name tfstate \
       --account-name "${STORAGE_ACCOUNT}" \
       --auth-mode login

     # Enable versioning for state history tracking (recommended)
     az storage account blob-service-properties update \
       --account-name "${STORAGE_ACCOUNT}" \
       --resource-group "${RESOURCE_GROUP}" \
       --enable-versioning true

     # Display storage account details for backend configuration
     echo "Storage Account: ${STORAGE_ACCOUNT}"
     echo "Container: tfstate"
     echo "Resource Group: ${RESOURCE_GROUP}"
     ```
   - [ ] Note storage account name, container name, and resource group
   - **Pro tip**: Modern Azure subscriptions often disable shared key access by policy. Terraform backend supports Azure AD authentication (`use_azuread_auth = true`) which is more secure and doesn't require managing access keys
   - [ ] **Expected result**: Storage account and container created, versioning enabled

2. **Configure** Terraform backend for remote state
   - [ ] Back up existing local `terraform.tfstate` file to safe location:
     ```bash
     cp terraform.tfstate terraform.tfstate.backup-$(date +%Y%m%d-%H%M%S)
     ```
   - [ ] Add backend configuration to `terraform.tf` file in your CAIRA deployment (after the `terraform {` block, before `required_providers`):
     ```hcl
     backend "azurerm" {
       resource_group_name  = "<your-resource-group>"     # e.g., "rg-basic-lfnfj"
       storage_account_name = "<your-storage-account>"    # e.g., "sttfcairastate32df8e"
       container_name       = "tfstate"
       key                  = "caira-foundry-basic.tfstate"
       use_azuread_auth     = true                        # Recommended: Uses Azure AD instead of access keys
     }
     ```
   - [ ] Note: If your subscription allows shared key access and you prefer access key authentication, set `use_azuread_auth = false` and add `access_key` parameter (retrieve key with `az storage account keys list`)
   - [ ] Grant yourself Storage Blob Data Contributor role for Azure AD authentication:
     ```bash
     USER_OBJECT_ID=$(az ad signed-in-user show --query id -o tsv)
     STORAGE_ACCOUNT="<your-storage-account>"
     RESOURCE_GROUP="<your-resource-group>"

     az role assignment create \
       --role "Storage Blob Data Contributor" \
       --assignee "${USER_OBJECT_ID}" \
       --scope "/subscriptions/$(az account show --query id -o tsv)/resourceGroups/${RESOURCE_GROUP}/providers/Microsoft.Storage/storageAccounts/${STORAGE_ACCOUNT}"
     ```
   - **Validation checkpoint**: Does your backend configuration include resource_group_name, storage_account_name, container_name, key, and use_azuread_auth?
   - [ ] **Expected result**: Backend configuration block present in terraform.tf with Azure AD authentication enabled

3. **Migrate** state to remote backend
   - [ ] Wait 30-60 seconds after granting RBAC permissions (Azure role assignments take time to propagate)
   - [ ] Run `terraform init -migrate-state` to move local state to Azure Storage
   - [ ] If migration doesn't prompt (happens when remote backend is empty), use alternative method:
     ```bash
     # Remove cached backend config to force re-initialization
     rm -rf .terraform

     # Re-initialize (this will configure the backend)
     terraform init

     # Push local state to remote backend
     terraform state push terraform.tfstate
     ```
   - [ ] Verify state file now exists in Azure Storage:
     ```bash
     az storage blob list \
       --container-name tfstate \
       --account-name "<your-storage-account>" \
       --auth-mode login \
       --query "[].{name:name, size:properties.contentLength}" \
       -o table
     ```
   - **Success check**: State file appears in Azure Storage container and `terraform state list` works correctly
   - [ ] **Expected result**: Terraform operations now use remote state storage with locking enabled

### Task 2: Execute Safe Infrastructure Updates (15 minutes)

**What You'll Do**: Add new resources and update existing configurations using Terraform's safe change workflow with preview, validation, and controlled application.

**Steps**:

1. **Plan** infrastructure changes
   - [ ] Update cost center tags in `terraform.tfvars` for new fiscal year:
     ```hcl
     tags = {
       cost_center = "CC-FY2026"  # Changed from CC-12345
       department  = "AI Platform"
       environment = "production"
     }
     ```
   - [ ] Add GPT-4o model deployment to your CAIRA project in `main.tf`:
     ```hcl
     model_deployments = [
       module.common_models.gpt_4_1,
       module.common_models.o4_mini,
       module.common_models.gpt_4o,      # NEW: Adding GPT-4o
       module.common_models.text_embedding_3_large
     ]
     ```
   - [ ] **IMPORTANT**: Do NOT change the `location` value in terraform.tfvars - changing region causes full resource replacement
   - [ ] Run `terraform plan -out=tfplan` to preview all changes
   - [ ] Review plan output carefully - identify which resources will be modified vs recreated
   - **Validation checkpoint**: The plan should show tag updates (in-place modifications) and one new model deployment (creation). If you see resources being replaced or destroyed, investigate before proceeding.
   - [ ] **Expected result**: Plan shows ~10 resources with updated tags, 1 new model deployment created, 0 resources destroyed

2. **Apply** changes with controlled execution
   - [ ] Apply the saved plan using `terraform apply tfplan`
   - [ ] Monitor apply progress and watch for any unexpected errors
   - [ ] Verify changes completed successfully without service disruption
   - **Pro tip**: For production changes, consider using `terraform apply -target` to update specific resources individually
   - [ ] **Expected result**: All planned changes applied successfully - tags updated, new model available

3. **Validate** updated infrastructure
   - [ ] Verify new GPT-4o model deployment appears in Azure AI Foundry portal
   - [ ] Check that existing model deployments (gpt-4.1, o4-mini, text-embedding-3-large) retained their configurations
   - [ ] Validate cost center tags updated across all resources:
     ```bash
     az resource list -g <your-resource-group> --query "[].{name:name, tags:tags.cost_center}" -o table
     ```
   - **Success check**: Running AI applications continue functioning without interruption
   - [ ] **Expected result**: Infrastructure updated with zero downtime - new model available, tags updated, existing services unaffected

### Task 3: Implement State Backup and Recovery (10 minutes)

**What You'll Do**: Create backup procedures and practice state recovery to protect against state corruption or accidental deletions.

**Steps**:

1. **Establish** backup procedures
   - [ ] Create backup directory and download current state file from Azure Storage:
     ```bash
     mkdir -p .backups
     TIMESTAMP=$(date +%Y%m%d-%H%M%S)

     az storage blob download \
       --container-name tfstate \
       --account-name <your-storage-account> \
       --name caira-foundry-basic.tfstate \
       --file ".backups/terraform.tfstate.backup-${TIMESTAMP}" \
       --auth-mode login

     echo "Backup saved to: .backups/terraform.tfstate.backup-${TIMESTAMP}"
     ```
   - [ ] Verify backup file size matches expected state file size (~170KB for foundry_basic)
   - [ ] Document backup procedure in team runbook (frequency, retention, restoration steps)
   - **Pro tip**: Azure Storage blob versioning provides automatic state history, but manual backups add another recovery layer
   - [ ] **Expected result**: State backup file saved locally with timestamp and procedure documented

2. **Implement** automated state backup script
   - [ ] Create `backup-terraform-state.sh` script with error handling:
     ```bash
     #!/bin/bash
     set -e  # Exit on error

     # Configuration
     STORAGE_ACCOUNT="${TF_STATE_STORAGE_ACCOUNT:-<your-storage-account>}"
     CONTAINER_NAME="${TF_STATE_CONTAINER:-tfstate}"
     STATE_FILE_NAME="${TF_STATE_KEY:-caira-foundry-basic.tfstate}"
     BACKUP_DIR=".backups"
     TIMESTAMP=$(date +%Y%m%d-%H%M%S)

     mkdir -p "${BACKUP_DIR}"

     # Check Azure authentication
     if ! az account show &>/dev/null; then
         echo "Error: Not authenticated with Azure CLI"
         exit 1
     fi

     # Download state file
     az storage blob download \
         --container-name "${CONTAINER_NAME}" \
         --account-name "${STORAGE_ACCOUNT}" \
         --name "${STATE_FILE_NAME}" \
         --file "${BACKUP_DIR}/terraform.tfstate.backup-${TIMESTAMP}" \
         --auth-mode login

     echo "Backup successful: ${BACKUP_DIR}/terraform.tfstate.backup-${TIMESTAMP}"
     ```
   - [ ] Make script executable: `chmod +x backup-terraform-state.sh`
   - [ ] Test script execution: `./backup-terraform-state.sh`
   - **Validation checkpoint**: Does your backup script handle authentication errors and verify file creation?
   - [ ] **Expected result**: Reusable backup script available for team with proper error handling

3. **Practice** state recovery scenario
   - [ ] List available backups to choose from:
     ```bash
     ls -lh .backups/
     ```
   - [ ] Test state recovery by pushing a backup to remote backend:
     ```bash
     # Choose a backup file
     BACKUP_FILE=".backups/terraform.tfstate.backup-<timestamp>"

     # Upload backup to Azure Storage (simulating recovery)
     az storage blob upload \
       --container-name tfstate \
       --account-name <your-storage-account> \
       --name caira-foundry-basic.tfstate \
       --file "${BACKUP_FILE}" \
       --auth-mode login \
       --overwrite
     ```
   - [ ] Verify state operations work after recovery: `terraform state list`
   - [ ] Run `terraform plan` to ensure no unexpected changes (drift check)
   - **Success check**: Can you restore state and terraform plan shows "No changes" (if infrastructure hasn't changed)?
   - [ ] **Expected result**: State recovery procedure validated - can restore from backup and resume operations

### Task 4: Handle State Locking and Collaboration (5 minutes)

**What You'll Do**: Understand and resolve state locking scenarios that occur in multi-user team environments.

**Steps**:

1. **Simulate** state lock scenario
   - [ ] Open a second terminal window in your deployment directory
   - [ ] In **Terminal 1**, start a long-running operation:
     ```bash
     terraform plan
     ```
   - [ ] Immediately in **Terminal 2**, attempt another operation while Terminal 1 is still running:
     ```bash
     terraform plan
     ```
   - [ ] Observe the lock error message in Terminal 2. Note the lock information:
     - Lock ID
     - Lock holder identity
     - Lock acquisition timestamp
     - Storage blob lease details
   - **Validation checkpoint**: The error message should show something like: `Error acquiring the state lock: ConditionalCheckFailedException` or `Error: Error locking state: Error acquiring the state lock`
   - [ ] **Expected result**: Terminal 2 blocked by lock, displaying who holds the lock and when it was acquired

2. **Resolve** lock conflict safely
   - [ ] Let the first operation (Terminal 1) complete naturally, or press Ctrl+C to cancel it
   - [ ] Verify lock is released by checking Terminal 2 can now proceed:
     ```bash
     terraform plan
     ```
   - [ ] View lock information in Azure Storage (optional):
     ```bash
     az storage blob show \
       --container-name tfstate \
       --account-name <your-storage-account> \
       --name caira-foundry-basic.tfstate \
       --auth-mode login \
       --query "properties.lease"
     ```
   - [ ] Document force-unlock emergency procedure (use ONLY when lock holder crashed):
     ```bash
     # EMERGENCY ONLY - Verify process truly terminated first
     terraform force-unlock <LOCK_ID>
     ```
   - **Pro tip**: Before force-unlock, always verify the process holding the lock has actually crashed using `ps aux | grep terraform`. Force-unlock during an active operation can corrupt state
   - [ ] **Expected result**: Lock resolved naturally without force-unlock, understanding when emergency force-unlock is appropriate

### Task 5: Plan Infrastructure Decommissioning (5 minutes)

**What You'll Do**: Create procedures for controlled environment tear-down with data preservation considerations.

**Steps**:

1. **Document** decommissioning checklist
   - [ ] Identify resources with persistent data in your CAIRA deployment:
     ```bash
     # List all resources to identify data-containing services
     terraform state list

     # Check for storage accounts, databases, and AI Search indexes
     az resource list -g <your-resource-group> \
       --query "[?type=='Microsoft.Storage/storageAccounts' || type=='Microsoft.Search/searchServices'].{name:name, type:type}" \
       -o table
     ```
   - [ ] Document data preservation requirements:
     - **AI Models**: Deployed models (gpt-4.1, o4-mini, gpt-4o, text-embedding-3-large) - configurations stored in code
     - **AI Search**: Indexes, data sources, and vector embeddings - requires backup if populated
     - **Application Insights**: Telemetry and logs - consider retention requirements
     - **Log Analytics**: Historical operational data - export if needed for compliance
     - **State Files**: Already backed up in Task 3, retain for audit/recovery
   - [ ] Document application dependencies:
     - Applications using AI Foundry endpoints must be reconfigured
     - API keys/connection strings need rotation in dependent services
     - DNS/firewall rules may reference infrastructure resources
   - **Validation checkpoint**: Have you identified all data sources and documented export/backup procedures for each?
   - [ ] **Expected result**: Comprehensive pre-decommissioning checklist with data backup plan

2. **Create** controlled tear-down procedure
   - [ ] Document phased destruction approach:
     ```bash
     # Phase 1: Remove model deployments first (fastest to recreate if needed)
     terraform destroy -target=module.ai_foundry.azurerm_cognitive_deployment.model_deployments

     # Phase 2: Remove AI Foundry Project (preserves hub for other projects)
     terraform destroy -target=module.default_project.azapi_resource.ai_foundry_project

     # Phase 3: Remove AI Foundry Hub and dependent resources
     terraform destroy -target=module.ai_foundry

     # Phase 4: Remove supporting infrastructure (App Insights, Log Analytics)
     terraform destroy -target=module.application_insights

     # Phase 5: Final cleanup (Resource Group if managed by Terraform)
     terraform destroy
     ```
   - [ ] Document data export commands before each phase:
     ```bash
     # Example: Export AI Search indexes before destruction
     # Example: Download Application Insights data for compliance
     # Example: Archive Log Analytics workspace data
     ```
   - **Pro tip**: Use `terraform destroy -target` to remove resources incrementally, allowing validation between phases. This provides rollback points if you discover missed dependencies
   - [ ] **Expected result**: Step-by-step decommissioning runbook with data preservation steps

3. **Test** preview-only destruction
   - [ ] Run `terraform plan -destroy` to preview complete destruction:
     ```bash
     terraform plan -destroy -out=destroy.tfplan
     ```
   - [ ] Review destroy plan output carefully:
     - Count of resources to be destroyed
     - Identify any resources with `prevent_destroy` lifecycle rules
     - Check for dependencies between resources
     - Verify no unexpected resources (resources not in your configuration)
   - [ ] Document resources requiring manual cleanup:
     - DNS records pointing to infrastructure
     - Application configurations with hardcoded endpoints
     - Firewall rules allowing access to infrastructure
     - Service principals or managed identities granted access
   - **Success check**: Destroy plan shows expected ~23 resources for foundry_basic (or your current count from `terraform state list | wc -l`)
   - [ ] **Expected result**: Complete understanding of destruction scope and manual cleanup requirements

## Completion Check

**You've Succeeded When**:

- [ ] Remote state storage configured with locking enabled
- [ ] Successfully applied infrastructure updates without service disruption
- [ ] State backup and recovery procedures implemented and tested
- [ ] Created operational runbooks for updates and decommissioning
- [ ] Can explain safe change management workflow to team members
- [ ] Understand state locking behavior and conflict resolution
- [ ] Have documented procedures for quarterly environment refresh

**Next Steps**: You've gained expertise in operational infrastructure lifecycle management. Consider exploring:

- **Multi-environment management** with Terraform workspaces
- **CI/CD integration** for automated Terraform deployments
- **Advanced state management** with terragrunt for complex scenarios
- **Contributing back** to CAIRA repository with operational improvements

---

## Reference Appendix

### Help Resources

- **Remote State Documentation**: [Terraform Backend Configuration](https://developer.hashicorp.com/terraform/language/settings/backends/azurerm)
- **Azure AD Authentication**: [Authenticating using Azure AD](https://developer.hashicorp.com/terraform/language/settings/backends/azurerm#authenticating-using-azure-ad)
- **State Management Guide**: [Terraform State](https://developer.hashicorp.com/terraform/language/state)
- **CAIRA Repository**: Review deployment documentation for operational best practices
- **Azure Storage**: [Blob versioning for state history](https://learn.microsoft.com/azure/storage/blobs/versioning-overview)

### Professional Tips

- Always run `terraform plan` before `apply` - no exceptions for production infrastructure
- Verify `location` in terraform.tfvars matches actual deployed region before making changes to avoid accidental full replacement
- Backup state before major changes (version upgrades, large-scale modifications)
- Test remote state configuration in non-production environments first before applying to production
- Use Azure AD authentication (`use_azuread_auth = true`) for better security and audit trails
- Grant RBAC permissions before running Terraform operations and allow time for propagation
- Use descriptive commit messages when state changes are committed to version control
- Document infrastructure change approvals and rollback procedures
- Consider implementing approval gates for production apply operations
- Regular state file backups should be automated and tested for recoverability
- Leverage blob versioning in Azure Storage to maintain state history for rollback scenarios
- When adding models, verify they're available in your deployment region with `az cognitiveservices model list`
- Schedule automated state backups before major changes (cron job or CI/CD pipeline integration)
- Test state recovery procedures regularly in non-production environments to validate your runbooks
- Keep backup retention policy aligned with your disaster recovery requirements (30-90 days typical)
- Communicate with team before long-running Terraform operations to avoid lock contention
- Use descriptive commit messages and PR descriptions when making infrastructure changes
- Consider implementing a Terraform operation queue or coordination tool for large teams (Terraform Cloud, Atlantis)
- Monitor state lock duration - operations taking >15 minutes may indicate performance issues or large changesets

### Troubleshooting

**Issue**: State lock won't release after crashed operation

- **Quick Fix**: Verify process truly terminated (`ps aux | grep terraform`), then check if there's an active lease on the state blob with `az storage blob show --container-name tfstate --account-name <storage-account> --name <state-file> --query "properties.lease"`. If the lease shows "locked" but no process exists, carefully use `terraform force-unlock <LOCK_ID>` only after confirming no active operations. The lock ID is shown in the error message

**Issue**: Cannot acquire state lock - another user has it

- **Quick Fix**: This is normal team collaboration behavior. Wait for the other operation to complete, or communicate with your team to coordinate. Check who has the lock from the error message (shows operation ID and timestamp). If urgent and confirmed the other process crashed, the lock holder can run `terraform force-unlock <LOCK_ID>` or contact the Azure admin to break the blob lease

**Issue**: State file shows drift from actual Azure resources

- **Quick Fix**: Run `terraform refresh` to sync state with reality, then investigate what caused the drift (manual changes? external automation?)

**Issue**: Backend initialization fails with authentication errors

- **Quick Fix**: If using Azure AD auth (`use_azuread_auth = true`), verify Azure CLI authentication is active (`az account show`) and you have Storage Blob Data Contributor role on the storage account. Wait 30-60 seconds after granting role assignment for RBAC propagation. If shared key access is disabled by policy, Azure AD auth is required. For access key auth, retrieve key with `az storage account keys list --account-name <name> --resource-group <rg> --query "[0].value" -o tsv`

**Issue**: Terraform init doesn't prompt for state migration

- **Quick Fix**: This happens when the remote backend is empty and Terraform doesn't detect existing local state to migrate. Use `terraform state push terraform.tfstate` after initialization to manually upload your local state to the remote backend, or remove `.terraform` directory and run `terraform init` again

**Issue**: Plan shows unexpected resource replacements

- **Quick Fix**: Review the plan output carefully for attribute changes marked "forces replacement" - these are immutable properties requiring resource recreation. Common culprits: changing `location` (region), changing resource names, or modifying SKU types that don't support in-place updates. If you see unwanted replacements, check if `location` in terraform.tfvars matches the actual deployed region

**Issue**: Changing location in terraform.tfvars causes all resources to be replaced

- **Quick Fix**: The `location` variable determines where resources are deployed. Changing it from the original deployment location (e.g., `eastus` to `canadacentral`) forces Terraform to destroy and recreate all location-dependent resources. Verify your `location` value matches the actual deployed region with `az group show -n <resource-group> --query location`

**Issue**: State backup script fails with authentication errors

- **Quick Fix**: Ensure Azure CLI is authenticated (`az account show`) and you have Storage Blob Data Contributor role on the storage account. The script uses `--auth-mode login` which requires Azure AD authentication. Wait 30-60 seconds after granting permissions for RBAC propagation

**Issue**: State recovery doesn't restore infrastructure correctly

- **Quick Fix**: After restoring state from backup, run `terraform plan` to identify any drift between state and actual infrastructure. If infrastructure changed since backup, you may need to either: 1) Apply changes to match state, 2) Use `terraform import` to update state for manually changed resources, or 3) Use `terraform refresh` to sync state with current reality

**Issue**: Terraform destroy fails with dependency errors

- **Quick Fix**: Use `terraform destroy -target` to remove dependent resources first, or investigate resource protection policies preventing deletion

---

<!-- markdownlint-disable MD036 -->
*🤖 Crafted with precision by ✨Copilot following brilliant human instruction,
then carefully refined by our team of discerning human reviewers.*
<!-- markdownlint-enable MD036 -->
