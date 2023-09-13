## 8.6 Deadlock Avoidance
Algoritma pencegahan deadlock membatasi permintaan sumber daya untuk mencegah deadlock, tetapi dapat mengakibatkan penggunaan perangkat yang rendah dan throughput sistem yang berkurang. Pendekatan alternatif untuk menghindari deadlock melibatkan pengambilan informasi tambahan tentang bagaimana sumber daya akan diminta. Pendekatan ini memungkinkan sistem untuk memutuskan, untuk setiap permintaan, apakah sebuah utas harus menunggu berdasarkan urutan lengkap permintaan dan pelepasan yang diberikan untuk setiap utas. Sistem mempertimbangkan sumber daya yang tersedia, sumber daya yang dialokasikan untuk setiap utas, dan permintaan dan pelepasan sumber daya di masa depan saat membuat keputusan ini.

Berbagai algoritma menggunakan jumlah dan jenis informasi yang berbeda. Model paling sederhana dan praktis memerlukan setiap utas untuk mendeklarasikan kebutuhan sumber daya maksimumnya untuk setiap jenis sumber daya yang mungkin diperlukan. Dengan informasi ini, algoritma dapat dibangun untuk memastikan bahwa sistem tidak pernah memasuki keadaan deadlock. Algoritma pencegahan deadlock ini secara dinamis menilai status alokasi sumber daya untuk mencegah kondisi antrian berputar. Status alokasi sumber daya ditentukan oleh sumber daya yang tersedia dan dialokasikan serta tuntutan maksimum dari utas-utas. Pada bagian berikutnya, dua algoritma pencegahan deadlock akan dijelaskan secara lebih detail.

Alat Linux "lockdep"

Meskipun memastikan bahwa sumber daya diperoleh dalam urutan yang benar adalah tanggung jawab pengembang kernel dan aplikasi, ada perangkat lunak tertentu yang dapat digunakan untuk memverifikasi bahwa penguncian diperoleh dalam urutan yang benar. Untuk mendeteksi kemungkinan deadlock, Linux menyediakan "lockdep," sebuah alat dengan fungsi kaya yang dapat digunakan untuk memeriksa urutan penguncian dalam kernel. "lockdep" dirancang untuk diaktifkan pada kernel yang sedang berjalan karena memantau pola penggunaan akuisisi dan pelepasan kunci terhadap seperangkat aturan penguncian dan pelepasan. Dua contoh akan diuraikan, namun perlu diperhatikan bahwa "lockdep" menyediakan fungsi yang lebih signifikan daripada yang dijelaskan di sini:

1. Urutan di mana kunci diperoleh secara dinamis dipelihara oleh sistem. Jika "lockdep" mendeteksi bahwa kunci diperoleh di luar urutan, itu melaporkan kondisi deadlock yang mungkin.
    
2. Dalam Linux, "spinlock" dapat digunakan dalam penanganan interrupt. Sumber potensial deadlock terjadi ketika kernel mengakuisisi "spinlock" yang juga digunakan dalam penanganan interrupt. Jika interrupt terjadi saat kunci sedang dipegang, penanganan interrupt akan menggantikan kode kernel yang saat ini memegang kunci dan kemudian berputar sambil mencoba mengakuisisi kunci, mengakibatkan deadlock. Strategi umum untuk menghindari situasi ini adalah dengan menonaktifkan interrupt pada prosesor saat ini sebelum mengakuisisi "spinlock" yang juga digunakan dalam penanganan interrupt. Jika "lockdep" mendeteksi bahwa interrupt diaktifkan saat kode kernel mengakuisisi kunci yang juga digunakan dalam penanganan interrupt, itu akan melaporkan skenario deadlock yang mungkin.
    

"lockdep" dikembangkan sebagai alat dalam pengembangan atau modifikasi kode dalam kernel dan bukan untuk digunakan pada sistem produksi, karena dapat mengurangi kinerja sistem secara signifikan. Tujuannya adalah untuk menguji apakah perangkat lunak seperti driver perangkat baru atau modul kernel memberikan sumber potensial deadlock. Para perancang "lockdep" melaporkan bahwa dalam beberapa tahun setelah pengembangannya pada tahun 2006, jumlah deadlock dari laporan sistem telah berkurang sebesar satu derajat. Meskipun "lockdep" awalnya dirancang hanya untuk digunakan dalam kernel, revisi terbaru dari alat ini sekarang dapat digunakan untuk mendeteksi deadlock dalam aplikasi pengguna yang menggunakan penguncian mutex Pthreads. Informasi lebih lanjut tentang alat "lockdep" dapat ditemukan di [https://www.kernel.org/doc/Documentation/locking/lockdep-design.txt](https://www.kernel.org/doc/Documentation/locking/lockdep-design.txt).

Dalam konteks penghindaran deadlock, ada dua strategi lainnya, yaitu:
1. **Penolakan Inisiasi Proses**: Pendekatan ini melibatkan penolakan inisiasi proses baru jika permintaan sumber daya potensialnya dapat menyebabkan deadlock. Proses baru diizinkan untuk dimulai hanya jika jumlah permintaan sumber daya maksimum potensialnya dan alokasi sumber daya saat ini dari semua proses lain dapat dipenuhi. Metode ini bisa terlalu konservatif karena mengasumsikan skenario terburuk untuk permintaan sumber daya.
    
2. **Penolakan Alokasi Sumber Daya (Algoritma Bankir)**: Strategi ini, dikenal sebagai algoritma bankir, adalah pendekatan yang lebih umum digunakan untuk penghindaran deadlock. Ideanya adalah membuat keputusan alokasi sumber daya sedemikian rupa sehingga sistem selalu tetap dalam keadaan aman, yang berarti ada urutan alokasi sumber daya yang memungkinkan semua proses menyelesaikan tugasnya.
    
    Algoritma ini melacak sumber daya yang tersedia dan sumber daya yang dialokasikan ke proses. Ketika sebuah proses membuat permintaan sumber daya, algoritma mensimulasikan alokasi sumber daya tersebut ke proses tersebut dan memeriksa apakah keadaan yang dihasilkan masih aman. Jika aman, permintaan tersebut disetujui; jika tidak, prosesnya ditangguhkan hingga menjadi aman untuk menyetujui permintaan tersebut.
    
    Pemeriksaan keamanan melibatkan upaya untuk menemukan proses yang dapat menyelesaikan tugasnya menggunakan sumber daya yang saat ini tersedia. Jika ada proses seperti itu, sumber dayanya dilepaskan, dan pemeriksaan diulang. Jika tidak ada proses yang memenuhi syarat, sistem berada dalam keadaan aman.
    

Penting untuk diingat bahwa algoritma bankir tidak dapat memprediksi deadlock dengan pasti, tetapi bertujuan untuk mencegahnya dengan memastikan bahwa alokasi sumber daya selalu dilakukan sedemikian rupa sehingga sistem tetap dalam keadaan aman. Pendekatan ini memerlukan pengetahuan tentang kebutuhan sumber daya maksimum setiap proses sebelumnya dan mengasumsikan bahwa proses-proses tersebut tidak keluar sambil masih memiliki sumber daya.

Meskipun penghindaran deadlock lebih fleksibel daripada pencegahan, pendekatan ini datang dengan overhead dari evaluasi terus-menerus dan potensial pemblokiran permintaan sumber daya. Selain itu, ini memerlukan proses untuk menyebutkan kebutuhan sumber daya maksimum mereka sebelumnya, yang mungkin tidak selalu dapat dilakukan dalam praktikanya.
### 8.6.1 Safe State
Safe state memiliki kondisi seperti penjelasan berikut:
- Suatu keadaan dianggap "aman" jika sistem dapat mengalokasikan sumber daya untuk setiap utas (hingga batas maksimumnya) dalam urutan tertentu dan menghindari deadlock.
- Lebih formal, sistem berada dalam keadaan aman jika ada "urutan aman" utas <T1, T2, ..., Tn>, di mana untuk setiap Ti dalam urutan tersebut, permintaan sumber daya dapat dipenuhi oleh sumber daya yang tersedia saat ini ditambah sumber daya yang dimiliki oleh semua utas Tj, di mana j < i.
- Dalam keadaan aman, jika sumber daya yang dibutuhkan oleh suatu utas tidak tersedia segera, utas tersebut dapat menunggu sampai semua utas sebelumnya dalam urutan selesai, memperoleh sumber daya yang diperlukan, menyelesaikan tugasnya, mengembalikan sumber daya yang dialokasikan, dan berakhir.
- Keadaan deadlock adalah keadaan "tidak aman", tetapi tidak semua keadaan "tidak aman" adalah deadlock. Keadaan "tidak aman" dapat mengarah ke deadlock, tetapi tidak selalu deadlock.
- Keadaan "tidak aman" dipengaruhi oleh perilaku utas; sistem operasi tidak dapat mencegah utas dari permintaan sumber daya dengan cara yang mengarah ke deadlock ketika sistem berada dalam keadaan "tidak aman".

Contoh ini mengilustrasikan konsep ini dengan sistem yang memiliki dua belas sumber daya dan tiga utas: T0, T1, dan T2. Pada waktu tertentu t0, T0 memegang lima sumber daya, T1 memegang dua, dan T2 memegang dua, meninggalkan tiga sumber daya yang tersedia.

Pada waktu t0, sistem berada dalam keadaan aman dengan urutan <T1, T0, T2>, memenuhi kondisi keamanan. Urutan ini memungkinkan setiap utas untuk memperoleh dan melepaskan sumber daya dalam cara yang menjaga sistem dalam keadaan aman.

Sistem dapat berpindah dari keadaan aman ke keadaan "tidak aman" jika alokasi sumber daya tidak diatasi dengan hati-hati. Sebagai contoh, jika pada waktu t1, utas T2 dialokasikan satu sumber daya tambahan, sistem menjadi tidak aman. Dalam keadaan tidak aman, deadlock dapat terjadi jika permintaan sumber daya tidak dikelola dengan benar.

Untuk menghindari deadlock, algoritma pencegahan dirancang untuk memastikan bahwa sistem selalu tetap berada dalam keadaan aman. Algoritma-algoritma ini membuat keputusan tentang apakah permintaan sumber daya dari suatu utas harus dipenuhi segera atau jika utas tersebut harus menunggu. Permintaan diberikan hanya jika itu memastikan bahwa sistem tetap berada dalam keadaan aman.

Meskipun algoritma pencegahan menghindari deadlock, mereka dapat menghasilkan penggunaan sumber daya yang lebih rendah karena utas-utas mungkin harus menunggu alokasi sumber daya, bahkan jika sumber daya tersedia.

### 8.6.2 Resources-Allocation-Graph Algorithm

Dalam algoritma pencegahan deadlock ini, sistem alokasi sumber daya dengan hanya satu instance dari setiap jenis sumber daya yang tersedia dianggap. Untuk mencegah deadlock, sistem menggunakan graf alokasi sumber daya yang mencakup tiga jenis edge: edge permintaan (request edges), edge alokasi (assignment edges), dan edge klaim (claim edges).

- **Edge Permintaan**: Diwakili oleh panah dari utas ke jenis sumber daya, menunjukkan bahwa utas tersebut aktif meminta sumber daya tersebut.
- **Edge Alokasi**: Diwakili oleh panah dari jenis sumber daya ke utas, menunjukkan bahwa sumber daya tersebut dialokasikan ke utas.
- **Edge Klaim**: Diwakili oleh garis putus-putus, edge ini menunjukkan bahwa suatu utas mungkin meminta jenis sumber daya di masa depan. Edge klaim diperkenalkan untuk memungkinkan utas untuk mendeklarasikan kebutuhan sumber daya potensial di masa depan.

Sumber daya harus di-klaim sebelumnya, yang berarti bahwa sebelum utas mulai dieksekusi, semua edge klaimnya harus dibuat. Namun, Anda dapat mengendurkan kondisi ini dengan mengizinkan edge klaim ditambahkan hanya jika semua edge yang terkait dengan utas tersebut adalah edge klaim.

Ketika suatu utas meminta sumber daya (misalnya, T_i meminta R_j), permintaan tersebut hanya dapat dipenuhi jika mengonversi edge klaim yang sesuai T_i → R_j menjadi edge alokasi R_j → T_i tidak membuat siklus dalam graf alokasi sumber daya. Untuk memeriksa keberadaan siklus, digunakan algoritma deteksi siklus, yang biasanya memerlukan O(n^2) operasi, di mana n adalah jumlah utas dalam sistem.

Jika tidak ada siklus yang ditemukan, pemberian permintaan sumber daya akan menjaga sistem dalam keadaan aman. Jika siklus terdeteksi, maka alokasi akan membuat sistem menjadi tidak aman, menunjukkan bahwa utas yang membuat permintaan harus menunggu sampai sumber daya menjadi tersedia. Algoritma ini memastikan bahwa sumber daya dialokasikan dengan cara yang menghindari situasi deadlock potensial.

Penting untuk dicatat bahwa algoritma ini memerlukan klaim sumber daya yang proaktif oleh utas dan memperkenalkan beberapa waktu tunggu jika alokasi sumber daya dapat menghasilkan siklus dalam graf. Meskipun mencegah deadlock, hal ini dapat mengakibatkan penggunaan sumber daya yang lebih rendah karena kebutuhan alokasi sumber daya yang lebih konservatif.

### 8.6.3 Banker's Algorithm
![[Pasted image 20230830201018.png]]
Algoritma bankir adalah algoritma pencegahan deadlock yang digunakan dalam sistem alokasi sumber daya di mana ada beberapa instance dari setiap jenis sumber daya yang tersedia. Algoritma ini dirancang untuk memastikan bahwa alokasi sumber daya tidak pernah mengarah ke situasi deadlock. Algoritma ini sering dibandingkan dengan cara bank mengalokasikan uang tunai yang tersedia kepada pelanggan untuk memastikan bahwa dapat memenuhi semua kebutuhan mereka.

Berikut adalah gambaran cara algoritma bankir bekerja:

1. **Inisialisasi**: Ketika utas baru memasuki sistem, ia harus mendeklarasikan jumlah maksimum instance dari setiap jenis sumber daya yang mungkin dibutuhkan. Deklarasi ini memastikan bahwa tidak ada utas yang dapat meminta lebih banyak sumber daya daripada yang tersedia dalam sistem.
    
2. **Struktur Data**:
    
    - **Available**: Vektor dengan panjang `m` (jumlah jenis sumber daya) yang menunjukkan jumlah sumber daya yang tersedia dari setiap jenisnya. Misalnya, jika `Available[j]` sama dengan `k`, maka ada `k` instance sumber daya jenis `Rj` yang tersedia.
    - **Max**: Matriks `n x m` mendefinisikan kebutuhan maksimum setiap utas. Jika `Max[i][j]` sama dengan `k`, maka utas `Ti` mungkin meminta paling banyak `k` instance sumber daya jenis `Rj`.
    - **Allocation**: Matriks `n x m` menunjukkan jumlah sumber daya dari setiap jenis yang saat ini dialokasikan ke setiap utas. Jika `Allocation[i][j]` sama dengan `k`, maka utas `Ti` saat ini dialokasikan `k` instance sumber daya jenis `Rj`.
    - **Need**: Matriks `n x m` menunjukkan kebutuhan sumber daya yang tersisa dari setiap utas. Jika `Need[i][j]` sama dengan `k`, maka utas `Ti` mungkin membutuhkan tambahan `k` instance sumber daya jenis `Rj` untuk menyelesaikan tugasnya. Perlu diperhatikan bahwa `Need[i][j]` sama dengan `Max[i][j] - Allocation[i][j]`.
3. **Notasi**: Algoritma ini menggunakan notasi untuk membandingkan vektor dan memeriksa ketersediaan sumber daya. Misalnya, `X ≤ Y` berarti bahwa vektor `X` kurang dari atau sama dengan vektor `Y`. Demikian pula, `Y < X` berarti bahwa vektor `Y` kurang dari vektor `X`.
    
4. **Permintaan dan Alokasi Sumber Daya**:
    
    - Ketika utas meminta serangkaian sumber daya, sistem memeriksa apakah mengalokasikan sumber daya ini akan menjaga sistem dalam keadaan aman. Keadaan aman adalah keadaan di mana ada urutan di mana semua utas dapat menyelesaikan tugasnya tanpa mengalami deadlock.
    - Sistem memeriksa apakah `Request ≤ Need`, di mana `Request` adalah vektor permintaan sumber daya, dan `Need` adalah vektor kebutuhan sumber daya yang tersisa untuk utas tersebut.
    - Jika `Request ≤ Need`, sistem selanjutnya memeriksa apakah `Request ≤ Available`. Jika kondisi ini terpenuhi, maka sumber daya dialokasikan.
    - Jika sumber daya tidak dapat dialokasikan segera, utas yang meminta harus menunggu hingga sumber daya menjadi tersedia tanpa melanggar kondisi keamanan.
5. **Pelepasan Sumber Daya**: Ketika utas selesai menggunakan sumber daya yang dialokasikan kepadanya, ia melepaskan sumber daya tersebut. Sistem kemudian memperbarui matriks `Available`, `Allocation`, dan `Need` sesuai.
    

Algoritma bankir memastikan bahwa sumber daya dialokasikan dengan cara yang menghindari situasi deadlock dengan memeriksa dan menjaga keadaan aman sistem. Namun, ini dapat mengakibatkan penggunaan sumber daya yang lebih rendah karena pendekatan alokasi yang konservatif.

##### 8.6.3.1 Safety Algorithm

Algoritma keamanan digunakan untuk menentukan apakah sistem berada dalam keadaan aman, yang berarti ada urutan eksekusi utas yang menghindari deadlock. Inilah cara algoritma tersebut bekerja:

1. **Inisialisasi**:
    
    - Inisialisasi dua vektor:
        - `Work`: Vektor dengan panjang `m` (jumlah jenis sumber daya) diinisialisasi dengan jumlah instance yang tersedia dari setiap jenis sumber daya. `Work[j]` mewakili instance yang tersedia dari jenis sumber daya `Rj`.
        - `Finish`: Vektor dengan panjang `n` (jumlah utas) diinisialisasi dengan `false` untuk semua utas. `Finish[i]` menunjukkan apakah utas `Ti` telah menyelesaikan eksekusinya (awalnya diatur ke `false` untuk semua utas).
2. **Mencari Utas yang Akan Dieksekusi**:
    
    - Algoritma mencari indeks `i` yang memenuhi kedua kondisi berikut: a. `Finish[i]` adalah `false`, yang berarti utas `Ti` belum selesai dieksekusi. b. `Needi ≤ Work`, menunjukkan bahwa kebutuhan sumber daya dari utas `Ti` dapat dipenuhi oleh sumber daya yang tersedia (`Work` vektor).
    - Jika tidak ada `i` yang memenuhi kondisi tersebut, itu berarti tidak ada utas yang kebutuhan sumber dayanya dapat dipenuhi dengan sumber daya yang tersedia. Dalam hal ini, sistem tidak dalam keadaan aman.
3. **Mengalokasikan Sumber Daya dan Menandai Utas Sebagai Selesai**:
    
    - Jika ditemukan indeks `i` yang memenuhi kondisi pada langkah 2, algoritma melanjutkan untuk mengalokasikan sumber daya ke utas `Ti`:
        - `Work = Work + Allocationi`: Menambahkan vektor `Work` dengan sumber daya yang dialokasikan ke utas `Ti`.
        - `Finish[i] = true`: Menandai utas `Ti` sebagai selesai.
    - Setelah alokasi ini dan penandaan, algoritma kembali ke langkah 2 untuk mencari utas berikutnya yang dapat dieksekusi.
4. **Pemeriksaan Keadaan Aman**:
    
    - Jika semua utas telah ditandai sebagai selesai (`Finish[i] == true` untuk semua `i`), maka sistem berada dalam keadaan aman. Ini berarti ada urutan di mana semua utas dapat menyelesaikan tugasnya tanpa menghadapi deadlock.

Algoritma keamanan melakukan iterasi melalui utas-utas dan mengalokasikan sumber daya jika kebutuhan utas dapat dipenuhi dengan sumber daya yang tersedia saat ini. Jika semua utas dapat ditandai sebagai selesai pada akhir algoritma, ini menunjukkan bahwa sistem berada dalam keadaan aman. Namun, penting untuk dicatat bahwa algoritma keamanan hanya memeriksa apakah suatu keadaan tertentu aman; itu tidak memberikan solusi untuk alokasi sumber daya. Algoritma bankir, yang telah kita diskusikan sebelumnya, menggunakan algoritma keamanan ini sebagai bagian fundamental untuk memastikan alokasi sumber daya menghindari deadlock sambil menjaga keadaan aman dalam sistem.

##### 8.6.3.2 Resource-Request Algorithm

Algoritma permintaan sumber daya digunakan untuk menentukan apakah permintaan sumber daya tambahan dari utas dapat diberikan dengan aman tanpa mengarah ke deadlock. Berikut adalah cara algoritma ini bekerja:

1. **Validasi Permintaan**:
    
    - Ketika utas `Ti` membuat permintaan sumber daya, yang direpresentasikan oleh vektor `Requesti`, algoritma pertama-tama memvalidasi permintaan tersebut.
    - Jika `Requesti ≤ Needi`, ini berarti bahwa permintaan sumber daya yang diminta tidak melebihi klaim maksimum utas, jadi permintaan tersebut valid. Namun, kondisi ini dilanggar jika utas telah melampaui klaim maksimumnya.
2. **Pemeriksaan Ketersediaan Sumber Daya**:
    
    - Setelah validasi permintaan, algoritma memeriksa apakah sumber daya yang diminta saat ini tersedia.
    - Jika `Requesti ≤ Available`, ini berarti bahwa ada cukup sumber daya yang tersedia untuk memenuhi permintaan tersebut, jadi algoritma melanjutkan ke langkah berikutnya. Jika kondisi ini tidak terpenuhi, utas harus menunggu karena sumber daya yang diminta saat ini tidak tersedia.
3. **Simulasi Alokasi Sumber Daya**:
    
    - Pada tahap ini, algoritma mensimulasikan alokasi sumber daya yang diminta ke utas `Ti` dengan memodifikasi keadaan sistem. Perubahan-perubahan berikut dilakukan:
        - `Available = Available - Requesti`: Mengurangi sumber daya yang tersedia sesuai dengan jumlah yang diminta.
        - `Allocationi = Allocationi + Requesti`: Menambahkan alokasi sumber daya yang diminta ke utas `Ti`.
        - `Needi = Needi - Requesti`: Mengurangi kebutuhan sumber daya yang tersisa untuk utas `Ti` sesuai.
4. **Pemeriksaan Keamanan**:
    
    - Setelah mensimulasikan alokasi, algoritma memeriksa apakah keadaan alokasi sumber daya yang dihasilkan aman menggunakan algoritma keamanan yang telah dijelaskan sebelumnya.
    - Jika keadaan baru aman, ini berarti bahwa memberikan permintaan tersebut tidak akan mengarah ke deadlock, sehingga transaksi selesai, dan utas `Ti` dialokasikan sumber daya yang diminta.
5. **Penanganan Pelanggaran Keamanan**:
    
    - Jika keadaan baru tidak aman, menunjukkan bahwa memberikan permintaan tersebut dapat mengarah ke deadlock, maka utas `Ti` harus menunggu permintaannya. Dalam hal ini, keadaan alokasi sumber daya sebelumnya (sebelum simulasi) dipulihkan.

Algoritma permintaan sumber daya memastikan bahwa permintaan sumber daya diberikan dengan cara yang menjaga keamanan sistem. Jika permintaan dapat diberikan tanpa mengancam keamanan sistem, sumber daya dialokasikan ke utas. Jika memberikan permintaan akan mengarah ke keadaan yang tidak aman, utas harus menunggu, dan keadaan sebelumnya dipulihkan. Algoritma ini adalah komponen fundamental dari algoritma bankir, yang bertujuan untuk memastikan alokasi sumber daya menghindari deadlock sambil menjaga keadaan aman dalam sistem.

