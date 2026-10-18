Konfigurasi yang dipakai:
-	Partition: 3
-	Consumer group: dpw3-case
-	Consumer: 2
-	Event key: order_id
-	Offset reset: earliest
-	Commit: manual, setelah processing berhasil
-	Delivery: at-least-once
File starter producer.py sudah menggunakan order_id sebagai key, dan consumer.py membaca pengaturan group dari environment variable. Konfigurasi ini juga sudah dicatat di config.py.
