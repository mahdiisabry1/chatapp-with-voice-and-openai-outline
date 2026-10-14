# Voice Assistant

A browser-based personal assistant built for a Coursera practice project. It accepts a typed message or microphone recording, transcribes recorded speech, generates a concise AI reply, and reads that reply aloud.

## Features

- Type a message or record one from the browser.
- Convert recorded audio to text with the Watson Speech-to-Text lab API.
- Generate short assistant responses with the OpenAI API.
- Convert replies to WAV audio with the Watson Text-to-Speech lab API.
- Choose from several English, French, and Italian voices.
- Replay original recordings or assistant responses.
- Switch between light and dark themes.

## How it works

```text
Browser
  ├─ typed message ────────────────────┐
  └─ microphone audio → /speech-to-text│ → Watson Speech-to-Text
                                        ▼
                         /process-message → OpenAI → Watson Text-to-Speech
                                        ▼
                         text reply + base64-encoded WAV audio
```

The Flask application serves the page and exposes two API endpoints:

| Endpoint | Method | Purpose |
| --- | --- | --- |
| `/` | `GET` | Serves the assistant interface. |
| `/speech-to-text` | `POST` | Sends the raw recorded audio to Speech-to-Text and returns `{"text": "..."}`. |
| `/process-message` | `POST` | Accepts `{"userMessage": "...", "voice": "..."}` and returns generated text plus base64 WAV audio. |

## Project layout

```text
server.py             Flask routes and web server
worker.py             OpenAI, speech-to-text, and text-to-speech calls
templates/index.html  Web interface
static/script.js      Recording, API calls, playback, and UI behavior
static/style.css      Light/dark theme and chat styling
models/               Optional IBM Watson local-container reference material
certs/                Lab certificate used by the supplied Docker image
```

## Prerequisites

- Python 3.10 or later
- An OpenAI API key with access to the model configured in `worker.py`
- A modern browser that supports `MediaRecorder` and grants microphone permission for voice input

The included speech endpoints point to the Coursera/Skills Network Watson lab services. They must be reachable for voice input and synthesized audio to work. The `models/` directory contains separate instructions for self-hosting compatible IBM Watson speech containers; those containers are not wired into `worker.py` by default.

## Run locally

1. Create and activate a virtual environment.

   ```bash
   python -m venv .venv
   source .venv/bin/activate
   ```

   On Windows PowerShell:

   ```powershell
   .venv\Scripts\Activate.ps1
   ```

2. Install the dependencies.

   ```bash
   pip install -r requirements.txt
   ```

3. Set your OpenAI API key for the current shell.

   ```bash
   export OPENAI_API_KEY="your_api_key_here"
   ```

   On Windows PowerShell:

   ```powershell
   $env:OPENAI_API_KEY="your_api_key_here"
   ```

4. Start the server.

   ```bash
   python server.py
   ```

5. Open [http://localhost:8000](http://localhost:8000), type a message and press Enter, or use the microphone button. Approve the browser microphone prompt before recording.

## Run with Docker

Build and run the application image:

```bash
docker build -t voice-assistant .
docker run --rm -p 8000:8000 -e OPENAI_API_KEY="your_api_key_here" voice-assistant
```

Then open [http://localhost:8000](http://localhost:8000).

> The supplied `Dockerfile` is tailored to the course lab: it includes a lab certificate and sets an example `OPENAI_API_KEY`. For a personal deployment, pass your key at runtime as shown above and do not rely on the image's default value. Never commit a real API key.

## Configuration

- **Assistant behavior:** Edit the `prompt` in `openai_process_message()` in `worker.py`.
- **OpenAI model:** Change the `model` value in `worker.py` to a model available to your API account.
- **Speech services:** The Watson service URLs and the Speech-to-Text model are defined in `worker.py`.
- **Voice menu:** Available UI voice options are in `templates/index.html`.

## Notes

- This project has no automated test suite.
- Audio recording depends on browser permissions and media-format support.
- The OpenAI key is read from `OPENAI_API_KEY` by the OpenAI Python client; keep it in your environment or secret manager, not in source control.
- See [models/README.md](models/README.md) for the optional Watson Speech-to-Text and Text-to-Speech container documentation.

## License

See [LICENSE](LICENSE).
