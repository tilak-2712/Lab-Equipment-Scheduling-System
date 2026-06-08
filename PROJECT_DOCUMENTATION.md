# SmartLab Equipment Booking System

## 1. Project Overview

SmartLab is a browser-based lab equipment scheduling system built as a small
Flask application. It allows students to reserve equipment for a date and time,
prevents overlapping reservations, maintains a priority waitlist when a slot is
already occupied, and promotes an eligible waiting student when a booking is
cancelled.

The application is designed as a single-page dashboard for a college laboratory.
The interface identifies Dayananda Sagar College of Engineering and uses student
USNs and academic years as booking and priority information.

The project has no separate frontend framework or database server. Flask serves
the page and API, vanilla JavaScript manages the browser interface, and a JSON
file provides local persistence.

## 2. Main Capabilities

- Display all registered laboratory equipment.
- Show live equipment states: `AVAILABLE`, `BOOKED`, or `IN USE`.
- Create date-and-time-based equipment bookings.
- Reject overlapping time slots for the same equipment.
- Offer the user a priority waitlist after a booking conflict.
- Rank waiting students by academic year and request time.
- Cancel current or upcoming bookings.
- Automatically promote a suitable waitlisted request after cancellation.
- Display active and upcoming reservations.
- Display booking records with computed status.
- Notify the browser when a booking starts or expires.
- Show dashboard statistics and a live clock.
- Clear all booking and waitlist data through the interface.
- Refresh data from the server every 30 seconds.

## 3. Technology Stack

### Backend

- Python
- Flask
- Python standard library:
  - `json` for persistence
  - `os` for file paths
  - `heapq` for waitlist promotion
  - `datetime` for timestamps

### Frontend

- HTML5
- CSS3
- Vanilla JavaScript
- Browser Fetch API

### Storage

- Local JSON file: `SmartLab/database.json`

No SQL database, ORM, frontend framework, authentication provider, or external
JavaScript package is used.

## 4. Project Structure

```text
equipment-booking-system/
|-- PROJECT_DOCUMENTATION.md
|-- SmartLab/
|   |-- app.py
|   |-- database.json
|   |-- templates/
|   |   `-- index.html
|   `-- static/
|       |-- app.js
|       `-- style.css
`-- venv/
```

### File Responsibilities

| File | Purpose |
|---|---|
| `SmartLab/app.py` | Flask server, API endpoints, validation, conflict detection, waitlist logic, cancellation, and JSON persistence. |
| `SmartLab/database.json` | Persistent equipment, booking, waitlist, and booking-counter data. |
| `SmartLab/templates/index.html` | Single-page dashboard structure and booking form. |
| `SmartLab/static/app.js` | API calls, form handling, dynamic rendering, live status calculations, notifications, modals, and polling. |
| `SmartLab/static/style.css` | Dark responsive dashboard theme, layouts, cards, tables, status indicators, animations, and notification styles. |
| `venv/` | Local Python virtual environment containing installed dependencies. It is environment-specific rather than application source code. |

## 5. Default Equipment

The application starts with five equipment records:

| ID | Equipment |
|---|---|
| `E001` | Oscilloscope |
| `E002` | 3D Printer |
| `E003` | Soldering Station |
| `E004` | Arduino Kit |
| `E005` | Raspberry Pi |

Each equipment record contains its ID, name, stored status, selected current
booking, complete booking list, and waitlist.

## 6. Data Model

The JSON database has this top-level shape:

```json
{
  "equipment": {
    "E001": {
      "id": "E001",
      "name": "Oscilloscope",
      "status": "AVAILABLE",
      "current_booking": null,
      "bookings": [],
      "waitlist": []
    }
  },
  "booking_counter": 1
}
```

### Booking Record

```json
{
  "booking_id": "B0001",
  "usn": "1DS21CS001",
  "equipment_id": "E001",
  "start_time": "2026-06-07T10:00:00",
  "end_time": "2026-06-07T11:00:00",
  "priority_year": "4th Year",
  "booked_at": "2026-06-07T09:15:00.000000"
}
```

Normal booking IDs are sequential (`B0001`, `B0002`, and so on) and use the
database's `booking_counter`.

### Waitlist Record

```json
{
  "usn": "1DS21CS002",
  "equipment_id": "E001",
  "start_time": "2026-06-07T10:00:00",
  "end_time": "2026-06-07T11:00:00",
  "priority_year": "3rd Year",
  "priority": 3,
  "timestamp": 1780803900.0,
  "booking_id": "W1780803900"
}
```

The numeric priority and timestamp are used to order requests. Promoted
waitlist entries retain their `W...` identifier as the resulting booking ID.

## 7. User Interface

The page served at `/` is a dashboard with the following sections.

### Navigation Bar

- SmartLab branding
- College name
- Live 24-hour clock updated every second

### Statistics Bar

- Total equipment
- Available equipment
- Equipment currently in use
- Equipment with upcoming bookings

### Equipment Cards

Each card shows:

- Equipment name and ID
- Live computed status
- USN and slot for the active or next booking
- Cancellation button when applicable
- Number of waiting requests

### Booking Form

The form accepts:

- Student USN
- Equipment
- Booking date
- Start time
- End time
- Academic year

The academic-year choices are first through fourth year, with fourth year
selected by default.

### Active Bookings

This panel shows reservations whose end time has not passed. Entries currently
within their slot are shown before future entries, followed by upcoming entries
ordered by start time.

### Priority Waitlist

The combined waitlist table shows:

- Overall rank
- USN
- Equipment
- Requested time slot
- Academic year

The first-ranked row receives special visual emphasis.

### Booking Logs

The logs table shows all bookings currently stored in equipment booking arrays.
Each record is labelled `In Use`, `Upcoming`, or `Completed` according to the
browser's current time.

The Clear button removes all bookings and waitlist entries after confirmation.

### Notifications

- Success, error, information, and warning toasts
- Conflict modal offering waitlist registration
- Confirmation modal before cancellation
- Confirmation modal before clearing records
- Persistent dismissible banner when a booking expires while the page is open

## 8. Booking Workflow

1. The student enters a USN.
2. The student chooses equipment, date, start time, end time, and academic year.
3. Browser validation checks that all fields are present.
4. Browser validation checks that the end time is later than the start time.
5. JavaScript combines date and time into ISO-like local datetime strings.
6. The browser sends the request to `POST /api/book`.
7. Flask verifies required fields and confirms that the equipment exists.
8. The backend compares the requested interval with every existing booking for
   that equipment.
9. If there is no overlap, the backend generates a booking ID, stores the
   booking, increments the counter, and writes the JSON database.
10. The browser displays a success message and reloads dashboard data.
11. If there is an overlap, the API returns `CONFLICT`, and the browser asks
    whether the student wants to join the priority waitlist.

## 9. Conflict Detection

Bookings are treated as half-open intervals:

```text
[start_time, end_time)
```

A requested slot conflicts with an existing slot when:

```text
new_start < existing_end AND new_end > existing_start
```

This means adjacent bookings are allowed. For example, a booking ending at
11:00 does not conflict with another booking starting at 11:00.

Conflict detection is performed independently for each equipment item, so two
different equipment items may be booked for the same period.

The implementation compares ISO-formatted datetime strings. This works for the
consistent `YYYY-MM-DDTHH:MM:SS` format generated by the frontend.

## 10. Priority Waitlist Workflow

When a requested slot conflicts with an existing booking:

1. The browser displays a conflict modal.
2. The student may choose `Join Waitlist`.
3. JavaScript submits the original booking details to `POST /api/waitlist`.
4. The backend converts the academic year to a numeric priority:

| Academic Year | Priority |
|---|---:|
| 4th Year | 4 |
| 3rd Year | 3 |
| 2nd Year | 2 |
| 1st Year | 1 |

5. A timestamp is recorded as a tie-breaker.
6. The equipment waitlist is sorted by descending priority, then ascending
   timestamp.

Therefore, senior students rank above junior students, while students in the
same academic year are handled first-in, first-out.

The dashboard combines all equipment waitlists into one display and sorts them
using the same rules. Actual promotion is still performed within the cancelled
equipment's own waitlist.

## 11. Cancellation and Automatic Promotion

When a user clicks `Cancel Booking`:

1. The browser asks for confirmation.
2. It sends the booking ID and equipment ID to `POST /api/cancel`.
3. The backend removes that booking from the equipment's booking list.
4. If the equipment has waiting requests, the backend builds a heap ordered by
   highest priority and earliest timestamp.
5. Requests are checked in priority order.
6. The first request whose desired interval no longer conflicts with a remaining
   booking is promoted.
7. Requests that still conflict remain on the waitlist.
8. The equipment's stored status and current booking are recalculated.
9. The JSON database is saved and the browser refreshes its display.

Only one waiting request is promoted per cancellation.

## 12. Live Status Logic

The frontend does not rely only on the status saved in `database.json`. It
recomputes status using the browser's current time:

- `IN USE`: current time is at or after a booking's start and before its end.
- `BOOKED`: no booking is active, but at least one booking starts in the future.
- `AVAILABLE`: no active or future booking exists.

The status monitor runs every second. It:

- Refreshes equipment cards and statistics when a status changes.
- Displays an informational toast when an upcoming booking becomes active.
- Displays an expiry banner when a booking ends while the page is open.

Server data is fetched immediately at startup and every 30 seconds. The live
status calculations between fetches happen entirely in the browser.

## 13. API Reference

### `GET /`

Renders `templates/index.html`.

### `GET /api/equipment`

Returns the complete equipment registry as an array.

Successful response:

```json
{
  "success": true,
  "equipment": []
}
```

### `POST /api/book`

Creates a booking when the slot is available.

Request:

```json
{
  "usn": "1DS21CS001",
  "equipment_id": "E001",
  "start_time": "2026-06-07T10:00:00",
  "end_time": "2026-06-07T11:00:00",
  "priority_year": "4th Year"
}
```

Success:

```json
{
  "success": true,
  "booking_id": "B0001"
}
```

Conflict:

```json
{
  "success": false,
  "reason": "CONFLICT"
}
```

### `POST /api/waitlist`

Adds a request to an equipment waitlist.

The request body uses the same fields as `/api/book`.

### `GET /api/waitlist/<equipment_id>`

Returns the waitlist for one equipment item.

### `POST /api/cancel`

Cancels a booking.

Request:

```json
{
  "booking_id": "B0001",
  "equipment_id": "E001"
}
```

### `POST /api/clear_logs`

Clears every booking and waitlist, resets each equipment to `AVAILABLE`, and
resets `booking_counter` to `1`.

### Error Behavior

- Missing fields: HTTP `400`
- Unknown equipment: HTTP `404`
- Unknown booking during cancellation: HTTP `404`
- Booking conflict: HTTP `200` with `success: false` and `reason: "CONFLICT"`

The application adds permissive CORS headers to every response and allows
`GET`, `POST`, and `OPTIONS`.

## 14. Running the Project

Run commands from the attached folder:

```bash
cd SmartLab
../venv/bin/python app.py
```

Then open:

```text
http://localhost:5001
```

The server binds to `0.0.0.0` on port `5001`, so it may also be reachable from
other devices on the same network if the host firewall permits it.

If the included virtual environment cannot be used on another machine, create a
new one and install Flask:

```bash
python3 -m venv venv
source venv/bin/activate
pip install Flask
python SmartLab/app.py
```

There is currently no `requirements.txt`; Flask is the project's direct
third-party runtime dependency.

## 15. Persistence and Initialization

`app.py` uses an absolute path based on its own location, so `database.json` is
found correctly regardless of the shell's working directory.

When `app.py` is executed directly, `init_db()` creates `database.json` from the
default data only if the file does not already exist. Existing data is preserved
across server restarts.

Each modifying API operation reads the full JSON file, changes in-memory data,
and rewrites the full file with indentation.

## 16. Design and Responsive Behavior

The CSS implements a dark dashboard with:

- Indigo, purple, cyan, green, amber, and red status accents
- Responsive card and table layouts
- Two-column desktop layout with a fixed-width right panel
- Single-column layout below 900 pixels
- Animated card entry, status pulse, spinners, toasts, and banners
- Dark-styled date, time, select, table, and scrollbar controls

The page uses semantic sections, labels, table headings, and selected ARIA
attributes. It is responsive, but wide tables may still require horizontal
space on smaller screens.

## 17. Important Current Limitations

These are characteristics of the code in its current form:

- There is no login, role system, or authorization. Any visitor can book,
  cancel, join a waitlist, or clear all records.
- Cancellation requires only booking and equipment IDs; ownership is not
  verified.
- The clear-logs endpoint has no administrator protection.
- The backend checks for missing fields but does not validate datetime format,
  verify that end time follows start time, or reject past bookings. Those checks
  mainly exist in the browser and can be bypassed by direct API calls.
- Academic-year values are not strictly validated. Unknown values receive
  priority `1`.
- Duplicate bookings or duplicate waitlist entries for the same student are not
  prevented.
- The JSON file has no file locking or transaction handling. Simultaneous writes
  can overwrite data or corrupt the file.
- A single-process local JSON store is unsuitable for high traffic or
  multi-server deployment.
- Completed bookings remain in each equipment's `bookings` list until cancelled
  or all logs are cleared.
- The backend's stored `status` may become stale as time passes; the frontend
  computes the accurate live status independently.
- Waitlist promotion happens only after cancellation, not automatically when a
  booking naturally expires.
- A cancellation promotes at most one compatible request, even if multiple
  non-overlapping waiting slots could now be accepted.
- Promoted waitlist bookings use `W...` IDs rather than the sequential booking
  counter.
- Timestamps and datetime strings do not carry explicit timezone offsets.
- Browser and server clocks can disagree, affecting displayed state.
- The frontend inserts data into HTML strings without escaping, which creates a
  cross-site-scripting risk if malicious values are submitted.
- CORS is open to all origins.
- Flask's development server is used directly and is not a production server.
- There are no automated tests, dependency lock file, deployment configuration,
  logging system, backup process, or recovery handling for malformed JSON.

## 18. Suitable Use

The current project is appropriate as:

- A college mini-project or demonstration
- A data-structures example using maps, vectors/lists, intervals, and a priority
  queue
- A small single-user or low-concurrency local lab scheduler
- A prototype for a larger authenticated reservation platform

For production use, the main next steps would be authentication and permissions,
a transactional database, strict server-side validation, protected
administrative operations, safe HTML rendering, automated tests, and deployment
through a production WSGI server.

## 19. End-to-End Summary

Flask serves the dashboard and exposes JSON APIs. The browser loads the equipment
registry, computes each item's status from booking times, and renders all cards,
statistics, lists, and tables. A booking request is accepted only if its interval
does not overlap an existing booking for that equipment. Conflicting users may
join a queue ranked by academic year and request order. Cancelling a booking
causes the backend to search that equipment's queue and promote the
highest-priority compatible request. Every change is persisted by rewriting
`database.json`, while the browser updates live every second and synchronizes
with the server every 30 seconds.
