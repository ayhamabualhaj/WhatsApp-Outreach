# CLAUDE.md — WhatsApp Outreach App

## Project overview
This project is a Python app that sends WhatsApp messages automatically from the owner's own WhatsApp number to a small list of companies, offering the owner's company services. It starts small (5–10 companies) and should be designed so it can grow later, including switching to the official WhatsApp Cloud API when budget allows.

The owner is a Data Science and AI student who knows Python well. Write clean, readable code with comments so he can understand and extend it.

## Tech stack
- **Language:** Python 3.10+ (local venv is `.venv`, currently Python 3.14)
- **WhatsApp library:** Neonize (github.com/krypton-byte/neonize), a Python SDK built on the Go library whatsmeow. It connects to WhatsApp directly with no browser. Pinned to `neonize==0.5.2`.
- **Contacts input:** a CSV file
- **Interface:** a simple local web interface built with Streamlit (pure Python, no HTML/JS needed)
- Always check the current Neonize documentation and examples (or the installed source in `.venv/lib/python*/site-packages/neonize/`) before writing code. Do not guess the API.

## Important constraints
- Neonize is unofficial. Automating a personal WhatsApp account breaks WhatsApp's terms, and the number can be banned.
- Keep volume low: default max 20 messages per run.
- Add a random delay between messages (default 30–90 seconds).
- Every message must be personalized (at least the company name). No identical mass blasts.
- Never send to the same number twice unless explicitly forced.
- Include a dry-run mode that prints messages without sending.

## Architecture (keep it swappable)
Put all WhatsApp-specific code behind one interface so the sending backend can be replaced later with the official WhatsApp Cloud API without rewriting the rest. Keep the interface (UI) separate from the logic: the UI only calls functions, it never talks to WhatsApp directly.

The repo root is the project folder:

```
WhatsApp_Outreach/
  CLAUDE.md
  README.md
  requirements.txt
  config.yaml              # limits, delays, template path, variants
  contacts.example.csv     # fake example input (real contacts.csv is gitignored)
  templates/
    offer_ar.txt           # default Arabic template
    offer_en.txt           # English template (chosen via CSV "template" column)
  src/
    app.py                 # Streamlit interface (entry point)
    sender_base.py         # abstract Sender interface
    neonize_sender.py      # Neonize implementation
    contacts.py            # load + validate CSV
    templating.py          # fill placeholders
    campaign.py            # sending loop (limits, delays, dry run)
    log_store.py           # record sent messages (SQLite)
  tests/
```

## Contacts CSV format
Columns: `company_name, phone, contact_name, industry, city, custom_line, template, notes` (all optional except `company_name` and `phone`).
- Phone numbers in international format without `+` or spaces, e.g. `9627XXXXXXXX` for Jordan.
- Validate and normalize numbers; skip and report invalid rows.

## Message template
Every company gets its own personalized message, built from a template plus its row in the CSV. This costs nothing extra; it's just text filling in code.

**Arabic first:** most messages will be in Arabic, so Arabic is the default language.
- The default template is `templates/offer_ar.txt`; English (`offer_en.txt`) is only used when the CSV `template` column says so.
- Read and write all files as UTF-8 (CSV, templates, SQLite, logs) so Arabic text never breaks. Handle CSVs saved from Excel with a BOM (`utf-8-sig`).
- The interface must display Arabic right-to-left (RTL) in the template editor, message previews, and tables.
- Arabic fallbacks: e.g. no `contact_name` → "مرحبًا،" instead of "مرحبًا أحمد،".
- Test with real Arabic company names, including mixed Arabic/English text and numbers.

Template rules:
- The template is a plain text file with placeholders like `{company_name}`, `{contact_name}`, `{industry}`, `{city}`, `{custom_line}`. Also `{my_name}`, `{my_company}` (from `config.yaml` → `sender`) and `{greeting}`, `{closing}` (from `config.yaml` → `variants`).
- Any CSV column automatically becomes a placeholder, so adding a new column (e.g. `industry`) lets the owner use `{industry}` without code changes.
- Fallbacks: if a field is empty, use a sensible default (e.g. no `contact_name` → "Hello," instead of "Hello Ahmad,"). Never send a message with a raw `{placeholder}` left in it; block it and show a warning.
- Custom line: an optional `custom_line` column holds one hand-written sentence for that company (e.g. something about their recent work). If present, insert it; if empty, skip it cleanly.
- Multiple templates: support several template files (e.g. `offer_en.txt`, `offer_ar.txt`) and an optional `template` column in the CSV to choose one per company, so Arabic and English companies can get messages in their language.
- Light variation: optionally pick randomly between a few greeting/closing variants so messages aren't word-for-word identical.
- The interface must show the final message for every company before sending, and let the owner edit a single company's message by hand.

## Build steps (do them in order)
1. **Setup:** create the folder structure, `requirements.txt`, and a virtual environment guide in README. ✅
2. **Connect:** write a minimal Neonize script that shows a QR code, lets the owner scan it with his phone, and saves the session so he doesn't rescan every time. Confirm it works by sending one test message to the owner's own number.
3. **Contacts:** implement CSV loading, phone validation, and an error report.
4. **Templates:** implement placeholder filling and preview.
5. **Sender interface:** create `Sender` base class with `send_text(phone, text)`; implement `NeonizeSender`.
6. **Sending loop:** send to each contact with random delays, the per-run limit, and dry-run mode.
7. **Logging:** store every send (phone, company, time, status, error) in SQLite; skip already-contacted numbers.
8. **Interface (Streamlit):** build `src/app.py`, run with `streamlit run src/app.py`. It should have:
   - **Connection:** shows whether WhatsApp is connected; if not, displays the QR code to scan (fall back to the terminal QR if showing it in the UI is hard).
   - **Contacts:** upload a CSV, show it as a table, highlight invalid rows and already-contacted numbers.
   - **Message:** edit the template in a text box and preview the final message for each company.
   - **Send:** a dry-run toggle, a limit setting, and a Send button with a progress bar and live status per contact. Run the sending in a background thread so the page doesn't freeze during the delays.
   - **History:** a table of everything sent from the SQLite log.
9. **Tests:** unit tests for CSV parsing, templating, and duplicate skipping (mock the sender).
10. **README:** explain setup, QR login, running, and the ban risks.

## Coding rules
- Small, well-named functions with docstrings.
- Config values in `config.yaml`, not hardcoded.
- Handle errors per contact so one failure doesn't stop the run.
- Never commit session files or contacts; add them to `.gitignore`.
- After each step, stop and explain what was built and how to test it before moving on.

## Git & GitHub
1. The repo is `git@github.com:ayhamabualhaj/WhatsApp-Outreach.git` (remote `origin`, branch `main`). It already exists, so no `gh repo create` is needed. If any GitHub CLI action is needed and `gh` isn't installed or logged in, stop and ask the owner to run `gh auth login`; never ask for or store a password or token in the code.
2. After each build step is finished and tested, commit with a clear message (e.g. `step 3: CSV loading and validation`) and push to `main`.
3. `.gitignore` must include: the Neonize session/database files, `contacts.csv` and any real contact lists, the SQLite send log, `.env`, `venv/`/`.venv/`, `__pycache__/`. Commit only an example file like `contacts.example.csv` with fake data.
4. Before every push, check `git status` to make sure no session files, real phone numbers, or logs are staged.

## Future (do not build yet)
- `CloudApiSender` using the official WhatsApp Cloud API.
- Handling replies and follow-ups.
