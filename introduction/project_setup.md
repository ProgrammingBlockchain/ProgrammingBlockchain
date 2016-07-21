## Project Setup {#project-setup}

Sebelum kita mulai, harus dijelaskan dahulu bagaimana kita mempersiapkan dalam membuat sebuah project.

1. Buat sebuah Project baru di Visual Studio \(.NET 4.5 atau diatasnya\)
2. Klik kanan pada menu “**References**” di **Solution Explorer** lalu pilih “**Manage NuGet Packages**…”
3. Cari “**NBitcoin”** kemudian install. \(atau **NBitcoin.Mono** untuk MAC dan Linux.\)
  ![](../assets/nuget.png)  

> **Tips:** Jika anda menggunakan MAC atau Linux, namun memakai NBitcoin bukannya NBitcoin.Mono, mungkin menemui beberapa _classes_ yang hilang.

NBitcoin adalah library .NET Bitcoin, dan open-source, dibuat oleh **Nicolas Dorier**, penulis utama buku ini. 
This library should always be included if you do anything Bitcoin related in C\#.  
NBitcoin supports cross-platform applications.

### \#\# \# How to debug into NBitcoin source code \(optional\)

NBitcoin lets you debug into its code, to make your life easier. For this feature to work make sure you have source server support enabled in Visual Studio \(Tools\/Options\).   
![](../assets/visualstudio_enablesourceserversupport.png)

Now, if you step into NBitcoin code, the source code will be automatically fetched on github, and appear in visual studio debugger.

