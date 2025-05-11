# **Membaca dan Menulis File JSON dengan Spark DataFrames**

## **1. Membaca dan Menulis File JSON**
### **Membaca File JSON**
#### Scala
```scala
// Baca file JSON
val df = spark.read.json("/path/ke/file.json")

// Baca dengan format eksplisit
val df = spark.read.format("json").load("/path/ke/file.json")

// Tampilkan skema
df.printSchema()
```

#### Python (PySpark)
```python
# Baca file JSON
df = spark.read.json("/path/ke/file.json")

# Baca dengan format eksplisit
df = spark.read.format("json").load("/path/ke/file.json")

# Tampilkan skema
df.printSchema()
```

### **Menulis DataFrame ke JSON**
#### Scala
```scala
// Tulis DataFrame ke JSON
df.write.json("/path/ke/output.json")

// Tulis dengan mode overwrite
df.write.mode("overwrite").json("/path/ke/output.json")

// Tulis dengan kompresi
df.write.option("compression", "gzip").json("/path/ke/output.json.gz")
```

#### Python (PySpark)
```python
# Tulis DataFrame ke JSON
df.write.json("/path/ke/output.json")

# Tulis dengan mode overwrite
df.write.mode("overwrite").json("/path/ke/output.json")

# Tulis dengan kompresi
df.write.option("compression", "gzip").json("/path/ke/output.json.gz")
```

---

## **2. Membaca Banyak File JSON Sekaligus**
Spark bisa membaca banyak file JSON dalam satu operasi.

#### Scala
```scala
// Baca semua file JSON di direktori
val df = spark.read.json("/path/ke/json_files/*.json")

// Baca file spesifik
val df = spark.read.json("/path/ke/file1.json", "/path/ke/file2.json")
```

#### Python (PySpark)
```python
# Baca semua file JSON di direktori
df = spark.read.json("/path/ke/json_files/*.json")

# Baca file spesifik
df = spark.read.json(["/path/ke/file1.json", "/path/ke/file2.json"])
```

---

## **3. Membaca File JSON Berdasarkan Pola**
Spark mendukung **pola glob** untuk membaca file dengan pola nama tertentu.

#### Scala
```scala
// Baca file dengan pola (misal: data_2023*.json)
val df = spark.read.json("/path/ke/data_2023*.json")

// Baca file dari banyak direktori
val df = spark.read.json("/path/ke/{folder1,folder2}/*.json")
```

#### Python (PySpark)
```python
# Baca file dengan pola
df = spark.read.json("/path/ke/data_2023*.json")

# Baca file dari banyak direktori
df = spark.read.json("/path/ke/{folder1,folder2}/*.json")
```

---

## **4. Query Langsung pada File JSON**
Spark memungkinkan query langsung ke file JSON **tanpa memuatnya ke DataFrames** (mirip dengan Parquet).

#### Scala
```scala
// Query langsung ke JSON
val hasil = spark.sql("SELECT * FROM json.`/path/ke/file.json` WHERE umur > 25")

// Join file JSON
spark.sql("""
  SELECT a.nama, b.id_pesanan 
  FROM json.`/path/ke/pengguna.json` a
  JOIN json.`/path/ke/pesanan.json` b ON a.id = b.id_pengguna
""").show()
```

#### Python (PySpark)
```python
# Query langsung ke JSON
hasil = spark.sql("SELECT * FROM json.`/path/ke/file.json` WHERE umur > 25")

# Join file JSON
spark.sql("""
  SELECT a.nama, b.id_pesanan 
  FROM json.`/path/ke/pengguna.json` a
  JOIN json.`/path/ke/pesanan.json` b ON a.id = b.id_pengguna
""").show()
```

---

## **5. Menyimpan DataFrame ke File JSON**
### **Opsi saat Menulis JSON**
- `mode`: `overwrite`, `append`, `ignore`, `error` (default).  
- `compression`: `gzip`, `snappy`, `lz4`.  
- `dateFormat`: Format tanggal kustom.  

#### Scala
```scala
// Tulis dengan opsi
df.write
  .mode("overwrite")
  .option("compression", "gzip")
  .json("/path/ke/output.json")
```

#### Python (PySpark)
```python
# Tulis dengan opsi
df.write \
  .mode("overwrite") \
  .option("compression", "gzip") \
  .json("/path/ke/output.json")
```

---

## **6. Memuat File JSON dengan Skema Kustom**
Jika file JSON memiliki skema tidak konsisten, Anda bisa memaksa **skema kustom**.

#### Scala
```scala
import org.apache.spark.sql.types._

// Definisikan skema
val skema = StructType(Seq(
  StructField("nama", StringType, nullable = false),
  StructField("umur", IntegerType, nullable = true),
  StructField("kota", StringType, nullable = true)
))

// Baca JSON dengan skema
val df = spark.read.schema(skema).json("/path/ke/file.json")
```

#### Python (PySpark)
```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

# Definisikan skema
skema = StructType([
  StructField("nama", StringType(), False),
  StructField("umur", IntegerType(), True),
  StructField("kota", StringType(), True)
])

# Baca JSON dengan skema
df = spark.read.schema(skema).json("/path/ke/file.json")
```

---

## **7. Bekerja dengan Struktur JSON Bertingkat**
Spark bisa menangani **JSON bertingkat** (objek dan array).

### **Contoh JSON (struktur bertingkat)**
```json
{
  "pengguna": "Alice",
  "umur": 30,
  "alamat": {
    "jalan": "123 Jalan Utama",
    "kota": "Jakarta"
  },
  "pesanan": [
    { "id": 1, "jumlah": 100 },
    { "id": 2, "jumlah": 200 }
  ]
}
```

### **Mengakses Field Bertingkat**
#### Scala
```scala
// Baca JSON bertingkat
val df = spark.read.json("/path/ke/data_tingkat.json")

// Query field bertingkat
df.select(
  col("pengguna"),
  col("alamat.kota").alias("kota"),
  explode(col("pesanan")).alias("pesanan")
).show()
```

#### Python (PySpark)
```python
# Baca JSON bertingkat
df = spark.read.json("/path/ke/data_tingkat.json")

# Query field bertingkat
from pyspark.sql.functions import col, explode

df.select(
  col("pengguna"),
  col("alamat.kota").alias("kota"),
  explode(col("pesanan")).alias("pesanan")
).show()
```

### **Membuat JSON Bertingkat Menjadi Rata**
```scala
// Ratakan JSON (Scala)
val dfRata = df.select(
  col("pengguna"),
  col("umur"),
  col("alamat.jalan").alias("jalan"),
  col("alamat.kota").alias("kota"),
  explode(col("pesanan")).alias("pesanan")
)
```

```python
# Ratakan JSON (Python)
df_rata = df.select(
  col("pengguna"),
  col("umur"),
  col("alamat.jalan").alias("jalan"),
  col("alamat.kota").alias("kota"),
  explode(col("pesanan")).alias("pesanan")
)
```

---

## **Ringkasan**
| **Fitur** | **Deskripsi** |
|------------|----------------|
| **Baca/Tulis JSON** | `spark.read.json()` & `df.write.json()` |
| **Banyak File** | Gunakan `*.json` atau daftar file |
| **Baca Berdasarkan Pola** | `data_2023*.json` |
| **Query Langsung SQL** | `spark.sql("SELECT * FROM json.`path`")` |
| **Skema Kustom** | Gunakan `StructType` untuk skema ketat |
| **JSON Bertingkat** | Gunakan `col("field.tingkat")` atau `explode()` untuk array |

**Tips Praktis:**
✅ Gunakan **`skema`** untuk parsing JSON yang konsisten.  
✅ **Partisi file JSON** jika menulis dataset besar.  
✅ Gunakan **`explode()`** untuk array bertingkat.  
✅ Pilih **`kompresi`** (gzip/snappy) untuk efisiensi penyimpanan.  

Panduan ini mencakup semua operasi utama JSON di Spark DataFrames! 🚀