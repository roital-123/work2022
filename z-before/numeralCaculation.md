1、T1147 不与最大数字相同的数字之和

```c++
#include<iostream>
using namespace std;

int main(){
	int n;
	long data[100];
	cin >> n;
	for (int i = 0; i < n; i++){
		cin >> data[i];
	}

    //核心的思想设计
	int cnt = 0;
	long sum = 0, max=-1000001;
	for (int i = 0; i < n; i++){
		if (data[i] > max){
			sum = sum + max*cnt;
            max = data[i];
			cnt = 1;
		}
		else if (data[i] == max){
			cnt++;
		}
		else{
			sum += data[i];
		}
	}

	cout << sum << endl;
	return 0;
}
```

2、T1150 整数去重

```c++
#include<iostream>
#include<string>
using namespace std;

int main(){
	
	int n, data[20000];
	cin >> n;
	for (int i = 0; i < n; i++){
		cin >> data[i];
	}

	cout << data[0] << ' ';
	for (int i = 1; i < n; i++){
		//判断data[i]是否重复
		for (int j = 0; j < i; j++){
			//data[i]为重复元素
			if (data[j] == data[i]){
				break;
			}
			//检查到最后一个元素
			if (j == i - 1 && data[j] != data[i]){
				cout << data[i] << ' ';
			}
		}
	}
	return 0;
}
```

3、T1152 成绩排序（利用结构体）

```c++
#include<iostream>
#include<string>
using namespace std;

struct mark{
	string name;
	int grade;
};
int main(){
	
	int n;
	cin >> n;
	mark stu[20];
	for (int i = 0; i < n; i++){
		cin >> stu[i].name >> stu[i].grade;
	}

	for (int i = n-1; i > 0; i--){
		for (int j = n-1; j > n - 1 - i; j--){
			if (stu[j].grade > stu[j - 1].grade){
				mark stuTmp1;
				stuTmp1.name = stu[j].name; stuTmp1.grade = stu[j].grade;
				stu[j].name = stu[j - 1].name; stu[j].grade = stu[j - 1].grade;
				stu[j - 1].name = stuTmp1.name; stu[j - 1].grade = stuTmp1.grade;
			}
			else if (stu[j].grade == stu[j - 1].grade){
				if (stu[j].name < stu[j - 1].name){
					mark stuTmp2;
					stuTmp2.name = stu[j].name; stuTmp2.grade = stu[j].grade;
					stu[j].name = stu[j - 1].name; stu[j].grade = stu[j - 1].grade;
					stu[j - 1].name = stuTmp2.name; stu[j - 1].grade = stuTmp2.grade;
				}
			}
		}
	}

	for (int i = 0; i < n; i++){
		cout << stu[i].name << ' ' << stu[i].grade << endl;
	}
	return 0;
}
```

4、T1153 整数奇偶排序

```c++
int a[10000];
//from a to b-1 bubble sort (include a not include b, 0 =< a <= b < n) 
//冒泡次数：
	//下标从 a to b-1，共 ((b-1)-a)+1 个数字，需要 (b-1-a)+1-1 次冒泡
	//b-a-1次冒泡 i form 0 to b-a-2 ( int i=0; i<b-a-1; i++ 或 int i=0; i<=b-a-2; i++)  
//每次冒泡的循环比较：j form a to b-2-i ( int j=a; j<=b-2-i; j++ 或 int j=a; j<b-1-i; j++)
for(int i=0; i<=b-a-2; i++){
    for(int j=a; j<=b-2-i ; j++){
        //比较交换
    }
}
```

​	程序实现

```c++
#include<iostream>
#include<string>
using namespace std;

int main(){
	
	int data[10];
	for (int i = 0; i < 10; i++){
		cin >> data[i];
	}

	//奇数在前从大到小，奇数排序的核心
	for (int i = 9; i >0; i--){
		for (int j = 9; j > 9 - i; j--){
			//data[j]为奇数
			if (data[j] % 2 == 1){
				//向前比较，判断其奇偶性
				if (data[j - 1] % 2 == 1){
					if (data[j] > data[j - 1]){
						int temp = data[j];
						data[j] = data[j - 1];
						data[j - 1] = temp;
					}
				}
				else{
					int temp = data[j];
					data[j] = data[j - 1];
					data[j - 1] = temp;
				}
			}
			//data[j]为偶数
			else{
				continue;
			}
		}
	}
	
	//对偶数由小到大排序
	int index=-1;
	for (int i = 0; i < 10; i++){
		if (data[i] % 2 == 0){
			index = i;
			break;
		}
	}
    //偶数排序的核心，重点是起始点和终止点
	for (int i = index; i < 9; i++){
		for (int j = index; j < 9 - (i - index); j++){
			if (data[j] > data[j + 1]){
				int temp = data[j];
				data[j] = data[j + 1];
				data[j + 1] = temp;
			}
		}
	}
	

	for (int i = 0; i < 10; i++){
		cout << data[i] << ' ';
	}
	return 0;
}
```

5、T1156查找最接近元素

```c++
#include<iostream>
#include<algorithm>
using namespace std;

/*二分法查找第一个大于等于给定元素的值下标，若全部小于则输出最接近的那个（即最大的元素下标）*/
int binaryFind(int a[], int left, int right, int x){
	
	int rtmp = right;
	while (left <= right){
		int mid = left + (right - left) / 2;
		if (a[mid] == x){
			while (a[mid] == x){
				mid--;
			}
			mid++;
			return mid;
		}
		else if (a[mid] < x){
			left = mid + 1;
		}
		else{
			right = mid - 1;
		}
	}
	if (right == rtmp){
		return right;
	}
	else{
		return left;
	}
}

int main(){
	
	int n;
	cin >> n;
	int a[100000];
	for (int i = 0; i < n; i++)	cin >> a[i];
	int m;
	cin >> m;
	int b[10000];
	for (int j = 0; j < m; j++)	cin >> b[j];

    /*<algorithm>中sort算法的实现形式*/
	sort(a, a + n);
	for (int i = 0; i < m; i++){
		int tmp = b[i];
		int index = binaryFind(a, 0, n, tmp);
		if (a[index] == tmp){
			cout << a[index] << endl;
		}
		else{
			if (index > 0){
				int lvar = abs(tmp - a[index - 1]);
				int rvar = abs(a[index] - tmp);
				if (lvar <= rvar){
					cout << a[index - 1] << endl;
				}
				else{
					cout << a[index] << endl;
				}
			}
			else{
				cout << a[index] << endl;
			}
		}
	}

	return 0;
}
```

6、T1158和为给定数

```c++
#include<iostream>
#include<algorithm>
using namespace std;

int main(){

	int n, m, input[100000];
	cin >> n;
	for (int i = 0; i < n; ++i)	cin >> input[i];
	cin >> m;

	sort(input, input + n);
	int add1 = 0, add2 = 0;
	bool flag = false;
	for (int i = 0; i < n; ++i){
		add1 = input[i];
		if (add1 > m / 2)	break;
		for (int j = n - 1; j > i; --j){
			if (input[j] < m / 2)	break;
			if (input[j] == m - add1){
				add2 = input[j];
				flag = true;
				break;
			}
		}
		if (flag)	break;
	}
	
	if (flag)	cout << add1 << ' ' << add2 << endl;
	else	cout << "No" << endl;
	return 0;
}
```

