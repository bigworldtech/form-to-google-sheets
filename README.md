## Google Apps Script for Spreadsheet Integration
This script is used to handle form submissions and store the data in a Google Spreadsheet. The script performs the following tasks:

Initial Setup: Set up the script properties with the active spreadsheet ID.
Handle POST Requests: When a POST request is received, the script locks the sheet, inserts a new row with the form data, and returns a success message.

## Code
var sheetName = 'Sheet1';
var scriptProp = PropertiesService.getScriptProperties();

function intialSetup() {
  var activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  scriptProp.setProperty('key', activeSpreadsheet.getId());
}

function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.tryLock(10000);

  try {
    var doc = SpreadsheetApp.openById(scriptProp.getProperty('key'));
    var sheet = doc.getSheetByName(sheetName);

    var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
    var nextRow = sheet.getLastRow() + 1;

    var newRow = headers.map(function(header) {
      return header === 'timestamp' ? new Date() : e.parameter[header];
    });

    sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow]);

    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'success', 'row': nextRow }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (e) {
    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'error', 'error': e }))
      .setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}

## Input your web app URL
## Web App URL Integration

To set up your form to submit data to Google Sheets, follow these steps:

1. **Input your web app URL**: Open the file named `index.html`.
2. **Replace the script URL**: On line 12, replace `<SCRIPT URL>` with your script URL:

   ```html
   <form name="submit-to-google-sheet">
     <input name="email" type="email" placeholder="Email" required>
     <button type="submit">Send</button>
   </form>

   <script>
     const scriptURL = '<SCRIPT URL>'
     const form = document.forms['submit-to-google-sheet']

     form.addEventListener('submit', e => {
       e.preventDefault()
       fetch(scriptURL, { method: 'POST', body: new FormData(form)})
         .then(response => console.log('Success!', response))
         .catch(error => console.error('Error!', error.message))
     })
   </script>
## Adding Additional Form Data

To capture additional data in your Google Sheet, follow these steps:

1. **Create new columns**: Open your Google Sheet and add new columns for the additional data you want to capture. Ensure the column titles match exactly with the `name` attributes from your form inputs.

2. **Update your form**: Modify your form in `index.html` to include additional input fields. For example, to capture first and last names, your form should look like this:

   ```html
   <form name="submit-to-google-sheet">
     <input name="email" type="email" placeholder="Email" required>
     <input name="firstName" type="text" placeholder="First Name">
     <input name="lastName" type="text" placeholder="Last Name">
     <button type="submit">Send</button>
   </form>
