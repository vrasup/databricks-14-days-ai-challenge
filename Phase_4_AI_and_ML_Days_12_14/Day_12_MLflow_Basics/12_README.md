
# DAY 12 (20/01/26) – MLflow Basics

## Learn

* Understanding **MLflow components**:

  * **Tracking**: log experiments, parameters, metrics
  * **Models**: save and load ML models
  * **Registry**: version control and stage models
* **Experiment tracking**: organize runs and track performance
* **Model logging**: log parameters, metrics, artifacts
* **MLflow UI**: visualize experiments, compare runs, inspect metrics

---

## Tasks Completed

* Trained baseline ML models using **scikit-learn**
* Logged **parameters, metrics, and models** to MLflow
* Created and organized **experiments** for multiple model runs
* Used **MLflow UI** to compare performance across runs
* Identified **best model** based on selected metrics (ROC-AUC, F1 score)

---

## Hands-On Practice

```python
import mlflow
import mlflow.sklearn
from sklearn.ensemble import GradientBoostingClassifier, RandomForestClassifier
from sklearn.metrics import roc_auc_score, f1_score

models = {
    "random_forest": RandomForestClassifier(n_estimators=100, max_depth=5),
    "gradient_boosting": GradientBoostingClassifier(n_estimators=150, max_depth=5, learning_rate=0.1)
}

for name, model in models.items():
    with mlflow.start_run(run_name=f"{name}_model"):
        mlflow.log_param("model_type", name)

        model.fit(X_train, y_train)
        y_pred = model.predict(X_test)
        y_proba = model.predict_proba(X_test)[:,1]

        roc = roc_auc_score(y_test, y_proba)
        f1 = f1_score(y_test, y_pred)

        mlflow.log_metric("roc_auc", roc)
        mlflow.log_metric("f1_score", f1)
        mlflow.sklearn.log_model(model, "model")

        print(f"{name}: ROC-AUC = {roc:.4f}, F1 = {f1:.4f}")
```

---

## **Output**

* **Random Forest**

  * ROC-AUC: 0.6323
  * F1: 0.0513
  * Threshold: 0.6

* **Gradient Boosting**

  * ROC-AUC: 0.6416
  * F1: 0.0560
  * Threshold: 0.6

* **MLflow UI**

  * Visualized runs
  * Compared metrics side by side
  * Identified **overall best model**: Gradient Boosting (ROC-AUC 0.6416, F1 0.0560)

---

## **Key Takeaways**

* MLflow allows **structured experiment tracking** for reproducibility
* Logging parameters and metrics makes it easy to **compare multiple models**
* Threshold tuning significantly impacts **F1 score and recall** in imbalanced datasets
* The MLflow UI is an invaluable tool for **experiment visibility** and model selection

---

## **Resources**

* [MLflow Documentation](https://docs.databricks.com/aws/en/mlflow)

* [Model Registry](https://docs.databricks.com/aws/en/machine-learning/manage-model-lifecycle)

---

## **Tags & Mentions**

Mentions: @Databricks @Codebasics @IndianDataClub

Hashtag: #DatabricksWithIDC 

---