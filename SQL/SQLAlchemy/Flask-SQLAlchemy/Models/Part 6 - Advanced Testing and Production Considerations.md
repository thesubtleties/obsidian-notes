## Load Testing and Performance
```python
class Project(db.Model):
    __tablename__ = 'projects'

    # Add indexes for frequently accessed columns
    __table_args__ = (
        db.Index('idx_project_owner', 'owner_id'),
        db.Index('idx_project_status_owner', 'status', 'owner_id'),
    )

    @classmethod
    def bulk_query_example(cls, user_ids):
        """Optimized bulk query"""
        return cls.query\
            .filter(cls.owner_id.in_(user_ids))\
            .options(
                db.joinedload(cls.features),
                db.joinedload(cls.sprints)
            )\
            .all()

    def to_dict(self):
        """Cached version of to_dict"""
        cache_key = f"project_{self.id}_dict"
        result = cache.get(cache_key)
        if result is None:
            result = {
                'id': self.id,
                'name': self.name,
                # ... other fields ...
            }
            cache.set(cache_key, result, timeout=300)  # 5 minute cache
        return result
```
Performance considerations:
- Add indexes strategically
- Use bulk operations
- Implement caching
- Optimize queries
- Monitor database load

## Race Condition Handling
```python
from sqlalchemy import and_, or_
from contextlib import contextmanager

class Feature(db.Model):
    __tablename__ = 'features'
    version = db.Column(db.Integer, default=1)

    @contextmanager
    def optimistic_lock(self):
        """Handle concurrent updates safely"""
        current_version = self.version
        try:
            yield self
            # Verify version hasn't changed
            success = db.session.query(Feature)\
                .filter(
                    and_(
                        Feature.id == self.id,
                        Feature.version == current_version
                    )
                )\
                .update(
                    {'version': Feature.version + 1},
                    synchronize_session=False
                )
            if not success:
                raise ValueError("Record was modified by another user")
            db.session.commit()
        except Exception:
            db.session.rollback()
            raise

    def safe_update(self, **kwargs):
        """Update with optimistic locking"""
        with self.optimistic_lock():
            for key, value in kwargs.items():
                setattr(self, key, value)
```
Race condition protection:
- Use optimistic locking
- Handle concurrent updates
- Prevent data corruption
- Maintain data integrity
- Provide clear error messages

## Production Monitoring
```python
from datetime import datetime
import logging

class AuditMixin:
    """Add monitoring capabilities to models"""
    
    def log_change(self, action, details=None):
        """Log model changes"""
        logging.info(
            f"Model {self.__class__.__name__} ID {self.id}: "
            f"{action} at {datetime.utcnow()} - {details or ''}"
        )

    def record_metric(self, metric_name, value):
        """Record metrics for monitoring"""
        from app.monitoring import statsd  # Example metrics client
        statsd.gauge(
            f"model.{self.__class__.__name__}.{metric_name}",
            value
        )

class Project(db.Model, AuditMixin):
    def update_status(self, new_status):
        old_status = self.status
        self.status = new_status
        db.session.commit()
        
        # Log change
        self.log_change(
            'status_update',
            f'Status changed from {old_status} to {new_status}'
        )
        
        # Record metrics
        self.record_metric('status_changes', 1)
```
Production monitoring helps:
- Track changes
- Debug issues
- Monitor performance
- Alert on problems
- Analyze usage patterns

## Performance Profiling
```python
import time
from functools import wraps

def profile_query(f):
    """Decorator to profile database queries"""
    @wraps(f)
    def wrapped(*args, **kwargs):
        from sqlalchemy import event
        queries = []
        
        def before_cursor_execute(conn, cursor, statement, *args):
            conn.info.setdefault('query_start_time', []).append(time.time())
            queries.append(statement)

        def after_cursor_execute(conn, cursor, statement, *args):
            total_time = time.time() - conn.info['query_start_time'].pop()
            logging.info(f"Query took {total_time:.2f}s: {statement}")

        # Add event listeners
        event.listen(db.engine, 'before_cursor_execute', before_cursor_execute)
        event.listen(db.engine, 'after_cursor_execute', after_cursor_execute)
        
        try:
            return f(*args, **kwargs)
        finally:
            # Remove event listeners
            event.remove(db.engine, 'before_cursor_execute', before_cursor_execute)
            event.remove(db.engine, 'after_cursor_execute', after_cursor_execute)
    
    return wrapped

class Project(db.Model):
    @classmethod
    @profile_query
    def get_complex_report(cls):
        """Complex query that might need optimization"""
        return cls.query\
            .join(Feature)\
            .join(Sprint)\
            .group_by(cls.id)\
            .all()
```
Profiling helps identify:
- Slow queries
- N+1 query problems
- Unnecessary database calls
- Performance bottlenecks
- Optimization opportunities

