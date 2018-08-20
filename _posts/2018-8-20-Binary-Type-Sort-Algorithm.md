### 1 二分类型排序

快速排序；归并排序；堆排序。

#### 1.1 快速排序

**思路**：调用 partition 将数组分为两部分，第一部分比标兵小，第二部分比标兵大。返回标兵位置。之后递归调用。partition 用 [low, i) 表示比 标兵小的数据，最后 i 即为标兵位置。

```c++
int partition(vector<int>& arr, int low, int high) {
    int i = low;
    for(int j = low; j < high; j++) {
        if(arr[j] < arr[high]) {  // arr[high] 作为标兵
            swap(arr[i++], arr[j]);
        }
    }
    swap(arr[i], arr[high]);
    return i;
}

void quickSort(vector<int>& arr, int low, int high) {
    if(low < high) {
        int i = partition(arr, low, high);
        quickSort(arr, low, i-1);
        quickSort(arr, i+1, high);
    }
}
```

**算法复杂度**： 平均 O(nlogn) ，最好 O(nlogn) ，最坏 O(n^2) 。

**最坏情况**：数组基本有序，选择最左 or 最右元素作为标兵。无法“二分数组”，退化为冒泡排序。

**标兵选择方法**：

1. 选择最左端 or 最右端元素；
2. 随机选择；
3. 取两端以及中间位置三个数的中位数作为标兵。

**稳定性**：不稳定。如数组 [1, 3, 2(1), 2(2)] ，取 2(2) 作为标兵，将小于 2 的数放左边，最后得到 [1, 2(2), 3, 2(1)] ，2(1) 与 2(2) 调换了位置，所以不稳定。

**优化**：

1. 当待排序序列的长度分割到一定大小后，使用插入排序。
2. 在一次分割结束后，可以把与Key相等的元素聚在一起，继续下次分割时，不用再对与key相等元素分割

**应用**：

1. 求第 K 大的数。通过将标兵的位置 i 与 K 进行比较缩小 partition 范围。复杂度：O(n)
2. 求最小的 K 个数。与 1. 类似。

#### 1.2 归并排序

**思路**：将数组不断二分，然后合并成一个有序的数组。合并过程可以通过一个数组存储中间结果，或者通过插入排序的方式进行合并（复杂度很高）。

```c++
// 插入合并 O(n^2logn)
void merge_v1(vector<int> &arr, int low, int mid, int high) {
    int index = low;
    for(int i = mid; i <= high; i++) {
        for(int j = index; j < mid; j++) {
            if(arr[i] < arr[j]) {
                int tmp = arr[i];
                for(int k = i - 1; k >= j; k--)
                    arr[k+1] = arr[k];
                arr[j] = tmp;
                mid++;
                index++;
                break;
            } else {
                index++;
            }
        }
    }
}
// 空间换时间 O(nlogn)
void merge_v2(vector<int> &arr, int low, int mid, int high) {
    vector<int> tmpV(high-low+1);
    int i = low, j = mid, k = 0;
    while(i < mid && j <= high)
        if(arr[i] <= arr[j]) tmpV[k++] = arr[i++];
        else tmpV[k++] = arr[j++];
    while(i < mid) tmpV[k++] = arr[i++];
    while(j <= high) tmpV[k++] = arr[j++];
    for(auto val : tmpV)
        arr[low++] = val;
}

void mergeSort(vector<int> &arr, int low, int high) {
    if(low >= high) return;
    int mid = low + (high - low) / 2;
    mergeSort(arr, low, mid);
    mergeSort(arr, mid+1, high);
    merge_v2(arr, low, mid+1, high);
}
```

**时间复杂度**：平均/最好/最坏 O(nlogn)；空间复杂度：O(n)。稳定性：稳定。

**应用**：

1. 外排序。待排序数据无法一次装入内存。一般存储在外存上，然后通过“排序-归并”的策略，排序多个小文件，归并多个小文件。
2. 统计数组中的逆序对。数组中，前面的数大于后面的数的组合。比如 {7, 5, 6, 4} 中，一共有 5 个逆序对：{7,5}, {7,6}, {7,4}, {5,4}, {6,4} 。 在合并比较大小决定那个元素为下一个元素的过程中，可以统计逆序对的个数。

```c++
int merge_v2(vector<int> &arr, int low, int mid, int high) {
    vector<int> tmpV(high-low+1);
    int i = low, j = mid, k = 0;
    int cnt = 0;
    while(i < mid && j <= high)
        if(arr[i] <= arr[j]) tmpV[k++] = arr[i++];
        else {
        	tmpV[k++] = arr[j++];
        	cnt = cnt + mid - i;  // 逆序，加上以 arr[j] 为逆序对第二个数的逆序组合数
        }
    while(i < mid) tmpV[k++] = arr[i++];
    while(j <= high) tmpV[k++] = arr[j++];
    for(auto val : tmpV)
        arr[low++] = val;
    return cnt;
}

int inversePairs(vector<int> &arr, int low, int high) {
    if(low >= high) return;
    int mid = low + (high - low) / 2;
    int leftCnt = inversePairs(arr, low, mid);
    int rightCnt = inversePairs(arr, mid+1, high);
    int mergeCnt = merge_v2(arr, low, mid+1, high);
    return leftCnt + rightCnt + mergeCnt;
}
```

#### 1.3 堆排序

**思路**：首先自底向上建立最大堆（自顶向下调整），然后将堆顶元素与最后一个元素交换，以前 n-1 个元素重新自顶向下调整为最大堆，重复上述过程。最后得到一个正序数组。

```c++
void heapAdjust(vector<int> &arr, int parent, int length) {
    int temp = arr[parent]; // 暂存“小元素”
    int child = 2 * parent + 1; 
	
    // 小元素下沉，大元素上浮
    while(child < length) {
        if(child + 1 < length && arr[child] < arr[child+1]) // 取最大的子树进行比较
            child++;
        if(temp >= arr[child])
            break;
        arr[parent] = arr[child];
        parent = child;
        child = 2 * parent + 1;
    }
    arr[parent] = temp; // 小元素下沉到的位置
}

void heapSort(vector<int>& arr) {
    for(int i = arr.size() / 2; i >= 0; i--) { 
        heapAdjust(arr, i, arr.size());
    }
    for(int i = arr.size()-1; i > 0; i--) {
        swap(arr[i], arr[0]);
        heapAdjust(arr, 0, i); 
    }
}
```

**复杂度**：平均/最好/最坏 O(nlogn) ，稳定性：不稳定。

**插入节点**：在数组的最末尾插入新节点。然后自下而上调整子节点与父节点（称作bubble-up操作）：比较当前节点与父节点，不满足*堆性质*则交换。从而使得当前子树满足二叉堆的性质。时间复杂度为 O(logn)。 

**删除节点**：堆排序过程中，交换堆顶/堆末元素，重新调整堆，即为删除节点操作。从上至下调整（bubble-down）。时间复杂度为 O(logn)。 

**应用**：

1. 大量数据中最大的前 K 个元素：维护一个 K 个元素的最小堆，每次读取数据中的一个数与堆顶比较，比堆顶大，则代替堆顶，并重新调整堆。复杂度 O(n*logk) 。
2. K 路有序归并（取Top-N）：从 K 个数组中各取一个数，并记录每个数的来源数组，建立一个含 K 个元素的大根堆。此时堆顶就是最大的数，取出堆顶元素，并从堆顶元素的来源数组中取下一个数加入堆，再取最大值，一直这样进行 N 次即可。完整归并也与此类似。 