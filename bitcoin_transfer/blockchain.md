## Blockchain {#blockchain}

Anda mungkin telah bisa memperhatikan bahwa saat kita dapat membuktikan kepemilikan pada transaksi pengeluaran TxOut, dan bahwa kita belum benar-benar membuktikan TxOut itu benar-benar ada. Di sinilah fungsi utama dari Blockchain yang membuatnya bersinar:

Blockchain adalah sebuah database, menyimpan semua transaksi yang telah terjadi sejak transaksi bitcoin pertama dilakukan, atau yang dikenal dengan sebutan **Genesis Block**. Blockchain ini dapat disimpan dan diduplikasi diseluruh dunia. Jika anda menggunakan Bitcoin Core, maka anda menyimpan keseluruhan Blockchain tersebut pada komputer anda. Sekali transaksi muncul di dalam Blockchain, maka sangat mudah untuk membuktikan keberadaannya.

Penambang adalah entitas yang hanya bertujuan untuk memasukkan transaksi ke dalam Blockchain. Meski demikian, penambang tidak mengubah Blockchain tersebut setiap mereka menerima satu transaksi. Saat penambang mencoba menambah seluruh \_batch \_transaksi, pada saat yang sama node lain di jaringan mengkonfirmasi block baru dengan memperhatikan dan mematuhi aturan yang telah ditetapkan dalam protokol Bitcoin.

Setelah penambang berhasil mengirim blok yang valid beserta semua transaksi yang ada didalamnya, maka penambang dapat memulai lagi dengan transaksi yang baru. Setelah block tersebut dikonfirmasi dan dimasukkan ke dalam Blockchain, kemungkinan transaksi itu batal menurun sangat drastis dengan setiap block berikutnya.

Dalam hal ini membuat pertama kalinya dalam sejarah, kita memiliki database yang tidak dapat ditulis ulang, menghilangkan kebutuhan tentang "_trust_", resisten atas sensor, dan dapat didistribusikan secara luas. Membandingkan Blockchain dengan sebuah buku besar _\(ledger\)_, hanya relevan jika kita mempertimbangkan Bitcoin sebagai mata uang.

Bitcoin adalah database, dan anda dapat memberikan makna sebagai data. Ketika anda berhasil menemukan, transaksi bitcoin dapat menangani informasi lebih dari sekedar transfer bitcoin. Sebagai pengguna, anda dapat memverifikasi keberadaan transaksi tertentu di dalam Blockchain dalam dua cara yang berbeda: Memeriksa seluruh Blockchain, yang saat ini berukuran beberapa gigabyte. Kedua dengan memeriksa pada merkel tree, yang hanya berukuran beberapa kilobyte saja. Tentang merkel tree ini, kita akan membahas lebih jauh pada bahasan Simple Payment Verification.

