# Advanced Flutter 2026 — วันที่ 3: Advanced Routing, Security & Production Readiness

**หลักสูตรอบรมเชิงปฏิบัติการ: Advanced Flutter 2026 (Scalable Architecture & Riverpod)**
**จัดอบรมให้: บริษัท กรุงเทพประกันชีวิต จำกัด (มหาชน)**
**วันที่ 3: Routing ขั้นสูง, ความปลอดภัย, การทดสอบ และเตรียมสู่ Production**
วันที่: 4 กรกฎาคม 2569 | เวลา 09:00–16:00 น. | Onsite Workshop
ผู้สอน: อ.สามิตร โกยม

---

## 🎯 วัตถุประสงค์การเรียนรู้ประจำวัน

เมื่อจบการอบรมวันที่ 3 ผู้เรียนจะสามารถ:

1. ตั้งค่า **GoRouter** ทำ Routing แบบ declarative ส่งพารามิเตอร์ และทำ Bottom Navigation 5 แท็บด้วย **StatefulShellRoute** ได้
2. ทำ **Route Guard** ที่ควบคุมด้วยสถานะจาก Riverpod (เช่น ต้อง Login ก่อนเข้าถึงหน้า) ได้
3. จัดเก็บข้อมูลความลับ (Token/Credentials) อย่างปลอดภัยด้วย **Flutter Secure Storage** และออกแบบ **Reactive Auth State**
4. ทำ **SSL Pinning** ป้องกัน MITM และ **เข้ารหัสข้อมูล (AES)** Request/Response payload พร้อมแนวทาง Obfuscation สำหรับ Release Build
5. เชื่อมต่อ **Native Platform** ผ่าน **Platform Channel (MethodChannel)** เรียก Native API บน Android (Kotlin) / iOS (Swift) ได้
6. เขียน **Unit Test** สำหรับ Providers และ Notifiers พร้อม **Mock Dependencies** ด้วย mocktail
7. **Optimize Performance** — ลด Jank/Frame Drop, จัดการ Memory และวัดผลด้วย **Flutter DevTools** พร้อม Production Checklist ก่อน Release
8. ลงมือประกอบร่างระบบ Authentication ครบวงจร (Master Workshop)

> **หมายเหตุสำคัญ:** วันนี้คือวันที่เราจะ "ประกอบร่าง" ทุกชิ้นส่วนของ 2 วันที่ผ่านมาเข้าด้วยกัน — Auth State จะกลายเป็นแหล่งความจริงเดียว (single source of truth) ที่ควบคุมทั้ง Route Guard, การแนบ Token ใน Dio และการแสดงผล UI ทั้งหมดแบบ Reactive

---

## 🧭 กำหนดการวันที่ 3 (โดยสังเขป)

| เวลา        | หัวข้อ                                                                                        |
| ----------- | --------------------------------------------------------------------------------------------- |
| 09:00–09:10 | ทบทวนวันที่ 1–2 + ภาพรวมระบบ Authentication ที่จะสร้าง                                        |
| 09:10–10:10 | **Module 1** Advanced Routing with GoRouter & Riverpod                                        |
| 10:10–11:20 | **Module 2** Application Security (Secure Storage + Reactive Auth + SSL Pinning + Encryption) |
| 11:20–12:00 | **Module 3** Platform Channel — Native Integration                                            |
| 12:00–13:00 | พักกลางวัน                                                                                    |
| 13:00–14:00 | **Module 4** Testing in Riverpod (Unit Test + Mock)                                           |
| 14:00–15:00 | **Module 5** Performance Optimization & Best Practices 2026                                   |
| 15:00–16:00 | **Master Workshop** ประกอบร่าง Auth + Route Guard + Secure Storage + SSL Pinning + Test       |

---

## 📚 Module 1: Advanced Routing with GoRouter & Riverpod

### เวลา 09:10–10:10 น.

> 💡 **หัวใจของ Module นี้:** `Navigator.push` แบบเดิมจัดการเส้นทางแบบ imperative ซึ่งคุมยากเมื่อแอปใหญ่ขึ้น GoRouter ให้เราประกาศเส้นทางทั้งหมดไว้ที่เดียวแบบ declarative และเชื่อมกับ Riverpod เพื่อ "เปลี่ยนเส้นทางอัตโนมัติตามสถานะ" ได้

---

### 1.1 ตั้งค่า GoRouter พื้นฐาน

```dart
// core/router/app_router.dart
import 'package:go_router/go_router.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../../features/policy/presentation/pages/policy_list_page.dart';
import '../../features/policy/presentation/pages/policy_detail_page.dart';
import '../../features/auth/presentation/pages/login_page.dart';

part 'app_router.g.dart';

@riverpod
GoRouter appRouter(AppRouterRef ref) {
  return GoRouter(
    initialLocation: '/policies',
    routes: [
      GoRoute(
        path: '/login',
        builder: (context, state) => const LoginPage(),
      ),
      GoRoute(
        path: '/policies',
        builder: (context, state) => const PolicyListPage(),
        routes: [
          // เส้นทางย่อย: /policies/:id (รับพารามิเตอร์)
          GoRoute(
            path: ':id',
            builder: (context, state) {
              final policyId = state.pathParameters['id']!;
              return PolicyDetailPage(policyId: policyId);
            },
          ),
        ],
      ),
    ],
  );
}
```

### 1.2 การนำทางและส่งพารามิเตอร์

```dart
// ไปยังหน้ารายละเอียดพร้อมส่ง id ผ่าน path
context.go('/policies/P001');

// แบบ push (ซ้อนหน้า กดย้อนกลับได้)
context.push('/policies/P001');

// ส่งข้อมูลเสริมผ่าน query parameters
context.go('/policies?status=active');

// ส่งออบเจกต์ผ่าน extra (ไม่ปรากฏใน URL)
context.go('/policies/P001', extra: selectedPolicy);
```

| รูปแบบการส่งค่า            | เข้าถึงด้วย                           | เหมาะกับ                       |
| -------------------------- | ------------------------------------- | ------------------------------ |
| Path parameter `:id`       | `state.pathParameters['id']`          | ID ของทรัพยากร                 |
| Query parameter `?status=` | `state.uri.queryParameters['status']` | ตัวกรอง/ตัวเลือก               |
| Extra object               | `state.extra`                         | ส่งออบเจกต์ทั้งก้อน (ภายในแอป) |

### 1.3 Route Guard ด้วย redirect + Riverpod

นี่คือจุดเชื่อม GoRouter กับ Riverpod ที่ทรงพลังที่สุด — เราใช้ `redirect` ตรวจสถานะ Auth ถ้ายังไม่ Login ให้เด้งไปหน้า Login อัตโนมัติ

```dart
@riverpod
GoRouter appRouter(AppRouterRef ref) {
  return GoRouter(
    initialLocation: '/policies',
    // refreshListenable: ให้ router คำนวณ redirect ใหม่เมื่อ auth state เปลี่ยน
    refreshListenable: ref.watch(routerRefreshProvider),
    redirect: (context, state) {
      final isLoggedIn = ref.read(authControllerProvider).valueOrNull != null;
      final goingToLogin = state.matchedLocation == '/login';

      // ยังไม่ login และไม่ได้ไปหน้า login → เด้งไป login
      if (!isLoggedIn && !goingToLogin) return '/login';
      // login แล้วแต่ยังอยู่หน้า login → เด้งเข้าแอป
      if (isLoggedIn && goingToLogin) return '/home';
      return null; // ไม่ต้อง redirect
    },
    routes: [/* ... */],
  );
}
```

> 💡 **กลไก refreshListenable:** GoRouter จะเรียก `redirect` ใหม่ก็ต่อเมื่อมี navigation event หรือ listenable แจ้งเตือน เราจึงต้องสร้างตัวเชื่อมที่แปลงการเปลี่ยนแปลงของ Riverpod provider เป็น `Listenable` ให้ router รับรู้ (โค้ดตัวเชื่อมอยู่ใน Master Workshop)

### 1.4 Bottom Navigation 5 แท็บ ด้วย StatefulShellRoute

แอป BLA Policy Companion เป็น **แอปฝั่งลูกค้า (self-service)** ที่มีหลายส่วนใช้งานคู่ขนานกัน เราจึงใช้ `StatefulShellRoute.indexedStack` ทำ Bottom Navigation 5 แท็บ จุดเด่นคือ **แต่ละแท็บมี Navigation Stack แยกของตัวเอง** (เช่น เข้าไปดูรายละเอียดกรมธรรม์ในแท็บ "กรมธรรม์" แล้วสลับไปแท็บอื่นและกลับมา หน้าเดิมยังอยู่)

```
เรียงลำดับแท็บ (ซ้าย→ขวา ตามความถี่การใช้งาน):
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│ หน้าแรก   │ กรมธรรม์  │  สินไหม   │ ชำระเบี้ย │ โปรไฟล์   │
│ /home    │/policies │ /claims  │/payments │ /profile │
└──────────┴──────────┴──────────┴──────────┴──────────┘
  Home       Policies   Claims     Payments   Profile
  (โปรไฟล์อยู่ขวาสุดเสมอ ตามแบบแผนที่ผู้ใช้คุ้นเคย — Jakob's Law)
```

**ขั้นที่ 1 — สร้าง Shell ที่มี NavigationBar (Material 3)**

```dart
// core/router/home_shell.dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

class HomeShell extends StatelessWidget {
  final StatefulNavigationShell navigationShell;
  const HomeShell({super.key, required this.navigationShell});

  void _onTap(int index) {
    // กดแท็บเดิมซ้ำ → เด้งกลับ root ของแท็บนั้น
    navigationShell.goBranch(
      index,
      initialLocation: index == navigationShell.currentIndex,
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: navigationShell, // เนื้อหาของแท็บที่เลือกอยู่
      bottomNavigationBar: NavigationBar(
        selectedIndex: navigationShell.currentIndex,
        onDestinationSelected: _onTap,
        destinations: const [
          NavigationDestination(
              icon: Icon(Icons.home_outlined),
              selectedIcon: Icon(Icons.home),
              label: 'หน้าแรก'),
          NavigationDestination(
              icon: Icon(Icons.shield_outlined),
              selectedIcon: Icon(Icons.shield),
              label: 'กรมธรรม์'),
          NavigationDestination(
              icon: Icon(Icons.assignment_outlined),
              selectedIcon: Icon(Icons.assignment),
              label: 'สินไหม'),
          NavigationDestination(
              icon: Icon(Icons.credit_card_outlined),
              selectedIcon: Icon(Icons.credit_card),
              label: 'ชำระเบี้ย'),
          NavigationDestination(
              icon: Icon(Icons.person_outline),
              selectedIcon: Icon(Icons.person),
              label: 'โปรไฟล์'),
        ],
      ),
    );
  }
}
```

**ขั้นที่ 2 — ห่อ 5 branch ด้วย StatefulShellRoute**

```dart
// core/router/app_router.dart (ส่วน routes)
import 'package:bla_policy_companion/core/router/home_shell.dart';
import 'package:bla_policy_companion/features/claim/presentation/pages/claims_page.dart';
import 'package:bla_policy_companion/features/home/presentation/pages/home_page.dart';
import 'package:bla_policy_companion/features/payment/presentation/pages/payments_page.dart';
import 'package:bla_policy_companion/features/profile/presentation/pages/profile_page.dart';
import 'package:go_router/go_router.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../../features/auth/presentation/pages/splash_page.dart';
import '../../features/auth/presentation/pages/login_page.dart';
import 'package:bla_policy_companion/features/policy/domain/policy.dart'
    show Policy;
import '../../features/policy/presentation/pages/policy_detail_page.dart';
import '../../features/policy/presentation/pages/policy_list_page.dart';

part 'app_router.g.dart';

@riverpod
GoRouter appRouter(Ref ref) {
  return GoRouter(
    initialLocation: '/splash',
    routes: [
      GoRoute(
        path: '/splash',
        builder: (context, state) => const SplashPage(),
      ),
      GoRoute(
        path: '/login',
        builder: (context, state) => const LoginPage(),
      ),
      // Bottom Navigation 5 แท็บ — แต่ละแท็บมี stack แยกของตัวเอง
      StatefulShellRoute.indexedStack(
        builder: (context, state, navigationShell) =>
            HomeShell(navigationShell: navigationShell),
        branches: [
          StatefulShellBranch(
            routes: [
              GoRoute(path: '/home', builder: (c, s) => const HomePage()),
            ],
          ),
          StatefulShellBranch(
            routes: [
              GoRoute(
                path: '/policies',
                builder: (c, s) => const PolicyListPage(),
                routes: [
                  // /policies/:id — รับ Policy ผ่าน extra (ส่งทั้งก้อนจากการ์ดในลิสต์)
                  GoRoute(
                    path: ':id',
                    builder: (c, s) =>
                        PolicyDetailPage(policy: s.extra! as Policy),
                  ),
                ],
              ),
            ],
          ),
          StatefulShellBranch(
            routes: [
              GoRoute(path: '/claims', builder: (c, s) => const ClaimsPage()),
            ],
          ),
          StatefulShellBranch(
            routes: [
              GoRoute(
                path: '/payments',
                builder: (c, s) => const PaymentsPage(),
              ),
            ],
          ),
          StatefulShellBranch(
            routes: [
              GoRoute(path: '/profile', builder: (c, s) => const ProfilePage()),
            ],
          ),
        ],
      ),
    ],
  );
}

```

> ✅ **จุดสำคัญ:** หน้า Login อยู่ **นอก** StatefulShellRoute (เป็น route ระดับบนสุด) เพราะเป็นหน้าที่ไม่ควรมี Bottom Navigation ส่วน Route Guard (`redirect`) ทำงานครอบทั้งหมด — ยังไม่ Login ก็เด้งออกไป `/login` เหมือนเดิม และปุ่ม "ออกจากระบบ" ย้ายไปอยู่ในแท็บ **โปรไฟล์** (ขวาสุด) เพื่อลดการกดผิดโดยไม่ตั้งใจ

> ⚠️ **ข้อจำกัดที่ควรจำ:** ทั้ง Material 3 และ Apple HIG แนะนำ Bottom Navigation ที่ **3–5 แท็บเท่านั้น** — 5 คือเพดานสูงสุด หากมีหมวดเพิ่มในอนาคต ให้ยุบเข้าไปใต้แท็บ "เพิ่มเติม (More)" แทนการเพิ่มแท็บที่ 6

---

## 📚 Module 2: Application Security

### เวลา 10:10–11:20 น.

> 💡 **หัวใจของ Module นี้:** Token คือกุญแจเข้าระบบ ถ้าเก็บผิดที่ (เช่น `SharedPreferences` ธรรมดา) ก็เหมือนวางกุญแจบ้านไว้หน้าประตู เราต้องเก็บใน Secure Storage ที่เข้ารหัสด้วยกลไกของ OS (Keychain บน iOS, Keystore บน Android) และทำให้สถานะการ Login ขับเคลื่อนทั้งแอปแบบ Reactive

---

### 2.1 Flutter Secure Storage — เก็บความลับอย่างปลอดภัย

```dart
// core/storage/secure_storage.dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'secure_storage.g.dart';

class SecureStorageService {
  final FlutterSecureStorage _storage;
  SecureStorageService(this._storage);

  static const _keyToken = 'auth_token';

  Future<void> saveToken(String token) =>
      _storage.write(key: _keyToken, value: token);

  Future<String?> readToken() => _storage.read(key: _keyToken);

  Future<void> deleteToken() => _storage.delete(key: _keyToken);
}

@riverpod
SecureStorageService secureStorage(SecureStorageRef ref) {
  return SecureStorageService(const FlutterSecureStorage());
}
```

| ข้อมูล                         | เก็บที่ไหน                 | เหตุผล                         |
| ------------------------------ | -------------------------- | ------------------------------ |
| Token, Credentials             | **Flutter Secure Storage** | เข้ารหัสด้วย Keychain/Keystore |
| ค่าตั้งค่าทั่วไป (theme, ภาษา) | `SharedPreferences`        | ไม่ลับ เข้าถึงเร็ว             |
| ข้อมูลชั่วคราว (cache)         | Memory / ไฟล์              | หายได้ ไม่ต้องเข้ารหัส         |

> ⚠️ **ห้ามเด็ดขาด:** อย่าเก็บ token, รหัสผ่าน หรือข้อมูลบัตรใน `SharedPreferences` เพราะเก็บเป็น plain text — บนเครื่องที่ root/jailbreak อ่านได้ทันที

### 2.2 ออกแบบ Reactive Auth State ด้วย AsyncNotifier

หัวใจของความปลอดภัยฝั่งแอปคือ "แหล่งความจริงเดียว" ของสถานะผู้ใช้ เราทำ `AuthController` เป็น `AsyncNotifier` ที่โหลดสถานะจาก Secure Storage ตอนเปิดแอป และให้เมธอด login/logout

```dart
// features/auth/presentation/controllers/auth_controller.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../../../../core/storage/secure_storage.dart';
import '../../data/auth_repository.dart';
import '../../domain/auth_user.dart';

part 'auth_controller.g.dart';

@Riverpod(keepAlive: true) // auth state ต้องอยู่ตลอดอายุแอป
class AuthController extends _$AuthController {
  // build() คืน user ปัจจุบัน (null = ยังไม่ login)
  @override
  Future<AuthUser?> build() async {
    final token = await ref.watch(secureStorageProvider).readToken();
    if (token == null) return null;
    // มี token อยู่แล้ว → ดึงข้อมูลผู้ใช้กลับมา
    return ref.watch(authRepositoryProvider).getCurrentUser(token);
  }

  Future<void> login(String username, String password) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      final result =
          await ref.read(authRepositoryProvider).login(username, password);
      // เก็บ token อย่างปลอดภัย
      await ref.read(secureStorageProvider).saveToken(result.token);
      return result.user;
    });
  }

  Future<void> logout() async {
    await ref.read(secureStorageProvider).deleteToken();
    state = const AsyncValue.data(null); // เคลียร์สถานะ → ทั้งแอป redirect ไป login
  }
}
```

```
ภาพรวม Reactive Auth:
┌─────────────────────────────────────────────────────────────┐
│  AuthController (single source of truth)                    │
│        │                                                     │
│        ├──→ GoRouter redirect  (login แล้วหรือยัง?)          │
│        ├──→ Dio AuthInterceptor (แนบ token อัตโนมัติ)        │
│        └──→ UI ทุกหน้า          (แสดงชื่อผู้ใช้ / ปุ่ม logout) │
└─────────────────────────────────────────────────────────────┘
   logout() ครั้งเดียว → ทุกส่วนตอบสนองพร้อมกันแบบ reactive
```

---

### 2.3 SSL Pinning — ปิดช่องโจมตีแบบ Man-in-the-Middle (MITM)

> 💡 **ทำไมต้องมี:** ปกติแอปเชื่อ certificate ใด ๆ ที่ออกโดย CA ที่เครื่องไว้ใจ แต่ผู้โจมตีที่ติดตั้ง CA ปลอม (เช่น ผ่าน proxy บน WiFi สาธารณะ หรือบนเครื่อง root) จะสามารถ "ดักอ่าน/แก้ไข" ทราฟฟิก HTTPS ได้ **SSL Pinning** คือการฝัง "ลายนิ้วมือ" ของ certificate/public key ของเซิร์ฟเวอร์จริงไว้ในแอป แล้วปฏิเสธการเชื่อมต่อทันทีหากไม่ตรง

```
ปกติ (เสี่ยง MITM):                    มี SSL Pinning (ปลอดภัย):
┌─────────────────────────────┐        ┌─────────────────────────────┐
│ App → เชื่อ CA ใด ๆ ที่ติดตั้ง │        │ App → เทียบ fingerprint ที่ฝัง │
│ Proxy ปลอมแทรกได้            │  ⟶    │ ไม่ตรง = ตัดการเชื่อมต่อทันที  │
│ อ่าน/แก้ payload ได้          │        │ Proxy ปลอมใช้ไม่ได้           │
└─────────────────────────────┘        └─────────────────────────────┘
```

มี 2 แนวทางหลัก — เราเลือก **Public Key Pinning** เพราะทนต่อการต่ออายุ certificate (cert หมดอายุแต่ public key เดิม pin ยังใช้ได้)

| แนวทาง                 | Pin อะไร                        | ข้อดี                            | ข้อเสีย                              |
| ---------------------- | ------------------------------- | -------------------------------- | ------------------------------------ |
| Certificate Pinning    | ทั้งใบ certificate (SHA-256)    | ตรวจเข้มที่สุด                   | ต้องอัปเดตแอปทุกครั้งที่ต่ออายุ cert |
| **Public Key Pinning** | เฉพาะ public key (SPKI SHA-256) | ต่ออายุ cert ได้โดยไม่ต้องแก้แอป | ต้องคุม key rotation ให้ดี           |

**วิธีทำกับ Dio** — ใช้ `onHttpClientCreate` ของ `IOHttpClientAdapter` เพื่อตรวจ fingerprint ของ certificate ก่อนยอมเชื่อมต่อ

```dart
// core/network/ssl_pinning.dart
import 'dart:io';
import 'package:crypto/crypto.dart';
import 'package:dio/dio.dart';
import 'package:dio/io.dart';

// SHA-256 ของ public key (SPKI) ของเซิร์ฟเวอร์จริง — ดึงด้วย openssl ล่วงหน้า
const _pinnedSpkiSha256 = <String>{
  'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=', // key หลัก
  'BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=', // key สำรอง (เผื่อ rotate)
};

void applySslPinning(Dio dio) {
  dio.httpClientAdapter = IOHttpClientAdapter(
    createHttpClient: () {
      final client = HttpClient();
      // ตรวจ public key ของ cert ที่เซิร์ฟเวอร์ส่งมา เทียบกับ pin ที่ฝังไว้
      client.badCertificateCallback = (cert, host, port) {
        final spkiHash = base64.encode(
          sha256.convert(cert.der).bytes, // หมายเหตุการ implement ดู note ด้านล่าง
        );
        return _pinnedSpkiSha256.contains(spkiHash);
      };
      return client;
    },
  );
}
```

> ⚠️ **ข้อควรระวังภาคปฏิบัติ:** การคำนวณ SPKI hash ที่ถูกต้องต้องดึง public key (subjectPublicKeyInfo) ออกจาก certificate ไม่ใช่ hash ทั้งใบ — ในงานจริงแนะนำใช้แพ็กเกจสำเร็จรูป เช่น `http_certificate_pinning` หรือ `ssl_pinning_plugin` ที่จัดการรายละเอียดนี้ให้ และ **อย่าลืมเตรียม pin สำรอง (backup pin)** เสมอ มิฉะนั้นวันที่เซิร์ฟเวอร์เปลี่ยน key ผู้ใช้ทั้งหมดจะเข้าแอปไม่ได้จนกว่าจะอัปเดต

วิธีดึง SPKI fingerprint ของเซิร์ฟเวอร์ (รันบนเครื่อง dev ล่วงหน้า):

```bash
openssl s_client -connect api.bla-demo.example.com:443 -servername api.bla-demo.example.com < /dev/null 2>/dev/null \
  | openssl x509 -pubkey -noout \
  | openssl pkey -pubin -outform der \
  | openssl dgst -sha256 -binary \
  | openssl enc -base64
```

ตัวอย่างของ ดึง SPKI SHA-256 fingerprint ของ www.google.com:

```bash
openssl s_client -connect www.google.com:443 -servername www.google.com < /dev/null 2>/dev/null \
  | openssl x509 -pubkey -noout \
  | openssl pkey -pubin -outform der \
  | openssl dgst -sha256 -binary \
  | openssl enc -base64
```

**อธิบายแต่ละขั้นของ pipeline** (5 ขั้นตอน ต่อกันเพื่อแปลง certificate → public key → SPKI hash → base64):

**ขั้นที่ 1 — `openssl s_client` (ต่อ TLS ไปดึง certificate)**

| Flag                                    | ความหมาย                                                                                                                                                                       |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `-connect api.bla-demo.example.com:443` | ระบุ host และ port ที่จะเปิด TLS connection ไปหา (443 = HTTPS)                                                                                                                 |
| `-servername api.bla-demo.example.com`  | ส่ง **SNI** (Server Name Indication) ไปใน TLS handshake — จำเป็นเมื่อเซิร์ฟเวอร์เดียวโฮสต์หลายโดเมน (เช่นอยู่หลัง CDN/load balancer) ถ้าไม่ใส่อาจได้ certificate ผิดใบ         |
| `< /dev/null`                           | ป้อน input ว่างให้คำสั่ง — ปกติ `s_client` จะค้างรอให้เราพิมพ์ข้อมูลส่งไปหาเซิร์ฟเวอร์ การ redirect จาก `/dev/null` ทำให้มัน handshake เสร็จแล้วปิด connection ทันที (ไม่ค้าง) |
| `2>/dev/null`                           | ทิ้งข้อความ stderr (พวก log ของ handshake, verify chain) เพื่อไม่ให้ปนกับ output ที่จะส่งเข้า pipe ถัดไป                                                                       |

Output ของขั้นนี้มี certificate ในรูป PEM ปนอยู่ ซึ่งขั้นถัดไปจะแกะออกมา

**ขั้นที่ 2 — `openssl x509 -pubkey -noout` (แกะ public key ออกจาก cert)**

| Flag      | ความหมาย                                                                                                        |
| --------- | --------------------------------------------------------------------------------------------------------------- |
| `-pubkey` | พิมพ์เฉพาะ **public key** (ในรูป PEM) ที่อยู่ในตัว certificate ออกมา                                            |
| `-noout`  | **ไม่ต้อง** พิมพ์ตัว certificate เองออกมาด้วย — ผลลัพธ์จึงเหลือแค่บล็อก `-----BEGIN PUBLIC KEY-----` อย่างเดียว |

**ขั้นที่ 3 — `openssl pkey -pubin -outform der` (แปลง key เป็น DER)**

| Flag           | ความหมาย                                                                                                                                                                             |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `-pubin`       | บอกว่า input ที่รับเข้ามาเป็น **public key** (ค่า default ของ `pkey` คือคาดหวัง private key ถ้าไม่ใส่จะ error)                                                                       |
| `-outform der` | แปลง output จาก PEM (base64 + header) เป็น **DER** ซึ่งเป็น binary encoding ของโครงสร้าง SubjectPublicKeyInfo (SPKI) — ต้อง hash จากรูป DER นี้เท่านั้นถึงจะได้ค่า pin ที่ตรงมาตรฐาน |

**ขั้นที่ 4 — `openssl dgst -sha256 -binary` (คำนวณ hash)**

| Flag      | ความหมาย                                                                                                                                                                                    |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-sha256` | ใช้อัลกอริทึม SHA-256 คำนวณ digest ของ DER bytes                                                                                                                                            |
| `-binary` | ให้ output เป็น **raw binary 32 bytes** แทนที่จะเป็น hex string ที่มี prefix (`SHA2-256(stdin)= ...`) — จำเป็นเพราะขั้นถัดไปจะเอา binary ไป base64 ตรง ๆ ถ้าไม่ใส่ flag นี้จะได้ค่า pin ผิด |

**ขั้นที่ 5 — `openssl enc -base64` (encode เป็นข้อความ)**

| Flag      | ความหมาย                                                                                                                                       |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `-base64` | เข้ารหัส binary hash 32 bytes เป็น **Base64** ได้ผลลัพธ์เป็นสตริง 44 ตัวอักษร (ลงท้าย `=`) เช่น `47DEQpj8HBSa+/TImW+5JCeuQeRkm5NMpJWZG3hSuFU=` |

ค่าที่ได้คือ SPKI fingerprint ที่เอาไปใช้ pin ได้เลย เช่นในรูปแบบ `sha256/<ค่า base64>` ของ OkHttp หรือใส่ในลิสต์ pin ของแพ็กเกจ pinning ฝั่ง Flutter

> 💡 **จุดที่คนพลาดบ่อย 2 จุด:** ต้อง hash จาก **DER ไม่ใช่ PEM** (ขั้นที่ 3) และต้องใช้ `-binary` ก่อน base64 (ขั้นที่ 4) — พลาดจุดใดจุดหนึ่งจะได้ fingerprint ที่ไม่ตรงกับที่ไลบรารีฝั่งแอปคำนวณ ทำให้ pinning fail ทั้งที่ certificate ถูกต้อง

### 2.4 Data Security & Encryption — เข้ารหัส Payload และซ่อนโค้ด

แม้จะมี HTTPS + SSL Pinning แล้ว ข้อมูลที่อ่อนไหวมาก (เช่น เลขกรมธรรม์, ข้อมูลสุขภาพ) อาจต้องการการเข้ารหัสอีกชั้น (end-to-end / application-level) เพื่อให้แม้ payload หลุดออกไปก็อ่านไม่ได้ เราใช้ **AES** ผ่านแพ็กเกจ `encrypt` (ซึ่งใช้ `pointycastle` เบื้องหลัง)

```bash
flutter pub add encrypt
```

```dart
// core/security/crypto_service.dart
import 'package:encrypt/encrypt.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../storage/secure_storage.dart';

part 'crypto_service.g.dart';

class CryptoService {
  final Encrypter _encrypter;
  final IV _iv;

  CryptoService(String key)
      : _encrypter = Encrypter(AES(Key.fromUtf8(key), mode: AESMode.cbc)),
        _iv = IV.fromLength(16);

  // เข้ารหัสก่อนส่งขึ้น API
  String encryptText(String plainText) =>
      _encrypter.encrypt(plainText, iv: _iv).base64;

  // ถอดรหัสหลังรับจาก API
  String decryptText(String base64CipherText) =>
      _encrypter.decrypt(Encrypted.fromBase64(base64CipherText), iv: _iv);
}

@riverpod
CryptoService cryptoService(CryptoServiceRef ref) {
  // key 32 ตัวอักษร (AES-256) — ในงานจริงต้องไม่ hardcode (ดู note ด้านล่าง)
  return CryptoService('0123456789abcdef0123456789abcdef');
}
```

นำไปเสียบใน Dio Interceptor เพื่อเข้ารหัส/ถอดรหัสอัตโนมัติทุก request/response:

```dart
// core/network/crypto_interceptor.dart
import 'package:dio/dio.dart';
import '../security/crypto_service.dart';

class CryptoInterceptor extends Interceptor {
  final CryptoService crypto;
  CryptoInterceptor(this.crypto);

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    // เข้ารหัส body ก่อนส่ง (ถ้ามี)
    if (options.data is String) {
      options.data = crypto.encryptText(options.data as String);
    }
    handler.next(options);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    // ถอดรหัส body ที่ได้กลับมา (ถ้าเซิร์ฟเวอร์ส่งมาเข้ารหัส)
    if (response.data is String) {
      response.data = crypto.decryptText(response.data as String);
    }
    handler.next(response);
  }
}
```

> ⚠️ **การจัดการ Key อย่างปลอดภัย (สำคัญที่สุด):** ห้าม hardcode key ลงในซอร์สโค้ดเด็ดขาด เพราะ decompile แอปแล้วเจอได้ทันที แนวทางที่ดีกว่าคือ (1) รับ key จากเซิร์ฟเวอร์หลัง login แล้วเก็บใน **Secure Storage**, (2) ใช้ `--dart-define` ฉีดตอน build, หรือ (3) ผูกกับ key ที่สร้างใน Keystore/Keychain ของอุปกรณ์ และควรใช้ IV แบบสุ่มต่อ message (ไม่ fix ค่าเดียว) ในงาน production จริง

**Obfuscation สำหรับ Release Build** — ทำให้ชื่อ class/method ในไบนารีอ่านยาก ลดโอกาส reverse-engineering

```bash
# ใส่ --obfuscate และเก็บ symbol map ไว้ถอดรหัส stack trace ภายหลัง
flutter build apk --release --obfuscate --split-debug-info=build/symbols
flutter build ipa --release --obfuscate --split-debug-info=build/symbols
```

> ✅ **สรุปชั้นความปลอดภัย (Defense in Depth):** Secure Storage (เก็บ token) → HTTPS → **SSL Pinning** (กัน MITM) → **AES Payload Encryption** (กันข้อมูลหลุด) → **Obfuscation** (กัน reverse-engineer) — แต่ละชั้นเสริมกัน ไม่มีชั้นใดชั้นเดียวที่พอ

---

## 📚 Module 3: Platform Channel — Native Integration

### เวลา 11:20–12:00 น.

> 💡 **หัวใจของ Module นี้:** Flutter วาด UI ของตัวเองทั้งหมด แต่บางความสามารถอยู่ "นอกโลก Dart" — เช่น Biometric (สแกนนิ้ว/หน้า), เซ็นเซอร์เฉพาะรุ่น, หรือ Native SDK ของพาร์ตเนอร์ (เช่น SDK ชำระเงินของธนาคาร) **Platform Channel** คือ "สะพาน" ส่งข้อความระหว่าง Dart กับโค้ด Native (Kotlin/Swift) เพื่อเรียกใช้ความสามารถเหล่านั้น

---

### 3.1 Platform Channel ทำงานอย่างไร

การสื่อสารเป็นแบบ **asynchronous message passing** ผ่าน "ชื่อช่อง (channel name)" ที่ตกลงกันทั้งสองฝั่ง ฝั่ง Dart เรียก `invokeMethod` ส่งชื่อเมธอด + อาร์กิวเมนต์ ฝั่ง Native รับที่ `setMethodCallHandler` แล้วส่งผลกลับ

```
┌─────────── Dart (Flutter) ───────────┐        ┌─────────── Native ───────────┐
│ MethodChannel('bla/device')          │        │ Android: MethodChannel (Kotlin)│
│   .invokeMethod('getDeviceId')       │  ⟶    │ iOS: FlutterMethodChannel(Swift)│
│        ↑ ผลลัพธ์ (Future)             │  ⟵    │   คืนค่า / โยน error            │
└──────────────────────────────────────┘        └──────────────────────────────┘
              ชื่อช่องต้อง "ตรงกันเป๊ะ" ทั้งสองฝั่ง
```

| ชนิด Channel        | ใช้เมื่อ                                                 |
| ------------------- | -------------------------------------------------------- |
| **MethodChannel**   | เรียกเมธอดครั้งเดียวแล้วได้ผลกลับ (ที่ใช้บ่อยที่สุด)     |
| EventChannel        | สตรีมข้อมูลต่อเนื่องจาก Native (เช่น เซ็นเซอร์, ตำแหน่ง) |
| BasicMessageChannel | ส่งข้อความสองทางแบบ custom codec                         |

### 3.2 ฝั่ง Dart — เรียก Native ผ่าน MethodChannel

```dart
// core/platform/device_info_service.dart
import 'package:flutter/services.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'device_info_service.g.dart';

class DeviceInfoService {
  // ชื่อช่องต้องตรงกับฝั่ง Native (แนะนำตั้งชื่อแบบ reverse-domain)
  static const _channel = MethodChannel('com.bla.policy/device');

  // เรียก native เพื่อขอ Device ID
  Future<String> getDeviceId() async {
    try {
      final id = await _channel.invokeMethod<String>('getDeviceId');
      return id ?? 'unknown';
    } on PlatformException catch (e) {
      // จัดการ error ที่ native โยนกลับมา (ดูข้อ 3.5)
      throw Exception('อ่าน Device ID ไม่สำเร็จ: ${e.message}');
    } on MissingPluginException {
      // เกิดเมื่อยังไม่ได้ทำฝั่ง native ของแพลตฟอร์มนั้น
      throw Exception('ยังไม่รองรับฟีเจอร์นี้บนแพลตฟอร์มนี้');
    }
  }

  // ตัวอย่าง: ยืนยันตัวตนด้วย Biometric ผ่าน native
  Future<bool> authenticateBiometric() async {
    final ok = await _channel.invokeMethod<bool>('authenticateBiometric');
    return ok ?? false;
  }
}

@riverpod
DeviceInfoService deviceInfoService(DeviceInfoServiceRef ref) {
  return DeviceInfoService();
}
```

### 3.3 ฝั่ง Android (Kotlin) — รับและตอบกลับ

```kotlin
// android/app/src/main/kotlin/com/bla/policy/MainActivity.kt
package com.bla.policy

import android.os.Build
import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.MethodChannel

class MainActivity : FlutterActivity() {
    private val channelName = "com.bla.policy/device"

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, channelName)
            .setMethodCallHandler { call, result ->
                when (call.method) {
                    "getDeviceId" -> {
                        // ส่งค่าจริงจาก native กลับไป
                        result.success("ANDROID-${Build.MODEL}-${Build.ID}")
                    }
                    else -> result.notImplemented()
                }
            }
    }
}
```

### 3.4 ฝั่ง iOS (Swift) — รับและตอบกลับ

```swift
// ios/Runner/AppDelegate.swift
import Flutter
import UIKit

@main
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    let controller = window?.rootViewController as! FlutterViewController
    let channel = FlutterMethodChannel(
      name: "com.bla.policy/device",
      binaryMessenger: controller.binaryMessenger
    )

    channel.setMethodCallHandler { (call, result) in
      switch call.method {
      case "getDeviceId":
        let id = UIDevice.current.identifierForVendor?.uuidString ?? "unknown"
        result("IOS-\(id)")
      default:
        result(FlutterMethodNotImplemented)
      }
    }

    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```

### 3.5 จัดการ Error และ Async Result จากฝั่ง Native

ฝั่ง Native ส่งผลกลับได้ 3 แบบ และฝั่ง Dart ควรจัดการครบ

| ฝั่ง Native (ตอบ)                  | ฝั่ง Dart (รับ)                                |
| ---------------------------------- | ---------------------------------------------- |
| `result.success(value)`            | ได้ค่าปกติใน `await invokeMethod`              |
| `result.error(code, msg, details)` | โยน `PlatformException` (มี `code`, `message`) |
| `result.notImplemented()`          | โยน `MissingPluginException`                   |

> ✅ **จุดสำคัญ:** เนื่องจาก Platform Channel เป็น async ทั้งหมด เราจึงห่อมันด้วย Provider แล้วเรียกผ่าน `AsyncNotifier`/`FutureProvider` ได้เหมือนการเรียก API ปกติ — UI ใช้ `AsyncValue.when()` จัดการ loading/error ได้ทันที ทำให้ Native call กลมกลืนกับสถาปัตยกรรม Riverpod ที่วางไว้

> ⚠️ **ข้อควรระวัง:** โค้ดฝั่ง Native ต้องรันบน UI thread เมื่อเรียก `result.success/error` (โดยเฉพาะบน Android หากทำงานหนักใน background thread ต้อง `runOnUiThread { ... }` ก่อนตอบกลับ) มิฉะนั้นแอปอาจค้างหรือ crash

---

## 📚 Module 4: Testing in Riverpod

### เวลา 13:00–14:00 น.

> 💡 **หัวใจของ Module นี้:** ข้อดีที่สุดของ Riverpod คือ "ทดสอบ Logic ได้โดยไม่ต้องสร้าง UI" เราสร้าง `ProviderContainer` ในเทสต์ แล้ว override dependency ด้วย Mock — ทดสอบเร็ว ไม่ต้องต่อเน็ต ไม่ต้องเปิด emulator

---

### 3.1 โครงสร้างการเขียน Test ด้วย ProviderContainer

```dart
// test/policy_list_controller_test.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

// สร้าง Mock ของ Repository
class MockPolicyRepository extends Mock implements PolicyRepository {}

void main() {
  test('policyListController คืนรายการกรมธรรม์เมื่อ repository สำเร็จ', () async {
    final mockRepo = MockPolicyRepository();
    // กำหนดให้ mock คืนข้อมูลจำลองเมื่อถูกเรียก
    when(() => mockRepo.fetchPolicies()).thenAnswer(
      (_) async => const [
        Policy(
          id: 'P001',
          planName: 'BLA ตลอดชีพ',
          policyNumber: 'BLA-0001',
          premium: 24000,
          status: PolicyStatus.active,
        ),
      ],
    );

    // สร้าง container พร้อม override repository ด้วย mock
    final container = ProviderContainer(
      overrides: [
        policyRepositoryProvider.overrideWithValue(mockRepo),
      ],
    );
    addTearDown(container.dispose); // คืนทรัพยากรเมื่อจบเทสต์

    // อ่านค่า provider แบบ async แล้วตรวจผล
    final policies =
        await container.read(policyListControllerProvider.future);

    expect(policies, hasLength(1));
    expect(policies.first.planName, 'BLA ตลอดชีพ');
    verify(() => mockRepo.fetchPolicies()).called(1);
  });
}
```

### 3.2 หลักการ Mock Dependencies

| ขั้นตอน       | เครื่องมือ                                  | ทำอะไร                           |
| ------------- | ------------------------------------------- | -------------------------------- |
| สร้าง Mock    | `class MockX extends Mock implements X`     | สร้างของปลอมที่ควบคุมพฤติกรรมได้ |
| กำหนดพฤติกรรม | `when(() => mock.method()).thenAnswer(...)` | บอกว่าเมื่อถูกเรียกให้คืนอะไร    |
| ฉีดเข้าระบบ   | `provider.overrideWithValue(mock)`          | แทน dependency จริงด้วย mock     |
| ตรวจการเรียก  | `verify(() => mock.method()).called(1)`     | ยืนยันว่าถูกเรียกตามที่คาด       |

### 3.3 ทดสอบสถานะ Error

```dart
test('policyListController คืน AsyncError เมื่อ repository ล้มเหลว', () async {
  final mockRepo = MockPolicyRepository();
  when(() => mockRepo.fetchPolicies())
      .thenThrow(const ApiException('เครือข่ายล้มเหลว'));

  final container = ProviderContainer(
    overrides: [policyRepositoryProvider.overrideWithValue(mockRepo)],
  );
  addTearDown(container.dispose);

  // คาดหวังว่าการอ่าน .future จะ throw
  await expectLater(
    container.read(policyListControllerProvider.future),
    throwsA(isA<ApiException>()),
  );
});
```

> ✅ **เคล็ดลับ:** เขียนเทสต์อย่างน้อย 3 กรณีต่อ controller — (1) สำเร็จคืนข้อมูลถูก, (2) ล้มเหลวเป็น error, (3) เมธอด mutation เปลี่ยน state ถูกต้อง การครอบคลุม 3 กรณีนี้จับบั๊กได้เกือบทั้งหมดของ logic การจัดการ state

---

## 📚 Module 5: Performance Optimization & Best Practices 2026

### เวลา 14:00–15:00 น.

> 💡 **หัวใจของ Module นี้:** แอปที่ "ทำงานได้" กับแอปที่ "ลื่นไหลระดับ Production" ต่างกันที่รายละเอียด — เฟรมที่หล่น (jank), หน่วยความจำที่รั่ว, และการ rebuild ที่เกินจำเป็น เราจะเรียนวิธี "วัด" ปัญหาเหล่านี้ด้วย Flutter DevTools ก่อน แล้วจึงแก้ด้วยเทคนิคที่ถูกต้อง — เพราะ "สิ่งที่วัดไม่ได้ ปรับปรุงไม่ได้"

---

### 5.1 ลด Jank และ Frame Drop

จอมือถือส่วนใหญ่รีเฟรช 60fps (บางรุ่น 120fps) แปลว่า Flutter มีเวลาราว **16 มิลลิวินาทีต่อเฟรม** ถ้าเฟรมใดใช้เวลาเกินนั้น ผู้ใช้จะเห็นอาการ "สะดุด" (jank)

```
Flutter Rendering Pipeline (ต่อ 1 เฟรม ~16ms):
Build → Layout → Paint → Composite → Raster
  ↑ ถ้าขั้นใดนานเกินงบ 16ms → เฟรมหล่น (jank)
```

เทคนิคหลักลด jank:

| เทคนิค                    | ทำอะไร                                   | ผลลัพธ์                              |
| ------------------------- | ---------------------------------------- | ------------------------------------ |
| `const` Widget            | บอก Flutter ว่า widget นี้ไม่เปลี่ยน     | ข้ามการ rebuild/repaint ทั้ง subtree |
| `RepaintBoundary`         | แยกชั้นการวาดของ widget ที่ animate บ่อย | repaint เฉพาะส่วนนั้น ไม่ลามทั้งจอ   |
| `ListView.builder`        | สร้างเฉพาะ item ที่มองเห็น               | ลด build/memory ของ list ยาว         |
| เลี่ยงงานหนักใน `build()` | ย้ายการคำนวณ/จัดเรียงออกไป provider      | `build()` เบา เฟรมไม่หล่น            |

```dart
// ❌ ไม่ดี: AnimatedBuilder ทำให้ทั้ง subtree repaint ทุกเฟรมของ animation
// ✅ ดี: ครอบส่วนที่เคลื่อนไหวด้วย RepaintBoundary เพื่อจำกัดขอบเขตการวาด
RepaintBoundary(
  child: AnimatedLogo(), // วาดใหม่เฉพาะกล่องนี้ ไม่ลามไปรายการกรมธรรม์ข้าง ๆ
)
```

**วัดด้วย Flutter DevTools (Performance Overlay)** — เปิด overlay เพื่อดูกราฟเวลาต่อเฟรมแบบเรียลไทม์

```bash
# รันแบบ profile mode เพื่อวัดประสิทธิภาพจริง (อย่าวัดใน debug mode)
flutter run --profile
```

> ⚠️ **กฎเหล็กของการวัด:** วัด performance ใน **profile mode** เท่านั้น ไม่ใช่ debug mode — เพราะ debug mode ปิด optimization หลายอย่างทำให้ตัวเลขช้ากว่าความจริงมาก แถบสีแดงใน Performance Overlay = เฟรมที่หล่น ให้ไล่หาว่าขั้นไหน (UI thread / Raster thread) เป็นคอขวด

### 5.2 Optimize Rendering — ใช้ select เพื่อลด Rebuild

บั๊กด้าน performance ที่พบบ่อยที่สุดใน Riverpod คือ "UI rebuild บ่อยเกินจำเป็น" เพราะ watch ค่าที่กว้างเกินไป ถ้า watch ทั้งออบเจกต์ UI จะ rebuild ทุกครั้งที่ฟิลด์ใดก็ตามเปลี่ยน แต่ถ้าใช้ `.select()` จะ rebuild เฉพาะเมื่อฟิลด์ที่เราสนใจเปลี่ยนเท่านั้น

```dart
// ❌ ไม่ดี: rebuild ทุกครั้งที่ user เปลี่ยนฟิลด์ใดก็ได้
final user = ref.watch(authControllerProvider).valueOrNull;
return Text(user?.displayName ?? '');

// ✅ ดี: rebuild เฉพาะเมื่อ displayName เปลี่ยนเท่านั้น
final name = ref.watch(
  authControllerProvider.select((async) => async.valueOrNull?.displayName),
);
return Text(name ?? '');
```

### 5.3 Memory Management — กัน Memory Leak

Memory leak ใน Flutter มักเกิดจาก "resource ที่สร้างแล้วลืมปิด" — controller, timer, stream subscription, listener สิ่งเหล่านี้ถ้าไม่ `dispose` จะค้างใน memory แม้หน้าจะถูกปิดไปแล้ว

| แหล่ง Leak ที่พบบ่อย                                               | วิธีป้องกัน                                |
| ------------------------------------------------------------------ | ------------------------------------------ |
| `TextEditingController`, `ScrollController`, `AnimationController` | `dispose()` ใน `dispose()` ของ State       |
| `Timer`, `StreamSubscription` ใน provider                          | `ref.onDispose(() => ...)`                 |
| `ref.keepAlive()` ที่ไม่มีวันปิด                                   | ตั้ง timer ปิด `link.close()` (ดูวันที่ 2) |
| Listener ที่ `addListener`                                         | `removeListener` ตอน dispose               |

```dart
// ภายใน @riverpod provider — ทุก resource ต้องมีคู่ onDispose เสมอ
@riverpod
Stream<int> tickStream(TickStreamRef ref) {
  final controller = StreamController<int>();
  final timer = Timer.periodic(const Duration(seconds: 1), (t) {
    controller.add(t.tick);
  });
  // เคลียร์ทั้งคู่เมื่อ provider ถูกทำลาย → ไม่มี leak
  ref.onDispose(() {
    timer.cancel();
    controller.close();
  });
  return controller.stream;
}
```

**ตรวจ Memory ด้วย DevTools (Memory tab)** — ดู heap snapshot เพื่อหา object ที่ "ควรถูกเก็บกวาด (GC) แล้วแต่ยังค้าง" วิธีคลาสสิกคือ เปิด-ปิดหน้าซ้ำ ๆ แล้วดูว่า instance ของหน้านั้นเพิ่มขึ้นเรื่อย ๆ หรือไม่ (ถ้าเพิ่ม = leak)

> 💡 **ข้อดีของ Riverpod:** auto-dispose ช่วยลด leak ให้มากแล้วโดยอัตโนมัติ แต่ resource ที่ "เราสร้างเอง" ภายใน provider ยังต้องเคลียร์เองใน `ref.onDispose()` เสมอ — Riverpod ไม่รู้จัก Timer/Stream ที่เราสร้าง

### 5.4 Production-ready App — Checklist ก่อน Release

ก่อนปล่อยแอปขึ้น Store มีงานที่มัก "ลืม" จนกลายเป็นปัญหาหลังบ้าน รวบเป็น checklist ได้ดังนี้

| หมวด                   | สิ่งที่ต้องทำ                                                                                    |
| ---------------------- | ------------------------------------------------------------------------------------------------ |
| **Code Obfuscation**   | `flutter build --obfuscate --split-debug-info=...` กัน reverse-engineer (ดู Module 2.4)          |
| **Logging Strategy**   | ปิด `print`/verbose log ใน release, ใช้ logger ที่ปรับระดับตาม flavor ได้                        |
| **Error Monitoring**   | ต่อ Crash Reporting (เช่น Crashlytics/Sentry) ดักผ่าน `FlutterError.onError` + `runZonedGuarded` |
| **Build Flavor**       | แยก Dev / Staging / Prod ให้คนละ bundle id, ไอคอน, ชื่อแอป                                       |
| **Environment Config** | ฉีดค่า (baseUrl, key) ด้วย `--dart-define` ไม่ hardcode ลงโค้ด                                   |
| **เวอร์ชัน & ขนาด**    | ตั้ง version/build number, ตรวจขนาด APK/IPA และทำ `--split-per-abi` ถ้าจำเป็น                    |

```dart
// ตัวอย่าง: ดักทุก error ทั้งแอปเพื่อส่งเข้า Crash Reporting
void main() {
  runZonedGuarded(() {
    FlutterError.onError = (details) {
      // ส่งเข้า Crashlytics/Sentry แทนการแสดงจอแดง
      // CrashReporter.record(details.exception, details.stack);
    };
    runApp(const ProviderScope(child: BlaApp()));
  }, (error, stack) {
    // CrashReporter.record(error, stack);
  });
}
```

```bash
# ตัวอย่างการ build แต่ละ flavor พร้อมฉีด environment
flutter build apk --release --flavor prod \
  --obfuscate --split-debug-info=build/symbols \
  --dart-define=BASE_URL=https://api.bla.example.com
```

### 5.5 สรุป Best Practices 2026

| แนวปฏิบัติ                                                   | เหตุผล                                       |
| ------------------------------------------------------------ | -------------------------------------------- |
| ใช้ `ref.watch().select()` เมื่อสนใจเฉพาะบางฟิลด์            | ลด rebuild ที่ไม่จำเป็น                      |
| ใช้ `ref.read()` ใน callback เสมอ ห้าม `watch`               | กัน subscribe ซ้ำซ้อน                        |
| แยก Widget ย่อยให้ watch provider ที่จำเป็นเท่านั้น          | จำกัดขอบเขตการ rebuild                       |
| ใช้ `const` constructor ทุกที่ที่ทำได้                       | Flutter ข้ามการ rebuild widget ที่ไม่เปลี่ยน |
| ใช้ Code Generation (`@riverpod`)                            | compile-safe, ลด bug จากการพิมพ์ชนิดผิด      |
| เคลียร์ resource ใน `ref.onDispose()` เสมอ                   | กัน memory leak                              |
| ใช้ `@Riverpod(keepAlive: true)` เฉพาะ state ที่ต้องอยู่ตลอด | ประหยัดหน่วยความจำ                           |

> ⚠️ **กับดักที่พบบ่อย:** การ `ref.watch` provider ที่เป็น `AsyncValue` ทั้งก้อนในหน้าใหญ่ ทำให้ทุก loading/refresh สั่น UI ทั้งหน้า — ให้ย้ายการ watch ลงไปใน widget ย่อยที่เล็กที่สุดที่ต้องใช้ค่านั้นจริง ๆ

---

## 🛠️ Master Workshop — BLA Policy Companion: ประกอบร่างระบบ Authentication ครบวงจร

### เวลา 15:00–16:00 น.

> **โจทย์:** สร้างระบบ Login/Logout จริงผ่าน API (`POST /auth/login` → token) เก็บ Token ด้วย Secure Storage, ควบคุม Route Guard ด้วย Auth State จาก Riverpod ผ่าน **GoRouter + StatefulShellRoute**, แนบ Token เข้า Dio อัตโนมัติ **พร้อมเปิด SSL Pinning + Crypto Interceptor**, เสริม **Biometric จริงผ่าน Platform Channel (BiometricPrompt)** และ **PIN Login**, ต่อยอดด้วย **แก้ไขโปรไฟล์/เปลี่ยนรหัสผ่าน**, **i18n ไทย/อังกฤษ**, **Dark Mode** และปิดท้ายด้วย **Unit Test** — เชื่อมทุกอย่างจากวันที่ 1–3 เข้าด้วยกัน · เนื้อหาส่วนนี้คือ **โค้ดจริงปัจจุบันครบทุกไฟล์** ของ branch `day3-stage3` ที่เพิ่ม/เปลี่ยนจาก `day2`

### 🗺️ ภาพรวม — อะไรเพิ่ม/เปลี่ยนจาก day2 (ทำเป็น 3 Stage)

Branch นี้พัฒนาเป็น 3 จังหวะตามประวัติ commit: **Stage 1** = Auth จริง + GoRouter + Secure Storage → **Stage 2** = Security (SSL Pinning, Crypto) + Biometric/PIN จริง → **Stage 3** = i18n (ไทย/EN) + Dark Mode + แก้ไขโปรไฟล์/เปลี่ยนรหัสผ่าน

**ไฟล์เพิ่มใหม่ (A)** — 23 ไฟล์ (+generated):

| กลุ่ม             | ไฟล์ใหม่                                                                                                                                                                                                    |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Storage/Security  | `core/storage/secure_storage.dart`, `core/security/crypto_service.dart`, `core/network/crypto_interceptor.dart`, `core/network/ssl_pinning.dart`                                                            |
| Routing           | `core/router/app_router.dart`, `core/router/home_shell.dart`, `core/router/router_refresh.dart`                                                                                                             |
| Platform Channel  | `core/platform/device_info_service.dart` (+ native: `MainActivity.kt` ✏️, `AppDelegate.swift` ✏️)                                                                                                           |
| Auth feature      | `auth/domain/auth_user.dart`, `auth/data/auth_repository.dart`, `auth/presentation/controllers/auth_controller.dart`, `biometric_controller.dart`, `pages/pin_setup_page.dart`, `pages/pin_login_page.dart` |
| Settings/i18n/ธีม | `core/settings/settings_providers.dart`, `core/theme/app_palette.dart`, `core/l10n/l10n_ext.dart`, `l10n.yaml`, `lib/l10n/app_th.arb`, `app_en.arb` (+ generated `app_localizations*.dart`)                 |
| Profile feature   | `profile/pages/edit_profile_page.dart`, `profile/pages/change_password_page.dart`                                                                                                                           |
| Tests             | `test/auth_controller_test.dart`, `test/policy_list_controller_test.dart`                                                                                                                                   |

**ไฟล์ที่แก้ไข (M)** — `pubspec.yaml`, `main.dart` (→ MaterialApp.router + i18n + ThemeMode), `dio_client.dart` (token จริง + pinning), `app_theme.dart` (light/dark), `app_text_field.dart`+`section_label.dart` (palette-aware), auth pages เดิมทั้ง 5 (ต่อ API จริง + l10n), `policy.dart`+`claim_record.dart` (label รับ AppLocalizations), `claim_controller.dart`, หน้าแท็บทั้ง 5 (l10n + palette + go_router) และ `widget_test.dart`

**กราฟ Provider ใหม่ของ Day 3** (ต่อยอดจากกราฟ Day 2 — Repository 3 ตัวเดิมยังอยู่ครบ):

```
sharedPreferencesProvider (override ใน main() ด้วยค่าจริง)
  ├─▶ localeControllerProvider (Locale th/en, keepAlive)  ──▶ MaterialApp.locale
  └─▶ themeModeControllerProvider (light/dark, keepAlive) ──▶ MaterialApp.themeMode

secureStorageProvider (Keychain/Keystore: token + PIN + bio credentials)
  ├─▶ dioClientProvider ← AuthInterceptor(() => storage.readToken())
  │        │              + CryptoInterceptor(cryptoServiceProvider) + applySslPinning()
  │        └─▶ authRepositoryProvider ─▶ authControllerProvider ★ (keepAlive)
  │                                          │ AsyncValue<AuthUser?>
  │                                          ├─▶ routerRefreshProvider (Listenable)
  │                                          │        └─▶ appRouterProvider (GoRouter.redirect)
  │                                          ├─▶ ProfilePage / EditProfilePage (ข้อมูลผู้ใช้)
  │                                          └─▶ LoginPage (loading/error)
  └─▶ biometricControllerProvider ← deviceInfoServiceProvider (MethodChannel → Kotlin/Swift)
```

★ `authControllerProvider` คือ **แหล่งความจริงเดียว** — login/logout ที่ใดก็ตาม ทั้งแอป (router, ทุกหน้า) ตอบสนองพร้อมกันโดยไม่มีการ navigate ด้วยมือแม้แต่บรรทัดเดียว

**Flow การยืนยันตัวตน (ครบทุกเส้นทาง):**

```
เปิดแอป ─▶ /splash ─ AuthController.build(): อ่าน token จาก Secure Storage
              │            ├─ ไม่มี token → state = null ─▶ redirect ─▶ /login
              │            └─ มี token → GET /auth/me ─▶ AuthUser ─▶ redirect ─▶ /home
/login (ครั้งแรก: ฟอร์ม username/password)
   ├─ "เข้าสู่ระบบ" → login() → POST /auth/login → saveToken + saveBiometricCredentials
   ├─ เคย login แล้ว (มี credentials) → โหมด Quick Sign-in:
   │     ├─ "สแกนลายนิ้วมือ" → Platform Channel → BiometricPrompt (Kotlin) → login() ด้วย credentials ที่เก็บไว้
   │     └─ "ใช้ PIN" → /pin-login → verifyPin() กับ Secure Storage → login()
   └─ ลืมรหัสผ่าน → /forgot-password → /reset (API จริง) · สมัครสมาชิก → /register (API จริง)
Logout (หน้าโปรไฟล์) → deleteToken → state = null → router เด้งทุกหน้ากลับ /login
```

### ขั้นที่ 1 — เพิ่ม dependency (pubspec.yaml)

```yaml
dependencies:
  # ...ของเดิมจาก Day 1–2...
  flutter_localizations: # i18n — รองรับหลายภาษา (ไทย/อังกฤษ)
    sdk: flutter
  intl: ^0.20.2 # ใช้คู่กับ gen-l10n
  # Day 3: Routing + Security
  go_router: ^14.6.2
  flutter_secure_storage: ^9.2.4
  crypto: ^3.0.6 # SHA-256 สำหรับ SSL Pinning
  encrypt: ^5.0.3 # AES encryption (Crypto Interceptor)
  shared_preferences: ^2.3.2 # เก็บค่าตั้งค่า (ภาษา/โหมดธีม) แบบ non-secret

dev_dependencies:
  # ...ของเดิม...
  mocktail: ^1.0.4 # สร้าง Mock สำหรับ Unit Test

flutter:
  uses-material-design: true
  generate: true # สร้าง AppLocalizations จาก lib/l10n/*.arb (i18n)
  assets:
    - assets/images/
```

> 🔗 สังเกตการแบ่งหน้าที่ของที่เก็บข้อมูล: **Secure Storage** (token/PIN/credentials — ความลับ) กับ **SharedPreferences** (ภาษา/ธีม — ไม่ลับ อ่านเร็ว) · และ `generate: true` เปิดให้ `flutter gen-l10n` สร้างคลาสแปลภาษาอัตโนมัติทุกครั้งที่รัน

### ขั้นที่ 2 — Secure Storage: เก็บ token / PIN / biometric credentials 🆕

```dart
// lib/core/storage/secure_storage.dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'secure_storage.g.dart';

/// บริการเก็บข้อมูลความลับ (token) แบบเข้ารหัสด้วย Keychain (iOS) / Keystore (Android)
///
/// ห้ามเก็บ token/รหัสผ่านใน SharedPreferences (plain text — เครื่อง root/jailbreak อ่านได้)
class SecureStorageService {
  final FlutterSecureStorage _storage;
  SecureStorageService(this._storage);

  static const _keyToken = 'auth_token';
  static const _keyBioUsername = 'bio_username';
  static const _keyBioPassword = 'bio_password';
  static const _keyPin = 'user_pin';

  // === Token Management ===
  Future<void> saveToken(String token) =>
      _storage.write(key: _keyToken, value: token);

  Future<String?> readToken() => _storage.read(key: _keyToken);

  Future<void> deleteToken() => _storage.delete(key: _keyToken);

  // === Biometric Credentials Management ===
  /// บันทึก username/password สำหรับ biometric login
  Future<void> saveBiometricCredentials(String username, String password) async {
    await _storage.write(key: _keyBioUsername, value: username);
    await _storage.write(key: _keyBioPassword, value: password);
  }

  /// ดึง username/password สำหรับ biometric login
  Future<({String username, String password})?> getBiometricCredentials() async {
    final username = await _storage.read(key: _keyBioUsername);
    final password = await _storage.read(key: _keyBioPassword);
    if (username == null || password == null) return null;
    return (username: username, password: password);
  }

  /// ตรวจสอบว่ามีข้อมูล biometric ที่บันทึกไว้หรือไม่
  Future<bool> hasBiometricCredentials() async {
    final username = await _storage.read(key: _keyBioUsername);
    return username != null;
  }

  /// ลบข้อมูล biometric credentials
  Future<void> clearBiometricCredentials() async {
    await _storage.delete(key: _keyBioUsername);
    await _storage.delete(key: _keyBioPassword);
  }

  // === PIN Management ===
  /// บันทึก PIN (6 หลัก)
  Future<void> savePin(String pin) => _storage.write(key: _keyPin, value: pin);

  /// ดึง PIN ที่บันทึกไว้
  Future<String?> getPin() => _storage.read(key: _keyPin);

  /// ตรวจสอบว่ามี PIN ตั้งไว้หรือไม่
  Future<bool> hasPin() async {
    final pin = await _storage.read(key: _keyPin);
    return pin != null;
  }

  /// ลบ PIN
  Future<void> clearPin() => _storage.delete(key: _keyPin);

  /// ตรวจสอบ PIN ถูกต้องหรือไม่
  Future<bool> verifyPin(String pin) async {
    final saved = await getPin();
    return saved == pin;
  }
}

@riverpod
SecureStorageService secureStorage(SecureStorageRef ref) {
  return SecureStorageService(const FlutterSecureStorage());
}
```

> 🔗 **การเชื่อมโยง:** ไฟล์เดียวนี้เป็น "ตู้เซฟ" ของทั้งระบบ — `dioClientProvider` อ่าน token แนบ header, `AuthController` เขียน/ลบ token ตอน login/logout, `LoginPage`/`PinLoginPage` อ่าน biometric credentials, `PinSetupPage`/`PinLoginPage` จัดการ PIN — ทุกอย่างผ่านคลาสห่อ (wrapper) ที่รวม key ไว้ที่เดียว ไม่มี magic string กระจัดกระจาย

### ขั้นที่ 3 — โมเดล Auth (ชั้น Domain) 🆕

```dart
// lib/features/auth/domain/auth_user.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'auth_user.freezed.dart';
part 'auth_user.g.dart';

/// ข้อมูลผู้ใช้ที่ล็อกอินอยู่ (ชั้น Domain)
@freezed
class AuthUser with _$AuthUser {
  const AuthUser._();

  const factory AuthUser({
    required String id,
    required String displayName,
    required String role,
    String? phone,
  }) = _AuthUser;

  factory AuthUser.fromJson(Map<String, dynamic> json) =>
      _$AuthUserFromJson(json);

  /// ป้ายบทบาทภาษาไทย (แอปนี้เป็นฝั่งลูกค้า)
  String get roleLabel => role == 'customer' ? 'ลูกค้า' : role;

  /// เบอร์โทรสำหรับแสดงผล (ถ้ายังไม่ได้ระบุจะขึ้นขีด)
  String get phoneOrDash => (phone == null || phone!.isEmpty) ? '-' : phone!;
}

/// ผลลัพธ์การเข้าสู่ระบบ (token + ข้อมูลผู้ใช้) — ตรงกับ response ของ POST /auth/login
@freezed
class LoginResult with _$LoginResult {
  const factory LoginResult({
    required AuthUser user,
    required String token,
  }) = _LoginResult;

  factory LoginResult.fromJson(Map<String, dynamic> json) =>
      _$LoginResultFromJson(json);
}
```

> 🔗 **จุดสังเกต:** `const AuthUser._();` (private constructor) คือแพตเทิร์นของ freezed ที่เปิดให้เขียน **custom getter** (`roleLabel`, `phoneOrDash`) เพิ่มในคลาสได้ — กฎการแสดงผลอยู่ในชั้น Domain ไม่หลุดไปอยู่ใน UI

### ขั้นที่ 4 — AuthRepository: ครบทุก endpoint ของ /auth 🆕

```dart
// lib/features/auth/data/auth_repository.dart
import 'package:dio/dio.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

import '../../../core/network/api_exception.dart';
import '../../../core/network/dio_client.dart';
import '../domain/auth_user.dart';

part 'auth_repository.g.dart';

/// Repository การยืนยันตัวตน — เรียก API จริงผ่าน Dio
///
/// บัญชีทดสอบ (seed ใน API): username = customer, password = 1234
class AuthRepository {
  final Dio _dio;
  AuthRepository(this._dio);

  /// เข้าสู่ระบบ — POST /auth/login → { token, user }
  Future<LoginResult> login(String username, String password) async {
    try {
      final response = await _dio.post<Map<String, dynamic>>(
        '/auth/login',
        data: {'username': username, 'password': password},
      );
      return LoginResult.fromJson(response.data!);
    } on DioException catch (e) {
      // ใช้ข้อความจากเซิร์ฟเวอร์ถ้ามี (เช่น "ชื่อผู้ใช้หรือรหัสผ่านไม่ถูกต้อง")
      final data = e.response?.data;
      final serverMsg = data is Map<String, dynamic> ? data['message'] as String? : null;
      throw ApiException(serverMsg ?? mapDioError(e).message,
          statusCode: e.response?.statusCode);
    }
  }

  /// กู้ session ตอนเปิดแอป — GET /auth/me (AuthInterceptor แนบ token ให้อัตโนมัติ)
  Future<AuthUser?> getCurrentUser(String token) async {
    if (token.isEmpty) return null;
    try {
      final response = await _dio.get<Map<String, dynamic>>('/auth/me');
      return AuthUser.fromJson(response.data!);
    } on DioException catch (e) {
      if (e.response?.statusCode == 401) return null; // token หมดอายุ → ถือว่ายังไม่ล็อกอิน
      throw mapDioError(e);
    }
  }

  /// แก้ไขข้อมูลส่วนตัว — PUT /auth/me → AuthUser ที่อัปเดตแล้ว
  /// (AuthInterceptor แนบ Bearer token ให้อัตโนมัติ)
  Future<AuthUser> updateProfile({
    required String displayName,
    String? phone,
  }) async {
    try {
      final response = await _dio.put<Map<String, dynamic>>(
        '/auth/me',
        data: {'displayName': displayName, 'phone': phone},
      );
      return AuthUser.fromJson(response.data!);
    } on DioException catch (e) {
      throw _serverError(e);
    }
  }

  /// เปลี่ยนรหัสผ่าน — POST /auth/change-password → ข้อความแจ้งผล
  /// เซิร์ฟเวอร์ตรวจรหัสเดิมก่อน (400 ถ้าไม่ถูกต้อง)
  Future<String> changePassword({
    required String currentPassword,
    required String newPassword,
  }) async {
    try {
      final response = await _dio.post<Map<String, dynamic>>(
        '/auth/change-password',
        data: {
          'currentPassword': currentPassword,
          'newPassword': newPassword,
        },
      );
      return response.data?['message'] as String? ?? 'เปลี่ยนรหัสผ่านสำเร็จ';
    } on DioException catch (e) {
      throw _serverError(e);
    }
  }

  /// สมัครสมาชิก — POST /auth/register (email = username) → { token, user }
  Future<LoginResult> register({
    required String name,
    required String email,
    required String phone,
    required String password,
  }) async {
    try {
      final response = await _dio.post<Map<String, dynamic>>(
        '/auth/register',
        data: {
          'name': name,
          'email': email,
          'phone': phone,
          'password': password,
        },
      );
      return LoginResult.fromJson(response.data!);
    } on DioException catch (e) {
      throw _serverError(e);
    }
  }

  /// ขอลิงก์รีเซ็ตรหัสผ่าน — POST /auth/forgot-password → ข้อความแจ้งผล
  Future<String> forgotPassword(String email) async {
    try {
      final response = await _dio.post<Map<String, dynamic>>(
        '/auth/forgot-password',
        data: {'email': email},
      );
      return response.data?['message'] as String? ?? 'ส่งคำขอแล้ว';
    } on DioException catch (e) {
      throw _serverError(e);
    }
  }

  /// ตั้งรหัสผ่านใหม่ — POST /auth/reset-password → ข้อความแจ้งผล
  Future<String> resetPassword({
    required String email,
    required String password,
  }) async {
    try {
      final response = await _dio.post<Map<String, dynamic>>(
        '/auth/reset-password',
        data: {'email': email, 'password': password},
      );
      return response.data?['message'] as String? ?? 'ตั้งรหัสผ่านใหม่สำเร็จ';
    } on DioException catch (e) {
      throw _serverError(e);
    }
  }

  /// แปลง error โดยใช้ข้อความจากเซิร์ฟเวอร์ถ้ามี (เช่น "อีเมลนี้ถูกใช้สมัครแล้ว")
  ApiException _serverError(DioException e) {
    final data = e.response?.data;
    final msg = data is Map<String, dynamic> ? data['message'] as String? : null;
    return ApiException(msg ?? mapDioError(e).message,
        statusCode: e.response?.statusCode);
  }
}

@riverpod
AuthRepository authRepository(AuthRepositoryRef ref) =>
    AuthRepository(ref.watch(dioClientProvider));
```

> 🔗 **การเชื่อมโยง / จุดสังเกต:** ครบ 7 endpoint ของ `/auth` — และเพิ่มแพตเทิร์นใหม่จาก Day 2: `_serverError()` เลือกใช้ **ข้อความ error จากเซิร์ฟเวอร์ก่อน** (เช่น "รหัสผ่านเดิมไม่ถูกต้อง") ค่อย fallback เป็น `mapDioError` — ผู้ใช้เห็นสาเหตุจริง ไม่ใช่ข้อความกลาง ๆ · `getCurrentUser` ตีความ 401 เป็น "ยังไม่ล็อกอิน" (คืน null) ไม่ใช่ error เพื่อให้เปิดแอปด้วย token หมดอายุแล้วเด้งไป login เงียบ ๆ

### ขั้นที่ 5 — AuthController + BiometricController (หัวใจ Reactive Auth) 🆕

```dart
// lib/features/auth/presentation/controllers/auth_controller.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

import '../../../../core/storage/secure_storage.dart';
import '../../data/auth_repository.dart';
import '../../domain/auth_user.dart';

part 'auth_controller.g.dart';

/// แหล่งความจริงเดียวของสถานะผู้ใช้ (single source of truth)
///
/// keepAlive: true เพราะต้องอยู่ตลอดอายุแอป
/// - build()  : โหลด user จาก token ที่เก็บไว้ (กู้ session ตอนเปิดแอป)
/// - login()  : เข้าสู่ระบบ → เก็บ token → state เป็น user
/// - logout() : ลบ token → state เป็น null (ทั้งแอป redirect ไป login แบบ reactive)
@Riverpod(keepAlive: true)
class AuthController extends _$AuthController {
  @override
  Future<AuthUser?> build() async {
    // หน่วงเล็กน้อยตอนเปิดแอป เพื่อให้หน้า Splash แสดงสักครู่ (UX)
    await Future<void>.delayed(const Duration(milliseconds: 1200));
    final token = await ref.watch(secureStorageProvider).readToken();
    if (token == null) return null;
    return ref.watch(authRepositoryProvider).getCurrentUser(token);
  }

  Future<void> login(String username, String password, {bool rememberForBiometric = false}) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      final result =
          await ref.read(authRepositoryProvider).login(username, password);
      await ref.read(secureStorageProvider).saveToken(result.token);

      // บันทึก credentials สำหรับ biometric login (ถ้าผู้ใช้เลือก)
      if (rememberForBiometric) {
        await ref.read(secureStorageProvider).saveBiometricCredentials(username, password);
      }

      return result.user;
    });
  }

  /// แก้ไขข้อมูลส่วนตัว → PUT /auth/me → อัปเดต state เป็น user ใหม่
  Future<void> updateProfile({
    required String displayName,
    String? phone,
  }) async {
    final updated = await ref.read(authRepositoryProvider).updateProfile(
          displayName: displayName,
          phone: phone,
        );
    state = AsyncValue.data(updated);
  }

  /// เปลี่ยนรหัสผ่าน → POST /auth/change-password
  /// ถ้าสำเร็จและเคยบันทึก credentials สำหรับ Biometric/PIN ไว้
  /// จะอัปเดตรหัสผ่านที่เก็บด้วย เพื่อให้เข้าสู่ระบบเร็วยังใช้งานได้
  Future<String> changePassword({
    required String currentPassword,
    required String newPassword,
  }) async {
    final message = await ref.read(authRepositoryProvider).changePassword(
          currentPassword: currentPassword,
          newPassword: newPassword,
        );
    final storage = ref.read(secureStorageProvider);
    final saved = await storage.getBiometricCredentials();
    if (saved != null) {
      await storage.saveBiometricCredentials(saved.username, newPassword);
    }
    return message;
  }

  Future<void> logout() async {
    await ref.read(secureStorageProvider).deleteToken();
    state = const AsyncValue.data(null);
  }
}
```

```dart
// lib/features/auth/presentation/controllers/biometric_controller.dart
import 'dart:async';

import 'package:riverpod_annotation/riverpod_annotation.dart';

import '../../../../core/platform/device_info_service.dart';

part 'biometric_controller.g.dart';

/// Controller ยืนยันตัวตนด้วย Biometric ผ่าน Platform Channel
///
/// ห่อการเรียก Native ด้วย AsyncValue เพื่อให้ UI ใช้ loading/error/data ได้ตามปกติ
/// (Native call ก็คือ Future ที่ห่อด้วย AsyncValue เหมือนการเรียก API)
@riverpod
class BiometricController extends _$BiometricController {
  @override
  FutureOr<bool> build() => false; // ยังไม่ผ่านการยืนยัน

  /// ยืนยันตัวตนด้วย Biometric และ return ผลลัพธ์กลับมาตรงๆ
  /// (ไม่ต้องเซ็ต state เพื่อหลีกเลี่ยง "Future already completed" error)
  Future<bool> authenticate() async {
    try {
      final result = await ref.read(deviceInfoServiceProvider).authenticateBiometric();
      return result;
    } catch (e) {
      return false;
    }
  }
}
```

> 🔗 **การเชื่อมโยง / จุดสังเกต:**
>
> - `@Riverpod(keepAlive: true)` — auth state ต้องมีชีวิตตลอดอายุแอป (ต่างจาก provider อื่นที่ autoDispose) เพราะ router และทุกหน้า watch ตัวนี้
> - `build()` = "กู้ session": อ่าน token → `GET /auth/me` → คืน `AuthUser?` — ระหว่างนี้ router ค้างที่ `/splash` (ดูขั้นที่ 7)
> - `changePassword()` **sync รหัสใหม่เข้า Secure Storage** ให้ Biometric/PIN login ยังใช้ได้หลังเปลี่ยนรหัส — ดีเทลเล็ก ๆ ที่แอปจริงพลาดบ่อย
> - `BiometricController.authenticate()` เลือก **return ค่า** แทนเซ็ต `state` — บทเรียนจากบั๊กจริง: การเซ็ต state ระหว่าง build ยังไม่จบทำให้เกิด "Future already completed"

### ขั้นที่ 6 — ชั้นความปลอดภัยของ Network: Crypto + SSL Pinning + Dio ฉบับสมบูรณ์

#### 6.1 `crypto_service.dart` — AES encrypt/decrypt 🆕

```dart
// lib/core/security/crypto_service.dart
import 'package:encrypt/encrypt.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'crypto_service.g.dart';

/// บริการเข้ารหัส/ถอดรหัสข้อความด้วย AES (application-level encryption)
///
/// ใช้เข้ารหัส payload ที่อ่อนไหวมากอีกชั้น แม้จะมี HTTPS + SSL Pinning แล้ว
/// (เผื่อ payload หลุดออกไปก็อ่านไม่ได้)
class CryptoService {
  final Encrypter _encrypter;
  final IV _iv;

  CryptoService(String key)
      : _encrypter = Encrypter(AES(Key.fromUtf8(key), mode: AESMode.cbc)),
        _iv = IV.fromLength(16);

  /// เข้ารหัสก่อนส่งขึ้น API
  String encryptText(String plainText) =>
      _encrypter.encrypt(plainText, iv: _iv).base64;

  /// ถอดรหัสหลังรับจาก API
  String decryptText(String base64CipherText) =>
      _encrypter.decrypt(Encrypted.fromBase64(base64CipherText), iv: _iv);
}

@riverpod
CryptoService cryptoService(CryptoServiceRef ref) {
  // ⚠️ key 32 ตัวอักษร (AES-256) — ห้าม hardcode ใน production เพราะ decompile เจอได้
  //    แนวทางจริง: รับ key จากเซิร์ฟเวอร์หลัง login แล้วเก็บใน Secure Storage,
  //    หรือฉีดด้วย --dart-define ตอน build, และใช้ IV สุ่มต่อ message
  return CryptoService('0123456789abcdef0123456789abcdef');
}
```

#### 6.2 `crypto_interceptor.dart` — เข้ารหัส payload อัตโนมัติ 🆕

```dart
// lib/core/network/crypto_interceptor.dart
import 'package:dio/dio.dart';

import '../security/crypto_service.dart';

/// Interceptor เข้ารหัส request body และถอดรหัส response body อัตโนมัติ
///
/// ใช้คู่กับ [CryptoService] — เสียบเข้า Dio เพื่อให้ทุก request/response ที่เป็น String
/// ถูกเข้ารหัส/ถอดรหัสโดยที่ชั้น Repository ไม่ต้องรู้
///
/// หมายเหตุ: ทำงานเฉพาะเมื่อ payload เป็น String — API ปัจจุบันส่ง/รับเป็น JSON (Map)
/// interceptor นี้จึงเป็น no-op สำหรับ API ตอนนี้ (โครงไว้สาธิตแนวคิด + พร้อมใช้เมื่อ
/// backend รองรับ encrypted payload)
class CryptoInterceptor extends Interceptor {
  final CryptoService crypto;
  CryptoInterceptor(this.crypto);

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    if (options.data is String) {
      options.data = crypto.encryptText(options.data as String);
    }
    handler.next(options);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    if (response.data is String) {
      response.data = crypto.decryptText(response.data as String);
    }
    handler.next(response);
  }
}
```

#### 6.3 `ssl_pinning.dart` — Certificate Pinning กัน MITM 🆕

````dart
// lib/core/network/ssl_pinning.dart
import 'dart:convert';
import 'dart:io';

import 'package:crypto/crypto.dart';
import 'package:dio/dio.dart';
import 'package:dio/io.dart';

/// สวิตช์เปิด/ปิด SSL Pinning
///
/// ตั้ง `false` เพื่อปิดชั่วคราว (เช่น ตอน cert ของ Render หมุนเวียนแล้วเชื่อมต่อไม่ได้
/// หรือทดสอบผ่าน proxy) โดยไม่ต้องแก้ logic
const bool kEnableSslPinning = true;

/// SHA-256 (base64) ของ **certificate (DER)** ที่เชื่อถือ — เทียบกับค่าที่ Dart คำนวณจาก
/// `base64(sha256(cert.der))` ของ leaf certificate ที่เซิร์ฟเวอร์ส่งมา
///
/// วิธีดึงค่าใหม่ (รันบนเครื่อง dev):
/// ```bash
/// echo | openssl s_client -connect bla-mock-api.onrender.com:443 \
///   -servername bla-mock-api.onrender.com 2>/dev/null \
///   | openssl x509 -outform der | openssl dgst -sha256 -binary | openssl base64
/// ```
///
/// ⚠️ นี่คือ **Certificate Pinning ของ leaf** — cert ของ Render (Google Trust Services)
/// หมดอายุ ~24 ส.ค. 2026 เมื่อ **ต่ออายุ cert ค่านี้จะเปลี่ยน** ต้องอัปเดต pin ใหม่
/// (ไม่งั้นแอปจะเชื่อมต่อไม่ได้) — production ควรใช้ **Public Key (SPKI) Pinning + pin สำรอง**
/// ผ่านแพ็กเกจสำเร็จรูป เช่น `http_certificate_pinning`
const Set<String> _pinnedCertSha256 = {
  // leaf ปัจจุบันของ bla-mock-api.onrender.com
  'svZQ+GWVBhyE3yjb5e+PnpdDsH4n9mfjjx4Alk3gH5o=',
  // (เว้นที่ไว้สำหรับ pin สำรอง / ค่าใหม่ตอนต่ออายุ cert)
};

/// ใส่ SSL Pinning ให้ Dio — ปฏิเสธการเชื่อมต่อทันทีหาก fingerprint ไม่ตรง (กัน MITM)
///
/// ใช้ `IOHttpClientAdapter.validateCertificate` ซึ่งถูกเรียกทุกการเชื่อมต่อ
/// (ทั้ง cert ที่ผ่าน/ไม่ผ่าน CA) จึง pin ได้แม้ cert ถูกต้องตามปกติ
void applySslPinning(Dio dio) {
  if (!kEnableSslPinning) return;
  dio.httpClientAdapter = IOHttpClientAdapter(
    validateCertificate: (X509Certificate? cert, String host, int port) {
      if (cert == null) return false;
      final fingerprint = base64.encode(sha256.convert(cert.der).bytes);
      return _pinnedCertSha256.contains(fingerprint);
    },
  );
}
````

#### 6.4 `dio_client.dart` — ฉบับสมบูรณ์ Day 3 ✏️

```dart
// lib/core/network/dio_client.dart
import 'package:dio/dio.dart';
import 'package:flutter/foundation.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

import '../config/app_config.dart';
import '../security/crypto_service.dart';
import '../storage/secure_storage.dart';
import 'auth_interceptor.dart';
import 'crypto_interceptor.dart';
import 'ssl_pinning.dart';

part 'dio_client.g.dart';

/// Provider ของ Dio client
///
/// ค่า config (baseUrl, timeout) อ่านจาก [AppConfig] ที่เดียว
/// Day 3: AuthInterceptor ดึง token จริงจาก Secure Storage (หลังผู้ใช้ล็อกอิน)
/// + เปิด SSL Pinning กัน MITM (Stage 2)
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

  final storage = ref.watch(secureStorageProvider);

  // 1) แนบ Bearer token จาก Secure Storage เข้าทุก request (อ่านสดทุกครั้ง)
  dio.interceptors.add(AuthInterceptor(() => storage.readToken()));

  // 2) เข้ารหัส/ถอดรหัส payload อัตโนมัติ (AES) — ทำงานเฉพาะ payload ที่เป็น String
  //    API ปัจจุบันเป็น JSON (Map) จึงเป็น no-op ปลอดภัย (ดูหมายเหตุใน CryptoInterceptor)
  dio.interceptors.add(CryptoInterceptor(ref.watch(cryptoServiceProvider)));

  // 3) log request/response เฉพาะใน debug mode (ปิดอัตโนมัติเมื่อ build release)
  if (kDebugMode) {
    dio.interceptors.add(LogInterceptor(responseBody: true));
  }

  // 3) เปิด SSL Pinning — ปฏิเสธการเชื่อมต่อหาก certificate ไม่ตรง pin (กัน MITM)
  applySslPinning(dio);

  return dio;
}
```

> 🔗 **การเชื่อมโยง / จุดเปลี่ยนจาก Day 2 (3 จุด):**
>
> 1. `AuthInterceptor(() => storage.readToken())` — จาก demo token คงที่ → **อ่าน token จริงสด ๆ ทุก request** จาก Secure Storage (login แล้ว request ถัดไปได้ token ใหม่ทันที ไม่ต้อง rebuild Dio)
> 2. `LogInterceptor` ครอบด้วย `kDebugMode` — build release แล้ว log หายเอง (ข้อควรระวังด้านความปลอดภัยจาก Module 2)
> 3. เสียบ `CryptoInterceptor` + `applySslPinning()` — ลำดับ interceptor สำคัญ: Auth → Crypto → Log ส่วน pinning ทำที่ระดับ HttpClientAdapter (ใต้ interceptor ทั้งหมด)

### ขั้นที่ 7 — Routing: GoRouter + Route Guard + StatefulShellRoute 🆕

#### 7.1 `router_refresh.dart` — สะพาน Riverpod → Listenable

```dart
// lib/core/router/router_refresh.dart
import 'package:flutter/foundation.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

import '../../features/auth/presentation/controllers/auth_controller.dart';

part 'router_refresh.g.dart';

/// แปลงการเปลี่ยนแปลงของ [AuthController] ให้เป็น [Listenable] ที่ GoRouter ฟังได้
///
/// GoRouter จะคำนวณ redirect ใหม่ก็ต่อเมื่อมี navigation event หรือ listenable แจ้งเตือน
/// ตัวนี้คือ "สะพาน" ที่ทำให้ auth state ของ Riverpod กระตุ้น router ให้ guard ทำงาน
@riverpod
Listenable routerRefresh(RouterRefreshRef ref) {
  final notifier = ValueNotifier<int>(0);
  ref.onDispose(notifier.dispose);
  // ทุกครั้งที่ auth state เปลี่ยน → กระตุ้น notifier ให้ router คำนวณ redirect ใหม่
  ref.listen(authControllerProvider, (_, __) => notifier.value++);
  return notifier;
}
```

#### 7.2 `app_router.dart` — เส้นทางทั้งแอป + Route Guard

```dart
// lib/core/router/app_router.dart
import 'package:go_router/go_router.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

import '../../features/auth/presentation/controllers/auth_controller.dart';
import '../../features/auth/presentation/pages/forgot_password_page.dart';
import '../../features/auth/presentation/pages/login_page.dart';
import '../../features/auth/presentation/pages/pin_login_page.dart';
import '../../features/auth/presentation/pages/pin_setup_page.dart';
import '../../features/auth/presentation/pages/register_page.dart';
import '../../features/auth/presentation/pages/splash_page.dart';
import '../../features/claim/presentation/pages/claims_page.dart';
import '../../features/home/presentation/pages/home_page.dart';
import '../../features/payment/presentation/pages/payments_page.dart';
import '../../features/policy/domain/policy.dart';
import '../../features/policy/presentation/pages/policy_detail_page.dart';
import '../../features/policy/presentation/pages/policy_list_page.dart';
import '../../features/profile/presentation/pages/change_password_page.dart';
import '../../features/profile/presentation/pages/edit_profile_page.dart';
import '../../features/profile/presentation/pages/profile_page.dart';
import 'home_shell.dart';
import 'router_refresh.dart';

part 'app_router.g.dart';

/// GoRouter หลักของแอป + Route Guard (redirect) ที่ผูกกับ Auth State
@riverpod
GoRouter appRouter(AppRouterRef ref) {
  return GoRouter(
    initialLocation: '/splash',
    // ให้ router คำนวณ redirect ใหม่ทุกครั้งที่ auth state เปลี่ยน
    refreshListenable: ref.watch(routerRefreshProvider),
    redirect: (context, state) {
      final auth = ref.read(authControllerProvider);
      final loc = state.matchedLocation;
      final onSplash = loc == '/splash';
      final onLogin = loc == '/login';

      // ระหว่างกู้ session หรือกำลัง login
      if (auth.isLoading) {
        // ถ้ากำลัง loading อยู่ที่หน้า login (กำลัง submit form) → อยู่หน้า login ต่อ
        if (onLogin) return null;
        // กรณีอื่น (กำลังกู้ session ตอนเปิดแอป) → แสดงหน้า Splash
        return onSplash ? null : '/splash';
      }

      final isLoggedIn = auth.valueOrNull != null;
      // ยังไม่ล็อกอิน → ไปหน้า login (ยกเว้นอยู่หน้า login อยู่แล้ว)
      if (!isLoggedIn) return onLogin ? null : '/login';
      // ล็อกอินแล้วแต่ยังค้างที่ splash/login → เข้าแอป
      if (onSplash || onLogin) return '/home';
      return null;
    },
    routes: [
      // หน้า Splash — แสดงระหว่างกู้ session (อยู่นอก Shell)
      GoRoute(
        path: '/splash',
        builder: (context, state) => const SplashPage(),
      ),
      // หน้า Login อยู่นอก Shell (ไม่มี Bottom Navigation)
      GoRoute(
        path: '/login',
        builder: (context, state) => const LoginPage(),
      ),
      // หน้าลืมรหัสผ่าน — fullscreen
      GoRoute(
        path: '/forgot-password',
        builder: (context, state) => const ForgotPasswordPage(),
      ),
      // หน้าสมัครสมาชิก — fullscreen
      GoRoute(
        path: '/register',
        builder: (context, state) => const RegisterPage(),
      ),
      // หน้าตั้งค่า PIN — fullscreen ไม่มี bottom nav
      GoRoute(
        path: '/pin-setup',
        builder: (context, state) => const PinSetupPage(),
      ),
      // หน้า Login ด้วย PIN — fullscreen ไม่มี bottom nav
      GoRoute(
        path: '/pin-login',
        builder: (context, state) => const PinLoginPage(),
      ),
      // หน้าแก้ไขข้อมูลส่วนตัว — fullscreen ไม่มี bottom nav
      GoRoute(
        path: '/edit-profile',
        builder: (context, state) => const EditProfilePage(),
      ),
      // หน้าเปลี่ยนรหัสผ่าน — fullscreen ไม่มี bottom nav
      GoRoute(
        path: '/change-password',
        builder: (context, state) => const ChangePasswordPage(),
      ),
      // หน้ารายละเอียดกรมธรรม์ — fullscreen ไม่มี bottom nav
      GoRoute(
        path: '/policy/:id',
        builder: (context, state) =>
            PolicyDetailPage(policy: state.extra as Policy),
      ),
      // Bottom Navigation 5 แท็บ — แต่ละแท็บมี stack แยกของตัวเอง
      StatefulShellRoute.indexedStack(
        builder: (context, state, navigationShell) =>
            HomeShell(navigationShell: navigationShell),
        branches: [
          StatefulShellBranch(routes: [
            GoRoute(path: '/home', builder: (c, s) => const HomePage()),
          ]),
          StatefulShellBranch(routes: [
            GoRoute(
              path: '/policies',
              builder: (c, s) => const PolicyListPage(),
            ),
          ]),
          StatefulShellBranch(routes: [
            GoRoute(path: '/claims', builder: (c, s) => const ClaimsPage()),
          ]),
          StatefulShellBranch(routes: [
            GoRoute(path: '/payments', builder: (c, s) => const PaymentsPage()),
          ]),
          StatefulShellBranch(routes: [
            GoRoute(path: '/profile', builder: (c, s) => const ProfilePage()),
          ]),
        ],
      ),
    ],
  );
}
```

#### 7.3 `home_shell.dart` — เปลือก Bottom Navigation (แทน MainShell เดิม)

```dart
// lib/core/router/home_shell.dart
// core/router/home_shell.dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

import '../icons/app_icons.dart';
import '../theme/app_colors.dart';

/// เชลล์หลักของ Bottom Navigation — ขับเคลื่อนด้วย GoRouter + StatefulShellRoute
/// แต่ละแท็บมี stack (ประวัติหน้า) แยกของตัวเอง และคงสถานะไว้เมื่อสลับแท็บ
class HomeShell extends StatelessWidget {
  final StatefulNavigationShell navigationShell;
  const HomeShell({super.key, required this.navigationShell});

  static const _items = [
    (
      icon: HugeIcons.strokeRoundedHome03,
      active: HugeIcons.strokeRoundedHome03,
      label: 'หน้าแรก',
    ),
    (
      icon: HugeIcons.strokeRoundedShield01,
      active: HugeIcons.strokeRoundedShield01,
      label: 'กรมธรรม์',
    ),
    (
      icon: HugeIcons.strokeRoundedFile01,
      active: HugeIcons.strokeRoundedFile01,
      label: 'สินไหม',
    ),
    (
      icon: HugeIcons.strokeRoundedCreditCard,
      active: HugeIcons.strokeRoundedCreditCard,
      label: 'ชำระเบี้ย',
    ),
    (
      icon: HugeIcons.strokeRoundedUser,
      active: HugeIcons.strokeRoundedUser,
      label: 'โปรไฟล์',
    ),
  ];

  void _onTap(int index) {
    // กดแท็บเดิมซ้ำ → เด้งกลับ root ของแท็บนั้น
    navigationShell.goBranch(
      index,
      initialLocation: index == navigationShell.currentIndex,
    );
  }

  @override
  Widget build(BuildContext context) {
    final index = navigationShell.currentIndex;
    return Scaffold(
      body: navigationShell, // เนื้อหาของแท็บที่เลือกอยู่
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
                    onTap: () => _onTap(i),
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

  const _NavItem({
    required this.item,
    required this.selected,
    required this.onTap,
  });

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
                color: color,
              ),
              const SizedBox(height: 3),
              Text(
                item.label,
                style: TextStyle(
                  fontSize: 10.5,
                  color: color,
                  fontWeight: selected ? FontWeight.w600 : FontWeight.w500,
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

```

> 🔗 **การเชื่อมโยง / จุดสังเกตของระบบ Routing:**
>
> - `main_shell.dart` + `mainTabProvider` (Day 1–2) **ถูกแทนที่** ด้วย `HomeShell` + `StatefulShellRoute.indexedStack` — แต่ละแท็บมี stack ของตัวเอง และการสลับแท็บกลายเป็น URL (`/home`, `/policies`, ...) ที่ deep-link ได้
> - ลอจิก guard อ่านง่าย: **loading → /splash** (ยกเว้นกำลัง submit ที่หน้า login), **ไม่มี user → /login**, **มี user แต่อยู่ splash/login → /home** — และถูกเรียกใหม่อัตโนมัติเมื่อ auth เปลี่ยนผ่าน `routerRefreshProvider`
> - `/policy/:id` ส่ง object ผ่าน `state.extra` (Day 3 ฉบับนี้) — path param `:id` เตรียมไว้สำหรับแบบฝึกหัด "ดึงจาก provider ด้วย id" ในความท้าทายเสริม
> - หน้าแรกของแอปเปลี่ยนวิธีคิด: ไม่มีใคร `Navigator.push` ไป Home อีกแล้ว — ทุกอย่างคือ **ผลof redirect** จาก auth state (declarative navigation)

### ขั้นที่ 8 — main.dart ฉบับสมบูรณ์ ✏️ (Router + i18n + ThemeMode + prefs override)

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:shared_preferences/shared_preferences.dart';

import 'core/router/app_router.dart';
import 'core/settings/settings_providers.dart';
import 'core/theme/app_colors.dart';
import 'core/theme/app_palette.dart';
import 'core/theme/app_theme.dart';
import 'l10n/app_localizations.dart';

Future<void> main() async {
  // ต้อง ensureInitialized ก่อนเรียก plugin (SharedPreferences) นอก runApp
  WidgetsFlutterBinding.ensureInitialized();

  // โหลดค่าตั้งค่าครั้งเดียว แล้วฉีดเข้า Riverpod ผ่าน override
  // (รูปแบบมาตรฐาน: ให้ provider อื่นอ่าน prefs แบบ synchronous ได้)
  final prefs = await SharedPreferences.getInstance();

  runApp(
    ProviderScope(
      overrides: [
        sharedPreferencesProvider.overrideWithValue(prefs),
      ],
      child: const BlaApp(),
    ),
  );
}

/// Day 3: MaterialApp.router + GoRouter (Route Guard ผูกกับ Auth State)
/// + i18n (ไทย/อังกฤษ) + สลับโหมดสว่าง/มืด (ThemeMode) แบบ reactive ผ่าน Riverpod
class BlaApp extends ConsumerWidget {
  const BlaApp({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final router = ref.watch(appRouterProvider);
    final locale = ref.watch(localeControllerProvider);
    final themeMode = ref.watch(themeModeControllerProvider);

    return MaterialApp.router(
      title: 'BLA Policy Companion',
      debugShowCheckedModeBanner: false,
      // ธีมสว่าง/มืด — เปลี่ยนตาม themeModeControllerProvider
      theme: AppTheme.light,
      darkTheme: AppTheme.dark,
      themeMode: themeMode,
      // i18n — ภาษาปัจจุบันจาก localeControllerProvider
      locale: locale,
      supportedLocales: AppLocalizations.supportedLocales,
      localizationsDelegates: AppLocalizations.localizationsDelegates,
      routerConfig: router,
      // scrim ทึบคลุมพื้นที่ status bar ทุกหน้า: กันเนื้อหาเลื่อนขึ้นไปซ้อนนาฬิกา/ไอคอนระบบ
      // และคง status bar เป็นแบรนด์ + ปรับ navigation bar ตามธีม
      builder: (context, child) {
        final topInset = MediaQuery.paddingOf(context).top;
        final isDark = Theme.of(context).brightness == Brightness.dark;
        return AnnotatedRegion<SystemUiOverlayStyle>(
          value: SystemUiOverlayStyle.light.copyWith(
            statusBarColor: Colors.transparent,
            systemNavigationBarColor: context.palette.card,
            systemNavigationBarIconBrightness:
                isDark ? Brightness.light : Brightness.dark,
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

> 🔗 **การเชื่อมโยง:** `BlaApp` เป็น `ConsumerWidget` ที่ watch 3 provider — `appRouter` (นำทาง), `localeController` (ภาษา), `themeModeController` (ธีม) — กดสลับภาษา/ธีมในหน้าโปรไฟล์แล้ว **ทั้งแอป rebuild ด้วยค่าใหม่ทันที** · `overrideWithValue(prefs)` คือแพตเทิร์นเดียวกับที่ใช้ mock ใน unit test (ขั้นที่ 13) — override เพื่อ "ฉีดค่าจริง" ก็ได้ "ฉีดของปลอม" ก็ได้

### ขั้นที่ 9 — Platform Channel: Biometric จริง (Dart ↔ Kotlin/Swift) 🆕

#### 9.1 ฝั่ง Dart — `device_info_service.dart`

```dart
// lib/core/platform/device_info_service.dart
import 'package:flutter/services.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'device_info_service.g.dart';

/// เรียกความสามารถ Native ผ่าน **Platform Channel (MethodChannel)**
///
/// ชื่อ channel ต้องตรงกันทั้งฝั่ง Dart และฝั่ง Native (Kotlin/Swift)
class DeviceInfoService {
  static const _channel = MethodChannel('com.bla.policy/device');

  /// ยืนยันตัวตนด้วย Biometric (ลายนิ้วมือ/ใบหน้า) — คืน true ถ้าผ่าน
  Future<bool> authenticateBiometric() async {
    try {
      final ok = await _channel.invokeMethod<bool>('authenticateBiometric');
      return ok ?? false;
    } on PlatformException {
      // ผู้ใช้ยกเลิก / ยืนยันไม่ผ่าน / อุปกรณ์ไม่รองรับ
      return false;
    } on MissingPluginException {
      // ยังไม่ได้ทำฝั่ง Native ของแพลตฟอร์มนี้ (เช่น iOS ที่ยังไม่ wire)
      throw Exception('อุปกรณ์นี้ยังไม่รองรับการยืนยันด้วย Biometric');
    }
  }

  /// ตัวอย่าง: ดึง Device ID จาก Native (สาธิตการรับค่ากลับจาก MethodChannel)
  Future<String> getDeviceId() async {
    try {
      final id = await _channel.invokeMethod<String>('getDeviceId');
      return id ?? 'unknown';
    } on PlatformException {
      return 'unknown';
    }
  }
}

@riverpod
DeviceInfoService deviceInfoService(DeviceInfoServiceRef ref) =>
    DeviceInfoService();
```

#### 9.2 ฝั่ง Android — `MainActivity.kt` (BiometricPrompt จริง) ✏️

ต้องเพิ่ม dependency ใน `android/app/build.gradle.kts` ด้วย:

```kotlin
dependencies {
    implementation("androidx.biometric:biometric:1.2.0-alpha05")
}
```

```kotlin
// android/app/src/main/kotlin/com/example/bla_policy_companion/MainActivity.kt
package com.example.bla_policy_companion

import android.os.Build
import androidx.biometric.BiometricManager
import androidx.biometric.BiometricPrompt
import androidx.core.content.ContextCompat
import io.flutter.embedding.android.FlutterFragmentActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.MethodChannel
import java.util.concurrent.Executor

/**
 * ฝั่ง Native Android (Kotlin) ของ Platform Channel
 *
 * รับ method call จากฝั่ง Dart (DeviceInfoService) ผ่าน channel ชื่อเดียวกัน
 * "com.bla.policy/device"
 */
class MainActivity : FlutterFragmentActivity() {
    private val channelName = "com.bla.policy/device"

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, channelName)
            .setMethodCallHandler { call, result ->
                when (call.method) {
                    "getDeviceId" -> {
                        result.success("ANDROID-${Build.MODEL}-${Build.ID}")
                    }
                    "authenticateBiometric" -> {
                        // ==== โค้ดเก่า (Mock) ====
                        // result.success(true)
                        // ==========================

                        // ==== โค้ดจริง: ใช้ BiometricPrompt ====
                        authenticateWithBiometric(result)
                    }
                    else -> result.notImplemented()
                }
            }
    }

    /**
     * แสดง BiometricPrompt ให้ผู้ใช้ยืนยันตัวตนด้วยลายนิ้วมือหรือใบหน้า
     */
    private fun authenticateWithBiometric(result: MethodChannel.Result) {
        // ตรวจสอบว่าอุปกรณ์รองรับ biometric หรือไม่
        val biometricManager = BiometricManager.from(this)
        when (biometricManager.canAuthenticate(BiometricManager.Authenticators.BIOMETRIC_STRONG)) {
            BiometricManager.BIOMETRIC_SUCCESS -> {
                // รองรับ → แสดง prompt
                showBiometricPrompt(result)
            }
            BiometricManager.BIOMETRIC_ERROR_NO_HARDWARE -> {
                result.error("NO_HARDWARE", "อุปกรณ์ไม่มีเซ็นเซอร์ biometric", null)
            }
            BiometricManager.BIOMETRIC_ERROR_HW_UNAVAILABLE -> {
                result.error("HW_UNAVAILABLE", "เซ็นเซอร์ biometric ไม่พร้อมใช้งาน", null)
            }
            BiometricManager.BIOMETRIC_ERROR_NONE_ENROLLED -> {
                result.error("NONE_ENROLLED", "ยังไม่ได้ลงทะเบียนลายนิ้วมือหรือใบหน้า", null)
            }
            else -> {
                result.error("UNKNOWN", "ไม่สามารถใช้งาน biometric ได้", null)
            }
        }
    }

    /**
     * แสดง BiometricPrompt dialog
     */
    private fun showBiometricPrompt(result: MethodChannel.Result) {
        val executor: Executor = ContextCompat.getMainExecutor(this)

        val biometricPrompt = BiometricPrompt(this, executor,
            object : BiometricPrompt.AuthenticationCallback() {
                override fun onAuthenticationError(errorCode: Int, errString: CharSequence) {
                    super.onAuthenticationError(errorCode, errString)
                    // ผู้ใช้กดยกเลิก หรือเกิด error
                    result.success(false)
                }

                override fun onAuthenticationSucceeded(authResult: BiometricPrompt.AuthenticationResult) {
                    super.onAuthenticationSucceeded(authResult)
                    // ยืนยันตัวตนสำเร็จ
                    result.success(true)
                }

                override fun onAuthenticationFailed() {
                    super.onAuthenticationFailed()
                    // ลายนิ้วมือไม่ตรง แต่ยังให้ลองใหม่ได้ (ไม่ต้อง result ที่นี่)
                }
            })

        val promptInfo = BiometricPrompt.PromptInfo.Builder()
            .setTitle("ยืนยันตัวตนเพื่อเข้าสู่ระบบ")
            .setSubtitle("สแกนลายนิ้วมือหรือใบหน้าของคุณ")
            .setNegativeButtonText("ยกเลิก")
            .build()

        biometricPrompt.authenticate(promptInfo)
    }
}
```

#### 9.3 ฝั่ง iOS — `AppDelegate.swift` ✏️

```swift
// ios/Runner/AppDelegate.swift
import Flutter
import UIKit

@main
@objc class AppDelegate: FlutterAppDelegate, FlutterImplicitEngineDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }

  func didInitializeImplicitFlutterEngine(_ engineBridge: FlutterImplicitEngineBridge) {
    GeneratedPluginRegistrant.register(with: engineBridge.pluginRegistry)

    // ── Platform Channel (Native iOS) — channel "com.bla.policy/device" ──
    // ใช้ messenger จาก registrar ของ pluginRegistry (รูปแบบ implicit engine ใหม่ของ Flutter)
    // หมายเหตุ: ถ้า API ต่างจากนี้ในเวอร์ชัน Flutter ที่ใช้ ให้ปรับวิธีดึง binaryMessenger
    if let messenger = engineBridge.pluginRegistry
      .registrar(forPlugin: "com.bla.policy/device")?.messenger() {
      let channel = FlutterMethodChannel(
        name: "com.bla.policy/device",
        binaryMessenger: messenger
      )
      channel.setMethodCallHandler { (call, result) in
        switch call.method {
        case "getDeviceId":
          let id = UIDevice.current.identifierForVendor?.uuidString ?? "unknown"
          result("IOS-\(UIDevice.current.model)-\(id)")
        case "authenticateBiometric":
          // สาธิต: คืน true ทันที
          // production จริงให้ใช้ LocalAuthentication (LAContext.evaluatePolicy)
          // แล้วเรียก result(true/false) ใน callback บน main thread
          result(true)
        default:
          result(FlutterMethodNotImplemented)
        }
      }
    }
  }
}
```

> 🔗 **การเชื่อมโยง / จุดสังเกต:**
>
> - เส้นทางการเรียก: `LoginPage._biometricLogin()` → `BiometricController.authenticate()` → `DeviceInfoService` (MethodChannel `com.bla.policy/device`) → **Kotlin BiometricPrompt จริง** (Android) / **mock true** (iOS — production ใช้ `LAContext`)
> - ฝั่ง Android ต้องเปลี่ยน base class เป็น `FlutterFragmentActivity` (BiometricPrompt ต้องการ FragmentActivity) และเพิ่ม dependency `androidx.biometric`
> - ข้อสังเกตการออกแบบ: `onAuthenticationFailed` (นิ้วไม่ตรง) **ไม่เรียก result** เพื่อให้ผู้ใช้ลองใหม่ใน dialog เดิมได้ ส่วน `onAuthenticationError` (กดยกเลิก) คืน `false` — เพราะ `MethodChannel.Result` เรียกซ้ำไม่ได้ (แอปจะ crash)

### ขั้นที่ 10 — หน้า Auth ทั้ง 7 หน้า (ต่อ API จริง + l10n)

#### 10.1 `splash_page.dart` ✏️ — จาก StatefulWidget+Timer → StatelessWidget ล้วน

```dart
// lib/features/auth/presentation/pages/splash_page.dart
import 'package:flutter/material.dart';

import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_logo.dart';

/// หน้า Splash — โลโก้กลางจอบนพื้น gradient + แถบโหลด
///
/// Day 3: เป็น "หน้าจอเปล่า" ที่แสดงระหว่างรอกู้ session (อ่าน token / GET /auth/me)
/// การเด้งไป /home หรือ /login ทำโดย Route Guard (redirect) ใน app_router — ไม่ navigate เอง
class SplashPage extends StatelessWidget {
  const SplashPage({super.key});

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

> 🔗 **จุดเปลี่ยนเชิงแนวคิด:** Day 1 Splash มี `Timer` + `pushReplacement` เอง — Day 3 มัน **ไม่รู้ด้วยซ้ำว่าจะไปไหนต่อ**: AuthController กู้ session อยู่เบื้องหลัง เมื่อรู้ผล router redirect ให้เอง (Splash เป็นแค่ view ของสถานะ loading)

#### 10.2 `login_page.dart` ✏️ — 2 โหมด: Quick Sign-in (Biometric/PIN) กับฟอร์มรหัสผ่าน

```dart
// lib/features/auth/presentation/pages/login_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/l10n/l10n_ext.dart';
import '../../../../l10n/app_localizations.dart';
import '../../../../core/storage/secure_storage.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_spacing.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_logo.dart';
import '../../../../core/widgets/app_text_field.dart';
import '../../../../core/widgets/info_hint.dart';
import '../controllers/auth_controller.dart';
import '../controllers/biometric_controller.dart';

/// หน้าเข้าสู่ระบบ
/// Day 3: ต่อ Reactive Auth จริง — login ผ่าน API, เก็บ token ใน Secure Storage,
/// แล้ว Route Guard เด้งเข้าแอปเองอัตโนมัติ (ไม่ต้อง navigate เอง)
class LoginPage extends ConsumerStatefulWidget {
  const LoginPage({super.key});

  @override
  ConsumerState<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends ConsumerState<LoginPage> {
  final _user = TextEditingController(text: 'customer');
  final _pass = TextEditingController(text: '1234');
  bool? _hasSavedCredentials;

  @override
  void initState() {
    super.initState();
    _checkSavedCredentials();
  }

  Future<void> _checkSavedCredentials() async {
    final hasCreds = await ref.read(secureStorageProvider).hasBiometricCredentials();
    if (mounted) {
      setState(() {
        _hasSavedCredentials = hasCreds;
      });
    }
  }

  @override
  void dispose() {
    _user.dispose();
    _pass.dispose();
    super.dispose();
  }

  void _login() {
    // จำ credentials อัตโนมัติ
    ref.read(authControllerProvider.notifier).login(
      _user.text.trim(),
      _pass.text,
      rememberForBiometric: true,
    );
  }

  void _notSupported() {
    ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(context.l10n.demoUsernameOnly)));
  }

  /// ยืนยันตัวตนด้วย Biometric แล้วเข้าสู่ระบบ
  Future<void> _biometricLogin() async {
    final ok = await ref.read(biometricControllerProvider.notifier).authenticate();
    if (!mounted) return;

    if (ok) {
      final storage = ref.read(secureStorageProvider);
      final credentials = await storage.getBiometricCredentials();

      if (credentials != null) {
        ref.read(authControllerProvider.notifier).login(
          credentials.username,
          credentials.password,
          rememberForBiometric: true,
        );
      }
    } else {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(context.l10n.biometricAuthFailed)));
    }
  }

  /// เปิดหน้า PIN login
  void _pinLogin() {
    context.push('/pin-login');
  }

  /// กลับไปใช้ฟอร์ม login ธรรมดา
  void _usePasswordLogin() {
    setState(() {
      _hasSavedCredentials = false;
    });
  }

  @override
  Widget build(BuildContext context) {
    final l = context.l10n;
    final authState = ref.watch(authControllerProvider);
    final loading = authState.isLoading;
    final hasError = authState.hasError && !authState.isLoading;

    // รอ check credentials เสร็จก่อน
    if (_hasSavedCredentials == null) {
      return const Scaffold(
        body: Center(child: CircularProgressIndicator()),
      );
    }

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
                      l.loginSubtitle,
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
            child: _hasSavedCredentials == true
                ? _buildQuickLoginOptions(l)
                : _buildPasswordLoginForm(l, loading, hasError, authState),
          ),
        ],
      ),
    );
  }

  /// UI สำหรับผู้ใช้ที่มี credentials บันทึกไว้ (หลัง logout)
  Widget _buildQuickLoginOptions(AppLocalizations l) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.stretch,
      children: [
        Text(
          l.quickSignInTitle,
          style: AppType.h2,
          textAlign: TextAlign.center,
        ),
        const SizedBox(height: 32),
        AppButton.primary(
          label: l.scanFingerprint,
          icon: HugeIcons.strokeRoundedFingerPrint,
          onPressed: _biometricLogin,
        ),
        const SizedBox(height: 16),
        AppButton.secondary(
          label: l.usePin,
          icon: HugeIcons.strokeRoundedFingerPrintScan,
          onPressed: _pinLogin,
        ),
        const SizedBox(height: 32),
        Center(
          child: TextButton(
            onPressed: _usePasswordLogin,
            child: Text(
              l.signInWithPassword,
              style: AppType.bodyStrong.copyWith(color: AppColors.primary),
            ),
          ),
        ),
      ],
    );
  }

  /// UI สำหรับ login ครั้งแรก (แสดงฟอร์ม)
  Widget _buildPasswordLoginForm(
    AppLocalizations l,
    bool loading,
    bool hasError,
    AsyncValue<dynamic> authState,
  ) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.stretch,
      children: [
        AppTextField(
          label: l.usernameLabel,
          hint: 'customer',
          icon: HugeIcons.strokeRoundedUser,
          controller: _user,
        ),
        const SizedBox(height: 14),
        AppTextField(
          label: l.passwordLabel,
          hint: '••••',
          icon: HugeIcons.strokeRoundedSquareLock01,
          obscure: true,
          controller: _pass,
        ),
        const SizedBox(height: 12),
        InfoHint(l.testAccountInfo),
        if (hasError) ...[
          const SizedBox(height: 12),
          Container(
            padding: const EdgeInsets.symmetric(horizontal: 14, vertical: 10),
            decoration: BoxDecoration(
              color: Colors.red.shade50,
              border: Border.all(color: Colors.red.shade300),
              borderRadius: BorderRadius.circular(8),
            ),
            child: Row(
              children: [
                Icon(Icons.error_outline,
                    size: 20, color: Colors.red.shade700),
                const SizedBox(width: 10),
                Expanded(
                  child: Text(
                    '${authState.error}',
                    style: AppType.body.copyWith(
                      color: Colors.red.shade700,
                      fontSize: 13,
                    ),
                  ),
                ),
              ],
            ),
          ),
        ],
        const SizedBox(height: 18),
        AppButton.primary(
          label: loading ? l.signingIn : l.signIn,
          icon: HugeIcons.strokeRoundedLogin03,
          onPressed: loading ? null : _login,
        ),
        const SizedBox(height: 12),
        Center(
          child: TextButton(
            onPressed: () => context.push('/forgot-password'),
            child: Text(l.forgotPasswordLink,
                style: AppType.bodyStrong
                    .copyWith(color: AppColors.primary)),
          ),
        ),
        _OrDivider(label: l.orContinueWith),
        const SizedBox(height: 16),
        AppButton.social(
          label: l.continueWithGoogle,
          icon: HugeIcons.strokeRoundedGoogle,
          color: const Color(0xFF4285F4),
          onPressed: _notSupported,
        ),
        const SizedBox(height: 12),
        AppButton.social(
          label: l.continueWithLine,
          icon: HugeIcons.strokeRoundedBubbleChat,
          color: const Color(0xFF06C755),
          onPressed: _notSupported,
        ),
        _OrDivider(label: l.noAccountYet),
        const SizedBox(height: 16),
        AppButton.secondary(
          label: l.signUp,
          icon: HugeIcons.strokeRoundedUserAdd01,
          onPressed: () => context.push('/register'),
        ),
        const SizedBox(height: 20),
        Center(
          child: Text(l.appFooter, style: AppType.caption),
        ),
      ],
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

> 🔗 **การเชื่อมโยง / จุดสังเกต:**
>
> - เปิดหน้ามาเช็ค `hasBiometricCredentials()` ก่อน → เคย login แล้วเห็น **Quick Sign-in** (สแกนนิ้ว/PIN), ครั้งแรกเห็นฟอร์ม — สลับกลับได้ด้วยปุ่ม "เข้าสู่ระบบด้วยรหัสผ่าน"
> - ปุ่มเข้าสู่ระบบเรียก `login()` เฉย ๆ **ไม่มีการ navigate แม้แต่บรรทัดเดียว** — สำเร็จแล้ว state เปลี่ยน → `routerRefresh` → guard พาเข้า `/home` เอง (จุดที่ต่างจาก Day 1 ชัดที่สุด)
> - error จาก server (`ApiException` เช่น "ชื่อผู้ใช้หรือรหัสผ่านไม่ถูกต้อง") แสดงเป็นกล่องแดงในฟอร์มผ่าน `authState.hasError` — ไม่ใช่ SnackBar เพราะผู้ใช้ต้องเห็นค้างไว้ขณะแก้รหัส
> - ทุกข้อความผ่าน `context.l10n` (สลับไทย/EN ได้ทั้งหน้า)

#### 10.3 `pin_setup_page.dart` 🆕 — ตั้ง PIN 6 หลัก (กรอก 2 รอบยืนยัน)

```dart
// lib/features/auth/presentation/pages/pin_setup_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/l10n/l10n_ext.dart';
import '../../../../core/storage/secure_storage.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_typography.dart';

/// หน้าตั้งค่า PIN 6 หลัก
class PinSetupPage extends ConsumerStatefulWidget {
  const PinSetupPage({super.key});

  @override
  ConsumerState<PinSetupPage> createState() => _PinSetupPageState();
}

class _PinSetupPageState extends ConsumerState<PinSetupPage> {
  String _pin = '';
  String? _confirmPin;
  bool _isConfirming = false;

  void _onNumberTap(String number) {
    if (_pin.length >= 6) return;
    setState(() {
      _pin += number;
      if (_pin.length == 6) {
        if (!_isConfirming) {
          // เข้าโหมดยืนยัน PIN
          _isConfirming = true;
          _confirmPin = _pin;
          _pin = '';
        } else {
          // ตรวจสอบ PIN ตรงกันหรือไม่
          _verifyAndSave();
        }
      }
    });
  }

  void _onBackspace() {
    if (_pin.isEmpty) return;
    setState(() {
      _pin = _pin.substring(0, _pin.length - 1);
    });
  }

  Future<void> _verifyAndSave() async {
    if (_pin != _confirmPin) {
      // PIN ไม่ตรงกัน
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(context.l10n.pinMismatch)),
      );
      setState(() {
        _pin = '';
        _confirmPin = null;
        _isConfirming = false;
      });
      return;
    }

    // บันทึก PIN
    await ref.read(secureStorageProvider).savePin(_pin);
    if (!mounted) return;
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(context.l10n.pinSetupSuccess)),
    );
    Navigator.of(context).pop(true);
  }

  @override
  Widget build(BuildContext context) {
    final l = context.l10n;
    return Scaffold(
      backgroundColor: AppColors.background,
      resizeToAvoidBottomInset: false,
      body: SafeArea(
        child: SingleChildScrollView(
          child: SizedBox(
            height: MediaQuery.of(context).size.height -
                MediaQuery.of(context).padding.top -
                MediaQuery.of(context).padding.bottom,
            child: Column(
              children: [
                // Header
                Padding(
                  padding: const EdgeInsets.all(16),
                  child: Row(
                    children: [
                      IconButton(
                        icon: const Icon(Icons.arrow_back),
                        onPressed: () => Navigator.of(context).pop(),
                      ),
                    ],
                  ),
                ),
                const Spacer(),
                // Icon
                Container(
                  width: 100,
                  height: 100,
                  decoration: BoxDecoration(
                    color: AppColors.primary.withOpacity(0.1),
                    shape: BoxShape.circle,
                  ),
                  child: const HugeIcon(
                    icon: HugeIcons.strokeRoundedFingerPrintScan,
                    size: 50,
                    color: AppColors.primary,
                  ),
                ),
                const SizedBox(height: 24),
                Text(
                  _isConfirming ? l.confirmPinTitle : l.enterPinTitle,
                  style: AppType.h2,
                ),
                const SizedBox(height: 8),
                Text(
                  _isConfirming ? l.confirmPinSubtitle : l.enterPinSubtitle,
                  style: AppType.body.copyWith(color: AppColors.textSubtle),
                ),
                const SizedBox(height: 32),
                // PIN Dots
                Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: List.generate(
                    6,
                    (index) => Container(
                      width: 16,
                      height: 16,
                      margin: const EdgeInsets.symmetric(horizontal: 8),
                      decoration: BoxDecoration(
                        color: index < _pin.length
                            ? AppColors.primary
                            : AppColors.border,
                        shape: BoxShape.circle,
                      ),
                    ),
                  ),
                ),
                const SizedBox(height: 16),
                Text(
                  l.testPinInfo,
                  style: AppType.caption.copyWith(color: AppColors.textSubtle),
                ),
                const Spacer(),
                // PIN Pad
                _PinPad(
                  onNumberTap: _onNumberTap,
                  onBackspace: _onBackspace,
                ),
                const SizedBox(height: 40),
              ],
            ),
          ),
        ),
      ),
    );
  }
}

/// PIN Keyboard
class _PinPad extends StatelessWidget {
  final void Function(String) onNumberTap;
  final VoidCallback onBackspace;

  const _PinPad({
    required this.onNumberTap,
    required this.onBackspace,
  });

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 40),
      child: Column(
        children: [
          _buildRow(['1', '2', '3']),
          const SizedBox(height: 20),
          _buildRow(['4', '5', '6']),
          const SizedBox(height: 20),
          _buildRow(['7', '8', '9']),
          const SizedBox(height: 20),
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            children: [
              const SizedBox(width: 70, height: 70), // Spacer
              _buildButton('0'),
              SizedBox(
                width: 70,
                height: 70,
                child: IconButton(
                  icon: const Icon(Icons.backspace_outlined),
                  onPressed: onBackspace,
                ),
              ),
            ],
          ),
        ],
      ),
    );
  }

  Widget _buildRow(List<String> numbers) {
    return Row(
      mainAxisAlignment: MainAxisAlignment.spaceEvenly,
      children: numbers.map(_buildButton).toList(),
    );
  }

  Widget _buildButton(String number) {
    return SizedBox(
      width: 70,
      height: 70,
      child: TextButton(
        onPressed: () => onNumberTap(number),
        style: TextButton.styleFrom(
          backgroundColor: AppColors.card,
          shape: const CircleBorder(),
        ),
        child: Text(
          number,
          style: AppType.h2.copyWith(fontSize: 24),
        ),
      ),
    );
  }
}
```

#### 10.4 `pin_login_page.dart` 🆕 — เข้าสู่ระบบด้วย PIN

```dart
// lib/features/auth/presentation/pages/pin_login_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/l10n/l10n_ext.dart';
import '../../../../core/storage/secure_storage.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_typography.dart';
import '../controllers/auth_controller.dart';

/// หน้า login ด้วย PIN 6 หลัก
class PinLoginPage extends ConsumerStatefulWidget {
  const PinLoginPage({super.key});

  @override
  ConsumerState<PinLoginPage> createState() => _PinLoginPageState();
}

class _PinLoginPageState extends ConsumerState<PinLoginPage> {
  String _pin = '';
  bool _isError = false;

  void _onNumberTap(String number) {
    if (_pin.length >= 6) return;
    setState(() {
      _pin += number;
      _isError = false;
      if (_pin.length == 6) {
        _verifyPin();
      }
    });
  }

  void _onBackspace() {
    if (_pin.isEmpty) return;
    setState(() {
      _pin = _pin.substring(0, _pin.length - 1);
      _isError = false;
    });
  }

  Future<void> _verifyPin() async {
    final storage = ref.read(secureStorageProvider);
    final isValid = await storage.verifyPin(_pin);

    if (!mounted) return;

    if (isValid) {
      // PIN ถูกต้อง → ดึง credentials และ login
      final credentials = await storage.getBiometricCredentials();
      if (credentials != null) {
        ref.read(authControllerProvider.notifier).login(
              credentials.username,
              credentials.password,
            );
      }
    } else {
      // PIN ผิด
      setState(() {
        _isError = true;
        _pin = '';
      });
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(context.l10n.incorrectPin)),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    final l = context.l10n;
    return Scaffold(
      backgroundColor: AppColors.background,
      resizeToAvoidBottomInset: false,
      body: SafeArea(
        child: SingleChildScrollView(
          child: SizedBox(
            height: MediaQuery.of(context).size.height -
                MediaQuery.of(context).padding.top -
                MediaQuery.of(context).padding.bottom,
            child: Column(
              children: [
                // Header
                Padding(
                  padding: const EdgeInsets.all(16),
                  child: Row(
                    children: [
                      IconButton(
                        icon: const Icon(Icons.arrow_back),
                        onPressed: () => Navigator.of(context).pop(),
                      ),
                    ],
                  ),
                ),
                const Spacer(),
                // Icon
                Container(
                  width: 100,
                  height: 100,
                  decoration: BoxDecoration(
                    color: AppColors.primary.withOpacity(0.1),
                    shape: BoxShape.circle,
                  ),
                  child: const HugeIcon(
                    icon: HugeIcons.strokeRoundedFingerPrintScan,
                    size: 50,
                    color: AppColors.primary,
                  ),
                ),
                const SizedBox(height: 24),
                Text(
                  l.enterPinTitle,
                  style: AppType.h2,
                ),
                const SizedBox(height: 8),
                Text(
                  l.enterPinSubtitle,
                  style: AppType.body.copyWith(color: AppColors.textSubtle),
                ),
                const SizedBox(height: 32),
                // PIN Dots
                Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: List.generate(
                    6,
                    (index) => Container(
                      width: 16,
                      height: 16,
                      margin: const EdgeInsets.symmetric(horizontal: 8),
                      decoration: BoxDecoration(
                        color: index < _pin.length
                            ? (_isError ? Colors.red : AppColors.primary)
                            : AppColors.border,
                        shape: BoxShape.circle,
                      ),
                    ),
                  ),
                ),
                const SizedBox(height: 16),
                Text(
                  l.testPinInfo,
                  style: AppType.caption.copyWith(color: AppColors.textSubtle),
                ),
                const Spacer(),
                // PIN Pad
                _PinPad(
                  onNumberTap: _onNumberTap,
                  onBackspace: _onBackspace,
                ),
                const SizedBox(height: 40),
              ],
            ),
          ),
        ),
      ),
    );
  }
}

/// PIN Keyboard (เหมือนกับ PinSetupPage)
class _PinPad extends StatelessWidget {
  final void Function(String) onNumberTap;
  final VoidCallback onBackspace;

  const _PinPad({
    required this.onNumberTap,
    required this.onBackspace,
  });

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 40),
      child: Column(
        children: [
          _buildRow(['1', '2', '3']),
          const SizedBox(height: 20),
          _buildRow(['4', '5', '6']),
          const SizedBox(height: 20),
          _buildRow(['7', '8', '9']),
          const SizedBox(height: 20),
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            children: [
              const SizedBox(width: 70, height: 70), // Spacer
              _buildButton('0'),
              SizedBox(
                width: 70,
                height: 70,
                child: IconButton(
                  icon: const Icon(Icons.backspace_outlined),
                  onPressed: onBackspace,
                ),
              ),
            ],
          ),
        ],
      ),
    );
  }

  Widget _buildRow(List<String> numbers) {
    return Row(
      mainAxisAlignment: MainAxisAlignment.spaceEvenly,
      children: numbers.map(_buildButton).toList(),
    );
  }

  Widget _buildButton(String number) {
    return SizedBox(
      width: 70,
      height: 70,
      child: TextButton(
        onPressed: () => onNumberTap(number),
        style: TextButton.styleFrom(
          backgroundColor: AppColors.card,
          shape: const CircleBorder(),
        ),
        child: Text(
          number,
          style: AppType.h2.copyWith(fontSize: 24),
        ),
      ),
    );
  }
}
```

> 🔗 **การเชื่อมโยงของระบบ PIN:** ตั้ง PIN จากหน้าโปรไฟล์ (`/pin-setup`) → เก็บใน Secure Storage → หน้า Login โหมด Quick Sign-in มีปุ่ม "ใช้ PIN" → `/pin-login` ตรวจด้วย `verifyPin()` → ถูกต้องแล้ว **ไม่ได้เข้าแอปตรง ๆ** แต่ดึง credentials ที่เก็บไว้มา `login()` ผ่าน API ตามปกติ (PIN เป็นเพียง "กุญแจปลดล็อกตู้เซฟ" — token จริงยังมาจากเซิร์ฟเวอร์เสมอ) · เมื่อ login สำเร็จ router เด้งจาก `/pin-login` เข้า `/home` เองด้วย guard เดิม · จุดฝึกคิดต่อ: PIN ถูกเก็บ plain — ควร hash (เช่น SHA-256 + salt) ก่อนเก็บ

#### 10.5 `register_page.dart` ✏️ — สมัครสมาชิกผ่าน API จริง

```dart
// lib/features/auth/presentation/pages/register_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/l10n/l10n_ext.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_text_field.dart';
import '../../../../core/widgets/gradient_header.dart';
import '../../data/auth_repository.dart';

/// หน้าสมัครสมาชิก — Day 3: ต่อ API จริง (POST /auth/register)
/// สมัครสำเร็จแล้วกลับไปหน้า Login เพื่อเข้าสู่ระบบด้วยบัญชีใหม่
class RegisterPage extends ConsumerStatefulWidget {
  const RegisterPage({super.key});

  @override
  ConsumerState<RegisterPage> createState() => _RegisterPageState();
}

class _RegisterPageState extends ConsumerState<RegisterPage> {
  final _name = TextEditingController();
  final _email = TextEditingController();
  final _phone = TextEditingController();
  final _password = TextEditingController();
  final _confirm = TextEditingController();

  bool _accept = false;
  bool _submitting = false;

  @override
  void dispose() {
    _name.dispose();
    _email.dispose();
    _phone.dispose();
    _password.dispose();
    _confirm.dispose();
    super.dispose();
  }

  void _toast(String msg) => ScaffoldMessenger.of(context)
      .showSnackBar(SnackBar(content: Text(msg)));

  Future<void> _submit() async {
    // ตรวจข้อมูลเบื้องต้นฝั่งแอปก่อนยิง API
    final l = context.l10n;
    if (_name.text.trim().isEmpty ||
        _email.text.trim().isEmpty ||
        _password.text.isEmpty) {
      _toast(l.registerValidationRequired);
      return;
    }
    if (_password.text.length < 8) {
      _toast(l.passwordMin8Required);
      return;
    }
    if (_password.text != _confirm.text) {
      _toast(l.passwordConfirmMismatch);
      return;
    }

    setState(() => _submitting = true);
    try {
      await ref.read(authRepositoryProvider).register(
            name: _name.text.trim(),
            email: _email.text.trim(),
            phone: _phone.text.trim(),
            password: _password.text,
          );
      if (!mounted) return;
      Navigator.of(context).pop(); // กลับไปหน้า Login
      _toast(l.registerSuccess);
    } catch (e) {
      if (!mounted) return;
      setState(() => _submitting = false);
      _toast('$e');
    }
  }

  @override
  Widget build(BuildContext context) {
    final l = context.l10n;
    final bottomInset = MediaQuery.paddingOf(context).bottom;
    return Scaffold(
      body: ListView(
        padding: EdgeInsets.zero,
        children: [
          GradientHeader(
            showBack: true,
            title: l.signUp,
            subtitle: l.registerSubtitle,
          ),
          Padding(
            padding: EdgeInsets.fromLTRB(20, 20, 20, 28 + bottomInset),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                AppTextField(
                    label: l.fullName,
                    hint: l.fullNameHint,
                    icon: HugeIcons.strokeRoundedUser,
                    controller: _name),
                const SizedBox(height: 14),
                AppTextField(
                    label: l.emailLabel,
                    hint: l.emailHint,
                    icon: HugeIcons.strokeRoundedMail01,
                    keyboardType: TextInputType.emailAddress,
                    controller: _email),
                const SizedBox(height: 14),
                AppTextField(
                    label: l.phone,
                    hint: l.phoneHint,
                    icon: HugeIcons.strokeRoundedCall,
                    keyboardType: TextInputType.phone,
                    controller: _phone),
                const SizedBox(height: 14),
                AppTextField(
                    label: l.passwordLabel,
                    hint: l.passwordHintMin8,
                    icon: HugeIcons.strokeRoundedSquareLock01,
                    obscure: true,
                    controller: _password),
                const SizedBox(height: 14),
                AppTextField(
                    label: l.confirmPasswordLabel,
                    hint: l.confirmPasswordHint,
                    icon: HugeIcons.strokeRoundedSquareLock01,
                    obscure: true,
                    controller: _confirm),
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
                          children: [
                            TextSpan(text: l.agreeToTerms),
                            TextSpan(
                              text: l.termsAndPrivacy,
                              style: const TextStyle(
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
                  label: _submitting ? l.creatingAccount : l.createAccount,
                  icon: HugeIcons.strokeRoundedUserAdd01,
                  onPressed: (_accept && !_submitting) ? _submit : null,
                ),
                const SizedBox(height: 16),
                AppButton.social(
                    label: l.continueWithGoogle,
                    icon: HugeIcons.strokeRoundedGoogle,
                    color: const Color(0xFF4285F4),
                    onPressed: () {}),
                const SizedBox(height: 12),
                AppButton.social(
                    label: l.continueWithLine,
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

#### 10.6 `forgot_password_page.dart` ✏️ — ขอลิงก์รีเซ็ตผ่าน API จริง

```dart
// lib/features/auth/presentation/pages/forgot_password_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/l10n/l10n_ext.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_text_field.dart';
import '../../../../core/widgets/gradient_header.dart';
import '../../data/auth_repository.dart';
import 'reset_password_page.dart';

/// หน้าลืมรหัสผ่าน — Day 3: ต่อ API จริง (POST /auth/forgot-password)
/// เมื่อสำเร็จ จำลองว่าผู้ใช้กดลิงก์ในอีเมล → ไปหน้าตั้งรหัสผ่านใหม่ (ส่ง email ต่อไป)
class ForgotPasswordPage extends ConsumerStatefulWidget {
  const ForgotPasswordPage({super.key});

  @override
  ConsumerState<ForgotPasswordPage> createState() => _ForgotPasswordPageState();
}

class _ForgotPasswordPageState extends ConsumerState<ForgotPasswordPage> {
  final _email = TextEditingController();
  bool _submitting = false;

  @override
  void dispose() {
    _email.dispose();
    super.dispose();
  }

  void _toast(String msg) => ScaffoldMessenger.of(context)
      .showSnackBar(SnackBar(content: Text(msg)));

  Future<void> _submit() async {
    final email = _email.text.trim();
    if (email.isEmpty) {
      _toast(context.l10n.pleaseEnterEmail);
      return;
    }
    setState(() => _submitting = true);
    try {
      final message = await ref.read(authRepositoryProvider).forgotPassword(email);
      if (!mounted) return;
      setState(() => _submitting = false);
      _toast(message);
      // จำลองว่าผู้ใช้กดลิงก์รีเซ็ตในอีเมล → ไปหน้าตั้งรหัสผ่านใหม่
      Navigator.of(context).push(
        MaterialPageRoute(builder: (_) => ResetPasswordPage(email: email)),
      );
    } catch (e) {
      if (!mounted) return;
      setState(() => _submitting = false);
      _toast('$e');
    }
  }

  @override
  Widget build(BuildContext context) {
    final l = context.l10n;
    return Scaffold(
      body: ListView(
        padding: EdgeInsets.zero,
        children: [
          GradientHeader(
            showBack: true,
            title: l.forgotPasswordTitle,
            subtitle: l.forgotPasswordSubtitle,
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
                    child: const HugeIcon(
                        icon: HugeIcons.strokeRoundedLockKey,
                        size: 34,
                        color: AppColors.primary),
                  ),
                ),
                const SizedBox(height: 28),
                AppTextField(
                    label: l.emailLabel,
                    hint: l.emailHint,
                    icon: HugeIcons.strokeRoundedMail01,
                    keyboardType: TextInputType.emailAddress,
                    controller: _email),
                const SizedBox(height: 18),
                AppButton.primary(
                  label: _submitting ? l.sending : l.sendResetLink,
                  icon: HugeIcons.strokeRoundedMail01,
                  onPressed: _submitting ? null : _submit,
                ),
                const SizedBox(height: 8),
                Center(
                  child: TextButton.icon(
                    onPressed: () => Navigator.of(context).pop(),
                    icon: const HugeIcon(
                        icon: HugeIcons.strokeRoundedArrowLeft01, size: 18),
                    label: Text(l.backToSignIn),
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

#### 10.7 `reset_password_page.dart` ✏️ — รับ email ผ่าน constructor + API จริง

```dart
// lib/features/auth/presentation/pages/reset_password_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/l10n/l10n_ext.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_text_field.dart';
import '../../../../core/widgets/gradient_header.dart';
import '../../../../core/widgets/info_hint.dart';
import '../../data/auth_repository.dart';

/// หน้าตั้งรหัสผ่านใหม่ — Day 3: ต่อ API จริง (POST /auth/reset-password)
/// รับ [email] มาจากหน้าลืมรหัสผ่าน (จำลองว่ามาจากลิงก์ในอีเมล)
class ResetPasswordPage extends ConsumerStatefulWidget {
  final String email;
  const ResetPasswordPage({super.key, required this.email});

  @override
  ConsumerState<ResetPasswordPage> createState() => _ResetPasswordPageState();
}

class _ResetPasswordPageState extends ConsumerState<ResetPasswordPage> {
  final _password = TextEditingController();
  final _confirm = TextEditingController();
  bool _submitting = false;

  @override
  void dispose() {
    _password.dispose();
    _confirm.dispose();
    super.dispose();
  }

  void _toast(String msg) => ScaffoldMessenger.of(context)
      .showSnackBar(SnackBar(content: Text(msg)));

  Future<void> _submit() async {
    final l = context.l10n;
    if (_password.text.length < 8) {
      _toast(l.passwordMin8Required);
      return;
    }
    if (_password.text != _confirm.text) {
      _toast(l.passwordConfirmMismatch);
      return;
    }
    setState(() => _submitting = true);
    try {
      final message = await ref.read(authRepositoryProvider).resetPassword(
            email: widget.email,
            password: _password.text,
          );
      if (!mounted) return;
      _toast(message);
      // กลับไปหน้า Login เพื่อเข้าสู่ระบบด้วยรหัสผ่านใหม่
      Navigator.of(context).popUntil((route) => route.isFirst);
    } catch (e) {
      if (!mounted) return;
      setState(() => _submitting = false);
      _toast('$e');
    }
  }

  @override
  Widget build(BuildContext context) {
    final l = context.l10n;
    return Scaffold(
      body: ListView(
        padding: EdgeInsets.zero,
        children: [
          GradientHeader(
            showBack: true,
            title: l.resetPasswordTitle,
            subtitle: l.resetPasswordSubtitle,
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
                    child: const HugeIcon(
                        icon: HugeIcons.strokeRoundedSecurityCheck,
                        size: 34,
                        color: AppColors.primary),
                  ),
                ),
                const SizedBox(height: 12),
                Center(
                  child: Text(l.forAccountEmail(widget.email),
                      style: AppType.caption
                          .copyWith(color: AppColors.textSubtle)),
                ),
                const SizedBox(height: 16),
                AppTextField(
                    label: l.newPasswordLabel,
                    hint: l.passwordHintMin8,
                    icon: HugeIcons.strokeRoundedSquareLock01,
                    obscure: true,
                    controller: _password),
                const SizedBox(height: 14),
                AppTextField(
                    label: l.confirmNewPasswordLabel,
                    hint: l.confirmPasswordHint,
                    icon: HugeIcons.strokeRoundedSquareLock01,
                    obscure: true,
                    controller: _confirm),
                const SizedBox(height: 12),
                InfoHint(l.passwordHintHelper),
                const SizedBox(height: 18),
                AppButton.primary(
                  label: _submitting ? l.saving : l.resetPasswordTitle,
                  icon: HugeIcons.strokeRoundedSecurityCheck,
                  onPressed: _submitting ? null : _submit,
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

> 🔗 **การเชื่อมโยงกลุ่มนี้:** ทั้ง 3 หน้าเปลี่ยนจาก "UI เปล่า" (Day 1) เป็น flow จริง: register → `POST /auth/register` แล้ว pop กลับ Login · forgot → `POST /auth/forgot-password` แล้ว **จำลองการกดลิงก์ในอีเมล** ด้วยการ push ไป `ResetPasswordPage(email: email)` (ส่งข้อมูลผ่าน constructor) · reset → `POST /auth/reset-password` แล้ว `popUntil(isFirst)` กลับ Login · ทุกหน้าใช้แพตเทิร์นเดียวกัน: validate ฝั่งแอปก่อน → `_submitting` กันกดซ้ำ → error จาก server แสดงผ่าน `_toast('$e')` ซึ่งได้ข้อความจริงจาก `ApiException`

### ขั้นที่ 11 — Settings + Dark Mode + i18n (Stage 3) 🆕

#### 11.1 `settings_providers.dart` — ภาษา + โหมดธีม (persisted ด้วย SharedPreferences)

```dart
// lib/core/settings/settings_providers.dart
import 'package:flutter/material.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'package:shared_preferences/shared_preferences.dart';

part 'settings_providers.g.dart';

/// อินสแตนซ์ SharedPreferences ที่โหลดครั้งเดียวตอนเปิดแอป
///
/// รูปแบบมาตรฐาน: โหลดใน main() แบบ async แล้ว override provider นี้ด้วย
/// ค่าจริง เพื่อให้ controller อื่นอ่านค่าแบบ synchronous ได้ (ไม่ต้อง await)
/// ดู main.dart → ProviderScope(overrides: [sharedPreferencesProvider.overrideWithValue(prefs)])
@Riverpod(keepAlive: true)
SharedPreferences sharedPreferences(SharedPreferencesRef ref) =>
    throw UnimplementedError('ต้อง override sharedPreferencesProvider ใน main()');

/// ตัวควบคุมภาษาของแอป (ไทย/อังกฤษ) — จำค่าไว้ด้วย SharedPreferences
@Riverpod(keepAlive: true)
class LocaleController extends _$LocaleController {
  static const _key = 'app_locale';

  @override
  Locale build() {
    final code = ref.watch(sharedPreferencesProvider).getString(_key) ?? 'th';
    return Locale(code);
  }

  bool get isThai => state.languageCode == 'th';

  Future<void> setLocale(String code) async {
    await ref.read(sharedPreferencesProvider).setString(_key, code);
    state = Locale(code);
  }

  Future<void> setThai(bool thai) => setLocale(thai ? 'th' : 'en');
}

/// ตัวควบคุมโหมดธีม (สว่าง/มืด) — จำค่าไว้ด้วย SharedPreferences
@Riverpod(keepAlive: true)
class ThemeModeController extends _$ThemeModeController {
  static const _key = 'app_theme_mode';

  @override
  ThemeMode build() {
    final v = ref.watch(sharedPreferencesProvider).getString(_key) ?? 'light';
    return v == 'dark' ? ThemeMode.dark : ThemeMode.light;
  }

  bool get isDark => state == ThemeMode.dark;

  Future<void> setDark(bool dark) async {
    await ref
        .read(sharedPreferencesProvider)
        .setString(_key, dark ? 'dark' : 'light');
    state = dark ? ThemeMode.dark : ThemeMode.light;
  }
}
```

> 🔗 **การเชื่อมโยง:** provider ตัวแรกจงใจ `throw UnimplementedError` — ต้องถูก override ใน `main()` เสมอ (แพตเทิร์นมาตรฐานของ Riverpod สำหรับ dependency ที่ต้อง init แบบ async) · toggle ภาษา/โหมดมืดในหน้าโปรไฟล์ (Day 1 เป็น `setState` เดโม) ตอนนี้เป็นของจริง: เขียน prefs + เปลี่ยน state → `MaterialApp` rebuild → **จำค่าไว้แม้ปิดแอป**

#### 11.2 `app_palette.dart` — ThemeExtension สำหรับ Dark Mode 🆕

```dart
// lib/core/theme/app_palette.dart
import 'package:flutter/material.dart';

import 'app_colors.dart';

/// โทเคนสีเชิงความหมาย (semantic) ที่เปลี่ยนตามธีม (สว่าง/มืด)
///
/// ใช้ ThemeExtension ซึ่งเป็นวิธีมาตรฐานของ Flutter ในการเพิ่มชุดสี custom
/// ให้กับ ThemeData แล้วให้มันสลับค่าตาม ThemeMode อัตโนมัติ
/// เรียกใช้ผ่าน `context.palette.card` เป็นต้น
///
/// หมายเหตุ (การ migrate เป็น Dark Mode): หน้าจอเดิมส่วนใหญ่ยังอ้าง const AppColors
/// โดยตรง จึงยังคงเป็นโทนสว่าง — ทยอยเปลี่ยนมาใช้ context.palette ทีละหน้าได้
@immutable
class AppPalette extends ThemeExtension<AppPalette> {
  final Color background;
  final Color card;
  final Color border;
  final Color textDark;
  final Color textMedium;
  final Color textSubtle;
  final Color fieldFill;

  const AppPalette({
    required this.background,
    required this.card,
    required this.border,
    required this.textDark,
    required this.textMedium,
    required this.textSubtle,
    required this.fieldFill,
  });

  /// โทนสว่าง — ตรงกับค่าเดิมใน AppColors (คงหน้าตาเดิมทุกประการ)
  static const AppPalette light = AppPalette(
    background: AppColors.background,
    card: AppColors.card,
    border: AppColors.border,
    textDark: AppColors.textDark,
    textMedium: AppColors.textMedium,
    textSubtle: AppColors.textSubtle,
    fieldFill: AppColors.card,
  );

  /// โทนมืด — ปรับให้คู่สีอ่านง่าย ยังคงกลิ่นแบรนด์น้ำเงิน
  static const AppPalette dark = AppPalette(
    background: Color(0xFF0E1726),
    card: Color(0xFF16213A),
    border: Color(0xFF27364F),
    textDark: Color(0xFFEAF0FA),
    textMedium: Color(0xFFAEBBD3),
    textSubtle: Color(0xFF7C8CA8),
    fieldFill: Color(0xFF1B2942),
  );

  @override
  AppPalette copyWith({
    Color? background,
    Color? card,
    Color? border,
    Color? textDark,
    Color? textMedium,
    Color? textSubtle,
    Color? fieldFill,
  }) {
    return AppPalette(
      background: background ?? this.background,
      card: card ?? this.card,
      border: border ?? this.border,
      textDark: textDark ?? this.textDark,
      textMedium: textMedium ?? this.textMedium,
      textSubtle: textSubtle ?? this.textSubtle,
      fieldFill: fieldFill ?? this.fieldFill,
    );
  }

  @override
  AppPalette lerp(ThemeExtension<AppPalette>? other, double t) {
    if (other is! AppPalette) return this;
    return AppPalette(
      background: Color.lerp(background, other.background, t)!,
      card: Color.lerp(card, other.card, t)!,
      border: Color.lerp(border, other.border, t)!,
      textDark: Color.lerp(textDark, other.textDark, t)!,
      textMedium: Color.lerp(textMedium, other.textMedium, t)!,
      textSubtle: Color.lerp(textSubtle, other.textSubtle, t)!,
      fieldFill: Color.lerp(fieldFill, other.fieldFill, t)!,
    );
  }
}

/// ช่วยเรียกใช้สั้น ๆ: `context.palette.card`
extension AppPaletteX on BuildContext {
  AppPalette get palette =>
      Theme.of(this).extension<AppPalette>() ?? AppPalette.light;
}
```

#### 11.3 `app_theme.dart` ✏️ — จาก light อย่างเดียว → light/dark จาก `_build` เดียว

```dart
// lib/core/theme/app_theme.dart
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';

import 'app_colors.dart';
import 'app_palette.dart';
import 'app_spacing.dart';

/// ThemeData กลางของแอป ประกอบจาก token ทั้งหมด
/// รองรับทั้งโทนสว่าง (light) และโทนมืด (dark) โดยใช้ AppPalette (ThemeExtension)
abstract final class AppTheme {
  static ThemeData get light => _build(Brightness.light, AppPalette.light);
  static ThemeData get dark => _build(Brightness.dark, AppPalette.dark);

  static ThemeData _build(Brightness brightness, AppPalette palette) {
    final base = ThemeData(
      useMaterial3: true,
      brightness: brightness,
      colorScheme: ColorScheme.fromSeed(
        seedColor: AppColors.primary,
        brightness: brightness,
        primary: AppColors.primary,
        surface: palette.card,
      ),
      scaffoldBackgroundColor: palette.background,
    );

    return base.copyWith(
      extensions: [palette],
      textTheme: GoogleFonts.interTextTheme(base.textTheme).apply(
        bodyColor: palette.textDark,
        displayColor: palette.textDark,
        fontFamilyFallback: [GoogleFonts.anuphan().fontFamily!],
      ),
      appBarTheme: const AppBarTheme(
        backgroundColor: Colors.transparent,
        elevation: 0,
        scrolledUnderElevation: 0,
        foregroundColor: Colors.white,
      ),
      cardTheme: CardThemeData(
        color: palette.card,
        elevation: 0,
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(AppSpacing.rMd),
        ),
      ),
      inputDecorationTheme: InputDecorationTheme(
        filled: true,
        fillColor: palette.fieldFill,
        contentPadding: const EdgeInsets.symmetric(horizontal: 14, vertical: 14),
        border: OutlineInputBorder(
          borderRadius: BorderRadius.circular(AppSpacing.rSm),
          borderSide: BorderSide(color: palette.border),
        ),
        enabledBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(AppSpacing.rSm),
          borderSide: BorderSide(color: palette.border),
        ),
        focusedBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(AppSpacing.rSm),
          borderSide: const BorderSide(color: AppColors.primary, width: 1.6),
        ),
        hintStyle: TextStyle(color: palette.textSubtle),
      ),
      dividerTheme: DividerThemeData(color: palette.border, thickness: 1),
    );
  }
}
```

#### 11.4 วิดเจ็ตกลางที่ปรับเป็น palette-aware ✏️ (`app_text_field.dart`, `section_label.dart`)

```dart
// lib/core/widgets/section_label.dart
import 'package:flutter/material.dart';

import '../theme/app_palette.dart';
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
          Text(text, style: AppType.h3.copyWith(color: context.palette.textDark)),
          if (trailing != null) trailing!,
        ],
      ),
    );
  }
}
```

```dart
// lib/core/widgets/app_text_field.dart
import 'package:flutter/material.dart';

import '../icons/app_icons.dart';
import '../theme/app_palette.dart';
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
    // ใช้ palette (ThemeExtension) เพื่อให้สีตัวอักษร/ไอคอนถูกต้องทั้งโหมดสว่างและมืด
    // ในโหมดสว่างค่าเท่ากับ AppColors เดิมทุกประการ
    final p = context.palette;
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(widget.label, style: AppType.label.copyWith(color: p.textMedium)),
        const SizedBox(height: 6),
        TextField(
          controller: widget.controller,
          obscureText: _hidden,
          keyboardType: widget.keyboardType,
          maxLines: widget.obscure ? 1 : widget.maxLines,
          style: AppType.bodyStrong.copyWith(color: p.textDark),
          decoration: InputDecoration(
            hintText: widget.hint,
            prefixIcon: widget.icon == null
                ? null
                : HugeIcon(icon: widget.icon!, size: 20, color: p.textSubtle),
            suffixIcon: widget.obscure
                ? IconButton(
                    icon: HugeIcon(
                      icon: _hidden
                          ? HugeIcons.strokeRoundedView
                          : HugeIcons.strokeRoundedViewOffSlash,
                      size: 20,
                      color: p.textSubtle,
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

#### 11.5 i18n: `l10n.yaml` + `l10n_ext.dart` + ไฟล์ .arb 🆕

```yaml
# l10n.yaml
# การตั้งค่า flutter gen-l10n (สร้าง AppLocalizations จากไฟล์ .arb)
# รันอัตโนมัติเมื่อ `flutter pub get` / `flutter run` เพราะตั้ง generate: true ใน pubspec
# หรือสั่งเองด้วย: flutter gen-l10n
arb-dir: lib/l10n
template-arb-file: app_en.arb
output-localization-file: app_localizations.dart
output-class: AppLocalizations
# ไม่ใช้ synthetic package (deprecated) — ให้ไฟล์ที่ถูกสร้างไปอยู่ใน lib/l10n
# แล้ว import ด้วย package:bla_policy_companion/l10n/app_localizations.dart
synthetic-package: false
nullable-getter: false
```

```dart
// lib/core/l10n/l10n_ext.dart
import 'package:flutter/widgets.dart';

import '../../l10n/app_localizations.dart';

/// ช่วยเรียกข้อความแปลสั้น ๆ: `context.l10n.save`
/// (AppLocalizations ถูกสร้างโดย flutter gen-l10n จากไฟล์ lib/l10n/*.arb)
extension L10nX on BuildContext {
  AppLocalizations get l10n => AppLocalizations.of(this);
}
```

> 🔗 **กลไก i18n ทั้งระบบ:** ข้อความทุกคำอยู่ใน `lib/l10n/app_th.arb` (ไทย) และ `app_en.arb` (อังกฤษ — เป็น template) → `flutter gen-l10n` สร้าง `app_localizations.dart` + `_th.dart` + `_en.dart` (รวม ~2,270 บรรทัด — generated ห้ามแก้มือ) → ทุกหน้าเรียกผ่าน `context.l10n.<key>` → เมื่อ `localeControllerProvider` เปลี่ยน `MaterialApp.locale` ทั้งแอปสลับภาษาทันที · ไฟล์ .arb ทั้งสอง (ฉบับเต็ม ~220 บรรทัดต่อไฟล์) อยู่ในหัวข้อถัดไป

### ขั้นที่ 12 — ไฟล์แปลภาษา .arb (source of truth ของ i18n) 🆕

ตาม `l10n.yaml` ไฟล์ `app_en.arb` คือ template (ตัวกำหนดว่ามี key อะไรบ้าง) ส่วน `app_th.arb` คือคำแปลภาษาไทย — ทุก key จะถูก `flutter gen-l10n` แปลงเป็น getter บนคลาส `AppLocalizations` ให้เรียกใช้ผ่าน `context.l10n.<key>` ได้ทันที

#### 12.1 `lib/l10n/app_th.arb`

```json
{
  "@@locale": "th",

  "verifiedBadge": "ยืนยันตัวตนแล้ว",
  "accountSection": "บัญชีของฉัน",
  "settingsSection": "การตั้งค่า",
  "helpSection": "ช่วยเหลือ",

  "menuPersonalInfo": "ข้อมูลส่วนตัว",
  "menuMyPolicies": "กรมธรรม์ของฉัน",
  "menuPaymentHistory": "ประวัติการชำระ",

  "notifications": "การแจ้งเตือน",
  "changePassword": "เปลี่ยนรหัสผ่าน",
  "setupPin": "ตั้งค่า PIN",
  "language": "ภาษา",
  "darkMode": "โหมดมืด",

  "helpContactAgent": "ติดต่อตัวแทน",
  "helpFaq": "คำถามที่พบบ่อย",

  "logout": "ออกจากระบบ",
  "logoutConfirmTitle": "ออกจากระบบ?",
  "logoutConfirmBody": "คุณจะต้องเข้าสู่ระบบอีกครั้งในครั้งถัดไป",
  "cancel": "ยกเลิก",
  "comingSoon": "ฟีเจอร์นี้กำลังพัฒนา",

  "editProfileTitle": "แก้ไขข้อมูลส่วนตัว",
  "editProfileSubtitle": "ปรับปรุงข้อมูลส่วนตัวของคุณ",
  "fullName": "ชื่อ-นามสกุล",
  "phone": "เบอร์โทรศัพท์",
  "save": "บันทึก",
  "saving": "กำลังบันทึก...",
  "profileUpdated": "บันทึกข้อมูลสำเร็จ",
  "nameRequired": "กรุณากรอกชื่อ",

  "changePasswordSubtitle": "ตั้งรหัสผ่านใหม่สำหรับบัญชีของคุณ",
  "currentPassword": "รหัสผ่านปัจจุบัน",
  "newPassword": "รหัสผ่านใหม่",
  "confirmPassword": "ยืนยันรหัสผ่านใหม่",
  "passwordMismatch": "รหัสผ่านใหม่ไม่ตรงกัน",
  "passwordTooShort": "รหัสผ่านต้องมีอย่างน้อย 4 ตัวอักษร",
  "passwordChanged": "เปลี่ยนรหัสผ่านสำเร็จ",

  "languageThai": "ไทย",
  "languageEnglish": "EN",

  "loginSubtitle": "เข้าสู่ระบบเพื่อจัดการกรมธรรม์ของคุณ",
  "quickSignInTitle": "เข้าสู่ระบบอย่างรวดเร็ว",
  "scanFingerprint": "สแกนลายนิ้วมือ",
  "usePin": "ใช้ PIN",
  "signInWithPassword": "เข้าสู่ระบบด้วยรหัสผ่าน",
  "usernameLabel": "ชื่อผู้ใช้",
  "passwordLabel": "รหัสผ่าน",
  "testAccountInfo": "บัญชีทดสอบ: ชื่อผู้ใช้ customer / รหัสผ่าน 1234",
  "demoUsernameOnly": "เดโมนี้รองรับเฉพาะการเข้าสู่ระบบด้วยชื่อผู้ใช้",
  "biometricAuthFailed": "ยืนยันตัวตนไม่สำเร็จ",
  "signingIn": "กำลังเข้าสู่ระบบ...",
  "signIn": "เข้าสู่ระบบ",
  "forgotPasswordLink": "ลืมรหัสผ่าน?",
  "orContinueWith": "หรือดำเนินการต่อด้วย",
  "continueWithGoogle": "ดำเนินการต่อด้วย Google",
  "continueWithLine": "ดำเนินการต่อด้วย LINE",
  "noAccountYet": "ยังไม่มีบัญชี?",
  "signUp": "สมัครสมาชิก",
  "appFooter": "กรุงเทพประกันชีวิต จำกัด (มหาชน) · v1.0.0",

  "registerSubtitle": "กรอกข้อมูลเพื่อเปิดบัญชีใหม่",
  "fullNameHint": "เช่น สมชาย ใจดี",
  "emailLabel": "อีเมล",
  "emailHint": "you@example.com",
  "phoneHint": "08X-XXX-XXXX",
  "passwordHintMin8": "อย่างน้อย 8 ตัวอักษร",
  "confirmPasswordLabel": "ยืนยันรหัสผ่าน",
  "confirmPasswordHint": "พิมพ์รหัสผ่านอีกครั้ง",
  "agreeToTerms": "ฉันยอมรับ ",
  "termsAndPrivacy": "ข้อกำหนดและนโยบายความเป็นส่วนตัว",
  "creatingAccount": "กำลังสมัคร...",
  "createAccount": "สร้างบัญชี",
  "registerValidationRequired": "กรุณากรอกชื่อ อีเมล และรหัสผ่านให้ครบ",
  "passwordMin8Required": "รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร",
  "passwordConfirmMismatch": "รหัสผ่านและการยืนยันไม่ตรงกัน",
  "registerSuccess": "สมัครสมาชิกสำเร็จ เข้าสู่ระบบได้เลย",

  "forgotPasswordTitle": "ลืมรหัสผ่าน",
  "forgotPasswordSubtitle": "กรอกอีเมลที่ลงทะเบียนไว้ เราจะส่งลิงก์รีเซ็ตรหัสผ่านให้คุณ",
  "pleaseEnterEmail": "กรุณากรอกอีเมล",
  "sending": "กำลังส่ง...",
  "sendResetLink": "ส่งลิงก์รีเซ็ต",
  "backToSignIn": "กลับไปหน้าเข้าสู่ระบบ",

  "resetPasswordTitle": "ตั้งรหัสผ่านใหม่",
  "resetPasswordSubtitle": "สร้างรหัสผ่านใหม่สำหรับบัญชีของคุณ",
  "forAccountEmail": "สำหรับบัญชี {email}",
  "@forAccountEmail": {
    "placeholders": { "email": { "type": "String" } }
  },
  "newPasswordLabel": "รหัสผ่านใหม่",
  "confirmNewPasswordLabel": "ยืนยันรหัสผ่านใหม่",
  "passwordHintHelper": "ใช้อย่างน้อย 8 ตัวอักษร ประกอบด้วยตัวอักษรและตัวเลข",

  "enterPinTitle": "ใส่รหัส PIN",
  "enterPinSubtitle": "ใส่รหัส PIN 6 หลักเพื่อเข้าสู่ระบบ",
  "confirmPinTitle": "ยืนยัน PIN",
  "confirmPinSubtitle": "กรอก PIN อีกครั้งเพื่อยืนยัน",
  "testPinInfo": "PIN ทดสอบ: 123456",
  "incorrectPin": "PIN ไม่ถูกต้อง",
  "pinMismatch": "PIN ไม่ตรงกัน กรุณาลองใหม่",
  "pinSetupSuccess": "ตั้งค่า PIN สำเร็จ",

  "helloGreeting": "สวัสดี 👋",
  "totalCoverageLabel": "วงเงินคุ้มครองรวม",
  "activePoliciesLabel": "กรมธรรม์มีผลบังคับ",
  "activePoliciesMockValue": "2 ฉบับ",
  "annualPremiumLabel": "เบี้ยรวม/ปี",
  "premiumDueMock": "เบี้ยครบกำหนด ฿50,000",
  "policyDueDateMock": "BLA สะสมทรัพย์ 10/5 · 15 ก.ค. 2569",
  "payShort": "ชำระ",
  "recentNotificationsLabel": "การแจ้งเตือนล่าสุด",
  "seeAllLabel": "ดูทั้งหมด",
  "payPremium": "ชำระเบี้ย",
  "fileClaim": "ยื่นสินไหม",
  "navPolicies": "กรมธรรม์",
  "contactUs": "ติดต่อเรา",

  "notifPremiumDueTitle": "เบี้ยใกล้ครบกำหนด",
  "notifPremiumDueSubtitleMock": "BLA สะสมทรัพย์ 10/5 · ฿50,000",
  "notifDaysLeftMock": "อีก 17 วัน",
  "notifClaimApprovedTitle": "สินไหมอนุมัติแล้ว",
  "notifClaimApprovedSubtitleMock": "CLM-48213 · ค่ารักษาพยาบาล",
  "notifDaysAgoMock": "2 วันที่แล้ว",
  "notifPolicyActiveSubtitleMock": "BLA ตลอดชีพ มั่นคง 99",
  "notifWeekAgoMock": "1 สัปดาห์ที่แล้ว",

  "myPoliciesTitle": "กรมธรรม์ของฉัน",
  "totalPoliciesSubtitle": "ทั้งหมด {total} ฉบับ",
  "@totalPoliciesSubtitle": {
    "placeholders": { "total": { "type": "int" } }
  },
  "searchPolicyHint": "ค้นหาชื่อแผน หรือ เลขที่กรมธรรม์",
  "filterAll": "ทั้งหมด",
  "totalPremiumActiveLabel": "เบี้ยรวม (มีผลบังคับ)",
  "premiumPerYear": "฿{amount}/ปี",
  "@premiumPerYear": {
    "placeholders": { "amount": { "type": "String" } }
  },
  "noPoliciesFound": "ไม่พบกรมธรรม์ที่ตรงเงื่อนไข",
  "retry": "ลองใหม่",

  "policyDetailsTitle": "รายละเอียดกรมธรรม์",
  "coverageAmountLabel": "วงเงินคุ้มครอง",
  "policyInfoSectionTitle": "ข้อมูลกรมธรรม์",
  "insuredLabel": "ผู้เอาประกัน",
  "premiumLabel": "เบี้ยประกัน",
  "startDateLabel": "วันเริ่มสัญญา",
  "endDateLabel": "วันสิ้นสุด",
  "startDateMock": "1 ม.ค. 2565",
  "endDateMock": "1 ม.ค. 2664",

  "paymentsPageTitle": "ชำระเบี้ย",
  "paymentsSubtitle": "จ่ายเบี้ยและดูประวัติ",
  "paymentDueDaysMock": "ครบกำหนดอีก 17 วัน",
  "mockPlanName": "BLA สะสมทรัพย์ 10/5",
  "dueDateInfoMock": "กำหนดชำระ 15 ก.ค. 2569 · BLA-2569-0002",
  "payNowButton": "จ่ายตอนนี้",
  "paymentMethodsSection": "ช่องทางชำระเงิน",
  "promptpayQrTitle": "พร้อมเพย์ / QR",
  "bankTransferSubtitle": "ตัดผ่านบัญชีธนาคาร",
  "defaultBadge": "ค่าเริ่มต้น",
  "creditCardTitleMock": "บัตรเครดิต ••6789",
  "addPaymentMethod": "เพิ่มช่องทางชำระเงิน",
  "paymentHistorySection": "ประวัติการชำระ",
  "loadFailedPayments": "โหลดประวัติไม่สำเร็จ: {error}",
  "@loadFailedPayments": {
    "placeholders": { "error": { "type": "String" } }
  },

  "claimsTitle": "สินไหม",
  "claimsSubtitle": "ยื่นและติดตามสถานะคำขอ",
  "totalClaimsLabel": "คำขอทั้งหมด",
  "claimsCountLabel": "{count} รายการ",
  "@claimsCountLabel": {
    "placeholders": { "count": { "type": "int" } }
  },
  "approvedLabel": "อนุมัติแล้ว",
  "fileNewClaimButton": "ยื่นสินไหมใหม่",
  "claimHistorySection": "ประวัติการยื่น",
  "noClaimHistory": "ยังไม่มีประวัติการยื่น",
  "loadFailedClaims": "โหลดสินไหมไม่สำเร็จ: {error}",
  "@loadFailedClaims": {
    "placeholders": { "error": { "type": "String" } }
  },

  "claimAmountLabel": "จำนวนเงิน (บาท)",
  "claimReasonLabel": "เหตุผล / รายละเอียด",
  "claimReasonHint": "ระบุสาเหตุและรายละเอียดการยื่นสินไหม...",
  "filingInProgress": "กำลังยื่น...",
  "confirmFiling": "ยืนยันการยื่น",
  "amountMustBePositive": "กรุณากรอกจำนวนเงินให้มากกว่า 0",
  "filingFailed": "ยื่นไม่สำเร็จ: {error}",
  "@filingFailed": {
    "placeholders": { "error": { "type": "String" } }
  },
  "claimFiledSuccess": "ยื่นสินไหมสำเร็จ",
  "claimReferenceLabel": "เลขอ้างอิงการยื่น",
  "close": "ปิด",

  "navHome": "หน้าแรก",
  "navProfile": "โปรไฟล์",

  "statusActive": "มีผลบังคับ",
  "statusLapsed": "ขาดอายุ",
  "statusPending": "รอดำเนินการ",

  "claimStatusSubmitted": "ยื่นแล้ว",
  "claimStatusReviewing": "กำลังพิจารณา",
  "claimStatusApproved": "อนุมัติ",
  "claimStatusRejected": "ไม่อนุมัติ",
  "claimRequestLabel": "คำขอสินไหม"
}
```

#### 12.2 `lib/l10n/app_en.arb`

```json
{
  "@@locale": "en",

  "verifiedBadge": "Verified",
  "accountSection": "My Account",
  "settingsSection": "Settings",
  "helpSection": "Help",

  "menuPersonalInfo": "Personal Info",
  "menuMyPolicies": "My Policies",
  "menuPaymentHistory": "Payment History",

  "notifications": "Notifications",
  "changePassword": "Change Password",
  "setupPin": "Set up PIN",
  "language": "Language",
  "darkMode": "Dark Mode",

  "helpContactAgent": "Contact Agent",
  "helpFaq": "FAQ",

  "logout": "Log Out",
  "logoutConfirmTitle": "Log out?",
  "logoutConfirmBody": "You will need to sign in again next time.",
  "cancel": "Cancel",
  "comingSoon": "This feature is coming soon",

  "editProfileTitle": "Edit Profile",
  "editProfileSubtitle": "Update your personal details",
  "fullName": "Full name",
  "phone": "Phone number",
  "save": "Save",
  "saving": "Saving...",
  "profileUpdated": "Profile updated successfully",
  "nameRequired": "Please enter your name",

  "changePasswordSubtitle": "Set a new password for your account",
  "currentPassword": "Current password",
  "newPassword": "New password",
  "confirmPassword": "Confirm new password",
  "passwordMismatch": "New passwords do not match",
  "passwordTooShort": "Password must be at least 4 characters",
  "passwordChanged": "Password changed successfully",

  "languageThai": "ไทย",
  "languageEnglish": "EN",

  "loginSubtitle": "Sign in to manage your policies",
  "quickSignInTitle": "Quick sign in",
  "scanFingerprint": "Scan fingerprint",
  "usePin": "Use PIN",
  "signInWithPassword": "Sign in with password",
  "usernameLabel": "Username",
  "passwordLabel": "Password",
  "testAccountInfo": "Test account: username customer / password 1234",
  "demoUsernameOnly": "This demo only supports username sign in",
  "biometricAuthFailed": "Authentication failed",
  "signingIn": "Signing in...",
  "signIn": "Sign in",
  "forgotPasswordLink": "Forgot password?",
  "orContinueWith": "Or continue with",
  "continueWithGoogle": "Continue with Google",
  "continueWithLine": "Continue with LINE",
  "noAccountYet": "Don't have an account?",
  "signUp": "Sign up",
  "appFooter": "Bangkok Life Assurance PCL · v1.0.0",

  "registerSubtitle": "Fill in your details to create a new account",
  "fullNameHint": "e.g. John Smith",
  "emailLabel": "Email",
  "emailHint": "you@example.com",
  "phoneHint": "08X-XXX-XXXX",
  "passwordHintMin8": "At least 8 characters",
  "confirmPasswordLabel": "Confirm password",
  "confirmPasswordHint": "Enter password again",
  "agreeToTerms": "I agree to ",
  "termsAndPrivacy": "Terms & Privacy Policy",
  "creatingAccount": "Creating account...",
  "createAccount": "Create account",
  "registerValidationRequired": "Please fill in name, email and password",
  "passwordMin8Required": "Password must be at least 8 characters",
  "passwordConfirmMismatch": "Passwords do not match",
  "registerSuccess": "Registration successful! You can sign in now.",

  "forgotPasswordTitle": "Forgot Password",
  "forgotPasswordSubtitle": "Enter your registered email and we'll send you a password reset link",
  "pleaseEnterEmail": "Please enter your email",
  "sending": "Sending...",
  "sendResetLink": "Send reset link",
  "backToSignIn": "Back to sign in",

  "resetPasswordTitle": "Reset Password",
  "resetPasswordSubtitle": "Create a new password for your account",
  "forAccountEmail": "For account {email}",
  "@forAccountEmail": {
    "placeholders": { "email": { "type": "String" } }
  },
  "newPasswordLabel": "New password",
  "confirmNewPasswordLabel": "Confirm new password",
  "passwordHintHelper": "Use at least 8 characters, including letters and numbers",

  "enterPinTitle": "Enter PIN",
  "enterPinSubtitle": "Enter your 6-digit PIN to sign in",
  "confirmPinTitle": "Confirm PIN",
  "confirmPinSubtitle": "Enter your PIN again to confirm",
  "testPinInfo": "Test PIN: 123456",
  "incorrectPin": "Incorrect PIN",
  "pinMismatch": "PINs don't match. Please try again.",
  "pinSetupSuccess": "PIN set successfully",

  "helloGreeting": "Hello 👋",
  "totalCoverageLabel": "Total Coverage",
  "activePoliciesLabel": "Active Policies",
  "activePoliciesMockValue": "2 policies",
  "annualPremiumLabel": "Annual Premium",
  "premiumDueMock": "Premium due ฿50,000",
  "policyDueDateMock": "BLA Savings 10/5 · Jul 15, 2569",
  "payShort": "Pay",
  "recentNotificationsLabel": "Recent Notifications",
  "seeAllLabel": "See all",
  "payPremium": "Pay",
  "fileClaim": "File Claim",
  "navPolicies": "Policies",
  "contactUs": "Contact",

  "notifPremiumDueTitle": "Premium due soon",
  "notifPremiumDueSubtitleMock": "BLA Savings 10/5 · ฿50,000",
  "notifDaysLeftMock": "In 17 days",
  "notifClaimApprovedTitle": "Claim approved",
  "notifClaimApprovedSubtitleMock": "CLM-48213 · Medical expenses",
  "notifDaysAgoMock": "2 days ago",
  "notifPolicyActiveSubtitleMock": "BLA Whole Life Mankong 99",
  "notifWeekAgoMock": "1 week ago",

  "myPoliciesTitle": "My Policies",
  "totalPoliciesSubtitle": "Total {total} policies",
  "@totalPoliciesSubtitle": {
    "placeholders": { "total": { "type": "int" } }
  },
  "searchPolicyHint": "Search plan name or policy number",
  "filterAll": "All",
  "totalPremiumActiveLabel": "Total Premium (Active)",
  "premiumPerYear": "฿{amount}/year",
  "@premiumPerYear": {
    "placeholders": { "amount": { "type": "String" } }
  },
  "noPoliciesFound": "No matching policies found",
  "retry": "Retry",

  "policyDetailsTitle": "Policy Details",
  "coverageAmountLabel": "Coverage Amount",
  "policyInfoSectionTitle": "Policy Information",
  "insuredLabel": "Insured",
  "premiumLabel": "Premium",
  "startDateLabel": "Start Date",
  "endDateLabel": "End Date",
  "startDateMock": "Jan 1, 2565",
  "endDateMock": "Jan 1, 2664",

  "paymentsPageTitle": "Payments",
  "paymentsSubtitle": "Pay premiums and view history",
  "paymentDueDaysMock": "Due in 17 days",
  "mockPlanName": "BLA Savings 10/5",
  "dueDateInfoMock": "Due Jul 15, 2569 · BLA-2569-0002",
  "payNowButton": "Pay Now",
  "paymentMethodsSection": "Payment Methods",
  "promptpayQrTitle": "PromptPay / QR",
  "bankTransferSubtitle": "Bank account debit",
  "defaultBadge": "Default",
  "creditCardTitleMock": "Credit card ••6789",
  "addPaymentMethod": "Add payment method",
  "paymentHistorySection": "Payment History",
  "loadFailedPayments": "Failed to load history: {error}",
  "@loadFailedPayments": {
    "placeholders": { "error": { "type": "String" } }
  },

  "claimsTitle": "Claims",
  "claimsSubtitle": "File and track claim status",
  "totalClaimsLabel": "Total Claims",
  "claimsCountLabel": "{count} claims",
  "@claimsCountLabel": {
    "placeholders": { "count": { "type": "int" } }
  },
  "approvedLabel": "Approved",
  "fileNewClaimButton": "File New Claim",
  "claimHistorySection": "Claim History",
  "noClaimHistory": "No claim history yet",
  "loadFailedClaims": "Failed to load claims: {error}",
  "@loadFailedClaims": {
    "placeholders": { "error": { "type": "String" } }
  },

  "claimAmountLabel": "Amount (Baht)",
  "claimReasonLabel": "Reason / Details",
  "claimReasonHint": "Describe the reason and details of your claim...",
  "filingInProgress": "Filing...",
  "confirmFiling": "Confirm Filing",
  "amountMustBePositive": "Please enter an amount greater than 0",
  "filingFailed": "Filing failed: {error}",
  "@filingFailed": {
    "placeholders": { "error": { "type": "String" } }
  },
  "claimFiledSuccess": "Claim filed successfully",
  "claimReferenceLabel": "Reference Number",
  "close": "Close",

  "navHome": "Home",
  "navProfile": "Profile",

  "statusActive": "Active",
  "statusLapsed": "Lapsed",
  "statusPending": "Pending",

  "claimStatusSubmitted": "Submitted",
  "claimStatusReviewing": "Reviewing",
  "claimStatusApproved": "Approved",
  "claimStatusRejected": "Rejected",
  "claimRequestLabel": "Claim Request"
}
```

> 🔗 **การเชื่อมโยง:** key ส่วนใหญ่กลายเป็น getter ธรรมดา แต่ key ที่มี placeholder เช่น `forAccountEmail` (`{email}`) จะถูก gen เป็น **method** ที่รับพารามิเตอร์ (`l10n.forAccountEmail(email)`) แทน getter · เวลาจะเพิ่มข้อความใหม่: เพิ่ม key ใน **ทั้งสองไฟล์** → รัน `flutter gen-l10n` (หรือแค่ `flutter run`) → เรียกใช้ด้วย `context.l10n.key` ได้ทันที

### ขั้นที่ 13 — หน้าเดิมทั้งหมดที่ปรับเป็น l10n + palette + router (ครบทุกไฟล์ที่แก้)

ไฟล์กลุ่มนี้ถูกแก้ไปในทิศทางเดียวกันทั้งหมด: ข้อความไทยที่เคย hardcode → `context.l10n.*`, สีบางจุด → `context.palette`, การนำทาง → go_router (`context.push` / `context.go`) และ extension แปลงสถานะเป็นข้อความ (`label()`) เปลี่ยนมารับ `AppLocalizations` เป็นพารามิเตอร์

#### 13.1 `lib/features/policy/domain/policy.dart` ✏️

```dart
// lib/features/policy/domain/policy.dart
import 'package:freezed_annotation/freezed_annotation.dart';

import '../../../l10n/app_localizations.dart';

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

/// ส่วนขยายเพื่อแปลงสถานะเป็นข้อความตามภาษาที่เลือก (กฎทางธุรกิจอยู่ในชั้น Domain)
extension PolicyStatusLabel on PolicyStatus {
  String label(AppLocalizations l) {
    switch (this) {
      case PolicyStatus.active:
        return l.statusActive;
      case PolicyStatus.lapsed:
        return l.statusLapsed;
      case PolicyStatus.pending:
        return l.statusPending;
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

> 🔗 **การเชื่อมโยง:** จุดที่เปลี่ยนจาก Day 2 คือ `label()` ไม่คืนสตริงไทยตายตัวอีกต่อไป แต่รับ `AppLocalizations` แล้วเลือกข้อความตามภาษาปัจจุบัน (`l.statusActive` ฯลฯ) — หน้า UI จะเรียก `policy.status.label(context.l10n)` ส่วน `PolicyFake` สำหรับ skeleton ไม่เปลี่ยนแปลง

#### 13.2 `lib/features/claim/domain/claim_record.dart` ✏️

```dart
// lib/features/claim/domain/claim_record.dart
import '../../../l10n/app_localizations.dart';

/// สถานะคำขอสินไหม
enum ClaimStatus { submitted, reviewing, approved, rejected }

extension ClaimStatusLabel on ClaimStatus {
  String label(AppLocalizations l) {
    switch (this) {
      case ClaimStatus.submitted:
        return l.claimStatusSubmitted;
      case ClaimStatus.reviewing:
        return l.claimStatusReviewing;
      case ClaimStatus.approved:
        return l.claimStatusApproved;
      case ClaimStatus.rejected:
        return l.claimStatusRejected;
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

> 🔗 **การเชื่อมโยง:** แพตเทิร์นเดียวกับ `policy.dart` — `ClaimStatusLabel.label()` รับ `AppLocalizations` เพื่อให้ป้ายสถานะสินไหม (ยื่นแล้ว/กำลังพิจารณา/อนุมัติ/ไม่อนุมัติ) สลับภาษาตาม locale ที่ผู้ใช้เลือกในหน้าโปรไฟล์

#### 13.3 `lib/features/claim/presentation/controllers/claim_controller.dart` ✏️

```dart
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
    state = await AsyncValue.guard(() async {
      final claimRef = await ref.read(policyRepositoryProvider).submitClaim(
            policyId: policyId,
            amount: amount,
            reason: reason,
          );
      // ยื่นสำเร็จ → สั่งให้รายการสินไหม + กรมธรรม์โหลดใหม่
      ref.invalidate(claimListProvider);
      ref.invalidate(policyListProvider);
      return claimRef;
    });
  }
}
```

> 🔗 **การเชื่อมโยง:** จุดต่างจาก Day 2 คือบรรทัด `state = const AsyncValue.loading();` ถูก **ลบออก** — `submit()` ตั้งค่า state เฉพาะผลลัพธ์จาก `AsyncValue.guard` เท่านั้น เพราะ `FileClaimSheet` มี flag `_submitting` ของตัวเองสำหรับแสดงสถานะกำลังยื่นอยู่แล้ว การตั้ง loading ซ้ำในนี้จึงไม่จำเป็นและทำให้ UI กะพริบ

#### 13.4 `lib/features/home/presentation/pages/home_page.dart` ✏️

```dart
// lib/features/home/presentation/pages/home_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/l10n/l10n_ext.dart';
import '../../../../l10n/app_localizations.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_spacing.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_card.dart';
import '../../../../core/widgets/gradient_header.dart';
import '../../../../core/widgets/section_label.dart';
import '../../../../core/widgets/status_badge.dart';
import 'package:go_router/go_router.dart';

import '../../../auth/presentation/controllers/auth_controller.dart';
import '../../../claim/presentation/widgets/file_claim_sheet.dart';

/// หน้าแรก (Dashboard) — สรุปวงเงิน, เบี้ยครบกำหนด, ปุ่มลัด, การแจ้งเตือน
/// Day 1: ข้อมูล mock แบบ static — Day 2 จะดึงจาก API จริง
class HomePage extends ConsumerWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final l = context.l10n;
    final user = ref.watch(authControllerProvider).valueOrNull;
    final displayName = user?.displayName ?? l.navProfile;

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
                      Text(l.helloGreeting,
                          style: AppType.caption.copyWith(color: Colors.white70)),
                      Text(displayName,
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
                const _SummaryCard(),
                const SizedBox(height: 14),
                const _DuePremiumCard(),
                const SizedBox(height: 18),
                const _QuickActions(),
                const SizedBox(height: 8),
                SectionLabel(l.recentNotificationsLabel,
                    trailing: Text(l.seeAllLabel,
                        style: AppType.label
                            .copyWith(color: AppColors.primary))),
                ..._buildNotifications(l).map((n) => Padding(
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
  const _SummaryCard();

  @override
  Widget build(BuildContext context) {
    final l = context.l10n;
    return AppCard(
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(l.totalCoverageLabel, style: AppType.label),
          const SizedBox(height: 4),
          Text('฿1,250,000', style: AppType.display),
          const SizedBox(height: 14),
          Row(
            children: [
              Expanded(
                  child: _MiniStat(
                      label: l.activePoliciesLabel,
                      value: l.activePoliciesMockValue)),
              const SizedBox(width: 12),
              Expanded(
                  child: _MiniStat(
                      label: l.annualPremiumLabel, value: '฿60,000')),
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
  const _DuePremiumCard();

  @override
  Widget build(BuildContext context) {
    final l = context.l10n;
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
                Text(l.premiumDueMock,
                    style: AppType.bodyStrong.copyWith(color: Colors.white)),
                const SizedBox(height: 2),
                Text(l.policyDueDateMock,
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
            child: Text(l.payShort),
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
    final l = context.l10n;
    final actions = <({AppIconData icon, String label, VoidCallback onTap})>[
      (
        icon: HugeIcons.strokeRoundedCreditCard,
        label: l.payPremium,
        onTap: () => context.go('/payments'),
      ),
      (
        icon: HugeIcons.strokeRoundedNoteAdd,
        label: l.fileClaim,
        onTap: () => showFileClaimSheet(context),
      ),
      (
        icon: HugeIcons.strokeRoundedShield01,
        label: l.navPolicies,
        onTap: () => context.go('/policies'),
      ),
      (
        icon: HugeIcons.strokeRoundedCustomerService01,
        label: l.contactUs,
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

List<_Noti> _buildNotifications(AppLocalizations l) => [
      _Noti(HugeIcons.strokeRoundedCreditCard, l.notifPremiumDueTitle,
          l.notifPremiumDueSubtitleMock, l.notifDaysLeftMock, BadgeTone.info),
      _Noti(
          HugeIcons.strokeRoundedCheckmarkCircle02,
          l.notifClaimApprovedTitle,
          l.notifClaimApprovedSubtitleMock,
          l.notifDaysAgoMock,
          BadgeTone.success),
      _Noti(HugeIcons.strokeRoundedShield01, l.activePoliciesLabel,
          l.notifPolicyActiveSubtitleMock, l.notifWeekAgoMock, BadgeTone.success),
    ];
```

> 🔗 **การเชื่อมโยง:** ปุ่มลัด (Quick Actions) เปลี่ยนจากการตั้งค่า `mainTabProvider` (ซึ่งหายไปพร้อม `MainShell`) มาเป็น `context.go('/payments')` / `context.go('/policies')` ผ่าน go_router — เปลี่ยนแท็บ = เปลี่ยน URL · ข้อความ mock ทั้งหมด (การแจ้งเตือน, เบี้ยครบกำหนด) ย้ายเข้า .arb แล้วสร้างผ่าน `_buildNotifications(l)` ที่รับ `AppLocalizations`

#### 13.5 `lib/features/policy/presentation/pages/policy_list_page.dart` ✏️

```dart
// lib/features/policy/presentation/pages/policy_list_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:skeletonizer/skeletonizer.dart';
import 'package:go_router/go_router.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/l10n/l10n_ext.dart';
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
    final l = context.l10n;
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
          title: l.myPoliciesTitle,
          subtitle: l.totalPoliciesSubtitle(total),
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
                  decoration: InputDecoration(
                    prefixIcon: const HugeIcon(
                        icon: HugeIcons.strokeRoundedSearch01,
                        color: AppColors.textSubtle),
                    hintText: l.searchPolicyHint,
                  ),
                ),
                const SizedBox(height: 12),
                // แถบกรองสถานะ
                SingleChildScrollView(
                  scrollDirection: Axis.horizontal,
                  child: Row(
                    children: [
                      _Chip(
                        label: l.filterAll,
                        selected: selected == null,
                        onTap: () => ref
                            .read(policyStatusFilterProvider.notifier)
                            .select(null),
                      ),
                      for (final s in PolicyStatus.values)
                        _Chip(
                          label: s.label(l),
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
                      Text(l.totalPremiumActiveLabel,
                          style: AppType.label
                              .copyWith(color: AppColors.infoText)),
                      const Spacer(),
                      Text(l.premiumPerYear(activePremium.toStringAsFixed(0)),
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
                          Padding(
                            padding: const EdgeInsets.only(top: 40),
                            child: Center(
                                child: Text(l.noPoliciesFound)),
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
            child: Text(context.l10n.retry),
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
    final l = context.l10n;
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
      onTap: () => context.push('/policy/${policy.id}', extra: policy),
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
                    Text(l.premiumPerYear(policy.premium.toStringAsFixed(0)),
                        style: AppType.bodyStrong),
                    const SizedBox(width: 8),
                    StatusBadge(label: policy.status.label(l), tone: tone),
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

> 🔗 **การเชื่อมโยง:** หัวข้อ/ตัวกรอง/ข้อความว่าง ทั้งหมดมาจาก `context.l10n` (รวม key แบบมี placeholder เช่น `totalPoliciesSubtitle(total)`, `premiumPerYear(...)`) · การแตะการ์ดเปลี่ยนจาก `Navigator.push` เป็น `context.push('/policy/${policy.id}', extra: policy)` — ส่ง Policy ผ่าน `extra` ของ go_router ให้ route `/policy/:id` ใน `app_router.dart` รับไปสร้าง `PolicyDetailPage`

#### 13.6 `lib/features/policy/presentation/pages/policy_detail_page.dart` ✏️

```dart
// lib/features/policy/presentation/pages/policy_detail_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/l10n/l10n_ext.dart';
import '../../../auth/presentation/controllers/auth_controller.dart';
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
class PolicyDetailPage extends ConsumerWidget {
  final Policy policy;
  const PolicyDetailPage({super.key, required this.policy});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final l = context.l10n;
    final user = ref.watch(authControllerProvider).valueOrNull;
    final displayName = user?.displayName ?? l.insuredLabel;

    return Scaffold(
      appBar: AppBar(
        backgroundColor: AppColors.primary,
        title: Text(l.policyDetailsTitle,
            style: const TextStyle(color: Colors.white, fontWeight: FontWeight.w600)),
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
                _MiniBadge(label: policy.status.label(l), status: policy.status),
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
                      Text(l.coverageAmountLabel,
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
                  child: Text(l.policyInfoSectionTitle, style: AppType.h3),
                ),
                const SizedBox(height: 8),
                const Divider(),
                _row(l.insuredLabel, displayName),
                _row(l.premiumLabel, l.premiumPerYear(policy.premium.toStringAsFixed(0))),
                _row(l.startDateLabel, l.startDateMock),
                _row(l.endDateLabel, l.endDateMock),
              ],
            ),
          ),
          const SizedBox(height: 16),
          Row(
            children: [
              Expanded(
                child: AppButton.secondary(
                    label: l.payPremium,
                    icon: HugeIcons.strokeRoundedCreditCard,
                    onPressed: () {}),
              ),
              const SizedBox(width: 12),
              Expanded(
                child: AppButton.primary(
                    label: l.fileClaim,
                    icon: HugeIcons.strokeRoundedNoteAdd,
                    onPressed: () => showFileClaimSheet(context,
                        policyId: policy.id, policyName: policy.planName)),
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

> 🔗 **การเชื่อมโยง:** อัปเกรดเป็น `ConsumerWidget` เพื่อดึงชื่อ "ผู้เอาประกัน" จริงจาก `authControllerProvider` (ไม่ hardcode อีกต่อไป) และทุก label มาจาก `context.l10n` — หน้านี้ถูกเปิดจาก route `/policy/:id` ที่รับ `Policy` ผ่าน `state.extra` ใน `app_router.dart`

#### 13.7 `lib/features/claim/presentation/pages/claims_page.dart` ✏️

```dart
// lib/features/claim/presentation/pages/claims_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:skeletonizer/skeletonizer.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/l10n/l10n_ext.dart';
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
    final l = context.l10n;
    final asyncClaims = ref.watch(claimListProvider);
    final claims = asyncClaims.valueOrNull ?? const <ClaimRecord>[];

    final approvedSum = claims
        .where((c) => c.status == ClaimStatus.approved)
        .fold<double>(0, (s, c) => s + c.amount);

    return Column(
      children: [
        GradientHeader(
          title: l.claimsTitle,
          subtitle: l.claimsSubtitle,
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
                            label: l.totalClaimsLabel,
                            value: l.claimsCountLabel(claims.length))),
                    const SizedBox(width: 12),
                    Expanded(
                        child: _Stat(
                            label: l.approvedLabel,
                            value: '฿${money(approvedSum)}',
                            valueColor: AppColors.successFg)),
                  ],
                ),
                const SizedBox(height: 14),
                AppButton.primary(
                    label: l.fileNewClaimButton,
                    icon: HugeIcons.strokeRoundedAdd01,
                    onPressed: () => showFileClaimSheet(context)),
                const SizedBox(height: 8),
                SectionLabel(l.claimHistorySection),
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
                      child: Center(child: Text(l.loadFailedClaims('$e'))),
                    )
                  ],
                  data: (list) => list.isEmpty
                      ? [
                          Padding(
                            padding: const EdgeInsets.only(top: 30),
                            child: Center(child: Text(l.noClaimHistory)),
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
    final l = context.l10n;
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
                    Text(l.claimRequestLabel, style: AppType.bodyStrong),
                    const SizedBox(height: 2),
                    Text(c.policyNumber, style: AppType.caption),
                  ],
                ),
              ),
              StatusBadge(label: c.status.label(l), tone: tone),
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

> 🔗 **การเชื่อมโยง:** โครงสร้างเหมือน Day 2 แต่ทุกข้อความมาจาก `context.l10n` — รวมทั้งป้ายสถานะบน badge ที่เรียก `c.status.label(l)` จาก extension ใน `claim_record.dart` (13.2) และ key แบบมี placeholder อย่าง `claimsCountLabel(claims.length)` / `loadFailedClaims('$e')`

#### 13.8 `lib/features/claim/presentation/widgets/file_claim_sheet.dart` ✏️

```dart
// lib/features/claim/presentation/widgets/file_claim_sheet.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/l10n/l10n_ext.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_spacing.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_text_field.dart';
import '../../../../l10n/app_localizations.dart';
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
    final l = context.l10n;
    final amount = double.tryParse(_amount.text.trim()) ?? 0;
    if (amount <= 0) {
      _toast(l.amountMustBePositive);
      return;
    }
    setState(() => _submitting = true);

    // Day 2: ถ้าเปิดจากหน้าสินไหมรวม (ไม่ระบุกรมธรรม์) ใช้ค่าเริ่มต้น P001 เพื่อสาธิต
    await ref.read(claimControllerProvider.notifier).submit(
          policyId: widget.policyId ?? 'P001',
          amount: amount,
          reason: _reason.text.trim(),
        );

    if (!mounted) return;
    final result = ref.read(claimControllerProvider);
    setState(() => _submitting = false);
    result.when(
      data: (ref0) {
        if (ref0 != null) setState(() => _claimRef = ref0);
      },
      error: (e, _) => _toast(l.filingFailed('$e')),
      loading: () {},
    );
  }

  void _toast(String msg) {
    ScaffoldMessenger.of(context)
        .showSnackBar(SnackBar(content: Text(msg)));
  }

  @override
  Widget build(BuildContext context) {
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
          if (_claimRef == null) ..._form(context.l10n) else ..._success(context.l10n, _claimRef!),
        ],
      ),
    );
  }

  // ── ฟอร์มกรอกข้อมูลการยื่นสินไหม ───────────────────────────────
  List<Widget> _form(AppLocalizations l) {
    return [
      Text(l.fileClaim, style: AppType.h2),
      if (widget.policyName != null) ...[
        const SizedBox(height: 2),
        Text(widget.policyName!,
            style: AppType.caption.copyWith(color: AppColors.primary)),
      ],
      const SizedBox(height: 16),
      AppTextField(
        label: l.claimAmountLabel,
        hint: '0.00',
        keyboardType: const TextInputType.numberWithOptions(decimal: true),
        controller: _amount,
      ),
      const SizedBox(height: 14),
      AppTextField(
        label: l.claimReasonLabel,
        hint: l.claimReasonHint,
        maxLines: 4,
        controller: _reason,
      ),
      const SizedBox(height: 20),
      AppButton.primary(
        label: _submitting ? l.filingInProgress : l.confirmFiling,
        onPressed: _submitting ? null : _submit,
      ),
    ];
  }

  // ── หน้ายืนยันยื่นสำเร็จ ───────────────────────────────────────
  List<Widget> _success(AppLocalizations l, String ref) {
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
      Center(child: Text(l.claimFiledSuccess, style: AppType.h2)),
      const SizedBox(height: 4),
      Center(
        child: Text(l.claimReferenceLabel,
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
        label: l.close,
        onPressed: () => Navigator.of(context).pop(),
      ),
    ];
  }
}
```

> 🔗 **การเชื่อมโยง:** ฟอร์ม/ข้อความสำเร็จทั้งหมดรับ `AppLocalizations` เป็นพารามิเตอร์ (`_form(l)` / `_success(l, ref)`) — sheet นี้จัดการสถานะกำลังยื่นด้วย `_submitting` ของตัวเอง ซึ่งเป็นเหตุผลที่ `ClaimController.submit()` (13.3) ไม่ต้องตั้ง `AsyncValue.loading()` อีก และเมื่อยื่นสำเร็จ controller จะ invalidate `claimListProvider` ให้ `ClaimsPage` (13.7) โหลดรายการใหม่เอง

#### 13.9 `lib/features/payment/presentation/pages/payments_page.dart` ✏️

```dart
// lib/features/payment/presentation/pages/payments_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:skeletonizer/skeletonizer.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/l10n/l10n_ext.dart';
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
    final l = context.l10n;
    final async = ref.watch(paymentHistoryControllerProvider);
    final hasMore = ref.read(paymentHistoryControllerProvider.notifier).hasMore;

    return Column(
      children: [
        GradientHeader(
          title: l.paymentsPageTitle,
          subtitle: l.paymentsSubtitle,
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
                SectionLabel(l.paymentMethodsSection),
                _MethodTile(
                    icon: HugeIcons.strokeRoundedQrCode,
                    title: l.promptpayQrTitle,
                    subtitle: l.bankTransferSubtitle,
                    trailing: StatusBadge(
                        label: l.defaultBadge, tone: BadgeTone.info),
                    selected: true),
                const SizedBox(height: 10),
                _MethodTile(
                    icon: HugeIcons.strokeRoundedCreditCard,
                    title: l.creditCardTitleMock,
                    subtitle: 'KBank Visa Platinum',
                    trailing: const HugeIcon(
                        icon: HugeIcons.strokeRoundedArrowRight01,
                        color: AppColors.textSubtle)),
                const SizedBox(height: 10),
                DottedAddBox(label: l.addPaymentMethod, onTap: () {}),
                const SizedBox(height: 8),
                SectionLabel(l.paymentHistorySection),
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
                        child: Center(child: Text(l.loadFailedPayments('$e'))))
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
  const _DueCard();

  @override
  Widget build(BuildContext context) {
    final l = context.l10n;
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
              Text(l.paymentDueDaysMock,
                  style: AppType.tiny.copyWith(color: Colors.white)),
            ]),
          ),
          const SizedBox(height: 12),
          Text(l.mockPlanName,
              style: AppType.body.copyWith(color: Colors.white70)),
          Text('฿50,000', style: AppType.display.copyWith(color: Colors.white)),
          const SizedBox(height: 2),
          Text(l.dueDateInfoMock,
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
              label: Text(l.payNowButton),
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

> 🔗 **การเชื่อมโยง:** Pagination / Infinite Scroll ทำงานเหมือน Day 2 ทุกประการ — สิ่งที่เปลี่ยนคือหัวเรื่อง, ช่องทางชำระ, ป้าย "ค่าเริ่มต้น" และข้อความ error ทั้งหมดย้ายไป .arb แล้วเรียกผ่าน `context.l10n` (เช่น `loadFailedPayments('$e')`) — หน้านี้ถูกเข้าถึงทั้งจากแท็บล่างและปุ่มลัด `context.go('/payments')` ใน HomePage (13.4)

#### 13.10 `lib/features/profile/presentation/pages/profile_page.dart` ✏️

```dart
// lib/features/profile/presentation/pages/profile_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/l10n/l10n_ext.dart';
import '../../../../core/settings/settings_providers.dart';
import '../../../../core/theme/app_colors.dart';
import '../../../../core/theme/app_palette.dart';
import '../../../../core/theme/app_spacing.dart';
import '../../../../core/theme/app_typography.dart';
import '../../../../core/widgets/gradient_header.dart';
import '../../../../core/widgets/section_label.dart';
import '../../../auth/presentation/controllers/auth_controller.dart';

/// หน้าโปรไฟล์ — ข้อมูลผู้ใช้ + เมนูบัญชี + การตั้งค่า
/// Day 3 Stage 3: แก้ไขโปรไฟล์จริง, ลิงก์เมนูใช้งานได้ทั้งหมด,
/// เปลี่ยนรหัสผ่าน, สลับภาษา (ไทย/EN) และสลับโหมดสว่าง/มืด — เก็บค่าด้วย SharedPreferences
class ProfilePage extends ConsumerStatefulWidget {
  const ProfilePage({super.key});

  @override
  ConsumerState<ProfilePage> createState() => _ProfilePageState();
}

class _ProfilePageState extends ConsumerState<ProfilePage> {
  bool _noti = true; // การแจ้งเตือน — เดโม (local)

  void _toast(String msg) =>
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(msg)));

  @override
  Widget build(BuildContext context) {
    final l = context.l10n;
    final p = context.palette;
    final user = ref.watch(authControllerProvider).valueOrNull;
    final displayName = user?.displayName ?? 'ผู้ใช้';
    final roleLabel = user?.roleLabel ?? 'ลูกค้า';

    // ค่าจาก provider (reactive + persisted)
    final isThai = ref.watch(localeControllerProvider).languageCode == 'th';
    final isDark = ref.watch(themeModeControllerProvider) == ThemeMode.dark;

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
                    Text(displayName,
                        style: AppType.h2.copyWith(color: Colors.white)),
                    Text('$roleLabel · BLA Insurance',
                        style: AppType.caption.copyWith(color: Colors.white70)),
                    const SizedBox(height: 6),
                    Container(
                      padding: const EdgeInsets.symmetric(
                          horizontal: 8, vertical: 3),
                      decoration: BoxDecoration(
                          color: Colors.white24,
                          borderRadius:
                              BorderRadius.circular(AppSpacing.rPill)),
                      child: Row(mainAxisSize: MainAxisSize.min, children: [
                        const HugeIcon(
                            icon: HugeIcons.strokeRoundedCheckmarkBadge01,
                            size: 13,
                            color: Colors.white),
                        const SizedBox(width: 4),
                        Text(l.verifiedBadge,
                            style: AppType.tiny.copyWith(color: Colors.white)),
                      ]),
                    ),
                  ],
                ),
              ),
              // ปุ่มแก้ไข → หน้าแก้ไขข้อมูลส่วนตัว
              CircleHeaderButton(
                icon: HugeIcons.strokeRoundedEdit02,
                onTap: () => context.push('/edit-profile'),
              ),
            ],
          ),
        ),
        Expanded(
          child: ListView(
            padding: const EdgeInsets.fromLTRB(16, 8, 16, 16),
            children: [
              SectionLabel(l.accountSection),
              _MenuCard(
                children: [
                  _NavRow(
                    icon: HugeIcons.strokeRoundedUser,
                    label: l.menuPersonalInfo,
                    onTap: () => context.push('/edit-profile'),
                  ),
                  _divider(p),
                  _NavRow(
                    icon: HugeIcons.strokeRoundedShield01,
                    label: l.menuMyPolicies,
                    onTap: () => context.go('/policies'),
                  ),
                  _divider(p),
                  _NavRow(
                    icon: HugeIcons.strokeRoundedClock01,
                    label: l.menuPaymentHistory,
                    onTap: () => context.go('/payments'),
                  ),
                ],
              ),
              SectionLabel(l.settingsSection),
              _MenuCard(
                children: [
                  _ToggleRow(
                    icon: HugeIcons.strokeRoundedNotification02,
                    label: l.notifications,
                    value: _noti,
                    onChanged: (v) => setState(() => _noti = v),
                  ),
                  _divider(p),
                  _NavRow(
                    icon: HugeIcons.strokeRoundedSquareLock01,
                    label: l.changePassword,
                    onTap: () => context.push('/change-password'),
                  ),
                  _divider(p),
                  _NavRow(
                    icon: HugeIcons.strokeRoundedFingerPrintScan,
                    label: l.setupPin,
                    onTap: () => context.push('/pin-setup'),
                  ),
                  _divider(p),
                  _SegmentRow(
                    icon: HugeIcons.strokeRoundedGlobe02,
                    label: l.language,
                    isThai: isThai,
                    onChanged: (thai) => ref
                        .read(localeControllerProvider.notifier)
                        .setThai(thai),
                  ),
                  _divider(p),
                  _ToggleRow(
                    icon: HugeIcons.strokeRoundedMoon02,
                    label: l.darkMode,
                    value: isDark,
                    onChanged: (v) => ref
                        .read(themeModeControllerProvider.notifier)
                        .setDark(v),
                  ),
                ],
              ),
              SectionLabel(l.helpSection),
              _MenuCard(
                children: [
                  _NavRow(
                    icon: HugeIcons.strokeRoundedCustomerService01,
                    label: l.helpContactAgent,
                    onTap: () => _toast(l.comingSoon),
                  ),
                  _divider(p),
                  _NavRow(
                    icon: HugeIcons.strokeRoundedHelpCircle,
                    label: l.helpFaq,
                    onTap: () => _toast(l.comingSoon),
                  ),
                ],
              ),
              const SizedBox(height: 20),
              _LogoutButton(
                label: l.logout,
                onTap: () =>
                    ref.read(authControllerProvider.notifier).logout(),
              ),
              const SizedBox(height: 14),
              Center(
                child: Text('BLA Policy Companion · v1.0.0',
                    style: AppType.caption.copyWith(color: p.textSubtle)),
              ),
              const SizedBox(height: 8),
            ],
          ),
        ),
      ],
    );
  }

  Widget _divider(AppPalette p) => Divider(height: 1, color: p.border);
}

/// การ์ดห่อรายการเมนู — palette-aware (สลับสว่าง/มืดตามธีม)
class _MenuCard extends StatelessWidget {
  final List<Widget> children;
  const _MenuCard({required this.children});

  @override
  Widget build(BuildContext context) {
    final p = context.palette;
    return Container(
      width: double.infinity,
      padding: const EdgeInsets.symmetric(horizontal: 8),
      decoration: BoxDecoration(
        color: p.card,
        borderRadius: BorderRadius.circular(AppSpacing.rMd),
        border: Border.all(color: p.border, width: 1),
      ),
      child: Material(
        type: MaterialType.transparency,
        child: Column(children: children),
      ),
    );
  }
}

/// ปุ่มออกจากระบบ — สไตล์ danger (พื้นแดงอ่อน ตัวอักษร/ไอคอนสีแดง)
class _LogoutButton extends StatelessWidget {
  final String label;
  final VoidCallback onTap;
  const _LogoutButton({required this.label, required this.onTap});

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
              const HugeIcon(
                  icon: HugeIcons.strokeRoundedLogout01,
                  size: 20,
                  color: AppColors.dangerFg),
              const SizedBox(width: 8),
              Text(label,
                  style: AppType.bodyStrong.copyWith(color: AppColors.dangerFg)),
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
  final VoidCallback? onTap;
  const _NavRow({required this.icon, required this.label, this.onTap});

  @override
  Widget build(BuildContext context) {
    final p = context.palette;
    return ListTile(
      contentPadding: const EdgeInsets.symmetric(horizontal: 8),
      leading: HugeIcon(icon: icon, color: AppColors.primary, size: 22),
      title: Text(label,
          style: AppType.bodyStrong.copyWith(color: p.textDark)),
      trailing: HugeIcon(
          icon: HugeIcons.strokeRoundedArrowRight01, color: p.textSubtle),
      onTap: onTap ?? () {},
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
    final p = context.palette;
    return ListTile(
      contentPadding: const EdgeInsets.symmetric(horizontal: 8),
      leading: HugeIcon(icon: icon, color: AppColors.primary, size: 22),
      title: Text(label,
          style: AppType.bodyStrong.copyWith(color: p.textDark)),
      trailing: Switch(
          value: value,
          onChanged: onChanged,
          activeThumbColor: AppColors.primary),
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
    final p = context.palette;
    return ListTile(
      contentPadding: const EdgeInsets.symmetric(horizontal: 8),
      leading: HugeIcon(icon: icon, color: AppColors.primary, size: 22),
      title: Text(label,
          style: AppType.bodyStrong.copyWith(color: p.textDark)),
      trailing: Container(
        decoration: BoxDecoration(
            color: p.background,
            borderRadius: BorderRadius.circular(AppSpacing.rPill)),
        padding: const EdgeInsets.all(3),
        child: Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            _seg(context, 'ไทย', isThai, () => onChanged(true)),
            _seg(context, 'EN', !isThai, () => onChanged(false)),
          ],
        ),
      ),
    );
  }

  Widget _seg(BuildContext context, String t, bool on, VoidCallback onTap) {
    return GestureDetector(
      onTap: onTap,
      child: Container(
        padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 5),
        decoration: BoxDecoration(
            color: on ? AppColors.primary : Colors.transparent,
            borderRadius: BorderRadius.circular(AppSpacing.rPill)),
        child: Text(t,
            style: AppType.tiny.copyWith(
                color: on ? Colors.white : context.palette.textMedium,
                fontWeight: FontWeight.w600)),
      ),
    );
  }
}
```

> 🔗 **การเชื่อมโยง:** หน้านี้เปลี่ยนมากที่สุด — ชื่อ/บทบาทผู้ใช้ดึงจริงจาก `authControllerProvider` (ไม่ใช่ mock), ทุกเมนูใช้งานได้จริงผ่าน `context.push('/edit-profile')` / `context.push('/change-password')` / `context.push('/pin-setup')` / `context.go('/policies', '/payments')` · ตัวสลับภาษาผูกกับ `localeControllerProvider.setThai()` และสวิตช์โหมดมืดผูกกับ `themeModeControllerProvider.setDark()` (persist ลง SharedPreferences) · ปุ่มออกจากระบบเรียก `logout()` จริง → router redirect กลับหน้า login เอง · UI ภายในเป็น palette-aware (`context.palette`) เพื่อรองรับ dark mode

#### 13.11 `lib/features/profile/presentation/pages/edit_profile_page.dart` 🆕

```dart
// lib/features/profile/presentation/pages/edit_profile_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/l10n/l10n_ext.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_text_field.dart';
import '../../../../core/widgets/gradient_header.dart';
import '../../../auth/presentation/controllers/auth_controller.dart';

/// หน้าแก้ไขข้อมูลส่วนตัว — Day 3 Stage 3
/// แก้ไขชื่อ/เบอร์โทร → PUT /auth/me → อัปเดต AuthController (state ทั้งแอปเปลี่ยนตาม)
class EditProfilePage extends ConsumerStatefulWidget {
  const EditProfilePage({super.key});

  @override
  ConsumerState<EditProfilePage> createState() => _EditProfilePageState();
}

class _EditProfilePageState extends ConsumerState<EditProfilePage> {
  late final TextEditingController _name;
  late final TextEditingController _phone;
  bool _submitting = false;

  @override
  void initState() {
    super.initState();
    final user = ref.read(authControllerProvider).valueOrNull;
    _name = TextEditingController(text: user?.displayName ?? '');
    _phone = TextEditingController(text: user?.phone ?? '');
  }

  @override
  void dispose() {
    _name.dispose();
    _phone.dispose();
    super.dispose();
  }

  void _toast(String msg) =>
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(msg)));

  Future<void> _save() async {
    if (_name.text.trim().isEmpty) {
      _toast(context.l10n.nameRequired);
      return;
    }
    setState(() => _submitting = true);
    try {
      await ref.read(authControllerProvider.notifier).updateProfile(
            displayName: _name.text.trim(),
            phone: _phone.text.trim(),
          );
      if (!mounted) return;
      _toast(context.l10n.profileUpdated);
      context.pop();
    } catch (e) {
      if (!mounted) return;
      setState(() => _submitting = false);
      _toast('$e');
    }
  }

  @override
  Widget build(BuildContext context) {
    final l = context.l10n;
    final bottomInset = MediaQuery.paddingOf(context).bottom;
    return Scaffold(
      body: ListView(
        padding: EdgeInsets.zero,
        children: [
          GradientHeader(
            showBack: true,
            title: l.editProfileTitle,
            subtitle: l.editProfileSubtitle,
          ),
          Padding(
            padding: EdgeInsets.fromLTRB(20, 22, 20, 28 + bottomInset),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                AppTextField(
                  label: l.fullName,
                  icon: HugeIcons.strokeRoundedUser,
                  controller: _name,
                ),
                const SizedBox(height: 16),
                AppTextField(
                  label: l.phone,
                  hint: '08X-XXX-XXXX',
                  icon: HugeIcons.strokeRoundedCall,
                  keyboardType: TextInputType.phone,
                  controller: _phone,
                ),
                const SizedBox(height: 24),
                AppButton.primary(
                  label: _submitting ? l.saving : l.save,
                  icon: HugeIcons.strokeRoundedCheckmarkCircle02,
                  onPressed: _submitting ? null : _save,
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

> 🔗 **การเชื่อมโยง:** หน้าใหม่ที่ route `/edit-profile` — ดึงค่าปัจจุบันจาก `authControllerProvider` มาเติมฟอร์ม แล้วบันทึกผ่าน `AuthController.updateProfile()` ซึ่งยิง `PUT /auth/me` และอัปเดต state ผู้ใช้กลาง → ชื่อบน HomePage / ProfilePage / PolicyDetailPage เปลี่ยนตามทันทีทั้งแอป

#### 13.12 `lib/features/profile/presentation/pages/change_password_page.dart` 🆕

```dart
// lib/features/profile/presentation/pages/change_password_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';

import '../../../../core/icons/app_icons.dart';
import '../../../../core/l10n/l10n_ext.dart';
import '../../../../core/widgets/app_button.dart';
import '../../../../core/widgets/app_text_field.dart';
import '../../../../core/widgets/gradient_header.dart';
import '../../../auth/presentation/controllers/auth_controller.dart';

/// หน้าเปลี่ยนรหัสผ่าน — Day 3 Stage 3
/// ตรวจฝั่งแอป (รหัสใหม่ตรงกัน/ยาวพอ) → POST /auth/change-password (ตรวจรหัสเดิมฝั่ง server)
class ChangePasswordPage extends ConsumerStatefulWidget {
  const ChangePasswordPage({super.key});

  @override
  ConsumerState<ChangePasswordPage> createState() => _ChangePasswordPageState();
}

class _ChangePasswordPageState extends ConsumerState<ChangePasswordPage> {
  final _current = TextEditingController();
  final _next = TextEditingController();
  final _confirm = TextEditingController();
  bool _submitting = false;

  @override
  void dispose() {
    _current.dispose();
    _next.dispose();
    _confirm.dispose();
    super.dispose();
  }

  void _toast(String msg) =>
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(msg)));

  Future<void> _submit() async {
    final l = context.l10n;
    if (_next.text.length < 4) {
      _toast(l.passwordTooShort);
      return;
    }
    if (_next.text != _confirm.text) {
      _toast(l.passwordMismatch);
      return;
    }
    setState(() => _submitting = true);
    try {
      await ref.read(authControllerProvider.notifier).changePassword(
            currentPassword: _current.text,
            newPassword: _next.text,
          );
      if (!mounted) return;
      _toast(l.passwordChanged);
      context.pop();
    } catch (e) {
      if (!mounted) return;
      setState(() => _submitting = false);
      _toast('$e'); // เช่น "รหัสผ่านเดิมไม่ถูกต้อง" จาก server
    }
  }

  @override
  Widget build(BuildContext context) {
    final l = context.l10n;
    final bottomInset = MediaQuery.paddingOf(context).bottom;
    return Scaffold(
      body: ListView(
        padding: EdgeInsets.zero,
        children: [
          GradientHeader(
            showBack: true,
            title: l.changePassword,
            subtitle: l.changePasswordSubtitle,
          ),
          Padding(
            padding: EdgeInsets.fromLTRB(20, 22, 20, 28 + bottomInset),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                AppTextField(
                  label: l.currentPassword,
                  icon: HugeIcons.strokeRoundedSquareLock01,
                  obscure: true,
                  controller: _current,
                ),
                const SizedBox(height: 16),
                AppTextField(
                  label: l.newPassword,
                  icon: HugeIcons.strokeRoundedSquareLock01,
                  obscure: true,
                  controller: _next,
                ),
                const SizedBox(height: 16),
                AppTextField(
                  label: l.confirmPassword,
                  icon: HugeIcons.strokeRoundedSquareLock01,
                  obscure: true,
                  controller: _confirm,
                ),
                const SizedBox(height: 24),
                AppButton.primary(
                  label: _submitting ? l.saving : l.changePassword,
                  icon: HugeIcons.strokeRoundedSecurityCheck,
                  onPressed: _submitting ? null : _submit,
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

> 🔗 **การเชื่อมโยง:** หน้าใหม่ที่ route `/change-password` — validate ฝั่งแอปเฉพาะเรื่องที่รู้ได้เอง (รหัสใหม่ยาวพอ + ยืนยันตรงกัน) ส่วน "รหัสผ่านเดิมถูกไหม" ให้ server ตรวจผ่าน `AuthController.changePassword()` → `POST /auth/change-password` ถ้าผิด server ตอบ error กลับมาแสดงเป็น toast

> 🔗 **สรุปภาพรวมขั้นที่ 13:** ทุกหน้าอ่านข้อความผ่าน `context.l10n` เพียงจุดเดียว ดังนั้นเมื่อผู้ใช้แตะสลับภาษาในหน้าโปรไฟล์ `localeControllerProvider` เปลี่ยน `MaterialApp.locale` แล้วทุกหน้า re-render เป็นภาษาใหม่พร้อมกันทันที · หน้าโปรไฟล์กลายเป็น "ห้องควบคุม" ของแอป — แก้ไขข้อมูลส่วนตัว, เปลี่ยนรหัสผ่าน, ตั้งค่า PIN, สลับภาษา, สลับโหมดมืด และออกจากระบบ ได้จริงทุกเมนู · การนำทางทุกจุดเป็นแบบ URL ผ่าน go_router (`context.push` สำหรับหน้าซ้อน, `context.go` สำหรับสลับแท็บ) แทน `Navigator` และ `mainTabProvider` แบบเดิม · ส่วน domain extension (`label(AppLocalizations)`) ทำให้แม้แต่ป้ายสถานะในชั้น Domain ก็สลับภาษาตามได้โดยไม่ผูกกับ BuildContext

### ขั้นที่ 14 — Testing: Unit Test + Widget Test (ครบ 3 ไฟล์) 🆕✏️

#### 14.1 `test/auth_controller_test.dart` — เทสต์หัวใจของระบบ auth

```dart
// test/auth_controller_test.dart
import 'package:bla_policy_companion/core/storage/secure_storage.dart';
import 'package:bla_policy_companion/features/auth/data/auth_repository.dart';
import 'package:bla_policy_companion/features/auth/domain/auth_user.dart';
import 'package:bla_policy_companion/features/auth/presentation/controllers/auth_controller.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

class MockAuthRepository extends Mock implements AuthRepository {}

class MockSecureStorage extends Mock implements SecureStorageService {}

void main() {
  late MockAuthRepository authRepo;
  late MockSecureStorage storage;

  setUp(() {
    authRepo = MockAuthRepository();
    storage = MockSecureStorage();
    // ค่าเริ่มต้น: ยังไม่มี token → build() คืน null
    when(() => storage.readToken()).thenAnswer((_) async => null);
    when(() => storage.saveToken(any())).thenAnswer((_) async {});
    when(() => storage.deleteToken()).thenAnswer((_) async {});
  });

  ProviderContainer makeContainer() {
    final container = ProviderContainer(overrides: [
      authRepositoryProvider.overrideWithValue(authRepo),
      secureStorageProvider.overrideWithValue(storage),
    ]);
    addTearDown(container.dispose);
    return container;
  }

  const user =
      AuthUser(id: 'C001', displayName: 'คุณสมชาย ใจดี', role: 'customer');

  test('เริ่มต้นไม่มี token → สถานะเป็น null (ยังไม่ล็อกอิน)', () async {
    final container = makeContainer();
    final result = await container.read(authControllerProvider.future);
    expect(result, isNull);
  });

  test('login สำเร็จ → เก็บ token และสถานะเป็น user', () async {
    when(() => authRepo.login('customer', '1234')).thenAnswer(
        (_) async => const LoginResult(user: user, token: 'tok-123'));

    final container = makeContainer();
    await container.read(authControllerProvider.future); // ให้ build เสร็จก่อน

    await container
        .read(authControllerProvider.notifier)
        .login('customer', '1234');

    expect(container.read(authControllerProvider).value?.id, 'C001');
    verify(() => storage.saveToken('tok-123')).called(1);
  });

  test('login ล้มเหลว → สถานะเป็น error และไม่เก็บ token', () async {
    when(() => authRepo.login(any(), any()))
        .thenThrow(Exception('ชื่อผู้ใช้หรือรหัสผ่านไม่ถูกต้อง'));

    final container = makeContainer();
    await container.read(authControllerProvider.future);

    await container.read(authControllerProvider.notifier).login('x', 'y');

    expect(container.read(authControllerProvider).hasError, isTrue);
    verifyNever(() => storage.saveToken(any()));
  });
}
```

#### 14.2 `test/policy_list_controller_test.dart` — เทสต์ provider chain ของ policy

```dart
// test/policy_list_controller_test.dart
import 'package:bla_policy_companion/features/policy/data/policy_repository.dart';
import 'package:bla_policy_companion/features/policy/domain/policy.dart';
import 'package:bla_policy_companion/features/policy/presentation/controllers/policy_list_controller.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

/// สร้าง "ของปลอม" ของ PolicyRepository ที่ควบคุมพฤติกรรมได้
class MockPolicyRepository extends Mock implements PolicyRepository {}

void main() {
  late MockPolicyRepository repo;

  setUp(() => repo = MockPolicyRepository());

  /// สร้าง ProviderContainer ที่ override repository เป็น mock
  /// (ตัดทั้งลูกโซ่ที่อยู่ใต้ repository เช่น Dio/network ออก — ทดสอบเร็ว ไม่ต้องต่อเน็ต)
  ProviderContainer makeContainer() {
    final container = ProviderContainer(overrides: [
      policyRepositoryProvider.overrideWithValue(repo),
    ]);
    addTearDown(container.dispose);
    return container;
  }

  const sample = [
    Policy(
        id: 'P001',
        planName: 'BLA ตลอดชีพ',
        policyNumber: 'BLA-1',
        premium: 1000,
        status: PolicyStatus.active),
    Policy(
        id: 'P002',
        planName: 'BLA สะสมทรัพย์',
        policyNumber: 'BLA-2',
        premium: 2000,
        status: PolicyStatus.lapsed),
  ];

  test('policyList คืนรายการเมื่อ repository สำเร็จ', () async {
    when(() => repo.fetchPolicies()).thenAnswer((_) async => sample);

    final container = makeContainer();
    final result = await container.read(policyListProvider.future);

    expect(result, sample);
    verify(() => repo.fetchPolicies()).called(1);
  });

  test('policyList เป็น error เมื่อ repository ล้มเหลว', () async {
    when(() => repo.fetchPolicies()).thenThrow(Exception('เครือข่ายล่ม'));

    final container = makeContainer();

    await expectLater(
      container.read(policyListProvider.future),
      throwsA(isA<Exception>()),
    );
    expect(container.read(policyListProvider).hasError, isTrue);
  });

  test('filteredPolicyList กรองตามคำค้นหาได้ถูกต้อง', () async {
    when(() => repo.fetchPolicies()).thenAnswer((_) async => sample);

    final container = makeContainer();
    await container.read(policyListProvider.future);

    // ค้นหา "ตลอดชีพ" → เหลือ 1 รายการ
    container.read(policySearchProvider.notifier).update('ตลอดชีพ');
    final filtered = container.read(filteredPolicyListProvider);

    expect(filtered.length, 1);
    expect(filtered.first.id, 'P001');
  });

  test('filteredPolicyList กรองตามสถานะได้ถูกต้อง', () async {
    when(() => repo.fetchPolicies()).thenAnswer((_) async => sample);

    final container = makeContainer();
    await container.read(policyListProvider.future);

    container.read(policyStatusFilterProvider.notifier).select(PolicyStatus.lapsed);
    final filtered = container.read(filteredPolicyListProvider);

    expect(filtered.length, 1);
    expect(filtered.first.status, PolicyStatus.lapsed);
  });
}
```

#### 14.3 `test/widget_test.dart` ✏️ — smoke test ที่ผ่านจริงแล้ว (เฉลยการบ้าน Day 1!)

```dart
// test/widget_test.dart
// Basic smoke test — ตรวจว่าแอป (BlaApp) ขึ้นต้นได้โดยไม่ throw
// (แอปนี้ใช้ Riverpod + GoRouter + SharedPreferences จึงต้องห่อด้วย
// ProviderScope และ override sharedPreferencesProvider ก่อน pump)

import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:shared_preferences/shared_preferences.dart';

import 'package:bla_policy_companion/core/settings/settings_providers.dart';
import 'package:bla_policy_companion/main.dart';

void main() {
  testWidgets('BlaApp starts and shows splash screen',
      (WidgetTester tester) async {
    SharedPreferences.setMockInitialValues({});
    final prefs = await SharedPreferences.getInstance();

    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          sharedPreferencesProvider.overrideWithValue(prefs),
        ],
        child: const BlaApp(),
      ),
    );

    // ยังไม่ pumpAndSettle เพราะ router มี async redirect (splash → login/home)
    // แค่ยืนยันว่าแอป build ได้โดยไม่มี exception ก็เพียงพอสำหรับ smoke test
    // AuthController.build() มี delay 1200ms (แสดง splash) ต้อง pump ให้เกินเวลานั้น
    // ไม่ให้เหลือ pending timer ตอนจบ test
    await tester.pump(const Duration(milliseconds: 1300));

    expect(tester.takeException(), isNull);
  });
}
```

> 🔗 **การเชื่อมโยง / จุดสังเกตของชุดเทสต์:**
>
> - หัวใจคือ **override ที่รอยต่อของชั้น**: เทสต์ controller → override repository (ตัด Dio/เน็ตทิ้งทั้งลูกโซ่), เทสต์ widget → override sharedPreferences ด้วย mock — ทำได้เพราะ DI ทั้งแอปวิ่งผ่าน provider ตามที่วางมาตั้งแต่ Day 1
> - `auth_controller_test` ครอบ 3 สถานการณ์: ไม่มี token / login สำเร็จ (`verify` ว่าเก็บ token จริง 1 ครั้ง) / login พลาด (`verifyNever` ว่าไม่แอบเก็บ)
> - `widget_test.dart` คือ **เฉลยการบ้านจาก Day 1**: template counter เดิมพังเพราะไม่มี ProviderScope + ไม่มีหน้า counter — ฉบับจริงห่อ scope, override prefs และ `pump(1300ms)` ให้พ้น delay ของ AuthController กัน pending timer

### ขั้นที่ 15 — ไฟล์ generated ที่เกิดใหม่ + สร้างโค้ดและรัน

**Generated จาก build_runner** (แพตเทิร์นเดียวกับ Day 1–2 ทุกไฟล์ — เปิดดูได้ในโปรเจกต์):

| ไฟล์ generated                       | provider/ของที่ได้                                                                     | ชนิด                                                                                           |
| ------------------------------------ | -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `secure_storage.g.dart`              | `secureStorageProvider`                                                                | `AutoDisposeProvider<SecureStorageService>`                                                    |
| `crypto_service.g.dart`              | `cryptoServiceProvider`                                                                | `AutoDisposeProvider<CryptoService>`                                                           |
| `device_info_service.g.dart`         | `deviceInfoServiceProvider`                                                            | `AutoDisposeProvider<DeviceInfoService>`                                                       |
| `router_refresh.g.dart`              | `routerRefreshProvider`                                                                | `AutoDisposeProvider<Listenable>`                                                              |
| `app_router.g.dart`                  | `appRouterProvider`                                                                    | `AutoDisposeProvider<GoRouter>`                                                                |
| `settings_providers.g.dart`          | `sharedPreferencesProvider`, `localeControllerProvider`, `themeModeControllerProvider` | keepAlive `Provider` + `NotifierProvider` ×2                                                   |
| `auth_repository.g.dart`             | `authRepositoryProvider`                                                               | `AutoDisposeProvider<AuthRepository>`                                                          |
| `auth_user.freezed.dart` / `.g.dart` | โครง `AuthUser`/`LoginResult` + fromJson/toJson                                        | (freezed + json_serializable)                                                                  |
| `auth_controller.g.dart`             | `authControllerProvider` ⭐                                                            | ⭐ `AsyncNotifierProvider<AuthController, AuthUser?>` (**keepAlive — ไม่มีคำว่า AutoDispose**) |
| `biometric_controller.g.dart`        | `biometricControllerProvider`                                                          | `AutoDisposeAsyncNotifierProvider<BiometricController, bool>`                                  |

**Generated จาก flutter gen-l10n** (ไม่ใช่ build_runner): `lib/l10n/app_localizations.dart` (1,171 บรรทัด — abstract class + delegate), `app_localizations_th.dart` / `app_localizations_en.dart` (ฝั่งละ 548 บรรทัด — ข้อความจริงของแต่ละภาษา)

```bash
dart run build_runner build --delete-conflicting-outputs   # riverpod + freezed + json
flutter gen-l10n                                           # i18n (หรือรันอัตโนมัติตอน flutter run)
flutter test                                               # 8 เทสต์ต้องผ่านทั้งหมด
flutter run
```

> ✅ **ผลลัพธ์ที่คาดหวัง:** เปิดแอปเห็น Splash ระหว่างกู้ session → ไม่มี token เด้งไป Login อัตโนมัติ · login ด้วย `customer/1234` → เข้า `/home` ทันทีแบบ reactive (token เก็บใน Secure Storage) · ปิดแอปเปิดใหม่ → เข้าแอปเลยไม่ต้อง login ซ้ำ (กู้ session ผ่าน GET /auth/me) · logout ที่โปรไฟล์ → กลับหน้า Login ซึ่งตอนนี้เป็นโหมด **Quick Sign-in**: สแกนนิ้ว (BiometricPrompt จริงบน Android) หรือกด PIN (ตั้งจากหน้าโปรไฟล์ก่อน) · แก้ไขโปรไฟล์/เปลี่ยนรหัสผ่านมีผลจริงผ่าน API · สลับภาษาไทย/EN และโหมดมืดได้จากโปรไฟล์ **และจำค่าไว้หลังปิดแอป** · `flutter test` เขียว 8/8

### 🔗 สรุปการเชื่อมโยงทั้งหมดของ Day 3 — "ทุกอย่างหมุนรอบ AuthController"

```
เส้น Auth:      UI (login/pin/biometric) ──▶ AuthController ──▶ AuthRepository ──▶ POST /auth/*
                                    │ เก็บ/ลบ token ▼
                                SecureStorage ◀── dioClient อ่าน token แนบทุก request
เส้น Routing:   AuthController ──▶ routerRefresh (Listenable) ──▶ GoRouter.redirect
                                                     └─ splash/login/home ตาม state
เส้น Native:    LoginPage ──▶ BiometricController ──▶ DeviceInfoService ──▶ MethodChannel
                                                     └─ Kotlin BiometricPrompt / Swift
เส้น Settings:  ProfilePage toggle ──▶ Locale/ThemeModeController ──▶ SharedPreferences
                                                     └─ MaterialApp (locale/themeMode) ทั้งแอป
เส้น Test:      override ที่รอยต่อ (repository / storage / prefs) ──▶ ทดสอบทุกชั้นโดยไม่ต่อเน็ต
```

### 🧩 ความท้าทายเสริม (ถ้าทำเสร็จก่อนเวลา)

- เพิ่ม Role-based guard: route `/admin` ที่เข้าได้เฉพาะ `role == 'admin'` (ตรวจเพิ่มใน redirect)
- ต่อ `onUnauthorized` ของ `AuthInterceptor` ให้เรียก `logout()` อัตโนมัติเมื่อเจอ 401 (token หมดอายุ → เด้ง login ทั้งแอป)
- เปลี่ยน `/policy/:id` จากส่ง `state.extra` เป็นดึงจาก provider ด้วย id (รองรับ deep link เต็มรูปแบบ)
- Hash PIN ด้วย SHA-256 + salt ก่อนเก็บใน Secure Storage (ตอนนี้เก็บ plain)
- ทำ iOS Biometric จริงด้วย `LocalAuthentication` (LAContext.evaluatePolicy) แทน mock `result(true)`
- เขียนเทสต์เพิ่ม: `logout()` เคลียร์ token จริง, `changePassword()` sync credentials ใหม่เข้า storage
- ไล่เปลี่ยนหน้าที่ยังอ้าง `AppColors` ตรง ๆ ให้ใช้ `context.palette` เพื่อให้ Dark Mode สมบูรณ์ทุกหน้า

---

## 📁 โครงสร้างไฟล์สรุปวันที่ 3 (ครบตาม branch `day3-stage3`)

เฉพาะไฟล์ที่ **เพิ่มใหม่ (🆕)** และ **แก้ไข (✏️)** จาก branch `day2`:

```
bla_policy_companion/
├── pubspec.yaml ✏️                    ← + go_router, flutter_secure_storage, crypto,
│                                        encrypt, shared_preferences, flutter_localizations,
│                                        intl, mocktail + generate: true
├── l10n.yaml 🆕                       ← ตั้งค่า flutter gen-l10n
├── android/app/
│   ├── build.gradle.kts ✏️            ← + androidx.biometric
│   └── .../MainActivity.kt ✏️         ← Platform Channel + BiometricPrompt จริง
├── ios/Runner/AppDelegate.swift ✏️    ← Platform Channel ฝั่ง iOS
├── lib/
│   ├── main.dart ✏️                   ← MaterialApp.router + i18n + ThemeMode + prefs override
│   ├── core/
│   │   ├── l10n/
│   │   │   └── l10n_ext.dart 🆕          ← context.l10n
│   │   ├── network/
│   │   │   ├── crypto_interceptor.dart 🆕 ← AES เข้า/ถอดรหัส payload
│   │   │   ├── dio_client.dart ✏️        ← token จาก Secure Storage + pinning + kDebugMode log
│   │   │   └── ssl_pinning.dart 🆕       ← Certificate Pinning (กัน MITM)
│   │   ├── platform/
│   │   │   └── device_info_service.dart 🆕 ← MethodChannel "com.bla.policy/device"
│   │   ├── router/ 🆕
│   │   │   ├── app_router.dart           ← GoRouter + Route Guard + StatefulShellRoute
│   │   │   ├── home_shell.dart           ← Bottom Nav 5 แท็บ (แทน MainShell เดิม)
│   │   │   └── router_refresh.dart       ← สะพาน Riverpod → Listenable
│   │   ├── security/
│   │   │   └── crypto_service.dart 🆕    ← AES-256 (encrypt package)
│   │   ├── settings/
│   │   │   └── settings_providers.dart 🆕 ← prefs + LocaleController + ThemeModeController
│   │   ├── storage/
│   │   │   └── secure_storage.dart 🆕    ← token + PIN + bio credentials (Keychain/Keystore)
│   │   ├── theme/
│   │   │   ├── app_palette.dart 🆕       ← ThemeExtension (โทนสว่าง/มืด)
│   │   │   └── app_theme.dart ✏️         ← AppTheme.light / AppTheme.dark
│   │   └── widgets/
│   │       ├── app_text_field.dart ✏️    ← palette-aware
│   │       └── section_label.dart ✏️     ← palette-aware
│   ├── l10n/ 🆕
│   │   ├── app_en.arb / app_th.arb       ← ข้อความ 2 ภาษา (source of truth)
│   │   └── app_localizations*.dart       ← (flutter gen-l10n สร้าง — ห้ามแก้มือ)
│   └── features/
│       ├── auth/
│       │   ├── domain/auth_user.dart 🆕          ← AuthUser + LoginResult (freezed)
│       │   ├── data/auth_repository.dart 🆕      ← 7 endpoint ของ /auth
│       │   └── presentation/
│       │       ├── controllers/
│       │       │   ├── auth_controller.dart 🆕      ← ★ single source of truth (keepAlive)
│       │       │   └── biometric_controller.dart 🆕 ← ห่อ Platform Channel
│       │       └── pages/
│       │           ├── splash_page.dart ✏️       ← view ของสถานะ loading (ไม่ navigate เอง)
│       │           ├── login_page.dart ✏️        ← ฟอร์ม + Quick Sign-in (Biometric/PIN)
│       │           ├── pin_setup_page.dart 🆕    ← ตั้ง PIN 6 หลัก
│       │           ├── pin_login_page.dart 🆕    ← เข้าระบบด้วย PIN
│       │           ├── register_page.dart ✏️     ← POST /auth/register จริง
│       │           ├── forgot_password_page.dart ✏️ ← POST /auth/forgot-password จริง
│       │           └── reset_password_page.dart ✏️  ← POST /auth/reset-password จริง
│       ├── home/.../home_page.dart ✏️            ← l10n + ปุ่มลัดเป็น context.go
│       ├── policy/
│       │   ├── domain/policy.dart ✏️             ← label(AppLocalizations)
│       │   └── presentation/pages/*.dart ✏️      ← l10n + เปิด detail ผ่าน route
│       ├── claim/
│       │   ├── domain/claim_record.dart ✏️       ← label(AppLocalizations)
│       │   └── presentation/... ✏️               ← l10n
│       ├── payment/.../payments_page.dart ✏️     ← l10n
│       └── profile/
│           ├── pages/profile_page.dart ✏️        ← ข้อมูล user จริง + toggle จริง + logout จริง
│           ├── pages/edit_profile_page.dart 🆕   ← PUT /auth/me
│           └── pages/change_password_page.dart 🆕 ← POST /auth/change-password
└── test/
    ├── widget_test.dart ✏️               ← smoke test ที่ผ่านจริง (เฉลยการบ้าน Day 1)
    ├── auth_controller_test.dart 🆕      ← เทสต์ AuthController 3 เคส
    └── policy_list_controller_test.dart 🆕 ← เทสต์ provider chain 4 เคส
```

---

## 📝 สรุปประจำวันที่ 3

| หัวข้อ                      | สิ่งที่ทำได้แล้ว                                                                |
| --------------------------- | ------------------------------------------------------------------------------- |
| Module 1 — GoRouter         | ตั้งค่า routing, ส่งพารามิเตอร์ และทำ Route Guard ด้วย redirect                 |
| Module 2 — Security         | Secure Storage + Reactive Auth + SSL Pinning + AES Encryption + Obfuscation     |
| Module 3 — Platform Channel | เรียก Native API (Kotlin/Swift) ผ่าน MethodChannel + จัดการ error               |
| Module 4 — Testing          | เขียน Unit Test ด้วย ProviderContainer + Mock dependencies                      |
| Module 5 — Performance      | ลด Jank/Memory Leak, วัดด้วย DevTools และ Production Checklist                  |
| ★ Master Workshop           | ประกอบร่าง Auth + Route Guard + Secure Storage + SSL Pinning + Biometric + Test |

### ✅ ตรวจสอบความเข้าใจหลังจบหลักสูตร

> ผู้เรียนควรทำได้:
>
> - วางโครงสร้าง Feature-first และแยก Presentation/Domain/Data ได้ (วันที่ 1)
> - จัดการ async data, caching, skeleton loading และ DI ด้วย Riverpod + Dio ได้ (วันที่ 2)
> - ทำ Route Guard ที่ขับเคลื่อนด้วย Auth State และเขียน Unit Test ได้ (วันที่ 3)
> - ทำ Mobile Security ครบชั้น: Secure Storage, SSL Pinning, AES Encryption, Obfuscation ได้
> - เชื่อม Native API ผ่าน Platform Channel และ optimize performance ด้วย DevTools ได้
> - อธิบายได้ว่าทำไม logout ครั้งเดียวจึงทำให้ทั้งแอปตอบสนองพร้อมกัน (single source of truth)

---

**💡 คำคมประจำวัน:**

> "แอประดับ Production ไม่ได้วัดกันที่ตอนทุกอย่างทำงานปกติ แต่วัดกันตอนที่ token หมดอายุ เครือข่ายล่ม และผู้ใช้กดผิด — สถาปัตยกรรมที่ดีคือสิ่งที่ทำให้ช่วงเวลาเหล่านั้นยังควบคุมได้"

---

## 🎓 บทส่งท้ายหลักสูตร

ตลอด 3 วัน ผู้เรียนได้พัฒนาแอป **BLA Policy Companion** ตั้งแต่การวางโครงสร้างเปล่า จนเป็นแอปที่มีระบบ Authentication, การดึงข้อมูลแบบ async พร้อม caching และ skeleton loading, การจัดการ error อย่างเป็นระบบ, Route Guard ที่ปลอดภัย, ความปลอดภัยระดับ Production (SSL Pinning, AES Encryption, Obfuscation), การเชื่อม Native ผ่าน Platform Channel, การ optimize performance ด้วย DevTools และชุด Unit Test ที่ครอบคลุม สิ่งสำคัญที่สุดไม่ใช่ syntax ของ Riverpod แต่คือ **วิธีคิดในการแยกความรับผิดชอบ** ที่ทำให้แอปขยายได้ ทดสอบได้ และบำรุงรักษาได้ในระยะยาว — ขอให้นำแนวทางเหล่านี้ไปต่อยอดกับโปรเจกต์จริงขององค์กรต่อไปครับ

---

_เอกสารจัดทำโดย: อาจารย์สามิตร โกยม | IT Genius Engineering Co., Ltd._
_สำหรับการอบรม: บริษัท กรุงเทพประกันชีวิต จำกัด (มหาชน)_
_หลักสูตร Advanced Flutter 2026 (Scalable Architecture & Riverpod) — วันที่ 3 จาก 3_
