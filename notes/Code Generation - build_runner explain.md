# ไฟล์ `.g.dart` และ `.freezed.dart` มาจากไหน? — อธิบายอย่างง่าย

> เอกสารประกอบการสอน · Advanced Flutter 2026 — Scalable Architecture & Riverpod

## หัวใจของเรื่อง: Code Generation

แทนที่เราจะนั่งพิมพ์โค้ดซ้ำ ๆ ที่น่าเบื่อเอง เราเขียนแค่ **"แบบย่อ"** พร้อมติด
**ป้ายกำกับ (annotation)** ไว้ แล้วให้เครื่องมือ **"ปั๊ม"** โค้ดส่วนที่เหลือออกมาให้อัตโนมัติ

> 💡 **อุปมา:** เราเป็น "คนออกแบบ" แล้วมี **ผู้ช่วยหุ่นยนต์** คอยเขียนงานจำเจแทน
> เราแค่บอกว่า "ช่วยสร้างของพวกนี้ให้หน่อย" ด้วยการติดป้าย `@freezed`, `@riverpod`, `@JsonSerializable`

---

## กลไก 3 ส่วนที่ทำงานร่วมกัน

### 1) ป้ายกำกับ (annotation) + คำว่า `part`
ในไฟล์ของเราเขียนสองอย่างคู่กันเสมอ:
- **ติดป้าย** บอกว่าอยากให้สร้างอะไร เช่น `@freezed`, `@riverpod`
- **ประกาศ `part`** บอกว่า "อีกครึ่งของไฟล์ฉันอยู่ในไฟล์ชื่อนี้นะ"

```dart
part 'policy.freezed.dart';
part 'policy.g.dart';
```

คำว่า **`part` คือ "กาว"** ที่เชื่อมไฟล์ที่ถูกปั๊มออกมา กับไฟล์ของเรา ให้กลายเป็น
**คลาสเดียวกัน** จึงเข้าถึงของภายในที่ขึ้นต้นด้วย `_$` ได้ (เช่น `_$Policy`, `_$PolicyFromJson`)

### 2) `build_runner` คือ "โรงงาน"
เมื่อรันคำสั่งนี้ build_runner จะเดินสแกนทุกไฟล์ เจอป้ายกำกับตรงไหน
ก็เรียก "ช่างเฉพาะทาง (builder)" มาปั๊มโค้ดออกมาเป็นไฟล์ใหม่:

```bash
dart run build_runner build --delete-conflicting-outputs
```

- `build` = สั่งสร้างครั้งเดียว
- `--delete-conflicting-outputs` = ของเก่าทับได้เลย ไม่ต้องถาม
- โหมดเฝ้าดู (สร้างใหม่อัตโนมัติทุกครั้งที่เซฟ): `dart run build_runner watch`

### 3) ช่างแต่ละคนสร้างคนละอย่าง (นามสกุลไฟล์บอกที่มา)

| นามสกุลไฟล์ | ช่าง (builder) | ปั๊มอะไรให้ |
|-------------|----------------|-------------|
| `.freezed.dart` | **freezed** | งานจำเจของ immutable model: `copyWith()`, `==`, `hashCode`, `toString()` |
| `.g.dart` | **json_serializable** | `fromJson` / `toJson` (เช่น `_$PolicyFromJson`) |
| `.g.dart` | **riverpod_generator** | ตัว Provider จาก `@riverpod` (เช่น `policyListProvider`) |

> `g` ย่อมาจาก **generated** — ไฟล์ `.g.dart` หนึ่งไฟล์อาจมีทั้งส่วนของ json_serializable และ riverpod รวมกันได้

---

## ตัวอย่างจริงในโปรเจกต์

ไฟล์ `policy.dart` ที่เราเขียนสั้น ๆ ~10 บรรทัด:

```dart
@freezed
class Policy with _$Policy {
  const factory Policy({
    required String id,
    required String planName,
    required double premium,
    required PolicyStatus status,
  }) = _Policy;

  factory Policy.fromJson(Map<String, dynamic> json) => _$PolicyFromJson(json);
}
```

พอรัน `build_runner` จะได้เพื่อนมา **2 ไฟล์อัตโนมัติ**:
- `policy.freezed.dart` → โค้ด `copyWith` / `==` / `hashCode` หลายร้อยบรรทัด
- `policy.g.dart` → โค้ด `fromJson` / `toJson`

**เราเขียนเอง ~10 บรรทัด แต่ได้โค้ดจริงหลายร้อยบรรทัด**

---

## เจาะลึก: `policy.dart` ไม่ได้เรียก `.g.dart` โดยตรง

จุดที่คนมักเข้าใจผิด คือคิดว่า `policy.dart` เรียก `.g.dart` เอง แต่จริง ๆ แล้ว
**`policy.dart` คุยกับ `.freezed.dart` เท่านั้น** ส่วน `.freezed.dart` ต่างหากที่ไปเรียก `.g.dart` อีกที

**หลักฐานจากไฟล์จริง** — บรรทัด 24 ของ `policy.dart` เรียก `_$PolicyFromJson(json)`
ซึ่งฟังก์ชันนี้อยู่ใน **`policy.freezed.dart`** (ไม่ใช่ `.g.dart`):

```dart
// policy.freezed.dart
Policy _$PolicyFromJson(Map<String, dynamic> json) {
  return _Policy.fromJson(json);       // ← freezed เป็นคนเรียกต่อ
}
```

แล้ว `_Policy.fromJson` (ก็อยู่ใน freezed.dart) ถึงจะไปเรียก `_$$PolicyImplFromJson`
ใน **`policy.g.dart`** — ตัวที่อ่าน `json['id']`, `json['planName']` จริง ๆ
(สังเกตชื่อมี `Impl` และ `$$`)

### ลูกโซ่การเรียกจริง (fromJson)
```
policy.dart
   Policy.fromJson  →  _$PolicyFromJson(json)
                              │  (นิยามอยู่ใน policy.freezed.dart)
                              ▼
policy.freezed.dart
   _$PolicyFromJson  →  _Policy.fromJson(json)
                              │  (นิยามอยู่ใน policy.g.dart)
                              ▼
policy.g.dart
   _$$PolicyImplFromJson(json)  →  อ่าน json['id'], json['planName'] ... จริง
```

### สรุปความสัมพันธ์ของ 3 ไฟล์
| จาก | ถึง | ผ่าน symbol |
|-----|-----|-------------|
| `policy.dart` | `policy.freezed.dart` | `_$Policy`, `_Policy`, `_$PolicyFromJson` |
| `policy.freezed.dart` | `policy.g.dart` | `_$$PolicyImplFromJson`, `_$$PolicyImplToJson`, `_$PolicyStatusEnumMap` |
| `policy.dart` | `policy.g.dart` | **ไม่มี — ไม่เรียกตรง ๆ เลย** |

> 💡 **เทียบง่าย ๆ:** `freezed` = **หัวหน้างาน/ตัวกลาง** ที่เราสั่งงานโดยตรง ·
> `.g.dart` (json) = **ลูกน้องเฉพาะทาง** ที่หัวหน้าเรียกใช้อีกที — เราไม่ได้สั่งลูกน้องตรง ๆ

> ฝั่ง `toJson` ก็ทิศเดียวกัน: `policy.dart` ไม่ได้เขียนเรียก `toJson` เลย แต่เมธอด
> `policy.toJson()` คือของใน `_Policy` (freezed.dart) ที่วิ่งไปเรียก `_$$PolicyImplToJson` ใน `.g.dart`

> ⚠️ หมายเหตุ: ทั้ง 3 ไฟล์ถูก `part` รวมเป็น **library เดียวกัน** จึงแชร์ namespace เดียวกัน
> (ไม่ใช่ import ข้ามไฟล์) — แต่ถ้าดูที่ "โค้ดที่เรียกถูกเขียนไว้ในไฟล์ไหน" ลูกโซ่ข้างบนคือคำตอบ

---

## ภาพรวมการทำงาน (Flow)

```
   ไฟล์เราเขียน                build_runner              ไฟล์ที่ถูกปั๊ม
 (annotated .dart)  ──scan──▶  (อ่านป้ายกำกับ)  ──emit──▶  .freezed.dart
   @freezed                    เรียก builder                .g.dart
   @riverpod                   ที่เกี่ยวข้อง                    │
   part '...'  ◀──────────── "กาว" เชื่อมเป็นคลาสเดียว ─────────┘
                                     │
                                     ▼
                              Dart compiler รวมทั้งหมด → แอปทำงาน
```

---

## ทำไมถึงคุ้ม

- **ลดโค้ด** — เขียนน้อย ได้เยอะ
- **ลด bug** — คนพิมพ์ผิดได้ เครื่องปั๊มไม่ผิด
- **compile-safe** — พิมพ์ชื่อ provider ผิด compiler ฟ้องทันที ไม่ต้องรอ runtime

---

## กฎที่ต้องจำ 3 ข้อ

1. **อย่าแก้ไฟล์ `.g.dart` / `.freezed.dart` ด้วยมือ** — แก้ไปเดี๋ยวก็ถูกปั๊มทับ ให้แก้ที่ไฟล์ต้นฉบับเสมอ
2. **แก้โค้ดที่มีป้ายกำกับเมื่อไร ต้องรัน `build_runner` ใหม่** (หรือเปิดโหมด `watch` ค้างไว้)
3. **ไม่ต้อง commit ขึ้น git** — มันสร้างใหม่ได้เสมอ ปกติตั้ง `.gitignore` ให้ข้าม `*.g.dart` / `*.freezed.dart`

> ⚠️ นี่คือเหตุผลว่าทำไม **พอ `git clone` โปรเจกต์มาใหม่ ต้องรัน `build_runner` ก่อน**
> ถึงจะ `flutter run` ได้ — ไม่งั้นจะ error หา `_$...` ไม่เจอ

---

## สรุปสั้น ๆ

> **เราเขียนพิมพ์เขียว → `build_runner` คือโรงงานที่อ่านป้ายกำกับแล้วปั๊มโค้ดจำเจให้ → `part` คือกาวเชื่อมสองไฟล์เป็นคลาสเดียว**

| เราทำ | เครื่องทำให้ |
|-------|--------------|
| ติดป้าย `@freezed` / `@riverpod` / `@JsonSerializable` | ปั๊มโค้ด boilerplate |
| ประกาศ `part '...'` | เชื่อมไฟล์เป็นคลาสเดียว |
| รัน `dart run build_runner build` | สร้าง `.g.dart` / `.freezed.dart` |
