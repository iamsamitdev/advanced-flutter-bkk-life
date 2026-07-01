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

| เวลา | หัวข้อ |
|------|--------|
| 09:00–09:30 | ทดสอบก่อนเรียน (Pretest) + แนะนำหลักสูตรและโปรเจกต์ BLA Policy Companion |
| 09:30–10:30 | **Module 1** Introduction to Modern Flutter Architecture (2026) |
| 10:30–12:00 | **Module 2** Deep Dive into Riverpod (ติดตั้ง + ProviderScope + Code Gen) |
| 12:00–13:00 | พักกลางวัน |
| 13:00–14:15 | **Module 3** Understanding Core Providers |
| 14:15–15:00 | **Module 4** Consuming State (Consumer & ref) |
| 15:00–16:00 | **Workshop Day 1** วาง Feature-first + State หน้ารายการกรมธรรม์ |

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

| ชั้น (Layer) | หน้าที่ | ตัวอย่างในแอป BLA |
|-------------|---------|-------------------|
| **Presentation** | แสดงผล UI และจัดการ State | หน้า `PolicyListPage`, Provider ของ Riverpod |
| **Domain** | กฎทางธุรกิจ + โมเดลแก่นกลาง (Entity) | คลาส `Policy`, การคำนวณสถานะกรมธรรม์ |
| **Data** | ดึง/บันทึกข้อมูล (API, DB, Cache) | `PolicyRepository`, การเรียก Dio, แปลง JSON |

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

| ประเด็น | Layer-first | Feature-first (ที่เราจะใช้) |
|---------|-------------|------------------------------|
| หาโค้ดของฟีเจอร์เดียว | กระจายหลายโฟลเดอร์ | อยู่รวมที่เดียว ค้นง่าย |
| ทีมขนาดใหญ่ทำงานพร้อมกัน | ชนกันบ่อย (แก้โฟลเดอร์เดียวกัน) | แยกฟีเจอร์ ลดการชนกัน |
| ลบ/ย้ายฟีเจอร์ | ยุ่งยาก ต้องตามเก็บหลายที่ | ลบโฟลเดอร์เดียวจบ |
| เหมาะกับ | โปรเจกต์เล็ก/ตัวอย่าง | **แอประดับองค์กร** |

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

| คุณสมบัติ | Provider (เดิม) | Riverpod (ที่เราจะใช้) |
|-----------|-----------------|------------------------|
| ผูกกับ BuildContext | ใช่ | **ไม่** |
| ความปลอดภัยตอน compile | ต่ำ | **สูง (compile-safe)** |
| ทดสอบ (Unit Test) | ต้องมี Widget | **ทดสอบ Logic ตรง ๆ ได้** |
| Auto-dispose เมื่อไม่ใช้ | ทำเองยาก | **มีในตัว** |
| Code Generation | ไม่มี | **มี (`@riverpod`)** |

### 2.2 ขั้นตอนที่ 1 — ติดตั้ง Package

ในโปรเจกต์ Flutter ของเรา ติดตั้ง dependency ทั้งหมดที่จะใช้ตลอด 3 วันด้วยคำสั่งเดียว:

```bash
# Package หลักของ Riverpod + Code Generation
flutter pub add flutter_riverpod riverpod_annotation

# Package สำหรับสร้างโมเดล (ใช้หนักในวันที่ 2)
flutter pub add freezed_annotation json_annotation

# Dev dependencies: ตัวสร้างโค้ดและตัวตรวจ
flutter pub add dev:build_runner dev:riverpod_generator dev:freezed dev:json_serializable dev:custom_lint dev:riverpod_lint

# Package สำหรับ UI / Design System (ใช้ตั้งแต่วันแรก): ไอคอน Hugeicons + ฟอนต์ Google Fonts
flutter pub add hugeicons google_fonts
```

ตรวจสอบว่าใน `pubspec.yaml` มีรายการครบ:

```yaml
dependencies:
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0
  freezed_annotation: ^2.4.0
  json_annotation: ^4.9.0
  hugeicons: ^1.1.7        # ชุดไอคอน stroke-rounded (Design System)
  google_fonts: ^6.2.1     # ฟอนต์ Inter + Anuphan (ไทย)

dev_dependencies:
  build_runner: ^2.4.0
  riverpod_generator: ^2.4.0
  freezed: ^2.5.0
  json_serializable: ^6.8.0
  custom_lint: ^0.6.0
  riverpod_lint: ^2.3.0
```

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

> ⚠️ **ข้อควรระวังที่พบบ่อย:** ทุกไฟล์ที่ใช้ `@riverpod` ต้องมีบรรทัด `part 'ชื่อไฟล์.g.dart';` ด้านบนเสมอ มิฉะนั้น build_runner จะไม่สร้างโค้ดให้ และจะขึ้น error สีแดงว่าหา `_$...` ไม่เจอ

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

| ชนิด Provider | ใช้เมื่อ | ค่าที่คืน |
|---------------|---------|-----------|
| `Provider` | ค่าคงที่/ออบเจกต์ที่ไม่เปลี่ยน เช่น Repository, config | ค่าใด ๆ (อ่านอย่างเดียว) |
| `StateProvider` | State ง่าย ๆ ที่เปลี่ยนได้ เช่น ตัวเลข, ข้อความค้นหา | ค่าที่แก้ไขได้ตรง ๆ |
| `FutureProvider` | ดึงข้อมูลแบบ async ครั้งเดียว | `AsyncValue<T>` |
| `Notifier` (ใหม่) | State ที่ซับซ้อนมี Logic หลายเมธอด | คลาส State + เมธอด |
| `AsyncNotifier` (ใหม่) | State async ที่มีทั้งดึงและแก้ไขข้อมูล | `AsyncValue<T>` + เมธอด |

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

| เมธอด | ใช้เมื่อ | rebuild UI? | ใช้ที่ไหน |
|-------|---------|-------------|----------|
| `ref.watch()` | ต้องการ "ฟัง" ค่าและให้ UI อัปเดตตาม | ✅ ใช่ | ใน `build()` เท่านั้น |
| `ref.read()` | เรียกเมธอด/อ่านค่าครั้งเดียว ไม่ต้องฟัง | ❌ ไม่ | ใน callback (onPressed ฯลฯ) |
| `ref.listen()` | ทำ side-effect เมื่อค่าเปลี่ยน (เช่น แสดง SnackBar, นำทาง) | ❌ ไม่ | ใน `build()` |

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
├── main.dart                          ← ProviderScope + MaterialApp (ธีม)
├── core/                              ← โครงสร้างพื้นฐานใช้ร่วมทั้งแอป
│   ├── theme/
│   │   ├── app_colors.dart            ← โทเคนสี (Design System)
│   │   ├── app_spacing.dart           ← โทเคนระยะห่าง/มุมโค้ง
│   │   ├── app_typography.dart        ← สไตล์ตัวอักษร (Inter + Anuphan)
│   │   └── app_theme.dart             ← ประกอบ ThemeData กลาง
│   ├── icons/
│   │   └── app_icons.dart             ← ครอบ Hugeicons (wrapper + AppIconData)
│   └── widgets/                       ← วิดเจ็ตกลาง reuse ได้
│       ├── app_card.dart
│       ├── status_badge.dart
│       └── gradient_header.dart
└── features/
    └── policy/
        ├── domain/
        │   └── policy.dart            ← โมเดลแก่นกลาง (Entity)
        ├── data/
        │   └── policy_repository.dart ← แหล่งข้อมูล (Mock)
        └── presentation/
            ├── controllers/
            │   └── policy_list_controller.dart  ← Notifier จัดการ State
            └── pages/
                └── policy_list_page.dart        ← หน้าจอ UI (Design System)
```

> 🗂️ **หมายเหตุ:** โปรเจกต์เต็มใน branch `day1` มีครบทั้ง 5 แท็บ (หน้าแรก/กรมธรรม์/สินไหม/ชำระเบี้ย/โปรไฟล์) + หน้า Auth
> โดยทุกหน้าใช้ Design System และวิดเจ็ตกลางชุดเดียวกันนี้ · Workshop ช่วงนี้เราโฟกัสที่ฟีเจอร์ **policy** เป็นแกน แล้วต่อยอดหน้าอื่นด้วยแพตเทิร์นเดียวกัน

### ขั้นที่ 2 — สร้างโมเดล Policy (ชั้น Domain)

ในวันแรกเราเขียนโมเดลแบบมือก่อนเพื่อให้เข้าใจโครงสร้างชัด ๆ (วันที่ 2 จะอัปเกรดเป็น `freezed`)

```dart
// lib/features/policy/domain/policy.dart

// สถานะของกรมธรรม์
enum PolicyStatus { active, lapsed, pending }

class Policy {
  final String id;
  final String planName;     // ชื่อแผนประกัน
  final String policyNumber; // เลขที่กรมธรรม์
  final double premium;      // เบี้ยประกัน (บาท/ปี)
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

// 3) ดึงรายการกรมธรรม์ทั้งหมดจาก repository
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
  final policies = asyncPolicies.valueOrNull ?? <Policy>[];

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
abstract final class AppType {
  static final List<String> _thaiFallback = [GoogleFonts.anuphan().fontFamily!];

  static TextStyle _base(double size, FontWeight weight, {Color? color, double? height}) {
    return GoogleFonts.inter(
      fontSize: size,
      fontWeight: weight,
      color: color ?? AppColors.textDark,
      height: height,
    ).copyWith(fontFamilyFallback: _thaiFallback);
  }

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
library;

import 'package:flutter/widgets.dart';
import 'package:hugeicons/hugeicons.dart' as hi;

// ส่งออกรายชื่อไอคอน (HugeIcons.strokeRounded*) ให้ทั้งแอปใช้ได้
// แต่ซ่อน HugeIcon ตัวจริงของแพ็กเกจ เพื่อใช้ wrapper ของเราแทน
export 'package:hugeicons/hugeicons.dart' hide HugeIcon;

/// ชนิดข้อมูลไอคอน Hugeicons (ใช้แทน IconData เดิม)
typedef AppIconData = List<List<dynamic>>;

/// ค่าชดเชยเชิงทัศนภาพ ให้ hugeicon ดูสมดุลเทียบเท่า Material icon ขนาดเดียวกัน
const double kIconOpticalScale = 0.84;

/// ไอคอนของแอป — ห่อ `HugeIcon` ของแพ็กเกจ พร้อมชดเชยขนาดให้สมดุล
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
    final total = asyncPolicies.valueOrNull?.length ?? 0;

    final activePremium = (asyncPolicies.valueOrNull ?? [])
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
      // โปรเจกต์จริง (branch day1) เปิดหน้า PolicyDetailPage แทน — ในเวิร์กช็อปนี้ใช้ SnackBar ให้รันได้ทันที
      onTap: () => ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('เปิดกรมธรรม์ ${policy.policyNumber}')),
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

`PolicyListPage` คืน `Column` (ไม่มี `Scaffold` ในตัว) เพราะออกแบบให้ฝังใน "เปลือก" ของแอป (ในโปรเจกต์จริงคือ `MainShell` ที่มี Bottom Navigation) สำหรับเวิร์กช็อปนี้เราห่อด้วย `Scaffold` ใน `main.dart` เพื่อรันหน้าเดียวก่อน — และใช้ธีมจริง `AppTheme.light`

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import 'core/theme/app_theme.dart';
import 'features/policy/presentation/pages/policy_list_page.dart';

void main() {
  // ครอบทั้งแอปด้วย ProviderScope — หัวใจที่เก็บ state ของทุก provider
  runApp(const ProviderScope(child: BlaApp()));
}

class BlaApp extends StatelessWidget {
  const BlaApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'BLA Policy Companion',
      debugShowCheckedModeBanner: false,
      theme: AppTheme.light, // ← ใช้ Design System theme
      home: const Scaffold(body: PolicyListPage()),
    );
  }
}
```

รันด้วยคำสั่ง (อย่าลืมสั่ง build_runner ก่อนหากยังไม่ได้สร้างไฟล์ `.g.dart`):

```bash
dart run build_runner build --delete-conflicting-outputs
flutter run
```

> ✅ **ผลลัพธ์ที่คาดหวัง:** หน้าจอแสดงหัว gradient น้ำเงิน + รายการกรมธรรม์ 4 ใบเป็นการ์ดสไตล์ Design System พิมพ์ค้นหาแล้วรายการกรองทันที กดชิปสถานะแล้วกรองตามสถานะ แถบ "เบี้ยรวม (มีผลบังคับ)" อัปเดตตามข้อมูล และมี loading spinner ตอนเปิดครั้งแรก · ทั้งหมดนี้เกิดจากการ `ref.watch` หลาย provider ประกอบกัน โดย UI **ไม่มี Logic การกรองเลย** — Logic อยู่ในชั้น Presentation ที่แยกออกมาแล้ว

### 🧱 ต่อยอดเป็นแอปเต็ม — Bottom Navigation (main_shell)

โปรเจกต์จริงห่อ 5 หน้าไว้ในเปลือกเดียวที่มีแถบเมนูล่าง โดย Day 1 ใช้ `StateProvider` ง่าย ๆ เก็บ index ของแท็บ (Day 3 จะยกระดับเป็น GoRouter + StatefulShellRoute)

```dart
// lib/features/app/main_shell.dart (ตัวอย่างแกนหลัก)
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../core/icons/app_icons.dart';
import '../../core/theme/app_colors.dart';
import '../home/presentation/pages/home_page.dart';
import '../policy/presentation/pages/policy_list_page.dart';
// ... import อีก 3 หน้า (claims / payments / profile)

/// แท็บที่เลือกอยู่ของ Bottom Navigation (Day 1 ใช้ StateProvider ง่าย ๆ)
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

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final index = ref.watch(mainTabProvider);
    return Scaffold(
      // IndexedStack เก็บ state ของทุกแท็บไว้ (ไม่ rebuild เมื่อสลับ)
      body: IndexedStack(index: index, children: _pages),
      bottomNavigationBar: /* แถบเมนู custom — ดูโค้ดเต็มใน branch day1 */ null,
    );
  }
}
```

> 🗂️ โปรเจกต์เต็ม (branch `day1`) มีอีก 4 หน้า (`home_page`, `claims_page`, `payments_page`, `profile_page`) + หน้า Auth (`splash/login/register/...`) และวิดเจ็ตกลางเพิ่มเติม (`app_button`, `section_label`, `app_text_field`, `app_logo`, `info_hint`) ทุกไฟล์ใช้ Design System + Feature-first แบบเดียวกับที่สร้างในเวิร์กช็อปนี้ — เปิดดูได้จาก repository เพื่อศึกษาต่อ

### 🧩 ความท้าทายเสริม (ถ้าทำเสร็จก่อนเวลา)

- เพิ่มการเรียงลำดับ (sort) ตามเบี้ยประกันจากมากไปน้อย โดยทำเป็น provider แยก แล้ว `ref.watch` เพิ่มในหน้า
- สร้างหน้า `PolicyDetailPage` แล้วเปลี่ยน `onTap` ของ `_PolicyCard` จาก `SnackBar` เป็นเปิดหน้ารายละเอียด (ฝึกส่งข้อมูลข้ามหน้า)
- แยกวิดเจ็ตกลางเพิ่ม เช่น `AppButton` / `SectionLabel` แล้วนำไปใช้ซ้ำในหน้าอื่น (ฝึกคิดแบบ Design System)

---

## 📁 โครงสร้างไฟล์สรุปวันที่ 1

```
bla_policy_companion/                 ← flutter create --org com.itgenius bla_policy_companion
├── pubspec.yaml                      ← riverpod + code gen + hugeicons + google_fonts
├── lib/
│   ├── main.dart                     ← ProviderScope + MaterialApp(AppTheme.light)
│   ├── core/
│   │   ├── theme/
│   │   │   ├── app_colors.dart        ← โทเคนสี
│   │   │   ├── app_spacing.dart       ← โทเคนระยะห่าง/มุมโค้ง
│   │   │   ├── app_typography.dart    ← สไตล์ตัวอักษร (Inter + Anuphan)
│   │   │   └── app_theme.dart         ← ThemeData กลาง
│   │   ├── icons/
│   │   │   └── app_icons.dart         ← wrapper Hugeicons
│   │   └── widgets/
│   │       ├── app_card.dart
│   │       ├── status_badge.dart
│   │       └── gradient_header.dart
│   └── features/
│       └── policy/
│           ├── domain/
│           │   └── policy.dart                    ← โมเดล Entity
│           ├── data/
│           │   ├── policy_repository.dart         ← Repository (Mock)
│           │   └── policy_repository.g.dart       ← (build_runner สร้าง)
│           └── presentation/
│               ├── controllers/
│               │   ├── policy_list_controller.dart
│               │   └── policy_list_controller.g.dart  ← (build_runner สร้าง)
│               └── pages/
│                   └── policy_list_page.dart        ← UI (Design System)
```

> โปรเจกต์เต็ม (branch `day1`) มี `features/app/main_shell.dart` (Bottom Nav) + อีก 4 แท็บ + หน้า Auth และวิดเจ็ตกลางเพิ่มเติม โดยใช้แพตเทิร์นเดียวกันทั้งหมด

---

## 📝 สรุปประจำวันที่ 1

| หัวข้อ | สิ่งที่ทำได้แล้ว |
|--------|----------------|
| Module 1 — Architecture | เข้าใจ Clean Architecture และเลือก Feature-first เป็นโครงสร้างหลัก |
| ★ Module 2 — Riverpod | ติดตั้ง package, ครอบ ProviderScope และเปิด Code Generation |
| Module 3 — Core Providers | ใช้ Provider, StateProvider, FutureProvider และเข้าใจ Notifier/AsyncNotifier |
| Module 4 — Consuming State | ใช้ ConsumerWidget และแยก watch/read/listen ได้ถูกต้อง |
| ★ Workshop Day 1 | วาง Feature-first + จัดการ State หน้ารายการกรมธรรม์ด้วย Riverpod ครบวงจร |

### ✅ ตรวจสอบความพร้อมก่อนวันพรุ่งนี้

> ให้แน่ใจว่า:
> - รัน `flutter run` แล้วเห็นหน้ารายการกรมธรรม์ทำงานได้ (ค้นหา + กรองสถานะได้)
> - มีไฟล์ `.g.dart` ถูกสร้างครบจาก `build_runner` โดยไม่มี error สีแดง
> - เข้าใจความต่างของ `ref.watch` (ฟัง+rebuild) กับ `ref.read` (เรียกครั้งเดียว)
> - เข้าใจว่าทำไม Repository ถึงถูกฉีดผ่าน Provider (เตรียมเปลี่ยนเป็น API จริงวันที่ 2)

---

**💡 คำคมประจำวัน:**

> "สถาปัตยกรรมที่ดีไม่ได้วัดกันที่จำนวนไฟล์หรือ pattern ที่ใช้ แต่วัดกันที่ว่า — เมื่อความต้องการเปลี่ยน คุณต้องแก้โค้ดน้อยที่สุดและมั่นใจที่สุด"

---

*เอกสารจัดทำโดย: อาจารย์สามิตร โกยม | IT Genius Engineering Co., Ltd.*
*สำหรับการอบรม: บริษัท กรุงเทพประกันชีวิต จำกัด (มหาชน)*
*หลักสูตร Advanced Flutter 2026 (Scalable Architecture & Riverpod) — วันที่ 1 จาก 3*
