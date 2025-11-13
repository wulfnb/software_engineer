# Test Automation

## Test Automation Pyramid
```mermaid
graph TD
    A[Unit Tests - Many] --> B[API Tests - Some]
    B --> C[UI Tests - Few]
    
    style A fill:#4CAF50,stroke:#388E3C,color:white
    style B fill:#FF9800,stroke:#F57C00,color:white
    style C fill:#F44336,stroke:#D32F2F,color:white
```

## Automation Strategy
### What to Automate
- Regression tests
- Smoke tests
- Data-driven tests
- Performance tests
- API tests

### What NOT to Automate
- UI tests that change frequently
- Tests requiring human judgment
- One-time tests
- Exploratory testing

