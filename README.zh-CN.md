# ε-Ball 随机几何图的渗流相变可视化

[English README](./README.md)

🎮 **在线交互体验**：
- **二维可视化 Demo**（点云、$\varepsilon$-ball 与 MST 跨簇桥）：<https://qiangliu.net/rgg-percolation-viz/>
- **高维 & 多簇探索器**（$d = 2\dots 100$、$m = 2\dots 6$ 簇、前 $m$ 大分量曲线、Log/Linear 横轴与距离集中直方图）：<https://qiangliu.net/rgg-percolation-viz/highdim.html>

这是一个面向二维 **$\varepsilon$-ball 随机几何图（Random Geometric Graphs, RGGs）** 的交互式可视化实验，用于直观观察均匀点云与多种非均匀空间分布下的 **渗流相变（Percolation Phase Transition）**、**分簇平台区（Cluster-Separation Plateau）** 以及 **多尺度跨簇合并现象**。

**零外部依赖、纯浏览器本地运行** —— 整个应用仅由单个 `index.html` 文件组成，无需后端、无需构建工具或任何外部依赖库。在浏览器中直接打开即可实时采样点云、计算欧氏最小生成树（Euclidean Minimum Spanning Tree, EMST）精确合并事件序列，并以 60 帧/秒（frames per second, fps）绘制连通分量演化与最大连通分量（Giant Connected Component, `GCC`）、次大连通分量（Second Giant Connected Component, `SGCC`）双轴渗流曲线。

---

## 问题背景

给定平面上的 $N$ 个点 $\{x_1, \dots, x_N\} \subset \mathbb{R}^2$，将所有欧氏距离满足 $\|x_i - x_j\|_2 \le \varepsilon$ 的点对连边，得到随机几何图 $G(N, \varepsilon)$。记 $D = \max_{i,j}\|x_i - x_j\|_2$ 为点集中**相距最远两点之间的欧氏距离**（用于将不同分布下的距离阈值统一归一化为 `% of D`）。当距离阈值 $\varepsilon$ 从 $0$ 增大到 $D$ 时，关注两个核心序参量：

- **最大连通分量占比（`GCC / N`）**：最大连通子图包含的节点比例；
- **次大连通分量占比（`SGCC / N`）**：第二大连通子图包含的节点比例（在两个大分量即将合并前会出现尖锐峰值）。

在聚类分析与复杂网络中（如 $\varepsilon$-邻域图、单链层次聚类、基于密度的噪声应用空间聚类（Density-Based Spatial Clustering of Applications with Noise, DBSCAN）类密度连通性以及相似性阈值网络），一个核心问题是：

> **在什么空间分布下，$G(N, \varepsilon)$ 会出现多次相变或稳定的“分簇平台区”（使各簇内部已充分连通、簇间尚未合并）？在什么情况下，单一全局阈值 $\varepsilon$ 必然失效？**

由于局部渗流由局部点密度 $\rho(x)$ 控制（当局部期望度 $\rho(x)\,\pi\varepsilon^2$ 跨过 $O(1)$ 量级的临界值时局部连通），**空间密度的非均匀性**会彻底改变渗流曲线的形态。本交互页面提供了 7 种典型空间分布进行对比实验。

---

## 内置的 7 种空间点分布

| 点分布 | 空间密度特征 | 随 $\varepsilon$ 增大的渗流行为 | 是否存在分簇平台区？ |
|---|---|---|---|
| **3 Equal-Density Blobs**（3 等密簇） | 3 个标准差 $\sigma$ 与权重相同、彼此分离的高斯簇 | 三簇在同一尺度 $\varepsilon_1$ 内部同时渗流，直到跨簇间隙 $\varepsilon_2 \gg \varepsilon_1$ 才合并 | **存在宽平台区**（`GCC ≈ 1/3`，`SGCC ≈ 1/3`） |
| **Uniform Square**（均匀正方形） | $[0,1]^2$ 内密度处处均匀 | 全空间在同一临界尺度发生唯一一次尖锐相变，`SGCC` 呈典型单峰 | **无**（本身无分簇结构） |
| **Two Moons**（双月牙流形） | 两条非凸弯曲流形 | 每条流形先沿局部弧长邻域连通，远早于跨流形桥接 | **存在清晰平台区**（`GCC ≈ SGCC ≈ 0.50`） |
| **3 Unequal-Density Blobs**（3 异密簇） | $\sigma = 0.03, 0.07, 0.11$，簇内平均点距相差数倍 | 最密簇早已连通时稀疏簇仍碎裂；等稀疏簇即将连起时簇间尾部已发生桥接合并 | **不存在能同时分出 3 簇的单一全局 $\varepsilon$** |
| **3 Blobs + 25% Noise**（3 簇 + 噪声） | 3 个密簇叠加 25% 均匀背景噪声 | 簇间噪声点充当“踏脚石（stepping stones）”提前搭桥，触发跨簇早熟合并 | **平台区被严重侵蚀/消失** |
| **Single Gaussian**（单个高斯） | 单峰径向密度衰减（中心密、外围稀） | 核心在极小 $\varepsilon$ 率先连通，随后向外连续吸附外围点，`SGCC` 始终很低 | **无**（核-边缘连续吞并） |
| **Power-Law Density**（幂律密度） | 径向幂律密度 $\rho(r) \propto r^{-4/3}$，无特征尺度 | 密度跨越多个数量级，`GCC` 从中心向外平滑增长而无陡峭跳变 | **无** |

---

## 核心算法：用欧氏最小生成树（EMST）精确求解全尺度渗流曲线

若对数百个 $\varepsilon$ 逐一暴力构图，稠密区边数可达 $O(N^2)$，无法实现实时交互。但图的**连通分量划分**与欧氏最小生成树具有严格等价性：

> 两点在 $G(N, \varepsilon)$ 中连通，**当且仅当**它们在欧氏最小生成树（EMST）唯一路径上的最大边权 $\le \varepsilon$。

因此：
1. **稠密图最优的 $O(N^2)$ Prim 算法**只需运行一次，即可精确求出全部 $N-1$ 个连通合并临界距离；
2. **带分量大小直方图的并查集（Union–Find）**按边权升序回放这 $N-1$ 条树边，在 $O(N\alpha(N))$ 时间内即可得到任意 $\varepsilon \in (0, \infty)$ 上**完全精确**的 `GCC(ε)` 与 `SGCC(ε)` 曲线；
3. **60 fps 实时渲染优化**：预计算 1001 桶对数距离点对直方图以实时给出精确边数 $E(\varepsilon)$ 与平均度 $\langle k \rangle = 2E/N$；当 $E \le 40{,}000$ 时使用二维空间哈希网格（Spatial Hash Grid）精确提取全部连边，超稠密区则保留全部已激活的最小生成树（Minimum Spanning Tree, MST）骨架边并叠加确定性均匀抽样边，保证拖动丝滑。

---

## 交互功能说明

- **实时扫描 $\varepsilon$**：拖动中间的 $\varepsilon$ 滑块、直接在右侧渗流曲线上点击/拖拽，或点击 **▶ Sweep ε** 自动播放从孤立点到全图连通的完整相变过程；
- **临界点一键定位**：自动检测并生成 `⚡ Jump`（GCC 陡增跳变点）、`▲ SGCC peak`（次大分量峰值）与 `🎯 Plateau`（分簇平台区几何中心）快捷按钮，支持 `⏮ / ⏭` 逐个跳转；
- **节点 $\varepsilon$-ball 悬停检视**：鼠标悬停左侧任意节点，实时画出该点的 $\varepsilon$ 邻域圆、邻接连边、局部度 $k_i$ 及所属连通分量排名；
- **MST 骨架与跨簇桥接边**：勾选 **Minimum Spanning Tree (MST) & bridges** 后，深灰实线显示当前已连通的树骨架（$\le \varepsilon$），橙色虚线高亮即将触发跨簇合并的关键桥接边（$> \varepsilon$）及其精确合并阈值（占直径百分比）；
- **实验状态分享**：点击 **🔗 Copy link** 可将当前分布、节点数 $N$ 与阈值 $\varepsilon$ 编码至 URL Hash，方便复现与分享。

---

## 本地运行

```bash
git clone https://github.com/pobooo/rgg-percolation-viz.git
cd rgg-percolation-viz
open index.html   # macOS（或用任意现代浏览器直接打开 index.html）
```

---

## 参考文献

- M. Penrose, *Random Geometric Graphs*, Oxford University Press, 2003.
- J. Dall and M. Christensen, "Random geometric graphs," *Physical Review E*, 66, 016121, 2002.
- J. C. Gower and G. J. S. Ross, "Minimum spanning trees and single linkage cluster analysis," *Applied Statistics*, 18(1), 54–64, 1969.
- J. Shi, R. Li, et al., "Local dominance unveils clusters in networks," *Communications Physics*, 7, 170, 2024.
