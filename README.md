# PIM-PAM Document Translation Pipeline — WBG Translation API

## Overview

This notebook translates PDF (and other Office) documents using the
**World Bank Group Document Translation API** (`POST /api/Translate/File`).
Unlike the Mistral pipeline, which extracts raw text and sends it to an LLM,
this API accepts the **original binary file** and returns a fully translated
document — preserving layout, tables, headers, and formatting.

### How the API works

| Input | Output |
|---|---|
| `.pdf` | `.pdf` (translated) |
| `.doc`, `.odt`, `.rtf` | `.docx` |
| `.xls`, `.ods` | `.xlsx` |
| `.ppt`, `.odp` | `.pptx` |

### Workflow

1. **Credential setup** — Fetches an OAuth 2.0 bearer token from Azure AD
   using a service principal stored in Databricks Secrets.
2. **Single-file translation** — Uploads a file via `multipart/form-data` to
   the WBG Translation API and saves the translated binary response.
3. **Bulk translation** — Iterates over all supported files in a directory,
   skipping already-translated outputs for resumability.
4. **Audit log** — Prints a structured summary of successes, skips, and failures.

### Key API parameters

| Parameter | Type | Description |
|---|---|---|
| `ToLanguage` | string | Target language code (e.g. `en`, `fr`, `es`) |
| `FromLanguage` | string | Source language code (e.g. `ko`, `ar`, `fr`) |
| `File` | binary | The document file to translate |
| `SourceStorageURI` | string | *(Optional)* Azure Blob source container URL |
| `TargetStorageURI` | string | *(Optional)* Azure Blob target container URL |
| `CategoryOverride` | string | *(Optional)* Custom translator category (e.g. ICSID model) |

### Authentication

Azure AD service principal via `azure-identity`.
Secrets are stored in Databricks Secret scope `DAPGPTKEYVAULT`.

| Secret key | Description |
|---|---|
| `WBG-Translate-Tenant-ID` | Azure AD tenant ID |
| `WBG-Translate-Client-ID` | Service principal client ID |
| `WBG-Translate-Client-Secret` | Service principal client secret |

> **Note:** The Client ID and Scope for the Dev environment are hardcoded
> below as constants since they are non-sensitive identifiers, not secrets.
 