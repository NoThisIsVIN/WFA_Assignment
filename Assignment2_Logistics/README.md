# Assignment 2: Logistics & Shipment Exception Management

A courier picks up a parcel from a **shipper**, moves it through hubs and delivers it to a **recipient**. The model shows the normal flow and what happens when something goes wrong.

## Files
| File | What it is |
|---|---|
| `shipment_exception_management.bpmn` | The process diagram (Camunda 8) |
| `booking-validation.dmn` | Rules: is the booking valid? |
| `collect-parcel.form` | Pickup agent: collected or failed |
| `attempt-delivery.form` | Delivery agent: delivered / not home / refused / damaged |
| `settle-claim.form` | Claims: payout amount |
| `Logistics_Presentation.pptx` | Presentation slides |

## Participants
- **Pools (external):** Shipper, Recipient
- **Lanes (internal):** Control Tower, Pickup Agent, Hub / Warehouse, Claims Department, Delivery Agent

## Happy path
1. Booking received from the shipper
2. Validate booking (DMN)
3. Collect parcel
4. Hand over at hub, and **in parallel** send the tracking number
5. Transport to destination hub (customs if international)
6. Attempt delivery, which ends with **Shipment delivered**

## Failure paths
| # | What goes wrong | How the model handles it |
|---|---|---|
| 1 | Invalid booking | DMN says no → *Booking rejected* |
| 2 | Pickup fails | → *Booking cancelled* |
| 3 | Delivery delayed | Non-interrupting timer → send the shipper a new ETA (parcel keeps moving) |
| 4 | Parcel lost (no scan for 48 h) | Timer boundary → **Claims** |
| 5 | Shipper cancels in transit | Message boundary → **Return to sender** |
| 6 | Customs hold over 5 days | Timer boundary → **Return to sender** |
| 7 | Recipient not home | Retry next day, **max 3 attempts** → **Return to sender** |
| 8 | Refused / COD not paid | → **Return to sender** |
| 9 | Parcel damaged | → **Claims** |

## BPMN elements used
- **Gateways:** exclusive (decisions), parallel (hand-over and tracking at the same time)
- **Events:** message start, timer boundaries (interrupting and non-interrupting), message boundary, timer catch
- **Tasks:**
  - business rule (DMN)
  - user (forms)
  - send and receive (messages)
  - script (attempt counter)
  - manual (physical work)
