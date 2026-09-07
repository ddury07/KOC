# Knights of the Chair Application Form
## Touch signatures · PDF · Email · Google Drive · Spreadsheet link

The form now:
- Collects **drawn signatures** (finger / stylus) from applicant and parent
- Builds a **signed PDF** of the application
- Sends the PDF to your **Google Apps Script**
- The script **saves the PDF to Google Drive**, **emails it to the applicant**, and **writes a row** in your spreadsheet that includes a link to the PDF

---

### 1. Upload the form
- Use `knights-of-the-chair-application.html` (or rename to `index.html` for GitHub Pages)
- Push/upload it to your site as before

### 2. Create the Google Spreadsheet

1. Create a new Google Sheet named e.g. **“Knights of the Chair Applications”**
2. Put these headers in **Row 1** (copy-paste):

```
Timestamp	Last Name	First Name	Middle Name	Street Address	City	State	Zip	Phone	Email	Grade	Date of Birth	Baptism Year	Convert	Conversion Year	Confirmed	Altar Server	Parishes Served	Trained Roles	Why Join	Applicant Name	Applicant Date	Parent Name	Parent Date	PDF Link
```

### 3. Create a Drive folder for the PDFs

1. In Google Drive, create a folder (e.g. **“KOC Signed Applications”**)
2. Open the folder → copy the **Folder ID** from the URL  
   (`https://drive.google.com/drive/folders/THIS_IS_THE_FOLDER_ID`)

### 4. Google Apps Script (replace the old one)

1. In the spreadsheet: **Extensions → Apps Script**
2. Delete any existing code and paste **all** of the following:

```javascript
// ========== CONFIG ==========
// Paste your Drive folder ID here (from step 3)
const DRIVE_FOLDER_ID = "PASTE_YOUR_DRIVE_FOLDER_ID_HERE";

// Optional: also email a copy to the director
const ADMIN_EMAIL = ""; // e.g. "director@example.com" or leave blank

function doPost(e) {
  try {
    var data = JSON.parse(e.postData.contents);
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();

    // --- Save PDF to Drive ---
    var pdfBlob = Utilities.newBlob(
      Utilities.base64Decode(data.pdfBase64),
      "application/pdf",
      data.pdfFileName || ("KOC-Application_" + Date.now() + ".pdf")
    );
    var folder = DriveApp.getFolderById(DRIVE_FOLDER_ID);
    var file = folder.createFile(pdfBlob);
    file.setSharing(DriveApp.Access.ANYONE_WITH_LINK, DriveApp.Permission.VIEW);
    var pdfUrl = file.getUrl();

    // --- Email PDF to applicant ---
    if (data.email) {
      MailApp.sendEmail({
        to: data.email,
        subject: "Knights of the Chair – Application Received",
        body:
          "Dear " + (data.firstName || "Applicant") + ",\n\n" +
          "Thank you for applying to the Knights of the Chair.\n\n" +
          "A copy of your signed application is attached for your records.\n\n" +
          "Someone from the Knights of the Chair will be in touch.\n\n" +
          "Saint Joseph Cathedral\nDiocese of Columbus",
        attachments: [pdfBlob]
      });
    }

    // Optional admin copy
    if (ADMIN_EMAIL) {
      MailApp.sendEmail({
        to: ADMIN_EMAIL,
        subject: "New Knights of the Chair Application – " + (data.lastName || "") + ", " + (data.firstName || ""),
        body: "A new application has been submitted.\n\nPDF: " + pdfUrl,
        attachments: [pdfBlob]
      });
    }

    // --- Append row to spreadsheet ---
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
      data.parentDate || "",
      pdfUrl
    ]);

    return ContentService
      .createTextOutput(JSON.stringify({ result: "success", pdfUrl: pdfUrl }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({ result: "error", error: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

3. Replace `PASTE_YOUR_DRIVE_FOLDER_ID_HERE` with your real folder ID.
4. (Optional) Put an admin email in `ADMIN_EMAIL` if you want a copy of every application.
5. **Save** the project.
6. **Deploy → New deployment → Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
7. Authorize when prompted (you will need to allow Drive + Gmail/Mail permissions).
8. Copy the **Web app URL**.

### 5. Put the Web App URL into the HTML form

Open `knights-of-the-chair-application.html` (or `index.html`) and find:

```javascript
const SCRIPT_URL = "PASTE_YOUR_GOOGLE_APPS_SCRIPT_WEB_APP_URL_HERE";
```

Replace with your Web app URL, then re-upload / push the file.

### 6. Test

1. Open the live form on a phone or tablet (or desktop with mouse).
2. Fill it out, draw both signatures, submit.
3. Check:
   - Spreadsheet has a new row with a **PDF Link**
   - The Drive folder contains the PDF
   - The applicant’s email received the PDF attachment

---

### Notes

- Signatures only work while the canvas is visible; the form validates that both pads have been signed before submitting.
- The PDF is generated in the browser (jsPDF) and sent as base64; the Apps Script never needs to “draw” the form.
- “Anyone with the link can view” is set on each PDF so the spreadsheet link works without extra sharing steps. You can tighten permissions later if desired.
- First deployment will ask for extra OAuth scopes (Drive + Mail). That is expected.
