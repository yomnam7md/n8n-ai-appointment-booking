# 📅 Automated Appointment Booking System | Telegram Bot & n8n

A fully automated, end-to-end appointment booking system integrated with **Telegram**, powered by **n8n** and **Google Gemini AI** for natural language processing and data extraction, featuring direct integration with **Google Sheets** for schedule management and conflict prevention.

---

## 🌟 Key Features

* **AI-Powered Natural Language Processing:** Parses user messages to extract structured booking details (Name, Service, Date, Time) using Google Gemini.
* **Smart Conflict Detection:** Queries existing appointments in Google Sheets to prevent double-booking overlapping time slots.
* **Business Hours Enforcement:** Rejects booking requests made outside configured operating hours (e.g., 09:00 AM – 09:00 PM).
* **Intent Recognition:** Distinguishes between booking requests and general greetings, guiding users accordingly.
* **Automatic End-Time Calculation:** Automatically calculates appointment end times based on service duration.
* **Automated Customer Response:** Sends immediate confirmation details or clear conflict explanations directly to the user via Telegram.

---

## 🏗️ Workflow Architecture

The n8n workflow consists of the following connected nodes:

1. **Telegram Trigger:** Receives incoming messages from users via Telegram.
2. **Booking Assistant Extractor (LangChain + Gemini):** Analyzes raw text and extracts structured JSON payload.
3. **Calculate End Time (Code Node):** Converts time to 24-hour format, calculates session end time, and cleans input data.
4. **Get Row(s) in Sheet (Google Sheets):** Fetches recorded bookings for the requested date.
5. **Validate Conflict & Business Hours (Code Node):** Checks slot availability against existing bookings and business hours.
6. **If Node:** Routes execution based on slot availability.
7. **Append Row in Sheet (Google Sheets):** Appends the new booking with a unique ID (`BK-XXXX`).
8. **Send Telegram Message:** Delivers the final response (Confirmation or Rejection reason) to the user.

---

## 📊 Google Sheets Setup

Create a new Google Sheet and set up the following column headers in the first row:

| Booking ID | Customer Name | Telegram ID | Service | Date | Start_Time | End_Time | Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| BK-1001 | John Doe | 123456789 | General Consultation | 2026-09-25 | 14:00 | 14:30 | Confirmed |

> ⚠️ **Note:** Ensure your worksheet tab name matches your n8n node settings (e.g., `Sheet1` or `gid=0`).

---

## ⚙️ Prerequisites

1. **n8n Instance:** Self-hosted or cloud instance of n8n.
2. **Telegram Bot Token:** Generated via [@BotFather](https://t.me/botfather).
3. **Google Gemini API Key:** Obtained from [Google AI Studio](https://aistudio.google.com/).
4. **Google Sheets Credentials:** Connected in n8n via OAuth2 or Service Account.

---

## 🚀 Installation & Setup

1. **Import Workflow:**
   * Open your n8n canvas.
   * Click on the top menu, select **Import from File**, and select your `workflow.json` file.
   * *(Alternatively, copy the raw JSON workflow code and press `Ctrl + V` directly on the n8n canvas).*

2. **Configure Credentials:**
   * Open **Telegram Trigger** and **Send Telegram Message** nodes to attach your Telegram Bot token.
   * Open **Google Gemini Chat Model** node and input your Gemini API Key.
   * Open both **Google Sheets** nodes to select your Google account and insert your Document ID.

3. **Activate Workflow:**
   * Switch the workflow toggle from `Inactive` to `Active` in the upper right corner.

---

## 🛠️ Customization

* **Business Operating Hours:**
  Modify start and end times inside the **Validate Conflict & Business Hours** code node:
  ```javascript
  const workStart = "09:00"; // Opening time
  const workEnd = "21:00";   // Closing time
