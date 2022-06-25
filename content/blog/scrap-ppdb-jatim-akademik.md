---
title: "Scraping Urutan Prestasi Nilai Akademik PPDB Jawa Timur"
date: 2022-06-25T19:09:46+07:00
---

Pakai Bahasa Indonesia dulu, ya :)

# Pembukaan

Pada saat postingan ini ditulis, [PPDB Jawa Timur](https://ppdbjatim.net) jalur "Prestasi Nilai Akademik" SMA masih berlangsung, dan saya adalah salah satu pendaftarnya. Pendaftarannya itu sendiri berlangsung hingga Minggu, 26 Juni 2022.

Tentu saja, setelah mendaftar, saya kerap memeriksa peringkat saya dengan pendaftar lain di jalur yang sama dan sekolah yang sama juga. Di situs nya tertulis bahwa _data_ di-_update_ setiap 30 menit. Berarti saya harus memeriksanya setiap 30 menit pula. Sangat tidak efisien.

Untuk mengisi _kegabutan_, saya berinisiatif untuk membuat sebuah _script_ sederhana untuk _scrapping_ situs PPDB Jawa Timur, khususnya pada halaman "Hasil Pemeringkatan Jalur Prestasi Nilai Akademik (SMA)". _Sepertinya seru, tuh._ Dan _Alhamdulillah_ berhasil!

# Apa Yang Ingin Dicapai?

Saya ingin membuat sebuah _script_ sederhana yang akan:

* _Scrap_ situs PPDB Jawa Timur, pendaftaran jalur "Prestasi Nilai Akademik" SMA
* Mengambil data yang diinginkan, yaitu
    1. Nomor urutan saya
    2. Nama dan nilai urutan pertama
    3. Nama dan nilai urutan terakhir
    4. Waktu _scraping_ dilakukan
* Mengirimkan hasil data ke bot Telegram, setiap 30 menit menggunakan _cron_

# _Briefcase_ Dari Alat dan Bahan

Saya menggunakan Python, bahasa pemrograman populer yang _simple_ dan mudah dimengerti sebagai bahasa pemrograman dari _script_ yang saya buat. Untuk _request_ situs, saya menggunakan modul `requests` dan `BeautifulSoup` untuk _parsing_ konten HTML. _Last but not least_, `python-telegram-bot` sebagai wrapper API untuk Bot Telegram

# Mulai!

## Persiapan

_Import_ modul yang diperlukan:

```python
import requests # Request HTML
from bs4 import BeautifulSoup # Parsing HTML
import telegram # Alias untuk `python-telegram-bot`, wrapper API Bot Telegram
from datetime import datetime # Ambil informasi tanggal dan waktu
```

, dan atur variabel:

```python
URL = "https://XX.ppdbjatim.net/pengumuman/pemeringkatan/sma_rapor/sekolah/XXX" # URL dari halaman pendaftaran
TG_TOKEN = "XXXXXXX" # Token Bot Telegram
TG_CHATID = "XXXXX" # Chat ID pengguna
RESULT_FILENAME = "Urutan Prestasi Nilai Akademik.txt" # Hasil file TXT
```

## Request Ke Situs

Pertama-tama, tentu saja, buat _request_ menuju ke situs PPDB:

```python
a = requests.get(URL)
```

## Analisa dan Merancang _Scraper_
Saya mulai dengan menganalisa isi HTML dari halaman PPDB, spesifik untuk sekolah yang saya pilih. Isi yang saya cari yaitu konten tabel pemeringkatan, kira-kira seperti ini:

```html
...
...

<tr>
<td>-</td>
<td><NISN SISWA></td>
<td>
<b><NAMA SISWA></b>
</td>
<td><NILAI RATA-RATA></td>
</tr>

<tr>
<td>-</td>
<td><NISN SISWA></td>
<td>
<b><NAMA SISWA></b>
</td>
<td><NILAI RATA-RATA></td>
</tr>

...
...
```

Elemen `tr` tersebut berada di elemen induk `table`, dengan `id` `tbl-rangking`. Jadi, kode untuk _parsing_ kode HTML tersebut menggunakan `BeautifulSoup` adalah seperti berikut:

```python
b = BeautifulSoup(a.content, "html.parser") # Parse HTML
c = b.find("table", {"id": "tbl-rangking"}) # Cari elemen `table` yang dimaksud
d = c.find_all("tr")[1:] # Ambil semua elemen `tr`, terkecuali elemen pertama karena berisi header tabel
```

## Simpan Data Di Dictionary

Sebelum memproses data akhir, saya menyimpan data _overall_ pada sebuah _dictionary_, sebagai jaga-jaga jika dibutuhkan.

```python
data = [] # Buat dictionary

for element in d: # `for loop` terhadap elemen `tr`
    elementt = element.find_all("td") # Ambil semua elemen `td` pada elemen `tr` saat ini
    data.append({
        "NISN": elementt[1].text.strip(), # Tulis NISN
        "NAMA": elementt[2].text.strip(), # Tulis nama pendaftar
        "NILAI": elementt[3].text.strip() # Tulis nilai pendaftar
    })
```

Nah, data sudah berada di dictionary. Sip!

## Tulis Hasil Akhir

Saatnya untuk menulis data akhir di _file_ _text_:

```python
tgText = "" # Buat variabel untuk menampung data akhir
urutanHendra = "" # Urutan saya, akan diperlukan nanti
namaPertama = data[0]["NAMA"] # Nama urutan pertama, akan diperlukan nanti
nilaiPertama = data[0]["NILAI"] # Nilai urutan pertama, akan diperlukan nanti
namaTerakhir = data[len(data) - 1]["NAMA"] # Nama urutan terakhir, akan diperlukan nanti
nilaiTerakhir = data[len(data) - 1]["NILAI"] # Nilai urutan terakhir, akan diperlukan nanti

for index, key in enumerate(data): # `for loop` terhadap isi dari dictionary
    index += 1 # Tambah `index`, karena selalu berawal dari 0
    tgText += f'URUTAN KE-{index}: {key["NAMA"]} ({key["NILAI"]} | {key["NISN"]})\n' # Tulis barisan

    if key["NAMA"] == "HENDRA MANUDINATA": # Jika nama pendaftar saat ini adalah saya,
        urutanHendra = index # maka atur variable `urutanHendra` dengan index saat ini

with open(RESULT_FILENAME, 'w') as file: # Buka,
    file.write(tgText) # dan isi file `RESULT_FILENAME` ("Urutan Prestasi Nilai Akademik.txt")
```

Sampai sini, _script_ akan melakukan _scraping_ situs, mengambil data yang diinginkan, dan menuliskannya ke _file_ _text_ ("Urutan Prestasi Nilai Akademik.txt").

## Pengiriman Data Melalui Bot Telegram

Mari lanjut ke proses pengiriman data melalui Bot Telegram, menggunakan `python-telegram-bot`:

```python
# Atur pesan Telegram
TG_TEXT = ""
TG_TEXT += f"<b><u>Urutan Prestasi Nilai Akademik</u></b>\n\n"
TG_TEXT += f"<b>Urutan Anda: {urutanHendra}</b>\n\n"
TG_TEXT += f"Urutan pertama: {namaPertama} | {nilaiPertama}\n"
TG_TEXT += f"Urutan terakhir: {namaTerakhir} | {nilaiTerakhir}\n\n"
TG_TEXT += f"<b>Waktu scraping: {datetime.now().strftime('%H:%M')}</b>"

# Kirim file text, dengan pesan Telegram sebelumnya sebagai caption
bot.send_document(chat_id=TG_CHATID, document=open(RESULT_FILENAME, "rb"), caption=TG_TEXT, parse_mode="HTML")
```

## _Full Code_

Berikut adalah _full code_ dari _script_ yang tersedia di [Gist](https://gist.github.com/hendramanudinata03/97d51391be332fc86688771b2e24ca0a) saya:

{{< gist hendramanudinata03 97d51391be332fc86688771b2e24ca0a >}}

# Penutup

Demikianlah pengalaman lainnya yang dapat saya _share_. Semoga dapat menjadi referensi Anda, terlebih untuk peserta PPDB di Jawa Timur.

Terima kasih sudah membaca!
