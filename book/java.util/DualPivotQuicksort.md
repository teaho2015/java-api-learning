# DualPivotQuicksort类

## 介绍

自java 1.7后，Arrays.sort排序不再使用quicksort实现，而是使用此类DualPivotQuicksort。

该类是双轴排序算法（Dual-Pivot Quicksort algorithm）的实现。提供出来的方法都是package-private的，设计出来主要就是给Arrays类使用的。  
作者是Vladimir Yaroslavskiy, Jon Bentley, Josh Bloch三位。

### 算法介绍

该算法是快排算法的变种，是俄罗斯开发者Vladimir Yaroslavskiy在2009发明的。
算法分析写在这里：  
[快速排序](../algorithm/quick_sort.md)的Dual-Pivot Quicksort。

## 代码参数分析


算法已经分析过了，所以具体流程代码这里不分析。

说说DualPivotQuicksort类一些重要属性含义。
比如，在排序时，某些时候采用插入排序和计数排序。

~~~
    /*
     * Tuning parameters.
     */

    /**
     * 在运行sort的开头运行该次数的判断，如果发现数组并非highly structured，即并非比较有序的，转而采用经典快排。
     */
    private static final int MAX_RUN_COUNT = 67;

    /**
     * 发现待排序数据有较多相等元素会直接采用双轴快排（Dual-Pivot Quicksort）。
     */
    private static final int MAX_RUN_LENGTH = 33;

    /**
     * 当数组长度小于该参数采用快排
     */
    private static final int QUICKSORT_THRESHOLD = 286;

    /**
     * 当数组的长度小于改参数，会采用插入排序
     */
    private static final int INSERTION_SORT_THRESHOLD = 47;

    /**
    如果字节数组的长度大于此参数，采用计数排序而非插入排序
     */
    private static final int COUNTING_SORT_THRESHOLD_FOR_BYTE = 29;

    /**
        如果short或者字符数组的长度大于此参数，采用计数排序而非快排
     */
    private static final int COUNTING_SORT_THRESHOLD_FOR_SHORT_OR_CHAR = 3200;


~~~


