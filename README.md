## Reflection

### Milestone 1: Understand what's inside the handle_connection method

- Menerima parameter TcpStream

  Method ini menerima stream bertipe data TcpStream yang berfungsi untuk membaca dan menulis data melalui koneksi TCP.

- Membuat BufReader

  BufReader digunakan untuk membaca data dari TcpStream dengan buffer, sehingga lebih efisien dalam membaca baris demi baris.

- Membaca HTTP Request

  `.lines()` akan membaca aliran data dari stream secara baris per baris. Setelah itu, `.map()` akan mengekstrak hasil menjadi String. Hal ini akan dilakukan terus oleh `.take_while(|line| !line.is_empty())` sampai menemukan baris kosong yang menandakan akhir dari header HTTP. Akhirnya, `.collect()` mengumpulkan baris-baris tersebut ke dalam Vec<String> yang berisi baris-baris HTTP Request. Data yang diperoleh kemudian disimpan pada variabel *http_request*.

- Mencetak HTTP Request

  `println!()` mencetak permintaan HTTP dalam format yang lebih mudah dibaca.

