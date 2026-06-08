# SmartLab – Laboratory Equipment Booking System

SmartLab is a web-based laboratory equipment scheduling and reservation system designed to streamline the management of shared lab resources. The platform enables students to reserve equipment, prevents double-booking through conflict detection, manages priority-based waitlists, and automatically promotes eligible waitlisted users when bookings are cancelled.

The project demonstrates the practical application of Data Structures and Algorithms such as Hash Maps, Priority Queues, Heaps, and Scheduling Logic in solving a real-world resource allocation problem.

---

## Features

* Equipment reservation and scheduling
* Real-time equipment status monitoring
* Conflict-free booking system
* Priority-based waitlist management
* Automatic waitlist promotion on cancellation
* Active and upcoming booking tracking
* Booking logs and history
* Dashboard statistics and analytics
* Live notifications and alerts
* JSON-based persistent storage
* Responsive web interface

---

## Technology Stack

### Backend

* Python
* Flask

### Frontend

* HTML5
* CSS3
* JavaScript (Vanilla JS)

### Storage

* JSON Database

---

## Data Structures & Algorithms Used

### Hash Maps (Python Dictionaries)

Used for efficient equipment storage and retrieval.

### Lists

Used to manage bookings, equipment records, and waitlists.

### Priority Queue (Heap)

Implemented using Python's `heapq` module to rank waitlisted users based on:

* Academic Year Priority
* Request Timestamp

### Interval Conflict Detection

Prevents overlapping reservations using interval scheduling logic.

---

## How It Works

### Booking Workflow

1. Student selects equipment and time slot.
2. System checks for scheduling conflicts.
3. If no conflict exists:

   * Booking is confirmed.
   * Data is stored.
4. If a conflict exists:

   * User is offered the option to join the waitlist.

### Waitlist Workflow

1. Conflicting users can join a priority waitlist.
2. Waitlist entries are ranked based on:

   * Academic year
   * Request time
3. When a booking is cancelled:

   * Highest-priority compatible request is automatically promoted.

---

## Project Structure

```text
equipment-booking-system/
│
├── PROJECT_DOCUMENTATION.md
│
├── SmartLab/
│   ├── app.py
│   ├── database.json
│   │
│   ├── templates/
│   │   └── index.html
│   │
│   └── static/
│       ├── app.js
│       └── style.css
│
└── venv/
```

---

## Running the Project

### Clone Repository

```bash
git clone <repository-url>
cd equipment-booking-system
```

### Create Virtual Environment

```bash
python -m venv venv
```

### Activate Virtual Environment

Windows:

```bash
venv\Scripts\activate
```

Linux / macOS:

```bash
source venv/bin/activate
```

### Install Dependencies

```bash
pip install Flask
```

### Start Application

```bash
python SmartLab/app.py
```

### Open Browser

```text
http://localhost:5001
```

---

## Sample Equipment

The system includes the following default equipment:

* Oscilloscope
* 3D Printer
* Soldering Station
* Arduino Kit
* Raspberry Pi

---

## Real-World Applications

* Engineering Laboratories
* Research Facilities
* University Resource Management
* Innovation and Maker Spaces
* Industrial Testing Labs
* Shared Equipment Scheduling Systems

---

## Future Enhancements

* Authentication and Role-Based Access Control
* Database Integration (MySQL/PostgreSQL)
* Automated Expiry-Based Waitlist Promotion
* Admin Dashboard
* Equipment Maintenance Mode
* Email Notifications
* Report Generation and Export

---

## Learning Outcomes

This project demonstrates:

* Application of Data Structures in real systems
* Priority Queue and Heap implementation
* Scheduling and Conflict Detection Algorithms
* Client-Server Architecture
* REST API Development
* Frontend-Backend Integration
* Persistent Data Management

---

## Contributors

* S Tilak
* Satish
* Sakshi

---

## License

This project is developed for educational and academic purposes.
