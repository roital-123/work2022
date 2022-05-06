DP解法

1、最长公共子串

```c++
#include<iostream>
#include<algorithm>
#include<string>
using namespace std;

int main(){

	string s1, s2;
	cin >> s1 >> s2;
	int l1 = s1.length();
	int l2 = s2.length();
	int maxlen = 0;
	int dp[50][50];
	for (int i = 0, j = 0; i < l1; i++){
		if (s1[i] = s2[j]){
			dp[i][j] = 1;
			maxlen = 1;
		}
		else{
			dp[i][j] = 0;
		}
	}
	for (int i = 0, j = 0; j < l2; j++){
		if (s1[i] = s2[j]){
			dp[i][j] = 1;
			maxlen = 1;
		}
		else{
			dp[i][j] = 0;
		}
	}

	for (int i = 1; i < l1; i++){
		for (int j = 1; j < l2; j++){
			if (s1[i] == s2[j]){
				dp[i][j] = dp[i - 1][j - 1] + 1;
				maxlen = max(maxlen, dp[i][j]);
			}
			else{
				dp[i][j] = 0;
			}
		}
	}

	cout << maxlen << endl;

	return 0;
}
```

2、最长公共子序列

```c++

```

