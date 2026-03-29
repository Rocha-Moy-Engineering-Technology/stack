[Header 1 ("openai-moderation", [], []) [Str "OpenAI Moderation"], BlockQuote [Para [Str "OpenAI API for classifying harmful text and image content"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Guardrails & Safety"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "No"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://developers.openai.com/api/docs/guides/moderation", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "OpenAI Moderation is a free API endpoint that classifies text and image content across 13 harm categories. It is designed to check whether user-generated or model-generated content violates OpenAI's usage policies, and it can be integrated into any application pipeline as a content safety filter. The endpoint returns per-category boolean flags and confidence scores, enabling both binary filtering and threshold-based custom policies. ", Str "[", Str "1", Str "]"], Para [Str "Two models are available: ", Code ("", [], []) "omni-moderation-latest", Str " (recommended, multi-modal with text and image support) and ", Code ("", [], []) "text-moderation-latest", Str " (legacy, text-only with fewer categorization options). The omni model provides expanded category coverage including ", Code ("", [], []) "illicit", Str " and ", Code ("", [], []) "illicit/violent", Str " categories that are unavailable in the legacy text-only model. The moderation endpoint is free to use for all OpenAI API customers. ", Str "[", Str "1", Str "]"], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("content-categories", ["unnumbered", "unlisted"], []) [Str "Content Categories"], Para [Str "The moderation system classifies content across 13 categories organized into five groups:"], Para [Strong [Str "Harassment:"]], BulletList [[Plain [Code ("", [], []) "harassment", Str " -- Language that expresses, incites, or promotes harassing behavior toward any target"]], [Plain [Code ("", [], []) "harassment/threatening", Str " -- Harassment content that includes violence or serious harm threats against a target"]]], Para [Strong [Str "Hate Speech:"]], BulletList [[Plain [Code ("", [], []) "hate", Str " -- Content that expresses, incites, or promotes hate based on race, gender, ethnicity, religion, nationality, sexual orientation, disability status, or caste"]], [Plain [Code ("", [], []) "hate/threatening", Str " -- Hateful content that includes violence or serious harm threats against a targeted group"]]], Para [Strong [Str "Illicit Activity (omni model only, text-only input):"]], BulletList [[Plain [Code ("", [], []) "illicit", Str " -- Content that provides advice or instructions for carrying out illegal or illicit acts"]], [Plain [Code ("", [], []) "illicit/violent", Str " -- Content related to illicit acts that also involve violence or weapons"]]], Para [Strong [Str "Self-Harm:"]], BulletList [[Plain [Code ("", [], []) "self-harm", Str " -- Content that promotes, encourages, or depicts acts of self-harm such as suicide, cutting, or eating disorders"]], [Plain [Code ("", [], []) "self-harm/intent", Str " -- Content where the speaker expresses that they are engaging in or intend to engage in acts of self-harm"]], [Plain [Code ("", [], []) "self-harm/instructions", Str " -- Content that encourages or gives instructions on how to commit self-harm"]]], Para [Strong [Str "Sexual Content:"]], BulletList [[Plain [Code ("", [], []) "sexual", Str " -- Content meant to describe or promote sexual activity, including arousal-oriented descriptions and depictions of sexual services"]], [Plain [Code ("", [], []) "sexual/minors", Str " -- Sexual content involving individuals under 18 years of age (text-only input)"]]], Para [Strong [Str "Violence:"]], BulletList [[Plain [Code ("", [], []) "violence", Str " -- Content that depicts death, violence, or physical injury"]], [Plain [Code ("", [], []) "violence/graphic", Str " -- Content that depicts death, violence, or physical injury in graphic detail ", Str "[", Str "1", Str "]"]]], Header 3 ("flagging-vs-scoring", ["unnumbered", "unlisted"], []) [Str "Flagging vs. Scoring"], Para [Str "Each moderation response provides two complementary signals per category:"], BulletList [[Plain [Strong [Code ("", [], []) "flagged"], Str " -- A top-level boolean indicating whether the content violates any policy category. This is a binary determination made by the model."]], [Plain [Strong [Code ("", [], []) "category_scores"], Str " -- Floating-point confidence scores between 0 and 1 for each category. Higher scores indicate greater confidence that the input violates the given category. These scores are not probabilities and should not be interpreted as such. ", Str "[", Str "1", Str "]"]]], Para [Str "Applications can use the boolean ", Code ("", [], []) "flagged", Str " field for simple pass/fail filtering, or use ", Code ("", [], []) "category_scores", Str " with custom thresholds for more nuanced content policies tailored to specific use cases."], Header 3 ("multi-modal-input", ["unnumbered", "unlisted"], []) [Str "Multi-Modal Input"], Para [Str "The ", Code ("", [], []) "omni-moderation-latest", Str " model accepts both text and image inputs. Images can be provided via HTTP URL or base64-encoded data URI. When processing image-only inputs, text-only categories (", Code ("", [], []) "illicit", Str ", ", Code ("", [], []) "illicit/violent", Str ", ", Code ("", [], []) "sexual/minors", Str ") return zero scores because these categories only apply to textual content. The ", Code ("", [], []) "category_applied_input_types", Str " field in the response indicates which input types were evaluated for each category. ", Str "[", Str "1", Str "]"], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Header 3 ("api-endpoint", ["unnumbered", "unlisted"], []) [Str "API Endpoint"], Para [Str "The moderation system is exposed as a single REST endpoint:"], BulletList [[Plain [Strong [Str "Endpoint"], Str ": ", Code ("", [], []) "POST https://api.openai.com/v1/moderations"]], [Plain [Strong [Str "Authentication"], Str ": Bearer token via ", Code ("", [], []) "Authorization", Str " header"]], [Plain [Strong [Str "Content-Type"], Str ": ", Code ("", [], []) "application/json"]], [Plain [Strong [Str "Cost"], Str ": Free for all OpenAI API users ", Str "[", Str "1", Str "]"]]], Header 3 ("processing-pipeline", ["unnumbered", "unlisted"], []) [Str "Processing Pipeline"], Para [Str "The moderation pipeline operates as follows:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Input ingestion"], Str " -- Accept text string, image URL, base64-encoded image, or an array combining multiple input types"]], [Plain [Strong [Str "Model inference"], Str " -- Run the selected moderation model (", Code ("", [], []) "omni-moderation-latest", Str " or ", Code ("", [], []) "text-moderation-latest", Str ") against all input content"]], [Plain [Strong [Str "Category classification"], Str " -- Evaluate each of the 13 content categories independently, producing boolean flags and confidence scores"]], [Plain [Strong [Str "Input type tracking"], Str " -- Record which input types (text, image) were applied to each category evaluation"]], [Plain [Strong [Str "Response assembly"], Str " -- Return a results array with ", Code ("", [], []) "flagged", Str ", ", Code ("", [], []) "categories", Str ", ", Code ("", [], []) "category_scores", Str ", and ", Code ("", [], []) "category_applied_input_types", Str " for each input item ", Str "[", Str "1", Str "]"]]], Header 3 ("model-comparison", ["unnumbered", "unlisted"], []) [Str "Model Comparison"], BulletList [[Plain [Strong [Code ("", [], []) "omni-moderation-latest"], Str " -- Recommended model. Supports text and image inputs. Covers all 13 categories. Snapshot aliases (e.g., ", Code ("", [], []) "omni-moderation-2024-09-26", Str ") are available for pinning a specific model version."]], [Plain [Strong [Code ("", [], []) "text-moderation-latest"], Str " -- Legacy model. Text-only input. Does not support ", Code ("", [], []) "illicit", Str " or ", Code ("", [], []) "illicit/violent", Str " categories. Snapshot alias: ", Code ("", [], []) "text-moderation-007", Str ". ", Str "[", Str "1", Str "]"]]], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Header 3 ("text-moderation", ["unnumbered", "unlisted"], []) [Str "Text Moderation"], Para [Str "The primary use case is classifying text content. A single string input is evaluated against all applicable categories:"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI()

moderation = client.moderations.create(
    model=\"omni-moderation-latest\",
    input=\"This is a sample text to moderate.\"
)

result = moderation.results[0]
if result.flagged:
    print(\"Content flagged for policy violation\")
    for category, flagged in vars(result.categories).items():
        if flagged:
            print(f\"  Violated: {category}\")
", Para [Str "[", Str "1", Str "]"], Header 3 ("image-moderation", ["unnumbered", "unlisted"], []) [Str "Image Moderation"], Para [Str "The omni model classifies images provided via URL or base64-encoded data URI:"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI()

moderation = client.moderations.create(
    model=\"omni-moderation-latest\",
    input=[
        {
            \"type\": \"image_url\",
            \"image_url\": {
                \"url\": \"https://example.com/image.jpg\"
            }
        }
    ]
)
", Para [Str "[", Str "1", Str "]"], Header 3 ("multi-modal-moderation", ["unnumbered", "unlisted"], []) [Str "Multi-Modal Moderation"], Para [Str "Text and images can be combined in a single request for simultaneous classification:"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI()

moderation = client.moderations.create(
    model=\"omni-moderation-latest\",
    input=[
        {\"type\": \"text\", \"text\": \"Here is an image of violence\"},
        {
            \"type\": \"image_url\",
            \"image_url\": {
                \"url\": \"https://example.com/image.jpg\"
            }
        }
    ]
)
", Para [Str "When combining text and image inputs, the model evaluates both modalities together. Text-only categories (", Code ("", [], []) "illicit", Str ", ", Code ("", [], []) "illicit/violent", Str ", ", Code ("", [], []) "sexual/minors", Str ") still apply only to the text portion and receive zero scores for image-only content. ", Str "[", Str "1", Str "]"], Header 3 ("base64-image-input", ["unnumbered", "unlisted"], []) [Str "Base64 Image Input"], Para [Str "Images can be provided as base64-encoded data URIs, avoiding the need for publicly accessible URLs:"], CodeBlock ("", ["python"], []) "import base64
from openai import OpenAI

client = OpenAI()

with open(\"image.png\", \"rb\") as f:
    b64_image = base64.b64encode(f.read()).decode(\"utf-8\")

moderation = client.moderations.create(
    model=\"omni-moderation-latest\",
    input=[
        {
            \"type\": \"image_url\",
            \"image_url\": {
                \"url\": f\"data:image/png;base64,{b64_image}\"
            }
        }
    ]
)
", Para [Str "[", Str "1", Str "]"], Header 3 ("category-score-thresholds", ["unnumbered", "unlisted"], []) [Str "Category Score Thresholds"], Para [Str "The ", Code ("", [], []) "category_scores", Str " field enables custom content policies with adjustable sensitivity per category:"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI()

THRESHOLDS = {
    \"harassment\": 0.7,
    \"violence\": 0.8,
    \"sexual\": 0.6,
    \"self-harm\": 0.5,
}

moderation = client.moderations.create(
    model=\"omni-moderation-latest\",
    input=\"Content to evaluate\"
)

scores = moderation.results[0].category_scores
for category, threshold in THRESHOLDS.items():
    score = getattr(scores, category.replace(\"-\", \"_\"))
    if score > threshold:
        print(f\"Category '{category}' exceeds threshold: {score:.4f} > {threshold}\")
", Para [Str "[", Str "1", Str "]"], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], Header 3 ("user-generated-content-filtering", ["unnumbered", "unlisted"], []) [Str "User-Generated Content Filtering"], Para [Str "Pre-screen user submissions (comments, posts, messages) before publishing. Use the boolean ", Code ("", [], []) "flagged", Str " field for automatic blocking or route flagged content to human review queues."], Header 3 ("llm-output-safety", ["unnumbered", "unlisted"], []) [Str "LLM Output Safety"], Para [Str "Run model-generated responses through the moderation endpoint before delivering them to end users. This provides a secondary safety layer beyond the model's built-in content policies."], Header 3 ("chat-application-moderation", ["unnumbered", "unlisted"], []) [Str "Chat Application Moderation"], Para [Str "Evaluate both incoming user messages and outgoing assistant responses in real time. Flag conversations that contain policy-violating content for review or automatic termination."], Header 3 ("content-platform-compliance", ["unnumbered", "unlisted"], []) [Str "Content Platform Compliance"], Para [Str "Implement category-specific policies with custom score thresholds. For example, a children's platform might use lower thresholds for ", Code ("", [], []) "sexual", Str " and ", Code ("", [], []) "violence", Str " categories while a news platform might tolerate higher ", Code ("", [], []) "violence", Str " scores for reporting contexts."], Header 3 ("image-upload-screening", ["unnumbered", "unlisted"], []) [Str "Image Upload Screening"], Para [Str "Screen user-uploaded images for violent, sexual, or self-harm content before they are stored or displayed. Combine with text moderation for captioned or annotated images."], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("request", ["unnumbered", "unlisted"], []) [Str "Request"], BulletList [[Plain [Strong [Str "Method"], Str ": ", Code ("", [], []) "POST"]], [Plain [Strong [Str "URL"], Str ": ", Code ("", [], []) "https://api.openai.com/v1/moderations"]], [Plain [Strong [Str "Headers"], Str ": ", Code ("", [], []) "Authorization: Bearer <API_KEY>", Str ", ", Code ("", [], []) "Content-Type: application/json"]]], Header 3 ("request-body", ["unnumbered", "unlisted"], []) [Str "Request Body"], BulletList [[Plain [Strong [Code ("", [], []) "model"], Str " -- String. Model to use. Values: ", Code ("", [], []) "omni-moderation-latest", Str ", ", Code ("", [], []) "omni-moderation-2024-09-26", Str ", ", Code ("", [], []) "text-moderation-latest", Str ", ", Code ("", [], []) "text-moderation-007", Str "."]], [Plain [Strong [Code ("", [], []) "input"], Str " -- String or array. For text-only: a plain string. For multi-modal: an array of objects with ", Code ("", [], []) "type", Str " field (", Code ("", [], []) "text", Str " or ", Code ("", [], []) "image_url", Str ")."]]], Header 3 ("response-object", ["unnumbered", "unlisted"], []) [Str "Response Object"], BulletList [[Plain [Strong [Code ("", [], []) "id"], Str " -- String. Unique identifier for the moderation request."]], [Plain [Strong [Code ("", [], []) "model"], Str " -- String. Model used for classification."]], [Plain [Strong [Code ("", [], []) "results"], Str " -- Array of result objects, one per input item."]]], Header 3 ("result-object", ["unnumbered", "unlisted"], []) [Str "Result Object"], BulletList [[Plain [Strong [Code ("", [], []) "flagged"], Str " -- Boolean. ", Code ("", [], []) "true", Str " if the model classifies the content as potentially harmful."]], [Plain [Strong [Code ("", [], []) "categories"], Str " -- Object. Boolean per category indicating whether the category was violated."]], [Plain [Strong [Code ("", [], []) "category_scores"], Str " -- Object. Float per category (0 to 1) indicating confidence of violation."]], [Plain [Strong [Code ("", [], []) "category_applied_input_types"], Str " -- Object. Array of input types (", Code ("", [], []) "text", Str ", ", Code ("", [], []) "image", Str ") evaluated for each category. ", Str "[", Str "1", Str "]"]]], Header 3 ("example-response", ["unnumbered", "unlisted"], []) [Str "Example Response"], CodeBlock ("", ["json"], []) "{
  \"id\": \"modr-abc123\",
  \"model\": \"omni-moderation-latest\",
  \"results\": [
    {
      \"flagged\": true,
      \"categories\": {
        \"harassment\": false,
        \"harassment/threatening\": false,
        \"hate\": false,
        \"hate/threatening\": false,
        \"illicit\": false,
        \"illicit/violent\": false,
        \"self-harm\": false,
        \"self-harm/intent\": false,
        \"self-harm/instructions\": false,
        \"sexual\": false,
        \"sexual/minors\": false,
        \"violence\": true,
        \"violence/graphic\": false
      },
      \"category_scores\": {
        \"harassment\": 0.0001,
        \"harassment/threatening\": 0.0002,
        \"hate\": 0.0001,
        \"hate/threatening\": 0.0001,
        \"illicit\": 0.0000,
        \"illicit/violent\": 0.0000,
        \"self-harm\": 0.0003,
        \"self-harm/intent\": 0.0001,
        \"self-harm/instructions\": 0.0000,
        \"sexual\": 0.0001,
        \"sexual/minors\": 0.0000,
        \"violence\": 0.9532,
        \"violence/graphic\": 0.0214
      },
      \"category_applied_input_types\": {
        \"harassment\": [\"text\"],
        \"harassment/threatening\": [\"text\"],
        \"hate\": [\"text\"],
        \"hate/threatening\": [\"text\"],
        \"illicit\": [\"text\"],
        \"illicit/violent\": [\"text\"],
        \"self-harm\": [\"text\", \"image\"],
        \"self-harm/intent\": [\"text\", \"image\"],
        \"self-harm/instructions\": [\"text\", \"image\"],
        \"sexual\": [\"text\", \"image\"],
        \"sexual/minors\": [\"text\"],
        \"violence\": [\"text\", \"image\"],
        \"violence/graphic\": [\"text\", \"image\"]
      }
    }
  ]
}
", Para [Str "[", Str "1", Str "]"], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("model-selection", ["unnumbered", "unlisted"], []) [Str "Model Selection"], BulletList [[Plain [Strong [Code ("", [], []) "omni-moderation-latest"], Str " -- Use for all new integrations. Supports text and images, all 13 categories. Points to the latest omni snapshot."]], [Plain [Strong [Code ("", [], []) "omni-moderation-2024-09-26"], Str " -- Pin to a specific snapshot for deterministic behavior across model updates."]], [Plain [Strong [Code ("", [], []) "text-moderation-latest"], Str " -- Legacy text-only model. Fewer categories (no ", Code ("", [], []) "illicit", Str "/", Code ("", [], []) "illicit/violent", Str "). Points to latest text snapshot."]], [Plain [Strong [Code ("", [], []) "text-moderation-007"], Str " -- Pin to a specific legacy text snapshot. ", Str "[", Str "1", Str "]"]]], Header 3 ("custom-thresholds", ["unnumbered", "unlisted"], []) [Str "Custom Thresholds"], Para [Str "The default ", Code ("", [], []) "flagged", Str " boolean uses OpenAI's internal thresholds. For custom content policies, use ", Code ("", [], []) "category_scores", Str " with application-specific thresholds. Different categories can have different thresholds depending on the sensitivity requirements of the platform. OpenAI recommends starting with the default ", Code ("", [], []) "flagged", Str " values and adjusting thresholds based on observed false positive and false negative rates. ", Str "[", Str "1", Str "]"], Header 3 ("score-recalibration", ["unnumbered", "unlisted"], []) [Str "Score Recalibration"], Para [Str "Category scores may shift when the underlying model is updated (e.g., when ", Code ("", [], []) "omni-moderation-latest", Str " points to a new snapshot). Applications that rely on specific score thresholds should:"], BulletList [[Plain [Str "Pin to a dated snapshot model (e.g., ", Code ("", [], []) "omni-moderation-2024-09-26", Str ") for stable scores"]], [Plain [Str "Re-evaluate thresholds when migrating to a new model snapshot"]], [Plain [Str "Monitor score distributions over time to detect drift ", Str "[", Str "1", Str "]"]]], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("pre-processing-filter-user-input", ["unnumbered", "unlisted"], []) [Str "Pre-Processing Filter (User Input)"], Para [Str "Screen user input before it reaches the language model:"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI()

def moderate_input(user_message: str) -> bool:
    moderation = client.moderations.create(
        model=\"omni-moderation-latest\",
        input=user_message
    )
    return moderation.results[0].flagged

user_input = \"User's message here\"
if moderate_input(user_input):
    print(\"Input rejected: content policy violation\")
else:
    # Proceed with LLM call
    response = client.responses.create(
        model=\"gpt-4o\",
        input=user_input
    )
", Header 3 ("post-processing-filter-model-output", ["unnumbered", "unlisted"], []) [Str "Post-Processing Filter (Model Output)"], Para [Str "Screen model-generated responses before delivering to the user:"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI()

response = client.responses.create(
    model=\"gpt-4o\",
    input=\"User's question\"
)

moderation = client.moderations.create(
    model=\"omni-moderation-latest\",
    input=response.output_text
)

if moderation.results[0].flagged:
    print(\"Response blocked: content policy violation\")
else:
    print(response.output_text)
", Header 3 ("with-agent-frameworks-langchain-langgraph", ["unnumbered", "unlisted"], []) [Str "With Agent Frameworks (LangChain, LangGraph)"], Para [Str "Integrate as a guardrail step in agent pipelines. Run moderation on tool inputs and outputs, user messages, and final agent responses."], Header 3 ("with-api-gateways-litellm-portkey", ["unnumbered", "unlisted"], []) [Str "With API Gateways (LiteLLM, Portkey)"], Para [Str "Deploy moderation as a middleware layer in API gateway configurations. Screen all inbound requests and outbound responses through the moderation endpoint before forwarding."], Header 3 ("with-guardrails-frameworks-guardrails-ai-nemo-guardrails", ["unnumbered", "unlisted"], []) [Str "With Guardrails Frameworks (Guardrails AI, NeMo Guardrails)"], Para [Str "Use OpenAI Moderation as one validator among many in a multi-layer guardrails pipeline. Combine with custom validators for domain-specific content policies."], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("batch-moderation-of-multiple-texts", ["unnumbered", "unlisted"], []) [Str "Batch Moderation of Multiple Texts"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI()

texts = [
    \"This is a friendly greeting.\",
    \"I want to hurt someone.\",
    \"The weather is nice today.\",
]

for text in texts:
    moderation = client.moderations.create(
        model=\"omni-moderation-latest\",
        input=text
    )
    result = moderation.results[0]
    status = \"FLAGGED\" if result.flagged else \"OK\"
    print(f\"[{status}] {text}\")
", Para [Str "[", Str "1", Str "]"], Header 3 ("multi-modal-content-review", ["unnumbered", "unlisted"], []) [Str "Multi-Modal Content Review"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI()

moderation = client.moderations.create(
    model=\"omni-moderation-latest\",
    input=[
        {\"type\": \"text\", \"text\": \"Check this image for harmful content\"},
        {
            \"type\": \"image_url\",
            \"image_url\": {
                \"url\": \"https://example.com/uploaded-image.jpg\"
            }
        }
    ]
)

result = moderation.results[0]
print(f\"Flagged: {result.flagged}\")

# Inspect which input types triggered each category
for category, input_types in vars(result.category_applied_input_types).items():
    score = getattr(result.category_scores, category.replace(\"/\", \"_\"))
    if score > 0.01:
        print(f\"  {category}: score={score:.4f}, inputs={input_types}\")
", Para [Str "[", Str "1", Str "]"], Header 3 ("javascript-express-middleware", ["unnumbered", "unlisted"], []) [Str "JavaScript Express Middleware"], CodeBlock ("", ["javascript"], []) "import OpenAI from \"openai\";

const openai = new OpenAI();

async function moderationMiddleware(req, res, next) {
    const { message } = req.body;

    const moderation = await openai.moderations.create({
        model: \"omni-moderation-latest\",
        input: message
    });

    if (moderation.results[0].flagged) {
        return res.status(400).json({
            error: \"Content policy violation\",
            categories: moderation.results[0].categories
        });
    }

    next();
}
", Para [Str "[", Str "1", Str "]"], Header 3 ("curl-with-multi-modal-input", ["unnumbered", "unlisted"], []) [Str "cURL with Multi-Modal Input"], CodeBlock ("", ["bash"], []) "curl https://api.openai.com/v1/moderations \\
  -X POST \\
  -H \"Content-Type: application/json\" \\
  -H \"Authorization: Bearer $OPENAI_API_KEY\" \\
  -d '{
    \"model\": \"omni-moderation-latest\",
    \"input\": [
      {\"type\": \"text\", \"text\": \"Describe this image\"},
      {
        \"type\": \"image_url\",
        \"image_url\": {
          \"url\": \"https://example.com/image.jpg\"
        }
      }
    ]
  }'
", Para [Str "[", Str "1", Str "]"], Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Plain [Strong [Str "Not a complete safety solution"], Str ": The moderation endpoint is one layer of defense; it should be combined with other safety measures (prompt engineering, human review, application-level rules)"]], [Plain [Strong [Str "Text-only categories"], Str ": ", Code ("", [], []) "illicit", Str ", ", Code ("", [], []) "illicit/violent", Str ", and ", Code ("", [], []) "sexual/minors", Str " only evaluate text input and return zero scores for image-only content ", Str "[", Str "1", Str "]"]], [Plain [Strong [Str "Score instability across updates"], Str ": When ", Code ("", [], []) "omni-moderation-latest", Str " points to a new model snapshot, ", Code ("", [], []) "category_scores", Str " values may shift, requiring recalibration of custom thresholds ", Str "[", Str "1", Str "]"]], [Plain [Strong [Str "No fine-tuning or customization"], Str ": The moderation categories and models are fixed; there is no way to add custom categories or train on domain-specific content policies"]], [Plain [Strong [Str "Latency"], Str ": Each moderation call adds latency to the request pipeline; for real-time applications, consider asynchronous or batched moderation strategies"]], [Plain [Strong [Str "English-centric"], Str ": While the models handle multilingual input, classification accuracy is highest for English content"]], [Plain [Strong [Str "No streaming support"], Str ": The moderation endpoint processes complete inputs and returns complete results; there is no streaming or incremental classification"]], [Plain [Strong [Str "Closed source"], Str ": The models and training data are proprietary; classification decisions cannot be inspected or explained beyond the provided scores"]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Strong [Code ("", [], []) "omni-moderation-latest"], Str ": Current recommended model with multi-modal support (text and images) and all 13 categories"]], [Plain [Strong [Code ("", [], []) "omni-moderation-2024-09-26"], Str ": First dated snapshot of the omni moderation model"]], [Plain [Strong [Code ("", [], []) "illicit", Str " categories"], Str ": Added with the omni model, covering illegal activity and violent illegal acts"]], [Plain [Strong [Str "Multi-modal input"], Str ": Image URL and base64 data URI support introduced with the omni model"]], [Plain [Strong [Code ("", [], []) "category_applied_input_types"], Str ": Response field added to indicate which input modalities were evaluated per category"]], [Plain [Strong [Code ("", [], []) "text-moderation-latest", Str " / ", Code ("", [], []) "text-moderation-007"], Str ": Legacy text-only models, still available but not recommended for new projects ", Str "[", Str "1", Str "]"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " Moderation Guide - ", Link ("", [], []) [Str "https://developers.openai.com/api/docs/guides/moderation"] ("https://developers.openai.com/api/docs/guides/moderation", "")]]]]