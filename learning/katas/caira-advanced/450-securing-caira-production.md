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
  - caira-advanced-400-customizing-caira-requirements
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

### Task 1: Prepare Prerequisites (5-30 minutes)

**What You'll Do**: Ensure you have the required prerequisite resources for foundry_standard_private architecture: VNet with subnets, Cosmos DB, Storage Account, and AI Search.

**Production Reality**: The `foundry_standard_private` architecture is designed to work with **existing resources**. In enterprise environments, networking and shared services are typically managed by separate teams (network teams manage VNets, database teams manage Cosmos, etc.). This kata teaches you to work with that reality.

**Two Paths**:

- **Path A**: You already have these resources → Gather their IDs and skip to Task 2 (5 min)
- **Path B**: You need to create these resources → Use Azure CLI to create secure infrastructure (20-30 min)

**Why Prerequisites Matter**: The foundry_standard_private architecture requires existing networking (VNet + subnets) and capability host resources (Cosmos DB, Storage, AI Search) to deploy AI Foundry with private networking and agent capabilities. This separation is a **production best practice**, not a limitation.

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

1. **Verify** security settings (optional but recommended for production)
   - [ ] Check if Cosmos DB has public access disabled: `az cosmosdb show --name <name> --resource-group <rg> --query publicNetworkAccess`
   - [ ] Check Storage Account: `az storage account show --name <name> --resource-group <rg> --query publicNetworkAccess`
   - [ ] Check AI Search: `az search service show --name <name> --resource-group <rg> --query publicNetworkAccess`
   - **Pro tip**: If public access is enabled, consider whether this meets your production security requirements. Kata focuses on AI Foundry security, but capability hosts should also be secured for true zero-trust.
   - [ ] **Expected result**: Awareness of existing resource security posture

1. **Skip** to Task 2 with your gathered values

---

#### Path B: Create Prerequisites with Azure CLI (20-30 minutes)

If you need to create VNet, subnets, and capability host resources for this kata:

**Why Separate Creation?**: The `foundry_standard_private` architecture is designed to consume **existing resources**. This mirrors real enterprise environments where infrastructure is managed by different teams. Creating prerequisites separately teaches this production pattern.

**What You'll Create** (with security hardening):

- Resource Group
- Virtual Network (10.0.0.0/16) with 2 subnets (foundry: 10.0.1.0/24, agents: 10.0.2.0/24)
- Cosmos DB NoSQL account (serverless, **public access disabled** ✅)
- Storage Account (**Azure AD auth only, no shared keys** ✅)
- AI Search service (basic SKU, **public access disabled** ✅)

**Steps**:

1. **Set environment variables** for consistent naming
   - [ ] In terminal, run:

     ```bash
     export LOCATION="eastus"
     export RANDOM_SUFFIX=$(openssl rand -hex 3)
     export RG_NAME="rg-kata450-prereqs-${RANDOM_SUFFIX}"
     echo "Resource Group: ${RG_NAME}"
     ```

   - [ ] **Expected result**: Resource group name displayed with random suffix

1. **Create resource group**
   - [ ] Run:

     ```bash
     az group create --name ${RG_NAME} --location ${LOCATION} \
       --tags Environment=Learning Project=CAIRA-Kata-450 ManagedBy=AzureCLI
     ```

   - [ ] **Expected result**: JSON output showing resource group created successfully

1. **Create Virtual Network with secure subnets**
   - [ ] Run:

     ```bash
     az network vnet create \
       --resource-group ${RG_NAME} \
       --name vnet-kata450 \
       --address-prefix 10.0.0.0/16 \
       --subnet-name foundry-subnet \
       --subnet-prefix 10.0.1.0/24 \
       --location ${LOCATION}

     az network vnet subnet create \
       --resource-group ${RG_NAME} \
       --vnet-name vnet-kata450 \
       --name agents-subnet \
       --address-prefix 10.0.2.0/24 \
       --delegations Microsoft.App/environments
     ```

   - [ ] **Expected result**: VNet created with 2 subnets, agents subnet delegated to Microsoft.App/environments

1. **Create Cosmos DB (serverless)** ✅

   **Important**: Azure CLI 2.81.0 has a known issue where `az cosmosdb create` requests an invalid API version. Use the Azure REST API directly as a workaround. This teaches a valuable production troubleshooting skill!

   - [ ] Get your subscription ID and create Cosmos DB:

     ```bash
     SUB_ID=$(az account show --query id -o tsv)

     az rest --method put \
       --url "https://management.azure.com/subscriptions/${SUB_ID}/resourceGroups/rg-kata450-prereqs-${RANDOM_SUFFIX}/providers/Microsoft.DocumentDB/databaseAccounts/cosmos-kata450-${RANDOM_SUFFIX}?api-version=2024-08-15" \
       --body '{
         "location": "eastus",
         "properties": {
           "databaseAccountOfferType": "Standard",
           "capabilities": [{"name": "EnableServerless"}],
           "consistencyPolicy": {"defaultConsistencyLevel": "Session"},
           "locations": [{"locationName": "eastus", "failoverPriority": 0}]
         }
       }'
     ```

   - [ ] Creation starts asynchronously - will take 8-10 minutes
   - **Note**: We'll disable public access after creation completes (some subscription policies block setting it during creation)
   - **Pro tip**: Using `az rest` with specific API versions bypasses CLI bugs and is a valuable production skill
   - [ ] **Expected result**: Cosmos DB creation initiated successfully

1. **Create Storage Account with maximum security** ✅ (runs in parallel with Cosmos DB)
   - [ ] While Cosmos DB is provisioning, create Storage Account:

     ```bash
     az storage account create \
       --name stkata450${RANDOM_SUFFIX} \
       --resource-group rg-kata450-prereqs-${RANDOM_SUFFIX} \
       --location eastus \
       --sku Standard_LRS \
       --kind StorageV2 \
       --allow-blob-public-access false \
       --public-network-access Disabled \
       --allow-shared-key-access false \
       --https-only true \
       --min-tls-version TLS1_2
     ```

   - [ ] **Expected result**: Storage account created with maximum security (Azure AD auth only, no public access)
   - **Security Highlight**: `--allow-shared-key-access false` enforces Azure AD authentication - no keys to leak!

1. **Create AI Search with public access disabled** ✅ (runs in parallel)
   - [ ] In parallel, create AI Search service:

     ```bash
     az search service create \
       --name search-kata450-${RANDOM_SUFFIX} \
       --resource-group rg-kata450-prereqs-${RANDOM_SUFFIX} \
       --location eastus \
       --sku basic \
       --public-network-access disabled \
       --partition-count 1 \
       --replica-count 1
     ```

   - [ ] Takes 3-5 minutes to provision
   - [ ] **Expected result**: AI Search created with `publicNetworkAccess: Disabled`
   - **Production Tip**: Creating resources in parallel saves time - Storage and Search don't depend on each other

1. **Wait for Cosmos DB and verify all resources**
   - [ ] Check Cosmos DB provisioning status (should take 8-10 minutes total):

     ```bash
     az cosmosdb show --name cosmos-kata450-${RANDOM_SUFFIX} --resource-group rg-kata450-prereqs-${RANDOM_SUFFIX} \
       --query "{Name:name, Status:provisioningState, Location:location}" -o table
     ```

   - [ ] Wait until Status shows "Succeeded" (check every 2-3 minutes)
   - [ ] **Expected result**: Cosmos DB shows `Status: Succeeded`

1. **Harden Cosmos DB security** (disable public access)
   - [ ] Once Cosmos DB is provisioned, disable public network access:

     ```bash
     SUB_ID=$(az account show --query id -o tsv)

     az rest --method patch \
       --url "https://management.azure.com/subscriptions/${SUB_ID}/resourceGroups/rg-kata450-prereqs-${RANDOM_SUFFIX}/providers/Microsoft.DocumentDB/databaseAccounts/cosmos-kata450-${RANDOM_SUFFIX}?api-version=2024-08-15" \
       --body '{"properties": {"publicNetworkAccess": "Disabled"}}'
     ```

   - [ ] Wait 1-2 minutes for update to complete
   - [ ] **Expected result**: Cosmos DB updated with `publicNetworkAccess: Disabled`
   - **Why Separate?**: Some subscription policies block setting public access during creation, but allow updates

1. **Verify security posture of all resources**
   - [ ] Run comprehensive security check:

     ```bash
     echo "=== Security Posture Verification ==="

     az cosmosdb show --name cosmos-kata450-${RANDOM_SUFFIX} \
       --resource-group rg-kata450-prereqs-${RANDOM_SUFFIX} \
       --query "{Name:name, PublicAccess:publicNetworkAccess, Status:provisioningState}" -o table

     az storage account show --name stkata450${RANDOM_SUFFIX} \
       --resource-group rg-kata450-prereqs-${RANDOM_SUFFIX} \
       --query "{Name:name, PublicAccess:publicNetworkAccess, SharedKeyAccess:allowSharedKeyAccess}" -o table

     az search service show --name search-kata450-${RANDOM_SUFFIX} \
       --resource-group rg-kata450-prereqs-${RANDOM_SUFFIX} \
       --query "{Name:name, PublicAccess:publicNetworkAccess}" -o table
     ```

   - [ ] Verify all show: `PublicAccess: Disabled`, Storage shows `SharedKeyAccess: false`
   - [ ] **Expected result**: All prerequisite resources meet zero-trust security requirements ✅

1. **Create Private DNS Zones** for AI Foundry private networking
    - [ ] Create required Private DNS Zones and link to VNet:

      ```bash
      # Create Private DNS Zones
      az network private-dns zone create \
        --resource-group rg-kata450-prereqs-${RANDOM_SUFFIX} \
        --name privatelink.cognitiveservices.azure.com

      az network private-dns zone create \
        --resource-group rg-kata450-prereqs-${RANDOM_SUFFIX} \
        --name privatelink.services.ai.azure.com

      az network private-dns zone create \
        --resource-group rg-kata450-prereqs-${RANDOM_SUFFIX} \
        --name privatelink.openai.azure.com

      # Link Private DNS Zones to VNet
      az network private-dns link vnet create \
        --resource-group rg-kata450-prereqs-${RANDOM_SUFFIX} \
        --zone-name privatelink.cognitiveservices.azure.com \
        --name cognitive-vnet-link \
        --virtual-network vnet-kata450 \
        --registration-enabled false

      az network private-dns link vnet create \
        --resource-group rg-kata450-prereqs-${RANDOM_SUFFIX} \
        --zone-name privatelink.services.ai.azure.com \
        --name aiservices-vnet-link \
        --virtual-network vnet-kata450 \
        --registration-enabled false

      az network private-dns link vnet create \
        --resource-group rg-kata450-prereqs-${RANDOM_SUFFIX} \
        --zone-name privatelink.openai.azure.com \
        --name openai-vnet-link \
        --virtual-network vnet-kata450 \
        --registration-enabled false
      ```

    - [ ] **Expected result**: Three Private DNS Zones created and linked to VNet for private endpoint name resolution
    - **Why Needed?**: Private endpoints need DNS zones to resolve private IP addresses for AI services. Without these, AI Foundry deployment will fail.

1. **Capture resource IDs** for AI Foundry deployment
    - [ ] Get all required resource identifiers:

      ```bash
      export FOUNDRY_SUBNET_ID=$(az network vnet subnet show \
        --resource-group rg-kata450-prereqs-${RANDOM_SUFFIX} \
        --vnet-name vnet-kata450 \
        --name foundry-subnet \
        --query id -o tsv)

      export AGENTS_SUBNET_ID=$(az network vnet subnet show \
        --resource-group rg-kata450-prereqs-${RANDOM_SUFFIX} \
        --vnet-name vnet-kata450 \
        --name agents-subnet \
        --query id -o tsv)

      export RG_ID=$(az group show \
        --name rg-kata450-prereqs-${RANDOM_SUFFIX} \
        --query id -o tsv)

      echo -e "\n📋 Copy these values for Task 2 (AI Foundry deployment):\n"
      echo "agents_subnet_id = \"${AGENTS_SUBNET_ID}\""
      echo "foundry_subnet_id = \"${FOUNDRY_SUBNET_ID}\""
      echo "existing_capability_host_resource_group_id = \"${RG_ID}\""
      echo "existing_cosmosdb_account_name = \"cosmos-kata450-${RANDOM_SUFFIX}\""
      echo "existing_storage_account_name = \"stkata450${RANDOM_SUFFIX}\""
      echo "existing_search_service_name = \"search-kata450-${RANDOM_SUFFIX}\""
      ```

    - [ ] Copy all 6 values to a temporary file - you'll paste into `terraform.tfvars` in Task 2
    - [ ] **Expected result**: All prerequisite resource identifiers ready for AI Foundry deployment

### Task 2: Deploy AI Foundry with Private Networking (35-50 minutes)

**What You'll Do**: Deploy AI Foundry and Project on top of prerequisite infrastructure. This deployment adds private endpoints, managed identities, RBAC, and configures zero-trust security.

> **⚠️ Important Timing Note**: AI Foundry with agent network injection takes **35-45 minutes** to provision. Terraform has a 30-minute default timeout, so you'll likely see a timeout error even though the deployment is still progressing in Azure. **This is expected behavior** - don't worry! We'll walk you through how to handle this

**Steps**:

1. **Navigate** to main deployment directory
   - [ ] In terminal, run: `cd /workspace/reference_architectures/foundry_standard_private`
   - [ ] Verify you're in correct directory: `pwd` should show `/workspaces/CAIRA/reference_architectures/foundry_standard_private`
   - [ ] **Expected result**: Main terraform workspace directory

1. **Configure** main deployment with prerequisite values
   - [ ] Create or edit file: `terraform.tfvars`
   - [ ] Add prerequisite resource values:
     - **If you used Path A** (existing resources): Use the IDs/names you gathered
     - **If you used Path B** (created with Azure CLI): Paste the values from Task 1, Step 7
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

1. **Initialize** main Terraform workspace
   - [ ] Set Azure subscription ID as environment variable:

     ```bash
     export ARM_SUBSCRIPTION_ID=$(az account show --query id -o tsv)
     echo "✅ ARM_SUBSCRIPTION_ID set to: ${ARM_SUBSCRIPTION_ID}"
     ```

   - [ ] Run: `terraform init`
   - [ ] Review output for module downloads (ai_foundry, ai_foundry_project, common_models)
   - [ ] **Expected result**: Successful initialization with modules

   > **💡 Why use environment variable?**: Setting `ARM_SUBSCRIPTION_ID` allows Terraform to authenticate without modifying provider configuration files. This is the preferred approach for dev containers and CI/CD pipelines.

1. **Generate** and review main deployment plan
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

1. **Deploy** AI Foundry infrastructure
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

1. **Verify** deployment in Azure Portal
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

1. **Document** private networking decision
   - [ ] In ADR under "Context" section, explain why private networking was required (healthcare compliance, zero-trust principles)
   - [ ] In "Decision" section, document: "Deployed foundry_standard_private with public_network_access_enabled=false, private endpoints for all services"
   - [ ] In "Consequences" section, note: "Services only accessible from VNet. Requires VPN or ExpressRoute for developer access. Increases deployment complexity but meets compliance requirements."
   - [ ] **Expected result**: Private networking decision clearly documented with business context

1. **Document** managed identity decision
   - [ ] Add section documenting authentication approach: "Implemented managed identities for service-to-service authentication"
   - [ ] Explain trade-off: "Disabled API key authentication. Simplifies credential management and rotation. Requires proper RBAC configuration. Aligns with zero-trust principles."
   - [ ] List specific managed identity assignments (AI Foundry  to Storage, AI Foundry to Key Vault)
   - **Validation checkpoint**: Does your documentation explain WHY these decisions were made, not just WHAT was configured?
   - [ ] **Expected result**: Authentication strategy documented with clear rationale

1. **Document** RBAC least-privilege approach
   - [ ] Add section: "Applied least-privilege RBAC with role-specific assignments"
   - [ ] List key role assignments: Azure AI Developer for developers, Cognitive Services User for applications, Storage Blob Data Reader for read-only scenarios
   - [ ] Note security benefit: "Prevents privilege escalation. Limits blast radius of compromised credentials. Supports audit and compliance requirements."
   - [ ] **Expected result**: RBAC strategy clearly documented with security benefits

1. **Review** and finalize ADR
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

- Try [Kata 350: Validating CAIRA with Sample AI Application](../caira-fundamentals/350-validating-caira-with-sample-ai-application.md) to test your secured infrastructure
- Explore [Kata 400: Customizing CAIRA for Your Requirements](./400-customizing-caira-for-your-requirements.md) to add organizational governance
- Review Azure Security Benchmark recommendations for AI workloads

---

## Reference Appendix

### Production Pattern: Existing Resources

The `foundry_standard_private` architecture is designed to work with **existing Azure resources** (VNet, Cosmos DB, Storage, AI Search). This reflects real enterprise environments where:

- **Network teams** manage VNets and subnets
- **Database teams** manage Cosmos DB instances
- **Platform teams** manage shared Storage and AI Search services
- **Application teams** deploy AI Foundry on top of existing infrastructure

**For this kata**, if you don't have existing resources, use **Path B (Azure CLI)** to create secure prerequisite infrastructure that meets production security requirements.

### Key Learnings

### Key Learnings

- **Production pattern: Existing resources**: The `foundry_standard_private` architecture works with existing infrastructure (VNet, Cosmos DB, Storage, AI Search), reflecting real enterprise separation of concerns
- **Infrastructure separation**: Network teams manage VNets, database teams manage Cosmos, platform teams manage shared services - this kata teaches working across team boundaries
- **Azure CLI troubleshooting**: When CLI tools have bugs (like Cosmos DB API version issue), use `az rest` to call Azure REST APIs directly with specific API versions
- **Create-then-harden pattern**: Sometimes you must create resources first, then apply security hardening (like disabling public access) as a separate update - common in enterprise environments with strict policies
- **Parallel resource creation**: Create independent resources (Storage, Search) in parallel while Cosmos DB provisions - saves time in production deployments
- **Azure AD authentication**: Using `--allow-shared-key-access false` on Storage Accounts is more secure than key-based access - no credentials to leak!
- **Security by default**: All prerequisite resources created with `public access disabled` for maximum security hardening
- **Agent network injection takes time**: AI Foundry with agent capabilities requires 35-45 minutes to provision due to managed network infrastructure setup
- **Terraform timeouts are normal**: The 30-minute Terraform timeout doesn't mean failure - Azure continues provisioning. Learn to check Azure status and import resources
- **Production resilience**: Understanding how to recover from tool bugs, timeouts, and import existing resources is a critical production skill
- Always run `terraform plan` and review security settings before applying production infrastructure
- Document security decisions as you make them - ADRs are invaluable during compliance audits
- **Prerequisites reusability**: One set of prerequisites (VNet, Cosmos, Storage, Search) can support multiple AI Foundry deployments
- **Security audit**: When using existing resources (Path A), verify they meet your security requirements (public access disabled, encryption enabled, authentication method)
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
- **Solution**: Either use existing resources (Path A) or create prerequisites with Azure CLI (Path B). Provide all required variable values in `terraform.tfvars`
- **Required Variables**: `agents_subnet_id`, `foundry_subnet_id`, `existing_capability_host_resource_group_id`, `existing_cosmosdb_account_name`, `existing_storage_account_name`, `existing_search_service_name`
- **Production Reality**: In enterprise environments, networking and shared services are usually managed separately from application infrastructure. This kata teaches that real-world pattern.

**Issue**: I want to create everything in one terraform apply

- **Root Cause**: The `foundry_standard_private` architecture is **designed** to work with existing resources, not create them
- **Why It Matters**: This mirrors real enterprise environments where network teams, database teams, and application teams manage different infrastructure layers
- **Production Pattern**: Separating infrastructure concerns (networking/shared services from application workloads) is best practice for enterprise deployments
- **Solution**: Create prerequisite resources using Azure CLI (Path B in Task 1) or use existing resources (Path A)

**Issue**: Cosmos DB creation fails with "InvalidApiVersionParameter" error

- **Root Cause**: Azure CLI 2.81.0 has a bug where `az cosmosdb create` requests API version `2025-04-15` which doesn't exist
- **Solution**: Use `az rest` with the REST API directly and specify a valid API version (e.g., `2024-08-15`)
- **Why This Works**: Bypasses the CLI bug by calling Azure Resource Manager REST API directly
- **Production Learning**: Using `az rest` is a valuable troubleshooting technique when CLI tools have issues
- **Command Example**: See Task 1, Step 4 in the kata for the working REST API approach

**Issue**: Cosmos DB creation fails with "Bad Request" when setting publicNetworkAccess during creation

- **Root Cause**: Some Azure subscription policies block setting `publicNetworkAccess: Disabled` during initial Cosmos DB creation
- **Solution**: Create Cosmos DB first without the public access parameter, then update it after provisioning completes
- **Why This Works**: Azure policies may allow updates that they block during creation
- **Production Pattern**: Create resources first, harden security settings second - common in enterprise environments
- **Command**: Use `az rest --method patch` to update `publicNetworkAccess` after creation (see Task 1, Step 8)

**Issue**: Storage Account deployment fails with "Key based authentication is not permitted"

- **Root Cause**: Your Azure subscription has a security policy that prevents key-based authentication on Storage Accounts (this is good!)
- **Solution**: Path B Azure CLI commands already use `--allow-shared-key-access false` to enforce Azure AD authentication only
- **Why This Is Better**: Azure AD authentication is more secure than key-based access. No credentials to manage or rotate!
- **Verification**: Check the storage account with `az storage account show --name <name> --resource-group <rg> --query allowSharedKeyAccess` - should return `false`

**Issue**: Private endpoint DNS resolution fails - services unreachable

- **Root Cause**: Private DNS zones are required for private endpoint name resolution within VNets
- **Solution**: The foundry_standard_private module creates private DNS zones and links automatically. Ensure you're accessing services from within the VNet or via VPN/ExpressRoute connected to the VNet
- **Additional Check**: Verify private DNS zone resources in Azure Portal under your AI Foundry resource group

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
  1. Wait until status changes from "Creating" to "Succeeded" (check every 2-3 minutes)
  1. Import the completed resource: `terraform import 'module.ai_foundry.azapi_resource.ai_foundry' '<full-resource-id>'`
  1. Run `terraform apply` to deploy remaining resources (Project, models, RBAC)
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
  1. Wait 1-2 minutes for deletion to complete
  1. Retry `terraform apply` - the system should auto-create a new CapabilityHost
  1. **Note**: Account CapabilityHost can take 45-60 minutes to provision with agent network injection
- **Solution if Still Creating**: Wait for "provisioningState" to become "Succeeded" before retrying Project creation

---

<!-- markdownlint-disable MD036 -->
*🤖 Crafted with precision by ✨Copilot following brilliant human instruction,
then carefully refined by our team of discerning human reviewers.*
<!-- markdownlint-enable MD036 -->
