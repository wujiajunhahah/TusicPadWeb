# TusicPadWeb — Apple Pencil ▶︎ Strudel（外壳 + REPL）

用 **Apple Pencil** 在网页上“画着演奏”，实时驱动官方 **Strudel REPL**（strudel.cc/repl）生成音乐与可视化。

## 特性亮点
- Nothing 风格界面：黑色点阵背景、虚线网格与极简红色强调，贴合 Nothing Phone / CMF 视觉语言。
- 字体栈采用开源组合：`DotGothic16` / `VT323`（点阵感显示字体）+ `Space Grotesk` / `Inter` 正文字体 + `Recursive` 等宽字体，既贴近 Nothing 质感又可合法使用。
- 上半区画布捕获 Pencil 的 `x / y / pressure` 并节流到约 60fps 发送至 Strudel，视觉尾迹为柔和红光点阵。
- 下半区嵌入官方 Strudel REPL，真实生成音频与可视化，无需任何自建合成器。
- Strudel REPL 默认载入 **TusicPad Patch**（底鼓 / 旋律 / hi-hat + 笔压映射），按下 Run 即可听到节奏。
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
2. 下方 Strudel REPL 已自动填入 TusicPad Patch，直接点击 **Run** 解锁音频。
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
- 进入 REPL 后还是空白：点击地址栏刷新或手动打开 `https://strudel.cc/repl`。若页面整体 404，请确认已启用 GitHub Pages。
- 压力值恒定：部分第三方触控笔不提供 pressure，系统会回退到 `0.35`，仍可控制横纵向。
- 状态 Pill 未变绿：确保使用 Apple Pencil 并且在画布区域按压；鼠标不会触发传输。
- 画面太亮或想重绘：点击顶部「清空画布」即可重置视觉尾迹，音频不会被中断。
- 更换音色：在接收片段中将 `gm_strings` 改为 `gm_sawtooth`（空灵）或在底鼓层加入 `.crush(3)`（工业）。

---

## 设计说明
- 色彩系统：主色 `#FF004F`（Nothing 红），辅色 `#FF7A00` 与柔和琥珀，用于尾迹渐变与交互反馈。
- 版式：显示与按钮使用 `DotGothic16`/`VT323` 点阵风格，正文采用 `Space Grotesk`，等宽场景用 `Recursive`，保留工业节奏同时避免版权风险。
- 组件：画布、状态条、启动面板统一使用虚线描边与点阵背景，呼应 Nothing 透明壳与打孔视觉。

---

## 开发提示
- 需要触觉（震动）：网页无法触发系统级振动，可考虑原生 Core Haptics 或外设（Arduino + LRA）。
- 想本地化或离线：可做 Swift Playgrounds 版本，在 iPad 本地直接生成音频与触觉反馈。

License

MIT

---
