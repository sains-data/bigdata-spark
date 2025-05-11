# **The Spark High-Level APIs**  

Spark menyediakan beberapa komponen API tingkat tinggi, seperti **Spark SQL, DataFrames, dan Datasets**, yang dirancang untuk memanipulasi data terstruktur. Data terstruktur mengacu pada informasi yang telah diorganisir dalam skema terstandar, sehingga mudah diakses dan dianalisis tanpa memerlukan pemrosesan tambahan. Contoh data terstruktur meliputi tabel database, file Excel, tabel RDBMS, dan file Parquet.  

API tingkat tinggi Spark memungkinkan pengoptimalan aplikasi yang bekerja dengan berbagai jenis data, termasuk file format biner, melebihi batasan yang dimiliki oleh RDD. **DataFrames dan Datasets** memanfaatkan **Catalyst Optimizer** (optimizer query dari Spark SQL) dan **Project Tungsten** (optimasi performa berbasis memori) untuk meningkatkan kinerja pemrosesan.  

Perbedaan utama antara **Dataset API** dan **DataFrame API** adalah **type safety**. Dataset menerapkan **compile-time type safety**, sedangkan DataFrame tidak. Artinya, Spark memeriksa tipe data pada DataFrame saat runtime berdasarkan skema yang ditentukan, sementara Dataset memvalidasi tipe data pada saat kompilasi. Konsep **compile-time type safety** akan dibahas lebih detail di bagian selanjutnya.  

---  
**Catatan:**  
- **Spark SQL, DataFrames, dan Datasets** digunakan untuk data terstruktur.  
- **Data terstruktur** memiliki skema yang jelas (contoh: tabel database, file Parquet).  
- **Optimasi performa** dilakukan melalui Catalyst Optimizer dan Project Tungsten.  
- **Perbedaan utama:** Dataset memiliki **compile-time type safety**, sedangkan DataFrame hanya runtime type checking.