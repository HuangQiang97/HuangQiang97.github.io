# 短URL生成

网页长URL分享时，对用户不友好，同时给同一个长URL生成多个短URL，每个短URL对应一个引流渠道，提升用户体验的同时，便于商业数据分析。

### 短URL设计

为长`URL`生成不重复的短`URL`，使用62进制编码，使用0-9, A-Z, a-Z表示0-61。为保持短`URL`的简洁性，假定短`URL`长度为6，此时可以生成$62^6 \approx 5.68\times10^{10}$，约568亿个短`URL`。虽然可支持的`URL`数据已经足够多，但考虑到长`URL`中可以包含查询字符串和路径参数，导致长`URL`本身可变范围较大，需要的短`URL`规模也相应地增大，同时短`URL`具有一定的时效性，所以短`URL`需要支持过期回收重用。



### 实时哈希生成

长` URL` 利用 `MD5` 或者 `SHA256` 等单项散列算法，得到 128bit 或者 256bit 的 Hash 值。然后对该 Hash 值进行 Base64 编码，得到 22 个或者 43 个 Base64 字符，再根据某种规则得到 6 个字符作为短` URL` 。

该方式可能会发生 Hash 冲突，即不同的长 `URL`，计算得到相同的短` URL`。对此需要事先校验该短` URL` 是否已经映射为其他的长 `URL`，如果是，则需要重新计算得到新的短 `URL`， 再重复上述查重过程，直至得到不重复的短`URL`。该方法需要在生成短`URL`时，实时查重验证并重试，存在性能瓶颈。



### 离线生成缓存

离线预先生成全部可选的6位短`URL`，获取短`URL`时直接返回预先生成的缓存结果。假设需要生成`n=20000000000`个短`URL`，可以通过生成200亿个不重复的1到`q=40000000000`范围内的随机数，并将其转换为62进制编码，得到200亿个短`URL`，同时需要保证生成的随机数序列乱序，防止短`URL`被外界预测。离线生成短`URL`可以有下面两种方式：



##### 重复随机生成+布隆过滤器去重

* 每次生成一个`[1,p]`之间的随机数`r`，并通过布隆过滤器查询`r`是否已经生成过，如若未被生成过，`r`为有效随机数，转换为62进制编码得到短`url`，并将`r`加入布隆过滤器；如若布隆过滤器反馈`r`已经存在，则尝试重新生成随机数，重复上述过程，直到生成`n`个不重复短`URL`。

  ```java
  public List<String> bloomFilterApproach() {
      // 布隆过滤器的预期插入量为n，误报率为p
      BloomFilter<Long> bloomFilter = BloomFilter.create(Funnels.longFunnel(), n, p);
      Random random = new Random();
      List<String> urls = new LinkedList<>();
      int count = 0;
      while (count < n) {
          // [1,q]间随机数
          long r = (long) (random.nextDouble() * q + 1);
          // 判重
          if (!bloomFilter.mightContain(r)) {
              bloomFilter.put(r);
              String url = long2Base62String(r);
              urls.add(url);
              count++;
          }
      }
      return urls;
  }
  ```

  该方法生成的随机数间完全独立，保证了生成随机序列`{r}`的无序性，保证了随机数在`[1,p]`间均匀分布；同时利用布隆过滤器保证了随机序列中无重复值。

  

* 时间与空间复杂度

  在生成第`i`(`i`从0开始)个随机数时，生成的随机数在之前未出现过的概率是$\frac{q-i}{q}$，则生成第`i`个不重复随机数的尝试次数的期望为$\frac{q}{q-i}$，所以生成`n`个不重复随机数一共需要尝试的次数是
  $$
  \sum_{i=0}^{n-1}\frac{q}{q-i}
  $$
  

  $\frac{q-i}{q}$次尝试中需要$\frac{q-i}{q}$次通过布隆过滤器判断随机数是否存在，外加1次将生成的随机数加入布隆过滤器。

  对于布隆过滤器，假设其误判概率为`p`，插入数据规模为`n`，用于记录数据的比特数组长度为`m`，hash函数个数为k。则在插入`n`个数据后，比特数组的任一下标处仍然为0的概率为$(1-\frac{1}{m})^{kn}$，则任一下标处为1的概率是

  $$
  1-(1-\frac{1}{m})^{kn}
  $$
  
  
  此时判断某个数`x`是否存在时，如果产生误判，即`x`经过`k`个hash函数指向的`k`位置都被置为1的概率为
  $$
  \begin{align}
  p&=[1-(1-\frac{1}{m})^{kn}]^k\\
  &=[1-((1-\frac{1}{m})^m)^{kn/m}]^k\\
  &\approx[1-(e^{-1})^{kn/m}]^k\\
  &=[1-e^{-kn/m}]^k
  \end{align}\\
  $$
  
  
  将$n/m$视为常数，`p`随着`k`的增大，先减小再增大。使得`p`最小的$k=\frac{mln2}{n}$，带入误判率公式，得到
  
  需要的比特数组长度$m=-\frac {n\ln(p)}{\ln^2(2)}$，哈希函数个数$ k=-\frac{ln(p)}{ln(2)}$。
  
  
  
  每次判断随机数是否存在以及将随机数加入布隆过滤器都需要进行`k`次哈希运算，所以总的时间复杂度为
  $$
  \begin{align}
  O(n)&=O(\sum_{i=0}^{n-1}\frac{qk}{q-i}+k)\\
  &=O(\sum_{i=0}^{n-1}\frac{(i-2q)ln(p)}{(q-i)ln(2)})\\
  &=O(\ln(p)\sum_{i=0}^{n-1}\frac{(i-2q)}{(q-i)})\\
  \end{align}\\
  $$
  空间复杂度与比特数组长度相关，空间复杂度为$O(m)=O(-{\frac {n\ln(p)}{\ln(2)^{2}}})=O(-n\ln(p))$

##### 递增序列+随机乱序

* 在`[1, p]`间生成`n`个不重复的随机数，如果将得到的随机数序列`{x}`进行排序，则两相邻随机数间间隔的期望为$p/n$。当前生成随机序列的方法中：

  --> 首先定义第一个随机数$x_0=000001$，得到第一个短`URL`，

  -->生成$[1, 2p/n-1]$间的随机递增值$r_1$，在62进制下进行加法，第二个随机数$x_1=x_0+r_1$，得到第二个短`URL`，

  -->生成$[1, 2p/n-1]$间的随机递增值$r_2$，第三个随机数$x_2=x_1+r_2$，得到第三个短`URL`，

  以此类推，直到生成全部的随机数序列${x}=\{x_0,x_1,...,x_{n-1}\}$。

  ```python
  urls = []
  curr = "000001"
  for i from 0 up to n-1:
      urls.append(curr)
      r ← random integer such that 1 ≤ j < 2 * p/n 
      curr = base62_plus(curr, r)
  
  ```

  

  对于随机递增值$r_i$，在$[1,2p/n-1]$间均匀分布，其数学期望为$E(r)=p/n$，保证了随机序列$\{x\}$在`[1, p]`内均匀分布。并且通过递增方式生成的序列保证了随机数间无重复，不需要判重处理。

  最后使用`Fisher-Yates shuffle`[^1]洗牌算法将序列$\{x\}$打乱顺序，保证了数据的乱序性质。

  ```python
  for i from n−1 down to 1 do
       j ← random integer such that 0 ≤ j ≤ i
       exchange a[j] and a[i]
  ```

  

  注意：由于`Fisher-Yates shuffle`要求集合支持随机下标访问，所以需要使用数组保存数据，但是由于数据量超过数组的长度限制，所以需要将随机序列$\{x\}$分散保存在多个数组中，相应的`Fisher-Yates shuffle`也需要做出变化。

  ```java
  
  private List<List<String>> randomIncreaseApproach() {
      // 每段数据长度
      int len = Integer.MAX_VALUE / 1024;
      // 数据段数
      int m = (int) Math.ceil(1.0 * n / len);
      List<List<String>> urls = new ArrayList<>(m);
      for (int i = 0; i < m; ++i) {
          urls.add(new ArrayList<>(len));
      }
      Random random = new Random();
      // 初始值
      StringBuilder builder = new StringBuilder();
      for (int i = 0; i < l - 1; i++) {
          builder.append('0');
      }
      builder.append('1');
      String curr = builder.toString();
  
      for (long i = 0; i < n; i++) {
          urls.get((int) (i / len)).add(curr);
          // [1,2b] 间随机数
          int r = 1 + random.nextInt(2 * b - 1);
          // 62进制加法
          curr = base62Plus(curr, r);
      }
      // 打乱顺序
      fisherYatesShuffle(urls, random, n, len);
      return urls;
  }
  
  public static void fisherYatesShuffle(List<List<String>> urls, Random random, long n, int l) {
      random.nextLong();
      for (long i = n - 1; i > 0; i--) {
          // [0,i] 间随机数
          long j = (long) (random.nextDouble() * (i + 1));
          // 确定i和j分别属于哪一段
          int segmentI = (int) (i / l);
          int indexI = (int) (i % l);
          int segmentJ = (int) (j / l);
          int indexJ = (int) (j % l);
          // 交换元素
          String temp = urls.get(segmentI).get(indexI);
          urls.get(segmentI).set(indexI, urls.get(segmentJ).get(indexJ));
          urls.get(segmentJ).set(indexJ, temp);
      }
  }
  ```

  完整代码将附录小结。

* 时间复杂度：$O(n)$

  空间复杂度：$O(1)$

##### 验证

* 时间：递增序列+随机乱序方法耗时约为重复随机生成+布隆过滤器去重方法的四分之一。

* 空间：递增序列+随机乱序方法无需额外空间，重复随机生成+布隆过滤器去重方法需要额外$-{\frac {n\ln(p)}{\ln(2)^{2}}}$比特空间。

* 分布是否均匀：使用卡方检验[^2]检测随机序列分布是否在`[0, p]`内均匀分布。重复随机生成+布隆过滤器去重方法的`p-value`为0.96；递增序列+随机乱序方法的`p-value`为1.0，均满足随机分布假设。

* 分布顺序是否随机：使用排列测试[^3]检测随机序内部数据顺序是否随机。重复随机生成+布隆过滤器去重方法和递增序列+随机乱序方法的`p-value`都为1.0，均满足随机排列假设。

### 附录

```java
import com.google.common.hash.BloomFilter;
import com.google.common.hash.Funnels;

import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;
import java.util.*;

public class RandomGenerator {
    // 短url长度
    private static final int l = 3;
    // 数据范围[1, q]
    private static final long q = (long) Math.pow(62, l);
    // 预期插入量
    private static final long n = q / 10;
    // 误报率
    private static final double p = 0.01;
    // 随机数间间隔期望
    private static final int b = (int) (q / n);


    private static final Map<Character, Integer> CHAR_TO_INT_MAP = new HashMap<>();
    private static final Map<Integer, Character> INT_TO_CHAR_MAP = new HashMap<>();

    static {
        String base62 = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz";
        for (int i = 0; i < base62.length(); i++) {
            char c = base62.charAt(i);
            CHAR_TO_INT_MAP.put(c, i);
            INT_TO_CHAR_MAP.put(i, c);
        }
    }

    public static void main(String[] args) throws IOException {
        RandomGenerator generator = new RandomGenerator();
        // warm up+jit
        for (int i = 0; i < 32; ++i) {
            generator.bloomFilterApproach();
            generator.randomIncreaseApproach();
        }

        long totalTime = 0;
        int iterations = 32;
        List<String> bloomFilterUrls = null;
        for (int i = 0; i < iterations; i++) {
            System.gc();
            long startTime = System.nanoTime();
            bloomFilterUrls = generator.bloomFilterApproach();
            long endTime = System.nanoTime();
            totalTime += (endTime - startTime);
        }
        long averageTime = totalTime / iterations;
        System.out.println("bf:" + averageTime / 1000 + "ms");

        // write to .txt
        try (BufferedWriter writer = new BufferedWriter(new FileWriter("./out/bloomFilterUrls.txt"))) {
            for (String url : bloomFilterUrls) {
                writer.write(url);
                writer.newLine();
            }
        }

        totalTime = 0;
        List<List<String>> randomIncreaseUrls = null;
        for (int i = 0; i < iterations; i++) {
            System.gc();
            long startTime = System.nanoTime();
            randomIncreaseUrls = generator.randomIncreaseApproach();
            long endTime = System.nanoTime();
            totalTime += (endTime - startTime);
        }
        averageTime = totalTime / iterations;
        System.out.println("ri:" + averageTime / 1000 + "ms");


        // write to .txt
        try (BufferedWriter writer = new BufferedWriter(new FileWriter("./out/randomIncreaseUrls.txt"))) {
            for (List<String> urls : randomIncreaseUrls) {
                for (String url : urls) {
                    writer.write(url);
                    writer.newLine();
                }
            }
        }
    }

    public List<String> bloomFilterApproach() {
        // 布隆过滤器的预期插入量为n，误报率为p
        BloomFilter<Long> bloomFilter = BloomFilter.create(Funnels.longFunnel(), n, p);
        Random random = new Random();
        List<String> urls = new LinkedList<>();
        int count = 0;
        while (count < n) {
            // [1,q]间随机数
            long r = (long) (random.nextDouble() * q + 1);
            // 判重
            if (!bloomFilter.mightContain(r)) {
                bloomFilter.put(r);
                String url = long2Base62String(r);
                urls.add(url);
                count++;
            }
        }
        return urls;
    }

    private static String long2Base62String(long value) {
        StringBuilder builder = new StringBuilder();
        for (int i = 0; i < l; i++) {
            builder.append('0');
        }
        for (int i = l - 1; i >= 0 && value > 0; i--) {
            builder.setCharAt(i, INT_TO_CHAR_MAP.get((int) (value % 62)));
            value /= 62;
        }
        return builder.toString();
    }

    private List<List<String>> randomIncreaseApproach() {
        // 每段数据长度
        int len = Integer.MAX_VALUE / 1024;
        // 数据段数
        int m = (int) Math.ceil(1.0 * n / len);
        List<List<String>> urls = new ArrayList<>(m);
        for (int i = 0; i < m; ++i) {
            urls.add(new ArrayList<>(len));
        }
        Random random = new Random();
        // 初始值
        StringBuilder builder = new StringBuilder();
        for (int i = 0; i < l - 1; i++) {
            builder.append('0');
        }
        builder.append('1');
        String curr = builder.toString();

        for (long i = 0; i < n; i++) {
            urls.get((int) (i / len)).add(curr);
            // [1,2b] 间随机数
            int r = 1 + random.nextInt(2 * b - 1);
            // 62进制加法
            curr = base62Plus(curr, r);
        }
        // 打乱顺序
        fisherYatesShuffle(urls, random, n, len);
        return urls;
    }

    public static String base62Plus(String curr, int r) {
        StringBuilder result = new StringBuilder(curr);
        // 初始化进位为r
        int carry = r;
        for (int i = l - 1; i >= 0 && carry > 0; i--) {
            // 计算当前位的和
            int sum = CHAR_TO_INT_MAP.get(curr.charAt(i)) + carry;
            // 更新当前位
            result.setCharAt(i, INT_TO_CHAR_MAP.get(sum % 62));
            // 更新进位
            carry = sum / 62;
        }
        return result.toString();
    }

    public static void fisherYatesShuffle(List<List<String>> urls, Random random, long n, int l) {
        random.nextLong();
        for (long i = n - 1; i > 0; i--) {
            // [0,i] 间随机数
            long j = (long) (random.nextDouble() * (i + 1));
            // 确定i和j分别属于哪一段
            int segmentI = (int) (i / l);
            int indexI = (int) (i % l);
            int segmentJ = (int) (j / l);
            int indexJ = (int) (j % l);
            // 交换元素
            String temp = urls.get(segmentI).get(indexI);
            urls.get(segmentI).set(indexI, urls.get(segmentJ).get(indexJ));
            urls.get(segmentJ).set(indexJ, temp);
        }
    }
}


```

```python
from scipy.stats import rankdata
from scipy.stats import norm
import numpy as np
from scipy.stats import chisquare
import numpy as np


def chi_squared_test(sequence, m, bound):
    n = len(sequence)
    # 每个区间的期望出现次数
    expected = n / m 

    # 划分区间并计算每个区间的实际出现次数
    counts = np.histogram(sequence, bins=m, range=(0, bound))[0]

    chi_squared_stat, p_value = chisquare(counts, expected * np.ones(m))

    return chi_squared_stat, p_value


def permutation_test(sequence, num_permutations=10000):
    n = len(sequence)
    original_rank_sum = sum(rankdata(sequence))

    count = 0
    for _ in range(num_permutations):
        permuted_sequence = np.random.permutation(sequence)
        permuted_rank_sum = sum(rankdata(permuted_sequence))

        if permuted_rank_sum >= original_rank_sum:
            count += 1

    p_value = (count + 1) / (num_permutations + 1)
    return p_value


def base62_to_decimal(base62):
    decimal = 0
    base = 62
    for i, char in enumerate(base62):
        if '0' <= char <= '9':
            value = ord(char) - ord('0')
        elif 'A' <= char <= 'Z':
            value = ord(char) - ord('A') + 10
        elif 'a' <= char <= 'z':
            value = ord(char) - ord('a') + 36
        else:
            raise ValueError(
                "Invalid character in base62 string: {}".format(char))
        decimal = decimal*base + value
    return decimal


if __name__ == '__main__':
    bound = 62**3
    sequence = []
    with open('./out/bloomFilterUrls.txt', mode='r', encoding='utf8') as f:
        for line in f.readlines():
            sequence.append(base62_to_decimal(line[:-1]))
    m = 1024  
    chi_squared_stat, p = chi_squared_test(sequence, m, bound)
    # p较小(0.05、0.01 和 0.001)，则表明序列的分布与均匀分布显著不同
    print(f"P-value: {p}")

    p = permutation_test(sequence)
    print(f"P-value: {p}")

    sequence = []
    with open('./out/randomIncreaseUrls.txt', mode='r', encoding='utf8') as f:
        for line in f.readlines():
            sequence.append(base62_to_decimal(line[:-1]))
    m = 1024  
    chi_squared_stat, p = chi_squared_test(sequence, m, bound)
    # p较小(0.05、0.01 和 0.001)，则表明序列的分布与均匀分布显著不同
    print(f"P-value: {p}")

    p = permutation_test(sequence)
    print(f"P-value: {p}")

```




### 参考
[^1]: [Fisher-Yates shuffle]( https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle)
[^2]: [卡方检验](https://en.wikipedia.org/wiki/Chi-squared_test)
[^3]: [排列测试](https://en.wikipedia.org/wiki/Permutation_test)



