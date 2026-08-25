# Supabase Knowledge Base Population

Populate Supabase as a vector-backed knowledge base from in-repo n8n workflows and JS chunks.

What this repo is

This repository contains n8n workflows and helper data used to create and insert short text "chunks" into a Supabase-based vector knowledge_base. It automates: preparing text chunks, generating OpenAI embeddings, and inserting documents/embeddings into a Supabase table so they can be used for semantic search or retrieval.

Stack
- **Language(s):** n8n workflow JSON, JavaScript (n8n Code nodes), JSON
- **Framework / runtime:** n8n (workflow automation) — import/run the workflows in n8n cloud or a self-hosted n8n instance
- **Notable libraries / services:** OpenAI Embeddings (text-embedding-3-large), Supabase (Postgres + vector support), n8n LangChain integration nodes (vectorStoreSupabase, embeddingsOpenAi)

How it's organized

```
README.md                                 # This file
Populate Knowledge Base.json              # n8n workflow: emits chunks -> prepare -> OpenAI embeddings -> insert via HTTP into Supabase
KB Add Summary Chunks (one-off).json      # n8n workflow: one-off summary-chunks loader using LangChain nodes
Populate Knowledge Base.png                # diagram / screenshot for the workflow
KB Add Summary Chunks (one-off).png        # diagram / screenshot for the one-off workflow
```

How it fits together
- The primary workflow (Populate Knowledge Base.json) emits or loads an array of content chunks, normalizes/sets text and metadata using n8n Set/Code nodes, calls the OpenAI embeddings endpoint to compute vector embeddings, and then issues HTTP POST requests to your Supabase REST endpoint to insert documents into a `knowledge_base` table.
- The one-off workflow (KB Add Summary Chunks (one-off).json) demonstrates a LangChain-based approach (n8n LangChain nodes) that prepares documents and calls the LangChain Supabase vector store node to insert items directly.

How to run it

Shortest path (n8n UI):
1. Clone this repository:

   git clone https://github.com/asadxagentic-ai/supabase-knowledge-base-population.git

2. Open your n8n instance (cloud or self-hosted) and import a workflow (Settings > Import > upload the JSON file):
   - Import `Populate Knowledge Base.json` to run the chunked insert flow.
   - Or import `KB Add Summary Chunks (one-off).json` to run the LangChain node-based one-off loader.

3. Configure credentials in n8n:
   - OpenAI: set an OpenAI API credential (the workflows expect a credential named like "OpenAI account" in the JSON). Provide an OpenAI API key with permissions to call the embeddings endpoint.
   - Supabase: create a HTTP / REST credential or a Supabase credential (the one-off workflow uses the LangChain Supabase credential). Provide:
     - SUPABASE_URL (e.g. https://your-project.supabase.co)
     - SUPABASE_SERVICE_ROLE_KEY or SUPABASE_ANON_KEY (service role key for writes via REST is common in scripts; rotate and protect)

4. (Optional) Verify or create the `knowledge_base` table schema in your Supabase database. Example minimal schema compatible with common vector setups:

   -- Example SQL (Postgres + pgvector / vector embedding column)
   CREATE TABLE public.knowledge_base (
     id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
     content text NOT NULL,
     metadata jsonb,
     embedding vector(1536) -- size depends on model; update to match model embedding dimension
   );

   -- If you use pgvector, create an index for similarity search:
   CREATE INDEX ON public.knowledge_base USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);

   Note: The workflows in this repo insert via the Supabase REST API; ensure the POST payload matches your columns (content, metadata, embedding) or adapt the HTTP/JSON body in the workflow.

5. Run the imported workflow in n8n. The workflow will emit prepared chunks, request embeddings from OpenAI, and then insert records into your Supabase table.

Security note
- The workflow JSONs in the repo may contain placeholder or previously-saved credentials/keys inside the exported nodes. Do NOT use those values. Rotate any leaked keys and store credentials securely in n8n's credential manager or environment variables.

Try asking
- How can I change the workflow to store additional metadata (source URL, author) for each chunk in `knowledge_base`?
- What SQL schema should I use if I want to store embeddings as float8[] instead of pgvector's `vector` type?
- How do I adapt the workflows to call a different embeddings provider or to use a local model instead of OpenAI?

License & notes
- This repository is a small collection of n8n workflow exports and example diagrams. It is intended as a starting point — adapt credentials, table schema, and models to your environment.
