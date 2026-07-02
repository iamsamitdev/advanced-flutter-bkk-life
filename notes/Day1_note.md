# Advanced Flutter 2026 — วันที่ 1: Scalable Architecture & Riverpod Fundamentals

**หลักสูตรอบรมเชิงปฏิบัติการ: Advanced Flutter 2026 (Scalable Architecture & Riverpod)**
**จัดอบรมให้: บริษัท กรุงเทพประกันชีวิต จำกัด (มหาชน)**
**วันที่ 1: วางสถาปัตยกรรมที่ขยายได้ และปูพื้นฐาน Riverpod**
วันที่: 2 กรกฎาคม 2569 | เวลา 09:00–16:00 น. | Onsite Workshop
ผู้สอน: อ.สามิตร โกยม

---

## 🎯 วัตถุประสงค์การเรียนรู้ประจำวัน

เมื่อจบการอบรมวันที่ 1 ผู้เรียนจะสามารถ:

1. อธิบายแนวคิด Clean Architecture และเหตุผลที่ต้องแยกชั้น (Layer) ของแอประดับองค์กรได้
2. เปรียบเทียบและเลือกใช้โครงสร้างโฟลเดอร์แบบ **Feature-first** กับ **Layer-first** ได้อย่างเหมาะสม
3. ติดตั้งและตั้งค่า **Riverpod** ด้วย `ProviderScope` รวมถึงเปิดใช้ Code Generation ด้วย `build_runner` ได้
4. ใช้งาน Provider หลักทุกชนิด — `Provider`, `StateProvider`, `FutureProvider` — และเข้าใจการเปลี่ยนผ่านสู่ `Notifier`/`AsyncNotifier` ได้
5. เลือกใช้ `ConsumerWidget`/`ConsumerStatefulWidget` และแยกความแตกต่างของ `ref.watch()`, `ref.read()`, `ref.listen()` ได้ถูกต้อง
6. ลงมือ Refactor โปรเจกต์ให้เป็น Feature-first และจัดการ State หน้ารายการกรมธรรม์ด้วย Riverpod ได้จริง (Workshop Day 1)

> **หมายเหตุสำคัญของหลักสูตรนี้:** ตลอด 3 วันเราจะใช้ **Riverpod แบบ Code Generation** (`@riverpod`) เป็นแกนหลัก เพราะเป็นมาตรฐานใหม่ปี 2026 ที่ปลอดภัยกว่า (compile-safe), ลดโค้ด boilerplate, และทำให้ refactor ได้ง่าย เราจะค่อย ๆ พัฒนาแอปเดียวต่อเนื่องคือ **BLA Policy Companion** — แอปจัดการกรมธรรม์ (ใช้ข้อมูลจำลองทั้งหมด ไม่เกี่ยวกับข้อมูลลูกค้าจริง)

---

## 🧭 กำหนดการวันที่ 1 (โดยสังเขป)

| เวลา        | หัวข้อ                                                                    |
| ----------- | ------------------------------------------------------------------------- |
| 09:00–09:30 | ทดสอบก่อนเรียน (Pretest) + แนะนำหลักสูตรและโปรเจกต์ BLA Policy Companion  |
| 09:30–10:30 | **Module 1** Introduction to Modern Flutter Architecture (2026)           |
| 10:30–12:00 | **Module 2** Deep Dive into Riverpod (ติดตั้ง + ProviderScope + Code Gen) |
| 12:00–13:00 | พักกลางวัน                                                                |
| 13:00–14:15 | **Module 3** Understanding Core Providers                                 |
| 14:15–15:00 | **Module 4** Consuming State (Consumer & ref)                             |
| 15:00–16:00 | **Workshop Day 1** วาง Feature-first + State หน้ารายการกรมธรรม์           |

---

## ✅ ทดสอบก่อนเรียน (Pretest)

### เวลา 09:00–09:30 น.

ก่อนเริ่มเรียน ให้ผู้เรียนทำแบบทดสอบก่อนเรียนตามลิงก์ที่วิทยากรแจ้งในห้องอบรม เพื่อวัดพื้นฐานความเข้าใจเดิมเกี่ยวกับ Flutter, State Management และ Architecture ผลทดสอบนี้ **ไม่มีผลต่อการประเมิน** ใช้เพียงเพื่อปรับจังหวะการสอนให้เหมาะกับกลุ่มผู้เรียน

**ตรวจความพร้อมเครื่องมือก่อนเริ่ม** — เปิด Terminal แล้วรัน:

```bash
flutter --version
flutter doctor
```

ควรเห็น Flutter เวอร์ชันล่าสุดปี 2026 (ช่อง Dart ต้องเป็นสีเขียวทั้งหมด) และมี Emulator/Simulator หรือเครื่องจริงพร้อมรัน

---

## 📚 Module 1: Introduction to Modern Flutter Architecture (2026)

### เวลา 09:30–10:30 น.

> 💡 **หัวใจของ Module นี้:** ก่อนจะเขียนโค้ดบรรทัดแรก เราต้องเข้าใจก่อนว่า "แอปที่ขยายได้" หน้าตาเป็นอย่างไร และทำไมการแยกความรับผิดชอบ (Separation of Concerns) จึงเป็นเรื่องคอขวดของทีมขนาดใหญ่

---

### 1.1 ทำไมสถาปัตยกรรมจึงสำคัญเมื่อแอปใหญ่ขึ้น

เมื่อแอปยังเล็ก โค้ดที่ปน UI, Business Logic และการเรียก API ไว้ใน Widget เดียวกันก็ยัง "พอทำงานได้" แต่เมื่อแอปโตขึ้น ฟีเจอร์เพิ่ม ทีมขยาย ปัญหาจะตามมาทันที เช่น แก้โค้ดที่หนึ่งแล้วพังอีกที่, ทดสอบยากเพราะ Logic ผูกติดกับ UI, และคนใหม่เข้าทีมแล้วหาโค้ดไม่เจอ

```
แอปเล็ก (ทุกอย่างใน Widget):          แอปองค์กร (แยกชั้นชัดเจน):
┌─────────────────────────────┐       ┌─────────────────────────────┐
│  Widget                     │       │  Presentation (UI + State)  │
│   ├─ UI                     │  ⟶   │        ↓ เรียกผ่าน abstract  │
│   ├─ เรียก API ตรง ๆ        │       │  Domain (Entity + Logic)    │
│   ├─ แปลง JSON              │       │        ↓                     │
│   └─ จัดการ error           │       │  Data (Repository + API)    │
└─────────────────────────────┘       └─────────────────────────────┘
   แก้ที่เดียวกระทบทั้งหมด                  แต่ละชั้นเปลี่ยนได้อิสระ ทดสอบง่าย
```

### 1.2 Clean Architecture สำหรับ Flutter (ฉบับใช้งานจริง)

แนวคิด Clean Architecture แบ่งโค้ดเป็น 3 ชั้นหลัก โดยมีกฎเหล็กว่า **ชั้นในไม่รู้จักชั้นนอก** (Dependency Rule) — ชั้น Domain ซึ่งเป็นหัวใจธุรกิจต้องไม่ผูกกับ Flutter หรือ API ใด ๆ

| ชั้น (Layer)     | หน้าที่                              | ตัวอย่างในแอป BLA                            |
| ---------------- | ------------------------------------ | -------------------------------------------- |
| **Presentation** | แสดงผล UI และจัดการ State            | หน้า `PolicyListPage`, Provider ของ Riverpod |
| **Domain**       | กฎทางธุรกิจ + โมเดลแก่นกลาง (Entity) | คลาส `Policy`, การคำนวณสถานะกรมธรรม์         |
| **Data**         | ดึง/บันทึกข้อมูล (API, DB, Cache)    | `PolicyRepository`, การเรียก Dio, แปลง JSON  |

> ⚠️ **ข้อควรระวัง:** ในหลักสูตรนี้เราใช้ Clean Architecture แบบ "พอดี" (pragmatic) ไม่ยึดทุกกฎจนเกิดความซับซ้อนเกินจำเป็น เป้าหมายคือ "ทดสอบง่าย เปลี่ยนชิ้นส่วนได้" ไม่ใช่ "มีไฟล์เยอะที่สุด"

### 1.3 Feature-first vs Layer-first — เลือกแบบไหน

นี่คือการตัดสินใจเชิงโครงสร้างที่สำคัญที่สุดของวันแรก มีสองแนวทางหลักในการจัดโฟลเดอร์:

```
Layer-first (จัดตามชั้น):              Feature-first (จัดตามฟีเจอร์):
lib/                                   lib/
├── presentation/                      ├── features/
│   ├── policy_page.dart               │   ├── policy/
│   ├── claim_page.dart                │   │   ├── presentation/
│   └── login_page.dart               │   │   ├── domain/
├── domain/                            │   │   └── data/
│   ├── policy.dart                    │   ├── claim/
│   ├── claim.dart                     │   │   ├── presentation/
│   └── user.dart                      │   │   ├── domain/
└── data/                              │   │   └── data/
    ├── policy_repo.dart               │   └── auth/
    ├── claim_repo.dart                │       ├── presentation/
    └── auth_repo.dart                 │       ├── domain/
                                       │       └── data/
                                       └── core/  (ของใช้ร่วมทั้งแอป)
```

| ประเด็น                  | Layer-first                     | Feature-first (ที่เราจะใช้) |
| ------------------------ | ------------------------------- | --------------------------- |
| หาโค้ดของฟีเจอร์เดียว    | กระจายหลายโฟลเดอร์              | อยู่รวมที่เดียว ค้นง่าย     |
| ทีมขนาดใหญ่ทำงานพร้อมกัน | ชนกันบ่อย (แก้โฟลเดอร์เดียวกัน) | แยกฟีเจอร์ ลดการชนกัน       |
| ลบ/ย้ายฟีเจอร์           | ยุ่งยาก ต้องตามเก็บหลายที่      | ลบโฟลเดอร์เดียวจบ           |
| เหมาะกับ                 | โปรเจกต์เล็ก/ตัวอย่าง           | **แอประดับองค์กร**          |

> ✅ **สรุปการตัดสินใจ:** สำหรับ BLA Policy Companion เราเลือก **Feature-first** เพราะรองรับการทำงานเป็นทีม แยกความรับผิดชอบชัดเจน และต่อยอดในวันที่ 2–3 (claim, auth) ได้สะอาด

### 1.4 สร้างโปรเจกต์เริ่มต้น (Project Bootstrap)

ก่อนลงมือเขียนโค้ด ให้สร้างโปรเจกต์ Flutter ใหม่ โดยกำหนด `--org` เป็นโดเมนขององค์กรตั้งแต่ต้น เพื่อให้ได้ `applicationId` (Android) และ `bundle id` (iOS) ที่เหมาะสม (แก้ทีหลังยุ่งยากเพราะกระทบไฟล์ native หลายที่):

```bash
flutter create --org com.itgenius bla_policy_companion
cd bla_policy_companion
```

- `--org com.itgenius` → ชื่อแพ็กเกจฐานเป็น `com.itgenius.bla_policy_companion`
- `bla_policy_companion` → ชื่อโปรเจกต์ (โฟลเดอร์ + ชื่อแพ็กเกจ Dart)
- ผลลัพธ์: ได้โครงสร้างมาตรฐานของ Flutter — จากนั้นเราจะ **จัดโครงใหม่เป็น Feature-first** ใน Workshop ช่วงบ่าย

> 💡 ตรวจว่าโปรเจกต์รันได้ก่อนเริ่ม: `flutter run` แล้วเห็นหน้า counter เริ่มต้นของ Flutter

---

## 📚 Module 2: Deep Dive into Riverpod

### เวลา 10:30–12:00 น.

> 💡 **หัวใจของ Module นี้:** Riverpod คือ "compile-safe Provider" — มันยกข้อจำกัดของ Provider เดิมที่ผูกกับ `BuildContext` ออกไป ทำให้เข้าถึง State จากที่ไหนก็ได้ ทดสอบง่าย และจับ error ได้ตั้งแต่ตอน compile ไม่ใช่ตอน runtime

---

### 2.1 ทำไมต้อง Riverpod และวิวัฒนาการจาก Provider

`Provider` (ตัวเดิม) ทำงานบน Widget Tree จึงเข้าถึง State ได้เฉพาะเมื่อมี `BuildContext` และเสี่ยงเจอ error ตอน runtime เช่น `ProviderNotFoundException` ส่วน Riverpod ออกแบบใหม่ให้ Provider เป็น **global ที่ปลอดภัย** ไม่ผูกกับ context

```
Provider เดิม:                          Riverpod:
┌─────────────────────────────┐         ┌─────────────────────────────┐
│ ต้องมี BuildContext          │         │ เข้าถึงผ่าน ref ที่ไหนก็ได้   │
│ เจอ error ตอน runtime        │   ⟶    │ จับ error ตอน compile        │
│ ProviderNotFoundException    │         │ ทดสอบไม่ต้องสร้าง Widget      │
│ ซ้อน MultiProvider ยุ่ง      │         │ ประกาศ provider อิสระต่อกัน   │
└─────────────────────────────┘         └─────────────────────────────┘
```

| คุณสมบัติ                | Provider (เดิม) | Riverpod (ที่เราจะใช้)    |
| ------------------------ | --------------- | ------------------------- |
| ผูกกับ BuildContext      | ใช่             | **ไม่**                   |
| ความปลอดภัยตอน compile   | ต่ำ             | **สูง (compile-safe)**    |
| ทดสอบ (Unit Test)        | ต้องมี Widget   | **ทดสอบ Logic ตรง ๆ ได้** |
| Auto-dispose เมื่อไม่ใช้ | ทำเองยาก        | **มีในตัว**               |
| Code Generation          | ไม่มี           | **มี (`@riverpod`)**      |

### 2.2 ขั้นตอนที่ 1 — ติดตั้ง Package

ในโปรเจกต์ Flutter ของเรา ติดตั้ง dependency ทั้งหมดที่จะใช้ตลอด 3 วันด้วยคำสั่งเดียว:

```bash
# Package หลักของ Riverpod + Code Generation
flutter pub add flutter_riverpod:^2.6.1 riverpod_annotation:^2.6.1

# Package สำหรับสร้างโมเดล (ใช้หนักในวันที่ 2)
flutter pub add freezed_annotation:^2.4.4 json_annotation:^4.9.0

# Dev dependencies: ตัวสร้างโค้ดและตัวตรวจ
flutter pub add dev:build_runner:^2.4.13 dev:riverpod_generator:^2.6.3 dev:freezed:^2.5.7 dev:json_serializable:^6.9.0 dev:custom_lint:^0.7.0 dev:riverpod_lint:^2.6.3

# Package สำหรับ UI / Design System (ใช้ตั้งแต่วันแรก): ไอคอน Hugeicons + ฟอนต์ Google Fonts
flutter pub add google_fonts:^6.2.1 hugeicons:^1.1.7
```

ตรวจสอบว่าใน `pubspec.yaml` มีรายการครบตรงกับโปรเจกต์จริง (branch `day1`):

```yaml
# pubspec.yaml (ฉบับจริงใน branch day1)
name: bla_policy_companion
description: "BLA Policy Companion — Advanced Flutter 2026. Day 1: Scalable Architecture & Riverpod + Design System. Mock data only."
publish_to: "none"
version: 1.0.0+1

environment:
  sdk: ">=3.4.0 <4.0.0"
  flutter: ">=3.27.0"

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8
  flutter_riverpod: ^2.6.1
  riverpod_annotation: ^2.6.1
  google_fonts: ^6.2.1
  freezed_annotation: ^2.4.4
  json_annotation: ^4.9.0
  hugeicons: ^1.1.7

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^5.0.0
  build_runner: ^2.4.13
  riverpod_generator: ^2.6.3
  freezed: ^2.5.7
  json_serializable: ^6.9.0
  custom_lint: ^0.7.0
  riverpod_lint: ^2.6.3

# analyzer_plugin 0.12.0 (pulled in by custom_lint 0.7.x) is source-incompatible
# with analyzer 7.4.5+ (Element2 API migration). Force the 0.13.x line which
# uses the new Element2 API.
dependency_overrides:
  analyzer_plugin: ^0.13.0

flutter:
  uses-material-design: true
  assets:
    - assets/images/
```

> ⚠️ **หมายเหตุเวอร์ชัน:** โปรเจกต์นี้ตรึงไว้ที่ **Riverpod สาย 2.6.x** (ตัวเสถียรที่ใช้กับ `riverpod_generator` 2.x) — แนวคิด `@riverpod` + Code Generation เหมือนสาย 3.x ทุกประการ ต่างเพียงรายละเอียด API ภายใน (เช่น typedef `PolicyRepositoryRef` ที่สาย 3.x จะเปลี่ยนเป็น `Ref` เฉย ๆ) และสังเกตบล็อก `dependency_overrides` ที่จำเป็นสำหรับให้ `custom_lint` ทำงานกับ analyzer รุ่นใหม่ได้
> 📌 **สำคัญ:** `pubspec.yaml` ต้องประกาศ `assets: - assets/images/` เพราะ `AppLogo` และ avatar ในหน้า Home/Profile อ้างรูปจากโฟลเดอร์นี้ — ถ้าลืม รูปจะไม่ขึ้นและ error ตอน runtime

#### 📦 หน้าที่ของแต่ละ Package ที่ติดตั้ง

เพื่อให้ผู้เรียนเข้าใจว่าเหตุใดจึงต้องติดตั้งแต่ละตัว ก่อนไปต่อในหัวข้อถัดไป สรุปหน้าที่ของแต่ละ Package ไว้ดังตาราง:

**กลุ่มที่ 1 — Riverpod หลัก + Code Generation**

| Package               | หน้าที่                                                                                                 |
| ---------------------- | ---------------------------------------------------------------------------------------------------------- |
| `flutter_riverpod`     | Package แกนหลักของ Riverpod ให้ `ProviderScope`, `ConsumerWidget`, `ConsumerStatefulWidget` และ `ref.watch/read/listen` |
| `riverpod_annotation`  | ให้ annotation `@riverpod` สำหรับประกาศ Provider แบบ Code Generation (ต้องทำงานคู่กับ `riverpod_generator` ด้านล่าง) |

**กลุ่มที่ 2 — สร้างโมเดล (ใช้หนักในวันที่ 2)**

| Package             | หน้าที่                                                                                     |
| -------------------- | -------------------------------------------------------------------------------------------- |
| `freezed_annotation` | ให้ annotation `@freezed` สำหรับสร้างคลาสโมเดล Immutable ที่มี `copyWith`/`==`/`hashCode` ให้อัตโนมัติ |
| `json_annotation`    | ให้ annotation `@JsonSerializable` สำหรับกำหนดกฎแปลงคลาส Dart ↔ JSON                        |

**กลุ่มที่ 3 — Dev Dependencies (ตัวสร้างโค้ด + ตัวตรวจ ทำงานเฉพาะตอนพัฒนา)**

| Package             | หน้าที่                                                                                                   |
| -------------------- | ------------------------------------------------------------------------------------------------------------ |
| `build_runner`       | Engine กลางที่สั่งให้ตัวสร้างโค้ดทั้งหมดทำงาน ผ่านคำสั่ง `build` หรือ `watch`                              |
| `riverpod_generator` | อ่าน `@riverpod` แล้วสร้างไฟล์ `.g.dart` ที่มี Provider จริง (เช่น `helloProvider`, `counterProvider`)      |
| `freezed`            | อ่าน `@freezed` แล้วสร้าง implementation ของคลาสโมเดล Immutable ให้                                        |
| `json_serializable`  | อ่าน `@JsonSerializable` แล้วสร้างเมธอด `fromJson`/`toJson` ให้อัตโนมัติ                                    |
| `custom_lint`        | ตัวกลางที่รองรับการเพิ่มกฎ Lint แบบกำหนดเอง นอกเหนือจากกฎมาตรฐานของ Dart Analyzer                          |
| `riverpod_lint`      | ชุดกฎ Lint เฉพาะของ Riverpod ทำงานผ่าน `custom_lint` เช่น เตือนเมื่อลืมใส่ `part '....g.dart';` หรือใช้ `ref.watch()` ผิดที่ (ดูหัวข้อ 4.3) |

**กลุ่มที่ 4 — UI / Design System**

| Package       | หน้าที่                                                                                                    |
| -------------- | -------------------------------------------------------------------------------------------------------------- |
| `google_fonts` | ดึงฟอนต์จาก Google Fonts มาใช้โดยไม่ต้องฝังไฟล์ฟอนต์เอง — ในโปรเจกต์นี้ใช้ผสม Inter (อังกฤษ) กับ Anuphan (ไทย) ตาม `core/theme/app_typography.dart` |
| `hugeicons`    | ชุดไอคอนสำเร็จรูป ถูกครอบ (wrap) ด้วย `core/icons/app_icons.dart` เพื่อคุมขนาด/สไตล์ให้สม่ำเสมอทั้งแอป          |

> 📖 **อ้างอิง:** รายละเอียด API และเวอร์ชันล่าสุดของแต่ละ Package ตรวจสอบได้ที่ pub.dev (เช่น pub.dev/packages/flutter_riverpod, pub.dev/packages/freezed) และเอกสารทางการของ Riverpod ที่ riverpod.dev

### 2.3 ขั้นตอนที่ 2 — ครอบแอปด้วย ProviderScope

`ProviderScope` คือ Widget ที่ทำหน้าที่เก็บ State ของทุก Provider ในแอป ต้องครอบไว้ที่ราก (root) ของแอปเสมอ — ถ้าลืม จะเข้าถึง Provider ไม่ได้เลย

```dart
// main.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  // ครอบทั้งแอปด้วย ProviderScope — หัวใจที่เก็บ state ของทุก provider
  runApp(
    const ProviderScope(
      child: BlaApp(),
    ),
  );
}

class BlaApp extends StatelessWidget {
  const BlaApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'BLA Policy Companion',
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.indigo),
      home: const Placeholder(), // จะแทนที่ด้วยหน้าจริงใน Workshop
    );
  }
}
```

> ✅ **จุดสำคัญ:** `ProviderScope` ต้องอยู่นอกสุด ถ้ามีหลายแอป (เช่น testing) สามารถสร้าง scope ซ้อนเพื่อ override provider ได้ — เราจะใช้ความสามารถนี้ตอนเขียน Unit Test ในวันที่ 3

### 2.4 ขั้นตอนที่ 3 — เปิดใช้ Code Generation ด้วย build_runner

มาตรฐานปี 2026 คือเขียน provider ด้วย annotation `@riverpod` แล้วให้เครื่องสร้างโค้ดส่วนที่เหลือให้ วิธีนี้ลด boilerplate และกัน error เรื่องการพิมพ์ชนิดข้อมูลผิด

```bash
# สั่งสร้างโค้ดครั้งเดียว
dart run build_runner build --delete-conflicting-outputs

# หรือสั่งให้เฝ้าดูไฟล์และสร้างใหม่อัตโนมัติเมื่อแก้โค้ด (แนะนำตอนทำ workshop)
dart run build_runner watch --delete-conflicting-outputs
```

> ⚠️ **ข้อควรระวังที่พบบ่อย:** ทุกไฟล์ที่ใช้ `@riverpod` ต้องมีบรรทัด `part 'ชื่อไฟล์.g.dart';` ด้านบนเสมอ มิฉะนั้น build*runner จะไม่สร้างโค้ดให้ และจะขึ้น error สีแดงว่าหา `*$...` ไม่เจอ

---

### 🧪 Lab 2.1 — สร้าง provider ตัวแรก + ยืนยันว่า Code Generation ทำงาน

> **เป้าหมาย:** พิสูจน์ว่า `ProviderScope` + `build_runner` ตั้งค่าถูกต้อง โดยสร้าง provider ตัวแรก แล้วนำค่ามาแสดงบนจอจริง

**ขั้นที่ 1 — สร้างไฟล์ `lib/hello_provider.dart`**

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'hello_provider.g.dart'; // ⬅ ห้ามลืม! build_runner จะสร้างไฟล์นี้ให้

// provider ตัวแรกของเรา — คืนข้อความทักทาย (อ่านอย่างเดียว)
@riverpod
String hello(HelloRef ref) => 'Hello Riverpod 👋 (BLA Policy Companion)';
```

**ขั้นที่ 2 — สั่งสร้างโค้ด แล้วสังเกตว่ามีไฟล์ `hello_provider.g.dart` โผล่ขึ้นมา (ข้างในมี `helloProvider`)**

```bash
dart run build_runner build --delete-conflicting-outputs
```

> ถ้าเปิด `dart run build_runner watch --delete-conflicting-outputs` ค้างไว้ ไฟล์ `.g.dart` จะถูกสร้าง/อัปเดตให้อัตโนมัติทุกครั้งที่กด Save

**ขั้นที่ 3 — อ่านค่าด้วย `ConsumerWidget` แล้วตั้งเป็น `home` ชั่วคราว**

```dart
// lib/hello_view.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'hello_provider.dart';

class HelloView extends ConsumerWidget {
  const HelloView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final text = ref.watch(helloProvider); // อ่านค่าจาก provider
    return Scaffold(body: Center(child: Text(text)));
  }
}

// จากนั้นใน BlaApp เปลี่ยนเป็น →  home: const HelloView(),
```

**ขั้นที่ 4 — รัน**

```bash
flutter run
```

> ✅ **ผลลัพธ์ที่คาดหวัง:** เห็นข้อความ `Hello Riverpod 👋 ...` กลางจอ = `ProviderScope` + Code Generation ทำงานครบวงจรแล้ว
> ⛔ ถ้าเจอ error ว่าหา `_$Hello...` / `helloProvider` ไม่พบ → ตรวจว่าใส่บรรทัด `part '...g.dart';` แล้ว และรัน build_runner แล้วหรือยัง

---

## 📚 Module 3: Understanding Core Providers

### เวลา 13:00–14:15 น.

> ตั้งแต่ Module นี้เป็นต้นไป เราจะลองสร้าง provider จริงในโปรเจกต์ ทุกครั้งที่แก้ไฟล์ที่มี `@riverpod` อย่าลืมให้ `build_runner watch` ทำงานอยู่เบื้องหลัง

---

### 3.1 ภาพรวม Provider ทั้ง 5 ชนิดที่ต้องรู้

Riverpod มี provider หลายชนิด แต่ละชนิดเหมาะกับงานต่างกัน ตารางนี้คือแผนที่ความเข้าใจของทั้งวัน:

| ชนิด Provider          | ใช้เมื่อ                                               | ค่าที่คืน                |
| ---------------------- | ------------------------------------------------------ | ------------------------ |
| `Provider`             | ค่าคงที่/ออบเจกต์ที่ไม่เปลี่ยน เช่น Repository, config | ค่าใด ๆ (อ่านอย่างเดียว) |
| `StateProvider`        | State ง่าย ๆ ที่เปลี่ยนได้ เช่น ตัวเลข, ข้อความค้นหา   | ค่าที่แก้ไขได้ตรง ๆ      |
| `FutureProvider`       | ดึงข้อมูลแบบ async ครั้งเดียว                          | `AsyncValue<T>`          |
| `Notifier` (ใหม่)      | State ที่ซับซ้อนมี Logic หลายเมธอด                     | คลาส State + เมธอด       |
| `AsyncNotifier` (ใหม่) | State async ที่มีทั้งดึงและแก้ไขข้อมูล                 | `AsyncValue<T>` + เมธอด  |

### 3.2 Provider — สำหรับค่า/ออบเจกต์ที่อ่านอย่างเดียว

`Provider` เหมาะกับการสร้างออบเจกต์ที่ใช้ร่วมทั้งแอป เช่น Repository หรือค่าที่คำนวณมาแล้ว

```dart
// แบบ Code Generation (มาตรฐาน 2026)
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'greeting_provider.g.dart';

@riverpod
String greeting(GreetingRef ref) {
  return 'ยินดีต้อนรับสู่ BLA Policy Companion';
}
```

```dart
// แบบเขียนมือ (ให้รู้จักไว้ เผื่อเจอโค้ดเก่า)
final greetingProvider = Provider<String>((ref) {
  return 'ยินดีต้อนรับสู่ BLA Policy Companion';
});
```

> 🧪 **ทดสอบเรียกใช้งาน — `Provider`**
> **ขั้นที่ 1** สร้างไฟล์ provider ด้านบน (`greeting_provider.dart`) แล้วรัน `dart run build_runner build --delete-conflicting-outputs`
> **ขั้นที่ 2** อ่านค่าด้วย widget แล้วตั้งเป็น `home` ชั่วคราว:
>
> ```dart
> class GreetingView extends ConsumerWidget {
>   const GreetingView({super.key});
>   @override
>   Widget build(BuildContext context, WidgetRef ref) {
>     final msg = ref.watch(greetingProvider);
>     return Scaffold(
>       appBar: AppBar(title: const Text('Provider')),
>       body: Center(child: Text(msg)),
>     );
>   }
> }
> ```
>
> **ผลลัพธ์:** เห็นข้อความทักทายกลางจอ · `Provider` ใช้กับค่าที่ "อ่านอย่างเดียว" จึง **ไม่มี** `.notifier` (แก้ค่าไม่ได้) — เหมาะกับ Repository/config

### 3.3 StateProvider — State ง่าย ๆ ที่แก้ไขได้

เหมาะกับ State ปลายทางที่เป็นค่าเดี่ยว เช่น คำค้นหาในช่อง search หรือ index ของ tab ที่เลือก

```dart
// state_provider_demo.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'state_provider_demo.g.dart';

// เก็บคำค้นหาในหน้ารายการกรมธรรม์ (ค่าเริ่มต้นเป็นข้อความว่าง)
@riverpod
class SearchKeyword extends _$SearchKeyword {
  @override
  String build() => '';

  void update(String keyword) => state = keyword;
  void clear() => state = '';
}
```

> 💡 **อัปเดตล่าสุด:** ในปี 2026 ทีม Riverpod แนะนำให้ใช้ `Notifier` (ผ่าน `@riverpod class`) แทน `StateProvider` แบบเก่า เพราะควบคุม Logic การแก้ไข State ได้ชัดเจนกว่าและทดสอบง่ายกว่า — เราจึงเขียน `SearchKeyword` เป็นคลาส Notifier ตั้งแต่ต้น

> 🧪 **ทดสอบเรียกใช้งาน — State ที่แก้ไขได้ (`watch` + `read.notifier`)**
> พิมพ์ในช่องแล้วดูค่าที่อัปเดตแบบ real-time — พิสูจน์ว่า `watch` สั่ง rebuild เมื่อ state เปลี่ยน:
>
> ```dart
> class SearchDemo extends ConsumerWidget {
>   const SearchDemo({super.key});
>   @override
>   Widget build(BuildContext context, WidgetRef ref) {
>     final keyword = ref.watch(searchKeywordProvider); // ฟังค่าปัจจุบัน
>     return Scaffold(
>       body: Padding(
>         padding: const EdgeInsets.all(16),
>         child: Column(
>           mainAxisAlignment: MainAxisAlignment.center,
>           children: [
>             TextField(
>               decoration: const InputDecoration(hintText: 'พิมพ์คำค้นหา...'),
>               // เรียกเมธอดใน callback → ใช้ read (ห้ามใช้ watch)
>               onChanged: (v) =>
>                   ref.read(searchKeywordProvider.notifier).update(v),
>             ),
>             const SizedBox(height: 16),
>             Text('คำค้นหาปัจจุบัน: "$keyword"'),
>             TextButton(
>               onPressed: () =>
>                   ref.read(searchKeywordProvider.notifier).clear(),
>               child: const Text('ล้างคำค้นหา'),
>             ),
>           ],
>         ),
>       ),
>     );
>   }
> }
> ```
>
> **ผลลัพธ์:** พิมพ์แล้วบรรทัด "คำค้นหาปัจจุบัน" เปลี่ยนทันที · กด "ล้างคำค้นหา" แล้วค่าเป็นว่าง — เพราะ `Text` ผูกกับ `watch(searchKeywordProvider)` ส่วนปุ่ม/ช่องพิมพ์เรียกเมธอดผ่าน `read(...notifier)`

### 3.4 FutureProvider — ดึงข้อมูล Async ครั้งเดียว

เมื่อต้องดึงข้อมูลจาก API หรือไฟล์แบบ asynchronous `FutureProvider` จะห่อผลลัพธ์ให้เป็น `AsyncValue` ที่มีสถานะ loading/data/error ครบในตัว (รายละเอียดเชิงลึกของ `AsyncValue` จะเรียนวันที่ 2)

```dart
// future_provider_demo.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'future_provider_demo.g.dart';

@riverpod
Future<int> policyCount(PolicyCountRef ref) async {
  // จำลองการดึงข้อมูลจาก API ที่ใช้เวลา
  await Future<void>.delayed(const Duration(seconds: 1));
  return 12; // จำนวนกรมธรรม์ทั้งหมด (ข้อมูลจำลอง)
}
```

> 🧪 **ทดสอบเรียกใช้งาน — Async ด้วย `AsyncValue.when()`**
> `FutureProvider` คืน `AsyncValue<int>` ที่มีสถานะ loading/data/error ครบในตัว — ใช้ `.when()` แยกทั้ง 3 สถานะ โดยไม่ต้องเขียน `setState` เอง:
>
> ```dart
> class PolicyCountView extends ConsumerWidget {
>   const PolicyCountView({super.key});
>   @override
>   Widget build(BuildContext context, WidgetRef ref) {
>     final asyncCount = ref.watch(policyCountProvider);
>     return Scaffold(
>       body: Center(
>         child: asyncCount.when(
>           loading: () => const CircularProgressIndicator(),
>           error: (e, _) => Text('ผิดพลาด: $e'),
>           data: (count) => Text('มีกรมธรรม์ทั้งหมด $count ฉบับ',
>               style: const TextStyle(fontSize: 20)),
>         ),
>       ),
>     );
>   }
> }
> ```
>
> **ผลลัพธ์:** เปิดหน้าแล้วเห็น spinner ~1 วินาที (ช่วง `loading`) → เปลี่ยนเป็น "มีกรมธรรม์ทั้งหมด 12 ฉบับ" (ช่วง `data`) · ลองเปลี่ยน `return 12;` เป็น `throw 'เน็ตล่ม';` เพื่อดูสาขา `error`

### 3.5 การเปลี่ยนผ่านสู่ Notifier และ AsyncNotifier (มาตรฐานใหม่)

`Notifier` และ `AsyncNotifier` คือทิศทางหลักของ Riverpod ยุคใหม่ ใช้แทน `StateNotifier`/`ChangeNotifier` เดิม จุดเด่นคือรวม "State" และ "เมธอดที่แก้ State" ไว้ในคลาสเดียว อ่านง่ายและทดสอบง่าย

```dart
// counter_notifier.dart — ตัวอย่าง Notifier แบบ sync
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter_notifier.g.dart';

@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0; // ค่าเริ่มต้น

  void increment() => state = state + 1;
  void decrement() => state = state - 1;
  void reset() => state = 0;
}
```

```
สรุปการเลือกใช้:
┌──────────────────────────────────────────────────────────┐
│ ข้อมูลไม่เปลี่ยน (Repo/config)      → Provider             │
│ State เดี่ยว แก้ไขได้ (sync)        → Notifier             │
│ ข้อมูล async ดึงครั้งเดียว           → FutureProvider        │
│ ข้อมูล async + แก้ไขข้อมูลได้        → AsyncNotifier (วันที่ 2)│
└──────────────────────────────────────────────────────────┘
```

> 🧪 **ทดสอบเรียกใช้งาน — `Notifier` (นับเลข +/−/รีเซ็ต)**
> ทดสอบว่า Logic ทั้งหมด (increment/decrement/reset) อยู่ในคลาส `Counter` ส่วน UI แค่เรียกผ่าน `.notifier`:
>
> ```dart
> class CounterDemo extends ConsumerWidget {
>   const CounterDemo({super.key});
>   @override
>   Widget build(BuildContext context, WidgetRef ref) {
>     final count = ref.watch(counterProvider);
>     return Scaffold(
>       body: Center(
>         child: Text('$count', style: const TextStyle(fontSize: 48)),
>       ),
>       floatingActionButton: Row(
>         mainAxisAlignment: MainAxisAlignment.end,
>         children: [
>           // หลาย FAB ในจอเดียวต้องใส่ heroTag ต่างกัน
>           FloatingActionButton(
>               heroTag: 'dec',
>               onPressed: () => ref.read(counterProvider.notifier).decrement(),
>               child: const Icon(Icons.remove)),
>           const SizedBox(width: 12),
>           FloatingActionButton(
>               heroTag: 'reset',
>               onPressed: () => ref.read(counterProvider.notifier).reset(),
>               child: const Icon(Icons.refresh)),
>           const SizedBox(width: 12),
>           FloatingActionButton(
>               heroTag: 'inc',
>               onPressed: () => ref.read(counterProvider.notifier).increment(),
>               child: const Icon(Icons.add)),
>         ],
>       ),
>     );
>   }
> }
> ```
>
> **ผลลัพธ์:** กด + / − / รีเซ็ต แล้วตัวเลขกลางจอเปลี่ยนตามทันที — จุดสำคัญคือ UI ไม่มีสูตรคำนวณเลย เพียงสั่ง `.increment()` ฯลฯ ผ่าน `.notifier`

---

## 📚 Module 4: Consuming State (การบริโภค State)

### เวลา 14:15–15:00 น.

> 💡 **หัวใจของ Module นี้:** การประกาศ Provider เป็นเพียงครึ่งเดียว อีกครึ่งคือการ "อ่าน" State มาแสดงผลอย่างถูกวิธี ซึ่งกุญแจอยู่ที่การเลือกใช้ `ref.watch`, `ref.read`, `ref.listen` ให้ถูกสถานการณ์

---

### 4.1 ConsumerWidget — Widget ที่อ่าน Provider ได้

แทนที่จะใช้ `StatelessWidget` เราใช้ `ConsumerWidget` ซึ่งเพิ่มพารามิเตอร์ `WidgetRef ref` เข้ามาในเมธอด `build` เพื่อใช้เข้าถึง provider

```dart
// consumer_demo.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class CounterView extends ConsumerWidget {
  const CounterView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // watch → UI จะ rebuild อัตโนมัติเมื่อค่าเปลี่ยน
    final count = ref.watch(counterProvider);

    return Scaffold(
      body: Center(child: Text('นับได้: $count')),
      floatingActionButton: FloatingActionButton(
        // read → เรียกเมธอดแบบครั้งเดียว ไม่ subscribe การเปลี่ยนแปลง
        onPressed: () => ref.read(counterProvider.notifier).increment(),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

> 🧪 **ทดสอบเรียกใช้งาน**
> **ขั้นที่ 1** ต้องมี `counterProvider` (จากหัวข้อ 3.5) แล้วรัน build_runner
> **ขั้นที่ 2** ตั้ง `home: const CounterView(),` ใน `BlaApp` → `flutter run`
> **ผลลัพธ์:** กดปุ่ม **+** (FAB) แล้วเลข "นับได้: N" เพิ่มขึ้นทันที · สังเกตกฎ: อ่านค่าเพื่อ **แสดงผล** ใช้ `watch`, เรียกเมธอดใน **`onPressed`** ใช้ `read`

### 4.2 ConsumerStatefulWidget — เมื่อต้องมี State ของ Widget เอง

ถ้า Widget ต้องมี `initState`, `dispose` หรือ controller (เช่น `TextEditingController`) ให้ใช้ `ConsumerStatefulWidget` ซึ่งเข้าถึง `ref` ได้ทั้งใน `build` และ lifecycle methods

```dart
class SearchBox extends ConsumerStatefulWidget {
  const SearchBox({super.key});

  @override
  ConsumerState<SearchBox> createState() => _SearchBoxState();
}

class _SearchBoxState extends ConsumerState<SearchBox> {
  final _controller = TextEditingController();

  @override
  void dispose() {
    _controller.dispose(); // คืนหน่วยความจำเสมอ
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return TextField(
      controller: _controller,
      onChanged: (value) => ref.read(searchKeywordProvider.notifier).update(value),
      decoration: const InputDecoration(hintText: 'ค้นหากรมธรรม์...'),
    );
  }
}
```

> 🧪 **ทดสอบเรียกใช้งาน — state แชร์ข้าม widget**
> ห่อ `SearchBox` (มี controller ของตัวเอง) กับบรรทัดแสดงผลไว้ด้วยกัน เพื่อเห็นว่า state ถูกแชร์ผ่าน provider:
>
> ```dart
> class SearchScreen extends ConsumerWidget {
>   const SearchScreen({super.key});
>   @override
>   Widget build(BuildContext context, WidgetRef ref) {
>     final keyword = ref.watch(searchKeywordProvider);
>     return Scaffold(
>       appBar: AppBar(title: const Text('ConsumerStatefulWidget')),
>       body: Column(
>         children: [
>           const Padding(padding: EdgeInsets.all(16), child: SearchBox()),
>           Text('กำลังค้นหา: "$keyword"'),
>         ],
>       ),
>     );
>   }
> }
> ```
>
> **ผลลัพธ์:** พิมพ์ใน `SearchBox` แล้วข้อความด้านล่างอัปเดตตาม — `SearchBox` ส่งค่าเข้า provider ด้วย `read(...notifier)`, ส่วน `SearchScreen` ฟังค่ากลับด้วย `watch` · เห็นชัดว่าทั้งสอง widget "คุย" กันผ่าน provider ไม่ใช่ส่ง callback ตรง ๆ

### 4.3 ความแตกต่างของ ref.watch / ref.read / ref.listen

นี่คือหัวใจที่ผู้เรียนต้องแยกให้ออก เพราะใช้ผิดคือต้นเหตุของบั๊ก rebuild และ performance ที่พบบ่อยที่สุด

| เมธอด          | ใช้เมื่อ                                                   | rebuild UI? | ใช้ที่ไหน                   |
| -------------- | ---------------------------------------------------------- | ----------- | --------------------------- |
| `ref.watch()`  | ต้องการ "ฟัง" ค่าและให้ UI อัปเดตตาม                       | ✅ ใช่      | ใน `build()` เท่านั้น       |
| `ref.read()`   | เรียกเมธอด/อ่านค่าครั้งเดียว ไม่ต้องฟัง                    | ❌ ไม่      | ใน callback (onPressed ฯลฯ) |
| `ref.listen()` | ทำ side-effect เมื่อค่าเปลี่ยน (เช่น แสดง SnackBar, นำทาง) | ❌ ไม่      | ใน `build()`                |

```dart
// ตัวอย่างการใช้ ref.listen ทำ side-effect (ไม่เกี่ยวกับการแสดงผล)
@override
Widget build(BuildContext context, WidgetRef ref) {
  // เมื่อ state เปลี่ยน ให้แสดง SnackBar — ไม่ใช่การ rebuild ค่าเพื่อแสดงผล
  ref.listen<int>(counterProvider, (previous, next) {
    if (next == 10) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('ครบ 10 แล้ว!')),
      );
    }
  });

  final count = ref.watch(counterProvider);
  return Text('$count');
}
```

> ⚠️ **กฎทอง:** อย่าใช้ `ref.watch()` ใน `onPressed` หรือ callback เด็ดขาด เพราะจะทำให้เกิดการ subscribe ซ้ำซ้อนและพฤติกรรมผิดเพี้ยน — ใน callback ให้ใช้ `ref.read()` เสมอ ส่วนใน `build()` ที่ต้องการแสดงผลให้ใช้ `ref.watch()`

> 🧪 **ทดสอบเรียกใช้งาน — `watch` + `read` + `listen` ในหน้าเดียว**
> รวมทั้งสามบทบาทเข้าด้วยกัน เพื่อเห็นความต่างชัด ๆ (ใช้ `counterProvider` จากหัวข้อ 3.5):
>
> ```dart
> class WatchReadListenDemo extends ConsumerWidget {
>   const WatchReadListenDemo({super.key});
>   @override
>   Widget build(BuildContext context, WidgetRef ref) {
>     // listen → side-effect (เด้ง SnackBar) ไม่ทำให้ rebuild
>     ref.listen<int>(counterProvider, (prev, next) {
>       if (next == 10) {
>         ScaffoldMessenger.of(context).showSnackBar(
>           const SnackBar(content: Text('🎉 ครบ 10 แล้ว!')),
>         );
>       }
>     });
>     // watch → ฟังค่าเพื่อแสดงผล (rebuild เมื่อเปลี่ยน)
>     final count = ref.watch(counterProvider);
>     return Scaffold(
>       body: Center(
>         child: Text('นับได้: $count', style: const TextStyle(fontSize: 32)),
>       ),
>       floatingActionButton: FloatingActionButton(
>         // read → เรียกเมธอดครั้งเดียว ไม่ subscribe
>         onPressed: () => ref.read(counterProvider.notifier).increment(),
>         child: const Icon(Icons.add),
>       ),
>     );
>   }
> }
> ```
>
> **ผลลัพธ์:** กด + เรื่อย ๆ เลขเพิ่มขึ้น (มาจาก `watch`) และเมื่อถึง **10** จะเด้ง SnackBar หนึ่งครั้ง (มาจาก `listen`) โดยปุ่มเรียกเมธอดผ่าน `read` — ครบทั้งสามบทบาทในหน้าจอเดียว

---

## 🛠️ Workshop Day 1 — BLA Policy Companion: วาง Feature-first + State หน้ารายการกรมธรรม์

### เวลา 15:00–16:00 น.

> **โจทย์:** วางโครงสร้างโปรเจกต์แบบ Feature-first สำหรับฟีเจอร์ `policy` สร้างโมเดล `Policy` (Domain), ทำ Repository ที่คืนข้อมูลจำลอง (Data), จัดการ State ด้วย `Notifier` ที่รองรับการค้นหาและกรองสถานะ (Presentation) แล้วแสดงรายการกรมธรรม์บนหน้าจอจริง

### ขั้นที่ 1 — วางโครงสร้างโฟลเดอร์ Feature-first

โครงสร้างนี้ตรงกับโปรเจกต์จริง (branch `day1`) — แยก `core/` (ของใช้ร่วมทั้งแอป: ธีม/ไอคอน/วิดเจ็ต) ออกจาก `features/` (แต่ละฟีเจอร์มี domain/data/presentation ของตัวเอง)

```
lib/
├── main.dart                          ← ProviderScope + MaterialApp (ธีม + status bar scrim)
├── core/                              ← โครงสร้างพื้นฐานใช้ร่วมทั้งแอป
│   ├── theme/
│   │   ├── app_colors.dart            ← โทเคนสี (Design System)
│   │   ├── app_spacing.dart           ← โทเคนระยะห่าง/มุมโค้ง
│   │   ├── app_typography.dart        ← สไตล์ตัวอักษร (Inter + Anuphan)
│   │   └── app_theme.dart             ← ประกอบ ThemeData กลาง
│   ├── icons/
│   │   └── app_icons.dart             ← ครอบ Hugeicons (wrapper + AppIconData)
│   └── widgets/                       ← วิดเจ็ตกลาง reuse ได้
│       ├── app_button.dart            ← ปุ่ม 3 แบบ (primary/secondary/social)
│       ├── app_card.dart              ← การ์ดขาว เงานุ่ม
│       ├── app_logo.dart              ← โลโก้ BLA (Splash/Login)
│       ├── app_text_field.dart        ← ช่องกรอก + label + ซ่อนรหัสผ่าน
│       ├── gradient_header.dart       ← หัวหน้าจอ gradient + CircleHeaderButton
│       ├── info_hint.dart             ← กล่องคำใบ้เล็ก ๆ
│       ├── section_label.dart         ← หัวข้อ section
│       └── status_badge.dart          ← ป้ายสถานะ (ใช้ร่วม 3 ฟีเจอร์)
└── features/
    ├── app/
    │   └── main_shell.dart            ← เปลือกแอป: Bottom Navigation 5 แท็บ
    ├── auth/presentation/pages/
    │   ├── splash_page.dart           ← จุดเริ่ม: โลโก้ + หน่วงเวลา → Login
    │   ├── login_page.dart            ← เข้าสู่ระบบ → MainShell
    │   ├── register_page.dart         ← สมัครสมาชิก
    │   ├── forgot_password_page.dart  ← ลืมรหัสผ่าน → Reset
    │   └── reset_password_page.dart   ← ตั้งรหัสผ่านใหม่
    ├── home/presentation/pages/
    │   └── home_page.dart             ← Dashboard (แท็บ 0)
    ├── policy/                        ← ฟีเจอร์แกนของ Workshop (แท็บ 1)
    │   ├── domain/
    │   │   └── policy.dart            ← โมเดลแก่นกลาง (Entity)
    │   ├── data/
    │   │   ├── policy_repository.dart ← แหล่งข้อมูล (Mock)
    │   │   └── policy_repository.g.dart        ← (build_runner สร้าง)
    │   └── presentation/
    │       ├── controllers/
    │       │   ├── policy_list_controller.dart ← Provider/Notifier จัดการ State
    │       │   └── policy_list_controller.g.dart ← (build_runner สร้าง)
    │       └── pages/
    │           ├── policy_list_page.dart       ← หน้ารายการ (ค้นหา+กรอง)
    │           └── policy_detail_page.dart     ← หน้ารายละเอียด (รับ Policy)
    ├── claim/presentation/
    │   ├── pages/claims_page.dart     ← หน้าสินไหม (แท็บ 2)
    │   └── widgets/file_claim_sheet.dart ← Bottom Sheet ยื่นสินไหม (ใช้ร่วม 3 ที่)
    ├── payment/presentation/pages/
    │   └── payments_page.dart         ← หน้าชำระเบี้ย (แท็บ 3)
    └── profile/presentation/pages/
        └── profile_page.dart          ← หน้าโปรไฟล์ (แท็บ 4)
```

> 🗂️ **หมายเหตุ:** นี่คือโครงสร้างจริงครบทุกไฟล์ใน branch `day1` — Workshop ช่วงนี้เราโฟกัสที่ฟีเจอร์ **policy** เป็นแกนก่อน (ขั้นที่ 2–8) จากนั้นดูโค้ดจริงของหน้าที่เหลือทั้งหมดใน **Part B** ท้ายเอกสาร ทุกหน้าใช้ Design System และวิดเจ็ตกลางชุดเดียวกัน

### ขั้นที่ 2 — สร้างโมเดล Policy (ชั้น Domain)

ในวันแรกเราเขียนโมเดลแบบมือก่อนเพื่อให้เข้าใจโครงสร้างชัด ๆ (วันที่ 2 จะอัปเกรดเป็น `freezed`)

```dart
// lib/features/policy/domain/policy.dart

// สถานะของกรมธรรม์
enum PolicyStatus { active, lapsed, pending }

/// โมเดลกรมธรรม์ (ชั้น Domain)
///
/// วันที่ 1 เขียนแบบมือก่อนเพื่อให้เข้าใจโครงสร้างชัด ๆ
/// วันที่ 2 จะอัปเกรดเป็น freezed + JSON (immutable + serialization)
class Policy {
  final String id;
  final String planName; // ชื่อแผนประกัน
  final String policyNumber; // เลขที่กรมธรรม์
  final double premium; // เบี้ยประกัน (บาท/ปี)
  final PolicyStatus status;

  const Policy({
    required this.id,
    required this.planName,
    required this.policyNumber,
    required this.premium,
    required this.status,
  });
}

// ส่วนขยายเพื่อแปลงสถานะเป็นข้อความภาษาไทย (กฎทางธุรกิจอยู่ในชั้น Domain)
extension PolicyStatusLabel on PolicyStatus {
  String get label {
    switch (this) {
      case PolicyStatus.active:
        return 'มีผลบังคับ';
      case PolicyStatus.lapsed:
        return 'ขาดอายุ';
      case PolicyStatus.pending:
        return 'รอดำเนินการ';
    }
  }
}
```

### ขั้นที่ 3 — สร้าง Repository คืนข้อมูลจำลอง (ชั้น Data)

Repository คือ "ประตู" สู่ข้อมูล วันนี้คืนข้อมูลจำลองในเครื่อง วันที่ 2 เราจะเปลี่ยนข้างในให้เรียก API จริงโดยที่ชั้นบนไม่ต้องแก้

```dart
// lib/features/policy/data/policy_repository.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

import '../domain/policy.dart';

part 'policy_repository.g.dart';

/// Repository คือ "ประตู" สู่ข้อมูล
///
/// วันที่ 1 คืนข้อมูลจำลองในเครื่อง (mock)
/// วันที่ 2 จะเปลี่ยนภายในให้เรียก API จริงผ่าน Dio โดยที่ชั้นบนไม่ต้องแก้
class PolicyRepository {
  // ดึงรายการกรมธรรม์ (จำลอง delay เหมือนเรียก API จริง)
  Future<List<Policy>> fetchPolicies() async {
    await Future<void>.delayed(const Duration(milliseconds: 600));
    return const [
      Policy(
        id: 'P001',
        planName: 'BLA ตลอดชีพ มั่นคง 99',
        policyNumber: 'BLA-2569-0001',
        premium: 24000,
        status: PolicyStatus.active,
      ),
      Policy(
        id: 'P002',
        planName: 'BLA สะสมทรัพย์ 10/5',
        policyNumber: 'BLA-2569-0002',
        premium: 50000,
        status: PolicyStatus.pending,
      ),
      Policy(
        id: 'P003',
        planName: 'BLA คุ้มครองสุขภาพ พลัส',
        policyNumber: 'BLA-2569-0003',
        premium: 18500,
        status: PolicyStatus.lapsed,
      ),
      Policy(
        id: 'P004',
        planName: 'BLA บำนาญมั่นคง 60',
        policyNumber: 'BLA-2569-0004',
        premium: 36000,
        status: PolicyStatus.active,
      ),
    ];
  }
}

// ฉีด Repository ผ่าน Provider — ชั้นบนเรียกใช้ผ่าน ref โดยไม่ต้องสร้างเอง
@riverpod
PolicyRepository policyRepository(PolicyRepositoryRef ref) {
  return PolicyRepository();
}
```

### ขั้นที่ 4 — จัดการ State ด้วย Notifier (ชั้น Presentation)

นี่คือหัวใจของ workshop วันแรก — เราใช้ `AsyncNotifier` เพื่อดึงข้อมูลจาก repository แล้วเก็บไว้ พร้อมเมธอดค้นหาและกรองสถานะ

```dart
// lib/features/policy/presentation/controllers/policy_list_controller.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

import '../../data/policy_repository.dart';
import '../../domain/policy.dart';

part 'policy_list_controller.g.dart';

// 1) เก็บคำค้นหา (Notifier sync ง่าย ๆ)
@riverpod
class PolicySearch extends _$PolicySearch {
  @override
  String build() => '';
  void update(String keyword) => state = keyword.trim();
}

// 2) เก็บตัวกรองสถานะ (null = ทั้งหมด)
@riverpod
class PolicyStatusFilter extends _$PolicyStatusFilter {
  @override
  PolicyStatus? build() => null;
  void select(PolicyStatus? status) => state = status;
}

// 3) ดึงรายการกรมธรรม์ทั้งหมดจาก repository (FutureProvider)
@riverpod
Future<List<Policy>> policyList(PolicyListRef ref) async {
  final repository = ref.watch(policyRepositoryProvider);
  return repository.fetchPolicies();
}

// 4) รายการที่ "ผ่านการกรองแล้ว" — รวมข้อมูล + คำค้นหา + ตัวกรองสถานะ
//    provider นี้ watch ทั้ง 3 ตัว เมื่อตัวใดเปลี่ยน UI จะอัปเดตเอง
@riverpod
List<Policy> filteredPolicyList(FilteredPolicyListRef ref) {
  final asyncPolicies = ref.watch(policyListProvider);
  final keyword = ref.watch(policySearchProvider).toLowerCase();
  final statusFilter = ref.watch(policyStatusFilterProvider);

  // ถ้าข้อมูลยังโหลดไม่เสร็จ ให้คืนลิสต์ว่างไปก่อน
  final policies = asyncPolicies.value ?? <Policy>[];

  return policies.where((policy) {
    final matchKeyword = keyword.isEmpty ||
        policy.planName.toLowerCase().contains(keyword) ||
        policy.policyNumber.toLowerCase().contains(keyword);
    final matchStatus = statusFilter == null || policy.status == statusFilter;
    return matchKeyword && matchStatus;
  }).toList();
}
```

### ขั้นที่ 5 — วาง Design System (core/theme)

โปรเจกต์จริงไม่ใช้สี/ฟอนต์กระจัดกระจาย แต่รวมเป็น **โทเคน (token)** ไว้ที่เดียว แก้ทีเดียวเปลี่ยนทั้งแอป — เริ่มจาก 4 ไฟล์ในโฟลเดอร์ `core/theme/`

```dart
// lib/core/theme/app_colors.dart
import 'package:flutter/material.dart';

/// โทเคนสีของ BLA Design System (อ้างอิงจาก mockup)
/// แยกเป็น token กลาง: แก้ที่เดียวเปลี่ยนทั้งระบบ
abstract final class AppColors {
  // แบรนด์หลัก
  static const Color primary = Color(0xFF1855A3);
  static const Color primaryLight = Color(0xFF2D87D8);
  static const Color primaryDark = Color(0xFF0F2D6B);
  static const Color cyan = Color(0xFF00AEEF);
  static const Color lime = Color(0xFF8DC63F);

  // พื้นหลัง / พื้นผิว / เส้น
  static const Color background = Color(0xFFEFF4FB);
  static const Color card = Color(0xFFFFFFFF);
  static const Color border = Color(0xFFD5E1F0);

  // ตัวอักษร
  static const Color textDark = Color(0xFF1A2B4A);
  static const Color textMedium = Color(0xFF4A5A78);
  static const Color textSubtle = Color(0xFF8896B0);

  // สถานะ: มีผลบังคับ / อนุมัติ (เขียว)
  static const Color successFg = Color(0xFF16A34A);
  static const Color successBg = Color(0xFFDCFCE7);
  static const Color successText = Color(0xFF166534);

  // สถานะ: รอดำเนินการ / กำลังพิจารณา (เหลือง-ส้ม)
  static const Color pendingFg = Color(0xFFD97706);
  static const Color pendingBg = Color(0xFFFEF9C3);
  static const Color pendingText = Color(0xFF92400E);

  // สถานะ: ขาดอายุ / ไม่อนุมัติ (แดง)
  static const Color dangerFg = Color(0xFFDC2626);
  static const Color dangerBg = Color(0xFFFEE2E2);
  static const Color dangerText = Color(0xFF991B1B);

  // ข้อมูล / จ่ายแล้ว (ฟ้า)
  static const Color infoBg = Color(0xFFE0F4FD);
  static const Color infoText = Color(0xFF075985);

  // Gradient หลัก (135°)
  static const Gradient primaryGradient = LinearGradient(
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
    colors: [primaryLight, primary],
  );

  // Gradient เข้ม (หัวหน้าจอใหญ่)
  static const Gradient headerGradient = LinearGradient(
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
    colors: [primaryLight, primary, primaryDark],
    stops: [0.0, 0.55, 1.0],
  );
}
```

```dart
// lib/core/theme/app_spacing.dart
import 'package:flutter/widgets.dart';

/// โทเคนระยะห่าง / มุมโค้ง ของ Design System
abstract final class AppSpacing {
  static const double xs = 4;
  static const double sm = 8;
  static const double md = 12;
  static const double lg = 16;
  static const double xl = 20;
  static const double xxl = 24;
  static const double xxxl = 32;

  // มุมโค้ง (radius)
  static const double rSm = 10; // input
  static const double rMd = 14; // card / button
  static const double rLg = 20; // การ์ดใหญ่ / sheet
  static const double rPill = 999; // chip / badge

  // EdgeInsets ที่ใช้บ่อย
  static const EdgeInsets pageH = EdgeInsets.symmetric(horizontal: lg);
  static const EdgeInsets card = EdgeInsets.all(lg);
}
```

```dart
// lib/core/theme/app_typography.dart
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';

import 'app_colors.dart';

/// ตัวอักษรของ Design System
/// ใช้ Inter (อังกฤษ/ตัวเลข) ผสม Anuphan (ไทย) — Anuphan เป็น fallback ที่รองรับไทยสวย
/// รวมไว้ที่เดียวเพื่อให้ขนาด/น้ำหนักสม่ำเสมอทั้งแอป
abstract final class AppType {
  /// ฟอนต์ไทย Anuphan ใช้เป็น fallback ให้ glyph ภาษาไทย
  /// (Inter ไม่มี glyph ไทย → ตัวอักษรไทยจะตกมาใช้ Anuphan, อังกฤษ/ตัวเลขใช้ Inter)
  static final List<String> _thaiFallback = [GoogleFonts.anuphan().fontFamily!];

  static TextStyle _base(double size, FontWeight weight, {Color? color, double? height}) {
    return GoogleFonts.inter(
      fontSize: size,
      fontWeight: weight,
      color: color ?? AppColors.textDark,
      height: height,
    ).copyWith(fontFamilyFallback: _thaiFallback);
  }

  // หัวเรื่องใหญ่ (ตัวเลขวงเงิน ฯลฯ)
  static TextStyle get display => _base(28, FontWeight.w700);
  static TextStyle get h1 => _base(22, FontWeight.w700);
  static TextStyle get h2 => _base(18, FontWeight.w700);
  static TextStyle get h3 => _base(16, FontWeight.w600);
  static TextStyle get title => _base(15, FontWeight.w600);
  static TextStyle get body => _base(14, FontWeight.w400, color: AppColors.textMedium, height: 1.4);
  static TextStyle get bodyStrong => _base(14, FontWeight.w600);
  static TextStyle get label => _base(13, FontWeight.w500, color: AppColors.textMedium);
  static TextStyle get caption => _base(12, FontWeight.w400, color: AppColors.textSubtle);
  static TextStyle get tiny => _base(11, FontWeight.w500, color: AppColors.textSubtle);
}
```

```dart
// lib/core/theme/app_theme.dart
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';

import 'app_colors.dart';
import 'app_spacing.dart';

/// ThemeData กลางของแอป ประกอบจาก token ทั้งหมด
abstract final class AppTheme {
  static ThemeData get light {
    final base = ThemeData(
      useMaterial3: true,
      colorScheme: ColorScheme.fromSeed(
        seedColor: AppColors.primary,
        primary: AppColors.primary,
        surface: AppColors.card,
      ),
      scaffoldBackgroundColor: AppColors.background,
    );

    return base.copyWith(
      textTheme: GoogleFonts.interTextTheme(base.textTheme).apply(
        bodyColor: AppColors.textDark,
        displayColor: AppColors.textDark,
        fontFamilyFallback: [GoogleFonts.anuphan().fontFamily!],
      ),
      appBarTheme: const AppBarTheme(
        backgroundColor: Colors.transparent,
        elevation: 0,
        scrolledUnderElevation: 0,
        foregroundColor: Colors.white,
      ),
      cardTheme: CardThemeData(
        color: AppColors.card,
        elevation: 0,
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(AppSpacing.rMd),
        ),
      ),
      inputDecorationTheme: InputDecorationTheme(
        filled: true,
        fillColor: AppColors.card,
        contentPadding: const EdgeInsets.symmetric(horizontal: 14, vertical: 14),
        border: OutlineInputBorder(
          borderRadius: BorderRadius.circular(AppSpacing.rSm),
          borderSide: const BorderSide(color: AppColors.border),
        ),
        enabledBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(AppSpacing.rSm),
          borderSide: const BorderSide(color: AppColors.border),
        ),
        focusedBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(AppSpacing.rSm),
          borderSide: const BorderSide(color: AppColors.primary, width: 1.6),
        ),
        hintStyle: const TextStyle(color: AppColors.textSubtle),
      ),
      dividerTheme: const DividerThemeData(color: AppColors.border, thickness: 1),
    );
  }
}
```

### ขั้นที่ 6 — ระบบไอคอน + วิดเจ็ตกลาง (core/icons, core/widgets)

เราครอบแพ็กเกจ `hugeicons` ด้วย wrapper ของเราเอง เพื่อคุมขนาด/สไตล์ให้สม่ำเสมอทั้งแอป แล้วสร้างวิดเจ็ตกลางที่ใช้ซ้ำ (การ์ด, ป้ายสถานะ, หัวหน้าจอ gradient)

```dart
// lib/core/icons/app_icons.dart

/// จุดรวมไอคอนของแอป — ใช้ Hugeicons (สไตล์ stroke rounded) ทั้งแอป
///
/// - ใช้ผ่าน widget `HugeIcon(icon: HugeIcons.strokeRoundedHome03, ...)`
/// - ชนิดข้อมูลไอคอนของ Hugeicons คือ `List<List<dynamic>>` (ข้อมูล SVG path)
///   จึงนิยามเป็น typedef [AppIconData] เพื่อใช้เป็น type ใน widget ต่าง ๆ
///
/// หมายเหตุเรื่องขนาด: ตัว `HugeIcon` ของแพ็กเกจวาด SVG เต็มกรอบ `size` พอดี
/// (ไม่มี padding ในตัว) ต่างจาก Material `Icon` ที่ glyph กินพื้นที่จริงราว 83%
/// ของกรอบ ทำให้ที่ขนาด px เท่ากัน hugeicon จะดู "ใหญ่/หนา" กว่า
/// เราจึงห่อด้วย [HugeIcon] ของเราเองที่คูณค่าชดเชย [kIconOpticalScale]
/// เพื่อให้สมดุลกับงานดีไซน์เดิม โดยโค้ดเรียกใช้ยังเขียน `HugeIcon(...)` เหมือนเดิม
library;

import 'package:flutter/widgets.dart';
import 'package:hugeicons/hugeicons.dart' as hi;

// ส่งออกรายชื่อไอคอน (HugeIcons.strokeRounded*) ให้ทั้งแอปใช้ได้
// แต่ซ่อน HugeIcon ตัวจริงของแพ็กเกจ เพื่อใช้ wrapper ของเราแทน
export 'package:hugeicons/hugeicons.dart' hide HugeIcon;

/// ชนิดข้อมูลไอคอน Hugeicons (ใช้แทน IconData เดิม)
typedef AppIconData = List<List<dynamic>>;

/// ค่าชดเชยเชิงทัศนภาพ ให้ hugeicon ดูสมดุลเทียบเท่า Material icon ขนาดเดียวกัน
/// (Material glyph กินพื้นที่ราว 0.83 ของกรอบ — ปรับค่านี้เพื่อจูนความใหญ่ทั้งแอป)
const double kIconOpticalScale = 0.84;

/// ไอคอนของแอป — ห่อ `HugeIcon` ของแพ็กเกจ พร้อมชดเชยขนาดให้สมดุล
///
/// ใช้ชื่อ `HugeIcon` เพื่อให้โค้ดเดิมที่เขียน `HugeIcon(icon: ..., size: ...)`
/// ทำงานได้ทันทีโดยไม่ต้องแก้ไข
class HugeIcon extends StatelessWidget {
  final AppIconData icon;
  final Color? color;
  final Color? secondaryColor;
  final double? size;
  final double? strokeWidth;

  const HugeIcon({
    super.key,
    required this.icon,
    this.color,
    this.secondaryColor,
    this.size,
    this.strokeWidth,
  });

  @override
  Widget build(BuildContext context) {
    final base = size ?? IconTheme.of(context).size ?? 24.0;
    // ห่อด้วย Center(widthFactor/heightFactor: 1) เพื่อให้:
    //  - constraints หลวม → shrink-wrap ตามขนาดไอคอนจริง
    //  - constraints ตายตัว (อยู่ในกล่อง width/height คงที่) → จัดกึ่งกลางที่ขนาดจริง
    //    แทนการถูก SvgPicture ยืดเต็มกล่อง
    return Center(
      widthFactor: 1,
      heightFactor: 1,
      child: hi.HugeIcon(
        icon: icon,
        color: color,
        secondaryColor: secondaryColor,
        size: base * kIconOpticalScale,
        strokeWidth: strokeWidth,
      ),
    );
  }
}
```

```dart
// lib/core/widgets/app_card.dart
import 'package:flutter/material.dart';

import '../theme/app_colors.dart';
import '../theme/app_spacing.dart';

/// การ์ดพื้นฐานสีขาว เงานุ่ม มุมโค้ง — กล่องห่อเนื้อหาที่ใช้ซ้ำทั้งแอป
class AppCard extends StatelessWidget {
  final Widget child;
  final EdgeInsetsGeometry padding;
  final VoidCallback? onTap;
  final Color? color;

  const AppCard({
    super.key,
    required this.child,
    this.padding = AppSpacing.card,
    this.onTap,
    this.color,
  });

  @override
  Widget build(BuildContext context) {
    final content = Container(
      width: double.infinity,
      padding: padding,
      decoration: BoxDecoration(
        color: color ?? AppColors.card,
        borderRadius: BorderRadius.circular(AppSpacing.rMd),
        border: Border.all(color: AppColors.border, width: 1),
        boxShadow: [
          BoxShadow(
            color: AppColors.primaryDark.withValues(alpha: 0.05),
            blurRadius: 14,
            offset: const Offset(0, 6),
          ),
        ],
      ),
      // Material โปร่งใสด้านในการ์ด เพื่อให้ ListTile/InkWell วาด
      // พื้นหลังและ ink splash บนผิวการ์ดได้ถูกต้อง (ไม่ถูก DecoratedBox บัง)
      child: Material(
        type: MaterialType.transparency,
        child: child,
      ),
    );
    if (onTap == null) return content;
    return Material(
      color: Colors.transparent,
      child: InkWell(
        borderRadius: BorderRadius.circular(AppSpacing.rMd),
        onTap: onTap,
        child: content,
      ),
    );
  }
}
```

```dart
// lib/core/widgets/status_badge.dart
import 'package:flutter/material.dart';

import '../theme/app_colors.dart';
import '../theme/app_spacing.dart';

/// โทนสีของ badge สถานะ
enum BadgeTone { success, pending, danger, info, neutral }

/// ป้ายสถานะ (เช่น มีผลบังคับ / รอดำเนินการ / ขาดอายุ / อนุมัติ)
/// ใช้ร่วมกันทั้งหน้า policy, claim, payment
class StatusBadge extends StatelessWidget {
  final String label;
  final BadgeTone tone;

  const StatusBadge({super.key, required this.label, required this.tone});

  ({Color bg, Color fg}) get _colors {
    switch (tone) {
      case BadgeTone.success:
        return (bg: AppColors.successBg, fg: AppColors.successText);
      case BadgeTone.pending:
        return (bg: AppColors.pendingBg, fg: AppColors.pendingText);
      case BadgeTone.danger:
        return (bg: AppColors.dangerBg, fg: AppColors.dangerText);
      case BadgeTone.info:
        return (bg: AppColors.infoBg, fg: AppColors.infoText);
      case BadgeTone.neutral:
        return (bg: AppColors.background, fg: AppColors.textMedium);
    }
  }

  @override
  Widget build(BuildContext context) {
    final c = _colors;
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 4),
      decoration: BoxDecoration(
        color: c.bg,
        borderRadius: BorderRadius.circular(AppSpacing.rPill),
      ),
      child: Text(
        label,
        style: TextStyle(fontSize: 11.5, fontWeight: FontWeight.w600, color: c.fg),
      ),
    );
  }
}
```

```dart
// lib/core/widgets/gradient_header.dart
import 'package:flutter/material.dart';

import '../icons/app_icons.dart';
import '../theme/app_colors.dart';
import '../theme/app_spacing.dart';
import '../theme/app_typography.dart';

/// หัวหน้าจอพื้น gradient น้ำเงิน มุมล่างโค้ง — ใช้ซ้ำเกือบทุกหน้า
///
/// ปรับแต่งได้: ปุ่มย้อนกลับ, title/subtitle, widget ท้าย (เช่น กระดิ่ง),
/// หรือใส่ [child] เป็นเนื้อหา custom (เช่น โลโก้, การ์ดสรุป)
class GradientHeader extends StatelessWidget {
  final String? title;
  final String? subtitle;
  final bool showBack;
  final Widget? trailing;
  final Widget? leading;
  final Widget? child;
  final EdgeInsetsGeometry padding;

  const GradientHeader({
    super.key,
    this.title,
    this.subtitle,
    this.showBack = false,
    this.trailing,
    this.leading,
    this.child,
    this.padding = const EdgeInsets.fromLTRB(20, 8, 20, 22),
  });

  @override
  Widget build(BuildContext context) {
    return Container(
      width: double.infinity,
      decoration: const BoxDecoration(
        gradient: AppColors.headerGradient,
        borderRadius: BorderRadius.vertical(bottom: Radius.circular(AppSpacing.rLg)),
      ),
      child: SafeArea(
        bottom: false,
        child: Padding(
          padding: padding,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              if (showBack || leading != null || trailing != null)
                Row(
                  children: [
                    if (showBack)
                      _CircleIcon(
                        icon: HugeIcons.strokeRoundedArrowLeft01,
                        onTap: () => Navigator.of(context).maybePop(),
                      ),
                    if (leading != null) ...[
                      if (showBack) const SizedBox(width: 10),
                      leading!,
                    ],
                    const Spacer(),
                    if (trailing != null) trailing!,
                  ],
                ),
              if (title != null) ...[
                if (showBack || leading != null || trailing != null)
                  const SizedBox(height: 12),
                Text(title!, style: AppType.h1.copyWith(color: Colors.white)),
              ],
              if (subtitle != null) ...[
                const SizedBox(height: 4),
                Text(subtitle!,
                    style: AppType.body.copyWith(color: Colors.white70)),
              ],
              if (child != null) child!,
            ],
          ),
        ),
      ),
    );
  }
}

/// ปุ่มไอคอนวงกลมโปร่งแสง (ใช้บนหัว gradient เช่น back / กระดิ่ง)
class CircleHeaderButton extends StatelessWidget {
  final AppIconData icon;
  final VoidCallback? onTap;
  const CircleHeaderButton({super.key, required this.icon, this.onTap});

  @override
  Widget build(BuildContext context) => _CircleIcon(icon: icon, onTap: onTap);
}

class _CircleIcon extends StatelessWidget {
  final AppIconData icon;
  final VoidCallback? onTap;
  const _CircleIcon({required this.icon, this.onTap});

  @override
  Widget build(BuildContext context) {
    return Material(
      color: Colors.white.withValues(alpha: 0.18),
      shape: const CircleBorder(),
      child: InkWell(
        customBorder: const CircleBorder(),
        onTap: onTap,
        child: Padding(
          padding: const EdgeInsets.all(9),
          child: HugeIcon(icon: icon, size: 18, color: Colors.white),
        ),
      ),
    );
  }
}
```

### ขั้นที่ 7 — หน้ารายการกรมธรรม์ด้วย Design System (UI)

ตอนนี้ประกอบทุกชิ้นเข้าด้วยกัน — หน้านี้คือเวอร์ชันจริงในโปรเจกต์ ใช้ `GradientHeader`, `AppCard`, `StatusBadge` และไอคอน Hugeicons · สังเกตว่า **Logic การกรองไม่อยู่ใน UI เลย** (อยู่ใน provider ที่เราสร้างในขั้นที่ 4) UI เพียง `ref.watch` มาแสดง

```dart
// lib/features/policy/presentation/pages/policy_list_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_spacing.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_card.dart';
import '../../../../core/widgets/gradient_header.dart';
import '../../../../core/widgets/status_badge.dart';
import '../../domain/policy.dart';
import '../controllers/policy_list_controller.dart';
import 'policy_detail_page.dart';

/// แปลงสถานะกรมธรรม์ → โทนสีของ badge (กฎการแสดงผลรวมไว้ที่เดียว)
BadgeTone toneOf(PolicyStatus s) => switch (s) {
      PolicyStatus.active => BadgeTone.success,
      PolicyStatus.pending => BadgeTone.pending,
      PolicyStatus.lapsed => BadgeTone.danger,
    };

class PolicyListPage extends ConsumerWidget {
  const PolicyListPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final asyncPolicies = ref.watch(policyListProvider);
    final policies = ref.watch(filteredPolicyListProvider);
    final selected = ref.watch(policyStatusFilterProvider);
    final total = asyncPolicies.value?.length ?? 0;

    final activePremium = (asyncPolicies.value ?? [])
        .where((p) => p.status == PolicyStatus.active)
        .fold<double>(0, (s, p) => s + p.premium);

    return Column(
      children: [
        GradientHeader(
          title: 'กรมธรรม์ของฉัน',
          subtitle: 'ทั้งหมด $total ฉบับ',
        ),
        Expanded(
          child: ListView(
            padding: const EdgeInsets.fromLTRB(16, 16, 16, 16),
            children: [
              // ช่องค้นหา
              TextField(
                onChanged: (v) =>
                    ref.read(policySearchProvider.notifier).update(v),
                decoration: const InputDecoration(
                  prefixIcon: HugeIcon(icon: HugeIcons.strokeRoundedSearch01, color: AppColors.textSubtle),
                  hintText: 'ค้นหาชื่อแผน หรือ เลขที่กรมธรรม์',
                ),
              ),
              const SizedBox(height: 12),
              // แถบกรองสถานะ
              SingleChildScrollView(
                scrollDirection: Axis.horizontal,
                child: Row(
                  children: [
                    _Chip(
                      label: 'ทั้งหมด',
                      selected: selected == null,
                      onTap: () => ref
                          .read(policyStatusFilterProvider.notifier)
                          .select(null),
                    ),
                    for (final s in PolicyStatus.values)
                      _Chip(
                        label: s.label,
                        selected: selected == s,
                        onTap: () => ref
                            .read(policyStatusFilterProvider.notifier)
                            .select(s),
                      ),
                  ],
                ),
              ),
              const SizedBox(height: 12),
              // แถบเบี้ยรวม
              Container(
                padding: const EdgeInsets.symmetric(horizontal: 14, vertical: 12),
                decoration: BoxDecoration(
                  color: AppColors.infoBg,
                  borderRadius: BorderRadius.circular(AppSpacing.rSm),
                ),
                child: Row(
                  children: [
                    const HugeIcon(icon: HugeIcons.strokeRoundedWallet01,
                        size: 18, color: AppColors.infoText),
                    const SizedBox(width: 8),
                    Text('เบี้ยรวม (มีผลบังคับ)',
                        style: AppType.label
                            .copyWith(color: AppColors.infoText)),
                    const Spacer(),
                    Text('฿${activePremium.toStringAsFixed(0)}/ปี',
                        style: AppType.bodyStrong
                            .copyWith(color: AppColors.infoText)),
                  ],
                ),
              ),
              const SizedBox(height: 14),
              // เนื้อหา: loading / error / data
              ...asyncPolicies.when(
                loading: () =>
                    [const Center(child: Padding(
                      padding: EdgeInsets.only(top: 40),
                      child: CircularProgressIndicator()))],
                error: (e, _) =>
                    [Center(child: Text('เกิดข้อผิดพลาด: $e'))],
                data: (_) => policies.isEmpty
                    ? [
                        const Padding(
                          padding: EdgeInsets.only(top: 40),
                          child: Center(
                              child: Text('ไม่พบกรมธรรม์ที่ตรงเงื่อนไข')),
                        )
                      ]
                    : policies
                        .map((p) => Padding(
                              padding: const EdgeInsets.only(bottom: 10),
                              child: _PolicyCard(policy: p),
                            ))
                        .toList(),
              ),
            ],
          ),
        ),
      ],
    );
  }
}

class _Chip extends StatelessWidget {
  final String label;
  final bool selected;
  final VoidCallback onTap;
  const _Chip(
      {required this.label, required this.selected, required this.onTap});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.only(right: 8),
      child: GestureDetector(
        onTap: onTap,
        child: Container(
          padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
          decoration: BoxDecoration(
            color: selected ? AppColors.primary : AppColors.card,
            borderRadius: BorderRadius.circular(AppSpacing.rPill),
            border: Border.all(
                color: selected ? AppColors.primary : AppColors.border),
          ),
          child: Text(label,
              style: AppType.label.copyWith(
                  color: selected ? Colors.white : AppColors.textMedium,
                  fontWeight: FontWeight.w600)),
        ),
      ),
    );
  }
}

class _PolicyCard extends StatelessWidget {
  final Policy policy;
  const _PolicyCard({required this.policy});

  @override
  Widget build(BuildContext context) {
    final tone = toneOf(policy.status);
    final iconColor = switch (tone) {
      BadgeTone.success => AppColors.successFg,
      BadgeTone.pending => AppColors.pendingFg,
      BadgeTone.danger => AppColors.dangerFg,
      _ => AppColors.primary,
    };
    final iconBg = switch (tone) {
      BadgeTone.success => AppColors.successBg,
      BadgeTone.pending => AppColors.pendingBg,
      BadgeTone.danger => AppColors.dangerBg,
      _ => AppColors.background,
    };
    return AppCard(
      padding: const EdgeInsets.all(14),
      onTap: () => Navigator.of(context).push(
        MaterialPageRoute(builder: (_) => PolicyDetailPage(policy: policy)),
      ),
      child: Row(
        children: [
          Container(
            width: 46,
            height: 46,
            decoration: BoxDecoration(
                color: iconBg, borderRadius: BorderRadius.circular(12)),
            child: HugeIcon(icon: HugeIcons.strokeRoundedShield01, color: iconColor),
          ),
          const SizedBox(width: 12),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(policy.planName, style: AppType.title),
                const SizedBox(height: 2),
                Text(policy.policyNumber, style: AppType.caption),
                const SizedBox(height: 6),
                Row(
                  children: [
                    Text('฿${policy.premium.toStringAsFixed(0)}/ปี',
                        style: AppType.bodyStrong),
                    const SizedBox(width: 8),
                    StatusBadge(label: policy.status.label, tone: tone),
                  ],
                ),
              ],
            ),
          ),
          const HugeIcon(icon: HugeIcons.strokeRoundedArrowRight01, color: AppColors.textSubtle),
        ],
      ),
    );
  }
}
```

### ขั้นที่ 8 — เชื่อมเข้ากับแอป (main.dart) และรัน

`PolicyListPage` คืน `Column` (ไม่มี `Scaffold` ในตัว) เพราะออกแบบให้ฝังใน "เปลือก" ของแอป คือ `MainShell` ที่มี Bottom Navigation (ดู Part B) — `main.dart` ฉบับจริงของ branch `day1` ตั้ง `home` เป็น `SplashPage` เพื่อเริ่ม flow เต็มของแอป: **Splash → Login → MainShell**

```dart
// lib/main.dart (ฉบับจริงใน branch day1)
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import 'core/theme/app_colors.dart';
import 'core/theme/app_theme.dart';
import 'features/auth/presentation/pages/splash_page.dart';

void main() {
  // ครอบแอปด้วย ProviderScope เพื่อให้ Riverpod ทำงานได้ทั้งแอป
  runApp(const ProviderScope(child: BlaApp()));
}

class BlaApp extends StatelessWidget {
  const BlaApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'BLA Policy Companion',
      debugShowCheckedModeBanner: false,
      theme: AppTheme.light,
      home: const SplashPage(),
      // scrim ทึบคลุมพื้นที่ status bar ทุกหน้า: ป้องกันเนื้อหาที่เลื่อนขึ้น
      // ไปซ้อนกับนาฬิกา/ไอคอนระบบ และคงไอคอน status bar เป็นสีขาว
      builder: (context, child) {
        final topInset = MediaQuery.paddingOf(context).top;
        return AnnotatedRegion<SystemUiOverlayStyle>(
          value: SystemUiOverlayStyle.light.copyWith(
            statusBarColor: Colors.transparent,
          ),
          child: Stack(
            children: [
              if (child != null) child,
              Positioned(
                top: 0,
                left: 0,
                right: 0,
                height: topInset,
                child: const ColoredBox(color: AppColors.primaryLight),
              ),
            ],
          ),
        );
      },
    );
  }
}
```

**จุดที่ควรอธิบายใน main.dart ฉบับจริง:**

1. `ProviderScope` — รากของ Riverpod ทั้งแอป (เหมือนที่เรียนใน Module 2)
2. `theme: AppTheme.light` — ดึง Design System ทั้งชุดเข้า MaterialApp ที่จุดเดียว ทุกหน้า/ทุก `TextField` ได้สไตล์เดียวกันอัตโนมัติ
3. `home: SplashPage()` — จุดเริ่ม flow ของแอปจริง (Splash หน่วง 1.8 วิ → Login → MainShell)
4. `builder:` — เทคนิคระดับโปร: ห่อทุกหน้าด้วย `Stack` แล้ววาง `ColoredBox` สีน้ำเงินทับพื้นที่ status bar (`MediaQuery.paddingOf(context).top`) เพื่อไม่ให้เนื้อหาที่ scroll เลื่อนไปชนนาฬิกา/แบตเตอรี่ พร้อม `AnnotatedRegion<SystemUiOverlayStyle>` บังคับไอคอน status bar เป็นสีขาวทุกหน้า — เขียนที่เดียว มีผลทั้งแอป

> 💡 **ระหว่างทำเวิร์กช็อป:** ถ้าอยากรันเฉพาะหน้ารายการกรมธรรม์โดยไม่ผ่าน Login ให้เปลี่ยนชั่วคราวเป็น `home: const Scaffold(body: PolicyListPage())` แล้วค่อยสลับกลับเป็น `SplashPage` เมื่อประกอบร่างเต็ม

รันด้วยคำสั่ง (อย่าลืมสั่ง build_runner ก่อนหากยังไม่ได้สร้างไฟล์ `.g.dart`):

```bash
dart run build_runner build --delete-conflicting-outputs
flutter run
```

> ✅ **ผลลัพธ์ที่คาดหวัง:** หน้าจอแสดงหัว gradient น้ำเงิน + รายการกรมธรรม์ 4 ใบเป็นการ์ดสไตล์ Design System พิมพ์ค้นหาแล้วรายการกรองทันที กดชิปสถานะแล้วกรองตามสถานะ แถบ "เบี้ยรวม (มีผลบังคับ)" อัปเดตตามข้อมูล และมี loading spinner ตอนเปิดครั้งแรก · ทั้งหมดนี้เกิดจากการ `ref.watch` หลาย provider ประกอบกัน โดย UI **ไม่มี Logic การกรองเลย** — Logic อยู่ในชั้น Presentation ที่แยกออกมาแล้ว

### 🧩 ความท้าทายเสริม (ถ้าทำเสร็จก่อนเวลา)

- เพิ่มการเรียงลำดับ (sort) ตามเบี้ยประกันจากมากไปน้อย โดยทำเป็น provider แยก แล้ว `ref.watch` เพิ่มในหน้า
- ลองเพิ่มสถานะใหม่ให้ `PolicyStatus` (เช่น `expired`) แล้วไล่ดูว่า compiler บังคับให้แก้ที่ไหนบ้าง (จุดแข็งของ `switch` แบบ exhaustive)
- แยกวิดเจ็ตกลางเพิ่มเอง เช่น การ์ดสรุปตัวเลข แล้วนำไปใช้ซ้ำทั้งหน้า Home และ Claims (ฝึกคิดแบบ Design System)

---

## 📦 Part B — โค้ดจริงครบทุกไฟล์ใน branch `day1` (ส่วนประกอบร่างแอปเต็ม)

> Workshop ช่วงบ่ายเราสร้างฟีเจอร์ **policy** ครบวงจรแล้ว (ขั้นที่ 1–8) ส่วนนี้คือโค้ด **ฉบับจริงปัจจุบัน** ของไฟล์ที่เหลือทั้งหมดใน branch `day1` พร้อมคำอธิบายว่าแต่ละไฟล์ทำหน้าที่อะไร และ "ต่อสาย" ถึงกันอย่างไร — ใช้ทบทวนหลังอบรม หรือ copy ไปประกอบร่างแอปเต็มได้ทันที

### B0. ภาพรวมการเชื่อมโยงทั้งแอป

**1) เส้นทางการนำทาง (Navigation Flow)** — เริ่มจาก `main.dart` ตั้ง `home: SplashPage()`:

```
main.dart (ProviderScope + AppTheme.light + status bar scrim)
└── SplashPage  ── หน่วง 1.8 วิ ──▶ pushReplacement ──▶ LoginPage
       LoginPage
       ├─ "เข้าสู่ระบบ" / Google / LINE ─▶ pushReplacement ─▶ MainShell
       ├─ "ลืมรหัสผ่าน?" ─▶ push ─▶ ForgotPasswordPage ─▶ push ─▶ ResetPasswordPage
       │                                                    └─ popUntil(isFirst) กลับ Login
       └─ "สมัครสมาชิก" ─▶ push ─▶ RegisterPage ─ "สร้างบัญชี" ─▶ pop กลับ Login

MainShell (Scaffold + IndexedStack + Bottom Navigation 5 แท็บ — mainTabProvider)
├─ [0] HomePage      ─ ปุ่มลัด "กรมธรรม์/ชำระเบี้ย" สลับแท็บผ่าน mainTabProvider
│                     ─ ปุ่มลัด "ยื่นสินไหม" เปิด FileClaimSheet
├─ [1] PolicyListPage ─ แตะการ์ด ─▶ push ─▶ PolicyDetailPage
│                                      └─ ปุ่ม "ยื่นสินไหม" เปิด FileClaimSheet
├─ [2] ClaimsPage    ─ ปุ่ม "ยื่นสินไหมใหม่" เปิด FileClaimSheet
├─ [3] PaymentsPage
└─ [4] ProfilePage   ─ "ออกจากระบบ" ─▶ pushAndRemoveUntil ─▶ LoginPage
```

**2) กราฟ Provider (Riverpod)** — state ทั้งหมดของ Day 1 มี 5 provider:

```
policyRepositoryProvider (Provider — ฉีด PolicyRepository จากชั้น Data)
        │ ref.watch
        ▼
policyListProvider (FutureProvider — AsyncValue<List<Policy>>)
        │ ref.watch
        ▼
filteredPolicyListProvider (Provider — List<Policy> ที่กรองแล้ว)
        ▲ ref.watch            ▲ ref.watch
policySearchProvider     policyStatusFilterProvider
(Notifier<String>)       (Notifier<PolicyStatus?>)

mainTabProvider (StateProvider<int> ใน main_shell.dart)
  ├─ MainShell   → ref.watch เพื่อเลือกหน้าใน IndexedStack
  └─ HomePage    → ref.read(...).state = i (ปุ่มลัดสลับแท็บข้ามฟีเจอร์)
```

พิมพ์คำค้นหรือกดชิปกรอง → `policySearchProvider`/`policyStatusFilterProvider` เปลี่ยน → `filteredPolicyListProvider` คำนวณใหม่ → `PolicyListPage` rebuild เฉพาะส่วนที่ watch อยู่ — UI ไม่มี logic กรองแม้แต่บรรทัดเดียว

**3) ชั้นการพึ่งพา (Dependency Layers)** — กฎ: ลูกศรชี้ "ลง" เท่านั้น ห้ามชั้นล่างรู้จักชั้นบน:

```
features/* (pages, controllers)          ← รู้จัก core ได้
   │ ใช้วิดเจ็ตกลาง + โทเคน
   ▼
core/widgets (AppButton, AppCard, GradientHeader, ...)
   │ ใช้โทเคน
   ▼
core/theme + core/icons (AppColors, AppSpacing, AppType, HugeIcon)
   ▼
Flutter SDK + packages (google_fonts, hugeicons, flutter_riverpod)
```

**4) ตารางสรุป: ใครใช้ใคร**

| ไฟล์                    | พึ่งพา (import จาก)                                        | ถูกเรียกใช้โดย                                      |
| ----------------------- | ----------------------------------------------------------- | ---------------------------------------------------- |
| `main.dart`             | app_theme, app_colors, splash_page                          | — (จุดเริ่มแอป)                                      |
| `splash_page.dart`      | app_colors, app_typography, app_logo                        | main.dart                                            |
| `login_page.dart`       | app_button, app_logo, app_text_field, info_hint, main_shell | splash_page, profile_page (logout)                   |
| `register_page.dart`    | app_button, app_text_field, gradient_header                 | login_page                                           |
| `forgot_password_page`  | app_button, app_text_field, gradient_header                 | login_page                                           |
| `reset_password_page`   | app_button, app_text_field, gradient_header, info_hint      | forgot_password_page                                 |
| `main_shell.dart`       | 5 หน้าแท็บ + app_icons + app_colors                          | login_page · **export** `mainTabProvider` ให้ home   |
| `home_page.dart`        | วิดเจ็ตกลาง + main_shell (mainTabProvider) + file_claim_sheet | main_shell (แท็บ 0)                                  |
| `policy_list_page.dart` | controller + domain + วิดเจ็ตกลาง + policy_detail_page        | main_shell (แท็บ 1)                                  |
| `policy_detail_page`    | domain (Policy) + วิดเจ็ตกลาง + file_claim_sheet              | policy_list_page (`push` พร้อมส่ง `Policy`)          |
| `claims_page.dart`      | วิดเจ็ตกลาง + file_claim_sheet                                | main_shell (แท็บ 2)                                  |
| `file_claim_sheet.dart` | app_button, app_text_field + โทเคน                           | home_page, policy_detail_page, claims_page (3 ที่)   |
| `payments_page.dart`    | วิดเจ็ตกลาง (ไม่มี state — static)                            | main_shell (แท็บ 3)                                  |
| `profile_page.dart`     | วิดเจ็ตกลาง + login_page (logout)                             | main_shell (แท็บ 4)                                  |
| `policy_repository`     | domain (Policy) + riverpod_annotation                        | policy_list_controller (ผ่าน provider)               |
| `policy_list_controller`| policy_repository + domain                                   | policy_list_page (ผ่าน 4 provider)                   |

### B1. วิดเจ็ตกลางเพิ่มเติม (core/widgets)

นอกจาก `AppCard` / `StatusBadge` / `GradientHeader` ที่สร้างใน Workshop แล้ว โปรเจกต์จริงมีวิดเจ็ตกลางอีก 5 ตัว — ทั้งหมดพึ่งพาเฉพาะ `core/theme` + `core/icons` (ไม่รู้จัก feature ใด ๆ) จึงถูกเรียกใช้ซ้ำจากทุกหน้าได้อิสระ

#### B1.1 `app_button.dart` — ปุ่มมาตรฐาน 3 แบบ

ใช้เทคนิค **named constructors** (`AppButton.primary / .secondary / .social`) ให้ทุกหน้าเรียกปุ่มหน้าตาเดียวกันโดยไม่ต้องรู้รายละเอียดสไตล์ · ปุ่ม primary เป็น gradient + เงา, ปุ่ม disabled (`onPressed == null`) เปลี่ยนเป็นสีเทาอัตโนมัติ (ดูหน้า Register ที่ปุ่ม "สร้างบัญชี" จะกดได้ต่อเมื่อติ๊กยอมรับเงื่อนไข)

```dart
// lib/core/widgets/app_button.dart
import 'package:flutter/material.dart';

import '../icons/app_icons.dart';
import '../theme/app_colors.dart';
import '../theme/app_spacing.dart';
import '../theme/app_typography.dart';

/// ปุ่มของ Design System — รองรับหลายแบบด้วย named constructors
/// เพื่อให้ทั้งแอปใช้ปุ่มหน้าตาเดียวกัน
class AppButton extends StatelessWidget {
  final String label;
  final VoidCallback? onPressed;
  final AppIconData? icon;
  final _ButtonKind _kind;
  final Color? customColor;
  final bool expand;

  const AppButton._(
    this._kind, {
    required this.label,
    required this.onPressed,
    this.icon,
    this.customColor,
    this.expand = true,
  });

  /// ปุ่มหลัก (gradient น้ำเงิน)
  const AppButton.primary({
    Key? key,
    required String label,
    required VoidCallback? onPressed,
    AppIconData? icon,
    bool expand = true,
  }) : this._(_ButtonKind.primary,
            label: label, onPressed: onPressed, icon: icon, expand: expand);

  /// ปุ่มรอง (ขอบ + พื้นขาว)
  const AppButton.secondary({
    Key? key,
    required String label,
    required VoidCallback? onPressed,
    AppIconData? icon,
    bool expand = true,
  }) : this._(_ButtonKind.secondary,
            label: label, onPressed: onPressed, icon: icon, expand: expand);

  /// ปุ่ม social (Google/LINE) — ส่งสีพื้นได้
  const AppButton.social({
    Key? key,
    required String label,
    required VoidCallback? onPressed,
    AppIconData? icon,
    Color? color,
    bool expand = true,
  }) : this._(_ButtonKind.social,
            label: label,
            onPressed: onPressed,
            icon: icon,
            customColor: color,
            expand: expand);

  @override
  Widget build(BuildContext context) {
    final child = _content();
    final button = switch (_kind) {
      _ButtonKind.primary => _gradientButton(child),
      _ButtonKind.secondary => _outlinedButton(child),
      _ButtonKind.social => _socialButton(child),
    };
    return expand ? SizedBox(width: double.infinity, child: button) : button;
  }

  Widget _content() {
    final color = switch (_kind) {
      _ButtonKind.primary => Colors.white,
      _ButtonKind.secondary => AppColors.primary,
      _ButtonKind.social =>
        customColor == null ? AppColors.textDark : Colors.white,
    };
    return Row(
      mainAxisAlignment: MainAxisAlignment.center,
      mainAxisSize: MainAxisSize.min,
      children: [
        if (icon != null) ...[HugeIcon(icon: icon!, size: 20, color: color), const SizedBox(width: 8)],
        Text(label, style: AppType.bodyStrong.copyWith(color: color)),
      ],
    );
  }

  Widget _gradientButton(Widget child) {
    return DecoratedBox(
      decoration: BoxDecoration(
        gradient: onPressed == null ? null : AppColors.primaryGradient,
        color: onPressed == null ? AppColors.textSubtle : null,
        borderRadius: BorderRadius.circular(AppSpacing.rMd),
        boxShadow: onPressed == null
            ? null
            : [
                BoxShadow(
                  color: AppColors.primary.withValues(alpha: 0.30),
                  blurRadius: 16,
                  offset: const Offset(0, 6),
                ),
              ],
      ),
      child: Material(
        color: Colors.transparent,
        child: InkWell(
          borderRadius: BorderRadius.circular(AppSpacing.rMd),
          onTap: onPressed,
          child: Padding(
            padding: const EdgeInsets.symmetric(vertical: 15),
            child: child,
          ),
        ),
      ),
    );
  }

  Widget _outlinedButton(Widget child) {
    return OutlinedButton(
      onPressed: onPressed,
      style: OutlinedButton.styleFrom(
        padding: const EdgeInsets.symmetric(vertical: 14),
        side: const BorderSide(color: AppColors.primary, width: 1.5),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(AppSpacing.rMd),
        ),
      ),
      child: child,
    );
  }

  Widget _socialButton(Widget child) {
    final bg = customColor ?? AppColors.card;
    return Material(
      color: bg,
      borderRadius: BorderRadius.circular(AppSpacing.rMd),
      child: InkWell(
        borderRadius: BorderRadius.circular(AppSpacing.rMd),
        onTap: onPressed,
        child: Container(
          decoration: BoxDecoration(
            borderRadius: BorderRadius.circular(AppSpacing.rMd),
            border: customColor == null
                ? Border.all(color: AppColors.border, width: 1.4)
                : null,
          ),
          padding: const EdgeInsets.symmetric(vertical: 14),
          child: child,
        ),
      ),
    );
  }
}

enum _ButtonKind { primary, secondary, social }
```

> 🔗 **การเชื่อมโยง:** ถูกใช้ใน login/register/forgot/reset (Auth ทั้ง 4 หน้า), claims_page ("ยื่นสินไหมใหม่"), policy_detail_page ("ชำระเบี้ย"/"ยื่นสินไหม") และ file_claim_sheet ("ยืนยันการยื่น"/"ปิด") — เปลี่ยนดีไซน์ปุ่มทั้งแอปได้จากไฟล์นี้ไฟล์เดียว

#### B1.2 `app_logo.dart` — โลโก้ BLA

```dart
// lib/core/widgets/app_logo.dart
import 'package:flutter/material.dart';

import '../theme/app_colors.dart';

/// โลโก้ BLA แบบการ์ดสี่เหลี่ยมมุมโค้ง (ใช้ในหน้า Splash / Login / หัวข้อ)
class AppLogo extends StatelessWidget {
  final double size;
  final bool full; // true = โลโก้เต็ม (มีข้อความ), false = ไอคอนอย่างเดียว
  final double radius;

  const AppLogo({super.key, this.size = 84, this.full = false, this.radius = 22});

  @override
  Widget build(BuildContext context) {
    return Container(
      // การ์ดทรงสี่เหลี่ยมจัตุรัสทั้งแบบเต็ม (โลโก้+ข้อความซ้อนแนวตั้ง) และแบบไอคอน
      width: size,
      height: size,
      decoration: BoxDecoration(
        color: AppColors.card,
        borderRadius: BorderRadius.circular(radius),
        boxShadow: [
          BoxShadow(
            color: AppColors.primaryDark.withValues(alpha: 0.18),
            blurRadius: 24,
            offset: const Offset(0, 10),
          ),
        ],
      ),
      padding: EdgeInsets.all(size * 0.16),
      child: Image.asset(
        full ? 'assets/images/bla_logo_full.png' : 'assets/images/bla_icon.png',
        fit: BoxFit.contain,
      ),
    );
  }
}
```

> 🔗 **การเชื่อมโยง:** `SplashPage` ใช้แบบเต็ม (`full: true, size: 240`), `LoginPage` ใช้แบบไอคอน (`size: 76`) · รูปมาจาก `assets/images/` ที่ประกาศใน `pubspec.yaml`

#### B1.3 `app_text_field.dart` — ช่องกรอกข้อมูล + ซ่อน/แสดงรหัสผ่าน

เป็น `StatefulWidget` เพราะเก็บ state ภายในตัวเอง (`_hidden` = ตาเปิด/ปิดรหัสผ่าน) — ตัวอย่างที่ดีของ **local UI state** ที่ไม่จำเป็นต้องใช้ Riverpod (state นี้ไม่มีใครนอก widget ต้องรู้)

```dart
// lib/core/widgets/app_text_field.dart
import 'package:flutter/material.dart';

import '../icons/app_icons.dart';
import '../theme/app_colors.dart';
import '../theme/app_typography.dart';

/// ช่องกรอกข้อมูลของ Design System: มี label ด้านบน + ไอคอนนำหน้า + รองรับรหัสผ่าน
class AppTextField extends StatefulWidget {
  final String label;
  final String? hint;
  final AppIconData? icon;
  final bool obscure;
  final TextInputType? keyboardType;
  final TextEditingController? controller;
  final int maxLines;

  const AppTextField({
    super.key,
    required this.label,
    this.hint,
    this.icon,
    this.obscure = false,
    this.keyboardType,
    this.controller,
    this.maxLines = 1,
  });

  @override
  State<AppTextField> createState() => _AppTextFieldState();
}

class _AppTextFieldState extends State<AppTextField> {
  late bool _hidden = widget.obscure;

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(widget.label, style: AppType.label),
        const SizedBox(height: 6),
        TextField(
          controller: widget.controller,
          obscureText: _hidden,
          keyboardType: widget.keyboardType,
          maxLines: widget.obscure ? 1 : widget.maxLines,
          style: AppType.bodyStrong,
          decoration: InputDecoration(
            hintText: widget.hint,
            prefixIcon: widget.icon == null
                ? null
                : HugeIcon(
                    icon: widget.icon!, size: 20, color: AppColors.textSubtle),
            suffixIcon: widget.obscure
                ? IconButton(
                    icon: HugeIcon(
                      icon: _hidden
                          ? HugeIcons.strokeRoundedView
                          : HugeIcons.strokeRoundedViewOffSlash,
                      size: 20,
                      color: AppColors.textSubtle,
                    ),
                    onPressed: () => setState(() => _hidden = !_hidden),
                  )
                : null,
          ),
        ),
      ],
    );
  }
}
```

> 🔗 **การเชื่อมโยง:** สังเกตว่าไม่ได้กำหนดกรอบ/สีเอง — กรอบมุมโค้ง, สี focus, hint style มาจาก `inputDecorationTheme` ใน `app_theme.dart` อัตโนมัติ · ใช้ใน Auth ทั้ง 4 หน้า + `FileClaimSheet`

#### B1.4 `info_hint.dart` — กล่องคำใบ้

```dart
// lib/core/widgets/info_hint.dart
import 'package:flutter/material.dart';

import '../icons/app_icons.dart';
import '../theme/app_colors.dart';
import '../theme/app_spacing.dart';
import '../theme/app_typography.dart';

/// กล่องคำใบ้/หมายเหตุเล็ก ๆ (เช่น "บัญชีทดสอบ: ...", เคล็ดลับรหัสผ่าน)
class InfoHint extends StatelessWidget {
  final String text;
  final AppIconData icon;
  const InfoHint(this.text,
      {super.key, this.icon = HugeIcons.strokeRoundedIdea});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 10),
      decoration: BoxDecoration(
        color: AppColors.background,
        borderRadius: BorderRadius.circular(AppSpacing.rSm),
        border: Border.all(color: AppColors.border),
      ),
      child: Row(
        children: [
          HugeIcon(icon: icon, size: 16, color: AppColors.textSubtle),
          const SizedBox(width: 8),
          Expanded(child: Text(text, style: AppType.caption)),
        ],
      ),
    );
  }
}
```

> 🔗 **การเชื่อมโยง:** ใช้ใน `LoginPage` (บอกบัญชีทดสอบ) และ `ResetPasswordPage` (เคล็ดลับรหัสผ่าน)

#### B1.5 `section_label.dart` — หัวข้อ section

```dart
// lib/core/widgets/section_label.dart
import 'package:flutter/material.dart';

import '../theme/app_typography.dart';

/// หัวข้อย่อยของ section (เช่น "บัญชีของฉัน", "การตั้งค่า", "ประวัติการชำระ")
class SectionLabel extends StatelessWidget {
  final String text;
  final Widget? trailing;

  const SectionLabel(this.text, {super.key, this.trailing});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.fromLTRB(4, 8, 4, 8),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(text, style: AppType.h3),
          if (trailing != null) trailing!,
        ],
      ),
    );
  }
}
```

> 🔗 **การเชื่อมโยง:** ใช้ใน home (พร้อม `trailing` "ดูทั้งหมด"), claims, payments, profile — คุมสไตล์หัวข้อย่อยทั้งแอปให้เท่ากันด้วย `AppType.h3`

### B2. เปลือกแอป — `main_shell.dart` (Bottom Navigation 5 แท็บ)

หัวใจ 3 จุด: (1) `mainTabProvider` เป็น `StateProvider<int>` ระดับ global — หน้าอื่น (เช่น ปุ่มลัดใน Home) สั่งสลับแท็บได้โดยไม่ต้องส่ง callback ข้ามหน้า (2) `IndexedStack` เก็บ state ของทุกแท็บไว้ ไม่ rebuild เมื่อสลับ — พิมพ์ค้นหาในแท็บกรมธรรม์ สลับไปแท็บอื่นแล้วกลับมา คำค้นยังอยู่ (3) แถบเมนูล่างเป็น custom widget ที่ทำ indicator ขีดบนไอคอนเอง

```dart
// lib/features/app/main_shell.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../core/icons/app_icons.dart';
import '../../core/theme/app_colors.dart';
import '../claim/presentation/pages/claims_page.dart';
import '../home/presentation/pages/home_page.dart';
import '../payment/presentation/pages/payments_page.dart';
import '../policy/presentation/pages/policy_list_page.dart';
import '../profile/presentation/pages/profile_page.dart';

/// แท็บที่เลือกอยู่ของ Bottom Navigation
///
/// Day 1: ใช้ StateProvider ง่าย ๆ เพื่อให้หน้าอื่น (เช่น ปุ่มลัดในหน้าแรก)
/// สั่งสลับแท็บได้ — Day 3 จะยกระดับเป็น GoRouter + StatefulShellRoute
final mainTabProvider = StateProvider<int>((ref) => 0);

class MainShell extends ConsumerWidget {
  const MainShell({super.key});

  static const _pages = [
    HomePage(),
    PolicyListPage(),
    ClaimsPage(),
    PaymentsPage(),
    ProfilePage(),
  ];

  static const _items = [
    (icon: HugeIcons.strokeRoundedHome03, active: HugeIcons.strokeRoundedHome03, label: 'หน้าแรก'),
    (icon: HugeIcons.strokeRoundedShield01, active: HugeIcons.strokeRoundedShield01, label: 'กรมธรรม์'),
    (icon: HugeIcons.strokeRoundedFile01, active: HugeIcons.strokeRoundedFile01, label: 'สินไหม'),
    (icon: HugeIcons.strokeRoundedCreditCard, active: HugeIcons.strokeRoundedCreditCard, label: 'ชำระเบี้ย'),
    (icon: HugeIcons.strokeRoundedUser, active: HugeIcons.strokeRoundedUser, label: 'โปรไฟล์'),
  ];

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final index = ref.watch(mainTabProvider);
    return Scaffold(
      body: IndexedStack(index: index, children: _pages),
      bottomNavigationBar: Container(
        decoration: BoxDecoration(
          color: AppColors.card,
          boxShadow: [
            BoxShadow(
              color: AppColors.primaryDark.withValues(alpha: 0.08),
              blurRadius: 16,
              offset: const Offset(0, -2),
            ),
          ],
        ),
        child: SafeArea(
          top: false,
          child: Padding(
            padding: const EdgeInsets.symmetric(vertical: 6, horizontal: 4),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              children: [
                for (var i = 0; i < _items.length; i++)
                  _NavItem(
                    item: _items[i],
                    selected: i == index,
                    onTap: () =>
                        ref.read(mainTabProvider.notifier).state = i,
                  ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}

class _NavItem extends StatelessWidget {
  final ({AppIconData icon, AppIconData active, String label}) item;
  final bool selected;
  final VoidCallback onTap;

  const _NavItem(
      {required this.item, required this.selected, required this.onTap});

  @override
  Widget build(BuildContext context) {
    final color = selected ? AppColors.primary : AppColors.textSubtle;
    return Expanded(
      child: InkWell(
        borderRadius: BorderRadius.circular(12),
        onTap: onTap,
        child: Padding(
          padding: const EdgeInsets.symmetric(vertical: 6),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              if (selected)
                Container(
                  width: 20,
                  height: 3,
                  margin: const EdgeInsets.only(bottom: 5),
                  decoration: BoxDecoration(
                    color: AppColors.primary,
                    borderRadius: BorderRadius.circular(2),
                  ),
                )
              else
                const SizedBox(height: 8),
              HugeIcon(
                  icon: selected ? item.active : item.icon,
                  size: 24,
                  color: color),
              const SizedBox(height: 3),
              Text(item.label,
                  style: TextStyle(
                      fontSize: 10.5,
                      color: color,
                      fontWeight:
                          selected ? FontWeight.w600 : FontWeight.w500)),
            ],
          ),
        ),
      ),
    );
  }
}
```

> 🔗 **การเชื่อมโยง:** ไฟล์นี้คือ "จุดรวมพล" ที่ import ทั้ง 5 หน้าแท็บ · `_items` ใช้ **record type** ของ Dart 3 (`({AppIconData icon, ..., String label})`) แทนการสร้างคลาสเล็ก ๆ · แถบเมนูกด → `ref.read(mainTabProvider.notifier).state = i` (แก้ state ใน callback ใช้ `read` ตามกฎ Module 4) ส่วน `IndexedStack` ฟังผ่าน `ref.watch` — ตัวอย่าง watch/read คู่กันในไฟล์เดียว

### B3. ฟีเจอร์ Auth — 5 หน้า (UI ครบตาม mockup, Day 3 จะต่อ auth จริง)

#### B3.1 `splash_page.dart` — จุดเริ่มของแอป

ใช้ `StatefulWidget` + `Timer` ใน `initState` (ไม่ใช่ Riverpod — เพราะเป็น one-shot side effect ของหน้าเดียว) · เช็ก `mounted` ก่อนใช้ context หลัง delay เสมอ

```dart
// lib/features/auth/presentation/pages/splash_page.dart
import 'dart:async';

import 'package:flutter/material.dart';

import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_logo.dart';
import 'login_page.dart';

/// หน้า Splash — โลโก้กลางจอบนพื้น gradient + แถบโหลด
/// Day 1: หน่วงเวลาแล้วไปหน้า Login (ยังไม่เช็ค session จริง — เป็นเนื้อหา Day 3)
class SplashPage extends StatefulWidget {
  const SplashPage({super.key});

  @override
  State<SplashPage> createState() => _SplashPageState();
}

class _SplashPageState extends State<SplashPage> {
  @override
  void initState() {
    super.initState();
    Timer(const Duration(milliseconds: 1800), _goNext);
  }

  void _goNext() {
    if (!mounted) return;
    Navigator.of(context).pushReplacement(
      MaterialPageRoute(builder: (_) => const LoginPage()),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        width: double.infinity,
        height: double.infinity,
        decoration: const BoxDecoration(gradient: AppColors.headerGradient),
        child: SafeArea(
          child: Column(
            children: [
              const Spacer(flex: 3),
              const AppLogo(full: true, size: 240, radius: 40),
              const SizedBox(height: 22),
              Text(
                'POLICY COMPANION',
                style: AppType.label.copyWith(
                  color: Colors.white70,
                  letterSpacing: 3,
                  fontWeight: FontWeight.w600,
                ),
              ),
              const Spacer(flex: 3),
              SizedBox(
                width: 180,
                child: ClipRRect(
                  borderRadius: BorderRadius.circular(8),
                  child: const LinearProgressIndicator(
                    minHeight: 3,
                    backgroundColor: Colors.white24,
                    valueColor: AlwaysStoppedAnimation(AppColors.cyan),
                  ),
                ),
              ),
              const SizedBox(height: 14),
              Text('v1.0.0',
                  style: AppType.caption.copyWith(color: Colors.white60)),
              const SizedBox(height: 20),
            ],
          ),
        ),
      ),
    );
  }
}
```

> 🔗 **การเชื่อมโยง:** ถูกตั้งเป็น `home` ใน `main.dart` → ใช้ `pushReplacement` ไป `LoginPage` (แทนที่ตัวเองใน stack เพื่อไม่ให้กด back กลับมา Splash ได้)

#### B3.2 `login_page.dart` — เข้าสู่ระบบ (ประตูเข้า MainShell)

```dart
// lib/features/auth/presentation/pages/login_page.dart
import 'package:flutter/material.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_spacing.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_logo.dart';
import '../../../../core/widgets/app_text_field.dart';
import '../../../../core/widgets/info_hint.dart';
import '../../../app/main_shell.dart';
import 'forgot_password_page.dart';
import 'register_page.dart';

/// หน้าเข้าสู่ระบบ
/// Day 1: เป็น UI ครบตาม mockup — ปุ่ม "เข้าสู่ระบบ" พาเข้าแอปเลย (ยังไม่ตรวจรหัสจริง)
/// Day 3: จะต่อ Reactive Auth + Secure Storage + Route Guard จริง
class LoginPage extends StatelessWidget {
  const LoginPage({super.key});

  void _enter(BuildContext context) {
    Navigator.of(context).pushReplacement(
      MaterialPageRoute(builder: (_) => const MainShell()),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: ListView(
        padding: EdgeInsets.zero,
        children: [
          // หัว gradient + โลโก้
          Container(
            decoration: const BoxDecoration(
              gradient: AppColors.headerGradient,
              borderRadius:
                  BorderRadius.vertical(bottom: Radius.circular(AppSpacing.rLg)),
            ),
            child: SafeArea(
              bottom: false,
              child: Padding(
                padding: const EdgeInsets.fromLTRB(24, 24, 24, 28),
                child: Column(
                  children: [
                    const AppLogo(size: 76),
                    const SizedBox(height: 14),
                    Text(
                      'เข้าสู่ระบบเพื่อจัดการกรมธรรม์ของคุณ',
                      textAlign: TextAlign.center,
                      style: AppType.body.copyWith(color: Colors.white70),
                    ),
                  ],
                ),
              ),
            ),
          ),
          Padding(
            padding: const EdgeInsets.fromLTRB(20, 20, 20, 24),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                const AppTextField(
                  label: 'ชื่อผู้ใช้',
                  hint: 'agent',
                  icon: HugeIcons.strokeRoundedUser,
                ),
                const SizedBox(height: 14),
                const AppTextField(
                  label: 'รหัสผ่าน',
                  hint: '••••',
                  icon: HugeIcons.strokeRoundedSquareLock01,
                  obscure: true,
                ),
                const SizedBox(height: 12),
                const InfoHint('บัญชีทดสอบ: ชื่อผู้ใช้ agent / รหัสผ่าน 1234'),
                const SizedBox(height: 18),
                AppButton.primary(
                  label: 'เข้าสู่ระบบ',
                  icon: HugeIcons.strokeRoundedLogin03,
                  onPressed: () => _enter(context),
                ),
                const SizedBox(height: 12),
                Center(
                  child: TextButton(
                    onPressed: () => Navigator.of(context).push(
                      MaterialPageRoute(
                          builder: (_) => const ForgotPasswordPage()),
                    ),
                    child: Text('ลืมรหัสผ่าน?',
                        style: AppType.bodyStrong
                            .copyWith(color: AppColors.primary)),
                  ),
                ),
                const _OrDivider(label: 'หรือดำเนินการต่อด้วย'),
                const SizedBox(height: 16),
                AppButton.social(
                  label: 'ดำเนินการต่อด้วย Google',
                  icon: HugeIcons.strokeRoundedGoogle,
                  color: const Color(0xFF4285F4),
                  onPressed: () => _enter(context),
                ),
                const SizedBox(height: 12),
                AppButton.social(
                  label: 'ดำเนินการต่อด้วย LINE',
                  icon: HugeIcons.strokeRoundedBubbleChat,
                  color: const Color(0xFF06C755),
                  onPressed: () => _enter(context),
                ),
                const _OrDivider(label: 'ยังไม่มีบัญชี?'),
                const SizedBox(height: 16),
                AppButton.secondary(
                  label: 'สมัครสมาชิก',
                  icon: HugeIcons.strokeRoundedUserAdd01,
                  onPressed: () => Navigator.of(context).push(
                    MaterialPageRoute(builder: (_) => const RegisterPage()),
                  ),
                ),
                const SizedBox(height: 20),
                Center(
                  child: Text('กรุงเทพประกันชีวิต จำกัด (มหาชน) · v1.0.0',
                      style: AppType.caption),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

/// เส้นคั่นพร้อมข้อความตรงกลาง (— หรือ —)
class _OrDivider extends StatelessWidget {
  final String label;
  const _OrDivider({required this.label});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.only(top: 18),
      child: Row(
        children: [
          const Expanded(child: Divider()),
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 12),
            child: Text(label, style: AppType.caption),
          ),
          const Expanded(child: Divider()),
        ],
      ),
    );
  }
}
```

> 🔗 **การเชื่อมโยง:** หน้า Login คือ "ชุมทาง" — แตกเส้นทางไป 3 หน้า (`MainShell` ด้วย `pushReplacement`, `ForgotPasswordPage`/`RegisterPage` ด้วย `push`) และใช้วิดเจ็ตกลางครบเกือบทุกตัว (AppLogo, AppTextField, InfoHint, AppButton ทั้ง 3 แบบ) — เป็นหน้าที่เหมาะใช้สาธิต Design System ที่สุด

#### B3.3 `register_page.dart` — สมัครสมาชิก

จุดเรียนรู้: ปุ่ม "สร้างบัญชี" ผูกกับ state `_accept` — ถ้ายังไม่ติ๊กยอมรับเงื่อนไข `onPressed` เป็น `null` → `AppButton.primary` เปลี่ยนเป็นสีเทา disabled อัตโนมัติ

```dart
// lib/features/auth/presentation/pages/register_page.dart
import 'package:flutter/material.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_text_field.dart';
import '../../../../core/widgets/gradient_header.dart';

/// หน้าสมัครสมาชิก — UI ครบตาม mockup (Day 1 ยังไม่ส่งข้อมูลจริง)
class RegisterPage extends StatefulWidget {
  const RegisterPage({super.key});

  @override
  State<RegisterPage> createState() => _RegisterPageState();
}

class _RegisterPageState extends State<RegisterPage> {
  bool _accept = false;

  @override
  Widget build(BuildContext context) {
    final bottomInset = MediaQuery.paddingOf(context).bottom;
    return Scaffold(
      body: ListView(
        padding: EdgeInsets.zero,
        children: [
          const GradientHeader(
            showBack: true,
            title: 'สมัครสมาชิก',
            subtitle: 'กรอกข้อมูลเพื่อเปิดบัญชีใหม่',
          ),
          Padding(
            padding: EdgeInsets.fromLTRB(20, 20, 20, 28 + bottomInset),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                const AppTextField(
                    label: 'ชื่อ-นามสกุล',
                    hint: 'เช่น สมชาย ใจดี',
                    icon: HugeIcons.strokeRoundedUser),
                const SizedBox(height: 14),
                const AppTextField(
                    label: 'อีเมล',
                    hint: 'you@example.com',
                    icon: HugeIcons.strokeRoundedMail01,
                    keyboardType: TextInputType.emailAddress),
                const SizedBox(height: 14),
                const AppTextField(
                    label: 'เบอร์โทรศัพท์',
                    hint: '08X-XXX-XXXX',
                    icon: HugeIcons.strokeRoundedCall,
                    keyboardType: TextInputType.phone),
                const SizedBox(height: 14),
                const AppTextField(
                    label: 'รหัสผ่าน',
                    hint: 'อย่างน้อย 8 ตัวอักษร',
                    icon: HugeIcons.strokeRoundedSquareLock01,
                    obscure: true),
                const SizedBox(height: 14),
                const AppTextField(
                    label: 'ยืนยันรหัสผ่าน',
                    hint: 'พิมพ์รหัสผ่านอีกครั้ง',
                    icon: HugeIcons.strokeRoundedSquareLock01,
                    obscure: true),
                const SizedBox(height: 14),
                Row(
                  children: [
                    Checkbox(
                      value: _accept,
                      onChanged: (v) => setState(() => _accept = v ?? false),
                      activeColor: AppColors.primary,
                    ),
                    Expanded(
                      child: Text.rich(
                        TextSpan(
                          style: AppType.caption,
                          children: const [
                            TextSpan(text: 'ฉันยอมรับ '),
                            TextSpan(
                              text: 'ข้อกำหนดและนโยบายความเป็นส่วนตัว',
                              style: TextStyle(
                                  color: AppColors.primary,
                                  fontWeight: FontWeight.w600),
                            ),
                          ],
                        ),
                      ),
                    ),
                  ],
                ),
                const SizedBox(height: 8),
                AppButton.primary(
                  label: 'สร้างบัญชี',
                  icon: HugeIcons.strokeRoundedUserAdd01,
                  onPressed: _accept ? () => Navigator.of(context).pop() : null,
                ),
                const SizedBox(height: 16),
                AppButton.social(
                    label: 'ดำเนินการต่อด้วย Google',
                    icon: HugeIcons.strokeRoundedGoogle,
                    color: const Color(0xFF4285F4),
                    onPressed: () {}),
                const SizedBox(height: 12),
                AppButton.social(
                    label: 'ดำเนินการต่อด้วย LINE',
                    icon: HugeIcons.strokeRoundedBubbleChat,
                    color: const Color(0xFF06C755),
                    onPressed: () {}),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

#### B3.4 `forgot_password_page.dart` — ลืมรหัสผ่าน

```dart
// lib/features/auth/presentation/pages/forgot_password_page.dart
import 'package:flutter/material.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_text_field.dart';
import '../../../../core/widgets/gradient_header.dart';
import 'reset_password_page.dart';

/// หน้าลืมรหัสผ่าน — กรอกอีเมลเพื่อรับลิงก์รีเซ็ต (Day 1: UI อย่างเดียว)
class ForgotPasswordPage extends StatelessWidget {
  const ForgotPasswordPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: ListView(
        padding: EdgeInsets.zero,
        children: [
          const GradientHeader(
            showBack: true,
            title: 'ลืมรหัสผ่าน',
            subtitle: 'กรอกอีเมลที่ลงทะเบียนไว้ เราจะส่งลิงก์รีเซ็ตรหัสผ่านให้คุณ',
          ),
          Padding(
            padding: const EdgeInsets.fromLTRB(20, 32, 20, 24),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                Center(
                  child: Container(
                    width: 72,
                    height: 72,
                    decoration: BoxDecoration(
                      color: AppColors.background,
                      borderRadius: BorderRadius.circular(20),
                      border: Border.all(color: AppColors.border),
                    ),
                    child: const HugeIcon(icon: HugeIcons.strokeRoundedLockKey,
                        size: 34, color: AppColors.primary),
                  ),
                ),
                const SizedBox(height: 28),
                const AppTextField(
                    label: 'อีเมล',
                    hint: 'you@example.com',
                    icon: HugeIcons.strokeRoundedMail01,
                    keyboardType: TextInputType.emailAddress),
                const SizedBox(height: 18),
                AppButton.primary(
                  label: 'ส่งลิงก์รีเซ็ต',
                  icon: HugeIcons.strokeRoundedMail01,
                  onPressed: () => Navigator.of(context).push(
                    MaterialPageRoute(
                        builder: (_) => const ResetPasswordPage()),
                  ),
                ),
                const SizedBox(height: 8),
                Center(
                  child: TextButton.icon(
                    onPressed: () => Navigator.of(context).pop(),
                    icon: const HugeIcon(icon: HugeIcons.strokeRoundedArrowLeft01, size: 18),
                    label: const Text('กลับไปหน้าเข้าสู่ระบบ'),
                    style: TextButton.styleFrom(
                        foregroundColor: AppColors.textMedium),
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

#### B3.5 `reset_password_page.dart` — ตั้งรหัสผ่านใหม่

จุดเรียนรู้: ปุ่ม "ตั้งรหัสผ่านใหม่" ใช้ `popUntil((route) => route.isFirst)` — เด้งกลับข้าม stack (Reset → Forgot → Login) ไปหน้าแรกสุดในครั้งเดียว

```dart
// lib/features/auth/presentation/pages/reset_password_page.dart
import 'package:flutter/material.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_text_field.dart';
import '../../../../core/widgets/gradient_header.dart';
import '../../../../core/widgets/info_hint.dart';

/// หน้าตั้งรหัสผ่านใหม่ (Day 1: UI อย่างเดียว)
class ResetPasswordPage extends StatelessWidget {
  const ResetPasswordPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: ListView(
        padding: EdgeInsets.zero,
        children: [
          const GradientHeader(
            showBack: true,
            title: 'ตั้งรหัสผ่านใหม่',
            subtitle: 'สร้างรหัสผ่านใหม่สำหรับบัญชีของคุณ',
          ),
          Padding(
            padding: const EdgeInsets.fromLTRB(20, 32, 20, 24),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                Center(
                  child: Container(
                    width: 72,
                    height: 72,
                    decoration: BoxDecoration(
                      color: AppColors.background,
                      borderRadius: BorderRadius.circular(20),
                      border: Border.all(color: AppColors.border),
                    ),
                    child: const HugeIcon(icon: HugeIcons.strokeRoundedSecurityCheck,
                        size: 34, color: AppColors.primary),
                  ),
                ),
                const SizedBox(height: 28),
                const AppTextField(
                    label: 'รหัสผ่านใหม่',
                    hint: 'อย่างน้อย 8 ตัวอักษร',
                    icon: HugeIcons.strokeRoundedSquareLock01,
                    obscure: true),
                const SizedBox(height: 14),
                const AppTextField(
                    label: 'ยืนยันรหัสผ่านใหม่',
                    hint: 'พิมพ์รหัสผ่านอีกครั้ง',
                    icon: HugeIcons.strokeRoundedSquareLock01,
                    obscure: true),
                const SizedBox(height: 12),
                const InfoHint('ใช้อย่างน้อย 8 ตัวอักษร ประกอบด้วยตัวอักษรและตัวเลข'),
                const SizedBox(height: 18),
                AppButton.primary(
                  label: 'ตั้งรหัสผ่านใหม่',
                  icon: HugeIcons.strokeRoundedSecurityCheck,
                  onPressed: () => Navigator.of(context)
                      .popUntil((route) => route.isFirst),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

> 🔗 **การเชื่อมโยงกลุ่ม Auth:** ทั้ง 4 หน้า (login/register/forgot/reset) ประกอบจากวิดเจ็ตกลางล้วน ๆ — ไม่มีการประกาศสี/ฟอนต์/กรอบเองแม้แต่จุดเดียว นี่คือประโยชน์ตรงของ Design System: หน้าใหม่เขียนเร็วและหน้าตาสม่ำเสมออัตโนมัติ

### B4. หน้าแรก — `home_page.dart` (Dashboard, แท็บ 0)

จุดเรียนรู้สำคัญ 3 จุด: (1) การ์ดสรุปใช้ `Transform.translate(offset: Offset(0, -52))` ให้ **ซ้อนทับขึ้นมาบนหัว gradient** (เทคนิคดีไซน์ overlap ยอดนิยม) (2) `_QuickActions` เป็น `ConsumerWidget` เพราะปุ่มลัดต้อง `ref.read(mainTabProvider.notifier)` เพื่อสลับแท็บข้ามฟีเจอร์ (3) รายการแจ้งเตือนเป็น mock data ที่โมเดล `_Noti` เป็น private class ภายในไฟล์ — ยังไม่ต้องแยกชั้น domain เพราะเป็น static UI (Day 2 จะดึงจาก API จริง)

```dart
// lib/features/home/presentation/pages/home_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_spacing.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_card.dart';
import '../../../../core/widgets/gradient_header.dart';
import '../../../../core/widgets/section_label.dart';
import '../../../../core/widgets/status_badge.dart';
import '../../../app/main_shell.dart';
import '../../../claim/presentation/widgets/file_claim_sheet.dart';

/// หน้าแรก (Dashboard) — สรุปวงเงิน, เบี้ยครบกำหนด, ปุ่มลัด, การแจ้งเตือน
/// Day 1: ข้อมูล mock แบบ static — Day 2 จะดึงจาก API จริง
class HomePage extends ConsumerWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ListView(
      padding: EdgeInsets.zero,
      children: [
        GradientHeader(
          padding: const EdgeInsets.fromLTRB(20, 8, 20, 70),
          child: Padding(
            padding: const EdgeInsets.only(top: 4),
            child: Row(
              children: [
                const CircleAvatar(
                  radius: 22,
                  backgroundColor: Colors.white24,
                  backgroundImage:
                      AssetImage('assets/images/bla_avatar.png'),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text('สวัสดี 👋',
                          style: AppType.caption.copyWith(color: Colors.white70)),
                      Text('คุณสมชาย ใจดี',
                          style: AppType.h2.copyWith(color: Colors.white)),
                    ],
                  ),
                ),
                const CircleHeaderButton(icon: HugeIcons.strokeRoundedNotification02),
              ],
            ),
          ),
        ),
        // การ์ดสรุป ซ้อนทับขึ้นมาบน header
        Transform.translate(
          offset: const Offset(0, -52),
          child: Padding(
            padding: AppSpacing.pageH,
            child: Column(
              children: [
                _SummaryCard(),
                const SizedBox(height: 14),
                _DuePremiumCard(),
                const SizedBox(height: 18),
                const _QuickActions(),
                const SizedBox(height: 8),
                SectionLabel('การแจ้งเตือนล่าสุด',
                    trailing: Text('ดูทั้งหมด',
                        style: AppType.label
                            .copyWith(color: AppColors.primary))),
                ..._notifications.map((n) => Padding(
                      padding: const EdgeInsets.only(bottom: 10),
                      child: _NotificationTile(n),
                    )),
              ],
            ),
          ),
        ),
      ],
    );
  }
}

class _SummaryCard extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return AppCard(
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text('วงเงินคุ้มครองรวม', style: AppType.label),
          const SizedBox(height: 4),
          Text('฿1,250,000', style: AppType.display),
          const SizedBox(height: 14),
          const Row(
            children: [
              Expanded(child: _MiniStat(label: 'กรมธรรม์มีผลบังคับ', value: '2 ฉบับ')),
              SizedBox(width: 12),
              Expanded(child: _MiniStat(label: 'เบี้ยรวม/ปี', value: '฿60,000')),
            ],
          ),
        ],
      ),
    );
  }
}

class _MiniStat extends StatelessWidget {
  final String label;
  final String value;
  const _MiniStat({required this.label, required this.value});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(12),
      decoration: BoxDecoration(
        color: AppColors.background,
        borderRadius: BorderRadius.circular(AppSpacing.rSm),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(label, style: AppType.caption),
          const SizedBox(height: 4),
          Text(value, style: AppType.h3),
        ],
      ),
    );
  }
}

class _DuePremiumCard extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        gradient: AppColors.primaryGradient,
        borderRadius: BorderRadius.circular(AppSpacing.rMd),
        boxShadow: [
          BoxShadow(
              color: AppColors.primary.withValues(alpha: 0.30),
              blurRadius: 16,
              offset: const Offset(0, 6)),
        ],
      ),
      child: Row(
        children: [
          Container(
            width: 44,
            height: 44,
            decoration: BoxDecoration(
                color: Colors.white24,
                borderRadius: BorderRadius.circular(12)),
            child: const HugeIcon(
                icon: HugeIcons.strokeRoundedCalendar03, color: Colors.white),
          ),
          const SizedBox(width: 12),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text('เบี้ยครบกำหนด ฿50,000',
                    style: AppType.bodyStrong.copyWith(color: Colors.white)),
                const SizedBox(height: 2),
                Text('BLA สะสมทรัพย์ 10/5 · 15 ก.ค. 2569',
                    style: AppType.caption.copyWith(color: Colors.white70)),
              ],
            ),
          ),
          FilledButton(
            onPressed: () {},
            style: FilledButton.styleFrom(
              backgroundColor: Colors.white,
              foregroundColor: AppColors.primary,
              padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
            ),
            child: const Text('ชำระ'),
          ),
        ],
      ),
    );
  }
}

class _QuickActions extends ConsumerWidget {
  const _QuickActions();

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final actions = <({AppIconData icon, String label, VoidCallback onTap})>[
      (
        icon: HugeIcons.strokeRoundedCreditCard,
        label: 'ชำระเบี้ย',
        onTap: () => ref.read(mainTabProvider.notifier).state = 3,
      ),
      (
        icon: HugeIcons.strokeRoundedNoteAdd,
        label: 'ยื่นสินไหม',
        onTap: () => showFileClaimSheet(context),
      ),
      (
        icon: HugeIcons.strokeRoundedShield01,
        label: 'กรมธรรม์',
        onTap: () => ref.read(mainTabProvider.notifier).state = 1,
      ),
      (
        icon: HugeIcons.strokeRoundedCustomerService01,
        label: 'ติดต่อเรา',
        onTap: () {},
      ),
    ];
    return Row(
      children: [
        for (final a in actions)
          Expanded(
            child: GestureDetector(
              onTap: a.onTap,
              child: Container(
                margin: const EdgeInsets.symmetric(horizontal: 4),
                padding: const EdgeInsets.fromLTRB(4, 12, 4, 10),
                decoration: BoxDecoration(
                  color: AppColors.card,
                  borderRadius: BorderRadius.circular(AppSpacing.rMd),
                  border: Border.all(color: AppColors.border),
                  boxShadow: const [
                    BoxShadow(
                      color: Color(0x0F1855A3),
                      blurRadius: 8,
                      offset: Offset(0, 2),
                    ),
                  ],
                ),
                child: Column(
                  children: [
                    Container(
                      width: 38,
                      height: 38,
                      decoration: BoxDecoration(
                        color: const Color(0x141855A3),
                        borderRadius: BorderRadius.circular(11),
                      ),
                      child: HugeIcon(
                          icon: a.icon, color: AppColors.primary, size: 22),
                    ),
                    const SizedBox(height: 7),
                    Text(a.label,
                        style: AppType.caption, textAlign: TextAlign.center),
                  ],
                ),
              ),
            ),
          ),
      ],
    );
  }
}

class _NotificationTile extends StatelessWidget {
  final _Noti n;
  const _NotificationTile(this.n);

  @override
  Widget build(BuildContext context) {
    return AppCard(
      padding: const EdgeInsets.all(12),
      child: Row(
        children: [
          Container(
            width: 40,
            height: 40,
            decoration: BoxDecoration(
                color: n.tone == BadgeTone.success
                    ? AppColors.successBg
                    : n.tone == BadgeTone.info
                        ? AppColors.infoBg
                        : AppColors.pendingBg,
                borderRadius: BorderRadius.circular(10)),
            child: HugeIcon(icon: n.icon, size: 20, color: _iconColor(n.tone)),
          ),
          const SizedBox(width: 12),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(n.title, style: AppType.bodyStrong),
                const SizedBox(height: 2),
                Text(n.subtitle, style: AppType.caption),
              ],
            ),
          ),
          Text(n.time, style: AppType.tiny),
        ],
      ),
    );
  }

  Color _iconColor(BadgeTone t) => switch (t) {
        BadgeTone.success => AppColors.successFg,
        BadgeTone.info => AppColors.infoText,
        _ => AppColors.pendingFg,
      };
}

class _Noti {
  final AppIconData icon;
  final String title;
  final String subtitle;
  final String time;
  final BadgeTone tone;
  const _Noti(this.icon, this.title, this.subtitle, this.time, this.tone);
}

const _notifications = [
  _Noti(HugeIcons.strokeRoundedCreditCard, 'เบี้ยใกล้ครบกำหนด',
      'BLA สะสมทรัพย์ 10/5 · ฿50,000', 'อีก 17 วัน', BadgeTone.info),
  _Noti(HugeIcons.strokeRoundedCheckmarkCircle02, 'สินไหมอนุมัติแล้ว',
      'CLM-48213 · ค่ารักษาพยาบาล', '2 วันที่แล้ว', BadgeTone.success),
  _Noti(HugeIcons.strokeRoundedShield01, 'กรมธรรม์มีผลบังคับ', 'BLA ตลอดชีพ มั่นคง 99',
      '1 สัปดาห์ที่แล้ว', BadgeTone.success),
];
```

> 🔗 **การเชื่อมโยง:** HomePage เป็นหน้าเดียวที่ import **ข้ามฟีเจอร์ 2 ทาง** — `main_shell.dart` (เพื่อใช้ `mainTabProvider` สลับแท็บ) และ `claim/.../file_claim_sheet.dart` (เปิด sheet ยื่นสินไหม) · การสลับแท็บผ่าน provider แทน callback คือตัวอย่างแรกของ "สื่อสารข้ามหน้าจอผ่าน state กลาง" ที่จะใช้หนักขึ้นใน Day 2–3

### B5. หน้ารายละเอียดกรมธรรม์ — `policy_detail_page.dart`

รับ `Policy` ผ่าน constructor (ส่งข้ามหน้าจาก `_PolicyCard` ใน list) — หน้านี้มี `Scaffold`+`AppBar` ของตัวเองเพราะถูก `push` เต็มจอ (ไม่ได้อยู่ใน MainShell แบบ 5 แท็บ)

```dart
// lib/features/policy/presentation/pages/policy_detail_page.dart
import 'package:flutter/material.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_spacing.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_card.dart';
import '../../../claim/presentation/widgets/file_claim_sheet.dart';
import '../../domain/policy.dart';

/// หน้ารายละเอียดกรมธรรม์
/// Day 1: รับ Policy ที่ส่งมาแล้วแสดงผล (ข้อมูลคงที่บางส่วนเป็น mock)
/// Day 2: จะดึงรายละเอียดเต็มจาก API ตาม id
class PolicyDetailPage extends StatelessWidget {
  final Policy policy;
  const PolicyDetailPage({super.key, required this.policy});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: AppColors.primary,
        title: const Text('รายละเอียดกรมธรรม์',
            style: TextStyle(color: Colors.white, fontWeight: FontWeight.w600)),
        iconTheme: const IconThemeData(color: Colors.white),
      ),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          // การ์ดหัว gradient
          Container(
            padding: const EdgeInsets.all(20),
            decoration: BoxDecoration(
              gradient: AppColors.headerGradient,
              borderRadius: BorderRadius.circular(AppSpacing.rLg),
            ),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                _MiniBadge(label: policy.status.label, status: policy.status),
                const SizedBox(height: 12),
                Text(policy.planName,
                    style: AppType.h2.copyWith(color: Colors.white)),
                const SizedBox(height: 2),
                Text(policy.policyNumber,
                    style: AppType.caption.copyWith(color: Colors.white70)),
                const SizedBox(height: 16),
                Container(
                  width: double.infinity,
                  padding: const EdgeInsets.all(14),
                  decoration: BoxDecoration(
                    color: Colors.white.withValues(alpha: 0.15),
                    borderRadius: BorderRadius.circular(AppSpacing.rMd),
                  ),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text('วงเงินคุ้มครอง',
                          style:
                              AppType.caption.copyWith(color: Colors.white70)),
                      const SizedBox(height: 4),
                      Text('฿500,000',
                          style:
                              AppType.display.copyWith(color: Colors.white)),
                    ],
                  ),
                ),
              ],
            ),
          ),
          const SizedBox(height: 16),
          AppCard(
            child: Column(
              children: [
                Align(
                  alignment: Alignment.centerLeft,
                  child: Text('ข้อมูลกรมธรรม์', style: AppType.h3),
                ),
                const SizedBox(height: 8),
                const Divider(),
                _row('ผู้เอาประกัน', 'นายสมชาย ใจดี'),
                _row('เบี้ยประกัน', '฿${policy.premium.toStringAsFixed(0)}/ปี'),
                _row('วันเริ่มสัญญา', '1 ม.ค. 2565'),
                _row('วันสิ้นสุด', '1 ม.ค. 2664'),
              ],
            ),
          ),
          const SizedBox(height: 16),
          Row(
            children: [
              Expanded(
                child: AppButton.secondary(
                    label: 'ชำระเบี้ย',
                    icon: HugeIcons.strokeRoundedCreditCard,
                    onPressed: () {}),
              ),
              const SizedBox(width: 12),
              Expanded(
                child: AppButton.primary(
                    label: 'ยื่นสินไหม',
                    icon: HugeIcons.strokeRoundedNoteAdd,
                    onPressed: () =>
                        showFileClaimSheet(context, policyName: policy.planName)),
              ),
            ],
          ),
        ],
      ),
    );
  }

  static Widget _row(String label, String value) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 10),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(label, style: AppType.body),
          Text(value, style: AppType.bodyStrong),
        ],
      ),
    );
  }
}

class _MiniBadge extends StatelessWidget {
  final String label;
  final PolicyStatus status;
  const _MiniBadge({required this.label, required this.status});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 5),
      decoration: BoxDecoration(
        color: Colors.white.withValues(alpha: 0.2),
        borderRadius: BorderRadius.circular(AppSpacing.rPill),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          Container(
            width: 7,
            height: 7,
            decoration: const BoxDecoration(
                color: Colors.white, shape: BoxShape.circle),
          ),
          const SizedBox(width: 6),
          Text(label,
              style: AppType.tiny.copyWith(
                  color: Colors.white, fontWeight: FontWeight.w600)),
        ],
      ),
    );
  }
}
```

> 🔗 **การเชื่อมโยง:** รับข้อมูลจาก `PolicyListPage` แบบ **ส่ง object ตรง ๆ ผ่าน constructor** (วิธีพื้นฐานที่สุด — Day 3 จะเทียบกับการส่ง id ผ่าน GoRouter แล้วให้หน้า detail ดึงเองจาก provider) · ปุ่ม "ยื่นสินไหม" เรียก `showFileClaimSheet(..., policyName: ...)` ส่งชื่อกรมธรรม์เข้า sheet — ตัวอย่างการ reuse วิดเจ็ตข้ามฟีเจอร์ (policy → claim)

### B6. ฟีเจอร์สินไหม — `claims_page.dart` + `file_claim_sheet.dart`

#### B6.1 `claims_page.dart` (แท็บ 2)

```dart
// lib/features/claim/presentation/pages/claims_page.dart
import 'package:flutter/material.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_card.dart';
import '../../../../core/widgets/gradient_header.dart';
import '../../../../core/widgets/section_label.dart';
import '../../../../core/widgets/status_badge.dart';
import '../widgets/file_claim_sheet.dart';

/// หน้าสินไหม — สรุป + ประวัติการยื่น
/// Day 1: ข้อมูล mock static — Day 2 จะดึงจาก API + เพิ่ม "ยื่นสินไหม" จริง (Mutation)
class ClaimsPage extends StatelessWidget {
  const ClaimsPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const GradientHeader(
          title: 'สินไหม',
          subtitle: 'ยื่นและติดตามสถานะคำขอ',
        ),
        Expanded(
          child: ListView(
            padding: const EdgeInsets.fromLTRB(16, 16, 16, 16),
            children: [
              Row(
                children: [
                  Expanded(
                      child: _Stat(
                          label: 'คำขอทั้งหมด', value: '4 รายการ')),
                  const SizedBox(width: 12),
                  Expanded(
                      child: _Stat(
                          label: 'อนุมัติแล้ว',
                          value: '฿20,700',
                          valueColor: AppColors.successFg)),
                ],
              ),
              const SizedBox(height: 14),
              AppButton.primary(
                  label: 'ยื่นสินไหมใหม่',
                  icon: HugeIcons.strokeRoundedAdd01,
                  onPressed: () => showFileClaimSheet(context)),
              const SizedBox(height: 8),
              const SectionLabel('ประวัติการยื่น'),
              ..._claims.map((c) =>
                  Padding(padding: const EdgeInsets.only(bottom: 10),
                      child: _ClaimCard(c))),
            ],
          ),
        ),
      ],
    );
  }
}

class _Stat extends StatelessWidget {
  final String label;
  final String value;
  final Color? valueColor;
  const _Stat({required this.label, required this.value, this.valueColor});

  @override
  Widget build(BuildContext context) {
    return AppCard(
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(label, style: AppType.caption),
          const SizedBox(height: 4),
          Text(value, style: AppType.h1.copyWith(color: valueColor)),
        ],
      ),
    );
  }
}

class _ClaimCard extends StatelessWidget {
  final _Claim c;
  const _ClaimCard(this.c);

  @override
  Widget build(BuildContext context) {
    return AppCard(
      child: Column(
        children: [
          Row(
            children: [
              Container(
                width: 40,
                height: 40,
                decoration: BoxDecoration(
                    color: _bg(c.tone),
                    borderRadius: BorderRadius.circular(10)),
                child: HugeIcon(icon: c.icon, size: 20, color: _fg(c.tone)),
              ),
              const SizedBox(width: 12),
              Expanded(
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(c.title, style: AppType.bodyStrong),
                    const SizedBox(height: 2),
                    Text(c.plan, style: AppType.caption),
                  ],
                ),
              ),
              StatusBadge(label: c.statusLabel, tone: c.tone),
            ],
          ),
          const Divider(height: 20),
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              Text('${c.ref} · ${c.date}', style: AppType.caption),
              Text('฿${c.amount}', style: AppType.h3),
            ],
          ),
        ],
      ),
    );
  }

  Color _bg(BadgeTone t) => switch (t) {
        BadgeTone.success => AppColors.successBg,
        BadgeTone.pending => AppColors.pendingBg,
        BadgeTone.danger => AppColors.dangerBg,
        _ => AppColors.infoBg,
      };
  Color _fg(BadgeTone t) => switch (t) {
        BadgeTone.success => AppColors.successFg,
        BadgeTone.pending => AppColors.pendingFg,
        BadgeTone.danger => AppColors.dangerFg,
        _ => AppColors.infoText,
      };
}

class _Claim {
  final AppIconData icon;
  final String title;
  final String plan;
  final String statusLabel;
  final BadgeTone tone;
  final String ref;
  final String date;
  final String amount;
  const _Claim(this.icon, this.title, this.plan, this.statusLabel, this.tone,
      this.ref, this.date, this.amount);
}

const _claims = [
  _Claim(HugeIcons.strokeRoundedCheckmarkCircle02, 'ค่ารักษาพยาบาล', 'BLA คุ้มครองสุขภาพ พลัส',
      'อนุมัติแล้ว', BadgeTone.success, 'CLM-48213', '12 มิ.ย. 2569', '12,500'),
  _Claim(HugeIcons.strokeRoundedClock01, 'ค่าชดเชยรายวัน', 'BLA ตลอดชีพ มั่นคง 99',
      'กำลังพิจารณา', BadgeTone.pending, 'CLM-47980', '3 มิ.ย. 2569', '6,000'),
  _Claim(HugeIcons.strokeRoundedMoney01, 'ค่ารักษาพยาบาล', 'BLA คุ้มครองสุขภาพ พลัส',
      'จ่ายแล้ว', BadgeTone.info, 'CLM-47621', '21 พ.ค. 2569', '8,200'),
  _Claim(HugeIcons.strokeRoundedCancelCircle, 'ค่ารักษาพยาบาล', 'BLA บำนาญมั่นคง 60',
      'ไม่อนุมัติ', BadgeTone.danger, 'CLM-47102', '8 พ.ค. 2569', '3,400'),
];
```

#### B6.2 `file_claim_sheet.dart` — Bottom Sheet ยื่นสินไหม (ใช้ร่วม 3 ที่)

จุดเรียนรู้: (1) แพตเทิร์น **helper function** `showFileClaimSheet(...)` ครอบ `showModalBottomSheet` ให้ผู้เรียกใช้บรรทัดเดียว (2) sheet มี 2 สถานะในตัว (ฟอร์ม ↔ สำเร็จ) สลับด้วย state ภายใน `_claimRef` (3) การคำนวณ `bottomInset` จาก `viewInsets` (คีย์บอร์ด) + `padding.bottom` (แถบ navigation) เพื่อไม่ให้ปุ่มถูกบัง

```dart
// lib/features/claim/presentation/widgets/file_claim_sheet.dart
import 'dart:math';

import 'package:flutter/material.dart';

import '../../../../core/theme/app_colors.dart';
import '../../../../core/icons/app_icons.dart';
import '../../../../core/theme/app_spacing.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_text_field.dart';

/// แสดง Modal Bottom Sheet "ยื่นสินไหม"
/// เรียกจากหน้าแรก, รายละเอียดกรมธรรม์ และหน้าสินไหม
/// [policyName] = ชื่อกรมธรรม์ที่จะยื่น (แสดงเป็นหัวข้อย่อย ถ้ามี)
Future<void> showFileClaimSheet(BuildContext context, {String? policyName}) {
  return showModalBottomSheet<void>(
    context: context,
    isScrollControlled: true,
    backgroundColor: Colors.transparent,
    builder: (_) => FileClaimSheet(policyName: policyName),
  );
}

class FileClaimSheet extends StatefulWidget {
  final String? policyName;
  const FileClaimSheet({super.key, this.policyName});

  @override
  State<FileClaimSheet> createState() => _FileClaimSheetState();
}

class _FileClaimSheetState extends State<FileClaimSheet> {
  final _amount = TextEditingController();
  final _reason = TextEditingController();

  // เลขอ้างอิงหลังยื่นสำเร็จ — ถ้ายังเป็น null แสดงฟอร์ม
  String? _claimRef;

  @override
  void dispose() {
    _amount.dispose();
    _reason.dispose();
    super.dispose();
  }

  void _submit() {
    final n = 10000 + Random().nextInt(90000);
    setState(() => _claimRef = 'CLM-$n');
  }

  @override
  Widget build(BuildContext context) {
    final media = MediaQuery.of(context);
    // เผื่อพื้นที่ให้คีย์บอร์ดดันเนื้อหาขึ้น (viewInsets) และเผื่อแถบ
    // navigation ของระบบ (padding.bottom) ไม่ให้ปุ่มถูกบดบัง
    final bottomInset = media.viewInsets.bottom + media.padding.bottom;
    return Container(
      decoration: const BoxDecoration(
        color: AppColors.card,
        borderRadius: BorderRadius.vertical(top: Radius.circular(24)),
      ),
      padding: EdgeInsets.fromLTRB(20, 10, 20, 20 + bottomInset),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // แถบจับลาก
          Center(
            child: Container(
              width: 44,
              height: 5,
              margin: const EdgeInsets.only(bottom: 16),
              decoration: BoxDecoration(
                color: AppColors.border,
                borderRadius: BorderRadius.circular(AppSpacing.rPill),
              ),
            ),
          ),
          if (_claimRef == null) ..._form() else ..._success(_claimRef!),
        ],
      ),
    );
  }

  // ── ฟอร์มกรอกข้อมูลการยื่นสินไหม ───────────────────────────────
  List<Widget> _form() {
    return [
      Text('ยื่นสินไหม', style: AppType.h2),
      if (widget.policyName != null) ...[
        const SizedBox(height: 2),
        Text(widget.policyName!,
            style: AppType.caption.copyWith(color: AppColors.primary)),
      ],
      const SizedBox(height: 16),
      AppTextField(
        label: 'จำนวนเงิน (บาท)',
        hint: '0.00',
        keyboardType: const TextInputType.numberWithOptions(decimal: true),
        controller: _amount,
      ),
      const SizedBox(height: 14),
      AppTextField(
        label: 'เหตุผล / รายละเอียด',
        hint: 'ระบุสาเหตุและรายละเอียดการยื่นสินไหม...',
        maxLines: 4,
        controller: _reason,
      ),
      const SizedBox(height: 20),
      AppButton.primary(
        label: 'ยืนยันการยื่น',
        onPressed: _submit,
      ),
    ];
  }

  // ── หน้ายืนยันยื่นสำเร็จ ───────────────────────────────────────
  List<Widget> _success(String ref) {
    return [
      const SizedBox(height: 8),
      Center(
        child: Container(
          width: 72,
          height: 72,
          decoration: const BoxDecoration(
            color: AppColors.successBg,
            shape: BoxShape.circle,
          ),
          child: const HugeIcon(
            icon: HugeIcons.strokeRoundedCheckmarkCircle02,
            size: 40,
            color: AppColors.successFg,
          ),
        ),
      ),
      const SizedBox(height: 16),
      Center(child: Text('ยื่นสินไหมสำเร็จ', style: AppType.h2)),
      const SizedBox(height: 4),
      Center(
        child: Text('เลขอ้างอิงการยื่น',
            style: AppType.caption.copyWith(color: AppColors.textSubtle)),
      ),
      const SizedBox(height: 10),
      Center(
        child: Container(
          padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
          decoration: BoxDecoration(
            color: AppColors.background,
            borderRadius: BorderRadius.circular(AppSpacing.rMd),
          ),
          child: Text(ref,
              style: AppType.h2.copyWith(color: AppColors.primary)),
        ),
      ),
      const SizedBox(height: 20),
      AppButton.primary(
        label: 'ปิด',
        onPressed: () => Navigator.of(context).pop(),
      ),
    ];
  }
}
```

> 🔗 **การเชื่อมโยง:** `showFileClaimSheet` ถูกเรียกจาก **3 ที่**: `HomePage` (ปุ่มลัด), `PolicyDetailPage` (ส่ง `policyName` มาด้วย), `ClaimsPage` (ปุ่มยื่นใหม่) — วางไฟล์ไว้ที่ `claim/presentation/widgets/` เพราะ "เจ้าของ" เชิงธุรกิจคือฟีเจอร์ claim แม้จะถูกใช้ข้ามฟีเจอร์ · Day 2 จะเปลี่ยน `_submit()` จาก mock เป็น Mutation ผ่าน AsyncNotifier จริง

### B7. หน้าชำระเบี้ย — `payments_page.dart` (แท็บ 3)

หน้านี้ static ล้วน (ไม่มี state เลย — เป็น `StatelessWidget` ทั้งไฟล์) · มี `DottedAddBox` เป็น public widget ที่ประกาศไว้ในไฟล์นี้ (candidate ที่จะย้ายไป `core/widgets` เมื่อมีหน้าอื่นใช้ซ้ำ — ประเด็นชวนคิดเรื่อง "เมื่อไหร่ควร promote widget ขึ้น core")

```dart
// lib/features/payment/presentation/pages/payments_page.dart
import 'package:flutter/material.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_spacing.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_card.dart';
import '../../../../core/widgets/gradient_header.dart';
import '../../../../core/widgets/section_label.dart';
import '../../../../core/widgets/status_badge.dart';

/// หน้าชำระเบี้ย — บิลครบกำหนด + ช่องทางชำระ + ประวัติการชำระ
/// Day 1: static — Day 2 จะทำประวัติแบบ Pagination (Infinite Scroll) จาก API
class PaymentsPage extends StatelessWidget {
  const PaymentsPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const GradientHeader(
          title: 'ชำระเบี้ย',
          subtitle: 'จ่ายเบี้ยและดูประวัติ',
        ),
        Expanded(
          child: ListView(
            padding: const EdgeInsets.fromLTRB(16, 16, 16, 16),
            children: [
              // การ์ดบิลครบกำหนด (gradient)
              Container(
                padding: const EdgeInsets.all(18),
                decoration: BoxDecoration(
                  gradient: AppColors.primaryGradient,
                  borderRadius: BorderRadius.circular(AppSpacing.rLg),
                ),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Container(
                      padding: const EdgeInsets.symmetric(
                          horizontal: 10, vertical: 4),
                      decoration: BoxDecoration(
                          color: Colors.white24,
                          borderRadius:
                              BorderRadius.circular(AppSpacing.rPill)),
                      child: Row(mainAxisSize: MainAxisSize.min, children: [
                        const HugeIcon(
                            icon: HugeIcons.strokeRoundedClock01,
                            size: 14,
                            color: Colors.white),
                        const SizedBox(width: 4),
                        Text('ครบกำหนดอีก 17 วัน',
                            style: AppType.tiny.copyWith(color: Colors.white)),
                      ]),
                    ),
                    const SizedBox(height: 12),
                    Text('BLA สะสมทรัพย์ 10/5',
                        style: AppType.body.copyWith(color: Colors.white70)),
                    Text('฿50,000',
                        style: AppType.display.copyWith(color: Colors.white)),
                    const SizedBox(height: 2),
                    Text('กำหนดชำระ 15 ก.ค. 2569 · BLA-2569-0002',
                        style: AppType.caption.copyWith(color: Colors.white70)),
                    const SizedBox(height: 14),
                    SizedBox(
                      width: double.infinity,
                      child: FilledButton.icon(
                        onPressed: () {},
                        style: FilledButton.styleFrom(
                          backgroundColor: Colors.white,
                          foregroundColor: AppColors.primary,
                          padding: const EdgeInsets.symmetric(vertical: 12),
                        ),
                        icon: const HugeIcon(icon: HugeIcons.strokeRoundedCreditCard, size: 18),
                        label: const Text('จ่ายตอนนี้'),
                      ),
                    ),
                  ],
                ),
              ),
              const SizedBox(height: 8),
              const SectionLabel('ช่องทางชำระเงิน'),
              _MethodTile(
                  icon: HugeIcons.strokeRoundedQrCode,
                  title: 'พร้อมเพย์ / QR',
                  subtitle: 'ตัดผ่านบัญชีธนาคาร',
                  trailing: const StatusBadge(
                      label: 'ค่าเริ่มต้น', tone: BadgeTone.info),
                  selected: true),
              const SizedBox(height: 10),
              _MethodTile(
                  icon: HugeIcons.strokeRoundedCreditCard,
                  title: 'บัตรเครดิต ••6789',
                  subtitle: 'KBank Visa Platinum',
                  trailing: const HugeIcon(icon: HugeIcons.strokeRoundedArrowRight01,
                      color: AppColors.textSubtle)),
              const SizedBox(height: 10),
              _AddMethod(),
              const SizedBox(height: 8),
              const SectionLabel('ประวัติการชำระ'),
              ..._payments.map((p) => Padding(
                  padding: const EdgeInsets.only(bottom: 10),
                  child: _PaymentTile(p))),
            ],
          ),
        ),
      ],
    );
  }
}

class _MethodTile extends StatelessWidget {
  final AppIconData icon;
  final String title;
  final String subtitle;
  final Widget trailing;
  final bool selected;
  const _MethodTile(
      {required this.icon,
      required this.title,
      required this.subtitle,
      required this.trailing,
      this.selected = false});

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(14),
      decoration: BoxDecoration(
        color: AppColors.card,
        borderRadius: BorderRadius.circular(AppSpacing.rMd),
        border: Border.all(
            color: selected ? AppColors.primary : AppColors.border,
            width: selected ? 1.6 : 1),
      ),
      child: Row(
        children: [
          HugeIcon(icon: icon, color: AppColors.primary),
          const SizedBox(width: 12),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(title, style: AppType.bodyStrong),
                Text(subtitle, style: AppType.caption),
              ],
            ),
          ),
          trailing,
        ],
      ),
    );
  }
}

class _AddMethod extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return DottedAddBox(
      label: 'เพิ่มช่องทางชำระเงิน',
      onTap: () {},
    );
  }
}

/// กล่องปุ่ม "เพิ่ม" แบบเส้นประ — reusable
class DottedAddBox extends StatelessWidget {
  final String label;
  final VoidCallback onTap;
  const DottedAddBox({super.key, required this.label, required this.onTap});

  @override
  Widget build(BuildContext context) {
    return InkWell(
      borderRadius: BorderRadius.circular(AppSpacing.rMd),
      onTap: onTap,
      child: Container(
        padding: const EdgeInsets.symmetric(vertical: 14),
        decoration: BoxDecoration(
          borderRadius: BorderRadius.circular(AppSpacing.rMd),
          border: Border.all(color: AppColors.border),
        ),
        child: Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const HugeIcon(icon: HugeIcons.strokeRoundedAdd01, size: 18, color: AppColors.textMedium),
            const SizedBox(width: 8),
            Text(label, style: AppType.label),
          ],
        ),
      ),
    );
  }
}

class _PaymentTile extends StatelessWidget {
  final _Payment p;
  const _PaymentTile(this.p);

  @override
  Widget build(BuildContext context) {
    return AppCard(
      padding: const EdgeInsets.all(12),
      child: Row(
        children: [
          Container(
            width: 40,
            height: 40,
            decoration: BoxDecoration(
                color: AppColors.successBg,
                borderRadius: BorderRadius.circular(10)),
            child: const HugeIcon(icon: HugeIcons.strokeRoundedCheckmarkCircle02,
                size: 20, color: AppColors.successFg),
          ),
          const SizedBox(width: 12),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(p.title, style: AppType.bodyStrong),
                Text('${p.date} · ${p.method}', style: AppType.caption),
              ],
            ),
          ),
          Text('฿${p.amount}', style: AppType.h3),
        ],
      ),
    );
  }
}

class _Payment {
  final String title;
  final String date;
  final String method;
  final String amount;
  const _Payment(this.title, this.date, this.method, this.amount);
}

const _payments = [
  _Payment('BLA ตลอดชีพ มั่นคง 99', '1 ม.ค. 2569', 'พร้อมเพย์ / QR', '24,000'),
  _Payment('BLA บำนาญมั่นคง 60', '10 ส.ค. 2568', 'บัตรเครดิต ••6789', '36,000'),
  _Payment('BLA คุ้มครองสุขภาพ พลัส', '5 มี.ค. 2568', 'พร้อมเพย์ / QR', '18,500'),
];
```

### B8. หน้าโปรไฟล์ — `profile_page.dart` (แท็บ 4)

จุดเรียนรู้: toggle 3 ตัว (`_noti`, `_dark`, `_thai`) เป็น **local state** ด้วย `setState` — เหมาะสมสำหรับ Day 1 เพราะยังไม่มีใครนอกหน้านี้ต้องใช้ค่า (Day 3 เมื่อทำโหมดมืดจริง ค่า `_dark` ต้องย้ายขึ้น provider เพราะ `MaterialApp` ต้อง watch) · ปุ่มออกจากระบบใช้ `pushAndRemoveUntil` ล้าง stack ทั้งหมดกลับหน้า Login

```dart
// lib/features/profile/presentation/pages/profile_page.dart
import 'package:flutter/material.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_spacing.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_card.dart';
import '../../../../core/widgets/gradient_header.dart';
import '../../../../core/widgets/section_label.dart';
import '../../../auth/presentation/pages/login_page.dart';

/// หน้าโปรไฟล์ — ข้อมูลผู้ใช้ + เมนูบัญชี + การตั้งค่า
/// Day 1: UI static + toggle เป็น local state — Day 3 จะต่อ Secure Storage / โหมดมืดจริง
class ProfilePage extends StatefulWidget {
  const ProfilePage({super.key});

  @override
  State<ProfilePage> createState() => _ProfilePageState();
}

class _ProfilePageState extends State<ProfilePage> {
  bool _noti = true;
  bool _dark = false;
  bool _thai = true;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        GradientHeader(
          padding: const EdgeInsets.fromLTRB(20, 12, 20, 22),
          child: Row(
            children: [
              const CircleAvatar(
                radius: 30,
                backgroundColor: Colors.white24,
                backgroundImage: AssetImage('assets/images/bla_avatar.png'),
              ),
              const SizedBox(width: 14),
              Expanded(
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text('คุณสมชาย ใจดี',
                        style: AppType.h2.copyWith(color: Colors.white)),
                    Text('ตัวแทน BLA · รหัส agent',
                        style:
                            AppType.caption.copyWith(color: Colors.white70)),
                    const SizedBox(height: 6),
                    Container(
                      padding: const EdgeInsets.symmetric(
                          horizontal: 8, vertical: 3),
                      decoration: BoxDecoration(
                          color: Colors.white24,
                          borderRadius:
                              BorderRadius.circular(AppSpacing.rPill)),
                      child: Row(mainAxisSize: MainAxisSize.min, children: [
                        const HugeIcon(icon: HugeIcons.strokeRoundedCheckmarkBadge01,
                            size: 13, color: Colors.white),
                        const SizedBox(width: 4),
                        Text('ยืนยันตัวตนแล้ว',
                            style:
                                AppType.tiny.copyWith(color: Colors.white)),
                      ]),
                    ),
                  ],
                ),
              ),
              const CircleHeaderButton(icon: HugeIcons.strokeRoundedEdit02),
            ],
          ),
        ),
        Expanded(
          child: ListView(
            padding: const EdgeInsets.fromLTRB(16, 8, 16, 16),
            children: [
              const SectionLabel('บัญชีของฉัน'),
              _MenuGroup(items: [
                _MenuItem(HugeIcons.strokeRoundedUser, 'ข้อมูลส่วนตัว'),
                _MenuItem(HugeIcons.strokeRoundedShield01, 'กรมธรรม์ของฉัน'),
                _MenuItem(HugeIcons.strokeRoundedClock01, 'ประวัติการชำระ'),
              ]),
              const SectionLabel('การตั้งค่า'),
              AppCard(
                padding: const EdgeInsets.symmetric(horizontal: 8),
                child: Column(
                  children: [
                    _ToggleRow(
                        icon: HugeIcons.strokeRoundedNotification02,
                        label: 'การแจ้งเตือน',
                        value: _noti,
                        onChanged: (v) => setState(() => _noti = v)),
                    const Divider(height: 1),
                    _NavRow(icon: HugeIcons.strokeRoundedSquareLock01, label: 'ความปลอดภัย'),
                    const Divider(height: 1),
                    _SegmentRow(
                        icon: HugeIcons.strokeRoundedGlobe02,
                        label: 'ภาษา',
                        isThai: _thai,
                        onChanged: (v) => setState(() => _thai = v)),
                    const Divider(height: 1),
                    _ToggleRow(
                        icon: HugeIcons.strokeRoundedMoon02,
                        label: 'โหมดมืด',
                        value: _dark,
                        onChanged: (v) => setState(() => _dark = v)),
                  ],
                ),
              ),
              const SectionLabel('ช่วยเหลือ'),
              _MenuGroup(items: [
                _MenuItem(HugeIcons.strokeRoundedCustomerService01, 'ติดต่อตัวแทน'),
                _MenuItem(HugeIcons.strokeRoundedHelpCircle, 'คำถามที่พบบ่อย'),
              ]),
              const SizedBox(height: 20),
              _LogoutButton(
                onTap: () => Navigator.of(context).pushAndRemoveUntil(
                  MaterialPageRoute(builder: (_) => const LoginPage()),
                  (route) => false,
                ),
              ),
              const SizedBox(height: 14),
              Center(
                child: Text('BLA Policy Companion · v1.0.0',
                    style: AppType.caption
                        .copyWith(color: AppColors.textSubtle)),
              ),
              const SizedBox(height: 8),
            ],
          ),
        ),
      ],
    );
  }
}

class _MenuGroup extends StatelessWidget {
  final List<_MenuItem> items;
  const _MenuGroup({required this.items});
  @override
  Widget build(BuildContext context) {
    return AppCard(
      padding: const EdgeInsets.symmetric(horizontal: 8),
      child: Column(
        children: [
          for (var i = 0; i < items.length; i++) ...[
            _NavRow(icon: items[i].icon, label: items[i].label),
            if (i != items.length - 1) const Divider(height: 1),
          ],
        ],
      ),
    );
  }
}

class _MenuItem {
  final AppIconData icon;
  final String label;
  const _MenuItem(this.icon, this.label);
}

/// ปุ่มออกจากระบบ — สไตล์ danger (พื้นแดงอ่อน ตัวอักษร/ไอคอนสีแดง)
class _LogoutButton extends StatelessWidget {
  final VoidCallback onTap;
  const _LogoutButton({required this.onTap});

  @override
  Widget build(BuildContext context) {
    return Material(
      color: AppColors.dangerBg,
      borderRadius: BorderRadius.circular(AppSpacing.rMd),
      child: InkWell(
        borderRadius: BorderRadius.circular(AppSpacing.rMd),
        onTap: onTap,
        child: Padding(
          padding: const EdgeInsets.symmetric(vertical: 15),
          child: Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const HugeIcon(icon: HugeIcons.strokeRoundedLogout01, size: 20, color: AppColors.dangerFg),
              const SizedBox(width: 8),
              Text('ออกจากระบบ',
                  style: AppType.bodyStrong
                      .copyWith(color: AppColors.dangerFg)),
            ],
          ),
        ),
      ),
    );
  }
}

class _NavRow extends StatelessWidget {
  final AppIconData icon;
  final String label;
  const _NavRow({required this.icon, required this.label});

  @override
  Widget build(BuildContext context) {
    return ListTile(
      contentPadding: const EdgeInsets.symmetric(horizontal: 8),
      leading: HugeIcon(icon: icon, color: AppColors.primary, size: 22),
      title: Text(label, style: AppType.bodyStrong),
      trailing:
          const HugeIcon(icon: HugeIcons.strokeRoundedArrowRight01, color: AppColors.textSubtle),
      onTap: () {},
    );
  }
}

class _ToggleRow extends StatelessWidget {
  final AppIconData icon;
  final String label;
  final bool value;
  final ValueChanged<bool> onChanged;
  const _ToggleRow(
      {required this.icon,
      required this.label,
      required this.value,
      required this.onChanged});

  @override
  Widget build(BuildContext context) {
    return ListTile(
      contentPadding: const EdgeInsets.symmetric(horizontal: 8),
      leading: HugeIcon(icon: icon, color: AppColors.primary, size: 22),
      title: Text(label, style: AppType.bodyStrong),
      trailing: Switch(
          value: value, onChanged: onChanged, activeColor: AppColors.primary),
    );
  }
}

class _SegmentRow extends StatelessWidget {
  final AppIconData icon;
  final String label;
  final bool isThai;
  final ValueChanged<bool> onChanged;
  const _SegmentRow(
      {required this.icon,
      required this.label,
      required this.isThai,
      required this.onChanged});

  @override
  Widget build(BuildContext context) {
    return ListTile(
      contentPadding: const EdgeInsets.symmetric(horizontal: 8),
      leading: HugeIcon(icon: icon, color: AppColors.primary, size: 22),
      title: Text(label, style: AppType.bodyStrong),
      trailing: Container(
        decoration: BoxDecoration(
            color: AppColors.background,
            borderRadius: BorderRadius.circular(AppSpacing.rPill)),
        padding: const EdgeInsets.all(3),
        child: Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            _seg('ไทย', isThai, () => onChanged(true)),
            _seg('EN', !isThai, () => onChanged(false)),
          ],
        ),
      ),
    );
  }

  Widget _seg(String t, bool on, VoidCallback onTap) {
    return GestureDetector(
      onTap: onTap,
      child: Container(
        padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 5),
        decoration: BoxDecoration(
            color: on ? AppColors.primary : Colors.transparent,
            borderRadius: BorderRadius.circular(AppSpacing.rPill)),
        child: Text(t,
            style: AppType.tiny.copyWith(
                color: on ? Colors.white : AppColors.textMedium,
                fontWeight: FontWeight.w600)),
      ),
    );
  }
}
```

### B9. ไฟล์ที่ build_runner สร้างให้ (.g.dart) — ห้ามแก้ด้วยมือ

เปิดดูเพื่อ "เข้าใจว่าเบื้องหลัง `@riverpod` คืออะไร" — จะเห็นว่า annotation ถูกแปลงเป็น provider แบบเขียนมือ (AutoDispose มาให้อัตโนมัติ) พร้อม hash สำหรับ hot-reload

```dart
// lib/features/policy/data/policy_repository.g.dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'policy_repository.dart';

// **************************************************************************
// RiverpodGenerator
// **************************************************************************

String _$policyRepositoryHash() => r'be10faec6ead403edb4efb836015ed25fd6149a6';

/// See also [policyRepository].
@ProviderFor(policyRepository)
final policyRepositoryProvider = AutoDisposeProvider<PolicyRepository>.internal(
  policyRepository,
  name: r'policyRepositoryProvider',
  debugGetCreateSourceHash: const bool.fromEnvironment('dart.vm.product')
      ? null
      : _$policyRepositoryHash,
  dependencies: null,
  allTransitiveDependencies: null,
);

@Deprecated('Will be removed in 3.0. Use Ref instead')
// ignore: unused_element
typedef PolicyRepositoryRef = AutoDisposeProviderRef<PolicyRepository>;
// ignore_for_file: type=lint
// ignore_for_file: subtype_of_sealed_class, invalid_use_of_internal_member, invalid_use_of_visible_for_testing_member, deprecated_member_use_from_same_package
```

```dart
// lib/features/policy/presentation/controllers/policy_list_controller.g.dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'policy_list_controller.dart';

// **************************************************************************
// RiverpodGenerator
// **************************************************************************

String _$policyListHash() => r'c388e2cc8b43c8b182d027614f3cec580daee9a4';

/// See also [policyList].
@ProviderFor(policyList)
final policyListProvider = AutoDisposeFutureProvider<List<Policy>>.internal(
  policyList,
  name: r'policyListProvider',
  debugGetCreateSourceHash:
      const bool.fromEnvironment('dart.vm.product') ? null : _$policyListHash,
  dependencies: null,
  allTransitiveDependencies: null,
);

@Deprecated('Will be removed in 3.0. Use Ref instead')
// ignore: unused_element
typedef PolicyListRef = AutoDisposeFutureProviderRef<List<Policy>>;
String _$filteredPolicyListHash() =>
    r'45913841383c075e3d833c33d4a9e209e67298f8';

/// See also [filteredPolicyList].
@ProviderFor(filteredPolicyList)
final filteredPolicyListProvider = AutoDisposeProvider<List<Policy>>.internal(
  filteredPolicyList,
  name: r'filteredPolicyListProvider',
  debugGetCreateSourceHash: const bool.fromEnvironment('dart.vm.product')
      ? null
      : _$filteredPolicyListHash,
  dependencies: null,
  allTransitiveDependencies: null,
);

@Deprecated('Will be removed in 3.0. Use Ref instead')
// ignore: unused_element
typedef FilteredPolicyListRef = AutoDisposeProviderRef<List<Policy>>;
String _$policySearchHash() => r'8951a491cf7e70c7cc595667ac2c5ec964f941e1';

/// See also [PolicySearch].
@ProviderFor(PolicySearch)
final policySearchProvider =
    AutoDisposeNotifierProvider<PolicySearch, String>.internal(
  PolicySearch.new,
  name: r'policySearchProvider',
  debugGetCreateSourceHash:
      const bool.fromEnvironment('dart.vm.product') ? null : _$policySearchHash,
  dependencies: null,
  allTransitiveDependencies: null,
);

typedef _$PolicySearch = AutoDisposeNotifier<String>;
String _$policyStatusFilterHash() =>
    r'9b2f426a7a3a87f5b5d077f53d70c9c016d31ec3';

/// See also [PolicyStatusFilter].
@ProviderFor(PolicyStatusFilter)
final policyStatusFilterProvider =
    AutoDisposeNotifierProvider<PolicyStatusFilter, PolicyStatus?>.internal(
  PolicyStatusFilter.new,
  name: r'policyStatusFilterProvider',
  debugGetCreateSourceHash: const bool.fromEnvironment('dart.vm.product')
      ? null
      : _$policyStatusFilterHash,
  dependencies: null,
  allTransitiveDependencies: null,
);

typedef _$PolicyStatusFilter = AutoDisposeNotifier<PolicyStatus?>;
// ignore_for_file: type=lint
// ignore_for_file: subtype_of_sealed_class, invalid_use_of_internal_member, invalid_use_of_visible_for_testing_member, deprecated_member_use_from_same_package
```

**สิ่งที่ generated code บอกเรา:**

1. `@riverpod` ฟังก์ชันธรรมดา → `AutoDisposeProvider` · ฟังก์ชัน `Future<...>` → `AutoDisposeFutureProvider` · คลาส `extends _$X` → `AutoDisposeNotifierProvider` — ตรงตามตาราง Module 3 ทุกช่อง
2. **AutoDispose เป็นค่าเริ่มต้น** ของ Code Generation: ไม่มีใคร watch แล้ว state ถูกทิ้งอัตโนมัติ (ประหยัดหน่วยความจำ)
3. คลาสแม่ `_$PolicySearch` ที่เรา extends คือ typedef ของ `AutoDisposeNotifier<String>` ที่ generator สร้าง — นี่คือเหตุผลที่ห้ามลืม `part '...g.dart';`
4. typedef `PolicyRepositoryRef` ถูกติดป้าย `@Deprecated('Will be removed in 3.0. Use Ref instead')` — signal จากทีม Riverpod ว่าสาย 3.x จะใช้ `Ref` ตรง ๆ (ความรู้ไว้เตรียม migrate)

### B10. `test/widget_test.dart` — จุดสังเกตสำคัญ (การบ้านคิดต่อ)

ไฟล์เทสต์ยังเป็น **template เดิมจาก `flutter create`** ที่ทดสอบแอป counter — ไม่ตรงกับแอปเราแล้ว (รันแล้ว **fail** เพราะไม่มี `Icons.add`/ตัวเลข 0 ในแอป และ `BlaApp` ต้องถูกครอบด้วย `ProviderScope` ตอน pump):

```dart
// test/widget_test.dart (ยังเป็น template เดิม — จะแก้จริงจังใน Day 3)
// This is a basic Flutter widget test.
//
// To perform an interaction with a widget in your test, use the WidgetTester
// utility in the flutter_test package. For example, you can send tap and scroll
// gestures. You can also use WidgetTester to find child widgets in the widget
// tree, read text, and verify that the values of widget properties are correct.

import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

import 'package:bla_policy_companion/main.dart';

void main() {
  testWidgets('Counter increments smoke test', (WidgetTester tester) async {
    // Build our app and trigger a frame.
    await tester.pumpWidget(const BlaApp());

    // Verify that our counter starts at 0.
    expect(find.text('0'), findsOneWidget);
    expect(find.text('1'), findsNothing);

    // Tap the '+' icon and trigger a frame.
    await tester.tap(find.byIcon(Icons.add));
    await tester.pump();

    // Verify that our counter has incremented.
    expect(find.text('0'), findsNothing);
    expect(find.text('1'), findsOneWidget);
  });
}
```

> 🏠 **การบ้านคิดต่อ (เฉลยใน Day 3):** ทำไมเทสต์นี้จะพังทันทีที่รัน? อย่างน้อย 2 เหตุผล — (1) `pumpWidget(const BlaApp())` ไม่มี `ProviderScope` ครอบ → widget ที่ใช้ `ref` จะ throw (2) แอปเราไม่มีหน้า counter แล้ว · Day 3 เราจะเขียน Widget Test + Unit Test ของ provider จริง พร้อมเทคนิค override `policyRepositoryProvider` ด้วย mock

### B11. สรุปภาพรวม Part B — สามกลไกที่ "ร้อย" แอปทั้งแอปเข้าด้วยกัน

1. **Design System (core/)** — ทุกหน้าประกอบจากโทเคน + วิดเจ็ตกลางชุดเดียว: `AppColors`/`AppSpacing`/`AppType` → `AppTheme` → `MaterialApp` และ `AppCard`/`AppButton`/`GradientHeader`/`StatusBadge`/`AppTextField` ถูก reuse ทุกหน้า — แก้ดีไซน์ที่ core แล้วเปลี่ยนทั้งแอป
2. **Riverpod Provider** — ข้อมูลไหลทางเดียว: `PolicyRepository` (Data) → `policyListProvider` (async) → `filteredPolicyListProvider` (คำนวณรวม search+filter) → UI `watch` · ส่วน `mainTabProvider` เป็นตัวกลางให้หน้า Home สั่งสลับแท็บของ MainShell ได้โดยไม่รู้จักกันโดยตรง
3. **Navigator** — เส้นทางหลัก Splash → Login → MainShell เป็น `pushReplacement` (ย้อนกลับไม่ได้), หน้า detail/auth ย่อยเป็น `push` (ย้อนกลับได้), logout เป็น `pushAndRemoveUntil` (ล้าง stack) — Day 3 ทั้งหมดนี้จะถูกแทนที่ด้วย GoRouter + Route Guard

---

---

## 📁 โครงสร้างไฟล์สรุปวันที่ 1 (ครบทุกไฟล์ตาม branch `day1`)

```
bla_policy_app/  (ชื่อแพ็กเกจ: bla_policy_companion)
├── pubspec.yaml                      ← riverpod 2.6.x + code gen + hugeicons + google_fonts
│                                       + dependency_overrides (analyzer_plugin) + assets
├── assets/images/                    ← bla_icon.png, bla_logo_full.png, bla_avatar.png
├── lib/
│   ├── main.dart                     ← ProviderScope + MaterialApp(AppTheme.light)
│   │                                   + home: SplashPage + status bar scrim
│   ├── core/
│   │   ├── theme/
│   │   │   ├── app_colors.dart        ← โทเคนสี + gradient
│   │   │   ├── app_spacing.dart       ← โทเคนระยะห่าง/มุมโค้ง
│   │   │   ├── app_typography.dart    ← สไตล์ตัวอักษร (Inter + Anuphan)
│   │   │   └── app_theme.dart         ← ThemeData กลาง
│   │   ├── icons/
│   │   │   └── app_icons.dart         ← wrapper Hugeicons + AppIconData
│   │   └── widgets/
│   │       ├── app_button.dart        ← ปุ่ม primary/secondary/social
│   │       ├── app_card.dart          ← การ์ดขาว เงานุ่ม
│   │       ├── app_logo.dart          ← โลโก้ BLA
│   │       ├── app_text_field.dart    ← ช่องกรอก + ซ่อนรหัสผ่าน
│   │       ├── gradient_header.dart   ← หัว gradient + CircleHeaderButton
│   │       ├── info_hint.dart         ← กล่องคำใบ้
│   │       ├── section_label.dart     ← หัวข้อ section
│   │       └── status_badge.dart      ← ป้ายสถานะ 5 โทน
│   └── features/
│       ├── app/
│       │   └── main_shell.dart        ← Bottom Nav 5 แท็บ + mainTabProvider
│       ├── auth/presentation/pages/
│       │   ├── splash_page.dart       ← จุดเริ่ม (home ของ MaterialApp)
│       │   ├── login_page.dart
│       │   ├── register_page.dart
│       │   ├── forgot_password_page.dart
│       │   └── reset_password_page.dart
│       ├── home/presentation/pages/
│       │   └── home_page.dart         ← Dashboard + ปุ่มลัดสลับแท็บ
│       ├── policy/
│       │   ├── domain/
│       │   │   └── policy.dart                    ← โมเดล Entity + extension label
│       │   ├── data/
│       │   │   ├── policy_repository.dart         ← Repository (Mock)
│       │   │   └── policy_repository.g.dart       ← (build_runner สร้าง)
│       │   └── presentation/
│       │       ├── controllers/
│       │       │   ├── policy_list_controller.dart     ← 4 provider (search/filter/list/filtered)
│       │       │   └── policy_list_controller.g.dart   ← (build_runner สร้าง)
│       │       └── pages/
│       │           ├── policy_list_page.dart      ← รายการ + ค้นหา + กรอง
│       │           └── policy_detail_page.dart    ← รายละเอียด (รับ Policy)
│       ├── claim/presentation/
│       │   ├── pages/claims_page.dart             ← สรุป + ประวัติสินไหม
│       │   └── widgets/file_claim_sheet.dart      ← Bottom Sheet (ใช้ร่วม 3 ที่)
│       ├── payment/presentation/pages/
│       │   └── payments_page.dart                 ← บิล + ช่องทาง + ประวัติ
│       └── profile/presentation/pages/
│           └── profile_page.dart                  ← โปรไฟล์ + ตั้งค่า + logout
└── test/
    └── widget_test.dart               ← ยังเป็น template เดิม (การบ้าน Day 3)
```

---

## 📝 สรุปประจำวันที่ 1

| หัวข้อ                     | สิ่งที่ทำได้แล้ว                                                             |
| -------------------------- | ---------------------------------------------------------------------------- |
| Module 1 — Architecture    | เข้าใจ Clean Architecture และเลือก Feature-first เป็นโครงสร้างหลัก           |
| ★ Module 2 — Riverpod      | ติดตั้ง package, ครอบ ProviderScope และเปิด Code Generation                  |
| Module 3 — Core Providers  | ใช้ Provider, StateProvider, FutureProvider และเข้าใจ Notifier/AsyncNotifier |
| Module 4 — Consuming State | ใช้ ConsumerWidget และแยก watch/read/listen ได้ถูกต้อง                       |
| ★ Workshop Day 1           | วาง Feature-first + จัดการ State หน้ารายการกรมธรรม์ด้วย Riverpod ครบวงจร     |
| Part B — แอปเต็ม           | อ่านโค้ดจริงครบทุกไฟล์: MainShell, Auth 5 หน้า, Home/Claims/Payments/Profile, .g.dart และเข้าใจการเชื่อมโยงทั้งแอป |

### ✅ ตรวจสอบความพร้อมก่อนวันพรุ่งนี้

> ให้แน่ใจว่า:
>
> - รัน `flutter run` แล้วเห็นหน้ารายการกรมธรรม์ทำงานได้ (ค้นหา + กรองสถานะได้)
> - มีไฟล์ `.g.dart` ถูกสร้างครบจาก `build_runner` โดยไม่มี error สีแดง
> - เข้าใจความต่างของ `ref.watch` (ฟัง+rebuild) กับ `ref.read` (เรียกครั้งเดียว)
> - เข้าใจว่าทำไม Repository ถึงถูกฉีดผ่าน Provider (เตรียมเปลี่ยนเป็น API จริงวันที่ 2)

---

**💡 คำคมประจำวัน:**

> "สถาปัตยกรรมที่ดีไม่ได้วัดกันที่จำนวนไฟล์หรือ pattern ที่ใช้ แต่วัดกันที่ว่า — เมื่อความต้องการเปลี่ยน คุณต้องแก้โค้ดน้อยที่สุดและมั่นใจที่สุด"

---

_เอกสารจัดทำโดย: อาจารย์สามิตร โกยม | IT Genius Engineering Co., Ltd._
_สำหรับการอบรม: บริษัท กรุงเทพประกันชีวิต จำกัด (มหาชน)_
_หลักสูตร Advanced Flutter 2026 (Scalable Architecture & Riverpod) — วันที่ 1 จาก 3_
