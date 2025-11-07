# System Design Concepts

## Scalability

Definition: The ability of a system to handle increased load by adding resources.
Types of Scalability:

```mermaid
graph TB
    A[Scalability] --> B[Vertical Scaling]
    A --> C[Horizontal Scaling]
    
    B --> D[Scale Up]
    B --> E[Scale Down]
    
    C --> F[Scale Out]
    C --> G[Scale In]
    
    D --> H[Add more CPU/RAM<br/>to existing machine]
    F --> I[Add more machines<br/>to the system]
```

Vertical Scaling (Scale Up/Down):

    Adding more resources (CPU, RAM) to existing machines

    Example: Upgrading from 8GB to 32GB RAM on a database server

    Limitation: Single point of failure, hardware limits

Horizontal Scaling (Scale Out/In):

    Adding more machines to the system

    Example: Adding more web servers behind a load balancer

    Advantage: Better fault tolerance, virtually unlimited scaling

## Throughput & Bandwidth

Throughput: Number of units of work processed per time unit

    Example: 1000 requests/second, 50 transactions/minute

Bandwidth: Maximum data transfer rate of a network

    Example: 1 Gbps network connection

```mermaid
graph LR
    A[Client] --> B[Load Balancer]
    B --> C[Server 1<br/>250 req/s]
    B --> D[Server 2<br/>250 req/s]
    B --> E[Server 3<br/>250 req/s]
    B --> F[Server 4<br/>250 req/s]
    
    G[Total Throughput<br/>1000 requests/second]
    H[Network Bandwidth<br/>1 Gbps]
```

## Concurrency vs Parallelism

```mermaid
graph TD
    A[Concurrency vs Parallelism] --> B[Concurrency]
    A --> C[Parallelism]
    
    B --> D[Single Core<br/>Task Switching]
    B --> E[Illusion of simultaneity]
    B --> F[Example: Web server handling<br/>multiple requests on one CPU]
    
    C --> G[Multiple Cores<br/>Simultaneous Execution]
    C --> H[True parallel processing]
    C --> I[Example: Video rendering<br/>using multiple CPU cores]
```

### Concurrency:

    Multiple tasks making progress without necessarily running simultaneously

    Example: A single-threaded web server handling multiple requests using async I/O

### Parallelism:

    Multiple tasks executing simultaneously

    Example: A multi-threaded application processing images using all CPU cores

### Consistency, Availability & Partition Tolerance

#### Core Concepts in Distributed Systems
```mermaid
graph TD
    A[Distributed System Properties] --> B[Consistency]
    A --> C[Availability]
    A --> D[Partition Tolerance]
    
    B --> E[All nodes see<br/>the same data<br/>at the same time]
    C --> F[Every request receives<br/>a response]
    D --> G[System continues operating<br/>despite network failures]
    
    H[Real-world Examples] --> I[Consistency: Banking systems]
    H --> J[Availability: Social media feeds]
    H --> K[Partition Tolerance: Global CDNs]
```

Consistency:

    Strong Consistency: All reads see the most recent write

    Eventual Consistency: All nodes will eventually have the same data

    Example: Banking systems require strong consistency for balance checks

Availability:

    System remains operational and responsive

    Example: Social media feeds prioritize availability over perfect consistency

Partition Tolerance:

    System continues functioning despite network partitions

    Example: Global CDNs that can serve content even if some regions are disconnected

### CAP Theorem

```mermaid
graph TD
    A[CAP Theorem] --> B[Pick 2 of 3]
    A --> C[Consistency]
    A --> D[Availability]
    A --> E[Partition Tolerance]
    
    F[System Types] --> G[CP System]
    F --> H[AP System]
    F --> I[CA System<br/>Theoretical Only]
    
    G --> J[Consistent & Partition Tolerant<br/>e.g., MongoDB, HBase]
    H --> K[Available & Partition Tolerant<br/>e.g., Cassandra, DynamoDB]
    I --> L[Not practical in distributed systems]
```

#### CAP Theorem: In distributed systems, you can only guarantee two of:

    Consistency: All nodes see the same data at the same time

    Availability: Every request receives a response

    Partition Tolerance: System continues operating despite network partitions

### PACELC Theorem

Extension of CAP: If Partition occurs, choose between Availability and Consistency; Else, choose between Latency and Consistency.

```mermaid
graph TD
    A[Network State] --> B{Partition?}
    B -->|Yes| C[PAC: Choose A or C]
    B -->|No| D[ELC: Choose L or C]
    
    C --> E[PA/EC: Choose Availability<br/>e.g., Cassandra]
    C --> F[PC/EC: Choose Consistency<br/>e.g., MongoDB]
    
    D --> G[Choose Low Latency<br/>e.g., Cache reads]
    D --> H[Choose Consistency<br/>e.g., Synchronous replication]
```

### Latency

Definition: Time delay between request and response
Latency Components:

```mermaid
graph TD
    A[Total Latency] --> B[Network Latency]
    A --> C[Processing Latency]
    A --> D[Database Latency]
    A --> E[Queueing Latency]
    
    B --> F[Propagation Delay]
    B --> G[Transmission Delay]
    B --> H[Serialization Delay]
    
    C --> I[CPU Processing]
    C --> J[I/O Operations]
    C --> K[Garbage Collection]
```

### Techniques That Reduce Latency

```mermaid
graph TB
    A[Latency Reduction Techniques] --> B[Caching]
    A --> C[CDN]
    A --> D[Load Balancing]
    A --> E[Database Optimization]
    A --> F[Asynchronous Processing]
    A --> G[Connection Pooling]
    
    B --> H[Redis, Memcached]
    C --> I[CloudFront, Akamai]
    D --> J[Round Robin, Least Connections]
    E --> K[Indexing, Query Optimization]
    F --> L[Message Queues]
    G --> M[Database Connection Pools]
```

Examples:

    Caching: Redis for session storage

    CDN: CloudFront for static assets

    Database Optimization: Proper indexing, query optimization

    Async Processing: RabbitMQ for background jobs

### Relational vs Non-Relational Databases

```mermaid
graph LR
    A[Database Types] --> B[SQL Databases]
    A --> C[NoSQL Databases]
    
    B --> D[Relational]
    B --> E[ACID Properties]
    B --> F[Structured Schema]
    B --> G[Examples: PostgreSQL, MySQL]
    
    C --> H[Document - MongoDB]
    C --> I[Key-Value - Redis]
    C --> J[Column-Family - Cassandra]
    C --> K[Graph - Neo4j]
```

Comparison Table:
Aspect	SQL	NoSQL
Schema	Fixed, predefined	Dynamic, flexible
ACID	Strong consistency	Eventual consistency
Scaling	Vertical	Horizontal
Use Case	Complex queries, transactions	High throughput, flexible data

### Transactions & Their Types

```mermaid
graph TB
    A[Transactions] --> B[ACID Properties]
    A --> C[Transaction Types]
    
    B --> D[Atomicity]
    B --> E[Consistency]
    B --> F[Isolation]
    B --> G[Durability]
    
    C --> H[Flat Transactions]
    C --> I[Nested Transactions]
    C --> J[Distributed Transactions]
    C --> K[Compensating Transactions]
    
    D --> L[All or nothing execution]
    E --> M[Valid state transition]
    F --> N[Concurrent execution control]
    G --> O[Commited changes persist]
```

Isolation Levels:

    Read Uncommitted - Lowest isolation, dirty reads possible

    Read Committed - Prevents dirty reads

    Repeatable Read - Prevents non-repeatable reads

    Serializable - Highest isolation, complete isolation

### Additional Concepts
#### Load Balancing

```mermaid
graph TB
    A[Client] --> B[Load Balancer]
    B --> C[Algorithm Selection]
    
    C --> D[Round Robin]
    C --> E[Least Connections]
    C --> F[IP Hash]
    C --> G[Weighted Round Robin]
    
    B --> H[Server Pool]
    H --> I[Web Server 1]
    H --> J[Web Server 2]
    H --> K[Web Server N]
```

#### Microservices Architecture

```mermaid
graph TB
    A[API Gateway] --> B[User Service]
    A --> C[Order Service]
    A --> D[Payment Service]
    A --> E[Inventory Service]
    
    B --> F[User Database]
    C --> G[Order Database]
    D --> H[Payment Database]
    E --> I[Inventory Database]
    
    C --> D
    C --> E
    D --> J[External Payment Gateway]
```

#### Message Queues

```mermaid
graph LR
    A[Producer] --> B[Message Queue]
    B --> C[Consumer 1]
    B --> D[Consumer 2]
    B --> E[Consumer 3]
    
    F[Examples] --> G[RabbitMQ]
    F --> H[Kafka]
    F --> I[SQS]
```

#### Caching Strategies

```mermaid
graph TB
    A[Caching Strategies] --> B[Cache-Aside]
    A --> C[Read-Through]
    A --> D[Write-Through]
    A --> E[Write-Behind]
    A --> F[Write-Around]
    
    B --> G[App checks cache first,<br/>then database]
    C --> H[Cache provider reads<br/>from DB on miss]
    D --> I[Write to cache and DB<br/>simultaneously]
```

#### Database Replication

```mermaid
graph TB
    A[Primary Database] --> B[Replication]
    
    B --> C[Synchronous Replication]
    B --> D[Asynchronous Replication]
    
    C --> E[Strong consistency<br/>Higher latency]
    D --> F[Eventual consistency<br/>Lower latency]
    
    A --> G[Read Replica 1]
    A --> H[Read Replica 2]
    A --> I[Read Replica 3]
```

Key Performance Metrics

    RPS: Requests per second

    P99 Latency: 99th percentile latency

    Error Rate: Percentage of failed requests

    Uptime: System availability percentage

    MTTR: Mean Time To Recovery

    MTBF: Mean Time Between Failures

This README provides a comprehensive overview of essential system design concepts with visual diagrams for better understanding. Keep it as a reference for your system design preparations!