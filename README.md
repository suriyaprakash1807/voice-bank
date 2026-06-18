# 🎙️ VoiceBank — Voice-First Accessible Banking

> **Innovative Prototype:** Enabling blind and visually impaired users to perform UPI-like payments (similar to GPay / PhonePe) entirely through voice commands — no screen interaction required.

---

## 📌 Project Overview

VoiceBank is a **voice-first banking prototype** designed specifically for **blind and visually impaired users**. Instead of navigating a visual UI, users speak commands naturally — the system listens, understands, confirms via spoken PIN, and executes transactions.

| Layer      | Technology                              |
|------------|-----------------------------------------|
| Frontend   | HTML5 + CSS3 + Vanilla JavaScript       |
| Backend    | Java 17 + Spring Boot 3.2 (REST API)    |
| Storage    | Browser `localStorage` (prototype)      |
| Voice I/O  | Web Speech API (STT + TTS — built-in)   |
| NLP        | Java NLPService + JS fallback           |

> **Prototype note:** No database is used. All user data, balances, and transactions are stored in browser `localStorage`. This is intentional for rapid prototyping and offline demo purposes.

---

## 🗂️ Project Structure

```
voicebank-complete/
│
├── index.html                          ← Main frontend (Voice UI)
│
└── voicebank-backend/                  ← Java Spring Boot backend
    ├── pom.xml                         ← Maven dependencies
    ├── README.md                       ← Backend-specific notes
    └── src/
        └── main/
            ├── java/com/voicebank/
            │   ├── VoiceBankApplication.java        ← Spring Boot entry point
            │   ├── config/
            │   │   └── CorsConfig.java              ← CORS (allows frontend access)
            │   ├── controller/
            │   │   └── VoiceBankController.java     ← REST endpoints
            │   ├── model/
            │   │   ├── VoiceCommand.java             ← NLP result model
            │   │   └── TransferRequest.java          ← Transfer validation model
            │   └── service/
            │       ├── NLPService.java               ← Natural language parser
            │       └── TransferService.java          ← Transfer validation logic
            └── resources/
                └── application.properties            ← Server config (port 8080)
```

---

## 🚀 How to Run

### Step 1 — Start the Java Backend

**Requirements:** Java 17+, Maven 3.6+

```bash
cd voicebank-backend
mvn spring-boot:run
```

The backend starts at: `http://localhost:8080`

Verify it's running:
```
GET http://localhost:8080/api/health
→ { "status": "UP", "service": "VoiceBank NLP API" }
```

### Step 2 — Open the Frontend

Open `index.html` in **Google Chrome** or **Microsoft Edge**.

> ⚠️ **Important:** The Web Speech API requires Chrome or Edge. Firefox and Safari have limited support.

> ⚠️ **HTTPS Note:** For microphone access on deployed servers, the page must be served over HTTPS. For local development, `localhost` works without HTTPS.

---

## 🎯 How It Works — User Flow

```
[Auth Page]
    ↓  Login / Register
[Wake Page]
    ↓  Say "Hey VoiceBank" OR tap the orb
[Blind Mode Activated]
    ↓  System speaks: "Say your 4-digit PIN"
    ↓  User speaks PIN (e.g. "one two three four")
[Active — Continuous Voice Loop]
    ↓  Speak a command → Java NLP parses it → Confirm with PIN → Execute
```

---

## 🗣️ Voice Commands

| What to Say                        | Action                         |
|------------------------------------|--------------------------------|
| `"Check my balance"`               | Read current balance (PIN req) |
| `"Send 500 to Rahul"`              | Transfer ₹500 to Rahul (PIN)  |
| `"Transfer 1000 to Priya"`         | Transfer ₹1000 (PIN required) |
| `"Pay Amit 250"`                   | Pay Amit ₹250 (PIN required)  |
| `"Deposit 1000"`                   | Add money to account (PIN)    |
| `"Withdraw 500"`                   | Withdraw money (PIN required) |
| `"Show transactions"`              | Read last 5 transactions       |
| `"Help"`                           | List all available commands    |
| `"Logout"` / `"Goodbye"`          | Sign out                       |

**Wake words (on Wake page):**
- `"Hey VoiceBank"`, `"Hello VoiceBank"`, `"Hi VoiceBank"`
- `"Activate blind mode"`, `"Start blind mode"`

---

## ☕ Java Backend — API Endpoints

### `GET /api/health`
Health check.
```json
{ "status": "UP", "service": "VoiceBank NLP API", "version": "1.0.0" }
```

### `POST /api/parse-voice-command`
Parse raw voice text into structured command.

**Request:**
```json
{ "text": "Send 500 to Rahul" }
```

**Response:**
```json
{
  "intent":   "TRANSFER",
  "amount":   500.0,
  "receiver": "Rahul",
  "rawText":  "Send 500 to Rahul",
  "success":  true,
  "message":  "Transferring ₹500.00 to Rahul"
}
```

**Supported intents:** `TRANSFER`, `CHECK_BALANCE`, `SHOW_TRANSACTIONS`, `DEPOSIT`, `WITHDRAW`, `ERROR`

### `POST /api/validate-transfer`
Server-side transfer validation before localStorage is updated.

**Request:**
```json
{
  "sender":        "user@email.com",
  "receiver":      "Rahul",
  "amount":        500.0,
  "senderBalance": 2000.0
}
```

**Response (success):**
```json
{ "valid": true, "message": "₹500.00 transfer to Rahul is valid." }
```

**Response (failure):**
```json
{ "valid": false, "message": "Insufficient balance. Available: ₹200.00, Required: ₹500.00" }
```

---

## 🔒 Security Features (Prototype)

| Feature                   | Implementation                                   |
|---------------------------|--------------------------------------------------|
| Voice PIN authentication  | Required before every balance view or transfer   |
| PIN spoken recognition    | Accepts digits spoken as words ("one two three") |
| Transfer validation       | Java backend validates amount & balance limits   |
| Max transfer limit        | ₹1,00,000 per single transaction                |
| Self-transfer prevention  | Backend + frontend both block it                 |
| Session management        | `localStorage` session key (email-based)         |

---

## 🌐 Frontend Architecture

### Pages
1. **Auth Page** — Login / Register form with backend health indicator
2. **Wake Page** — Passive microphone listens for wake word
3. **Blind Mode Page** — Full voice UI with animated orb, live transcript, and NLP result card

### Key JS Modules

| Module              | Purpose                                                   |
|---------------------|-----------------------------------------------------------|
| `checkBackend()`    | Polls Java backend health every 30 seconds               |
| `apiParseCommand()` | Calls Java NLP API; falls back to local JS parser        |
| `apiValidateTransfer()` | Calls Java validation; falls back to local check    |
| `localParseCommand()` | Mirrors Java NLPService patterns (offline fallback)   |
| `textToPin()`       | Converts spoken words to 4-digit PIN string              |
| `listenOnce()`      | Single-shot speech recognition session                   |
| `startPassive()`    | Continuous wake-word detection loop                      |
| `blindListen()`     | Main voice command loop in blind mode                    |
| `speak()`           | Text-to-speech with callback chaining                    |

### Graceful Degradation
If the Java backend is **offline**, the frontend:
- Shows an amber warning badge ("Backend offline — using local NLP fallback")
- Automatically switches to the built-in JS NLP parser
- Continues to work fully — just without server-side validation

---

## 🧪 Demo Accounts

Three accounts are pre-seeded in localStorage on first launch:

| Name          | Email             | Password  | PIN  | Balance  |
|---------------|-------------------|-----------|------|----------|
| Rahul Sharma  | rahul@demo.com    | demo123   | 1234 | ₹10,000  |
| Priya Patel   | priya@demo.com    | demo123   | 1234 | ₹5,000   |
| Amit Kumar    | amit@demo.com     | demo123   | 5678 | ₹8,000   |

---

## 🔧 Configuration

### Change Backend Port
Edit `voicebank-backend/src/main/resources/application.properties`:
```properties
server.port=8080
```

Update frontend to match — in `index.html`, find:
```javascript
const API_BASE = 'http://localhost:8080/api';
```

### Reset All Data
Open browser console on `index.html` and run:
```javascript
localStorage.clear(); location.reload();
```

---

## 🛣️ Future Roadmap (beyond prototype)

- [ ] Replace localStorage with a real database (PostgreSQL / MySQL)
- [ ] Add JWT authentication instead of session strings
- [ ] Integrate real UPI payment rails (NPCI / bank APIs)
- [ ] Support regional Indian languages (Hindi, Tamil, Telugu, etc.)
- [ ] Android/iOS app with native TTS & STT
- [ ] OTP-based 2FA for transfers above ₹10,000
- [ ] Voice biometrics (speaker recognition) as an additional auth layer
- [ ] Screen reader (NVDA / JAWS) compatibility mode

---

## 💡 Innovation Highlights

1. **No screen required** — 100% of the banking workflow is voice-driven
2. **Dual NLP** — Java backend for accuracy + JS fallback for offline resilience
3. **Spoken PIN confirmation** — Mirrors physical UPI PIN security in voice form
4. **Wake word activation** — Hands-free from sleep, like a voice assistant
5. **Real-time feedback** — Orb animations + spoken responses for every state change
6. **Inclusive by design** — Built for blind users first; sighted users can use it too

---

## 📋 Requirements

| Item                | Minimum                         |
|---------------------|---------------------------------|
| Java                | 17+                             |
| Maven               | 3.6+                            |
| Browser             | Chrome 80+ or Edge 80+          |
| Internet            | Required for Google Fonts only  |
| Microphone          | Required for voice input        |

---

*VoiceBank — Making digital payments accessible for everyone.*
