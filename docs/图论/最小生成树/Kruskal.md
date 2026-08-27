## Kruskal 算法
Kruskal 是一种基于**贪心**的思想、用**并查集**维护，非常好写的一种最小生成树的算法。

## 核心思想
核心思想是维护一个`森林（不要求联通的无环图）`，将两个**不在同一棵树**上的节点连接，最终形成最小生成树。

## 算法流程
1. 先将所有边按照边权大小排序，同时为每一个节点建树。
2. 以此遍历排序后的边 $E_i = (u_i, v_i, w_i)$，对于不在同一个祖宗下的 $u_i$ 和 $v_i$ 进行合并操作。

**总时间复杂度**：`O(m log m + m · α(n))`，通常简写为 **`O(m log m)`** 其中 `α(n)` 为反阿克曼函数，近似常数

## 代码实现 by @TB__QGSS__423
```cpp
#include<bits/stdc++.h>
#define ll long long
using namespace std;
const int N=2e5+5;
int n,m;
int bcj[5005];
struct node{
	int len,x,y;
}t[N];
bool cmp(node a,node b){
	return a.len<b.len;
}
int find(int i){
	return bcj[i]==i ? i : bcj[i]=find(bcj[i]);
}
int main(){
	cin>>n>>m;
	for(int j=1; j<=m; j++){
		cin>>t[j].x>>t[j].y>>t[j].len;
	}
	sort(t+1,t+m+1,cmp);
	for(int j=1; j<=n; j++) bcj[j]=j;
	int cnt=0,ans=0;
	for(int j=1; j<=m; j++){
		int le=find(t[j].x),ri=find(t[j].y);
		if(le!=ri){
			bcj[le]=ri;
			cnt++;
			ans+=t[j].len;
		}
		if(cnt==n-1){
			cout<<ans;
			return 0;
		}
	}
	cout<<"orz"; // 图不连通
	return 0;
}
```