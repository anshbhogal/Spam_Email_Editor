# Spam_Editor1


## Spam Editor – Gmail Spam & Important Email Classifier

This project connects to your Gmail account, classifies incoming emails as **IMPORTANT** or **SPAM** using a fine-tuned **DistilBERT** model, and updates Gmail labels accordingly (star/important vs. spam).

### 1) Features
- Gmail integration (official Gmail API for read/modify).
- DistilBERT classifier for important vs spam.
- Hybrid rules + ML: critical security keywords (OTP, bank, login, etc.) force **IMPORTANT**.
- Background listener: polls unread mail and labels automatically.

### 2) Project structure
- `main.py` – Entry point; starts the Gmail listener.
- `gmail/`
  - `auth.py` – OAuth flow and token caching.
  - `fetcher.py` – Fetches/parses Gmail messages.
  - `actions.py` – Marks messages IMPORTANT/STARRED or SPAM.
  - `listener.py` – Polls unread emails and calls the classifier.
- `ml/`
  - `train.py` – Fine-tunes DistilBERT on a labeled email dataset.
  - `predict.py` – Loads the trained model and exposes `predict_email(...)`.
- `models/distilbert/` – Saved model weights/tokenizer.
- `data/` – Place your dataset here (default filename: `emails.csv`; not shipped).
- `tools/` – Helper scripts to build/clean datasets from raw Gmail exports.
- `requirements.txt` – Python dependencies.

### 3) Setup
- Python 3.10+ is recommended.
- Gmail API requires a Google Cloud project with Gmail API enabled.

#### 3.1 Install dependencies
```bash
pip install -r requirements.txt
```

#### 3.2 Configure Gmail API credentials (Google Cloud Console)
1) Create a project  
   - Go to `https://console.cloud.google.com/`.  
   - Project dropdown → **New Project** → name it (e.g., `spam-editor`) → **Create**.

2) Enable Gmail API  
   - **APIs & Services → Library** → search **Gmail API** → **Enable**.

3) OAuth consent screen  
   - **APIs & Services → OAuth consent screen**.  
   - User type: **External** (simplest).  
   - Fill **App name**, **User support email**, **Developer contact email**.  
   - Save/continue; keep default scopes.  
   - Add yourself as **Test user** (recommended).  
   - Save and finish.

4) Create OAuth client credentials  
   - **APIs & Services → Credentials → + CREATE CREDENTIALS → OAuth client ID**.  
   - Application type: **Desktop app** (for local scripts).  
   - Name it (e.g., `Spam Editor Desktop`) → **Create**.  
   - **Download JSON** and save as `credentials.json` in the project root (same folder as `main.py`).

5) First run & token caching  
   - On first use, a browser window opens for Google login and consent.  
   - A `token.pickle` (or similar) is created and reused on subsequent runs.

6) Security/Git hygiene  
   - Do **not** commit `credentials.json`, `token.pickle`, or any `.env` with secrets.  
   - Add them to `.gitignore`; treat them as sensitive credentials.

### 4) Running the listener
```bash
python main.py
```
What it does:
- Authenticates with Gmail.
- Every ~60 seconds, fetches unread emails.
- Concatenates subject + body, runs `ml.predict.predict_email`, then:
  - Marks IMPORTANT emails as STARRED and IMPORTANT.
  - Marks non-important emails as SPAM.
Keep it running while you want auto-labeling.

### 5) Training or retraining the model
The model expects a CSV at `data/emails.csv` with:
- `text` – full email text (subject + body).
- `label` – `1` for IMPORTANT, `0` for SPAM.

Train:
```bash
python ml/train.py
```
This loads the dataset, fine-tunes `distilbert-base-uncased`, and writes weights/tokenizer to `models/distilbert/` (used by `ml/predict.py`).

### 6) Building your own dataset (using `tools/`)
No real dataset is shipped. Build your own from your Gmail data:

1) Fetch raw emails  
   - Use the Gmail utilities in `gmail/` (e.g., `fetcher.py`) after credentials are set.  
   - They should save raw messages locally (subject/body/label). Check script help for exact output format/path.

2) Build a labeled CSV with `tools/dataset_builder.py`  
   - From repo root, adjust flags to your script’s CLI (example):
```bash
python tools/dataset_builder.py \
  --input-path path/to/raw_emails.json \
  --output-path data/my_custom_emails_raw.csv
```

3) Convert/validate with `tools/convert_and_validate_dataset.py`  
   - Ensure schema and labels match expectations (`text`, `label` in {0,1}):
```bash
python tools/convert_and_validate_dataset.py \
  --input data/my_custom_emails_raw.csv \
  --output data/emails.csv
```

4) Train on your custom data  
```bash
python ml/train.py
```

### 7) Privacy & safety notes
- Never commit real email content, credentials, tokens, or secrets.  
- Prefer synthetic/anonymized datasets; or keep `data/` git-ignored.  
- Follow Google API policies and your organization’s security rules.

### 8) Troubleshooting
- Authentication prompts every run: ensure `token.pickle` is writable and reused.  
- Dataset errors: verify `data/emails.csv` has `text` and `label` columns with labels `0/1`.  
- Model issues: confirm `models/distilbert/` exists after training; rerun `ml/train.py` if missing.



