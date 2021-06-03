# 快速排序


## 前言

快速排序（Quicksort），又称分区交换排序（partition-exchange sort）。由C. A. R. Hoare大佬在1960年提出。  
在平均状况下，排序`n`个项目要`O(n log n)`次比较。在最坏状况下则需要`O(n^2)次比较，但这种状况并不常见。
事实上，快速排序O(n log n)通常明显比其他算法更快，因为它的内部循环（inner loop）可以在大部分的架构上很有效率地达成。


## 算法介绍

快速排序使用分治法（Divide and conquer）策略来把一个序列（list）分为较小和较大的2个子序列，然后递归地排序两个子序列。

步骤为：
1. 挑选基准值：从数列中挑出一个元素，称为“基准”（pivot），
2. 分割：重新排序数列，所有比基准值小的元素摆放在基准前面，所有比基准值大的元素摆在基准后面（与基准值相等的数可以到任何一边）。在这个分割结束之后，对基准值的排序就已经完成，
3. 递归排序子序列：递归地将小于基准值元素的子序列和大于基准值元素的子序列排序。


快速排序是一种分治的排序算法。它将一个数组分成两个子数组,将两部分独立地排序。
快速排序和归并排序是互补的:归并排序将数组分成两个子数组分别排序,并将有序的子数组归并以将整个 数组排序;而快速排序将数组排序的方式则是当两个子数组都有序时整个数组也就自然有序了。
在第一种情况中,递归调用发生在处理整个数组之前;在第二种情况中,递归调用发生在处理整个数组之后。 
在归并排序中, 一个数组被等分为两半；在快速排序中, 切分(partition) 的位置取决于数组的内容。

该方法的关键在于切分,这个过程使得数组满足下面三个条件: 
* 对于某个j,a[j]已经排定; 
* a[1o]到a[j-1]中的所有元素都不大于a[j]; 
* a[j+1]到a[hi]中的所有元素都不小于a[j]。 
我们就是通过递归地调用切分来排序的。

快排切分的数据移动图：
![quick_sort_数据移动.jpg](quick_sort_数据移动.jpg)



## 快排实现

简单手写一下快排。

~~~

/**
 *
 * 快速排序算法实现
 *
 * 时间复杂度：O(n log n) <br/>
 * 空间复杂度: O(1)
 *
 * @author teaho2015@gmail.com
 */
public class Quicksort {


    public void sort(Comparable[] a) {
        sort(a, 0, a.length - 1);

    }

    public void sort(Comparable[] a, int lo, int hi) {
        if (hi <= lo) {
            return;
        }
        int j = partition(a, lo, hi);
        sort(a, lo, j-1);
        sort(a, j+1, hi);

    }

    public int partition(Comparable[] a, int lo, int hi) {
        //将数组切分为a[lo...i-1], a[i], a[i+1, hi]进行partition
        int i = lo, j = hi +1;

        Comparable v = a[lo];

        while (true) {
            while (a[++i].compareTo(v) < 0) if (i == hi) break;
            while (a[--j].compareTo(v) > 0) if (j == lo) break;

            if (i >= j) {
                break;
            }
            swap(a, i, j);
        }
        swap(a, lo, j);
        return j;
    }


    private void swap(Comparable[] nums, int index1, int index2) {
        Comparable tmp = nums[index1];
        nums[index1] = nums[index2];
        nums[index2] = tmp;
    }

    public static void main(String[] args) {
        {
            Integer[] arr = new Integer[]{12, 15, 6, 7, 9, 1, 5, 32};
            new Quicksort().sort(arr);
            System.out.println(Arrays.toString(arr));

        }

        {
            Integer[] arr = new Integer[]{1};
            new Quicksort().sort(arr);
            System.out.println(Arrays.toString(arr));

        }
    }

}

~~~

## 双轴快速排序算法（Dual-Pivot Quicksort algorithm）

该算法是快排算法的变种，是俄罗斯开发者Vladimir Yaroslavskiy在2009发明的。


### 算法思想

快排的核心是切分（partition），思想在于将大区间分割为小区间，分段排序。

双轴快排执行步骤：
![dual-pivot_quicksort.png](dual-pivot_quicksort.png)
1. 对于小数组（长度<47）使用插入排序
2. 选择两个轴（pivot）元素P1和P2。比如，我们可以将第一个元素a[left]作为P1而最后一个元素a[right]作为P2。
3. P1必须小于P2，否则他们互换，紧接着划分有如下部分：
   1. part I，即left+1到L-1的元素，该部分都小于P1
   2. part II，即L到K-1的元素，该部分元素大于或等于P1并且小于等P2
   3. part III，即G+1到right-1的元素，该部分元素大于P2
   4. part IV，即K到G的元素，该部分元素待检验大小。
4. 接着取part IV的下一元素a[k]与P1和P2进行比较后，将元素移动到对应的部分（part I、part II或part III）
5. 接着指针L、K、G作相应移动。
6. 当K ≤ G重复上述4、5步。
7. 轴元素P1与part I的最后元素交换位置，轴元素P2与part III的第一个元素交换位置
8. 对每个部分part I、part II 和part III递归地重复步骤 1-7。

#### 伪代码实现
~~~~
DUALPIVOTQUICKSORT(A,left,right) // sort A[left..right]
1 if right−left ≥ 1
    // Take outermost elements as pivots (replace by sampling)
2   p := min {A[left],A[right]}
3   q := max{A[left],A[right]}
4   ℓ := left +1; g := right −1; k := ℓ
5   while k ≤ g
6       if A[k] < p
7           Swap A[k] and A[ℓ]; ℓ := ℓ+1
8       else if A[k] ≥ q
9           while A[g] > q and k < g
10              g := g −1
11          end while
12          Swap A[k] and A[g]; g := g −1
13          if A[k] < p
14              Swap A[k] and A[ℓ]; ℓ := ℓ+1
15          end if
16      end if
17      k := k +1
18  end while
19  ℓ := ℓ−1; g := g +1
20  A[left] := A[ℓ]; A[ℓ] := p // p to final position
21  A[right] := A[g]; A[g] := q // q to final position
22  DUALPIVOTQUICKSORT(A, left , ℓ−1)
23  DUALPIVOTQUICKSORT(A, ℓ+1,g −1)
24  DUALPIVOTQUICKSORT(A,g +1,right)
25 end if


~~~~

### 性能

对多种快排变种的性能分析Sebastian Wild的硕士论文有分析到。
[Java 7’s Dual Pivot Quicksort|硕士论文|Sebastian Wild|Technische Universität|2013](https://kluedo.ub.uni-kl.de/frontdoor/deliver/index/docId/3463/file/wild-master-thesis.pdf)


![](quicksort_variant_performance_compare1.jpg)

![](quicksort_variant_performance_compare2.jpg)


在java中，我们看到Yaroslavskiy这一快排变种性能最优秀。  
同时，我们看到c++的STL introsort表现比Yaroslavskiy更优异些。（introsort当数据小时采用插入排序，而递归层数较大时会采用堆排序）





## Reference

[1][Java 7’s Dual Pivot Quicksort|硕士论文|Sebastian Wild|Technische Universität|2013](https://kluedo.ub.uni-kl.de/frontdoor/deliver/index/docId/3463/file/wild-master-thesis.pdf)  
[2][新的快速排序算法: 《Dual-Pivot QuickSort》阅读笔记](https://www.jianshu.com/p/2c6f79e8ce6e)   
[3][Why Is Dual-Pivot Quicksort Fast|Sebastian Wild|2016](https://arxiv.org/pdf/1511.01138.pdf)  
[4]Robert Sedgewick,Kevin Wayne.算法（第4版）.中国：人民邮电出版社,2012




