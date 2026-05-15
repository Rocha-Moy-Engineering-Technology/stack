# PEFT

> HuggingFace library for parameter-efficient fine-tuning with LoRA and adapter methods

| Field | Value |
|-------|-------|
| Name | PEFT |
| Group | Fine-tuning |
| Type | SDK |
| Open Source | yes |
| GitHub | [huggingface/peft](https://github.com/huggingface/peft) |
| Stars | 21114 |
| Docs | [huggingface.co/docs/peft](https://huggingface.co/docs/peft/en/index) |

## Overview

PEFT (Parameter-Efficient Fine-Tuning) is a Hugging Face library for adapting large pretrained models to downstream tasks by training only a small number of extra parameters instead of all model weights. This dramatically reduces computational and storage costs while yielding performance comparable to full fine-tuning [1].

PEFT is integrated with the Hugging Face Transformers, Diffusers, and Accelerate libraries. It supports training with the Transformers Trainer, Accelerate, or custom PyTorch training loops. Trained adapters are stored as small files (e.g., 6MB for a LoRA adapter on a 350M model vs. 700MB for the full model) and can be loaded, swapped, and merged at inference time [2].

The library supports two broad categories of methods: **adapter-based methods** (LoRA, AdaLoRA, LoHa, LoKr, OFT, BOFT, HRA, MiSS, Llama-Adapter) that add trainable parameters to frozen model layers, and **soft prompting methods** (Prompt Tuning, Prefix Tuning, P-Tuning, Multitask Prompt Tuning, CPT) that prepend learnable tokens to model inputs [3][4].

## Core Concepts

### Low-Rank Adaptation (LoRA)

LoRA decomposes weight updates into two smaller low-rank matrices (A and B) instead of modifying the full weight matrix. The original weights remain frozen, and only the low-rank matrices are trained. Key parameters [5][6]:

- **r**: Rank dimension of the decomposition (higher = more parameters, more capacity)
- **lora_alpha**: Scaling factor (effective scaling is `lora_alpha/r`, or `lora_alpha/sqrt(r)` with rsLoRA)
- **target_modules**: Which layers to apply LoRA to (e.g., `"all-linear"` for QLoRA-style)
- **lora_dropout**: Dropout probability for LoRA layers

LoRA adapters can be merged into the base model via `merge_and_unload()` to eliminate inference latency, or kept separate for swapping between tasks [6].

### LoRA Variants

PEFT supports many LoRA initialization and optimization strategies [5][6]:

- **DoRA (Weight-Decomposed Low-Rank Adaptation)**: Separates weight updates into magnitude and direction components, improving performance especially at low ranks
- **rsLoRA (Rank-Stabilized LoRA)**: Uses `lora_alpha/sqrt(r)` scaling for more stable training at higher ranks
- **PiSSA**: Initializes LoRA from principal singular values for faster convergence
- **OLoRA**: Uses QR decomposition initialization for improved stability
- **EVA (Explained Variance Adaptation)**: Data-driven initialization via SVD of layer input activations with adaptive rank redistribution
- **CorDA (Context-Oriented Decomposition Adaptation)**: Task-aware initialization with instruction-previewed or knowledge-preserved modes
- **LoftQ**: Initializes LoRA to minimize quantization error for QLoRA training
- **aLoRA (Activated LoRA)**: Selectively activates adapters only on tokens after an invocation sequence, enabling KV cache reuse

### Other Adapter Methods

- **AdaLoRA**: Adaptively allocates rank across layers based on importance scoring via SVD-like parameterization [3]
- **LoHa**: Uses Hadamard product of four low-rank matrices for higher expressivity at the same parameter count [3]
- **LoKr**: Uses Kronecker product decomposition preserving rank of original weights [3]
- **OFT (Orthogonal Finetuning)**: Learns orthogonal transformations preserving cosine similarity between neurons [3]
- **BOFT (Orthogonal Butterfly)**: Factorizes orthogonal transformation into sparse butterfly matrices with O(d log d) parameters [3]
- **HRA (Householder Reflection Adaptation)**: Chains trainable Householder reflections bridging LoRA and OFT [3]
- **MiSS (Matrix Shard Sharing)**: Uses a single trainable matrix with shard-sharing mechanism [3]
- **X-LoRA**: Mixture of LoRA experts with dynamic gating for token-level adapter activation [3]
- **Llama-Adapter**: Zero-initialized attention with learnable adaption prompts for upper model layers [3]

### Soft Prompting Methods

- **Prompt Tuning**: Adds learnable prompt tokens to model input embeddings; model parameters remain frozen [4]
- **Prefix Tuning**: Inserts trainable prefix parameters into all model layers (not just input), optimized via a feed-forward network [4]
- **P-Tuning**: Learnable prompt tokens insertable anywhere in the input sequence, optimized by a bidirectional LSTM encoder [4]
- **Multitask Prompt Tuning**: Learns a single shared prompt from multiple tasks via Hadamard product decomposition [4]
- **CPT (Context-Aware Prompt Tuning)**: Refines context embeddings for few-shot classification with controlled perturbations [4]

### Quantization Support

PEFT works with quantized models via multiple backends [7]:

- **bitsandbytes**: 4-bit and 8-bit quantization (QLoRA)
- **GPTQ**: 2/3/4/8-bit post-training quantization
- **AWQ**: Activation-aware weight quantization
- **AQLM**: Additive quantization down to 2-bit
- **EETQ**: Efficient 8-bit quantization
- **HQQ**: Half-Quadratic Quantization
- **torchao**: PyTorch native int8 quantization
- **INC**: Intel Neural Compressor for FP8 on HPU devices

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                   PeftConfig                         │
│  LoraConfig │ PrefixTuningConfig │ PromptTuningConfig│
│  AdaLoraConfig │ OFTConfig │ LoHaConfig │ ...       │
└──────────────────────┬──────────────────────────────┘
                       │
                       v
┌─────────────────────────────────────────────────────┐
│              get_peft_model()                         │
│  Wraps base model + config into PeftModel            │
└──────────────────────┬──────────────────────────────┘
                       │
          ┌────────────┼────────────┐
          v            v            v
┌──────────────┐ ┌──────────┐ ┌──────────────────┐
│  Adapter     │ │   Soft   │ │  Base Model      │
│  Methods     │ │  Prompts │ │  (Frozen)         │
│              │ │          │ │                   │
│  LoRA A/B    │ │  Learned │ │  Transformers     │
│  OFT blocks  │ │  tokens  │ │  Diffusers        │
│  LoHa/LoKr   │ │  prefixes│ │  Any PyTorch      │
└──────────────┘ └──────────┘ └──────────────────┘
                       │
                       v
┌─────────────────────────────────────────────────────┐
│                   Output                             │
│  save_pretrained() → adapter_config.json             │
│                    + adapter_model.safetensors        │
│  push_to_hub() → Hugging Face Hub                    │
│  merge_and_unload() → Standalone merged model        │
└─────────────────────────────────────────────────────┘
```

PEFT wraps any base model (Transformers, Diffusers, or custom PyTorch) with a PeftModel that injects trainable adapter layers while keeping the base frozen. Only adapter weights are saved and loaded. Multiple adapters can coexist on the same base model and be activated, swapped, or merged independently [2][6].

## Key Features

- **15+ PEFT Methods**: LoRA, DoRA, AdaLoRA, LoHa, LoKr, OFT, BOFT, HRA, MiSS, X-LoRA, Llama-Adapter, Prompt Tuning, Prefix Tuning, P-Tuning, Multitask Prompt Tuning, CPT
- **7+ LoRA Initialization Strategies**: Default, Gaussian, PiSSA, OLoRA, EVA, CorDA, LoftQ, orthogonal
- **Adapter Merging**: Merge multiple LoRA adapters via SVD, linear combination, TIES, DARE, magnitude pruning, or concatenation
- **Adapter Swapping**: Load, activate, and switch between multiple adapters at inference time without reloading the base model
- **Mixed-Adapter Batches**: Use different LoRA adapters for different samples in the same batch via `adapter_names`
- **Weight Merging**: Merge adapters into base model via `merge_and_unload()` for zero-overhead inference
- **8+ Quantization Backends**: bitsandbytes (QLoRA), GPTQ, AWQ, AQLM, EETQ, HQQ, torchao, INC
- **Specialized Optimizers**: LoRA-FA (fixed A matrix) and LoRA+ (differential learning rates for A and B)
- **Trainable Token Indices**: Efficiently fine-tune specific embedding tokens alongside LoRA
- **Per-Layer Rank Control**: `rank_pattern` and `alpha_pattern` for layer-specific ranks and scaling
- **Layer Replication**: Memory-efficient model expansion by duplicating layers with separate LoRA adapters
- **Arrow Routing**: Gradient-free token-wise mixture-of-experts routing across LoRA adapters
- **Integration**: Seamless with Transformers Trainer, Accelerate, DeepSpeed, FSDP, Diffusers, and Hugging Face Hub

## Use Cases

- **LLM Fine-Tuning**: Adapt large language models to domain-specific tasks with LoRA/QLoRA using minimal GPU memory
- **QLoRA Training**: Fine-tune 65B+ parameter models on a single 48GB GPU by combining 4-bit quantization with LoRA
- **Image Generation**: Fine-tune Stable Diffusion and FLUX models with LoRA, LoHa, or LoKr adapters via Diffusers
- **Multi-Task Adapters**: Train separate LoRA adapters for different tasks on the same base model, swapping at inference
- **Instruction Following**: Adapt base models into instruction-following assistants with Llama-Adapter or LoRA
- **Speech Recognition**: Apply adapter methods to automatic speech recognition models like Whisper
- **Classification**: Use soft prompting or LoRA for text/image classification with minimal trainable parameters
- **Adapter Composition**: Combine multiple trained LoRA adapters into new capabilities via Arrow routing or weighted merging

## API Reference

### Training

```python
from peft import LoraConfig, get_peft_model, TaskType

# Configure LoRA
config = LoraConfig(
    task_type=TaskType.CAUSAL_LM,
    r=16,
    lora_alpha=32,
    lora_dropout=0.1,
    target_modules=["q_proj", "v_proj", "k_proj", "o_proj"],
)

# Wrap base model
model = get_peft_model(base_model, config)
model.print_trainable_parameters()
# "trainable params: 2359296 || all params: 1231940608 || trainable%: 0.19"
```

### Saving and Loading

```python
# Save adapter only
model.save_pretrained("output_dir")

# Push to Hub
model.push_to_hub("username/model-lora")

# Load for inference
from peft import AutoPeftModelForCausalLM
model = AutoPeftModelForCausalLM.from_pretrained("username/model-lora")
```

### Adapter Operations

```python
from peft import PeftModel

# Load base + adapter
model = PeftModel.from_pretrained(base_model, "adapter-path", adapter_name="sft")

# Load additional adapter
model.load_adapter("another-adapter-path", adapter_name="dpo")

# Switch active adapter
model.set_adapter("dpo")

# Merge into base model
model = model.merge_and_unload()

# Or merge/unmerge reversibly
model.merge_adapter()
model.unmerge_adapter()
```

### Weighted Adapter Merging

```python
model.add_weighted_adapter(
    adapters=["sft", "dpo"],
    weights=[0.7, 0.3],
    adapter_name="merged",
    combination_type="linear",  # or svd, ties, dare_linear, etc.
)
```

## Configuration

### LoraConfig Parameters

- `r` (int): LoRA rank dimension
- `lora_alpha` (int): Scaling factor
- `lora_dropout` (float): Dropout probability (default: 0.0)
- `target_modules` (list/str): Modules to apply LoRA; `"all-linear"` for all linear layers
- `bias` (str): `"none"`, `"all"`, or `"lora_only"`
- `task_type` (TaskType): `CAUSAL_LM`, `SEQ_2_SEQ_LM`, `TOKEN_CLS`, `SEQ_CLS`, `FEATURE_EXTRACTION`
- `use_rslora` (bool): Rank-stabilized scaling (default: False)
- `use_dora` (bool): Weight-Decomposed adaptation (default: False)
- `init_lora_weights` (str/bool): Initialization — `True`, `"gaussian"`, `"pissa"`, `"olora"`, `"eva"`, `"corda"`, `"loftq"`, `"orthogonal"`
- `rank_pattern` (dict): Per-layer rank overrides via regex
- `alpha_pattern` (dict): Per-layer alpha overrides via regex
- `modules_to_save` (list): Additional modules to train and save beyond adapters

### QLoRA Setup

```python
from transformers import BitsAndBytesConfig
from peft import prepare_model_for_kbit_training

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_use_double_quant=True,
    bnb_4bit_compute_dtype=torch.bfloat16,
)

model = AutoModelForCausalLM.from_pretrained(model_id, quantization_config=bnb_config)
model = prepare_model_for_kbit_training(model)
model = get_peft_model(model, lora_config)
```

## Integration Patterns

### Transformers Trainer

```python
from transformers import Trainer, TrainingArguments

training_args = TrainingArguments(
    output_dir="output",
    learning_rate=1e-3,
    per_device_train_batch_size=32,
    num_train_epochs=2,
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=dataset["train"],
    processing_class=tokenizer,
)
trainer.train()
```

### Hugging Face Hub

Adapters are uploaded to and loaded from the Hub. Only the small adapter files (config + weights) are stored, with the base model referenced by name [2].

### DeepSpeed and FSDP

PEFT adapters work with DeepSpeed ZeRO stages and Fully Sharded Data Parallel via Accelerate for distributed training [1].

### Diffusers

PEFT integrates with Diffusers for training LoRA adapters on Stable Diffusion, SDXL, and FLUX models [1].

## Examples

### LoRA on Causal LM

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import LoraConfig, get_peft_model, TaskType

model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-3.2-1B")
tokenizer = AutoTokenizer.from_pretrained("meta-llama/Llama-3.2-1B")

config = LoraConfig(
    task_type=TaskType.CAUSAL_LM,
    r=8,
    lora_alpha=32,
    target_modules=["q_proj", "v_proj"],
    lora_dropout=0.05,
)

model = get_peft_model(model, config)
model.print_trainable_parameters()
# Train with Trainer or custom loop...
model.save_pretrained("llama-lora")
```

### Inference with AutoPeftModel

```python
from peft import AutoPeftModelForCausalLM
from transformers import AutoTokenizer

model = AutoPeftModelForCausalLM.from_pretrained("username/llama-lora")
tokenizer = AutoTokenizer.from_pretrained("meta-llama/Llama-3.2-1B")

inputs = tokenizer("The capital of France is", return_tensors="pt")
outputs = model.generate(**inputs, max_new_tokens=20)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```

### EVA Initialization

```python
from peft import LoraConfig, EvaConfig, get_peft_model, initialize_lora_eva_weights

config = LoraConfig(
    init_lora_weights="eva",
    eva_config=EvaConfig(rho=2.0),
    r=16,
    target_modules="all-linear",
)

model = get_peft_model(base_model, config, low_cpu_mem_usage=True)
initialize_lora_eva_weights(model, dataloader)
```

## Limitations

- **Soft prompts not human-readable**: Learned prompt tokens are virtual embeddings that don't correspond to real words, making interpretation difficult [4]
- **DoRA inference overhead**: DoRA introduces larger overhead than pure LoRA during inference; weight merging is recommended for production [6]
- **aLoRA cannot be merged**: Activated LoRA adapters cannot be merged into the base model due to selective token application [6]
- **Mixed-adapter batches inference only**: Using different adapters per sample in a batch works only for inference, not training [6]
- **AQLM merging not supported**: LoRA adapters trained on AQLM-quantized models cannot be merged with quantized weights [7]
- **torchao limited support**: Only int8 weight-only quantization is fully supported; int4 and NF4 not yet available; merging only works with LoRA + int8 [7]
- **INC no merge/unmerge**: Intel Neural Compressor quantized models do not support adapter merging [7]
- **Adapter composition complexity**: Methods like Arrow and X-LoRA require all adapters to share the same rank and target modules [6]

## Changelog

- **v0.18.0**: Current release with aLoRA (Activated LoRA), Arrow routing, GenKnowSub, MiSS, target_parameters for MoE nn.Parameter support
- **EVA**: Data-driven initialization with adaptive rank redistribution
- **CorDA**: Context-oriented decomposition with instruction-previewed and knowledge-preserved modes
- **DoRA**: Weight-decomposed adaptation with magnitude/direction separation
- **Arrow + GenKnowSub**: Modular routing and general knowledge subtraction for multi-adapter composition
- **LoRA-FA and LoRA+**: Specialized optimizers for improved LoRA training
- **CPT**: Context-Aware Prompt Tuning for few-shot classification
- **Trainable Token Indices**: Memory-efficient selective token fine-tuning alongside LoRA
- **8+ Quantization Backends**: bitsandbytes, GPTQ, AWQ, AQLM, EETQ, HQQ, torchao, INC

## Citations

- [1] PEFT Overview - https://huggingface.co/docs/peft/en/index
- [2] Quicktour - https://huggingface.co/docs/peft/en/quicktour
- [3] Adapter Conceptual Guide - https://huggingface.co/docs/peft/en/conceptual_guides/adapter
- [4] Prompting Conceptual Guide - https://huggingface.co/docs/peft/en/conceptual_guides/prompting
- [5] LoRA API Reference - https://huggingface.co/docs/peft/en/package_reference/lora
- [6] LoRA Developer Guide - https://huggingface.co/docs/peft/en/developer_guides/lora
- [7] Quantization Guide - https://huggingface.co/docs/peft/en/developer_guides/quantization
- [8] Installation - https://huggingface.co/docs/peft/en/install

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

PEFT, parameter-efficient fine-tuning, LoRA, DoRA, AdaLoRA, LoHa, LoKr, OFT, BOFT, HRA, MiSS, X-LoRA, Llama-Adapter, Prompt Tuning, Prefix Tuning, P-Tuning, CPT, PiSSA, OLoRA, EVA, CorDA, LoftQ, aLoRA, rsLoRA, adapter swapping, adapter merging, merge_and_unload, mixed-adapter batches, QLoRA, bitsandbytes, GPTQ, AWQ, AQLM, HQQ, torchao, Diffusers LoRA, Arrow routing

### Verb-Noun Tasks

- Wrap a base model with `get_peft_model(base, LoraConfig(...))`
- Configure LoRA rank, alpha, target_modules, and `"all-linear"` selection
- Merge a trained adapter into the base model via `merge_and_unload()`
- Swap multiple adapters at inference with `set_adapter` and `load_adapter`
- Combine adapters with `add_weighted_adapter` using SVD, TIES, or DARE
- Set up QLoRA with `BitsAndBytesConfig` and `prepare_model_for_kbit_training`
- Initialize LoRA with PiSSA, OLoRA, EVA, CorDA, or LoftQ
- Apply per-layer rank overrides via `rank_pattern` and `alpha_pattern`
- Train LoRA adapters on Stable Diffusion / FLUX through Diffusers
- Run mixed-adapter batches at inference with `adapter_names` per sample
- Save and load adapters as small artifacts on the Hugging Face Hub
- Use DoRA to separate weight magnitude and direction for low-rank training

### User Intent Phrases

- How do I fine-tune a model by training only a small number of extra parameters?
- What is the difference between LoRA, DoRA, and AdaLoRA?
- How do I run QLoRA with 4-bit bitsandbytes quantization?
- How do I merge a LoRA adapter into the base model for zero-overhead inference?
- How do I swap between two trained adapters at inference time?
- How can I serve different LoRA adapters to different samples in the same batch?
- What initialization method should I use for LoRA — PiSSA, EVA, or LoftQ?
- How do I train a LoRA adapter for Stable Diffusion or FLUX?
- How do I save just the adapter weights instead of the full model?
- What soft prompting method (Prefix Tuning, P-Tuning, Prompt Tuning) should I use for classification?

### Problem Statements

- Full fine-tuning requires too much GPU memory and storage per task
- Each task-specific full model costs hundreds of MB to GB; managing dozens is impractical
- LoRA's vanilla initialization is slow to converge on some tasks
- Soft prompts are not human-readable and hard to debug
- Quantization backends differ in which PEFT methods they support for merging
- Composing multiple adapters into a routed mixture requires coordination across ranks and modules
- DoRA introduces inference overhead unless adapters are merged

### When to Pick This

- Pick this when you want library-level breadth of adapter methods (15+ PEFT methods including DoRA, OFT, LoHa, LoKr, X-LoRA) inside your own training code — Unsloth wins when you need speed/VRAM optimization, Axolotl wins when you want a YAML-driven training pipeline
- Pick this when you need adapter swapping, merging, or mixed-adapter batches at inference
- Pick this when you train across Transformers, Diffusers, and custom PyTorch with one library
- Pick this when you need fine-grained QLoRA configuration with multiple quantization backends (bitsandbytes, GPTQ, AWQ, AQLM, EETQ, HQQ, torchao)
- Pick this when you want advanced LoRA initialization (PiSSA, OLoRA, EVA, CorDA, LoftQ)
- Pick this when soft prompting methods (Prompt Tuning, Prefix Tuning, P-Tuning, CPT) are a requirement
- Pick this when you want first-class Hugging Face Hub adapter publishing and loading

### Related Terms and Aliases

- huggingface/peft
- Parameter-Efficient Fine-Tuning, adapter tuning
- Low-Rank Adaptation, weight-decomposed adaptation
- QLoRA (Quantized LoRA), 4-bit fine-tuning
- Adapter hub, adapter merging, adapter swapping
- LoRA-FA, LoRA+ (specialized optimizers)
- TIES merging, DARE merging, SVD merging
- AutoPeftModelForCausalLM, PeftModel
- Soft prompts, virtual tokens, learnable embeddings
