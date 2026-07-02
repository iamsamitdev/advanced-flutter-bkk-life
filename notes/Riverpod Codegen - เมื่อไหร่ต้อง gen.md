# Riverpod + Codegen — จะรู้ได้อย่างไรว่าเรื่องไหนต้อง gen code

**คำถาม:** เวลาเขียน Flutter ด้วย Riverpod จะรู้ได้อย่างไรว่าเรื่องนี้จำเป็นต้องใช้ `@riverpod`, `@freezed`, `@JsonSerializable` (แล้วต้องรัน build_runner สร้างไฟล์ `.g.dart` / `.freezed.dart`)

**คำตอบสั้นที่สุด (ประโยคเดียวให้จำ):**

> **"Widget ไม่เคย gen — ที่ต้อง gen คือของที่ประกาศ 3 อย่าง: ของกลาง (provider), ข้อมูล (model), และข้อมูลที่ข้าม JSON — และมันคือทางเลือก ไม่ใช่ข้อบังคับ"**

---

## 1. หลักใหญ่ข้อเดียว: codegen เกิดกับ "ของที่ประกาศ" ไม่เกิดกับ "UI"

เวลาเขียน Flutter + Riverpod โค้ดแบ่งได้เป็น 2 ฝั่งเสมอ:

| ฝั่ง | ตัวอย่าง | ต้อง gen? |
| --- | --- | --- |
| **ฝั่งเรียกใช้ (UI/Widget)** | `ConsumerWidget`, `ref.watch`, `ref.read`, หน้าจอทุกหน้า | ❌ ไม่มีวันต้อง gen ไม่ว่าแอปจะซับซ้อนแค่ไหน |
| **ฝั่งประกาศของ (declare)** | provider, model, ตัวแปลง JSON | ✅ คือ "ผู้สมัคร" ของ codegen (มี 3 เรื่องเท่านั้น) |

---

## 2. สามเรื่องที่เป็นผู้สมัคร codegen — ดูจาก "สิ่งที่กำลังจะสร้าง"

### 2.1 กำลังสร้าง "ของกลางให้ทั้งแอปใช้ร่วมกัน" → `@riverpod`

**สัญญาณ:** ความคิดในหัวว่า *"อยากให้หน้าอื่น/widget อื่นเข้าถึงของชิ้นนี้ได้"*

- object ที่ใช้ร่วม เช่น `Dio`, Repository
- ค่า state เช่น คำค้นหา, ตัวกรอง, แท็บที่เลือก
- ข้อมูล async เช่น รายการกรมธรรม์จาก API

ทั้งหมดนี้คือ **provider** — เมื่อประกาศ provider ด้วย `@riverpod` ก็ต้อง gen (`riverpod_generator` → ไฟล์ `.g.dart` ที่มีตัวแปร `xxxProvider` ให้)

### 2.2 กำลังสร้าง "คลาสที่เป็นข้อมูล" (data class) → `@freezed`

**สัญญาณ:** คลาสที่มีแต่ field ไม่มีพฤติกรรมซับซ้อน เช่น `Policy`, `AuthUser`

ของพวกนี้ควรเป็น immutable, เทียบค่า `==` ได้, มี `copyWith` — ซึ่งเขียนมือแล้วทั้งยาวทั้งพลาดง่าย freezed จึงคุ้มเมื่อ field เยอะหรือแก้บ่อย (`freezed` → ไฟล์ `.freezed.dart`)

### 2.3 ข้อมูลนั้นต้อง "ข้ามพรมแดนแอป" (JSON จาก API/ไฟล์/DB) → `@JsonSerializable`

**สัญญาณ:** คำว่า *fromJson / toJson*

เมื่อข้อมูลเข้า-ออกแอปเป็น JSON ต้องมีตัวแปลง จะเขียนมือหรือให้ gen ก็ได้ — ถ้าใช้ freezed อยู่แล้ว แค่เติม `factory .fromJson` + `part '.g.dart'` มันต่อ `json_serializable` ให้เลย

---

## 3. จุดสำคัญที่มักเข้าใจผิด: ทั้ง 3 ตัว "ไม่บังคับ" สักตัว

Codegen **ไม่ใช่ข้อกำหนดทางเทคนิค** — เป็นทางเลือกเพื่อลด boilerplate ทุกเรื่องเขียนมือได้หมด ในโปรเจกต์ BLA Policy Companion เราจงใจทำให้เห็นครบทั้งสองแบบ:

| งาน | แบบเขียนมือ (ไม่ gen) | แบบ codegen |
| --- | --- | --- |
| Provider | `mainTabProvider = StateProvider<int>(...)` (Day 1) | `@riverpod class PolicyList ...` |
| Model | `ClaimRecord`, `Payment` | `Policy`, `AuthUser` (`@freezed`) |
| JSON | `ClaimRecord.fromJson` เขียนเอง | `Policy.fromJson` ผ่าน `_$PolicyFromJson` |

### เกณฑ์เลือกที่แนะนำ (rule of thumb)

- **Provider** → ใช้ `@riverpod` เป็นค่าเริ่มต้นเสมอ (ได้ compile-safe + autoDispose ฟรี)
- **Model** → field ≤ 3–4 ตัวและไม่ต้องใช้ `copyWith` เขียนมือได้ / มากกว่านั้นหรือใช้ยาว ๆ ให้ `@freezed`
- **JSON** → โครงสร้างแบน ๆ เขียนมือได้ / ซ้อนหลายชั้นหรือมี enum ให้ gen

---

## 4. วิธีสังเกตจากไฟล์ที่มีอยู่แล้ว (มุมกลับ: อ่านโค้ดคนอื่น)

ถามตัวเอง 3 ข้อ — ถ้า "ใช่" แม้แต่ข้อเดียว = ไฟล์นี้ต้องผ่าน build_runner:

1. มีบรรทัด `part '....g.dart';` หรือ `part '....freezed.dart';` หรือไม่ (ตัวชี้ขาด)
2. มี annotation `@riverpod` / `@Riverpod(...)` / `@freezed` / `@JsonSerializable` หรือไม่
3. โค้ดอ้างถึง symbol ที่เราไม่ได้เขียนเอง เช่น `_$PolicyList`, `_$PolicyFromJson`, `xxxProvider` หรือไม่

**สัญญาณ error คลาสสิกว่า "ลืม gen":**

- `Target of URI hasn't been generated: '....g.dart'`
- `Undefined name 'dioClientProvider'`
- `The name '_$...' isn't defined`

---

## 5. ตารางสรุป: annotation ไหน ได้ไฟล์อะไร

| Annotation | Generator | ไฟล์ที่ได้ | สิ่งที่เครื่องเขียนให้ |
| --- | --- | --- | --- |
| `@riverpod` (ฟังก์ชัน/คลาส) | riverpod_generator | `.g.dart` | ตัวแปร `xxxProvider` + คลาสแม่ `_$Xxx` |
| `@freezed` | freezed | `.freezed.dart` | `copyWith`, `==`, `hashCode`, `toString` |
| `@freezed` + `factory .fromJson` + `part '.g.dart'` | json_serializable | `.g.dart` (เพิ่ม) | `fromJson` / `toJson` |
| `@JsonSerializable` | json_serializable | `.g.dart` | `fromJson` / `toJson` |
| — (ไม่มี annotation, ไม่มี part) | — | ไม่ต้อง gen | — |

> หมายเหตุ: `app_localizations*.dart` (i18n) เป็นข้อยกเว้น — มาจาก `flutter gen-l10n` ที่อ่านไฟล์ `.arb` ไม่เกี่ยวกับ build_runner

---

## 6. คำสั่งที่ใช้

```bash
# สร้างครั้งเดียว
dart run build_runner build --delete-conflicting-outputs

# เฝ้าดูไฟล์และสร้างใหม่อัตโนมัติเมื่อกด Save (แนะนำระหว่างพัฒนา)
dart run build_runner watch --delete-conflicting-outputs
```

ต้องรันใหม่เมื่อแก้อะไรที่กระทบ "หน้าตา" ของสิ่งที่ generator สร้าง เช่น สร้าง/ลบ/เปลี่ยนชื่อ provider, เปลี่ยนชนิดค่าที่คืน, เพิ่ม/ลบ field ในคลาส `@freezed`, เปลี่ยน `@riverpod` เป็น `@Riverpod(keepAlive: true)` — ถ้าเปิด `watch` ทิ้งไว้ก็ไม่ต้องคิดเรื่องนี้เลย

---

## อ้างอิง

- Riverpod — About code generation: <https://riverpod.dev/docs/concepts/about_code_generation>
- freezed (ดูหัวข้อ Motivation): <https://pub.dev/packages/freezed>
- json_serializable: <https://pub.dev/packages/json_serializable>
- build_runner: <https://dart.dev/tools/build_runner>
- ในโฟลเดอร์ notes: `Code Generation - build_runner explain.md`

---

*เอกสารประกอบหลักสูตร Advanced Flutter 2026 (Scalable Architecture & Riverpod) — อ.สามิตร โกยม | IT Genius Engineering Co., Ltd.*
