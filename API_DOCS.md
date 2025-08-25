# MutasiYuk API Docs (Stage 4)

## Auth (JWT)
- `POST /auth/register` → `{ email, password }`
- `POST /auth/login` → `{ token }`

Header untuk semua endpoint JWT:
```
Authorization: Bearer <token>
```

## User
- `POST /user/set-credentials` (JWT)  
  Body: `{ ok_username, ok_token, qris_base, webhook_url? }`
- `POST /user/api-key/rotate` (JWT) → `{ api_key }` (tampil **sekali**)
- `GET  /user/api-key` (JWT)
- `POST /user/webhook/secret/rotate` (JWT) → `{ webhook_secret }` (tampil **sekali**)
- `GET  /user/webhook` (JWT) → info URL + createdAt

## Admin
- `GET  /admin/settings` (JWT admin)
- `POST /admin/settings` (JWT admin) `{ polling_interval_ms }`
- `POST /admin/deposit/check-now` (JWT admin) `{ kode_deposit }`

## Public API (pakai API Key)
Header:
```
x-api-key: <API_KEY>
```

- `POST /api/deposit/create` `{ nominal, kode_unik_digits?: 3 }`
- `POST /api/deposit/cancel` `{ kode_deposit }` → **Cancelled**
- `GET  /api/deposit/status/:kode`
- `GET  /api/mutasi?nominal=xxx&starttime=HH:mm` (proxy)
- `GET  /api/mutasi/history?limit=50`

### Response Format (contoh)
**Create Deposit (Success)**
```json
{
  "status": true,
  "data": {
    "kode_deposit": "9879410072",
    "metode": "QRIS",
    "nominal": 3000,
    "kode_unik": "081",
    "jumlah_transfer": 3081,
    "saldo_didapat": 3081,
    "status": "Pending",
    "starttime": "2025-02-12 22:10",
    "expired": "2025-02-12 22:20",
    "link_qr": "https://.../qris.png"
  }
}
```

**Cancel Deposit**
```json
{
  "status": true,
  "data": { "kode_deposit": "8641897556", "status": "Cancelled", "jumlah_transfer": "2827" }
}
```

**Cek Status**
```json
{
  "status": true,
  "data": {
    "kode_deposit": "8641897556",
    "status": "Success",
    "jumlah_transfer": "2827",
    "detail_pengirim": "NOBU / AZHARI ROMAHI / Bank Jago / 1anxric08582"
  }
}
```

## Webhook
User dapat men-setup `webhook_url` (Dashboard atau `POST /user/set-credentials`) dan opsional **webhook secret**.

### Header
- `x-webhook-timestamp`: detik epoch
- `x-webhook-signature`: HMAC-SHA256 dari raw body dengan secret

### Event
1. `deposit.update`
```json
{
  "event": "deposit.update",
  "data": {
    "kode_deposit": "9879410072",
    "status": "Success | Cancelled | Error",
    "jumlah_transfer": 3081
  }
}
```
2. `mutasi.new`
```json
{
  "event": "mutasi.new",
  "data": {
    "tx_id": 167129835,
    "nominal": 3081,
    "keterangan": "NOBU / GOPAY",
    "tanggal": "24/08/2025 13:22",
    "brand": { "name": "GOPAY", "logo": "https://..." }
  }
}
```

### Retry
Jika webhook gagal (non-2xx), server retry 3x (5s, 15s, 30s).

