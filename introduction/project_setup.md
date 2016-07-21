## Project Setup {#project-setup}

Sebelum kita mulai, harus dijelaskan dahulu bagaimana kita mempersiapkan dalam membuat sebuah project.

1. Buat sebuah Project baru di Visual Studio \(.NET 4.5 atau diatasnya\)
2. Klik kanan pada menu “**References**” di **Solution Explorer** lalu pilih “**Manage NuGet Packages**…”
3. Cari “**NBitcoin”** kemudian install. \(atau **NBitcoin.Mono** untuk MAC dan Linux.\)
  ![](../assets/nuget.png)  

> **Tips:** Jika anda menggunakan MAC atau Linux, namun memakai NBitcoin bukannya NBitcoin.Mono, mungkin menemui beberapa _classes_ yang hilang.

NBitcoin adalah library .NET Bitcoin, dan open-source, dibuat oleh **Nicolas Dorier**, penulis utama buku ini. 
Libryry ini harus selalu digunakan jika anda melakukan apa saja dengan Bitcoin yang berkaitan dengan C\#.   
NBitcoin sendiri, telah support untuk aplikasi yang bersifat _cross-platform_.

### \#\# \# Cara _debug_ di NBitcoin source code \(pendukung\)

NBitcoin dapat melakukan proses debug pada kode, untuk memudahkan anda. Agar fitur ini dapat berjalan, pastikan anda telah mengaktifkan fitur ini di Visual Studio \(Tools\/Options\). Lihat gambar di bawah: 
![](../assets/visualstudio_enablesourceserversupport.png)

Sekarang, jika anda melangkah ke kode Bitcoin, source code itu akan secara otomatis diambil dari github, dan muncul di debugger visual studio anda. .

