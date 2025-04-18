# Top-k问题

> 给定一个长度为k的无序数组 nums ，请返回数组中最大的个元素。

### 1. 方法一：遍历选择

可以进行k轮遍历，分别在每轮中提取第 1、2、...、k 大的元素，时间复杂度为 O(nk)

此方法只适用于 k<=n 的情况，因为当 k与n 比较接近时，其时间复杂度趋向于 O(n²) ，非常耗时

### 2. 方法二：排序

可以先对数组 nums 进行排序，再返回最右边的k个元素，时间复杂度为 O(nlogn)

显然，该方法`超额`完成任务，因为只需要找出最大的 k 个元素即可，而不需要排序其他元素

### 3. 方法三：堆

1. 初始化一个小顶堆，其堆顶元素最小

2. 先将数组的前 k 个元素依次入堆

3. 从第 k+1 个元素开始，若当前元素大于堆顶元素，则将堆顶元素出堆，并将当前元素入堆

4. 遍历完成后，堆中保存的就是最大的 k 个元素

    ```java
    /* 基于堆查找数组中最大的 k 个元素 */
    Queue<Integer> topKHeap(int[] nums, int k) {
        // 初始化小顶堆
        Queue<Integer> heap = new PriorityQueue<Integer>();
        // 将数组的前 k 个元素入堆
        for (int i = 0; i < k; i++) {
            heap.offer(nums[i]);
        }
        // 从第 k+1 个元素开始，保持堆的长度为 k
        for (int i = k; i < nums.length; i++) {
            // 若当前元素大于堆顶元素，则将堆顶元素出堆、当前元素入堆
            if (nums[i] > heap.peek()) {
                heap.poll();
                heap.offer(nums[i]);
            }
        }
        return heap;
    }
    ```

总共执行了 n 轮入堆和出堆，堆的最大长度为 k ，因此时间复杂度为 O(nlogk)

当 k 较小时，时间复杂度趋向于 O(n) ; 当 
k 较大时，时间复杂度不会超过 O(nlogn)

另外，该方法适用于动态数据流的使用场景，在不断加入数据时，可与持续维护堆内的元素，从而实现最大的 k 个元素的动态更新