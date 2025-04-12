# 🚀 Deploying a Hugging Face Model in OpenSearch (Step-by-Step Guide)

This guide walks you through the complete process of deploying a **Hugging Face sentence transformer model** in **OpenSearch** using the **ML Commons Plugin**.

---

## 📘 Prerequisites

- OpenSearch cluster with ML Commons plugin enabled.
- API access to the cluster (e.g., using `curl`, Postman, or OpenSearch Dashboards Dev Tools).
- Optional: A cluster with **no dedicated ML nodes**.

---

## 🔧 Step 0 (Optional): Update Cluster Settings

If your cluster does not have dedicated ML nodes, allow inference on any node:

```http
PUT _cluster/settings
{
  "persistent": {
    "plugins.ml_commons.only_run_on_ml_node": false
  }
}
```

---

## 📦 Step 1: Register a Model Group

Model groups help organize models. Start by creating a group for sentence transformers:

```http
POST /_plugins/_ml/model_groups/_register
{
  "name": "sentence_transformer_group",
  "description": "A model group for sentence transformer models"
}
```

**📌 Note:** Save the `model_group_id` from the response for the next step.

---

## 📥 Step 2: Register the Model

Now register a Hugging Face model under the created group. Replace `<model_group_id_from_step_1>`:

```http
POST /_plugins/_ml/models/_register
{
  "name": "huggingface/sentence-transformers/msmarco-distilbert-base-tas-b",
  "version": "1.0.2",
  "model_group_id": "<model_group_id_from_step_1>",
  "model_format": "TORCH_SCRIPT"
}
```

**📌 Note:** Save the `task_id` returned in the response.

---

## 🕵️ Step 3: Check Model Registration Status

Track the model registration task:

```http
GET /_plugins/_ml/tasks/<task_id>
```

✅ **Wait until** the task `state` is `"COMPLETED"`.

**📌 Note:** Save the `model_id` returned in the response.

---

## 🚀 Step 4: Deploy the Model

Deploy the registered model using its `model_id`:

```http
POST /_plugins/_ml/models/<model_id>/_deploy
```

**📌 Note:** Save the `task_id` from the response to monitor deployment.

---

## 🕵️ Step 5: Check Deployment Status

Track the model deployment task:

```http
GET /_plugins/_ml/tasks/<task_id>
```

✅ **Wait until** the task `state` is `"COMPLETED"`.

---

## 🧪 Step 6: Test the Model with Predict API

Now you're ready to use the model for inference:

```http
POST /_plugins/_ml/_predict/text_embedding/<model_id>
{
  "text_docs": ["today is sunny"],
  "return_number": true,
  "target_response": ["sentence_embedding"]
}
```

✅ **Expected Response:** A JSON object containing the **text embedding vector** for your input.

---

## ✅ Summary

| Step | Description | Key Output |
|------|-------------|------------|
| 0    | (Optional) Allow inference on non-ML nodes | - |
| 1    | Register model group | `model_group_id` |
| 2    | Register model | `task_id` |
| 3    | Check registration | `model_id` |
| 4    | Deploy model | `task_id` |
| 5    | Check deployment | - |
| 6    | Predict / Embed text | Embeddings |