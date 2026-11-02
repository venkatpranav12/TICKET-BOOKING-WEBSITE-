# CineBook 🎬 — Full-Stack Movie Ticket Booking System

A full-stack movie ticket booking web application built with **Django**, **SQLite**, and modern **vanilla HTML5/CSS/JavaScript**.

CineBook features a cinema-inspired UI (Midnight Black, Cinematic Red, and Warm Gold) featuring real-time seat selection, multi-seat atomic reservations, promotional coupon validation (`CINE10`), unique sequential booking IDs (`BK1001`), printable tickets, and one-click cancellations with an **80% refund policy**.

---

## 🌟 Key Features

1. **OTT-Inspired Discovery Experience**:
   - Cinematic hero banner featuring popular releases.
   - Now Showing and Coming Soon movie sections.
   - Real-time search and filter by genre and language.
   - Comprehensive movie detail pages with synopsis, metadata, and showtime selectors.
2. **Interactive 5 × 8 Cinema Seat Map**:
   - Curved screen indicator (`SCREEN THIS WAY`).
   - 40 independent seats per show arranged across 5 rows (A–E) and 8 columns (1–8).
   - Seat categories:
     - **Regular**: Rows A & B (`₹150.00`)
     - **Premium**: Rows C & D (`₹250.00`)
     - **VIP**: Row E (`₹400.00`)
   - Visual states: *Available*, *Selected* (cinematic red glow), and *Booked / Unavailable* (disabled).
   - Live estimated total, chip counters, and coupon input.
3. **Atomic Booking & Concurrency**:
   - Concurrency-safe booking powered by Django's `transaction.atomic()`.
   - All-or-nothing seat reservations: if any selected seat is unavailable or invalid, all changes roll back.
   - Re-checks seat availability and calculates pricing server-side (never trusts client-side figures).
4. **Coupon Discounts**:
   - Promo code **`CINE10`** provides an instant **10% discount** on ticket subtotals.
   - Verified server-side with support for date validity and max discount ceilings.
5. **Cancellations & 80% Refund Policy**:
   - Authenticated users can cancel their confirmed bookings with a single click.
   - Returns **80%** of the amount actually paid; the remaining 20% is retained as a cancellation fee.
   - Releases the reserved seats back into the show inventory while preserving historical booking snapshots (`BookingSeat`).
6. **Authentication & Ownership**:
   - Custom login and registration pages styled to match the dark cinema theme.
   - Intended booking destination preserved via `?next=...` redirects.
   - Ownership verification on every booking and cancellation endpoint (users cannot view or cancel others' tickets).
7. **Printable E-Tickets**:
   - Clean, printable cinema pass complete with perforated styling, barcode simulation, and print mode (`window.print()`).

---

## 🏗️ Architecture & OOP Concept Mapping

This project maps classical Object-Oriented Programming (OOP) and software design principles into idiomatic Django/Python:

| Classical OOP / Java Concept | CineBook Implementation | File Reference |
| :--- | :--- | :--- |
| **Enum with Fields & Logic** | `SeatType(Enum)` defines code, label, price (`Decimal`), and row mapping. Single source of truth for pricing. | [`core/enums.py`](file:///Users/venkatpranav/Downloads/TICKET%20BOOKING%20/core/enums.py) |
| **Interface / Contract** | `Bookable(ABC)` declares abstract methods `book_seats()` and `cancel_booking()`. | [`core/interfaces.py`](file:///Users/venkatpranav/Downloads/TICKET%20BOOKING%20/core/interfaces.py) |
| **Service Layer Pattern** | `BookingService(Bookable)` encapsulates domain transactions, reservations, and refund calculations outside views. | [`bookings/services.py`](file:///Users/venkatpranav/Downloads/TICKET%20BOOKING%20/bookings/services.py) |
| **Composition** | `Theatre` has multiple `Shows`; each `Show` has its own isolated set of 40 `Seat` records. | [`cinema/models.py`](file:///Users/venkatpranav/Downloads/TICKET%20BOOKING%20/cinema/models.py) |
| **Two-Dimensional Array** | Genuine 5 × 8 matrix generated using nested loops: `matrix[row_idx][col_idx]`. | `build_seat_matrix_2d()` in [`bookings/services.py`](file:///Users/venkatpranav/Downloads/TICKET%20BOOKING%20/bookings/services.py) |
| **Method Overloading** | Python optional keyword arguments equivalent: `calculate_total(seats, coupon_code=None)`. | `calculate_total()` in [`bookings/services.py`](file:///Users/venkatpranav/Downloads/TICKET%20BOOKING%20/bookings/services.py) |
| **Historical Snapshots** | `BookingSeat` model records seat code, type, and price at time of booking, preserved even if the booking is cancelled. | [`bookings/models.py`](file:///Users/venkatpranav/Downloads/TICKET%20BOOKING%20/bookings/models.py) |
| **Sequential ID Generator** | `BookingIdGenerator` generates collision-free IDs like `BK1001`, `BK1002`. | `BookingIdGenerator` in [`bookings/services.py`](file:///Users/venkatpranav/Downloads/TICKET%20BOOKING%20/bookings/services.py) |

---

## 📁 Project Structure

```
cinebook/
├── manage.py
├── requirements.txt
├── README.md
├── .env.example
├── .env
├── config/
│   ├── settings.py           # App configuration, security, message tags
│   ├── urls.py               # Main URL router
│   ├── views.py              # Custom 403, 404, 500 handlers
│   ├── wsgi.py
│   └── asgi.py
├── core/
│   ├── enums.py              # SeatType (pricing source of truth), BookingStatus
│   ├── interfaces.py         # Bookable ABC contract
│   └── utils.py              # Seat parsing, validation, custom exceptions
├── movies/
│   ├── models.py             # Movie catalog model
│   ├── views.py              # Home & movie detail views
│   └── urls.py
├── cinema/
│   ├── models.py             # Theatre, Show, Seat models
│   ├── views.py              # Seat selection page & show seats API
│   ├── urls.py
│   └── management/
│       └── commands/
│           └── seed_cinebook.py  # Repeatable database seeding command
├── bookings/
│   ├── models.py             # Booking, BookingSeat, Coupon models
│   ├── views.py              # Booking APIs, ticket summary, auth views
│   ├── urls.py
│   ├── services.py           # BookingService, 2D matrix, calculate_total
│   ├── forms.py              # Registration form
│   └── tests.py              # 18 automated test cases
├── templates/
│   ├── base.html             # Global cinematic navbar, alerts, modal, footer
│   ├── 403.html, 404.html, 500.html
│   ├── movies/
│   │   ├── home.html         # Hero banner, Now Showing & Upcoming grids
│   │   └── movie_detail.html # Synopsis, metadata, showtime picker
│   ├── cinema/
│   │   └── seat_selection.html # 5x8 seat map & live checkout sidebar
│   ├── bookings/
│   │   ├── booking_confirmation.html # Printable ticket summary
│   │   └── my_bookings.html  # User bookings list & cancellation modal
│   └── registration/
│       ├── login.html
│       └── register.html
└── static/
    ├── css/
    │   └── style.css         # Midnight black, cinematic red & warm gold design system
    └── js/
        ├── app.js            # Modals, alerts, image fallbacks
        └── seat-map.js       # Interactive seat grid, Fetch API & live math
```

---

## 🚀 Getting Started (Setup & Execution)

### 1. Prerequisites
- Python 3.10+ (tested with Python 3.14)
- `pip` package manager

### 2. Virtual Environment Setup
Open your terminal in the project directory:

**macOS / Linux:**
```bash
python3 -m venv .venv
source .venv/bin/activate
```

**Windows:**
```powershell
python -m venv .venv
.venv\Scripts\activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

### 4. Apply Database Migrations
```bash
python manage.py makemigrations
python manage.py migrate
```

### 5. Seed Initial Data
Run the repeatable seeding command to populate initial movies, theatres, shows, seats, coupons, and a demo user:
```bash
python manage.py seed_cinebook
```

This creates:
- 6 Movies (*Interstellar, Inception, The Dark Knight, Dune 2, Spider-Man, The Martian*)
- 2 Theatres with multiple show dates/times
- 40 independent seats (A1–E8) for each show
- Active **`CINE10`** coupon (10% off)
- Demo User credentials:
  - **Username**: `cinefan`
  - **Password**: `Cinebook@2026`

### 6. Run Automated Tests
Verify all 18 automated unit and integration tests:
```bash
python manage.py test
```

### 7. Start the Development Server

To access the website from your development computer **and** other devices on the same Wi-Fi (Android, iPhone, iPad, other laptops), listen on all network interfaces:
```bash
python manage.py runserver 0.0.0.0:8000
```

Access URLs:
- **Local machine**: [http://127.0.0.1:8000/](http://127.0.0.1:8000/) or [http://localhost:8000/](http://localhost:8000/)
- **Mobile / Other Devices on Same Wi-Fi**: `http://<YOUR_LAN_IP>:8000/` (e.g. `http://192.168.1.3:8000/`)

*(Optional)* Create a Django superuser for administrative access at `/admin/`:
```bash
python manage.py createsuperuser
```

---

## 🧪 Automated Test Suite Coverage

The test suite in [`bookings/tests.py`](file:///Users/venkatpranav/Downloads/TICKET%20BOOKING%20/bookings/tests.py) validates 18 distinct test scenarios:
1. `test_seat_category_mapping_and_pricing`: Validates Regular (₹150), Premium (₹250), and VIP (₹400) seat pricing.
2. `test_book_single_seat`: Validates reserving a single seat and snapshot creation.
3. `test_book_multiple_seats`: Validates multi-category seat reservations and sum calculation.
4. `test_invalid_seat_codes`: Rejects out-of-range codes (e.g. `Z9`, `A0`, `A9`, blank).
5. `test_duplicate_seat_codes`: Prevents submitting duplicate codes in one request.
6. `test_book_already_booked_seat`: Prevents booking a seat that is already reserved.
7. `test_all_or_nothing_behavior`: Verifies atomic rollback when any selected seat is unavailable.
8. `test_cine10_coupon_calculation`: Validates 10% discount deduction with `CINE10`.
9. `test_invalid_coupon_rejection`: Rejects non-existent or expired coupon codes.
10. `test_booking_id_generation_and_uniqueness`: Ensures unique, sequential `BK1001` format.
11. `test_independent_seat_grids_across_shows`: Confirms booking seat A1 in Show 1 does not affect Show 2.
12. `test_cancellation_and_80_percent_refund`: Validates 80% refund and 20% cancellation fee deduction.
13. `test_prevention_of_duplicate_cancellations`: Rejects attempting to cancel an already cancelled booking.
14. `test_seat_availability_after_cancellation`: Verifies seats are released while historical snapshots are kept.
15. `test_unauthorized_cancellation`: Prevents users from cancelling tickets they do not own.
16. `test_unauthorized_ticket_view`: Blocks unauthorized users from viewing others' tickets (HTTP 403).
17. `test_2d_array_seat_matrix_structure`: Verifies the genuine 5 × 8 nested list domain matrix.
18. `test_booking_api_flow`: Validates end-to-end REST API booking creation and seat map responses.

---

## 🔒 Security & Concurrency Considerations

- **Atomic Transactions**: Bookings and cancellations execute inside `transaction.atomic()`, ensuring consistent state even during interruptions.
- **Seat Locking**: Uses `select_for_update()` in the service layer.
- **SQLite Concurrency Limitation**: SQLite provides database-level locking rather than granular row-level locking. In a high-traffic production environment, simply configure PostgreSQL in `DATABASES`; the service layer is already architected with `select_for_update()` to support true row-level locks on PostgreSQL without code changes.
- **Server-Side Validation**: All seat codes, availability, and amounts are computed on the backend using `Decimal`. Prices submitted by client browsers are strictly ignored.
- **CSRF & XSS Protection**: All forms and Fetch API state-changing requests require valid CSRF tokens.

---

## 📄 License & Academic Note
Built as an academic full-stack project demonstrating clean architectural layering, object-oriented design patterns, and Django development best practices.
