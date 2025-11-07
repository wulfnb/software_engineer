# System Architecture Interview Tasks & Solutions

This repository contains comprehensive system architecture tasks and solutions for technical interviews. Each solution includes detailed Mermaid.js diagrams and architectural explanations.

## Task 1: URL Shortening Service (like TinyURL)

### Requirements
- Shorten long URLs to short codes
- Redirect short URLs to original URLs
- Handle 100M+ URLs
- High availability and low latency

### Architecture Diagram

```mermaid
graph TB
    subgraph Clients
        A[Web/Mobile App]
        B[Browser]
    end

    subgraph Application Layer
        C[Load Balancer]
        D[API Gateway]
        E[URL Service]
    end

    subgraph Cache Layer
        F[Redis Cluster]
    end

    subgraph Data Storage
        G[MySQL Sharded]
        H[Snowflake ID Generator]
        I[Analytics DB]
    end

    A --> C
    B --> C
    C --> D
    D --> E
    E --> F
    E --> G
    E --> H
    F --> E
    G --> E
    H --> E
    
    style A fill:#e1f5fe
    style B fill:#e1f5fe
    style F fill:#fff3e0
    style G fill:#f3e5f5
```

### Key Components
1. **Base62 Encoding**: For short, readable URLs
2. **Distributed ID Generation**: Snowflake for unique IDs across servers
3. **Caching Strategy**: Redis for hot data, reducing database load
4. **Database Sharding**: Scale horizontally by sharding on short code

### Data Flow
1. **Shorten URL**: Client → Load Balancer → API Server → ID Generator → DB + Cache
2. **Redirect**: Client → Load Balancer → API Server → Cache → DB (if cache miss)

---

## Task 2: Real-time Chat Application

### Requirements
- Support 1M+ concurrent users
- Deliver messages in <100ms
- Store chat history
- Support multiple chat rooms

### Architecture Diagram

```mermaid
graph TB
    subgraph Clients
        A[Web Client]
        B[Mobile App]
    end

    subgraph Backend Services
        C[Load Balancer]
        D[API Gateway]
        E[WebSocket Servers]
        F[Message Queue]
        G[Chat Service]
    end

    subgraph Data Layer
        H[Redis Cache]
        I[Cassandra/MongoDB]
        J[Presence Service]
    end

    A --> C
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    G --> H
    G --> I
    G --> J
    H --> G
    I --> G
    J --> G
    
    style E fill:#fff3e0
    style F fill:#e8f5e8
    style H fill:#fff3e0
```

### Real-time Message Flow
1. User A sends message → WebSocket Server → Message Queue → Chat Service
2. Chat Service → Database (store) + Presence Service (get online users)
3. Chat Service → WebSocket Servers → User B, User C, etc.

---

## Task 3: E-commerce Inventory System

### Requirements
- Handle high concurrent orders
- Prevent overselling
- Real-time inventory updates
- Support flash sales

### Architecture Diagram

```mermaid
graph TB
    subgraph Clients
        A[Web App]
        B[Mobile App]
    end

    subgraph Application Layer
        C[Load Balancer]
        D[API Gateway]
        E[Inventory Service]
        F[Order Service]
    end

    subgraph Message Layer
        G[RabbitMQ/Kafka]
    end

    subgraph Data Layer
        H[Redis Cache]
        I[PostgreSQL]
        J[Cassandra]
    end

    A --> C
    B --> C
    C --> D
    D --> E
    D --> F
    E --> G
    E --> H
    E --> I
    F --> G
    F --> I
    G --> E
    G --> F
    H --> E
    I --> E
    I --> F
    
    style H fill:#fff3e0
    style G fill:#e8f5e8
```

### Key Features
1. **Redis for Atomic Operations**: Prevents race conditions using Lua scripts
2. **Circuit Breaker Pattern**: Prevents system overload during flash sales
3. **Event Sourcing**: Track all inventory changes
4. **Read/Write Separation**: Separate databases for reads and writes

---

## Task 4: Distributed File Storage System

### Requirements
- Store 1PB+ of files
- High durability (99.999999999%)
- Fast upload/download
- Cost-effective storage

### Architecture Diagram

```mermaid
graph TB
    subgraph Clients
        A[Web App]
        B[Mobile App]
        C[TV/OTT Devices]
    end

    subgraph Application Layer
        D[Load Balancer]
        E[API Gateway]
        F[File Service]
    end

    subgraph Metadata Layer
        G[Redis Cache]
        H[Cassandra Metadata DB]
    end

    subgraph Storage Layer
        I[Storage Node 1]
        J[Storage Node 2]
        K[Storage Node N]
    end

    A --> D
    B --> D
    C --> D
    D --> E
    E --> F
    F --> G
    F --> H
    F --> I
    F --> J
    F --> K
    G --> F
    H --> F
    
    style I fill:#f3e5f5
    style J fill:#f3e5f5
    style K fill:#f3e5f5
```

### File Upload Flow
1. Client → API Gateway → File Service
2. File Service splits file into chunks
3. Each chunk replicated to 3 storage nodes (consistent hashing)
4. Metadata stored in Cassandra + Cache

---

## Task 5: Food Delivery System (like UberEats)

### Requirements
- Real-time order tracking
- Restaurant and delivery partner management
- Payment processing
- Location-based services

### Architecture Diagram

```mermaid
graph TB
    subgraph Clients
        A[Customer App]
        B[Restaurant App]
        C[Delivery Partner App]
    end

    subgraph Application Layer
        D[Load Balancer]
        E[API Gateway]
    end

    subgraph Microservices
        F[Order Service]
        G[Restaurant Service]
        H[Delivery Service]
        I[Payment Service]
        J[Location Service]
        K[Notification Service]
    end

    subgraph Data Layer
        L[Redis Cache]
        M[PostgreSQL]
        N[Elasticsearch]
    end

    subgraph External Integrations
        O[Maps API]
        P[Payment Gateway]
        Q[SMS/Push Notifications]
        R[Analytics Platform]
    end

    A --> D
    B --> D
    C --> D
    D --> E
    E --> F
    E --> G
    E --> H
    E --> I
    E --> J
    E --> K
    
    F --> L
    F --> M
    G --> L
    G --> M
    H --> L
    H --> M
    I --> M
    J --> L
    K --> Q
    
    F --> O
    I --> P
    M --> R
    N --> R
    
    style F fill:#e1f5fe
    style H fill:#e8f5e8
    style I fill:#fff3e0
```

### Key Workflows
1. **Order Placement**: Customer → Order Service → Restaurant Service → Payment Service
2. **Delivery Matching**: Order Service → Delivery Service → Location-based matching
3. **Real-time Tracking**: Tracking Service → WebSocket → Customer/Delivery Partner Apps

---

## Task 6: Video Streaming Service (like YouTube/Netflix)

### Requirements
- Support millions of concurrent viewers
- Adaptive bitrate streaming
- Global content delivery
- Video processing pipeline

### Architecture Diagram

```mermaid
graph TB
    subgraph Clients
        A[Web Browser]
        B[Mobile App]
        C[TV/OTT Devices]
    end

    subgraph CDN Layer
        D[CDN Edge NA]
        E[CDN Edge EU]
        F[CDN Edge Asia]
        G[CDN Edge SA]
    end

    subgraph Origin Servers
        H[Video Processing Pipeline]
        I[API Gateway]
        J[Metadata Service]
        K[User Service]
        L[Recommendation Service]
    end

    subgraph Storage Layer
        M[Hot Storage SSD]
        N[Cold Storage S3/Glacier]
        O[Database Cluster]
    end

    A --> D
    B --> E
    C --> F
    D --> H
    E --> H
    F --> H
    G --> H
    
    H --> M
    H --> N
    I --> O
    J --> O
    K --> O
    L --> O
    
    style D fill:#e8f5e8
    style E fill:#e8f5e8
    style F fill:#e8f5e8
    style G fill:#e8f5e8
    style M fill:#fff3e0
    style N fill:#f3e5f5
```

### Video Delivery Flow
1. **Upload**: Client → Ingest Service → Transcoding Pipeline → Multiple Quality Versions
2. **Storage**: Processed videos → Hot Storage (popular) + Cold Storage (archived)
3. **Delivery**: CDN Edge Nodes cache videos based on regional popularity
4. **Adaptive Streaming**: Client automatically switches quality based on bandwidth

---

## Task 7: Banking/Payment System

### Requirements
- ACID compliance for transactions
- High security and fraud detection
- Regulatory compliance
- 99.99%+ availability

### Architecture Diagram

```mermaid
graph TB
    subgraph Channels
        A[Mobile Banking]
        B[Internet Banking]
        C[ATM/Branch Systems]
    end

    subgraph Security Layer
        D[WAF & Rate Limiter]
        E[API Gateway]
        F[Load Balancer]
    end

    subgraph Core Banking Services
        G[Account Service]
        H[Transaction Service]
        I[Payment Service]
        J[Fraud Detection]
        K[Loan Service]
        L[Reporting Service]
    end

    subgraph Data Layer
        M[Oracle/DB2 Database]
        N[Redis Cache]
        O[Message Queue]
        P[Audit & Logging]
    end

    subgraph External Systems
        Q[SWIFT/Fedwire]
        R[Card Networks]
        S[Credit Bureaus]
        T[Regulatory Reporting]
    end

    A --> D
    B --> D
    C --> D
    D --> E
    E --> F
    F --> G
    F --> H
    F --> I
    F --> J
    F --> K
    F --> L
    
    G --> M
    G --> N
    G --> O
    H --> M
    H --> O
    I --> M
    I --> O
    I --> Q
    I --> R
    J --> S
    L --> T
    M --> P
    O --> P
    
    style M fill:#ffebee
    style J fill:#fff3e0
    style P fill:#e8f5e8
```

### Key Features
1. **ACID Compliance**: Strong consistency for financial transactions
2. **Fraud Detection**: Real-time pattern analysis and machine learning
3. **Audit Trail**: Complete transaction logging for compliance
4. **High Availability**: 99.99%+ uptime requirements

---

## Task 8: IoT Platform (Smart Home/Industrial IoT)

### Requirements
- Handle millions of connected devices
- Real-time data processing
- Edge computing capabilities
- Predictive maintenance

### Architecture Diagram

```mermaid
graph TB
    subgraph IoT Devices
        A[Sensors & Actuators]
        B[Cameras & Smart Devices]
        C[Industrial Equipment]
    end

    subgraph Edge Layer
        D[Protocol Adapters]
        E[Edge Processing]
        F[Local Analytics]
    end

    subgraph Cloud Platform
        G[IoT Core Service]
        H[Stream Processing]
        I[Device Management]
        J[Message Broker]
    end

    subgraph Data & Analytics
        K[Time Series Database]
        L[Data Warehouse]
        M[ML/AI Services]
        N[Dashboard & Reporting]
    end

    subgraph External Systems
        O[Mobile Apps]
        P[CRM Systems]
        Q[ERP Systems]
    end

    A --> D
    B --> D
    C --> D
    D --> E
    E --> F
    D --> G
    E --> G
    
    G --> H
    G --> I
    G --> J
    
    H --> K
    H --> L
    K --> M
    L --> M
    M --> N
    
    G --> O
    G --> P
    G --> Q
    
    style D fill:#e8f5e8
    style E fill:#e8f5e8
    style K fill:#fff3e0
    style M fill:#f3e5f5
```

### Data Flow
1. **Device to Cloud**: Sensors → Protocol Adapters → IoT Core → Stream Processing
2. **Real-time Processing**: Kafka/Kinesis → Spark Streaming → Real-time Alerts
3. **Batch Analytics**: Data Warehouse → ML Services → Predictive Maintenance
4. **Command & Control**: Dashboard → IoT Core → Actuators/Devices

---

## Task 9: Multi-player Gaming Platform

### Requirements
- Low latency real-time gameplay
- Matchmaking services
- Virtual economy system
- Anti-cheat mechanisms

### Architecture Diagram

```mermaid
graph TB
    subgraph Game Clients
        A[PC/Console]
        B[Mobile Devices]
        C[Web Browser]
    end

    subgraph Game Servers
        D[Regional Game Servers]
        E[Game Logic Server]
        F[Physics Engine]
        G[Matchmaking Service]
        H[Session Management]
    end

    subgraph Platform Services
        I[User Service]
        J[Social Service]
        K[Economy Service]
        L[Leaderboard Service]
    end

    subgraph Data & Analytics
        M[Redis Cluster]
        N[Cassandra]
        O[Analytics Pipeline]
        P[Data Lake]
    end

    subgraph External Integrations
        Q[Payment Gateways]
        R[Push Notifications]
        S[Anti-Cheat Systems]
        T[Content Delivery Network]
    end

    A --> D
    B --> D
    C --> D
    D --> E
    D --> F
    D --> G
    D --> H
    
    G --> I
    H --> I
    I --> J
    I --> K
    I --> L
    
    E --> M
    F --> M
    I --> M
    I --> N
    K --> N
    L --> N
    
    M --> O
    N --> O
    O --> P
    
    K --> Q
    J --> R
    E --> S
    D --> T
    
    style D fill:#e1f5fe
    style G fill:#e8f5e8
    style M fill:#fff3e0
    style S fill:#ffebee
```

### Key Features
1. **Low Latency**: Regional game servers for real-time gameplay
2. **Session Management**: Handle player connections and disconnections
3. **Anti-Cheat**: Real-time pattern detection and validation
4. **Economy System**: Secure virtual currency and item transactions

---

## Evaluation Criteria for System Design Interviews

### 1. System Scalability
- Horizontal vs vertical scaling strategies
- Database partitioning/sharding approaches
- Load balancing techniques

### 2. Data Consistency
- CAP theorem tradeoffs
- Consistency models chosen (strong, eventual)
- Conflict resolution strategies

### 3. Fault Tolerance
- Failure handling strategies
- Redundancy and backup plans
- Disaster recovery procedures

### 4. Performance Optimization
- Caching strategies at different levels
- Database optimization techniques
- Content delivery optimization

### 5. Technology Choices
- Appropriate database selections for use cases
- Message queue and caching solutions
- Monitoring and logging infrastructure

### 6. Security Considerations
- Authentication and authorization
- Data encryption
- API security and rate limiting

### 7. Cost Optimization
- Resource utilization efficiency
- Storage tiering strategies
- Compute optimization

---

## How to Use These Examples

1. **Study the Patterns**: Understand the common architectural patterns across different domains
2. **Practice Explaining**: Practice walking through each architecture and explaining design decisions
3. **Adapt to New Problems**: Learn to apply these patterns to new, unfamiliar problems
4. **Focus on Trade-offs**: Be prepared to discuss the trade-offs in your design choices

Each architecture demonstrates real-world scalability challenges and solutions that are commonly discussed in system design interviews at top tech companies.
