﻿# 搜索

如果会重复遍历某些状态，一定要记忆化，使用能确定唯一状态的哈希值（或者 $ \mathbb{Z}_2^d $ 中的一个向量（`bitset`））进行记录，由于此时程序的操作次数一般已经很大，考虑手写哈希表。

需要一层层进行搜索或者答案出现在比较浅的位置，但是状态很大，无法宽搜的考虑迭代加深。

考虑将能合并的状态合并在一起。
有时候可能成为最终答案的（等价）状态大小可能不超过某个范围（一半），和/或者能被拆成几个容易合并的状态，此时考虑（类）折半搜索。

考虑随机化搜索，`random_shuffle()` 等操作，并在一定搜索次数后直接返回当前答案（或者认为是无解）。
Las Vegas: 随机搜索一条路径，然后在其子树内暴力搜索。

有时搜索的状态是对一对对象 $ (A, A') $ 进行对等的变换，第 $ i $ 类变换应用 $ j $ 次于 $ A $ 记为 $ T_{i, j}(A) $，此时如果各类变换之间相互独立（某一类变换不可能被其它任何变换组合得到等价变换），我们可能会希望搜索过程中不经过重复的状态（若一个状态里的一对对象在交换后能变成另一个状态，则二者等价）。
此时搜索到第 $ i $ 类变换时，若 $ A = A' $，只搜索 $ (T_{i,j}(A), T_{i,k}(A')), (j \neq k) $，否则搜索 $ (T_{i,j}(A), T_{i,k}(A')) $ 和 $ (T_{i,j}(A'), T_{i,k}(A)) $。同时对于所有情况下只搜索 $ (T_{i,j}(A), T_{i,j}(A')) $。

# 匈牙利算法

# 数据结构

## 一个用于分析贡献的图表

形如下表：
```plain
    读取
   +-+-+-+
   | |*| |
写 +-+-+-+
入 | |*| |
   +-+-+-+
   | | |*|
   +-+-+-+
```

在其中可以用一些线段表示状态的累加，一些箭头表示状态的转移。
也可以考虑以某种“标准”将贡献方式分类，或者加入一些辅助状态来加速状态转移。

## 基底分解
所维护的数据存在某种 $ + $ （抽象的）运算，且满足结合律。
此时可以将数据，或者其区间下标映射到整数集，进而将整数看作（位数）维向量，对其基底进行分解。

### 常见应用
1. 线段树。
2. 树状数组。
3. 字典树。
4. 多重背包中将物品数量分解成 $ 2 $ 的幂和一个余数，平行转移后能表示出所有可能选择的数量；同样的操作也可用于平行减法运算，此时将高位舍去就是模运算；如果只是判定是否能凑出某个数量，可以考虑用 `bitset` 加速。
5. 快速幂。
6. 比较大小时从高位向低位比较，__当两个数字中至少有一位不同时两个数字不同__。

### 线性基
在解决有关 XOR 的问题时，应考虑线性基，因其问题中每个数的 $ d $ 位二进制拆位可以等价于 $ \mathbb{Z}_2^d $ 中的一个向量。
一般的实现保证线性基中的第 $ i $ 个元素的第 $ i $ 位为 $ 1 $，且所有高于 $ i $ 位的 $ 1 $ 已被消去（成倒三角）。

XOR 的组合问题可以先考虑线性基内的元素组合的方案，再考虑线性基外的元素组合的方案，最后再利用向量分解的唯一性建立对应关系。[5006]

一个大小为 $ k $ 的线性基可以用高斯消元使其中 $ k $ 列有且仅有一个 $ 1 $，从而允许对这 $ k $ 个 $ 1 $ 进行背包。[5006]

## 不等式分解（分块）
当存在 $ A \to B $ 和 $ B \to A $ 两种贡献答案的方式，且两者的执行次数受关于题目条件的不等式限制，则以不等式的最小解作为分块条件。

### 常见应用
1. 将连续的数据分块，完整包含一块的直接统计，两端暴力统计；修改时如是区间修改，对于每一个完整的块直接修改，两端暴力修改；如是单点修改，则可以块内直接暴力修改；如果时间复杂度允许，可在块内以比较特殊的方式存储数据，修改时暴力更新而方便查询，例如解决数颜色问题时对每一块内元素排序。
2. 一部分节点写入其它节点，另一部分读取其它节点。
3. 写入贡献每一块/本块内，读取时读入本块内/每一块。
4. 每过一块进行重构，暴力计算之前所有操作贡献，当前块内操作单独统计。或者，类似的，每过一块记录类似于前缀和的“可加的”信息，这一般是用于压空间。

### 例 1：集合交集
给定一些集合 $ S_i $，保证 $ N = \sum \lvert S_i \rvert \le 10^6 $，再给定 $ q $ 个询问（$ q \le 10^6 $)，询问两个集合交集的最小元素。

首先将两个集合用 `bitset` 表示某个元素是否在集合中，然后按照某个元素的出现次数（$ \mathit{count} $）分块，块大小为 $ \mathit{size} $。
对于 $ \mathit{count} < \mathit{size} $，暴力贡献所有的集合对，插入：$ O(\mathit{size}^2 \log \mathit{size}) $，查询：$ O(q \log \mathit{size}) $；
对于 $ \mathit{count} \ge \mathit{size} $，在每次查询时遍历这些元素，$ O(q \frac{N}{\mathit{size} \cdot \omega}) $。
最小化总复杂度，$ \mathit{size} \approx e^{W(N q \omega)} $（在题目范围下 $ size \approx 2000 $，直接当根号就行）

### 例 2：字符串前缀/后缀匹配 [5182]

给你 $ n $ 个字符串 $ a_i $ ，和 $ Q $ 个询问，每次对两个串 $ S = a_i, T = a_j $ 询问最大的 $ L(0 \leq L \leq \min\{|S|,|T|\})$ 使得 $S[|S| - L + 1 \dots |S|] = T[1\dots L] $ 。（$ S = \sum_{i = 1} ^ {n} |a_i| \leq 6 \times 10^5, 1 \leq Q \leq 10^6 $）

对于所有询问，直接暴力哈希匹配，只不过对于 $ n < \mathit{size} $，对查询结果记忆化，这是考虑到字符串数量较少时，每个串的长度相对较长；数量较多时，每个串长度相对较短，且__由于是前缀/后缀匹配，两个串比较的执行次数取决于较短的串__。

对于 $ n < \mathit{size} $，$ O(\mathit{size}^2 \frac{S}{\mathit{size}}) = O(\mathit{size} \cdot S) $；
对于 $ n \ge \mathit{size} $，$ O(q \frac{S}{\mathit{size}}) $。
最小化总复杂度，$ \mathit{size} \approx \sqrt{q} $，总复杂度：$ O(S \sqrt{q}) $

### 例 1 与例 2 的比较

二者实际上是非常相似的问题，但是一个公共前/后缀的长度具有一定的“连续性”，并且能够被较短串的长度限制，所以只需要对串的数量分块即可，而集合交集问题则需要考虑每个元素的出现次数分块。假如使用例 2 的方法解决例 1，必然需要在一个串中查询另一个串中的所有元素，此时不论是暴力比较或是 $ O(\log n) $ 查询，都无法在当前限制下解决问题。

### 图中贡献的分类/分块

在树中，点与点之间的贡献可以只考虑儿子对父亲的贡献（只有 $ 1 $ 个）；
而在一般图中，考虑对度数分块，则每个点只需要考虑其对大点（$ \frac{n}{\mathit{size}} $ 个）的贡献，当然，小点在统计时要遍历 $ \mathit{size} $ 个邻居。[5214]

## 区间的意义
因为 $ [l',r'] \subseteq [l,r] \to l' \ge l \and r' \leq r$ ，
所以可以将区间看作坐标，将查找一个区间包含的所有区间的问题转化为二维数点问题。

## 数颜色问题

#### 1. 容斥法
将区间表示成二维平面上的点，
则对于__相邻__两个同颜色的点 $ [a,a],[b,b] $，在它们对应的点上 $ +1 $，在 $ [a, b] $ 对应的点上 $ - 1 $。
而对于树上问题，则可以认为每个叶子节点管辖一个数轴上的点，而非叶子节点管辖其儿子所管辖的区间之并；
则对于两个 $ \mathit{DFN} $ 临近的同颜色点，在它们的 $ \mathit{LCA} $ 打 $ -1 $ 标记。

#### 2. 存在性
一个区间不存在某种颜色 $ \to $ 该区间前出现的某种颜色出现的下一个位置在该区间之后；
该区间中某个位置的颜色被统计 $ \to $ 与某个位置同颜色的上一个位置在该区间之前。

一个点出发所能到达的颜色 $ \to $ 遇到的某种颜色的第一个元素
$ \to $ 对排序后相邻两个同种颜色之间所夹的区间/该点与树上与其同色的最近父亲节点所夹的链贡献。

## 二分查找的常数问题
_启发：[4760] 操作次数卡常_
在长为 $ 2^n $ 的数组中二分查找 $ m $ 个点，每个区间可用一次操作判断该区间中是否含至少一个点，找到一个点后可将其删除。

#### 法一

$ 1 $ 次查找 $ m $ 个点，对于每个区间，若其中有点，则将其分成左右两个区间查找。

期望操作次数：
$$
E(T_1) = 1 + 2 \sum_{l=1}^n \frac{2^n}{2^l} \left(1 - \left(\frac{2^n-2^l}{2^n} \right)^m \right)
$$

最多操作次数：
$$
\max(T_1) = 1 + 2 \sum_{l = 0}^{n - 1} \min \left(m,2^l \right)
$$

#### 法二

$ m $ 次查找，每次 $ 1 $ 个点，对于每个区间只操作其一半区间，并决定下一步查找哪一半。

操作次数：
$$
T_2 = mn
$$

在 $ m $ 较小时，法二更加优秀，而在 $ m $ 较大时，法一更加优秀（增长似乎是对数的？）。

#### 取等条件


||$ E(T_1) = T_{2} $|$ \max(T_1) = T_2 $|
|:--:|:--:|:--:|
|$ 2^n = 512 $|$ m \approx 25 $|$ m \approx 42 $|
|$ 2^n = 1024 $|$m \approx 36$|$ m \approx 64 $|
|$ 2^n = 1048576 $|$m \approx 1106$|$ m \approx 2047 $|

## 并查集

不能路径压缩时要启发式合并，但是删除时要按插入顺序的倒序删除。
能用于实现一个类似于链表的数据结构。
可用于将一些元素替换为这些元素的等价元素。
带权并查集。

## 启发式合并

涉及到多次将一个数据结构内的数据完全转移到另一个中时考虑启发式合并，在有其它方法时这么做一般也更好写。

# Kruskal 重构树

# Tarjan

``` c++
if (!dfn[to])
{
	tarjan(to);
	low[v] = min(low[v], low[to]);
}
else
{
	low[v]= min(low[v], dfn[to]); // 此处对 low[v] 更新方式的判断不多余。
}
```

#### 任何情况下直接使用 `low[v] = min(low[v], low[to]); ` 的问题
其在简单环中相当于保证经过任意条树边和/或某几条（而不是一条）非树边相联通。
“某几条”的具体数值和 DFS 顺序有关，所以不能将所有 `low[v]` 相同的点认为是一个边双连通分量中的。
在判断点双连通分量时不正确。

```plain
1 - 2
\   /
  3
/   \
4 - 5
```

`1,2,3,4,5` 此时被认为是一个点双连通分量。

通过一条非树边与更早时访问过的点联通维持了双连通分量的性质。
边双连通分量要求任意两点存在两条边不相交路径，所以只要两个边双连通分量存在公共点即可合并，此时上述写法仍然正确。
而点双连通分量间的公共点也可能是割点，对于一个简单环，正确写法保证其中出现的点均有且仅被 DFS 访问过一次，而经过多条非树边相联通时（其中两边必将引入一个新的点）可能有如上的重复情况。

#### 对于复杂环：

```plain
1.  ||-----+
    ||     |
    ||---+ |
    ||   | |
    ||---+ |
    ||     |
    ||-----+

2. ||------+
   ||      |
   ||---+-+
   ||   |
   ||---+
```

其正确性可被归纳法证明。

#### 横向边：
只存在于有向图，从一个子树的点指向另一个子树的点的边，由于其指向的强连通分量没有连回该点的边（如果存在则已经被访问过），所以不会拓展已有的强连通分量，应舍去。

## 圆方树

# KMP
$ \mathit{next} $ 数组记录模式串  时最大的 $ j(0<=j<i,\text{无匹配:}-1) $，自我匹配时前缀和后缀偏移至少为 $ 1 $（否则直接自我开头匹配到结尾，无意义），所以应从 $ i = 1 $ 而不是 $ i = 0 $ 开始自我匹配。
匹配成功后继续移动 $ \mathit{fail} $ 指针避免漏掉该位置的其它匹配。

# exGcd

# 栈，树，括号，网格，Catalan 数

Catalan 数有多种意义，但是各种意义都有自己的优势和劣势，如用网格上的 DP 式（有 $ i $ 个入栈，$ j $ 个出栈的方案数）就丢失了__顺序信息__，难以表示具体是__哪一对__被匹配。

# 求和/贡献转化

_这些在很多地方都有应用，对推数学题的式子尤其有帮助。_

## 维度分解

$$
\begin{align}
  & \sum_{i = 1}^n \sum_{j = 1}^m f(i)g(j) \\
= & \left[\sum_{i = 1}^n f(i) \right] \left[\sum_{j = 1}^m g(j) \right] \end{align}
$$

## 定义域-值域变换

$$
y= f(x), x \in X, y \in Y \\
\begin{align}
  & \sum_{x \in X} f(x) \\
= & \sum_{y \in Y} y \sum_{x \in X} [f(x) = y]
\end{align}
$$

这种变换的基础一般是变换后第二个求和的范围很小，甚至可以常数时间内求和。
其同样适用于多元函数，如枚举 $ \gcd $ 的值。

## 前缀和

$$
\begin{align}
  & \sum_{i = a}^b f(i) \\
= & \sum_{i = 1}^b f(i) - \sum_{i = 1}^{a - 1} f(i)
\end{align}
$$

注意其在数论中可用于解决边界比较麻烦的问题。

## 对称

$$
\begin{align}
  & \sum_{i = 1}^n [f(i) + f(n - i + 1)] \\
= & 2 \sum_{i = 1}^n f(i)
\end{align}
$$

方便使用前缀和。

# 数论

_感谢 RSY 同志提供的帮助_

## 质因数分解

$$
n = p_1^{a_1}p_2^{a_2}...p_k^{a_k}
$$

## 积性函数

$$
f(ab) = f(a)f(b)
$$

## 常用关系

$$
\sum_{i = 1}^{n} \left\lfloor \frac{n}{i} \right\rfloor = \sum_{i = 1}^{n} d(i)
$$

在一定范围内，枚举所有数的倍数等价于枚举所有数的约数。

$$
d(xy) = \sum_{i \mid x} \sum_{j \mid y} [\gcd(i, j) = 1]
$$

考虑 $ x = p_1^{a_1}p_2^{a_2}...p_k^{a_k},y = p_1^{a'_1}p_2^{a'_2}...p_k^{a'_k}, xy = p_1^{a_1 + a'_1}p_2^{a_2 + a'_2}...p_k^{a_k + a'_k} $；
则 $ d(xy) = (a_1 + a'_1 + 1)(a_2 + a'_2 + 1)...(a_k + a'_k + 1) $。
考虑 $ d(xy) $ 的展开形式，其中每一项都表示从 $ x $ 和 $ y $ 中分别挑几个质数，然后考虑它们的所有不同选择个数；
而这其中所有的方案都等价于选择 $ x $ 和 $ y $ 中所有互质的约数对，因为 $ a_i $ 和 $ a'_i $ 不会出现在同一项，所以每一个质数只由其中一个数贡献。

## 积性数论函数

$$
\begin{align}
1(n) & = 1 \\
\varepsilon(n) & = [n = 1] \\
\mathrm{Id}(n) & = n \\
d(n) & = \lvert \{d : d \mid n \} \rvert \\
\sigma(n) & ={} \sum_{d \perp n} d \\
\varphi(n) & ={} \lvert \{d : d < n,d \perp n \} \rvert = n \prod_{p \mid n}(1 - \frac{1}{p}), \varphi(p^k) = p^k - p^{k - 1} \\
\mu(n) & = \begin{cases}
        (-1)^k, \forall a_i = 1 , \\
        0, \exist a_i > 1 . \\
        \end{cases}
\end{align}
$$

## 狄利克雷（Dirichlet）卷积

$$
(f * g)(n) = \sum_{d \mid n} f(d)g(\frac{n}{d})
$$
满足结合律，交换律，分配律，存在__单位元__ $ \varepsilon $。

可以利用积性函数的性质，考虑 $ n $ 的每一个质因数，然后分别证明：
$$
\forall i, (f * g)\left(p_i^j \right) = \sum_{j = 0}^{a_i} f\left(p_i^j \right)g\left(p_i^{a_i - j} \right)
$$

常见等式：
$$
\begin{align}	
\varepsilon & = \mu * 1 \tag{1} \\
g & = f * 1 \iff f = g * \mu \tag{2} \\
\\
\mathrm{Id} & = \varphi * 1 \tag{3} \\
\varphi & = \mathrm{Id} * \mu \tag{4} \\
\\
d & = 1 * 1 \tag{5} \\
\sigma & = \mathrm{Id} * 1 \tag{6} \\
\sigma & = \varphi * d \tag{7} \\
\mathrm{Id} & = \sigma * \mu \tag{8} \\
\end{align}
$$

(1) 只考虑每个质因数出现一次或零次的情况，由 $ \sum_{i = 0}^k (-1)^i \binom{k}{i} = (1 + (-1))^k $（此处 $ 0^0 = 1 $ ）得到。
(2) 为__莫比乌斯反演__，在原式两侧同乘 $ \mu $ 可得反演后的式子。
(3) 对于其中每一个质因数，由 $ \varphi(p^k) = p^k - p^{k - 1} $ 裂项和得到。
(4) __常用性质__，由 (3) 莫比乌斯反演得到。
(7) 由 (3),(5),(6) 得到。
(8) 由 (6) 莫比乌斯反演得到。

## 例：求 $ \sum_{i = 1}^n \sum_{j = 1}^m \gcd(i, j) $

### 法一：

路线：$ \varepsilon \Rarr \mu \Rarr \varphi$

$$
\begin{align}
  & \sum_{i = 1}^n \sum_{j = 1}^m \gcd(i, j) \\
= & \sum_{d = 1}^{\min(n, m)} d \sum_{i = 1}^n \sum_{j = 1}^m [\gcd(i, j) = d] \\
= & \sum_{d = 1}^{\min(n, m)} d \sum_{i = 1}^{\left\lfloor \frac{n}{d} \right\rfloor} \sum_{j = 1}^{\left \lfloor \frac{m}{d} \right\rfloor} [\gcd(i, j) = 1] & \left(i \larr \frac{i}{d}, j \larr \frac{j}{d} \right) \\
= & \sum_{d = 1}^{\min(n, m)} d \sum_{i = 1}^{\left\lfloor \frac{n}{d} \right\rfloor} \sum_{j = 1}^{\left \lfloor \frac{m}{d} \right\rfloor} \varepsilon[\gcd(i, j)] \\
= & \sum_{d = 1}^{\min(n, m)} d \sum_{i = 1}^{\left\lfloor \frac{n}{d} \right\rfloor} \sum_{j = 1}^{\left \lfloor \frac{m}{d} \right\rfloor} \sum_{u \mid \gcd(i, j)} \mu(u) \cdot 1\left(\frac{\gcd(i, j)}{u}\right) \\
= & \sum_{d = 1}^{\min(n, m)} d \sum_{i = 1}^{\left\lfloor \frac{n}{d} \right\rfloor} \sum_{j = 1}^{\left \lfloor \frac{m}{d} \right\rfloor} \sum_{u \mid i, u \mid j} \mu(u) & (u \mid i, u \mid j \iff u \mid \gcd(i, j)) \\
= & \sum_{d = 1}^{\min(n, m)} d \sum_{u = 1}^{\min\left(\left\lfloor \frac{n}{d} \right\rfloor, \left \lfloor \frac{m}{d} \right\rfloor \right)} \mu(u) \sum_{i = 1}^{\left\lfloor \frac{n}{ud} \right\rfloor} \sum_{j = 1}^{\left \lfloor \frac{m}{ud} \right\rfloor} 1 & \left(i \larr \frac{i}{u}, j \larr \frac{j}{u}; \left\lfloor \frac{\left\lfloor \frac{a}{b} \right\rfloor}{c} \right\rfloor = \left\lfloor \frac{a}{bc} \right\rfloor \right) \\
= & \sum_{d = 1}^{\min(n, m)} d \sum_{u = 1}^{\min\left(\left\lfloor \frac{n}{d} \right\rfloor, \left \lfloor \frac{m}{d} \right\rfloor\right)} \mu(u) \left\lfloor \frac{n}{ud} \right\rfloor \left \lfloor \frac{m}{ud} \right\rfloor \\
= & \sum_{d = 1}^{\min(n, m)} d \sum_{\frac{T}{d} = 1}^{\min(n, m)} \mu\left(\frac{T}{d}\right) \left\lfloor \frac{n}{T} \right\rfloor \left \lfloor \frac{m}{T} \right\rfloor & (T \larr ud) \tag{*} \\
= & \sum_{T = 1}^{\min(n, m)} \sum_{d \mid T} \mu\left(\frac{T}{d} \right) \cdot 1(d) \left\lfloor \frac{n}{T} \right\rfloor \left \lfloor \frac{m}{T} \right\rfloor \\
= & \sum_{T = 1}^{\min(n, m)} \varphi(T) \left\lfloor \frac{n}{T} \right\rfloor \left \lfloor \frac{m}{T} \right\rfloor
\end{align}
$$

注意 (\*) 处凑卷积的一个小技巧，更一般来说：将枚举两个因数转变为枚举它们的积和其中一个因数。

### 法二：

路线：$ \mathrm{Id} \Rarr \varphi $

$$
\begin{align}
  & \sum_{i = 1}^n \sum_{j = 1}^m \gcd(i, j) \\
= & \sum_{i = 1}^n \sum_{j = 1}^m \mathrm{Id}[\gcd(i, j)] \\
= & \sum_{i = 1}^n \sum_{j = 1}^m \sum_{d \mid \gcd(i, j)} \varphi(d) \\
= & \sum_{d = 1}^{\min(n, m)} \varphi(d) \left\lfloor \frac{n}{d} \right\rfloor \left \lfloor \frac{m}{d} \right\rfloor
\end{align}
$$

总结：法二更加简单，但是需要直接对函数本身直接卷积，这依赖于给定函数的性质和在式子中的位置，使这种办法有时难以使用。

## 杜教筛

$$
\begin{align}
\sum_{i = 1}^n (f * g)(i) =& \sum_{i = 1}^n \sum_{j \mid i} f(j)g\left(\frac{i}{j} \right) \\
\sum_{i = 1}^n (f * g)(i) =& \sum_{i = 1}^n g(i) \sum_{j = 1}^{\left\lfloor \frac{n}{i} \right\rfloor} f\left(j \right) \\
\sum_{i = 1}^n (f * g)(i) =& \sum_{i = 1}^n g(i) S\left(\left\lfloor \frac{n}{i} \right\rfloor \right) & \left(S(n) = \sum_{i = 1}^n f(i) \right) \\
g(1)S(n) =& \sum_{i = 1}^n (f * g)(i) - \sum_{i = 2}^n g(i) S\left(\left\lfloor \frac{n}{i} \right\rfloor \right)
\end{align}
$$

# 双重缓存

当一系列元素需要“同时”贡献对方，并造成__相互作用__时，就应进行双重缓存，例如分层图中每一层同时向下一层拓展时，此时应该：
1. 遍历缓存 $ A $，计算元素间的贡献。
2. 遍历缓存 $ A $，更新元素状态，写入缓存 $ A' $。
3. 交换缓存 $ A $，$ A' $。

应该注意的情况有：
1. $ A \to C, B \to D $，平凡。
2. $ A \to B, B \to A $。
3. $ A \to C, B \to C $。


# 读题

将题目完整地读完。
时空限制，全题的限制不要看成是某个子任务的限制，同时要注意__下界__。
某个范围明显较一般情况小的变量。
非常好拿的部分分，这些部分分有时也是对正解的启发，但并不绝对。
括号里的东西，有时候这才是难点。
题目条件退化情况。
题目条件中的不变量。
注意精度要求是绝对精度还是相对精度，特别地，注意其中的连接词是“和”还是“或”。

# 求解

各种题目都可以打表找规律，有些 DP 题之类的其实也可以这么做。
一定要__列出式子和/或贡献方式示意图__，这二者对数据结构的选择很有启发作用。
尽量将题目条件__可视化__，如把题目中的“时间”转化为“位置”，这远比考虑抽象的概念方便地多。[5235]
如果手模样例似乎比让一个暴力程序来模拟方便的话，那么考虑这么做，反正比一动不动地空想好。
使用 Excel 进行函数拟合。

# 实现

开编译警告。
获取一个 512 MB 的栈空间`-Wl,--stack=536870912`。

先写程序的框架再实现其中每一部分。
完成一个关键部分后先对该部分编译/调试。
程序的每个部分应尽量少依赖于之前部分的"保证"或副作用。

考虑题目条件的退化情况，无解，空集，$ 0 $ 等。
类似的，对于图__考虑开头，结尾，重边，自环，有向图中的双向边，图不连通，单独一个点，是树或基环树等特殊情况__（不一定在题目中给出，可能是经过某些算法处理后的结果）。

选择一个恰当的 $ \mathit{null} $ 值，这个值需要能够与其它“存在”状态区分开，不一定是 $ 0 $，因为有时候空状态也必须要记录下来（如记忆化搜索）。
选择一个恰当的 $ \infin $ 值，要足够大，但是不会在几步运算后溢出。

使用动态内存的数据结构时应当注意获取正确的空间大小，同时在需要时清空，如不能保证清空时释放内存，考虑将其用空的对象替换。

遍历一个数据结构时应当以数据结构自身的限制（大小等）作为循环的范围，而不是题目给的 $ n $ 之类的，否则在调整代码时容易出现问题。

__更改代码实现中的数据范围时要注意更改数组大小和其它相关位置的范围（如暴力优化为正解，数据范围算错等）。__

线段树所需的最大内存是与 $ 2^n $ 对齐的，可以适当开大一些，但是不要过大。

注意阶乘数组的大小（注意组合数的取值范围）。

引入一些临时变量，防止间接引用某个元素时忘记你是在“间接”引用，如令 `to = edge[v][i]`。

如果能简化实现的话，考虑预处理一些信息，特别是某些字符串题考虑记录下一个匹/失配位置。

用一个查询表来记录信息，而不是写一堆判断（将数据与代码分离），如枚举四个方向常用的 `xMove[] ` 与 `yMove[] ` 表。（但在某些情况下，这样反而会变慢）

考虑使用时间戳而不是真的清空数据结构。

哈希时要注意使数值与模数互质，选取一个质数的确可以保证这个，但是当使用 `unsigned long long` 类型的自然溢出时，实际上是对 $ 2^{64} $ 取模，所以不能使用 $ 2, 26 $ 之类的权值。而且，哈希碰撞是有可能发生的，考虑双哈希。

哈希表可以考虑在每个位置处建立一个链式前向星，但是为了节省内存，又考虑到哈希碰撞发生可能没有那么频繁，直接暴力寻找下一个位置一般也是可以的。

使用二进制第一位寻找反向边时，边的编号从 $ 2 $ 开始（当然，$ 0 $ 也是可以的，只不过不太方便）。

不要忘记线段树深入时下放标记，__回溯时合并该点的子节点答案__。

对对象进行合并时考虑其中一个或多个是退化情况。

对于规整的多层循环嵌套（枚举坐标），考虑将其置于同一缩进深度，但此时一定要__注意 `break` 和 `continue` 的使用，考虑 `goto`__。

对于 `long long` 类型，应使用 `__builtin_popcountll() `。

在边界引入占位符，简化代码。

整数除法默认是向 $ 0 $ 取整的，所以当数据范围包括正数和负数时，如果要进行二分之类的操作，考虑使用位运算保证向下取整。

GNU C 库里定义了 3 个糟糕的宏：`major`, `minor`, `makedev`，不要使用这 3 个名字作为标识符。

#### 常见浮点数精度

`float` 和 `double` 一般为 IEEE-754，`long double` 一般为 80 位 x87 浮点类型。

|类型          |十进制有效位|
|:-----------:|:--------:|
|`float`      |$ 7.22 $  |
|`double`     |$ 15.95 $ |
|`long double`|$ 19.56 $ |

#### Off-By-One!

注意循环开始结束的边界，注意取等条件。

注意下标从 $ 0 $ 还是 $ 1 $ 开始，$ n - 1 $ 还是 $ n $ 还是 $ n + 1 $ 结束；
STL 中的区间是左闭右开，查找时未找到，或者调用 `std::unique()` 时返回的是最后一个元素之后的指针；
将以上两点结合时尤其容易出错。

# 调试

容易实现出现问题的一定要写暴力。

尽量输出方便人眼阅读的调试信息，尽量直观，不要吝啬空间；
最好添加一些明显的分隔符。

优先考虑构造数据的基本情况（有代表性的），如果基本情况正确，很多情况下复杂情况的正确性就可以被归纳证明，这样也方便调试。
对程序的瓶颈构造极限数据，在树论问题中考虑链和菊花图。
对拍时从一个大的数据开始，逐渐减小数据规模，最后对找到的导致错误的最小数据再进行手动压缩。

# 杂项

对于环上问题，考虑将环长扩大一倍。[2347]
求含某个区间的最小区间数完全覆盖 $ \to $ 将区间排序（去除相互包含的情况），每个区间所能接触到的最远区间向当前区间连边，DFS，用队列维护当前覆盖方案。[2347]

计数时考虑求出满足部分条件的答案，再减去不满足的情况。[5147]
$ \text{图不联通的情况数} \to \text{枚举1号节点联通块的所有大小} \times \text{枚举剩余点任意连接的方案} $。[5147]

矩阵，向量（或其他较复杂的代数对象）与整数，实数（等简单代数对象）共享很多性质，有些公式可直接套用。[5145]
$$
\frac{1}{1 - a} = \sum_{i = 0}^{\infty} a^i(a < 0)
$$

将每个状态映射到容易处理的集合中（如 $ \mathbb{Z} $），然后将复杂操作看成等价的基本运算。（势能化）

图的邻接矩阵与一般的矩阵可以相互转化，尤其注意_逆着转化_的方向。[5145]
矩阵的无穷次幂中的非零项等价于以其为邻接矩阵的图中对应两点联通。[5145]

解决配对问题时有时可以不考虑具体是谁和谁配对，而是考虑用（无标号的）整数运算统计配对情况；
还有时钦定一些元素也不影响最终的答案。
两个集合元素相互匹配，考虑分别给两个集合的每个元素赋值 1/-1，然后相加，以描述二者的匹配/一方比另一方的多少情况。[5127]

将点对点贡献__距离__转化为__贡献（链/子树/联通块）大小（的次数）__（可理解为这些点都需要经过当前的一条边/点而来到达另一个点）

$+1 +2 +3 +4 \to (+1) + (+1 +1) + (+1 +1 +1) + (+1 +1 +1 +1)）$。[5142]

构造题：对于每个目标变量先找极端情况，然后考虑一个变动造成的差分，并考虑某些量的变化是否是单调的。[5124]
不要用高维算法解决低维问题（用图描述临近元素之间的关系等基本关系）。

传递闭包的已知最快算法：先将图用 Tarjan 缩点，然后在形成的 DAG 上拓扑排序，使用 bitset 传递信息，时间复杂度 $ O(\frac{n^2}{\omega}) $。

树上背包时如果用 $ \mathit{size} $ 限制循环的范围__（注意：应遍历父节点和子节点的值来主动更新父节点的值）__，此时虽然是3重循环，但是时间复杂度为 $ O(n^2) $

模运算可以认为是多次减法运算，同理，多次减法运算也可以转化为模运算，之后使用快速幂等方式快速求解，这些题目一般会对减法运算的前提条件做出限制，使其满足模运算的性质。

用图描述条件之间的依赖与排斥关系，各种等式与不等式（不只是差分约束系统），将其描述为连通性，奇偶性，遍历的性质，是否有环等。

证明边界正确性时引入占位符，同实现中的类似技巧。

懒惰思想：题目要求按照某个顺序进行一个操作（操作间有依赖关系），此时可以将该顺序中某些操作先合并到一起，使其与按顺序进行等价。[5152]

贪心时尝试考虑物品的平均价格进行贪心，但是这很多时候只是感性理解，不一定正确。[5238]
但其中有一种常见情况是正确的，当在一个排序上选择的某个点 $ i $ 会与这个选择之后的每个点 $ j $ 产生贡献时，且贡献为 $ a_i \sum_{j = i + 1}^n b_j $ 时，考虑对于两个点 $ i, j $ 比较 $ a_i b_j $ 与 $ b_i a_j $ 的大小，这就类似于按“平均值”排序（树上可能有类似问题）。[5152]

轮廓线 DP：用 $ 0/1 $ 表示一条轮廓线上的向下/向右线段，每次遍历轮廓线上的每一个点，每个点对应于其右上角的格子。转移时注意先找出一条轮廓线上的向下-向右对，然后再转移，降低时间复杂度。[5128]

解决两个集合间关系的问题时，考虑二分图（染色）。[4760]

图的 $ 2^n $ 染色可以看作 $ n $ 次二分图染色。[3876]

考虑均摊复杂度，有时进行 $ A $ 操作的次数是受限于某一个较小的变量的，同时有些 $ B' $ 操作的进行必须与之前的另一个操作 $ B $ 相对应，其又受限于另一个变量。[5214]

在每个状态对前/后状态均有后效性，但是后效性可被限制在某个区间内时，考虑区间 DP。

区间 DP 不是真的直接将一个区间断成两截（这根本没有意义，因为 $ [1,3] = [1,2] + [3] = [1] + [2,3] = [1] + [2] + [3] $ ），而是考虑其中有代表性的元素（或者直接遍历每一个元素），将其移除，然后将区间断成  $ L-x-R $ 的形式。

这是预处理区间 $ \mathit{min}/\mathit{max} $ 的代码：

```c++
F[x][x] = y;
for (int l = 1; l <= n; ++l)
{
    for (int r = l + 1; r <= n; ++r)
    {
        F[l][r] = min(F[l][r - 1], F[r][r]);
    }
}
```

优化建图的基本思路：将多组点与点之间的关系（边）拆开为一系列的新点和新边，其中边的贡献可以叠加，而点表示边的自由组合，使得新图的关系等价于原图的关系。

当一个问题的答案是关于一些区间的状态（值）的函数时（有时出现区间重叠也是可以的），且区间的贡献是关于位置和定值数组的的函数时，考虑用一个类似于前缀和的函数 $ f(\mathit{pos}) \to \mathit{value} $ 表示区间，如果有多（类）区间相互影响，考虑构造运算 $ f \oplus g $ 表示整体的局面。[3665]

研究图的连通性，奇偶性时可以尝试从度数角度分析。[5196]

构造解决配对问题时可以考虑将 $ i $ 与 $ i + \frac{n}{2} $ 配对。[5196]

整数三分（注意边界）（此处为开口向下）：

```c++
int l(0), r(n);
while (l < r)
{
    int lm(l + (r - l) / 3), rm(r - (r - l) / 3);
    if (eval(lm) < eval(rm))
    {
        l = lm + 1;
    }
    else
    {
        r = rm - 1;
    }
}
```

树的重心，树的直径

到其它点距离最小的点

高维前缀和

区间 DP 时考虑区间最值对答案的限制。[5274]

维护平面上的连续对象时，可以考虑分治，将平面不断二分；然后对于每一条分割线，只维护跨过这条分割线的元素，同时考虑其分割开的两块区域的答案对当前分割线上答案的限制（如：平面最近点对）。[5276]

维护网格图上单调向右/向下的路径时，注意所有的路径都被夹在两条路径之中：优先向右和优先向下，这可以方便考虑连通性问题。同时可以考虑预处理从起点和终点出发能到达的网格，然后枚举一条分割线，将从起点和终点出发的路径合并。[5277]

求根时可以通过当前点 $ x_i $ 的答案 $ f(x_i) $ 处理出 $x_{i + 1} $ 的下界 $ X_{i + 1} $，使得 $ x_{i + 1} \ge X_{i + 1} $，这可以认为是在最理想的情况下 $ f(X_{i + 1}) = 0 $。[5278]

网络流中一系列的边在某些问题中可以认为是对其容量求最小值（最小割），此时可以用流量为 $ \infin $ 的边表示不等式约束关系（不可割）。[5279]

当求解方案的存在性/一种方案/极值方案时，可能只有有限的非法状态，一般存在于图论问题中的一些特殊情况，又例如换根 DP 时经常保留两个极值以保证其中一个一定在子树外，此时对于最多 $ k $ 种非法状态，仅需要保留 $ k + 1 $ 种状态 。 [5309] [5330]

#### 函数 DP 的一个思想：

例如 [5315]：
你有一个长度为 $ n - 1 $ 的正整数数组 $ b $ ，现在你需要构造一个长度为 $ n $ 的正实数数组 $ a $ ，满足对于任意 $ i $ ，都有 $ a_i \cdot a_{i + 1} \ge b_i $。在此基础上你需要最小化 $ \sum_{i = 1}^{n} a_i $ 。请求出这个最小值。

按照题意可以定义：
$$
f_{i+1}(a_{i + 1}) = \min_{a_i \ge \frac{b_i}{a_{i + 1}}}  f_i(a_i) + a_{i + 1}
$$

但是这么定义相当于将区间代入记录点信息的函数中，实际上并没有办法求解。
考虑到代入的是一个对变量最小值的限制，可以将函数定义变成其后缀最小值：
$$
g_{i+1}(a_{i + 1}) = g_{\min\ i} \left(\frac{b_i}{a_{i + 1}} \right) + a_{i + 1} \\
g_{\min\ i + 1}(a_{i + 1}) = \min_{x \ge a_{i + 1}} g_i(x)
$$
这样使得函数一个点的值能够代表一个区间。

本题的具体情况是 $ g $ 函数在每一次转移后会有与打勾函数一样的单调性，
此时从 $ \mathrm{argmin}(g) $ 的位置向左拍平函数，以求得 $ g_{\min} $。

分析 DP 时用图像表示转移的路径，分析哪些状态是不可能到达的，通过平移/旋转使得斜向转移变成网格上的转移。根据最终贡献答案的状态调整转移方式。在适宜的情况下使一个状态表示前缀和。[4654]

在进行路径计数时，可以用翻折的方式统计跨过某一条直线的方案数，具体地，将该直线向跨过后的一侧平移一个单位，然后将路径第一次跨过原直线后的部分沿这条直线对称。[1276]

解决概率与期望的问题时考虑一个局面的答案是否可由其子局面线性组合而成，若无法判断可以先大胆猜想。[5375]

[5363][5367]
[5359]
[5330][5373]

分块单调栈
