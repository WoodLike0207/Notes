![image-20260130230417168](N%E7%9A%87%E5%90%8E%E9%97%AE%E9%A2%98.assets/image-20260130230417168.png)



# 1. 数组实现

测试链接 : https://leetcode.cn/problems/n-queens-ii/



## 代码

```java
public class NQueens {
    public static int totalNQueens(int n) {
        if (n < 1) {
            return 0;
        }

        return f1(0, new int[n], n);
    }

    private static int f1(int i, int[] path, int n) {
        if (i == n) {
            return 1;
        }

        int ans = 0;

        for (int j = 0; j < n; j++) {
            if (check(path, i, j)) {
                path[i] = j;
                ans += f1(i + 1, path, n);
            }
        }

        return ans;
    }

    private static boolean check(int[] path, int i, int j) {
        for (int k = 0; k < i; k++) {
            if (j == path[k] || Math.abs(i - k) == Math.abs(j - path[k])) {
                return false;
            }
        }
        return true;
    }
}
```



## 思路

### 1. 【一句话核心】

**“一行放一个，列斜别打架”**：我们逐行放置皇后，每一行只放一个，放的时候检查这一列、这两条对角线上是否有以前放过的皇后，如果安全就放下去，去下一行；如果这一行所有列都放不了，就回溯。

### 2. 【算法技巧操作】

- **降维打击（2D 变 1D）**：
  - 通常我们要用二维数组 `board[N][N]` 存棋盘。
  - 但左神（以及经典解法）这里只用了一个一维数组 `int[] path`。
  - **含义**：`path[i] = j` 表示 **第 `i` 行的皇后，放在了第 `j` 列**。
  - *好处*：天然保证了“每一行只放一个皇后”，不需要检查行冲突。
- **数学法判对角线**：
  - 如何判断两个点 `(r1, c1)` 和 `(r2, c2)` 是否在同一条斜线上？
  - 只要 **行差的绝对值 == 列差的绝对值** (`Math.abs(r1 - r2) == Math.abs(c1 - c2)`)，它们就在对角线上（等腰直角三角形原理）。

### 3. 【图解原理】

假设我们要解决 **4 皇后** 问题。

**步骤演示：**

1. **Level 0 (第0行)**：我们把皇后放在第 1 列 (Index 1)。 `path[0] = 1`。
2. **Level 1 (第1行)**：
   - 试第 0 列？不行，对角线冲突（左上图虚线）。
   - 试第 1 列？不行，同列冲突。
   - 试第 2 列？不行，对角线冲突。
   - 试第 3 列？**可以！** `path[1] = 3`。
3. **Level 2 (第2行)**：继续尝试... 如果发现没地方放，就**回溯**，回到 Level 1 改位置。

![image-20260131095012476](N%E7%9A%87%E5%90%8E%E9%97%AE%E9%A2%98.assets/image-20260131095012476.png)



**图解说明：**

- **左侧递归树**：展示了尝试的过程。第一行选了 Col 1，第二行分别尝试 Col 0, 1, 2 都失败（红色X），最后尝试 Col 3 成功（绿色勾）。
- **右侧棋盘**：直观展示了冲突。
  - 蓝色的皇后 Q 是已经放好的 `(0, 1)`。
  - 当我们尝试在 Row 1 放置黄色的皇后 Q 时，红色的虚线标出了蓝色皇后的攻击范围。
  - 可以看到，Row 1 的 Col 0, Col 1, Col 2 都在蓝色皇后的攻击线上（竖线或斜线），所以不能放。只有 Col 3 是安全的。

### 4. 【代码逐行精读】

**Part 1: 主递归函数 `f1`**

```java
// i: 当前正在做决定的行号 (Level)
// path: 记录之前的决定，path[k] 表示第 k 行皇后在第几列
// n: 总共几皇后
private static int f1(int i, int[] path, int n) {
    // Base Case: i 到了 n，说明 0 ~ n-1 行都放好了，找到了一种解法
    if (i == n) {
        return 1;
    }

    int ans = 0;
    // 遍历当前行的每一列 (j)，看看能不能放
    for (int j = 0; j < n; j++) {
        // 【剪枝】检查放在 (i, j) 会不会和之前的皇后打架
        if (check(path, i, j)) {
            path[i] = j; // 记录决定：第 i 行放在 j 列
            ans += f1(i + 1, path, n); // 递归去放下一行，并累加结果
            
            // 注意：这里不需要显式的 path[i] = -1 (回溯)。
            // 因为 path 是覆盖式的，下一次循环 j+1 时，path[i] 会被新值覆盖。
            // 而且 check 函数只检查 k < i 的部分，后面的脏数据不影响。
        }
    }
    return ans;
}
```

**Part 2: 冲突检查函数 `check`**

```java
// 检查如果在第 i 行、第 j 列放皇后，是否合法
private static boolean check(int[] path, int i, int j) {
    // 只需要检查之前的行 (0 到 i-1)
    for (int k = 0; k < i; k++) {
        // path[k] 是之前那一行皇后的列号
        
        // 1. 同列检查：j == path[k]
        // 2. 对角线检查：行差 == 列差
        if (j == path[k] || Math.abs(i - k) == Math.abs(j - path[k])) {
            return false;
        }
    }
    return true;
}
```

### 5. 【死记硬背点 / 易错点】

1. **对角线公式**：`Math.abs(i - k) == Math.abs(j - path[k])`。这个必须背下来，考场上推导容易心慌出错。
2. **`check` 范围**：循环必须是 `k < i`。只看“以前”的行，别看自己，也别看“未来”。
3. **回溯的隐蔽性**：很多人疑惑为什么没有 `path[i] = 0` 这种撤销操作。因为数组是**被复用**的，且 `check` 逻辑限制了只读前缀。下次循环修改 `path[i]` 就自然覆盖了旧状态，这也是一种隐式的回溯。

### 6. 【复习闪卡 (Flashcard)】

1. **Q: 为什么 N 皇后问题不需要检查“行冲突”？**
   - A: 因为我们的递归逻辑是 `f1(i...)`，每次递归 `i+1`，天然保证了每一行只放一个，绝对不会重行。
2. **Q: 用一维数组 `path` 记录棋盘，`path[2] = 5` 是什么意思？**
   - A: 表示第 2 行（Row 2）的皇后放在了第 5 列（Column 5）。
3. **Q: 两个点 (r1, c1) 和 (r2, c2) 在对角线上的数学条件是什么？**
   - A: `|r1 - r2| == |c1 - c2|`。





# 2. 位运算实现



## 代码

```java
public static int totalNQueens2(int n) {
    if (n < 1) {
        return 0;
    }

    int limit = (1 << n) - 1;
    return f2(limit, 0, 0, 0);
}

public static int f2(int limit, int col, int left, int right) {
    if (col == limit) {
        return 1;
    }

    int ban = col | left | right;
    int candidate = limit & (~ban);
    int place = 0;
    int ans = 0;

    while (candidate != 0) {
        place = candidate & (-candidate);

        candidate ^= place;

        ans += f2(limit, col | place, (left | place) >> 1, (right | place) << 1);
    }

    return ans;
}
```



## 思路

### 1.【一句话核心】

**“三束激光扫射法”**：把每一行看作一条二进制线，皇后的攻击范围就像是向下发射的三束激光：**垂直激光 (`col`)**、**向左倾斜激光 (`right`)**、**向右倾斜激光 (`left`)**。通过位运算模拟激光的移动和叠加，瞬间找出安全区。

### 2.【算法技巧操作】

- **位图 (Bitmap)**：用一个 `int` 整数代表一行格子。`1` 代表“有毒/被攻击”，`0` 代表“安全”。
- **激光传导**：
  - **列限制 (`col`)**：垂直向下传，下一行保持不变。
  - **左对角线限制 (`left`)**：下一行攻击位置要**右移**（`>>> 1`，因为行每下移一层，左上来的斜线就往右偏一格）。
  - **右对角线限制 (`right`)**：下一行攻击位置要**左移**（`<< 1`，因为行每下移一层，右上来的斜线就往左偏一格）。
- **截断 (`limit`)**：因为 `int` 有 32 位，我们只需要 N 位。用 `limit`（如 `00011111`）把高位多余的激光截断。

### 3. 【图解原理】

假设我们解决 **5 皇后** 问题。 我们在 **Row 0** 的中间位置放了一个皇后（对应二进制 `00100`）。 现在我们要看 **Row 1** 也就是下一行的哪些位置被“封杀”了。

![image-20260131105918688](N%E7%9A%87%E5%90%8E%E9%97%AE%E9%A2%98.assets/image-20260131105918688.png)



**图解详细说明（结合上方图片）：**

1. **上一行的决定 (`place`)**：

   - 皇后在 **Col 2**。
   - 二进制：`0 0 1 0 0`。

   **三路激光传导**：

   - **`col` (垂直攻击)**：
     - 直接继承上一行。
     - 二进制：`0 0 1 0 0` (中间被占)。
   - **`left` (来自左上角的攻击)**：
     - *视觉*：斜线向右下延伸。
     - *位运算*：`place >>> 1` (右移)。
     - 二进制：`0 0 1 0 0` 变成 `0 0 0 1 0` (Col 3 被占)。
   - **`right` (来自右上角的攻击)**：
     - *视觉*：斜线向左下延伸。
     - *位运算*：`place << 1` (左移)。
     - 二进制：`0 0 1 0 0` 变成 `0 1 0 0 0` (Col 1 被占)。

   **合并限制 (`ban`)**：

   - 把三路攻击并起来 (`OR` 运算)。
   - `col | left | right` = `0 1 1 1 0`。
   - 这意味着 Row 1 的 **Col 1, Col 2, Col 3** 都是“禁区”。

   **计算生路 (`candidate`)**：

   - **`limit`** (5位掩码)：`1 1 1 1 1`。
   - **`~ban`** (取反)：禁区变 0，生路变 1 -> `1 0 0 0 1`。
   - **结果**：只有 **Col 0 (`10000`)** 和 **Col 4 (`00001`)** 是安全的（绿色格子）。

### 4. 【代码逐行精读】

```java
// limit: 用来限制棋盘宽度的掩码，比如 5 皇后就是 0...011111
// col, left, right: 分别代表来自 列、左上、右上 的攻击限制
public static int f2(int limit, int col, int left, int right) {
    // 1. Base Case: 所有的列都被皇后占满了，说明放完了
    if (col == limit) {
        return 1;
    }

    // 2. 计算总限制 (Ban)
    // 把三个方向的“激光”并集起来，1 代表不能放
    int ban = col | left | right;

    // 3. 找出能放的位置 (Candidate)
    // ~ban: 取反，让 1 变成可放，0 变成不可放
    // & limit: 核心操作！把 int 高位生成的 1 全部切掉，只保留 N 皇后的范围
    int candidate = limit & (~ban);

    int place = 0;
    int ans = 0;

    // 4. 遍历所有可放的位置
    while (candidate != 0) {
        // [技巧] 提取 candidate 二进制中最右侧的 1
        // 比如 candidate = 001010，place = 000010
        place = candidate & (-candidate);

        // 把这个位置从候选列表中划掉
        // candidate = 001010 ^ 000010 = 001000
        candidate ^= place;

        // 5. 递归去下一行
        // col | place: 列限制直接继承，加上当前占用的列
        // (left | place) >> 1: 左对角线限制，下一行要往右移一位
        // (right | place) << 1: 右对角线限制，下一行要往左移一位
        ans += f2(limit, col | place, (left | place) >> 1, (right | place) << 1);
    }
    return ans;
}
```

### 5.【死记硬背点 / 易错点】

1. **左移右移别搞反**：
   - `left` 变量（左上角攻击）对应 **右移 `>> 1`**（因为行数变大，列号变大/位权变小）。
   - `right` 变量（右上角攻击）对应 **左移 `<< 1`**（因为行数变大，列号变小/位权变大）。
   - *记忆口诀*：**“Left 变 Right (>>), Right 变 Left (<<)”**。
2. **Limit 必不可少**：在计算 `candidate` 时，`limit &` 这一步是保命的。如果没有它，`~ban` 会产生无数个前导 1，循环永远停不下来。
3. **Lowbit 提取**：`place = candidate & (-candidate)` 是位运算中最常用的提取最右侧 1 的技巧，必须背得滚瓜烂熟。

### 6.【复习闪卡】

1. **Q: 为什么位运算版本比数组版本快很多？**
   - A: 因为它把数组的遍历检查（O(N)）变成了位运算操作（O(1)），直接利用 CPU 并行处理位逻辑的能力。
2. **Q: 变量 `left` 传递给下一层递归时为什么要 `>> 1`？**
   - A: 因为左上角的攻击线延伸到下一行时，在棋盘上是向右下方移动的，对应二进制位向低位移动。
3. **Q: `limit` 是怎么生成的？**
   - A: `limit = (1 << n) - 1`。比如 N=3，`1<<3` 是 `1000`，减 1 变成 `0111`，刚好掩盖后 3 位。









