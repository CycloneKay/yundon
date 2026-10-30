# 12 周增肌计划

一个单文件网页 App（打包成 Android APK 自用），包含**训练计划**和**饮食记录**两块。

- 训练：上肢四分化（周一推胸 / 周三拉背宽 / 周五推肩 / 周日拉背厚），12 周周期化，含减载周、每动作真人 GIF 示范。
- 饮食：拍照或文字描述 → 豆包（火山方舟）拆菜估重 → 本地《中国食物成分表》算热量，成分表没收录的才让 AI 补估值。

## 目录

```
index.html              根目录副本（引用 www/media/）
www/index.html          主文件（Capacitor 打包用）
www/media/*.gif         23 个动作示范 GIF
www/icons/              PWA 图标（192 / 512 / maskable）
AppIcon-1024-noalpha.png  iOS / PakePlus 用的源图
resources/android/res/  Android 自适应图标（anydpi-v26 + xxhdpi/xxxhdpi）
.github/workflows/      云端编译 APK
```

## 饮食记录怎么配

打开 App → 右上角 🍽 图标 → 齿轮 → 填：

| 项 | 值 |
| --- | --- |
| 接口地址 | `https://ark.cn-beijing.volces.com/api/v3` |
| API Key | 火山方舟控制台里的 API Key |
| 模型名 | 方舟的模型名或推理接入点 ID（`ep-` 开头） |
| 每日热量目标 | 自己的目标 kcal |

Key 只存本机 localStorage，不上传别处。

> 网页版直接打开会被浏览器 CORS 拦下，只有在打包后的 APK 里能直连（已通过 CapacitorHttp 走原生网络）。想在浏览器里调试需要自建代理。

## 食物数据

`fooddb.js` 由《中国食物成分表》第 6 版整理，1,635 条，字段为每 100g 的热量/蛋白/碳水/脂肪。构建时内联进 `index.html`。

匹配顺序：显类别名 → 精确名 → 包含关系（只接受"蒸/煮/代表值"这类限定词）→ 二元组相似度；都不中才走 AI 估算，结果按每 100g 缓存到本地。

## 构建

建议直接跑 GitHub Actions（Actions → Build Android APK → Run workflow），产物在 Artifacts 里。

本地构建需要 Node、JDK 17、Android SDK：

```bash
npm install
npx cap add android
cp -r resources/android/res/. android/app/src/main/res/
npx cap sync
cd android && ./gradlew assembleDebug
```

`index.html` 是构建产物，日常改内容直接改它；若要重新生成饮食模块，见 `build_food.py`（需要 `_base_clean.html` 基线）。
