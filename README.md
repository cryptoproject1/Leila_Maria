# SecureChat (RSA-OAEP + AES-GCM) — with Login + DB + RSA Proof

This repo now includes:

- **User registration + password login** (JWT)
- **SQLite database** for users + stored public RSA keys
- **RSA “proof-of-possession” step**: after login, the server sends an RSA-OAEP encrypted challenge; the client must decrypt it using its private key to prove it owns the key pair.
- **WebSocket routing** of encrypted messages/files (server cannot read them)

## 1) Backend (serves the UI too — no CORS)

```bash
cd backend
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# run
cd ..
uvicorn backend.server:app --reload --host 0.0.0.0 --port 8000
```

This creates `securechat.db` (SQLite) in the backend folder.

Open the UI at:

- `http://127.0.0.1:8000/login`

Because the UI is served by the backend, you **avoid CORS** issues.

## 2) Frontend config (.env)

If you want to serve the frontend separately (e.g. another static server), set the backend base URL in:

- `frontend/.env` (copy from `frontend/.env.example`)

Then generate `frontend/env.js`:

```bash
cd frontend
cp .env.example .env
python build_env.py
```

`SECURECHAT_WS_URL` is **auto-derived** from `SECURECHAT_BACKEND_BASE_URL`.

## Flow

1) **Register** (first time): generates RSA keys in the browser (IndexedDB) and uploads only the **public key**.
2) **Login**: password login returns JWT.
3) **RSA proof** runs automatically after login (challenge/response).
4) **Lobby**: pick one user to chat with.
5) **Chat**: 1-to-1 text chat (attachments removed).

## Notes

- Only **public keys** are stored in the database.
- Messages are encrypted client-side; the server only relays ciphertext.
- For a real system, consider stronger key storage and additional hardening.
