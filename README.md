# README.md

# Web-Based Chat Assistant with FastAPI and LiteLLM

This project implements a web-based chat assistant that allows users to have continuous conversations with language models, select different models, and upload multimedia files. The backend is built with FastAPI, and the frontend uses JavaScript to interact with the API. The application is designed to be deployed on Render.com.

## Features

- Continuous conversation sessions
- Dynamic model selection
- Optional multimedia uploads (images, audio, etc.)
- Easy deployment on Render.com

## Technologies Used

- Python 3.11+
- FastAPI
- Uvicorn
- LiteLLM
- Starlette
- Pydantic
- HTTPX
- JavaScript (for frontend UI)

## Setup Instructions

### 1. Clone the repository

```bash
git clone <repository_url>
cd <repository_directory>
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Run the server locally

```bash
uvicorn main:app --reload
```

The server will be available at `http://127.0.0.1:8000`.

### 4. Deploy on Render.com

Follow Render's deployment instructions for Python web services, specifying the command:

```bash
uvicorn main:app --host 0.0.0.0 --port 10000
```

## API Endpoints

### POST `/chat/`

Send a message and receive a response from the LLM.

**Request body:**

```json
{
  "session_id": "string",
  "message": "string",
  "model": "string",
  "media": "optional base64 string or URL"
}
```

**Response:**

```json
{
  "reply": "string",
  "conversation": [ ... ]  // conversation history
}
```

### GET `/models/`

Retrieve available models.

## Frontend UI

The frontend is a simple HTML/JavaScript interface that interacts with the backend API for a seamless chat experience.

## Files

- `main.py`: FastAPI server implementation
- `requirements.txt`: Dependencies
- `README.md`: This documentation

## License

MIT License

---

# main.py

from fastapi import FastAPI, HTTPException, UploadFile, File
from pydantic import BaseModel
from typing import List, Optional, Dict, Any
import uuid
import base64
import asyncio
import httpx
from lite_llm import LiteLLM

app = FastAPI()

# Initialize LiteLLM (assuming default configuration)
llm = LiteLLM()

# In-memory storage for conversations
conversations: Dict[str, List[Dict[str, str]]] = {}

class ChatRequest(BaseModel):
    session_id: str
    message: str
    model: Optional[str] = None
    media: Optional[str] = None  # base64 encoded media or URL

class ChatResponse(BaseModel):
    reply: str
    conversation: List[Dict[str, str]]

@app.post("/chat/", response_model=ChatResponse)
async def chat_endpoint(request: ChatRequest):
    """
    Handle chat messages, maintain conversation context, and interact with LLM.
    """
    session_id = request.session_id
    message = request.message
    model_name = request.model
    media = request.media

    # Initialize conversation history if not exists
    if session_id not in conversations:
        conversations[session_id] = []

    # Append user message
    conversations[session_id].append({"role": "user", "content": message})

    # Prepare prompt with conversation history
    prompt = ""
    for turn in conversations[session_id]:
        role = turn["role"]
        content = turn["content"]
        prompt += f"{role.capitalize()}: {content}\n"

    # Optionally handle media (not implemented here, placeholder)
    # For example, process media if provided

    # Interact with LiteLLM
    try:
        # Assuming LiteLLM has a method `generate` accepting prompt and model
        response_text = await generate_response(prompt, model_name)
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

    # Append assistant response
    conversations[session_id].append({"role": "assistant", "content": response_text})

    return ChatResponse(reply=response_text, conversation=conversations[session_id])

async def generate_response(prompt: str, model_name: Optional[str]) -> str:
    """
    Generate a response from LiteLLM based on the prompt.
    """
    # For simplicity, ignoring model selection; extend as needed
    response = await asyncio.to_thread(llm.generate, prompt)
    return response

@app.get("/models/")
async def get_models():
    """
    Return available models.
    """
    # Placeholder: return a static list or fetch from LiteLLM
    return {"models": ["lite-llm-default"]}

# Additional endpoints for media upload, model switching, etc., can be added as needed

# requirements.txt

fastapi
uvicorn
lite_llm
starlette
pydantic
httpx