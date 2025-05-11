# **Membaca dan Menulis File CSV, Hive Tables, dan Database JDBC dengan Spark DataFrames**

## **Read and Write CSV Files with Spark**

### Membaca File CSV

```python
df = spark.read.option("header", True).option("inferSchema", True).csv("path/to/file.csv")
```

### Menulis ke File CSV

```python
df.write.option("header", True).mode("overwrite").csv("output/path")
```

### ⚠️ Catatan Penting:

* `header=True`: baris pertama dianggap sebagai nama kolom.
* `inferSchema=True`: Spark akan menebak tipe data, namun bisa memperlambat proses.
* File CSV ditulis sebagai **multiple part files** jika tidak dikumpulkan (`coalesce(1)`).
* Format CSV tidak menyimpan tipe data; perlu inferSchema atau definisi skema eksplisit.
* Gunakan `delimiter` untuk file dengan separator lain, seperti `;`.

---

## **Read and Write Hive Tables**

### Membaca Tabel Hive

```python
df = spark.sql("SELECT * FROM database.table_name")
```

### Menulis ke Tabel Hive

```python
df.write.mode("overwrite").saveAsTable("database.new_table_name")
```

### ⚠️ Catatan Penting:

* Spark harus dikonfigurasi dengan Hive support (`enableHiveSupport()` saat membuat SparkSession).
* Tabel Hive bisa **managed** (disimpan di warehouse default) atau **external**.
* `saveAsTable()` membuat tabel baru, gunakan `.insertInto()` untuk menambahkan data ke tabel yang sudah ada.
* Pastikan database dan tabel sudah tersedia jika tidak membuatnya langsung dari Spark.

---

## **Read and Write Data via JDBC from and to Databases**

### Membaca dari Database via JDBC

```python
df = spark.read.format("jdbc").options(
    url="jdbc:mysql://host:port/database",
    driver="com.mysql.cj.jdbc.Driver",
    dbtable="table_name",
    user="username",
    password="password"
).load()
```

### Menulis ke Database via JDBC

```python
df.write.format("jdbc").options(
    url="jdbc:mysql://host:port/database",
    driver="com.mysql.cj.jdbc.Driver",
    dbtable="target_table",
    user="username",
    password="password"
).mode("overwrite").save()
```

### ⚠️ Catatan Penting:

* Format harus `jdbc` dan driver harus sesuai (contoh: `com.mysql.cj.jdbc.Driver`).
* Mendukung berbagai mode penulisan: `overwrite`, `append`, `ignore`, `error`.
* Penulisan ke database besar bisa dioptimalkan dengan `batchsize` dan `numPartitions`.
* Pastikan koneksi JDBC tersedia dan kredensial aman.