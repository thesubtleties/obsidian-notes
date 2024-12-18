## Model Testing Setup
```python
import pytest
from datetime import datetime, timedelta
from app.models import db, User, Project, Task

class TestProject:
    """Test suite for Project model"""
    
    @pytest.fixture
    def sample_user(self):
        """Fixture to create test user"""
        user = User(
            username="testuser",
            email="test@test.com",
            password="password"
        )
        db.session.add(user)
        db.session.commit()
        yield user  # Use yield for cleanup
        db.session.delete(user)
        db.session.commit()

    @pytest.fixture
    def sample_project(self, sample_user):
        """Fixture with dependencies"""
        project = Project(
            name="Test Project",
            owner_id=sample_user.id,
            due_date=datetime.utcnow() + timedelta(days=7)
        )
        db.session.add(project)
        db.session.commit()
        yield project
        db.session.delete(project)
        db.session.commit()
```
Testing setup best practices:
- Use fixtures for test data
- Handle cleanup with yield
- Test dependencies properly
- Keep tests isolated
- Use meaningful test data

## Model Test Cases
```python
def test_project_creation(sample_user):
    """Test basic model creation"""
    project = Project(
        name="New Project",
        owner_id=sample_user.id,
        due_date=datetime.utcnow()
    )
    db.session.add(project)
    db.session.commit()
    
    assert project.id is not None
    assert project.name == "New Project"
    assert project.owner_id == sample_user.id

def test_project_validation():
    """Test model validation"""
    with pytest.raises(ValueError) as exc:
        Project.create_project(
            name="",  # Should fail validation
            owner_id=999  # Non-existent user
        )
    assert "Name is required" in str(exc.value)

def test_relationship_cascade(sample_project):
    """Test relationship behavior"""
    feature = Feature(
        name="Test Feature",
        project_id=sample_project.id
    )
    db.session.add(feature)
    db.session.commit()
    
    # Test cascade delete
    db.session.delete(sample_project)
    db.session.commit()
    
    assert Feature.query.filter_by(id=feature.id).first() is None
```
Different types of tests:
- Basic CRUD operations
- Validation logic
- Relationship behavior
- Error conditions
- Business logic

## Model Documentation
```python
class Project(db.Model):
    """
    Project Model
    
    Represents a project in the system. Projects can have multiple features
    and sprints, and belong to an owner while having multiple team members.
    
    Relationships:
        owner (User): The project owner (one-to-one)
        users (User): Team members (many-to-many)
        features (Feature): Project features (one-to-many)
        sprints (Sprint): Project sprints (one-to-many)
    
    Validations:
        - name: Required, max length 100
        - owner_id: Must reference valid user
        - due_date: Must be in future
    """
    
    def update_status(self, new_status):
        """
        Update project status with validation
        
        Args:
            new_status (str): New status value
            
        Raises:
            ValueError: If status is invalid
            
        Returns:
            Project: Updated project instance
        """
        if new_status not in self.VALID_STATUSES:
            raise ValueError(f"Invalid status. Must be one of: {self.VALID_STATUSES}")
        self.status = new_status
        db.session.commit()
        return self
```
Documentation best practices:
- Clear class docstrings
- Document relationships
- Document validations
- Method documentation
- Example usage where helpful

## Integration Testing
```python
class TestProjectIntegration:
    """Integration tests for Project model"""
    
    def test_project_lifecycle(self, sample_user):
        """Test complete project lifecycle"""
        # Create project
        project = Project.create_project(
            name="Integration Test",
            owner_id=sample_user.id,
            due_date=datetime.utcnow() + timedelta(days=7)
        )
        
        # Add team member
        new_user = User(username="team_member")
        db.session.add(new_user)
        db.session.commit()
        project.users.append(new_user)
        
        # Create feature
        feature = Feature(
            name="Test Feature",
            project_id=project.id
        )
        db.session.add(feature)
        
        # Create sprint
        sprint = Sprint(
            name="Sprint 1",
            project_id=project.id
        )
        db.session.add(sprint)
        
        # Test relationships
        assert new_user in project.users
        assert feature in project.features
        assert sprint in project.sprints
        
        # Test cascading delete
        db.session.delete(project)
        db.session.commit()
        
        assert Feature.query.filter_by(id=feature.id).first() is None
        assert Sprint.query.filter_by(id=sprint.id).first() is None
        assert new_user in db.session  # User should not be deleted
```
Integration tests verify:
- Complete workflows
- Relationship behaviors
- Cascade operations
- Business logic
- System integrity

