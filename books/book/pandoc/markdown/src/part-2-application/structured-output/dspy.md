[Header 1 ("dspy", [], []) [Str "DSPy"], BlockQuote [Para [Str "Stanford NLP declarative framework for programming language models. \"Iterate fast on structured code, rather than brittle strings.\" Compiles AI programs into optimized prompts and weights."]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "DSPy"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Structured Output & Prompt Engineering"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "stanfordnlp/dspy"] ("https://github.com/stanfordnlp/dspy", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "32,331"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Docs"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "dspy.ai"] ("https://dspy.ai/", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "License"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "MIT"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Contributors"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "250+"]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "DSPy replaces hand-written prompts and brittle string manipulation with a programming model for language models. Rather than crafting prompt templates, developers define typed signatures that specify input and output fields, compose them into modules, and let optimizers automatically tune the prompts and weights for a given metric. The framework treats LM calls as declarative operations, compiling high-level AI programs into efficient prompts or fine-tuning configurations. DSPy supports 40+ LM providers and delivers measurable accuracy improvements through its optimization pipeline, typically costing around $2 USD and taking approximately 20 minutes to run."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("signatures", ["unnumbered", "unlisted"], []) [Str "Signatures"], Para [Str "Signatures are typed declarations of input and output fields for a language model call. They replace free-form prompt strings with structured contracts."], CodeBlock ("", ["python"], []) "import dspy

# Inline signature: question -> answer
predict = dspy.Predict(\"question -> answer\")

# Class-based signature with typed fields
class Assess(dspy.Signature):
    \"\"\"Assess the quality of a tweet along the specified dimension.\"\"\"
    assessed_text: str = dspy.InputField()
    assessment_question: str = dspy.InputField()
    assessment_answer: float = dspy.OutputField()
", Para [Str "Signatures define what the LM should do without prescribing how it should do it. The framework handles prompt formatting, parsing, and retry logic."], Header 3 ("modules", ["unnumbered", "unlisted"], []) [Str "Modules"], Para [Str "Modules are composable building blocks that wrap signatures with specific inference strategies."], BulletList [[Plain [Strong [Str "Predict"], Str " - Direct signature invocation, the simplest module"]], [Plain [Strong [Str "ChainOfThought"], Str " - Adds intermediate reasoning steps before producing the final output"]], [Plain [Strong [Str "ReAct"], Str " - Interleaves reasoning with tool use actions in a loop"]], [Plain [Strong [Str "ProgramOfThought"], Str " - Generates and executes code to arrive at answers"]], [Plain [Strong [Str "MultiChainComparison"], Str " - Runs multiple chains and selects the best output"]], [Plain [Strong [Str "RAG"], Str " - Retrieval-augmented generation combining a retriever with a generator"]]], Para [Str "Custom modules inherit from ", Code ("", [], []) "dspy.Module", Str " and compose other modules in their ", Code ("", [], []) "forward", Str " method."], Header 3 ("optimizers", ["unnumbered", "unlisted"], []) [Str "Optimizers"], Para [Str "Optimizers automatically improve program performance against a defined metric by tuning prompts, few-shot examples, or model weights."], BulletList [[Plain [Strong [Str "BootstrapRS"], Str " - Random search over bootstrapped few-shot demonstrations"]], [Plain [Strong [Str "MIPROv2"], Str " - Multi-prompt instruction proposal with Bayesian optimization"]], [Plain [Strong [Str "GEPA"], Str " - Genetic evolutionary prompt algorithm for instruction tuning"]], [Plain [Strong [Str "BetterTogether"], Str " - Joint optimization of prompts and weights together"]]], Header 3 ("adapters", ["unnumbered", "unlisted"], []) [Str "Adapters"], Para [Str "Adapters translate signatures into the specific prompt format expected by a given LM provider. They handle serialization, field mapping, and structured output parsing transparently."], Header 2 ("installation-and-setup", ["unnumbered", "unlisted"], []) [Str "Installation and Setup"], CodeBlock ("", ["bash"], []) "pip install -U dspy
", Para [Str "For specific provider support:"], CodeBlock ("", ["bash"], []) "# With Anthropic
pip install -U dspy[anthropic]

# With Google
pip install -U dspy[google]
", Para [Str "Requires Python 3.9 or higher."], Header 3 ("quick-start", ["unnumbered", "unlisted"], []) [Str "Quick Start"], CodeBlock ("", ["python"], []) "import dspy

lm = dspy.LM(\"openai/gpt-4o-mini\")
dspy.configure(lm=lm)

qa = dspy.ChainOfThought(\"question -> answer\")
result = qa(question=\"What is the tallest mountain in the world?\")
print(result.answer)
", Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "DSPy follows a Define, Evaluate, Compile, Deploy workflow:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Define"], Str " - Write signatures and compose modules into a program"]], [Plain [Strong [Str "Evaluate"], Str " - Measure program accuracy on a development dataset using a metric function"]], [Plain [Strong [Str "Compile"], Str " - Run an optimizer that searches for better prompts, demonstrations, or weights"]], [Plain [Strong [Str "Deploy"], Str " - Use the compiled program with the tuned configuration in production"]]], CodeBlock ("", [""], []) "Signature --> Module --> Program --> Optimizer --> Compiled Program
    ^                                   ^
  Types                              Metric
  Fields                             Dataset
", Para [Str "The compilation step differentiates DSPy from standard prompt engineering. Instead of manually iterating on prompt text, the optimizer systematically explores the space of possible prompts and demonstrations, guided by the evaluation metric."], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], BulletList [[Plain [Strong [Str "Declarative LM programming"], Str " - Define what the model should compute via typed signatures rather than how via prompt strings"]], [Plain [Strong [Str "Automatic prompt optimization"], Str " - Optimizers tune prompts and few-shot examples to maximize a user-defined metric"]], [Plain [Strong [Str "Composable modules"], Str " - Chain, nest, and reuse modules like standard software components"]], [Plain [Strong [Str "Provider agnostic"], Str " - Supports 40+ LM providers through a unified interface"]], [Plain [Strong [Str "Typed input/output"], Str " - Signatures enforce structured contracts between program components"]], [Plain [Strong [Str "Reproducible optimization"], Str " - Compilation produces deterministic, serializable configurations"]], [Plain [Strong [Str "Built-in evaluation"], Str " - Integrated metric evaluation over datasets for systematic benchmarking"]], [Plain [Strong [Str "Assertion and constraint system"], Str " - ", Code ("", [], []) "dspy.Assert", Str " and ", Code ("", [], []) "dspy.Suggest", Str " enforce runtime constraints on LM outputs"]], [Plain [Strong [Str "Automatic few-shot bootstrapping"], Str " - Generates high-quality demonstrations from training data without manual curation"]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Multi-hop question answering"], Str " - Compose retrieval and reasoning modules to answer questions requiring multiple evidence steps"]], [Plain [Strong [Str "Classification pipelines"], Str " - Build typed classifiers with automatic few-shot optimization (reported improvements from 66% to 87%)"]], [Plain [Strong [Str "Agentic workflows"], Str " - Use ReAct modules for tool-augmented reasoning (reported improvements from 24% to 51% on HotPotQA)"]], [Plain [Strong [Str "Information extraction"], Str " - Define output signatures with structured fields for entity and relation extraction"]], [Plain [Strong [Str "RAG systems"], Str " - Combine retrieval modules with generation modules, optimizing the full pipeline end-to-end"]], [Plain [Strong [Str "Data labeling and assessment"], Str " - Use typed output fields (including floats and enums) for structured scoring tasks"]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("language-model-configuration", ["unnumbered", "unlisted"], []) [Str "Language Model Configuration"], CodeBlock ("", ["python"], []) "import dspy

# Configure the default LM
lm = dspy.LM(\"openai/gpt-4o-mini\", temperature=0.7)
dspy.configure(lm=lm)

# Use Anthropic
lm = dspy.LM(\"anthropic/claude-sonnet-4-20250514\")
dspy.configure(lm=lm)
", Header 3 ("signatures-1", ["unnumbered", "unlisted"], []) [Str "Signatures"], CodeBlock ("", ["python"], []) "# Inline notation
\"question -> answer\"
\"context, question -> answer\"
\"question -> answer: float\"

# Class-based
class Summarize(dspy.Signature):
    \"\"\"Summarize the document in one sentence.\"\"\"
    document: str = dspy.InputField(desc=\"The document to summarize\")
    summary: str = dspy.OutputField(desc=\"A one-sentence summary\")
", Header 3 ("modules-1", ["unnumbered", "unlisted"], []) [Str "Modules"], CodeBlock ("", ["python"], []) "# Basic prediction
predict = dspy.Predict(Summarize)
result = predict(document=\"...\")

# Chain of thought
cot = dspy.ChainOfThought(\"question -> answer\")
result = cot(question=\"What is the capital of France?\")

# Custom module
class RAGModule(dspy.Module):
    def __init__(self, num_passages=3):
        self.retrieve = dspy.Retrieve(k=num_passages)
        self.generate = dspy.ChainOfThought(\"context, question -> answer\")

    def forward(self, question):
        context = self.retrieve(question).passages
        return self.generate(context=context, question=question)
", Header 3 ("evaluation", ["unnumbered", "unlisted"], []) [Str "Evaluation"], CodeBlock ("", ["python"], []) "def answer_exact_match(example, prediction, trace=None):
    return example.answer.lower() == prediction.answer.lower()

evaluate = dspy.Evaluate(
    devset=dev_examples,
    metric=answer_exact_match,
    num_threads=4,
    display_progress=True
)
score = evaluate(program)
", Header 3 ("optimization", ["unnumbered", "unlisted"], []) [Str "Optimization"], CodeBlock ("", ["python"], []) "# Bootstrap few-shot examples
optimizer = dspy.BootstrapRS(
    metric=answer_exact_match,
    max_bootstrapped_demos=4,
    num_candidate_programs=10
)
compiled_program = optimizer.compile(program, trainset=train_examples)

# MIPROv2 for instruction optimization
optimizer = dspy.MIPROv2(
    metric=answer_exact_match,
    auto=\"medium\"
)
compiled_program = optimizer.compile(program, trainset=train_examples)
", Header 3 ("saving-and-loading", ["unnumbered", "unlisted"], []) [Str "Saving and Loading"], CodeBlock ("", ["python"], []) "compiled_program.save(\"optimized_program.json\")

program = RAGModule()
program.load(\"optimized_program.json\")
", Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("global-settings", ["unnumbered", "unlisted"], []) [Str "Global Settings"], CodeBlock ("", ["python"], []) "dspy.configure(
    lm=dspy.LM(\"openai/gpt-4o-mini\"),
    rm=dspy.ColBERTv2(url=\"http://localhost:8893\"),  # retrieval model
    trace=[],  # enable tracing
)
", Header 3 ("per-call-overrides", ["unnumbered", "unlisted"], []) [Str "Per-Call Overrides"], CodeBlock ("", ["python"], []) "with dspy.context(lm=dspy.LM(\"anthropic/claude-sonnet-4-20250514\")):
    result = program(question=\"...\")
", Header 3 ("assertions-and-constraints", ["unnumbered", "unlisted"], []) [Str "Assertions and Constraints"], CodeBlock ("", ["python"], []) "class FactCheckedAnswer(dspy.Module):
    def __init__(self):
        self.generate = dspy.ChainOfThought(\"question -> answer\")

    def forward(self, question):
        result = self.generate(question=question)
        dspy.Assert(
            len(result.answer) > 10,
            \"Answer must be substantive (more than 10 characters)\"
        )
        return result
", Para [Code ("", [], []) "dspy.Assert", Str " raises a hard failure when the constraint is not met, while ", Code ("", [], []) "dspy.Suggest", Str " provides a soft signal that the optimizer can use during compilation to improve outputs."], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("with-retrieval-systems", ["unnumbered", "unlisted"], []) [Str "With Retrieval Systems"], CodeBlock ("", ["python"], []) "import dspy

colbert = dspy.ColBERTv2(url=\"http://localhost:8893\")
dspy.configure(rm=colbert)

class SearchAndAnswer(dspy.Module):
    def __init__(self):
        self.retrieve = dspy.Retrieve(k=5)
        self.answer = dspy.ChainOfThought(\"context, question -> answer\")

    def forward(self, question):
        passages = self.retrieve(question).passages
        return self.answer(context=passages, question=question)
", Header 3 ("with-custom-tools", ["unnumbered", "unlisted"], []) [Str "With Custom Tools"], CodeBlock ("", ["python"], []) "def search_wikipedia(query: str) -> str:
    \"\"\"Search Wikipedia for information.\"\"\"
    # implementation
    return result

react = dspy.ReAct(
    \"question -> answer\",
    tools=[search_wikipedia]
)
", Header 3 ("pipeline-composition", ["unnumbered", "unlisted"], []) [Str "Pipeline Composition"], CodeBlock ("", ["python"], []) "class MultiStepPipeline(dspy.Module):
    def __init__(self):
        self.extract = dspy.Predict(\"document -> entities: list[str]\")
        self.classify = dspy.ChainOfThought(\"entity, context -> category\")
        self.summarize = dspy.Predict(\"entities, categories -> summary\")

    def forward(self, document):
        entities = self.extract(document=document).entities
        categories = [
            self.classify(entity=e, context=document).category
            for e in entities
        ]
        return self.summarize(entities=entities, categories=categories)
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("optimized-classification", ["unnumbered", "unlisted"], []) [Str "Optimized Classification"], CodeBlock ("", ["python"], []) "import dspy

class ClassifyIntent(dspy.Signature):
    \"\"\"Classify the user message into an intent category.\"\"\"
    message: str = dspy.InputField()
    intent: str = dspy.OutputField(
        desc=\"One of: greeting, question, complaint, feedback\"
    )

classifier = dspy.Predict(ClassifyIntent)

def intent_match(example, prediction, trace=None):
    return example.intent == prediction.intent

optimizer = dspy.BootstrapRS(metric=intent_match, max_bootstrapped_demos=4)
optimized = optimizer.compile(classifier, trainset=train_data)

result = optimized(message=\"I'm having trouble with my order\")
print(result.intent)
", Header 3 ("multi-hop-rag-with-optimization", ["unnumbered", "unlisted"], []) [Str "Multi-Hop RAG with Optimization"], CodeBlock ("", ["python"], []) "import dspy

class MultiHopRAG(dspy.Module):
    def __init__(self, passages_per_hop=3, num_hops=2):
        self.retrieve = [
            dspy.Retrieve(k=passages_per_hop)
            for _ in range(num_hops)
        ]
        self.generate_query = dspy.ChainOfThought(
            \"context, question -> search_query\"
        )
        self.generate_answer = dspy.ChainOfThought(
            \"context, question -> answer\"
        )

    def forward(self, question):
        context = []
        for hop in range(len(self.retrieve)):
            if hop == 0:
                passages = self.retrieve[hop](question).passages
            else:
                query = self.generate_query(
                    context=context, question=question
                ).search_query
                passages = self.retrieve[hop](query).passages
            context = deduplicate(context + passages)
        return self.generate_answer(context=context, question=question)

optimizer = dspy.MIPROv2(metric=answer_f1, auto=\"medium\")
compiled_rag = optimizer.compile(MultiHopRAG(), trainset=train_examples)
", Header 3 ("structured-assessment", ["unnumbered", "unlisted"], []) [Str "Structured Assessment"], CodeBlock ("", ["python"], []) "import dspy

class TweetAssessment(dspy.Signature):
    \"\"\"Assess tweet quality on a numeric scale.\"\"\"
    tweet: str = dspy.InputField()
    dimension: str = dspy.InputField(
        desc=\"The quality dimension to assess\"
    )
    score: float = dspy.OutputField(
        desc=\"Quality score from 0.0 to 1.0\"
    )

assessor = dspy.ChainOfThought(TweetAssessment)
result = assessor(
    tweet=\"DSPy lets you program LMs declaratively!\",
    dimension=\"informativeness\"
)
print(f\"Score: {result.score}\")
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Plain [Strong [Str "Optimization cost"], Str " - Compiling programs requires LM calls over the training set, which incurs API costs and wall-clock time (typically ", Str "~", Str "$2 and ", Str "~", Str "20 minutes for medium-sized programs)"]], [Plain [Strong [Str "Dataset requirement"], Str " - Optimizers need labeled examples to tune against; cold-start scenarios with no evaluation data cannot leverage compilation"]], [Plain [Strong [Str "Debugging complexity"], Str " - Compiled programs with optimized prompts and demonstrations can be harder to inspect and debug than hand-written prompts"]], [Plain [Strong [Str "Provider-specific behavior"], Str " - While DSPy abstracts across providers, underlying model differences can cause optimized programs to transfer poorly between LMs"]], [Plain [Strong [Str "Learning curve"], Str " - The programming model (signatures, modules, optimizers) introduces concepts that differ from conventional prompt engineering workflows"]], [Plain [Strong [Str "Non-determinism"], Str " - LM outputs are inherently stochastic; optimization results may vary across runs depending on the training data sampling"]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Str "DSPy 2.0 introduced the current signature and module system, replacing the earlier template-based approach"]], [Plain [Str "Adapter system added for structured output mapping across providers"]], [Plain [Str "MIPROv2 and BetterTogether optimizers added for more sophisticated prompt and weight tuning"]], [Plain [Str "Support expanded to 40+ LM providers"]], [Plain [Str "Assertion and suggestion system introduced for runtime constraint enforcement"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " ", Link ("", [], []) [Str "DSPy Documentation"] ("https://dspy.ai/", "")]]]]