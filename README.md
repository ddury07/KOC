# KOC
knights of the chair sign up website


# Knights of the Chair Application Form
## How to put the form on your website and send responses to a Google Spreadsheet

### 1. Upload the form to your website
- Take the file `knights-of-the-chair-application.html`
- Upload it to your web server (or GitHub Pages, Netlify, Cloudflare Pages, etc.)
- You can rename it to `index.html` if you want it to be the main page of a folder.

### 2. Create the Google Spreadsheet (one-time setup)

1. Go to [sheets.google.com](https://sheets.google.com) and create a **new blank spreadsheet**.
2. Name it something like **“Knights of the Chair Applications”**.
3. In **Row 1** put these exact column headers (copy-paste):

```
Timestamp	Last Name	First Name	Middle Name	Street Address	City	State	Zip	Phone	Email	Grade	Date of Birth	Baptism Year	Convert	Conversion Year	Confirmed	Altar Server	Parishes Served	Trained Roles	Why Join	Applicant Signature	Applicant Date	Parent Signature	Parent Date
```

4. You can freeze the first row (View → Freeze → 1 row) so the headers stay visible.

### 3. Create the Google Apps Script that receives the form data

1. In the same Google Spreadsheet, go to **Extensions → Apps Script**.
2. Delete any code that is already there and paste the following:

```javascript
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var data = JSON.parse(e.postData.contents);

    sheet.appendRow([
      data.timestamp || new Date().toISOString(),
      data.lastName || "",
      data.firstName || "",
      data.middleName || "",
      data.streetAddress || "",
      data.city || "",
      data.state || "",
      data.zip || "",
      data.phone || "",
      data.email || "",
      data.grade || "",
      data.dob || "",
      data.baptismYear || "",
      data.convert || "",
      data.conversionYear || "",
      data.confirmed || "",
      data.altarServer || "",
      data.parishes || "",
      data.trained || "",
      data.whyJoin || "",
      data.applicantName || "",
      data.applicantDate || "",
      data.parentName || "",
      data.parentDate || ""
    ]);

    return ContentService
      .createTextOutput(JSON.stringify({ result: "success" }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({ result: "error", error: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

3. Click the **Save** icon (or Ctrl+S). Give the project a name such as “Knights Form Receiver”.
4. Click **Deploy → New deployment**.
5. Click the gear icon next to “Select type” and choose **Web app**.
6. Settings:
   - Description: `Knights Application Form`
   - Execute as: **Me**
   - Who has access: **Anyone**
7. Click **Deploy**.
8. Authorize the script when prompted (you may need to click “Advanced” → “Go to … (unsafe)” because it is your own script).
9. After deployment you will see a **Web app URL** that looks like:
   `https://script.google.com/macros/s/AKfycb.../exec`
10. **Copy that entire URL**.

### 4. Put the URL into the HTML form

1. Open `knights-of-the-chair-application.html` in a text editor.
2. Find this line near the bottom (inside the `<script>` tag):

```javascript
const SCRIPT_URL = "PASTE_YOUR_GOOGLE_APPS_SCRIPT_WEB_APP_URL_HERE";
```

3. Replace the placeholder with the Web app URL you just copied, for example:

```javascript
const SCRIPT_URL = "https://script.google.com/macros/s/AKfycbXXXXXXXXXXXXXXXX/exec";
```

4. Save the file and re-upload it to your website.

### 5. Test it

1. Open the form on your website.
2. Fill out a test application and click **Submit Application**.
3. Go back to your Google Spreadsheet – a new row should appear within a few seconds.

That’s it! Every future submission will automatically appear as a new row in the spreadsheet.

---

### Optional improvements

- **Email notification**: In the Apps Script you can add a line that sends you an email when a new application arrives (using `MailApp.sendEmail`).
- **Confirmation email to the applican
... 