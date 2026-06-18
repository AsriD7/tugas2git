# Penjelasan Project LintasArena

Dokumen ini menjelaskan cara kerja project **LintasArena** dari awal, terutama untuk membantu memahami kode saat presentasi. Project ini dibuat dengan **Flutter** sebagai frontend dan **Supabase** sebagai backend untuk login, database, dan penyimpanan gambar.

## 1. Ringkasan Project

LintasArena adalah aplikasi booking venue/lapangan olahraga. Aplikasi ini memiliki dua jenis pengguna:

1. **Pelanggan**
   - Daftar akun.
   - Login.
   - Melihat daftar venue/lapangan.
   - Mencari venue berdasarkan nama, lokasi, atau jenis olahraga.
   - Melihat detail venue.
   - Memilih tanggal dan jam booking.
   - Melihat riwayat booking.
   - Mendapat notifikasi jika booking disetujui atau ditolak.

2. **Admin**
   - Login sebagai admin.
   - Melihat dashboard admin.
   - Menambah venue/lapangan.
   - Mengedit venue/lapangan.
   - Menghapus venue/lapangan.
   - Melihat daftar booking pada venue tertentu.
   - Menyetujui, menolak, atau menyelesaikan booking.

## 2. Teknologi yang Digunakan

Project ini menggunakan beberapa package penting:

```yaml
provider: ^6.1.1
intl: ^0.20.2
flutter_staggered_animations: ^1.1.1
supabase_flutter: ^2.14.2
image_picker: ^1.2.2
```

Penjelasannya:

- **provider** digunakan untuk state management. Data login, data lapangan, dan data pesanan disimpan di controller, lalu bisa dipakai oleh banyak halaman.
- **intl** digunakan untuk format tanggal, misalnya format tanggal Indonesia.
- **flutter_staggered_animations** digunakan untuk animasi tampilan list/card.
- **supabase_flutter** digunakan untuk koneksi ke Supabase.
- **image_picker** digunakan admin untuk memilih gambar venue dari perangkat.

## 3. Struktur Folder Utama

Struktur kode utama ada di folder `lib`.

```text
lib/
  main.dart
  konstanta.dart
  model/
    pengguna.dart
    lapangan.dart
    pesanan.dart
  pengontrol/
    pengontrol_otentikasi.dart
    pengontrol_lapangan.dart
    pengontrol_pesanan.dart
  tampilan/
    tampilan_masuk.dart
    tampilan_daftar.dart
    tampilan_utama.dart
    tampilan_detail_venue.dart
    tampilan_pilih_jadwal.dart
    tampilan_notifikasi.dart
    tampilan_profil.dart
    tampilan_ubah_profil.dart
    tampilan_ubah_password.dart
  admin/
    tampilan_dashboard_admin.dart
    tampilan_form_lapangan.dart
    tampilan_daftar_booking_admin.dart
```

Pola pembagiannya:

- `model/` berisi bentuk data.
- `pengontrol/` berisi logic aplikasi dan komunikasi ke Supabase.
- `tampilan/` berisi halaman untuk pelanggan.
- `admin/` berisi halaman khusus admin.
- `main.dart` adalah pintu masuk aplikasi.
- `konstanta.dart` menyimpan konfigurasi Supabase.

## 4. Cara Project Berjalan dari Awal

File utama project adalah `lib/main.dart`.

Saat aplikasi dijalankan, fungsi `main()` akan berjalan terlebih dahulu.

```dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await Supabase.initialize(
    url: Konstanta.supabaseUrl,
    anonKey: Konstanta.supabaseAnonKey,
  );

  await initializeDateFormatting('id_ID', null);

  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => PengontrolOtentikasi()),
        ChangeNotifierProvider(create: (_) => PengontrolPesanan()),
        ChangeNotifierProvider(create: (_) => PengontrolLapangan()),
      ],
      child: const AplikasiLintasArena(),
    ),
  );
}
```

Urutannya:

1. `WidgetsFlutterBinding.ensureInitialized()` memastikan Flutter siap sebelum menjalankan kode async.
2. `Supabase.initialize()` menghubungkan aplikasi ke Supabase.
3. `initializeDateFormatting('id_ID', null)` menyiapkan format tanggal Indonesia.
4. `runApp()` menjalankan aplikasi.
5. `MultiProvider` memasang tiga controller:
   - `PengontrolOtentikasi`
   - `PengontrolPesanan`
   - `PengontrolLapangan`

## 5. Konsep Provider dan Controller

Project ini memakai `Provider` untuk menyimpan dan membagikan state.

Contohnya:

```dart
ChangeNotifierProvider(create: (_) => PengontrolOtentikasi())
```

Artinya, `PengontrolOtentikasi` bisa diakses dari halaman lain.

Ada dua cara umum mengakses Provider:

```dart
context.read<PengontrolOtentikasi>()
```

Dipakai jika hanya ingin memanggil fungsi, misalnya login.

```dart
context.watch<PengontrolOtentikasi>()
```

Dipakai jika tampilan harus otomatis berubah ketika data berubah.

Setiap controller mewarisi `ChangeNotifier`. Jika data berubah, controller memanggil:

```dart
notifyListeners();
```

Maka widget yang memakai `watch` akan rebuild otomatis.

### Jika Dosen Bertanya: Mana State Management-nya?

State management project ini ada di `main.dart`, tepatnya pada bagian `MultiProvider`.

```dart
runApp(
  MultiProvider(
    providers: [
      ChangeNotifierProvider(create: (_) => PengontrolOtentikasi()),
      ChangeNotifierProvider(create: (_) => PengontrolPesanan()),
      ChangeNotifierProvider(create: (_) => PengontrolLapangan()),
    ],
    child: const AplikasiLintasArena(),
  ),
);
```

Bagian tersebut adalah pusat pendaftaran state management. Artinya, tiga controller didaftarkan agar bisa dipakai oleh halaman-halaman lain.

Tiga state utama pada project ini:

1. `PengontrolOtentikasi`

   Mengatur state login user, data profil user, role admin/pelanggan, logout, update profil, dan update password.

2. `PengontrolLapangan`

   Mengatur state daftar venue/lapangan, tambah venue, edit venue, hapus venue, dan upload gambar.

3. `PengontrolPesanan`

   Mengatur state riwayat booking, pesanan aktif, countdown waktu, dan status pesanan.

Contoh state login ada di `pengontrol_otentikasi.dart`.

```dart
Pengguna? _penggunaSaatIni;

Pengguna? get penggunaSaatIni => _penggunaSaatIni;
bool get apakahSudahMasuk => _penggunaSaatIni != null;
```

Penjelasan:

- `_penggunaSaatIni` menyimpan data user yang sedang login.
- Jika `_penggunaSaatIni` bernilai `null`, berarti user belum login.
- Jika `_penggunaSaatIni` ada isinya, berarti user sudah login.
- Getter `apakahSudahMasuk` dipakai UI untuk mengecek status login.

Saat user berhasil login, controller mengambil data profil dari Supabase lalu menyimpan ke `_penggunaSaatIni`.

```dart
_penggunaSaatIni = Pengguna.fromJson(response);
notifyListeners();
```

`notifyListeners()` adalah bagian penting dari state management. Fungsinya memberi tahu UI bahwa data sudah berubah.

Contoh UI yang mendengarkan perubahan state ada di `main.dart`.

```dart
home: Consumer<PengontrolOtentikasi>(
  builder: (context, auth, child) {
    if (auth.apakahSudahMasuk) {
      final peran = auth.penggunaSaatIni?.peran;
      if (peran == 'admin') {
        return const TampilanDashboardAdmin();
      }
      return const TampilanUtama();
    }
    return const TampilanMasuk();
  },
),
```

Alurnya:

1. Awalnya user belum login, maka `apakahSudahMasuk` bernilai `false`.
2. Aplikasi menampilkan `TampilanMasuk`.
3. User login.
4. `PengontrolOtentikasi` menyimpan data user ke `_penggunaSaatIni`.
5. Controller memanggil `notifyListeners()`.
6. `Consumer<PengontrolOtentikasi>` di `main.dart` rebuild otomatis.
7. Jika user admin, masuk ke `TampilanDashboardAdmin`.
8. Jika user pelanggan, masuk ke `TampilanUtama`.

Contoh lain penggunaan Provider ada di `tampilan_utama.dart`.

```dart
final daftarVenue = context.watch<PengontrolLapangan>().daftarVenue;
```

Artinya halaman utama mendengarkan perubahan daftar venue. Jika admin menambah, mengedit, atau menghapus venue lalu controller memanggil `notifyListeners()`, data venue di UI bisa diperbarui.

Contoh penggunaan `read` ada saat tombol login ditekan.

```dart
await context.read<PengontrolOtentikasi>().masuk(
  _emailPengontrol.text,
  _kataSandiPengontrol.text,
);
```

`read` dipakai karena tombol hanya perlu memanggil fungsi login, bukan terus-menerus mendengarkan perubahan data.

Perbedaan `read`, `watch`, dan `Consumer`:

```text
context.read<T>()
```

Dipakai untuk memanggil fungsi dari controller. Tidak rebuild otomatis.

```text
context.watch<T>()
```

Dipakai untuk membaca data dan rebuild otomatis ketika data berubah.

```text
Consumer<T>()
```

Dipakai untuk membuat bagian UI tertentu mendengarkan perubahan dari controller.

Jawaban singkat untuk presentasi:

```text
State management di project ini menggunakan Provider.
Provider dipasang di main.dart dengan MultiProvider.
Ada tiga controller utama: PengontrolOtentikasi untuk state login,
PengontrolLapangan untuk state data venue, dan PengontrolPesanan untuk state booking.
Ketika data berubah, controller memanggil notifyListeners(),
lalu UI yang memakai Consumer, context.watch, atau context.read akan merespons perubahan tersebut.
```

## 6. Routing Awal Aplikasi

Di `main.dart`, halaman awal ditentukan oleh kondisi login.

```dart
home: Consumer<PengontrolOtentikasi>(
  builder: (context, auth, child) {
    if (auth.apakahSudahMasuk) {
      final peran = auth.penggunaSaatIni?.peran;
      if (peran == 'admin') {
        return const TampilanDashboardAdmin();
      }
      return const TampilanUtama();
    }
    return const TampilanMasuk();
  },
),
```

Artinya:

- Jika belum login, tampilkan `TampilanMasuk`.
- Jika sudah login dan perannya `admin`, tampilkan `TampilanDashboardAdmin`.
- Jika sudah login dan bukan admin, tampilkan `TampilanUtama`.

## 7. Setup Supabase pada Project

Konfigurasi Supabase ada di `lib/konstanta.dart`.

```dart
class Konstanta {
  static const String supabaseUrl = 'https://tyedbqebeszvjcxdxoyl.supabase.co';
  static const String supabaseAnonKey = '...';
}
```

Lalu dipakai di `main.dart`:

```dart
await Supabase.initialize(
  url: Konstanta.supabaseUrl,
  anonKey: Konstanta.supabaseAnonKey,
);
```

Penjelasan:

- `supabaseUrl` adalah alamat project Supabase.
- `supabaseAnonKey` adalah public anon key dari Supabase.
- Key ini dipakai aplikasi Flutter agar bisa mengakses Supabase sesuai permission yang sudah diatur di Supabase.

### Cara Menyiapkan Supabase dari Awal

Jika menjelaskan ke dosen, alur setup Supabase secara umum seperti ini:

1. Buat project di Supabase.
2. Ambil `Project URL` dan `anon public key`.
3. Masukkan ke file `lib/konstanta.dart`.
4. Aktifkan Authentication dengan email/password.
5. Buat tabel database:
   - `profil`
   - `lapangan`
   - `pesanan`
6. Buat bucket storage:
   - `lapangan_images`
7. Atur policy/permission agar aplikasi bisa membaca/menulis data sesuai kebutuhan.

### Konfigurasi Supabase Praktis

Bagian ini adalah langkah konfigurasi Supabase yang bisa dijelaskan saat presentasi atau dipakai jika project ingin dibuat ulang dari awal.

#### 1. Membuat Project Supabase

Langkah awal:

1. Buka website Supabase.
2. Login atau daftar akun.
3. Klik `New project`.
4. Isi nama project, password database, dan region.
5. Tunggu sampai project selesai dibuat.

Setelah project jadi, Supabase akan menyediakan:

- Database PostgreSQL.
- Authentication.
- Storage.
- API URL.
- API key.

#### 2. Mengambil URL dan Anon Key

Masuk ke menu:

```text
Project Settings -> API
```

Ambil dua data berikut:

```text
Project URL
anon public key
```

Lalu masukkan ke file `lib/konstanta.dart`.

```dart
class Konstanta {
  static const String supabaseUrl = 'ISI_PROJECT_URL_SUPABASE';
  static const String supabaseAnonKey = 'ISI_ANON_PUBLIC_KEY_SUPABASE';
}
```

Di project ini, file `konstanta.dart` sudah berisi URL dan anon key Supabase.

```dart
class Konstanta {
  static const String supabaseUrl = 'https://tyedbqebeszvjcxdxoyl.supabase.co';
  static const String supabaseAnonKey = '...';
}
```

File ini kemudian dipanggil di `main.dart`.

```dart
await Supabase.initialize(
  url: Konstanta.supabaseUrl,
  anonKey: Konstanta.supabaseAnonKey,
);
```

Jadi koneksi Flutter ke Supabase terjadi saat aplikasi pertama kali dijalankan.

#### 3. Mengaktifkan Authentication Email dan Password

Masuk ke menu:

```text
Authentication -> Providers
```

Pastikan provider `Email` aktif.

Project ini memakai login email dan password lewat kode:

```dart
await _supabase.auth.signInWithPassword(
  email: email,
  password: kataSandi,
);
```

Register memakai:

```dart
await _supabase.auth.signUp(
  email: email,
  password: kataSandi,
);
```

#### 4. Membuat Tabel `profil`

Tabel `profil` dipakai untuk menyimpan data tambahan user karena Supabase Auth hanya menangani akun login. Data seperti nama lengkap dan role disimpan di tabel ini.

Contoh struktur tabel:

```sql
create table public.profil (
  id uuid primary key references auth.users(id) on delete cascade,
  nama_lengkap text not null,
  email text not null,
  peran text not null default 'pelanggan'
);
```

Penjelasan:

- `id` mengikuti id dari Supabase Auth.
- `nama_lengkap` menyimpan nama user.
- `email` menyimpan email user.
- `peran` menyimpan role, misalnya `pelanggan` atau `admin`.

Saat user daftar, kode memasukkan data ke tabel ini:

```dart
await _supabase.from('profil').insert({
  'id': response.user!.id,
  'nama_lengkap': namaLengkap,
  'email': email,
  'peran': 'pelanggan',
});
```

#### 5. Membuat Tabel `lapangan`

Tabel `lapangan` menyimpan data venue yang tampil di halaman pelanggan dan dashboard admin.

Contoh struktur tabel:

```sql
create table public.lapangan (
  id text primary key,
  nama text not null,
  lokasi text not null,
  jenis text not null,
  rating numeric default 5.0,
  harga_mulai text,
  harga_coret text,
  nama_pemilik text not null,
  warna_bg text,
  deskripsi text,
  gambar_urls text[] default '{}'
);
```

Penjelasan:

- `id` adalah id lapangan.
- `nama` adalah nama venue/lapangan.
- `lokasi` adalah lokasi venue.
- `jenis` adalah jenis olahraga, misalnya futsal atau badminton.
- `rating` adalah nilai rating.
- `harga_mulai` adalah harga yang ditampilkan.
- `harga_coret` adalah harga lama jika ada.
- `nama_pemilik` adalah nama pemilik venue.
- `warna_bg` adalah warna latar fallback jika gambar tidak tampil.
- `deskripsi` adalah deskripsi venue.
- `gambar_urls` menyimpan daftar URL gambar dari Supabase Storage.

Data tabel ini diambil di `PengontrolLapangan`:

```dart
final response = await _supabase.from('lapangan').select();
```

#### 6. Membuat Tabel `pesanan`

Tabel `pesanan` menyimpan data booking pelanggan.

Contoh struktur tabel:

```sql
create table public.pesanan (
  id uuid primary key default gen_random_uuid(),
  lapangan_id text not null references public.lapangan(id) on delete cascade,
  pengguna_id uuid not null references public.profil(id) on delete cascade,
  tanggal date not null,
  jam_mulai time not null,
  jam_selesai time not null,
  status text not null default 'menunggu',
  dibuat_pada timestamptz not null default now()
);
```

Penjelasan:

- `id` dibuat otomatis oleh database.
- `lapangan_id` menghubungkan pesanan ke tabel `lapangan`.
- `pengguna_id` menghubungkan pesanan ke tabel `profil`.
- `tanggal` adalah tanggal booking.
- `jam_mulai` adalah jam mulai booking.
- `jam_selesai` adalah jam selesai booking.
- `status` adalah status booking.
- `dibuat_pada` adalah waktu data booking dibuat.

Status yang digunakan project:

```text
menunggu
disetujui
ditolak
selesai
```

Saat pelanggan booking, kode insert ke tabel ini ada di `PengontrolPesanan`:

```dart
final response = await _supabase
    .from('pesanan')
    .insert(pesananBaru.toJson())
    .select()
    .single();
```

#### 7. Membuat Bucket Storage `lapangan_images`

Masuk ke menu:

```text
Storage -> New bucket
```

Buat bucket dengan nama:

```text
lapangan_images
```

Bucket ini dipakai untuk menyimpan foto venue/lapangan.

Di kode, bucket ini dipakai pada `PengontrolLapangan`:

```dart
await _supabase.storage.from('lapangan_images').uploadBinary(
  namaFile,
  bytes,
);
```

Lalu URL public gambar diambil:

```dart
final urlGambar = _supabase.storage
    .from('lapangan_images')
    .getPublicUrl(namaFile);
```

Jika ingin gambar bisa tampil dengan `Image.network`, bucket perlu bisa diakses publik atau dibuat policy agar file dapat dibaca.

#### 8. Mengatur Policy atau Permission

Supabase memiliki fitur keamanan bernama **Row Level Security** atau **RLS**. Jika RLS aktif tapi belum ada policy, aplikasi bisa gagal membaca atau menulis data.

Untuk kebutuhan demo/presentasi, cara paling sederhana adalah membuat policy yang mengizinkan user login membaca dan menulis data sesuai kebutuhan aplikasi.

Contoh policy sederhana untuk tabel `profil`:

```sql
alter table public.profil enable row level security;

create policy "User bisa membaca profil"
on public.profil
for select
to authenticated
using (true);

create policy "User bisa membuat profil sendiri"
on public.profil
for insert
to authenticated
with check (auth.uid() = id);

create policy "User bisa update profil sendiri"
on public.profil
for update
to authenticated
using (auth.uid() = id)
with check (auth.uid() = id);
```

Contoh policy sederhana untuk tabel `lapangan`:

```sql
alter table public.lapangan enable row level security;

create policy "Semua user login bisa membaca lapangan"
on public.lapangan
for select
to authenticated
using (true);

create policy "User login bisa mengelola lapangan"
on public.lapangan
for all
to authenticated
using (true)
with check (true);
```

Contoh policy sederhana untuk tabel `pesanan`:

```sql
alter table public.pesanan enable row level security;

create policy "User login bisa membaca pesanan"
on public.pesanan
for select
to authenticated
using (true);

create policy "User login bisa membuat pesanan"
on public.pesanan
for insert
to authenticated
with check (auth.uid() = pengguna_id);

create policy "User login bisa update pesanan"
on public.pesanan
for update
to authenticated
using (true)
with check (true);
```

Catatan penting untuk presentasi:

```text
Untuk project demo, policy bisa dibuat sederhana agar fitur berjalan.
Untuk aplikasi produksi, policy sebaiknya dibuat lebih ketat.
Misalnya hanya admin yang boleh tambah/edit/hapus lapangan,
dan pelanggan hanya boleh melihat atau membuat pesanan miliknya sendiri.
```

#### 9. Membuat Akun Admin

Project ini tidak memiliki form khusus membuat admin. Akun baru dari aplikasi otomatis menjadi pelanggan.

Kode register:

```dart
'peran': 'pelanggan',
```

Untuk membuat admin:

1. Daftar akun lewat aplikasi.
2. Buka Supabase.
3. Masuk ke tabel `profil`.
4. Cari email akun tersebut.
5. Ubah kolom `peran` dari `pelanggan` menjadi `admin`.
6. Logout dari aplikasi.
7. Login ulang.

Setelah login ulang, `main.dart` akan membaca role admin:

```dart
if (peran == 'admin') {
  return const TampilanDashboardAdmin();
}
```

#### 10. Cara Menjelaskan Konfigurasi Supabase ke Dosen

Jawaban singkat:

```text
Supabase dikonfigurasi di file konstanta.dart dengan Project URL dan anon key.
Saat aplikasi dijalankan, main.dart memanggil Supabase.initialize untuk menghubungkan Flutter ke Supabase.
Authentication Supabase dipakai untuk login dan register.
Tabel profil menyimpan data user dan role.
Tabel lapangan menyimpan data venue.
Tabel pesanan menyimpan data booking.
Supabase Storage bucket lapangan_images dipakai untuk menyimpan gambar venue.
```

### Tabel `profil`

Tabel ini menyimpan data tambahan user setelah register.

Field yang dipakai kode:

```text
id
nama_lengkap
email
peran
```

Contoh data:

```text
id            : id user dari Supabase Auth
nama_lengkap  : Andi
email         : andi@email.com
peran         : pelanggan
```

Untuk membuat akun admin, nilai `peran` diubah menjadi:

```text
admin
```

### Tabel `lapangan`

Tabel ini menyimpan data venue/lapangan.

Field yang dipakai kode:

```text
id
nama
lokasi
jenis
rating
harga_mulai
harga_coret
nama_pemilik
warna_bg
deskripsi
gambar_urls
```

Catatan:

- `gambar_urls` berisi list URL gambar.
- `warna_bg` disimpan sebagai string warna, misalnya `#1A6B3A`.
- Data ini dibaca oleh `PengontrolLapangan`.

### Tabel `pesanan`

Tabel ini menyimpan data booking.

Field yang dipakai kode:

```text
id
lapangan_id
pengguna_id
tanggal
jam_mulai
jam_selesai
status
dibuat_pada
```

Status booking:

```text
menunggu
disetujui
ditolak
selesai
```

### Storage Bucket `lapangan_images`

Bucket ini dipakai untuk menyimpan gambar venue yang di-upload admin.

Kode upload ada di `PengontrolLapangan`:

```dart
await _supabase.storage.from('lapangan_images').uploadBinary(
  namaFile,
  bytes,
);

final urlGambar = _supabase.storage
    .from('lapangan_images')
    .getPublicUrl(namaFile);
```

Alurnya:

1. Admin memilih gambar.
2. File gambar dibaca menjadi bytes.
3. Bytes di-upload ke bucket `lapangan_images`.
4. Supabase memberi public URL.
5. URL disimpan ke database `lapangan`.

## 8. Model Data

Model adalah class yang mewakili bentuk data dari database.

### Model `Pengguna`

File: `lib/model/pengguna.dart`

```dart
class Pengguna {
  final String id;
  final String namaLengkap;
  final String email;
  final String peran;
}
```

Model ini mewakili data user dari tabel `profil`.

Fungsi `fromJson()` mengubah data dari Supabase menjadi object Dart:

```dart
factory Pengguna.fromJson(Map<String, dynamic> json) {
  return Pengguna(
    id: json['id'],
    namaLengkap: json['nama_lengkap'],
    email: json['email'],
    peran: json['peran'] ?? 'pelanggan',
  );
}
```

Jika `peran` kosong, default-nya adalah `pelanggan`.

### Model `Lapangan`

File: `lib/model/lapangan.dart`

Model ini menyimpan data venue/lapangan.

```dart
class Lapangan {
  final String id;
  final String nama;
  final String lokasi;
  final String jenis;
  final double rating;
  final String hargaMulai;
  final String? hargaCoret;
  final String namaPemilik;
  final String? warnaBg;
  final String? deskripsi;
  final List<String> gambarUrls;
}
```

Fungsi `fromJson()` membaca field dari Supabase, seperti `harga_mulai`, `harga_coret`, dan `gambar_urls`.

Fungsi `toJson()` mengubah object `Lapangan` menjadi format yang bisa dikirim ke Supabase.

### Model `Pesanan`

File: `lib/model/pesanan.dart`

Model ini menyimpan data booking.

```dart
class Pesanan {
  final String id;
  final String lapanganId;
  final String penggunaId;
  final String tanggal;
  final String jamMulai;
  final String jamSelesai;
  String status;
  Lapangan? lapangan;
}
```

Field `lapangan` bersifat optional karena kadang data pesanan diambil bersamaan dengan data lapangan melalui join:

```dart
.select('*, lapangan(*)')
```

## 9. Controller Otentikasi

File: `lib/pengontrol/pengontrol_otentikasi.dart`

Controller ini mengatur login, register, logout, profil, dan password.

### State yang Disimpan

```dart
Pengguna? _penggunaSaatIni;
```

Jika `_penggunaSaatIni` tidak null, berarti user sudah login.

```dart
bool get apakahSudahMasuk => _penggunaSaatIni != null;
```

### Inisialisasi Session

Saat controller dibuat, constructor menjalankan:

```dart
PengontrolOtentikasi() {
  _inisialisasiSesi();
}
```

Lalu:

```dart
final session = _supabase.auth.currentSession;
if (session != null) {
  await _ambilDataProfil(session.user.id);
}
```

Artinya jika Supabase masih menyimpan session login, aplikasi langsung mengambil profil user.

### Login

```dart
final response = await _supabase.auth.signInWithPassword(
  email: email,
  password: kataSandi,
);
```

Jika login berhasil:

```dart
await _ambilDataProfil(response.user!.id);
```

Data profil diambil dari tabel `profil`.

### Register

```dart
final response = await _supabase.auth.signUp(
  email: email,
  password: kataSandi,
);
```

Setelah user dibuat di Supabase Auth, kode memasukkan data ke tabel `profil`:

```dart
await _supabase.from('profil').insert({
  'id': response.user!.id,
  'nama_lengkap': namaLengkap,
  'email': email,
  'peran': 'pelanggan',
});
```

Jadi akun baru otomatis menjadi pelanggan.

### Update Profil

Jika email berubah, Supabase Auth juga di-update:

```dart
await _supabase.auth.updateUser(UserAttributes(email: emailBaru));
```

Lalu tabel `profil` di-update:

```dart
await _supabase.from('profil').update({
  'nama_lengkap': namaBaru,
  'email': emailBaru,
}).eq('id', _penggunaSaatIni!.id);
```

### Update Password

```dart
await _supabase.auth.updateUser(UserAttributes(password: passwordBaru));
```

### Logout

```dart
await _supabase.auth.signOut();
_penggunaSaatIni = null;
notifyListeners();
```

Setelah logout, `main.dart` akan mendeteksi user tidak login dan kembali ke halaman masuk.

## 10. Controller Lapangan

File: `lib/pengontrol/pengontrol_lapangan.dart`

Controller ini mengatur semua data venue/lapangan.

### Mengambil Daftar Lapangan

```dart
final response = await _supabase.from('lapangan').select();
```

Data dari Supabase diubah menjadi list `Lapangan`:

```dart
_daftarLapangan = (response as List<dynamic>)
    .map((json) => Lapangan.fromJson(json))
    .toList();
```

Setelah itu:

```dart
notifyListeners();
```

Maka halaman yang menampilkan daftar venue akan update.

### Getter `daftarVenue`

Kode UI memakai `Map<String, dynamic>`, sedangkan controller menyimpan data dalam bentuk `Lapangan`.

Karena itu ada getter:

```dart
List<Map<String, dynamic>> get daftarVenue
```

Getter ini mengubah list `Lapangan` menjadi list `Map` agar cocok dengan UI yang sudah dibuat.

### Tambah Venue

```dart
await _supabase.from('lapangan').insert(lap.toJson());
await ambilDaftarLapangan();
```

Setelah insert, controller mengambil ulang daftar lapangan agar UI terbaru.

### Edit Venue

```dart
await _supabase.from('lapangan').update(lap.toJson()).eq('id', id);
await ambilDaftarLapangan();
```

### Hapus Venue

```dart
await _supabase.from('lapangan').delete().eq('id', id);
await ambilDaftarLapangan();
```

### Upload Gambar

```dart
final bytes = await file.readAsBytes();
await _supabase.storage.from('lapangan_images').uploadBinary(
  namaFile,
  bytes,
);
```

Setelah upload, URL gambar diambil:

```dart
final urlGambar = _supabase.storage
    .from('lapangan_images')
    .getPublicUrl(namaFile);
```

## 11. Controller Pesanan

File: `lib/pengontrol/pengontrol_pesanan.dart`

Controller ini mengatur booking user.

### State yang Disimpan

```dart
List<Pesanan> riwayatPesanan = [];
Pesanan? pesananAktif;
int sisaWaktuDetik = 0;
Timer? _pengaturWaktu;
```

Penjelasan:

- `riwayatPesanan` menyimpan daftar booking user.
- `pesananAktif` menyimpan booking yang baru dibuat/aktif.
- `sisaWaktuDetik` dipakai untuk countdown waktu bermain.
- `_pengaturWaktu` adalah timer yang berjalan setiap detik.

### Mengambil Riwayat Pesanan

```dart
final user = _supabase.auth.currentUser;
if (user == null) return;
```

Kode memastikan hanya user login yang bisa mengambil riwayat.

Query:

```dart
final response = await _supabase
    .from('pesanan')
    .select('*, lapangan(*)')
    .eq('pengguna_id', user.id)
    .order('dibuat_pada', ascending: false);
```

Penjelasan:

- Ambil dari tabel `pesanan`.
- Join dengan tabel `lapangan`.
- Filter hanya pesanan milik user login.
- Urutkan berdasarkan waktu dibuat.

### Membuat Pesanan

Fungsi:

```dart
Future<void> buatPesanan(
  Map<String, dynamic> lapanganData,
  DateTime tanggal,
  TimeOfDay jamMulai,
  TimeOfDay jamSelesai,
)
```

Langkahnya:

1. Cek user login.
2. Format tanggal menjadi `yyyy-MM-dd`.
3. Format jam menjadi `HH:mm:ss`.
4. Buat object `Pesanan`.
5. Insert ke tabel `pesanan`.
6. Simpan sebagai `pesananAktif`.
7. Hitung durasi bermain.
8. Jalankan timer countdown.
9. Refresh riwayat pesanan.

Kode insert:

```dart
final response = await _supabase
    .from('pesanan')
    .insert(pesananBaru.toJson())
    .select()
    .single();
```

Status awal pesanan:

```dart
status: 'menunggu',
```

### Timer Pesanan

Timer berjalan setiap 1 detik:

```dart
_pengaturWaktu = Timer.periodic(const Duration(seconds: 1), (timer) {
  if (sisaWaktuDetik > 0) {
    sisaWaktuDetik--;
    notifyListeners();
  } else {
    selesaikanPesananOtomatis();
  }
});
```

Jika waktu habis, status pesanan diubah menjadi `selesai`.

### Menyelesaikan Pesanan Otomatis

```dart
await _supabase
    .from('pesanan')
    .update({'status': 'selesai'})
    .eq('id', pesananAktif!.id);
```

## 12. Halaman Login

File: `lib/tampilan/tampilan_masuk.dart`

Halaman ini berisi input email dan password.

Ketika tombol `MASUK` ditekan:

```dart
await context.read<PengontrolOtentikasi>().masuk(
  _emailPengontrol.text,
  _kataSandiPengontrol.text,
);
```

Jika login berhasil, controller memanggil `notifyListeners()`. Karena `main.dart` memakai `Consumer<PengontrolOtentikasi>`, aplikasi otomatis berpindah halaman.

## 13. Halaman Daftar

File: `lib/tampilan/tampilan_daftar.dart`

Halaman ini berisi input:

- Nama lengkap
- Email
- Password

Saat tombol `DAFTAR` ditekan:

```dart
await context.read<PengontrolOtentikasi>().daftar(
  _emailPengontrol.text,
  _kataSandiPengontrol.text,
  _namaPengontrol.text,
);
```

Akun baru akan disimpan di Supabase Auth dan tabel `profil`.

## 14. Halaman Utama Pelanggan

File: `lib/tampilan/tampilan_utama.dart`

Halaman ini menampilkan daftar venue.

Data venue diambil dari:

```dart
final daftarVenue = context.watch<PengontrolLapangan>().daftarVenue;
```

Karena memakai `watch`, jika admin menambah/mengubah/menghapus venue dan controller refresh data, UI bisa ikut berubah.

### Fitur Search

Search disimpan di:

```dart
String _kataKunciCari = '';
```

Filtering:

```dart
daftarVenue.where((venue) =>
  venue['nama'].toString().toLowerCase().contains(_kataKunciCari.toLowerCase()) ||
  venue['jenis'].toString().toLowerCase().contains(_kataKunciCari.toLowerCase()) ||
  venue['lokasi'].toString().toLowerCase().contains(_kataKunciCari.toLowerCase()))
```

Jadi venue bisa dicari berdasarkan:

- Nama
- Jenis olahraga
- Lokasi

### Klik Venue

Jika card venue diklik:

```dart
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (...) => TampilanDetailVenue(venue: venue),
  ),
);
```

User masuk ke halaman detail venue.

## 15. Halaman Detail Venue

File: `lib/tampilan/tampilan_detail_venue.dart`

Halaman ini menampilkan:

- Gambar venue
- Nama venue
- Lokasi
- Deskripsi
- Tombol `Pilih Jadwal`

Jika tombol `Pilih Jadwal` ditekan:

```dart
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => TampilanPilihJadwal(venue: venue),
  ),
);
```

## 16. Halaman Pilih Jadwal

File: `lib/tampilan/tampilan_pilih_jadwal.dart`

Halaman ini digunakan untuk booking venue.

State yang dipakai:

```dart
DateTime? _tanggalDipilih;
TimeOfDay? _jamMulaiDipilih;
TimeOfDay? _jamSelesaiDipilih;
bool _sedangMemproses = false;
```

### Memilih Tanggal

Menggunakan `showDatePicker`:

```dart
final DateTime? hasil = await showDatePicker(
  context: context,
  initialDate: sekarang,
  firstDate: sekarang,
  lastDate: sekarang.add(const Duration(days: 30)),
);
```

Artinya user hanya bisa booking mulai hari ini sampai 30 hari ke depan.

### Memilih Jam

Menggunakan `showTimePicker`:

```dart
final TimeOfDay? hasil = await showTimePicker(
  context: context,
  initialTime: waktuAwal,
);
```

### Validasi Booking

Sebelum booking dibuat, kode mengecek:

```dart
if (_tanggalDipilih == null ||
    _jamMulaiDipilih == null ||
    _jamSelesaiDipilih == null) {
  ...
}
```

Lalu memastikan jam selesai lebih besar dari jam mulai:

```dart
if (selesaiMenit <= mulaiMenit) {
  ...
}
```

Jika valid:

```dart
await context.read<PengontrolPesanan>().buatPesanan(
  widget.venue,
  _tanggalDipilih!,
  _jamMulaiDipilih!,
  _jamSelesaiDipilih!,
);
```

## 17. Halaman Profil

File: `lib/tampilan/tampilan_profil.dart`

Halaman ini menampilkan:

- Nama user
- Username dari email
- Tombol ubah profil
- Menu notifikasi
- Menu ubah password
- Menu syarat dan ketentuan
- Menu kebijakan privasi
- Tombol sign out

Data user diambil dari:

```dart
final auth = context.watch<PengontrolOtentikasi>();
final pengguna = auth.penggunaSaatIni;
```

Jika tombol sign out ditekan:

```dart
context.read<PengontrolOtentikasi>().keluar();
```

## 18. Halaman Ubah Profil

File: `lib/tampilan/tampilan_ubah_profil.dart`

Halaman ini berisi form untuk mengubah:

- Nama lengkap
- Email

Saat disimpan:

```dart
await context.read<PengontrolOtentikasi>().perbaruiProfil(
  _namaController.text,
  _emailController.text,
);
```

Validasi:

- Nama tidak boleh kosong.
- Email tidak boleh kosong.
- Email harus mengandung `@`.

## 19. Halaman Ubah Password

File: `lib/tampilan/tampilan_ubah_password.dart`

Halaman ini berisi:

- Password baru
- Konfirmasi password baru

Validasi:

- Password tidak boleh kosong.
- Password minimal 6 karakter.
- Konfirmasi password harus sama.

Saat disimpan:

```dart
await context.read<PengontrolOtentikasi>().perbaruiPassword(
  _passwordController.text,
);
```

## 20. Halaman Notifikasi

File: `lib/tampilan/tampilan_notifikasi.dart`

Notifikasi dibuat dari riwayat pesanan user.

```dart
final riwayat = context.watch<PengontrolPesanan>().riwayatPesanan;
```

Lalu difilter:

```dart
riwayat.where((p) => p.status == 'disetujui' || p.status == 'ditolak')
```

Artinya notifikasi muncul jika booking sudah disetujui atau ditolak admin.

Catatan penting: notifikasi di project ini belum memakai push notification. Notifikasi dibuat dari data pesanan yang sudah ada di aplikasi.

## 21. Dashboard Admin

File: `lib/admin/tampilan_dashboard_admin.dart`

Dashboard admin menampilkan:

- Total lapangan.
- Tombol tambah venue.
- Daftar venue.
- Tombol edit venue.
- Tombol hapus venue.
- Tombol logout.

Data venue:

```dart
final daftarVenue = context.watch<PengontrolLapangan>().daftarVenue;
```

Jika klik tombol tambah:

```dart
Navigator.push(
  context,
  MaterialPageRoute(builder: (_) => const TampilanFormLapangan()),
);
```

Jika klik edit:

```dart
Navigator.push(
  context,
  MaterialPageRoute(builder: (_) => TampilanFormLapangan(venue: venue)),
);
```

Jika klik hapus:

```dart
context.read<PengontrolLapangan>().hapusVenue(id);
```

Jika klik card venue:

```dart
Navigator.push(
  context,
  MaterialPageRoute(builder: (_) => TampilanDaftarBookingAdmin(venue: venue)),
);
```

Admin akan masuk ke daftar booking untuk venue tersebut.

## 22. Form Tambah/Edit Lapangan

File: `lib/admin/tampilan_form_lapangan.dart`

Halaman ini dipakai untuk dua fungsi:

1. Tambah lapangan.
2. Edit lapangan.

Pembedanya:

```dart
final isEdit = widget.venue != null;
```

Jika `widget.venue == null`, berarti tambah venue baru.

Jika `widget.venue != null`, berarti edit venue lama.

Input form:

- Nama venue
- Lokasi
- Jenis olahraga
- Deskripsi lapangan
- Nama pemilik
- Foto lapangan

### Memilih Gambar

```dart
final ImagePicker picker = ImagePicker();
final List<XFile> images = await picker.pickMultiImage();
```

Admin bisa memilih banyak gambar.

### Simpan Data

Jika ada gambar baru, gambar di-upload dulu:

```dart
urlsBaru = await context
    .read<PengontrolLapangan>()
    .unggahGambar(_gambarDipilih);
```

Lalu semua gambar digabung:

```dart
final List<String> semuaGambar = [..._gambarLamaUrls, ...urlsBaru];
```

Jika tambah:

```dart
await context.read<PengontrolLapangan>().tambahVenue(dataBaru);
```

Jika edit:

```dart
await context.read<PengontrolLapangan>().editVenue(
  widget.venue!['id'],
  dataBaru,
);
```

## 23. Daftar Booking Admin

File: `lib/admin/tampilan_daftar_booking_admin.dart`

Halaman ini menampilkan semua booking pada satu venue.

Query:

```dart
final response = await _supabase
    .from('pesanan')
    .select('*, profil:pengguna_id(nama_lengkap, email)')
    .eq('lapangan_id', widget.venue['id'])
    .order('tanggal', ascending: false)
    .order('jam_mulai', ascending: true);
```

Penjelasan:

- Ambil data dari tabel `pesanan`.
- Join ke tabel `profil` untuk mengambil nama dan email user.
- Filter berdasarkan `lapangan_id`.
- Urutkan berdasarkan tanggal dan jam mulai.

### Mengubah Status Booking

Fungsi:

```dart
Future<void> _ubahStatus(String idPesanan, String statusBaru) async
```

Query update:

```dart
await _supabase
    .from('pesanan')
    .update({'status': statusBaru})
    .eq('id', idPesanan);
```

Jika status `menunggu`, admin bisa:

- Setujui -> `disetujui`
- Tolak -> `ditolak`

Jika status `disetujui`, admin bisa:

- Tandai selesai -> `selesai`

## 24. Cara Login Sebagai Admin

Saat daftar lewat aplikasi, akun otomatis dibuat sebagai pelanggan:

```dart
'peran': 'pelanggan',
```

Untuk menjadi admin:

1. Daftar akun seperti biasa.
2. Buka Supabase.
3. Masuk ke tabel `profil`.
4. Cari akun berdasarkan email.
5. Ubah kolom `peran` dari `pelanggan` menjadi `admin`.
6. Logout dari aplikasi.
7. Login lagi.

Setelah login ulang, kode di `main.dart` akan mendeteksi:

```dart
if (peran == 'admin') {
  return const TampilanDashboardAdmin();
}
```

Maka user masuk ke dashboard admin.

## 25. Alur Booking dari Pelanggan sampai Admin

Alur lengkapnya:

1. Pelanggan login.
2. Aplikasi membuka `TampilanUtama`.
3. Pelanggan memilih venue.
4. Aplikasi membuka `TampilanDetailVenue`.
5. Pelanggan klik `Pilih Jadwal`.
6. Aplikasi membuka `TampilanPilihJadwal`.
7. Pelanggan memilih tanggal, jam mulai, dan jam selesai.
8. Kode melakukan validasi.
9. Jika valid, `PengontrolPesanan.buatPesanan()` dipanggil.
10. Data masuk ke tabel `pesanan` dengan status `menunggu`.
11. Admin login.
12. Admin membuka dashboard.
13. Admin memilih venue.
14. Admin melihat daftar booking.
15. Admin menekan `Setujui` atau `Tolak`.
16. Status pesanan berubah di Supabase.
17. Pelanggan melihat notifikasi berdasarkan status pesanan.

## 26. Alur Tambah Venue oleh Admin

Alurnya:

1. Admin login.
2. Admin masuk ke dashboard.
3. Admin klik `Tambah Venue`.
4. Admin mengisi form.
5. Admin memilih gambar.
6. Gambar di-upload ke Supabase Storage.
7. URL gambar disimpan ke field `gambar_urls`.
8. Data venue disimpan ke tabel `lapangan`.
9. Controller mengambil ulang daftar lapangan.
10. Dashboard admin dan halaman pelanggan bisa menampilkan venue baru.

## 27. Hal yang Perlu Diperhatikan Saat Presentasi

Beberapa poin penting yang kemungkinan ditanya:

### Apa fungsi `main.dart`?

`main.dart` adalah entry point aplikasi. File ini menginisialisasi Supabase, memasang Provider, mengatur tema, localization, dan menentukan halaman awal berdasarkan status login dan role user.

### Apa fungsi `konstanta.dart`?

File ini menyimpan URL dan anon key Supabase agar konfigurasi Supabase tidak ditulis langsung di banyak file.

### Apa fungsi folder `model`?

Folder `model` berisi class data seperti `Pengguna`, `Lapangan`, dan `Pesanan`. Model memudahkan konversi data dari Supabase ke object Dart dan sebaliknya.

### Apa fungsi folder `pengontrol`?

Folder `pengontrol` berisi logic aplikasi. Misalnya login, register, ambil data lapangan, tambah venue, buat booking, dan update status pesanan.

### Apa fungsi folder `tampilan`?

Folder `tampilan` berisi UI untuk pelanggan.

### Apa fungsi folder `admin`?

Folder `admin` berisi UI untuk admin, seperti dashboard, form lapangan, dan daftar booking.

### Mengapa memakai Supabase?

Supabase menyediakan backend cepat untuk:

- Authentication
- Database PostgreSQL
- Storage gambar
- API otomatis

Jadi Flutter tidak perlu membuat backend manual.

### Mengapa memakai Provider?

Provider memudahkan state management. Contohnya, setelah user login, data user disimpan di `PengontrolOtentikasi`, lalu halaman lain bisa membaca data itu tanpa harus mengirim manual lewat constructor.

### Bagaimana data user tersimpan?

Login utama disimpan di Supabase Auth. Data tambahan seperti nama lengkap dan role disimpan di tabel `profil`.

### Bagaimana membedakan admin dan pelanggan?

Dari kolom `peran` di tabel `profil`. Jika nilainya `admin`, aplikasi membuka dashboard admin. Jika bukan, aplikasi membuka halaman utama pelanggan.

### Bagaimana booking dibuat?

Booking dibuat saat pelanggan memilih venue, tanggal, jam mulai, dan jam selesai. Data tersebut dikirim ke tabel `pesanan` dengan status awal `menunggu`.

### Bagaimana admin menyetujui booking?

Admin membuka daftar booking venue, lalu menekan tombol `Setujui`. Kode akan mengubah field `status` di tabel `pesanan` menjadi `disetujui`.

### Apakah notifikasi menggunakan push notification?

Belum. Notifikasi di aplikasi ini dibuat dari data riwayat pesanan. Jika status pesanan `disetujui` atau `ditolak`, halaman notifikasi menampilkan pesan.

## 28. Catatan Kelebihan Project

Kelebihan project ini:

- Sudah memakai backend online Supabase.
- Sudah ada login dan register.
- Sudah ada role admin dan pelanggan.
- Sudah ada CRUD venue untuk admin.
- Sudah ada upload gambar venue.
- Sudah ada booking lapangan.
- Sudah ada update status booking oleh admin.
- UI sudah cukup lengkap dan memakai animasi.

## 29. Catatan Kekurangan atau Pengembangan Lanjutan

Jika dosen bertanya pengembangan selanjutnya, bisa jawab:

1. Menambahkan validasi agar jadwal yang sama tidak bisa dibooking dua kali.
2. Menambahkan pembayaran.
3. Menambahkan push notification.
4. Menambahkan fitur lupa password.
5. Menambahkan filter berdasarkan jenis olahraga.
6. Menambahkan rating dan review user.
7. Menambahkan keamanan Row Level Security yang lebih detail di Supabase.
8. Menambahkan laporan pemasukan untuk admin.
9. Menambahkan status pembayaran.
10. Menambahkan testing yang lebih lengkap.

## 30. Ringkasan Singkat untuk Presentasi

Kalimat yang bisa digunakan:

```text
LintasArena adalah aplikasi booking lapangan olahraga berbasis Flutter dan Supabase.
Flutter digunakan untuk membuat tampilan mobile, Provider digunakan untuk state management,
dan Supabase digunakan untuk autentikasi, database, serta penyimpanan gambar.
Aplikasi memiliki dua role, yaitu pelanggan dan admin.
Pelanggan dapat melihat venue, memilih jadwal, dan membuat booking.
Admin dapat mengelola venue dan mengubah status booking menjadi disetujui, ditolak, atau selesai.
```

## 31. File Penting yang Harus Dipahami

Jika waktu belajar terbatas, pahami file ini terlebih dahulu:

```text
lib/main.dart
lib/konstanta.dart
lib/pengontrol/pengontrol_otentikasi.dart
lib/pengontrol/pengontrol_lapangan.dart
lib/pengontrol/pengontrol_pesanan.dart
lib/model/pengguna.dart
lib/model/lapangan.dart
lib/model/pesanan.dart
lib/tampilan/tampilan_utama.dart
lib/tampilan/tampilan_pilih_jadwal.dart
lib/admin/tampilan_dashboard_admin.dart
lib/admin/tampilan_daftar_booking_admin.dart
lib/admin/tampilan_form_lapangan.dart
```

## 32. Cara Menjalankan Project

Pastikan Flutter sudah terinstall. Lalu dari root project jalankan:

```bash
flutter pub get
flutter run
```

Jika ingin mengecek error/lint:

```bash
flutter analyze
```

Jika ingin menjalankan test:

```bash
flutter test
```

## 33. Catatan Test

Project ini memiliki file test:

```text
test/widget_test.dart
```

Test tersebut adalah smoke test sederhana untuk memastikan aplikasi bisa dibangun dan halaman masuk muncul.

```dart
expect(find.text('Masuk'), findsWidgets);
```

Artinya test mencari teks `Masuk` pada UI.

## 34. Kesimpulan

Project ini memakai arsitektur sederhana:

```text
UI/Tampilan -> Controller/Pengontrol -> Supabase -> Database/Storage
```

Contoh:

```text
TampilanPilihJadwal
  -> PengontrolPesanan.buatPesanan()
  -> Supabase tabel pesanan
  -> status awal menunggu
  -> admin mengubah status
  -> pelanggan melihat notifikasi
```

Dengan memahami alur ini, kode project akan lebih mudah dijelaskan saat presentasi.
