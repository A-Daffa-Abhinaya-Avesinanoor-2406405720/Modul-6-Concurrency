# Commit 1:

Pada milestone ini, code merupakan fondasi web server sederhana berbasis `TcpListener` yang dapat di-run di `127.0.0.1:7878`. Di fungsi `main`, server melakukan `bind` ke address tersebut lalu melakukan iterasi pada `listener.incoming()` untuk menerima setiap connection yang masuk. Setiap connection direpresentasikan sebagai `TcpStream`, lalu diteruskan ke function `handle_connection`.

Di dalam `handle_connection`,  `BufReader` digunakan untuk read data dari stream per baris. Request HTTP kemudian dikumpulkan dengan `lines()`, diubah menjadi kumpulan `String`, lalu read sampai menemukan baris kosong dengan `take_while(|line| !line.is_empty())`. Baris kosong menandai akhir header pada HTTP request. Hasil akhirnya dikeluarkan dengan `println!` untuk melihat isi request yang dikirim browser.

Hal yang saya pelajari dari milestone ini adalah bagaimana alur dasar sebuah web server bekerja: server membuka socket, menunggu connection, menerima request, lalu proses. Implementasi ini masih bersifat single-threaded, sehingga connection di-handle satu per satu berurutan. 

# Commit 2:

![Commit 2 screen capture](/assets/images/commit2.png) 

Pada milestone ini, server dibuat agar tidak hanya read request, tetapi juga return halaman HTML ke browser. Di `handle_connection`, server tetap read request dengan `BufReader`, lalu  `status_line` dengan nilai `HTTP/1.1 200 OK`. Setelah itu, isi file `hello.html` di-read menggunakan `fs::read_to_string`, panjang kontennya dihitung untuk `Content-Length`, lalu seluruh response HTTP disusun dengan `format!` sebelum dikirim ke client melalui `stream.write_all()`.

Hal yang saya pelajari dari milestone ini adalah bahwa browser membutuhkan response HTTP yang valid, bukan hanya data biasa. Selain body HTML, server juga harus mengirim status line dan header yang sesuai agar browser bisa menampilkan halaman dengan benar. Rust dapat membaca file dari filesystem dan menggabungkannya ke dalam response.

# Commit 3:
![Commit 3 screen capture](/assets/images/commit3.png) 

Pada milestone ini, server tidak lagi selalu mengirim response yang sama, tetapi mulai memvalidasi request dan memilih response yang sesuai. Di `handle_connection`, request di-read hanya pada baris pertama dengan `buf_reader.lines().next()` karena bagian itu sudah cukup untuk menentukan apakah client meminta `GET / HTTP/1.1` atau path lain. Jika request sesuai, server memilih `HTTP/1.1 200 OK` dan file `hello.html`; jika tidak, server memilih `HTTP/1.1 404 NOT FOUND` dan file `404.html`.

Bagian refactoring dilakukan dengan memisahkan bagian yang berbeda dari response dan bagian yang sama. Yang berbeda hanya `status_line` dan `filename`, sehingga keduanya diletakkan di dalam `if/else` sebagai tuple `(status_line, filename)`. Setelah itu, bagian yang sama seperti read file, hitung `Content-Length`, menyusun string response, dan mengirimkannya ke stream dilakukan satu kali di luar `if/else`. Refactoring ini diperlukan agar kode tidak berulang, lebih readable, dan lebih mudah diubah ke depannya karena jika format response berubah, cukup modifikasi suatu bagian saja.

Hal yang saya pelajari dari milestone ini adalah bahwa validasi request sangat penting agar server bisa memberikan response yang benar sesuai resource yang diminta. Saya juga jadi lebih paham struktur request line HTTP, penggunaan tuple destructuring di Rust, dan manfaat refactoring untuk mengurangi duplikasi logika. Dari sini saya melihat bahwa program yang berjalan benar belum tentu sudah rapi; dengan refactoring, kode menjadi lebih clean, lebih maintainable, dan lebih jelas menunjukkan perbedaan antara response sukses dan response error.

# Commit 4:

Pada milestone ini, server ditambahkan simulasi slow response dengan menambahkan satu route baru, yaitu `GET /sleep HTTP/1.1`. Di `handle_connection`, request line dicocokkan menggunakan `match` karena sekarang ada tiga kemungkinan response: request ke `/`, request ke `/sleep`, dan request lain yang akan diarahkan ke `404.html`. Khusus untuk `/sleep`, server menjalankan `thread::sleep(Duration::from_secs(5))` sebelum menyusun response `200 OK`, sehingga browser baru menerima halaman setelah delay lima detik selesai.

Cara kerjanya, `thread::sleep` tidak membuat server mengerjakan hal lain selama lima detik, tetapi justru menghentikan thread yang sedang menangani request tersebut. Karena server ini masih single-threaded dan di `main` setiap koneksi diproses berurutan lewat `handle_connection(stream)`, maka ketika request `/sleep` sedang berjalan, request berikutnya harus menunggu sampai proses sleep selesai.

Hal yang saya pelajari dari milestone ini adalah bahwa bottleneck pada server dapat berasal dari satu request lambat yang memblokir alur eksekusi lain. Saya juga memahami kenapa `match` lebih cocok dipakai dibanding `if/else` ketika jumlah kasus bertambah banyak. Dengan mengalami masalahnya secara langsung, saya bisa lebih memahami kenapa server single-threaded kurang ideal untuk menangani banyak request secara bersamaan.
