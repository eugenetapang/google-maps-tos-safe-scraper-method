# Google Maps Results Data Collector - TOS Safe Scraping Method | Human-In-The-Loop-Sucka

Excellent question. This is a perfect use case for Google Workspace's built-in tools. We can create a "semi-automated" system that dramatically speeds up the manual collection process without violating any terms of service, as you are still the one performing the search and identifying the data.

The goal is to eliminate the tedious task of switching tabs, clicking into specific cells, and manually formatting each entry. We will build a simple **Data Entry Sidebar** directly within Google Sheets using Google Apps Script.

Here is the concept and the step-by-step guide.

### The Concept: A "Heads-Up Display" for Data Entry

1.  You will have two windows side-by-side: Google Maps in one, and your Google Sheet in the other.
2.  Inside the Google Sheet, we will create a custom sidebar with fields for "Name," "Address," "Phone," etc.
3.  Your workflow will be:
    *   Look at a business listing in Google Maps.
    *   Copy a piece of data (like the address).
    *   Paste it into the corresponding field in the sidebar.
    *   Repeat for the phone, website, etc.
    *   Click a single "Submit" button in the sidebar.
4.  The script will then automatically take all the data from the sidebar, add it to the **next available row** in your sheet, add a timestamp, and clear the form for the next entry.

This transforms a multi-click, error-prone process into a streamlined "copy, paste, submit" workflow.

---

### Step-by-Step Implementation Guide

#### Step 1: Set Up Your Google Sheet

1.  Create a new Google Sheet. Name it something like "Houston Paving Contractors Data".
2.  In the first sheet (you can rename it to "Listings"), set up your column headers in the first row:
    *   `A1`: `Name`
    *   `B1`: `Address`
    *   `C1`: `Phone`
    *   `D1`: `Website`
    *   `E1`: `Notes`
    *   `F1`: `Timestamp`



#### Step 2: Open the Apps Script Editor

1.  In your Google Sheet, go to the menu and click **Extensions > Apps Script**.
2.  This will open a new browser tab with the Apps Script editor. It will have a default file named `Code.gs`.

#### Step 3: Write the Backend Code (`Code.gs`)

Delete any existing code in the `Code.gs` file and replace it with the following. This script contains the logic to receive data from the sidebar and append it to the sheet.

```javascript
// This function runs when the spreadsheet is opened. It creates a custom menu.
function onOpen() {
  SpreadsheetApp.getUi()
      .createMenu('Data Collector')
      .addItem('Open Entry Form', 'showSidebar')
      .addToUi();
}

// This function creates and shows the sidebar.
function showSidebar() {
  var html = HtmlService.createHtmlOutputFromFile('Sidebar')
      .setTitle('Data Entry Form');
  SpreadsheetApp.getUi().showSidebar(html);
}

// This is the function that the sidebar will call to add data to the sheet.
// It takes the data from the form as arguments.
function appendDataToSheet(name, address, phone, website, notes) {
  // Get the active sheet where data will be stored.
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("Listings");
  
  // Get the current date and time for the timestamp.
  var timestamp = new Date();
  
  // Append a new row with all the data.
  sheet.appendRow([name, address, phone, website, notes, timestamp]);
}
```

#### Step 4: Create the Sidebar User Interface (`Sidebar.html`)

1.  In the Apps Script editor, click the **`+`** icon next to "Files" and choose **HTML**.
2.  Name the new file **`Sidebar`** (case-sensitive) and press Enter.
3.  Delete the default content in the `Sidebar.html` file and replace it with the code below. This is the form you will see and interact with.

```html
<!DOCTYPE html>
<html>
  <head>
    <base target="_top">
    <!-- Using a simple stylesheet for a clean look -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
    <style>
      .container { padding: 15px; }
      label { font-weight: bold; }
      .form-group { margin-bottom: 10px; }
      button { width: 100%; }
    </style>
  </head>
  <body>
    <div class="container">
      <div class="form-group">
        <label for="name">Business Name</label>
        <input type="text" class="form-control" id="name">
      </div>
      <div class="form-group">
        <label for="address">Address</label>
        <input type="text" class="form-control" id="address">
      </div>
      <div class="form-group">
        <label for="phone">Phone Number</label>
        <input type="text" class="form-control" id="phone">
      </div>
      <div class="form-group">
        <label for="website">Website</label>
        <input type="text" class="form-control" id="website">
      </div>
      <div class="form-group">
        <label for="notes">Notes</label>
        <textarea class="form-control" id="notes" rows="3"></textarea>
      </div>
      <button class="btn btn-primary" onclick="submitData()">Add to Sheet</button>
    </div>

    <script>
      // This function runs when the "Add to Sheet" button is clicked.
      function submitData() {
        // Get all the values from the form fields.
        var name = document.getElementById('name').value;
        var address = document.getElementById('address').value;
        var phone = document.getElementById('phone').value;
        var website = document.getElementById('website').value;
        var notes = document.getElementById('notes').value;

        // Disable the button to prevent double-clicks
        document.querySelector('button').disabled = true;
        document.querySelector('button').textContent = 'Adding...';

        // Call the backend Apps Script function with the data.
        google.script.run
          .withSuccessHandler(clearForm) // Run clearForm() on success
          .appendDataToSheet(name, address, phone, website, notes);
      }

      // This function is called after the data is successfully added.
      function clearForm() {
        // Clear all the form fields.
        document.getElementById('name').value = '';
        document.getElementById('address').value = '';
        document.getElementById('phone').value = '';
        document.getElementById('website').value = '';
        document.getElementById('notes').value = '';
        
        // Re-enable the button
        document.querySelector('button').disabled = false;
        document.querySelector('button').textContent = 'Add to Sheet';
      }
    </script>
  </body>
</html>
```

4.  **Save both files** by clicking the floppy disk icon (Save project).

---

### How to Use Your New System

1.  **Reload your Google Sheet.** After reloading, you should see a new menu item called **"Data Collector"**.
2.  Click **Data Collector > Open Entry Form**.
3.  The first time you do this, Google will ask for **authorization**.
    *   Click "Continue".
    *   Choose your Google account.
    *   You may see a "Google hasn't verified this app" warning. This is normal for your own scripts. Click **"Advanced"** and then **"Go to [Your Project Name] (unsafe)"**.
    *   Click "Allow" to grant the script permission to edit your spreadsheet.
4.  The sidebar will now appear on the right side of your sheet.
5.  Arrange your windows: Google Maps on the left, your Google Sheet with the sidebar on the right.
6.  **Start collecting!** Copy and paste the information from a Google Maps listing into the sidebar fields and click "Add to Sheet". The data will appear in the sheet, and the form will clear, ready for the next entry.



This semi-automated approach provides the best of both worlds: it's fast and efficient, ensures consistent data formatting, and fully complies with the terms of service because a human is still at the helm.
