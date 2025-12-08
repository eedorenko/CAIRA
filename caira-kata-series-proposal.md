# CAIRA Kata Series Proposal

## Series Overview

This kata series provides a complete learning path from CAIRA beginner to production-ready deployment expert. The series is designed to build progressively, with each kata introducing new concepts while reinforcing previous learning.

**Total Series Time**: 5.5-6 hours of focused practice
**Target Audience**: Cloud engineers, DevOps professionals, AI platform engineers
**Prerequisites**: Basic Azure knowledge, familiarity with terminal/CLI, Docker Desktop

---

## Foundation Track: Build Foundation (1.5 hours)

### Kata 100: Gathering Requirements with CAIRA Assistant ✅
**Status**: EXISTS in microsoft/CAIRA repository
**Difficulty**: ⭐ Foundation (Level 1)
**Time**: 30 minutes
**File**: `100-gathering-requirements-with-caira-assistant.md`

**What Learners Will Do**:
- Activate and use the CAIRA Assistant chatmode in VS Code
- Ask effective requirements-gathering questions to understand business needs
- Analyze different business scenarios (POC, production, enterprise compliance)
- Match business requirements to CAIRA architecture choices
- Document architecture recommendations with clear justifications

**Learning Objectives**:
- Understand the four CAIRA reference architectures
- Use AI assistance for architecture decision-making
- Translate business requirements into technical architecture choices
- Document recommendations clearly and professionally

**Real-World Context**: You're consulting for a customer service team building an AI chatbot. They need production-level security, agent capabilities for knowledge retrieval, and GDPR compliance. Use CAIRA Assistant to determine the right architecture.

**Key Skills**: Requirements gathering, architecture selection, AI chatmode usage, technical documentation

**Prerequisites**: None - perfect starting point

**Success Criteria**:
- Completed requirements conversation with CAIRA Assistant
- Documented architecture recommendation with justification
- Can explain when to use each CAIRA architecture
- Understands trade-offs between basic and standard variants

---

### Kata 150: Understanding CAIRA Architecture Patterns
**Status**: NEW - To Be Created
**Difficulty**: ⭐ Foundation (Level 1)
**Time**: 30 minutes
**File**: `150-understanding-caira-architecture-patterns.md`

**What Learners Will Do**:
- Explore CAIRA reference architecture documentation systematically
- Compare resources deployed in basic vs standard architectures
- Analyze public vs private networking implications
- Fill out architecture comparison matrix with key differences
- Match 5 different business scenarios to appropriate architectures
- Calculate estimated costs for different architecture choices
- Use CAIRA Assistant to validate architecture understanding

**Learning Objectives**:
- Explain differences between all 4 CAIRA architectures in own words
- List resources included in each architecture from memory
- Understand when Azure AI Agent Service requires standard architecture
- Recognize cost implications of architecture choices
- Make informed architecture decisions based on requirements

**Real-World Scenarios**:
1. **Startup POC**: $200/month budget, public endpoints acceptable, no agent features needed
2. **Enterprise Chatbot**: Agent-based knowledge retrieval, private networking required, compliance needs
3. **Internal Developer Tool**: Standard features needed, no network isolation, moderate budget
4. **Healthcare AI**: HIPAA considerations, complete isolation, agent capabilities, audit logging
5. **Multi-National Deployment**: Data residency requirements, high availability, cost optimization

**Key Skills**: Architecture analysis, cost estimation, requirements mapping, decision documentation

**Prerequisites**:
- Completed Kata 100 (Gathering Requirements)
- Basic understanding of cloud infrastructure concepts

**Success Criteria**:
- Can name all 4 architectures and their primary use cases
- Completed architecture comparison matrix with accuracy
- Successfully matched all 5 scenarios to correct architectures
- Can explain basic vs standard resource differences
- Understands public vs private networking trade-offs

**Technologies**: CAIRA, Azure AI Foundry, Terraform (conceptual understanding)

---

### Kata 200: Devcontainer & Foundry Basic Deployment
**Status**: NEW - To Be Created
**Difficulty**: ⭐⭐ Skill (Level 2)
**Time**: 35 minutes
**File**: `200-devcontainer-foundry-basic-deployment.md`

**What Learners Will Do**:
- Open CAIRA repository in VS Code devcontainer
- Authenticate with Azure CLI inside the devcontainer
- Use caira-assistant chatmode to confirm prerequisites and configuration
- Decide between terraform.tfvars or main.tf edits for configuration
- Initialize Terraform in foundry_basic directory
- Generate and review terraform plan with assistant guidance
- Deploy foundry_basic architecture after explicit confirmation
- Verify deployment in Azure Portal
- Validate resource group and deployed resources

**Learning Objectives**:
- Develop inside CAIRA devcontainer environment
- Leverage caira-assistant for guided deployment workflow
- Execute complete Terraform deployment (init → plan → apply)
- Make informed configuration decisions (tfvars vs main.tf)
- Validate successful infrastructure deployment
- Follow CAIRA workflow standards and safety checks

**Real-World Context**: New team members must rapidly achieve a minimal Azure AI Foundry environment while following CAIRA workflow standards. You'll use the built-in devcontainer and assistant chatmode for guided, compliant deployment that follows team best practices.

**Key Skills**: Devcontainer development, AI-assisted deployment, Terraform workflow, configuration management, deployment validation

**Prerequisites**:
- Completed Kata 150 (Understanding Architecture Patterns)
- Docker Desktop installed and running
- VS Code with Dev Containers extension
- Azure subscription with Contributor + User Access Administrator roles
- Azure CLI authentication capability

**Success Criteria**:
- Devcontainer opens successfully with Azure CLI working
- Assistant session confirms foundry_basic selection and prerequisites
- Configuration approach chosen and validated by assistant
- Terraform plan shows only expected create actions
- Deployment completes and Azure lists resources in resource group
- Can explain devcontainer benefits and assistant-driven workflow

**Technologies**: Docker, VS Code devcontainer, caira-assistant chatmode, Terraform, Azure CLI, Azure AI Foundry, CAIRA foundry_basic

**Common Pitfalls**:
- Docker Desktop not running when opening devcontainer
- Terraform init fails due to network/proxy issues in container
- Applying without reviewing plan with assistant first
- Insufficient Azure permissions (need both Contributor and User Access Administrator)

---

## Skill Track: Build Proficiency (2 hours)

### Kata 300: Troubleshooting CAIRA Deployments
**Status**: NEW - To Be Created
**Difficulty**: ⭐⭐⭐ Advanced (Level 3)
**Time**: 45 minutes
**File**: `300-troubleshooting-caira-deployments.md`

**What Learners Will Do**:
- Diagnose authentication and permission errors in Terraform output
- Resolve resource quota exceeded errors by checking subscription limits
- Fix naming conflicts for globally unique resources (storage accounts)
- Troubleshoot provider version compatibility issues
- Debug state file locking problems in multi-user scenarios
- Resolve resource dependency ordering errors
- Use Azure Activity Log to investigate deployment failures
- Interpret Terraform error messages and trace root causes
- Apply fixes using targeted terraform apply commands
- Document troubleshooting steps and solutions
- Implement preventive measures to avoid common issues

**Learning Objectives**:
- Read and interpret Terraform error messages effectively
- Diagnose Azure permission and quota issues
- Resolve state management and locking conflicts
- Use Azure diagnostic tools for infrastructure troubleshooting
- Apply targeted fixes without full redeployment
- Build troubleshooting methodology and documentation habits

**Real-World Scenarios**:
1. **Permission Denied**: Deployment fails with "insufficient privileges" - diagnose RBAC requirements
2. **Quota Exceeded**: AI Search deployment fails due to subscription limits - investigate and request increase
3. **Name Conflict**: Storage account name already exists - implement unique naming strategy
4. **State Locked**: Another team member's terraform process locked state - resolve safely
5. **Dependency Error**: Resources fail due to incorrect dependency ordering - identify and fix
6. **Provider Mismatch**: Version incompatibility between providers - resolve version constraints

**Key Skills**: Terraform debugging, Azure diagnostics, error analysis, root cause investigation, documentation

**Prerequisites**:
- Completed Kata 200 (Devcontainer & Foundry Basic Deployment)
- Experience with at least one failed deployment (real or simulated)
- Access to Azure Portal and Activity Logs

**Success Criteria**:
- Successfully diagnosed and resolved all 6 troubleshooting scenarios
- Can read Terraform error messages and identify root causes
- Knows where to find Azure diagnostic information
- Built personal troubleshooting checklist
- Documented solutions for future reference
- Can explain prevention strategies for each issue type

**Technologies**: Terraform, Azure CLI, Azure Portal diagnostics, CAIRA (any architecture)

**Common Pitfalls**:
- Rushing to fix without understanding root cause
- Not checking Azure Activity Logs for detailed error information
- Attempting terraform apply again without addressing underlying issue
- Modifying state file directly (dangerous!)

---

### Kata 350: Validating CAIRA with Sample AI Application
**Status**: NEW - To Be Created
**Difficulty**: ⭐⭐⭐ Advanced (Level 3)
**Time**: 45 minutes
**File**: `350-validating-caira-with-sample-ai-application.md`

**What Learners Will Do**:
- Create systematic validation checklist for CAIRA deployments
- Verify all expected resources exist in Azure Portal
- Deploy a sample AI application to test the infrastructure
- Configure the sample app to connect to deployed AI Foundry resources
- Test end-to-end AI inference through the sample application
- Validate model deployment and API connectivity
- Perform smoke tests with various prompts and scenarios
- Monitor application performance and resource utilization
- Test RBAC and managed identity authentication from application
- Document validation results and application behavior
- Create reusable validation scripts including app deployment

**Learning Objectives**:
- Build systematic deployment validation methodology
- Deploy and configure sample AI applications on CAIRA infrastructure
- Test infrastructure components through real application workloads
- Verify end-to-end AI inference workflows
- Validate security configurations with application-level authentication
- Document validation procedures for team use

**Real-World Context**: Before handing off the CAIRA deployment to the application team, you need to validate that all infrastructure works end-to-end with a real AI application. Deploy a sample chatbot or inference app to test model connectivity, authentication, and performance under realistic conditions.

**Key Skills**: Application deployment, AI inference testing, end-to-end validation, performance monitoring, security verification

**Prerequisites**:
- Completed Kata 200 (Devcontainer & Foundry Basic Deployment)
- Deployed CAIRA infrastructure (foundry_basic minimum)
- Basic understanding of AI application architecture
- Familiarity with REST APIs or Python SDK

**Success Criteria**:
- Created comprehensive validation checklist (15+ items)
- Sample AI application deployed and connected to CAIRA resources
- Successful AI inference calls through application
- RBAC and managed identity authentication working from app
- Performance metrics collected and analyzed
- End-to-end workflow documented with screenshots/logs
- Validation script created for automation
- Documentation ready for team knowledge base

**Technologies**: Azure CLI, Python/JavaScript, Azure AI Foundry SDK, Azure Portal, CAIRA, sample AI applications, REST APIs

---

## Expert Track: Production Readiness (2.5 hours)

### Kata 400: Customizing CAIRA for Your Requirements
**Status**: NEW - To Be Created
**Difficulty**: ⭐⭐⭐⭐ Expert (Level 4)
**Time**: 45 minutes
**File**: `400-customizing-caira-requirements.md`

**What Learners Will Do**:
- Analyze organizational requirements for naming, tagging, and governance
- Customize terraform.tfvars with specific naming conventions and prefixes
- Override default SKUs for AI Search and other services (performance tuning)
- Add custom tags for cost tracking, department allocation, and compliance
- Configure optional features like Application Insights integration
- Modify location settings for data residency requirements (Canada Central example)
- Document customization decisions with rationale and impact analysis
- Validate customizations don't break CAIRA module contracts
- Test customized deployment in non-production environment
- Create customization template for future deployments
- Review customizations against Azure Well-Architected Framework

**Learning Objectives**:
- Safely customize CAIRA while maintaining module integrity
- Implement organizational governance requirements
- Balance customization with maintainability
- Document infrastructure-as-code decisions effectively
- Understand CAIRA module variable contracts and boundaries

**Real-World Context**: Your organization has specific requirements: resources must deploy to Canada Central for data residency, follow "proj-env-service-####" naming convention, include cost center tags (CC-12345), use Premium AI Search tier for production performance, and integrate with existing Application Insights workspace for centralized monitoring.

**Key Skills**: Variable management, governance implementation, naming conventions, tagging strategies, documentation

**Prerequisites**:
- Completed Kata 200 (Devcontainer & Foundry Basic Deployment)
- Understanding of organizational governance requirements
- Familiarity with Terraform variable precedence

**Success Criteria**:
- All customizations implemented according to requirements
- Resources deployed with correct naming and tagging
- Data residency requirement met (Canada Central)
- Performance settings validated (Premium AI Search)
- Customization decisions documented with rationale
- Deployment validated without module contract violations
- Reusable customization template created

**Technologies**: Terraform, CAIRA, Azure governance, tagging frameworks

**Common Pitfalls**:
- Overriding variables that break module dependencies
- Creating naming conventions that violate Azure length limits
- Forgetting to document why customizations were made
- Not testing customizations before production deployment

---

### Kata 450: Securing CAIRA for Production
**Status**: NEW - To Be Created
**Difficulty**: ⭐⭐⭐⭐ Expert (Level 4)
**Time**: 45 minutes
**File**: `450-securing-caira-production.md`

**What Learners Will Do**:
- Deploy foundry_standard_private architecture with complete network isolation
- Configure private endpoints for all Azure services
- Implement managed identities and disable API key authentication
- Set up RBAC with least-privilege access principles
- Configure virtual network and subnet security rules
- Enable encryption settings and Azure Key Vault integration
- Review security posture against Azure Security Benchmark
- Configure network security groups (NSGs) for traffic control
- Implement diagnostic logging for security audit trails
- Document security decisions in ADR (Architecture Decision Record) format
- Validate security configuration with Azure Advisor recommendations
- Create security hardening checklist for production deployments

**Learning Objectives**:
- Deploy and configure private networking for AI infrastructure
- Implement zero-trust security principles
- Configure identity and access management for production
- Document security architecture decisions formally
- Validate compliance with security frameworks

**Real-World Context**: Your enterprise security team requires all AI infrastructure to follow zero-trust principles for healthcare data (HIPAA considerations). Deploy foundry_standard_private with: no public endpoints, managed identities only (no API keys), complete network isolation, encryption at rest and in transit, least-privilege RBAC, and comprehensive audit logging. Document all security decisions for compliance review.

**Key Skills**: Private networking, identity management, security hardening, compliance awareness, ADR documentation

**Prerequisites**:
- Completed Kata 200 (Devcontainer & Foundry Basic Deployment)
- Understanding of Azure networking fundamentals
- Familiarity with zero-trust security principles
- ADR template knowledge (recommended)

**Success Criteria**:
- Private architecture deployed with no public endpoints
- All services using private endpoints and managed identities
- API key authentication disabled across all services
- RBAC configured with least-privilege assignments
- Network security groups properly configured
- Encryption validated for data at rest and in transit
- Security posture reviewed against Azure Security Benchmark
- ADR document completed with security decisions and rationale
- Security hardening checklist created for future use

**Technologies**: CAIRA foundry_standard_private, Azure Private Link, Managed Identities, RBAC, NSGs, Azure Key Vault, ADR templates

**Common Pitfalls**:
- Forgetting to configure private DNS zones for private endpoints
- Overly permissive RBAC assignments
- Not documenting security trade-offs and decisions
- Incomplete private endpoint coverage (missing a service)

---

### Kata 500: Managing CAIRA Infrastructure Lifecycle
**Status**: NEW - To Be Created
**Difficulty**: ⭐⭐⭐⭐⭐ Legendary (Level 5)
**Time**: 45 minutes
**File**: `500-managing-caira-lifecycle.md`

**What Learners Will Do**:
- Update existing CAIRA deployment with new variable values (tags, SKUs)
- Add new resources to deployed environment using Terraform
- Use terraform plan to preview changes before applying updates
- Handle Terraform state file management and remote state configuration
- Resolve state locking scenarios in team collaboration contexts
- Perform controlled resource tear-down with terraform destroy
- Implement backup and restore procedures for Terraform state
- Add lifecycle management tags for resource governance
- Update model deployments and AI service configurations
- Document upgrade and maintenance procedures
- Create runbook for infrastructure refresh and updates
- Plan for environment decommissioning with data preservation

**Learning Objectives**:
- Manage infrastructure changes safely using Terraform workflow
- Handle state management and team collaboration scenarios
- Execute controlled updates without service disruption
- Implement backup and disaster recovery for IaC state
- Document operational procedures for infrastructure lifecycle

**Real-World Context**: Your CAIRA deployment has been running for 3 months. You need to: add a new GPT-4o model deployment, upgrade AI Search from Basic to Standard tier, update cost center tags for new fiscal year allocation, add Application Insights integration for better monitoring, and eventually plan for quarterly environment refresh while preserving trained model data and conversation history.

**Key Skills**: Terraform state management, change planning, safe updates, backup/restore procedures, operational documentation

**Prerequisites**:
- Completed Kata 400 or 450 (Customization or Security)
- Existing CAIRA deployment (at least 1 week old preferred)
- Understanding of Terraform state concepts
- Experience with production infrastructure management (helpful)

**Success Criteria**:
- Successfully updated deployment without disruption
- Added new resources using Terraform workflow
- Implemented remote state storage with locking
- Backed up and restored state file successfully
- Updated tags and SKUs without recreating resources
- Created infrastructure update runbook
- Documented decommissioning procedures
- Can explain safe infrastructure change management

**Technologies**: Terraform, Azure Storage (remote state), CAIRA, Azure CLI, Git (for state backup)

**Common Pitfalls**:
- Not running terraform plan before apply (preview changes!)
- Modifying state file directly instead of using Terraform commands
- Not backing up state before major changes
- Forcing unlock without verifying safety
- Deleting resources without considering data preservation

---

### Learning Path Progression

**Foundation Path** (for new CAIRA users):
1. Kata 100 → Kata 150 → Kata 200

**Validation & Testing Path** (with sample applications):
1. Kata 100 → Kata 150 → Kata 200 → Kata 350

**Troubleshooting Focus**:
1. Kata 200 → Kata 300 (when issues arise)

**Production Readiness Path** (complete series):
1. Kata 100 → 150 → 200 → 300 → 350 → 400 → 450 → 500

**Security-First Path** (for compliance-heavy environments):
1. Kata 100 → 150 → 200 → 450 → 350 → 500


**Time to Competency**:
- Basic proficiency: 1.5 hours (Katas 100-200)
- Application validation: 2.5 hours (Katas 100-350)
- Full expertise: 5.5-6 hours (complete series through 500)

---

## Category README Structure (To Be Created)

**File**: `docs/learning/katas/caira-fundamentals/README.md`

**Required Sections** (per kata-category-readme.instructions.md):
1. Title + Brief Description
2. Category Overview (theme, technologies, progressive learning)
3. Prerequisites (Required Knowledge, Required Tools, Recommended Preparation)
4. Learning Path (visual progression diagram)
5. Category Katas (individual H3 sections for each kata 100-500)
6. Kata Comparison Matrix (table with all katas)
7. Suggested Learning Sequences (Beginners, Intermediate, Advanced)
8. Real-World Applications (industry contexts)
9. Common Challenges and Solutions
10. Integration with Learning Paths
11. Hands-On Labs (if applicable)
12. Additional Resources (documentation, community, related categories)
13. Feedback and Contributions
14. Version History
15. Standard Footer (AI attribution)

