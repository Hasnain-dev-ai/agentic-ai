# AI / Spec-Driven Hackathon — Step-by-Step Guide (Beginner Friendly)

**Goal:** Build a Docusaurus book + embed a Retrieval-Augmented Generation (RAG) chatbot that answers questions from the book and from user-selected text. Use Claude Code, Spec-Kit Plus, FastAPI, Qdrant Cloud (Free Cluster), and OpenAI ChatKit/Agents. Deploy book on GitHub Pages and backend to any free host.

> This README is a single-file, step-by-step guide designed for complete beginners. Follow each step in order. Replace `<PLACEHOLDER>` values with your own.

---

## Table of contents

* [Before you start](#before-you-start)
* [Project layout (recommended)](#project-layout-recommended)
* [1. Prepare environment & tools](#1-prepare-environment--tools)
* [2. Join hackathon group & timing](#2-join-hackathon-group--timing)
* [3. Create the book (Docusaurus)](#3-create-the-book-docusaurus)
* [4. Qdrant Cloud: Create a Free Cluster (IMPORTANT)](#4-qdrant-cloud-create-a-free-cluster-important)
* [5. Build FastAPI RAG backend (create collection from code)](#5-build-fastapi-rag-backend-create-collection-from-code)
* [6. Create embeddings & upload to Qdrant](#6-create-embeddings--upload-to-qdrant)
* [7. Integrate Chatbot into Docusaurus (UI + selected-text feature)](#7-integrate-chatbot-into-docusaurus-ui--selected-text-feature)
* [8. Deploy backend and book](#8-deploy-backend-and-book)
* [9. Submission checklist & presentation](#9-submission-checklist--presentation)
* [10. Troubleshooting & tips](#10-troubleshooting--tips)
* [Appendix: useful snippets & placeholders](#appendix-useful-snippets--placeholders)

---

## Before you start

Do these now (essential):

* Join WhatsApp group (book title announced at hackathon start): `https://chat.whatsapp.com/BRmVNFadm4IE33uwOL1Q5P`
* Install: Node.js (LTS), Git, Python 3.10+, VS Code (optional)
* Create accounts: GitHub, OpenAI (for ChatKit/Agents), Qdrant Cloud
* Install Claude Code + Spec-Kit Plus (VSCode extensions or follow docs)

---

## Project layout (recommended)

```
hackathon-project/
├── book/           # Docusaurus project (frontend)
├── rag-backend/    # FastAPI + Qdrant + embedding scripts (backend)
└── agents/         # Claude Code subagents & skills (optional, bonus)
```

---

## 1. Prepare environment & tools

**Install basic tools**

* Node.js: [https://nodejs.org](https://nodejs.org)
* Git: [https://git-scm.com](https://git-scm.com)
* Python: [https://python.org](https://python.org)
* VS Code: [https://code.visualstudio.com](https://code.visualstudio.com)

**Python packages (backend)**

```bash
pip install fastapi uvicorn openai qdrant-client python-dotenv
```

**Docusaurus (book)**
No global install needed — use `npx`.

---

## 2. Join hackathon group & timing

* WhatsApp group (updates + book title): `https://chat.whatsapp.com/BRmVNFadm4IE33uwOL1Q5P`
* Book title announced: **Nov 30 at 9:00 AM** (local event time)
* Submission form (use at end): `https://forms.gle/CQsSEGM3GeCrL43c8`
* Presentation Zoom (6:00 PM Nov 30): link provided in hackathon materials

---

## 3. Create the Book (Docusaurus) — Baby steps

1. Create project folder and Docusaurus skeleton:

   ```bash
   mkdir hackathon-project
   cd hackathon-project
   npx create-docusaurus@latest book classic
   cd book
   npm install
   npm run start
   ```

   * Open `http://localhost:3000` to preview.

2. Use Claude Code + Spec-Kit Plus to produce an outline & Markdown content:

   * Ask Claude Code to produce chapters, sections, and Markdown files for the announced title.
   * Save generated Markdown files into `book/docs/` (e.g. `intro.md`, `chapter1.md`, ...).

3. Edit `sidebars.js` or Docusaurus config to include docs.

4. Set deployment config in `docusaurus.config.js`:

   ```js
   url: 'https://<your-username>.github.io',
   baseUrl: '/<repo-name>/',
   organizationName: '<your-username>',
   projectName: '<repo-name>',
   ```

5. Push to GitHub:

   ```bash
   git init
   git add .
   git commit -m "initial book"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<repo>.git
   git push -u origin main
   ```

6. Deploy to GitHub Pages:

   ```bash
   GIT_USER=<your-username> npm run deploy
   ```

   * Live URL: `https://<your-username>.github.io/<repo>/`

---

## 4. Qdrant Cloud: Create a Free Cluster (IMPORTANT)

**Create a *Cluster*, not a collection**:

1. Visit: [https://cloud.qdrant.io/](https://cloud.qdrant.io/) and sign up.
2. Click **Create Cluster** → choose **Free Tier** → choose region → name it (e.g. `hackathon-rag-cluster`).
3. After deploy, copy:

   * `QDRANT_URL` (something like `https://<id>-<region>.qdrant.cloud`)
   * `QDRANT_API_KEY`

**Note:** Do **not** create collections in dashboard. Collections are created from your backend code (see next section).

---

## 5. Build FastAPI RAG backend (create collection from code)

Create `rag-backend/` folder and files.

**Install deps**

```bash
cd rag-backend
python -m venv .venv
source .venv/bin/activate   # mac/linux
# or .venv\Scripts\activate on Windows
pip install fastapi uvicorn openai qdrant-client python-dotenv
```

**Create `.env`**

```
QDRANT_URL=<your-qdrant-cluster-url>
QDRANT_API_KEY=<your-qdrant-api-key>
OPENAI_API_KEY=<your-openai-api-key>
```

**`main.py` (minimal)**

```python
from fastapi import FastAPI
from qdrant_client import QdrantClient
from qdrant_client.http import models
import os
from dotenv import load_dotenv

load_dotenv()
QDRANT_URL = os.getenv("QDRANT_URL")
QDRANT_API_KEY = os.getenv("QDRANT_API_KEY")

app = FastAPI()

qdrant = QdrantClient(url=QDRANT_URL, api_key=QDRANT_API_KEY)

# Create (or recreate) a collection from code
qdrant.recreate_collection(
    collection_name="book_vectors",
    vectors_config=models.VectorParams(size=1536, distance="Cosine")
)

@app.get("/")
def ping():
    return {"status": "RAG backend running"}
```

**Run locally**

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

* This call creates the `book_vectors` collection inside your Qdrant cluster programmatically.

---

## 6. Create embeddings & upload to Qdrant

Place the book `.md` files into `rag-backend/book_text/` (or read from `book/docs/`).

**`embed.py`**

```python
import glob, os
from openai import OpenAI
from qdrant_client import QdrantClient, models
from dotenv import load_dotenv

load_dotenv()
QDRANT_URL = os.getenv("QDRANT_URL")
QDRANT_API_KEY = os.getenv("QDRANT_API_KEY")
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")

openai = OpenAI(api_key=OPENAI_API_KEY)
qdrant = QdrantClient(url=QDRANT_URL, api_key=QDRANT_API_KEY)

collection = "book_vectors"
files = glob.glob("book_text/*.md")
points = []
pid = 1

for file in files:
    text = open(file, "r", encoding="utf-8").read()
    # OPTIONAL: chunk text if very large (recommended)
    resp = openai.embeddings.create(model="text-embedding-3-small", input=text)
    vector = resp.data[0].embedding
    points.append(models.PointStruct(id=pid, vector=vector, payload={"text": text, "source": file}))
    pid += 1

# upload to Qdrant
qdrant.upsert(collection_name=collection, points=points)
print(f"Uploaded {len(points)} points")
```

**Run**

```bash
python embed.py
```

**Important tips**

* If text is large, split into chunks (e.g., 500–1000 tokens) and upload each chunk as a vector with `chapter`, `start`, `end` metadata.
* Use an embedding model that matches the vector size configured in your collection (e.g., 1536).

---

## 7. Integrate Chatbot into Docusaurus (UI + selected-text feature)

You have two parts:

1. Chat UI (frontend inside Docusaurus)
2. Backend endpoint that accepts `question` and optional `context` (selected text)

### Backend endpoint for chat (add to `main.py`)

```python
from fastapi import Body
from openai import OpenAI

openai = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

@app.post("/chat")
def chat_endpoint(question: str = Body(...), context: str = Body("")):
    # If context (selected text) is given, the backend should prioritize that.
    if context:
        system_prompt = "Answer ONLY using the given context. If the context doesn't contain the answer say you don't know."
        user_prompt = f"Context:\n{context}\n\nQuestion: {question}"
    else:
        # Do retrieval from Qdrant
        hits = qdrant.search(collection_name="book_vectors", query_vector=openai.embeddings.create(model="text-embedding-3-small", input=question).data[0].embedding, limit=4)
        retrieved_text = "\n\n".join([h.payload['text'] for h in hits])
        system_prompt = "Use retrieved book content to answer the user's question."
        user_prompt = f"Retrieved:\n{retrieved_text}\n\nQuestion: {question}"
    response = openai.chat.completions.create(model="gpt-4o-mini", messages=[
        {"role":"system","content":system_prompt},
        {"role":"user","content":user_prompt}
    ])
    answer = response.choices[0].message["content"]
    return {"answer": answer}
```

### Frontend: Docusaurus component (React)

Create `book/src/components/Chatbot.jsx`:

```jsx
import React, {useState} from 'react';

export default function Chatbot() {
  const [question, setQuestion] = useState('');
  const [answer, setAnswer] = useState('');

  async function ask(selectedContext="") {
    const res = await fetch("https://<your-backend-url>/chat", {
      method: "POST",
      headers: {"Content-Type":"application/json"},
      body: JSON.stringify({question, context: selectedContext})
    });
    const data = await res.json();
    setAnswer(data.answer);
  }

  function getSelection() {
    const s = window.getSelection().toString();
    return s || "";
  }

  return (
    <div style={{border:"1px solid #ddd", padding:16, borderRadius:8}}>
      <textarea value={question} onChange={e=>setQuestion(e.target.value)} rows={3} style={{width:"100%"}} />
      <div style={{marginTop:8}}>
        <button onClick={()=>ask("")}>Ask (entire book)</button>
        <button onClick={()=>ask(getSelection())} style={{marginLeft:8}}>Ask selected text only</button>
      </div>
      <div style={{marginTop:12, whiteSpace:"pre-wrap"}}>{answer}</div>
    </div>
  );
}
```

* Import the component into any doc page or custom page:

  ```jsx
  import Chatbot from '@site/src/components/Chatbot';
  <Chatbot />
  ```

**Security note**: Do not expose OpenAI keys in frontend. Frontend must call your FastAPI backend which calls OpenAI.

---

## 8. Deploy backend and book

**Backend deploy options (free tiers):**

* Railway (railway.app)
* Render (render.com)
* Deta (deta.sh) — small apps only
* Fly (free tier) or any provider you prefer

Deploy steps (general):

1. Push `rag-backend` to GitHub.
2. Connect repo to chosen host.
3. Set environment variables (`QDRANT_URL`, `QDRANT_API_KEY`, `OPENAI_API_KEY`).
4. Deploy and copy the public URL. Use this URL in the Docusaurus Chatbot component.

**Book deployment**

* GitHub Pages via `npm run deploy` (configured in `docusaurus.config.js`)
* Or host elsewhere (Netlify, Vercel) — but GitHub Pages is simplest for this hackathon.

---

## 9. Submission checklist & presentation

**Before submission (essential)**

* [ ] GitHub repo for `book/` (link)
* [ ] Live GitHub Pages URL (book)
* [ ] GitHub repo for `rag-backend/`
* [ ] Backend live URL
* [ ] Demonstration that:

  * Chatbot answers book questions
  * Chatbot answers from selected text only
* [ ] (Optional) `agents/` with Claude Subagents & Skills for bonus marks

**Submit project (mandatory)**

* Submission form: `https://forms.gle/CQsSEGM3GeCrL43c8`

**Presentation**

* Join Zoom (6:00 PM Nov 30) — provide live demo and answer judges' questions.

---

## 10. Troubleshooting & tips

* **Qdrant errors**: Confirm `QDRANT_URL` and `QDRANT_API_KEY` are correct. Ensure collection name matches code.
* **Embedding mismatched size**: Make sure the embedding model vector size matches the `VectorParams.size` set when creating collection (e.g., 1536).
* **Large files**: Chunk long chapters into 500–1000 token pieces before embedding.
* **CORS**: If front-end calls backend from GitHub Pages, add proper CORS to FastAPI (`fastapi.middleware.cors`).
* **Rate limits**: OpenAI / embedding calls may be rate-limited. Batch embeddings when possible.
* **Keep keys secret**: Store keys server-side; do not commit `.env` to GitHub.

---

## Appendix: useful snippets & placeholders

**Environment variables (.env)**

```
QDRANT_URL=https://<id>-<region>.qdrant.cloud
QDRANT_API_KEY=<qdrant-api-key>
OPENAI_API_KEY=<openai-api-key>
```

**Create collection from code (Qdrant)**

```python
qdrant.recreate_collection(
    collection_name="book_vectors",
    vectors_config=models.VectorParams(size=1536, distance="Cosine")
)
```

**Search example (Qdrant)**

```python
hits = qdrant.search(
    collection_name="book_vectors",
    query_vector=query_vector,
    limit=4
)
```

**Chunking idea (pseudo)**

* Split chapter text into ~800-character or ~500-token chunks
* Keep `source`, `chapter`, `start_offset` metadata in payload

---

## Final notes

* The book title will be given at hackathon start — use it exactly.
* Collections are created in code (not dashboard) — **remember this key update**.
* Bonus: implement Claude Code Subagents and Skills for reusable intelligence (extra marks).

---
