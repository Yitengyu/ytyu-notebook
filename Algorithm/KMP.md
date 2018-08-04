## KMP (**Knuth–Morris–Pratt string-searching algorithm**)

> In [computer science](https://www.wikiwand.com/en/Computer_science), the Knuth–Morris–Pratt string-searching algorithm (or **KMP algorithm**) searches for occurrences of a "word" `W` within a main "text string" `S` by employing the observation that **when a mismatch occurs, the word itself embodies sufficient information to determine where the next match could begin**, thus bypassing re-examination of previously matched characters.



KMP算法用来查找子字符串在字符串中第一次出现的位置。

以Leetcode第28题的题干为例，就是解决一个这样的问题。



> Implement [strStr()](http://www.cplusplus.com/reference/cstring/strstr/).
>
> Return the index of the first occurrence of needle in haystack, or **-1** if needle is not part of haystack.
>
> **Example 1:**
>
> ```
> Input: haystack = "hello", needle = "ll"
> Output: 2
> ```
>
> **Example 2:**
>
> ```
> Input: haystack = "aaaaa", needle = "bba"
> Output: -1
> ```



最简单的代码像这样，让子字符串的从字符串的第一位开始逐一匹配，当失配后，从第二位开始匹配，直到找到为止。

```javascript
/**
 * @param {string} haystack
 * @param {string} needle
 * @return {number}
 */
var strStr = function(haystack, needle) {
    const l1 = haystack.length
    const l2 = needle.length
    let res = l2 !== 0 ? -1 : 0

    if (l2 && l1 >= l2) {
        for (let i = 0; i <= l1 - l2; ++i) {
            let j = 0
            for (; j < l2; ++j) {
                if (haystack[i + j] !== needle[j]) {
                    break
                }
            }
            if (j === l2) {
                res = i
                break
            }
        }
    }

    return res
};
```



但其实这样的匹配是有重复的。

如查找 'ababc' 中的 'abc'： 在 'aba' 与 'abc' 的匹配失败后，不必再去匹配 'bab' 与 'abc'，因为此时我们知道之前的 'ab' 既然已经匹配，'ab'中的`a`与`b`又不相同，若只移一位进行匹配，匹配失败是必然的。

那么最多可以移几位去匹配呢？ 这就是KMP算法中需要解决的核心问题。

 **当字符串第x位失配，字符串最多可以移动几位去继续匹配？**

如果解决了这个问题，那么在实现函数strStr的问题中，我们可以为`needle`建立一个匹配位移表，代码可以改写成这样。

```javascript
/**
 * @param {string} haystack
 * @param {string} needle
 * @return {number}
 */
var strStr = function(haystack, needle) {
    const l1 = haystack.length
    const l2 = needle.length
    const matchOffset = getMatchOffset(needle) 
    let res = l2 !== 0 ? -1 : 0

    if (l2 && l1 >= l2) {
        let i = 0
        while (i <= l1 - l2) {
            let j = 0
            for (; j < l2; ++j) {
                if (haystack[i + j] !== needle[j]) {
                    i += matchOffset[j]
                    break
                }
            }
            if (j === l2) {
                res = i
                break
            }
        }
    }

    return res
};
```



有公式

```
移动位数 = 已匹配的字符数 - 已匹配字符串的部分匹配值
```

"部分匹配值"（Partial Match Table）： "前缀"和"后缀"的最长的共有元素的长度

"前缀"： 指除了最后一个字符以外，一个字符串的全部头部组合；

"后缀"： 指除了第一个字符以外，一个字符串的全部尾部组合。



我对部分匹配值的理解是： **字符串与自身错位匹配的最大匹配字符数**。前缀和后缀是尝试匹配时的前者的组合和后者的组合。

为了建立匹配位移表，我们需要求出长度为n的字符串的n个头部子串的部分匹配值。如“ABCAB”，需要求出["A", "AB", "ABC", "ABCA", "ABCAB"]各个字符串的部分匹配值。



KMP的复杂度一定比之前那个低吗？

