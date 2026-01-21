# DAY 13 (21/01/26) – Model Comparison & Feature Engineering

## Learn

* Comparing multiple **machine learning models** systematically
* **Metrics & evaluation** for imbalanced datasets:
    * Accuracy, ROC-AUC, F1-score
    * Threshold tuning to improve F1
* **MLflow tracking:**
    * Logging parameters, metrics, and models for reproducibility
    * Comparing runs and identifying best model
* **Spark ML:**
    * Scalable model training on sampled Spark DataFrames
    * Logging Spark ML models and artifacts to MLflow
---

## Tasks Completed

* Trained multiple **scikit-learn models:** Logistic Regression, Random Forest, Gradient Boosting
* Tuned classification thresholds to improve **F1-score**
* Logged **parameters, metrics, and models** in MLflow for all runs
* Visualized **feature importances** and confusion matrices
* Trained **Spark ML Random Forest** on sampled data and logged the model
* Compared models systematically and **identified best-performing model**

---

## Hands-On Practice
```python
import mlflow
import mlflow.sklearn
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.metrics import accuracy_score, roc_auc_score, f1_score

models = {
    "logistic_regression": LogisticRegression(max_iter=200, class_weight="balanced"),
    "random_forest": RandomForestClassifier(n_estimators=200, max_depth=5, class_weight="balanced"),
    "gradient_boosting": GradientBoostingClassifier(n_estimators=400, max_depth=10, learning_rate=0.2)
}

for name, model in models.items():
    with mlflow.start_run(run_name=f"{name}_model"):
        mlflow.log_param("model_type", name)

        model.fit(X_train, y_train)
        y_proba = model.predict_proba(X_test)[:,1]
        
        # Threshold tuning (example)
        thresholds = [0.1,0.2,0.3,0.4,0.5]
        best_f1 = -1
        best_threshold = 0.5
        for t in thresholds:
            y_pred = (y_proba > t).astype(int)
            f1 = f1_score(y_test, y_pred)
            if f1 > best_f1:
                best_f1 = f1
                best_threshold = t

        y_pred = (y_proba > best_threshold).astype(int)
        acc = accuracy_score(y_test, y_pred)
        roc = roc_auc_score(y_test, y_proba)

        mlflow.log_metric("accuracy", acc)
        mlflow.log_metric("roc_auc", roc)
        mlflow.log_metric("f1_score", best_f1)
        mlflow.log_param("best_threshold", best_threshold)

        mlflow.sklearn.log_model(model, "model")
        print(f"{name}: Accuracy = {acc:.4f}, ROC-AUC = {roc:.4f}, F1 = {best_f1:.4f}, Threshold = {best_threshold}")

```
---

## **Output**

* **Logistic Regression**

    * Accuracy: 0.6021
    * ROC-AUC: 0.6220
    * F1: 0.0410
    * Threshold: 0.5

* **Random Forest**

    * Accuracy: 0.6134
    * ROC-AUC: 0.6432
    * F1: 0.0425
    * Threshold: 0.5

* **Gradient Boosting**

    * Accuracy: 0.6281
    * ROC-AUC: 0.6405
    * F1: 0.0378
    * Threshold: 0.4

* **Spark Random Forest (sampled)**

    * Accuracy: 0.6150
    * ROC-AUC: 0.6420
    * F1: 0.0410

* **MLflow UI**

    * Compared runs across models
    * Identified best overall model: Random Forest (ROC-AUC 0.6432, F1 0.0425)

---

## **Key Takeaways**

* Comparing multiple models systematically helps identify **best-performing algorithm**
* **Threshold tuning** can significantly improve F1-score for imbalanced datasets
* MLflow ensures **reproducibility** and easy comparison across runs
* Spark ML enables **scalable model training** with familiar workflow
* Feature importance and metrics provide **insight into model decisions**

---

## **Resources**

* [Spark ML](https://spark.apache.org/docs/latest/ml-classification-regression.html)

---

## **Tags & Mentions**

Mentions: @Databricks @Codebasics @IndianDataClub

Hashtag: #DatabricksWithIDC 

---