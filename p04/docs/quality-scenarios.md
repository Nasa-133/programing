# Quality scenarios

## Stakeholders

| Who | What they care about |
|---|---|
| Guest | Finds a free room quickly; never loses a booking |
| Receptionist | Never promises one room twice; can cancel at the desk |
| Hotel manager (domain expert) | Knows the rules; wants deposits and SMS soon |
| Development team | Can change one area without breaking the others |
| Email/SMS and payment providers | Outside systems that can be slow or down |

## S3 · Availability: a failed message keeps the booking

| Part | S3 |
|---|---|
| Source | The email provider |
| Stimulus | It stops answering |
| Environment | For 10 minutes, during normal bookings |
| Artifact | BookingService.reserve and the notification sender |
| Response | Every valid booking is still Accepted and saved; the failed message is reported |
| Measure | 0 lost bookings; every failed message is counted (100%) |
| Checked in | P04 (R6 test, done) and P10 |

## S1 · Performance

| Part | S1 |
|---|---|
| Source | 200 guests |
| Stimulus | They search for free rooms for 12–14 November |
| Environment | Within one minute |
| Artifact | The free-room search in BookingService.find_free_rooms |
| Response | Return the correct free rooms within the response-time target |
| Measure | p95 response time ≤ 1 second; 100% of results are correct |
| Checked in | P08 (cache), P23 (load test) |

## S2 · Data integrity

| Part | S2 |
|---|---|
| Source | 100 pairs of guests |
| Stimulus | Both guests in each pair try to book the same room for the same nights at the same moment |
| Environment | Concurrent booking requests; the room is initially free for each pair |
| Artifact | BookingService.reserve and the reservation store |
| Response | Accept and save exactly one Confirmed reservation per pair; refuse the conflicting request |
| Measure | Exactly 1 Confirmed reservation per pair across 100 pairs; 0 double bookings |
| Checked in | P09 |

## S4 · Security

| Part | S4 |
|---|---|
| Source | A guest |
| Stimulus | They try 100 times to read another guest's reservation |
| Environment | During normal use, while accessing a reservation belonging to another guest |
| Artifact | Reservation lookup and its authorization check |
| Response | Deny access without revealing another guest's reservation |
| Measure | 0 reservations shown to the wrong guest across 100 attempts |
| Checked in | P21 |

## S5 · Modifiability

| Part | S5 |
|---|---|
| Source | The hotel manager |
| Stimulus | The hotel requests SMS confirmations |
| Environment | During development, with the existing notification interface in place |
| Artifact | The notification sender implementations and application wiring |
| Response | Add and select an SMS sender through the existing interface while keeping BookingService unchanged |
| Measure | 0 edits to BookingService; at most 2 files change |
| Checked in | P04 (SMS sender, done) |

## S6 · Sustainability

| Part | S6 |
|---|---|
| Source | A search workload of 1,000 requests |
| Stimulus | The same searches are run before and after a change |
| Environment | Comparable hardware, data and load conditions for both runs |
| Artifact | The free-room search in BookingService.find_free_rooms and its supporting infrastructure |
| Response | Complete the same searches with correct results while consuming less energy |
| Measure | At least 20% fewer watt-hours per 1,000 completed searches compared with the baseline |
| Checked in | P08, P24 |
