# ASTRA Backend

**Advanced System for Testbed Recording and Analysis**
NASA JPL Capstone Project - Backend Service

## Quick Start

```bash
# 1. Create virtual environment
python -m venv venv
source venv/bin/activate   # Mac/Linux
# venv\Scripts\activate    # Windows

# 2. Install dependencies
pip install -r requirements.txt

# 3. Start server
uvicorn app.main:app --reload

# 4. Open browser
# API Docs: http://localhost:8000/docs
# Health:   http://localhost:8000/health
```

## Project Structure

```
astra-backend/
├── app/
│   ├── main.py           # FastAPI application entry point
│   ├── database.py       # In-memory storage (dev/test)
│   ├── routes/
│   │   ├── sessions.py   # Session management
│   │   ├── notes.py      # Notes CRUD + export
│   │   ├── telemetry.py  # Telemetry ingestion & query
│   │   └── websocket.py  # Real-time updates
│   ├── schemas/
│   │   └── schemas.py    # Pydantic models (API formats)
│   └── models/           # Future: switch to other models
├── smoke_test.py         # API validation script
├── requirements.txt
└── README.md
```

## API Endpoints

### Sessions
| Method | Endpoint | Description | Called By |
|--------|----------|-------------|-----------|
| POST | `/api/sessions` | Create new session | Frontend |
| GET | `/api/sessions` | List all sessions | Frontend |
| GET | `/api/sessions/{sid}` | Get specific session | Frontend |
| PATCH | `/api/sessions/{sid}` | Update metadata (e.g., status=ended) | Frontend |

### Notes (Session-Scoped)
| Method | Endpoint | Description | Called By |
|--------|----------|-------------|-----------|
| POST | `/api/sessions/{sid}/notes` | Create note | AI/Data Module |
| GET | `/api/sessions/{sid}/notes` | List notes (filters: speaker, type, from/to) | Frontend |
| GET | `/api/sessions/{sid}/notes/export` | Export as Markdown or JSON | Frontend |
| GET | `/api/sessions/{sid}/notes/{id}` | Get specific note | Frontend |
| PUT | `/api/sessions/{sid}/notes/{id}` | Edit note (operator correction) | Frontend |
| DELETE | `/api/sessions/{sid}/notes/{id}` | Delete note | Frontend |

### Telemetry (Session-Scoped)
| Method | Endpoint | Description | Called By |
|--------|----------|-------------|-----------|
| POST | `/api/sessions/{sid}/telemetry` | Ingest telemetry | Telemetry Source |
| POST | `/api/sessions/{sid}/telemetry/batch` | Batch ingest | Telemetry Source |
| GET | `/api/sessions/{sid}/telemetry` | Query (filters: channel, from/to) | Frontend / AI |
| GET | `/api/sessions/{sid}/telemetry/latest?channel=X` | Get latest value | AI Module |
| GET | `/api/sessions/{sid}/telemetry/channels` | List available channels | Frontend |

### WebSocket
| Method | Endpoint | Description | Called By |
|--------|----------|-------------|-----------|
| WS | `/ws/sessions/{sid}` | Real-time note updates | Frontend |

## Data Formats

### Note Object
```json
{
  "timestamp": "2025-01-26T10:32:15Z",
  "speaker": "Engineer A",
  "content": "Motor current rising",
  "type": "observation",
  "tags": ["motor", "current"],
  "telemetry_snapshot": {
    "battery_voltage": 32.5,
    "motor_current": 2.3
  }
}
```

### Telemetry Object
```json
{
  "timestamp": "2025-01-26T10:32:15Z",
  "channel": "battery_voltage",
  "value": 32.5,
  "unit": "V"
}
```

### WebSocket Message
```json
{
  "event": "note.created",
  "data": { ... note object ... }
}
```

## Export Notes

Export notes as Markdown (for copy/paste to other systems):
```
GET /api/sessions/{sid}/notes/export?format=markdown
```

Export as JSON:
```
GET /api/sessions/{sid}/notes/export?format=json
```

## Testing

Run the smoke test to validate all API endpoints:

```bash
# Make sure backend is running first
uvicorn app.main:app --reload

# In another terminal, run smoke test
python smoke_test.py
```

The smoke test validates:
- Session CRUD (create, list, get, update)
- Notes CRUD + export (Markdown/JSON)
- Telemetry ingestion + query
- WebSocket connectivity
- End-to-end flow

Expected output: `All tests passed! Backend is ready for integration.`


## Development Status

- [x] Session management
- [x] Notes CRUD
- [x] Notes export (Markdown/JSON)
- [x] Telemetry ingestion & query
- [x] WebSocket real-time push
- [x] In-memory storage (dev)
- [ ] PostgreSQL integration
- [ ] Authentication

## Tech Stack

- **Framework:** FastAPI
- **Validation:** Pydantic
- **WebSocket:** FastAPI native
- **Database:** In-memory (dev) → PostgreSQL (prod)
