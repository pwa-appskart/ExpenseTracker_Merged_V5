# Expense Tracker - Python PaddleOCR Backend

This `ocrversionwithPython` branch uses a local Python Flask backend to process bill images with PaddleOCR's `PPStructureV3` pipeline. The browser selects and uploads the image, while Python performs document OCR and layout analysis on the laptop.

## Architecture

```text
Browser Scan Bill button
        |
        | multipart/form-data upload: file
        v
Flask: POST http://127.0.0.1:5000/ocr
        |
        v
PPStructureV3 / PP-OCRv5 / English
        |
        v
OCR text + layout blocks + amount suggestion
        |
        v
Browser expense form for review
```

The image is sent only to the local machine at `127.0.0.1`. It is not sent to a hosted OCR service. The Flask server must be running while the browser app is used.

## Components

- `index.html` - local expense tracker using the Python OCR endpoint.
- `index-cloud.html` - cloud-sync tracker using the same local OCR endpoint.
- `ocr/server.py` - Flask API and `PPStructureV3` integration. This directory is a sibling of the tracker repository in the workspace.
- `ocr/requirements.txt` - Python dependencies for the OCR backend.

## Setup

The backend uses the existing Python virtual environment at `ocr/venv`.

From `/Users/sreenivasareddysaragada/Git`:

```bash
ocr/venv/bin/python -m pip install -r ocr/requirements.txt
```

The first model initialization can take time. PaddleOCR downloads or loads its cached models, including the document orientation, layout, text, table, and formula models used by `PPStructureV3`.

## Start the backend

From `/Users/sreenivasareddysaragada/Git`:

```bash
ocr/venv/bin/python ocr/server.py
```

Keep this terminal running. A successful startup shows:

```text
Running on http://127.0.0.1:5000
```

The server is a local development server intended for this laptop. It is not configured for production deployment or remote access.

## API

### Health check

```http
GET http://127.0.0.1:5000/health
```

Example response:

```json
{
  "ok": true,
  "model": "PPStructureV3 / PP-OCRv5 / en"
}
```

### OCR upload

```http
POST http://127.0.0.1:5000/ocr
Content-Type: multipart/form-data
```

The image must be provided in a multipart field named `file`.

Example command:

```bash
curl -X POST \
  -F "file=@ocr/Screenshot 2026-09-06 at 11.37.46 AM.png" \
  http://127.0.0.1:5000/ocr
```

Successful responses contain:

```json
{
  "rawText": "recognized document text",
  "totalAmount": 123.45,
  "descriptionGuess": "vendor or document description",
  "blocks": 2,
  "model": "PPStructureV3 / PP-OCRv5 / en"
}
```

`totalAmount` and `descriptionGuess` are best-effort suggestions. The browser always places them into the editable expense form for user confirmation; the backend does not save expenses.

## Browser scan flow

1. Click **Scan Bill**.
2. Select or capture an image.
3. The browser creates a `FormData` object and appends the image as `file`.
4. JavaScript sends a `POST` request to `http://127.0.0.1:5000/ocr`.
5. Flask saves the upload temporarily and passes it to `PPStructureV3`.
6. The backend extracts text from structured layout results and applies amount/description heuristics.
7. JSON is returned to the browser.
8. The browser pre-fills a flat-amount expense entry with the OCR result.
9. Review the values and select **Save entry**.

A successful browser scan produces this Flask log:

```text
"POST /ocr HTTP/1.1" 200 -
```

## Running the browser app

The tracker is a static web application. From `/Users/sreenivasareddysaragada/Git/ExpenseTracker_Merged_V5`, run:

```bash
python3 -m http.server 8080
```

Then open:

```text
http://127.0.0.1:8080/index.html
```

The cloud-sync variant is available at:

```text
http://127.0.0.1:8080/index-cloud.html
```

The browser and Flask server must run on the same laptop. A page opened on another device cannot reach `127.0.0.1` on this Mac.

## Requirements

- macOS laptop running the Flask server.
- Python virtual environment at `ocr/venv`.
- Flask and PaddleOCR dependencies installed.
- A modern browser.
- A clear, well-lit bill image.
- A configured flat-amount item such as `Bill` or `Receipt`.

## Accuracy and limitations

`PPStructureV3` combines document preprocessing, orientation handling, layout detection, text recognition, and optional table/formula processing. It is more suitable for structured documents than plain browser text OCR.

The backend still uses heuristics to select a likely amount from the recognized text. A receipt can contain subtotal, tax, discount, payment, phone, and reference numbers, so the suggested amount may be wrong. Always verify the amount and description before saving.

PaddleOCR uses native runtime components. Requests are serialized in `server.py` and Flask threading is disabled to reduce concurrency-related native runtime crashes. If the Python process exits with code `139`, that indicates a native segmentation fault; restart the server and inspect the Paddle/PaddleX environment rather than treating it as an HTTP or browser error.
