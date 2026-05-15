# Vertex AI

> Google Cloud unified platform for building and scaling ML/GenAI models

| Field | Value |
|-------|-------|
| Group | Managed AI Platforms |
| Type | API/SDK/Infra |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Documentation | [Official Docs](https://docs.cloud.google.com/vertex-ai/docs) |

## Overview

Vertex AI is Google Cloud's unified platform for building, deploying, and scaling generative AI and machine learning models. It provides access to Model Garden with over 200 models, including Google's foundation models and partner/open-source options, along with TPU and GPU infrastructure for training and serving workloads.

The platform consolidates Google Cloud's ML services into a single environment, covering the full lifecycle from data preparation and model training through evaluation, deployment, and monitoring. It supports both code-free AutoML workflows and fully custom training pipelines, making it accessible to teams with varying levels of ML expertise.

## Core Concepts

- **Model Garden**: a curated catalog of over 200 enterprise-ready models, including Gemini (multimodal), Imagen (image generation), Veo (video generation), and partner models such as Claude, Mistral, and Llama
- **Vertex AI Studio**: an interactive environment for prototyping and deploying generative AI applications, providing prompt design, model tuning, and evaluation tools
- **AutoML**: a code-free approach to training high-quality models on tabular, image, text, and video data without writing training code
- **Custom Training**: full control over the training process using custom containers, frameworks, and hyperparameter tuning
- **Feature Store**: a centralized repository for managing, sharing, and serving ML feature data across training and serving pipelines
- **Model Registry**: a versioned catalog for managing model lifecycle stages from experimentation through production deployment
- **Vertex AI Pipelines**: orchestration service for automating and reproducing ML workflows as directed acyclic graphs
- **Grounding**: a mechanism that connects model outputs to external data sources and APIs, reducing hallucinations and improving factual accuracy
- **Training Clusters**: dedicated large-scale compute resources for distributed training workloads

## Architecture

Vertex AI operates as a managed cloud service within the Google Cloud ecosystem. Its architecture spans several layers:

- **Compute Layer**: provides managed TPU and GPU clusters for training and inference, with support for autoscaling and dedicated training clusters
- **Model Layer**: hosts Model Garden with pre-trained foundation models, custom-trained models, and partner models, all accessible through a unified API surface
- **Pipeline Layer**: Vertex AI Pipelines orchestrate multi-step ML workflows, integrating data preparation, training, evaluation, and deployment into reproducible pipelines
- **Serving Layer**: online endpoints handle real-time inference with autoscaling, while batch prediction processes large datasets asynchronously
- **Data Layer**: integrates with Cloud Storage, BigQuery, and Feature Store for data ingestion, feature management, and dataset versioning
- **Monitoring Layer**: tracks model performance in production, detecting data skew and drift to signal when retraining is needed

All components communicate through Google Cloud's internal network and are accessible via REST APIs, gRPC, and the Python SDK.

## Key Features and Functionality

### Generative AI

- **Vertex AI Studio**: prototype and deploy generative AI applications with prompt design and evaluation tools
- **Model Garden**: access to over 200 enterprise-ready models spanning text, image, video, and multimodal capabilities
- **Customization**: Supervised Fine-Tuning (SFT) and Parameter-Efficient Fine-Tuning (PEFT) for adapting foundation models to specific domains
- **Grounding and Function Calling**: connect model outputs to external data sources and APIs for improved accuracy
- **Safety Features**: built-in content filtering and responsible AI tooling

### ML Workflow

- **Data Preparation**: Workbench notebooks with direct Cloud Storage and BigQuery integration
- **Training**: AutoML for code-free training or Custom Training with full framework control
- **Experiments**: Vertex AI Experiments for tracking and comparing training runs
- **Evaluation**: model metrics, fairness assessments, and evaluation pipelines
- **Serving**: online and batch inference with prebuilt or custom containers
- **Monitoring**: production model monitoring with data skew and drift detection

### MLOps

- **Vertex AI Pipelines**: automate and reproduce ML workflows
- **Model Registry**: lifecycle management, versioning, and deployment tracking
- **Feature Store**: centralized feature management and serving
- **Vertex ML Metadata**: track artifact metadata across the ML lifecycle
- **Ray on Vertex AI**: scale workloads using the open-source Ray framework on managed infrastructure

## Use Cases

- **Enterprise Generative AI Applications**: building chatbots, content generation systems, and summarization tools using foundation models from Model Garden
- **Custom Model Training**: training domain-specific models on proprietary data using AutoML or custom training pipelines
- **Model Fine-Tuning**: adapting foundation models to specialized tasks through SFT or PEFT
- **Real-Time Inference**: deploying models behind autoscaling endpoints for low-latency predictions in production applications
- **Batch Processing**: running large-scale offline predictions on datasets stored in Cloud Storage or BigQuery
- **MLOps Automation**: building reproducible ML pipelines that automate the full lifecycle from data ingestion to model deployment
- **Multimodal Applications**: leveraging Gemini and other multimodal models for applications that process text, images, video, and audio together

## API Reference Summary

### Model Deployment and Prediction

```python
from google.cloud import aiplatform

# Deploy a model to an endpoint
model = aiplatform.Model(model_name="projects/PROJECT/locations/LOCATION/models/MODEL_ID")
endpoint = model.deploy(
    machine_type="n1-standard-4",
    min_replica_count=1,
    max_replica_count=3,
)

# Online prediction
prediction = endpoint.predict(instances=[{"input": "sample data"}])
```

### Generative AI

```python
import vertexai
from vertexai.generative_models import GenerativeModel

vertexai.init(project="your-project-id", location="us-central1")

model = GenerativeModel("gemini-2.0-flash")
response = model.generate_content("Explain quantum computing in simple terms.")
print(response.text)
```

### AutoML Training

```python
from google.cloud import aiplatform

dataset = aiplatform.TabularDataset.create(
    display_name="my-dataset",
    bq_source="bq://project.dataset.table",
)

job = aiplatform.AutoMLTabularTrainingJob(
    display_name="my-automl-job",
    optimization_prediction_type="classification",
)

model = job.run(
    dataset=dataset,
    target_column="label",
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1,
)
```

### Custom Training

```python
from google.cloud import aiplatform

job = aiplatform.CustomTrainingJob(
    display_name="my-custom-job",
    script_path="train.py",
    container_uri="us-docker.pkg.dev/vertex-ai/training/tf-gpu.2-14:latest",
    requirements=["transformers", "datasets"],
)

model = job.run(
    replica_count=1,
    machine_type="n1-standard-8",
    accelerator_type="NVIDIA_TESLA_V100",
    accelerator_count=1,
)
```

### Batch Prediction

```python
model = aiplatform.Model(model_name="projects/PROJECT/locations/LOCATION/models/MODEL_ID")

batch_prediction_job = model.batch_predict(
    job_display_name="my-batch-job",
    gcs_source="gs://bucket/input.jsonl",
    gcs_destination_prefix="gs://bucket/output/",
    machine_type="n1-standard-4",
)
```

## Configuration and Customization

### Endpoint Configuration

- **Machine Type**: select compute resources for serving (e.g., `n1-standard-4`, `n1-highmem-8`)
- **Accelerators**: attach GPUs or TPUs for inference (e.g., `NVIDIA_TESLA_T4`, `NVIDIA_TESLA_V100`)
- **Autoscaling**: configure minimum and maximum replica counts based on traffic
- **Traffic Split**: distribute traffic across multiple deployed model versions

### Training Configuration

- **Compute Resources**: specify machine type, accelerator type, and accelerator count
- **Distributed Training**: configure worker pools for multi-node training
- **Hyperparameter Tuning**: define search spaces and optimization objectives
- **Timeout and Scheduling**: set maximum training duration and scheduling priorities

### Network Configuration

- **VPC Peering**: deploy endpoints within a VPC for private network access
- **Private Endpoints**: restrict access to internal traffic only
- **Service Accounts**: assign specific IAM service accounts for fine-grained access control

## Integration Patterns

- **BigQuery**: read training data directly from BigQuery tables and write predictions back to BigQuery
- **Cloud Storage**: store training data, model artifacts, and batch prediction results in GCS buckets
- **Cloud Functions / Cloud Run**: trigger predictions or pipeline runs in response to events
- **Vertex AI Pipelines + Kubeflow**: define ML workflows as pipeline components using the Kubeflow Pipelines SDK
- **Feature Store**: serve precomputed features to both training jobs and online prediction endpoints
- **Ray on Vertex AI**: distribute training and data processing workloads across managed Ray clusters
- **Logging and Monitoring**: integrate with Cloud Logging and Cloud Monitoring for observability

## Examples

### Text Generation with Gemini

```python
import vertexai
from vertexai.generative_models import GenerativeModel

vertexai.init(project="my-project", location="us-central1")

model = GenerativeModel("gemini-2.0-flash")
response = model.generate_content(
    "Write a product description for a noise-canceling headphone."
)
print(response.text)
```

### Image Generation with Imagen

```python
import vertexai
from vertexai.preview.vision_models import ImageGenerationModel

vertexai.init(project="my-project", location="us-central1")

model = ImageGenerationModel.from_pretrained("imagen-3.0-generate-002")
images = model.generate_images(
    prompt="A mountain landscape at sunset with a lake in the foreground",
    number_of_images=1,
)
images[0].save("output.png")
```

### Pipeline Definition

```python
from kfp.v2 import dsl, compiler

@dsl.pipeline(name="my-ml-pipeline")
def ml_pipeline():
    preprocess_task = preprocess_op(input_data="gs://bucket/raw-data")
    train_task = train_op(training_data=preprocess_task.output)
    deploy_task = deploy_op(model=train_task.output)

compiler.Compiler().compile(
    pipeline_func=ml_pipeline,
    package_path="pipeline.json",
)
```

## Limitations and Considerations

- **Vendor Lock-In**: tightly coupled with Google Cloud infrastructure; migrating trained models and pipelines to other clouds requires significant effort
- **Cost**: GPU/TPU compute, model serving endpoints, and storage costs can accumulate quickly, particularly for large-scale training and always-on endpoints
- **Regional Availability**: not all features and model types are available in every Google Cloud region
- **Cold Start Latency**: endpoints scaled to zero replicas incur cold start delays when receiving the first request after idle periods
- **Quotas and Limits**: GPU/TPU quotas are subject to availability and may require requesting quota increases for large workloads
- **Closed Source**: the platform itself is proprietary; users depend on Google Cloud for updates, pricing changes, and feature availability
- **Model Garden Licensing**: partner and open-source models in Model Garden may carry their own licensing terms separate from the Vertex AI service

## Changelog Highlights

Vertex AI is a managed cloud service with continuous updates. Notable capabilities include the introduction of Gemini multimodal models, Model Garden expansion to over 200 models, Grounding with Google Search, Ray on Vertex AI for distributed workloads, and ongoing additions of partner models such as Claude and Mistral. Consult the official release notes for the most current changes.

## Citations

- [1] Vertex AI Documentation - https://docs.cloud.google.com/vertex-ai/docs
- [2] Vertex AI Introduction - https://docs.cloud.google.com/vertex-ai/docs/start/introduction-unified-platform

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- vertex ai
- google cloud ai
- model garden
- vertex ai studio
- gemini
- imagen
- veo
- automl
- custom training
- feature store
- model registry
- vertex ai pipelines
- kubeflow pipelines
- grounding
- supervised fine-tuning
- peft on vertex
- ray on vertex ai
- batch prediction
- online endpoint
- tpu
- v100
- t4 inference
- vpc peering
- private endpoint
- bigquery integration
- gcs integration
- workbench notebooks
- mlops
- data skew detection
- drift detection

### Verb-Noun Tasks

- Browse Model Garden for Gemini, Claude, Mistral, or Llama
- Generate text with `GenerativeModel("gemini-2.0-flash")`
- Generate images with Imagen via `ImageGenerationModel`
- Train a tabular AutoML model from a BigQuery table
- Run a custom training job with a TF/PyTorch container and accelerators
- Deploy a model to a Vertex AI online endpoint with autoscaling
- Submit a batch prediction over GCS inputs
- Fine-tune a foundation model with SFT or PEFT
- Track experiments with Vertex AI Experiments
- Compose multi-step ML pipelines with the KFP DSL
- Serve features from Feature Store to training and online endpoints
- Ground model outputs against external sources and Google Search
- Distribute workloads with Ray on Vertex AI

### User Intent Phrases

- How do I access Gemini, Imagen, and Veo through one Google Cloud SDK?
- How do I deploy a custom PyTorch model on Google Cloud with autoscaling?
- How do I train an AutoML model on BigQuery data without writing training code?
- How do I run batch prediction over millions of records in GCS?
- How do I fine-tune Gemini for my domain?
- How do I build a Kubeflow pipeline that retrains weekly?
- How do I monitor data drift on a production endpoint?
- Where do I find Claude or Llama on Google Cloud?
- How do I serve features consistently to training and serving?
- How do I detect when a deployed model needs retraining?

### Problem Statements

- We are a GCP shop and need ML training, serving, and monitoring under one billing relationship.
- We need TPU access for training jobs.
- Our data lives in BigQuery and we want to train directly against it.
- We need an MLOps platform with pipelines, registry, and feature store, not just inference.
- We want to ground LLM responses against Google Search to reduce hallucination.
- We need data-skew and drift detection on production endpoints.
- We need a managed Ray cluster on Google infrastructure.

### When to Pick This

- Pick this when your data, identity, and billing are already on Google Cloud (vs AWS Bedrock when you are on AWS).
- Pick this when you need TPUs in addition to GPUs.
- Pick this when AutoML for tabular/image/text/video is a primary requirement.
- Pick this when you want a single platform spanning data prep, training, registry, pipelines, serving, and monitoring (vs renting GPU pods on RunPod/Vast/Modal).
- Pick this when grounding with Google Search and Gemini multimodal models are core capabilities.
- Pick this when managed Ray clusters on the same platform are valuable.
- Pick this when Kubeflow Pipelines DSL is your preferred orchestration.

### Related Terms and Aliases

- gcp ai platform
- google managed ml
- foundation model platform
- google gen ai studio
- vertex ai endpoints
- gemini api on gcp
- gcp ml pipelines
- google ai infrastructure
- managed kubeflow
- gcp mlops
