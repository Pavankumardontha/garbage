## Aggregate roots — one door per family

The root is the only object outside code may hold or call; everything inside
is reached through it. Across families, only ids travel.

| Aggregate | Root | Inside the family | Points at others by id |
| --- | --- | --- | --- |
| Event | Event | its PriceTiers | organizerId, venueId, seatMapId |
| SeatMap | SeatMap | its sections and seat labels | venueId |
| SeatInventoryBlock | SeatInventoryBlock | its capacity/allocated counts | eventId |
| SeatHold | SeatHold | the list of held seats | eventId, customerId |
| Order | Order | its OrderLines | customerId; eventId and tierCode per line |
| Payment | Payment | its capture/refund entries | orderId |
| Ticket | Ticket | (single object) | orderId, eventId |

## Cross-aggregate rules — eventual consistency, saga in Phase 6

Rules whose facts live in different aggregates. No single commit can guarantee
them, so the other side catches up shortly after; Phase 6 builds the saga (a
sequence of local transactions with compensating undo steps) that coordinates
them.

1. An order's total matches the sum of the current tier prices (Order +
   Event). A price may change mid-checkout; checkout re-verifies, and the gap
   is chased, never assumed away.
2. A confirmed order always has a captured payment (Order + Payment). For a
   few seconds after the charge they may disagree; on failure the undo step
   fails the order shortly after.
3. The browse page's "seats remaining" equals the true count (Catalog's copy +
   SeatInventoryBlock). Allowed to lag by design; corrected on the next
   announcement.
