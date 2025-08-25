# mutasiyuk-docs
MutasiYUK-docs


## Public API (pakai API Key)

ENDPOINT:

**api.mutasiyuk.my.id**

Header:
```
x-api-key: <API_KEY>
```

- `POST /api/deposit/create` `{ nominal, kode_unik_digits?: 3 }`
- `POST /api/deposit/cancel` `{ kode_deposit }` → **Cancelled**
- `GET  /api/deposit/status/:kode`

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
    "detail_pengirim": "NOBU / GOPAY"
  }
}
```

## Webhook
User dapat men-setup `webhook_url` (Dashboard)

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

