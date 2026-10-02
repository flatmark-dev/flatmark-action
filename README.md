# flatmark API — GitHub Action

Document to Markdown API and MCP server for PDF, Word, PowerPoint, Excel and HTML. OCR queue for large files. Hosted in Germany.

flatmark converts PDF, Word, PowerPoint, Excel and HTML to Markdown over a REST API and an MCP server. Files up to 8 MB convert in one call with MarkItDown. Files up to 25 MB and 200 pages go through a queue that runs Docling with OCR and table detection. The queue returns Markdown and a JSON structure file, by polling or a signed webhook. The servers are in Germany. The free plan has 100 credits a month and needs no card.

Calls the [flatmark API](https://flatmark.dev) from a workflow: every operation, one step each. Jobs are waited for and their result is downloaded.

## Get an API key

[Create a key](https://flatmark.dev/go/gh-action?to=/app/api-keys) and store it as the repository secret `FLATMARK_API_KEY`. The key is optional: without `api-key` the action runs on the anonymous tier, with lower limits.

## Usage

```yaml
on: push
jobs:
  flatmark:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7.0.1
      - uses: flatmark-dev/flatmark-action@v1.0.0
        with:
          operation: convert_document
          file: sample.pdf
          api-key: ${{ secrets.FLATMARK_API_KEY }}
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `operation` | yes |  | The API operation to call: get_me, get_job, list_jobs, convert_document, submit_conversion_job, get_conversion_result. |
| `file` | no |  | Path of the file to upload, for operations that take one. |
| `json` | no |  | The request body as a JSON object, for operations that take one. |
| `query` | no |  | Path, query and form parameters, one key=value per line; repeat a key for a list. |
| `api-key` | no |  | Your flatmark API key, from a secret. Optional: without one the anonymous tier's lower limits apply; a key raises them. Create one at https://flatmark.dev/go/gh-action?to=/app/api-keys |
| `output` | no |  | Where to write the answer (default: a file in the runner's temp directory, named after the operation). |
| `fail-if` | no |  | A jq expression on a JSON answer; the step fails when it is true (e.g. `.valid == false`). |
| `base-url` | no | `https://api.flatmark.dev` | The API base URL. |

## Outputs

| Output | Description |
|---|---|
| `status` | The HTTP status of the last API call. |
| `output` | The path of the file holding the answer. |

## Operations

### `get_me`

Your plan, remaining requests, and remaining credits — `GET /v1/me`.

```yaml
- uses: flatmark-dev/flatmark-action@v1.0.0
  with:
    operation: get_me
    api-key: ${{ secrets.FLATMARK_API_KEY }}
```

### `get_job`

Get one queued job by ID — `GET /v1/jobs/{job_id}`.

```yaml
- uses: flatmark-dev/flatmark-action@v1.0.0
  with:
    operation: get_job
    query: |
      job_id=<job_id>
    api-key: ${{ secrets.FLATMARK_API_KEY }}
```

### `list_jobs`

List your queued jobs, newest first — `GET /v1/jobs`.

```yaml
- uses: flatmark-dev/flatmark-action@v1.0.0
  with:
    operation: list_jobs
    api-key: ${{ secrets.FLATMARK_API_KEY }}
```

### `convert_document`

Convert a document to Markdown in one call — `POST /v1/convert`.
Costs 1 credit.

```yaml
- uses: flatmark-dev/flatmark-action@v1.0.0
  with:
    operation: convert_document
    file: sample.pdf
    api-key: ${{ secrets.FLATMARK_API_KEY }}
```

### `submit_conversion_job`

Queue a document for conversion with OCR and tables — `POST /v1/convert/jobs`.
Submits a job, polls `get_job` until it is done and downloads `get_conversion_result`.
Costs 10 credits.

```yaml
- uses: flatmark-dev/flatmark-action@v1.0.0
  with:
    operation: submit_conversion_job
    file: sample.pdf
    api-key: ${{ secrets.FLATMARK_API_KEY }}
```

### `get_conversion_result`

Download a queued job's Markdown or JSON — `GET /v1/convert/jobs/{job_id}/result`.

```yaml
- uses: flatmark-dev/flatmark-action@v1.0.0
  with:
    operation: get_conversion_result
    query: |
      job_id=<job_id>
    api-key: ${{ secrets.FLATMARK_API_KEY }}
```

API reference: https://flatmark.dev/docs · Base URL: `https://api.flatmark.dev`

## Support

https://flatmark.dev/support

This repository is generated from the live API; changes to its files are overwritten.
