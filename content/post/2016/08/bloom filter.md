+++
categories = ["技术"]
date = "2016-08-02T09:36:46-07:00"
draft = false
slug = ""
tags = ["bloom","tool"]
title = "bloom filter 教程  "

+++

## Bloom Filter概念和原理介绍  
```xml
    Bloom Filter是一种空间效率很高的随机数据结构，它利用位数组很简洁地表示一个集合，并能判断一个元素是否属于这个集合。  
    Bloom Filter的这种高效是有一定代价的： 在判断一个元素是否属于某个集合时，有可能会把不属于这个集合的元素误认为属于这个集合（false positive）。  
    因此，Bloom Filter不适合那些“零错误”的应用场合。 而在能容忍低错误率的应用场合下，Bloom Filter通过极少的错误换取了存储空间的极大节省。   
    
    为了说明Bloom Filter存在的重要意义，举一个实例：
　　假设要你写一个网络蜘蛛（web crawler）。由于网络间的链接错综复杂，蜘蛛在网络间爬行很可能会形成“环”。为了避免形成“环”，就需要知道蜘蛛已经访问过那些URL。给一个URL，怎样知道蜘蛛是否已经访问过呢？稍微想想，就会有如下几种方案：
　　1. 将访问过的URL保存到数据库。  

　　2. 用HashSet将访问过的URL保存起来。那只需接近O(1)的代价就可以查到一个URL是否被访问过了。  

　　3. URL经过MD5或SHA-1等单向哈希后再保存到HashSet或数据库。 

　　4. Bit-Map方法。建立一个BitSet，将每个URL经过一个哈希函数映射到某一位。  

　　方法1~3都是将访问过的URL完整保存，方法4则只标记URL的一个映射位。  
　　以上方法在数据量较小的情况下都能完美解决问题，但是当数据量变得非常庞大时问题就来了。  
　　方法1的缺点：数据量变得非常庞大后关系型数据库查询的效率会变得很低。而且每来一个URL就启动一次数据库查询是不是太小题大做了？  
　　方法2的缺点：太消耗内存。随着URL的增多，占用的内存会越来越多。就算只有1亿个URL，每个URL只算50个字符，就需要5GB内存。 
　　方法3：由于字符串经过MD5处理后的信息摘要长度只有128Bit，SHA-1处理后也只有160Bit，因此方法3比方法2节省了好几倍的内存。  
　　方法4消耗内存是相对较少的，但缺点是单一哈希函数发生冲突的概率太高。若要降低冲突发生的概率到1%，就要将BitSet的长度设置为URL个数的100倍。  
　　实质上上面的算法都忽略了一个重要的隐含条件：允许小概率的出错，不一定要100%准确！也就是说少量url实际上没有没网络蜘蛛访问，而将它们错判为已访问的代价是很小的——大不了少抓几个网页呗。  
  
    Bloom Filter的算法  

    下面引入本篇的主角——Bloom Filter。其实上面方法4的思想已经很接近Bloom Filter了。方法四的致命缺点是冲突概率高，为了降低冲突的概念，Bloom Filter使用了多个哈希函数，而不是一个。

   　Bloom Filter算法如下：

  　 创建一个m位BitSet，先将所有位初始化为0，然后选择k个不同的哈希函数。第i个哈希函数对字符串str哈希的结果记为h（i，str），且h（i，str）的范围是0到m-1 。
  
     (1) 加入字符串过程   
     下面是每个字符串处理的过程，首先是将字符串str“记录”到BitSet中的过程：

　　对于字符串str，分别计算h（1，str），h（2，str）…… h（k，str）。然后将BitSet的第h（1，str）、h（2，str）…… h（k，str）位设为1。
   这样就将字符串str映射到BitSet中的k个二进制位了.
     
     (2) 检查字符串是否存在的过程 

　　下面是检查字符串str是否被BitSet记录过的过程：

　　对于字符串str，分别计算h（1，str），h（2，str）…… h（k，str）。然后检查BitSet的第h（1，str）、h（2，str）…… h（k，str）位是否为1，若其中任何一位不为1则可以判定str一定没有被记录过。若全部位都是1，则“认为”字符串str存在。
 
　　若一个字符串对应的Bit不全为1，则可以肯定该字符串一定没有被Bloom Filter记录过。（这是显然的，因为字符串被记录过，其对应的二进制位肯定全部被设为1了）

　　但是若一个字符串对应的Bit全为1，实际上是不能100%的肯定该字符串被Bloom Filter记录过的。（因为有可能该字符串的所有位都刚好是被其他字符串所对应）这种将该字符串划分错的情况，称为false positive 。
   
    (3) 删除字符串过程 

   字符串加入了就被不能删除了，因为删除会影响到其他字符串。实在需要删除字符串的可以使用Counting bloomfilter(CBF)，这是一种基本Bloom Filter的变体，CBF将基本Bloom Filter每一个Bit改为一个计数器，这样就可以实现删除字符串的功能了。

　　Bloom Filter跟单哈希函数Bit-Map不同之处在于：Bloom Filter使用了k个哈希函数，每个字符串跟k个bit对应。从而降低了冲突的概率。
     
```
## Bloom Filter 参考文档 和 调优参数介绍  
点击这里 参考文档http://blog.csdn.net/jiaomeng/article/details/1495500   
Java 计算错误率的公式：  
```xml
       /**
     *
     * @param k  hash function numbers
     * @param vectorSize bloom filter size
     * @param elementSize element  size
     * @return false positive
     */
    public static double getErrorRatio(int k ,int vectorSize,int elementSize){

        //k = m/n *ln2;
        double e  = Math.pow((1-Math.exp(-k*(double)elementSize / (double)vectorSize)),k); //0.0019148543506468739
        System.out.println("E:"+e);

        return e;
    }
```  

## Bloom Filter 、Count bloom filter 对比结果
  
```
Bloom Filter 数组长度 | hash 函数个数 | test-data-size(原数据大小) | bloom filter 初始化大小（byte）/设置完数据后大小 （byte）| bloom filter add times （ms） | count bloom filter 初始化大小（byte）/设置完数据后大小（byte） | count bloom filter add times （ms）
------|-----------|------|------|-----------|----------|----
1<< 18|8|240w(33618kb)|32841/32841| 3390 |131099/131099 | 3348
1<< 20 |8|240w(33618kb)|131145/131145 | 3526 |524315/524315 | 3498
1<< 22|8|240w(33618kb)|524361/524361 | 4079 | 2097179/2097179 |3903
1<< 24|8|240w(33618kb)|2097225/2097225 | 4275 | 8388635/8388635 | 3944
1<< 26|8|240w(33618kb)|8388681/8388681 | 4507  | 33554459/33554459 | 3904
1<< 18|8|480W(67286kb)|32841/32841|  6461 |131099/131099 | 6072
1<< 20 |8|480W(67286kb)|131145/131145 | 6648 |524315/524315 | 7267
1<< 22|8|480W(67286kb)|524361/524361 | 7811 | 2097179/2097179 |7310
1<< 24|8|480W(67286kb)|2097225/2097225 | 8139 | 8388635/8388635 | 7609
1<< 26|8|480W(67286kb)|8388681/8388681 | 8492  | 33554459/33554459 | 9237
1<< 18|8|960w(134677kb)|32841/32841|  12506|131099/131099 | 11150
1<< 20 |8|960w(134677kb)|131145/131145 | 12654|524315/524315 | 12614
1<< 22|8|960w(134677kb)|524361/524361 | 14289| 2097179/2097179 |14361
1<< 24|8|960w(134677kb)|2097225/2097225 | 15655| 8388635/8388635 | 15287
1<< 26|8|960w(134677kb)|8388681/8388681 | 16191| 33554459/33554459 | 19161
```
结论：
  经过以上测试，得出结论，存储同样的数据 带计数的bloom filter 大约多占用普通bloom filter 4倍左右的空间，init add 时间上彼此没有太大差异。   

