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

  