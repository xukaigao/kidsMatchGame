# 小朋友连连看 — 开发设计说明 (v4)

> 交接文档，可直接作为上下文交给另一个 agent，继续扩展或修复。

---

## 1. 游戏规则（v4 更新版）

| 规则项 | 说明 |
|--------|------|
| 棋盘 | N×N 格子，部分有 emoji 图案（固定），其余为空白 |
| 目标 | 用线连接每对相同图案 |
| 连线限制 | 只能走空白格，不能穿过其他图案，**不能与已有连线交叉** |
| 图案和连线 | 连接后图案**不变样**，地板**不变色**，只显示一条细线 |
| **胜利条件** | **所有配对都连上 + 整个棋盘每个格子都被走到（铺满）** |
| 操作 | 拖动：从图案 A 按住拖到相同图案 B 画线（支持鼠标 + 触屏） |

> ⚠️ **铺满要求**是 v4 的核心变化：只把每对连上还不够，必须让所有空格都被某条线经过；留有空格则提示"还没填满"，不算通关。

---

## 2. 技术栈

- 纯 **HTML + CSS + 原生 JavaScript**，无框架、无构建
- **Web Audio API**：实时合成音效（无音频文件）
- **Canvas API**：绘制管道连线（canvas 覆盖在 board 上方，`pointer-events:none`）
- **localStorage**：未使用（本版本无最高分，后续可添加）

---

## 3. 核心数据结构

```javascript
tileGrid[r][c]    // string | null  — 固定图案位置（不变）
occupancy[r][c]   // number | null  — 哪条连线占据了这格（-1 空）

connections[i] = {
  icon:     string,       // 图案 emoji
  pos1:     {r, c},       // 第一个图案位置
  pos2:     {r, c},       // 第二个图案位置
  color:    string,       // 连线颜色（CSS color）
  path:     [{r,c}] | null,  // 玩家绘制的路径（null = 未连接）
  solution: [{r,c}]       // 生成时算出的"铺满解"线段（提示用）
}
```

---

## 4. 关卡生成算法（保证存在铺满解）★核心

因为胜利要求**铺满全盘**，所以不能随机撒点后再验证——必须**反过来先造出铺满解，再倒推题目**。

函数：`hamiltonianPath(S)` + `generateLevel(size, numPairs)`

1. **造哈密顿路径**：用随机化 DFS + Warnsdorff 启发式（优先走"后续可选最少"的邻格）生成一条**经过每个格子恰好一次**的路径。失败则回退到蛇形（serpentine）路径——蛇形天然铺满。
2. **切段**：把这条覆盖全盘的路径切成 `numPairs` 段连续子段，每段 ≥ 2 格。随机分配各段长度（先各给 2，再把剩余格子随机摊派）。
3. **取端点为图案**：每段首尾两格放同一个图案 → 一对。段本身存进 `conn.solution` 作为答案。

### 为什么一定有铺满解
- 这些子段**互不重叠、合起来正好覆盖所有格子**（它们是同一条全覆盖路径切出来的）
- 每段是连续格子 → 本身就是一条合法的不交叉路径
- 所以"按 solution 连"就是一个保证铺满的解；玩家也可能找到别的铺满解

### 覆盖判定
- `coveredCount()`：统计 `tileGrid[r][c]!==null || occupancy[r][c]!==null` 的格子数
- `isBoardFull()`：覆盖数 === `size*size`
- 胜利 = `completedPairs === connections.length` **且** `isBoardFull()`

### 提示
- `showHint()` 直接画出某对的 `conn.solution`（真正的铺满答案线段），不再用 BFS 现算

---

## 6. Canvas 绘制

Canvas 覆盖在棋盘上方（`position:absolute; z-index:10; pointer-events:none`），精确匹配棋盘大小。

绘制顺序（每次 `renderCanvas()`）：
1. **管道线**：`lineWidth = cellPx × 0.52`，`lineCap/lineJoin: round`，`alpha=0.78`
2. **端点实心圆**：`radius = cellPx × 0.38`，`alpha=0.82`（叠在图案上方，呈现 Flow Free 配色效果）
3. **提示虚线**：白色，`lineWidth = cellPx × 0.28`，虚线效果
4. **选中虚线环**：金色，围绕当前选中的图案

> 图案 emoji（DOM `.cell.tile` 元素）的 `z-index:2` 高于 canvas（z-index:10 时反而是 canvas 在上）。
> **实际情况**：canvas 是 absolute，`.cell.tile` 是 `z-index:2` 相对 `#board`。
> 由于 `.cell.tile` 与 canvas 在不同 stacking context，canvas 在上。
> 半透明管道（alpha 0.78）叠在图案上产生"Flow Free"配色效果 — 图案隐约可见。

---

## 7. 可通行 vs 不可通行（对玩家操作）

| 格子状态 | 能否通过 |
|----------|----------|
| 空格，无连线 | ✅ |
| 空格，有本次连线旧路 | ✅（重绘时 skipConn 忽略） |
| 空格，有**其他**连线 | ❌ 不能交叉 |
| 图案格（非目标） | ❌ 不能穿过 |
| 图案格（目标终点） | ✅ 可作为终点 |

---

## 8. 难度配置

```javascript
DIFF = {
  easy:   { size: 4, pairs: 3 },  // 4×4，3 对
  medium: { size: 6, pairs: 5 },  // 6×6，5 对
  hard:   { size: 8, pairs: 8 },  // 8×8，8 对
}
```

格子尺寸（CSS `--cell-size`）随棋盘大小自动切换：
```javascript
CELL_PX = { 4:80, 5:72, 6:64, 7:56, 8:50 }
```

---

## 9. 后续扩展建议

- **拖动绘制**：`mousedown` → `mousemove` → `mouseup` 拖画路径，更直观
- **关卡进度**：每难度内级别数递增，pairs 逐渐增多
- **撤销**：存储历史操作栈，支持一步步撤销
- **动画**：连接成功时沿路径播放光点动画
- **主题切换**：食物、交通工具等不同 emoji 主题组
- **填满奖励**：所有格子都被管道覆盖时额外加星

---

## 10. 部署（GitHub Pages）

```bash
cd C:\gxk\11Code\kidsMatchGame
git init
git add .
git commit -m "🎮 v2：管道连线玩法"
git remote add origin https://github.com/<用户名>/kidsMatchGame.git
git push -u origin main
# GitHub 仓库 Settings → Pages → Source: main / root
```
