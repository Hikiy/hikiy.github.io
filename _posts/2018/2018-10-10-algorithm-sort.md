---
layout: post
title:  "算法：排序算法"
date:   2018-10-10 16:00:00 +0200
categories: 算法
excerpt: 
tagg: algorithm
---

# 排序算法

不稳定的算法：

> 选择排序、希尔排序、堆排序、快速排序

稳定的算法：

> 冒泡排序、插入排序、归并排序

## 1.冒泡排序O(n^2)

> 
> // 分类 -------------- 内部比较排序  
> // 数据结构 ---------- 数组  
> // 最差时间复杂度 ---- O(n^2)  
> // 最优时间复杂度 ---- 如果能在内部循环第一次运行时,使用一个旗标来表示有无需要交换的可能,可以把最优时间复杂度降低到O(n)  
> // 平均时间复杂度 ---- **O(n^2)**  
> // 所需辅助空间 ------ **O(1)**  
> // 稳定性 ------------ **稳定**

### 步骤

1.比较相邻的元素，如果前一个比后一个大，就把它们两个调换位置。  
2.对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。这步做完后，最后的元素会是最大的数。  
3.针对所有的元素重复以上的步骤，除了最后一个。  
4.持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。  

### 代码示例

```
void Swap(int A[], int i, int j)
{
    int temp = A[i];
    A[i] = A[j];
    A[j] = temp;
}

void BubbleSort(int A[], int n)
{
    for (int j = 0; j < n - 1; j++)         // 每次最大元素就像气泡一样"浮"到数组的最后
    {
        for (int i = 0; i < n - 1 - j; i++) // 依次比较相邻的两个元素,使较大的那个向后移
        {
            if (A[i] > A[i + 1])            // 如果条件改成A[i] >= A[i + 1],则变为不稳定的排序算法
            {
                Swap(A, i, i + 1);
            }
        }
    }
}
```

## 2.选择排序O(n^2)

> // 分类 -------------- 内部比较排序  
> // 数据结构 ---------- 数组  
> // 最差时间复杂度 ---- O(n^2)  
> // 最优时间复杂度 ---- O(n^2)  
> // 平均时间复杂度 ---- **O(n^2)**  
> // 所需辅助空间 ------ **O(1)**  
> // 稳定性 ------------ **不稳定**    

### 步骤

遍历一次，将最小的放在第一位，然后遍历后面的n-1位，以此类推

1.初始时在序列中找到最小（大）元素，放到序列的起始位置作为已排序序列  
2.从剩余未排序元素中继续寻找最小（大）元素，放到已排序序列的末尾。  
3.以此类推，直到所有元素均排序完毕。

### 选择排序与冒泡排序的区别

冒泡排序通过依次交换相邻两个顺序不合法的元素位置，从而将当前最小（大）元素放到合适的位置；而选择排序每遍历一次都记住了当前最小（大）元素的位置，最后**仅需一次交换**操作即可将其放到合适的位置。

### 为什么是不稳定的算法

不稳定发生在最小元素与A[i]交换的时刻。
比如序列： `{ 5, 8, 5, 2, 9 }` ，一次选择的最小元素是`2`，然后把 `2` 和第一个 `5` 进行交换，从而改变了两个元素5的相对次序。

### 代码示例

```
void Swap(int A[], int i, int j)
{
    int temp = A[i];
    A[i] = A[j];
    A[j] = temp;
}

void SelectionSort(int A[], int n)
{
    for (int i = 0; i < n - 1; i++)         // i为已排序序列的末尾
    {
        int min = i;
        for (int j = i + 1; j < n; j++)     // 未排序序列
        {
            if (A[j] < A[min])              // 找出未排序序列中的最小值
            {
                min = j;
            }
        }
        if (min != i)
        {
            Swap(A, min, i);    // 放到已排序序列的末尾，该操作很有可能把稳定性打乱，所以选择排序是不稳定的排序算法
        }
    }
}
```

## 3.插入排序O(n^2)

> // 分类 ------------- 内部比较排序  
// 数据结构 ---------- 数组  
// 最差时间复杂度 ---- 最坏情况为输入序列是降序排列的,此时时间复杂度O(n^2)  
// 最优时间复杂度 ---- 最好情况为输入序列是升序排列的,此时时间复杂度O(n)  
// 平均时间复杂度 ---- **O(n^2)**  
// 所需辅助空间 ------ **O(1)**  
// 稳定性 ------------ **稳定**  


### 步骤

1.从第一个元素开始，该元素可以认为已经被排序  
2.取出下一个元素，在已经排序的元素序列中从后向前扫描  
3.如果该元素（已排序）大于新元素，将该元素移到下一位置  
4.重复步骤3，直到找到已排序的元素小于或者等于新元素的位置  
5.将新元素插入到该位置后  
6.重复步骤2~5  

### 代码示例

```
void InsertionSort(int A[], int n)
{
    for (int i = 1; i < n; i++)         // 类似抓扑克牌排序
    {
        int get = A[i];                 // 右手抓到一张扑克牌
        int j = i - 1;                  // 拿在左手上的牌总是排序好的
        while (j >= 0 && A[j] > get)    // 将抓到的牌与手牌从右向左进行比较
        {
            A[j + 1] = A[j];            // 如果该手牌比抓到的牌大，就将其右移
            j--;
        }
        A[j + 1] = get; // 直到该手牌比抓到的牌小(或二者相等)，将抓到的牌插入到该手牌右边(相等元素的相对次序未变，所以插入排序是稳定的)
    }
}
```

### 使用二分法改进

因为左边有排序好的数组，在进行比较的时候，可以使用二分法进行比较减少比较次数

```
void InsertionSortDichotomy(int A[], int n)
{
    for (int i = 1; i < n; i++)
    {
        int get = A[i];                    // 右手抓到一张扑克牌
        int left = 0;                    // 拿在左手上的牌总是排序好的，所以可以用二分法
        int right = i - 1;                // 手牌左右边界进行初始化
        while (left <= right)            // 采用二分法定位新牌的位置
        {
            int mid = (left + right) / 2;
            if (A[mid] > get)
                right = mid - 1;
            else
                left = mid + 1;
        }
        for (int j = i - 1; j >= left; j--)    // 将欲插入新牌位置右边的牌整体向右移动一个单位
        {
            A[j + 1] = A[j];
        }
        A[left] = get;                    // 将抓到的牌插入手牌
    }
}
```

## 4.希尔排序

> // 分类 -------------- 内部比较排序  
// 数据结构 ---------- 数组  
// 最差时间复杂度 ---- 根据步长序列的不同而不同。已知最好的为O(n(logn)^2)  
// 最优时间复杂度 ---- O(n)  
// 平均时间复杂度 ---- 根据步长序列的不同而不同。  
// 所需辅助空间 ------ O(1)  
// 稳定性 ------------ **不稳定**  

其实就是减少移动操作的插入排序，也叫递减增量排序

### 步骤

- 1.设定一个初始增量，建议用细而增量即 `gap = length/2`
- 2.将数组分为好几组，例如 `gap = 5`  则数组被分为了5组
- 3.每组进行直接插入排序
- 4.增量变为 `gap = gap/2`
- 5.重复步骤3-4，直到增量为1

### 代码示例

```
    void ShellSort(int A[], int n)
    /**
      * 希尔排序 针对有序序列在插入时采用交换法
      * @param arr
      */
     public static void sort(int []arr){
         //增量gap，并逐步缩小增量
        for(int gap=arr.length/2;gap>0;gap/=2){
            //从第gap个元素，逐个对其所在组进行直接插入排序操作
            for(int i=gap;i<arr.length;i++){
                int j = i;
                while(j-gap>=0 && arr[j]<arr[j-gap]){
                    //插入排序采用交换法
                    swap(arr,j,j-gap);
                    j-=gap;
                }
            }
        }
     }
 
     /**
      * 希尔排序 针对有序序列在插入时采用移动法。
      * @param arr
      */
     public static void sort1(int []arr){
         //增量gap，并逐步缩小增量
         for(int gap=arr.length/2;gap>0;gap/=2){
             //从第gap个元素，逐个对其所在组进行直接插入排序操作
             for(int i=gap;i<arr.length;i++){
                 int j = i;
                 int temp = arr[j];
                 if(arr[j]<arr[j-gap]){
                     while(j-gap>=0 && temp<arr[j-gap]){
                         //移动法
                         arr[j] = arr[j-gap];
                         j-=gap;
                     }
                     arr[j] = temp;
                 }
             }
         }
     }
```

## 5.归并排序O(nlogn)

> // 分类 -------------- 内部比较排序  
// 数据结构 ---------- 数组  
// 最差时间复杂度 ---- O(nlogn)  
// 最优时间复杂度 ---- O(nlogn)  
// 平均时间复杂度 ---- **O(nlogn)**  
// 所需辅助空间 ------ **O(n)**  
// 稳定性 ------------ **稳定**

归并排序的实现分为递归实现与非递归(迭代)实现。递归实现的归并排序是算法设计中分治策略的典型应用，我们将一个大问题分割成小问题分别解决，然后用所有小问题的答案来解决整个大问题。非递归(迭代)实现的归并排序首先进行是两两归并，然后四四归并，然后是八八归并，一直下去直到归并了整个数组。

### 步骤

- 1.申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列
- 2.设定两个指针，最初位置分别为两个已经排序序列的起始位置
- 3.比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置
- 4.重复步骤3直到某一指针到达序列尾
- 5.将另一序列剩下的所有元素直接复制到合并序列尾

![](https://hiki-blog.oss-cn-shenzhen.aliyuncs.com/githubio/algorithm/sort/sort_guibing.png)

### 代码示例

```
    public static void sort(int []arr){
        int []temp = new int[arr.length];//在排序前，先建好一个长度等于原数组长度的临时数组，避免递归中频繁开辟空间
        sort(arr,0,arr.length-1,temp);
    }
    private static void sort(int[] arr,int left,int right,int []temp){
        if(left<right){
            int mid = (left+right)/2;
            sort(arr,left,mid,temp);//左边归并排序，使得左子序列有序
            sort(arr,mid+1,right,temp);//右边归并排序，使得右子序列有序
            merge(arr,left,mid,right,temp);//将两个有序子数组合并操作
        }
    }
    private static void merge(int[] arr,int left,int mid,int right,int[] temp){
        int i = left;//左序列指针
        int j = mid+1;//右序列指针
        int t = 0;//临时数组指针
        while (i<=mid && j<=right){
            if(arr[i]<=arr[j]){
                temp[t++] = arr[i++];
            }else {
                temp[t++] = arr[j++];
            }
        }
        while(i<=mid){//将左边剩余元素填充进temp中
            temp[t++] = arr[i++];
        }
        while(j<=right){//将右序列剩余元素填充进temp中
            temp[t++] = arr[j++];
        }
        t = 0;
        //将temp中的元素全部拷贝到原数组中
        while(left <= right){
            arr[left++] = temp[t++];
        }
    }
    
void MergeSortIteration(int A[], int len)    // 非递归(迭代)实现的归并排序(自底向上)
{
    int left, mid, right;// 子数组索引,前一个为A[left...mid]，后一个子数组为A[mid+1...right]
    for (int i = 1; i < len; i *= 2)        // 子数组的大小i初始为1，每轮翻倍
    {
        left = 0;
        while (left + i < len)              // 后一个子数组存在(需要归并)
        {
            mid = left + i - 1;
            right = mid + i < len ? mid + i : len - 1;// 后一个子数组大小可能不够
            Merge(A, left, mid, right);
            left = right + 1;               // 前一个子数组索引向后移动
        }
    }
}
```

## 6.堆排序O(nlogn)

> // 分类 -------------- 内部比较排序  
// 数据结构 ---------- 数组  
// 最差时间复杂度 ---- O(nlogn)  
// 最优时间复杂度 ---- O(nlogn)  
// 平均时间复杂度 ---- **O(nlogn)**  
// 所需辅助空间 ------ **O(1)**  
// 稳定性 ------------ **不稳定**  

堆是具有以下性质的完全二叉树：

每个结点的值都大于或等于其左右孩子结点的值，称为**大顶堆**

每个结点的值都小于或等于其左右孩子结点的值，称为**小顶堆**

### 步骤

将初始堆构造为大顶堆（升序）或小顶堆（降序）：

- 1.找到第一个非叶子节点：length/2 -1
- 2.从左至右，从下至上进行调整。
- 3.子根如果混乱则调整

![](https://hiki-blog.oss-cn-shenzhen.aliyuncs.com/githubio/algorithm/sort/sort_heapSort1.png)

![](https://hiki-blog.oss-cn-shenzhen.aliyuncs.com/githubio/algorithm/sort/sort_heapSort2.png)

![](https://hiki-blog.oss-cn-shenzhen.aliyuncs.com/githubio/algorithm/sort/sort_heapSort3.png)

![](https://hiki-blog.oss-cn-shenzhen.aliyuncs.com/githubio/algorithm/sort/sort_heapSort4.png)

![](https://hiki-blog.oss-cn-shenzhen.aliyuncs.com/githubio/algorithm/sort/sort_heapSort5.png)

![](https://hiki-blog.oss-cn-shenzhen.aliyuncs.com/githubio/algorithm/sort/sort_heapSort6.png)

![](https://hiki-blog.oss-cn-shenzhen.aliyuncs.com/githubio/algorithm/sort/sort_heapSort7.png)

### 代码示例

```
    public static void sort(int []arr){
        //1.构建大顶堆
        for(int i=arr.length/2-1;i>=0;i--){
            //从第一个非叶子结点从下至上，从右至左调整结构
            adjustHeap(arr,i,arr.length);
        }
        //2.调整堆结构+交换堆顶元素与末尾元素
        for(int j=arr.length-1;j>0;j--){
            swap(arr,0,j);//将堆顶元素与末尾元素进行交换
            adjustHeap(arr,0,j);//重新对堆进行调整
        }

    }

    /**
     * 调整大顶堆（仅是调整过程，建立在大顶堆已构建的基础上）
     * @param arr
     * @param i
     * @param length
     */
    public static void adjustHeap(int []arr,int i,int length){
        int temp = arr[i];//先取出当前元素i
        for(int k=i*2+1;k<length;k=k*2+1){//从i结点的左子结点开始，也就是2i+1处开始
            if(k+1<length && arr[k]<arr[k+1]){//如果左子结点小于右子结点，k指向右子结点
                k++;
            }
            if(arr[k] >temp){//如果子节点大于父节点，将子节点值赋给父节点（不用进行交换）
                arr[i] = arr[k];
                i = k;
            }else{
                break;
            }
        }
        arr[i] = temp;//将temp值放到最终的位置
    }
```

## 7.快速排序O(nlogn)

> // 分类 ------------ 内部比较排序  
// 数据结构 --------- 数组  
// 最差时间复杂度 ---- 每次选取的基准都是最大（或最小）的元素，导致每次只划分出了一个分区，需要进行n-1次划分才能结束递归，时间复杂度为O(n^2)  
// 最优时间复杂度 ---- 每次选取的基准都是中位数，这样每次都均匀的划分出两个分区，只需要logn次划分就能结束递归，时间复杂度为O(nlogn)  
// 平均时间复杂度 ---- **O(nlogn)**  
// 所需辅助空间 ------ 主要是递归造成的栈空间的使用(用来保存left和right等局部变量)，取决于递归树的深度，一般为**O(logn)**，最差为O(n)         
// 稳定性 ---------- **不稳定**

快速排序通常明显比其他O(nlogn)算法更快，因为它的内部循环可以在大部分的架构上很有效率地被实现出来。

### 步骤

- 1.从序列中挑出一个元素，作为"基准"(pivot).
- 2.把所有比基准值小的元素放在基准前面，所有比基准值大的元素放在基准的后面（相同的数可以到任一边），这个称为分区(partition)操作。
- 3.对每个分区递归地进行步骤1~2，递归的结束条件是序列的大小是0或1，这时整体已经被排好序了。

步骤二详细：

- 1.一般以第一个数为基准,也可以是最后一个
- 2.两个指针，一个指向头为low，一个指向尾为high
- 3.先从high开始，只要 high的值>基准 ，则high-1
- 4.如果 high的值<=基准 , 则将high的值赋值给low
- 5.开始判断low，只要 low的值<基准，则low+1
- 6.如果 low的值>=基准，则将low的值赋值给high
- 7.重复步骤3-6，直到low=high,将基准赋值给low
- 8.完成第一次快速排序，然后将low的前部分和后部分分别快速排序即可

### 代码示例

```
    public static void main(String[] args) {
		int[] arr = { 52, 63, 65, 96, 14, 22, 76, 2, 6, 7, 2, 0, -24, 35 };
		quickSort(arr, 0, arr.length - 1);
		System.out.println("排序后:");
		for (int i : arr) {
			System.out.println(i);
		}
	}

    private static void quickSort(int[] arr, int low, int high) {

		if (low < high) {
			// 找寻基准数据的正确索引
			int index = getIndex(arr, low, high);

			// 进行迭代对index之前和之后的数组进行相同的操作使整个数组变成有序
			quickSort(arr, low, index - 1);
			quickSort(arr, index + 1, high);
		}

	}

	private static int getIndex(int[] arr, int low, int high) {
		// 基准数据
		int tmp = arr[low];
		while (low < high) {
			// 当队尾的元素大于等于基准数据时,向前挪动high指针
			while (low < high && arr[high] >= tmp) {
				high--;
			}
			// 如果队尾元素小于tmp了,需要将其赋值给low
			arr[low] = arr[high];
			// 当队首元素小于等于tmp时,向前挪动low指针
			while (low < high && arr[low] <= tmp) {
				low++;
			}
			// 当队首元素大于tmp时,需要将其赋值给high
			arr[high] = arr[low];

		}
		// 跳出循环时low和high相等,此时的low或high就是tmp的正确索引位置
		// 由原理部分可以很清楚的知道low位置的值并不是tmp,所以需要将tmp赋值给arr[low]
		arr[low] = tmp;
		return low; // 返回tmp的正确位置
	}
```

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2018.10.10  
> 更新日期：2018.10.10

<center>(<font color=red size=2>转载文章请注明作者和出处 </font><a href="https://github.com/Hikiy">Hiki)</a></center>  