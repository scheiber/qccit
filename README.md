# QCC IT Help Desk Demo

## [See the live demo here!](https://qccit.scheiber.dev/)

A clickable prototype of a virtual IT help desk for Queensborough Community College, built for a student proposal to replace the help desk phone line with AWS services (Amazon Connect, Lex, Transcribe, Translate, Polly, Lambda, DynamoDB, SNS).

It is a single HTML file with no build step and no server. Open it in a browser and it works offline in demo mode.

> **Not an official service.** This is a student prototype. It is not affiliated with, endorsed by, or operated by CUNY or Queensborough Community College. Use made-up names and IDs only.

## Running it

1. Open `qcc-it-help-demo.html` in Chrome, Edge, Safari, or Firefox.
2. That's it. Everything (code, styles, fonts, logo, sample data) is inside the one file.

For the best experience, use **Chrome or Edge**. They support both speaking and listening in the phone call demo. See [Browser notes](#browser-notes).

## What it shows

The left-hand navigation has seven sections.

### Student channels

| Section | What it demonstrates | AWS services it stands in for |
|---|---|---|
| **Web chat** | A chat assistant that answers only from approved content (the CUNY IT Help Q&A). Anything it can't answer becomes a ticket instead of a guess. Answers can be read aloud. | Amazon Connect chat, Amazon Lex, Lambda |
| **Text message** | The same assistant in a phone mockup, with short SMS-sized replies and working STOP/START opt-out. | Amazon Connect SMS |
| **Phone call** | A simulated call. The assistant speaks; you can speak back (Chrome/Edge) or type. If it can't help, it takes a message, which becomes a ticket. Repeat calls from the same number update one ticket instead of creating duplicates. | Amazon Connect voice, Lex |
| **Voice message** | After-hours voicemail. Pick a sample (English, Spanish, Mandarin, Korean, Bengali) or record your own. An animated pipeline shows upload, transcription, translation, ticket creation, and the staff alert. Recordings are capped at 90 seconds. | S3, Transcribe, Translate, Lambda, DynamoDB, SNS |

### Help desk

| Section | What it shows |
|---|---|
| **Ticket queue** | The staff view: 18 sample tickets plus any you create, with filters (status, channel, assignee), priority, category, assignment, notes, translation-check flags, duplicate merging, and CSV export. Below it, an assistant activity panel (labeled as sample data) and a live feed of recent conversations. |
| **Status and contacts** | Service announcements and the campus help desk directory, taken from the Q&A. |
| **Cost meter** | What your demo session would have cost on AWS, plus an annual projection with the cost analysis's scenarios A, B, and C. |

## Where the content comes from

- **Assistant answers:** only from the *CUNY IT Help: Common Questions and Quick Fixes* Q&A, which is adapted from the CUNY IT Help portal (cunyithelp.cuny.edu). The assistant never makes up an answer; anything outside that content is handed off to a person as a ticket. Confirm that CUNY allows reuse of its portal text before publishing.
- **Costs:** per-unit AWS rates from the project's cost risk analysis:

  | Item | Rate |
  |---|---|
  | Connect chat message | $0.010 |
  | Connect SMS message | $0.014 |
  | Connect voice | $0.038 per minute |
  | Inbound telephony | $0.0022 per minute |
  | Transcribe | $0.024 per minute (15-second minimum) |
  | Translate | $15 per million characters |
  | Ticket processing (Lambda, DynamoDB, SNS) | about $0.002 per ticket |

  The annual projection adds $11 a year for the phone number. It excludes SMS carrier fees, telecom taxes, support, and build labor.
- **Sample data:** all names, tickets, staff, phone numbers (555 numbers), and activity figures are invented for the demo. The assistant performance panel is labeled "Sample data" and does not reflect real usage.

## Settings

Open **Settings** in the top right.

- **Theme:** six themes, all built from the QCC Identity Standards palette.
  - *With sidebar:* **Light** (default; light page, Midnight Blue sidebar), **Midnight** (Midnight Blue page, Marigold sidebar), **Marigold** (Marigold page, Midnight Blue sidebar).
  - *Without sidebar:* **Plain** (light page), **Campus** (white header with the Midnight Blue logo), **High contrast** (white page, black text, strong borders, underlined links).

  Picking a theme previews it immediately. **Save** keeps it, and **Cancel** or **Esc** reverts.
- **Live mode:** paste a Lex relay URL to send chat and text messages to a real Amazon Lex bot (see below). Leave it empty for demo mode.
- **Read chat answers aloud automatically** and **Assistant voice:** control the browser's built-in text-to-speech.
- **Reset demo data:** clears your tickets and session costs and restores the sample tickets. Your theme is kept.

## Live mode (optional)

Demo mode uses built-in answers. To connect the chat and text channels to a real Lex V2 bot, deploy a small relay, such as a Lambda function with a function URL, and paste its URL into Settings.

**Request** (POST, JSON):

```json
{ "text": "I forgot my password", "sessionId": "chat-1" }
```

**Response** (JSON): the page shows the `message` field (or `reply` if `message` is missing).

```json
{ "message": "To reset your CUNY Login password, go to managelogin.cuny.edu..." }
```

If the request fails, the page shows a notice and falls back to the built-in answers. Allow the page's origin in the function URL's CORS settings.

A minimal relay in Python might look like this. It's an untested sketch; replace the IDs with your bot's.

```python
import json, boto3

lex = boto3.client("lexv2-runtime")
BOT_ID, ALIAS_ID, LOCALE = "YOUR_BOT_ID", "YOUR_ALIAS_ID", "en_US"

def handler(event, context):
    body = json.loads(event.get("body") or "{}")
    resp = lex.recognize_text(botId=BOT_ID, botAliasId=ALIAS_ID, localeId=LOCALE,
                              sessionId=body.get("sessionId", "demo"), text=body.get("text", ""))
    msgs = [m.get("content", "") for m in resp.get("messages", [])]
    return {"statusCode": 200, "headers": {"Content-Type": "application/json"},
            "body": json.dumps({"message": " ".join(msgs) or "Sorry, I didn't catch that."})}
```

In an AWS Academy Learner Lab, attach the pre-made `LabRole` to the function, since Learner Lab doesn't let you create IAM roles.

The phone call, voice message, and staff alert features stay simulated in live mode.

## Data and privacy

- Tickets and settings are saved in your browser's `localStorage` under the key `qccItDemo`. Nothing is sent anywhere unless you turn on live mode.
- Each browser and device keeps its own copy. Clearing site data, or **Reset demo data**, removes your tickets.
- Microphone access is requested only when you record a voice message or speak during the phone call. Speech recognition uses the browser's built-in service, which in Chrome and Edge may send audio to the browser vendor.
- Don't enter real student information. The demo isn't built to protect personal data (FERPA).

## Branding

The page follows the *Queensborough Community College Identity Standards* (2018):

- **Logo:** the full horizontal logo, white on a Midnight Blue band or Midnight Blue on a white band (the two web treatments on page 39), upper left, 48px tall with the required clear space.
- **Colors:** Midnight Blue `#00306E` and Marigold `#F7B72D` lead, with the web palette's Light Blue `#61ACF0` and Dark Grey `#575757` in support. Secondary colors (Coral, Burnt Umber, Gold) appear only in small accents, under the guide's 10% limit. Pale fills are tints of the official colors.
- **Type:** Interstate for headings and Archer for body text (page 39). Small section labels use the guide's Archer Semibold capitals subhead style.

**Known small deviations:** a few neutral grays outside the guide's palette (helper text, the record light's off state, one note in the call panel). The text-message phone mockup also uses iPhone-style grays and the device's system font on purpose. Interstate is used for small helper text, where the guide's rule of thumb is Archer.

### Fonts and licensing

Interstate and Archer are commercial typefaces (Font Bureau and Hoefler&Co). The file embeds Latin-only subsets made from the college's licensed font files, with these changes for consistent rendering across systems:

- vertical metrics unified, which fixed bold text sitting too high on macOS
- hinting removed and full smoothing enabled, which fixed jagged text on Windows

The letter shapes are unchanged. Because the fonts are embedded, **anyone who receives the HTML file also receives the fonts.** Archer's files are marked for preview and print embedding only. Presenting from your own computer is fine, but check that the college's web font license covers it before posting the file publicly.

## Browser notes

| Feature | Chrome / Edge | Safari | Firefox |
|---|---|---|---|
| Everything except speech | Yes | Yes | Yes |
| Read aloud (text-to-speech) | Yes | Yes | Yes (see below) |
| Speaking during the phone call | Yes | Varies | No; type instead |
| Live transcript while recording a voice message | Yes | Varies | No; type the transcript |

- **Firefox on Mac** adds a beep after spoken sentences. This is a browser bug: it happens even with a one-line `speechSynthesis.speak()` call on any page, and it doesn't happen in Firefox on Windows or Android. The page shows a one-time notice in that browser. Use Chrome for presenting.
- In production, read-aloud would use Amazon Polly audio files, which play the same way in every browser.

## Project files

| File | Description |
|---|---|
| `qcc-it-help-demo.html` | The demo (single file, about 250 KB). |
| `README.md` | This file. |
| `Help Desk QA .pdf` | Source content for the assistant's answers. |
| `Cost Risk Analysis Replacing the Phone System with AWS Standard Account.pdf` | Source for the cost rates and scenarios. |
| `QCCIdentityStandards.pdf` | Brand guidelines the design follows. |
| `AWS_Sandbox_and_Free_Demo_Plan.pdf` | Research on free AWS sandboxes (AWS Academy Learner Lab, Builder Center) and how to build a live version at no cost. |

## Limitations

- The assistant matches keywords against a fixed set of answers; it isn't a real language model. Live mode connects the chat and text channels to an actual Lex bot.
- Translation in demo mode uses pre-written translations for the sample voice messages. Recordings in other languages show a placeholder unless live services are added.
- Phone calls, SMS, and email alerts are simulated. Real phone numbers and texting need Amazon Connect and carrier registration, which aren't available in a free Learner Lab (see the sandbox plan PDF).
- Announcements and directory entries are from the Q&A dated October 8, 2026, and may be out of date.
