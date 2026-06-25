# 小朋友连连看 🎮

适合 3–12 岁儿童玩的连连看网页游戏。用浏览器直接打开，无需安装任何东西。

## 🕹️ 怎么玩

1. 点击一个图案（动物/水果/物品）
2. 再点击另一个**相同的图案**
3. 如果两个图案之间可以用**不超过 2 个拐角**的线连起来，就可以消除！
4. 消除所有图案就算赢！

## 🌟 功能

| 功能 | 说明 |
|------|------|
| 🌱 简单 4×4 | 适合 3–5 岁，8 种图案，更大的格子 |
| 🌿 中等 6×6 | 适合 6–8 岁，18 种图案 |
| 🌳 困难 8×8 | 适合 9–12 岁，32 种图案 |
| 💡 提示 | 高亮显示一对可消除的图案（扣 5 分）|
| 🔀 重排 | 重新打乱剩余图案（扣 2 分）|
| 🏆 最高分 | 每个难度分开记录，保存在浏览器里 |
| 🔊 音效 | 点击、匹配、胜利都有声音 |

## 🚀 运行方法

### 方法一：直接打开
直接双击 `index.html`，用浏览器打开即可游玩。

### 方法二：GitHub Pages（推荐）
1. 在 GitHub 上创建仓库（例如 `kidsMatchGame`）
2. 把所有文件推送上去
3. 进入仓库 **Settings → Pages → Source: main branch / root**
4. 等 1–2 分钟，访问 `https://你的用户名.github.io/kidsMatchGame` 即可

```bash
git init
git add .
git commit -m "🎮 初始版本：小朋友连连看"
git remote add origin https://github.com/你的用户名/kidsMatchGame.git
git push -u origin main
```

## 📁 文件结构

```
kidsMatchGame/
├── index.html          # 游戏主页面
├── css/
│   └── style.css       # 样式（色彩、动画、响应式）
├── js/
│   └── game.js         # 游戏逻辑（算法、音效、渲染）
├── docs/
│   └── GAME_DESIGN.md  # 开发设计说明
└── README.md
```

## 🛠️ 技术栈

- 纯 **HTML + CSS + 原生 JavaScript**，无框架、无构建步骤
- **Web Audio API**：在浏览器里直接生成音效，无需音频文件
- **Canvas API**：绘制连线动画
- **CSS Grid + 动画**：响应式棋盘、消除特效
- **localStorage**：保存各难度最高分
