# Flask Full CRUD RESTful API — Event Management

A simple RESTful API built with Python and Flask that demonstrates full CRUD
(Create, Read, Update, Delete) operations on an in-memory list of events.

## Purpose

This API simulates a basic event management system. It allows clients to
create, read, update, and delete event records, following RESTful naming
conventions and returning structured JSON responses with appropriate HTTP
status codes.

## Tech Stack

- Python 3
- Flask

## Setup

1. Clone the repository:
```bash
   git clone <your-fork-url>
   cd <repo-folder>
```

2. Install dependencies:
```bash
   pip install flask pytest
```

3. Run the server:
```bash
   python app.py
```

   The API will be available at `http://localhost:5000`.

4. Run the tests:
```bash
   pytest test_app.py -v
```

## Data Model

Each event has the following shape:

```json
{
  "id": 1,
  "title": "Tech Meetup"
}
```

Data is stored in memory (a Python list of `Event` objects) and resets every
time the server restarts.

## Routes

### `GET /`
Returns a welcome message.

**Response — 200 OK**
```json
{
  "message": "Welcome to the Event Management API"
}
```

---

### `GET /events`
Returns a list of all events.

**Response — 200 OK**
```json
[
  { "id": 1, "title": "Tech Meetup" },
  { "id": 2, "title": "Python Workshop" }
]
```

---

### `POST /events`
Creates a new event. Requires a `title` field in the JSON body.

**Request**
```json
{ "title": "Hackathon" }
```

**Response — 201 Created**
```json
{ "id": 3, "title": "Hackathon" }
```

**Response — 400 Bad Request** (missing/empty title)
```json
{ "error": "Missing required field: title" }
```

---

### `PATCH /events/<id>`
Updates the title of an existing event.

**Request**
```json
{ "title": "Hackathon 2025" }
```

**Response — 200 OK**
```json
{ "id": 3, "title": "Hackathon 2025" }
```

**Response — 404 Not Found** (no event with that id)
```json
{ "error": "Event with id 99 not found" }
```

**Response — 400 Bad Request** (missing/empty title)
```json
{ "error": "Missing required field: title" }
```

---

### `DELETE /events/<id>`
Removes an event from the list.

**Response — 204 No Content**
(empty body)

**Response — 404 Not Found** (no event with that id)
```json
{ "error": "Event with id 99 not found" }
```

## Example curl Requests

```bash
# Create an event
curl -X POST http://localhost:5000/events \
  -H "Content-Type: application/json" \
  -d '{"title": "Hackathon"}'

# Update an event
curl -X PATCH http://localhost:5000/events/1 \
  -H "Content-Type: application/json" \
  -d '{"title": "Hackathon 2025"}'

# Delete an event
curl -X DELETE http://localhost:5000/events/2
```

## Design Notes

- **Validation**: Every write operation checks that `title` is present and
  non-empty before mutating data, returning a `400` otherwise.
- **Lookup helper**: A `find_event(event_id)` helper centralizes the
  "search by id" logic used by both `PATCH` and `DELETE`, avoiding
  duplication.
- **Not found handling**: `PATCH` and `DELETE` both return `404` with a
  clear message when the id doesn't match any event.
- **Scalability**: Logic is kept in small, focused route handlers. The
  in-memory list can later be swapped for a real database (e.g.
  SQLAlchemy) without changing the route contracts.

## Status Codes Summary

| Method | Route            | Success | Error Cases       |
|--------|-------------------|---------|--------------------|
| GET    | `/`               | 200     | —                  |
| GET    | `/events`         | 200     | —                  |
| POST   | `/events`         | 201     | 400                |
| PATCH  | `/events/<id>`    | 200     | 400, 404           |
| DELETE | `/events/<id>`    | 204     | 404                |