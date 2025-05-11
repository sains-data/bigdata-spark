# Spark Dataset

Dalam ekosistem Apache Spark, **DataFrame dan Dataset** merupakan API tingkat tinggi yang dibangun di atas RDD. Pengguna Spark umumnya lebih memilih menggunakan DataFrame dan Dataset karena keduanya lebih efisien dalam menyimpan data serta dioptimalkan secara otomatis oleh Spark. Hal ini membuat proses pengolahan data menjadi lebih cepat dan optimal. Selain itu, karena bentuk dan konsepnya mirip dengan tabel dalam database relasional, banyak teknisi yang memiliki latar belakang SQL dan RDBMS bisa dengan cepat beradaptasi dan memanfaatkan kemampuan Spark menggunakan DataFrame dan Dataset.

## Apa Itu Spark Dataset?

**Dataset** adalah kumpulan objek spesifik domain yang bertipe kuat (strongly typed), yang bisa diproses secara paralel menggunakan operasi fungsional atau relasional. Dataset diperkenalkan sejak Spark versi 1.6 sebagai solusi terhadap beberapa keterbatasan yang ada di DataFrame. Kemudian, pada Spark 2.0, API DataFrame dan Dataset digabung ke dalam satu API tunggal: **Dataset API**. Jadi, sebenarnya DataFrame adalah bentuk khusus dari Dataset dengan tipe data `Row` (yaitu `Dataset[Row]`).

Dataset menggabungkan keunggulan dari RDD seperti keamanan tipe data saat kompilasi (compile-time type safety) dan dukungan untuk fungsi lambda, serta keunggulan dari DataFrame seperti optimalisasi kueri secara otomatis menggunakan SQL. Fitur keamanan tipe saat kompilasi ini hanya tersedia dalam bahasa terkompilasi seperti Java dan Scala, sehingga Dataset hanya tersedia di dua bahasa tersebut—tidak tersedia untuk Python (PySpark) maupun R (SparkR).

## Cara Membuat Spark Dataset

Dataset bisa dibuat dengan berbagai cara, yaitu:

1. Dari **urutan elemen** biasa menggunakan fungsi `toDS()`.
2. Dari **urutan case class** (khusus Scala).
3. Dari **RDD**.
4. Dari **DataFrame**.

Contohnya, kita bisa membuat Dataset dari list biasa seperti daftar angka atau string, lalu mengubahnya menjadi Dataset dengan `toDS()`.

## Adaptive Query Execution (AQE)

Pada setiap versi baru, Spark terus memperkenalkan cara baru untuk meningkatkan performa dalam eksekusi kueri. Di Spark 1.x, diperkenalkan **Catalyst Optimizer** dan **Tungsten Execution Engine**. Spark 2.x menambahkan **Cost-Based Optimizer**. Kemudian di Spark 3.0, dikenalkan **Adaptive Query Execution (AQE)**.

Sebelum Spark 3.0, Spark menyusun rencana eksekusi kueri (query execution plan) di awal, dan rencana ini tidak bisa diubah meskipun saat proses berjalan ditemukan kondisi yang berbeda dari perkiraan. AQE yang diperkenalkan di Spark 3.0 memungkinkan Spark untuk **mengadaptasi rencana eksekusi secara dinamis** berdasarkan data dan statistik yang ditemukan saat proses eksekusi berlangsung. Hal ini memberikan fleksibilitas dan efisiensi yang jauh lebih tinggi dibanding pendekatan lama yang statis. Cara kerja AQE ditunjukkan pada Gambar di bawah ini:

![](https://github.com/sains-data/bigdata-spark/blob/main/Materi%20Spark/Picture/adaptive-query-execution.jpg)

---

## Penentuan Jumlah Partisi Shuffle Berdasarkan Data

Secara default, Apache Spark menetapkan jumlah partisi untuk proses *shuffle* (seperti *join* dan *aggregation*) sebanyak **200 partisi**. Namun, angka ini belum tentu ideal untuk semua situasi, terutama saat menangani dataset berukuran besar. Proses shuffle sangat mempengaruhi performa karena melibatkan pemindahan data antar node di dalam kluster Spark. Bila jumlah partisi terlalu sedikit, maka data yang ditampung per partisi bisa terlalu besar untuk dimuat di memori, dan akhirnya ditulis ke disk, yang akan memperlambat kinerja. Sebaliknya, jika jumlah partisi terlalu banyak, maka akan terjadi banyak operasi I/O yang kecil dan pekerjaan Spark akan semakin berat karena terlalu banyak task kecil yang harus dijalankan.

Melalui **Adaptive Query Execution (AQE)**, Spark dapat menyesuaikan jumlah partisi **secara otomatis saat runtime** berdasarkan statistik data yang dikumpulkan antar tahapan. Kita juga bisa memberikan perkiraan jumlah partisi berdasarkan kapasitas komputer atau jumlah data yang diproses, dan Spark akan menyempurnakannya selama proses berlangsung. Ilustrasi AQE dapat dilihat pada Gambar di bawah ini.

![](https://github.com/sains-data/bigdata-spark/blob/main/Materi%20Spark/Picture/AQE.jpg)

---

## Penyesuaian Strategi Join Saat Runtime

Saat melakukan operasi *join*, Spark memiliki beberapa strategi yang bisa digunakan, seperti **sort-merge join** dan **broadcast hash join**. Strategi join yang paling efisien biasanya adalah broadcast join, yaitu saat salah satu sisi tabel cukup kecil untuk dimuat ke dalam memori. Dengan AQE, Spark dapat memilih strategi join terbaik **secara otomatis saat eksekusi**, bukan hanya saat rencana awal disusun. Misalnya, Spark bisa memulai dengan sort-merge join, tetapi kemudian menggantinya ke broadcast join jika mendeteksi bahwa salah satu tabel cukup kecil untuk dibroadcast. Strategi Join Saat Runtime dapat dilihat pada Gambar di bawah ini.

![](https://github.com/sains-data/bigdata-spark/blob/main/Materi%20Spark/Picture/dynamic-join.jpg)

---

### Optimasi Join pada Data yang Tidak Merata (Skewed Data)

Pada praktik, distribusi data seringkali tidak merata. Misalnya, saat melakukan join berdasarkan kolom tertentu, bisa jadi ada satu nilai kunci yang muncul jauh lebih sering dibandingkan nilai lainnya. Hal ini disebut **data skew**. Saat Spark memproses data seperti ini, partisi yang memuat kunci tersebut akan berisi data jauh lebih banyak, membuat eksekusi menjadi lambat, CPU kurang efisien, bahkan bisa menimbulkan error karena kehabisan memori.

Berdasarkan konsep AQE, Spark bisa **mendeteksi data skew** berdasarkan statistik dari proses shuffle, lalu membagi partisi besar menjadi beberapa partisi kecil agar bisa diproses lebih cepat dan merata. Untuk mengaktifkan fitur ini, kamu perlu memastikan bahwa dua konfigurasi diaktifkan:

```bash
spark.sql.adaptive.enabled=true
spark.sql.adaptive.skewJoin.enabled=true
```

ilustrasi positif skew dapat dilihat pada Gambar di bawah ini.

![](https://github.com/sains-data/bigdata-spark/blob/main/Materi%20Spark/Picture/positive-skew.jpg)


---

### Cara Mengaktifkan AQE

Secara default, AQE **tidak aktif** di Spark versi 3.0. Untuk menggunakannya, kamu harus mengaktifkan konfigurasi berikut:

```bash
spark.sql.adaptive.enabled=true
```

Namun, perlu diperhatikan bahwa AQE hanya bekerja untuk query **non-streaming**, dan untuk operasi yang melibatkan pertukaran data seperti **join**, **aggregasi**, atau **windowing**.

## Penutup

Dataset merupakan bagian dari API tingkat tinggi Spark, bersama dengan DataFrame. Namun, berbeda dengan DataFrame yang tersedia untuk berbagai bahasa, Dataset hanya dapat digunakan pada bahasa pemrograman terkompilasi seperti Java dan Scala. Hal ini memberikan keunggulan berupa keamanan tipe data yang lebih ketat (*strongly typed*), namun juga menjadi tantangan karena bahasa-bahasa tersebut memiliki kurva belajar yang lebih curam dibandingkan Python, sehingga penggunaannya lebih terbatas.

Setelah memahami konsep dasar Dataset, pembahasan difokuskan pada **Adaptive Query Execution (AQE)**, salah satu fitur terbaru dan paling menarik yang diperkenalkan di Spark versi 3.0. AQE memberikan peningkatan performa eksekusi query secara signifikan karena mampu menyesuaikan *query plan* secara otomatis berdasarkan statistik data yang dikumpulkan saat runtime.
