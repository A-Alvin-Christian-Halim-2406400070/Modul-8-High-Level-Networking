# Modul-8-High-Level-Networking


> What are the key differences between unary, server streaming, and bi-directional streaming RPC (Remote Procedure Call) methods, and in what scenarios would each be most suitable?

- Unary RPC menggunakan 1 request dan 1 response. Metode ini biasa diguakan untuk operasi CRUD atau auth.
- Server Streaming RPC menggunakan 1 request namun memungkinkan server untuk mengirim banyak response secara bertahap. Metode ini biasa digunakan untuk data yang besar atau yang dihasilkan secara bertahap contohnya monitoring real time.
- Bi-directional Streaming RPC memungkinkan client dan server untuk saling mengirim data secara independen sehingga memungkinkan komunikasi dua arah. Metode ini cocok untuk aplikasi seperti chat atau game

> What are the potential security considerations involved in implementing a gRPC service in Rust, particularly regarding authentication, authorization, and data encryption?

- Rust mendukung banyak mekanisme authentikasi seperti mTLS dan JWT. Hal ini penting untung memverifikasi identitas client yang dapat digunakan untuk hal-hal lain seperti profile atau autorization. Salah satu hal yang perlu diperhatikan adalah memverifikasi signing dari JWT atau certificate untuk mTLS. Apabila server tidak melakukan verifikasi dengan baik, maka dapat terjadi impersonation, dimana seorang user berpura-pura menjadi user lain.

- Authorization digunakan untuk menentukan apakah sebuah client dapat melakukan atau mengakses sesuatu. Ada berbagai macam metode authorization yang bisa digunakan seperti RBAC dan ABAC. Keamanan dari proses authorization bergantung langsung dengan authentication sehingga jika sebuah client dapat melakukan impersonation maka dia juga akan dapat authorization client tersebut. Selain itu penting juga untuk menentukan authorization sebuah client berdasarkan database, bukan input pengguna. 
- Data encryption digunakan untuk mengamankan data dalam komunikasi gRPC. Implementasi yang paling umum digunakan adalah SSL/TLS. Data encryption penting karena request yang dikirim oleh client dapat mengandung informasi sensitif seperti password. Untuk memastikan bahwa data encryption aman, harus dipastikan bahwa certificate SSL/TLS tidak dapat diakses oleh umum.

> What are the potential challenges or issues that may arise when handling bidirectional streaming in Rust gRPC, especially in scenarios like chat applications?

Issue pertama yang dapat terjadi adalah issue concurrency dimana sebuah client mengirikan sebuah request ke server dan server mengirim response ke client dalam waktu yang sama. Apabila tidak menerapkan concurrency, maka akan ada latensi yang tinggi atau bahkan terjadi deadlock. Issue kedua adalah menangani state management, terutama untuk chat applications yang dapat memadai banyak server. Contohnya kita melakukan chatting dengan 2 orang, A dan B. Aplikasi kita harus dapat menangani state antara A dan B sehingga jika kita pindah chat dari satu orang ke orang yang lain kita tidak perlu melakukan login ulang.

> What are the advantages and disadvantages of using the tokio_stream::wrappers::ReceiverStream for streaming responses in Rust gRPC services?

`tokio_stream::wrappers::ReceiverStream` memberikan keuntungan seperti integrasi dengan Tokio, built-in penanganan backpressure lewat stream dengan kapasitas yang terbatas dan juga abstraksi yang mempermudah pengelolaan aliran data tanpa harus menangani detail rendah secara langsung.

Namun pendekatan ini juga dapat memiliki kelemahan seperti performance overhead. Selain itu untuk developer yang belum mahir dalam asynchornous programming dalam rust, dapat mempersulit implementasi dan meningkatkan kemungkinan adanya bugs dalam aplikasi kita. 

> In what ways could the Rust gRPC code be structured to facilitate code reuse and modularity, promoting maintainability and extensibility over time?

Kita dapat meningkatkan maintainability kode kita dengan memisahkan implementasi service kita menjadi modul masing-masing. Selain itu kita juga dapat mengimplementasikan beberapa design pattern seperti dependency injection untuk mengurangi dependency antar komponen dan strategy pattern untuk memudahkan extensi dan juga testing.

> In the MyPaymentService implementation, what additional steps might be necessary to handle more complex payment processing logic?

Hal pertama yang kita bisa tambahkan adalah implementasi rollback sehingga jika ada langkah yang gagal, seluruh transaksi akan dibatalkan. Selain itu kita juga dapat menambahkan input validation dan juga logging dan monitoring. Kita juga bisa mengimplementasikan OTP untuk mengurangi security risk jika akun seorang user compromised, atau mengintegrasikan external payment gateway. Terakhir kita bisa mengimplementasikan notification service untuk menotifikasi pengguna jika transaksi sudah selesai.

> What impact does the adoption of gRPC as a communication protocol have on the overall architecture and design of distributed systems, particularly in terms of interoperability with other technologies and platforms?

Adopsi gRPC mengubah arsitektur sistem terdistribusi menjadi lebih efisien melalui pendekatan contract-first, di mana penggunaan Protocol Buffers memastikan komunikasi antar-layanan tetap konsisten, cepat, dan terstandarisasi secara otomatis dalam berbagai bahasa pemrograman dan fitur automatic code generation yang mengurangi keperluan menulis boilerplate. Secara desain, gRPC mengoptimalkan performa melalui protokol HTTP/2 yang mendukung streaming data dua arah dan kompresi biner, namun dalam hal interoperabilitas, ia menciptakan tantangan khusus karena memerlukan proxy tambahan (seperti gRPC-Web) jika ingin diakses langsung oleh browser web.

> What are the advantages and disadvantages of using HTTP/2, the underlying protocol for gRPC, compared to HTTP/1.1 or HTTP/1.1 with WebSocket for REST APIs?

HTTP/2 menawarkan keunggulan signifikan dibandingkan HTTP/1.1 melalui multiplexing, yang memungkinkan pengiriman banyak data secara paralel dalam satu koneksi tanpa hambatan head-of-line blocking, serta penggunaan format biner dan kompresi header yang lebih efisien daripada teks standar pada REST API. Dibandingkan dengan WebSocket, HTTP/2 menyediakan fitur streaming yang lebih terstruktur dalam ekosistem HTTP, namun tetap memiliki kekurangan berupa kerentanan terhadap pemblokiran di level TCP jika terjadi kehilangan paket, serta kompleksitas tinggi yang membuatnya sulit mendebug secara manual. Selain itu, karena banyak browser yang belum mendukung HTTP/2 sepenuhnya, arsitek biasanya tetap memilih HTTP/1.1 atau WebSocket agar bisa diakses oleh semua pengguna.

> How does the request-response model of REST APIs contrast with the bidirectional streaming capabilities of gRPC in terms of real-time communication and responsiveness?

REST API bekerja secara satu arah dan stateless, di mana klien harus meminta data secara manual setiap kali dan menunggu respons server. Pola ini menciptakan latensi yang kurang efisien karena setiap permintaan membawa overhead tersendiri.

Sebaliknya, gRPC mendukung streaming dua arah, sehingga klien dan server dapat bertukar data secara bersamaan melalui satu koneksi HTTP/2 yang terus terbuka, tanpa perlu memicu permintaan baru untuk setiap data yang dikirim. Hasilnya, komunikasi real-time menjadi jauh lebih responsif dan hemat bandwidth, karena data langsung dikirim begitu tersedia.

> What are the implications of the schema-based approach of gRPC, using Protocol Buffers, compared to the more flexible, schema-less nature of JSON in REST API payloads?

Pendekatan berbasis skema pada gRPC menggunakan Protocol Buffers menciptakan kontrak yang kaku dan terstandarisasi, yang secara otomatis menjamin konsistensi tipe data di berbagai platform serta menghasilkan ukuran payload biner yang jauh lebih kecil dan cepat untuk diproses. Hal ini berbeda dengan sifat JSON pada REST yang schema-less dan fleksibel, di mana pengembang dapat mengubah struktur data dengan mudah tanpa kompilasi ulang, namun harus membayar harga berupa overhead penguraian teks yang lebih lambat dan risiko kesalahan integrasi karena tidak adanya validasi kontrak yang ketat. 
