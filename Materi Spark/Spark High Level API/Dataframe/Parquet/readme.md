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

---

### **Kesimpulan**  
- **Gunakan Parquet** untuk:  
  - Penyimpanan data besar yang efisien.  
  - Query analitik yang membutuhkan kecepatan tinggi.  
- **Gunakan CSV** hanya untuk:  
  - Data kecil atau pertukaran data sederhana.  
  - Kebutuhan kompatibilitas dengan tools non-Spark.  

**Contoh Use Case Parquet:**  
- Data warehouse (DW) di Hadoop/HDFS.  
- Penyimpanan intermediate data di pipeline ETL Spark.