[TOC]

# Vector底层实现：模版类的实现思路

```c++
/* vector 纲领目录

一、vector的实现原理：
	1、vector的基类介绍
	2、vector从最后面插入元素时发生了什么
		1）、对空vector插入一个元素
		2）、vector当前内存用完时插入
	3、在vector中间插入一个元素会发生什么
	4、vector删除元素内存会被释放吗
		1）、从容器最后删除
		2）、从容器中间删除
	5、vector如何修改某个位置的元素值
	6、vector读取元素值效率
	7、c++11为vector增加了什么
	8、vector底层实现总结
二、使用vector的注意事项
	1、不确定时使用at而不是operator[]
	2、什么类型不可以作为vector的模板类型
	3、什么情况下vector的迭代器会失效
	4、vector怎么迅速的释放内存
		1）、通过swap函数
		2）、使用shrink_to_fit函数
	5、vector还能在作为基类被继承吗？
*/
```



- 基类----_Vector_base

  ```c++
  template<typename _Tp, typename _Alloc = std::allocator<_Tp> >
  class vector : protected _Vector_base<_Tp, _Alloc>
  {
  	//...
  };
  /*
  vector 是一个模板类，其基类是_Vector_base模板类，继承方式为保护继承
  	--保护继承：基类公有的成员->保护的成员
  					  基类其他的成员->维持不变
  					  
  不同的基类实现不同：
  	但是，大体上维护类三个指针：区间[ , )
  		pointer _M_start;	//容器开始位置
      pointer _M_finish;	//容器结束位置
      pointer _M_end_of_storage;	//容器所申请的动态内存最后一个位置的下一个位置
      
  具体到构造函数分配空间的时候：
  	调用空间分配器：
  */
  
  ```

- 中间插入、末尾插入

  - 剩余空间够用，则_M_finish指针++
  - 剩余空间不够用，需要空间分配器另分配空间，在执行拷贝

- 删除元素：

  - 删除元素不会将分配到的空间释放掉，内部调用的是容器内元素的析构函数

- c++ 11新增

  - 迭代器：cbegin系列函数（返回常量迭代器）
  - 增加移动构造和移动赋值
  - 增加成员函数shrink_to_fit函数，允许释放未使用的内存
  - 增加emplace和emplace_back函数，支持在指定位置上直接构造元素，以右值引用的方式传递参数，所以相较于push_back这类函数，少了一个拷贝的动作

- 如何释放内存：

  - swap

    ```c++
    /*
    作为一个空容器的时候，vector的大小为0，所以用一个空的vector与当前vector进行交换,v这个vector所代表的内存空间与一个空vector的内存空间进行交换，这样v的内存空间等于被释放掉了，而这个空的vector作为一个临时变量，它在这行代码执行结束后，会自动调用vector的析构函数释放掉动态申请的内存空间。
    */
    vector<int>().swap(v);
    
    // 只需要交换三根指针即可
    // 交换内容
      void swap(vector<_Tp, _Alloc>& __x) {
        __STD::swap(_M_start, __x._M_start);
        __STD::swap(_M_finish, __x._M_finish);
        __STD::swap(_M_end_of_storage, __x._M_end_of_storage);
      }
    ```

  - shrink_to_fit

    ```c++
    /*
    c++ 11后新增的这个函数，他的作用是释放掉未使用的内存，先使用clear将所有的元素清除（调用元素的析构函数），这样整块容器就变成未使用的，在调用shrink_to_fit函数把未使用的部分内存释放掉即释放了整个vector
    */
    ```

- vector不能在作为基类

  - 析构函数非虚，在delete的时候会出错

- reverse和resize函数

  - reverse

    reverse函数只有在参数n大于当前容器的capacity时才会作用，通过空间分配器取到一块更大的内存，将数据拷贝至新内存处并释放原内存空间

    ```c++
    // 预留存储空间，若 __n 大于当前的 capacity() ，则分配新存储，否则该方法不做任何事。
    // 精美的源码
      void reserve(size_type __n) {
        if (capacity() < __n) {
          const size_type __old_size = size();
          // 分配新空间，并拷贝旧数据
          iterator __tmp = _M_allocate_and_copy(__n, _M_start, _M_finish);
          // 元素做析构
          destroy(_M_start, _M_finish);
          // 释放
          _M_deallocate(_M_start, _M_end_of_storage - _M_start);
          _M_start = __tmp;
          _M_finish = __tmp + __old_size;
          _M_end_of_storage = _M_start + __n;
        }
      }
    ```

    

  - `resize(size_type _new_size, const _Tp& _x)`

    1、小于当前size()：直接调用erase函数

    ```c++
    // 改变容器中可存储元素的个数 
    void resize(size_type __new_size, const _Tp& __x) {
    	if (__new_size < size()) 
      	erase(begin() + __new_size, end());// 此处的操作：[ , )
      else
       	insert(end(), __new_size - size(), __x); //可能会导致capacity不够用
    }
    void resize(size_type __new_size) { resize(__new_size, _Tp()); }
    ```

    2、大于等于当前size()：`insert(end(), _new_size()-size(), _x);`

    ​	insert内部会维护这些情况：

    ​	涉及到在末尾插入n个元素的情况：

    ​		a、剩余空间够用

    ​		b、剩余空间不够用（alloc在分配）

- insert函数

  ```c++
  // 在 __position 前插入 __x 
  // 1、入口：
    iterator insert(iterator __position, const _Tp& __x) {
      size_type __n = __position - begin();
      // 能够直接push_back()
      if (_M_finish != _M_end_of_storage && __position == end()) {
        construct(_M_finish, __x); // 内置构造函数，创建对应元素
        ++_M_finish;
      }
      else
        _M_insert_aux(__position, __x); //其他情况
      return begin() + __n;
    }
  
  // 2、核心函数实现（push_back内部在需要分配新空间的时候也是调用这个函数）
  // push_back 调用, insert(position, x)调用，且只会插入一个元素
  // 源码可见：插入后迭代器_pos指向的位置不变，只是值变成了传入的值
  template <class _Tp, class _Alloc>
  void 
  vector<_Tp, _Alloc>::_M_insert_aux(iterator __position, const _Tp& __x)
  {
    // 不需要reallocate
    if (_M_finish != _M_end_of_storage) {
      construct(_M_finish, *(_M_finish - 1));
      ++_M_finish;
      _Tp __x_copy = __x;
      
      
      // 从后向前拷贝元素这里使用的 [first, last) 
      /* c++ 20源码
      template< class BidirIt1, class BidirIt2 >
  BidirIt2 copy_backward(BidirIt1 first, BidirIt1 last, BidirIt2 d_last)
  {
      while (first != last) {
          *(--d_last) = *(--last); //前置--
      }
      return d_last; //注意返回的迭代器
  }
      */
      
      copy_backward(__position, _M_finish - 2, _M_finish - 1);
      *__position = __x_copy;
    }
    // 没有备用空间，需要reallocate
    else {
      const size_type __old_size = size();
      const size_type __len = __old_size != 0 ? 2 * __old_size : 1;
      iterator __new_start = _M_allocate(__len);
      iterator __new_finish = __new_start;
      __STL_TRY {
        // 左开右闭区间 [ ), 返回值时下一个待放入元素的迭代器
        __new_finish = uninitialized_copy(_M_start, __position, __new_start);
        construct(__new_finish, __x);
        ++__new_finish;
        __new_finish = uninitialized_copy(__position, _M_finish, __new_finish);
      }
      __STL_UNWIND((destroy(__new_start,__new_finish), 
                    _M_deallocate(__new_start,__len)));
      // 处理旧数据
      destroy(begin(), end());
      _M_deallocate(_M_start, _M_end_of_storage - _M_start);
      _M_start = __new_start;
      _M_finish = __new_finish;
      _M_end_of_storage = __new_start + __len;
    }
  }
  
  /*
  其他的一些插入函数
  // insert(position, n, x) 调用
  template <class _Tp, class _Alloc>
  void vector<_Tp, _Alloc>::_M_fill_insert(iterator __position, size_type __n, 
                                           const _Tp& __x);
  // insert(pos, first, last) 调用
  template <class _Tp, class _Alloc> template <class _InputIterator>
  void 
  vector<_Tp, _Alloc>::_M_range_insert(iterator __pos, 
                                       _InputIterator __first, 
                                       _InputIterator __last,
                                       input_iterator_tag);
  
  // 区分迭代器的类型
  template <class _Tp, class _Alloc> template <class _ForwardIterator>
  void 
  vector<_Tp, _Alloc>::_M_range_insert(iterator __position,
                                       _ForwardIterator __first,
                                       _ForwardIterator __last,
                                       forward_iterator_tag)
                                       
                                      
  */
  ```

- erase 函数

  ```c++
  // 清除某个位置上的元素
  // 清除后，_postion位置不变
    iterator erase(iterator __position) {
      // 中间删除
      if (__position + 1 != end())
        copy(__position + 1, _M_finish, __position); 
      --_M_finish;
      destroy(_M_finish);
      return __position;
    }
    
   // 清除 [first, last) 中的所有元素
    iterator erase(iterator __first, iterator __last) {
      iterator __i = copy(__last, _M_finish, __first);
      destroy(__i, _M_finish);
      _M_finish = _M_finish - (__last - __first);
      return __first;
    }
  ```

- vector一些基本函数不混淆

  ```c++
  /*
  front() //返回头部元素
  back()
  
  push_back(...)
  emplace_back(...)
  pop_back()
  
  insert(it, x)
  erase(it)
  
  sort(v.begin(), v.end()); // 算法库中共用的sort
  */
  ```



# list底层原理

- list的基本API

```c++
/*
list<int> lst;
lst.front();
lst.back();

lst.push_back();
lst.push_front();
lst.pop_back();
lst.pop_back();


*/
```



# Deque底层原理

[参考](http://c.biancheng.net/view/6908.html)

> - deque容器存储数据的空间是分段的，段之间不一定是连续的，但是段内部内存空间是连续的；
> - 为了管理这个不连续的段，deque容器使用vector容器-- **map** --存储各个连续空间的首地址。map中存储的都是指针 `T*`；
> - 因为这种设计，deque的迭代器是比较特殊的：
>   - 能够定为具体数据  `T* cur`
>   - 能够定位到当前数据位于那个段内 `T** node ` （二级指针）
>   - 能够定位当前段的开始和结束位置 `T* first; T* last`
> - 因此，deque容器需要维护一个容器**map**；**start**和**finish**两个迭代器，在`push_front()` 和 `push_back()` 时起作用。
> - `start` 迭代器记录**map**数组中首个连续空间的信息，`finish` 迭代器记录**map**数组中最后一个连续空间的信息。此外，`start`  中 `cur` 指向的是连续空间中首个元素；而`finish` 中 `cur` 指向的是最后一个连续空间中最后一个元素的下一个位置。
> - 当`start`段内元素满时，`start->cur = start->first`；
> - 当`finish`段内元素满时，`finish->cur = finish->last`；
> - 优点：面对大量数据的情况下，避免像vector那样重新分配空间，再拷贝的过程；如果deque内数据达到当前上限，只需要对**map**重新分配空间执行拷贝即可，因此，可以避免对每个段内数据的拷贝。

<img src="/Users/apple/Documents/work2022/C++ knowledge/picture/deque.png" alt="内存结构" style="zoom:30%;" />

- deque的基本API

  ```c++
  /* 双端队列
  deque<int> dq;
  dq.front();
  dq.back();
  
  dq.push_back();
  dq.push_front();
  dq.pop_back();
  dq.pop_front();
  */
  ```

- queue 

  ```c++
  /* queue的底层就是deque，只是对部分API进行了封装和屏蔽
  queue<int> q;
  q.front();
  q.back();
  
  q.push(); 	// 入队 只能从队尾进入，相当于dq.push_back()
  q.pop();	// 出队，只能从队头离开，相当于dq.pop_front()
  
  */
  ```

- stack

  ```c++
  /* stack的底层也是deque，同样对部分API进行了封装和屏蔽
  stack<int> stk;
  stk.top();
  
  stk.push(); 	// 入栈：dq.push_back()
  stk.pop();	// 出栈：dq.pop_back()
  */
  ```




# map底层原理（set）——红黑树（RBTree）

- 红黑树性质：

  - 根节点为黑色
  - 叶子节点（无论红黑，**逻辑**上认为它的左、右子节点为空的黑节点NIL（**逻辑叶子**））
  - 红节点，左右孩子节点只能为黑节点
  - 从任意一个节点出发，到**其下的NIL节点**经过的黑节点个数相同

- 添加节点**c** ---- **cur**（默认每次新增的节点为**红色节点**）

  - 每次新增的节点只会在叶子节点（**物理叶子**）上；
  - 初次添加，树为空；当前节点变色
  - 父节点**p** ---- **parent**为黑色：直接添加（没有破坏红黑树性质）
  - p为红色（一定存在祖父节点**g** ---- **grandParent**且为黑，因为树根必须黑）
  - **（1）p**是**g**的左孩子
    - 叔父节点**u** ---- **uncle**存在且为红
      - **g**变红色，**p和u**变黑色；递归向上
    - u不存在或者存在且为黑（不存在在逻辑上就是**黑NIL**）
      - **c**是**p**的左孩子，**g**右旋，**p**变黑色变为当前的根，**g**变红色（结束，不需要递归）
      - **c**是**p**的右孩子，先对**p**左旋，变为上面的情况，做相同的操作
  - **（2）p**是**g**的右孩子 （就是情况1的逆向操作）
    - 叔父节点**u** ---- **uncle**存在且为红
      - 。。。
      - 。。。
    - u不存在或者存在且为黑（不存在在逻辑上就是**黑NIL**）

- 删除节点

  - 删除节点太复杂了 😭😭
  - [删除节点分类讨论](https://blog.csdn.net/qq_40843865/article/details/102498310)
    - 其：组合5&6中 (a)图中后继节点为黑色时，应转为组合2或4

- 实现思路：

  ```c++
  // RBTree.h
  // 只含有插入情况：
  enum COLOR
  {
      RED,
      BLACK
  };
  
  template<class K, class V>
  struct RBTreeNode
  {
      RBTreeNode(const K& _key, const V& _val, COLOR _color = RED)
          : key(_key), value(_val), color(_color), left(nullptr), right(nullptr), parent(nullptr)
      {}
      K key;
      V value;
      COLOR color;
      RBTreeNode<K, V>* left;
      RBTreeNode<K, V>* right;
      RBTreeNode<K, V>* parent;
  };
  
  template<class K, class V>
  class RBTree
  {
  public:
      typedef RBTreeNode<K, V> Node;
      typedef RBTreeNode<K, V>* pNode;
  
      RBTree() : root(nullptr) {}
  
      void Insert(const K& key, const V& val)
      {       
          if (root == nullptr)
          {
              root = new RBTreeNode<K, V>(key, val, BLACK);
              return;
          }
          pNode parent = nullptr;
          pNode cur = root;
          while (cur) {
              if (cur->key > key) parent = cur, cur = cur->left;
              else if (cur->key < key) parent = cur, cur = cur->right;
              else return;	// 插入失败，可以通过返回值修改
          }
  
          cur = new RBTreeNode<K, V>(key, val);
          if (key < parent->key) parent->left = cur;
          else parent->right = cur;
          cur->parent = parent;
  
          while (parent && parent->color == RED)
          {
              pNode grandP = parent->parent;
              if (parent == grandP->left)
              {
                  pNode uncle = grandP->right;
                  if (uncle && uncle->color == RED)
                  {
                      parent->color = BLACK;
                      uncle->color = BLACK;
                      grandP->color = RED;
  
                      cur = grandP;
                      parent = grandP->parent;
                  }
                  else
                  {
                      if (cur == parent->right)
                      {
                          RotateL(parent);
                          swap(cur, parent);
                      }
                      RotateR(grandP);
                      parent->color = BLACK;
                      grandP->color = RED;
                      break;
                  }
              }
              else
              {
                  pNode uncle = grandP->left;
                  if (uncle && uncle->color == RED)
                  {
                      parent->color = BLACK;
                      uncle->color = BLACK;
                      grandP->color = RED;
  
                      cur = grandP;
                      parent = grandP->parent;
                  }
                  else
                  {
                      if (cur == parent->left)
                      {
                          RotateR(parent);
                          swap(cur, parent);
                      }
                      RotateL(grandP);
                      parent->color = BLACK;
                      grandP->color = RED;
                      break;
                  }
              }
          }
          
        	// 这一步很重要
          root->color = BLACK;
      }
  
      pNode getRoot() const { return root; }
  
  private:
      void RotateL(pNode pParent)
      {
          pNode subR = pParent->right;
          pNode subRL = subR->left;
  
          subR->left = pParent;
          pParent->right = subRL;
          if (subRL) subRL->parent = pParent;
  
          pNode pParentP = pParent->parent;
          subR->parent = pParentP;
          pParent->parent = subR;
          if (pParent == root) root = subR;
          if (pParentP)
          {
              if (pParentP->left == pParent) pParentP->left = subR;
              else pParentP->right = subR;
          }
      }
  
      void RotateR(pNode pParent)
      {
          pNode subL = pParent->left;
          pNode subLR = subL->right;
          
          subL->right = pParent;
          pParent->left = subLR;
          if (subLR) subLR->parent = pParent;
  
          pNode pParentP = pParent->parent;
          subL->parent = pParentP;
          pParent->parent = subL;
          if (pParent == root) root = subL;
          if (pParentP)
          {
              if (pParentP->left == pParent) pParentP->left = subL;
              else pParentP->right = subL;
          }
      }
  
      pNode root;
  };
  
  template<class K, class V>
  void inOrder(RBTreeNode<K, V> *node)
  {
      if (node == nullptr) return;
      inOrder(node->left);
      cout << node->value << ' ';
      inOrder(node->right);
  }
  
  template<class K, class V>
  void preOrder(RBTreeNode<K, V> *node)
  {
      if (node == nullptr) return;
      cout << node->value << ' ';
      preOrder(node->left); 
      preOrder(node->right);
  }
  ```




# unordered_map

- 底层基于哈希表时间，内部是无序的
- 作为一种关联式容器，容器中元素类型`value type` 为 `std::pair` 
- 当通过 `key` 来查询 `value` 时，会有 `rehash` 操作

> unordered_map 内部采用哈希表的数据结构存储，容器中每个key会通过特点的哈希运算映射到一个特定的位置上；但是，哈希表是可能存在冲突的，即不同的key值经过哈希函数计算后得到相同的结果；
>
> 哈希表解决的冲突的方法主要有：**开放定址法和链表法两种**
>
> unordered_map采用链表法，每个位置放一个**桶（bucket）**，原来存放映射到此位置上的元素，同一个位置的元素会按照顺序链在后面；
>
> **bucket_vector**中每个元素对应一个桶（Java-HashMap 中当桶内元素个数载8以内时使用链表顺序链接，当元素个数超过8时自动转为红黑树结构）

- **桶相关成员函数**

  > **bucket(key);**	// 返回key值所在桶号
  >
  > **bucket_count();**	//返回容器中桶的个数
  >
  > **max_bucket_count();** 	//返回容器可以包含最大桶的数量
  >
  > **bucket_size(n);**	//返回给定桶号的桶中元素个数

- **负载因子load_factor**

  > **负载系数load_factor是容器中当前元素数量size()与当前桶的数量bucket_count的比值；**
  >
  > `load_factor = size/bucket_count`
  >
  > 负载系数会影响哈希表中发生冲突的概率：**负载系数越大，发生冲突概率越大；负载系数越小，发生冲突概率越小；**（但是过小的负载系统会造成额外的内存浪费，即大量的桶中没有存放元素）
  >
  > 因此，**容器会自动增加桶的数量，以保持将负载系数维持在特定的阈值（max_load_factor）以下**；每次扩充桶数的时候都会 `rehash()`；
  >
  > 可使用 `max_load_factor() `获取和设置最大负载系数；**默认阈值为1.0**。
  >
  > 1. `float max_load_factor() const noexcept;`			*//get-max_load_factor*
  > 2. `void max_load_factor(float z);`			*//set-max_load_factor*

- **reverse**

  > **std::map** 是没有这个成员函数的；
  >
  > `void reserve( size_type n );`		*// n 为元素数量*
  >
  > 将容器中的桶数设置为最适合的桶数量（包含至少n个元素）
  >
  > - 如果参数`n`大于`bucket_count*max_load_factor`，则增加容器的`bucket_count`并强制进行`rehash`操作。
  > - 如果参数`n`小于上述值，则函数可能无效；

- **rehash**

  > 重新设置桶的数量
  >
  > `void rehash(size_type n);`
  >
  > - 若参数`n`大于当前桶数（`bucket_count`），则将强制进行重建哈希表（`rehash`）
  > - 若参数`n`小于当前桶数，该函数可能对桶数没有影响，可能不会强制进行重建哈希表
  >
  > **当容器的负载系数大于阈值时，容器便会自动执行重建哈希表**



# priority_queue(堆、优先队列)的底层原理

- 底层容器一般都是vetor，默认是大根堆

  ```c
  namespace pri_que{
    template<typename T>
    struct less{
      bool operator()(const T& left, const T& right)
      {
          return left < right;
      }
    };
    
    template<typename T>
    struct greater{
      bool operator()(const T& left, const T& right) {
          return left > right;
      }
    };
    
    template<typename T, typename Container = vector<T>, typename Comp = less<T>>
    class Priority_queue{
    public:
    	void push(const T& x) {
      	c.push_back(x);
       	AdjustUp(c.size()-1);
      }
      
      void pop(){
         	swap(c[0], c[c.size()-1]);
        	c.pop_back();
        	AdjustDown(0);
      }
      
      T& top() {
        	return c[0];
      }
      
      size_t size() {
        	return c.size();
      }
      
      bool empty() {
        	return c.empty();
      }
      
    private:
      Container c;
      
      void AdjustUp(int cur) {
        Comp cmp;
        while(cur != 0) {
          int pa = (cur-1)/2;
          if(cmp(pa, cur)) {
            swap(c[pa], c[cur]);
            cur = pa;
          }
        }
      }
      
      void AdjustDown(int cur) {
        	int child = 2*cur + 1;
          Comp cmp;
          while (child < c.size()) {
              if (child+1 < c.size() && cmp(c[child], c[child+1])) {
                  ++child;
              }
              if (cmp(c[cur], c[child])) {
                  swap(c[child], c[cur]);
                  cur = child;
                  child = 2*cur + 1;
              }
              else break;
          }
      }
    };
  }
  ```
  
  

- priority_queue 基本API

  ```c++
  /*
  priority_queue<int, vector<int>, less<int>> bq; // 大根堆
  priority_queue<int, vector<int>, greater<int>> sq; // 小根堆
  
  bq.push();
  bq.top();
  bq.pop();
  */
  ```




# string底层实现

- **basic_string**
- string的**operator new**就是**placement new**，他申请了额外的空间

如何实现一个string，他的内存是什么样子的？

- [c++实现string](https://blog.csdn.net/qq_48083892/article/details/121547108)

  [手写实现C的常用面试函数](https://blog.csdn.net/Ternence_zq/article/details/114685453)

- strlen、strcpy：

```c++
// strlen
// C库的函数原型：size_t strlen(const char *str)
size_t strlen(const char *str)
{
	assert(str != NULL);
  size_t len=0;
  while((*str++) != '\0') len++; // 解引用和后置自增的优先级
  return len;
}

// strcpy
// C库的函数原型：char *strcpy(char *dest, const char *src)
char *strcpy(char *dest, const char *src)
{
  // 内存重叠会出错，解决内存重叠可以从后向前拷贝
  if (src == NULL || dest == NULL) return NULL;
  char *addr = dest;
  while(*src != '\0' ) {
    *addr++ = *src++;
  }
  return addr; //返回局部变量，是一种比较危险的操作
}

char *strncpy(char* dest, const char* src, size_t n)
{
    assert( (dest != NULL) && (src != NULL));
    char *address = dest;
    while ( n-- && (*src != '\0')) {
      *dest++ = *src++;
    }
    return addr;
}

// 模拟memcpy
// 就是将对象的指针转位char*，因为char就是按照字节拷贝
void* mymemcpy(void *dest, const void *src, size_t n)
{
    if (dest == NULL || src == NULL) {
      return NULL;
    }
    char *pDest = static_cast <char*>(dest);
    const char *pSrc  = static_cast <const char*>(src);
    if (pDest > pSrc && pDest < pSrc+n) {
        for (size_t i=n-1; i != -1; --i) {
          pDest[i] = pSrc[i];
        }
    }
    else{
        for (size_t i=0; i < n; i++){
        	pDest[i] = pSrc[i];
        }
    }

    return dest;
}

//提高拷贝的效率，按照四个字节去拷贝
void * mymemcpy(void *dest, const void *src, size_t n)
{
    if (dest == NULL || src == NULL)
          return NULL;
  	int cntInt = n/4;
  	int cntChar = n%4;
  	char* srcCh = (char*)src;
  	char* destCh = (char*)dest
    int* srcP = (int*) src;
  	int* destP = (int*) dest;
  
  	if (destCh > srcCh && srcCh+n > destCh) {
      for (int i=cntChar-1; i>=0; i--) {
        destCh[i]=srcCh[i];
      }
      
      for(int i=cntChar; i>=0; i--) {
        destP[i]=srcP[i];
      }
    }
  	else {
      while (cntCh--) {
        *destCh++ = *srcCh++;
      }
      
      srcP = (int*)srcCh;
      destP = (int*)destCh;
      while(cntInt--) {
       	*destP++ = *srcP++;
      }
    }
    return dest;
}
```



# B、B+数底层原理