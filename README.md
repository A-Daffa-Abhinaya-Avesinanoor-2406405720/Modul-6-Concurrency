# Commit 1:

Pada milestone ini, code merupakan fondasi web server sederhana berbasis `TcpListener` yang dapat di-run di `127.0.0.1:7878`. Di fungsi `main`, server melakukan `bind` ke address tersebut lalu melakukan iterasi pada `listener.incoming()` untuk menerima setiap connection yang masuk. Setiap connection direpresentasikan sebagai `TcpStream`, lalu diteruskan ke function `handle_connection`.

Di dalam `handle_connection`,  `BufReader` digunakan untuk read data dari stream per baris. Request HTTP kemudian dikumpulkan dengan `lines()`, diubah menjadi kumpulan `String`, lalu read sampai menemukan baris kosong dengan `take_while(|line| !line.is_empty())`. Baris kosong menandai akhir header pada HTTP request. Hasil akhirnya dikeluarkan dengan `println!` untuk melihat isi request yang dikirim browser.

Hal yang saya pelajari dari milestone ini adalah bagaimana alur dasar sebuah web server bekerja: server membuka socket, menunggu connection, menerima request, lalu proses. Implementasi ini masih bersifat single-threaded, sehingga connection di-handle satu per satu berurutan. 
