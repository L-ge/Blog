1. 算法基本知识
- 稳定排序：如果 a 原本在 b 的前面，且 a == b，排序之后 a 仍然在 b 的前面，则为稳定排序。
- 非稳定排序：如果 a 原本在 b 的前面，且 a == b，排序之后 a 可能不在 b 的前面，则为非稳定排序。
- 原地排序：原地排序就是指在排序过程中不申请多余的存储空间，只利用原来存储待排数据的存储空间进行比较和交换的数据排序。
- 非原地排序：需要利用额外的数组来辅助排序。
- 时间复杂度：一个算法执行所消耗的时间。
- 空间复杂度：运行完一个算法所需的内存大小。

2. 十大排序

算法 | 最佳时间复杂度 | 平均时间复杂度 | 最差时间复杂度 | 最差空间复杂度
---|---|---|---|---
快速排序 | O(nlog(n)) | O(nlog(n)) | O(n^2^) | O(log(n))
归并排序 | O(nlog(n)) | O(nlog(n)) | O(nlog(n)) | O(n)
Timesort | O(n) | O(nlog(n)) | O(nlog(n)) | O(n)
堆排序 | O(nlog(n)) | O(nlog(n)) | O(nlog(n)) | O(1)
冒泡排序 | O(n) | O(n^2^) | O(n^2^) | O(1)
插入排序 | O(n) | O(n^2^) | O(n^2^) | O(1)
选择排序 | O(n^2^) | O(n^2^) | O(n^2^) | O(1)
希尔排序 | O(n) | O((nlog(n))^2^) | O((nlog(n))^2^) | O(1)
桶排序 | O(n+k) | O(n+k) | O(n^2^) | O(n)
基数排序 | O(nk) | O(nk) | O(nk) | O(n+k) 

3. 十大排序中的稳定排序
- 冒泡排序（bubble sort） — O(n2)
- 插入排序（insertion sort）— O(n2)
- 归并排序（merge sort）— O(n log n)

4. 十大排序中的非稳定排序（面试考察中一般问快排，选择，希尔，堆这几种非稳定排序）
- 选择排序（selection sort）— O(n2)
- 希尔排序（shell sort）— O(n log n)
- 堆排序（heapsort）— O(n log n)
- 快速排序（quicksort）— O(n log n)

5. 冒泡排序
- 冒泡排序就是把小的元素往前调或者把大的元素往后调，比较是相邻的两个元素比较，交换也发生在这两个元素之间。
- 所以，如果两个元素相等，我想你是不会再无聊地把他们俩交换一下的；如果两个相等的元素没有相邻，那么即使通过前面的两两交换把两个相邻起来，这时候也不会交换，所以相同元素的前后顺序并没有改变，所以冒泡排序是一种稳定排序算法。
- 实现：
    ```
    void bubbleSort(vector<int>& a, int n) 
    {
        for (auto i = 0; i < n; ++i) 
        {
            for (int j = 0; j < n - i - 1; ++j) 
            {
                if (a[j] > a[j + 1])
                    swap(a[j], a[j + 1]);
            }
        }
    }
    ```
- 冒泡优化版本：假如从开始的第一对到结尾的最后一对，相邻的元素之间都没有发生交换的操作，这意味着右边的元素总是大于等于左边的元素，此时的数组已经是有序的了，我们无需再对剩余的元素重复比较下去了。

6. 选择排序
- 选择排序是给每个位置选择当前元素最小的，比如给第一个位置选择最小的，在剩余元素里面给第二个元素选择第二小的，依次类推，直到第n-1个元素，第n个元素不用选择了，因为只剩下它一个最大的元素了。
- 那么，在一趟选择，如果当前元素比一个元素小，而该小的元素又出现在一个和当前元素相等的元素后面，那么交换后稳定性就被破坏了。所以选择排序不是一个稳定的排序算法。
- 实现：
    ```
    void selectSort(vector<int>& nums)
    {
        int len = nums.size();
        int minIndex = 0;
        for (int i = 0; i < len; ++i) 
        {
            minIndex = i;
            for (int j = i + 1; j < len; ++j) 
            {
                if (nums[j] < nums[minIndex]) minIndex = j;
            }
            swap(nums[i], nums[minIndex]);
        }
    }
    ```

7. 插入排序
- 插入排序是在一个已经有序的小序列的基础上，一次插入一个元素。
- 当然，刚开始这个有序的小序列只有1个元素，就是第一个元素。比较是从有序序列的末尾开始，也就是想要插入的元素和已经有序的最大者开始比起，如果比它大则直接插入在其后面，否则一直往前找直到找到它该插入的位置。如果碰见一个和插入元素相等的，那么插入元素把想插入的元素放在相等元素的后面。所以，相等元素的前后顺序没有改变，从原无序序列出去的顺序就是排好序后的顺序，所以插入排序是稳定的。
- 实现：
    ```
    void insertionSort(vector<int>& a, int n) 
    {
        for (int i = 1; i < n; ++i) 
        {
            if (a[i] < a[i - 1])                // 若第i个元素大于i-1元素，直接插入。小于的话，移动有序表后插入 
            {   
                int j = i - 1;
                int x = a[i];                   //复制为哨兵，即存储待排序元素
                while (j >= 0 && x < a[j])      //查找在有序表的插入位置，还必须要保证j是>=0的，因为a[j]要合法
                {   
                    a[j + 1] = a[j];
                    j--;     //元素后移
                }
                a[j + 1] = x;     //插入到正确位置
            }
        }
    }
    ```

8. 快速排序
- 算法思想
    - 1、选取第一个数为基准
    - 2、将比基准小的数交换到前面，比基准大的数交换到后面
    - 3、对左右区间重复第二步，直到各区间只有一个数
- 实现：
    ```
    void quickSort(vector<int>& a, int low, int high) 
    {
        if (low >= high)        // 结束标志
            return;
        
        int first = low;        // 低位下标
        int last = high;        // 高位下标
        int key = a[first];     // 设第一个为基准
        while (first < last)
        {
            // 从后往前走，将比第一个小的移到前面
            while (first < last && a[last] > key)
                last--;
            if (first < last)
                a[first++] = a[last];
            //从前往后走，将比第一个大的移到后面
            while (first < last && a[first] <= key)
                first++;
            if (first < last)
                a[last--] = a[first];
        }
        a[first] = key;
        // 前半递归
        quickSort(a, low, first - 1);
        // 后半递归
        quickSort(a, first + 1, high);
    }
    
    quickSort(A, 0, A.size()-1);
    ```

9. 希尔排序 
- 希尔排序可以说是插入排序的一种变种。无论是插入排序还是冒泡排序，如果数组的最大值刚好是在第一位，要将它挪到正确的位置就需要 n - 1 次移动。也就是说，原数组的一个元素如果距离它正确的位置很远的话，则需要与相邻元素交换很多次才能到达正确的位置，这样是相对比较花时间了。
- 希尔排序就是为了加快速度简单地改进了插入排序，交换不相邻的元素以对数组的局部进行排序。
- 希尔排序的思想是采用插入排序的方法，先让数组中任意间隔为 h 的元素有序，刚开始 h 的大小可以是 h = n / 2,接着让 h = n / 4 ，让 h 一直缩小，当h = 1 时，也就是此时数组中任意间隔为 1的元素有序，此时的数组就是有序的了。
- 实现：
    ```
    void shellSortCore(vector<int>& nums, int gap, int i) 
    {
        int inserted = nums[i];
        int j;
        //  插入的时候按组进行插入
        for (j = i - gap; j >= 0 && inserted < nums[j]; j -= gap) 
        {
            nums[j + gap] = nums[j];
        }
        nums[j + gap] = inserted;
    }
    
    void shellSort(vector<int>& nums) 
    {
        int len = nums.size();
        //进行分组，最开始的时候， gap为数组长度一半
        for (int gap = len / 2; gap > 0; gap /= 2) 
        {
            //对各个分组进行插入分组
            for (int i = gap; i < len; ++i) 
            {
                //将nums[i]插入到所在分组正确的位置上
                shellSortCore(nums,gap,i);
            }
        }
    }
    ```

10. 归并排序
- 将一个大的无序数组有序，我们可以把大的数组分成两个，然后对这两个数组分别进行排序，之后在把这两个数组合并成一个有序的数组。由于两个小的数组都是有序的，所以在合并的时候是很快的。
- 通过递归的方式将大的数组一直分割，直到数组的大小为1，此时只有一个元素，那么该数组就是有序的了，之后再把两个数组大小为 1的合并成一个大小为2的，再把两个大小为2的合并成4的...直到全部小的数组合并起来。
- 归并排序是建立在归并操作上的一种有效的排序算法。该算法是采用分治法（Divide and Conquer）的一个非常典型的应用。将已有序的子序列合并，得到完全有序的序列；即先使每个子序列有序，再使子序列段间有序。若将两个有序表合并成一个有序表，称为2-路归并。
- 实现：
    ```
    void mergeSortCore(vector<int>& data, vector<int>& dataTemp, int low, int high) 
    {
        if (low >= high) return;
        int len = high - low, mid = low + len / 2;
        int start1 = low, end1 = mid, start2 = mid + 1, end2 = high;
        mergeSortCore(data, dataTemp, start1, end1);
        mergeSortCore(data, dataTemp, start2, end2);
        
        int index = low;
        while (start1 <= end1 && start2 <= end2) 
        {
            dataTemp[index++] = data[start1] < data[start2] ? data[start1++] : data[start2++];
        }
        
        while (start1 <= end1) 
        {
            dataTemp[index++] = data[start1++];
        }
        while (start2 <= end2) 
        {
            dataTemp[index++] = data[start2++];
        }
        
        for (index = low; index <= high; ++index) 
        {
            data[index] = dataTemp[index];
        }
    }
    
    void mergeSort(vector<int>& data) 
    {
        int len = data.size();
        vector<int> dataTemp(len, 0);
        mergeSortCore(data, dataTemp, 0, len - 1);
    }
    ```

11. 堆排序
- 实现：
    ```
    void heapify(vector<int>& nums, int n, int i)   // 对有一定顺序的堆，
    // 当前第i个结点取根左右的最大值（这个操作称heapfiy）
    {
        int l = i * 2 + 1, r = i * 2 + 2;
        int max = i;
        if (l<n && nums[l]>nums[max])
            max = l;
        if (r<n && nums[r]>nums[max])
            max = r;
        if (max != i)
        {
            swap(nums[max], nums[i]);
            heapify(nums, n, max);
        }
    }
    
    void heapify_build(vector<int>& nums, int n)    // 建立大根堆，从树的倒数第二层第一个结点开始， 
    // 对每个结点进行heapify操作，然后向上走
    {
        int temp = (n - 2) / 2;
        for (int i = temp; i >= 0; i--)
            heapify(nums, n, i);
    }
    
    void heapify_sort(vector<int>& nums, int n)     // 建立大根堆之后，每次交换最后一个结点和根节点（最大值），
    // 对交换后的根节点继续进行heapify（此时堆的最后一位是最大值，因此不用管他，n变为n-1）
    {
        heapify_build(nums, n);
        for (int i = 0; i < n; i++)
        {
            swap(nums.front(), nums[n - i - 1]);
            heapify(nums, n - i - 1, 0);
        }
    }
    ```

12. 计数排序
- 计数排序统计小于等于该元素值的元素的个数i，于是该元素就放在目标数组的索引i位（i≥0）。
    - 计数排序基于一个假设，待排序数列的所有数均为整数，且出现在（ 0 ，k）的区间之内。
    - 如果 k（待排数组的最大值）过大则会引起较大的空间复杂度，一般是用来排序 0 到 100 之间的数字的最好的算法，但是它不适合按字母顺序排序人名。
    - 计数排序不是比较排序，排序的速度快于任何比较排序算法。
- 算法思想：
    1. 找出待排序的数组中最大和最小的元素；
    2. 统计数组中每个值为 i 的元素出现的次数，存入数组 C 的第 i项；
    3. 对所有的计数累加（从 C 中的第一个元素开始，每一项和前一项相加）；
    4. 向填充目标数组：将每个元素 i 放在新数组的第 C[i] 项，每放一个元素就将 C[i] 减去 1 ；
- 实现：
    ```
    // 计数排序
    void CountSort(vector<int>& vecRaw, vector<int>& vecObj)
    {
        // 确保待排序容器非空
        if (vecRaw.size() == 0)
            return;
        
        // 使用 vecRaw 的最大值 + 1 作为计数容器 countVec 的大小
        int vecCountLength = (*max_element(begin(vecRaw), end(vecRaw))) + 1; 
        vector<int> vecCount(vecCountLength, 0);
        
        // 统计每个键值出现的次数
        for (int i = 0; i < vecRaw.size(); i++)
            vecCount[vecRaw[i]]++;
        // 后面的键值出现的位置为前面所有键值出现的次数之和
        for (int i = 1; i < vecCountLength; i++)
            vecCount[i] += vecCount[i - 1];
        // 将键值放到目标位置
        for (int i = vecRaw.size(); i > 0; i--) // 此处逆序是为了保持相同键值的稳定性
            vecObj[--vecCount[vecRaw[i - 1]]] = vecRaw[i - 1];
    }
    ```

13. 桶排序
- 将值为i的元素放入i号桶，最后依次把桶里的元素倒出来。
- 算法思想：
    1. 设置一个定量的数组当作空桶子。
    2. 寻访序列，并且把项目一个一个放到对应的桶子去。
    3. 对每个不是空的桶子进行排序。
    4. 从不是空的桶子里把项目再放回原来的序列中。
- 实现：
    ```
    function bucketSort(arr, bucketSize) 
    {
        if (arr.length === 0) 
        {
            return arr;
        }
        
        var i;
        var minValue = arr[0];
        var maxValue = arr[0];
        for (i = 1; i < arr.length; i++) 
        {
            if (arr[i] < minValue)
            {
                minValue = arr[i];                // 输入数据的最小值
            } 
            else if (arr[i] > maxValue)
            {
                maxValue = arr[i];                // 输入数据的最大值
            }
        }
        
        // 桶的初始化
        var DEFAULT_BUCKET_SIZE = 5;            // 设置桶的默认数量为5
        bucketSize = bucketSize || DEFAULT_BUCKET_SIZE;
        var bucketCount = Math.floor((maxValue - minValue) / bucketSize) + 1; 
        var buckets = new Array(bucketCount);
        for (i = 0; i < buckets.length; i++) 
        {
            buckets[i] = [];
        }
        
        // 利用映射函数将数据分配到各个桶中
        for (i = 0; i < arr.length; i++) 
        {
            buckets[Math.floor((arr[i] - minValue) / bucketSize)].push(arr[i]); 
        }
        
        arr.length = 0;
        for (i = 0; i < buckets.length; i++) 
        {
            insertionSort(buckets[i]);	// 对每个桶进行排序，这里使用了插入排序
            for (var j = 0; j < buckets[i].length; j++)
            {
                arr.push(buckets[i][j]);
            }
        }
        return arr;
    }
    ```

14. 基数排序
- 一种多关键字的排序算法，可用桶排序实现。
- 算法思想：
    1. 取得数组中的最大数，并取得位数；
    2. arr为原始数组，从最低位开始取每个位组成radix数组；
    3. 对radix进行计数排序（利用计数排序适用于小范围数的特点）
- 实现：
    ```
    int maxbit(int data[], int n)   //辅助函数，求数据的最大位数
    {
        int maxData = data[0];      // 最大数
        // 先求出最大数，再求其位数，这样有原先依次每个数判断其位数，稍微优化点。
        for (int i = 1; i < n; ++i)
        {
            if (maxData < data[i])
                maxData = data[i];
        }
        int d = 1;
        int p = 10;
        while (maxData >= p)
        {
            maxData /= 10;
            ++d;
        }
        return d;
    }
    void radixsort(int data[], int n)   // 基数排序
    {
        int d = maxbit(data, n);
        int *tmp = new int[n];
        int *count = new int[10];   // 计数器
        int i, j, k;
        int radix = 1;
        for(i = 1; i <= d; i++)     // 进行d次排序
        {
            for(j = 0; j < 10; j++)
            count[j] = 0;           // 每次分配前清空计数器
            for(j = 0; j < n; j++)
            {
                k = (data[j] / radix) % 10;         // 统计每个桶中的记录数
                count[k]++;
            }
            for(j = 1; j < 10; j++)
                count[j] = count[j - 1] + count[j]; // 将tmp中的位置依次分配给每个桶 
            for(j = n - 1; j >= 0; j--)             // 将所有桶中记录依次收集到tmp中
            {
                k = (data[j] / radix) % 10;
                tmp[count[k] - 1] = data[j];
                count[k]--;
            }
            for(j = 0; j < n; j++)                  // 将临时数组的内容复制到data中
                data[j] = tmp[j];
            radix = radix * 10;
        }
        delete []tmp;
        delete []count;
    }
    ```

15. 高频面试题——合并有序链表
```
myList* merge(myList* l1, myList* l2) 
{
    if (l1 == nullptr) return l2;
    if (l2 == nullptr) return l1;
    myList head(0);
    myList* node = &head;
    while (l1 != nullptr && l2 != nullptr) 
    {
        if (l1->val < l2->val) 
        {
            node->next = l1;
            l1 = l1->next;
        }
        else 
        {
            node->next = l2;
            l2 = l2->next;
        }
        node = node->next;
    }
    if (l1 == nullptr)
        node->next = l2;
    if (l2 == nullptr)
        node->next = l1;
    return head.next;
};
```

16. 高频面试题——反转链表
```
struct node* reverse(node* head) 
{
    struct node* pre = new node(-1);
    struct node* temp = new node(-1);
    pre = head;
    temp = head->next;
    pre->next = nullptr;
    struct node* cur = new node(-1);
    cur = temp;
    while (cur != nullptr) 
    {
        temp = cur;
        cur = cur->next;
        temp->next = pre;
        pre = temp;
    }
    return pre;
}
```

17. 高频面试题——单例模式
```
// 饿汉模式
class singlePattern
{
private:
    singlePattern() {};
    static singlePattern* p;
public:
    static singlePattern* instance();
    class CG 
    {
        public:
        ~CG() 
        {
            if (singlePattern::p != nullptr) 
            {
                delete singlePattern::p;
                singlePattern::p = nullptr;
            }
        }
    };
};

singlePattern* singlePattern::p = new singlePattern();
singlePattern* singlePattern::instance() 
{
    return p;
}


// 懒汉模式
class singlePattern {
private:
    static singlePattern* p;
    singlePattern(){}
public:
    static singlePattern* instance();
    class CG 
    {
    public:
        ~CG()
        {
            if (singlePattern::p != nullptr) 
            {
                delete singlePattern::p;
                singlePattern::p = nullptr;
            }
        }
    };
};
singlePattern* singlePattern::p = nullptr;
singlePattern* singlePattern::instance() 
{
    if (p == nullptr) 
    {
    return new singlePattern();
    }
    return p;
}
```

18. 高频面试题——简单工厂模式
```
typedef enum productType 
{
    TypeA,
    TypeB,
    TypeC
} productTypeTag;

class Product 
{
public:
    virtual void show() = 0;
    virtual ~Product() = 0;
};

class ProductA : public Product
{
public:
    void show() 
    {
        cout << "ProductA" << endl;
    }
    ~ProductA() 
    {
        cout << "~ProductA" << endl;
    }
};

class ProductB : public Product 
{
public:
    void show() 
    {
        cout << "ProductB" << endl;
    }
    ~ProductB() 
    {
        cout << "~ProductB" << endl;
    }
};

class ProductC : public Product 
{
public:
    void show() 
    {
        cout << "ProductC" << endl;
    }
    ~ProductC()
    {
        cout << "~ProductC" << endl;
    }
};

class Factory 
{
public:
    Product* createProduct(productType type)
    {
        switch (type) 
        {
        case TypeA:
            return new ProductA();
        case TypeB:
            return new ProductB();
        case TypeC:
            return new ProductC();
        default:
            return nullptr;
        }
    }
};
```

19. 高频面试题——快速排序
```
void quickSort(vector<int>& data, int low, int high) 
{
    if (low >= high) return;
    int key = data[low], begin = low, end = high;
    while (begin < end) 
    {
        while (begin<end && data[end]>key) 
        {
            end--;
        }
        if (begin < end) data[begin++] = data[end];
        
        while (begin<end && data[begin]<= key) 
        {
            begin++;
        }
        if (begin < end) data[end--] = data[begin];
    }
    
    data[begin] = key;
    quickSort(data, low, begin - 1);
    quickSort(data, begin + 1,high);
}
```

20. 高频面试题——归并排序
```
void mergeSort(vector<int>& data, vector<int>& copy, int begin, int end) 
{ 
    if (begin >= end) return;
    int mid = begin + (end - begin) / 2;
    int low1 = begin, high1 = mid, low2 = mid + 1, high2 = end, index = begin; 
	mergeSort(copy, data, low1, high1);
    mergeSort(copy, data, low2, high2);
    while (low1 <= high1 && low2 <= high2) 
    {
        copy[index++] = data[low1] < data[low2] ? data[low1++] : data[low2++]; 
    }
    
    while (low1 <= high1) 
    {
        copy[index++] = data[low1++];
    }
    
    while (low2 <= high2) 
    {
        copy[index++] = data[low2++];
    }
}
```

21. 高频面试题——实现一个堆排序
- 堆排序的基本过程：
    - 将n个元素的序列构建一个大顶堆或小顶堆
    - 将堆顶的元素放到序列末尾
    - 将前n-1个元素重新构建大顶堆或小顶堆，重复这个过程，直到所有元素都已经排序
- 整体时间复杂度为nlogn
```
void swap(vector<int>& arr, int a,int b)
{
    arr[a]=arr[a]^arr[b];
    arr[b]=arr[a]^arr[b];
    arr[a]=arr[a]^arr[b];
}

void adjust(vector<int>& arr,int len,int index)
{
    int maxid=index;
    // 计算左右子节点的下标   left=2*i+1  right=2*i+2  parent=(i-1)/2
    int left=2*index+1,right=2*index+2;
    // 寻找当前以index为根的子树中最大/最小的元素的下标
    if(left<len and arr[left]<arr[maxid]) maxid=left;
    if(right<len and arr[right]<arr[maxid]) maxid=right;
    // 进行交换，记得要递归进行adjust,传入的index是maxid
    if(maxid!=index)
    {
        swap(arr,maxid,index);
        adjust(arr,len,maxid);
    }
}

void heapsort(vector<int>&arr,int len)
{
    // 初次构建堆，i要从最后一个非叶子节点开始，所以是(len-1-1)/2，0这个位置要加等号
    for(int i=(len-1-1)/2;i>=0;i--)
    {
        adjust(arr,len,i);
    }
    // 从最后一个元素的下标开始往前遍历，每次将堆顶元素交换至当前位置，并且缩小长度（i为长度），从0处开始adjust
    for(int i=len-1;i>0;i--)
    {
        swap(arr,0,i);
        adjust(arr,i,0);// 注意每次adjust是从根往下调整，所以这里index是0！ 
    }
}
```

22. 高频面试题——设计LRU缓存
```

```

23. 高频面试题——重排链表
```

```

24. 高频面试题——奇偶链表
```
ListNode* oddEvenList(ListNode* head) 
{
    if(head==NULL || head->next==NULL)
    {
        return head;
    }
    ListNode* first=head;           //奇链表头结点
    ListNode* second=head->next;    //偶链表头结点
    ListNode* cur=second;           //保存偶链表头结点
    while(second != nullptr && second->next  != nullptr)
    {
        first->next=second->next;
        second->next=first->next->next;
        first=first->next;
        second=second->next;
    }
    first->next=cur;
    return head;
}
```

25. 高频面试题——写三个线程交替打印ABC
```
mutex mymutex;
condition_variable cv;
int flag=0;
void printa()
{
    unique_lock<mutex> lk(mymutex);
    int count=0;
    while(count<10)
    {
        while(flag!=0) cv.wait(lk);
        cout<<"thread 1: a"<<endl;
        flag=1;
        cv.notify_all();
        count++;
    }
}
void printb()
{
    unique_lock<mutex> lk(mymutex);
    for(int i=0;i<10;i++)
    {
        while(flag!=1) cv.wait(lk);
        cout<<"thread 2: b"<<endl;
        flag=2;
        cv.notify_all();
    }
}
void printc()
{
    unique_lock<mutex> lk(mymutex);
    for(int i=0; i<10; i++)
    {
        while(flag!=2) cv.wait(lk);
        cout<<"thread 3: c"<<endl;
        flag=0;
        cv.notify_all();
    }
}
int main()
{
    thread th2(printa);
    thread th1(printb);
    thread th3(printc);
    th1.join();
    th2.join();
    th3.join();
}
```

26. 高频面试题——Top K问题
- Top K 问题的常见形式：
    - 给定 10000个整数，找第K大（第K小）的数
    - 给定 10000个整数，找出最大（最小）的前K个数
    - 给定 100000个单词，求前K词频的单词
- 解决Top K问题若干种方法
    - 使用最大最小堆。求最大的数用最小堆，求最小的数用最大堆。
    - Quick Select算法。使用类似快排的思路，根据pivot划分数组。
    - 使用排序方法，排序后再寻找top K元素。
    - 使用选择排序的思想，对前K个元素部分排序。
    - 将 1000.....个数分成m组，每组寻找top K个数，得到m×K个数，在这m×k个数里面找top K个数。
- 1. 使用最大最小堆的思路（以top K 最大元素为例）
    - 按顺序扫描这 10000个数，先取出K个元素构建一个大小为K的最小堆。每扫描到一个元素，如果这个元素大于堆顶的元素（这个堆最小的一个数），就放入堆中，并删除堆顶的元素，同时整理堆。如果这个元素小于堆顶的元素，就直接pass。最后堆中剩下的元素就是最大的前Top K个元素，最右的叶节点就是Top 第K大的元素。
    - note：最小堆的插入时间复杂度为log(n) ，n为堆中元素个数，在这里是K。最小堆的初始化时间复杂度是nlog(n)
    - C++中的最大最小堆要用标准库的priority_queue来实现。
        ```
        struct Node 
        {
            int value;
            int idx;
            Node (int v, int i): value(v), idx(i) {}
            friend bool operator < (const struct Node &n1, const struct Node &n2);
        };
            
        inline bool operator < (const struct Node &n1, const struct Node &n2) { return n1.value < n2.value; }
        priority_queue<Node> pq; // 此时pq为最大堆
        ```
- 2. 使用Quick Select的思路（以寻找第K大的元素为例）
    - Quick Select脱胎于快速排序，提出这两个算法的都是同一个人。算法的过程是这样的：
        - 首先选取一个枢轴，然后将数组中小于该枢轴的数放到左边，大于该枢轴的数放到右边。
        - 此时，如果左边的数组中的元素个数大于等于K，则第K大的数肯定在左边数组中，继续对左边数组执行相同操作；
        - 如果左边的数组元素个数等于K-1，则第K大的数就是pivot；
        - 如果左边的数组元素个数小于K ，则第K大的数肯定在右边数组中，对右边数组执行相同操作。
    - 这个算法与快排最大的区别是，每次划分后只处理左半边或者右半边，而快排在划分后对左右半边都继续排序。
    - 实现：
        ```
        //此为Java实现
        public int findKthLargest(int[] nums, int k) 
        {
            return quickSelect(nums, k, 0, nums.length - 1);
        }

        // quick select to find the kth-largest element
        public int quickSelect(int[] arr, int k, int left, int right) 
        {
        if (left == right) return arr[right];
        int index = partition(arr, left, right);
        if (index - left + 1 > k)
            return quickSelect(arr, k, left, index - 1);
        else if (index - left + 1 == k)
            return arr[index];
        else
            return quickSelect(arr, k-(index-left+1), index+1, right);
        }
        ```
- 3. 使用选择排序的思想对前K个元素排序（以寻找前K大个元素为例）
    - 扫描一遍数组，选出最大的一个元素，然后再扫描一遍数组，找出第二大的元素，再扫描一遍数组，找出第三大的元素。。。。。以此类推，找K个元素，时间复杂度为O(N*K)

27. 高频面试题——布隆过滤器原理与优点
- 布隆过滤器是一个比特向量或者比特数组，它本质上是一种概率型数据结构，用来查找一个元素是否在集合中，支持高效插入和查询某条记录。常作为针对超大数据量下高效查找数据的一种方法。
- 它的具体工作过程是这样子的：
    - 假设布隆过滤器的大小为m（比特向量的长度为m ），有k个哈希函数，它对每个数据用这k个哈希函数 计算哈希，得到k个哈希值，然后将向量中相应的位设为1。在查询某个数据是否存在的时候，对这个数 据用k个哈希函数得到k个哈希值，再在比特向量中相应的位查找是否为1 ，如果某一个相应的位不为 1 ， 那这个数据就肯定不存在。但是如果全找到了，则这个数据有可能存在。
- 为什么说有可能存在呢？
    - 因为不同的数据经过哈希后可能有相同的哈希值，在比特向量上某个位置查找到1也可能是由于某个另外的数据映射得到的。
- 支持删除操作吗
    - 目前布隆过滤器只支持插入和查找操作，不支持删除操作，如果要支持删除，就要另外使用一个计数变量，每次将相应的位置为 1则计数加一，删除则减一。
    - 布隆过滤器中哈希函数的个数需要选择。如果太多则很快所有位都置为 1，如果太少会容易误报。

28. 如果判断一个链表是否有环？其最坏时间复杂度是什么？
快慢指针法，O(n)

29. 无序链表的排序？
 