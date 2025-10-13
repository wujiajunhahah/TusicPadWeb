# TusicPadWeb — Apple Pencil ▶︎ Strudel（外壳 + REPL）

用 **Apple Pencil** 在网页上“画着演奏”，实时驱动官方 **Strudel REPL**（strudel.cc/repl）生成音乐与可视化。

## 特性亮点
- Nothing 风格界面：黑色点阵背景、虚线网格与极简红色强调，贴合 Nothing Phone / CMF 视觉语言。
- 字体栈采用开源组合：`DotGothic16` / `VT323`（点阵感显示字体）+ `Space Grotesk` / `Inter` 正文字体 + `Recursive` 等宽字体，既贴近 Nothing 质感又可合法使用。
- 上半区画布捕获 Pencil 的 `x / y / pressure` 并节流到约 60fps 发送至 Strudel，视觉尾迹为柔和红光点阵。
- 下半区嵌入官方 Strudel REPL，真实生成音频与可视化，无需任何自建合成器。
- Strudel 面板内置预设切换：`Default`（Kick/Strings/Hats）与 `Steel Halo — Kanye × Frank`（工业 909 + 氛围人声），一键加载后可继续编辑。
- Strudel 官方已将编辑器迁至根路径 `https://strudel.cc/`（详见 [Strudel 官网主页](https://strudel.cc/) 发布说明）；外壳 iframe 直接访问根域并携带 `?code=` 片段可避免旧版 `/repl` 产生的 404。
- 顶部提供 Nothing 风状态条与 `CLEAR CANVAS` / `Open Strudel` 控件，覆盖式开始面板提醒用户解锁音频。

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
2. 预设默认为 **Default TusicPad Patch**，可通过顶部 `Default / Steel Halo` 按钮切换。
3. 每次切换预设或粘贴新片段后，请在 Strudel 中点击 **Run ▶︎** 解锁音频（iOS 需要此步骤）。
4. 返回上半区用 Apple Pencil 绘制，观察顶部状态条变为红色并显示 `LIVE`（实时传输中）。
5. 如需大视图，可点击 `Open Strudel` 在新标签打开原生编辑器；`Clear Canvas` 仅重置上方视觉，不影响音频。

> 想听 “Steel Halo — Kanye × Frank”？点击 `Steel Halo`，随后按 **Run** 即可。

---

## 预设代码备份
- **Default TusicPad Patch**（外壳默认加载）
- **Steel Halo — Kanye × Frank**（工业 × 空灵映射）

> 两段代码都已内置在预设按钮里，以下为备份，方便手动粘贴或进一步改造。

### Default TusicPad Patch

```js
// === TusicPad Receiver ===
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

setInterval(()=>{
  state.x = lerp(state.x, pen.x, 0.2);
  state.y = lerp(state.y, pen.y, 0.2);
  state.p = lerp(state.p, pen.p, 0.2);
}, 30);

$: stack(
  // Kick layer：压力→音量，横向→速度
  s("bd ~ sd ~").gain(()=>0.45 + state.p*0.8).speed(()=>0.9 + state.x*0.4),

  // Strings：纵向→音高，压力→音量，横向→音色
  note(()=> midi(48 + Math.floor((1 - state.y)*18)))
   .sound("gm_strings")
   .gain(()=> 0.32 + state.p*0.6)
   .shape(()=> state.x)
   .pan(()=> state.x*2-1),

  // Hats：压力→音量，横向→速度
  s("hh*4").gain(()=> state.p*0.5).hp(5000).speed(()=> 1 + state.x*0.5)
)
._pianoroll({cycles:4})._spectrum({speed:0.5}).gain(0.9)
```

### Steel Halo — Kanye × Frank

```js
// === Steel Halo: Kanye × Frank ===
globalThis.pen = {x:0.5, y:0.5, p:0.0};
let state = {x:0.5, y:0.5, p:0.0};
function lerp(a,b,t){return a+(b-a)*t;}

window.addEventListener('message', (e)=>{
  const d=e.data;
  if(!d || d.type!=='pen')return;
  pen.x=Math.max(0,Math.min(1,d.x));
  pen.y=Math.max(0,Math.min(1,d.y));
  pen.p=Math.max(0,Math.min(1,d.pressure||0));
});

setInterval(()=>{
  state.x=lerp(state.x,pen.x,0.2);
  state.y=lerp(state.y,pen.y,0.2);
  state.p=lerp(state.p,pen.p,0.2);
},30);

$: stack(
  s("bd*2 [~ sd] bd [~ hh] bd sd")
    .gain(()=>0.6 + state.p*0.4)
    .speed(()=>0.8 + state.x*0.4)
    .distort(()=>state.p*0.2)
    .crush(()=>2 + state.x*5),

  note(()=> midi(36 + Math.floor((1-state.y)*12)))
    .sound("gm_fretless_bass")
    .gain(()=>0.4 + state.p*0.5)
    .room(0.4)
    .pan(()=>state.x*2-1),

  note(()=> choose(["c5","d#5","g5","a#4"]))
    .sound("gm_choir_aahs")
    .gain(()=>0.3 + state.p*0.4)
    .room(0.7)
    .shape(()=>state.x)
    .slow(2)
    .cut(2),

  s("~ hh*4 [cp*2 ~]")
    .gain(()=>0.2 + state.p*0.3)
    .hp(()=>4000 + state.x*4000)
    .speed(()=>1 + state.x*0.5)
    .pan(()=>Math.sin(performance.now()/500))
)
._pianoroll({cycles:8,playhead:0.4})._spectrum({thickness:3,speed:0.5}).gain(0.9)
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
- 进入 REPL 后还是空白：点击地址栏刷新或手动打开 `https://strudel.cc/`。若页面整体 404，请确认已启用 GitHub Pages。
- iframe 显示 nginx 404：Strudel 已弃用旧路径 `/repl`，请确保外壳加载的是 `https://strudel.cc/`（可携带 `?code=` 查询参数分享片段）。
- 看不到 “Running” 指示或仍然无声：预设切换后必须手动按 **Run ▶︎**；若仍静音，点右上角 `Open Strudel` 在新标签尝试。
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
