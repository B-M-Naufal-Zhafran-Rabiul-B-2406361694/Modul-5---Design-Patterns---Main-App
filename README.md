# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

1. Pada kasus BambangShop ini, satu `struct Subscriber` sudah cukup untuk saat ini karena semua subscriber masih punya perilaku yang sama: menerima notifikasi via HTTP dengan pola payload yang sama. Dalam istilah Observer, relasi Subject-Observer tetap diterapkan, tetapi belum butuh banyak implementasi observer dengan logika `update` yang berbeda. `trait` baru menjadi penting jika nanti ada kebutuhan perilaku subscriber yang polymorphic, misalnya sebagian lewat HTTP, sebagian lewat message queue, atau punya strategi retry/fallback yang berbeda.

2. Karena `id`/`url` dimaksudkan unik, penyimpanan berbasis map lebih tepat daripada `Vec`. Jika memakai `Vec`, pengecekan keunikan harus dilakukan manual dengan scan list dulu (`O(n)`), dan operasi hapus juga linear. Dengan `DashMap<String, Subscriber>`, keunikan direpresentasikan langsung lewat key, operasi lookup/hapus mendekati `O(1)`, dan maksud desain tentang identitas unik jadi lebih jelas di struktur datanya.

3. Singleton dan thread-safe map menyelesaikan masalah yang berbeda, jadi keduanya saling melengkapi, bukan saling menggantikan. Singleton menjawab "berapa instance global yang dipakai" (satu penyimpanan subscriber bersama), sedangkan `DashMap` menjawab "apakah baca/tulis paralel aman tanpa data race." Pada web server yang menangani banyak request secara bersamaan, kita tetap butuh sinkronisasi internal yang thread-safe. Jadi desain yang tepat adalah singleton yang membungkus struktur concurrent (misalnya `DashMap` atau `RwLock<HashMap<...>>`); singleton saja tidak cukup untuk menjamin thread safety.

#### Reflection Publisher-2

1. Walaupun dalam konsep MVC klasik "Model" bisa mencakup data dan logic, pada praktik aplikasi yang mulai berkembang pemisahan `Model`, `Service`, dan `Repository` membantu menjaga desain tetap bersih. `Model` sebaiknya fokus pada representasi data/domain, `Repository` fokus pada akses dan penyimpanan data, sedangkan `Service` fokus pada aturan bisnis dan orkestrasi alur use case. Pemisahan ini mengikuti prinsip Single Responsibility dan Separation of Concerns, sehingga kode lebih mudah diuji, diubah, dan dikembangkan tanpa efek samping yang luas.

2. Jika semua hal dipaksa masuk ke `Model` saja, maka setiap model akan cepat menjadi "god object". Interaksi antar model (`Program`, `Subscriber`, `Notification`) akan saling menempel kuat: satu model harus tahu detail storage, validasi, sekaligus alur notifikasi model lain. Akibatnya kompleksitas naik, dependensi melingkar lebih mudah muncul, testing unit jadi sulit (karena harus menyiapkan banyak konteks sekaligus), dan perubahan kecil pada satu alur berpotensi merusak banyak bagian lain.

3. Ya, saya mengeksplor Postman dan tool ini sangat membantu untuk menguji endpoint tanpa membuat client manual. Untuk pekerjaan saat ini, Postman mempermudah set method/URL/header/body JSON, melihat status code dan response body dengan cepat, serta mengulang request subscribe/unsubscribe secara konsisten. Fitur yang menurut saya paling berguna untuk proyek kelompok dan proyek ke depan adalah Collection (menyimpan skenario endpoint), Environment Variables (ganti base URL/token tanpa ubah request satu per satu), Tests Script (assertion otomatis setelah request), dan Collection Runner (menjalankan rangkaian tes API secara berurutan).

#### Reflection Publisher-3

1. Pada tutorial ini, variasi Observer yang digunakan adalah **Push model**. Publisher (main app) langsung mengirim payload notifikasi ke subscriber melalui HTTP POST saat event terjadi (`CREATED`, `PROMOTION`, `DELETED`). Subscriber menerima data yang sudah disiapkan publisher (judul produk, tipe, URL produk, nama subscriber, status), bukan menarik data sendiri dari publisher.

2. Jika kita membayangkan menggunakan variasi satunya (Pull model), ada beberapa trade-off:
   - Kelebihan Pull: payload notifikasi awal bisa lebih ringan karena publisher cukup memberi sinyal/event sederhana; subscriber lebih fleksibel menentukan data apa yang ingin diambil dari publisher.
   - Kekurangan Pull: subscriber harus melakukan request tambahan ke publisher setelah menerima sinyal, sehingga total latency end-to-end bisa bertambah. Beban request ke publisher juga naik karena ada extra "follow-up fetch" dari tiap subscriber. Implementasi juga jadi lebih kompleks karena perlu desain endpoint pengambilan data yang konsisten, kontrol versi data, dan kemungkinan race condition jika data berubah di antara momen event dan momen fetch.
   - Untuk kasus tutorial ini, Push lebih cocok karena event notifikasi relatif sederhana dan datanya sudah jelas.

3. Jika proses notifikasi tidak memakai multi-threading, pengiriman notifikasi ke subscriber akan berjalan berurutan (synchronous one-by-one). Dampaknya:
   - Response endpoint utama (`create/publish/delete`) bisa menjadi lebih lambat karena harus menunggu semua request notifikasi selesai.
   - Satu subscriber yang lambat/tidak responsif dapat menahan seluruh alur, sehingga throughput aplikasi menurun.
   - Pengalaman pengguna memburuk pada beban tinggi karena request bisnis inti ikut terblokir oleh proses notifikasi.
   Multi-threading membantu memisahkan bottleneck I/O notifikasi dari alur utama, sehingga aplikasi tetap lebih responsif.
