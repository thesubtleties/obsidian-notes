Query optimization handles how to efficiently fetch data from your database, especially when dealing with related models, complex filters, or large datasets. Think of it like creating a efficient shopping list instead of making multiple trips to the store.

### Why Use Query Optimization Patterns?
Instead of making multiple database queries or fetching more data than needed, these patterns help you get exactly what you need in the most efficient way.

### Example Without Query Optimization (Problematic):
```python
@app.route('/api/events/<int:event_id>/dashboard')
def event_dashboard(event_id):
    # ❌ BAD: Multiple separate queries
    event = Event.query.get(event_id)
    
    # ❌ BAD: N+1 query problem - one query per session
    sessions = Session.query.filter_by(event_id=event_id).all()
    for session in sessions:
        # ❌ BAD: Additional query for each session's speakers
        speakers = session.get_speakers()
        
    # ❌ BAD: Another separate query
    attendee_count = EventUser.query.filter_by(
        event_id=event_id, 
        role='attendee'
    ).count()
    
    # ❌ BAD: And another...
    recent_registrations = EventUser.query.filter_by(
        event_id=event_id
    ).order_by(EventUser.created_at.desc()).limit(5).all()
    
    # ❌ BAD: Each registration causes another query for user info
    for registration in recent_registrations:
        user = User.query.get(registration.user_id)
```

### Example With Query Optimization:
```python
class Event(db.Model):
    """Event model with optimized queries"""
    
    def get_dashboard_data(self):
        """
        Get all dashboard data in minimal queries.
        
        Returns:
            dict: Dashboard data including sessions, speakers, stats
        """
        # ✅ GOOD: Single query with joins and eager loading
        sessions_with_speakers = (
            Session.query
            .options(
                # Eager load relationships to prevent N+1
                db.joinedload(Session.speakers)
                .joinedload(User.profile),  # Nested relationship
                db.joinedload(Session.room)
            )
            .filter(Session.event_id == self.id)
            .all()
        )
        
        # ✅ GOOD: Single query for multiple stats using subqueries
        stats = db.session.query(
            db.func.count(distinct(EventUser.id)).label('total_attendees'),
            db.func.count(distinct(Session.id)).label('total_sessions'),
            db.func.count(distinct(
                case([
                    (EventUser.role == EventUserRole.SPEAKER, EventUser.user_id)
                ])
            )).label('total_speakers')
        ).select_from(Event).outerjoin(
            EventUser
        ).outerjoin(
            Session
        ).filter(
            Event.id == self.id
        ).first()
        
        # ✅ GOOD: Efficient recent registrations query
        recent_registrations = (
            EventUser.query
            .options(db.joinedload(EventUser.user))  # Eager load user data
            .filter_by(event_id=self.id)
            .order_by(EventUser.created_at.desc())
            .limit(5)
            .all()
        )
        
        return {
            'sessions': sessions_with_speakers,
            'stats': {
                'attendee_count': stats.total_attendees,
                'session_count': stats.total_sessions,
                'speaker_count': stats.total_speakers
            },
            'recent_registrations': recent_registrations
        }
```

### Example of Common Optimization Patterns:
```python
class Event(db.Model):
    """Examples of query optimization patterns"""
    
    # ✅ GOOD: Reusable query building
    @classmethod
    def base_query(cls):
        """Base query with commonly needed joins"""
        return cls.query.options(
            db.joinedload(cls.organization),
            db.joinedload(cls.sessions)
        )
    
    # ✅ GOOD: Efficient filtering
    @classmethod
    def search(cls, filters=None):
        """
        Efficient search with dynamic filters.
        
        Args:
            filters: Dict of filter parameters
        """
        query = cls.base_query()
        
        if filters:
            # ✅ GOOD: Build all filters before executing query
            if filters.get('status'):
                query = query.filter(cls.status == filters['status'])
                
            if filters.get('date_range'):
                start, end = filters['date_range']
                query = query.filter(
                    cls.start_date >= start,
                    cls.end_date <= end
                )
                
            if filters.get('organization_id'):
                query = query.filter(
                    cls.organization_id == filters['organization_id']
                )
        
        return query
    
    # ✅ GOOD: Efficient counting
    @classmethod
    def count_by_status(cls):
        """Get counts of events by status efficiently"""
        return db.session.query(
            cls.status,
            db.func.count(cls.id).label('count')
        ).group_by(
            cls.status
        ).all()
```

### Clean Usage in Routes:
```python
@app.route('/api/events/<int:event_id>/dashboard')
def event_dashboard(event_id):
    event = Event.query.get_or_404(event_id)
    
    # ✅ GOOD: Single method call gets all needed data
    dashboard_data = event.get_dashboard_data()
    
    return jsonify(dashboard_data)

@app.route('/api/events/search')
def search_events():
    filters = {
        'status': request.args.get('status'),
        'date_range': parse_date_range(request.args.get('dates')),
        'organization_id': request.args.get('org_id', type=int)
    }
    
    # ✅ GOOD: Efficient search with pagination
    page = request.args.get('page', 1, type=int)
    events = Event.search(filters).paginate(page=page)
    
    return jsonify({
        'items': events.items,
        'total': events.total,
        'pages': events.pages
    })
```

### When to Use Query Optimization:
1. **Complex Dashboards**
   - Multiple related models
   - Aggregate statistics
   - Recent activity lists

2. **Search Operations**
   - Multiple filter conditions
   - Sorting and pagination
   - Related data needed

3. **Bulk Operations**
   - Processing multiple records
   - Aggregate calculations
   - Report generation

### Benefits:
1. **Performance**
   - Fewer database queries
   - Less data transfer
   - Efficient memory usage

2. **Scalability**
   - Handles larger datasets
   - Better response times
   - Lower server load

3. **Maintainability**
   - Centralized query logic
   - Reusable patterns
   - Easier optimization

### Common Optimization Techniques:
```python
class QueryOptimizationExamples:
    """Examples of common optimization techniques"""
    
    @staticmethod
    def eager_loading():
        """Prevent N+1 queries with eager loading"""
        # Instead of:
        # ❌ BAD: Causes N+1 queries
        events = Event.query.all()
        for event in events:
            org = event.organization  # Extra query per event
            
        # Do:
        # ✅ GOOD: Single query with joined load
        events = Event.query.options(
            db.joinedload(Event.organization)
        ).all()
    
    @staticmethod
    def selective_loading():
        """Load only needed columns"""
        # Instead of:
        # ❌ BAD: Fetches all columns
        users = User.query.all()
        
        # Do:
        # ✅ GOOD: Fetch only needed columns
        users = db.session.query(
            User.id,
            User.email,
            User.name
        ).all()
    
    @staticmethod
    def efficient_counting():
        """Efficient counting techniques"""
        # Instead of:
        # ❌ BAD: Loads all records
        count = len(User.query.all())
        
        # Do:
        # ✅ GOOD: Counts at database level
        count = db.session.query(db.func.count(User.id)).scalar()
```

