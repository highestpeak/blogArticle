妄图几招搞定所有字符串的题也不是可能的，只能来分类慢慢涵盖一部分，以不至于手足无措

数组和字符串由很大的相似性，因为字符串就是字符数组，因此数组上的很多算法都可以用于字符串

字符串有一个不太一样的地方就是，字符串中的字符是可以穷举的，数量是有限制的，例如英文字母只有26个，那么就很适合用Hash做法，并进一步的只用一个table[26]这样一个数组的hash实现，

这样的hash打表的方法可以避免在循环中引入条件判断从而降低性能



字符串的另一个特点，就是字符串有很大的重复性，一个字符很可能会多次出现，这也在某种程度上导向了hash解法





字符串问题的查找问题通常针对字符或者子字符串（子数组/子序列），这又和数组有些许类似，但不是完全相同



两个/多个字符串子序列的问题：

1. [最长特殊序列 Ⅰ](https://leetcode-cn.com/problems/longest-uncommon-subsequence-i/)

   这个题很巧，只比较长度和两个字符串是否相等，(其实也是必须仔细分析题目特点后的出来的解法)

2. 重复字符串

   [重复的子字符串](https://leetcode-cn.com/problems/repeated-substring-pattern/)

   [How do I check if a string is entirely made of the same substring?](https://stackoverflow.com/questions/55823298/how-do-i-check-if-a-string-is-entirely-made-of-the-same-substring)

   [重复叠加字符串匹配](https://leetcode-cn.com/problems/repeated-string-match/)





其他问题:

1. [反转字符串中的单词 III](https://leetcode-cn.com/problems/reverse-words-in-a-string-iii/)  即

   ``` 
   输入: "I love drag queen"
   输出: "I evol gard neeuq" 
   ```

   (每次反转一个单词就行了)
   (解法仅供和下面对比)字符串单词切片，每个数组位置是一个单词，反转数组顺序。然后顺序拼接字符串，然后反转字符串，即：

   ``` 
   第一步 “I love drag queen”
   第二步 "queen drag love I"
   第三步 ”I evol gard neeuq“
   ```

   题2：this is a teacher -> teacher a is this
   反转字符串，反转每个单词

2. [ 验证回文字符串 Ⅱ](https://leetcode-cn.com/problems/valid-palindrome-ii/)

   双指针l和r，向中间遍历，遇到l和r字符不一致的就删掉l的或者r的字符，然后判断l到r-1或者l+1到r是否回文

3. [计数二进制子串](https://leetcode-cn.com/problems/count-binary-substrings/)

   这个问题不简单感觉，但是解法很有趣

   对0和1分组，则满足题意的数量是min(group[pre],group[curr])

   这样的问题好像是只有两种情况的问题，0和1各自代表一种情况，形如[删除回文子序列](https://leetcode-cn.com/problems/remove-palindromic-subsequences/)这个问题只有a和b则和只有0和1类同，这是否是一个通用类别则不得而知

4. 大小写转换：

   大写变小写、小写变大写 : 字符 ^= 32;
   大写变小写、小写变小写 : 字符 |= 32;
   小写变大写、大写变大写 : 字符 &= -33;

5. [特殊等价字符串组](https://leetcode-cn.com/problems/groups-of-special-equivalent-strings/)

   问题需要转换：转换为奇数位和偶数位字符排序后，唯一字符串的计数数目

6. [独特的电子邮件地址](https://leetcode-cn.com/problems/unique-email-addresses/)

   状态机解法

7. [字符串的最大公因子](https://leetcode-cn.com/problems/greatest-common-divisor-of-strings/)

   str1 + str2 === str2 + str1 就意味着有解

   GCD 算法

   ```js
   const gcd = (a, b) => (0 === b ? a : gcd(b, a % b))
   ```

8. [比较字符串最小字母出现频次](https://leetcode-cn.com/problems/compare-strings-by-frequency-of-the-smallest-character/)

   注意这个解法（实质上不是字符串的题感觉）

   ```java
   public int[] numSmallerByFrequency(String[] queries, String[] words) {
           
           // 统计
           int [] counter = new int[12];
           for (int i = 0; i < words.length; i++)
               counter[fs(words[i])]++;
           
           // 累和
           for (int i = 9; i >= 0; i--)
               counter[i] += counter[i + 1];
               
           // 拿值
           int[] ret = new int[queries.length];
           for (int i = 0; i < queries.length; i++) 
               ret[i] = counter[fs(queries[i]) + 1];
               
           return ret;
   }
   ```

9. [解码字母到整数映射](https://leetcode-cn.com/problems/decrypt-string-from-alphabet-to-integer-mapping/)

   反向遍历的便利 `"10#11#12" --> "jkab"` 

10. 





遍历字符串时上面的代码改成先`char[] c = S.toCharArray()`再索引效率更高，通过`s.charAt(i)`方法索引多了方法栈和越界检查的消耗。

3[ab2[cd]]还原为abcdcdabcdcdabcdcd

 中缀表达式的计算

网格中的最短路径

``` java
public boolean isSubsequence(String x, String y) {
    int j = 0;
    for (int i = 0; i < y.length() && j < x.length(); i++)
        if (x.charAt(j) == y.charAt(i))
            j++;
    return j == x.length();
}
```



java底层正则实现，这才是字符串处理的巅峰？

