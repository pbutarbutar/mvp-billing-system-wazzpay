# Webhook Flow

Setelah pembayaran sukses, WazzPay akan mengirim webhook ke merchant.

## Flow

1. Customer melakukan pembayaran
2. Payment provider mengirim notifikasi ke WazzPay
3. WazzPay memvalidasi pembayaran
4. Status bill berubah menjadi PAID
5. WazzPay mengirim webhook ke merchant