# E-commerce Fraud Detection - Business Documentation

## Executive Summary

This document explains how our fraud detection model works in business terms and provides recommendations for implementing it in production. Our model achieved a **59% F2-score** on the test set, meaning it successfully identifies fraudulent transactions while minimizing false alarms that would frustrate legitimate customers.

---

## 1. How the Model Works (Non-Technical Explanation)

### The Core Principle: Time is the Tell

Our fraud detection model has identified one overwhelmingly important pattern: **fraudsters act fast, legitimate customers take their time.**

**The numbers tell the story:**
- **Legitimate customers** wait an average of ~60 days between signing up and making their first purchase
- **Fraudsters** make purchases within seconds (median: 1 second) of creating an account

This single indicator accounts for **96% of the model's decision-making**, making it the most powerful fraud signal we have.

### Why This Makes Sense

Think about normal customer behavior:
1. A real customer discovers your site
2. They browse products, maybe sign up for updates
3. They come back days or weeks later when they're ready to buy
4. They make a purchase decision

A fraudster with a stolen credit card:
1. Creates an account immediately
2. Purchases within seconds before the card is reported stolen
3. Uses maxed-out quantities or high-value items
4. Needs to act before detection

### Other Risk Factors (The Supporting Evidence)

While time-to-purchase is the strongest signal, the model also considers:

**Purchase Behavior:**
- **Transaction value**: Higher value purchases have slightly elevated risk
- **Day of week**: Fraudulent activity peaks on certain weekdays (Monday, Tuesday, Thursday)
- **Time of day**: Certain hours show different risk patterns

**User Profile:**
- **Browser type**: Chrome and IE users show different fraud patterns
- **Geographic location**: Some countries have higher fraud rates (US, China, Japan, UK, Korea)
- **Age**: Slight variations, but less significant

**What Doesn't Matter Much:**
- Gender: Nearly identical fraud rates between M/F
- Traffic source (SEO, Ads, Direct): Minimal difference
- Most other demographic factors

---

## 2. Risky User Profiles

### HIGH RISK ⚠️

A transaction is considered high risk when it exhibits these characteristics:

1. **Instant Gratification Pattern**
   - Account created and purchase made within minutes (especially under 1 hour)
   - No browsing history or product research time
   - First-time buyer with no previous engagement

2. **Red Flag Combinations**
   - Instant purchase + Unknown/suspicious geographic location
   - Instant purchase + High transaction value (>$100)
   - Instant purchase + Transaction during off-peak hours

3. **Geographic Risk**
   - IP address from high-fraud countries (notably when combined with other factors)
   - IP country doesn't match payment card country (not in current dataset, but recommended)
   - "Unknown" country classification (suspicious IP patterns)

### MEDIUM RISK ⚡

1. **Fast but Not Instant**
   - Purchase within 1-24 hours of signup
   - Moderate transaction value ($50-$100)
   - Limited site engagement before purchase

2. **Behavioral Anomalies**
   - Purchase patterns that deviate from user's country norms
   - Specific browser/device combinations with elevated fraud rates
   - Purchases during high-fraud time windows

### LOW RISK ✅

1. **Normal Customer Behavior**
   - Days or weeks between signup and first purchase
   - Multiple browsing sessions before purchase
   - Moderate transaction values
   - Standard geographic patterns

2. **Established Customers**
   - Previous successful purchases
   - Consistent purchase patterns
   - Verified payment methods

---

## 3. Model Confidence Levels

Our model outputs a **probability score from 0-100%**:

- **0-20%**: Very Low Risk - Standard processing
- **20-40%**: Low Risk - Standard processing, enhanced monitoring
- **40-60%**: Medium Risk - Additional verification recommended
- **60-80%**: High Risk - Manual review required
- **80-100%**: Very High Risk - Block and investigate

**Important Note:** The model is calibrated to minimize false positives (blocking legitimate customers) while catching most fraud. We optimize for **F2-score** which weighs recall (catching fraud) twice as heavily as precision (avoiding false alarms).

---

## 4. Model Limitations & Transparency

### What the Model Can't See

1. **Device fingerprinting**: Whether the same device is used for multiple accounts
2. **Payment details**: Card verification, BIN data, billing address match
3. **Behavioral biometrics**: Mouse movements, typing patterns
4. **Historical fraud patterns**: Whether this email/device was previously flagged
5. **Inventory abuse**: Whether user is buying out limited stock

### Known Biases

1. **New customer bias**: Legitimate customers who happen to buy quickly will be flagged
   - Example: Gift purchasers, limited-time sales, urgency buyers
2. **Geographic bias**: Users from high-fraud countries face more scrutiny
3. **Time-based patterns**: Model trained on 2015 data may not reflect current patterns

### Accuracy Expectations

- **Recall (Fraud Detection Rate)**: ~70% - We catch 7 out of 10 fraudulent transactions
- **Precision (Accuracy when flagging fraud)**: ~45% - About half of flagged transactions are actually fraud
- **False Positive Rate**: ~8% - About 8% of legitimate customers may need additional verification

**This is the trade-off:** To catch more fraud, we accept some inconvenience to legitimate customers. The key is designing the UX to minimize this friction (see next section).

---

## 5. Real-Time Implementation Strategy

### System Architecture

```
Customer Purchase → Real-time API Call → Model Prediction → Risk-Based Action
                         ↓
                    < 100ms response
```

**Technical Requirements:**
1. Model deployed as REST API endpoint
2. Sub-100ms prediction latency required
3. Feature calculation must happen in real-time
4. Fallback to "approve" if model service is down (never block revenue)

### Integration Points

**At Checkout:**
```python
# Pseudocode
purchase_time = now()
signup_time = get_user_signup_time(user_id)
time_delta = (purchase_time - signup_time).total_minutes()

features = {
    'signup_purchase_timedelta': time_delta,
    'purchase_value': cart_total,
    'age': user_age,
    'country': user_country,
    'browser': user_browser,
    'purchase_weekday': purchase_time.weekday(),
    'purchase_hour': purchase_time.hour,
    # ... other features
}

fraud_probability = model.predict(features)
risk_level = categorize_risk(fraud_probability)

# Take action based on risk level
if risk_level == 'VERY_HIGH':
    block_transaction()
    trigger_manual_review()
elif risk_level == 'HIGH':
    require_additional_verification()
elif risk_level == 'MEDIUM':
    allow_but_flag_for_review()
else:
    approve_transaction()
```

### Risk-Based Actions Matrix

| Risk Level | Probability | Action | User Experience |
|------------|-------------|--------|-----------------|
| Very Low | 0-20% | Approve | Standard checkout |
| Low | 20-40% | Approve + Monitor | Standard checkout |
| Medium | 40-60% | Additional verification | SMS code or email verification |
| High | 60-80% | Manual review | "Processing - will notify within 1 hour" |
| Very High | 80-100% | Block | "Payment declined - contact support" |

---

## 6. User Experience Design

### Principle: Security Without Friction

The goal is to stop fraud while keeping legitimate customers happy. Here's how:

### UX Recommendation 1: Invisible Security

**For Low Risk Transactions (80% of customers):**
- No visible security measures
- Standard checkout flow
- Instant order confirmation
- Silent background monitoring

**Why:** Most customers are legitimate. Don't add friction where it's not needed.

### UX Recommendation 2: Progressive Friction

**For Medium Risk Transactions (15% of customers):**

**Option A: Soft Verification (Recommended)**
```
"To protect your account, we've sent a verification code to your email/phone.
Please enter it to complete your purchase."

[Verification Code Input]
[Didn't receive it? Resend code]

Why am I seeing this? [link to help article]
```

**Option B: Delayed Approval**
```
"Your order is being processed for security.
You'll receive confirmation within 1 hour.

Order Number: #12345
Status: Security Review (This is normal for new customers)

We'll email you at: user@example.com"
```

**Why:** Frame security as protection for the customer, not suspicion.

### UX Recommendation 3: Clear Communication for High Risk

**For High Risk Transactions (5% of customers):**

**Block Screen:**
```
"We couldn't process this transaction"

To protect our customers, we've declined this payment.
This can happen for several reasons:

• New account with immediate purchase
• Unusual purchase patterns
• Geographic verification needed

What you can do:
1. Contact our support team (instant chat available)
2. Try a different payment method
3. Visit our FAQ: "Why was my payment declined?"

Order Reference: #12345

[Contact Support] [Try Different Payment]
```

**Why:**
- Be honest but not accusatory
- Provide immediate resolution paths
- Don't say "fraud" - it alienates legitimate customers caught in false positives

### UX Recommendation 4: Fast-Track for Legitimate "Fast Buyers"

**Problem:** Sale shoppers, gift buyers, and impulse purchasers look like fraudsters.

**Solution:** Quick identity verification options:
1. **Phone verification**: "Confirm your identity with a quick phone call"
2. **Social login**: "Sign in with Google/Facebook to fast-track approval"
3. **Micro-deposit verification**: "We'll send a small charge to verify your card"
4. **Document upload**: "Upload a photo of your ID for instant approval"

**Implementation:**
- Offer these options immediately when flagging a transaction
- Gamify it: "Get verified in 60 seconds!"
- Incentivize: "Verified accounts get free shipping"

### UX Recommendation 5: Build Trust Over Time

**Customer Account Scoring System:**
- New accounts: Higher scrutiny
- After 1st successful purchase: Medium trust
- After 3 purchases over 30+ days: High trust
- Verified identity: Trust boost

**Show customers their trust level:**
```
Your Account: Verified Customer ✓
- Member since: January 2025
- Successful orders: 5
- Trust level: Gold

Benefits:
• Faster checkout
• Priority support
• Early access to sales
```

**Why:** Encourages legitimate customers to build history, making future purchases seamless.

---

## 7. Monitoring & Continuous Improvement

### Key Metrics to Track

1. **Fraud Detection Rate**: % of actual fraud caught by model
2. **False Positive Rate**: % of legitimate customers inconvenienced
3. **Customer Satisfaction**: Impact on NPS/CSAT scores
4. **Revenue Impact**: Lost sales due to false positives
5. **Chargeback Rate**: The ultimate measure of fraud prevention success

### Monthly Model Reviews

- **Retrain model** with new fraud patterns
- **Adjust thresholds** based on business tolerance for risk
- **Feature importance drift**: Are new patterns emerging?
- **Geographic patterns**: New high-risk countries?

### A/B Testing Recommendations

Test different strategies:
- Control: No fraud detection (measure baseline fraud rate)
- Variant A: Current model with standard thresholds
- Variant B: More aggressive thresholds (less friction, more fraud)
- Variant C: Conservative thresholds (more friction, less fraud)

Measure: Fraud rate, customer conversion rate, revenue impact

---

## 8. Operational Playbook

### For Customer Support Teams

**When a customer contacts about blocked transaction:**

1. **Apologize first**: "I apologize for the inconvenience..."
2. **Explain briefly**: "Our security system flagged this transaction to protect customers"
3. **Verify identity**: Ask security questions (address, last 4 of card, etc.)
4. **Offer resolution**:
   - Manual approval after verification
   - Alternative payment method
   - Phone order with verification
5. **Follow up**: "I've noted your account to prevent this in future"

**Never:**
- Say "You look like a fraudster"
- Imply the customer did something wrong
- Leave them without resolution options

### For Fraud Analysts

**When reviewing flagged transactions:**

1. **Check model confidence**: How certain is the model?
2. **Review feature values**: What triggered the flag?
3. **Cross-reference**:
   - Email domain (free email vs. corporate)
   - Phone number (VoIP, burner phones)
   - Shipping address (residential vs. freight forwarder)
   - Device fingerprint (if available)
4. **Make decision**: Approve, request more info, or decline
5. **Feedback loop**: Mark outcome for model retraining

**Target SLA:** All flagged transactions reviewed within 1 hour

---

## 9. ROI & Business Impact

### Cost-Benefit Analysis

**Fraud Costs (Without Model):**
- Chargebacks: $50 per incident + merchandise loss
- Chargeback fees: $20-100 per chargeback
- Account closure risk: Multiple chargebacks = payment processor termination

**Model Costs:**
- Infrastructure: $500/month (API hosting, monitoring)
- False positive impact: ~$10 per inconvenienced customer in lost sales
- Manual review: 2 hours/day analyst time

**Estimated Savings:**
- Assuming 10,000 transactions/month
- 9.36% fraud rate = 936 fraudulent transactions
- Model catches 70% = 655 fraudulent transactions blocked
- Average fraud value: $37
- **Monthly savings**: 655 × $37 = $24,235
- **Minus false positive costs**: ~$2,000
- **Net monthly savings**: ~$22,000

**Payback period:** Immediate

### Success Metrics (6 months)

- **Primary:** Reduce chargeback rate from 9.36% to <3%
- **Secondary:** Maintain customer satisfaction >4.5/5 stars
- **Tertiary:** Keep false positive rate <5%

---

## 10. Next Steps & Recommendations

### Short Term (Month 1-3)

1. ✅ Deploy model to staging environment
2. ✅ A/B test with 10% of traffic
3. ✅ Train customer support on new workflows
4. ✅ Implement basic verification flows (SMS/email)
5. ✅ Set up monitoring dashboards

### Medium Term (Month 4-6)

1. 📊 Collect performance data and retrain model
2. 🔧 Optimize thresholds based on real-world results
3. 🎨 Refine UX based on customer feedback
4. 🌍 Add geographic personalization
5. 🔗 Integrate additional data sources (device fingerprinting)

### Long Term (6+ months)

1. 🤖 Implement advanced ML techniques (deep learning, ensemble methods)
2. 📱 Add behavioral biometrics
3. 🔄 Real-time model updates (online learning)
4. 🌐 Multi-language fraud detection
5. 🎯 Personalized risk scoring per customer

---

## Appendix: Technical Details

### Model Specifications

- **Algorithm**: Random Forest Classifier
- **Training samples**: 120,889 transactions
- **Test samples**: 30,223 transactions
- **Features**: 14 (after feature selection)
- **Performance**:
  - F2-score: 0.59
  - ROC-AUC: ~0.85
  - Recall: ~70%
  - Precision: ~45%

### Most Important Features (Top 5)

1. `signup_purchase_timedelta` (96.1%)
2. `purchase_value` (1.0%)
3. `age` (0.9%)
4. `purchase_weekday` (0.5%)
5. `browser_type` (0.4%)

### Model Update Cadence

- **Retraining**: Monthly with new fraud data
- **A/B testing**: Quarterly for threshold optimization
- **Feature engineering**: Ongoing as new data sources added

---

## Questions?

For technical questions: Contact data science team
For business questions: Contact fraud prevention lead
For customer impact questions: Contact customer experience team

**Document Version**: 1.0
**Last Updated**: November 2025
**Next Review**: December 2025
