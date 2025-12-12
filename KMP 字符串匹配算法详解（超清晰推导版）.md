# KMP 字符串匹配算法详解（超清晰推导版）

## ✨ 1. KMP 用来解决什么问题？

用来在 **文本串 text** 中快速查找 **模式串 pattern** 出现的位置。

例如：

```
文本串：aabaabaaf  
模式串：aabaaf  
```

KMP 的优势在于：
 **当遇到不匹配时，不需要从头重新比较，而是利用 pattern 自己的结构进行“跳跃”。**

这个“跳跃信息”就来自 **前缀表（LPS/next 数组）**。

------

# ✨ 2. 前缀表是什么？如何求？

**前缀表 = 每个位置下，最长相等的前后缀长度。**

以 `aabaaf` 为例：

| 子串   | 最长相等前后缀长度             |
| ------ | ------------------------------ |
| a      | 0                              |
| aa     | 1                              |
| aab    | 0                              |
| aaba   | 1                              |
| aabaa  | 2 → 因为前缀“aa”和后缀“aa”相等 |
| aabaaf | 0                              |

最终得到前缀表：

```
a  a  b  a  a  f
0  1  0  1  2  0
```

------

# ✨ 3. 为什么不匹配时会回退？

以文本：`aabaabaaf`
 模式：`aabaaf`

匹配到最后 `b != f` 时，不重新从 0 开始，而是利用 **前缀表**：

我们已经匹配了：

```
a a b a a (f?)  
```

前后缀相等的部分是 `aa`（长度 2）

也就是说：

- 模式串的 **前缀 aa** 与文本串末尾的 **aa** 一定相等
- 所以可以把模式串“向右移”，从前缀后面继续匹配（即下标 2）

前缀表告诉我们的跳转点：

```
f 对应的前缀表前一位 = 2
```

所以 **回退到模式串下标 2**。

这就是 KMP 的加速原理。

------

# ✨ 4. 如何求前缀表？你推导出的规律非常正确

你推导出的核心规则：

- 用 `pattern[j]` 与 `pattern[i]` 比较
- 如果不相等：`j = next[j - 1]`（回退）
- 如果相等：`j++`
- 最终 `next[i] = j`

**这就是前缀表构造的完整逻辑！**

------

# ✨ 5. KMP 前缀表构造代码（依据你的推导）

```java
public int[] getNext(String pattern) {
    int j = 0; // 前缀尾部指针
    int n = pattern.length();
    int[] next = new int[n];
    next[0] = 0;

    for (int i = 1; i < n; i++) {
        // 回退逻辑
        while (j > 0 && pattern.charAt(j) != pattern.charAt(i)) {
            j = next[j - 1];
        }

        // 匹配成功，前缀尾 j 向后移动
        if (pattern.charAt(j) == pattern.charAt(i)) {
            j++;
        }

        next[i] = j;
    }

    return next;
}
```

------

# ✨ 6. 使用 KMP 查找字符串 strStr()

匹配逻辑与前缀表构造完全一样：

```java
public int strStr(String text, String pattern) {
    if (pattern.isEmpty()) return 0;

    int n = text.length(), m = pattern.length();
    int[] next = getNext(pattern);
    int j = 0; // 模式串的指针

    for (int i = 0; i < n; i++) {
        // 回退逻辑
        while (j > 0 && text.charAt(i) != pattern.charAt(j)) {
            j = next[j - 1];
        }

        // 匹配
        if (text.charAt(i) == pattern.charAt(j)) {
            j++;
        }

        // 完全匹配
        if (j == m) {
            return i - j + 1;
        }
    }

    return -1;
}
```

------

# ✨ 7. 全代码整合

```java
public class KMP {

    private int[] getNext(String pattern) {
        int j = 0; // 表示前缀的尾
        int n = pattern.length();
        int[] next = new int[n];
        next[0] = 0;

        for (int i = 1; i < n; i++) {
            while (j > 0 && pattern.charAt(j) != pattern.charAt(i)) {
                j = next[j - 1];
            }
            if (pattern.charAt(j) == pattern.charAt(i)) {
                j++;
            }
            next[i] = j;
        }
        return next;
    }

    public int strStr(String text, String pattern) {
        if (pattern.isEmpty()) return 0;

        int n = text.length(), m = pattern.length();
        int[] next = getNext(pattern);
        int j = 0;

        for (int i = 0; i < n; i++) {
            while (j > 0 && text.charAt(i) != pattern.charAt(j)) {
                j = next[j - 1];
            }
            if (text.charAt(i) == pattern.charAt(j)) {
                j++;
            }
            if (j == m) {
                return i - j + 1; // 完全匹配
            }
        }
        return -1;
    }
}
```

------

# ✨ 8. KMP 的核心一句话总结

> **不匹配时，不回退文本指针，只回退模式指针。
>  模式指针如何回退由前缀表决定。**