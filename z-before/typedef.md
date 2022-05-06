- 基本数据类型

```
typedef int size_type;
size_type size;
```

- 数组类型

```
typedef int score[100]; 
//s表示一个容量为100的数组，s是指向数组第一个元素值的指针
score s;
//name是字符串类型
//为了防止访问越界，字符串长度最长19，最后因为需要留给'\n'
typedef char name[20];
cin>>name;
cout<<name;
```

- 指针类型

```
typedef int* int_ptr; 
//int_ptr_arr表示指针数组类型，一个数组里面存放的是指针
typedef int* int_ptr_arr[5];
//int_arr_ptr表示数组指针类型，一个指向数组的指针
typedef int (*int_arr_ptr)[5];
```

- 函数类型

```
//max_int为函数类型
typedef int (max_int)(int a, int b);
//max_double为函数指针类型
typedef int (*max_double)(double a, double b);
```

- 声明typedef方法总结：

按照定义方法写出定义形式，

将变量名（函数名）替换为新类型名

在定义式前面加上typedef，即完成

- 测试代码：

```c++
#include<iostream>
#include<string>
using namespace std;

typedef int size_type;

typedef int score[100];
typedef char name[10]; 

typedef int* int_ptr;
typedef int* int_ptr_arr[5];
typedef int (*int_arr_ptr)[5];

typedef int max_min(int a, int b);
typedef int (*max_or_min_fun_ptr)(int a, int b);

int max(int a, int b) {

	return a > b?a : b;
}

int min(int a, int b) {
	return a > b ? b : a;
}

int main()
{

	/* 01-基本数据类型
	size_type size;
	cin >> size;
	cout << size << endl;
	*/
	 
	/* 02-数组类型
	score s;
	for (int i = 0; i < 5; i++)
		cin >> s[i];
	cout << *s << endl;
	*/

	/* 03-字符串类型（字符数组）
	char color[5];
	cin >> color;
	cout << color;

	name n;
	cin >> n;
	cout << n;
	*/


	/* 04-基本指针类型
	int_ptr p;
	int a;
	cin >> a;
	p = &a;
	cout << p << ' ' << *p << endl;
	*/

	/* 05-指针数组
	int_ptr_arr pa;
	int a;
	cin >> a;
	pa[0] = &a;
	cout << *pa[0];
	*/

	/* 06-函数类型
	int a, b;
	cin >> a >> b;
	max_or_min_fun_ptr m;
	max_min *fun_type;

	m = max;
	cout << m(a, b) << endl;
	m = min;
	cout << m(a, b) << endl;

	fun_type = max;
	cout << fun_type(a, b) << endl;
	fun_type = min;
	cout << fun_type(a, b) << endl;
	*/

	return 0;
}
```

