[Header 1 ("axolotl", [], []) [Str "Axolotl"], BlockQuote [Para [Str "Open-source LLM post-training framework with YAML config for LoRA and full fine-tuning"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Axolotl"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Fine-tuning"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "axolotl-ai-cloud/axolotl"] ("https://github.com/axolotl-ai-cloud/axolotl", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "11903"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Docs"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "docs.axolotl.ai"] ("https://docs.axolotl.ai/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Axolotl is an open-source framework for fine-tuning and post-training large language models (LLMs). It provides a YAML-driven configuration system that abstracts the complexity of training pipelines, supporting LoRA, QLoRA, full fine-tuning, and reinforcement learning from human feedback (RLHF) methods. The framework wraps Hugging Face Transformers, PEFT, TRL, and DeepSpeed into a unified interface controlled by a single configuration file ", Str "[", Str "1", Str "]", Str "."], Para [Str "Key capabilities include multimodal training (Vision-Language Models), multiple model architecture support (Llama, Mistral, Mixtral, Qwen, Gemma, Phi, Falcon, and others), sample packing for training efficiency, and distributed training via Fully Sharded Data Parallel (FSDP) and DeepSpeed ", Str "[", Str "1", Str "]", Str "."], Para [Str "Axolotl requires an NVIDIA Ampere or newer GPU (or AMD GPU), Python 3.11+, and PyTorch 2.8.0 or higher. macOS M-series is also supported ", Str "[", Str "2", Str "]", Str "."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("yaml-configuration", ["unnumbered", "unlisted"], []) [Str "YAML Configuration"], Para [Str "All training parameters are specified in a single YAML configuration file. This file controls the base model, adapter type, dataset paths and formats, training hyperparameters, optimizer, scheduler, precision settings, and output location. The CLI passes this config file to all commands ", Str "[", Str "3", Str "]", Str "[", Str "4", Str "]", Str "."], Header 3 ("training-methods", ["unnumbered", "unlisted"], []) [Str "Training Methods"], Para [Str "Axolotl supports several training approaches ", Str "[", Str "3", Str "]", Str ":"], BulletList [[Plain [Strong [Str "Full Fine-tuning"], Str ": Updates all model parameters"]], [Plain [Strong [Str "LoRA (Low-Rank Adaptation)"], Str ": Attaches low-rank adapter matrices to target modules, training only the adapter weights"]], [Plain [Strong [Str "QLoRA"], Str ": Combines 4-bit quantization of the base model with LoRA adapters for reduced memory usage"]], [Plain [Strong [Str "llama-adapter"], Str ": Lightweight adapter method for Llama models"]]], Header 3 ("dataset-formats", ["unnumbered", "unlisted"], []) [Str "Dataset Formats"], Para [Str "Axolotl provides built-in support for multiple dataset formats ", Str "[", Str "5", Str "]", Str "[", Str "6", Str "]", Str ":"], Para [Strong [Str "Instruction Tuning"], Str ": ", Code ("", [], []) "alpaca", Str " (instruction/input/output), ", Code ("", [], []) "gpteacher", Str ", ", Code ("", [], []) "oasst", Str ", ", Code ("", [], []) "reflection", Str ", ", Code ("", [], []) "summarizetldr", Str ", ", Code ("", [], []) "jeopardy", Str ", ", Code ("", [], []) "context_qa", Str ", and custom field mappings."], Para [Strong [Str "Conversation/Chat"], Str ": The recommended ", Code ("", [], []) "chat_template", Str " format uses Jinja2 templates to convert message lists into model-specific prompts. It supports tokenizer defaults, built-in templates (chatml, gemma, llama4, qwen3), and custom templates. Legacy ", Code ("", [], []) "sharegpt", Str " and ", Code ("", [], []) "pygmalion", Str " formats are also supported but deprecated in favor of chat_template ", Str "[", Str "6", Str "]", Str "."], Para [Strong [Str "Pre-training"], Str ": Raw text datasets for continued pre-training of base models."], Para [Strong [Str "Preference Data"], Str ": Chosen/rejected pairs for DPO, IPO, KTO, and ORPO training methods ", Str "[", Str "7", Str "]", Str "."], Header 3 ("sample-packing-multipack", ["unnumbered", "unlisted"], []) [Str "Sample Packing (Multipack)"], Para [Str "Multipack is a technique that packs multiple sequences into a single batch to increase training throughput. With Flash Attention, sequences are concatenated and Flash Attention is notified of sequence boundaries through ", Code ("", [], []) "cu_seqlens", Str " parameters, preventing cross-sequence attention while maintaining efficiency. Without Flash Attention, packing uses 4D attention masks with reduced efficiency ", Str "[", Str "8", Str "]", Str "."], Para [Str "Benefits include reduced padding waste, better GPU utilization, and consistent token counts per training step despite variable input lengths."], Header 3 ("reinforcement-learning-methods", ["unnumbered", "unlisted"], []) [Str "Reinforcement Learning Methods"], Para [Str "Axolotl wraps the TRL library to support multiple RL methods (beta feature) ", Str "[", Str "7", Str "]", Str ":"], BulletList [[Plain [Strong [Str "DPO (Direct Preference Optimization)"], Str ": 15+ dataset format variants"]], [Plain [Strong [Str "IPO (Identity Preference Optimization)"], Str ": DPO with a different loss function"]], [Plain [Strong [Str "KTO (Kahneman-Tversky Optimization)"], Str ": Completion-based formats with boolean labels"]], [Plain [Strong [Str "ORPO (Odds Ratio Preference Optimization)"], Str ": Configurable via ", Code ("", [], []) "orpo_alpha"]], [Plain [Strong [Str "GRPO (Group Relative Policy Optimization)"], Str ": Uses vLLM for trajectory generation with custom reward functions"]], [Plain [Strong [Str "GDPO (Group Reward-Decoupled Policy Optimization)"], Str ": Extends GRPO for multi-reward training"]], [Plain [Strong [Str "SimPO"], Str ": Alternative loss function using CPOTrainer"]]], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], CodeBlock ("", [""], []) "┌─────────────────────────────────────────────────────┐
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
", Para [Str "The framework orchestrates a pipeline from YAML configuration through dataset preprocessing, model loading (with optional quantization and adapter attachment), training execution (via Hugging Face Trainer or TRL), and output of trained weights. Distributed training is handled through FSDP or DeepSpeed integration ", Str "[", Str "1", Str "]", Str "[", Str "3", Str "]", Str "."], Header 2 ("key-features", ["unnumbered", "unlisted"], []) [Str "Key Features"], BulletList [[Plain [Strong [Str "YAML-Driven Configuration"], Str ": Single config file controls all training parameters — model, data, hyperparameters, and infrastructure"]], [Plain [Strong [Str "Multiple Training Methods"], Str ": Full fine-tuning, LoRA, QLoRA, and llama-adapter with configurable target modules and ranks"]], [Plain [Strong [Str "RLHF/Preference Training"], Str ": DPO, IPO, KTO, ORPO, GRPO, GDPO, and SimPO via TRL integration"]], [Plain [Strong [Str "Sample Packing (Multipack)"], Str ": Packs multiple sequences per batch using Flash Attention block diagonal masks for improved throughput"]], [Plain [Strong [Str "Flash Attention"], Str ": Native integration for memory-efficient attention computation"]], [Plain [Strong [Str "FSDP + QLoRA"], Str ": Train 70B+ parameter models on consumer GPUs (e.g., two 24GB GPUs) by combining Fully Sharded Data Parallel with quantized LoRA"]], [Plain [Strong [Str "DeepSpeed Integration"], Str ": ZeRO optimization stages for distributed training with CPU/disk offloading"]], [Plain [Strong [Str "Multimodal Training"], Str ": Vision-Language Model (VLM) fine-tuning support"]], [Plain [Strong [Str "12+ Dataset Formats"], Str ": Built-in support for alpaca, chat_template, sharegpt, oasst, gpteacher, reflection, and custom formats"]], [Plain [Strong [Str "Chat Template System"], Str ": Jinja2-based templates with per-token loss masking, tool use support, and reasoning split (Qwen3)"]], [Plain [Strong [Str "Gradient Checkpointing"], Str ": Memory optimization trading compute for reduced VRAM usage"]], [Plain [Strong [Str "Mixed Precision Training"], Str ": BF16 and FP16 with automatic detection"]], [Plain [Strong [Str "Hyperparameter Sweeps"], Str ": YAML-based sweep configurations for automated tuning"]], [Plain [Strong [Str "Cloud Execution"], Str ": Modal integration for remote GPU training with ", Code ("", [], []) "--cloud", Str " flag"]], [Plain [Strong [Str "Experiment Tracking"], Str ": Weights & Biases, TensorBoard, and MLflow logging"]], [Plain [Strong [Str "torch.compile"], Str ": Optional compilation for optimized training performance"]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Instruction Tuning"], Str ": Fine-tune base models to follow instructions using alpaca, chat, or custom formats"]], [Plain [Strong [Str "Chat Model Training"], Str ": Build conversational models using multi-turn chat datasets with chat_template formatting"]], [Plain [Strong [Str "Preference Alignment"], Str ": Align models with human preferences using DPO, ORPO, or KTO on chosen/rejected pairs"]], [Plain [Strong [Str "Domain Adaptation"], Str ": Continue pre-training on domain-specific corpora then fine-tune for specialized tasks"]], [Plain [Strong [Str "LoRA Adapter Training"], Str ": Create lightweight task-specific adapters that can be merged or swapped at inference time"]], [Plain [Strong [Str "Large Model Training on Consumer Hardware"], Str ": Fine-tune 70B+ models using FSDP + QLoRA across multiple consumer GPUs"]], [Plain [Strong [Str "Multimodal Fine-tuning"], Str ": Train Vision-Language Models on image-text datasets"]], [Plain [Strong [Str "Reward Model Training"], Str ": Build reward models for RLHF pipelines using preference datasets"]]], Header 2 ("api-reference", ["unnumbered", "unlisted"], []) [Str "API Reference"], Header 3 ("cli-commands", ["unnumbered", "unlisted"], []) [Str "CLI Commands"], CodeBlock ("", ["bash"], []) "# Fetch example configs
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
axolotl inference config.yml --lora-model-dir=\"./outputs/lora-out\"

# Run inference (Gradio UI)
axolotl inference config.yml --gradio

# Merge LoRA adapters into base model
axolotl merge-lora config.yml --lora-model-dir=\"./outputs/lora-out\"

# Evaluate model
axolotl evaluate config.yml

# Run LM evaluation harness
axolotl lm-eval config.yml

# Hyperparameter sweep
axolotl train config.yml --sweep path/to/sweep.yaml

# Cloud execution (Modal)
axolotl train config.yml --cloud cloud_config.yml
", Header 3 ("debug-preprocessing", ["unnumbered", "unlisted"], []) [Str "Debug Preprocessing"], CodeBlock ("", ["bash"], []) "axolotl preprocess config.yml --debug --debug-num-examples 5
", Header 2 ("configuration", ["unnumbered", "unlisted"], []) [Str "Configuration"], Header 3 ("model-configuration", ["unnumbered", "unlisted"], []) [Str "Model Configuration"], CodeBlock ("", ["yaml"], []) "base_model: NousResearch/Llama-3.2-1B
model_type: LlamaForCausalLM
tokenizer_type: AutoTokenizer
load_in_8bit: false
load_in_4bit: true
", Header 3 ("lora-configuration", ["unnumbered", "unlisted"], []) [Str "LoRA Configuration"], CodeBlock ("", ["yaml"], []) "adapter: lora
lora_r: 16
lora_alpha: 32
lora_dropout: 0.05
lora_target_modules:
  - q_proj
  - v_proj
  - k_proj
  - o_proj
", Header 3 ("dataset-configuration", ["unnumbered", "unlisted"], []) [Str "Dataset Configuration"], CodeBlock ("", ["yaml"], []) "datasets:
  - path: my_dataset.jsonl
    type: alpaca
    ds_type: json
  - path: HuggingFaceH4/ultrachat_200k
    type: chat_template
    chat_template: chatml
    split: train_sft

val_set_size: 0.05
", Header 3 ("training-hyperparameters", ["unnumbered", "unlisted"], []) [Str "Training Hyperparameters"], CodeBlock ("", ["yaml"], []) "learning_rate: 3e-4
num_epochs: 3
micro_batch_size: 2
gradient_accumulation_steps: 4
sequence_len: 4096
optimizer: adamw_torch_fused
lr_scheduler: cosine
weight_decay: 0.01
max_grad_norm: 1.0
", Header 3 ("performance-options", ["unnumbered", "unlisted"], []) [Str "Performance Options"], CodeBlock ("", ["yaml"], []) "bf16: auto
flash_attention: true
sample_packing: true
gradient_checkpointing: true
pad_to_sequence_len: true
torch_compile: true
", Header 3 ("rlhf-configuration", ["unnumbered", "unlisted"], []) [Str "RLHF Configuration"], CodeBlock ("", ["yaml"], []) "rl: dpo
rl_beta: 0.1
remove_unused_columns: false
datasets:
  - path: Intel/orca_dpo_pairs
    type: chatml.intel
    split: train
", Header 3 ("distributed-training", ["unnumbered", "unlisted"], []) [Str "Distributed Training"], CodeBlock ("", ["yaml"], []) "# FSDP
fsdp:
  - full_shard
  - auto_wrap
fsdp_config:
  fsdp_offload_params: true

# Or DeepSpeed
deepspeed: deepspeed_configs/zero3_bf16.json
", Header 3 ("logging", ["unnumbered", "unlisted"], []) [Str "Logging"], CodeBlock ("", ["yaml"], []) "use_wandb: true
wandb_project: my-project
wandb_entity: my-team
save_steps: 100
eval_steps: 100
logging_steps: 10
save_total_limit: 3
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("hugging-face-hub", ["unnumbered", "unlisted"], []) [Str "Hugging Face Hub"], Para [Str "Models are loaded directly from Hugging Face Hub via ", Code ("", [], []) "base_model", Str ". Trained models and adapters can be pushed back to the Hub. Datasets are loaded from Hub paths or local files ", Str "[", Str "3", Str "]", Str "."], Header 3 ("peft-parameter-efficient-fine-tuning", ["unnumbered", "unlisted"], []) [Str "PEFT (Parameter-Efficient Fine-Tuning)"], Para [Str "Axolotl uses the PEFT library for LoRA and QLoRA adapter management. Adapters can be merged into the base model via ", Code ("", [], []) "axolotl merge-lora", Str " or loaded separately at inference time ", Str "[", Str "3", Str "]", Str "."], Header 3 ("deepspeed", ["unnumbered", "unlisted"], []) [Str "DeepSpeed"], Para [Str "DeepSpeed ZeRO stages are configured via JSON config files passed through the ", Code ("", [], []) "deepspeed", Str " YAML key. Supports ZeRO-1, ZeRO-2, and ZeRO-3 with CPU and disk offloading ", Str "[", Str "3", Str "]", Str "."], Header 3 ("weights--biases", ["unnumbered", "unlisted"], []) [Str "Weights & Biases"], Para [Str "Native integration for experiment tracking, hyperparameter logging, and loss visualization via ", Code ("", [], []) "use_wandb: true", Str " ", Str "[", Str "3", Str "]", Str "."], Header 3 ("modal-cloud-training", ["unnumbered", "unlisted"], []) [Str "Modal (Cloud Training)"], Para [Str "The ", Code ("", [], []) "--cloud", Str " flag enables remote GPU training on Modal with configurable GPU types and persistent storage volumes ", Str "[", Str "4", Str "]", Str "."], Header 3 ("unsloth", ["unnumbered", "unlisted"], []) [Str "Unsloth"], Para [Str "Axolotl integrates with Unsloth for optimized LoRA training kernels that reduce memory usage and increase training speed ", Str "[", Str "1", Str "]", Str "."], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("basic-lora-fine-tuning", ["unnumbered", "unlisted"], []) [Str "Basic LoRA Fine-Tuning"], CodeBlock ("", ["yaml"], []) "base_model: NousResearch/Llama-3.2-1B
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
", CodeBlock ("", ["bash"], []) "axolotl train lora_config.yml
axolotl merge-lora lora_config.yml
", Header 3 ("chat-model-with-dpo", ["unnumbered", "unlisted"], []) [Str "Chat Model with DPO"], CodeBlock ("", ["yaml"], []) "base_model: NousResearch/Llama-3.2-1B
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
", Header 3 ("fsdp--qlora-for-70b-models", ["unnumbered", "unlisted"], []) [Str "FSDP + QLoRA for 70B Models"], CodeBlock ("", ["yaml"], []) "base_model: meta-llama/Llama-2-70b-hf
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
", Header 2 ("limitations", ["unnumbered", "unlisted"], []) [Str "Limitations"], BulletList [[Plain [Strong [Str "RLHF is beta"], Str ": The documentation notes RLHF features are in beta and many features are not fully implemented ", Str "[", Str "7", Str "]"]], [Plain [Strong [Str "GPU requirements"], Str ": Requires NVIDIA Ampere or newer GPUs (or AMD); older NVIDIA architectures are not supported ", Str "[", Str "2", Str "]"]], [Plain [Strong [Str "Flash Attention dependency"], Str ": Maximum sample packing efficiency requires Flash Attention support; without it, packing uses less efficient 4D masks ", Str "[", Str "8", Str "]"]], [Plain [Strong [Str "Memory demands"], Str ": Full fine-tuning of large models requires substantial GPU memory; even QLoRA of 70B models needs multiple 24GB GPUs ", Str "[", Str "9", Str "]"]], [Plain [Strong [Str "Python version constraint"], Str ": Requires Python 3.11+ specifically, which may conflict with environments using older Python versions ", Str "[", Str "2", Str "]"]], [Plain [Strong [Str "PyTorch minimum version"], Str ": Requires PyTorch 2.8.0+, limiting compatibility with older CUDA toolkit installations ", Str "[", Str "2", Str "]"]], [Plain [Strong [Str "Blackwell GPU constraints"], Str ": NVIDIA Blackwell GPUs require PyTorch 2.9.1+ and CUDA 12.8 with specific nightly builds ", Str "[", Str "2", Str "]"]], [Plain [Strong [Str "Dataset format complexity"], Str ": The number of dataset formats and configuration options creates a learning curve for new users"]]], Header 2 ("changelog", ["unnumbered", "unlisted"], []) [Str "Changelog"], BulletList [[Plain [Strong [Str "2026/03"], Str ": Qwen3.5 support, Mixture-of-Experts (MoE) expert quantization, SageAttention v2"]], [Plain [Strong [Str "2026/02"], Str ": ScatterMoE LoRA, SageAttention v1, GDPO (Group Reward-Decoupled Policy Optimization)"]], [Plain [Strong [Str "2026/01"], Str ": EAFT (Efficient Attention Fine-Tuning), Scalable Softmax"]], [Plain [Strong [Str "GRPO"], Str ": Group Relative Policy Optimization with vLLM trajectory generation and custom reward functions"]], [Plain [Strong [Str "Chat Template System"], Str ": Jinja2-based conversation formatting replacing legacy ShareGPT"]], [Plain [Strong [Str "Modal Cloud"], Str ": Remote GPU training integration"]], [Plain [Strong [Str "N-D Parallelism"], Str ": Multi-dimensional parallelism support"]], [Plain [Strong [Str "Sequence Parallelism"], Str ": Distributed sequence processing across GPUs"]], [Plain [Strong [Str "Quantization-Aware Training (QAT)"], Str ": Training-time quantization via torchao"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " Axolotl Documentation Home - https://docs.axolotl.ai/"]], [Plain [Str "[", Str "2", Str "]", Str " Installation Guide - https://docs.axolotl.ai/docs/installation.html"]], [Plain [Str "[", Str "3", Str "]", Str " Config Reference - https://docs.axolotl.ai/docs/config-reference.html"]], [Plain [Str "[", Str "4", Str "]", Str " CLI Reference - https://docs.axolotl.ai/docs/cli.html"]], [Plain [Str "[", Str "5", Str "]", Str " Instruction Tuning Formats - https://docs.axolotl.ai/docs/dataset-formats/inst_tune.html"]], [Plain [Str "[", Str "6", Str "]", Str " Conversation Formats - https://docs.axolotl.ai/docs/dataset-formats/conversation.html"]], [Plain [Str "[", Str "7", Str "]", Str " RLHF Guide - https://docs.axolotl.ai/docs/rlhf.html"]], [Plain [Str "[", Str "8", Str "]", Str " Multipack - https://docs.axolotl.ai/docs/multipack.html"]], [Plain [Str "[", Str "9", Str "]", Str " FSDP + QLoRA - https://docs.axolotl.ai/docs/fsdp_qlora.html"]]], Header 2 ("discovery-signals", ["unnumbered", "unlisted"], []) [Str "Discovery Signals"], BlockQuote [Para [Str "Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search."]], Header 3 ("keywords", ["unnumbered", "unlisted"], []) [Str "Keywords"], Para [Str "YAML configuration, LLM fine-tuning, LoRA, QLoRA, full fine-tuning, post-training, DPO, IPO, KTO, ORPO, GRPO, GDPO, SimPO, RLHF, sample packing, multipack, Flash Attention, FSDP, DeepSpeed, chat_template, alpaca, sharegpt, Vision-Language Model, VLM, Modal cloud, axolotl CLI, hyperparameter sweep, Jinja2 templates, Llama, Mistral, Qwen, Gemma, Phi"], Header 3 ("verb-noun-tasks", ["unnumbered", "unlisted"], []) [Str "Verb-Noun Tasks"], BulletList [[Plain [Str "Fine-tune Llama or Mistral with LoRA using a single YAML config"]], [Plain [Str "Train a chat model on multi-turn conversations using chat_template formatting"]], [Plain [Str "Run DPO preference alignment on chosen/rejected pairs with TRL"]], [Plain [Str "Train 70B models on consumer GPUs using FSDP + QLoRA"]], [Plain [Str "Pack multiple sequences per batch with multipack and Flash Attention"]], [Plain [Str "Merge a trained LoRA adapter into the base model with ", Code ("", [], []) "axolotl merge-lora"]], [Plain [Str "Resume training from a checkpoint after interruption"]], [Plain [Str "Run hyperparameter sweeps with YAML sweep configs"]], [Plain [Str "Execute remote GPU training on Modal with ", Code ("", [], []) "--cloud"]], [Plain [Str "Fine-tune Vision-Language Models on image-text datasets"]], [Plain [Str "Evaluate trained models with the LM evaluation harness"]], [Plain [Str "Run GRPO with vLLM trajectory generation and custom reward functions"]]], Header 3 ("user-intent-phrases", ["unnumbered", "unlisted"], []) [Str "User Intent Phrases"], BulletList [[Plain [Str "How do I fine-tune Llama 3 with LoRA using YAML instead of writing PyTorch code?"]], [Plain [Str "What is the easiest way to run DPO on a preference dataset?"]], [Plain [Str "How do I train a 70B model with only two 24GB consumer GPUs?"]], [Plain [Str "How do I configure FSDP and DeepSpeed ZeRO-3 for distributed fine-tuning?"]], [Plain [Str "How do I use chat_template with chatml or qwen3 conversation formatting?"]], [Plain [Str "How do I run a hyperparameter sweep over learning rate and LoRA rank?"]], [Plain [Str "How can I launch axolotl training remotely on a Modal GPU?"]], [Plain [Str "How do I fine-tune a Vision-Language Model on a custom image-text dataset?"]], [Plain [Str "How do I switch from full fine-tuning to QLoRA via a YAML flag?"]], [Plain [Str "How do I integrate Weights & Biases logging with axolotl?"]]], Header 3 ("problem-statements", ["unnumbered", "unlisted"], []) [Str "Problem Statements"], BulletList [[Plain [Str "Writing training scripts in raw PyTorch + Trainer is verbose and error-prone"]], [Plain [Str "Switching between LoRA, QLoRA, DPO, and ORPO requires rewriting training code each time"]], [Plain [Str "Distributed training with FSDP or DeepSpeed has a steep configuration learning curve"]], [Plain [Str "Multi-GPU 70B fine-tuning is impossible without combining quantization, sharding, and adapters"]], [Plain [Str "Dataset format inconsistency across alpaca/sharegpt/chatml causes silent training bugs"]], [Plain [Str "Reproducing a peer's fine-tuning recipe requires hunting through scripts and notebooks"]], [Plain [Str "Sample padding wastes GPU time when sequence lengths vary wildly"]]], Header 3 ("when-to-pick-this", ["unnumbered", "unlisted"], []) [Str "When to Pick This"], BulletList [[Plain [Str "Pick this when you want a single YAML config to control model, dataset, hyperparameters, and infrastructure — Unsloth wins when raw speed and VRAM efficiency are the top priority, and PEFT wins when you need library-level control inside Python code"]], [Plain [Str "Pick this when you need full SFT plus DPO/ORPO/KTO/GRPO in one framework"]], [Plain [Str "Pick this when you want native FSDP + QLoRA for very large models on commodity GPUs"]], [Plain [Str "Pick this when you want to run identical recipes across Llama, Mistral, Mixtral, Qwen, Gemma, Phi, Falcon families"]], [Plain [Str "Pick this when reproducibility via version-controlled YAML is more important than ad-hoc notebook training"]], [Plain [Str "Pick this when you need cloud GPU execution via Modal with a single CLI flag"]], [Plain [Str "Pick this when sample packing with Flash Attention's cu_seqlens matters for throughput"]]], Header 3 ("related-terms-and-aliases", ["unnumbered", "unlisted"], []) [Str "Related Terms and Aliases"], BulletList [[Plain [Str "axolotl-ai-cloud/axolotl"]], [Plain [Str "Post-training framework, LLM fine-tuning framework"]], [Plain [Str "YAML-driven training, declarative training config"]], [Plain [Str "TRL wrapper, Hugging Face Trainer wrapper"]], [Plain [Str "Multipack, sample packing, block-diagonal attention"]], [Plain [Str "FSDP+QLoRA, ZeRO-3 offload"]], [Plain [Str "Chat template, ShareGPT (deprecated), Jinja2 conversation formatting"]], [Plain [Str "Direct Preference Optimization, Group Relative Policy Optimization"]]]]