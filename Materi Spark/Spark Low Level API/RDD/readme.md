# **Spark Low-Level API**

## Resilient Distributed Datasets (RDDs)

**Apa itu RDD?**

* RDD adalah **inti dari Spark**.
* Bentuknya adalah **kumpulan data (collection) yang tidak bisa diubah (immutable)**.
* Jika kita melakukan operasi terhadap RDD, maka hasilnya adalah **RDD baru**, yang tetap menjaga data asli.

**Kenapa pakai RDD?**

* Bisa diproses secara **paralel** (dibagi ke beberapa node komputer dalam cluster).
* Sudah otomatis mengatur partisi data untuk diproses, kita tidak perlu repot mikir cara bagi datanya.
* **Tahan terhadap kegagalan**: Kalau ada partisi rusak, Spark bisa mengambil kembali data dari salinan lain (misalnya di HDFS, S3, dll).

RDD logical partitions:

![](https://github.com/sains-data/bigdata-spark/blob/main/Materi%20Spark/Picture/RDD.jpg)

## Cara Membuat RDD

Terdapat 3 cara dalam membuat RDD:
1. Membuat RDD dari Koleksi Paralel
2. Membuat RDD dari Kumpulan Data Eksternal
3. Membuat RDD dari RDD yang Ada


### 1. **Membuat RDD dari Koleksi Paralel**

Dari Koleksi di Memori (List, Array, dsb.)

PySpark:

```python
# Contoh membuat RDD dari list
dataList = ['a','b','c','d','e','f','g','h','i','j','k','l']
rdd = spark.sparkContext.parallelize(dataList, 4)  # dibagi jadi 4 partisi

# Cek jumlah partisi
print("Jumlah partisi:", rdd.getNumPartitions())
```

Membuat RDD menggunakan sparkContext.parallelize():

![](https://github.com/sains-data/bigdata-spark/blob/main/Materi%20Spark/Picture/RDD1.jpg)


```python
# Contoh lain: membuat RDD dari list angka
rdd = sc.parallelize([1,2,3,4,5,6,7,8,9,10])

# Operasi RDD
print("Jumlah:", rdd.reduce(lambda a, b: a + b))      # Output: 55
print("Minimum:", rdd.reduce(lambda a, b: min(a, b))) # Output: 1
print("Maksimum:", rdd.reduce(lambda a, b: max(a, b)))# Output: 10
```

Scala:

```scala
// Contoh membuat RDD dari array
val myCollection = Array(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
val rdd = spark.sparkContext.parallelize(myCollection)

// Operasi RDD
rdd.reduce(_ + _)   // Output: 55
rdd.reduce(_ min _) // Output: 1
rdd.reduce(_ max _) // Output: 10
```

> ⚙️ *Note:* Kita bisa tentukan jumlah partisi saat membuat RDD, misalnya:

```scala
val rdd = spark.sparkContext.parallelize(myCollection, 4)  // Jadi 4 partisi
```

* **RDD adalah cara dasar untuk bekerja dengan data di Spark.**
* Cocok digunakan jika butuh kontrol lebih rendah (low-level) terhadap data.
* Cocok untuk data dari:
  * Koleksi lokal (List, Array)
  * File eksternal (CSV, JSON, HDFS, S3, dll.) → akan dibahas di bagian berikutnya.

## 2. **Membuat RDD dari Kumpulan Data Eksternal**

RDD dapat dibuat dari berbagai sumber eksternal yang kompatibel dengan Hadoop, seperti:

* Local file system
* HDFS
* Cassandra
* HBase
* AWS S3
* dll.

**Spark mendukung berbagai format file:**

* Text files (`.txt`)
* Sequence files
* CSV files
* JSON files
* Format lain yang didukung Hadoop


### **1. Membaca File Teks (textFile)**

**Scala:**

```scala
val readmeFile = sc.textFile("/YOUR/SPARK/HOME/README.md")
```

**Python (PySpark):**

```python
readme_file = sc.textFile("/YOUR/SPARK/HOME/README.md")
```

Untuk file di sistem file terdistribusi (HDFS, S3, GCS):

**Scala:**

```scala
val myFile = sc.textFile("s3a://bucket-name/file.txt")
```

**Python:**

```python
my_file = sc.textFile("s3a://bucket-name/file.txt")
```

### **2. Dukungan Folder, Wildcard, dan File Kompresi**

Contoh:

**Scala:**

```scala
val data = sc.textFile("/data/*.csv")
```

**Python:**

```python
data = sc.textFile("/data/*.csv")
```

### **3. Kontrol Jumlah Partisi**

**Scala:**

```scala
val myFile = sc.textFile("/file.txt", 10)
```

**Python:**

```python
my_file = sc.textFile("/file.txt", 10)
```

📌 *Catatan: Jumlah partisi tidak boleh lebih kecil dari jumlah blok file.*


### **4. Membaca Banyak File Kecil (wholeTextFiles)**

Menghasilkan pasangan `(filename, content)`. Cocok untuk file kecil.

**Scala:**

```scala
val files = sc.wholeTextFiles("/path/to/folder")
```

**Python:**

```python
files = sc.wholeTextFiles("/path/to/folder")
```

### **5. Sequence Files**

Format Hadoop yang menyimpan pasangan `(key, value)` dan mendukung kompresi.

**Scala (menulis):**

```scala
rdd.saveAsSequenceFile("/output/path")
```

**Python (menulis):**

```python
rdd.saveAsSequenceFile("/output/path")
```

**Scala (membaca):**

```scala
val rdd = sc.sequenceFile[String, Int]("/input/path")
```

**Python (membaca):**

```python
rdd = sc.sequenceFile("/input/path", "org.apache.hadoop.io.Text", "org.apache.hadoop.io.IntWritable")
```

### **6. Menyimpan dan Memuat RDD sebagai Objek**

**Menyimpan sebagai Object File**

**Scala:**

```scala
val list = sc.parallelize(List("España", "México", "Colombia"))
list.saveAsObjectFile("/tmp/SpanishCountries")
```

**Python:**

```python
data = sc.parallelize(["España", "México", "Colombia"])
data.saveAsPickleFile("/tmp/SpanishCountries")
```

**Membaca kembali**

**Scala:**

```scala
val newlist = sc.objectFile[String]("/tmp/SpanishCountries")
```

**Python:**

```python
newlist = sc.pickleFile("/tmp/SpanishCountries")
```

## 3. **Membuat RDD dari RDD yang Ada**

### **Immutabilitas RDD**

* RDD (Resilient Distributed Dataset) bersifat **immutable**, yang berarti kita tidak dapat mengubah data yang ada dalam RDD secara langsung.
* Namun, kita bisa **membuat RDD baru** dengan menerapkan **transformasi** pada RDD yang sudah ada.

Beberapa transformasi yang umum digunakan:

* `map()`
* `filter()`
* `count()`
* `distinct()`
* `flatMap()`


### **Langkah-langkah untuk Membuat RDD Baru dari RDD yang Ada**

**Contoh Praktis**: Membuat RDD dari Koleksi dan Transformasi

**1. Membuat Koleksi Musim**

```scala
val seasonsCollection = Seq("Summer", "Autumn", "Spring", "Winter")
```

**2. Mengubah Koleksi Menjadi RDD dengan `parallelize()`**

```scala
val seasonsParallel = spark.sparkContext.parallelize(seasonsCollection, 4)
```

**3. Menerapkan Transformasi `map()` untuk Membuat RDD Baru**

```scala
val findSeasons = seasonsParallel.map(s => (s.charAt(0), s))
```

**4. Melihat Isi RDD dengan `collect()`**

```scala
findSeasons.collect().foreach(c => println(c))
```

**Output**:

```
(S,Spring)
(W,Winter)
(S,Summer)
(A,Autumn)
```

**5. Mengecek Jumlah Partisi RDD**

```scala
println("Partitions: " + findSeasons.getNumPartitions)
```

**Output**:

```
Partitions: 4
```

⚠️ **Perhatian pada Penggunaan `collect()`**

* **`collect()`** mengumpulkan semua data dari RDD yang terdistribusi dan membawa mereka ke node driver.
* Ini **berisiko** karena dapat menyebabkan **kekurangan memori** jika ukuran dataset tidak muat di memori driver.
* **Inefisiensi:** Semua data harus melewati jaringan ke driver, yang lebih lambat dibandingkan menulis ke disk atau melakukan komputasi dalam memori.

🌟 **Alternatif yang Lebih Aman**: Gunakan `take()` untuk Mengambil Sampel

Misalnya, mengambil 2 elemen pertama dari RDD:

```scala
findSeasons.take(2).foreach(c => println(c))
```

**Output**:

```
(S,Summer)
(A,Autumn)
```

### 🐍 **Versi Python (PySpark)**

**Membuat Koleksi dan RDD**

```python
seasons_collection = ["Summer", "Autumn", "Spring", "Winter"]
seasons_parallel = sc.parallelize(seasons_collection, 4)
```

**Menerapkan Transformasi `map()`**

```python
find_seasons = seasons_parallel.map(lambda s: (s[0], s))
```

**Melihat Isi RDD dengan `collect()`**

```python
for item in find_seasons.collect():
    print(item)
```

**Output**:

```
('S', 'Spring')
('W', 'Winter')
('S', 'Summer')
('A', 'Autumn')
```

**Mengecek Jumlah Partisi RDD**

```python
print("Partitions:", find_seasons.getNumPartitions())
```

**Output**:

```
Partitions: 4
```