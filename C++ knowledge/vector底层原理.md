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

  



































