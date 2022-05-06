1、T1123 字符串最大跨距

```c++
#include<iostream>
#include<string>
#include<algorithm>

using namespace std;
string s;//主字符串
string s1;//字符串s1
string s2;//字符串S2
int lefts1;//记录从左到右第一个s1字串的位置
int rights2;//记录从右到左第一个s2字串的位置
int main()
{
	getline(cin, s, ',');//输入主字符串
	getline(cin, s1, ',');//子字符串s1
	getline(cin, s2);//字字符串s2
    
	int ls = s.size();//获取字符串长度
	int ls1 = s1.size();
	int ls2 = s2.size();
	if (s.find(s1) != s.npos)//如果顺序查找到s1的位置，记录
	{
		lefts1 = s.find(s1);
	}
	else//否则输出-1，结束
	{
		cout << "-1" << endl;
		return 0;
	}
	reverse(s.begin(), s.end());//对s和s2进行反转，倒序查找s2；
	reverse(s2.begin(), s2.end());
	if (s.find(s2) != s.npos)//如果查找到s2的位置，记录
	{
		rights2 = s.find(s2);
	}
	else//否则输出-1，结束
	{
		cout << "-1" << endl;
		return 0;
	}
	if ((ls - rights2 - ls2) - (lefts1 + ls1) < 0)//如果s2在s1的右边，输出-1
	{
		cout << "-1" << endl;
	}
	else
		cout << (ls - rights2 - ls2) - (lefts1 + ls1) << endl;//否则输出间距
}
```

2、T1130 所有回文子串

```c++
#include<iostream>
#include<algorithm>
#include<string>
using namespace std;

int main(){

	string s;
	cin >> s;
	int l = s.length();

	//遍历字串，从长度2至l-1
	for (int i = 2; i < l; i++)
	{
		//在子串长度确定的条件下，遍历整条字符串
		for (int j = 0; j < l - i + 1; j++)
		{
			int flag = 1;
			for (int m = j, n = j + i - 1; m <= n; m++, n--)
			{
				if (s[m] != s[n])
				{
					flag = 0;
					break;
				}
			}
			if (flag == 1)
			{
				//输出满足条件的回文子串
				for (int k = j; k < j + i; k++)
				{
					cout << s[k];
				}
				cout << endl;
			}	
		}
	}
	return 0;
}
```

3、T1149 连续出现的字符

```c++
#include<iostream>
#include<string>
using namespace std;

int main(){
	int k, len, flag=1;
	cin >> k;
	string s;
	cin >> s;
	len = s.length();
	
	char ch;
	for (int i = 0; i < len; i++){
		ch = s[i];
		for (int j = i; j < i+k; j++){
			if (s[j] != ch){
				flag = 0;
				break;
			}
			if (j == i + k - 1  && s[j]==ch){
				flag = 1;
			}
		}
		if (flag == 0){
			continue;
		}
		else{
			break;
		}
	}
	if (flag == 0){
		cout << "No" << endl;
	}
	else{
		cout << ch << endl;
	}
	return 0;
}
```

