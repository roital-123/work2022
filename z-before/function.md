# c++通用函数解析

```c++
1、c_str()
    /*
    原型
    标准库string类提供的从一个string得到c类型的字符数组
    返回 const char* 指向c字符串的指针，内容与string类的本身对象一致
    注：
    	1、一定要使用strcpy()函数操作c_str() 返回的指针
    		char c[20];
    		string s="1234";
    		strcpy(c, s.c_str());
    	2、
    		char* c;
    		c = s.c_str();
    		这样c指向的内容是垃圾，因为s对象被析构，c_str()返回的是一个临时指针，不能对其操作
    */
    const char* c_str() const;	//返回一个以NULL终止的c字符串

 
```

```c++
2、strcpy_s(); 与 strcpy();
/*
1、strcpy()原型
	char* strcpy(){
		char Destination;
		char const* Source;
	}
	注： Destination的长度为 strlen(Source)+1
2、strcpy_s()原型
	strcpy_s(){
		char* Destination;
		int sizeBytes;
		char const* Source;
	}
	注：一般在获取字符串长度的时候strlen(Source)+1		把结尾的/0结束字符加上
*/
```

```c++
3、strcmp() 与 compare()
    /*
    1、strcmp()原型
    	int strcmp(const char* _Str1, const char* _Str2);
    	两个字符串自左向右逐个字符比较（按照ASCII值大小比较），直到出现不同的字符（开始比较ASCII值）或遇到'/0'为止
    	
    2、compare()原型
    	compare用于比较两个字符串是否相等
    	str1.compare(str2);		相等输出0， 不等输出-1
    	用法：
    	//condition 为：如果str1与abc相等，则执行
    	if(str1.compare("abc") == 0){......}
    	if(!str1.compare("abc")){......}
    	
    	//重载：
    		1、if(str1.compare(0,4,str2)==0){......}	str1从索引为0，开始4个字符与str2比较
    		2、if(str1.compare(0,4,str2,0,4)==0){......}	str2也是子串比较
    		3、if(str1.compare(0,4,"abcdefg",4)==0){......}	与指定字符串的前4个比较
    */
```

```c++
4、mkdir()
```

```
5、time(NULL) 与 localtime()
```

```c++
6、sort()	/* include<algorithm> */    
```

```c++
7、memset()	/* include<string> */
   /*
   为新申请的内存做初始化工作
   */
   //原型，用ch的值或者对应的ASCII码值初始化s指向内存的前n个字节
   //int c;	该值以int形式传递，但是函数在填充内存块时是使用该值的无符号字符形式
   memset(void *s, int ch, int n);
   char ch[10];
   memset(ch, 67, 10);	//ch[i] = 'C' 没有问题

   int num[10] = { 0 };
   memset(num, 1, 10);	//num[i] = 16843009 （i=0，1，2，3）
   /*原因是：memset赋值是按照字节赋值的，int类型占四个字节，num一共40个字节，所以num[0]的每个字节赋值01，最后转      换成十进制为16843009
   */
```

