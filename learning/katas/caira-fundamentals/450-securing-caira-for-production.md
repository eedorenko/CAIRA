---
# Core Metadata
title: "Kata: Securing CAIRA for Production"
description: Deploy foundry_standard_private architecture with complete network isolation, managed identities, RBAC, and zero-trust security principles for production AI workloads.
author: HVE Essentials Team
ms.date: 2025-12-11
ms.topic: how-to-guide

# Kata Identity
kata_id: caira-fundamentals-450-securing-caira-for-production
kata_category:
  - caira-fundamentals
kata_difficulty: 4
estimated_time_minutes: 45
requires_dev_container: true

# Learning Content
learning_objectives:
  - Deploy foundry_standard_private architecture with complete network isolation
  - Configure private endpoints and disable public network access for all services
  - Implement managed identities and disable API key authentication
  - Apply RBAC with least-privilege access principles
  - Validate security posture against Azure Security Benchmark
  - Document security architecture decisions in ADR format
prerequisite_katas:
  - caira-fundamentals-150-understanding-architecture-patterns
  - caira-fundamentals-200-devcontainer-foundry-basic-deployment
technologies:
  - CAIRA foundry_standard_private
  - Azure Private Link
  - Azure Managed Identity
  - Azure RBAC
  - Terraform
  - Azure CLI
  - VS Code Dev Containers
success_criteria:
  - Private architecture deployed with no public endpoints
  - All services using managed identities (API key auth disabled)
  - RBAC configured with least-privilege role assignments
  - Security configuration validated against Azure Security Benchmark
  - ADR document completed with security decisions and rationale

# AI Coaching
ai_coaching_level: guided
scaffolding_level: medium-heavy
hint_strategy: progressive
hint_frequency: strategic
common_pitfalls:
  - Forgetting to configure private DNS zones for private endpoints
  - Deploying without required prerequisite infrastructure (VNet, subnets, capability hosts)
  - Overly permissive RBAC role assignments instead of least-privilege
  - Skipping ADR documentation and losing track of security trade-off decisions
  - Not validating that public network access is actually disabled after deployment

# Requirements
requires_azure_subscription: true
requires_local_environment: true
requires_github_account: false

# SEO & Discoverability
tags:
  - caira-fundamentals
  - security
  - zero-trust
  - private-networking
search_keywords:
  - CAIRA security hardening
  - Azure AI Foundry private endpoints
  - zero-trust AI infrastructure
  - managed identity authentication
  - production security CAIRA
real_world_application: Enterprise security teams deploy AI infrastructure following zero-trust principles for regulated workloads (healthcare, finance) requiring complete network isolation, managed identities, least-privilege access, and comprehensive audit logging with formal security decision documentation.

# Optional Fields
related_katas:
  - caira-fundamentals-400-customizing-caira-for-your-requirements
  - caira-fundamentals-200-devcontainer-foundry-basic-deployment
related_labs: []
skill_assessment_id: ""
validation_commands:
  - az network vnet show
  - az network private-endpoint list
  - terraform validate
  - terraform plan
  - az cognitiveservices account show
chatmode_references:
  - caira-assistant
  - learning-kata-coach
file_references:
  - reference_architectures/foundry_standard_private/README.md
  - reference_architectures/foundry_standard_private/variables.tf
  - reference_architectures/foundry_standard_private/main.tf
  - docs/security_posture.md
  - testing/infrastructure_pools/foundry_standard_private/README.md
troubleshooting_guide: "../../../docs/troubleshooting.md"
---

## Quick Context

### You'll Learn

- How to deploy CAIRA's most secure architecture (foundry_standard_private) with complete network isolation
- How to configure private endpoints, managed identities, and RBAC for zero-trust security
- How to validate security posture against Azure Security Benchmark
- How to document security decisions formally using Architecture Decision Records (ADR)

### Prerequisites

- Completed [Kata 150: Understanding CAIRA Architecture Patterns](./150-understanding-caira-architecture-patterns.md)
- Completed [Kata 200: Devcontainer & Foundry Basic Deployment](./200-devcontainer-foundry-basic-deployment.md)
- Azure subscription with **Contributor** + **User Access Administrator** (or **Owner**)
- Understanding of Azure networking fundamentals (VNets, subnets, private endpoints)
- Familiarity with zero-trust security principles (recommended)

### Real Challenge

You're a platform engineer at a healthcare organization building an AI-powered patient records analysis system. Your enterprise security team has issued a mandate: all AI infrastructure must follow zero-trust security principles for HIPAA compliance. Specifically, you need:

- **No public endpoints**: Complete network isolation with private endpoints only
- **No API keys**: Managed identities for all authentication (disable API key auth)
- **Least-privilege access**: RBAC with minimal required permissions
- **Encryption everywhere**: Data at rest and in transit
- **Audit logging**: Comprehensive diagnostic logging for compliance review
- **Security documentation**: Formal ADR documenting all security decisions and trade-offs

**Your Task**: Deploy the foundry_standard_private architecture with all required security controls, validate the security posture, and document your security decisions in ADR format for the compliance review.

## Essential Setup

**Required** (check these first):

- [ ] CAIRA repository cloned locally
- [ ] VS Code with Dev Containers extension installed
- [ ] Docker Desktop installed and running
- [ ] Azure CLI authenticated (`az account show` works in devcontainer)
- [ ] Azure subscription with sufficient permissions and quota

**Important Context**: The foundry_standard_private architecture requires **existing prerequisite infrastructure** before deployment. You'll need to either create or reference:

- Virtual Network (VNet) with at least 2 subnets (foundry subnet + agents subnet)
- Private DNS zones for name resolution
- Existing capability host resources (Cosmos DB, Storage Account, AI Search)

For this kata, we'll use the CAIRA testing infrastructure pool which provides all prerequisites.

**Quick Validation**: In devcontainer terminal, run `az account show && terraform version` and confirm both succeed.

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

### Task 1: Understand Security Architecture Requirements (10 minutes)

**What You'll Do**: Review the foundry_standard_private architecture and security controls to understand what makes it production-ready.

**Steps**:

1. **Review** the security posture documentation
   - [ ] Open `docs/security_posture.md` in VS Code
   - [ ] Read the "Key Security Decisions" section (lines 42-155)
   - [ ] Identify the 4 main security decision areas: network access, authentication, encryption, and public vs private patterns
   - **Pro tip**: This document explains CAIRA's security philosophy and what's intentionally included vs excluded
   - [ ] **Expected result**: You understand CAIRA's baseline security approach

2. **Compare** foundry_standard_private architecture to foundry_basic
   - [ ] Open `reference_architectures/foundry_standard_private/README.md`
   - [ ] Review the "Key Features" section and note the security-specific features
   - [ ] Compare with `reference_architectures/foundry_basic/README.md` to see the differences
   - **Validation checkpoint**: Can you explain why private networking matters for healthcare/finance workloads?
   - [ ] **Expected result**: Clear understanding of security enhancements in private architecture

3. **Examine** the required variables for foundry_standard_private
   - [ ] Open `reference_architectures/foundry_standard_private/variables.tf`
   - [ ] Find the networking-related variables: `foundry_subnet_id`, `agents_subnet_id`
   - [ ] Find the existing resource variables: `existing_capability_host_resource_group_id`, `existing_cosmosdb_account_name`, `existing_storage_account_name`, `existing_search_service_name`
   - **Pro tip**: These variables enforce the dependency on existing secure infrastructure
   - [ ] **Expected result**: You know what prerequisite infrastructure is required

### Task 2: Deploy Testing Infrastructure Pool (15 minutes)

**What You'll Do**: Deploy the durable testing infrastructure that provides all prerequisites for foundry_standard_private.

**Steps**:

1. **Navigate** to the testing infrastructure directory
   - [ ] In devcontainer terminal, run: `cd testing/infrastructure_pools/foundry_standard_private`
   - [ ] Review the README to understand what resources will be created
   - [ ] Open `main.tf` and note the VNet configuration (172.16.0.0/16) with connection subnet
   - **Pro tip**: This infrastructure pool creates VNet, private DNS zones, and capability host resources (Cosmos DB, Storage, AI Search)
   - [ ] **Expected result**: You understand the prerequisite infrastructure being deployed

2. **Initialize** Terraform for the infrastructure pool
   - [ ] Run: `terraform init`
   - [ ] Verify initialization completes successfully
   - **Success check**: You should see "Terraform has been successfully initialized!"
   - [ ] **Expected result**: Terraform providers downloaded and ready

3. **Plan** the infrastructure pool deployment
   - [ ] Run: `terraform plan -out=tfplan`
   - [ ] Review the resources that will be created (VNet, subnets, DNS zones, Cosmos DB, Storage, AI Search)
   - [ ] Verify no unexpected resources appear in the plan
   - **Validation checkpoint**: Are you seeing approximately 15-20 resources to be created?
   - [ ] **Expected result**: Clean plan with only expected infrastructure resources

4. **Deploy** the infrastructure pool
   - [ ] Run: `terraform apply tfplan`
   - [ ] Wait for deployment to complete (approximately 5-10 minutes)
   - **Pro tip**: Cosmos DB and AI Search can take several minutes to provision
   - [ ] **Expected result**: All resources deployed successfully

5. **Capture** the infrastructure pool outputs
   - [ ] Run: `terraform output -json > ../outputs.json`
   - [ ] View the outputs: `terraform output`
   - [ ] Note the subnet IDs, resource group ID, and resource names - you'll need these for the next task
   - **Success check**: You should see outputs for VNet, subnets, DNS zones, and capability host resources
   - [ ] **Expected result**: All required values available for foundry_standard_private configuration

### Task 3: Configure and Deploy foundry_standard_private (15 minutes)

**What You'll Do**: Configure and deploy the secure private architecture using the testing infrastructure prerequisites.

**Steps**:

1. **Navigate** to the foundry_standard_private directory
   - [ ] In terminal, run: `cd ../../../reference_architectures/foundry_standard_private`
   - [ ] List the files: `ls -la`
   - [ ] Identify the key files: `main.tf`, `variables.tf`, `terraform.tfvars` (if exists)
   - **Pro tip**: You'll configure this deployment via terraform.tfvars
   - [ ] **Expected result**: You're in the correct directory with deployment files

2. **Create** terraform.tfvars with secure configuration
   - [ ] Create a new file: `terraform.tfvars`
   - [ ] Add configuration with subnet IDs, capability host resource group ID, and resource names from infrastructure pool outputs
   - [ ] Add tags for governance: environment, purpose, security
   - **Pro tip**: Get the actual values by running `cd testing/infrastructure_pools/foundry_standard_private && terraform output` in another terminal
   - [ ] **Expected result**: Valid terraform.tfvars file with all required variables

3. **Initialize** Terraform for foundry_standard_private
   - [ ] Run: `terraform init`
   - [ ] Verify the ai_foundry module and other dependencies are initialized
   - **Success check**: "Terraform has been successfully initialized!"
   - [ ] **Expected result**: Ready to plan the secure deployment

4. **Generate** and review the deployment plan
   - [ ] Run: `terraform plan -out=tfplan`
   - [ ] Review the plan carefully and verify security configurations
   - [ ] Confirm `publicNetworkAccess = "Disabled"` for AI Foundry
   - [ ] Confirm `disableLocalAuth = true` (API keys disabled)
   - [ ] Confirm private endpoints are being created
   - [ ] Confirm managed identity is enabled (`identity.type = "SystemAssigned"`)
   - **Validation checkpoint**: Do you see private endpoint resources with `subresource_names = ["account"]`?
   - [ ] **Expected result**: Security controls are properly configured in the plan

5. **Deploy** the secure private architecture
   - [ ] Run: `terraform apply tfplan`
   - [ ] Wait for deployment to complete (approximately 10-15 minutes)
   - **Pro tip**: Private endpoint provisioning and DNS propagation can take extra time
   - [ ] **Expected result**: foundry_standard_private architecture fully deployed

6. **Capture** the deployment outputs
   - [ ] Run: `terraform output`
   - [ ] Note the AI Foundry resource ID, project ID, and managed identity principal ID
   - [ ] Save these for validation in the next task
   - **Success check**: You should see outputs including `ai_foundry_project_identity_principal_id`
   - [ ] **Expected result**: All deployment information captured

### Task 4: Validate Security Configuration (10 minutes)

**What You'll Do**: Verify that all security controls are properly configured and working as expected.

**Steps**:

1. **Verify** public network access is disabled
   - [ ] Get your AI Foundry resource name from terraform output
   - [ ] Run: `az cognitiveservices account show --name <foundry-name> --resource-group <rg-name> --query "{publicNetworkAccess:properties.publicNetworkAccess, localAuth:properties.disableLocalAuth}"`
   - [ ] Confirm `publicNetworkAccess` shows "Disabled"
   - [ ] Confirm `disableLocalAuth` shows `true`
   - **Validation checkpoint**: These two settings enforce zero-trust networking and authentication
   - [ ] **Expected result**: Both security flags are properly set

2. **Validate** private endpoints are created
   - [ ] Get your resource group name from terraform output
   - [ ] Run: `az network private-endpoint list --resource-group <rg-name> --output table`
   - [ ] Verify you see private endpoint(s) for the AI Foundry account
   - [ ] Check the connection state shows "Approved"
   - **Success check**: Private endpoint should show connection to cognitiveservices resource
   - [ ] **Expected result**: Private endpoints successfully provisioned

3. **Verify** managed identity is configured
   - [ ] Run: `az cognitiveservices account show --name <foundry-name> --resource-group <rg-name> --query "identity"`
   - [ ] Confirm `type` is "SystemAssigned"
   - [ ] Note the `principalId` matches your terraform output `ai_foundry_project_identity_principal_id`
   - **Pro tip**: This managed identity is used for service-to-service authentication
   - [ ] **Expected result**: System-assigned managed identity is active

4. **Check** RBAC role assignments
   - [ ] Get the AI Foundry resource ID from terraform output
   - [ ] Run: `az role assignment list --scope <ai-foundry-resource-id> --output table`
   - [ ] Review the role assignments and verify least-privilege principle
   - [ ] Identify which roles are assigned to the managed identity
   - **Validation checkpoint**: Are there any overly permissive roles like "Owner" or "Contributor" that shouldn't be there?
   - [ ] **Expected result**: Role assignments follow least-privilege principle

5. **Validate** private DNS integration
   - [ ] Get your VNet ID from the infrastructure pool outputs
   - [ ] Run: `az network private-dns link vnet list --resource-group rg-fstdprv-durable --zone-name privatelink.cognitiveservices.azure.com --output table`
   - [ ] Verify the VNet link exists and registration is enabled
   - **Pro tip**: Private DNS zones are required for name resolution of private endpoints
   - [ ] **Expected result**: DNS zone linked to VNet for private name resolution

6. **Review** security posture with Azure Advisor
   - [ ] In Azure Portal, navigate to Azure Advisor
   - [ ] Filter by "Security" recommendations
   - [ ] Review any recommendations for your deployed resources
   - [ ] Note any high or medium severity items for documentation
   - **Success check**: Ideally you should see minimal security recommendations for your private architecture
   - [ ] **Expected result**: Security recommendations reviewed and documented

### Task 5: Document Security Decisions in ADR (5 minutes)

**What You'll Do**: Create an Architecture Decision Record documenting your security decisions, rationale, and trade-offs.

**Steps**:

1. **Create** ADR document structure
   - [ ] Create a new file: `security-adr-kata450.md` in the foundry_standard_private directory
   - [ ] Add the ADR header with title, status, date, and decision makers
   - **Pro tip**: ADRs follow a standard format: Context, Decision, Consequences, Alternatives Considered
   - [ ] **Expected result**: ADR document template ready

2. **Document** the security context
   - [ ] Write the "Context" section explaining the healthcare HIPAA compliance requirement
   - [ ] List the specific security mandates: no public endpoints, managed identities, least-privilege, encryption, audit logging
   - [ ] Explain why zero-trust principles are required for this workload
   - **Validation checkpoint**: Does your context clearly explain *why* these security decisions were needed?
   - [ ] **Expected result**: Clear context section documenting requirements

3. **Document** the security decisions made
   - [ ] Write the "Decision" section listing each security control implemented
   - [ ] Include architecture choice, network isolation, authentication method, access control, and encryption
   - **Success check**: Each decision should be specific and actionable
   - [ ] **Expected result**: Complete list of security decisions documented

4. **Document** the consequences and trade-offs
   - [ ] Write the "Consequences" section covering positive and negative impacts
   - [ ] Include security benefits, compliance achievements, complexity costs, and usability trade-offs
   - **Pro tip**: Honest documentation of trade-offs helps future teams understand why decisions were made
   - [ ] **Expected result**: Balanced view of consequences documented

5. **Document** alternatives considered
   - [ ] Write the "Alternatives Considered" section
   - [ ] List foundry_basic, foundry_standard (public), and custom infrastructure as alternatives
   - [ ] Explain why each alternative was rejected
   - **Validation checkpoint**: Does your ADR explain *why* each alternative was rejected?
   - [ ] **Expected result**: Thoughtful comparison of alternatives documented

6. **Finalize** and review ADR
   - [ ] Add a "Status" field: "Accepted"
   - [ ] Add date and decision makers
   - [ ] Review the complete ADR for clarity and completeness
   - [ ] Save the file for compliance review
   - **Success check**: ADR is clear enough that a new team member could understand the security decisions 6 months from now
   - [ ] **Expected result**: Professional ADR document ready for compliance review

## Completion Check

**You've Succeeded When**:

- [ ] foundry_standard_private architecture deployed with complete network isolation
- [ ] Public network access disabled and verified (`publicNetworkAccess = "Disabled"`)
- [ ] API key authentication disabled and verified (`disableLocalAuth = true`)
- [ ] Private endpoints created and connection state is "Approved"
- [ ] Managed identity configured and role assignments follow least-privilege
- [ ] Private DNS zones linked to VNet for name resolution
- [ ] Azure Advisor security recommendations reviewed
- [ ] Complete ADR document created with context, decisions, consequences, and alternatives
- [ ] Can explain the security architecture and trade-offs to a colleague

**Next Steps**: You now have a production-ready secure AI infrastructure. Consider:

- [Kata 350: Validating CAIRA with Sample AI Application](./350-validating-caira-with-sample-ai-application.md) to test end-to-end security
- [Kata 500: Managing CAIRA Infrastructure Lifecycle](./500-managing-caira-lifecycle.md) to learn ongoing security maintenance

---

## Reference Appendix

### Help Resources

- **CAIRA Assistant**: Use for architecture-specific security guidance and best practices
- **Learning Kata Coach**: Use for progress tracking and hints on security validation
- **[CAIRA Security Posture](../../../docs/security_posture.md)**: Comprehensive security philosophy and decisions
- **[Azure Security Benchmark](https://learn.microsoft.com/security/benchmark/azure/)**: Microsoft's security baseline for Azure
- **[Zero Trust Security](https://learn.microsoft.com/security/zero-trust/)**: Zero-trust architecture principles

### Professional Tips

- **Private endpoints take time**: DNS propagation and endpoint provisioning can take 10-15 minutes
- **Test from within VNet**: Remember, with public access disabled, you need VPN or ExpressRoute to access resources
- **Document trade-offs**: Always capture security vs usability trade-offs in ADRs for future reference
- **Validate continuously**: Use Azure Advisor and Security Center for ongoing security posture monitoring
- **Least-privilege by default**: Start with minimal permissions and add only what's needed

### Troubleshooting

**Issue**: Private endpoint connection shows "Pending" instead of "Approved"

- **Quick Fix**: Wait 5-10 minutes for automatic approval, or manually approve in Azure Portal under Private Endpoint Connections

**Issue**: Cannot access AI Foundry resources after deployment

- **Quick Fix**: Verify you're accessing from within the VNet or through VPN/ExpressRoute. Public access is disabled by design.

**Issue**: Terraform plan shows unexpected resource recreations

- **Quick Fix**: Review variable changes carefully. Changing network settings often requires resource replacement. Use `terraform plan` before apply.

**Issue**: DNS resolution fails for private endpoints

- **Quick Fix**: Verify private DNS zones are linked to your VNet. Check `az network private-dns link vnet list`.

**Issue**: Managed identity doesn't have required permissions

- **Quick Fix**: Review role assignments in `modules/ai_foundry_project/agent_capability_host_connections.role_assignments.tf` and verify they match your requirements.

---

<!-- markdownlint-disable MD036 -->
*🤖 Crafted with precision by ✨Copilot following brilliant human instruction,
then carefully refined by our team of discerning human reviewers.*
<!-- markdownlint-enable MD036 -->
