# Testing Without Mocks - Visual Guide

Diagrams summarizing James Shore's "Testing Without Mocks" patterns and techniques.

## 1. Pattern Categories Overview

```mermaid
mindmap
  root((Testing Without Mocks))
    Foundational Patterns
      Narrow Tests
      State-Based Tests
      Overlapping Sociable Tests
      Smoke Tests
      Zero-Impact Instantiation
      Parameterless Instantiation
      Signature Shielding
    Architectural Patterns
      A-Frame Architecture
      Logic Sandwich
      Traffic Cop
      Grow Evolutionary Seeds
    Logic Patterns
      Easily-Visible Behavior
      Testable Libraries
      Collaborator-Based Isolation
    Infrastructure Patterns
      Infrastructure Wrappers
      Narrow Integration Tests
      Paranoic Telemetry
    Nullability Patterns
      Nullables
      Embedded Stub
      Thin Wrapper
      Configurable Responses
      Output Tracking
      Behavior Simulation
    Legacy Code Patterns
      Descend the Ladder
      Climb the Ladder
      Replace Mocks with Nullables
      Throwaway Stub
```

## 2. A-Frame Architecture

The three-layer architecture that separates concerns for testability:

```mermaid
graph TB
    subgraph "A-Frame Architecture"
        direction TB
        
        subgraph APP["Application Layer"]
            A1[Orchestrates workflows]
            A2[Coordinates infrastructure + logic]
            A3[Uses Logic Sandwich pattern]
        end
        
        subgraph LOGIC["Logic Layer"]
            L1[Pure business logic]
            L2[Value objects]
            L3[No external dependencies]
            L4[Deterministic & fast]
        end
        
        subgraph INFRA["Infrastructure Layer"]
            I1[External system interfaces]
            I2[Network / Filesystem / DB]
            I3[Infrastructure Wrappers]
            I4[Implements Nullables]
        end
    end
    
    APP --> LOGIC
    APP --> INFRA
    
    style APP fill:#e1f5fe
    style LOGIC fill:#e8f5e9
    style INFRA fill:#fff3e0
```

## 3. Logic Sandwich Pattern

Application layer pattern: Read → Process → Write

```mermaid
flowchart LR
    subgraph "Logic Sandwich"
        direction LR
        R["📖 READ<br/>(Infrastructure)"]
        P["⚙️ PROCESS<br/>(Logic)"]
        W["📝 WRITE<br/>(Infrastructure)"]
        
        R --> P --> W
    end
    
    DB1[(Database)] -.-> R
    API1[External API] -.-> R
    W -.-> DB2[(Database)]
    W -.-> API2[External API]
    
    style R fill:#fff3e0
    style P fill:#e8f5e9
    style W fill:#fff3e0
```

## 4. Nullables Pattern

Production code with an "off switch" for testing:

```mermaid
flowchart TB
    subgraph "Nullables Pattern"
        direction TB
        
        subgraph Factory["Static Factory Methods"]
            CREATE[".create()"]
            CREATENULL[".createNull(config)"]
        end
        
        subgraph Production["Production Path"]
            REAL_DEP[Real Dependencies]
            REAL_SYS[Real External Systems]
        end
        
        subgraph Test["Test Path"]
            NULL_DEP[Nulled Dependencies]
            STUB[Embedded Stubs]
            CONFIG[Configurable Responses]
            TRACK[Output Tracking]
        end
        
        CREATE --> REAL_DEP --> REAL_SYS
        CREATENULL --> NULL_DEP --> STUB
        NULL_DEP --> CONFIG
        NULL_DEP --> TRACK
    end
    
    style CREATE fill:#e8f5e9
    style CREATENULL fill:#e1f5fe
    style STUB fill:#fff3e0
```

## 5. Infrastructure Wrapper with Embedded Stub

```mermaid
flowchart TB
    subgraph "Infrastructure Wrapper Pattern"
        direction TB
        
        subgraph Wrapper["HttpClient (Infrastructure Wrapper)"]
            C1[".create()"]
            C2[".createNull()"]
            CONS["constructor(http)"]
            METHODS["request(), get(), post()..."]
        end
        
        subgraph ThirdParty["Third-Party Code"]
            REAL["Real HTTP Library<br/>(Node.js http module)"]
            STUBBED["StubbedHttp<br/>(Embedded Stub)"]
        end
        
        C1 --> |"passes"| REAL
        C2 --> |"passes"| STUBBED
        REAL --> CONS
        STUBBED --> CONS
        CONS --> METHODS
    end
    
    EXT[External Network] <-.-> REAL
    STUBBED -.-> |"returns canned responses"| METHODS
    
    style REAL fill:#e8f5e9
    style STUBBED fill:#e1f5fe
```

## 6. Configurable Responses

Define responses from behavior perspective, not implementation:

```mermaid
flowchart LR
    subgraph "❌ Wrong: Implementation-Focused"
        W1["HTTP status: 200"]
        W2["JSON body: '{...}'"]
        W3["Headers: {...}"]
    end
    
    subgraph "✅ Right: Behavior-Focused"
        R1["loginSucceeds: true"]
        R2["emailVerified: false"]
        R3["userId: 'abc123'"]
    end
    
    style W1 fill:#ffcdd2
    style W2 fill:#ffcdd2
    style W3 fill:#ffcdd2
    style R1 fill:#c8e6c9
    style R2 fill:#c8e6c9
    style R3 fill:#c8e6c9
```

```mermaid
flowchart TB
    subgraph "createNull with Configurable Responses"
        CALL["LoginClient.createNull({<br/>  email: 'test@example.com',<br/>  emailVerified: false<br/>})"]
        
        DEFAULTS["Sensible Defaults:<br/>• Named parameters<br/>• Optional with defaults<br/>• Tests configure only what matters"]
        
        CALL --> DEFAULTS
    end
```

## 7. Output Tracking

Track behavior (what was written), not function calls:

```mermaid
sequenceDiagram
    participant Test
    participant Service
    participant Tracker as OutputTracker
    participant Emitter as EventEmitter

    Test->>Service: createNull()
    Test->>Service: trackSentEmails()
    Service->>Tracker: OutputTracker.create(emitter, event)
    Service-->>Test: tracker
    
    Test->>Service: sendEmail(to, subject, body)
    Service->>Emitter: emit("email-sent", { to, subject, body })
    Emitter->>Tracker: records behavior
    
    Test->>Tracker: tracker.data
    Tracker-->>Test: [{ to, subject, body }]
```

## 8. Testing Strategy

```mermaid
flowchart TB
    subgraph "Test Types"
        direction TB
        
        subgraph UNIT["Unit Tests (Many)"]
            U1["Use .createNull()"]
            U2["Narrow & focused"]
            U3["State-based assertions"]
            U4["Sociable - real dependencies"]
            U5["Fast execution"]
        end
        
        subgraph INT["Integration Tests (Few)"]
            I1["Use .create()"]
            I2["Narrow scope"]
            I3["Verify infrastructure wrappers"]
            I4["Real external systems"]
            I5["Smoke tests"]
        end
    end
    
    UNIT --> |"verify"| BEHAVIOR[Application Behavior]
    INT --> |"verify"| WRAPPERS[Infrastructure Works]
    
    style UNIT fill:#e8f5e9
    style INT fill:#fff3e0
```

## 9. Test Characteristics Comparison

```mermaid
flowchart LR
    subgraph "Testing Without Mocks"
        TWM1["Narrow"]
        TWM2["Sociable"]
        TWM3["State-Based"]
    end
    
    subgraph "Traditional Mocking"
        TM1["Often Broad"]
        TM2["Solitary/Isolated"]
        TM3["Interaction-Based"]
    end
    
    TWM1 ---|vs| TM1
    TWM2 ---|vs| TM2
    TWM3 ---|vs| TM3
    
    style TWM1 fill:#c8e6c9
    style TWM2 fill:#c8e6c9
    style TWM3 fill:#c8e6c9
    style TM1 fill:#ffcdd2
    style TM2 fill:#ffcdd2
    style TM3 fill:#ffcdd2
```

## 10. Naming Conventions

```mermaid
flowchart TB
    subgraph "Method Naming Patterns"
        direction LR
        
        subgraph Factory["Factory Methods"]
            F1[".create()"]
            F2[".createNull()"]
        end
        
        subgraph Tracking["Output Tracking"]
            T1["track[OutputType]()"]
            T2["e.g., trackSentEmails()"]
            T3["e.g., trackOutput()"]
        end
        
        subgraph Simulation["Behavior Simulation"]
            S1["simulate[Event]()"]
            S2["e.g., simulateMessage()"]
            S3["e.g., simulateConnection()"]
        end
        
        subgraph Stubs["Embedded Stubs"]
            ST1["Stubbed[ThirdParty]"]
            ST2["e.g., StubbedHttp"]
            ST3["e.g., StubbedProcess"]
        end
    end
```

## 11. Legacy Code Migration Path

```mermaid
flowchart LR
    subgraph "Descend the Ladder"
        D1["Start with broad tests"]
        D2["Introduce infrastructure wrappers"]
        D3["Add Nullables"]
        D4["Write narrow tests"]
        D5["Remove broad tests"]
        
        D1 --> D2 --> D3 --> D4 --> D5
    end
    
    subgraph "Climb the Ladder"
        C1["Start with narrow tests"]
        C2["Build up coverage"]
        C3["Refactor safely"]
        
        C1 --> C2 --> C3
    end
```

## Quick Reference

| Pattern | Purpose |
|---------|---------|
| **A-Frame** | Separate Application, Logic, Infrastructure layers |
| **Nullables** | Production code with test "off switch" |
| **Embedded Stub** | Stub third-party code, not your code |
| **Configurable Responses** | Test setup via `.createNull({ behavior })` |
| **Output Tracking** | Track what would be written externally |
| **Logic Sandwich** | Read → Process → Write |
| **Narrow Tests** | Focus on specific behavior |
| **Sociable Tests** | Use real dependencies (nulled) |
| **State-Based Tests** | Verify outputs, not interactions |
