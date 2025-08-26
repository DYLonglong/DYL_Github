# C++ STL 中 `resize`和 `reserve`的区别详解

`resize`和 `reserve`都是 `std::vector`中用于容量管理的重要方法，但它们的作用和用途有本质区别。

## 1. 基本概念对比

|特性|`resize()`|`reserve()`|
|---|---|---|
|​**​作用对象​**​|改变 vector 的 ​**​size​**​（元素数量）|改变 vector 的 ​**​capacity​**​（容量）|
|​**​元素影响​**​|可能创建或销毁元素|不影响现有元素|
|​**​内存分配​**​|可能分配或释放内存|只分配内存，不初始化元素|
|​**​主要用途​**​|调整元素数量|预分配内存避免频繁重新分配|

## 2. 详细区别说明

### `resize(n)`- 调整大小

```
void resize(size_type n);
void resize(size_type n, const value_type& val);
```

​**​作用​**​：改变 vector 中元素的数量

- 如果 `n < current_size`：删除多余元素（从末尾开始删）
    
- 如果 `n > current_size`：添加新元素（默认构造或使用指定值）
    

​**​示例​**​：

```
vector<int> v = {1, 2, 3, 4, 5};

v.resize(3);    // v = {1, 2, 3}，删除后2个元素
v.resize(5);    // v = {1, 2, 3, 0, 0}，添加2个0
v.resize(7, 9); // v = {1, 2, 3, 0, 0, 9, 9}，添加2个9
```

### `reserve(n)`- 预留容量

```
void reserve(size_type n);
```

​**​作用​**​：预分配至少能容纳 n 个元素的内存空间

- 如果 `n > current_capacity`：分配新的内存空间
    
- 如果 `n <= current_capacity`：什么都不做
    
- ​**​不影响元素数量​**​，只是预留空间
    

​**​示例​**​：

```
vector<int> v;

v.reserve(100); // 分配至少100个元素的空间
cout << v.size();     // 输出: 0（元素数量不变）
cout << v.capacity(); // 输出: >=100（容量增加）

// 现在添加元素不会触发重新分配
for (int i = 0; i < 100; ++i) {
    v.push_back(i);
}
```

## 3. 内存布局对比

### 使用 `resize()`

```
vector<int> v;
v.resize(5); // 分配内存并构造5个元素
```

内存状态：

```
[ 0 | 0 | 0 | 0 | 0 ]  // 5个已构造的元素
size = 5, capacity = 5
```

### 使用 `reserve()`

```
vector<int> v;
v.reserve(5); // 只分配内存，不构造元素
```

内存状态：

```
[ ? | ? | ? | ? | ? ]  // 5个未初始化的内存位置
size = 0, capacity = 5
```

## 4. 性能影响

### `resize()`的性能考虑

```
vector<int> v;

// 不好的做法：频繁resize
for (int i = 0; i < 1000; ++i) {
    v.resize(i + 1); // 每次都会重新分配和构造
    v[i] = i;
}

// 好的做法：先reserve再push_back
v.clear();
v.reserve(1000);
for (int i = 0; i < 1000; ++i) {
    v.push_back(i); // 不会重新分配
}
```

### `reserve()`的最佳实践

```
vector<int> processData() {
    vector<int> result;
    
    // 预估需要处理10000个元素
    result.reserve(10000);
    
    for (int i = 0; i < 10000; ++i) {
        if (shouldInclude(i)) {
            result.push_back(process(i));
        }
    }
    
    // 如果实际元素较少，可以释放多余内存
    result.shrink_to_fit();
    return result;
}
```

## 5. 实际应用场景

### 适合使用 `resize()`的场景

```
// 场景1：需要固定大小的数组
vector<int> createMatrix(int rows, int cols) {
    vector<int> matrix;
    matrix.resize(rows * cols, 0); // 创建并初始化矩阵
    return matrix;
}

// 场景2：批量修改元素值
vector<string> names(10); // 10个空字符串
names.resize(5);         // 删除后5个元素
names.resize(8, "Unknown"); // 添加3个"Unknown"
```

### 适合使用 `reserve()`的场景

```
// 场景1：预先知道要添加大量元素
vector<Data> processLargeDataset() {
    vector<Data> results;
    results.reserve(1000000); // 预分配空间
    
    while (hasMoreData()) {
        results.push_back(processNext());
    }
    
    return results;
}

// 场景2：避免插入操作中的重新分配
vector<int> mergeVectors(const vector<int>& a, const vector<int>& b) {
    vector<int> result;
    result.reserve(a.size() + b.size()); // 预分配足够空间
    
    result.insert(result.end(), a.begin(), a.end());
    result.insert(result.end(), b.begin(), b.end());
    
    return result;
}
```

## 6. 错误用法示例

### 错误1：混淆 size 和 capacity

```
vector<int> v;
v.reserve(10); // 只分配空间，不创建元素

// 错误：访问未构造的元素
v[0] = 1;      // 未定义行为！
v.at(0) = 1;   // 抛出 std::out_of_range 异常

// 正确做法：使用resize或push_back
v.resize(10);  // 先创建元素
v[0] = 1;      // 现在安全了
```

### 错误2：不必要的resize

```
vector<int> v;

// 不好：每次resize都可能重新分配
for (int i = 0; i < 1000; ++i) {
    v.resize(v.size() + 1);
    v[i] = i;
}

// 更好：使用reserve + push_back
v.reserve(1000);
for (int i = 0; i < 1000; ++i) {
    v.push_back(i);
}
```

## 7. 总结对比表

|方面|`resize(n)`|`reserve(n)`|
|---|---|---|
|​**​改变什么​**​|元素数量 (size)|内存容量 (capacity)|
|​**​元素构造​**​|会构造或销毁元素|不会构造元素|
|​**​内存分配​**​|可能分配或释放内存|只分配内存|
|​**​访问安全​**​|调整后可以安全访问 [0, n-1]|不能直接访问预留的空间|
|​**​典型用途​**​|设置确切的元素数量|预分配空间避免重新分配|
|​**​性能影响​**​|可能触发元素构造/析构|只影响内存分配|

## 8. 最佳实践建议

1. ​**​需要确切元素数量​**​ → 使用 `resize()`
    
2. ​**​需要预分配空间优化性能​**​ → 使用 `reserve()`
    
3. ​**​不确定最终大小但知道上限​**​ → 使用 `reserve(上限)`+ `push_back()`
    
4. ​**​既要预分配又要初始化​**​ → 使用 `reserve()`+ `resize()`或直接 `resize()`
    

正确理解和使用 `resize`与 `reserve`可以显著提高程序的性能和内存使用效率。