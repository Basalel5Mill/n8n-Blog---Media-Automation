# Blog Post Automation Workflow

A complete n8n workflow to automate end-to-end blog publishing—from prompt selection and semantic retrieval to AI-driven writing, image generation, and WordPress publishing.

<p align="center">
  <img src="https://primary-production-2548.up.railway.app/wp-content/uploads/2025/07/n8n-blog.png" alt="Blog Post Automation Workflow" width="600"/>
</p>
---

## Story

> **You:** "I need a fresh blog post tomorrow, no stress."
>
> **Workflow:** "Say no more! I'll pick a topic, pull in your past research, draft a post, whip up a header image, and publish it live—while logging everything for you."

This README captures how this "assistant" works behind the scenes to save you hours each week so you can focus on high-level strategy.

---

## Description

Automates your entire blog-post pipeline:
- **Prompt Selection:** Randomly picks a topic from a Google Sheet.
- **Semantic Retrieval:** Uses Supabase as a vector database (built on Postgres) to store embeddings via Google Gemini Embeddings (`models/text-embedding-004`). This enables fast, context-rich retrieval of your own documents.
- **AI Writing:** Google Gemini Chat (`models/gemini-2.5-flash-preview-05-20`) drafts JSON-formatted posts (title, question, description, slug, content).
- **Image Generation:** Fal-run AI `/fal-ai/imagen4/preview/fast` generates a blurred-background base, then n8n image nodes overlay text and branding.
- **Publishing:** Uploads media and creates posts on WordPress via its REST APIs.
- **Logging:** Appends Title, Description, Slug, Image GUID, and URL to a Google Sheet for auditing.

---

## Prerequisites

1. **n8n** instance (self-hosted or cloud)
2. **Google Account** with:
   - Google Sheets API enabled
   - Google Drive API enabled
3. **Supabase** project with:
   - PostgreSQL database
   - Vector extension installed
4. **WordPress** site with REST API credentials
5. **Fal-run AI** API access for image generation
6. **Environment Variables** set in n8n:
   - `GOOGLE_SHEETS_OAUTH2`
   - `GOOGLE_DRIVE_OAUTH2`
   - `SUPABASE_URL` & `SUPABASE_KEY`
   - `WORDPRESS_API_URL`, `WORDPRESS_USERNAME`, `WORDPRESS_PASSWORD`
   - `FALRUN_API_KEY`

---

## Workflow Overview

1. **Webhook Trigger**
   - Listens for an HTTP request to start the flow.

2. **Fetch Prompts**
   - Node: **Google Sheets – Read Rows**
   - Retrieves rows: `Title`, `Intent`, `Keywords`, `Primary Keyword`.

3. **Select Prompt**
   - Node: **Random Row Selector (Code)**
   - Picks one prompt to keep content varied.

4. **Retrieve Context**
   - Node: **n8n vectorStoreSupabase**
   - Uses embeddings from `models/text-embedding-004` to fetch related docs from Supabase.

5. **Draft Post**
   - Node: **HTTP Request (Chat)**
   - Endpoint: Google Gemini Chat (`models/gemini-2.5-flash-preview-05-20`)
   - System prompt instructs output as JSON with keys: `title`, `question`, `description`, `slug`, `content`.

6. **Map Fields**
   - Node: **Edit Fields**
   - Maps JSON keys to workflow variables.

7. **Generate Header Image**
   - Node: **HTTP Request** to `/fal-ai/imagen4/preview/fast`
   - Sends `question` text for art direction.

8. **Process Image**
   - Nodes:
     - **Download Image** (HTTP Request)
     - **Edit Image**: Overlay Question text, apply extra blur.
     - **Google Drive**: Select random branded overlay asset.
     - **Merge & Resize**: Combine base image and overlay into final header.

9. **Upload Media**
   - Node: **HTTP Request (Media)**
   - Uploads final header image to WordPress; captures `media ID`.

10. **Publish Post**
    - Node: **HTTP Request (Posts)**
    - Uses: `title`, `content`, `featured_media`.

11. **Log Results**
    - Node: **Google Sheets – Append Row**
    - Records: Title, Description, Slug, Image GUID, Post URL.

---

## Configuration

- **Google Sheet URL:** Set in the Google Sheets credentials node.
- **Supabase Credentials:** `SUPABASE_URL` & `SUPABASE_KEY` environment variables.
- **WordPress Endpoint:** `https://your-site.com/wp-json/wp/v2/`.
- **Fal-run AI:** API key in `FALRUN_API_KEY`.

Adjust model names or endpoints as needed.

---

## Contact

For simple front-end integration, custom tweaks, or troubleshooting, email **basalelr@gmail.com**. 👋
