# Axolotl

> Open-source LLM post-training framework with YAML config for LoRA and full fine-tuning

| Field | Value |
|-------|-------|
| Name | Axolotl |
| Group | Fine-tuning |
| Type | SDK |
| Open Source | yes |
| GitHub | [axolotl-ai-cloud/axolotl](https://github.com/axolotl-ai-cloud/axolotl) |
| Stars | 11393 |
| Docs | [docs.axolotl.ai](https://docs.axolotl.ai/) |

## Overview

Axolotl is an open-source framework for fine-tuning and post-training large language models (LLMs). It provides a YAML-driven configuration system that abstracts the complexity of training pipelines, supporting LoRA, QLoRA, full fine-tuning, and reinforcement learning from human feedback (RLHF) methods. The framework wraps Hugging Face Transformers, PEFT, TRL, and DeepSpeed into a unified interface controlled by a single configuration file [1].

Key capabilities include multimodal training (Vision-Language Models), multiple model architecture support (Llama, Mistral, Mixtral, Qwen, Gemma, Phi, Falcon, and others), sample packing for training efficiency, and distributed training via Fully Sharded Data Parallel (FSDP) and DeepSpeed [1].

Axolotl requires an NVIDIA Ampere or newer GPU (or AMD GPU), Python 3.11+, and PyTorch 2.8.0 or higher. macOS M-series is also supported [2].

## Core Concepts

### YAML Configuration

All training parameters are specified in a single YAML configuration file. This file controls the base model, adapter type, dataset paths and formats, training hyperparameters, optimizer, scheduler, precision settings, and output location. The CLI passes this config file to all commands [3][4].

### Training Methods

Axolotl supports several training approaches [3]:

- **Full Fine-tuning**: Updates all model parameters
- **LoRA (Low-Rank Adaptation)**: Attaches low-rank adapter matrices to target modules, training only the adapter weights
- **QLoRA**: Combines 4-bit quantization of the base model with LoRA adapters for reduced memory usage
- **llama-adapter**: Lightweight adapter method for Llama models

### Dataset Formats

Axolotl provides built-in support for multiple dataset formats [5][6]:

**Instruction Tuning**: `alpaca` (instruction/input/output), `gpteacher`, `oasst`, `reflection`, `summarizetldr`, `jeopardy`, `context_qa`, and custom field mappings.

**Conversation/Chat**: The recommended `chat_template` format uses Jinja2 templates to convert message lists into model-specific prompts. It supports tokenizer defaults, built-in templates (chatml, gemma, llama4, qwen3), and custom templates. Legacy `sharegpt` and `pygmalion` formats are also supported but deprecated in favor of chat_template [6].

**Pre-training**: Raw text datasets for continued pre-training of base models.

**Preference Data**: Chosen/rejected pairs for DPO, IPO, KTO, and ORPO training methods [7].

### Sample Packing (Multipack)

Multipack is a technique that packs multiple sequences into a single batch to increase training throughput. With Flash Attention, sequences are concatenated and Flash Attention is notified of sequence boundaries through `cu_seqlens` parameters, preventing cross-sequence attention while maintaining efficiency. Without Flash Attention, packing uses 4D attention masks with reduced efficiency [8].

Benefits include reduced padding waste, better GPU utilization, and consistent token counts per training step despite variable input lengths.

### Reinforcement Learning Methods

Axolotl wraps the TRL library to support multiple RL methods (beta feature) [7]:

- **DPO (Direct Preference Optimization)**: 15+ dataset format variants
- **IPO (Identity Preference Optimization)**: DPO with a different loss function
- **KTO (Kahneman-Tversky Optimization)**: Completion-based formats with boolean labels
- **ORPO (Odds Ratio Preference Optimization)**: Configurable via `orpo_alpha`
- **GRPO (Group Relative Policy Optimization)**: Uses vLLM for trajectory generation with custom reward functions
- **GDPO (Group Reward-Decoupled Policy Optimization)**: Extends GRPO for multi-reward training
- **SimPO**: Alternative loss function using CPOTrainer

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    YAML Config                       │
│  base_model, adapter, datasets, hyperparameters     │
└──────────────────────┬──────────────────────────────┘
                       │
                       v
┌─────────────────────────────────────────────────────┐
│                  Axolotl CLI                          │
│  preprocess │ train │ inference │ merge-lora │ eval  │
└──────────────────────┬──────────────────────────────┘
                       │
          ┌────────────┼────────────┐
          v            v            v
┌──────────────┐ ┌──────────┐ ┌──────────────────┐
│  Dataset     │ │  Model   │ │  Training Loop   │
│  Pipeline    │ │  Loading │ │                   │
│              │ │          │ │  ┌─────────────┐  │
│  Tokenize    │ │  HF Hub  │ │  │ Transformers│  │
│  Format      │ │  PEFT    │ │  │ Trainer     │  │
│  Pack        │ │  BnB     │ │  │ + TRL       │  │
│  Validate    │ │  Quant   │ │  └─────────────┘  │
└──────────────┘ └──────────┘ └────────┬──────────┘
                                        │
                       ┌────────────────┼──────────┐
                       v                v          v
                ┌────────────┐  ┌──────────┐ ┌────────┐
                │   FSDP     │  │DeepSpeed │ │ Single │
                │  Multi-GPU │  │  ZeRO    │ │  GPU   │
                └────────────┘  └──────────┘ └────────┘
                       │
                       v
┌─────────────────────────────────────────────────────┐
│                     Output                           │
│  Checkpoints │ Merged Model │ LoRA Adapters          │
│  WandB Logs  │ TensorBoard  │ MLflow                 │
└─────────────────────────────────────────────────────┘
```

The framework orchestrates a pipeline from YAML configuration through dataset preprocessing, model loading (with optional quantization and adapter attachment), training execution (via Hugging Face Trainer or TRL), and output of trained weights. Distributed training is handled through FSDP or DeepSpeed integration [1][3].

## Key Features

- **YAML-Driven Configuration**: Single config file controls all training parameters — model, data, hyperparameters, and infrastructure
- **Multiple Training Methods**: Full fine-tuning, LoRA, QLoRA, and llama-adapter with configurable target modules and ranks
- **RLHF/Preference Training**: DPO, IPO, KTO, ORPO, GRPO, GDPO, and SimPO via TRL integration
- **Sample Packing (Multipack)**: Packs multiple sequences per batch using Flash Attention block diagonal masks for improved throughput
- **Flash Attention**: Native integration for memory-efficient attention computation
- **FSDP + QLoRA**: Train 70B+ parameter models on consumer GPUs (e.g., two 24GB GPUs) by combining Fully Sharded Data Parallel with quantized LoRA
- **DeepSpeed Integration**: ZeRO optimization stages for distributed training with CPU/disk offloading
- **Multimodal Training**: Vision-Language Model (VLM) fine-tuning support
- **12+ Dataset Formats**: Built-in support for alpaca, chat_template, sharegpt, oasst, gpteacher, reflection, and custom formats
- **Chat Template System**: Jinja2-based templates with per-token loss masking, tool use support, and reasoning split (Qwen3)
- **Gradient Checkpointing**: Memory optimization trading compute for reduced VRAM usage
- **Mixed Precision Training**: BF16 and FP16 with automatic detection
- **Hyperparameter Sweeps**: YAML-based sweep configurations for automated tuning
- **Cloud Execution**: Modal integration for remote GPU training with `--cloud` flag
- **Experiment Tracking**: Weights & Biases, TensorBoard, and MLflow logging
- **torch.compile**: Optional compilation for optimized training performance

## Use Cases

- **Instruction Tuning**: Fine-tune base models to follow instructions using alpaca, chat, or custom formats
- **Chat Model Training**: Build conversational models using multi-turn chat datasets with chat_template formatting
- **Preference Alignment**: Align models with human preferences using DPO, ORPO, or KTO on chosen/rejected pairs
- **Domain Adaptation**: Continue pre-training on domain-specific corpora then fine-tune for specialized tasks
- **LoRA Adapter Training**: Create lightweight task-specific adapters that can be merged or swapped at inference time
- **Large Model Training on Consumer Hardware**: Fine-tune 70B+ models using FSDP + QLoRA across multiple consumer GPUs
- **Multimodal Fine-tuning**: Train Vision-Language Models on image-text datasets
- **Reward Model Training**: Build reward models for RLHF pipelines using preference datasets

## API Reference

### CLI Commands

```bash
# Fetch example configs
axolotl fetch examples

# Preprocess datasets (tokenization)
axolotl preprocess config.yml

# Train a model
axolotl train config.yml

# Train with overrides
axolotl train config.yml --learning-rate 1e-4 --micro-batch-size 2

# Resume from checkpoint
axolotl train config.yml --resume-from-checkpoint path/to/checkpoint

# Multi-GPU training
axolotl train config.yml --launcher torchrun -- --nproc_per_node=4

# Run inference (CLI)
axolotl inference config.yml --lora-model-dir="./outputs/lora-out"

# Run inference (Gradio UI)
axolotl inference config.yml --gradio

# Merge LoRA adapters into base model
axolotl merge-lora config.yml --lora-model-dir="./outputs/lora-out"

# Evaluate model
axolotl evaluate config.yml

# Run LM evaluation harness
axolotl lm-eval config.yml

# Hyperparameter sweep
axolotl train config.yml --sweep path/to/sweep.yaml

# Cloud execution (Modal)
axolotl train config.yml --cloud cloud_config.yml
```

### Debug Preprocessing

```bash
axolotl preprocess config.yml --debug --debug-num-examples 5
```

## Configuration

### Model Configuration

```yaml
base_model: NousResearch/Llama-3.2-1B
model_type: LlamaForCausalLM
tokenizer_type: AutoTokenizer
load_in_8bit: false
load_in_4bit: true
```

### LoRA Configuration

```yaml
adapter: lora
lora_r: 16
lora_alpha: 32
lora_dropout: 0.05
lora_target_modules:
  - q_proj
  - v_proj
  - k_proj
  - o_proj
```

### Dataset Configuration

```yaml
datasets:
  - path: my_dataset.jsonl
    type: alpaca
    ds_type: json
  - path: HuggingFaceH4/ultrachat_200k
    type: chat_template
    chat_template: chatml
    split: train_sft

val_set_size: 0.05
```

### Training Hyperparameters

```yaml
learning_rate: 3e-4
num_epochs: 3
micro_batch_size: 2
gradient_accumulation_steps: 4
sequence_len: 4096
optimizer: adamw_torch_fused
lr_scheduler: cosine
weight_decay: 0.01
max_grad_norm: 1.0
```

### Performance Options

```yaml
bf16: auto
flash_attention: true
sample_packing: true
gradient_checkpointing: true
pad_to_sequence_len: true
torch_compile: true
```

### RLHF Configuration

```yaml
rl: dpo
rl_beta: 0.1
remove_unused_columns: false
datasets:
  - path: Intel/orca_dpo_pairs
    type: chatml.intel
    split: train
```

### Distributed Training

```yaml
# FSDP
fsdp:
  - full_shard
  - auto_wrap
fsdp_config:
  fsdp_offload_params: true

# Or DeepSpeed
deepspeed: deepspeed_configs/zero3_bf16.json
```

### Logging

```yaml
use_wandb: true
wandb_project: my-project
wandb_entity: my-team
save_steps: 100
eval_steps: 100
logging_steps: 10
save_total_limit: 3
```

## Integration Patterns

### Hugging Face Hub

Models are loaded directly from Hugging Face Hub via `base_model`. Trained models and adapters can be pushed back to the Hub. Datasets are loaded from Hub paths or local files [3].

### PEFT (Parameter-Efficient Fine-Tuning)

Axolotl uses the PEFT library for LoRA and QLoRA adapter management. Adapters can be merged into the base model via `axolotl merge-lora` or loaded separately at inference time [3].

### DeepSpeed

DeepSpeed ZeRO stages are configured via JSON config files passed through the `deepspeed` YAML key. Supports ZeRO-1, ZeRO-2, and ZeRO-3 with CPU and disk offloading [3].

### Weights & Biases

Native integration for experiment tracking, hyperparameter logging, and loss visualization via `use_wandb: true` [3].

### Modal (Cloud Training)

The `--cloud` flag enables remote GPU training on Modal with configurable GPU types and persistent storage volumes [4].

### Unsloth

Axolotl integrates with Unsloth for optimized LoRA training kernels that reduce memory usage and increase training speed [1].

## Examples

### Basic LoRA Fine-Tuning

```yaml
base_model: NousResearch/Llama-3.2-1B
load_in_8bit: true
adapter: lora
lora_r: 8
lora_alpha: 16
lora_dropout: 0.05
lora_target_modules:
  - q_proj
  - v_proj

datasets:
  - path: mhenrichsen/alpaca_2k_test
    type: alpaca

sequence_len: 2048
micro_batch_size: 2
gradient_accumulation_steps: 4
num_epochs: 3
learning_rate: 3e-4
optimizer: adamw_torch_fused
lr_scheduler: cosine

bf16: auto
flash_attention: true
sample_packing: true
output_dir: ./outputs/lora-out
```

```bash
axolotl train lora_config.yml
axolotl merge-lora lora_config.yml
```

### Chat Model with DPO

```yaml
base_model: NousResearch/Llama-3.2-1B
adapter: lora
lora_r: 16
lora_alpha: 32

rl: dpo
rl_beta: 0.1
remove_unused_columns: false

datasets:
  - path: Intel/orca_dpo_pairs
    type: chatml.intel
    split: train

sequence_len: 2048
micro_batch_size: 1
num_epochs: 1
learning_rate: 5e-6
optimizer: adamw_torch_fused

bf16: auto
flash_attention: true
output_dir: ./outputs/dpo-out
```

### FSDP + QLoRA for 70B Models

```yaml
base_model: meta-llama/Llama-2-70b-hf
load_in_4bit: true
adapter: qlora
lora_r: 32
lora_alpha: 64

fsdp:
  - full_shard
  - auto_wrap
fsdp_config:
  fsdp_offload_params: true
  fsdp_cpu_ram_efficient_loading: true

datasets:
  - path: my_dataset.jsonl
    type: alpaca

sequence_len: 4096
micro_batch_size: 1
gradient_accumulation_steps: 8
num_epochs: 1
learning_rate: 2e-4
bf16: auto
flash_attention: true
gradient_checkpointing: true
output_dir: ./outputs/70b-qlora
```

## Limitations

- **RLHF is beta**: The documentation notes RLHF features are in beta and many features are not fully implemented [7]
- **GPU requirements**: Requires NVIDIA Ampere or newer GPUs (or AMD); older NVIDIA architectures are not supported [2]
- **Flash Attention dependency**: Maximum sample packing efficiency requires Flash Attention support; without it, packing uses less efficient 4D masks [8]
- **Memory demands**: Full fine-tuning of large models requires substantial GPU memory; even QLoRA of 70B models needs multiple 24GB GPUs [9]
- **Python version constraint**: Requires Python 3.11+ specifically, which may conflict with environments using older Python versions [2]
- **PyTorch minimum version**: Requires PyTorch 2.8.0+, limiting compatibility with older CUDA toolkit installations [2]
- **Blackwell GPU constraints**: NVIDIA Blackwell GPUs require PyTorch 2.9.1+ and CUDA 12.8 with specific nightly builds [2]
- **Dataset format complexity**: The number of dataset formats and configuration options creates a learning curve for new users

## Changelog

- **2026/03**: Qwen3.5 support, Mixture-of-Experts (MoE) expert quantization, SageAttention v2
- **2026/02**: ScatterMoE LoRA, SageAttention v1, GDPO (Group Reward-Decoupled Policy Optimization)
- **2026/01**: EAFT (Efficient Attention Fine-Tuning), Scalable Softmax
- **GRPO**: Group Relative Policy Optimization with vLLM trajectory generation and custom reward functions
- **Chat Template System**: Jinja2-based conversation formatting replacing legacy ShareGPT
- **Modal Cloud**: Remote GPU training integration
- **N-D Parallelism**: Multi-dimensional parallelism support
- **Sequence Parallelism**: Distributed sequence processing across GPUs
- **Quantization-Aware Training (QAT)**: Training-time quantization via torchao

## Citations

- [1] Axolotl Documentation Home - https://docs.axolotl.ai/
- [2] Installation Guide - https://docs.axolotl.ai/docs/installation.html
- [3] Config Reference - https://docs.axolotl.ai/docs/config-reference.html
- [4] CLI Reference - https://docs.axolotl.ai/docs/cli.html
- [5] Instruction Tuning Formats - https://docs.axolotl.ai/docs/dataset-formats/inst_tune.html
- [6] Conversation Formats - https://docs.axolotl.ai/docs/dataset-formats/conversation.html
- [7] RLHF Guide - https://docs.axolotl.ai/docs/rlhf.html
- [8] Multipack - https://docs.axolotl.ai/docs/multipack.html
- [9] FSDP + QLoRA - https://docs.axolotl.ai/docs/fsdp_qlora.html
