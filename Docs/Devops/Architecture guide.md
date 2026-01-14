# OpenBank Architecture Guide

**Project**: OpenBank Cloud Simulation - IBM DevOps Edition  
**Version**: 1.0.0  
**Last Updated**: January 13, 2025  
**Document Owner**: DevOps Team (Ntando, Kagiso, Florence, Tumelo)

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [System Overview](#system-overview)
3. [Architecture Patterns](#architecture-patterns)
4. [Component Architecture](#component-architecture)
5. [Network Architecture](#network-architecture)
6. [Data Architecture](#data-architecture)
7. [Monitoring Architecture](#monitoring-architecture)
8. [Security Architecture](#security-architecture)
9. [Deployment Architecture](#deployment-architecture)
10. [Scalability Considerations](#scalability-considerations)
11. [Technology Stack](#technology-stack)
12. [Design Decisions](#design-decisions)

---

## 1. Executive Summary

OpenBank is a containerized banking application built using modern microservices principles and DevOps practices. The architecture emphasizes modularity, observability, and operational excellence through comprehensive monitoring and Site Reliability Engineering (SRE) practices.

### Architecture Principles

- **Separation of Concerns**: Frontend, backend, and database are independently deployable
- **Container-First**: All components run in Docker containers for consistency
- **Observability**: Built-in monitoring with Prometheus and Grafana
- **Infrastructure as Code**: Declarative configuration using Docker Compose
- **Stateless Services**: Application tier designed for horizontal scalability
- **Data Persistence**: Stateful services use Docker volumes

---

## 2. System Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        User Layer                                │
│                   (Web Browsers / API Clients)                   │
└────────────────────────────┬────────────────────────────────────┘
                             │ HTTPS/HTTP
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Presentation Layer                            │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │         Frontend (React.js Application)                  │   │
│  │         - User Interface                                 │   │
│  │         - Client-Side Routing                           │   │
│  │         - State Management                              │   │
│  │         Port: 3000                                       │   │
│  └──────────────────────────────────────────────────────────┘   │
└────────────────────────────┬────────────────────────────────────┘
                             │ REST API
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Application Layer                            │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │         Backend (FastAPI Service)                        │   │
│  │         - Business Logic                                 │   │
│  │         - API Endpoints                                  │   │
│  │         - Authentication & Authorization                 │   │
│  │         - Transaction Processing                         │   │
│  │         Port: 8000                                       │   │
│  └──────────────────────────┬───────────────────────────────┘   │
└─────────────────────────────┼────────────────────────────────────┘
                              │ MongoDB Protocol
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       Data Layer                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │         MongoDB Database                                 │   │
│  │         - User Data                                      │   │
│  │         - Transaction Records                            │   │
│  │         - Account Information                            │   │
│  │         Port: 27017                                      │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   Observability Layer                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Prometheus  │  │   Grafana    │  │   Exporters  │          │
│  │   (Metrics)  │  │ (Dashboards) │  │  (Collectors)│          │
│  │  Port: 9090  │  │  Port: 3001  │  │  Multiple    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

### System Components

| Component | Type | Purpose | Technology |
|-----------|------|---------|------------|
| Frontend | Presentation | User interface and client logic | React.js |
| Backend | Application | Business logic and API | Python FastAPI |
| Database | Data | Persistent data storage | MongoDB |
| Prometheus | Monitoring | Metrics collection and storage | Prometheus |
| Grafana | Visualization | Dashboard and alerting | Grafana |
| Node Exporter | Monitoring | System metrics collection | Prometheus Exporter |
| cAdvisor | Monitoring | Container metrics collection | Google cAdvisor |

---

## 3. Architecture Patterns

### 3.1 Microservices Architecture

OpenBank follows a microservices architecture pattern with the following characteristics:

**Service Decomposition:**
```
OpenBank Application
├── Frontend Service (Presentation Microservice)
│   └── Responsibilities: UI rendering, user interactions, routing
│
├── Backend Service (Business Logic Microservice)
│   └── Responsibilities: API endpoints, authentication, transactions
│
└── Database Service (Data Microservice)
    └── Responsibilities: Data persistence, queries, indexing
```

**Benefits:**
- Independent deployment and scaling
- Technology flexibility per service
- Isolated failure domains
- Team autonomy

**Trade-offs:**
- Increased operational complexity
- Network latency between services
- Distributed transaction challenges

### 3.2 Three-Tier Architecture

```
┌─────────────────────────────────────┐
│        Tier 1: Presentation         │
│          (Frontend Layer)           │
│                                     │
│  - React Components                 │
│  - UI/UX Logic                      │
│  - Client State Management          │
└─────────────────┬───────────────────┘
                  │ HTTP/REST
                  ▼
┌─────────────────────────────────────┐
│        Tier 2: Application          │
│         (Backend Layer)             │
│                                     │
│  - FastAPI Routes                   │
│  - Business Logic                   │
│  - Authentication                   │
│  - Data Validation                  │
└─────────────────┬───────────────────┘
                  │ MongoDB Protocol
                  ▼
┌─────────────────────────────────────┐
│         Tier 3: Data                │
│        (Database Layer)             │
│                                     │
│  - MongoDB Collections              │
│  - Indexes                          │
│  - Data Persistence                 │
└─────────────────────────────────────┘
```

### 3.3 Container Orchestration Pattern

**Docker Compose Orchestration:**

```yaml
Services Definition (docker-compose.yml)
│
├── Service 1: frontend
│   ├── Build Context: ./frontend
│   ├── Dockerfile: ./frontend/Dockerfile
│   ├── Networks: [banking-network]
│   └── Depends On: [backend]
│
├── Service 2: backend
│   ├── Build Context: ./backend
│   ├── Dockerfile: ./backend/Dockerfile
│   ├── Networks: [banking-network]
│   └── Depends On: [mongodb]
│
└── Service 3: mongodb
    ├── Image: mongo:latest
    ├── Networks: [banking-network]
    └── Volumes: [mongodb-data]
```

---

## 4. Component Architecture

### 4.1 Frontend Architecture

```
Frontend Container (banking-app-dev-frontend)
│
├── React Application Structure
│   ├── /src
│   │   ├── /components
│   │   │   ├── Dashboard.jsx
│   │   │   ├── Login.jsx
│   │   │   ├── Transaction.jsx
│   │   │   └── Account.jsx
│   │   │
│   │   ├── /services
│   │   │   └── api.js (Backend API calls)
│   │   │
│   │   ├── /utils
│   │   │   ├── auth.js
│   │   │   └── validation.js
│   │   │
│   │   ├── App.js (Main component)
│   │   └── index.js (Entry point)
│   │
│   └── /public
│       ├── index.html
│       └── assets/
│
├── Build Process
│   ├── npm install (Dependencies)
│   ├── npm run build (Production build)
│   └── Serve via Node.js/nginx
│
└── Runtime Configuration 
    ├── Environment: Production
    ├── Port: 3000
    ├── API Endpoint: http://127.0.0.1:8000/                                      
    └── Network: banking-network
```

**Frontend Responsibilities:**
- Render user interface
- Handle user interactions
- Manage client-side state
- Make API calls to backend
- Display data from backend
- Client-side validation

**Technology Stack:**
- React.js (UI framework)
- HTML5/CSS3 (Markup and styling)
- JavaScript ES6+ (Programming language)
- Node.js (Build tooling)

### 4.2 Backend Architecture

```
Backend Container (banking-app-dev-backend)
│
├── FastAPI Application Structure
│   ├── /app
│   │   ├── main.py (Application entry point)
│   │   │
│   │   ├── /routes
│   │   │   ├── auth.py (Authentication endpoints)
│   │   │   ├── accounts.py (Account management)
│   │   │   ├── transactions.py (Transaction processing)
│   │   │   └── users.py (User management)
│   │   │
│   │   ├── /models
│   │   │   ├── user.py (User data models)
│   │   │   ├── account.py (Account models)
│   │   │   └── transaction.py (Transaction models)
│   │   │
│   │   ├── /services
│   │   │   ├── auth_service.py (Business logic)
│   │   │   ├── account_service.py
│   │   │   └── transaction_service.py
│   │   │
│   │   ├── /database
│   │   │   └── connection.py (MongoDB connection)
│   │   │
│   │   └── /utils
│   │       ├── security.py (JWT, hashing)
│   │       └── validators.py (Input validation)
│   │
│   └── requirements.txt (Python dependencies)
│
├── API Endpoints
│   ├── POST /api/auth/register
│   ├── POST /api/auth/login
│   ├── GET /api/accounts/{id}
│   ├── POST /api/transactions
│   └── GET /api/transactions/{id}
│
└── Runtime Configuration
    ├── Environment: Production
    ├── Port: 8000
    ├── Database: mongodb://banking-mongodb:27017
    └── Network: banking-network
```

**Backend Responsibilities:**
- Process API requests
- Implement business logic
- Authenticate and authorize users
- Validate input data
- Interact with database
- Generate responses

**Technology Stack:**
- Python 3.9+ (Programming language)
- FastAPI (Web framework)
- Pydantic (Data validation)
- PyMongo (MongoDB driver)
- JWT (Authentication tokens)

### 4.3 Database Architecture

```
MongoDB Container (banking-mongodb)
│
├── Database Structure
│   ├── Database: banking_db
│   │   │
│   │   ├── Collection: users
│   │   │   ├── Document Schema:
│   │   │   │   {
│   │   │   │     _id: ObjectId,
│   │   │   │     username: String,
│   │   │   │     email: String,
│   │   │   │     password_hash: String,
│   │   │   │     created_at: DateTime
│   │   │   │   }
│   │   │   └── Indexes: [username, email]
│   │   │
│   │   ├── Collection: accounts
│   │   │   ├── Document Schema:
│   │   │   │   {
│   │   │   │     _id: ObjectId,
│   │   │   │     user_id: ObjectId,
│   │   │   │     account_number: String,
│   │   │   │     balance: Decimal,
│   │   │   │     currency: String,
│   │   │   │     created_at: DateTime
│   │   │   │   }
│   │   │   └── Indexes: [user_id, account_number]
│   │   │
│   │   └── Collection: transactions
│   │       ├── Document Schema:
│   │       │   {
│   │       │     _id: ObjectId,
│   │       │     account_id: ObjectId,
│   │       │     type: String,
│   │       │     amount: Decimal,
│   │       │     description: String,
│   │       │     timestamp: DateTime,
│   │       │     status: String
│   │       │   }
│   │       └── Indexes: [account_id, timestamp]
│   │
│   └── Persistent Storage
│       └── Volume: mongodb-data (Docker volume)
│
└── Runtime Configuration
    ├── Port: 27017
    ├── Storage Engine: WiredTiger
    ├── Network: banking-network
    └── Persistence: Enabled via volumes
```

**Database Responsibilities:**
- Store user credentials
- Maintain account balances
- Record transaction history
- Ensure data integrity
- Provide query performance

---

## 5. Network Architecture

### 5.1 Docker Network Topology

```
┌─────────────────────────────────────────────────────────────┐
│              Host Machine (Development)                      │
│                                                              │
│  ┌────────────────────────────────────────────────────┐     │
│  │         Docker Bridge Network: banking-network     │     │
│  │         Subnet: 172.18.0.0/16                      │     │
│  │                                                     │     │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────┐ │     │
│  │  │  Frontend    │  │   Backend    │  │ MongoDB  │ │     │
│  │  │ 172.18.0.2   │  │  172.18.0.3  │  │172.18.0.4│ │     │
│  │  │ Port: 3000   │  │  Port: 8000  │  │Port:27017│ │     │
│  │  └──────┬───────┘  └──────┬───────┘  └────┬─────┘ │     │
│  │         │                 │                │       │     │
│  │         └─────────────────┴────────────────┘       │     │
│  │              Internal Network Communication        │     │
│  └────────────────────────────────────────────────────┘     │
│                              │                               │
│  ┌───────────────────────────┴──────────────────────┐       │
│  │    Docker Bridge Network: monitoring-network     │       │
│  │    Subnet: 172.19.0.0/16                         │       │
│  │                                                   │       │
│  │  ┌──────────┐ ┌─────────┐ ┌──────────┐ ┌──────┐ │       │
│  │  │Prometheus│ │ Grafana │ │  Node    │ │cAdvis│ │       │
│  │  │172.19.0.2│ │172.19.0.3│ │ Exporter │ │172.19│ │       │
│  │  │Port: 9090│ │Port: 3001│ │Port: 9100│ │Port  │ │       │
│  │  └──────────┘ └─────────┘ └──────────┘ └──────┘ │       │
│  └───────────────────────────────────────────────────┘       │
│                                                              │
│         Port Mappings (Host → Container)                    │
│         localhost:3000  → frontend:3000                     │
│         localhost:8000  → backend:8000                      │
│         localhost:27017 → mongodb:27017                     │
│         localhost:9090  → prometheus:9090                   │
│         localhost:3001  → grafana:3000                      │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 Network Communication Flow

**User Request Flow:**

```
1. User Browser
   │
   │ HTTP GET http://127.0.0.1:5500/
   ▼
2. Docker Host Port Mapping
   │
   │ Forward to 172.18.0.2:3000
   ▼
3. Frontend Container
   │
   │ Serves React Application
   ▼
4. User Browser (React App Loaded)
   │
   │ HTTP POST http://127.0.0.1:5500/                        
5. Docker Host Port Mapping
   │
   │ Forward to 172.18.0.3:8000
   ▼
6. Backend Container
   │
   │ Process API Request
   ▼
7. Backend → MongoDB Communication
   │
   │ mongodb://banking-mongodb:27017
   │ (Internal network, no port mapping needed)
   ▼
8. MongoDB Container
   │
   │ Query Database
   ▼
9. Response Flow (Reverse)
   │
   └─→ MongoDB → Backend → Frontend → User
```

**Monitoring Data Flow:**

```
1. Prometheus Container
   │
   │ Scrape Interval: 15 seconds
   ▼
2. Target Discovery
   ├─→ node-exporter:9100 (System metrics)
   ├─→ cadvisor:8080 (Container metrics)
   ├─→ prometheus:9090 (Self-monitoring)
   └─→ Application endpoints (planned)
   │
   ▼
3. Prometheus Storage
   │
   │ Store Time-Series Data
   ▼
4. Grafana Queries
   │
   │ HTTP GET prometheus:9090/api/v1/query
   ▼
5. Dashboard Visualization
   │
   └─→ Display to User via localhost:3001
```

### 5.3 Network Security

**Container Isolation:**
- Each container runs in isolated namespace
- Only exposed ports are accessible from host
- Internal communication uses Docker DNS

**Network Policies:**
```
Frontend Container:
  - Inbound: Port 3000 (from host)
  - Outbound: Port 8000 (to backend)

Backend Container:
  - Inbound: Port 8000 (from frontend, host)
  - Outbound: Port 27017 (to MongoDB)

MongoDB Container:
  - Inbound: Port 27017 (from backend only)
  - Outbound: None (no external connections)

Monitoring Containers:
  - Inbound: Specific ports (9090, 3001, etc.)
  - Outbound: Scrape targets
```

---

## 6. Data Architecture

### 6.1 Data Model

**Entity Relationship Diagram:**

```
┌─────────────────────┐
│       Users         │
│─────────────────────│
│ _id: ObjectId (PK)  │
│ username: String    │
│ email: String       │
│ password_hash: String│
│ created_at: DateTime│
└──────────┬──────────┘
           │
           │ 1:N
           │
           ▼
┌─────────────────────┐
│      Accounts       │
│─────────────────────│
│ _id: ObjectId (PK)  │
│ user_id: ObjectId(FK)│
│ account_number: Str │
│ balance: Decimal    │
│ currency: String    │
│ created_at: DateTime│
└──────────┬──────────┘
           │
           │ 1:N
           │
           ▼
┌─────────────────────┐
│    Transactions     │
│─────────────────────│
│ _id: ObjectId (PK)  │
│ account_id: ObjectId│
│ type: String        │
│ amount: Decimal     │
│ description: String │
│ timestamp: DateTime │
│ status: String      │
└─────────────────────┘
```

### 6.2 Data Flow

**Transaction Processing Flow:**

```
1. User Initiates Transaction
   │ (Frontend)
   │
   ▼
2. API Request Validation
   │ (Backend - Input validation)
   │
   ▼
3. Business Logic Processing
   │ (Backend - Check balance, rules)
   │
   ▼
4. Database Transaction
   │ (MongoDB - Atomic operation)
   │
   ├─→ Update Account Balance
   │   (accounts collection)
   │
   └─→ Create Transaction Record
       (transactions collection)
   │
   ▼
5. Response Generation
   │ (Backend - Format response)
   │
   ▼
6. UI Update
   └ (Frontend - Display confirmation)
```

### 6.3 Data Persistence

**Volume Architecture:**

```
Docker Host Filesystem
│
├── /var/lib/docker/volumes/
│   │
│   ├── mongodb-data/
│   │   └── _data/
│   │       ├── WiredTiger files
│   │       ├── Collection data
│   │       ├── Indexes
│   │       └── Journal logs
│   │
│   ├── prometheus-data/
│   │   └── _data/
│   │       ├── Time-series database
│   │       ├── WAL (Write-Ahead Log)
│   │       └── Chunks
│   │
│   └── grafana-data/
│       └── _data/
│           ├── Dashboard definitions
│           ├── Data sources config
│           └── User preferences
```

**Backup Strategy (Planned):**
- Scheduled MongoDB backups
- Prometheus snapshot exports
- Grafana dashboard exports
- Configuration file versioning

---

## 7. Monitoring Architecture

### 7.1 Observability Stack

```
┌──────────────────────────────────────────────────────────┐
│                    Visualization Layer                    │
│                                                           │
│  ┌─────────────────────────────────────────────────┐     │
│  │              Grafana Dashboards                 │     │
│  │  ┌──────────────────────────────────────┐      │     │
│  │  │  Banking App - System Overview       │      │     │
│  │  │  - HTTP Requests per Second          │      │     │
│  │  │  - Service Status                   │      │     │
│  │  │  - Memory Usage                      │      │     │
│  │  │  - CPU Usage                         │      │     │
│  │  │  - Container Health                  │      │     │
│  │  └──────────────────────────────────────┘      │     │
│  └─────────────────────┬───────────────────────────┘     │
└────────────────────────┼─────────────────────────────────┘
                         │ PromQL Queries
                         ▼
┌──────────────────────────────────────────────────────────┐
│                    Metrics Storage Layer                  │
│                                                           │
│  ┌─────────────────────────────────────────────────┐     │
│  │              Prometheus TSDB                    │     │
│  │  - Time-series data storage                     │     │
│  │  - Data retention: 15 days                      │     │
│  │  - Scrape interval: 15 seconds                  │     │
│  │  - Storage: ~245MB (current)                    │     │
│  └─────────────────────┬───────────────────────────┘     │
└────────────────────────┼─────────────────────────────────┘
                         │ Scrape Metrics
                         │
        ┌────────────────┼────────────────┬────────────────┐
        │                │                │                │
        ▼                ▼                ▼                ▼
┌──────────────────────────────────────────────────────────┐
│                  Metrics Collection Layer                 │
│                                                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │    Node     │  │  cAdvisor   │  │   Application   │  │
│  │  Exporter   │  │  (Container │  │     Metrics     │  │
│  │             │  │   Metrics)  │  │    (Planned)    │  │
│  │  - CPU      │  │             │  │                 │  │
│  │  - Memory   │  │  - Per-     │  │  - Request      │  │
│  │  - Disk I/O │  │    container│  │    rates        │  │
│  │  - Network  │  │    CPU      │  │  - Latency      │  │
│  │             │  │  - Memory   │  │  - Errors       │  │
│  └─────────────┘  └─────────────┘  └─────────────────┘  │
└──────────────────────────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────┐
│                   Target Systems Layer                    │
│                                                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐               │
│  │ Frontend │  │ Backend  │  │ MongoDB  │               │
│  │Container │  │Container │  │Container │               │
│  └──────────┘  └──────────┘  └──────────┘               │
└──────────────────────────────────────────────────────────┘
```

### 7.2 Metrics Collection Strategy

**Metrics Hierarchy:**

```
Level 1: Infrastructure Metrics (Node Exporter)
├── Hardware Metrics
│   ├── CPU utilization
│   ├── Memory usage
│   ├── Disk I/O
│   └── Network throughput
│
└── Operating System Metrics
    ├── Process count
    ├── File descriptors
    ├── System load
    └── Uptime

Level 2: Container Metrics (cAdvisor)
├── Resource Usage
│   ├── CPU per container
│   ├── Memory per container
│   ├── Network I/O per container
│   └── Filesystem per container
│
└── Container State
    ├── Running/stopped status
    ├── Restart count
    ├── Health check results
    └── Resource limits

Level 3: Application Metrics (Planned)
├── Request Metrics
│   ├── Request rate
│   ├── Request duration
│   ├── Status codes (2xx, 4xx, 5xx)
│   └── Endpoint performance
│
└── Business Metrics
    ├── Active users
    ├── Transaction count
    ├── Account operations
    └── Error rates
```

### 7.3 Dashboard Design

**Dashboard Hierarchy:**

```
Main Dashboard: Banking App - System Overview
├── Row 1: Service Health
│   └── Panel: Service Status (up metric)
│
├── Row 2: Application Performance
│   ├── Panel: HTTP Requests per Second
│   └── Panel: Response Time Distribution (planned)
│
├── Row 3: Resource Utilization
│   ├── Panel: CPU Usage
│   └── Panel: Memory Usage
│
└── Row 4: Container Health
    └── Panel: Container Resource Usage
```

**Future Dashboards (Planned):**
- Infrastructure Deep Dive
- Application Performance Monitoring (APM)
- Business Metrics Dashboard
- SLO/SLI Dashboard

---

## 8. Security Architecture

### 8.1 Security Layers

```
┌──────────────────────────────────────────────────────┐
│              Layer 5: Application Security           │
│  - Input validation                                  │
│  - SQL injection prevention                          │
│  - XSS protection                                    │
│  - CSRF tokens                                       │
└────────────────────┬─────────────────────────────────┘
                     │
┌────────────────────┴─────────────────────────────────┐
│        Layer 4: Authentication & Authorization       │
│  - JWT token-based authentication                    │
│  - Password hashing (bcrypt)                         │
│  - Role-based access control                         │
│  - Session management                                │
└────────────────────┬─────────────────────────────────┘
                     │
┌────────────────────┴─────────────────────────────────┐
│            Layer 3: Network Security                 │
│  - Container network isolation                       │
│  - Internal DNS resolution                           │
│  - Port exposure minimization                        │
│  - TLS/SSL (planned for production)                  │
└────────────────────┬─────────────────────────────────┘
                     │
┌────────────────────┴─────────────────────────────────┐
│          Layer 2: Container Security                 │
│  - Non-root user execution                           │
│  - Read-only filesystems (where applicable)          │
│  - Resource limits (CPU, memory)                     │
│  - Security scanning (planned)                       │
└────────────────────┬─────────────────────────────────┘
                     │
┌────────────────────┴─────────────────────────────────┐
│           Layer 1: Infrastructure Security           │
│  - Host firewall rules                               │
│  - Volume encryption (planned)                       │
│  - Backup encryption (planned)                       │
│  - Secrets management (planned)                      │
└──────────────────────────────────────────────────────┘
```

### 8.2 Authentication Flow

```
1. User Login Request
   │ POST /api/auth/login
   │ { username, password }
   ▼
2. Backend Receives Request
   │
   ├─→ Validate Input Format
   ▼
3. Database Query
   │ Query users collection
   │ WHERE username = provided_username
   ▼
4. Password Verification
   │ Compare: bcrypt.verify(provided_password, stored_hash)
   ▼
5. JWT Token Generation (if valid)
   │ Create token with:
   │ - user_id
   │ - username
   │ - expiration time (24 hours)
   │ - secret key signature
   ▼
6. Return Response
   │ { token: "jwt_token_here", user: {...} }
   │
   └─→ Frontend stores token
       └─→ Include in subsequent requests
           └─→ Header: Authorization: Bearer <token>
```

### 8.3 Data Security

**Sensitive Data Handling:**

```
User Password Flow:
User Input (plaintext)
  → Frontend (plaintext, HTTPS only in production)
    → Backend receives
      → bcrypt.hash(password, salt_rounds=12)
        → Store hash in MongoDB
          → Never store plaintext password

Transaction Data:
Sensitive Fields:
  - Account numbers (consider encryption)
  - Transaction amounts (encrypted in transit)
  - User PII (Personal Identifiable Information)

Protection Methods:
  - TLS/SSL for data in transit (production)
  - Volume encryption for data at rest (planned)
  - Access control policies
  - Audit logging (planned)
```

---

## 9. Deployment Architecture

### 9.1 Development Deployment

**Current Setup:**

```
Development Environment (Docker Desktop)
│
├── Application Stack
│   └── docker-compose.yml
│       ├── frontend service
│       ├── backend service
│       └── mongodb service
│
├── Monitoring Stack
│   └── docker-compose.monitoring.yml
│       ├── prometheus service
│       ├── grafana service
│       ├── node-exporter service
│       └── cadvisor service
│
└── Deployment Commands
    ├── docker-compose up -d (Application)
    └── docker-compose -f docker-compose.monitoring.yml up -d
```

**Container Lifecycle:**

```
Docker Compose Up
  │
  ├─→ Pull/Build Images
  │   ├── Check local image cache
  │   ├── Pull from registry if needed
  │   └── Build custom images (frontend, backend)
  │
  ├─→ Create Networks
  │   ├── banking-network
  │   └── monitoring-network
  │
  ├─→ Create Volumes
  │   ├── mongodb-data
  │   ├── prometheus-data
  │   └── grafana-data
  │
  ├─→ Start Containers (dependency order)
  │   ├── 1. MongoDB (no dependencies)
  │   ├── 2. Backend (depends on MongoDB)
  │   ├── 3. Frontend (depends on Backend)
  │   └── 4-7. Monitoring stack (parallel)
  │
  └─→ Health Checks
      ├── Wait for container ready
      └── Verify port bindings
```

### 9.2 Production Deployment (Planned)

**Kubernetes Architecture (Future):**

```
Kubernetes Cluster
│
├── Namespace: openbank-prod
│   │
│   ├── Deployments
│   │   ├── frontend-deployment (3 replicas)
│   │   ├── backend-deployment (3 replicas)
│   │   └── mongodb-statefulset (1 replica)
│   │
│   ├── Services
│   │   ├── frontend-service (LoadBalancer)
│   │   ├── backend-service (ClusterIP)
│   │   └── mongodb-service (ClusterIP)
│   │
│   ├── ConfigMaps
│   │   ├── app-config
│   │   └── monitoring-config
│   │
│   ├── Secrets
│   │   ├── mongodb-credentials
│   │   ├── jwt-secret
│   │   └── api-keys
│   │
│   └── PersistentVolumeClaims
│       ├── mongodb-pvc (100Gi)
│       ├── prometheus-pvc (50Gi)
│       └── grafana-pvc (10Gi)
│
└── Namespace: openbank-monitoring
    ├── Prometheus
    ├── Grafana
    ├── Alertmanager
    └── Exporters
```

### 9.3 CI/CD Pipeline Architecture

```
Developer Workflow
│
├─→ Code Commit (Git push)
│   └─→ GitHub Repository
│       └─→ Triggers GitHub Actions
│
└─→ CI Pipeline
    │
    ├─→ Stage 1: Build
    │   ├── Checkout code
    │   ├── Install dependencies
    │   ├── Run linters
    │   └── Build Docker images
    │
    ├─→ Stage 2: Test
    │   ├── Unit tests
    │   ├── Integration tests
    │   ├── Security scans
    │   └── Code coverage
    │
    ├─→ Stage 3: Push
    │   ├── Tag images
    │   ├── Push to registry
    │   └── Update manifests
    │
    └─→ Stage 4: Deploy
        ├── Deploy to staging
        ├── Run smoke tests
        ├── Manual approval gate
        └── Deploy to production
```

---

## 10. Scalability Considerations

### 10.1 Horizontal Scaling Strategy

**Current State (Single Instance):**

```
Load: 0.03 req/sec
│
└─→ [Frontend] ──→ [Backend] ──→ [MongoDB]
     1 instance    1 instance    1 instance
```

**Scaled Architecture (Future):**

```
Load: 1000 req/sec
│
├─→ Load Balancer
│   │
│   ├─→ [Frontend-1]
│   ├─→ [Frontend-2]     ┌─→ [Backend-1]
│   └─→ [Frontend-3] ────┤   [Backend-2] ──┬─→ [MongoDB Primary]
│                         │   [Backend-3]   │      │
│                         └─→ [Backend-4]   │   ┌──┴─────┐
│                                           │   │        │
└─→ Session Store (Redis)                   │   ▼        ▼
                                            └─[Replica] [Replica]
```

### 10.2 Scaling Triggers

**Auto-scaling Metrics:**

| Metric | Scale Up Threshold | Scale Down Threshold |
|--------|-------------------|---------------------|
| CPU Usage | > 70% for 5 min | < 30% for 10 min |
| Memory Usage | > 80% for 5 min | < 40% for 10 min |
| Request Rate | > 100 req/sec | < 20 req/sec |
| Response Time | P95 > 1000ms | P95 < 200ms |

### 10.3 Database Scaling

**MongoDB Scaling Options:**

```
Vertical Scaling (Current Approach)
├── Increase container resources
├── Add more CPU/memory
└── Limited by host machine

Horizontal Scaling (Future)
├── Replica Set
│   ├── Primary (writes)
│   ├── Secondary-1 (reads)
│   └── Secondary-2 (reads)
│
└── Sharding (High Scale)
    ├── Config servers
    ├── Query routers (mongos)
    └── Shard clusters
        ├── Shard-1 (accounts A-M)
        └── Shard-2 (accounts N-Z)
```

---

## 11. Technology Stack

### 11.1 Technology Matrix

| Layer | Technology | Version | Purpose | Status |
|-------|-----------|---------|---------|---------|
| Frontend | React.js | 18.x | UI framework | Production |
| Frontend | HTML5/CSS3 | - | Markup/styling | Production |
| Frontend | JavaScript | ES6+ | Programming | Production |
| Backend | Python | 3.9+ | Runtime | Production |
| Backend | FastAPI | 0.104+ | Web framework | Production |
| Backend | Pydantic | 2.x | Validation | Production |
| Backend | PyMongo | 4.x | MongoDB driver | Production |
| Database | MongoDB | 6.0+ | NoSQL database | Production |
| Container | Docker | 24.x | Containerization | Production |
| Orchestration | Docker Compose | 2.x | Multi-container | Production |
| Monitoring | Prometheus | Latest | Metrics collection | Production |
| Monitoring | Grafana | Latest | Visualization | Production |
| Monitoring | Node Exporter | Latest | System metrics | Production |
| Monitoring | cAdvisor | Latest | Container metrics | Production |
| CI/CD | GitHub Actions | - | Automation | Production |
| Orchestration | Kubernetes | 1.28+ | Production deploy | Production |

### 11.2 Technology Dependencies

```
Frontend Dependencies
├── react (^18.2.0)
├── react-dom (^18.2.0)
├── react-router-dom (^6.x)
├── axios (HTTP client)
└── Development Tools
    ├── webpack
    ├── babel
    └── eslint

Backend Dependencies
├── fastapi (^0.104.0)
├── uvicorn (ASGI server)
├── pymongo (^4.5.0)
├── pydantic (^2.4.0)
├── python-jose[cryptography] (JWT)
├── passlib[bcrypt] (Password hashing)
└── python-multipart (Form data)

Infrastructure Dependencies
├── docker (^24.0.0)
├── docker-compose (^2.20.0)
└── Git (version control)
```

---

## 12. Design Decisions

### 12.1 Architectural Decisions Record (ADR)

**ADR-001: Microservices vs Monolith**

**Status**: Accepted  
**Context**: Need to decide application architecture pattern  
**Decision**: Implement microservices architecture  
**Rationale**:
- Independent scaling of components
- Technology flexibility per service
- Easier to maintain and update
- Better alignment with DevOps practices
**Consequences**:
- Increased operational complexity
- Need for service discovery
- Distributed transaction challenges

---

**ADR-002: Docker Compose vs Kubernetes**

**Status**: Accepted (Development), Future (Production)  
**Context**: Need container orchestration solution  
**Decision**: Use Docker Compose for development, plan Kubernetes for production  
**Rationale**:
- Docker Compose simpler for local development
- Faster setup and iteration
- Kubernetes overkill for current scale
- Can migrate later when needed
**Consequences**:
- Need to rewrite configs for Kubernetes
- Different networking model to learn
- Migration effort in future

---

**ADR-003: MongoDB vs PostgreSQL**

**Status**: Accepted  
**Context**: Need to choose database system  
**Decision**: Use MongoDB (NoSQL)  
**Rationale**:
- Flexible schema for rapid development
- JSON-like documents match API structure
- Horizontal scaling capabilities
- Strong community and documentation
**Consequences**:
- No ACID transactions across documents (initially)
- Need to manage data consistency
- Learning curve for team

---

**ADR-004: Prometheus + Grafana vs ELK Stack**

**Status**: Accepted  
**Context**: Need monitoring and observability solution  
**Decision**: Implement Prometheus + Grafana  
**Rationale**:
- Time-series data perfect for metrics
- Grafana provides excellent visualization
- Lightweight and fast
- Strong Kubernetes integration for future
- Pull-based model preferred
**Consequences**:
- Need separate solution for logs
- Learning PromQL query language
- Storage considerations for long retention

---

**ADR-005: JWT vs Session-based Authentication**

**Status**: Accepted  
**Context**: Need to implement user authentication  
**Decision**: Use JWT token-based authentication  
**Rationale**:
- Stateless authentication
- Scales better (no session storage)
- Works well with microservices
- Mobile-friendly
**Consequences**:
- Token revocation challenges
- Need to manage token expiration
- Larger payload than session IDs

---

### 12.2 Trade-offs Analysis

**Performance vs Simplicity:**
```
Decision: Simple architecture first
Trade-off: May not handle extreme scale
Justification: Current scale doesn't require complex solutions
Mitigation: Monitor and scale when needed
```

**Security vs Development Speed:**
```
Decision: Implement core security, plan advanced features
Trade-off: Some security features deferred
Justification: Time constraints of 4-week project
Mitigation: Document security roadmap for production
```

**Observability vs Resource Usage:**
```
Decision: Comprehensive monitoring stack
Trade-off: Additional resource consumption (~5%)
Justification: Observability critical for DevOps
Mitigation: Efficient tools (Prometheus) minimize overhead
```

---

## 13. Future Architecture Enhancements

### 13.1 Short-term (Next 3 Months)

1. **Application Metrics Instrumentation**
   - Add Prometheus client to backend
   - Instrument key endpoints
   - Custom business metrics

2. **Alerting Configuration**
   - Prometheus alert rules
   - Alertmanager setup
   - Notification channels (email, Slack)

3. **MongoDB Exporter**
   - Deploy mongodb_exporter
   - Database performance metrics
   - Query performance tracking

### 13.2 Medium-term (3-6 Months)

1. **Log Aggregation**
   - Deploy ELK stack or Loki
   - Centralized log collection
   - Log-based alerting

2. **Distributed Tracing**
   - Implement Jaeger
   - Request flow visualization
   - Performance bottleneck identification

3. **High Availability**
   - MongoDB replica set
   - Multi-instance deployments
   - Load balancing

### 13.3 Long-term (6-12 Months)

1. **Cloud Migration**
   - Deploy to IBM Cloud/AWS
   - Managed services integration
   - Multi-region deployment

2. **Advanced Monitoring**
   - APM (Application Performance Monitoring)
   - User experience monitoring
   - Business metrics dashboard

3. **Chaos Engineering**
   - Failure injection testing
   - Resilience validation
   - Disaster recovery drills

---

## 14. Architecture Validation

### 14.1 Non-Functional Requirements

| Requirement | Target | Current Status | Architecture Support |
|-------------|--------|----------------|---------------------|
| Availability | 99.9% | 100% (week 4) | Container restart policies |
| Scalability | 1000 req/sec | 0.03 req/sec | Stateless services ready |
| Performance | P95 < 500ms | P95 = 455ms | Efficient tech stack |
| Security | Industry standard | Basic implemented | Layered security model |
| Observability | Full stack | 85% coverage | Comprehensive monitoring |
| Maintainability | Easy updates | Good | Modular architecture |

### 14.2 Architecture Quality Attributes

**Modularity**:
- Clear separation of concerns
- Independent deployability
- Loose coupling between services

**Scalability**:
- Horizontal scaling supported
- Stateless application tier
- Database scaling planned

**Reliability**:
- Container health checks
- Restart policies configured
- Monitoring in place

**Security**:
- Basic security implemented
- Authentication working
- Production hardening needed

**Observability**:
- Comprehensive metrics
- Real-time dashboards
- Application metrics planned

---

## 15. Conclusion

The OpenBank architecture demonstrates a modern, cloud-native approach to building scalable banking applications. The microservices architecture, combined with comprehensive monitoring and DevOps practices, provides a solid foundation for both educational purposes and potential production deployment.

### Key Architectural Strengths

1. **Modularity**: Clear separation between frontend, backend, and data layers
2. **Observability**: Built-in monitoring from day one
3. **Scalability**: Architecture supports horizontal scaling
4. **Maintainability**: Well-documented, Infrastructure as Code approach
5. **DevOps-Ready**: Designed for automated deployment and monitoring

### Architecture Maturity

Current State: **Level 3 - Defined**
- Documented processes and architecture
- Standardized tooling and practices
- Comprehensive monitoring

Target State: **Level 4 - Managed**
- Automated scaling
- Advanced observability
- Chaos engineering

---

**Document Version**: 1.0  
**Status**: Complete  
**Next Review**: Post-deployment feedback  
**Related Documents**:
- DevOps Metrics Report
- SLO Document
- Week 4 Summary
- Deployment Guide

---

*This architecture guide is a living document and should be updated as the system evolves.
