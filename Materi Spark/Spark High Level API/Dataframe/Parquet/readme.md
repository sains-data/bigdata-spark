# **Membaca dan Menulis File Apache Parquet dengan Spark DataFrame**

## **Apa Itu Apache Parquet?**  
Apache Parquet adalah **format file kolumnar** yang dioptimalkan untuk:  
✅ **Efisiensi penyimpanan** (kompresi tinggi)  
✅ **Performa baca/tulis cepat** (khusus operasi analitik)  
✅ **Kompatibilitas luas** (digunakan di Hadoop, Spark, Hive, dll)  

**Keunggulan Parquet:**  
1. **Kolumnar Storage**: Data disimpan per kolom (bukan per baris), sehingga query yang hanya membutuhkan beberapa kolom lebih cepat.  
2. **Schema Enforcement**: Mempertahankan struktur data (schema) saat disimpan.  
3. **Predicate Pushdown**: Hanya membaca bagian data yang diperlukan.  

---

## **1. Membaca File Parquet**  
### **Scala**  
```scala
// Baca file Parquet  
val df = spark.read.parquet("/path/to/file.parquet")  

// Baca dengan format eksplisit  
val df2 = spark.read.format("parquet").load("/path/to/file.parquet")  

// Tampilkan data  
df.show()  
```  

### **Python (PySpark)**  
```python
# Baca file Parquet  
df = spark.read.parquet("/path/to/file.parquet")  

# Baca dengan format eksplisit  
df2 = spark.read.format("parquet").load("/path/to/file.parquet")  

# Tampilkan data  
df.show()  
```  

---

## **2. Menulis File Parquet**  
### **Scala**  
```scala
// Simpan DataFrame ke Parquet (default mode: ErrorIfExists)  
df.write.parquet("/path/to/output.parquet")  

// Simpan dengan mode overwrite  
df.write.mode("overwrite").parquet("/path/to/output.parquet")  

// Simpan dengan partisi (contoh: partisi berdasarkan kolom "country")  
df.write.partitionBy("country").parquet("/path/to/partitioned_data")  
```  

### **Python (PySpark)**  
```python
# Simpan DataFrame ke Parquet  
df.write.parquet("/path/to/output.parquet")  

# Simpan dengan mode overwrite  
df.write.mode("overwrite").parquet("/path/to/output.parquet")  

# Simpan dengan partisi  
df.write.partitionBy("country").parquet("/path/to/partitioned_data")  
```  

---

## **3. Contoh Lengkap (Baca → Transform → Tulis)**  
### **Scala**  
```scala
// 1. Baca data CSV  
val df = spark.read.option("header", "true").csv("/path/to/data.csv")  

// 2. Filter data  
val filteredDF = df.filter("age > 30")  

// 3. Simpan sebagai Parquet  
filteredDF.write.parquet("/path/to/filtered_data.parquet")  
```  

### **Python (PySpark)**  
```python
# 1. Baca data CSV  
df = spark.read.option("header", "true").csv("/path/to/data.csv")  

# 2. Filter data  
filteredDF = df.filter("age > 30")  

# 3. Simpan sebagai Parquet  
filteredDF.write.parquet("/path/to/filtered_data.parquet")  
```  

---

## **4. Fitur Lanjutan Parquet di Spark**  
| **Fitur**                  | **Contoh Penggunaan**                                   |
|----------------------------|-------------------------------------------------------|
| **Kompresi**               | `df.write.option("compression", "snappy").parquet(...)` |
| **Schema Evolution**       | `spark.sql("SET spark.sql.parquet.mergeSchema=true")`  |
| **Predicate Pushdown**     | `spark.sql("SELECT * FROM parquet.`/path/` WHERE age > 30")` |

---

## **Perbandingan Parquet vs CSV**  
| **Parameter**      | **Parquet**                          | **CSV**                     |
|--------------------|--------------------------------------|-----------------------------|
| **Format**         | Binary (kolumnar)                   | Text (baris)                |
| **Kompresi**       | Sangat efisien (Snappy, Gzip)       | Minimal                     |
| **Kecepatan Baca** | Cepat (hanya baca kolom diperlukan) | Lambat (baca seluruh file)  |
| **Schema**         | Disimpan dalam file                 | Harus diinferensi           |

## **Query Langsung, Partitioning, dan Membaca Partisi di Parquet**

---

### **1. Direct Queries on Parquet Files (Query Langsung Tanpa Load ke DataFrame)**
Spark memungkinkan query langsung ke file Parquet **tanpa memuatnya ke memori** terlebih dahulu, mirip dengan query SQL di database.

#### **Scala**
```scala
// Query langsung ke file Parquet
val hasil = spark.sql("SELECT * FROM parquet.`/path/to/file.parquet` WHERE usia > 25")
hasil.show()

// Query dengan agregasi
spark.sql("""
  SELECT departemen, AVG(gaji) as rata_gaji 
  FROM parquet.`/data/karyawan.parquet` 
  GROUP BY departemen
""").show()
```

#### **Python (PySpark)**
```python
# Query langsung
hasil = spark.sql("SELECT nama, usia FROM parquet.`/path/to/file.parquet` WHERE kota = 'Jakarta'")
hasil.show()

# Join dengan file Parquet lain
spark.sql("""
  SELECT a.nama, b.nama_proyek 
  FROM parquet.`/data/karyawan.parquet` a
  JOIN parquet.`/data/proyek.parquet` b ON a.id = b.id_karyawan
""").show()
```

**Keuntungan:**  
- Tidak perlu `spark.read.parquet()` terlebih dahulu.  
- Cocok untuk eksplorasi data cepat.  

---

### **2. Parquet File Partitioning (Partisi untuk Percepatan Query)**
Partisi mengelompokkan data berdasarkan nilai kolom tertentu (misal: `tahun`, `negara`).  
Struktur direktori:  
```
/path/to/data/
  ├── tahun=2023/
  │   ├── file1.parquet
  │   └── file2.parquet
  └── tahun=2024/
      ├── file3.parquet
```

#### **Membuat Partisi (Scala & Python)**
```scala
df.write.partitionBy("tahun", "bulan").parquet("/path/to/partitioned_data")
```
```python
df.write.partitionBy("tahun", "bulan").parquet("/path/to/partitioned_data")
```

#### **Keuntungan Partitioning:**  
✅ **Query lebih cepat**: Hanya membaca partisi yang dibutuhkan (misal: `WHERE tahun = 2023`).  
✅ **Manajemen data efisien**: Bisa menghapus partisi tertentu tanpa memproses seluruh data.  

---

### **3. Reading Parquet File Partitions (Membaca Partisi Tertentu)**
#### **Baca Semua Partisi (Default)**
```scala
val df = spark.read.parquet("/path/to/partitioned_data")
```

#### **Baca Partisi Spesifik (Pushdown Filtering)**
```scala
// Hanya baca data tahun 2023 (Spark otomatis deteksi partisi)
val df2023 = spark.read.parquet("/path/to/partitioned_data/tahun=2023")

// Atau gunakan filter SQL
val df2023 = spark.sql("""
  SELECT * FROM parquet.`/path/to/partitioned_data` 
  WHERE tahun = 2023 AND bulan = 'Januari'
""")
```

#### **Python (PySpark)**
```python
# Baca partisi tertentu
df_2023 = spark.read.parquet("/path/to/partitioned_data/tahun=2023")

# Filter eksplisit
df_jan = spark.sql("""
  SELECT * FROM parquet.`/path/to/partitioned_data` 
  WHERE bulan = 'Januari'
""")
```

**Catatan:**  
- Partisi akan otomatis menjadi kolom di DataFrame.  
- Gunakan **`partition discovery`** untuk membaca struktur partisi yang kompleks:  
  ```scala
  spark.sql("SET spark.sql.sources.partitionColumnTypeInference.enabled=true")
  ```

---

## **Contoh Lengkap: Pipeline dengan Partitioning**

### **1. Tulis Data dengan Partisi**
```scala
val data = Seq(
  ("Andi", "Jakarta", 2023, "Januari"),
  ("Budi", "Bandung", 2023, "Februari"),
  ("Citra", "Jakarta", 2024, "Januari")
)

val df = data.toDF("nama", "kota", "tahun", "bulan")
df.write.partitionBy("tahun", "bulan").parquet("/data/transaksi")
```

### **2. Baca Partisi Tertentu**
```scala
// Hanya baca transaksi Januari 2023
val dfJan2023 = spark.read.parquet("/data/transaksi/tahun=2023/bulan=Januari")
dfJan2023.show()
```

**Output:**
```
+-----+-------+
| nama|  kota |
+-----+-------+
| Andi|Jakarta|
+-----+-------+
```

---

### **Best Practices**  
1. **Pilih Kolom Partisi yang Sering Difilter** (contoh: `tahun`, `negara`).  
2. **Hindari Partisi dengan Nilai Unik** (misal: `id_user`) karena akan membuat banyak partisi kecil.  
3. **Ukuran Partisi Ideal**: 100MB–1GB per file.  
4. **Gunakan `Predicate Pushdown`** untuk optimasi query:  
   ```sql
   SELECT * FROM parquet.`/data/transaksi` WHERE tahun = 2023
   ```  
   (Hanya membaca file di direktori `tahun=2023`).  

--- 

### **Ringkasan**  
| **Fitur**               | **Kegunaan**                                                                 |
|-------------------------|-----------------------------------------------------------------------------|
| **Direct Queries**      | Query file Parquet tanpa `spark.read`.                                      |
| **Partitioning**        | Tingkatkan kecepatan query dengan pengelompokan data.                       |
| **Baca Partisi**        | Ambil hanya data yang diperlukan menggunakan path atau filter SQL.          |


---

## **Kesimpulan**  
- **Gunakan Parquet** untuk:  
  - Penyimpanan data besar yang efisien.  
  - Query analitik yang membutuhkan kecepatan tinggi.  
- **Gunakan CSV** hanya untuk:  
  - Data kecil atau pertukaran data sederhana.  
  - Kebutuhan kompatibilitas dengan tools non-Spark.  

**Contoh Use Case Parquet:**  
- Data warehouse (DW) di Hadoop/HDFS.  
- Penyimpanan intermediate data di pipeline ETL Spark.