# Q&A over Google Drive docs

Answers a question from Google Docs stored in one Drive folder. Retrieval is lexical: the workflow downloads the docs, splits them into paragraphs, keeps the paragraphs that share terms with the question, and asks the model to answer only from those passages.

## Use case

A teammate asks "what is our refund window?" or "which plan includes SSO?" and the answer has to come from the docs you already keep in Drive, with the file names attached. This is the small-corpus version of retrieval augmented generation. It does not need a vector database. When the folder grows past a handful of documents, swap **Retrieve Passages** for a vector store and keep the same answer prompt.

## Required credentials

| Node | Credential |
| --- | --- |
| Search Docs and Download Doc | Google Drive OAuth2 (`googleDriveOAuth2Api`) with access to the folder |
| OpenAI Chat Model | OpenAI API (`openAiApi`) |

## Setup

1. Import `workflow.json`.
2. Put the Drive folder id in **Config**. Open the folder in the browser and copy the id at the end of the URL, after `/folders/`.
3. Put the source material in that folder as Google Docs. The download step converts Docs to plain text. PDFs and uploaded Office files are not converted by this template.
4. `maxContextChars` caps how much text is sent to the model (default 8000). **Search Docs** reads at most 8 documents.
5. Attach credentials and activate the workflow.

## Input

`POST` JSON to the webhook path `drive-docs-qa`:

```json
{ "question": "How long do customers have to request a refund?" }
```

The response is JSON: `question`, `answer`, `sources` (file names), and `chunkCount`.

## How to customize

- Tighten the Drive query on **Search Docs** if the folder also contains drafts you want to skip. It already excludes trashed files and non-Docs.
- Edit the stop-word list and the top-6 cap inside **Retrieve Passages**. A term has to be longer than two characters to count.
- The answer prompt forbids outside knowledge. Remove that sentence if you want the model to fill gaps, and accept that it may leave the documents.
- To index PDFs, add an Extract From File PDF branch next to the Docs download.

## Test

Add a short Google Doc that states a specific policy, ask that question through the webhook, and confirm the file name appears in `sources`. Ask something the doc does not cover and confirm the answer says it does not know. Point `folderId` at an empty folder and confirm the no-docs response.
