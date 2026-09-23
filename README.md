# PitchPlay — Private Fantasy Contest Platform

Admin-managed private contest platform. Admin posts contests hosted on any external app (link + entry fee + prize pool); users pay via UPI, upload the payment screenshot, get the play link after admin approval, and withdraw winnings from their wallet — all approved manually by the admin.

## Features

**Users**
- Signup / login with mobile number + password (no OTP)
- Browse contests with live match countdown
- Pay entry fee via UPI (QR code, copy UPI ID, one-tap `upi://pay` link), upload screenshot + UTR
- Play link unlocked on the card after admin approval
- Wallet with full transaction history, withdrawal requests to own UPI
- Recent winners board, WhatsApp share for contests

**Admin** (only role that can see mobile numbers)
- Create / edit / close / delete contests (title, link, fee, prize, max players, match time)
- Review payment screenshots → approve / reject entries
- Declare winners → prize credited to user wallet
- Process withdrawals → mark paid / reject (auto-refund)
- Manage users: add, remove, block/unblock, credit/debit wallet, search by mobile
- Payment settings: UPI ID, payee name, instructions, upload custom QR image

## Tech stack

| Layer    | Tech                                                        |
|----------|-------------------------------------------------------------|
| Frontend | React 19, Tailwind CSS, shadcn/ui, Phosphor icons, qrcode.react |
| Backend  | FastAPI, Motor (async MongoDB), PyJWT, bcrypt               |
| Database | MongoDB                                                     |
| Storage  | Emergent Object Storage (payment screenshots, QR image)     |

## Project structure

```
backend/
  server.py          # all API routes (prefix /api)
  requirements.txt
  .env               # see below (not committed)
frontend/
  src/pages/Landing.jsx   # login / signup
  src/pages/UserApp.jsx   # user dashboard
  src/pages/AdminApp.jsx  # admin console
  src/lib/api.js, auth.jsx
  .env               # see below (not committed)
memory/
  PRD.md, test_credentials.md
```

## Environment variables

`backend/.env`
```
MONGO_URL=mongodb://localhost:27017
DB_NAME=pitchplay
CORS_ORIGINS=*
JWT_SECRET=<long-random-string>
ADMIN_MOBILE=<admin mobile>
ADMIN_PASSWORD=<admin password>
ADMIN_UPI_ID=<default upi id>
APP_NAME=fantasy-contest
EMERGENT_LLM_KEY=<key used for object storage>
```

`frontend/.env`
```
REACT_APP_BACKEND_URL=http://localhost:8001
```

The admin account is auto-seeded on backend startup from `ADMIN_MOBILE` / `ADMIN_PASSWORD`.

## Run locally

```bash
# backend
cd backend
pip install -r requirements.txt
uvicorn server:app --host 0.0.0.0 --port 8001 --reload

# frontend
cd frontend
yarn install
yarn start        # http://localhost:3000
```

## API overview (all under `/api`)

| Area       | Endpoints |
|------------|-----------|
| Auth       | `POST /auth/signup`, `POST /auth/login`, `GET /auth/me` |
| Contests   | `GET /contests`, `POST /contests`, `PATCH /contests/{id}`, `DELETE /contests/{id}` |
| Entries    | `POST /entries` (multipart screenshot), `GET /entries/mine`, `GET /entries`, `POST /entries/{id}/decision`, `POST /entries/{id}/declare-winner` |
| Wallet     | `GET /wallet/config`, `GET /wallet/history`, `GET /winners` |
| Withdrawals| `POST /withdrawals`, `GET /withdrawals/mine`, `GET /withdrawals`, `POST /withdrawals/{id}/decision` |
| Admin users| `GET/POST /admin/users`, `DELETE /admin/users/{id}`, `POST /admin/users/{id}/block`, `POST /admin/users/{id}/wallet` |
| Settings   | `GET/PUT /admin/payment-settings`, `POST/DELETE /admin/payment-settings/qr`, `GET /admin/stats` |
| Files      | `GET /files?path=` (auth required) |

## Payment flow

1. User opens a contest → sees UPI ID / QR → pays in any UPI app.
2. User uploads screenshot (+ optional UTR) → entry is **pending**.
3. Admin reviews screenshot → **approve** (play link unlocked) or **reject**.
4. After the match, admin **declares winner** with prize → wallet credited.
5. User requests **withdrawal** → admin pays to user's UPI → marks **paid** (reject refunds wallet).
