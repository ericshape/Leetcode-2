Reference:
[从头彻尾理解KMP算法] (http://blog.csdn.net/v_july_v/article/details/7041827)

### 1. KMP定义
* 假设现在文本串S匹配到 i 位置，模式串P匹配到 j 位置
* 如果j = -1，或者当前字符匹配成功（即S[i] == P[j]），都令i++，j++，继续匹配下一个字符；
* 如果j != -1，且当前字符匹配失败（即S[i] != P[j]），则令 i 不变，j = next[j]。此举意味着失配时，
  模式串P相对于文本串S向右移动了j - next [j] 位。
  * 换言之，当匹配失败时，模式串向右移动的位数为：失配字符所在位置 - 失配字符对应的next 值
  （next 数组的求解会在下文的3.3.3节中详细阐述），即移动的实际位数为：j - next[j]，且此值大于等于1。

#####next 数组各值的含义：代表当前字符之前的字符串中，有多大长度的相同前缀后缀。
#####例如如果next [j] = k，代表j 之前的字符串中有最大长度为k 的相同前缀后缀。
* 此也意味着在某个字符失配时，该字符对应的next 值会告诉你下一步匹配中，模式串应该跳到哪个位置（跳到next [j] 的位置）。
如果next [j] 等于0或-1，则跳到模式串的开头字符，若next [j] = k 且 k > 0，代表下次匹配跳到j 之前的某个字符，而不是跳到开头，
且具体跳过了k 个字符。

### 2. 求next数组
#####a) 寻找前缀后缀最长公共元素长度
* 对于P = p0 p1 ...pj-1 pj，寻找模式串P中长度最大且相等的前缀和后缀。如果存在p0 p1 ...pk-1 pk = pj- k pj-k+1...pj-1 pj，
那么在包含pj的模式串中有最大长度为k+1的相同前缀后缀。举个例子，如果给定的模式串为“abab”，那么它的各个子串的前缀后缀的
公共元素的最大长度如下表格所示：

|模式串 | a | b | a | b |
|---|:---:|---:|---:|---|
|最大前缀后缀公共元素长度| 0 | 0 | 1 | 2 |

比如对于字符串aba来说，它有长度为1的相同前缀后缀a；而对于字符串abab来说，它有长度为2的相同前缀后缀ab
（相同前缀后缀的长度为k + 1，k + 1 = 2）。

#####b) next数组
next 数组考虑的是除当前字符外的最长相同前缀后缀，所以通过第①步骤求得各个前缀后缀的公共元素的最大长度后，
只要稍作变形即可：将第①步骤中求得的值整体右移一位，然后初值赋为-1，如下表格所示：

| 模式串 | a | b | a | b |
|---|:---:|---:|---:|---|
|next数组| －1 | 0 | 0 | 1 |

#####c) 根据next数组进行匹配
匹配失配，j = next [j]，模式串向右移动的位数为：j - next[j]。换言之，当模式串的后缀pj-k pj-k+1, ..., pj-1 跟文本串si-k si-k+1, ..., si-1匹配成功，但pj 跟si匹配失败时，因为next[j] = k，相当于在不包含pj的模式串中有最大长度为k 的相同前缀后缀，即p0 p1 ...pk-1 = pj-k pj-k+1...pj-1，故令j = next[j]，从而让模式串右移j - next[j] 位，使得模式串的前缀p0 p1, ..., pk-1对应着文本串 si-k si-k+1, ..., si-1，而后让pk 跟si 继续匹配。

#####d) 代码求next数组
* 已知next [0, ..., j]，如何求出next [j + 1]呢？
对于P的前j+1个序列字符：

* 若p[k] == p[j]，则next[j + 1 ] = next [j] + 1 = k + 1；
* 若p[k ] ≠ p[j]，如果此时p[ next[k] ] == p[j ]，则next[ j + 1 ] =  next[k] + 1，否则继续递归前缀索引k = next[k]，而后重复此过程。 相当于在字符p[j+1]之前不存在长度为k+1的前缀"p0 p1, …, pk-1 pk"跟后缀“pj-k pj-k+1, …, pj-1 pj"相等，那么是否可能存在另一个值t+1 < k+1，使得长度更小的前缀 “p0 p1, …, pt-1 pt” 等于长度更小的后缀 “pj-t pj-t+1, …, pj-1 pj” 呢？如果存在，那么这个t+1 便是next[ j+1]的值，此相当于利用已经求得的next 数组（next [0, ..., k, ..., j]）进行P串前缀跟P串后缀的匹配。

```python
# nextarr[i] 表示the last longest common prefixes and suffixes in the string p(0:i-1)
# initialization: nextarr[0] is -1
# k means the prefixes and j means the suffixes 
# Note: if p[k]!=p[j], k = nextarr[k]
def nextfuc(self,p):
        length = len(p)
        nextarr = [0] * length
        nextarr[0] = -1   # 
        k = -1
        j = 0
        while j < length - 1:
            if k == -1 or p[k] == p[j]:
                k += 1
                j += 1
                nextarr[j] = k
            else:
                k = nextarr[k] 
        return nextarr
```

Implement strStr()
```python
    def nextfuc(self,p):
        length = len(p)
        nextarr = [0] * length
        nextarr[0] = -1
        k = -1
        j = 0
        while j < length - 1:
            if k == -1 or p[k] == p[j]:
                k += 1
                j += 1
                nextarr[j] = k
            else:
                k = nextarr[k]
        return nextarr
    
    def strStr(self, haystack, needle):
        if needle == "": return 0
        nextarr = self.nextfuc(needle)
        i = 0; j = 0
        lengthh = len(haystack)
        lengthn = len(needle)
        
        while i < lengthh and j < lengthn:
            if j == -1 or haystack[i] == needle[j]:
                i += 1
                j += 1
            else:
                j = nextarr[j]
        if j == lengthn:        
            return i - j
        else:
            return -1
```       

####[Shortest Palindrome](https://github.com/UmassJin/Leetcode/blob/master/Array/Shortest%20Palindrome.py)
