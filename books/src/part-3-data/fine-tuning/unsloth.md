# Unsloth

> Fine-tuning and reinforcement learning framework that trains LLMs 2x faster with 70% less VRAM

| Field | Value |
|-------|-------|
| Name | Unsloth |
| Group | Fine-tuning |
| Type | SDK |
| Open Source | yes |
| GitHub | [unslothai/unsloth](https://github.com/unslothai/unsloth) |
| Stars | 53240 |
| Docs | [unsloth.ai/docs](https://unsloth.ai/docs) |

## Overview

Unsloth is an open-source framework for fine-tuning and reinforcement learning of large language models that achieves 2x faster training speeds with 70% less VRAM compared to standard implementations. It supports 500+ models including text, vision, text-to-speech, and embedding models, with optimized kernels written in Triton for memory-efficient training [1].

The framework collaborates directly with model teams behind gpt-oss, Qwen3, Llama 4, Mistral, Gemma, and Phi-4, fixing critical bugs that improve model accuracy. It provides a streamlined pipeline from training through evaluation to deployment with Ollama, llama.cpp, vLLM, and other inference engines [1].

Unsloth claims 0% loss in accuracy — no approximation methods are used, all computations are exact. The VRAM savings come from optimized Triton kernels, smart gradient checkpointing, and memory-efficient loss calculations rather than quantization or approximation during training [1].

The framework supports Linux, Windows, NVIDIA GPUs (CUDA Capability 7.0+), AMD GPUs, and Intel GPUs. Python 3.13 is supported [6].

## Core Concepts

### FastLanguageModel

The primary API is `FastLanguageModel`, which handles model loading, adapter configuration, inference optimization, and model saving. It wraps Hugging Face Transformers and PEFT with optimized Triton kernels [2]:

- `FastLanguageModel.from_pretrained()`: Loads models with 4-bit, 8-bit, 16-bit, or full precision
- `FastLanguageModel.get_peft_model()`: Applies LoRA adapters with Unsloth-optimized kernels
- `FastLanguageModel.for_inference()`: Enables 2x faster native inference after training

### Training Methods

Unsloth supports multiple training approaches [2]:

- **QLoRA (4-bit)**: Default recommended mode; loads model quantized to 4-bit, trains LoRA adapters in 16-bit
- **LoRA (16-bit)**: Full 16-bit LoRA fine-tuning with ~4x more VRAM than QLoRA
- **Full Fine-Tuning**: Updates all model parameters; most compute-intensive
- **Continued Pretraining**: Extend base model training on domain-specific corpora

### Reinforcement Learning

Unsloth provides the most memory-efficient RL implementation, using up to 90% less VRAM than standard implementations with Flash Attention 2. Supported methods [3]:

- **GRPO (Group Relative Policy Optimization)**: DeepSeek's method that removes both value and reward models, using statistical sampling across multiple outputs to estimate advantages via Z-score standardization
- **PPO (Proximal Policy Optimization)**: Traditional three-component system with generating policy, reference policy, and value model
- **RLHF (Reinforcement Learning from Human Feedback)**: Training agents to produce outputs rated useful by human evaluators
- **RLVR (Reinforcement Learning with Verifiable Rewards)**: Rewards based on tasks with verifiable solutions (math, code)
- **GSPO, DR-GRPO**: Additional variants accessible via `GRPOConfig` parameters

### Dynamic 2.0 GGUFs

Unsloth's Dynamic 2.0 quantization system intelligently varies quantization types per layer and per model, rather than applying uniform quantization. It uses a calibration dataset of 1.5+ million hand-curated tokens optimized for conversational performance. Available formats include IQ1_S through Q5_1 [5].

### Vision Fine-Tuning

Unsloth supports fine-tuning Vision-Language Models (VLMs) including Qwen3-VL, Gemma 3, Llama 3.2 Vision, and Qwen2.5 VL. Users can selectively fine-tune vision layers, language layers, attention modules, or MLP modules independently [7].

## Architecture

```
┌─────────────────────────────────────────────────────┐
│              FastLanguageModel API                    │
│  from_pretrained │ get_peft_model │ for_inference    │
└──────────────────────┬──────────────────────────────┘
                       │
          ┌────────────┼────────────┐
          v            v            v
┌──────────────┐ ┌──────────┐ ┌──────────────────┐
│  Model       │ │  LoRA    │ │  Optimized       │
│  Loading     │ │  Adapter │ │  Triton Kernels  │
│              │ │          │ │                   │
│  4/8/16-bit  │ │  PEFT    │ │  Memory-efficient│
│  HF Hub      │ │  QLoRA   │ │  loss functions  │
│  BitsAndBytes│ │  rsLoRA  │ │  Gradient ckpt   │
└──────────────┘ └──────────┘ └──────────────────┘
                       │
          ┌────────────┼────────────┐
          v            v            v
┌──────────────┐ ┌──────────┐ ┌──────────────────┐
│  SFTTrainer  │ │  GRPO    │ │  DPO/ORPO/PPO    │
│  (TRL)       │ │  Trainer │ │  Trainers        │
│              │ │          │ │                   │
│  Supervised  │ │  Reward  │ │  Preference       │
│  Fine-tuning │ │  funcs   │ │  Alignment        │
└──────────────┘ └──────────┘ └──────────────────┘
                       │
                       v
┌─────────────────────────────────────────────────────┐
│                  Export / Deploy                      │
│  GGUF (Ollama, llama.cpp, LM Studio)                │
│  vLLM (FP8, AWQ) │ SGLang │ HF Hub │ LoRA merge    │
└─────────────────────────────────────────────────────┘
```

Unsloth sits between the Hugging Face ecosystem (Transformers, PEFT, TRL) and optimized Triton kernels. The framework intercepts standard training operations and replaces them with memory-efficient implementations while maintaining mathematical equivalence [1][2].

## Key Features

- **2x Faster Training**: Optimized Triton kernels for training with zero accuracy loss
- **70-90% Less VRAM**: Memory-efficient implementations for both SFT and RL training
- **500+ Model Support**: Text, vision, TTS, embedding, and MoE models from Llama, Qwen, Gemma, DeepSeek, Mistral, Phi, and more
- **QLoRA/LoRA/Full Fine-Tuning**: 4-bit, 8-bit, 16-bit, and full precision training modes
- **Reinforcement Learning**: GRPO, PPO, RLHF, RLVR, GSPO, DR-GRPO with up to 90% VRAM reduction
- **Vision Fine-Tuning**: Selective layer fine-tuning for VLMs (vision, language, attention, MLP)
- **Text-to-Speech Fine-Tuning**: TTS model training support
- **Embedding Fine-Tuning**: Train custom embedding models
- **Dynamic 2.0 GGUFs**: Intelligent per-layer quantization with custom calibration datasets
- **Ultra Long Context RL**: 500K+ context length fine-tuning support
- **Multi-GPU Training**: Distributed training across multiple GPUs
- **Faster MoE Training**: 12x faster Mixture-of-Experts training with less VRAM
- **GGUF Export**: Direct conversion for Ollama, llama.cpp, and LM Studio deployment
- **vLLM Integration**: Enterprise deployment with FP8/AWQ quantization and LoRA hot-swapping
- **2x Faster Inference**: Native accelerated inference via `for_inference()`
- **Chat Templates**: Flexible template system for conversation formatting
- **Ready-to-Use Notebooks**: Google Colab notebooks for all supported models

## Use Cases

- **Instruction Tuning**: Fine-tune base models to follow instructions using SFTTrainer with alpaca or chat formats
- **Reasoning Model Training**: Use GRPO to train reasoning capabilities similar to DeepSeek-R1 approach
- **Domain Adaptation**: Continue pretraining on domain-specific data then fine-tune for specialized tasks
- **Medical Imaging**: Fine-tune Llama 3.2 Vision on radiography and other medical imaging datasets
- **Document Analysis**: Train VLMs for handwriting-to-LaTeX conversion and document understanding
- **Local LLM Deployment**: Train, quantize to GGUF, and deploy via Ollama or llama.cpp for local inference
- **Enterprise Serving**: Fine-tune and deploy via vLLM with LoRA hot-swapping for multi-tenant systems
- **Preference Alignment**: Align models with human preferences using DPO, ORPO, or GRPO
- **Code Generation**: Fine-tune coding models using RL with verifiable rewards from code execution

## API Reference

### Model Loading

```python
from unsloth import FastLanguageModel

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Llama-3.1-8B-bnb-4bit",
    max_seq_length=2048,
    dtype=None,            # Auto-detect; or torch.float16/bfloat16
    load_in_4bit=True,     # QLoRA mode
)
```

### LoRA Configuration

```python
model = FastLanguageModel.get_peft_model(
    model,
    r=16,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                     "gate_proj", "up_proj", "down_proj"],
    lora_alpha=16,
    lora_dropout=0,
    bias="none",
    use_rslora=False,
    use_gradient_checkpointing="unsloth",
)
```

### Supervised Fine-Tuning

```python
from trl import SFTTrainer
from transformers import TrainingArguments

trainer = SFTTrainer(
    model=model,
    tokenizer=tokenizer,
    train_dataset=dataset,
    args=TrainingArguments(
        per_device_train_batch_size=2,
        gradient_accumulation_steps=4,
        max_steps=60,
        learning_rate=2e-4,
        fp16=not torch.cuda.is_bf16_supported(),
        bf16=torch.cuda.is_bf16_supported(),
        output_dir="outputs",
    ),
)
trainer.train()
```

### GRPO Training

```python
from trl import GRPOConfig, GRPOTrainer

training_args = GRPOConfig(
    learning_rate=5e-6,
    num_generations=8,
    max_completion_length=256,
    per_device_train_batch_size=1,
    gradient_accumulation_steps=4,
    output_dir="grpo_outputs",
)

trainer = GRPOTrainer(
    model=model,
    processing_class=tokenizer,
    reward_funcs=[correctness_reward_func, format_reward_func],
    args=training_args,
    train_dataset=dataset,
)
trainer.train()
```

### Inference

```python
FastLanguageModel.for_inference(model)

inputs = tokenizer(["What is AI?"], return_tensors="pt").to("cuda")
outputs = model.generate(**inputs, max_new_tokens=128)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```

### Export and Saving

```python
# Save LoRA adapter (~100MB)
model.save_pretrained("lora_model")

# Save to GGUF for Ollama/llama.cpp
model.save_pretrained_gguf("model_gguf", tokenizer, quantization_method="q4_k_m")

# Push to Hugging Face Hub
model.push_to_hub("username/model-name", token="hf_...")
model.push_to_hub_gguf("username/model-gguf", tokenizer, quantization_method="q4_k_m", token="hf_...")
```

## Configuration

### Model Loading Options

- `model_name`: Hugging Face model ID or local path
- `max_seq_length`: Maximum context length (default: 2048)
- `dtype`: `None` (auto), `torch.float16`, or `torch.bfloat16`
- `load_in_4bit`: Enable 4-bit QLoRA (default: True)
- `load_in_16bit`: Enable 16-bit LoRA
- `full_finetuning`: Enable full parameter fine-tuning

### LoRA Parameters

- `r`: LoRA rank (8, 16, 32, 64 common values)
- `lora_alpha`: Scaling factor (typically equal to r)
- `lora_dropout`: Dropout rate (0 recommended for Unsloth)
- `target_modules`: List of modules to apply LoRA
- `use_rslora`: Rank-stabilized LoRA scaling
- `use_gradient_checkpointing`: `"unsloth"` for optimized checkpointing

### VRAM Requirements

| Model Size | QLoRA (4-bit) | LoRA (16-bit) |
|------------|---------------|---------------|
| 3B | 3.5 GB | 8 GB |
| 7-8B | 5 GB | 19 GB |
| 14B | 10 GB | 38 GB |
| 70B | 41 GB | 164 GB |
| 405B | 237 GB | 950 GB |

### Environment Flags

Unsloth provides environment flags for controlling behavior (logging, memory management, kernel selection) via the `UNSLOTH_*` environment variable prefix [1].

## Integration Patterns

### Ollama Deployment

Export trained models to GGUF format, then serve via Ollama for local inference with `ollama run` [4].

### vLLM Serving

Deploy fine-tuned models via vLLM for enterprise serving with LoRA hot-swapping — swap adapters without reloading the base model [4].

### Hugging Face TRL

Unsloth uses TRL's `SFTTrainer`, `GRPOTrainer`, `DPOTrainer`, and `ORPOTrainer` directly. The optimization is transparent — standard TRL code works with Unsloth models [2][3].

### llama.cpp and LM Studio

GGUF exports are directly compatible with llama.cpp for CLI inference and LM Studio for GUI-based local inference [4].

## Examples

### Quick LoRA Fine-Tuning

```python
from unsloth import FastLanguageModel
from trl import SFTTrainer
from transformers import TrainingArguments
from datasets import load_dataset

# Load model
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Llama-3.1-8B-bnb-4bit",
    max_seq_length=2048,
    load_in_4bit=True,
)

# Apply LoRA
model = FastLanguageModel.get_peft_model(
    model, r=16, target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                                  "gate_proj", "up_proj", "down_proj"],
    lora_alpha=16, lora_dropout=0,
)

# Train
dataset = load_dataset("yahma/alpaca-cleaned", split="train")
trainer = SFTTrainer(
    model=model, tokenizer=tokenizer, train_dataset=dataset,
    args=TrainingArguments(
        per_device_train_batch_size=2,
        gradient_accumulation_steps=4,
        num_train_epochs=1,
        learning_rate=2e-4,
        output_dir="outputs",
    ),
)
trainer.train()

# Export to GGUF
model.save_pretrained_gguf("model_gguf", tokenizer, quantization_method="q4_k_m")
```

### Vision Fine-Tuning

```python
from unsloth import FastVisionModel

model, tokenizer = FastVisionModel.from_pretrained(
    "unsloth/Qwen2-VL-7B-Instruct-bnb-4bit",
    load_in_4bit=True,
)

model = FastVisionModel.get_peft_model(
    model, r=16,
    finetune_vision_layers=True,
    finetune_language_layers=True,
    finetune_attention_modules=True,
    finetune_mlp_modules=True,
)
```

## Limitations

- **NVIDIA GPU focused**: Requires CUDA Capability 7.0+; AMD and Intel support available but less mature [6]
- **No Apple Silicon support**: macOS/M-series GPU support is not yet available [6]
- **Single GPU default**: Multi-GPU training works but an improved version is still in development [1]
- **TRL dependency**: Training workflows depend on Hugging Face TRL; custom training loops require more manual integration
- **RL minimum model size**: Reasoning token generation requires minimum 1.5B parameter models [3]
- **RL convergence time**: GRPO training requires minimum 300 steps before meaningful reward increases [3]
- **Memory for large models**: Despite optimizations, 70B+ models still require 41GB+ VRAM even with QLoRA [6]

## Changelog

- **Dynamic 2.0 GGUFs**: Intelligent per-layer quantization with 1.5M+ token calibration dataset
- **Faster MoE Training**: 12x faster Mixture-of-Experts training with less VRAM
- **Ultra Long Context RL**: 500K+ context length support for GRPO training
- **Embedding Fine-Tuning**: Custom embedding model training
- **TTS Fine-Tuning**: Text-to-speech model training support
- **Vision Fine-Tuning**: Selective layer fine-tuning for VLMs
- **GRPO/GSPO/DR-GRPO**: Multiple RL method variants with 90% VRAM reduction
- **3x Faster Training**: Packing optimizations for additional speedup
- **Quantization-Aware Training**: QAT support for training-time quantization
- **Blackwell/RTX 50 Support**: Compatibility with NVIDIA Blackwell architecture

## Citations

- [1] Unsloth Documentation Home - https://unsloth.ai/docs
- [2] Fine-tuning Guide - https://unsloth.ai/docs/get-started/fine-tuning-llms-guide
- [3] Reinforcement Learning Guide - https://unsloth.ai/docs/get-started/reinforcement-learning-rl-guide
- [4] Inference & Deployment - https://unsloth.ai/docs/basics/inference-and-deployment
- [5] Dynamic 2.0 GGUFs - https://unsloth.ai/docs/basics/unsloth-dynamic-2.0-ggufs
- [6] System Requirements - https://unsloth.ai/docs/get-started/fine-tuning-for-beginners/unsloth-requirements
- [7] Vision Fine-tuning - https://unsloth.ai/docs/basics/vision-fine-tuning
- [8] Installation - https://unsloth.ai/docs/get-started/install
