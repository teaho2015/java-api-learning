# 红黑二叉查找树

红黑二叉查找树，下简称红黑树。

摘自[《算法导论》](#references):

> 一棵红黑树是满足如下条件的二叉查找树（binary search tree）：
> 1. 每个节点要么是红色，要么是黑色。
> 2. 根节点是黑色的。
> 3. 每个叶节点是黑色的。
> 4. 如果一个节点是红色，则它的两个子节点都是黑色。
> 5. 对于每个节点，从该节点到其所有后代叶节点的简单路径，都含有相同数目的黑色节点。

红黑树也可以看作是具有2-3树性质的二叉查找树。一棵二叉查找树(BST)是一棵二叉树，其中每个结点都含有一个Comparable的键（以及相关联的值）
且每个结点的键都大于其左子树中的任意结点的键而小于右子树的任意结点的键。

那么，围绕上面的性质，在对红黑树进行插入、删除等操作时，会进行一些旋转、颜色变化等操作。

## 时间复杂度

在N次插入后，平均情况的查找为lgN，插入为lgN；最坏情况，查找为2lgN,插入为2lgN。

## 适用范围比较和应用场景

红黑树是一种接近于完美平衡的二叉搜索树。红黑树能保证在插入、删除等时调整在3次旋转内完成。红黑树的适用性挺广的，
红黑树可以保证在最坏的情况下基本动态集合操作的时间复杂度为O(lg n)。

应用场景有：nginx中，用红黑树管理timer等，Java中的TreeMap实现，Java8后的HashMap用于处理哈希碰撞后的存储实现等等。

与其他常用查找算法做些简单比较：

* AVL树

    保证完美平衡的树，但通常，维护这种高度平衡所付出的代价比从中获得的效率收益还大，
    但比起红黑树，如果所需场景中插入删除操作非常少，单就查找来说，AVL是优于红黑树的。

* 二分查找数组(有序数组)

    维护有序的代价比从中获得的效率收益还大，但最差情况下查找的复杂度为lgN（比起红黑树的2lgN稍好），
    所以，同AVL树，单就查找，是稍优于红黑的。

* 跳表(又称跳跃表，skipList)

    相对红黑，省内存，但性能会差一点点。应用场景：redis。
    如何是摘自[知乎 | 余佳晋的回答 | 为啥 redis 使用跳表(skiplist)而不是使用 red-black？][link:3]：

    > There are a few reasons:
    >
    > 1) They are not very memory intensive. It's up to you basically. Changing parameters about the probability of a node to have a given number of levels will make then less memory intensive than btrees.
    >
    > 2) A sorted set is often target of many ZRANGE or ZREVRANGE operations, that is, traversing the skip list as a linked list. With this operation the cache locality of skip lists is at least as good as with other kind of balanced trees.
    >
    > 3) They are simpler to implement, debug, and so forth. For instance thanks to the skip list simplicity I received a patch (already in Redis master) with augmented skip lists implementing ZRANK in O(log(N)). It required little changes to the code.
    >
    > About the Append Only durability & speed, I don't think it is a good idea to optimize Redis at cost of more code and more complexity for a use case that IMHO should be rare for the Redis target (fsync() at every command). Almost no one is using this feature even with ACID SQL databases, as the performance hint is big anyway.About threads: our experience shows that Redis is mostly I/O bound. I'm using threads to serve things from Virtual Memory. The long term solution to exploit all the cores, assuming your link is so fast that you can saturate a single core, is running multiple instances of Redis (no locks, almost fully scalable linearly with number of cores), and using the "Redis Cluster" solution that I plan to develop in the future.

    简单说下：

    1. 跳表相对来说并不内存密集。
    2. 一些范围查找操作体现在跳表中，至少是和其他平衡树实现性能上差不多的。
    3. 跳表更易于实现、debug等等。

* 哈希

    理想情况下，是比红黑要好的，但是，假如计算hashcode的算法太复杂的时候，
    或因为计算hashcode的算法的缺陷而导致频繁的哈希碰撞的时候，要慎重考虑。
    应用场景：Java中的HashMap集合等等。

* B/B+树

    主要用在文件系统以及数据库中做索引等。


## references

[1] Sedgewick.R,Wayne.K.算法,第四版[M].中国北京:人民邮电出版社,2012

[2] Thomas H. Cormen,Charles E. Leiserson,Ronald L. Rivest,Clifford Stein.算法导论,第三版[M].中国:机械工业出版社,2013

[3] [知乎 | 余佳晋的回答 | 为啥 redis 使用跳表(skiplist)而不是使用 red-black？][link:3]

[4] [Red–black tree](https://en.wikipedia.org/wiki/Red%E2%80%93black_tree)

[link:3]: https://www.zhihu.com/question/20202931/answer/30983057