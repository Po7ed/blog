本文字符串下标从 $0$ 开始，$n=|S|$。

### 自动机

**自动机（automaton）** 是一种对信号序列进行判定的数学模型，可以模拟人类的思考方式，对给定的一串信号给出真或假的判定。

**确定有限状态自动机（DFA）** 是自动机的一种，在 OI 中较常用，其中信号序列一般使用字符串。

形式化地说，它由五部分组成：

- **字符集（$\Sigma$）**，为信号序列所含信号组成的集合。
- **状态集合（$Q$）**，为所有状态构成的集合。DFA 有许多不同含义的状态，可以视作有向图中点。$Q$ 可以视作点集。
- **起始状态（$s$）**，$s\in Q$，为一个特殊的状态，是所有转移的起点。
- **接受状态集合（$F$）**，$F\subseteq Q$，为一些特殊的状态，是判定的依据。
- **转移函数（$\delta$）**，$\delta(u,c)$ 表示 DFA 在读取到字符 $c$，且当前在状态 $u$ 的情况下，DFA 的状态转移到 $\delta(u,c)$。如果 $u$ 没有 $c$ 的转移，则约定 $\delta(u,v)\gets\mathrm{null}$。$\mathrm{null}\notin F$ 是一个特殊的状态，其不能转移到任何其他状态，不是可接受状态。

若将 DFA 视作无向图，则状态集合可视作点集，转移函数可以视作无向图的边。

设 DFA $A=(\Sigma,Q,s,F,\delta)$ 判定信号序列 $S$，我们有 $A$ 工作流程：

- 设当前状态 $u$，$u$ 初始时为 $s$；
- 对于 $S$ 中的每一个字符 $c$，执行 $u\gets\delta(u,c)$，即通过转移函数，转移到新状态；
- 记 $A(S)=[u\in F]$ 表示是否接受 $S$。

### Trie

考虑构建一个自动机，使得其只接受字符串 $s$。很显然，我们可以对 $s$ 的所有前缀建立状态，将该前缀称为其状态的对应串，然后在所有相邻前缀的状态间建立转移即可。类似的，若使其只接受字符串集合 $S$ 中的字符串，可以依次建立状态。上述自动机即为 Trie 树。

形式化地说，构建一颗 Trie 树需要以下步骤：

- 对于 $S$ 中每个字符串 $s$，枚举其前缀 $s(1,i)$，即遍历 $i=1\dots |s|$；
- 设状态 $u$ 的对应串为 $s(1,i)$，状态 $v$ 的对应串为 $s(1,i-1)$。如果不存在状态对应 $s(1,i)$，新建一个状态；
- 构建转移 $\delta(v,s(i))\gets u$。

```text
         he* -> her*
        /
(s) -> h -> hi -> his*
  \
   i -> it*
```

上图为 $S=\{\texttt{he},\texttt{her},\texttt{his},\texttt{it}\}$ 时的 Trie 树。其中 `(s)` 为起始状态，每个字符串都代表一个状态，`->`、`\`、`/` 代表状态间的转移，带 `*` 的是接受状态。

### KMP 与 AC 自动机

#### 自动机结构

我们希望借助自动机完成下述任务：给定字符串 $s,t$，找出所有等于 $t$ 的 $s$ 的子串。

我们知道 $s$ 的子串相当于 $s$ 的一个前缀的一个后缀，所以我们考虑建立一个自动机使得其只接受后缀为 $t$ 的字符串，然后依次判定 $s$ 的每个前缀（输入串）。

类似 Trie，我们对 $t$ 的所有前缀建立状态，将该前缀称为其状态的对应串，并让每个状态表示输入串的某个后缀等于该状态的对应串。

设状态 $u$ 的深度 $\operatorname{dep}(u)$ 为其对应串长度，定义邻转移为 $u$ 到 $v$ 的转移，使得 $v=\delta(u,c)\land dep(v)=dep(u)+1$。那么显然可以建立若干邻转移，其他默认转移到初始状态 $s$（$s$ 对应串为 $\varnothing$）。如下图中，所有 `->` 均为邻转移。

```text
(s) -> a -> ap -> app -> appl -> appl -> apple
```

但由于一个输入串有若干后缀，可能满足多个状态。可以发现较短后缀一定是较长后缀的后缀，即符合状态 $u$ 的输入串一定符合对应串是“$u$ 的对应串的后缀”的状态。

所以我们修改定义，令每个状态表示**输入串的某个后缀等于该状态的对应串**，且**不存在更长的后缀等于其他状态对应串**。也就是说，输入串只满足最长可匹配后缀的状态。

自然的，对于状态 $u$，若状态 $v$ 是对应串最长的状态，使得 $v$ 的对应串是 $u$ 的对应串的真后缀，则称 $u$ 的失配链接（fail 指针）指向 $v$。通过不断地跳失配链接，我们可以遍历所有对应串是“$u$ 的对应串的后缀”的状态。值得注意的是，失配链接并非转移，其表达的是一种“继承”关系。若 $\delta(u,c)=s$，我们可以 $\delta(u,c)\gets\delta(v,c)$，即继承 $v$ 的转移。故这些继承来的转移也可以视作跳若干次失配链接后邻转移。

```text
                           fail
             <===============================
(s) -> t -> to -> tom -> toma -> tomat -> tomato
```

如上图，`tomato` 的失配链接指向 `to`。

#### 构建过程

- 建立初始状态 $s$；
- 遍历 $i=1\dots |t|$，
  - 设 $s(1,i)$ 的状态为 $u$，$s(1,i-1)$ 的状态为 $v$，状态 $x$ 的失配链接为 $f(x)$；
  - $f(u)\gets\delta(f(v),s(i))$；
  - $\delta(v,s(i))=u$；
  - $\delta(u,*)=\delta(f(u),*)$；

#### 复杂度分析

不难发现判定一个字符串 $s$ 的时间复杂度为均摊 $O(|s|)$，但构建 KMP 自动机的时间复杂度为 $O(|\Sigma||t|)$，因为需要拷贝转移。故实际实现中不会拷贝继承来的转移，构建时间复杂度就降到了 $O(|t|)$，判定字符串时直接跳失配链接，不难证明判定时间复杂度仍为 $O(|s|)$。

综上 KMP 算法时间复杂度为 $O(|s|+|t|)$。

#### 推广

上述 DFA 即为 KMP 自动机，观察其形态可以发现其邻转移形成一条链，其他继承来的转移为“返祖边”。结合 Trie 的思想，我们就可以造出 AC 自动机。但这次我们就需要拷贝继承来的转移，（为什么？）。

如果你理解了 KMP 自动机，那么 AC 自动机的构建方式就十分显然了。

### 参考

- [Bilibili 【manim | 算法】OI自动机大炒饭（字典树，KMP自动机，AC自动机，后缀自动机，广义后缀自动机）](https://www.bilibili.com/video/BV1uV4y1W7cB/)