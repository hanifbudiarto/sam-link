---
icon: book-blank
---

# Project Configuration

Ada beberapa cara untuk mengonfigurasi proyek, tergantung pada IDE yang digunakan. Cara yang paling umum adalah dengan menggunakan perintah **idf.py menuconfig**. Perintah ini membuka TUI (Text-based User Interface) atau Antarmuka Pengguna Berbasis Teks tempat pengguna dapat mengatur opsi konfigurasi. Konfigurasi disimpan ke dalam file sdkconfig.

## Configuration with Menuconfig

Berikut cara membuka menuconfig.

1. Buka ESP-IDF Terminal

<figure><img src="../.gitbook/assets/Screenshot 2025-04-29 084445.png" alt="" width="236"><figcaption></figcaption></figure>

2. Ketikkan pada terminal tersebut, idf.py menuconfig. Maka akan muncul tampilan sebagai berikut.

<figure><img src="../.gitbook/assets/Screenshot 2025-04-28 171153.png" alt="" width="563"><figcaption></figcaption></figure>

3. Sesuaikan ukuran flash size dengan device Anda. Masuk ke dalam menu **Serial flasher config -> Flash size** . Disini ukuran flash size nya 4 MB.

<figure><img src="../.gitbook/assets/Flash size 4mb.png" alt="" width="563"><figcaption></figcaption></figure>

4. Jika pada saat pengembangan aplikasi menemukan error yang berkaitan dengan main task size, kemungkinan ukurannya kurang besar. Temukan pada menu **Component config -> ESP System Settings -> Main task stack size**

<figure><img src="../.gitbook/assets/Main task size.png" alt="" width="559"><figcaption></figcaption></figure>

5. Jika saat pengembangan didapati error yang berkaitan dengan panjang http uri, maka kemungkinan ukurannya kurang panjang. Temukan pada menu **Component config -> HTTP Server -> Max HTTP URI Length**

<figure><img src="../.gitbook/assets/max http uri.png" alt="" width="563"><figcaption></figcaption></figure>

6. Jangan lupa untuk menentukan default log level. Temukan pada menu **Component config -> Log -> Log Level -> Default log verbosity**

<figure><img src="../.gitbook/assets/Screenshot 2025-05-10 234818.png" alt=""><figcaption></figcaption></figure>

## Configuration with CMake

Dalam panduan ini, Anda akan mempelajari cara memodifikasi file CMakeLists.txt untuk menyiapkan lingkungan untuk ESP-IDF VS Code. CMakeLists.txt digunakan oleh CMake untuk mengonfigurasi sistem build yang digunakan oleh ESP-IDF.

1. Buka CMakeLists.txt yang terletak di direktori root. Saat project pertama kali dibuat maka isinya akan seperti ini.

```
//
   file : /CMakeLists.txt 
//

cmake_minimum_required(VERSION 3.16)
set(COMPONENTS main)
include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(myfirstproject)

```

2. Anda dapat menambahkan komponen lain dan mendaftarkannya melalui CMakeLists. Disini ditambahkan extra component dengan menaruh sebuah folder baru pada root directory dengan nama components.&#x20;

<pre><code>//
   file : /CMakeLists.txt 
//

cmake_minimum_required(VERSION 3.16)
set(COMPONENTS main)

<strong>set(EXTRA_COMPONENT_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/components)
</strong>
include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(myfirstproject)
</code></pre>

3. Anda juga dapat menentukan nama dan version dari project Anda sekarang. Gunanya agar file firmware .bin yang dibentuk dapat disesuaikan dengan nama dan versinya.

```
//
   file : /CMakeLists.txt 
//

cmake_minimum_required(VERSION 3.16)
set(COMPONENTS main)
set(EXTRA_COMPONENT_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/components)
include($ENV{IDF_PATH}/tools/cmake/project.cmake)

set(PROJECT_VER "1.1.0")
set(PROJECT_NAME "My_First_Project")
project(${PROJECT_NAME}_${PROJECT_VER})
```

Anda dapat juga melakukan konfigurasi lain menggunakan CMakeLists.txt yang terletak dalam folder main. Bedanya perubahan file cmake pada folder main dilakukan untuk mengkonfigurasi aplikasi yang sedang Anda buat. Berikut salah satu contoh pengaturan main->CMakeLists.txt.

```
//
    file: main/CMakeLists.txt
//

idf_component_register(SRCS "main.c"
EMBED_TXTFILES index.html success.html dashboard.html
PRIV_REQUIRES spi_flash
REQUIRES esp_http_client mbedtls
INCLUDE_DIRS ".")
```

Seperti yang Anda lihat pada contoh di atas, project ini membutuhkan library lain yang dapat diimport melalui CMakeLists.txt seperti **spi\_flash, esp\_http\_client dan mbedtls**. Anda juga dapat mengimpor file textfile melalui EMBED\_TXTFILES. &#x20;
