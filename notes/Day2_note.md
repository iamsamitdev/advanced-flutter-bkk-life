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

| เวลา | หัวข้อ |
|------|--------|
| 09:00–09:15 | ทบทวนวันที่ 1 + ภาพรวมวันนี้ |
| 09:15–10:15 | **Module 1** Asynchronous Data Management (AsyncValue & AsyncNotifier) |
| 10:15–11:15 | **Module 2** Smart Caching & Optimization (keepAlive, Auto-dispose, Polling) |
| 11:15–12:00 | **Module 3** Dependency Injection with Riverpod |
| 12:00–13:00 | พักกลางวัน |
| 13:00–14:00 | **Module 4** Advanced API Integration ด้วย Dio (Interceptors + Error Handling) |
| 14:00–15:00 | **Module 5** Skeleton Loading, Shimmer UI & Lazy Loading |
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

| เมธอด/พร็อพเพอร์ตี้ | ความหมาย |
|----------------------|----------|
| `.when()` | จัดการครบ 3 สถานะ (loading/error/data) |
| `.valueOrNull` | ดึงค่าถ้ามี ไม่งั้นได้ `null` (ไม่ throw) |
| `.isLoading` | กำลังโหลดอยู่หรือไม่ (true/false) |
| `.hasError` | มี error หรือไม่ |
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

| สถานการณ์ | ควรใช้ |
|-----------|--------|
| ข้อมูลเปลี่ยนบ่อย/เบา เช่น แจ้งเตือน | Auto-dispose (ปริยาย) |
| ข้อมูลโหลดหนัก/ช้า เช่น รายการกรมธรรม์ | `keepAlive()` |
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

| คำสั่ง | พฤติกรรม | ใช้เมื่อ |
|--------|----------|---------|
| `ref.invalidate(provider)` | ทำเครื่องหมายว่า "เก่าแล้ว" จะดึงใหม่เมื่อมีคน watch ครั้งหน้า | ต้องการให้ refresh แบบ lazy |
| `ref.refresh(provider)` | สั่งดึงใหม่ทันที และคืนค่าใหม่กลับมา | ต้องการผลลัพธ์ใหม่เดี๋ยวนี้ |
| `ref.invalidateSelf()` | provider สั่งให้ตัวเองดึงใหม่ | ใช้ใน polling |

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

| รูปแบบ Loading | ผู้ใช้รับรู้ | เหมาะกับ |
|----------------|-------------|----------|
| Spinner กลางจอ | "กำลังโหลด แต่ไม่รู้ว่านานแค่ไหน/ได้อะไร" | งานสั้น ๆ, ปุ่มกด, dialog |
| **Skeleton Screen** | "เห็นเค้าโครงเนื้อหาแล้ว ใกล้เสร็จ" | หน้า list / รายละเอียดที่โหลดจาก API |
| Progress bar (%) | "รู้ความคืบหน้าชัดเจน" | ดาวน์โหลด/อัปโหลดที่วัด % ได้ |

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

> **โจทย์:** เปลี่ยน Mock Repository ของวันที่ 1 ให้ดึงข้อมูลผ่าน Dio (จำลอง endpoint), อัปเกรดโมเดลเป็น `freezed` + JSON, ทำ Pull-to-refresh, ทำ Caching ด้วย keepAlive, แสดง **Skeleton Loading ด้วย skeletonizer** และเพิ่มฟีเจอร์ "ยื่นคำขอสินไหม (Claim)" แบบ Mutation พร้อมจัดการ Error State

### ขั้นที่ 1 — อัปเกรดโมเดล Policy เป็น freezed + JSON

freezed สร้างคลาส immutable พร้อม `copyWith`, `==`, `toJson`/`fromJson` ให้อัตโนมัติ ลดโค้ดมือและกัน bug

```dart
// lib/features/policy/domain/policy.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'policy.freezed.dart';
part 'policy.g.dart';

enum PolicyStatus { active, lapsed, pending }

@freezed
class Policy with _$Policy {
  const factory Policy({
    required String id,
    required String planName,
    required String policyNumber,
    required double premium,
    required PolicyStatus status,
  }) = _Policy;

  // แปลงจาก JSON ที่ได้จาก API
  factory Policy.fromJson(Map<String, dynamic> json) => _$PolicyFromJson(json);
}
```

### ขั้นที่ 2 — เขียน Repository ที่เรียก API จริงผ่าน Dio

```dart
// lib/features/policy/data/policy_repository.dart
import 'package:dio/dio.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../../../core/network/dio_client.dart';
import '../../../core/network/api_exception.dart';
import '../domain/policy.dart';

part 'policy_repository.g.dart';

class PolicyRepository {
  final Dio _dio;
  PolicyRepository(this._dio);

  // ดึงรายการกรมธรรม์จาก API
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

  // ยื่นคำขอสินไหม (Mutation) — ส่ง POST แล้วคืนเลขอ้างอิง
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

// ฉีด Dio เข้า Repository
@riverpod
PolicyRepository policyRepository(PolicyRepositoryRef ref) {
  return PolicyRepository(ref.watch(dioClientProvider));
}
```

### ขั้นที่ 3 — Controller แบบ AsyncNotifier พร้อม Caching และ Refresh

```dart
// lib/features/policy/presentation/controllers/policy_list_controller.dart
import 'dart:async';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../../data/policy_repository.dart';
import '../../domain/policy.dart';

part 'policy_list_controller.g.dart';

@riverpod
class PolicyListController extends _$PolicyListController {
  @override
  Future<List<Policy>> build() async {
    // เก็บ cache ไว้ 5 นาที ไม่ต้องโหลดซ้ำเมื่อกลับเข้าหน้า
    final link = ref.keepAlive();
    final timer = Timer(const Duration(minutes: 5), link.close);
    ref.onDispose(timer.cancel);

    return ref.watch(policyRepositoryProvider).fetchPolicies();
  }

  // Pull-to-refresh: บังคับดึงใหม่ทันที
  Future<void> refresh() async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(
      () => ref.read(policyRepositoryProvider).fetchPolicies(),
    );
  }
}
```

### ขั้นที่ 4 — Controller สำหรับยื่นคำขอสินไหม (Mutation)

```dart
// lib/features/claim/presentation/controllers/claim_controller.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../../../policy/data/policy_repository.dart';
import '../../../policy/presentation/controllers/policy_list_controller.dart';

part 'claim_controller.g.dart';

@riverpod
class ClaimController extends _$ClaimController {
  // build() ของ mutation controller ไม่ดึงข้อมูล จึงเริ่มเป็น null
  @override
  FutureOr<String?> build() => null;

  Future<void> submit({
    required String policyId,
    required double amount,
    required String reason,
  }) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      final claimRef = await ref.read(policyRepositoryProvider).submitClaim(
            policyId: policyId,
            amount: amount,
            reason: reason,
          );
      // หลังยื่นสำเร็จ สั่งให้รายการกรมธรรม์โหลดใหม่ (ข้อมูลอาจเปลี่ยน)
      ref.invalidate(policyListControllerProvider);
      return claimRef;
    });
  }
}
```

### ขั้นที่ 5 — UI: Pull-to-refresh และฟอร์มยื่นสินไหม

```dart
// lib/features/policy/presentation/pages/policy_list_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../domain/policy.dart';
import '../controllers/policy_list_controller.dart';

class PolicyListPage extends ConsumerWidget {
  const PolicyListPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final asyncPolicies = ref.watch(policyListControllerProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('กรมธรรม์ของฉัน')),
      body: asyncPolicies.when(
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (error, _) => _ErrorView(
          message: '$error',
          onRetry: () =>
              ref.read(policyListControllerProvider.notifier).refresh(),
        ),
        data: (policies) => RefreshIndicator(
          // ดึงลงเพื่อรีเฟรช → เรียกเมธอด refresh ของ controller
          onRefresh: () =>
              ref.read(policyListControllerProvider.notifier).refresh(),
          child: ListView.builder(
            itemCount: policies.length,
            itemBuilder: (context, index) {
              final policy = policies[index];
              return ListTile(
                title: Text(policy.planName),
                subtitle: Text(policy.policyNumber),
                trailing: TextButton(
                  onPressed: () => _openClaimSheet(context, policy),
                  child: const Text('ยื่นสินไหม'),
                ),
              );
            },
          ),
        ),
      ),
    );
  }

  void _openClaimSheet(BuildContext context, Policy policy) {
    showModalBottomSheet<void>(
      context: context,
      isScrollControlled: true,
      builder: (_) => ClaimSheet(policy: policy),
    );
  }
}

class _ErrorView extends StatelessWidget {
  final String message;
  final VoidCallback onRetry;
  const _ErrorView({required this.message, required this.onRetry});

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          const Icon(Icons.cloud_off, size: 48),
          Padding(padding: const EdgeInsets.all(12), child: Text(message)),
          ElevatedButton(onPressed: onRetry, child: const Text('ลองอีกครั้ง')),
        ],
      ),
    );
  }
}
```

```dart
// lib/features/claim/presentation/widgets/claim_sheet.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../../policy/domain/policy.dart';
import '../controllers/claim_controller.dart';

class ClaimSheet extends ConsumerStatefulWidget {
  final Policy policy;
  const ClaimSheet({super.key, required this.policy});

  @override
  ConsumerState<ClaimSheet> createState() => _ClaimSheetState();
}

class _ClaimSheetState extends ConsumerState<ClaimSheet> {
  final _amountController = TextEditingController();
  final _reasonController = TextEditingController();

  @override
  void dispose() {
    _amountController.dispose();
    _reasonController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final claimState = ref.watch(claimControllerProvider);

    // ใช้ ref.listen ทำ side-effect: แจ้งผลสำเร็จ/ล้มเหลว แล้วปิด sheet
    ref.listen(claimControllerProvider, (previous, next) {
      next.whenOrNull(
        data: (claimRef) {
          if (claimRef != null) {
            Navigator.of(context).pop();
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text('ยื่นสินไหมสำเร็จ เลขอ้างอิง $claimRef')),
            );
          }
        },
        error: (error, _) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text('ยื่นไม่สำเร็จ: $error')),
          );
        },
      );
    });

    return Padding(
      padding: EdgeInsets.only(
        left: 16,
        right: 16,
        top: 16,
        bottom: MediaQuery.of(context).viewInsets.bottom + 16,
      ),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        crossAxisAlignment: CrossAxisAlignment.stretch,
        children: [
          Text('ยื่นสินไหม: ${widget.policy.planName}',
              style: Theme.of(context).textTheme.titleMedium),
          TextField(
            controller: _amountController,
            keyboardType: TextInputType.number,
            decoration: const InputDecoration(labelText: 'จำนวนเงิน (บาท)'),
          ),
          TextField(
            controller: _reasonController,
            decoration: const InputDecoration(labelText: 'เหตุผล/รายละเอียด'),
          ),
          const SizedBox(height: 16),
          ElevatedButton(
            // ปิดปุ่มขณะกำลังส่ง เพื่อกันกดซ้ำ
            onPressed: claimState.isLoading
                ? null
                : () => ref.read(claimControllerProvider.notifier).submit(
                      policyId: widget.policy.id,
                      amount: double.tryParse(_amountController.text) ?? 0,
                      reason: _reasonController.text,
                    ),
            child: claimState.isLoading
                ? const SizedBox(
                    height: 20,
                    width: 20,
                    child: CircularProgressIndicator(strokeWidth: 2),
                  )
                : const Text('ยืนยันการยื่น'),
          ),
        ],
      ),
    );
  }
}
```

### ขั้นที่ 6 — แทน Loading Spinner ด้วย Skeleton (skeletonizer)

อัปเกรด UI ของหน้ารายการให้แสดง Skeleton ระหว่างโหลดแทน spinner กลางจอ โดยครอบ `ListView` ด้วย `Skeletonizer` ตาม Module 5.3

```dart
// ปรับ build() ของ PolicyListPage — ส่วน data/loading รวมเป็น Skeletonizer เดียว
import 'package:skeletonizer/skeletonizer.dart';

@override
Widget build(BuildContext context, WidgetRef ref) {
  final asyncPolicies = ref.watch(policyListControllerProvider);

  return Scaffold(
    appBar: AppBar(title: const Text('กรมธรรม์ของฉัน')),
    body: asyncPolicies.when(
      // error ยังแยกจัดการเหมือนเดิม
      error: (error, _) => _ErrorView(
        message: '$error',
        onRetry: () =>
            ref.read(policyListControllerProvider.notifier).refresh(),
      ),
      // skipLoadingOnRefresh: ตอน pull-to-refresh ไม่ต้องโชว์ skeleton ทับ
      skipLoadingOnRefresh: false,
      loading: () => Skeletonizer(
        enabled: true,
        child: ListView.builder(
          itemCount: 6,
          itemBuilder: (context, index) =>
              _PolicyTile(policy: PolicyFake.fake()),
        ),
      ),
      data: (policies) => RefreshIndicator(
        onRefresh: () =>
            ref.read(policyListControllerProvider.notifier).refresh(),
        child: ListView.builder(
          itemCount: policies.length,
          itemBuilder: (context, index) =>
              _PolicyTile(policy: policies[index]),
        ),
      ),
    ),
  );
}
```

> ✅ **สังเกต:** เราใช้ `_PolicyTile` ตัวเดียวกันทั้งตอน skeleton และตอนแสดงจริง จึงไม่มีทางที่ skeleton จะ "หน้าตาไม่ตรง" กับ UI จริง — นี่คือข้อได้เปรียบของ `skeletonizer` เหนือการวาดมือ

### ขั้นที่ 7 — สร้างโค้ดและทดสอบ

```bash
# สร้างไฟล์ freezed/json/riverpod ใหม่ทั้งหมด
dart run build_runner build --delete-conflicting-outputs
flutter run
```

> ✅ **ผลลัพธ์ที่คาดหวัง:** เปิดหน้าครั้งแรกเห็น **Skeleton การ์ดไหลแสง** แทน spinner แล้วค่อยแทนที่ด้วยข้อมูลจริงเมื่อ Dio โหลดเสร็จ, ดึงลง (pull) เพื่อรีเฟรชได้, กด "ยื่นสินไหม" แล้วเปิดฟอร์ม กรอกแล้วส่ง ระหว่างส่งปุ่มแสดง spinner และกันกดซ้ำ เมื่อสำเร็จแสดง SnackBar พร้อมเลขอ้างอิงและรายการรีเฟรชอัตโนมัติ ส่วนกรณี error จะเห็นข้อความที่เป็นมิตรพร้อมปุ่มลองใหม่ — ทั้งหมดนี้ UI ไม่มี try/catch เลย เพราะ `AsyncValue` + `AsyncValue.guard` จัดการให้

> ⚠️ **หมายเหตุสำหรับห้องเรียน:** หาก endpoint จริงยังไม่พร้อม ให้ใช้บริการ mock เช่น `https://jsonplaceholder.typicode.com` หรือคง Mock Repository ของวันที่ 1 ไว้ชั่วคราว แล้วสลับ provider เพียงตัวเดียว — นี่คือข้อดีของ DI ที่เราวางไว้

### 🧩 ความท้าทายเสริม (ถ้าทำเสร็จก่อนเวลา)

- เพิ่ม `LogInterceptor` ของ Dio และดู log request/response ใน console
- ทำ provider `claimStatus` แบบ polling ทุก 10 วินาที ตาม Module 2.3
- เพิ่ม validation ในฟอร์มสินไหม (จำนวนเงินต้องมากกว่า 0 และไม่เกินเบี้ยรวม)
- ทำหน้า "ประวัติการชำระเบี้ย" แบบ Lazy Loading (Pagination + Infinite Scroll) ตาม Module 5.5 พร้อมแสดง skeleton ตอนโหลดหน้าแรก

---

## 📁 โครงสร้างไฟล์สรุปวันที่ 2

```
bla_policy_companion/lib/
├── core/
│   └── network/
│       ├── dio_client.dart           ← Provider ของ Dio
│       ├── auth_interceptor.dart     ← แนบ token / จับ 401
│       └── api_exception.dart        ← แปลง error ให้เป็นมิตร
└── features/
    ├── policy/
    │   ├── domain/policy.dart        ← freezed + JSON (+ PolicyFake.fake())
    │   ├── data/policy_repository.dart  ← เรียก API ผ่าน Dio
    │   └── presentation/
    │       ├── controllers/policy_list_controller.dart  ← AsyncNotifier + cache
    │       ├── widgets/policy_skeleton.dart             ← Skeleton/Shimmer (Module 5)
    │       └── pages/policy_list_page.dart              ← Pull-to-refresh + Skeletonizer
    ├── claim/
    │   └── presentation/
    │       ├── controllers/claim_controller.dart        ← Mutation
    │       └── widgets/claim_sheet.dart                 ← ฟอร์มยื่นสินไหม
    └── payment/   (ความท้าทายเสริม)
        ├── data/payment_repository.dart                 ← fetchPayments(page, limit)
        └── presentation/
            ├── controllers/payment_history_controller.dart  ← Lazy Loading / Pagination
            └── pages/payment_history_page.dart              ← Infinite Scroll
```

---

## 📝 สรุปประจำวันที่ 2

| หัวข้อ | สิ่งที่ทำได้แล้ว |
|--------|----------------|
| Module 1 — Async Data | จัดการ loading/data/error ด้วย AsyncValue และ AsyncNotifier |
| Module 2 — Caching | ใช้ keepAlive, auto-dispose, polling และเข้าใจ invalidate/refresh |
| Module 3 — DI | ฉีด Repository/Dio เป็นลูกโซ่ผ่าน Riverpod โดยไม่พึ่ง package อื่น |
| Module 4 — Dio | ตั้งค่า Dio, ทำ Interceptors และ Error Mapping อย่างเป็นระบบ |
| Module 5 — Skeleton/Lazy | ทำ Skeleton/Shimmer UI และ Lazy Loading (Pagination + Infinite Scroll) |
| ★ Workshop Day 2 | API จริง + Caching + Pull-to-refresh + Skeleton + Claim Mutation ครบวงจร |

### ✅ ตรวจสอบความพร้อมก่อนวันพรุ่งนี้

> ให้แน่ใจว่า:
> - ดึงข้อมูลผ่าน Repository ที่ใช้ Dio ได้ และ Pull-to-refresh ทำงาน
> - เห็น Skeleton Loading ตอนเปิดหน้าครั้งแรก (skeletonizer ทำงานถูกต้อง)
> - เข้าใจว่า `AsyncValue.guard` ช่วยจับ error แทน try/catch อย่างไร
> - ยื่นสินไหม (mutation) แล้วรายการ invalidate ตัวเองให้โหลดใหม่ได้
> - เข้าใจ provider graph (Dio → Repository → Controller) เพื่อเตรียม override ตอนเขียน Test วันที่ 3

---

**💡 คำคมประจำวัน:**

> "การจัดการข้อมูล async ที่ดี ไม่ใช่การทำให้สำเร็จเสมอ แต่คือการเตรียมพร้อมรับมือกับความล้มเหลวอย่างมีศักดิ์ศรี — ทุก loading มีจุดจบ ทุก error มีทางออกให้ผู้ใช้"

---

*เอกสารจัดทำโดย: อาจารย์สามิตร โกยม | IT Genius Engineering Co., Ltd.*
*สำหรับการอบรม: บริษัท กรุงเทพประกันชีวิต จำกัด (มหาชน)*
*หลักสูตร Advanced Flutter 2026 (Scalable Architecture & Riverpod) — วันที่ 2 จาก 3*
