# Fraudulent activities

> Jedha Data Science Bootcamp - Fullstack - week 4

> _Session dsmft-paris-08_

### OBJECTIF

Les sites de E-commerce font transiter beaucoup d'argent. Cela peut engendrer des risques non négligeables d'activités frauduleuses, comme l'utilisation de carte de crédit volées, du blanchiment d'argent, etc.

Fort heureusement, le Machine Learning peut nous aider à identifier ces activités frauduleuses. Tous les sites web où vous devez entrer vos informations de paiements ont une équipe qui s'occupe de gérer les risques de fraude via le ML.

Le but de ce challenge est de construire un modèle qui vous permet de prédire une probabilité de transaction frauduleuse.

### DESCRIPTION

L'entreprise X fait du E-commerce et vend des vêtements faits-main. Votre but est de construire un modèle qui permette de prédire si l'achat d'un vêtement doit être considéré comme une transaction frauduleuse ou non.

Voici précisément ce que vous devez faire :

1. Pour chacun des utilisateurs, déterminez le pays d'origine depuis son adresse IP

2. Construisez un modèle qui permette de prédire si l'activité est frauduleuse ou non. Expliquez aussi vos choix / hypothèses en termes d'optimisation de faux-positifs et faux-négatifs

3. Votre patron aimerait comprendre votre modèle car il est inquiet d'utiliser un modèle _black box_. Comment l'expliqueriez vous d'un point utilisateur, et non pas mathématique. Par exemple, quels sont les utilisateurs qui peuvent être classés comme _risqués_ ?

4. Supposons que vous pouvez utiliser votre modèle en live pour qu'il fasse sa prédiction en temps réel. D'un point de vue Produit, comment l'utiliseriez-vous ? Comment pourriez-vous penser l'expérience utilisateur face à ce produit ?


### DATA

Vous pouvez utiliser les deux tables suivantes :

```python
Fraud_Data
```

&

```python
IpAddress_to_Country
```

---

## PROJECT DELIVERABLES

### 📊 Analysis & Modeling

1. **Exploratory Data Analysis** - `fraud_detection_eda.ipynb`
   - IP address to country mapping
   - Feature engineering (signup_purchase_timedelta, temporal features)
   - Comprehensive visualizations of fraud patterns
   - Key finding: Fraudulent transactions occur within seconds of signup (median: 1 second)

2. **Machine Learning Models** - `fraud_detection_ml.ipynb`
   - Logistic Regression
   - Random Forest (best performer: F2-score ~0.59)
   - XGBoost
   - Stacking Ensemble
   - Feature importance analysis (signup_purchase_timedelta accounts for 96% of predictions)

### 📖 Business Documentation

3. **Model Explanation for Non-Technical Stakeholders** - `MODEL_EXPLANATION.md`
   - How the model works in plain language
   - Risky user profiles and fraud patterns
   - Model confidence levels and accuracy expectations
   - Limitations and biases
   - ROI analysis and business impact

4. **Product & UX Implementation Guide** - `PRODUCT_UX_GUIDE.md`
   - Real-time fraud detection system architecture
   - User flows for all risk levels (Low, Medium, High, Very High)
   - Screen-by-screen specifications with copy
   - Edge cases and error handling
   - Performance requirements and monitoring
   - Implementation timeline (12-week rollout plan)
   - Customer communication templates
   - Support team scripts

### 🎯 Key Insights

**Primary Fraud Indicator:**
- **Time-to-purchase**: Fraudsters act instantly (seconds), legitimate customers take days/weeks
- This single feature has 96% importance in the model

**Model Performance:**
- F2-score: 0.59 (optimized for catching fraud while minimizing false positives)
- Catches ~70% of fraudulent transactions
- False positive rate: ~5% (acceptable friction for legitimate customers)

**Risk Factors:**
- Instant purchase after signup
- Unknown/high-risk geographic locations
- Specific browser patterns
- Transaction timing (weekday, hour)
- Purchase value

**Business Impact:**
- Current fraud cost: ~$24k/month
- Projected savings: ~$22k/month (70% reduction)
- Immediate ROI with proper UX implementation

### 🚀 Production Readiness

The documentation includes:
- Real-time API specifications
- Frontend integration examples
- UX flows that balance security with customer experience
- Support playbooks for handling false positives
- Monitoring dashboards and alert conditions
- 12-week phased rollout plan

**Recommended Next Steps:**
1. Deploy model to staging environment (Week 1-2)
2. A/B test with 10% traffic (Week 3-4)
3. Implement verification flows (Week 5-8)
4. Roll out advanced features (Week 9-12)
5. Continuous monitoring and improvement (Ongoing)
