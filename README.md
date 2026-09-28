# Engineering Document RAG & Conflict Detection System

An AI-powered engineering document intelligence platform designed to process **RFQ packages** and analyze information across **BOQ spreadsheets, technical specifications, BOM documents, and CAD drawings**.

The system combines **Hybrid RAG, structured document extraction, CAD parsing, entity normalization, fuzzy matching, and engineering conflict detection** to help users retrieve information from technical documents and identify inconsistencies across multiple engineering sources.

---

## 🚀 Features

* **Hybrid RAG Pipeline:** Combines **FAISS semantic search** and **BM25 keyword retrieval** to improve document-grounded question answering across engineering terminology, identifiers, specifications, and quantities.

* **Multi-Format Engineering Document Processing:** Supports **Excel, PDF, DOCX, CSV, TXT, DXF, and DWG** files through format-specific extraction pipelines.

* **BOQ Intelligence:** Processes Excel-based Bills of Quantities row-by-row, extracting **item descriptions, quantities, units, and sheet-level metadata** for downstream analysis and retrieval.

* **CAD Document Analysis:** Parses **DXF entities** and converts **DWG → DXF** using ODA File Converter, enabling extraction of **layers, blocks, entity types, dimensions, and CAD metadata**.

* **Technical Specification & BOM Extraction:** Extracts structured engineering information such as **materials, tolerances, surface finishes, coatings, heat treatment, and BOM quantities** from technical documents.

* **Engineering Entity Normalization:** Normalizes item names, quantities, punctuation, whitespace, and engineering identifiers before performing cross-document comparison.

* **Fuzzy Entity Matching with Numeric Safety:** Uses fuzzy matching to identify similar engineering items while applying **numeric/dimensional validation** to prevent incorrect matches such as `DN50` being merged with `DN80`.

* **Cross-Document Conflict Detection:** Compares quantities across **BOQ, BOM, CAD, and specification sources** to identify inconsistencies and generate a structured conflict matrix.

* **Persistent Vector Store:** Maintains a persistent **FAISS index and document metadata store**, allowing indexed document data to be reloaded across application restarts.

* **Duplicate Chunk Prevention:** Uses content hashing to prevent identical document chunks from being repeatedly indexed.

* **Context-Grounded AI Answers:** Sends retrieved document context to an OpenAI LLM with instructions to answer using the available evidence and avoid unsupported assumptions.

* **BOQ-Aware Querying:** Provides a dedicated processing path for broad BOQ questions such as **"List all BOQ items"**, avoiding the limitations of retrieving only a small top-K subset.

* **Source Attribution:** Returns source-document information alongside generated answers to provide traceability back to the uploaded RFQ files.

* **Conflict Report Export:** Converts engineering conflict analysis into a structured **CSV report** for further review and downstream processing.

* **Interactive Engineering UI:** Provides a browser-based interface for **RFQ uploads, multi-file analysis, document management, grounded Q&A, conflict visualization, and report export**.

* **REST API Architecture:** Exposes modular FastAPI endpoints for **document processing, multi-file RFQ analysis, querying, quotation analysis, document management, and conflict export**.

* **Configurable AI Pipeline:** Supports configurable **embedding models, LLM models, chunk size, chunk overlap, retrieval depth, context limits, retry behavior, upload limits, and CAD conversion paths** through environment-based configuration.

* **Resilient AI Processing:** Includes **embedding retry logic, request timeouts, file validation, format validation, extraction error handling, and configuration checks** for more reliable document processing.

---

## 🧠 System Architecture

### Document RAG Workflow

```text
                 RFQ Documents
                      │
        ┌─────────────┼─────────────┐
        │             │             │
       PDF          Excel          CAD
       DOCX          BOQ          DWG/DXF
       CSV/TXT
        │             │             │
        └─────────────┼─────────────┘
                      ↓
             Format-Specific
               Extraction
                      ↓
              Text / Entities
                      ↓
             Recursive Chunking
                      ↓
             OpenAI Embeddings
                      ↓
          ┌───────────────────────┐
          │     Hybrid Index     │
          │                       │
          │ FAISS + BM25         │
          │ + Duplicate Control  │
          └───────────┬───────────┘
                      ↓
                Hybrid Search
                      ↓
               Context Builder
                      ↓
                OpenAI LLM
                      ↓
             Grounded AI Answer
                      ↓
              Source Documents
```

### Engineering Conflict Workflow

```text
             Multiple RFQ Files
                     │
                     ↓
             Format Processing
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
      BOQ           BOM           CAD
       │             │             │
       └─────────────┼─────────────┘
                     ↓
             Entity Extraction
                     ↓
             Entity Normalization
                     ↓
               Deduplication
                     ↓
              Fuzzy Matching
                     ↓
             Numeric Validation
                     ↓
             Quantity Aggregation
                     ↓
             Source Comparison
                     ↓
             Conflict Detection
                     ↓
              Conflict Matrix
                     ↓
                CSV Export
```

---

## 🔍 Hybrid RAG Pipeline

The retrieval system combines two complementary search strategies.

### FAISS Semantic Search

Document chunks are converted into embeddings using:

```text
text-embedding-3-large
```

The embeddings are stored in a FAISS index using normalized vectors.

This allows semantically similar questions and document content to be matched even when the wording is different.

### BM25 Keyword Search

BM25 provides lexical retrieval for exact engineering terminology such as:

```text
DN50
ASTM A106
SS316
Valve
Pressure Rating
Material Grade
```

### Combined Retrieval

```text
User Question
     │
     ├───────────────┐
     ↓               ↓
FAISS Search      BM25 Search
     │               │
     └───────┬───────┘
             ↓
      Merge Results
             ↓
     Remove Duplicates
             ↓
          Top-K
             ↓
      Context Builder
             ↓
         OpenAI LLM
```

---

## 📄 Supported File Formats

| Format  | Processing                |
| ------- | ------------------------- |
| `.pdf`  | PDF text extraction       |
| `.docx` | Word document extraction  |
| `.xlsx` | Excel / BOQ processing    |
| `.xls`  | Excel processing          |
| `.csv`  | Structured CSV extraction |
| `.txt`  | Text extraction           |
| `.dxf`  | CAD entity parsing        |
| `.dwg`  | DWG → DXF → CAD parsing   |

### DWG Processing

DWG files require the **ODA File Converter**.

```text
DWG
 ↓
ODA File Converter
 ↓
DXF
 ↓
ezdxf
 ↓
CAD Entity Extraction
```

DXF files can be processed directly without ODA File Converter.

---

## 📊 BOQ Processing

Excel BOQs are processed using a dedicated pipeline.

The system identifies likely header rows using fields such as:

```text
description
item
qty
quantity
unit
```

Example:

```text
Item: V-001
Description: Carbon Steel Valve
Quantity: 10
Unit: Nos
```

The extracted information can then be used for:

* RAG retrieval
* Quantity analysis
* Entity comparison
* Conflict detection

---

## 🧾 Technical Specification Extraction

The specification pipeline can extract engineering attributes such as:

```text
Material
Tolerance
Surface Finish
Coating
Heat Treatment
```

Example:

```text
Material: Stainless Steel
Tolerance: ±0.05 mm
Surface Finish: Ra 3.2
Coating: Zinc
Heat Treatment: Annealed
```

---

## 🔩 BOM Extraction

The BOM extraction pipeline supports structured pipe-separated content.

Example:

```text
Part | Material | Quantity

Pipe | Steel | 10
Valve | Stainless Steel | 5
```

The resulting structured representation contains:

```json
{
  "part": "Valve",
  "material": "Stainless Steel",
  "qty": 5
}
```

---

## 🧩 Engineering Entity Resolution

Before comparing information across files, engineering entities are normalized.

The normalization process includes:

* Lowercasing
* Whitespace normalization
* Punctuation normalization
* Quantity normalization
* Source tracking
* Entity type tracking

Example:

```text
Valve-DN50
```

can be normalized to:

```text
valve dn50
```

---

## 🔎 Fuzzy Matching with Numeric Safety

Engineering documents frequently use slightly different names for the same component.

For example:

```text
Carbon Steel Valve DN50
CS Valve DN50
Valve - DN50
```

Fuzzy matching helps identify potentially related entities.

However, numeric identifiers are validated separately.

Therefore:

```text
Valve DN50
```

and

```text
Valve DN80
```

are not treated as the same engineering entity solely because their textual descriptions are similar.

This numeric safety layer is particularly important for dimensions and engineering identifiers.

---

## ⚠️ Conflict Detection

The system compares extracted entities across different sources.

Example:

```text
BOQ:
Valve DN50 → 10

Specification BOM:
Valve DN50 → 12
```

The system can identify this as a quantity conflict.

Example result:

```json
{
  "entity": "Valve DN50",
  "quantities": {
    "BOQ": 10,
    "Spec BOM": 12
  },
  "conflict_detected": true
}
```

Matching quantities can also be represented as non-conflicting entities.

---

## 📋 Conflict Matrix

The engineering analysis maintains source-specific quantities.

Example:

| Entity     | BOQ | Spec BOM | CAD     | Conflict |
| ---------- | --: | -------: | ------- | -------- |
| Valve DN50 |  10 |       12 | Present | Yes      |
| Pipe DN25  |  20 |       20 | Present | No       |
| Valve DN80 |   5 |        5 | Present | No       |

The complete analysis can be exported as CSV.

---

## 📤 Conflict CSV Export

The system provides an export endpoint that converts conflict analysis into a CSV report.

Example:

```text
Item Name,Conflict Status,Source Quantities

Valve DN50,CONFLICT,BOQ: 10 | Spec BOM: 12
Pipe DN25,MATCH,BOQ: 20 | Spec BOM: 20
```

This allows engineering teams to continue analysis outside the application.

---

## 🤖 Grounded Question Answering

Users can ask questions about uploaded documents.

Examples:

```text
What is the quantity of Valve DN50?
```

```text
What material is specified for the valve?
```

```text
What is the required surface finish?
```

```text
List all BOQ items.
```

The system retrieves relevant document context before sending it to the LLM.

```text
Question
   ↓
Hybrid Retrieval
   ↓
Relevant Chunks
   ↓
Context Construction
   ↓
OpenAI LLM
   ↓
Grounded Answer
   ↓
Sources
```

---

## 📌 BOQ-Aware Querying

Large BOQ spreadsheets can contain many rows.

For broad questions such as:

```text
List all BOQ items.
```

the system can use a dedicated BOQ processing path rather than relying only on top-K vector retrieval.

This allows the complete BOQ representation to be supplied for broad analysis.

---

## 💾 Persistent Vector Store

The application persists the vector index locally.

```text
vectorstore/
├── index.faiss
└── docs.pkl
```

The FAISS index stores vector representations while the document store maintains the corresponding chunks and metadata.

The vector store can be loaded again when the application restarts.

---

## ♻️ Duplicate Prevention

Document chunks are hashed before indexing.

```text
Document Chunk
      ↓
Content Hash
      ↓
Duplicate Check
      ↓
New → Index
Existing → Skip
```

This helps prevent identical chunks from being unnecessarily indexed multiple times.

---

## 🏗️ Project Structure

```text
RFQ-AI-System/
│
├── app/
│   │
│   ├── brain/
│   │   ├── chunk_service.py
│   │   ├── conflict_engine.py
│   │   ├── document_upload.py
│   │   ├── embedding_service.py
│   │   ├── llm_service.py
│   │   └── vector_service.py
│   │
│   ├── extraction/
│   │   ├── bom_extractor.py
│   │   ├── spec_extractor.py
│   │   └── table_extractor.py
│   │
│   ├── models/
│   │   ├── query_model.py
│   │   └── rfq_model.py
│   │
│   ├── pipeline/
│   │   ├── rfq_pipeline.py
│   │   ├── query_pipeline.py
│   │   └── quotation_pipeline.py
│   │
│   ├── routers/
│   │   ├── document_router.py
│   │   ├── export_router.py
│   │   ├── query_router.py
│   │   ├── quote_router.py
│   │   └── upload_router.py
│   │
│   ├── services/
│   │   ├── cad_service.py
│   │   ├── csv_service.py
│   │   ├── docx_service.py
│   │   ├── excel_service.py
│   │   ├── export_service.py
│   │   ├── pdf_service.py
│   │   └── text_service.py
│   │
│   ├── utils/
│   │   └── fuzzy_match.py
│   │
│   ├── web/
│   │   ├── index.html
│   │   ├── app.js
│   │   └── style.css
│   │
│   └── main.py
│
├── uploads/
├── vectorstore/
├── temp_dxf/
├── requirements.txt
├── .env
└── README.md
```

---

## 🛠️ Technology Stack

| Area                 | Technology                      |
| -------------------- | ------------------------------- |
| Language             | Python                          |
| Backend              | FastAPI                         |
| API Server           | Uvicorn                         |
| Validation           | Pydantic                        |
| LLM                  | OpenAI                          |
| Embeddings           | OpenAI `text-embedding-3-large` |
| Vector Search        | FAISS                           |
| Keyword Search       | BM25                            |
| Chunking             | LangChain Text Splitters        |
| Excel                | Pandas, OpenPyXL                |
| PDF                  | PyPDF2                          |
| DOCX                 | python-docx                     |
| CAD                  | ezdxf                           |
| DWG Conversion       | ODA File Converter              |
| Numerical Processing | NumPy                           |
| Frontend             | HTML, CSS, JavaScript           |
| Configuration        | python-dotenv                   |

---

## 🛠️ Setup & Installation

### 1. Prerequisites

* Python 3.9+
* OpenAI API key
* ODA File Converter — only required for `.dwg` processing

### 2. Upgrade Pip

```bash
python -m pip install --upgrade pip
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Create Environment Configuration

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=your_api_key_here

# Only required for DWG processing
ODA_PATH=/path/to/ODAFileConverter

# Optional configuration
CHUNK_SIZE=1000
CHUNK_OVERLAP=200
EMBEDDING_MODEL=text-embedding-3-large
OPENAI_MODEL=gpt-4o-mini
TOP_K=8
MAX_CONTEXT_CHARS=12000
```

**Never commit your real `.env` file or API credentials to GitHub.**

---

## ⚡ Running the Application

Start the FastAPI server:

```bash
uvicorn app.main:app --reload --port 8000
```

Application:

```text
http://127.0.0.1:8000/
```

Swagger API documentation:

```text
http://127.0.0.1:8000/docs
```

ReDoc:

```text
http://127.0.0.1:8000/redoc
```

---

## 🔌 API Endpoints

### Document Management

```text
GET /documents/
DELETE /documents/{filename}
```

### Document Processing

```text
POST /upload/process
POST /upload/bundle
```

### RAG Query

```text
POST /query/ask
```

### Engineering Analysis

```text
POST /quote/excel
```

### Conflict Export

```text
POST /export/conflicts
```

---

## 💬 Example API Query

```json
{
  "question": "What is the quantity of Valve DN50?",
  "top_k": 5
}
```

Example response:

```json
{
  "answer": "The quantity of Valve DN50 is 10.",
  "sources": [
    "project.xlsx"
  ]
}
```

---

## 🔎 Example Queries

Try questions such as:

```text
List all items from the BOQ with their quantities.
```

```text
What is the quantity of Valve DN50?
```

```text
What material is specified for the valve?
```

```text
What are the required surface finish specifications?
```

```text
Are there any quantity conflicts across the RFQ documents?
```

```text
Which engineering items have different quantities between the BOQ and BOM?
```

```text
List all BOQ items.
```

---

## 🧪 Example End-to-End Workflow

Upload an RFQ package containing:

```text
RFQ_BOQ.xlsx
Technical_Spec.pdf
Material_BOM.docx
Plant_Drawing.dxf
```

The system processes the files:

```text
RFQ Package
     │
     ├── BOQ.xlsx
     │      ↓
     │   BOQ Entities
     │
     ├── Technical_Spec.pdf
     │      ↓
     │   Specification Data
     │
     ├── Material_BOM.docx
     │      ↓
     │   BOM Entities
     │
     └── Plant_Drawing.dxf
            ↓
        CAD Entities
```

Then:

```text
Extract
  ↓
Normalize
  ↓
Deduplicate
  ↓
Fuzzy Match
  ↓
Numeric Validation
  ↓
Quantity Comparison
  ↓
Conflict Detection
  ↓
Conflict Report
```

At the same time, the documents become available for RAG-based question answering.

---

## 🛡️ Error Handling & Reliability

The application includes handling for common processing failures such as:

* Unsupported file formats
* Missing files
* Oversized files
* Empty documents
* Invalid CAD files
* Missing ODA configuration
* Embedding API failures
* LLM failures
* Vector-store loading issues

The embedding pipeline also uses retry handling and request timeouts for temporary API failures.

---

## ⚙️ Configuration

Important configuration parameters include:

| Variable            | Purpose                            |
| ------------------- | ---------------------------------- |
| `OPENAI_API_KEY`    | OpenAI authentication              |
| `OPENAI_MODEL`      | LLM model                          |
| `EMBEDDING_MODEL`   | Embedding model                    |
| `CHUNK_SIZE`        | Chunk size                         |
| `CHUNK_OVERLAP`     | Chunk overlap                      |
| `TOP_K`             | Retrieval depth                    |
| `MAX_CONTEXT_CHARS` | Maximum RAG context                |
| `MAX_RETRIES`       | API retry count                    |
| `UPLOAD_DIR`        | Uploaded document directory        |
| `SAVE_DIR`          | Persistent vector store            |
| `DWG_TEMP_DIR`      | Temporary CAD conversion directory |
| `ODA_PATH`          | ODA converter executable           |

---

## ⚠️ Current Limitations

The current implementation intentionally focuses on document intelligence and engineering data reconciliation.

### Document Deletion

The current document management flow does not fully synchronize deletion with the persisted FAISS/BM25 indexes.

### Authentication

Authentication and authorization are not currently implemented.

### Multi-Tenant Isolation

The vector store is currently shared rather than isolated by user or tenant.

### CAD Intelligence

CAD processing focuses on supported DXF entities and metadata extraction. It is not a complete geometric or semantic CAD understanding system.

### Citations

The current RAG response provides source-document attribution rather than full page-level or chunk-level citations.

### BOM Format

The current BOM extraction logic expects specific structured patterns and may require additional parsing for highly varied BOM formats.

---

## 🔮 Future Improvements

Potential extensions include:

* Page-level and chunk-level citations
* Metadata-aware retrieval
* Retrieval reranking
* MMR retrieval
* Engineering revision comparison
* Material conflict detection
* Dimension conflict detection
* Unit mismatch detection
* Missing component detection
* BOQ vs CAD reconciliation
* BOQ vs specification reconciliation
* Authentication and authorization
* Project-level document isolation
* Database-backed document management
* Production object storage
* Managed vector database integration
* Automated document versioning
* Advanced CAD geometry analysis
* Expanded engineering rule validation

---

## 🎯 Design Principles

### Format-Specific Processing

Different engineering formats require different extraction strategies rather than treating every file as plain text.

### Hybrid Retrieval

Semantic and lexical retrieval are combined to handle both natural-language questions and exact engineering terminology.

### Grounded Generation

The LLM receives retrieved document context instead of being expected to independently know the contents of uploaded engineering documents.

### Numeric Safety

Engineering identifiers and dimensions are validated before fuzzy entities are merged.

### Duplicate Prevention

Content hashing reduces repeated indexing of identical chunks.

### Persistent Indexing

FAISS and document metadata are stored locally so indexed content can survive application restarts.

### Modular Architecture

Document extraction, embeddings, retrieval, pipelines, API routes, and frontend components are separated into dedicated modules.

---

## 📈 Engineering Intelligence Flow

```text
                 RFQ Package
                      │
                      ↓
             Multi-Format Parser
                      │
                      ↓
             Structured Extraction
                      │
          ┌───────────┴───────────┐
          ↓                       ↓
      RAG Pipeline          Conflict Pipeline
          │                       │
          ↓                       ↓
    Chunk + Embed           Normalize Entities
          │                       │
          ↓                       ↓
     FAISS + BM25            Fuzzy Matching
          │                       │
          ↓                 Numeric Validation
    Hybrid Retrieval              │
          │                       ↓
          ↓                 Quantity Comparison
    Context Builder               │
          │                       ↓
          ↓                 Conflict Detection
      OpenAI LLM                   │
          │                       ↓
          ↓                  CSV Report
     Grounded Answer
```

---

## 📌 Project Summary

**Engineering Document RAG & Conflict Detection System** brings together:

```text
Document Intelligence
        +
Hybrid RAG
        +
Structured Extraction
        +
CAD Processing
        +
Entity Resolution
        +
Engineering Validation
        +
Conflict Detection
```

The system is designed to move beyond basic document Q&A toward **engineering-focused RFQ analysis and cross-document information reconciliation**.

It provides a practical foundation for applications involving **technical document intelligence, procurement analysis, engineering validation, and automated RFQ review**.
