---
title: "CAIRA Advanced - Learning Katas"
description: "Expert-level katas for production-ready CAIRA deployments with advanced security, customization, and lifecycle management"
ms.topic: kata-category
ms.date: 12/15/2024
tags:
  - caira
  - advanced
  - production
  - security
  - lifecycle-management
---

# CAIRA Advanced

Develop expertise in production-ready CAIRA deployments with expert-level katas covering security hardening, infrastructure customization, and lifecycle management for enterprise AI platforms.

## Category Overview

The CAIRA Advanced category builds upon foundational CAIRA skills to prepare you for real-world production deployments. These expert-level katas focus on security best practices, organizational customization requirements, and operational excellence for Azure AI Foundry infrastructure.

You'll progress from customizing CAIRA for organizational governance to implementing zero-trust security architectures, and finally to managing the complete infrastructure lifecycle. Each kata presents realistic production scenarios requiring architectural decisions, security trade-offs, and documentation practices that mirror enterprise deployment requirements.

The technologies covered—private networking, managed identities, RBAC, Terraform state management, and ADR documentation—represent industry-standard approaches for production AI infrastructure. By completing this category, you'll develop proficiency in deploying and maintaining enterprise-grade AI platforms that meet compliance, security, and operational requirements.

## Prerequisites

### Required Knowledge

- Terraform fundamentals (init, plan, apply workflow)
- Azure AI Foundry basic architecture concepts
- Azure RBAC and identity management basics
- Cloud networking fundamentals (VNets, subnets, private endpoints)
- Infrastructure-as-code principles
- Azure CLI command-line operations

### Required Tools

- Docker Desktop (latest stable version)
- VS Code with Dev Containers extension
- Azure CLI (v2.50 or higher)
- Git (v2.30 or higher)
- Azure subscription with Contributor + User Access Administrator roles

### Recommended Preparation

- [Kata 200: Devcontainer & Foundry Basic Deployment](../caira-fundamentals/200-devcontainer-foundry-basic-deployment.md) - Essential prerequisite
- [Kata 300: Troubleshooting CAIRA Deployments](../caira-fundamentals/300-troubleshooting-caira-deployments.md) - Helpful for debugging
- Understanding of Azure Well-Architected Framework principles
- Familiarity with zero-trust security concepts

## Learning Path

Progressive learning path from customization to production security:

```text
[Kata 400: Customization] → [Kata 450: Production Security]
         ↓                              ↓
  Org Requirements              Zero-Trust Architecture
  Naming & Tagging              Private Networking
  SKU Selection                 Managed Identity
  Governance                    RBAC & Compliance
```

**Recommended Sequence**: Complete Kata 400 first to understand customization, then Kata 450 for production security hardening.

## Category Katas

### [Kata 400: Customizing CAIRA for Your Requirements](./400-customizing-caira-for-your-requirements.md)

**Difficulty**: ⭐⭐⭐⭐ Expert (4/5) | **Time**: 45 minutes

Learn to customize CAIRA deployments to meet organizational governance requirements including naming conventions, tagging strategies, SKU selection, and data residency compliance.

**You'll Learn**:

- Implement organizational naming conventions and tagging frameworks
- Override default SKUs for performance and cost optimization
- Configure data residency requirements (region selection)
- Document customization decisions with rationale
- Validate customizations without breaking module contracts

**Prerequisites**: [Kata 200: Devcontainer & Foundry Basic Deployment](../caira-fundamentals/200-devcontainer-foundry-basic-deployment.md)

**Real-World Context**: Your organization requires Canada Central deployment for data residency, "proj-env-service-####" naming format, cost center tags, Premium AI Search tier, and Application Insights integration.

---

### [Kata 450: Securing CAIRA for Production](./450-securing-caira-production.md)

**Difficulty**: ⭐⭐⭐⭐ Expert (4/5) | **Time**: 45 minutes

Deploy CAIRA with zero-trust security including private networking, managed identities, least-privilege RBAC, and comprehensive security documentation for compliance review.

**You'll Learn**:

- Deploy foundry_standard_private architecture with complete network isolation
- Configure private endpoints and disable public network access
- Implement managed identities and disable API key authentication
- Apply least-privilege RBAC principles
- Document security decisions in ADR format
- Validate security posture against Azure Security Benchmark

**Prerequisites**: [Kata 200: Devcontainer & Foundry Basic Deployment](../caira-fundamentals/200-devcontainer-foundry-basic-deployment.md), basic Azure networking concepts

**Real-World Context**: Healthcare AI platform requiring HIPAA compliance with zero-trust principles: no public endpoints, managed identities only, complete network isolation, and comprehensive audit logging.

## Kata Comparison Matrix

| Kata                                                                                          | Difficulty      | Time   | Technologies                                           | Scaffolding  | Prerequisites                                                                           |
|-----------------------------------------------------------------------------------------------|-----------------|--------|--------------------------------------------------------|--------------|-----------------------------------------------------------------------------------------|
| [Kata 400: Customizing CAIRA](./400-customizing-caira-for-your-requirements.md)              | ⭐⭐⭐⭐ (4/5)     | 45 min | Terraform, CAIRA, Azure governance, tagging frameworks | Medium-Heavy | Kata 200 (Basic Deployment)                                                            |
| [Kata 450: Securing CAIRA](./450-securing-caira-production.md)                                | ⭐⭐⭐⭐ (4/5)     | 45 min | Private Link, Managed Identity, RBAC, NSGs, ADR docs   | Medium-Heavy | Kata 200 (Basic Deployment), Azure networking basics                                   |

## Suggested Learning Sequences

### For Production Readiness (Recommended)

1. [Kata 400: Customizing CAIRA](./400-customizing-caira-for-your-requirements.md) - Organizational requirements
1. [Kata 450: Securing CAIRA](./450-securing-caira-production.md) - Zero-trust security

**Total Time**: 1.5 hours | **Outcome**: Production-ready deployment with organizational governance and security hardening

### For Security-First Organizations (Compliance-Heavy)

1. [Kata 450: Securing CAIRA](./450-securing-caira-production.md) - Start with security fundamentals
1. [Kata 400: Customizing CAIRA](./400-customizing-caira-for-your-requirements.md) - Add organizational governance

**Total Time**: 1.5 hours | **Outcome**: Security-first deployment with governance integration

## Real-World Applications

### Healthcare & Life Sciences

Deploy AI-powered clinical decision support tools with HIPAA compliance: private networking for PHI protection, managed identities for zero-trust authentication, comprehensive audit logging for regulatory requirements. Kata 450 provides essential security patterns.

### Financial Services

Build fraud detection and risk analysis platforms meeting PCI-DSS requirements: network isolation, least-privilege RBAC, encryption at rest and in transit. Combine Kata 400 (organizational tagging) with Kata 450 (security hardening).

### Government & Public Sector

Implement citizen-facing AI services with FedRAMP/NIST compliance: data residency controls, comprehensive documentation (ADRs), secure state management. All three katas provide critical compliance capabilities.

### Manufacturing & Supply Chain

Deploy edge AI for predictive maintenance and quality control: customized naming conventions for multi-site deployments (Kata 400), secure remote management (Kata 450), lifecycle management for fleet updates (Kata 500).

### Retail & E-Commerce

Build personalized recommendation engines with customer data privacy: private endpoints for customer PII protection, managed identities for secure data access, documented security decisions for privacy compliance.

## Common Challenges and Solutions

### Challenge: Balancing Security with Developer Productivity

**Scenario**: Private networking blocks developer access to AI services, causing friction.

**Solution**:

- Implement Azure Bastion or VPN for secure developer access
- Use Azure Private DNS resolver for name resolution from on-premises
- Consider hybrid approach: dev/test with public endpoints, production with private
- Document trade-offs in ADR (covered in Kata 450)

### Challenge: Customization Breaking Module Updates

**Scenario**: Heavy customization prevents upgrading to newer CAIRA module versions.

**Solution**:

- Document all customizations with rationale (Kata 400)
- Test module updates in non-production first
- Use variable overrides rather than forking modules
- Contribute improvements back to CAIRA repository
- Balance customization needs with maintainability

### Challenge: Compliance Documentation Requirements

**Scenario**: Security team requires comprehensive documentation for audit and compliance review.

**Solution**:

- Create ADRs for all architectural decisions (Kata 450)
- Maintain security hardening checklist with validation
- Document RBAC assignments with justification
- Keep runbooks updated for operational procedures
- Use Infrastructure-as-Code for auditable change history

### Challenge: Cost Optimization vs. Performance Requirements

**Scenario**: Default SKUs exceed budget but custom SKUs risk insufficient performance.

**Solution**:

- Start with recommended SKUs, monitor utilization (Kata 400)
- Use Azure Cost Management for tracking and alerting
- Implement tagging strategy for cost allocation visibility
- Test performance with realistic workloads before downgrading
- Document SKU decisions with performance/cost trade-offs

## Integration with Learning Paths

### Azure AI Foundry Deployment Path

This category represents the **Expert** tier of the Azure AI Foundry deployment learning path:

- **Foundation**: [CAIRA Fundamentals Category](../caira-fundamentals/) (Katas 100-350)
- **Advanced**: This category (Katas 400-450) ← You are here
- **Mastery**: Production case studies and team collaboration labs

### Security & Compliance Path

Kata 450 integrates with broader security learning:

- **Prerequisites**: Azure networking fundamentals, zero-trust principles
- **This Kata**: CAIRA production security implementation
- **Next Steps**: Azure Security Benchmark validation, penetration testing, incident response

### Platform Engineering Path

Complete category supports platform engineering competencies:

- **Infrastructure-as-Code**: Terraform customization and lifecycle management
- **Security Engineering**: Zero-trust architecture implementation
- **Operational Excellence**: State management, backup procedures, team collaboration

## Hands-On Labs

### Training Lab: Enterprise CAIRA Deployment

**Duration**: 3-4 hours | **Difficulty**: Expert

Comprehensive lab combining both katas into a realistic enterprise deployment scenario:

1. **Phase 1**: Deploy customized CAIRA with organizational requirements (Kata 400 skills)
1. **Phase 2**: Harden security with private networking and managed identities (Kata 450 skills)
1. **Phase 3**: Validate against Azure Well-Architected Framework
1. **Phase 4**: Deploy sample AI application on secured infrastructure

**Prerequisites**: Completed both katas in this category

**Link**: [Enterprise CAIRA Deployment Lab](../../training-labs/enterprise-caira-deployment/) *(Coming Soon)*

## Additional Resources

### Official Documentation

- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-studio/)
- [Azure Private Link Documentation](https://learn.microsoft.com/azure/private-link/)
- [Terraform Azure Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
- [Azure Security Benchmark](https://learn.microsoft.com/security/benchmark/azure/)
- [Azure Well-Architected Framework](https://learn.microsoft.com/azure/well-architected/)

### Community Resources

- [CAIRA GitHub Repository](https://github.com/microsoft/CAIRA) - Latest reference architectures and updates
- [Azure Architecture Center](https://learn.microsoft.com/azure/architecture/) - Design patterns and best practices
- [Terraform Best Practices](https://www.terraform-best-practices.com/) - Community-driven guidelines

### Related Categories

- [CAIRA Fundamentals](../caira-fundamentals/) - Foundation-level CAIRA skills (prerequisite)
- [Troubleshooting](../troubleshooting/) - Diagnostic and problem-solving techniques
- [Prompt Engineering](../prompt-engineering/) - AI interaction patterns for development

## Feedback and Contributions

We continuously improve these learning katas based on community feedback.

**Share Your Experience**:

- What worked well in your learning journey?
- Which concepts were challenging and need better explanation?
- What real-world scenarios should we add?

**Contribute**:

- Submit issues for errors or unclear instructions
- Propose new kata ideas for advanced CAIRA topics
- Share your production deployment stories and lessons learned

[Submit Feedback](https://github.com/microsoft/CAIRA/issues) | [Contribution Guidelines](../../../../CONTRIBUTING.md)

## Version History

| Version | Date       | Changes                                                                 |
|---------|------------|-------------------------------------------------------------------------|
| 1.0.0   | 12/18/2024 | Initial release with Kata 400 and 450 (Customization and Security)     |

---

<!-- markdownlint-disable MD036 -->
*🤖 Crafted with precision by ✨Copilot following brilliant human instruction,
then carefully refined by our team of discerning human reviewers.*
<!-- markdownlint-enable MD036 -->
