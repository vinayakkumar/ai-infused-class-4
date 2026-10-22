# AI Engineering · Class 3 — LangChain Agents (runnable code)

Every file here matches a code block from the Class 3 deck. They are
self-contained — run any one of them directly.

**Class materials (slides, recordings, resources):** [Google Drive](https://drive.google.com/drive/folders/1JV2aZ49WWw5nKCCwUuLWTwrkywl_mphv)

---

## One-time setup

**macOS / Linux**
```bash
# 1. Create and activate a virtual environment
python -m venv venv
source venv/bin/activate

# 2. Install dependencies
pip install -r requirements.txt

# 3. Add your Groq API key
echo "GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxx" > .env
```

**Windows (Command Prompt)**
```cmd
python -m venv venv
venv\Scripts\activate

pip install -r requirements.txt

echo GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxx > .env
```

Get a free key at https://console.groq.com/keys

### Corporate / VPN network fix (SSL error)

If you see `CERTIFICATE_VERIFY_FAILED` when running any file, your network
intercepts HTTPS traffic with a self-signed certificate. Fix it once:

**macOS**
```bash
security find-certificate -a -p /Library/Keychains/System.keychain >> $(python -c "import certifi; print(certifi.where())")
```

**Windows (PowerShell)**
```powershell
$bundle = python -c "import certifi; print(certifi.where())"
Get-ChildItem -Path Cert:\LocalMachine\Root | ForEach-Object {
    $pem = "-----BEGIN CERTIFICATE-----`n" + [Convert]::ToBase64String($_.RawData, 'InsertLineBreaks') + "`n-----END CERTIFICATE-----`n"
    Add-Content -Path $bundle -Value $pem
}
```

> **Note:** If you recreate the venv, run the fix command again.

---

## Files — what each one does

| File | What it shows | Run |
|------|---------------|-----|
| `b.py` | **Single-tool agent** — LangChain `create_agent` with one tool (`get_weather`). The simplest possible agent loop. | `python b.py` |
| `a.py` | **Multi-tool agent** — same setup but with four tools (weather, add, multiply, population). Prints every tool call and result so you can see the agent's decision loop. | `python a.py` |
| `scraper.py` | **Scraper helper** — turns any URL into plain readable text. No AI. Reused by `summarizer_langchain.py`. | `python scraper.py` |
| `summarizer_langchain.py` | **LangChain chain** — the Class 1 summarizer rebuilt using `prompt \| model \| parser`. Shows the pipe syntax. | `python summarizer_langchain.py` |
| `memory_demo.py` | **Memory (manual)** — hardcodes a chat history and passes it to the model. Demonstrates that "memory" is just re-sending past messages. | `python memory_demo.py` |
| `memory_chat.py` | **Memory (auto)** — a real chat loop that appends each turn to the history automatically. Type `quit` to exit. | `python memory_chat.py` |
| `agent.py` | **Raw agent** — same agent concept built with the plain OpenAI SDK (no LangChain). Shows the think → tool call → answer loop explicitly. | `python agent.py` |
| `app.py` | **Gradio UI** — wraps `agent.py` in a chat interface with a public shareable link. | `python app.py` |

---

## Models

All models run on the **free Groq API** — same key, same endpoint.

| File | Model | Why |
|------|-------|-----|
| `a.py`, `b.py` | `openai/gpt-oss-20b` | Agents need **tool/function calling** — the model must decide when to call a tool and parse its arguments. `openai/gpt-oss-20b` supports this on the free Groq plan. |
| `agent.py`, `app.py` | `openai/gpt-oss-20b` | Same reason — raw tool-calling loop requires a function-calling capable model. |
| `memory_chat.py`, `memory_demo.py` | `qwen/qwen3.8-27b` | Pure chat + memory, no tool calling needed. Matches the Class 1 model for consistency. |
| `summarizer_langchain.py` | `qwen/qwen3.8-27b` | Text summarization — simple text-in / text-out, no tools needed. |

---

## Notes

- `summarizer_langchain.py` imports `fetch_website_contents` from `scraper.py` — keep them in the same folder.
- `app.py` imports `agent` from `agent.py` — keep them in the same folder.
- Never commit `.env`. The included `.gitignore` already excludes it.

## Stop a running app

Press `Ctrl + C` in the terminal.
