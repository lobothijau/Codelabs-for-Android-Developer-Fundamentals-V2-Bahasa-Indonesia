# Android fundamentals 09.1: Shared preferences

## Selamat Datang

Codelab ini merupakan bagian dari [Unit 4: Saving user data](https://developer.android.com/courses/fundamentals-training/toc-v2#unit_4_saving_user_data) dalam kelas belajar Android Developer Fundamentals (Versi 2). Kamu akan mendapatkan pemahaman yang lebih lengkap jika mengikuti codelab ini secara berurutan:

- Untuk daftar codelabs di kelas ini, silahkan lihat [Codelabs Android Developer Fundamentals (V2)](https://developer.android.com/courses/fundamentals-training/toc-v2).
- Informasi lebih lengkap tentang kelas, termasuk tautan ke bab-bab konsep, contoh aplikasi, dan slide, silahkan lihat halaman [Android Developer Fundamentals (Versi 2)](https://developer.android.com/courses/fundamentals-training/overview-v2).

> **Catatan**: Istilah “codelab” dan “praktik” akan sering dipakai secara bergantian tanpa mengubah makna. 

## Pendahuluan

Shared preferences memungkinkan kita untuk menyimpan data sederhana dalam bentuk pasangan  key/value di dalam sebuah file yang tersimpan di tiap-tiap perangkat. Untuk mendapatkan akses ke sebuah file preference, juga mendapat akses menulis, membaca, serta mengelola preference data, gunakan kelas [`SharedPreferences`](https://developer.android.com/reference/android/content/SharedPreferences). Android framework mengelola file shared preferences sendiri. File ini bisa diakses oleh semua komponen yang ada di dalam aplikasi sendiri, tapi tidak bisa diakses oleh aplikasi lain. 

Data yang di simpan di sebuah shared preferences berbeda dengan data yang disimpan di dalam status activity yang pernah kita pelajari di beberapa bab sebelumnya:

- Data yang disimpan di activity hanya bertahan selama activity hidup di satu sesi. 
- Shared preference disimpan secara permanen, tetap akan bertahan meski aplikasi di stop, di restart, maupun perangkat di reboot. 

Gunakan shared preferences hanya jika perlu menyimpan data sederhana dalam bentuk pasangan key/value. Untuk mengelola jumlah data aplikasi yang lebih kompleks, gunakan sistem penyimpanan yang lebih canggih seperti library Room atau database SQL. 

### Apa yang kamu seharusnya sudah pelajari

Untuk mengikuti modul ini, kamu harus sudah mengerti:

- Membuat, membangun, serta menjalankan aplikasi di Android Studio. 
- Mendesain layout dengan button dan text view. 
- Menggunakan styles dan themes. 
- Menyimpan dan me-restore activity instsance state. 

### Apa yang akan kamu pelajari

- Mengidentifikasi apa itu shared preferences.
- Membuat sebuah shared perefences di aplikasi. 
- Menyimpan data ke shared preferences, dan membaca data yang sudah disimpan.
- Membersihkan data dari shared preferences.

### Apa yang akan kamu lakukan

- Meng-update sebuah aplikasi sehingga aplikasi ini bisa menyimpan, mengambil, dan me-reset shared preferences.



## Gambaran aplikasi

Aplikasi HelloSharedPrefs adalah variasi dari aplikasi HelloToast yang kita buat di pelajaran pertama. Aplikasi ini memiliki tombol untuk men-increment angka, mengubah warna latar serta me-reset angka dan warnya ke pengaturan awal. Aplikasi ini juga menggunakan theme dan styles untuk mendefinisikan button. 

![img](https://codelabs.developers.google.com/codelabs/android-training-shared-preferences/img/5dfd63922ee74727.png)

Kita akan mulai dengan sebuah starter app dan menambahkan shared preferences ke kode main activity-nya. Kita juga bisa menambahkan sebuah tombol reset yang kemudian mengubah angka dan warna latar ke pengaturan semua serta membersihkan isi file preferences.

## Tahap 1: Eksplor HelloSharedPrefs 

Starter app untuk codelab ini tersedia di [HelloSharedPrefs-Starter](https://github.com/google-developer-training/android-fundamentals-starter-apps-v2/tree/master/HelloSharedPrefs). Di tahap ini kita akan memuat project tadi ke Android Studio dan mengeksplor beberapa fitur utamanya. 

### 1.1 Buka dan jalankan project HelloSharedPrefs

1. Download kode [HelloSharedPrefs-Starter](https://github.com/google-developer-training/android-fundamentals-starter-apps-v2/tree/master/HelloSharedPrefs).
2. Buka project ini di Android Studio lalu jalankan aplikasinya. 

Coba aplikasi tadi dengan menjalankan tahapan berikut:

1. Klik tombol **Count** untuk men-increment angka di text view utama.
2. Klik tombol pilihan warna untuk mengganti warna latar text view utama. 
3. Putar layar dan pastikan warna serta angka tidak berubah.
4. Klik tombol  **Reset** untuk mengubah warna serta angka ke pengaturan semula. 

Sekrang coba lagi aplikasi tadi setelah ditutup dan dibuka ulang:

1. Tutup secara paksa aplikasi dengan salah satu cara berikut:
   - Di Android Studio, pilih **Run > Stop 'app'** atau klik gambar ikon Stop ![img](https://codelabs.developers.google.com/codelabs/android-training-shared-preferences/img/2fd848e8888a39ed.png) di toolbar.
   - Di perangkat, tekan tombol Recents (tombol kotak yang berada di sebelah tombol home). Swipe aplikasi HelloSharedPrefs untuk menutupnya atau klik tombol X dipojok kanan atas. Tunggu beberapa saat agar sistem menutup aplikasi ini secara sempurna. 

2. Muat ulang aplikasi. Aplikasi ini akan muncul dengan pengaturan awal, angka menjadi 0 dan warna latar menjadi abu-abu. 

### 1.2 Eksplor kode Activity

1. Buka  `MainActivity`.
2. Perhatikan kodenya.

Perhatikan beberapa hal berikut:

- Variabel `mCount` di definisikan sebagai sebuah integer. Method onClick `countUp()` menaikkan angkanya serta mengubah isi  `TextView`.
- Variabel warna `mColor` juga sebuah integer yang nilai awalnya adalah warna abu-abu yang telah dibuat dalam file  `colors.xml` sebagai `default_background`.
- Method onClick `changeBackground()` mengambil warna latar masing-masing button yang diklik untuk mengubah warna text view utama dengan warna tersebut. 
- Baik variavel  `mCount` maupun `mColor` keduanya disimpan di dalam bundle instance state di `onSaveInstanceState()`, lalu di restore di  `onCreate()`. Key untuk  bundle count dan color dibuat dalam sebuah variabel konstan private bernama `COUNT_KEY` dan `COLOR_KEY`.



## Tahap 2: Save dan restore data ke dalam file shared preferences

Di tahap ini kita akan menyimpan data aplikasi ke dalam sebuah file shared preferences, keudian membaca kembali data tersebut ketika aplikasi di restart. Karena data yang disimpan di state dengan shared preferences **sama**, maka kita bisa mengganti penggunaan instance state dengan shared preference.

### 2.1 Inisialisasi preferences

1. Tambahkan sebuah member variabel ke kelas  `MainActivity` untuk menyimpan nama file shared preferences, dan sebuah objek berjenis `SharedPreferences`.

   ```java
   private SharedPreferences mPreferences;
   private String sharedPrefFile = 
      "com.example.android.hellosharedprefs";
   ```

   Kita bisa memberi nama preferences ini dengan nama apapun, tapi biasanya akan mengikuti package name aplikasi. 

2. Di dalam method `onCreate()` method, inisialisasi shared preferences. Tambahkan kode berikut sebelum perintah `if`:

   ```java
   mPreferences = getSharedPreferences(sharedPrefFile, MODE_PRIVATE);
   ```

   Method [`getSharedPreferences()`](https://developer.android.com/reference/android/content/Context#getSharedPreferences(java.lang.String,%20int)) (dari activity [`Context`](https://developer.android.com/reference/android/content/Context)) membuka file yang telah ditentukan namanya (`sharedPrefFile`) dengan mode `MODE_PRIVATE`.

   > **Catatan:** Versi lama Android memiliki mode yang memungkinkan file shared preference untuk bisa di baca dan ditulis oleh aplikasi lain. Mode ini telah di deprecated-kan di API 17, dan sekarang tidak disarankan untuk mengaktifkannya karena alasan keamanan. Jika ingin berbagi data dengan aplikasi lain, pertimbangkan untuk menggunakan URI dari kelas [`FileProvider`](https://developer.android.com/reference/android/support/v4/content/FileProvider).

Solusi `MainActivity` untuk saat ini:

```java
public class MainActivity extends AppCompatActivity {
   private int mCount = 0;
   private TextView mShowCount;
   private int mColor;

   private SharedPreferences mPreferences;
   private String sharedPrefFile = 
      "com.example.android.hellosharedprefs";

   @Override
   protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.activity_main);

       mShowCount = (TextView) findViewById(R.id.textview);
       mColor = ContextCompat.getColor(this, 
          R.color.default_background);
       mPreferences = getSharedPreferences(
          sharedPrefFile, MODE_PRIVATE);

       // ...
    }
}
```

### **2.2 Simpan preferences di onPause()**

Menyimpan preferences tidak seperti menyimpan instance state meskipun sama-sama menggunakan objek Bundle. Untuk shared preferences di project ini, kita menyimpan datanya di dalam callback lifecycle  `onPause()`  dan kita akan memerlukan sebuah objek shared editor ([`SharedPreferences.Editor`](https://developer.android.com/reference/android/content/SharedPreferences.Editor.html)) untuk menuliskannya ke dalam file shared preferences. 

1. Tambahkan method lifecycle `onPause()` di MainActivity:

   ```java
   @Override
   protected void onPause(){
      super.onPause();
   
      // ...
   }
   ```

   

2. Di dalam `onPause()`, ambil sebuah editor dari objek `SharedPreferences`:

   ```java
   SharedPreferences.Editor preferencesEditor = mPreferences.edit();
   
   ```

   Sebuah editor shared preferences diperlukan untuk menuliskan datanya ke dalam objek shared preferences. Tambahkan baris tadi ke atas pemanggilan`super.onPause()`.

    

3. Gunakan method [`putInt()`](https://developer.android.com/reference/android/content/SharedPreferences.Editor#putInt(java.lang.String,%20int)) untuk menyimpan data `mCount` dan `mColor` ke dalam shared preferences  dengan key yang sudah ada:

   ```java
   preferencesEditor.putInt(COUNT_KEY, mCount);
   preferencesEditor.putInt(COLOR_KEY, mColor);
   ```

   Kelas `SharedPreferences.Editor` memiliki beberapa method "put" untuk beberapa tipe data diataranya [`putInt()`](https://developer.android.com/reference/android/content/SharedPreferences.Editor#putInt(java.lang.String,%20int)) dan [`putString()`](https://developer.android.com/reference/android/content/SharedPreferences.Editor#putString(java.lang.String,%20java.lang.String)).

   

4. Panggil `apply()` untuk menyimpannya:

   ```java
   preferencesEditor.apply();
   ```

   Method [`apply()`](https://developer.android.com/reference/android/content/SharedPreferences.Editor#apply()) akan menyimpan preference secara asynchronous, diluar UI thread. Shared preferences editor juga memiliki method bernama [`commit()`](https://developer.android.com/reference/android/content/SharedPreferences.Editor#commit()) untuk melakukan penyimpanan preference secara synchronous. Method `commit()` kurang disarankan karena bisa memblok operasi di UI thread.

    

5. Hapus method `onSaveInstanceState()`. Karena activity instance state akan memiliki data yang sama dengan shared preference, maka kit bisa mengganti seluruh instance state

Solusi untuk method `onPause()`  `MainActivity` :

```
@Override
protected void onPause(){
   super.onPause();

   SharedPreferences.Editor preferencesEditor = mPreferences.edit();
   preferencesEditor.putInt(COUNT_KEY, mCount);
   preferencesEditor.putInt(COLOR_KEY, mColor);
   preferencesEditor.apply();
}
```



### 2.3 Restore preference di onCreate()

Sama seperti instance state, aplikasi kita perlu membaca isi shared preferences di method `onCreate()` agar bisa menampilkan data yang lama. Sekali lagi, karena shared preferences memiliki data yang sama dengan instance stte, maka kita bisa mengganti state dengan kode-kode shared preferenes juga di sini. Setiap kali `onCreate()` dipanggil, yaitu saat aplikasi berjalan, atau terjadi configuration chages, maka shared prefrerence akan dipakai untuk me-restore state view.  

1. Hapus bagian dimana terdapat pemeriksaan apakah `savedInstanceState` bernilai null dan me-restore instance state di dalam method  `onCreate()` :

    ```java
   if (savedInstanceState != null) {
      mCount = savedInstanceState.getInt(COUNT_KEY);
      if (mCount != 0) {
          mShowCountTextView.setText(String.format("%s", mCount));
      }
      mColor = savedInstanceState.getInt(COLOR_KEY);
      mShowCountTextView.setBackgroundColor(mColor);
   }
    ```

2. Hapus blok di atas (bukan blok `onCreate`-nya).

3. Di dalam method `onCreate`, dibagian yang baru saja di hapus, ambil variabel count dari dalam preferences dengan key `COUNT_KEY` dan simpan ke dalam variabel lokal `mCount`. 

   ```java
   mCount = mPreferences.getInt(COUNT_KEY, 0);
   
   ```

   Saat mengambil data dari preferences, kita tidak perlu sebuah shared preferences editor. Gunakan method "get" yang sesuai dengan tipe data di dalam objek shared preferences (misalnya [`getInt()`](https://developer.android.com/reference/android/content/SharedPreferences#getInt(java.lang.String,%20int))atau [`getString()`](https://developer.android.com/reference/android/content/SharedPreferences#getString(java.lang.String,%20java.lang.String)) untuk mengambil data dari dalam preference.

   Perhatikan bahwa `getInt()` meminta dua argumen: satu untuk key dan satu lagi untuk default value jika key yang bersangkutan tidak ditemukan. Dalam kasus ini kita akan berikan nilai 0 jika key tersebut tidak ada dan akan menjadi nilai awal `mCount`. 

4. Perbarui konten `TextView` utama dengan nilai yang sudah di simpan di dalam variabel `mCount`. 

5. Ambil warna dari dalam preferences dengan `COLOR_KEY` dan simpan ke dalam variable `mColor`.

   ```java
   mColor = mPreferenes.getInt(COLOR_KEY, mColor);
   ```

   Sama seperti langkah sebelumnya,  `getInt()` memerlukan sebuah default value jika key yang diinginkan tidak ada. Dalam kasus ini, kia akan memberikan nilai yang tersimpan di dalam variabel `mColor` yang sebelumnya sudah diisi dengan warna latar default. 

6. Perbarui warna latar text view. 

   

7. Jalankan aplikasi. Klik tombol **Count** dan coba ubah warna latar untuk menguji instance state dan shared preference. 

8. Putar layar atau emulator untuk memastikan bahwa count dan color tersimpan saat terjadi configuration change .

9. Keluar dari aplikasi dengan salah satu cara berikut:

   - di Android Studio, pilih **Run > Stop 'app'**
   - di perangkat, sentuh tombol Recents (bergambar kotak di sebelah tombol home). Swipe aplikasi HelloSharedPrefs untuk keluar dari aplikasi, atau klik tombol X yang ada di pojok kanan atas. 

10. Jalankan kembali aplikasinya. Aplikasi akan di restart dan memuat isi preferences sehingga data yang sudah diubah tetap sama. 

Solusi untuk method `onCreate()` `MainActiivty`:

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   setContentView(R.layout.activity_main);

   // Initialize views, color, preferences
   mShowCountTextView = (TextView) findViewById(R.id.count_textview);
   mColor = ContextCompat.getColor(this, R.color.default_background);
   mPreferences = getSharedPreferences(
                         mSharedPrefFile, MODE_PRIVATE);

   // Restore preferences
   mCount = mPreferences.getInt(COUNT_KEY, 0);
   mShowCountTextView.setText(String.format("%s", mCount));
   mColor = mPreferences.getInt(COLOR_KEY, mColor);
   mShowCountTextView.setBackgroundColor(mColor);
}
```

### 2.4 **Reset preference di reset() click handler**

Tombol rset di aset starter app sudah bisa mengatur agar count dan color activity berubah ke sedia kalah. Karena saat ini preference sudah menyimpan data tersebut, maka kita juga perlu me-reset isi preference. 

1. Di dalam mehod onClick  `reset()`, setelah color dan count di reset, buat sebuah editor dari objek  `SharedPreferences`:

   ```java
   SharedPreferences.Editor preferencesEditor = mPreferences.edit();
   
   ```

2. Hapus semua isi shared preferences:

   ```java
   preferencesEditor.clear();
   
   ```

   

3. Aplikasikan perubahan di atas:

   ```java
   preferencesEditor.apply();
   
   ```

Solusi untuk method `reset()`:

```java
public void reset(View view) {
   // Reset count
   mCount = 0;
   mShowCountTextView.setText(String.format("%s", mCount));

   // Reset color
   mColor = ContextCompat.getColor(this, R.color.default_background);
   mShowCountTextView.setBackgroundColor(mColor);

   // Clear preferences
   SharedPreferences.Editor preferencesEditor = mPreferences.edit();
   preferencesEditor.clear();
   preferencesEditor.apply();

}
```

Android Studio project: [HelloSharedPrefs](https://github.com/google-developer-training/android-fundamentals-apps-v2/tree/master/HelloSharedPrefs)



## Coding challenge

> **Note**: Semua coding challenge bersifat opsional dan tidak menjadi prasyarat untuk melanjutkan ke materi berikutnya.

**Challenge:** Modifikasi aplikasi HelloSharedPrefs sehingga tidak menyimpan state secara otomatis, melainkan diubah, di reset dan disimpan dari activity baru Tambahkan sebuah button bernama Settings untuk meluncurkan activity tersebut. Tambahkan button toggle dan spinner untuk memodifikasi preferences, lalu buttom  **Save** dan **Reset** untuk membersihkan preferences.

# Kesimpulan

- Kelas [`SharedPreferences`](https://developer.android.com/reference/android/content/SharedPreferences.html) memungkinkan aplikasi untuk menyimpan data primitif sederhana menggunakan pasangan key-value.

- Data yang disimpan dengan shared preferences tetap bertahan dalam aplikasi yang sama. 

- Untuk menyiimpan ke shared preferences, gunakan objek [`SharedPreferences.Editor`](https://developer.android.com/reference/android/content/SharedPreferences.Editor).

- Gunakan berbagai method "put" yang tersedia di objek  `SharedPreferences.Editor`, misalnya  [`putInt()`](https://developer.android.com/reference/android/content/SharedPreferences.Editor#putInt(java.lang.String,%20int)) atau [`putString()`](https://developer.android.com/reference/android/content/SharedPreferences.Editor#putString(java.lang.String,%20java.lang.String)), untuk menyimpan data ke dalam shared preferences dengan menggunakan key dan value.

- Gunakan berbagai method "get" yang tersedia di objek  `SharedPreferences`, misalnya [`getInt()`](https://developer.android.com/reference/android/content/SharedPreferences#getInt(java.lang.String,%20int)) atau [`getString()`](https://developer.android.com/reference/android/content/SharedPreferences#getString(java.lang.String,%20java.lang.String)), untuk mengambil data dari dalam shared preferences dengan sebuah key.

- Gunakan method [`clear()`](https://developer.android.com/reference/android/content/SharedPreferences.Editor#clear()) di objek `SharedPreferences.Editor` untuk menghapus data yang tersimpan di dalam preferences.

- Gunakan method [`apply()`](https://developer.android.com/reference/android/content/SharedPreferences.Editor#apply()) di objek `SharedPreferences.Editor` untuk menyimpan perubahan di dalam file preferences.

  

### Konsep terkait

Konsep terkait modul ini ada di [9.0: Data storage](https://google-developer-training.github.io/android-developer-fundamentals-course-concepts-v2/unit-4-saving-user-data/lesson-9-preferences-and-settings/9-0-c-data-storage/9-0-c-data-storage.html) dan [9.1: Shared preferences](https://google-developer-training.github.io/android-developer-fundamentals-course-concepts-v2/unit-4-saving-user-data/lesson-9-preferences-and-settings/9-1-c-shared-preferences/9-1-c-shared-preferences.html).

### Lebih lanjut

Dokumentasi Android developer:

- [Data and file storage overview](https://developer.android.com/guide/topics/data/data-storage)
- [Save key-value data](https://developer.android.com/training/data-storage/shared-preferences)
- [`SharedPreferences`](https://developer.android.com/reference/android/content/SharedPreferences.html)
- [`SharedPreferences.Editor`](https://developer.android.com/reference/android/content/SharedPreferences.Editor.html)

Stackoverflow:

- [How to use SharedPreferences in Android to store, fetch and edit values](http://stackoverflow.com/questions/3624280/how-to-use-sharedpreferences-in-android-to-store-fetch-and-edit-values)

- [onSavedInstanceState vs. SharedPreferences](http://stackoverflow.com/questions/5901482/onsavedinstancestate-vs-sharedpreferences)

  

## Pekerjaan rumah

Bagian ini menampilkan tugas yang mungkin bisa dikerjakan oleh peserta yang sedang mempelajari codelab ini sebagai bagian dari kelas berinstruktur. Instruktur bisa melakukan:

- Pemberian tugas jika diperlukan.
- Menjelaskan bagaimana peserta mengerjakan tugasnya.
- Menilai tugas.

Instruktur dapat menggunakan saran-saran seperlunya dan bebas untuk menggunakan tugas lain jika memang dirasa perlu. 

Jika kamu mempelajari codelab ini sendiri, silahkan gunakan tugas-tugas berikut untuk menguji pemahaman kamu. 

### **Buat dan jalankan sebuah aplikasi**

Buka aplikasi  [ScoreKeeper app](https://github.com/google-developer-training/android-fundamentals-apps-v2/tree/master/Scorekeeper) yang kamu buat di modul [Android fundamentals 5.1: Drawables, styles, and themes](https://codelabs.developers.google.com/codelabs/android-training-drawables-styles-and-themes).

1. Ganti saved instance state dengan shared preferences untuk setiap score.
2. Untuk menguji aplikasi tersebut, putar layar sehingga terjadi configuration change lalu membaca dan yang disimpan di saved preferences dan meng-update UI-nya.
3. Tutup aplikasi, lalu jalankan lagi untuk memastikan bahwa preferences di simpan. 
4. Tambahkan sebuah button **Reset** untuk mengubah skor kembali menjadi 0 dan mengosongkan isi shared preferences.





### **Jawab pertanyaan-pertanyaan berikut**

#### **Soal 1**

Di dalam method lifecycle apa kita menyimpan state aplikasi ke dalam shared preferences?

#### **Soal 2**

Di dalam method lifecycle apa kita mengembalikan state aplikasi?

#### **Soal 3**

Menurut kamu, kapan kasus yang bisa memanfaatkan kelebihan baik itu shared preferences maupun instance state?

### **Kirimkan aplikasi kamu untuk dinilai**

#### **Panduan untuk penilai**

Periksa apakah aplikasi sudah memiliki fitur-fitur berikut:

- Aplikasi tetap menampilkan skor yang benar saat layar diputar.
- Aplikasi tetap menampilkan skor terakhir saat ditutup dan dibuka ulang. 
- Aplikasi menyimpan skor saat ini ke shred preferences di dalam method `onPause()`. 
- Aplikasi me-restore shared preferences di dalam method `onCreate()`. 
- Aplikasi menampilkan bomtol **Reset** yang mengubah skor kembali menjadi 0. 

Pastikan implementasi method onClick handler untuk tombol **Reset** melakukan hal berikut:

- Me-reset skor kembali menjadi 0.
- Mengupdate kedua text view.
- Menghapus isi shared preferences. 

## Codelab berikutnya

Untuk melihat codelab berikutnya di kelas Android Developer Fundamentals (V2), lihat [Codelabs for Android Developer Fundamentals (V2)](https://developer.android.com/courses/fundamentals-training/toc-v2).

Untuk ikhtisar kelas, termasuk tautan untuk bab konseptual, contoh aplikasi, dan slide, lihat [Android Developer Fundamentals (Version 2)](https://developer.android.com/courses/fundamentals-training/overview-v2).