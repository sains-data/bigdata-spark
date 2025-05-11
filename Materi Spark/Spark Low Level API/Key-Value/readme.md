# Bekerja dengan Key-Value Pairs

## Pendahuluan

Pair RDD adalah RDD yang berisi data dalam format key-value. Operasi khusus ini tersedia melalui class `PairRDDFunctions`.

## a. Membuat Pair RDD

Pair RDD merupakan fondasi penting dalam pemrograman Spark yang memungkinkan operasi berbasis kunci. Proses pembentukannya melibatkan transformasi RDD biasa menjadi format key-value melalui operasi map. Dalam Scala, struktur tuple secara otomatis dikenali sebagai pair RDD, sedangkan di Python menggunakan tuple biasa. Transformasi ini mempertahankan sifat immutable RDD asli sambil menambahkan kemampuan operasi khusus key-value. Pemilihan kunci yang tepat sangat krusial karena akan menentukan efektivitas operasi agregasi dan pengelompokan data.

**Scala:**
```scala
// Membuat RDD biasa
val data = sc.parallelize(List("apel", "jeruk", "apel", "mangga", "jeruk"))

// Mengubah menjadi Pair RDD (key: buah, value: 1)
val pairRDD = data.map(buah => (buah, 1))
pairRDD.foreach(println)
/*
Output:
(apel,1)
(jeruk,1)
(apel,1)
(mangga,1)
(jeruk,1)
*/
```

**Python:**
```python
data = sc.parallelize(["apel", "jeruk", "apel", "mangga", "jeruk"])
pair_rdd = data.map(lambda buah: (buah, 1))
pair_rdd.foreach(print)
```

## b. Menampilkan Key Unik

Operasi distinct pada kunci pair RDD memberikan gambaran unik tentang distribusi data. Fungsi ini bekerja dengan melakukan hashing pada semua kunci dan menghapus duplikat, mirip dengan operasi DISTINCT di SQL. Proses ini memerlukan shuffle operation di balik layar sehingga perlu digunakan dengan bijak pada dataset besar. Hasilnya sangat berguna untuk analisis awal distribusi data, memahami cardinality kunci, atau memeriksa data quality.

**Scala:**
```scala
val keys = pairRDD.keys.distinct()
keys.foreach(println)
/*
Output:
apel
jeruk
mangga
*/
```

**Python:**
```python
keys = pair_rdd.keys().distinct()
keys.foreach(print)
```

## c. Transformasi pada Pair RDD

operasi transformasi yang dijalankan pada RDD mengembalikan satu atau beberapa RDD baru tanpa mengubah yang asli, sehingga menciptakan garis keturunan RDD, yang digunakan oleh Spark untuk mengoptimalkan eksekusi kode dan memulihkan dari kegagalan. Apache Spark memanfaatkan garis keturunan RDD untuk membangun kembali partisi RDD yang hilang. Representasi grafis dari garis keturunan RDD atau grafik ketergantungan RDD dapat dilihat pada Gambar di bawah ini.

![](https://github.com/sains-data/bigdata-spark/blob/main/Materi%20Spark/Picture/transformation-rdd.jpg)

### 1. Mengurutkan Pair RDD by Key

Sorting merupakan operasi berat di Spark yang melibatkan pembagian ulang seluruh data berdasarkan kunci. Proses ini menggunakan range partitioning untuk mendistribusikan data secara merata. Parameter ascending mengontrol urutan sorting, sementara numPartitions memengaruhi paralelisme operasi berikutnya. Hasil sorting sangat penting untuk operasi seperti range query atau persiapan data sebelum disimpan ke sistem terurut.

**Scala:**
```scala
val sortedRDD = pairRDD.sortByKey()
sortedRDD.foreach(println)
/*
Output:
(apel,1)
(apel,1)
(jeruk,1)
(jeruk,1)
(mangga,1)
*/
```

**Python:**
```python
sorted_rdd = pair_rdd.sortByKey()
sorted_rdd.foreach(print)
```

### 2. Menjumlahkan Nilai by Key

ReduceByKey menawarkan optimasi signifikan dibanding pendekatan naif dengan melakukan partial aggregation di masing-masing partisi sebelum pertukaran data antar worker node. Pola ini mengurangi volume data yang perlu di-shuffle secara dramatis. Operasi ini menjadi tulang punggung banyak algoritma analitik seperti word count, agregasi metrik, atau perhitungan statistik sederhana.

**Scala:**
```scala
val countByFruit = pairRDD.reduceByKey(_ + _)
countByFruit.foreach(println)
/*
Output:
(apel,2)
(jeruk,2)
(mangga,1)
*/
```

**Python:**
```python
count_by_fruit = pair_rdd.reduceByKey(lambda a, b: a + b)
count_by_fruit.foreach(print)
```

### 3. Menyimpan RDD sebagai File Teks

Mekanisme penyimpanan Spark dirancang untuk fault-tolerant dengan menulis output ke partisi terpisah. Setiap eksekutor menulis bagian datanya sendiri secara paralel, lalu menandai penyelesaian dengan _SUCCESS file. Format kompresi seperti gzip atau snappy bisa diaplikasikan untuk menghemat ruang penyimpanan. Perlu diperhatikan bahwa path yang dimaksud sebenarnya adalah direktori, bukan file tunggal.

**Scala:**
```scala
countByFruit.saveAsTextFile("hdfs:///output/jumlah_buah")
```

**Python:**
```python
count_by_fruit.saveAsTextFile("hdfs:///output/jumlah_buah")
```

### 4. Menggabungkan Data di Setiap Partisi

AggregateByKey memberikan fleksibilitas lebih dibanding reduceByKey dengan memisahkan logika aggregasi dalam partisi dan antar partisi. Fungsi ini memungkinkan tipe output berbeda dari tipe nilai asli, membuka kemungkinan untuk implementasi algoritma kompleks seperti perhitungan rata-rata atau akumulasi berbasis koleksi. Desain ini sangat efisien untuk skenario agregasi multi-tahap.

**Scala:**
```scala
val initialCount = 0
val addToCounts = (n: Int, v: Int) => n + v
val sumPartitionCounts = (p1: Int, p2: Int) => p1 + p2
val aggregateRDD = pairRDD.aggregateByKey(initialCount)(addToCounts, sumPartitionCounts)
aggregateRDD.foreach(println)
```

**Python:**
```python
initial_count = 0
add_to_counts = lambda n, v: n + v
sum_partition_counts = lambda p1, p2: p1 + p2
aggregate_rdd = pair_rdd.aggregateByKey(initial_count, add_to_counts, sum_partition_counts)
aggregate_rdd.foreach(print)
```

### 5. Menggabungkan Nilai dengan ZeroValue

FoldByKey menyediakan mekanisme agregasi yang aman dengan nilai inisialisasi yang netral secara matematis. Nilai awal ini berfungsi sebagai basis akumulasi dan dijamin tidak mempengaruhi hasil akhir. Pendekatan ini sangat berguna ketika menghadapi partisi kosong atau memastikan konsistensi hasil saat data terdistribusi tidak merata.

**Scala:**
```scala
val foldRDD = pairRDD.foldByKey(0)(_ + _)
foldRDD.foreach(println)
/*
Output sama seperti reduceByKey
*/
```

**Python:**
```python
fold_rdd = pair_rdd.foldByKey(0, lambda a, b: a + b)
fold_rdd.foreach(print)
```

### 6. Menggabungkan Elemen dengan Fungsi Kustom

CombineByKey merupakan primitif paling fleksibel untuk operasi agregasi, menjadi dasar implementasi berbagai operasi level tinggi lainnya. Fungsi ini memungkinkan spesifikasi tiga tahap proses: inisialisasi combiner, penggabungan nilai dalam partisi, dan penggabungan hasil antar partisi. Fleksibilitas ini memungkinkan implementasi algoritma seperti perhitungan rata-rata bergerak, histogram, atau akumulasi statistik kompleks lainnya.

**Scala:**
```scala
val combineRDD = pairRDD.combineByKey(
  (v) => (v, 1),  // Membuat combiner
  (acc: (Int, Int), v) => (acc._1 + v, acc._2 + 1),  // Merge value dalam partisi
  (acc1: (Int, Int), acc2: (Int, Int)) => (acc1._1 + acc2._1, acc1._2 + acc2._2)  // Merge combiner antar partisi
)

// Hitung rata-rata
val avgRDD = combineRDD.map{ case (key, value) => (key, value._1 / value._2.toFloat) }
avgRDD.foreach(println)
```

**Python:**
```python
def create_combiner(v):
    return (v, 1)

def merge_value(acc, v):
    return (acc[0] + v, acc[1] + 1)

def merge_combiners(acc1, acc2):
    return (acc1[0] + acc2[0], acc1[1] + acc2[1])

combine_rdd = pair_rdd.combineByKey(create_combiner, merge_value, merge_combiners)
avg_rdd = combine_rdd.map(lambda x: (x[0], x[1][0] / float(x[1][1])))
avg_rdd.foreach(print)
```

Best Practices
1. Gunakan `reduceByKey` atau `aggregateByKey` daripada `groupByKey` untuk operasi agregasi yang lebih efisien
2. Pertimbangkan jumlah partisi saat bekerja dengan data besar
3. Gunakan `persist()` untuk RDD yang akan digunakan berulang kali
4. Hindari operasi shuffle yang tidak perlu

### 7. Grouping Data pada Pair RDDs

#### **Pengertian GroupByKey**  
Salah satu operasi umum dalam pemrosesan data berbasis kunci adalah mengelompokkan semua nilai yang terkait dengan kunci yang sama. Fungsi `groupByKey()` pada Spark memungkinkan pengelompokan nilai-nilai tersebut, menghasilkan RDD baru di mana setiap kunci dihubungkan dengan sebuah iterator yang berisi semua nilai terkait.  

Berbeda dengan `reduceByKey()` yang melakukan agregasi (seperti penjumlahan atau penggabungan) selama proses pengelompokan, `groupByKey()` hanya melakukan pengelompokan tanpa operasi tambahan. Hasilnya berupa struktur data yang dapat diubah ke dalam bentuk koleksi seperti List atau Set untuk pemrosesan lebih lanjut.  

#### **Pertimbangan Performa: groupByKey vs reduceByKey**  
Meskipun `groupByKey()` dan `reduceByKey()` dapat memberikan hasil yang sama dalam beberapa kasus, `reduceByKey()` umumnya lebih efisien untuk dataset besar karena:  

1. **Optimasi Sebelum Shuffle**  
   - `reduceByKey()` melakukan **partial aggregation** di setiap partisi sebelum data di-shuffle ke node lain.  
   - Ini mengurangi volume data yang harus ditransfer melalui jaringan, sehingga meningkatkan performa.  
   - Contoh: Jika sebuah partisi memiliki beberapa entri dengan kunci yang sama, `reduceByKey()` akan menjumlahkannya terlebih dahulu sebelum mengirim data ke partisi lain.  

2. **Risiko pada `groupByKey()`**  
   - `groupByKey()` **tidak melakukan optimasi** sebelum shuffle, sehingga semua pasangan key-value dikirim dalam bentuk aslinya.  
   - Pada dataset besar, hal ini dapat menyebabkan:  
     - **Pemborosan bandwidth jaringan** karena data yang tidak perlu ditransfer.  
     - **Out-of-Memory (OOM) Error** jika sebuah executor menerima terlalu banyak data untuk satu kunci.  
     - **Spill ke Disk** yang memperlambat proses karena Spark harus menulis data ke penyimpanan sementara saat memori tidak cukup.  

3. **Kasus Penggunaan yang Tepat**  
   - Gunakan `groupByKey()` hanya jika:  
     - Dataset relatif kecil.  
     - Anda benar-benar membutuhkan semua nilai dalam bentuk iterator (misalnya untuk operasi kompleks yang tidak bisa dilakukan dengan `reduceByKey`).  
   - Untuk agregasi sederhana (seperti `sum`, `count`, `average`), selalu **utamakan `reduceByKey()`, `foldByKey()`, atau `aggregateByKey()`**.  

#### **Ilustrasi Perbedaan groupByKey dan reduceByKey**  
- **`reduceByKey()`**  
  ```
  Partisi 1: (A,1), (A,1), (B,1) → Partial sum: (A,2), (B,1)  
  Partisi 2: (A,1), (B,1), (B,1) → Partial sum: (A,1), (B,2)  
  Setelah Shuffle: (A,3), (B,3)  
  ```
  (Data dikurangi sebelum dikirim ke partisi lain.) Ilustrasi reducebykey:

  ![](https://github.com/sains-data/bigdata-spark/blob/main/Materi%20Spark/Picture/reducekey.jpg)


- **`groupByKey()`**  
  ```
  Partisi 1: (A,1), (A,1), (B,1) → Dikirim apa adanya  
  Partisi 2: (A,1), (B,1), (B,1) → Dikirim apa adanya  
  Setelah Shuffle: (A, [1,1,1]), (B, [1,1,1])  
  ```
  (Semua nilai dikirim tanpa optimasi, meningkatkan beban jaringan.) Ilustrasi groupbykey: 

  ![](https://github.com/sains-data/bigdata-spark/blob/main/Materi%20Spark/Picture/groupby.jpg)

- **Gunakan `groupByKey()` hanya jika diperlukan**, terutama untuk operasi yang memerlukan akses ke semua nilai dalam satu kunci.  
- **Untuk agregasi, selalu pilih `reduceByKey()` atau alternatif lain** (`aggregateByKey`, `combineByKey`, `foldByKey`) karena lebih efisien dalam hal memori dan jaringan.  
- **Hindari `groupByKey` pada dataset besar** untuk mencegah bottleneck jaringan dan risiko OOM.

**1. Scala Version**
**Data Input & Convert to RDD**
```scala
// Data contoh dalam bentuk tuple (key, value)
val countriesTuples = Seq(
  ("España", 1),
  ("Kazakhstan", 1),
  ("Denmark", 1),
  ("España", 1),
  ("España", 1),
  ("Kazakhstan", 1),
  ("Kazakhstan", 1)
)

// Konversi ke Pair RDD
val pairRDD = spark.sparkContext.parallelize(countriesTuples)
```

**Operasi groupByKey()**
```scala
// Kelompokkan nilai berdasarkan kunci
val groupedRDD = pairRDD.groupByKey()

// Tampilkan hasil (kunci + iterator nilai)
groupedRDD.collect().foreach { case (country, counts) =>
  println(s"$country: ${counts.mkString(", ")}")
}
```
**Output:**
```
España: 1, 1, 1
Kazakhstan: 1, 1, 1
Denmark: 1
```

**Operasi reduceByKey() (Sebagai Perbandingan)**
```scala
// Hitung total per kunci
val reducedRDD = pairRDD.reduceByKey(_ + _)

// Tampilkan hasil
reducedRDD.collect().foreach(println)
```
**Output:**
```
(España,3)
(Kazakhstan,3)
(Denmark,1)
```

**2. Python Version (PySpark)**

**Data Input & Convert to RDD**

```python
from pyspark import SparkContext

sc = SparkContext("local", "GroupByKeyExample")

# Data contoh
countries_tuples = [
  ("España", 1),
  ("Kazakhstan", 1),
  ("Denmark", 1),
  ("España", 1),
  ("España", 1),
  ("Kazakhstan", 1),
  ("Kazakhstan", 1)
]

# Konversi ke Pair RDD
pair_rdd = sc.parallelize(countries_tuples)
```

**Operasi groupByKey()**

```python
# Kelompokkan nilai berdasarkan kunci
grouped_rdd = pair_rdd.groupByKey()

# Tampilkan hasil (kunci + list nilai)
for country, counts in grouped_rdd.collect():
    print(f"{country}: {list(counts)}")
```
**Output:**
```
España: [1, 1, 1]
Kazakhstan: [1, 1, 1]
Denmark: [1]
```

**Operasi reduceByKey() (Sebagai Perbandingan)**

```python
# Hitung total per kunci
reduced_rdd = pair_rdd.reduceByKey(lambda a, b: a + b)

# Tampilkan hasil
for result in reduced_rdd.collect():
    print(result)
```
**Output:**
```
('España', 3)
('Kazakhstan', 3)
('Denmark', 1)
```

**Perbedaan Utama**
| Operasi         | Output                          | Optimasi Jaringan | Rekomendasi Penggunaan           |
|-----------------|---------------------------------|-------------------|----------------------------------|
| `groupByKey()`  | `(Kunci, Iterator[Values])`     | ❌ Tanpa optimasi | Jika butuh **semua nilai** dalam bentuk iterable |
| `reduceByKey()` | `(Kunci, AggregatedValue)`      | ✅ Partial aggregation sebelum shuffle | Untuk **agregasi** (sum, count, dll.) |

> **Catatan:**  
> - `groupByKey()` cocok untuk operasi lanjutan seperti `flatMapValues()`.  
> - `reduceByKey()` lebih efisien untuk perhitungan sederhana (e.g., word count).

### 8. **Joins pada Pair RDDs di Spark**  

Joining (penggabungan) adalah operasi penting saat bekerja dengan data berbasis key-value di Spark. Dengan menggabungkan RDD yang berbeda, kita bisa mendapatkan insight yang lebih dalam dari data. Berikut penjelasan dan contoh implementasi berbagai jenis join di **Scala** dan **Python (PySpark)**.

#### **1. `join()`: Menggabungkan Kunci yang Ada di Kedua RDD**  
**Fungsi:**  
- Mengembalikan pasangan `(key, (value1, value2))` **hanya jika key ada di kedua RDD**.  
- Mirip dengan **INNER JOIN** di SQL.  

**Contoh Kode**  

**Scala**  
```scala
val rdd1 = sc.parallelize(Seq(("A", 1), ("B", 2), ("C", 3)))
val rdd2 = sc.parallelize(Seq(("A", "X"), ("B", "Y"), ("D", "Z")))

val joinedRDD = rdd1.join(rdd2)
joinedRDD.collect().foreach(println)
```
**Output:**  
```
(A,(1,X))
(B,(2,Y))
```

**Python (PySpark)**  
```python
rdd1 = sc.parallelize([("A", 1), ("B", 2), ("C", 3)])
rdd2 = sc.parallelize([("A", "X"), ("B", "Y"), ("D", "Z")])

joined_rdd = rdd1.join(rdd2)
print(joined_rdd.collect())
```
**Output:**  
```
[('A', (1, 'X')), ('B', (2, 'Y'))]
```


#### **2. `leftOuterJoin()`: Menggabungkan Semua Kunci dari RDD Kiri**  
**Fungsi:**  
- Mengembalikan semua kunci dari **RDD kiri** (source), dan nilai dari RDD kanan jika ada.  
- Jika tidak ada pasangan di RDD kanan, nilai diganti dengan `None` (Scala: `None`, Python: `None`).  
- Mirip dengan **LEFT JOIN** di SQL.  

**Contoh Kode**  

**Scala**  
```scala
val leftJoinedRDD = rdd1.leftOuterJoin(rdd2)
leftJoinedRDD.collect().foreach(println)
```
**Output:**  
```
(A,(1,Some(X)))
(B,(2,Some(Y)))
(C,(3,None))  // "C" hanya ada di rdd1
```

**Python (PySpark)**  
```python
left_joined_rdd = rdd1.leftOuterJoin(rdd2)
print(left_joined_rdd.collect())
```
**Output:**  
```
[('A', (1, 'X')), ('B', (2, 'Y')), ('C', (3, None))]
```

#### **3. `rightOuterJoin()`: Menggabungkan Semua Kunci dari RDD Kanan**  
**Fungsi:**  
- Mengembalikan semua kunci dari **RDD kanan**, dan nilai dari RDD kiri jika ada.  
- Jika tidak ada pasangan di RDD kiri, nilai diganti dengan `None`.  
- Mirip dengan **RIGHT JOIN** di SQL.  

**Contoh Kode**  

**Scala**  
```scala
val rightJoinedRDD = rdd1.rightOuterJoin(rdd2)
rightJoinedRDD.collect().foreach(println)
```
**Output:**  
```
(A,(Some(1),X))
(B,(Some(2),Y))
(D,(None,Z))  // "D" hanya ada di rdd2
```

**Python (PySpark)**  
```python
right_joined_rdd = rdd1.rightOuterJoin(rdd2)
print(right_joined_rdd.collect())
```
**Output:**  
```
[('A', (1, 'X')), ('B', (2, 'Y')), ('D', (None, 'Z'))]
```

#### **Perbandingan Jenis Join di Pair RDD**  

| **Jenis Join**       | **Fungsi**                          | **Contoh SQL Equivalent** |  
|----------------------|------------------------------------|--------------------------|  
| `join()`             | Hanya kunci yang ada di kedua RDD  | `INNER JOIN`             |  
| `leftOuterJoin()`    | Semua kunci dari RDD kiri + nilai kanan (jika ada) | `LEFT JOIN` |  
| `rightOuterJoin()`   | Semua kunci dari RDD kanan + nilai kiri (jika ada) | `RIGHT JOIN` |  
| `fullOuterJoin()`    | Semua kunci dari kedua RDD (jika ada) | `FULL OUTER JOIN` |  

**Kapan Menggunakan Join di RDD?**

✅ **Menggabungkan dataset dengan kunci yang sama**  
✅ **Analisis data relasional (seperti tabel database)**  
✅ **Data enrichment (misal: menggabungkan data user dengan data transaksi)**  

**Catatan Performa**  
- **Join adalah operasi berat** karena memerlukan **shuffle** (pertukaran data antar worker).  
- **Optimasi:**  
  - Gunakan `partitionBy()` jika RDD sering di-join.  
  - Hindari `groupByKey()` sebelum join (lebih baik `reduceByKey()`).  

### 9. **Sorting dan Actions pada Pair RDD di Spark**

#### 1. **Mengurutkan RDD Berdasarkan Key (sortBy)**

**Fungsi `sortBy()`**
- Digunakan untuk mengurutkan RDD berdasarkan key tertentu.
- **Parameter:**
  - `keyfunc`: Fungsi untuk mengekstrak key dari elemen RDD.
  - `ascending`: Urutan naik (`true`) atau turun (`false`).
  - `numPartitions`: Jumlah partisi untuk distribusi data setelah sorting.

**Contoh Kode**

**Scala**
```scala
val rdd1 = sc.parallelize(Array(("PySpark",10), ("Scala",15), ("R",100)))

// Urutkan berdasarkan key (ascending)
rdd1.sortBy(x => x._1).collect().foreach(println)
/*
Output:
(PySpark,10)
(R,100)
(Scala,15)
*/

// Urutkan berdasarkan value (descending)
rdd1.sortBy(x => x._2, ascending = false).collect().foreach(println)
/*
Output:
(R,100)
(Scala,15)
(PySpark,10)
*/
```

**Python (PySpark)**
```python
rdd1 = sc.parallelize([("PySpark",10), ("Scala",15), ("R",100)])

# Urutkan berdasarkan key (ascending)
sorted_rdd = rdd1.sortBy(lambda x: x[0])
print(sorted_rdd.collect())
# Output: [('PySpark', 10), ('R', 100), ('Scala', 15)]

# Urutkan berdasarkan value (descending)
sorted_rdd = rdd1.sortBy(lambda x: x[1], ascending=False)
print(sorted_rdd.collect())
# Output: [('R', 100), ('Scala', 15), ('PySpark', 10)]
```

## e. **Actions pada Pair RDD**

### **1. Menghitung Jumlah Elemen per Key (`countByKey`)**
- Menghitung berapa kali setiap key muncul di RDD.
- Mengembalikan `Map`/`dict` dengan format `(key -> count)`.

**Contoh Kode**

**Scala**
```scala
val rdd2 = sc.parallelize(Array(("Scala",11), ("Scala",20), ("PySpark",75), ("PySpark",35)))

val countByKey = rdd2.countByKey()
println(countByKey)
// Output: Map(PySpark -> 2, Scala -> 2)
```

**Python (PySpark)**
```python
rdd2 = sc.parallelize([("Scala",11), ("Scala",20), ("PySpark",75), ("PySpark",35)])

count_by_key = rdd2.countByKey()
print(count_by_key)
# Output: defaultdict(<class 'int'>, {'Scala': 2, 'PySpark': 2})
```

### **2. Menghitung Jumlah Kemunculan Setiap Value (`countByValue`)**
- Menghitung berapa kali setiap **pasangan (key, value)** muncul di RDD.
- Berguna untuk analisis frekuensi data.

**Contoh Kode**

**Scala**
```scala
val countByValue = rdd2.countByValue()
println(countByValue)
/*
Output:
Map((PySpark,35) -> 1, 
    (Scala,11) -> 1, 
    (Scala,20) -> 1, 
    (PySpark,75) -> 1)
*/
```

**Python (PySpark)**
```python
count_by_value = rdd2.countByValue()
print(sorted(count_by_value.items()))
# Output: [(('PySpark', 35), 1), (('PySpark', 75), 1), (('Scala', 11), 1), (('Scala', 20), 1)]
```

### **3. Mengambil Semua Key-Value sebagai Dictionary (`collectAsMap`)**
- Mengumpulkan semua elemen RDD ke driver dan mengubahnya menjadi `Map`/`dict`.
- **Hati-hati!** Hanya gunakan untuk RDD kecil karena semua data dikirim ke driver.

**Contoh Kode**

**Scala**
```scala
val rdd1 = sc.parallelize(Array(("PySpark",10), ("Scala",15), ("R",100)))

val resultMap = rdd1.collectAsMap()
println(resultMap)
// Output: Map(PySpark -> 10, Scala -> 15, R -> 100)
```

**Python (PySpark)**
```python
result_map = rdd1.collectAsMap()
print(result_map)
# Output: {'PySpark': 10, 'Scala': 15, 'R': 100}
```

### **4. Mencari Semua Value untuk Key Tertentu (`lookup`)**
- Mengembalikan semua value yang terkait dengan key tertentu dalam bentuk list.

**Contoh Kode**

**Scala**
```scala
val values = rdd2.lookup("PySpark")
println(values)
// Output: List(75, 35)
```

**Python (PySpark)**
```python
values = rdd2.lookup("PySpark")
print(values)
# Output: [75, 35]
```

### **Ringkasan Actions pada Pair RDD**
| **Action**          | **Fungsi**                                  | **Contoh Output**                     |
|----------------------|--------------------------------------------|---------------------------------------|
| `countByKey()`       | Hitung jumlah elemen per key               | `{"Scala": 2, "PySpark": 2}`          |
| `countByValue()`     | Hitung frekuensi setiap (key, value)       | `{(("Scala",11), 1), ...}`            |
| `collectAsMap()`     | Konversi RDD ke dictionary                 | `{"PySpark":10, "Scala":15}`          |
| `lookup(key)`        | Ambil semua value untuk key tertentu       | `[75, 35]` (jika key="PySpark")       |

**Kapan Menggunakan Actions Ini?**

✅ `countByKey()` → Analisis distribusi key (e.g., "Berapa banyak transaksi per user?")  
✅ `countByValue()` → Deteksi duplikat atau frekuensi data unik.  
✅ `collectAsMap()` → Konversi RDD kecil ke dictionary untuk pemrosesan lokal.  
✅ `lookup(key)` → Pencarian cepat nilai berdasarkan key (seperti `dict.get()`).  

Dengan memahami operasi sorting dan actions di Pair RDD, Anda bisa menganalisis data lebih efisien di Spark! 🚀

## **Kapan Menggunakan RDD di Spark**  

Meskipun **DataFrame dan Dataset** (API tingkat tinggi Spark) umumnya lebih disukai karena optimasinya (Catalyst optimizer, Tungsten execution engine), tetap ada situasi di mana **RDD (Resilient Distributed Dataset)** masih lebih cocok digunakan.  

### **Situasi di mana RDD Masih Relevan**  

#### **1. Kontrol Tingkat Rendah atas Distribusi Data**  
- RDD memungkinkan pengaturan eksplisit untuk:  
  - **Partisi** (misal: `repartition()`, `coalesce()`, custom partitioner).  
  - **Penempatan data** (misal: `persist()` dengan opsi penyimpanan seperti MEMORY_ONLY, DISK_ONLY).  
- Berguna untuk **algoritma terdistribusi** yang membutuhkan tuning performa (misal: pemrosesan graf dengan Pregel).  

#### **2. Transformasi & Aksi yang Lebih Fleksibel**  
- Beberapa operasi **hanya tersedia di RDD**, seperti:  
  - **`mapPartitions()`** (proses data per partisi).  
  - **`aggregate()` / `treeAggregate()`** (logika agregasi kustom).  
  - **`zipWithIndex()`** (memberi indeks pada elemen).  
- Diperlukan untuk **logika bisnis khusus** yang tidak bisa diimplementasikan di DataFrame/Dataset.  

#### **3. Kompatibilitas dengan Kode Lama**  
- Aplikasi Spark versi lama mungkin masih menggunakan RDD.  
- Beberapa **library eksternal** (misal: MLlib versi lama) membutuhkan input berbasis RDD.  

#### **4. Pemrosesan Data Tidak Terstruktur atau Kompleks**  
- Ketika data:  
  - **Tidak memiliki skema tetap** (misal: teks mentah, file log, data biner).  
  - Membutuhkan **operasi non-tabuler** (misal: algoritma graf, pipeline machine learning khusus).  

#### **5. Menghindari Overhead DataFrame**  
- Jika **pemrosesan kolom tidak diperlukan** (misal: transformasi per baris).  
- Ketika **skema tidak penting** (misal: pemrosesan pasangan key-value tanpa nama kolom).  

### **Perbandingan: RDD vs. DataFrame/Dataset**  

| **Kriteria**               | **RDD**                          | **DataFrame/Dataset**              |  
|----------------------------|----------------------------------|-----------------------------------|  
| **Optimasi**               | Tidak ada optimasi otomatis      | Optimasi query (Catalyst) & eksekusi cepat (Tungsten) |  
| **Struktur Data**          | Bisa tidak terstruktur          | Harus terstruktur (skema jelas)   |  
| **Kontrol Partisi**        | Penuh (custom partitioner)      | Terbatas (default partitioning)   |  
| **Operasi Khusus**         | Mendukung operasi low-level     | Hanya operasi SQL-like & UDF      |  
| **Kinerja**                | Lebih lambat (karena Java/Scala objects) | Lebih cepat (binary format Tungsten) |  


### **Kapan Memilih RDD?**  

✅ Butuh kontrol penuh partisi & distribusi data  
✅ Membuat algoritma kustom (misal: machine learning, graf)  
✅ Bekerja dengan data tidak terstruktur (raw text, sensor data)  
✅ Mempertahankan kode Spark lama  

### **Kapan Lebih Baik Pakai DataFrame/Dataset?** 

✔ Analisis data berbasis kolom (seperti SQL)  
✔ Butuh performa tinggi (Catalyst & Tungsten optimizations)
✔ Ingin skema jelas & type safety (Dataset di Scala) 

## **Penutup**  

- Gunakan RDD untuk kebutuhan low-level atau data tidak terstruktur.  
- Pilih DataFrame/Dataset untuk analisis data tradisional & performa optimal.  
- RDD masih relevan dalam skenario khusus, tetapi umumnya DataFrame lebih efisien.  
