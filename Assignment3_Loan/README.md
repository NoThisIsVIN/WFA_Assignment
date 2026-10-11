# Assignment 3: Loan Origination & Approval System

A bank receives a loan application, checks the applicant, decides, makes an offer and pays out the loan. The model shows the normal flow and every point where the application can stop.

## Files
| File | What it is |
|---|---|
| `loan_origination.bpmn` | The process diagram (Camunda 8) |
| `loan-eligibility.dmn` | Rules: eligible, ineligible, or documents missing |
| `investigate-alert.form` | Compliance: fraud or false alarm |
| `underwrite-application.form` | Underwriter: approve or decline |
| `fix-disbursement.form` | Operations: retry payment or cancel |
| `Loan_Presentation.pptx` | Presentation slides |

## Participants
- **Pools (external):** Applicant, Credit Bureau
- **Lanes (internal):** Loan Officer, Core Banking System, Fraud & Compliance, Underwriter, Operations

## Happy path
1. Application received
2. Check eligibility (DMN)
3. **In parallel:** get credit score, verify identity (KYC), AML screening
4. Underwriter approves
5. Send offer, then the signed agreement is received
6. Disburse funds and send confirmation, which ends with **Loan disbursed**

## Failure paths
| # | What goes wrong | How the model handles it |
|---|---|---|
| 1 | Documents missing | Request documents and wait 7 days → *Application lapsed* if none arrive |
| 2 | Not eligible | → Rejection letter |
| 3 | Credit bureau down | Error boundary → use the backup bureau |
| 4 | KYC failed | → Rejection letter |
| 5 | AML alert | Compliance investigates → fraud ends everything (**terminate end**) |
| 6 | Underwriter declines | → Rejection letter |
| 7 | Applicant declines the offer | → *Offer declined* |
| 8 | No response in 15 days | Event-based gateway timer → *Offer expired* |
| 9 | Payment fails | Error boundary → fix bank details → retry or cancel |
| 10 | Applicant withdraws at any time | Event sub-process → withdrawal confirmed |

## BPMN elements used
- **Gateways:** exclusive, parallel (the three checks), event-based (signed / declined / timeout)
- **Events:**
  - message start
  - timer and error boundaries
  - terminate end
  - an interrupting event sub-process (withdrawal)
- **Tasks:** business rule (DMN), user (forms), service (system calls), send and receive (messages)
