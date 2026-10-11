# Workflow Automation – BPMN Project Assignments

My submission for the BPMN project assignments. Both processes are modelled in **BPMN 2.0** for **Camunda 8**, with **DMN** decision tables and **Camunda forms**.

- **Assignment 2:** Logistics & Shipment Exception Management
- **Assignment 3:** Loan Origination & Approval System

---

## Assignment 2: Logistics & Shipment Exception Management

A courier picks up a parcel from a **shipper**, moves it through hubs and delivers it to a **recipient**. The model shows the normal flow and how each exception is handled.

- **Pools:** Shipper, Recipient (external; they only exchange messages)
- **Lanes:** Control Tower, Pickup Agent, Hub / Warehouse, Claims Department, Delivery Agent

![Logistics BPMN diagram](Assignment2_Logistics/diagram.png)

### Happy path
1. Booking received → **Validate booking** (DMN)
2. **Collect parcel** (pickup agent)
3. **Hand over at hub** and **send tracking number**, in parallel
4. **Transport to destination hub** (customs if international)
5. **Attempt delivery** → *Shipment delivered*

### Failure paths
| ID | Failure | How it's handled | Limit |
|---|---|---|---|
| E1 | Invalid booking | DMN says no → *Booking rejected* | – |
| E2 | Pickup fails | *Collected?* = No → *Booking cancelled* | – |
| E3 | Transit delay | Non-interrupting timer → delay notice, parcel keeps moving | 4 h |
| E4 | Parcel lost | Timer *No scan* → Claims | 48 h |
| E5 | Shipper cancels | Message boundary → Return to sender | – |
| E6 | Customs hold | Timer boundary → Return to sender | 5 days |
| E7 | Recipient not home | Count attempt → retry next day → Return to sender | 3 tries |
| E8 | Refused / COD unpaid | → Return to sender | – |
| E9 | Parcel damaged | → Claims | – |

### Forms & DMN
| File | Used by | Purpose |
|---|---|---|
| `booking-validation.dmn` | Validate booking | Rejects prohibited items, unserviceable addresses, parcels over 70 kg |
| `collect-parcel.form` | Pickup Agent | Collected or failed |
| `attempt-delivery.form` | Delivery Agent | Delivered / not home / refused / damaged |
| `settle-claim.form` | Claims | Payout amount |

**Elements used:**
- **Tasks:** business rule, user, send, receive, script, manual
- **Gateways:** exclusive, parallel
- **Events:** timer and message boundary events, timer catch

---

## Assignment 3: Loan Origination & Approval System

A bank checks a loan application, verifies the applicant, decides, makes an offer and pays out the loan. The model shows every point where the application can stall, be rejected or be cancelled.

- **Pools:** Applicant, Credit Bureau (external)
- **Lanes:** Loan Officer, Core Banking System, Fraud & Compliance, Underwriter, Operations

![Loan BPMN diagram](Assignment3_Loan/diagram.png)

### Happy path
1. Application received → **Check eligibility** (DMN)
2. **Credit score**, **KYC** and **AML screening**, in parallel
3. **Underwrite application** → approved
4. **Send offer** → signed agreement received
5. **Disburse funds** → *Loan disbursed*

### Failure paths
| ID | Failure | How it's handled | Limit |
|---|---|---|---|
| L1 | Documents missing | Request documents → wait → *Application lapsed* | 7 days |
| L2 | Not eligible | → Rejection letter | – |
| L3 | Credit bureau down | Error boundary → backup bureau | – |
| L4 | KYC failed | → Rejection letter | – |
| L5 | AML alert | Investigate → fraud = terminate end | – |
| L6 | Underwriter declines | → Rejection letter | – |
| L7 | Offer declined | Event-based gateway → *Offer declined* | – |
| L8 | No reply | Event-based gateway timer → *Offer expired* | 15 days |
| L9 | Payment fails | Error boundary → fix bank details → retry or cancel | – |
| L10 | Applicant withdraws | Event sub-process → *Withdrawn* | Any time |

### Forms & DMN
| File | Used by | Purpose |
|---|---|---|
| `loan-eligibility.dmn` | Check eligibility | Age 21–65, resident, documents complete |
| `investigate-alert.form` | Fraud & Compliance | Fraud or false alarm |
| `underwrite-application.form` | Underwriter | Approve or decline |
| `fix-disbursement.form` | Operations | Retry payment or cancel |

**Elements used:**
- **Tasks:** business rule, user, service, send, receive
- **Gateways:** exclusive, parallel, event-based
- **Events:** timer and error boundary events, message and timer catch, terminate end, event sub-process
