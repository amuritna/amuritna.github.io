---
title: "Bermain dengan HTTP APIs menggunakan Google Apps Script"
date: 2024-11-21
draft: false
categories: [tutorial]
tags: [javascript]
description: API publik + manipulasi DOM dengan JavaScript + *Fetch API* untuk *spreadsheet* hasil Google Forms.
---
---

*Tulisan ini adalah hasil gabungan beberapa tutorial yang saya tulis selama jabatan saya sebagai Ketua Divisi Komputer HME ITB. Tulisan-tulisan tersebut telah diedit ulang dan dipost kembali untuk keperluan pengarsipan.*

---

Sejauh ini, yang terjadi saat browser [nge-*render*](https://en.wikipedia.org/wiki/Browser_engine) webpage adalah proses interpretasi (HTML dan CSS) dan [kompilasi](https://en.wikipedia.org/wiki/JavaScript_engine) (JavaScript) terhadap kode yang diberikan di `.html`, `.css`, dan `.js`. Informasi yang ditampilkan di webpage sepenuhnya berdasarkan apa yang telah ditulis di kode - jika ingin dilakukan perubahan, kode harus diubah atau ditulis ulang.

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/static-web.png?raw=true)

Bagaimana jika kita ingin menampilkan suatu informasi yang berubah-ubah tiap hari — contohnya, suhu dan cuaca di suatu tempat? Bagaimana jika kita ingin mengubah-ubah konten, mungkin bahkan secara otomatis, tanpa mengubah *source code* langsung? Ini adalah faktor yang membedakan *static websites* dan *dynamic websites* (website statis vs dinamis). Konten dari website statis hanya berubah saat *source code* diubah, sedangkan konten website dinamis dapat berinteraksi dengan hal-hal seperti *database* dan malah bisa jadi menampilkan hal yang berbeda untuk tiap user.

Untuk merancang *dynamic websites*, kita dapat memulai dengan konsep "*front-end*" versus "*back-end*" serta Web APIs. Modul ini lalu menjelaskkan suatu mekanisme untuk mempraktikkan dan melakukan *prototyping* dengan cepat menggunakan Google Sheets and Google Apps Script. 

---

[Bagian 1: JavaScript dan HTML](#bagian-1-javascript-dan-html)

[Bagian 2: *Review* Variabel dan Fungsi JavaScript](#bagian-2-review-variabel-dan-fungsi-javascript)

[Bagian 3: *Front-End* dan *Back-End*](#bagian-3-front-end-dan-back-end)

[Bagian 4: *Public APIs*](#bagian-4-public-apis)

[Bagian 5: Merancang *Fetch API* menggunakan Google Apps Script](#bagian-5-merancang-fetch-api-menggunakan-google-apps-script)

---

## Bagian 1: JavaScript dan HTML

Dalam *web development*, ada tiga alat paling mendasar yang dapat digunakan: HTML, CSS, dan JavaScript untuk konten, penampilan, dan interaksi secara berturut-turut. Semua *modern browser* tidak hanya dapat menjalankan ketiga hal ini dengan cukup baik, tetapi juga sudah dilengkapi berbagai [*developer tools*](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/Tools_and_setup/What_are_browser_developer_tools). Oleh karena itu, untuk mencoba-coba sesuatu dengan cepat dan tanpa mesti melakukan *software installation* atau *set-up* tambahan, membuat *webpage* sederhana dengan HTML/CSS/JavaScript dasar (*vanilla*) dapat menjadi opsi.

Salah satu fitur JavaScript yang dimanfaatkan untuk tutorial ini adalah [mengubah *Document Object Model*](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Client-side_web_APIs/Manipulating_documents) (*DOM manipulation*). Salah satu *HTML method* yang dapat digunakan adalah `getElementById()`. Pada contoh di bawah, JavaScript "mencari" elemen HTML (dengan `id="demo"`), lalu mengubah konten elemen (`innerHTML`) menjadi "*Hello JavaScript*":

```javascript
document.getElementById("demo").innerHTML = "Hello JavaScript";
```

JavaScript juga dapat digunakan untuk berinteraksi dengan atribut ...

```html
<button onclick="document.getElementById('myImage').src='lampu_nyala.gif'">
  Nyalakan lampu
</button>

<img id="myImage" src="lampu_mati.gif" style="width:100px">

<button onclick="document.getElementById('myImage').src='lampu_mati.gif'">
  Matikan lampu
</button>
```

... dan *styling* (konfigurasi CSS) ...

```html
<p id="iniText">
  Iya betul ini text.
</p>
```

```javascript
document.getElementById("iniText").style.fontSize = "35px";
```

... misalnya, untuk menyembunyikan suatu elemen.

```javascript
document.getElementById("iniText").style.display = "none";
```

---

Untuk menyisipkan suatu *script* JavaScript ke dalam dokumen HTML, masukkan ke dalam tag `script` di `head` atau `body`. Jika menggunakan `<script>`, disarankan untuk menaruhnya di bagian paling bawah `body` untuk meningkatkan kecepatan *rendering*, karena proses *rendering* browser akan menjadi lebih lama apabila mesti menginterpretasi isi `script` terlebih dahulu. 

Secara umum, dokumen HTML akan berbentuk seperti

```html
<!DOCTYPE HTML>
<html>
  <head>
    <title>Title</title>
    ...
  </head>
  <body>
    ...
    ...

    <script>
      ...
    </script>
  </body>
</html>
```

Untuk menulis *script* di file terpisah, contohnya yang file-nya bernama `myScript.js`, ganti `<script> ... </script>` dengan:

```html
<script src="myScript.js"></script>
```

Untuk mencoba contoh `getElementById()` dari atas:

```html
<!DOCTYPE HTML>
<html>
  <head>
    <title>Hello, HTML!</title>
  </head>
  <body>
    <h1 id="demo">Hello, HTML!</h1>
    <p>Lorem ipsum ...<p>

    <h1>Goodbye!</h1>

    <script>
      document.getElementById("demo").innerHTML = "Hello JavaScript";
    </script>
  </body>
</html>
```

Untuk mencoba suatu *script* dengan cepat dan praktis, misalnya untuk *prototyping*, opsi lainnya adalah langsung menggunakan *Developer Tools*. Buka *Inspect Element* lalu cari tab *Console*. Masukkan kode yang ingin dijalankan.

***Warning***: [Salah satu bentuk penipuan adalah menyuruh seseorang untuk menjalankan kode *harmful* via *console* ini](https://en.wikipedia.org/wiki/Self-XSS), jadi jangan mudah percaya.

---

## Bagian 2: Review Variabel dan Fungsi JavaScript
Saat menulis program, kita menginstruksikan komputer tentang cara memproses informasi, seperti menampilkan teks di layar atau melakukan operasi perhitungan. Untuk mempermudah pengelolaan dan pengambilan data, variabel digunakan untuk menyimpan nilai atau informasi dalam program.

Dalam bahasa pemrograman JavaScript, ada tiga cara untuk mendeklarasikan variabel, yaitu dengan menggunakan kata kunci var, let, dan const. Pada versi ECMAScript 2015 (ES6), metode deklarasi variabel dengan `let` dan `const` diperkenalkan sebagai pengganti `var`, yang dianggap kontroversial dan dapat menyebabkan *bug*.

```javascript
let x;
x = 6;
const y = 100;
y = 200; // error
```

Variabel yang dideklarasikan menggunakan `var` selalu memiliki *scope* global.
Variabel yang dideklarasikan dengan `var` TIDAK bisa memiliki *block scope*, contohnya:

```javascript
function myFunction(a, b) {
  var iniVar = 1;
  let iniLet = 2;
  const iniConst = 3;

  return a * b;
}

const x = iniVar; // var bisa diakses walau diluar block function, yg laen kagak
```

---

Selanjutnya, kita dapat menggunakan fungsi atau *functions* (sama seperti di Python dan C). Kita bisa memanggil mereka sesuai keperluan dan memanfaatkan mereka untuk menulis kode yang lebih *compact*, sehingga programnya menjadi lebih kecil (dan bisa jadi lebih cepat.

Fungsi JavaScript dibuat dengan menggunakan keyword `function` yang diikuti nama dan tanda kurung `()`. Tanda kurung ini dapat berisi nama parameter yang dipisahkan oleh tanda koma: `(parameter1, parameter2, ...)`

Kode yang akan dieksekusi oleh fungsi ditempatkan di dalam *curly brackets*: `{}`

Berikut adalah deklarasi fungsi JavaScript secara umum:

```javascript
function name(parameter1, parameter2, parameter3) {
  // code to be executed
}
```

Fungsi dapat di-*trigger* oleh *event* (seperti nge-*hover* *thumbnail*), dipanggil oleh *script*, atau diinisiasi secara otomatis dengan memanggil dirinya sendiri.

Ketika ditemukan pernyataan `return` di JavaScript, eksekusi fungsi dihentikan. Nilai yang dispesifikan pada pernyataan `return` dikembalikan ke "*caller*".

```javascript
// Fungsi dipanggil, nilai return bakal berakhir di x
let x = myFunction(4, 3);
document.getElementById("iniText").innerHTML = x;

function myFunction(a, b) {
// Fungsi mengembalikan hasil perkalian a dan b
  return a * b;
}

```

---

## Bagian 3: *Front-End* dan *Back-End*

Dalam konteks pengembangan website — serta dalam konteks lowongan pekerjaan *web developer* — ada dua istilah yang sering muncul, yaitu "[*front-end*](https://en.wikipedia.org/wiki/Front-end_web_development)" dan "*back-end*". Kedua istilah ini digunakan untuk "membagi" proses yang terjadi di balik layar menjadi dua.

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/fe-vs-be.png?raw=true)

Singkatnya, *front-end* adalah aspek-aspek yang dapat dilihat oleh dan secara langsung berinteraksi dengan user ([*client-side*](https://en.wikipedia.org/wiki/Client-side)), seperti HTML, CSS, dan JavaScript — yang telah kita pelajari sedikit demi sedikit sejauh ini. Sedangkan *back-end* adalah infrastruktur "sisi server" ([*server-side*](https://en.wikipedia.org/wiki/Server-side)) yang mengurus *database*. 

Komponen "*front-end*" dan "*back-end*" ini dihubungkan dengan satu sama lain menggunakan perantara *Application Programming Interface* — biasa disingkat menjadi [APIs](https://en.wikipedia.org/wiki/API). Saat website Edunex meminta, ke server, informasi daftar "*Courses*" yang sedang diikuti, sisi *front-end* membuat *request* ke *back-end* menggunakan API. *Back-end* lalu merespons dengan mengirim kembali informasi yang diminta, lagi-lagi menggunakan API. 

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/request-resp.png?raw=true)

Pengiriman *request* dan pembacaan *response* API ini dilakukan oleh browser dan dapat dipantau prosesnya menggunakan *Developer Tools*.

Di Mozilla Firefox atau Google Chrome, buka "*Developer Tools*" lalu pilih opsi "*Network*" untuk membuka *Network Monitor*. Untuk mendapatkan hasil yang seperti di contoh berikut (di Firefox), telah diklik salah satu *request*. Sekilas, *request* ini meminta informasi mengenai *courses* menggunakan API Edunex.

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/edunex-get-courses.png?raw=true)

Perhatikan bahwa *Network Monitor* dari Firefox ini melabeli *method* dari *request* ini sebagai GET. Ada dua metode fundamental dalam dunia *web APIs*, yaitu [GET](https://en.wikipedia.org/wiki/HTTP#Request_methods) (meminta informasi) dan [POST](https://en.wikipedia.org/wiki/POST_(HTTP)) (mengirim informasi). Untuk melihat metode POST, sambil membuka *Network Monitor* lakukan sesuatu yang mengirim informasi ke server.

... Contohnya, nge-*tweet*. Buka *Network Monitor*, buat sebuah *tweet*, *post*, lalu amati yang terjadi.

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/createtweet.png?raw=true)


---

## Bagian 4: *Public APIs*
API yang digunakan pada suatu website dapat dipelajari dengan, antara lain, berinteraksi dengan website sambil memantau *Network Monitor*. Namun, kecuali memang ada intensi untuk melakukan *reverse engineering*, ini bukan metode yang paling efektif. 

Seringkali, sudah ada dokumentasi publik tentang API yang boleh digunakan yang dapat dibaca-baca. API seperti ini disebut sebagai *Public APIs* atau *Open APIs*. API jenis ini berkebalikan dengan API yang hanya untuk digunakan oleh *developer* internal organisasi.

Dokumentasi API publik ini umumnya menjelaskan:
* API apa saja yang tersedia/diperbolehkan
* batasan-batasan atau ketentuan (jika berlaku) yang diberikan
* bagaimana cara melakukan *authentication* (jika berlaku; sebagian API juga berbayar)
* parameter

Beberapa contoh dokumentasi API publik: [milik GitHub](https://docs.github.com/en/rest), [milik Twitter](https://developer.twitter.com/en/docs/twitter-api), [milik Google](https://developers.google.com/apis-explorer).

Beberapa orang juga mengompilasi daftar dokumentasi API publik, contohnya [di sini](https://github.com/public-apis/public-apis?tab=readme-ov-file).

---

Sebagian API berbentuk lebih sederhana dan lebih mudah digunakan. Salah satu contoh API yang relatif lebih sederhana karena tidak memerlukan *authentication* atau *key* adalah API publik milik NBP (Narodowy Bank Polski, alias Bank Nasional Polandia). 

[*API documentation* dari NBP dapat dilihat di sini.](http://api.nbp.pl/en.html)

Kita akan mencoba menggunakan API dari NBP untuk menunjukkan bagaimana menggunakan JavaScript untuk berinteraksi dengan API, lalu menampilkan hasilnya di webpage.

*Scroll* ke *query examples*. Contoh *query* pertama yang diberikan adalah

```
http://api.nbp.pl/api/exchangerates/rates/a/chf/
```

*Query* ini meminta informasi (mengirim *request*) nilai penukaran mata uang CHF. Di browser, informasi yang diberikan sebagai *response* oleh API dapat dilihat langsung dengan membuka *link* [http://api.nbp.pl/api/exchangerates/rates/a/chf/](http://api.nbp.pl/api/exchangerates/rates/a/chf/) secara langsung.

Saat kita melakukan ini, browser mengirim *request* lalu menampilkan *response* secara langsung. Untuk mendapatkan nilai penukaran mata uang IDR, kita tinggal memodifikasi contoh *request query* yang telah diberikan dengan mengganti `chf` dengan `idr`.

Berikut adalah hasil (kira-kira) yang seharusnya diberikan. Angka persisnya dapat berubah tergantung kurs yang sedang berlaku.

```xml
<ExchangeRatesSeries>
  <Table>A</Table>
  <Currency>rupia indonezyjska</Currency>
  <Code>IDR</Code>
  <Rates>
  <Rate>
    <No>049/A/NBP/2024</No>
    <EffectiveDate>2024-03-08</EffectiveDate>
    <Mid>0.00025267</Mid>
  </Rate>
 </Rates>
</ExchangeRatesSeries>
```

Mengecek *response* dari hasil *request* yang kita kirim via browser berguna untuk mengecek apakah kita sudah membuat *query* dan menggunakan API terkait dengan benar. Selanjutnya, untuk melakukan otomatisasi, proses *request* dan pembacaan *response* dapat dilakukan di JavaScript menggunakan `fetch()` sebagai berikut:

```javascript
async function getIDR() {
  const response = await fetch("http://api.nbp.pl/api/exchangerates/rates/a/idr/");
  const logIDR = await response.json();
  console.log(logIDR);
}

getIDR()
```

Berikut adalah hasil yang didapatkan setelah *script* di atas diuji dengan dijalankan langsung pada *Console* di *Firefox Developer Tools*.

Dapat dilihat bahwa *response* yang diberikan sebagai `Object` memiliki informasi yang sama dengan *response* yang sebelumnya didapatkan:

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/idr-console.png?raw=true)

Jika di-*copy-paste* (klik-kanan > *Copy Object*), didapatkan formatting sebagai berikut — formatting yang bernama formatting JSON (JavaScript Object Notation).

```json
{
  "table": "A",
  "currency": "rupia indonezyjska",
  "code": "IDR",
  "rates": [
    {
      "no": "049/A/NBP/2024",
      "effectiveDate": "2024-03-08",
      "mid": 0.00025267
    }
  ]
}
```

Informasi berapa nilai 1 IDR dalam mata uang Polandia, PLN, ada pada bagian `"mid"`. Untuk mengekstraksi nilai dari `"mid"`, kodenya adalah

```javascript
logIDR.rates[0].mid
```

sehingga (perhatikan bahwa bagian `console.log(...)` telah dimodifikasi)

```javascript
async function getIDR() {
  const response = await fetch("http://api.nbp.pl/api/exchangerates/rates/a/idr/");
  const logIDR = await response.json();
  console.log(logIDR.rates[0].mid);
}
```

akan menghasilkan output `0.00025267`.

Silakan cek apakah informasi ini sesuai dengan nilai kurs yang sesungguhnya sedang berlaku.

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/kurs-rupiah.png?raw=true)

Sekarang kita mengetahui cara menggunakan `fetch()` di JavaScript untuk mendapatkan *response* suatu *public API* (dalam kasus ini, *public API* milik NBP yang memberikan *response* informasi kurs yang sedang berlaku).

Setelahnya, kita bisa membuat webpage HTML dan JavaScript yang menampilkan ini.

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Nilai IDR</title>
    </head>
    <body>
        <h1>Saat ini, 1 IDR sama dengan ...</h1>
        <p>... <span id="idr-rate">(loading ...)</span> PLN.</p>

        <script>
            const idrEl = document.getElementById("idr-rate");

            async function getIDR() {
                const response = await fetch("http://api.nbp.pl/api/exchangerates/rates/a/idr/");
                const logIDR = await response.json();

                // hanya untuk debugging saja
                // console.log(logIDR.rates[0].mid); 

                idrEl.innerText = logIDR.rates[0].mid;
            }

            getIDR();
            
        </script>
    </body>
</html>
```

Hasilnya adalah sebagai berikut (dengan nilai kurs yang mungkin berbeda, jika nilai kurs telah berubah):

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/idr-html.png?raw=true)

---

## Bagian 5: Merancang *Fetch API* menggunakan Google Apps Script
Seseorang yang ingin merancang APIs sendiri umumnya memulai dengan memilih *framework* atau *library* seperti Django, Flask, PHP, atau Node.js. Kemudian, dilakukan *set-up* database contohnya dengan MySQL atau MongoDB. *Hosting* dan *deployment* dapat memanfaatkan platform *cloud*, biasanya berbayar.

Namun, terdapat alternatif metode *deployment* APIs yang lebih cepat dan mudah. Metode ini meliputi merancang API menggunakan Google Apps Script yang akan digunakan untuk mendapatkan informasi dari *database* berupa suatu *spreadsheet* di Google Sheets (yang juga dapat diintegrasikan dengan Google Forms).

Ada berbagai alasan APIs yang dibuat dengan metode ini tidak akan bisa menyaingi APIs yang dirancang mengikuti "*best practices*" (seperti menggunakan *framework* khusus *back-end development*), contohnya isu skalabilitas. Meski demikian, metode ini dapat dipertimbangkan untuk proyek kecil, *prototyping*, untuk belajar tentang konsep web APIs, dan tentu saja untuk bermain-bermain. 

---

Pertama, buat sebuah *spreadsheet* menggunakan Google Sheets. *Spreadsheet* inilah yang akan menjadi *database*. Sebagai contoh, terdapat *spreadsheet* dengan informasi *timestamp* dan tanggal sebagai berikut yang merupakan hasil submisi Google Forms.

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/gsheet-example.png?raw=true)

Kita berencana merancang API yang akan melakukan *fetch* terhadap informasi yang ada pada *spreadsheet* ini.

Pada *spreadsheet* yang telah dibuka, pilih "*Extensions*" > "*Apps Script*". Tindakan ini akan membuka sebuah tab baru.

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/apps-script1.png?raw=true)

Berikut adalah tampilan yang ada pada tab baru tersebut.

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/apps-script2.png?raw=true)

Untuk membuat sebuah *fetch API*, kita mesti menggunakan `doGet()`. Pada `Code.gs`, ganti kode *template* yang sudah ada dengan *sample* berikut.

Jangan lupa untuk menyesuaikan `var sheetName = ...` berdasarkan nama *spreadsheet* yang digunakan. (Untuk contoh ini, nama *spreadsheet* tersebut adalah "*Form Responses 1*".

```javascript
function doGet() {
  var sheetName = "Form Responses 1";
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(sheetName);

  if (!sheet) {
    return {
      error: "404"
    };
  }

  var data = sheet.getDataRange().getValues();
  var headers = data[0];

  var jsonData = [];

  for (var i = 1; i < data.length; i++) {
    var row = data[i];
    var rowObject = {};

    for (var j = 0; j < headers.length; j++) {
      rowObject[headers[j]] = row[j];
    }

    jsonData.push(rowObject);
  }

  return ContentService.createTextOutput(JSON.stringify(jsonData))
    .setMimeType(ContentService.MimeType.JSON);
}

function myFunction() {
  doGet();  
}
```

Sekarang, kita siap menguji API ini!

Untuk melakukan *deployment*, klik tombol biru *Deploy* lalu pilih *New deployment*.

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/apps-script3.png?raw=true)

Selanjutnya, pilih "*Web app*" saat ditanyakan *deployment type*.

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/apps-script4.png?raw=true)

Pada pengaturan konfigurasi web app deployment yang muncul, pilih opsi "*Execute as > User accessing the web app*" dan "*Who has access > Only myself*". Klik tombol biru *Deploy*.

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/apps-script5.png?raw=true)

Setelah proses *updating* selesai, akan muncul tampilan sebagai berikut. *Copy* URL yang ada pada bagian "*Web app*".

URL ini sama seperti URL yang sebelumnya kita gunakan untuk mendapatkan nilai kurs IDR pada *public API* Bank Polandia — kita dapat mengecek hasilnya langsung di browser. *Paste* pada *URL bar* browser, lalu *enter*. 

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/apps-script6.png?raw=true)

Saat diminta melakukan verifikasi *permissions*, ikuti saja alurnya. Pilih *Review Permissions*. 

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/apps-script7.png?raw=true)

Selanjutnya, seleksi *Advanced* lalu pilih *Go to [project name] (unsafe)*. 

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/apps-script8.png?raw=true)

Klik *Allow*. Tindakan ini juga mungkin akan memberikan *security notification*.

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/apps-script9.png?raw=true)

Akhirnya, didapatkan *response* dari *fetch API* yang telah kita rancang menggunakan Google Apps Script.

Perhatikan bahwa isinya *harusnya* sama dengan isi yang ada pada *spreadsheet*.

![alt](https://github.com/amuritna/amuritna.github.io/blob/main/assets/apps-script10.png?raw=true)

Kita telah berhasil membuat sebuah *fetch API*!

API ini dapat dipanggil dengan cara yang sama seperti API publik dari NBP yang sebelumnya digunakan sebagai contoh.

---

## *Further reading*
[*What are Browser Developer Tools?*](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/Tools_and_setup/What_are_browser_developer_tools) 

[Tutorial JavaScript sebagai bahasa pemrograman](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide) yang lebih general di Mozilla Developer Network. Lebih ke menjelaskan *pure JavaScript* daripada menjelaskan JavaScript sebagai alat untuk memanipulasi elemen HTML.

[Dokumentasi JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference) di Mozilla Developer Network.

[*How the Web Works: A Primer for Newcomers to Web Development (or anyone, really)*](https://www.freecodecamp.org/news/how-the-web-works-a-primer-for-newcomers-to-web-development-or-anyone-really-b4584e63585c/) di FreeCodeCamp membahas konsep-konsep *client, server, request*, dan *response* secara (sedikit) lebih mendalam.

Penjelasan tentang [*Object* dalam JavaScript](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Basics) dari MDN. Serta, [apa itu JSON?](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON)

Tentang [konsep "*fetching*"](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Client-side_web_APIs/Fetching_data). "... *many websites use JavaScript APIs to request data from the server and update the page content without a page load*."

[Pengenalan *asynchronous JavaScript dari MDN*](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous/Introducing) untuk yang penasaran dan/atau tidak ingin sekadar *copy-paste*.

[Mengenal HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview), protokol di balik Fetch API. [Artikel tentang JavaScript Fetch API](https://www.freecodecamp.org/news/javascript-fetch-api-for-beginners/) di FreeCodeCamp.

[Dokumentasi terkait](https://developers.google.com/apps-script/guides/web) untuk Google Apps Script.

[Dokumentasi Network Monitor untuk Firefox](https://firefox-source-docs.mozilla.org/devtools-user/network_monitor/).

[*What went wrong? Troubleshooting JavaScript*](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/First_steps/What_went_wrong) dari MDN (lagi).
