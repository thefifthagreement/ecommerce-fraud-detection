# E-commerce Fraud Detection - Product & UX Implementation Guide

## Table of Contents
1. [User Flows](#user-flows)
2. [Screen-by-Screen Specifications](#screen-specifications)
3. [Edge Cases & Error Handling](#edge-cases)
4. [Performance Requirements](#performance-requirements)
5. [Implementation Timeline](#implementation-timeline)

---

## User Flows

### Flow 1: Low Risk Transaction (80% of customers)

```
User adds items to cart
    ↓
Proceeds to checkout
    ↓
Enters payment info
    ↓
Clicks "Complete Purchase"
    ↓
[BACKGROUND: Model scores transaction → Low Risk 15%]
    ↓
INSTANT: Order confirmation page
    ↓
Email confirmation sent
    ↓
Order fulfilled normally

USER EXPERIENCE: Seamless, no friction
TIME TO COMPLETE: 30 seconds (standard checkout)
```

---

### Flow 2: Medium Risk Transaction (15% of customers)

```
User adds items to cart
    ↓
Proceeds to checkout
    ↓
Enters payment info
    ↓
Clicks "Complete Purchase"
    ↓
[BACKGROUND: Model scores transaction → Medium Risk 52%]
    ↓
Loading spinner (2 seconds)
    ↓
┌─────────────────────────────────────────┐
│  Verify Your Identity                   │
│                                          │
│  To protect your account, please enter  │
│  the code we sent to:                   │
│  ••••@gmail.com                         │
│                                          │
│  [____  ____  ____  ____  ____  ____]  │
│                                          │
│  Didn't receive it? [Resend] [Try SMS] │
│                                          │
│  Why am I seeing this?                  │
└─────────────────────────────────────────┘
    ↓
User enters code
    ↓
[BACKGROUND: Verify code]
    ↓
Order confirmation page
    ↓
Email confirmation sent

USER EXPERIENCE: Minor friction, clear explanation
TIME TO COMPLETE: 2-3 minutes
ABANDONMENT RISK: Medium (10-15% may abandon)
```

**Alternative Path: Phone Verification**
```
User clicks "Try SMS"
    ↓
Receives SMS with code
    ↓
Enters code → Approved

OR

User clicks "Why am I seeing this?"
    ↓
Help article explains security
    ↓
Option to contact support
```

---

### Flow 3: High Risk Transaction (4% of customers)

```
User adds items to cart
    ↓
Proceeds to checkout
    ↓
Enters payment info
    ↓
Clicks "Complete Purchase"
    ↓
[BACKGROUND: Model scores transaction → High Risk 73%]
    ↓
Loading spinner (2 seconds)
    ↓
┌─────────────────────────────────────────┐
│  Order Under Review                     │
│                                          │
│  Your order #12345 is being reviewed    │
│  for security. This is normal for new   │
│  customers.                              │
│                                          │
│  You'll receive an email within 1 hour  │
│  at: user@example.com                   │
│                                          │
│  Want faster approval?                  │
│  [Verify Identity Now] [Contact Support]│
│                                          │
│  Your payment has NOT been charged yet. │
└─────────────────────────────────────────┘
    ↓
[OPTION A: User waits]
    ↓
Fraud analyst reviews (15 min avg)
    ↓
If approved: Email + order proceeds
If declined: Email with explanation + refund
    ↓
[OPTION B: User clicks "Verify Identity Now"]
    ↓
┌─────────────────────────────────────────┐
│  Quick Verification Options             │
│                                          │
│  Choose one method:                     │
│                                          │
│  📱 Phone Call (2 minutes)              │
│     We'll call to verify your identity  │
│     [Start Call]                        │
│                                          │
│  📸 Photo ID (5 minutes)                │
│     Upload driver's license or passport │
│     [Upload Photo]                      │
│                                          │
│  💳 Bank Verification (10 minutes)      │
│     We'll send a micro-deposit          │
│     [Start Verification]                │
└─────────────────────────────────────────┘
    ↓
User completes verification
    ↓
Instant approval or manual review expedited

USER EXPERIENCE: Friction acknowledged, options provided
TIME TO COMPLETE: 1 hour (passive) or 5-10 min (active verification)
ABANDONMENT RISK: High (30-40% may abandon)
MITIGATION: Clear communication, fast resolution options
```

---

### Flow 4: Very High Risk Transaction (1% of customers)

```
User adds items to cart
    ↓
Proceeds to checkout
    ↓
Enters payment info
    ↓
Clicks "Complete Purchase"
    ↓
[BACKGROUND: Model scores transaction → Very High Risk 94%]
    ↓
Loading spinner (2 seconds)
    ↓
┌─────────────────────────────────────────┐
│  ⚠️ Payment Declined                    │
│                                          │
│  We couldn't process this transaction.  │
│                                          │
│  This can happen if:                    │
│  • Payment information is incorrect     │
│  • Your bank declined the charge        │
│  • Additional verification is needed    │
│                                          │
│  What you can do:                       │
│  • [Try Different Card]                 │
│  • [Contact Support] (Instant Chat)     │
│  • [Review Payment Info]                │
│                                          │
│  Reference: #12345                      │
└─────────────────────────────────────────┘
    ↓
[OPTION A: User tries different card]
    ↓
Re-score with new attempt
    ↓
If still high risk → Same message
If risk drops → Proceed to appropriate flow
    ↓
[OPTION B: User contacts support]
    ↓
Live chat with support agent
    ↓
Agent verifies identity (security questions)
    ↓
If legitimate: Manual override, order approved
If suspicious: Politely decline, log attempt
    ↓
[OPTION C: User abandons]
    ↓
Send follow-up email within 24 hours:
"We noticed you had trouble completing your order.
We're here to help! Contact support at..."

USER EXPERIENCE: Clear decline, no ambiguity
TIME TO COMPLETE: Declined (or 10-30 min with support)
ABANDONMENT RISK: Very High (60-80% abandon)
MITIGATION: Excellent support experience for false positives
```

---

## Screen Specifications

### Screen 1: Verification Code Entry (Medium Risk)

**Layout:**
```
┌─────────────────────────────────────────────┐
│  [Logo]                          [Help (?) ]│
├─────────────────────────────────────────────┤
│                                              │
│         🔒 Verify Your Identity              │
│                                              │
│  To protect your account, please enter the  │
│  code we sent to:                           │
│                                              │
│  ✉️  m••••k@gmail.com                       │
│                                              │
│  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐      │
│  │ 1 │ │ 2 │ │ 3 │ │ 4 │ │ 5 │ │ 6 │      │
│  └───┘ └───┘ └───┘ └───┘ └───┘ └───┘      │
│                                              │
│  Code expires in: 9:45                      │
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │  Didn't receive it?                    │ │
│  │  [Resend Email]  [Send SMS Instead]    │ │
│  └────────────────────────────────────────┘ │
│                                              │
│  [Why am I seeing this?]                    │
│                                              │
│            [Verify and Continue]            │
│                                              │
│  Your order total: $127.50                  │
│  Payment method: •••• 4242                  │
└─────────────────────────────────────────────┘
```

**Specifications:**
- **Code format**: 6 digits, auto-advance on entry
- **Expiration**: 10 minutes
- **Resend cooldown**: 30 seconds
- **Max attempts**: 3 failed attempts → trigger manual review
- **Accessibility**: Screen reader compatible, high contrast mode
- **Mobile**: Full screen on mobile, easy thumb typing

**Copy Variations:**

| Scenario | Header | Body |
|----------|--------|------|
| First-time buyer | "Verify Your Identity" | "As a new customer, we need to verify..." |
| High-value purchase | "Secure Your Purchase" | "For high-value orders, we require..." |
| Unusual pattern | "Account Security Check" | "We noticed unusual activity and need to verify..." |

**A/B Testing:**
- Test emoji usage (🔒 vs. no emoji)
- Test tone (formal vs. friendly)
- Test social proof ("99% of verifications complete in 60 seconds")

---

### Screen 2: Order Under Review (High Risk)

**Layout:**
```
┌─────────────────────────────────────────────┐
│  [Logo]                          [Support] │
├─────────────────────────────────────────────┤
│                                              │
│       ⏳ Your Order is Being Reviewed        │
│                                              │
│  Order #12345                               │
│  Status: Security Review                    │
│                                              │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                              │
│  What's happening?                          │
│                                              │
│  To protect all our customers, we review    │
│  certain orders before processing. This is  │
│  completely normal, especially for:         │
│                                              │
│  ✓ First-time purchases                     │
│  ✓ New account orders                       │
│  ✓ High-value transactions                  │
│                                              │
│  You'll hear from us within 1 hour at:     │
│  📧 mickael.vitry@gmail.com                 │
│                                              │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                              │
│  Want faster approval?                      │
│                                              │
│  ┌──────────────────────────────────────┐  │
│  │  [🎯 Verify Identity Now - 5 mins]  │  │
│  └──────────────────────────────────────┘  │
│                                              │
│  [💬 Chat with Support]  [📧 Email Us]     │
│                                              │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                              │
│  Your Order Details:                        │
│  • 2 items                                  │
│  • Total: $127.50                           │
│  • Payment: NOT charged yet                 │
│  • Shipping: Holds after approval           │
│                                              │
│  [View Full Order]                          │
└─────────────────────────────────────────────┘
```

**Specifications:**
- **Auto-refresh**: Every 30 seconds to update status
- **Notifications**: Browser notification when approved
- **Email**: Immediate confirmation email with same info
- **SMS option**: "Text me when approved" checkbox
- **Transparency**: Show what we're checking without revealing security methods

**Key Messages:**
1. "This is normal" - Normalize the experience
2. "Not charged yet" - Reduce financial anxiety
3. "Fast options available" - Empower the user
4. "We're protecting you" - Frame as customer benefit

---

### Screen 3: Payment Declined (Very High Risk)

**Layout:**
```
┌─────────────────────────────────────────────┐
│  [Logo]                     [Need Help? →] │
├─────────────────────────────────────────────┤
│                                              │
│       ⚠️ We Couldn't Process This Payment   │
│                                              │
│  Your transaction was declined for          │
│  security reasons.                          │
│                                              │
│  This can happen when:                      │
│                                              │
│  • Payment information doesn't match        │
│  • Your bank declined the transaction       │
│  • Additional verification is required      │
│  • Account security concerns                │
│                                              │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                              │
│  What you can do:                           │
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │  💳 Try a Different Payment Method     │ │
│  │     Use another card or PayPal         │ │
│  │     [Change Payment Method]            │ │
│  └────────────────────────────────────────┘ │
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │  💬 Contact Support (Available Now)    │ │
│  │     Chat with us to resolve this       │ │
│  │     [Start Chat] [Call Us]             │ │
│  └────────────────────────────────────────┘ │
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │  📝 Review Your Information            │ │
│  │     Check billing address & card info  │ │
│  │     [Review & Edit]                    │ │
│  └────────────────────────────────────────┘ │
│                                              │
│  Reference Number: #RF-12345                │
│  (Share this with support)                  │
│                                              │
│  No charges were made to your account.      │
└─────────────────────────────────────────────┘
```

**Specifications:**
- **Tone**: Helpful, not accusatory
- **Never mention**: "Fraud", "Suspicious", "Blocked"
- **Always provide**: Multiple resolution paths
- **Support SLA**: Live chat available 24/7
- **Follow-up**: Email within 1 hour offering help

**Copy Testing:**
- Avoid: "Your transaction was flagged as fraudulent"
- Use: "We couldn't process this payment"
- Avoid: "Your account is suspicious"
- Use: "Additional verification is needed"

---

## Edge Cases & Error Handling

### Edge Case 1: User Changes Behavior Mid-Session

**Scenario**: User browses for 2 hours, then completes purchase quickly

**Model Input**: Signup date (weeks ago) vs. session start time

**Solution**: Track session start time separately, don't flag long-time customers who happen to checkout quickly in one session

```python
# Feature engineering improvement
if user.days_since_signup > 7:
    time_delta = session_time_to_purchase
else:
    time_delta = signup_time_to_purchase
```

---

### Edge Case 2: Multiple Failed Verification Attempts

**Scenario**: User enters wrong verification code 3 times

**Risk**: Legitimate customer locked out vs. fraudster brute-forcing

**Solution**: Progressive escalation
```
Attempt 1: "Code incorrect, please try again"
Attempt 2: "Code incorrect. You have 1 more attempt before we need to verify manually"
Attempt 3: "For security, we've sent this to manual review. Contact support for immediate help."
```

**Alternative**: After 2 failed attempts, offer "Call me instead" button

---

### Edge Case 3: Model Service Downtime

**Scenario**: ML model API is down or slow (>5 second response time)

**Business Rule**: Never block revenue due to technical issues

**Fallback Strategy**:
```
if model_response_time > 5_seconds or model_error:
    # Approve transaction
    # Flag for post-purchase review
    log_incident(transaction_id, "Model timeout - approved by fallback")
    send_alert_to_fraud_team()

    # Post-purchase check within 10 minutes
    schedule_review(transaction_id, priority="high")
```

**User Experience**: Seamless approval, no indication of technical issue

---

### Edge Case 4: International Customers

**Scenario**: Customer from country not in training data

**Risk**: Model treats as "unknown" (higher fraud risk)

**Solution**:
1. Country mapping to similar fraud-profile countries
2. Separate model or threshold for international customers
3. Offer localized verification methods

```python
# Improved country handling
if country not in trained_countries:
    if country in low_fraud_regions:
        use_optimistic_threshold()
    elif country in high_fraud_regions:
        use_conservative_threshold()
    else:
        use_default_threshold()
        offer_additional_verification_options()
```

---

### Edge Case 5: Gift Purchases

**Scenario**: User buying gift for someone else (shipping to different address)

**Red Flag**: Billing ≠ Shipping address (classic fraud indicator)

**Legitimate Use Case**: Common gift behavior

**Solution**:
```
if billing_address != shipping_address:
    risk_score += 15  # Moderate increase

    # Contextual verification
    if is_holiday_season() or gift_message_included():
        risk_score -= 10  # Reduce for likely gift

    # Ask context question
    show_question("Is this a gift?")
    if user_confirms_gift:
        risk_score -= 5
        add_gift_receipt()
```

---

### Edge Case 6: VPN/Proxy Users

**Scenario**: User's IP location doesn't match payment card country

**Why**: VPN usage, traveling, privacy-conscious users

**Current Problem**: Flagged as "unknown" country or high-risk location

**Solution**:
```
if suspicious_ip_pattern:
    # Don't auto-decline
    # Instead, use additional signals

    check_device_fingerprint()
    check_email_domain()
    check_browser_timezone_vs_ip_location()

    # If multiple signals align with legitimate use
    if confidence_score > threshold:
        proceed_with_verification()
    else:
        manual_review()
```

---

## Performance Requirements

### Latency SLAs

| Component | Target | Max Acceptable | User Impact |
|-----------|--------|----------------|-------------|
| Model prediction | <100ms | 200ms | None (imperceptible) |
| Email verification send | <2s | 5s | Minor (loading state) |
| SMS verification send | <5s | 10s | Noticeable (progress bar) |
| Manual review assignment | <1s | 5s | None (async) |
| Support chat response | <30s | 2min | High (show wait time) |

### Availability Requirements

| System | Uptime SLA | Fallback Strategy |
|--------|------------|-------------------|
| ML Model API | 99.9% | Auto-approve + post-review |
| Verification Service | 99.5% | Multiple backup channels |
| Manual Review Queue | 99.99% | Critical path, no SPF |
| Customer Support | 99% | Overflow to email/callback |

### Scalability Targets

| Metric | Current | Year 1 | Year 3 |
|--------|---------|--------|--------|
| Transactions/day | 10,000 | 50,000 | 200,000 |
| Model predictions/sec | 5 | 25 | 100 |
| Concurrent verifications | 50 | 500 | 2,000 |
| Manual reviews/day | 400 | 2,000 | 5,000 |

### Model Performance Monitoring

**Real-time Dashboards:**
```
┌─────────────────────────────────────────────┐
│  Fraud Detection Dashboard (Live)           │
├─────────────────────────────────────────────┤
│  Transactions Today: 8,432                  │
│                                              │
│  Risk Distribution:                         │
│  ████████████████░░░░ 80% Low Risk          │
│  ███░░░░░░░░░░░░░░░░ 15% Medium Risk        │
│  █░░░░░░░░░░░░░░░░░░  4% High Risk          │
│  ░░░░░░░░░░░░░░░░░░░░  1% Very High Risk    │
│                                              │
│  Verifications Sent: 1,265                  │
│  ✓ Completed: 1,140 (90%)                   │
│  ⏳ Pending: 98 (8%)                         │
│  ✗ Failed: 27 (2%)                          │
│                                              │
│  Manual Reviews:                            │
│  Queue: 12 transactions                     │
│  Avg Review Time: 8 minutes                 │
│  Approved: 89% | Declined: 11%              │
│                                              │
│  Model Health:                              │
│  ✓ API Latency: 87ms (Good)                 │
│  ✓ Availability: 99.98%                     │
│  ⚠ False Positive Rate: 6.2% (Target: <5%) │
│                                              │
│  [View Details] [Export Report] [Alerts]   │
└─────────────────────────────────────────────┘
```

**Alert Conditions:**
- False positive rate >7% for 1 hour
- Model latency >200ms for 5 minutes
- Manual review queue >50 items
- Verification failure rate >10%
- Customer complaints >5/hour about declined transactions

---

## Implementation Timeline

### Phase 1: MVP (Weeks 1-4)

**Goals**: Basic fraud detection with manual fallback

**Week 1-2: Infrastructure**
- ✅ Deploy model to production API
- ✅ Set up monitoring/logging
- ✅ Create manual review queue
- ✅ Train fraud analyst team

**Week 3-4: Basic Integration**
- ✅ Integrate model into checkout flow
- ✅ Implement high-risk decline screen
- ✅ Set up email verification for medium risk
- ✅ A/B test with 10% of traffic

**Success Criteria:**
- Model responds <200ms
- Zero downtime deployments
- Manual review SLA <1 hour
- Test group shows <10% increased cart abandonment

---

### Phase 2: Verification & UX (Weeks 5-8)

**Goals**: Reduce friction, improve conversion

**Week 5-6: Enhanced Verification**
- ✅ SMS verification option
- ✅ Multiple verification code channels
- ✅ Quick identity verification flows
- ✅ Support team training & scripts

**Week 7-8: UX Optimization**
- ✅ A/B test verification screen variations
- ✅ Implement progress indicators
- ✅ Add contextual help content
- ✅ Optimize copy for different risk levels
- ✅ Increase to 50% of traffic

**Success Criteria:**
- Verification completion rate >85%
- Customer satisfaction maintained >4.2/5
- Support ticket volume <100/week
- False positive complaints <1% of flagged transactions

---

### Phase 3: Advanced Features (Weeks 9-12)

**Goals**: Personalization and automation

**Week 9-10: Risk Personalization**
- ✅ Customer trust scoring
- ✅ Behavioral history integration
- ✅ Dynamic threshold adjustment
- ✅ Geographic personalization

**Week 11-12: Automation & Scale**
- ✅ Auto-approve previously verified users
- ✅ Automated follow-up for declined transactions
- ✅ Predictive support (reach out before user complains)
- ✅ Roll out to 100% of traffic

**Success Criteria:**
- Chargeback rate reduced by >50%
- Cart abandonment increase <3%
- Manual review volume decreases by 30%
- Customer satisfaction returns to baseline

---

### Phase 4: Continuous Improvement (Ongoing)

**Monthly:**
- Retrain model with new fraud patterns
- Review false positive cases
- Update verification copy based on feedback
- Optimize thresholds per customer segment

**Quarterly:**
- Major UX tests (new verification methods)
- Model architecture improvements
- Expansion to new markets/products
- Competitive analysis

**Annually:**
- Complete model rebuild with new techniques
- Technology stack evaluation
- Strategic fraud prevention roadmap
- ROI analysis and business case update

---

## Key Metrics & Success Criteria

### Primary KPIs

1. **Fraud Detection Rate**
   - Current: 0% (no system)
   - Target: 70% (catch 7/10 fraudulent transactions)
   - Measured: Chargebacks prevented

2. **False Positive Rate**
   - Current: N/A
   - Target: <5% (95% of flagged customers are legitimate)
   - Measured: Flagged transactions that complete successfully

3. **Customer Satisfaction**
   - Current: 4.5/5 stars
   - Target: >4.3/5 stars (minimal impact)
   - Measured: CSAT surveys, NPS

4. **Cart Abandonment Rate**
   - Current: 22% (industry baseline)
   - Target: <25% (max 3% increase)
   - Measured: Checkout funnel analytics

5. **Revenue Protection**
   - Current: $24k/month lost to fraud
   - Target: <$8k/month (70% reduction)
   - Measured: Chargebacks + fraud losses

### Secondary KPIs

- Verification completion rate: >85%
- Manual review SLA met: >95% within 1 hour
- Support ticket volume: <2% of flagged transactions
- Model prediction latency: <100ms p99
- System uptime: >99.9%

---

## Customer Communication Strategy

### Email Templates

**Template 1: Order Under Review**
```
Subject: Your Order #12345 - Security Review in Progress

Hi [Name],

Thanks for your order! We're currently reviewing it as part of our
security process to protect all customers.

Order Status: Under Review
Expected Resolution: Within 1 hour
Payment Status: Not charged yet

This is normal for new customers and certain purchase types. You'll
receive an email as soon as we approve your order.

Want faster approval?
→ Verify your identity (5 minutes): [Link]
→ Contact support: [Link]

Order Details:
- Order #12345
- 2 items, Total: $127.50
- Shipping to: [Address]

Thanks for your patience!

The [Company] Team

P.S. - Once approved, your order will ship immediately.
```

**Template 2: Order Approved After Review**
```
Subject: Great News! Your Order #12345 is Confirmed ✓

Hi [Name],

Your order has been approved and is being processed!

Order #12345: Confirmed
Payment: Charged
Shipping: Preparing now
Tracking: [Link] (available within 24 hours)

Thanks for your patience with our security check. This was a one-time
review, and future orders will process instantly.

Track Your Order: [Link]
View Order Details: [Link]

Questions? We're here to help: [Support Link]

Happy shopping!
The [Company] Team
```

**Template 3: Transaction Declined**
```
Subject: Action Needed - Issue with Order #12345

Hi [Name],

We had trouble processing your order. This can happen for several
reasons:

• Payment information doesn't match our records
• Your bank declined the transaction
• Additional verification is needed

Your items are still in your cart, and we'd love to help you complete
this order!

What You Can Do:
→ Try a different payment method: [Link]
→ Contact support (we're available 24/7): [Link]
→ Review your payment information: [Link]

Your Cart (Saved):
- 2 items, Total: $127.50
- Cart expires in: 7 days

Reference #RF-12345 (share this with support)

We're here to help!
The [Company] Team
```

---

## Support Team Scripts

### Script 1: Customer Calls About Declined Transaction

**Agent Opening:**
"Thank you for calling [Company]. My name is [Name]. I understand you had some trouble completing an order. I apologize for the inconvenience - I'm here to help you get this resolved right away. Can you provide your order reference number or email address?"

**Verification Steps:**
1. Look up order by reference number
2. Confirm customer identity:
   - "For security, can you confirm the billing address on file?"
   - "What are the last 4 digits of the card you used?"
   - "What items were in your order?"

**If Legitimate Customer (90% of cases):**
"Thank you for confirming. I can see why our system flagged this - it was [reason: new account/high value/etc]. This is completely normal, and I can approve this order right now for you. Your card will be charged $127.50, and you'll receive a confirmation email within 5 minutes. Sound good?"

[Process manual approval]

"Perfect! Your order #12345 is confirmed and will ship within 24 hours. I've also added a note to your account so this won't happen again. Is there anything else I can help you with today?"

**If Suspicious (10% of cases):**
"I appreciate your patience. I'm seeing some information that doesn't quite match up. For security, I'd like to verify a few more details..."

[Additional verification questions]

"Thank you for working with me on this. Unfortunately, I'm not able to approve this transaction at this time. This is to protect both you and our customers. You're welcome to try again with a different payment method, or I can escalate this to our fraud prevention team for further review. Which would you prefer?"

**Key Phrases:**
- ✅ "I apologize for the inconvenience"
- ✅ "This is to protect all our customers"
- ✅ "I can help you resolve this right now"
- ✅ "This is completely normal"
- ❌ "You were flagged for fraud"
- ❌ "Your account is suspicious"
- ❌ "We can't trust this transaction"

---

## Appendix: Technical Integration

### API Endpoint Specification

**POST /api/fraud/predict**

Request:
```json
{
  "user_id": 12345,
  "signup_time": "2025-01-15T10:30:00Z",
  "purchase_time": "2025-01-15T10:35:00Z",
  "purchase_value": 127.50,
  "age": 32,
  "country": "United States",
  "browser": "Chrome",
  "sex": "M",
  "source": "SEO",
  "purchase_weekday": 1,
  "purchase_hour": 10
}
```

Response:
```json
{
  "transaction_id": "txn_abc123",
  "fraud_probability": 0.52,
  "risk_level": "MEDIUM",
  "recommended_action": "VERIFY_EMAIL",
  "confidence": 0.89,
  "model_version": "v2.1.0",
  "prediction_time_ms": 87,
  "features_used": 14,
  "top_risk_factors": [
    {"feature": "signup_purchase_timedelta", "contribution": 0.72},
    {"feature": "purchase_value", "contribution": 0.15},
    {"feature": "country", "contribution": 0.08}
  ]
}
```

### Frontend Integration Example

```javascript
// At checkout submission
async function handleCheckout(checkoutData) {
  showLoading();

  try {
    // Call fraud detection API
    const fraudCheck = await fetch('/api/fraud/predict', {
      method: 'POST',
      body: JSON.stringify(checkoutData)
    });

    const result = await fraudCheck.json();

    // Handle based on risk level
    switch(result.risk_level) {
      case 'LOW':
      case 'VERY_LOW':
        // Standard checkout flow
        return processPayment();

      case 'MEDIUM':
        // Show verification modal
        return showVerificationModal(result);

      case 'HIGH':
        // Show manual review screen
        return showManualReviewScreen(result);

      case 'VERY_HIGH':
        // Decline with support options
        return showDeclineScreen(result);
    }
  } catch (error) {
    // Fallback: approve and review post-purchase
    logError(error);
    return processPaymentWithFlag();
  }
}
```

---

**Document Version**: 1.0
**Last Updated**: November 2025
**Owner**: Product & Engineering Teams
**Next Review**: Monthly during Phase 1-3, Quarterly thereafter
