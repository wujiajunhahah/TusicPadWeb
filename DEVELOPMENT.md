## TusicPad 开发指南（模块与接口）

本工程当前为 Web 外壳（Apple Pencil → Strudel）。目标原生 MVP（iPadOS 17+/18，Swift/SwiftUI）模块化结构如下，便于后续迁移与对齐：

### 目录规划
- `/Core`
  - `MusicEventBus`
  - `AudioEngine`（AudioKit/AVAudioEngine）
  - `HapticsEngine`（Core Haptics）
  - `BLEClient`（CoreBluetooth）
  - `SyncClock`（统一时间源）
  - `SessionLogger`（JSONL）
- `/UI`
  - `MainView`（鼓垫界面，波纹动画）
  - `SettingsView`（触觉强度档位、休息提醒）
- `/Resources`
  - 音色/图样（采样或合成预设）

### 统一事件总线：MusicEventBus
职责：统一输入（触摸/笔/外设）→ 音频/触觉/外设 的分发与时间戳对齐。

事件：
- `PadHit { id, x, y, pressure, velocity, atTime }`
- `PadLift { id, atTime }`
- `MetronomeTick { bar, beat, atTime }`
- `BLEStatus { connected, rssi, atTime }`

API（Swift 伪码）：
```swift
protocol MusicEvent {}
struct PadHit: MusicEvent { let id: Int; let x: Float; let y: Float; let pressure: Float; let velocity: Float; let atTime: TimeInterval }

final class MusicEventBus {
    static let shared = MusicEventBus()
    func post(_ event: MusicEvent)
    func subscribe<T: MusicEvent>(_ type: T.Type, _ handler: @escaping (T)->Void) -> AnyCancellable
}
```

参考：`Combine`/`AsyncSequence` 事件流；建议把 `atTime` 基于统一 `SyncClock`（见下）。

### 音频引擎：AudioEngine
目标：低延迟触发（鼓垫），压力/位置映射音色参数，统一时间源调度。

建议：
- 基于 AudioKit（示例：DrumPadPlayground）或原生 `AVAudioEngine` + `AVAudioSession`（category `.playback`/`.soloAmbient` 视需求）。
- 采样鼓（bd/sd/hh）+ 合成器主音（Oscillator/FM），参数映射：x→speed/滤波，y→pitch，p→gain。

参考：
- AudioKit Docs（`https://audiokit.io/docs/`）
- Apple AVAudioEngine Guide（`https://developer.apple.com/documentation/avfaudio/avaudioengine`）

### 触觉引擎：HapticsEngine（Core Haptics）
需求：`.hapticTransient`（点击）与 `.hapticContinuous`（长震）两类；三档强度；>5s 自动降级；15 分钟休息提醒。

设计要点：
- 触发 `PadHit` 时产生瞬时触觉（强度=档位×压力）；
- 长按/循环时用连续模式，但添加超时降级与冷却；
- UI 配置持久化（UserDefaults/Keychain）。

参考：
- Core Haptics Programming Guide（`https://developer.apple.com/documentation/corehaptics`）

### BLE 客户端：BLEClient（CoreBluetooth）
需求：扫描/自动连接 ESP32 Demo；写入自定义 8B 帧（BeatFrame），无响应写（Write Without Response）。

建议接口：
```swift
struct BeatFrame { let t: UInt16; let padId: UInt8; let vel: UInt8; let x: UInt8; let y: UInt8; let p: UInt8; let rsv: UInt8 }

final class BLEClient: NSObject {
    func start()
    func stop()
    func write(_ frame: BeatFrame)
    var isConnected: Bool { get }
}
```

参考：
- CoreBluetooth Guide（`https://developer.apple.com/documentation/corebluetooth`）

### 同步时钟：SyncClock
目标：所有声音/触觉/外设基于同一时间源（`mach_absolute_time`/`AudioDeviceClock`）。

要点：
- 启动时建立“应用时钟起点”，定期与音频设备时钟校正；
- `MusicEventBus` 发布事件必须带 `atTime`（相对统一时钟）。

参考：
- Audio Timebase & Latency（`https://developer.apple.com/documentation/avfaudio` 搜索 timebase/latency）

### 会话日志：SessionLogger（JSONL）
需求：每次 note/beat 一行 JSON，对齐统一时间源，作为 RL 训练输入。

格式建议：
```json
{"ts": 0.1234, "type": "hit", "pad": 2, "x": 0.42, "y": 0.66, "p": 0.7, "vel": 0.85}
```

实现：
- 采用 `FileHandle` 流式写入；
- 追加会话元数据头（设备、采样率、会话 UUID）。

### UI 层：MainView / SettingsView（SwiftUI）
MainView：
- 网格鼓垫；点击触发 `PadHit`；波纹动画（CAReplicatorLayer/SwiftUI Animations）。
- 链接到 `MusicEventBus`；同时预览触觉档位。

SettingsView：
- 触觉三档；>5s 连续强震自动降级开关；15 分钟休息提醒。

### Web 外壳与原生对齐建议
- 当前 Web 外壳已实现：Pencil → 事件 → 音频（纯 WebAudio 合成）；
- Swift 侧将 Pencil/触摸改为 `UIPointerInteraction`/`Gesture`，数据结构对齐 `PadHit`，直接投递到 `MusicEventBus`；
- 音频/触觉/BLE/日志均订阅 `MusicEventBus`，保证数据驱动。

### 链接速览（参考资料）
- **AudioKit**：`https://audiokit.io/`
- **AudioKit Docs**：`https://audiokit.io/docs/`
- **AVAudioEngine**：`https://developer.apple.com/documentation/avfaudio/avaudioengine`
- **Core Haptics**：`https://developer.apple.com/documentation/corehaptics`
- **CoreBluetooth**：`https://developer.apple.com/documentation/corebluetooth`
- **Human Interface Guidelines (iPadOS)**：`https://developer.apple.com/design/human-interface-guidelines/`
- **WebAudio（对照理解）**：`https://developer.mozilla.org/docs/Web/API/Web_Audio_API`

---

如需我先起一版 SwiftPM 工程骨架（含上述模块空实现与接口），告诉我目标包名与最低 iPadOS 版本即可。


