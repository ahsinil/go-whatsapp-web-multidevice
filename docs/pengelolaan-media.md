# Pengelolaan Media WhatsApp

Dokumen ini menjelaskan bagaimana aplikasi **Go WhatsApp Web Multidevice** mengelola media seperti gambar, audio, video, file/dokumen, dan sticker.

Pengelolaan media di aplikasi ini mencakup dua sisi utama:

1. **Mengirim media** dari aplikasi ke WhatsApp.
2. **Menerima, membaca, menyimpan, dan mengunduh media** dari pesan WhatsApp yang masuk.

Jenis media utama yang didukung:

- Gambar/image
- Audio
- Video
- File/dokumen
- Sticker

---

## 1. Endpoint Pengiriman Media

Aplikasi menyediakan endpoint REST berikut untuk mengirim media:

| Jenis Media | Endpoint |
|---|---|
| Gambar | `POST /send/image` |
| Audio | `POST /send/audio` |
| File/dokumen | `POST /send/file` |
| Sticker | `POST /send/sticker` |
| Video | `POST /send/video` |

Semua endpoint di atas umumnya menggunakan `multipart/form-data`, karena media biasanya dikirim dalam bentuk file upload.

Jika aplikasi berjalan dalam mode multi-device, gunakan header berikut untuk menentukan nomor/perangkat WhatsApp yang dipakai:

```http
X-Device-Id: my-device
```

---

## 2. Field Umum untuk Pengiriman Media

Sebagian besar request media memiliki field dasar berikut:

| Field | Fungsi |
|---|---|
| `phone` | Nomor atau JID tujuan. |
| `duration` | Durasi disappearing message dalam detik. Opsional. |
| `is_forwarded` | Menandai pesan sebagai forwarded. Opsional. |

Contoh `phone`:

```text
628123456789@s.whatsapp.net
```

Untuk grup, formatnya biasanya:

```text
120363xxxxxxxx@g.us
```

---

## 3. Pengelolaan Gambar / Image

### 3.1 Fungsi

Fitur gambar digunakan untuk mengirim foto, screenshot, bukti transfer, katalog produk, laporan visual, atau gambar promosi.

Endpoint:

```http
POST /send/image
```

### 3.2 Field yang Didukung

| Field | Fungsi |
|---|---|
| `phone` | Nomor/JID tujuan. |
| `caption` | Caption gambar. |
| `view_once` | Mengirim gambar sebagai view once. |
| `image` | File gambar yang diupload. |
| `image_url` | URL gambar yang ingin dikirim. |
| `compress` | Mengompres gambar sebelum dikirim. |
| `duration` | Durasi disappearing message. |
| `is_forwarded` | Menandai gambar sebagai forwarded. |

### 3.3 Contoh Kirim Gambar dari File

```bash
curl -X POST "http://localhost:3000/send/image" \
  -H "X-Device-Id: my-device" \
  -F "phone=628123456789@s.whatsapp.net" \
  -F "caption=Ini contoh gambar" \
  -F "image=@/path/to/image.jpg" \
  -F "compress=true"
```

### 3.4 Contoh Kirim Gambar dari URL

```bash
curl -X POST "http://localhost:3000/send/image" \
  -H "X-Device-Id: my-device" \
  -F "phone=628123456789@s.whatsapp.net" \
  -F "caption=Gambar dari URL" \
  -F "image_url=https://example.com/image.jpg"
```

### 3.5 Kompresi Gambar

Aplikasi mendukung kompresi gambar sebelum dikirim. Ini berguna untuk:

- mengurangi ukuran file,
- mempercepat pengiriman,
- menghemat bandwidth,
- mengurangi risiko gagal upload karena file terlalu besar.

### 3.6 View Once

Jika `view_once=true`, gambar dikirim sebagai pesan sekali lihat.

Contoh:

```bash
-F "view_once=true"
```

---

## 4. Pengelolaan Audio

### 4.1 Fungsi

Fitur audio digunakan untuk mengirim file suara, rekaman, voice note, instruksi audio, atau hasil text-to-speech.

Endpoint:

```http
POST /send/audio
```

### 4.2 Field yang Didukung

| Field | Fungsi |
|---|---|
| `phone` | Nomor/JID tujuan. |
| `audio` | File audio yang diupload. |
| `audio_url` | URL audio yang ingin dikirim. |
| `is_forwarded` | Menandai audio sebagai forwarded. |
| `duration` | Durasi disappearing message. |
| `ptt` | Mengirim sebagai voice note/push-to-talk jika didukung oleh request. |

### 4.3 Contoh Kirim Audio dari File

```bash
curl -X POST "http://localhost:3000/send/audio" \
  -H "X-Device-Id: my-device" \
  -F "phone=628123456789@s.whatsapp.net" \
  -F "audio=@/path/to/audio.mp3"
```

### 4.4 Contoh Kirim Audio dari URL

```bash
curl -X POST "http://localhost:3000/send/audio" \
  -H "X-Device-Id: my-device" \
  -F "phone=628123456789@s.whatsapp.net" \
  -F "audio_url=https://example.com/audio.mp3"
```

### 4.5 Voice Note / PTT

Untuk voice note, WhatsApp biasanya membutuhkan format audio OGG Opus. Aplikasi memiliki dukungan untuk PTT/voice note, termasuk penyesuaian MIME type dan pembuatan waveform.

Contoh:

```bash
curl -X POST "http://localhost:3000/send/audio" \
  -H "X-Device-Id: my-device" \
  -F "phone=628123456789@s.whatsapp.net" \
  -F "audio=@/path/to/voice.ogg" \
  -F "ptt=true"
```

---

## 5. Pengelolaan Video

### 5.1 Fungsi

Fitur video digunakan untuk mengirim video promosi, demo produk, laporan video, komplain pelanggan, atau konten visual lainnya.

Endpoint:

```http
POST /send/video
```

### 5.2 Field yang Didukung

| Field | Fungsi |
|---|---|
| `phone` | Nomor/JID tujuan. |
| `caption` | Caption video. |
| `view_once` | Mengirim video sebagai view once. |
| `video` | File video yang diupload. |
| `video_url` | URL video yang ingin dikirim. |
| `compress` | Mengompres video sebelum dikirim. |
| `gif_playback` | Menampilkan video seperti GIF: looping, silent, autoplay. |
| `duration` | Durasi disappearing message. |
| `is_forwarded` | Menandai video sebagai forwarded. |

### 5.3 Contoh Kirim Video dari File

```bash
curl -X POST "http://localhost:3000/send/video" \
  -H "X-Device-Id: my-device" \
  -F "phone=628123456789@s.whatsapp.net" \
  -F "caption=Ini contoh video" \
  -F "video=@/path/to/video.mp4" \
  -F "compress=true"
```

### 5.4 Contoh Kirim Video dari URL

```bash
curl -X POST "http://localhost:3000/send/video" \
  -H "X-Device-Id: my-device" \
  -F "phone=628123456789@s.whatsapp.net" \
  -F "caption=Video dari URL" \
  -F "video_url=https://example.com/video.mp4"
```

### 5.5 GIF Playback

Jika ingin video tampil seperti GIF, gunakan:

```bash
-F "gif_playback=true"
```

Mode ini cocok untuk video pendek yang ingin terlihat seperti animasi loop.

### 5.6 Kompresi Video

Aplikasi mendukung kompresi video sebelum dikirim. Ini berguna untuk:

- mengurangi ukuran file,
- mempercepat upload,
- menghindari batas ukuran file WhatsApp,
- menghemat bandwidth.

Pengolahan video membutuhkan `ffmpeg`, terutama untuk membuat thumbnail dan melakukan kompresi.

---

## 6. Pengelolaan File / Dokumen

### 6.1 Fungsi

Fitur file/dokumen digunakan untuk mengirim PDF, spreadsheet, dokumen legal, invoice, laporan, atau file administrasi lain.

Endpoint:

```http
POST /send/file
```

### 6.2 Field yang Didukung

| Field | Fungsi |
|---|---|
| `phone` | Nomor/JID tujuan. |
| `caption` | Caption dokumen. |
| `file` | File yang diupload. |
| `file_url` | URL file yang ingin dikirim. |
| `is_forwarded` | Menandai dokumen sebagai forwarded. |
| `duration` | Durasi disappearing message. |

### 6.3 Contoh Kirim Dokumen dari File

```bash
curl -X POST "http://localhost:3000/send/file" \
  -H "X-Device-Id: my-device" \
  -F "phone=628123456789@s.whatsapp.net" \
  -F "caption=Invoice bulan ini" \
  -F "file=@/path/to/invoice.pdf"
```

### 6.4 Contoh Kirim Dokumen dari URL

```bash
curl -X POST "http://localhost:3000/send/file" \
  -H "X-Device-Id: my-device" \
  -F "phone=628123456789@s.whatsapp.net" \
  -F "caption=Dokumen dari URL" \
  -F "file_url=https://example.com/invoice.pdf"
```

### 6.5 Metadata Dokumen

Saat mengirim dokumen, aplikasi mengelola beberapa metadata seperti:

- nama file,
- MIME type,
- ukuran file,
- caption,
- thumbnail dokumen jika bisa dibuat,
- direct path media WhatsApp,
- hash file,
- media key.

Thumbnail dokumen bersifat best-effort. Jika gagal dibuat, pengiriman dokumen tetap dapat berjalan.

---

## 7. Pengelolaan Sticker

### 7.1 Fungsi

Fitur sticker digunakan untuk mengirim gambar sebagai sticker WhatsApp.

Endpoint:

```http
POST /send/sticker
```

### 7.2 Field yang Didukung

| Field | Fungsi |
|---|---|
| `phone` | Nomor/JID tujuan. |
| `sticker` | File sticker/gambar yang diupload. |
| `sticker_url` | URL sticker/gambar. |
| `duration` | Durasi disappearing message. |
| `is_forwarded` | Menandai sticker sebagai forwarded. |

### 7.3 Contoh Kirim Sticker dari File

```bash
curl -X POST "http://localhost:3000/send/sticker" \
  -H "X-Device-Id: my-device" \
  -F "phone=628123456789@s.whatsapp.net" \
  -F "sticker=@/path/to/sticker.png"
```

### 7.4 Contoh Kirim Sticker dari URL

```bash
curl -X POST "http://localhost:3000/send/sticker" \
  -H "X-Device-Id: my-device" \
  -F "phone=628123456789@s.whatsapp.net" \
  -F "sticker_url=https://example.com/sticker.png"
```

### 7.5 Format Sticker yang Didukung

Aplikasi mendukung beberapa format input sticker:

- JPG
- JPEG
- PNG
- WebP
- GIF

Aplikasi dapat mengonversi gambar menjadi WebP agar kompatibel dengan WhatsApp sticker.

### 7.6 Konversi Sticker

Untuk sticker biasa, aplikasi akan:

1. menerima file atau URL,
2. menyimpan sementara file tersebut,
3. membuka gambar,
4. mengubah ukuran ke format yang sesuai,
5. mengonversi ke WebP,
6. mengupload ke WhatsApp,
7. mengirim sebagai sticker.

### 7.7 Animated WebP Sticker

Animated WebP sticker didukung dengan syarat:

| Syarat | Nilai |
|---|---|
| Dimensi | Tepat 512x512 pixel |
| Ukuran file | Di bawah 500KB |
| Durasi maksimum | 10 detik |

Jika animated sticker tidak memenuhi syarat tersebut, sebaiknya resize/kompres terlebih dahulu sebelum dikirim.

---

## 8. Media Masuk dari WhatsApp

Saat nomor WhatsApp menerima pesan media, aplikasi dapat mengirim webhook dengan:

```json
{
  "event": "message",
  "device_id": "628987654321@s.whatsapp.net",
  "payload": {
    "id": "MESSAGE_ID",
    "chat_id": "CHAT_ID",
    "from": "SENDER_JID",
    "from_name": "Sender Name",
    "timestamp": "2025-07-13T11:05:51Z",
    "image": "statics/media/file.jpeg"
  }
}
```

Jenis field media yang mungkin muncul:

| Jenis Media | Field Webhook |
|---|---|
| Gambar | `image` |
| Video | `video` |
| Audio | `audio` |
| Dokumen | `document` |
| Sticker | `sticker` |
| Video note | `video_note` |

---

## 9. Auto-download Media Masuk

Perilaku media masuk dipengaruhi konfigurasi:

```bash
WHATSAPP_AUTO_DOWNLOAD_MEDIA
```

Jika aktif:

- media akan otomatis diunduh,
- webhook berisi path lokal seperti `statics/media/...`.

Jika nonaktif:

- media tidak langsung diunduh,
- webhook dapat berisi URL media,
- backend Anda bisa memutuskan apakah ingin mengunduh media tersebut.

---

## 10. Caption pada Media Masuk

Jika media memiliki caption, caption dapat muncul di:

- field `body`,
- field `caption` di dalam object media.

Contoh:

```json
{
  "event": "message",
  "payload": {
    "body": "Ini bukti transfer",
    "image": {
      "path": "statics/media/bukti-transfer.jpeg",
      "caption": "Ini bukti transfer"
    }
  }
}
```

Backend sebaiknya membaca kedua kemungkinan tersebut agar caption tidak hilang.

---

## 11. Download Ulang Media dari Pesan Tersimpan

Aplikasi menyediakan endpoint untuk mengunduh media dari pesan yang sudah tersimpan:

```http
GET /message/{message_id}/download
```

Contoh:

```bash
curl -X GET "http://localhost:3000/message/3EB0MEDIA123/download?phone=628123456789@s.whatsapp.net" \
  -H "X-Device-Id: my-device"
```

Response download media berisi informasi seperti:

| Field | Fungsi |
|---|---|
| `message_id` | ID pesan yang medianya diunduh. |
| `status` | Status download. |
| `media_type` | Jenis media. |
| `filename` | Nama file hasil download. |
| `file_path` | Lokasi file hasil download. |
| `file_size` | Ukuran file. |

Media yang bisa diunduh ulang meliputi:

- image,
- video,
- audio,
- document,
- sticker.

---

## 12. Rekomendasi Penyimpanan Media

Jika aplikasi digunakan untuk kebutuhan produksi, sebaiknya metadata media disimpan ke database.

Contoh kolom yang disarankan:

| Kolom | Isi |
|---|---|
| `message_id` | ID pesan WhatsApp. |
| `device_id` | Device/nomor WhatsApp penerima event. |
| `chat_id` | Chat tempat media dikirim/diterima. |
| `sender_jid` | Pengirim media. |
| `sender_name` | Nama pengirim. |
| `timestamp` | Waktu pesan. |
| `media_type` | image, video, audio, document, sticker, atau video_note. |
| `caption` | Caption media. |
| `local_path` | Path lokal jika media sudah diunduh. |
| `remote_url` | URL media jika auto-download nonaktif. |
| `filename` | Nama file. |
| `file_size` | Ukuran file jika tersedia. |
| `raw_payload` | Payload webhook mentah untuk audit/debugging. |

---

## 13. Contoh Workflow Praktis

### 13.1 Pelanggan Mengirim Bukti Transfer

1. Pelanggan mengirim gambar bukti transfer.
2. Webhook menerima `event: "message"` dengan field `image`.
3. Backend menyimpan `message_id`, `chat_id`, `from`, `caption`, dan path/URL gambar.
4. Backend bisa menjalankan OCR atau meneruskan ke admin.
5. Backend mengirim balasan otomatis.

### 13.2 Admin Mengirim Invoice PDF

```bash
curl -X POST "http://localhost:3000/send/file" \
  -H "X-Device-Id: my-device" \
  -F "phone=628123456789@s.whatsapp.net" \
  -F "caption=Invoice bulan ini" \
  -F "file=@/path/to/invoice.pdf"
```

### 13.3 Bot Mengirim Video Promosi

```bash
curl -X POST "http://localhost:3000/send/video" \
  -H "X-Device-Id: my-device" \
  -F "phone=628123456789@s.whatsapp.net" \
  -F "caption=Promo terbaru kami" \
  -F "video=@/path/to/promo.mp4" \
  -F "compress=true"
```

### 13.4 Sistem Mengirim Sticker

```bash
curl -X POST "http://localhost:3000/send/sticker" \
  -H "X-Device-Id: my-device" \
  -F "phone=628123456789@s.whatsapp.net" \
  -F "sticker=@/path/to/sticker.png"
```

---

## 14. Best Practice

Untuk penggunaan produksi, sebaiknya:

1. Simpan metadata media ke database.
2. Simpan file penting ke storage permanen seperti S3, MinIO, atau storage internal.
3. Jangan memproses file besar langsung di request webhook.
4. Proses OCR, transkripsi, scan dokumen, atau kompresi tambahan secara asynchronous.
5. Validasi ukuran file sebelum mengirim.
6. Gunakan `message_id` sebagai kunci utama untuk tracking.
7. Tangani dua bentuk media masuk: path lokal dan URL remote.
8. Amankan folder media karena bisa berisi data sensitif.
9. Gunakan HTTPS untuk webhook dan API publik.
10. Batasi akses endpoint dengan Basic Auth, firewall, atau reverse proxy.

---

## 15. Ringkasan

Aplikasi ini mendukung pengelolaan media WhatsApp secara lengkap:

- **Gambar**: upload/URL, caption, view once, kompresi, thumbnail.
- **Audio**: upload/URL, MIME detection, durasi, voice note/PTT.
- **Video**: upload/URL, caption, view once, thumbnail, kompresi, GIF playback.
- **File/dokumen**: upload/URL, caption, filename, MIME type, thumbnail dokumen.
- **Sticker**: upload/URL, konversi WebP, resize 512x512, dukungan animated WebP bersyarat.
- **Media masuk**: diterima melalui webhook sebagai `event: "message"` dengan field media seperti `image`, `video`, `audio`, `document`, `sticker`, atau `video_note`.
- **Download ulang**: tersedia melalui endpoint `GET /message/{message_id}/download`.
