# 🚀 Claude Code + Gemini — Complete Windows Setup Guide

A simple and professional guide for **developers, students, and AI enthusiasts** to integrate **Claude Code** with **Google Gemini models** on Windows using:

- `claude-code`
- `claude-code-router`

This setup enables smooth coding assistance, AI-powered workflows, and lightweight local development.

---

## 🧩 Step 0 — Check Your Node.js Installation

Open **PowerShell** and run:

```bash
node --version
````

✔ Version should be **18+**

If not installed, download the latest version:
👉 [https://nodejs.org](https://nodejs.org)

---

## 🔑 Step 1 — Generate Your Google API Key

1. Go to: [https://aistudio.google.com](https://aistudio.google.com)
2. Click **Get API Key**
3. Click **Create API Key**
4. Copy your key (example):

```
AIzaSy************
```

---

## 🛠 Step 2 — Install Required Global Packages

Run this in **PowerShell (Admin Mode)**:

```bash
npm install -g @anthropic-ai/claude-code @musistudio/claude-code-router
```

---

## 📁 Step 3 — Create Configuration Directories

Open a normal PowerShell window:

```bash
mkdir $HOME/.claude-code-router
mkdir $HOME/.claude
```

---

## 📄 Step 4 — Create the `config.json` File

Windows does not support `cat <<EOF` syntax, so we use Notepad.

```bash
notepad $HOME/.claude-code-router/config.json
```

Paste the following JSON:

```json
{
  "LOG": true,
  "LOG_LEVEL": "info",
  "HOST": "127.0.0.1",
  "PORT": 3456,
  "API_TIMEOUT_MS": 600000,
  "Providers": [
    {
      "name": "gemini",
      "api_base_url": "https://generativelanguage.googleapis.com/v1beta/models/",
      "api_key": "$GOOGLE_API_KEY",
      "models": [
        "gemini-2.5-flash",
        "gemini-2.0-flash"
      ],
      "transformer": {
        "use": ["gemini"]
      }
    }
  ],
  "Router": {
    "default": "gemini,gemini-2.5-flash",
    "background": "gemini,gemini-2.5-flash",
    "think": "gemini,gemini-2.5-flash",
    "longContext": "gemini,gemini-2.5-flash",
    "longContextThreshold": 60000
  }
}
```

Save → close Notepad.

---

## 🔐 Step 5 — Add Your Google API Key to Environment Variables

Run **PowerShell as Administrator**:

```powershell
[System.Environment]::SetEnvironmentVariable('GOOGLE_API_KEY', 'YOUR_API_KEY_HERE', 'User')
```

Example:

```powershell
[System.Environment]::SetEnvironmentVariable('GOOGLE_API_KEY', 'AIzaSyXXXX...', 'User')
```

### ✔ Verify Key Loaded

Close PowerShell → reopen → run:

```bash
echo $env:GOOGLE_API_KEY
```

If the key prints out → you're good to go.

---

## 🧪 Step 6 — Validate Tool Installation

Run:

```bash
claude --version
ccr version
echo $env:GOOGLE_API_KEY
```

All three should display output correctly.

---

## 💼 Step 7 — Your Daily Usage Workflow

### 🖥 Terminal 1 — Start the Router

```bash
ccr start
```

Wait for:

```
✔ Service started successfully
```

### 🖥 Terminal 2 — Launch Claude Code

Navigate into your project:

```bash
cd your-project-folder
ccr code
```

or activate it globally:

```bash
eval "$(ccr activate)"
claude
```

---

## 🧷 Step 8 — Quick Functionality Test

Run:

```bash
ccr code
```

Then type:

```
Assalamu alaikum
```

If Claude responds → 🎉 **SUCCESS! Claude Code + Gemini fully working.**

---

## ⭐ Support & Contributions

If you found this guide helpful:

* ⭐ Star this repository
* 🔁 Share with developers & students
* 📝 Create issues or pull requests for improvements

---

## 📜 License

Just tell me!
```

