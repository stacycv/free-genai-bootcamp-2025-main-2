# Technical Specs

## Business Goal

A language learning school wants to build a prototype of learning portal which will act as three things:
- Inventory of possible vocabulary that can be learned
- Act as a  Learning record store (LRS), providing correct and wrong score on practice vocabulary
- A unified launchpad to launch different learning apps

## Technical Requirements

- The backend should be built using Go.
- The database should will be SQLite3.
- The API will be built using Gin
- The API will always return JSON.
- There will be no authentication or authorization. 
- Everything will be public and treated as a single user.

## Database Schema

We have the following tables in the database:

- words - stores all words that can be learned
    - id - integer primary key
    - Formal Spanish  - string
    - Informal Spanish - string
    - English - string
    - parts - json
- word_groups  - join table for wods and groups (many-to-many)
    - id - integer
    - word_id - integer
    - group_id - integer
- groups - thematic groups of words
    - id - integer primary key
    - name - string
- study_sessions - records of study sessions grouping word_review_items
    - id - integer 
    - group_id - integer
    - created_at - timestamp     
    - study_activity_id - integer
- study_activities - a specific instance of a study session, linking a study_session to a group
    - id - integer
    - study_session_id - integer
    - group_id - integer
    - created_at - timestamp
- word_review_items - a record of word practice, determining if the word was answered correctly or incorrectly
    - word_id - integer
    - study_session_id - integer
    - correct - boolean
    - created_at - timestamp   
## API Endpoints
- GET /api/dashboard/last-study-session
- GET /api/dashboard/study-progress
- GET /api/dashboard/quick-stats
- GET /api/study-activities/:id
- GET /api/study-activities/:id/study-sessions 
- POST /api/study-activities/
    - required params: group_id, study_activity_id
- GET /api/words
    - pagination with 100 items per page
- GET /api/words/:id
- GET /api/groups
    - pagination with 100 items per page
- GET /api/groups/:id/words
- GET /api/groups/:id/study-sessions
- GET /api/study-sessions
    - pagination with 100 items per page
- GET /api/study-sessions/:id
- POST /api/reset-history
- POST /api/full-reset
- POST /api/study-sessions/:id/words/:word_id/review
    - required params: correct
    




