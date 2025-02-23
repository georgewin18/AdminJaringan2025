# Review - Workshop Adminsitrasi Jaringan

## TUGAS 1: Analisa file http.cap

### A. Versi HTTP yang digunakan
Untuk dapat melihat versi HTTP yang digunakan pada file http.cap tersebut, pertama kita dapat memilih salah satu packet HTTP atau TCP yang ada

![img1](img/1.png)

Kemudian klik kanan pada salah satu packet dan pilih `Follow` > `HTTP Stream`
Maka akan ditampilkan sebagai berikut

![img2](img/2.png)

Pada tampilan HTTP Stream yang ditampilkan dapat terlihat beberapa informasi mengenai data mengenai packet HTTP yang ditransmisikan. Pada tampilan HTTP Stream di atas, dapat terlihat bahwa versi HTTP yang digunakan adalah `HTTP/1.1`

### B. IP Address dari client dan server
Pada file http.cap, kita dapat melihat informasi IP Address dari client dan server langsung pada packet pertama pada file tersebut

![img3](img/3.png)

Pada tiga packet pertama, dapat kita lihat, ketiga packet tersebut menunjukkan proses TCP three way handshake dan pada packet pertama dapat kita lihat source address dan destination addressnya. Untuk Source Address menunjukkan IP Address dari client, sedangkan untuk Destination Address menunjukkan IP Address dari server.
- Client IP Address : `145.254.160.237` 
- Server IP Address : `65.208.228.223`

### C. Waktu dari client mengirimkan HTTP request
Pada menu bar di bagian atas, kita dapat memilih menu `Statistic` > `Flow Graph` untuk melihat alur dari packet-packet yang dikirimkan seperti berikut:

![img4](img/4.png)

Untuk mengetahui kapan waktu dari client mengirimkan HTTP request, dapat kita lihat pada packet yang berisi `HTTP GET`, dan pada sisi kiri dari Flow Graph menunjukkan kapan waktu packet tersebut dikirimkan, yaitu `0.911310` detik dari awal packet capture dijalankan.

### D. Waktu dari server menerima HTTP request dari client
![img5](img/5.png)

Sedangkan untuk waktu dari server untuk menerima packet `HTTP GET` dari client sebelumnya ada pada packet selanjutnya, yaitu packet `ACK` dari server pada gambar di atas yang waktunya menunjukkan `1.472116` detik dari awal packet capture dijalankan.

### E. Waktu yang dibutuhkan untuk transfer dan response dari client ke server
Untuk mengetahui waktu yang dibutuhkan dari awal client request hingga server response, dapat kita lihat dari waktu response dari server dikirimkan

![img6](img/6.png)

Dapat dilihat bahwa packet `HTTP OK` yang merupakan response dari server ke client request dikirimkan pada waktu `4.8466969` detik dari awal packet capture. Untuk mendapatkan waktu yang dibutuhkan untuk transfer dan response dapat dihitung dari waktu packet `HTTP GET` dikirim hingga packet `HTTP OK` dikirimkan. Maka dapat dihitung sebagai berikut:
- Waktu packet `HTTP GET` dikirim : `0.911310`
- Waktu packet `HTTP OK` dikirim : `4.8466969`
- Selisih : `4.8466969` - `0.911310` = `3,9353869` detik

## TUGAS 2: Analisa Gambar Types of Data Deliveries

## TUGAS 3: Resume Tahapan TCP
