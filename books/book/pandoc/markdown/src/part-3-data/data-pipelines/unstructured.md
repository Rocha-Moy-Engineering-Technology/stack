[Header 1 ("unstructured", [], []) [Str "Unstructured"], BlockQuote [Para [Str "Open-source ETL library for converting documents into structured data for language models"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Unstructured"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Data Pipelines"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Unstructured-IO/unstructured"] ("https://github.com/Unstructured-IO/unstructured", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "14118"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Docs"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "docs.unstructured.io"] ("https://docs.unstructured.io/open-source/ingestion/overview", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Unstructured is an open-source document ingestion and processing platform that converts unstructured data from files (PDFs, Word documents, images, emails, HTML, and 25+ other formats) into structured, enriched content suitable for large language models (LLMs) and Retrieval-Augmented Generation (RAG) systems ", Str "[", Str "1", Str "]", Str "."], Para [Str "The platform is available through three interfaces ", Str "[", Str "1", Str "]", Str ":"], BulletList [[Plain [Strong [Str "Unstructured UI"], Str ": No-code web interface for batch processing"]], [Plain [Strong [Str "Unstructured API"], Str ": REST API with full feature access"]], [Plain [Strong [Str "Open Source Library"], Str ": Python library and CLI for programmatic use"]]], Para [Str "The open-source library provides four core operations: ", Strong [Str "partitioning"], Str " (extracting structured elements from documents), ", Strong [Str "cleaning"], Str " (removing unwanted content), ", Strong [Str "extraction"], Str " (retrieving specific content), and ", Strong [Str "staging"], Str " (preparing data for downstream applications). An ingestion pipeline extends these with source/destination connectors for batch ETL workflows ", Str "[", Str "2", Str "]", Str "."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("partitioning", ["unnumbered", "unlisted"], []) [Str "Partitioning"], Para [Str "Partitioning is the core operation that converts raw documents into structured ", Strong [Str "elements"], Str " — semantic units like Title, NarrativeText, ListItem, Table, Image, Header, Footer, and PageBreak. The ", Code ("", [], []) "partition()", Str " function auto-detects file types using ", Code ("", [], []) "libmagic", Str " and routes to appropriate handlers ", Str "[", Str "3", Str "]", Str "."], Para [Str "Four partitioning strategies control accuracy-speed tradeoffs for PDFs and images ", Str "[", Str "3", Str "]", Str ":"], BulletList [[Plain [Strong [Str "auto"], Str " (default): Selects strategy based on document characteristics"]], [Plain [Strong [Str "hi_res"], Str ": Uses ", Code ("", [], []) "detectron2_onnx", Str " layout analysis for precise element classification; best for structured documents with tables"]], [Plain [Strong [Str "fast"], Str ": Uses ", Code ("", [], []) "pdfminer", Str " text extraction; recommended for standard PDFs with extractable text"]], [Plain [Strong [Str "ocr_only"], Str ": Runs Tesseract OCR then processes via text partitioning; handles multi-column layouts and scanned documents"]]], Header 3 ("document-elements", ["unnumbered", "unlisted"], []) [Str "Document Elements"], Para [Str "Elements are the output units of partitioning. Each element has a type (Title, NarrativeText, ListItem, Table, etc.) and carries metadata including page numbers, element classification, coordinates, and source information. Email-specific elements include Subject, Sender, and Recipient ", Str "[", Str "3", Str "]", Str "."], Header 3 ("chunking", ["unnumbered", "unlisted"], []) [Str "Chunking"], Para [Str "Chunking operates on partitioned elements (not raw text), combining consecutive elements to form chunks as large as possible without exceeding a maximum size. Two strategies are available ", Str "[", Str "4", Str "]", Str ":"], BulletList [[Plain [Strong [Str "basic"], Str ": Sequentially combines elements respecting character limits; tables remain isolated"]], [Plain [Strong [Str "by_title"], Str ": Preserves section boundaries by treating Title elements as section starts; supports ", Code ("", [], []) "multipage_sections", Str " and ", Code ("", [], []) "combine_text_under_n_chars", Str " parameters"]]], Para [Str "Key parameters: ", Code ("", [], []) "max_characters", Str " (hard limit, default 500), ", Code ("", [], []) "new_after_n_chars", Str " (soft limit), ", Code ("", [], []) "overlap", Str " (characters shared between split chunks), ", Code ("", [], []) "overlap_all", Str " (extend overlap to all chunks) ", Str "[", Str "4", Str "]", Str "."], Header 3 ("ingestion-pipeline", ["unnumbered", "unlisted"], []) [Str "Ingestion Pipeline"], Para [Str "The ingestion pipeline follows an 11-step ETL workflow ", Str "[", Str "1", Str "]", Str ":"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Index"], Str " → 2. ", Strong [Str "Post-Index Filter"], Str " → 3. ", Strong [Str "Download"], Str " → 4. ", Strong [Str "Post-Download Filter"], Str " → 5. ", Strong [Str "Uncompress"], Str " → 6. ", Strong [Str "Post-Uncompress Filter"], Str " → 7. ", Strong [Str "Partition"], Str " → 8. ", Strong [Str "Chunk"], Str " → 9. ", Strong [Str "Embed"], Str " → 10. ", Strong [Str "Stage"], Str " → 11. ", Strong [Str "Upload"]]]], Para [Str "Filtering can be applied at three stages (post-index, post-download, post-uncompress) to exclude files by type, name, path, or size before processing ", Str "[", Str "1", Str "]", Str "."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], CodeBlock ("", [""], []) "┌─────────────────────────────────────────────────────┐
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
", Para [Str "The partitioning engine supports multiple strategies with pluggable OCR backends (Tesseract, PaddleOCR) and layout detection models (detectron2_onnx). Connectors are modular — any source can connect to any destination ", Str "[", Str "1", Str "]", Str "."], Header 2 ("key-features", ["unnumbered", "unlisted"], []) [Str "Key Features"], BulletList [[Plain [Strong [Str "25+ File Formats"], Str ": PDF, DOCX, PPTX, XLSX, HTML, Markdown, XML, CSV, TSV, JSON, EML, MSG, EPUB, RTF, ODT, images (PNG, JPG, TIFF, BMP, HEIC), and more"]], [Plain [Strong [Str "Layout Detection"], Str ": Hi-res strategy uses detectron2_onnx for precise element classification in structured documents"]], [Plain [Strong [Str "OCR Support"], Str ": Tesseract OCR with multi-language support via ISO 639-3 codes; PaddleOCR as alternative backend"]], [Plain [Strong [Str "Table Extraction"], Str ": Extracts tables as HTML representation via ", Code ("", [], []) "text_as_html", Str " metadata field"]], [Plain [Strong [Str "Image Extraction"], Str ": Extracts images with base64 encoding from PDFs and documents"]], [Plain [Strong [Str "Semantic Chunking"], Str ": Element-aware chunking that respects document structure (sections, titles, page boundaries)"]], [Plain [Strong [Str "Text Cleaning"], Str ": Functions for removing bullets, dashes, non-ASCII characters, extra whitespace, punctuation, and unicode quote normalization"]], [Plain [Strong [Str "Text Translation"], Str ": Built-in translation using Helsinki NLP MT models via transformers"]], [Plain [Strong [Str "34 Source Connectors"], Str ": S3, Azure, GCS, Google Drive, SharePoint, Confluence, Dropbox, Slack, GitHub, GitLab, Kafka, PostgreSQL, MongoDB, Salesforce, Jira, Notion, and more"]], [Plain [Strong [Str "36 Destination Connectors"], Str ": Pinecone, Qdrant, Weaviate, Milvus, Chroma, Elasticsearch, Neo4j, PostgreSQL, Snowflake, S3, DuckDB, LanceDB, Redis, and more"]], [Plain [Strong [Str "Embedding Integration"], Str ": Support for OpenAI, HuggingFace, Amazon Bedrock, Vertex AI, Voyage AI, OctoAI, and together.ai via the ingest pipeline"]], [Plain [Strong [Str "Multi-Stage Filtering"], Str ": Filter files by type, name, path, or size at three pipeline stages"]], [Plain [Strong [Str "Batch Processing"], Str ": Process large file collections with asynchronous and multiprocessing execution"]], [Plain [Strong [Str "Email Processing"], Str ": Parse EML and MSG formats with header extraction (Subject, From, To, CC, BCC) and attachment processing"]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "RAG Data Preparation"], Str ": Extract and chunk documents from diverse sources into vector-database-ready formats for Retrieval-Augmented Generation pipelines"]], [Plain [Strong [Str "Document Intelligence"], Str ": Parse PDFs, scanned documents, and images to extract structured content including tables, headers, and narrative text"]], [Plain [Strong [Str "Knowledge Base Construction"], Str ": Ingest documents from SharePoint, Confluence, Google Drive, and other enterprise sources into searchable knowledge stores"]], [Plain [Strong [Str "Email Processing"], Str ": Extract structured content from email archives for compliance, search, and analysis"]], [Plain [Strong [Str "Data Lake Ingestion"], Str ": Convert unstructured files from cloud storage (S3, GCS, Azure) into structured JSON for data warehouses"]], [Plain [Strong [Str "Multi-Language Document Processing"], Str ": Process documents in multiple languages with OCR and translation capabilities"]], [Plain [Strong [Str "Legal and Financial Document Parsing"], Str ": Extract structured data from contracts, reports, and regulatory filings"]]], Header 2 ("api-reference", ["unnumbered", "unlisted"], []) [Str "API Reference"], Header 3 ("partitioning-1", ["unnumbered", "unlisted"], []) [Str "Partitioning"], CodeBlock ("", ["python"], []) "from unstructured.partition.auto import partition

# Auto-detect file type and partition
elements = partition(filename=\"document.pdf\")

# With strategy selection
elements = partition(filename=\"scan.pdf\", strategy=\"hi_res\")

# With OCR language support
elements = partition(filename=\"german_doc.pdf\", languages=[\"eng\", \"deu\"])

# From URL
elements = partition(url=\"https://example.com/report.pdf\")
", Header 3 ("type-specific-partitioning", ["unnumbered", "unlisted"], []) [Str "Type-Specific Partitioning"], CodeBlock ("", ["python"], []) "from unstructured.partition.pdf import partition_pdf
from unstructured.partition.html import partition_html
from unstructured.partition.email import partition_email

# PDF with hi-res layout detection
elements = partition_pdf(\"document.pdf\", strategy=\"hi_res\")

# HTML from URL
elements = partition_html(url=\"https://example.com\")

# Email with attachments
elements = partition_email(
    filename=\"message.eml\",
    include_headers=True,
    process_attachments=True,
)
", Header 3 ("chunking-1", ["unnumbered", "unlisted"], []) [Str "Chunking"], CodeBlock ("", ["python"], []) "from unstructured.partition.auto import partition
from unstructured.chunking.basic import chunk_elements

# Chunk during partitioning
chunks = partition(\"document.pdf\", chunking_strategy=\"basic\")

# Chunk separately
elements = partition(\"document.pdf\")
chunks = chunk_elements(elements, max_characters=1000, overlap=200)
", Header 3 ("cleaning", ["unnumbered", "unlisted"], []) [Str "Cleaning"], CodeBlock ("", ["python"], []) "from unstructured.cleaners.core import clean, clean_non_ascii_chars

# Multi-option cleaning
result = clean(\"● An excellent point!\", bullets=True, lowercase=True)
# Returns: \"an excellent point!\"

# Remove non-ASCII characters
result = clean_non_ascii_chars(\"\\x88Text with ®special chars●\")
# Returns: \"Text with special chars\"
", Header 3 ("api-based-processing", ["unnumbered", "unlisted"], []) [Str "API-Based Processing"], CodeBlock ("", ["python"], []) "from unstructured.partition.api import partition_via_api

elements = partition_via_api(
    filename=\"document.pdf\",
    api_key=\"YOUR_API_KEY\",
    strategy=\"auto\",
)
", Header 2 ("configuration", ["unnumbered", "unlisted"], []) [Str "Configuration"], Header 3 ("partitioning-parameters", ["unnumbered", "unlisted"], []) [Str "Partitioning Parameters"], BulletList [[Plain [Code ("", [], []) "strategy", Str ": Processing approach — ", Code ("", [], []) "auto", Str ", ", Code ("", [], []) "hi_res", Str ", ", Code ("", [], []) "fast", Str ", ", Code ("", [], []) "ocr_only", Str " (default: ", Code ("", [], []) "auto", Str ")"]], [Plain [Code ("", [], []) "languages", Str ": OCR language codes as list (default: English)"]], [Plain [Code ("", [], []) "include_page_breaks", Str ": Include PageBreak elements (default: False)"]], [Plain [Code ("", [], []) "max_partition", Str ": Character limit per element (default: 1500)"]], [Plain [Code ("", [], []) "content_type", Str ": Override MIME type auto-detection"]], [Plain [Code ("", [], []) "extract_image_block_to_payload", Str ": Extract images as base64 (default: False) ", Str "[", Str "3", Str "]"]]], Header 3 ("chunking-parameters", ["unnumbered", "unlisted"], []) [Str "Chunking Parameters"], BulletList [[Plain [Code ("", [], []) "chunking_strategy", Str ": ", Code ("", [], []) "basic", Str " or ", Code ("", [], []) "by_title"]], [Plain [Code ("", [], []) "max_characters", Str ": Hard character limit per chunk (default: 500)"]], [Plain [Code ("", [], []) "new_after_n_chars", Str ": Soft limit for preferred chunk size"]], [Plain [Code ("", [], []) "overlap", Str ": Characters shared between split chunks (default: 0)"]], [Plain [Code ("", [], []) "multipage_sections", Str ": Allow chunks to span pages (default: True, by_title only)"]], [Plain [Code ("", [], []) "combine_text_under_n_chars", Str ": Merge small sections (by_title only) ", Str "[", Str "4", Str "]"]]], Header 3 ("embedding-configuration", ["unnumbered", "unlisted"], []) [Str "Embedding Configuration"], Para [Str "Embedding providers are configured within the ingest pipeline. Supported providers: Amazon Bedrock (Titan), HuggingFace, OctoAI, OpenAI, together.ai, Vertex AI, and Voyage AI ", Str "[", Str "7", Str "]", Str "."], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("langchain-integration", ["unnumbered", "unlisted"], []) [Str "LangChain Integration"], Para [Str "Unstructured provides LangChain document loaders for direct integration with LangChain RAG pipelines."], Header 3 ("vector-database-integration", ["unnumbered", "unlisted"], []) [Str "Vector Database Integration"], Para [Str "The destination connector system supports direct ingestion into vector databases: Pinecone, Qdrant, Weaviate, Milvus, Chroma, Elasticsearch, LanceDB, Vectara, and Redis."], Header 3 ("third-party-embedding", ["unnumbered", "unlisted"], []) [Str "Third-Party Embedding"], CodeBlock ("", ["python"], []) "from langchain.embeddings import HuggingFaceEmbeddings
from unstructured.staging.base import elements_from_json

# Load processed elements
elements = elements_from_json(filename=\"output.json\")

# Generate embeddings with external library
embedder = HuggingFaceEmbeddings(model_name=\"sentence-transformers/all-MiniLM-L6-v2\")
for element in elements:
    element[\"embeddings\"] = embedder.embed_query(str(element))
", Header 3 ("ingest-pipeline-python", ["unnumbered", "unlisted"], []) [Str "Ingest Pipeline (Python)"], CodeBlock ("", ["python"], []) "import airbyte as ab
from unstructured.staging.base import elements_from_json

# Rehydrate JSON output into element objects
elements = elements_from_json(filename=\"path/to/output.json\")
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("basic-document-processing", ["unnumbered", "unlisted"], []) [Str "Basic Document Processing"], CodeBlock ("", ["python"], []) "from unstructured.partition.auto import partition

# Process a PDF
elements = partition(filename=\"annual_report.pdf\", strategy=\"hi_res\")

# Filter to narrative text only
narratives = [e for e in elements if e.category == \"NarrativeText\"]

# Access element metadata
for element in elements:
    print(f\"Type: {element.category}, Page: {element.metadata.page_number}\")
    print(f\"Text: {element.text[:100]}\")
", Header 3 ("end-to-end-rag-preparation", ["unnumbered", "unlisted"], []) [Str "End-to-End RAG Preparation"], CodeBlock ("", ["python"], []) "from unstructured.partition.auto import partition
from unstructured.chunking.basic import chunk_elements

# Partition document
elements = partition(\"research_paper.pdf\", strategy=\"hi_res\")

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
    print(f\"Chunk covers pages: {pages}\")
    print(f\"Text: {chunk.text[:200]}\")
", Header 3 ("email-processing", ["unnumbered", "unlisted"], []) [Str "Email Processing"], CodeBlock ("", ["python"], []) "from unstructured.partition.email import partition_email

elements = partition_email(
    filename=\"message.eml\",
    include_headers=True,
    process_attachments=True,
)

for element in elements:
    if hasattr(element.metadata, 'sent_from'):
        print(f\"From: {element.metadata.sent_from}\")
    print(f\"{element.category}: {element.text[:100]}\")
", Header 2 ("limitations", ["unnumbered", "unlisted"], []) [Str "Limitations"], BulletList [[Plain [Strong [Str "Open source not actively updated"], Str ": The documentation notes that the open-source tools \"are not actively updated with latest features\" — the API and UI receive priority updates ", Str "[", Str "1", Str "]"]], [Plain [Strong [Str "System dependency complexity"], Str ": Full installation requires libmagic, poppler, tesseract, libreoffice, and pandoc as system-level dependencies ", Str "[", Str "6", Str "]"]], [Plain [Strong [Str "Hi-res strategy requirements"], Str ": The ", Code ("", [], []) "hi_res", Str " partitioning strategy requires ", Code ("", [], []) "detectron2_onnx", Str " and additional ML model dependencies; falls back to ", Code ("", [], []) "ocr_only", Str " if unavailable ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "Multi-column PDF limitations"], Str ": The ", Code ("", [], []) "hi_res", Str " strategy \"excels with structured documents but struggles with multi-column layouts\"; ", Code ("", [], []) "ocr_only", Str " handles these better ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "Format conversion dependencies"], Str ": Processing ", Code ("", [], []) ".doc", Str ", ", Code ("", [], []) ".ppt", Str ", ", Code ("", [], []) ".epub", Str ", ", Code ("", [], []) ".rst", Str ", ", Code ("", [], []) ".rtf", Str ", and ", Code ("", [], []) ".odt", Str " requires intermediate conversion via LibreOffice or Pandoc ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "PGP-encrypted emails"], Str ": Encrypted email files return empty element lists with warnings ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "No built-in embedding"], Str ": The open-source library does not directly call embedding providers; embedding requires the ingest pipeline or external libraries ", Str "[", Str "7", Str "]"]], [Plain [Strong [Str "Metadata loss during chunking"], Str ": Chunking consolidates elements, losing granular metadata (page numbers, coordinates); original elements can be accessed via ", Code ("", [], []) "metadata.orig_elements", Str " ", Str "[", Str "4", Str "]"]], [Plain [Strong [Str "Per-page billing"], Str ": The managed platform charges per page (PDFs/presentations) or per 100 KB increment (other formats) ", Str "[", Str "1", Str "]"]]], Header 2 ("changelog", ["unnumbered", "unlisted"], []) [Str "Changelog"], BulletList [[Plain [Strong [Str "Ingestion Pipeline"], Str ": 11-step ETL pipeline with source/destination connectors, filtering, chunking, and embedding"]], [Plain [Strong [Str "34 Source Connectors"], Str ": Cloud storage, SaaS platforms, databases, and messaging systems"]], [Plain [Strong [Str "36 Destination Connectors"], Str ": Vector databases, cloud storage, data warehouses, and graph databases"]], [Plain [Strong [Str "Hi-Res Strategy"], Str ": detectron2_onnx layout detection for precise document element classification"]], [Plain [Strong [Str "Chunking Strategies"], Str ": Basic and by_title strategies with overlap and section-boundary awareness"]], [Plain [Strong [Str "Multi-Language OCR"], Str ": Tesseract and PaddleOCR support with ISO 639-3 language codes"]], [Plain [Strong [Str "API Processing"], Str ": Remote partitioning via ", Code ("", [], []) "partition_via_api", Str " and batch processing via ", Code ("", [], []) "partition_multiple_via_api"]], [Plain [Strong [Str "Direct-Load Table Support"], Str ": Emerging integration for typed destination outputs"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " Ingestion Overview - https://docs.unstructured.io/open-source/ingestion/overview"]], [Plain [Str "[", Str "2", Str "]", Str " Core Functionality Overview - https://docs.unstructured.io/open-source/core-functionality/overview"]], [Plain [Str "[", Str "3", Str "]", Str " Partitioning - https://docs.unstructured.io/open-source/core-functionality/partitioning"]], [Plain [Str "[", Str "4", Str "]", Str " Chunking - https://docs.unstructured.io/open-source/core-functionality/chunking"]], [Plain [Str "[", Str "5", Str "]", Str " Cleaning - https://docs.unstructured.io/open-source/core-functionality/cleaning"]], [Plain [Str "[", Str "6", Str "]", Str " Full Installation - https://docs.unstructured.io/open-source/installation/full-installation"]], [Plain [Str "[", Str "7", Str "]", Str " Embedding - https://docs.unstructured.io/open-source/core-functionality/embedding"]], [Plain [Str "[", Str "8", Str "]", Str " Source Connectors - https://docs.unstructured.io/open-source/ingestion/source-connectors/overview"]], [Plain [Str "[", Str "9", Str "]", Str " Destination Connectors - https://docs.unstructured.io/open-source/ingestion/destination-connectors/overview"]]]]