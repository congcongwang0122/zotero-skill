---
name: Zotero
description: Use Zotero Desktop from Codex to enable/probe the local API, search a local Zotero library, list items/collections/tags, export BibTeX, insert citation keys into LaTeX or Markdown drafts, read indexed full text and PDF content when requested, and import BibTeX/RIS records into Zotero through the connector server. Use when the user mentions Zotero, citations, references.bib, BibTeX export, local Zotero API, localhost:23119, or adding citations from a Zotero library.
---

# Zotero

Use this skill to operate a user's local Zotero Desktop library from Codex.

## Connection Setup

Before any operation, ensure Zotero is running and the local API is enabled:

1. **Open Zotero Desktop** application
2. **Enable Local API**: Edit > Preferences > Advanced > check "Allow other applications on this computer to communicate with Zotero" (可用于 http://localhost:23119/api/)
3. **Verify connection**:

### PowerShell (Windows)
```powershell
Invoke-WebRequest -Uri "http://127.0.0.1:23119/api/users/0/collections" -Headers @{ "Zotero-API-Version" = "3" } -UseBasicParsing
```

### curl (cross-platform)
```bash
curl -s -H "Zotero-API-Version: 3" "http://127.0.0.1:23119/api/users/0/collections"
```

If the connection fails with "connection closed":
- The local API preference is not yet enabled
- Close and reopen Zotero after checking the preference
- Or manually add to `prefs.js`: `user_pref("extensions.zotero.httpServer.localAPI.enabled", true);`

## Direct HTTP API Usage

When the Python helper script is unavailable or fails, use direct HTTP calls to the Zotero local API (port 23119).

### List collections (folders)
```bash
curl -s -H "Zotero-API-Version: 3" "http://127.0.0.1:23119/api/users/0/collections"
```
Response includes `key`, `name`, and `meta.numItems` for each collection.

### List items in a collection
```bash
# Top-level items only
curl -s -H "Zotero-API-Version: 3" "http://127.0.0.1:23119/api/users/0/collections/{collectionKey}/items/top?limit=25"

# All items (including attachments)
curl -s -H "Zotero-API-Version: 3" "http://127.0.0.1:23119/api/users/0/collections/{collectionKey}/items?limit=25"
```

### Search across library
```bash
curl -s -H "Zotero-API-Version: 3" "http://127.0.0.1:23119/api/users/0/items?q=transformer&limit=10"
```

### Export BibTeX
```bash
curl -s -H "Zotero-API-Version: 3" "http://127.0.0.1:23119/api/users/0/items?format=bibtex&limit=100"
```

### Get item children (attachments)
```bash
curl -s -H "Zotero-API-Version: 3" "http://127.0.0.1:23119/api/users/0/items/{itemKey}/children"
```

### Get attachment file URL
```bash
curl -s -H "Zotero-API-Version: 3" "http://127.0.0.1:23119/api/users/0/items/{attachmentKey}/file/view/url"
```

## Reading PDFs from Zotero

Zotero stores PDFs in its local storage. To read PDF content:

### 1. Find the PDF local path

Zotero stores attachments under `%USERPROFILE%\Zotero\storage\{attachmentKey}\`.

Get the attachment key from the item's children:
```bash
curl -s -H "Zotero-API-Version: 3" "http://127.0.0.1:23119/api/users/0/items/{itemKey}/children"
```

The response includes `links.attachment.href` and `data.filename`.

### 2. Locate the PDF file

The PDF is typically at:
```
%USERPROFILE%\Zotero\storage\{attachmentKey}\{filename}.pdf
```

Example:
```powershell
Get-ChildItem "$env:USERPROFILE\Zotero\storage\HQV9AFD4"
```

### 3. Read PDF content with pdfplumber

Install pdfplumber (requires Python):
```bash
python3 -m pip install pdfplumber
```

Extract text:
```python
import pdfplumber
pdf_path = r"C:\Users\{user}\Zotero\storage\{attachmentKey}\{filename}.pdf"
with pdfplumber.open(pdf_path) as pdf:
    for i, page in enumerate(pdf.pages[:5]):  # first 5 pages
        text = page.extract_text()
        print(f"--- Page {i+1} ---")
        print(text[:2000] if text else "(no text)")
```

### 4. Alternative: Use Codex bundled Python

If system Python lacks pip, use Codex's bundled Python:
```powershell
$python = "C:\Users\{user}\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe"
& $python -m pip install pdfplumber
& $python -c "import pdfplumber; ..."
```

## Python Helper Script

When available, the stdlib-only helper script provides convenience:

```bash
python3 <skill-root>/scripts/zotero.py <command>
```

### Fast starts

Check readiness:
```bash
python3 scripts/zotero.py status --json
```

Enable local API and restart Zotero:
```bash
python3 scripts/zotero.py enable --restart
```

Search and export:
```bash
python3 scripts/zotero.py search "transformer" --json
python3 scripts/zotero.py export-bibtex --out references.bib
```

## Workflow

1. **Verify Zotero is running** and local API is enabled.
2. **Choose connection method**:
   - Prefer direct HTTP API calls when the Python helper is unavailable
   - Use the helper script when available for convenience
3. **Read-only operations** (safe by default):
   - List collections/items via HTTP API
   - Search library
   - Export BibTeX
   - Read PDFs from local storage
4. **Attachment/PDF access**: Only retrieve when explicitly requested
5. **Write operations**: Confirm with user before import/save actions

## Common HTTP API Patterns

### Summarize a collection
```bash
# 1. Get collection key
curl -s -H "Zotero-API-Version: 3" "http://127.0.0.1:23119/api/users/0/collections"

# 2. Get items in collection (extract title, abstract, creators)
curl -s -H "Zotero-API-Version: 3" "http://127.0.0.1:23119/api/users/0/collections/{key}/items/top?limit=50"

# 3. For each item, get children to find PDF attachment key
curl -s -H "Zotero-API-Version: 3" "http://127.0.0.1:23119/api/users/0/items/{itemKey}/children"

# 4. Read PDF from %USERPROFILE%\Zotero\storage\{attachmentKey}\
```

### Export collection to BibTeX
```bash
curl -s -H "Zotero-API-Version: 3" "http://127.0.0.1:23119/api/users/0/collections/{key}/items?format=bibtex" > collection.bib
```

## Output standards

- For inventory/search, return title, creators, year, Zotero item key, and BibTeX key when available.
- For PDF reading, summarize key findings: research question, method, results, and conclusions.
- For `.bib` export, return the absolute output path and entry count.
- For blockers, name the exact gate: Zotero app missing, local API disabled, port closed, no matching item, or PDF not found.

## Route details

Read `references/local-api-routes.md` for complete endpoint documentation.
