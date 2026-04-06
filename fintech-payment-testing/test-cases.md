# 📋 Test Cases - Fintech Payment Gateway

**Project:** Payment Gateway Testing  
**Tester:** Uzair Akhtar  
**Environment:** Hypothetical Payment System (Practice-Based)  
**Total Test Cases:** 50  
**Last Updated:** 2025

---

## Module 1 - Successful Transaction (Happy Path)

| TC ID | Title | Precondition | Steps | Expected Result | Severity | Result |
|-------|-------|-------------|-------|-----------------|----------|--------|
| PG-001 | Successful card payment with valid Visa card | Account has sufficient balance, card is active | 1. Enter valid card number 2. Enter correct expiry 3. Enter correct CVV 4. Enter amount within limit 5. Submit | Transaction approved. Response code 00. Balance deducted. Confirmation message shown. | Critical | ✅ Pass |
| PG-002 | Successful card payment with valid Mastercard | Account has sufficient balance, card is active | Same steps as PG-001 with Mastercard number | Transaction approved. Same expected result as PG-001. | Critical | ✅ Pass |
| PG-003 | Verify correct transaction amount is deducted | Account has PKR 5000 balance | 1. Make payment of PKR 1000 2. Check balance after | Balance should show exactly PKR 4000 after transaction | Critical | ✅ Pass |
| PG-004 | Verify transaction ID is generated | Any valid payment | Complete a successful transaction | A unique transaction ID is generated and displayed in the response | High | ✅ Pass |
| PG-005 | Verify transaction appears in history | Any valid payment | Complete transaction → open transaction history | Transaction appears with correct amount, date, merchant name, and status "Approved" | High | ✅ Pass |
| PG-006 | Successful transaction with minimum allowed amount | Card active | Submit transaction with minimum amount (e.g. PKR 1) | Transaction processes successfully without error | Medium | ✅ Pass |
| PG-007 | Successful transaction with maximum allowed amount | Card active, sufficient balance | Submit transaction at daily limit (e.g. PKR 100,000) | Transaction approved without error | High | ✅ Pass |
| PG-008 | Response time for successful transaction | N/A | Complete a transaction and record response time | Response received within 10 seconds | Medium | ✅ Pass |

---

## Module 2 - Card Validation

| TC ID | Title | Precondition | Steps | Expected Result | Severity | Result |
|-------|-------|-------------|-------|-----------------|----------|--------|
| PG-009 | Payment with invalid card number (wrong length) | N/A | Enter a 15-digit card number instead of 16 | System rejects input before submission. Error: "Invalid card number." | High | ✅ Pass |
| PG-010 | Payment with invalid card number (letters in field) | N/A | Enter letters (e.g. "ABCD1234EFGH5678") | System rejects non-numeric input. Error message shown. | High | ✅ Pass |
| PG-011 | Payment with expired card | Card expiry date in the past | Enter a card with expiry 01/22 | Transaction declined. Response code 54. Error: "Card expired." | Critical | ✅ Pass |
| PG-012 | Payment with wrong CVV | Valid card number, valid expiry | Enter correct card but wrong 3-digit CVV | Transaction declined. Error: "Invalid CVV." | Critical | ✅ Pass |
| PG-013 | Payment with wrong expiry date | Valid card number, valid CVV | Enter card number and CVV correctly but wrong expiry year | Transaction declined. Error: "Card details do not match." | Critical | ✅ Pass |
| PG-014 | Card blocked after 3 wrong CVV attempts | Valid card | Enter wrong CVV 3 times consecutively | After third attempt: card is blocked. Message: "Card blocked. Contact your bank." | Critical | ✅ Pass |
| PG-015 | Payment with empty card number field | N/A | Leave card number field blank and submit | Form validation error: "Card number is required." Transaction not submitted. | High | ✅ Pass |
| PG-016 | Payment with empty CVV field | N/A | Leave CVV field blank and submit | Form validation error: "CVV is required." | High | ✅ Pass |
| PG-017 | Payment with unsupported card type | N/A | Enter an American Express card number | Error: "This card type is not supported." Transaction not processed. | Medium | ✅ Pass |
| PG-018 | Card number with spaces | N/A | Enter card number with spaces between groups (e.g. "4111 1111 1111 1111") | System accepts formatted input OR shows clear error - must not crash | Medium | ✅ Pass |
| PG-019 | Payment with all zeros card number | N/A | Enter "0000000000000000" as card number | Rejected as invalid. No transaction initiated. | High | ✅ Pass |
| PG-020 | Verify card number is masked in response | Any transaction | Submit transaction and inspect API response payload | Card number should show only last 4 digits (e.g. ****1111). Full number never exposed. | Critical | ❌ Fail |

---

## Module 3 - Insufficient Funds

| TC ID | Title | Precondition | Steps | Expected Result | Severity | Result |
|-------|-------|-------------|-------|-----------------|----------|--------|
| PG-021 | Payment amount exceeds account balance | Account balance is PKR 500 | Submit transaction for PKR 1000 | Declined. Response code 51. Error: "Insufficient funds." No amount deducted. | Critical | ✅ Pass |
| PG-022 | Verify balance is NOT deducted on decline | Account balance is PKR 500 | Submit transaction for PKR 2000 → check balance | Balance remains exactly PKR 500 after failed transaction | Critical | ✅ Pass |
| PG-023 | Payment exactly equal to account balance | Account balance is PKR 1000 | Submit transaction for exactly PKR 1000 | Transaction approved. Balance becomes PKR 0. | High | ✅ Pass |
| PG-024 | Payment one rupee over account balance | Account balance is PKR 999 | Submit transaction for PKR 1000 | Declined. Insufficient funds. Balance unchanged. | High | ✅ Pass |
| PG-025 | Error message is clear and user-friendly | Any declined transaction | Trigger an insufficient funds decline | Error message explains the reason clearly. Does not expose internal system details or error codes to the user. | Medium | ✅ Pass |

---

## Module 4 - Duplicate Transaction Detection

| TC ID | Title | Precondition | Steps | Expected Result | Severity | Result |
|-------|-------|-------------|-------|-----------------|----------|--------|
| PG-026 | Submit identical transaction twice within 60 seconds | Valid card, sufficient balance | 1. Submit transaction PKR 500 to Merchant A 2. Immediately submit exact same transaction | Second transaction is rejected with error: "Duplicate transaction detected." Only one deduction made. | Critical | ❌ Fail |
| PG-027 | Same amount to different merchant is not flagged | Valid card | Submit PKR 500 to Merchant A then PKR 500 to Merchant B | Both transactions approved. Different merchants means not a duplicate. | High | ✅ Pass |
| PG-028 | Same transaction after 5 minutes is allowed | Valid card | Submit transaction → wait 5 minutes → submit same transaction again | Second transaction is approved. Time window has expired. | Medium | ✅ Pass |
| PG-029 | Duplicate detection window is 60 seconds | Valid card | Submit transaction → wait 61 seconds → submit same transaction | Second transaction is approved (just outside the window) | Medium | ✅ Pass |

---

## Module 5 - Timeout and Network Failure

| TC ID | Title | Precondition | Steps | Expected Result | Severity | Result |
|-------|-------|-------------|-------|-----------------|----------|--------|
| PG-030 | Transaction in progress - network drops | Transaction submitted | Simulate network disconnect mid-transaction | System handles gracefully. Either transaction completes and customer is notified, OR transaction is rolled back - funds are NOT deducted if transaction did not complete | Critical | ✅ Pass |
| PG-031 | No double deduction after network timeout | Account has PKR 5000 | Submit transaction → cause timeout → check balance | If transaction timed out, balance should remain unchanged. | Critical | ✅ Pass |
| PG-032 | User sees clear message on timeout | Any timeout scenario | Trigger a timeout | User sees message such as "Transaction could not be completed. Please try again." Not a blank screen or crash. | High | ✅ Pass |
| PG-033 | System recovers after timeout and accepts new transaction | After a timeout | Trigger timeout → wait → attempt new valid transaction | New transaction processes successfully. System is not stuck. | High | ✅ Pass |
| PG-034 | Timeout response is returned within defined SLA | Any transaction | Simulate a slow/unresponsive downstream system | System returns a timeout response within 30 seconds. Does not hang indefinitely. | Medium | ✅ Pass |
| PG-035 | Transaction status is clear after timeout | Any timeout | Trigger timeout → open transaction history | Transaction status shows "Pending" or "Failed" - not stuck as "Processing" | High | ✅ Pass |

---

## Module 6 - Refund and Reversal Flow

| TC ID | Title | Precondition | Steps | Expected Result | Severity | Result |
|-------|-------|-------------|-------|-----------------|----------|--------|
| PG-036 | Full refund for a completed transaction | Transaction approved and settled | Initiate a full refund on a settled transaction | Refund initiated. Transaction status changes to "Refunded". Customer balance restored within SLA window. | Critical | ✅ Pass |
| PG-037 | Partial refund for a completed transaction | Transaction approved | Initiate refund for half the transaction amount | Partial refund processed. Correct amount credited back. Remaining amount stays with merchant. | High | ✅ Pass |
| PG-038 | Refund cannot exceed original transaction amount | Transaction for PKR 1000 | Attempt refund of PKR 1500 | Refund rejected. Error: "Refund amount cannot exceed original transaction." | High | ✅ Pass |
| PG-039 | Refund of an already refunded transaction | Transaction already fully refunded | Attempt to refund it again | Rejected. Error: "Transaction already refunded." | Critical | ✅ Pass |
| PG-040 | Refund of a declined transaction | Transaction that was declined | Attempt to initiate refund | Rejected. Error: "Cannot refund a declined transaction." | High | ✅ Pass |
| PG-041 | Transaction reversal within same day | Transaction made today | Initiate a reversal (void) within the same business day | Reversal accepted. Transaction cancelled. No settlement occurs. Balance fully restored immediately. | Critical | ✅ Pass |
| PG-042 | Reversal after settlement is not allowed | Transaction settled yesterday | Attempt a void/reversal | System rejects void. Advises to initiate a refund instead. | High | ✅ Pass |
| PG-043 | Refund status appears in transaction history | After a refund | Open transaction history | Original transaction shows "Refunded" status. Refund amount and date are shown separately. | Medium | ✅ Pass |

---

## Module 7 - Security Testing (Basic)

| TC ID | Title | Precondition | Steps | Expected Result | Severity | Result |
|-------|-------|-------------|-------|-----------------|----------|--------|
| PG-044 | Card number is masked in API response | Any transaction | Submit transaction and inspect JSON response body | Card number shows only last 4 digits (e.g. ****5678). Full PAN never returned in any response. | Critical | ❌ Fail |
| PG-045 | CVV is never stored or returned | Any transaction | Submit transaction and inspect all response payloads | CVV field is absent from all responses. It must never be returned or stored after authorization. | Critical | ✅ Pass |
| PG-046 | SQL injection in card number field | N/A | Enter: ' OR '1'='1 in card number field | Input is rejected. System does not crash. No database error exposed in response. | Critical | ✅ Pass |
| PG-047 | SQL injection in amount field | N/A | Enter: 1; DROP TABLE transactions in amount field | Input rejected. No database manipulation occurs. | Critical | ✅ Pass |
| PG-048 | Error messages do not expose system internals | Trigger any error | Cause various errors (wrong card, timeout, etc.) | Error messages are user-friendly. No stack traces, database errors, or internal code exposed to the user. | High | ✅ Pass |
| PG-049 | Session expires after inactivity | Active session | Leave the payment page idle for the defined timeout period | Session expires. User is prompted to re-authenticate before submitting payment. | High | ✅ Pass |
| PG-050 | HTTPS is enforced for all payment endpoints | N/A | Attempt to access payment endpoint via HTTP (not HTTPS) | Request is redirected to HTTPS or rejected. No payment data is ever transmitted over unencrypted HTTP. | Critical | ✅ Pass |

---

## 🐞 Bug Summary

| Bug ID | TC | Title | Severity | Status |
|--------|-----|-------|----------|--------|
| BUG-001 | PG-020 | Full card number exposed in API response body | Critical | Open |
| BUG-002 | PG-026 | Duplicate transaction within 60 seconds is NOT blocked - customer charged twice | Critical | Open |
| BUG-003 | PG-044 | Card PAN returned unmasked in refund confirmation response | Critical | Open |
