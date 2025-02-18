# Study Sessions Implementation Plans

## POST /study_sessions
This plan outlines the steps to implement the POST /study_sessions endpoint that creates a new study session.

### Requirements
The endpoint should:
- Accept a POST request with group_id and study_activity_id
- Create a new study session in the database
- Return the created session details

### Implementation Steps

#### Setup and Research
- [ ] Review the study_sessions table schema in sql/setup/create_table_study_sessions.sql
- [ ] Review existing study session routes in routes/study_sessions.py
- [ ] Ensure test database is working (run `invoke init-db` with test config)

#### Implementation
- [ ] Add the new route handler in routes/study_sessions.py:
```python
@app.route('/api/study-sessions', methods=['POST'])
@cross_origin()
def create_study_session():
    try:
        data = request.get_json()
        
        # Validate required fields
        if not data or 'group_id' not in data or 'study_activity_id' not in data:
            return jsonify({"error": "Missing required fields"}), 400
            
        cursor = app.db.cursor()
        
        # Verify group exists
        cursor.execute('SELECT id FROM groups WHERE id = ?', (data['group_id'],))
        if not cursor.fetchone():
            return jsonify({"error": "Group not found"}), 404
            
        # Verify study activity exists
        cursor.execute('SELECT id FROM study_activities WHERE id = ?', 
                      (data['study_activity_id'],))
        if not cursor.fetchone():
            return jsonify({"error": "Study activity not found"}), 404
            
        # Insert new session
        cursor.execute('''
            INSERT INTO study_sessions (group_id, study_activity_id)
            VALUES (?, ?)
        ''', (data['group_id'], data['study_activity_id']))
        
        app.db.commit()
        session_id = cursor.lastrowid
        
        # Get created session
        cursor.execute('''
            SELECT 
                ss.id,
                ss.group_id,
                g.name as group_name,
                sa.id as activity_id,
                sa.name as activity_name,
                ss.created_at
            FROM study_sessions ss
            JOIN groups g ON g.id = ss.group_id
            JOIN study_activities sa ON sa.id = ss.study_activity_id
            WHERE ss.id = ?
        ''', (session_id,))
        
        session = cursor.fetchone()
        
        return jsonify({
            'id': session['id'],
            'group_id': session['group_id'],
            'group_name': session['group_name'],
            'activity_id': session['activity_id'],
            'activity_name': session['activity_name'],
            'start_time': session['created_at'],
            'end_time': session['created_at']
        }), 201
        
    except Exception as e:
        return jsonify({"error": str(e)}), 500
```

#### Testing
- [ ] Create test file test_study_sessions.py:
```python
def test_create_study_session(client):
    # Test missing fields
    response = client.post('/api/study-sessions', json={})
    assert response.status_code == 400
    
    # Test invalid group
    response = client.post('/api/study-sessions', json={
        'group_id': 999,
        'study_activity_id': 1
    })
    assert response.status_code == 404
    
    # Test invalid activity
    response = client.post('/api/study-sessions', json={
        'group_id': 1,
        'study_activity_id': 999
    })
    assert response.status_code == 404
    
    # Test successful creation
    response = client.post('/api/study-sessions', json={
        'group_id': 1,
        'study_activity_id': 1
    })
    assert response.status_code == 201
    data = response.get_json()
    assert 'id' in data
    assert data['group_id'] == 1
    assert data['activity_id'] == 1
    assert 'start_time' in data
```

#### Manual Testing
- [ ] Test the endpoint using curl:
```bash
# Test missing fields
curl -X POST http://localhost:5000/api/study-sessions \
  -H "Content-Type: application/json" \
  -d '{}'

# Test successful creation  
curl -X POST http://localhost:5000/api/study-sessions \
  -H "Content-Type: application/json" \
  -d '{"group_id": 1, "study_activity_id": 1}'
```

#### API Documentation
```markdown
## Create Study Session

POST /api/study-sessions

Creates a new study session.

### Request Body
{
    "group_id": 1,
    "study_activity_id": 1
}

### Response
{
    "id": 1,
    "group_id": 1,
    "group_name": "Core Verbs",
    "activity_id": 1,
    "activity_name": "Typing Tutor",
    "start_time": "2024-03-20T10:30:00",
    "end_time": "2024-03-20T10:30:00"
}

### Status Codes
- 201: Created successfully
- 400: Missing required fields
- 404: Group or activity not found
- 500: Server error
```

#### Final Steps
- [ ] Run all tests to ensure nothing was broken
- [ ] Test the endpoint with the frontend if available
- [ ] Commit changes with a descriptive message

### Notes
- The endpoint uses SQLite3 for the database
- All dates are stored and returned in ISO format
- CORS is enabled for all routes
- Error handling follows the existing pattern in other routes

### Questions?
If you have questions during implementation, please refer to:
1. The existing routes in routes/study_sessions.py
2. The database schema in sql/setup/
3. The test configuration in conftest.py

## TODO: Other Study Session Endpoints
- [ ] POST /study_sessions/:id/review
- [ ] Other endpoints as needed...
