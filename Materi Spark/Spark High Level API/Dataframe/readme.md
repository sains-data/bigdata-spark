# **Spark DataFrames**  

## **1. Pengenalan Spark DataFrames**  
DataFrame diperkenalkan di **Spark 1.3** untuk meningkatkan performa dan skalabilitas. Konsep utamanya adalah **skema**, yang mendeskripsikan struktur data sehingga Spark dapat mengoptimasi operasi *shuffle* (perpindahan data antar-node) secara efisien. Secara visual, DataFrame mirip dengan tabel database relasional atau spreadsheet.
**DataFrame** cocok untuk manipulasi data terstruktur dengan optimasi otomatis. **Kelemahan:** Tidak ada *type safety* saat kompilasi (harus hati-hati dalam operasi kolom). **Solusi:** Gunakan **Dataset** jika butuh *type safety*, atau gunakan sintaks kolom eksplisit (`col()`, `$"nama"`) di DataFrame.  

**Contoh Skema DataFrame:**  
```plaintext
root  
|-- Nombre: string (nullable = true)  
|-- Primer_Apellido: string (nullable = true)  
|-- Segundo_Apellido: string (nullable = true)  
|-- Edad: integer (nullable = true)  
|-- Sexo: string (nullable = true)  
```  
Setiap kolom memiliki:  
- **Nama** (misal: `Nombre`, `Edad`)  
- **Tipe data** (misal: `string`, `integer`)  
- **Nullable flag** (menunjukkan apakah kolom boleh berisi nilai `null`).  

## **2. Perbedaan DataFrame vs RDD vs Dataset**

- **DataFrame** = **Dataset[Row]** (tipe tidak terdefinisi saat kompilasi).  
- **RDD & Dataset** memiliki **compile-time type safety**, sedangkan **DataFrame tidak**.  
- **DataFrame** mengembalikan objek `Row` yang tidak memiliki tipe eksplisit, sehingga perlu konversi manual (misal: `asInstanceOf[T]` di Scala atau `cast()` di PySpark).  

## **3. Contoh Praktik: Filtering pada DataFrame vs Dataset**

```scala
// Membuat DataFrame dan Dataset dari RDD  
val writersDF = SpanishWritersRDD.toDF()  
val writersDS = SpanishWritersRDD.toDS()  

// Filter pada DataFrame (ERROR karena tipe Row tidak punya atribut "Edad")  
val writersDFResult = writersDF.filter(writer => writer.Edad > 53)  
// Output: Error: value Edad is not a member of org.apache.spark.sql.Row  

// Filter pada Dataset (Berhasil karena tipe SpanishWriters diketahui saat kompilasi)  
val writersDSResult = writersDS.filter(writer => writer.Edad > 53)  
// Output: Dataset[SpanishWriters] dengan filter umur > 53  
```  
**Solusi untuk DataFrame:** Gunakan kolom eksplisit (`$"Edad"` atau `col("Edad")`):  
```scala
val writersDFResult = writersDF.filter(col("Edad") > 53)  
```  

## **4. Fitur Utama Spark DataFrames**

Spark DataFrames memiliki banyak fitur penting. Salah satunya adalah kemungkinan untuk membuat dataframe dari sumber eksternal—kondisi yang sangat membantu dalam kehidupan nyata, ketika sebagian besar waktu data akan diberikan dalam bentuk file, basis data, dll. Contoh format file eksternal yang didukung langsung oleh Spark untuk memuat data ke dalam DataFrames dapat dilihat pada Gambar di bawah ini.

![](https://github.com/sains-data/bigdata-spark/blob/main/Materi%20Spark/Picture/dataframe.jpg)

Fitur penting lain dari DataFrames adalah kapasitasnya untuk menangani data dalam jumlah besar, mulai dari megabyte hingga petabyte. Dengan demikian, Spark DataFrames memungkinkan pengelolaan data dalam skala besar.
Fitur lainnya adalah sebagai berikut:
- **Distributed Computing:** DataFrame didistribusikan di seluruh node cluster.  
- **Optimasi Otomatis:** Menggunakan **Catalyst Optimizer** untuk membuat rencana eksekusi query yang efisien.  
- **Dukungan Sumber Eksternal:**  
  - Dapat dibuat dari file (CSV, JSON, Parquet, dll.), database (JDBC), atau sumber lain.  
- **Skalabilitas:** Mampu menangani data dari ukuran kecil (MB) hingga sangat besar (PB).  

## **5. Methods for Creating Spark DataFrames**  

Spark DataFrames dapat dibuat dari berbagai sumber, termasuk:  
- **Data terstruktur** (CSV, JSON, Parquet, Excel, dll.)  
- **Database relasional** (MySQL, PostgreSQL, Oracle)  
- **NoSQL** (Hive, Cassandra, MongoDB)  
- **RDD, list, atau sequence**  
- **File biner**  

Berikut dua metode utama untuk membuat DataFrame secara manual:  

---

### **1. Membuat DataFrame dengan `toDF()`**  

**Versi Scala**  
```scala
// Import library implicits untuk konversi RDD ke DataFrame  
import spark.implicits._  

// Buat data sequence  
val carsData = Seq(  
  ("USA", "Chrysler", "Chrysler 300", 292),  
  ("Germany", "BMW", "BMW 8 Series", 617),  
  ("Spain", "Spania GTA", "GTA Spano", 925)  
)  

// Konversi ke RDD lalu ke DataFrame  
val carsRDD = spark.sparkContext.parallelize(carsData)  
val dfCars = carsRDD.toDF()  

// Tampilkan hasil (default column: _1, _2, _3, _4)  
dfCars.show()  
```  
**Output:**  
```
+-------+----------+------------+---+  
|    _1 |       _2 |         _3 | _4|  
+-------+----------+------------+---+  
|   USA | Chrysler |Chrysler 300|292|  
|Germany |      BMW |BMW 8 Series|617|  
|  Spain|Spania GTA|  GTA Spano |925|  
+-------+----------+------------+---+  
```  

**Versi Python (PySpark)**  
```python
from pyspark.sql import SparkSession  

spark = SparkSession.builder.appName("DataFrameExample").getOrCreate()  

# Buat data list  
cars_data = [  
    ("USA", "Chrysler", "Chrysler 300", 292),  
    ("Germany", "BMW", "BMW 8 Series", 617),  
    ("Spain", "Spania GTA", "GTA Spano", 925)  
]  

# Konversi ke DataFrame  
df_cars = spark.createDataFrame(cars_data)  

df_cars.show()  
```  
**Output:** Sama seperti Scala.  

---

### **2. Membuat DataFrame dengan `createDataFrame()`**  

**Versi Scala**  
```scala
// Buat DataFrame dengan nama kolom eksplisit  
val df2 = spark.createDataFrame(carsData)  
  .toDF("Country", "Manufacturer", "Model", "Power")  

df2.show()  
```  
**Output:**  
```
+-------+------------+------------+-----+  
|Country|Manufacturer|       Model|Power|  
+-------+------------+------------+-----+  
|    USA|    Chrysler|Chrysler 300|  292|  
|Germany|         BMW|BMW 8 Series|  617|  
|  Spain|  Spania GTA|   GTA Spano|  925|  
+-------+------------+------------+-----+  
```  

**Versi Python (PySpark)**  
```python
df2 = spark.createDataFrame(cars_data, ["Country", "Manufacturer", "Model", "Power"])  
df2.show()  
```  
**Output:** Sama seperti Scala.  

---

### **3. Membaca DataFrame dari File Eksternal**  
Spark mendukung berbagai format file, tetapi **Parquet** adalah format default.  

#### **Membaca File CSV (Perlu Format Eksplisit)**  

**Scala**  
```scala
val dfCSV = spark.read  
  .option("header", "true")  
  .option("inferSchema", "true")  
  .csv("/path/to/file.csv")  
```  

**Python**  
```python
df_csv = spark.read \  
    .option("header", "true") \  
    .option("inferSchema", "true") \  
    .csv("/path/to/file.csv")  
```  

#### **Membaca File Parquet (Default Spark)**  

**Scala & Python**  
```scala
val dfParquet = spark.read.load("/path/to/file.parquet")  
```  
```python
df_parquet = spark.read.load("/path/to/file.parquet")  
```  

---

### **4. Query Langsung File Tanpa Load ke DataFrame**  
Spark memungkinkan query langsung ke file tanpa memuatnya ke DataFrame terlebih dahulu.  

**Scala & Python**  
```scala
spark.sql("SELECT * FROM parquet.`/path/to/file.parquet`").show()  
```  
```python
spark.sql("SELECT * FROM parquet.`/path/to/file.parquet`").show()  
```  

---

### **5. Mengabaikan File Korup atau Hilang**  
Spark menyediakan opsi untuk:  
- **Abaikan file korup** (`spark.sql.files.ignoreCorruptFiles=true`)  
- **Abaikan file yang hilang** (`spark.sql.files.ignoreMissingFiles=true`)  

**Contoh (Scala & Python)**  
```scala
// Aktifkan pengabaian file korup/hilang  
spark.sql("set spark.sql.files.ignoreCorruptFiles=true")  
spark.sql("set spark.sql.files.ignoreMissingFiles=true")  

// Baca file (jika ada yang korup/hilang, tidak error)  
val df = spark.read.parquet(  
  "/path/to/file1.parquet",  
  "/path/to/file_corrupt.parquet"  
)  

df.show()  // Hanya menampilkan data yang berhasil dibaca  
```  

### Perbandingan

| Metode | Kelebihan | Kekurangan |  
|--------|-----------|------------|  
| **`toDF()`** | Mudah untuk konversi RDD/list | Nama kolom default (_1, _2, ...) |  
| **`createDataFrame()`** | Bisa tentukan nama kolom | Lebih verbose |  
| **Baca dari file** | Support banyak format | Parquet adalah default |  
| **Query langsung** | Tidak perlu load ke memori | Hanya untuk format tertentu |  
| **Ignore corrupt/missing files** | Mencegah job gagal | Data yang korup tidak diproses |  

**Rekomendasi:**  
- Gunakan **`createDataFrame()`** jika butuh penamaan kolom eksplisit.  
- Format **Parquet** lebih efisien untuk big data.  
- Aktifkan `ignoreCorruptFiles` jika bekerja dengan data tidak terjamin integritasnya.

## **6. Time-Based Paths & Menyimpan DataFrame di Spark**  

### **1. Filter Berdasarkan Waktu Modifikasi File**  
Spark menyediakan opsi untuk membaca file berdasarkan waktu modifikasi:  
- **`modifiedAfter`**: Hanya baca file yang dimodifikasi **setelah** waktu tertentu.  
- **`modifiedBefore`**: Hanya baca file yang dimodifikasi **sebelum** waktu tertentu.  

Format timestamp: **`YYYY-MM-DDTHH:mm:ss`** (contoh: `2022-10-30T05:30:00`).  

**Contoh di Scala**  
```scala
val modifiedAfterDF = spark.read.format("csv")  
  .option("header", "true")  
  .option("modifiedAfter", "2022-10-30T05:30:00")  
  .load("/path/to/files")  

modifiedAfterDF.show()  
```  

**Contoh di Python (PySpark)**  
```python
modified_after_df = spark.read.format("csv") \  
  .option("header", "true") \  
  .option("modifiedAfter", "2022-10-30T05:30:00") \  
  .load("/path/to/files")  

modified_after_df.show()  
```  

---

### **2. Mode Penyimpanan DataFrame**  
Spark mendukung berbagai format penyimpanan (Parquet, CSV, JSON, ORC, dll.) dengan beberapa mode:  

| **Mode (Scala/Java)**       | **Mode (Semua Bahasa)**       | **Keterangan**                                                                 |
|-----------------------------|-------------------------------|-------------------------------------------------------------------------------|
| `SaveMode.ErrorIfExists` (default) | `"error"` / `"errorifexists"` | Error jika data sudah ada di lokasi tujuan.                                   |
| `SaveMode.Append`           | `"append"`                    | Menambahkan data ke data yang sudah ada.                                      |
| `SaveMode.Overwrite`        | `"overwrite"`                 | Menimpa data yang sudah ada.                                                 |
| `SaveMode.Ignore`           | `"ignore"`                    | Tidak melakukan apa-apa jika data sudah ada (seperti `IF NOT EXISTS` di SQL). |

**Contoh Penyimpanan di Scala**  
```scala
// Simpan sebagai Parquet (default)  
df.write.mode("overwrite").save("/path/to/output")  

// Simpan sebagai CSV  
df.write.mode("append").csv("/path/to/output.csv")  
```  

**Contoh Penyimpanan di Python**  
```python
# Simpan sebagai JSON  
df.write.mode("ignore").json("/path/to/output.json")  
```  

---

### **3. Menyimpan dalam Satu File (Coalesce/Repartition)**  
Secara default, Spark menyimpan data dalam **multiple part files** (satu file per partisi). Untuk menggabungkannya menjadi **satu file**:  

#### **Menggunakan `coalesce(1)` atau `repartition(1)`**  

**Scala**  
```scala
df.coalesce(1)  
  .write.csv("/path/to/single_file.csv")  
```  

**Python**  
```python
df.coalesce(1) \  
  .write.csv("/path/to/single_file.csv")  
```  

**Catatan:**  
- Masih akan terbentuk folder dengan file `_SUCCESS` dan `.crc`.  
- **Hati-hati** menggunakan `coalesce(1)` untuk data besar karena bisa menyebabkan **OutOfMemory**.  

---

### **4. Menghapus File _SUCCESS dan .crc**  
Jika perlu menghapus file metadata (_SUCCESS, .crc), gunakan **Hadoop FileSystem API**:  

**Contoh di Scala**  
```scala
import org.apache.hadoop.fs._

val fs = FileSystem.get(spark.sparkContext.hadoopConfiguration)  
val outputPath = new Path("/path/to/output")  

// Hapus _SUCCESS  
fs.delete(new Path(outputPath, "_SUCCESS"), false)  

// Hapus .crc files  
fs.listStatus(outputPath)  
  .filter(_.getPath.getName.startsWith("."))  
  .foreach(file => fs.delete(file.getPath, false))  
```  

**Contoh di Python**  
```python
from py4j.java_gateway import java_import  

# Akses Java FileSystem  
java_import(spark._jvm, "org.apache.hadoop.fs.*")  
fs = spark._jvm.FileSystem.get(spark._jsc.hadoopConfiguration())  

# Hapus _SUCCESS  
success_file = spark._jvm.Path("/path/to/output/_SUCCESS")  
fs.delete(success_file, True)  
``` 

### **Ringkasan**  
| **Fitur**                     | **Kegunaan**                                                                 |
|-------------------------------|-----------------------------------------------------------------------------|
| **`modifiedAfter/modifiedBefore`** | Filter file berdasarkan waktu modifikasi.                                  |
| **Mode Penyimpanan (`overwrite`, `append`, dll.)** | Kontrol bagaimana data disimpan (timpa, tambah, error jika sudah ada).     |
| **`coalesce(1)`**             | Menggabungkan output menjadi satu file (hati-hati untuk data besar).       |
| **Hapus _SUCCESS/.crc**       | Butuh manipulasi dengan Hadoop FileSystem API.                             |

**Rekomendasi:**  
- Gunakan **`overwrite`** atau **`append`** untuk mengontrol penulisan data.  
- Hindari `coalesce(1)` untuk dataset sangat besar.  
- Jika perlu file tunggal tanpa metadata, pertimbangkan **pemrosesan pasca-Spark** (misal: `hadoop fs -getmerge`).

