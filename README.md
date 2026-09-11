# Expense Tracker - JavaScript PaddleOCR

This `ocrversion` branch uses the JavaScript version of PaddleOCR to scan bills directly in the browser.

## OCR approach

The Scan Bill feature loads PaddleOCR dynamically from jsDelivr:

```js
const { PaddleOCR } = await import(
  'https://cdn.jsdelivr.net/npm/@paddleocr/paddleocr-js/+esm'
);

ocrEngine = await PaddleOCR.create({
  lang: 'en',
  ocrVersion: 'PP-OCRv5',
  ortOptions: { backend: 'auto' }
});
```

The OCR model runs locally in the browser after the JavaScript package and model have been downloaded. No Python server or Flask endpoint is required.

## Scan Bill flow

1. Select or capture a bill using the **Scan Bill** button.
2. The browser receives the image as a `File` object.
3. PaddleOCR recognizes text in the image.
4. The app extracts the recognized text from the OCR result.
5. `parseBillText()` looks for likely total labels such as `Total`, `Amount Due`, and `Amount Paid`.
6. The app guesses a description from the first meaningful receipt lines.
7. The amount, description, and raw OCR text are placed into the expense form.
8. Review the values and choose **Save entry** to store the expense.

The scan does not save an expense automatically. The detected amount is only a suggestion and must be checked by the user.

## Files

- `index.html` - local/offline expense tracker using browser PaddleOCR.
- `index-cloud.html` - cloud-sync variant using browser PaddleOCR.
- `sw.js` - service worker for the local app.
- `sw-cloud.js` - service worker for the cloud-sync app.

## Running the app

The app is a static web application. It can be opened through a static web server or hosted as a static site. A server is recommended because browser security rules can restrict modules, service workers, or file access when an HTML file is opened directly.

For a quick local server, run from this directory:

```bash
python3 -m http.server 8080
```

Then open:

```text
http://127.0.0.1:8080/index.html
```

## Requirements

- A modern browser with ES module and WebAssembly support.
- Internet access on the first scan so the PaddleOCR JavaScript package and model can be downloaded.
- A clear, well-lit bill image for best results.
- An item configured as a flat-amount item, such as `Bill` or `Receipt`, so the scanned total can be entered.

After the OCR assets are cached, browser behavior may depend on the browser cache and service-worker state. Clearing the site cache or unregistering the service worker can force fresh assets to load.

## Accuracy and limitations

PP-OCRv5 recognizes text; it does not understand the expense meaning by itself. The app uses JavaScript heuristics to infer the final amount from the recognized text. This can be wrong when a document contains several amounts, tables, phone numbers, PINs, taxes, or unclear text.

The browser implementation is convenient and private because the image is processed on the user device. It also avoids a Python backend, but performance and memory use depend on the device and browser. Complex tables and structured documents may be better handled by a server-side `PPStructureV3` pipeline.

Always verify the suggested amount and description before saving.

How to use this tracker - Default Offline Tracker
1. Set up your items
Tap 🏷️ Items. Add a category (e.g. Groceries), then add items under it. Each item is either priced (quantity × unit price, like Milk) or a flat amount (like an Electricity Bill).

2. Log an entry
Switch to Day view, pick a date from the strip, and tap + Add entry. Choose category → item → enter quantity or amount.

3. Repeating expenses
Check 🔁 Repeat this every month when adding an entry — it will auto-appear on the same date every month afterward, so you never have to re-add it. Manage or remove repeating entries from the Items screen.

4. Review your month
Switch to Month view for a calendar overview — each day shows its total and entry count. The bottom of the page shows your total spend and a category breakdown.

5. Export & share
Export CSV — opens cleanly in Excel/Sheets
Share — sends a quick summary via WhatsApp or your phone's share sheet
Export/Import JSON — full backup, or move data between devices
About this version
This is the offline version — everything stays on this device only. Use Export JSON and Import JSON to move data to another device.

How to use this tracker - Cloud Version 
1. Set up your items
Tap 🏷️ Items. Add a category (e.g. Groceries), then add items under it. Each item is either priced (quantity × unit price, like Milk) or a flat amount (like an Electricity Bill).

2. Log an entry
Switch to Day view, pick a date from the strip, and tap + Add entry. Choose category → item → enter quantity or amount.

3. Repeating expenses
Check 🔁 Repeat this every month when adding an entry — it will auto-appear on the same date every month afterward, so you never have to re-add it. Manage or remove repeating entries from the Items screen.

4. Review your month
Switch to Month view for a calendar overview — each day shows its total and entry count. The bottom of the page shows your total spend and a category breakdown.

5. Export & share
Export CSV — opens cleanly in Excel/Sheets
Share — sends a quick summary via WhatsApp or your phone's share sheet
Export/Import JSON — full backup, or move data between devices
Sharing with others
Tap Workspace and set a shared code — everyone who enters the exact same code sees and edits the same data, live.
