# Advanced Flutter 2026 — วันที่ 2: Advanced Data Handling & Dependency Injection

**หลักสูตรอบรมเชิงปฏิบัติการ: Advanced Flutter 2026 (Scalable Architecture & Riverpod)**
**จัดอบรมให้: บริษัท กรุงเทพประกันชีวิต จำกัด (มหาชน)**
**วันที่ 2: จัดการข้อมูล Asynchronous, Caching และ Dependency Injection**
วันที่: 3 กรกฎาคม 2569 | เวลา 09:00–16:00 น. | Onsite Workshop
ผู้สอน: อ.สามิตร โกยม

---

## 🎯 วัตถุประสงค์การเรียนรู้ประจำวัน

เมื่อจบการอบรมวันที่ 2 ผู้เรียนจะสามารถ:

1. จัดการสถานะ Loading / Data / Error อย่างสง่างามด้วย **`AsyncValue`** และเมธอด `.when()` / `.guard()` ได้
2. ใช้ **`AsyncNotifier`** สำหรับทั้งการดึงข้อมูล (Fetch) และการแก้ไขข้อมูล (Mutation) ได้
3. ทำ **Caching, keepAlive และ Auto-dispose** เพื่อควบคุมอายุของ State และประหยัดทรัพยากรได้
4. ออกแบบ **Dependency Injection (DI)** ด้วย Riverpod ฉีด Repository/Service โดยไม่พึ่ง package อื่น
5. เชื่อมต่อ **REST API ขั้นสูงด้วย Dio** ทำ Interceptors และ Error Handling อย่างเป็นระบบได้
6. ออกแบบ **Loading State ระดับ Production** ด้วย **Skeleton/Shimmer UI** และทำ **Lazy Loading / Pagination / Infinite Scroll** ร่วมกับ Riverpod ได้
7. ลงมือสร้างระบบดึงข้อมูลกรมธรรม์จาก API พร้อม Caching, Pull-to-refresh, Skeleton Loading และยื่นคำขอสินไหม (Workshop Day 2)

> **หมายเหตุสำคัญ:** วันนี้คือหัวใจของการทำงานกับข้อมูลจริง เราจะ "ถอด" Mock Repository ของวันที่ 1 ออก แล้วเสียบ Repository ที่เรียก API จริงผ่าน Dio เข้าไปแทน — โดยที่ชั้น Presentation (UI/Controller) แทบไม่ต้องแก้เลย นี่คือพลังของการแยกชั้นและ DI ที่เราวางไว้ตั้งแต่วันแรก

---

## 🧭 กำหนดการวันที่ 2 (โดยสังเขป)

| เวลา        | หัวข้อ                                                                         |
| ----------- | ------------------------------------------------------------------------------ |
| 09:00–09:15 | ทบทวนวันที่ 1 + ภาพรวมวันนี้                                                   |
| 09:15–10:15 | **Module 1** Asynchronous Data Management (AsyncValue & AsyncNotifier)         |
| 10:15–11:15 | **Module 2** Smart Caching & Optimization (keepAlive, Auto-dispose, Polling)   |
| 11:15–12:00 | **Module 3** Dependency Injection with Riverpod                                |
| 12:00–13:00 | พักกลางวัน                                                                     |
| 13:00–14:00 | **Module 4** Advanced API Integration ด้วย Dio (Interceptors + Error Handling) |
| 14:00–15:00 | **Module 5** Skeleton Loading, Shimmer UI & Lazy Loading                       |
| 15:00–16:00 | **Workshop Day 2** API + Caching + Pull-to-refresh + Skeleton + Claim Mutation |

---

## 📚 Module 1: Asynchronous Data Management

### เวลา 09:15–10:15 น.

> 💡 **หัวใจของ Module นี้:** ในโลกจริงข้อมูลมาจากเครือข่ายที่ "ช้าและพังได้" การจัดการ 3 สถานะ — กำลังโหลด, ได้ข้อมูล, เกิดข้อผิดพลาด — อย่างเป็นระบบคือความต่างระหว่างแอปสมัครเล่นกับแอประดับองค์กร และ Riverpod ห่อทั้ง 3 สถานะนี้ไว้ใน `AsyncValue` ตัวเดียว

---

### 1.1 ทำความรู้จัก AsyncValue

`AsyncValue<T>` คือชนิดข้อมูลที่แทน "ผลลัพธ์ของงาน async" ซึ่งมีได้ 3 สถานะ มันบังคับให้เราจัดการทุกสถานะ จึงไม่มีทางลืม loading หรือ error เหมือนการเขียนมือ

```
สถานะของ AsyncValue<List<Policy>>:
┌───────────────────────────────────────────────────────┐
│  AsyncLoading   → กำลังดึงข้อมูล (แสดง spinner)         │
│  AsyncData      → ได้ข้อมูลแล้ว (แสดงรายการ)            │
│  AsyncError     → เกิดข้อผิดพลาด (แสดงข้อความ + ปุ่มลองใหม่)│
└───────────────────────────────────────────────────────┘
```

### 1.2 จัดการ 3 สถานะด้วย .when()

`.when()` คือวิธีมาตรฐานในการแปลง `AsyncValue` ออกมาเป็น UI ครบทุกสถานะในที่เดียว

```dart
// ใน build() ของ ConsumerWidget
final asyncPolicies = ref.watch(policyListProvider);

return asyncPolicies.when(
  loading: () => const Center(child: CircularProgressIndicator()),
  error: (error, stackTrace) => Center(
    child: Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        Text('โหลดข้อมูลไม่สำเร็จ: $error'),
        ElevatedButton(
          onPressed: () => ref.invalidate(policyListProvider), // ลองใหม่
          child: const Text('ลองอีกครั้ง'),
        ),
      ],
    ),
  ),
  data: (policies) => ListView(/* ... แสดงรายการ ... */),
);
```

| เมธอด/พร็อพเพอร์ตี้                  | ความหมาย                                     |
| ------------------------------------ | -------------------------------------------- |
| `.when()`                            | จัดการครบ 3 สถานะ (loading/error/data)       |
| `.valueOrNull`                       | ดึงค่าถ้ามี ไม่งั้นได้ `null` (ไม่ throw)    |
| `.isLoading`                         | กำลังโหลดอยู่หรือไม่ (true/false)            |
| `.hasError`                          | มี error หรือไม่                             |
| `.when(skipLoadingOnRefresh: false)` | ควบคุมว่าให้แสดง loading ตอน refresh หรือไม่ |

### 1.3 AsyncNotifier — ดึงและแก้ไขข้อมูลในคลาสเดียว

`AsyncNotifier` คือ provider สำหรับ State แบบ async ที่มีทั้ง "ดึงข้อมูล" (ใน `build`) และ "แก้ไขข้อมูล" (เมธอดเพิ่มเติม) จุดเด่นคือเราสามารถสั่งให้ดึงใหม่ หรือเปลี่ยน state เป็น loading/error เองได้

```dart
// policy_list_controller.dart (เวอร์ชัน AsyncNotifier เต็มรูปแบบ)
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../../data/policy_repository.dart';
import '../../domain/policy.dart';

part 'policy_list_controller.g.dart';

@riverpod
class PolicyListController extends _$PolicyListController {
  // build() ทำหน้าที่ "ดึงข้อมูลครั้งแรก" — ค่าที่ return จะถูกห่อเป็น AsyncData
  @override
  Future<List<Policy>> build() async {
    final repository = ref.watch(policyRepositoryProvider);
    return repository.fetchPolicies();
  }

  // เมธอดสำหรับสั่งโหลดใหม่ (เช่น pull-to-refresh)
  Future<void> refresh() async {
    state = const AsyncValue.loading();
    // AsyncValue.guard ดึงข้อมูลและจับ error ให้อัตโนมัติ
    state = await AsyncValue.guard(() {
      return ref.read(policyRepositoryProvider).fetchPolicies();
    });
  }
}
```

> 💡 **AsyncValue.guard คือพระเอก:** มันรับฟังก์ชัน async แล้วคืน `AsyncData` ถ้าสำเร็จ หรือ `AsyncError` ถ้า throw โดยอัตโนมัติ ทำให้เราไม่ต้องเขียน `try/catch` เองทุกครั้ง ลดความผิดพลาดของการจัดการ error

---

## 📚 Module 2: Smart Caching & Optimization

### เวลา 10:15–11:15 น.

> 💡 **หัวใจของ Module นี้:** โดยปริยาย Riverpod จะ "ทำลาย" State ทิ้งเมื่อไม่มีใครใช้ (auto-dispose) ซึ่งดีต่อหน่วยความจำ แต่บางครั้งเราอยากให้ข้อมูลอยู่ต่อ (cache) เพื่อไม่ต้องโหลดซ้ำ — การควบคุมจุดนี้คือศิลปะของการ optimize

---

### 2.1 Auto-dispose ทำงานอย่างไร

provider ที่สร้างด้วย `@riverpod` จะเป็น auto-dispose โดยปริยาย — เมื่อไม่มี widget ใด watch อยู่แล้ว มันจะทำลาย state และเริ่มใหม่ในครั้งถัดไป

```
ผู้ใช้เปิดหน้า A (watch provider)  →  provider สร้าง state, ดึงข้อมูล
ผู้ใช้ออกจากหน้า A (ไม่มีใคร watch) →  provider ทำลาย state อัตโนมัติ
ผู้ใช้กลับเข้าหน้า A อีกครั้ง       →  provider ดึงข้อมูลใหม่ (อาจช้า)
```

### 2.2 keepAlive — เก็บ State ไว้ไม่ให้ถูกทำลาย

ถ้าข้อมูลโหลดยาก/ช้า และเราอยากให้มัน cache ไว้แม้ออกจากหน้าไปแล้ว ใช้ `ref.keepAlive()`

```dart
@riverpod
Future<List<Policy>> policyList(PolicyListRef ref) async {
  // เก็บ state ไว้ไม่ให้ถูกทำลายเมื่อไม่มีใคร watch
  final link = ref.keepAlive();

  // ตัวอย่าง: ยกเลิก cache หลังผ่านไป 5 นาที เพื่อให้ข้อมูลไม่เก่าเกินไป
  final timer = Timer(const Duration(minutes: 5), () => link.close());
  ref.onDispose(timer.cancel);

  final repository = ref.watch(policyRepositoryProvider);
  return repository.fetchPolicies();
}
```

| สถานการณ์                                    | ควรใช้                 |
| -------------------------------------------- | ---------------------- |
| ข้อมูลเปลี่ยนบ่อย/เบา เช่น แจ้งเตือน         | Auto-dispose (ปริยาย)  |
| ข้อมูลโหลดหนัก/ช้า เช่น รายการกรมธรรม์       | `keepAlive()`          |
| ข้อมูลที่ต้อง fresh เสมอ เช่น ยอดเงินคงเหลือ | Auto-dispose + Polling |

### 2.3 Polling & Auto-refresh — ดึงข้อมูลซ้ำเป็นรอบ

บางข้อมูลต้องอัปเดตเองเป็นระยะ (เช่น สถานะการอนุมัติสินไหม) เราทำ polling ได้ด้วยการตั้ง timer ภายใน provider

```dart
@riverpod
Future<ClaimStatus> claimStatus(ClaimStatusRef ref, String claimId) async {
  // ตั้งให้ provider นี้ invalidate ตัวเองทุก 10 วินาที → ดึงใหม่อัตโนมัติ
  final timer = Timer(const Duration(seconds: 10), () {
    ref.invalidateSelf();
  });
  ref.onDispose(timer.cancel); // อย่าลืมเคลียร์ timer เมื่อ provider ถูกทำลาย

  final repository = ref.watch(claimRepositoryProvider);
  return repository.fetchClaimStatus(claimId);
}
```

> ⚠️ **ข้อควรระวัง:** ทุกครั้งที่สร้าง resource (Timer, StreamSubscription, controller) ภายใน provider ต้องเคลียร์ใน `ref.onDispose()` เสมอ มิฉะนั้นจะเกิด memory leak — นี่คือบั๊กที่ตรวจจับยากในแอปจริง

### 2.4 invalidate vs refresh

| คำสั่ง                     | พฤติกรรม                                                       | ใช้เมื่อ                    |
| -------------------------- | -------------------------------------------------------------- | --------------------------- |
| `ref.invalidate(provider)` | ทำเครื่องหมายว่า "เก่าแล้ว" จะดึงใหม่เมื่อมีคน watch ครั้งหน้า | ต้องการให้ refresh แบบ lazy |
| `ref.refresh(provider)`    | สั่งดึงใหม่ทันที และคืนค่าใหม่กลับมา                           | ต้องการผลลัพธ์ใหม่เดี๋ยวนี้ |
| `ref.invalidateSelf()`     | provider สั่งให้ตัวเองดึงใหม่                                  | ใช้ใน polling               |

---

## 📚 Module 3: Dependency Injection with Riverpod

### เวลา 11:15–12:00 น.

> 💡 **หัวใจของ Module นี้:** Riverpod เป็น DI container ในตัวอยู่แล้ว เราไม่ต้องใช้ `get_it` หรือ package DI อื่น — แค่ประกาศ provider ของ dependency แล้วให้ provider อื่น `ref.watch` มัน นี่คือการ "ฉีด" dependency ที่ทดสอบง่ายที่สุด

---

### 3.1 DI คืออะไร และทำไมต้องมี

Dependency Injection คือการ "ส่ง" สิ่งที่คลาสต้องใช้เข้าไปจากภายนอก แทนที่จะให้คลาสสร้างเอง ข้อดีคือเราสลับของจริง/ของจำลองได้ง่าย (สำคัญมากตอนเขียน Test ในวันที่ 3)

```
แบบไม่มี DI (แย่):                    แบบมี DI ด้วย Riverpod (ดี):
┌─────────────────────────────┐       ┌─────────────────────────────┐
│ class Controller {          │       │ Controller watch             │
│   final repo =              │  ⟶   │   repositoryProvider          │
│     PolicyRepository();     │       │ → ทดสอบ override เป็น mock ได้ │
│   // ผูกตายตัว เปลี่ยนไม่ได้ │       │ → สลับ API จริง/จำลองได้       │
│ }                           │       │                             │
└─────────────────────────────┘       └─────────────────────────────┘
```

### 3.2 การฉีด Dependency เป็นลูกโซ่ (Provider Graph)

ในแอปจริง dependency มักเป็นชั้น ๆ เช่น Controller ใช้ Repository, Repository ใช้ Dio client เราเชื่อมทั้งหมดด้วย provider ที่ watch กันเป็นทอด ๆ

```dart
// 1) Provider ของ Dio client (dependency ระดับล่างสุด)
@riverpod
Dio dioClient(DioClientRef ref) {
  final dio = Dio(BaseOptions(
    baseUrl: 'https://api.bla-demo.example.com',
    connectTimeout: const Duration(seconds: 10),
  ));
  return dio;
}

// 2) Repository watch Dio client (ฉีด Dio เข้ามา)
@riverpod
PolicyRepository policyRepository(PolicyRepositoryRef ref) {
  final dio = ref.watch(dioClientProvider);
  return PolicyRepository(dio);
}

// 3) Controller watch Repository (ฉีด Repository เข้ามา) — ทำใน build() ของ Notifier
//    @riverpod class PolicyListController ... build() { ref.watch(policyRepositoryProvider) }
```

```
ลูกโซ่ Dependency:
dioClientProvider  →  policyRepositoryProvider  →  policyListControllerProvider
   (Dio)                  (Repository)                  (State + UI)
```

> ✅ **จุดสำคัญ:** เมื่อเขียน Test เราเพียง override `policyRepositoryProvider` ให้คืน Mock ก็ตัดทั้งลูกโซ่ที่อยู่ใต้มัน (Dio, network) ออกได้ทันที — ทดสอบเร็วและไม่ต้องต่อเน็ต

---

## 📚 Module 4: Advanced API Integration ด้วย Dio

### เวลา 13:00–14:00 น.

> 💡 **หัวใจของ Module นี้:** Dio คือ HTTP client ที่ทรงพลังกว่า `http` ธรรมดา จุดเด่นคือ **Interceptors** — ตัวดักจับ request/response ที่ให้เราแทรก logic ส่วนกลาง เช่น แนบ token, log, จัดการ error และ retry ได้โดยไม่ต้องเขียนซ้ำทุกที่

---

### 4.1 ตั้งค่า Dio พื้นฐาน

```dart
// core/network/dio_client.dart
import 'package:dio/dio.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'dio_client.g.dart';

@riverpod
Dio dioClient(DioClientRef ref) {
  final dio = Dio(
    BaseOptions(
      baseUrl: 'https://api.bla-demo.example.com/v1',
      connectTimeout: const Duration(seconds: 10),
      receiveTimeout: const Duration(seconds: 10),
      headers: {'Content-Type': 'application/json'},
    ),
  );

  // ใส่ interceptor (รายละเอียดข้อ 4.2)
  dio.interceptors.add(LogInterceptor(responseBody: true));

  return dio;
}
```

### 4.2 Interceptors — ดักจับ Request/Response ส่วนกลาง

Interceptor มี 3 จุดให้แทรก logic: ก่อนส่ง request, เมื่อได้ response, และเมื่อเกิด error

```dart
// core/network/auth_interceptor.dart
import 'package:dio/dio.dart';

class AuthInterceptor extends Interceptor {
  // ฟังก์ชันดึง token (ในวันที่ 3 จะมาจาก Secure Storage)
  final Future<String?> Function() getToken;

  AuthInterceptor(this.getToken);

  @override
  void onRequest(
    RequestOptions options,
    RequestInterceptorHandler handler,
  ) async {
    // แนบ token เข้าทุก request โดยอัตโนมัติ
    final token = await getToken();
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options); // ส่งต่อให้ทำงานต่อ
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    // จุดรวมศูนย์สำหรับจัดการ error เช่น token หมดอายุ (401)
    if (err.response?.statusCode == 401) {
      // วันที่ 3: ที่นี่คือจุด logout อัตโนมัติ
    }
    handler.next(err);
  }
}
```

### 4.3 จัดการ Error อย่างเป็นระบบ (Error Mapping)

แทนที่จะปล่อย `DioException` ดิบ ๆ ขึ้นไปถึง UI เราควรแปลงมันเป็น Exception ของแอปที่สื่อความหมาย เพื่อให้ UI แสดงข้อความที่ผู้ใช้เข้าใจ

```dart
// core/network/api_exception.dart
class ApiException implements Exception {
  final String message;
  final int? statusCode;
  const ApiException(this.message, {this.statusCode});

  @override
  String toString() => message;
}

// แปลง DioException → ApiException ที่เป็นมิตรกับผู้ใช้
ApiException mapDioError(DioException error) {
  switch (error.type) {
    case DioExceptionType.connectionTimeout:
    case DioExceptionType.receiveTimeout:
      return const ApiException('การเชื่อมต่อใช้เวลานานเกินไป กรุณาลองใหม่');
    case DioExceptionType.badResponse:
      final code = error.response?.statusCode;
      if (code == 401) return const ApiException('เซสชันหมดอายุ กรุณาเข้าสู่ระบบใหม่', statusCode: 401);
      if (code == 404) return const ApiException('ไม่พบข้อมูลที่ต้องการ', statusCode: 404);
      return ApiException('เซิร์ฟเวอร์ตอบกลับผิดพลาด ($code)', statusCode: code);
    case DioExceptionType.connectionError:
      return const ApiException('ไม่สามารถเชื่อมต่อเครือข่ายได้');
    default:
      return const ApiException('เกิดข้อผิดพลาดที่ไม่คาดคิด');
  }
}
```

---

## 📚 Module 5: Skeleton Loading, Shimmer UI & Lazy Loading

### เวลา 14:00–15:00 น.

> 💡 **หัวใจของ Module นี้:** `CircularProgressIndicator` กลางจอบอกผู้ใช้แค่ว่า "รอเถอะ" แต่ไม่บอกว่ากำลังจะได้อะไร แอประดับ Production สมัยใหม่จึงใช้ **Skeleton Screen** — โครงร่างจาง ๆ ของเนื้อหาจริงที่กำลังจะมา ทำให้ผู้ใช้รู้สึกว่าแอป "เร็วขึ้น" ทั้งที่เวลาโหลดเท่าเดิม (perceived performance) และเมื่อข้อมูลมีจำนวนมาก เราต้องโหลดเป็นช่วง ๆ (Lazy Loading) แทนการดึงทั้งก้อน

---

### 5.1 ทำไมต้องเลิกใช้ Loading Spinner กลางจอ

| รูปแบบ Loading      | ผู้ใช้รับรู้                              | เหมาะกับ                             |
| ------------------- | ----------------------------------------- | ------------------------------------ |
| Spinner กลางจอ      | "กำลังโหลด แต่ไม่รู้ว่านานแค่ไหน/ได้อะไร" | งานสั้น ๆ, ปุ่มกด, dialog            |
| **Skeleton Screen** | "เห็นเค้าโครงเนื้อหาแล้ว ใกล้เสร็จ"       | หน้า list / รายละเอียดที่โหลดจาก API |
| Progress bar (%)    | "รู้ความคืบหน้าชัดเจน"                    | ดาวน์โหลด/อัปโหลดที่วัด % ได้        |

> 💡 **หลักคิด UX:** Skeleton ที่ดีต้องมี "รูปร่างใกล้เคียงเนื้อหาจริง" (การ์ด, บรรทัดข้อความ, วงกลม avatar) เพื่อให้สายตาผู้ใช้ปรับตัวล่วงหน้า เมื่อข้อมูลจริงมาถึงจึงไม่กระตุก (layout shift)

### 5.2 ทางเลือกที่ 1 — `shimmer` (วาดโครงร่างเอง)

`shimmer` Package ให้เอฟเฟกต์แสงไล่ผ่าน (shimmer) บน widget ใด ๆ ที่เราวาดเป็นโครงร่าง เหมาะเมื่อต้องการคุมรูปทรง skeleton เองทุกจุด

```bash
flutter pub add shimmer
```

```dart
// lib/features/policy/presentation/widgets/policy_skeleton.dart
import 'package:flutter/material.dart';
import 'package:shimmer/shimmer.dart';

class PolicyListSkeleton extends StatelessWidget {
  const PolicyListSkeleton({super.key});

  @override
  Widget build(BuildContext context) {
    return Shimmer.fromColors(
      baseColor: Colors.grey.shade300,
      highlightColor: Colors.grey.shade100,
      // วาดการ์ดเปล่า 6 ใบ เลียนแบบโครงของ _PolicyTile จริง
      child: ListView.builder(
        itemCount: 6,
        itemBuilder: (context, index) => const _SkeletonTile(),
      ),
    );
  }
}

class _SkeletonTile extends StatelessWidget {
  const _SkeletonTile();

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
      child: ListTile(
        leading: const CircleAvatar(backgroundColor: Colors.white),
        title: Container(height: 14, color: Colors.white),
        subtitle: Padding(
          padding: const EdgeInsets.only(top: 8),
          child: Container(height: 12, width: 160, color: Colors.white),
        ),
      ),
    );
  }
}
```

### 5.3 ทางเลือกที่ 2 — `skeletonizer` (สร้าง skeleton จาก UI จริงอัตโนมัติ)

`skeletonizer` ฉลาดกว่า — มัน "แปลง" widget เนื้อหาจริงให้กลายเป็น skeleton ให้อัตโนมัติ เราจึงไม่ต้องวาดโครงซ้ำสองรอบ (ลดโค้ดและกัน skeleton ไม่ตรงกับ UI จริง)

```bash
flutter pub add skeletonizer
```

```dart
// ใช้ Skeletonizer ครอบ ListView เดิม แล้วสลับด้วย flag enabled
import 'package:skeletonizer/skeletonizer.dart';

// ภายใน build() ของ ConsumerWidget
final asyncPolicies = ref.watch(policyListControllerProvider);

return Skeletonizer(
  // เปิด skeleton เมื่อยังโหลดอยู่ ปิดเมื่อได้ข้อมูลจริง
  enabled: asyncPolicies.isLoading,
  child: ListView.builder(
    // ตอนโหลด ใช้ข้อมูลปลอม (fake) เพื่อให้มีอะไรให้ skeletonize
    itemCount: asyncPolicies.valueOrNull?.length ?? 6,
    itemBuilder: (context, index) {
      final policies = asyncPolicies.valueOrNull;
      final policy = policies == null
          ? PolicyFake.fake() // ข้อมูลปลอมระหว่างโหลด (ดูข้อ 5.4)
          : policies[index];
      return _PolicyTile(policy: policy);
    },
  ),
);
```

> ✅ **เลือกอย่างไร:** ถ้าอยากได้เร็วและ UI ตรงกับของจริงเสมอ ใช้ **`skeletonizer`** เป็นค่าเริ่มต้น แต่ถ้าต้องการคุมรูปทรง skeleton แบบพิเศษ (เช่น หน้า dashboard ที่ layout ซับซ้อน) ให้วาดเองด้วย **`shimmer`**

### 5.4 เตรียมข้อมูลปลอม (Fake) สำหรับ Skeletonizer

```dart
// เพิ่ม factory ใน domain/policy.dart สำหรับโหมด skeleton เท่านั้น
extension PolicyFake on Policy {
  static Policy fake() => const Policy(
        id: 'xxxx',
        planName: 'BLA xxxxxxxxxxxxxx', // ความยาวใกล้เคียงของจริง
        policyNumber: 'BLA-2569-0000',
        premium: 00000,
        status: PolicyStatus.active,
      );
}
```

### 5.5 Lazy Loading — โหลดทีละหน้า (Pagination + Infinite Scroll)

เมื่อรายการมีจำนวนมาก (เช่น ประวัติการชำระเบี้ยหลายร้อยรายการ) การดึงทั้งหมดในครั้งเดียวทั้งช้าและกิน memory เราจึงโหลด "ทีละหน้า" แล้วโหลดเพิ่มเมื่อผู้ใช้เลื่อนใกล้ท้าย list

```dart
// lib/features/payment/presentation/controllers/payment_history_controller.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../../data/payment_repository.dart';
import '../../domain/payment.dart';

part 'payment_history_controller.g.dart';

@riverpod
class PaymentHistoryController extends _$PaymentHistoryController {
  int _page = 1;
  bool _hasMore = true;
  final List<Payment> _items = [];

  @override
  Future<List<Payment>> build() async {
    // โหลดหน้าแรกตอนสร้าง provider
    return _fetchPage(reset: true);
  }

  Future<List<Payment>> _fetchPage({bool reset = false}) async {
    if (reset) {
      _page = 1;
      _hasMore = true;
      _items.clear();
    }
    final repo = ref.read(paymentRepositoryProvider);
    final pageItems = await repo.fetchPayments(page: _page, limit: 20);
    _hasMore = pageItems.length == 20; // ถ้าได้ครบ 20 แปลว่าอาจมีหน้าถัดไป
    _items.addAll(pageItems);
    return List.unmodifiable(_items);
  }

  // เรียกเมื่อผู้ใช้เลื่อนใกล้ท้าย list
  Future<void> loadMore() async {
    if (!_hasMore || state.isLoading) return;
    _page++;
    final more = await _fetchPage();
    state = AsyncValue.data(more);
  }
}
```

```dart
// UI: ตรวจจับการเลื่อนใกล้ท้ายด้วย ScrollController
class PaymentHistoryPage extends ConsumerStatefulWidget {
  const PaymentHistoryPage({super.key});

  @override
  ConsumerState<PaymentHistoryPage> createState() =>
      _PaymentHistoryPageState();
}

class _PaymentHistoryPageState extends ConsumerState<PaymentHistoryPage> {
  final _scrollController = ScrollController();

  @override
  void initState() {
    super.initState();
    _scrollController.addListener(_onScroll);
  }

  void _onScroll() {
    // เหลืออีก 200px ถึงท้าย → โหลดหน้าถัดไปล่วงหน้า
    final position = _scrollController.position;
    if (position.pixels >= position.maxScrollExtent - 200) {
      ref.read(paymentHistoryControllerProvider.notifier).loadMore();
    }
  }

  @override
  void dispose() {
    _scrollController.dispose(); // กัน memory leak เสมอ
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final asyncPayments = ref.watch(paymentHistoryControllerProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('ประวัติการชำระเบี้ย')),
      body: asyncPayments.when(
        loading: () => const PolicyListSkeleton(), // ใช้ skeleton จากข้อ 5.2
        error: (error, _) => Center(child: Text('โหลดไม่สำเร็จ: $error')),
        data: (payments) => ListView.builder(
          controller: _scrollController,
          itemCount: payments.length,
          itemBuilder: (context, index) {
            final payment = payments[index];
            return ListTile(
              title: Text(payment.title),
              trailing: Text('${payment.amount.toStringAsFixed(0)} บาท'),
            );
          },
        ),
      ),
    );
  }
}
```

> ⚠️ **ข้อควรระวัง:** `ListView.builder` สำคัญมากสำหรับ Lazy Loading เพราะมัน "สร้าง widget เฉพาะรายการที่มองเห็น" (lazy) ไม่ใช่สร้างทั้งหมดเหมือน `ListView(children: [...])` — การใช้ `ListView` ธรรมดากับข้อมูลหลายร้อยรายการคือสาเหตุของ jank และ memory ที่พบบ่อยที่สุด

---

## 🛠️ Workshop Day 2 — BLA Policy Companion: API + Caching + Pull-to-refresh + Skeleton + Claim

### เวลา 15:00–16:00 น.

> **โจทย์:** เปลี่ยน Mock Repository ของวันที่ 1 ให้ดึงข้อมูลจาก **Mock API จริงบน Render** ผ่าน Dio, อัปเกรดโมเดลเป็น `freezed` + JSON, ทำ Caching ด้วย keepAlive + Pull-to-refresh, แสดง **Skeleton Loading ด้วย skeletonizer**, เพิ่ม "ยื่นคำขอสินไหม (Claim)" แบบ Mutation พร้อมจัดการ Error State และทำประวัติการชำระเบี้ยแบบ **Pagination / Infinite Scroll** — เนื้อหาส่วนนี้คือ **โค้ดจริงปัจจุบันครบทุกไฟล์** ของ branch `day2` ที่เพิ่ม/เปลี่ยนจาก `day1`

### 🗺️ ภาพรวม — อะไรเพิ่ม/เปลี่ยนจาก day1

**ไฟล์เพิ่มใหม่ (A)** — 10 ไฟล์ + generated อีก 8:

| ไฟล์ใหม่                                                                   | หน้าที่                                                       |
| -------------------------------------------------------------------------- | ------------------------------------------------------------- |
| `core/config/app_config.dart`                                              | ค่า config กลาง: baseUrl, demo token, timeout                 |
| `core/network/api_exception.dart`                                          | แปลง `DioException` → ข้อความไทยที่เป็นมิตร                   |
| `core/network/auth_interceptor.dart`                                       | แนบ Bearer token ทุก request + ดักจับ 401                     |
| `core/network/dio_client.dart` (+`.g`)                                     | Provider ของ Dio (ประกอบ config + interceptors)               |
| `core/utils/date_format.dart`                                              | `thaiDate()` (พ.ศ.) + `money()` (คอมมา) ใช้ร่วม claim/payment |
| `claim/domain/claim_record.dart`                                           | โมเดลประวัติสินไหม + `fromJson` เขียนมือ                      |
| `claim/data/claim_repository.dart` (+`.g`)                                 | GET /claims + `claimListProvider`                             |
| `claim/presentation/controllers/claim_controller.dart` (+`.g`)             | Mutation ยื่นสินไหม + invalidate รายการ                       |
| `payment/domain/payment.dart`                                              | โมเดลรายการชำระเบี้ย                                          |
| `payment/data/payment_repository.dart` (+`.g`)                             | GET /payments?page=&limit= (แบ่งหน้า)                         |
| `payment/presentation/controllers/payment_history_controller.dart` (+`.g`) | Pagination / Infinite Scroll                                  |
| `policy/domain/policy.freezed.dart`, `policy.g.dart`                       | (build_runner สร้างจาก `@freezed`)                            |

**ไฟล์ที่แก้ไข (M)** — `pubspec.yaml` (เพิ่ม dio + skeletonizer), `policy/domain/policy.dart` (→ freezed + `PolicyFake`), `policy/data/policy_repository.dart` (mock → Dio), `policy_list_controller.dart` (FutureProvider → AsyncNotifier + cache), `policy_list_page.dart` (Refresh + Skeleton + ErrorBox), `policy_detail_page.dart` (ส่ง `policyId` เข้า sheet), `claims_page.dart` (API จริง + skeleton), `file_claim_sheet.dart` (mock → ClaimController), `payments_page.dart` (ประวัติ → Infinite Scroll), `home_page.dart` (แก้ const เล็กน้อย — ไม่มีผลเชิงพฤติกรรม)

**กราฟ Provider ของ Day 2** — ทุกเส้นทางข้อมูลเริ่มจาก `dioClientProvider` ตัวเดียว:

```
AppConfig (ค่าคงที่: baseUrl / demoToken / timeout)
   │ อ่านโดย
   ▼
dioClientProvider (Dio + AuthInterceptor + LogInterceptor)
   ├─▶ policyRepositoryProvider ──▶ policyListProvider (AsyncNotifier `PolicyList`)
   │        │                          │  • cache 5 นาที (keepAlive + Timer)
   │        │                          │  • refresh() สำหรับ Pull-to-refresh
   │        │                          ▼
   │        │                    filteredPolicyListProvider ◀── policySearchProvider
   │        │                          │                    ◀── policyStatusFilterProvider
   │        │                          ▼
   │        │                    PolicyListPage (UI)
   │        └────────────▶ claimControllerProvider (Mutation: submitClaim)
   │                            │ สำเร็จแล้ว invalidate ↴
   ├─▶ claimRepositoryProvider ─▶ claimListProvider ──▶ ClaimsPage
   └─▶ paymentRepositoryProvider ─▶ paymentHistoryControllerProvider ──▶ PaymentsPage
                                        (Pagination: build()=หน้าแรก, loadMore()=หน้าถัดไป)
```

### ขั้นที่ 1 — เพิ่ม dependency ใน pubspec.yaml

```yaml
dependencies:
  # ...ของเดิมจาก Day 1...
  # Day 2: เชื่อมต่อ REST API + UI โหลด
  dio: ^5.7.0
  skeletonizer: ^2.1.3
```

```bash
flutter pub add dio:^5.7.0 skeletonizer:^2.1.3
```

### ขั้นที่ 2 — ค่า config กลาง (core/config/app_config.dart) 🆕

Single Source of Truth ของค่าที่ขึ้นกับสภาพแวดล้อม — จุดเดียวที่ต้องแก้เมื่อสลับ dev/staging/prod

```dart
// lib/core/config/app_config.dart

/// ค่าตั้งของสภาพแวดล้อม (Environment Config) — Single Source of Truth
///
/// รวมค่า config ที่ขึ้นกับสภาพแวดล้อมไว้ที่เดียว แก้ที่นี่ที่เดียวเพื่อสลับ
/// dev / staging / prod หรือเปลี่ยน endpoint โดยไม่ต้องไล่แก้หลายไฟล์
abstract final class AppConfig {
  /// Base URL ของ Mock API (deploy บน Render)
  static const String apiBaseUrl = 'https://bla-mock-api.onrender.com/v1';

  /// Base URL ของรูปภาพ (เผื่อใช้ในอนาคต — ตัวอย่างการเก็บค่า config รวมที่เดียว)
  static const String imageBaseUrl = 'https://bla-mock-api.onrender.com/images';

  /// โทเคนทดสอบสำหรับวันที่ 2 (ยังไม่มีระบบ Login จริง)
  ///
  /// วันที่ 3 จะเลิกใช้ค่านี้ แล้วให้ AuthInterceptor อ่าน token จริงจาก
  /// Secure Storage หลังผู้ใช้เข้าสู่ระบบแทน
  static const String demoToken = 'mock-token-abc123';

  /// timeout การเชื่อมต่อ/รับข้อมูล (เผื่อ cold start ของ Render free tier ~30-60 วิ)
  static const Duration connectTimeout = Duration(seconds: 25);
  static const Duration receiveTimeout = Duration(seconds: 25);
}
```

> 🔗 **การเชื่อมโยง:** ถูกอ่านโดย `dio_client.dart` เท่านั้น (baseUrl, timeout, demoToken) — แต่แยกไฟล์ไว้เพื่อให้ Day 3 เพิ่มค่าอื่น (เช่น key ของ Secure Storage) ได้โดยไม่แตะชั้น network

### ขั้นที่ 3 — ชั้น Network กลาง (core/network) 🆕 3 ไฟล์

#### 3.1 `api_exception.dart` — แปล error ให้เป็นภาษาคน

```dart
// lib/core/network/api_exception.dart
import 'package:dio/dio.dart';

/// Exception ของแอปที่สื่อความหมายกับผู้ใช้ (แปลงมาจาก DioException)
class ApiException implements Exception {
  final String message;
  final int? statusCode;
  const ApiException(this.message, {this.statusCode});

  @override
  String toString() => message;
}

/// แปลง DioException ดิบ → ApiException ที่เป็นมิตรกับผู้ใช้
///
/// รวม logic การแปล error ไว้ที่เดียว เรียกใช้จากทุก repository
ApiException mapDioError(DioException error) {
  switch (error.type) {
    case DioExceptionType.connectionTimeout:
    case DioExceptionType.sendTimeout:
    case DioExceptionType.receiveTimeout:
      return const ApiException('การเชื่อมต่อใช้เวลานานเกินไป กรุณาลองใหม่');
    case DioExceptionType.badResponse:
      final code = error.response?.statusCode;
      if (code == 401) {
        return const ApiException('เซสชันหมดอายุ กรุณาเข้าสู่ระบบใหม่',
            statusCode: 401);
      }
      if (code == 404) {
        return const ApiException('ไม่พบข้อมูลที่ต้องการ', statusCode: 404);
      }
      return ApiException('เซิร์ฟเวอร์ตอบกลับผิดพลาด ($code)', statusCode: code);
    case DioExceptionType.connectionError:
      return const ApiException('ไม่สามารถเชื่อมต่อเครือข่ายได้');
    default:
      return const ApiException('เกิดข้อผิดพลาดที่ไม่คาดคิด');
  }
}
```

> 🔗 **การเชื่อมโยง:** ทุก repository (policy/claim/payment) ครอบการเรียก Dio ด้วย `try { ... } on DioException catch (e) { throw mapDioError(e); }` — และเพราะ `toString()` คืน `message` ตรง ๆ ทำให้ UI ที่เขียน `Text('$e')` แสดงข้อความไทยสวย ๆ โดยอัตโนมัติ (สังเกตใน `_ErrorBox` ของหน้า policy)

#### 3.2 `auth_interceptor.dart` — แนบ token + ดักจับ 401

```dart
// lib/core/network/auth_interceptor.dart
import 'package:dio/dio.dart';

/// Interceptor ที่แนบ Bearer token เข้าทุก request และดักจับ 401
///
/// รับ [getToken] เป็นฟังก์ชันอ่าน token — Day 2 คืน demo token,
/// Day 3 จะสลับให้คืน token จาก Secure Storage โดยไม่ต้องแก้ที่อื่น
class AuthInterceptor extends Interceptor {
  final Future<String?> Function() getToken;
  final Future<void> Function()? onUnauthorized;

  AuthInterceptor(this.getToken, {this.onUnauthorized});

  @override
  void onRequest(
    RequestOptions options,
    RequestInterceptorHandler handler,
  ) async {
    final token = await getToken();
    if (token != null && token.isNotEmpty) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    if (err.response?.statusCode == 401) {
      await onUnauthorized?.call();
    }
    handler.next(err);
  }
}
```

> 🔗 **การเชื่อมโยง:** รับ `getToken` เป็น **ฟังก์ชัน** (ไม่ใช่ค่า) — นี่คือ Dependency Injection ระดับฟังก์ชัน: Day 2 ส่ง `() async => AppConfig.demoToken`, Day 3 เปลี่ยนเป็นอ่านจาก Secure Storage โดยคลาสนี้ไม่ต้องแก้เลย · `onUnauthorized` เตรียมไว้สำหรับ auto-logout ใน Day 3

#### 3.3 `dio_client.dart` — Provider ของ Dio (ประตูสู่เครือข่ายทั้งแอป)

```dart
// lib/core/network/dio_client.dart
import 'package:dio/dio.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

import '../config/app_config.dart';
import 'auth_interceptor.dart';

part 'dio_client.g.dart';

/// Provider ของ Dio client — เวอร์ชัน Day 2 (เน้น API Integration ล้วน)
///
/// ค่า config (baseUrl, token, timeout) อ่านจาก [AppConfig] ที่เดียว
/// ยังไม่เปิด SSL Pinning / Crypto Interceptor (เป็นเนื้อหา Day 3)
/// โครงสร้าง interceptor: AuthInterceptor (แนบ token + ดักจับ 401) → LogInterceptor (log)
@riverpod
Dio dioClient(DioClientRef ref) {
  final dio = Dio(
    BaseOptions(
      baseUrl: AppConfig.apiBaseUrl,
      connectTimeout: AppConfig.connectTimeout,
      receiveTimeout: AppConfig.receiveTimeout,
      headers: {'Content-Type': 'application/json'},
    ),
  );

  // 1) แนบ Bearer token เข้าทุก request (Day 2 ใช้ demo token จาก AppConfig)
  dio.interceptors.add(AuthInterceptor(() async => AppConfig.demoToken));

  // 2) log request/response ระหว่างพัฒนา (ปิดได้เมื่อขึ้น production)
  dio.interceptors.add(LogInterceptor(responseBody: true));

  return dio;
}
```

> 🔗 **การเชื่อมโยง:** นี่คือ "รากของกราฟ" — repository ทั้ง 3 ตัว (`policyRepositoryProvider`, `claimRepositoryProvider`, `paymentRepositoryProvider`) ต่างก็ `ref.watch(dioClientProvider)` ทำให้ทั้งแอปแชร์ Dio instance เดียว (interceptor + timeout ชุดเดียว) และตอนเขียนเทสต์ Day 3 override ตัวเดียวก็เปลี่ยนได้ทั้งแอป

### ขั้นที่ 4 — Utility กลาง (core/utils/date_format.dart) 🆕

```dart
// lib/core/utils/date_format.dart

/// ฟอร์แมตวันที่เป็นภาษาไทยแบบย่อ (พ.ศ.) เช่น "12 มิ.ย. 2569"
/// รวมไว้ที่เดียวเพื่อใช้ซ้ำทั้งแอป (claims, payments)
String thaiDate(DateTime dt) {
  const months = [
    'ม.ค.', 'ก.พ.', 'มี.ค.', 'เม.ย.', 'พ.ค.', 'มิ.ย.',
    'ก.ค.', 'ส.ค.', 'ก.ย.', 'ต.ค.', 'พ.ย.', 'ธ.ค.',
  ];
  final local = dt.toLocal();
  return '${local.day} ${months[local.month - 1]} ${local.year + 543}';
}

/// ฟอร์แมตจำนวนเงินใส่คอมมา เช่น 12500 → "12,500"
String money(num value) {
  final s = value.round().toString();
  final buf = StringBuffer();
  for (var i = 0; i < s.length; i++) {
    if (i > 0 && (s.length - i) % 3 == 0) buf.write(',');
    buf.write(s[i]);
  }
  return buf.toString();
}
```

> 🔗 **การเชื่อมโยง:** Day 1 วันที่/จำนวนเงินเป็น string ที่พิมพ์ตายตัวใน mock — Day 2 ข้อมูลจริงมาเป็น `DateTime`/`double` จาก JSON จึงต้องมีฟังก์ชันฟอร์แมตกลาง ใช้ใน `claims_page.dart` และ `payments_page.dart`

### ขั้นที่ 5 — อัปเกรดโมเดล Policy เป็น freezed + JSON (+ PolicyFake สำหรับ Skeleton)

```dart
// lib/features/policy/domain/policy.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'policy.freezed.dart';
part 'policy.g.dart';

/// สถานะของกรมธรรม์
enum PolicyStatus { active, lapsed, pending }

/// โมเดลกรมธรรม์ (ชั้น Domain)
///
/// วันที่ 2 อัปเกรดเป็น **freezed + JSON** — immutable, copyWith, ==, hashCode
/// และ `fromJson`/`toJson` อัตโนมัติ (ลดโค้ดที่เขียนมือและ bug)
@freezed
class Policy with _$Policy {
  const factory Policy({
    required String id,
    required String planName, // ชื่อแผนประกัน
    required String policyNumber, // เลขที่กรมธรรม์
    required double premium, // เบี้ยประกัน (บาท/ปี)
    required PolicyStatus status,
  }) = _Policy;

  /// แปลงจาก JSON ที่ได้จาก API (GET /policies)
  factory Policy.fromJson(Map<String, dynamic> json) => _$PolicyFromJson(json);
}

/// ส่วนขยายเพื่อแปลงสถานะเป็นข้อความภาษาไทย (กฎทางธุรกิจอยู่ในชั้น Domain)
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

/// ข้อมูลปลอม (placeholder) สำหรับโหมด Skeleton Loading เท่านั้น
/// ใช้คู่กับ `skeletonizer` เพื่อให้มี "รูปร่าง" ข้อมูลให้ skeletonize ระหว่างโหลด
extension PolicyFake on Policy {
  static Policy fake() => const Policy(
        id: 'xxxx',
        planName: 'BLA xxxxxxxxxxxxxx',
        policyNumber: 'BLA-2569-0000',
        premium: 0,
        status: PolicyStatus.active,
      );
}
```

> 🔗 **การเชื่อมโยง:** จุดเปลี่ยนสำคัญจาก Day 1 คือ (1) class ธรรมดา → `@freezed` ได้ `copyWith/==` ฟรี (2) เพิ่ม `fromJson` เพราะข้อมูลมาจาก API แล้ว (3) เพิ่ม `PolicyFake.fake()` — การ์ดปลอมที่ `policy_list_page.dart` ใช้แสดงตอน skeleton loading (skeletonizer ต้องมี "ข้อมูลรูปร่างจริง" เพื่อวาดโครง) · build_runner จะสร้าง `policy.freezed.dart` (โครง copyWith/==) และ `policy.g.dart` (fromJson/toJson) ให้ — ดูขั้นที่ 12

### ขั้นที่ 6 — Repository เรียก API จริงผ่าน Dio (แทน mock)

```dart
// lib/features/policy/data/policy_repository.dart
import 'package:dio/dio.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

import '../../../core/network/api_exception.dart';
import '../../../core/network/dio_client.dart';
import '../domain/policy.dart';

part 'policy_repository.g.dart';

/// Repository ของกรมธรรม์ — เวอร์ชันเรียก API จริงผ่าน Dio
///
/// วันที่ 1 เป็น mock ในเครื่อง วันนี้ "ถอด mock ออก เสียบ Dio เข้าไปแทน"
/// โดย signature เหมือนเดิม → Controller/UI ไม่ต้องแก้ (พลังของการแยกชั้น + DI)
class PolicyRepository {
  final Dio _dio;
  PolicyRepository(this._dio);

  /// ดึงรายการกรมธรรม์ — GET /policies
  Future<List<Policy>> fetchPolicies() async {
    try {
      final response = await _dio.get<List<dynamic>>('/policies');
      final data = response.data ?? <dynamic>[];
      return data
          .map((json) => Policy.fromJson(json as Map<String, dynamic>))
          .toList();
    } on DioException catch (e) {
      throw mapDioError(e); // แปลง error ให้เป็นมิตรกับผู้ใช้
    }
  }

  /// ยื่นคำขอสินไหม (Mutation) — POST /policies/:id/claims → คืนเลขอ้างอิง
  Future<String> submitClaim({
    required String policyId,
    required double amount,
    required String reason,
  }) async {
    try {
      final response = await _dio.post<Map<String, dynamic>>(
        '/policies/$policyId/claims',
        data: {'amount': amount, 'reason': reason},
      );
      return response.data?['claimRef'] as String? ?? '-';
    } on DioException catch (e) {
      throw mapDioError(e);
    }
  }
}

/// ฉีด Dio เข้า Repository ผ่าน Provider
@riverpod
PolicyRepository policyRepository(PolicyRepositoryRef ref) {
  return PolicyRepository(ref.watch(dioClientProvider));
}
```

> 🔗 **การเชื่อมโยง:** เทียบกับ Day 1 — คลาสเปลี่ยนภายใน (mock → `_dio.get('/policies')`) แต่ **signature `fetchPolicies()` เท่าเดิม** และชื่อ provider `policyRepositoryProvider` เท่าเดิม จึงไม่มีไฟล์อื่นพัง · เพิ่ม `submitClaim()` ให้ `ClaimController` ใช้ยื่นสินไหม (POST) — นี่คือบทพิสูจน์ของการวางชั้น Day 1: "เปลี่ยนแหล่งข้อมูลทั้งแอปโดยแก้ไฟล์เดียว"

### ขั้นที่ 7 — Controller: AsyncNotifier + Caching + Refresh (policy_list_controller.dart)

จุดเปลี่ยนจาก Day 1: provider ตัวที่ (3) เปลี่ยนจาก **ฟังก์ชัน `FutureProvider`** เป็น **คลาส `AsyncNotifier` ชื่อ `PolicyList`** เพื่อเพิ่มเมธอด `refresh()` — แต่ตั้งชื่อคลาสให้ generator สร้าง provider ชื่อ `policyListProvider` เท่าเดิม UI จึงไม่ต้องแก้จุด watch เลย

```dart
// lib/features/policy/presentation/controllers/policy_list_controller.dart
import 'dart:async';

import 'package:flutter_riverpod/flutter_riverpod.dart';
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

// 3) ดึงรายการกรมธรรม์ + Caching (keepAlive 5 นาที) + Pull-to-refresh
//    เปลี่ยนจาก FutureProvider (Day 1) เป็น AsyncNotifier เพื่อรองรับ refresh()
//    ชื่อ provider ที่ generate ยังเป็น `policyListProvider` เหมือนเดิม → UI ไม่ต้องแก้
@riverpod
class PolicyList extends _$PolicyList {
  @override
  Future<List<Policy>> build() async {
    // เก็บ cache 5 นาที ไม่ต้องโหลดซ้ำเมื่อสลับแท็บกลับมา
    final link = ref.keepAlive();
    final timer = Timer(const Duration(minutes: 5), link.close);
    ref.onDispose(timer.cancel);

    return ref.watch(policyRepositoryProvider).fetchPolicies();
  }

  /// Pull-to-refresh: บังคับดึงใหม่ทันที (กันพังด้วย AsyncValue.guard)
  Future<void> refresh() async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(
      () => ref.read(policyRepositoryProvider).fetchPolicies(),
    );
  }
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

> 🔗 **การเชื่อมโยง / จุดสังเกต 3 จุด:**
>
> 1. **Caching:** `ref.keepAlive()` คืน link ที่ทำให้ provider ไม่ถูก dispose เมื่อไม่มีใคร watch — เราตั้ง `Timer` 5 นาทีค่อย `link.close` และผูก `ref.onDispose(timer.cancel)` กัน timer ค้าง → สลับแท็บไปมา **ไม่ยิง API ซ้ำ** ภายใน 5 นาที
> 2. **refresh() ต่างจาก invalidate:** refresh ตั้ง loading เองแล้ว `AsyncValue.guard` จับ error ให้กลายเป็น `AsyncError` (UI ไม่ crash) — เหมาะกับ Pull-to-refresh · ส่วน `ref.invalidate(...)` ใช้จากภายนอก เช่นใน `ClaimController` หลังยื่นสินไหมสำเร็จ
> 3. **`valueOrNull`** (เดิม Day 1 ใช้ `.value`) — ตั้งแต่ Riverpod 2.6 แนะนำ `valueOrNull` เพราะ `.value` บน `AsyncError` จะ throw ส่วน `valueOrNull` คืน null เฉย ๆ ปลอดภัยกว่าใน provider ที่คำนวณต่อยอด

### ขั้นที่ 8 — หน้ารายการกรมธรรม์: Pull-to-refresh + Skeleton + Error Retry (policy_list_page.dart)

โครงหน้า (search/chip/แถบเบี้ยรวม/การ์ด) เหมือน Day 1 ทุกจุด — ที่เพิ่มมี 3 อย่าง: `RefreshIndicator` ครอบ ListView, สาขา `loading:` เปลี่ยนจาก spinner เป็น **Skeletonizer + PolicyFake**, และสาขา `error:` เปลี่ยนเป็น `_ErrorBox` พร้อมปุ่มลองใหม่

```dart
// lib/features/policy/presentation/pages/policy_list_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:skeletonizer/skeletonizer.dart';

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
          // Pull-to-refresh: ลากลงเพื่อโหลดใหม่จาก API
          child: RefreshIndicator(
            onRefresh: () => ref.read(policyListProvider.notifier).refresh(),
            child: ListView(
              physics: const AlwaysScrollableScrollPhysics(),
              padding: const EdgeInsets.fromLTRB(16, 16, 16, 16),
              children: [
                // ช่องค้นหา
                TextField(
                  onChanged: (v) =>
                      ref.read(policySearchProvider.notifier).update(v),
                  decoration: const InputDecoration(
                    prefixIcon: HugeIcon(
                        icon: HugeIcons.strokeRoundedSearch01,
                        color: AppColors.textSubtle),
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
                  padding:
                      const EdgeInsets.symmetric(horizontal: 14, vertical: 12),
                  decoration: BoxDecoration(
                    color: AppColors.infoBg,
                    borderRadius: BorderRadius.circular(AppSpacing.rSm),
                  ),
                  child: Row(
                    children: [
                      const HugeIcon(
                          icon: HugeIcons.strokeRoundedWallet01,
                          size: 18,
                          color: AppColors.infoText),
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
                // เนื้อหา: loading (skeleton) / error / data
                ...asyncPolicies.when(
                  // Skeleton Loading — แสดงโครงการ์ดปลอมระหว่างรอ API
                  loading: () => List.generate(
                    4,
                    (_) => Padding(
                      padding: const EdgeInsets.only(bottom: 10),
                      child: Skeletonizer(
                        child: _PolicyCard(policy: PolicyFake.fake()),
                      ),
                    ),
                  ),
                  error: (e, _) => [_ErrorBox(message: '$e', ref: ref)],
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
        ),
      ],
    );
  }
}

/// กล่องแสดง error + ปุ่มลองใหม่
class _ErrorBox extends StatelessWidget {
  final String message;
  final WidgetRef ref;
  const _ErrorBox({required this.message, required this.ref});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.only(top: 30),
      child: Column(
        children: [
          const HugeIcon(
              icon: HugeIcons.strokeRoundedAlert02,
              size: 40,
              color: AppColors.dangerFg),
          const SizedBox(height: 12),
          Text(message,
              textAlign: TextAlign.center, style: AppType.body),
          const SizedBox(height: 12),
          OutlinedButton(
            onPressed: () => ref.read(policyListProvider.notifier).refresh(),
            child: const Text('ลองใหม่'),
          ),
        ],
      ),
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
            child: HugeIcon(
                icon: HugeIcons.strokeRoundedShield01, color: iconColor),
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
          const HugeIcon(
              icon: HugeIcons.strokeRoundedArrowRight01,
              color: AppColors.textSubtle),
        ],
      ),
    );
  }
}
```

> 🔗 **การเชื่อมโยง / จุดสังเกต:**
>
> - `RefreshIndicator.onRefresh` เรียก `refresh()` ของ AsyncNotifier — และต้องใส่ `physics: AlwaysScrollableScrollPhysics()` มิฉะนั้นตอนรายการสั้นจนไม่ scroll จะ "ลากลง" ไม่ได้
> - Skeleton ใช้ `_PolicyCard` **ตัวเดียวกับของจริง** โดยป้อน `PolicyFake.fake()` — skeleton จึงหน้าตาตรงกับ UI จริงเสมอ (ข้อได้เปรียบของ skeletonizer เหนือการวาดโครงเอง)
> - `_ErrorBox` แสดง `'$e'` ซึ่งได้ข้อความไทยจาก `ApiException.toString()` — เส้นทาง error ไหลจาก `mapDioError` → repository throw → `AsyncValue.guard`/build จับ → UI แสดง โดย**ไม่มี try/catch ใน UI เลย**

### ขั้นที่ 9 — ฟีเจอร์สินไหมครบวงจร (claim/) 🆕

#### 9.1 `claim_record.dart` — โมเดล + fromJson แบบเขียนมือ (ชั้น Domain)

จงใจเขียน `fromJson` มือ (ไม่ใช้ freezed) เพื่อให้เห็นเปรียบเทียบกับ `Policy` ว่า code generation ช่วยลดอะไร

```dart
// lib/features/claim/domain/claim_record.dart

/// สถานะคำขอสินไหม
enum ClaimStatus { submitted, reviewing, approved, rejected }

extension ClaimStatusLabel on ClaimStatus {
  String get label {
    switch (this) {
      case ClaimStatus.submitted:
        return 'ยื่นแล้ว';
      case ClaimStatus.reviewing:
        return 'กำลังพิจารณา';
      case ClaimStatus.approved:
        return 'อนุมัติ';
      case ClaimStatus.rejected:
        return 'ไม่อนุมัติ';
    }
  }
}

/// รายการประวัติการยื่นสินไหมหนึ่งรายการ (ชั้น Domain)
class ClaimRecord {
  final String claimRef;
  final String policyNumber;
  final double amount;
  final ClaimStatus status;
  final DateTime submittedAt;

  const ClaimRecord({
    required this.claimRef,
    required this.policyNumber,
    required this.amount,
    required this.status,
    required this.submittedAt,
  });

  /// แปลงจาก JSON ที่ได้จาก API (GET /claims)
  factory ClaimRecord.fromJson(Map<String, dynamic> json) => ClaimRecord(
        claimRef: json['claimRef'] as String,
        policyNumber: json['policyNumber'] as String,
        amount: (json['amount'] as num).toDouble(),
        status: ClaimStatus.values.byName(json['status'] as String),
        submittedAt: DateTime.parse(json['submittedAt'] as String),
      );
}
```

#### 9.2 `claim_repository.dart` — GET /claims + provider 2 ตัว (ชั้น Data)

```dart
// lib/features/claim/data/claim_repository.dart
import 'package:dio/dio.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

import '../../../core/network/api_exception.dart';
import '../../../core/network/dio_client.dart';
import '../domain/claim_record.dart';

part 'claim_repository.g.dart';

/// Repository ประวัติสินไหม — เรียก API จริงผ่าน Dio
class ClaimRepository {
  final Dio _dio;
  ClaimRepository(this._dio);

  /// ดึงประวัติสินไหมทั้งหมด — GET /claims
  Future<List<ClaimRecord>> fetchClaims() async {
    try {
      final response = await _dio.get<List<dynamic>>('/claims');
      final data = response.data ?? <dynamic>[];
      return data
          .map((json) => ClaimRecord.fromJson(json as Map<String, dynamic>))
          .toList();
    } on DioException catch (e) {
      throw mapDioError(e);
    }
  }
}

@riverpod
ClaimRepository claimRepository(ClaimRepositoryRef ref) =>
    ClaimRepository(ref.watch(dioClientProvider));

/// รายการสินไหม (FutureProvider) — UI watch ตัวนี้
@riverpod
Future<List<ClaimRecord>> claimList(ClaimListRef ref) async {
  return ref.watch(claimRepositoryProvider).fetchClaims();
}
```

#### 9.3 `claim_controller.dart` — Mutation ยื่นสินไหม (ชั้น Presentation)

```dart
// lib/features/claim/presentation/controllers/claim_controller.dart
// lib/features/claim/presentation/controllers/claim_controller.dart
import 'dart:async';

import 'package:riverpod_annotation/riverpod_annotation.dart';

import '../../../policy/data/policy_repository.dart';
import '../../../policy/presentation/controllers/policy_list_controller.dart';
import '../../data/claim_repository.dart';

part 'claim_controller.g.dart';

/// Controller สำหรับ "ยื่นคำขอสินไหม" (Mutation)
///
/// state เริ่มเป็น null (ยังไม่ยื่น), เป็น AsyncData(claimRef) เมื่อสำเร็จ,
/// เป็น AsyncError เมื่อพลาด — UI ใช้สถานะนี้แสดง loading/ผลลัพธ์/error
@riverpod
class ClaimController extends _$ClaimController {
  @override
  FutureOr<String?> build() => null;

  Future<void> submit({
    required String policyId,
    required double amount,
    required String reason,
  }) async {
    state = const AsyncValue.loading();
    final result = await AsyncValue.guard(() {
      return ref
          .read(policyRepositoryProvider)
          .submitClaim(policyId: policyId, amount: amount, reason: reason);
    });

    // provider นี้อาจถูก dispose ระหว่างรอ (เช่น ปิด bottom sheet ก่อนยื่นเสร็จ)
    // ต้องเช็ค ref.mounted ก่อนใช้ ref/state ต่อ ไม่งั้นจะเจอ UnmountedRefException
    if (!ref.mounted) return;

    state = result;
    if (result.hasValue) {
      // ยื่นสำเร็จ → สั่งให้รายการสินไหม + กรมธรรม์โหลดใหม่
      ref.invalidate(claimListProvider);
      ref.invalidate(policyListProvider);
    }
  }
}
```

> 🔗 **การเชื่อมโยง:** ควรสังเกตว่า claim feature เรียก `submitClaim()` ของ **policyRepository** (เพราะ endpoint คือ `/policies/:id/claims` — เจ้าของทรัพยากรคือ policy) แต่ **invalidate สอง provider ข้ามฟีเจอร์**: `claimListProvider` (ให้ประวัติสินไหมขึ้นรายการใหม่) และ `policyListProvider` (ข้อมูลกรมธรรม์อาจเปลี่ยน) — นี่คือแพตเทิร์น "Mutation แล้วทำให้ cache stale" ที่ตรงกับ Module 2

#### 9.4 `file_claim_sheet.dart` — Bottom Sheet ยื่นสินไหม (เชื่อม API จริงแล้ว)

เปลี่ยนจาก Day 1 ที่สุ่มเลข CLM ปลอม → ยื่นจริงผ่าน `ClaimController` และเปลี่ยนจาก `StatefulWidget` เป็น **`ConsumerStatefulWidget`** เพราะต้องใช้ `ref`

```dart
// lib/features/claim/presentation/widgets/file_claim_sheet.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_spacing.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_text_field.dart';
import '../controllers/claim_controller.dart';

/// แสดง Modal Bottom Sheet "ยื่นสินไหม"
/// เรียกจากหน้าแรก, รายละเอียดกรมธรรม์ และหน้าสินไหม
/// [policyId] = กรมธรรม์ที่จะยื่น (จำเป็นต่อการยื่นผ่าน API), [policyName] = ชื่อแสดงผล
Future<void> showFileClaimSheet(
  BuildContext context, {
  String? policyId,
  String? policyName,
}) {
  return showModalBottomSheet<void>(
    context: context,
    isScrollControlled: true,
    backgroundColor: Colors.transparent,
    builder: (_) => FileClaimSheet(policyId: policyId, policyName: policyName),
  );
}

class FileClaimSheet extends ConsumerStatefulWidget {
  final String? policyId;
  final String? policyName;
  const FileClaimSheet({super.key, this.policyId, this.policyName});

  @override
  ConsumerState<FileClaimSheet> createState() => _FileClaimSheetState();
}

class _FileClaimSheetState extends ConsumerState<FileClaimSheet> {
  final _amount = TextEditingController();
  final _reason = TextEditingController();

  bool _submitting = false;

  // เลขอ้างอิงหลังยื่นสำเร็จ — ถ้ายังเป็น null แสดงฟอร์ม
  String? _claimRef;

  @override
  void dispose() {
    _amount.dispose();
    _reason.dispose();
    super.dispose();
  }

  /// ยื่นสินไหมจริงผ่าน API (POST /policies/:id/claims) ด้วย ClaimController
  Future<void> _submit() async {
    final amount = double.tryParse(_amount.text.trim()) ?? 0;
    if (amount <= 0) {
      _toast('กรุณากรอกจำนวนเงินให้มากกว่า 0');
      return;
    }
    setState(() => _submitting = true);

    // Day 2: ถ้าเปิดจากหน้าสินไหมรวม (ไม่ระบุกรมธรรม์) ใช้ค่าเริ่มต้น P001 เพื่อสาธิต
    // ผลลัพธ์ (สำเร็จ/พลาด) จะถูกจัดการผ่าน ref.listen ใน build() แทนการอ่าน state ตรงนี้
    // เพราะ claimControllerProvider เป็น autoDispose — ถ้าไม่มีใคร watch/listen อยู่
    // มันอาจถูก dispose ไปก่อนที่เราจะอ่านผลลัพธ์ทัน
    await ref
        .read(claimControllerProvider.notifier)
        .submit(
          policyId: widget.policyId ?? 'P001',
          amount: amount,
          reason: _reason.text.trim(),
        );
  }

  void _toast(String msg) {
    ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(msg)));
  }

  @override
  Widget build(BuildContext context) {
    // ต้อง listen provider นี้ระหว่างที่ sheet ยังเปิดอยู่ ไม่งั้น autoDispose
    // จะเคลียร์ผลลัพธ์ทิ้งก่อนที่ UI จะทันแสดงหน้า success
    ref.listen<AsyncValue<String?>>(claimControllerProvider, (previous, next) {
      next.whenOrNull(
        data: (claimRef) {
          if (claimRef == null) return;
          setState(() {
            _submitting = false;
            _claimRef = claimRef;
          });
        },
        error: (e, _) {
          setState(() => _submitting = false);
          _toast('ยื่นไม่สำเร็จ: $e');
        },
      );
    });

    final media = MediaQuery.of(context);
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
        Text(
          widget.policyName!,
          style: AppType.caption.copyWith(color: AppColors.primary),
        ),
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
        label: _submitting ? 'กำลังยื่น...' : 'ยืนยันการยื่น',
        onPressed: _submitting ? null : _submit,
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
        child: Text(
          'เลขอ้างอิงการยื่น',
          style: AppType.caption.copyWith(color: AppColors.textSubtle),
        ),
      ),
      const SizedBox(height: 10),
      Center(
        child: Container(
          padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
          decoration: BoxDecoration(
            color: AppColors.background,
            borderRadius: BorderRadius.circular(AppSpacing.rMd),
          ),
          child: Text(
            ref,
            style: AppType.h2.copyWith(color: AppColors.primary),
          ),
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

> 🔗 **การเชื่อมโยง / จุดสังเกต:**
>
> - Sheet ยังถูกเรียกจาก 3 ที่เหมือน Day 1 แต่ signature เพิ่ม `policyId` — `PolicyDetailPage` ส่งครบ (`policyId: policy.id, policyName: policy.planName`) ส่วนหน้า Home/Claims ที่ไม่รู้กรมธรรม์ใช้ค่า default `'P001'` เพื่อสาธิต
> - ปุ่มยืนยันกันกดซ้ำ: ระหว่างส่ง `_submitting = true` → `onPressed: null` → `AppButton.primary` กลายเป็นสีเทา disabled อัตโนมัติ (พฤติกรรมที่ออกแบบไว้ตั้งแต่ Day 1)
> - หลัง `submit()` เสร็จ อ่านผลด้วย `ref.read(claimControllerProvider)` แล้ว `result.when(...)` — สำเร็จสลับเป็นหน้าเลขอ้างอิง (เลขจริงจาก API), พลาดเด้ง SnackBar ข้อความไทยจาก `ApiException`
> - validation ขั้นต้น (จำนวนเงิน > 0) ทำที่ UI ก่อนยิง API — ตรงกับความท้าทายเสริมของ Module 1

#### 9.5 `claims_page.dart` — หน้าสินไหมดึงจาก API จริง + Skeleton

เปลี่ยนจาก `StatelessWidget` + mock list (Day 1) → `ConsumerWidget` ที่ watch `claimListProvider` · ตัวเลขสรุป ("คำขอทั้งหมด/อนุมัติแล้ว") **คำนวณจากข้อมูลจริง** แทนตัวเลข hardcode

```dart
// lib/features/claim/presentation/pages/claims_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:skeletonizer/skeletonizer.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/utils/date_format.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_card.dart';
import '../../../../core/widgets/gradient_header.dart';
import '../../../../core/widgets/section_label.dart';
import '../../../../core/widgets/status_badge.dart';
import '../../data/claim_repository.dart';
import '../../domain/claim_record.dart';
import '../widgets/file_claim_sheet.dart';

/// แปลงสถานะสินไหม → โทนสี + ไอคอน (กฎการแสดงผลรวมไว้ที่เดียว)
BadgeTone claimTone(ClaimStatus s) => switch (s) {
      ClaimStatus.approved => BadgeTone.success,
      ClaimStatus.reviewing => BadgeTone.pending,
      ClaimStatus.submitted => BadgeTone.info,
      ClaimStatus.rejected => BadgeTone.danger,
    };

AppIconData claimIcon(ClaimStatus s) => switch (s) {
      ClaimStatus.approved => HugeIcons.strokeRoundedCheckmarkCircle02,
      ClaimStatus.reviewing => HugeIcons.strokeRoundedClock01,
      ClaimStatus.submitted => HugeIcons.strokeRoundedNoteAdd,
      ClaimStatus.rejected => HugeIcons.strokeRoundedCancelCircle,
    };

/// หน้าสินไหม — สรุป + ประวัติการยื่น (Day 2: ดึงจาก API จริง)
class ClaimsPage extends ConsumerWidget {
  const ClaimsPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final asyncClaims = ref.watch(claimListProvider);
    final claims = asyncClaims.valueOrNull ?? const <ClaimRecord>[];

    final approvedSum = claims
        .where((c) => c.status == ClaimStatus.approved)
        .fold<double>(0, (s, c) => s + c.amount);

    return Column(
      children: [
        GradientHeader(
          title: 'สินไหม',
          subtitle: 'ยื่นและติดตามสถานะคำขอ',
          trailing: const CircleHeaderButton(
              icon: HugeIcons.strokeRoundedNotification02),
        ),
        Expanded(
          child: RefreshIndicator(
            onRefresh: () async => ref.invalidate(claimListProvider),
            child: ListView(
              physics: const AlwaysScrollableScrollPhysics(),
              padding: const EdgeInsets.fromLTRB(16, 16, 16, 16),
              children: [
                Row(
                  children: [
                    Expanded(
                        child: _Stat(
                            label: 'คำขอทั้งหมด',
                            value: '${claims.length} รายการ')),
                    const SizedBox(width: 12),
                    Expanded(
                        child: _Stat(
                            label: 'อนุมัติแล้ว',
                            value: '฿${money(approvedSum)}',
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
                ...asyncClaims.when(
                  loading: () => List.generate(
                    3,
                    (_) => Padding(
                      padding: const EdgeInsets.only(bottom: 10),
                      child: Skeletonizer(child: _ClaimCard(_fakeClaim)),
                    ),
                  ),
                  error: (e, _) => [
                    Padding(
                      padding: const EdgeInsets.only(top: 24),
                      child: Center(child: Text('โหลดสินไหมไม่สำเร็จ: $e')),
                    )
                  ],
                  data: (list) => list.isEmpty
                      ? [
                          const Padding(
                            padding: EdgeInsets.only(top: 30),
                            child: Center(child: Text('ยังไม่มีประวัติการยื่น')),
                          )
                        ]
                      : list
                          .map((c) => Padding(
                                padding: const EdgeInsets.only(bottom: 10),
                                child: _ClaimCard(c),
                              ))
                          .toList(),
                ),
              ],
            ),
          ),
        ),
      ],
    );
  }
}

final _fakeClaim = ClaimRecord(
  claimRef: 'CLM-000000',
  policyNumber: 'BLA-2569-0000',
  amount: 0,
  status: ClaimStatus.approved,
  submittedAt: DateTime(2569),
);

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
  final ClaimRecord c;
  const _ClaimCard(this.c);

  @override
  Widget build(BuildContext context) {
    final tone = claimTone(c.status);
    return AppCard(
      child: Column(
        children: [
          Row(
            children: [
              Container(
                width: 40,
                height: 40,
                decoration: BoxDecoration(
                    color: _bg(tone),
                    borderRadius: BorderRadius.circular(10)),
                child:
                    HugeIcon(icon: claimIcon(c.status), size: 20, color: _fg(tone)),
              ),
              const SizedBox(width: 12),
              Expanded(
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text('คำขอสินไหม', style: AppType.bodyStrong),
                    const SizedBox(height: 2),
                    Text(c.policyNumber, style: AppType.caption),
                  ],
                ),
              ),
              StatusBadge(label: c.status.label, tone: tone),
            ],
          ),
          const Divider(height: 20),
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              Text('${c.claimRef} · ${thaiDate(c.submittedAt)}',
                  style: AppType.caption),
              Text('฿${money(c.amount)}', style: AppType.h3),
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
```

> 🔗 **การเชื่อมโยง:** Pull-to-refresh หน้านี้ใช้ `ref.invalidate(claimListProvider)` (ไม่ใช่ refresh() แบบหน้า policy) เพราะ `claimList` เป็น FutureProvider ธรรมดาไม่มีเมธอด — เห็นทั้งสองแพตเทิร์นเทียบกันในแอปเดียว · ยื่นสินไหมจาก sheet สำเร็จ → `ClaimController` invalidate `claimListProvider` → หน้านี้โหลดรายการใหม่**อัตโนมัติ**โดยไม่ต้องส่ง callback ใด ๆ ข้ามหน้า

#### 9.6 จุดแก้ใน `policy_detail_page.dart` — ส่ง policyId เข้า sheet

ไฟล์นี้เหมือน Day 1 ทั้งไฟล์ ยกเว้นปุ่ม "ยื่นสินไหม" ที่เพิ่ม `policyId`:

```dart
// lib/features/policy/presentation/pages/policy_detail_page.dart (จุดที่แก้)
              Expanded(
                child: AppButton.primary(
                    label: 'ยื่นสินไหม',
                    icon: HugeIcons.strokeRoundedNoteAdd,
                    onPressed: () => showFileClaimSheet(context,
                        policyId: policy.id, policyName: policy.planName)),
              ),
```

> 🔗 ยื่นจากหน้า detail จึงผูกกับกรมธรรม์ฉบับนั้นจริง ๆ (`POST /policies/${policy.id}/claims`) ต่างจากปุ่มลัดหน้า Home/Claims ที่ fallback เป็น `P001`

### ขั้นที่ 10 — ฟีเจอร์ชำระเบี้ย: Pagination / Infinite Scroll (payment/) 🆕

#### 10.1 `payment.dart` — โมเดล (ชั้น Domain)

```dart
// lib/features/payment/domain/payment.dart

/// รายการประวัติการชำระเบี้ยหนึ่งรายการ (ใช้สาธิต Lazy Loading / Pagination)
class Payment {
  final String id;
  final String title;
  final double amount;
  final DateTime paidAt;

  const Payment({
    required this.id,
    required this.title,
    required this.amount,
    required this.paidAt,
  });

  /// แปลงจาก JSON ที่ได้จาก API (GET /payments)
  factory Payment.fromJson(Map<String, dynamic> json) => Payment(
        id: json['id'] as String,
        title: json['title'] as String,
        amount: (json['amount'] as num).toDouble(),
        paidAt: DateTime.parse(json['paidAt'] as String),
      );
}
```

#### 10.2 `payment_repository.dart` — endpoint แบบแบ่งหน้า (ชั้น Data)

```dart
// lib/features/payment/data/payment_repository.dart
import 'package:dio/dio.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

import '../../../core/network/api_exception.dart';
import '../../../core/network/dio_client.dart';
import '../domain/payment.dart';

part 'payment_repository.g.dart';

/// Repository ประวัติการชำระเบี้ย — เรียก API จริงผ่าน Dio
///
/// endpoint แบบแบ่งหน้า: GET /payments?page=&limit=
class PaymentRepository {
  final Dio _dio;
  PaymentRepository(this._dio);

  Future<List<Payment>> fetchPayments({
    required int page,
    required int limit,
  }) async {
    try {
      final response = await _dio.get<List<dynamic>>(
        '/payments',
        queryParameters: {'page': page, 'limit': limit},
      );
      final data = response.data ?? <dynamic>[];
      return data
          .map((json) => Payment.fromJson(json as Map<String, dynamic>))
          .toList();
    } on DioException catch (e) {
      throw mapDioError(e);
    }
  }
}

@riverpod
PaymentRepository paymentRepository(PaymentRepositoryRef ref) {
  return PaymentRepository(ref.watch(dioClientProvider));
}
```

#### 10.3 `payment_history_controller.dart` — Pagination Controller (ชั้น Presentation)

```dart
// lib/features/payment/presentation/controllers/payment_history_controller.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

import '../../data/payment_repository.dart';
import '../../domain/payment.dart';

part 'payment_history_controller.g.dart';

/// Controller โหลดประวัติการชำระเบี้ยแบบทีละหน้า (Lazy Loading / Pagination)
///
/// - `build()` โหลดหน้าแรก
/// - `loadMore()` โหลดหน้าถัดไปเมื่อผู้ใช้เลื่อนใกล้ท้าย list
@riverpod
class PaymentHistoryController extends _$PaymentHistoryController {
  static const int _limit = 20;

  int _page = 1;
  bool _hasMore = true;
  bool _isLoadingMore = false;
  final List<Payment> _items = [];

  /// ยังมีหน้าถัดไปให้โหลดอีกหรือไม่ (UI ใช้ตัดสินใจแสดง spinner ท้าย list)
  bool get hasMore => _hasMore;

  @override
  Future<List<Payment>> build() async {
    return _fetchPage(reset: true);
  }

  Future<List<Payment>> _fetchPage({bool reset = false}) async {
    if (reset) {
      _page = 1;
      _hasMore = true;
      _items.clear();
    }
    final repo = ref.read(paymentRepositoryProvider);
    final pageItems = await repo.fetchPayments(page: _page, limit: _limit);
    _hasMore = pageItems.length == _limit;
    _items.addAll(pageItems);
    return List.unmodifiable(_items);
  }

  /// โหลดหน้าถัดไป (เรียกตอนเลื่อนใกล้ท้าย list)
  Future<void> loadMore() async {
    if (!_hasMore || _isLoadingMore || state.isLoading) return;
    _isLoadingMore = true;
    _page++;
    final more = await _fetchPage();
    state = AsyncValue.data(more);
    _isLoadingMore = false;
  }

  /// รีเฟรชทั้งหมด (กลับไปหน้าแรก)
  Future<void> refresh() async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() => _fetchPage(reset: true));
  }
}
```

> 🔗 **การเชื่อมโยง / จุดสังเกต:** state ที่ UI เห็นคือ `AsyncValue<List<Payment>>` ของ "รายการสะสมทุกหน้า" — ส่วน `_page/_hasMore/_isLoadingMore` เป็น **internal state ของ controller** ที่ UI ไม่ต้องรู้ · `loadMore()` มี guard 3 ชั้นกันเรียกซ้ำซ้อน (`!_hasMore`, `_isLoadingMore`, `state.isLoading`) — สำคัญเพราะ scroll listener ยิงถี่มาก · เดา "หน้าสุดท้าย" จาก `pageItems.length == _limit` (ได้ไม่ครบ limit = หมดแล้ว)

#### 10.4 `payments_page.dart` — Infinite Scroll + Skeleton

```dart
// lib/features/payment/presentation/pages/payments_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:skeletonizer/skeletonizer.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_spacing.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/utils/date_format.dart';
import '../../../../core/widgets/app_card.dart';
import '../../../../core/widgets/gradient_header.dart';
import '../../../../core/widgets/section_label.dart';
import '../../../../core/widgets/status_badge.dart';
import '../../domain/payment.dart';
import '../controllers/payment_history_controller.dart';

/// หน้าชำระเบี้ย — บิลครบกำหนด + ช่องทางชำระ + ประวัติการชำระ
/// Day 2: ประวัติการชำระเป็น Pagination / Infinite Scroll จาก API จริง
class PaymentsPage extends ConsumerStatefulWidget {
  const PaymentsPage({super.key});

  @override
  ConsumerState<PaymentsPage> createState() => _PaymentsPageState();
}

class _PaymentsPageState extends ConsumerState<PaymentsPage> {
  final _scroll = ScrollController();

  @override
  void initState() {
    super.initState();
    _scroll.addListener(_onScroll);
  }

  @override
  void dispose() {
    _scroll.removeListener(_onScroll);
    _scroll.dispose();
    super.dispose();
  }

  // เลื่อนใกล้ท้าย list → โหลดหน้าถัดไป (Infinite Scroll)
  void _onScroll() {
    if (_scroll.position.pixels >=
        _scroll.position.maxScrollExtent - 300) {
      ref.read(paymentHistoryControllerProvider.notifier).loadMore();
    }
  }

  @override
  Widget build(BuildContext context) {
    final async = ref.watch(paymentHistoryControllerProvider);
    final hasMore = ref.read(paymentHistoryControllerProvider.notifier).hasMore;

    return Column(
      children: [
        const GradientHeader(
          title: 'ชำระเบี้ย',
          subtitle: 'จ่ายเบี้ยและดูประวัติ',
        ),
        Expanded(
          child: RefreshIndicator(
            onRefresh: () =>
                ref.read(paymentHistoryControllerProvider.notifier).refresh(),
            child: ListView(
              controller: _scroll,
              physics: const AlwaysScrollableScrollPhysics(),
              padding: const EdgeInsets.fromLTRB(16, 16, 16, 16),
              children: [
                _DueCard(),
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
                    trailing: const HugeIcon(
                        icon: HugeIcons.strokeRoundedArrowRight01,
                        color: AppColors.textSubtle)),
                const SizedBox(height: 10),
                DottedAddBox(label: 'เพิ่มช่องทางชำระเงิน', onTap: () {}),
                const SizedBox(height: 8),
                const SectionLabel('ประวัติการชำระ'),
                // ประวัติ: loading (skeleton) / error / data + ตัวโหลดท้าย list
                ...async.when(
                  loading: () => List.generate(
                    5,
                    (_) => Padding(
                      padding: const EdgeInsets.only(bottom: 10),
                      child: Skeletonizer(child: _PaymentTile(_fakePayment)),
                    ),
                  ),
                  error: (e, _) => [
                    Padding(
                        padding: const EdgeInsets.only(top: 24),
                        child: Center(child: Text('โหลดประวัติไม่สำเร็จ: $e')))
                  ],
                  data: (list) => [
                    ...list.map((p) => Padding(
                        padding: const EdgeInsets.only(bottom: 10),
                        child: _PaymentTile(p))),
                    if (hasMore)
                      const Padding(
                        padding: EdgeInsets.symmetric(vertical: 12),
                        child: Center(
                            child: SizedBox(
                                width: 24,
                                height: 24,
                                child:
                                    CircularProgressIndicator(strokeWidth: 2))),
                      ),
                  ],
                ),
              ],
            ),
          ),
        ),
      ],
    );
  }
}

final _fakePayment = Payment(
  id: 'PMT-0000',
  title: 'ชำระเบี้ยงวดที่ 0',
  amount: 0,
  paidAt: DateTime(2569),
);

class _DueCard extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(18),
      decoration: BoxDecoration(
        gradient: AppColors.primaryGradient,
        borderRadius: BorderRadius.circular(AppSpacing.rLg),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Container(
            padding:
                const EdgeInsets.symmetric(horizontal: 10, vertical: 4),
            decoration: BoxDecoration(
                color: Colors.white24,
                borderRadius: BorderRadius.circular(AppSpacing.rPill)),
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
          Text('฿50,000', style: AppType.display.copyWith(color: Colors.white)),
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
              icon: const HugeIcon(
                  icon: HugeIcons.strokeRoundedCreditCard, size: 18),
              label: const Text('จ่ายตอนนี้'),
            ),
          ),
        ],
      ),
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
            const HugeIcon(
                icon: HugeIcons.strokeRoundedAdd01,
                size: 18,
                color: AppColors.textMedium),
            const SizedBox(width: 8),
            Text(label, style: AppType.label),
          ],
        ),
      ),
    );
  }
}

class _PaymentTile extends StatelessWidget {
  final Payment p;
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
            child: const HugeIcon(
                icon: HugeIcons.strokeRoundedCheckmarkCircle02,
                size: 20,
                color: AppColors.successFg),
          ),
          const SizedBox(width: 12),
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(p.title, style: AppType.bodyStrong),
                Text(thaiDate(p.paidAt), style: AppType.caption),
              ],
            ),
          ),
          Text('฿${money(p.amount)}', style: AppType.h3),
        ],
      ),
    );
  }
}
```

> 🔗 **การเชื่อมโยง / จุดสังเกต:**
>
> - เปลี่ยนจาก `StatelessWidget` (Day 1) เป็น `ConsumerStatefulWidget` เพราะต้องมี `ScrollController` + lifecycle (`initState`/`dispose`) — ตรงแพตเทิร์น Module 4 Day 1
> - Infinite Scroll: listener เช็ค `pixels >= maxScrollExtent - 300` (เผื่อระยะ 300px ให้โหลดก่อนถึงจริง) → `loadMore()` → เมื่อได้ข้อมูล `state` เปลี่ยน → ListView ต่อรายการ + spinner ท้าย list แสดงเฉพาะเมื่อ `hasMore`
> - ส่วนบนของหน้า (บิลครบกำหนด/ช่องทางชำระ) ยังเป็น static UI ตาม Day 1 — เราอัปเกรดเฉพาะ "ประวัติการชำระ" เป็นของจริง แสดงการอยู่ร่วมกันของ static + dynamic content ในหน้าเดียว

### ขั้นที่ 11 — ไฟล์ generated ที่เกิดใหม่ (.freezed.dart / .g.dart)

รัน `dart run build_runner build --delete-conflicting-outputs` แล้วจะได้ไฟล์เพิ่ม 8 ไฟล์ — เปิดดูเพื่อเข้าใจว่าเครื่องสร้างอะไรให้:

**11.1 `policy.g.dart`** (จาก `json_serializable`) — แปลง JSON ↔ Policy รวมถึง map ของ enum:

```dart
// lib/features/policy/domain/policy.g.dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'policy.dart';

// **************************************************************************
// JsonSerializableGenerator
// **************************************************************************

_$PolicyImpl _$$PolicyImplFromJson(Map<String, dynamic> json) => _$PolicyImpl(
      id: json['id'] as String,
      planName: json['planName'] as String,
      policyNumber: json['policyNumber'] as String,
      premium: (json['premium'] as num).toDouble(),
      status: $enumDecode(_$PolicyStatusEnumMap, json['status']),
    );

Map<String, dynamic> _$$PolicyImplToJson(_$PolicyImpl instance) =>
    <String, dynamic>{
      'id': instance.id,
      'planName': instance.planName,
      'policyNumber': instance.policyNumber,
      'premium': instance.premium,
      'status': _$PolicyStatusEnumMap[instance.status]!,
    };

const _$PolicyStatusEnumMap = {
  PolicyStatus.active: 'active',
  PolicyStatus.lapsed: 'lapsed',
  PolicyStatus.pending: 'pending',
};
```

**11.2 `policy.freezed.dart`** (จาก `freezed`, 252 บรรทัด) — โครง immutable ทั้งหมด: mixin `_$Policy` (getter ทุก field), `$PolicyCopyWith` (copyWith แบบ type-safe), `==`/`hashCode`/`toString` — ยาวเกินกว่าจะคัดลอกทั้งไฟล์ลงโน้ต ดูเต็มได้ในโปรเจกต์ ส่วนหัวที่ควรรู้จัก:

```dart
// lib/features/policy/domain/policy.freezed.dart (ตัดมาเฉพาะส่วนหัว — ไฟล์เต็ม 252 บรรทัด)
Policy _$PolicyFromJson(Map<String, dynamic> json) {
  return _Policy.fromJson(json);
}

/// @nodoc
mixin _$Policy {
  String get id => throw _privateConstructorUsedError;
  String get planName => throw _privateConstructorUsedError; // ชื่อแผนประกัน
  String get policyNumber =>
      throw _privateConstructorUsedError; // เลขที่กรมธรรม์
  double get premium =>
      throw _privateConstructorUsedError; // เบี้ยประกัน (บาท/ปี)
  PolicyStatus get status => throw _privateConstructorUsedError;

  /// Serializes this Policy to a JSON map.
  Map<String, dynamic> toJson() => throw _privateConstructorUsedError;

  /// Create a copy of Policy
  /// with the given fields replaced by the non-null parameter values.
  @JsonKey(includeFromJson: false, includeToJson: false)
  $PolicyCopyWith<Policy> get copyWith => throw _privateConstructorUsedError;
}
```

**11.3 `.g.dart` ของ Riverpod อีก 6 ไฟล์** — แพตเทิร์นเดียวกับที่เรียนใน Day 1 ทุกไฟล์ สรุปสิ่งที่แต่ละตัว generate:

| ไฟล์ generated                      | provider ที่ได้                                  | ชนิด                                                                        |
| ----------------------------------- | ------------------------------------------------ | --------------------------------------------------------------------------- |
| `dio_client.g.dart`                 | `dioClientProvider`                              | `AutoDisposeProvider<Dio>`                                                  |
| `policy_repository.g.dart`          | `policyRepositoryProvider`                       | `AutoDisposeProvider<PolicyRepository>`                                     |
| `policy_list_controller.g.dart`     | `policyListProvider` ⭐ + search/filter/filtered | ⭐ `AutoDisposeAsyncNotifierProvider<PolicyList, List<Policy>>`             |
| `claim_repository.g.dart`           | `claimRepositoryProvider`, `claimListProvider`   | Provider + `AutoDisposeFutureProvider`                                      |
| `claim_controller.g.dart`           | `claimControllerProvider`                        | `AutoDisposeAsyncNotifierProvider<ClaimController, String?>`                |
| `payment_repository.g.dart`         | `paymentRepositoryProvider`                      | `AutoDisposeProvider<PaymentRepository>`                                    |
| `payment_history_controller.g.dart` | `paymentHistoryControllerProvider`               | `AutoDisposeAsyncNotifierProvider<PaymentHistoryController, List<Payment>>` |

⭐ จุดยืนยันสำคัญ: ใน `policy_list_controller.g.dart` จะเห็นว่า `policyListProvider` เปลี่ยนชนิดจาก `AutoDisposeFutureProvider` (Day 1) เป็น `AutoDisposeAsyncNotifierProvider<PolicyList, List<Policy>>` — **ชื่อเดิม ชนิดใหม่** ตามที่ออกแบบไว้:

```dart
// lib/features/policy/presentation/controllers/policy_list_controller.g.dart (เฉพาะส่วน PolicyList)
String _$policyListHash() => r'9c28af0604bf43bc42e8aec6486cea92ee4905ac';

/// See also [PolicyList].
@ProviderFor(PolicyList)
final policyListProvider =
    AutoDisposeAsyncNotifierProvider<PolicyList, List<Policy>>.internal(
  PolicyList.new,
  name: r'policyListProvider',
  debugGetCreateSourceHash:
      const bool.fromEnvironment('dart.vm.product') ? null : _$policyListHash,
  dependencies: null,
  allTransitiveDependencies: null,
);

typedef _$PolicyList = AutoDisposeAsyncNotifier<List<Policy>>;
```

### ขั้นที่ 12 — สร้างโค้ดและทดสอบ

```bash
# สร้างไฟล์ freezed/json/riverpod ใหม่ทั้งหมด
dart run build_runner build --delete-conflicting-outputs
flutter run
```

> ✅ **ผลลัพธ์ที่คาดหวัง:** เปิดแท็บกรมธรรม์ครั้งแรกเห็น **Skeleton การ์ดไหลแสง 4 ใบ** (อาจนานหน่อยถ้า Render กำลัง cold start — timeout ตั้งไว้ 25 วิ) แล้วแทนที่ด้วยข้อมูลจริงจาก `GET /policies` · ดึงลงรีเฟรชได้ทุกแท็บ · แท็บสินไหมโหลดจริงจาก `GET /claims` ตัวเลขสรุปคำนวณจากข้อมูล · ยื่นสินไหมจากหน้า detail แล้วได้เลขอ้างอิงจริงจาก API และแท็บสินไหมอัปเดตเอง (invalidate) · แท็บชำระเบี้ยเลื่อนลงเรื่อย ๆ โหลดหน้าถัดไปอัตโนมัติ มี spinner ท้ายรายการจนกว่าจะหมด · ปิดเน็ตแล้วรีเฟรชจะเห็นข้อความ error ภาษาไทย + ปุ่ม "ลองใหม่" — **UI ทุกหน้าไม่มี try/catch เลย**

> ⚠️ **หมายเหตุสำหรับห้องเรียน:** Mock API อยู่บน Render free tier — ครั้งแรกของวันอาจ cold start 30–60 วินาที ให้เปิดรอหรือยิง `curl https://bla-mock-api.onrender.com/v1/policies -H "Authorization: Bearer mock-token-abc123"` ปลุกก่อนสอน · ถ้า endpoint ล่มให้สลับกลับ Mock Repository ของ Day 1 โดยแก้ที่ `policyRepositoryProvider` จุดเดียว — นี่คือข้อดีของ DI ที่วางไว้

### 🔗 สรุปการเชื่อมโยงทั้งหมดของ Day 2 (เส้นทางข้อมูล 3 เส้น)

```
เส้นอ่าน (Query):
  AppConfig → dioClient(+AuthInterceptor แนบ token) → Repository(.fetchXxx + mapDioError)
    → Controller/FutureProvider (AsyncValue) → UI (.when: skeleton/error/data)

เส้นเขียน (Mutation):
  FileClaimSheet → ClaimController.submit() → PolicyRepository.submitClaim (POST)
    → สำเร็จ: invalidate(claimList, policyList) → หน้าอื่น rebuild อัตโนมัติ

เส้น error:
  DioException → mapDioError → ApiException(ข้อความไทย) → AsyncError ใน AsyncValue
    → UI แสดงข้อความ + ปุ่มลองใหม่ (ไม่มี try/catch ใน UI)
```

สามฟีเจอร์สาธิตแพตเทิร์นดึงข้อมูลคนละแบบโดยตั้งใจ เพื่อให้เทียบกันได้ในแอปเดียว: **policy** = AsyncNotifier + cache + refresh() (ครบเครื่องที่สุด), **claim** = FutureProvider เรียบง่าย + invalidate, **payment** = AsyncNotifier + internal pagination state

### 🧩 ความท้าทายเสริม (ถ้าทำเสร็จก่อนเวลา)

- เพิ่ม validation ในฟอร์มสินไหมให้ครบ: จำนวนเงินต้องไม่เกินวงเงินคุ้มครอง และ `เหตุผล` ห้ามว่าง
- ทำ provider `claimStatus` แบบ polling ทุก 10 วินาที ตาม Module 2.3 แล้วให้ badge สถานะในหน้าสินไหมอัปเดตเอง
- เพิ่ม cache แบบ keepAlive 5 นาทีให้ `claimListProvider` แบบเดียวกับ `PolicyList`
- แปลง `ClaimRecord`/`Payment` เป็น freezed แบบเดียวกับ `Policy` แล้วเทียบจำนวนบรรทัดที่หายไป

---

## 📁 โครงสร้างไฟล์สรุปวันที่ 2

เฉพาะไฟล์ที่ **เพิ่มใหม่ (🆕)** และ **แก้ไข (✏️)** จาก branch `day1` — ไฟล์อื่น (core/theme, core/widgets, auth, home, main_shell ฯลฯ) เหมือนเดิมทุกไฟล์:

```
bla_policy_companion/
├── pubspec.yaml ✏️                       ← เพิ่ม dio ^5.7.0 + skeletonizer ^2.1.3
└── lib/
    ├── core/
    │   ├── config/
    │   │   └── app_config.dart 🆕        ← baseUrl / demoToken / timeout (จุดเดียว)
    │   ├── network/ 🆕
    │   │   ├── api_exception.dart        ← ApiException + mapDioError (error ภาษาไทย)
    │   │   ├── auth_interceptor.dart     ← แนบ Bearer token / ดักจับ 401
    │   │   ├── dio_client.dart           ← Provider ของ Dio (config + interceptors)
    │   │   └── dio_client.g.dart         ← (build_runner สร้าง)
    │   └── utils/
    │       └── date_format.dart 🆕       ← thaiDate() + money()
    └── features/
        ├── policy/
        │   ├── domain/
        │   │   ├── policy.dart ✏️            ← freezed + JSON + PolicyFake.fake()
        │   │   ├── policy.freezed.dart 🆕    ← (build_runner: copyWith/==/hashCode)
        │   │   └── policy.g.dart 🆕          ← (build_runner: fromJson/toJson)
        │   ├── data/
        │   │   ├── policy_repository.dart ✏️ ← mock → Dio (GET /policies, POST claim)
        │   │   └── policy_repository.g.dart
        │   └── presentation/
        │       ├── controllers/
        │       │   ├── policy_list_controller.dart ✏️ ← PolicyList (AsyncNotifier)
        │       │   │                                     + cache 5 นาที + refresh()
        │       │   └── policy_list_controller.g.dart
        │       └── pages/
        │           ├── policy_list_page.dart ✏️   ← RefreshIndicator + Skeletonizer + _ErrorBox
        │           └── policy_detail_page.dart ✏️ ← ส่ง policyId เข้า FileClaimSheet
        ├── claim/
        │   ├── domain/
        │   │   └── claim_record.dart 🆕       ← โมเดล + fromJson เขียนมือ
        │   ├── data/
        │   │   ├── claim_repository.dart 🆕   ← GET /claims + claimListProvider
        │   │   └── claim_repository.g.dart 🆕
        │   └── presentation/
        │       ├── controllers/
        │       │   ├── claim_controller.dart 🆕 ← Mutation + invalidate 2 provider
        │       │   └── claim_controller.g.dart 🆕
        │       ├── pages/
        │       │   └── claims_page.dart ✏️      ← API จริง + skeleton + สถิติคำนวณจริง
        │       └── widgets/
        │           └── file_claim_sheet.dart ✏️ ← ยื่นจริงผ่าน ClaimController
        ├── payment/
        │   ├── domain/
        │   │   └── payment.dart 🆕            ← โมเดล + fromJson
        │   ├── data/
        │   │   ├── payment_repository.dart 🆕 ← GET /payments?page=&limit=
        │   │   └── payment_repository.g.dart 🆕
        │   └── presentation/
        │       ├── controllers/
        │       │   ├── payment_history_controller.dart 🆕 ← Pagination/loadMore/refresh
        │       │   └── payment_history_controller.g.dart 🆕
        │       └── pages/
        │           └── payments_page.dart ✏️  ← Infinite Scroll + skeleton
        └── home/presentation/pages/
            └── home_page.dart ✏️              ← แก้ const เล็กน้อย (ไม่มีผลพฤติกรรม)
```

---

## 📝 สรุปประจำวันที่ 2

| หัวข้อ                   | สิ่งที่ทำได้แล้ว                                                         |
| ------------------------ | ------------------------------------------------------------------------ |
| Module 1 — Async Data    | จัดการ loading/data/error ด้วย AsyncValue และ AsyncNotifier              |
| Module 2 — Caching       | ใช้ keepAlive, auto-dispose, polling และเข้าใจ invalidate/refresh        |
| Module 3 — DI            | ฉีด Repository/Dio เป็นลูกโซ่ผ่าน Riverpod โดยไม่พึ่ง package อื่น       |
| Module 4 — Dio           | ตั้งค่า Dio, ทำ Interceptors และ Error Mapping อย่างเป็นระบบ             |
| Module 5 — Skeleton/Lazy | ทำ Skeleton/Shimmer UI และ Lazy Loading (Pagination + Infinite Scroll)   |
| ★ Workshop Day 2         | API จริง + Caching + Pull-to-refresh + Skeleton + Claim Mutation ครบวงจร |

### ✅ ตรวจสอบความพร้อมก่อนวันพรุ่งนี้

> ให้แน่ใจว่า:
>
> - ดึงข้อมูลผ่าน Repository ที่ใช้ Dio ได้ และ Pull-to-refresh ทำงาน
> - เห็น Skeleton Loading ตอนเปิดหน้าครั้งแรก (skeletonizer ทำงานถูกต้อง)
> - เข้าใจว่า `AsyncValue.guard` ช่วยจับ error แทน try/catch อย่างไร
> - ยื่นสินไหม (mutation) แล้วรายการ invalidate ตัวเองให้โหลดใหม่ได้
> - เข้าใจ provider graph (Dio → Repository → Controller) เพื่อเตรียม override ตอนเขียน Test วันที่ 3

---

**💡 คำคมประจำวัน:**

> "การจัดการข้อมูล async ที่ดี ไม่ใช่การทำให้สำเร็จเสมอ แต่คือการเตรียมพร้อมรับมือกับความล้มเหลวอย่างมีศักดิ์ศรี — ทุก loading มีจุดจบ ทุก error มีทางออกให้ผู้ใช้"

---

_เอกสารจัดทำโดย: อาจารย์สามิตร โกยม | IT Genius Engineering Co., Ltd._
_สำหรับการอบรม: บริษัท กรุงเทพประกันชีวิต จำกัด (มหาชน)_
_หลักสูตร Advanced Flutter 2026 (Scalable Architecture & Riverpod) — วันที่ 2 จาก 3_
