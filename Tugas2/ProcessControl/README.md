# Chapter 4 : Process Control

# Komponen Proses

Sebuah proses terdiri dari sebuah alamat space dan sebuah set struktur data dalam kernel. Alamat space merupakan sekumpulan halaman memori yang ditandai oleh kernel untuk digunakan oleh sebuah proses. Halaman ini digunakan untuk menyimpan kode proses, data, dan stack. Struktur data dalam kernel melacak status proses, prioritasnya, parameter penjadwalan, dan sebagainya

Kita bisa mengibaratkan proses sebagai sebuah container untuk sekumpulan resources yang dikelola oleh kernel untuk program yang berjalan. Resources ini mencakup halaman memori yang menyimpam kode program dan data program, file descriptor merujuk ke file yang terbuka, serta berbagai atribut yang menggambarkan status proses.

Struktur data internal kernel mencatat berbagai infromasi mengenai setiap proses, termasuk:
- map dari alamat space
- status dari proses (running, sleeping, dll)
- process priority
- Informasi tentang resources yang digunakan proses (CPU, memori, dll)
- Infromasi mengenai file dan port jaringan yang dibuka oleh proses
- Signal mask dari proses (kumpulan signal yang diblokir)
- Pemilik process (user ID dari user yang memulai proses)

Sebuah "thread" merupakan eksekusi konteks dalam suatu proses. Satu proses bisa memiliki beberapa thread, yang semuanya berbagi alamat space dan resources yang sama. Thread digunakan untuk mencapai paralelisme dalam proses. Thread juga dikenal sebagai proses kecil karena lebih murah untuk dibuat (penggunaan resource) dan di-destroy dibandingkan dengan proses

## PID: process ID number

Tiap proses diidentifikasi dengan sebuah process ID number yang unik, atau PID. PID adalah sebuah integer yang diberikan oleh kernel saat sebuah proses dibuat. PID diguinakan untuk merujuk pada proses dalam berbagai system calls, contohnya untuk mengirim sinyal ke proses.

Konsep "namespace" dari proses mengijinkan beberapa proses memiliki PID yang sama. Namespace digunakan untuk membuat containter, yang merupakan environment terisolasi yang memilii pandangan tersendiri terhadap sistem. Containers digunakan untuk menjalankan beberapa instance aplikasi pada sistem yang sama, masing-masing dalam environment yang terisolasi.

## PPID: parent process ID number

Setiap proses juga memiliki parent process, yaitu proses yang membuatnya. Parent process ID number atau PPID adalah PID dari parent process. PPID digunakan untuk merujuk ke parent process dalam berbagai system calls, contohnya untuk mengirim sinyal ke parent process.

## UID dan EUID: user ID dan effective user ID

User ID atau UID, adalah ID dari user yang memulai proses. Effective User ID atai EUID, merupakan ID user yang digunakan proses untuk menentukan resource apa yang bisa diakses oleh proses. EUID digunakan untuk mengontrol akses file, port jaringan, dan reources lainnya.

# Lifecycle of a Process

Untuk membuat proses baru, sebuah proses menyalin dirinya sendiri menggunakan system call fork. Fork membuat salinan dari proses asli, dan salinan tersebut hampir identik dengan parent prosesnya. Proses baru memiliki PID yang berbeda dan memiliki PID yang berbeda dan memiliki informasi accounting tersendiri. (Secara teknis, sistem Linux menggunakan clone, yang merupakan superset dari fork yang menangani thread dan mencakup fitur tambahan. Fork tetap ada di kernel untuk kompatibilitas ke belakang tetapi secara internal memanggil clone).

Saat sistemmelakukan boot, kernel secara mandiri membuuat dan menginstall beberapa proses. Yang paling penting di antaranya adalah init atau systemd, yang selalu memiki PID 1. Proses ini menjalankan script startup system, meskipun cara kerjanya sedikit berbeda antara UNIX dan Linux. Semua proses selain yang dibuat oleh kernel adlaah keturunan dari proses primordial ini.

## Signal

Signal adalah ada;aj cara untuk mengirim notifikasi ke sebuah proses. Signal digunakan untuk memberi tahu suatu proses bawha sebuah peristiwa tertentu telah terjadi.

Terdapat sekitar 30 jenis signal yang didefinisikan, dan masing-masing digunakan untuk berbagai keperluan, seperti:
- Dikirim antara proses sebagai sarana komunikasi
- Dikirim oleh driver terminal untuk kill, interrupt, atau suspend process
- Dikirim oleh administrator (menggunakan kill) untuk berbagai tujuan
- Dikirim oleh kernel saat suatu proses melakukan pelanggaran, seperti division by zero
- Dikirim oleh kernel untuk memberi tahu suatu proses tentang kondisi "menarik", seperti kematian child process atau tersedianya data pada I/O channel


Signal KILL, INT, TERM, HUP, dam QUIT terdengar seperti memiliki arti yang mirip. tetapi sebenarnya memiliki kegunaan yang berbeda:
- KILL (SIGKILL)
Signal ini tidak dapat diblokir dan langsung menghentikan proses di tingkat kernel. Proses tidak pernah benar-benar menerima atau menangani sinyal ini.
- INT (SIGINT)
Dikirim oleh driver terminal saat pengguna menekan Ctrl + C. Ini adalah permintaan untuk menghentikan operasi saat ini. Program sederhana sebaiknya keluar (jika menangkap sinyal ini) atau membiarkan dirinya dihentikan oleh sistem, yang merupakan perilaku default jika signal tidak ditangkap. Program dengan interface interaktif (seperti shell) harus menghentikan tugas yang sedang berjalan, membersihkan resource, dan menunggu input user.
- TERM (SIGTERM)
Merupakan permintaan untuk menghentikan eksekusi secara normal. Proses yang menerima signal ini diharapkan membersihkan statenya terlebih dahulu sebelum keluar.
- HUP (SIGHUP)
Dikirim ke duatu proses ketika terminal pengontrolnya ditutup. Awalnya digunakan untuk menandakan koneksi telepon "terputus" (hang up), tetapi kini sering digunakan untuk memberi instruksi pada proses daemonuntuk restart dan memuat ulang konfigurasi> behavior dari signal ini tergantung bagaimana proses menangani SIGHUP
- QUIT (SIGQUIT)
Mirip dengan TERM, tetapi jika tidak ditangkap, ia akan menghasilkan core dump. Beberapa program menginterpretasikan signal dengan arti yang berbeda.

## kill: send signals

Seperti namanya, perintah kill paling sering digunakan untuk menghentikan proses. kill dapat mengirim signal apapun, tetapi secara default, ia mengirimkan signam TERM. Normal user dapat menggunakan kill untuk menghentikan proses mereka sendiri, sedangkan root user dapat menggunakan kill unutk menghentikan proses manapun.

### Syntax:
```bash
kill [-signal] pid
```

- signal adalah nomor atau nama simbolik dari signal yang akan dikirim
- pid adalah nomor ID dari proses yang menjadi target

jika kill dijalankan tanpa nomor sinyal, prosees tidak dijamin langsung mati, karena TERM bisa ditangkap, diblock. atau diabaikan (ignored) oleh proses target.

killall menghentikan proses berdasarkan namanya, bukan berdasarkan PID nya. Namun tidak semua sistemn memiliki perintah ini. Contoh:
```bash
killall firefox-esr
```

pkill mirip dengan killall, tetapi menyediakan lebih banyak opsi. Contoh:
```bash
pkill -u abdoufermat # menghentikan semua proses yang dimiliki user abdoufermat
```

# ps: Monitoring Processes
Perintah ps adalah alat utama bagi administrator sistem untuk memantau proses yang berjalan. Meskipun versi ps berbeda di berbagai sistem, semuanya menyajikan informasi yang serupa.

Dengan ps, kita bisa melihat PID, UID, priority, dan control terminal dari proses. Kita juga bisa melihat penggunaan CPU dan memori dari proses dan status dari proses (running, stopped, sleeping, dll.).

Kita bisa menggunakan ps aux untuk memantau seluruh proses.
- `a` -> menampilkan proses dari semua pengguna
- `u` -> menampilkan informasi detail setiap proses
- `x` -> menampilkan proses yang tidak terkait dengan terminal

Perintah ini memberikan gambaran umum tentang semua proses yang berjalan di sistem.

Set argumen berguna lainnya adalah `lax`. yang akan memberikan lebih banyak informasi teknikal dari proses. lax sedikit lebih cepat dari aux karena tidak perlu mencari nama user dan groups.

Untuk melihat proses spesifik, kita bisa menggunakan grep untuk melakukan filter dari output ps

kita dapat menemukan PID dari proses menggunakan pgrep

atau pidof

# Interactive monitoring with top

Perintah top memberikan tampilan dinamis dan real-time dari sistem yang sedang berjalan. top menampilkan ringkasan sistem (CPU, RAM, load average, dll.). top juga menampilkan daftar proses atau thread yang sedang dikelola oleh kernel Linux. top dapat dikonfigurasi sesuai kebutuhan dan konfigurasi tersebut bisa disimpan untuk dipakai lagi. 

Secara default, tampilan akan diperbarui setiap 1-2 detik tergantung sistem.

Ada juga pernitah htop yang juga merupakan penampil proses interaktif berbasis teks untuk sistem UNIX. htop memerlukan ncurses, sehingga memiliki tampilan yang lebih berwarna dan interaktif, juga mendukung scrolling vertikal dan horizontal untuk melihat semua proses dan detailnya. htop menampilkan perintah lengkap yang diunakan untuk menjalankan setiap proses. htop lebih mudah dinavigasi dengan tampilan grafis CPU dan RAM.

# Nice and renice: changing process priority

niceness merupakan numeric hint yang diberikan ke kernel mengenai seberapa penting suatu proses dibandingkan proses lain yang bersaing untuk mendapatkan CPU time

niceness tinggi berarti prioritas proses rendah. Nilai niceness rendah atau negatif berarti prioritas proses tinggi.

Rentang nilai niceness yang diizinkan bervariasi di antara sistem.
Di Linux, rentangnya antara -20 hingga +19
Di FreeBSD, rentangnya antara -20 hingga +20

Proses dengan prioritas rendah adalah proses yang tidak terlalu penting. Proses ini akan mendapatkan lebih sedikit CPU time dibandingkan dengan proses prioritas tinggi, yang dianggap lebih penting dan harus mendapatkan lebih banyak CPU time.

perintah nice dignuakan untuk memulai proses dengan nilai niceness tertentu.
Syntax:
```bash
nice -n nice_val [command]
```

perintah renice digunakan untuk mengubah nilai niceness dari proses yang sedang berjalan
Syntax:
```bash
renice -n nice_val -p pid
```

Nilai prioritas digunakan oleh kernel Linux untuk menjadwalkan suatu pekerjaan. Sistem prioritasnya untuk 0 - 99 digunakan untuk proses real-time dan 100 - 139 untuk proses user

Di Linux, nilai prioritas dihitung dengan rumus:

> priority_value = 20 + nice value

# /proc filesystem

versi Linux dari ps dan top membaca informasi status proses dari direktori /proc, sebuah pseudo-filesystem dimana kernel mengekspos berbagai informasi tentang keadaan sistem.

Meskipun namanya /proc, direktori ini tidak hanya berisi informasi tentang proses, tetapi juga statistik sistem yang dihasilkan oleh kernel dan lainnya.

Setiap proses direpresentasikan sebagai sebuah direktori dalam /proc, dengan nama direktori sesuai dengan PID proses tersebut. driektori /proc berisi berbagai file yang menyediakan informasi tentang proses seperti Command Line yang digunakan, environment variables, file descriptor, dan informasi lainnya.

# Strace and Truss

Untuk mengetahui apa yang sedang dilakukan suatu proses, anda dapat menggunakan strace di Linux atau truss di FreeBSD. Perintah-perintah ini melacak system call dan signal yang terjadi dalam suatu proses.

perintah ini berguna untuk debugging atau sekedar memahami cara kerja suatu program

top mulai dengan memeriksa waktu saat ini. Kemudian membuka dan membaca direktori /proc, serta mengambil informasi dari file /proc/1/stat untuk mendapatkan detail tentang proses init.

# Runaway Processes

Terkadang, sebuah proses berhenti merespons dan mulai berjalan di luar kendali. Proses ini mengabaikan prioritas penjadwalannya dan terus menggunakan 100% CPU, sehingga sistem menjadi sangat lambat. PRoses seperti ini disebut runaway process.

Untuk menghentikannya kita bisa menggunakan perintah kill. Jika proses tidak meresponse terhadap signal TERM, gunakan signal KILL untuk menghentikan secara paksa:
`kill -9 pid

or

kill -KILL pid`

Kita dapat menginvestigasi penyebab runaway process dengan menggunakan strace atau truss untuk mengetahui apa yang dilakukan proses tersebut. Jika proses runaway menghasilkan banyak output, bisa saja filesystem menjadi penuh.

Cek penggunaan filesystem dengan `df -h` atau dengan `du` untuk menemukan file dan direktori terbesar

Kita bisa juga menggunakan `lsof` untuk melihat file apa saja yang sedang dibuka oleh runaway process:
```bash
lsof -p pid
```

# Periodic Processes

## cron: scchedule command

cron (crond di RedHat) adalah alat tradisional untuk menjadwalkan perintah berdasarkan jadwal tertentu. cron dimulai saat sistem booting dan terus berjalan selama sistem aktif.

cron membaca file konfigurasi yang berisi daftar perintah dan waktu eksekusinya. Perintah-perintah ini dieksekusi menggunakan sh, sehingga hampir semua yang bisa dilakukan secara manual dari shell bisa dijalankan melalui cron.

File konfigurasi cron disebut `crontab` (cron table). crontab untuk setiap pengguna disiman di /var/spool/cron/ (Linux) atau /var/cron/tabs/ (FreeBSD).

## format crontab

file crontab memiliki 5 lolom untuk menentukan waktu dan tanggal eksekusi, diikuti oleh perintah yang akan dijalankan.

## crontab management

peintah crontab digunakan untuk membuat, mengubah, dan menghapus crontab:
- `crontab -e` untuk mengedit crontab
- `crontab -l` untuk menampilkan crontab saat ini
- `crontab -r` untuk menghapus crontab saat ini

## Sytsemd timer

Systemd timer adalah file konfigurasi untuk yang berhakhiran `.timer`. Systemd timer lebih fleksibel dan kuat dibanding cron. Untuk melihat daftar timer yang aktif, gunakan `systemctl list-timers`

## Common use for scheduled tasks

### Mengirimkan mail

Kita dapat secara otomatis mengirimkan email dari output laporan harian menggunakan cron atau systemd timers.

### Membersihkan filesystem

kita dapat menggunakan cron atau systemd timers untuk menjalankan script yang akan membersihkan filesystem. Misalnya menggunakan script untuk menghapus isi dari direktori trash tiap tengah malam

### Rotasi log file

Membagi log files menjadi beberapa bagian berdasarkan ukuran atau tanggal untuk memastikan beberapa old version dari log tersedia kapanpun.

### Menjalankan batch jobs

Menjalankan scipt yang memproses data dalam jumlah besar. Contohnya messages bisa terakumulasi dalam queue atau database. Kita dapat menggunakan cron job untuk memproses semua queued message sebagai ETL (Extract, Transform, Load) ke lokasi yang berbeda.

### Backup dan mirroring

Kita dapat menjadwalkan task untuk secara otomatis melakukan back up ke sebuah direktori untuk remote system. Kita bisa menggunakan Mirror untuk melakukan backup ke system lain yang berbeda. kita bisa menggunakan periodic execution dari rsync untuk memastikan mirror up to date.
