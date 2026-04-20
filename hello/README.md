# Commit 1 Reflection

## Apa itu handle_connection?

Dalam milestone ini, fungsi handle_connection bertugas untuk memproses setiap koneksi masuk yang diterima oleh TcpListener. Saat sebuah client (seperti browser) terhubung ke server kita, koneksi tersebut direpresentasikan sebagai TcpStream.

Di dalam fungsi ini, kita melakukan beberapa hal:

1. Membaca Data: Kita membungkus stream ke dalam BufReader. Mengapa? Karena stream mentah bersifat low-level. Dengan BufReader, kita bisa membaca data baris demi baris secara efisien melalui buffer memori.

2. Iterasi Request: Kita mengambil baris-baris teks dari request HTTP menggunakan .lines().

3. Parsing: Kita menggunakan .map(|result| result.unwrap()) untuk mengambil string aslinya, dan .take_while(|line| !line.is_empty()) karena spesifikasi HTTP menyatakan bahwa akhir dari sebuah request header ditandai oleh baris kosong.

4. Logging: Terakhir, semua baris tersebut dikumpulkan ke dalam Vec<_> dan dicetak ke terminal agar kita bisa melihat isi request dari browser (seperti Method, Path, User-Agent, dll).

## Kenapa Ada Banyak Pesan "Connection Established"?

Saat saya menjalankan server dan mengaksesnya melalui browser, terminal seringkali menunjukkan pesan "Connection established!" lebih dari satu kali untuk satu kali akses. Hal ini terjadi karena:

1. Browser Modern: Browser seringkali membuka beberapa koneksi TCP sekaligus untuk mempercepat loading (misalnya satu untuk konten utama, satu lagi untuk mencari favicon.ico).

2. Retry Logic: Jika server tidak segera merespon (karena di Milestone 1 kita belum mengirim response balik), browser mungkin mencoba mengirim ulang request untuk memastikan koneksi tidak terputus.


# Commit 2 Reflection

## Penjelasan Kode handle_connection

Pada milestone ini, fungsi handle_connection telah ditingkatkan untuk tidak hanya membaca request, tetapi juga memberikan respon balik berupa konten HTML. Berikut adalah alur kerjanya:

1. Pemisahan Request: Meskipun kita membaca request dari buf_reader, pada tahap ini kita fokus pada pengiriman response.

2. Pembacaan File: Menggunakan fs::read_to_string("hello.html"), server membaca file fisik di disk menjadi String. Ini memberikan fleksibilitas untuk mengubah tampilan web tanpa harus mengubah kode Rust (selama nama filenya tetap).

3. Struktur Response HTTP: Kita menyusun string respon dengan format:

    a.Status Line: HTTP/1.1 200 OK yang memberitahu browser bahwa permintaan berhasil diproses.

    b. Header: Content-Length sangat krusial di sini. Ini memberitahu browser berapa banyak bytes data yang harus mereka baca dari stream. Tanpa ini, browser mungkin tidak tahu kapan sebuah file selesai di-download.

    c. Body: Isi dari file hello.html yang telah dibaca sebelumnya.

4. Penulisan Stream: stream.write_all(response.as_bytes()).unwrap() mengirimkan string yang sudah diubah menjadi byte array kembali melalui koneksi TCP.

## Mengapa Kita Membutuhkan Content-Length?

Dalam protokol HTTP, Content-Length adalah salah satu header yang memberi tahu penerima ukuran body pesan dalam satuan octets (bytes).

Jika kita tidak menyertakan header ini atau ukurannya salah:

1. Browser mungkin akan terus menunggu data tambahan (loading tidak selesai).

2. Koneksi mungkin ditutup secara prematur, mengakibatkan halaman web tampil tidak utuh atau rusak.

## Analisis Hasil

Setelah menjalankan cargo run dan mengakses 127.0.0.1:7878, browser berhasil merender elemen h1 dan p yang ada di dalam hello.html. Hal ini membuktikan bahwa server Rust kita telah berfungsi sebagai server web statis sederhana yang mampu menangani handshake TCP dan merespon dengan protokol HTTP yang valid.

Berikut adalah tampilan halaman web dari mesin saya:
![Commit 2 screen capture](/assets/images/commit2.png)


# Commit 3 Reflection

## Mengenal Perbedaan Response

Pada milestone ini, saya mengimplementasikan logika percabangan untuk menangani berbagai request path.

Jika request berisi GET / HTTP/1.1, server merespon dengan status 200 OK dan menampilkan hello.html.

Jika request berisi jalur lain (misalnya /random), server merespon dengan status 404 NOT FOUND dan menampilkan 404.html.

## Mengapa Refactoring Diperlukan?

Sebelum direfaktorisasi, kita mungkin tergoda untuk menulis blok if/else yang di dalamnya penuh dengan kode pembacaan file dan pengiriman stream yang berulang.
Refaktorisasi pada kode di atas sangat penting karena:

Readability: Kode menjadi jauh lebih bersih. Logika penentuan "apa yang dikirim" dipisahkan dari logika "bagaimana cara mengirimnya".

Maintainability: Jika nanti saya ingin mengubah format header HTTP, saya hanya perlu mengubah satu tempat (di fungsi format!), bukan di setiap blok if.

Efficiency: Mengurangi risiko kesalahan ketik (typo) pada bagian pengiriman data karena kodenya hanya ditulis satu kali.

## Analisis Hasil

Setelah dijalankan, saat saya mengakses 127.0.0.1:7878/bad, browser sekarang dengan benar menampilkan halaman "Oops!" dengan status code 404. Ini menunjukkan server sudah memiliki kemampuan dasar untuk memvalidasi input dari user.