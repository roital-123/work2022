### list列表

```c++
#include <list> // 头文件
// 常见初始化
list<int> l1; // 初始化为空
list<int> l2(10, 6); //初始化10个为6的int元素
list<int> l3(l2.begin(),--l2.end()); //迭代器指针指向的位置之间元素，注意--需要为前置操作符
list<int>::iterator it = l2.begin(); // 迭代器

l2.push_back(2);
l2.push_front(2);
l2.pop_back();
l2.pop_front();

l2.erase(it); // 删除迭代器指向的元素
```



### map

```c++
#include <map> // 头文件
map<int, string> ID_Name;

// 插入方法
map<int, string> ID_Name = {{2015, "jim"}, {2016, "tom"}}; // C++11开始
// []插入，如果键已经存在，则会修改
ID_Name[2015] = "bob";
// insert插入
// 1、插入单个键值对，返回插入位置和成功标志
pair<iterator, bool> insert(const value_type& val); // 返回类型是pair，value_type是pair类型
ID_Name.insert(pair<int, string>(2017, "tony"));
ID_Name.insert(map<int, string>::value_type(2017, "niko"));
ID_Name.insert(make_pair(2018, "rose"));
```



### pair

结构体实现

```c++
#include <utility>

template<class T1, class T2> 
struct pair
{
    T1 first;
    T2 second;
    ......
};
```

































