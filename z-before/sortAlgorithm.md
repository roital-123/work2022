**冒泡排序**

```c++
/*
*	@desc	冒泡排序，每一趟最大数据调整到数组末尾部分
*	@param	arr	输入数组
*	@param	a	数组起始数据下标
*	@param	b	数组结束数据下标
*	@return	无返回值
*/
void bubbleSort(int arr[], int a, int b){
    /*b-a+1个元素，需要b-a趟冒泡*/
    for(int i=0; i<b-a; i++){
        bool flag = false;
        for(int j=a; j<b-i; j++){
            if(arr[j]>arr[j+1]){
                swap(arr[j], arr[j+1]);
                flag = true;
            }
        }
        if(flag == false)	break;        
    }
}

/*每次把最小元素调整到数组开头部分*/
for(int i=0; i<b-a; i++){
    bool flag = false;
    for(int j=b; j>a+i; j--){
        if(arr[j]<arr[j-1]){
            swap(arr[j], arr[j-1]);
            flag = true;
        }
    }
    if(flag == false)	break;
}

/*
*	@des	简化数组的元素下标从0到n-1
*	@param	arr:	待排序数组
*	@param	n:	数组长度
*	@return	无返回值
*/
void bubbleSort(int arr[], int n){  
    /* i 记录冒泡的趟数，0 ~ n-1只需要n-1趟排序,每一趟冒泡会把最大元素调整到数组末尾部位*/
    for(int i=0; i<n-1; i++){
        /*记录i次冒泡过程有没有交换数据*/
        bool flag = false;	
        /* j 与 j+1 跟踪数组待比较元素*/
        for(int j=0; j<n-1-i; j++){
            if(arr[j] > arr[j+1]){
                int tmp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = tmp;
                flag = true；
            }
        }
        if(flag == false){
            break;
        }
    }
}

/*每次冒泡把最小元素调整到数组开头部位*/
for(int i=0; i<n-1; i++){
    bool flag = false;
    /*j 与 j-1 跟踪需要调整位置元素*/
    for(int j=n-1; j>i; j--){
        if(arr[j] < arr[j-1]){
            swap(arr[j], arr[j-1]);
            flag = true;
        }  
    }
    if(flag == false){
        break;
    }
}
```

**快速排序**

```c++
/*
*	@desc	快速排序
*	@param	arr	数组
*	@param	left	开始元素下标
*	@param	right	结束元素下标
*	@return	无返回值
*/
void quickSort(int arr[], int left, int right){
    int pivotPos = partition(arr, left, right);
    quickSort(arr, left, pivotPos-1);
    quickSort(arr, pivotPos+1, right);
}

/*
*	@desc	一次快排确定一个标兵的最终位置，每次标兵选择开头元素
*	@param	arr	数组
*	@param	left	开始元素
*	@param	right	结束元素
*	@return	返回标兵元素的最终下标位置
*/
int partition(int arr[], int left, int right){
    int pivot = arr[left];
    while(left<right){
        while(arr[right] >= pivot && left<right)	right--;
        arr[left] = arr[right];
        while(arr[left] < pivot && left<right)	left++;
        arr[right] = arr[left];
    }
    return left;
}
```

**标准二分查找**

```c++
/*
*	@desc	给定有序数组，查找指定元素值               
*	@param	a	有序数组（由小到大）
*	@param	left	开始元素下标
*	@param	right	结束元素下标
*	@return	返回元素下标值，未查找到返回-1
*/
int binarySearch(int a[], int left, int right, int data){
    /*注：left = right条件下，该位置还未判断，所以while条件中需要包含*/
    while(left<=right){
        int mid = left + (right-left)/2;
        if(a[mid] == data){
            return mid;
        }
        else if(a[mid] > data){
            right = mid-1;
        }
        else{
            left = mid+1;
        }
    }
    return -1;
}
```

**二分查找大于等于给定元素的第一个元素**

```c++
/*
*	@desc	二分查找第一个大于或者等于给定元素的值
*	@param	a	有序数组（由小到大）
*	@param	left	起始下标
*	@param	right	结束下标
*	@param	data	给定元素值
*	@return	返回最接近元素的值
*/
int binarySearch(int a[], int left, int right, int data){
    int l=left, r=right;
    while(left<=right){
        int mid = left + (right - left)/2;
        if(a[mid] == data){
            while(a[mid] == data){
                mid--;
            }
            mid++;
            return mid;
        }
        else(a[mid] < data){
            left = mid+1;
        }
        else{
            right = mid-1;
        }
    }
    /*数组中没有等于data的元素，则寻找第一个大于的元素*/
    /*直接比较a[right]与data a[left]与data*/
    if(right<left){
        if(right<l){
            return left;
        }
        else if(left>r){
            return right;
        }
        else{         
           	return left;
        }
    }
}
```

