# 从无序到有序的 7种排序法
http://www.360doc.com/content/18/0815/21/11881101_778564690.shtml

# 内存分配
result = malloc(sizeof(char) * (max+1));

# swap的库函数
void Swap(int arr[], int low, int high)
{
    int temp;
    temp = arr[low];
    arr[low] = arr[high];
    arr[high] = temp;
}

# 插入排序
// insert order
// what we want is to reorder the para of array from small to bigger
void reoder_small_2_bigger(int *array, int len)
{
    int i = 0;
    int j = 0;
    int tmp = 0;

    for(j = 1;j < len;j++){
        i = j - 1;
        while( (i >= 0) && (array[i] > array[i + 1])){
            tmp = array[i + 1];
            array[i + 1] = array[i];
            array[i] = tmp;
            i--;
        }
    }
}

# 快速排序

int Partition(int arr[], int low, int high)
{
    int base = arr[low];
    while(low < high)
    {
        while(low < high && arr[high] >= base)
        {
            high --;
        }
        Swap(arr, low, high);
        while(low < high && arr[low] <= base)
        {
            low ++;
        }
        Swap(arr, low, high);
    }
    return low;
}

void QuickSort(int arr[], int low, int high)
{
    if(low < high)
    {
        int base = Partition(arr, low, high);
        QuickSort(arr, low, base - 1);
        QuickSort(arr, base + 1, high);
    }
}
