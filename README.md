# 🚀 Churn Model MLOps

A compact, end-to-end example demonstrating how to apply **MLOps principles** for building and serving a customer churn prediction model.

---

## 🎯 What This Project Does

### 🏢 Business Context

Consider a telecom company with a large customer base. Some users remain loyal for years, while others stop using the service after a short time — this is known as **churn**.

This project focuses on predicting **which customers are likely to churn**, enabling proactive action.

---

### 👤 Example Customer

* **Name:** John
* **Age:** 38
* **Tenure:** 12 months
* **Monthly Charges:** $65.50
* **Total Spend:** $786
* **Support Calls:** 5

---

### 🤖 Model Output

```json
{
  "churn": 1,
  "churn_probability": 0.73
}
```

---

### 🔍 Interpretation

* `churn = 1` → Customer likely to leave
* `0.73` → 73% probability

👉 Possible reasons:

* Frequent support interactions → dissatisfaction
* Higher monthly charges → pricing concern
* Moderate tenure → not fully loyal

---

### 💡 Business Value

With these insights, organizations can:

* Provide targeted offers or discounts
* Improve customer support experience
* Engage customers before they leave

👉 The goal is to **prevent churn instead of reacting to it**

---

### 🧠 Model Behavior

The model identifies patterns such as:

* 💸 Higher billing → increased churn likelihood
* 📞 More support calls → dissatisfaction signal
* ⏳ Lower tenure → weak customer attachment

---

## 📂 Project Structure

```
churn-model/
├── generate_data.py          # Synthetic dataset creation
├── train.py                  # Model training script
├── api.py                    # FastAPI inference server
├── requirements.txt          # Dependencies
├── Dockerfile                # Container definition
├── .dvc/config               # DVC configuration
├── models/
│   └── churn_model.pkl.dvc   # Model tracked via DVC
├── k8s/
│   ├── deployment.yaml       # Kubernetes deployment
│   └── inference.yaml        # KServe inference config
├── .github/workflows/
│   └── mlops-pipeline.yaml   # CI/CD pipeline
└── argocd/
    └── application.yaml      # GitOps deployment
```

---

## ⚙️ MLOps Pipeline

### 1️⃣ Initial Setup

```bash
uv install -r requirements.txt

python generate_data.py
python train.py

python api.py
# Open: http://localhost:8000/docs
```

---

## 🔌 API Example

```bash
curl -X POST http://localhost:8000/predict \
  -H "Content-Type: application/json" \
  -d '{
    "age": 45,
    "tenure_months": 24,
    "monthly_charges": 79.99,
    "total_charges": 1920.00,
    "num_support_calls": 3
  }'
```

---

### 📤 Response

```json
{
  "churn": 1,
  "churn_probability": 0.73
}
```
### ✅ What does `churn: 1` mean?

* Your model is doing **binary classification**

  * `1` → Customer **will churn (leave)**
  * `0` → Customer **will NOT churn**
* This is standard in churn ML models

👉 So:

```
"churn": 1
```

➡️ Model predicts this customer is **likely to leave**

---

## 🛠 Troubleshooting

Flow reference:

```text
DVC → KServe → Kubernetes → GitHub Actions → ArgoCD
```

---
