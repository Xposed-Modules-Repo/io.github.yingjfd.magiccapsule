# MagicCapsule

让荣耀 MagicOS 的**灵动胶囊保持展开**：胶囊出现后一直显示长条形态（图标 + 信息），
不会过一会儿自动收缩成只显示图标的小圆点。

> ⚠️ 本项目由 AI 在人类指导下生成并维护，包括全部逆向分析与 Hook 代码。使用请自担风险。

## 功能

| 功能 | 说明 |
|------|------|
| 📌 胶囊不收起 | 拦截系统的「收起」状态，灵动胶囊始终停留在展开态 |
| 🎛️ 开关可关 | 设置页可随时关掉，恢复系统原生行为（需软重启 SystemUI） |

## 适用场景

音乐、计时、通知等会用到灵动胶囊的场合 —— 系统原本会在一段时间后把胶囊收成一个小圆点，
信息随之消失；开启本模块后胶囊会一直保持展开，信息持续可见。

## 下载

见 [Releases](https://github.com/Xposed-Modules-Repo/io.github.yingjfd.magiccapsule/releases) 页面的 `MagicCapsule-v1.0.0.apk`。

## 作用域

| 包名 | 用途 |
|------|------|
| `com.android.systemui` | 灵动胶囊的展开/收起状态机 |

## 要求

- 荣耀 MagicOS 11（Android 17）实测通过；同类实现理论上适用于其它 MagicOS 版本
- KernelSU / Magisk + LSPosed（Zygisk 模式，API 102）
- 模块声明 `staticScope=true`，作用域由 APK 内的 `scope.list` 固定为 `com.android.systemui`

## 使用方法

1. 安装 APK，在 LSPosed 管理器中**启用模块**（作用域会自动带上 `com.android.systemui`）
2. **软重启 SystemUI**（无需重启手机），两种方式任选：
   - 设置页底部的 **「重启系统界面（SystemUI）」** 按钮，点一下即可（需要 root，会二次确认）；
   - 或命令行：
   ```bash
   adb shell su -c 'killall com.android.systemui'
   ```
   或：设置 → 应用 → 显示系统进程 → 系统界面 → 强行停止
3. 播放音乐 / 触发胶囊，观察它是否一直保持长条

## 原理

反编译 SystemUI 后定位到展开/收起的唯一决策点：

```
com.hihonor.systemui.dynamicserver.view.dynamiccapsule.HnDynamicCapsuleSetLayout
    #setDisplayState(int displayState, boolean animate)
```

状态值：`0` = 展开（长条），`1` = 收起（圆点）。模块把入参 `1` 改成 `0`，
并把另一条直接调用的 `setToCollapseState(...)` 重定向到展开方法，其余原生逻辑（动画、
状态机、可见性）一行未改。关掉模块即完全恢复原生行为。

## 安全

- 所有 Hook 回调 `try/catch(Throwable)`，异常不会抛回 SystemUI
- 目标类/方法不存在时静默跳过，换机型或系统版本只会失效，不会造成系统异常
- 只改方法入参，不写系统文件、不改布局参数、不常驻服务

## 源码与构建

完整源码、构建说明与逆向记录见 [GitHub](https://github.com/yingjfd/MagicCapsule)。

## License

MIT
