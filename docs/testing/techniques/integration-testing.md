# Integration Testing Comprehensive Guide

## Integration Testing Fundamentals

### What is Integration Testing?
```python
# Integration tests verify interactions between components
def test_user_registration_flow():
    # Tests multiple components working together
    user_service = UserService()
    email_service = EmailService()
    database = Database()
    
    # Register user (integrates multiple services)
    result = user_service.register(
        email="test@example.com",
        password="password123",
        email_service=email_service,
        database=database
    )
    
    # Verify integration points
    assert result.success == True
    assert database.user_exists(result.user_id)
    assert email_service.sent_welcome_email(result.user_id)
```

### Integration Testing in the Testing Pyramid
```mermaid
graph TD
    A[Unit Tests - 70%<br/>Single components] --> B[Integration Tests - 20%<br/>Component interactions]
    B --> C[E2E Tests - 10%<br/>Complete user flows]
    
    style A fill:#e8f5e8
    style B fill:#fff3e0
    style C fill:#ffebee
```

## Integration Testing Strategies

### Integration Approaches
```mermaid
graph TB
    A[Integration Testing] --> B[Big Bang]
    A --> C[Incremental]
    
    C --> D[Top-Down]
    C --> E[Bottom-Up]
    C --> F[Sandwich/Hybrid]
    
    B --> B1[Test all at once]
    B --> B2[Simple but hard to debug]
    
    D --> D1[Start with top modules]
    D --> D2[Use stubs for lower levels]
    
    E --> E1[Start with bottom modules]
    E --> E2[Use drivers for upper levels]
    
    F --> F1[Combine top-down & bottom-up]
    F --> F2[Most practical approach]
    
    style B fill:#FFEBEE
    style D fill:#E3F2FD
    style E fill:#E8F5E8
    style F fill:#FFF3E0
```

## Database Integration Testing

### Testing Database Operations
```python
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

class TestUserRepositoryIntegration:
    @pytest.fixture
    def database_session(self):
        """Real database connection for integration testing"""
        engine = create_engine('postgresql://test:test@localhost:5432/test_db')
        Session = sessionmaker(bind=engine)
        session = Session()
        
        # Clean slate for testing
        session.execute("DELETE FROM users")
        session.commit()
        
        yield session
        session.close()

    def test_create_and_retrieve_user(self, database_session):
        # Arrange
        user_repo = UserRepository(database_session)
        
        # Act - Create user
        user = user_repo.create_user(
            email="test@example.com",
            name="Test User"
        )
        
        # Assert - Verify database persistence
        retrieved_user = user_repo.get_user_by_id(user.id)
        assert retrieved_user is not None
        assert retrieved_user.email == "test@example.com"
        assert retrieved_user.name == "Test User"

    def test_user_email_uniqueness(self, database_session):
        # Arrange
        user_repo = UserRepository(database_session)
        user_repo.create_user(email="duplicate@example.com", name="User 1")
        
        # Act & Assert - Should fail on duplicate email
        with pytest.raises(IntegrityError):
            user_repo.create_user(email="duplicate@example.com", name="User 2")
```

### Test Containers for Database Testing
```java
// Java with TestContainers
@Testcontainers
class UserRepositoryIntegrationTest {
    @Container
    private static final PostgreSQLContainer<?> postgres = 
        new PostgreSQLContainer<>("postgres:13")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    @Test
    void shouldSaveAndRetrieveUser() {
        // Given
        UserRepository repository = new UserRepository(postgres.getJdbcUrl());
        User user = new User("test@example.com", "Test User");
        
        // When
        User savedUser = repository.save(user);
        User foundUser = repository.findById(savedUser.getId());
        
        // Then
        assertThat(foundUser).isEqualTo(savedUser);
    }
}
```

## API Integration Testing

### REST API Integration Tests
```python
import requests
import pytest

class TestUserAPI:
    BASE_URL = "http://localhost:8000/api"
    
    @pytest.fixture
    def auth_headers(self):
        # Get authentication token
        response = requests.post(
            f"{self.BASE_URL}/auth/login",
            json={"email": "admin@example.com", "password": "admin123"}
        )
        token = response.json()["token"]
        return {"Authorization": f"Bearer {token}"}
    
    def test_create_user_integration(self, auth_headers):
        # Act - Create user via API
        user_data = {
            "email": "integration@example.com",
            "name": "Integration Test User",
            "password": "secure123"
        }
        
        response = requests.post(
            f"{self.BASE_URL}/users",
            json=user_data,
            headers=auth_headers
        )
        
        # Assert - Verify API response and side effects
        assert response.status_code == 201
        user_response = response.json()
        assert user_response["email"] == user_data["email"]
        assert user_response["name"] == user_data["name"]
        assert "id" in user_response
        
        # Verify user was actually created in database
        get_response = requests.get(
            f"{self.BASE_URL}/users/{user_response['id']}",
            headers=auth_headers
        )
        assert get_response.status_code == 200

    def test_user_validation_integration(self, auth_headers):
        # Test validation across service boundaries
        invalid_user_data = {
            "email": "invalid-email",
            "name": "",
            "password": "123"
        }
        
        response = requests.post(
            f"{self.BASE_URL}/users",
            json=invalid_user_data,
            headers=auth_headers
        )
        
        assert response.status_code == 400
        errors = response.json()["errors"]
        assert "email" in errors
        assert "name" in errors
        assert "password" in errors
```

### Microservices Integration Testing
```python
import wiremock
import pytest

class TestOrderServiceIntegration:
    @pytest.fixture
    def mock_inventory_service(self):
        with wiremock.Client('http://localhost:8080') as wm:
            # Mock inventory service responses
            wm.register(
                wiremock.Get("/api/inventory/123")
                .will_return(wiremock.Response(
                    json={"productId": "123", "stock": 10},
                    status=200
                ))
            )
            wm.register(
                wiremock.Post("/api/inventory/123/reserve")
                .will_return(wiremock.Response(
                    json={"reservationId": "res-456", "success": True},
                    status=200
                ))
            )
            yield wm

    def test_create_order_integration(self, mock_inventory_service):
        # Arrange
        order_service = OrderService()
        
        # Act - Create order that integrates with inventory service
        order = order_service.create_order(
            product_id="123",
            quantity=2,
            customer_id="cust-789"
        )
        
        # Assert - Verify order creation and inventory reservation
        assert order.status == "CONFIRMED"
        assert order.reservation_id == "res-456"
        
        # Verify mock was called
        mock_inventory_service.verify(
            wiremock.Post("/api/inventory/123/reserve")
            .with_header("Content-Type", "application/json")
            .with_body({"quantity": 2})
        )
```

## Message Queue Integration Testing

### Testing with Message Brokers
```python
import pika
import pytest
import json

class TestMessageQueueIntegration:
    @pytest.fixture
    def rabbitmq_connection(self):
        connection = pika.BlockingConnection(
            pika.ConnectionParameters('localhost')
        )
        channel = connection.channel()
        channel.queue_declare(queue='test_orders')
        yield channel
        connection.close()

    def test_order_processing_flow(self, rabbitmq_connection):
        # Arrange
        order_processor = OrderProcessor()
        order_data = {
            "orderId": "order-123",
            "productId": "prod-456",
            "quantity": 2
        }
        
        # Act - Publish message to queue
        rabbitmq_connection.basic_publish(
            exchange='',
            routing_key='test_orders',
            body=json.dumps(order_data)
        )
        
        # Process message (this would typically be in a separate consumer)
        method_frame, header_frame, body = rabbitmq_connection.basic_get(
            queue='test_orders'
        )
        
        if method_frame:
            processed_order = order_processor.process_order(
                json.loads(body)
            )
            
            # Assert - Verify message processing
            assert processed_order.order_id == "order-123"
            assert processed_order.status == "PROCESSED"
            
            # Acknowledge message
            rabbitmq_connection.basic_ack(method_frame.delivery_tag)
```

## File System Integration Testing

### Testing File Operations
```python
import tempfile
import os
import pytest

class TestFileProcessorIntegration:
    @pytest.fixture
    def temp_directory(self):
        with tempfile.TemporaryDirectory() as temp_dir:
            yield temp_dir

    def test_file_processing_workflow(self, temp_directory):
        # Arrange
        file_processor = FileProcessor()
        input_file = os.path.join(temp_directory, "input.txt")
        output_file = os.path.join(temp_directory, "output.txt")
        
        # Create test input file
        with open(input_file, 'w') as f:
            f.write("line1\nline2\nline3\n")
        
        # Act - Process file
        file_processor.process_file(input_file, output_file)
        
        # Assert - Verify output file was created and processed correctly
        assert os.path.exists(output_file)
        
        with open(output_file, 'r') as f:
            content = f.read()
            assert "PROCESSED: line1" in content
            assert "PROCESSED: line2" in content
            assert "PROCESSED: line3" in content

    def test_file_cleanup_integration(self, temp_directory):
        # Test file cleanup workflow
        file_manager = FileManager()
        test_file = os.path.join(temp_directory, "temporary.txt")
        
        with open(test_file, 'w') as f:
            f.write("temporary content")
        
        # Verify file exists
        assert os.path.exists(test_file)
        
        # Cleanup
        file_manager.cleanup_old_files(temp_directory, days_old=0)
        
        # Verify file was deleted
        assert not os.path.exists(test_file)
```

## External Service Integration Testing

### Testing Third-Party APIs
```python
import pytest
import responses
from unittest.mock import patch

class TestPaymentGatewayIntegration:
    @responses.activate
    def test_payment_processing_integration(self):
        # Arrange - Mock external payment gateway
        responses.add(
            responses.POST,
            'https://api.payment-gateway.com/charges',
            json={
                'id': 'ch_123',
                'status': 'succeeded',
                'amount': 1000
            },
            status=200
        )
        
        payment_processor = PaymentProcessor()
        
        # Act - Process payment
        result = payment_processor.charge(
            amount=1000,
            token="tok_visa",
            currency="usd"
        )
        
        # Assert - Verify payment was processed
        assert result.success == True
        assert result.transaction_id == "ch_123"
        
        # Verify correct API call was made
        assert len(responses.calls) == 1
        assert responses.calls[0].request.url == 'https://api.payment-gateway.com/charges'

    @responses.activate
    def test_payment_failure_integration(self):
        # Test error handling with external service
        responses.add(
            responses.POST,
            'https://api.payment-gateway.com/charges',
            json={'error': 'card_declined'},
            status=402
        )
        
        payment_processor = PaymentProcessor()
        
        # Act & Assert
        with pytest.raises(PaymentError, match="card_declined"):
            payment_processor.charge(
                amount=1000,
                token="tok_visa",
                currency="usd"
            )
```

## Integration Testing Patterns

### Contract Testing
```mermaid
graph LR
    A[Consumer] --> B[Provider]
    A --> C[Pact Broker]
    B --> C
    
    subgraph "Contract Testing"
        D[Consumer Tests] --> E[Generate Pact]
        E --> C
        F[Provider Tests] --> G[Verify Pact]
        G --> C
    end
    
    style A fill:#E3F2FD
    style B fill:#E8F5E8
    style C fill:#FFF3E0
```

### Test Data Management
```python
import factory
from factory.alchemy import SQLAlchemyModelFactory

class UserFactory(SQLAlchemyModelFactory):
    class Meta:
        model = User
        sqlalchemy_session = test_session
    
    id = factory.Sequence(lambda n: n)
    email = factory.Sequence(lambda n: f"user{n}@example.com")
    name = factory.Faker('name')
    is_active = True

class TestDataManagement:
    def test_complex_business_workflow(self):
        # Arrange - Create complex test data
        admin_user = UserFactory(role='admin')
        customer_user = UserFactory(role='customer')
        
        product = ProductFactory(price=100, stock=10)
        order = OrderFactory(user=customer_user)
        OrderItemFactory(order=order, product=product, quantity=2)
        
        # Act - Execute complex workflow
        workflow_result = BusinessWorkflow().process_order(order.id)
        
        # Assert - Verify entire workflow
        assert workflow_result.success
        assert workflow_result.inventory_adjusted
        assert workflow_result.notifications_sent
```

## CI/CD Integration Testing

### Docker-Based Integration Tests
```yaml
# docker-compose.test.yml
version: '3.8'
services:
  test-database:
    image: postgres:13
    environment:
      POSTGRES_DB: test_db
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
    ports:
      - "5432:5432"
  
  test-rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
  
  test-app:
    build: .
    depends_on:
      - test-database
      - test-rabbitmq
    environment:
      DATABASE_URL: postgresql://test:test@test-database:5432/test_db
      RABBITMQ_URL: amqp://test-rabbitmq:5672
    command: ["./wait-for-services.sh", "test-database:5432", "test-rabbitmq:5672", "--", "pytest", "tests/integration/"]
```

### Integration Test Execution
```python
# conftest.py - Integration test configuration
import pytest
import docker
import time

@pytest.fixture(scope="session")
def docker_compose():
    """Start Docker containers for integration testing"""
    client = docker.from_env()
    
    # Start required services
    postgres_container = client.containers.run(
        "postgres:13",
        environment={
            "POSTGRES_DB": "test_db",
            "POSTGRES_USER": "test", 
            "POSTGRES_PASSWORD": "test"
        },
        ports={'5432/tcp': 5432},
        detach=True
    )
    
    # Wait for services to be ready
    time.sleep(10)
    
    yield
    
    # Cleanup
    postgres_container.stop()
    postgres_container.remove()
```

## Performance Integration Testing

### Database Performance Tests
```python
import time
import pytest

class TestDatabasePerformance:
    @pytest.mark.performance
    def test_bulk_insert_performance(self, database_session):
        # Arrange
        user_repo = UserRepository(database_session)
        start_time = time.time()
        
        # Act - Insert large dataset
        for i in range(1000):
            user_repo.create_user(
                email=f"user{i}@example.com",
                name=f"User {i}"
            )
        
        # Assert - Verify performance
        end_time = time.time()
        execution_time = end_time - start_time
        
        assert execution_time < 5.0  # Should complete within 5 seconds
        
        # Verify data integrity
        user_count = database_session.execute(
            "SELECT COUNT(*) FROM users"
        ).scalar()
        assert user_count == 1000
```

## Security Integration Testing

### Authentication Flow Testing
```python
class TestAuthenticationIntegration:
    def test_complete_auth_flow(self):
        # Test complete authentication workflow
        auth_service = AuthenticationService()
        user_service = UserService()
        session_service = SessionService()
        
        # Step 1: Register user
        registration_result = auth_service.register(
            email="auth_test@example.com",
            password="secure123"
        )
        assert registration_result.success
        
        # Step 2: Login
        login_result = auth_service.login(
            email="auth_test@example.com", 
            password="secure123"
        )
        assert login_result.success
        assert login_result.token is not None
        
        # Step 3: Verify session
        session = session_service.validate_token(login_result.token)
        assert session.is_valid
        assert session.user_email == "auth_test@example.com"
        
        # Step 4: Access protected resource
        user_profile = user_service.get_user_profile(
            token=login_result.token
        )
        assert user_profile.email == "auth_test@example.com"
```

## Best Practices & Patterns

### Integration Testing Principles
```mermaid
graph TD
    A[Test Real Integrations] --> B[Isolate External Dependencies]
    B --> C[Use Test Doubles Wisely]
    C --> D[Maintain Test Data]
    D --> E[Clean Up Resources]
    E --> F[Monitor Test Performance]
    
    style A fill:#E8F5E8
    style B fill:#E3F2FD
    style C fill:#FFF3E0
    style D fill:#F3E5F5
    style E fill:#FFEBEE
    style F fill:#E0F2F1
```

### Test Organization
```
tests/
├── unit/                    # Unit tests
├── integration/             # Integration tests
│   ├── database/           # Database integration tests
│   ├── api/                # API integration tests  
│   ├── messaging/          # Message queue tests
│   ├── external/           # External service tests
│   └── workflows/          # Business workflow tests
├── e2e/                    # End-to-end tests
└── fixtures/               # Test fixtures and data
```

### Common Integration Testing Commands
```bash
# Run integration tests
pytest tests/integration/ -v

# Run with specific markers
pytest -m "integration and database" --tb=short

# Run with coverage for integration tests
pytest tests/integration/ --cov=src --cov-report=html

# Run with specific environment
DATABASE_URL=postgresql://test:test@localhost:5432/test_db pytest

# Run using test containers
docker-compose -f docker-compose.test.yml up --abort-on-container-exit
```

This comprehensive integration testing guide covers testing strategies, patterns, and practical examples for testing interactions between components, databases, APIs, message queues, and external services.