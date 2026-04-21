# Commit 1:

Pada milestone ini, code merupakan fondasi web server sederhana berbasis `TcpListener` yang dapat di-run di `127.0.0.1:7878`. Di fungsi `main`, server melakukan `bind` ke address tersebut lalu melakukan iterasi pada `listener.incoming()` untuk menerima setiap connection yang masuk. Setiap connection direpresentasikan sebagai `TcpStream`, lalu diteruskan ke function `handle_connection`.

Di dalam `handle_connection`,  `BufReader` digunakan untuk read data dari stream per baris. Request HTTP kemudian dikumpulkan dengan `lines()`, diubah menjadi kumpulan `String`, lalu read sampai menemukan baris kosong dengan `take_while(|line| !line.is_empty())`. Baris kosong menandai akhir header pada HTTP request. Hasil akhirnya dikeluarkan dengan `println!` untuk melihat isi request yang dikirim browser.

Hal yang saya pelajari dari milestone ini adalah bagaimana alur dasar sebuah web server bekerja: server membuka socket, menunggu connection, menerima request, lalu proses. Implementasi ini masih bersifat single-threaded, sehingga connection di-handle satu per satu berurutan. 

# Commit 2:

![Commit 2 screen capture](/assets/images/commit2.png) 

Pada milestone ini, server dibuat agar tidak hanya read request, tetapi juga return halaman HTML ke browser. Di `handle_connection`, server tetap read request dengan `BufReader`, lalu  `status_line` dengan nilai `HTTP/1.1 200 OK`. Setelah itu, isi file `hello.html` di-read menggunakan `fs::read_to_string`, panjang kontennya dihitung untuk `Content-Length`, lalu seluruh response HTTP disusun dengan `format!` sebelum dikirim ke client melalui `stream.write_all()`.

Hal yang saya pelajari dari milestone ini adalah bahwa browser membutuhkan response HTTP yang valid, bukan hanya data biasa. Selain body HTML, server juga harus mengirim status line dan header yang sesuai agar browser bisa menampilkan halaman dengan benar. Rust dapat membaca file dari filesystem dan menggabungkannya ke dalam response.
