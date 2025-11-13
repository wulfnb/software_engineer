# Test Data Management

## Test Data Strategies
```python
# Factory pattern for test data
class UserFactory:
    @staticmethod
    def create_user(**kwargs):
        return {
            'email': kwargs.get('email', 'test@example.com'),
            'name': kwargs.get('name', 'Test User'),
            'role': kwargs.get('role', 'user'),
            'is_active': kwargs.get('is_active', True)
        }

# Usage in tests
def test_user_creation():
    user_data = UserFactory.create_user(role='admin')
    user = create_user(user_data)
    assert user.role == 'admin'
```

## Test Data Best Practices
- Use factory patterns
- Clean up test data
- Isolate test data
- Use realistic data
- Version control test data

