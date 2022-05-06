# 动态规划

1、T1219 糖果

```c++
/*
输入：N（1-100）件产品，K（1-100）倍数
	 N行：每行表示第i件产品包含糖果数量（1-1000000）
输出：糖果总数（K的倍数），无满足条件输出0
*/
/*
分析：
    状态矩阵dp[i][j]:表示从前i件产品选择总数对k取余数为j时，最大糖果数量
    初始状态：dp[0][0]=0	(从0件产品中选择，总数只能为0且%k为0)
    状态转移方程：dp[i][j] = max{dp[i-1][j], dp[i-1][ (k + j - candy[i]%k) % k ] + candy[i]}
		dp[i][j]的来源有两种，一种为不选择第i件产品，则为dp[i-1][j]；另一种为选择第i件产品，则从dp[i-1][x]到		 dp[i][j]，从不选择candy[i] (sum%k=x) 到选择candy[i]后 （sum+candy[i])%k=j。然后比较两种来源取大者     目标解：dp[n][0]=target
*/		        
#include<iostream>
#include<algorithm>
using namespace std;
#define INF 100*1000000
int main(){
	int n, k;
	cin >> n >> k;
	int candy[105];
	for (int i = 1; i <= n; i++)	cin >> candy[i];
	int dp[101][100];
	dp[0][0] = 0;
	for (int j = 1; j<k; j++)	dp[0][j] = -INF;
	for (int i = 1; i <= n; i++){
		for (int j = 0; j < k; j++){
		    dp[i][j] = max(dp[i - 1][j], (dp[i - 1][(k + j - candy[i] % k) % k] + candy[i]));	
		}
	}
		
	if (dp[n][0]>0)
		cout << dp[n][0] << endl;
	else
		cout << 0 << endl;

	return 0;
} 
```

2、T1227 大盗阿福

```c++
/*
输入：T([1,50])组数据，每组数据分为两行输入；第一行为店铺个数N([1-100000])、第二行为店铺价值([0,1000])
输出：对于每组数据，可以得到的最大价值
约束：不可以获取相邻店铺的价值
*/
/*
分析：
容易想到设计状态矩阵dp[N+1]([1-N]),表示前i家店铺可以得到的最大价值
但是状态转移方程的时候容易陷入判断第i-1家店铺是否被选择的问题
正确的状态转移方程：不考虑第i-1家店铺是否被选择——dp[i] = max{dp[i-1], dp[i-2]+A[i]}
*/
#include<iostream>
#include<algorithm>
using namespace std;
int main(){
	int T, ans[51], k=1;
    cin>>T;
    while(T--){
        int N, a[100001], dp[100001];
        cin>>N;
        for(int i=1; i<=N; ++i)	cin>>a[i];
        dp[1] = a[1];
        dp[2] = max(a[1], a[2]);
        for(int j=3; j<=N; ++j){
            dp[j] = max(dp[j-1], dp[j-2]+a[j]); 
        }
        ans[k] = dp[N];
        k++;
    }
    for(int i=1; i<k; ++i){
        cout<<ans[i]<<endl;
    }
    return 0;
}
```

3、T1224 开餐馆

```c++
/*
输入：输入t，(t组数据,t[1,1000])，接下来每三行为一组数据
	 输入n，表示n个选择地点(1,100);输入k，表示间隔必须大于k
	 输入m[n+1]，表示地点相对位置(升序)
	 输入p[n+1],表示利润
输出：t行，对应每组数据的最大利润	
*/
/*
分析：容易想到dp[n+1];dp[i]表示从前i个地点中选择可以得到的最大利润
	 状态转移方程：
	 	(1) m[i]-m[i-1]>k
	 		dp[i] = dp[i-1] + p[i];
	 	(2) m[i]-m[i-1]<=k;
	 		dp[i] = max{dp[i-1], dp[j]+p[i]}；
	 		其中dp[i-1]表示不选择m[i],dp[j]+p[i]则是选择m[i];
	 		主要分析得到j的取值：要求满足m[i]-m[j]>k %% m[i]-m[j+1]<=k
	 初始状态：
	 	dp[0]=0;
	 	dp[1]=p[1];
*/
#include<iostream>
#include<algorithm>
using namespace std;
int main(){
	/*约束：t [1,1000]*/
	int t, ans[1001], c = 1;
	cin >> t;
	while (t--){
		/*约束：n (1,100)*/
		int n, k, m[100], p[100];
		cin >> n >> k;
		for (int i = 1; i <= n; ++i)	cin >> m[i];
		for (int j = 1; j <= n; ++j)	cin >> p[j];
		/*状态矩阵设计以及初始化*/
		int dp[100];
		dp[0] = 0, dp[1] = p[1];

		/*一次for循环，确定一组数据的dp[]*/
		m[0] = 0;	//辅助地点
		for (int i = 2; i <= n; ++i){
			/*如果第i个地点与i-1相隔距离大于k*/
			if (m[i] - m[i - 1]>k){
				dp[i] = dp[i - 1] + p[i];
			}
			/*距离相隔<=k，向前寻找第一个距离大于k的地点，或者距离全部<=k，定位到辅助地点m[0]*/
			else{
				int start;
				for (int p = i-1; p>=0; --p){
					/*m[i]-m[p]<=k，若i地点距离辅助地点不满足间隔>k，则设置起始点为辅助地点*/
					if (m[i] - m[p] > k || p==0){
						start = p;
						break;
					}
				}
				dp[i] = max(dp[i - 1], dp[start] + p[i]);
			}
		}
		/*为第c组数据确定可以得到的最大利润，为dp[n]，c从1到n*/
		ans[c] = dp[n];
		c++;
	}
	for (int i = 1; i<c; ++i)	cout << ans[i] << endl;

	return 0;
}
```

4、T1768搬书

```c++
/*
输入：n本书
	 n个整数，升序，a[i]表示一次搬i本书需要的体力
输出：搬n本书的最小体力
*/
/*
分析：容易设计状态矩阵dp[n]表示搬n本书需要的最小体力
	 且初始值dp[0]=0, dp[1]=a[1];
	 重点是状态转移方程：
	 	min = a[i];	//表示一次搬i本书需要体力设置为最小
	 	dp[i] = min{min, dp[j]+dp[i-j]}	//遍历比较
*/
#include<iostream>
#include<algorithm>
using namespace std;
int main(){
    /*约束：n [1, 5000]*/
    int n, a[5001];
    cin>>n;
    for(int i=1; i<=n; ++i)	cin>>a[i];
    int dp[5001], Min;
    dp[1]=a[1];
    for(int i=2; i<=n; ++i){
        Min=a[i];
        for(int j=1; j<i; ++j){
            Min=min(Min, dp[j]+dp[i-j]);
        }
        dp[i]=Min;
    }
    cout<<dp[n]<<endl;
    
    return 0;
}
```

5、T1551魔法少女

```c++
/*
输入：N[1,10000]，楼层数量
	 N行，每行表示一个楼层的高度
输出：攀登所有楼层最少使用时间
条件：可以一次跳跃一层或两层，但是必须爬一层以补充体力(单位高度消耗一秒)
*/
/*
分析：
	状态矩阵
	dp[i][0]表示前i层需要的最少时间且第i层使用跳跃
	dp[i][1]表示第i层需要的最少时间且第i层爬行
	dp[1][0]=0, dp[1][1]=a[1];	//边界
	dp[2][0]=0, dp[2][1]=a[2];	
	转移方程:
		dp[i][0] = min(dp[i-1][1], dp[i-2][1]);
		dp[i][1] = dp[i-1][0]+a[i];
*/
#include<iostream>
#include<algorithm>
using namespace std;
int main(){
    int n, a[10001], dp[10001][2];
    cin>>n;
    for(int c=1; c<=n; ++c)	cin>>a[c];
    /*状态矩阵边界值*/
    dp[1][0]=0;
    dp[1][1]=a[1];
    dp[2][0]=0;
    dp[2][1]=a[2];
    /*状态转移方程*/
    for(int i=3; i<=n; ++i){
        dp[i][1]=min(dp[i-1][0]+a[i], dp[i-1][1]+a[i]);
        dp[i][0]=min(dp[i-1][1], dp[i-2][1]);
    }
    int min_n = min(dp[n][0], dp[n][1]);
    cout<<min_n<<endl;
    
    return 0;
}
```

6、T1852杂务

```c++
/*
输入：
	n ([3,10000])表示n个杂务待完成
	每一行表示一个杂务的信息：序号 完成所需时间 前置项序号 0结尾
输出：
	完成n个杂务的最少时间
*/
/*
分析：
	dp[i]
*/
#include<iostream>
#include<algorithm>
using namespace std;
int main(){
	int n, index, time, ans[10001], x, maxans;
	cin >> n;
	for (int i = 1; i <= n; ++i){
		cin >> index;
		cin >> time;
		int tmp = 0;
		while (cin >> x && x){
			tmp = max(ans[x], tmp);
		}
		ans[i] = tmp + time;
		maxans = max(ans[i], maxans);
	}
	
	cout << maxans << endl;
	return 0;
}
```

7、T1722利润——最大子段和

```c++
/*
输入：n（表示利润个数 [1,100000]）
	 每行输入一个利润数 [-1000, 1000]
输出：连续的最大利润
分析1：
	状态矩阵dp[n] 表示前n个利润中可连续的最大利润值
	初始化：dp[1]=a[1],
		   dp[2]主要是分析a[i]是否被选择:
		   		如果不选择dp[2]=dp[2-1];
		   		如果选择：分为不放弃之前的选择；dp[2]=dp[j]+a[j]+a[j+1]+...+a[i]也就是dp[2]=dp[1]+a[2];
		   				放弃之前的选择：dp[2]=a[2];
	状态矩阵转移方程：
	dp[i]=max{x, y, z};
	x=dp[i-1]; y=dp[j]+a[j+1]+...+a[i]; z=dp[i];
分析2：
	状态矩阵dp[i]表示以i结尾的最大利润
	初始化：dp[1]=a[1]
	状态转移方程：
		dp[i]=max{d[i-1]+a[i], a[i]}
		遍历到dp[i]的时候判断dp[i-1]，若dp[i-1]<0,则dp[i]=a[i];反之dp[i]=dp[i-1]+a[i]
*/
#include<iostream>
#include<algorithm>
using namespace std;
int main(){
	int n, a[100001], dp[100001];
	cin >> n;
	for (int i = 1; i <= n; i++)	{
		cin >> a[i];
		dp[i] = a[i];
	}
	int ma = dp[1];
	for (int i = 1; i <= n; i++){
		dp[i] = max(dp[i - 1] + a[i], dp[i]);
		ma = max(ma, dp[i]);
	}
	cout << ma << endl;
	return 0;
}
```

8、T1850书本整理

```c++
/*
输入：n本书，抽出m本书
	n行：每一行表示两个数字：高度*宽度（高度不会重复）
	不整齐度：相邻书本宽度绝对值之和
输出：求n本书抽出m本之和最小的不整齐度
分析：
*/
#include<iostream>
#include<algorithm>
using namespace std;
int main(){
    int n, m, a[101][2];
    cin>>n>>m;
    for(int i=1; i<=n; ++i)	cin>>a[i][0]>>a[i][1];
}
```

9、T1900盒子与球（数学的排列组合问题。。。）

```c++
/*
输入：r个盒子，n个球
	约束：1 <= r <= n <=10
输出：n个球放入r个盒子中，在无空盒子的情况下，输出所有的可能
分析：
	dp[i][j]表示在j个盒子中放入i个球
	dp[i][j]=dp[i-1][j-1]+j*dp[i-1][j]
		i-1个球放入j-1个盒子中，第i个球放入第j个盒子；
		i-1个球放入j个盒子中，第i个球随机放入
	因为盒子有区别所以再乘以盒子数阶乘
*/
#include<iostream>
#include<algorithm>
using namespace std;
int fac(int n){
    if(n==0)
        return 1;
    int res=1;
    for(int i=1; i<=n; i++)
        res = res*i;
    return res;
}
int main(){
    int n,r,dp[11][11];
    cin>>n>>r;
    /*初始条件*/
    for(int i=1; i<=n; i++)
        dp[i][1]=1;
    for(int i=1; i<=r; i++)
        dp[i][i]=1;
    /*利用j遍历盒子*/
    for(int j=2; j<=r; j++){
        /*利用n遍历小球*/
        for(int i=j+1; i<=n; i++){
            dp[i][j]=dp[i-1][j-1]+j*dp[i-1][j];
        }
    }
    cout<<dp[n][r]*fac(r)<<endl;
}
```

