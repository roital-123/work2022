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
    ```

  - shrink_to_fit

    ```c++
    /*
    c++ 11后新增的这个函数，他的作用是释放掉未使用的内存，先使用clear将所有的元素清除（调用元素的析构函数），这样整块容器就变成未使用的，在调用shrink_to_fit函数吧未使用的部分内存释放掉即释放了整个vector
    */
    ```

- vector不能在作为基类

  - 析构函数非虚，在delete的时候会出错

- reverse和resize函数

  - reverse

    reverse函数只有在参数n大于当前容器的capacity时才会作用，通过空间分配器取到一块更大的内存，将数据拷贝至新内存处并释放原内存空间

  - resize(size_type _new_size, const _Tp& _x)

    1、小于当前size()：直接调用erase函数

    2、大于等于当前size()：insert(end(), _new_size()-size(), _x);

    ​	涉及到在末尾插入n个元素的情况：

    ​		a、剩余空间够用

    ​		b、剩余空间不够用（alloc在分配）

































