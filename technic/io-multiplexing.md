# I/O Multiplexing Explained

I/O multiplexing represents the third of five fundamental I/O models in system programming. The following narrative illustrates the key distinctions between different I/O paradigms:

## Scenario Setup

**Context**: Mr. Li needs to purchase a train ticket, specifically a refund ticket that becomes available after three days
**Characters**: Mr. Li (client), Ticket Scalper (intermediary), Ticket Clerk (server), Courier (delivery service)
**Time Cost**: Round trip to station takes 1 hour

## 1. Blocking I/O Model

Mr. Li goes to the train station to buy a ticket and waits in line for three days to purchase a refund ticket.

**Cost**: Spends 3 days at the station for eating, drinking, sleeping, and other necessities, accomplishing nothing else.

---

## 2. Non-blocking I/O Model

Mr. Li visits the train station periodically every 12 hours to inquire about refund tickets, eventually purchasing one after three days.

**Cost**: Makes 6 round trips to the station, spending 6 hours traveling, but manages to accomplish other tasks during waiting periods.

---

## 3. I/O Multiplexing Model

### 3.1 select/poll

Mr. Li delegates the ticket purchase to a scalper, then calls every 6 hours to check status. The scalper acquires the ticket within three days, after which Mr. Li goes to the station to pay and collect the ticket.

**Cost**: 2 round trips to station (2 hours travel time), 100 yuan commission fee, 17 phone calls

### 3.2 epoll

Mr. Li delegates the ticket purchase to a scalper, who notifies Mr. Li immediately upon acquisition. Mr. Li then proceeds to the station to complete the transaction.

**Cost**: 2 round trips to station (2 hours travel time), 100 yuan commission fee, no phone calls required

---

## 4. Signal-driven I/O Model

Mr. Li provides his phone number to the ticket clerk at the station. When a ticket becomes available, the clerk calls Mr. Li directly, who then proceeds to the station to complete the purchase.

**Cost**: 2 round trips to station (2 hours travel time), no scalper fee (100 yuan saved), no proactive calls required

---

## 5. Asynchronous I/O Model

Mr. Li provides contact information to the ticket clerk. Upon ticket availability, the clerk notifies Mr. Li and arranges courier delivery to his location.

**Cost**: 1 round trip to station (1 hour travel time), no scalper fee (100 yuan saved), no proactive calls required

---

## Model Comparison Summary

| Model Type | Key Distinction |
|------------|----------------|
| Blocking I/O vs Non-blocking I/O | Self-polling approach |
| Non-blocking I/O vs I/O Multiplexing | Delegation to intermediary |
| I/O Multiplexing vs Signal-driven I/O | Phone calls replace intermediaries |
| Signal-driven I/O vs Asynchronous I/O | Notification method (pickup vs delivery) |