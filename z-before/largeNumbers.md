# 2020-7-25

cin读取字符串在遇到空格或换行时，读取结束，如果读取一整行字符串可以使用getline(cin, s);

大数加法：

```c++
#include<iostream>
#include<algorithm>
#include<string>
using namespace std;
int main(){
    
    string s1,s2;
    cin>>s1>>s2;
    int a[201]={0}, b[201]={0}, res[201]={0};
    //0~l1-1 and 1~l2-1
    int l1=s1.length();
    int l2=s2.length();
    for(int i=0; i<l1; i++){
        a[i]=s1[l1-1-i]-'0';
    }
    for(int j=0; j<l2; j++){
        b[j]=s2[l2-1-j]-'0';
    }
    int len=max(l1,l2);
    for(int i=0; i<len; i++)
        res[i]=a[i]+b[i];
    for(int i=0; i<len; i++){
        res[i+1] += res[i]/10;
        res[i] = res[i]%10;
    }
    while(res[len]!=0){
        res[len+1] += res[len]/10;
        res[len] = res[len]%10;
        len++;
    }
    while(res[len]==0 && len>=0)
        len--;
    if(len==-1)
        cout<<0;
    else{
        for(int i=len; i>=0; i--)
            cout<<res[i];
    }
    return 0;
}
```

大数减法：

```c++
#include<iostream>
#include<string>
#include<algorithm>
using namespace std;

bool cmp(string a, string b){
    if(a.length()!=b.length())
        return a.length()>b.length();
    else
        return a>b;
}

int main(){
    bool sgn=true;
    string s1, s2;
    int len1, len2, a[201]={0}, b[201]={0}, res[201];
    cin>>s1>>s2;
    if(!cmp(s1,s2)){
        sgn=false;
        swap(s1,s2);
    }
    len1 = s1.length();
    len2 = s2.length();
    for(int i=0; i<len1; i++)
        a[i] = s1[len1-1-i]-'0';
    for(int i=0; i<len2; i++)
        b[i] = s2[len2-1-i]-'0';
    
    for(int i=0; i<len1; i++){
        res[i]=a[i]-b[i];
    }
    for(int i=0; i<len1; i++){
        if(res[i]<0){
            res[i] = res[i]+10;
            res[i+1] = res[i+1]-1;
        }
    }

    while(res[len1-1]==0 && len1>1) len1--;
    if(!sgn){
        cout<<'-';
    }
    for(int i=len1-1; i>=0; i--){
        cout<<res[i];
    }
    
    return 0;
}
```

