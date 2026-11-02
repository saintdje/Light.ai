light-ai/
│
├── frontend/
│   ├── app/
│   │   ├── page.tsx
│   │   ├── layout.tsx
│   │   └── globals.css
│   ├── package.json
│   └── .env.local.example
│
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── config.py
│   │   └── ai_gateway.py
│   ├── requirements.txt
│   └── .env.example
│
├── infrastructure/
│   └── docker-compose.yml
│
├── documentation/
│   └── architecture.md
│
├── .gitignore
└── README.md
fastapi
uvicorn[standard]
httpx
pydantic-settings
python-dotenv

from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    app_name: str = "Light.AI"

    # Community-hosted OpenAI-compatible inference service
    ai_base_url: str = ""
    ai_api_key: str = ""
    ai_model: str = ""

    model_config = SettingsConfigDict(
        env_file=".env",
        extra="ignore"
    )


settings = Settings()

import httpx
from app.config import settings


async def generate_response(messages: list):
    if not settings.ai_base_url:
        raise RuntimeError("AI provider is not configured.")

    if not settings.ai_model:
        raise RuntimeError("AI model is not configured.")

    headers = {}

    if settings.ai_api_key:
        headers["Authorization"] = (
            f"Bearer {settings.ai_api_key}"
        )

    payload = {
        "model": settings.ai_model,
        "messages": messages,
        "temperature": 0.7
    }

    async with httpx.AsyncClient(timeout=120) as client:
        response = await client.post(
            settings.ai_base_url.rstrip("/")
            + "/chat/completions",
            headers=headers,
            json=payload
        )

        response.raise_for_status()
        data = response.json()

    return data["choices"][0]["message"]["content"]

from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

from app.ai_gateway import generate_response

app = FastAPI(title="Light.AI API")


class ChatRequest(BaseModel):
    message: str


@app.get("/api/health")
async def health():
    return {
        "status": "online",
        "application": "Light.AI",
        "version": "0.1"
    }


@app.post("/api/chat")
async def chat(request: ChatRequest):
    try:
        result = await generate_response([
            {
                "role": "system",
                "content": (
                    "You are Light.AI, a helpful "
                    "and capable AI assistant."
                )
            },
            {
                "role": "user",
                "content": request.message
            }
        ])

        return {"response": result}

    except Exception:
        raise HTTPException(
            status_code=502,
            detail="The AI provider could not complete the request."
        )
npx create-next-app@latest frontend --typescript --eslint --app
"use client";

import { useState } from "react";

export default function Home() {
  const [message, setMessage] = useState("");
  const [response, setResponse] = useState("");
  const [loading, setLoading] = useState(false);

  async function sendMessage() {
    if (!message.trim() || loading) return;

    setLoading(true);
    setResponse("");

    try {
      const result = await fetch(
        `${process.env.NEXT_PUBLIC_API_URL}/api/chat`,
        {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({ message }),
        }
      );

      if (!result.ok) {
        throw new Error("Request failed");
      }

      const data = await result.json();
      setResponse(data.response);
    } catch {
      setResponse(
        "Light.AI could not connect to its AI provider."
      );
    } finally {
      setLoading(false);
    }
  }

  return (
    <main style={{
      maxWidth: 760,
      margin: "60px auto",
      padding: 24,
      fontFamily: "Arial, sans-serif"
    }}>
      <h1>Light.AI</h1>
      <p>Your AI workspace.</p>

      <textarea
        value={message}
        onChange={(e) => setMessage(e.target.value)}
        placeholder="Ask Light.AI something..."
        rows={5}
        style={{
          width: "100%",
          padding: 14,
          boxSizing: "border-box"
        }}
      />

      <button
        onClick={sendMessage}
        disabled={loading || !message.trim()}
      >
        {loading ? "Thinking..." : "Send message"}
      </button>

      {response && (
        <section style={{ marginTop: 30 }}>
          <h2>Light.AI</h2>
          <p style={{ whiteSpace: "pre-wrap" }}>
            {response}
          </p>
        </section>
      )}
    </main>
  );
}
frontend/.env.local:NEXT_PUBLIC_API_URL=http://localhost:8000AI_BASE_URL=https://YOUR-COMMUNITY-PROVIDER/v1
AI_API_KEY=YOUR_PRIVATE_API_KEY
AI_MODEL=YOUR_MODEL_NAMEsource .venv/bin/activatepip install -r requirements.txt

uvicorn app.main:app --reload --port 8000cd light-ai/frontend

npm install
npm run de
