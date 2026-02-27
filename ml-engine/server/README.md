# Server

API FastAPI exposant fonctions ML.

## Structure
```
server/
├── main.py                 # Entry point FastAPI
└── endpoints.py            # Routes API
```

## Endpoints

### POST /encode
```python
@app.post("/encode")
async def encode_batch(request: EncodeBatchRequest):
    embeddings = encoder.encode(request.texts)
    return {"embeddings": embeddings.tolist()}
```

### POST /match
```python
@app.post("/match")
async def match_texts(request: MatchRequest):
    matches = matcher.match(
        request.source_texts,
        request.target_texts,
        request.threshold
    )
    return {"matches": matches}
```

### GET /health
```python
@app.get("/health")
async def health_check():
    return {"status": "healthy", "model": "loaded"}
```

## Run Server
```bash
uvicorn server.main:app --host 0.0.0.0 --port 8001
```

## API Docs

http://localhost:8001/docs