[Header 1 ("hugging-face-transformers", [], []) [Str "Hugging Face Transformers"], BlockQuote [Para [Str "Model-definition framework for state-of-the-art ML models in text/vision/audio/video with inference and training"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Inference Serving"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "https://github.com/huggingface/transformers"] ("https://github.com/huggingface/transformers", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "156821"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://huggingface.co/docs/transformers/en/index", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Hugging Face Transformers is the model-definition framework for state-of-the-art machine learning models spanning text, computer vision, audio, video, and multimodal tasks, for both inference and training. It centralizes model definitions so they are agreed upon across the ecosystem -- if a model definition is supported in Transformers, it is compatible with the majority of training frameworks (Axolotl, Unsloth, DeepSpeed, FSDP, PyTorch-Lightning), inference engines (vLLM, SGLang, TGI), and adjacent modeling libraries (llama.cpp, MLX). ", Str "[", Str "1", Str "]"], Para [Str "With over 1M+ model checkpoints on the Hugging Face Hub, Transformers provides the Pipeline API for easy inference, the Trainer API for training, and the ", Code ("", [], []) "generate", Str " method for fast text generation with LLMs and VLMs. Every model is implemented from three main classes: configuration, model, and preprocessor. ", Str "[", Str "1", Str "]"], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("pipeline", ["unnumbered", "unlisted"], []) [Str "Pipeline"], Para [Str "The Pipeline API is the simplest inference interface, supporting many ML tasks with a single function call. Pipelines handle tokenization, model inference, and postprocessing automatically:"], CodeBlock ("", ["python"], []) "from transformers import pipeline

classifier = pipeline(\"sentiment-analysis\")
result = classifier(\"I love this product!\")
# [{'label': 'POSITIVE', 'score': 0.9998}]
", Para [Str "Supported tasks include text generation, text classification, question answering, summarization, translation, image classification, object detection, automatic speech recognition, and more. ", Str "[", Str "1", Str "]"], Header 3 ("automodel-classes", ["unnumbered", "unlisted"], []) [Str "AutoModel Classes"], Para [Str "AutoModel classes automatically detect and load the correct model architecture based on the model name or path:"], BulletList [[Plain [Code ("", [], []) "AutoModelForCausalLM", Str " -- Causal language models (GPT, Llama, Mistral)"]], [Plain [Code ("", [], []) "AutoModelForSequenceClassification", Str " -- Text classification"]], [Plain [Code ("", [], []) "AutoModelForTokenClassification", Str " -- Named entity recognition"]], [Plain [Code ("", [], []) "AutoModelForQuestionAnswering", Str " -- Extractive QA"]], [Plain [Code ("", [], []) "AutoModelForSeq2SeqLM", Str " -- Encoder-decoder models (T5, BART)"]], [Plain [Code ("", [], []) "AutoModel", Str " -- Base model without task head ", Str "[", Str "1", Str "]"]]], Header 3 ("tokenizer", ["unnumbered", "unlisted"], []) [Str "Tokenizer"], Para [Str "Tokenizers convert text to model-compatible token IDs and back. AutoTokenizer loads the correct tokenizer for any model:"], CodeBlock ("", ["python"], []) "from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained(\"meta-llama/Llama-3.1-8B-Instruct\")
tokens = tokenizer(\"Hello, world!\", return_tensors=\"pt\")
", Para [Str "[", Str "1", Str "]"], Header 3 ("trainer", ["unnumbered", "unlisted"], []) [Str "Trainer"], Para [Str "The Trainer API provides a comprehensive training loop supporting mixed precision, gradient accumulation, distributed training, evaluation, and logging. It handles the complexity of training modern transformer models. ", Str "[", Str "1", Str "]"], Header 3 ("generate", ["unnumbered", "unlisted"], []) [Str "Generate"], Para [Str "The ", Code ("", [], []) "generate", Str " method provides fast text generation for LLMs and VLMs with support for multiple decoding strategies (greedy, sampling, beam search, contrastive), streaming, and KV cache optimization. ", Str "[", Str "1", Str "]"], Header 2 ("installation-and-setup", ["unnumbered", "unlisted"], []) [Str "Installation and Setup"], Header 3 ("pip-install", ["unnumbered", "unlisted"], []) [Str "pip Install"], CodeBlock ("", ["bash"], []) "pip install transformers
", Header 3 ("with-framework-backends", ["unnumbered", "unlisted"], []) [Str "With Framework Backends"], CodeBlock ("", ["bash"], []) "# PyTorch (most common)
pip install transformers[torch]

# TensorFlow
pip install transformers[tf-cpu]   # CPU only
pip install transformers[tf]       # With GPU support

# JAX/Flax
pip install transformers[flax]
", Header 3 ("from-source", ["unnumbered", "unlisted"], []) [Str "From Source"], CodeBlock ("", ["bash"], []) "pip install git+https://github.com/huggingface/transformers
", Header 3 ("additional-dependencies", ["unnumbered", "unlisted"], []) [Str "Additional Dependencies"], CodeBlock ("", ["bash"], []) "# For tokenizers
pip install transformers[sentencepiece]

# For audio
pip install transformers[audio]

# For vision
pip install transformers[vision]
", Para [Str "[", Str "1", Str "]"], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Header 3 ("design-principles", ["unnumbered", "unlisted"], []) [Str "Design Principles"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Three classes per model"], Str " -- Configuration (hyperparameters), Model (architecture), and Preprocessor (tokenizer/feature extractor)"]], [Plain [Strong [Str "Pretrained models"], Str " -- Every model loads pretrained weights for immediate use"]], [Plain [Strong [Str "Framework agnostic"], Str " -- Core model definitions work across PyTorch, TensorFlow, and JAX"]]], Header 3 ("model-architecture", ["unnumbered", "unlisted"], []) [Str "Model Architecture"], CodeBlock ("", [""], []) "Configuration (config.json)
    └── Model (model weights)
        └── Preprocessor (tokenizer/feature extractor)
", Para [Str "Each model is self-contained with its configuration, weights, and preprocessing requirements. The Hub stores all three components together. ", Str "[", Str "1", Str "]"], Header 3 ("hub-integration", ["unnumbered", "unlisted"], []) [Str "Hub Integration"], Para [Str "Transformers tightly integrates with the Hugging Face Hub for model discovery, downloading, sharing, and versioning. Models are identified by ", Code ("", [], []) "organization/model-name", Str " and automatically downloaded on first use. ", Str "[", Str "1", Str "]"], Header 3 ("ecosystem-pivot", ["unnumbered", "unlisted"], []) [Str "Ecosystem Pivot"], Para [Str "Transformers serves as the central model definition that other tools build upon:"], BulletList [[Plain [Strong [Str "Training"], Str ": Axolotl, Unsloth, DeepSpeed, FSDP reference Transformers model definitions"]], [Plain [Strong [Str "Inference"], Str ": vLLM, SGLang, TGI use Transformers model architectures"]], [Plain [Strong [Str "Export"], Str ": llama.cpp, MLX, ONNX converters read Transformers models ", Str "[", Str "1", Str "]"]]], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Header 3 ("pipeline-api", ["unnumbered", "unlisted"], []) [Str "Pipeline API"], CodeBlock ("", ["python"], []) "from transformers import pipeline

# Text generation
generator = pipeline(\"text-generation\", model=\"meta-llama/Llama-3.1-8B-Instruct\")
output = generator(\"Once upon a time\", max_length=50)

# Image classification
classifier = pipeline(\"image-classification\", model=\"google/vit-base-patch16-224\")
result = classifier(\"image.jpg\")

# Automatic speech recognition
transcriber = pipeline(\"automatic-speech-recognition\", model=\"openai/whisper-large-v3\")
text = transcriber(\"audio.mp3\")

# Document question answering
qa = pipeline(\"document-question-answering\", model=\"impira/layoutlm-document-qa\")
answer = qa(image=\"document.png\", question=\"What is the total?\")
", Para [Str "[", Str "1", Str "]"], Header 3 ("text-generation-with-llms", ["unnumbered", "unlisted"], []) [Str "Text Generation with LLMs"], CodeBlock ("", ["python"], []) "from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

model_name = \"meta-llama/Llama-3.1-8B-Instruct\"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name, torch_dtype=torch.bfloat16, device_map=\"auto\")

messages = [
    {\"role\": \"system\", \"content\": \"You are a helpful assistant.\"},
    {\"role\": \"user\", \"content\": \"What is machine learning?\"},
]
inputs = tokenizer.apply_chat_template(messages, return_tensors=\"pt\").to(model.device)
outputs = model.generate(inputs, max_new_tokens=256)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
", Para [Str "[", Str "1", Str "]"], Header 3 ("streaming-generation", ["unnumbered", "unlisted"], []) [Str "Streaming Generation"], CodeBlock ("", ["python"], []) "from transformers import AutoModelForCausalLM, AutoTokenizer, TextStreamer

model = AutoModelForCausalLM.from_pretrained(\"meta-llama/Llama-3.1-8B-Instruct\", device_map=\"auto\")
tokenizer = AutoTokenizer.from_pretrained(\"meta-llama/Llama-3.1-8B-Instruct\")
streamer = TextStreamer(tokenizer)

inputs = tokenizer(\"Explain quantum computing:\", return_tensors=\"pt\").to(model.device)
model.generate(**inputs, streamer=streamer, max_new_tokens=200)
", Para [Str "[", Str "1", Str "]"], Header 3 ("training-with-trainer", ["unnumbered", "unlisted"], []) [Str "Training with Trainer"], CodeBlock ("", ["python"], []) "from transformers import AutoModelForSequenceClassification, TrainingArguments, Trainer

model = AutoModelForSequenceClassification.from_pretrained(\"bert-base-uncased\", num_labels=2)

training_args = TrainingArguments(
    output_dir=\"./results\",
    num_train_epochs=3,
    per_device_train_batch_size=16,
    evaluation_strategy=\"epoch\",
    learning_rate=5e-5,
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=train_dataset,
    eval_dataset=eval_dataset,
)
trainer.train()
", Para [Str "[", Str "1", Str "]"], Header 3 ("quantization", ["unnumbered", "unlisted"], []) [Str "Quantization"], CodeBlock ("", ["python"], []) "from transformers import AutoModelForCausalLM, BitsAndBytesConfig

quantization_config = BitsAndBytesConfig(load_in_4bit=True)
model = AutoModelForCausalLM.from_pretrained(
    \"meta-llama/Llama-3.1-8B-Instruct\",
    quantization_config=quantization_config,
    device_map=\"auto\",
)
", Para [Str "[", Str "1", Str "]"], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], Header 3 ("text-generation", ["unnumbered", "unlisted"], []) [Str "Text Generation"], Para [Str "Use Pipeline or AutoModelForCausalLM for chatbots, content generation, code completion, and summarization with pretrained LLMs. ", Str "[", Str "1", Str "]"], Header 3 ("computer-vision", ["unnumbered", "unlisted"], []) [Str "Computer Vision"], Para [Str "Image classification, object detection, image segmentation, and image generation using vision transformer models. ", Str "[", Str "1", Str "]"], Header 3 ("audio-processing", ["unnumbered", "unlisted"], []) [Str "Audio Processing"], Para [Str "Speech recognition, audio classification, and text-to-speech using audio transformer models like Whisper. ", Str "[", Str "1", Str "]"], Header 3 ("fine-tuning", ["unnumbered", "unlisted"], []) [Str "Fine-Tuning"], Para [Str "Adapt pretrained models to domain-specific tasks using the Trainer API with custom datasets. ", Str "[", Str "1", Str "]"], Header 3 ("feature-extraction", ["unnumbered", "unlisted"], []) [Str "Feature Extraction"], Para [Str "Generate embeddings for semantic search, clustering, and similarity using model hidden states or dedicated embedding models. ", Str "[", Str "1", Str "]"], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("key-classes", ["unnumbered", "unlisted"], []) [Str "Key Classes"], BulletList [[Plain [Code ("", [], []) "pipeline(task, model)", Str " -- Create task-specific inference pipeline"]], [Plain [Code ("", [], []) "AutoModel.from_pretrained(name)", Str " -- Load pretrained model"]], [Plain [Code ("", [], []) "AutoTokenizer.from_pretrained(name)", Str " -- Load pretrained tokenizer"]], [Plain [Code ("", [], []) "AutoConfig.from_pretrained(name)", Str " -- Load model configuration"]], [Plain [Code ("", [], []) "Trainer(model, args, train_dataset)", Str " -- Create training loop"]], [Plain [Code ("", [], []) "TrainingArguments(...)", Str " -- Configure training parameters"]]], Header 3 ("generation-methods", ["unnumbered", "unlisted"], []) [Str "Generation Methods"], BulletList [[Plain [Code ("", [], []) "model.generate(inputs, max_new_tokens, temperature, ...)", Str " -- Generate text"]], [Plain [Code ("", [], []) "TextStreamer(tokenizer)", Str " -- Stream generated tokens"]], [Plain [Code ("", [], []) "TextIteratorStreamer(tokenizer)", Str " -- Iterate over generated tokens"]]], Header 3 ("model-savingloading", ["unnumbered", "unlisted"], []) [Str "Model Saving/Loading"], BulletList [[Plain [Code ("", [], []) "model.save_pretrained(path)", Str " -- Save model locally"]], [Plain [Code ("", [], []) "model.push_to_hub(repo_id)", Str " -- Upload to HuggingFace Hub"]], [Plain [Code ("", [], []) "AutoModel.from_pretrained(path_or_hub_id)", Str " -- Load from local or Hub ", Str "[", Str "1", Str "]"]]], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("model-configuration", ["unnumbered", "unlisted"], []) [Str "Model Configuration"], BulletList [[Plain [Strong [Code ("", [], []) "torch_dtype"], Str " -- Precision (float32, float16, bfloat16)"]], [Plain [Strong [Code ("", [], []) "device_map"], Str " -- Device placement (\"auto\", \"cpu\", \"cuda:0\")"]], [Plain [Strong [Code ("", [], []) "quantization_config"], Str " -- Quantization settings (BitsAndBytes, GPTQ, AWQ)"]], [Plain [Strong [Code ("", [], []) "attn_implementation"], Str " -- Attention backend (\"flash_attention_2\", \"sdpa\")"]], [Plain [Strong [Code ("", [], []) "low_cpu_mem_usage"], Str " -- Reduce CPU memory during loading"]]], Header 3 ("generation-configuration", ["unnumbered", "unlisted"], []) [Str "Generation Configuration"], BulletList [[Plain [Strong [Code ("", [], []) "max_new_tokens"], Str " -- Maximum generated tokens"]], [Plain [Strong [Code ("", [], []) "temperature"], Str " -- Sampling temperature"]], [Plain [Strong [Code ("", [], []) "top_p"], Str " / ", Strong [Code ("", [], []) "top_k"], Str " -- Sampling parameters"]], [Plain [Strong [Code ("", [], []) "do_sample"], Str " -- Enable sampling (vs greedy)"]], [Plain [Strong [Code ("", [], []) "num_beams"], Str " -- Beam search width"]], [Plain [Strong [Code ("", [], []) "repetition_penalty"], Str " -- Penalize repeated tokens"]]], Header 3 ("training-configuration", ["unnumbered", "unlisted"], []) [Str "Training Configuration"], BulletList [[Plain [Strong [Code ("", [], []) "num_train_epochs"], Str " -- Number of training epochs"]], [Plain [Strong [Code ("", [], []) "per_device_train_batch_size"], Str " -- Batch size per GPU"]], [Plain [Strong [Code ("", [], []) "learning_rate"], Str " -- Optimizer learning rate"]], [Plain [Strong [Code ("", [], []) "fp16"], Str " / ", Strong [Code ("", [], []) "bf16"], Str " -- Mixed precision training"]], [Plain [Strong [Code ("", [], []) "gradient_accumulation_steps"], Str " -- Effective batch size multiplier ", Str "[", Str "1", Str "]"]]], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("with-inference-engines-vllm-sglang-tgi", ["unnumbered", "unlisted"], []) [Str "With Inference Engines (vLLM, SGLang, TGI)"], Para [Str "Transformers model definitions are the foundation for inference engines. Models defined in Transformers automatically work with vLLM, SGLang, and TGI."], Header 3 ("with-training-frameworks-axolotl-unsloth-deepspeed", ["unnumbered", "unlisted"], []) [Str "With Training Frameworks (Axolotl, Unsloth, DeepSpeed)"], Para [Str "Training frameworks build on Transformers models and Trainer for distributed fine-tuning."], Header 3 ("with-export-tools-onnx-llamacpp-mlx", ["unnumbered", "unlisted"], []) [Str "With Export Tools (ONNX, llama.cpp, MLX)"], Para [Str "Convert Transformers models to optimized formats for deployment on specific hardware."], Header 3 ("with-hugging-face-hub", ["unnumbered", "unlisted"], []) [Str "With Hugging Face Hub"], Para [Str "Seamless integration for model discovery, downloading, sharing, and versioning."], Header 3 ("with-datasets-library", ["unnumbered", "unlisted"], []) [Str "With Datasets Library"], Para [Str "Hugging Face Datasets integrates with Trainer for efficient data loading and preprocessing."], Header 3 ("with-peft-parameter-efficient-fine-tuning", ["unnumbered", "unlisted"], []) [Str "With PEFT (Parameter-Efficient Fine-Tuning)"], Para [Str "LoRA, QLoRA, and other PEFT methods integrate with Transformers models for efficient fine-tuning."], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("sentiment-analysis-pipeline", ["unnumbered", "unlisted"], []) [Str "Sentiment Analysis Pipeline"], CodeBlock ("", ["python"], []) "from transformers import pipeline

classifier = pipeline(\"sentiment-analysis\")
results = classifier([
    \"I love this movie!\",
    \"This was terrible.\",
])
for result in results:
    print(f\"{result['label']}: {result['score']:.4f}\")
", Para [Str "[", Str "1", Str "]"], Header 3 ("multi-turn-chat", ["unnumbered", "unlisted"], []) [Str "Multi-Turn Chat"], CodeBlock ("", ["python"], []) "from transformers import AutoModelForCausalLM, AutoTokenizer

model_name = \"meta-llama/Llama-3.1-8B-Instruct\"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name, device_map=\"auto\")

messages = [
    {\"role\": \"user\", \"content\": \"What is Python?\"},
    {\"role\": \"assistant\", \"content\": \"Python is a high-level programming language.\"},
    {\"role\": \"user\", \"content\": \"What makes it popular?\"},
]
inputs = tokenizer.apply_chat_template(messages, return_tensors=\"pt\").to(model.device)
outputs = model.generate(inputs, max_new_tokens=200)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
", Para [Str "[", Str "1", Str "]"], Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Plain [Strong [Str "Memory requirements"], Str " -- Large models require significant GPU memory; quantization or device_map=\"auto\" helps"]], [Plain [Strong [Str "Inference speed"], Str " -- Native Transformers inference is slower than optimized engines (vLLM, TGI); use Transformers for prototyping, optimized engines for production"]], [Plain [Strong [Str "Model compatibility"], Str " -- Not all model architectures are supported; new models may need community contributions"]], [Plain [Strong [Str "Framework coupling"], Str " -- While supporting PyTorch, TensorFlow, and JAX, the majority of models are PyTorch-only"]], [Plain [Strong [Str "API complexity"], Str " -- The library has a large surface area; many ways to accomplish the same task"]], [Plain [Strong [Str "Breaking changes"], Str " -- Major version updates may change APIs; pin versions for production ", Str "[", Str "1", Str "]"]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Strong [Str "v5.x"], Str " -- Current major version with latest model architectures"]], [Plain [Strong [Str "1M+ models"], Str " -- Over one million model checkpoints on HuggingFace Hub"]], [Plain [Strong [Str "Pipeline API"], Str " -- Simplified inference for dozens of tasks"]], [Plain [Strong [Str "Trainer API"], Str " -- Comprehensive training with mixed precision and distributed support"]], [Plain [Strong [Str "generate()"], Str " -- Optimized text generation with streaming and KV cache"]], [Plain [Strong [Str "Flash Attention 2"], Str " -- Hardware-accelerated attention computation"]], [Plain [Strong [Str "BitsAndBytes"], Str " -- 4-bit and 8-bit quantization for memory reduction"]], [Plain [Strong [Str "Chat templates"], Str " -- Standardized chat formatting across models"]], [Plain [Strong [Str "Vision/Audio/Video"], Str " -- Multimodal model support beyond text"]], [Plain [Strong [Str "Ecosystem pivot"], Str " -- Central model definition for training and inference tools ", Str "[", Str "1", Str "]"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " Hugging Face Transformers Documentation - ", Link ("", [], []) [Str "https://huggingface.co/docs/transformers/index"] ("https://huggingface.co/docs/transformers/index", "")]]]]