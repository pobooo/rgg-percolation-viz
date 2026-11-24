# Percolation Phase Transitions in ε-Ball Random Geometric Graphs

[中文文档 (Chinese README)](./README.zh-CN.md)

🎮 **Live demo**: <https://qiangliu.net/rgg-percolation-viz/>

An interactive visualization of **percolation phase transitions**, **cluster-separation plateaus**, and **multi-scale merging** in 2D **$\varepsilon$-ball Random Geometric Graphs (RGGs)** across uniform and inhomogeneous spatial point distributions.

**Zero-dependency, browser-only** — a single standalone `index.html` generates the point clouds, computes the exact Euclidean Minimum Spanning Tree (EMST) merger sequence, evaluates `GCC(ε)` and `SGCC(ε)` across the full log-scale range of $\varepsilon$, and renders both the live component graph and percolation curves on HTML5 Canvas at 60 fps. No backend, no npm, no external CDN libraries.

---

## Motivation

Given $N$ points $\{x_1, \dots, x_N\} \subset \mathbb{R}^2$, connect every pair with Euclidean distance $\|x_i - x_j\|_2 \le \varepsilon$ to form the $\varepsilon$-ball graph $G(N, \varepsilon)$. As $\varepsilon$ increases from $0$ to the point-set diameter $D$:

- **Giant Connected Component (`GCC / N`)**: fraction of nodes in the largest connected component.
- **Second Giant Connected Component (`SGCC / N`)**: fraction of nodes in the second-largest connected component, which peaks sharply right before two major components merge.

In clustering and network science (e.g., $\varepsilon$-neighborhood graphs, single-linkage / DBSCAN-style connectivity, and distance-thresholded similarity networks), a recurring question is:

> **When does $G(N, \varepsilon)$ exhibit multiple phase transitions or a stable plateau where clusters are internally connected yet mutually separated — and when does a single global threshold $\varepsilon$ fail?**

Because local percolation is governed by local point density $\rho(x)$ (a neighborhood percolates once its expected degree $\rho(x)\,\pi\varepsilon^2$ crosses an $O(1)$ threshold), **spatial density heterogeneity** completely reshapes the transition curve. This demo makes all seven canonical regimes concrete and interactive.

---

## Seven Spatial Regimes Included

| Distribution | Intra-cluster / spatial density | Percolation behavior as $\varepsilon$ grows | Cluster-separation plateau? |
|---|---|---|---|
| **3 Equal-Density Blobs** | Three separated Gaussians with identical $\sigma$ and weight | All three blobs percolate internally at the same $\varepsilon_1$, then merge across inter-cluster gaps at $\varepsilon_2 \gg \varepsilon_1$ | **Yes — wide plateau** (`GCC ≈ 1/3`, `SGCC ≈ 1/3`) |
| **Uniform Square** | Homogeneous density on $[0,1]^2$ | Whole domain crosses criticality simultaneously in a single sharp transition; single-peak `SGCC` | **No** (no clusters exist) |
| **Two Moons** | Two non-convex curved manifolds | Each manifold percolates along its local arc neighborhood long before cross-manifold bridging | **Yes — clean plateau** (`GCC ≈ SGCC ≈ 0.50`) |
| **3 Unequal-Density Blobs** | Three Gaussians with $\sigma = 0.03, 0.07, 0.11$ (different intra-cluster spacing) | Densest blob connects early while sparser blobs remain shattered; inter-blob tails bridge before the sparsest blob finishes connecting | **No global $\varepsilon$ separates all 3** |
| **3 Blobs + 25% Noise** | Three dense blobs embedded in uniform background noise | Noise points act as stepping-stone bridges across inter-cluster gaps, triggering premature merging | **Eroded / destroyed** |
| **Single Gaussian** | Unimodal radial density decay from core to periphery | Dense core percolates at tiny $\varepsilon$ and smoothly absorbs outer nodes without abrupt jumps; `SGCC` stays near zero | **No** (core-periphery accretion) |
| **Power-Law Density** | Scale-free radial density $\rho(r) \propto r^{-4/3}$ | Density spans orders of magnitude with no characteristic scale; `GCC` grows smoothly outward | **No** |

---

## Exact Algorithmic Shortcut: Why EMST Suffices for Connected Components

Computing the full edge set of $G(N, \varepsilon)$ at hundreds of $\varepsilon$ values naively would be slow, especially in dense regimes where $|E| = O(N^2)$. However, connected components depend **only** on cycle-free reachability:

> Two nodes belong to the same connected component of $G(N, \varepsilon)$ **if and only if** the maximum edge weight along their unique path in the **Euclidean Minimum Spanning Tree (EMST)** is $\le \varepsilon$.

Consequently:
1. **Exact $O(N^2)$ Prim EMST** (without heap overhead, optimal for dense Euclidean complete graphs) finds all $N-1$ critical merger thresholds once per realization.
2. **Dynamic Union–Find with size histogram** replays the sorted $N-1$ tree edges in $O(N \alpha(N))$ time to produce the **exact** `GCC(ε)` and `SGCC(ε)` step functions for *every* $\varepsilon \in (0, \infty)$, with binary search evaluating any threshold in $O(\log N)$.
3. **60 fps interactive rendering**:
   - A 1001-bin log-distance pair histogram precomputes exact edge counts $E(\varepsilon)$ and mean degree $\langle k \rangle = 2E/N$.
   - When $E(\varepsilon) \le 40{,}000$, a 2D spatial hash grid extracts every exact $\varepsilon$-edge in real time; at higher densities, all active EMST backbone edges ($\le \varepsilon$) plus a deterministic strided edge sample are rendered to keep scrubbing smooth.

---

## Interactive Features

- **Real-time $\varepsilon$ scrubbing**: drag the $\varepsilon$ slider, click/drag directly on the `GCC(ε) / SGCC(ε)` chart, or press **▶ Sweep ε** to animate percolation from isolated nodes to full connectivity.
- **Automatic critical-point chips**: jump directly to detected `GCC` jumps (`⚡ Jump`), the `SGCC` susceptibility-like peak (`▲ SGCC peak`), or the geometric midpoint of a cluster-separation plateau (`🎯 Plateau`), or step through them with `⏮ / ⏭`.
- **Node $\varepsilon$-ball inspector**: hover over any point in the scatter plot to visualize its exact radius-$\varepsilon$ disk, incident neighbors, local degree $k_i$, and component rank/size.
- **MST backbone & inter-cluster bridges**: toggle **MST backbone & bridges** to overlay active tree edges ($\le \varepsilon$) in dark gray and highlight the next critical inter-cluster bridge edges ($> \varepsilon$) in dashed orange with their exact threshold `% D`.
- **Shareable URL state**: **🔗 Copy link** encodes `#dist=...&n=...&eps=...` in the URL hash for reproducible sharing.

---

## Quick Start

```bash
git clone https://github.com/pobooo/rgg-percolation-viz.git
cd rgg-percolation-viz
open index.html   # macOS (or open directly in any modern browser)
```

---

## Directory Layout

```
rgg-percolation-viz/
├── index.html       # Standalone interactive app (HTML + CSS + vanilla Canvas/JS)
├── README.md        # English documentation
└── README.zh-CN.md  # Chinese documentation
```

---

## References

- M. Penrose, *Random Geometric Graphs*, Oxford University Press, 2003.
- J. Dall and M. Christensen, "Random geometric graphs," *Physical Review E*, 66, 016121, 2002.
- J. C. Gower and G. J. S. Ross, "Minimum spanning trees and single linkage cluster analysis," *Applied Statistics*, 18(1), 54–64, 1969.
- J. Shi, R. Li, et al., "Local dominance unveils clusters in networks," *Communications Physics*, 7, 170, 2024.
