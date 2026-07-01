# ✅ Checklist การเตรียมเครื่องสำหรับ Advanced Flutter 2026
### สำหรับผู้ใช้ VS Code — หลักสูตร BKK Life (2–4 กรกฎาคม 2569)

---

## 1. Software หลักที่ต้องติดตั้ง (ตามลำดับ)

- [ ] **Java JDK 21** (LTS) — ติดตั้งก่อน Flutter SDK
- [ ] **Git** — https://git-scm.com
- [ ] **Flutter SDK** (เวอร์ชันล่าสุด 2026) — https://docs.flutter.dev/get-started/install
- [ ] **Dart SDK** — ติดมาพร้อม Flutter SDK อัตโนมัติ ไม่ต้องติดตั้งแยก
- [ ] **VS Code** (เวอร์ชันล่าสุด) — https://code.visualstudio.com
- [ ] **Android Studio** (เฉพาะส่วน Android SDK + Emulator เท่านั้น ไม่ต้องใช้เป็น IDE หลัก) — https://developer.android.com/studio

> **หมายเหตุ:** ติดตั้ง Android Studio เพื่อรับ Android SDK และสร้าง Emulator เท่านั้น ใช้ VS Code เป็น IDE หลักในการเขียนโค้ด

---

## 1.0 ขั้นตอนการติดตั้ง Java JDK 21 (Windows)

> อ้างอิง: https://www.oracle.com/asean/java/technologies/downloads/#java21
> เวอร์ชันปัจจุบัน: **JDK 21.0.11** (LTS — Long-Term Support) ใช้ได้ฟรีภายใต้ Oracle No-Fee Terms and Conditions (NFTC)

### ทำไมต้องติดตั้ง JDK 21 ก่อน Flutter SDK

Flutter SDK ตั้งแต่เวอร์ชัน 3.13+ มี JDK 17 bundle มาให้ในตัว แต่การมี JDK ติดตั้งใน System ไว้ก่อนจะช่วยให้ Android Studio, Gradle และ build tools ต่าง ๆ ทำงานได้ถูกต้องและสมบูรณ์ยิ่งขึ้น JDK 21 เป็น LTS ที่รองรับ Android Gradle Plugin (AGP) 8.x เต็มรูปแบบ

### ขั้นตอนที่ 1 — ดาวน์โหลด JDK 21 สำหรับ Windows

- [ ] ไปที่ https://www.oracle.com/asean/java/technologies/downloads/#java21
- [ ] เลือก Tab **Windows** ใต้หัวข้อ "Java SE Development Kit 21.0.11 downloads"
- [ ] ดาวน์โหลด **x64 Installer** (ประมาณ 166 MB)
  - ลิงก์โดยตรง: https://download.oracle.com/java/21/latest/jdk-21_windows-x64_bin.exe

### ขั้นตอนที่ 2 — ติดตั้ง JDK 21

- [ ] รันไฟล์ `jdk-21_windows-x64_bin.exe` ที่ดาวน์โหลดมา
- [ ] คลิก **Next** ตามขั้นตอน (ไม่จำเป็นต้องเปลี่ยน path ที่ติดตั้ง)
- [ ] Default path คือ `C:\Program Files\Java\jdk-21\`
- [ ] รอจนติดตั้งเสร็จ แล้วคลิก **Close**

### ขั้นตอนที่ 3 — ตั้งค่า JAVA_HOME (Environment Variable)

- [ ] กด `Windows + Pause` → **Advanced System Settings** → **Environment Variables...**
- [ ] ในส่วน **System variables** คลิก **New...**
  - Variable name: `JAVA_HOME`
  - Variable value: `C:\Program Files\Java\jdk-21`
- [ ] หา **Path** ใน System variables แล้ว Double-click เพิ่มค่าใหม่:
  ```
  %JAVA_HOME%\bin
  ```
- [ ] กด **OK** 3 ครั้งเพื่อบันทึก

### ขั้นตอนที่ 4 — ตรวจสอบการติดตั้ง

- [ ] เปิด Command Prompt ใหม่แล้วรัน:
  ```
  java -version
  javac -version
  ```
- [ ] ควรเห็นผลลัพธ์ประมาณนี้:
  ```
  java version "21.0.11" 2025-01-21 LTS
  javac 21.0.11
  ```

---

## 1.1 ขั้นตอนการติดตั้ง Git (Windows)

> อ้างอิง: https://git-scm.com/downloads/win
> เวอร์ชันปัจจุบัน: **Git 2.49.0** (2025)

### ขั้นตอนที่ 1 — ดาวน์โหลด Git for Windows

- [ ] ไปที่ https://git-scm.com/downloads/win
- [ ] คลิก **"Click here to download"** (64-bit Git for Windows Setup) ประมาณ 65 MB

### ขั้นตอนที่ 2 — ติดตั้ง Git

- [ ] รันไฟล์ `Git-2.49.0-64-bit.exe`
- [ ] ในขั้นตอน **"Choosing the default editor used by Git"** → เลือก **Use Visual Studio Code as Git's default editor**
- [ ] ในขั้นตอน **"Adjusting the name of the initial branch in new repositories"** → เลือก **Override the default branch name** แล้วพิมพ์ `main`
- [ ] ในขั้นตอน **"Adjusting your PATH environment"** → เลือก **Git from the command line and also from 3rd-party software** (แนะนำ)
- [ ] ขั้นตอนอื่นใช้ค่า Default แล้วคลิก **Next** จนถึง **Install**
- [ ] คลิก **Finish** เมื่อติดตั้งเสร็จ

### ขั้นตอนที่ 3 — ตั้งค่า Git Identity ครั้งแรก

- [ ] เปิด Command Prompt หรือ PowerShell ใหม่แล้วรัน:
  ```bash
  git config --global user.name "ชื่อของคุณ"
  git config --global user.email "email@example.com"
  ```

### ขั้นตอนที่ 4 — ตรวจสอบการติดตั้ง

- [ ] รัน:
  ```
  git --version
  ```
- [ ] ควรเห็นผลลัพธ์ประมาณ: `git version 2.49.0.windows.1`

---

## 1.2 ขั้นตอนการติดตั้ง Android Studio (Windows & macOS)

> อ้างอิง: https://developer.android.com/studio?hl=th
> เวอร์ชันปัจจุบัน: **Android Studio Quail 1 | 2026.1.1 Patch 2**

### ดาวน์โหลด Android Studio

| ระบบปฏิบัติการ | ไฟล์ที่ต้องดาวน์โหลด | ขนาด |
|---|---|---|
| Windows 64-bit | `android-studio-quail1-patch2-windows.exe` | 1.5 GB |
| macOS (Intel 64-bit) | `android-studio-quail1-patch2-mac.dmg` | 1.5 GB |
| macOS (Apple Silicon ARM) | `android-studio-quail1-patch2-mac_arm.dmg` | 1.5 GB |

---

### การติดตั้งบน Windows

- [ ] รันไฟล์ `android-studio-quail1-patch2-windows.exe`
- [ ] คลิก **Next** ผ่านทุกขั้นตอน (ติ๊กเลือก **Android Virtual Device** ไว้ด้วย)
- [ ] คลิก **Install** แล้วรอจนเสร็จ
- [ ] คลิก **Finish** เพื่อเปิด Android Studio ครั้งแรก
- [ ] เลือก **Do not import settings** แล้วกด **OK**
- [ ] ที่ Setup Wizard เลือก **Standard** installation แล้วคลิก **Next** จนเสร็จ

### การติดตั้งบน macOS

- [ ] เปิดไฟล์ `.dmg` ที่ดาวน์โหลด
- [ ] ลาก **Android Studio** ไปวางที่โฟลเดอร์ **Applications**
- [ ] เปิด Android Studio จาก Applications (ครั้งแรกอาจต้อง Right-click → Open เพื่อข้าม Gatekeeper)
- [ ] เลือก **Do not import settings** แล้วกด **OK**
- [ ] ที่ Setup Wizard เลือก **Standard** installation แล้วคลิก **Next** จนเสร็จ

---

### ตั้งค่า SDK Tools (Settings → Languages & Frameworks → Android SDK → SDK Tools)

ติ๊กถูกให้ครบตามรายการนี้แล้วกด **Apply** / **OK**:

- [ ] **Android SDK Build-Tools** ✅
- [ ] **NDK (Side by side)** ✅
- [ ] **Android SDK Command-line Tools (latest)** ✅
- [ ] **CMake** ✅
- [ ] **Android Emulator** (v36.6.11+) ✅
- [ ] **Android Emulator hypervisor driver (installer)** ✅
- [ ] **Android SDK Platform-Tools** (v37.0.0+) ✅
- [ ] **Android Support Repository** ✅
- [ ] **Google Play APK Expansion library** ✅
- [ ] **Google Play services** ✅
- [ ] **Google USB Driver** ✅ *(สำหรับรันบนอุปกรณ์จริง Windows)*

> **Android SDK Location:** `C:\Users\<username>\AppData\Local\Android\Sdk`

---

### ตั้งค่า SDK Platforms (SDK Tools → SDK Platforms)

ติ๊กถูกอย่างน้อย 1 เวอร์ชัน แนะนำ:

- [ ] **Android 16.0 "Baklava"** API 36.0 ✅ *(แนะนำสำหรับหลักสูตรนี้)*
- [ ] **Android 16.0 "Baklava"** API 36.1 ✅
- [ ] **Android 17.0 "CinnamonBun"** API 37.0 *(ถ้ามีพื้นที่เพียงพอ)*

---

### สร้าง Virtual Device (Android Emulator)

- [ ] เปิด **Device Manager** ใน Android Studio (ไอคอน 📱 หรือ Tools → Device Manager)
- [ ] คลิก **+** เพื่อสร้าง Virtual Device ใหม่
- [ ] เลือก **Phone** → **Medium Phone** (หรือ Pixel 7)
- [ ] เลือก System Image: **Android 16.0 "Baklava" API 36** x86_64
- [ ] ตั้งชื่อ: `Medium Phone API 36.0` แล้วคลิก **Finish**
- [ ] กด **▶** (Play) เพื่อทดสอบรัน Emulator ให้แน่ใจว่าเปิดได้ปกติ

---

## 1.3 ขั้นตอนการติดตั้ง Flutter SDK แบบ Manual (Windows)

> อ้างอิง: https://docs.flutter.dev/install/manual | Flutter 3.44.0 (อัปเดต 2026-05-05)

### ขั้นตอนที่ 1 — ติดตั้ง Software ที่ต้องการ

- [ ] ดาวน์โหลดและติดตั้ง **Git for Windows** — https://git-scm.com/downloads/win

### ขั้นตอนที่ 2 — ดาวน์โหลด Flutter SDK Bundle

- [ ] ไปที่หน้า SDK Archive — https://docs.flutter.dev/install/archive
- [ ] เลือก **Flutter SDK สำหรับ Windows** (Stable) แล้วดาวน์โหลดไฟล์ `.zip`

### ขั้นตอนที่ 3 — สร้างโฟลเดอร์เก็บ SDK

- [ ] สร้างโฟลเดอร์สำหรับเก็บ SDK เช่น `C:\Users\<username>\develop`

> ⚠️ **ข้อควรระวัง:** เลือก path ที่ไม่มีช่องว่าง (space), ไม่มีอักขระพิเศษ และไม่ต้องการสิทธิ์ Admin

### ขั้นตอนที่ 4 — แตกไฟล์ Flutter SDK

- [ ] เปิด PowerShell แล้วรันคำสั่งต่อไปนี้ (แก้ path ให้ตรงกับเครื่อง)

```powershell
Expand-Archive `
  -Path $env:USERPROFILE\Downloads\flutter_windows_3.44.4-stable.zip `
  -Destination $env:USERPROFILE\develop\
```

> หลังแตกไฟล์จะได้โฟลเดอร์ `develop\flutter\` ที่มีไฟล์ `flutter.bat` อยู่ใน `bin\`
>
> ⚠️ ถ้าไม่พบไฟล์ `flutter.bat` ใน `bin\` อาจถูก Antivirus กักกัน ให้ตั้งค่า Antivirus ให้ไว้ใจโฟลเดอร์นี้แล้วแตกไฟล์ใหม่

### ขั้นตอนที่ 5 — เพิ่ม Flutter เข้า PATH (Environment Variable)

- [ ] กด `Windows + Pause` เพื่อเปิด **System > About**
  - (ถ้าไม่มีปุ่ม Pause ให้กด `Windows + Fn + B`)
- [ ] คลิก **Advanced System Settings** → **Advanced** → **Environment Variables...**
- [ ] ในส่วน **User variables** หาค่า **Path** แล้ว Double-click
- [ ] คลิกแถวว่างใหม่แล้วพิมพ์ path ของ Flutter `bin` เช่น:
  ```
  %USERPROFILE%\develop\flutter\bin
  ```
- [ ] คลิก entry ที่เพิ่มแล้ว **Move Up** จนอยู่ด้านบนสุด
- [ ] กด **OK** 3 ครั้งเพื่อบันทึก

### ขั้นตอนที่ 6 — ปิดแล้วเปิด Terminal / VS Code ใหม่ แล้วตรวจสอบ

- [ ] เปิด Command Prompt หรือ PowerShell ใหม่แล้วรัน:
  ```
  flutter --version
  dart --version
  ```
- [ ] ถ้าขึ้นเวอร์ชันถูกต้อง แสดงว่าติดตั้งสำเร็จ ✅
- [ ] ถ้าไม่พบคำสั่ง ดู Troubleshoot ได้ที่ https://docs.flutter.dev/install/troubleshoot

---

## 1.4 ขั้นตอนการติดตั้ง VS Code (Windows)

> อ้างอิง: https://code.visualstudio.com
> เวอร์ชันปัจจุบัน: **VS Code 1.100+** (2026)

### ขั้นตอนที่ 1 — ดาวน์โหลด VS Code

- [ ] ไปที่ https://code.visualstudio.com
- [ ] คลิก **Download for Windows** — ดาวน์โหลด **User Installer** 64-bit (ประมาณ 100 MB)
  - ลิงก์โดยตรง: https://code.visualstudio.com/sha/download?build=stable&os=win32-x64-user

### ขั้นตอนที่ 2 — ติดตั้ง VS Code

- [ ] รันไฟล์ `VSCodeUserSetup-x64-xxx.exe`
- [ ] ยอมรับ License Agreement แล้วคลิก **Next**
- [ ] เลือก Destination Folder — ใช้ Default แล้วคลิก **Next**
- [ ] ในขั้นตอน **"Select Additional Tasks"** ติ๊กเลือกทุกรายการให้ครบ:
  - [x] **Add "Open with Code" action to Windows Explorer file context menu**
  - [x] **Add "Open with Code" action to Windows Explorer directory context menu**
  - [x] **Register Code as an editor for supported file types**
  - [x] **Add to PATH (requires shell restart)** ← สำคัญมาก!
- [ ] คลิก **Install** แล้วรอจนเสร็จ
- [ ] คลิก **Finish** — VS Code จะเปิดขึ้นมาอัตโนมัติ

### ขั้นตอนที่ 3 — ตรวจสอบการติดตั้ง

- [ ] เปิด Command Prompt หรือ PowerShell **ใหม่** แล้วรัน:
  ```
  code --version
  ```
- [ ] ควรเห็นเลขเวอร์ชัน เช่น `1.100.2`
- [ ] ทดสอบเปิด VS Code จาก Terminal ด้วยคำสั่ง `code .`

> **หมายเหตุ:** ติดตั้ง Extensions ต่าง ๆ ในขั้นตอนถัดไป (Section 2)

---

## 2. VS Code Extensions ที่ต้องติดตั้ง

- [ ] **Flutter** (by Dart Code) — `ext install Dart-Code.flutter`
- [ ] **Dart** (by Dart Code) — `ext install Dart-Code.dart-code`
- [ ] **Riverpod Snippets** (by Robert Brunhage) — `ext install robert-brunhage.flutter-riverpod-snippets`
- [ ] **Awesome Flutter Snippets** (by Neevash Ramdial) — `ext install Nash.awesome-flutter-snippets`
- [ ] **Flutter Widget Snippets** (by Alexis Villegas Torres) — `ext install alexisvt.flutter-snippets`
- [ ] **Pubspec Assist** (by Jeroen Meijer) — `ext install jeroen-meijer.pubspec-assist`
- [ ] **Bracket Pair Color DLW** (by Bracket Pair Color DLW) — `ext install BracketPairColorDLW.bracket-pair-color-dlw`
- [ ] **Catppuccin for VSCode** (by Catppuccin) — `ext install Catppuccin.catppuccin-vsc`
- [ ] **Catppuccin Icons for VSCode** (by Catppuccin) — `ext install Catppuccin.catppuccin-vsc-icons`

---

## 3. ตรวจสอบ Flutter SDK ให้พร้อม

### ขั้นตอนที่ 1 — ตรวจสอบเวอร์ชัน

- [ ] รัน `flutter --version` แล้วเห็นเวอร์ชัน Flutter และ Dart
- [ ] รัน `dart --version` แล้วเห็นเวอร์ชัน Dart

### ขั้นตอนที่ 2 — รัน flutter doctor ครั้งแรก

- [ ] รัน `flutter doctor` แล้วดูผลลัพธ์
  - ถ้าเห็น `[!] Android toolchain` พร้อมข้อความ **Some Android licenses not accepted** ให้ทำขั้นตอนถัดไป
  - ถ้าทุกรายการเป็น `[✓]` ข้ามไปขั้นตอนที่ 4 ได้เลย

### ขั้นตอนที่ 3 — ยอมรับ Android Licenses

- [ ] รัน `flutter doctor --android-licenses`
- [ ] ระบบจะถามว่า **Review licenses that have not been accepted (y/N)?** พิมพ์ `y` แล้วกด Enter
- [ ] อ่าน License แต่ละรายการและพิมพ์ `y` เพื่อยอมรับทุกรายการจนครบ
- [ ] รอจนขึ้น **All SDK package licenses accepted**

### ขั้นตอนที่ 4 — ตรวจสอบ flutter doctor อีกครั้ง

- [ ] รัน `flutter doctor` อีกครั้ง ควรเห็นผลลัพธ์ **No issues found!**
  ```
  [✓] Flutter (Channel stable, 3.44.4, ...)
  [✓] Windows Version (Windows 11 or higher, ...)
  [✓] Android toolchain - develop for Android devices (Android SDK version 37.0.0)
  [✓] Chrome - develop for the web
  [✓] Visual Studio - develop Windows apps
  [✓] Connected device (4 available)
  [✓] Network resources
  ```

### ขั้นตอนที่ 5 — ตรวจสอบรายละเอียดด้วย flutter doctor -v

- [ ] รัน `flutter doctor -v` เพื่อดูข้อมูลเชิงลึก ควรเห็นข้อมูลครบดังนี้:
  - Flutter version **3.44.4** channel stable
  - Dart version **3.12.2**
  - DevTools version **2.57.0**
  - Android SDK at `C:\Users\<username>\AppData\Local\Android\sdk`
  - Java version **OpenJDK 21.x**
  - **All Android licenses accepted**

---

## 4. ตั้งค่า Emulator / อุปกรณ์รันแอป

- [ ] สร้าง **Android Emulator** ผ่าน Android Studio → Virtual Device Manager
  - แนะนำ: Pixel 7 หรือสูงกว่า, API Level 34+
- [ ] **หรือ** เตรียม **Android จริง** พร้อมเปิด USB Debugging และ Developer Options
- [ ] รัน `flutter devices` แล้วเห็น Device อย่างน้อย 1 เครื่อง

---

## 5. ทดสอบ VS Code + Flutter ให้พร้อมก่อนวันอบรม

- [ ] สร้างโปรเจกต์ทดสอบด้วย `flutter create --org com.itgenius test_app`
- [ ] เปิดโปรเจกต์ใน VS Code และรันบน Emulator หรืออุปกรณ์จริงได้สำเร็จ
- [ ] Hot Reload (บันทึกไฟล์) ทำงานได้ปกติ

---

## 6. การติดตั้งบน macOS (Complete Setup Guide)

> คู่มือนี้รองรับทั้งชิป **Intel** และ **Apple Silicon (M1/M2/M3/M4)**
> แนะนำใช้ **Homebrew** เป็น Package Manager หลักสำหรับ macOS

---

### 6.0 ติดตั้ง Homebrew (ถ้ายังไม่มี)

Homebrew คือ Package Manager มาตรฐานของ macOS — จำเป็นสำหรับขั้นตอนถัดไปทั้งหมด

- [ ] เปิด **Terminal** (`Command + Space` พิมพ์ `Terminal`)
- [ ] รันคำสั่งติดตั้ง Homebrew:
  ```bash
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  ```
- [ ] ทำตามคำแนะนำบนหน้าจอ (ใส่ Password และกด Enter เมื่อถาม)
- [ ] บน **Apple Silicon (M1/M2/M3/M4)** รันเพิ่มเพื่อเพิ่ม Homebrew เข้า PATH:
  ```bash
  echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
  eval "$(/opt/homebrew/bin/brew shellenv)"
  ```
- [ ] ตรวจสอบ:
  ```bash
  brew --version
  ```

---

### 6.1 ติดตั้ง Java JDK 21 บน macOS

> ใช้ **Eclipse Temurin** (Adoptium) แทน Oracle JDK เนื่องจากไม่มีข้อจำกัด License ใน Production

- [ ] ติดตั้ง JDK 21 ผ่าน Homebrew:
  ```bash
  brew install --cask temurin@21
  ```
- [ ] ตั้งค่า JAVA_HOME ใน `~/.zprofile`:
  ```bash
  echo 'export JAVA_HOME=$(/usr/libexec/java_home -v 21)' >> ~/.zprofile
  source ~/.zprofile
  ```
- [ ] ตรวจสอบ:
  ```bash
  java -version
  javac -version
  ```
  ควรเห็น: `openjdk version "21.x.x" ...`

---

### 6.2 ติดตั้ง Git บน macOS

macOS มี Git ติดมากับ Xcode Command Line Tools แต่เวอร์ชันอาจเก่า แนะนำติดตั้งผ่าน Homebrew:

- [ ] รัน:
  ```bash
  brew install git
  ```
- [ ] ตั้งค่า Git Identity ครั้งแรก:
  ```bash
  git config --global user.name "ชื่อของคุณ"
  git config --global user.email "email@example.com"
  ```
- [ ] ตรวจสอบ:
  ```bash
  git --version
  ```

---

### 6.3 ติดตั้ง Flutter SDK บน macOS

#### วิธีที่ 1 — ผ่าน Homebrew (แนะนำสำหรับผู้ใช้ทั่วไป)

- [ ] รัน:
  ```bash
  brew install --cask flutter
  ```
- [ ] ตรวจสอบ:
  ```bash
  flutter --version
  ```

#### วิธีที่ 2 — Manual Download (ถ้าต้องการเลือกเวอร์ชันเอง)

- [ ] ไปที่ https://docs.flutter.dev/install/archive
- [ ] เลือก Flutter SDK สำหรับ **macOS** — มีให้เลือกแยก Intel และ Apple Silicon
- [ ] แตกไฟล์และวางใน `~/develop/`:
  ```bash
  mkdir -p ~/develop
  unzip ~/Downloads/flutter_macos_arm64_3.44.x-stable.zip -d ~/develop/
  ```
- [ ] เพิ่ม Flutter เข้า PATH ใน `~/.zprofile`:
  ```bash
  echo 'export PATH="$HOME/develop/flutter/bin:$PATH"' >> ~/.zprofile
  source ~/.zprofile
  ```
- [ ] ตรวจสอบ:
  ```bash
  flutter --version
  dart --version
  ```

---

### 6.4 ติดตั้ง VS Code บน macOS

- [ ] ไปที่ https://code.visualstudio.com
- [ ] คลิก **Download for Mac** — เลือก **Universal** (รองรับทั้ง Intel และ Apple Silicon)
- [ ] เปิดไฟล์ `.zip` ที่ดาวน์โหลด — macOS จะแตกไฟล์เป็น `Visual Studio Code.app` อัตโนมัติ
- [ ] ลาก `Visual Studio Code.app` ไปวางที่โฟลเดอร์ **Applications**
- [ ] เปิด VS Code จาก Launchpad หรือ Applications
  - ครั้งแรกอาจต้อง Right-click → Open เพื่อข้าม Gatekeeper
- [ ] เพิ่มคำสั่ง `code` เข้า Terminal:
  - กด `Cmd + Shift + P` เปิด Command Palette
  - พิมพ์ `Shell Command: Install 'code' command in PATH` แล้วกด Enter
- [ ] ติดตั้ง Extensions ตาม **Section 2** ด้านบน
- [ ] ตรวจสอบ:
  ```bash
  code --version
  ```

---

### 6.5 ติดตั้ง Xcode + CocoaPods (จำเป็นสำหรับ iOS Development)

> ต้องมี Xcode เพื่อรันแอปบน iOS Simulator และอุปกรณ์ iPhone/iPad จริง

- [ ] ติดตั้ง **Xcode** จาก Mac App Store (ขนาดประมาณ 13–15 GB)
  - หรือดาวน์โหลดจาก https://developer.apple.com/xcode/
- [ ] เปิด Xcode อย่างน้อย 1 ครั้งและยอมรับ License Agreement
- [ ] ติดตั้ง Xcode Command Line Tools:
  ```bash
  xcode-select --install
  sudo xcodebuild -license accept
  ```
- [ ] ติดตั้ง **CocoaPods** (Dependency Manager ฝั่ง iOS):
  ```bash
  brew install cocoapods
  ```
- [ ] ตรวจสอบ:
  ```bash
  pod --version
  xcode-select -p
  ```

---

### 6.6 ติดตั้ง Android Studio บน macOS

> ดูขั้นตอนการติดตั้งฉบับ macOS แบบละเอียดได้ที่ **หัวข้อ 1.2 → การติดตั้งบน macOS** ด้านบน

---

### 6.7 รัน flutter doctor บน macOS — ตรวจสอบขั้นสุดท้าย

- [ ] รัน:
  ```bash
  flutter doctor
  ```
- [ ] ยอมรับ Android Licenses:
  ```bash
  flutter doctor --android-licenses
  ```
- [ ] ผลลัพธ์สุดท้ายที่ควรได้:
  ```
  [✓] Flutter (Channel stable, 3.44.x, on macOS ...)
  [✓] Android toolchain - develop for Android devices
  [✓] Xcode - develop for iOS and macOS (Xcode xx.x)
  [✓] Chrome - develop for the web
  [✓] Android Studio (version 2026.x)
  [✓] VS Code (version 1.100.x)
  [✓] Connected device (x available)
  [✓] Network resources
  ```
- [ ] ถ้ามี `[!]` ที่ Xcode ให้รัน:
  ```bash
  sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
  sudo xcodebuild -runFirstLaunch
  ```
- [ ] สร้างโปรเจกต์ทดสอบและรันบน iOS Simulator:
  ```bash
  flutter create --org com.itgenius test_app
  cd test_app
  open -a Simulator
  flutter run
  ```
