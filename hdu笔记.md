[TOC]

## 字符串

### 画8

HDU 1256

http://acm.hdu.edu.cn/showproblem.php?pid=1256

可以直接string(num, ch)创建临时字符串，不用写变量名。

 此外，多个输入案例中，最后一个案例在while(n--)后n是变为0而不是1

```
#include <iostream>
#include <cstdio>
#include <vector>
#include <cstring>
#include <string>
#include <map>

using namespace std;

int main(){
    int n;
    cin>>n;
    char ch;
    int h, thick, downh;
    while(n--){
        cin>>ch>>h;
        thick = h/6+1;
        downh = h-1-(h-1)/2-1;  //下圈高度
        string s1 = string(thick, ' ')+string(downh, ch);
        string s2 = string(thick, ch)+string(downh, ' ')+string(thick, ch);
        for(int i=0; i<h; i++){
            if(i==0 || i==(h-1)/2 || i==h-1)
                cout<<s1<<endl;
            else{
                if(i==h-1) cout<<s2;
                else cout<<s2<<endl;
            }
        }
        if(n)
            cout<<endl;
    }
    return 0;
}

/*
2
A 7
B 8
 */

```



### Flying to the Mars

**HDU 1800**

http://acm.hdu.edu.cn/showproblem.php?pid=1800

一开始没用hash，超时并且数组指针溢出。超时应该也是由于cin。

依然是**BKDRHash**，我本来想的是像桶一样分配一个很大很大的数组，然后对应的level自增。但是太耗内存了。最后改成了直接顺序存储，然后统计HashT[i] == HashT]i-1]的个数。

（当然直接用map存储字符串也能水过：[点击跳转](http://acm.hdu.edu.cn/viewcode.php?rid=33538342)）

另外，本题用cin输入字符串会超时，我改成了scnaf输入char[]。注意 char数组中strlen(str)和我string中常用的获取长度是一样的。

```
#include <iostream>
#include <cstdio>
#include <cstring>
#include <vector>
#include <queue>
#include <string>
#include <algorithm>
#include <map>
#include <cmath>

using namespace std;
const int MAXN = 3010;
int seed = 131;
int hashT[MAXN];

int BKDRHash(char* s){
    long long res = 0;
    int i = 0;
    while(s[i]=='0')  //去除前导0
        i++;
    for(;i<strlen(s); i++){
        res = res*seed + (s[i]-'0');
    }
    return (res & 0x7fffffff);
    
}

int main(){
    int n;
    while(cin>>n){
        memset(hashT, 0, sizeof(hashT));
        char level[32];
        int maxx = 1;
        
        for(int i=0; i<n; i++){
            scanf("%s", level);
            hashT[i] = BKDRHash(level);
        }
        sort(hashT, hashT+n);
        int number = 1;
        for(int i=1; i<n; i++){
            if(hashT[i]==hashT[i-1]){
                number++;
                if(number>maxx)
                    maxx = number;
            }
            else number = 1; //碰见不一样的number还原为1
        }
        cout<<maxx<<endl;

    }
}

/*
4
10
20
30
04
5
2
3
4
3
4
 */

```



## DFS

### Tempter of the Bone

**HDU 1010**

http://acm.hdu.edu.cn/showproblem.php?pid=1010

这一题用BFS会内存溢出。

要看题！！ 是步数（层数）恰好等于t时才能成功，之前瞎了以为小于等于t就行。

**奇偶剪枝：**

> 设起点坐标是bx，by；终点坐标是ex，ey；最短路径是abs（bx-ex）+abs（by-ey）
>
> 但是我们可以很显然的看出来，无论是走，我们的步数肯定比最短路径数多一个偶数
>
> 所以，简化的第一点来了：如果题目中输入的步数比最短路径数多得是一个奇数，那么我们永远也不可能走到，所以直接输出NO；



```
#include <iostream>
#include <queue>
#include <cstdio>
#include <string>
#include <cstring>
#include <cmath>
using namespace std;
const int MAXN = 8;

char maze[MAXN][MAXN];
int dir[4][2]={{-1,0},{1,0},{0,-1},{0,1}};//上下左右
int n,m, t;
int startX, startY;
int doorX, doorY;
int visited[MAXN][MAXN];

bool dfs(int x, int y, int layer){
    bool flag = false;  //默认找不到
    visited[x][y] = true;  //先把自己访问
    if(x==doorX && y==doorY && layer==t){//找到了，设置flag为true.  注意是layer恰好等于t！！！！！！！
        flag =true;
        return true;
    }
    if(layer<t){//保证下一层不会过深
        for(int i=0; i<4; i++){//继续向四个方向深度搜索
            int newX =x+dir[i][0];
            int newY =y+dir[i][1];
            
            if(newX<0 || newX>=n ||newY<0 || newY>=m ||visited[newX][newY] ||maze[newX][newY]=='X'){//越界、已访问、被挡住 这些情况下不搜索
                continue;
            }
            flag = dfs(newX, newY, layer+1);  //继承结果
            if(flag)  //如果找到一个解，直接break
                break;
        }
    }
    visited[x][y]=false; //取消访问
    return flag;
}

int main(){
    
    while(cin>>n>>m>>t && n && m &&t){
        bool flag = false; //默认失败
        //设置边界
        for(int i=0; i<n; i++)
            maze[i][m]='X';
        for(int j=0; j<m; j++)
            maze[n][j]='X';
        
        for(int i=0; i<n; i++){
            for(int j=0; j<m; j++){
                visited[i][j] = false;
                cin>>maze[i][j];
                if(maze[i][j]=='S'){
                    startX = i;
                    startY = j;
                }
                if(maze[i][j]=='D'){
                    doorX= i;
                    doorY= j;
                }
            }
        }
        int minLen = abs(doorX-startX) + abs(doorY-startY);
        if((t-minLen)%2){  //奇偶剪枝。 到达终点的步数一定是曼哈顿距离+偶数，如果目标t与曼哈顿距离相差奇数，直接失败
            cout<<"NO"<<endl;
        }
        else{
            flag = dfs(startX, startY, 0);

            if(flag)
                cout<<"YES"<<endl;
            else cout<<"NO"<<endl;
        }
        
    }
    return 0;
}

```



### Prime Ring Problem

**HDU 1016**

http://acm.hdu.edu.cn/showproblem.php?pid=1016

素数表+DFS，不难

我感觉我的DFS形式变了。。 

以前是：

```
visited[i] = true;
dfs(i, .., ..);
visited[i] = false;
```

现在是dfs开始之后，在函数首行设置visited[i] = true;在return之前设置visited[i] = false; 好像都行

还有判断是否能搜索，我现在习惯在搜索前就判断，下一级不满足条件就不搜它（在BFS里是不入队），以前我是先放进去，在下一级开始是判断，如果不符合就continue

```
#include <iostream>
#include <queue>
#include <cstdio>
#include <string>
#include <cstring>
#include <cmath>
using namespace std;
const int MAXN = 21;
bool visited[MAXN];
bool isPrime[2*MAXN];
int n;
int num[MAXN]; //存放结果输出

//素数表
void init(){
    double bound = sqrt(MAXN);
    isPrime[1]=false;
    for(int i=2; i<2*MAXN; i++)
        isPrime[i] = true;  //先默认都是素数
    
    for(int i=2; i<2*MAXN; i++){
        if(!isPrime[i]){//如果不是素数，继续
            continue;
        }
        if(i>=bound)  //i*i越界
            continue;  //换成break也行
        for(int j=i*i; j<2*MAXN; j+=i){
            isPrime[j] = false;
        }
    }
    return ;
}

void dfs(int current, int pos, int* num){//前面的数，当前数，当前位置
    visited[current] = true; //访问当前数字
    num[pos] = current;

    if(pos==n){//搜索到最后了
        if(isPrime[current+1]){//只需要和首位相加看是不是素数
            for(int i=1; i<=n; i++){
                if(i==1)
                    cout<<num[i];
                else cout<<" "<<num[i];
            }
            cout<<endl;
        }
    }
    for(int i=1; i<=n; i++){
        if(!visited[i] && isPrime[current+i]){//没有被访问并且和前面的和是素数
            dfs(i, pos+1, num);
        }
    }
    visited[current] = false; //取消访问
    
    return ;
}

int main(){
    
    init(); //素数表
    int times =0;
    while(cin>>n){
        times ++;
        //初始化
        for(int i=1; i<=n; i++){
            num[i] = 0;
            visited[i] = false;
        }
        cout<<"Case "<<times<<":"<<endl;
        dfs(1,1, num); //假设第一位前的数字是0，从数字1开始搜索，位置也是1
        cout<<endl;
    }
}

```





### 连连看

http://acm.hdu.edu.cn/showproblem.php?pid=1175

DFS+剪枝

提交了11次！！！！各种错误往[这儿](http://acm.hdu.edu.cn/status.php?first=&pid=1175&user=Lenvia&lang=0&status=0)看

DFS在判断终点时，**尽量使用限定正确结果的条件，而不是排除错误的条件。（比如这里应该用横纵坐标对应来判断到目的地，而不是用当前地图的值来判定。因为可能中途会走到值相等但不是终点的位置）**

还是尽量把visited写在dfs调用的前后。

有时候多给函数加一个参数可以更方便剪枝！！！！！！！！！！！！！！！

```
#include <iostream>
#include <cstdio>
#include <cstring>
#include <map>
using namespace std;
const int MAXN = 1002;

int n,m;
int graph[MAXN][MAXN];
int visited[MAXN][MAXN];
int q;
int startX, startY, endX, endY; //查询坐标
int startP, endP;
char dir[4][2] ={{-1,0}, {1,0}, {0,-1}, {0,1}}; //上下左右

int flag;

void dfs(int cX, int cY, int cT, int turns){
    if(turns>2)
        return ;
    
    if(cX == endX && cY ==endY){//坐标对应
        flag =1;
        return;
    }
    
    else{
        for(int i=0; i<4; i++){//向四个方向搜索
            int newX = cX + dir[i][0];
            int newY = cY + dir[i][1];
            int addT = 0;
            
            //如果和上一步的方向不一样，转弯数+1
            if(cT!=-1 && cT!=i)
                addT=1;
            
            //越界 或者已经访问
            if(newX<0 || newX>=n || newY<0 || newY>=m || visited[newX][newY])
                continue;
            
            //剪枝。 转两次弯了，判断终点和当前点在不在一条直线上
            if(turns + addT==2){
                if(cX != endX && cY !=endY){  //注意 是还没有转移的点，而不是newX和newY
                    continue;
                }
            }
            
            //如果下一步是路或者是终点坐标，才能继续搜索。 注意是坐标相等，而不是值相等
            //坐标不相等但值相等，也相当于墙
            if(graph[newX][newY]==0 || (newX == endX && newY ==endY)){
                visited[newX][newY] =1;
                dfs(newX, newY, i, turns+addT);
                visited[newX][newY] =0;
                if(flag) return;
            }
        }
    }
    return ;
}

int main(){
    while(cin>>n>>m && n &&m){
        memset(graph, 0, sizeof(graph));
        for(int i=0; i<n; i++){
            for(int j=0; j<m; j++){
                cin>>graph[i][j];
            }
        }
        cin>>q;
        while(q--){
            cin>>startX>>startY>>endX>>endY;
            startX--; startY--; endX--; endY--;
            
            memset(visited, 0, sizeof(visited));
            startP = graph[startX][startY]; endP = graph[endX][endY];

            if(startP==0 || endP==0 ||(startP!=endP)){//如果起点终点有一个为0或者两个不相等
                cout<<"NO"<<endl;
                continue;
            }
            if(startX==endX && startY==endY){  //起始点坐标一样也不行
                cout<<"NO"<<endl;
                continue;
            }

            flag = 0;
            visited[startX][startY] = 1;
            dfs(startX, startY, -1, 0);
            
            if(flag)
                cout<<"YES"<<endl;
            else cout<<"NO"<<endl;
        }
    }
    return 0;
}
```



### Hat's Tea

**HDU 1288**

http://acm.hdu.edu.cn/showproblem.php?pid=1288

这道题和字符串没关系啊？ 我用过的DFS。 本来想的是DP，但是太麻烦了。直接暴力三层循环会超时。 这个DFS情况一定要理清楚。

```
#include <iostream>
#include <cstdio>
#include <vector>
#include <cstring>
#include <string>
#include <map>
#include <cmath>

using namespace std;

int num[3]; //输入数量
int value[3] = {1, 5, 10};
int amount[3];
int n;
int flag;
bool dfs(int pos, int current){
    if(pos==0){//能走到这说明一定有解
        amount[0] = n-current;
        if(amount[0]<0){
            return 0;
        }
        cout<<amount[0]<<" YiJiao, "<<amount[1]<<" WuJiao, and "<<amount[2]<<" ShiJiao"<<endl;
        return 1;
    }
    else{
        for(int i=0; i<=num[pos]; i++){  //这里是小于等于！！
            if(pos==2){
                if(i*value[pos]+num[1]*value[1]+num[0]*value[0]<n)  //接下来最大也不可能达到n
                    continue;
                amount[pos] = i;
                flag = dfs(pos-1, current+i*value[pos]);
                if(flag) return 1;
            }
            else if(pos==1){
//                cout<<amount[2]<<" "<<i<<endl;
                if(current+i*value[pos]+num[0]*value[0]<n) //接下来最大也不可能达到n
                    continue;
                amount[pos] = i;
                
                flag = dfs(pos-1, current+i*value[pos]);
                if(flag) return 1;
            }
        }
    }
    
    return 0;
}

int main(){
    while(cin>>n){
        int sum=0;
        for(int i=0; i<3; i++){
            cin>>num[i];
            sum+=num[i]*value[i];
        }
        if(!n && !num[0] && !num[1] &&!num[2]) break;
        if(sum<n){
            cout<<"Hat cannot buy tea."<<endl;
            continue;
        }
        if(sum==n){
            cout<<num[0]<<" YiJiao, "<<num[1]<<" WuJiao, and "<<num[2]<<" ShiJiao"<<endl;
            continue;
        }
        
        flag = 0;
        flag = dfs(2,0);
        if(!flag)
            cout<<"Hat cannot buy tea."<<endl;
    }
    return 0;
}

/*
23 2 5 5
6653 226 72 352
578 5 127 951
0 0 0 0
 */

```



## BFS

### Ignatius and the Princess I

**HDU 1026**

http://acm.hdu.edu.cn/showproblem.php?pid=1026

这一题[DFS](http://acm.hdu.edu.cn/viewcode.php?rid=33415610)超时，而且明确要求最短时间，当时没看到。

BFS记录状态太复杂了。。因为到达目标状态时，队列中只有最后一个节点的信息。所以要知道过程，需要用字符串记录中途操作。

我这题用了优先队列，由于需要打怪，我令 到达一个节点的当前时间=前一个节点 + 1 + 打怪时间。

输出那部分，，详见代码。

另外本题还有在string后添加一个char的操作。不能直接加，需要先把单个char ch转为string s(1, ch)，然后再拼接

```
#include <iostream>
#include <queue>
#include <cstdio>
#include <string>
#include <cstring>
#include <cmath>
#include <vector>
using namespace std;
const int MAXN = 102;

char maze[MAXN][MAXN];
int dir[4][2]={{-1,0},{1,0},{0,-1},{0,1}};//上下左右
int n,m;
int total; //总时间
int startX, startY;
int doorX, doorY;
int visited[MAXN][MAXN];


struct Info{
    int x;
    int y;
    int curT; //当前时间
    string times;//打怪时间集合
    string dirs;  //方向集合
    Info(int a, int b,int c, string t, string s): x(a), y(b), curT(c), times(t), dirs(s){}
    bool operator< (const Info& p) const{
        return curT>p.curT;
    }
};

//上面的Info就很，别扭。 因为用的BFS，所以到达终点只能得到最后一个状态的Info
//所以我设置了times, dirs表示前面操作的集合，但是这俩的长度是不一样的
//比如从(0,0)->(1,0)->(2,0)，times="002"，而dirs="11"
//由(x1,y1)->(x2,y2)
//正确逻辑是先到达终点再打怪，假设起点是第i个状态，下一个点是第i+1个状态
//所以我在由 dir[i]和x1,y1计算出x2,y2时，就使用了times[i+1]输出了(x2,y2)的打怪信息


bool bfs(int x, int y){
    bool flag = false;  //默认找不到
    visited[x][y] = true;  //先把自己访问
    
    priority_queue<Info>q;
    q.push(Info(x,y,0,"0", "")); //起点不会有怪物
    while(!q.empty()){
        Info temp = q.top();
        q.pop();
        
        if(temp.x==doorX && temp.y==doorY){//找到了，设置flag为true.
            //输出
            cout<<"It takes "<<temp.curT<<" seconds to reach the target position, let me show you the way."<<endl;
            string path = temp.dirs;
            string times = temp.times;
            int currentTime =0;
            
            int x1,y1,x2,y2;
            x1=0;y1=0;
            
            for(int i=0; i<times.length()-1; i++){
                int index = path[i]-'0';//方向
                x2 = x1+dir[index][0];
                y2 = y1+dir[index][1];
                
                //先输出状态转移
                currentTime++;
                cout<<currentTime<<"s:";
                cout<<"("<<x1<<","<<y1<<")"<<"->"<<"("<<x2<<","<<y2<<")"<<endl;
                
                //输出转移后的节点的打怪信息
                if(times[i+1]>'0'){
                    int k = times[i+1]-'0';
                    for(int j=0; j<k; j++){
                        currentTime++;
                        cout<<currentTime<<"s:";
                        cout<<"FIGHT AT "<<"("<<x2<<","<<y2<<")"<<endl;
                    }
                }
                
                //更新起点坐标
                x1 = x2;
                y1 = y2;
            }
            return true;
        }
        
        for(int i=0; i<4; i++){//继续向四个方向
            char extra = '0'; //打怪额外时间
            int newX =temp.x+dir[i][0];
            int newY =temp.y+dir[i][1];
            if(newX<0 || newX>=n ||newY<0 || newY>=m ||visited[newX][newY] ||maze[newX][newY]=='X'){//越界、已访问、被挡住 这些情况下不搜索
                continue;
            }
            if(isdigit(maze[newX][newY]))  //打怪时间
                extra = maze[newX][newY];
                
            //char不能直接转string，需要这么个操作
            char curD = i+'0';  //当前方向
            string d(1,curD);
            string dirs = temp.dirs+d;
            string t(1,extra); //当前打怪时间
            string times = temp.times+t;
            
            
            q.push(Info(newX, newY, temp.curT+1+(extra-'0'), times, dirs));
            visited[newX][newY]=true; //设置访问
        }
    }
    return flag;
}

int main(){
    while(cin>>n>>m&& n && m){
        
        total = 0;
        bool flag = false; //默认失败
        //设置边界
        for(int i=0; i<n; i++)
            maze[i][m]='X';
        for(int j=0; j<m; j++)
            maze[n][j]='X';
        
        for(int i=0; i<n; i++){
            for(int j=0; j<m; j++){
                visited[i][j] = false;
                cin>>maze[i][j];
            }
        }
        startX =0; startY=0; doorX = n-1; doorY = m-1;
        flag = bfs(startX, startY);

        if(flag){//有路
            //在函数里就已经输出了
        }
        else
            cout<<"God please help our poor hero."<<endl;
        cout<<"FINISH"<<endl;
        
    }
    return 0;
}

/*
5 6
.XX.1.
..X.2.
2...X.
...XX.
XXXXX.
 */
```



### <font color=red>Eight</font>

**HDU 1043**

http://acm.hdu.edu.cn/showproblem.php?pid=1043

我裂开了。这题BFS太坑了。超时、超内存。另外了解到“康托排列”。

[未通过的代码](http://acm.hdu.edu.cn/status.php?user=Lenvia&pid=1043&lang=0&status=0#status)



还没有来及看的代码

```
#include<iostream>
#include<cstdio>
#include<queue>
#include<cstring>
using namespace std;
const char direct[4] = { 'd','u','r','l' };
const int dir[4][2] = { {-1,0},{1,0},{0,-1},{0,1} }, N = 362880;
vector<char> path[N];
bool visit[N];
const int aim = 46233;
const int n = 9;

//阶乘
const int fac[n] = { 1,1,2,6,24,120,720,5040,40320 };
//康托展开
int cantor(int s[]) {
    int result = 0, cnt = 0;
    for (int i = 0; i < n - 1; ++i) {
        cnt = 0;
        for (int j = i + 1; j < n; ++j) {
            if (s[i] > s[j])++cnt;
        }
        result += fac[n - 1 - i] * cnt;
    }
    return result;
}
void reverseCantor(int hash, int s[], int &space) {
    bool visited[n] = {};
    int temp;
    for (int i = 0; i < n; ++i) {
        temp = hash / fac[n - 1 - i];
        for (int j = 0; j < n; ++j) {
            if (!visited[j]) {
                if (temp == 0) {
                    s[i] = j;
                    if (j == 0)space = i;
                    visited[j] = true;
                    break;
                }
                --temp;
            }
        }
        hash %= fac[n - 1 - i];
    }
}
//从终点逆向搜索，找到所有的可到达局面
void bfs() {
    queue<int> q;
    q.push(aim);
    visit[aim] = true;
    int state[n], space;
    while (!q.empty()) {
        int preHash = q.front();
        q.pop();
        reverseCantor(preHash, state, space);
        for (int i = 0; i < 4; ++i) {
            int tx = space / 3 + dir[i][0];
            int ty = space % 3 + dir[i][1];
            if (tx < 0 || tx>2 || ty < 0 || ty>2)continue;
            int tz = tx * 3 + ty;
            state[space] = state[tz];
            state[tz] = 0;
            int hash = cantor(state);
            if (!visit[hash]) {
                visit[hash] = true;
                q.push(hash);
                path[hash] = path[preHash];
                path[hash].push_back(direct[i]);
            }
            state[tz] = state[space];
            state[space] = 0;
        }
    }
}
int main() {
    bfs();
    char x;
    int a[n];
    while (~scanf(" %c", &x)) {
        if (x == 'x')a[0] = 0;
        else a[0] = x - '0';
        for (int i = 1; i < n; ++i) {
            scanf(" %c", &x);
            if (x == 'x')a[i] = 0;
            else a[i] = x - '0';
        }
        int hash = cantor(a);
        if (visit[hash]) {
            for (int i = path[hash].size() - 1; i >= 0; --i)printf("%c", path[hash][i]);
            printf("\n");
        }
        else printf("unsolvable\n");
    }
    return 0;
}
```





### <font color=blue>Gap</font>

**HDU 1067**

http://acm.hdu.edu.cn/showproblem.php?pid=1067

**BKDRHash** 字符串哈希算法。

https://blog.csdn.net/wanglx_/article/details/40400693

https://www.cnblogs.com/zl1991/p/11820922.html

（但是我直接用map<string, int> 也过了。。 就是时间有点长）http://acm.hdu.edu.cn/viewcode.php?rid=33424092

```
#include<stdio.h>
#include<queue>
#include<string.h>
#include <iostream>
using namespace std;

const int seed = 3 ;
const int mod = 1000000 + 3 ;

struct status{//状态
    int step ;//转移次数
    int curMap[4][8] ;//当前矩阵
};

int T;//案例个数
status ans, tmp ;

long long H[mod];  //桶

void bfs (status ans){
    queue<status> q;
    while (!q.empty ()) q.pop () ;
    
    memset (H , 0 , sizeof(H)) ;  //初始化哈希表
    
    ans.step = 0;
    q.push (ans) ;  //放入初始状态
    
    while (!q.empty ()) {
        ans = q.front () ;
        q.pop () ;
        
        bool flag = 1 ;
        //判断是否达到目标状态，即每一位都要保证ans.curMap[i][j] == (i + 1) * 10 + j + 1
        for (int i = 0 ; i < 4 && flag ; i ++)
            for (int j = 0 ; j < 7 && flag ; j ++)
                if (ans.curMap[i][j] != (i + 1) * 10 + j + 1)
                    flag = 0 ;
        
        if (flag) {//达到目标状态
            cout<<ans.step<<endl;
            return ;
        }
        //没有达到目标态
        //对当前状态ans，每次再产生4个状态（分别对四个空格进行转移）
        for (int i = 0 ; i < 4 ; i ++) {
            for (int j = 1 ;  j < 8 ; j ++) {
                tmp = ans;  //只是个临时的
                if (ans.curMap[i][j] == 0 ) {//当前位置是空格
                    int num = ans.curMap[i][j - 1] + 1 ;//num是比本空格左边的数大1的数
                    //不能移动任何卡片到值为7的卡片右边，也不能移动到空位的右边。
                    if ((num % 10 > 7 )|| (num % 10 == 1))
                        continue ;
                    
                    for (int s = 0 ; s < 4 ; s ++) {
                            for (int t = 1 ; t < 8 ; t ++) {
                                if (ans.curMap[s][t] == num) {//找到了要移动的数
                                    tmp.curMap[i][j] = num ;
                                    tmp.curMap[s][t] = 0 ;
                                }
                            }
                    }
                    
                    long long rhs = 1 ;
                    //得到地图对应的桶号
                    for (int a = 0 ; a < 4 ; a ++) {
                        for (int b = 1 ; b < 8 ; b ++) {
                            //虽然是mod是int类型的，但rhs还是要用long long，因为中途可能会超过int范围
                            rhs = (rhs * seed + tmp.curMap[a][b]) % mod ;  //每次都要mod一下
                        }
                    }
                    
                    if (!H[rhs])//当前地图没出现过，放入hash表
                        H[rhs]=1;
                    else  //已经出现过了，跳过
                        continue ;
                    
                    tmp.step ++;
                    q.push (tmp) ;
                }
            }
        }
    }
//    puts ("-1") ;//简易输出
    //没搜索到
    cout<<-1<<endl;
}

int main (){
    
    cin>>T;
    while (T --) {
        for (int i = 0 ; i < 4 ; i ++)
            for (int j = 1 ; j < 8 ; j ++)
                cin>>ans.curMap[i][j];
        
        for (int i = 0 ; i < 4 ; i ++)
            for (int j = 1 ; j < 8 ; j ++)
                if (ans.curMap[i][j] % 10 == 1)
                    ans.curMap[i][j] = 0 ;
        //系统操作，把11 21 31 41放到最左侧，从此状态开始搜索
        ans.curMap[0][0] = 11; ans.curMap[1][0] = 21; ans.curMap[2][0] = 31; ans.curMap[3][0] = 41;
        
//        for (int i = 0 ; i < 4 ; i ++) {
//            for (int j = 0 ; j < 8 ; j ++) {
//                cout<<ans.curMap[i][j]<<" ";
//            }
//            cout<<endl;
//        }
        bfs (ans) ;
    }
    return 0 ;
}

```



### Remainder

**HDU 1104**

http://acm.hdu.edu.cn/showproblem.php?pid=1104

（我发现我不是算法不会，我是数学不会。）

思路来自[这儿](https://blog.csdn.net/tim12300/article/details/9299431)

mod与%不一样，mod总会得到正数。而且%k %m取模顺序不同，结果也不同。

在num因为大量运算时会很大，于是我们要取余，但不能直接用%k，而是用%km（这里km就是k*m），因为不止要和k线性同余还要和m线性同余。也就是说，如果只用%k，那么我对n做了一步处理后，产生的结果只对k线性同余，假如下一步用%m，那么就错了。

**只有在放置visited时， if(visited[mod(result, k)]) ，以及判断最终答案时 if(mod(temp.curN,k) ==ans)， 才会对k进行取模。**

```
#include <iostream>
#include <cstdio>
#include <cstring>
#include <string>
#include <sstream>
#include <queue>
#include <map>

using namespace std;
int n,k,m;
int ans;
int km;
map<int, int>visited;
char op[4]={'+', '-', '*', '%'};


int mod(int x, int y){
    int r=-1;
    r = (x%y +y)%y;
    
    return r;
}


struct point{
    int curN; //当前N
    int layer; //运算次数
    string ops;  //运算符
    point(int c, int l, string o): curN(c), layer(l), ops(o){}
};

int cal(int a, int b, int index){//n,m,运算符下标
    int result = -1;
    if(index==0)
        result=a+b;
    if(index==1)
        result=a-b;
    if(index==2)
        result=a*b;
    if(index==3){
        result = mod(a,b);
    }
    result = mod(result, km);
    
    return result;
}

void bfs(){
    queue<point>q;
    while(!q.empty()) q.pop();
    visited.clear();
    
    q.push(point(n,0,""));
    visited[mod(n,k)] =1;
    
    int flag = 0;

    while(!q.empty()){
        point temp = q.front();
        q.pop();

        if(mod(temp.curN,k) ==ans){
            cout<<temp.layer<<endl;
            cout<<temp.ops<<endl;
            flag =1;
            return ;
        }
        
        for(int i=0; i<4; i++){
            string curOp = temp.ops;
            int result = cal(temp.curN, m, i);
            if(visited[mod(result, k)])
                continue;
            else{
                visited[mod(result, k)]=1;
                curOp.push_back(op[i]);
                q.push(point(result, temp.layer+1, curOp));
            }
        }
    }
    if(!flag)
    cout<<0<<endl;
    
}

int main(){
    while(cin>>n>>k>>m &&n &&k &&m){
        ans = mod((n+1),k);
        //不要想当然的认为 k,m都为偶数且n>0时会失败，这是不对的。
        km=k*m;
        bfs();
    }
}

/*
2 2 2
-1 12 10
0 0 0
 */

```



## 并查集

### Farm Irrigation

http://acm.hdu.edu.cn/showproblem.php?pid=1198

**HDU 1198**

思路很简单，就是并查集。但是有以下问题需要先解决。

- 如何表示一个点（或如何记录图的信息）

  题目给的一个点是一个字符，每个字符代表不同的开口的单元。于是我设置字符串s="xxxx"，s[i]表示在i的**相反**方向上有无开口（见下文），例如s="0011"表示右0、下0、左1、上1，对应题目中的单元A。然后我使用vector\<string>graph[MAXN]来保存整个图的信息。

- 父亲节点的寻找find()

  由于本题是图（二维数组），所以用一个数是表示不了二维坐标的。于是我使用了pair<int, int>类型。pair类型可以直接判断是否相等，且相等的pair一定唯一（不像结构体，即使属性都相等，两个结构体也不是同一个结构体）。

  pair的用法见这里：https://blog.csdn.net/sevenjoin/article/details/81937695

- 什么时候才能合并

  在k方向上需要满足以下条件：

  - k方向的邻居不越界
  - 本节点有开口
  - k方向上的邻居在k的相反方向上有开口

  由于我在字符串中，设置的是**相反**方向。即 k=0,1,2,3表示左、上、右、下。而s[k]分别表示右、下、左、上的开口。

  所以要满足上面三个条件，有：（设ex, ey为邻居节点的坐标）

  - **if**(ex<0 || ex>=m || ey<0 ||ey>=n) //越界
  - l = (k+2)%4。 此时s[l]才是真正的在k方向上的开口。
  - **if**(graph\[i]\[j][l] =='1' && graph\[ex]\[ey][k] =='1')

完整代码：

```
#include <iostream>
#include <cstdio>
#include <algorithm>
#include <map>
#include <vector>
#include <cstring>
#include <string>
using namespace std;
const int MAXN = 51;

pair<int, int> father[MAXN][MAXN];
vector<string> graph[MAXN];
map<char, string>f;
int m, n;

//偏移
int bias[4][2] = {{0,-1}, {-1,0}, {0,1}, {1,0}}; //左上右下

void init(){
    f['A'] = "0011";
    f['B'] = "1001";
    f['C'] = "0110";
    f['D'] = "1100";
    f['E'] = "0101";
    f['F'] = "1010";
    f['G'] = "1011";
    f['H'] = "0111";
    f['I'] = "1110";
    f['J'] = "1101";
    f['K'] = "1111";
}

pair<int,int> find(int x, int y){
    if(father[x][y]!= make_pair(x, y)){
        father[x][y] = find(father[x][y].first, father[x][y].second);
    }
    return father[x][y];
}

void Union(int i, int j, int x, int y){//将(i,j)与(x,y)连通
    pair<int, int>p1 = find(i,j);
    pair<int, int>p2 = find(x,y);
    if(p1!=p2){
        father[p2.first][p2.second] = p1;
    }
}

int main(){
    char ch;
    init();
    while(cin>>m>>n && m!=-1 &&n!=-1){
        memset(graph, 0, sizeof(graph));
        int count = 0;
        string row;
        for(int i=0; i<m; i++){
            cin>>row;
            for(int j=0; j<n; j++){
                ch = row[j];
                graph[i].push_back(f[ch]);
                father[i][j] = make_pair(i,j);  //父亲节点为自己
            }
        }
        
        for(int i=0; i<m; i++){
            for(int j=0; j<n; j++){
                for(int k=0; k<4; k++){//检查自己的四个方向 左上右下
                    int l = (k+2)%4; //自己的开口方向
                    //当前点在k方向上的邻居
                    int ex = i+bias[k][0];
                    int ey = j+bias[k][1];
                    if(ex<0 || ex>=m || ey<0 ||ey>=n) //越界
                        continue;
                    //自己在该方向上有开口并且k方向的邻居在对应的方向也有开口
                    //比如 k=0时，表示本节点左边有开口，并且左边的邻居右边有开口
                    if(graph[i][j][l] =='1' && graph[ex][ey][k] =='1'){
                        Union(i,j,ex,ey);
                    }
                }
            }
        }
        
        for(int i=0; i<m; i++){
            for(int j=0; j<n; j++){
                if(find(i,j)==make_pair(i,j))
                    count++;
            }
        }
        cout<<count<<endl;
    }
    return 0;
}

/*
2 2
DK
HF

3 3
ADC
FJK
IHE

-1 -1
 */

```



### 小希的迷宫

**HDU 1272**

http://acm.hdu.edu.cn/showproblem.php?pid=1272

与《刷题笔记》下半“**Is it A Tree?**” 相同。

判断是否为树：

- 根节点（入度为0的节点）有且仅有1个
- 终点（并查集中father[i]==i的节点）有且仅有1个
- 只要出现入度大于1，一定不是树
- 空树（无节点）有时也算树

```
#include <iostream>
#include <cstdio>
#include <cstring>
#include <string>
#include <algorithm>
#include <map>

using namespace std;
const int MAXN = 100001;
int father[MAXN];
int height[MAXN];
int inDegree[MAXN];
map<int, int>visited;

void init(){
    for(int i=0; i<MAXN; i++){
        father[i] = i;
        height[i] = 0;
        inDegree[i] = 0;
    }
    visited.clear();
}

int Find(int x){
    if(father[x]!=x){
        father[x] = Find(father[x]);
    }
    return father[x];
}

void Union(int x, int y){
    x = Find(x);
    y = Find(y);
    if(x!=y){
        if(height[x]>height[y]){
            father[y] = x;
        }
        else if(height[y]>height[x]){
            father[x] = y;
        }
        else{
            father[y] = x;
            height[x]++;
        }
    }
}

bool judge(){
    int flag =1;
    int root = 0;  //根节点——入度为0
    int terminal = 0;  //father为自己
    
    map<int, int>::iterator it;
    for(it=visited.begin(); it!=visited.end(); it++){
        int i = it->first; //节点序号
        if(inDegree[i]==0)
            root++;
        if(father[i]==i)
            terminal++;
        if(inDegree[i]>1)
            flag = 0;
    }
    
    if(terminal!=1 || root!=1)
        flag = 0;
    if(terminal==0 && root==0)
        flag = 1;
    return flag;
    
}

int main(){
    int p1, p2;
    init();
    while(cin>>p1>>p2){
        if(p1==-1 && p2==-1)
            break;
        if(p1==0 && p2==0){//输入结束，判断结果
            if(judge()){
                cout<<"Yes"<<endl;
            }
            else cout<<"No"<<endl;
            
            init();
            
        }
        else{//正常输入
            Union(p1, p2);
            inDegree[p2]++;
            visited[p1] = 1;
            visited[p2] = 1;
        }
    }
}

```



## 最小生成树

### Eddy's picture

**HDU 1162**

与《刷题笔记》“**Freckles**”相同。

http://acm.hdu.edu.cn/showproblem.php?pid=1162

kruskal 最小生成树。只不过相当于所有的边都要自己计算一下。别忘了初始化啊。

```
#include <iostream>
#include <cstdio>
#include <cstring>
#include <string>
#include <algorithm>
#include <map>
#include <queue>
#include <cmath>

using namespace std;
const int MAXN = 101;
int n;

struct edge{
    int start;
    int dest;
    double len;
    edge(int s, int d, double l):start(s), dest(d), len(l){}
};

struct point{
    double x;
    double y;
    point(double x, double y): x(x), y(y){}
};

vector<edge>e;
vector<point>p;
int father[MAXN];
int number;
double sum; //总代价

double getDis(int i, int j){
    double x1,y1,x2,y2;
    double dis = 0;
    x1 = p[i].x; y1 = p[i].y; x2 = p[j].x; y2 = p[j].y;
    dis = sqrt(abs(x1-x2)*abs(x1-x2) + abs(y1-y2)*abs(y1-y2));

    return dis;
}

bool cmp(edge a, edge b){
    return a.len<b.len;
}

int Find(int x){
    if(father[x]!=x){
        father[x] = Find(father[x]);
    }
    return father[x];
}

void Union (int x, int y, double len){
    x = Find(x);
    y = Find(y);
    if(x!=y){
        father[y] = x;
        sum+=len;
        number++;
    }
}

int main(){
    while(cin>>n){
        double xi,yi;
        number = 0; //边的条数清零
        sum = 0; //总长度清零
        p.clear();
        e.clear();
        for(int i=0; i<n; i++){
            cin>>xi>>yi;
            p.push_back(point(xi,yi));
            father[i] = i;
        }
        for(int i=0; i<p.size(); i++){
            for(int j=i+1; j<p.size(); j++){
                e.push_back(edge(i,j,getDis(i,j)));
            }
        }
        sort(e.begin(), e.end(), cmp);

        for(int i=0; i<e.size(); i++){
            if(number==n-1) //已经连通了
                break;
            Union(e[i].start, e[i].dest, e[i].len);
        }

        printf("%.2f\n", sum);

    }
    return 0;
}

```



## 最短路径

### A Walk Through the Forest (Dijkstra)

**HDU 1142**

http://acm.hdu.edu.cn/showproblem.php?pid=1142

Dijkstra+DFS

这题有坑。题目的意思 不是 让求从公司到家的最短路径有几条。

而是说从公司到家的路上每一个点A，它有某个邻接点B。如果家到A的最短距离大于，家到B的最短距离。那么它下一步就可以走B。 所以它的原理类似于动态规划，但是是用dfs实现。

整条路径的代价，是很可能比最短路径代价高的。

举个例子：

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1ggwgq5f17ej30h60fkjsb.jpg" alt="截屏2020-07-19 18.34.36" style="zoom:50%;" />

1是公司，2是家。家到各点的最短距离 dis[1] = 6, dis[2]=0, dis[3] = 5

此人在公司（1号），发现2到3的最短距离小于2到1的最短距离，所以认为经过3是一种进步。所以下一步可以走3。

即便最终行走的距离 = 2+5>7，但是没关系，他认为自己赚了，他开心就好。

所以每到达一个节点u，它的走法数就是它附近的所有认为是赚的节点 走法的和。

```
#include <iostream>
#include <cstdio>
#include <cstring>
#include <string>
#include <algorithm>
#include <map>
#include <queue>

using namespace std;
const int MAXN = 1001;
const int INF = 0x3f3f3f3f;
int n,m;
int lim;
int num;

struct edge{
    int dest;
    int len;
    edge(int d, int l):dest(d), len(l){}
};

struct point{
    int num;
    int dis;
    point(int n, int d): num(n), dis(d){}
    bool operator< (const point& p)const{
        return dis>p.dis;
    }
};

int dis[MAXN];
vector<edge>graph[MAXN];
int visited[MAXN];
int p[MAXN]; //从该点出发到家的进步路线数

void Dijkstra(int start){
    dis[start] = 0;
    priority_queue<point>pq;
    pq.push(point(start, 0));
    while(!pq.empty()){
        point temp  = pq.top();
        pq.pop();
        int u = temp.num;
        for(int i=0; i<graph[u].size(); i++){
            int v = graph[u][i].dest;
            if(dis[v]>dis[u]+graph[u][i].len){
                dis[v] = dis[u]+graph[u][i].len;
                pq.push(point(v,dis[v]));
            }
        }
    }
}

int dfs(int pos){
    int sum = 0;
    if(p[pos]!=0)
        return p[pos];
    if(pos==2){
        return 1;
    }
    else{
        for(int i=0; i<graph[pos].size(); i++){
            int v = graph[pos][i].dest;
            if(dis[pos]>dis[v]){//如果存在一条从B到他家的路线比从A到他的任何可能路线都短的话，那么从A到B的路线就是进步
                sum+=dfs(v);
            }
        }
        p[pos] = sum;
    }
    return p[pos];
}


int main(){
    while(cin>>n&& n){
        cin>>m;
        fill(dis, dis+MAXN, INF);
        memset(graph, 0, sizeof(graph));
        memset(visited, 0, sizeof(visited));
        memset(p, 0, sizeof(p));
        num = 0;
        
        int p1, p2, len;
        for(int i=0; i<m; i++){
            cin>>p1>>p2>>len;
            graph[p1].push_back(edge(p2, len));
            graph[p2].push_back(edge(p1, len));
        }

        Dijkstra(2);  //求源点到各节点的最短路径
        visited[1] = 1;
        cout<<dfs(1)<<endl;
    }
    return 0;
}

/*
5 6
1 3 2
1 4 2
3 4 3
1 5 12
4 2 34
5 2 24
7 8
1 3 1
1 4 1
3 7 1
7 4 1
7 5 1
6 7 1
5 2 1
6 2 1
0
 */

```





### <font color=red>Arbitrage</font> (SPFA)

**HDU 1217**

http://acm.hdu.edu.cn/showproblem.php?pid=1217

SPFA判环。 本来SPFA是用来判负环，但本题比较特殊，也可以用。

因为本题是乘积，且松弛操作是尽量取大。 所以如果从start能套利，则从start乘一连串之后一定可以到自己的汇率大于1，所以使用SPFA不断松弛，如果出现了dis[start]>1则立即break并返回true，如果队列为空说明当前该种货币不能套利。

需要检查所有的货币，所以要用n次SPFA。

```
#include <iostream>
#include <cstdio>
#include <cstring>
#include <string>
#include <algorithm>
#include <map>
#include <queue>
#include <cmath>

using namespace std;
const int MAXN = 31;
int n,m;

struct edge{
    int dest;
    double len;
    edge(int d, double l):dest(d), len(l){}
};

struct point{
    int num;
    double distance;
    point(double n, double d): num(n), distance(d){}
};

vector<edge>graph[MAXN];
map<string, int>str2int;
double dis[MAXN];
int visited[MAXN];

void init(){
    memset(dis, 0, sizeof(dis));
    memset(visited, 0, sizeof(visited));
}

//每次仅检查start到自己的汇率，松弛操作定为扩大rate。如果当前start可以套利，随着循环进行，乘积一定能大于1
bool SPFA(int start){
    dis[start] = 1;
    visited[start] = 1;
    queue<point>q;
    q.push(point(start, 1));
    while(!q.empty()){
        point temp = q.front();
        q.pop();
        int u = temp.num;
        visited[u] = 0;
        for(int i=0; i<graph[u].size(); i++){
            int v = graph[u][i].dest;
            double r = graph[u][i].len;
            if(dis[v]<dis[u] * r){
                dis[v] = dis[u]*r;
                if(dis[start]>1)  //如果原点的权值大于1，立即返回true
                    return true;
                if(visited[v]) continue; //如果v已经在队里了 直接continue
                else{
                    q.push(point(v,dis[v]));
                    visited[v] = 1;
                }
            }
        }
    }
    return false; //没有无限递增环
}

bool judge(){
    int flag = 0;
    for(int i=0; i<n; i++){
        init();
        flag = SPFA(i);
        if(flag) break;
    }
    return flag;
}

int main(){
    int t = 0;
    while(cin>>n &&n){
        t++;
        memset(graph, 0, sizeof(graph));
        str2int.clear();
        init();

        string str;
        for(int i=0; i<n; i++){
            cin>>str;
            str2int[str] = i;
        }
        cin>>m;
        string s1, s2;
        double rate;
        for(int i=0; i<m; i++){
            cin>>s1>>rate>>s2;
            graph[str2int[s1]].push_back(edge(str2int[s2], rate));
        }
        cout<<"Case "<<t<<": ";
        if(judge()){
            cout<<"Yes"<<endl;
        }
        else cout<<"No"<<endl;
    }
    return 0;
}


/*
3
USDollar
BritishPound
FrenchFranc
3
USDollar 0.5 BritishPound
BritishPound 10.0 FrenchFranc
FrenchFranc 0.21 USDollar

3
USDollar
BritishPound
FrenchFranc
6
USDollar 0.5 BritishPound
USDollar 4.9 FrenchFranc
BritishPound 10.0 FrenchFranc
BritishPound 1.99 USDollar
FrenchFranc 0.09 BritishPound
FrenchFranc 0.19 USDollar

0
 */

```





### <font color=red>AOE网上的关键路径</font>

SDUT OJ 2498

https://acm.sdut.edu.cn/onlinejudge3/problems/2498

<font color=red>**最长路径（关键路径）+字典序**</font>

反向建图+反反向输出

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1ggzkozr05vj30ki09iwj8.jpg" alt="截屏2020-07-22 11.08.34" style="zoom:50%;" />



#### 方法1: SPFA

我在本题创建的是sub数组，之所以没有用pre，比如u=5，v=2和3时。如果2先出来3再出来，那么pre[5]=3。然后pre[2]=1, pre[3]=0导致路走死了。

**所以在构建数组时，一般应该与当前探索方向相反。**

```
if(dis[v]< dis[u]+l){  //最长路径松弛
	dis[v] = dis[u]+l;
	pq.push(v);
	sub[v] = u;  //由于是反向建图，而输出的时候是正向，所以这里使用后继 v->sub[v]
}
else if(dis[v]==dis[u]+l){ //最长路径相同时，选择节点小的
	if(u<sub[v]){
	sub[v] = u;
}
```



```
#include <iostream>
#include <cstdio>
#include <cstring>
#include <vector>
#include <queue>
#include <string>
#include <algorithm>
#include <map>
#include <cmath>

using namespace std;

const int MAXN = 10001;

struct edge{
    int dest;
    int len;
    edge(){}
    edge(int d, int l):dest(d), len(l){}
};

int dis[MAXN];
int sub[MAXN];
int inDegree[MAXN];
int outDegree[MAXN];
vector<edge>graph[MAXN];
int s, t;

void SPFA(int start){
    priority_queue<int, vector<int>, greater<int>>pq;  //从终点向前看，节点小的优先放进来
    dis[start] = 0;
    pq.push(start);
    while(!pq.empty()){
        int u = pq.top();
        pq.pop();
        for(int i=0; i<graph[u].size(); i++){
            int v= graph[u][i].dest;
            int l = graph[u][i].len;
            if(dis[v]< dis[u]+l){  //最长路径松弛
                dis[v] = dis[u]+l;
                pq.push(v);
                sub[v] = u;  //由于是反向建图，而输出的时候是正向，所以这里使用后继 v->sub[v]
            }
            else if(dis[v]==dis[u]+l){ //最长路径相同时，选择节点小的
                if(u<sub[v]){
                    sub[v] = u;
                }
            }
        }
    }
}

int main(){
    int n,m;
    while(cin>>n>>m && n && m){
        memset(graph, 0, sizeof(graph));
        memset(dis, 0, sizeof(dis));
        memset(sub, 0, sizeof(sub));
        memset(inDegree, 0, sizeof(inDegree));
        memset(outDegree, 0, sizeof(outDegree));

        int v1, v2, l;
        for(int i=0; i<m; i++){
            cin>>v1>>v2>>l;
            graph[v2].push_back(edge(v1,l));  //反向建图
            outDegree[v2]++;
            inDegree[v1]++;
        }
        for(int i=1; i<=n; i++){
            if(!inDegree[i]) s=i;  //建图起点（原终点）
            if(!outDegree[i]) t=i;  //建图终点（原起点）
        }
        SPFA(s);
        int maxx = 0;
        int index = 0;
        for(int i=1; i<=n; i++){
            if(maxx<dis[i]){
                maxx = dis[i];
                index = i;
            }
        }
        cout<<maxx<<endl;
        
        int k = index;
        while(sub[k]){
            cout<<k<<" "<<sub[k]<<endl;
            k = sub[k];
        }
    }
    return 0;
}

/*
9 11
1 2 6
1 3 4
1 4 5
2 5 1
3 5 1
4 6 2
5 7 9
5 8 7
6 8 4
8 9 4
7 9 2

 */

```



#### 方法2:关键路径

其实只是在求最长路的方式和方法1不同罢了。求路径的时候还是用的优先队列最小堆+后继数组

```
if(ve[v]< ve[u]+l){
    ve[v] = ve[u]+l;
    sub[v] = u;
}
else if(ve[v]==ve[u]+l)
    if(u<sub[v])
        sub[v] = u;
    
inDegree[v]--;
if(!inDegree[v]){
    pq.push(v);
}
```

完整代码：

```
#include <iostream>
#include <cstdio>
#include <cstring>
#include <vector>
#include <queue>
#include <string>
#include <algorithm>
#include <map>
#include <cmath>

using namespace std;

const int MAXN = 10001;

struct edge{
    int dest;
    int len;
    edge(){}
    edge(int d, int l):dest(d), len(l){}
};

int ve[MAXN];
int sub[MAXN];
int inDegree[MAXN];
int outDegree[MAXN];
vector<edge>graph[MAXN];
int n,m;
int s, t;

void topoSort(){
    priority_queue<int, vector<int>, greater<int>>pq;  //从终点向前看，节点小的优先放进来
    for(int i=1; i<=n; i++){
        if(!inDegree[i]){
            pq.push(i);
        }
    }
    while(!pq.empty()){
        int u = pq.top();
        pq.pop();
        for(int i=0; i<graph[u].size(); i++){
            int v= graph[u][i].dest;
            int l = graph[u][i].len;
            
            if(ve[v]< ve[u]+l){
                ve[v] = ve[u]+l;
                sub[v] = u;
            }
            else if(ve[v]==ve[u]+l)
                if(u<sub[v])
                    sub[v] = u;
                
            inDegree[v]--;
            if(!inDegree[v]){
                pq.push(v);
            }
        }
    }
}

int main(){
    
    while(cin>>n>>m && n && m){
        memset(graph, 0, sizeof(graph));
        memset(sub, 0, sizeof(sub));
        memset(ve, 0, sizeof(ve));
        memset(inDegree, 0, sizeof(inDegree));
        memset(outDegree, 0, sizeof(outDegree));

        int v1, v2, l;
        for(int i=0; i<m; i++){
            cin>>v1>>v2>>l;
            graph[v2].push_back(edge(v1,l));  //反向建图
            outDegree[v2]++;
            inDegree[v1]++;
        }
        for(int i=1; i<=n; i++){
            if(!inDegree[i]) s=i;  //建图起点（原终点）
            if(!outDegree[i]) t=i;  //建图终点（原起点）
        }
        topoSort();
        cout<<ve[t]<<endl;
        int k = t;
        while(sub[k]){
            cout<<k<<" "<<sub[k]<<endl;
            k = sub[k];
        }
    }
    return 0;
}

/*
9 11
1 2 6
1 3 4
1 4 5
2 5 1
3 5 1
4 6 2
5 7 9
5 8 7
6 8 4
8 9 4
7 9 2
 */

```





## 拓扑排序

无论何时，当需要判断某个图是否是 **有向无环图** 时，都应立刻联想到求拓扑排序。



### <font color = DodgerBlue>Legal or Not</font>

**HDU 3342**

http://acm.hdu.edu.cn/showproblem.php?pid=3342

拓扑排序模版题。

```
#include <iostream>
#include <cstdio>
#include <cstring>
#include <string>
#include <algorithm>
#include <map>
#include <queue>
#include <cmath>

using namespace std;
const int MAXN = 101;
int n,m;

vector<int>graph[MAXN];
int inDegree[MAXN];

bool Topo(){
    queue<int>q;
    for(int i=0; i<n; i++){
        if(inDegree[i]==0){
            q.push(i);
        }
    }
    int number=0; //当number==n时表示无环
    while(!q.empty()){
        int u = q.front();
        q.pop();
        number++;  //+1
        for(int i=0; i<graph[u].size(); i++){ //对所有的指向节点入度减一
            int v = graph[u][i];
            inDegree[v]--;
            if(!inDegree[v]){
                q.push(v);
            }
        }
    }
    if(number==n)
        return true;
    else return false;
}

int main(){
    while(cin>>n && n){
        memset(graph, 0, sizeof(graph));
        memset(inDegree, 0, sizeof(inDegree));
        cin>>m;
        int v1, v2;
        for(int i=0; i<m; i++){
            cin>>v1>>v2;
            graph[v1].push_back(v2);
            inDegree[v2]++;
        }
        if(Topo()){
            cout<<"YES"<<endl;
        }
        else cout<<"NO"<<endl;
    }
    return 0;
}

```



### <font color=red>Rank of Tetris</font>

**HDU 1811**

http://acm.hdu.edu.cn/showproblem.php?pid=1811

拓扑排序+并查集。主要是题目中存在“=”关系。对于相等的元素，需要使用并查集，用父亲节点代替整个集合的点。然后检查"<"">"关系中是否有同一并查集的元素，如果有则立刻冲突。

在入队时是入的父亲节点，都需要Find。而且既然相等的都用父亲节点代替了，所以每次队列中最大只能存在1个节点。因为若存在两个且又不在同一并查集，则它们是UNCERTAIN的。

这里需要用数组a[i], ch[i], b[i]保存每个输入，因为需要遍历两次。第一次是并查集，第二次是检查"<"">"关系。

```
#include<iostream>
#include<cstdio>
#include<cstring>
#include<algorithm>
#include<vector>
#include<queue>
using namespace std;
const int N=10005;
vector<int>G[N*2];
int inDegree[N],father[N];
 
int n,num;
int cnt,flag;
int a[N],b[N];
char ch[N];

int Find(int x){
    if(father[x]!=x){
        father[x] = Find(father[x]);
    }
    return father[x];
}

void Union(int x, int y){
    x=Find(x);
    y=Find(y);
    if(x!=y)
        father[x]=y;
}

void topo(){
    queue<int> Q;
    flag=0;
    num=0;
    for(int i=0;i<n;i++){
        int fx=Find(i);
        if(fx==i){//必须是并查集的父亲节点
            num++;
            if(inDegree[fx]==0)
                Q.push(fx);
             
        }
    }
    while(!Q.empty()){
        if(Q.size()>1) //如果队列里有1个以上，则出现了无法比较
            flag=1;
        int y=Q.front();
        Q.pop();
        cnt++;
        for(int i=0;i<G[y].size();i++){
             int fy=Find(G[y][i]);
             inDegree[fy]--;
             if(inDegree[fy]==0)
                Q.push(fy);
        }
    }
}


int main(){
    int m;
    while(scanf("%d%d",&n,&m)!=-1){
        memset(inDegree,0,sizeof(inDegree));
        for(int i=0;i<n;i++){
            G[i].clear();
            father[i]=i;
        }
        cnt=0;
        //并查集将相等的合并在一起，用最终父亲节点代表这一个集合
        for(int i=0;i<m;i++){
            cin>>a[i]>>ch[i]>>b[i];
            if(ch[i]=='=')
                Union(a[i], b[i]);
        }
        int vis=0;
        for(int i=0;i<m;i++){
            if(ch[i]!='='){
                int x=Find(a[i]),y=Find(b[i]);
                if(x==y){ //如果在同一相等集里的两个元素存在大小关系，则conflict
                    vis=1;
                    break;
                }
                if(ch[i]=='<')
                    swap(a[i],b[i]);
                
                x=Find(a[i]);
                y=Find(b[i]);
                G[x].push_back(y);
                inDegree[y]++;
            }
            
        }
        if(vis==1){
            printf("CONFLICT\n");
            continue;
        }
        topo();

        if(cnt!=num)
            printf("CONFLICT\n");
        else{
            if(flag==1)
                printf("UNCERTAIN\n");
            else
                printf("OK\n");
        }
    }
    return 0;
}

```



### Reward

**HDU 2647**

http://acm.hdu.edu.cn/showproblem.php?pid=2647

一定要分清谁指向谁。本题a b表示a要比b多，而且我们最后是统计总量。所以应该把b放在前面，从小到大拓扑排序。

```
#include<iostream>
#include<cstdio>
#include<cstring>
#include<algorithm>
#include<vector>
#include<queue>
#include <map>
using namespace std;
const int MAXN = 10001;

vector<int>graph[MAXN];
int inDegree[MAXN];
int n,m;
int number;
map<int, int>layer;

bool topo(){
    queue<int>q;
    number = 0;
    for(int i=1; i<=n; i++){
        if(!inDegree[i]){
            q.push(i);
            layer[i] = 0; //首批入队的layer都为0
        }
    }
    while(!q.empty()){
        int u = q.front();
        q.pop();
        number++;
        for(int i=0; i<graph[u].size(); i++){
            int v = graph[u][i];
            inDegree[v]--;
            if(!inDegree[v]){
                q.push(v);
                layer[v] = layer[u]+1;
            }
        }
    }
    if(n==number) return true;
    else return false;
}

int main(){
    while(cin>>n>>m){
        memset(graph, 0, sizeof(graph));
        memset(inDegree, 0, sizeof(inDegree));
        layer.clear();
        int v1, v2;
        for(int i=0; i<m; i++){
            cin>>v1>>v2;
            graph[v2].push_back(v1);  //小的指向大的
            inDegree[v1]++;
        }
        if(topo()){
            map<int, int>::iterator it;
            int extra = 0;
            for(it = layer.begin(); it!=layer.end(); it++){
                extra+= it->second;
            }
            cout<<888*n+extra<<endl;
        }
        else cout<<-1<<endl;
    }
    return 0;
}

/*
2 1
1 2
2 2
1 2
2 1
 */

```



### <font color=red>逃生</font>

**HDU 4857**

http://acm.hdu.edu.cn/showproblem.php?pid=4857

参考博客：[点击跳转](https://blog.csdn.net/a1097304791/article/details/89432821?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)

**反向拓扑建图+优先队列。**

题目要求的是让序号小的尽量在前面。必须反向建图并且大顶堆。比如

1->4->2 

5->3->2

如果按照正向建图小顶堆，得到序号 1 4 5 3 2。 而理想结果是 1 5 3 4 2。



**另外，本题使用cin输入会超时！！！！！！！！ 要使用scanf**

```
#include <iostream>
#include <cstdio>
#include <cstring>
#include <vector>
#include <queue>
#include <string>
#include <algorithm>
#include <map>

using namespace std;
const int MAXN = 30010;
int n,m;
int number;
vector<int>graph[MAXN];
int inDegree[MAXN];
vector<int>path;

void topoSort(){
path.clear();
    priority_queue<int> pq;
    for(int i=1; i<=n; i++){
        if(inDegree[i]==0)
            pq.push(i);
    }
    number = 0;
    while(!pq.empty()){
        int u  = pq.top();
        pq.pop();
        number++;
        path.push_back(u);
        for(int i=0; i<graph[u].size(); i++){
            int v = graph[u][i];
            inDegree[v]--;
            if(inDegree[v]==0) pq.push(v);
            
        }
    }
    
    return ;
}

int main(){
    int t;
    int v1, v2;
    scanf("%d", &t);
    while(t--){
        memset(graph,0,sizeof(graph));
        memset(inDegree, 0, sizeof(inDegree));
        scanf("%d%d", &n, &m); 
           
        for(int i=1; i<=m; i++){
            scanf("%d%d", &v1, &v2);     
            graph[v2].push_back(v1); //反向建图
            inDegree[v1]++;
        }
        topoSort();
        //打印路径
        for(int i=path.size()-1; i>=0; i--){
            if(i==0)
                cout<<path[i];
            else cout<<path[i]<<" ";
        }
        cout<<endl;
    }
    return 0;
}

/*
20
4 3
4 1
4 2
3 2
 
5 10
3 5
1 4
2 5
1 2
3 4
1 4
2 3
1 5
3 5
1 2
 */

```



## 关键路径

### <font color=dodgerblue>Instruction Arrangement</font>

**HDU 4109**

http://acm.hdu.edu.cn/showproblem.php?pid=4109

关键路径模版题。虽然这一题可以不用求vl。

```
#include <iostream>
#include <cstdio>
#include <cstring>
#include <vector>
#include <queue>
#include <string>
#include <algorithm>
#include <map>
#include <cmath>

using namespace std;

const int MAXN = 1005;
const int inf = 0x3fffffff;

struct edge{
    int dest;
    int len;
    edge(){}
    edge(int d, int l): dest(d), len(l){}
};

int ve[MAXN];
int vl[MAXN];
vector<edge>graph[MAXN];
vector<int>topo;
int inDegree[MAXN];
int n,m;



void critical(){
    queue<int>q;
    for(int i=0; i<n; i++){
        if(!inDegree[i]){
            q.push(i);
            //这里因为单个指令还有执行时间1ns，由于指令距离都不小于1ns，所以没啥影响。（如果执行时间很长，则路径长度就相当于max(执行时间, 指令距离)
            //但我们希望事件是瞬时的（即刚发生就能去看下一个事件），所以用执行完毕事件代表发生时间
            ve[i]=1;
        }
    }
    while(!q.empty()){
        int u = q.front();
        q.pop();
        topo.push_back(u);
        for(int i=0; i<graph[u].size(); i++){
            int v = graph[u][i].dest;
            int l = graph[u][i].len;
            inDegree[v]--;
            ve[v] = max(ve[v], ve[u]+l);  //事件最早开始时间为它前面的所有事件+活动时间的最大值
            if(!inDegree[v]){
                q.push(v);
            }
        }
    }
    //下面这些代码本题可以省略
    for(int i=topo.size()-1; i>=0; i--){
        int u = topo[i];
        if(graph[u].size()==0)
            vl[u] = ve[u];
        for(int j=0; j<graph[u].size(); j++){
            int v = graph[u][j].dest;
            int l = graph[u][j].len;
            vl[u] = min(vl[u], vl[v]-l);
        }
    }
}

int main(){
    while(cin>>n>>m && n && m){
        memset(graph, 0, sizeof(graph));
        memset(ve, 0, sizeof(ve));
        memset(inDegree, 0, sizeof(inDegree));
        fill(vl, vl+n, inf);
        topo.clear();

        int v1, v2, l;
        for(int i=0; i<m; i++){
            cin>>v1>>v2>>l;
            graph[v1].push_back(edge(v2, l));
            inDegree[v2]++;
        }
        critical();  //关键路径（其实只用拓扑排序就行了，不用求vl
        
        int ans = -1;
        for(int i=0; i<n; i++){
            if(ans<ve[i])
                ans = ve[i];
        }
        cout<<ans<<endl;
    }
    return 0;
}

/*
5 2
1 2 1
3 4 1
 */

```



## 贪心

### Tian Ji -- The Horse Racing

**HDU 1052**

http://acm.hdu.edu.cn/showproblem.php?pid=1052

表面上简单的贪心，但并不是我想象的那样。

数据可能会有以下特殊情况：🐎的速度可能会相同；tj的🐎可能比qw的跑的还快。

参考思路：[点击跳转](https://blog.csdn.net/dgq8211/article/details/7370765)，但是关于最快的马比较时逻辑不太完善，我在评论区进行了补充说明。

算法思路：

```
if (tj最差的🐎比qw最差的🐎好)
	直接赢一局
else if (tj最差的🐎比qw最差的🐎还差)
	肯定输了，所以用最差的马去消耗qw最好的马
else //tj当前最差的马和qw的一样
	if(tj最快的🐎已经比qw最快的🐎快)
		不需要最差的马牺牲
	else if(tj最快的🐎已经比qw最快的🐎慢)
		tj最差的马牺牲
	else //最好的马相同，此时只有两种情况，tj最差的马比qw最好的马差，或者相同
		if(tj最差的🐎与qw最好的🐎相同)
			不需要最差的马牺牲
		else
			tj最差的马牺牲
```



```
#include <iostream>
#include <cstdio>
#include <queue>
#include <vector>
#include <algorithm>

using namespace std;
const int MAXN = 1001;

int tj[MAXN];
int qw[MAXN];

int n;

int main(){
    while(cin>>n && n){

        for(int i=0; i<n; i++){
            cin>>tj[i];
        }
        for(int i=0; i<n; i++){
            cin>>qw[i];
        }
        
        sort(tj, tj+n);
        sort(qw, qw+n);


        int win = 0;  //胜利
        int lose = 0;
        int count = 0;
        int draw = 0;
        int startT, endT, startQ, endQ;
        startT = startQ =0; endT=endQ=n-1;
        
        for(int i=startT; i<=endT;){
            for(int j=startQ; j<=endQ;){  //j每次都要从startQ启动！！！ 而i不是
                if(tj[i]>qw[j]){//tj最差的马比qw最差的马好，直接赢一局
                    i++; startQ++; win++;
                    break;
                }
                else if(tj[i]<qw[j]){//tj最差的马比qw最差的马还差，肯定输了，所以用最差的马去消耗qw最好的马
                    i++; endQ--; lose++;
                    break;
                }
                else{//tj当前最差的马和qw的一样
                    if(tj[endT]>qw[endQ]){//tj最快的马已经比qw最快的马快，不需要最差的马牺牲
                        win++; endT--; endQ--;
                        break;
                    }
                    else if(tj[endT]<qw[endQ]){  //tj最快的马已经比qw最快的马慢，tj最差的马牺牲
                        lose++; i++; endQ--;
                        break;
                    }
                    else{//最好的马相同，此时只有两种情况，tj最差的马比qw最好的马差，或者相同
                        if(tj[i]==qw[endQ]){
                            i++; startQ++; draw++;
                            break;
                        }
                        else{
                            i++; endQ--; lose++;
                            break;
                        }
                    }
                }
                
            }
            
        }

        count = (win-lose)*200;
        cout<<count<<endl;
    }
    return 0;
}

/*
3
92 83 71
95 87 74
2
20 20
20 20
2
20 19
22 18
0
 */

```



### <font color=purple>今年暑假不AC（区间贪心）</font>

**HDU 2037**

http://acm.hdu.edu.cn/showproblem.php?pid=2037

我使用了dfs去做。来自[夏令营机试D题](https://s10.aconvert.com/convert/p3r68-cdx67/azpon-z2i9b.html)的灵感。 可以处理其他因素影响的情况，只要把current改成目标因素（如学分）就行。

当时D题给了学分，学分多的方案好，而不是上的课多的方案好。

比较两个区间是否有重叠，应该是l1<r2 && l2 < r1。

**这里引入了pair**

```
#include <iostream>
#include <cstdio>
#include <algorithm>
#include <cstring>
#include <string>
#include <map>
#include <vector>

using namespace std;

struct point{
    int start;
    int end;
    point(){}
    point(int s, int e):start(s), end(e){}
};

int maxn = 0;
int n;


vector<pair<int, int>>lim;
vector<point>cla;

bool judge(int start, int end){
    int flag =1;
    for(int i=0; i<lim.size(); i++){
        if((start<lim[i].second && end>lim[i].first))//区间重合
            flag = 0;
    }
    return flag;
}

void dfs(int pos, int current){
    if(pos==n){
        if(current>maxn){
            maxn = current;
            return;
        }
    }
    else{
        if(pos-current>=n-maxn)  //pos-current为在此之前没拿的课，n-maxn为最多允许不拿的
            return;
        for (int i = 0; i <= 1; i++) { //拿不拿当前
            if (!i) { //不拿
                dfs(pos + 1, current);
            } else {//拿
                int flag = judge(cla[pos].start, cla[pos].end);
                if (flag) {//可以拿
                    lim.push_back(make_pair(cla[pos].start, cla[pos].end));
                    dfs(pos + 1, current + 1);
                    lim.pop_back();
                }
            }
        }
    }
}


int main(){
    while(cin>>n && n){
        cla.clear();
        lim.clear();
        maxn = 0;
        int start, end;
        for(int i=0; i<n; i++){
            cin>>start>>end;
            cla.push_back(point(start, end));
        }
        dfs(0,0);
        cout<<maxn<<endl;
    }
    return 0;
}

/*
12
1 3
3 4
0 7
3 8
15 19
15 20
10 15
8 18
6 12
5 10
4 14
2 9
0
 */

```



## 动态规划

### Greatest Common Increasing Subsequence

**HDU 1423**

http://acm.hdu.edu.cn/showproblem.php?pid=1423

<font color=red>**最长公共递增子序列(LCIS)**</font>

1. **确定状态**
     **可以定义 dp\[i][j] 表示以 a 数组的前 i 个整数与 b 数组的前 j 个整数且<font color=red>以 b[j] 为结尾</font>构成的公共子序列的长度。**
        对于解决DP问题,第一步定义状态是很重要的!
        需要注意，<font color=red>**以 b[j] 结尾构成的公共子序列的长度不一定是最长公共子序列的长度！**</font>
2. **确定状态转移方程**
   - 当 a[i] == b[j] 时,我们只需要在前面找到一个能将 b[j] 接到后面的最长的公共子序列.
     - 之前最长的公共子序列在哪呢？首先我们要去找的 dp[][] 的第一维必然是 i - 1 ,因为 i 已经拿去和 b[j] 配对去了，不能用了。并且也不能是 i - 2 ，因为 i - 1 必然比 i - 2 更优。
     - 第二维呢？那就需要枚举 b[1]...b[j-1] 了，因为你不知道这里面哪个最长且哪个小于 b[j] 。
     - 这里还有一个问题，可不可能不配对呢？也就是在 a[i] == b[j] 的情况下，需不需要考虑 dp\[i][j] == dp\[i-1][j] 的决策呢？答案是不需要。因为如果 b[j] 不和 a[i] 配对，那就是和之前的 a[1]...a[j-1] 配对（假设 dp\[i-1][j]>0 ，等于0不考虑），这样必然没有和 a[i] 配对优越。（为什么必然呢？因为 b[j] 和 a[i] 配对之后的转移是 max(dp\[i-1][k])+1 ，而和之前的 i1 配对则是 max(dp\[i1-1][k])+1 。
   - **当 a[i] != b[j] 时**, 因为 dp[i][j] 是以 b[j] 为结尾的LCIS，如果 dp[i][j] > 0 那么就说明 a[1]..a[i] 中必然有一个整数 a[k] 等于 b[j] ,因为 a[k] != a[i] ，那么 a[i] 对 dp[i][j] 没有贡献，于是我们不考虑它照样能得出 dp[i][j] 的最优值。所以在 a[i] != b[j] 的情况下必然有 dp\[i][j] == dp\[i-1][j] 。

- **综上所述,我们可以得到状态转移方程:**
  `① a[i] != b[j], dp[i][j] = dp[i-1][j]`
  `② a[i] == b[j], dp[i][j] = max(dp[i-1][k]+1) (1 <= k <= j-1 && b[j] > b[k])`

```
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <cmath>
#include <vector>
using namespace std;
const int MAXN = 501;

int A[MAXN];
int B[MAXN];
int dp[MAXN][MAXN];  //dp[i][j]是A串前i个，B串前j个，且已B[j]为结尾的LCIS的长度 （这里暂时以1为下标起点）
//注意是LCIS 最长递增公共子串


int main(){
    int n1, n2;
    int n;
    cin>>n;
    while(n--){
        memset(dp, 0, sizeof(dp));
        cin>>n1;
        for(int i=1; i<=n1; i++)
            cin>>A[i];
        
        cin>>n2;
        for(int i=1; i<=n2; i++)
            cin>>B[i];
        
        for(int i=1; i<=n1; i++){
            for(int j=1; j<=n2; j++){
                dp[i][j] = dp[i-1][j];  //缺省值
                if(A[i]==B[j]){
                    for(int k=0; k<j; k++){
                        if(B[k]<B[j])
                            dp[i][j] = max(dp[i][j], dp[i-1][k]+1);  //注意后面的是i-1
                    }
                }
            }
        }

        int ans = 0;
        for(int j=1; j<=n2; j++){
            if(dp[n1][j]>ans){
                ans = dp[n1][j];
            }
        }
        cout<<ans<<endl;
        if(n)
            cout<<endl;
    }
    return 0;
    
}

```



还有优化的办法，但是我吸收不了了。





### 最少拦截系统

http://acm.hdu.edu.cn/showproblem.php?pid=1257

规律：<font color=red>**下降子序列的个数等于最长上升子序列的长度。**</font>

 在一个序列里，不上升子序列的个数等于最长上升子序列的长度。反过来也一样。

参考博客：[博客1](https://blog.csdn.net/m0_37971327/article/details/77822851)、[博客2](https://blog.csdn.net/sdnulixianrui/article/details/80036203)

我原本的思路是，一遍一遍遍历，每次选出最长的下降子序列，然后设置pre[]数组记录dp继承的前一个的坐标，然后利用pre数组倒着索引将本次最长子序列的元素全部设为visited[i] = true。 然后再从数组中未被访问的元素中选出最长下降子序列，循环往复。直到被清除的个数达到元素总个数。 每次大循环就增加一次炮弹数量。

然后果然超时了。



下面是正确解法：

```
#include <iostream>
#include <cstdio>
#include <vector>
#include <cstring>
#include <cmath>
#include <algorithm>
using namespace std;

const int MAXN = 0x3fffff;
int height[MAXN];

int dp[MAXN];
int main(){
    int n;
    while(cin>>n && n ){
        for(int i=0; i<n; i++){
            cin>>height[i];
        }
        fill(dp, dp+n, 1);
        for(int i=0;i<n; i++){
            for(int j=0; j<i; j++){
                if(height[j]<=height[i]){
                    dp[i]= max(dp[i], dp[j]+1);
                }
            }
        }
        int maximum = -1;
        for(int i=0; i<n; i++){
            if(maximum<dp[i])
                maximum = dp[i];
        }
        cout<<maximum<<endl;
    }
    return 0;
}
//8 389 207 155 300 299 170 158 65

```

