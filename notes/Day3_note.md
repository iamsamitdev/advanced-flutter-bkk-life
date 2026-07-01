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

| เวลา | หัวข้อ |
|------|--------|
| 09:00–09:10 | ทบทวนวันที่ 1–2 + ภาพรวมระบบ Authentication ที่จะสร้าง |
| 09:10–10:10 | **Module 1** Advanced Routing with GoRouter & Riverpod |
| 10:10–11:20 | **Module 2** Application Security (Secure Storage + Reactive Auth + SSL Pinning + Encryption) |
| 11:20–12:00 | **Module 3** Platform Channel — Native Integration |
| 12:00–13:00 | พักกลางวัน |
| 13:00–14:00 | **Module 4** Testing in Riverpod (Unit Test + Mock) |
| 14:00–15:00 | **Module 5** Performance Optimization & Best Practices 2026 |
| 15:00–16:00 | **Master Workshop** ประกอบร่าง Auth + Route Guard + Secure Storage + SSL Pinning + Test |

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

| รูปแบบการส่งค่า | เข้าถึงด้วย | เหมาะกับ |
|------------------|-------------|----------|
| Path parameter `:id` | `state.pathParameters['id']` | ID ของทรัพยากร |
| Query parameter `?status=` | `state.uri.queryParameters['status']` | ตัวกรอง/ตัวเลือก |
| Extra object | `state.extra` | ส่งออบเจกต์ทั้งก้อน (ภายในแอป) |

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
        routes: [
          // /policies/:id ยังอยู่ใน "stack ของแท็บกรมธรรม์"
          GoRoute(
            path: ':id',
            builder: (c, s) =>
                PolicyDetailPage(policyId: s.pathParameters['id']!),
          ),
        ],
      ),
    ]),
    StatefulShellBranch(routes: [
      GoRoute(path: '/claims', builder: (c, s) => const ClaimListPage()),
    ]),
    StatefulShellBranch(routes: [
      GoRoute(path: '/payments', builder: (c, s) => const PaymentsPage()),
    ]),
    StatefulShellBranch(routes: [
      GoRoute(path: '/profile', builder: (c, s) => const ProfilePage()),
    ]),
  ],
)
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

| ข้อมูล | เก็บที่ไหน | เหตุผล |
|--------|-----------|--------|
| Token, Credentials | **Flutter Secure Storage** | เข้ารหัสด้วย Keychain/Keystore |
| ค่าตั้งค่าทั่วไป (theme, ภาษา) | `SharedPreferences` | ไม่ลับ เข้าถึงเร็ว |
| ข้อมูลชั่วคราว (cache) | Memory / ไฟล์ | หายได้ ไม่ต้องเข้ารหัส |

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

| แนวทาง | Pin อะไร | ข้อดี | ข้อเสีย |
|--------|---------|------|--------|
| Certificate Pinning | ทั้งใบ certificate (SHA-256) | ตรวจเข้มที่สุด | ต้องอัปเดตแอปทุกครั้งที่ต่ออายุ cert |
| **Public Key Pinning** | เฉพาะ public key (SPKI SHA-256) | ต่ออายุ cert ได้โดยไม่ต้องแก้แอป | ต้องคุม key rotation ให้ดี |

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

| ชนิด Channel | ใช้เมื่อ |
|--------------|---------|
| **MethodChannel** | เรียกเมธอดครั้งเดียวแล้วได้ผลกลับ (ที่ใช้บ่อยที่สุด) |
| EventChannel | สตรีมข้อมูลต่อเนื่องจาก Native (เช่น เซ็นเซอร์, ตำแหน่ง) |
| BasicMessageChannel | ส่งข้อความสองทางแบบ custom codec |

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

| ฝั่ง Native (ตอบ) | ฝั่ง Dart (รับ) |
|-------------------|------------------|
| `result.success(value)` | ได้ค่าปกติใน `await invokeMethod` |
| `result.error(code, msg, details)` | โยน `PlatformException` (มี `code`, `message`) |
| `result.notImplemented()` | โยน `MissingPluginException` |

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

| ขั้นตอน | เครื่องมือ | ทำอะไร |
|---------|-----------|--------|
| สร้าง Mock | `class MockX extends Mock implements X` | สร้างของปลอมที่ควบคุมพฤติกรรมได้ |
| กำหนดพฤติกรรม | `when(() => mock.method()).thenAnswer(...)` | บอกว่าเมื่อถูกเรียกให้คืนอะไร |
| ฉีดเข้าระบบ | `provider.overrideWithValue(mock)` | แทน dependency จริงด้วย mock |
| ตรวจการเรียก | `verify(() => mock.method()).called(1)` | ยืนยันว่าถูกเรียกตามที่คาด |

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

| เทคนิค | ทำอะไร | ผลลัพธ์ |
|--------|--------|---------|
| `const` Widget | บอก Flutter ว่า widget นี้ไม่เปลี่ยน | ข้ามการ rebuild/repaint ทั้ง subtree |
| `RepaintBoundary` | แยกชั้นการวาดของ widget ที่ animate บ่อย | repaint เฉพาะส่วนนั้น ไม่ลามทั้งจอ |
| `ListView.builder` | สร้างเฉพาะ item ที่มองเห็น | ลด build/memory ของ list ยาว |
| เลี่ยงงานหนักใน `build()` | ย้ายการคำนวณ/จัดเรียงออกไป provider | `build()` เบา เฟรมไม่หล่น |

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

| แหล่ง Leak ที่พบบ่อย | วิธีป้องกัน |
|----------------------|------------|
| `TextEditingController`, `ScrollController`, `AnimationController` | `dispose()` ใน `dispose()` ของ State |
| `Timer`, `StreamSubscription` ใน provider | `ref.onDispose(() => ...)` |
| `ref.keepAlive()` ที่ไม่มีวันปิด | ตั้ง timer ปิด `link.close()` (ดูวันที่ 2) |
| Listener ที่ `addListener` | `removeListener` ตอน dispose |

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

| หมวด | สิ่งที่ต้องทำ |
|------|---------------|
| **Code Obfuscation** | `flutter build --obfuscate --split-debug-info=...` กัน reverse-engineer (ดู Module 2.4) |
| **Logging Strategy** | ปิด `print`/verbose log ใน release, ใช้ logger ที่ปรับระดับตาม flavor ได้ |
| **Error Monitoring** | ต่อ Crash Reporting (เช่น Crashlytics/Sentry) ดักผ่าน `FlutterError.onError` + `runZonedGuarded` |
| **Build Flavor** | แยก Dev / Staging / Prod ให้คนละ bundle id, ไอคอน, ชื่อแอป |
| **Environment Config** | ฉีดค่า (baseUrl, key) ด้วย `--dart-define` ไม่ hardcode ลงโค้ด |
| **เวอร์ชัน & ขนาด** | ตั้ง version/build number, ตรวจขนาด APK/IPA และทำ `--split-per-abi` ถ้าจำเป็น |

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

| แนวปฏิบัติ | เหตุผล |
|-----------|--------|
| ใช้ `ref.watch().select()` เมื่อสนใจเฉพาะบางฟิลด์ | ลด rebuild ที่ไม่จำเป็น |
| ใช้ `ref.read()` ใน callback เสมอ ห้าม `watch` | กัน subscribe ซ้ำซ้อน |
| แยก Widget ย่อยให้ watch provider ที่จำเป็นเท่านั้น | จำกัดขอบเขตการ rebuild |
| ใช้ `const` constructor ทุกที่ที่ทำได้ | Flutter ข้ามการ rebuild widget ที่ไม่เปลี่ยน |
| ใช้ Code Generation (`@riverpod`) | compile-safe, ลด bug จากการพิมพ์ชนิดผิด |
| เคลียร์ resource ใน `ref.onDispose()` เสมอ | กัน memory leak |
| ใช้ `@Riverpod(keepAlive: true)` เฉพาะ state ที่ต้องอยู่ตลอด | ประหยัดหน่วยความจำ |

> ⚠️ **กับดักที่พบบ่อย:** การ `ref.watch` provider ที่เป็น `AsyncValue` ทั้งก้อนในหน้าใหญ่ ทำให้ทุก loading/refresh สั่น UI ทั้งหน้า — ให้ย้ายการ watch ลงไปใน widget ย่อยที่เล็กที่สุดที่ต้องใช้ค่านั้นจริง ๆ

---

## 🛠️ Master Workshop — BLA Policy Companion: ประกอบร่างระบบ Authentication ครบวงจร

### เวลา 15:00–16:00 น.

> **โจทย์:** สร้างระบบ Login/Logout ที่เก็บ Token ด้วย Secure Storage, ควบคุม Route Guard ด้วย Auth State จาก Riverpod, แนบ Token เข้า Dio อัตโนมัติ **พร้อมเปิด SSL Pinning**, เสริม **Biometric ผ่าน Platform Channel**, แสดงผลแบบ Reactive และเขียน Unit Test ของ AuthController — เชื่อมทุกอย่างจากวันที่ 1–3 เข้าด้วยกัน

### ขั้นที่ 1 — โมเดลและ Repository ของ Auth

```dart
// features/auth/domain/auth_user.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'auth_user.freezed.dart';
part 'auth_user.g.dart';

@freezed
class AuthUser with _$AuthUser {
  const AuthUser._();

  const factory AuthUser({
    required String id,
    required String displayName,
    required String role,
  }) = _AuthUser;

  // ป้ายบทบาทภาษาไทย (แอปนี้เป็นฝั่งลูกค้า)
  String get roleLabel => role == 'customer' ? 'ลูกค้า' : role;

  factory AuthUser.fromJson(Map<String, dynamic> json) =>
      _$AuthUserFromJson(json);
}

@freezed
class LoginResult with _$LoginResult {
  const factory LoginResult({
    required AuthUser user,
    required String token,
  }) = _LoginResult;
}
```

```dart
// features/auth/data/auth_repository.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../domain/auth_user.dart';

part 'auth_repository.g.dart';

class AuthRepository {
  // จำลองการ login (ในงานจริงเรียก API ผ่าน Dio)
  Future<LoginResult> login(String username, String password) async {
    await Future<void>.delayed(const Duration(milliseconds: 800));
    if (username == 'customer' && password == '1234') {
      return const LoginResult(
        user: AuthUser(id: 'C001', displayName: 'คุณสมชาย ใจดี', role: 'customer'),
        token: 'mock-token-abc123',
      );
    }
    throw Exception('ชื่อผู้ใช้หรือรหัสผ่านไม่ถูกต้อง');
  }

  Future<AuthUser?> getCurrentUser(String token) async {
    // จำลองการตรวจ token แล้วคืนข้อมูลผู้ใช้
    await Future<void>.delayed(const Duration(milliseconds: 300));
    return const AuthUser(id: 'C001', displayName: 'คุณสมชาย ใจดี', role: 'customer');
  }
}

@riverpod
AuthRepository authRepository(AuthRepositoryRef ref) => AuthRepository();
```

### ขั้นที่ 2 — AuthController (ใช้โค้ดจาก Module 2.2)

ใช้ `AuthController` แบบ `@Riverpod(keepAlive: true)` ตามที่เขียนไว้ใน Module 2.2 — เป็นแหล่งความจริงเดียวของสถานะผู้ใช้

### ขั้นที่ 3 — ตัวเชื่อม Auth State เข้ากับ GoRouter (refreshListenable)

```dart
// core/router/router_refresh.dart
import 'package:flutter/foundation.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../../features/auth/presentation/controllers/auth_controller.dart';

part 'router_refresh.g.dart';

// แปลงการเปลี่ยนแปลงของ authController ให้เป็น Listenable ที่ GoRouter ฟังได้
@riverpod
Listenable routerRefresh(RouterRefreshRef ref) {
  final notifier = ValueNotifier<int>(0);
  ref.onDispose(notifier.dispose);
  // ทุกครั้งที่ auth state เปลี่ยน → กระตุ้น notifier ให้ router คำนวณ redirect ใหม่
  ref.listen(authControllerProvider, (_, __) => notifier.value++);
  return notifier;
}
```

### ขั้นที่ 4 — หน้า Login

```dart
// features/auth/presentation/pages/login_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../controllers/auth_controller.dart';

class LoginPage extends ConsumerStatefulWidget {
  const LoginPage({super.key});

  @override
  ConsumerState<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends ConsumerState<LoginPage> {
  final _userController = TextEditingController();
  final _passController = TextEditingController();

  @override
  void dispose() {
    _userController.dispose();
    _passController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final authState = ref.watch(authControllerProvider);

    // แสดง error เมื่อ login ล้มเหลว
    ref.listen(authControllerProvider, (previous, next) {
      if (next.hasError && !next.isLoading) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('${next.error}')),
        );
      }
    });

    return Scaffold(
      appBar: AppBar(title: const Text('เข้าสู่ระบบ BLA')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            TextField(
              controller: _userController,
              decoration: const InputDecoration(labelText: 'ชื่อผู้ใช้ (customer)'),
            ),
            TextField(
              controller: _passController,
              obscureText: true,
              decoration: const InputDecoration(labelText: 'รหัสผ่าน (1234)'),
            ),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: authState.isLoading
                  ? null
                  : () => ref.read(authControllerProvider.notifier).login(
                        _userController.text,
                        _passController.text,
                      ),
              child: authState.isLoading
                  ? const CircularProgressIndicator()
                  : const Text('เข้าสู่ระบบ'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### ขั้นที่ 5 — แนบ Token เข้า Dio + เปิด SSL Pinning (เชื่อมกับวันที่ 2 + Module 2.3)

```dart
// core/network/dio_client.dart (เวอร์ชันสมบูรณ์ + ความปลอดภัย)
import 'package:dio/dio.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../storage/secure_storage.dart';
import 'auth_interceptor.dart';
import 'ssl_pinning.dart'; // จาก Module 2.3

part 'dio_client.g.dart';

@riverpod
Dio dioClient(DioClientRef ref) {
  final dio = Dio(BaseOptions(
    baseUrl: 'https://api.bla-demo.example.com/v1',
    connectTimeout: const Duration(seconds: 10),
  ));

  final storage = ref.watch(secureStorageProvider);
  // 1) ฉีด interceptor ที่ดึง token จาก Secure Storage มาแนบทุก request
  dio.interceptors.add(AuthInterceptor(() => storage.readToken()));

  // 2) เปิด SSL Pinning — ปฏิเสธการเชื่อมต่อหาก certificate ไม่ตรง pin
  applySslPinning(dio);

  return dio;
}
```

### ขั้นที่ 6 — เสริม Biometric ก่อน Login ด้วย Platform Channel (Module 3)

เพิ่มความปลอดภัยอีกชั้น: ก่อนยอมให้กรอกรหัสผ่าน ให้ยืนยันตัวตนด้วย Biometric ที่เรียกผ่าน Native — ห่อด้วย `FutureProvider` เพื่อให้ UI ใช้ `AsyncValue` ได้ตามปกติ

```dart
// features/auth/presentation/controllers/biometric_controller.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../../../../core/platform/device_info_service.dart';

part 'biometric_controller.g.dart';

@riverpod
class BiometricController extends _$BiometricController {
  @override
  FutureOr<bool> build() => false; // ยังไม่ผ่านการยืนยัน

  Future<void> authenticate() async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(
      () => ref.read(deviceInfoServiceProvider).authenticateBiometric(),
    );
  }
}
```

```dart
// ในหน้า Login: ปุ่ม "ยืนยันด้วยลายนิ้วมือ/ใบหน้า"
final bioState = ref.watch(biometricControllerProvider);
ElevatedButton.icon(
  icon: const Icon(Icons.fingerprint),
  label: const Text('ยืนยันด้วย Biometric'),
  onPressed: bioState.isLoading
      ? null
      : () => ref.read(biometricControllerProvider.notifier).authenticate(),
);
```

> ✅ **จุดสำคัญ:** การเรียก Native (Platform Channel) กลมกลืนกับ Riverpod อย่างสมบูรณ์ — มันก็เป็นแค่ `Future` ที่ห่อด้วย `AsyncValue` เหมือนการเรียก API ทุกประการ UI จึงไม่ต้องรู้เลยว่าเบื้องหลังเป็น Kotlin/Swift

### ขั้นที่ 7 — เชื่อม Router เข้าแอป

```dart
// main.dart (เวอร์ชันสมบูรณ์)
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'core/router/app_router.dart';

void main() {
  runApp(const ProviderScope(child: BlaApp()));
}

class BlaApp extends ConsumerWidget {
  const BlaApp({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final router = ref.watch(appRouterProvider);
    return MaterialApp.router(
      title: 'BLA Policy Companion',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.indigo),
      routerConfig: router, // ใช้ GoRouter ที่มี Route Guard
    );
  }
}
```

### ขั้นที่ 8 — เขียน Unit Test ของ AuthController

```dart
// test/auth_controller_test.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

class MockAuthRepository extends Mock implements AuthRepository {}
class MockSecureStorage extends Mock implements SecureStorageService {}

void main() {
  test('login สำเร็จ → เก็บ token และ state เป็น AuthUser', () async {
    final mockRepo = MockAuthRepository();
    final mockStorage = MockSecureStorage();

    when(() => mockRepo.login('customer', '1234')).thenAnswer(
      (_) async => const LoginResult(
        user: AuthUser(id: 'C001', displayName: 'คุณสมชาย ใจดี', role: 'customer'),
        token: 'mock-token-abc123',
      ),
    );
    when(() => mockStorage.readToken()).thenAnswer((_) async => null);
    when(() => mockStorage.saveToken(any())).thenAnswer((_) async {});

    final container = ProviderContainer(
      overrides: [
        authRepositoryProvider.overrideWithValue(mockRepo),
        secureStorageProvider.overrideWithValue(mockStorage),
      ],
    );
    addTearDown(container.dispose);

    // รอ build() แรกเสร็จ (อ่าน token = null → user = null)
    await container.read(authControllerProvider.future);

    // สั่ง login
    await container.read(authControllerProvider.notifier).login('customer', '1234');

    final user = container.read(authControllerProvider).valueOrNull;
    expect(user?.displayName, 'คุณสมชาย ใจดี');
    // ยืนยันว่ามีการเก็บ token ลง secure storage
    verify(() => mockStorage.saveToken('mock-token-abc123')).called(1);
  });
}
```

รันเทสต์ทั้งหมด:

```bash
dart run build_runner build --delete-conflicting-outputs
flutter test
```

> ✅ **ผลลัพธ์ที่คาดหวัง:** เปิดแอปครั้งแรกไม่มี token → ถูก redirect ไปหน้า Login อัตโนมัติ, login ด้วย customer/1234 สำเร็จ → เด้งเข้าหน้าแรก (Home) ทันที (reactive), token ถูกเก็บใน Secure Storage และแนบเข้าทุก request ของ Dio อัตโนมัติ **โดยมี SSL Pinning ตรวจ certificate ทุกครั้ง**, ปุ่ม Biometric เรียก Native ผ่าน Platform Channel ได้, กด Logout → ทั้งแอป redirect กลับ Login พร้อมกัน และ `flutter test` ผ่านทุกเคส

### 🧩 ความท้าทายเสริม (ถ้าทำเสร็จก่อนเวลา)

- เพิ่ม Role-based guard: หน้าบางหน้าเข้าได้เฉพาะ role ที่กำหนด (ตรวจใน redirect)
- จัดการ token หมดอายุ (401) ใน `AuthInterceptor` ให้สั่ง logout อัตโนมัติ
- เขียนเทสต์เพิ่มสำหรับ `logout()` ว่าเคลียร์ token และ state เป็น null จริง
- เพิ่มหน้า Splash ที่แสดงระหว่าง `build()` ของ AuthController กำลังตรวจ token
- เสียบ `CryptoInterceptor` (Module 2.4) เข้ารหัส payload และทดสอบ encrypt/decrypt ไป-กลับ
- เพิ่ม `getDeviceId` ฝั่ง Native (Kotlin/Swift) แล้วแสดง Device ID ในแท็บโปรไฟล์ (Module 3)
- วัดเฟรมด้วย `flutter run --profile` + Performance Overlay หาจุด jank ในหน้ารายการ (Module 5.1)

---

## 📁 โครงสร้างไฟล์สรุปวันที่ 3

```
bla_policy_companion/lib/
├── main.dart                         ← MaterialApp.router + GoRouter (+ runZonedGuarded)
├── core/
│   ├── router/
│   │   ├── app_router.dart           ← StatefulShellRoute 5 แท็บ + Route Guard
│   │   ├── home_shell.dart           ← Scaffold + NavigationBar 5 แท็บ
│   │   └── router_refresh.dart       ← ตัวเชื่อม Riverpod → Listenable
│   ├── storage/
│   │   └── secure_storage.dart       ← เก็บ token เข้ารหัส
│   ├── security/
│   │   └── crypto_service.dart       ← AES encrypt/decrypt payload (Module 2.4)
│   ├── platform/
│   │   └── device_info_service.dart  ← MethodChannel เรียก Native (Module 3)
│   └── network/
│       ├── dio_client.dart           ← แนบ token + SSL Pinning
│       ├── auth_interceptor.dart
│       ├── ssl_pinning.dart          ← Public Key Pinning (Module 2.3)
│       └── crypto_interceptor.dart   ← เข้ารหัส/ถอดรหัสอัตโนมัติ (Module 2.4)
├── android/.../MainActivity.kt       ← ฝั่ง Native Android (Kotlin)
├── ios/Runner/AppDelegate.swift      ← ฝั่ง Native iOS (Swift)
└── features/
    ├── home/      presentation/pages/home_page.dart      ← แท็บ "หน้าแรก" (Dashboard)
    ├── policy/    ...                                    ← แท็บ "กรมธรรม์" (วันที่ 1–2)
    ├── claim/     presentation/pages/claim_list_page.dart ← แท็บ "สินไหม"
    ├── payment/   presentation/pages/payments_page.dart  ← แท็บ "ชำระเบี้ย"
    ├── profile/   presentation/pages/profile_page.dart   ← แท็บ "โปรไฟล์" (+ ปุ่มออกจากระบบ)
    └── auth/
        ├── domain/auth_user.dart     ← freezed (+ roleLabel: 'ลูกค้า')
        ├── data/auth_repository.dart  ← login / getCurrentUser (persona ลูกค้า)
        └── presentation/
            ├── controllers/auth_controller.dart       ← Reactive Auth State
            ├── controllers/biometric_controller.dart  ← Biometric ผ่าน Platform Channel
            └── pages/login_page.dart  ← บัญชีทดสอบ: customer / 1234
test/
├── policy_list_controller_test.dart  ← เทสต์จาก Module 4
└── auth_controller_test.dart         ← เทสต์ Master Workshop
```

---

## 📝 สรุปประจำวันที่ 3

| หัวข้อ | สิ่งที่ทำได้แล้ว |
|--------|----------------|
| Module 1 — GoRouter | ตั้งค่า routing, ส่งพารามิเตอร์ และทำ Route Guard ด้วย redirect |
| Module 2 — Security | Secure Storage + Reactive Auth + SSL Pinning + AES Encryption + Obfuscation |
| Module 3 — Platform Channel | เรียก Native API (Kotlin/Swift) ผ่าน MethodChannel + จัดการ error |
| Module 4 — Testing | เขียน Unit Test ด้วย ProviderContainer + Mock dependencies |
| Module 5 — Performance | ลด Jank/Memory Leak, วัดด้วย DevTools และ Production Checklist |
| ★ Master Workshop | ประกอบร่าง Auth + Route Guard + Secure Storage + SSL Pinning + Biometric + Test |

### ✅ ตรวจสอบความเข้าใจหลังจบหลักสูตร

> ผู้เรียนควรทำได้:
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

*เอกสารจัดทำโดย: อาจารย์สามิตร โกยม | IT Genius Engineering Co., Ltd.*
*สำหรับการอบรม: บริษัท กรุงเทพประกันชีวิต จำกัด (มหาชน)*
*หลักสูตร Advanced Flutter 2026 (Scalable Architecture & Riverpod) — วันที่ 3 จาก 3*
