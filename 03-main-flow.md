# Main Flow

Alur utama WazzPay Billing System:

1. Merchant membuat bill
2. WazzPay generate payment link
3. Merchant mengirim link ke customer
4. Customer membuka link
5. Customer memilih metode pembayaran
6. Customer melakukan pembayaran
7. WazzPay menerima konfirmasi pembayaran
8. Status bill berubah menjadi PAID
9. WazzPay mengirim webhook ke merchant