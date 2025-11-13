# Python Testing Examples

## Unit Test Example
```python
import pytest
from calculator import Calculator

class TestCalculator:
    def test_addition(self):
        calc = Calculator()
        assert calc.add(2, 3) == 5
    
    def test_division_by_zero(self):
        calc = Calculator()
        with pytest.raises(ValueError):
            calc.divide(10, 0)
```

## Integration Test Example
```python
import pytest
from database import Database
from user_service import UserService

class TestUserServiceIntegration:
    @pytest.fixture
    def database(self):
        db = Database(testing=True)
        yield db
        db.cleanup()
    
    def test_user_creation(self, database):
        service = UserService(database)
        user = service.create_user('test@example.com')
        assert user.id is not None
        assert database.user_exists(user.id)
```

