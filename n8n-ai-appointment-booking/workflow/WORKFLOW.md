\# 🛠️ Workflow Node Documentation



This document outlines the detailed configuration and responsibilities of each node in the n8n appointment booking pipeline.



\---



\## 1. Telegram Trigger

\- \*\*Type:\*\* `n8n-nodes-base.telegramTrigger`

\- \*\*Updates:\*\* `message`

\- \*\*Output:\*\* Captures incoming user message text, user ID, and first name.



\---



\## 2. Extract Booking Details (Gemini LLM)

\- \*\*Type:\*\* `@n8n/n8n-nodes-langchain.chainLlm`

\- \*\*Role:\*\* Extracts intent, requested service, date, time, and duration into a JSON structure.

\- \*\*Key Prompt Rules:\*\*

&#x20; - Standardizes times into 24h format (`HH:mm`).

&#x20; - Sets `is\_booking\_intent` to `false` for greetings or generic text.

&#x20; - Defaults `service` to `"General Consultation"` if omitted.



\---



\## 3. Calculate End Time (Code Node)

\- \*\*Type:\*\* `n8n-nodes-base.code`

\- \*\*Language:\*\* JavaScript

\- \*\*Functionality:\*\*

&#x20; - Safely extracts JSON from raw text or markdown code blocks returned by AI.

&#x20; - Converts any remaining 12-hour strings (`5:00 PM`) to 24-hour integers (`17:00`).

&#x20; - Calculates start time and end time based on `duration\_minutes`.



\---



\## 4. Get Existing Bookings (Google Sheets)

\- \*\*Type:\*\* `n8n-nodes-base.googleSheets`

\- \*\*Operation:\*\* `read`

\- \*\*Filter:\*\* Filters rows where `Date` equals `{{ $('Calculate End Time').item.json.requested\_date }}`.



\---



\## 5. Validate Conflict \& Business Hours (Code Node)

\- \*\*Type:\*\* `n8n-nodes-base.code`

\- \*\*Language:\*\* JavaScript

\- \*\*Validation Sequence:\*\*

&#x20; 1. \*\*Intent Check:\*\* Flags non-booking intents (greetings).

&#x20; 2. \*\*Missing Time Check:\*\* Flags missing times.

&#x20; 3. \*\*Business Hours:\*\* Enforces `09:00` to `21:00` range.

&#x20; 4. \*\*Conflict Loop:\*\* Checks if `\[start\_time, end\_time]` overlaps with any existing booking's `\[Start\_Time, End\_Time]`.



\---



\## 6. If Available (Switch / IF Node)

\- \*\*Type:\*\* `n8n-nodes-base.if`

\- \*\*Condition:\*\* `{{ $json.is\_available }}` EQUALS `true`

\- \*\*True Branch:\*\* Append row to Google Sheets ➔ Send Telegram Confirmation.

\- \*\*False Branch:\*\* Send Telegram Rejection / Guidance Alert.

