# Tugas 1 — Analisis Pitfall FoodGo

**Kelompok:** [Kelompok 14]

| Nama | NIM | Kontribusi |
|---|---|---|
| [Angeli Thie] | [103072400032] | [Analisis Pitfall 1 (The Network is Reliable)] |
| [Sefia Nuraini] | [103072400043] | [pitfall/bagian yang dikerjakan] |
| [nama 3] | [nim] | [pitfall/bagian yang dikerjakan] |

## Pitfall 1: [The Network is Reliable] — ditulis oleh [Angeli Thie]

**Bukti di skenario:** [Tim menemukan bahwa kode mereka menulis asumsi seperti "# network is always reliable, no need for retry" dan tidak ada timeout sama sekali pada pemanggilan antar service (modul pesanan memanggil modul pembayaran dan menunggu tanpa batas waktu)]

**Kenapa ini keliru:** [Jaringan dalam sistem terdistribusi pada kenyataannya bersifat unreliable (tidak dapat diandalkan 100%). Koneksi dapat terputus, mengalami packet loss, high latency, atau layanan tujuan mengalami masalah sementara. Berasumsi bahwa panggilan jaringan selalu berhasil dan pasti merespons secara instan membuat aplikasi rentan mengalami hang atau deadlock]

**Dampak ke FoodGo:** [Saat modul pembayaran mengalami gangguan atau keterlambatan merespons, modul pesanan akan blocking (menunggu tanpa batas waktu/tanpa timeout). Hal ini menyebabkan pemakaian thread/connection pool terus menumpuk di modul pesanan hingga akhirnya server kehabisan resource dan crash]

**Solusi desain awal:** [
    1. Menerapkan Timeout pada setiap panggilan jaringan antar-layanan (misalnya batas maksimal 3-5 detik)
    2. Menerapkan strategi Retry with Exponential Backoff and Jitter untuk menangani kegagalan sementara (transient failure)
    3. Mengintegrasikan pola Circuit Breaker untuk memutus panggilan ke layanan pembayaran secara otomatis jika tingkat kegagalannya melampaui ambang batas tertentu
]

**Trade-off:** [Penerapan retry dapat memperparah beban pada layanan target yg sedang overloaded (cascading failure atau retry storm). Penggunaan Circuit Breaker juga berisiko menolak transaksi pengguna secara instan jika parameter threshold kurang tepat]

---

## Pitfall 2: [nama pitfall] — ditulis oleh [sefia nur aini]
**Bukti di skenario:**waktu bikin tracing order real- time front end nukis kode yang yang memanggil ApI status pemesanan tiap request langsung dianggap "instant",jadi mereja memasang polling tiap 1 detik ke service tracking tanpa mikirin delay jaringan. Di local testing semua kelihatan lancar-lancar aja karena latency-nya emang deket ke nol.

**Kenapa ini keliru:**
Dalam sistem terdistribusi yang sehat, setiap service idealnya punya batas kegagalan (failure boundary) sendiri—kalau satu service down, service lain tetap bisa jalan meski dengan fungsi terbatas (graceful degradation). Kalau semua modul digabung jadi satu unit deployment atau saling bergantung tanpa isolasi (tight coupling), maka satu titik kegagalan bisa menjatuhkan seluruh sistem sekaligus.ini bertentangan dalam prinsip dasar sistem terdistribusi yang seharus nya toleran terhadab kegagalan parsial

**Dampak ke FoodGo::** Kalau modul pembayaran crash atau restart, bukan cuma proses pembayaran yang berhenti—user jadi nggak bisa buat pesanan baru sama sekali, notifikasi ke driver juga ikut macet, padahal secara logis kedua hal itu nggak seharusnya saling menjatuhkan. Downtime yang harusnya cuma mempengaruhi satu fitur jadi melebar ke seluruh aplikasi, bikin kerugian bisnis (pesanan hilang,driver ngaggur,komplain user) jauh lebih dasar dari yang segarus nya
---
**Solusi desain awal:**
   1.Pisahkan modul jadi service independen (order service, payment service, notification service) dengan deployment dan database masing-masing, supaya kegagalan satu service nggak otomatis menjatuhkan yang lain
   2.Terapkan pola bulkhead—alokasikan resource (thread pool, connection pool) secara terpisah per dependency, supaya satu service yang bermasalah nggak menghabiskan resource yang dipakai service lain
   3.Sediakan mekanisme graceful degradation, misalnya kalau notifikasi down, order tetap bisa diproses dan notifikasi dikirim belakangan lewat antrian (queue,message broker)


**Trade-off:** Memecah monolith jadi microservices menambah kompleksitas operasional yang signifikan—butuh mekanisme komunikasi antar-service (API/message broker), monitoring terdistribusi, dan koordinasi deployment yang lebih rumit dibanding satu aplikasi tunggal. Tim juga perlu effort ekstra untuk menjaga konsistensi data antar-service (misalnya lewat pola saga untuk transaksi pesanan-pembayaran), yang jauh lebih kompleks dibanding transaksi database tunggal di arsitektur monolitik.

## Pitfall 3: [nama pitfall] — ditulis oleh [nama]

**Bukti di skenario:**Komunikasi antar-modul FoodGo (pesanan, pembayaran, notifikasi) memakai HTTP biasa tanpa enkripsi, dan modul pembayaran menerima request tanpa validasi/autentikasi asal pengirimnya.
**Kenapa ini keliru:**Jaringan internal tidak otomatis aman, apalagi di arsitektur terdistribusi/cloud yang traffic-nya bisa melewati banyak segmen jaringan. Tanpa enkripsi dan autentikasi antar-service, pihak yang menyusup ke jaringan bisa menyadap data atau menyamar sebagai service lain.
**Dampak ke FoodGo:**Data sensitif seperti info pembayaran bisa di sadap (man-in-thr-middle),dan tanpa autentifikasi antar-service,pihak tak sah bisa mengirim tequest palsu langsung ke modul pembayaran untuk membuat transaksi ilegal

## Kesimpulan Kelompok

[Ringkasan: jika FoodGo memperbaiki ketiga pitfall ini, apa arsitektur yang disarankan secara garis besar? Kaitkan dengan Tugas 2.]
