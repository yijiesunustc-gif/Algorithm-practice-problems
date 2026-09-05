### 题目二

- **编号 / 来源**：LeetCode 295. 数据流的中位数（Find Median from Data Stream）
- **考点类型**：堆（优先队列）/ 设计
- **难度评级**：⭐⭐⭐⭐ 困难

**题目描述：**

**中位数**是有序整数列表中的中间值。如果列表的大小是偶数，则没有中间值，中位数是中间两个值的平均值。

- 例如 `arr = [2,3,4]` 的中位数是 `3`
- 例如 `arr = [2,3]` 的中位数是 `(2 + 3) / 2 = 2.5`

设计一个支持以下两种操作的数据结构：

- `void addNum(int num)` - 从数据流中添加一个整数到数据结构中。
- `double findMedian()` - 返回目前所有元素的中位数。

**示例：**

```
输入：
["MedianFinder", "addNum", "addNum", "findMedian", "addNum", "findMedian"]
[[], [1], [2], [], [3], []]
输出：
[null, null, null, 1.5, null, 2.0]

解释：
MedianFinder medianFinder = new MedianFinder();
medianFinder.addNum(1);    // arr = [1]
medianFinder.addNum(2);    // arr = [1, 2]
medianFinder.findMedian(); // 返回 1.5
medianFinder.addNum(3);    // arr = [1, 2, 3]
medianFinder.findMedian(); // 返回 2.0
```

**提示：**

- `-10⁵ <= num <= 10⁵`
- 在调用 `findMedian` 时，数据结构中至少有一个元素
- 最多 `5 * 10⁴` 次调用 `addNum` 和 `findMedian`
### 题目二详解：数据流的中位数

#### 解题思路

本题要求动态维护一个数据流的中位数，核心挑战是：每次添加元素后都要能快速获取中位数。

**暴力思路**：每次添加元素后排序，时间复杂度 O (n log n)，数据量大时超时。

**最优解法：双堆（大顶堆 + 小顶堆）**

核心思想：将数据流分成两部分，用两个堆维护：

- **大顶堆 `maxHeap`**：存储较小的一半元素，堆顶是这一半的最大值（即左半部分的最后一个元素）。
- **小顶堆 `minHeap`**：存储较大的一半元素，堆顶是这一半的最小值（即右半部分的第一个元素）。

**维护规则：**

1. 保证 `maxHeap` 的元素个数要么等于 `minHeap`（总数偶数），要么比 `minHeap` 多 1（总数奇数）。
2. 保证 `maxHeap` 的堆顶 <= `minHeap` 的堆顶（即左半部分所有元素 <= 右半部分所有元素）。

**添加元素 `addNum(num)`：**

- 如果 `maxHeap` 为空或 `num <= maxHeap.top()`，加入 `maxHeap`；否则加入 `minHeap`。
- 加入后调整两个堆的大小平衡：
  - 如果 `maxHeap.size() > minHeap.size() + 1`：将 `maxHeap` 堆顶移到 `minHeap`。
  - 如果 `minHeap.size() > maxHeap.size()`：将 `minHeap` 堆顶移到 `maxHeap`。

**获取中位数 `findMedian()`：**

- 如果总数为奇数（`maxHeap.size() > minHeap.size()`）：中位数就是 `maxHeap.top()`。
- 如果总数为偶数：中位数是 `(maxHeap.top() + minHeap.top()) / 2.0`。

> 
> **为什么这样设计？** 中位数是排序后中间的数。大顶堆维护左半部分的最大值，小顶堆维护右半部分的最小值，这两个堆顶恰好就是中间的一个或两个数。堆的插入和弹出都是 O (log n)，获取堆顶是 O (1)，因此整体效率很高。

#### C++ 完整代码

```cpp
#include <queue>
#include <vector>
using namespace std;

class MedianFinder {
private:
    priority_queue<int> maxHeap;                  // 大顶堆：存较小的一半
    priority_queue<int, vector<int>, greater<int>> minHeap; // 小顶堆：存较大的一半

public:
    MedianFinder() {}

    void addNum(int num) {
        // 步骤1：根据 num 的大小决定加入哪个堆
        if (maxHeap.empty() || num <= maxHeap.top()) {
            maxHeap.push(num);
        } else {
            minHeap.push(num);
        }

        // 步骤2：平衡两个堆的大小
        // 大顶堆最多比小顶堆多1个元素
        if (maxHeap.size() > minHeap.size() + 1) {
            minHeap.push(maxHeap.top());
            maxHeap.pop();
        }
        // 小顶堆不能比大顶堆多
        else if (minHeap.size() > maxHeap.size()) {
            maxHeap.push(minHeap.top());
            minHeap.pop();
        }
    }

    double findMedian() {
        // 奇数个元素：中位数是大顶堆堆顶
        if (maxHeap.size() > minHeap.size()) {
            return maxHeap.top();
        }
        // 偶数个元素：中位数是两个堆顶的平均值
        return (maxHeap.top() + minHeap.top()) / 2.0;
    }
};
```

#### 复杂度分析

表格

| 操作 | 时间复杂度 | 空间复杂度 |
| --- | --- | --- |
| `addNum` | O (log n)，堆的插入和调整 | O (n)，存储所有元素 |
| `findMedian` | O (1)，直接取堆顶 | - |

#### 优化方向与同类题拓展

- **进阶挑战**：LeetCode 480. 滑动窗口中位数（在滑动窗口中维护中位数，需要支持删除操作，双堆 + 延迟删除）
- **同类题**：
  - LeetCode 703. 数据流中的第 K 大元素（小顶堆维护前 K 大）
  - LeetCode 1046. 最后一块石头的重量（大顶堆）
  - LeetCode 378. 有序矩阵中第 K 小的元素（堆 / 二分）
  - 剑指 Offer 41. 数据流中的中位数（同题）
- **其他解法**：
  - **有序数组 + 二分插入**：插入 O (n)（移动元素），查询 O (1)。
  - **平衡二叉搜索树（BST）**：如 C++ `multiset`，插入 O (log n)，用迭代器维护中位数位置，查询 O (1)。
- **技巧总结**：
  - "动态维护中位数" → 直接想到**双堆**。
  - 大顶堆存小的一半，小顶堆存大的一半，堆顶即中位数。
  - 大小平衡规则：大顶堆 = 小顶堆，或大顶堆 = 小顶堆 + 1。
  - C++ 中 `priority_queue` 默认是大顶堆；小顶堆需要 `priority_queue<int, vector<int>, greater<int>>`。
