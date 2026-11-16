# LAMA STAR — Flutter Voice Chat Room Starter

Starter project Android untuk aplikasi **LAMA STAR**. Proyek ini berisi UI awal, mode demo, koneksi Firebase Authentication/Cloud Firestore/Cloud Functions, dan integrasi Agora RTC dengan token AccessToken2 dari server.

> **Status:** ini adalah source code starter, bukan APK final yang sudah ditandatangani. Mode demo dapat digunakan untuk melihat alur dan UI. Suara realtime memerlukan proyek Firebase, proyek Agora, secret server, dan build Android pada komputer yang memiliki Flutter SDK.

## Isi yang sudah tersedia

- Halaman login dan daftar dengan Email/Password Firebase.
- Tombol **Coba Mode Demo** untuk membuka aplikasi tanpa Firebase.
- Beranda, daftar room, filter room, profil, dan saldo tampilan coin/diamond.
- Membuat room publik atau privat. Password room di-hash di server dan hash disimpan di koleksi server-only.
- Bergabung/keluar room dan daftar anggota melalui Cloud Functions/Firestore.
- Layout voice room, kontrol mute/unmute, Raise Hand, hadiah demo, daftar anggota, serta tampilan Admin Panel awal.
- Agora RTC join channel, audio publishing, dan penerbitan token AccessToken2 di Cloud Functions.
- Firestore Security Rules dasar.

## 1. Siapkan komputer

Pasang Flutter SDK, Android Studio, Android SDK, Node.js 22, dan Firebase CLI. Periksa instalasi Flutter:

```bash
flutter doctor
```

## 2. Buat folder platform Android

Ekstrak ZIP, buka terminal di folder `lama_star_app`, lalu jalankan:

```bash
flutter create --platforms=android --org com.lamastar .
flutter pub get
```

Perintah ini membuat folder Android native yang diperlukan Flutter. Jika Flutter meminta konfirmasi untuk berkas yang sudah ada, pertahankan source `lib/` dan `pubspec.yaml` dari paket LAMA STAR.

Tambahkan izin berikut ke `android/app/src/main/AndroidManifest.xml` sebagai anak langsung dari tag `<manifest>`, sebelum `<application>`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.BLUETOOTH" android:maxSdkVersion="30" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
```

Cuplikan ini juga tersedia di `android_config/AndroidManifest.permissions.xml`.

## 3. Jalankan mode demo

Tanpa Firebase, jalankan:

```bash
flutter run
```

Pada halaman awal, tekan **COBA MODE DEMO**. Mode demo memperlihatkan alur UI menggunakan data contoh. Mode ini tidak menghubungkan audio ke pengguna lain, tidak menyimpan room ke cloud, dan transaksi hadiah tidak aktif.

## 4. Hubungkan Firebase

Buat proyek di Firebase Console. Tambahkan aplikasi Android dengan package name:

```text
com.lamastar.lama_star
```

Aktifkan:

1. Authentication → Sign-in method → Email/Password.
2. Cloud Firestore → Create database.
3. Cloud Functions (gunakan paket Blaze jika dibutuhkan untuk deployment layanan server).

Install Firebase CLI dan FlutterFire CLI, lalu dari folder proyek jalankan:

```bash
firebase login
dart pub global activate flutterfire_cli
flutterfire configure
```

`flutterfire configure` akan mengganti file placeholder `lib/firebase_options.dart` dengan konfigurasi proyek Firebase Anda. Jangan menghapus file hasil konfigurasi tersebut.

Pilih proyek Firebase yang sama untuk deployment:

```bash
firebase use --add
```

## 5. Siapkan Agora dan token server

Buat proyek di Agora Console, salin **App ID** dan aktifkan **Primary App Certificate**. Jangan memasukkan App Certificate ke Flutter, file `.env` aplikasi, atau source code Android.

Dari folder proyek:

```bash
firebase functions:secrets:set AGORA_APP_ID
firebase functions:secrets:set AGORA_APP_CERTIFICATE
firebase deploy --only firestore:rules,functions
```

Masukkan nilai masing-masing saat CLI memintanya. Secret hanya dibaca oleh Cloud Functions. Fungsi `getAgoraRtcToken` memeriksa autentikasi dan keanggotaan room sebelum menerbitkan token.

## 6. Jalankan dengan Firebase dan Agora

```bash
flutter run --dart-define=USE_FIREBASE=true
```

Pastikan Functions sudah berhasil dideploy di region `asia-southeast1`. App ID dikembalikan oleh fungsi token server kepada klien; App Certificate tetap berada di server.

## 7. Buat APK release

Setelah Firebase dan Agora terkonfigurasi:

```bash
flutter build apk --release --dart-define=USE_FIREBASE=true
```

Hasil build Flutter biasanya berada di:

```text
build/app/outputs/flutter-apk/app-release.apk
```

## Struktur data Firestore

```text
users/{uid}
  displayName, email, role, coins, diamonds, createdAt

rooms/{roomId}
  title, topic, isPrivate, ownerId, memberCount, coverEmoji, createdAt

rooms/{roomId}/members/{uid}
  uid, displayName, role, isMuted, handRaised, joinedAt

roomSecrets/{roomId}
  passwordHash, updatedAt    # server-only; Rules menolak akses klien
```

## Catatan penting sebelum dipakai publik

- Mode demo hanya simulasi UI. Jangan menganggap tombol hadiah, top up, coin, diamond, atau panel admin sebagai sistem transaksi yang sudah aktif.
- Tombol mute/raise hand dan daftar anggota memakai Firestore/Agora ketika Firebase sudah aktif. Panel admin saat ini adalah kerangka UI; operasi kick, block, pindah host, dan penetapan moderator perlu endpoint server khusus serta validasi role.
- Sebelum rilis, lengkapi pengujian pada beberapa perangkat, rate limiting, App Check, moderasi konten, pelaporan/blokir pengguna, kebijakan privasi, aturan transaksi, serta pembaruan token saat mendekati kedaluwarsa.
- Jangan pernah membagikan App Certificate Agora, password Firebase, private keystore, atau kredensial admin.

## Referensi setup

- Firebase untuk Flutter: https://firebase.google.com/docs/flutter/setup
- Agora Voice-only Flutter quickstart: https://docs.agora.io/en/realtime-media/rtc/voice-quickstart/flutter
- Agora token server / AccessToken2: https://docs.agora.io/en/realtime-media/rtc/build/authenticate-users/deploy-token-server
