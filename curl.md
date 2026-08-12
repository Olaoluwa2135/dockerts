# Project endpoint curl examples

This project exposes:

- Backend API on http://localhost:8000
- Frontend proxy routes on http://localhost:3000

The backend health endpoint is not under /api/v1. The record APIs are under /api/v1, while the browser-facing frontend routes are under /api.

---

## 1) Backend health check

```bash
curl -i http://localhost:8000/health
```

Expected: HTTP 200 with JSON like:

```json
{"status":"ok","database":"ok"}
```

---

## 2) Create a biodata record

```bash
curl -i -X POST http://localhost:8000/api/v1/bio-records \
  -H 'Content-Type: application/json' \
  -H 'Idempotency-Key: 12345678' \
  -d '{
    "full_name": "Ada Lovelace",
    "email": "ada@example.com",
    "phone": "+2348000000000",
    "date_of_birth": "1990-05-02",
    "sex": "female",
    "height_cm": 170.0,
    "weight_kg": 62.5,
    "blood_type": "O+",
    "allergies": "None",
    "medical_notes": "Healthy and active",
    "country": "NG"
  }'
```

Notes:
- `Idempotency-Key` is required and must be 8 to 128 characters.
- First request returns `201 Created`.
- Repeating the same request with the same key returns `200 OK` instead of creating a duplicate.
- Reusing the same key with different body data returns `409 Conflict`.

---

## 3) List biodata records

```bash
curl -i "http://localhost:8000/api/v1/bio-records?limit=20&offset=0"
```

Optional filters:

```bash
curl -i "http://localhost:8000/api/v1/bio-records?limit=10&offset=20"
```

---

## 4) Get one biodata record by ID

Replace the ID below with an actual record UUID returned from the create or list response.

```bash
curl -i http://localhost:8000/api/v1/bio-records/00000000-0000-0000-0000-000000000000
```

---

## 5) Frontend proxy: create via browser route

The frontend route is what the browser calls; it forwards to the backend.

```bash
curl -i -X POST http://localhost:3000/api/bio-records \
  -H 'Content-Type: application/json' \
  -H 'Idempotency-Key: 12345678' \
  -d '{
    "full_name": "Ada Lovelace",
    "email": "ada@example.com",
    "phone": "+2348000000000",
    "date_of_birth": "1990-05-02",
    "sex": "female",
    "height_cm": 170.0,
    "weight_kg": 62.5,
    "blood_type": "O+",
    "allergies": "None",
    "medical_notes": "Healthy and active",
    "country": "NG"
  }'
```

---

## 6) Frontend proxy: list via browser route

```bash
curl -i "http://localhost:3000/api/bio-records?limit=20&offset=0"
```

---

## 7) Repeated create with same idempotency key

This is the safe retry pattern:

```bash
curl -i -X POST http://localhost:8000/api/v1/bio-records \
  -H 'Content-Type: application/json' \
  -H 'Idempotency-Key: 12345678' \
  -d '{
    "full_name": "Ada Lovelace",
    "email": "ada@example.com",
    "phone": "+2348000000000",
    "date_of_birth": "1990-05-02",
    "sex": "female",
    "height_cm": 170.0,
    "weight_kg": 62.5,
    "blood_type": "O+",
    "allergies": "None",
    "medical_notes": "Healthy and active",
    "country": "NG"
  }'
```

Run the same command again with the same `Idempotency-Key` and identical JSON to get a replay response instead of a duplicate record.

---

## 8) Example of an invalid payload

```bash
curl -i -X POST http://localhost:8000/api/v1/bio-records \
  -H 'Content-Type: application/json' \
  -H 'Idempotency-Key: 12345678' \
  -d '{
    "full_name": "",
    "email": "not-an-email",
    "date_of_birth": "2099-01-01",
    "sex": "other",
    "height_cm": 10,
    "weight_kg": 1
  }'
```

This should return `422 Unprocessable Entity` due to validation rules.
