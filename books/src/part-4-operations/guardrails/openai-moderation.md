# OpenAI Moderation

> OpenAI API for classifying harmful text and image content

| Field | Value |
|-------|-------|
| Group | Guardrails |
| Type | API |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Documentation | [Official Docs](https://developers.openai.com/api/docs/guides/moderation) |

## Overview

OpenAI Moderation is a free API endpoint that classifies text and image content across 13 harm categories. It is designed to check whether user-generated or model-generated content violates OpenAI's usage policies, and it can be integrated into any application pipeline as a content safety filter. The endpoint returns per-category boolean flags and confidence scores, enabling both binary filtering and threshold-based custom policies. [1]

Two models are available: `omni-moderation-latest` (recommended, multi-modal with text and image support) and `text-moderation-latest` (legacy, text-only with fewer categorization options). The omni model provides expanded category coverage including `illicit` and `illicit/violent` categories that are unavailable in the legacy text-only model. The moderation endpoint is free to use for all OpenAI API customers. [1]

## Core Concepts

### Content Categories

The moderation system classifies content across 13 categories organized into five groups:

**Harassment:**
- `harassment` -- Language that expresses, incites, or promotes harassing behavior toward any target
- `harassment/threatening` -- Harassment content that includes violence or serious harm threats against a target

**Hate Speech:**
- `hate` -- Content that expresses, incites, or promotes hate based on race, gender, ethnicity, religion, nationality, sexual orientation, disability status, or caste
- `hate/threatening` -- Hateful content that includes violence or serious harm threats against a targeted group

**Illicit Activity (omni model only, text-only input):**
- `illicit` -- Content that provides advice or instructions for carrying out illegal or illicit acts
- `illicit/violent` -- Content related to illicit acts that also involve violence or weapons

**Self-Harm:**
- `self-harm` -- Content that promotes, encourages, or depicts acts of self-harm such as suicide, cutting, or eating disorders
- `self-harm/intent` -- Content where the speaker expresses that they are engaging in or intend to engage in acts of self-harm
- `self-harm/instructions` -- Content that encourages or gives instructions on how to commit self-harm

**Sexual Content:**
- `sexual` -- Content meant to describe or promote sexual activity, including arousal-oriented descriptions and depictions of sexual services
- `sexual/minors` -- Sexual content involving individuals under 18 years of age (text-only input)

**Violence:**
- `violence` -- Content that depicts death, violence, or physical injury
- `violence/graphic` -- Content that depicts death, violence, or physical injury in graphic detail [1]

### Flagging vs. Scoring

Each moderation response provides two complementary signals per category:

- **`flagged`** -- A top-level boolean indicating whether the content violates any policy category. This is a binary determination made by the model.
- **`category_scores`** -- Floating-point confidence scores between 0 and 1 for each category. Higher scores indicate greater confidence that the input violates the given category. These scores are not probabilities and should not be interpreted as such. [1]

Applications can use the boolean `flagged` field for simple pass/fail filtering, or use `category_scores` with custom thresholds for more nuanced content policies tailored to specific use cases.

### Multi-Modal Input

The `omni-moderation-latest` model accepts both text and image inputs. Images can be provided via HTTP URL or base64-encoded data URI. When processing image-only inputs, text-only categories (`illicit`, `illicit/violent`, `sexual/minors`) return zero scores because these categories only apply to textual content. The `category_applied_input_types` field in the response indicates which input types were evaluated for each category. [1]

## Architecture

### API Endpoint

The moderation system is exposed as a single REST endpoint:

- **Endpoint**: `POST https://api.openai.com/v1/moderations`
- **Authentication**: Bearer token via `Authorization` header
- **Content-Type**: `application/json`
- **Cost**: Free for all OpenAI API users [1]

### Processing Pipeline

The moderation pipeline operates as follows:

1. **Input ingestion** -- Accept text string, image URL, base64-encoded image, or an array combining multiple input types
2. **Model inference** -- Run the selected moderation model (`omni-moderation-latest` or `text-moderation-latest`) against all input content
3. **Category classification** -- Evaluate each of the 13 content categories independently, producing boolean flags and confidence scores
4. **Input type tracking** -- Record which input types (text, image) were applied to each category evaluation
5. **Response assembly** -- Return a results array with `flagged`, `categories`, `category_scores`, and `category_applied_input_types` for each input item [1]

### Model Comparison

- **`omni-moderation-latest`** -- Recommended model. Supports text and image inputs. Covers all 13 categories. Snapshot aliases (e.g., `omni-moderation-2024-09-26`) are available for pinning a specific model version.
- **`text-moderation-latest`** -- Legacy model. Text-only input. Does not support `illicit` or `illicit/violent` categories. Snapshot alias: `text-moderation-007`. [1]

## Key Features and Functionality

### Text Moderation

The primary use case is classifying text content. A single string input is evaluated against all applicable categories:

```python
from openai import OpenAI

client = OpenAI()

moderation = client.moderations.create(
    model="omni-moderation-latest",
    input="This is a sample text to moderate."
)

result = moderation.results[0]
if result.flagged:
    print("Content flagged for policy violation")
    for category, flagged in vars(result.categories).items():
        if flagged:
            print(f"  Violated: {category}")
```
[1]

### Image Moderation

The omni model classifies images provided via URL or base64-encoded data URI:

```python
from openai import OpenAI

client = OpenAI()

moderation = client.moderations.create(
    model="omni-moderation-latest",
    input=[
        {
            "type": "image_url",
            "image_url": {
                "url": "https://example.com/image.jpg"
            }
        }
    ]
)
```
[1]

### Multi-Modal Moderation

Text and images can be combined in a single request for simultaneous classification:

```python
from openai import OpenAI

client = OpenAI()

moderation = client.moderations.create(
    model="omni-moderation-latest",
    input=[
        {"type": "text", "text": "Here is an image of violence"},
        {
            "type": "image_url",
            "image_url": {
                "url": "https://example.com/image.jpg"
            }
        }
    ]
)
```

When combining text and image inputs, the model evaluates both modalities together. Text-only categories (`illicit`, `illicit/violent`, `sexual/minors`) still apply only to the text portion and receive zero scores for image-only content. [1]

### Base64 Image Input

Images can be provided as base64-encoded data URIs, avoiding the need for publicly accessible URLs:

```python
import base64
from openai import OpenAI

client = OpenAI()

with open("image.png", "rb") as f:
    b64_image = base64.b64encode(f.read()).decode("utf-8")

moderation = client.moderations.create(
    model="omni-moderation-latest",
    input=[
        {
            "type": "image_url",
            "image_url": {
                "url": f"data:image/png;base64,{b64_image}"
            }
        }
    ]
)
```
[1]

### Category Score Thresholds

The `category_scores` field enables custom content policies with adjustable sensitivity per category:

```python
from openai import OpenAI

client = OpenAI()

THRESHOLDS = {
    "harassment": 0.7,
    "violence": 0.8,
    "sexual": 0.6,
    "self-harm": 0.5,
}

moderation = client.moderations.create(
    model="omni-moderation-latest",
    input="Content to evaluate"
)

scores = moderation.results[0].category_scores
for category, threshold in THRESHOLDS.items():
    score = getattr(scores, category.replace("-", "_"))
    if score > threshold:
        print(f"Category '{category}' exceeds threshold: {score:.4f} > {threshold}")
```
[1]

## Use Cases

### User-Generated Content Filtering

Pre-screen user submissions (comments, posts, messages) before publishing. Use the boolean `flagged` field for automatic blocking or route flagged content to human review queues.

### LLM Output Safety

Run model-generated responses through the moderation endpoint before delivering them to end users. This provides a secondary safety layer beyond the model's built-in content policies.

### Chat Application Moderation

Evaluate both incoming user messages and outgoing assistant responses in real time. Flag conversations that contain policy-violating content for review or automatic termination.

### Content Platform Compliance

Implement category-specific policies with custom score thresholds. For example, a children's platform might use lower thresholds for `sexual` and `violence` categories while a news platform might tolerate higher `violence` scores for reporting contexts.

### Image Upload Screening

Screen user-uploaded images for violent, sexual, or self-harm content before they are stored or displayed. Combine with text moderation for captioned or annotated images.

## API Reference Summary

### Request

- **Method**: `POST`
- **URL**: `https://api.openai.com/v1/moderations`
- **Headers**: `Authorization: Bearer <API_KEY>`, `Content-Type: application/json`

### Request Body

- **`model`** -- String. Model to use. Values: `omni-moderation-latest`, `omni-moderation-2024-09-26`, `text-moderation-latest`, `text-moderation-007`.
- **`input`** -- String or array. For text-only: a plain string. For multi-modal: an array of objects with `type` field (`text` or `image_url`).

### Response Object

- **`id`** -- String. Unique identifier for the moderation request.
- **`model`** -- String. Model used for classification.
- **`results`** -- Array of result objects, one per input item.

### Result Object

- **`flagged`** -- Boolean. `true` if the model classifies the content as potentially harmful.
- **`categories`** -- Object. Boolean per category indicating whether the category was violated.
- **`category_scores`** -- Object. Float per category (0 to 1) indicating confidence of violation.
- **`category_applied_input_types`** -- Object. Array of input types (`text`, `image`) evaluated for each category. [1]

### Example Response

```json
{
  "id": "modr-abc123",
  "model": "omni-moderation-latest",
  "results": [
    {
      "flagged": true,
      "categories": {
        "harassment": false,
        "harassment/threatening": false,
        "hate": false,
        "hate/threatening": false,
        "illicit": false,
        "illicit/violent": false,
        "self-harm": false,
        "self-harm/intent": false,
        "self-harm/instructions": false,
        "sexual": false,
        "sexual/minors": false,
        "violence": true,
        "violence/graphic": false
      },
      "category_scores": {
        "harassment": 0.0001,
        "harassment/threatening": 0.0002,
        "hate": 0.0001,
        "hate/threatening": 0.0001,
        "illicit": 0.0000,
        "illicit/violent": 0.0000,
        "self-harm": 0.0003,
        "self-harm/intent": 0.0001,
        "self-harm/instructions": 0.0000,
        "sexual": 0.0001,
        "sexual/minors": 0.0000,
        "violence": 0.9532,
        "violence/graphic": 0.0214
      },
      "category_applied_input_types": {
        "harassment": ["text"],
        "harassment/threatening": ["text"],
        "hate": ["text"],
        "hate/threatening": ["text"],
        "illicit": ["text"],
        "illicit/violent": ["text"],
        "self-harm": ["text", "image"],
        "self-harm/intent": ["text", "image"],
        "self-harm/instructions": ["text", "image"],
        "sexual": ["text", "image"],
        "sexual/minors": ["text"],
        "violence": ["text", "image"],
        "violence/graphic": ["text", "image"]
      }
    }
  ]
}
```
[1]

## Configuration and Customization

### Model Selection

- **`omni-moderation-latest`** -- Use for all new integrations. Supports text and images, all 13 categories. Points to the latest omni snapshot.
- **`omni-moderation-2024-09-26`** -- Pin to a specific snapshot for deterministic behavior across model updates.
- **`text-moderation-latest`** -- Legacy text-only model. Fewer categories (no `illicit`/`illicit/violent`). Points to latest text snapshot.
- **`text-moderation-007`** -- Pin to a specific legacy text snapshot. [1]

### Custom Thresholds

The default `flagged` boolean uses OpenAI's internal thresholds. For custom content policies, use `category_scores` with application-specific thresholds. Different categories can have different thresholds depending on the sensitivity requirements of the platform. OpenAI recommends starting with the default `flagged` values and adjusting thresholds based on observed false positive and false negative rates. [1]

### Score Recalibration

Category scores may shift when the underlying model is updated (e.g., when `omni-moderation-latest` points to a new snapshot). Applications that rely on specific score thresholds should:

- Pin to a dated snapshot model (e.g., `omni-moderation-2024-09-26`) for stable scores
- Re-evaluate thresholds when migrating to a new model snapshot
- Monitor score distributions over time to detect drift [1]

## Integration Patterns

### Pre-Processing Filter (User Input)

Screen user input before it reaches the language model:

```python
from openai import OpenAI

client = OpenAI()

def moderate_input(user_message: str) -> bool:
    moderation = client.moderations.create(
        model="omni-moderation-latest",
        input=user_message
    )
    return moderation.results[0].flagged

user_input = "User's message here"
if moderate_input(user_input):
    print("Input rejected: content policy violation")
else:
    # Proceed with LLM call
    response = client.responses.create(
        model="gpt-4o",
        input=user_input
    )
```

### Post-Processing Filter (Model Output)

Screen model-generated responses before delivering to the user:

```python
from openai import OpenAI

client = OpenAI()

response = client.responses.create(
    model="gpt-4o",
    input="User's question"
)

moderation = client.moderations.create(
    model="omni-moderation-latest",
    input=response.output_text
)

if moderation.results[0].flagged:
    print("Response blocked: content policy violation")
else:
    print(response.output_text)
```

### With Agent Frameworks (LangChain, LangGraph)

Integrate as a guardrail step in agent pipelines. Run moderation on tool inputs and outputs, user messages, and final agent responses.

### With API Gateways (LiteLLM, Portkey)

Deploy moderation as a middleware layer in API gateway configurations. Screen all inbound requests and outbound responses through the moderation endpoint before forwarding.

### With Guardrails Frameworks (Guardrails AI, NeMo Guardrails)

Use OpenAI Moderation as one validator among many in a multi-layer guardrails pipeline. Combine with custom validators for domain-specific content policies.

## Examples

### Batch Moderation of Multiple Texts

```python
from openai import OpenAI

client = OpenAI()

texts = [
    "This is a friendly greeting.",
    "I want to hurt someone.",
    "The weather is nice today.",
]

for text in texts:
    moderation = client.moderations.create(
        model="omni-moderation-latest",
        input=text
    )
    result = moderation.results[0]
    status = "FLAGGED" if result.flagged else "OK"
    print(f"[{status}] {text}")
```
[1]

### Multi-Modal Content Review

```python
from openai import OpenAI

client = OpenAI()

moderation = client.moderations.create(
    model="omni-moderation-latest",
    input=[
        {"type": "text", "text": "Check this image for harmful content"},
        {
            "type": "image_url",
            "image_url": {
                "url": "https://example.com/uploaded-image.jpg"
            }
        }
    ]
)

result = moderation.results[0]
print(f"Flagged: {result.flagged}")

# Inspect which input types triggered each category
for category, input_types in vars(result.category_applied_input_types).items():
    score = getattr(result.category_scores, category.replace("/", "_"))
    if score > 0.01:
        print(f"  {category}: score={score:.4f}, inputs={input_types}")
```
[1]

### JavaScript Express Middleware

```javascript
import OpenAI from "openai";

const openai = new OpenAI();

async function moderationMiddleware(req, res, next) {
    const { message } = req.body;

    const moderation = await openai.moderations.create({
        model: "omni-moderation-latest",
        input: message
    });

    if (moderation.results[0].flagged) {
        return res.status(400).json({
            error: "Content policy violation",
            categories: moderation.results[0].categories
        });
    }

    next();
}
```
[1]

### cURL with Multi-Modal Input

```bash
curl https://api.openai.com/v1/moderations \
  -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "omni-moderation-latest",
    "input": [
      {"type": "text", "text": "Describe this image"},
      {
        "type": "image_url",
        "image_url": {
          "url": "https://example.com/image.jpg"
        }
      }
    ]
  }'
```
[1]

## Limitations and Considerations

- **Not a complete safety solution**: The moderation endpoint is one layer of defense; it should be combined with other safety measures (prompt engineering, human review, application-level rules)
- **Text-only categories**: `illicit`, `illicit/violent`, and `sexual/minors` only evaluate text input and return zero scores for image-only content [1]
- **Score instability across updates**: When `omni-moderation-latest` points to a new model snapshot, `category_scores` values may shift, requiring recalibration of custom thresholds [1]
- **No fine-tuning or customization**: The moderation categories and models are fixed; there is no way to add custom categories or train on domain-specific content policies
- **Latency**: Each moderation call adds latency to the request pipeline; for real-time applications, consider asynchronous or batched moderation strategies
- **English-centric**: While the models handle multilingual input, classification accuracy is highest for English content
- **No streaming support**: The moderation endpoint processes complete inputs and returns complete results; there is no streaming or incremental classification
- **Closed source**: The models and training data are proprietary; classification decisions cannot be inspected or explained beyond the provided scores

## Changelog Highlights

- **`omni-moderation-latest`**: Current recommended model with multi-modal support (text and images) and all 13 categories
- **`omni-moderation-2024-09-26`**: First dated snapshot of the omni moderation model
- **`illicit` categories**: Added with the omni model, covering illegal activity and violent illegal acts
- **Multi-modal input**: Image URL and base64 data URI support introduced with the omni model
- **`category_applied_input_types`**: Response field added to indicate which input modalities were evaluated per category
- **`text-moderation-latest` / `text-moderation-007`**: Legacy text-only models, still available but not recommended for new projects [1]

## Citations

- [1] Moderation Guide - <https://developers.openai.com/api/docs/guides/moderation>

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- OpenAI Moderation
- omni-moderation-latest
- text-moderation-latest
- content moderation API
- harm categories
- harassment
- hate
- self-harm
- sexual
- violence
- illicit
- multimodal moderation
- image moderation
- category scores
- flagged
- free API
- category_applied_input_types
- base64 image
- threshold tuning
- 13 categories
- moderation endpoint
- safety classifier
- content safety filter

### Verb-Noun Tasks

- Classify user-generated text for harmful content
- Moderate model-generated output before delivery
- Screen images via URL or base64 data URI
- Combine text and image in a single moderation request
- Tune custom thresholds per category using category_scores
- Pin a dated moderation snapshot for stable scores
- Batch-moderate a list of messages
- Add a moderation middleware in an Express server
- Route flagged content to a human review queue
- Use moderation as a layer in a multi-guardrail pipeline
- Detect sexual/minors content in user text
- Distinguish text-only versus image-applicable categories

### User Intent Phrases

- How do I check if user content violates OpenAI's policy?
- I need a free safety classifier for chat messages.
- How do I moderate images for violence or sexual content?
- Show me OpenAI's harassment and self-harm classifier.
- Can I set custom thresholds per harm category?
- How do I combine OpenAI Moderation with Guardrails AI or NeMo Guardrails?
- Pre-screen user input before sending to gpt-4o.
- Post-screen the LLM response before showing it to the user.
- What categories does the omni moderation model cover?
- How do I send a base64-encoded image to the moderation endpoint?

### Problem Statements

- User-generated content includes hate, harassment, or self-harm.
- Model outputs occasionally produce violent or sexual content.
- No free, simple safety classifier for chat applications.
- Need image safety screening, not just text classification.
- Pass/fail moderation is too coarse for nuanced platform policies.
- Moderation scores shift when model snapshots update.

### When to Pick This

- Pick this when you want a free, focused harm-category classifier for text and images.
- Pick this over Guardrails AI when all you need is harm classification, not structured-output enforcement or re-ask loops.
- Pick this over NeMo Guardrails when you do not need Colang flows, RAG retrieval rails, or dialog management.
- Pick this over Lakera when prompt-injection defense, PII detection, and provider-agnostic screening are not required and cost is a constraint.
- Pick this as one layer in a defense-in-depth stack alongside other guardrails.
- Pick this when image moderation via the same endpoint is a hard requirement.

### Related Terms and Aliases

- OpenAI moderation endpoint
- /v1/moderations
- omni-moderation
- text-moderation-007
- harm classification API
- content policy filter
- safety classifier
- image content moderation
- multimodal safety
- CSAM detection (sexual/minors)
- harassment classifier
- free content safety API
