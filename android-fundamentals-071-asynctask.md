---
summary: In this codelab, you’ll build a Progressive Web App, which loads quickly, even on flaky networks, has an icon on the homescreen, and loads as a top-level, full screen experience.
id: android-fundamentals-asynctask
category: Android
environment: android,background-task,asynctask,thread
status: published
---





# Android Fundamentals 07.1: AsyncTask

## Selamat Datang

Codelab ini merupakan bagian dari [Unit 3: Working in the background](https://developer.android.com/courses/fundamentals-training/toc-v2#unit_3_working_in_the_background) dalam kelas belajar Android Developer Fundamentals (Versi 2). Kamu akan mendapatkan pemahaman yang lebih lengkap jika mengikuti codelab ini secara berurutan:

- Untuk daftar codelabs di kelas ini, silahkan lihat [Codelabs Android Developer Fundamentals (V2)](https://developer.android.com/courses/fundamentals-training/toc-v2).
- Informasi lebih lengkap tentang kelas, termasuk tautan ke bab-bab konsep, contoh aplikasi, dan slide, silahkan lihat halaman [Android Developer Fundamentals (Versi 2)](https://developer.android.com/courses/fundamentals-training/overview-v2).

> **Catatan**: Istilah “codelab” dan “praktik” akan sering dipakai secara bergantian tanpa mengubah makna. 

## Pendahuluan

***Thread*** adalah sebuah jalur eksekusi independen dari masing-masing aplikasi yang berjalan. Saat sebuah program Android diluncurkan, sitem akan membuat sebuah *main thread*, yang juga disebut dengan *UI thread*. UI thread ini adalah jalur dimana kita bisa berinteraksi dengan komponen-komponen UI Android. 

Seringkali sebuah aplikasi perlu melakukan tugas-tugas yang berat dan lama seperti mengunduh file, melakukan query ke database, memutar media, atau melakukan analisis komputasi yang kompleks. Tugas-tugas berat semacam ini bisa mem-blok UI thread sehingga aplikasi pun tidak dapat merespon masukan user atau menampilkan sesuatu ke layar. User yang frustasi mungkin akan meng-uninstall aplikasi kita. 

Agar pengalaman user (*user experience/*UX) berjalan dengan mulus, Android framework menyediakan sebuah kelas pembantu bernama [`AsyncTask`](http://developer.android.com/reference/android/os/AsyncTask.html), yang berfungsi untuk menggeksekusi tugas-tugas dibelakang layar. Dengan menggunakan `AsyncTask` untuk memindahkan pekerjaan berat di thread terpisah artinya thread UI akan tetap responsif. 

Karena thread terpisah tidak sinkron (sejalur) dengan thread asal, maka ia dinamakan dengan asynchronous thread (thread yang asinkron/tidak sinkron). Sebuah `AsyncTask` juga memiliki callback yang memungkinkan kita untuk menampilkan hasil perhitungan yang dilakukan di belakang layar kembali ke UI thread. 

Di modul ini kita akan belajar bagaimana membuat sebuah background task di aplikasi Android menggunakan `AsyncTask`. 

### Apa yang kamu seharusnya sudah pelajari

Untuk mengikuti modul ini, kamu harus sudah mengerti:

- Cara membuat `Activity`.
- Menambah sebuah `TextView` ke layout milik  `Activity`.
- Mengambil id `TextView` dan mengubah kontennya dari Java. 
- Menggunakan `Button` dan mengaplikasikan fungsi onClick.

### Apa yang akan kamu pelajari

- Bagaimana cara menambah komponen `AsyncTask` di aplikasi agar bisa menjalankan tugas di belakang layar..
- Memahami kelebihan dan kekurangan `AsyncTask` untuk menjalankan *background task*.

### Apa yang akan kamu lakukan

- Membuat aplikasi sederhana yang mengeksekusi sebuah *background task* menggunakan  `AsyncTask`.
- Menjalankan aplikasi dan melihat apa yang terjadi saat perangkat diputar.
- Mengimplementasi *instance state activity* untuk menyimpan state sebuah pesan  `TextView`.Selamat Datang

## Gambaran aplikasi

Kita akan membuat sebuah aplikasi yang memiliki satu TextView dan satu Button. Saat user mengklik Button tersebut, aplikasi akan melakuan “sleep” dalam jangka waktu acak, lalu menampilkan sebuah pesan di TextView saat sudah kembali aktif. Berikut gambaran aplikasi yang akan dibuat:

![img](https://lh6.googleusercontent.com/g3dw4jjAVHOybWWZ-Ss93f9bBXGwOczYZSMDyeAV9dzC2GUY7ZsOmMZeU8mtuqULSabltxKk46cMlDeY0Abq22eP6PDQEjwwoZrR507vw6hEtCTYZUqvp2f1rjjgaHIKoc8kxCTC)

## Tahap 1: Menyiapkan project SimpleAsyncTask 

Antarmuka SimpleAsyncTask memiliki sebuah Button yang akan memulai sebuah AsyncTask, lalu TextView akan menampilkan status aplikasi.

### 1.1 Membuat project dan layout

1. Buat sebuah project baru bernama SimpleAsyncTask menggunakan template Empty Activity. Ikuti saja pengaturan bawaan untuk opsi lainnya.
2. Buka file layout `activity_main.xml`. Klik tab  Text.
3. Tambahkan atribut layout_margin ke top-level ConstraintLayout:
	`android:layout_margin="16dp"`

4. Buat atau modifikasi atribut TextView Hello World! Untuk memiliki atribut-atribut berikut. Jangan lupa untuk meng-extract string ke dalam file resources. 

    | **Attribute**    | **Value**                   |
    | ---------------- | --------------------------- |
    | android:id       | "@+id/textView1”            |
    | android:text     | "I am ready to start work!" |
    | android:textSize | "24sp"                      |

5. Hapus atribut  app:layout_constraintRight_toRightOf dan app:layout_constraintTop_toTopOf.
6. Tambahkan sebuah elemen Button tepat di bawah TextView, lalu berikan atribut berikut. Ekstrak teks Button ke sebuah resource string.

    | **Attribute**                        | **Value**        |
    | ------------------------------------ | ---------------- |
    | android:id                           | "@+id/button"    |
    | android:layout_width                 | "wrap_content"   |
    | android:layout_height                | "wrap_content"   |
    | android:text                         | "Start Task"     |
    | android:layout_marginTop             | "24dp"           |
    | android:onClick                      | "startTask"      |
    | app:layout_constraintStart_toStartOf | "parent"         |
    | app:layout_constraintTop_toBottomOf  | "@+id/textView1" |

1. Atribut onClick di button akan berwarna kuning karena method `startTask()` belum dibuat di `MainActivity`. Arahkan kursor ke teks yang berwarna, tekan Alt + Enter (Option + Enter di Mac) lalu pilih Create 'startTask(View) in 'MainActivity' untuk membuat method tersebut di `MainActivity`.

Berikut solusi untuk file activity_main.xml:

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
   xmlns:android="http://schemas.android.com/apk/res/android"
   xmlns:app="http://schemas.android.com/apk/res-auto"
   xmlns:tools="http://schemas.android.com/tools"
   android:layout_width="match_parent"
   android:layout_height="match_parent"
   android:layout_margin="16dp"
   tools:context=".MainActivity">

   <TextView
       android:id="@+id/textView1"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:text="@string/ready_to_start"
       android:textSize="24sp"
       app:layout_constraintStart_toStartOf="parent"
       app:layout_constraintTop_toTopOf="parent"/>
  
   <Button
       android:id="@+id/button"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:layout_marginTop="24dp"
       android:onClick="startTask"
       android:text="@string/start_task"
       app:layout_constraintStart_toStartOf="parent"
       app:layout_constraintTop_toBottomOf="@+id/textView1"/>

</android.support.constraint.ConstraintLayout>
```

## Tahap 2: Membuat subclass AsyncTask 

`AsyncTask` adalah sebuah kelas *abstract*, artinya kita harus membuat sebuah *subclass* jika ingin memanfaatkannya. Di contoh ini, `AsyncTask` akan melakukan sebuah tugas sederhana di belakang layar: Ia akan berhenti bekerja selama beberapa detik secara acak. Dalam aplikasi sungguhan, *background task* bisa jadi pengambilan data dari database, melakukan koneksi ke internet, hingga mengkalkulasi langkah yang perlu dijalankan untuk mengalahkan juara Go (ingat AlphaGO). 

Sebuah subclass `AsyncTask` akan memiliki beberapa method berikut untuk melaksanakan tugasnya:

- [onPreExecute()](https://developer.android.com/reference/android/os/AsyncTask.html#onPreExecute()): Method ini berjalan di UI thread dan dipakai untuk menyiapkan proses misalnya dengan menampilkan progress bar.

- [doInBackground()](https://developer.android.com/reference/android/os/AsyncTask.html#doInBackground(Params...)): Method ini adalah tempat dimana kita menuliskan kode yang menjalankan tugasnya di thread terpisah. 

- [onProgressUpdate()](https://developer.android.com/reference/android/os/AsyncTask.html#onProgressUpdate(Progress...)): Method ini dipanggil di UI thread dan bisa dimanfaatkan untuk memberikan update proses di UI misalnya dengan menaikkan persentasi  progress bar

- [onPostExecute()](https://developer.android.com/reference/android/os/AsyncTask.html#onPostExecute(Result)): Method ini juga akan dijalankan di UI thread, dipakai untuk mengupdate hasil di UI setelah AsyncTask selesai menunaikan tugasnya.

  

![img](https://lh3.googleusercontent.com/_ZE07u911PMSGimT-rahhuk7c3T7oSi0qgdxO2a6FPpp4x1QF_o88XV22H3xKC5kiqlt9LIKpVMCLpEM2aQhD9q8qkTlFrtN80TFE-N1Ca1XygdA-gPH4L62f7eXnASAH-b8eIzM)

> **Catatan**:  Sebuah background atau worker thread adalah semua thread yang bukan  main atau UI thread.

Saat membuat subclass sebuah `AsyncTask`, anda mungkin perlu mengirimkan informasi tentang tugas apa yang akan dilakukan, bagaimana melaporkan progress, dan apa yang ingin dilakukan pada hasil yang didapat. Saat membuat subclass sebuah `AsyncTask`, kita bisa mengaturnya menggunakan tiga parameter berikut:

- **Params**: Tipe data yang dikirimkan saat pengeksekusian method `doInBackground()`.
- **Progress:**  Tipe data untuk melaporkan sejauh mana tugas dilaksanakan dengan mempublikasikannya melalui method `onProgressUpdated()`.
- **Result**: Tipe data dari hasil tugas yang telah dilakukan untuk dikirim kembali ke UI thread melalui method  `onPostExecute()`

Contoh, sebuah kelas AsyncTask bernama MyAsyncTask dapat memiliki tiga jenis parameter ini saat dideklarasikan:

- Sebuah String sebagai parameter untuk `doInBackground()`, misalnya untuk dipakai melakukan query .

- Sebuah Integer untuk memproses `onProgressUpdate()`, mewakili persentase penyelesaian tugas.

- Sebuah Bitmap sebagai result yang dipublikasikan oleh method  `onPostExecute()`, menandakan hasil query.

  ```java
  public class MyAsyncTask 
     extends AsyncTask <String, Integer, Bitmap>{}
  ```

Di tahap ini kita akan membuat sebuah subclass `AsyncTask` untuk menentukan tugas yang berjalan di thread yang berbeda dari UI thread.

### 2.1 Membuat subclass AsyncTask

Di aplikasi ini, subclass `AsyncTask` yang kita buat tidak memerlukan *query parameter* juga tidak akan mempublikasikan *progress*. Kita hanya akan memanfaatkan method `doInBackground()` dan `onPostExecute()`.

1. Buat sebuah kelas Java bernama `SimpleAsyncTask` yang meng-extends `AsyncTask` dan menerima tiga parameter generic.

   Gunakan Void sebagai query parameter karena  `AsyncTask` tidak memerlukan input apapun. Gunakan Void juga untuk tipe progress karena kita tidak akan melaporkan progress apa-apa. Gunakan String sebagai tipe nilai kembaliannya,  karena kita akan mengupdate `TextView` dengan sebuah String ketika tugas `AsyncTask` sudah selesai di eksekusi. 

   ```java
   public class SimpleAsyncTask extends AsyncTask <Void, Void, String>{}
   ```

   > **Catatan**: Deklarasi class akan berwarna merah karena method yang wajib ada, doInBackground() belum diimplement. 

2. Di bagian atas kelas ini, buat variabel bernama `mTextView` dengan tipe data `WeakReference<TextView>`:

   ```java
   private WeakReference<TextView> mTextView;
   ```

3. Buat sebuah konstruktor untuk yang menerima TextView sebagai parameternya untuk membuat sebuah weak reference ke TextView tadi:

   ```java
   SimpleAsyncTask(TextView tv) {
   	mTextView = new WeakReference<>(tv);
   }
   ```

`AsyncTask` perlu mengubah isi `TextView` yang ada di dalam `Activity` setelah tugasnya selesai (dari dalam method `onPostExecute()`). Oleh karena itu konstruktor kelas ini akan memerlukan referensi ke `TextView` yang ingin di ubah isinya. 

Apa fungsi weak reference (kelas  [WeakReference](https://developer.android.com/reference/java/lang/ref/WeakReference))? Jika kita mengirimkan sebuah `TextView` ke konstruktor `AsyncTask`, lalu menyimpannya ke sebuah variabel, maka artinya Activity tadi tidak akan bisa di *garbage collected*-kan (pembersihan memori dari proses yang sudah tidak terpakai) yang mengakibatkan terjadinya kebocoran memori (*memory leak*), bahkan jika Activity di destroy misalnya saat terjadi perubahan konfigurasi (configuration change, contohnya di rotate). Android Studio akan memberikan peringatan jika kita bakal mengakibatkan sebuah memory leak. 

*Weak reference* yang kita gunakan tadi mencegah `memory leak` dengan memungkinkan objek yang alamat memorinya dipegang di tempat lain untuk bisa di `garbage collected` saat dibutuhkan. 

### 2.2 Impelementasi doInBackground()

Method `doInBackground()` diperlukan oleh subclass `AsyncTask`.

1. Arahkan kursor ke deklarasi kelas, lalu tekan Alt + Enter (Option + Enter di Mac) dan pilih  Implement methods. Pilih doInBackground() lalu klik OK. Template method berikut akan ditambahkan ke dalam kelas kita:

   ```java
   @Override
   protected String doInBackground(Void... voids) {
      return null;
   }
   ```

   

2. Tambahkan kode untuk membuat angka Integer acak antara 0 sampai 10. Angka ini menentukan berapa lama tugas di “pause” di belakang layar dalam milidetik. Karena angka 0-10 akan terlalu cepat, kalikan angka tersebut dengan 200 sehingga agak lama. 

   ```java
   Random r = new Random();
   int n = r.nextInt(11);
   
   int s = n * 200;
   
   ```

   

3. Buat sebuah blok try/catch lalu sleep-kan thread ini.

   ```java
   try {
      Thread.sleep(s);
   } catch (InterruptedException e) {
      e.printStackTrace();
   }
   ```

   

4. Ganti perintah return yang sudah ada untuk mengirimkan String "Awake at last after sleeping for *xx* milliseconds", dimana *xx* adalah angka dalam milidetik berapa lama thread tersebut terhenti.

   ```java
   return "Awake at last after sleeping for " + s + " milliseconds!";
   ```

Isi method `doInBackground()` yang lengkap akan terlihat sebagai berikut:

```java
@Override
protected String doInBackground(Void... voids) {

   // Generate a random number between 0 and 10
   Random r = new Random();
   int n = r.nextInt(11);

   // Make the task take long enough that we have
   // time to rotate the phone while it is running
   int s = n * 200;

   // Sleep for the random amount of time
   try {
       Thread.sleep(s);
   } catch (InterruptedException e) {
       e.printStackTrace();
   }

   // Return a String result
   return "Awake at last after sleeping for " + s + " milliseconds!";
}

```

### 2.3 Implement onPostExecute()

Saat method `doInBackground()` selesai dieksekusi, nilai return akan dikirimkan secara otomatis ke callback `onPostExecute()`.

1. Implementasi method `onPostExecute()` untuk menerima sebuah argumen bertipe  String dan memberikan string tersebut ke `TextView`:

   ```java
   protected void onPostExecute(String result) {
      mTextView.get().setText(result);
   }
   ```

Parameter string di method ini adalah tipe data yang kita tuliskan di parameter ketiga kelas `AsyncTask` juga tipe data yang di-return oleh method `doInBackground()`.

Karena `mTextView` adalah sebuah weak reference, kita bisa melakukan deference menggunakan method `get()` untuk mendapatkan objek `TextView`-nya lalu memanggil `setText()`. 

> **Catatan**: Kita bisa meng-update UI di  onPostExecute() karena method ini berada di main thread. Kita tidak bisa mengupdate TextView dengan string baru tersebut di method doInBackground(), karena method tersebut berjalan di thread yang berbeda. 

## Tahap 3: Implementasi terakhir 

### 3.1 Implementasi method yang mengeksekusi AsyncTask

Aplikasi kita sekarang memiliki sebuah kelas `AsyncTask` yang melakukan tugasnya di belakang layar. Kita bisa mengimplementasi tombol “Start Task” untuk memulai proses di belakang layar tersebut.

1. Di dalam file `MainActivity.java`, buat sebuah variabel untuk menyimpan objek `TextView`.

   ```java
   private TextView mTextView;
   ```

   

2. Di dalam method `onCreate()`, inisialisasi `mTextView` ke komponen `TextView` yang ada di layout.

   ```java
   mTextView = findViewById(R.id.textView1);
   ```

   

3. Di dalam method `startTask()`, update isi `TextView` untuk menampilkan teks "Napping...". Ekstrak pesan tersebut ke sebuah string resource.

   ```java
   mTextView.setText(R.string.napping);
   ```

   

4. Buat instance dari kelas  `SimpleAsyncTask`, kirimkan  `TextView` `mTextView` ke dalam konstruktor. Panggil  `execute()` di instance `SimpleAsyncTask`.

   ```java
   new SimpleAsyncTask(mTextView).execute();
   ```

> **Catatan**: Method execute() adalah bagian dimana kita mengirimkan parameter yang dipisahkan oleh koma ke dalam  doInBackground() oleh sistem. Karena AsyncTask ini tidak menerima parameter, maka bisa kita kosongkan.

Kode  `MainActivity` yang lengkap:

```java
package com.example.android.simpleasynctask;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.TextView;

/**
* The SimpleAsyncTask app contains a button that launches an AsyncTask
* which sleeps in the asynchronous thread for a random amount of time.
*/
public class MainActivity extends AppCompatActivity {

   // The TextView where we will show results
   private TextView mTextView;

  @Override
   protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.activity_main);

       mTextView = findViewById(R.id.textView1);
   }

   public void startTask(View view) {
       // Put a message in the text view
       mTextView.setText(R.string.napping);

       // Start the AsyncTask.
       new SimpleAsyncTask(mTextView).execute();
   }
}
```

### 3.2 Implementasi onSaveInstanceState()

1. Jalankan aplikasinya lalu klik tombol `Start Task`. Berapa lama aplikasi “mati”?

2. Klik tombol `Start Task` sekali lagi, dan saat aplikasi sedang “mati” coba putar layar. Jika background task selesai sebelum memutar layar, coba lagi. 

   > **Catatan**: Kamu akan menyadari bahwa saat layar diputar, TextView akan me-reset isinya ke konten awal sehingga AsyncTask terlihat tidak mengupdate isi  TextView tersebut. 

   Ada beberapa hal yang terjadi di sini:

   - Saat layar di putar, sistem me-restart aplikasi dengan memanggil `onDestroy()` lalu `onCreate()`. `AsyncTask` akan terus berjalan meskipun `Activity` di destroy, tapi kehilangan kemampuan untuk mengubah UI si `Activity`. Dia tidak akan pernah bisa mengubah `TextView` yang dikirimkan ke dalamnya karena activity dimana `TextView` itu hidup telah di destroy. 
   - Setelah activity di destroy,  maka  `AsyncTask` akan terus berjalan untuk menyelesaikan tugasnya di belakang layar, memakan sumberdaya sistem. Ketika sistem kehabisan sumberdaya, maka AsyncTask akan gagal melaksanakan tugasnya.
   - Meskipun tanpa `AsyncTask`, rotasi layar tetap akan mereset semua elemen UI ke status awalnya, yang mana untuk `TextView` kita akan menampilkan string yang terisi di layout. 

   Karena alasan ini, `AsyncTask` tidak cocok dipakai untuk tugas yang bisa terganggu saat `Activity` di destroy. Dalam kasus dimana hal ini krusial kita lebih disarankan untuk menggunakan `AsyncTaskLoader` yang akan kita pelajari di modul berikutnya. 

   Untuk mencegah `TextView` kembali ke string awal, kita perlu menyimpan nilai yang sedang ditampilkan. Untuk itu kita sekarang akan mengimplementasi method `onSaveInstanceState()` yang berfungsi untuk menyimpan konten TextView saat activity di destroy karena configuration change misalnya saat perputaran layar. 

3. Di bagian atas kelas activity, tambahkan sebuah variabel konstan untuk menyimpan key teks yang berada di dalam `TextView` ke dalam bundle. 

   ```java
   private static final String TEXT_STATE = "currentText";
   ```

   

4. Override method `onSaveInstanceState()` di `MainActivity` untuk menyimpan teks yang berada di dalam  `TextView` saat activity di destroy:

   ```java
   @Override
       protected void onSaveInstanceState(Bundle outState) {
           super.onSaveInstanceState(outState);
           // Save the state of the TextView
           outState.putString(TEXT_STATE, 
               mTextView.getText().toString());
       }
   }
   
   ```

   

5. Di dalam onCreate(), ambil isi TextView dari dalam bundle saat activity kembali di tampilkan. 

   ```java
   // Restore TextView if there is a savedInstanceState
   if(savedInstanceState!=null){
     mTextView.setText(savedInstanceState.getString(TEXT_STATE));
   
   ```

Kode lengkap untuk `MainActivity`:

```java
package android.example.com.simpleasynctask;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.TextView;

/**
 * The SimpleAsyncTask app contains a button that launches an AsyncTask
 * which sleeps in the asynchronous thread for a random amount of time.
 */
public class MainActivity extends AppCompatActivity {

    //Key for saving the state of the TextView
    private static final String TEXT_STATE = "currentText";

    // The TextView where we will show results
    private TextView mTextView = null;

    /**
     * Initializes the activity.
     * @param savedInstanceState The current state data
     */
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //  Initialize mTextView
        mTextView = (TextView) findViewById(R.id.textView1);

        // Restore TextView if there is a savedInstanceState
        if(savedInstanceState!=null){
           mTextView.setText(savedInstanceState.getString(TEXT_STATE));
        }
    }

    /**`
     * Handles the onCLick for the "Start Task" button. Launches the AsyncTask
     * which performs work off of the UI thread.
     *
     * @param view The view (Button) that was clicked.
     */
    public void startTask (View view) {
        // Put a message in the text view
        mTextView.setText(R.string.napping);

        // Start the AsyncTask.
        // The AsyncTask has a callback that will update the text view.
        new SimpleAsyncTask(mTextView).execute();
    }


    /**
     * Saves the contents of the TextView to restore on configuration change.
     * @param outState The bundle in which the state of the activity is saved
     * when it is spontaneously destroyed.
     */
    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        // Save the state of the TextView
        outState.putString(TEXT_STATE, mTextView.getText().toString());
    }
}

```

Android Studio project: [SimpleAsyncTask](https://github.com/google-developer-training/android-fundamentals-apps-v2/tree/master/SimpleAsyncTask).

## Coding challenge

> **Note**: Semua coding challenge bersifat opsional dan tidak menjadi prasyarat untuk melanjutkan ke materi berikutnya.

Challenge: Kelas `AsyncTask` menyediakan method lain yang berguna yaitu `onProgressUpdate()` yang memungkinkan kita mengupdate UI saat  `AsyncTask` sedang berjalan. Gunakan method ini untuk mengupdate UI dengan menampilkan waktu sleep time. Lihat [AsyncTask documentation](https://developer.android.com/reference/android/os/AsyncTask) untuk mempelajari bagaimana cara kerja `onProgressUpdate()`. Ingat bahwa di dalam definisi kelas `AsyncTask` yang dibuat kita perlu menentukan tipe data yang ingin digunakan di dalam `onProgressUpdate()`.

# Kesimpulan

- Sebuah [AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) adalah abstract Java class yang memungkinkan pemrosesan data intensif yang berlangsung di thread terpisah.
- Untuk menggunakan `AsyncTask` kita harus membuat sebuah subclass.
- Hindari pengeksekusian tugas berat di UI thread karena bisa mengganggu kelancaran UI aplikasi. 
- Semua kode yang tidak berhubungan dengan menampilkan UI atau merespon terhadap masukan pengguna hendaklah dipindah dari UI thread ke thread lain yang terpisah. 
- AsyncTask memiliki empat method utama yakni: [onPreExecute()](https://developer.android.com/reference/android/os/AsyncTask.html#onPreExecute()), [doInBackground()](https://developer.android.com/reference/android/os/AsyncTask.html#doInBackground(Params...)), [onPostExecute()](https://developer.android.com/reference/android/os/AsyncTask.html#onPostExecute(Result)) dan [onProgressUpdate()](https://developer.android.com/reference/android/os/AsyncTask.html#onProgressUpdate(Progress...)).
- Method `doInBackground()` merupakan satu-satunya method yang bekerja di sebuah worker thread. Jangan panggil UI dari dalam  `doInBackground()`.
- Method lain `AsyncTask` berjalan di UI thread sehingga memungkinkan kita untuk memanggil komponen UI.
- Memutar layar Android akan men-destroy dan membuat ulang sebuah `Activity`. Proses ini bisa memutuskan hubungan UI dari background thread di dalam  `AsyncTask`, yang akan tetap berjalan. 

### Konsep terkait

Konsep terkait modul ini ada di [7.1: AsyncTask dan AsyncTaskLoader](https://google-developer-training.github.io/android-developer-fundamentals-course-concepts-v2/unit-3-working-in-the-background/lesson-7-background-tasks/7-1-c-asynctask-and-asynctaskloader/7-1-c-asynctask-and-asynctaskloader.html).

### Lebih lanjut

Dokumentasi Android developer:

- [Processes and threads overview](http://developer.android.com/guide/components/processes-and-threads.html)
- [AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html)

Sumber lain:

- [Android Threading & Background Tasks](https://realm.io/news/android-threading-background-tasks/)

Video:

- [Threading Performance 101](https://www.youtube.com/watch?v=qk5F6Bxqhr4)
- [Good AsyncTask Hunting](https://www.youtube.com/watch?v=jtlRNNhane0)

## Pekerjaan rumah

Bagian ini menampilkan tugas yang mungkin bisa dikerjakan oleh peserta yang sedang mempelajari codelab ini sebagai bagian dari kelas berinstruktur. Instruktur bisa melakukan:

- Pemberian tugas jika diperlukan.
- Menjelaskan bagaimana peserta mengerjakan tugasnya.
- Menilai tugas.

Instruktur dapat menggunakan saran-saran seperlunya dan bebas untuk menggunakan tugas lain jika memang dirasa perlu. 

Jika kamu mempelajari codelab ini sendiri, silahkan gunakan tugas-tugas berikut untuk menguji pemahaman kamu. 

### **Buat dan jalankan sebuah aplikasi**

Buka aplikasi  [SimpleAsyncTask](https://github.com/google-developer-training/android-fundamentals-apps-v2/tree/master/SimpleAsyncTask). Tambahkan sebuah [`ProgressBar`](https://developer.android.com/reference/android/widget/ProgressBar.html) yang menampilkan persentase sleep tie yang sudah dilalui. Progress bar akan terisi mengikuti proses sleep thread  `AsyncTask` dari 0 sampai 100 (persen). 

**Petunjuk:** Pecah sleep time menjadi bagian-bagian yang lebih kecil.

Untuk bantuan, silahkan lihat, [`AsyncTask` reference](http://developer.android.com/reference/android/os/AsyncTask.html).

![img](https://codelabs.developers.google.com/codelabs/android-training-create-asynctask/img/968ccbd79491f28b.png)

### **Jawab pertanyaan-pertanyaan berikut**

#### **Soal 1**

Untuk `ProgressBar`:

1. Bagaimana mennetukan jangkauan nilai yang bisa ditampilkan oleh  `ProgressBar` ?
2. Bagaimana mengubah bagian progress bar yang terisi?

#### **Soal 2**

Jika sebuah `AsyncTask` dibuat dengan perintah berikut:

```
 private class DownloadFilesTask extends AsyncTask<URL, Integer, Long>
```

1. Apa tipe data yang dikirim ke  `doInBackground()` ke dalam AsyncTask?
2. Apa tipe data yang dikirim ke dalam callback yang melaporkan progress task yang sedang berjalan?
3. Apa tipe data yang dikirim ke dalam callback yang akan dipanggil setelah task selesai?

#### **Soal 3**

Untuk melaporkan progress pekerjaan yang dieksekusi oleh `AsyncTask`, callback method apa yang harus di *implement*, dan method apa yang perlu dipanggil  dari dalam subclass `AsyncTask` yang dibuat?

- Implement `publishProgress()`. 
  Call `publishProgress()`.
- Implement `publishProgress()`. 
  Call `onProgressUpdate()`.
- Implement `onProgressUpdate()`. 
  Call `publishProgress()`.
- Implement `onProgressUpdate()`. 
  Call `onProgressUpdate()`.

### **Kirimkan aplikasi kamu untuk dinilai**

#### **Panduan untuk penilai**

Periksa apakah aplikasi sudah memiliki fitur-fitur berikut:

- Layout memiliki sebuah  `ProgressBar` dengan atribut-atribut yang sesuai untuk menentukan jangkauan nilai persentase. 
-  `AsyncTask` membagi-bagi total sleep time menjadi beberapa bagian untuk mengirimkannya sebagai update k eprogress bar.
-  `AsyncTask` memanggil method yang diperlukan dan mengimplement callback yang diperlukan untuk meng-update progress bar.
-  `AsyncTask` perlu tahu view mana yang akan di update. Bergantung jenis `AsyncTask`, apakah ia dibuat sebagai inner class atau bukan, view yang dipanggil bisa dikirim sebagai constructor AsyncTask atau didefinisikan sebagai member variabel si `Activity`.

## Codelab berikutnya

Untuk melihat codelab berikutnya di kelas Android Developer Fundamentals (V2), lihat [Codelabs for Android Developer Fundamentals (V2)](https://developer.android.com/courses/fundamentals-training/toc-v2).

Untuk ikhtisar kelas, termasuk tautan untuk bab konseptual, contoh aplikasi, dan slide, lihat [Android Developer Fundamentals (Version 2)](https://developer.android.com/courses/fundamentals-training/overview-v2).

