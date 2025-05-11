# 📘 **Hands-On Spark Shell**

## 🔹 **Spark Shell Utilities**

Apache Spark menyediakan beberapa utilitas shell yang bisa digunakan langsung:

| Shell         | Bahasa           | Perintah        |
| ------------- | ---------------- | --------------- |
| `spark-shell` | Scala            | `$ spark-shell` |
| `pyspark`     | Python (PySpark) | `$ pyspark`     |
| `spark-sql`   | SQL              | `$ spark-sql`   |
| `sparkR`      | R                | `$ sparkR`      |

Selain itu, tersedia:

* `spark-submit` → Jalankan aplikasi mandiri di cluster (Java/Scala/Python/R)
* `run-example` → Jalankan contoh bawaan
* `spark-class` → Jalankan class-level tools

### 🔄 Menggunakan spark-shell (Scala REPL)

REPL: Baca → Evaluasi → Tampilkan

* Ideal untuk eksperimen dan debugging interaktif.
* Otomatis membuat `SparkSession (spark)` dan `SparkContext (sc)`.
* Spark Web UI: [http://localhost:4040](http://localhost:4040)

* Contoh:

```scala
scala> System.getenv("PWD")
```

* Bantuan:

```scala
scala> :help
```

* Untuk runtime options:

```bash
$ spark-shell -h
```

🧰 Contoh spark-shell dengan konfigurasi lengkap:

```bash
$ spark-shell \
--master yarn \
--deploy-mode cluster \
--driver-memory 16g \
--executor-memory 32g \
--executor-cores 4 \
--conf "spark.sql.shuffle.partitions=1000" \
--conf "spark.executor.memoryOverhead=4024" \
--conf "spark.memory.fraction=0.7" \
--conf "spark.memory.storageFraction=0.3" \
--packages org.apache.hudi:hudi-spark3.3-bundle_2.12:0.12.0 \
--conf "spark.serializer=org.apache.spark.serializer.KryoSerializer" \
--conf "spark.sql.catalog.spark_catalog=org.apache.spark.sql.hudi.catalog.HoodieCatalog" \
--conf "spark.sql.extensions=org.apache.spark.sql.hudi.HoodieSparkSessionExtension"
```

> Jika kamu hanya testing di container tanpa cluster manajer, gunakan `--master local[*]`.


### 🐍 Menggunakan pyspark

```bash
$ pyspark
```

* Sama seperti `spark-shell`, akan otomatis menyediakan `spark` dan `sc`.
* Spark Web UI: [http://localhost:4040](http://localhost:4040)

* Untuk keluar:

```python
>>> quit()
>>> exit()
# atau Ctrl+D
```

* Menjalankan dengan konfigurasi:

```bash
$ pyspark \
--master yarn \
--deploy-mode client \
--executor-memory 16G \
--executor-cores 8 \
--conf spark.sql.parquet.mergeSchema=true \
--conf spark.sql.parquet.filterPushdown=true
```

Menggunakan `spark-submit`

```bash
$ spark-submit --master local[*] app.py
```

* Menampilkan semua opsi:

```bash
$ spark-submit --help
```


## 📦 Opsi Penting:

⚙️ Mode Deploy dan Cluster Manager

| Mode                    | Perintah / Nilai                        |
| ----------------------- | --------------------------------------- |
| `--deploy-mode client`  | Driver di mesin local (default, dalam container) |
| `--deploy-mode cluster` | Driver di node cluster                  |
| `--master yarn`         | Jalankan di atas YARN                   |
| `--master local[*]`     | Jalankan secara lokal                   |

| Opsi            | Keterangan                                                          |
| --------------- | ------------------------------------------------------------------- |
| `--master`      | Tentukan cluster manager (lihat tabel di bawah)                     |


📊 Cluster Manager:

| Manager    | Template                        | Keterangan                           |
| ---------- | ------------------------------- | ------------------------------------ |
| Standalone | `spark://host:7077`             | Default port 7077                    |
| Mesos      | `mesos://host:port`             | Untuk cluster Mesos                  |
| YARN       | `--master yarn`                 | Dijalankan dengan Hadoop YARN        |
| Kubernetes | `k8s://https://host:port`       | Menggunakan API Kubernetes           |
| Local      | `local`, `local[K]`, `local[*]` | Menjalankan lokal sesuai jumlah core |

> contoh penggunaan: `--master spark://host:7077`, `--master yarn`, `--master k8s://https://host:port`, dll


## Submit Spark Job dari Container

```bash
# Jalankan contoh SparkPi secara lokal menggunakan 4 thread
spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master local[4] \
  $SPARK_HOME/examples/jars/spark-examples_2.12-3.5.5.jar 80
```

```bash
# Jalankan Python Spark job ke Spark standalone cluster
spark-submit \
  --master spark://localhost:7077 \
  $SPARK_HOME/examples/src/main/python/pi.py
```

```bash
# Submit ke Spark standalone cluster (client mode) dengan 100 cores
spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://localhost:7077 \
  --executor-memory 20G \
  --total-executor-cores 100 \
  $SPARK_HOME/examples/jars/spark-examples_2.12-3.5.5.jar 80
```

```bash
# Submit ke Spark standalone cluster (cluster mode) dengan supervise
spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://localhost:7077 \
  --deploy-mode cluster \
  --supervise \
  --executor-memory 20G \
  --total-executor-cores 100 \
  $SPARK_HOME/examples/jars/spark-examples_2.12-3.5.5.jar 80
```

```bash
# Submit ke YARN cluster (cluster mode)
export HADOOP_CONF_DIR=/etc/hadoop/conf
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --class org.apache.spark.examples.SparkPi \
  --executor-memory 10G \
  --num-executors 20 \
  $SPARK_HOME/examples/jars/spark-examples_2.12-3.5.5.jar 80
```

```bash
# Submit ke Mesos cluster
spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master mesos://207.184.161.138:7077 \
  --deploy-mode cluster \
  --executor-memory 16G \
  --total-executor-cores 64 \
  $SPARK_HOME/examples/jars/spark-examples_2.12-3.5.5.jar 80
```

```bash
# Submit ke Kubernetes cluster
spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master k8s://xx.yy.zz.ww:443 \
  --deploy-mode cluster \
  --executor-memory 20G \
  --num-executors 50 \
  $SPARK_HOME/examples/jars/spark-examples_2.12-3.5.5.jar 80
```

Berikut adalah **catatan lengkap dan rapi** mengenai *Tuning Resource Allocation* di Apache Spark yang bisa kamu gunakan untuk referensi konfigurasi `spark-submit` dan optimasi performa:

---

## 🧠 **Tuning Resource Allocation in Apache Spark**

Ketika menjalankan aplikasi di kluster Spark, penting untuk mengatur alokasi resource seperti CPU dan RAM agar program berjalan lebih cepat dan efisien. Spark memungkinkan pengaturan **jumlah core dan memori** untuk setiap proses driver maupun executor.

Konfigurasi ini sangat berpengaruh tidak hanya terhadap performa Spark, tetapi juga terhadap kesesuaian resource yang tersedia di cluster manager (seperti YARN, Kubernetes, atau Standalone).


⚙️ **Tabel Konfigurasi Resource Allocation (Tabel 2-3)**

| **Option**               | **Deskripsi**                                                                                                                                                                                                           |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--executor-cores`       | - Jumlah CPU core yang digunakan untuk tiap proses executor.  <br> - Mempengaruhi jumlah *concurrent tasks* per executor. <br> - Contoh: `--executor-cores 5` artinya tiap executor dapat menjalankan 5 task sekaligus. |
| `--executor-memory`      | - Jumlah RAM yang digunakan untuk tiap proses executor. <br> - Mempengaruhi ukuran maksimum cache dan data shuffle. <br> - Berdampak pada performa operasi seperti `groupBy`, `join`, dll.                              |
| `--num-executors`        | - Jumlah executor yang diminta untuk menjalankan aplikasi. <br> - Contoh: `--num-executors 20` artinya Spark akan mencoba meluncurkan 20 executor. (*Tersedia di YARN dan beberapa mode cluster lain*)                  |
| `--driver-memory`        | - Jumlah RAM yang digunakan oleh driver Spark. <br> - Contoh: `--driver-memory 4G`                                                                                                                                      |
| `--driver-cores`         | - Jumlah CPU core yang dialokasikan untuk driver Spark. <br> - Contoh: `--driver-cores 2`                                                                                                                               |
| `--total-executor-cores` | - Jumlah total core untuk seluruh executor. <br> - Contoh: `--total-executor-cores 100` (akan membagi ke beberapa executor tergantung konfigurasi).                                                                     |

## 💡 **Tips Praktis**

* Sesuaikan konfigurasi di atas berdasarkan **ukuran data** dan **kapasitas hardware** cluster-mu.
* Jika terlalu kecil, aplikasi akan lambat dan tidak efisien.
* Jika terlalu besar, bisa menyebabkan **resource contention** atau **out-of-memory error**.

## 🧠 **Dynamically Loading Spark Submit Configurations**

Untuk meningkatkan fleksibilitas dan menghindari penguncian terhadap konfigurasi tertentu, disarankan **tidak menulis konfigurasi Spark secara hard-coded** ke dalam kode sumber. Hal ini sangat penting terutama jika aplikasi dijalankan di berbagai lingkungan klaster yang memiliki konfigurasi berbeda-beda.

❌ **Contoh Hard-Coded Configuration (Kurang Disarankan)**

- Scala

```scala
val conf = new SparkConf()
  .setMaster("local[4]")
  .setAppName("Hands-On Spark 3")
  .set("spark.executor.memory", "32g")
  .set("spark.driver.memory", "16g")

val sc = new SparkContext(conf)
```

- PySpark

```python
from pyspark import SparkConf, SparkContext

conf = SparkConf()
conf.setMaster("spark://localhost:7077")
conf.setAppName("Hands-On Spark 3")
conf.set("spark.executor.memory", "32g")
conf.set("spark.driver.memory", "16g")

sc = SparkContext(conf=conf)
```

✅ **Solusi Fleksibel: Gunakan `spark-submit` dengan Parameter Dinamis**

- Scala (void config)

```scala
val sc = new SparkContext(new SparkConf())
```

- PySpark (void config)

```python
from pyspark import SparkConf, SparkContext

conf = SparkConf()
sc = SparkContext(conf=conf)
```

- SparkR

```r
library(SparkR)
sparkR.session()
```

## 🧪 **Menjalankan Spark Secara Dinamis dengan spark-submit**

```bash
spark-submit \
--name "Hands-On Spark 3" \
--master local[4] \
--deploy-mode client \
--conf spark.eventLog.enabled=false \
--conf "spark.executor.extraJavaOptions=-XX:+PrintGCDetails -XX:+PrintGCTimeStamps" \
--class org.apache.spark.examples.SparkPi \
/$SPARK_HOME/examples/jars/spark-examples_2.12-3.5.5.jar 80
```

> Gantilah nama JAR dan versinya sesuai yang digunakan (`3.5.5`).


🧩 Contoh Konfigurasi Tambahan via spark-submit

```bash
--conf spark.sql.shuffle.partitions=100 \
--conf spark.sql.adaptive.enabled=true \
--conf spark.dynamicAllocation.enabled=true \
--conf spark.dynamicAllocation.minExecutors=1 \
--conf spark.dynamicAllocation.maxExecutors=10
```

**Properti Aplikasi Spark**

| Properti              | Deskripsi                                                                                                                                                     |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `spark.app.name`      | Nama aplikasi Spark (default: tidak ada).                                                                                                                     |
| `spark.driver.cores`  | Jumlah core yang digunakan oleh proses driver dalam *cluster mode* (default: 1).                                                                              |
| `spark.driver.memory` | Jumlah memori untuk driver (default: 1g). Untuk *client mode*, dapat disetel menggunakan opsi `--driver-memory` pada command line atau melalui file properti. |

**Lingkungan Runtime Spark**

| Properti                      | Deskripsi                                                                                                                                                                                                           |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `spark.driver.extraClassPath` | Jalur classpath tambahan untuk proses driver (default: tidak ada). Di *client mode*, dapat ditetapkan dengan `--driver-class-path`. Cocok untuk memuat file seperti konektor basis data dan file eksternal lainnya. |


**Alokasi Sumber Daya Spark (Dynamic Allocation)**

| Properti                                            | Deskripsi                                                                             |
| --------------------------------------------------- | ------------------------------------------------------------------------------------- |
| `spark.dynamicAllocation.enabled`                   | Mengaktifkan alokasi sumber daya dinamis berdasarkan beban kerja (default: `false`).  |
| `spark.dynamicAllocation.executorIdleTimeout`       | Waktu tunggu sebelum executor idle dihentikan, dalam detik (default: `60s`).          |
| `spark.dynamicAllocation.cachedExecutorIdleTimeout` | Waktu tunggu executor yang memiliki cache sebelum dihentikan (default: tak terbatas). |


**Properti Spark Lainnya untuk Kontrol Aplikasi**

| Properti                        | Deskripsi                                                                                                                                                   |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `spark.sql.shuffle.partitions`  | Jumlah partisi untuk operasi *shuffle* seperti *join* atau *aggregation* (default: `200`).                                                                  |
| `spark.rdd.compress`            | Mengaktifkan kompresi pada RDD (default: `false`). Menghemat ruang penyimpanan, namun menambah beban CPU.                                                   |
| `spark.executor.pyspark.memory` | Jumlah memori khusus untuk PySpark di setiap executor (default: tidak ditetapkan).                                                                          |
| `spark.executor.memoryOverhead` | Memori tambahan untuk setiap proses executor. Total memori executor adalah jumlah dari beberapa komponen termasuk `memoryOverhead`, `executor.memory`, dll. |


**Contoh Penggunaan di spark-submit**

```bash
spark-submit \
--master yarn \
--deploy-mode cluster \
--conf "spark.sql.shuffle.partitions=10000" \
--conf "spark.executor.memoryOverhead=8192" \
--conf "spark.memory.fraction=0.7" \
--conf "spark.memory.storageFraction=0.3" \
--conf "spark.dynamicAllocation.minExecutors=10" \
--conf "spark.dynamicAllocation.maxExecutors=2000" \
--conf "spark.dynamicAllocation.enabled=true" \
--conf "spark.executor.extraJavaOptions=-XX:+PrintGCDetails -XX:+PrintGCTimeStamps" \
--files /path/of/config.conf,/path/to/mypropeties.json \
--class org.apache.spark.examples.SparkPi \
$SPARK_HOME/examples/jars/spark-examples_2.12-3.3.0.jar 80
```

## 🔥 Spark Application dan SparkSession

Apa Itu SparkSession?

* `SparkSession` adalah **titik masuk utama** (entry point) untuk semua fungsionalitas Spark, termasuk API **Dataset** dan **DataFrame**.
* Diperkenalkan sejak **Spark versi 2.0**, menggantikan kelas-kelas lama seperti:

  * `SparkContext`
  * `SQLContext`
  * `StreamingContext`
  * `HiveContext`

Karakteristik SparkSession

* Kamu bisa memiliki **banyak SparkSession** dalam satu aplikasi Spark.
* Tetapi, kamu hanya bisa memiliki **satu SparkContext per JVM**.
* SparkSession bisa dibuat menggunakan:

  * `SparkSession.builder()`
  * `SparkSession.newSession()` → untuk membuat session baru dari session aktif.

## 🧠 Mengakses SparkSession yang Sudah Ada

- Scala

```scala
import org.apache.spark.sql.SparkSession

val currentSparkSession = SparkSession.builder().getOrCreate()
println(currentSparkSession)
```

- Python (PySpark)

```python
from pyspark.sql import SparkSession

currentSparkSession = SparkSession.builder.getOrCreate()
print(currentSparkSession)
```

Mendapatkan Session Aktif (opsional)

```python
# Python
s = SparkSession.getActiveSession()
print(s)
```

## ✨ Membuat SparkSession Secara Programatik

- Scala

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
  .appName("ContohSparkScala")
  .master("local[*]")
  .getOrCreate()

// Contoh penggunaan
val df = spark.read.json("contoh.json")
df.show()
```

- Python (PySpark)

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("ContohSparkPython") \
    .master("local[*]") \
    .getOrCreate()

# Contoh penggunaan
df = spark.read.json("contoh.json")
df.show()
```

- R (SparkR)

```r
library(SparkR)

sparkR.session(appName = "ContohSparkR", master = "local[*]")

# Contoh penggunaan
df <- read.json("contoh.json")
showDF(df)
```

📌 Catatan Penting

* Gunakan `.getOrCreate()` untuk **mengakses session yang sudah ada**, atau **membuat baru jika belum ada**.
* Gunakan `.newSession()` untuk **membuat session terpisah** dalam satu aplikasi.
* `.appName()` dan `.master()` penting untuk memberi nama aplikasi dan menentukan mode eksekusi lokal/cluster.


## 🧱 Transformations, Actions, Immutability, dan Lazy Evaluation

### 🔐 Immutability

* **RDD (Resilient Distributed Dataset)** dan **DataFrame** di Spark bersifat **immutable** (tidak dapat diubah setelah dibuat).
* Operasi apa pun tidak mengubah struktur aslinya, melainkan membuat **struktur baru**.
* Hal ini menjamin **keamanan dan konsistensi data** dalam komputasi paralel.


### ⚙️ Transformations

* **Transformations** adalah operasi yang **menghasilkan RDD/DataFrame baru** dari input RDD/DataFrame.

Example of a transformations lineage
![](/Picture/transformation-lineage.jpg)

* **Tidak langsung dieksekusi**, tetapi hanya dicatat dalam DAG (Directed Acyclic Graph) — inilah yang disebut **lazy evaluation**.

An example of a Directed Acyclic Graph
![](/Picture/DAG.jpg)

* **Transformations tidak men-trigger eksekusi** hingga sebuah action dipanggil.

🧾 Contoh Fungsi Transformations:

* `map()`
* `flatMap()`
* `filter()`
* `groupByKey()`
* `reduceByKey()`
* `sample()`
* `union()`
* `distinct()`

#### 📊 Jenis Transformations:

| Jenis      | Penjelasan                                                     | Contoh Fungsi                                           |
| ---------- | -------------------------------------------------------------- | ------------------------------------------------------- |
| **Narrow** | Tidak ada pergerakan data antar partisi                        | `map()`, `filter()`, `union()`                          |
| **Wide**   | Melibatkan data shuffling antar partisi (lebih mahal biayanya) | `groupByKey()`, `join()`, `distinct()`, `repartition()` |

- Narrow
![](/Picture/narrow-transformation.jpg)

- Wide
![](/Picture/wide-transformation.jpg)


### 💤 Lazy Evaluation

* Transformasi hanya **dicatat**, bukan langsung dieksekusi.
* Eksekusi baru dimulai ketika **action** dipanggil.
* Keuntungan: Spark bisa **mengoptimasi eksekusi** berdasarkan DAG yang terbentuk.


### 🎯 Actions

* **Actions** adalah operasi yang **mengembalikan hasil akhir** (bukan RDD/DataFrame).
* Saat action dipanggil, **semua transformasi sebelumnya dieksekusi** sesuai urutan dalam DAG.

Contoh Fungsi Actions:

* `collect()` → mengembalikan semua data ke driver
* `count()` → menghitung jumlah elemen
* `first()` → mengembalikan elemen pertama
* `min()` / `max()` → nilai minimum/maksimum
* `top()` → n elemen teratas
* `fold()` / `aggregate()` → menggabungkan data

Tabel konsep dari Transformations, Actions, Immutability, dan Lazy Evaluation

| Konsep          | Penjelasan                                             |
| --------------- | ------------------------------------------------------ |
| Immutability    | Data tidak bisa dimodifikasi setelah dibuat            |
| Transformations | Operasi yang menghasilkan RDD/DataFrame baru           |
| Lazy Evaluation | Transformasi hanya dievaluasi saat action dipanggil    |
| Actions         | Menjalankan transformasi dan mengembalikan hasil akhir |

