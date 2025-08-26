

## 1. 基本概念和定义

### 什么是 priority_queue？

`priority_queue`是一个容器适配器，提供常数时间最大元素访问，对数时间插入和删除。

```
#include <queue>
#include <iostream>
using namespace std;

// 默认构造：大顶堆
priority_queue<int> pq; 

// 带容器和比较器的构造
priority_queue<int, vector<int>, greater<int>> min_pq;
```

## 2. 基本操作和方法

### 核心方法

```
priority_queue<int> pq;

// 插入元素
pq.push(10);
pq.push(5);
pq.push(20);

// 访问顶部元素（最大元素）
cout << pq.top();    // 输出: 20

// 删除顶部元素
pq.pop();            // 删除20
cout << pq.top();    // 输出: 10

// 其他操作
cout << pq.size();   // 元素个数: 2
cout << pq.empty();  // 是否为空: 0 (false)

// 清空队列
while (!pq.empty()) {
    pq.pop();
}
```

## 3. 三种常用的优先队列类型

### 3.1 最大堆（默认）

```
// 默认是大顶堆，最大元素在顶部
priority_queue<int> max_heap;
max_heap.push(3);
max_heap.push(1);
max_heap.push(4);
max_heap.push(2);

cout << max_heap.top();  // 输出: 4 (最大元素)
```

### 3.2 最小堆

```
// 使用 greater<int> 作为比较器创建小顶堆
priority_queue<int, vector<int>, greater<int>> min_heap;
min_heap.push(3);
min_heap.push(1);
min_heap.push(4);
min_heap.push(2);

cout << min_heap.top();  // 输出: 1 (最小元素)
```

### 3.3 自定义比较器堆

```
// 自定义比较函数
auto cmp = [](int left, int right) { 
    return left > right;  // 小顶堆
};

// 使用 decltype 声明类型
priority_queue<int, vector<int>, decltype(cmp)> custom_heap(cmp);
```

## 4. 自定义比较器详解

### 4.1 函数对象（Functor）

```
struct CompareByAbs {
    bool operator()(int a, int b) const {
        return abs(a) < abs(b);  // 按绝对值的大顶堆
    }
};

priority_queue<int, vector<int>, CompareByAbs> abs_heap;
abs_heap.push(-5);
abs_heap.push(3);
abs_heap.push(-7);
cout << abs_heap.top();  // 输出: -7 (绝对值最大)
```

### 4.2 Lambda 表达式

```
// 按数字的各位数之和排序
auto digit_sum_cmp = [](int a, int b) {
    auto sum_digits = [](int n) {
        int sum = 0;
        while (n) { sum += n % 10; n /= 10; }
        return sum;
    };
    return sum_digits(a) < sum_digits(b);
};

priority_queue<int, vector<int>, decltype(digit_sum_cmp)> digit_sum_heap(digit_sum_cmp);
```

### 4.3 标准库函数对象

```
#include <functional>

// 使用标准库函数对象
priority_queue<int, vector<int>, greater<int>> min_heap1;
priority_queue<int, vector<int>, less<int>> max_heap1;      // 默认，等同于 priority_queue<int>

// 对于自定义类型
priority_queue<pair<int, int>, vector<pair<int, int>>, 
               greater<pair<int, int>>> pair_min_heap;
```

## 5. 自定义数据类型的使用

### 5.1 结构体优先队列

```
struct Task {
    int priority;
    string name;
    time_t timestamp;
    
    // 重载 < 运算符（用于默认大顶堆）
    bool operator<(const Task& other) const {
        return priority < other.priority;  // 优先级高的在前
    }
};

// 使用默认比较
priority_queue<Task> task_queue;
task_queue.push({3, "Low priority task", time(nullptr)});
task_queue.push({1, "High priority task", time(nullptr)});

cout << task_queue.top().name;  // 输出: "High priority task"
```

### 5.2 使用外部比较器

```
struct Student {
    string name;
    double gpa;
    int age;
};

// 自定义比较器：先按GPA降序，再按年龄升序
auto student_cmp = [](const Student& a, const Student& b) {
    if (a.gpa != b.gpa) return a.gpa < b.gpa;  // GPA高的优先
    return a.age > b.age;                      // 年龄小的优先
};

priority_queue<Student, vector<Student>, decltype(student_cmp)> student_queue(student_cmp);
```

## 6. 底层容器选择

### 6.1 使用 vector（默认）

```
// 默认底层容器是 vector
priority_queue<int> pq1;  // 等价于:
priority_queue<int, vector<int>, less<int>> pq2;
```

### 6.2 使用 deque

```
// 使用 deque 作为底层容器
priority_queue<int, deque<int>> pq;
pq.push(1);
pq.push(2);
pq.push(3);

// deque 支持随机访问，但priority_queue不需要这个特性
```

### 6.3 自定义容器要求

底层容器必须满足：

- 随机访问迭代器
    
- `front()`, `push_back()`, `pop_back()`方法
    
- 满足序列容器要求
    

## 7. 实际应用场景

### 7.1 Top K 问题

```
// 找出数组中前K大的元素
vector<int> findTopK(const vector<int>& nums, int k) {
    priority_queue<int, vector<int>, greater<int>> min_heap;  // 小顶堆
    
    for (int num : nums) {
        min_heap.push(num);
        if (min_heap.size() > k) {
            min_heap.pop();  // 保持堆中只有K个元素
        }
    }
    
    vector<int> result;
    while (!min_heap.empty()) {
        result.push_back(min_heap.top());
        min_heap.pop();
    }
    reverse(result.begin(), result.end());  // 从大到小排序
    return result;
}
```

### 7.2 合并K个有序链表

```
struct ListNode {
    int val;
    ListNode* next;
};

struct CompareListNode {
    bool operator()(ListNode* a, ListNode* b) {
        return a->val > b->val;  // 小顶堆
    }
};

ListNode* mergeKLists(vector<ListNode*>& lists) {
    priority_queue<ListNode*, vector<ListNode*>, CompareListNode> min_heap;
    
    // 将所有链表的头节点加入堆中
    for (auto node : lists) {
        if (node) min_heap.push(node);
    }
    
    ListNode dummy(0);
    ListNode* current = &dummy;
    
    while (!min_heap.empty()) {
        ListNode* smallest = min_heap.top();
        min_heap.pop();
        
        current->next = smallest;
        current = current->next;
        
        if (smallest->next) {
            min_heap.push(smallest->next);
        }
    }
    
    return dummy.next;
}
```

### 7.3 任务调度

```
struct ScheduledTask {
    int priority;
    time_t execute_time;
    string task_id;
    
    bool operator<(const ScheduledTask& other) const {
        // 先按执行时间，再按优先级
        if (execute_time != other.execute_time) 
            return execute_time > other.execute_time;  // 时间早的优先
        return priority < other.priority;              // 优先级高的优先
    }
};

priority_queue<ScheduledTask> task_scheduler;
```

## 8. 性能特点和复杂度分析

### 时间复杂度：

- `push()`: O(log n)
    
- `pop()`: O(log n)
    
- `top()`: O(1)
    
- `empty()`: O(1)
    
- `size()`: O(1)
    

### 空间复杂度：O(n)

### 底层实现：通常是二叉堆

```
// 伪代码表示堆操作：
void push(const T& value) {
    container.push_back(value);
    push_heap(container.begin(), container.end(), comp);
}

void pop() {
    pop_heap(container.begin(), container.end(), comp);
    container.pop_back();
}
```

## 9. 注意事项和最佳实践

### 9.1 空队列访问

```
priority_queue<int> pq;
// cout << pq.top();  // ❌ 未定义行为，可能崩溃

if (!pq.empty()) {
    cout << pq.top();  // ✅ 安全访问
}
```

### 9.2 自定义比较器规则

```
// 必须满足严格弱序
struct ValidCompare {
    bool operator()(int a, int b) const {
        return a > b;  // 正确：满足反对称性、传递性
    }
};

struct InvalidCompare {
    bool operator()(int a, int b) const {
        return a >= b;  // ❌ 不满足反对称性
    }
};
```

### 9.3 性能优化

```
// 如果知道元素数量，可以预先分配空间
vector<int> container;
container.reserve(1000);
priority_queue<int, vector<int>> pq(less<int>(), move(container));
```

## 10. 完整示例代码

### 综合示例

```
#include <iostream>
#include <queue>
#include <vector>
#include <algorithm>
#include <functional>
using namespace std;

// 自定义数据类型
struct Process {
    int pid;
    int priority;
    string name;
    
    // 重载 < 用于默认大顶堆
    bool operator<(const Process& other) const {
        return priority < other.priority;
    }
};

// 自定义比较器
struct ProcessComparator {
    bool operator()(const Process& a, const Process& b) const {
        if (a.priority != b.priority) 
            return a.priority < b.priority;
        return a.pid > b.pid;
    }
};

int main() {
    // 1. 默认大顶堆
    priority_queue<int> max_heap;
    for (int i : {3, 1, 4, 2}) max_heap.push(i);
    cout << "Max heap top: " << max_heap.top() << endl;  // 4
    
    // 2. 小顶堆
    priority_queue<int, vector<int>, greater<int>> min_heap;
    for (int i : {3, 1, 4, 2}) min_heap.push(i);
    cout << "Min heap top: " << min_heap.top() << endl;  // 1
    
    // 3. 自定义数据类型
    priority_queue<Process> process_queue;
    process_queue.push({1, 3, "Process A"});
    process_queue.push({2, 1, "Process B"});
    process_queue.push({3, 2, "Process C"});
    cout << "Highest priority process: " << process_queue.top().name << endl;
    
    // 4. 使用自定义比较器
    priority_queue<Process, vector<Process>, ProcessComparator> custom_queue;
    custom_queue.push({1, 3, "Process A"});
    custom_queue.push({2, 3, "Process B"});  // 相同优先级，PID小的优先
    custom_queue.push({3, 2, "Process C"});
    cout << "Custom queue top: " << custom_queue.top().name << endl;
    
    return 0;
}
```

## 总结表格

|特性|说明|示例|
|---|---|---|
|​**​默认行为​**​|大顶堆|`priority_queue<int>`|
|​**​小顶堆​**​|使用 `greater<int>`|`priority_queue<int, vector<int>, greater<int>>`|
|​**​自定义比较​**​|函数对象或lambda|`decltype(cmp)`|
|​**​时间复杂度​**​|插入删除O(log n)，访问O(1)|-|
|​**​底层容器​**​|默认`vector`，可用`deque`|`priority_queue<int, deque<int>>`|
|​**​适用场景​**​|Top K，任务调度，Dijkstra算法|-|

​**​选择指南​**​：

- 需要最大元素：默认大顶堆
    
- 需要最小元素：使用`greater<int>`
    
- 复杂排序规则：自定义比较器
    
- 性能敏感：预分配底层容器空间