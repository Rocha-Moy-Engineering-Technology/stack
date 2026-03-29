# Unstructured

> Open-source ETL library for converting documents into structured data for language models

| Field | Value |
|-------|-------|
| Name | Unstructured |
| Group | Data Pipelines |
| Type | SDK |
| Open Source | yes |
| GitHub | [Unstructured-IO/unstructured](https://github.com/Unstructured-IO/unstructured) |
| Stars | 14118 |
| Docs | [docs.unstructured.io](https://docs.unstructured.io/open-source/ingestion/overview) |

## Overview

Unstructured is an open-source document ingestion and processing platform that converts unstructured data from files (PDFs, Word documents, images, emails, HTML, and 25+ other formats) into structured, enriched content suitable for large language models (LLMs) and Retrieval-Augmented Generation (RAG) systems [1].

The platform is available through three interfaces [1]:

- **Unstructured UI**: No-code web interface for batch processing
- **Unstructured API**: REST API with full feature access
- **Open Source Library**: Python library and CLI for programmatic use

The open-source library provides four core operations: **partitioning** (extracting structured elements from documents), **cleaning** (removing unwanted content), **extraction** (retrieving specific content), and **staging** (preparing data for downstream applications). An ingestion pipeline extends these with source/destination connectors for batch ETL workflows [2].

## Core Concepts

### Partitioning

Partitioning is the core operation that converts raw documents into structured **elements** — semantic units like Title, NarrativeText, ListItem, Table, Image, Header, Footer, and PageBreak. The `partition()` function auto-detects file types using `libmagic` and routes to appropriate handlers [3].

Four partitioning strategies control accuracy-speed tradeoffs for PDFs and images [3]:

- **auto** (default): Selects strategy based on document characteristics
- **hi_res**: Uses `detectron2_onnx` layout analysis for precise element classification; best for structured documents with tables
- **fast**: Uses `pdfminer` text extraction; recommended for standard PDFs with extractable text
- **ocr_only**: Runs Tesseract OCR then processes via text partitioning; handles multi-column layouts and scanned documents

### Document Elements

Elements are the output units of partitioning. Each element has a type (Title, NarrativeText, ListItem, Table, etc.) and carries metadata including page numbers, element classification, coordinates, and source information. Email-specific elements include Subject, Sender, and Recipient [3].

### Chunking

Chunking operates on partitioned elements (not raw text), combining consecutive elements to form chunks as large as possible without exceeding a maximum size. Two strategies are available [4]:

- **basic**: Sequentially combines elements respecting character limits; tables remain isolated
- **by_title**: Preserves section boundaries by treating Title elements as section starts; supports `multipage_sections` and `combine_text_under_n_chars` parameters

Key parameters: `max_characters` (hard limit, default 500), `new_after_n_chars` (soft limit), `overlap` (characters shared between split chunks), `overlap_all` (extend overlap to all chunks) [4].

### Ingestion Pipeline

The ingestion pipeline follows an 11-step ETL workflow [1]:

1. **Index** → 2. **Post-Index Filter** → 3. **Download** → 4. **Post-Download Filter** → 5. **Uncompress** → 6. **Post-Uncompress Filter** → 7. **Partition** → 8. **Chunk** → 9. **Embed** → 10. **Stage** → 11. **Upload**

Filtering can be applied at three stages (post-index, post-download, post-uncompress) to exclude files by type, name, path, or size before processing [1].

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Source Connectors (34)               │
│  S3, Azure, GCS, Local, Dropbox, Google Drive,      │
│  SharePoint, Confluence, Slack, GitHub, PostgreSQL...│
└──────────────────────┬──────────────────────────────┘
                       │
                       v
┌─────────────────────────────────────────────────────┐
│              Ingestion Pipeline                      │
│                                                      │
│  ┌───────┐  ┌────────┐  ┌──────────┐  ┌─────────┐ │
│  │ Index │→ │Download │→ │Uncompress│→ │Partition │ │
│  └───────┘  └────────┘  └──────────┘  └────┬────┘ │
│                                              │      │
│  Filters applied at 3 stages                 v      │
│                                        ┌──────────┐ │
│  ┌───────┐  ┌───────┐  ┌──────────┐  │  Chunk   │ │
│  │Upload │← │ Stage │← │  Embed   │← │          │ │
│  └───────┘  └───────┘  └──────────┘  └──────────┘ │
└──────────────────────┬──────────────────────────────┘
                       │
                       v
┌─────────────────────────────────────────────────────┐
│              Destination Connectors (36)              │
│  Pinecone, Qdrant, Weaviate, Milvus, Chroma,       │
│  Elasticsearch, PostgreSQL, S3, Snowflake, Neo4j... │
└─────────────────────────────────────────────────────┘
```

The partitioning engine supports multiple strategies with pluggable OCR backends (Tesseract, PaddleOCR) and layout detection models (detectron2_onnx). Connectors are modular — any source can connect to any destination [1].

## Key Features

- **25+ File Formats**: PDF, DOCX, PPTX, XLSX, HTML, Markdown, XML, CSV, TSV, JSON, EML, MSG, EPUB, RTF, ODT, images (PNG, JPG, TIFF, BMP, HEIC), and more
- **Layout Detection**: Hi-res strategy uses detectron2_onnx for precise element classification in structured documents
- **OCR Support**: Tesseract OCR with multi-language support via ISO 639-3 codes; PaddleOCR as alternative backend
- **Table Extraction**: Extracts tables as HTML representation via `text_as_html` metadata field
- **Image Extraction**: Extracts images with base64 encoding from PDFs and documents
- **Semantic Chunking**: Element-aware chunking that respects document structure (sections, titles, page boundaries)
- **Text Cleaning**: Functions for removing bullets, dashes, non-ASCII characters, extra whitespace, punctuation, and unicode quote normalization
- **Text Translation**: Built-in translation using Helsinki NLP MT models via transformers
- **34 Source Connectors**: S3, Azure, GCS, Google Drive, SharePoint, Confluence, Dropbox, Slack, GitHub, GitLab, Kafka, PostgreSQL, MongoDB, Salesforce, Jira, Notion, and more
- **36 Destination Connectors**: Pinecone, Qdrant, Weaviate, Milvus, Chroma, Elasticsearch, Neo4j, PostgreSQL, Snowflake, S3, DuckDB, LanceDB, Redis, and more
- **Embedding Integration**: Support for OpenAI, HuggingFace, Amazon Bedrock, Vertex AI, Voyage AI, OctoAI, and together.ai via the ingest pipeline
- **Multi-Stage Filtering**: Filter files by type, name, path, or size at three pipeline stages
- **Batch Processing**: Process large file collections with asynchronous and multiprocessing execution
- **Email Processing**: Parse EML and MSG formats with header extraction (Subject, From, To, CC, BCC) and attachment processing

## Use Cases

- **RAG Data Preparation**: Extract and chunk documents from diverse sources into vector-database-ready formats for Retrieval-Augmented Generation pipelines
- **Document Intelligence**: Parse PDFs, scanned documents, and images to extract structured content including tables, headers, and narrative text
- **Knowledge Base Construction**: Ingest documents from SharePoint, Confluence, Google Drive, and other enterprise sources into searchable knowledge stores
- **Email Processing**: Extract structured content from email archives for compliance, search, and analysis
- **Data Lake Ingestion**: Convert unstructured files from cloud storage (S3, GCS, Azure) into structured JSON for data warehouses
- **Multi-Language Document Processing**: Process documents in multiple languages with OCR and translation capabilities
- **Legal and Financial Document Parsing**: Extract structured data from contracts, reports, and regulatory filings

## API Reference

### Partitioning

```python
from unstructured.partition.auto import partition

# Auto-detect file type and partition
elements = partition(filename="document.pdf")

# With strategy selection
elements = partition(filename="scan.pdf", strategy="hi_res")

# With OCR language support
elements = partition(filename="german_doc.pdf", languages=["eng", "deu"])

# From URL
elements = partition(url="https://example.com/report.pdf")
```

### Type-Specific Partitioning

```python
from unstructured.partition.pdf import partition_pdf
from unstructured.partition.html import partition_html
from unstructured.partition.email import partition_email

# PDF with hi-res layout detection
elements = partition_pdf("document.pdf", strategy="hi_res")

# HTML from URL
elements = partition_html(url="https://example.com")

# Email with attachments
elements = partition_email(
    filename="message.eml",
    include_headers=True,
    process_attachments=True,
)
```

### Chunking

```python
from unstructured.partition.auto import partition
from unstructured.chunking.basic import chunk_elements

# Chunk during partitioning
chunks = partition("document.pdf", chunking_strategy="basic")

# Chunk separately
elements = partition("document.pdf")
chunks = chunk_elements(elements, max_characters=1000, overlap=200)
```

### Cleaning

```python
from unstructured.cleaners.core import clean, clean_non_ascii_chars

# Multi-option cleaning
result = clean("● An excellent point!", bullets=True, lowercase=True)
# Returns: "an excellent point!"

# Remove non-ASCII characters
result = clean_non_ascii_chars("\x88Text with ®special chars●")
# Returns: "Text with special chars"
```

### API-Based Processing

```python
from unstructured.partition.api import partition_via_api

elements = partition_via_api(
    filename="document.pdf",
    api_key="YOUR_API_KEY",
    strategy="auto",
)
```

## Configuration

### Partitioning Parameters

- `strategy`: Processing approach — `auto`, `hi_res`, `fast`, `ocr_only` (default: `auto`)
- `languages`: OCR language codes as list (default: English)
- `include_page_breaks`: Include PageBreak elements (default: False)
- `max_partition`: Character limit per element (default: 1500)
- `content_type`: Override MIME type auto-detection
- `extract_image_block_to_payload`: Extract images as base64 (default: False) [3]

### Chunking Parameters

- `chunking_strategy`: `basic` or `by_title`
- `max_characters`: Hard character limit per chunk (default: 500)
- `new_after_n_chars`: Soft limit for preferred chunk size
- `overlap`: Characters shared between split chunks (default: 0)
- `multipage_sections`: Allow chunks to span pages (default: True, by_title only)
- `combine_text_under_n_chars`: Merge small sections (by_title only) [4]

### Embedding Configuration

Embedding providers are configured within the ingest pipeline. Supported providers: Amazon Bedrock (Titan), HuggingFace, OctoAI, OpenAI, together.ai, Vertex AI, and Voyage AI [7].

## Integration Patterns

### LangChain Integration

Unstructured provides LangChain document loaders for direct integration with LangChain RAG pipelines.

### Vector Database Integration

The destination connector system supports direct ingestion into vector databases: Pinecone, Qdrant, Weaviate, Milvus, Chroma, Elasticsearch, LanceDB, Vectara, and Redis.

### Third-Party Embedding

```python
from langchain.embeddings import HuggingFaceEmbeddings
from unstructured.staging.base import elements_from_json

# Load processed elements
elements = elements_from_json(filename="output.json")

# Generate embeddings with external library
embedder = HuggingFaceEmbeddings(model_name="sentence-transformers/all-MiniLM-L6-v2")
for element in elements:
    element["embeddings"] = embedder.embed_query(str(element))
```

### Ingest Pipeline (Python)

```python
import airbyte as ab
from unstructured.staging.base import elements_from_json

# Rehydrate JSON output into element objects
elements = elements_from_json(filename="path/to/output.json")
```

## Examples

### Basic Document Processing

```python
from unstructured.partition.auto import partition

# Process a PDF
elements = partition(filename="annual_report.pdf", strategy="hi_res")

# Filter to narrative text only
narratives = [e for e in elements if e.category == "NarrativeText"]

# Access element metadata
for element in elements:
    print(f"Type: {element.category}, Page: {element.metadata.page_number}")
    print(f"Text: {element.text[:100]}")
```

### End-to-End RAG Preparation

```python
from unstructured.partition.auto import partition
from unstructured.chunking.basic import chunk_elements

# Partition document
elements = partition("research_paper.pdf", strategy="hi_res")

# Chunk for vector database
chunks = chunk_elements(
    elements,
    max_characters=1000,
    new_after_n_chars=800,
    overlap=100,
)

# Access original elements from chunks
for chunk in chunks:
    pages = {e.metadata.page_number for e in chunk.metadata.orig_elements}
    print(f"Chunk covers pages: {pages}")
    print(f"Text: {chunk.text[:200]}")
```

### Email Processing

```python
from unstructured.partition.email import partition_email

elements = partition_email(
    filename="message.eml",
    include_headers=True,
    process_attachments=True,
)

for element in elements:
    if hasattr(element.metadata, 'sent_from'):
        print(f"From: {element.metadata.sent_from}")
    print(f"{element.category}: {element.text[:100]}")
```

## Limitations

- **Open source not actively updated**: The documentation notes that the open-source tools "are not actively updated with latest features" — the API and UI receive priority updates [1]
- **System dependency complexity**: Full installation requires libmagic, poppler, tesseract, libreoffice, and pandoc as system-level dependencies [6]
- **Hi-res strategy requirements**: The `hi_res` partitioning strategy requires `detectron2_onnx` and additional ML model dependencies; falls back to `ocr_only` if unavailable [3]
- **Multi-column PDF limitations**: The `hi_res` strategy "excels with structured documents but struggles with multi-column layouts"; `ocr_only` handles these better [3]
- **Format conversion dependencies**: Processing `.doc`, `.ppt`, `.epub`, `.rst`, `.rtf`, and `.odt` requires intermediate conversion via LibreOffice or Pandoc [3]
- **PGP-encrypted emails**: Encrypted email files return empty element lists with warnings [3]
- **No built-in embedding**: The open-source library does not directly call embedding providers; embedding requires the ingest pipeline or external libraries [7]
- **Metadata loss during chunking**: Chunking consolidates elements, losing granular metadata (page numbers, coordinates); original elements can be accessed via `metadata.orig_elements` [4]
- **Per-page billing**: The managed platform charges per page (PDFs/presentations) or per 100 KB increment (other formats) [1]

## Changelog

- **Ingestion Pipeline**: 11-step ETL pipeline with source/destination connectors, filtering, chunking, and embedding
- **34 Source Connectors**: Cloud storage, SaaS platforms, databases, and messaging systems
- **36 Destination Connectors**: Vector databases, cloud storage, data warehouses, and graph databases
- **Hi-Res Strategy**: detectron2_onnx layout detection for precise document element classification
- **Chunking Strategies**: Basic and by_title strategies with overlap and section-boundary awareness
- **Multi-Language OCR**: Tesseract and PaddleOCR support with ISO 639-3 language codes
- **API Processing**: Remote partitioning via `partition_via_api` and batch processing via `partition_multiple_via_api`
- **Direct-Load Table Support**: Emerging integration for typed destination outputs

## Citations

- [1] Ingestion Overview - https://docs.unstructured.io/open-source/ingestion/overview
- [2] Core Functionality Overview - https://docs.unstructured.io/open-source/core-functionality/overview
- [3] Partitioning - https://docs.unstructured.io/open-source/core-functionality/partitioning
- [4] Chunking - https://docs.unstructured.io/open-source/core-functionality/chunking
- [5] Cleaning - https://docs.unstructured.io/open-source/core-functionality/cleaning
- [6] Full Installation - https://docs.unstructured.io/open-source/installation/full-installation
- [7] Embedding - https://docs.unstructured.io/open-source/core-functionality/embedding
- [8] Source Connectors - https://docs.unstructured.io/open-source/ingestion/source-connectors/overview
- [9] Destination Connectors - https://docs.unstructured.io/open-source/ingestion/destination-connectors/overview
