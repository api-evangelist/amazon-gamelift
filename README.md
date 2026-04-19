# Amazon GameLift (amazon-gamelift)
Amazon GameLift is a dedicated game server hosting solution that deploys, operates, and scales cloud servers for multiplayer games. It provides low-latency, low-cost server infrastructure, eliminates operational overhead, and allows you to focus on creating great gaming experiences. The service includes FlexMatch for matchmaking, FleetIQ for optimized Spot Instance usage, and Realtime Servers for rapid game server deployment.

**URL:** [https://aws.amazon.com/gamelift/](https://aws.amazon.com/gamelift/)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - AWS, Cloud Computing, Game Servers, Gaming, Multiplayer, Matchmaking, FlexMatch, FleetIQ

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-19

## APIs

### Amazon GameLift API
The Amazon GameLift API provides programmatic access to create and manage fleets, game sessions, player sessions, matchmaking configurations, and game server groups for hosting multiplayer game servers. It includes operations for managed hosting resources, FlexMatch matchmaking, FleetIQ optimization, and Realtime Servers configuration.

**Human URL:** [https://aws.amazon.com/gamelift/](https://aws.amazon.com/gamelift/)

#### Tags:

 - Game Servers, Gaming, Multiplayer, Matchmaking, Fleets, Sessions

#### Properties

- [Documentation](https://docs.aws.amazon.com/gamelift/latest/apireference/Welcome.html)
- [OpenAPI](openapi/amazon-gamelift-openapi.yaml)
- [GettingStarted](https://aws.amazon.com/gamelift/getting-started/)
- [Pricing](https://aws.amazon.com/gamelift/pricing/)
- [FAQ](https://aws.amazon.com/gamelift/faq/)
- [APIReference](https://docs.aws.amazon.com/gamelift/latest/apireference/Welcome.html)
- [Authentication](https://docs.aws.amazon.com/gamelift/latest/apireference/CommonParameters.html)
- [JSONSchema](json-schema/gamelift-fleet-schema.json)
- [JSONLD](json-ld/amazon-gamelift-context.jsonld)

## Common Properties

- [Portal](https://aws.amazon.com/gamelift/)
- [Documentation](https://docs.aws.amazon.com/gamelift/)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Blog](https://aws.amazon.com/blogs/gametech/tag/amazon-gamelift/)
- [GitHubOrganization](https://github.com/aws)
- [Console](https://console.aws.amazon.com/gamelift/)
- [SignUp](https://portal.aws.amazon.com/billing/signup)
- [StatusPage](https://health.aws.amazon.com/health/status)
- [Contact](https://aws.amazon.com/contact-us/)
- [SDK](https://aws.amazon.com/developer/tools/)
- [CLI](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/gamelift/index.html)

## Features

| Name | Description |
|------|-------------|
| Managed Game Server Hosting | Fully managed service to deploy, operate, and scale dedicated game servers with automatic lifecycle management for game and player sessions. |
| FlexMatch Matchmaking | Customizable matchmaking service that connects up to 200 players into single game sessions based on configurable matching rules. |
| FleetIQ Optimization | Optimizes use of low-cost Spot Instances for game hosting, improving viability and reducing costs while maintaining performance. |
| Realtime Servers | Rapidly configurable game server framework with core Amazon GameLift infrastructure built in for quick deployment. |
| Auto Scaling | Automatically scales fleet capacity up to 9,000 servers per minute to balance player demand and hosting costs. |
| Multi-Region Deployment | Deploy game servers across multiple AWS regions with global queues for optimal player latency and resiliency. |
| Game Session Queues | Multi-fleet, multi-region queues that use FleetIQ algorithms to prioritize game session placements based on latency, cost, and availability. |
| VPC Peering | Create and manage VPC peering connections between GameLift hosting resources and other AWS resources. |
| Alias Management | Create fleet aliases to simplify game server transitions and updates without changing client configurations. |

## Use Cases

| Name | Description |
|------|-------------|
| Multiplayer Game Launches | Deploy and scale game servers for launch day without uncertainty about player demand using predictive autoscaling. |
| Session-Based Multiplayer | Host dedicated game servers for real-time multiplayer games with low latency and high performance. |
| Player Matchmaking | Use FlexMatch to create fair, customizable matchmaking for players based on skill level, region, and other criteria. |
| Cost-Optimized Hosting | Leverage FleetIQ with Spot Instances to reduce game hosting costs while maintaining reliability. |
| Global Game Distribution | Deploy game servers across multiple AWS regions to minimize player latency worldwide. |

## Integrations

| Name | Description |
|------|-------------|
| AWS EC2 | GameLift uses EC2 instances as the underlying compute for managed game server hosting. |
| AWS Auto Scaling | Integrates with Auto Scaling groups for FleetIQ standalone deployment. |
| Amazon CloudWatch | Monitor fleet metrics, game session activity, and performance data through CloudWatch. |
| Amazon SNS | Receive notifications for matchmaking events and game session placement status. |
| AWS IAM | Use IAM roles and policies to control access to GameLift resources and operations. |
| Amazon S3 | Store and retrieve game server build files using S3 buckets for fleet deployment. |
| AWS CloudFormation | Provision and manage GameLift resources using infrastructure-as-code templates. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [Amazon GameLift OpenAPI](openapi/amazon-gamelift-openapi.yaml)

### JSON Schema

472 schema files in [json-schema/](json-schema/)

### JSON Structure

472 structure files in [json-structure/](json-structure/)

### JSON-LD

- [Amazon GameLift Context](json-ld/amazon-gamelift-context.jsonld)

### Examples

472 example files in [examples/](examples/)

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [Amazon GameLift](capabilities/shared/amazon-gamelift.yaml) — 104 operations for game server hosting and management

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Amazon GameLift Game Operations](capabilities/amazon-gamelift-game-operations.yaml) | Amazon GameLift | 20 | Game Developer, DevOps Engineer, Game Operations Team |

## Vocabulary

- [Amazon GameLift Vocabulary](vocabulary/amazon-gamelift-vocabulary.yaml) — Unified taxonomy mapping 10 resources, 9 actions, 1 workflow, and 3 personas across operational (OpenAPI) and capability (Naftiko) dimensions

## Rules

- [Amazon GameLift Spectral Rules](rules/amazon-gamelift-spectral-rules.yml) — 18 rules across 8 categories enforcing Amazon GameLift API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
