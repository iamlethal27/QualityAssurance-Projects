# 🐞 Bug Reports - Fintech Payment Gateway Testing

**Project:** Payment Gateway Testing  
**Tester:** Uzair Akhtar  
**Tool:** JIRA (simulated format)  
**Total Bugs Found:** 3  
**Critical:** 3 | **High:** 0 | **Medium:** 0

---

## BUG-001 - Full Card Number Exposed in API Response

| Field | Detail |
|-------|--------|
| **Bug ID** | BUG-001 |
| **Title** | Full card PAN number returned unmasked in transaction API response body |
| **Module** | Card Validation / Security |
| **Severity** | 🔴 Critical |
| **Priority** | 🔴 High |
| **Status** | Open |
| **Environment** | Test Environment v1.0 |
| **Reported By** | Uzair Akhtar |

**Steps to Reproduce:**
1. Initiate a payment with a valid card (card number: 4111111111111111)
2. Submit the transaction
3. Inspect the API response body in Postman or Browser DevTools

**Expected Result:**
Card number should be masked in the response - only the last 4 digits visible (e.g. `****1111`). The full 16-digit PAN should never appear in any API response.

**Actual Result:**
The full card number `4111111111111111` is returned in plain text inside the response JSON under the field `card_number`.

**Why This is Critical:**
In Fintech and card processing, exposing the full Primary Account Number (PAN) in API responses is a serious PCI-DSS compliance violation. If this response is logged or intercepted, cardholder data is compromised. This must be fixed before any production release.

**Attachment:** screenshot-bug-001-pan-exposed.png

---

## BUG-002 - Duplicate Transaction Not Blocked Within 60-Second Window

| Field | Detail |
|-------|--------|
| **Bug ID** | BUG-002 |
| **Title** | Identical transaction submitted twice within 60 seconds - both are approved and customer is charged twice |
| **Module** | Duplicate Transaction Detection |
| **Severity** | 🔴 Critical |
| **Priority** | 🔴 High |
| **Status** | Open |
| **Environment** | Test Environment v1.0 |
| **Reported By** | Uzair Akhtar |

**Steps to Reproduce:**
1. Log in with a valid account (balance: PKR 5000)
2. Submit a payment of PKR 500 to Merchant ID: M-001
3. Within 10 seconds, submit the exact same payment (PKR 500 to M-001)
4. Check account balance and transaction history

**Expected Result:**
The second transaction should be rejected with the error message: "Duplicate transaction detected. Please wait before retrying." Only one deduction of PKR 500 should occur. Account balance should be PKR 4500.

**Actual Result:**
Both transactions are approved. Account balance shows PKR 4000 - customer has been charged twice. Transaction history shows two separate approved entries for the identical transaction.

**Why This is Critical:**
Double-charging a customer is a direct financial harm. In payment systems, duplicate detection is a fundamental safeguard. This bug would erode customer trust, generate chargebacks, and expose the business to financial liability.

**Attachment:** screenshot-bug-002-duplicate-charge.png

---

## BUG-003 - Card PAN Returned Unmasked in Refund Confirmation Response

| Field | Detail |
|-------|--------|
| **Bug ID** | BUG-003 |
| **Title** | Full card number exposed in refund confirmation API response - separate from BUG-001 |
| **Module** | Refund & Reversal / Security |
| **Severity** | 🔴 Critical |
| **Priority** | 🔴 High |
| **Status** | Open |
| **Environment** | Test Environment v1.0 |
| **Reported By** | Uzair Akhtar |

**Steps to Reproduce:**
1. Complete a successful transaction
2. Initiate a full refund on that transaction
3. Inspect the refund confirmation API response

**Expected Result:**
Refund response should contain only masked card data (last 4 digits). Full PAN must not appear.

**Actual Result:**
The refund confirmation response contains the full unmasked card number in the `refund_to_card` field - separate from the issue in BUG-001 which occurs in the initial transaction response.

**Why This is Critical:**
This is a second independent occurrence of PAN exposure. Even if BUG-001 is fixed in the transaction flow, this bug would still expose card data through the refund flow. Both must be patched independently. This is a PCI-DSS violation.

**Note for Developer:**
Please ensure the masking logic is applied globally - not just in the initial transaction response, but in all responses that reference card data including refunds, reversals, transaction history, and receipts.

**Attachment:** screenshot-bug-003-pan-refund.png
