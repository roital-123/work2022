# C/C++头文件的引用问题

- **例如**：

  工程下文件夹

  ```c++
  base_：
  	func3.h;
  	func3.c;
  	main_:
  		main.c;
  		func1.h;
  		func1.c;
  		func2_:
  			func2.h;
  			func2.c;
  	func4_:
  		func4.h;
  		func4.c;
  ```

  - main.c引用头文件func1.h：处于同一文件夹下

    **#include “func1.h”**

  - main.c引用头文件func2.h：处于平行子文件夹下

    **#include "func2\func2.h"**

  - main.c引用头文件func3.h：处于main的上级文件夹下

    **#include "..\func3.h"**

  - main.c引用头文件func4.h：处于main的上级文件夹的下一级文件夹

    **#include "..\func4\func4.h"**

- **注**：

  文件引用中：'\\'和'/'功能一致；在字符串中前者为转义字符，而'/a/b'与"\\\a\\\b"等价

  #include加载头文件时，"./"表示当前目录；"../"表示当前目录的上一级目录

  