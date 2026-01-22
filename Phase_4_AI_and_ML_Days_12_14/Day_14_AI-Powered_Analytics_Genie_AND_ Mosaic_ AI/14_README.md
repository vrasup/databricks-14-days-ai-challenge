# **DAY 14 (22/01/26) – AI-Powered Analytics: Genie & Mosaic AI**

## **Learn**

* Understanding **AI-powered analytics** in Databricks
* **Databricks Genie:**

  * Natural language → SQL analytics
  * Lowering the barrier for data access by non-technical users
* **Mosaic AI concepts:**

  * Using **pretrained foundation models** for inference
  * Applying GenAI without training custom models
* **MLflow for GenAI workflows:**

  * Logging AI parameters, metrics, and outputs
  * Ensuring reproducibility and governance of AI-assisted analytics
* Running **Transformer-based NLP models on CPU** in constrained environments

---

## **Tasks Completed**

* Documented **Genie-style natural language queries** for analytical use cases
* Performed **sentiment analysis** using a pretrained **Transformer model**
* Integrated **Hugging Face Transformers** in Databricks Free Notebook Edition
* Logged AI workflow details, metrics, and prediction artifacts to **MLflow**
* Interpreted AI outputs and generated **AI-powered insights**

---

## **Hands-On Practice**

```python
import torch
from transformers import pipeline
import mlflow

# Load pretrained sentiment analysis pipeline (CPU inference)
sentiment_pipeline = pipeline(
    "sentiment-analysis",
    model="distilbert-base-uncased-finetuned-sst-2-english"
)

texts = [
    "checkout was smooth and fast",
    "product quality is very bad",
    "customer support was okay",
    "highly recommend this product",
    "Hello"
]

# Run sentiment analysis
results = sentiment_pipeline(texts)

# Log AI workflow to MLflow
mlflow.set_experiment("/Shared/day14_ai_powered_analytics")

with mlflow.start_run(run_name="day14_transformers_sentiment"):
    mlflow.log_param("model", "distilbert-base-uncased-finetuned-sst-2-english")
    mlflow.log_param("ai_framework", "huggingface_transformers")
    mlflow.log_param("device", "cpu")
    mlflow.log_metric("num_text_samples", len(texts))

    predictions = {
        text: {
            "label": res["label"],
            "score": float(res["score"])
        }
        for text, res in zip(texts, results)
    }

    mlflow.log_dict(predictions, "sentiment_predictions.json")

for text, result in zip(texts, results):
    print(f"{text} → {result['label']} ({result['score']:.2f})")
```

---

## **Output**

* **Transformer-based Sentiment Analysis (CPU)**

  * checkout was smooth and fast → POSITIVE (1.00)
  * product quality is very bad → NEGATIVE (1.00)
  * customer support was okay → POSITIVE (1.00)
  * highly recommend this product → POSITIVE (1.00)
  * Hello → POSITIVE (1.00)

* **MLflow Tracking**

  * Parameters logged:

    * Model name
    * AI framework
    * Execution device (CPU)
  * Metrics logged:

    * Number of text samples
  * Artifacts:

    * `sentiment_predictions.json` containing AI predictions

---

## **Key Takeaways**

* **Pretrained Transformer models** enable fast AI adoption without training
* Binary sentiment models classify inputs as **positive or negative only**, so neutral text may be labeled as positive
* Databricks supports **GenAI inference workflows** even in CPU-only environments
* **MLflow** brings reproducibility, traceability, and governance to AI-powered analytics
* Natural language analytics tools like Genie improve accessibility of insights

---

## **Resources**

* [Databricks Genie](https://www.youtube.com/watch?v=naFraZ1kMi8)

* [Mosaic AI Documentation](https://docs.databricks.com/generative-ai/)

* [Hugging Face Transformers](https://huggingface.co/docs/transformers)

---

## **Tags & Mentions**

Mentions: @Databricks @Codebasics @IndianDataClub

Hashtag: #DatabricksWithIDC

---


