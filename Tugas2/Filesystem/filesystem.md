# Filesystem

Tujuan dasar dari filesystem adalah untuk merepresentasikan dan mengorganisasi storage resources dalam sistem.

Filesystem terdiri dari 4 komponen utama:
1. Namespace - cara untuk memberi nama dan mengorganisasi data dalam hierarki
2. API - sekumpulan system call untuk navigasi dan manipulasi objek dalam filesystem
3. Model Keamanan - skema untuk menlindungi, menyembunyikan, dan berbagi data
4. Implementasi - software yang menghubungkan model logis dengan hardware

Filesystem berbasis disk yang umum digunakan meliputi ext4, XFS, dan UFS serta ZFS dari Oracle dan Btrfs. Selain itu, ada juga filesystem lain seperti VxFS dari Veritas dan JFS dari IBM.

Kita juga memiliki beberapa foreign filesystem seperti FAT dan NTFS yang digunakan oleh Windows serta ISO 9660 yang digunakan untuk CD dan DVD.

Sebagian besar filesystem modern berusaha meningkatkan kecepatan dan keandalan dibandingkan filesystem tradisional atau menambahkan fitur tambahan sebagai lapisan di atas standard filesystem sematics.

## Pathnames

Istilah "folder" sebenarnya berasal dari dunia Windows dan macOS. Istilah ini memiliki arti yang sama dengan "directory", tetapi dalam konteks teknis, lebih baik menggunakan istilah "directory" karena lebih formal.

Sekumpulan direktori yang mengarah ke suatu berkas disebut pathname. Pathname adalah string yang mendeskripsikan lokasi berkas dalam hierarki filesystem.
- Pathname absolut - menunjuk ke lokasi penuh dari root, contoh: `/home/username/file.txt`
- Pathname relatif - menunjuk ke lokasi berdasarkan direktori saat ini, contoh: `./file.txt`

## Filesystem Mounting and Unmounting

Filesystem terdiri dari bagian-bagian kecil yang juga disebut "filesystems" dimana masing-masing terdiri dari satu direktori beserta subdirektori dan files didalamnya.

Kit amenggunakan istilah "file tree" untuk merujuk pada keseluruhan struktur filesystem dan menggunakan kata "filesystem" untuk menyebut cabang-cabang dari file tree tersebut.

Dalam kebanyakan situasi, filesystem terpasang dalam file tree menggunakan perintah mount. Perintah mount akan memetakan suatu direktori dalam file tree, yang disebut mount point, ke root dari filesystem yang baru.

Contoh:
`mount /dev/sda4 /users`

Linux memiliki opsi lazy unmount dengan perintah `umount -l` yang akan menghapus filesystem dari hierarki nama, tetapi tidak benar-benar melakukan unmount hingga filesystem tersebut tidak digunakan lagi.

Jika filesystem sedang sibuk, kita bisa menggunakan opsi forceful unmount dengan perintah `umount -f`. Perintah ini akan memaksa unmount, tetapi sebaiknya digunakan dengan hati-hati karena dapat menyebabkan kehilangan data atau ketidakstabilan sistem.

Daripada langsung menggunakan umount -f, lebih baik cari dulu proses mana yang masih menggunakan file system dengan `lsof` atau `fuser` baru menghentikan proses tersebut sebelum melakukan unmount.

untuk menginvestigasi proses yang menggunakan filesystem, kita bisa juga menggunakan `ps`

## Oraganization of the file tree

Root filesystem setidaknya mencakup direkori root (/) dan sekumpulan file serta subdirektori minimal
- Kernel OS biasanya berada di /boot, tetapi lokasi dan namanya bisa bervariasi. Pada BSD dan beberapa sistem UNIX lainnya, kernel bukan hanya satu file tunggal, melainkan sekumpulan beberapa komponen.
- /etc menyimpan file sistem dan konfigurasi yang penting
- /sbin dan /bin berisi utilitas penting
- /tmp kadang digunakan untuk file sementara
- /dev dulunya merupakan bagian dari root filesystem, tetapi sekarangmerupakan virtual filesystem yang dipasang secara terpisah.

Beberapa sistem menyimpan file library bersama (shared libraries) di /lib atau /lib64, sementara sistem lain telah memindahkannya ke /usr/lib, dengan /lib sering kali hanya berupa symbolic link

- /user adalah tempat sebagian besar program standar tetapi tidak kritis untuk sistem disimpan. Direktori ini juga berisi dokumentasi online dan library. FreeBSD menyimapn banyak konfigurasi lokal di /usr/local
- /var berisi direktori spool, file log, informasi accounting, dan berbagai item lain yang sering berubah atau berkembang di setiap host.

Baik /usr maupun /var harus tersedia agar sistem bisa boot hingga mode multiuser.

## File Types

Sebagian besar implementasi filesystem mendefinisikan 7 jenis file:
1. Regilar files
2. Directories
3. Character device files
4. Block device files
5. Local domain sockets
6. Named popes (FIFOs)
7. Symbolic links

Kita dapat mengetahui jenis suatu file dengan perintah `file`

Atau gunakan ls -ld untuk melihat informasi tentang direktori tanpa menampilkan isinya.

### Regular files

Regular files terdiri dari deretan byte tanpa struktur khusus. Contohnya file teks, file data, program eksekusi, dan shared libraries.

### Directories

Directories adalah referensi yang menunjuk ke file lain dalam sistem.

### Hard Links

Hard link memungkinkan satu file memiliki beberapa nama. Untuk membuathard link ke file yang ada, kita bisa menggunakan perintah `ln`. Gunakan opsi -i pada ls untuk melhiat jumlah hard link yang dimiliki suatu file

contoh:
`ln /etc/passwd /tmp/passwd`

### Character and Block Devices Files

Devices files memungkinkan program berkomunikasi dengan hardware dan sytem peripherals. Kernel memiliki (atau memuat) driver untuk setiap perangkat dalam system. Driver ini menangani detail teknis pengelolaan perangkat sehingga kernel tetap abstrak dan independen terhadap perangkat keras.

Perbedaan antara character dan block device cukup teknis dan tidak terlalu penting untuk dibahas secara mendetail.

Setiap device file memiliki major dan minor device number:
- Major number mengidentifikasi driver yang mengontrol perangkat
- Minor number menentukan unit fisik yang akan diakses oleh driver

Contoh:
Pada sistem Linux, major device number 4 menandakan serial driver.
- /dev/tty0 -> Major : 4, Minor : 0 (port serial pertama)
- /dev/tty1 -> Major : 4, Minor : 1 (port serial kedua)

Dulu, `/dev` adalah direktori biasa, dan perangkat dibuat secara manual dengan `mknod` serta dihapus dengan `rm`. Namun, cara ini menjadi tidak praktis karena banyaknya jenis perangkat baru yang muncul. Sekarang `/dev` biasanya dipasang sebagai filesystem khusus, dimana isinya dikelola otomatis oleh kernel bersama user-level daemon.

### Local Domain Sockets

Local domain sockets digunakan untuk komunikasi antar-proses dalam satu sistem. Mirip dengan network sockets , tetapi terbatas dalam host lokal. Contoh penggunaannya adalah `syslog` dan `X Window System`

### Named Pipes (FIFOs)

Sama seperti local domain sockets, Named pipes memungkinkan komunikasi antar-proses dalam satu host. Berbeda dengan anonymous pipes, karena named pipes dapat diakses oleh proses yang tidak memiliki hubungan langsung satu sama lain.

### Symbolic Links

Symbolic links atau soft links adalah referensi ke file lain berdasarkan nama. Soft links lebih fleksibel dibandingkan hard links, bisa menunjuk ke file di filesystem yang berbeda, bisa menunjuk ke direktori, tidak seperti hard links.

Contoh penggunaannya adalah sistem yang memiliki /usr/bin sebagai symbolic link ke /bin untuk menghemat ruang di root filesystem dan memudahkan berbagi software di beberapa host.

## File attributes

setiap file dalam sistem UNIX/Linux memiliki seperangkat atribut yang menentukan siapa yang dapat membaca, menulis, dan mengeksekusi file tersebut. Atribut ini dikenal sebagai permission bits dan merupakan bagian dari file mode.

Selain 9 permision bits, ada 3 mode bits tambahan yang terutama berpengaruh pada file eksekusi, menjadikan totalnya 12 mode bits. Mode bits ini disimpan bersama 4 file-type bits, yang menentukan jenis file dan hanya bisa diatur saat file dibuat.

Superuser dan pemilik file bisa mengubah mode bits menggunakan perintah `chmod`

### Permission Bits

Permision bits terdiri dari 3 group, masing masing terdiri dari 3 bit:
1. Pemilik file (user - u)
2. Grup pemilik (group - g)
3. Pengguna lain (others - o)

Karena setiap grup memiliki 3 bit, kita bisa menggunakan notasi oktal (basis 8) untuk menyederhanakan izin file

### Arti dari setiap bit

- Read (r) -> Bisa membaca file
- Write (w) -> Bisa mengedit atau menghapus isi file
- Execute (x) -> Bisa mengeksekusi file jika berupa binary atau script

Namun izin menghapus atau mengganti nama fie tidak ditentukan oelh izin file itu sendiri, melainkan oleh izin direktori tempat file itu berada.

Terdapat 2 jenis file eksekusi:
- Binary executable -> langsung dijalankan oleh CPU, seperti program compiled
- Script executable -> harus diinterpretasi oleh program lain seperti shell atau python.

Script biasanya diawali dengan shebang line seperti :
`#!/usr/bin/perl`

File eksekusi non-binary yang tidak memiliki shebang akan diasumsikan sebagai script shell (sh script). Jernel mengenali dan menangani shebang secara langsung. Jika interpreter tidak ditentukan dengan benar, kernel akan menolak menjalankan file. Jika kernel gagal, shell akan mencoba mengeksekusi file tersebut sebagai script shell (sh)

### setuid and setgid bits

- Setuid (octal: 4000)

Jika set pada file eksekusi, file akan dijalankan dengan hak akses pemilik file, bukan pengguna yang mengeksekusi file. COntohnya `/usr/bin/passwd` yang mengizinkan pengguna biasa untuk mengubah password mereka.

- Setgid (octal: 2000)

Jika set pada file eksekusi, file akan dijalankan dengan hak akses group file. JIka set pada direktori, semua file baru yang dibuat dalam direktori tersebut akan memiliki grup yang sama dengan direktori.

### The sticky bit (octal: 1000)

Jika set pada direktori, hanya pemilik file (atau root) yang bisa menghapus atau mengganti nama file di dalam direktori tersebut. Sticky bit digunakan untuk direktori bersama seperti `/tmp`

### ls: list and inspect files

Perintah `ls` digunakan juga untuk melihat daftar file dan atributnya.
Dengan menggunakan opsi -l, `ls` akan menampilkan dengan format yang lebih panjang yang mencakup, mode file, jumlah hard link, pemilik file, group pemilik, ukuran file dalam byte, dan waktu modifikasi terakhir.

Setiap direktori memiliki minimal 2 hard link: `.` yaitu hard link ke direktori itu sendiri dan `..` yaitu hard link ke parent directory.

### chmod: change permissions

Perintah `chmod` digunakan untuk mengubah mode dari sebuah file. Kita dapat menggunakan octal notation atau dengan symobilc notation

### chown: change ownership

Perintah `chown` digunakan untuk mengubah pemilik dan group dari sebuah file. Opsi -R dapat digunakan pada chown untuk mengubah kepemilikan dari konten sebuah file secara reskursif

### chgrp: change group

perintah `chgrp` digunakan untuk mengubah group dari sebuah file. Sama seperti chown, perintah ini juga memiliki opsi -R yang akan mengubah griooup dari konten sebuah file secara rekursif

### umask: set default permission

perintah `umask` menentukan default permission untuk file baru dan direktori yang dibuat. Perintah `umask` merupakan bit mask yang dikurangi dari default permission untuk menentukan perizian sebenarnya.

contoh:
`umask 022`

sebagai contoh, `umask 027` akan mengijinkan `rwx` untuk pemilik, rx ke group, dan pengguna lain tidak mendapatkan izin.

## Access Control Lists

Model perizinan tradisional UNIX sederhana dan efektif, tetapi memiliki keterbatasan dalam kesulitan memberikan banyak pemilik pada satu file dan sulit memberikan izin berbeda untuk kelompok pengguna yang berbeda pada file yang sama

Access Control LIsts (ACLs) adalah solusi untuk memperluas model izin tradisional UNIX
- ACL memungkinkan file memiliki banyak pemilik
- ACL memungkinkan kelompok pengguna memiliki izin berbeda pada file yang sama.

Setiap aturan dalam ACL disebut Accress Control Entry (ACE). 
Struktur dari ACE:
1. User atau group spesifik
2. Permission mask
3. Type

Perintah `getfacl` akan menampilkan ACL dari sebuah file dan perintah setfacl menentukan ACL dari sebuah file.
`getfacl /etc/passwd`
`setfacl -m u:abdou:rw /etc/passwd`

Terdapat 2 jenis ACL: POSIX ACLs dan NFSv4 ACLs. POSIX ACLs meruapakan model tradisional ACL di UNIX/Linux. Sedangkan NFSv4 ACLs adalah model ACL yang lebih baru dan lebih kuat.

### Implementation of ACLs

Implementasi ACLs bisa dilakukan di berbagai level, misalnya:
- Kernel -> Mengelola ACL untuk semua filesystem
- Filesystem -> Setiap sistem file dapat menerapkan ACCL sendiri
- High-level software -> Seperti server NFS atau SMV yang mendukung ACL secara spesifik.

### POSIX ACLs

POSIX ACLs merupakan ACL tradisional UNIX yang didukung oleh kebanyakan UNIX-like OS seperti Linux, FreeBSD, dan Solaris.

### NFSv4 ACLs

NFSv4 ACLs adalah tipe ACL yang lebih baru dan lebih powerful. NFSv4 didukung oleh kebanyakan UNIX-like OS seperti Linux dan FreeBSD.

NFSv4 ACLs mirip dengan POSIX ACLs, namun NFSv4 memiliki beberapa fitur tambahan seperti default ACLs yang bisa digunakan untuk untuk mengatur ACL dari file dan direktori baru.
