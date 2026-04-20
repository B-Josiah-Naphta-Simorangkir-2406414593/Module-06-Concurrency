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