# Pertemuan 8 — Finishing, Build, dan Deployment

## I. Tujuan

1. Mahasiswa mampu memberikan sentuhan akhir (_polishing_) pada aplikasi menggunakan ikon dan splash screen custom.
2. Mahasiswa mampu melakukan konfigurasi _signing_ aplikasi Android menggunakan Keystore.
3. Mahasiswa mampu membangun (_build_) file APK rilis yang siap didistribusikan dan diinstal pada perangkat pengguna.

---

## II. Teori

### 2.1 Identitas Aplikasi

Sebuah aplikasi yang profesional tidak hanya berfungsi dengan baik, tetapi juga harus memiliki identitas visual yang kuat. Saat kita membuat proyek Flutter baru, secara default aplikasi akan menggunakan ikon logo Flutter dan layar putih kosong saat dimuat (_splash screen_).

- **App Launcher Icon**: Ikon yang muncul di menu aplikasi (_app drawer_) atau _home screen_ pengguna. Ini adalah representasi visual pertama dari brand aplikasi kita.
- **Splash Screen**: Tampilan awal yang muncul sesaat ketika aplikasi sedang memuat engine Flutter di latar belakang. Splash screen yang baik memberikan kesan responsif dan profesional.

### 2.2 Application Signing (Keystore)

Google Play Store dan perangkat Android membutuhkan jaminan bahwa aplikasi yang diinstal adalah asli dan berasal dari pengembang yang sah. Mekanisme ini disebut _App Signing_.

- _Debug Key_: Saat kita menjalankan `flutter run`, Flutter otomatis menandatangani aplikasi dengan kunci debug yang tidak aman dan sama untuk semua developer.
- _Release Key (Keystore)_: Untuk rilis, kita harus membuat kunci kriptografi unik kita sendiri yang disimpan dalam file `.jks` (Java KeyStore).

**PENTING**: File Keystore dan password-nya harus dijaga kerahasiaannya. Jika file ini hilang, Anda TIDAK BISA memperbarui aplikasi Anda di Play Store selamanya.

---

## III. Praktik

### 3.1 Ubah nama aplikasi

`android/app/src/main/AndroidManifest.xml`

```dart
<application
   //  Ganti nama app sesuai keinginanmu
    android:label="ToDo"
    android:name="${applicationName}"
    android:icon="@mipmap/ic_launcher">
```

### 3.2 App Polish (App Icon & Splash Screen)

Kita akan menggunakan dua package populer untuk mengotomatisasi pembuatan aset gambar yang rumit untuk berbagai ukuran layar Android.

1. Siapkan assets untuk icon app dan spashscreen. Simpan pada `assets/images` kedua gambar berikut [LINK]
1. Buka terminal di dalam IDE (pastikan berada di root folder project)
1. Jalankan perintah berikut:

**Terminal**

```bash
flutter pub add flutter_launcher_icons
```

**Terminal**

```bash
flutter pub add flutter_native_splash
```

4. Pastikan sudah terdaftar pada pubspec.yaml

**`pubspec.yaml`**

```yaml
flutter_launcher_icons: ^0.14.2

flutter_native_splash: ^2.4.3
```

Setelah itu kita perlu memastikan sudah memiliki file gambar logo di `assets/images/app_icon.png` dan `assets/images/splash_screen_icon.png`. Lalu tambahkan konfigurasi berikut di bagian paling bawah pubspec.yaml.

**`pubspec.yaml`**

```yaml
flutter_launcher_icons:
  android: true
  ios: false
  image_path: "assets/images/app_icon.png"
  adaptive_icon_background: "#FFFFFF"
  adaptive_icon_foreground: "assets/images/app_icon.png"
```

Ini berfungsi untuk mengatur konfigurasi icon aplikasi.

**`pubspec.yaml`**

```yaml
flutter_native_splash:
  android: true
  ios: false
  web: false
  color: "#FFFFFF"
  image: assets/images/splash_screen_icon.png
  android_12:
    color: "#FFFFFF"
    image: assets/images/splash_screen_icon.png
```

Ini berfungsi untuk mengatur splash screen aplikasi.

Setelah itu jalankan perintah berikut untuk menghasilkan ikon dan splash screen secara otomatis.

1. Pastikan semua dependencies sudah synchronized dengan menggunakan perintah

**Terminal**

```bash
flutter pub get
```

2. Jalankan perintah ini supaya flutter dapat mengenerate app icon secara otomatis

**Terminal**

```bash
dart run flutter_launcher_icons
```

3. Jalankan perintah ini supaya flutter dapat mengatur splash screen secara otomatis

**Terminal**

```bash
dart run flutter_native_splash:create
```

### 3.3 Konfigurasi _Build Release_ (_Signing_)

Kita akan membuat brankas kunci digital dan memberitahu alat pembangun Android (Gradle) cara menggunakannya.

#### 3.3.1 Membuat Keystore

Jalankan perintah ini di terminal (pastikan Anda berada di root folder proyek). Perintah ini menggunakan alat keytool bawaan Java.

**Terminal**

```bash
keytool -genkey -v -keystore android/app/upload-keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

**Masukkan passwordnya `Admin123`**

File `upload-keystore.jks` akan tercipta di folder `android/app/`. Jangan pernah menghapus atau memindahkan file ini sembarangan.

#### 3.3.2 Mengamankan Kredensial

Kita tidak boleh menulis password langsung di dalam file build.gradle karena file tersebut akan di-upload ke Git. Kita gunakan file properti terpisah yang nantinya diabaikan oleh Git.

Buat file `key.properties` dalam folder `android`, lalu isi sesuai dengan codesnap berikut

**`key.properties`**

```properties
storePassword=Admin123
keyPassword=Admin123
keyAlias=upload
storeFile=upload-keystore.jks
```

Setelah itu pastikan file `.gitignore` di folder `android/` sudah mencantumkan `key.properties` dan `*.jks` agar file rahasia ini tidak ikut ter-_commit_.

**`.gitignore`**

```text
gradle-wrapper.jar
/.gradle
/captures/
/gradlew
/gradlew.bat
/local.properties
GeneratedPluginRegistrant.java
.cxx/

# Remember to never publicly share your keystore.
# See https://flutter.dev/to/reference-keystore
key.properties
**/*.keystore
**/*.jks
```

#### 3.3.3 Konfigurasi build.gradle

Kita perlu menginstruksikan Gradle untuk membaca file key.properties dan menggunakannya saat mode Rilis.

1. Buka `android/app/build.gradle` (Level App).
2. Tambahkan kode ini sebelum blok `android {}`:

**`build.gradle.kts`**

```kotlin
val keystorePropertiesFile = rootProject.file("key.properties")
val keystoreProperties = Properties()
if (keystorePropertiesFile.exists()) {
    FileInputStream(keystorePropertiesFile).use {
        keystoreProperties.load(it)
    }
}
```

3. Buat Konfigurasi Signing di dalam blok `android {}`, sebelum buildTypes

**`build.gradle.kts`**

```kotlin
defaultConfig {
    applicationId = ""com.example.modul_6"
    minSdk = flutter.minSdkVersion
    targetSdk = flutter.targetSdkVersion
    versionCode = flutter.versionCode
    versionName = flutter.versionName
}
```

4. Sebelum `buildTypes{}` tambahkan `signingConfigs{}` berikut

```dart
    signingConfigs {
        create("release") {
            keyAlias = keystoreProperties["keyAlias"] as String
            keyPassword = keystoreProperties["keyPassword"] as String
            storeFile = file(keystoreProperties["storeFile"] as String)
            storePassword = keystoreProperties["storePassword"] as String
        }
    }
```

4. Terapkan Config di dalam `buildTypes { release {} }`

**`build.gradle.kts`**

```kotlin
buildTypes {
    release {
        signingConfig = signingConfigs.getByName("release")
    }
}
```

### 3.4 Finalisasi Firebase & Build APK

Masalah umum yang sering terjadi ketika akan melakukan deployment aplikasi adalah Aplikasi jalan lancar saat debug, tapi Login Google gagal total setelah jadi APK. Penyebabnya yaitu _Fingerprint_ (SHA-1) aplikasi berubah saat kita mengganti kunci dari Debug ke Release. Google menganggap APK rilis tersebut sebagai aplikasi asing.

#### 3.4.1 Mendapatkan _Release_ SHA-1

Jalankan perintah ini untuk melihat _fingerprint_ dari _Keystore_ yang baru:

- Masuk ke folder android: ketik **cd android**

**Terminal**

```bash
cd android
```

- Jalankan perintah: `./gradlew signingReport` (Mac/Linux) atau `.\gradlew signingReport` (Windows).

**Terminal (Mac/Linux)**

```bash
./gradlew signingReport
```

**Terminal (Windows)**

```powershell
.\gradlew signingReport
```

- Terminal akan menampilkan daftar panjang. Cari blok yang berjudul Variant: release. Salin kode SHA-1 dan SHA-256 dari blok tersebut.

#### 3.4.2 Update Firebase Console

Untuk update firebase console, tahapannya sama seperti saat kita mendaftarkan fingerprint baru saat mengurus Firebase Authentication.

1. Buka Project Settings Firebase.
2. Di bagian aplikasi Android, klik Add fingerprint.
3. Masukkan SHA-1 dan SHA-256 dari varian Release tadi.
4. Sekarang Anda punya 2 set sidik jari (satu untuk Debug, satu untuk Release). Jangan hapus yang Debug agar Anda tetap bisa mengembangkan aplikasi.
5. Download google-services.json yang baru (sebenarnya isinya hanya update daftar hash, tapi lebih aman diganti).
6. Timpa file `android/app/google-services.json` lama dengan yang baru.

#### 3.4.3 Build APK

Untuk melakukan build APK, jalankan perintah berikut dalam terminal:

**Terminal**

```bash
flutter build apk --release
```

APK akan berada di `build/app/outputs/flutter-apk/app-release.apk`. Ukurannya akan jauh lebih kecil dibanding versi debug (biasanya 15-50 MB vs 100+ MB).

#### 3.4.4 Github
Buat repository github baru, lakukan push project ke github melalui terminal

### 3.5 CI/CD dengan GitHub Actions

Di dunia kerja, kita jarang melakukan build manual di laptop sendiri. Kita menggunakan layanan otomatisasi seperti GitHub Actions.

#### 3.5.1 Membuat Workflow File

Kita perlu memberikan instruksi kepada GitHub Actions tentang apa yang harus dilakukan.

1. Di root project, buat folder `.github/workflows/`.
2. Buat file baru bernama `build-release.yml`.
3. Salin kode berikut ke dalamnya:

**`build-release.yml`**

```yaml
name: Build and Release APK

on:
  push:
    tags:
      - "v*"
  workflow_dispatch:

permissions:
  contents: write

jobs:
  build:
    name: Build Flutter APK
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.get_version.outputs.version }}

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: "temurin"
          java-version: "17"

      - name: Setup Android SDK
        uses: android-actions/setup-android@v3
        with:
          packages: ""

      - name: Accept Android SDK licenses
        run: yes | sdkmanager --licenses || true

      - name: Install NDK
        run: sdkmanager --install "ndk;29.0.13113456"

      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: "3.47.0"
          channel: "stable"
          cache: true

      - name: Install Dependencies
        run: flutter pub get

      - name: Decode Keystore
        env:
          KEYSTORE_BASE64: ${{ secrets.KEYSTORE_BASE64 }}
        run: |
          echo "$KEYSTORE_BASE64" | base64 --decode > android/app/upload-keystore.jks

      - name: Create key.properties
        env:
          STORE_PASSWORD: ${{ secrets.STORE_PASSWORD }}
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
        run: |
          cat > android/key.properties << EOF
          storePassword=$STORE_PASSWORD
          keyPassword=$KEY_PASSWORD
          keyAlias=$KEY_ALIAS
          storeFile=upload-keystore.jks
          EOF

      - name: Decode google-services.json
        env:
          GOOGLE_SERVICES_JSON_BASE64: ${{ secrets.GOOGLE_SERVICES_JSON_BASE64 }}
        run: |
          echo "$GOOGLE_SERVICES_JSON_BASE64" | base64 --decode > android/app/google-services.json

      - name: Build Release APK
        run: flutter build apk --release

      - name: Get Version
        id: get_version
        run: |
          VERSION=$(grep '^version: ' pubspec.yaml | cut -d ' ' -f 2 | cut -d '+' -f 1)
          echo "version=$VERSION" >> $GITHUB_OUTPUT

      - name: Upload APK Artifact
        uses: actions/upload-artifact@v4
        with:
          name: todo-${{ steps.get_version.outputs.version }}
          path: build/app/outputs/flutter-apk/app-release.apk

  release:
    name: Create GitHub Release
    needs: build
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/v')

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Download APK
        uses: actions/download-artifact@v4
        with:
          name: todo-${{ needs.build.outputs.version }}
          path: ./

      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          files: app-release.apk
          body: |
            ## ToDo v${{ needs.build.outputs.version }}

            ### 📱 Download
            Download the APK below and install on your Android device.

            ### ✨ What's New
            - Automated build via GitHub Actions
            - Release signed with production keystore

            ### 🔒 Security
            - Signed with release keystore
            - Firebase configured for production
            - Firestore security rules enabled
          draft: false
          prerelease: false
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

#### 3.5.2 Encode Keystore & google-services.json

GitHub Secrets hanya bisa menyimpan teks, sedangkan file Keystore adalah biner. Kita harus mengubahnya menjadi teks Base64.

1. Jalankan perintah berikut di terminal masing-masing:

   Powershell:

**Terminal (PowerShell)**

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("android/app/upload-keystore.jks")) | Out-File -FilePath keystore-base64.txt
```

**Terminal (PowerShell)**

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("D:\Semester 7\IKLC_Flutter\modul\anime_verse\android\app\google-services.json")) | Out-File -FilePath google-services-base64.txt
```

2. Buka file `keystore-base64.txt` & `google-services-base64.txt` yang baru terbentuk. Anda akan melihat deretan teks acak yang sangat panjang. **Jangan share teks ini ke siapapun!**

#### 3.5.3 Konfigurasi GitHub Secrets

Kita akan menyimpan "rahasia" aplikasi di brankas GitHub.

1. Buka repository proyek Anda di GitHub.
2. Masuk ke tab Settings -> Secrets and variables -> Actions.
3. Klik New repository secret.
4. Tambahkan 5 secret berikut persis sesuai namanya (Case Sensitive):

| Nama Secret                   | Isi                                                         |
| ----------------------------- | ----------------------------------------------------------- |
| `KEYSTORE_BASE64`             | Salin seluruh isi teks dari file keystore-base64.txt        |
| `STORE_PASSWORD`              | Password keystore yang Anda buat di Langkah 3.3             |
| `KEY_PASSWORD`                | Password key (biasanya sama dengan store password)          |
| `KEY_ALIAS`                   | upload (atau alias yang Anda gunakan di Langkah 3.1)        |
| `GOOGLE_SERVICES_JSON_BASE64` | Salin seluruh isi teks dari file google-services-base64.txt |

5. Hapus file `keystore-base64.txt` dan `google-services-base64.txt`

#### 3.5.4 Trigger Build Otomatis

Lakukan `git add`, `git commit`, kemudian `git tag v1.0.0` lalu `git push origin v1.0.0`. Anda akan melihat sebuah workflow run sedang berjalan (biasanya berwarna kuning). Tunggu hingga proses selesai (hijau). Periksa apakah ada tag baru di tab release.

**Terminal**

```bash
git add .
git commit -m "Build and Release APK"
git push origin main

git tag v1.0.0
git push origin v1.0.0
```
