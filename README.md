# Sender Score Domain Scraper 🤖📊

A hybrid automation tool that connects **Google Sheets** with a **local Python bot** to scrape "Sending Domains" from [SenderScore.org](https://senderscore.org/).

Designed for ease of use: users input an IP address in Google Sheets and click a custom menu button. The Python bot (running locally) picks up the job, scrapes the data (handling pagination and login), and pastes the results back into the sheet in real-time.

## 🌟 Features

* **Google Sheets "Frontend":** No need to touch code. Enter IP in the sheet, click "Run".
* **Live Status Updates:** The bot writes its status (e.g., *"Scraping Page 3... Found 24 items"*) directly into the Sheet cell so you know it's working.
* **Persistent Memory:** Uses a local Chrome profile (`chrome_profile`) to save your session. You only need to login/fill the CAPTCHA once; the bot remembers you forever.
* **Smart Pagination:** Automatically clicks "Next" and scrapes all pages until the data ends.
* **Anti-Duplicate Logic:** Prevents infinite loops if the website glitches.
* **Apps Script Integration:** Adds a professional "Sender Score Tool" menu to the Google Sheet toolbar.

---

## 🛠️ Prerequisites

* **Python 3.10+** installed on your machine.
* **Google Chrome** browser installed.
* A **Google Cloud Project** with the Drive and Sheets APIs enabled.

---

## 📦 Installation

1. **Clone or Download** this repository.
2. **Install Python Dependencies:**
Open your terminal in the project folder and run:
```bash
pip install gspread oauth2client selenium webdriver-manager pyperclip

```


3. **Setup Credentials:**
* Place your Google Service Account JSON key file in the project folder.
* Rename it to `credentials.json`.
* *Important:* Share your Google Sheet with the `client_email` found inside that JSON file (give it "Editor" access).



---

## ⚙️ Google Sheet Setup

1. **Create a new Google Sheet** named `Sender Score Tool`.
2. **Design the Interface:**
* **Cell B2:** This is where you type the Target IP. (Feel free to add a thick border!).
* **Cell C3:** This is the Status Bar (The bot writes updates here).
* **Row 6 (Header):** Write "Sending Domains" in A5. The data will be pasted starting at **A6**.


3. **Add the Menu Button (Apps Script):**
* In the Sheet, go to **Extensions > Apps Script**.
* Paste the code from `apps_script_code.js` (provided below).
* Save and Refresh the sheet. You should see a new menu: **Sender Score Tool**.



---

## 🚀 Usage

### 1. Start the Python Bot

Run the script on your computer. It needs to be running to "listen" for jobs.

```bash
python sender_score_live_update.py

```

*On the first run, it will open Chrome. Log in to SenderScore manually and solve any Captchas. Close Chrome. The bot has now saved your session.*

### 2. Control via Google Sheets

1. Open your **Sender Score Tool** Google Sheet.
2. Type an IP address into Cell **B2** (e.g., `1.2.3.4`).
3. In the menu bar, click **Sender Score Tool** > **▶ Run IP Scraper**.
4. Watch Cell **C3**! It will change from "Button Clicked..." to "Scraping Page 1..."
5. When finished, the domains will appear in **Column A**.

---

## 📂 Project Structure

```text
Sender_Score_Scraper/
│
├── sender_score_live_update.py  # MAIN SCRIPT: The Python bot
├── credentials.json             # YOUR KEY (Do not commit to GitHub!)
├── apps_script_code.js          # Copy this into Google Extensions > Apps Script
├── chrome_profile/              # (Auto-generated) Stores cookies/login session
├── requirements.txt             # List of python libraries
└── README.md                    # This file

```

---

## 📝 Appendix: Apps Script Code

*(Copy this into Extensions > Apps Script)*

```javascript
function onOpen() {
  const ui = SpreadsheetApp.getUi();
  ui.createMenu('Sender Score Tool')
      .addItem('▶ Run IP Scraper', 'runScraper')
      .addToUi();
}

function runScraper() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  // Try to find specific sheet, otherwise use active
  let sheet = ss.getSheetByName("Sender Score Tool");
  if (!sheet) sheet = ss.getActiveSheet();

  const ui = SpreadsheetApp.getUi();
  
  // 1. Check IP
  const ipAddress = sheet.getRange("B2").getValue();
  if (ipAddress === "" || ipAddress === null) {
    ui.alert("⚠️ Error: Cell B2 is empty. Please enter an IP Address.");
    return;
  }

  // 2. Set Trigger
  sheet.getRange("B3").setValue("TRUE");
  sheet.getRange("C3").setValue("Button Clicked... Waiting for Python...");
  ss.toast("🚀 Signal sent to Python Bot!", "Scraper Started");
}

```

---

## ⚠️ Troubleshooting

* **"Browser opens and closes instantly":**
* Delete the `chrome_profile` folder and run the script again.


* **"Quota Exceeded" / API Error:**
* The bot checks the sheet every 5-10 seconds. If you get API limits, increase the `time.sleep(5)` in the Python script to `time.sleep(10)`.


* **Bot is stuck on "Waiting...":**
* Ensure the Sheet name in Python (`SHEET_NAME = "Sender Score Tool"`) matches your actual Google Sheet name exactly.
