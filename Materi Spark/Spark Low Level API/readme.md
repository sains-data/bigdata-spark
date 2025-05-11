# 🚀 Apache Spark - Low Level API

Apache Spark menyediakan **Low Level API** bagi pengguna yang ingin mengontrol langsung struktur data terdistribusi menggunakan **RDD (Resilient Distributed Dataset)**. API ini cocok untuk operasi yang membutuhkan kontrol granular atas partisi, transformasi, dan aksi dalam sistem terdistribusi.

## 1. RDD (Resilient Distributed Dataset)

* RDD adalah struktur data utama dalam Spark untuk menyimpan data terdistribusi dan immutable.
* Dapat dibuat dari:

  * Koleksi lokal menggunakan `parallelize()`
  * File eksternal (CSV, JSON, HDFS, S3, dll.)
  * Transformasi dari RDD lain

Karakteristik:

* **Immutable**: Tidak dapat diubah setelah dibuat.
* **Lazy Evaluation**: Transformasi dieksekusi saat aksi (`action`) dipanggil.
* **Fault Tolerance**: Dapat dipulihkan dari lineage.

Transformasi Umum:

* `map(func)`
* `filter(func)`
* `flatMap(func)`
* `distinct()`
* `union()`

Aksi (Actions):

* `collect()`
* `count()`
* `take(n)`
* `first()`
* `reduce(func)`

🐍 Python Contoh:

```python
rdd = sc.parallelize(["Spring", "Summer", "Fall", "Winter"], 4)
mapped = rdd.map(lambda x: (x[0], x))
print(mapped.collect())
```

## 2. Key-Value RDDs

* RDD yang berisi elemen pasangan `(key, value)`
* Ideal untuk operasi seperti **grouping**, **aggregation**, dan **joins**

Transformasi Khusus:

* `reduceByKey(func)`
* `groupByKey()`
* `mapValues(func)`
* `flatMapValues(func)`
* `keys()`, `values()`
* `join()`, `leftOuterJoin()`, `rightOuterJoin()`

Contoh Operasi:

```python
pairs = sc.parallelize([("a", 1), ("b", 2), ("a", 3)])
reduced = pairs.reduceByKey(lambda x, y: x + y)
print(reduced.collect())  # Output: [('a', 4), ('b', 2)]
```

⚠️ Tips:

* Gunakan `reduceByKey()` untuk efisiensi dibanding `groupByKey()`
* Gunakan `partitionBy()` untuk kontrol partisi berdasarkan key

## 📚 Referensi Tambahan:

* [RDD Programming Guide - Spark Documentation](https://spark.apache.org/docs/latest/rdd-programming-guide.html)
* Gunakan **RDDs** bila:

  * Anda perlu kontrol detail partisi dan performa
  * Dataset tidak cocok untuk struktur tabular (DataFrames)
  * Anda sedang melakukan pembelajaran atau eksperimen low-level

## Referensi Pustaka

1. García, A. A. (2023). Hands-on Guide to Apache Spark 3: Build Scalable Computing Engines for Batch and Stream Data Processing. Apress.
2. Luu, H., & Luu. (2021). Beginning Apache Spark 3. Apress.
3. Haines, S. (2022). Modern Data Engineering with Apache Spark: A Hands-On Guide for Building Mission-Critical Streaming Applications, English. San Jose, CA, USA: Apress, a Springer Nature company.