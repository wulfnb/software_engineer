# Acceptance Test-Driven Development (ATDD)

## ATDD Process
```mermaid
sequenceDiagram
    participant B as Business
    participant D as Development
    participant T as Testing
    
    B->>D: Define acceptance criteria
    D->>T: Write acceptance tests
    T->>D: Tests fail (RED)
    D->>D: Implement feature
    D->>T: Tests pass (GREEN)
    T->>B: Feature accepted
```

## ATDD vs TDD
| Aspect | TDD | ATDD |
|--------|-----|------|
| Focus | Technical implementation | Business requirements |
| Tests | Unit tests | Acceptance tests |
| Audience | Developers | Whole team |
| Scope | Code level | Feature level |

