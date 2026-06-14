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

Artinya notifikasi muncul jika booking sudah disetujui atau 