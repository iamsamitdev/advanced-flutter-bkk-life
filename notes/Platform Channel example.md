# Platform Channel — ตัวอย่างจริง 2 เคส

> เอกสารประกอบการสอน · Advanced Flutter 2026 · ต่อยอดจาก `device_info_service.dart` (Day 3 Module 3)
> ครบทั้งฝั่ง **Dart + Android (Kotlin) + iOS (Swift)** — คัดลอกไปใช้/สาธิตได้ทันที

Platform Channel มี 3 แบบ ในเอกสารนี้ใช้ 2 แบบ:

| แบบ | เหมาะกับ | ตัวอย่างในเอกสาร |
|-----|---------|------------------|
| **MethodChannel** | เรียกครั้งเดียวได้ผลกลับ (คำสั่ง) | เคส 2: เปิด/ปิดกันแคปหน้าจอ |
| **EventChannel** | สตรีมข้อมูลต่อเนื่อง (เหตุการณ์) | เคส 1: อ่านระดับแบตเตอรี่แบบเรียลไทม์ |

> ทั้งสองเคสลงทะเบียน channel ในไฟล์เดิม (Android: `MainActivity.configureFlutterEngine`,
> iOS: `AppDelegate.didInitializeImplicitFlutterEngine`) — ลงทะเบียนได้หลาย channel ในที่เดียว

---

## เอาไปประยุกต์ทำอะไรได้บ้าง (6 กลุ่มการใช้งานจริง)

Platform Channel คือ "สะพาน" ไปเรียกความสามารถของ **Native** ที่โลก Dart/Flutter เข้าถึงเองไม่ได้
ในแอปจริง (โดยเฉพาะแอปประกัน/การเงิน) ประยุกต์ได้กว้างมาก:

**1) ความปลอดภัย / ยืนยันตัวตน** — Biometric ปลดล็อกแอป/อนุมัติรายการ (ที่เราทำใน `device_info_service.dart`),
สร้าง/เก็บ key ใน Keystore/Secure Enclave (ฮาร์ดแวร์), ตรวจ root/jailbreak ก่อนแสดงข้อมูลอ่อนไหว,
**กันแคปหน้าจอ** ในหน้าเลขกรมธรรม์/บัตร/OTP (ดูเคส 2 ด้านล่าง)

**2) เชื่อม Native SDK ของพาร์ตเนอร์** (ธุรกิจประกัน/ธนาคารเจอบ่อยสุด) — **SDK ชำระเงิน** ของธนาคาร/พร้อมเพย์/บัตร
(Omise, 2C2P), **SDK eKYC** (ตรวจใบหน้า liveness ยืนยันตัวตน), SDK วิเคราะห์/แจ้งเตือนเฉพาะเจ้า
ที่มีให้เฉพาะฝั่ง Native

**3) ฮาร์ดแวร์ / เซนเซอร์** — **NFC อ่านชิปบัตรประชาชน/พาสปอร์ต** (ทำ eKYC เปิดกรมธรรม์),
Bluetooth ต่ออุปกรณ์วัดสุขภาพ, GPS/geofencing เบื้องหลัง, **HealthKit/Google Fit** ดึงก้าวเดิน/ชีพจร
→ ประกันสุขภาพให้ส่วนลดเบี้ยตามพฤติกรรม, ระดับแบตเตอรี่ (ดูเคส 1 ด้านล่าง)

**4) ข้อมูล/ความสามารถของเครื่อง** (แบบ `getDeviceId` ในตัวอย่าง) — Device ID, รุ่น, เวอร์ชัน OS,
แบตเตอรี่, ชนิดเครือข่าย, ข้อมูลซิม/ค่ายมือถือ → ทำ device binding (ผูกบัญชีกับเครื่อง), analytics, ตรวจ fraud

**5) ผสานกับระบบปฏิบัติการ** — บันทึกไฟล์ PDF กรมธรรม์ลง Files/รูปภาพ, แชร์ผ่าน share sheet,
เพิ่มวันครบกำหนดชำระเข้าปฏิทิน, Home Screen Widget แสดงเบี้ยครบกำหนด, Siri Shortcuts / App Actions,
background task ซิงก์ข้อมูล

**6) ใช้โค้ด Native เดิมที่มีอยู่** — บริษัทมี library Kotlin/Swift/Java เดิม หรือ logic ที่ต้องรันเร็ว
(ประมวลผลภาพ, ML ผ่าน CoreML/ML Kit) — เรียกกลับมาใช้ได้โดยไม่ต้องเขียนใหม่

> 💡 **เมื่อไรควรเขียน channel เอง:** ถ้ามี plugin บน pub.dev อยู่แล้ว (เช่น `local_auth`, `nfc_manager`,
> `url_launcher`, `image_picker`, `health`, `battery_plus`) ให้ใช้ plugin เลย (ข้างในมันก็ใช้ platform channel นี่แหละ)
> — เขียน channel เองเฉพาะตอน **ไม่มี plugin** หรือต้องต่อ **SDK เฉพาะของพาร์ตเนอร์**

---

## เคส 1 — อ่านระดับแบตเตอรี่แบบเรียลไทม์ (EventChannel)

ใช้ **EventChannel** เพราะแบตเตอรี่เป็น "เหตุการณ์ที่เปลี่ยนเรื่อย ๆ" (stream) ไม่ใช่ถามครั้งเดียว

### 1.1 ฝั่ง Dart

```dart
// lib/core/platform/battery_service.dart
import 'package:flutter/services.dart';

/// อ่านระดับแบตเตอรี่ (%) แบบสตรีมผ่าน EventChannel
class BatteryService {
  static const _events = EventChannel('com.bla.policy/battery');

  /// สตรีมระดับแบตเตอรี่ (0–100) — ส่งค่าใหม่ทุกครั้งที่ระดับเปลี่ยน
  Stream<int> get level =>
      _events.receiveBroadcastStream().map((e) => e as int);
}
```

ใช้งานด้วย `StreamBuilder` (หรือห่อด้วย `StreamProvider` ของ Riverpod ก็ได้):

```dart
StreamBuilder<int>(
  stream: BatteryService().level,
  builder: (context, snapshot) {
    final level = snapshot.data;
    if (level == null) return const Text('กำลังอ่านแบตเตอรี่...');
    return Text('แบตเตอรี่ $level%');
  },
)
```

> เวอร์ชัน Riverpod:
> ```dart
> final batteryLevelProvider = StreamProvider<int>((ref) => BatteryService().level);
> // ใน widget: ref.watch(batteryLevelProvider).when(...)
> ```

### 1.2 ฝั่ง Android (Kotlin) — ใน `MainActivity.configureFlutterEngine`

```kotlin
import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.content.IntentFilter
import android.os.BatteryManager
import io.flutter.plugin.common.EventChannel

EventChannel(flutterEngine.dartExecutor.binaryMessenger, "com.bla.policy/battery")
    .setStreamHandler(object : EventChannel.StreamHandler {
        private var receiver: BroadcastReceiver? = null

        // Dart เริ่ม listen → เริ่มฟังการเปลี่ยนแปลงของแบตเตอรี่
        override fun onListen(arguments: Any?, sink: EventChannel.EventSink) {
            receiver = object : BroadcastReceiver() {
                override fun onReceive(ctx: Context, intent: Intent) {
                    val level = intent.getIntExtra(BatteryManager.EXTRA_LEVEL, -1)
                    val scale = intent.getIntExtra(BatteryManager.EXTRA_SCALE, -1)
                    if (level >= 0 && scale > 0) {
                        sink.success(level * 100 / scale) // ส่ง % กลับไป Dart
                    }
                }
            }
            registerReceiver(receiver, IntentFilter(Intent.ACTION_BATTERY_CHANGED))
        }

        // Dart ยกเลิก listen (เช่น widget ถูก dispose) → เลิกฟัง กัน memory leak
        override fun onCancel(arguments: Any?) {
            receiver?.let { unregisterReceiver(it) }
            receiver = null
        }
    })
```

> ✅ **จุดสำคัญ:** `onListen`/`onCancel` คือหัวใจของ EventChannel — ลงทะเบียน resource ใน onListen
> และ **คืน resource ใน onCancel เสมอ** (unregister receiver) มิฉะนั้นจะรั่ว

### 1.3 ฝั่ง iOS (Swift)

```swift
// สร้าง StreamHandler แยกไว้ (เพิ่มในไฟล์ AppDelegate.swift หรือไฟล์ใหม่)
class BatteryStreamHandler: NSObject, FlutterStreamHandler {
  private var sink: FlutterEventSink?

  func onListen(withArguments arguments: Any?,
                eventSink events: @escaping FlutterEventSink) -> FlutterError? {
    sink = events
    UIDevice.current.isBatteryMonitoringEnabled = true
    NotificationCenter.default.addObserver(
      self, selector: #selector(sendLevel),
      name: UIDevice.batteryLevelDidChangeNotification, object: nil)
    sendLevel() // ส่งค่าแรกทันที
    return nil
  }

  func onCancel(withArguments arguments: Any?) -> FlutterError? {
    NotificationCenter.default.removeObserver(self)
    sink = nil
    return nil
  }

  @objc private func sendLevel() {
    let level = UIDevice.current.batteryLevel // 0.0–1.0 (-1 ถ้าอ่านไม่ได้ เช่นบน simulator)
    if level >= 0 { sink?(Int(level * 100)) }
  }
}

// ลงทะเบียนใน didInitializeImplicitFlutterEngine (มี messenger อยู่แล้วจากตัวอย่าง biometric)
let batteryChannel = FlutterEventChannel(
  name: "com.bla.policy/battery", binaryMessenger: messenger)
batteryChannel.setStreamHandler(BatteryStreamHandler())
```

> หมายเหตุ: บน iOS Simulator `batteryLevel` มักคืน -1 (ต้องทดสอบบนเครื่องจริง)

---

## เคส 2 — ป้องกันการแคปหน้าจอ (MethodChannel + FLAG_SECURE)

ใช้ **MethodChannel** เพราะเป็น "คำสั่ง" เปิด/ปิด — เหมาะมากกับแอปประกัน/ธนาคาร:
เปิดกันแคปเฉพาะหน้าที่อ่อนไหว (เลขกรมธรรม์, บัตร, OTP)

### 2.1 ฝั่ง Dart

```dart
// lib/core/platform/secure_screen.dart
import 'package:flutter/services.dart';

/// สั่งเปิด/ปิดการกันแคปหน้าจอ (Android: FLAG_SECURE)
class SecureScreen {
  static const _channel = MethodChannel('com.bla.policy/secure');

  Future<void> enable() => _channel.invokeMethod('enableSecure');
  Future<void> disable() => _channel.invokeMethod('disableSecure');
}
```

ใช้กับหน้าที่อ่อนไหว — เปิดตอนเข้า ปิดตอนออก (ผูกกับ lifecycle ของ widget):

```dart
class PolicyDetailPage extends StatefulWidget { /* ... */ }

class _PolicyDetailPageState extends State<PolicyDetailPage> {
  final _secure = SecureScreen();

  @override
  void initState() {
    super.initState();
    _secure.enable();   // เข้าหน้านี้ → กันแคป
  }

  @override
  void dispose() {
    _secure.disable();  // ออกจากหน้า → ปลดล็อก
    super.dispose();
  }
  // ... build ...
}
```

### 2.2 ฝั่ง Android (Kotlin) — ใน `MainActivity.configureFlutterEngine`

```kotlin
import android.view.WindowManager
import io.flutter.plugin.common.MethodChannel

MethodChannel(flutterEngine.dartExecutor.binaryMessenger, "com.bla.policy/secure")
    .setMethodCallHandler { call, result ->
        when (call.method) {
            "enableSecure" -> {
                // FLAG_SECURE: กันแคปหน้าจอ + ไม่แสดง preview ใน recent apps
                runOnUiThread {
                    window.addFlags(WindowManager.LayoutParams.FLAG_SECURE)
                }
                result.success(null)
            }
            "disableSecure" -> {
                runOnUiThread {
                    window.clearFlags(WindowManager.LayoutParams.FLAG_SECURE)
                }
                result.success(null)
            }
            else -> result.notImplemented()
        }
    }
```

> ✅ ต้องเรียก `window.addFlags(...)` บน **UI thread** เสมอ (ห่อด้วย `runOnUiThread`)
> · ไม่ต้องขอ permission ใด ๆ · เมื่อเปิด FLAG_SECURE ระบบจะบล็อกทั้งการแคปและการอัดจอ

### 2.3 ฝั่ง iOS (Swift) — ข้อจำกัดสำคัญ

> ⚠️ **iOS ไม่มี FLAG_SECURE ตรง ๆ** — บล็อกการแคปหน้าจอแบบเด็ดขาดไม่ได้ (ยกเว้นเทคนิค
> secure `UITextField` ที่ซับซ้อน) แนวทางที่ทำได้จริงคือ **ตรวจจับ** แล้วตอบสนอง:

```swift
// (ก) ตรวจจับเมื่อผู้ใช้แคปหน้าจอ → แจ้งเตือน/ส่ง log ความเสี่ยง
NotificationCenter.default.addObserver(
  forName: UIApplication.userDidTakeScreenshotNotification,
  object: nil, queue: .main) { _ in
    // เช่น: บันทึกเหตุการณ์, แจ้งเตือนผู้ใช้, หรือส่งขึ้น backend
    print("⚠️ ผู้ใช้แคปหน้าจอในหน้าที่อ่อนไหว")
}

// (ข) เบลอหน้าจอตอนสลับแอป (กันเห็นข้อมูลใน App Switcher)
//     ทำใน SceneDelegate.sceneWillResignActive: ครอบด้วย UIVisualEffectView (blur)
//     แล้วเอาออกใน sceneDidBecomeActive
```

ในทางปฏิบัติ MethodChannel ฝั่ง iOS มักตอบ `result(nil)` สำหรับ enable/disable (no-op)
แล้วใช้กลไกตรวจจับ/เบลอข้างบนแทน — ให้ฝั่ง Dart เขียนโค้ดชุดเดียว ทำงานได้ทั้งสองแพลตฟอร์ม

---

## สรุปแนวคิดที่ได้จาก 2 เคสนี้

- **เลือกชนิด channel ตามลักษณะข้อมูล:** คำสั่งครั้งเดียว → `MethodChannel` · สตรีมต่อเนื่อง → `EventChannel`
- **EventChannel ต้องจัดการ lifecycle** (`onListen`/`onCancel`) เพื่อลงทะเบียน/คืน resource กันรั่ว
- **งาน UI ฝั่ง Native ต้องอยู่บน UI thread** (Android: `runOnUiThread`)
- **แต่ละแพลตฟอร์มความสามารถไม่เท่ากัน** (เช่น FLAG_SECURE มีแค่ Android) — ออกแบบ Dart API
  ให้เป็นกลาง แล้วให้แต่ละฝั่ง Native ทำเท่าที่ทำได้
- **ก่อนเขียน channel เอง เช็ค pub.dev ก่อน** — เช่น `battery_plus` (แบตเตอรี่),
  `screen_protector` / `flutter_windowmanager` (กันแคป) มีให้ใช้แล้ว เขียนเองเมื่อไม่มี plugin
  หรือต้องต่อ SDK เฉพาะของพาร์ตเนอร์
