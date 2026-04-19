# Amazon EC2 Image Builder (amazon-ec2-image-builder)
EC2 Image Builder simplifies the building, testing, and deployment of Virtual Machine and container images for use on AWS or on-premises. It provides an automated pipeline to create and maintain secure, up-to-date server images without requiring scripting expertise.

**URL:** [Visit APIs.json URL](https://aws.amazon.com/image-builder/)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - Amazon Web Services, Automation, AWS, Container Images, EC2, Image Building, Virtual Machine Images

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-19

## APIs

### Amazon EC2 Image Builder API
The EC2 Image Builder API provides programmatic access to create and manage image pipelines, recipes, components, infrastructure configurations, and distribution settings for automated VM and container image building workflows.

**Human URL:** [https://aws.amazon.com/image-builder/](https://aws.amazon.com/image-builder/)

#### Tags:

 - Automation, EC2, Image Building, Pipeline

#### Properties

- [Documentation](https://docs.aws.amazon.com/imagebuilder/latest/APIReference/Welcome.html)
- [OpenAPI](openapi/amazon-ec2-image-builder-openapi.yaml)
- [OpenAPI](https://api.apis.guru/v2/specs/amazonaws.com/imagebuilder/2019-12-02/openapi.yaml)
- [JSONSchema](json-schema/ec2-image-builder-image-pipeline-schema.json)
- [JSONLD](json-ld/amazon-ec2-image-builder-context.jsonld)
- [GettingStarted](https://docs.aws.amazon.com/imagebuilder/latest/userguide/getting-started-image-builder.html)
- [Pricing](https://aws.amazon.com/image-builder/pricing/)
- [FAQ](https://aws.amazon.com/image-builder/faqs/)
- [APIReference](https://docs.aws.amazon.com/imagebuilder/latest/APIReference/)
- [Authentication](https://docs.aws.amazon.com/imagebuilder/latest/APIReference/CommonParameters.html)
- [RateLimits](https://docs.aws.amazon.com/imagebuilder/latest/userguide/limits.html)
- [JSONSchema](json-schema/ec2-image-builder-account-aggregation-schema.json)
- [JSONStructure](json-structure/ec2-image-builder-account-aggregation-structure.json)
- [Example](examples/ec2-image-builder-account-aggregation-example.json)

## Common Properties

- [Portal](https://aws.amazon.com/)
- [DeveloperPortal](https://aws.amazon.com/developer/)
- [Documentation](https://docs.aws.amazon.com/imagebuilder/)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [Support](https://aws.amazon.com/support/)
- [Blog](https://aws.amazon.com/blogs/compute/tag/ec2-image-builder/)
- [GitHubOrganization](https://github.com/aws)
- [Console](https://console.aws.amazon.com/imagebuilder/)
- [SignUp](https://portal.aws.amazon.com/billing/signup)
- [Login](https://signin.aws.amazon.com/)
- [StatusPage](https://health.aws.amazon.com/health/status)
- [KnowledgeCenter](https://repost.aws/knowledge-center)
- [YouTube](https://www.youtube.com/user/AmazonWebServices)
- [StackOverflow](https://stackoverflow.com/questions/tagged/ec2-image-builder)
- [Contact](https://aws.amazon.com/contact-us/)
- [Security](https://aws.amazon.com/security/)
- [Compliance](https://aws.amazon.com/compliance/)
- [SpectralRules](rules/amazon-ec2-image-builder-spectral-rules.yml)
- [Vocabulary](vocabulary/amazon-ec2-image-builder-vocabulary.yaml)
- [NaftikoCapability](capabilities/ec2-image-builder-management.yaml)

## Features

| Name | Description |
|------|-------------|
| Automated Image Pipelines | Define end-to-end image creation workflows with build, test, and distribution phases without scripting expertise. |
| Image Recipes | Compose reusable image definitions from components including OS, software packages, and custom scripts. |
| Component Library | Catalog of pre-built AWS-managed and custom components for common software installation and configuration tasks. |
| Automated Testing | Run automated tests on images before distribution to validate software, security, and compliance requirements. |
| Multi-Region Distribution | Automatically distribute approved images to multiple AWS regions with configurable permissions. |
| Container Image Support | Build, test, and publish container images to Amazon ECR alongside traditional AMI workflows. |
| CIS and STIG Hardening | Built-in support for CIS Benchmarks and DISA STIG security hardening standards for compliance. |
| Scheduled Builds | Automatically rebuild images on a schedule to incorporate OS patches and security updates. |

## Use Cases

| Name | Description |
|------|-------------|
| Golden AMI Management | Create and maintain standardized, secure, and up-to-date base AMIs for all EC2 workloads across the organization. |
| Security Patch Automation | Automatically rebuild images with the latest OS security patches and distribute them across regions. |
| Compliance Image Hardening | Apply CIS or STIG security benchmarks to create compliant images for regulated environments. |
| Container Base Image Management | Maintain secure, up-to-date container base images and publish them to ECR for development teams. |
| Multi-Account Image Sharing | Build images in a central account and distribute them to multiple AWS accounts and regions. |

## Integrations

| Name | Description |
|------|-------------|
| Amazon EC2 | Produces AMIs that can be launched as EC2 instances across regions and accounts. |
| Amazon ECR | Publishes container images to ECR repositories as part of container image build pipelines. |
| AWS Systems Manager | Uses SSM Agent for image build and test execution on temporary build instances. |
| Amazon Inspector | Integrates with Amazon Inspector for automated vulnerability scanning of built images. |
| AWS Key Management Service | Encrypts AMIs and snapshots using KMS customer-managed keys during distribution. |
| AWS CloudTrail | Logs all Image Builder API calls for auditing and compliance tracking. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [Amazon Ec2 Image Builder](openapi/amazon-ec2-image-builder-openapi.yaml)

### JSON Schema

- [Ec2 Image Builder Account Aggregation](json-schema/ec2-image-builder-account-aggregation-schema.json)
- [Ec2 Image Builder Account Id](json-schema/ec2-image-builder-account-id-schema.json)
- [Ec2 Image Builder Account List](json-schema/ec2-image-builder-account-list-schema.json)
- [Ec2 Image Builder Additional Instance Configuration](json-schema/ec2-image-builder-additional-instance-configuration-schema.json)
- [Ec2 Image Builder Ami Distribution Configuration](json-schema/ec2-image-builder-ami-distribution-configuration-schema.json)
- [Ec2 Image Builder Ami List](json-schema/ec2-image-builder-ami-list-schema.json)
- [Ec2 Image Builder Ami Name String](json-schema/ec2-image-builder-ami-name-string-schema.json)
- [Ec2 Image Builder Ami](json-schema/ec2-image-builder-ami-schema.json)
- [Ec2 Image Builder Arn](json-schema/ec2-image-builder-arn-schema.json)
- [Ec2 Image Builder Boolean](json-schema/ec2-image-builder-boolean-schema.json)
- *...and 328 more*

### JSON Structure

- [Ec2 Image Builder Account Aggregation](json-structure/ec2-image-builder-account-aggregation-structure.json)
- [Ec2 Image Builder Account Id](json-structure/ec2-image-builder-account-id-structure.json)
- [Ec2 Image Builder Account List](json-structure/ec2-image-builder-account-list-structure.json)
- [Ec2 Image Builder Additional Instance Configuration](json-structure/ec2-image-builder-additional-instance-configuration-structure.json)
- [Ec2 Image Builder Ami Distribution Configuration](json-structure/ec2-image-builder-ami-distribution-configuration-structure.json)
- [Ec2 Image Builder Ami List](json-structure/ec2-image-builder-ami-list-structure.json)
- [Ec2 Image Builder Ami Name String](json-structure/ec2-image-builder-ami-name-string-structure.json)
- [Ec2 Image Builder Ami](json-structure/ec2-image-builder-ami-structure.json)
- [Ec2 Image Builder Arn](json-structure/ec2-image-builder-arn-structure.json)
- [Ec2 Image Builder Boolean](json-structure/ec2-image-builder-boolean-structure.json)
- *...and 328 more*

### JSON-LD

- [Amazon Ec2 Image Builder](json-ld/amazon-ec2-image-builder-context.jsonld)

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [Ec2 Image Builder](capabilities/shared/ec2-image-builder.yaml) — 50 operations

### Workflow Capabilities

| Workflow | Tools | Persona |
|----------|-------|---------|
| [Ec2 Image Builder Management](capabilities/ec2-image-builder-management.yaml) | 10 | managing EC2 Image Builder pipelines, recipes, and components for DevOps engineers |

## Vocabulary

- [Amazon EC2 Image Builder Vocabulary](vocabulary/amazon-ec2-image-builder-vocabulary.yaml) — Unified taxonomy mapping 54 resources, 11 actions, 1 workflows, and 1 personas across operational (OpenAPI) and capability (Naftiko) dimensions

## Rules

- [Amazon Ec2 Image Builder Spectral Rules](rules/amazon-ec2-image-builder-spectral-rules.yml) — 19 rules enforcing Amazon EC2 Image Builder API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com