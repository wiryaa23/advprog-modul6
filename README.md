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
2. Header Content-Length: Panjang konten dalam byte
3. Dua baris kosong (\r\n\r\n) sebagai pemisah header dan konten
4. Isi file hello.html
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