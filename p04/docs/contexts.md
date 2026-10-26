# Hotel contexts

## Words

| Word | Meaning here | Name in our code |
|---|---|---|
| Stay | Check-in to check-out, half-open: 12–14 Nov = the nights of the 12th and 13th | `DateRange` (`check_in`, `check_out`) |
| Reservation | Guest + room + stay + status, with an id such as R101 | `Reservation` (`id`, `guest`, `room`, `stay`, `status`) |
| Confirmed | The room is promised to this guest for these nights | `ReservationStatus.CONFIRMED` |
| Guest | A person booking a room, such as Maya, identified here by name and email | `Guest` |
| Room | A room the hotel sells, such as R8 | `Room` (`number`, `type`) |
| Room type | A room category, such as Standard or Deluxe | `Room.type` |
| Nightly price | The price of one night for a room, depending on its type | not in code yet |
| Out of service | A room is temporarily unavailable for sale, for example during repairs | not in code yet |
| Cancelled | The promise is withdrawn; the nights are free again | `ReservationStatus.CANCELLED` |
| Book | Ask for a room for a stay and create a Confirmed reservation if accepted | `BookingService.reserve(guest, room, stay)` |
| Cancel | Withdraw a reservation, such as R101, so its nights become free again | `BookingService.cancel(reservation_id)` |
| Free rooms | Rooms with no Confirmed reservation sharing any night of the requested stay | `BookingService.find_free_rooms(stay)` |
| Deposit | 30% of the stay's price, taken when the guest books | not in code yet |
| Paid | A deposit payment has succeeded | not in code yet |
| Declined | A deposit payment has been refused | not in code yet |
| Confirmation message | An email or SMS telling the guest that their reservation is confirmed | `NotificationSender.send_confirmation(reservation)` |
| Sent | A message has been successfully sent through its channel | not in code yet |
| Failed | A message could not be sent, for example because the provider is down | not in code yet |

## Rooms

Job: knows which rooms the hotel sells.

| Word | Meaning here |
|---|---|
| Room | A room the hotel sells, such as R8 |
| Room type | A room category, such as Standard or Deluxe |
| Nightly price | The price of one night for a room, depending on its type |
| Out of service | A room is temporarily unavailable for sale, for example during repairs |

Rules this context owns:

- Card 4: A room that is out of service is not offered to guests.
- Card 5: The nightly price depends on the room type (Deluxe costs more than Standard).

## Reservations

Job: decides whether a guest gets a room for some nights, and keeps every reservation.

| Word | Meaning here |
|---|---|
| Stay | Check-in to check-out, half-open: 12–14 Nov = the nights of the 12th and 13th |
| Reservation | Guest + room number + stay + status, with an id such as R101 |
| Confirmed | The room is promised to this guest for these nights |
| Cancelled | The promise is withdrawn; the nights are free again |
| Guest | A person booking a room, such as Maya, identified here by name and email |
| Book | Ask for a room for a stay and create a Confirmed reservation if accepted |
| Cancel | Withdraw a reservation, such as R101, so its nights become free again |
| Free rooms | Rooms with no Confirmed reservation sharing any night of the requested stay |

Rules this context owns:

- Card 1: A room never has two Confirmed reservations that share a night.
- Card 2: Check-out must be after check-in.
- Card 3: Cancelling a reservation frees its nights for new bookings.

## Payments

Job: takes deposits and records whether they were paid.

| Word | Meaning here |
|---|---|
| Deposit | 30% of the stay's price, taken when the guest books |
| Paid | A deposit payment has succeeded |
| Declined | A deposit payment has been refused |

Rules this context owns:

- Card 6: The deposit is 30% of the stay's price.
- Card 7: Only a Paid deposit can be refunded.

## Notifications

Job: tells guests what happened, by email or SMS.

| Word | Meaning here |
|---|---|
| Confirmation message | An email or SMS telling the guest that their reservation is confirmed |
| Sent | A message has been successfully sent through its channel |
| Failed | A message could not be sent, for example because the provider is down |

Rules this context owns:

- Card 8: A failed message is recorded as Failed and never changes the reservation.

## Same word, different meaning

| Word | Rooms | Reservations | Payments | Notifications |
|---|---|---|---|---|
| Status | In service / out of service | Confirmed / Cancelled | Paid / Declined | Sent / Failed |
| Room | The full room record: R8, Deluxe, nightly price and service availability | The room number R8 attached to a reservation | Not used | Room R8 written in the guest's message |
| Cancel | Stop offering a room for sale | Withdraw the guest's reservation and release its nights | Refund a deposit | Not used |

If Notifications could change Maya's reservation R101 from Confirmed to Failed after an SMS failure, Maya would lose R7, breaking R6: a failed confirmation must leave the booking Accepted and saved.

## Rule owners

| Card | Rule | Owner | Why this owner |
|---|---|---|---|
| 1 | A room never has two Confirmed reservations that share a night. | Reservations | It keeps every reservation of the room, so it can check their stays and statuses together. |
| 2 | Check-out must be after check-in. | Reservations | It keeps the stay's check-in and check-out dates and validates them before accepting a booking. |
| 3 | Cancelling a reservation frees its nights for new bookings. | Reservations | It keeps the reservation's stay and status, so it can mark it Cancelled and stop blocking those nights. |
| 4 | A room that is out of service is not offered to guests. | Rooms | It keeps each room's service availability, so it decides which rooms can be offered. |
| 5 | The nightly price depends on the room type (Deluxe costs more than Standard). | Rooms | It keeps room types and their nightly prices, so it owns the type-to-price rule. |
| 6 | The deposit is 30% of the stay's price. | Payments | It owns deposit amounts and the 30% calculation, using a nightly price from Rooms and the stay length supplied by Reservations. |
| 7 | Only a Paid deposit can be refunded. | Payments | It keeps each deposit's payment status and can allow a refund only when that status is Paid. |
| 8 | A failed message is recorded as Failed and never changes the reservation. | Notifications | It keeps message delivery outcomes and records its own Failed status without changing data owned by Reservations. |

## Context map

The **Context map** page in `diagrams.drawio` shows four contexts and three arrows, all starting at Reservations:

- Reservations → Rooms: **Asks: which rooms exist?** (solid; waits for an answer).
- Reservations → Payments: **Asks: take a deposit (P17)** (solid; waits for an answer).
- Reservations → Notifications: **Announces: ReservationConfirmed** (green dotted; does not wait).

Reservations is green because it owns the booking rule. Payments and message delivery statuses describe the planned context models; they are not implemented in P04 yet.
