# Setting up your own Google Sheet as the data store

This connects the form to a Google Sheet **you own**, so every submission is permanently saved
in your own spreadsheet — no dependency on Claude at all, and nothing "Anthropic" visible
anywhere in the page.

## Step 1 — Create the Sheet
1. Go to https://sheets.google.com and create a new blank spreadsheet.
2. Name it whatever you like, e.g. "ASC Instrument Register — Live Data".

## Step 2 — Add the backend script
1. In the Sheet, go to **Extensions → Apps Script**.
2. Delete anything in the editor and paste in the code below.
3. Click the **Save** icon (or Ctrl/Cmd+S).
4. In the function dropdown at the top, select **setupSheet**, then click **Run** (▶).
   - The first time, Google will ask you to authorize the script — click through
     "Advanced" → "Go to (project name)" → "Allow". This is normal for your own scripts.
   - This creates a "Submissions" tab in your sheet with all the correct column headers already filled in.

```javascript
function setupSheet() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheetByName('Submissions');
  if (!sheet) sheet = ss.insertSheet('Submissions');
  var headers = ["Sr No","Facility","Make","Model","Date of installation",
    "Managed by (division)","Category","Available equipment / mode of use",
    "Facility in-charge","Facility manager","Facility operator","Department",
    "Features & working principle","Specifications","Description","Sample preparation",
    "Applications","Chemicals allowed","Gases allowed","SOP document",
    "Training / policy documents","Other details","Publications",
    "Last Filled By","Last Updated"];
  sheet.getRange(1,1,1,headers.length).setValues([headers]);
  sheet.setFrozenRows(1);
}

function doGet(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Submissions');
  var data = sheet.getDataRange().getValues();
  var headers = data[0];
  var rows = data.slice(1).map(function(r){
    var obj = {};
    headers.forEach(function(h,i){ obj[h] = r[i]; });
    return obj;
  }).filter(function(o){ return o['Sr No'] !== '' && o['Sr No'] !== null; });
  return ContentService.createTextOutput(JSON.stringify(rows))
    .setMimeType(ContentService.MimeType.JSON);
}

function doPost(e) {
  var body = JSON.parse(e.postData.contents);
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Submissions');
  var data = sheet.getDataRange().getValues();
  var headers = data[0];
  var srNoCol = headers.indexOf('Sr No');

  var rowIndex = -1;
  for (var i = 1; i < data.length; i++) {
    if (String(data[i][srNoCol]) === String(body.srNo)) { rowIndex = i; break; }
  }

  var rowData = headers.map(function(h){
    if (h === 'Sr No') return body.srNo;
    if (h === 'Facility') return body.facility;
    if (h === 'Last Filled By') return body.filledBy || '';
    if (h === 'Last Updated') return body.updatedAt || '';
    if (body.fields && body.fields[h] !== undefined) return body.fields[h];
    return (rowIndex >= 0) ? data[rowIndex][headers.indexOf(h)] : '';
  });

  if (rowIndex >= 0) {
    sheet.getRange(rowIndex + 1, 1, 1, rowData.length).setValues([rowData]);
  } else {
    sheet.appendRow(rowData);
  }

  return ContentService.createTextOutput(JSON.stringify({status: 'ok'}))
    .setMimeType(ContentService.MimeType.JSON);
}
```

> Note: the header labels above must match exactly what's used in the HTML file's
> `FIELD_LABELS`, which they already do — don't rename them.

## Step 3 — Deploy as a Web App
1. Click **Deploy → New deployment**.
2. Click the gear icon next to "Select type" and choose **Web app**.
3. Set:
   - **Execute as:** Me
   - **Who has access:** Anyone
4. Click **Deploy**, authorize again if prompted.
5. Copy the **Web app URL** it gives you (looks like `https://script.google.com/macros/s/XXXXX/exec`).

## Step 4 — Plug the URL into the HTML file
1. Open `instrument_register.html` in a text editor.
2. Find this line near the top of the `<script>` section:
   ```javascript
   const API_URL = "PASTE_YOUR_GOOGLE_APPS_SCRIPT_URL_HERE";
   ```
3. Replace the placeholder with the URL you copied in Step 3.
4. Also find this line, a bit further down:
   ```javascript
   const ADMIN_CODE = "changeme123";
   ```
   Change `"changeme123"` to your own private code. This controls who can see the
   "Progress & export" tab (see the note below).
5. Save the file.

## Only you can see Progress & export
The "Progress & export" tab is hidden from the normal page — your 15+ people will only
ever see "Find your instrument". It only appears if the page is opened with `?admin=YOUR_CODE`
added to the URL, e.g.:
```
https://yourusername.github.io/repo-name/?admin=your-secret-code
```
Bookmark that exact link for yourself.

**Important honesty note:** this is a convenience gate, not real security. Since GitHub Pages
serves your raw HTML/JS file publicly, anyone who views the page source (or your GitHub repo)
could technically find the `ADMIN_CODE` value and construct the admin link themselves. It
keeps casual users from stumbling onto the dashboard, but it will not stop someone who
deliberately goes looking. If you need this to be genuinely private, you'd want a private
repo plus a host that supports real server-side authentication (e.g. a paid Netlify plan with
password protection) — happy to help set that up if it matters for your case.

## Every save now writes the complete record
Each time someone hits "Save my additions", the page sends **all** fields for that
instrument — both the already-known ones and the newly filled ones — not just the new
additions. So each row in your Sheet is always a complete, standalone record; you don't
need to merge it with the original JSON/Excel afterward.

## Step 5 — Host the file
See `HOSTING_INSTRUCTIONS.md` for the easiest free hosting options (no Anthropic branding,
works for all your 15+ people).

## Where your data lives now
Every submission is a row in the **Submissions** tab of your own Google Sheet — visible,
editable, and exportable by you at any time (**File → Download → Microsoft Excel (.xlsx)**
directly from Google Sheets). The HTML page itself never stores anything; it just reads and
writes to your Sheet through the script above.

## A caution about scale
Apps Script re-scans the whole sheet on every save to find the matching row, which is fine
for ~175 rows and a few dozen people, but if this ever needs to scale to thousands of rows or
very heavy concurrent traffic, a proper database (Firebase, Airtable, etc.) would hold up
better. For your case, this will work reliably.
