# Hugging Face Transformers

> Model-definition framework for state-of-the-art ML models in text/vision/audio/video with inference and training

| Field | Value |
|-------|-------|
| Group | Inference Serving |
| Type | SDK |
| Open Source | Yes |
| GitHub | [https://github.com/huggingface/transformers](https://github.com/huggingface/transformers) |
| Stars | 156821 |
| Documentation | [Official Docs](https://huggingface.co/docs/transformers/en/index) |

## Overview

Hugging Face Transformers is the model-definition framework for state-of-the-art machine learning models spanning text, computer vision, audio, video, and multimodal tasks, for both inference and training. It centralizes model definitions so they are agreed upon across the ecosystem -- if a model definition is supported in Transformers, it is compatible with the majority of training frameworks (Axolotl, Unsloth, DeepSpeed, FSDP, PyTorch-Lightning), inference engines (vLLM, SGLang, TGI), and adjacent modeling libraries (llama.cpp, MLX). [1]

With over 1M+ model checkpoints on the Hugging Face Hub, Transformers provides the Pipeline API for easy inference, the Trainer API for training, and the `generate` method for fast text generation with LLMs and VLMs. Every model is implemented from three main classes: configuration, model, and preprocessor. [1]

## Core Concepts

### Pipeline

The Pipeline API is the simplest inference interface, supporting many ML tasks with a single function call. Pipelines handle tokenization, model inference, and postprocessing automatically:

```python
from transformers import pipeline

classifier = pipeline("sentiment-analysis")
result = classifier("I love this product!")
# [{'label': 'POSITIVE', 'score': 0.9998}]
```

Supported tasks include text generation, text classification, question answering, summarization, translation, image classification, object detection, automatic speech recognition, and more. [1]

### AutoModel Classes

AutoModel classes automatically detect and load the correct model architecture based on the model name or path:

- `AutoModelForCausalLM` -- Causal language models (GPT, Llama, Mistral)
- `AutoModelForSequenceClassification` -- Text classification
- `AutoModelForTokenClassification` -- Named entity recognition
- `AutoModelForQuestionAnswering` -- Extractive QA
- `AutoModelForSeq2SeqLM` -- Encoder-decoder models (T5, BART)
- `AutoModel` -- Base model without task head [1]

### Tokenizer

Tokenizers convert text to model-compatible token IDs and back. AutoTokenizer loads the correct tokenizer for any model:

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("meta-llama/Llama-3.1-8B-Instruct")
tokens = tokenizer("Hello, world!", return_tensors="pt")
```
[1]

### Trainer

The Trainer API provides a comprehensive training loop supporting mixed precision, gradient accumulation, distributed training, evaluation, and logging. It handles the complexity of training modern transformer models. [1]

### Generate

The `generate` method provides fast text generation for LLMs and VLMs with support for multiple decoding strategies (greedy, sampling, beam search, contrastive), streaming, and KV cache optimization. [1]

## Installation and Setup

### pip Install

```bash
pip install transformers
```

### With Framework Backends

```bash
# PyTorch (most common)
pip install transformers[torch]

# TensorFlow
pip install transformers[tf-cpu]   # CPU only
pip install transformers[tf]       # With GPU support

# JAX/Flax
pip install transformers[flax]
```

### From Source

```bash
pip install git+https://github.com/huggingface/transformers
```

### Additional Dependencies

```bash
# For tokenizers
pip install transformers[sentencepiece]

# For audio
pip install transformers[audio]

# For vision
pip install transformers[vision]
```
[1]

## Architecture

### Design Principles

1. **Three classes per model** -- Configuration (hyperparameters), Model (architecture), and Preprocessor (tokenizer/feature extractor)
2. **Pretrained models** -- Every model loads pretrained weights for immediate use
3. **Framework agnostic** -- Core model definitions work across PyTorch, TensorFlow, and JAX

### Model Architecture

```
Configuration (config.json)
    └── Model (model weights)
        └── Preprocessor (tokenizer/feature extractor)
```

Each model is self-contained with its configuration, weights, and preprocessing requirements. The Hub stores all three components together. [1]

### Hub Integration

Transformers tightly integrates with the Hugging Face Hub for model discovery, downloading, sharing, and versioning. Models are identified by `organization/model-name` and automatically downloaded on first use. [1]

### Ecosystem Pivot

Transformers serves as the central model definition that other tools build upon:

- **Training**: Axolotl, Unsloth, DeepSpeed, FSDP reference Transformers model definitions
- **Inference**: vLLM, SGLang, TGI use Transformers model architectures
- **Export**: llama.cpp, MLX, ONNX converters read Transformers models [1]

## Key Features and Functionality

### Pipeline API

```python
from transformers import pipeline

# Text generation
generator = pipeline("text-generation", model="meta-llama/Llama-3.1-8B-Instruct")
output = generator("Once upon a time", max_length=50)

# Image classification
classifier = pipeline("image-classification", model="google/vit-base-patch16-224")
result = classifier("image.jpg")

# Automatic speech recognition
transcriber = pipeline("automatic-speech-recognition", model="openai/whisper-large-v3")
text = transcriber("audio.mp3")

# Document question answering
qa = pipeline("document-question-answering", model="impira/layoutlm-document-qa")
answer = qa(image="document.png", question="What is the total?")
```
[1]

### Text Generation with LLMs

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

model_name = "meta-llama/Llama-3.1-8B-Instruct"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name, torch_dtype=torch.bfloat16, device_map="auto")

messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "What is machine learning?"},
]
inputs = tokenizer.apply_chat_template(messages, return_tensors="pt").to(model.device)
outputs = model.generate(inputs, max_new_tokens=256)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```
[1]

### Streaming Generation

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, TextStreamer

model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-3.1-8B-Instruct", device_map="auto")
tokenizer = AutoTokenizer.from_pretrained("meta-llama/Llama-3.1-8B-Instruct")
streamer = TextStreamer(tokenizer)

inputs = tokenizer("Explain quantum computing:", return_tensors="pt").to(model.device)
model.generate(**inputs, streamer=streamer, max_new_tokens=200)
```
[1]

### Training with Trainer

```python
from transformers import AutoModelForSequenceClassification, TrainingArguments, Trainer

model = AutoModelForSequenceClassification.from_pretrained("bert-base-uncased", num_labels=2)

training_args = TrainingArguments(
    output_dir="./results",
    num_train_epochs=3,
    per_device_train_batch_size=16,
    evaluation_strategy="epoch",
    learning_rate=5e-5,
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=train_dataset,
    eval_dataset=eval_dataset,
)
trainer.train()
```
[1]

### Quantization

```python
from transformers import AutoModelForCausalLM, BitsAndBytesConfig

quantization_config = BitsAndBytesConfig(load_in_4bit=True)
model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-3.1-8B-Instruct",
    quantization_config=quantization_config,
    device_map="auto",
)
```
[1]

## Use Cases

### Text Generation

Use Pipeline or AutoModelForCausalLM for chatbots, content generation, code completion, and summarization with pretrained LLMs. [1]

### Computer Vision

Image classification, object detection, image segmentation, and image generation using vision transformer models. [1]

### Audio Processing

Speech recognition, audio classification, and text-to-speech using audio transformer models like Whisper. [1]

### Fine-Tuning

Adapt pretrained models to domain-specific tasks using the Trainer API with custom datasets. [1]

### Feature Extraction

Generate embeddings for semantic search, clustering, and similarity using model hidden states or dedicated embedding models. [1]

## API Reference Summary

### Key Classes

- `pipeline(task, model)` -- Create task-specific inference pipeline
- `AutoModel.from_pretrained(name)` -- Load pretrained model
- `AutoTokenizer.from_pretrained(name)` -- Load pretrained tokenizer
- `AutoConfig.from_pretrained(name)` -- Load model configuration
- `Trainer(model, args, train_dataset)` -- Create training loop
- `TrainingArguments(...)` -- Configure training parameters

### Generation Methods

- `model.generate(inputs, max_new_tokens, temperature, ...)` -- Generate text
- `TextStreamer(tokenizer)` -- Stream generated tokens
- `TextIteratorStreamer(tokenizer)` -- Iterate over generated tokens

### Model Saving/Loading

- `model.save_pretrained(path)` -- Save model locally
- `model.push_to_hub(repo_id)` -- Upload to HuggingFace Hub
- `AutoModel.from_pretrained(path_or_hub_id)` -- Load from local or Hub [1]

## Configuration and Customization

### Model Configuration

- **`torch_dtype`** -- Precision (float32, float16, bfloat16)
- **`device_map`** -- Device placement ("auto", "cpu", "cuda:0")
- **`quantization_config`** -- Quantization settings (BitsAndBytes, GPTQ, AWQ)
- **`attn_implementation`** -- Attention backend ("flash_attention_2", "sdpa")
- **`low_cpu_mem_usage`** -- Reduce CPU memory during loading

### Generation Configuration

- **`max_new_tokens`** -- Maximum generated tokens
- **`temperature`** -- Sampling temperature
- **`top_p`** / **`top_k`** -- Sampling parameters
- **`do_sample`** -- Enable sampling (vs greedy)
- **`num_beams`** -- Beam search width
- **`repetition_penalty`** -- Penalize repeated tokens

### Training Configuration

- **`num_train_epochs`** -- Number of training epochs
- **`per_device_train_batch_size`** -- Batch size per GPU
- **`learning_rate`** -- Optimizer learning rate
- **`fp16`** / **`bf16`** -- Mixed precision training
- **`gradient_accumulation_steps`** -- Effective batch size multiplier [1]

## Integration Patterns

### With Inference Engines (vLLM, SGLang, TGI)

Transformers model definitions are the foundation for inference engines. Models defined in Transformers automatically work with vLLM, SGLang, and TGI.

### With Training Frameworks (Axolotl, Unsloth, DeepSpeed)

Training frameworks build on Transformers models and Trainer for distributed fine-tuning.

### With Export Tools (ONNX, llama.cpp, MLX)

Convert Transformers models to optimized formats for deployment on specific hardware.

### With Hugging Face Hub

Seamless integration for model discovery, downloading, sharing, and versioning.

### With Datasets Library

Hugging Face Datasets integrates with Trainer for efficient data loading and preprocessing.

### With PEFT (Parameter-Efficient Fine-Tuning)

LoRA, QLoRA, and other PEFT methods integrate with Transformers models for efficient fine-tuning.

## Examples

### Sentiment Analysis Pipeline

```python
from transformers import pipeline

classifier = pipeline("sentiment-analysis")
results = classifier([
    "I love this movie!",
    "This was terrible.",
])
for result in results:
    print(f"{result['label']}: {result['score']:.4f}")
```
[1]

### Multi-Turn Chat

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model_name = "meta-llama/Llama-3.1-8B-Instruct"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name, device_map="auto")

messages = [
    {"role": "user", "content": "What is Python?"},
    {"role": "assistant", "content": "Python is a high-level programming language."},
    {"role": "user", "content": "What makes it popular?"},
]
inputs = tokenizer.apply_chat_template(messages, return_tensors="pt").to(model.device)
outputs = model.generate(inputs, max_new_tokens=200)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```
[1]

## Limitations and Considerations

- **Memory requirements** -- Large models require significant GPU memory; quantization or device_map="auto" helps
- **Inference speed** -- Native Transformers inference is slower than optimized engines (vLLM, TGI); use Transformers for prototyping, optimized engines for production
- **Model compatibility** -- Not all model architectures are supported; new models may need community contributions
- **Framework coupling** -- While supporting PyTorch, TensorFlow, and JAX, the majority of models are PyTorch-only
- **API complexity** -- The library has a large surface area; many ways to accomplish the same task
- **Breaking changes** -- Major version updates may change APIs; pin versions for production [1]

## Changelog Highlights

- **v5.x** -- Current major version with latest model architectures
- **1M+ models** -- Over one million model checkpoints on HuggingFace Hub
- **Pipeline API** -- Simplified inference for dozens of tasks
- **Trainer API** -- Comprehensive training with mixed precision and distributed support
- **generate()** -- Optimized text generation with streaming and KV cache
- **Flash Attention 2** -- Hardware-accelerated attention computation
- **BitsAndBytes** -- 4-bit and 8-bit quantization for memory reduction
- **Chat templates** -- Standardized chat formatting across models
- **Vision/Audio/Video** -- Multimodal model support beyond text
- **Ecosystem pivot** -- Central model definition for training and inference tools [1]

## Citations

- [1] Hugging Face Transformers Documentation - <https://huggingface.co/docs/transformers/index>
