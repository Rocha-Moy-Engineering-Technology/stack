[Header 1 ("unsloth", [], []) [Str "Unsloth"], BlockQuote [Para [Str "Fine-tuning and reinforcement learning framework that trains LLMs 2x faster with 70% less VRAM"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Unsloth"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Fine-tuning"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "unslothai/unsloth"] ("https://github.com/unslothai/unsloth", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "53240"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Docs"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "unsloth.ai/docs"] ("https://unsloth.ai/docs", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Unsloth is an open-source framework for fine-tuning and reinforcement learning of large language models that achieves 2x faster training speeds with 70% less VRAM compared to standard implementations. It supports 500+ models including text, vision, text-to-speech, and embedding models, with optimized kernels written in Triton for memory-efficient training ", Str "[", Str "1", Str "]", Str "."], Para [Str "The framework collaborates directly with model teams behind gpt-oss, Qwen3, Llama 4, Mistral, Gemma, and Phi-4, fixing critical bugs that improve model accuracy. It provides a streamlined pipeline from training through evaluation to deployment with Ollama, llama.cpp, vLLM, and other inference engines ", Str "[", Str "1", Str "]", Str "."], Para [Str "Unsloth claims 0% loss in accuracy — no approximation methods are used, all computations are exact. The VRAM savings come from optimized Triton kernels, smart gradient checkpointing, and memory-efficient loss calculations rather than quantization or approximation during training ", Str "[", Str "1", Str "]", Str "."], Para [Str "The framework supports Linux, Windows, NVIDIA GPUs (CUDA Capability 7.0+), AMD GPUs, and Intel GPUs. Python 3.13 is supported ", Str "[", Str "6", Str "]", Str "."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("fastlanguagemodel", ["unnumbered", "unlisted"], []) [Str "FastLanguageModel"], Para [Str "The primary API is ", Code ("", [], []) "FastLanguageModel", Str ", which handles model loading, adapter configuration, inference optimization, and model saving. It wraps Hugging Face Transformers and PEFT with optimized Triton kernels ", Str "[", Str "2", Str "]", Str ":"], BulletList [[Plain [Code ("", [], []) "FastLanguageModel.from_pretrained()", Str ": Loads models with 4-bit, 8-bit, 16-bit, or full precision"]], [Plain [Code ("", [], []) "FastLanguageModel.get_peft_model()", Str ": Applies LoRA adapters with Unsloth-optimized kernels"]], [Plain [Code ("", [], []) "FastLanguageModel.for_inference()", Str ": Enables 2x faster native inference after training"]]], Header 3 ("training-methods", ["unnumbered", "unlisted"], []) [Str "Training Methods"], Para [Str "Unsloth supports multiple training approaches ", Str "[", Str "2", Str "]", Str ":"], BulletList [[Plain [Strong [Str "QLoRA (4-bit)"], Str ": Default recommended mode; loads model quantized to 4-bit, trains LoRA adapters in 16-bit"]], [Plain [Strong [Str "LoRA (16-bit)"], Str ": Full 16-bit LoRA fine-tuning with ", Str "~", Str "4x more VRAM than QLoRA"]], [Plain [Strong [Str "Full Fine-Tuning"], Str ": Updates all model parameters; most compute-intensive"]], [Plain [Strong [Str "Continued Pretraining"], Str ": Extend base model training on domain-specific corpora"]]], Header 3 ("reinforcement-learning", ["unnumbered", "unlisted"], []) [Str "Reinforcement Learning"], Para [Str "Unsloth provides the most memory-efficient RL implementation, using up to 90% less VRAM than standard implementations with Flash Attention 2. Supported methods ", Str "[", Str "3", Str "]", Str ":"], BulletList [[Plain [Strong [Str "GRPO (Group Relative Policy Optimization)"], Str ": DeepSeek's method that removes both value and reward models, using statistical sampling across multiple outputs to estimate advantages via Z-score standardization"]], [Plain [Strong [Str "PPO (Proximal Policy Optimization)"], Str ": Traditional three-component system with generating policy, reference policy, and value model"]], [Plain [Strong [Str "RLHF (Reinforcement Learning from Human Feedback)"], Str ": Training agents to produce outputs rated useful by human evaluators"]], [Plain [Strong [Str "RLVR (Reinforcement Learning with Verifiable Rewards)"], Str ": Rewards based on tasks with verifiable solutions (math, code)"]], [Plain [Strong [Str "GSPO, DR-GRPO"], Str ": Additional variants accessible via ", Code ("", [], []) "GRPOConfig", Str " parameters"]]], Header 3 ("dynamic-20-ggufs", ["unnumbered", "unlisted"], []) [Str "Dynamic 2.0 GGUFs"], Para [Str "Unsloth's Dynamic 2.0 quantization system intelligently varies quantization types per layer and per model, rather than applying uniform quantization. It uses a calibration dataset of 1.5+ million hand-curated tokens optimized for conversational performance. Available formats include IQ1_S through Q5_1 ", Str "[", Str "5", Str "]", Str "."], Header 3 ("vision-fine-tuning", ["unnumbered", "unlisted"], []) [Str "Vision Fine-Tuning"], Para [Str "Unsloth supports fine-tuning Vision-Language Models (VLMs) including Qwen3-VL, Gemma 3, Llama 3.2 Vision, and Qwen2.5 VL. Users can selectively fine-tune vision layers, language layers, attention modules, or MLP modules independently ", Str "[", Str "7", Str "]", Str "."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], CodeBlock ("", [""], []) "┌─────────────────────────────────────────────────────┐
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
", Para [Str "Unsloth sits between the Hugging Face ecosystem (Transformers, PEFT, TRL) and optimized Triton kernels. The framework intercepts standard training operations and replaces them with memory-efficient implementations while maintaining mathematical equivalence ", Str "[", Str "1", Str "]", Str "[", Str "2", Str "]", Str "."], Header 2 ("key-features", ["unnumbered", "unlisted"], []) [Str "Key Features"], BulletList [[Plain [Strong [Str "2x Faster Training"], Str ": Optimized Triton kernels for training with zero accuracy loss"]], [Plain [Strong [Str "70-90% Less VRAM"], Str ": Memory-efficient implementations for both SFT and RL training"]], [Plain [Strong [Str "500+ Model Support"], Str ": Text, vision, TTS, embedding, and MoE models from Llama, Qwen, Gemma, DeepSeek, Mistral, Phi, and more"]], [Plain [Strong [Str "QLoRA/LoRA/Full Fine-Tuning"], Str ": 4-bit, 8-bit, 16-bit, and full precision training modes"]], [Plain [Strong [Str "Reinforcement Learning"], Str ": GRPO, PPO, RLHF, RLVR, GSPO, DR-GRPO with up to 90% VRAM reduction"]], [Plain [Strong [Str "Vision Fine-Tuning"], Str ": Selective layer fine-tuning for VLMs (vision, language, attention, MLP)"]], [Plain [Strong [Str "Text-to-Speech Fine-Tuning"], Str ": TTS model training support"]], [Plain [Strong [Str "Embedding Fine-Tuning"], Str ": Train custom embedding models"]], [Plain [Strong [Str "Dynamic 2.0 GGUFs"], Str ": Intelligent per-layer quantization with custom calibration datasets"]], [Plain [Strong [Str "Ultra Long Context RL"], Str ": 500K+ context length fine-tuning support"]], [Plain [Strong [Str "Multi-GPU Training"], Str ": Distributed training across multiple GPUs"]], [Plain [Strong [Str "Faster MoE Training"], Str ": 12x faster Mixture-of-Experts training with less VRAM"]], [Plain [Strong [Str "GGUF Export"], Str ": Direct conversion for Ollama, llama.cpp, and LM Studio deployment"]], [Plain [Strong [Str "vLLM Integration"], Str ": Enterprise deployment with FP8/AWQ quantization and LoRA hot-swapping"]], [Plain [Strong [Str "2x Faster Inference"], Str ": Native accelerated inference via ", Code ("", [], []) "for_inference()"]], [Plain [Strong [Str "Chat Templates"], Str ": Flexible template system for conversation formatting"]], [Plain [Strong [Str "Ready-to-Use Notebooks"], Str ": Google Colab notebooks for all supported models"]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Instruction Tuning"], Str ": Fine-tune base models to follow instructions using SFTTrainer with alpaca or chat formats"]], [Plain [Strong [Str "Reasoning Model Training"], Str ": Use GRPO to train reasoning capabilities similar to DeepSeek-R1 approach"]], [Plain [Strong [Str "Domain Adaptation"], Str ": Continue pretraining on domain-specific data then fine-tune for specialized tasks"]], [Plain [Strong [Str "Medical Imaging"], Str ": Fine-tune Llama 3.2 Vision on radiography and other medical imaging datasets"]], [Plain [Strong [Str "Document Analysis"], Str ": Train VLMs for handwriting-to-LaTeX conversion and document understanding"]], [Plain [Strong [Str "Local LLM Deployment"], Str ": Train, quantize to GGUF, and deploy via Ollama or llama.cpp for local inference"]], [Plain [Strong [Str "Enterprise Serving"], Str ": Fine-tune and deploy via vLLM with LoRA hot-swapping for multi-tenant systems"]], [Plain [Strong [Str "Preference Alignment"], Str ": Align models with human preferences using DPO, ORPO, or GRPO"]], [Plain [Strong [Str "Code Generation"], Str ": Fine-tune coding models using RL with verifiable rewards from code execution"]]], Header 2 ("api-reference", ["unnumbered", "unlisted"], []) [Str "API Reference"], Header 3 ("model-loading", ["unnumbered", "unlisted"], []) [Str "Model Loading"], CodeBlock ("", ["python"], []) "from unsloth import FastLanguageModel

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name=\"unsloth/Llama-3.1-8B-bnb-4bit\",
    max_seq_length=2048,
    dtype=None,            # Auto-detect; or torch.float16/bfloat16
    load_in_4bit=True,     # QLoRA mode
)
", Header 3 ("lora-configuration", ["unnumbered", "unlisted"], []) [Str "LoRA Configuration"], CodeBlock ("", ["python"], []) "model = FastLanguageModel.get_peft_model(
    model,
    r=16,
    target_modules=[\"q_proj\", \"k_proj\", \"v_proj\", \"o_proj\",
                     \"gate_proj\", \"up_proj\", \"down_proj\"],
    lora_alpha=16,
    lora_dropout=0,
    bias=\"none\",
    use_rslora=False,
    use_gradient_checkpointing=\"unsloth\",
)
", Header 3 ("supervised-fine-tuning", ["unnumbered", "unlisted"], []) [Str "Supervised Fine-Tuning"], CodeBlock ("", ["python"], []) "from trl import SFTTrainer
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
        output_dir=\"outputs\",
    ),
)
trainer.train()
", Header 3 ("grpo-training", ["unnumbered", "unlisted"], []) [Str "GRPO Training"], CodeBlock ("", ["python"], []) "from trl import GRPOConfig, GRPOTrainer

training_args = GRPOConfig(
    learning_rate=5e-6,
    num_generations=8,
    max_completion_length=256,
    per_device_train_batch_size=1,
    gradient_accumulation_steps=4,
    output_dir=\"grpo_outputs\",
)

trainer = GRPOTrainer(
    model=model,
    processing_class=tokenizer,
    reward_funcs=[correctness_reward_func, format_reward_func],
    args=training_args,
    train_dataset=dataset,
)
trainer.train()
", Header 3 ("inference", ["unnumbered", "unlisted"], []) [Str "Inference"], CodeBlock ("", ["python"], []) "FastLanguageModel.for_inference(model)

inputs = tokenizer([\"What is AI?\"], return_tensors=\"pt\").to(\"cuda\")
outputs = model.generate(**inputs, max_new_tokens=128)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
", Header 3 ("export-and-saving", ["unnumbered", "unlisted"], []) [Str "Export and Saving"], CodeBlock ("", ["python"], []) "# Save LoRA adapter (~100MB)
model.save_pretrained(\"lora_model\")

# Save to GGUF for Ollama/llama.cpp
model.save_pretrained_gguf(\"model_gguf\", tokenizer, quantization_method=\"q4_k_m\")

# Push to Hugging Face Hub
model.push_to_hub(\"username/model-name\", token=\"hf_...\")
model.push_to_hub_gguf(\"username/model-gguf\", tokenizer, quantization_method=\"q4_k_m\", token=\"hf_...\")
", Header 2 ("configuration", ["unnumbered", "unlisted"], []) [Str "Configuration"], Header 3 ("model-loading-options", ["unnumbered", "unlisted"], []) [Str "Model Loading Options"], BulletList [[Plain [Code ("", [], []) "model_name", Str ": Hugging Face model ID or local path"]], [Plain [Code ("", [], []) "max_seq_length", Str ": Maximum context length (default: 2048)"]], [Plain [Code ("", [], []) "dtype", Str ": ", Code ("", [], []) "None", Str " (auto), ", Code ("", [], []) "torch.float16", Str ", or ", Code ("", [], []) "torch.bfloat16"]], [Plain [Code ("", [], []) "load_in_4bit", Str ": Enable 4-bit QLoRA (default: True)"]], [Plain [Code ("", [], []) "load_in_16bit", Str ": Enable 16-bit LoRA"]], [Plain [Code ("", [], []) "full_finetuning", Str ": Enable full parameter fine-tuning"]]], Header 3 ("lora-parameters", ["unnumbered", "unlisted"], []) [Str "LoRA Parameters"], BulletList [[Plain [Code ("", [], []) "r", Str ": LoRA rank (8, 16, 32, 64 common values)"]], [Plain [Code ("", [], []) "lora_alpha", Str ": Scaling factor (typically equal to r)"]], [Plain [Code ("", [], []) "lora_dropout", Str ": Dropout rate (0 recommended for Unsloth)"]], [Plain [Code ("", [], []) "target_modules", Str ": List of modules to apply LoRA"]], [Plain [Code ("", [], []) "use_rslora", Str ": Rank-stabilized LoRA scaling"]], [Plain [Code ("", [], []) "use_gradient_checkpointing", Str ": ", Code ("", [], []) "\"unsloth\"", Str " for optimized checkpointing"]]], Header 3 ("vram-requirements", ["unnumbered", "unlisted"], []) [Str "VRAM Requirements"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Model Size"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "QLoRA (4-bit)"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "LoRA (16-bit)"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "3B"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "3.5 GB"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "8 GB"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "7-8B"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "5 GB"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "19 GB"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "14B"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "10 GB"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "38 GB"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "70B"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "41 GB"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "164 GB"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "405B"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "237 GB"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "950 GB"]]]])] (TableFoot ("", [], []) []), Header 3 ("environment-flags", ["unnumbered", "unlisted"], []) [Str "Environment Flags"], Para [Str "Unsloth provides environment flags for controlling behavior (logging, memory management, kernel selection) via the ", Code ("", [], []) "UNSLOTH_*", Str " environment variable prefix ", Str "[", Str "1", Str "]", Str "."], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("ollama-deployment", ["unnumbered", "unlisted"], []) [Str "Ollama Deployment"], Para [Str "Export trained models to GGUF format, then serve via Ollama for local inference with ", Code ("", [], []) "ollama run", Str " ", Str "[", Str "4", Str "]", Str "."], Header 3 ("vllm-serving", ["unnumbered", "unlisted"], []) [Str "vLLM Serving"], Para [Str "Deploy fine-tuned models via vLLM for enterprise serving with LoRA hot-swapping — swap adapters without reloading the base model ", Str "[", Str "4", Str "]", Str "."], Header 3 ("hugging-face-trl", ["unnumbered", "unlisted"], []) [Str "Hugging Face TRL"], Para [Str "Unsloth uses TRL's ", Code ("", [], []) "SFTTrainer", Str ", ", Code ("", [], []) "GRPOTrainer", Str ", ", Code ("", [], []) "DPOTrainer", Str ", and ", Code ("", [], []) "ORPOTrainer", Str " directly. The optimization is transparent — standard TRL code works with Unsloth models ", Str "[", Str "2", Str "]", Str "[", Str "3", Str "]", Str "."], Header 3 ("llamacpp-and-lm-studio", ["unnumbered", "unlisted"], []) [Str "llama.cpp and LM Studio"], Para [Str "GGUF exports are directly compatible with llama.cpp for CLI inference and LM Studio for GUI-based local inference ", Str "[", Str "4", Str "]", Str "."], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("quick-lora-fine-tuning", ["unnumbered", "unlisted"], []) [Str "Quick LoRA Fine-Tuning"], CodeBlock ("", ["python"], []) "from unsloth import FastLanguageModel
from trl import SFTTrainer
from transformers import TrainingArguments
from datasets import load_dataset

# Load model
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name=\"unsloth/Llama-3.1-8B-bnb-4bit\",
    max_seq_length=2048,
    load_in_4bit=True,
)

# Apply LoRA
model = FastLanguageModel.get_peft_model(
    model, r=16, target_modules=[\"q_proj\", \"k_proj\", \"v_proj\", \"o_proj\",
                                  \"gate_proj\", \"up_proj\", \"down_proj\"],
    lora_alpha=16, lora_dropout=0,
)

# Train
dataset = load_dataset(\"yahma/alpaca-cleaned\", split=\"train\")
trainer = SFTTrainer(
    model=model, tokenizer=tokenizer, train_dataset=dataset,
    args=TrainingArguments(
        per_device_train_batch_size=2,
        gradient_accumulation_steps=4,
        num_train_epochs=1,
        learning_rate=2e-4,
        output_dir=\"outputs\",
    ),
)
trainer.train()

# Export to GGUF
model.save_pretrained_gguf(\"model_gguf\", tokenizer, quantization_method=\"q4_k_m\")
", Header 3 ("vision-fine-tuning-1", ["unnumbered", "unlisted"], []) [Str "Vision Fine-Tuning"], CodeBlock ("", ["python"], []) "from unsloth import FastVisionModel

model, tokenizer = FastVisionModel.from_pretrained(
    \"unsloth/Qwen2-VL-7B-Instruct-bnb-4bit\",
    load_in_4bit=True,
)

model = FastVisionModel.get_peft_model(
    model, r=16,
    finetune_vision_layers=True,
    finetune_language_layers=True,
    finetune_attention_modules=True,
    finetune_mlp_modules=True,
)
", Header 2 ("limitations", ["unnumbered", "unlisted"], []) [Str "Limitations"], BulletList [[Plain [Strong [Str "NVIDIA GPU focused"], Str ": Requires CUDA Capability 7.0+; AMD and Intel support available but less mature ", Str "[", Str "6", Str "]"]], [Plain [Strong [Str "No Apple Silicon support"], Str ": macOS/M-series GPU support is not yet available ", Str "[", Str "6", Str "]"]], [Plain [Strong [Str "Single GPU default"], Str ": Multi-GPU training works but an improved version is still in development ", Str "[", Str "1", Str "]"]], [Plain [Strong [Str "TRL dependency"], Str ": Training workflows depend on Hugging Face TRL; custom training loops require more manual integration"]], [Plain [Strong [Str "RL minimum model size"], Str ": Reasoning token generation requires minimum 1.5B parameter models ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "RL convergence time"], Str ": GRPO training requires minimum 300 steps before meaningful reward increases ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "Memory for large models"], Str ": Despite optimizations, 70B+ models still require 41GB+ VRAM even with QLoRA ", Str "[", Str "6", Str "]"]]], Header 2 ("changelog", ["unnumbered", "unlisted"], []) [Str "Changelog"], BulletList [[Plain [Strong [Str "Dynamic 2.0 GGUFs"], Str ": Intelligent per-layer quantization with 1.5M+ token calibration dataset"]], [Plain [Strong [Str "Faster MoE Training"], Str ": 12x faster Mixture-of-Experts training with less VRAM"]], [Plain [Strong [Str "Ultra Long Context RL"], Str ": 500K+ context length support for GRPO training"]], [Plain [Strong [Str "Embedding Fine-Tuning"], Str ": Custom embedding model training"]], [Plain [Strong [Str "TTS Fine-Tuning"], Str ": Text-to-speech model training support"]], [Plain [Strong [Str "Vision Fine-Tuning"], Str ": Selective layer fine-tuning for VLMs"]], [Plain [Strong [Str "GRPO/GSPO/DR-GRPO"], Str ": Multiple RL method variants with 90% VRAM reduction"]], [Plain [Strong [Str "3x Faster Training"], Str ": Packing optimizations for additional speedup"]], [Plain [Strong [Str "Quantization-Aware Training"], Str ": QAT support for training-time quantization"]], [Plain [Strong [Str "Blackwell/RTX 50 Support"], Str ": Compatibility with NVIDIA Blackwell architecture"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " Unsloth Documentation Home - https://unsloth.ai/docs"]], [Plain [Str "[", Str "2", Str "]", Str " Fine-tuning Guide - https://unsloth.ai/docs/get-started/fine-tuning-llms-guide"]], [Plain [Str "[", Str "3", Str "]", Str " Reinforcement Learning Guide - https://unsloth.ai/docs/get-started/reinforcement-learning-rl-guide"]], [Plain [Str "[", Str "4", Str "]", Str " Inference & Deployment - https://unsloth.ai/docs/basics/inference-and-deployment"]], [Plain [Str "[", Str "5", Str "]", Str " Dynamic 2.0 GGUFs - https://unsloth.ai/docs/basics/unsloth-dynamic-2.0-ggufs"]], [Plain [Str "[", Str "6", Str "]", Str " System Requirements - https://unsloth.ai/docs/get-started/fine-tuning-for-beginners/unsloth-requirements"]], [Plain [Str "[", Str "7", Str "]", Str " Vision Fine-tuning - https://unsloth.ai/docs/basics/vision-fine-tuning"]], [Plain [Str "[", Str "8", Str "]", Str " Installation - https://unsloth.ai/docs/get-started/install"]]]]