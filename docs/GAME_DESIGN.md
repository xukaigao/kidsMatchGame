# 小朋友连连看 — 开发设计说明 (v2)

> 交接文档，可直接作为上下文交给另一个 agent，继续扩展或修复。

---

## 1. 游戏规则（v2 更新版）

| 规则项 | 说明 |
|--------|------|
| 棋盘 | N×N 格子，部分有 emoji 图案（固定），其余为空白 |
| 目标 | 用线连接每对相同图案 |
| 连线限制 | 只能走空白格，不能穿过其他图案，**不能与已有连线交叉** |
| 图案和连线 | 连接后图案**不消失**，连线**永久保留** |
| 胜利 | 所有配对全部连线成功 |
| 操作 | 点击图案 A → 再点相同图案 B → 自动寻路 |

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
  icon:  string,       // 图案 emoji
  pos1:  {r, c},       // 第一个图案位置
  pos2:  {r, c},       // 第二个图案位置
  color: string,       // 管道颜色（CSS color）
  path:  [{r,c}] | null  // 玩家绘制的路径（null = 未连接）
}
```

---

## 4. BFS 寻路算法

函数：`bfsPath(tiles, occ, r1, c1, r2, c2, S, skipConn)`

- 标准 BFS，找最短路径
- **可通行条件**（对中间格子）：
  1. `tiles[r][c] === null`（不是图案格）
  2. `occ[r][c] === null || occ[r][c] === skipConn`（未被其他线占据）
- `skipConn`：当玩家重新绘制某条线时，忽略自身旧路径的占据
- 不支持越界（无虚拟边框），路径只在棋盘内走

---

## 5. 关卡生成算法

函数：`generateLevel(size, numPairs, maxTries)`

1. 随机打乱所有坐标，选取 `numPairs × 2` 个位置作为图案端点
2. 依次用 BFS 验证每对图案是否可连通（前一对的路径会占用格子）
3. 全部可连通 → 关卡有效，返回 `{size, tiles, conns}`
4. 失败 → 最多重试 300 次，仍失败则递减 pairs 重试

### 注意
- 生成只**验证可解性**，不存储解路径（玩家自己找路）
- 贪心策略：按顺序路由，可能导致某些有解的布局被错误判断为无解
  → 缓解方法：多重试次数（300）+ 减少 pairs 的兜底

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
