### 蓝（hhoppitree）

#### P11325 【MX-S7-T3】「SMOI-R2」Monotonic Queue

只有 $[1,r_n]$ 会被单调队列处理。$sum$ 为所有出队的 $i$ 的 $c_i$ 之和。$i$ 会将 $j<i\land a_j<a_i$ 的 $j$ pop 掉，但 $l_i$ 会导致 $[r_i+1,r_{i+1}]$ 只能 pop $[l_i,\cdot-1]$。

取 $[1,n]$ 个区间均合法，设 $f_i$ 表示当前选择的最后一个区间右端点为 $i$。考虑对于 $i$ 的所有可能贡献区间 $[\cdot,i-1]$ 转移，对于每个被 $i$ pop 的 $j$，将上一个区间的左端点设为 $j$，右端点也设在 $j$，因为后面的转移过了。

另外的，对于 $k<i\land a_k>a_i$ 的 $k$，贡献区间不能向左覆盖到 $k$，因为 $i$ pop 不掉 $k$。额外地进行转移即可。

