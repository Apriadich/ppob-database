# CLAUDE.md — Admin Desktop Kioskitah (WinUI 3 + C#)

Salin file ini ke akar proyek WinUI 3 (di samping file `.sln`). Claude di VS Code membacanya otomatis.

## 1. Sumber kebenaran (Figma)

Desain ada di satu Section Figma. Jika ada yang bertentangan, urutan menang:

1. **ATURAN BERSAMA** (frame di dalam section)
2. **ATURAN & HANDOFF** tiap workspace
3. **DEV NOTES** tiap workspace (peta kontrol WinUI 3 + kondisi Loading / Kosong / Error)
4. Gambar layar dan popup

Link (file `12rwKEuykKHIceSImzfJON`, bagian nama setelah kunci file boleh diabaikan):

| Bagian | node-id |
|---|---|
| Section utama (semua workspace) | `5269-354` |
| BACA DULU (peta workspace) | `5269-323` |
| ATURAN BERSAMA | `5248-323` |
| WORKSPACE 04 — User & Agen | `5137-224` |
| WORKSPACE 05 — Konten & Dokumen | `5172-236` |
| WORKSPACE 06 — Keuangan | `5183-245` |
| WORKSPACE 07 — Laporan | `5196-263` |
| WORKSPACE 08 — Keamanan & Audit | `5234-293` |
| WORKSPACE 09 — Pengaturan Sistem | `5242-308` |

Format: `https://www.figma.com/design/12rwKEuykKHIceSImzfJON/Aplikasi-PPOB-Kioskitah?node-id=<node-id>`

Semua angka, nama, dan tanggal di desain adalah **contoh**. Jangan ditanam di kode.

## 2. Teknologi

- WinUI 3 (Windows App SDK), C#, .NET sesuai template proyek.
- MVVM dengan **CommunityToolkit.Mvvm** (`ObservableObject`, `[ObservableProperty]`, `[RelayCommand]`).
- Dependency injection: `Microsoft.Extensions.DependencyInjection`. HTTP: `HttpClient` lewat `IHttpClientFactory`.
- Backend: Go + PostgreSQL di GCP. Dashboard hanya memanggil **API backend**. Tidak ada akses langsung ke database, provider, atau gateway dari aplikasi ini.
- Kontrol sesuai DEV NOTES (mis. `NavigationView` mode atas, `DataGrid` dari CommunityToolkit, `ContentDialog`, `NumberBox`, `ToggleSwitch`, `InfoBar`).

## 3. Struktur folder

```
src/
  App/                      App.xaml, DI, navigasi
  Core/                     model, enum, interface API (tanpa UI)
  Services/                 klien API, dialog, penyimpanan token
  ViewModels/<Modul>/       satu ViewModel per tab
  Views/<Modul>/            satu Page / UserControl per tab
  Controls/                 KpiCard, StatusBadge, GoldButton, EmptyState, dll.
  Styles/                   Colors.xaml, Buttons.xaml, Typography.xaml
tests/
```

Modul = nama workspace: `UserAgen`, `Konten`, `Keuangan`, `Laporan`, `Keamanan`, `Pengaturan`.
Satu tab Figma = satu `Page` + satu `ViewModel` (mis. `AkunPage` + `AkunViewModel`).

## 4. Aturan kode

- Logika di ViewModel; code-behind (`.xaml.cs`) hanya untuk hal murni UI.
- Gunakan `x:Bind` (OneWay / TwoWay) dan `[RelayCommand]`. `CanExecute` mengatur `IsEnabled` tombol.
- Semua layar punya satu enum **`ViewState { Loading, Empty, Error, Ready }`** di ViewModel dasar. Tampilan Loading (skeleton), Kosong, dan Error mengikuti contoh Figma "CONTOH KONDISI — AKUN" dan "ACUAN GAYA KONDISI LAYAR".
- Panggilan API selalu `async`/`await` dengan `CancellationToken`. Selama request berjalan, tombol aksi nonaktif dan menampilkan `ProgressRing`.
- Operasi tulis (simpan, hapus, setujui) mengirim **idempotency key**. Jangan kirim ganda saat tombol ditekan dua kali.
- Pencarian, urut, dan paging dilakukan **di server**. Jangan memuat seluruh tabel ke memori.
- Nama kelas dan properti dalam bahasa Inggris; teks yang tampil ke pengguna dalam bahasa Indonesia, disimpan di file resource (`.resw`), bukan ditulis langsung di XAML.

## 5. Gaya UI (ringkas)

| Elemen | Nilai |
|---|---|
| Font | Inter |
| Latar halaman | `#F4F7F9` |
| Biru utama / aksen | `#3866A3` / `#2161A8` |
| Merah / hijau / kuning | `#A1241F` / `#14A14F` / `#EE9A2C` |
| Garis kartu / garis input | `#CFDEE8` / `#7F9FBE` |
| Teks / teks redup | `#263642` / `#5E7080` |
| Header tabel / baris zebra | `#D6E4F0` / `#F0F5F9` |

Tombol (semua berikon, gaya desktop):

- **Simpan / Terapkan / Publish**: gradasi `#FFF1B8 → #FFD97A → #FBB64B`, garis `#7A4F14` tebal 2, teks `#4A2F0F`, radius 4, ikon cokelat tanpa kotak ikon.
- **Batal / Tutup**: isi `#FDEDEC`, garis `#C95C52` tebal 2, kotak ikon merah `#C63D33`, teks `#8B2C24`.
- **Hapus / Nonaktifkan / Blokir**: merah solid `#A1241F`.
- **Sekunder (Ekspor, Reset, Detail)**: putih, garis `#7F9FBE`, kotak ikon biru `#3866A3`.
- Tombol kecil di baris tabel: ikon 12 px tanpa kotak.
- `ToggleSwitch`: radius 5. Popup: `ContentDialog` tanpa latar gelap, hanya bayangan; tombol ✕ di sudut menjadi merah saat disentuh; urutan tombol Batal lalu aksi.
- Status selalu **warna + teks**, tidak hanya warna.

## 6. Uang, waktu, bahasa

- Uang disimpan dan dikirim sebagai **integer rupiah** (`long`). Tampilan: `Rp 1.234.567` (titik ribuan). Jangan pakai `double`/`float`.
- Waktu dari server dalam UTC; tampilkan **WITA** (Asia/Makassar).

## 7. Keamanan

- Secret, token, PIN, signature, URL privat, DB URL **tidak pernah** ada di aplikasi ini. Kunci ditampilkan bertopeng; nilai asli hanya di Secret Manager (backend).
- Token admin disimpan aman (Windows Credential Locker / `PasswordVault`), bukan di file teks.
- Jangan menulis secret ke log.
- Izin: sembunyikan atau nonaktifkan aksi yang tidak diizinkan, tetapi **backend tetap yang memutuskan**.

## 8. Keputusan produk yang berlaku

- Hanya dua tipe akun: **User** dan **Agen** (tidak ada level akun).
- **Mode maker-checker mati** (satu admin): admin langsung menyetujui sendiri; alasan tetap wajib dan masuk AuditLog. Bendera `RequireChecker` dibaca dari Pengaturan → Keamanan. Tampilan harus bisa berubah bila bendera dihidupkan.
- Penarikan komisi referral otomatis oleh backend, tanpa persetujuan admin.
- **Satu pemilik per pengaturan** (lihat ATURAN BERSAMA bagian A): batas top up di WS04, kontak CS dan force update di WS05, gateway dan biaya admin di WS06, sisanya di WS09. Layar lain hanya membaca.
- Di luar aplikasi ini (sudah ada di dashboard utama): Produk & Harga, Provider, Transaksi, Deposit, koreksi saldo manual, antrean tiket Pengaduan.

## 9. Cara bekerja

1. Kerjakan **satu tab per tugas**. Baca ATURAN BERSAMA, ATURAN & HANDOFF workspace, lalu DEV NOTES-nya.
2. Buat: model → interface API → ViewModel → View → gaya → tes.
3. Setelah selesai, cek terhadap desain: layout, tombol, teks Indonesia, kondisi Loading/Kosong/Error, dan tombol nonaktif.
4. Jika desain ambigu atau bertentangan dengan aturan, **tanya dulu**; jangan menebak.
5. Jangan mengubah aturan, gaya global, atau modul lain tanpa diminta.

## 10. Perintah

```
dotnet build
dotnet test
```

(Sesuaikan jika proyek memakai skrip lain.)

## 11. Contoh prompt

```
Baca ATURAN BERSAMA: <link node 5248-323>
Implementasikan WORKSPACE 04 — User & Agen, tab Akun, dari <link node 5137-224>.
Ikuti ATURAN & HANDOFF, DEV NOTES, dan contoh Loading/Kosong/Error.
Kerjakan tab Akun saja dulu.
```
