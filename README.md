# Business Logic
Before starting the code, I want to define the business requirements and make a few assumptions about how the system will work and what features it should support. 

* The system is designed for a theme park that receives up to 10,000 visitors per day. 
* The park operates 24 hours, with around 500 visitors arriving each hour, and possibly up to 1,000 during peak times.

There are 50 different rides or activities available for visitors.
* Each ride runs continuously throughout the day and offers 48 time slots of 30 minutes each. 
* Every slot has a maximum number of people it can accommodate.
* A visitor can book a slot either for themselves or for their family, but they can't book more spots than the available capacity in a given slot.
* Also, visitors are not allowed to book multiple rides at the same time to prevent scheduling conflicts.

At the start of each day, the system will automatically generate all time slots for every ride. 

* Visitors can book a ride either by scanning a QR code at the ride location or through the web or mobile app. 
* Once a visitor books a ride and completes the confirmation and payment, they are free to enjoy other areas of the park until it’s time for their scheduled ride.
* The system will notify the visitor ahead of their ride time based on a configured reminder time, either set by default or based on the user’s app preferences. 

Visitors should also be able to change or cancel their booking, as long as they do it before a specific cutoff time. 
Lastly, each visitor will be able to view all their current and upcoming bookings easily.

# Solution

I will use following step-by-step implementation plan for virtual queue system using Java, Spring Boot, REST APIs, MVC architecture, a relational database, and scheduled notifications:

**1. Project Setup:**
Start by creating a Spring Boot project using Maven and follow the MVC pattern. Use separate packages for controllers, services, repositories, and entities. Spring Data JPA will handle database access, and the project will expose REST APIs for interaction.

**2. Database Design:**
Design a relational schema with tables for Visitors, Rides, TimeSlots, Bookings, and Notifications. Model relationships such as each ride having multiple slots and each visitor having multiple bookings. Enforce constraints for slot capacity and no overlapping bookings.

**3. Daily Slot Generation:**
Implement a scheduled job that runs at the start of each day to auto-generate 30-minute slots for every ride. Each slot is linked to a ride and has its own capacity, ensuring the system is ready to take bookings every morning.

**4. REST API for Bookings:**
Create REST endpoints for booking, canceling, and viewing rides. The booking process includes validating slot availability, checking for overlapping bookings, and updating capacity. Assume payment is completed before final confirmation.

**5. Booking Management:**
Enable visitors to manage their bookings—view current ones, cancel before a cutoff time, or reschedule. Cancellation restores slot capacity and ensures cutoff time rules are enforced.

**6. Notifications:**
Set up a background scheduler that runs periodically to send reminders for upcoming rides. Initially log-based, this can later be connected to real notification services. Track sent notifications to avoid duplicates.

**7. Web or QR Booking Flow:**
Allow users to book through a web portal or by scanning a ride-specific QR code that links to available time slots. Frontend interacts with the backend via REST APIs.


Excellent — you're doing exactly the right thing by focusing on **business requirements and assumptions** first. Let's now break down your input into clear, well-defined **functional and non-functional requirements** and **prepare for the system design** stage.

---

## Functional Requirements (FRs)

### ✅ Slot & Ride Management

* \[FR1] System auto-generates **daily ride slots** at the start of each day (48 × 30-min slots per ride).
* \[FR2] Each ride has a configurable **maximum capacity per slot**.

### ✅ Booking Logic

* \[FR3] A visitor can book **a slot for self or family** (within slot capacity).
* \[FR4] **No overlapping bookings** allowed for the same visitor.
* \[FR5] Booking is confirmed after **payment**.
* \[FR6] Slot availability is updated in **real-time** as bookings happen.

### ✅ Booking Interaction

* \[FR7] Visitors can book:

  * By **scanning a QR code** at the facility
  * Using a **web/mobile portal**
* \[FR8] Visitors can **view all their bookings**.
* \[FR9] Visitors can **cancel** or **change** a booking before a specified cutoff time (e.g., 30 mins before slot).

### ✅ Notifications

* \[FR10] System sends **notifications** (e.g., 15–30 minutes before ride time), based on booking or app preference.

---

## 🚫 Constraints and Business Rules

* \[R1] **One active booking per time slot** per visitor.
* \[R2] **No overbooking** — check and enforce capacity before confirming.
* \[R3] Visitors **cannot cancel** a booking **after the cutoff time**.
* \[R4] **System time is in park's local time zone**.

---

## 📊 Scale and Performance Considerations

| Item                 | Assumption             |
| -------------------- | ---------------------- |
| Visitors per day     | \~10,000               |
| Rides                | 50 total               |
| Slots per ride/day   | 48 (30-min each × 24h) |
| Bookings per day     | Up to 100,000+         |
| Peak booking periods | 1000+ bookings/hour    |

---

## 📌 Suggested Entities (Domain Model)

| Entity           | Attributes                                                                             |
| ---------------- | -------------------------------------------------------------------------------------- |
| **Visitor**      | id, name, email/phone, QR ID                                                           |
| **Ride**         | id, name, description, capacity per slot                                               |
| **TimeSlot**     | id, ride\_id, start\_time, capacity, booked\_count                                     |
| **Booking**      | id, visitor\_id, slot\_id, num\_guests, status (confirmed, cancelled), payment\_status |
| **Notification** | id, visitor\_id, booking\_id, time\_sent                                               |

---

## 📅 Day Start Job (Slot Generator)

* Runs daily at **midnight** or park open time
* For each ride, create 48 slots
* Each slot has `ride_id`, `start_time`, `capacity`, and `0 booked_count`

---

## 📲 User Journey Summary (Web/Mobile/QR Flow)

1. Visitor logs in or scans QR
2. Chooses a ride and sees available slots
3. Selects slot and number of guests
4. Confirms booking and completes payment
5. Receives booking confirmation
6. Receives reminder notification before ride
7. Can cancel or modify (before allowed time)
8. Shows QR code or app at ride entrance

