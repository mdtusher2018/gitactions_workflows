# 🚀 Flutter Android CI/CD with GitHub Actions (Signed APK & AAB)

This guide helps you set up **GitHub Actions** to automatically build **signed APK and AAB files** for your Flutter app.

> ⚡ This workflow is triggered automatically on every push to the `main` branch.

---

## 📁 Project Structure

Ensure your project contains the following structure:

```
project-root/
├── .github/
│   └── workflows/
│       └── android-build.yml
├── android/
│   ├── app/
│   │   ├── build.gradle.kts
│   │   └── keystore.jks (for release Signed)
│   └── key.properties (for release Signed)
├── lib/
├── pubspec.yaml
```

---

## 🔐 Step 1: Prepare Signing Files

### 1. Create `key.properties`

Example:

```
storePassword=your_store_password
keyPassword=your_key_password
keyAlias=your_key_alias
storeFile=keystore.jks
```

### 2. Convert files to Base64

Run locally:

### Windows (PowerShell)

```
[Convert]::ToBase64String([IO.File]::ReadAllBytes("keystore.jks")) > keystore.txt
[Convert]::ToBase64String([IO.File]::ReadAllBytes("key.properties")) > key_properties.txt
```

---

## 🔑 Step 2: Add GitHub Secrets

Go to:

**Repo → Settings → Secrets and variables → Actions → New repository secret**

Add:

| Secret Name             | Value                           |
| ----------------------- | ------------------------------- |
| `KEYSTORE_BASE64`       | content of `keystore.txt`       |
| `KEY_PROPERTIES_BASE64` | content of `key_properties.txt` |

---

## ⚙️ Step 3: Configure GitHub Action

Create file:

```
.github/workflows/android-build.yml
```

Paste:

```yaml
name: Flutter - Android Build APK & App Bundle

on:
  push:
    branches: ["main"]


jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # 📥 Checkout repo
      - name: Checkout repository
        uses: actions/checkout@v4

      # ☕ Setup Java
      - name: Setup JDK
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'zulu'

      # 🐦 Setup Flutter
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: stable

      # 📦 Install dependencies
      - name: Install dependencies
        run: flutter pub get

      # 🔐 Decode keystore
      - name: Decode Keystore
        run: echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 --decode > android/app/keystore.jks

      # 🔐 Decode key.properties
      - name: Decode key.properties
        run: echo "${{ secrets.KEY_PROPERTIES_BASE64 }}" | base64 --decode > android/key.properties

      # 🧱 Build APK (Obfuscated)
      - name: Build APK
        run: flutter build apk --release --obfuscate --split-debug-info=build/debug-info

      # 📦 Build AAB (Obfuscated)
      - name: Build App Bundle
        run: flutter build appbundle --release --obfuscate --split-debug-info=build/debug-info

      # 📤 Upload APK
      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: user-apk
          path: build/app/outputs/flutter-apk/app-release.apk

      # 📤 Upload AAB
      - name: Upload User AAB
        uses: actions/upload-artifact@v4
        with:
          name: user-aab
          path: build/app/outputs/bundle/release/app-release.aab


```

---

## 🧱 Step 4: Android Signing Configuration

Inside:

```
android/app/build.gradle.kts
```

Ensure:

```kotlin
val keystoreProperties = Properties()
val keystorePropertiesFile = rootProject.file("key.properties")

if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(FileInputStream(keystorePropertiesFile))
}

android {
    signingConfigs {
        if (keystorePropertiesFile.exists()) {
            create("release") {
                storeFile = file(keystoreProperties["storeFile"] as String)
                storePassword = keystoreProperties["storePassword"] as String
                keyAlias = keystoreProperties["keyAlias"] as String
                keyPassword = keystoreProperties["keyPassword"] as String
            }
        }
    }

    buildTypes {
        release {
            if (keystorePropertiesFile.exists()) {
                signingConfig = signingConfigs.getByName("release")
            }
        }
    }
}
```

---

## 📦 Output Files

After the workflow runs:

* ✅ APK → `app-release.apk`
* ✅ AAB → `app-release.aab`

Download from:

**GitHub → Actions → Your Workflow → Artifacts**
<img width="1842" height="520" alt="image" src="https://github.com/user-attachments/assets/73de34a0-767e-4ba4-8a03-bf2c943ae441" />

<img width="1906" height="889" alt="image" src="https://github.com/user-attachments/assets/7b48fb8e-6c1c-40c6-98b3-46d9cca1cf3c" />

<img width="1900" height="934" alt="image" src="https://github.com/user-attachments/assets/39befcad-e19a-4b27-99f5-6e0237bb5438" />




---

## ⚠️ Important Notes

* Never commit:

  * `keystore.jks`
  * `key.properties`
* Always use GitHub Secrets
* Ensure `storeFile=keystore.jks` (relative path)

<img width="947" height="867" alt="image" src="https://github.com/user-attachments/assets/be402150-d640-479f-a8fc-826c7f58df39" />


---

## 🎯 Done!

Now every push to `main` will:

* Build signed APK & AAB
* Upload artifacts automatically


Happy shipping 🚀







