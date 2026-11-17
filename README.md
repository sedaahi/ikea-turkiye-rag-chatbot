# IKEA Türkiye RAG Chatbot

A Turkish-language RAG chatbot built with **n8n, Google Gemini, and Supabase pgvector**. It retrieves information from public IKEA Türkiye webpages and generates answers with source references.

Developed as an educational project during the **Tech Istanbul Applied AI Development Bootcamp**.

> This is an independent educational project. It is not affiliated with or endorsed by IKEA.

## Features

- Separate workflows for document ingestion and chat.
- Automated webpage fetching, HTML text extraction, and text preparation.
- Text chunking with source names and URLs stored as metadata.
- Gemini embeddings stored in Supabase PostgreSQL with pgvector.
- Semantic retrieval to provide relevant context to the language model.
- Turkish answers with supporting source references.
- Prompt instructions to acknowledge missing information and treat retrieved content as data.

## Technology Stack

| Component | Technology |
|---|---|
| Workflow automation | n8n |
| Language model | Google Gemini |
| Embeddings | Google Gemini Embeddings |
| Vector database | Supabase PostgreSQL + pgvector |
| Text splitting | Recursive Character Text Splitter |
| Custom processing | JavaScript in n8n Code nodes |

## Architecture

### Document Ingestion

The ingestion workflow:

1. Reads the list of source URLs.
2. Fetches the webpages through HTTP requests.
3. Extracts text from the HTML.
4. Prepares documents and source metadata.
5. Splits the documents into overlapping chunks.
6. Generates embeddings with Gemini.
7. Stores the chunks, embeddings, and metadata in Supabase.

### Chat and Retrieval

The chat workflow:

1. Receives a question through the n8n Chat Trigger.
2. Embeds the question and retrieves relevant chunks from Supabase.
3. Combines the retrieved text and source metadata into context.
4. Passes the question and context to Gemini.
5. Returns an answer with supporting source references.

The system prompt instructs the model to use only the supplied context and explicitly state when the sources do not contain the requested information.

## Configuration

| Parameter | Value |
|---|---|
| Document table | `public.documents` |
| Search function | `public.match_documents` |
| Embedding dimensions | 3072 |
| Chunk size | 800 characters |
| Chunk overlap | 100 characters |
| Search function default result count | 4 |
| Source webpages | 2 |

The embedding model must produce vectors with **3072 dimensions**. Use the same embedding model for ingestion and retrieval.

The exported embedding nodes rely on n8n defaults rather than explicitly specifying a model. Verify the model and output dimensions in your environment before indexing.

The exported chat workflow references `models/gemini-3.5-flash`. Select an available Gemini chat model if that identifier is unavailable in your environment.

## Repository Structure

| Path | Description |
|---|---|
| `workflows/ingestion.json` | Webpage extraction, chunking, and vector indexing |
| `workflows/chat.json` | Question retrieval and answer generation |
| `sql/setup.sql` | Database extension, document table, and search function |
| `docs/screenshots/services-answer.png` | Workflow and source-referenced service answer |
| `docs/screenshots/collection-policy.png` | Source-referenced collection policy answer |
| `docs/screenshots/out-of-scope.png` | Unsupported question example |

## Setup

### Prerequisites

- An n8n instance with the nodes used in the exported workflows.
- A Supabase project.
- Google Gemini API access.

### 1. Prepare Supabase

Open the Supabase SQL Editor and run the contents of:

`sql/setup.sql`

The script creates:

- The pgvector extension if it is not already installed.
- A `public.documents` table for text, metadata, and 3072-dimensional embeddings.
- A `public.match_documents` function for similarity search and metadata filtering.

The script is intended as a starting point for a new project. It does not migrate an existing table with a different schema.

### 2. Import the Workflows

Import these files into n8n:

- `workflows/ingestion.json`
- `workflows/chat.json`

### 3. Configure Credentials and Models

In both workflows:

- Select your own Google Gemini credentials for the embedding nodes.
- Select your own Supabase credentials for the vector store nodes.
- Confirm the vector store uses the `documents` table.
- Verify that both embedding nodes use the same model and produce 3072-dimensional vectors.

In the chat workflow:

- Select Google Gemini credentials for the chat model.
- Verify that the selected chat model is available.
- Confirm the vector search function is `match_documents`.

Credentials and database contents are not included in this repository.

### 4. Index the Sources

Run the ingestion workflow manually.

Check that document chunks and embeddings have been inserted into the Supabase `documents` table.

**Re-running ingestion currently inserts documents again and may create duplicates.**

### 5. Test the Chatbot

Open the chat workflow and start a test chat through the Chat Trigger.

Example questions:

- IKEA Türkiye hangi montaj, nakliye ve ölçüm hizmetlerini sunuyor?
- Siparişimi seçtiğim gün ve saatte mağazadan teslim almazsam ne olur?
- IKEA Türkiye 2027 yılında hangi şehirlerde yeni mağaza açacak?

## Example Evaluation

The following cases were manually tested during development. They document observed behavior at the time of testing; they are not a comprehensive accuracy benchmark.

| Scenario | Question | Observed behavior |
|---|---|---|
| Service information | IKEA Türkiye hangi montaj, nakliye ve ölçüm hizmetlerini sunuyor? | Listed services with a supporting source link. |
| Specific policy | Siparişimi seçtiğim gün ve saatte mağazadan teslim almazsam ne olur? | Returned the collection policy described in the retrieved source, with a reference. |
| Unsupported information | IKEA Türkiye 2027 yılında hangi şehirlerde yeni mağaza açacak? | Responded: “Bu bilgiyi incelediğim kaynaklarda bulamadım.” |

## Screenshots

### Workflow and Source-Referenced Service Answer

![Chat workflow and service answer](docs/screenshots/services-answer.png)

### Collection Policy Answer

![Collection policy answer with source reference](docs/screenshots/collection-policy.png)

### Unsupported Question

![Chatbot acknowledging information missing from its sources](docs/screenshots/out-of-scope.png)

## Limitations

- Knowledge is limited to the two indexed webpages.
- Webpage content and structure may change after development.
- Stored content remains unchanged until ingestion is run again.
- Repeated ingestion may create duplicate records.
- Retrieval can miss relevant information.
- Prompt instructions do not guarantee factual correctness.
- Model availability and n8n defaults may differ between environments.
- The current evaluation consists of a small set of manual tests.

## Sources

- [IKEA Türkiye — Hizmetler](https://www.ikea.com.tr/hizmetler)
- [IKEA Türkiye — Mağazadan Teslim Al](https://www.ikea.com.tr/evden-alisveris-yap-magazadan-teslim-al)

Source content belongs to its respective owner.

## Author

**Seda Ahi**

[GitHub](https://github.com/sedaahi)
