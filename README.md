# BaobabGPT — Handoff & Deployment Runbook

**Last updated:** 2026-03-16  
**Live at:** https://starsystems.me · https://mvumi.me  
**Repo:** solomonmozark-del/mvumi-website (`main`)

---

## What Was Built (Completed)

All of the following are live in `index.html` as of commit `4708533`:

| Feature | Status |
|---|---|
| Chat UI (sidebar, history, streaming replies) | ✅ Done |
| Markdown rendering (tables, code blocks, lists) | ✅ Done |
| TF-IDF local knowledge base (150+ Q&A, Zimbabwe/Africa focus) | ✅ Done |
| `/teach`, `/forget`, `/list` knowledge commands | ✅ Done |
| Agent tool-calling (`/time`, `/date`, `/calc`, `/models`, `/tools`) | ✅ Done |
| Session memory (`/remember`, `/memories`, `/forgetmemory`) | ✅ Done |
| API key management (`/setkey`, `/clearkey`, `/keystatus`) | ✅ Done |
| Real Groq + OpenRouter API calls (via stored keys or backend proxy) | ✅ Done |
| Advanced model picker: provider filter tabs (All/Local/Groq/OpenRouter) | ✅ Done |
| Model pinning (★ favorites, persisted to `localStorage`) | ✅ Done |
| Recently used models section (top 5, persisted) | ✅ Done |
| 7 models in picker: BaobabGPT Local, Llama 3.1 8B, Llama 3.3 70B, Gemma 2 9B, Grok 3 Mini, DeepSeek R1 free, Gemini 2.0 Flash free | ✅ Done |
| Mobile responsive sidebar | ✅ Done |

---

## Architecture

```
Browser (index.html)
  │
  ├─► Local knowledge engine (TF-IDF, stemmer, synonym expansion)
  │     no network needed
  │
  ├─► Backend proxy  https://mvumi.me/api/baobab/chat
  │     tried first for Groq/OpenRouter calls
  │
  ├─► Groq API  https://api.groq.com/openai/v1/chat/completions
  │     direct fallback using key from localStorage (baobab_groq_key_v1)
  │
  └─► OpenRouter API  https://openrouter.ai/api/v1/chat/completions
        direct fallback using key from localStorage (baobab_groq_key_v1_or)
```

**Single file** — the entire frontend is `index.html`. `styles.css` is present but all critical styles are inlined in `index.html`.

---

## localStorage Keys

| Key | Purpose |
|---|---|
| `stargen_mini_model_v1` | Saved knowledge (JSON array) |
| `stargen_model_mode_v1` | Last selected model ID |
| `baobab_chats_v1` | Chat history (JSON array, max 100) |
| `baobab_active_chat_v1` | Active chat ID |
| `baobab_session_memory_v1` | Session memory notes (JSON array, max 80) |
| `baobab_groq_key_v1` | User's Groq API key |
| `baobab_groq_key_v1_or` | User's OpenRouter API key |
| `baobab_model_pins_v1` | Pinned model IDs (JSON array) |
| `baobab_model_recents_v1` | Recently used model IDs (JSON array, max 5) |
| `mvumi_session_v1` | Login session email (guards chat page access) |

---

## Model IDs

| Display name | Internal ID |
|---|---|
| BaobabGPT Local | `baobab_gpti` |
| Llama 3.1 8B (Groq) | `groq:llama-3.1-8b-instant` |
| Llama 3.3 70B (Groq) | `groq:llama-3.3-70b-versatile` |
| Gemma 2 9B (Groq) | `groq:gemma2-9b-it` |
| Grok 3 Mini (OpenRouter) | `or:x-ai/grok-3-mini` |
| DeepSeek R1 free (OpenRouter) | `or:deepseek/deepseek-r1:free` |
| Gemini 2.0 Flash free (OpenRouter) | `or:google/gemini-2.0-flash-exp:free` |

---

## Security Policy

- **Never hardcode provider API keys** in frontend source.
- Production routing should go through the backend proxy (`mvumi.me/api/baobab/chat`) so keys stay server-side.
- The `/setkey` command is a power-user escape hatch for direct API access — keys are stored in `localStorage` only, never sent anywhere except directly to the provider.
- Never upload `.git`, secret files, or backend source to any static hosting container.

---

## Deployment

### Option A — Azure App Service (primary, used now)

App Service `mvumi-site` in resource group `mvumi-rg` (South Africa North).  
Bound domains: `starsystems.me`, `www.starsystems.me`, `mvumi.me`, `www.mvumi.me`.

```powershell
cd "c:\Users\HP\Documents\BAOBAB intelligence"

# Package
Compress-Archive -Force -Path index.html, styles.css, groq_models.txt, openrouter_models.txt -DestinationPath deploy.zip

# Deploy
az webapp deploy --resource-group mvumi-rg --name mvumi-site --src-path deploy.zip --type zip

# Cleanup
Remove-Item deploy.zip
```

### Option B — GitHub (secondary, kept in sync)

```powershell
git add index.html styles.css groq_models.txt openrouter_models.txt README.md
git commit -m "your message"
git push origin main
```

GitHub Pages serves from `main` via the `CNAME` file (`starsystems.me`) as a fallback/mirror.

---

## Frontend Update Workflow

1. Edit `index.html` locally.
2. Open in browser and smoke-test (see checklist below).
3. Deploy to Azure (Option A).
4. Push to GitHub (Option B) to keep repo in sync.

---

## Smoke Test Checklist

- [ ] Page loads at https://starsystems.me
- [ ] Login/session guard redirects unauthenticated users
- [ ] Send a message → gets a reply (local engine)
- [ ] Change model to **Llama 3.1 8B** → chip updates → `/keystatus` shows key state
- [ ] `/setkey sk-...` → confirmation toast → `/keystatus` confirms
- [ ] `/calc 12 * 7` → returns `84` with tool trace card
- [ ] `/remember test note` → `/memories` lists it → `/forgetmemory 1` removes it
- [ ] `/tools` → lists all available tools
- [ ] ★ pin a model → refresh dropdown → model appears in Pinned section
- [ ] Provider filter tabs (All / Local / Groq / OpenRouter) filter correctly
- [ ] Mobile: sidebar opens/closes via hamburger
- [ ] `/teach Q => A` → `/list` shows it → ask Q → gets A

---

## Rollback

```powershell
# Revert last commit locally
git revert HEAD --no-edit
git push origin main

# Re-deploy to Azure
Compress-Archive -Force -Path index.html, styles.css, groq_models.txt, openrouter_models.txt -DestinationPath deploy.zip
az webapp deploy --resource-group mvumi-rg --name mvumi-site --src-path deploy.zip --type zip
Remove-Item deploy.zip
```

---

## Roadmap — Next Phase

| Priority | Item |
|---|---|
| 1 | Autonomous task loop (plan → act → check, bounded steps) |
| 2 | Model handoff mid-task (escalate to specialist model) |
| 3 | Backend live model catalog endpoint (dynamic picker from server) |
| 4 | Persistent memory across sessions (server-side, not just localStorage) |
| 5 | User-facing settings panel (manage keys, clear memory, export chat) |

---

## Key Files

| File | Purpose |
|---|---|
| `index.html` | Entire frontend — HTML, CSS, JS in one file |
| `styles.css` | Supplemental styles (most styles are inlined) |
| `groq_models.txt` | Baseline Groq model ID list |
| `openrouter_models.txt` | Baseline OpenRouter model ID list |
| `CNAME` | GitHub Pages custom domain (`starsystems.me`) |
| `README.md` | This file |
