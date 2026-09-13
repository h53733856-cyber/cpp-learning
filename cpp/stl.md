# vector

## size() vs capacity()

`size()` 是当前真正存放了多少个元素，`capacity()` 是当前已经分配的内存空间最多能够容纳多少个元素而不需要重新分配。因为不可能 `push_back` 每次都重新申请内存，如果每次都重新申请，可能会变成：第一次申请 1，第二次申请 2 并复制原来的数据，第三次申请 3 再复制，第四次申请 4 再复制……效率太低。所以 `vector` 通常会一次申请一块更大的连续内存，容量不够时重新分配，然后把原来的元素移动或复制到新内存，释放旧内存，再添加新元素。具体扩容多少由实现决定，不应该认为一定是 2 倍。

## reserve() vs resize()

`v.reserve(100);` 的意思是提前准备至少能够容纳 100 个元素的空间，此时 `size()` 仍然是 0，因为只是准备了空间，并没有创建元素。`reserve()` 主要影响 `capacity()`，不会改变 `size()`。而 `resize(100)` 是直接把元素数量调整成 100，`size()` 此时就是 100。注意：如果 `resize(n)` 的 `n` 比原来的 `size()` 小，就删除末尾多出来的元素；如果比原来的 `size()` 大，就新增元素，对于 `vector<int>` 新增的元素一般为 0。可以简单记成：`reserve()` 是“准备空间”，`resize()` 是“调整元素数量”。

## push_back() vs emplace_back()

`push_back()` 可以理解为把一个已经存在的对象放进去，而 `emplace_back()` 可以把构造对象所需要的参数直接交给容器，让容器在末尾直接构造对象。例如：

```cpp
std::vector<std::string> v;
std::string s = "hello";

v.push_back(s);              // 把已经存在的 s 放进去
v.emplace_back("world");     // 直接用 "world" 构造 string
```

不要简单理解成 `emplace_back()` 一定比 `push_back()` 快，具体是否有性能优势要看对象类型和使用方式。

## empty()

`empty()` 是判断 `size()` 是否为 0。`v.empty()` 为 `true` 表示当前没有元素。它判断的是有没有元素，不是 `capacity()` 是否为 0。

## operator[] vs at()

`v[100]` 不检查越界。如果当前容器只有 3 个元素，访问 `v[100]` 属于未定义行为，可能崩溃，也可能出现其他结果。`v.at(100)` 会进行边界检查，如果越界，会抛出 `std::out_of_range` 异常。

## vector 中指针、引用、迭代器失效

```cpp
std::vector<int> v = {1, 2, 3};
int* p = &v[0];

v.push_back(4);

std::cout << *p << std::endl;
```

`p` 保存的是 1 所在内存的地址，但是 `push_back()` 有可能触发重新分配。重新分配后，原来的元素会被移动或复制到新的连续内存，旧内存被释放，而 `p` 不会自动更新，因此如果发生了重新分配，`p` 就变成了悬空指针，继续解引用属于未定义行为。除了指针，指向 `vector` 元素的引用和迭代器也可能因为重新分配而失效。

## reserve() 的一个例子

```cpp
std::vector<int> v;
v.reserve(10);

for (int i = 0; i < 5; ++i) {
    v.push_back(i);
}
```

执行完之后，`size() == 5`，`capacity() >= 10`，并且这 5 次 `push_back()` 不会因为容量不足而触发重新分配。

如果把 `reserve(10)` 改成：

```cpp
v.resize(10);
```

那么此时 `size() == 10`，已经有 10 个元素了，再执行 5 次 `push_back()` 后，`size() == 15`。因为 `push_back()` 是在尾部追加元素，不会覆盖原来的 10 个元素。

## 时间复杂度

`vector` 底层是连续的动态数组。访问 `v[i]` 时，可以通过首地址加偏移量直接计算出元素地址，不需要遍历，因此随机访问是 `O(1)`，和下标大小无关。`push_back()` 的均摊复杂度是 `O(1)`，但单次操作不一定是 `O(1)`；如果容量不足需要扩容并搬移原来的 `n` 个元素，这一次可能是 `O(n)`。

---

# string

## size() vs length()

对于 `std::string`，`size()` 和 `length()` 没有实际区别，返回的数值相同。需要注意的是，它们统计的是字符串中的**字节数**，不是一定意义上的“人类所看到的字符数”。例如 UTF-8 编码下：

```cpp
std::string s = "你好";
```

通常 `s.size() == 6`，因为一个汉字通常占 3 个字节。因此处理中文、emoji 等非 ASCII 字符时，不能简单把 `size()` 当成人类字符数量。

## empty()

和 `vector` 一样，`s.empty()` 用来判断字符串是否为空，也就是判断 `size() == 0`。

## push_back()

`push_back()` 添加的是一个字符：

```cpp
s.push_back('d');    // 正确
s.push_back("d");    // 错误
```

如果要追加一个字符串，可以使用 `+=` 或 `append()`。

## += vs append()

```cpp
std::string s = "hello";

s += " world";
```

执行结果是 `"hello world"`，也可以写成：

```cpp
s.append(" world");
```

## substr()

```cpp
std::string s = "hello world";
std::string x = s.substr(0, 5);
```

得到 `"hello"`。`substr(pos, count)` 表示从 `pos` 开始取 `count` 个字符。如果只给一个参数：

```cpp
s.substr(6);
```

表示从下标 6 一直取到字符串末尾。

## find()

```cpp
std::string s = "hello world";
auto pos = s.find("world");
```

得到 `pos == 6`，也就是 `"world"` 中 `w` 的下标。如果找不到，会返回 `std::string::npos`。

一个非常容易犯的错误是：

```cpp
if (s.find("hello")) {
    // ...
}
```

不能这样判断是否找到，因为如果 `"hello"` 正好出现在下标 0，返回值是 0，在条件判断中会被当成 `false`。正确写法是：

```cpp
if (s.find("hello") != std::string::npos) {
    // 找到了
}
```

## operator[] vs at()

和 `vector` 一样，`s[i]` 不进行越界检查，`s.at(i)` 会进行边界检查，越界时抛出 `std::out_of_range`。

---

# map

## 基本特点

`map` 最重要的特点是**有序**，会按照 `key` 进行排序，默认升序。底层通常使用红黑树等平衡树实现，因此查找、插入、删除的复杂度通常都是 `O(log n)`。

## operator[]

对于：

```cpp
std::map<std::string, int> m;

m["apple"] = 10;
std::cout << m["apple"] << std::endl;
std::cout << m["banana"] << std::endl;
std::cout << m.size() << std::endl;
```

`m["banana"]` 如果 `banana` 不存在，`operator[]` 会创建 `banana`，并给它一个 `int` 的默认值 `0`，然后返回这个 `int` 的引用。因此执行 `m["banana"]` 本身就可能改变 `map`。

所以如果只是想判断某个 key 是否存在，不应该使用 `operator[]`。

## find() / count() / contains()

`find()` 返回一个迭代器：

```cpp
auto it = m.find("apple");

if (it != m.end()) {
    std::cout << it->first;   // key
    std::cout << it->second;  // value
}
```

如果没有找到，返回 `m.end()`。

如果只是想判断 key 是否存在，可以使用：

```cpp
if (m.count("apple")) {
    // 存在
}
```

对于 `map`，因为 key 不能重复，所以 `count()` 的结果只有 `0` 或 `1`。


`count()` / `contains()` 更适合“只判断存不存在”，`find()` 更适合“找到以后还要使用这个元素”。

## insert() vs emplace()

```cpp
m.insert({"apple", 10});
m.emplace("banana", 20);
```

二者都是插入元素。如果 key 已经存在，它们都不会直接覆盖原来的 value。需要修改已有 key 对应的 value，可以使用：

```cpp
m["apple"] = 100;
```

`emplace()` 的特点是可以直接使用构造参数构造元素，不需要先单独创建一个完整对象。不要简单认为 `emplace()` 一定比 `insert()` 快。

## erase()

可以通过 key 删除：

```cpp
m.erase("apple");
```

也可以通过迭代器删除：

```cpp
auto it = m.find("apple");

if (it != m.end()) {
    m.erase(it);
}
```

---

# unordered_map

## 基本特点

`unordered_map` 底层通常使用哈希表实现，key 没有排序，遍历顺序也不应该依赖。查找、插入、删除的**平均**时间复杂度一般是 `O(1)`，最坏情况下可能退化到 `O(n)`。

不同的 key 经过 hash 后可能进入同一个 bucket，这叫做**哈希冲突**。冲突较多时，查找性能可能下降。

## operator[] / find() / count() / contains()

这些基本用法和 `map` 类似：

```cpp
std::unordered_map<std::string, int> m;

m["apple"] = 10;

m["banana"];       // 如果不存在，会创建 banana -> 0

m.find("apple");   // 查找，不会因为查找而创建

m.count("apple");  // 存在返回 1，不存在返回 0

m.contains("apple"); // C++20，判断是否存在
```

如果只是判断 key 是否存在，优先考虑 `find()`、`count()` 或 C++20 的 `contains()`，不要使用 `operator[]`。

## bucket / load_factor

`unordered_map` 会把元素放到不同的 bucket 中，可以通过：

```cpp
m.bucket_count();      // bucket 的数量
m.bucket(key);         // 某个 key 属于哪个 bucket
m.bucket_size(n);      // 第 n 个 bucket 中有多少元素
m.load_factor();       // 当前负载因子，大致为 size / bucket_count
m.max_load_factor();   // 最大允许负载因子
```

理解这些 API 的核心是：`unordered_map` 的性能和“元素分布在多少个 bucket 中、冲突是否严重”有关。

## reserve() vs rehash()

`reserve(n)` 主要从“准备容纳至少 n 个元素”的角度出发，必要时会重新分配 bucket 并进行 rehash：

```cpp
m.reserve(1000);
```

表示提前为大约 1000 个元素做好准备，减少之后不断插入导致的 rehash。

`rehash(n)` 则是从 bucket 数量的角度出发：

```cpp
m.rehash(100);
```

要求容器至少有足够数量的 bucket，并重新进行哈希分布。实际 bucket 数量可能比传入的数值更大。

可以简单记：

```text
reserve → 我准备放多少个元素
rehash  → 我希望有多少个 bucket
```

## 迭代器失效

`unordered_map` 在发生 rehash 时，原来的迭代器会失效。因此执行 `reserve()`、`rehash()`，或者某次插入因为负载因子导致 rehash 后，不应该继续使用之前保存的迭代器。

需要注意：rehash 不会因为重新分配 bucket 而直接让元素本身的引用和指针失效；真正被删除的元素，其对应的引用、指针、迭代器当然都会失效。

---

# map vs unordered_map

最核心的选择：

```text
map
→ key 有序
→ 通常 O(log n)
→ 可以利用有序性进行遍历、范围查询等

unordered_map
→ key 无序
→ 平均 O(1)
→ 主要关心快速通过 key 查找 value
```

如果我需要按照 key 排序、进行范围查询等操作，通常选择 `map`；如果我只关心通过 key 快速查找，不需要顺序，通常优先考虑 `unordered_map`。