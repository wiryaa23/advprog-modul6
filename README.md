## Reflection

### Milestone 1: Understanding what's inside the handle_connection method

- Menerima parameter TcpStream

  Method ini menerima stream bertipe data TcpStream yang berfungsi untuk membaca dan menulis data melalui koneksi TCP.

- Membuat BufReader

  BufReader digunakan untuk membaca data dari TcpStream dengan buffer, sehingga lebih efisien dalam membaca baris demi baris.

- Membaca HTTP Request

  `.lines()` akan membaca aliran data dari stream secara baris per baris. Setelah itu, `.map()` akan mengekstrak hasil menjadi String. Hal ini akan dilakukan terus oleh `.take_while(|line| !line.is_empty())` sampai menemukan baris kosong yang menandakan akhir dari header HTTP. Akhirnya, `.collect()` mengumpulkan baris-baris tersebut ke dalam Vec<String> yang berisi baris-baris HTTP Request. Data yang diperoleh kemudian disimpan pada variabel *http_request*.

- Mencetak HTTP Request

  `println!()` mencetak permintaan HTTP dalam format yang lebih mudah dibaca.

### Milestone 2: New handle_connection method

Metode `handle_connection` sekarang tidak hanya membaca HTTP request dari klien, tetapi juga mengirimkan HTTP response yang berisi file hello.html sebagai kontennya.

- Menyiapkan status response

Setelah membaca request, kita menyiapkan respons dengan status HTTP 200 OK, yang berarti permintaan berhasil diproses.
```rust
let status_line = "HTTP/1.1 200 OK";
```

- Membaca konten file HTML

Program membaca isi file hello.html menjadi string menggunakan `fs::read_to_string`.
```rust
let contents = fs::read_to_string("hello.html").unwrap();
```

- Menghitung panjang konten HTML yang telah dibaca
```rust
let length = contents.len();
```

- Membuat HTTP Response yang lengkap

Respons HTTP terdiri dari:
1. Status baris pertama: "HTTP/1.1 200 OK"
2. Header Content-Length: Panjang konten
3. Dua baris kosong (\r\n\r\n) sebagai pemisah header dan konten
4. Contents: Isi file hello.html
```rust
let response =
        format!("{status_line}\r\nContent-Length:{length}\r\n\r\n{contents}");
```

- Mengirimkan HTTP Response ke klien

HTTP Response dikirim ke klien menggunakan `write_all()` dengan mengkonversi string response menjadi byte array menggunakan `as_bytes()`. Terakhir, program memastikan semua data terkirim dengan `unwrap()` dan mengirimkannya melalui stream.
```rust
stream.write_all(response.as_bytes()).unwrap();
```

Gambar response di web browser:
![Commit 2 screen capture](/assets/images/commit2.png)

### Milestone 3: New handle_connection method (again)

- Membaca Baris Pertama Request

Proram menggunakan `buf_reader.lines().next().unwrap().unwrap()` untuk mendapatkan baris pertama dari request HTTP; berisi request method, path, dan HTTP version; yang disimpan di dalam *request_line*.

- Memeriksa Request dan Menentukan Response

Jika request berupa "GET / HTTP/1.1", server akan mengembalikan status "HTTP/1.1 200 OK" dan mengirimkan file hello.html. Namun untuk request yang lain, server akan memberikan response dengan "HTTP/1.1 404 NOT FOUND" dan mengirimkan file 404.html.

- Membaca File yang Dikirim

`fs::read_to_string(file).unwrap()` akan digunakan untuk membaca isi file yang telah ditentukan sebelumnya (hello.html atau 404.html). Selanjutnya, `contents.len()` menghitung ukuran konten untuk dimasukkan dalam header Content-Length.

- Mengirimkan Response

Menggunakan `stream.write_all(response.as_bytes()).unwrap()` untuk mengirimkan data ke klien.

Gambar response di web server:
![Commit 3 screen capture](/assets/images/commit3.png)

### Milestone 4: Why it works like that

`handle_connection` dimodifikasi dengan tambahan penanganan path `/sleep` untuk mensimulasikan request yang butuh waktu lebih lama. Jika kita mengaksesnya, maka akan memicu `thread::sleep(Duration::from_secs(10))` yang menunda response selama 10 detik sebelum mengirim halaman `hello.html`. Simulasi ini bertujuan untuk menunjukkan kelemahan server dengan *single-threaded model*, karena server akan memproses permintaan satu per satu. Ketika server mulai memproses request pada endpoint `/sleep`, maka ia akan menunda request lain (dalam kasus ini, endpoint biasa) sampai request dari `/sleep` selesai diproses.

### Milestone 5: Multi-threading

Kita menambahkan file baru yaitu `lib.rs` yang digunakan untuk mendukung implementasi *multi-threading*. Setelah itu, `main.rs` dimodifikasi untuk menerapkan model *multi-threading* menggunakan ThreadPool. Perubahannya berupa inisialisasi *thread pool* dengan sintaks `let pool = ThreadPool::new(4)` yang menciptakan 4 buah *worker thread*. `pool.execure()` kemudian mendelegasikan penanganan koneksi ke *thread* yang tersedia dari *pool* dengan pendekatan `|| { handle_connection(stream); }`. File `lib.rs` yang baru dibuat berfungsi sebagai modul terpisah yang mengimplementasikan ThreadPool, dengan elemen utama seperti struktur Worker, tipe Job, dan kanal MPSC (Multiple Producer, Single Consumer) untuk komunikasi antar *thread*. Semua ini bertujuan untuk mengatasi keterbatasan server *single-thread*, yang sebelumnya hanya dapat menangani satu permintaan dalam satu waktu. Dengan adanya ThreadPool, server kini mampu memproses beberapa koneksi secara paralel, sehingga request yang memerlukan waktu lama (misalnya endpoint `/sleep`) tidak menghambat request lainnya. Hal ini secara signifikan meningkatkan responsivitas server sehingga kita bisa lebih efisien dalam menangani banyak tugas secara paralel.

### Bonus

Saya mengimplementasikan fungsi `build()` yang berfungsi untuk menggantikan fungsi `new()`. Perubahan ini dilakukan untuk melakukan peningkatan khususnya dalam bagian *Refactoring to Improve Modularity and Error Handling*. Fungsi `new()` awalnya sudah cukup baik, namun akan langsung memicu *panic* jika mengalami *error* (misalnya seperti *size* yang tidak masuk akal) karena langsung mengembalikan ThreadPool. Maka dari itu, dibuat fungsi `build()` yang mengatasi masalah ini, dengan memberikan penanganan lebih baik ketika bertemu *error* karena mengembalikan Result<ThreadPool, &str>. Dengan demikian, kita dapat melakukan implementasi validasi ukuran ThreadPool sehingga membuat kode menjadi lebih modular dan dapat menghindari *error* yang tidak terduga.