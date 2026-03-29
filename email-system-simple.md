# Email System - Simplified Flow

```mermaid
flowchart TD
    %% Entry Points
    A[User Subscribes<br/>Website/Book/Event]
    
    %% Subscription
    B[Subscription Handler Lambda]
    C[(Subscribers Table)]
    D[(Enrollments Table)]
    
    %% Campaign Storage
    E[(Campaigns Table<br/>7 Campaign Groups)]
    
    %% Processing
    F[EventBridge<br/>Daily 9 AM]
    G[Drip Processor Lambda<br/>Check Due Emails]
    H[SQS Queue]
    I[Email Sender Lambda<br/>Add Tracking]
    
    %% Delivery
    J[Amazon SES<br/>Send Email]
    K[User Receives Email]
    
    %% Tracking
    L[User Opens/Clicks]
    M[CloudFront + API Gateway]
    N[(Events Table<br/>Opens & Clicks)]
    
    %% Analytics
    O[Campaign Manager<br/>Analytics Dashboard]
    
    %% Main Flow
    A --> B
    B --> C
    B --> D
    E -.-> G
    
    F -->|Trigger| G
    G -->|Query| D
    G -->|Get Content| E
    G -->|Queue Email| H
    
    H --> I
    I -->|Get Subscriber| C
    I --> J
    I -->|Log Sent| N
    
    J --> K
    K --> L
    L --> M
    M -->|Log Event| N
    
    N --> O
    D --> O
    E --> O
    
    %% Styling
    style A fill:#e1f5ff
    style B fill:#fff3cd
    style G fill:#fff3cd
    style I fill:#fff3cd
    style C fill:#d1fae5
    style D fill:#d1fae5
    style E fill:#d1fae5
    style N fill:#d1fae5
    style J fill:#fee2e2
    style M fill:#fce7f3
```

## Simple Flow Summary

1. **User subscribes** → Subscription Handler creates subscriber + enrollment
2. **Daily trigger** → Drip Processor checks which emails are due
3. **Queue email** → SQS decouples processing from sending
4. **Send email** → Email Sender adds tracking pixels/links and sends via SES
5. **User interacts** → Opens/clicks tracked via CloudFront → logged to Events table
6. **View analytics** → Campaign Manager shows stats from Events + Enrollments

## Key Components

- **3 Lambda Functions**: Subscription Handler, Drip Processor, Email Sender
- **4 DynamoDB Tables**: Subscribers, Enrollments, Campaigns, Events
- **1 SQS Queue**: Decouples drip processing from email sending
- **EventBridge**: Daily trigger at 9 AM UTC
- **CloudFront + API Gateway**: Tracking pipeline for opens/clicks
- **Amazon SES**: Email delivery service

## Campaign Groups (7 emails each)

1. `pre-purchase-book-sequence` - Free survival kit signups
2. `post-purchase-sequence` - Book buyers
3. `election-map-transition-sequence` - Election map users
4. `general-newsletter-sequence` - Website subscribers (includes 2 book promo emails)
