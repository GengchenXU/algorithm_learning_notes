

　　KMP 算法是一种**字符串匹配**算法，可以在 O(n+m) 的时间复杂度内实现两个字符串的匹配。

  
字符串匹配问题
----------

  
　　 所谓字符串匹配，是这样一种问题：“字符串 P 是否为字符串 S 的子串？如果是，它出现在 S 的哪些位置？” 其中 S 称为**主串**；P 称为**模式串**。下面的图片展示了一个例子。  

![](https://pic4.zhimg.com/v2-2967e415f490e03a2a9400a92b185310_r.jpg?source=1940ef5c)

　　主串是莎翁那句著名的 “to be or not to be”，这里删去了空格。“no” 这个模式串的匹配结果是 “出现了一次，从 S[6] 开始”；“ob”这个模式串的匹配结果是 “出现了两次，分别从 s[1]、s[10] 开始”。按惯例，主串和模式串都以 0 开始编号。  
　　字符串匹配是一个非常频繁的任务。例如，今有一份名单，你急切地想知道自己在不在名单上；又如，假设你拿到了一份文献，你希望快速地找到某个关键字（keyword）所在的章节…… 凡此种种，不胜枚举。  
　　我们先从最朴素的 Brute-Force 算法开始讲起。

  
Brute-Force
--------------

  
　　顾名思义，Brute-Force 是一个纯暴力算法。说句题外话，我怀疑，“暴力” 一词在算法领域表示 “穷举、极低效率的实现”，可能就是源于这个英文词。  
　　首先，我们应该如何实现两个字符串 A,B 的比较？所谓**字符串比较**，就是问 “两个字符串是否相等”。最朴素的思想，就是从前往后逐字符比较，一旦遇到不相同的字符，就返回 False；如果两个字符串都结束了，仍然没有出现不对应的字符，则返回 True。实现如下：  

![](https://pic1.zhimg.com/v2-f9a7d55f60e346529f70c409dfcda786_r.jpg?source=1940ef5c)

　　既然我们可以知道 “两个字符串是否相等”，那么最朴素的字符串匹配算法 Brute-Force 就呼之欲出了——  

*   枚举 i = 0, 1, 2 ... , len(S)-len(P)
*   将 S[i : i+len(P)] 与 P 作比较。如果一致，则找到了一个匹配。

　　现在我们来模拟 Brute-Force 算法，对主串 “AAAAAABC” 和模式串 “AAAB” 做匹配：  

![](https://pic4.zhimg.com/v2-1892c7f6bee02e0fc7baf22aaef7151f_r.jpg?source=1940ef5c)

　　这是一个清晰明了的算法，实现也极其简单。下面给出 Python 和 C++ 的实现：  

![](https://pic1.zhimg.com/v2-36589bc0279263ec8641a295aea66a0c_r.jpg?source=1940ef5c)![](https://pic2.zhimg.com/v2-ed28c8d60516720cc38c48d135091a58_r.jpg?source=1940ef5c)

　　我们成功实现了 Brute-Force 算法。现在，我们需要对它的时间复杂度做一点讨论。按照惯例，记 n = |S| 为串 S 的长度，m = |P| 为串 P 的长度。  
　　考虑 “字符串比较” 这个小任务的复杂度。最坏情况发生在：两个字符串唯一的差别在最后一个字符。这种情况下，字符串比较必须走完整个字符串，才能给出结果，因此复杂度是 O(len) 的。　　

　　由此，不难想到 Brute-Force 算法所面对的最坏情况：主串形如 “AAAAAAAAAAA...B”，而模式串形如 “AAAAA...B”。每次字符串比较都需要付出 |P| 次字符比较的代价，总共需要比较 |S| - |P| + 1 次，因此总时间复杂度是 ![](https://www.zhihu.com/equation?tex=O%28%7CP%7C%5Ccdot+%28%7CS%7C+-+%7CP%7C+%2B+1%29+%29) . 考虑到主串一般比模式串长很多，故 Brute-Force 的复杂度是 ![](https://www.zhihu.com/equation?tex=O%28%7CP%7C+%5Ccdot+%7CS%7C%29) ，也就是 O(nm) 的。这太慢了！

  
Brute-Force 的改进思路
--------------------

  
　　经过刚刚的分析，您已经看到，Brute-Force 慢得像爬一样。它最坏的情况如下图所示：  

![](https://pic1.zhimg.com/v2-4fe5612ff13a6286e1a8e50a0b06cd96_r.jpg?source=1940ef5c)

　　我们很难降低字符串比较的复杂度（因为比较两个字符串，真的只能逐个比较字符）。因此，我们考虑**降低比较的趟数**。如果比较的趟数能降到足够低，那么总的复杂度也将会下降很多。　　要优化一个算法，首先要回答的问题是 “我手上有什么信息？”　我们手上的信息是否足够、是否有效，决定了我们能把算法优化到何种程度。请记住：**尽可能利用残余的信息，是 KMP 算法的思想所在**。  
　　在 Brute-Force 中，如果从 S[i] 开始的那一趟比较失败了，算法会直接开始尝试从 S[i+1] 开始比较。这种行为，属于典型的 “没有从之前的错误中学到东西”。我们应当注意到，一次失败的匹配，会给我们提供宝贵的信息——如果 S[i : i+len(P)] 与 P 的匹配是在第 r 个位置失败的，那么从 S[i] 开始的 (r-1) 个连续字符，一定与 P 的前 (r-1) 个字符一模一样！  

![](https://pic4.zhimg.com/v2-7dc61b0836af61e302d9474eeeecfe83_r.jpg?source=1940ef5c)

　　需要实现的任务是 “字符串匹配”，而每一次失败都会给我们换来一些信息——能告诉我们，主串的某一个子串等于模式串的某一个前缀。但是这又有什么用呢？

  
跳过不可能成功的字符串比较  

-------------------

　　有些趟字符串比较是有可能会成功的；有些则毫无可能。我们刚刚提到过，优化 Brute-Force 的路线是 “尽量减少比较的趟数”，而如果我们跳过那些**绝不可能成功的**字符串比较，则可以希望复杂度降低到能接受的范围。  
　　那么，哪些字符串比较是不可能成功的？来看一个例子。已知信息如下：  

*   模式串 P = "abcabd".
*   和主串从 S[0] 开始匹配时，在 P[5] 处失配。

![](https://pic2.zhimg.com/v2-372dc6c567ba53a1e4559fdb0cb6b206_r.jpg?source=1940ef5c)

  
　　首先，利用上一节的结论。既然是在 P[5] 失配的，那么说明 S[0:5] 等于 P[0:5]，即 "abcab". 现在我们来考虑：从 S[1]、S[2]、S[3] 开始的匹配尝试，有没有可能成功？  
　　从 S[1] 开始肯定没办法成功，因为 S[1] = P[1] = 'b'，和 P[0] 并不相等。从 S[2] 开始也是没戏的，因为 S[2] = P[2] = 'c'，并不等于 P[0]. 但是从 S[3] 开始是有可能成功的——至少按照已知的信息，我们推不出矛盾。  

![](https://pic1.zhimg.com/v2-67dd66b86323d3d08f976589cf712a1a_r.jpg?source=1940ef5c)

　　带着 “跳过不可能成功的尝试” 的思想，我们来看 next 数组。

  
next 数组
----------

  
　　next 数组是对于模式串而言的。P 的 next 数组定义为：next[i] 表示 P[0] ~ P[i] 这一个子串，使得 **前 k 个字符**恰等于**后 k 个字符** 的最大的 k. 特别地，k 不能取 i+1（因为这个子串一共才 i+1 个字符，自己肯定与自己相等，就没有意义了）。  

![](https://pic3.zhimg.com/v2-49c7168b5184cc1744459f325e426a4a_r.jpg?source=1940ef5c)

　　上图给出了一个例子。P="abcabd" 时，next[4]=2，这是因为 P[0] ~ P[4] 这个子串是 "abcab"，前两个字符与后两个字符相等，因此 next[4] 取 2. 而 next[5]=0，是因为 "abcabd" 找不到前缀与后缀相同，因此只能取 0.  
　　如果把模式串视为一把标尺，在主串上移动，那么 Brute-Force 就是每次失配之后只右移一位；改进算法则是**每次失配之后，移很多位**，跳过那些不可能匹配成功的位置。但是该如何确定要移多少位呢？  

![](https://pic3.zhimg.com/v2-d6c6d433813595dce5aad08b40dc0b72_r.jpg?source=1940ef5c)

　　在 S[0] 尝试匹配，失配于 S[3] <=> P[3] 之后，我们直接把模式串往右移了两位，让 S[3] 对准 P[1]. 接着继续匹配，失配于 S[8] <=> P[6], 接下来我们把 P 往右平移了三位，把 S[8] 对准 P[3]. 此后继续匹配直到成功。  
　　我们应该如何移动这把标尺？**很明显，如图中蓝色箭头所示，旧的后缀要与新的前缀一致**（如果不一致，那就肯定没法匹配上了）！

  
　　回忆 next 数组的性质：P[0] 到 P[i] 这一段子串中，前 next[i] 个字符与后 next[i] 个字符一模一样。既然如此，如果失配在 P[r], 那么 P[0]~P[r-1] 这一段里面，**前 next[r-1] 个字符恰好和后 next[r-1] 个字符相等**——也就是说，我们可以拿长度为 next[r-1] 的那一段前缀，来顶替当前后缀的位置，让匹配继续下去！  
　　您可以验证一下上面的匹配例子：P[3] 失配后，把 P[next[3-1]] 也就是 P[1] 对准了主串刚刚失配的那一位；P[6] 失配后，把 P[next[6-1]] 也就是 P[3] 对准了主串刚刚失配的那一位。  

![](https://pic4.zhimg.com/v2-6ddb50d021e9fa660b5add8ea225383a_r.jpg?source=1940ef5c)

　　如上图所示，绿色部分是成功匹配，失配于红色部分。深绿色手绘线条标出了相等的前缀和后缀，**其长度为 next[右端]**. 由于手绘线条部分的字符是一样的，所以直接把前面那条移到后面那条的位置。因此说，**next 数组为我们如何移动标尺提供了依据**。接下来，我们实现这个优化的算法。

  
利用 next 数组进行匹配
-----------------

  
　　了解了利用 next 数组加速字符串匹配的原理，我们接下来代码实现之。分为两个部分：建立 next 数组、利用 next 数组进行匹配。  
　　首先是建立 next 数组。我们暂且用最朴素的做法，以后再回来优化：  

![](https://pic4.zhimg.com/v2-1dda8f33e5847449cd9784e76e972cab_r.jpg?source=1940ef5c)

　　如上图代码所示，直接根据 next 数组的定义来建立 next 数组。不难发现它的复杂度是 ![](https://www.zhihu.com/equation?tex=O%28m%5E2%29) 的。  
　　接下来，实现利用 next 数组加速字符串匹配。代码如下：  

![](https://pic4.zhimg.com/v2-a6bd81af7cf9bbda32b2cfb0e4858276_r.jpg?source=1940ef5c)

　　如何分析这个字符串匹配的复杂度呢？乍一看，pos 值可能不停地变成 next[pos-1]，代价会很高；但我们使用摊还分析，显然 pos 值一共顶多自增 len(S) 次，因此 pos 值减少的次数不会高于 len(S) 次。由此，复杂度是可以接受的，不难分析出整个匹配算法的时间复杂度：O(n+m).

  
快速求 next 数组
--------------

  
　　终于来到了我们最后一个问题——如何快速构建 next 数组。  
　　首先说一句：快速构建 next 数组，是 KMP 算法的精髓所在，核心思想是 “**P 自己与自己做匹配**”。  
　　为什么这样说呢？回顾 next 数组的完整定义：  

*   定义 “k - 前缀” 为一个字符串的前 k 个字符； “k - 后缀” 为一个字符串的后 k 个字符。k 必须小于字符串长度。
*   next[x] 定义为： P[0]~P[x] 这一段字符串，使得 **k - 前缀恰等于 k - 后缀**的最大的 k.

　　这个定义中，不知不觉地就包含了一个匹配——前缀和后缀相等。接下来，我们考虑采用递推的方式求出 next 数组。如果 next[0], next[1], ... next[x-1] 均已知，那么如何求出 next[x] 呢？  
　　来分情况讨论。首先，已经知道了 next[x-1]（以下记为 now），如果 P[x] 与 P[now] 一样，那最长相等前后缀的长度就可以扩展一位，很明显 next[x] = now + 1. 图示如下。  

![](https://pic1.zhimg.com/v2-6d6a40331cd9e44bfccd27ac5a764618_r.jpg?source=1940ef5c)

  
　　刚刚解决了 P[x] = P[now] 的情况。那如果 P[x] 与 P[now] 不一样，又该怎么办？  

![](https://pic1.zhimg.com/v2-ce1d46a1e3603b07a13789b6ece6022f_r.jpg?source=1940ef5c)

　　如图。长度为 now 的子串 A 和子串 B 是 P[0]~P[x-1] 中最长的公共前后缀。可惜 A 右边的字符和 B 右边的那个字符不相等，next[x] 不能改成 now+1 了。因此，我们应该**缩短这个 now**，把它改成小一点的值，再来试试 P[x] 是否等于 P[now].  
　　now 该缩小到多少呢？显然，我们不想让 now 缩小太多。因此我们决定，在保持 “P[0]~P[x-1] 的 now - 前缀仍然等于 now - 后缀”的前提下，让这个新的 now 尽可能大一点。 P[0]~P[x-1] 的公共前后缀，前缀一定落在串 A 里面、后缀一定落在串 B 里面。换句话讲：接下来 now 应该改成：使得 **A 的 k - 前缀**等于 **B 的 k - 后缀** 的最大的 k.  
　　您应该已经注意到了一个非常强的性质——**串 A 和串 B 是相同的**！B 的后缀等于 A 的后缀！因此，使得 A 的 k - 前缀等于 B 的 k - 后缀的最大的 k，其实就是串 A 的最长公共前后缀的长度 —— next[now-1]！  

![](https://pic3.zhimg.com/v2-c5ff4faaab9c3e13690deb86d8d17d71_r.jpg?source=1940ef5c)

　　来看上面的例子。当 P[now] 与 P[x] 不相等的时候，我们需要缩小 now——把 now 变成 next[now-1]，直到 P[now]=P[x] 为止。P[now]=P[x] 时，就可以直接向右扩展了。  
　　代码实现如下：  

![](https://pic2.zhimg.com/v2-010a582b0c92a92044c43a2a2ea88928_r.jpg?source=1940ef5c)

　　应用摊还分析，不难证明构建 next 数组的时间复杂度是 O(m) 的。至此，我们以 O(n+m) 的时间复杂度，实现了构建 next 数组、利用 next 数组进行字符串匹配。  
　　以上就是 KMP 算法。它于 1977 年被提出，全称 Knuth–Morris–Pratt 算法。让我们记住前辈们的名字：[Donald Knuth](https://en.wikipedia.org/wiki/Donald_Knuth)(K), [James H. Morris](https://en.wikipedia.org/wiki/James_H._Morris)(M), [Vaughan Pratt](https://en.wikipedia.org/wiki/Vaughan_Pratt)(P).  
* * *

  
　　最后附上洛谷 P3375 [【模板】KMP 字符串匹配](https://www.luogu.com.cn/problem/P3375) 的 Python 和 Java 版代码：  

![](https://pic4.zhimg.com/v2-bb2bb039faf1234a74f479d87a0c8410_r.jpg?source=1940ef5c)![](https://pic3.zhimg.com/v2-55db85d27e4989303c9c396292e75eda_r.jpg?source=1940ef5c)![](https://pic4.zhimg.com/v2-4367659cbc867e1bf7d91e93087a5b6b_xs.jpg?source=1940ef5c)海纳

有些算法，适合从它产生的动机，如何设计与解决问题这样正向地去介绍。但 KMP 算法真的不适合这样去学。最好的办法是先搞清楚它所用的数据结构是什么，再搞清楚怎么用，最后为什么的问题就会有恍然大悟的感觉。我试着从这个思路再介绍一下。大家只需要记住一点，PMT 是什么东西。然后自己临时推这个算法也是能推出来的，完全不需要死记硬背。

KMP 算法的核心，是一个被称为部分匹配表 (Partial Match Table) 的数组。我觉得理解 KMP 的最大障碍就是很多人在看了很多关于 KMP 的文章之后，仍然搞不懂 PMT 中的值代表了什么意思。这里我们抛开所有的枝枝蔓蔓，先来解释一下这个数据到底是什么。

对于字符串 “abababca”，它的 PMT 如下表所示：

![](https://pic2.zhimg.com/v2-e905ece7e7d8be90afc62fe9595a9b0f_r.jpg?source=1940ef5c)

就像例子中所示的，如果待匹配的模式字符串有 8 个字符，那么 PMT 就会有 8 个值。

我先解释一下字符串的前缀和后缀。如果字符串 A 和 B，存在 A=BS，其中 S 是任意的非空字符串，那就称 B 为 A 的前缀。例如，”Harry”的前缀包括 {”H”, ”Ha”, ”Har”, ”Harr”}，我们把所有前缀组成的集合，称为字符串的前缀集合。同样可以定义后缀 A=SB， 其中 S 是任意的非空字符串，那就称 B 为 A 的后缀，例如，”Potter” 的后缀包括{”otter”, ”tter”, ”ter”, ”er”, ”r”}，然后把所有后缀组成的集合，称为字符串的后缀集合。要注意的是，字符串本身并不是自己的后缀。

有了这个定义，就可以说明 PMT 中的值的意义了。**PMT 中的值是字符串的前缀集合与后缀集合的交集中最长元素的长度**。例如，对于”aba”，它的前缀集合为 {”a”, ”ab”}，后缀 集合为{”ba”, ”a”}。两个集合的交集为{”a”}，那么长度最长的元素就是字符串”a” 了，长 度为 1，所以对于”aba”而言，它在 PMT 表中对应的值就是 1。再比如，对于字符串”ababa”，它的前缀集合为{”a”, ”ab”, ”aba”, ”abab”}，它的后缀集合为{”baba”, ”aba”, ”ba”, ”a”}， 两个集合的交集为{”a”, ”aba”}，其中最长的元素为”aba”，长度为 3。

好了，解释清楚这个表是什么之后，我们再来看如何使用这个表来加速字符串的查找，以及这样用的道理是什么。如图 1.12 所示，要在主字符串 "ababababca" 中查找模式字符串 "abababca"。如果在 j 处字符不匹配，那么由于前边所说的模式字符串 PMT 的性质，主字符串中 i 指针之前的 PMT[j −1] 位就一定与模式字符串的第 0 位至第 PMT[j−1] 位是相同的。这是因为主字符串在 i 位失配，也就意味着主字符串从 i−j 到 i 这一段是与模式字符串的 0 到 j 这一段是完全相同的。而我们上面也解释了，模式字符串从 0 到 j−1 ，在这个例子中就是”ababab”，其前缀集合与后缀集合的交集的最长元素为”abab”， 长度为 4。所以就可以断言，主字符串中 i 指针之前的 4 位一定与模式字符串的第 0 位至第 4 位是相同的，即长度为 4 的后缀与前缀相同。这样一来，我们就可以将这些字符段的比较省略掉。具体的做法是，保持 i 指针不动，然后将 j 指针指向模式字符串的 PMT[j −1] 位即可。

简言之，以图中的例子来说，在 i 处失配，那么主字符串和模式字符串的前边 6 位就是相同的。又因为模式字符串的前 6 位，它的前 4 位前缀和后 4 位后缀是相同的，所以我们推知主字符串 i 之前的 4 位和模式字符串开头的 4 位是相同的。就是图中的灰色部分。那这部分就不用再比较了。

![](https://pic4.zhimg.com/v2-03a0d005badd0b8e7116d8d07947681c_r.jpg?source=1940ef5c)

有了上面的思路，我们就可以使用 PMT 加速字符串的查找了。我们看到如果是在 j 位 失配，那么影响 j 指针回溯的位置的其实是第 j −1 位的 PMT 值，所以为了编程的方便， 我们不直接使用 PMT 数组，而是将 PMT 数组向后偏移一位。我们把新得到的这个数组称为 next 数组。下面给出根据 next 数组进行字符串匹配加速的字符串匹配程序。其中要注意的一个技巧是，在把 PMT 进行向右偏移时，第 0 位的值，我们将其设成了 - 1，这只是为了编程的方便，并没有其他的意义。在本节的例子中，next 数组如下表所示。

![](https://pic2.zhimg.com/v2-40b4885aace7b31499da9b90b7c46ed3_r.jpg?source=1940ef5c)

具体的程序如下所示：

```
int KMP(char * t, char * p) 
{
	int i = 0; 
	int j = 0;

	while (i < strlen(t) && j < strlen(p))
	{
		if (j == -1 || t[i] == p[j]) 
		{
			i++;
           		j++;
		}
	 	else 
           		j = next[j];
    	}

    if (j == strlen(p))
       return i - j;
    else 
       return -1;
}
```

好了，讲到这里，其实 KMP 算法的主体就已经讲解完了。你会发现，其实 KMP 算法的动机是很简单的，解决的方案也很简单。远没有很多教材和算法书里所讲的那么乱七八糟，只要搞明白了 PMT 的意义，其实整个算法都迎刃而解。

现在，我们再看一下如何编程快速求得 next 数组。其实，求 next 数组的过程完全可以看成字符串匹配的过程，即以模式字符串为主字符串，以模式字符串的前缀为目标字符串，一旦字符串匹配成功，那么当前的 next 值就是匹配成功的字符串的长度。

具体来说，就是从模式字符串的第一位 (注意，不包括第 0 位) 开始对自身进行匹配运算。 在任一位置，能匹配的最长长度就是当前位置的 next 值。如下图所示。

![](https://pic3.zhimg.com/v2-645f3ec49836d3c680869403e74f7934_r.jpg?source=1940ef5c)![](https://pic4.zhimg.com/v2-06477b79eadce2d7d22b4410b0d49aba_r.jpg?source=1940ef5c)![](https://pic2.zhimg.com/v2-8a1a205df5cad7ab2f07498484a54a89_r.jpg?source=1940ef5c)![](https://pic3.zhimg.com/v2-f2b50c15e7744a7b358154610204cc62_r.jpg?source=1940ef5c)![](https://pic4.zhimg.com/v2-bd42e34a9266717b63706087a81092ac_r.jpg?source=1940ef5c)

求 next 数组值的程序如下所示：

```
void getNext(char * p, int * next)
{
	next[0] = -1;
	int i = 0, j = -1;

	while (i < strlen(p))
	{
		if (j == -1 || p[i] == p[j])
		{
			++i;
			++j;
			next[i] = j;
		}	
		else
			j = next[j];
	}
}
```

至此，KMP 算法就全部介绍完了。

=====================


角色：

甲：abbaabbaaba  
乙：abbaaba

乙对甲说：「帮忙找一下我在你的哪个位置。」

甲从头开始与乙一一比较，发现第 7 个字符不匹配。

要是在往常，甲会回退到自己的第 2 个字符，乙则回退到自己的开头，然后两人开始重新比较。[[1]](#ref_1) 这样的事情在字符串王国中每天都在上演：不匹配，回退，不匹配，回退，……

但总有一些妖艳字符串要花自己不少的时间。

上了年纪的甲想做出一些改变。于是乎定了个小目标：**发生不匹配，自身不回退。**

甲发现，若要成功与乙匹配，必须要匹配 7 个长度的字符。所以就算自己回退到第 2 个字符，在后续的匹配流程中，肯定还会重新匹配到自己的第 7 个字符上。

**当在甲的某个字符 _c_ 上发生不匹配时，甲即使回退，最终还是会重新匹配到字符 _c_ 上。**

那干脆不回退，岂不美哉！

**甲不回退，乙必须回退地尽可能少，并且乙回退位置的前面那段已经和甲匹配，这样甲才能不用回退。**

如何找到乙回退的位置？

「不匹配发生时，前面匹配的那一小段 abbaab 于我俩是相同的」，甲想，「这样的话，用 abbaab 的头部去匹配 abbaab 的尾部，最长的那段就是答案。」

```
abbaab 的头部有 a, ab, abb, abba, abbaa（不包含最后一个字符。下文称之为「真前缀」）
abbaab 的尾部有 b, ab, aab, baab, bbaab（不包含第一个字符。下文称之为「真后缀」）
这样最长匹配是 ab。也就是说甲不回退时，乙需要回退到第三个字符去和甲继续匹配。
```

「要计算的内容只和乙有关」，甲想，「那就假设乙在其所有位置上都发生了不匹配，乙在和我匹配前把其所有位置的最长匹配都算出来（算个长度就行），生成一张表，之后我俩发生不匹配时直接查这张表就行。」

据此，甲总结出了一条规则并告诉了乙：

**所有要与甲匹配的字符串（称之为模式串），必须先自身匹配：对每个子字符串 [0...i]，算出其「相匹配的真前缀与真后缀中，最长的字符串的长度」。**

「小 case，我对自己还不了解吗」，乙眨了一下眼睛，「那我回退到第三个字符和你继续匹配吧~」

* * *

新的规则很快传遍了字符串王国。现在来看看如何**高效地**计算这条规则。这里有个很好的例子：abababzabababa。

列个表手算一下：（最大匹配数为子字符串 [0...i] 的最长匹配的长度）

```
子字符串　 a b a b a b z a b a b a b a
最大匹配数 0 0 1 2 3 4 0 1 2 3 4 5 6 ?
```

一直算到 6 都很容易。在往下算之前，先回顾下我们所做的工作：

对子字符串 abababzababab 来说，

真前缀有 a, ab, aba, abab, ababa, ababab, abababz, ...

真后缀有 b, ab, bab, abab, babab, ababab, zababab, ...

所以子字符串 abababzababab 的真前缀和真后缀最大匹配了 6 个（ababab），那**次大**匹配了多少呢？

容易看出次大匹配了 4 个（abab），更仔细地观察可以发现，**次大匹配必定在最大匹配 ababab 中，所以次大匹配数就是 ababab 的最大匹配数！**

直接去查算出的表，可以得出该值为 4。

第三大的匹配数同理，它既然比 4 要小，那真前缀和真后缀也只能在 abab 中找，即 abab 的最大匹配数，查表可得该值为 2。

再往下就没有更短的匹配了。

回顾完毕，来计算 ? 的值：既然末尾字母不是 z，那么就不能直接 6+1=7 了，我们回退到次大匹配 abab，刚好 abab 之后的 a 与末尾的 a 匹配，所以 ? 处的最大匹配数为 5。

* * *

上 Java 代码，它已经呼之欲出了：

```
// 构造模式串 pattern 的最大匹配数表
int[] calculateMaxMatchLengths(String pattern) {
    int[] maxMatchLengths = new int[pattern.length()];
    int maxLength = 0;
    for (int i = 1; i < pattern.length(); i++) {
        while (maxLength > 0 && pattern.charAt(maxLength) != pattern.charAt(i)) {
            maxLength = maxMatchLengths[maxLength - 1]; // ①
        }
        if (pattern.charAt(maxLength) == pattern.charAt(i)) {
            maxLength++; // ②
        }
        maxMatchLengths[i] = maxLength;
    }
    return maxMatchLengths;
}
```

有了代码后，容易证明它的复杂度是线性的（即运算时间与模式串 pattern 的长度是线性关系）：由 ② 可以看出 maxLength 在整个 for 循环中最多增加 pattern.length() - 1 次，所以让 maxLength 减少的 ① 在整个 for 循环中最多会执行 pattern.length() - 1 次，从而 calculateMaxMatchLengths 的复杂度是线性的。

KMP 匹配的过程和求最大匹配数的过程类似，从 count 值的增减容易看出它也是线性复杂度的：

```
// 在文本 text 中寻找模式串 pattern，返回所有匹配的位置开头
List<Integer> search(String text, String pattern) {
    List<Integer> positions = new ArrayList<>();
    int[] maxMatchLengths = calculateMaxMatchLengths(pattern);
    int count = 0;
    for (int i = 0; i < text.length(); i++) {
        while (count > 0 && pattern.charAt(count) != text.charAt(i)) {
            count = maxMatchLengths[count - 1];
        }
        if (pattern.charAt(count) == text.charAt(i)) {
            count++;
        }
        if (count == pattern.length()) {
            positions.add(i - pattern.length() + 1);
            count = maxMatchLengths[count - 1];
        }
    }
    return positions;
}
```

最后总结下这个算法：

1.  匹配失败时，总是能够让模式串回退到某个位置，使文本不用回退。
2.  在字符串比较时，模式串提供的信息越多，计算复杂度越低。（有兴趣的可以了解一下 Trie 树，这是文本提供的信息越多，计算复杂度越低的一个例子。）

参考
--

1.  [^](#ref_1_0) 这一段描述的是最朴素的字符串暴力匹配算法（BF 算法）