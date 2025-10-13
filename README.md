# TusicPadWeb — Apple Pencil ▶︎ Strudel（外壳 + REPL）

用 **Apple Pencil** 在网页上“画着演奏”，实时驱动官方 **Strudel REPL**（strudel.cc/repl）生成音乐与可视化。

## 特性亮点
- Nothing 风格界面：黑色点阵背景、Dot Matrix 字体和极简红色强调，贴合 Nothing Phone/CMF 的视觉语言。
- 上半区画布捕获 Pencil 的 `x / y / pressure` 并节流到约 60fps 发送至 Strudel，视觉尾迹为柔和红光点阵。
- 下半区嵌入官方 Strudel REPL，真实生成音频与可视化，无需任何自建合成器。
- 顶部提供 Nothing 风状态条与 `CLEAR CANVAS` 控件，覆盖式开始面板提醒用户解锁音频。

---

## 快速启动（GitHub Pages）
1. 克隆或直接在 GitHub 上创建仓库 `TusicPadWeb`，将本项目三份文件放在仓库根目录。
2. 访问仓库 **Settings → Pages**。
   - Source 选择 **Deploy from a branch**。
   - Branch 选择 `main`，目录选择 `/ (root)`，点击 **Save**。
3. 稍等 1–2 分钟，访问：`https://<你的 GitHub 用户名>.github.io/TusicPadWeb/`。
4. iPad/Safari 打开该地址，点击「开始演奏」，随后在下方 REPL 粘贴接收片段并点 **Run** 解锁音频。

> 对于仓库 `wujiajunhahah/TusicPadWeb`，发布地址为：`https://wujiajunhahah.github.io/TusicPadWeb/`。

---

## 使用流程
1. 打开页面后点击「Start」，覆盖层会消失并提示在 REPL 内点一次 **Run**。
2. 将下列 Strudel 接收片段粘贴进 REPL，点 **Run**。
3. 返回上半区用 Apple Pencil 绘制，观察顶部状态条变为红色并显示 `LIVE`（实时传输中）。
4. 随时点击「Clear Canvas」重置尾迹，不影响音频映射。

---

## Strudel 端“接收片段”（粘贴到 REPL，点 Run）
> 在下半屏 REPL 的代码区粘贴如下内容，点击 **Run**。这段代码会接收外壳页面发送的 x/y/pressure，并用它们控制声音与可视化。

```js
// === Strudel Receiver ===
// 1) 接收父页面的 Apple Pencil 数据
globalThis.pen = { x:0.5, y:0.5, p:0.0 };
let state = { x:0.5, y:0.5, p:0.0 }; // 平滑后
function lerp(a,b,t){ return a + (b-a)*t; }

window.addEventListener('message', (e)=>{
  const d = e.data;
  if(!d || d.type!=='pen') return;
  pen.x = Math.max(0, Math.min(1, d.x));
  pen.y = Math.max(0, Math.min(1, d.y));
  pen.p = Math.max(0, Math.min(1, d.pressure || 0));
});

// 2) 平滑插值（笔抬起也会保留一段尾音）
setInterval(()=>{
  state.x = lerp(state.x, pen.x, 0.2);
  state.y = lerp(state.y, pen.y, 0.2);
  state.p = lerp(state.p, pen.p, 0.2);
}, 30);

// 3) 用 pen/state 控制音乐 + 可视化
$: stack(
  // 底鼓层：压力→音量，横向→速度
  s("bd ~ sd ~").gain(()=>0.4 + state.p).speed(()=>0.8 + state.x),

  // 旋律层：纵向→音高，压力→音量，横向→音色
  note(()=> midi(48 + Math.floor((1 - state.y)*24)))
   .sound("gm_strings")
   .gain(()=> 0.3 + state.p*0.7)
   .shape(()=> state.x)
   .pan(()=> state.x*2-1),

  // 高频点：速度增强节奏细节
  s("hh*8").gain(()=> state.p*0.5).hp(5000).speed(()=> 1 + state.x*0.5)
)
._pianoroll({cycles:4})._spectrum({speed:0.5}).gain(0.9)
```

---

## 交互映射（默认）
- 横向 (x) → 节奏密度 / 速度
- 纵向 (y) → 音高（上高下低）
- 压力 (p) → 音量 / 强度
- 笔停下：插值保留尾音，衔接更自然

---

## 常见问题（Troubleshooting）
- 没有声音：必须在 REPL 里点一次 **Run**（iOS 音频策略要求用户手势），确认设备未静音并允许声音播放。
- 压力值恒定：部分第三方触控笔不提供 pressure，系统会回退到 `0.35`，仍可控制横纵向。
- 状态 Pill 未变绿：确保使用 Apple Pencil 并且在画布区域按压；鼠标不会触发传输。
- 画面太亮或想重绘：点击顶部「清空画布」即可重置视觉尾迹，音频不会被中断。
- 更换音色：在接收片段中将 `gm_strings` 改为 `gm_sawtooth`（空灵）或在底鼓层加入 `.crush(3)`（工业）。

---

## 设计说明
- 采用 Nothing 标志性的点阵网格、虚线描边与红色强调，保持“透明工业”风格。
- 顶部状态条在 `READY`/`LIVE` 间切换，并以红色点提示实时传输。
- 启动覆盖层为虚线框卡片，凸显硬件感，适合直接部署在 GitHub Pages 展示。

---

## 开发提示
- 需要触觉（震动）：网页无法触发系统级振动，可考虑原生 Core Haptics 或外设（Arduino + LRA）。
- 想本地化或离线：可做 Swift Playgrounds 版本，在 iPad 本地直接生成音频与触觉反馈。

License

MIT

---
