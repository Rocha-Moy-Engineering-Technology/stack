[Header 1 ("peft", [], []) [Str "PEFT"], BlockQuote [Para [Str "HuggingFace library for parameter-efficient fine-tuning with LoRA and adapter methods"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "PEFT"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Fine-tuning"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "huggingface/peft"] ("https://github.com/huggingface/peft", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "20720"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Docs"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "huggingface.co/docs/peft"] ("https://huggingface.co/docs/peft/en/index", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "PEFT (Parameter-Efficient Fine-Tuning) is a Hugging Face library for adapting large pretrained models to downstream tasks by training only a small number of extra parameters instead of all model weights. This dramatically reduces computational and storage costs while yielding performance comparable to full fine-tuning ", Str "[", Str "1", Str "]", Str "."], Para [Str "PEFT is integrated with the Hugging Face Transformers, Diffusers, and Accelerate libraries. It supports training with the Transformers Trainer, Accelerate, or custom PyTorch training loops. Trained adapters are stored as small files (e.g., 6MB for a LoRA adapter on a 350M model vs. 700MB for the full model) and can be loaded, swapped, and merged at inference time ", Str "[", Str "2", Str "]", Str "."], Para [Str "The library supports two broad categories of methods: ", Strong [Str "adapter-based methods"], Str " (LoRA, AdaLoRA, LoHa, LoKr, OFT, BOFT, HRA, MiSS, Llama-Adapter) that add trainable parameters to frozen model layers, and ", Strong [Str "soft prompting methods"], Str " (Prompt Tuning, Prefix Tuning, P-Tuning, Multitask Prompt Tuning, CPT) that prepend learnable tokens to model inputs ", Str "[", Str "3", Str "]", Str "[", Str "4", Str "]", Str "."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("low-rank-adaptation-lora", ["unnumbered", "unlisted"], []) [Str "Low-Rank Adaptation (LoRA)"], Para [Str "LoRA decomposes weight updates into two smaller low-rank matrices (A and B) instead of modifying the full weight matrix. The original weights remain frozen, and only the low-rank matrices are trained. Key parameters ", Str "[", Str "5", Str "]", Str "[", Str "6", Str "]", Str ":"], BulletList [[Plain [Strong [Str "r"], Str ": Rank dimension of the decomposition (higher = more parameters, more capacity)"]], [Plain [Strong [Str "lora_alpha"], Str ": Scaling factor (effective scaling is ", Code ("", [], []) "lora_alpha/r", Str ", or ", Code ("", [], []) "lora_alpha/sqrt(r)", Str " with rsLoRA)"]], [Plain [Strong [Str "target_modules"], Str ": Which layers to apply LoRA to (e.g., ", Code ("", [], []) "\"all-linear\"", Str " for QLoRA-style)"]], [Plain [Strong [Str "lora_dropout"], Str ": Dropout probability for LoRA layers"]]], Para [Str "LoRA adapters can be merged into the base model via ", Code ("", [], []) "merge_and_unload()", Str " to eliminate inference latency, or kept separate for swapping between tasks ", Str "[", Str "6", Str "]", Str "."], Header 3 ("lora-variants", ["unnumbered", "unlisted"], []) [Str "LoRA Variants"], Para [Str "PEFT supports many LoRA initialization and optimization strategies ", Str "[", Str "5", Str "]", Str "[", Str "6", Str "]", Str ":"], BulletList [[Plain [Strong [Str "DoRA (Weight-Decomposed Low-Rank Adaptation)"], Str ": Separates weight updates into magnitude and direction components, improving performance especially at low ranks"]], [Plain [Strong [Str "rsLoRA (Rank-Stabilized LoRA)"], Str ": Uses ", Code ("", [], []) "lora_alpha/sqrt(r)", Str " scaling for more stable training at higher ranks"]], [Plain [Strong [Str "PiSSA"], Str ": Initializes LoRA from principal singular values for faster convergence"]], [Plain [Strong [Str "OLoRA"], Str ": Uses QR decomposition initialization for improved stability"]], [Plain [Strong [Str "EVA (Explained Variance Adaptation)"], Str ": Data-driven initialization via SVD of layer input activations with adaptive rank redistribution"]], [Plain [Strong [Str "CorDA (Context-Oriented Decomposition Adaptation)"], Str ": Task-aware initialization with instruction-previewed or knowledge-preserved modes"]], [Plain [Strong [Str "LoftQ"], Str ": Initializes LoRA to minimize quantization error for QLoRA training"]], [Plain [Strong [Str "aLoRA (Activated LoRA)"], Str ": Selectively activates adapters only on tokens after an invocation sequence, enabling KV cache reuse"]]], Header 3 ("other-adapter-methods", ["unnumbered", "unlisted"], []) [Str "Other Adapter Methods"], BulletList [[Plain [Strong [Str "AdaLoRA"], Str ": Adaptively allocates rank across layers based on importance scoring via SVD-like parameterization ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "LoHa"], Str ": Uses Hadamard product of four low-rank matrices for higher expressivity at the same parameter count ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "LoKr"], Str ": Uses Kronecker product decomposition preserving rank of original weights ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "OFT (Orthogonal Finetuning)"], Str ": Learns orthogonal transformations preserving cosine similarity between neurons ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "BOFT (Orthogonal Butterfly)"], Str ": Factorizes orthogonal transformation into sparse butterfly matrices with O(d log d) parameters ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "HRA (Householder Reflection Adaptation)"], Str ": Chains trainable Householder reflections bridging LoRA and OFT ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "MiSS (Matrix Shard Sharing)"], Str ": Uses a single trainable matrix with shard-sharing mechanism ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "X-LoRA"], Str ": Mixture of LoRA experts with dynamic gating for token-level adapter activation ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "Llama-Adapter"], Str ": Zero-initialized attention with learnable adaption prompts for upper model layers ", Str "[", Str "3", Str "]"]]], Header 3 ("soft-prompting-methods", ["unnumbered", "unlisted"], []) [Str "Soft Prompting Methods"], BulletList [[Plain [Strong [Str "Prompt Tuning"], Str ": Adds learnable prompt tokens to model input embeddings; model parameters remain frozen ", Str "[", Str "4", Str "]"]], [Plain [Strong [Str "Prefix Tuning"], Str ": Inserts trainable prefix parameters into all model layers (not just input), optimized via a feed-forward network ", Str "[", Str "4", Str "]"]], [Plain [Strong [Str "P-Tuning"], Str ": Learnable prompt tokens insertable anywhere in the input sequence, optimized by a bidirectional LSTM encoder ", Str "[", Str "4", Str "]"]], [Plain [Strong [Str "Multitask Prompt Tuning"], Str ": Learns a single shared prompt from multiple tasks via Hadamard product decomposition ", Str "[", Str "4", Str "]"]], [Plain [Strong [Str "CPT (Context-Aware Prompt Tuning)"], Str ": Refines context embeddings for few-shot classification with controlled perturbations ", Str "[", Str "4", Str "]"]]], Header 3 ("quantization-support", ["unnumbered", "unlisted"], []) [Str "Quantization Support"], Para [Str "PEFT works with quantized models via multiple backends ", Str "[", Str "7", Str "]", Str ":"], BulletList [[Plain [Strong [Str "bitsandbytes"], Str ": 4-bit and 8-bit quantization (QLoRA)"]], [Plain [Strong [Str "GPTQ"], Str ": 2/3/4/8-bit post-training quantization"]], [Plain [Strong [Str "AWQ"], Str ": Activation-aware weight quantization"]], [Plain [Strong [Str "AQLM"], Str ": Additive quantization down to 2-bit"]], [Plain [Strong [Str "EETQ"], Str ": Efficient 8-bit quantization"]], [Plain [Strong [Str "HQQ"], Str ": Half-Quadratic Quantization"]], [Plain [Strong [Str "torchao"], Str ": PyTorch native int8 quantization"]], [Plain [Strong [Str "INC"], Str ": Intel Neural Compressor for FP8 on HPU devices"]]], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], CodeBlock ("", [""], []) "┌─────────────────────────────────────────────────────┐
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
", Para [Str "PEFT wraps any base model (Transformers, Diffusers, or custom PyTorch) with a PeftModel that injects trainable adapter layers while keeping the base frozen. Only adapter weights are saved and loaded. Multiple adapters can coexist on the same base model and be activated, swapped, or merged independently ", Str "[", Str "2", Str "]", Str "[", Str "6", Str "]", Str "."], Header 2 ("key-features", ["unnumbered", "unlisted"], []) [Str "Key Features"], BulletList [[Plain [Strong [Str "15+ PEFT Methods"], Str ": LoRA, DoRA, AdaLoRA, LoHa, LoKr, OFT, BOFT, HRA, MiSS, X-LoRA, Llama-Adapter, Prompt Tuning, Prefix Tuning, P-Tuning, Multitask Prompt Tuning, CPT"]], [Plain [Strong [Str "7+ LoRA Initialization Strategies"], Str ": Default, Gaussian, PiSSA, OLoRA, EVA, CorDA, LoftQ, orthogonal"]], [Plain [Strong [Str "Adapter Merging"], Str ": Merge multiple LoRA adapters via SVD, linear combination, TIES, DARE, magnitude pruning, or concatenation"]], [Plain [Strong [Str "Adapter Swapping"], Str ": Load, activate, and switch between multiple adapters at inference time without reloading the base model"]], [Plain [Strong [Str "Mixed-Adapter Batches"], Str ": Use different LoRA adapters for different samples in the same batch via ", Code ("", [], []) "adapter_names"]], [Plain [Strong [Str "Weight Merging"], Str ": Merge adapters into base model via ", Code ("", [], []) "merge_and_unload()", Str " for zero-overhead inference"]], [Plain [Strong [Str "8+ Quantization Backends"], Str ": bitsandbytes (QLoRA), GPTQ, AWQ, AQLM, EETQ, HQQ, torchao, INC"]], [Plain [Strong [Str "Specialized Optimizers"], Str ": LoRA-FA (fixed A matrix) and LoRA+ (differential learning rates for A and B)"]], [Plain [Strong [Str "Trainable Token Indices"], Str ": Efficiently fine-tune specific embedding tokens alongside LoRA"]], [Plain [Strong [Str "Per-Layer Rank Control"], Str ": ", Code ("", [], []) "rank_pattern", Str " and ", Code ("", [], []) "alpha_pattern", Str " for layer-specific ranks and scaling"]], [Plain [Strong [Str "Layer Replication"], Str ": Memory-efficient model expansion by duplicating layers with separate LoRA adapters"]], [Plain [Strong [Str "Arrow Routing"], Str ": Gradient-free token-wise mixture-of-experts routing across LoRA adapters"]], [Plain [Strong [Str "Integration"], Str ": Seamless with Transformers Trainer, Accelerate, DeepSpeed, FSDP, Diffusers, and Hugging Face Hub"]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "LLM Fine-Tuning"], Str ": Adapt large language models to domain-specific tasks with LoRA/QLoRA using minimal GPU memory"]], [Plain [Strong [Str "QLoRA Training"], Str ": Fine-tune 65B+ parameter models on a single 48GB GPU by combining 4-bit quantization with LoRA"]], [Plain [Strong [Str "Image Generation"], Str ": Fine-tune Stable Diffusion and FLUX models with LoRA, LoHa, or LoKr adapters via Diffusers"]], [Plain [Strong [Str "Multi-Task Adapters"], Str ": Train separate LoRA adapters for different tasks on the same base model, swapping at inference"]], [Plain [Strong [Str "Instruction Following"], Str ": Adapt base models into instruction-following assistants with Llama-Adapter or LoRA"]], [Plain [Strong [Str "Speech Recognition"], Str ": Apply adapter methods to automatic speech recognition models like Whisper"]], [Plain [Strong [Str "Classification"], Str ": Use soft prompting or LoRA for text/image classification with minimal trainable parameters"]], [Plain [Strong [Str "Adapter Composition"], Str ": Combine multiple trained LoRA adapters into new capabilities via Arrow routing or weighted merging"]]], Header 2 ("api-reference", ["unnumbered", "unlisted"], []) [Str "API Reference"], Header 3 ("training", ["unnumbered", "unlisted"], []) [Str "Training"], CodeBlock ("", ["python"], []) "from peft import LoraConfig, get_peft_model, TaskType

# Configure LoRA
config = LoraConfig(
    task_type=TaskType.CAUSAL_LM,
    r=16,
    lora_alpha=32,
    lora_dropout=0.1,
    target_modules=[\"q_proj\", \"v_proj\", \"k_proj\", \"o_proj\"],
)

# Wrap base model
model = get_peft_model(base_model, config)
model.print_trainable_parameters()
# \"trainable params: 2359296 || all params: 1231940608 || trainable%: 0.19\"
", Header 3 ("saving-and-loading", ["unnumbered", "unlisted"], []) [Str "Saving and Loading"], CodeBlock ("", ["python"], []) "# Save adapter only
model.save_pretrained(\"output_dir\")

# Push to Hub
model.push_to_hub(\"username/model-lora\")

# Load for inference
from peft import AutoPeftModelForCausalLM
model = AutoPeftModelForCausalLM.from_pretrained(\"username/model-lora\")
", Header 3 ("adapter-operations", ["unnumbered", "unlisted"], []) [Str "Adapter Operations"], CodeBlock ("", ["python"], []) "from peft import PeftModel

# Load base + adapter
model = PeftModel.from_pretrained(base_model, \"adapter-path\", adapter_name=\"sft\")

# Load additional adapter
model.load_adapter(\"another-adapter-path\", adapter_name=\"dpo\")

# Switch active adapter
model.set_adapter(\"dpo\")

# Merge into base model
model = model.merge_and_unload()

# Or merge/unmerge reversibly
model.merge_adapter()
model.unmerge_adapter()
", Header 3 ("weighted-adapter-merging", ["unnumbered", "unlisted"], []) [Str "Weighted Adapter Merging"], CodeBlock ("", ["python"], []) "model.add_weighted_adapter(
    adapters=[\"sft\", \"dpo\"],
    weights=[0.7, 0.3],
    adapter_name=\"merged\",
    combination_type=\"linear\",  # or svd, ties, dare_linear, etc.
)
", Header 2 ("configuration", ["unnumbered", "unlisted"], []) [Str "Configuration"], Header 3 ("loraconfig-parameters", ["unnumbered", "unlisted"], []) [Str "LoraConfig Parameters"], BulletList [[Plain [Code ("", [], []) "r", Str " (int): LoRA rank dimension"]], [Plain [Code ("", [], []) "lora_alpha", Str " (int): Scaling factor"]], [Plain [Code ("", [], []) "lora_dropout", Str " (float): Dropout probability (default: 0.0)"]], [Plain [Code ("", [], []) "target_modules", Str " (list/str): Modules to apply LoRA; ", Code ("", [], []) "\"all-linear\"", Str " for all linear layers"]], [Plain [Code ("", [], []) "bias", Str " (str): ", Code ("", [], []) "\"none\"", Str ", ", Code ("", [], []) "\"all\"", Str ", or ", Code ("", [], []) "\"lora_only\""]], [Plain [Code ("", [], []) "task_type", Str " (TaskType): ", Code ("", [], []) "CAUSAL_LM", Str ", ", Code ("", [], []) "SEQ_2_SEQ_LM", Str ", ", Code ("", [], []) "TOKEN_CLS", Str ", ", Code ("", [], []) "SEQ_CLS", Str ", ", Code ("", [], []) "FEATURE_EXTRACTION"]], [Plain [Code ("", [], []) "use_rslora", Str " (bool): Rank-stabilized scaling (default: False)"]], [Plain [Code ("", [], []) "use_dora", Str " (bool): Weight-Decomposed adaptation (default: False)"]], [Plain [Code ("", [], []) "init_lora_weights", Str " (str/bool): Initialization — ", Code ("", [], []) "True", Str ", ", Code ("", [], []) "\"gaussian\"", Str ", ", Code ("", [], []) "\"pissa\"", Str ", ", Code ("", [], []) "\"olora\"", Str ", ", Code ("", [], []) "\"eva\"", Str ", ", Code ("", [], []) "\"corda\"", Str ", ", Code ("", [], []) "\"loftq\"", Str ", ", Code ("", [], []) "\"orthogonal\""]], [Plain [Code ("", [], []) "rank_pattern", Str " (dict): Per-layer rank overrides via regex"]], [Plain [Code ("", [], []) "alpha_pattern", Str " (dict): Per-layer alpha overrides via regex"]], [Plain [Code ("", [], []) "modules_to_save", Str " (list): Additional modules to train and save beyond adapters"]]], Header 3 ("qlora-setup", ["unnumbered", "unlisted"], []) [Str "QLoRA Setup"], CodeBlock ("", ["python"], []) "from transformers import BitsAndBytesConfig
from peft import prepare_model_for_kbit_training

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type=\"nf4\",
    bnb_4bit_use_double_quant=True,
    bnb_4bit_compute_dtype=torch.bfloat16,
)

model = AutoModelForCausalLM.from_pretrained(model_id, quantization_config=bnb_config)
model = prepare_model_for_kbit_training(model)
model = get_peft_model(model, lora_config)
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("transformers-trainer", ["unnumbered", "unlisted"], []) [Str "Transformers Trainer"], CodeBlock ("", ["python"], []) "from transformers import Trainer, TrainingArguments

training_args = TrainingArguments(
    output_dir=\"output\",
    learning_rate=1e-3,
    per_device_train_batch_size=32,
    num_train_epochs=2,
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=dataset[\"train\"],
    processing_class=tokenizer,
)
trainer.train()
", Header 3 ("hugging-face-hub", ["unnumbered", "unlisted"], []) [Str "Hugging Face Hub"], Para [Str "Adapters are uploaded to and loaded from the Hub. Only the small adapter files (config + weights) are stored, with the base model referenced by name ", Str "[", Str "2", Str "]", Str "."], Header 3 ("deepspeed-and-fsdp", ["unnumbered", "unlisted"], []) [Str "DeepSpeed and FSDP"], Para [Str "PEFT adapters work with DeepSpeed ZeRO stages and Fully Sharded Data Parallel via Accelerate for distributed training ", Str "[", Str "1", Str "]", Str "."], Header 3 ("diffusers", ["unnumbered", "unlisted"], []) [Str "Diffusers"], Para [Str "PEFT integrates with Diffusers for training LoRA adapters on Stable Diffusion, SDXL, and FLUX models ", Str "[", Str "1", Str "]", Str "."], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("lora-on-causal-lm", ["unnumbered", "unlisted"], []) [Str "LoRA on Causal LM"], CodeBlock ("", ["python"], []) "from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import LoraConfig, get_peft_model, TaskType

model = AutoModelForCausalLM.from_pretrained(\"meta-llama/Llama-3.2-1B\")
tokenizer = AutoTokenizer.from_pretrained(\"meta-llama/Llama-3.2-1B\")

config = LoraConfig(
    task_type=TaskType.CAUSAL_LM,
    r=8,
    lora_alpha=32,
    target_modules=[\"q_proj\", \"v_proj\"],
    lora_dropout=0.05,
)

model = get_peft_model(model, config)
model.print_trainable_parameters()
# Train with Trainer or custom loop...
model.save_pretrained(\"llama-lora\")
", Header 3 ("inference-with-autopeftmodel", ["unnumbered", "unlisted"], []) [Str "Inference with AutoPeftModel"], CodeBlock ("", ["python"], []) "from peft import AutoPeftModelForCausalLM
from transformers import AutoTokenizer

model = AutoPeftModelForCausalLM.from_pretrained(\"username/llama-lora\")
tokenizer = AutoTokenizer.from_pretrained(\"meta-llama/Llama-3.2-1B\")

inputs = tokenizer(\"The capital of France is\", return_tensors=\"pt\")
outputs = model.generate(**inputs, max_new_tokens=20)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
", Header 3 ("eva-initialization", ["unnumbered", "unlisted"], []) [Str "EVA Initialization"], CodeBlock ("", ["python"], []) "from peft import LoraConfig, EvaConfig, get_peft_model, initialize_lora_eva_weights

config = LoraConfig(
    init_lora_weights=\"eva\",
    eva_config=EvaConfig(rho=2.0),
    r=16,
    target_modules=\"all-linear\",
)

model = get_peft_model(base_model, config, low_cpu_mem_usage=True)
initialize_lora_eva_weights(model, dataloader)
", Header 2 ("limitations", ["unnumbered", "unlisted"], []) [Str "Limitations"], BulletList [[Plain [Strong [Str "Soft prompts not human-readable"], Str ": Learned prompt tokens are virtual embeddings that don't correspond to real words, making interpretation difficult ", Str "[", Str "4", Str "]"]], [Plain [Strong [Str "DoRA inference overhead"], Str ": DoRA introduces larger overhead than pure LoRA during inference; weight merging is recommended for production ", Str "[", Str "6", Str "]"]], [Plain [Strong [Str "aLoRA cannot be merged"], Str ": Activated LoRA adapters cannot be merged into the base model due to selective token application ", Str "[", Str "6", Str "]"]], [Plain [Strong [Str "Mixed-adapter batches inference only"], Str ": Using different adapters per sample in a batch works only for inference, not training ", Str "[", Str "6", Str "]"]], [Plain [Strong [Str "AQLM merging not supported"], Str ": LoRA adapters trained on AQLM-quantized models cannot be merged with quantized weights ", Str "[", Str "7", Str "]"]], [Plain [Strong [Str "torchao limited support"], Str ": Only int8 weight-only quantization is fully supported; int4 and NF4 not yet available; merging only works with LoRA + int8 ", Str "[", Str "7", Str "]"]], [Plain [Strong [Str "INC no merge/unmerge"], Str ": Intel Neural Compressor quantized models do not support adapter merging ", Str "[", Str "7", Str "]"]], [Plain [Strong [Str "Adapter composition complexity"], Str ": Methods like Arrow and X-LoRA require all adapters to share the same rank and target modules ", Str "[", Str "6", Str "]"]]], Header 2 ("changelog", ["unnumbered", "unlisted"], []) [Str "Changelog"], BulletList [[Plain [Strong [Str "v0.18.0"], Str ": Current release with aLoRA (Activated LoRA), Arrow routing, GenKnowSub, MiSS, target_parameters for MoE nn.Parameter support"]], [Plain [Strong [Str "EVA"], Str ": Data-driven initialization with adaptive rank redistribution"]], [Plain [Strong [Str "CorDA"], Str ": Context-oriented decomposition with instruction-previewed and knowledge-preserved modes"]], [Plain [Strong [Str "DoRA"], Str ": Weight-decomposed adaptation with magnitude/direction separation"]], [Plain [Strong [Str "Arrow + GenKnowSub"], Str ": Modular routing and general knowledge subtraction for multi-adapter composition"]], [Plain [Strong [Str "LoRA-FA and LoRA+"], Str ": Specialized optimizers for improved LoRA training"]], [Plain [Strong [Str "CPT"], Str ": Context-Aware Prompt Tuning for few-shot classification"]], [Plain [Strong [Str "Trainable Token Indices"], Str ": Memory-efficient selective token fine-tuning alongside LoRA"]], [Plain [Strong [Str "8+ Quantization Backends"], Str ": bitsandbytes, GPTQ, AWQ, AQLM, EETQ, HQQ, torchao, INC"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " PEFT Overview - https://huggingface.co/docs/peft/en/index"]], [Plain [Str "[", Str "2", Str "]", Str " Quicktour - https://huggingface.co/docs/peft/en/quicktour"]], [Plain [Str "[", Str "3", Str "]", Str " Adapter Conceptual Guide - https://huggingface.co/docs/peft/en/conceptual_guides/adapter"]], [Plain [Str "[", Str "4", Str "]", Str " Prompting Conceptual Guide - https://huggingface.co/docs/peft/en/conceptual_guides/prompting"]], [Plain [Str "[", Str "5", Str "]", Str " LoRA API Reference - https://huggingface.co/docs/peft/en/package_reference/lora"]], [Plain [Str "[", Str "6", Str "]", Str " LoRA Developer Guide - https://huggingface.co/docs/peft/en/developer_guides/lora"]], [Plain [Str "[", Str "7", Str "]", Str " Quantization Guide - https://huggingface.co/docs/peft/en/developer_guides/quantization"]], [Plain [Str "[", Str "8", Str "]", Str " Installation - https://huggingface.co/docs/peft/en/install"]]]]