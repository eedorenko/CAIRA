---
# Core Metadata
title: "Kata: Securing CAIRA for Production"
description: Deploy CAIRA with zero-trust security, private networking, managed identities, and comprehensive security hardening for production workloads.
author: HVE Essentials Team
ms.date: 12/15/2024
ms.topic: how-to-guide

# Kata Identity
kata_id: caira-advanced-450-securing-production
kata_category:
  - caira-advanced
kata_difficulty: 4
estimated_time_minutes: 75

# Learning Content
learning_objectives:
  - Deploy foundry_standard_private architecture with complete network isolation
  - Configure private endpoints and disable public network access across all services
  - Implement managed identities and RBAC with least-privilege access principles
  - Document security decisions using Architecture Decision Record format
  - Validate security posture against Azure Security Benchmark recommendations
prerequisite_katas:
  - caira-fundamentals-200-devcontainer-foundry-basic-deployment
technologies:
  - CAIRA
  - Azure AI Foundry
  - Azure Private Link
  - Managed Identities
  - Azure RBAC
  - Network Security Groups
  - Azure Key Vault
  - Terraform
success_criteria:
  - Private architecture deployed with no public endpoints exposed
  - All services using private endpoints and managed identities exclusively
  - API key authentication disabled across all AI services
  - RBAC configured with least-privilege role assignments
  - Security hardening checklist completed with validation
  - Architecture Decision Record documenting all security choices

# AI Coaching
ai_coaching_level: guided
scaffolding_level: medium-heavy
hint_strategy: progressive
common_pitfalls:
  - Forgetting to configure private DNS zones for private endpoints (services become unreachable)
  - Overly permissive RBAC assignments (violates least-privilege principle)
  - Not documenting security trade-offs and decisions in ADR format
  - Incomplete private endpoint coverage (missing a service leaves security gap)
  - Deploying without first reviewing terraform plan for public endpoint creation

# Requirements
requires_azure_subscription: true
requires_local_environment: true
requires_dev_container: true

# Optional Fields
related_katas:
  - caira-fundamentals-200-devcontainer-foundry-basic-deployment
  - caira-fundamentals-400-customizing-caira-requirements
  - caira-fundamentals-500-managing-caira-lifecycle
chatmode_references:
  - caira-assistant
  - learning-kata-coach
file_references:
  - reference_architectures/foundry_standard_private/main.tf
  - reference_architectures/foundry_standard_private/variables.tf
  - reference_architectures/foundry_standard_private/terraform.tfvars
validation_commands:
  - terraform plan
  - terraform apply
  - az resource list --resource-group <rg-name> --query "[].{Name:name, Type:type}" -o table

# SEO & Discoverability
tags:
  - caira
  - security
  - production
  - private-networking
  - managed-identity
  - zero-trust
  - rbac
search_keywords:
  - CAIRA production security
  - Azure private endpoints
  - managed identities
  - zero-trust architecture
  - network isolation
  - RBAC least privilege
  - security hardening checklist
  - ADR security documentation
---

## Quick Context

### You'll Learn

- Deploy and configure CAIRA's foundry_standard_private architecture with complete network isolation
- Implement zero-trust security principles using private endpoints and managed identities
- Configure least-privilege RBAC and disable API key authentication
- Document security architecture decisions in ADR format
- Validate security posture against Azure Security Benchmark

### Prerequisites

- Completed [Kata 200: Devcontainer & Foundry Basic Deployment](./200-devcontainer-foundry-basic-deployment.md)
- Azure subscription with Contributor + User Access Administrator roles
- Docker Desktop installed and running
- Basic understanding of Azure networking concepts (VNets, subnets, private endpoints)
- Familiarity with zero-trust security principles (helpful but not required)

### Real Challenge

You're a platform engineer at a healthcare technology company building AI-powered clinical decision support tools. Your enterprise security team requires all AI infrastructure to follow zero-trust principles for HIPAA compliance: no public endpoints, managed identities only (no API keys), complete network isolation, encryption at rest and in transit, least-privilege RBAC, and comprehensive audit logging. You must deploy CAIRA's foundry_standard_private architecture and document all security decisions for the compliance review board.

**Note**: This kata uses role-based scenarios without company names to maintain focus on technical learning objectives.

## Essential Setup

**Required** (check these first):

- [ ] Docker Desktop running
- [ ] VS Code with Dev Containers extension installed
- [ ] Azure CLI authentication active (run `az login` and `az account show`)
- [ ] Azure subscription with both Contributor and User Access Administrator roles
- [ ] CAIRA repository cloned locally
- [ ] Completed Kata 200 (basic deployment understanding)

**Quick Validation**: Open terminal and run `az account show`. You should see your subscription details with sufficient permissions.

> **🤖 Want Interactive AI Coaching?**
>
> Load the **Learning Kata Coach** chat mode for task check-offs, progress tracking, progressive hints, and personalized guidance.
>
> In GitHub Copilot Chat, select **Learning Kata Coach** mode and say:
>
> ```text
> I'm working on Kata 450: Securing CAIRA for Production and want interactive coaching with progress tracking.
> ```

## Practice Tasks

### Task 1: Prepare Prerequisites (5-15 minutes)

**What You'll Do**: Ensure you have the required prerequisite resources for foundry_standard_private architecture: VNet with subnets, Cosmos DB, Storage Account, and AI Search.

**Two Paths**:

- **Path A**: You already have these resources → Gather their IDs and skip to Task 2
- **Path B**: You need to create these resources → Use the prerequisites terraform folder (15 min)

**Why Prerequisites Matter**: The foundry_standard_private architecture requires existing networking (VNet + subnets) and capability host resources (Cosmos DB, Storage, AI Search) to deploy AI Foundry with private networking and agent capabilities.

**Choose Your Path**:

#### Path A: Using Existing Resources (5 minutes)

If you already have VNet, subnets, Cosmos DB, Storage Account, and AI Search:

**Steps**:

1. **Gather** existing resource information
   - [ ] Get VNet subnet IDs:

     ```bash
     az network vnet subnet show --resource-group <rg-name> --vnet-name <vnet-name> --name <foundry-subnet-name> --query id -o tsv
     az network vnet subnet show --resource-group <rg-name> --vnet-name <vnet-name> --name <agents-subnet-name> --query id -o tsv
     ```

   - [ ] Get capability host resource names: Cosmos DB account name, Storage account name, AI Search service name
   - [ ] Get resource group ID containing those resources
   - [ ] **Expected result**: Have 6 values ready (2 subnet IDs, 3 resource names, 1 RG ID)

2. **Verify** security settings (optional but recommended for production)
   - [ ] Check if Cosmos DB has public access disabled: `az cosmosdb show --name <name> --resource-group <rg> --query publicNetworkAccess`
   - [ ] Check Storage Account: `az storage account show --name <name> --resource-group <rg> --query publicNetworkAccess`
   - [ ] Check AI Search: `az search service show --name <name> --resource-group <rg> --query publicNetworkAccess`
   - **Pro tip**: If public access is enabled, consider whether this meets your production security requirements. Kata focuses on AI Foundry security, but capability hosts should also be secured for true zero-trust.
   - [ ] **Expected result**: Awareness of existing resource security posture

3. **Skip** to Task 2 with your gathered values

---

#### Path B: Deploy Prerequisites (15 minutes)

If you need to create VNet, subnets, and capability host resources for this kata:

**Why Separate Deployment?**: Terraform can't evaluate computed subnet IDs during planning. Deploying prerequisites separately is a **production pattern** for separating infrastructure concerns - not a workaround!

**Steps**:

1. **Open** CAIRA repository in devcontainer
   - [ ] Launch VS Code, open CAIRA repository folder
   - [ ] Click "Reopen in Container" when prompted (or Command Palette → "Dev Containers: Reopen in Container")
   - [ ] Wait for container build to complete
   - [ ] **Expected result**: Terminal shows prompt inside devcontainer environment

2. **Navigate** to prerequisites directory
   - [ ] In terminal, run: `cd reference_architectures/foundry_standard_private/prerequisites`
   - [ ] Verify you're in correct directory: `pwd` should show `/workspaces/CAIRA/reference_architectures/foundry_standard_private/prerequisites`
   - [ ] List files: `ls -la` - you should see `main.tf`, `variables.tf`, `terraform.tfvars`, `outputs.tf`
   - [ ] **Expected result**: Prerequisites terraform workspace ready

3. **Review** prerequisites configuration
   - [ ] Open `terraform.tfvars` in editor
   - [ ] Confirm `location = "eastus"` (or change to your preferred region)
   - [ ] Optional: Set `capability_host_location = "westus2"` if you want split-region deployment for capacity resilience
   - [ ] Review tags - customize if needed for your organization
   - [ ] Open `main.tf` to understand what will be created:
     - Virtual Network (10.0.0.0/16) in primary region
     - Two subnets: foundry (10.0.1.0/24) and agents (10.0.2.0/24)
     - Cosmos DB with `public_network_access_enabled = false` ✅ Secure!
     - Storage Account with `shared_access_key_enabled = false` (Azure AD auth) ✅ Even more secure!
     - AI Search with `public_network_access_enabled = false` ✅ Secure!
     - Private DNS zones for AI Foundry (cognitive services, AI services, OpenAI)
     - VNet links connecting DNS zones to VNet
   - **Pro tip**: Notice all capability host resources have public access DISABLED and Storage uses Azure AD authentication instead of keys - this is maximum security hardening for production!
   - [ ] **Expected result**: Understanding of complete secure infrastructure including private DNS zones

4. **Initialize** prerequisites Terraform workspace

---

### Task 2: Deploy AI Foundry with Private Networking (20our-subscription-id"` (get from `az account show`)

- [ ] Run: `terraform init`
- [ ] Review output for successful provider downloads
- **Validation checkpoint**: Did initialization complete without errors?
- [ ] **Expected result**: Message "Terraform has been successfully initialized!"

5. **Generate** and review prerequisites plan
   - [ ] Run: `terraform plan`
   - [ ] Review resources being created:
     - Resource group for prerequisites
     - VNet and 2 subnets (foundry & agents)
     - Cosmos DB, Storage Account, AI Search
   - [ ] Verify security settings: `public_network_access_enabled = false` on all capability host resources
   - **Validation checkpoint**: Should see ~8-10 resources to create
   - [ ] **Expected result**: Clean plan with secure resources

6. **Deploy** prerequisites infrastructure
   - [ ] Run: `terraform apply`
   - [ ] Type `yes` when prompted to confirm
   - [ ] Monitor deployment progress (typically 10-15 minutes)
   - **Pro tip**: Cosmos DB, AI Search, and Private DNS zones take the longest to deploy
   - **Note**: If deployment is interrupted, run `terraform apply` again to complete. DNS zones may need to be imported if they were partially created.
   - [ ] **Expected result**: "Apply complete!" message with ~14 resources added
   - **What Gets Created**:
     - Resource group (e.g., `rg-kata450-prereqs-<random>`)
     - VNet with 2 subnets (foundry, agents)
     - Cosmos DB account (NoSQL API, serverless, public access disabled)
     - Storage Account (Azure AD auth, no shared keys, public access disabled)
     - AI Search service (basic SKU, public access disabled)
     - 3 Private DNS zones (cognitive services, AI services, OpenAI)
     - 3 VNet links (connecting DNS zones to VNet)
     - 2 Network Security Groups (one per subnet)

7. **Capture** output values for main deployment
   - [ ] Run: `terraform output copy_to_main_tfvars`
   - [ ] Copy the entire output block - this contains properly formatted variables for main deployment
   - [ ] The output provides these key values:
     - `foundry_subnet_id` - Full Azure resource ID for AI Foundry subnet
     - `agents_subnet_id` - Full Azure resource ID for agents subnet
     - `existing_capability_host_resource_group_id` - Resource group containing Cosmos/Storage/Search
     - `existing_cosmosdb_account_name` - Name of Cosmos DB account
     - `existing_storage_account_name` - Name of Storage account
     - `existing_search_service_name` - Name of AI Search service
   - [ ] Save to a temporary file or note-taking app (you'll paste into main terraform.tfvars in Task 2)
   - [ ] **Expected result**: Full output block ready to use in main deployment configuration

8. **Verify** prerequisites deployment in Azure Portal (optional but recommended)
   - [ ] Navigate to Azure Portal → Resource Groups
   - [ ] Find your prerequisites resource group (e.g., `rg-kata450-prereqs-<random>`)
   - [ ] Verify all resources show "Succeeded" status
   - [ ] Click on Storage Account → verify "Public network access" is disabled
   - [ ] Click on Cosmos DB → verify "Public network access" is disabled
   - [ ] Click on AI Search → verify "Public network access" is disabled
   - [ ] **Expected result**: All prerequisite resources healthy with security settings confirmed

### Task 2: Deploy AI Foundry with Private Networking (35-50 minutes)

**What You'll Do**: Deploy AI Foundry and Project on top of prerequisite infrastructure. This deployment adds private endpoints, managed identities, RBAC, and configures zero-trust security.

> **⚠️ Important Timing Note**: AI Foundry with agent network injection takes **35-45 minutes** to provision. Terraform has a 30-minute default timeout, so you'll likely see a timeout error even though the deployment is still progressing in Azure. **This is expected behavior** - don't worry! We'll walk you through how to handle this

**Steps**:

1. **Navigate** to main deployment directory
   - [ ] In terminal, run: `cd /workspace/reference_architectures/foundry_standard_private`
   - [ ] Verify you're in correct directory: `pwd` should show `/workspaces/CAIRA/reference_architectures/foundry_standard_private`
   - [ ] **Expected result**: Main terraform workspace directory

2. **Configure** main deployment with prerequisite values
   - [ ] Create file: `terraform.tfvars`
   - [ ] Add prerequisite resource values:
     - **If you used Path A** (existing resources): Use the IDs/names you gathered
     - **If you used Path B** (deployed prerequisites): Paste the output from `terraform output copy_to_main_tfvars`utputs
   - [ ] Create file: `terraform.tfvars`
   - [ ] Paste the output values from Task 1, Step 7
   - [ ] Add additional configuration:

     ```hcl
     location = "eastus"  # Match prerequisites location

     tags = {
       Environment = "Production"
       Project     = "CAIRA-Kata-450"
       ManagedBy   = "Terraform"
     }
     ```

   - [ ] Verify subnet IDs and resource names match prerequisite outputs
   - **Validation checkpoint**: terraform.tfvars should have 6 prerequisite values plus location and tags
   - [ ] **Expected result**: Complete configuration ready for deployment

3. **Initialize** main Terraform workspace
   - [ ] Ensure subscription is set: `export ARM_SUBSCRIPTION_ID="your-subscription-id"`
   - [ ] Run: `terraform init`
   - [ ] Review output for module downloads (ai_foundry, ai_foundry_project, common_models)
   - [ ] **Expected result**: Successful initialization with modules

4. **Generate** and review main deployment plan
   - [ ] Run: `terraform plan -out=tfplan`
   - [ ] Review plan output carefully for security indicators:
     - Private endpoints for AI Foundry resources
     - Managed identity resources being created
     - RBAC role assignments
     - Private DNS zone links
     - NO resources with `public_network_access = "Enabled"`
   - [ ] Use `caira-assistant` to review: "Review my terraform plan for security best practices"
   - **Validation checkpoint**: Should see ~40-50 resources to create, all using private networking
   - [ ] **Expected result**: Clean plan with secure AI Foundry architecture

5. **Deploy** AI Foundry infrastructure
   - [ ] Run: `terraform apply tfplan`
   - [ ] Monitor deployment progress - **this takes 35-45 minutes** due to agent network injection
   - **⚠️ Terraform Timeout Expected**: After ~30 minutes, Terraform will show a timeout error like `context deadline exceeded`. **This is NOT a failure!** The AI Foundry resource continues provisioning in Azure.
   - [ ] If you see the timeout error, proceed to Step 5a below
   - [ ] If deployment completes without timeout (rare), skip to Step 6
   - [ ] **Expected result**: Either "Apply complete!" OR timeout error (both are okay)

5a. **Handle** Terraform timeout (if needed)

- [ ] Check if AI Foundry is still provisioning:

     ```bash
     az cognitiveservices account show --name cog-standard-private-<suffix> \
       --resource-group rg-standard-private-<suffix> \
       --query "properties.provisioningState" -o tsv
     ```

- [ ] If status is "Creating", wait and check every 2-3 minutes until it shows "Succeeded"
- **Pro tip**: Total provisioning time is typically 35-45 minutes. If Terraform timed out at 30 minutes, you may only need to wait another 5-15 minutes.
- [ ] Once status is "Succeeded", import the resource into Terraform state:

     ```bash
     terraform import 'module.ai_foundry.azapi_resource.ai_foundry' \
       '/subscriptions/<your-sub-id>/resourceGroups/rg-standard-private-<suffix>/providers/Microsoft.CognitiveServices/accounts/cog-standard-private-<suffix>'
     ```

- [ ] Continue deployment to create remaining resources (Project, models, RBAC):

     ```bash
     terraform apply -auto-approve
     ```

- [ ] **Expected result**: Remaining resources created, full deployment complete

6. **Verify** deployment in Azure Portal
   - [ ] Open Azure Portal, search for resource group `rg-standard-private-*`
   - [ ] Confirm presence of: AI Foundry, AI Foundry Project, Application Insights, Private Endpoints
   - [ ] Click on AI Foundry resource → Networking → verify "Public network access: Disabled"
   - [ ] Navigate to prerequisites resource group, verify VNet and capability host resources exist
   - [ ] **Expected result**: Complete private architecture deployed across two resource groups

### Task 3: Document Security Decisions (10 minutes)

**What You'll Do**: Create an Architecture Decision Record documenting security choices, trade-offs, and rationale for compliance review.

**Steps**:

1. **Create** ADR document structure
   - [ ] In your local workspace, create directory: `mkdir -p docs/adr`
   - [ ] Create new file: `docs/adr/001-caira-production-security.md`
   - [ ] Add ADR template structure with sections: Context, Decision, Consequences, Alternatives Considered
   - **Pro tip**: Use `caira-assistant` or `@workspace` to ask: "Show me the ADR template structure from CAIRA docs/adr/ directory"
   - [ ] **Expected result**: ADR markdown file with proper template structure ready for content

2. **Document** private networking decision
   - [ ] In ADR under "Context" section, explain why private networking was required (healthcare compliance, zero-trust principles)
   - [ ] In "Decision" section, document: "Deployed foundry_standard_private with public_network_access_enabled=false, private endpoints for all services"
   - [ ] In "Consequences" section, note: "Services only accessible from VNet. Requires VPN or ExpressRoute for developer access. Increases deployment complexity but meets compliance requirements."
   - [ ] **Expected result**: Private networking decision clearly documented with business context

3. **Document** managed identity decision
   - [ ] Add section documenting authentication approach: "Implemented managed identities for service-to-service authentication"
   - [ ] Explain trade-off: "Disabled API key authentication. Simplifies credential management and rotation. Requires proper RBAC configuration. Aligns with zero-trust principles."
   - [ ] List specific managed identity assignments (AI Foundry  to Storage, AI Foundry to Key Vault)
   - **Validation checkpoint**: Does your documentation explain WHY these decisions were made, not just WHAT was configured?
   - [ ] **Expected result**: Authentication strategy documented with clear rationale

4. **Document** RBAC least-privilege approach
   - [ ] Add section: "Applied least-privilege RBAC with role-specific assignments"
   - [ ] List key role assignments: Azure AI Developer for developers, Cognitive Services User for applications, Storage Blob Data Reader for read-only scenarios
   - [ ] Note security benefit: "Prevents privilege escalation. Limits blast radius of compromised credentials. Supports audit and compliance requirements."
   - [ ] **Expected result**: RBAC strategy clearly documented with security benefits

5. **Review** and finalize ADR
   - [ ] Add "Alternatives Considered" section listing options like: foundry_basic (rejected - no private networking), manual configuration (rejected - not repeatable)
   - [ ] Add date, author, status (Accepted), and any review board approval tracking
   - [ ] Save and commit ADR to version control
   - **Success check**: ADR provides complete context for security team to understand deployment decisions
   - [ ] **Expected result**: Professional ADR document ready for compliance review board

## Completion Check

**You've Succeeded When**:

- [ ] Prerequisites deployed: VNet, subnets, Cosmos DB, Storage, AI Search with `public_network_access_enabled = false`
- [ ] AI Foundry and Project deployed with private networking
- [ ] Azure Portal shows "Public network access: Disabled" on all services
- [ ] Private endpoints visible in Azure Portal for all AI Foundry resources
- [ ] Managed identities configured (verify in Portal IAM blade)
- [ ] RBAC assignments follow least-privilege principle
- [ ] ADR document completed with two-stage deployment rationale
- [ ] You can explain why two-stage deployment is a production pattern
- [ ] Security hardening checklist created for future deployments

**Next Steps**:

- Try [Kata 350: Validating CAIRA with Sample AI Application](./350-validating-caira-with-sample-ai-application.md) to test your secured infrastructure
- Explore [Kata 500: Managing CAIRA Infrastructure Lifecycle](./500-managing-caira-lifecycle.md) for update procedures
- Review Azure Security Benchmark recommendations for AI workloads

---

## Reference Appendix

- **Prerequisites Folder**: `reference_architectures/foundry_standard_private/prerequisites/` - Optional Terraform configuration for creating VNet, capability host resources, private DNS zones, and VNet links

### Key Learnings

- **Two-stage deployment pattern**: Separating networking/shared services from application infrastructure is production best practice, not a workaround
- **Azure AD authentication**: Using `shared_access_key_enabled = false` on Storage Accounts is more secure than key-based access
- **Private DNS zones**: Required for AI Foundry private endpoints to resolve correctly within the VNet
- **Split-region deployment**: VNet can be in one region (e.g., eastus for AI Foundry) while capability hosts are in another (e.g., westus2 for capacity)
- **Use existing resources when possible**: In real environments, networking and shared services often already exist - this kata supports both scenarios
- **Agent network injection takes time**: AI Foundry with agent capabilities requires 35-45 minutes to provision due to managed network infrastructure setup
- **Terraform timeouts are normal**: The 30-minute Terraform timeout doesn't mean failure - Azure continues provisioning. Learn to check Azure status and import resources.
- **Production resilience**: Understanding how to recover from timeouts and import existing resources is a critical production skill
- Always run `terraform plan` and review security settings before applying production infrastructure
- Document security decisions as you make them - ADRs are invaluable during compliance audits
- Use Azure Advisor Security recommendations to identify additional hardening opportunities
- **Prerequisites reusability**: One set of prerequisites (VNet, Cosmos, Storage, Search, DNS zones) can support multiple AI Foundry deployments
- **Security audit**: When using existing resources, verify they meet your security requirements (public access disabled, encryption enabled, authentication method)
- Test managed identity authentication from applications before disabling API keys in production
- Keep security hardening checklist updated as Azure security best practices evolve

### Help Resources

- **caira-assistant**: Use for deployment guidance, security best practices, and CAIRA architecture questions
- **Azure Security Benchmark**: [AI Security Baseline](https://learn.microsoft.com/security/benchmark/azure/baselines/ai-security-baseline) for compliance validation
- **ADR Templates**: Check `docs/adr/` directory in CAIRA repository for ADR examples
- **Azure Private Link**: [Private endpoint documentation](https://learn.microsoft.com/azure/private-link/private-endpoint-overview)

### Common Issues and Solutions

**Issue**: "Required variable not set" errors for subnet IDs or resource names

- **Root Cause**: The foundry_standard_private architecture requires existing resources - it doesn't create VNet or capability host resources
- **Solution**: Either use existing resources (Path A) or deploy prerequisites first (Path B). Provide all required variable values in `terraform.tfvars`
- **Required Variables**: `agents_subnet_id`, `foundry_subnet_id`, `existing_capability_host_resource_group_id`, `existing_cosmosdb_account_name`, `existing_storage_account_name`, `existing_search_service_name`

**Issue**: I want to create everything in one terraform apply

- **Root Cause**: Terraform can't determine `count` values when they depend on computed resource outputs (like subnet IDs created in same configuration)
- **Why It Matters**: The AI Foundry module needs to know at plan time whether subnets exist to decide resource creation strategy
- **Production Reality**: In enterprise environments, networking and shared services are usually managed separately from application infrastructure anyway. This kata teaches that real-world pattern.ble until after apply.
- **Production Pattern**: This two-stage approach is actually best practice for enterprise deployments - separating networking/shared services from application infrastructure

**Issue**: Prerequisites deployment shows `public_network_access_enabled = true`

- **Quick Fix**: Verify you're using the updated `prerequisites/main.tf` that explicitly sets `public_network_access_enabled = false` on Cosmos DB, Storage Account, and AI Search resources
- **Why It Matters**: Default Azure settings often enable public access. Explicit configuration ensures production security compliance

**Issue**: Storage Account deployment fails with "Key based authentication is not permitted"

- **Root Cause**: Your Azure subscription has a security policy that prevents key-based authentication on Storage Accounts (this is good!)
- **Solution**: The prerequisites terraform is configured to use Azure AD authentication instead (`shared_access_key_enabled = false` with `storage_use_azuread = true` in provider)
- **Why This Is Better**: Azure AD authentication is more secure than key-based access. No credentials to manage or rotate!
- **If Still Failing**: Ensure the Terraform azurerm provider version supports `storage_use_azuread` (requires ~> 4.40 or later)

**Issue**: Terraform plan fails with "Private DNS Zone not found"

- **Root Cause**: The AI Foundry module requires private DNS zones to exist before deployment
- **Solution**: The prerequisites terraform now creates all required DNS zones (privatelink.cognitiveservices.azure.com, privatelink.services.ai.azure.com, privatelink.openai.azure.com) and VNet links automatically
- **If Already Deployed Prerequisites Without DNS Zones**: Add the DNS zone resources to prerequisites/main.tf, run `terraform apply`, import any existing zones if needed, then deploy main AI Foundry

**Issue**: DNS zones created but deployment still fails

- **Quick Fix**: The DNS zones need VNet links. Verify prerequisites/main.tf includes `azurerm_private_dns_zone_virtual_network_link` resources for each DNS zone
- **Import if Needed**: If zones exist but aren't in Terraform state, import them: `terraform import azurerm_private_dns_zone.cognitive <zone-resource-id>`

**Issue**: Private endpoint DNS resolution fails - services unreachable

- **Quick Fix**: Verify private DNS zones are created and linked to VNet. Check main deployment outputs for private DNS zone resources. May need custom DNS configuration on VNet or Azure DNS Private Resolver
- **Additional Check**: Ensure you're accessing services from within the VNet or via VPN/ExpressRoute connected to the VNet

- **Quick Fix**: Verify private DNS zones are created and linked to VNet. Check `terraform plan` output for `azurerm_private_dns_zone` and `azurerm_private_dns_zone_virtual_network_link` resources. May need to configure custom DNS on VNet or use Azure DNS Private Resolver.

**Issue**: "Insufficient privileges" error during RBAC configuration

- **Quick Fix**: Verify your account has "User Access Administrator" role at subscription or resource group level. This role is required to assign roles to other identities. Run `az role assignment list --assignee $(az ad signed-in-user show --query id -o tsv)` to check.

**Issue**: Terraform plan shows public endpoints being created

- **Quick Fix**: Verify `public_network_access_enabled = false` is set in `terraform.tfvars`. Check module version supports private networking. Review `variables.tf` for correct variable name (may vary by CAIRA version).

**Issue**: API key authentication still enabled after deployment

- **Quick Fix**: Check if CAIRA module version supports `disable_api_key_authentication` variable. Some older versions may require manual configuration in Azure Portal. Navigate to service → Keys and Endpoints → Disable key-based authentication.

**Issue**: Terraform times out with "context deadline exceeded" during AI Foundry creation

- **Root Cause**: AI Foundry with agent network injection takes 35-45 minutes to provision. Terraform's default timeout is 30 minutes.
- **This is NOT a failure!** The resource continues provisioning in Azure after Terraform times out.
- **Solution**:
  1. Check provisioning status: `az cognitiveservices account show --name <name> --resource-group <rg> --query "properties.provisioningState" -o tsv`
  2. Wait until status changes from "Creating" to "Succeeded" (check every 2-3 minutes)
  3. Import the completed resource: `terraform import 'module.ai_foundry.azapi_resource.ai_foundry' '<full-resource-id>'`
  4. Run `terraform apply` to deploy remaining resources (Project, models, RBAC)
- **Why So Long?**: Agent network injection involves creating managed network infrastructure, configuring VNet integration, setting up private endpoints, and establishing secure connectivity - all of which require careful orchestration by Azure.

**Issue**: "Resource already exists" error after Terraform timeout

- **Root Cause**: The AI Foundry resource completed provisioning after Terraform timed out, but Terraform doesn't know about it because it's not in the state file.
- **Solution**: Import the resource into Terraform state (see timeout handling above), then run `terraform apply` again.

**Issue**: "FlagMustBeSetForRestore" error when redeploying

- **Root Cause**: A previous AI Foundry resource with the same name was soft-deleted and needs to be purged before creating a new one.
- **Solution**: Purge the soft-deleted resource:

  ```bash
  az cognitiveservices account purge --location <location> --resource-group <rg> --name <name>
  ```

  Wait 30 seconds, then retry `terraform apply`.

**Issue**: Subscription ID not set error during terraform operations

- **Root Cause**: The azurerm provider requires the subscription ID to be explicitly set.
- **Solution**: Set the environment variable before running terraform commands:

  ```bash
  export ARM_SUBSCRIPTION_ID=$(az account show --query id -o tsv)
  ```

**Issue**: "Account CapabilityHost is not in succeeded state" error when creating Project

- **Root Cause**: The Account-level CapabilityHost infrastructure for agent network injection has failed or is still provisioning. This is rare but can happen with complex network injection scenarios.
- **Diagnosis**: Check Account CapabilityHost status:

  ```bash
  az rest --method get --url "https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.CognitiveServices/accounts/<account-name>/capabilityHosts?api-version=2025-04-01-preview" --query "value[].{name:name, state:properties.provisioningState}" -o table
  ```

- **Solution if Failed**:
  1. Delete the failed CapabilityHost: `az rest --method delete --url "https://management.azure.com/.../capabilityHosts/<capability-host-name>?api-version=2025-04-01-preview"`
  2. Wait 1-2 minutes for deletion to complete
  3. Retry `terraform apply` - the system should auto-create a new CapabilityHost
  4. **Note**: Account CapabilityHost can take 45-60 minutes to provision with agent network injection
- **Solution if Still Creating**: Wait for "provisioningState" to become "Succeeded" before retrying Project creation

---

<!-- markdownlint-disable MD036 -->
*🤖 Crafted with precision by ✨Copilot following brilliant human instruction,
then carefully refined by our team of discerning human reviewers.*
<!-- markdownlint-enable MD036 -->
