# WazzPay Billing System

## Gambaran Singkat
WazzPay Billing System adalah sistem penagihan di mana **WazzPay berperan sebagai biller** untuk para merchant. Setiap bisnis yang sudah terdaftar sebagai **Merchant WazzPay** dapat membuat **bill/invoice** melalui:

- **WazzPay Open API B2B**
- **Dashboard B2B WazzPay**

Setelah invoice dibuat, WazzPay akan menyediakan **payment link** yang dapat diarahkan ke:

- **WhatsApp**
- **Aplikasi Android WazzPay**

End user merchant dapat melakukan pembayaran melalui channel yang tersedia:

- **QRIS**
- **Virtual Account (VA)**
- **DANA**
- **OVO**

Setelah pembayaran berhasil, WazzPay akan mengirimkan **notifikasi callback/webhook** ke sistem merchant.

---

# Actor

## 1. Merchant WazzPay
Pihak bisnis atau perusahaan yang menggunakan WazzPay untuk membuat tagihan kepada pelanggan mereka.

## 2. End User Merchant
Pelanggan dari merchant yang menerima tagihan dan melakukan pembayaran.

## 3. WazzPay Open API B2B
Layanan API yang digunakan merchant untuk integrasi pembuatan invoice, cek status pembayaran, dan menerima webhook pembayaran.

---

# Tujuan Sistem

- Memudahkan merchant membuat tagihan secara cepat
- Menyediakan banyak channel pembayaran dalam satu sistem
- Memudahkan end user membayar tagihan melalui link pembayaran
- Memberikan notifikasi otomatis ke merchant setelah pembayaran berhasil
- Menjadikan WazzPay sebagai pusat billing dan payment collection untuk berbagai jenis bisnis

---

# Struktur Sistem WazzPay Billing

```text
+-------------------------+
|     Merchant WazzPay    |
|-------------------------|
| - Dashboard B2B         |
| - Sistem Merchant       |
| - Integrasi Open API    |
+-----------+-------------+
            |
            | Create Bill / Check Status
            v
+-------------------------+
|   WazzPay Open API B2B  |
|-------------------------|
| - Auth/API Key          |
| - Create Invoice        |
| - Inquiry Invoice       |
| - Payment Link          |
| - Webhook Notification  |
+-----------+-------------+
            |
            | Generate Billing & Payment Session
            v
+-------------------------+
|   WazzPay Billing Core  |
|-------------------------|
| - Invoice Management    |
| - Payment Routing       |
| - Status Payment        |
| - Callback Engine       |
+-----------+-------------+
            |
            | Payment Channel
            v
+----------------------------------------------+
|               Payment Channels                |
|----------------------------------------------|
| QRIS | VA | DANA | OVO                        |
+-----------+----------------------------------+
            |
            | Paid by Customer
            v
+-------------------------+
|   End User Merchant     |
|-------------------------|
| - Buka payment link     |
| - Pilih channel bayar   |
| - Selesaikan pembayaran |
+-------------------------+
```

---

# Struktur Modul yang Mudah Dipahami

## A. Merchant Layer
Bagian yang dipakai merchant untuk membuat dan memantau tagihan.

### Komponen:
- Dashboard B2B WazzPay
- Integrasi Open API B2B
- Manajemen customer/tagihan
- Monitoring status invoice

## B. WazzPay API Layer
Bagian penghubung antara merchant dan sistem billing WazzPay.

### Komponen:
- Autentikasi merchant
- Endpoint create bill/invoice
- Endpoint inquiry/check status
- Endpoint daftar payment channel
- Endpoint webhook callback

## C. Billing Core
Bagian inti untuk mengelola seluruh transaksi tagihan.

### Komponen:
- Generate invoice
- Generate payment link
- Menyimpan status tagihan
- Mapping transaksi merchant
- Validasi pembayaran
- Trigger webhook ke merchant

## D. Payment Processing Layer
Bagian yang menangani pembayaran dari berbagai channel.

### Channel:
- QRIS
- VA
- DANA
- OVO

## E. Notification Layer
Bagian yang mengirim notifikasi status pembayaran ke merchant.

### Bentuk notifikasi:
- Webhook paid
- Webhook expired
- Webhook failed
- Optional dashboard status update

---

# Flow Utama Sistem

## Flow 1 — Merchant Registrasi dan Aktif
1. Bisnis mendaftar sebagai **Merchant WazzPay**.
2. WazzPay melakukan proses aktivasi merchant.
3. Merchant mendapatkan akses ke:
   - Dashboard B2B
   - API credential / API key / secret
4. Merchant siap membuat invoice.

---

## Flow 2 — Merchant Membuat Bill/Invoice
Merchant dapat membuat tagihan melalui 2 cara:

### Opsi A: Melalui Dashboard B2B
1. Merchant login ke Dashboard B2B WazzPay.
2. Merchant input data tagihan:
   - nomor invoice
   - nama customer
   - nominal
   - deskripsi tagihan
   - expired time
3. Merchant klik **create bill**.
4. WazzPay membuat invoice dan payment link.
5. Invoice tersimpan di sistem WazzPay.

### Opsi B: Melalui Open API B2B
1. Sistem merchant mengirim request ke **WazzPay Open API B2B**.
2. Request berisi data invoice/customer.
3. WazzPay memvalidasi request.
4. Jika valid, WazzPay membuat invoice.
5. WazzPay mengembalikan response:
   - invoice id
   - external id
   - amount
   - payment link
   - status invoice
   - expired time

---

## Flow 3 — Payment Link Dikirim ke End User
1. Setelah invoice berhasil dibuat, WazzPay menghasilkan **payment link**.
2. Payment link dapat:
   - dibagikan oleh merchant ke customer
   - redirect ke WhatsApp
   - dibuka melalui aplikasi Android WazzPay
3. End user membuka payment link.
4. Sistem menampilkan detail invoice dan pilihan channel pembayaran.

---

## Flow 4 — End User Melakukan Pembayaran
1. End user memilih salah satu channel pembayaran:
   - QRIS
   - VA
   - DANA
   - OVO
2. Sistem WazzPay membuat sesi pembayaran.
3. End user menyelesaikan pembayaran.
4. Payment channel mengirim status hasil pembayaran ke WazzPay.
5. WazzPay mengubah status invoice menjadi:
   - **PAID** jika berhasil
   - **FAILED** jika gagal
   - **EXPIRED** jika lewat waktu

---

## Flow 5 — Webhook ke Merchant Setelah Pembayaran
1. Setelah invoice berhasil dibayar, WazzPay menjalankan proses webhook.
2. WazzPay mengirim notifikasi ke URL webhook merchant.
3. Data webhook minimal berisi:
   - invoice id
   - external id
   - amount paid
   - payment channel
   - paid time
   - status = PAID
4. Sistem merchant menerima webhook.
5. Merchant memperbarui status transaksi di sistem internal mereka.

---

# Flow Sederhana End-to-End

```text
Merchant WazzPay
   |
   | 1. Create Bill/Invoice
   |    - via Dashboard B2B
   |    - via Open API B2B
   v
WazzPay Billing System
   |
   | 2. Generate Invoice + Payment Link
   v
End User Merchant
   |
   | 3. Open Payment Link
   | 4. Choose Payment Channel
   | 5. Make Payment
   v
Payment Channel (QRIS / VA / DANA / OVO)
   |
   | 6. Send Payment Result
   v
WazzPay Billing System
   |
   | 7. Update Invoice Status
   | 8. Trigger Webhook
   v
Merchant WazzPay
   |
   | 9. Receive Payment Notification
   | 10. Mark Transaction as Paid
```

---

# Flow Sequence yang Sangat Mudah Dipahami

## Sequence 1 — Create Invoice sampai Paid

```text
Merchant            WazzPay API/Billing         End User           Payment Channel
   |                       |                       |                      |
   |--- Create Invoice --->|                       |                      |
   |<-- Invoice + Link ----|                       |                      |
   |                       |--- Show Link -------->|                      |
   |                       |                       |--- Choose Pay ------>|
   |                       |                       |<-- Payment Process --|
   |                       |<-- Payment Result ----|                      |
   |<-- Webhook Paid ------|                       |                      |
```

---

# Status Invoice yang Disarankan

Agar sederhana dan mudah dipahami, gunakan status berikut:

- **PENDING** → invoice berhasil dibuat, belum dibayar
- **PAID** → pembayaran berhasil
- **FAILED** → pembayaran gagal
- **EXPIRED** → invoice melewati batas waktu pembayaran
- **CANCELLED** → invoice dibatalkan

---

# Data Utama dalam Billing System

## Data Merchant
- merchant_id
- merchant_name
- api_key
- api_secret
- callback_url
- status_active

## Data Invoice
- invoice_id
- external_id
- merchant_id
- customer_name
- amount
- description
- payment_link
- expired_at
- status
- created_at
- updated_at

## Data Payment
- payment_id
- invoice_id
- payment_channel
- amount_paid
- payment_time
- payment_reference
- payment_status

## Data Webhook Log
- webhook_id
- invoice_id
- merchant_id
- callback_url
- payload
- response_status
- retry_count
- sent_at

---

# Struktur Folder/Modul Sistem (Friendly Version)

```text
wazzpay-billing-system/
├── docs/
│   └── billing-flow.md
├── merchant/
│   ├── dashboard/
│   ├── invoice/
│   └── customer/
├── openapi/
│   ├── auth/
│   ├── invoice/
│   ├── payment/
│   └── webhook/
├── billing-core/
│   ├── invoice-engine/
│   ├── payment-link/
│   ├── status-manager/
│   └── callback-engine/
├── payment-channel/
│   ├── qris/
│   ├── va/
│   ├── dana/
│   └── ovo/
├── notifications/
│   ├── webhook/
│   └── logs/
└── shared/
    ├── config/
    ├── utils/
    ├── models/
    └── constants/
```

---

# Alur Bisnis Sederhana

## Dari sudut pandang Merchant
- Saya daftar menjadi merchant WazzPay
- Saya membuat invoice
- Saya kirim payment link ke customer
- Customer bayar
- Saya menerima webhook dari WazzPay
- Transaksi saya selesai

## Dari sudut pandang End User
- Saya menerima link pembayaran
- Saya buka link
- Saya pilih metode bayar
- Saya bayar
- Pembayaran selesai

## Dari sudut pandang WazzPay
- Menerima request invoice dari merchant
- Membuat invoice dan payment link
- Menyediakan channel pembayaran
- Menerima hasil pembayaran
- Mengubah status invoice
- Mengirim webhook ke merchant

---

# Rekomendasi Endpoint API Sederhana

## Merchant / Invoice
- `POST /v1/invoices` → create invoice
- `GET /v1/invoices/{invoice_id}` → detail invoice
- `GET /v1/invoices` → list invoice
- `POST /v1/invoices/{invoice_id}/cancel` → cancel invoice

## Payment
- `GET /v1/payment-channels` → list channel pembayaran
- `GET /v1/payment-link/{invoice_id}` → ambil payment link

## Webhook
- `POST /merchant/webhook/payment` → endpoint callback di sisi merchant

---

# Contoh Alur Paling Ringkas

```text
1. Merchant create invoice
2. WazzPay generate payment link
3. Link dikirim ke customer
4. Customer bayar via QRIS / VA / DANA / OVO
5. WazzPay menerima status paid
6. WazzPay trigger webhook ke merchant
7. Merchant update status transaksi
```

---

# Kesimpulan

WazzPay Billing System dibuat agar **mudah dipakai merchant**, **mudah dibayar oleh customer**, dan **mudah diintegrasikan lewat API**.

Inti alurnya sangat sederhana:

- Merchant membuat tagihan
- WazzPay membuat payment link
- End user membayar
- WazzPay mengirim webhook ke merchant

Dengan struktur ini, WazzPay dapat digunakan oleh berbagai jenis bisnis sebagai **biller modern yang simple, friendly, dan scalable**.
