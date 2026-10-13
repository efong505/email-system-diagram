# AWS Email Marketing System — Architecture Case Study

> **Employer-safe architecture summary.** Environment-specific domains, distribution IDs, API identifiers, user IDs, bucket names, and business campaign names have been generalized.

```mermaid
graph TB
    subgraph "Entry Points"
        A1[Website Newsletter]
        A2[Landing Page]
        A3[Event Registration]
        A4[Post-Purchase Enrollment]
        A5[Campaign Manager]
    end

    subgraph "Subscription Handling"
        B[Subscription Handler Lambda]
        B1[Create / Update Subscriber]
        B2[Confirmation Workflow]
        B3[Campaign Enrollment]
        B4[Manual Trigger]
    end

    subgraph "DynamoDB"
        D1[(Subscribers)]
        D2[(Campaigns)]
        D3[(Drip Enrollments)]
        D4[(Events)]
    end

    subgraph "Processing"
        F1[EventBridge Schedule]
        F2[Drip Processor Lambda]
        F3[SQS Queue]
        F4[Email Sender Lambda]
    end

    subgraph "Delivery"
        G1[Amazon SES]
        G2[Tracked Email]
    end

    subgraph "Tracking"
        H1[CloudFront]
        H2[API Gateway]
        H3[Tracking Lambda]
        H4[(Events)]
    end

    subgraph "Analytics"
        I1[Campaign Manager]
        I2[Analytics Dashboard]
    end

    A1 --> B
    A2 --> B
    A3 --> B
    A4 --> B3
    A5 --> B4

    B --> D1
    B --> D3
    B3 --> D3

    F1 --> F2
    F2 --> D3
    F2 --> D2
    F2 --> F3
    F3 --> F4
    F4 --> D1
    F4 --> D2
    F4 --> G1
    F4 --> D4

    G1 --> G2
    G2 --> H1
    H1 --> H2
    H2 --> H3
    H3 --> D4

    D2 --> I1
    D3 --> I1
    D4 --> I2
```

## Architecture Summary

This design uses managed AWS services to separate subscription handling, campaign scheduling, message queuing, delivery, tracking, and analytics.

### Core services

- **AWS Lambda** — subscription, drip-processing, sending, tracking, and API logic
- **Amazon DynamoDB** — subscribers, campaigns, enrollments, and event records
- **Amazon SQS** — decouples scheduling from email sending
- **Amazon EventBridge** — scheduled campaign processing
- **Amazon SES** — email delivery
- **Amazon CloudFront + API Gateway** — tracked open/click request path
- **Static management UI** — campaign and analytics administration

## Processing Model

1. A user subscribes or is enrolled through an approved entry point.
2. Subscriber and enrollment records are written to DynamoDB.
3. EventBridge invokes the drip processor on a schedule.
4. Due messages are placed on SQS.
5. The sender Lambda retrieves campaign/subscriber data, adds tracking, and sends through SES.
6. Open/click requests flow through CloudFront and API Gateway.
7. Tracking events are stored in DynamoDB and surfaced through analytics.

## Engineering Considerations

- asynchronous processing through SQS
- idempotent enrollment and send logic
- least-privilege IAM between functions and data stores
- bounce/complaint handling through SES events
- opt-out and consent enforcement
- retry / DLQ behavior for failed sends
- observability through CloudWatch metrics and structured logs
- separation between campaign content, enrollment state, and event telemetry

## Evidence Boundary

This repository is an architecture case study. It demonstrates system decomposition, AWS service selection, event-driven processing, and operational design. It does not expose live customer data, account identifiers, private infrastructure IDs, or production credentials.
