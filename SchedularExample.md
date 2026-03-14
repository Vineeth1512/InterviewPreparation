Vineeth, let’s explain the **scheduler scenario step-by-step with a simple real example** so you clearly understand **how the appointment gets cancelled automatically**.

---

# 🏥 Scenario: Doctor Appointment System

Doctor schedule:

```
Doctor: Dr. Rao
Available: 10:00 AM – 2:00 PM
```

Patient books appointment:

```
Patient: Vineeth
Appointment Time: 11:30 AM
```

System checks:

```
Is 11:30 between 10 and 2?
YES → Appointment booked
```

Database record:

| id | patientId | doctorId | appointment_time | status |
| -- | --------- | -------- | ---------------- | ------ |
| 1  | 101       | 1        | 11:30            | BOOKED |

---

# ⏱ Current Time Example

Suppose current time becomes:

```
12:10 PM
```

Now the system checks:

```
appointment_time < current_time
```

```
11:30 < 12:10
```

Condition is **TRUE**.

Meaning:

```
Patient missed the appointment.
```

So the system updates:

| id | patientId | doctorId | appointment_time | status    |
| -- | --------- | -------- | ---------------- | --------- |
| 1  | 101       | 1        | 11:30            | CANCELLED |

---

# ⚙ How the Scheduler Does This Automatically

Your scheduler runs every few minutes.

Example:

```java
@Scheduled(fixedRate = 300000) // every 5 minutes
```

Execution timeline:

```
11:35 → scheduler runs
11:40 → scheduler runs
11:45 → scheduler runs
12:00 → scheduler runs
12:05 → scheduler runs
```

Each time it runs, it checks the database.

---

# 🔎 What the Scheduler Checks

Scheduler query:

```
Find appointments where
status = BOOKED
AND appointment_time < current_time
```

Example data in DB:

| id | patient | appointment_time | status |
| -- | ------- | ---------------- | ------ |
| 1  | Vineeth | 11:30            | BOOKED |
| 2  | Rahul   | 12:45            | BOOKED |

Current time:

```
12:10
```

Scheduler result:

```
11:30 < 12:10 → CANCELLED
12:45 > 12:10 → still BOOKED
```

Updated table:

| id | patient | appointment_time | status    |
| -- | ------- | ---------------- | --------- |
| 1  | Vineeth | 11:30            | CANCELLED |
| 2  | Rahul   | 12:45            | BOOKED    |

---

# 🔁 Automatic Process Flow

```
Patient books appointment
        │
        ▼
System saves appointment
status = BOOKED
        │
        ▼
Scheduler runs automatically
(every few minutes)
        │
        ▼
Check appointment_time < current_time
        │
        ▼
Update status → CANCELLED
```

---

# 🧠 Real-Life Analogy

Think about **restaurant reservation**.

```
Reservation Time: 7:00 PM
```

Restaurant rule:

```
If customer doesn't arrive within 15 minutes
→ reservation cancelled
```

No human cancels it manually.
The **system automatically releases the table**.

That automatic check is exactly what **scheduler does**.

---

# 💻 Actual Scheduler Code

Example:

```java
@Scheduled(fixedRate = 300000)
public void cancelMissedAppointments() {

    LocalDateTime now = LocalDateTime.now();

    List<Appointment> missed =
            appointmentRepository
            .findByStatusAndAppointmentTimeBefore("BOOKED", now);

    for (Appointment appointment : missed) {
        appointment.setStatus("CANCELLED");
    }

    appointmentRepository.saveAll(missed);
}
```

Meaning:

```
Get appointments
where appointment_time < current_time
and status = BOOKED
```

Then:

```
status → CANCELLED
```

---

