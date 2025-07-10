# Digital-Image-Processing [Kompresi-Lossless-RLE-dan-Huffman-Untuk-Klasifikasi-Citra]

Penelitian ini membahas perbandingan dua teknik kompresi citra digital lossless, yaitu Run Length Encoding (RLE) dan Huffman Coding, dalam upaya mengoptimalkan efisiensi penyimpanan dan mendukung performa klasifikasi citra menggunakan algoritma Random Forest. Dataset yang digunakan adalah Butterfly Mimics Dataset yang berisi 1.028 citra kupu-kupu, yang masing-masing mengalami pre-processing berupa kuantisasi warna dan resize ke ukuran 128×128 piksel. Proses kompresi dilakukan menggunakan kedua metode, dan perbandingan dilakukan berdasarkan rasio kompresi yang dihasilkan. Hasil menunjukkan bahwa Huffman Coding memiliki performa kompresi lebih baik dibandingkan RLE, terutama karena kemampuannya dalam memanfaatkan frekuensi simbol secara efisien. Oleh karena itu, hanya Huffman Coding yang digunakan pada tahap klasifikasi selanjutnya. Setelah citra didekompresi dan diekstraksi cirinya menggunakan Histogram of Oriented Gradients (HOG), fitur tersebut digunakan sebagai input untuk model Random Forest. Model ini berhasil mencapai akurasi klasifikasi sebesar 90,94% dan nilai ROC AUC multiclass sebesar 0,9843, yang menunjukkan kemampuan diskriminatif yang tinggi. Evaluasi juga dilakukan dengan validasi silang dan confusion matrix, yang mengindikasikan kestabilan performa serta kesalahan klasifikasi yang relatif kecil, terutama pada kelas dengan kemiripan visual. Berdasarkan hasil ini, dapat disimpulkan bahwa Huffman Coding merupakan metode kompresi lossless yang lebih unggul dalam konteks klasifikasi citra digital karena mampu mengurangi ukuran file tanpa mengorbankan akurasi klasifikasi.

## Method
### Kompresi
1. Untuk RLE, setiap saluran warna di-flatten menjadi satu vektor 1D, kemudian di-encode ke dalam rangkaian pasangan (nilai piksel, panjang run) berdasarkan urutan kemunculan piksel yang berulang (Salomon, 2017). Metode ini sangat efektif ketika citra memiliki banyak area warna seragam karena menyederhanakan representasi data.
2. Huffman Coding membangun pohon biner berdasarkan frekuensi kemunculan simbol 8‑bit pada setiap saluran, menghasilkan kode variabel dengan panjang minimal untuk simbol yang sering muncul (Cover & Thomas, 2006). Implementasi menggunakan min‑heap untuk penggabungan node frekuensi terendah dan traversal pohon untuk membangkitkan peta kode biner per simbol.

### Ekstraksi Fitur: Histogram of Oriented Gradients
Citra hasil kompresi terpilih didekompresi lalu diubah ke grayscale. Ekstraksi fitur HOG dilakukan dengan membagi gambar ke sel 8 × 8 piksel, tumpang‑tindih blok 2 × 2 sel, dan menghitung histogram gradien dalam 9 orientasi, dinormalisasi L2‑Hys (Dalal & Triggs, 2005). Vektor fitur HOG dari citra terkompresi ini yang kemudian digunakan sebagai input model.

### Klasifikasi: Random Forest
Model Random Forest dilatih hanya pada fitur HOG dari citra yang telah diproses dengan metode kompresi terbaik. Konfigurasi mencakup 200 pohon (n_estimators=200), max_depth 20, serta random_state 42 untuk reproduktibilitas (Breiman, 2001). Data dibagi 80/20 untuk training dan testing, dan evaluasi menggunakan metrik akurasi, precision, recall, dan F1‑score.

### Evaluasi
1. Accuracy, Precision, Recall, F1‑score: untuk menilai ketepatan klasifikasi pada setiap kelas
2. Stratified 5‑Fold Cross‑Validation: untuk menilai stabilitas akurasi di berbagai subset data (Kohavi, 1995).
3. ROC‑AUC One‑vs‑Rest: untuk multiclass discrimination (Fawcett, 2006).

## Result
### Classification Report
![image](https://github.com/user-attachments/assets/80e688b4-6d2d-4399-b5f6-98606be52d44)

Nilai precision dan recall yang tinggi pada hampir semua kelas menunjukkan bahwa model tidak hanya mampu mengenali kelas dengan benar, tetapi juga jarang memberikan prediksi keliru ke kelas lain. Kelas “pipevine” dan “black” sempat menunjukkan kesalahan klasifikasi minor ke beberapa kelas lain, misalnya sebanyak 5 citra kelas “black” salah diklasifikasikan sebagai “pipevine”, dan 2 citra kelas “black” juga diprediksi sebagai “monarch”. Kesalahan ini mungkin terjadi karena kemiripan tekstur atau pola warna sayap antar spesies tersebut. Meskipun demikian, tingkat kesalahan ini masih dalam batas yang dapat ditoleransi dan tidak berdampak signifikan terhadap keseluruhan akurasi model.

### Confusion Matrix
![image](https://github.com/user-attachments/assets/0682eeab-b82a-490b-b915-31a783bf4264)
Confusion matrix menunjukkan bahwa secara umum model Random Forest berhasil mengklasifikasikan sebagian besar citra ke dalam kelas yang benar dengan akurasi tinggi. Kelas “pipevine” mencatat jumlah prediksi benar tertinggi, yaitu 73 dari total 77 citra, yang berarti hanya 4 citra yang salah klasifikasi. Ini menunjukkan bahwa fitur visual pada kelas ini cukup khas dan mudah dikenali oleh model.
Kelas “black” juga menunjukkan performa yang baik dengan 57 prediksi benar dari 64 citra. Namun, terdapat 5 citra yang salah diklasifikasikan sebagai “pipevine” dan 2 lainnya sebagai “monarch”. Kemungkinan besar, hal ini terjadi karena adanya kemiripan tekstur atau warna sayap antara spesies “black” dan “pipevine”, yang cukup mempengaruhi hasil ekstraksi fitur HOG.
Pada kelas “monarch”, dari total 53 citra, sebanyak 48 berhasil diklasifikasikan dengan benar, sementara 3 citra diprediksi sebagai “pipevine” dan 2 sebagai “black”. Ini menunjukkan bahwa meskipun akurasi pada kelas ini tinggi, masih ada tumpang tindih fitur antara spesies “monarch” dan dua kelas lainnya.
Untuk kelas “spicebush”, 39 dari 43 citra diklasifikasikan dengan benar, dengan hanya 2 citra keliru diprediksi sebagai “pipevine” dan 2 lainnya sebagai “black”. Sementara itu, kelas “tiger” mencatat 54 prediksi tepat dari total 61 citra, dan sisanya menyebar ke “black” (5 citra), “pipevine” (1 citra), dan “viceroy” (1 citra). Kesalahan ini mungkin disebabkan karena gradasi warna dan orientasi tepi yang cukup mirip, sehingga fitur HOG tidak dapat sepenuhnya membedakan karakteristik antar kelas.
Terakhir, kelas “viceroy” memiliki performa klasifikasi yang stabil dengan 40 prediksi benar dari 44 citra, dan hanya 4 citra yang salah klasifikasi (2 ke “black” dan 2 ke “spicebush”).
Secara keseluruhan, kesalahan klasifikasi yang terjadi masih tergolong rendah dan cenderung terjadi pada kelas-kelas dengan kemiripan visual yang tinggi. Ini memperkuat indikasi bahwa model memiliki kemampuan generalisasi yang cukup baik, terlebih dengan akurasi total mencapai 90.94% dan nilai ROC AUC multiclass sebesar 0.9843, yang menunjukkan performa diskriminatif model sangat tinggi terhadap semua kelas.

### Learning Curve
![image](https://github.com/user-attachments/assets/4f8b6119-cc81-47d9-9739-781f77d1e68f)
Learning curve menunjukkan hubungan antara jumlah data pelatihan dengan akurasi model, baik pada data pelatihan maupun validasi. Garis biru yang merepresentasikan akurasi pelatihan terlihat stabil di angka 1.0 sepanjang rentang ukuran data pelatihan. Hal ini menunjukkan bahwa model mampu menghafal data pelatihan dengan sangat baik sejak awal, yang merupakan karakteristik umum pada model Random Forest dengan jumlah pohon dan kedalaman yang cukup tinggi.
Sementara itu, garis oranye yang mewakili akurasi validasi mengalami peningkatan secara konsisten seiring bertambahnya jumlah pada data pelatihan. Pada tahap awal (sekitar 140–400 data), akurasi validasi masih di bawah 0.6 karena model belum memiliki informasi yang cukup. Namun, setelah mencapai sekitar 600–800 data, akurasi meningkat tajam hingga mendekati 0.95 pada titik akhir.
Perbedaan antara akurasi pelatihan dan validasi yang semakin kecil menunjukkan bahwa model memiliki kemampuan generalisasi yang baik dan tidak mengalami overfitting. Hal ini mengindikasikan bahwa meskipun data telah melalui proses kompresi dengan metode Huffman, informasi penting tetap terjaga dan mampu mendukung proses klasifikasi dengan akurasi yang tinggi.

## Team
| Nama                        | Email                              |
|-----------------------------|------------------------------------|
| Fadhil Mumtaz               | fadhilmumtaz@apps.ipb.ac.id        |
| Maysa Fazilla Lubis         | fzlmaysa@apps.ipb.ac.id            |
| Nasywa Nozumi               | nasywanozumi@apps.ipb.ac.id        |
| Rio Alvein Hasana           | alveinrio@apps.ipb.ac.id           |
| Tiffany Anastasia Jocelyn  | tiffanyanastasia@apps.ipb.ac.id    |





