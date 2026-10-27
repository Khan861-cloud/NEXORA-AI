# NEXORA AI — Your Personal Intelligent Agent

A locally runnable, modular AI assistant with a React dashboard, FastAPI backend, Google Gemini integration, local Ollama support, voice interaction, secure accounts, and persistent conversations.

## 1. Requirements

Install the following before starting:

- Python 3.11.
- Node.js 22.12 or a compatible supported Node.js 22 release.
- npm, installed with Node.js.
- Git (recommended).
- Ollama, if you want to use a local model.

Check your versions:

```powershell
py -3.11 --version
node --version
npm --version
```

## 2. Create the environment

From the project root:

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

If PowerShell blocks activation, use this command in the current terminal:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

Alternatively, call the environment's Python directly:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

## 3. Configure environment variables

Create the root `.env` from the example:

```powershell
Copy-Item .env.example .env
```

Generate a session secret:

```powershell
python -c "import secrets; print(secrets.token_urlsafe(48))"
```

Put the generated value into `SESSION_SECRET` in `.env`.

Generate a separate Fernet-compatible credential-encryption key:

```powershell
python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
```

Store that value in `CREDENTIAL_ENCRYPTION_KEY` if encrypted credential storage is enabled.

Do not commit `.env`. Back up encryption keys securely: losing an encryption key can make stored credentials unrecoverable. Never use a sample key in production.

## 4. Configure the frontend

Open a second terminal at the project root:

```powershell
Copy-Item frontend/.env.example frontend/.env
Set-Location frontend
npm install
```

Return to the project root when you need to run backend commands:

```powershell
Set-Location ..
```

Keep `frontend/package-lock.json` in version control.

## 5. Configure Ollama

Install Ollama from its official website if it is not already installed.

Pull the requested model:

```powershell
ollama pull qwen3:4b
```

Check that it is installed:

```powershell
ollama list
```

Test it directly:

```powershell
ollama run qwen3:4b
```

Ask it a question, then exit the interactive session. Confirm the local API responds:

```powershell
Invoke-RestMethod http://127.0.0.1:11434/api/tags
```

If Ollama is not running, launch the Ollama application or start its service using the installation method for your system.

The application should remain usable in text mode even when Ollama is unavailable.

## 6. Configure Google Gemini

Create an API key through Google's official Gemini API setup.

For a single-user local installation, the key may be supplied in the private root `.env`:

```dotenv
GEMINI_API_KEY=your_actual_key_here
```

Do not commit the key or expose it to the browser.

For multi-user use, configure a protected settings endpoint that validates the key and stores it using encrypted per-user storage. Never replace another user's credentials with a global environment variable.

## 7. Start the application

Start the backend from the root directory:

```powershell
.\.venv\Scripts\python.exe -m uvicorn app.main:app --app-dir backend --host 127.0.0.1 --port 8000 --reload
```

This command assumes the backend implementation has been created and the environment is configured.

In another terminal:

```powershell
Set-Location frontend
npm run dev
```

Open the local URL printed by Vite, normally:

http://localhost:5173

The backend OpenAPI documentation will be available at:

http://127.0.0.1:8000/docs

**Note:** the startup commands become runnable after the implementation phase creates `backend/app/main.py` and the frontend entry point.

## 8. Voice interaction

The initial browser voice implementation uses supported browser speech APIs.

- Grant microphone permission when prompted.
- Use the microphone button only when you want to speak.
- Stop listening when finished.
- Use the text input if speech recognition is unsupported or permission is denied.
- Use the stop-speaking control to interrupt synthesis.

Microphone access generally requires HTTPS or localhost. Browser speech recognition may use a remote service and is not guaranteed to work offline. Browser speech synthesis availability and voice quality depend on the operating system and browser.

Optional offline speech recognition engines can be installed later:

- **Faster-Whisper:** Python speech recognition with downloaded model weights; installation size and speed depend on the selected model and hardware.
- **Vosk:** local speech recognition using compatible downloaded acoustic models.

These are speech-to-text engines, not text-to-speech engines.

## 9. Tests

After the test suite has been implemented:

```powershell
.\.venv\Scripts\python.exe -m pytest backend/tests -v
```

The automated tests should mock external AI services where possible. Do not interpret a passing unit test as proof that a real Gemini key, local microphone, or hardware-dependent speech model works.

## 10. Data and privacy

The local application stores account information in SQLite and conversation history in `data/long_term_memory.json`.

- Do not commit runtime data or credentials.
- Restrict access to the machine and project directory.
- Use authenticated authorization checks for every private conversation operation.
- Set an explicit retention period.
- Back up data and encryption keys securely.
- For concurrent multi-user production deployments, move conversation storage to a transactional database and use proper per-user credential encryption or a secrets manager.

A JSON archive is appropriate for a local prototype, but is not a replacement for transactional multi-user persistence.

## 11. Troubleshooting

### Python or pip cannot find a package

Check that the virtual environment is activated and that you are using Python 3.11. Inspect dependency conflicts before changing version pins.

### Vite fails to start

Check Node.js with `node --version`, then run `npm install` inside `frontend`.

### Ollama connection refused

Start Ollama and test `http://127.0.0.1:11434/api/tags`. Confirm `qwen3:4b` appears in the installed model list.

### Gemini authentication or quota error

Check the configured key, API access, billing or quota status, and selected model. A quota error is not the same as an invalid key. The application should report fallback status accurately.

### Microphone does not work

Check browser permissions, operating-system privacy settings, browser compatibility, and whether the app is served through localhost or HTTPS. Use text input while troubleshooting.

### Conversation history does not persist

Check the configured memory path, file permissions, runtime logs, and whether the application has write access to the data directory. Never fix permissions by exposing private data to all users.

## 12. Production checklist

Before exposing the application to other machines or users:

- Enable HTTPS and secure cookies.
- Replace development secrets with securely managed production secrets.
- Configure trusted frontend origins and CSRF defenses.
- Add rate limiting and account abuse protections.
- Encrypt user-specific provider credentials.
- Move conversations to a transactional database.
- Set backups and retention policies.
- Audit authorization, error handling, and provider fallback.
- Disable debug mode and protect operational logs.
- Test with multiple separate accounts.

## Project status

The README documents the intended setup. It does not claim that modules, endpoints, or tests have been implemented until their corresponding implementation phases are completed and verified.
