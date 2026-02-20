# General IT Knowledge

## Application Maintenance
- **Logging**
  - Async vs Sync Logging
  - What information must be included into logging message
  - ELK
- **Application Monitoring**
  - Health check probes
- Crash dump analysis
- Tracing based on service-mesh
- **Application Design Patterns**
  - Monolithic architecture
    - Pros and cons of monolithic architecture vs microservices
  - Event-driven Architecture
  - Microservices Architecture
    - Service mesh
    - Service registry
    - Transactional outbox
    - Remote procedure invocation
    - Messaging pattern
    - Circuit Breaker
    - Saga pattern

## How OSs Work in General
- How random access memory works
- Understanding process concept
- Understanding thread concept
- Understand stack memory concept
- File System
- **Linux basic commands**
  - `grep`
  - `ls`
  - `tail`
  - `cat`
  - `chmod`

## Working with Data

### Data Formats
- Fixed-length format
- Protocol Buffers
- JSON
- YAML
- Binary Serialization
- XML
- CSV

### Web Services
- **Authentication**
  - OpenID
  - SAML
  - JWT
  - OAuth 2.0
- **Internet Protocols**
  - REST
  - SOAP
  - FTP

### Databases
- **Relational Databases**
  - PostgreSQL
  - SQL migrations
  - MySQL
  - Oracle
  - SQL commands: `INSERT`, `UPDATE`, `DELETE`, `SELECT`
  - Using Inner Joins
  - Using Left Joins
  - Sequence
  - Stored Procedure
  - Understanding SQL plan
  - **Indexes**
    - Single-Column Index
    - Multi-column Index
    - Understanding index direction
    - Primary Index
    - Cluster index
  - Database partitioning
  - Auto-increment values
  - **SQL Transactions**
    - Transaction Isolation levels
  - Aggregate Functions
- Database Normalization
- Database replication
- Database Sharding
- NoSQL Databases
- Column-oriented databases

## Math for Programmers
- Set Theory
- Category theory
- Algebra
  - Predicate
  - Commutativity
  - Distributivity
  - Associativity
- Mathematical Logic
- Geometry
  - Cartesian Coordinate System

---

# Application Lifecycle

## Version Control Systems
- Git
  - Git commit
    - Git tagging
    - Git stash
    - Git stage
  - Gitflow
  - Git Branching
    - Git merge
      - Git rebase
      - Git squash
  - Remote vs local repository
    - Git fetch
    - Git push
    - Git pull
  - `.gitignore`
  - Git limitations

## Java Build Systems
- Maven
- Gradle
- Package dependency
  - Dependency scope
- Package managers
  - Nexus
- Java application versioning

## Containers
- Docker
  - Docker Installation and Setup
  - Docker Images
  - Docker Containers
  - Container Networking
  - Docker volumes
  - Docker compose
  - Docker registry
  - Docker container orchestration
  - Docker Security
  - Docker Monitoring and Logging
- Kubernetes
  - Cluster Setup and Configuration
  - Containerization
  - Pod Management
  - Deployment
  - Scaling
  - Service and networking
  - Persistent Storage
  - Monitoring and Logging
  - Security and Access Control
  - Troubleshooting and Debugging
  - Cluster Maintenance and Upgrades
- OpenShift

---

# Code Quality
- SOLID
- Code testing
  - Unit Testing
  - Functional Testing
  - Integration Testing
  - Performance Testing
- DRY
- Application maintainability principles
- KISS

---

# Algorithms
- Binary Search
- Depth-first search
- Breadth-first search
- **Comparison metrics**
  - Time complexity notations
  - Worst time complexity
  - Average time complexity
  - Memory consumption of algorithm
  - Stable sorting
- Quicksort
- Heapsort
- Counting sort

---

# Core Java

## Java Syntax
- Keywords
- Control Flow
- Arrays
- Packages and Imports
- Java I/O
- Strings and Regular Expressions
- Object-oriented programming
  - Interfaces vs abstract classes
- Exceptions
  - Runtime handling of exceptions
  - Fatal errors
- Identifiers
- Literals
- Operators

## Types
- Generic types
  - Type erasure
  - Wildcards in generics
  - Heap pollution
- Primitive types
  - Boxing and unboxing
  - Number overflow
  - Number precision
  - Decimal vs double
  - Floating-point rounding
  - Primitive type memory size
- Reference types
  - Anonymous classes
- Raw types
- Casting conversion
  - Widening primitive conversion
  - Narrowing primitive conversion
  - Boxing conversion
  - Unboxing conversion
  - Widening reference conversion
  - Narrowing reference conversion

## Variables
- `final` variables
  - Restrictions in lambda expressions
  - Capturing final variables
  - Compiler optimizations
  - Thread safety
- Variable Naming Conventions
- Variable Memory (Stack vs Heap)

## Java Collections
- List
  - ArrayList
  - LinkedList
- Queue
  - BlockingQueue
  - PriorityQueue
  - ArrayDeque
- Set
  - EnumSet
  - HashSet
- Map
  - HashMap
  - WeakHashMap
  - ConcurrentHashMap
  - TreeMap
- Streams
  - Performance considerations
  - Lazy evaluation
- Common pitfalls
  - ConcurrentModificationException
  - Correct equals & hashCode
  - Mutable objects in hash collections

## Concurrency
- Process vs Thread
- Thread sleep & interrupt
- ThreadLocal
- Memory Model
  - Happens-before principle
  - Atomicity
  - Race condition
  - Visibility
  - Reordering
  - `volatile`
  - Immutable objects
- Synchronization Patterns
  - Monitor locking
  - `synchronized`
  - Lock
    - ReadWriteLock
    - ReentrantLock
  - Condition
  - Reentrancy
- Common problems
  - Deadlock
  - Livelock
  - Starvation
- Executors
  - ThreadPool
  - ForkJoinPool

---

# Java Frameworks and Libraries
- Spring Boot
  - Spring Data JPA
  - Application properties
  - Application profile
  - Bean lifecycle
  - Common annotations
- Apache Commons
- Log4j
- Mockito
- JUnit
- Apache Tomcat
- Java ORMs
  - Hibernate
  - MyBatis
  - Jooq

---

# How JVM Works

## Java Memory Organization
- Heap
  - Eden Space
  - Survivor Spaces
  - Tenured Generation
  - Memory optimization techniques
  - JVM flags
- Method Area
- Stack
  - Stack frame
  - Stack pointer
  - Recursive calls

## Garbage Collection
- GC roots
- Mark and sweep
- WeakReference
- Memory leak detection
- GC tuning

## Class Loaders
- Class Loader Hierarchy
- Custom Class Loading
- Isolation
- Dynamic Loading

## Execution Engine
- Interpreter
- JIT Compiler
  - Code optimization
  - Escape analysis
  - Method inlining

## Common JVM Errors
- ClassNotFoundException
- NoClassDefFoundError
- OutOfMemoryError
- StackOverflowError

---

# How the Internet Works

## Transport Protocols
- TCP/IP model
  - IP address
  - Network port
  - MAC address
  - Network route
- Proxy
  - Reverse proxy
  - Forward proxy
- DNS

## Browsers
- Browser caching
- URL
- Cookies
- Compression

## Application Protocols
- HTTP
  - GET, POST, PUT, DELETE, HEAD
  - Headers
  - Authentication
  - HTTPS
  - SSL/TLS
  - mTLS
  - SSE
- GraphQL
- WebSockets
- HTTP/2

## Internet Security
- Same-origin policy
- CORS
- CSRF
- XSS
- SQL Injection
- DoS attacks

---

# Teamwork
- Collaboration
- Conflict resolution
- Persuasion
- Coordination
- Creative Thinking
- Active listening
- Responsibility
- Communication skills
- Goal Setting
- Honesty
- Giving Feedback
- Empathy

---

# UI Design Patterns
- Model-View-View-Model (MVVM)
- Model-View-Presenter (MVP)
- Model-View-Controller (MVC)
