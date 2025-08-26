# C++ STL容器大全及常用方法

## 🗂️ STL容器分类

### 1. 序列容器（Sequence Containers）

### 2. 关联容器（Associative Containers）

### 3. 无序关联容器（Unordered Associational Containers）

### 4. 容器适配器（Container Adapters）

---

## 1. 序列容器

### 🎯 std::vector（动态数组）

​**​特点​**​：动态扩容的连续内存数组

```
#include <vector>
using namespace std;

vector<int> vec; // 创建空vector

// 常用方法：
vec.push_back(10);          // 尾部添加元素
vec.pop_back();             // 删除尾部元素
vec.insert(vec.begin(), 5); // 指定位置插入
vec.erase(vec.begin());     // 删除指定位置元素
vec.clear();                // 清空所有元素
vec.size();                 // 返回元素个数
vec.empty();                // 判断是否为空
vec[0];                     // 随机访问元素
vec.at(0);                  // 带边界检查的访问
vec.front();                // 第一个元素
vec.back();                 // 最后一个元素
vec.resize(100);            // 调整大小
vec.reserve(1000);          // 预分配内存
```

### 🎯 std::deque（双端队列）

​**​特点​**​：头尾高效插入删除的双端队列

```
#include <deque>
using namespace std;

deque<int> dq;

// 常用方法：
dq.push_back(10);           // 尾部添加
dq.push_front(5);           // 头部添加（vector没有！）
dq.pop_back();              // 尾部删除  
dq.pop_front();             // 头部删除（vector没有！）
dq.front();                 // 访问头部
dq.back();                  // 访问尾部
dq.size();                  // 元素个数
dq.empty();                 // 是否为空
```

### 🎯 std::list（双向链表）

​**​特点​**​：任意位置高效插入删除的双向链表

```
#include <list>
using namespace std;

list<int> lst;

// 常用方法：
lst.push_back(10);          // 尾部添加
lst.push_front(5);          // 头部添加
lst.pop_back();             // 尾部删除
lst.pop_front();            // 头部删除
lst.insert(++lst.begin(), 7); // 指定位置插入
lst.erase(lst.begin());     // 删除指定位置
lst.remove(5);              // 删除所有值为5的元素
lst.unique();               // 删除连续重复元素
lst.sort();                 // 排序（成员函数！）
lst.reverse();              // 反转（成员函数！）
lst.size();                 // 元素个数
lst.empty();                // 是否为空
lst.front();                // 头部元素
lst.back();                 // 尾部元素
lst.splice(lst.begin(), other_list); // 转移元素
```

### 🎯 std::forward_list（单向链表）

​**​特点​**​：更节省内存的单向链表

```
#include <forward_list>
using namespace std;

forward_list<int> flst;

// 常用方法：
flst.push_front(10);        // 头部添加（没有push_back！）
flst.pop_front();           // 头部删除
flst.insert_after(flst.before_begin(), 5); // 在指定位置后插入
flst.erase_after(flst.before_begin()); // 删除指定位置后的元素
flst.front();               // 访问头部元素
flst.empty();               // 是否为空
```

### 🎯 std::array（固定大小数组）

​**​特点​**​：STL风格的固定大小数组

```
#include <array>
using namespace std;

array<int, 5> arr = {1, 2, 3, 4, 5};

// 常用方法：
arr.fill(0);                // 填充相同值
arr.size();                 // 元素个数（固定）
arr.empty();                // 是否为空（总是false）
arr[0];                     // 随机访问
arr.at(0);                  // 带边界检查的访问
arr.front();                // 第一个元素
arr.back();                 // 最后一个元素
```

---

## 2. 关联容器

### 🎯 std::set（集合）

​**​特点​**​：有序唯一元素集合

```
#include <set>
using namespace std;

set<int> s;

// 常用方法：
s.insert(10);               // 插入元素
s.erase(10);                // 删除元素
s.find(10);                 // 查找元素，返回迭代器
s.count(10);                // 统计元素出现次数（0或1）
s.lower_bound(5);           // 返回第一个>=5的迭代器
s.upper_bound(5);           // 返回第一个>5的迭代器
s.size();                   // 元素个数
s.empty();                  // 是否为空
s.clear();                  // 清空集合
```

### 🎯 std::multiset（多重集合）

​**​特点​**​：有序可重复元素集合

```
#include <set>
using namespace std;

multiset<int> ms;

// 常用方法：（与set类似，但允许重复）
ms.insert(10);              // 可重复插入
ms.insert(10);              // 现在有两个10
ms.count(10);               // 返回2
ms.erase(10);               // 删除所有值为10的元素
```

### 🎯 std::map（键值对映射）

​**​特点​**​：有序键值对集合，键唯一

```
#include <map>
using namespace std;

map<string, int> m;

// 常用方法：
m["apple"] = 5;             // 插入或修改
m.insert({"banana", 3});    // 插入键值对
m.erase("apple");           // 删除键
m.find("apple");            // 查找键，返回迭代器
m.count("apple");           // 统计键出现次数（0或1）
m.size();                   // 元素个数
m.empty();                  // 是否为空
m.clear();                  // 清空
m.lower_bound("a");         // 返回第一个键>="a"的迭代器
m.upper_bound("a");         // 返回第一个键>"a"的迭代器
```

### 🎯 std::multimap（多重键值对映射）

​**​特点​**​：有序键值对集合，键可重复

```
#include <map>
using namespace std;

multimap<string, int> mm;

// 常用方法：
mm.insert({"fruit", 5});     // 插入键值对
mm.insert({"fruit", 3});     // 可重复插入相同键
mm.count("fruit");          // 返回2
mm.erase("fruit");          // 删除所有"fruit"键
```

---

## 3. 无序关联容器（C++11）

### 🎯 std::unordered_set（无序集合）

​**​特点​**​：哈希表实现的集合，查找O(1)

```
#include <unordered_set>
using namespace std;

unordered_set<int> us;

// 常用方法：
us.insert(10);              // 插入元素
us.erase(10);               // 删除元素
us.find(10);                // 查找元素
us.count(10);               // 统计元素出现次数
us.size();                  // 元素个数
us.empty();                 // 是否为空
us.clear();                 // 清空
```

### 🎯 std::unordered_multiset（无序多重集合）

​**​特点​**​：哈希表实现的可重复集合

```
#include <unordered_set>
using namespace std;

unordered_multiset<int> ums;

// 常用方法：
ums.insert(10);             // 可重复插入
ums.insert(10);             // 现在有两个10
ums.count(10);              // 返回2
```

### 🎯 std::unordered_map（无序映射）

​**​特点​**​：哈希表实现的键值对映射

```
#include <unordered_map>
using namespace std;

unordered_map<string, int> um;

// 常用方法：
um["apple"] = 5;            // 插入或修改
um.insert({"banana", 3});    // 插入键值对
um.erase("apple");           // 删除键
um.find("apple");            // 查找键
um.count("apple");           // 统计键出现次数
um.size();                   // 元素个数
```

### 🎯 std::unordered_multimap（无序多重映射）

​**​特点​**​：哈希表实现的可重复键映射

```
#include <unordered_map>
using namespace std;

unordered_multimap<string, int> umm;

// 常用方法：
umm.insert({"fruit", 5});    // 插入键值对
umm.insert({"fruit", 3});    // 可重复插入相同键
umm.count("fruit");          // 返回2
```

---

## 4. 容器适配器

### 🎯 std::stack（栈）

​**​特点​**​：LIFO（后进先出）数据结构

```
#include <stack>
using namespace std;

stack<int> st;

// 常用方法：
st.push(10);                // 压栈
st.pop();                   // 弹栈（不返回元素！）
st.top();                   // 访问栈顶元素
st.empty();                 // 判断是否为空
st.size();                  // 元素个数
```

### 🎯 std::queue（队列）

​**​特点​**​：FIFO（先进先出）数据结构

```
#include <queue>
using namespace std;

queue<int> q;

// 常用方法：
q.push(10);                 // 入队
q.pop();                    // 出队（不返回元素！）
q.front();                  // 访问队首元素
q.back();                   // 访问队尾元素
q.empty();                  // 判断是否为空
q.size();                   // 元素个数
```

### 🎯 std::priority_queue（优先队列）

​**​特点​**​：自动排序的队列（默认大顶堆）

```
#include <queue>
using namespace std;

priority_queue<int> pq;      // 大顶堆

// 常用方法：
pq.push(10);                // 插入元素
pq.pop();                   // 删除最大元素
pq.top();                   // 访问最大元素（堆顶）
pq.empty();                 // 判断是否为空
pq.size();                  // 元素个数

// 小顶堆示例：
priority_queue<int, vector<int>, greater<int>> min_pq;
```

---

## 📊 容器选择指南

|容器|特点|适用场景|
|---|---|---|
|`vector`|动态数组，随机访问快|需要频繁随机访问|
|`deque`|双端队列，头尾操作快|需要头尾频繁操作|
|`list`|双向链表，任意位置插入快|需要频繁中间插入删除|
|`set/map`|红黑树，有序存储|需要有序且快速查找|
|`unordered_set/map`|哈希表，查找最快|只需要快速查找，不关心顺序|
|`stack`|LIFO栈|后进先出需求|
|`queue`|FIFO队列|先进先出需求|

## 🔄 通用操作方法

几乎所有容器都支持：

```
container.size();        // 元素个数
container.empty();       // 判断为空
container.clear();       // 清空容器
container.begin();       // 起始迭代器
container.end();         // 结束迭代器
container.rbegin();      // 反向起始迭代器
container.rend();        // 反向结束迭代器
container.swap(other);   // 交换两个容器
```