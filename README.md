# WhatsApp Outreach

Sends personalized WhatsApp messages (Arabic first) from your own number to a small list of companies, using [Neonize](https://github.com/krypton-byte/neonize) and a local Streamlit interface.

> ⚠️ **Ban risk:** Neonize is unofficial. Automating a personal WhatsApp account breaks WhatsApp's terms and the number can be banned. Keep volumes low (default max 20 per run, 30–90 s random delay) and always personalize.

## Setup

Requires **Python 3.10+**.

### 1. Create and activate a virtual environment

Linux / macOS:
```bash
python3 -m venv .venv
source .venv/bin/activate
```

Windows (PowerShell):
```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

(PyCharm: *Settings → Project → Python Interpreter → Add → Virtualenv → Existing* → `.venv`.)

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

### 3. Configure
- Edit `config.yaml` (your name and company under `sender`, limits, delays).
- Copy the example contacts and fill in real ones (the real file is gitignored):
  ```bash
  cp contacts.example.csv contacts.csv
  ```
- Edit the message in `templates/offer_ar.txt` (and `offer_en.txt` if needed).

## Project layout
| Path | Purpose |
|---|---|
| `config.yaml` | Limits, delays, template defaults, greeting/closing variants, file paths |
| `contacts.example.csv` | Fake example contact list (format reference) |
| `templates/` | Message templates with `{placeholders}` |
| `src/` | Application code |
| `tests/` | Unit tests (`pytest`) |

## QR login, running, and usage
_Coming in later build steps (see `CLAUDE.md`)._
