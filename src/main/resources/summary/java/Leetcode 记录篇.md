# 十七、Leetcode 记录篇

### 1.1. 数据结构 - 链表

#### 21. 合并有序链表 | easy 

##### 1）迭代法 | O（m + n）

- **思路**：
  1. 新建一个节点作为新返回链表的头结点，它在 L1、L2 比较完之后，把较小者加入 next 指针，然后往下移动，继续比较 L1、L2，直到 L1 或者 L2 任意一个遍历完。
  2. 如果 L1 先遍历完，则把 L2 剩余的节点加入这个 next 指针。
  3. 否则，如果 L2 先遍历完，则把 L1 剩余的节点加入这个 next 指针。
- **结论**：合并链表首先要想到的思路，实现简单，面试可用（0ms，100%）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        if(list1 == null) {
            return list2;
        }
        if(list2 == null) {
            return list1;
        }

        ListNode head = new ListNode();
        ListNode pre = head;
        
        while(list1 != null && list2 != null) {
            if(list1.val <= list2.val) {
                pre.next = list1;
                list1 = list1.next;
                pre = pre.next;
            } else {
                pre.next = list2;
                list2 = list2.next;
                pre = pre.next;
            }
        }
        pre.next = list1 == null? list2 : list1;

        return head.next;
    }
}
```

##### 2）递归法 | O（m + n）

- **思路**：思路很清晰，实现简单，不关心细节，站在宏观角度来看待合并这件事。
  1. 如果当前节点 L1 比 L2 小，可以看作保留 L1 当前节点，递归调用子过程保证 L1.next 和 L2 剩余节点合并，合并完毕后返回 L1 即为最终答案。
  2. 如果当前节点 L2 比 L1 小，可以看作保留 L2 当前节点，递归调用子过程保证 L2.next 和 L1 剩余节点合并，合并完毕后返回 L2 即为最终答案。
- **缺点**：时间复杂度虽然和迭代法的一样（0ms，100%），但额外空间复杂度上，迭代法只需要额外几个变量O（1），而递归法却是每递归一次就需要一次额外空间，最终为 O（m + n）的额外空间复杂度。
- **结论**：合并链表还是比较适合用**迭代法**来实现。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        if(list1 == null) {
            return list2;
        }
        if(list2 == null) {
            return list1;
        }
        if(list1.val <= list2.val) {
            list1.next = mergeTwoLists(list1.next, list2);
            return list1;
        } else {
            list2.next = mergeTwoLists(list1, list2.next);
            return list2;
        }
    }
}
```

#### 23. 合并 K 个升序链表 | hard

##### 1）小根堆法 | O（n * logn）

- 思路：
  1. 参考两个有序链表合并的迭代法，即使用一个前置节点，不断地在链表比较之后，把小者加入 next 指针，周而复始。
  2. 不过不同的是，由于这里是 K 个有序链表，比较采用的是小根堆的做法，通过一个优先级队列来实现，从队列取出结果就相当于完成一次 K 个有序链表头节点的比较，直到队列元素被取完，合并结束。
- **结论**：时间上还行（4ms，69.27%），实现也简单，面试可用！

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        if(lists == null || lists.length == 0) {
            return null;
        }
        if(lists.length == 1) {
            return lists[0];
        }
        
        PriorityQueue<ListNode> queue = new PriorityQueue<>((o1, o2) -> o1.val - o2.val);
        for(ListNode listNode : lists) {
            if(listNode != null) {
                queue.add(listNode);
            }
        }

        ListNode head = new ListNode();

        ListNode pre = head;
        while(!queue.isEmpty()) {
            ListNode listNode = queue.poll();
            pre.next = listNode;
            listNode = listNode.next;
            if(listNode != null) {
                queue.add(listNode);
            }
            pre = pre.next;
        }

        return head.next;
    }
}
```

#### 24. 两两交换链表中的节点 | medium

##### 1）模拟交换法 | O（n）

- **思路**：
  1. 类似于数组交换那样，设计一个 swap 函数，不同的是， 链表的 swap 还需要一个 pre 的前节点，作为 from 引用的持有，以便交换后能持有 to 引用。
  2. 然后，swap 函数做的事情就是，from 节点为 null，那么 pre 就持有 to 节点，反之，to 节点为 null，那么 pre 就持有 from 节点。
  3. 接着，from 和 to 互相交换 next 引用，实现节点的交换，pre 重新持有交换后的 to 节点即可。
  4. 而主方法，则是先构造一个虚拟的头前节点 hp，用于持有交换后的 head 引用，剩余节点则是通过不断调用 swap 函数来完成相邻点的交换。
  5. 最后，则返回 hp.next 节点，即交换后的 head 引用即可。
- **结论**：时间，0 ms，100%，空间，39.3 mb，18.39%，时间上，由于需要遍历一次链表，所以时间复杂度为 O（n），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode swapPairs(ListNode head) {
        if(head == null || head.next == null) {
            return head;
        }
        
        ListNode hp = new ListNode();
        hp.next = head;
        swap(hp, head, head.next);

        ListNode pre = hp.next.next;
        while(pre != null && pre.next != null && pre.next.next != null) {
            swap(pre, pre.next, pre.next.next);
            pre = pre.next.next;
        }

        return hp.next;
    }

    private void swap(ListNode pre, ListNode from, ListNode to) {
        if(to == null) {
            pre.next = from;
            return;
        }
        if(from == null) {
            pre.next = to;
            return;
        }

        from.next = to.next;
        to.next = from;
        pre.next = to;
    }
}
```

##### 2）递归法 | O（n）

- **思路**：利用本身的 swapPairs 进行递归，即获取 next 作为新的头节点、递归交换剩余的节点、组装交换当前节点、并返回新头节点即可。
- **结论**：时间，0 ms，100%，空间，39.2 mb，19.47%，时间上，由于需要遍历一次链表，所以时间复杂度为 O（n），空间上，由于递归深度为 n，所以额外空间复杂度为 O（n）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode swapPairs(ListNode head) {
        if(head == null || head.next == null) {
            return head;
        }
        
        ListNode newHead = head.next;
        ListNode remainHead = swapPairs(newHead.next);
        newHead.next = head;
        head.next = remainHead;
        
        return newHead;
    }
}
```

#### 25. K 个一组翻转链表 | hard

##### 1）模拟法 | O（2 * n）

- **思路**：
  1. 模拟反转链表即可。
  2. 设计一个 doReverseKGroup 函数，返回一个 Info 结构，其中包括当前段反转过后的一个 head 头节点、tail 尾节点、next 下一段的头节点。
  3. 然后，在主方法中，反转 n / k 次，利用 pre 前驱节点的机制，不断地链接反转后的前一个链表和反转后的后一个链表。
  4. 最后，返回前驱节点 newHead 的下一个节点即可。
  5. 因此，看到链表的反转，应该要想到用 pre 前驱机制去反转，通过改变 pre 后面的内容，最后返回 pre.next 即可获得反转后的结果。 
- **结论**：时间，0 ms，100%，空间，41.1 mb，41.65%，时间上，由于主方法需要遍历 n / k 次，doReverseKGroup 方法中，先统计节点个数需要花费 O（k），然后反转节点需要花费 O（k），所以时间复杂度为 O（n / k * [k + k]）= O（2 * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {

    private class Info {
        ListNode head;
        ListNode tail;
        ListNode next;
        
        Info(ListNode head, ListNode tail, ListNode next) {
            this.head = head;
            this.tail = tail;
            this.next = next;
        }
    }

    public ListNode reverseKGroup(ListNode head, int k) {
        if(head == null) {
            return head;
        }

        ListNode newHead = new ListNode(), pre = newHead;

        Info info;
        while(head != null) {
            info = doReverseKGroup(head, k);
            pre.next = info.head;
            pre = info.tail;
            head = info.next;
        }
        
        return newHead.next;
    }

    // 反转k个节点, 有则反转, 无则不变
    private Info doReverseKGroup(ListNode head, int k) {
        if(head == null) {
            return new Info(null, null, null);
        }

        // 先查一次有没有k个
        int count = 1;
        ListNode tmp = head;
        while(tmp.next != null) {
            count++;
            tmp = tmp.next;
        }

        // 如果不够k个, 则直接返回
        if(count < k) {
            return new Info(head, tmp, null);
        }

        // 如果够k个, 则只反转k个
        ListNode pre = new ListNode(), next;
        tmp = head;
        while(k-- > 0) {
            next = tmp.next;
            tmp.next = pre.next;
            pre.next = tmp;
            tmp = next;
        }

        return new Info(pre.next, head, tmp);
    }
}
```

#### 61. 旋转链表 | medium

##### 1）哈希表 + 模拟 | O（2 * n）

- **思路**：
  1. 使用一张哈希表存储 pre 前一个节点，然后开始模拟向右旋转，链接新节点、取消链接旧节点、更新头尾节点即可。
  2. 其中要注意的是，k 可能会大于链表长度，产生重复的旋转，导致超时，所以 while 模拟右旋转时，需要对 k 进行 size 的取模，以减少旋转次数。
- **结论**：时间，1 ms，10.26%，空间，40.8 mb，67.83%，时间上，设置 preMap 集合花费 O（n），模拟旋转最多花费 O（n），所以整体时间复杂度为 O（2 * n），空间上，由于使用了一张 n 长的 preMap，所以额外空间复杂度为 O（n）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode rotateRight(ListNode head, int k) {
        if(head == null) {
            return null;
        }

        // pre集合
        Map<ListNode, ListNode> preMap = new HashMap<>();

        // 获取尾节点
        ListNode tail = head;
        while(tail.next != null) {
            preMap.put(tail.next, tail);
            tail = tail.next;
        }

        // 判空: 如果只有一个节点, 则无需旋转, 直接返回即可
        if(preMap.isEmpty()) {
            return head;
        }

        // 旋转
        ListNode pre;
        k %= preMap.size() + 1;
        while(k > 0) {
            pre = preMap.remove(tail);
            pre.next = null;

            tail.next = head;
            preMap.put(head, tail);

            head = tail;
            tail = pre;

            k--;
        }

        return head;
    }
}
```

##### 2）双指针 + 闭合成环 | O（2 * n）

- **思路**：先链表成为一个环，然后找规律发现，新链表的尾节点序号为 n - k （从 1 开始），所以直接把尾节点定位到那里，最后更新头尾指针，断开直接的链接，把环变为新链表即可。
- **结论**：时间，0 ms，100%，空间，41.1 mb，27.06%，时间上，由于需要遍历两次链表，所以整体时间复杂度尾 O（2 * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode rotateRight(ListNode head, int k) {
        if(head == null) {
            return null;
        }

        // 获取尾节点
        int n = 1;
        ListNode tail = head;
        while(tail.next != null) {
            tail = tail.next;
            n++;
        }

        // 对k取模：
        k %= n;
        if(k == 0) {
            return head;
        }

        // 闭合成环
        tail.next = head;

        // 通过观察发现, 新链表的尾节点序号为n-k(从1开始)
        int start = 1;
        while(start != n - k) {
            head = head.next;
            start++;
        }

        // 所以, 这里直接移动到k的新尾节点、head则是再下一个节点, 以保持n不变
        tail = head;
        head = head.next;
        tail.next = null;

        // 返回环切割后的头节点
        return head;
    }
}
```

#### 82. 删除排序链表中的重复元素 II | medium

##### 1）模拟 + 双指针 | O（n）

- **思路**：见代码注释，要注意使用一个哑节点，来删除第 0 个节点就重复的情况即可。
- **结论**：时间，0 ms，100%，空间，40 mb，10.08%，时间上，由于只需要遍历一次链表即可，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if(head == null) {
            return null;
        }

        // 设置哑节点的作用是, 防止第0个节点就是重复的节点了, 这样可以通过cur.next删除掉
        ListNode dummy = new ListNode(-1, head), cur = dummy;
        while(cur.next != null && cur.next.next != null) {
            // 相等, 则控制好cur指针的next指向, 以便删除重复节点
            if(cur.next.val == cur.next.next.val) {
                int repval = cur.next.val;

                // 如果cur.next等于重复值, 那么则跳过cur.next
                // 注意! 这里如果这里与cur.next.next的val判断, 会导致最后一个val节点删不掉的情况
                // 所以, 这里与提前存好的val值判断, 使得可以删除最后一个val节点
                while(cur.next != null && cur.next.val == repval) {
                    cur.next = cur.next.next;
                }
            }
            // 不等, 则继续移动合法指针cur
            else {
                cur = cur.next;
            }
        }
    
        return dummy.next;
    }
}
```

#### 83. 删除排序链表中的重复元素 | easy

##### 1）模拟 + 双指针 | O（n）

- **思路**：参考《删除排序链表中的重复元素 II》，不过这里不同的是，在碰到 cur.next.next 和 cur.next 不同时，至少保留一个 cur.next 节点。
- **结论**： 时间，0 ms，100%，空间，40 mb，44.88%，时间上，由于只需要遍历一次链表即可，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if(head == null) {
            return null;
        }

        // 设置哑节点的作用是, 防止第0个节点就是重复的节点了, 这样可以通过cur.next删除掉
        ListNode dummy = new ListNode(-1, head), cur = dummy;
        while(cur.next != null && cur.next.next != null) {
            // 相等, 则控制好cur指针的next指向, 以便删除重复节点
            if(cur.next.val == cur.next.next.val) {
                int repval = cur.next.val;

                // 如果cur.next等于重复值, 那么就看cur.next.next是否还等于重复值
                while(cur.next != null && cur.next.val == repval) {
                    // 是的话, 那么就跳过cur.next
                    if(cur.next.next != null && cur.next.next.val == repval) {
                        cur.next = cur.next.next;
                    } 
                    // 不是的话, 那么至少还保留一个值, 即cur.next
                    else {
                        cur = cur.next;
                    }
                }
            }
            // 不等, 则继续移动合法指针cur
            else {
                cur = cur.next;
            }
        }
    
        return dummy.next;
    }
}
```

##### 2）直接模拟 | O（n）

- **思路**：跳出解法 1 中的双指针思维，直接采用 cur 节点模拟即可。
- **结论**：时间，0 ms，100%，空间，40 mb，54.29%，时间上，由于只需要遍历一次链表即可，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode cur = head;
        while(cur != null) {
            // 如果cur与cur.next值相等, 则跳过cur.next, 直接链接cur与cur.next.next
            if(cur.next != null && cur.val == cur.next.val) {
                cur.next = cur.next.next;
            } 
            // 如果cur与cur.next值不等, 则正常遍历即可
            else {
                cur = cur.next;
            }
        }

        return head;
    }
}
```

#### 86. 分隔链表 | medium

##### 1）直接模拟 | O（n）

- **思路**：
  1. 构造一个 li、hi 链表，分别存放低、高节点，比 x 小的存放到 li 中，大于等于 x 的存放到 hi 中，最后返回 li + hi 链表即可。
  2. 其中要注意的是，重新调整 li、hi 节点过程中，需要每移动一个节点，记得清空那个节点的 next 指针。
- **结论**：时间，0 ms，100%，空间，41.3 mb，54.29%，时间上，由于只需要遍历一次链表即可，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode partition(ListNode head, int x) {
        ListNode li = new ListNode(Integer.MIN_VALUE), lih = li;
        ListNode hi = new ListNode(Integer.MAX_VALUE), hih = hi;

        ListNode cur = head, next;
        while(cur != null) {
            next = cur.next;
            cur.next = null;

            if(cur.val < x) {
                li.next = cur;
                li = li.next;
            } else {
                hi.next = cur;
                hi = hi.next;
            }

            cur = next;
        }

        li.next = hih.next;
        return lih.next;
    }
}
```

#### 92. 反转链表 II | medium

##### 1）模拟 + 切割 + 递归 | O（2 * n）

- **思路**：参考《206. 反转链表》- 模拟 + 递归的思路，可以先找出要反转的链表，然后调用反转方法，反转后再重新拼接回来即可。
- **结论**：时间，0 ms，100%，空间，38.9 mb，81.90%，时间上，由于链表最多需要遍历 2 次，所以时间复杂度为 O（2 * n），空间上，递归的最大栈深度为 n，所以额外空间复杂度为 O（n）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
 class Info {
    ListNode head;
    ListNode tail;
    
    Info(ListNode head, ListNode tail) {
        this.head = head;
        this.tail = tail;
    }
}

class Solution {
    public ListNode reverseBetween(ListNode head, int left, int right) {
        // 找出left的前驱
        ListNode dummy = new ListNode(-1, head), pre = dummy;
        for(int i = 0; i < left - 1; i++) {
            pre = pre.next;
        }

        // 找出right的后继
        ListNode list_tail = pre.next;
        for(int i = left; i < right; i++) {
            list_tail = list_tail.next;   
        }

        // 切割出[left, right]的链表
        ListNode list_head = pre.next, succ = list_tail.next;
        pre.next = null;
        list_tail.next = null;

        // 反转[left, right]的链表
        Info info = doReverseList(list_head);

        // 拼接pre+新头、新尾+succ
        pre.next = info.head;
        if(info.tail != null) {
            info.tail.next = succ;
        }

        return dummy.next;
    }

    private Info doReverseList(ListNode head) {
        // 最后一个节点, 则返回当前节点
        if(head.next == null) {
            return new Info(head, head);
        }

        // 先反转后面的
        Info info = doReverseList(head.next);
        
        // 再反转当前的
        info.tail.next = head;
        head.next = null;
        
        // tail链接当前head节点
        info.tail = head;

        // 返回info信息
        return info;  
    }
}
```

##### 2）模拟 + 切割 + 迭代 | O（2 * n）

- **思路**：参考《206. 反转链表》- 模拟 + 迭代的思路，可以先找出要反转的链表，然后调用反转方法，反转后再重新拼接回来即可。
- **结论**：时间，0 ms，100%，空间，38.7 mb，99.00%，时间上，由于链表最多需要遍历 2 次，所以时间复杂度为 O（2 * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode reverseBetween(ListNode head, int left, int right) {
        // 找出left的前驱
        ListNode dummy = new ListNode(-1, head), pre = dummy;
        for(int i = 0; i < left - 1; i++) {
            pre = pre.next;
        }

        // 找出right的后继
        ListNode list_tail = pre.next;
        for(int i = left; i < right; i++) {
            list_tail = list_tail.next;   
        }

        // 切割出[left, right]的链表
        ListNode list_head = pre.next, succ = list_tail.next;
        pre.next = null;
        list_tail.next = null;

        // 反转[left, right]的链表
        ListNode newHead = reverseList(list_head);

        // 拼接pre+新头、新尾+succ
        pre.next = newHead;
        list_head.next = succ;// 这里list_head反转后, 作为了新尾

        return dummy.next;
    }

    private ListNode reverseList(ListNode head) {
        ListNode pre = null, next;
        while(head != null) {
            // 反转
            next = head.next;
            head.next = pre;
            
            // 继续遍历
            pre = head;
            head = next;
        }

        return pre;
    }
}
```

##### 3）模拟 + 节点冒泡 | O（n）

- **思路**：类似于冒泡的方式，不断地把后续的节点，挨个挨个地冒泡到 pre 的后面，当 cur 从 left 位置沉底到 right 位置，则反转过程结束，此时 [left，right] 之间的链表已经完成反转。

  ![1659076307284](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1659076307284.png)

- **结论**：时间，0 ms，100%，空间，39.1 mb，62.52%，时间上，由于链表只需要遍历 1 次，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode reverseBetween(ListNode head, int left, int right) {
        // 找出left的前驱
        ListNode dummy = new ListNode(-1, head), pre = dummy;
        for(int i = 0; i < left - 1; i++) {
            pre = pre.next;
        }

        // 找出right的后继
        ListNode cur = pre.next, next;
        for(int i = left; i < right; i++) {
            // 备份next
            next = cur.next;

            // 先脱钩next
            cur.next = next.next;

            // 再在pre->?->pre的后继之间, 插入next
            next.next = pre.next;
            pre.next = next;
        }

        return dummy.next;
    }
}
```

#### 141. 环形链表 | easy

##### 1）快慢指针法 | O（n）

- **思路**：
  1. 判断链表中有没有环，核心思路就是会不会从末尾走回前面，这可以用前后指针来实现，即快指针每次走 2 步，慢指针每次走 1 步，如果链表中有环，那么快指针必然会走到慢指针后面，而由于快指针的步长大，时间久了也必然会从后面追上慢指针，也就是相遇时则返回 true，否则返回 false 即可。 
  2. 不过有个细节要注意的是，由于是判断指针地址是否相等，作为返回 true 的条件，所以为了避免一开始指针地址相等，需要先在 while 外面，先手动对快、慢指针提前走一下，来工整下代码。
- **结论**：时间，0ms，100%，空间，39.5mb，57.42%，效率非常高，就这样吧~

```java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public boolean hasCycle(ListNode head) {
        if(head == null) {
            return false;
        }
        if(head.next == null) {
            return false;
        }

        ListNode h1 = head.next, h2 = head.next.next;
        while(h1 != null && h2 != null) {
            if(h1 == h2) {
                return true;
            }
            if(h2.next == null) {
                return false;
            }

            h1 = h1.next;
            h2 = h2.next.next;
        }

        return false;
    }
}
```

#### 142. 环形链表 II | medium

##### 1）快慢指针法 | O（n+a）

- **思路**：

  1. 设快慢指针在环中相遇的点为 b，此时快指针已在环中走过了 n 圈，如果 b 在环中继续往后走 c 才能回到入环点，此时入环点与原点距离为 a。
  2. 由于快指针的步长是慢指针的 2 倍，也就是在相同时间里，快指针所走的距离为慢指针的 2 倍，即 a + n * （b + c） + b = 2 * （a + b），可推导出 a = （n - 1）*（b + c）+ c，如果某个点在环上 b 的位置出发, 那么它走过的 n-1 圈的环再回到入环点的距离, 刚好等于后来一起从原点出发到入环点的距离。

  ![1643002320112](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1643002320112.png)

- **结论**：

  1. 时间，1ms，24.37%，空间，38.5mb，46.66%，由于在换上相遇了以后，还需要重新走 a 步，所以时间复杂度为 O（n+a），空间上只使用了有限几个变量，所以额外空间复杂度为 O（1）。
  2. 其中要注意的是，由于这里是快指针提前走了 2 个步长，慢指针也提前走了 1 个步长，所以推导出的公式中的 n 最小值不是 1 而是 0，此时公式应该为 a = c，即从原点走到入环点的距离等于从环中相遇点走到入环点的距离，也就是入环点到原点的距离等于入环点到相遇点的距离，也就是入环点就在原点上，所以 h2 在第一次相遇后移动回原点时，还需要**马上判断一次**是否已经第二次在原点相遇了，如果相遇则直接返回原点即可，否则就还要走 a 的距离才能到入环点。

```java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    //  a + n(b+c) + b = 2(a+b)
    //  a + (n+1)b + nc = 2a + 2b
    // -a + (n-1)b + nc = 0
    //  a = (n-1)b + nc = (n-1)(b+c) + c
    // 即如果某个点在环上b的位置出发, 那么它走过的n-1圈的环再回到入环点的距离, 刚好等于后来一起从原点出发到入环点的距离
    public ListNode detectCycle(ListNode head) {
        if(head == null) {
            return null;
        }
        if(head.next == null) {
            return null;
        }

        boolean hasMeet = false;
        ListNode h1 = head.next, h2 = head.next.next;
        while(h1 != null && h2 != null) {
            // 未相遇时, h2 一次走两步
            if(!hasMeet && h2.next == null) {
                return null;
            }
            // 第一次相遇后, h2回到原点, 并更改步长与h1一起走
            if(!hasMeet && h1 == h2) {
                h2 = head;
                hasMeet = true;
            }
            // 再次相遇, 则相遇在入环掉
            if(hasMeet && h1 == h2) {
                return h1;
            }

            h1 = h1.next;
            h2 = !hasMeet? h2.next.next : h2.next;
        }

        return null;
    }
}
```

#### 143. 重排链表 | medium

##### 1）寻找链表中点 + 反转链表 + 合并链表 | O（3 * n）

- **思路**：见代码注释。
- **结论**：时间，1 ms，99.87%，空间，44 mb，67.67%，时间上，寻找链表中点花费 O（n），反转链表花费 O（n），合并链表花费 O（n），所以整体时间复杂度为 O（3 * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public void reorderList(ListNode head) {
        // 找到链表中点: mid必有值
        ListNode mid = findMidListNode(head);

        // 清空mid之后的指针, 脱钩链表mid之后的链表
        ListNode next = mid.next;
        mid.next = null;

        // 反转中点后的链表：reversedHead可能为null, 说明只有一个节点, 此时直接返回即可
        ListNode reversedHead = reverseList(next);
        if(reversedHead == null) {
            return;
        }

        // 合并链表, 反转后的链表隔一个位置插入到原链表
        mergeList(head, reversedHead);
    }

    // 合并链表, hi隔一个位置插入lo
    private void mergeList(ListNode lo, ListNode hi) {
        ListNode loNext, hiNext;
        while(lo != null && hi != null) {
            // 备份next指针
            loNext = lo.next;
            hiNext = hi.next;

            // 链接lo、hi
            lo.next = hi;
            hi.next = loNext;

            // 继续移动
            lo = loNext;
            hi = hiNext;
        }
    }

    // 反转链表
    private ListNode reverseList(ListNode head) {
        if(head == null) {
            return null;
        }

        ListNode pre = null, next;
        while(head != null) {
            // 反转
            next = head.next;
            head.next = pre;

            // 继续遍历
            pre = head;
            head = next;
        }
        return pre;
    }

    // 找到链表中点
    private ListNode findMidListNode(ListNode head) {
        ListNode slow = head, fast = head.next;
        while(fast != null && fast.next != null) {
            fast = fast.next.next;
            slow = slow.next;
        }
        return slow;
    }
}
```

#### 146. LRU 缓存 | medium

##### 1）暴力解法 | O（1）

- **思路**：利用 LinkedHashMap 已经实现了的 LRU 结构（访问后移回链表末尾，新增后判断是否超出了最大容量，如果超出则删除最久没被访问的元素），其中要注意的细节有：
  1. 初始化时，要保证初始容量不会发生扩容。
  2. 初始化时，要把 accessOrder 改为 true，代表访问时需要把最近访问的 Entry 放回链表末尾。
  3. 初始化时，要重写 `removeEldestEntry(Map.Entry<K,V>)` 方法， 使新增元素之后能够回调删除方法。
  4. 获取元素时，要判断是否已经被删除了，如果已经被删除了，则返回 -1，避免空指针异常。
- **结论**：时间，41ms，98.99%，空间，108.4mb，47.86%，由于使用的是 JDK 实现的 LinkedHashMap，性能自然是十分高的，但面试要求的应该不只是这点，还需要自己把底层也实现出来~

```java
class LRUCache {

    private int maxSize;
    private Map<Integer, Integer> map;

    public LRUCache(int capacity) {
        this.maxSize = capacity;
        this.map = new LinkedHashMap<Integer, Integer>(
           // 相除是为了保证顶格初始容量, 使其对于maxSize不会再扩容了 => 取大于等于临界值的最小值
            (int) Math.ceil(capacity / 0.75f), 0.75f, true) {
            protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
                return map.size() > maxSize;
            }
        };
    }
    
    public int get(int key) {
        Integer res = map.get(key);
        return res != null? res : -1;
    }
    
    public void put(int key, int value) {
        map.put(key, value);
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```

##### 2）双向链表法 | O（1）

- **思路**：通过双向链表+哈希表模拟实现 LinkedHashMap，即分为 4 种情况：
  1. 新增元素：新增后，节点需要移动至链表末尾，代表最近被访问过，共有 2 种情况，分为新增的是第一个节点，以及新增的不是第一个节点。
  2. 替换元素值：替换节点值后，节点需要移动至链表末尾，代表最近被访问过，共有 3 种情况，分为替换的是头节点、替换的是尾节点，以及替换的是中间节点。
  3. 获取元素值：获取节点值后，节点需要移动至链表末尾，代表最近被访问过，共有 2 种情况，分为节点不存在，以及节点存在。
  4. 删除元素：新增节点后，如果当前哈希表的大小 > 最大大小，那么就要触发 LRU 淘汰机制，淘汰最久没被访问过的元素，即把链表表头的元素脱离链表。
- **结论**：时间，44 ms，60.91%，空间，108.9 mb，49.85%，效率自然就没 JDK 实现的 LinkedHashMap 的高，面试应该到这里就可以了吧~

```java
class LRUCache {

    private class Element {
        Integer key;
        Integer val;
        Element pre;
        Element next;

        Element(Integer key, Integer val) {
            this.key = key;
            this.val = val;
        }
    }

    private Map<Integer, Element> elementMap;
    private int maxSize;

    private Element head;
    private Element tail;

    public LRUCache(int capacity) {
        if(capacity < 0) {
            throw new RuntimeException("容量不能为0!");
        }

        this.maxSize = capacity;
        this.elementMap = new HashMap<>();
    }
    
    public int get(int key) {
        Element element = elementMap.get(key);
        if(element != null) {
            link2Last(element);
            return element.val;
        } else {
            return -1;
        }
    }
    
    public void put(int key, int val) {
        Element element = elementMap.get(key);

        if(element != null) {
            element.val = val;
            link2Last(element);
        } else {
            if(elementMap.size() == this.maxSize) {
                Element remove = unlinkFirst();
                elementMap.remove(remove.key);
            }

            element = new Element(key, val);
            elementMap.put(key, element);
            add2Last(element);
        }
    }

   private Element unlinkFirst() {
        Element remove = head;

        // 链表只有一个节点
        if(head == tail) {
            head = tail = null;
        } 
        // 链表有多个节点
        else {
            Element next = head.next;
            head.next = null;

            next.pre = null;
            head = next;
            
            // head为最后一个节点
            if(head.next == null) {
                tail = head;
            }
        }

        return remove;
    }

    private void add2Last(Element element) {
        // 链表为空
        if(head == null) {
            head = tail = element;
        } 
        // 链表不为空
        else {
            element.pre = tail;
            tail.next = element;
            tail = element;
        }
    }

    private void link2Last(Element element) {
        // 节点为尾节点
        if(element == tail) {
            return;
        }
        // 节点为头节点
        if(element == head) {
            // 链表只有一个节点
            if(head == tail) {
                return;
            } 
            // 链表有多个节点
            else {
                head = element.next;
                head.pre = null;
                element.next = null;

                element.pre = tail;
                tail.next = element;
                tail = element;
                return;
            }
        }
        
        // 节点为中间节点
        element.pre.next = element.next;
        element.next.pre = element.pre;

        element.pre = tail;
        tail.next = element;
        
        element.next = null;
        tail = element;
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```

#### 148. 排序链表 | medium

##### 1）暴力解法 | O（n * logn + n）

- **思路**：先把元素取出，然后进行排序，然后再重组链表即可。
- **结论**：时间，1295ms，5.77%，空间，48.8mb，5.02%，时间上，排序花费 O（n * logn），重新组装链表花费 O（n），所以时间复杂度为 O（n * logn + n），空间上，快速排序的递归栈深度为 logn，所以额外空间复杂度为 O（logn）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {

    public ListNode sortList(ListNode head) {
        if(head == null) {
            return null;
        }

        List<ListNode> list = new LinkedList<>();
        while(head != null) {
            list.add(head);
            head = head.next;
        }
        
        // 排序
        list.sort((o1, o2) -> o1.val - o2.val);

        // 重组
        ListNode cur = head = list.get(0), next;
        head.next = null;
        for(int i = 1; i < list.size(); i++) {
            next = list.get(i);
            next.next = null;
            
            cur.next = next;
            cur = cur.next;
        }

        return head;
    }
}
```

##### 2）归并排序（自顶向下递归） | O（n * logn）

- **思路**：
  1. 利用归并排序的思路，对左半边的链表排好序，对右半边的链表排好序，然后调用《合并有序链表》中的思路，合并左右两边有序的链表，即可得到整个有序的链表，而对左、右半边的链表又可以以同样的方式递归进行排序。
  2. 其中，寻找链表中点的方法，可以利用快慢指针来实现，本题实现的慢指针等于 mid.next。
  3. 另外，还有要注意的是，由于是链表没有左右界限做分割，容易导致合并的时候造成把后面的链表也合并上来了，因此，**最关键的一步**是在对左半边链表进行排序时，需要对左半边的右界进行置空，以达到分割左右半边链表的目的。
- **结论**：时间，5ms，98.99%，空间，49.2 mb，5.02%，时间上，根据 Master 公式可得到时间复杂度为 O（n * logn），空间上，递归深度最大为  logn，所以额外空间复杂度为 O（logn），因此还需要优化掉递归。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {

    public ListNode sortList(ListNode head) {
        return sortList(head, null);
    }

    private ListNode sortList(ListNode head, ListNode tail) {
        if(head == null) {
            return head;
        }
        // mid.next = slow
        if(head.next == tail) {
            head.next = null;// 最关键的一步, 用于分割前后两个链表
            return head;
        }
        
        ListNode mid = findMidNode(head, tail);
        ListNode list1 = sortList(head, mid);
        ListNode list2 = sortList(mid, tail);
        return merge(list1, list2);
    }

    // 快慢指针法寻找链表中点: mid.next = slow
    private ListNode findMidNode(ListNode head, ListNode tail) {
        ListNode slow = head, fast = head;
        while(fast != tail) {
            slow = slow.next;
            fast = fast.next;
            if(fast != tail) {
                fast = fast.next;
            }
        }
        return slow;
    }

    // 合并两个有序链表
    private ListNode merge(ListNode list1, ListNode list2) {
        ListNode head = new ListNode(), pre = head;
        while(list1 != null && list2 != null) {
            if(list1.val < list2.val) {
                pre.next = list1;
                list1 = list1.next;
                pre = pre.next;
            } else {
                pre.next = list2;
                list2 = list2.next;
                pre = pre.next;
            }
        }
        pre.next = list1 != null? list1 : list2;
        return head.next;
    }
}
```

##### 3）归并排序（自底向上迭代）| O（n * logn）

- **思路**：
  1. 还是沿用归并排序的思路，不过这次是迭代从下往上，模拟归并排序的实现，即首先归并长度为 1 的有序子链表，再归并长度为 2 的有序子链表...，直到归并到长度为 2^n 超过链表长度的有序子链表，最终得到合并后的有序链表。
  2. 其中要注意的是，子链表的分割、子链表的合并，以及合并子链表的临时保存，如何用 O（1）的方式实现。
- **结论**：时间，9ms，37.98%，空间，48.9mb，5.02%，明明是在原链表上用迭代法实现，但空间和时间最终的效果并不理想，不管了，思路对就行了，其他就是编码问题了~

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {

    public ListNode sortList(ListNode head) {
        if(head == null) {
            return head;
        }

        int listSize = getLen(head);
        ListNode list1 = new ListNode(), list2 = new ListNode();
        ListNode preHead = new ListNode(0, head), end;
        for(int len = 1; len < listSize; len <<= 1) {
            // 获取最新的head, 以及备份头节点
            end = preHead.next = head;

            // 开始归并头节点链表
            head = preHead;
            while(end != null) {
                // 获取长度为len的一个有序左边
                end = subListNode(end, len, list1);

                // 获取长度为len的一个有序右边
                end = subListNode(end, len, list2);

                // 合并, 一个有序左 + 一个有序右 = 一个有序链表
                head.next = merge(list1.next, list2.next);

                // 继续合并剩下的子链表
                while(head.next != null) {
                    head = head.next;
                }
            }

            // 还原头节点
            head = preHead.next;
        }

        return head;
    }

    // 获取链表的长度
    private int getLen(ListNode head) {
        int cur = 0;
        while(head != null) {
            head = head.next;
            cur++;
        }
        return cur;
    }

    // 寻找len长度的子链表
    private ListNode subListNode(ListNode head, int len, ListNode preList) {
        int cur = 0;
        ListNode tail = head, pre = tail;
        while(tail != null && cur < len) {
            pre = tail;
            tail = tail.next;
            cur++;
        }
        
        // 如果子链表的尾节点已经到末尾了, 则无需分割子链表了
        if(tail == null) {
            preList.next = head;
            return tail;
        } 
        // 否则, 根据pre分割子链表
        else {
            pre.next = null;
            preList.next = head;
            return tail;
        }
    }

    // 合并两个有序链表
    private ListNode merge(ListNode list1, ListNode list2) {
        ListNode head = new ListNode(), pre = head;
        while(list1 != null && list2 != null) {
            if(list1.val < list2.val) {
                pre.next = list1;
                list1 = list1.next;
                pre = pre.next;
            } else {
                pre.next = list2;
                list2 = list2.next;
                pre = pre.next;
            }
        }
        pre.next = list1 != null? list1 : list2;
        return head.next;
    }
}
```

#### 160. 相交链表 | easy

##### 1）暴力解法 | O（m+n）

- **思路**：由于求的是两个链表相交的节点，也就是相同地址的节点，因此可以先遍历其中一个链表，把路过的节点都加入哈希表中，接着再遍历第二个链表，如果第一次发现哈希表中存在这个节点，返回那个节点就是答案了。
- **结论**：时间，7ms，25.11%，空间，42.7mb，5.02%，时间上，由于使用了哈希表，所以即使都是 O（m+n），但其他方法可能常数项较低，所以就显得当前方法效率慢点，而空间上，由于使用了一个哈希表，所以额外空间复杂度为 O（m）或者 O（n）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if(headA == null || headB == null) {
            return null;
        }

        Map<ListNode, ListNode> map = new HashMap<>();
        while(headA != null) {
            map.put(headA, headA);
            headA = headA.next;
        }

        while(headB != null) {
            if(map.containsKey(headB)) {
                return headB;
            } else {
                headB = headB.next;
            }
        }

        return null;
    }
}
```

##### 2）双指针法 | O（m+n+a）

- **思路**：先分别统计两个链表的长度，然后得知哪个是长链表，哪个是短链表后，再让长链表指针从头开始走长度差那么多步数，长者走完差值步后再与短链表指针一起走，走到第一个相同的节点则为第一个相交的节点（因此相交后面的节点地址必定相同）。
- **结论**：时间，1ms，99.96%，空间，41.5mb，22.12%，时间上，由于需要先遍历完两个链表，再走差值步以及到第一个相交的节点，所以时间复杂度为 O（m+n+a），a 代表相交前要走的步数。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if(headA == null || headB == null) {
            return null;
        }

        int n = 0;
        ListNode node1 = headA, node2 = headB;
        while(node1.next != null) {
            n++;
            node1 = node1.next;
        }
        while(node2.next != null) {
            n--;
            node2 = node2.next;
        }
        
        // n = headA - headB
        node1 = n > 0? headA : headB;
        node2 = node1 == headA? headB : headA;

        // 长者node1走差值步
        n = Math.abs(n);
        while(n > 0) {
            n--;
            node1 = node1.next;
        }

        // 长者走完差值步再同时起步
        while(node1 != node2) {
            node1 = node1.next;
            node2 = node2.next;
        }

        return node1;
    }
}
```

#### 206. 反转链表 | easy

##### 1）模拟 + 迭代 | O（n）

- **思路**：思路很简单，直接模拟反转即可。
- **结论**：时间，0ms，100%，空间，38.4mb，12.95，时间上，由于只遍历了一次链表，所以时间复杂度为 O（n），而空间上，由于只用了几个有限变量，所以额外空间复杂度为 O（1）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode pre = null, next;
        while(head != null) {
            // 反转
            next = head.next;
            head.next = pre;
            
            // 继续遍历
            pre = head;
            head = next;
        }

        return pre;
    }
}
```

##### 2）模拟 + 递归 | O（n）

- **思路**：见代码注释。
- **结论**：时间，0ms，100%，空间，41.2mb，53.42%，时间上，由于链表只遍历了一次，所以时间复杂度为 O（n），空间上，递归的最大栈深度为 n，所以额外空间复杂度为 O（n）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Info {
    ListNode head;
    ListNode tail;
    
    Info(ListNode head, ListNode tail) {
        this.head = head;
        this.tail = tail;
    }
}

class Solution {
    public ListNode reverseList(ListNode head) {
        if(head == null) {
            return null;
        }
        return doReverseList(head).head;
    }

    private Info doReverseList(ListNode head) {
        // 最后一个节点, 则返回当前节点
        if(head.next == null) {
            return new Info(head, head);
        }

        // 先反转后面的
        Info info = doReverseList(head.next);
        
        // 再反转当前的
        info.tail.next = head;
        head.next = null;
        
        // tail链接当前head节点
        info.tail = head;

        // 返回info信息
        return info;  
    }
}
```

#### 234. 回文链表 | easy

##### 1）暴力解法 | O（n）

- **思路**：先遍历链表，把节点收集到 ArrayList 中，然后再从头遍历判断链表是否为回文。
- **结论**：时间，5ms，66.85%，空间，53.3mb，18.89%，时间上，需要遍历 2 次，不过第二次只需要判断一半即可，而空间上，由于使用了一个数组，所以空间复杂度为 O（n），可以用双指针优化成 O（1）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public boolean isPalindrome(ListNode head) {
        if(head == null) {
            return true;
        }

        int len = 0;
        ListNode node = head;
        List<ListNode> list = new ArrayList<>();
        while(node != null) {
            len++;
            list.add(node);
            node = node.next;
        }

        int index = 0;
        node = head;
        ListNode node2;
        while(node != null && index <= (len-1 >>> 1)) {
            node2 = list.get(len-1-index);
            if(node2.val != node.val) {
                return false;
            }

            index++;
            node = node.next;
        }

        return true;
    }
}
```

##### 2）前后指针法 | O（n）

- **思路**：先遍历一次算出链表长度，然后前指针先走到链表中点的 **next 节点**，这样可以翻转前指针往后的链表，最后后指针从头开始遍历，前指针从翻转链表的头节点开始遍历，如果遍历过程中有节点不等，则返回 false，否则返回 true。
- **结论**：时间，5ms，66.85%，空间，57.8mb，5.01%，时间上，由于要遍历 4 次，1 次整个链表，3 次半个链表，所以时间复杂度为 O（2.5n），空间上，由于只使用了几个变量，所以额外空间复杂度为 O（1）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public boolean isPalindrome(ListNode head) {
        if(head == null) {
            return true;
        }

        int len = 0;
        ListNode node1 = head;
        while(node1 != null) {
            len++;
            node1 = node1.next;
        }

        int index = 0;
        ListNode node2 = head;
        while(node2 != null && index <= (len-1 >>> 1)) {
            node2 = node2.next;
            index++;
        }

        // 翻转后半段链表
        node2 = reverseList(node2);
        
        // 如果为偶数个节点
        node1 = head;
        while(node1 != null && node2 != null) {
            if(node1.val != node2.val) {
                return false;
            }

            node1 = node1.next;
            node2 = node2.next;
        }
        
        return true;
    }

    private ListNode reverseList(ListNode head) {
        if(head == null) {
            return null;
        }
        if(head.next == null) {
            return head;
        }

        ListNode next = head.next, pre = new ListNode(head.val);
        while(next != null) {
            head = next;
            next = next.next;
            head.next = pre;// 翻转
            pre = head;// 继续移动翻转后的链表
        }

        return head;
    }
}
```

### 1.2. 数据结构 - 二叉树

#### 模板代码

##### 1）Morris 遍历

```java
package com.jsonyao.interview.test.morris;

import java.util.ArrayList;
import java.util.List;

/**
 * 单纯的Morris遍历：
 * 初始时, cur = root
 *   1）没有左孩子, 则cur继续往右遍历
 *   2）有左孩子, 但前驱->null, 说明是第一次来到cur, 则前驱->cur, cur继续往左遍历
 *   3）有左孩子, 且前驱->cur, 说明是第二次回到cur, 则前驱->null, cur继续往右遍历
 * 周而复始, 直到结束...
 *
 * @author yaocs2
 * @since 2022-08-07
 */
public class MorrisTraversal {

    public static void main(String[] args) {
        TreeNode root = new TreeNode(1);
        root.left = new TreeNode(2);
        root.left.left = new TreeNode(4);
        root.left.right = new TreeNode(5);
        root.right = new TreeNode(3);
        root.right.left = new TreeNode(6);
        root.right.right = new TreeNode(7);

        // 单纯的Morris遍历
        List<Integer> res = new ArrayList<>();
        new MorrisTraversal().morrisTraversal(root, res);

        // 打印遍历结果
        printList(res);
    }

    /**
     * 打印遍历结果
     */
    private static void printList(List<Integer> list) {
        for (Integer i : list) {
            System.out.print(i + " ");
        }
    }

    private void morrisTraversal(TreeNode root, List<Integer> res) {
        // 初始时, cur来到root位置
        TreeNode cur = root, mostRight = null;

        // cur为null, 则停止, 说明遍历到了树的最右最底的叶子节点了
        while (cur != null) {
            // 单纯的Morris遍历
            res.add(cur.val);

            // 如果cur没有左孩子, 则cur向右移动
            mostRight = cur.left;
            if (mostRight == null) {
                cur = cur.right;
                continue;
            }

            // 如果cur有右孩子, 则找到左子树中最右的节点, 这里不等于mostRight.right != cur, 是为了避免发生第二次找不到正确前驱的错误
            while (mostRight.right != null && mostRight.right != cur) {
                // 一直向右移动
                mostRight = mostRight.right;
            }

            // 找到左子树最右的节点后, 则判断：
            // 如果mostRight右指针为null, 说明是第一次来到cur节点, 则让其指向cur, 然后cur向左移动
            if (mostRight.right == null) {
                mostRight.right = cur;
                cur = cur.left;
            }
            // 如果mostRight右指针指向cur, 说明是第二次来到cur节点, 则清空其指向, 以恢复本来的二叉树, 然后cur向右移动(此后cur再也不回不到这里及其左子树了)
            else {
                mostRight.right = null;
                cur = cur.right;
            }
        }
    }
}
```

#### 94. 二叉树的中序遍历 | 深度优先遍历 | easy

##### 1）递归法 | O（n）

- **思路**：递归遍历，左 -> 中 -> 右，因此在回到中的时候把节点的值加入集合中即可。
- **结论**：时间，0ms，100%，36.3mb，95.76%，时间上，由于每个节点只需要遍历一次，所以时间复杂度为 O（n），空间上，由于递归栈深度最大为 log n，所以额外空间复杂度为 O（log n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> res = new LinkedList<>();
        f(root, res);
        return res;
    }

    private void f(TreeNode root, List<Integer> res) {
        if(root == null) {
            return;
        }

        f(root.left, res);
        res.add(root.val);
        f(root.right, res);
    }
}
```

##### 2）迭代法 | O（n）

- **思路**：
  1. 中序遍历的迭代法实现也是很简单的，模拟系统递归压栈的方式即可。
  2. 如果是递归实现中序遍历的话，res.add 是放在了左和右的中间，此时系统会一直找左找到空为止，那么迭代法也可以模拟这个操作，即一直找左，把路上的根节点都先入栈。
  3. 然后回到递归的中序遍历上，如果系统找左找到了空，那么就会返回，此时返回就回到了中间那步 res.add 操作，此时迭代法也可以模拟这操作，即栈弹出一个然后调用 res.add。
  4. 再回到递归的中序遍历上，接着系统收集完值后，就会把找右节点，然后又会一直找它的左节点一路为空，此时用迭代法来模拟就是，把右节点作为根节点，然后重复 2 的操作即可。
- **结论**：
  - 时间，0ms，100%，36.3%，96.77%，先序和中序是直接模拟系统栈来实现的，而后序如果用系统栈方式来模拟感觉比较麻烦，就用了先序颠倒再逆序的方式来实现简单一点。
  - 时间上，由于每个节点只需要遍历一次，所以时间复杂度为 O（n）。
  - 空间上，由于 stack 栈在添加了 左边界全部的节点 后达到最长，为 log n，所以额外空间复杂度为 O（log n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        LinkedList<TreeNode> stack = new LinkedList<>();
        while(root != null) {
            stack.push(root);
            root = root.left;
        }

        List<Integer> res = new LinkedList<>();
        while(!stack.isEmpty()) {
            root = stack.pop();
            if(root != null) {
                res.add(root.val);
                if(root.right != null) {
                    root = root.right;
                    while(root != null) {
                        stack.push(root);
                        root = root.left;
                    }
                }
            }
        }

        return res;
    }
}
```

##### 3）Morris 中序遍历 | O（3 * n）

- **思路**：见代码注释。
- **结论**：时间，0 ms，100%，空间，39.7 mb，39.49%，时间上，由于每个节点至少遍历一次，花费 O（n），有左孩子的根节点会多访问一次，最多花费 O（n），查找节点的前驱最多也花费 O（n），所以整体时间复杂度为 O（3 * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public void recoverTree(TreeNode root) {
        // 0: 前一个交换点, 1：后一个交换点
        TreeNode[] swapNodes = new TreeNode[2];
        
        // Morris中序遍历
        morrisInTraversal(root, swapNodes);

        // 交换from和to节点的值
        swapNodeVal(swapNodes[0], swapNodes[1]);
    }

    // 交换from和to节点的值
    private void swapNodeVal(TreeNode from, TreeNode to) {
        int tmp = to.val;
        to.val = from.val;
        from.val = tmp;
    }

    // Morris中序遍历：只出现一次的节点, 直接收集; 出现两次的节点, 第一次不收集, 第二次才收集
    private void morrisInTraversal(TreeNode root, TreeNode[] swapNodes) {
        // 初始时, cur来到root位置
        TreeNode cur = root, mostRight = null, pre = null;

        // cur为null, 则停止, 说明遍历到了树的最右最底的叶子节点了
        while (cur != null) {
            // 如果cur没有左孩子, 则cur向右移动
            mostRight = cur.left;
            if (mostRight == null) {
                // 改写Morris中序遍历 => 设置前后交换点
                setSwapNodes(swapNodes, cur, pre);
                // 更新前驱
                pre = cur;

                cur = cur.right;
                continue;
            }

            // 如果cur有右孩子, 则找到左子树中最右的节点, 这里不等于mostRight.right != cur, 是为了避免发生第二次找不到正确前驱的错误
            while (mostRight.right != null && mostRight.right != cur) {
                // 一直向右移动
                mostRight = mostRight.right;
            }

            // 找到左子树最右的节点后, 则判断：
            // 如果mostRight右指针为null, 说明是第一次来到cur节点, 则让其指向cur, 然后cur向左移动
            if (mostRight.right == null) {
                mostRight.right = cur;
                cur = cur.left;
            }
            // 如果mostRight右指针指向cur, 说明是第二次来到cur节点, 则清空其指向, 以恢复本来的二叉树, 然后cur向右移动(此后cur再也不回不到这里及其左子树了)
            else {
                mostRight.right = null;

                // 改写Morris中序遍历 => 设置前后交换点
                setSwapNodes(swapNodes, cur, pre);
                // 更新前驱
                pre = cur;

                cur = cur.right;
            }
        }
    }

    // 设置前后交换点
    private void setSwapNodes(TreeNode[] swapNodes, TreeNode cur, TreeNode pre) {
        if(pre != null && pre.val > cur.val) {
            // 第一个问题对, 取前一个节点
            if(swapNodes[0] == null) {
                swapNodes[0] = pre;
            }

            // 第一个或者第二个问题对, 取后一个节点
            swapNodes[1] = cur;
        }
    }
}
```

#### 144. 二叉树的先序遍历 | easy

##### 1）递归法 | O（n）

- **思路**：递归遍历，中 -> 左 -> 右，因此在一开始来到中时，就把节点的值加入集合中即可。
- **结论**：时间，0ms，100%，36.4mb，92.56%，时间上，由于每个节点只需要遍历一次，所以时间复杂度为 O（n），空间上，由于递归栈深度最大为 log n，所以额外空间复杂度为 O（log n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> res = new LinkedList<>();
        f(root, res);
        return res;
    }
    
    private void f(TreeNode root, List<Integer> res) {
        if(root == null) {
            return;
        }

        res.add(root.val);
        f(root.left, res);
        f(root.right, res);
    }
}
```

##### 2）迭代法 | O（n）

- **思路**：迭代法中，先序遍历的实现最简单，因为当前使用的 while 循环可以认为是“中”，然后模拟系统栈把右、左一次压栈即可（可以认为是方法倒过来压栈）。
- **结论**：时间，0ms，100%，36.4mb，93.30%，注意添加和使用前都做个判空，防止空指针和节省空间的使用，时间上，由于每个节点只需要遍历一次，所以时间复杂度为 O（n），空间上，由于 stack 栈在添加了 最后一行全部的节点 后达到最长，为 2^( log n )，所以额外空间复杂度为 O（ 2^[ log n ] ）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        LinkedList<TreeNode> stack = new LinkedList<>();
        if(root != null) {
            stack.push(root);
        }
        
        List<Integer> res = new LinkedList<>();
        while(!stack.isEmpty()) {
            root = stack.pop();
            if(root != null) {
                res.add(root.val);
                if(root.right != null) {
                    stack.push(root.right);
                }
                if(root.left != null) {
                    stack.push(root.left);
                }
            }
        }

        return res;
    }
}
```

##### 3）Morris 先序遍历 | O（3 * n）

- **思路**：见代码注释。
- **结论**：时间，0 ms，100%，空间，39.5 mb，71.45%，时间上，由于每个节点至少遍历一次，花费 O（n），有左孩子的根节点会多访问一次，最多花费 O（n），查找节点的前驱最多也花费 O（n），所以整体时间复杂度为 O（3 * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        if(root == null) {
            return res;
        }

        // Morris先序遍历
        morrisPreTraversal(root, res);
        return res;
    }

    // Morris先序遍历：只出现一次的节点, 直接收集; 出现两次的节点, 第一次收集, 第二次不收集
    private void morrisPreTraversal(TreeNode root, List<Integer> res) {
        // 初始时, cur来到root位置
        TreeNode cur = root, mostRight = null;

        // cur为null, 则停止, 说明遍历到了树的最右最底的叶子节点了
        while (cur != null) {
            // 如果cur没有左孩子, 则cur向右移动
            mostRight = cur.left;
            if (mostRight == null) {
                // Morris先序遍历
                res.add(cur.val);
                cur = cur.right;
                continue;
            }

            // 如果cur有右孩子, 则找到左子树中最右的节点, 这里不等于mostRight.right != cur, 是为了避免发生第二次找不到正确前驱的错误
            while (mostRight.right != null && mostRight.right != cur) {
                // 一直向右移动
                mostRight = mostRight.right;
            }

            // 找到左子树最右的节点后, 则判断：
            // 如果mostRight右指针为null, 说明是第一次来到cur节点, 则让其指向cur, 然后cur向左移动
            if (mostRight.right == null) {
                mostRight.right = cur;

                // Morris先序遍历
                res.add(cur.val);
                cur = cur.left;
            }
            // 如果mostRight右指针指向cur, 说明是第二次来到cur节点, 则清空其指向, 以恢复本来的二叉树, 然后cur向右移动(此后cur再也不回不到这里及其左子树了)
            else {
                mostRight.right = null;
                cur = cur.right;
            }
        }
    }
}
```

#### 145. 二叉树的后序遍历 | easy

##### 1）递归法 | O（n）

- **思路**：递归遍历，左 -> 右 -> 中，因此在最后一次来到中时，再把节点的值加入集合中即可。
- **结论**：时间，0ms，100%，36.4mb，95.34%，时间上，由于每个节点只需要遍历一次，所以时间复杂度为 O（n），空间上，由于递归栈深度最大为 log n，所以额外空间复杂度为 O（log n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> res = new LinkedList<>();
        f(root, res);
        return res;
    }
    
    private void f(TreeNode root, List<Integer> res) {
        if(root == null) {
            return;
        }

        f(root.left, res);
        f(root.right, res);
        res.add(root.val);
    }
}
```

##### 2）迭代法 | O（n）

- **思路**：
  1. 观察后序遍历的左 -> 右 -> 中，对比先序遍历的中 -> 左 -> 右可知，如果可以先把先序遍历的迭代法结果改成中 -> 右 -> 左的话，那么后序遍历就刚好等于中 -> 右 -> 左的逆序。
  2. 因此，要把先序遍历的迭代法结果改成中 -> 右 -> 左，就需要先压入左、再压入右（从后往前），然后初始栈弹出时不能直接打印，而是装入另一个收集栈中，这样在最后依次弹出收集栈中的节点并打印，即可得到上面操作的逆序，即后序遍历。
- **结论**：时间，0ms，100%，36.5mb，78.24%，思路很简单，就是先序颠倒然后再逆序输出即可。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        LinkedList<TreeNode> stack = new LinkedList<>();
        LinkedList<TreeNode> rstack = new LinkedList<>();
        if(root != null) {
            stack.push(root);
        }

        while(!stack.isEmpty()) {
            root = stack.pop();
            if(root != null) {
                rstack.push(root);
                if(root.left != null) {
                    stack.push(root.left);
                }
                if(root.right != null) {
                    stack.push(root.right);
                }
            }
        }

        List<Integer> res = new LinkedList<>();
        while(!rstack.isEmpty()) {
            root = rstack.pop();
            if(root != null) {
                res.add(root.val);
            }
        }
        
        return res;
    }
}
```

##### 3）Morris 后序遍历 | O（6 * n）

- **思路**：见代码注释。
- **结论**：时间，0 ms，100%，空间，39.8 mb，20.31%，时间上，由于每个节点至少遍历一次，花费 O（n），有左孩子的根节点会多访问一次，最多花费 O（n），查找节点的前驱最多也花费 O（n），反转 + 收集 + 还原左孩子右边界，最多花费 O（3 * n），所以整体时间复杂度为 O（6 * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        if(root == null) {
            return res;
        }

        // Morris后序遍历
        morrisPostTraversal(root, res);
        return res;
    }

    // Morris后序遍历：只出现一次的节点, 不收集; 出现两次的节点, 第一次不收集, 
    // 第二次不收集, 但逆序收集左孩子为根节点的树的右边界, 最后再逆序收集原始root为根节点的树的右边界
    private void morrisPostTraversal(TreeNode root, List<Integer> res) {
        // 初始时, cur来到root位置
        TreeNode cur = root, mostRight = null;

        // cur为null, 则停止, 说明遍历到了树的最右最底的叶子节点了
        while (cur != null) {
            // 如果cur没有左孩子, 则cur向右移动
            mostRight = cur.left;
            if (mostRight == null) {
                cur = cur.right;
                continue;
            }

            // 如果cur有右孩子, 则找到左子树中最右的节点, 这里不等于mostRight.right != cur, 是为了避免发生第二次找不到正确前驱的错误
            while (mostRight.right != null && mostRight.right != cur) {
                // 一直向右移动
                mostRight = mostRight.right;
            }

            // 找到左子树最右的节点后, 则判断：
            // 如果mostRight右指针为null, 说明是第一次来到cur节点, 则让其指向cur, 然后cur向左移动
            if (mostRight.right == null) {
                mostRight.right = cur;
                cur = cur.left;
            }
            // 如果mostRight右指针指向cur, 说明是第二次来到cur节点, 则清空其指向, 以恢复本来的二叉树, 然后cur向右移动(此后cur再也不回不到这里及其左子树了)
            else {
                mostRight.right = null;

                // Morris后序遍历
                addPostTraversal(cur.left, res);
                cur = cur.right;
            }
        }

        // Morris后序遍历
        addPostTraversal(root, res);
    }

    // 逆序收集root为根节点的树的右边界
    private void addPostTraversal(TreeNode root, List<Integer> res) {
        if (root == null) {
            return;
        }

        // 先反转链表
        TreeNode orgTail = reverseList(root);

        // 再收集
        root = orgTail;
        while (root != null) {
            res.add(root.val);
            root = root.right;

        }

        // 再反转还原
        reverseList(orgTail);
    }

    // 反转链表：以right为指针
    private TreeNode reverseList(TreeNode head) {
        TreeNode pre = null, next;
        while (head != null) {
            // 反转
            next = head.right;
            head.right = pre;

            // 继续遍历
            pre = head;
            head = next;
        }
        return pre;
    }
}
```

#### 105.1. 重建二叉树 | 先序 + 中序 | medium

##### 1）递归法 | O（n）

- **思路**：
  1. 通过先序遍历的首位，确定并建立好根节点。
  2. 然后通过该根结点的值去哈希表中，拿到中序遍历的索引，根据该索引判断有没有右孩子和左孩子。
  3. 如果该索引往左至少还有 1 个节点，那么就认为还有左孩子，则把参数调整至左边，然后递归调用建立左孩子节点。
  4. 如果该索引往右至少还有 1 个节点，那么就认为还有右孩子，则把参数调整至右边，然后递归调用建立右孩子节点。
  5. 如果只有当前索引自己，那么则认为当前节点是一个叶子节点，则直接返回即可，周而复始。
- **结论**：时间，1ms，99.09%，空间，38.7mb，13.18%，效率还行，就这样吧~

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        if(preorder == null || preorder.length == 0 || inorder == null || inorder.length == 0) {
            return null;
        }

        Map<Integer, Integer> map = new HashMap<>();
        for(int i = 0; i < inorder.length; i++) {
            map.put(inorder[i], i);
        }

        return f(map, preorder, 0, preorder.length-1, inorder, 0, inorder.length-1);
    }

    // f代表从[ip...jp]与[ii...ji]范围内构建二叉树
    private TreeNode f(Map<Integer, Integer> map, int[] pre, int ip, int jp, int[] in, int ii, int ji) {
        TreeNode root = new TreeNode(pre[ip]);
        int r = map.get(pre[ip]);

        // 如果有左孩子
        if(r - ii > 0) {
            root.left = f(map, pre, ip+1, ip+r-ii, in, ii, r-1);
        }

        // 如果有右孩子
        if(ji - r > 0) {
            root.right = f(map, pre, ip+r-ii+1, jp, in, r+1, ji);
        }

        return root;
    }
}
```

#### 105.2. 重建二叉树 | 中序 + 后序 | medium

##### 1）递归法 | O（n）

- **思路**：
  1. 通过后序遍历的最后一位，确定并建立好根节点。
  2. 然后通过该根结点的值去哈希表中，拿到中序遍历的索引，根据该索引判断有没有右孩子和左孩子。
  3. 如果该索引往左至少还有 1 个节点，那么就认为还有左孩子，则把参数调整至左边，然后递归调用建立左孩子节点。
  4. 如果该索引往右至少还有 1 个节点，那么就认为还有右孩子，则把参数调整至右边，然后递归调用建立右孩子节点。
  5. 如果只有当前索引自己，那么则认为当前节点是一个叶子节点，则直接返回即可，周而复始。
- **结论**：时间，1ms，99.56%，空间，38.5mb，46.67%，效率还行，就这样吧~

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public TreeNode buildTree(int[] inorder, int[] postorder) {
        if(inorder == null || inorder.length == 0 || postorder == null || postorder.length == 0) {
            return null;
        }

        Map<Integer, Integer> map = new HashMap<>();
        for(int i = 0; i < inorder.length; i++) {
            map.put(inorder[i], i);
        }

        return f(map, inorder, 0, inorder.length-1, postorder, 0, postorder.length-1);
    }

    // f代表从[ii...ji]与[ip...jp]范围内构建二叉树
    private TreeNode f(Map<Integer, Integer> map, int[] in, int ii, int ji, int[] post, int ip, int jp) {
        TreeNode root = new TreeNode(post[jp]);
        int r = map.get(post[jp]);

        // 如果有左孩子
        if(r - ii > 0) {
            root.left = f(map, in, ii, r-1, post, ip, ip+r-ii-1);
        }

        // 如果有右孩子
        if(ji - r > 0) {
            root.right = f(map, in, r+1, ji, post, ip+r-ii, jp-1);
        }

        return root;
    }
}
```



#### 95. 不同的二叉搜索树 II | medium

##### 1）树形回溯 + 根节点枚举 | O（ [ n^2 / 4 ]* 2^[ n * (n - 1) ] ）

- **思路**：

  1. 根据二叉搜索树的特性（左子树都比根节点小、右子树都比根节点大），可以通过枚举根节点 i ，然后继续获取 [start, i - 1] 之间的左子树，[i + 1, end] 之间的右子树，来与当前根节点 i 进行拼装，拼装好之后则加入 res 返回结果中。 
  2. 其中要注意的是，枚举过程可以能出现 start > end 的情况，此时不应该返回空的 list，而是返回一个 null 元素的 list，这样方便上层进行封装为 null 的左右子树，而不是一个结果都没。

- **结论**：

  1. 时间，1 ms，97.36%，空间，42.2 mb，14.74%。
  2. 时间上，由于 dfs 每次需要枚举 n, n/2, n/4, ..., 1, 即 n * (n / 2) * (n / 4) * ... * 1，根据等比乘积公式可知，a1 = 1，q = 2，所以枚举花费了 2^[n * (n - 1)]，每次枚举组装 res 返回结果最多花费 n / 2 * n / 2 = n^2 / 4，所以整体时间复杂度为 O（ [ n^2 / 4 ]* 2^[ n * (n - 1) ] ）。
  3. 空间上，由于 dfs 栈的最大深度为 logn，所以额外空间复杂度为 O（logn）。

  ![1659489610094](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1659489610094.png)

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<TreeNode> generateTrees(int n) {
        return dfs(1, n);
    }

    // 获取[start, end]范围内的所有可行解
    private List<TreeNode> dfs(int start, int end) {
        LinkedList<TreeNode> res = new LinkedList<>();

        // 保证返回的List中, 肯定有元素, 即左右子树为空时, 返回个null
        if(start > end) {
            res.add(null);
            return res;
        }

        // 枚举[start, end]范围中的根节点
        for(int i = start; i <= end; i++) {
            // 获取左孩子的可行解, 由于是二叉搜索树, 所以范围是[start, i - 1]
            List<TreeNode> lefts = dfs(start, i - 1);

            // 获取右孩子的可行解, 由于是二叉搜索树, 所以范围是[i + 1, end]
            List<TreeNode> rights = dfs(i + 1, end);

            // 遍历左子树、右子树
            for(TreeNode left : lefts) {
                for(TreeNode right : rights) {
                    TreeNode root = new TreeNode(i);
                    root.left = left;
                    root.right = right;
                    res.add(root);
                }
            }
        }

        return res;
    }
}
```

#### 96. 不同的二叉搜索树 | medium

##### 1）暴力递归 | O（n^2）

- **思路**：
  1. 观察每个 n 的结果可以发现，如果固定好 n=0,1,2,3 这些返回值，然后把 n=4 看做成一个 [0,1,2,3] 的节点值数组，就可以很容易直到答案了。
  2. n=4 时，[0,1,2,3]，从左到右分为：f(0) * f(3) + f(1) * f(2) + f(2) * f(1) + f(3) * f(0) = 5 + 2 + 2 + 5 = 14。
     1. 当使用 nums[0] 作为根节点，左边有 0 个节点为 f（0），而右边有 3 个节点为 f（3），因此总的二叉搜索树种数有 f（0） * f（3）种。
     2. 当使用 nums[1] 作为根节点，左边有 1 个节点为 f（1），而右边有 2 个节点为 f（2），因此总的二叉搜索树种数有 f（1） * f（2）种。
     3. 当使用 nums[1] 作为根节点，左边有 2 个节点为 f（2），而右边有 1 个节点为 f（1），因此总的二叉搜索树种数有 f（2） * f（1）种。
     4. 当使用 nums[3] 作为根节点，左边有 3 个节点为 f（3），而右边有 0 个节点为 f（0），因此总的二叉搜索树种数有 f（3） * f（0）种。
     5. 因此，n=4 时，总的二叉树种数为：f(0) * f(3) + f(1) * f(2) + f(2) * f(1) + f(3) * f(0) = 5 + 2 + 2 + 5 = 14。
- **结论**：时间，475ms，6.63%，空间，34.9mb，84.22%，由于每次递归都需要往外扩散 n，从左到右需要触发 n 次，所以时间复杂度为 O（n^2）。

```java
class Solution {
    public int numTrees(int n) {
        if(n == 0) {
            return 1;
        }
        if(n == 1 || n == 2) {
            return n;
        }
        if(n == 3) {
            return 5;
        }

        // n=4时, [0,1,2,3], 从左到右分为: f(3)+f(1)*f(2)+f(2)*f(1)+f(3)=5+2+2+5=14
        int sum = 0;
        for(int i = 0; i < n; i++) {
            sum += numTrees(i) * numTrees(n-1-i);
        }

        return sum;
    }
}
```

##### 2）记忆化搜索 | O（n）

- **思路**：基于暴力递归的思路，增加了一个一维表的 dp 缓存数组，递归时，如果发现缓存中存在则从缓存获取，否则返回前先设置结果到缓存中再返回。
- **结论**：时间，0ms，100%，34.8mb，92.99%，效率非常高，这应该是该思路最优的了，再优化成动态规划就有点画蛇添足了。

```java
class Solution {
    public int numTrees(int n) {
        int[] dp = new int[n+1];
        Arrays.fill(dp, -1);
        return f(dp, n);
    }

    // n=4时, [0,1,2,3], 从左到右分为: f(3)+f(1)*f(2)+f(2)*f(1)+f(3)=5+2+2+5=14
    private int f(int[] dp, int n) {
        if(dp[n] != -1) {
            return dp[n];
        }
        if(n == 0) {
            dp[n] = 1;
            return dp[n];
        }
        if(n == 1 || n == 2) {
            dp[n] = n;
            return dp[n];
        }
        if(n == 3) {
            dp[n] = 5;
            return dp[n];
        }

        int sum = 0;
        for(int i = 0; i < n; i++) {
            sum += f(dp, i) * f(dp, n-1-i);
        }

        dp[n] = sum;
        return dp[n];
    }
}
```

#### 98. 验证二叉搜索树 | 树形 dp | medium

##### 1）树形 dp 递归 | O（n）

- **思路**：
  1. 通过根节点不断向下收集左孩子和右孩子的信息，来实现树形 dp。
  2. 由于需要左右孩子的信息来判断是否为二叉搜索树，所以结合概念得知，需要直到左、右孩子是否为二叉搜索树，以及最大值和最小值，因此需要建立一个特定数据结构 BstRes。
  3. 如果收集到左右孩子的信息后，则还要进行判断，生成当前根节点的信息，包括是否为二叉搜索树、最大值和最小值。
     1. 如果没有左右孩子，说明当前为叶子节点，那么返回是二叉搜索树，最大值为 val，最小值为 val。
     2. 如果同时有左右孩子，且左、右孩子都为二叉搜索树、 val 大于左孩子的最大值、val 小于右孩子的最小值，那么返回是二叉搜索树，最小值为左孩子最小，最大值为右孩子最大，否则返回 false。
     3. 如果只有左孩子，且左孩子为二叉搜索树、 val 大于左孩子的最大值，那么返回是二叉搜索树，最小值为左孩子最小，最大值为 val，否则返回 false。
     4. 如果只有右孩子，且右孩子为二叉搜索树、 val 小于右孩子的最小值，那么返回是二叉搜索树，最小值为 val，最大值为右孩子最大，否则返回 false。
- **结论**：时间，0ms，100%，38.1mb，51.71%，效率非常好，就这吧~

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {

    class BstRes {
        TreeNode node;
        boolean isBST;
        int min;
        int max;
        
        BstRes(TreeNode node, boolean isBST, int min, int max) {
            this.node = node;
            this.isBST = isBST;
            this.min = min;
            this.max = max;
        }
    }

    public boolean isValidBST(TreeNode root) {
        BstRes bstRes = f(root);
        return bstRes.isBST;
    }

    private BstRes f(TreeNode root) {
        if(root == null) {
            return new BstRes(root, true, root.val, root.val);
        }
        if(root.left == null && root.right == null) {
            return new BstRes(root, true, root.val, root.val);
        } 
        // 左孩子不为空, 右孩子不为空
        else if(root.left != null && root.right != null) {
            BstRes leftRes = f(root.left);
            BstRes rightRes = f(root.right);

            if(leftRes.isBST && rightRes.isBST
                & leftRes.max < root.val && root.val < rightRes.min) {
                return new BstRes(root, true, leftRes.min, rightRes.max);
            }
        } 
        // 左孩子不为空, 右孩子为空
        else if(root.left != null){
            BstRes leftRes = f(root.left);
            if(leftRes.max < root.val && leftRes.isBST) {
                return new BstRes(root, true, leftRes.min, root.val);
            }
        } 
        // 右孩子不为空, 左孩子为空
        else {
            BstRes rightRes = f(root.right);
            if(root.val < rightRes.min && rightRes.isBST) {
                return new BstRes(root, true, root.val, rightRes.max);
            }
        }

        return new BstRes(root, false, Integer.MAX_VALUE, Integer.MIN_VALUE);
    }
}
```

#### 99. 恢复二叉搜索树 | medium

##### 1）模拟 + 递归式中序遍历 + 值交换 | O（3 * n）

- **思路**：见代码注释，即中序遍历 => 找出问题索引（pre > list.get(i)）=> 根据索引找出问题节点 => 交换问题节点的值。
- **结论**：
  1. 时间，2 ms，40.98%，空间，41.4 mb，62.85%。
  2. 时间上，dfs 中序遍历收集结果花费 O（n），查找要交换的索引花费 O（n），dfs 查找要交换的节点花费 O（n），交换节点的值花费 O（1），所以整体时间复杂度为 O（3 * n）。
  3. 空间上，由于 dfs 中序遍历的栈最大深度为 logn，存放收集的结果 list 长 n，查找要交换节点的栈深度为 logn，所以整体额外空间复杂度为 O（2 * logn + n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    private int idx = 0;
    private int[] swapIdxs;
    private TreeNode[] swapNodes;

    public void recoverTree(TreeNode root) {
        // 中序遍历, 左中右：获取遍历结果
        List<Integer> list = new ArrayList<>();
        dfs(root, list);

        // 找出要交换的索引: 其中list#size为[2, 1000]
        this.swapIdxs = findSwapIdxs(list);
        this.swapNodes = new TreeNode[2];
        
        // 中序遍历, 左中右：根据swapIdxs查找要交换的节点
        findSwapNodes(root);

        // 交换from和to节点的值
        swapNodeVal(swapNodes[0], swapNodes[1]);
    }

    // 交换from和to节点的值
    private void swapNodeVal(TreeNode from, TreeNode to) {
        int tmp = to.val;
        to.val = from.val;
        from.val = tmp;
    }

    // 中序遍历, 左中右：根据swapIdxs查找要交换的节点
    private void findSwapNodes(TreeNode root) {
        if(root == null) {
            return;
        }

        // 左
        findSwapNodes(root.left);

        // 中
        if(idx == swapIdxs[0]) {
            this.swapNodes[0] = root;
        } else if(idx == swapIdxs[1]) {
            this.swapNodes[1] = root;
        }
        idx++;

        // 右
        findSwapNodes(root.right);
    }

    // 找出要交换的索引: 其中list#size为[2, 1000]
    private int[] findSwapIdxs(List<Integer> list) {
        int[] res = new int[2];

        int pre = list.get(0), errorCnt = 0;
        for(int i = 1; i < list.size(); i++) {
            // 前一个比后一个大
            if(pre > list.get(i)) {
                // 第一个索引
                if(errorCnt == 0) {
                    res[0] = i - 1;
                    errorCnt++;
                } 
                // 如果到这里, 那么说明list出现了2处问题对, 则再把i加入结果, 代表交换1问题前、2问题后的元素
                else if(errorCnt == 1) {
                    res[1] = i;
                    errorCnt++;
                    break;
                }
            }
            pre = list.get(i);
        }
        
        // 如果errorCnt为1, 那么说明list只出现1处问题对, 则把res[0]+1加入结果, 代表交换问题对的前后元素
        if(errorCnt == 1) {
            res[1] = res[0] + 1;
        }

        return res;
    }

    // 中序遍历, 左中右：获取遍历结果
    private void dfs(TreeNode root, List<Integer> list) {
        if(root == null) {
            return;
        }

        // 左
        dfs(root.left, list);

        // 中
        list.add(root.val);

        // 右
        dfs(root.right, list);
    }
}
```

##### 2）模拟 + 迭代式中序遍历 + 值交换 | O（n）

- **思路**：同样是中序遍历 => 找出问题索引（pre > list.get(i)）=> 根据索引找出问题节点 => 交换问题节点的值，但这里用了迭代法实现，使得只需要中序遍历一次即可。
- **结论**：时间，1 ms，100.00%，空间，41.8 mb，20.05%，时间上，由于最坏情况下只需要遍历一次整棵树，所以时间复杂度为 O（n），空间上，由于使用了一个栈，深度最大为 logn，所以额外空间复杂度为 O（log n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public void recoverTree(TreeNode root) {
        LinkedList<TreeNode> stack = new LinkedList<>();
        while(root != null) {
            stack.push(root);
            root = root.left;
        }

        int cnt = 0;
        TreeNode pre = null, swapNode1 = null, swapNode2 = null;
        while(!stack.isEmpty()) {
            root = stack.pop();
            if(root != null) {
                if(pre != null && pre.val > root.val) {
                    // 第一个问题对, 取前一个节点
                    if(swapNode1 == null) {
                        swapNode1 = pre;
                    }

                    // 第一个或者第二个问题对, 取后一个节点
                    swapNode2 = root;
                    cnt++;
                    
                    // 最多存在2对, 多了则退出
                    if(cnt == 2) {
                        break;
                    }
                }
                
                // 更新前驱
                pre = root;

                // 迭代法实现中序遍历
                if(root.right != null) {
                    root = root.right;
                    while(root != null) {
                        stack.push(root);
                        root = root.left;
                    }
                }
            }
        }

        // 交换from和to节点的值
        swapNodeVal(swapNode1, swapNode2);
    }

    // 交换from和to节点的值
    private void swapNodeVal(TreeNode from, TreeNode to) {
        int tmp = to.val;
        to.val = from.val;
        from.val = tmp;
    }
}
```

##### 3）模拟 + Morris 中序遍历 + 值交换 | O（3 * n）

- **思路**：同样是中序遍历 => 找出问题索引（pre > list.get(i)）=> 根据索引找出问题节点 => 交换问题节点的值，但这里用了 Morris 中序遍历实现，只需要使用有限几个变量即可，不过要注意不能像迭代法那样提前 break 掉，因为提前退出 Morris 循环，会使得某些叶子节点的 right 指针没清空掉，树没还原回去就返回了，导致树循环的情况发生。
- **结论**：时间，1 ms，100.00%，空间，41.3 mb，84.83%，时间上，由于每个节点至少遍历一次，花费 O（n），有左孩子的根节点会多访问一次，最多花费 O（n），查找节点的前驱最多也花费 O（n），所以整体时间复杂度为 O（3 * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public void recoverTree(TreeNode root) {
        // 0: 前一个交换点, 1：后一个交换点
        TreeNode[] swapNodes = morrisInTraversal(root);

        // 交换from和to节点的值
        swapNodeVal(swapNodes[0], swapNodes[1]);
    }

    // 交换from和to节点的值
    private void swapNodeVal(TreeNode from, TreeNode to) {
        int tmp = to.val;
        to.val = from.val;
        from.val = tmp;
    }

    // Morris中序遍历：只出现一次的节点, 直接收集; 出现两次的节点, 第一次不收集, 第二次才收集
    private TreeNode[] morrisInTraversal(TreeNode root) {
        // 0: 前一个交换点, 1：后一个交换点
        TreeNode[] swapNodes = new TreeNode[2];

        // 初始时, cur来到root位置
        TreeNode cur = root, mostRight = null, pre = null;

        // cur为null, 则停止, 说明遍历到了树的最右最底的叶子节点了
        while (cur != null) {
            // 如果cur没有左孩子, 则cur向右移动
            mostRight = cur.left;
            if (mostRight == null) {
                // 改写Morris中序遍历 => 设置前后交换点
                setSwapNodes(swapNodes, cur, pre);
                // 更新前驱
                pre = cur;

                cur = cur.right;
                continue;
            }

            // 如果cur有右孩子, 则找到左子树中最右的节点, 这里不等于mostRight.right != cur, 是为了避免发生第二次找不到正确前驱的错误
            while (mostRight.right != null && mostRight.right != cur) {
                // 一直向右移动
                mostRight = mostRight.right;
            }

            // 找到左子树最右的节点后, 则判断：
            // 如果mostRight右指针为null, 说明是第一次来到cur节点, 则让其指向cur, 然后cur向左移动
            if (mostRight.right == null) {
                mostRight.right = cur;
                cur = cur.left;
            }
            // 如果mostRight右指针指向cur, 说明是第二次来到cur节点, 则清空其指向, 以恢复本来的二叉树, 然后cur向右移动(此后cur再也不回不到这里及其左子树了)
            else {
                mostRight.right = null;

                // 改写Morris中序遍历 => 设置前后交换点
                setSwapNodes(swapNodes, cur, pre);
                // 更新前驱
                pre = cur;

                cur = cur.right;
            }
        }

        return swapNodes;
    }

    // 设置前后交换点
    private void setSwapNodes(TreeNode[] swapNodes, TreeNode cur, TreeNode pre) {
        if(pre != null && pre.val > cur.val) {
            // 第一个问题对, 取前一个节点
            if(swapNodes[0] == null) {
                swapNodes[0] = pre;
            }

            // 第一个或者第二个问题对, 取后一个节点
            swapNodes[1] = cur;
        }
    }
}
```

#### 100. 相同的树 | easy

##### 1）深度优先搜索 | O（n）

- **思路**：只当结构相同、每个根节点相等、每个左节点相等、每个右节点相等，两棵树才完全相同。
- **结论**：时间，0 ms，100%，空间，38.6 mb，95.50%，时间上，由于最坏情况下需要遍历整棵树，所以时间复杂度为 O（n），空间上，递归栈的最大深度为 logn，所以额外空间复杂度为 O（logn）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public boolean isSameTree(TreeNode p, TreeNode q) {
        if(p == null && q == null) {
            return true;
        }
        if(p == null || q == null) {
            return false;
        }

        return p.val == q.val && isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
    }
}
```

##### 2）广度优先搜索 | O（n）

- **思路**：通过队列实现广度优先搜索，判断的逻辑一样，都是只当结构相同、每个根节点相等、每个左节点相等、每个右节点相等，两棵树才完全相同。
- **结论**：时间，0 ms，100%，空间，38.8 mb，77.05%，由于最坏情况下需要遍历整棵树，所以时间复杂度为 O（n），空间上，队列长度为 n，所以额外空间复杂度为 O（n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public boolean isSameTree(TreeNode p, TreeNode q) {
        if(p == null && q == null) {
            return true;
        }
        if(p == null || q == null) {
            return false;
        }

        Queue<TreeNode[]> queue = new LinkedList<>();
        queue.offer(new TreeNode[] { p, q });
        while(!queue.isEmpty()) {
            TreeNode[] nodes = queue.poll();
            p = nodes[0];
            q = nodes[1];

            if(p == null && q == null) {
                continue;
            }
            if(p != null && q == null){
                return false;
            }
            if(p == null && q != null){
                return false;
            }
            if(p.val != q.val) {
                return false;
            }

            // 左孩子
            queue.offer(new TreeNode[] { p.left, q.left });

            // 右孩子
            queue.offer(new TreeNode[] { p.right, q.right });
        }

        return true;
    }
}
```

#### 101. 对称二叉树 | easy

##### 1）深度优先搜索1 | O（2n）

- **思路**：
  1. 观察对称二叉树可以发现，它的中序遍历也是对称的，因此可以沿用中序遍历的套路收集到一个 list 中，最后再作判断。
  2. 其中要注意的是，由于要求是对称的，也就是左对右、右对左，因此，list 收集到的元素是要有方向标识的，这里采用了 val * 1 代表左，val * （-1）代表右，而刚开始的 root.val 不变。
  3. 这样在最后对 list 做判断时，就需要双指针从两个不同的方向判断，如果除中间外，处处左右值相加和为 0，且中间的位置等于 root.val，那么就说明确实是棵对称二叉树，否则就不是。 
- **结论**：时间，1ms，24.74%，空间，37.9mb，6.50%，可能是因为做了两次遍历，时间复杂度为 O（2n），所以效率不是那么高？

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public boolean isSymmetric(TreeNode root) {
        if(root == null) {
            return true;
        }

        List<Integer> res = new ArrayList<>();
        f(root, res, true);

        int i = 0, j = res.size() - 1, sum = 0;
        while(i < j) {
            sum += res.get(i++) + res.get(j--);
            if(sum != 0) {
                return false;
            }
        }

        return res.get(i) == root.val;
    }

    private void f(TreeNode root, List<Integer> res, boolean isLeft) {
        if(root == null) {
            return;
        }

        f(root.left, res, true);// * 1
        res.add(isLeft? root.val : -root.val);
        f(root.right, res, false);// * (-1)
    }
}
```

##### 2）深度优先搜索2 | O（n）

- **思路**：
  1. 同样是深度优先搜索，但该思路不同于第一种深度优先搜索的思路，将不再拘泥于取到数值后再判断是否对称，而是在遍历过程中就判断到是否为对称，这样做的好处一来简单好实现，二来如果发现不对称，则可以提前返回结果，而不用等到全部遍历完再做处理，提升了效率。
  2. 由于要求的是判断是否为对称二叉树，也就是左.val == 右.val、左.left.val == 右.right.val、左.right.val == 右.left.val，则为对称二叉树，否则就不是。
- **结论**：
  1. 时间，0ms，100%，36.3mb，84.68%，可见对比第一种深度优先搜索，这种边递归边判断的方式大大提高了效率。
  2. 在最坏的情况下，树是链表的形状，使得整个递归深度等于节点的个数，因此额外空间复杂度也为 O（n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public boolean isSymmetric(TreeNode root) {
        if(root == null) {
            return true;
        }
        if(root.left == null && root.right == null) {
            return true;
        }
        if(root.left != null && root.right != null) {
            return f(root.left, root.right);
        }
        return false;
    }

    // f函数用于判断t1和t2是否互为镜像, 也就是t1==t2, t1.l == t2.r, t1.r == t2.l
    private boolean f(TreeNode t1, TreeNode t2) {
        if(t1 == null && t2 == null) {
            return true;
        } else if(t1 != null && t2 != null) {
            if(t1.val != t2.val) {
                return false;
            } else {
                return f(t1.left, t2.right) && f(t1.right, t2.left);
            }
        } else {
            return false;
        }
    }
}
```

##### 3）宽度优先搜索 | O（n）

- **思路**：宽度优先搜索首先想到的就是队列，但与层序遍历不同的是，由于这里要判断的是否为对称二叉树，所以要交叉比较，而不是顺序加入节点，即先加入 t1.left 和 t2.right，再加入 t1.right 和 t1.left。
- **结论**：时间，1ms，24.68%，37.6mb，20.57%，虽然时间复杂度也是 O（n），但由于使用了额外的数据结构，使得时间上不如深度优先搜索2（系统压栈）来的快？以及额外空间使用了队列，所以也是 O（n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public boolean isSymmetric(TreeNode root) {
        if(root == null) {
            return true;
        }
        if(root.left == null && root.right == null) {
            return true;
        }
        if(root.left != null && root.right != null) {
            Queue<TreeNode> queue = new LinkedList<>();
            queue.offer(root.left);
            queue.offer(root.right);

            TreeNode t1, t2;
            while(!queue.isEmpty()) {
                t1 = queue.poll();
                t2 = queue.poll();

                if(t1 == null && t2 == null) {
                    continue;    
                }
                else if(t1 != null && t2 != null) {
                    if(t1.val != t2.val) {
                        return false;
                    }
                    queue.offer(t1.left);
                    queue.offer(t2.right);
                    queue.offer(t1.right);
                    queue.offer(t2.left);
                }
                else {
                    return false;     
                }
            }

            return true;
        }
        
        return false;
    }
}
```

#### 102. 二叉树的层序遍历 | 宽度优先遍历 | medium

##### 1）迭代式广度优先搜索 + 从头顺序插入结果 + 最终顺序收集 | O（n）

- **思路**：
  1. 宽度优先遍历首先想到的就是用队列，先把头节点入队，然后出队时添加它的左孩子和右孩子，周而复始。
  2. 但这样仅仅只能得到遍历的序列而已，还无法确定那几个值属于同一层的，所以还需要增加一个层号表，以及维护一个当前层号。
  3. 在节点进队前，先把层号与节点映射起来（其中如果入队的是左孩子或者右孩子，其层号要+1），下次节点出队时就可以比较当前层号，如果发现节点层号与当前层号不同，说明当前出队的节点已经是下一层的了，这时候就结算上一层的节点，然后当前层号+1。
- **结论**：时间，2 ms，89.72%，空间，38.5 mb，71.49%，时间上，由于遍历整棵树，所以时间复杂度为 O（n），空间上，由于使用了一条最长为 2^（logn）的 queue 来实现广搜，所以额外空间复杂度为 O（2^[logn]）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        if(root == null) {
            return new ArrayList<>();
        }

        Map<TreeNode, Integer> levelMap = new HashMap<>();
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        levelMap.put(root, 1);
        int curLevel = 1, nodelevel;

        List<List<Integer>> res = new ArrayList<>();
        List<Integer> curLevelList = new ArrayList();

        while(!queue.isEmpty()) {
            root = queue.poll();
            nodelevel = levelMap.get(root);

            // 同一层, 则加入curLevelList
            if(nodelevel == curLevel) {
                curLevelList.add(root.val);
            } 
            // 不同一层, 则结算curLevelList, 并开启新的-curLevelList
            else {
                res.add(new ArrayList<>(curLevelList));
                curLevel++;
                curLevelList.clear();
                curLevelList.add(root.val);
            }

            // 左孩子
            if(root.left != null) {
                queue.offer(root.left);
                levelMap.put(root.left, nodelevel + 1);
            }

            // 右孩子
            if(root.right != null) {
                queue.offer(root.right);
                levelMap.put(root.right, nodelevel + 1);
            }
        }

        // 结算最后一层的节点
        res.add(curLevelList);
        return res;
    }
}
```

#### 103. 二叉树的锯齿形层序遍历 | medium

##### 1）迭代式广度优先搜索 + 头尾轮流插入结果 + 最终顺序收集 | O（n）

- **思路**：参考《102. 二叉树的层序遍历》的广搜思路，不过这里由于偶数层（从 1 开始）收集的结果要反转，所以使用了一个双端队列来替代，碰到这种情况则从头反向插入，其他情况则继续顺序插入即可。
- **结论**：时间，1 ms，70.33%，空间，41.2 mb，10.50%，时间上，由于遍历整棵树，所以时间复杂度为 O（n），空间上，由于使用了一条最长为 2^（logn）的 queue 来实现广搜，所以额外空间复杂度为 O（2^[logn]）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        if(root == null) {
            return new ArrayList<>();
        }

        Map<TreeNode, Integer> levelMap = new HashMap<>();
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        levelMap.put(root, 1);

        int curLevel = 1, nodelevel;
        List<List<Integer>> res = new ArrayList<>();
        List<Integer> curLevelDeque = new LinkedList();

        while(!queue.isEmpty()) {
            root = queue.poll();
            nodelevel = levelMap.get(root);

            // 同一层, 则加入curLevelDeque
            if(nodelevel == curLevel) {
                insertValByLevel((Deque<Integer>) curLevelDeque, curLevel, root.val);
            } 
            // 不同一层, 则结算curLevelDeque, 并开启新的-curLevelDeque
            else {
                res.add(new ArrayList<>(curLevelDeque));
                curLevel++;
                curLevelDeque.clear();
                insertValByLevel((Deque<Integer>) curLevelDeque, curLevel, root.val);
            }

            // 左孩子
            if(root.left != null) {
                queue.offer(root.left);
                levelMap.put(root.left, nodelevel + 1);
            }

            // 右孩子
            if(root.right != null) {
                queue.offer(root.right);
                levelMap.put(root.right, nodelevel + 1);
            }
        }

        // 结算最后一层的节点
        res.add(curLevelDeque);
        return res;
    }

    // 根据层号插入值到结果中
    // 比如level=2时拿到9->20, 应该从头插入, 最后从头遍历便得到[20, 9], 
    // 再如level=3时拿到15->7, 应该从尾插入, 最后从头遍历便得到[15, 7] 
    private void insertValByLevel(Deque<Integer> curLevelDeque, int curLevel, int val) {
        // 偶数头插
        if(curLevel % 2 == 0) {
            curLevelDeque.offerFirst(val);
        } 
        // 奇数尾插
        else {
            curLevelDeque.offerLast(val);
        }
    }
}
```

#### 104. 二叉树的最大深度 | easy

##### 1）深度优先搜索 | O（n）

- **思路**：
  1. 定义一个 f（root，curlevel）函数，用于获取 root 的最大深度。
  2. 这样每来到一个 root 节点时，都可以先拿左右孩子的最大深度，然后比较当前的深度，最后返回个最大的深度即可。
- **结论**：时间，0ms，100%，38mb，91.85%，也可以尝试宽度有限遍历实现。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public int maxDepth(TreeNode root) {
        return f(root, 1);
    }

    // f代表获取root的最大深度
    private int f(TreeNode root, int curlevel) {
        if(root == null) {
            return 0;
        }

        int leftMax = f(root.left, curlevel+1);
        int rightMax = f(root.right, curlevel+1);

        return Math.max(curlevel, Math.max(leftMax, rightMax));
    }
}
```

##### 2）宽度优先搜索 | O（n）

- **思路**：
  1. 宽度优先搜索首先想到的就是用队列，首先把根节点加入队列，然后出队时不断把左、右孩子加入队列，循环往复。
  2. 不过如果要返回最大深度，则还要通过使用哈希表，提前记录好节点的层数，再节点出队时与当前层数变量相比较，如果比较发现层数不一样，那么说明来到下一层了，然后更新当前层数的值，最后返回拿到的当前层数的值即可。
- **结论**：时间，3ms，21.57%，空间，38.2mb，73.07%，可见面对求最大深度的问题，深度优先搜索比宽度优先搜索，无论是在时间上，还是在空间上都更有优势！

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public int maxDepth(TreeNode root) {
        if(root == null) {
            return 0;
        }

        Map<TreeNode, Integer> map = new HashMap<>();
        LinkedList<TreeNode> queue = new LinkedList<>();
        if(root != null) {
            map.put(root, 1);
            queue.offer(root);
        }

        int curlevel = 1, level;
        while(!queue.isEmpty()) {
            root = queue.poll();
            if(root != null) {
                level = map.get(root);
                if(level != curlevel) {
                    curlevel++;
                }
                if(root.left != null) {
                    map.put(root.left, level+1);
                    queue.offer(root.left);
                }
                if(root.right != null) {
                    map.put(root.right, level+1);
                    queue.offer(root.right);
                }
            }
        }

        return curlevel;
    }
}
```

#### 107. 二叉树的层序遍历 II | medium

##### 1）迭代式广度优先搜索 + 从头顺序插入结果 + 最终逆序收集| O（n）

- **思路**：参考《102. 二叉树的层序遍历》的广搜思路，不过这里最后要从头反向插入中间结果到 res 中，才能实现从最后一行到第一行的效果。
- **结论**：时间，2 ms，91.31%，空间，41.3 mb，78.53%，时间上，由于遍历整棵树，所以时间复杂度为 O（n），空间上，由于使用了一条最长为 2^（logn）的 queue 来实现广搜，所以额外空间复杂度为 O（2^[logn]）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<List<Integer>> levelOrderBottom(TreeNode root) {
        if(root == null) {
            return new ArrayList<>();
        }

        Map<TreeNode, Integer> levelMap = new HashMap<>();
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        levelMap.put(root, 1);
        int curLevel = 1, nodelevel;

        List<List<Integer>> res = new LinkedList<>();
        List<Integer> curLevelList = new ArrayList();

        while(!queue.isEmpty()) {
            root = queue.poll();
            nodelevel = levelMap.get(root);

            // 同一层, 则加入curLevelList
            if(nodelevel == curLevel) {
                curLevelList.add(root.val);
            } 
            // 不同一层, 则结算curLevelList, 并开启新的-curLevelList
            else {
                ((Deque<List<Integer>>) res).offerFirst(new ArrayList<>(curLevelList));
                curLevel++;
                curLevelList.clear();
                curLevelList.add(root.val);
            }
            
            // 左孩子
            if(root.left != null) {
                queue.offer(root.left);
                levelMap.put(root.left, nodelevel + 1);
            }

            // 右孩子
            if(root.right != null) {
                queue.offer(root.right);
                levelMap.put(root.right, nodelevel + 1);
            }
        }

        // 结算最后一层的节点
        ((Deque<List<Integer>>) res).offerFirst(curLevelList);
        return res;
    }
}
```

#### 108. 将有序数组转换为二叉搜索树 | easy

##### 1）分治 + 中序遍历 + 中左作为根节点 | O（n）

- **思路**：采用分治的递归操作，不断找到中点来建立根节点，以保证左右平衡，然后根据根节点和左右边界建立左右子树，最后返回当前建立好的二叉搜索树，周而复始。不过这里是选择中左作为根节点。
- **结论**：时间，0 ms，空间，41.3 mb，52.67%，时间上，由于需要遍历整个数组，所以时间复杂度为 O（n），空间上，由于递归栈最深为 log n，所以额外空间复杂度为 O（log n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        return f(nums, 0, nums.length - 1);
    }

    private TreeNode f(int[] nums, int l, int r) {
        // 当上层l==r时，下层才会出现这种情况
        if(l > r) {
            return null;
        }
		
        // 选取中间节点的左边，来作为根节点
        int mid = l + (r - l) / 2;
        TreeNode root = new TreeNode(nums[mid]);
        root.left = f(nums, l, mid - 1);
        root.right = f(nums, mid + 1, r);
        return root;
    }
}
```

##### 2）分治+ 中序遍历 + 中右作为根节点 | O（n）

- **思路**：采用分治的递归操作，不断找到中点来建立根节点，以保证左右平衡，然后根据根节点和左右边界建立左右子树，最后返回当前建立好的二叉搜索树，周而复始。不过这里是选择中右作为根节点。
- **结论**：时间，0 ms，空间，41.3 mb，52.67%，时间上，由于需要遍历整个数组，所以时间复杂度为 O（n），空间上，由于递归栈最深为 log n，所以额外空间复杂度为 O（log n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        return f(nums, 0, nums.length - 1);
    }

    private TreeNode f(int[] nums, int l, int r) {
        // 当上层l==r时，下层才会出现这种情况
        if(l > r) {
            return null;
        }

        // 选取中间节点的右边，来作为根节点
        int mid = l + (r - l + 1)/ 2;
        TreeNode root = new TreeNode(nums[mid]);
        root.left = f(nums, l, mid - 1);
        root.right = f(nums, mid + 1, r);
        return root;
    }
}
```

#### 109. 有序链表转换二叉搜索树 | medium

##### 1）分治 + 中序遍历 + 快慢指针 | O（n * logn）

- **思路**：

  1. 采用分治的递归操作，不断找到中点来建立根节点，以保证左右平衡，然后根据根节点和左右边界建立左右子树，最后返回当前建立好的二叉搜索树，周而复始。
  2. 不过这里查找根节点，是通过快慢指针的方式，查找链表的中点，相当于中左作为根节点。
  3. 而想要中右作为根节点的话，则需要更改快慢查找根逻辑。

- **结论**：

  1. 时间，0 ms，空间，41.3 mb，52.67%。

  2. 时间上，很明显，该递归操作符合 Master 公式，即 T（n）= 2 * T(n / 2) + O（n^d），n^d 是指快慢指针查好链表中点的时间复杂度，为 O（n），所以这里 a =2，b = 2，d = 1，即 d = log b [a]，因此整体时间复杂度为 O（n * logn）。

     ![1654349496899](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1654349496899.png)

  3. 空间上，由于递归深度最大为 log n，所以额外空间复杂度为 O（log n）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public TreeNode sortedListToBST(ListNode head) {
        return f(head, null);
    }

    // 根据链表, 建立二叉树, 左闭右开
    private TreeNode f(ListNode l, ListNode r) {
        if(l == r) {
            return null;
        }

        ListNode mid = findMidListNode(l, r);
        TreeNode root = new TreeNode(mid.val);
        root.left = f(l, mid);
        root.right = f(mid.next, r);
        return root;
    }

    // 快慢指针法查找链表中点
    private ListNode findMidListNode(ListNode head, ListNode tail) {
        ListNode slow = head, fast = head.next;
        while(fast != tail && fast.next != tail) {
            fast = fast.next.next;
            slow = slow.next;
        }

        return slow;
    }
}
```

##### 2）分治 + 中序遍历 + 提前占位根节点 | O（2 * n）

- **思路**：参考《108. 将有序数组转换为二叉搜索树》的思路，不过这里是通过全局变量 cur，设置根节点的值，再利用中序遍历的顺序，改变 cur 的指向，最后完成搜索二叉树的重建。
- **结论**：时间，0 ms，100%，空间，42.7 mb，45.63%，时间上，由于需要遍历整个链表 2 次，所以时间复杂度为 O（2 * n），空间上，由于递归栈最深为 log n，所以额外空间复杂度为 O（log n）。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    private ListNode cur;

    public TreeNode sortedListToBST(ListNode head) {
        this.cur = head;
        return f(0, getListLen(head) - 1);
    }

    // 左闭右闭, 重建搜索二叉树
    private TreeNode f(int l, int r) {
        // 当上层l==r时，下层才会出现这种情况
        if(l > r) {
            return null;
        }

        // 选取中间节点的左边，来作为根节点
        // int mid = l + (r - l) / 2;
        // 选取中间节点的右边，来作为根节点
        int mid = l + (r - l + 1)/ 2;

        // 提前占位
        TreeNode root = new TreeNode();

        // 中序遍历, 设置左子树
        root.left = f(l, mid - 1);

        // 中序遍历, 设置根节点的值
        root.val = cur.val;
        cur = cur.next;

        // 中序遍历, 设置右子树
        root.right = f(mid + 1, r);
        return root;
    }

    // 获取链表长度
    private int getListLen(ListNode head) {
        int len = 0;
        while(head != null) {
            len++;
            head = head.next;
        }
        return len;
    }
}
```

#### 110. 平衡二叉树 | 树形 dp | easy

##### 1）树形 dp | O（n）

- **思路**：树形 dp，不断向左、向右子树获取信息，然后聚合判断是否平衡，最后把结果向上汇报，周而复始。
- **结论**：时间，1 ms，44.70%，空间，41 mb，68.21%，时间上，由于需要遍历整棵二叉树，所以时间复杂度为 O（n），空间上，由于树可能退化为链表，即递归栈深度最大为 n，所以额外空间复杂度为 O（n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Info {
    int height;
    boolean isBalanced;
    
    Info(int height, boolean isBalanced) {
        this.height = height;
        this.isBalanced = isBalanced;
    }
}
class Solution {
    public boolean isBalanced(TreeNode root) {
        if(root == null) {
            return true;
        }

        return f(root).isBalanced;
    }

    private Info f(TreeNode root) {
        if(root == null) {
            return new Info(0, true);
        }

        Info li = f(root.left);
        Info ri = f(root.right);

        // 左平衡、右平衡、整体平衡
        if(li.isBalanced && ri.isBalanced && Math.abs(li.height - ri.height) <= 1) {
            return new Info(Math.max(li.height, ri.height) + 1, true);
        } 
        // 非平衡二叉树
        else {
            return new Info(Math.max(li.height, ri.height) + 1, false);
        }
    }
}
```

##### 2）树形 dp 优化 | O（n）

- **思路**：在 1 的基础上，递归函数不再返回 Info 数据结构，而只返回 root 二叉树的高度，以及同时利用这个值等于 -1，来表示 root 二叉树非平衡，以减少内存开销。
- **结论**：时间，0 ms，100.00%，空间，41.3 mb，27.58%，时间上，由于需要遍历整棵二叉树，所以时间复杂度为 O（n），空间上，由于树可能退化为链表，即递归栈深度最大为 n，所以额外空间复杂度为 O（n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public boolean isBalanced(TreeNode root) {
        if(root == null) {
            return true;
        }
        return isBalanced(f(root));
    }

    // 根据高度, 判断二叉树是否平衡
    private boolean isBalanced(int height) {
        return height != -1;
    }

    private int f(TreeNode root) {
        if(root == null) {
            return 0;
        }

        int lh = f(root.left);
        int rh = f(root.right);

        // 左平衡、右平衡、整体平衡
        if(isBalanced(lh) && isBalanced(rh) && Math.abs(lh - rh) <= 1) {
            return Math.max(lh, rh) + 1;
        } 
        // 非平衡二叉树
        else {
            return -1;
        }
    }
}
```

#### 111. 二叉树的最小深度 | easy

##### 1）树形 dp | O（n）

- **思路**：树形 dp，不断判断是向左、向右子树获取信息，然后把结果向上汇报，只有碰到叶子节点，才决定最小的深度为 1，其他的则在这基础上累加 1，周而复始。
- **结论**：时间，8 ms，31.57%，空间，61.1 mb，46.45%，时间上，由于需要遍历整棵二叉树，所以时间复杂度为 O（n），空间上，由于树可能退化为链表，即递归栈深度最大为 n，所以额外空间复杂度为 O（n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public int minDepth(TreeNode root) {
        return f(root);
    }

    private int f(TreeNode root) {
        if(root == null) {
            return 0;
        }
        // 到达叶子节点
        if(root.left == null && root.right == null) {
            return 1;
        }

        // 如果不存在左孩子, 那么就看右孩子
        if(root.left == null) {
            return f(root.right) + 1;
        } 
        // 如果不存在右孩子, 那么就看左孩子
        else if(root.right == null) {
            return f(root.left) + 1;
        }
        // 如果左、右孩子同时存在, 那么返回最小值+1
        else {
            return Math.min(f(root.left), f(root.right)) + 1;
        }
    }
}
```

#### 112. 路径总和 | 树形 dp | easy

##### 1）树形 dp | O（n）

- **思路**：树形 dp，不断判断是向左、向右子树获取信息，然后把结果向上汇报，只有碰到叶子节点，才决定路径总和是否等于剩余的 rest，其他的则在原始的 rest 上扣减 val，周而复始。
- **结论**：时间，0 ms，100.00%，空间，41.5 mb，24.36%，时间上，由于需要遍历整棵二叉树，所以时间复杂度为 O（n），空间上，由于树可能退化为链表，即递归栈深度最大为 n，所以额外空间复杂度为 O（n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public boolean hasPathSum(TreeNode root, int rest) {
        // 通过设计, 这里不会到达
        if(root == null) {
            return false;
        }
        // 到达叶子节点
        if(root.left == null && root.right == null) {
            return rest == root.val;
        }

        // 非叶子节点
        rest -= root.val;

        // 如果不存在左孩子, 那么就看右孩子
        if(root.left == null) {
            return hasPathSum(root.right, rest);
        } 
        // 如果不存在右孩子, 那么就看左孩子
        else if(root.right == null) {
            return hasPathSum(root.left, rest);
        }
        // 如果左、右孩子同时存在, 那么就同时看左右孩子
        else {
            return hasPathSum(root.left, rest) || hasPathSum(root.right, rest);
        }
    }
}
```

#### 113. 路径总和 II | 树形 dp | medium

##### 1）树形 dp + 回溯 | O（n）

- **思路**：参考《112. 路径总和》的思路，不断向下收集信息，但不同的是，这里结合了回溯，对路上符合条件的进行收集，每碰到一个叶子节点，则判断是否符合条件，符合的则结算，周而复始。
- **结论**：时间，1 ms，99.98%，空间，42.1 mb，12.64%，时间上，由于需要遍历整棵二叉树，所以时间复杂度为 O（n），空间上，由于树可能退化为链表，即递归栈深度最大为 n，所以额外空间复杂度为 O（n）。

```shell
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    private List<List<Integer>> res;
    private List<Integer> cur;

    public List<List<Integer>> pathSum(TreeNode root, int rest) {
        this.res = new ArrayList<>();
        this.cur = new ArrayList<>();
        f(root, rest);
        return res;
    }

    private void f(TreeNode root, int rest) {
        // 通过设计, 这里不会到达
        if(root == null) {
            return;
        }
        // 到达叶子节点, 结算是否存在路径
        if(root.left == null && root.right == null) {
            if(rest == root.val) {
                cur.add(root.val);
                res.add(new ArrayList<>(cur));
                cur.remove(cur.size() - 1);
            }
            return;
        }

        // 非叶子节点, 则追加当前节点的值
        cur.add(root.val);
        rest -= root.val;

        // 如果不存在左孩子, 那么就看右孩子
        if(root.left == null) {
            f(root.right, rest);
        } 
        // 如果不存在右孩子, 那么就看左孩子
        else if(root.right == null) {
            f(root.left, rest);
        }
        // 如果左、右孩子同时存在, 那么就同时看左右孩子
        else {
            f(root.left, rest);
            f(root.right, rest);
        }

        // 回溯
        cur.remove(cur.size() - 1);
    }
}
```

#### 114. 二叉树展开为链表 | medium

##### 1）暴力解法1 | O（2n）

- **思路**：先做一次先序遍历，把需要的值都收集起来，然后再顺序遍历，使用 root 节点构造满足题意的链表即可。
- **结论**：时间，2ms，35.68%，空间，37.7mb，76.02%，由于需要先收集再遍历，所以时间复杂度为 O（2n），而空间上由于遍历时构建了 n-1 个新的节点，以及使用了 1 个 list，以及递归时最大深度为 n，所以额外空间复杂度为 O（3n），这种取巧的方式面试会挂，需要在原树上进行修改指针！

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public void flatten(TreeNode root) {
        List<Integer> res = new LinkedList<>();
        f(root, res);
        
        for(int i = 1; i < res.size(); i++) {
            root.right = new TreeNode(res.get(i));
            root.left = null;
            root = root.right;
        }
    }

    private void f(TreeNode root, List<Integer> res) {
        if(root == null) {
            return;
        }

        res.add(root.val);
        f(root.left, res);
        f(root.right, res);
    }
}
```

##### 2）暴力解法2 | O（n）

- **思路**：还是沿用先序遍历的方式，不过采用了迭代法遍历进行优化，使得能够在遍历的过程中实现链表的拼接，在时间复杂度的常数项上进行了缩减，使得只遍历 1 次即可。
- **结论**：时间，1ms，36.68%，空间，37.8mb，61.20%，由于空间上使用了一个栈，因此额外空间复杂度为 O（n），还需要继续优化。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public void flatten(TreeNode root) {
        LinkedList<TreeNode> stack = new LinkedList<>();
        if(root != null) {
            stack.push(root);
        }

        TreeNode t;
        while(!stack.isEmpty()) {
            t = stack.pop();
            if(t.right != null) {
                stack.push(t.right);
            }
            if(t.left != null) {
                stack.push(t.left);
            }
            if(t != root) {
                root.left = null;
                root.right = t;
                root = root.right;
            }
        }
    }
}
```

##### 3）原地修改法 | O（n）

- **思路**：经过研究二叉树的规律，可得知，转换成链表只需要把左孩子的前驱连接到右孩子，然后剩下的再周而复始接合，所以列入了毁天灭地式的问题，因为没看过题解是真的没想到有这种方法。
- **结论**：时间，0ms，100%，37.8mb，69.48%，由于需要遍历 n 个节点，所以时间复杂度为 O（n），而空间上，由于只在原地修改节点的指针，所以额外空间复杂度为 O（1），这（前驱节点）应该才是面试要考的！

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public void flatten(TreeNode root) {
        TreeNode precursor;
        while(root != null) {
            precursor = findPrecursor(root);
            
            // 存在前驱, 则把前驱的右指针指向根结点右孩子
            if(precursor != null) {
                precursor.right = root.right;
                root.right = root.left;
                root.left = null;
                root = root.right;
            }
            // 前驱不存在, 说明肯定没有左孩子
            else {
                root = root.right;
            }
        }
    }
    
    // 查找前驱节点
    private TreeNode findPrecursor(TreeNode root) {
        TreeNode precursor = root.left;
        if(precursor != null) {
            while(precursor.right != null) {
                precursor = precursor.right;
            }
        }
        return precursor;
    }
}
```

#### 116. 填充每个节点的下一个右侧节点指针 | medium

##### 1）深度优先搜索 + 递归 | O（ [log n + 1] *  log n / 2）

- **思路**：见代码注释。
- **结论**：时间，0 ms，100%，空间，41.6 mb，45.41%，时间上，由于 1 级节点最多会被访问 1 次，2 级最多会被访问 2 次，3 级 3 次，log n 级就 log n 次，所以整体时间复杂度为 O（1 + 2 + ... + log n）= [log n + 1] *  log n / 2，空间上，由于完美二叉树的递归栈深度最大为 log n，所以额外空间复杂度为 O（log n）。 

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public Node left;
    public Node right;
    public Node next;

    public Node() {}
    
    public Node(int _val) {
        val = _val;
    }

    public Node(int _val, Node _left, Node _right, Node _next) {
        val = _val;
        left = _left;
        right = _right;
        next = _next;
    }
};
*/
class Solution {
    public Node connect(Node root) {
        if(root == null) {
            return root;
        }

        // 先连接好根的左和右
        Node left = root.left, right = root.right;
        while(left != null && right != null) {
            left.next = right;
            left = left.right;
            right = right.left;
        }

        // 再交给左、右子树自己处理各自的左和右
        connect(root.left);
        connect(root.right);
        return root;
    }
}
```

##### 2）广度优先搜索 + 迭代 + 队列 | O（n）

- **思路**：层序遍历的套路，不过这里由于不用按层号分类，所以可以通过队列的大小，来获取同一层的节点，而不用一个 map 来标记层号。
- **结论**：时间，2 ms，68.80%，空间，41.9 mb，14.18%，时间上，由于需要遍历整棵二叉树的节点，所以时间复杂度为 O（n），空间上，由于使用了一张最多 2^[log n - 1] = O（n/2）长的队列，所以额外空间复杂度为 O（n/2）。

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public Node left;
    public Node right;
    public Node next;

    public Node() {}
    
    public Node(int _val) {
        val = _val;
    }

    public Node(int _val, Node _left, Node _right, Node _next) {
        val = _val;
        left = _left;
        right = _right;
        next = _next;
    }
};
*/
class Solution {
    public Node connect(Node root) {
        if(root == null) {
            return null;
        }
        
        Queue<Node> queue = new LinkedList<>();
        queue.offer(root);

        int size;
        Node node, pre;
        while(!queue.isEmpty()) {
            size = queue.size();
            pre = null;

            for(int i = 0; i < size; i++) {
                node = queue.poll();
                if(pre == null) {
                    pre = node;
                } else {
                    pre.next = node;
                    pre = node;
                }
                
                // 判断左孩子，二叉树通用
                if(node.left != null) {                   
                    queue.offer(node.left);
                }
                // 判断右孩子，二叉树通用
                if(node.right != null) {
                    queue.offer(node.right);
                }
            }
        }

        return root;
    }
}
```

##### 3）广度优先搜索 + 迭代 + 链表 | O（2 * n）

- **思路**：在 2 的基础上，通过遍历上层组装好的链表，来省去用队列来辅助遍历，uphead 代表上层的链表，dummy 代表下层链表的哑节点，然后 dummpy 不断连接 uphead 的左右孩子，当上层链表遍历完毕，则继续移动到下一层链表，周而复始。
- **结论**：时间，0 ms，100%，空间，41.4 mb，73.66%，时间上，由于每个节点最多被访问 2 次，所以整体时间复杂度为 O（2 * n），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public Node left;
    public Node right;
    public Node next;

    public Node() {}
    
    public Node(int _val) {
        val = _val;
    }

    public Node(int _val, Node _left, Node _right, Node _next) {
        val = _val;
        left = _left;
        right = _right;
        next = _next;
    }
};
*/
class Solution {
    public Node connect(Node root) {
        if(root == null) {
            return null;
        }

        Node upHead = root, dummy, pre;
        while(upHead != null) {
            // 构造哑节点
            pre = dummy = new Node();

            // 开始连接
            while(upHead != null) {
                // 有左
                if(upHead.left != null) {
                    pre.next = upHead.left;
                    pre = pre.next;
                }
                // 有右
                if(upHead.right != null) {
                    pre.next = upHead.right;
                    pre = pre.next;
                }

                // 当前upHead的下层链表设置完毕, 则移动上层upHead
                upHead = upHead.next;
            }

            // 上层链表走到尽头了, 则继续移动到下一层
            upHead = dummy.next;
        }

        return root;
    }
}
```

#### 117. 填充每个节点的下一个右侧节点指针 II | medium

由于这里是非满二叉树，可能会出现当前层，不知道下层的孩子的分布，导致出现选错左右的情况，所以 dfs 实现起来比较麻烦，需要分各种情况讨论，代码与 116 题的不统一，且性能来得也没有 bfs 的高，因此，这里就就不做对应的实现了。

##### 1）广度优先搜索 + 迭代 + 队列 | O（n）

见《116. 填充每个节点的下一个右侧节点指针》的解法 2。

##### 2）广度优先搜索 + 迭代 + 链表 | O（2 * n）

见《116. 填充每个节点的下一个右侧节点指针》的解法 3。

#### 124. 二叉树中的最大路径和 | 树形 dp | hard

##### 1）树形 dp 递归 | O（n）

- **思路**：
  1. 通过根节点不断向下收集左孩子和右孩子的信息，来实现树形 dp。
  2. 根据题意是求最大路径和，而最大路径和有三个方向：左边+中间、右边+中间、左边+中间+右边，因此设计的递归返回的数据结构有：左边+中间最大和 lmax、右边+中间最大和 rmax、当前的最大路径和 max，来用于给上层组装它自己的最大路径和，其中任意一层 root 可以分为 4 种情况：
     1. root 既没有左孩子也没有右孩子：则直接用当前值作为 dp 信息 返回给上一层。
     2. root 既有左孩子又有右孩子：则先获取左右孩子的 dp 信息，来辅助构建自己的 dp 信息：
        - 当前 lmax = Max{当前值，当前值+左孩子的lmax，当前值+左孩子的rmax}。
        - 当前 rmax = Max{当前值，当前值+右孩子的lmax，当前值+右孩子的rmax}。
        - 当前 max = Max{当前值，当期 lmax，当前 rmax， 当前 lmax +当前 rmax - 当前值，左孩子的 max，右孩子的 max}。
     3. root 只有左孩子没有右孩子：则只获取左孩子的 dp 信息，来辅助构建自己的 dp 信息：
        - 当前 lmax = Max{当前值，当前值+左孩子的lmax，当前值+左孩子的rmax}。
        - 当前 rmax = Max{当前值}。
        - 当前 max = Max{当前值，当期 lmax，当前 rmax， 当前 lmax +当前 rmax - 当前值，左孩子的 max}。
     4. root 没有左孩子只有右孩子：则只获取右孩子的 dp 信息，来辅助构建自己的 dp 信息：
        - 当前 lmax = Max{当前值，当前值+右孩子的lmax，当前值+右孩子的rmax}。
        - 当前 rmax = Max{当前值}。
        - 当前 max = Max{当前值，当期 lmax，当前 rmax， 当前 lmax +当前 rmax - 当前值，右孩子的 max}。
- **结论**：时间，1ms，46.45%，空间，40.7mb，5.08%，虽然效率不是很高，但胜在稳，直接使用树形 dp 的套路来思考又快又准！

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {

    class NodeInfo {
        int lmax;// 左边含根结点最大路径和
        int rmax;// 右边含根结点最大路径和
        int max;// 根结点开始的树的最大路径和

        NodeInfo(int val) {
            this.lmax = val;
            this.rmax = val;
            this.max = val;
        }
    }

    public int maxPathSum(TreeNode root) {
        if(root == null) {
            return 0;
        }
        return f(root).max;
    }

    // f代表获取以root作为根结点的NodeInfo信息, 要保证root不为空!
    private NodeInfo f(TreeNode root) {
        NodeInfo curInfo = new NodeInfo(root.val);

        if(root.left != null && root.right != null) {
            NodeInfo lInfo = f(root.left);
            NodeInfo rInfo = f(root.right);

            // 左边含根结点最大路径和
            curInfo.lmax = Math.max(
                root.val,
                Math.max(root.val + lInfo.lmax, root.val + lInfo.rmax)
            );

            // 右边含根结点最大路径和
            curInfo.rmax = Math.max(
                root.val, 
                Math.max(root.val + rInfo.lmax, root.val + rInfo.rmax)
            );

            // 根结点开始的树的最大路径和
            curInfo.max = Math.max(
                // 中间, 左边最大, 右边最大
                Math.max(root.val, Math.max(lInfo.max, rInfo.max)),
                Math.max(
                    // 左边中间, 右边中间
                    Math.max(curInfo.lmax, curInfo.rmax),
                    // 左边中间 + 右边中间 - 中间
                    curInfo.lmax + curInfo.rmax - root.val
                )
            );
        } else if(root.left != null) {
            NodeInfo lInfo = f(root.left);

            // 左边含根结点最大路径和
            curInfo.lmax = Math.max(
                root.val,
                Math.max(root.val + lInfo.lmax, root.val + lInfo.rmax)
            );

            // 右边含根结点最大路径和
            curInfo.rmax = root.val;

            // 根结点开始的树的最大路径和
            curInfo.max = Math.max(
                // 中间, 左边最大
                Math.max(root.val, lInfo.max),
                Math.max(
                    // 左边中间, 右边中间
                    Math.max(curInfo.lmax, curInfo.rmax),
                    // 左边中间 + 右边中间 - 中间
                    curInfo.lmax + curInfo.rmax - root.val
                )
            );
        } else if(root.right != null) {
            NodeInfo rInfo = f(root.right);

            // 左边含根结点最大路径和
            curInfo.lmax = root.val;

            // 右边含根结点最大路径和
            curInfo.rmax = Math.max(
                root.val, 
                Math.max(root.val + rInfo.lmax, root.val + rInfo.rmax)
            );

            // 根结点开始的树的最大路径和
            curInfo.max = Math.max(
                // 中间, 左边最大, 右边最大
                Math.max(root.val, rInfo.max),
                Math.max(
                    // 左边中间, 右边中间
                    Math.max(curInfo.lmax, curInfo.rmax),
                    // 左边中间 + 右边中间 - 中间
                    curInfo.lmax + curInfo.rmax - root.val
                )
            );
        }

        return curInfo;
    }
}
```

#### 138. 复制带随机指针的链表 | medium

##### 1）复制节点 + 改变 random 指针 + 脱离原链表 | O（3 * n）

- **思路**：
  1. 先遍历一次，分别为每个节点都复制一个到 next 指针上，此时，整个链表变为 2 * n 的长度。
  2. 再遍历一次，根据克隆节点的 random 指针，定位到随机到的原节点，通过再往后 next 一下，即可得到随机到的新克隆节点，然后修改本身这个克隆节点的  random 指针。
  3. 最后再遍历一次，脱离克隆节点出原链表，得到新链表返回即可。
- **结论**：时间，0 ms，100%，空间，41.3 mb，5.11%，时间上，由于需要遍历 3 次链表，所以时间复杂度为 O（3 * n），空间上，由于只是用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
/*
// Definition for a Node.
class Node {
    int val;
    Node next;
    Node random;

    public Node(int val) {
        this.val = val;
        this.next = null;
        this.random = null;
    }
}
*/

class Solution {
    public Node copyRandomList(Node head) {
        if(head == null) {
            return head;
        }

        // 复制一份链表
        Node tmp = head, newNode;
        while(tmp != null) {
            newNode = new Node(tmp.val);
            newNode.next = tmp.next;
            newNode.random = tmp.random;

            tmp.next = newNode;
            tmp = tmp.next.next;
        }

        // 改变复制链表的随机指针
        tmp = head;
        while(tmp != null) {
            newNode = tmp.next;

            // 更新复制节点的random指针
            if(newNode.random != null) {
                newNode.random = newNode.random.next;
            }

            tmp = tmp.next.next;
        }

        // 脱离原链表
        Node newHead = (tmp = head).next;
        while(tmp != null) {
            newNode = tmp.next;
            tmp.next = tmp.next.next;

            // 更新复制节点的next指针
            if(newNode.next != null) {
                newNode.next = newNode.next.next;
            }
  
            tmp = tmp.next;
        }

        return newHead;
    }
}
```

#### 226. 翻转二叉树 | easy

##### 1）深度优先搜索 | O（n）

- **思路**：一路递归到最左或者最右节点，交换后返回根节点，作为上一层翻转后的结果设置到左或者右节点即可。
- **结论**：时间，0ms，100%，空间，38.8mb，5.12%，效率非常高了，就这样吧~

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public TreeNode invertTree(TreeNode root) {
        if(root == null) {
            return null;
        }
        
        TreeNode left = root.left;
        TreeNode right = root.right;

        root.right = invertTree(left);
        root.left = invertTree(right);

        return root;
    }
}
```

##### 2）宽度优先搜索 | O（n）

- **思路**：看到宽度优先搜索首先想到队列，出队则处理左右孩子翻转，然后再把左、右节点入队，周而复始即可。
- **结论**：时间，0ms，100%，空间，39ms，5.12%，效率非常高了，就这样吧~

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public TreeNode invertTree(TreeNode root) {
        if(root == null) {
            return null;
        }
        
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        TreeNode node, left, right;
        while(!queue.isEmpty()) {
            node = queue.poll();
            if(node == null) {
                continue;
            }

            left = node.left;
            right = node.right;

            node.left = right;
            node.right = left;

            queue.offer(left);
            queue.offer(right);
        }

        return root;
    }
}
```

#### 236. 二叉树的最近公共祖先 | medium

##### 1）迭代法 | O（n + p + q）

- **思路**：
  1. 先使用任意遍历，建立好 fatherMap，代表所有节点的父结点集合。
  2. 然后把求二叉树最低公共祖先的问题，转换为求两单链表的相交节点问题，即利用 fatherMap 先装好 p 到根节点一路上所经过的所有节点在 HashSet 中，然后在利用 father 往上遍历 q 的过程中，判断是否有节点出现在提前装好的 HashSet 中，其中第一个出现的节点就是它们的最近公共祖先。
- **结论**：时间，10ms，16.77%，空间，42.1mb，5.00%，时间上，遍历二叉树花费了 O（n），往上遍历 p 花费了 O（p），往上遍历 q 花费了 O（q），所以总共的时间复杂度为 O（n + p + q），空间上，遍历二叉树的递归深度为 O（logn），而且还是用了一个 fatherMap O（n）和一个 HashSet O（p），所以总共的额外空间复杂度为 O（logn + n + p）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(root == null || p == null || q == null) {
            return null;
        }

        Map<TreeNode, TreeNode> fatherMap = new HashMap<>();
        fatherMap.put(root, root);
        preOrderTravel(root, fatherMap);

        TreeNode node;
        Set<TreeNode> set = new HashSet<>();
        while(p != (node = fatherMap.get(p))) {
            set.add(p);
            p = node;
        }
        set.add(p);

        while(q != (node = fatherMap.get(q))) {
            if(set.contains(q)) {
                return q;
            }
            q = node;
        }
        if(set.contains(q)) {
            return q;
        }

        return null;
    }

    private void preOrderTravel(TreeNode root, Map<TreeNode, TreeNode> fatherMap) {
        if(root == null) {
            return;
        }

        fatherMap.put(root.left, root);
        fatherMap.put(root.right, root);

        preOrderTravel(root.left, fatherMap);
        preOrderTravel(root.right, fatherMap);
    }
}
```

##### 2）递归法 | O（n）

- **思路**：
  1. 分析二叉树任意节点的最低公共祖先，一共有 3 种情况：
     1. p 是 q 的最低公共祖先，此时 p 的父结点调用递归函数，一边返回结果 p，而另一边返回结果则是 null，代表没有 p 和 q 节点出现。
     2. q 是 p 的最低公共祖先，此时 q 的父结点调用递归函数，一边返回结果 q，而另一边返回结果则是 null，代表没有 p 和 q 节点出现。
     3. p 不是 p 的最低公共祖先，q 也不是 p 的最低公共祖先，此时公共父节点调用递归函数，一边返回结果 p，而另一边返回结果则是 q。
  2. 可见，递归函数实现为，如果两边的返回结果同时不为 null，则说明本节点为 p 和 q 的最低公共祖先；否则如果左边结果不为 null，右边结果为 null，则返回左边结果；如果右边结果不为 null，左边结果为 null，则返回右边结果。
- **结论**：时间，6ms，100%，空间，43mb，5.00%，时间上，由于只需要遍历一次二叉树，所以时间复杂度为 O（n），空间上，递归最大深度为 O（logn），所以额外空间复杂度为 O（logn）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(root == null) {
            return null;
        }
        if(p == root || q == root) {
            return root;
        }
        
        TreeNode leftLca = lowestCommonAncestor(root.left, p, q);
        TreeNode rightLca = lowestCommonAncestor(root.right, p, q);

        if(leftLca != null && rightLca != null) {
            return root;
        }

        return leftLca != null? leftLca : rightLca;
    }
}
```

#### 297. 二叉树的序列化与反序列化 | hard

##### 1）深度优先搜索 | O（n）& O（n）

- **思路**：
  1. 序列化过程：深度优先搜索首先想到的是先序遍历，在遍历过程中搜集 val+","，如果碰到 null 则序列化为 "null" 即可。
  2. 反序列化过程：由于序列化过程是用栈实现的，所以反序列过程也用栈，也就是递归实现，但要注意的点是不能使用普通类型的 i 表示当前字符串数组读取的进度，因为读取左节点字符串回来后，并不知道读到哪里了，所以需要用一个对象比如题目给出的 TreeNode，用 TreeNode.val 来跟踪字符串读取的进度即可。
- **结论**：时间，6ms，98.21%，空间，43mb，5.02%，时间上，序列化和反序列化都只需要遍历一次二叉树的所有节点即可，所以时间复杂度为 O（n），空间上，额外空间复杂度取决于递归深度+字符串数组，为 O（logn + n），不过再尝试一下广度优先搜索方式的实现看看。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Codec {

    // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        sf(root, sb);
        return sb.toString().substring(0, sb.length()-1);
    }

    private void sf(TreeNode root, StringBuilder sb) {
        if(root == null) {
            sappend(sb, "null");
            return;
        }
        
        sappend(sb, String.valueOf(root.val));
        sf(root.left, sb);
        sf(root.right, sb);
    }

    private void sappend(StringBuilder sb, String value) {
        sb.append(value).append(",");
    }

    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {
        if(data == null || data.length() == 0) {
            return null;
        }
        return df(data.split(","), new TreeNode(0));
    }

    private TreeNode df(String[] strs, TreeNode c) {
        if(c.val >= strs.length) {
            return null;
        }
        if("null".equals(strs[c.val])) {
            return null;
        }

        // 反序列化根节点
        TreeNode root = new TreeNode(Integer.valueOf(strs[c.val]));

        // 反序列化左孩子
        c.val++;
        root.left = df(strs, c);

        // 反序列化右孩子
        c.val++;
        root.right = df(strs, c);

        // 返回根节点
        return root;
    }
}

// Your Codec object will be instantiated and called as such:
// Codec ser = new Codec();
// Codec deser = new Codec();
// TreeNode ans = deser.deserialize(ser.serialize(root));
```

##### 2）宽度优先搜索 | O（n）& O（n）

- **思路**：
  1. 序列化过程：宽度优先搜索首先想到的是队列与层次遍历，在遍历过程中搜集 val+","，如果碰到 null 则序列化为 "null" 即可。
  2. 反序列化过程：由于序列化过程是用队列实现的，所以反序列过程也用队列，也就是 while 循环，不过队列存放的是根节点，在循环遍历序列化字符串过程中，先从队列取出当前要处理的根节点（必不为 null，因为是只有不为 null 的节点才加入队列的），然后字符串数组指针右移动处理左孩子和右孩子，同样如果碰到 null 则不做任何处理，否则处理完后把它们又当做根节点加入队列，周而复始。
- **结论**：时间，14ms，66.76%，空间，42.6mb，5.02%，时间上，序列化和反序列化都只需要遍历一次二叉树的所有节点即可，所以时间复杂度为 O（n），空间上，由于序列化和反序列化都使用了一个队列和一个字符串数组，额外空间复杂度为 O（2n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Codec {

    // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        
        StringBuilder sb = new StringBuilder();
        while(!queue.isEmpty()) {
            root = queue.poll();
            sappend(sb, root);
            if(root != null) {
                queue.offer(root.left);
                queue.offer(root.right);
            }
        }

        return sb.toString().substring(0, sb.length()-1);
    }

    private void sappend(StringBuilder sb, TreeNode root) {
        if(root == null) {
            sb.append("null").append(",");
        } else {
            sb.append(root.val).append(",");
        }
    }

    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {
        if(data == null || data.length() == 0) {
            return null;
        }

        String[] strs = data.split(",");
        if("null".equals(strs[0])) {
            return null;
        }

        TreeNode root = new TreeNode(Integer.valueOf(strs[0]));
        Queue<TreeNode> fatherQueue = new LinkedList<>();
        fatherQueue.offer(root);
        
        int i = 0;
        TreeNode node;
        while(i < strs.length) {
            node = fatherQueue.poll();

            // 左孩子
            i++;
            if(i < strs.length && !"null".equals(strs[i])) {
                node.left = new TreeNode(Integer.valueOf(strs[i]));
                fatherQueue.offer(node.left);
            }

            // 右孩子
            i++;
            if(i < strs.length && !"null".equals(strs[i])) {
                node.right = new TreeNode(Integer.valueOf(strs[i]));
                fatherQueue.offer(node.right);
            }
        }

        return root;
    }
}

// Your Codec object will be instantiated and called as such:
// Codec ser = new Codec();
// Codec deser = new Codec();
// TreeNode ans = deser.deserialize(ser.serialize(root));
```

#### 337. 打家劫舍3 | 树形 dp | medium

##### 1）暴力递归 | > O（2 * n）

- **思路**：
  1. 设计一个 Info 结构，用于跟左右孩子要信息：一个是 isRob，代表孩子节点是否偷了钱，一个是 total，代表孩子节点总共偷了多少钱（偷了钱时包括自身）。
  2. 设计一个 f（root，canRob）函数，代表在前一个节点偷或者没有（当前节点不可以偷或者可以偷）的前提下，所能获得的左右孩子信息。
  3. f 函数实现一共 3 种情况：
     1. 如果 root 当前节点为 null，说明为叶子节点，则返回 0 代表的 Info 信息。
     2. 否则，如果当前节点不可以偷，则获取左右孩子可以偷的信息，然后组装返回即可，因为当前节点是不可以偷的，无需比较其他的了。
     3. 如果当前节点可以偷，此时又可以分为两种情况，即当前偷和当前不偷，取当前偷+左右孩子不偷，和当前不偷+左右孩子偷的最大信息返回即可。
- **结论**：执行超时，时间上，由于最坏情况下，当前节点可以偷需要分为偷和不偷分别递归执行，此时至少需要花费 O（2 * n），空间上，额外空间复杂度取决于递归深度，为 O（2 * n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {

    class Info {
        boolean isRob;// 当前是否偷了钱
        int total;// 当前总共偷了多少钱, 包括自己
        Info(boolean isRob, int total) {
            this.isRob = isRob;
            this.total = total;
        }
    }

    public int rob(TreeNode root) {
        return getMaxInfo(f(root, true), f(root, false)).total;
    }

    private Info getMaxInfo(Info rob1, Info rob2) {
        return rob1.total > rob2.total? rob1 : rob2;
    }

    // f代表在前一个节点偷或者没偷所能获得的左右孩子信息
    private Info f(TreeNode root, boolean canRob) {
        if(root == null) {
            return new Info(false, 0);
        }

        // 前一个偷了, 当前不能偷, 左右孩子全都要
        if(!canRob) {
            return new Info(false, f(root.left, true).total + f(root.right, true).total);
        } else {
            return getMaxInfo(
                // 前一个没偷, 则当前可以偷
                getCurRobInfo(root, f(root.left, false), f(root.right, false)),
                // 前一个没偷, 当前也可以不偷
                new Info(false, f(root.left, true).total + f(root.right, true).total)
            );
        }
    }

    private Info getCurRobInfo(TreeNode root, Info linfo, Info rinfo) {
        // 前一个没偷, 则还需要取舍左右孩子
        // 左右孩子都偷了
        int tmp;
        if(linfo.isRob && rinfo.isRob) {
            // 不偷左, 不偷右, 偷当前
            if(root.val > (tmp = linfo.total + rinfo.total)) {
                return new Info(true, root.val);
            } 
            // 偷左, 偷右, 不偷当前
            else {
                return new Info(false, tmp);
            }
        } 
        // 左孩子偷了, 右孩子没偷
        else if(linfo.isRob) {
            // 不偷左, 偷右, 偷当前
            if((tmp = root.val + rinfo.total) > linfo.total) {
                return new Info(true, tmp);
            } 
            // 偷左, 不偷右, 不偷当前
            else {
                return new Info(false, linfo.total);
            }
        }
        // 右孩子偷了, 左孩子没偷
        else if(rinfo.isRob) {
            // 偷左, 不偷右, 偷当前
            if((tmp = root.val + linfo.total) > rinfo.total) {
                return new Info(true, tmp);
            } 
            // 不偷左, 偷右, 不偷当前
            else {
                return new Info(false, rinfo.total);
            }
        } 
        // 左右孩子都没有偷
        else {
            // 偷左, 偷右, 偷当前
            return new Info(true, root.val + linfo.total + rinfo.total);
        }
    }
}
```

##### 2）记忆化搜索 | O（2 * n）

- **思路**：在暴力递归的基础上，增加了 trueDp 和 falseDp 哈希表缓存，如果缓存中存在，则从缓存中获取，否则先把结果设置到缓存中再返回。
- **结论**：执行超时，时间上，由于增加了缓存，访问过一遍的参数将不再重复处理，所以时间复杂度取暴力递归的下限 O（2 * n），空间上，额外空间复杂度取决于递归深度+两张 dp 缓存哈希表，所以为 O（4 * n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {

    class Info {
        boolean isRob;// 当前是否偷了钱
        int total;// 当前总共偷了多少钱, 包括自己
        Info(boolean isRob, int total) {
            this.isRob = isRob;
            this.total = total;
        }
    }

    private Map<TreeNode, Info> trueDp;
    private Map<TreeNode, Info> falseDp;

    public int rob(TreeNode root) {
        trueDp = new HashMap<>();
        falseDp = new HashMap<>();

        Info nullInfo = new Info(false, 0);
        trueDp.put(null, nullInfo);
        falseDp.put(null, nullInfo);

        return getMaxInfo(f(root, true), f(root, false)).total;
    }

    private Info getMaxInfo(Info rob1, Info rob2) {
        return rob1.total > rob2.total? rob1 : rob2;
    }

    // f代表在前一个节点偷或者没偷所能获得的左右孩子信息
    private Info f(TreeNode root, boolean canRob) {
        if(canRob && trueDp.containsKey(root)) {
            return trueDp.get(root);
        }
        if(!canRob && falseDp.containsKey(root)) {
            return falseDp.get(root);
        }

        // 前一个偷了, 当前不能偷, 左右孩子全都要
        Info res;
        if(!canRob) {
            res = new Info(false, f(root.left, true).total + f(root.right, true).total);
            falseDp.put(root, res);
        } else {
            res = getMaxInfo(
                // 前一个没偷, 则当前可以偷
                getCurRobInfo(root, f(root.left, false), f(root.right, false)),
                // 前一个没偷, 当前也可以不偷
                new Info(false, f(root.left, true).total + f(root.right, true).total)
            );
            trueDp.put(root, res);
        }

        return res;
    }

    private Info getCurRobInfo(TreeNode root, Info linfo, Info rinfo) {
        // 前一个没偷, 则还需要取舍左右孩子
        // 左右孩子都偷了
        int tmp;
        if(linfo.isRob && rinfo.isRob) {
            // 不偷左, 不偷右, 偷当前
            if(root.val > (tmp = linfo.total + rinfo.total)) {
                return new Info(true, root.val);
            } 
            // 偷左, 偷右, 不偷当前
            else {
                return new Info(false, tmp);
            }
        } 
        // 左孩子偷了, 右孩子没偷
        else if(linfo.isRob) {
            // 不偷左, 偷右, 偷当前
            if((tmp = root.val + rinfo.total) > linfo.total) {
                return new Info(true, tmp);
            } 
            // 偷左, 不偷右, 不偷当前
            else {
                return new Info(false, linfo.total);
            }
        }
        // 右孩子偷了, 左孩子没偷
        else if(rinfo.isRob) {
            // 偷左, 不偷右, 偷当前
            if((tmp = root.val + linfo.total) > rinfo.total) {
                return new Info(true, tmp);
            } 
            // 不偷左, 偷右, 不偷当前
            else {
                return new Info(false, rinfo.total);
            }
        } 
        // 左右孩子都没有偷
        else {
            // 偷左, 偷右, 偷当前
            return new Info(true, root.val + linfo.total + rinfo.total);
        }
    }
}
```

##### 3）严格表结构优化 | O（n）

- **思路**：在记忆化搜索的基础上，经过研究值依赖关系，发现发现 falseDp 依赖 trueDp.get（left）和 trueDp.get（right），以及 trueDp 依赖 Max { falseDp.get（left）和 falseDp.get（right），trueDp.get（left）和 trueDp.get（right） }，因此，可以只进行一次后序遍历，在其中分别设置 falseDp 和 trueDp 即可。
- **结论**：时间，3ms，22.15%，空间，42.3mb，5.01%，时间上，由于只需要做一次后序遍历，所以时间复杂度为 O（n），空间上，由于使用了两张 dp 哈希表，以及递归深度，所以额外空间复杂度 O（3 * n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {

    class Info {
        boolean isRob;// 当前是否偷了钱
        int total;// 当前总共偷了多少钱, 包括自己
        Info(boolean isRob, int total) {
            this.isRob = isRob;
            this.total = total;
        }
    }

    private Map<TreeNode, Info> trueDp;
    private Map<TreeNode, Info> falseDp;

    public int rob(TreeNode root) {
        trueDp = new HashMap<>();
        falseDp = new HashMap<>();

        Info nullInfo = new Info(false, 0);
        trueDp.put(null, nullInfo);
        falseDp.put(null, nullInfo);

        dfs(root);
        return getMaxInfo(trueDp.get(root), falseDp.get(root)).total;
    }

    private Info getMaxInfo(Info rob1, Info rob2) {
        return rob1.total > rob2.total? rob1 : rob2;
    }

    private void dfs(TreeNode root) {
        if(root == null) {
            return;
        }

        dfs(root.left);
        dfs(root.right);

        // 前一个偷了, 当前不能偷, 左右孩子全都要
        falseDp.put(
            root, 
            new Info(false, trueDp.get(root.left).total + trueDp.get(root.right).total)
        );
        trueDp.put(root, getMaxInfo(
            // 前一个没偷, 则当前可以偷
            getCurRobInfo(root, falseDp.get(root.left), falseDp.get(root.right)),
            // 前一个没偷, 当前也可以不偷
            new Info(false, trueDp.get(root.left).total + trueDp.get(root.right).total)
        ));
    }

    private Info getCurRobInfo(TreeNode root, Info linfo, Info rinfo) {
        // 前一个没偷, 则还需要取舍左右孩子
        // 左右孩子都偷了
        int tmp;
        if(linfo.isRob && rinfo.isRob) {
            // 不偷左, 不偷右, 偷当前
            if(root.val > (tmp = linfo.total + rinfo.total)) {
                return new Info(true, root.val);
            } 
            // 偷左, 偷右, 不偷当前
            else {
                return new Info(false, tmp);
            }
        } 
        // 左孩子偷了, 右孩子没偷
        else if(linfo.isRob) {
            // 不偷左, 偷右, 偷当前
            if((tmp = root.val + rinfo.total) > linfo.total) {
                return new Info(true, tmp);
            } 
            // 偷左, 不偷右, 不偷当前
            else {
                return new Info(false, linfo.total);
            }
        }
        // 右孩子偷了, 左孩子没偷
        else if(rinfo.isRob) {
            // 偷左, 不偷右, 偷当前
            if((tmp = root.val + linfo.total) > rinfo.total) {
                return new Info(true, tmp);
            } 
            // 不偷左, 偷右, 不偷当前
            else {
                return new Info(false, rinfo.total);
            }
        } 
        // 左右孩子都没有偷
        else {
            // 偷左, 偷右, 偷当前
            return new Info(true, root.val + linfo.total + rinfo.total);
        }
    }
}
```

##### 4）空间压缩优化 | O（n）

- **思路**：在严格表结构优化的基础上，由于 dp 哈希表存放了所有节点的信息，又分为 trueDp 和 falseDp，所以可以尝试把这两张 dp 哈希表优化成一个 Info[2] 的数组，0 位置存放节点为 false 时的信息，1 位置存放节点为 true 时的信息，构建完成后通过 dfs 返回值返回。
- **结论**：时间，1ms，47.20%，空间，40.7mb，6.85%，时间上，由于只需要执行一次后序遍历，所以时间复杂度为 O（n），空间上，由于递归深度 O（n）是不可以避免的，以及需要一张不断动态申请和释放的 Info[2] 数组，所以额外空间复杂度为 O（n + 2）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {

    class Info {
        boolean isRob;// 当前是否偷了钱
        int total;// 当前总共偷了多少钱, 包括自己
        Info(boolean isRob, int total) {
            this.isRob = isRob;
            this.total = total;
        }
    }

    private Info NULL_INFO = new Info(false, 0);
    private Info[] NUll_ARR = new Info[] {NULL_INFO, NULL_INFO};

    public int rob(TreeNode root) {
        Info[] infos = dfs(root);
        return getMaxInfo(infos[0], infos[1]).total;
    }

    private Info getMaxInfo(Info rob1, Info rob2) {
        return rob1.total > rob2.total? rob1 : rob2;
    }

    // 0: false, 1: true
    private Info[] dfs(TreeNode root) {
        if(root == null) {
            return NUll_ARR;
        }

        Info[] linfos = dfs(root.left);
        Info[] rinfos = dfs(root.right);

        // 前一个偷了, 当前不能偷, 左右孩子全都要
        return new Info[] {
            new Info(false, linfos[1].total + rinfos[1].total),
            getMaxInfo(
                // 前一个没偷, 则当前可以偷
                getCurRobInfo(root, linfos[0], rinfos[0]),
                // 前一个没偷, 当前也可以不偷
                new Info(false, linfos[1].total + rinfos[1].total)
            )
        };
    }

    private Info getCurRobInfo(TreeNode root, Info linfo, Info rinfo) {
        // 前一个没偷, 则还需要取舍左右孩子
        // 左右孩子都偷了
        int tmp;
        if(linfo.isRob && rinfo.isRob) {
            // 不偷左, 不偷右, 偷当前
            if(root.val > (tmp = linfo.total + rinfo.total)) {
                return new Info(true, root.val);
            } 
            // 偷左, 偷右, 不偷当前
            else {
                return new Info(false, tmp);
            }
        } 
        // 左孩子偷了, 右孩子没偷
        else if(linfo.isRob) {
            // 不偷左, 偷右, 偷当前
            if((tmp = root.val + rinfo.total) > linfo.total) {
                return new Info(true, tmp);
            } 
            // 偷左, 不偷右, 不偷当前
            else {
                return new Info(false, linfo.total);
            }
        }
        // 右孩子偷了, 左孩子没偷
        else if(rinfo.isRob) {
            // 偷左, 不偷右, 偷当前
            if((tmp = root.val + linfo.total) > rinfo.total) {
                return new Info(true, tmp);
            } 
            // 不偷左, 偷右, 不偷当前
            else {
                return new Info(false, rinfo.total);
            }
        } 
        // 左右孩子都没有偷
        else {
            // 偷左, 偷右, 偷当前
            return new Info(true, root.val + linfo.total + rinfo.total);
        }
    }
}
```

#### 437. 路径总和3 | 树形 dp | medium

##### 1）暴力递归 | O（n）

- **思路**：
  1. 设计一个 Info 数据结构： count，代表统计当前节点#独立 val 路径、左 + val 路径、右 + val 路径中，满足等于 target 的数量；sumList，代表存放当前独立 val 路径、左 + val 路径、右 + val 路径三种所有的路径。
  2. 然后设计一个 f（root）函数，可以要取 root 节点的 Info 信息，包括 count 和 sumList。
  3. f 函数的实现，需要先要取 root.left 左孩子的 Info 信息，然后获取 root.right右孩子的 Info 信息，再使用   lInfo.count + rInfo.count 生成当前的 curInfo，接着还需要统计 3 个部分的信息再返回 curInfo：
     1. 把当前独立 val 加入 sumList，然后统计是否等于 target。
     2. 遍历 lInfo.sumList，把当前 val 累加后再加入 sumList，然后统计其中等于 target 的数量。
     3. 遍历 rInfo.sumList，把当前 val 累加后再加入 sumList，然后统计其中等于 target 的数量。
- **结论**：时间，26ms，39.16%，空间，41.7mb，5.04%，时间上，由于需要遍历整棵树的所有节点，所以时间复杂度为 O（n），空间上，最坏情况下，完全二叉树时，i 层节点的 sumList 会存放 2^i 个值，也就是 logn 层的 root节点的 sumList 长度为 O（2^logn），递归深度最大为 O（logn），因此额外空间复杂度为 O（2^logn + logn）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {

    class Info {
        int count;
        List<Integer> sumList = new ArrayList<>();

        Info(int count) {
            this.count = count;
        } 
    }

    // 不能跨左根右, 但可不限于一个方向, 如根+左+右...
    public int pathSum(TreeNode root, int targetSum) {
        return f(root, targetSum).count;
    }

    private Info f(TreeNode root, int targetSum) {
        if(root == null) {
            return new Info(0);
        }

        Info lInfo = f(root.left, targetSum);
        Info rInfo = f(root.right, targetSum);
        Info curInfo = new Info(lInfo.count + rInfo.count);

        // 统计独立val路径, 是否等于target
        curInfo.sumList.add(root.val);
        curInfo.count += root.val == targetSum? 1 : 0;

        // 统计左+val路径中, 满足等于target的数量
        if(!lInfo.sumList.isEmpty()) {
            for(Integer sum : lInfo.sumList) {
                sum += root.val;
                curInfo.sumList.add(sum);
                if(sum == targetSum) {
                    curInfo.count++;
                }
            }
        }

        // 统计右+val路径中, 满足等于target的数量
        if(!rInfo.sumList.isEmpty()) {
            for(Integer sum : rInfo.sumList) {
                sum += root.val;
                curInfo.sumList.add(sum);
                if(sum == targetSum) {
                    curInfo.count++;
                }
            }
        }

        return curInfo;
    }
}
```

#### 538. 把二叉搜索树转换为累加树 | medium

##### 1）深度优先搜索 - 局部变量法 | O（n）

- **思路**：
  1. 设计一个 f（root，rsum）函数，代表右孩子累加 rsum，当前节点累加右孩子 + rsum，左孩子累加右孩子 + rsum + 当前节点。
  2. f 函数分为 3 个累加部分，首先累加右孩子，然后把结果累加到当前节点上，最后再把当前节点累加后的结果累加到左孩子上再返回。
- **结论**：时间，0ms，100%，空间，41.3mb，7.99%，时间上，由于需要遍历整棵树所有节点，所以时间复杂度为 O（n），空间上，由于递归深度最大为 logn，所以额外空间复杂度为 O（logn）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {

    public TreeNode convertBST(TreeNode root) {
        f(root, 0);
        return root;
    }

    // f代表右孩子累加 rsum，当前节点累加右孩子 + rsum，左孩子累加右孩子 + rsum + 当前节点
    private int f(TreeNode root, int rsum) {
        // 当前为叶子节点, 没当前、没左、没右, 累加和为rsum自己
        if(root == null) {
            return rsum;
        }

        // 当前 + 获取右边累加后的rsum: 包括所有右孩子的左孩子们的累加!
        root.val += f(root.right, rsum);

        // rusm累加完左孩子后, 返回给父节点, 因为当前左孩子也会大于等于父节点的!
        return f(root.left, root.val);
    }
}
```

##### 2）深度优先搜索 - 全局变量法 | O（n）

- **思路**：
  1. 与局部变量法的思路一样，也是反中序遍历，即右中左，因此，可以把 sum 累加变量放到全局变量上，然后递归函数调用就不用使用 int 参数和返回 int 值了，递归调法也类似，即先累加右、再右累加中、再右+中累加左再返回。
  2. 不过，虽然左边统统累加了右边，但如果左边负得过于小，而右边正得也不大，那么最后形成的也不一定是倒叙的二叉搜索树，所以两者没什么关联~
- **结论**：时间，0ms，100%，空间，41.5mb，6.15%，时间上，由于需要遍历整棵树所有节点，所以时间复杂度为 O（n），空间上，由于递归深度最大为 logn，所以额外空间复杂度为 O（logn）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {

    private int sum = 0;

    public TreeNode convertBST(TreeNode root) {
        if(root == null) {
            return null;
        }

        // 先累加右孩子
        convertBST(root.right);

        // 再累加当前节点
        sum += root.val;
        root.val = sum;

        // 最后累加左孩子
        convertBST(root.left);

        return root;
    }
}
```

#### 543. 二叉树的直径 | 树形 dp | easy

##### 1）暴力递归1 | O（n^2）

- **思路**：先求出每个节点的左高度和右高度，根据题意【左高度 + 右高度】并不是它的直径，而答案可能藏在孩子节点里，所以还要递归找它的孩子节点，取最大的【左高度 + 右高度】然后返回即可。
- **结论**：时间，15ms，9.45%，空间，41.2mb，5.07%，时间上，由于当前节点找左高度和找右高度需要花费 O（n）的时间，而比较每个节点的【左高度+右高度】的最大值，又需要遍历整棵树的所有节点，所以时间复杂度为 O（n^2），空间上，寻找高度递归深度为 O（logn），遍历整棵树的最大深度为 O（logn），所以额外空间复杂度为 O（logn * logn）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public int diameterOfBinaryTree(TreeNode root) {
        if(root == null) {
            return 0;
        }

        return Math.max(
            getHight(root.left) + getHight(root.right), 
            Math.max(diameterOfBinaryTree(root.left), diameterOfBinaryTree(root.right))
        );
    }

    private int getHight(TreeNode root) {
        if(root == null) {
            return 0;
        }

        int lh = getHight(root.left);
        int rh = getHight(root.right);

        return lh > rh? lh + 1 : rh + 1;
    }
}
```

##### 2）记忆化搜索1 | O（2 * n）

- **思路**：在暴力递归的基础上，对高度查找的函数增加了一张 dp 哈希表缓存，如果缓存中存在，则从缓存中获取，否则先把结果设置到缓存中再返回。
- **结论**：时间，2ms，10.13%，空间，41.2mb，5.07%，时间上，由于高度查找增加了一层缓存，使得每个节点找它们的【左高度和右高度】只需要处理一次，也就是找高度花费 O（n），遍历树花费 O（n），所以总共的时间复杂度为 O（2 * n），空间上，由于递归深度没改变过，等于暴力递归的递归深度总和 O（logn * logn），同时还增加了一张 dp 缓存表 O（n），所以额外空间复杂度为 O（logn * logn + n）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public int diameterOfBinaryTree(TreeNode root) {
        if(root == null) {
            return 0;
        }

        return Math.max(
            getHight(root.left) + getHight(root.right), 
            Math.max(diameterOfBinaryTree(root.left), diameterOfBinaryTree(root.right))
        );
    }

    private Map<TreeNode, Integer> dp = new HashMap<>();

    private int getHight(TreeNode root) {
        if(dp.containsKey(root)) {
            return dp.get(root);
        }
        if(root == null) {
            dp.put(root, 0);
            return 0;
        }

        int lh = getHight(root.left);
        int rh = getHight(root.right);

        dp.put(root, lh > rh? lh + 1 : rh + 1);
        return dp.get(root);
    }
}
```

##### 3）暴力递归2 | O（n）

- **思路**：
  1. 在记忆化搜索1 的基础上，研究发现，由于每次都是先找左、右高度，然后再判断最大的左右深度和，即使增加了记忆化缓存，也仍然有大量的重复递归行为，所以，可以在每次返回左、右高度前，增加对最大左右深度和的判断，这样可以减少外层 O（n） 的整棵树遍历。
  2. 而由于调用一次高度查找时，对于每个节点只会被遍历 1 次，没有重复的递归行为，所以并不需要记忆化缓存，去掉就好。
- **结论**：时间，0ms，100%，空间，41mb，7.16%，时间上，由于每个节点只会被遍历 1 次，所以时间复杂度为 O（n），空间上，最大递归深度为 O（logn），所以额外空间复杂度为 O（logn）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {

    private int max;

    public int diameterOfBinaryTree(TreeNode root) {
        if(root == null) {
            return 0;
        }
        getHight(root);
        return max;
    }

    private int getHight(TreeNode root) {
        if(root == null) {
            return 0;
        }

        int lh = getHight(root.left);
        int rh = getHight(root.right);
        max = Math.max(max, lh + rh);
        return lh > rh? lh + 1 : rh + 1;
    }
}
```

#### 617. 合并二叉树 | easy

##### 1）深度优先搜索 | O（n）

- **思路**：二叉树遍历变种，这里改的是后序遍历，在左右孩子遍历处理完后，处理当前节点，把左右孩子返回的节点 + 当前累加的值，作为新的节点返回。
- **结论**：时间，0ms，100%，空间，41.5mb，5.24%，时间上，由于是后序遍历，其遍历的最大值取决于两棵树中，节点多的那棵树，记 n = max{n1, n2}，所以时间复杂度为 O（n），空间上，额外空间复杂度取决于递归深度，为 O（logn）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
        if(root1 == null) {
            return root2;
        }
        if(root2 == null) {
            return root1;
        }

        TreeNode lnode = mergeTrees(root1.left, root2.left);
        TreeNode rnode = mergeTrees(root1.right, root2.right);
        return new TreeNode(root1.val + root2.val, lnode, rnode);
    }
}
```

#### 2265. 第 292 场周赛 - 统计值等于子树平均值的节点数 | 树形 dp | medium

##### 1）深度优先搜索 | O（n）

- **思路**：
  1. 建立一个 Info 的数据结构，sum 表示当前树（包括根节点）的节点值总和，count 表示当前树（包括根节点）的节点总个数。
  2. 然后遍历每一层树时，都不断地向下收集子节点地 Info 信息来做决策，即判断平均值是否等于当前根节点的值，是的话则 res + 1。
  3. 最后返回 res 就是最终结果了。
- **结论**：时间，0 ms，100%，空间，41.1 mb，38.35%，时间上，由于需要遍历完 n 个节点，所以时间复杂度为 O（n），空间上，由于树的最大深度位 logn，所以递归栈的额外空间复杂度为 O（logn）。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {

    private class Info {
        int sum;
        int count;

        Info(int sum, int count) {
            this.sum = sum;
            this.count = count;
        }
    }

    private int res = 0;

    public int averageOfSubtree(TreeNode root) {
        if(root == null) {
            return 0;
        }
        
        f(root);
        return res;
    }

    private Info f(TreeNode root) {
        int sum = root.val, count = 1;

        // 左
        if(root.left != null) {
            Info linfo = f(root.left);
            sum += linfo.sum;
            count += linfo.count;
        }

        // 右
        if(root.right != null) {
            Info rinfo = f(root.right);
            sum += rinfo.sum;
            count += rinfo.count;
        }

        // 求平均
        if(sum / count == root.val) {
            res++;
        }

        return new Info(sum, count);
    }
}
```

### 1.3. 数据结构 - 哈希表

#### 1. 两数之和 | easy

##### 1）缓存法 | O（n）

- **思路**：遍历数组，每次都判断哈希表中是否存在和的另一半，存在则返回，不存在则添加到表中，然后继续。
- **优点**：缓存法，一次遍历搞定（1ms，99.36%）。
- **结论**：空间复杂度 O（n），如果仅仅是两数之和，那么这就是最优解，否则如果嵌套两数之和求解，整体复杂度 > O（n）的话，那么用**双向指针**会省很多空间。

```java
import java.util.HashMap;
class Solution {
    public int[] twoSum(int[] nums, int target) {
        if(nums == null || nums.length < 2) {
            return null;
        }
        
        Integer tmp = null;
        HashMap<Integer, Integer> map = new HashMap<>();
        for(int i = 0; i < nums.length; i++) {
            tmp = map.get(target - nums[i]);
            if(tmp != null) {
                return new int[] {i, tmp};    
            }
            map.put(nums[i], i);
        }

        return new int[] {-1, -1};
    }
}
```

#### 36. 有效的数独 | medium

##### 1）模拟 + 哈希表 + 位运算 | O（3 * m * n）

- **思路**：模拟规则判断流程，即判断每行、每列、每个九宫格是否有重复元素，其中这里的哈希表使用了位运算的方式来节省内存。
- **结论**：时间，2 ms，33.59%，空间，41.2 mb，77.06%，时间上，由于需要分三次遍历整个矩阵，所以整体时间复杂度为 O（3 * m * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Info {
    long bitmap;
    Info(long bitmap) {
        this.bitmap = bitmap;
    }
}

class Solution {

    private static final int[][] BLOCK_CENTER_IDXS = new int[][] {
        new int[] {1, 1}, new int[] {1, 4}, new int[] {1, 7},
        new int[] {4, 1}, new int[] {4, 4}, new int[] {4, 7},
        new int[] {7, 1}, new int[] {7, 4}, new int[] {7, 7},
    };

    public boolean isValidSudoku(char[][] board) {
        return verfiyEveryRow(board) && verfiyEveryCol(board) && verfiyBlocks(board);
    }

    // 校验某个格子, 并加入缓存
    private boolean verifyAndMarkVis(char val, Info info) {
        if(val == '.') {
            return true;
        }

        int idx = val - '0';
        if(((info.bitmap >> idx) & 1) == 1) {
            return false;
        }

        info.bitmap |= (1 << idx);
        return true;
    }

    // 校验行是否重复
    private boolean verfiyEveryRow(char[][] board) {
        for(int i = 0; i < board.length; i++) {
            Info info = new Info(0);
            for(int j = 0; j < board[0].length; j++) {
                if(!verifyAndMarkVis(board[i][j], info)) {
                    return false;
                }
            }
        }

        return true;
    }

    // 校验列是否重复
    private boolean verfiyEveryCol(char[][] board) {
        for(int j = 0; j < board[0].length; j++) {
            Info info = new Info(0);
            for(int i = 0; i < board.length; i++) {
                if(!verifyAndMarkVis(board[i][j], info)) {
                    return false;
                }
            }
        }

        return true;
    }

    // 校验九宫格是否重复
    private boolean verfiyBlocks(char[][] board) {
        for(int[] center : BLOCK_CENTER_IDXS) {
            int i = center[0], j = center[1];

            Info info = new Info(0);
            if(
                // 中间
                !verifyAndMarkVis(board[i][j], info)
                // 左上
                || !verifyAndMarkVis(board[i - 1][j - 1], info) 
                // 右上
                || !verifyAndMarkVis(board[i - 1][j + 1], info)
                // 左下
                || !verifyAndMarkVis(board[i + 1][j - 1], info)
                // 右下 
                || !verifyAndMarkVis(board[i + 1][j + 1], info)
                // 上
                || !verifyAndMarkVis(board[i - 1][j], info)
                // 下
                || !verifyAndMarkVis(board[i + 1][j], info)
                // 左
                || !verifyAndMarkVis(board[i][j - 1], info)
                // 右
                || !verifyAndMarkVis(board[i][j + 1], info)
            ) {
                return false;
            }
        }

        return true;
    }
}
```

##### 2）哈希表 + 二维表遍历 | O（m * n）

- **思路**：
  1. rowCnts 表示按统计数字出现的次数，通过 i 来表示行索引。
  2. colCnts 表示按列统计数字出现的次数，通过 j 来表示列索引。
  3. regionCnts 表示按九宫格区域统计数字出现的次数，通过 `[i / 3][j / 3]` 来表示九宫格区域索引。
  4. 最后，根据对应索引，累加结果到对应位置，再判断累加后的结果是否大于 1，如果大于则不符合题目要求，返回 false，否则到最终通过时，则返回 true。
- **结论**：时间，1 ms，99.28%，空间，41.6 mb，18.58%，时间上，由于只需要遍历一次二维表，所以时间复杂度为 O（m * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public boolean isValidSudoku(char[][] board) {
        // board默认9行9列
        int[][] rowCnts = new int[9][9];// 1~9, 从0开始
        int[][] colCnts = new int[9][9];// 1~9, 从0开始
        int[][][] regionCnts = new int[3][3][9];// 1~9, 从0开始

        // 遍历一遍二维表即可
        for(int i = 0; i < 9; i++) {
            for(int j = 0; j < 9; j++) {
                char c = board[i][j];
                if(c == '.') {
                    continue;
                }

                // 1~9, 从0开始
                int idx = c - '0' - 1;
                rowCnts[i][idx] += 1;
                colCnts[j][idx] += 1;
                regionCnts[i / 3][j / 3][idx] += 1;

                // 判断数独是否无效
                if(rowCnts[i][idx] > 1 || colCnts[j][idx] > 1 || regionCnts[i / 3][j / 3][idx] > 1) {
                    return false;
                }
            }
        }

        return true;
    }
}
```

#### 41. 缺失的第一个整数 | hard

##### 1）原数组改造 + 替代哈希表 + 打标 | O（3 * n）

- **思路**：
  1. 由于要求的是 [1, n] 中, 最小的 n，或者其都存在时，则返回 n + 1，所以，可以先对不在 [1, n] 范围内的负数统一更新为 n + 1。
  2. 然后，再对 [1, n] 内的数进行打标，比如 3 应该出现在 2 的位置，此时就需要对 2 的位置进行打标，从而在原数组中完成标记作用，无需额外构造一个哈希表。
  3. 其中，这里打标方式采用原数组打标，最简单的就是打负号了，但由于可能i位置的元素已经被打过标了, 所以为了能够看到打标签前的值，还需要先对其求绝对值，如果已经打过标了，则需继续打了，否则需要。
  4. 最后，遍历找到第一个正数的位置 i，此时该位置就是没有被打标的，返回 i + 1 即可，如果找不到任何没打标的元素，则认为 [1, N] 元素都存在，此时返回 N + 1 作为结果即可。
- **结论**：时间，2 ms，94.75%，空间，97.3 mb，31.64%，时间上，由于需要遍历 3 次数组，所以时间复杂度为 O（3 * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int firstMissingPositive(int[] nums) {
        // 由于要求的是[1,n]中, 最小的n或者n+1
        // 所以, 可以先对不在范围内的负数, 更新为n+1
        int n = nums.length;
        for(int i = 0; i < n; i++) {
            if(nums[i] <= 0) {
                nums[i] = n + 1;
            }
        }

        // 再对[1, n]内的数进行打标, 比如3应该出现在2的位置, 此时就需要对2的位置进行打标
        // 这里打标方式采用元素组打标, 最简单的就是打负号了
        for(int i = 0; i < n; i++) {
            // 由于可能i位置的元素已经被打过标了, 所以为了能够看到打标签前的值, 还需要对其求绝对值
            int val = Math.abs(nums[i]);

            // 如果已经打过标了, 则需继续打了, 否则需要
            if(val >= 1 && val <= n && nums[val - 1] > 0) {
                nums[val - 1] *= -1;
            }
        }

        // 最后, 遍历找到第一个正数的位置i, 此时该位置就是没有被打标的, 返回i+1即可
        int idx = -1;
        for(int i = 0; i < n; i++) {
            if(nums[i] > 0) {
                idx = i;
                break;
            }
        }

        // 如果找不到任何没打标的元素, 则认为[1,N]元素都存在, 此时返回N+1作为结果即可
        return idx == -1? n + 1 : idx + 1;
    }
}
```

##### 2）正确位置置换法 | O（2 * n）

- **思路**：
  1. 从左到右交换 i 值的元素, 放到正确的 i - 1 位置上，其中，由于交换过来的值也可能没放对位置，所以还需要 while 循环，继续判断当前 i 位置的元素是否放对，另外，如果碰到相同的，就不交换当前 i 位置元素了，因为其中有一个放对位置了，另一个就是错的。
  2. 最后，从左到右遍历，如果 i 位置上的元素，值不为 i + 1，那么就返回该 i 位置元素的值 i + 1，如果找不到，则返回 n + 1。
- **结论**：时间，2 ms，94.75%，空间，97.5 mb，17.13%，时间上，由于一共有 n 个位置，每次都至少把一个元素放到对的位置，所以最多需要交换 n 次，时间复杂度为 O（n），而最后的遍历最多也需要花费 O（n），因此整体时间复杂度为 O（2 * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int firstMissingPositive(int[] nums) {
        // 从左到右交换i值的元素, 放到正确的i-1位置上
        int n = nums.length;
        for(int i = 0; i < n; i++) {
            // 相同的就不交换了, 因为其中有一个放对位置了, 另一个就是错的
            while(nums[i] >= 1 && nums[i] <= n && nums[i] != nums[nums[i] - 1]) {
                swap(nums, i, nums[i] - 1);
            }
        }

        // 从左到右遍历, 如果i位置上的元素, 值不为i+1, 那么就返回该i位置元素的值i+1
        for(int i = 0; i < n; i++) {
            if(nums[i] != i + 1) {
                return i + 1;
            }
        }
		
        // 如果找不到, 则返回 n + 1
        return n + 1;
    }

    private void swap(int[] nums, int from, int to) {
        int tmp = nums[to];
        nums[to] = nums[from];
        nums[from] = tmp;
    }
}
```

#### 73. 矩阵置零 | medium

##### 1）哈希表 + 行列标记数组 | O（m * n）

- **思路**：先遍历一次矩阵，使用行、列数组来标记是否出现 0，然后再遍历一次矩阵，根据标记数组来判断，是否需要把当前位置置为 0 即可。
- **结论**：时间，0 ms，100%，空间，43 mb，47.59%，时间上，由于需要遍历两次矩阵，所以时间复杂度为 O（m * n），空间上，由于使用了一张 m 长行数组、n 长列数组，所以额外空间复杂度为 O（m + n）。

```java
class Solution {
    public void setZeroes(int[][] matrix) {
        int m = matrix.length, n = matrix[0].length;
        boolean[] rowHasZeros = new boolean[m];
        boolean[] colHasZeros = new boolean[n];

        // 标记i行、j列是否出现过0
        for(int i = 0; i < m; i++) {
            for(int j = 0; j < n; j++) {
                if(matrix[i][j] == 0) {
                    rowHasZeros[i] = colHasZeros[j] = true;
                }
            }
        }

        // 行上、列上出现过0的, 则更新为0
        for(int i = 0; i < m; i++) {
            for(int j = 0; j < n; j++) {
                if(matrix[i][j] == 0) {
                    continue;
                }
                if(rowHasZeros[i] || colHasZeros[j]) {
                    matrix[i][j] = 0;
                }
            }
        }
    }
}
```

##### 2）哈希表 + 首行首列作为标记数组 +  2 个中间变量 | O（m * n）

- **思路**：
  1. 基于 1 的思路发现，行列标记数组可以复用矩阵的首列、首行空间，但这样就不能直到原来行、列上是否存在过 0，会导致最后首行、首列出现不能全更新为 0 的情况。
  2. 所以还需要两个额外的中间变量作为标记，标记首行、首列之前是否就为 0，是的话则保证最后更新首行、首列全为 0。
- **结论**：时间，0 ms，100%，空间，43.1 mb，24.93%，时间上，由于最多需要遍历两次矩阵，所以时间复杂度为 O（m * n），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public void setZeroes(int[][] matrix) {
        int m = matrix.length, n = matrix[0].length;
        boolean row0HasZeros = false, col0HasZeros = false;

        // 标记第0行、0列是否原来就有0
        for(int i = 0; i < m; i++) {
            if(matrix[i][0] == 0) {
                col0HasZeros = true;
                break;
            }
        }
        for(int j = 0; j < n; j++) {
            if(matrix[0][j] == 0) {
                row0HasZeros = true;
                break;
            }
        }

        // 使用第0行、第0列作为标记数组, 标记i行、j列是否出现过0
        for(int i = 1; i < m; i++) {
            for(int j = 1; j < n; j++) {
                if(matrix[i][j] == 0) {
                    matrix[i][0] = matrix[0][j] = 0;
                }
            }
        }

        // 行上、列上出现过0的, 则更新为0
        for(int i = 1; i < m; i++) {
            for(int j = 1; j < n; j++) {
                if(matrix[i][j] == 0) {
                    continue;
                }
                if(matrix[i][0] == 0 || matrix[0][j] == 0) {
                    matrix[i][j] = 0;
                }
            }
        }

        // 如果第0行、第0列原来就有0, 那么就整行、整列都更新为0, 否则只需要保持上面更新到的某些行、列为0即可
        if(col0HasZeros) {
            for(int i = 0; i < m; i++) {
                matrix[i][0] = 0;
            }
        }
        if(row0HasZeros) {
            for(int j = 0; j < n; j++) {
                matrix[0][j] = 0;
            }
        }
    }
}
```

##### 3）哈希表 + 首行首列作为标记数组 +  1 个中间变量 | O（m * n）

- **思路**：
  1. 基于 2 的思路，由于首行、首列都包含点（0，0），所以还可以省掉一个中间变量 row0HasZeros，改用（0，0）标记首行原本是否有 0。
  2. 但要注意的是，最后根据原始是否有 0 的标记，更新首行、首列全为 0 时，需要先用掉（0，0）的标记，再更新首列全为 0，防止（0，0）标记被更新首列全为 0 时覆盖了。
- **结论**：时间，0 ms，100%，空间，43 mb，38.58%，时间上，由于最多需要遍历两次矩阵，所以时间复杂度为 O（m * n），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public void setZeroes(int[][] matrix) {
        int m = matrix.length, n = matrix[0].length;
        boolean col0HasZeros = false;

        // 标记第0列是否原来就有0
        for(int i = 0; i < m; i++) {
            if(matrix[i][0] == 0) {
                col0HasZeros = true;
                break;
            }
        }

        // 使用第0行、第0列作为标记数组, 标记i行、j列是否出现过0
        // 注意, 由于这里去掉了row0HasZeros, 所以如果第0行任意列出现0, 则(0, 0)位置的标记需要更新为0
        for(int i = 0; i < m; i++) {
            for(int j = 1; j < n; j++) {
                if(matrix[i][j] == 0) {
                    matrix[i][0] = matrix[0][j] = 0;
                }
            }
        }

        // 行上、列上出现过0的, 则更新为0
        for(int i = 1; i < m; i++) {
            for(int j = 1; j < n; j++) {
                if(matrix[i][j] == 0) {
                    continue;
                }
                if(matrix[i][0] == 0 || matrix[0][j] == 0) {
                    matrix[i][j] = 0;
                }
            }
        }

        // 如果第0行、第0列原来就有0, 那么就整行、整列都更新为0, 否则只需要保持上面更新到的某些行、列为0即可
        // 注意, 由于这里去掉了row0HasZeros, 所以需要根据(0, 0)位置的标记, 来更新第0行是否全为0
        if(matrix[0][0] == 0) {
            for(int j = 0; j < n; j++) {
                matrix[0][j] = 0;
            }
        }
        if(col0HasZeros) {
            for(int i = 0; i < m; i++) {
                matrix[i][0] = 0;
            }
        }
    }
}
```

#### 347. 前 k 个高频元素 | medium

##### 1）哈希计数法 | O（k * s）

- **思路**：先对词频进行统计，然后分别找出第 1,2,...,k 大的元素放在结果数组 res 的第 k-1,k-2,...0 位并返回。
- **结论**：时间，3ms，99.72%，空间，43.8mb，5.32%，时间上，由于需要遍历 k 次 counts 数组，每次遍历 counts 数组需要花费 O（max-min+1），记录 max-min+1为 s，所以时间复杂度为 O（k * s），空间上，由于使用了一张 max-min+1 大的 counts 数组，所以额外空间复杂度为 O（s）。

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        if(nums == null || nums.length == 0 || k > nums.length) {
            return new int[0];
        }
        
        int min = Integer.MAX_VALUE, max = Integer.MIN_VALUE;
        for(int i = 0; i < nums.length; i++) {
            min = Math.min(min, nums[i]);
            max = Math.max(max, nums[i]);
        }

        int[] counts = new int[max-min+1];
        for(int i = 0; i < nums.length; i++) {
            counts[nums[i]-min] += 1;
        }

        int maxi;
        int[] res = new int[k];
        while(k > 0) {
            maxi = 0;
            max = Integer.MIN_VALUE;
            for(int i = 0; i < counts.length; i++) {
                if(counts[i] > max) {
                    maxi = i;
                    max = counts[i];
                }
            }
            res[k-1] = maxi + min;
            counts[maxi] = Integer.MIN_VALUE;
            k--;
        }
        
        return res;
    }
}
```

#### 621. 任务调度器 | medium

##### 1）暴力解法 + 贪心 | O（n + m + n * 2m）

- **思路**：
  1. 经过数组状态的研究，发现只要选择优先选择不在冷却中的，且执行次数最多的任务，优先执行，那么就可以得到最优解，因为这样可以平均剩余任务的执行次数，让 cpu 处于等待状态的概率尽量小，从而获取最短的任务执行时间。
  2. 在实现方面，是通过维护 time 时间轴来得到结果的，其中要注意的地方有：
     1. 在任务都冷却时，设置 time + n + 1可以跳过冷却时间。
     2. 而获取到最小冷却后可执行时间的任务时，需要和当前时间轴相比，如果当前时间大则不能更新，因为时间不能倒流，而如果最小冷却后可执行时间大于当前时间，那么时间轴则跳到该可执行时间，以减少时间复杂度。
     3. 每次都优先选择不在冷却中，且执行次数最多的任务来执行，执行后减少可执行次数，以及更新可执行时间为下次冷却后可执行的时间。
- **结论**：时间，72ms，5.30%，空间，42.3mb，6.17%，时间上，初始化 countMap 需要 O（n），初始化 nextValids 和 rests 需要 O（m），遍历一次所有任务为 O（n），每次任务遍历中需要有 2 次的所有的任务种类遍历为 O（m），所以时间复杂度为 O（n + m + n * 2m），空间上，由于使用了一张 m 大的 countMap，一个 m 长的 nextValids 和 rests，所以额外空间复杂度为 O（3m）。

```java
class Solution {
    public int leastInterval(char[] tasks, int n) {
        if(tasks == null || tasks.length == 0) {
            return 0;
        }
        if(n == 0) {
            return tasks.length;
        }

        // 统计每个任务的个数
        Map<Character, Integer> countMap = new HashMap<>();
        for(int i = 0; i < tasks.length; i++) {
            countMap.put(tasks[i], countMap.getOrDefault(tasks[i], 0) + 1);
        }

        // 初始化最早可执行的时间数组nextValids, 以及任务剩余执行次数数组rests
        int taskSize = countMap.size();
        List<Integer> nextValids = new ArrayList<>(taskSize);
        List<Integer> rests = new ArrayList<>(taskSize);
        for(Map.Entry<Character, Integer> entry : countMap.entrySet()) {
            nextValids.add(1);
            rests.add(entry.getValue());
        }

        // 更新时间轴time, 默认从0开始
        int time = 0, nextValid, besti;
        for(int i = 0; i < tasks.length; i++) {
            // 时间轴每次都运行, 且后面负责让time跳过等待时间, 以减少时间复杂度
            time++;

            // 选择其中最小的被冷却限制后的时间
            nextValid = getNextValid(nextValids, rests);
            
            // 时间轴移动到nextValid或者保持不动
            time = Math.max(time, nextValid);

            // 核心逻辑: 选择不在冷却中任务, 且剩余执行次数最多的任务索引
            besti = chooseBestTaskIndex(nextValids, rests, time);
            if(besti != -1) {
                // 执行一次besti处的任务
                rests.set(besti, rests.get(besti) - 1);
                // 更新besti处的任务经过冷却后下次可执行的时间, time跳过等待时间
                nextValids.set(besti, time + n + 1);
            }
        }

        // 全部任务遍历执行并等待完毕, 则返回时间轴当前来到的时间
        return time;
    }

    private int chooseBestTaskIndex(List<Integer> nextValids, List<Integer> rests, int time) {
        int besti = -1;
        for(int i = 0; i < nextValids.size(); i++) {
            // 从不在冷却中的任务进行选择
            if(rests.get(i) != 0 && nextValids.get(i) <= time) {
                // 初始时选择第一个不在冷却中的任务
                if(besti == -1) {
                    besti = i;
                }
                // 优先选择剩余执行次数最多的任务
                else if(rests.get(i) > rests.get(besti)){
                    besti = i;
                }
            }
        }
        return besti;
    }

    private int getNextValid(List<Integer> nextValids, List<Integer> rests) {
        int min = Integer.MAX_VALUE;
        for(int i = 0; i < nextValids.size(); i++) {
            // 从还有执行次数的任务中选取
            if(rests.get(i) != 0) {
                min = Math.min(min, nextValids.get(i));
            }
        }
        return min;
    }
}
```

##### 2）多边形分析法 | O（n + m）

- **思路**：把最多执行次数的任务放满第一列，其余任务从 n-1 开始，由下往上堆叠任务组成 m 长 n+1 宽的多边形，此时，这些任务的最短执行时间=(最多的执行次数-1) * (n+1) + 额外执行的任务数量，证明略，不过由于要执行完所有任务，所以还要将其与任务总数求最大值才返回。

  ![1644310807277](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1644310807277.png)

- **结论**：时间，15ms，38.41%，空间，41.8mb，10.25%，时间上，统计任务个数要 O（n），统计任务最大执行次数要 O（m），所以时间复杂度为 O（n + m），空间上，由于使用了一张 m 长的 countMap，所以额外空间复杂度为 O（m）。

```java
class Solution {
    public int leastInterval(char[] tasks, int n) {
        if(tasks == null || tasks.length == 0) {
            return 0;
        }
        if(n == 0) {
            return tasks.length;
        }

        // 统计每个任务的个数
        int count, maxCount = 0;
        Map<Character, Integer> countMap = new HashMap<>();
        for(int i = 0; i < tasks.length; i++) {
            count = countMap.getOrDefault(tasks[i], 0) + 1;
            countMap.put(tasks[i], count);
            maxCount = Math.max(maxCount, count);
        }

        // 具有最多执行次数的任务数量
        int extraCount = 0; 
        for(Map.Entry<Character, Integer> entry : countMap.entrySet()) {
            if(entry.getValue() == maxCount) {
                extraCount++;
            }
        }

        // 最短时间=(最多的执行次数-1) * (n+1) + 额外执行的任务数量
        return Math.max((maxCount-1) * (n+1) + extraCount, tasks.length);
    }
}
```

#### 2255. 第 77 双周赛 - 统计是给定字符串前缀的字符串数目 | easy

##### 1）哈希表法 | O（n^2 + m）

- **思路**：遍历 s，然后截取每一个前缀到哈希表中，接着遍历 words 数组，判断到在哈希表中有，旧结果加 1 即可。
- **结论**：时间，1 ms，57.93%，空间，41.2 mb，58.79%，时间上，由于遍历 s 要遍历 n 次，substring() 最多也需要遍历 n 长的 s，遍历 words 需要 m 次，所以时间复杂度为 O（n^2 + m），空间上，由于使用了一张 n 长的哈希表，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public int countPrefixes(String[] words, String s) {
        if(words == null || words.length == 0 || s == null) {
            return 0;
        }

        Set<String> set = new HashSet<>();
        for(int len = 1; len <= s.length(); len++) {
            set.add(s.substring(0, len));
        }

        int count = 0;
        for(int i = 0; i < words.length; i++) {
            if(set.contains(words[i])) {
                count++;
            }
        }

        return count;
    }
}
```

##### 2）系统函数法 | O（n^2）

- **思路**：遍历数组 words，然后挨个判断是否是 s 的开头，如果是，则结果 +1。
- **结论**：时间，0 ms，100%，空间，41.4 mb，45.90%，时间上，由于遍历 words 需要 m，startsWith() 最多需要 n，所以时间复杂度为 O（m * n），空间上，由于只是用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int countPrefixes(String[] words, String s) {
        if(words == null || words.length == 0 || s == null) {
            return 0;
        }

        int count = 0;
        for(int i = 0; i < words.length; i++) {
            if(s.startsWith(words[i])) {
                count++;
            }
        }

        return count;
    }
}
```

#### 2283. 第 79 双周赛 - 判断一个数的数字计数是否等于数位的值 | easy

##### 1）哈希表法 | O（2 * n）

- **思路**：先遍历 chars 数组统计，再遍历 chars 数组判断。
- **结论**：时间，0 ms，100%，空间，39.9 mb，31.45%，时间复杂度为 O（2 * n），额外空间复杂度为 O（10）。

```java
class Solution {
    public boolean digitCount(String num) {
        if(num == null || num.length() == 0) {
            return false;
        }
        
        char[] chars = num.toCharArray();
        int[] counts = new int[10];
        
        // 统计
        for(int i = 0; i < chars.length; i++) {
            counts[chars[i] - '0']++;
        }
        
        // 判断
        for(int i = 0; i < chars.length; i++) {
            if(counts[i] != chars[i] - '0') {
                return false;
            }
        }
        
        return true;
    }
}
```

#### 2284. 第 79 双周赛 - 最多单词数的发件人 | easy

##### 1）哈希表法 | O（n * m）

- **思路**：
  1. 遍历 messages 数组，然后分别统计每个 message 的空格数量，+1 就等于单词数量了。
  2. 然后入哈希表缓存 senders 的单词统计次数，用于后面做累加。
  3. 同时，还根据当前的累加次数做最大值比较，count 值比 max 大的，则取当前 senders[i] 作为结果。
  4. counts 值与 max 相同的，则将当前 sends[i] 与 res 做字典序比较，取大者作为结果。
  5. 其中，要注意的是，统计和排序分析，可以放在一起完成，没必要先统计，然后再起一个优先级队列取排序。还有就是，当时字符串排序还写了一个单独的实现方法，浪费了一些时间，其实可以直接使用 `s1.compareTo(s2)` 方法的，都相同的情况下，字符串短的越小。
- **结论**：时间，27 ms，93.74%，空间，48.8 mb，61.94%，时间上，设遍历 messages 花费 O（n），遍历 messages[i] 统计单词数花费 O（m），所以时间复杂度为 O（n * m），空间上，由于使用了一张 n 长的哈希表，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public String largestWordCount(String[] messages, String[] senders) {
        if(messages == null || messages.length == 0 || senders == null || senders.length == 0 || messages.length != senders.length) {
            return null;
        }
        
        // 统计
        String res = "";
        int max = Integer.MIN_VALUE, tmp;
        Map<String, Integer> countMap = new HashMap<>();
        for(int i = 0; i < messages.length; i++) {
            tmp = countMap.getOrDefault(senders[i], 0) + countBlanks(messages[i]);
            countMap.put(senders[i], tmp);

            // count值排序
            if(tmp > max) {
                max = tmp;
                res = senders[i];
            } 
            // 名称字典排序
            else if(tmp == max) {
                res = res.compareTo(senders[i]) < 0? senders[i] : res;
            }
        }

        return res;
    }

    private int countBlanks(String s) {
        int count = 1;
        char[] chars = s.toCharArray();
        for(int i = 0; i < chars.length; i++) {
            if(chars[i] == ' ') {
                count++;
            }
        }
        return count;
    }
}
```

#### 2287. 第 295 周赛 - 重排字符形成目标字符串 | easy

##### 1）哈希表法 | O（2  * s）

- **思路**：遍历 s 数组，统计每个字符出现的次数，再遍历 s / t 次的 t 数组，去扣减 s 统计数组中的次数，并累加 res，如果统计完毕，或者碰到某个 t 数组所需的字符次数为 0 了，就退出扣减循环，返回结果即可。
- **结论**：时间，0 ms，100%，空间，39.3 mb，72.48%，时间上，统计 s 数组中字符出现次数，花费 O（s），遍历 s / t 次的 t 数组，花费 O（s / t * t）=  O（s），所以时间复杂度为 O（2 * s），空间上，由于使用了一个 counts 数组，所以额外空间复杂度为 O（26）。 

```java
class Solution {
    public int rearrangeCharacters(String s, String target) {
        if(s == null || s.length() == 0 || target == null || target.length() == 0) {
            return 0;
        }
        
        char[] schars = s.toCharArray();
        char[] tchars = target.toCharArray();
        
        char[] counts = new char[26];
        for(int i = 0; i < schars.length; i++) {
            counts[schars[i] - 'a']++;
        }
        
        int res = 0, tmp;
        jfor:
        for(int j = 0; j < schars.length / tchars.length; j++) {
            for(int i = 0; i < tchars.length; i++) {
                tmp = tchars[i] - 'a';
                if(counts[tmp] == 0) {
                    break jfor;
                }
                counts[tmp]--;
            }
        
            res++;
        }

        return res;
    }
}
```

#### 2309. 第 298 周赛 - 兼具大小写的最好英文字母 | easy

##### 1）哈希表 + 枚举 | O（n + 26）

- **思路**：
  1. 先用一张哈希表，对 s 中的每个字符缓存起来。
  2. 然后，从 'Z' 开始枚举 26 个大写字符，如果碰到大写和小写同时存在于哈希表中，则认为符合条件，且是最大的，所以直接返回即可，否则如果枚举完都没找到，则返回空串。
  3. 因此，碰到字符题，要往枚举思路去想~
- **结论**：时间，1 ms，98.10%，空间，41.5 mb，18.53%，时间上，遍历 chars 数组花费 O（n），枚举 26 个字符花费 O（26），所以整体的时间复杂度为 O（n + 26），空间上，由于使用了一张长 58 的哈希表，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public String greatestLetter(String s) {
        char[] chars = s.toCharArray();

        int[] cnts = new int['z' - 'A' + 1];
        for(int i = 0; i < chars.length; i++) {
            cnts[chars[i] - 'A']++;
        }

        int offset = 'a' - 'A'; 
        for(int i = 'Z' - 'A'; i > -1; i--) {
            if(cnts[i] > 0 && cnts[i + offset] > 0) {
                return String.valueOf((char) (i + 'A'));
            }
        }
        
        return "";
    }
}
```

##### 2）位运算 + 哈希表 | O（n）

- **思路**：
  1. 同样是先用一张哈希表，对 s 中的每个字符缓存起来，但由于字符个数有限，只有 58 个，所以可以仅仅使用一个 64 位的 long 类型变量，进行按位存储即可。
  2. 然后，按位存储以后也不需要枚举了，直接高位与低位相与，返回高位的第一个 1 对应的大写字符即可，因为这肯定是最大的，且同时出现大写和小写的字符。
  3. 因此，看到有限几个元素时，要往位运算思路去想~
- **结论**：时间，1 ms，98.10%，空间，41.2 mb，41.84%，时间上，遍历 chars 数组花费 O（n），所以时间复杂度为O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public String greatestLetter(String s) {
        char[] chars = s.toCharArray();
        
        // z-a, [\]^-`, Z-A => [122, 65], 一共57+1=58个字符, 所以可以用一个long类型存储, 后面可位运算
        long mask = 0;
        for(int i = 0; i < chars.length; i++) {
            // A: mask & (2^0), B: mask & (2^25), a: mask & (2^32), z: mask & (2^57)
            // 注意, 这里必须为1L, 否则用1则表示32位, 左移会导致溢出
            mask |= 1L << (chars[i] - 'A');
        }

        // 高位的[z, a]右移, 与低位的[Z, A]相与, 最后返回第一个1的索引对应的大写字符即可
        mask &= mask >> ('a' - 'A');
        if(mask == 0) {
            return "";
        } else {
            // 这里的numberOfLeadingZeros是求第一个1位置前面有多少个0
            // 而63-numberOfLeadingZeros则得到第一个1位置的索引, 用索引加'A', 恰好可以得到对应大写字母
            return String.valueOf((char) (63 - Long.numberOfLeadingZeros(mask) + 'A'));
        }
    }
}
```

### 1.4. 数据结构 - 并查集

#### 128. 最长连续序列 | medium

##### 1）暴力解法 | O（n * logn）

- **思路**：先去重，再顺序排序，最后顺序遍历，依次判断是否连续，以及更新连续的最大长度。
- **结论**：时间，20ms，45.04%，空间，53.2mb，62.74%，时间上，由于使用了排序，所以整体时间复杂度为 O（n * logn），空间上，由于额外使用了一个 HashSet 和一个 ArrayList，所以额外空间复杂度为 O（2n），面试会挂，继续优化吧~

```java
class Solution {
    public int longestConsecutive(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        // 去重
        Set<Integer> set = new HashSet<>();
        for(int i = 0; i < nums.length; i++) {
            set.add(nums[i]);
        }

        // 从小到大排序
        List<Integer> list = new ArrayList<>(set);
        list.sort((o1, o2) -> o1 - o2);
        
        int maxlen = 1, curlen = 1;
        for(int i = 1; i < list.size(); i++) {
            if(list.get(i) - 1 == list.get(i-1)) {
                curlen++;
                maxlen = Math.max(maxlen, curlen);
            } else {
                curlen = 1;
            }
        }

        return maxlen;
    }
}
```

##### 2）并查集法 | O（n * α(n)）

- **思路**：根据题意，需要连续的数字序列，因此可以构建通用 Integer 类型的并查集，通过不断合并自身值 +1 的连续集，最后获取最大连续集的大小即可。
- **结论**：
  1. 时间，92ms，31.26%，空间，73.2mb，4.99%。
  2. 空间上，由于使用的是高级数据结构，所以空间复杂度非常高。
  3. 而时间上，由于只遍历一次，且使用了路径压缩的优化，其中 α（n）反阿克曼函数的值不会超过 5，所以时间复杂度为 O（n），但在这里，由于额外的操作较多，时间开销相对来说就比较大了，因此，还需要找到另外一种高效的解法！

```java
class Solution {
    
    class Element<V> {
        private V value;

        Element(V value) {
            this.value = value;
        }
    }

    class UnionFindSet<V> {
        private Map<V, Element<V>> elementMap = new HashMap<>();
        private Map<Element<V>, Element<V>> fatherMap = new HashMap<>();
        private Map<Element<V>, Integer> sizeMap = new HashMap<>();
        private int maxSize = 1;

        UnionFindSet(Set<V> set) {
            for(V value : set) {
                Element<V> e = new Element<>(value);
                elementMap.put(value, e);
                fatherMap.put(e, e);
                sizeMap.put(e, 1);
            }
        }

        private Element<V> findHeader(V value) {
            LinkedList<Element<V>> stack = new LinkedList<>();

            Element<V> e = elementMap.get(value), ef;
            while(e != (ef = fatherMap.get(e))) {
                stack.push(e);
                e = ef;
            }
            while (!stack.isEmpty()) {
                fatherMap.put(stack.pop(), e);
            }
            
            return e;
        }

        boolean isSameSet(V a, V b) {
            if(!elementMap.containsKey(a) || !elementMap.containsKey(b)) {
                return false;
            }
            return findHeader(a) == findHeader(b);
        }

        void union(V a, V b) {
            if(!elementMap.containsKey(a) || !elementMap.containsKey(b)) {
                return;
            }

            Element<V> af = findHeader(a);
            Element<V> bf = findHeader(b);

            if(af != bf) {
                Element<V> big = sizeMap.get(af) > sizeMap.get(bf)? af : bf;
                Element<V> small = big == af? bf : af;
                fatherMap.put(small, big);

                int newSize = sizeMap.get(small) + sizeMap.get(big);
                sizeMap.put(big, newSize);
                sizeMap.remove(small);
                maxSize = Math.max(maxSize, newSize);
            }
        }

        boolean isElement(V v) {
            return elementMap.containsKey(v);
        }

        int getMaxSize() {
            return this.maxSize;
        }
    }

    public int longestConsecutive(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        // 去重
        Set<Integer> set = new HashSet<>();
        for(int i = 0; i < nums.length; i++) {
            set.add(nums[i]);
        }

        // 并差集初始化
        UnionFindSet<Integer> unionFindSet = new UnionFindSet<>(set);

        // 连续集合并
        for(Integer value : set) {
            // 大1存在, 且还不是同一个连续集, 则合并
            if(unionFindSet.isElement(value+1) && !unionFindSet.isSameSet(value, value+1)) {
                unionFindSet.union(value, value+1);
            }
        }

        return unionFindSet.getMaxSize();
    }
}
```

##### 3）哈希表法 | O（n）

- **思路**：
  1. 如果采用从左到右遍历的暴力解法的话，那么还需要从 i 出发再从左往右遍历 nums[i] + 1 是否存在，即使采用哈希表判存也还需要 O（n^2）的时间开销，而这肯定会执行超时的~
  2. 而经过研究发现，遍历+判存的两次循环是不可避免的，不过出现 O（n^2）的原因是，每个元素被多访问了 n 遍，如果能够做到每个元素只被访问一遍，那么即使有两个循环，但整体上时间复杂度还是 O（n）的。
  3. 所以，如果在来到每个元素判存之前，加上判断这个元素是否已经被判断过，即如果 nums[i]-1 的元素存在，说明当前元素肯定会从 nums[i]-1 判存上来的，所以跳过就好，这样就做到了每个元素只被访问一次，时间复杂度为 O（n）。
- **结论**：时间，13ms，85.99%，空间，53.3mb，58.31%，效率非常高了，就这样吧~

```java
class Solution {
    public int longestConsecutive(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        Set<Integer> set = new HashSet<>();
        for(int i = 0; i < nums.length; i++) {
            set.add(nums[i]);
        }

        int max = 0, len, target;
        for(Integer value : set) {
            if(set.contains(value-1)) {
                continue;
            }

            len = 1;
            target = value;

            while(set.contains(target+1)) {
                len++;
                target++;
            }

            max = Math.max(max, len);
        }

        return max;
    }
}
```

#### 200. 岛屿数量 | medium

##### 1）深度优先搜索 | O（2 * m * n）

- **思路**：从左到右、从上到下进行遍历，碰到 '1' 后，则进行 count++，然后感染上、下、左、右附近的 '1'，直到遍历完，返回累加好的 count 即可。
- **结论**：时间，2ms，99.80%，空间，46.8mb，14.58%，时间上，遍历+感染过程由于设置了不为 '1' 则返回，且为 '1' 的也会被感染一次后变为 '2'，所以每个位置最多也就被访问 2 遍，时间复杂度为 O（2 * m * n），空间上，由于递归深度最大能达到整个二维数组，所以额外空间复杂度为 O（m * n）。

```java
class Solution {
    public int numIslands(char[][] grid) {
        if(grid == null || grid.length == 0 || grid[0].length == 0) {
            return 0;
        }
        
        int m = grid.length, n = grid[0].length, count = 0;
        for(int i = 0; i < m; i++) {
            for(int j = 0; j < n; j++) {
                if(grid[i][j] == '1') {
                    count++;
                    infect(grid, m, n, i, j);
                }
            }
        }

        return count;
    }

    // m代表高度, n代表宽度
    private void infect(char[][] grid, int m, int n, int i, int j) {
        if(i < 0 || i >= m || j < 0 || j >= n) {
            return;         
        }
        if(grid[i][j] != '1') {
            return;
        }
        
        // 标记访问过的位置为2
        grid[i][j] = '2';
        
        // 继续感染其他相邻的为1的位置
        infect(grid, m, n, i-1, j);// 上
        infect(grid, m, n, i+1, j);// 下
        infect(grid, m, n, i, j-1);// 左
        infect(grid, m, n, i, j+1);// 右
    }
}

```

##### 2）宽度优先搜索 | O（2 * m * n）

- **思路**：
  1. 看到宽度优先搜索首先想到的就是用队列，碰到为 '1' 的位置则进行 count++，以及感染附近的 '1' 位置。
  2. 不过感染的过程要是宽度优先的，也就是上 -> 下 -> 左 -> 右, 先找完同层的，再找下一层的。
  3. 关键在于找到下层的 '1' 位置后不向递归那样马上进入下层去查找，而是先找完该层才从队列 poll 出来继续找下层的。
- **结论**：
  1. 时间，6ms，22.87%，空间，47.1mb，5.76%，可见对于这些测试用例，效率不如深度优先搜索的。
  2. 时间上，由于每个位置最多被访问两次，所以时间复杂度也是 O（2 * m * n）。
  3. 空间上，由于最多需要构建 m * n 个 Info 对象，所以额外空间复杂度为 O（m * n），不过这里优化成用 row * m + col 的方式可以优化成 max{O（m），O（n）}，即最坏情况下，队列中要么存放了 m 个 Info 对象，表示遍历完了当前 col 的所有行，要么存放了 n 个 Info 对象，表示遍历完了当前 row 的所有列。

```java
class Solution {

    class Info {
        int row;
        int col;

        Info(int row, int col) {
            this.row = row;
            this.col = col;
        }
    }

    public int numIslands(char[][] grid) {
        if(grid == null || grid.length == 0 || grid[0].length == 0) {
            return 0;
        }
        
        Queue<Info> queue = new LinkedList<>();
        Info info;
        
        int m = grid.length, n = grid[0].length, count = 0;
        for(int i = 0; i < m; i++) {
            for(int j = 0; j < n; j++) {
                if(grid[i][j] != '1') {
                    continue;
                }

                count++;
                grid[i][j] = '2';
                queue.offer(new Info(i, j));
                
                // 感染过程 => 宽度优先搜索, 上 -> 下 -> 左 -> 右, 先找完同层的, 再找下一层的
                while(!queue.isEmpty()) {
                    info = queue.poll();

                    // 上
                    if(info.row-1 > -1 && grid[info.row-1][info.col] == '1') {
                        grid[info.row-1][info.col] = '2';
                        queue.offer(new Info(info.row-1, info.col));
                    }
                    // 下
                    if(info.row+1 < m && grid[info.row+1][info.col] == '1') {
                        grid[info.row+1][info.col] = '2';
                        queue.offer(new Info(info.row+1, info.col));
                    }
                    // 左
                    if(info.col-1 > -1 && grid[info.row][info.col-1] == '1') {
                        grid[info.row][info.col-1] = '2';
                        queue.offer(new Info(info.row, info.col-1));
                    }
                    // 右
                    if(info.col+1 < n && grid[info.row][info.col+1] == '1') {
                        grid[info.row][info.col+1] = '2';
                        queue.offer(new Info(info.row, info.col+1));
                    }
                }
            }
        }

        return count;
    }
}
```

##### 3）并查集法 | O（m * n * α（m*n））

- **思路**：根据题意，需要求附近一片 '1' 的个数，因此可以构建通用 Integer 类型的并查集，通过不断合并 '1' 的集合，最后获取不同 '1' 集合个数即可。
- **结论**：
  1. 时间，55ms，6.38%，50.9mb，5.02%，单线程下的效率并不好，但多线程下可以使得矩阵分块进行并查集统计，大大提升效率。
  2. 时间上，由于只遍历一次，且使用了路径压缩的优化，其中 α（n）反阿克曼函数的值不会超过 5，所以时间复杂度为 O（m * n）。
  3. 空间上，由于使用的是高级数据结构，所以空间复杂度非常高~

```java
class Solution {

    class Element<V> {
        private V value;

        Element(V value) {
            this.value = value;
        }
    }

    class UnionFindSet<V> {
        private Map<V, Element<V>> elementMap = new HashMap<>();
        private Map<Element<V>, Element<V>> fatherMap = new HashMap<>();
        private Map<Element<V>, Integer> sizeMap = new HashMap<>();

        UnionFindSet(Set<V> set) {
            for(V v : set) {
                Element<V> e = new Element<>(v);
                elementMap.put(v, e);
                fatherMap.put(e, e);
                sizeMap.put(e, 1);
            }
        }

        private Element<V> findHeader(V value) {
            LinkedList<Element<V>> stack = new LinkedList<>();

            Element e = elementMap.get(value), ef;
            while(e != (ef = fatherMap.get(e))) {
                stack.push(e);
                e = ef;
            }

            // 路径压缩
            while(!stack.isEmpty()) {
                fatherMap.put(stack.pop(), e);
            }

            return e;
        }

        private boolean isSameSet(V a, V b) {
            if(!elementMap.containsKey(a) || !elementMap.containsKey(b)) {
                return false;
            }
            return findHeader(a) == findHeader(b);
        }

        void union(V a, V b) {
            if(!elementMap.containsKey(a) || !elementMap.containsKey(b)) {
                return;
            }
            if(isSameSet(a, b)) {
                return;
            }

            Element<V> af = findHeader(a); 
            Element<V> bf = findHeader(b); 

            if(af != bf) {
                Element<V> big = sizeMap.get(af) > sizeMap.get(bf)? af : bf;
                Element<V> small = big == af? bf : af;
                fatherMap.put(small, big);

                int newSize = sizeMap.get(small) + sizeMap.get(big);
                sizeMap.put(big, newSize);
                sizeMap.remove(small);
            }
        }

        int getSetCount() {
            return sizeMap.size();
        }
    }

    public int numIslands(char[][] grid) {
        if(grid == null || grid.length == 0 || grid[0].length == 0) {
            return 0;
        }
        
        int m = grid.length, n = grid[0].length;
        Set<Integer> set = new HashSet<>();
        for(int i = 0; i < m; i++) {
            for(int j = 0; j < n; j++) {
                // 只初始化'1'的位置, 作为并查集的元素
                if(grid[i][j] == '1') {
                    set.add(i * m + j);
                }
            }
        }

        UnionFindSet<Integer> unionFindSet = new UnionFindSet<>(set);
        for(int i = 0; i < m; i++) {
            for(int j = 0; j < n; j++) {
                if(grid[i][j] != '1') {
                    continue;
                }

                // 合并并查集
                grid[i][j] = '2';
                int p = i * m + j;
                if(i-1 > -1 && grid[i-1][j] == '1') {
                    unionFindSet.union(p, (i-1) * m + j);
                }
                if(i+1 < m && grid[i+1][j] == '1') {
                    unionFindSet.union(p, (i+1) * m + j);
                }
                if(j-1 > -1 && grid[i][j-1] == '1') {
                    unionFindSet.union(p, i * m + (j-1));
                }
                if(j+1 < n && grid[i][j+1] == '1') {
                    unionFindSet.union(p, i * m + (j+1));
                }
            }
        }

        return unionFindSet.getSetCount();
    }
}
```

#### 2316. 第 81 双周赛 - 统计无向图中无法互相到达点对数 | medium

##### 1）并查集法 | O（n + m + k）

- **思路**：使用并查集的模板，然后根据 edges 边集收集点，形成不同的集合，最后看每个集合中有多少个点，差几个点没连上，这就是 `(n - size) * n` 的含义所在。
- **结论**：时间，191 ms，7.22%，空间，116 mb，13.96%，时间上，并查集构造花费 O（n），n 代表点的个数，边集遍历花费 O（m），m 代表 edges 边集的行数，数目统计花费 O（k），k 代表并查集集合的个数，所以整体时间复杂度为 O（n + m + k），空间上，由于使用了 4 张哈希表，所以额外空间复杂度为 O（4 * n）。

 ```java
class Element<V> {
    V value;

    Element(V value) {
        this.value = value;
    }
}

class UnionFindSet<V> {
    Map<V, Element<V>> elementMap = new HashMap<>();
    Map<Element<V>, Element<V>> fatherMap = new HashMap<>();
    Map<Element<V>, Integer> sizeMap = new HashMap<>();

    UnionFindSet(Set<V> set) {
        for(V value : set) {
            Element<V> e = new Element<>(value);
            elementMap.put(value, e);
            fatherMap.put(e, e);
            sizeMap.put(e, 1);
        }
    }

    private Element<V> findHeader(V value) {
        LinkedList<Element<V>> stack = new LinkedList<>();

        Element<V> e = elementMap.get(value), ef;
        while(e != (ef = fatherMap.get(e))) {
            stack.push(e);
            e = ef;
        }
        while (!stack.isEmpty()) {
            fatherMap.put(stack.pop(), e);
        }

        return e;
    }

    boolean isSameSet(V a, V b) {
        if(!elementMap.containsKey(a) || !elementMap.containsKey(b)) {
            return false;
        }
        return findHeader(a) == findHeader(b);
    }

    void union(V a, V b) {
        if(!elementMap.containsKey(a) || !elementMap.containsKey(b)) {
            return;
        }

        Element<V> af = findHeader(a);
        Element<V> bf = findHeader(b);

        if(af != bf) {
            Element<V> big = sizeMap.get(af) > sizeMap.get(bf)? af : bf;
            Element<V> small = big == af? bf : af;
            fatherMap.put(small, big);

            int newSize = sizeMap.get(small) + sizeMap.get(big);
            sizeMap.put(big, newSize);
            sizeMap.remove(small);
        }
    }

    boolean isElement(V v) {
        return elementMap.containsKey(v);
    }
}

class Solution {
    public long countPairs(int n, int[][] edges) {
        Set<Integer> set = new HashSet<>();
        for(int i = 0; i < n; i++) {
            set.add(i);
        }
        
        UnionFindSet<Integer> unionSet = new UnionFindSet(set);
        for(int i = 0; i < edges.length; i++) {
            int from = edges[i][0];
            int to = edges[i][1];
            unionSet.union(from, to);
        }
        
        
        long cnt = 0;
        for(Map.Entry<Element<Integer>, Integer> entry : unionSet.sizeMap.entrySet()) {
            cnt += (n - entry.getValue()) * (long) entry.getValue();
        }
        
        return cnt / 2;
    }
}
 ```

##### 2）深度优先搜索 | O（2 * n + m）

- **思路**：先根据 edges 数组建立一张连通图，然后根据连通图做 dfs 搜索，并统计每张连通图的大小，通过 `size * (n - size)` 得出 A 连通图没连到 B 连通图的数量，最后累加每张连通图没连接的数量即可，而由于这里的累加，把 A 缺 B 和 B 缺 A 都看作是不同结果，但题目给的是无向图，所以需要看作上面两者属于同一结果，因此结果需要除以 2 再返回。
- **结论**：时间，37 ms，36.80%，空间，106.5 mb，36.33%，时间上，初始化连通图花费 O（n），建立连通图花费 O（m），遍历每张连通图时做 dfs 花费 O（n），所以整体时间复杂度为 O（2 * n + m），空间上，由于使用了一张 n * m 大的 lists 二维表，一张 n 长的 vis 表，dfs 递归栈深度最大为 n，所以额外空间复杂度为 O（n * m + 2 * n）。

```java
class Solution {
    public long countPairs(int n, int[][] edges) {
        // 初始化连通图
        List<Integer>[] lists = new ArrayList[n];
        Arrays.setAll(lists, o -> new ArrayList<Integer>());

        // 建立连通图
        for(int i = 0; i < edges.length; i++) {
            int x = edges[i][0];
            int y = edges[i][1];
            lists[x].add(y);
            lists[y].add(x);
        }

        // 访问结果集, 访问过则不能再访问
        boolean[] vis = new boolean[n];

        // dfs遍历, 得出每个连通图的大小, 并计算结果
        long cnt = 0;
        for(int i = 0; i < n; i++) {
            int size = dfs(lists, vis, i);
            cnt += (long) size * (n - size);
        }

        // 返回结果
        return cnt / 2; 
    }

    // 求start开始, 未访问过的连通区域大小
    private int dfs(List<Integer>[] lists, boolean[] vis, int start) {
        if(vis[start]) {
            return 0;
        }

        int size = 1;
        vis[start] = true;
        for(Integer i : lists[start]) {
            size += dfs(lists, vis, i);
        }
     
        return size;
    }
}
```

##### 3）广度优先搜索 | O（2 * n + m）

- **思路**：同理，与深度优先搜索思路类似，但这里是通过队列的方式实现 bfs，而非递归栈。
- **结论**：时间，53 ms，23.96%，空间，132.5 mb，5.04%，时间上，初始化连通图花费 O（n），建立连通图花费 O（m），遍历每张连通图时做 bfs 花费 O（n），所以整体时间复杂度为 O（2 * n + m），空间上，由于使用了一张 n * m 大的 lists 二维表，一张 n 长的 vis 表，bfs 队列最大长 n，所以额外空间复杂度为 O（n * m + 2 * n）。

```java
class Solution {
    public long countPairs(int n, int[][] edges) {
        // 初始化连通图
        List<Integer>[] lists = new ArrayList[n];
        Arrays.setAll(lists, o -> new ArrayList<Integer>());

        // 建立连通图
        for(int i = 0; i < edges.length; i++) {
            int x = edges[i][0];
            int y = edges[i][1];
            lists[x].add(y);
            lists[y].add(x);
        }

        // 访问结果集, 访问过则不能再访问
        boolean[] vis = new boolean[n];

        // dfs遍历, 得出每个连通图的大小, 并计算结果
        long cnt = 0;
        for(int i = 0; i < n; i++) {
            int size = bfs(lists, vis, i);
            cnt += (long) size * (n - size);
        }

        // 返回结果
        return cnt / 2; 
    }

    // 求start开始, 未访问过的连通区域大小
    private int bfs(List<Integer>[] lists, boolean[] vis, int cur) {
        Queue<Integer> queue = new LinkedList<>();
        queue.offer(cur);

        int size = 0;
        while(!queue.isEmpty()) {
            cur = queue.poll();
            if(!vis[cur]) {
                vis[cur] = true;
                size++;

                for(Integer i : lists[cur]) {
                    queue.offer(i);
                }
            }
        }
     
        return size;
    }
}
```

### 1.5. 数据结构 - 滑动窗口

#### 3. 无重复字符的最长子串 | medium

##### 1）滑动窗口 + 队列 + 哈希表 | O（n）

- **思路**：通过队列存放无重复字符，哈希表维护队列中字符出现的次数，遍历 chars 加入新字符前，先判断是否存在重复字符，存在的则移除再添加，然后使用 max 记录全局最大无重复字串长度。
- **结论**：时间，7 ms，25.47%，空间，42.5 mb，5.08%，时间上，由于只需迭代一次字符串，所以时间复杂度为 O（n），空间上，由于使用了一张长 n 的 chars 数组，n 长的 queue 队列，n 长的 map 集合，所以额外空间复杂度为 O（3 * n）。

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        char[] chars = s.toCharArray();
        int n = chars.length;
        
        int max = 0;
        Queue<Character> queue = new LinkedList<>();
        Map<Character, Integer> map = new HashMap<>();
        for(int i = 0; i < n; i++) {
            char c = chars[i];
            while(map.containsKey(c)) {
                char first = queue.poll();

                int cnt = map.get(first);
                if(cnt == 1) {
                    map.remove(first);
                } else {
                    map.put(first, cnt - 1);
                }
            }

            queue.offer(c);
            map.put(c, map.getOrDefault(c, 0) + 1);
            max = Math.max(max, queue.size());
        }

        return max;
    }
}
```

##### 2）滑动窗口 + 哈希表 | O（n）

- **思路**：由于是队列中是连续字串，所以可以直接使用一个 firsti 变量来替代队头，从而减少队列的出队、入队开销。
- **结论**：时间，6 ms，40.84%，空间，41.8 mb，28.24%，时间上，由于只需迭代一次字符串，所以时间复杂度为 O（n），空间上，由于使用了一张长 n 的 chars 数组，n 长的 map 集合，所以额外空间复杂度为 O（2 * n）。

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        char[] chars = s.toCharArray();
        int n = chars.length;
        
        int max = 0, firsti = 0;
        Map<Character, Integer> map = new HashMap<>();
        for(int i = 0; i < n; i++) {
            char c = chars[i];
            while(map.containsKey(c)) {
                char first = chars[firsti++];
                int cnt = map.get(first);
                if(cnt == 1) {
                    map.remove(first);
                } else {
                    map.put(first, cnt - 1);
                }
            }

            map.put(c, map.getOrDefault(c, 0) + 1);
            max = Math.max(max, i - firsti + 1);
        }

        return max;
    }
}
```

#### 30. 串联所有单词的子串 | hard

##### 1）暴力解法 + 滑动窗口 | O（sn * m）

- **思路**：
  1. 枚举各种开头的子串，然后分别根据 words 数组，构造初始化集合 diffMap，相当于右侧入窗。
  2. 接着，以 m 长为一组，作为一个 m 子串，遍历 i 开头的 m 子串，并更新到 diffMap 中，相当于左侧入窗。
  3. 最后，如果 diffMap 对应的子串个数为 0，那么就移出集合，在所有 m 子串遍历完毕后，都判断 diffMap 集合是否为空，为空则把当前 i 索引加入返回结果中，代表各 m 子串都与 words 的相同，且数量也与 words 中的相等。
- **结论**：时间，365 ms，10.62%，空间，42.1 mb，50.91%，时间上，i 枚举花费 O（sn），m 子串枚举花费 O（m），所以整体时间复杂度为 O（sn * m），空间上，由于使用了 sn 张  n + sn / n 长的哈希表，所以额外空间复杂度为 O（sn * n + sn^2 / n）。

```java
class Solution {
    public List<Integer> findSubstring(String s, String[] words) {
        int m = words.length, n = words[0].length(), sn = s.length();

        // 枚举各种开头的子串
        List<Integer> res = new ArrayList<>();
        for(int i = 0; i + m * n <= sn; i++) {
            // 根据words数组, 初始化差异初始集合diffMap
            Map<String, Integer> diffMap = new HashMap<>();
            for(String word : words) {
                diffMap.put(word, diffMap.getOrDefault(word, 0) - 1);
            }

            // 遍历每个子串中n长的子串, 一共有m个
            for(int j = 0; j < m; j++) {
                String sub = s.substring(i + j * n, i + (j + 1) * n);

                // 进入窗口, 则+1
                diffMap.put(sub, diffMap.getOrDefault(sub, 0) + 1);
                
                // 如果diff为0, 那么说明这个单词是需要的, 从集合中删除
                if(diffMap.get(sub) == 0) {
                    diffMap.remove(sub);
                }
            }

            // 如果集合被移除为空了, 那么就认为各子串m相同, 且数量相等, 则把当前索引加入集合中
            if(diffMap.isEmpty()) {
                res.add(i);
            }
        } 

        return res;
    }
}
```

##### 2）单词统计 + 哈希表法 | O（m + sn * m）

- **思路**：
  1. 先统计一遍 words 出现的次数，然后枚举 m * n 长的子串。
  2. 其中，枚举过程中要判断子串是否与 words 中所有的串组合匹配，这里判断的方法是，判断子串中 n 长串的个数、种类，是否都和 words 中的匹配。
  3. 所以，采用了第二张哈希表，来存储子串中 n 长串出现的个数，以及判断该种类是否有在 words 中出现。
  4. 当种类、次数都符合要求，则把索引 i 加入结果集中，代表完全匹配。
- **结论**：时间，92 ms，47.78%，空间，42.1 mb，56.90%，时间上，统计 words 次数花费 O（m），枚举 s 子串花费 O（sn），拆分 m 子串花费 O（m），所以整体时间复杂度为 O（m + sn * m），空间上，由于使用了一张 m 长的 wmap 哈希表，sn 张 m 长的 submap 哈希表，所以额外空间复杂度为 O（m + sn * m）。

```java
class Solution {
    public List<Integer> findSubstring(String s, String[] words) {
        int m = words.length, n = words[0].length(), sn = s.length();

        // 统计word出现次数
        Map<String, Integer> wmap = new HashMap<>();
        for(String word : words) {
            wmap.put(word, wmap.getOrDefault(word, 0) + 1);
        }

        // 枚举子串
        List<Integer> res = new ArrayList<>();
        for(int i = 0; i + m * n <= sn; i++) {
            Map<String, Integer> submap = new HashMap<>();

            // 挨个挨个拆分sub为n长度的子串, 每次都是m中的一个, 或者不是
            int cnt = 0;
            for(int j = 0; j < m; j++) {
                String sub = s.substring(i + j * n, i + (j + 1) * n);

                // 判断是否出现在words中
                if(!wmap.containsKey(sub)) {
                    break;
                }

                // 判断次数是否等于words中的次数
                submap.put(sub, submap.getOrDefault(sub, 0) + 1);
                if(submap.get(sub) > wmap.get(sub)) {
                    break;
                }

                // 都符合, 则个数加1
                cnt++;
            }

            // 个数符合、出现的种类符合, 说明完全匹配, 则加入结果集中
            if(cnt == m) {
                res.add(i);
            }
        }

        return res;
    }
}
```

#### 76. 最小覆盖子串 | hard

##### 1）暴力解法 | O（n^3）

- **思路**：
  1. 从左到右遍历匹配串 s，然后再从 0,1,2,...,n 一直尝试不同长度的子串，去匹配原串 t。
  2. 判断是否匹配时，由于匹配不要求顺序，所以对应的字母词频相同即可，所以再遍历每个子串的词频，看是否大于等于 t 的词频，如果相等，则返回 true，否则返回 false。
  3. 最后，判断获取长度最小的匹配子串，然后返回即可。
- **结论**：
  1. 执行超时，由于每次都统计 t 和各子串 的词频，做多了很多重复的工作，所以这方面还可以优化下。
  2. 同时，由于题目约定的用例全都是字母，所以统计词频也不用开辟 128 长度的数组，只需要 59 个格子就好，因为中间多了（58 '：'，59 '；'，60 '<'，61 '='，62 '>'，63 '?'，64 '@'）这 7 个字符！

```java
class Solution {
    public String minWindow(String s, String t) {
        if(s == null || t == null) {
            return "";
        }
        if(t.length() == 0 || s.length() == 0) {
            return "";
        }

        char[] charsS = s.toCharArray();
        char[] charsT = t.toCharArray();

        int[] tCount = new int[59];
        for(int j = 0; j < charsT.length; j++) {
            tCount[charsT[j] - 'A'] += 1;
        }

        int minlen = Integer.MAX_VALUE, minStart = 0;
        for(int i = 0; i < charsS.length; i++) {
            for(int len = 0; len < charsS.length - i; len++) {
                if(f(charsS, tCount, i, len)) {
                    if(len < minlen) {
                        minlen = len;
                        minStart = i;
                    }
                }
            }
        }

        return s.substring(minStart, minStart + minlen + 1);
    }

    private boolean f(char[] charsS, int[] tCount, int i, int len) {
        int[] sCount = new int[59];
        for(int j = i; j <= i + len; j++) {
            sCount[charsS[j] - 'A'] += 1;
        }

        for(int j = 0; j < tCount.length; j++) {
            if(tCount[j] > sCount[j]) {
                return false;
            }
        }

        return true;
    }
}
```

##### 2）滑动窗口法 | O（n）

- **思路**：
  1. 观察暴力解法的过程可得知，暴力解法中，常常由于增加了一个字符，或者换了一个字符开头，就要重新对词频进行一次统计，导致工作了很多重复的内容。
  2. 而这点可以用**滑动窗口**来优化，减少不必要的重复计算：
     1. 设定一个 l 和 r 代表滑动窗口的左右边界，为了方便计算（让 r - l 即等于滑动窗口长度），约定这个区间是一个左闭右开的区间 [l，r)，至此可以让滑动窗口从左到右进行 s 的字符数组扫描。
     2. 同时，由于控制了每次扫描都会增加，或者减少滑动窗口内的一个字符，所以可以通过使用**编辑距离**的方式来优化掉 t 匹配判断函数 f，使其降低为 O（1）。
        1. 即首先初始化好 t 的词频数组，然后使用一个 distance 来表示滑动窗口内接近等于 t 词频的距离。
        2. 如果增加了一个 t 需要的字符，则距离-1，如果减少了一个 t 需要的字符，则距离+1。
        3. 而对于那些 t  不关心的字符，距离不用变，只需要在词频数组中做好记录就行。
     3. 然后滑动窗口的移动逻辑为，首先移动 r，在 r 不越界的情况下，登记 r 字符的词频，以及比较距离是否为 0，如果距离不为 0，说明此时滑动窗口内还未完全拥有 t 需要的字符，所以 r 继续向右移动，扩大滑动窗口长度，寻找可行解。
     4. 如果距离为 0 了，说明此时滑动内已经完全拥有了 t 需要的字符，但由于可能有多余的字符，所以还需要 l 从左边进行移动，缩减滑动窗口的大小，逐步可行解转换为最优解，但如果减少了字符后，发现距离不为 0，说明把 t 需要的字符也删了，那么 l 就停止移动，重新移动 r，转向寻找下一堆可行解，然后周而复始。
     5. 就这样，当 r 越界时，说明所有可行解已经寻找完毕，且滑动窗口内也没有多余的字符了，即也是最优解了，因为 r 移动前，是上次 l 移动后发生的，此时 l~r 内必定是缺一个 t 需要的字符的，如果 r 越界前，刚刚找到最后一个 t 需要的字符，那么说明 l~r 内是刚好满足 t 需要的字符的，所以如果此时匹配 t 的话，就已经是此时的最优解了。
     6. 最后，把最小的每个最优解进行对比，然后截取 s 字符串并返回。
- **结论**：时间，2ms，99.64%，38.5mb，79.91%，效率非常高，以后对于这种**缩减暴力解的优化**，可以考虑下滑动窗口。

```java
class Solution {
    public String minWindow(String s, String t) {
        if(s == null || s.length() == 0 || t == null || t.length() == 0) {
            return "";
        }
        if(s.length() < t.length()) {
            return "";
        }

        char[] sChars = s.toCharArray();
        char[] tChars = t.toCharArray();

        int[] tcounts = new int['z' - 'A' + 1];
        for(int i = 0; i < tChars.length; i++) {
            tcounts[tChars[i] - 'A']++;
        }

        String res = "";
        int diff = tChars.length, min = Integer.MAX_VALUE, idx;
        Deque<Integer> deque = new LinkedList<>();
        for(int i = 0; i < sChars.length; i++) {
            // 入队
            idx = sChars[i] - 'A';
            deque.offerLast(i);

            // 只有当t是需要时, diff才减减
            if(tcounts[idx] > 0) {
                diff--;
            }

            // 这里用负数来搞定, 滑动窗口内的多个重复t字符
            tcounts[idx]--;

            // 只有当diff刚好凑齐时, 才开始出队
            while(diff == 0) {
                // 队列为空diff都为0, 说明t可能是个空串
                if(deque.isEmpty()) {
                    return "";
                }

                // 出现更小的, 则更新结果
                if(deque.size() < min) {
                    min = deque.size();
                    res = s.substring(deque.peekFirst(), deque.peekLast() + 1);
                }

                idx = sChars[deque.pollFirst()] - 'A';
                
                // 只有当t是需要时, diff才加加, 因为出队了
                if(tcounts[idx] == 0) {
                    diff++;
                }

                // 这里用负数来搞定, 滑动窗口内的多个重复t字符
                tcounts[idx]++;
            }
        }

        return res;
    }

    private void mark(String s) {
        System.out.println(s);
    }
}
```

#### 159. 至多包含两个不同字符的最长子串 | medium

##### 1）滑动窗口法 | O（n）

- **思路**：
  1. 建立只能存储两种元素的滑动窗口，然后从左到右遍历原数组，判断是否可加入滑动窗口，分为 3 种情况：
     - 1）小于2个时，直接进队。
     - 2）大于等于 2 个时, 则判断是否可以进队。
     - 3）不可进队，则先出队。
  2. 其中，还需要一张哈希表，来记录每种元素在滑动窗口内出现的次数，以实现在元素次数为 0 时出队。
- **结论**：时间，34 ms，64.94%，空间，42.6 mb，8.42%，时间上，由于最坏情况下，需要遍历 n - 1 长数组，然后在添加最后一个元素需要移出 n - 2 个元素，此时时间复杂度为 O（n - 1 + n - 2 )，空间上，由于需要一个 n 长的队列，n 长的哈希表，所以额外空间复杂度为 O（2 * n）。

```java
import java.util.HashMap;
import java.util.LinkedList;
class Solution {
    public int lengthOfLongestSubstringTwoDistinct(String s) {
        if(s == null || s.length() == 0) {
            return 0;
        }
        if(s.length() <= 2) {
            return s.length();
        }

        char[] chars = s.toCharArray();
        LinkedList<Character> deque = new LinkedList<>();
        HashMap<Character, Integer> map = new HashMap<>();

        deque.offerLast(chars[0]);
        map.put(chars[0], map.getOrDefault(chars[0], 0) + 1);

        deque.offerLast(chars[1]);
        map.put(chars[1], map.getOrDefault(chars[1], 0) + 1);

        int max = deque.size();
        for (int i = 2; i < chars.length; i++) {
            // 小于2个时, 直接进队
            if(map.size() < 2) {
                deque.offerLast(chars[i]);
                map.put(chars[i], map.getOrDefault(chars[i], 0) + 1);
            }
            // 大于等于2个时, 则判断是否可以进队
            else if (map.containsKey(chars[i])) {
                deque.offerLast(chars[i]);
                map.put(chars[i], map.get(chars[i]) + 1);
            }
            // 不可进队, 则先出队
            else {
                while (map.size() >= 2) {
                    Character c = deque.pollFirst();
                    Integer count = map.get(c);
                    if(count == 1) {
                        map.remove(c);
                    } else {
                        map.put(c, count - 1);
                    }
                }

                deque.offerLast(chars[i]);
                map.put(chars[i], map.getOrDefault(chars[i], 0) + 1);
            }

            max = Math.max(deque.size(), max);
        }

        return max;
    }
}
```

#### 239. 滑动窗口最大值 | hard

##### 1）单调双端队列法 | O（n）

- **思路**：
  1. 设计一个单调递减的双端队列，存放数组的索引（这样可以存放更多的信息），队头到队尾索引对应的元素值从大到小排列。
  2. 根据单调双端队列的特性，在滑动窗口向右滑动时，需要从队尾判断加入的元素是否符合单调性，如果不符合（即大于队列中的元素），则把队列中不符合（小于等于）的元素从队尾一次弹出，再加入当前元素，此时右边界移动完毕。
  3. 由于本题的滑动窗口规定为 k，所以移动右边界完毕还需要移动左边界，右边界为 i 时的理论左边界应该为 i-k+1，如果队头的索引小于该理论值，则认为左边界需要向右移动，此时则从队头弹出元素。
  4. 最后，左右边界移动完毕，此时的滑动窗口已经维护完毕，所以把队头索引对应的值，代表滑动窗口内部的最大值，加入结果集中，然后周而复始即可。 
- **结论**：时间，28ms，90.13%，空间，54.4mb，34.08%，效率非常高了，就这样吧~

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        if(nums == null || k < 1 || nums.length < k) {
            return null;
        }

        // 双端队列, 存的是数组的下标
        Deque<Integer> dequeue = new LinkedList<>();
        int[] res = new int[nums.length - k + 1];

        // 从左到右移动右边界R
        int index = 0;
        for(int i = 0; i < nums.length; i++) {
            // 从双端队列队尾弹出小于等于当前值的元素
            while(!dequeue.isEmpty() && nums[dequeue.peekLast()] <= nums[i]) {
                dequeue.pollLast();
            }

            // 从双端队列队尾加入当前元素
            dequeue.offerLast(i);

            // 如果双端队列中元素索引跨度大于k, 则从队头弹出, 代表移动左边界L
            if(dequeue.peekFirst() < i-k+1) {
                dequeue.pollFirst();
            }

            // 维护好双端队列后, 把滑动窗口内的最大值(队头元素), 加入结果集中
            if(i-k+1 >= 0) {
                res[index++] = nums[dequeue.peekFirst()];   
            }
        }

        return res;
    }
}
```

#### 438. 找到字符串中所有字母异位词 | medium

##### 1）暴力 + 滑动窗口 | O （p + [s - p] * p）

- **思路**：
  1. 使用双端队列做滑动窗口，从右进从左出，先输入固定大小为 p 的数据到滑动窗口，然后从 p 开始遍历 s 数组，最左元素出队前，先判断窗口内的字符串是否符合异位词格式，然后再左出队一个吗，右进队一个，周而复始。 
  2. 其中，判断窗口内的字符串是否符合异位词格式，采用遍历的方式去判断，每次遍历 p 次，如果任意有一次不合法，则返回 false，否则返回 true。
- **结论**：
  1. 时间，2418 ms，5.02%，空间，43.2mb，5.00%。
  2. 时间上，初始化窗口需要遍历 p 次，然后从 p 开始遍历到 s 结束，需要遍历 s-p 次，每次遍历需要判断窗口内字符串是否合法，需要遍历 p 次，因此时间复杂度为 O（p + [s-p] * p）。
  3. 空间上，由于使用了两张哈希表，最多存放 p 个元素，一共为 O（2 * p），以及一个滑动窗口，最多存放 s 个元素，所以总的额外空间复杂度为 O（2 * p + s）。

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        if(s == null || p == null || s.length() == 0 || p.length() == 0 
            || s.length() < p.length()) {
            return new ArrayList<>();
        }

        int sn = s.length(), pn = p.length();
        Map<Character, Integer> pcountMap = new HashMap<>();
        for(char pc : p.toCharArray()) {
            if(!pcountMap.containsKey(pc)) {
                pcountMap.put(pc, 1);
            } else {
                pcountMap.put(pc, pcountMap.get(pc) + 1);
            }
        }

        // 固定滑动窗口大小
        Deque<Integer> slidingWindow = new LinkedList<>();
        Map<Character, Integer> scountMap = new HashMap<>();
        for(int i = 0; i < pn; i++) {
            slidingWindow.offerLast(i);
            maintainScountMap(scountMap, s.charAt(i), 1);
        }

        // 滑动窗口从pn处向右移动
        int li, ri;
        List<Integer> res = new LinkedList<>();
        for(int i = pn; i < sn; i++) {
            li = slidingWindow.peekFirst();
            ri = slidingWindow.peekLast();

            // 判断是否为p的异位词
            if(isValid(scountMap, pcountMap, s, li, ri)) {
                res.add(li);
            }
            
            // 弹出
            slidingWindow.pollFirst();
            maintainScountMap(scountMap, s.charAt(li), -1);

            // 进入
            slidingWindow.offerLast(i);
            maintainScountMap(scountMap, s.charAt(i), 1);
        }

        // 结算滑动窗口内最后的字符串
        li = slidingWindow.peekFirst();
        ri = slidingWindow.peekLast();
        if(isValid(scountMap, pcountMap, s, li, ri)) {
            res.add(li);
        } 

        return res;
    }

    private boolean isValid(Map<Character, Integer> scountMap, Map<Character, Integer> pcountMap, String s, int li, int ri) {
        char c;
        for(int i = li; i <= ri; i++) {
            c = s.charAt(i);
            if(!pcountMap.containsKey(c)) {
                return false;
            }
            if(!scountMap.get(c).equals(pcountMap.get(c))) {
                return false;
            }   
        }

        return true;
    }

    private void maintainScountMap(Map<Character, Integer> scountMap, char sc, int status) {
        // 加
        if(status == 1) {
            if(!scountMap.containsKey(sc)) {
                scountMap.put(sc, 1);
            } else {
                scountMap.put(sc, scountMap.get(sc) + 1);
            }
        } 
        // 减
        else {
            if(!scountMap.containsKey(sc)) {
                return;
            } else {
                scountMap.put(sc, scountMap.get(sc) - 1);
            }
        }   
    }
}
```

##### 2）词频数组 + 滑动窗口 | O（p + [s - p] * 26）

- **思路**：在暴力解法的基础上，由于使用了哈希表长度不固定，因此可以用 26 长的词频数组来替代，以及判断异位词合法也不再遍历 p 次了，而是使用了遍历 26 长度词频数组来替代，把时间复杂度从 p 次优化成 26 常数次。
- **结论**：
  1. 时间，13ms，32.67%，空间，43.5mb，4.99%。
  2. 时间上，初始化窗口需要遍历 p 次，然后从 p 开始遍历到 s 结束，需要遍历 s-p 次，每次遍历需要判断窗口内字符串是否合法，需要遍历 26 次，因此时间复杂度为 O（p + [s-p] * 26）。
  3. 空间上，由于使用了两张词频数组，最多存放 26 个元素，一共为 O（52），以及一个滑动窗口，最多存放 s 个元素，所以总的额外空间复杂度为 O（52 + s）。

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        if(s == null || p == null || s.length() == 0 || p.length() == 0 
            || s.length() < p.length()) {
            return new ArrayList<>();
        }
        
        char[] schars = s.toCharArray(), pchars = p.toCharArray();
        int sn = schars.length, pn = pchars.length;

        // 固定滑动窗口大小
        int[] pcount = new int[26];
        int[] swcount = new int[26];
        Deque<Integer> slidingWindow = new LinkedList<>();
        for(int i = 0; i < pn; i++) {
            pcount[pchars[i] - 'a']++;
            swcount[schars[i] - 'a']++;
            slidingWindow.offerLast(i);
        }

        // 滑动窗口从pn处向右移动
        int li;
        List<Integer> res = new LinkedList<>();
        for(int i = pn; i < sn; i++) {
            li = slidingWindow.peekFirst();

            // 判断是否为p的异位词
            if(Arrays.equals(swcount, pcount)) {
                res.add(li);
            }
            
            // 弹出
            slidingWindow.pollFirst();
            swcount[schars[li] - 'a']--;

            // 进入
            slidingWindow.offerLast(i);
            swcount[schars[i] - 'a']++;
        }

        // 结算滑动窗口内最后的字符串
        if(Arrays.equals(swcount, pcount)) {
            res.add(slidingWindow.peekFirst());
        } 

        return res;
    }
}
```

##### 3）编辑距离 + 滑动窗口 | O（26 + s）

- **思路**：
  1. 在词频统计法的基础上，由于每次都需要比较两个词频数组是否相等，使得很多元素重复比较了，因为变化的就只有出队和入队的两个元素。
  2. 所以，采用编辑距离的方式，通过 upDiff 来记录 sc 多了的距离，downDiff 来记录 sc 少了的距离，然后滑动窗口向右移动时，不断判断 upDiff 和 downDiff 是否都为 0 来决定窗口内字符串是否合法。
  3. 以及通过 count[ci] 的状态，来维护 upDiff 和 downDiff 的值，周而复始。
- **结论**：
  1. 时间，11ms，37.91%，空间，42.7mb，4.99%。
  2. 时间上，初始化窗口需要遍历 p 次，然后初始化 upDiff 和 downDiff 需要遍历 26 次  count 数组，然后从 p 开始遍历到 s 结束，需要遍历 s-p 次，因此时间复杂度为 O（p + 26 + [s-p]）= O（26 + s）。
  3. 空间上，由于只使用了 1 张词频数组，最多存放 26 个元素，一共为 O（26），以及一个滑动窗口，最多存放 s 个元素，所以总的额外空间复杂度为 O（26 + s）。

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        if(s == null || p == null || s.length() == 0 || p.length() == 0 
            || s.length() < p.length()) {
            return new ArrayList<>();
        }
        
        char[] schars = s.toCharArray(), pchars = p.toCharArray();
        int sn = schars.length, pn = pchars.length;

        // 固定滑动窗口大小: pc - sc, 大于0表示sc字母少了, 小于0表示sc字母多了
        int[] count = new int[26];
        Deque<Integer> slidingWindow = new LinkedList<>();
        for(int i = 0; i < pn; i++) {
            count[pchars[i] - 'a']--;
            count[schars[i] - 'a']++;
            slidingWindow.offerLast(i);
        }

        // 获取编辑距离差距
        int upDiff = 0, downDiff = 0;
        for(int i = 0; i < count.length; i++) {
            if(count[i] > 0) {
                upDiff += count[i];// sc多了
            } else if(count[i] < 0) {
                downDiff += count[i];// sc少了
            }
        }

        // 滑动窗口从pn处向右移动
        int li, ci;
        List<Integer> res = new LinkedList<>();
        for(int i = pn; i < sn; i++) {
            li = slidingWindow.pollFirst();

            // 判断是否为p的异位词
            if(upDiff == 0 && downDiff == 0) {
                res.add(li);
            }

            // 弹出, 相当于sc在减少
            if(count[ci = schars[li] - 'a'] == 0) {
                downDiff--;
            } else if(count[ci] > 0) {
                upDiff--;
            } else {
                downDiff--;
            }
            count[ci]--;
            
            // 进入, 相当于sc在增加
            slidingWindow.offerLast(i);
            if(count[ci = schars[i] - 'a'] == 0) {
                upDiff++;
            } else if(count[ci] > 0) {
                upDiff++;
            } else {
                downDiff++;
            }
            count[ci]++;
        }

        // 结算滑动窗口内最后的字符串
        if(upDiff == 0 && downDiff == 0) {
            res.add(slidingWindow.peekFirst());
        }

        return res;
    }
}
```

#### 713. 乘积小于 K 的子数组 | medium

##### 2）前缀积 + 滑动窗口 + 双指针 | O（n）

- **思路**：同《2302. 第 80 双周赛 - 统计得分小于 K 的子数组数目》，使用双指针来替代双端队列，从而实现一个滑动窗口，滑动窗口内的元素都是符合条件的元素，最后通过公式 r - l + 1 来计算窗口内的子数组个数即可，证明见 2302 题。
- **结论**：时间，5 ms，38.55%，空间，48.3 mb，5.37%，时间上，由于只需要遍历一次数组，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int numSubarrayProductLessThanK(int[] nums, int k) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        int l = 0, r = 0, cnt = 0;
        long mulSum = 1;
        while(r < nums.length) {
            // 入队, 并累加前缀积
            mulSum *= nums[r];

            // 如果[l, r]内元素的乘积, 超出了k, 那么就右移l, 以减少乘积, 直到符合条件
            while(l < nums.length && mulSum >= k) {
                // 由于nums[i]必定大于1, 所以直接除就好, 不用考虑除0的情况
                mulSum /= nums[l++];
            }

            // 如果当前[l, r]窗口内还有元素, 那么结算窗口内子数组的数量, 公式=r-l+1
            if(r >= l) {
                cnt += r - l + 1;
            }

            // 继续看下一个以r结尾的数组
            r++;
        }

        return cnt;
    }
}
```

#### 2264. 第 292 场周赛 - 字符串中最大的 3 位相同数字 | easy

##### 1）暴力 + 类滑动窗口 | O（n）

- **思路**：类似于滑动窗口，遍历数组，然后碰到三个重复的就再判断大小，由于字符都一样，所以只需要判断一个就够了，然后再决定 i 跳几步就好。
- **结论**：时间，0 ms，100%，空间，39.7 mb，85.59%，时间上，由于只需要遍历一次，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public String largestGoodInteger(String num) {
        if(num == null || num.length() < 3) {
            return "";
        }

        int i = 0, maxi = -1;
        char[] chars = num.toCharArray();
        while(i < chars.length - 2) {
            if(chars[i] == chars[i + 1] && chars[i] == chars[i + 2]) {
                if(maxi == -1 || chars[i] > chars[maxi]) {
                    maxi = i;
                }
                i += 3;
            } else {
                i++;
            }
        }
     
        return maxi == -1? "" : num.substring(maxi, maxi + 3);
    }
}
```

##### 2）枚举法 | O（10 * n）

- **思路**：由于只是数字，所以从小到大遍历到 10，然后每步组装 3 个相同数字的字符串，判断是否再 num 存在即可。
- **结论**：时间，6 ms，31.22%，空间，40.1 mb，57.29%，时间上，由于要遍历 10 次，且每次都需要从头遍历 num 字符串，所以时间复杂度为 O（10 * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度位 O（1）。

```java
class Solution {
    public String largestGoodInteger(String num) {
        if(num == null || num.length() < 3) {
            return "";
        }

        String res = "", tmp = res;
        for(int i = 0; i < 10; i++) {
            tmp = "" + i + i + i;
            if(num.contains(tmp)) {
                res = tmp;
            }
        }

        return res;
    }
}
```

#### 2269. 第 78 双周赛 - 找到一个数字的 k 美丽值 | easy

##### 1）定长滑动窗口法 | O（n - k - 1）

- **思路**：给定一个滑动窗口 [l, r]，然后不断移动并统计窗口内的数字，是否能整除 num 。
- **结论**：时间，0 ms，100%，空间，38.3 mb，67.96%，时间上，由于从 k 开始，遍历到 n - 1，所以时间复杂度为 O（n - k - 1），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int divisorSubstrings(int num, int k) {
        String s = String.valueOf(num);
        
        int len = s.length(), l = 0, r = k - 1, count = 0, tmp;
        while(r < len) {
            tmp = Integer.parseInt(s.substring(l, r + 1));
            if(tmp != 0 && num % tmp == 0) {
                count++;
            }

            l++;
            r++;
        }

        return count;
    }
}
```

#### 2271. 第 78 双周赛 - 毯子覆盖的最多白色砖块数 | medium

##### 1）滑动窗口 + 双端队列 | O（n * logn + m * n）

- **思路**：
  1. 先按白色方块 int[] 开头 int[0] 进行顺序排序。
  2. 然后遍历 int[] 中的每个小白色方块，如果小白色方块 - 队头 < 地毯长度，那么说明即使当前元素入队后，地毯也还是能够完全覆盖的，所以，从队尾加入一个元素。
  3. 否则，说明地毯将不能完全覆盖，此时需要从队头移出一个元素，队尾加入一个元素。
- **结论**：超出内存限制，时间上，排序花费 O（n * logn），遍历 tiles 花费 O（m * n），所以时间复杂度为 O（n * logn + m * n），空间上，快速排序递归栈最深为 O（log m），设 tiles 最大的末尾元素为 t，那么最大的额外空间复杂度为 O（log m + t），这样用是非常耗内存的，因为把每个小白色方块加入了队列中，所以需要双指针的滑动窗口进行优化。

```java
class Solution {
    public int maximumWhiteTiles(int[][] tiles, int carpetLen) {
        if(tiles == null || tiles.length == 0 || tiles[0] == null || tiles[0].length == 0) {
            return 0;
        }

       	// 先对白色块进行顺序排序
        Arrays.sort(tiles, (o1, o2) -> o1[0] - o2[0]);

        // 初始化白色块哈希表
        int max = 0, x, y;
        Deque<Integer> carpetDeque = new LinkedList<>();
        for(int i = 0; i < tiles.length; i++) {
            x = tiles[i][0];
            y = tiles[i][1];

            for(int j = x; j <= y; j++) {
                // 队列为空, 或者队列还没达到最长时, 则直接加入
                if(carpetDeque.isEmpty() || ( j - carpetDeque.peekFirst() ) < carpetLen) {
                    carpetDeque.offerLast(j);               
                } 
                // 队列达到最长时, 则让出队头, 再从队尾加入
                else {
                    carpetDeque.pollFirst();
                    carpetDeque.offerLast(j);
                }

                max = Math.max(max, carpetDeque.size());
            }
        }

        return max;
    }
}
```

##### 2）变长滑动窗口 | O（n * logn + 2 * m）

- **思路**：
  1. 先按白色方块 int[] 开头 int[0] 进行顺序排序。
  2. 然后，采用双指针的方式，进行移动滑动窗口的左边界和右边界，从而实现变长滑动窗口，分为几种情况：
     - 1）如果某个白色方块能被毯子能够完全覆盖的，则累加整段格子的长度。
     - 2）如果右窗口已经移动到了 tiles 的尽头，则说明毯子能够完全覆盖所有格子，此时返回当前累加到的所有格子之和即可。
     - 3）否则，说明毯子不能完全覆盖某个白色方块，即需要累加多余的一小段，但要注意，这个小段不能累加到 sum 中，因为不能被后面减掉，所以只能作为局部变量累加，Math.max 结算到 max 结果中。
     - 4）最后，到这里，说明以当前左边界的变长滑动窗口已经尝试了最大次数，此时需要继续右移左边界，尝试其他区间，但在移动之前，需要先减去当前左边界起使部分的长度，剩余的则作为下一个区间的计算，以节省计算次数。
- **结论**：时间，43 ms，96.19%，空间，60.5 mb，56.90%，时间上，排序花费 O（n * logn），滑动窗口遍历过程，需要右边界从左尝试到右，花费 O（m），还需要左边界也从左尝试到右，也花费 O（m），所以时间复杂度为 O（n * logn + 2 * m），空间上，由于快速排序递归栈深度为 O（log m），所以额外空间复杂度为 O（log m）。

```java
class Solution {
    public int maximumWhiteTiles(int[][] tiles, int carpetLen) {
        if(tiles == null || tiles.length == 0 || tiles[0] == null || tiles[0].length == 0) {
            return 0;
        }

       	// 先对白色块进行顺序排序
        Arrays.sort(tiles, (o1, o2) -> o1[0] - o2[0]);

        // 变长滑动窗口
        int cpi, l = 0, r = l, lensum = 0, max = lensum;
        while(l < tiles.length) {
            // 毯子以x为基准, 能碰到的最远索引
            cpi = tiles[l][0] + carpetLen - 1;

            // 能够完全覆盖的, 则累加整段格子的长度
            while(r < tiles.length && tiles[r][1] <= cpi) {
                lensum += tiles[r][1] - tiles[r][0] + 1;
                r++;
            }
            
            // 如果是右窗口移到了尽头, 则说明毯子能够完全覆盖所有格子, 此时返回即可
            if(r == tiles.length) {
                max = Math.max(max, lensum);
                break;
            }
            
            // 如果是不能完全覆盖, 则累加多余的一小段, 注意小段不能累加到sum中, 因为减不掉
            max = Math.max(max, lensum + Math.max(0, cpi - tiles[r][0] + 1));

            // 减去l部分的一段长度, 开始下一个l的计算
            lensum -= tiles[l][1] - tiles[l][0] + 1;
            l++;
        }

        return max;
    }
}
```

#### 2302. 第 80 双周赛 - 统计得分小于 K 的子数组数目 | hard

##### 1）前缀和 + 滑动窗口 + 双端队列 | O（n）

- **思路**：
  1. 根据题意，可发现，如果 [l, r] 的数组符合条件，那它们里面的子数组也统统符合条件，所以只需要统计一次 [l, r] 的和即可，里面的子数组肯定符合条件，所以无需计算。其中这里的和统计，用到了前缀和，经过可以只使用一个 sum 变量，来替代前缀和数组，节省了 n 长的额外空间。
  2. 而至于统计窗口内的子数组数量，为了不重复计算，这里统一用以 r 结尾的数组，来计算其内部的子数组个数，公式为 r - l + 1，即以 r 结尾的子数组数量，刚好等于元素个数。
  3. 可证明，使用这种方式统计，由于每个符合条件的 r，都会计算一遍，所以不会漏算，又由于每轮的 r 结尾不一样，所以每轮统计的子数组也不一样，所以不会重算，因此，统计方式是正确的。
- **结论**：时间，14 ms，30.51%，空间，54.6 mb，5.00%，时间上，由于遍历一次数组，所以时间复杂度为 O（n），空间上，由于使用了一个 n 长的双端队列，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public long countSubarrays(int[] nums, long k) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        long cnt = 0, sum = 0;
        Deque<Integer> deque = new LinkedList<>();
        for(int i = 0; i < nums.length; i++) {
            // 入队, 并累加前缀和
            deque.offerLast(i);
            sum += nums[i];

            // 如果窗口内的以r结尾的子数组不符合条件, 那么从左出队, 减少该子数组的长度, 直到符合条件
            while(!deque.isEmpty() && sum * deque.size() >= k) {
                sum -= nums[deque.pollFirst()];
            }

            // 如果窗口内的以r结尾的子数组还有元素, 那么结算窗口内子数组的数量, 公式=r-l+1
            if(!deque.isEmpty()) {
                cnt += deque.peekLast() - deque.peekFirst() + 1;
            }
        }

        return cnt;
    }
}
```

##### 2）前缀和 + 滑动窗口 + 双指针 | O（n）

- **思路**：根据 1 的思路，就可改写双端队列成为双指针，以进一步减少 n 长双端队列的使用。
- **结论**：时间，3 ms，86.41%，空间，51.1 mb，42.65%，时间上，由于只需要遍历一次数组，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public long countSubarrays(int[] nums, long k) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        int l = 0, r = 0;
        long cnt = 0, sum = 0;
        while(r < nums.length) {
            // 入队, 并累加前缀和
            sum += nums[r];

            // 如果窗口内的以r结尾的子数组不符合条件, 那么从左出队, 减少该子数组的长度, 直到符合条件
            while(l < nums.length && sum * (r - l + 1) >= k) {
                sum -= nums[l++];
            }

            // 如果窗口内的以r结尾的子数组还有元素, 那么结算窗口内子数组的数量, 公式=r-l+1
            if(r >= l) {
                cnt += r - l + 1;
            }

            // 继续看下一个以r结尾的数组
            r++;
        }

        return cnt;
    }
}
```

### 1.6. 数据结构 - 单调栈

#### 56. 合并区间 | medium

##### 1）单调栈法 | O（n * logn + 2 * n）

- **思路**：
  1. 先对 intervals，按 `intervals[i][0]` 顺序排序，再按 `intervals[i][1]` 顺序排序。
  2. 从左往右遍历，如果发现栈顶元素 top[1] >= 当前遍历的元素 `intervals[i][0]`，则出栈、合并完区间后再入栈。其中，合并这个区间时，左边界取 top[0]，右边界需要考虑 top[1] 和 `intervals[i][1]` 的最大值，以保证右小区间也能合并入左大区间中。
  3. 最后，再逆向收集栈结果并即可。
- **结论**：时间，7 ms，66.86 %，空间，46.4 mb，36.88%，时间上，排序花费 O（n * logn），遍历入栈需要花费 O（n），最后收集结果遍历栈需要花费 O（n），所以整体的时间复杂度为 O（n * logn + 2 * n），空间上，由于快速排序递归栈深度为 log n，以及使用了一个 n 长的单调栈，所以额外空间复杂度为 O（logn + n）。

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        // 先对intervals, 按intervals[i][0]顺序排序, 再按intervals[i][1]顺序排序
        Arrays.sort(intervals, (o1, o2) -> {
            if(o1[0] != o2[0]) {
                return o1[0] - o2[0];
            } else {
                return o1[1] - o2[1];
            }
        });

        // 从左往右遍历, 如果发现栈顶元素top[1] >= 当前遍历的元素intervals[i][0], 则出栈、合并完区间后再入栈
        LinkedList<int[]> stack = new LinkedList<>();
        for(int i = 0; i < intervals.length; i++) {
            if(stack.isEmpty()) {
                stack.push(intervals[i]);
            } else {
                int[] top = stack.peek();
                if(top[1] >= intervals[i][0]) {
                    stack.pop();
                    stack.push(new int[] { top[0], Math.max(top[1], intervals[i][1]) });
                } else {
                    stack.push(intervals[i]);
                }
            }
        }

        // 最后，再逆向收集栈结果并即可
        int[][] res = new int[stack.size()][2];
        for(int i = res.length - 1; i > -1; i--) {
            res[i] = stack.pop();
        }

        return res;
    }
}
```

#### 57. 插入区间 | medium

##### 1）二分查找 + 数组拷贝 + 单调栈法 | O（logn + 3 * n）

- **思路**：
  1. 由于原列表是有序列表，所以可以使用二分查找，找到第一个比 newInterval 大的索引 idx，即要么 x 大，或者 x 相同时 y 大。
  2. 然后，拷贝 idx 前后的元素到新数组 newIntervals，并空出 idx 位置，以插入新元素 newInterval。
  3. 最后，参考《56. 合并区间》的思路，合并新数组 newIntervals 的区间，返回合并后的区间即是答案，其中要注意的是，单存地合并区间，是需要先排序的，但由于这里本身 newIntervals 就是有序的，所以可以在合并操作中，额外去掉排序的操作。
- **结论**：时间，1 ms，84.98%，空间，43.8 mb，29.62%，时间上，二分查找花费 O（logn），拷贝元素到新数组花费 O（n），合并区间花费 O（2 * n），所以整体的时间复杂度为 O（logn + 3 * n），空间上，由于使用了一张 n 长的新数组，以及合并区间中使用了一个 n 长的单调栈，所以额外空间复杂度为 O（2 * n）。

```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        // 二分查找, 找到第一个比newInterval大的索引(x大, 或者x相同y大)
        int idx = findFirstGtIndex(intervals, newInterval);

        // 先拷贝前面的
        int n = intervals.length;
        int[][] newIntervals = new int[n + 1][2];
        System.arraycopy(intervals, 0, newIntervals, 0, idx);

        // 再拷贝后面的
        newIntervals[idx] = newInterval;
        System.arraycopy(intervals, idx, newIntervals, idx + 1, n - idx);

        // 最后再合并区间, 注意, 这里对比合并区间版本的, 额外去掉了排序操作
        return merge(newIntervals);
    }

    // 二分查找, 找到第一个比newInterval大的索引(x大, 或者x相同y大)
    private int findFirstGtIndex(int[][] intervals, int[] target) {
        int l = -1, r = intervals.length, mid;
        while(l + 1 != r) {
            mid = l + (r - l) / 2;
            
            // 红色区域
            int[] e = intervals[mid];
            if(e[0] > target[0] || (e[0] == target[0] && e[1] > target[1])) {
                r = mid;
            } 
            // 蓝色区域
            else {
                l = mid;
            }
        }

        return  r;
    }

    private int[][] merge(int[][] intervals) {
 // 从左往右遍历, 如果发现栈顶元素top[1] >= 当前遍历的元素intervals[i][0], 则出栈、合并完区间后再入栈
        LinkedList<int[]> stack = new LinkedList<>();
        for(int i = 0; i < intervals.length; i++) {
            if(stack.isEmpty()) {
                stack.push(intervals[i]);
            } else {
                int[] top = stack.peek();
                if(top[1] >= intervals[i][0]) {
                    stack.pop();
                    stack.push(new int[] { top[0], Math.max(top[1], intervals[i][1]) });
                } else {
                    stack.push(intervals[i]);
                }
            }
        }

        // 最后，再逆向收集栈结果并即可
        int[][] res = new int[stack.size()][2];
        for(int i = res.length - 1; i > -1; i--) {
            res[i] = stack.pop();
        }

        return res;
    }
}
```

##### 2）单调栈法 | O（2 * n）

- **思路**：参考《56. 合并区间》的思路，视 newInterval 为手上的区间，每次合并数组的区间前，先与手上的 newInterval 比较，看看 newInterval 比数组的小，如果是的话，那么需要合并手上的，再更新手上的，否则由于数组本身就是顺序且无重叠的，所以直接入栈即可，最后把栈的元素倒出来，逆序放到结果中即可。
- **结论**：时间，1 ms，84.98%，空间，43.8 mb，29.02%，时间上，遍历入栈需要花费 O（n），最后收集结果遍历栈需要花费 O（n），所以整体的时间复杂度为 O（2 * n），空间上，由于使用了一个 n 长的单调栈，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        if(intervals.length == 0) {
            return new int[][] { newInterval };
        }

        // 从左到右遍历, 判断栈顶元素, 与手上的元素以及当前元素的区间大小, 看是否需要出栈、合并区间后再入栈
        LinkedList<int[]> stack = new LinkedList<>();
        for(int i = 0; i < intervals.length; i++) {
            // 与手上的newInterval对比, intervals[i]小, 由于intervals本身无重叠, 直接跳过就好
            if(leftIsGt(newInterval, intervals[i])) {
                stack.push(intervals[i]);
                continue;
            }

            // intervals[i]大, 则交换newInterval和intervals[i]
            int[] tmp = intervals[i];
            intervals[i] = newInterval;
            newInterval = tmp;
            
            // 开始与intervals[i]合并区间
            if(stack.isEmpty()) {
                stack.push(intervals[i]);
            } else {
                int[] top = stack.peek();
                if(top[1] >= intervals[i][0]) {
                    stack.pop();
                    stack.push(new int[] { top[0], Math.max(top[1], intervals[i][1]) });
                } else {
                    stack.push(intervals[i]);
                }
            }
        }

        // 数组中的遍历完后, 再合并手上的, 如果栈顶比手上的newInterval大, 则合并手上的再放入栈中
        int[] top = stack.peek();
        if(top[1] >= newInterval[0]) {
            stack.pop();
            stack.push(new int[] { top[0], Math.max(top[1], newInterval[1]) });
        }
        // 否则, 直接手上的直接入栈
        else {
            stack.push(newInterval);
        }

        // 最后, 再逆向收集栈结果并即可
        int[][] res = new int[stack.size()][2];
        for(int i = res.length - 1; i > -1; i--) {
            res[i] = stack.pop();
        }

        return res;
    }

    private boolean leftIsGt(int[] left, int[] right) {
        if(left[0] < right[0]) {
            return false;
        }
        return left[0] > right[0] || left[1] >= right[1];
    }
}
```

##### 3）模拟法 | O（2 * n）

- **思路**：见代码注释，不再用单调栈的套路，而是具体情况具体分析。
- **结论**：时间，0 ms，100%，空间，43.7 mb，45.75%，时间上，遍历原数组花费 O（n），组装返回结果花费 O（n），所以整体时间复杂度为 O（2 * n），空间上，由于使用了一张 n 长的 list 数组（合并后的区间个数未知，不能直接使用 res 存储），所以额外空间复杂度为 O（n）。

```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        boolean hasPlace = false;
        int l = newInterval[0], r = newInterval[1], n = intervals.length;
        List<int[]> list = new ArrayList<>();
        for(int i = 0; i < n; i++) {
            int[] e = intervals[i];
            int il = e[0], ir = e[1];

            // 1、如果ir比l小, 说明i在newInterval前, 且没冲突
            if(ir < l) {
                list.add(e);
            }
            // 2、如果il比r大, 说明i在newInterval后, 且没冲突
            else if(il > r){
                // 5.1、然后再在下一轮, 找到第一个il比r大的地方, 放入合并后的区间
                if(!hasPlace) {
                    list.add(new int[] { l, r });
                    hasPlace = true;
                }

                // 5.2、无论要不要放入合并区间, 都依然放入当前没冲突的区间
                list.add(e);
            }
            // 3、如果ir比l大, 或者il比r小, 说明i与newInterval发生冲突, 则需要合并区间
            else {
                // 4、通过min{il, l}合并左区间, max{ir, r}合并右区间
                l = Math.min(il, l);
                r = Math.max(ir, r);
            }
        }

        // 6、如果数组遍历完, 都没找到放入合并后的区间, 或者无需合并区间, 则只能放到list末尾了
        if(!hasPlace) {
            list.add(new int[] { l, r });
        }

        // 7、最后, 收集list元素到res中并返回即可
        int[][] res = new int[list.size()][2];
        for(int i = 0; i < list.size(); i++) {
            res[i] = list.get(i);
        }

        return res;
    }
}
```

#### 84. 柱状图中最大的矩形 | hard

##### 1）暴力解法1 | O（n^2）

- **思路**：
  1. 宽度不固定，高度也不固定。
  2. 从左遍历到右先固定 1 位数字作为左边界，然后选择一个右边界再从左遍历到右，矩形的面积 = 宽 * 高，其中高指的是遍历过程中最小的高度 limit，然后周而复始，选出最大的一个矩形即可。
- **结论**：执行超时，因为重复计算了很多次无用的，比如右边界遍历过程中，明知道某个位置的值已经是最小高度了，下次还要用它作为左边界再算一遍，浪费了很多时间。

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        if(heights == null || heights.length == 0) {
            return 0;
        }

        int max = Integer.MIN_VALUE, limit;
        for(int i = 0; i < heights.length; i++) {
            limit = Integer.MAX_VALUE;
            for(int j = i; j < heights.length; j++) {
                limit = Math.min(limit, Math.min(heights[i], heights[j]));
                max = Math.max(max, (j - i + 1) * limit);
            }
        }

        return max;
    }
}
```

##### 2）暴力解法2 | O（n^2）

- **思路**：
  1. 高度固定，宽度不固定。
  2. 从左到右选择每一个高度进行尝试，其最大宽度等于高度变小前的索引中间的那部分，面积 = 高度 * 宽度。
- **结论**：执行也超时，但通过用例比暴力解法 1 的多了 3 个，明显该算法的效率高一点。

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        if(heights == null || heights.length == 0) {
            return 0;
        }

        int max = Integer.MIN_VALUE, limit, l, r;
        for(int i = 0; i < heights.length; i++) {
            limit = heights[i];
            l = r = i;
            while(l - 1 > -1 && heights[l - 1] >= heights[i]) {
                l--;
            }
            while(r + 1 < heights.length && heights[r + 1] >= heights[i]) {
                r++;
            }
            max = Math.max(max, (r - l + 1) * limit);
        }

        return max;
    }
}
```

##### 3）单调栈法 | O（n）

- **思路**：
  1. 经过研究暴力解法的思路可知，固定一个高度 i 后，如果没确定最大宽度有多少，可以先把它放入栈中，继续遍历。
  2. 等遍历到它的边界，即高度开始减小时，则把它出栈并计算以该值为高度，以当前索引 i 左右它的最右边界（右边开始下降，最右默认为 len），以剩余栈顶 j 作为它的最左边界（左边开始下降，最左默认为 -1），中间作为该高度的最大宽度，再高度 * 宽度，得出该高度的最大矩形面积。
  3. 周而复始，计算出每个值作为高度的最大矩形面积，返回最大值即可。
- **结论**：时间，20ms，67.94%，空间，46.7mb，99.21%，效率可以了。

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        if(heights == null || heights.length == 0) {
            return 0;
        }

        // 单调栈: 底->顶, 高度从小到大
        LinkedList<Integer> stack = new LinkedList<>();
        stack.push(-1);

        // 遍历高度数组
        int max = Integer.MIN_VALUE, i = 0, top;
        while(i < heights.length) {
            top = stack.peek();

            // 如果符合单调栈高度从小到大排序的特点, 则加入栈中 
            if(getStackValue(heights, top) <= heights[i]) {
                stack.push(i++);
            } 
            // 如果以高度为i的矩形遇到了瓶颈, 那么则弹出高度i
            else {
                stack.pop();

                // 高度i的矩形最大宽度为上一个最小和当前最小的差(肯定有一个-1再里面)
                max = Math.max(max, heights[top] * (i - stack.peek() - 1));
            }
        }

        // 遍历剩余高度i
        while(!stack.isEmpty() && stack.peek() != -1) {
            top = stack.pop();
            max = Math.max(max, heights[top] * (heights.length - stack.peek() - 1));
        }

        return max;
    }

    private int getStackValue(int[] heights, int top) {
        if(top == -1) {
            return 0;
        } else {
            return heights[top];
        }
    }
}
```

#### 85. 最大矩形 | hard

##### 1）暴力解法 | O（m^2 * n）

- **思路**：
  1. 通过遍历矩阵中的每个元素，求以它 [i，j] 结尾的最大矩形面积。
  2. 其中，一个以 [i，j] 结尾的最大矩形面积可以通过比较每行的宽 * 高得出，而每行的高初始为 1，宽初始为当前位置的 '1' 的累加和，同理，上一层的高为 2，宽则为上一层位置的 '1' 的累加和...，可见，把最大宽度的求解转换为 '1' 的累加和，可以很轻松就求出当前位置结尾的最大矩形面积。
- **结论**：
  1. 时间，16ms，23.43%，空间，41.7mb，48.05%，遍历整个矩阵的元素需要 O（m * n），而求每个元素作为结尾的最大矩形面积，最大需要遍历 m 行，因此，整个时间复杂度最大为 O（m^2 * n）。
  2. 而由于每个元素都求了最大矩形面积，使得很多方块被重新用来计算了，有时候明知道不可能是最大面积了，还要拿来计算，所以，效率还可以继续优化。

```java
class Solution {
    public int maximalRectangle(char[][] matrix) {
        if(matrix == null || matrix.length == 0 || matrix[0].length == 0) {
            return 0;
        }

        // 累加连续1的个数, 遇到0则归零
        int[][] counts = getCounts(matrix);

        // 求以[i,j]结尾的最大矩形面积
        int max = 0;
        for(int i = 0; i < matrix.length; i++) {
            for(int j = 0; j < matrix[0].length; j++) {
                if('1' == matrix[i][j]) {
                    max = Math.max(max, getMax(matrix, counts, i, j));
                }
            }
        }

        return max;
    }

    /** 求以[i,j]结尾的矩形的最大面积 */
    private int getMax(char[][] matrix, int[][] counts, int row, int col) {
        int gao = 1;
        int kuan = counts[row][col];
        int max = kuan * gao;

        while(row - 1 > -1 && counts[row - 1][col] > 0) {
            gao++;
            row--;

            // 如果宽大于上一层的宽, 则取上一层的宽
            if(counts[row][col] < kuan) {
                kuan = counts[row][col];
            }

            max = Integer.max(max, kuan * gao);
        }

        return max;
    }

    /** 累加连续1的个数, 遇到0则归零 */
    private int[][] getCounts(char[][] matrix) {
        int[][] counts = new int[matrix.length][matrix[0].length];

        int sum;
        for(int i = 0; i < matrix.length; i++) {
            sum = 0;
            for(int j = 0; j < matrix[0].length; j++) {
                if('0' == matrix[i][j]) {
                    sum = 0;
                } else {
                    sum += 1;
                }
                counts[i][j] = sum;
            }
        }

        return counts;
    }
}
```

##### 2）单调栈法 | O（m * n）

- **思路**：

  1. 观察暴力解法可以看出，实际上等同于在**横向**做了多层柱状图的判断，然后每层的判断又回到了《柱状图中最大的矩形》中的暴力解法，花费了 O（ n^2）的时间，因此，可以参考当时的单调栈解法，优化每层的判断最大矩形为花费 O（n）。

  2. 为了直观理解，这里把二维矩阵看作成**竖向**的多层柱状图，然后调用《柱状图中最大的矩形》中的单调栈解法即可解决问题，比如第三层的竖向状态图为，其他层也同理：

     ![1642483436060](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1642483436060.png)

- **结论**：时间，5ms，89.77%，空间，43.4mb，5.05%，由于需要遍历 m 层，每层判断柱状图的最大矩形需要花费 O（n），因此总的时间复杂度为 O（m * n），额外空间花费了若干个变量和二维数组，共 O（m * n）。

```java
class Solution {
    public int maximalRectangle(char[][] matrix) {
        if(matrix == null || matrix.length == 0 || matrix[0].length == 0) {
            return 0;
        }

        // 竖向: 累加连续1的个数, 遇到0则归零
        int[][] counts = getCounts(matrix);

        // 计算每层的heights, 然后调用上一题的函数
        int max = 0;
        for(int i = 0; i < matrix.length; i++) {
            max = Math.max(max, largestRectangleArea(counts[i]));
        }

        return max;
    }

    /** 竖向: 累加连续1的个数, 遇到0则归零 */
    private int[][] getCounts(char[][] matrix) {
        int[][] counts = new int[matrix.length][matrix[0].length];

        int sum;
        for(int j = 0; j < matrix[0].length; j++) {
            sum = 0;
            for(int i = 0; i < matrix.length; i++) {
                if('0' == matrix[i][j]) {
                    sum = 0;
                } else {
                    sum += 1;
                }
                counts[i][j] = sum;
            }
        }

        return counts;
    }

    private int largestRectangleArea(int[] heights) {
        if(heights == null || heights.length == 0) {
            return 0;
        }

        // 单调栈: 底->顶, 高度从小到大
        LinkedList<Integer> stack = new LinkedList<>();
        stack.push(-1);

        // 遍历高度数组
        int max = Integer.MIN_VALUE, i = 0, top;
        while(i < heights.length) {
            top = stack.peek();

            // 如果符合单调栈高度从小到大排序的特点, 则加入栈中 
            if(getStackValue(heights, top) <= heights[i]) {
                stack.push(i++);
            } 
            // 如果以高度为i的矩形遇到了瓶颈, 那么则弹出高度i
            else {
                stack.pop();

                // 高度i的矩形最大宽度为上一个最小和当前最小的差(肯定有一个-1再里面)
                max = Math.max(max, heights[top] * (i - stack.peek() - 1));
            }
        }

        // 遍历剩余高度i
        while(!stack.isEmpty() && stack.peek() != -1) {
            top = stack.pop();
            max = Math.max(max, heights[top] * (heights.length - stack.peek() - 1));
        }

        return max;
    }

    private int getStackValue(int[] heights, int top) {
        if(top == -1) {
            return 0;
        } else {
            return heights[top];
        }
    }
}
```

#### 155. 最小栈 | easy

##### 1）单调栈法 | O（1）

- **思路**：维护两个栈，一个数据栈，用于记录数据的存放顺序，所有添加的值都要入栈；一个最小单调栈，用于记录最小值，只有在添加的值小于等于单调栈顶时才能入单调栈，否则忽略那个值。
- **结论**：时间，4ms，99.03%，空间，40.5mb，5.01%，效率非常高，就这样吧~

```java
class MinStack {

    private LinkedList<Integer> dataStack;
    private LinkedList<Integer> minStack;

    public MinStack() {
        dataStack = new LinkedList<>();
        minStack = new LinkedList<>();
    }
    
    public void push(int val) {
        dataStack.push(val);
        if(minStack.isEmpty()) {
            minStack.push(val);
        } else {
            if(val <= minStack.peek()) {
                minStack.push(val);
            }
        }
    }
    
    public void pop() {
        int val = dataStack.pop();
        if(val == minStack.peek()) {
            minStack.pop();
        }
    }
    
    public int top() {
        return dataStack.peek();
    }
    
    public int getMin() {
        return minStack.peek();
    }
}

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack obj = new MinStack();
 * obj.push(val);
 * obj.pop();
 * int param_3 = obj.top();
 * int param_4 = obj.getMin();
 */
```

#### 221. 最大正方形 | medium

##### 1）暴力解法 | O（m^2 * n）

- **思路**：在《单调栈 - 最大矩形》的暴力解法基础上，改写了 getMax 方法，从获取矩形面积=高 * 宽，改为获取正方形面积=min{高，宽} ^ 2。 
- **结论**：
  1. 时间，56ms，5.02%，空间，53.9mb，5.03%，时间上，遍历整个矩阵的元素需要 O（m * n），而求每个元素作为结尾的最大正方形面积，最大需要遍历 m 行，因此，整个时间复杂度最大为 O（m^2 * n）。
  2. 而由于每个元素都求了最大正方形面积，使得很多方块被重新用来计算了，有时候明知道不可能是最大面积了，还要拿来计算，所以，效率还可以继续优化。

```java
class Solution {
    public int maximalSquare(char[][] matrix) {
        if(matrix == null || matrix.length == 0 || matrix[0].length == 0) {
            return 0;
        }

        // 累加连续1的个数, 遇到0则归零
        int[][] counts = getCounts(matrix);

        // 求以[i,j]结尾的最大矩形面积
        int max = 0;
        for(int i = 0; i < matrix.length; i++) {
            for(int j = 0; j < matrix[0].length; j++) {
                if('1' == matrix[i][j]) {
                    max = Math.max(max, getMax(matrix, counts, i, j));
                }
            }
        }

        return max;
    }

    /** 求以[i,j]结尾的正方形的最大面积 */
    private int getMax(char[][] matrix, int[][] counts, int row, int col) {
        int gao = 1, kuan = counts[row][col];
        int bian = Math.min(gao, kuan), max = bian * bian;
        while(row - 1 > -1 && counts[row - 1][col] > 0) {
            gao++;
            row--;

            // 如果宽大于上一层的宽, 则取上一层的宽
            if(counts[row][col] < kuan) {
                kuan = counts[row][col];
            }

            bian = Math.min(gao, kuan);
            max = Integer.max(max, bian * bian);
        }

        return max;
    }

    /** 累加连续1的个数, 遇到0则归零 */
    private int[][] getCounts(char[][] matrix) {
        int[][] counts = new int[matrix.length][matrix[0].length];

        int sum;
        for(int i = 0; i < matrix.length; i++) {
            sum = 0;
            for(int j = 0; j < matrix[0].length; j++) {
                if('0' == matrix[i][j]) {
                    sum = 0;
                } else {
                    sum += 1;
                }
                counts[i][j] = sum;
            }
        }

        return counts;
    }
}
```

##### 2）单调栈法 | O（m * n）

- **思路**：

  1. 观察暴力解法可以看出，实际上等同于在**横向**做了多层柱状图的判断，然后每层的判断又回到了《柱状图中最大的矩形》中的暴力解法，花费了 O（n^2）的时间，因此，可以参考当时的单调栈解法，优化每层的判断最大矩形为花费 O（n）。

  2. 为了直观理解，这里把二维矩阵看作成**竖向**的多层柱状图，然后调用《柱状图中最大的矩形》中的单调栈解法即可解决问题，比如第三层的竖向状态图为，其他层也同理：

     ![1642483436060](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1642483436060.png)

  3. 这里求最大正方形只要求出 min{kuan，gao} 作为正方形的边求面积即可。

- **结论**：时间，8ms，14.74%，空间，54.4mb，5.03%，由于需要遍历 m 层，每层判断柱状图的最大正方形需要花费 O（n），因此总的时间复杂度为 O（m * n），额外空间花费了若干个变量和二维数组，共 O（m * n）。

```java
class Solution {
    public int maximalSquare(char[][] matrix) {
        if(matrix == null || matrix.length == 0 || matrix[0].length == 0) {
            return 0;
        }

        // 竖向: 累加连续1的个数, 遇到0则归零
        int[][] counts = getCounts(matrix);

        // 计算每层的heights, 然后调用上一题的函数
        int max = 0;
        for(int i = 0; i < matrix.length; i++) {
            max = Math.max(max, largestRectangleArea(counts[i]));
        }

        return max;
    }

    /** 竖向: 累加连续1的个数, 遇到0则归零 */
    private int[][] getCounts(char[][] matrix) {
        int[][] counts = new int[matrix.length][matrix[0].length];

        int sum;
        for(int j = 0; j < matrix[0].length; j++) {
            sum = 0;
            for(int i = 0; i < matrix.length; i++) {
                if('0' == matrix[i][j]) {
                    sum = 0;
                } else {
                    sum += 1;
                }
                counts[i][j] = sum;
            }
        }

        return counts;
    }

    private int largestRectangleArea(int[] heights) {
        if(heights == null || heights.length == 0) {
            return 0;
        }

        // 单调栈: 底->顶, 高度从小到大
        LinkedList<Integer> stack = new LinkedList<>();
        stack.push(-1);

        // 遍历高度数组
        int max = Integer.MIN_VALUE, i = 0, top, bian;
        while(i < heights.length) {
            top = stack.peek();

            // 如果符合单调栈高度从小到大排序的特点, 则加入栈中 
            if(getStackValue(heights, top) <= heights[i]) {
                stack.push(i++);
            } 
            // 如果以高度为i的矩形遇到了瓶颈, 那么则弹出高度i
            else {
                stack.pop();

                // 高度i的矩形最大宽度为上一个最小和当前最小的差(肯定有一个-1再里面)
                bian = Math.min(heights[top], i - stack.peek() - 1);
                max = Math.max(max, bian * bian);
            }
        }

        // 遍历剩余高度i
        while(!stack.isEmpty() && stack.peek() != -1) {
            top = stack.pop();
            bian = Math.min(heights[top], heights.length - stack.peek() - 1);
            max = Math.max(max, bian * bian);
        }

        return max;
    }

    private int getStackValue(int[] heights, int top) {
        if(top == -1) {
            return 0;
        } else {
            return heights[top];
        }
    }
}
```

#### 739. 每日温度 | medium

##### 1）单调栈法 | O（n）

- **思路**：构建单调栈，使得其自底到顶的元素 i 对应的值 v 从大到小排列，这样，每次遍历到的 i2 对应的新值 v2，则与栈顶 itop 对应的值 vtop（栈中最小值）相比，如果 v2 < vtop，那么把 v2 加入栈顶，否则说明 v2 就是 vtop 后面的第一个最高温度，此时弹出 vtop 并设置结果 = i2- itop，直到数组遍历完毕，那么残留再栈中的元素就是后面没有更高温度的元素，所以它们在 res 数组中的值默认为 0。
- **结论**：时间，21ms，93.69%，空间，57.3mb，5.04%，时间上，遍历一次数组需要 O（n），空间上，由于使用了一个单调栈，最多存储 n 个元素，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        if(temperatures == null || temperatures.length == 0) {
            return new int[0];
        }

        // 单调栈: 底到顶=从大到小
        int topi;
        LinkedList<Integer> istack = new LinkedList<>();

        int[] res = new int[temperatures.length];
        for(int i = 0; i < temperatures.length; i++) {
            while(!istack.isEmpty() && temperatures[istack.peek()] < temperatures[i]) {
                topi = istack.pop();
                res[topi] = i - topi;
            }
            istack.push(i);
        }

        return res;
    }
}
```

#### 2289. 第 295 周赛 - 使数组按非递减顺序排列 | medium

##### 1）模拟法 | O（2 * n）

- **思路**：模仿题目的那样操作，每轮都从后往前，剔除 nums[i] < nums[i - 1] 的元素，然后装入到另一个栈中，接着进入下一轮剔除，重新倒回去，周而复始，期间维护好轮数并在最后返回即可。
- **结论**：执行超时，时间上，最坏情况下，每次都需要遍历 n 个元素，但只移除一个元素，且需要移除 n - 1 轮，此时时间复杂度为 O（n^2），空间上，使用了 2 个 n 长的栈，所以额外空间复杂度为 O（2 * n）。

```java
class Solution {
    public int totalSteps(int[] nums) {
        if(nums == null || nums.length <= 1) {
            return 0;
        }

        LinkedList<Integer> stack = new LinkedList<>();
        LinkedList<Integer> revrStack = new LinkedList<>();

        int diff = Integer.MAX_VALUE;
        revrStack.push(nums[0]);
        for(int i = 1; i < nums.length; i++) {
            diff = Math.min(diff, nums[i] - nums[i - 1]);
            revrStack.push(nums[i]);
        }
        
        int res = 0, oldTop, tmp; 
        while(diff < 0) {
            diff = Integer.MAX_VALUE;
            res++;
            while(!revrStack.isEmpty()) {
                oldTop = revrStack.pop();
                if(!revrStack.isEmpty() && (tmp = oldTop - revrStack.peek()) < 0) {
                    diff = Math.min(diff, tmp);
                } else {
                    stack.push(oldTop);
                }
            }
            while(!stack.isEmpty()) {
                revrStack.push(stack.pop());
            }
        }

        return res != 0? res - 1 : res;
    }
}
```

##### 2）单调栈法 | O（n）

- **思路**：
  1. 经过研究数组元素状态发现，比如可把 `9，1，3，2，3，4，1，5` 分为 `9，1`、`3，2`、`3`、`4，1`、 `5` 五个区间，剔除一次后，又变为 `9，3`、`3`、`4`、`5` 四个区间，再剔除一次，则变为 `9，3`、`4`、`5` 三个区间，同理，到最后结果的 `9`，还需要剔除 3 次，累加前 2 次，一共需要剔除 5 次，则恰恰正是正确的结果。
  2. 因此，可见，第一，统计每个区间的最大值，即可知道当前数字在第几轮被删除，第二，如果在当前数字之前存在其他数字被删除的，那么当前数字的删除论数是之前的最大值 + 1，即 `3，2` 和 `3` 属于不同的区间，但最终都是被 `9` 所删除的，所以第二个 `3` 是第一个 `3` 的轮数 + 1。
  3. 所以，算法的思路就是，设计一个单调递减的栈，存放的元素是 int[] {元素值，当前元素被删除的轮数}，然后碰到栈顶比自己小于的或者等于的，则要弹出栈顶元素，开始取最大的轮数 + 1，作为自己的轮数进行栈。
  4. 同时，还合并一个 max 局部变量，用来记录上一个顺序元素的最大轮数，然后 + 1，作为自己的轮数。
  5. 另外，如果碰到栈刚开始为空，或者被弹空的情况，则最大轮数即为 0 入栈，因为这个元素没有前一轮元素，或者根本不会被前一轮删除。
- **结论**：时间，18 ms，77.53%，空间，57.9 mb，7.44%，时间上，由于只需要遍历一次 nums 数组，所以时间复杂度为 O（n），空间上，由于使用了一个 n 长的栈，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public int totalSteps(int[] nums) {
        if(nums == null || nums.length <= 1) {
            return 0;
        }

        // 单调递减栈：从下往上递减
        LinkedList<int[]> stack = new LinkedList<>();

        int res = 0, max;
        for(int i = 0; i < nums.length; i++) {
            max = 0;

            // 小于等于numsp[i]的栈顶, 出栈
            while(!stack.isEmpty() && nums[i] >= stack.peek()[0]) {
                max = Math.max(max, stack.pop()[1]);
            }

            // 如果剩余栈内元素为空, 则当前区间最大值是0, 否则位max+1
            max = stack.isEmpty()? 0 : max + 1;

            // 然后入栈
            stack.push(new int[] {nums[i], max});

            // 最后结算一下结果res
            res = Math.max(res, max);
        }

        return res;
    }
}
```

### 1.7. 数据结构 - 前缀树

#### 17. 电话号码的字母组合 | medium

##### 1）前缀树法 | O（4^n - 2）

- **思路**：首先生成电话字母哈希表，然后遍历原数组，取得电话字母列表，按规则放入前缀树，然后深度优先遍历就好。

  ![1641642900473](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1641642900473.png)

- **结论**：1ms，59.32%，面试应该可以。

```java
class Solution {
    
    // 前缀树节点
    class Node {
        String value;
        Node[] nodes;
    }

    public List<String> letterCombinations(String digits) {
        if(digits == null || digits == "") {
            return null;
        }

        HashMap<Integer, String[]> map = initMap();
        List<String> res = new ArrayList<>();
        
        Node head = new Node();
        head.nodes = new Node[4];
        List<Node> firstNodeList = new ArrayList<>();
        firstNodeList.add(head);

        HashMap<Integer, List<Node>> nodesMap = new HashMap<>(); 
        nodesMap.put(0, firstNodeList);

        String[] contents;
        char[] chars = digits.toCharArray();
        for(int i = 0; i < chars.length; i++) {
            // 根据数组获取电话字母组合
            contents = map.get(chars[i] - '0');
            
            // 寻找同层的待添加节点
            List<Node> nodeList = nodesMap.get(i);

            // 设置同层节点缓存
            if(!nodesMap.containsKey(i+1)) {
                nodesMap.put(i+1, new ArrayList<>());
            }

            // 每个待添加节点都设置相同的前缀树数组
            for(Node node : nodeList) {
                // 遍历电话字母组合
                for(int j = 0; j < contents.length; j++) {
                    if("".equals(contents[j])) {
                        continue;
                    }

                    Node next = new Node();
                    node.nodes[j] = next;
                    nodesMap.get(i+1).add(next);

                    next.value = contents[j];    
                    if(i != chars.length - 1) {
                        next.nodes = new Node[4];
                    }
                }
            }
        }

        // 深度优先遍历前缀树
        f(res, head, "");
        return res;
    }

    private void f(List<String> res, Node head, String cur) {
        if(head.nodes == null) {
            res.add(cur);
            return;
        }

        for(int i = 0; i < head.nodes.length; i++) {
            if(head.nodes[i] != null) {
                f(res, head.nodes[i], cur + head.nodes[i].value);
            }
        }
    }

    private HashMap<Integer, String[]> initMap() {
        HashMap<Integer, String[]> map = new HashMap<>();

        map.put(2, new String[] {
            "a", "b", "c", ""
        });
        map.put(3, new String[] {
            "d", "e", "f", ""
        });
        map.put(4, new String[] {
            "g", "h", "i", ""
        });
        map.put(5, new String[] {
            "j", "k", "l", ""
        });
        map.put(6, new String[] {
            "m", "n", "o", ""
        });
        map.put(7, new String[] {
            "p", "q", "r", "s"
        });
        map.put(8, new String[] {
            "t", "u", "v", ""
        });
        map.put(9, new String[] {
            "w", "x", "y", "z"
        });

        return map;
    }
}
```

#### 208. 实现 Trie（前缀树） | medium

##### 1）前缀树法 | O（n）

- **思路**：使用通过前缀树套路，通过 Node#value 表示点值，Node#nodes 表示下一个点值，把字符串拆开一个个字符，分别放入一层层的 Node 节点中即可，其中，nodes 使用了数组的形式，可以减少空间的使用和加快访问的效率。
- **结论**：时间，32ms，82.15%，空间，52.1mb，6.47%，时间上，由于每次都只需要遍历字符串的长度，所以时间复杂度为 O（n），空间上，由于可能输入 n 次字符串，记每次字符串长度为 m，因此，额外空间复杂度为 O（n * m * 26）。

```java
class Trie {

    class Node {
        char value;
        Node[] nodes;
        int endCount = 0;

        Node() {
            
        }

        Node(char value) {
            this.value = value;
        }
    }

    private Node head;

    public Trie() {
        head = new Node();
        head.nodes = new Node[26];
    }
    
    public void insert(String word) {
        if(word == null || word.length() == 0) {
            return;
        }

        Node node, pnode = head;
        char[] chars = word.toCharArray();
        for(int i = 0; i < chars.length; i++) {
            node = pnode.nodes[chars[i] - 'a'];
            if(node == null) {
                node = new Node(chars[i]);
                node.nodes = new Node[26];
                pnode.nodes[chars[i] - 'a'] = node;
            }
            if(i == chars.length-1) {
                node.endCount += 1;
            }

            pnode = node;
        }
    }
    
    public boolean search(String word) {
        if(word == null || word.length() == 0) {
            return true;
        }

        Node nextnode = head;
        char[] chars = word.toCharArray();
        for(int i = 0; i < chars.length; i++) {
            nextnode = nextnode.nodes[chars[i] - 'a'];
            if(nextnode == null) {
                return false;
            }
        }

        return nextnode.endCount > 0;
    }
    
    public boolean startsWith(String prefix) {
        if(prefix == null || prefix.length() == 0) {
            return true;
        }

        Node nextnode = head;
        char[] chars = prefix.toCharArray();
        for(int i = 0; i < chars.length; i++) {
            nextnode = nextnode.nodes[chars[i] - 'a'];
            if(nextnode == null) {
                return false;
            }
        }

        return true;
    }
}

/**
 * Your Trie object will be instantiated and called as such:
 * Trie obj = new Trie();
 * obj.insert(word);
 * boolean param_2 = obj.search(word);
 * boolean param_3 = obj.startsWith(prefix);
 */
```

### 1.8. 数据结构 - 前缀和 | 后缀和

#### 238. 除自身外数组的乘积 | medium

##### 1）前缀积后缀积法 | O（3 * n）

- **思路**：根据题意，不能使用除法，且要在 O（n）时间内完成计算，经过研究可得知，每个元素除自身以外的乘积 = 它的前缀元素之积 * 它的后缀元素之积，因此，只要先提前算好前缀积、后缀积，即可得到答案。
- **结论**：时间，1 ms，100%，空间，50.3 mb，5.00%，时间上，由于需要遍历 3 次数组，所以时间复杂度为 O（3 * n），空间上，由于使用了一张 n 长的前缀积数组，一张 n 长的后缀积数组，所以额外空间复杂度为 O（2 * n）。

```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int n = nums.length;

        // 构造前缀积数组, 其中preMuls[0] = 1
        int[] preMuls = new int[n];
        preMuls[0] = 1;
        for(int i = 1; i < n; i++) {
            preMuls[i] = preMuls[i - 1] * nums[i - 1];
        }

        // 构造后缀积数组, 其中sufMuls[n - 1] = 1
        int[] sufMuls = new int[n];
        sufMuls[n - 1] = 1;
        for(int i = n - 2; i > -1; i--) {
            sufMuls[i] = sufMuls[i + 1] * nums[i + 1];
        }

        // 最后遍历前缀/后缀积数组, ans[i] = preMuls[i] * sufMuls[i]
        int[] ans = new int[n];
        for(int i = 0; i < n; i++) {
            ans[i] = preMuls[i] * sufMuls[i];
        }

        return ans;
    }
}
```

##### 2）前缀积后缀积法 + 空间压缩优化 | O（2 * n）

- **思路**：根据以往前缀和的经验，像这种前缀积、后缀积数组是可以通过一个临时变量来替代的，把临时变量看作上一次的前缀/后缀积，然后计算本次的前缀/后缀积，再更新回临时变量即可，这样就能够让额外空间复杂度降低到了 O（1），时间复杂度减少一次最后的设置，因为这里结果的计算，合并到了后缀积计算的循环当中去了。
- **结论**：时间，1 ms，100%，空间，49.4 mb，58.58%，时间上，时间上，由于需要遍历 2 次数组，所以时间复杂度为 O（2 * n），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java 
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int n = nums.length;

        // 计算前缀积
        int[] ans = new int[n];
        ans[0] = 1;
        for(int i = 1; i < n; i++) {
            ans[i] = ans[i - 1] * nums[i - 1];
        }

        // 计算后缀积和结果
        int suf = 1;
        for(int i = n - 2; i > -1; i--) {
            // 计算后缀积
            suf *= nums[i + 1];

            // 计算结果 = 前缀积 * 后缀积
            ans[i] *= suf;
        }

        return ans;
    }
}
```

#### 560. 和为 k 的子数组 | medium

##### 1）暴力解法 | O（n^2）

- **思路**：枚举所有子数组，然后统计每个子数组的和，看是否等于 k，从而累加好 count 并返回。
- **结论**：时间，1474ms，14.99%，空间，44.4mb，5.04%，时间复杂度 O（n^2），额外空间复杂度 O（1）。

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        int count = 0, sum;
        for(int i = 0; i < nums.length; i++) {
            sum = 0;
            for(int j = i; j < nums.length; j++) {
                sum += nums[j];
                if(sum == k) {
                    count++;
                }
            }
        }

        return count;
    }
}
```

##### 2）前缀和法 | O（n）

- **思路**：
  1. 建立前缀和数组，pre[i] = pre[i-1] + nums[i]，经过研究发现，pre[i] - pre[j] 刚好就等于 nums[j] +... + nums[i] 连续子数组的元素总和，如果要求 nums[j] +... + nums[i] = k，那么就有 pre[i] - pre[j] = k，即 pre[i] - k = pre[j]，所以问题就转换为了，遍历前缀和数组到 pre[i]，如果能发现 pre[j] = pre[i] - k，那么就说明原数组中元素总和为 k 的存在连续子数组。
  2. 经过尝试发现，并不需要建立 pre 前缀和数组，只需要记录一个 pre 变量，代表上一个前缀和，然后每次累加该变量即可。
  3. 同时，建立前缀和词频表，使得每次计算 pre[j] = pre[i] - k，可以只需要 O（1）的时间内发现 pre[j]，不过要提前录好 pre[j] = 0，以保证任何以 nums[0] 开头的子数组不被算漏~
- **结论**：时间，21ms，92.31%，空间，43.8mb，7.76%，时间上，由于只需要遍历一次 nums 数组，所以时间复杂度为 O（n），空间上，由于使用了一张哈希表，最多存储 n 个元素，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        int pre = 0, count = 0;

        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, 1);

        for(int i = 0; i < nums.length; i++) {
            // 直接使用preSum[1]=1, 而不是preSum[0]=0
            pre += nums[i];

            // 累加: 在初始化之前, 确保get(pre-k)不会到自己
            if(map.containsKey(pre-k)) {
                count += map.get(pre-k);
            }

            // 初始化
            map.put(pre, map.getOrDefault(pre, 0) + 1);
        }

        return count;
    }
}
```

#### 2256. 第 77 双周赛 - 最小平均差 | medium

##### 1）前缀和法 | O（2 * n）

- **思路**：
  1. 先计算好每个 i 的前缀和，并存到 preSums 的数组中。
  2. 然后遍历原始数组 nums，结合前缀和数组，计算每个 i 的平均差，即等于 `( preSums[i] / (i + 1) ) - （ ( preSums[nums.length - 1] - preSums[i] ) / (nums.length - 1 - i) )`，再取绝对值即可。
  3. 最后再返回最小的那个平均差的索引即可，但其中要注意的是：
     - 1）这种方式计算的平均差，对于最后一位来说，由于分母为 0，所以要走特殊处理。
     - 2）当设计大数据量时，这种方式计算会导致 int 的精度溢出，所以改用了 long 类型来存储。
     - 3）这种方式计算的平均差，可能最后会有很多个相同的值，但题目要求的是，只返回最小的索引而已。
- **结论**：时间，7 ms，65.89%，空间，58.9 mb，44.73%，时间上，计算一次前缀和数组要 n 次，计算每个的平均差也要 n 次，所以时间复杂度为 O（2 * n），空间上，由于需要使用一个前缀和数组，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public int minimumAverageDifference(int[] nums) {
        if(nums == null || nums.length == 0) {
            return -1;
        }
        if(nums.length == 1) {
            return 0;
        }

        long[] preSums = new long[nums.length];
        preSums[0] = nums[0];
        for(int i = 1; i < nums.length; i++) {
            preSums[i] = preSums[i - 1] + nums[i];
        }

        int mini = -1;
        long min = Long.MAX_VALUE, tmp;
        for(int i = 0; i < nums.length; i++) {
            if(i == nums.length - 1) {
                tmp = Math.abs(preSums[i] / nums.length);
            } else {
                tmp = Math.abs(
                    ( preSums[i] / (i + 1) ) -
                    ( ( preSums[nums.length - 1] - preSums[i] ) / (nums.length - 1 - i) )
                );
            }
            if(tmp < min) {
                mini = i;
                min = tmp;
            }
        }

        return mini;
    }
}
```

#### 2270. 第 78 双周赛 - 分割数组的方案数 | medium

##### 1）前缀和 + 后缀和 | O（3 * n）

- **思路**：
  1. 计算前缀和数组、后缀和数组，统计前缀和是否大于下一个后缀和，即可知道当前是否为合法的分割方案。
  2. 注意，要用 long 类型数组，防止大数相加导致的 int 值溢出问题。
- **结论**：时间，5 ms，11.19%，空间，57.4 mb，54.14%，时间上，由于需要遍历 3 次 nums 数组，所以时间复杂度为 O（3 * n），空间上，由于使用了一个 preSum 前缀和数组，和一个 postSums 后缀和数组，所以额外空间复杂度为 O（2 * n）。

```java
class Solution {
    public int waysToSplitArray(int[] nums) {
        if(nums == null || nums.length < 2) {
            return 0;
        }

        int len = nums.length;
        long[] preSums = new long[len];
        preSums[0] = nums[0];
        for(int i = 1; i < len; i++) {
            preSums[i] = preSums[i - 1] + nums[i];
        }

        long[] postSums = new long[len];
        postSums[len - 1] = nums[len - 1];
        for(int i = len - 2; i > -1; i--) {
            postSums[i] = postSums[i + 1] + nums[i];
        }

        int count = 0;
        for(int i = 0; i < len - 1; i++) {
            if(preSums[i] >= postSums[i + 1]) {
                count++;
            }
        }

        return count;
    }
}
```

##### 2）前缀和 + 后缀和优化2 | O（2 * n）

- **思路**：在 1 的基础上进行优化，发现前缀和数组可以省略掉，进而改用一个 preSum 变量，进行累加替代。
- **结论**：时间，4 ms，41.66%，空间，57.8 mb，22.12%，时间上，由于省去了一个前缀和数组的设置，所以时间复杂度为 O（2 * n），空间上，同理也是因为少了一个前缀和数组的使用，所以额外空间复杂度为 O（2 * n）。

```java
class Solution {
    public int waysToSplitArray(int[] nums) {
        if(nums == null || nums.length < 2) {
            return 0;
        }

        int len = nums.length;
        long[] postSums = new long[len];
        postSums[len - 1] = nums[len - 1];
        for(int i = len - 2; i > -1; i--) {
            postSums[i] = postSums[i + 1] + nums[i];
        }

        int count = 0;
        long preSum = 0;
        for(int i = 0; i < len - 1; i++) {
            preSum += nums[i];
            if(preSum >= postSums[i + 1]) {
                count++;
            }
        }

        return count;
    }
}
```

##### 3）前缀和 + 后缀和优化3 | O（2 * n）

- **思路**：在 2 的基础上进行优化，发现后缀和数组也可以省略，进行改用 sum - preSum 来得到。
- **结论**：时间，2 ms，100%，空间，55.9 mb，88.74%，时间上，由于虽然省略了后缀和数组的使用，但计算 sum 还是需要先遍历一次数组，所以时间复杂度还是 O（2 * n），空间上，由于只是用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int waysToSplitArray(int[] nums) {
        if(nums == null || nums.length < 2) {
            return 0;
        }

        int count = 0;
        long sum = 0, preSum = 0;
        for(int i = 0; i < nums.length; i++) {
            sum += nums[i];
        }
        for(int i = 0; i < nums.length - 1; i++) {
            preSum += nums[i];

            // preSum >= sum - preSum
            if( (preSum << 1) >= sum) {
                count++;
            }
        }

        return count;
    }
}
```

### 1.9. 数据结构 - 线段树（区间修改树）

#### 模板代码

模板代码，经过了对数器比较过，可放心使用~

```java
/**
 * 线段树 Integer数组版
 *
 * @author yaocs2
 * @since 2022-06-03
 */
public class SegmentTree {

    private int MAX_LEN;// n
    private int[] arr;// 原数组, 从1开始, 0位置弃用
    private int[] sums;// 区间和数组
    private int[] lazys;// 懒更新、懒增加数组
    private boolean[] isUpdates;// 是否更新数组, boolean数组, 防止能够更新的是0
    private int[] changes;// 更新数组

    private int ROOT;
    private int L;
    private int R;

    public SegmentTree(int[] origin) {
        // 数组从1开始, 0位置弃用
        this.MAX_LEN = origin.length + 1;
        this.arr = new int[this.MAX_LEN];
        for (int i = 1; i < this.MAX_LEN; i++) {
            arr[i] = origin[i - 1];
        }

        this.sums = new int[MAX_LEN << 2];// 4n
        this.lazys = new int[MAX_LEN << 2];// 4n
        this.isUpdates = new boolean[MAX_LEN << 2];// 4n
        this.changes = new int[MAX_LEN << 2];// 4n

        // 构造[1, n]范围内、以root为根节点的区间和
        this.ROOT = 1;
        this.L = 1;
        this.R = origin.length;
        build(this.L, this.R, this.ROOT);
    }

    // O(n): 构造[l, r]范围内、以root为根节点的区间和
    private void build(int l, int r, int root) {
        // 叶子节点, 则直接添加到sums数组中
        if(l == r) {
            sums[root] = arr[l];
            return;
        }

        // (mid认为是左孩子)
        int mid = (l + r) / 2;

        // 构造左孩子区间
        build(l, mid, root << 1);// 2 * i

        // 构造右孩子区间
        build(mid + 1, r, root << 1 | 1);// 2 * i + 1

        // 收集左右孩子节点的值, 相加后向上汇报
        pushUp(root);
    }

    // O(1): 收集左右孩子节点的值, 相加后向上汇报
    private void pushUp(int root) {
        // root值 = 2 * i + 2 * i + 1
        sums[root] = sums[root << 1] + sums[root << 1 | 1];
    }
    
    // O(1): 分发root中的懒信息给下面, lcnt表示左子树的节点数量, rcnt表示右子树的节点数量
    private void pushDown(int root, int lcnt, int rcnt) {
        // 先下发懒更新, 保证覆盖之前的孩子节点的懒信息
        int li = root << 1, ri = root << 1 | 1;
        if(isUpdates[root]) {
            isUpdates[li] = true;// 2 * i
            isUpdates[ri] = true;// 2 * i + 1

            sums[li] = changes[root] * lcnt;// 2 * i
            sums[ri] = changes[root] * rcnt;// 2 * i + 1

            changes[li] = changes[root];// 2 * i
            changes[ri] = changes[root];// 2 * i + 1

            lazys[li] = 0;// 2 * i
            lazys[ri] = 0;// 2 * i + 1

            isUpdates[root] = false;
            changes[root] = 0;
        }

        // 再下发懒增加, 保证root上的懒增加能在懒更新后, 也能下发成功
        if(lazys[root] != 0) {
            sums[li] += lazys[root] * lcnt;// 2 * i
            sums[ri] += lazys[root] * rcnt;// 2 * i + 1

            lazys[li] += lazys[root];// 2 * i
            lazys[ri] += lazys[root];// 2 * i + 1

            lazys[root] = 0;
        }
    }
    
    // O(logn): 在[L, R]任务范围上, 累加C值, [l, r]表示表达的范围, root为表达的根节点
    private void add(int L, int R, int C, int l, int r, int root) {
        // 如果任务范围, 完全包括了区间的范围, 那么就懒住
        if(L <= l && R >= r) {
            sums[root] += C * (r - l + 1);
            lazys[root] += C;
            return;
        }

        // 如果任务范围包不住区间范围, 那么就需要向下分发(mid认为是左孩子)
        int mid = (l + r) / 2;

        // 不过下发之前, 先把之前在当前节点的懒信息, 先分发下去
        pushDown(root, mid - l + 1, r - mid);

        // 需要下发给左孩子区间
        if(L <= mid) {
            add(L, R, C, l, mid, root << 1);// 2 * i
        }

        // 需要下发给右孩子区间
        if(R > mid) {
            add(L, R, C, mid + 1, r, root << 1 | 1);// 2 * i + 1
        }

        // 收集左右孩子节点的值, 相加后向上汇报
        pushUp(root);
    }

    // O(logn): 在[L, R]任务范围上, 全部更新为C值, [l, r]表示表达的范围, root为表达的根节点
    private void update(int L, int R, int C, int l, int r, int root) {
        // 如果任务范围, 完全包括了区间的范围, 那么就懒住
        if(L <= l && R >= r) {
            isUpdates[root] = true;
            changes[root] = C;// 懒更新时, 清空之前的所有懒更新内容
            sums[root] = C * (r - l + 1);
            lazys[root] = 0;// 懒更新时, 清空之前的所有懒增加内容
            return;
        }

        // 如果任务范围包不住区间范围, 那么就需要向下分发(mid认为是左孩子)
        int mid = (l + r) / 2;

        // 不过下发之前, 先把之前在当前节点的懒信息, 先分发下去
        pushDown(root, mid - l + 1, r - mid);

        // 需要下发给左孩子区间
        if(L <= mid) {
            update(L, R, C, l, mid, root << 1);// 2 * i
        }

        // 需要下发给右孩子区间
        if(R > mid) {
            update(L, R, C, mid + 1, r, root << 1 | 1);// 2 * i + 1
        }

        // 收集左右孩子节点的值, 相加后向上汇报
        pushUp(root);
    }

    // O(logn): 查询在[L, R]任务范围上的区间和, [l, r]表示表达的范围, root为表达的根节点
    private long querySum(int L, int R, int l, int r, int root) {
        // 如果任务范围, 完全包括了区间的范围, 那么就返回该整段区间的和
        if(L <= l && R >= r) {
            return sums[root];
        }

        // 如果任务范围包不住区间范围, 那么就需要向下分发(mid认为是左孩子)
        int mid = (l + r) / 2;

        // 不过下发之前, 先把之前在当前节点的懒信息, 先分发下去
        pushDown(root, mid - l + 1, r - mid);

        // 需要下发给左孩子区间
        long res = 0;
        if(L <= mid) {
            res += querySum(L, R, l, mid, root << 1);// 2 * i
        }

        // 需要下发给右孩子区间
        if(R > mid) {
            res += querySum(L, R, mid + 1, r, root << 1 | 1);// 2 * i + 1
        }

        return res;
    }

    // 在[L, R]任务范围上, 累加C值
    public void add(int L, int R, int C) {
        // 超出任务范围, 则直接返回
        if(L < this.L || R > this.R) {
            return;
        }

        // 在[L, R]任务范围上, 累加C值, [l, r]表示表达的范围, root为表达的根节点
        add(L, R, C, this.L, this.R, this.ROOT);
    }

    // 在[L, R]任务范围上, 更新为C值
    public void update(int L, int R, int C) {
        // 超出任务范围, 则直接返回
        if(L < this.L || R > this.R) {
            return;
        }

        // 在[L, R]任务范围上, 累加C值, [l, r]表示表达的范围, root为表达的根节点
        update(L, R, C, this.L, this.R, this.ROOT);
    }
    
    // 查询在[L, R]任务范围上的区间和
    public long querySum(int L, int R) {
        // 超出任务范围, 则直接返回
        if(L < this.L || R > this.R) {
            return 0;
        }

        // 查询在[L, R]任务范围上的区间和, [l, r]表示表达的范围, root为表达的根节点
        return querySum(L, R, this.L, this.R, this.ROOT);
    }
}
```

#### 2286. 第 79 双周赛 - 以组为单位订音乐会的门票 | hard

##### 1）线段树 + 二分查找 | O（ g * 3 * logn + s * [ 2 logn + * 2 * q * logn ]）

- **思路**：
  1. 分析题目可知，需要有一个高效的数据结构，去查出某个区间现在有多少观众，这时用线段树就最合适了。
     - 1）`gather()` 操作，是要把 k 个人坐在同一排，且坐的排号越小越好，同时，为了避免找排号，不会导致退化到 O（n * logn），所以需要有个二分的方式去找出满足条件最小排号的方法 `queryIndex()`，以及计算 k 坐下去起始下标的方法 ``querySum()``，即和把 k 人塞到同一排的方法 `add()`。
     - 2）`scatter()` 操作，是要把 k 个人坐下去，但不要求同一排，可以分开坐，且坐的排号越小越号，同时，为了避免找排号，不会导致退化到 O（n * logn），所以需要有个二分的方式去找出满足条件最小排号的方法 `queryIndex()`，以及计算 k 坐下去起始下标的方法 ``querySum()``，即和把 k 人塞到同一排的方法 `add()`。
  2. 这样，建立线段树，就需要建立 `querySum()`、`queryIndex()`、以及 `add()` 方法，其中，`querySum()` 和 `add()` 属于模板代码，默写就好。
  3. 同时，由于需要 `queryIndex()` 进行二分，所以需要记录区间的最小值，以便去判断区间是否符合条件，因此，需要建立并维护 mins 数组，维护方式同 sums 数组，碰到任务能够完全包裹的区间，则懒住，递归完向上汇报信息时维护最小值，在 `querySum()` 和 `queryIndex()` 进入子树前，需要下发之前存到的懒任务，避免发生错误~
  4. 其中，`queryIndex()` 二分的逻辑就是，判断根区间最小值是否满足 C，不满足的就不用二分了，直接返回即可，满足的则先 `pushDown()` ，再二分，由于坐的排数越小越好，所以二分是先判断左子区间是否满足 C，左不满足的再判断能够二分右，如果 mid 超过了最大的 maxRows，那么就返回 0 说明不存在即可，满足的则继续二分右区间，周而复始。直到碰到根节点，返回索引，最小的也就是最靠前的排号即可。
- **结论**：
  1. 时间，137 ms，77.11%，空间，85.7 mb，5.15%。
  2. 时间上，
     - 1）`add()` 花费 O（log n），`querySum()` 花费 O（log n），`queryIndex()` 花费 O（log n），由于 `gatter()` 需要先 `queryIndex()`、再 `querySum()`、再 `add()`，所以时间复杂度为 O（3 * logn）。
     - 而 `scatter()` 需要先 `querySum()`、再 `queryIndex()`、再从找到 idx 开始遍历到 maxRows + 1，设需要遍历 q 次，每次需要先 `querySum()`、再 `add()`，所以时间复杂度为 O（2 logn + q * 2 * logn）。
     - 因此，总的时间复杂度为 O（ g * 3 * logn + s * [ 2 logn + * 2 * q * logn ]），g 为 `gatter()` 方法的调用次数，s 为 `scatter()` 方法的调用次数。
  3. 空间上，由于需要 4 * n 长的 sums、lazys、mins 数组，所以如果不算递归栈空间，那么额外空间复杂度为 O（12 * n），加上递归栈的话，`gatter()` 花费 O（log n），`scatter()` 花费 O（logn），所以总的额外空间复杂度为 O（12 * n + logn）。

```java
class SegmentTree {

    private long[] sums;
    private long[] lazys;
    private long[] mins;
    private int ROOT;
    private int L;
    private int R;

    SegmentTree(int n) {
        this.sums = new long[n << 2];
        this.lazys = new long[n << 2];
        this.mins = new long[n << 2];
        this.ROOT = 1;
        this.L = 1;
        this.R = n;
    }

    // [L, R]范围内的数都加上C
    public void add(int L, int R, int C) {
        if(L < this.L || R > this.R) {
            return;
        }

        // [l, r]为表达的范围, root为表达的根节点
        add(L, R, C, this.L, this.R, this.ROOT);
    }

    // [l, r]为表达的范围, root为表达的根节点
    private void add(int L, int R, int C, int l, int r, int root) {
        // 由于没有了懒增加, 所以需要加到底层
        if(L <= l && R >= r) {
            sums[root] += C * (r - l + 1);
            mins[root] += C * (r - l + 1);
            lazys[root] += C;
            return;
        }

        int mid = (r + l) >> 1;

        // 下发之前, 先把之前在当前节点的懒信息, 先分发下去
        pushDown(root, mid - l + 1, r - mid);

        // 下发任务给左区间
        if(L <= mid) {
            add(L, R, C, l, mid, root << 1);
        }

        // 下发任务给右区间
        if(R > mid) {
            add(L, R, C, mid + 1, r, root << 1 | 1);
        }

        // 向上汇报信息
        pushUp(root);
    }

    private void pushUp(int root) {
        sums[root] = sums[root << 1] + sums[root << 1 | 1];
        mins[root] = Math.min(mins[root << 1], mins[root << 1 | 1]);
    }

    // O(1): 分发root中的懒信息给下面, lcnt表示左子树的节点数量, rcnt表示右子树的节点数量
    private void pushDown(int root, int lcnt, int rcnt) {
        // 下发懒增加, 保证root上的懒增加能在懒更新后, 也能下发成功
        if(lazys[root] != 0) {
            int li = root << 1, ri = root << 1 | 1;

            sums[li] += lazys[root] * lcnt;// 2 * i
            sums[ri] += lazys[root] * rcnt;// 2 * i + 1

            mins[li] += lazys[root] * lcnt;// 2 * i
            mins[ri] += lazys[root] * rcnt;// 2 * i + 1

            lazys[li] += lazys[root];// 2 * i
            lazys[ri] += lazys[root];// 2 * i + 1

            lazys[root] = 0;
        }
    }

    // 查询[L, R]范围内的累加和
    public long querySum(int L, int R) {
        if(L < this.L || R > this.R) {
            return 0;// 非法索引
        }

        // [l, r]为表达的范围, root为表达的根节点
        return querySum(L, R, this.L, this.R, this.ROOT);
    }

    // [l, r]为表达的范围, root为表达的根节点
    private long querySum(int L, int R, int l, int r, int root) {
        // [L, R]任务完全覆盖区间, 则直接返回区间和即可
        if(L <= l && R >= r) {
            return sums[root];
        }

        int mid = (r + l) >> 1;

        // 下发之前, 先把之前在当前节点的懒信息, 先分发下去
        pushDown(root, mid - l + 1, r - mid);

        // 下发任务给左区间
        long ans = 0;
        if(L <= mid) {
            ans += querySum(L, R, l, mid, root << 1);
        }

        // 下发任务给右区间
        if(R > mid) {
            ans += querySum(L, R, mid + 1, r, root << 1 | 1);
        }

        return ans;
    }

    // 返回[L, R]范围内, 小于等于C的最小区间的索引
    public int queryIndex(int L, int R, long C) {
        if(L < this.L || R > this.R) {
            return 0;// 非法索引
        }

        // [l, r]为表达的范围, root为表达的根节点
        return queryIndex(L, R, C, this.L, this.R, this.ROOT);
    }

    // [l, r]为表达的范围, root为表达的根节点
    private int queryIndex(int L, int R, long C, int l, int r, int root) {
        // 当前区间的最小值, 都大于所需要的C, 那么就没必要找下去了, 直接返回0即可
        if(this.mins[root] > C) {
            return 0;// 非法索引
        }
        // 可以找到叶子节点, 则返回
        if(l == r) {
            return l;
        }

        int mid = (r + l) >> 1;

        // 下发之前, 先把之前在当前节点的懒信息, 先分发下去
        pushDown(root, mid - l + 1, r - mid);

        // 由于要小的排优先, 所以先判断左, 不符合才判断右 
        if(this.mins[root << 1] <= C) {
            return queryIndex(L, R, C, l, mid, root << 1);
        } else if(R > mid) {
            return queryIndex(L, R, C, mid + 1, r, root << 1 | 1);
        }

        // 非法索引
        return 0;
    }
}

class BookMyShow {

    private int n;// 排
    private int m;// 列
    private SegmentTree seg;

    public BookMyShow(int n, int m) {
        this.n = n;
        this.m = m;
        this.seg = new SegmentTree(n);
    }
    
    public int[] gather(int k, int r) {
        // 查询[1, r + 1]区间, 是否存在m-k位置的排
        int idx = this.seg.queryIndex(1, r + 1, m - k);
        
        // 非法索引, 说明不存在满足要求的排
        if(idx == 0) {
            return new int[0];
        }

        // 存在符合条件的排, 则先查出最小的座位索引
        int start = (int) this.seg.querySum(idx, idx);
        
        // 开始坐下k个人
        this.seg.add(idx, idx, k);

        // 返回[排号, 开始座位号]
        return new int[] { idx - 1, start };
    }
    
    public boolean scatter(int k, int r) {
        // 查询[1, r + 1]区间, 目前所有的观众数
        long all = this.seg.querySum(1, r + 1);
        
        // r + 1排座位, 不够k个座位, 则返回false
        if((long) (r + 1) * this.m - all < k) {
            return false;
        }

        // 如果足够, 则找到至少还有1个座位的排开始
        for(int i = this.seg.queryIndex(1, r + 1, m - 1); i <= r + 1; i++) {
            long start = this.seg.querySum(i, i);

            // 如果k个人刚好坐的完, 则k个人全部坐下去, 并结束循环
            int leftSeats = (int) ((long) this.m - start);
            if(leftSeats >= k) {
                this.seg.add(i, i, k);
                break;
            } 
            // 否则先坐满这排, 继续下一排
            else {
                this.seg.add(i, i, leftSeats);
                k -= leftSeats;
            }
        }

        return true;
    }
}

/**
 * Your BookMyShow object will be instantiated and called as such:
 * BookMyShow obj = new BookMyShow(n, m);
 * int[] param_1 = obj.gather(k,maxRow);
 * boolean param_2 = obj.scatter(k,maxRow);
 */
```

### 2.0. 数据结构 - 堆

#### 215. 数组中的第 k 个最大元素 | medium

##### 1）暴力解法 | O（n * logn）

- **思路**：从小到大排序，从后面取第 k 个元素即可。
- **结论**：时间，2ms，82.75%，41.4mb，5.04%，效率非常高，但没答到点上，本题主要考的是堆排序和快速排序算法~

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        if(nums == null || nums.length == 0) {
            return -1;
        }

        Arrays.sort(nums);
        return nums[nums.length-k];
    }
}
```

##### 2）堆排序 | O（n * logn）

- **思路**：建立容量为 k 的小根堆，遍历数组加入堆，如果堆大小大于 k，则弹出堆顶，数组遍历完毕后，堆顶即为第 k 个最大的元素。
- **结论**：
  1. 时间，5ms，46.25%，空间，42mb，5.04%，时间上，遍历一次数组+调整堆，所以时间复杂度为 O（n * logn），空间上，容量为 k 的堆额外需要长度为 k 的数组，所以额外空间复杂度为 O（k）。
  2. 该算法最适合用于，初始时数据长度不定的流式数据。

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        if(nums == null || nums.length == 0) {
            return -1;
        }

        // 从小到大排序
        PriorityQueue<Integer> queue = new PriorityQueue<>((o1, o2) -> o1 - o2);
        for(int i = 0; i < nums.length; i++) {
            queue.offer(nums[i]);
            if(queue.size() > k) {
                queue.poll();
            }
        }

        return queue.poll();
    }
}
```

##### 3）快速选择 | <= O（n * logn）

- **思路**：
  1. 对暴力解法进行了改进，由于暴力解法底层以及快速排序，时间复杂度为 O（n * logn），而经过研究，可在快排进行改进，得到比暴力解法更优的解法。
  2. 首先是随机 partition，这是快排 3.0 的基础，旨在使得 partition 目标值的选取变为概率事件，平均期望为 O（n * logn） ，然后每次随机 partition 返回一个数组，代表 partition 后中间位置的左边界和右边界：
     1. 如果第 k 个元素落在中间位置，则返回中间位置的值。
     2. 如果第 k 个元素落在左边区域，则继续从左边区域选择值。
     3. 如果第 k 个元素落在右边区域，则继续从右边区域选择值。
- **结论**：时间，2ms，82.75%，空间，42mb，5.04%，时间上，对比暴力解法的快排后取下标的方式，每次都少了一半的 partition，因此，时间复杂度更低 <= O（n * logn），空间上，额外空间复杂度取决于递归栈的最大深度，为 O（n * logn）。

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        if(nums == null || nums.length == 0) {
            return -1;
        }

        // 从全部区域中选择值, 第k个元素落在全部区域上
        return quickSelect(nums, 0, nums.length-1, nums.length-k);
    }

    private int quickSelect(int nums[], int L, int R, int targetIndex) {
        // 获取partition后中间位置的左边界和右边界
        int[] p = randomPartition(nums, L, R);

        // 如果第k个元素落在中间位置, 则返回中间位置的值
        if(p[0] <= targetIndex && targetIndex <= p[1]) {
            return nums[p[0]];
        } 
        // 如果第k个元素落在左边区域, 则继续从左边区域选择值
        else if(targetIndex < p[0]) {
            return quickSelect(nums, L, p[0]-1, targetIndex);
        } 
        // 如果第k个元素落在右边区域, 则继续从右边区域选择值
        else {
            return quickSelect(nums, p[1]+1, R, targetIndex);
        }
    }

    private int[] randomPartition(int nums[], int L, int R) {
        // 快排3.0: 随机选择一个值交换到末尾做partition
        swap(nums, L + (int) (Math.random() * (R - L + 1)), R);

        // <区域的左边界, >区域的右边界
        int less = L - 1, more = R;

        // L表示当前数的位置, R表示要比较值的位置, more表示右边界的位置
        while(L < more) {
            // 当前数 < 比较值, 则<区域的左边界+1, 并且继续比较下一个值
            if(nums[L] < nums[R]) {
                swap(nums, ++less, L++);
            }
            // 当前数 == 比较值, 则继续比较下一个值
            else if(nums[L] == nums[R]) {
                L++;
            }
            // 当前数 > 比较值, 则右边界-1, 并且还需要比较当前值
            else {
                swap(nums, --more, L);
            }
        }

        swap(nums, more, R);
        return new int[] {less+1, more};
    }

    private void swap(int nums[], int from, int to) {
        int tmp = nums[from];
        nums[from] = nums[to];
        nums[to] = tmp;
    }
}
```

#### 253. 会议室2 | medium

##### 1）小根堆模拟法 | O（2 * n * logn）

- **思路**：
  1. 先按**开始时间**对会议进行顺序排序。
  2. 然后提前构造好一个小根堆, 按**结束时间**对会议进行顺序排序。
  3. 接着模拟平常选择会议室的流程（先从最早的会议开始）：
     1. 如果会议室都没人开会，那么直接进入会议室，此时会议室使用数量 + 1。
     2. 如果最早开会的会议室早就空闲了或者刚刚空闲，那么久先请他们出来，然后进入会议室，此时会议室复用，使用数量不变。
     3. 如果确实没有空闲的会议室，然后自己的会议又要开始了，那么就只能开多一间会议室了，此时会议室使用数量 + 1。
- **结论**：时间，6ms，81.11%，空间，41.2mb，11.78%，时间上，对数组排序需要 O（n * logn），遍历每个会议需要 O（n），并且在里面做堆调整需要 O（logn），所以时间复杂度为 O（2 * n * logn），空间上，由于对数组快速排序需要 O（logn）的递归深度，以及还使用了一个小根堆 O（n），所以额外空间复杂度为 O（logn + n）。

```java
class Solution {
    public int minMeetingRooms(int[][] intervals) {
        if(intervals == null || intervals.length == 0) {
            return 0;
        }
        if(intervals.length == 1) {
            return 1;
        }
        
        // 先按开始时间对会议进行顺序排序
        Arrays.sort(intervals, (o1, o2) -> o1[0] - o2[0]);

        // 构建小根堆, 按结束时间对会议进行顺序排序
        PriorityQueue<int[]> queue = new PriorityQueue<>((o1, o2) -> o1[1] - o2[1]);

        // 先从最早的会议开始
        int count = 0;
        for(int i = 0; i < intervals.length; i++) {
            // 没人开会, 则直接进入会议室, 会议室+1
            if(queue.isEmpty()) {
                queue.offer(intervals[i]);
                count++;
            } 
            // 最早结束的会议室空闲了, 则先请他出来, 再进入会议室
            else if(queue.peek()[1] <= intervals[i][0]) {
                queue.poll();
                queue.offer(intervals[i]);
            }
            // 否则开多一间会议室
            else {
                queue.offer(intervals[i]);
                count++;
            }
        }

        return count;
    }
}
```

#### 面试题 17.14. 最小 k 个数 | medium

##### 1）快速排序法 | O（n * logn）

- **思路**：从小到大排序，然后取最小的 k 个数返回即可。
- **缺点**：在所有内容都可以加载到内存的小数组上可以使用，但对于 100 亿大数据量这种情况就不合适了，此时就需要手写堆，做流式处理。
- **结论**：时间，6ms，69.48，空间，50.7mb，7.16%，时间上，快速排序需要 O（n * logn），空间上，快速排序栈深度最大为 O（logn），所以额外空间复杂度为 O（logn）。

```java
class Solution {
    public int[] smallestK(int[] arr, int k) {
        if(arr == null || arr.length == 0 || k == 0) {
            return new int[0];
        }
        if(arr.length < k) {
            return arr;
        }

        Arrays.sort(arr);
        
        int[] res = new int[k];
        for(int i = 0; i < k; i++) {
            res[i] = arr[i];
        }

        return res;
    }
}
```

##### 2）大根堆法 | O（n * logn）

- **思路**：父节点 = (i - 1) / 2，左孩子 = i * 2 + 1，右孩子 = i * 2 + 2，向上调整直到 0 位置的根节点，向下调整直到没有孩子节点，期间不断比较当前节点的值与父节点、或者最大的孩子节点的值，目的是把大的交换到堆顶，从而保持大根堆的性质。
- **结论**：时间，6ms，69.48，空间，50.5mb，23.00%，时间上，由于每次调整堆需要 O（logn），最多要调整 n 次，所以时间复杂度为 O（n * logn），空间上，由于 heap 是需要返回给题目的数组，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int[] smallestK(int[] arr, int k) {
        if(arr == null || arr.length == 0 || k == 0) {
            return new int[0];
        }
        if(arr.length < k) {
            return arr;
        }

        // 初始化大根堆
        int[] heap = new int[k];
        for(int i = 0; i < k; i++) {
            heap[i] = arr[i];
            siftUp(heap, i);
        }

        // 比较、加入大根堆
        for(int i = k; i < arr.length; i++) {
            if(arr[i] < heap[0]) {
                heap[0] = arr[i];
                siftDown(heap, k, 0);
            }
        }

        return heap;
    }

    // 向上调整大根堆
    private void siftUp(int[] heap, int i) {
        while(i > 0) {
            int k = (i - 1) >>> 1;
            if(heap[i] > heap[k]) {
                swap(heap, i, k);
            }
            i = k;
        }
    }

    // 向下调整大根堆
    private void siftDown(int[] heap, int size, int i) {
        int half = size >>> 1;
        while(i < half) {
            int k = (i << 1) + 1;
            int maxi = k + 1 < size && heap[k + 1] > heap[k]? k + 1 : k;
            if(heap[maxi] > heap[i]) {
                swap(heap, i, maxi);
            }
            i = maxi;
        }
    }

    private void swap(int[] arr, int from, int to) {
        int tmp = arr[to];
        arr[to] = arr[from];
        arr[from] = tmp;
    }
}
```

### 2.1. 数据结构 - 图

#### 模板代码

##### 1）构建有向图

```java
// 点
class Node {
    int value;
    int in = 0;
    int out = 0;
    List<Node> nextList = new ArrayList<>();
    List<Edge> edgeList = new ArrayList<>();

    Node(int value) {
        this.value = value;
    }
}

// 边
class Edge {
    int weight;
    Node fromNode;
    Node toNode;

    Edge(int weight, Node fromNode, Node toNode) {
        this.weight = weight;
        this.fromNode = fromNode;
        this.toNode = toNode;
    }
}

// 图
class Graph {
    Map<Integer, Node> nodeMap = new HashMap<>();// 点集
    Set<Edge> edgeSet = new HashSet<>();// 边集
}

// 构建有向图
public Graph createGraph(int[][] matrix) {
    Graph graph = new Graph();
    for(int i = 0; i < matrix.length; i++) {
        Integer fromValue = matrix[i][0];
        Integer toValue = matrix[i][1];
        Integer edgeWeight = matrix[i][2];

        if(!graph.nodeMap.containsKey(fromValue)) {
            graph.nodeMap.put(fromValue, new Node(fromValue));
        }
        if(!graph.nodeMap.containsKey(toValue)) {
            graph.nodeMap.put(toValue, new Node(toValue));
        }

        Node fromNode = graph.nodeMap.get(fromValue);
        Node toNode = graph.nodeMap.get(toValue);
        Edge newEdge = new Edge(edgeWeight, fromNode, toNode);
        fromNode.nextList.add(toNode);
        fromNode.out++;
        toNode.in++;
        fromNode.edgeList.add(newEdge);
        graph.edgeSet.add(newEdge);
    }
    return graph;
}
```

##### 2）宽度优先遍历

```java
// 宽度优先遍历
private void bfs(Node node) {
    if(node == null) {
        return;
    }

    Queue<Node> queue = new LinkedList<>();
    Set<Node> set = new HashSet<>();
    queue.add(node);
    set.add(node);

    while(!queue.isEmpty()) {
        node = queue.poll();
        // 执行node的宽度优先遍历的业务处理
        for(Node next : node.nextList) {
            if(!set.contains(node)) {
                queue.add(next);
                set.add(next);
            }
        }
    }
}
```

##### 3）深度优先遍历

```java
// 深度优先遍历
private void dfs(Node node) {
    if(node == null) {
        return;
    }

    LinkedList<Node> stack = new LinkedList<>();
    Set<Node> set = new HashSet<>();
    stack.push(node);
    set.add(node);
    // 执行node的深度优先遍历的业务处理

    while(!stack.isEmpty()) {
        node = stack.pop();
        for(Node next : node.nextList) {
            if(!set.contains(next)) {
                stack.push(node);
                stack.push(next);
                set.add(next);
                // 执行next的深度优先遍历的业务处理
                break;
            }
        }
    }
}
```

##### 4）拓扑排序

```java
// 拓扑排序
private List<Node> sortedTopology(Graph graph) {
    Map<Node, Integer> inMap = new HashMap<>();
    Queue<Node> zeroInQueue = new LinkedList<>();
    for(Node node : graph.nodeMap.values()) {
        inMap.put(node, node.in);
        if(node.in == 0) {
            zeroInQueue.add(node);
        }
    }

    Node cur;
    List<Node> res = new LinkedList<>();
    while(!zeroInQueue.isEmpty()) {
        cur = zeroInQueue.poll();
        res.add(cur);
        for(Node next : cur.nextList) {
            inMap.put(next, inMap.get(next)-1);
            if(inMap.get(next) == 0) {
                zeroInQueue.add(next);
            }
        }
    }

    return res;
}

```

##### 5）最小生成树 | Kruskal 算法

```java
// k算法(克鲁斯卡尔算法) -> 从边的角度, 去求无向图的最小生成树
private Set<Edge> kruskalMST(Graph graph) {
    UnionFindSet<Node> unionFindSet = new UnionFindSet<>(new HashSet<>(graph.nodeMap.values()));

    // 所有边按权重从小到大排序
    PriorityQueue<Edge> queue = new PriorityQueue<>((o1, o2) -> o1.weight - o2.weight);
    for(Edge edge : graph.edgeSet) {
        queue.offer(edge);
    }

    Edge edge;
    Set<Edge> res = new HashSet<>();
    while(!queue.isEmpty()) {
        edge = queue.poll();
        if(!unionFindSet.isSameSet(edge.fromNode, edge.toNode)) {
            res.add(edge);
            unionFindSet.union(edge.fromNode, edge.toNode);
        }
    }

    return res;
}

// 并查集元素 -> 无向图求最小生成树
class Element<V> {
    private V value;

    Element(V value) {
        this.value = value;
    }
}

// 并查集 -> 无向图求最小生成树
class UnionFindSet<V> {
    private Map<V, Element<V>> elementMap = new HashMap<>();
    private Map<Element<V>, Element<V>> fatherMap = new HashMap<>();
    private Map<Element<V>, Integer> sizeMap = new HashMap<>();

    UnionFindSet(Set<V> set) {
        for(V v : set) {
            Element<V> e = new Element<>(v);
            elementMap.put(v, e);
            fatherMap.put(e, e);
            sizeMap.put(e, 1);
        }
    }

    private Element<V> findHeader(V value) {
        LinkedList<Element<V>> stack = new LinkedList<>();

        Element e = elementMap.get(value), ef;
        while(e != (ef = fatherMap.get(e))) {
            stack.push(e);
            e = ef;
        }

        // 路径压缩
        while(!stack.isEmpty()) {
            fatherMap.put(stack.pop(), e);
        }

        return e;
    }

    private boolean isSameSet(V a, V b) {
        if(!elementMap.containsKey(a) || !elementMap.containsKey(b)) {
            return false;
        }
        return findHeader(a) == findHeader(b);
    }

    void union(V a, V b) {
        if(!elementMap.containsKey(a) || !elementMap.containsKey(b)) {
            return;
        }
        if(isSameSet(a, b)) {
            return;
        }

        Element<V> af = findHeader(a);
        Element<V> bf = findHeader(b);

        if(af != bf) {
            Element<V> big = sizeMap.get(af) > sizeMap.get(bf)? af : bf;
            Element<V> small = big == af? bf : af;
            fatherMap.put(small, big);

            int newSize = sizeMap.get(small) + sizeMap.get(big);
            sizeMap.put(big, newSize);
            sizeMap.remove(small);
        }
    }
}
```

##### 6）最小生成树 | Prim 算法

```java
// p算法(普里姆算法) -> 从点的角度, 去求无向图的最小生成树
private Set<Edge> primMST(Graph graph) {
    Set<Edge> res = new HashSet<>();
    HashSet<Node> nodeSet = new HashSet<>();
    PriorityQueue<Edge> queue = new PriorityQueue<>((o1, o2) -> o1.weight - o2.weight);

    // 遍历每个点, 处理森林的问题, 队列没边时会从这里拿点, 去开启另外一个点团的判断 -> 这正是不需要像k算法那样用并查集去合并的原因
    for (Node node : graph.nodeMap.values()) {
        if(nodeSet.contains(node)) {
            continue;
        }

        // 解锁新点中的边, 所有边按权重从小到大排序
        nodeSet.add(node);
        for (Edge edge : node.edgeList) {
            queue.offer(edge);
        }

        // 遍历解锁出来的边, 判断是否能否有新的边又被解锁出来
        while(!queue.isEmpty()){
            Edge edge = queue.poll();

            // 解锁新点中的边, 所有边按权重从小到大排序
            if(!nodeSet.contains(edge.toNode)) {
                nodeSet.add(edge.toNode);
                res.add(edge);
                for (Edge nextEdge : edge.toNode.edgeList) {
                    queue.offer(nextEdge);
                }
            }
        }
    }

    return res;
}
```

##### 7）点之间最短路径 | Dijkstra 算法

```java
// Dijkstra算法(迪克斯特拉算法) -> 求 head 到每个点之间的最短路径, 其中图内可以有权重为负数的边, 但要求无累加和为负数的环, 但这里实现的算法规定不能有权重为负数的边!
private Map<Node, Integer> dijkstraShortestPath(Node head) {
    // head 到每个点之间的最短路径, 0表示最短, 不存在的则代表正无穷
    Map<Node, Integer> distanceMap = new HashMap<>();
    distanceMap.put(head, 0);

    // 已经得出了最短路径的点, 锁好无需再重复判断
    Set<Node> selectedNodeSet = new HashSet<>();
    Node minNode = getMinDistanceAndUnselectedNode(distanceMap, selectedNodeSet);
    while (minNode != null) {
        int distance = distanceMap.get(minNode);
        for (Edge edge : minNode.edgeList) {
            Node toNode = edge.toNode;

            // 不存在则认为路径无限大, 则更新它的最短路径为当前权重
            if(!distanceMap.containsKey(toNode)) {
                distanceMap.put(toNode, distance + edge.weight);
            }
            // 否则更新它的最短路径: 从to以前的路径与当前点到to的路径做比较, 去最小值作为最小路径
            else {
                distanceMap.put(toNode, Math.min(distanceMap.get(toNode), distance + edge.weight));
            }
        }

        // 判断完最小路径后, 则加入锁定集合中, 无需再重复判断
        selectedNodeSet.add(minNode);
        minNode = getMinDistanceAndUnselectedNode(distanceMap, selectedNodeSet);
    }

    return distanceMap;
}

// 根据最短路径Map+锁定的点集合, 筛选出下一个用于继续判断最短路径的点 => 经典实现版本O(n), 可自改小根堆优化成O(logn)
private Node getMinDistanceAndUnselectedNode(Map<Node, Integer> distanceMap, Set<Node> selectedNodeSet) {
    Node minNode = null;
    int minDistance = Integer.MAX_VALUE;

    for (Map.Entry<Node, Integer> entry : distanceMap.entrySet()) {
        Node node = entry.getKey();
        int distance = entry.getValue();
        if (!selectedNodeSet.contains(node) && distance < minDistance) {
            minNode = node;
            minDistance = distance;
        }
    }

    return minNode;
}
```

#### 207. 课程表 | medium

##### 1）拓扑排序 | O（3m+2n）

- **思路**：
  1. 分析题目可知，对于 [a, b] 表示要学习 a 必须先学习 b，那么可以建立有向图 a <- b，此时 a 的入度为 1（[5，5] 表示要想学习 5 必须先学习 5，这是不可能完成的事件），因此，要想判断最终的结果是否能够全部学完 n 门课程，则可以通过拓扑排序检验入度的方式，把能够学完的课程加入集合，然后判断集合大小与 n 做比较即可。
  2. 其中要注意的是，给出的 [a, b] 数组可能不全，也就是并没有 n 个这个多，即可能会有孤立节点没给出，此时需要在构建图是补充回去~
- **结论**：
  1. 时间，13ms，14.44%，空间，42.2mb，5.00%，时间上，在构建图时，遍历课程先修数组花费了 O（m），遍历给出的 n 门课程花费了 O（n），在拓扑排序时需要花费 O（n + 2m），因此总的时间复杂度为 O（3m+2n），空间上，由于使用的是高级数据结构，所以是非常高的~
  2. 虽然效率不是很理想，但胜在有统一的模板代码，可以把不同的图题目转换统一的代码格式，大大提高了通过率~

```java
class Solution {

    // 点
    class Node {
        int value;
        int in = 0;
        int out = 0;
        List<Node> nextList = new ArrayList<>();
        List<Edge> edgeList = new ArrayList<>();

        Node(int value) {
            this.value = value;
        }
    }

    // 边
    class Edge {
        int weight;
        Node fromNode;
        Node toNode;

        Edge(int weight, Node fromNode, Node toNode) {
            this.weight = weight;
            this.fromNode = fromNode;
            this.toNode = toNode;
        }
    }

    // 图
    class Graph {
        Map<Integer, Node> nodeMap = new HashMap<>();// 点集
        Set<Edge> edgeSet = new HashSet<>();// 边集
    }

    // 构建有向图
    public Graph createGraph(int numCourses, int[][] prerequisites) {
        Graph graph = new Graph();

        for(int i = 0; i < prerequisites.length; i++) {
            Integer toValue = prerequisites[i][0];
            Integer fromValue = prerequisites[i][1];

            if(!graph.nodeMap.containsKey(fromValue)) {
                graph.nodeMap.put(fromValue, new Node(fromValue));
            }
            if(!graph.nodeMap.containsKey(toValue)) {
                graph.nodeMap.put(toValue, new Node(toValue));
            }

            Node fromNode = graph.nodeMap.get(fromValue);
            Node toNode = graph.nodeMap.get(toValue);
            Edge newEdge = new Edge(0, fromNode, toNode);
            fromNode.nextList.add(toNode);
            fromNode.out++;
            toNode.in++;
            fromNode.edgeList.add(newEdge);
            graph.edgeSet.add(newEdge);
        }

        // 补充孤立节点
        for(int i = 0; i < numCourses; i++) {
            if(!graph.nodeMap.containsKey(i)) {
                graph.nodeMap.put(i, new Node(i));
            }
        }

        return graph;
    }

    public boolean canFinish(int numCourses, int[][] prerequisites) {
        if(prerequisites == null || prerequisites.length == 0 || prerequisites[0].length == 0) {
            return true;
        }

        Graph graph = createGraph(numCourses, prerequisites);
        return sortedTopology(graph).size() == numCourses;
    }

     // 拓扑排序
    private List<Node> sortedTopology(Graph graph) {
        Map<Node, Integer> inMap = new HashMap<>();
        Queue<Node> zeroInQueue = new LinkedList<>();
        for(Node node : graph.nodeMap.values()) {
            inMap.put(node, node.in);
            if(node.in == 0) {
                zeroInQueue.add(node);
            }
        }

        Node cur;
        List<Node> res = new LinkedList<>();
        while(!zeroInQueue.isEmpty()) {
            cur = zeroInQueue.poll();
            res.add(cur);
            for(Node next : cur.nextList) {
                inMap.put(next, inMap.get(next)-1);
                if(inMap.get(next) == 0) {
                    zeroInQueue.add(next);
                }
            }
        }

        return res;
    }
}
```

#### 399. 除法求值 | medium

##### 1）深度优先搜索 | O（m + [n * 2m]）

- **思路**：根据题意，给定条件 a / b、b / c，要求 b / a 和 a / b、c / a 和 a / c，所以需要构建有向图，且是双向图，然后执行深度优先遍历，如果目标节点存在，则返回遍历路径累计乘积，否则返回 -1。
- **结论**：时间，1ms，55.20%，空间，39.7mb，7.51%，时间上，记条件列表长度 = 值列表长度 = m，问题列表长度 = n，构建图花费了 O（m），遍历问题列表需要花费 O（n），深度优先遍历需要遍历所有节点 O（2 * m），所以时间复杂度为 O（m + [n * 2m]），空间上，由于使用的是高级数据结构，所以额外空间复杂度为非常高~

```java
class Solution {
    
    // 点
    class Node {
        String value;
        int in;
        int out;
        List<Node> nextList = new ArrayList<>();
        List<Edge> toEdgeList = new ArrayList<>();

        Node(String value) {
            this.value = value;
        }
    }

    // 边
    class Edge {
        double weight;
        Node fromNode;
        Node toNode;

        Edge(double weight, Node fromNode, Node toNode) {
            this.weight = weight;
            this.fromNode = fromNode;
            this.toNode = toNode;
        }
    }

    // 图
    class Graph {
        Map<String, Node> nodeMap = new HashMap<>();// 点集
        Set<Edge> edgeSet = new HashSet<>();// 边集
    }

    // 构造双向图: 无向图的变种, a / b 初始化为 a -> b, a <- b
    private Graph createGraph(List<List<String>> equations, double[] values) {
        Graph graph = new Graph();

        // a -> b, b -> a
        Map<String, String> equation1Map = new HashMap<>();
        Map<String, String> equation2Map = new HashMap<>();

        for(int i = 0; i < equations.size(); i++) {
            List<String> equation = equations.get(i);
        
            String value1 = equation.get(0);
            String value2 = equation.get(1);
            double edgeWeight1 = values[i];
            double edgeWeight2 = 1 / values[i];

            // a -> b 相同, 则只取第一个
            if(value2.equals(equation1Map.get(value1))) {
                continue;
            }
            // b -> a 相同, 则只取第一个
            if(value1.equals(equation2Map.get(value2))) {
                continue;
            }           
            equation1Map.put(value1, value2);
            equation2Map.put(value2, value1);

            if(!graph.nodeMap.containsKey(value1)) {
                graph.nodeMap.put(value1, new Node(value1));
            }
            if(!graph.nodeMap.containsKey(value2)) {
                graph.nodeMap.put(value2, new Node(value2));
            }

            Node node1 = graph.nodeMap.get(value1);
            Node node2 = graph.nodeMap.get(value2);

            // a / b 初始化为 a -> b
            Edge edge1 = new Edge(edgeWeight1, node1, node2);
            node1.out++;
            node2.in++;
            node1.nextList.add(node2);
            node1.toEdgeList.add(edge1);
            graph.edgeSet.add(edge1);

            // a / b 初始化为 b -> a
            Edge edge2 = new Edge(edgeWeight2, node2, node1);
            node2.out++;
            node1.in++;
            node2.nextList.add(node1);
            node2.toEdgeList.add(edge2);
            graph.edgeSet.add(edge2);
        }

        return graph;
    }

    public double[] calcEquation(List<List<String>> equations, double[] values, List<List<String>> queries) {
        // 构造双向图: 无向图的变种, a / b 初始化为 a -> b, a <- b
        Graph graph = createGraph(equations, values);

        // 创建返回结果
        double[] res = new double[queries.size()];

        // 遍历问题列表
        for(int i = 0; i < queries.size(); i++) {
            List<String> query = queries.get(i);
            String fromQuery = query.get(0);
            String toQuery = query.get(1);

            // 没有这个条件, 则返回-1
            if(!graph.nodeMap.containsKey(fromQuery) || !graph.nodeMap.containsKey(toQuery)) {
                res[i] = -1;
                continue;
            }

            Node fromNode = graph.nodeMap.get(fromQuery);
            Node toNode = graph.nodeMap.get(toQuery);

            // 条件都存在, 如果比较的是相同节点, 那么直接返回1
            if(fromNode == toNode){
                res[i] = 1;
                continue;
            }

            // 如果比较的是不同节点, 那么遍历图获取对应的结果
            res[i] = dfs(fromNode, toNode);
        }

        return res;
    }

    // 深度优先搜索
    private double dfs(Node fromNode, Node toNode) {
        LinkedList<Node> stack = new LinkedList<>();
        Set<Node> set = new HashSet<>();

        stack.push(fromNode);
        set.add(fromNode);

        boolean hasToNode = false;
        LinkedList<Edge> edgeStack = new LinkedList<>();
        while(!stack.isEmpty()) {
            fromNode = stack.pop();
            for(Edge toEdge : fromNode.toEdgeList) {
                if(!set.contains(toEdge.toNode)) {
                    stack.push(fromNode);
                    stack.push(toEdge.toNode);
                    set.add(toEdge.toNode);
                    edgeStack.push(toEdge);// 边进栈
                    if(toEdge.toNode == toNode) {
                        hasToNode = true;
                    }
                    break;
                }
            }
            // 弹出没用的边
            if(!edgeStack.isEmpty() && edgeStack.peek().toNode == fromNode) {
                edgeStack.pop();
            }
            if(hasToNode) {
                break;
            }
        }

        // 累计路径乘积
        double res = 1;
        while(!edgeStack.isEmpty()) {
            res *= edgeStack.pop().weight;
        }

        return hasToNode? res : -1;
    }
}
```

#### 2285. 第 79 双周赛 - 道路的最大总重要性 | medium

##### 1）图 + 贪心 + 排序 | O（2 * n + n * logn + m）

- **思路**：
  1. 先根据图模板，构建一张图。
  2. 然后，遍历图中的所有点，并且按照点的入度，从大到小排序，优先给大值的点设置权值。
  3. 接着，遍历图中的所有边，并计算边上的权重，并且累加所有边的权重和，作为返回结果即可。
- **结论**：时间，147 ms，5.19%，空间，79.9 mb，5.10%，时间上，构建图花费 O（n），根据入度逆序排序点，花费 O（n * logn），遍历点设置权值，花费 O（n），遍历所有边花费 O（m），所以时间复杂度为 O（2 * n + n * logn + m），空间上，点集花费 O（n），边集花费 O（m），对点集进行快速排序，花费 O（logn），所以额外空间复杂度为 O（n + m + logn）。

```java
class Solution {

    class Graph {
        Map<Integer, Node> nodeMap = new HashMap<>();
        LinkedList<Node> nodeList = new LinkedList<>();
        Map<Node, List<Edge>> nodeEdgeMap = new HashMap<>();
        Set<Edge> edgeSet = new HashSet<>();
    }
    
    class Node {
        int no;
        int in;
        int out;
        int value;
        
        Node(int no) {
            this.no = no;
        }
    }
    
    class Edge {
        Node from;
        Node to;
        int weight;
        
        Edge(Node from, Node to) {
            this.from = from;
            this.to = to;
        }
    }

    private Graph createGraph(int n, int[][] roads) {
        Graph graph = new Graph();
        
        int x, y;
        for(int i = 0; i < roads.length; i++) {
            x = roads[i][0];
            y = roads[i][1];
            
            Node xnode = graph.nodeMap.get(x);
            if(xnode == null) {
                xnode = new Node(x);
                graph.nodeMap.put(x, xnode);
                graph.nodeList.add(xnode);
            }
            
            Node ynode = graph.nodeMap.get(y);
            if(ynode == null) {
                ynode = new Node(y);
                graph.nodeMap.put(y, ynode);
                graph.nodeList.add(ynode);
            }
            
            xnode.in++;
            xnode.out++;
            ynode.in++;
            ynode.out++;
            
            Edge edge = new Edge(xnode, ynode);
            graph.edgeSet.add(edge);
            
            if(!graph.nodeEdgeMap.containsKey(xnode)) {
                graph.nodeEdgeMap.put(xnode, new ArrayList<>());
            }
            graph.nodeEdgeMap.get(xnode).add(edge);
            
            if(!graph.nodeEdgeMap.containsKey(ynode)) {
                graph.nodeEdgeMap.put(ynode, new ArrayList<>());
            }
            graph.nodeEdgeMap.get(ynode).add(edge);
        }
        
        return graph;
    }
    
    public long maximumImportance(int n, int[][] roads) {
        if(roads == null || roads.length == 0 || roads[0] == null || roads[0].length == 0) {
            return 0;
        }
        
        Graph graph = createGraph(n, roads);
        graph.nodeList.sort((o1, o2) -> o2.in - o1.in);
        while(!graph.nodeList.isEmpty()) {
            Node node = graph.nodeList.poll();
            node.value = n--;
        }
        
        long res = 0;
        for(Edge edge : graph.edgeSet) {
            edge.weight = edge.from.value + edge.to.value;
            res += edge.weight;
        }
        
        return res;
    }
}
```

##### 2）贪心 + 排序 |  O（m + 2 * n）

- **思路**：

  1. 分析  1 中的图算法发现，其实就是先对点的入度进行排序，然后编号，再统计累加，即：

     ```java
     // 点：个数 编号 结果 = 个数 * 编号
     // 2： 4   5   20
     // 1： 3   4   12
     // 3： 3   3   9
     // 0： 2   2   4
     // 4： 1   1   1
     ```

  2. 所以，采用一张哈希表存储统计结果，然后进行顺序排序，再顺序编号，同时进行累加结果，最后返回。

- **结论**：

  1. 时间，6 ms，78.42%，空间，63.2 mb，37.87%。
  2. 时间上，统计花费 O（m），排序花费 O（n），编号与累加花费 O（n），所以时间复杂度为 O（m + 2 * n）。其实，如果能在值更新时，进行一次堆排序，性能也是需要花费 O（n * logn），所以重新手写堆没意义。
  3. 空间上，由于使用了一张哈希表花费 O（n），快速排序花费 O（logn），所以额外空间复杂度为 O（n + logn）。

```java
class Solution {
    public long maximumImportance(int n, int[][] roads) {
        if(roads == null || roads.length == 0 || roads[0] == null || roads[0].length == 0) {
            return 0;
        }

        long[] counts = new long[n];
        for(int i = 0; i < roads.length; i++) {
            counts[roads[i][0]]++;
            counts[roads[i][1]]++;   
        }

        // 顺序排序
        Arrays.sort(counts);

        // 顺序编号并统计
        long res = 0;
        for(int i = 1; i <= n; i++) {
            res += i * counts[i - 1];
        }

        return res;
    }
}
```

### 2.2. 算法思想 - 模拟 | 数学

#### 6. Z 字变换 | medium

##### 1）模拟法 | O（n）

- **思路**：模拟题目的要求，从上到下，从左到右进行追加字符，其中使用 StringBuilder 数组进行追加，对比二维数组可以节省空间，避免浪费。
- **结论**：时间，4 ms，82.51，空间，41.8 mb，60.80%，时间上，由于只需要遍历一次 chars 数组，所以时间复杂度为 O（n），空间上，由于 StringBuilder 数组最多存储 n 个字符，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public String convert(String s, int numRows) {
        if(s == null || s.length() == 0 || numRows <= 1) {
            return s;
        }

        char[] chars = s.toCharArray();
        int row = 1, idx = 0, isDown = 1;
        StringBuilder[] sbs = new StringBuilder[numRows + 1];
        while(idx < chars.length){
            if(sbs[row] == null) {
                sbs[row] = new StringBuilder();
            }

            // 追加到i行
            sbs[row].append(chars[idx++]);
            
            // 正在向下
            if(isDown == 1) {
                if(row != numRows) {
                    row++;
                } else {
                    isDown = 0;
                    row--;
                }
            } 
            // 正在向上
            else {
                if(row != 1) {
                    row--;
                } else {
                    isDown = 1;
                    row++;
                }
            }
        }

        StringBuilder res = new StringBuilder();
        for(int i = 1; i < sbs.length; i++) {
            if(sbs[i] != null) {
                res.append(sbs[i].toString());
            }
        }

        return res.toString();
    }
}
```

##### 2）下标构造法 | O（n）

- **思路**：按照 'Z' 字形，列出每个元素的索引，然后与各种变量 `n、acol、t、tcount、col` 进行分析，最终发现有以下关系：
  1. **第一行**：只有一个元素，Z 索引转换为原索引 = row + round * t。
  2. **中间行**：中间行最多有两个元素，第一个元素原索引 = row + round * t，第二个元素的原索引 = (round + 1) * t - row。
  3. **最后一行**：只有一个元素，Z 索引转换为原索引 = row + round * t。
- **结论**：时间，2 ms，98.63%，空间，41.4 mb，87.80%，时间上，由于遍历 chars 数组，所以时间复杂度为 O（n），空间上，由于只是用有限几个额外变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public String convert(String s, int numRows) {
        if(s == null || s.length() == 0 || numRows <= 1 || numRows >= s.length()) {
            return s;
        }

        char[] chars = s.toCharArray();
        StringBuilder res = new StringBuilder();

        /**
         * 0             0+t                    0+2t                     0+3t
         * 1      t-1    1+t            0+2t-1  1+2t            0+3t-1   1+3t
         * 2  t-2        2+t  0+2t-2            2+2t  0+3t-2             2+3t  
         * 3             3+t                    3+2t                     3+3t
         */
        // 总字符数、每个周期的列数、每个周期的字符数、周期数, 铺满所有周期的总列数
        int n = chars.length, acol = numRows - 1, t = 2 * numRows - 2, tcount = n / t + 1, col = n / t * acol + 1, idx;
        for(int row = 0; row < numRows; row++) {
            for(int round = 0; round < tcount; round++) {
                idx = row + round * t;
                if(idx < n) {
                    res.append(chars[idx]);
                }

                // 非第一行、非最后一行, 存 在周期的第二个字符
                if(row != 0 && row != numRows - 1) {
                    idx = (round + 1) * t - row;
                    if(idx < n) {
                        res.append(chars[idx]);
                    }
                }
            }
        }

        return res.toString();
    }
}
```

#### 7. 整数反转 | medium

##### 1）暴力模拟法 | O（n）

- **思路**：模拟反转流程，把整数转换为字符串，然后反转非符号位，最后判断是否溢出、转换为整数、添加符号位返回即可。
- **结论**：时间，1 ms，42.49%，空间，38.8 mb，46.78%，时间上，由于需要遍历一次 chars 数组，所以时间复杂度为 O（n），n 为整数 x 的十进制位数，空间上，由于需要一个额外的字符串数组，额外空间复杂度为 O（n）。

```java
class Solution {
    public int reverse(int x) {
        String s = String.valueOf(x);
        char[] chars = s.toCharArray();
        if(chars.length <= 1) {
            return x;
        }

        StringBuilder sb = new StringBuilder();
        for(int i = chars.length - 1; i > (x < 0? 0 : -1); i--) {
            sb.append(chars[i]);
        }
    
        try {
            return Integer.valueOf(sb.toString()) * (x < 0? -1 : 1);
        } catch (Exception e) {
            return 0;
        } 
    }
}
```

##### 2）精确判断法 | O（n）

- **思路**：
  1. 模拟反转流程，把整数转换为字符串，然后反转非符号位，最后判断是否溢出、转换为整数、添加符号位返回即可。
  2. 这里是通过累加十进制的方式，得到的反转后的整数值，以及通过与 min 和 max 判断，是否会发生溢出，而不是利用异常~
- **结论**：时间，1 ms，42.49%，空间，38.8 mb，48.48%，时间上，由于需要遍历一次 chars 数组，所以时间复杂度为 O（n），n 为整数 x 的十进制位数，空间上，由于需要一个额外的字符串数组，额外空间复杂度为 O（n）。

```java
class Solution {
    public int reverse(int x) {
        String s = String.valueOf(x);
        char[] chars = s.toCharArray();
        if(chars.length <= 1) {
            return x;
        }

        int min = -1 * (1 << 31), max = (1 << 31) - 1, digit = 0, res = 0, tmp;
        for(int i = (x < 0? 1 : 0); i < chars.length ; i++) {
            tmp = chars[i] - '0';
            if(res + Math.pow(10, digit) * tmp > max) {
                return 0;
            }

            res += Math.pow(10, digit) * tmp;
            digit++;
        }
    
        if(x < 0) {
            return -1 * res < min? 0 : -1 * res; 
        } else {
            return res;
        }
    }
}
```

##### 3）数学法 | O（n）

- **思路**：
  1. 根据数学公式，`x % 10` 可取出最低十进制位的数字，`x / 10 % 10` 可取出次低十进制位的数字，`res * 10 + adigit` 可根据数字，模拟得出十进制的数字。
  2. 根据 `Integer.MIN_VALUE <= 最终结果 <= Integer.MAX_VALUE` 可推出，res * 10 之前，不能小于 `Integer.MIN_VALUE / 10`，同时，不能大于 `Integer.MAX_VALUE / 10`，由于这题是反转，追加的是高位，所以 adigit 那部分不用校验，是不会引起溢出的，根本没这么小。
- **结论**：时间，0 ms，100%，空间，38.4 mb，92.82%，时间上，由于需要遍历 n 位十进制位，所以时间复杂度位 O（n），空间上，由于只是用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int reverse(int x) {
        int res = 0;
        while(x != 0) {
            if(res < Integer.MIN_VALUE / 10 || res > Integer.MAX_VALUE / 10) {
                return 0;
            }

            int adigit = x % 10;
            x /= 10;
            res = res * 10 + adigit;
        }
        return res;
    }
}
```

#### 8. 字符串转换整数（atoi） | medium

##### 1）模拟法 | O（n）

- **思路**：
  1. 遍历字符数组，分情况判断：
     - 1）空格符 ' ' ASCII 码 32：不存在数字和符号位时，说明为前导空格，跳过，否则说明为后置空格，退出。
     - 2）'-' 负号 45，'+' 正号 43：不存在数字和符号位时，说明为前导符号，第一个符号可以设置为符号位，以及符号标志位，其余的则认为是后置符号，退出。
     - 3）数字 [48，57]：设置数字标志位，根据 res * 10 + adigit <= Integer.MAX_VALUE 得到，res 要小于等于 ( Integer.MAX_VALUE - adigit ) / 10，因为这道题是追加个位，所以 adigit 会引发溢出，所以需要校验
     - 4）其他字符：退出。
  2. 最后返回 res 结果即可。
- **结论**：时间，1 ms，100%，空间，41.1 mb，89.20%，时间上，由于需要遍历 n 长的 chars 数组，所以时间复杂度为 O（n），由于最多只需要有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int myAtoi(String s) {
        if(s == null || s.length() == 0) {
            return 0;
        }
        
        char[] chars = s.toCharArray();

        int hasNum = 0, hasFlag = 0, isNegative = 0, res = 0, adigit;
        Queue<Integer> queue = new LinkedList<>();
        for(int i = 0; i < chars.length; i++) {
            // 空格
            if(chars[i] == ' ') {
                // 前导空格
                if(hasNum == 0 && hasFlag == 0) {
                    continue;
                }
                // 后置空格
                else {
                    break;
                }
            }
            // 符号位
            else if(chars[i] == '-' || chars[i] == '+') {
                // 前导符号
                if(hasNum == 0 && hasFlag == 0) {
                    hasFlag = 1;
                    isNegative = chars[i] == '-'? 1 : 0;
                }
                // 后置符号
                else {
                    break;
                }
            }
            // 数字
            else if(chars[i] >= '0' && chars[i] <= '9') {// [48, 57]
                hasNum = 1;
                adigit = chars[i] - '0';
                if(res > ( Integer.MAX_VALUE - adigit ) / 10) {
                    return isNegative == 1? Integer.MIN_VALUE : Integer.MAX_VALUE;
                }
                res = res * 10 + adigit;
            } 
            // 碰到其他字符, 则终止遍历
            else {
                break;
            }
        }

        return isNegative == 1? -1 * res : res;
    }
}
```

##### 2）状态自动机法 | O（n）

- **思路**：

  ![1654261550414](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1654261550414.png)

  1. 根据题意划分为 4 种状态，它们之间的转换关系如上图所示。
  2. 根据这样的自动机来编程，可以提高程序可读性，避免臃肿的代码。

- **结论**：时间，1 ms，100%，空间，41.4 mb，30.92%，时间上，由于需要遍历 n 长的 chars 数组，所以时间复杂度为 O（n），由于最多只需要有限几个变量和一个内容有限的 map，所以额外空间复杂度为 O（1）。

```java
/* 
* 自动机:
*              ' '	    +/-	    number	    other
* start	       start	signed	in_number	end
* signed	   end	    end	    in_number	end
* in_number    end	    end	    in_number	end
* end	       end	    end	    end	        end
*/
class Automaton {

    private static final Map<String, String[]> STATE_MAP = new HashMap<>(){{
        put("start", new String[] { "start", "signed", "in_number", "end" });
        put("signed", new String[] { "end", "end", "in_number", "end" });
        put("in_number", new String[] { "end", "end", "in_number", "end" });
        put("end", new String[] { "end", "end", "end", "end" });
    }};

    private int ans = 0;
    private int flag = 1;
    private int adigit;
    private boolean hasOverflow = false;
    private String lastState = "start";

    Automaton() {
        
    }

    public int getAns() {
        return this.hasOverflow? this.ans : this.flag * this.ans;
    }

    public void add(char c) {
        // 出现了溢出, 则直接返回
        if(this.hasOverflow) {
            return;
        }

        // 符号状态
        this.lastState = STATE_MAP.get(this.lastState)[getCol(c)];
        if(this.lastState == "signed") {
            this.flag = c == '-'? -1 : 1;
        } 
        // 数字状态
        else if(this.lastState == "in_number") {
            if(this.ans > (Integer.MAX_VALUE - (this.adigit = c - '0')) / 10) {
                this.ans = this.flag == -1? Integer.MIN_VALUE : Integer.MAX_VALUE;
                this.hasOverflow = true;
            } else {
                this.ans = this.ans * 10 + this.adigit;
            }
        } 
    }

    private int getCol(char c) {
        if(c == ' ') {
            return 0;
        } else if(c == '+' || c == '-') {
            return 1;
        } else if(Character.isDigit(c)) {
            return 2;
        } else {
            return 3;
        }
    }
}

class Solution {
    public int myAtoi(String s) {
        if(s == null || s.length() == 0) {
            return 0;
        }
        
        Automaton automaton = new Automaton();
        char[] chars = s.toCharArray();

        for(int i = 0; i < chars.length; i++) {
            automaton.add(chars[i]);
        }

        return automaton.getAns();
    }
}
```

#### 12. 整数转罗马数字 | medium

##### 1）模拟法 | O（13）

- **思路**：模拟转罗马数字的方式，从大到小依次尝试，符合的则加入字符串，然后开始新的尝试，否则就进行一次项的尝试。
- **结论**：时间，2 ms，100%，空间，41.2 mb，39.83%，时间上，由于 trys 数组长 13，所以遍历找到一个对应的值，最大需要 O（13），由于使用了 2 个 13 长度的数组，所以额外空间复杂度为 O（26）。

```java
class Solution {

    private static final int[] trys = new int[] {
         1, 4, 5, 9, 10, 40, 50, 90, 100, 400, 500, 900, 1000 
    };
    private static final String[] strs = new String[] { 
        "I", "IV", "V", "IX", "X", "XL", "L", "XC", "C", "CD", "D", "CM", "M" 
    };

    public String intToRoman(int num) {
        StringBuilder sb = new StringBuilder();

        // 开始尝试
        int start = trys.length - 1;
        for(int i = start; i > -1 && num > 0; i--) {
            if(num >= trys[i]) {
                num -= trys[i];
                sb.append(strs[i]);
                i = start + 1;
            }
        }

        return sb.toString();
    }
}
```

##### 2）二分查找 | O（log 13）

- **思路**：
  1. 研究 1 的做法可发现，每次尝试都需要从最大值开始尝试，又由于 trys 数组本身是有序的，所以可以使用二分查找，找到第一个比当前 num 大或者等于的索引，可以减少很多次尝试。
  2. 其中，二分查找需要分情况讨论：
     - 1）mid 直接命中：此时直接返回 mid 即可。
     - 2）mid 在坡上，但当前 mid 刚好比 target 小一点，也就是命中，则直接返回 mid 即可。
     - 3）mid 在坡上，但当前区间没有命中，也就是当前 mid 比 target 小很多，说明 target 在 mid + 1 的右边，则继续往右二分。
     - 4）mid 在坡下，但当前 mid 刚好比 target 大一点，也就是命中，则直接返回 mid  - 1即可。
     - 5）mid 在坡下，但当前区间没有命中，也就是当前 mid 比 target 大很多，说明 target 在 mid - 1 的左边，则继续往左二分。
- **结论**：时间，3 ms，95.84%，空间，41.2 mb，42.24%，时间上，由于 trys 数组长 13，所以二分找到一个对应的值，最大需要 O（log 13），由于使用了 2 个 13 长度的数组，所以额外空间复杂度为 O（26）。

```java
class Solution {

    private static final int[] trys = new int[] {
         1, 4, 5, 9, 10, 40, 50, 90, 100, 400, 500, 900, 1000 
    };
    private static final String[] strs = new String[] { 
        "I", "IV", "V", "IX", "X", "XL", "L", "XC", "C", "CD", "D", "CM", "M" 
    };

    // 二分查找
    private int index(int target) {
        return f(target, 1, 12);
    }
    private int f(int target, int l, int r) {
        if(l >= r) {
            return l;
        }

        int mid = (l + r) >> 1;

        // 中点命中
        if(target == trys[mid]) {
            return mid;
        } 
        // 坡上
        else if(target > trys[mid]) {
            // 坡上, 当前区间命中
            if(mid + 1 < 13 && target < trys[mid + 1]) {
                return mid;
            } 
            // 坡上, 当前区间没命中
            else {
                return f(target, mid + 1, r);
            }
        } 
        // 坡下
        else {
            // 坡下, 当前区间命中
            if(mid - 1 > -1 && target > trys[mid - 1]) {
                return mid - 1;
            } 
            // 坡下, 当前区间没命中
            else {
                return f(target, l, mid);
            }
        }
    }

    public String intToRoman(int num) {
        StringBuilder sb = new StringBuilder();

        // 开始尝试
        int start = trys.length - 1;
        for(int i = index(num); i > -1 && num > 0; i--) {
            if(num >= trys[i]) {
                num -= trys[i];
                sb.append(strs[i]);
                i = index(num) + 1;
            }
        }

        return sb.toString();
    }
}
```

##### 3）打表法 | O（1）

- **思路**：
  1. 经过研究数据状态，可以发现：
     - 1）千位数字，只能由 'M' 来组合表示。
     - 2）百位数字，只能由 'C'、'CD'、'D' 和 'CM' 来组合表示。
     - 3）十位数字，只能由 'X'、'XL'、'L' 和 'XC' 来组合表示。
     - 4）个位数字，只能由 'I'、'IV'、'V' 和 'IX' 来组合表示。
  2. 这就恰好把这 13个符号分为四组，而且，组与组之间没有公共的符号。因此，整数 num 的十进制中的每一个数字，都是可以被单独处理，组成一张硬编码表，如题中所示。
- **结论**：时间，2 ms，100%，空间，41.1 mb，55.33%，时间复杂度为 O（1），空间复杂度为 O（4 + 3 * 10） = O（34）。

```java
/**
* 打表法：
*
*   千  百  十	  个
* 0	-	-	-	-
* 1	M	C	X	I
* 2	MM	CC	XX	II
* 3	MMM	CCC	XXX	III
* 4	-	CD	XL	IV
* 5	-	D	L	V
* 6	-	DC	LX	VI
* 7	-	DCC	LXX	VII
* 8	- DCCC LXXX VIII
* 9	-	CM	XC	IX
 */
class Solution {

    private static final String[] qians = new String[] { 
        "", "M", "MM", "MMM"
    };
    private static final String[] bais = new String[] { 
        "", "C", "CC", "CCC", "CD", "D", "DC", "DCC", "DCCC", "CM", 
    };
    private static final String[] shis = new String[] { 
        "", "X", "XX", "XXX", "XL", "L", "LX", "LXX", "LXXX", "XC", 
    };
    private static final String[] ges = new String[] { 
        "", "I", "II", "III", "IV", "V", "VI", "VII", "VIII", "IX", 
    };

    public String intToRoman(int num) {
        StringBuilder sb = new StringBuilder();
        
        // 千: 十进制右移3位
        sb.append(qians[num / 1000]);
        
        // 百: 十进制取低3位, 再右移2位
        sb.append(bais[(num % 1000) / 100]);

        // 十: 十进制取低2位, 再右移1位
        sb.append(shis[(num % 100) / 10]);

        // 个: 十进制取低1位
        sb.append(ges[num % 10]);

        return sb.toString();
    }
}
```

#### 13. 罗马数字转整数 | easy

##### 1）暴力解法 | O（3999）

- **思路**：直接遍历 4000 个数字，挨个转换为字符串比较是否相等，是的话则返回数字即可。
- **结论**：时间，755 ms，5.72%，空间，42 mb，11.90%，时间上，最大需要遍历 3999 次，每次的 `intToRoman()` 方法只需要 O（1），所以时间复杂度为 O（3999），空间上，由于最多需要建立 `3999` 的 StringBuilder `MMMCMXCIX`，取建立 9 长的 char 数组，所以额外空间复杂度为 O（9）。

```java
/**
* 打表法：
*
*   千  百  十	  个
* 0	-	-	-	-
* 1	M	C	X	I
* 2	MM	CC	XX	II
* 3	MMM	CCC	XXX	III
* 4	-	CD	XL	IV
* 5	-	D	L	V
* 6	-	DC	LX	VI
* 7	-	DCC	LXX	VII
* 8	- DCCC LXXX VIII
* 9	-	CM	XC	IX
 */
class Solution {

    private static final String[] qians = new String[] { 
        "", "M", "MM", "MMM"
    };
    private static final String[] bais = new String[] { 
        "", "C", "CC", "CCC", "CD", "D", "DC", "DCC", "DCCC", "CM", 
    };
    private static final String[] shis = new String[] { 
        "", "X", "XX", "XXX", "XL", "L", "LX", "LXX", "LXXX", "XC", 
    };
    private static final String[] ges = new String[] { 
        "", "I", "II", "III", "IV", "V", "VI", "VII", "VIII", "IX", 
    };

    private String intToRoman(int num) {
        StringBuilder sb = new StringBuilder();
        
        // 千: 十进制右移3位
        sb.append(qians[num / 1000]);
        
        // 百: 十进制取低3位, 再右移2位
        sb.append(bais[(num % 1000) / 100]);

        // 十: 十进制取低2位, 再右移1位
        sb.append(shis[(num % 100) / 10]);

        // 个: 十进制取低1位
        sb.append(ges[num % 10]);

        return sb.toString();
    }

    public int romanToInt(String s) {
        if(s == null || s.length() == 0) {
            return -1;
        }

        for(int i = 1; i < 4000; i++) {
            if(s.equals(intToRoman(i))) {
                return i;
            }
        }

        return -1;
    }
}
```

##### 2）模拟法 | O（n）

- **思路**：经过研究罗马数字的规律，发现是长的优先，所以先用长字符去匹配累加，如果匹配不到，才用单个字符去匹配累加。
- **结论**：时间，4 ms，61.54%，空间，41.9 mb，23.80%，时间上，由于遍历一次字符串，所以时间复杂度为 O（n），空间上，由于使用了一张 13 长的哈希表，所以额外空间复杂度为 O（13）。

```java
class Solution {

    private static final Map<String, Integer> map = new HashMap<>() {{
        put("I", 1);  put("IV", 4);  put("IX", 9); 
        put("V", 5);  
        put("X", 10); put("XL", 40); put("XC", 90);
        put("L", 50); 
        put("C", 100); put("CD", 400); put("CM", 900);
        put("D", 500); 
        put("M", 1000); 
    }};
    
    public int romanToInt(String s) {
        if(s == null || s.length() == 0) {
            return -1;
        }

        int res = 0;
        String tmps;
        Integer tmp;
        for(int i = 0; i < s.length(); i++) {
            if(i + 1 < s.length()) {
                tmp = map.get(s.substring(i, i + 2));
                if(tmp != null) {
                    res += tmp;
                    i++;
                    continue;
                }
            }

            tmp = map.get(s.substring(i, i + 1));
            res += tmp;
        }

        return res;
    }
}
```

##### 3）模拟法2 | O（n）

- **思路**：经过研究发现，通常情况下，罗马数字中小的数字在大的数字的右边，如果右边的比左边的大, 那么当前字符需要减掉，否则, 则当前字符需要加上。
- **结论**：时间，3 ms，74.70%，空间，41.2 mb，88.25%，时间上，由于需要遍历一次字符串，所以时间复杂度为 O（n），空间上，由于使用了一张长 7 长的哈希表，所以额外空间复杂度为 O（9）。

```java
class Solution {

    private static final Map<Character, Integer> map = new HashMap<>() {{
        put('I', 1);
        put('V', 5);  
        put('X', 10);
        put('L', 50); 
        put('C', 100);
        put('D', 500); 
        put('M', 1000); 
    }};
    
    public int romanToInt(String s) {
        if(s == null || s.length() == 0) {
            return -1;
        }

        int res = 0, tmp;
        char[] chars = s.toCharArray();
        for(int i = 0; i < chars.length; i++) {
            // 通常情况下，罗马数字中小的数字在大的数字的右边
            // 如果右边的比左边的大, 那么当前字符需要减掉
            tmp = map.get(chars[i]);
            if(i + 1 < chars.length && map.get(chars[i + 1]) > tmp) {
                res -= tmp;
            }
            // 否则, 则当前字符需要加上
            else {
                res += tmp;
            }
        }

        return res;
    }
}
```

#### 14. 最长公共前缀 | easy

##### 1）模拟法 - 纵向扫描每位字符1 | O（m^2 * n）

- **思路**：
  1. 先取第一个字符串作为比较基准，遍历 m 次它的子串，但要注意从后往前遍历，减少遍历次数。
  2. 每次子串遍历过程中，都与其他字符串做前缀判断，全都符合的直接返回作为结果即可。
- **结论**：时间，2 ms，19.43%， 空间，39.2 mb，68.90%，时间上，str[0] 遍历子串最多花费 O（m），每次遍历过程中判断其他字符串花费 O（n），每次判断使用了 `startsWith()` ，花费时间取决于子串的长度 O（m），所以时间复杂度为 O（m^2 * n），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        if(strs == null || strs.length == 0) {
            return "";
        }
        if(strs.length == 1) {
            return strs[0];
        }

        boolean isAll;
        String res = "", tmp;
        for(int len = strs[0].length(); len > -1; len--) {
            tmp = strs[0].substring(0, len);

            isAll = true;
            for(int i = 1; i < strs.length; i++) {
                if(!strs[i].startsWith(tmp)) {
                    isAll = false;
                    break;
                }
            }
            if(isAll) {
                res = tmp;
                break;
            }
        }        

        return res;
    }
}
```

##### 2）模拟法 - 纵向扫描每位字符2 | O（m * n）

- **思路**：经过研究 1 的思路发现，由于每次纵向比较都从头开始重新比较，每次都浪费了 m 次的性能，所以在本思路中，利用上了前一个字符相同的特点，每次只比较当前字符就好，提升了性能。
- **结论**：时间，1 ms，69.16%，空间，39.1 mb，78.91%，时间上，遍历第一个字符串的长度，花费 O（m），遍历 strs 字符数组，花费 O（n），所以时间复杂度为 O（m * n），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        if(strs == null || strs.length == 0) {
            return "";
        }
        if(strs.length == 1) {
            return strs[0];
        }

        char c;
        for(int i = 0; i < strs[0].length(); i++) {
            c = strs[0].charAt(i);
            for(int j = 1; j < strs.length; j++) {
                if(i == strs[j].length() || strs[j].charAt(i) != c) {
                    return strs[0].substring(0, i);
                }
            }                     
        }

        return strs[0];
    }
}
```

##### 3）横向扫描每个字符串 | O（m * n）

- **思路**：
  1. 经过研究，求最长公共前缀这个过程，符合结合法，即 f(s1, ..., sn) = f( f( f(s1, f2), s3), ..., sn)，即每一个 sm 都决定前缀的长度，由于结合法并不会漏掉任意一个，从而保证结果不变。
  2. 故，设计一个 f（s1， s2） 函数来获取 s1 和 s2 的最长公共前缀，这时，只需要从头遍历 strs 到尾，每个都调一次 f 函数，即可得到最终结果。
- **结论**：时间，1 ms，69.21%，空间，40.8 mb，6.60%，时间上，遍历最初的字符串花费 O（m），遍历 strs 花费 O（n），所以时间复杂度为 O（m * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        if(strs == null || strs.length == 0) {
            return "";
        }
       
        String res = strs[0];
        for(int i = 1; i < strs.length; i++) {
            res = f(res, strs[i]);
        }

        return res;
    }

    // f(s1, ..., sn) = f( f( f(s1, f2), s3), ..., sn)
    private String f(String s1, String s2) {
        if(s1 == "" || s2 == "") {
            return "";
        }

        char[] chars1 = s1.toCharArray();
        char[] chars2 = s2.toCharArray();

        // 遍历短的串
        StringBuilder sb = new StringBuilder();
        for(int i = 0; i < Math.min(chars1.length, chars2.length); i++) {
            if(chars1[i] == chars2[i]) {
                sb.append(chars1[i]);
            } else {
                break;
            }
        }

        return sb.toString();
    }
}
```

##### 4）归并扫描每个字符串 | O（m * logn）或者 O（n）

- **思路**：

  1. 同理，由于最长公共前缀这个过程，符合结合法，即 f(s1, ..., sn) = f( f( f(s1, f2), s3), ..., sn)，即每一个 sm 都决定前缀的长度，由于结合法并不会漏掉任意一个，从而保证结果不变。
  2. 所以，可以使用归并的方式进行扫描，merge 的操作实际上就为之前的 f 函数。

- **结论**：

  1. 时间，1 ms，69.21%，空间，41.1 mb，5.10%。

  2. 时间上，归并操作符合 Master 公式，即 T（n）= 2 * T(n / 2) + O（m），m 是指最长字符串的长度，可见这里， a = 2，b = 2，d = log [ m (N) ]：

     - 1）当字符串长度 m > n，即 d > log [ b (a) ] 时，时间复杂度为 O（m）。
     - 2）当字符串长度 m = n，即 d = log [ b(a) ] 时，时间复杂度为 O（m * logn）。
     - 3）当字符串长度 m < n，即 d < log [ b(a) ] 时，时间复杂度为 O（n）。

     => 故，最大的时间复杂度为 O（m * logn）或者 O（n）。

     ![1654349496899](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1654349496899.png)

  3. 空间上，由于递归深度最大为 logn，所以额外空间复杂度为 O（logn）。

```java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        if(strs == null || strs.length == 0) {
            return "";
        }

        return f(strs, 0, strs.length - 1);
    }

    private String f(String[] strs, int l, int r) {
        if(l >= r) {
            return strs[l];
        }

        int mid = (r + l) >> 1;
        String ls = f(strs, l, mid);
        String rs = f(strs, mid + 1, r);
        return merge(ls, rs);
    }

    // merge(s1, ..., sn) = merge( f( f(s1, f2), s3), ..., sn)
    private String merge(String s1, String s2) {
        if(s1 == "" || s2 == "") {
            return "";
        }

        char[] chars1 = s1.toCharArray();
        char[] chars2 = s2.toCharArray();

        // 遍历短的串
        StringBuilder sb = new StringBuilder();
        for(int i = 0; i < Math.min(chars1.length, chars2.length); i++) {
            if(chars1[i] == chars2[i]) {
                sb.append(chars1[i]);
            } else {
                break;
            }
        }

        return sb.toString();
    }
}
```

##### 5）二分扫描最长公共前缀 | O（m * n * logm）

- **思路**：
  1. 经过研究发现，最长公共前缀的长度从 [0, 1, ..., n]，是有序的，所以可以进行二分。
  2. 首先，找到最小的长度，作为最长的 r 进行二分，以减少循环的次数，然后选择 0 作为 l，作为二分的起点，以保证不存在任何公共子串时，还能返回空串。
  3. 然后，每次找到 [l, r] 之间的中点，判断以中点为长度的子串，是否符合公共前缀，如果是的话，就更新 l 为 mid，说明最长公共前缀必定 >= mid，如果不是的话，就更新 r 为 mid - 1，说明最长公共前缀必定比 mid 小。其中要注意的是，求 mid 时，为了避免 [0, 1] 这种二分有出口，所以每次 mid = (r + l + 1) / 2，多加一个 1，所以，二分这种中点、l、r 的取值逻辑，是不固定的，需要视具体的情况做相应的改造。
  4. 最后，就是判断 len 长度的子串，是否符合公共前缀的方法实现了，取第一个字符串作为判断基准（取哪个都一样），然后截取 len 长的子串，从头判断每个字符是否都相同，都是的话就返回 true，否则返回 false。
- **结论**：时间，1 ms，69.16%，空间，39.3 mb，54.88%，时间上，找到最小长度，花费 O（n），二分最小长度，花费 O（log m），每次二分时都做公共前缀判断，花费 O（m），所以时间复杂度为 O（m * n * logm），空间上，由于只是用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        if(strs == null || strs.length == 0) {
            return "";
        }
        if(strs.length == 1) {
            return strs[0];
        }

        int minLen = Integer.MAX_VALUE;
        for(int i = 0; i < strs.length; i++) {
            minLen = Math.min(minLen, strs[i].length());
        }

        int l = 0, r = minLen, mid;
        while(l < r) {
            // +1保证[0, 1]二分不会死循环, 必定会有出口
            mid = (r + l + 1) >> 1;
            
            // 如果mid是最长公共前缀, 那么再看看是否存在>=mid公共前缀, 有就更新, 没有就mid了
            if(hasLenCommonPrefix(strs, mid)) {
                l = mid;
            } 
            // 如果mid不是最长公共前缀, 说明结果应该更小, 所以再往左看
            else {
                r = mid - 1;
            }
        }

        // 返回最长公共前缀
        return strs[0].substring(0, l);
    }

    // 获取strs[0]中长度为len的字符串, 是否符合公共前缀
    private boolean hasLenCommonPrefix(String[] strs, int len) {
        String str0 = strs[0].substring(0, len);

        char c;
        for(int i = 0; i < str0.length(); i++) {
            c = str0.charAt(i);

            // 由于len小于、或者等于数组中最小长度的字符串, 所以无需再做长度溢出判断
            for(int j = 1; j < strs.length; j++) {
                if(strs[j].charAt(i) != c) {
                    return false;
                }
            }
        }

        return true;
    }
}
```

#### 38. 外观序列 | medium

##### 1）模拟 + 遍历 | O（n * m）

- **思路**：模拟 n 从 1~n，一次生成对应的外观序列，其中，外观序列的生成逻辑为，从头遍历，记录每段连续字符出现的次数，碰到新的字符以及到数组末尾时，则结算字符次数，生成 n 个 ["次数" + "字符"] 的连续序列作作为结果即可。
- **结论**：时间，1 ms，95.71%，空间，39.2 mb，58.16%，时间上，需要遍历 n 次花费 O（n），设生成的外观序列最长为 m，那么每次遍历最大花费 O（m），所以整体时间复杂度为 O（n * m），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {

    public String countAndSay(int n) {
        String res = "1";
        for(int i = 2; i <= n; i++) {
            res = getCntStr(res);
        }
        return res;
    }

    private String getCntStr(String s) {
        if(s == null || s.length() == 0) {
            return "";
        }

        char[] chars = s.toCharArray();
        int pre = chars[0] - '0', cur, cnt = 1;
        StringBuilder sb = new StringBuilder(pre);
        for(int i = 1; i < chars.length; i++) {
            cur = chars[i] - '0';
            
            // 与前面的字符相同, 则累加出现次数
            if(cur == pre) {
                cnt++;
            } 
            // 与前面字符不同, 则结算字符串
            else {
                sb.append(cnt).append(pre);
                pre = cur;
                cnt = 1;
            }
        }

        // 最后再追加一次
        sb.append(cnt).append(pre);
        return sb.toString();
    }
}
```

#### 43. 字符串相乘 | medium

##### 1）竖式乘法 + 暴力模拟 | O（slen * [ slen + 4 * glen ] ）

- **思路**：
  1. 模拟乘法过程，短者的低位乘上长者的每一位，得到余数则加入结果，同时累加进位。
  2. 然后，反转过来即得到一位乘法的结果，不过需要注意，不是最低位的乘法时，还需要往结果后面补 0，表示 * 10 倍。
  3. 最后，每位乘法结束后，采用《415. 字符串相加》的方案，累加当前乘算结果 和 上次乘算结果，而最终的累加结果就是答案。
  4. 其中，还需要注意 * 0 的情况，这种算法如果不做提前判断，有肯能会得到 "0000" 的结果，但题目是不要求前导 0 的，所以只能先做提前判断，即如果 num1 和 nums2 任意有一个为 "0"，则直接返回 "0" 即可，因为 0 乘上任何数都是 0。
- **结论**：
  1. 时间，12 ms，38.34%，空间，42.4 mb，5.11%。
  2. 时间上，模拟 slen 次低位乘法，花费 O（slen），每次低位乘法需要模拟 glen 次拼接字符串，花费 O（glen），同时每次拼接之后还需要 O（glen）反转，以及 O（slen）追加 0 ，和 O（2 * glen）进行竖式加法，所以整体时间复杂度为 O（slen * [ slen + 4 * glen ] ）。
  3. 空间上，需要 slen 长的 shorts 数组，glen 长的 glen 数组，以及 slen 个 glen 长的 StringBuilder 对象，竖式加法花费 O（1）额外空间，所以整体的额外空间复杂度为 O（slen + glen + slen * glen）。

```java
class Solution {
    public String multiply(String num1, String num2) {
        String res = "0";
        if(res.equals(num1) || res.equals(num2)) {
            return res;
        }

        char[] chars1 = num1.toCharArray();
        char[] chars2 = num2.toCharArray();
        int m = num1.length(), n = num2.length();

        // 较短者shorts, 较长者longs
        char[] shorts = m > n? chars2 : chars1;
        char[] longs = shorts == chars1? chars2 : chars1;
        int slen = shorts.length, glen = longs.length;
        
        // 模拟从低位乘到高位: 123 * 45
        int x, y, sum, carry;
        for(int i = slen - 1; i > -1; i--) {
            carry = 0;
            x = shorts[i] - '0';
    
            StringBuilder sb = new StringBuilder();
            for(int j = glen - 1; j > -1; j--) {
                y = longs[j] - '0';
                sum = x * y + carry;
                carry = sum / 10;
                sb.append(sum % 10);
            }

            // 如果最后还有进位, 那么还需要追加结果
            if(carry > 0) {
                sb.append(carry);
            }

            // 反转后、再追加i个0
            sb.reverse();
            for(int k = 1; k < slen - i; k++) {
                sb.append("0");
            }

            res = addStrings(res, sb.toString());
        }

        return res;
    }

    // 竖式加法
    public String addStrings(String num1, String num2) {
        char[] chars1 = num1.toCharArray();
        char[] chars2 = num2.toCharArray();
        int i = chars1.length - 1, j = chars2.length - 1;

        // 开始模拟累加: 至少存在i、j或者进位, 才进行累加
        int carry = 0, x, y, sum;
        StringBuilder sb = new StringBuilder();
        while(i >= 0 || j >= 0 || carry > 0) {
            x = i >= 0? chars1[i] - '0' : 0;
            y = j >= 0? chars2[j] - '0' : 0;
            sum = x + y + carry;
            sb.append(sum % 10);
            
            carry = sum / 10;
            i--;
            j--;
        }

        // 反转结果返回
        sb.reverse();
        return sb.toString();
    }
}
```

##### 2）哈希表 + 乘法替代加法 + 延后进位 |  O（slen * glen + 2 * [slen + glen]）

- **思路**：
  1. 由于 num1 和 nums2 为非负整数，所以最小的时候为 10^[m-1] （参考 1 这个数），最大的时候为 10^m - 1，所以 nums1 * nums2 的长度，最小时为 10^[m + n - 2]，最大时为 10^[m + n] - 10^m - 10^n + 1，可见，nums1 * nums2 的极限长度应该为 O（m + n）。
  2. 因此，可以创建一个 m + n 长的数组，来存储合法 i、j 的相乘结果 arr[i + j + 1]，然后可以利用这个 m * n 的数组，进行进位更新、以及最终结果拼接。
- **结论**：
  1. 时间，1 ms，100%，空间，41.5 mb，48.59%。
  2. 时间上，枚举模拟 i、j 花费 O(slen * glen)，处理进位数据花费 O（slen + glen），拼接结果花费 O（slen + glen），所以整体时间复杂度为 O（slen * glen + 2 * [slen + glen]）。
  3. 空间上，需要 slen 长的 shorts 数组，glen 长的 glen 数组，以及一个 slen + glen 长的 arr 数组，所以整体的额外空间复杂度为 O（ 2 * [slen + glen] ）。

```java
class Solution {
    public String multiply(String num1, String num2) {
        if("0".equals(num1) || "0".equals(num2)) {
            return "0";
        }

        char[] chars1 = num1.toCharArray();
        char[] chars2 = num2.toCharArray();
        int m = num1.length(), n = num2.length();

        // 较短者shorts, 较长者longs
        char[] shorts = m > n? chars2 : chars1;
        char[] longs = shorts == chars1? chars2 : chars1;
        int slen = shorts.length, glen = longs.length;
        
        // 模拟从低位乘到高位: 123 * 45
        int x, y;
        int[] arr = new int[slen + glen];
        for(int i = glen - 1; i > -1; i--) {
            x = longs[i] - '0';

            for(int j = slen - 1; j > -1; j--) {
                y = shorts[j] - '0';

                // 从1开始存起, 0位表示最高位(可能会有进位)
                arr[i + j + 1] += x * y;
            }
        }

        // 从后往前处理>10的数, 因为可以不断地确定遍历到的i元素
        for(int i = arr.length - 1; i > 0; i--) {
            // 如果存在进位, 则累加到前面一个元素
            arr[i - 1] += arr[i] / 10;

            // 取摸求余
            arr[i] %= 10;
        }

        // 处理最高位是否位进位
        int start = arr[0] == 0? 1 : 0;
        StringBuilder sb = new StringBuilder();
        for(int i = start; i < arr.length; i++) {
            sb.append(arr[i]);
        }

        return sb.toString();
    }
}
```

#### 58. 最后一个单词的长度 | easy

##### 1）系统函数法 | O（n）

- **思路**：使用 split(..) 函数，由于其自带 trim() 功能，所以直接取分割出来的数组 strs，中的最后一个元素，的长度作为返回结果即可。
- **结论**：时间，0 ms，100%，空间，39.7 mb，31.02%，时间上，由于 split(..) 函数花费 O（n），所以时间复杂度为 O（n），空间上，由于使用了 n - m 个字符串 strs 数组，n 代表原来的字符数，m 代表去掉空格后的字符数，所以额外空间复杂度为 O（n - m）。

```java
class Solution {
    public int lengthOfLastWord(String s) {
        // split(..)函数自带trim()功能
        String[] strs = s.split(" ");
        return strs[strs.length - 1].length();
    }
}
```

##### 2）反向模拟法 | O（n）

- **思路**：反向遍历，统计最后一个空格，到最后一个单词前的第一个空格，之间的字符数即可。
- **结论**：时间，0 ms，100%，空间，39.7 mb，44.43%，时间上，由于只需要遍历一次 chars 数组，所以时间复杂度为 O（n），空间上，由于使用了一张 n 长的 chars 数组，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public int lengthOfLastWord(String s) {
        char[] chars = s.toCharArray();

        int cnt = 0;
        for(int i = chars.length - 1; i > -1; i--) {
            // 空格
            if(chars[i] == ' ') {
                // 最后一个空格
                if(cnt == 0) {
                    continue;
                } 
                // 最后一个单词前的第一个空格
                else {
                    break;
                }
            } 
            // 非空格
            else {
                cnt++;
            }
        }

        return cnt;
    }
}
```

#### 65. 有效数字 | hard

##### 1）暴力模拟 | O（n）

- **思路**：见代码注释。
- **结论**：时间，1 ms，100%，空间，40.9 mb，97.08%，时间上，由于只需要遍历一次字符串，所以时间复杂度为 O（n），空间上，由于使用了一张 n 长的 chars 数组，所以额外空间复杂度为 O（n）。

```java 
class Solution {
    public boolean isNumber(String s) {
        char[] chars = s.toCharArray();
        
        // 检查开头
        char c = chars[0];
        if(!Character.isDigit(c) && c != '+' && c != '-' && c != '.') {
            return false;
        }

        // 检查结尾
        int n = chars.length;
        c = chars[n - 1];
        if(!Character.isDigit(c) && c != '.') {
            return false;
        }

        // 检查中间
        boolean hasE = false, hasPoint = false;
        for(int i = 0; i < n; i++) {
            c = chars[i];
            
            // e/E：可选, 只有一个, 必须后跟整数, 前面必须为整数或者小数点
            if(c == 'e' || c == 'E') {
                if(hasE) {
                    return false;
                }

                hasE = true;
                if(i + 1 < n && (Character.isDigit(chars[i + 1]) || chars[i + 1] == '+' || chars[i + 1] == '-') 
                    && i - 1 > -1 && (Character.isDigit(chars[i - 1]) || chars[i - 1] == '.')
                ) {
                    // do nothing
                } else {
                    return false;
                }
            }
            // 符号位：可选, 只有一个, 必须开头
            else if(c == '+' || c == '-') {
                if(i == 0 
                    || (i - 1 > - 1 && (chars[i - 1] == 'e' || chars[i - 1] == 'E') && i + 1 < n && Character.isDigit(chars[i + 1]))) {
                    // do nothing
                } else {
                    return false;
                }
            }
            // 小数点：可选, 只有一个, 数字+'.'、数字+'.'+数字、'.'+数字, 且必须在E前面
            else if(c == '.') {
                if(hasPoint || hasE) {
                    return false;
                }

                hasPoint = true;
                if((i - 1 > - 1 && Character.isDigit(chars[i - 1])) 
                    || (i + 1 < n && Character.isDigit(chars[i + 1])) 
                ) {
                    // do nothing
                } else {
                    return false;
                }
            }
            // 数字位：必须位数字
            else {
                if(!Character.isDigit(c)) {
                    return false;
                }
            }
        }

        return true;
    }
}
```

##### 2）状态自动机法 | O（n）

- **思路**：根据题意划分状态，提前确定好之间的转换关系，如下图所示，可以提高程序可读性，避免臃肿的代码。

  ![1658219358208](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1658219358208.png)

- **结论**：时间，3 ms，37.96%，空间，41.3 mb，68.78%，时间上，由于只需要遍历一次字符串，所以时间复杂度为 O（n），空间上，几张固定大小的 map 集合，花费 O（1），n 长的 chars 数组，花费 O（n），所以整体的额外空间复杂度为 O（n）。

```java 
// 字符类型
enum CharType {
    CHAR_NUMBER,// 数字
    CHAR_EXP,// 指数字符
    CHAR_POINT,// 小数点
    CHAR_SIGN,// 符号
    CHAR_ILLEGAL// 非法字符
}

// 状态类型
enum State {
    STATE_INITIAL,// 初始状态
    STATE_INT_SIGN,// 整数部分的符号位
    STATE_INTEGER,// 整数部分
    STATE_POINT,// 小数点(左侧有整数)
    STATE_POINT_WITHOUT_INT,// 小数点(左侧无整数)
    STATE_FRACTION,// 小数部分
    STATE_EXP,// 指数字符
    STATE_EXP_SIGN,// 指数部分的符号位
    STATE_EXP_NUMBER// 指数部分的整数
}

class Solution {
    // 目标状态集合-初始状态
    private static final Map<CharType, State> initial2Map = new HashMap<>() {{
        put(CharType.CHAR_NUMBER, State.STATE_INTEGER);
        put(CharType.CHAR_SIGN, State.STATE_INT_SIGN);
        put(CharType.CHAR_POINT, State.STATE_POINT_WITHOUT_INT);
    }};

    // 目标状态集合-整数部分的符号位 
    private static final Map<CharType, State> intSign2Map = new HashMap<>() {{
        put(CharType.CHAR_NUMBER, State.STATE_INTEGER);
        put(CharType.CHAR_POINT, State.STATE_POINT_WITHOUT_INT);
    }};

    // 目标状态集合-整数部分 
    private static final Map<CharType, State> integer2Map = new HashMap<>() {{
        put(CharType.CHAR_EXP, State.STATE_EXP);
        put(CharType.CHAR_POINT, State.STATE_POINT);
        put(CharType.CHAR_NUMBER, State.STATE_INTEGER);
    }};

    // 目标状态集合-小数点(左侧有整数)
    private static final Map<CharType, State> point2Map = new HashMap<>() {{
        put(CharType.CHAR_EXP, State.STATE_EXP);
        put(CharType.CHAR_NUMBER, State.STATE_FRACTION);
    }};

    // 目标状态集合-小数点(左侧无整数)
    private static final Map<CharType, State> pointWithoutInt2Map = new HashMap<>() {{
        put(CharType.CHAR_NUMBER, State.STATE_FRACTION);
    }};

    // 目标状态集合-小数部分
    private static final Map<CharType, State> fraction2Map = new HashMap<>() {{
        put(CharType.CHAR_EXP, State.STATE_EXP);
        put(CharType.CHAR_NUMBER, State.STATE_FRACTION);
    }};

    // 目标状态集合-指数字符
    private static final Map<CharType, State> exp2Map = new HashMap<>() {{
        put(CharType.CHAR_SIGN, State.STATE_EXP_SIGN);
        put(CharType.CHAR_NUMBER, State.STATE_EXP_NUMBER);
    }};

    // 目标状态集合-指数部分的符号位
    private static final Map<CharType, State> expSign2Map = new HashMap<>() {{
        put(CharType.CHAR_NUMBER, State.STATE_EXP_NUMBER);
    }};

    // 目标状态集合-指数部分的整数
    private static final Map<CharType, State> expNumber2Map = new HashMap<>() {{
        put(CharType.CHAR_NUMBER, State.STATE_EXP_NUMBER);
    }};

    // 原始状态的转换集合
    private static final Map<State, Map<CharType, State>> transferMap = new HashMap<>() {{
        put(State.STATE_INITIAL, initial2Map);
        put(State.STATE_INT_SIGN, intSign2Map);
        put(State.STATE_INTEGER, integer2Map);
        put(State.STATE_POINT, point2Map);
        put(State.STATE_POINT_WITHOUT_INT, pointWithoutInt2Map);
        put(State.STATE_FRACTION, fraction2Map);
        put(State.STATE_EXP, exp2Map);
        put(State.STATE_EXP_SIGN, expSign2Map);
        put(State.STATE_EXP_NUMBER, expNumber2Map);
    }};

    // 根据字符, 获取字符类型
    private CharType getCharType(char c) {
        if(Character.isDigit(c)) {
            return CharType.CHAR_NUMBER;
        } else if(c == 'e' || c == 'E') {
            return CharType.CHAR_EXP;
        } else if(c == '.') {
            return CharType.CHAR_POINT;
        } else if(c == '+' || c == '-') {
            return CharType.CHAR_SIGN;
        } else {
            return CharType.CHAR_ILLEGAL;
        }
    }

    public boolean isNumber(String s) {
        char[] chars = s.toCharArray();

        // 一开始为初始状态
        State state = State.STATE_INITIAL;

        // 自动机开始判断
        for(int i = 0; i < chars.length; i++) {
            CharType type = getCharType(chars[i]);
            
            // 如果当前状态, 不能转换到下一个约定好的状态, 则返回false
            Map<CharType, State> type2StateMap = transferMap.get(state);
            if(!type2StateMap.containsKey(type)) {
                return false;
            } 
            // 如果当前状态, 能转换到下一个约定好的状态, 则转换到下一个状态
            else {
                state = type2StateMap.get(type); 
            }
        } 

        // 最后判断是否为可接受状态：整数部分、小数点、小数的整数部分、指数的整数部分
        return state == State.STATE_INTEGER
             || state == State.STATE_POINT 
             || state == State.STATE_FRACTION 
             || state == State.STATE_EXP_NUMBER; 
    }
}
```

#### 66. 加一 | easy

##### 1）竖式加法 + 暴力模拟 | O（2 * n）

- **思路**：模拟竖式加法，从低位开始加起，直到最高位结束，然后设置返回结果即可。
- **结论**：时间，0 ms，100%，空间，40.1 mb，18.87%，时间上，由于最多需要遍历 2 次数组，所以时间复杂度为 O（2 * n），空间上，由于复用了原始数组，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int[] plusOne(int[] digits) {
        int n = digits.length, carry = 1, sum = 0;
        for(int i = n - 1; i > -1; i--) {
            sum = digits[i] + carry;
            digits[i] = sum % 10;
            carry = sum / 10;
        }
        
        // 如果还有进位, 则构造新数组
        if(carry > 0) {
            int[] res = new int[n + 1];
            res[0] = carry;
            System.arraycopy(digits, 0, res, 1, n);
            return res;
        } 
        // 如果没有进位, 则返回原始数组
        else {
            return digits;
        }
    }
}
```

##### 2）模拟 + 最短后缀法 | O（n）

- **思路**：从后往前找，找出最短的 9 后缀，然后根据加 1 规则，设置 0 或者直接累加即可，详细逻辑见代码注释。
- **结论**：时间，0 ms，100%，空间，39.8 mb，63.93%，时间上，由于只需要遍历 1 次数组，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int[] plusOne(int[] digits) {
        int n = digits.length;

        // 从后往前找
        for(int i = n - 1; i > -1; i--) {
            // 找到第一个不为9的元素
            if(digits[i] != 9) {
                // 该元素+1
                digits[i]++;

                // 后面的元素置0
                for(int j = i + 1; j < n; j++) {
                    digits[j] = 0;
                }

                // 返回置0后的原始数组
                return digits;
            }
        }

        // 如果找不到不为9的元素, 即数组中全部为9, 那么构造一个长度+1的新数组, 且第一位为1, 其余为0
        int[] res = new int[n + 1];
        res[0] = 1;
        return res;
    }
}
```

#### 67. 二进制求和 | easy

##### 1）竖式加法 + 双指针 + 模拟 | O（2 * n）

- **思路**：参考《415. 字符串相加》，只需要把 10 进制改为 2 进制即可。
- **结论**：时间，1 ms，99.92%，空间，40 mb，76.48%，时间上，双指针模拟花费 O（n），最后的结果反转花费 O（n），n 位 i、j 中的大者，所以整体时间复杂度为 O（2 * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public String addBinary(String a, String b) {
        char[] chars1 = a.toCharArray();
        char[] chars2 = b.toCharArray();
        int i = chars1.length - 1, j = chars2.length - 1;

        // 开始模拟累加: 至少存在i、j或者进位, 才进行累加
        int carry = 0, x, y, sum;
        StringBuilder sb = new StringBuilder();
        while(i >= 0 || j >= 0 || carry > 0) {
            x = i >= 0? chars1[i] - '0' : 0;
            y = j >= 0? chars2[j] - '0' : 0;
            sum = x + y + carry;
            sb.append(sum % 2);
            
            carry = sum / 2;
            i--;
            j--;
        }

        // 反转结果返回
        sb.reverse();
        return sb.toString();
    }
}
```

#### 68. 文本左右对齐 | hard

##### 1）模拟法 | O（n * [p - q]）

- **思路**：从左到右遍历 words 单词，判断到底属于哪种情况，然后周而复始即可。
  1. 情况一，如果当前行是最后一行，则单词左对齐、且单词之间只有一个空格，最后用空串补位。
  2. 情况二，如果当前行不是最后一行，且当前行只有一个单词，则单词左对齐，最后用空格补位。
  3. 情况三，如果当前行不只有一个单词，则空格按均匀分配，其分配策略是：由于题目要求左空格数要大于右空格数，所以，如果遇到除不尽产生了额外的空格的情况，则需要左边要补充 avg+1 个空格，右边补充 avg 个空格，以平均消耗完这 extra 个空格。
- **结论**：时间，1 ms，53.06%，空间，39.4 mb，89.41%，时间上，设 words 长度为 n，最长的单词长度为 p，最短的单词长度为 q，所以最多需要遍历 n 行，每行最多补充空格 p - q，整体时间复杂度为 O（n * [p - q]），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    private static final String BLANK_STR = " ";

    public List<String> fullJustify(String[] words, int maxWidth) {
        List<String> res = new ArrayList<>();
        int n = words.length, r = 0, l, sumLen;
        while(true) {
            // 开始当前行的格式化
            l = r;
            sumLen = 0;

            // 计算一行上, 到底能放多少个单词, 其中, r-l代表[l,r]之间至少存在一个空格
            while(r < n && (sumLen + words[r].length() + r - l) <= maxWidth) {
                // 符合的, 则累加单词之间的长度
                sumLen += words[r].length();

                // 继续移动到下一个单词, 所以会出现r不在统计范围内的情况
                r++;
            }

            // 如果当前行是最后一行(r用尽了), 则单词左对齐、且单词之间只有一个空格, 最后用空串补位
            if(r == n) {
                StringBuilder sb = joinlr(words, l, r, BLANK_STR);
                sb.append(getBlankStr(maxWidth - sb.length()));
                res.add(sb.toString());
                return res;// 最后一行格式化后, 直接返回即可
            }

            // 如果当前行不是最后一行, 则先统计单词数和需要填充空格的数量
            int numWords = r - l;
            int numSpaces = maxWidth - sumLen;

            // 如果当前行只有一个单词, 则单词左对齐, 最后用空格补位
            if(numWords == 1) {
                StringBuilder sb = new StringBuilder(words[l]);
                sb.append(getBlankStr(numSpaces));
                res.add(sb.toString());
                continue;// 然后继续格式化下一行
            }

            // 如果当前行不只有一个单词, 则空格按均匀分配
            int avg = numSpaces / (numWords - 1);
            int extra = numSpaces % (numWords - 1);

            // 分配策略：由于题目要求左空格数要大于右空格数, 所以, 如果遇到除不尽, 产生了额外的空格的情况, 
            // 则需要左边要补充avg+1个空格, 右边补充avg个空格, 以平均消耗完这extra个空格
            StringBuilder sb = new StringBuilder();
            sb.append(joinlr(words, l, l + extra + 1, getBlankStr(avg + 1)));
            sb.append(getBlankStr(avg));
            sb.append(joinlr(words, l + extra + 1, r, getBlankStr(avg)));
            res.add(sb.toString());

            // 继续下一行的格式化
        }

        // 退出出口仅在最后一行的判断里, 这里不作返回
    }

    // 返回由n长度组成的空串
    private String getBlankStr(int n) {
        StringBuilder sb = new StringBuilder();
        for(int i = 0; i < n; i++) {
            sb.append(BLANK_STR);
        }
        return sb.toString();
    }

    // 使用link连接[l, r)范围内的word单词
    private StringBuilder joinlr(String[] words, int l, int r, String link) {
        StringBuilder sb = new StringBuilder(words[l]);
        for(int i = l + 1; i < r; i++) {
            sb.append(link);
            sb.append(words[i]);
        }
        return sb;
    }
}
```

#### 71. 简化路径 | medium

##### 1）模拟法 | O（3 * n）

- **思路**：见代码注释。
- **结论**：时间，3 ms，91.47%，空间，41.6 mb，44.22%，时间上，分割 "/" 花费 O（n），遍历目录花费 O（n），出栈拼接返回值花费 O（n），所以整体时间复杂度为 O（3 * n），空间上，分割后的数组花费 O（n），stack 花费 O（n），所以额外空间复杂度为 O（2 * n）。

```java
class Solution {
    public String simplifyPath(String path) {
        // 栈(这里用双端队列, 来模拟栈方便一些)
        Deque<String> stack = new LinkedList<>();

        // 按"/"分割字符串, 然后入队
        for(String dir : path.split("/")) {
            // 如果为连续"/", 会导致空串的出现, 为".", 则跳过, 继续判断下一个目录
            if("".equals(dir) || ".".equals(dir)) {
                continue;
            } 
            // 如果为"..", 则出栈, 代表回到上一层目录
            else if("..".equals(dir)) {
                if(!stack.isEmpty()) {
                    stack.pollLast();
                }
            }
            // 如果为其他目录, 则入栈
            else {
                stack.offerLast(dir);
            }
        }
        
        // 最后, 再把栈逆序倒出即可
        StringBuilder sb = new StringBuilder("/");
        while(!stack.isEmpty()) {
            sb.append(stack.pollFirst());
            if(!stack.isEmpty()) {
                sb.append("/");
            }
        }

        return sb.toString();
    }
}
```

#### 415. 字符串相加 | easy

##### 1）竖式加法 + 暴力模拟 | O（m + 3 * n）

- **思路**：模拟加法过程，低位累加余数，高位累加进位，最后反转过来即可。
- **结论**：时间，1 ms，100%，空间，41.3 mb，69.01%，时间上，反转 shorts 花费 O（m），反转 longs 花费 O（n），模拟累加花费 O（n），最后反转结果花费 O（n），所以整体时间复杂度为 O（m + 3 * n），空间上，由于使用了一张 m 长的 shorts 数组，n 长的 longs 数组， n 长的 StringBuilder，所以额外空间复杂度为 O（m + 2 * n）。

```java
class Solution {
    public String addStrings(String num1, String num2) {
        char[] chars1 = num1.toCharArray();
        char[] chars2 = num2.toCharArray();
        int m = num1.length(), n = num2.length();

        // 较短者shorts, 较长者longs
        char[] shorts = m > n? chars2 : chars1;
        char[] longs = shorts == chars1? chars2 : chars1;
        int slen = shorts.length, glen = longs.length;

        // 先反转
        reverse(shorts);
        reverse(longs);

        // 模拟从低位加到高位: 456 + 77
        int x, y, sum, carry = 0, remaind;
        StringBuilder sb = new StringBuilder();
        for(int i = 0; i < glen; i++) {
            x = i < slen? shorts[i] - '0' : 0;
            y = longs[i] - '0';
            sum = x + y + carry;
            carry = sum / 10;
            remaind = sum % 10;
            sb.append(remaind);
        }

        // 如果最后还有进位, 那么还需要累加
        if(carry > 0) {
            sb.append(carry);
        }

        return reverse(sb.toString());
    }

    private String reverse(String s) {
        char[] chars = s.toCharArray();
        reverse(chars);
        return new String(chars);
    }

    private void reverse(char[] chars) {
        int l = 0, r = chars.length - 1;
        while(l < r) {
            swap(chars, l++, r--);
        }
    }

    private void swap(char[] chars, int from, int to) {
        char tmp = chars[to];
        chars[to] = chars[from];
        chars[from] = tmp;
    }
}
```

##### 2）竖式加法 + 双指针 + 模拟 | O（2 * n）

- **思路**：
  1. 与暴力模拟的思路一样，都是从低位到高位，不断模拟累加余数、求进位。
  2. 但这里不同的是，采用了双指针的方式去模拟，遇到超出长度的，则往前补 0 即可，去除了暴力模拟那种要先确定短者的麻烦。
  3. 最后，也利用 StringBuilder 才有的 reverse 方法，避免了手写反转函数。
- **结论**：时间，1 ms，100%，空间，41.5 mb，39.19%，时间上，双指针模拟花费 O（n），最后的结果反转花费 O（n），n 位 i、j 中的大者，所以整体时间复杂度为 O（2 * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java 
class Solution {
    public String addStrings(String num1, String num2) {
        char[] chars1 = num1.toCharArray();
        char[] chars2 = num2.toCharArray();
        int i = chars1.length - 1, j = chars2.length - 1;

        // 开始模拟累加: 至少存在i、j或者进位, 才进行累加
        int carry = 0, x, y, sum;
        StringBuilder sb = new StringBuilder();
        while(i >= 0 || j >= 0 || carry > 0) {
            x = i >= 0? chars1[i] - '0' : 0;
            y = j >= 0? chars2[j] - '0' : 0;
            sum = x + y + carry;
            sb.append(sum % 10);
            
            carry = sum / 10;
            i--;
            j--;
        }

        // 反转结果返回
        sb.reverse();
        return sb.toString();
    }
}
```

#### 504. 七进制数 | easy

##### 1）模拟法 | O（ log(7) [n] ）

- **思路**：
  1. 本题的思路关键在于，自己给出一个 test case，比如 "101"，然后根据 101 做数学运算，不断推演出 `num = ai * 7^bi + ai-1 * 7^bi-1 + ... + a0 * 7 ^b0` 中的 ai 和 bi 来，然后把 ai 加入返回结果中。
  2. 这里推演用到的方式是，求商和取模，通过观察，101 = 2 * 7^2 + 0 * 7^0 + 3 * 7^0 = 98 + 3 = 203，所以为了得到 2、0、3，可以用 101 % 7 得到最低位的 3，再用 101 / 7 % 7 得到次低位的 0，再用 101 / 7 / 7 % 7 得到最高位的 2，组合起来就是 "203" 了。其中，还要注意 num 为 0 以及为负数的情况。
  3. 然后额外做一下解码的方法，解码则是通过从最低位开始，不断根据编码串中的数字，去编码表 BASE_ARR 中，找到对应的数字 num ，然后利用 res += Math.power(7, digit++) * num 公式，累加得到最终的结果。其中，要注意 int 数字溢出的问题（有符号时最大表示 21 亿，无符号时最大表示 42亿），这里用 long 解决就好（有符号时最大表示 900 亿亿，无符号时最大表示 1800 亿亿）。
- **结论**：时间，1 ms，77.36%，空间，38.9 mb，49.88%，时间上，无论是编码还是解码，取决于它在 7 进制下，有多少位，所以时间复杂度为 O（ log[7] n），空间上，由于只是用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {

    private static final char[] BASE_ARR = "0123456".toCharArray();

    // 编码
    public String convertToBase7(int num) {
        if(num == 0) {
            return "0";
        }

        boolean isNegative = false;
        if(num < 0) {
            isNegative = true;
            num = -num;
        }

        StringBuilder sb = new StringBuilder();
        while(num > 0) {
            // 余数, 用作当前进制的数字
            sb.append(BASE_ARR[num % 7]);

            // 商, 则继续交给高位相除
            num /= 7;
        }

        String res = sb.reverse().toString();
        return isNegative? "-" + res : res;
    }

    // 解码
    private static String decode(String s) {
        char[] chars = s.toCharArray();
        boolean isNegative = false;
        if(chars[0] == '-') {
            isNegative = true;
        }

        int num, digit = 0, base = BASE_ARR.length;
        long res = 0;

        // 62进制时："0123456789ABCDEFGHIJKLMNOPQRSTUVWSYZabcdefghijklmnopqrstuvwxyz";
        for (int i = chars.length - 1; i > (isNegative? 0 : -1); i--) {
            if(chars[i] >= '0' && chars[i] <= '9') {
                num = chars[i] - '0';
            } else if(chars[i] >= 'A' && chars[i] <= 'Z') {
                num = chars[i] - 'A' + 10;
            } else {
                // chars[i] >= 'a' && chars[i] <= 'z'
                num = chars[i] - 'a' + 36;
            }

            res += num * Math.pow(base, digit++);
        }

        String numStr = String.valueOf(res);
        return isNegative? "-" + numStr : numStr;
    }
}
```

#### 2288. 第 295 周赛 - 价格减免 | medium

##### 1）模拟法 | O（2 * n）

- **思路**：
  1. 模拟算法执行流程，遍历字符串，拿到 '$' 开头的单词，做 long 类型转换，转换成功的就格式化为 2位小数结尾，异常的则跳过，最后再拼回去即可。
  2. 其中要注意的是， '$' 开头的价格单词可能很大，需要用 long 类型去接，以及2 位小数的格式化问题，可以用 s.format("%.2f") 来进行，用 BigDecimal 来做四舍五入是不行的！
- **结论**：时间，1070 ms，10.02%，空间，49.4 mb，74.41%，时间上，遍历 s 数组花费 O（n），重新组装 s 字符串花费 O（n），所以时间复杂度位 O（2 * n），空间上，由于只使用了额外的几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public String discountPrices(String sentence, int discount) {
        if(sentence == null || sentence.length() == 0) {
            return null;
        }

        String[] strs = sentence.split(" ");
        StringBuilder sb = new StringBuilder();
        for(int i = 0; i < strs.length; i++) {
            if(isPrice(strs[i])) {
                sb.append(transformPrice(strs[i], discount)).append(" ");
            } else {
                sb.append(strs[i]).append(" ");
            }
        }
        
        return sb.toString().trim();
    }
    
    private boolean isPrice(String s) {
        if(s == null || !s.startsWith("$") || s.length() <= 1) {
            return false;
        }

        char[] chars = s.toCharArray();
        for(int i = 1; i < chars.length; i++) {
            if(!Character.isDigit(chars[i])) {
                return false;
            }
        }

        return true;
    }

    private String transformPrice(String s, int discount) {
        // 判断价格
        try {
            long price = Long.parseLong(s.substring(1, s.length()));
            return "$" + String.format("%.2f", discount((double) price, discount));
        } catch(Exception e) {  
            return "";
        }
    }

    private double discount(double f, int discount) {
        return f - f * discount / 100;
    }
}
```

#### 2299. 第 80 双周赛 - 强密码检验器 II

##### 1）模拟法 | O（n）

- **思路**：遍历字符串，看是否没有出现连续两个字符、小写字母、大写字母、数字、特殊字符，如果都符合才返回 true。
- **结论**：时间，1 ms，69.46%，空间，39.5 mb，58.79%，时间上，由于只需要遍历一次字符串，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public boolean strongPasswordCheckerII(String s) {
        if(s == null || s.length() < 8) {
            return false;
        }
        
        char[] chars = s.toCharArray();
        Set<Character> specialCharSet = getSpecialCharSet();
        
        boolean hasLowerCase = false, hasUpperCase = false, hasDigit = false, hasSpecial = false;
        for(int i = 0; i < chars.length; i++) {
            // 不包含2个连续相同的字符
            if(i + 1 < chars.length && chars[i + 1] == chars[i]) {
                return false;
            }
            if(!hasLowerCase && chars[i] >= 'a' && chars[i] <= 'z') {
                hasLowerCase = true;
            }
            if(!hasUpperCase && chars[i] >= 'A' && chars[i] <= 'Z') {
                hasUpperCase = true;
            }
            if(!hasDigit && chars[i] >= '0' && chars[i] <= '9') {
                hasDigit = true;
            }
            // !@#$%^&*()-+
            if(!hasSpecial && specialCharSet.contains(chars[i])) {
                hasSpecial = true;
            }
        }
        
        return hasLowerCase && hasUpperCase && hasDigit && hasSpecial;
    }
    
    private Set<Character> getSpecialCharSet() {
        Set<Character> set = new HashSet<>();
        for(char c : "!@#$%^&*()-+".toCharArray()) {
            set.add(c);
        }
        return set;
    }
}
```

#### 2301. 第 80 双周赛 - 替换字符后匹配 | hard

##### 1）暴力枚举 | O（k + n * m）

- **思路**：
  1. 由于 "sub 中每个字符不能被替换超过一次"，所以枚举到 sub[bidx] 时发现与 s[j] 不匹配时，则只需要根据 sub[bidx] 的字符去哈希表缓存中，查看是否存在 s[j] 的替换，如果存在则认为匹配，否则认为不匹配。
  2. 如果当前 s 子串遍历完，都没发现不匹配，则可以直接返回 true，否则需要继续尝试后面的子串尝试，当所有的 s 子串都枚举完后，如果发现依然没匹配，则返回 false。
  3. 本题测试用例中，最长也才 5000 个元素，所以可以走暴力解，如果为 10^5 时，暴力解会超时，此时最优解应该时 FFT 快速傅里叶变换，时间复杂度为 O（n * logn）。
- **结论**：时间，248 ms，81.60%，空间，41.8 mb，78.79%，时间上，组装哈希表缓存花费 O（k），枚举 s 子串开头花费 O（n），枚举比较 s 子串的每个字符花费 O（m），所以时间复杂度为 O（k + n * m），空间上，由于使用了一张 k 长的哈希表，所以额外空间复杂度为 O（k）。

```java
class Solution {
    public boolean matchReplacement(String s, String sub, char[][] mappings) {
        if(s == null || sub == null || mappings == null) {
            throw new RuntimeException("参数异常!");
        }

        // 初始化好替代品哈希表
        Map<Character, Set<Character>> csetMap = new HashMap<>();
        for(int i = 0; i < mappings.length; i++) {
            char subchar = mappings[i][0];
            char tchar = mappings[i][1];

            Set<Character> set = csetMap.getOrDefault(subchar, new HashSet<>());
            set.add(tchar);
            csetMap.put(subchar, set);
        }

        char[] schars = s.toCharArray();
        char[] bchars = sub.toCharArray();
        int sn = schars.length, bn = bchars.length, sm, bidx, isSuccess;
        for(int i = 0; i < sn; i++) {
            // 如果加上sub的长度, 发现i超长了, 且还没匹配出来, 那么认为匹配不了, 返回false即可
            sm = i + bn - 1;
            if(sm >= sn) {
                return false;
            }

            // 否则, 说明可能匹配, 则从j开始枚举
            isSuccess = 1;
            for(int j = i; j <= sm; j++) {
                bidx = j - i + 0;

                // 如果s的j位置和sub的j-i+0位置不匹配, 那么开始尝试替换成替代品
                if(schars[j] != bchars[bidx]) {
                    // 如果没有bidx的替代品, 或者有但却不能替换成sidx的
                    Set<Character> set = csetMap.get(bchars[bidx]);
                    if(set == null || !set.contains(schars[j])) {
                        // 则认为当前i开头的s子串不用尝试了, 已经匹配不出来了
                        isSuccess = 0;
                        break;
                    }
                }
            }

            // 如果当前i开头的s子串遍历完毕, 都能成功, 说明匹配成功, 返回true即可
            // 如果没有匹配成功, 则继续尝试下一个i+1开头的s子串
            if(isSuccess == 1) {
                return true;
            }
        }

        // 如果都能来到这里, 说明肯定没匹配成功, 则返回false即可
        return false;
    }
}
```

#### 2303. 第 297 周赛 - 计算应缴税款总额 | easy

##### 1）模拟法 | O（n）

- **思路**：直接模拟，分情况讨论见程序注释。
- **结论**：时间，0 ms，100%，空间，41.3 mb，53.82%，时间上，由于只需要遍历一次税点数组，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public double calculateTax(int[][] b, int ic) {
        if(b == null || b.length == 0 || b[0] == null || b[0].length == 0) {
            return 0d;
        }
        
        double res = 0d;
        int m = b.length, n = b[0].length;

        // 遍历每个税点并与收入进行比较、计算
        for(int i = 0; i < m; i++) {
            // 如果收入比当前征税点高
            if(ic >= b[i][0]) {
                // 如果存在前一个征税点, 那么就征收当前税点-上一个税点的部分
                if(i - 1 > - 1) {
                    res += (double) (b[i][0] - b[i - 1][0]) * b[i][1] / 100;
                } 
                // 如果不存在前一个税点, 那么久征收满额的当前税点
                else {
                    res += (double) b[i][0] * b[i][1] / 100;
                }
            } 
            // 如果收入比当前征税点低
            else {
                // 如果存在前一个征税点, 那么就征收当前收入-上一个税点的部分
                if(i - 1 > - 1) {
                    res += (double) Math.max((ic - b[i - 1][0]), 0) * b[i][1] / 100;
                } 
                // 如果不存在前一个税点, 那么久征收当前收入满额的税收
                else {
                    res += (double) ic * b[i][1] / 100;
                }
            }
        }
        
        return res;
    }
}
```

#### 2310. 第 298 周赛 - 个位数字为 k 的整数之和 | medium

##### 1）枚举 + 数学推导 | O（n）

- **思路**：
  1. 根据题意，要求 i 满足 10 * m + i * k = num，而由于本题上限很低，可以通过该枚举到和为 num，来遍历判断每一个 i 是否符合 10 的倍数，返回最小的那个 i 即可。
  2. 其中，由于不满足有序性，即确定一个 mid 后，不知道能不能丢弃另一半，所以不能用二分查找。
- **结论**：时间，0 ms，100%，空间，38.6 mb，60.40%，时间上，由于最多需要遍历到 num，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int minimumNumbers(int num, int k) {
        if(num == 0) {
            return 0;
        }
        if(k == 0) {
            return num % 10 == 0? 1 : -1;
        }

        int tmp;
        for(int i = 1; (tmp = num - i * k) >= 0; i++) {
            if(tmp % 10 == 0) {
                return i;
            }
        }

        return -1;
    }
}
```

##### 2）枚举 + 数学推导 + 取模性质 | O（10）

- **思路**：在 1 的基础上，根据取模性质， `(a + b) % p = ( (a % p) + (b % p) ) % p、(a * b) % p = ( (a % p) * (b % p) ) % p` 可发现，(num - i * k) % 10 = (num % 10 - [ { (i % 10) * (k % 10) } % 10 ] ) % 10，即当 n 枚举大于 10 时，i % 10 则与已经出现过枚举 [0, 10] 的情况里了，所以这里最多只需要遍历到 10 即可返回了。
- **结论**：时间，0 ms，100%，空间，38.3 mb，95.60%，时间上，由于最多遍历 10 次，所以时间复杂度为 O（10），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int minimumNumbers(int num, int k) {
        if(num == 0) {
            return 0;
        }
        if(k == 0) {
            return num % 10 == 0? 1 : -1;
        }
    
        int tmp;
        for(int i = 1; i <= 10 && (tmp = num - i * k) >= 0; i++) {
            if(tmp % 10 == 0) {
                return i;
            }
        }

        return -1;
    }
}
```

#### 2315. 第 81 双周赛 - 统计星号 | easy

##### 1）字符串模拟法 | O（n + n / 2）

- **思路**：
  1. 以 "|" 作为分割符，分割字符串，然后偶数位置的字符串，统计 "*" 的数量即可。
  2. 其中，"|" 分割时，由于传入给 `split` 函数的是一个正则表达式，所以如果直接传 "|"，则会被认为是 "A || B" 的符号，被当做了无效表达式，即空串 "" 来处理，而之所以要传 `\\|` ，是因为第一个 `\`  表示的是 Java 的转义字符，第二个 `\` 表示的是 `\|` 的转义。
- **结论**：时间，1 ms，99.82%，空间，39.7 mb，33.48%，时间上，正则表达式匹配花费 O（n），遍历一半的字符串花费 O（n / 2），所以整体的时间复杂度为 O（n + n / 2），空间上，由于使用了一个额外的 sarr 数组，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public int countAsterisks(String s) {
        String[] sarr = s.split("\\|");
        
        int cnt = 0;
        for(int i = 0; i < sarr.length; i = i + 2) {
            char[] chars = sarr[i].toCharArray();
            for(int j = 0; j < chars.length; j++) {
                if(chars[j] == '*') {
                    cnt++;
                }
            }
        }
        
        return cnt;
    }
}
```

#### 2319. 第 299 场周赛 - 判断矩阵是否是一个 X 矩阵 | easy

##### 1）模拟法 | O（2 * n）

- **思路**：正方形矩阵中，负斜率对角线的元素下标 i、j 满足，i == j，正斜率对角线的元素下表 i、j 满足，i + j == n - 1，然后判断所有对角线的元素是否都为 0 即可。
- **结论**：时间，1 ms，99.91%，空间，42.4 mb，30.94%，时间上，由于最多只会有 2 * n 个对角线的元素被访问到，所以时间复杂度为 O（2 * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public boolean checkXMatrix(int[][] grid) {
        int n = grid.length;
        
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < n; j++) {
                if(i == j || i + j == n - 1) {
                    if(grid[i][j] == 0) {
                        return false;
                    }
                } else {
                    if(grid[i][j] != 0) {
                        return false;
                    }
                }
            }
        }
        
        return true;
    }
}
```

### 2.3. 算法思想 - 位运算

#### 136. 只出现一次的数字 | easy

##### 1）异或运算法 | O（n）

- **思路**：利用  a ^ b，相同则等于 0，以及 0 ^ a = a 的特点，只需要遍历一遍即可找出答案。
- **结论**：时间，1ms，100%，空间，38.3mb，89.63%，效率非常好，就这样吧~

```java
class Solution {
    public int singleNumber(int[] nums) {
        if(nums == null || nums.length == 0) {
            throw new IllegalArgumentException("参数不合法!");
        }
        
        int index = 1, res = nums[0];
        while(index < nums.length) {
            res ^= nums[index++];
        }

        return res;
    }
}
```

#### 338. 比特位计数 | easy

##### 1）内置函数法 | O（n）

- **思路**：从头遍历数组，然后分别调用 Integer.bitCount 方法获取为 1 的比特位即可。
- **结论**：时间，1ms，99.92%，空间，45.3mb，5.29%，时间上，由于只需要遍历一次数组，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int[] countBits(int n) {
        int[] res = new int[n+1];
        for(int i = 0; i <= n; i++) {
            res[i] = Integer.bitCount(i);
        }
        return res;
    }
}
```

##### 2）右位移法 | O（n * logn）

- **思路**：从头遍历每个 i，然后再遍历它的每一位比特位，判断是否为 1，分别统计它们的比特位为 1 的计数，记录到返回结果中再返回即可。
- **结论**：时间，5ms，15.94%，空间，45.2mb，5.78%，时间上，由于遍历一次数组需要花费 O（n），对于最大值为 n 的比特位需要遍历 log（n）位（因为 2^k = n，k = log(n)），所以时间复杂度为 O（n * logn），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int[] countBits(int n) {
        int[] res = new int[n+1];
        
        int count, tmp;
        for(int i = 0; i <= n; i++) {
            count = 0;
            tmp = i;
            do {
                if((tmp & 1) == 1) {
                    count++;
                }
            } while((tmp >>>= 1) > 0);
            res[i] = count;
        }

        return res;
    }
}
```

##### 3）Brian Kernighan 算法 | O（n * logn）

- **思路**：Brian Kernighan （布莱恩·克尼汉）算法给出，对于任意整数 x，令 x = x &（x-1），可将 x 的二进制表示的最后一个 1 变为 0，因此，如果不断循环执行这个过程则可以获得 x 为 1的比特位个数。
- **结论**：时间，1ms，99.92%，空间，45.1mb，6.45%，时间上，由于遍历一次数组需要花费 O（n），对于最大值为 n 的比特位最多遍历 log（n）位（因为 2^k = n，k = log(n)），所以时间复杂度为 O（n * logn），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int[] countBits(int n) {
        int[] res = new int[n+1];

        int count, tmp;
        for(int i = 0; i <= n; i++) {
            count = 0;
            tmp = i;
            while(tmp > 0) {
                tmp &= tmp - 1;
                count++;
            }
            res[i] = count;
        }

        return res;
    }
}
```

##### 4）动态规划 - 最高位 1  | O（n）

- **思路**：利用最高位+杂项来组合成当前的 i，最高位有一个为 1的比特位，而杂项又可以通过数组中以前算过的结果来获取，所以只需要遍历一次即可。
- **结论**：时间，1ms，99.92%，空间，45.3mb，5.64%，时间上，由于只需要遍历 1 次数组，所以时间复杂度为 O（n），空间上，由于只使用了几个有限变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int[] countBits(int n) {
        int[] res = new int[n+1];
        
        int prehval = 0;
        for(int i = 1; i <= n; i++) {
            // i为2的整数次幂, 则设置上一个高位值为i
            if((i & (i-1)) == 0) {
                prehval = i;
                res[i] = 1;
            } 
            // i不是2的整数次幂, 说明除了高位之后, 低位还有一些杂项, 可在以前算过的结果里取
            else {
                res[i] = res[i - prehval] + 1;
            }
        }

        return res;
    }
}
```

##### 5）动态规划 - 最低位 1 | O（n）

- **思路**：同理，这种解法是利用了最低位 1 的特性，通过判断 i 是否为奇偶数，最低位是否 1，如果为偶数，则结果与右移结果一致，如果为奇数，则认为比上一个偶数多了一个低位的 1 而已。
- **结论**：时间，1ms，99.92%，空间，45.6mb，5.02%，时间上，由于只需要遍历 1 次数组，所以时间复杂度为 O（n），空间上，由于只使用了几个有限变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int[] countBits(int n) {
        int[] res = new int[n+1];
        for(int i = 1; i <= n; i++) {
            // i为偶数, 说明0位上的bit为0, 此时结果与右移1位的结果一致
            if(i % 2 == 0) {
                res[i] = res[i >>> 1];
            } 
            // i为奇数, 说明0位上的bit为1, 比上一个值多了1个bit
            else {
                res[i] = res[i-1] + 1;
            }
        }

        return res;
    }
}
```

##### 6）动态规划 - 最右位 1 | O（n）

- **思路**：解本题的关键就在于，如果利用好前面 1 的结果，来生成本次 1 的结果，同理，本题利用上次最右 1 的结果，来生成本次 1 的结果，由 i & (i-1) 可把最右的 1 变为 0，也就是少了一个 1，所以结果取上次的结果加回来就好。
- **结论**：时间，1ms，99.92%，空间，45.4mb吗，5.02%，时间上，由于只需要遍历 1 次数组，所以时间复杂度为 O（n），空间上，由于只使用了几个有限变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int[] countBits(int n) {
        int[] res = new int[n+1];
        
        for(int i = 1; i <= n; i++) {
            res[i] = res[i & (i-1)] + 1;
        }

        return res;
    }
}
```

#### 461. 汉明距离 | easy

##### 1）内置函数法 | O（1）

- **思路**：通过异或操作，得到不同 1 的位置后，再利用 《比特位计数 - 内置函数》的思路解决即可。
- **结论**：时间，0ms，100%，空间，38.1mb，7.67%，时间上，由于内置函数也是位移操作，所以时间复杂度为 O（1），空间上，由于只使用有限几个变量，所以空间复杂度也是 O（1）。

```java
class Solution {
    public int hammingDistance(int x, int y) {
        return Integer.bitCount(x ^ y);
    }
}
```

##### 2）右位移法 | O（logn）

- **思路**：通过异或操作，得到不同 1 的位置后，再利用 《比特位计数 - 右位移法》的思路解决即可。
- **结论**：时间，0ms，100%，空间，38.6mb，5.04%，时间上，由于最多遍历 logn 位，所以时间复杂度为 O（logn），空间上，由于只使用有限几个变量，所以空间复杂度是 O（1）。

```java
class Solution {
    public int hammingDistance(int x, int y) {
        int res = (x ^ y), count = 0;
        do {
            if((res & 1) == 1) {
                count++;
            }
        } while((res >>>= 1) > 0);
        return count;
    }
}
```

##### 3）Brian Kernighan  算法 | O（logn）

- **思路**：通过异或操作，得到不同 1 的位置后，再利用 《比特位计数 - Brian Kernighan  算法 》的思路解决即可。
- **结论**：时间，0ms，100%，38.5mb，5.04%，时间上，由于最多遍历 logn 位，所以时间复杂度为 O（logn），空间上，由于只使用有限几个变量，所以空间复杂度是 O（1）。

```java
class Solution {
    public int hammingDistance(int x, int y) {
        int res = (x ^ y), count = 0;
        while(res > 0) {
            res &= res - 1;
            count++;
        }
        return count;
    }
}
```

#### 89. 格雷编码 | meidum

##### 1）递归生成码表 + 对称生成 | O（2^n - 1）

- **思路**：
  1. 格雷码，属于可靠性编码，由于相邻两个码组之间只有一位不同，不仅减少了数字电路中，由一个状态转换到下一个状态的逻辑混淆，还避免了转换时产生很大的尖峰电流脉冲的情况。
  2. 其中，格雷码的转换方法有，递归生成码表、二进制异或转换、利用卡诺图等方式。
  3. 而这里的算法思路，则是基于【递归生场码表】的方式进行设计的，即：
     - 1）1 位格雷码有2个码字。
     - 2）(n+1)位格雷码中的前2n个码字, 等于n位格雷码的码字, 按顺序书写, 加前缀0。
     - 3）(n+1)位格雷码中的后2n个码字, 等于n位格雷码的码字, 按逆序书写, 加前缀1。
     - 4）n+1位格雷码的集合 = n位格雷码集合(顺序)加前缀0 + n位格雷码集合(逆序)加前缀1。
- **结论**：时间，6 ms，69.77%，空间，47.4 mb，20.75%，时间上，由于一共需要组装 n 次，每次组装分别需要累加 2、4、8、...、2^（n-1），所以整体时间复杂度为 O（2 + 4 + 8 + ... + 2^[n - 1] ）= O（2^n - 1），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public List<Integer> grayCode(int n) {
        // 1位格雷码有2个码字
        List<Integer> res = new ArrayList<>();
        res.add(0);// 00
        res.add(1);// 01

        // 2, 3, 4, ..., n位格雷码
        for(int i = 2; i <= n; i++) {
            // (n+1)位格雷码中的前2n个码字, 等于n位格雷码的码字, 按顺序书写, 加前缀0
            // (n+1)位格雷码中的后2n个码字, 等于n位格雷码的码字, 按逆序书写, 加前缀1
            // n+1位格雷码的集合 = n位格雷码集合(顺序)加前缀0 + n位格雷码集合(逆序)加前缀1
            int size = res.size(), factor = 1 << (i - 1);
            for(int j = size - 1; j > -1; j--) {
                res.add(res.get(j) + factor);
            }
        }

        return res;
    }
}
```

##### 2）二进制异或转换 | O（2^n）

- **思路**：这里的算法思路，则是基于【二进制异或转换】的方式进行设计的：

  1. 根据公式有：

     ![1658906215353](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1658906215353.png)

  2. 因此，用算法实现则是，先把 i 右移一位（高位补 0），再把右移后的结果，异或上 i，即模拟实现了 b（i + 1）异或 b（i）。

- **结论**：时间，3 ms，99.01%，空间，47.5 mb，20.29%，时间上，由于 res 的长度为 2^n，所以基于二进制序列异或转换的时间复杂度为 O（2^n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public List<Integer> grayCode(int n) {
        int len = 1 << n;
        List<Integer> res = new ArrayList<>(len);
        for(int i = 0; i < len; i++) {
            res.add((i >>> 1) ^ i);
        }
        return res;
    }
}
```

#### 2311. 第 298 周赛 - 小于等于 K 的最长二进制子序列 | medium

##### 1）贪心 + 枚举 + 位运算 | O（n + 29）

- **思路**：
  1. 由于求的是最长的二进制子序列，且允许有前导 0，所以贪心的点就在于，从后面开始找，且往前拼凑尽量多的 0 到子序列中。
  2. 这样，就需要对前导 0 进行缓存起来，cnts[i] 表示 i 前有多少个 0，方便后面找到最前 1 后能够以 O（1）的复杂度，取到前导 0 有多少个。
  3. 然后，还需要判断究竟是保留后两位，还是后一位，如果后两位的 int 值比 k 大，那么说明只能保留一位，否则可以保留两位。
  4. 如果只能保留一位，那么就不需要往前找最前 1 了，因为最前 1 就是最后 1 位，所以此时最终结果为 cnts[n - 1] + 1。
  5. 如果可以保留两位，那么还需要往前找最前 1，且由于 k 是无符号正的 int 值，所以最多往前找 31 长度或者 n 长即可，否则就溢出了，找到以后最终结果则是 len + cnts[n - len] 。
- **结论**：时间，1 ms，99.85%，空间，40 mb，58.64%，时间上，初始化 cnts 数组花费 O（n），遍历往前找最前 1 最多花费 O（31 - 2）= O（29），所以整体时间复杂度为 O（n + 29），空间上，由于使用了一张 n 长的 cnts 数组，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public int longestSubsequence(String s, int k) {
        char[] chars = s.toCharArray();
        int n = chars.length;
        if(n == 1) {
            return chars[0] - '0' <= k? 1 : 0;
        }

        // cnts表示i位前有多少个0
        int[] cnts = new int[n];
        for(int i = 1; i < n; i++) {
            cnts[i] = chars[i - 1] == '0'? 1 + cnts[i - 1] : cnts[i - 1];
        }

        // 获取后两位的int值
        int val = getBinaryIntValue(chars[n - 2], chars[n - 1]);

        // k < 2, 说明只有最后一位bit被选中
        if(val > k) {
            return cnts[n - 1] + 1;
        }

        // k >= 2, 说明后两位bit必定被选中, 且是00、01、10、11其中一种
        int len = 2;

        // 由于k位int值, 且是无符号int值, 所以最多只有31位, 所以这里只需要枚举到31或者n即可
        while(len + 1 < 32 && len + 1 <= n) {
            // n-len-1可以得到前一个1的字符索引
            val += (chars[n - len - 1] - '0') * (1 << len);
            if(val > k) {
                break;
            }

            len++;
        }
        
        // 因此, 结果长度 = 最前的、符合条件的1到结尾的长度 + 之前0的个数
        return len + cnts[n - len];
    }

    // 获取后两位的int值
    private int getBinaryIntValue(char lastLastc, char lastc) {
        int lastLast = lastLastc - '0';
        int last = lastc - '0';
        return lastLast * 2 + last; 
    }
}
```

##### 2）贪心 + k 长对比 + 系统函数 | O（2 * n）

- **思路**：
  1. 贪心的思路同 1 方法，不过这里与 1 不同的是，求最前 1 的思路不同。
  2. 这里是把整个有效 k 都放到 s 中去比较，如果 k 长的 s 二进制数值，小于等于 k 小的话，说明这个长度就是最前 1 的长度了，否则最前 1 的长度为 m - 1，因为 m 长 s 二进制不符合小于等于 k，那么 m - 1长就必定符合。
  3. 找到最前 1 的长度后，只需要累加前导 0 的个数，即得到最终最长子序列长度结果了。
  4. 其中，32 - Integer.numberOfLeadingZeros(k) 系统函数，可以求出有效 k 的二进制长度是多少，Integer.parseInt(sub, 2)，可以求出 sub 二进制字符串在十进制下的 int 值是多少，切记切记~
- **结论**：时间，4 ms，26.99%，空间，39.6 mb，88.99%，时间上，由于需要截取两次 s 字符串，所以时间复杂度最多为 O（2 * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int longestSubsequence(String s, int k) {
        // 如果s的长度, 小于k的有效二进制长度k#size, 说明s肯定比k小, 那么整个s都算作结果
        int n = s.length(), m = 32 - Integer.numberOfLeadingZeros(k);
        if(n < m) {
            return n;
        }

        // 否则, 说明s比k长, 那么s就需要截取
        // 这里是先假设k长的s是否小于k, 如果小于那就是最前1的长度了, 否则说明最前1的长度应该比k少一位
        int len = Integer.parseInt(s.substring(n - m), 2) <= k? m : m - 1;
        
        // 最后结果则等于, 最前1的长度 + 往前找到的0的个数
        return len + (int) s.substring(0, n - len).chars().filter(f -> f == '0').count();
    }
}
```

#### 2317. 第 81 双周赛 - 操作后的最大异或和 | medium

##### 1）脑筋直转弯 + 位运算 | O（n）

- **思路**：
  1. 需要从位运算操作本身的意义来考虑，先操作 `nums[i] ^ x` 相当于把 nums[i] 变为任意的数字。
  2. 然后，操作 `nums[i] & (nums[i] ^ x)` 相当于让 nums[i] 与上任意的数字，但由于 nums[i] 二进制 1 的个数有限，所以无论与谁，最大的结果也只能是 nums[i]。
  3. 最后，由于操作 `nums[i] & (nums[i] ^ x)`  可以对数字中的任意元素、执行任意多次，且最终要求得是最大值，所以需要保证数组中，出现过 1 的位置不发生丢失才行，因此，求最终结果只需要把每位 1 或起来即可。
- **结论**：时间，1 ms，100%，空间，49.4 mb，90.99%，时间上，由于只需要遍历一次数组，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java 
class Solution {
    public int maximumXOR(int[] nums) {
        int ans = 0;
        for(int num : nums) {
            ans |= num;
        }
        return ans;
    }
}
```

### 2.4. 算法思想 - 排序

#### 49. 字母异位词分组 | medium

##### 1）哈希表 + 排序 | O（n * nlogn）

- **思路**：遍历每个字符串，并获取他们的字符数组，然后顺序排序字符数组，再组合成顺序 Key，接着加入哈希表中，这样，含有相同字符的字符串都会进入哈希表同一个桶中，完成分组。
- **结论**：5ms，99.01%，41.3mb，56.10%，垃圾题~

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        if(strs == null) {
            return new ArrayList<>();
        }

        Map<String, List<String>> map = new HashMap<>();
        for(String s : strs) {
            char[] chars = s.toCharArray();
            Arrays.sort(chars);
            String key = String.valueOf(chars);
            if(map.containsKey(key)) {
                map.get(key).add(s);
            } else {
                List<String> list = new ArrayList<>();
                list.add(s);
                map.put(key, list);
            }
        }

        List<List<String>> res = new ArrayList<>(map.size());
        for(List<String> list : map.values()) {
            res.add(list);
        }

        return res;
    }
}
```

#### 75. 颜色分类 | medium

##### 1）荷兰国旗 partition 法 | O（n） 

- **思路**：这是经典的荷兰国旗问题，可以设计一个通用的 partition （midValue，start，end）函数，通过不断与 midValue 比较，把 [start,end] 范围内的数组, 划分成 [左边<,中间=,右边>] 的三个区域。
  1. 初始化一个左边界 -1，和一个右边界 num.length+1，开始从左到右遍历。
  2. 如果 nums[i] 小于 midValue，那么需要把它加入左边界里，即**左边界 L++**，然后交换 i 位置的值，接着 i++，继续比较下一个。
  3. 如果 nums[i] 等于 midValue，则认为当前位置是正确的，因为在左边界的右边。
  4. 如果 nums[i] 大于 midValue，那么需要把它加入右边界里，即**右边界 R--**，然后交换 i 位置的值，但由于新交换上来的值还没有比较，不知道是留在 i 位置还是左边界里，所以还需要继续比较，此时 i **保持不变**，开始下一轮比较。
- **结论**：时间，0ms，100%，空间，36.9mb，53.27%，效率非常高，就这样吧~

```java
class Solution {
    public void sortColors(int[] nums) {
        if(nums == null || nums.length <= 1) {
            return;
        }
        partition(nums, 1, 0, nums.length-1);
    }

    // partition荷兰国旗问题, 可以把[start,end]范围内的数组, 划分成[左边<,中间=,右边>]的三个区域
    private void partition(int[] nums, int midValue, int start, int end) {
        if(start > end) {
            return;
        }

        // < midValue 的左边界, > midValue 的右边界
        int L = start - 1, R = end + 1;
        
        // 开始partition
        while(start < R) {
            // 划分到左边: 左边界+1, 与start交换, start+1
            if(nums[start] < midValue) {
                swap(nums, ++L, start++);
            } 
            // 划分到中间, 原地不动, 继续遍历
            else if(nums[start] == midValue){
                start++;
            }
            // 划分到右边: 右边界-1, 与start交换, 继续比对当前位置(刚交换上来的值)
            else {
                swap(nums, --R, start);
            }
        }
    }

    private void swap(int[] nums, int from, int to) {
        int tmp = nums[from];
        nums[from] = nums[to];
        nums[to] = tmp;
    }
}
```

#### 88. 合并两个有序数组 | easy

##### 1）拷贝 + 排序法 | O（n + [m + n] * log[ m + n ] ）

- **思路**：这个思路在于，先简单粗暴的把 nums2 中数字，拷贝到 nums1，然后做个顺序排序即可。
- **结论**：时间，1 ms，26.63，空间，41.9 mb，5.11%，时间上，拷贝 nums2 花费 O（n），排序花费 O（ [m + n] * log[ m + n] ），所以整体的时间复杂度为 O（ [m + n] * log[ m + n] ），空间上，快速排序的递归栈深度为 log(m + n)，所以额外空间复杂度为 O（ log [m + n]）。

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        if(nums1 == null || nums2 == null) {
            return;
        }
        // nums2没有元素
        if(n == 0) {
            return;
        }
        // nums1没有元素
        if(m == 0) {
            copyArray(nums2, nums1, 0, n);
            return;
        }

        // O(n)
        copyArray(nums2, nums1, m, n);

        // O(m * logm)
        Arrays.sort(nums1);
    }

    private void copyArray(int[] from, int[] to, int ti, int size) {
        for(int i = 0; i < size; i++) {
            to[ti++] = from[i];
        }
    }
}
```

##### 2）模拟合并有序链表 | O（2 * m + n）

- **思路**：模拟合并有序链表那样，谁小拷贝谁到数组开头，不过这里要做个预处理，那就是先把 nums1 中 0~m-1 的数字，拷贝到 nums1 末尾的 m 位上，以让出空位进行合并。
- **结论**：时间，0 ms，100%，空间，41.7 mb，5.73%，时间上，预处理花费 O（m），合并花费 O（m + n），所以时间复杂度为 O（2 * m + n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        if(nums1 == null || nums2 == null) {
            return;
        }
        // nums2没有元素
        if(n == 0) {
            return;
        }
        // nums1没有元素
        if(m == 0) {
            for(int i = 0; i < n; i++) {
                nums1[i] = nums2[i];
            }
            return;
        }

        // 先拷贝nums1到末尾x
        for(int i = nums1.length - 1, j = m - 1; j > -1; i--, j--) {
            nums1[i] = nums1[j];
        }

        int i1 = nums1.length - m, i2 = 0, idx = 0;
        while(i1 < nums1.length && i2 < nums2.length) {
            if(nums1[i1] <= nums2[i2]) {
                nums1[idx++] = nums1[i1++];
            } else {
                nums1[idx++] = nums2[i2++];
            }
        }

        // i1复制完, 但i2没复制完
        if(i1 == nums1.length) {
            while(i2 < nums2.length) {
                nums1[idx++] = nums2[i2++];
            }
        }

        // i2复制完, 但i1没复制完
        if(i2 == nums1.length) {
            while(i1 < nums1.length) {
                nums1[idx++] = nums1[i1++];
            }
        }
    }
}
```

##### 3）逆序合并 | O（m + n）

- **思路**：由于 num1 和 nums2 中的 m 和 n 个元素，一开始都是有序的，所以要合并 nums2 到 nums1，可以从大的开始合并，也就是从后往前合，这样可以减少预处理的发生。
- **结论**：时间，0 ms，100%，空间，41.7 mb，5.11%，时间复杂度为 O（m + n），额外空间复杂度为 O（1）。

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        if(nums1 == null || nums2 == null) {
            return;
        }
        // nums2没有元素
        if(n == 0) {
            return;
        }
        // nums1没有元素
        if(m == 0) {
            for(int i = 0; i < n; i++) {
                nums1[i] = nums2[i];
            }
            return;
        }

        int idx = nums1.length - 1, i1 = m - 1, i2 = n - 1;
        while(i1 > -1 && i2 > -1) {
            if(nums2[i2] > nums1[i1]) {
                nums1[idx--] = nums2[i2--];
            } else {
                nums1[idx--] = nums1[i1--];
            }
        }
        
        // i1没拷贝完
        if(i2 == -1) {
            while(i1 > -1) {
                nums1[idx--] = nums1[i1--];
            }
        } 
        // i2没拷贝完
        else {
            while(i2 > -1) {
                nums1[idx--] = nums2[i2--];
            }
        }
    }
}
```

##### 4）逆序合并 + 代码量优化 | O（m + n）

- **思路**：观察拷贝过程可发现，由于 nums1 的 m 个元素是有序的，可以在 nums2 结束的时候，如果 nums1 还没拷贝完，那么就不用继续拷贝了，因为 nums1 本身就是有序的，所以结束循环即可。
- **结论**：时间，0 ms，100%，空间，41.9 mb，5.11%，时间复杂度为 O（m + n），额外空间复杂度为 O（1）。

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int idx = nums1.length - 1, i1 = m - 1, i2 = n - 1;

        // i2拷贝完时, 如果i1还没拷贝完, 那也没关系, 因为i1本来剩下的都是有序的 
        while(i2 > -1) {
            // 如果i1拷贝完了, 或者i2比较大, 那么就拷贝i2到目标数组
            if(i1 < 0 || nums2[i2] > nums1[i1]) {
                nums1[idx--] = nums2[i2--];
            } 
            // 如果i1还没拷贝完, 且是i1比较大, 那么就拷贝i2到目标数组
            else {
                nums1[idx--] = nums1[i1--];
            }
        }
    }
}
```

#### 169. 多数元素 | easy

##### 1）暴力解法 | O（2n）

- **思路**：遍历统计每个元素出现的次数，在一个哈希表中登记好，然后遍历这个哈希表，取出现次数大于 n / 2 的元素返回即可。
- **结论**：时间，14ms，14.91%，空间，43.9mb，72.03%，时间上，由于遍历两遍，时间复杂度最坏 O（2n），空间上由于需要一张哈希表，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public int majorityElement(int[] nums) {
        if(nums == null || nums.length == 0) {
            return Integer.MIN_VALUE;
        }

        Map<Integer, Integer> map = new HashMap<>();
        for(int i = 0; i < nums.length; i++) {
            if(map.containsKey(nums[i])) {
                map.put(nums[i], map.get(nums[i]) + 1);
            } else {
                map.put(nums[i], 1);
            }
        }

        for(Map.Entry<Integer, Integer> entry : map.entrySet()) {
            if(entry.getValue() > (nums.length >>> 1)) {
                return entry.getKey();
            }
        }

        return Integer.MIN_VALUE;
    }
}
```

##### 2）排序法 | O（n * logn）

- **思路**：由于求的是出现次数大于 n / 2 的元素，所以对原数组进行顺序排序，目标元素必经过中点，因此取中点元素返回即可。
- **结论**：时间，2ms，61.20%，空间，44.7mb，5.04%，时间上，快排需要 O（n * logn），应该是测试用例没有足够多，所以才看起来比暴力解法的 O（n）还快，而空间上，递归栈的深度为 O（logn）。

```java
class Solution {
    public int majorityElement(int[] nums) {
        if(nums == null || nums.length == 0) {
            return Integer.MIN_VALUE;
        }

        Arrays.sort(nums);
        return nums[nums.length >>> 1];
    }
}
```

##### 3）摩尔投票法 | O（n）

- **思路**：
  1. Boyer- Moore，摩尔投票算法，初始时，记投票数为 0，然后开始遍历。
  2. 如果投票数为 0 时，则选举当前遍历到的元素作为返回值，然后继续遍历。
  3. 如果投票数不为 0，则要看当前遍历到的元素是否等于标记的返回值，如果等于则投票 + 1，否则投票 -1，继续遍历。
  4. 遍历完整个数组，最后标记的返回值就是答案。
- **结论**：时间，1ms，99.91%，空间，44.6mb，5.04%，时间复杂度为 O（n），空间复杂度为 O（1），效率非常好，但没心思去证明它了~

```java
class Solution {
    public int majorityElement(int[] nums) {
        if(nums == null || nums.length == 0) {
            return Integer.MIN_VALUE;
        }

        int count = 0, res = Integer.MIN_VALUE;
        for(int i = 0; i < nums.length; i++) {
            if(count == 0) {
                res = nums[i];
                count++;
            } else {
                count += nums[i] == res? 1 : -1;
            }
        }

        return res;
    }
}
```

#### 283. 移动零 | easy

##### 1）类冒泡排序 | O（n）

- **思路**：参考冒泡排序的思路，不断从头把 0 移动到数组尾部即可，不过这里做了优化，即先记录好一路上 0 的个数，下次碰到非 0 需要与 0 做交换时，直接跳到最前面的 0 即可，避免了 O（n^2）的发生。
- **结论**：时间，2ms，59.21%，空间，42.6mb，5.04%，时间上，由于数组只遍历一次，所以时间复杂度为 O（n），空间上，由于只是用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public void moveZeroes(int[] nums) {
        if(nums == null || nums.length == 0) {
            return;
        }

        int k = nums[0] == 0? 1 : 0;
        for(int i = 1; i < nums.length; i++) {
            if(nums[i] == 0) {
                k++;
            } 
            // 从右向左往前寻找最前的不为0的位置, 找到后则交换当前非0数即可
            else if(nums[i-k] == 0) {
                swap(nums, i, i-k);
            }
        }
    }

    private void swap(int[] nums, int from, int to) {
        int tmp = nums[from];
        nums[from] = nums[to];
        nums[to] = tmp;
    }
}
```

#### 406. 根据身高重建队列 | medium

##### 1）暴力解法 | O（n * [logn + n + k]）

- **思路**：先按 ki 从小到大排序，然后根据使用交换排序变种，遍历每个位置时，再往前统计大于或者等于当前 hi 的元素个数，如果发现统计结果超过了当前 ki，则交换移动差值步，周而复始。
- **结论**：时间，39ms，5.11%，空间，42mb，5.51%，时间上，根据 ki 从小到大排序需要 O（n * logn），遍历时统计最坏时需要 O（n * n），遍历时移动差值步需要 O（n * k），所以时间复杂度为 O（n * [logn + n + k]），空间上，由于使用了一张长度为 n 的一维表以及排序也需要递归深度 logn，所以额外空间复杂度为 O（logn + n）。

```java
class Solution {

    class Info {
        int hi;
        int ki;

        Info(int hi, int ki) {
            this.hi = hi;
            this.ki = ki;
        }
    }

    public int[][] reconstructQueue(int[][] people) {
        if(people == null || people.length == 0 || people[0].length == 0) {
            return new int[0][0];
        }
        
        // 先按ki进行从小到大排序
        List<Info> list = new ArrayList<>(people.length);
        for(int[] member : people) {
            list.add(new Info(member[0], member[1]));
        }
        list.sort((o1, o2) -> o1.ki - o2.ki);

        // 排序后, 从左到右遍历
        int k, m;
        Info io, jo;
        for(int i = 1; i < list.size(); i++) {
            io = list.get(i);
            k = 0;
            for(int j = i - 1; j > -1; j--) {
                jo = list.get(j);
                if(io.hi <= jo.hi) {
                    k++;
                }
            }
            // 往前交换
            m = i;
            if(io.ki < k) {
                while(k - io.ki > 0) {
                    swap(list, m, m-1);
                    m = m - 1;
                    k--;
                }
            }
        }

        // 输出结果
        int[][] res = new int[list.size()][2];
        for(int i = 0; i < list.size(); i++) {
            io = list.get(i);
            res[i][0] = io.hi;
            res[i][1] = io.ki;
        }
        return res;
    }

    private void swap(List<Info> list, int from, int to) {
        Info tmp = list.get(from);
        list.set(from, list.get(to));
        list.set(to, tmp);
    }
}

// 优化后的版本, 时间复杂度不变 O（n * [logn + n + k]），空间复杂度变为 O(logn)
class Solution {
    public int[][] reconstructQueue(int[][] people) {
        if(people == null || people.length == 0 || people[0].length == 0) {
            return new int[0][0];
        }

        Arrays.sort(people, (o1, o2) -> {
            if(o1[1] != o2[1]) {
                return o1[1] - o2[1];
            } else {
                return o1[0] - o2[0];
            }
        });

        int k, m, iki;
        for(int i = 1; i < people.length; i++) {
            k = 0;
            m = i;
            iki = people[i][1];
            for(int j = i - 1; j > -1; j--) {
                // i.hi <= j.hi
                if(people[i][0] <= people[j][0]) {
                    k++;
                }
            }
            // i.ki < k, 即理论ki要为k的, 但实际只有ki, 所以要向前交换
            while(k - iki > 0) {
                swap(people, m, m-1);
                m--;
                k--;
            }
        }

        return people;   
    }

    private void swap(int[][] people, int from, int to) {
        int[] tmp = people[from];
        people[from] = people[to];
        people[to] = tmp;
    }
}
```

##### 2）线性探测法 | O（n * [logn + n]）

- **思路**：
  1. 对原数组先按 hi 顺序排序，同时对相同 hi 的元素按 ki 逆序排序，不过这样得到的只是从低到高的队列，由于 ki 题意表示的是前面数组中，比当前大于或者等于的元素个数，所以当前元素可能站的位置不对。
  2. 因此，由于原数组对相同 hi 的 ki 进行逆序排序，也就是 hi 相同时大的 ki 在前，所以可以利用线性探测的方式，从左到右遍历到原数组，把 i 看作是第一个元素（如果已经有元素了则跳过，只比较为 null 的位置），排在第 ki 位置 ，所以向后探测 ki 次，然后放入 ki + 1 的位置上，
- **结论**：时间，11ms，26.52%，空间，41.9%，6.23%，时间上，根据排序需要 O（n * logn），线性探测需要 O（n^2），所以时间复杂度为 O（n * [logn + n]），空间上，需要递归深度 logn，所以额外空间复杂度为 O（logn）。

```java
class Solution {
    public int[][] reconstructQueue(int[][] people) {
        if(people == null || people.length == 0 || people[0].length == 0) {
            return new int[0][0];
        }

        Arrays.sort(people, (o1, o2) -> {
            if(o1[0] != o2[0]) {
                return o1[0] - o2[0];
            } else {
                // 由于后面是判断res[j]是否为null时, skip才-1, 所以这里排序要逆序排, 保证相同的hi时能够先处理ki大的元素
                return o2[1] - o1[1];
            }
        });

        int skip;
        int[][] res = new int[people.length][];
        for(int i = 0; i < people.length; i++) {
            skip = people[i][1];
            for(int j = 0; j < people.length; j++) {
                // 如果j处的数组还没初始化
                if(res[j] == null) {
                    skip--;
                    if(skip == -1) {
                        res[j] = people[i];
                        break;
                    }
                }
            }
        }

        return res; 
    }
}
```

##### 3）元素挤兑法 | O（n * [logn + n]）

- **思路**：
  1. 先对原数组的 hi 从大到小排序, 相同 hi 的按 ki 从小到大排序, 使得小的 ki 和 hi 先去 list 占位, 保证 list 挤兑后面时不会报 size 校验异常。
  2. 然后利用 list#add 方法的特性，如果有一个元素提前占据了 i 位置，下次再添加元素到 i 位置时，会把之前的元素往后挤兑，所以由于原数组 hi 从大到小排序，相同 hi 的 ki 从小到大排序，因此：
     1. 没有冲突时，可认为数组中 i 元素就排第 i 位，此时相同 hi 大 ki 元素肯定会放置在小 ki 元素的后面。
     2. 发生位置冲突，后面添加的小 hi 元素会把以前大 hi 的元素挤兑，从而保证即使元素后挪了，但 ki 仍无需变化，因为前面插入的是 hi 比它小的元素。
- **结论**：时间，5ms，99.70%，空间，42.1mb，5.35%，时间上，根据排序需要 O（n * logn），list#add需要遍历后面 n次为 O（n^2），所以时间复杂度为 O（n * [logn + n]），空间上，由于使用了排序需要递归深度 logn，并且增加了一个 list，所以额外空间复杂度为 O（logn + n）。

```java 
class Solution {
    public int[][] reconstructQueue(int[][] people) {
        if(people == null || people.length == 0 || people[0].length == 0) {
            return new int[0][0];
        }

        // hi从大到小排序, 相同hi的按ki从小到大排序, 使得小的ki和hi先去list占位, 保证list挤兑后面时不会报size校验异常
        Arrays.sort(people, (o1, o2) -> {
            if(o1[0] != o2[0]) {
                return o2[0] - o1[0];
            } else {
                return o1[1] - o2[1];
            }
        });

        // 亲测: ArrayList每次拷贝后面i-1个元素, 效率比LinkedList遍历到i位置然后添加节点的要高20%+
        List<int[]> list = new ArrayList<>(people.length);
        for(int i = 0; i < people.length; i++) {
            list.add(people[i][1], people[i]);
        }

        return list.toArray(new int[list.size()][]); 
    }
}
```

#### 448. 找出所有数组中消失的数字 | easy

##### 1）暴力解法 | O（3 * n）

- **思路**：先使用 arr 数组存放 [1,n] 作为哈希表，然后从左到右遍历 nums，如果发现 nums[i] 在 arr 中存在，那么清空它，清空完毕后，再遍历一次 arr，收集没被清空的那些就是答案了。 
- **结论**：时间，3ms，99.99%，48.9mb，8.48，时间上，由于需要遍历 3 次 n 长的数组，所以时间复杂度为 O（3 * n），空间上，由于使用了一张 arr 数组，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public List<Integer> findDisappearedNumbers(int[] nums) {
        if(nums == null || nums.length == 0) {
            return new ArrayList<>();
        }

        int[] arr = new int[nums.length];
        for(int i = 0; i < arr.length; i++) {
            arr[i] = i + 1;
        }

        // 4存在3位置, 但值为4
        for(int i = 0; i < nums.length; i++) {
            arr[nums[i] - 1] = -1;
        }

        List<Integer> res = new ArrayList<>();
        for(int i = 0; i < arr.length; i++) {
            if(arr[i] != -1) {
                res.add(arr[i]);
            }
        }
        
        return res;
    }
}
```

##### 2）比较交换法 | O（2 * n）

- **思路**：经过研究数据状态发现，只要每次遍历都把元素放到对的位置，如果碰到目标位置已经是对了，但当前不对，那么说明这个位置就是错误的位置，所以可以在下次遍历时统计收集起来返回即可。
- **结论**：时间，5ms，49.16%，空间，49.1mb，7.54%，时间上，遍历放元素时，由于每个元素放对位置后就不再处理了，所以只会被处理一次，花费 O（n），收集返回结果时遍历数组花费 O（n），因此时间复杂度为 O（2 * n），空间上，由于只使用了有限几个变量，所以空间复杂度为 O（1）。

```java
class Solution {
    public List<Integer> findDisappearedNumbers(int[] nums) {
        if(nums == null || nums.length == 0) {
            return new ArrayList<>();
        }

        int ival, target, i = 0;
        while(i < nums.length) {
            ival = nums[i];
            if(ival == i + 1) {
                i++;
                continue;
            }

            target = nums[ival-1];
            if(target == ival) {
                i++;
            } else {
                nums[i] = target;
                nums[ival-1] = ival;
            }
        }

        List<Integer> res = new ArrayList<>();
        for(i = 0; i < nums.length; i++) {
            if(nums[i] != i + 1) {
                res.add(i+1);
            }
        }

        return res;
    }
}
```

### 2.5. 算法思想 - 二分查找

#### 模板代码

##### 1）数组格子索引角度划分

```java
// 伪代码，这里的l、r、mid是站在格子索引的角度进行分析的
// l、r初始时为非数组格子索引，代表左区间L、右区间R，初始时的长度都为0
l = -1，r = n；

// 结束条件保证，结束时l+1==r
while(l + 1 != r) {
	// mid下界=l下界+r下界=(-1+1)/2=0
	// mid上界=l上界+r上界=(n-2+n)/2=n-1
	mid = l + (r - l) / 2；
	
	// 如果属于左区间，那么把mid归为l，代表纳入左区间，这个需要视具体题目要求而决定
    if(isBlue(mid)) {
    	l = mid；
    } 
    // 如果属于右区间，那么把mid归为r，代表纳入右区间，这个需要视具体题目要求而决定
    else {
        r = mid；
    }
}

// 最后返回的结果，视具体题目要求而决定，比如：
// 1）找到第一个>=5的元素，那么isBlue为<5，那么返回结果就为R的开头，即为r
// 2）找到最后一个<5的元素，那么isBlue为<5，那么返回结果就为L的结尾，即为l
// 3）找到第一个>5的元素，那么isBlue为<=5，那么返回结果就为R的开头，即为r
// 4）找到最后一个<=5的元素，那么isBlue为<=5，那么返回结果就为L的结尾，即为l
return l or r or ...
```

![1655261301568](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1655261301568.png)

##### 2）数组格子间隙角度划分

用的很少，对于要直接用 mid 作为索引，去访问数组的情况，不适用。

```java
// 伪代码，这里的l、r、mid是站在格子间隙的角度进行分析的
// l、r初始时分别为0的左间隙，n的左间隙，代表左区间L、右区间R，初始时的长度分别为0、n
int l = 0, r = n;

// 开始二分划分nums1, 这里的l1和r1, 实质上是在枚举所有的间隙情况, 初始时为0左边的间隙和n左边的间隙
while(l <= r) {
    // 这里的mid, 实质上是[-OO, mid - 1]为左区间, [mid, +OO]为右区间
    mid = l + (r - l) / 2;
    
    // 如果属于左区间，那么把mid纳入L，继续向右枚举间隙
    if(isBlue(mid)) {
        l = mid + 1;
    } 
    // 如果属于右区间，那么把mid纳入R，继续向左枚举间隙
    else {
        r = mid - 1;
    }
}

// 最后返回的结果，视具体题目要求而决定，比如：
// 一般是mid，代表[-OO, mid - 1]为左区间, [mid, +OO]为右区间
// mid最大值为2n/2=n，表示[--OO, n - 1]的区间，最小值为0, 表示[mid, ++OO]的区间
return mid
```

##### 3）考核区间元素角度划分

适用于，考核 [0, n - 1] 范围内的元素，是否符合某种条件。

```java
// 二分开始时，l代表数组第一个元素，r代表最后一个元素，二分的意义则是，在[0，n - 1]范围内去考核元素
int l = 0, r = n - 1;

// 二分终止时，l > r
while(l <= r) {
    // mid最大值为2n-2/2=n-1，表示访问nums[n-1]的元素，最小值为0，表示访问nums[0]的元素
    mid = l + (r - l) / 2;
    
    // 使用mid索引去访问数组，如果符合某种条件，则退出二分
    if(use(mid)) {
        break;
    }
    
    // 应该访问右边，则访问[mid + 1, r]
    if(shouldFindRight()) {
        l = mid + 1;
    }
    // 应该访问左边, 则访问[l, mid - 1]
    else {
        r = mid - 1;
    }
}
```

#### 4. 寻找两个正序数组的中位数 | hard

##### 1）合并两个有序数组 + 取中位数  | O（ m + n ）

- **思路**：构造一个长度为 m + n 的新数组，然后采用 《88. 合并两个有序数组》的思路，把 nums1 和 nums2 按顺序合并到新数组中，最后对新数组分奇、偶数情况，分别求中位数即可。
- **结论**：时间，1 ms，100%，空间，42.3 mb，50.75%，时间上，由于需要遍历完 nums1 和 nums2 数组，所以时间复杂度为 O（m + n），空间上，由于使用了一个 m + n 长的新数组，所以额外空间复杂度为 O（m + n）。

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        if(nums1 == null || nums2 == null) {
            return 0;
        }

        int m = nums1.length, n = nums2.length;
        int[] nums = new int[m + n];
        int i1 = m - 1, i2 = n - 1, idx = nums.length - 1;
        while(i1 > -1 && i2 > -1) {
            if(nums1[i1] >= nums2[i2]) {
                nums[idx--] = nums1[i1--];
            } else {
                nums[idx--] = nums2[i2--];
            }
        }

        // 拷贝剩余
        while(i1 > -1) {
            nums[idx--] = nums1[i1--];
        }
        while(i2 > -1) {
            nums[idx--] = nums2[i2--];
        }

        if(nums.length % 2 == 1) {
            return (double) nums[nums.length / 2];
        } else {
            int r = nums.length / 2;
            int l = r - 1;
            return ((double)  nums[r] + nums[l]) / 2;
        }
    }
}
```

##### 2）异位双指针 + 取中位数 | O（ [ m + n] / 2 ）

- **思路**：
  1. 观察上面的情况发现，其实并不需要把所有的数都访问一遍，只需要访问前 k 个元素就好，所以可以采用双指针的方式，严格控制访问的顺序。
  2. 这需要分 4 种情况讨论，
     - 1）前 2 种是：当 nums1 已经遍历完了、或者 nums1 比较小时，这时可以取访问 nums1 的元素，看 idx 是否符合要求了，是的话则设置好结果。
     - 2）同理，后两者则是：当 nums2 已经遍历完了、或者 nums2 比较小时，这时可以取访问 nums2 的元素，看 idx 是否符合要求了，是的话则设置好结果。
  3. 最后，对得到的结果，分奇、偶数情况，分别求中位数即可。
- **结论**：时间，1 ms，100%，空间，42.6 mb，8.20%，时间上，由于只需要访问前 k 个元素，所以时间复杂度为 O（ [ m + n] / 2 ），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Info {
    int li;
    int ri;
    int left;
    int right;

    Info(int li, int ri, int left, int right) {
        this.li = li;
        this.ri = ri;
        this.left = left;
        this.right = right;
    }
}

class Solution {

    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        if(nums1 == null || nums2 == null) {
            return 0d;
        }

        int m = nums1.length, n = nums2.length, len = m + n, half = len / 2;
        int i1 = 0, i2 = 0, idx = 0;
        // li = len % 2 == 1? half : half - 1, ri = half, 
        // left = 0, right = 0;
        Info info = new Info(len % 2 == 1? half : half - 1, half, 0, 0);

        // 开始遍历 O(1/2 * [m + n])
        while(idx <= half) {
            // nums1已用完, nums2必没用完
            if(i1 == nums1.length) {
                setValue(info, idx++, nums2[i2++]);
            }
            // nums2已用完, nums1必没用完
            else if(i2 == nums2.length) {
                setValue(info, idx++, nums1[i1++]);
            }
            // nums1小
            else if(nums1[i1] <= nums2[i2]) {
                setValue(info, idx++, nums1[i1++]);
            }
            // nums2小
            else if(nums2[i2] < nums1[i1]) {
                setValue(info, idx++, nums2[i2++]);
            }
        }
    
        return (double) (info.left + info.right) / 2;
    }

    private void setValue(Info info, int idx, int val) {
        if(idx == info.li) {
            info.li = -1;
            info.left = val;
        }
        if(idx == info.ri) {
            info.ri = -1;
            info.right = val;
        }
    }
}
```

##### 3）奇偶判断 + 寻找第 k 小地数 + 异位二分 | O（ log [ m + n ] ）

- **思路**：

  1. 首先，由于题目要求求中位数，这时应该分情况讨论，如果总数为奇数的话，那么中位数的索引为 k / 2，如果总数为偶数的话，那么中位数的索引为 k / 2 - 1 和 k / 2。

  2. 然后，由于是中位数，且数组是顺序数组，所以中位数相当于求第 k 小的元素（序号从 1 开始），只不过这道题是需要在两个数组中求第 k 小的元素，回到刚才的奇偶数分析，此时，奇数情况下，要求的是第 k / 2 + 1 小的元素，偶数情况下，要求的是第 k / 2 和 第 k / 2 + 1 小的元素。

  3. 接着，由于题目要求时间复杂度为 log 水平，所以应该想到的就是二分了，也就是规定什么时候、丢弃哪一半的元素。经过研究数据状态，可以发现，要求查找第 k 小的数，那么就需要比较两数组中前 k / 2 个数，那么他们最后一个元素的下标则是 k / 2 - 1，然后分为以下三种情况来讨论：

     - 1）A [k / 2 - 1] < B[ k / 2 - 1]：A 和 B 都是顺序的，由于要求的是第 k 小的数，此时，如果 A 的元素比 B 小，那么说明 A[1, k / 2 - 1] 中的元素，只是第 0,1,..., k / 2 的数，第 k 小的数则肯定是落在了， A [k / 2 + 1, ..., m ] 或者 B[ 0, 1, ..., n ] 之间的某一个。所以，这时应该把 A 的前半部分抛弃，K 减少掉被抛弃的元素个数，然后继续二分处理后面的数据。
     - 2）A[k / 2 - 1] > B[K / 2 - 1]：这种情况与第一种相反，所以，这时应该把 B 的前半部分抛弃，K 减少掉被抛弃的元素个数，然后继续二分处理后面的数据。
     - 3）A[k / 2 - 1] == B[k / 2 - 1]：这种情况，只能说明 A[0, 1, ..., k / 2 - 2] 中的元素肯定不是第 k 小的数，但不能说明 A[k / 2 - 1] 不是第 k 小的数，不过由于 A 和 B 元素相等，那么也可以把 B[k / 2 - 1] 看作替换掉 A[k / 2 - 1]，抛弃掉 A[k / 2 - 1] 及其之前的元素即可，因此，这里的处理方法等价于情况一。

     ![1655127840962](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1655127840962.png)

  4. 最后，就是边界处理了：

     - 1）如果碰到 nums1 的 i1 越界了，说明 nums1 遍历完了，此时只需要在 nums2 上，直接求第 k 小的数即可。
     - 2）同理，如果碰到 nums2 的 i2 越界了，说明 nums2 遍历完了，此时只需要在 nums1 上，直接求第 k 小的数即可。
     - 3）如果 i1 和 i2 都没越界，那么比较前，先判断 k 是否为 1 了，如果为 1，说明要求的只是第 1 小，也就是最小的数，此时只需要比较 nums1[0] 和 nums2[1] 的大小，返回小者即可。
     - 4）如果 i1 和 i2 都没越界，且 k 不为 1 即至少要比较的是 [0, ..., k / 2 - 1] 个数，那么就走上面的 A 和 B 的二分处理逻辑，周而复始，最后定能从 1、2、3 点边界处理中返回。

- **结论**：时间，1 ms，100%，空间，42.2 mb，51.97%，时间上，由于二分是不断地丢弃掉一半，所以时间复杂度为 O（ log [ m + n ] ），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {

    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        if(nums1 == null || nums2 == null) {
            return 0d;
        }

        int m = nums1.length, n = nums2.length, k = m + n, half = k / 2;
        
        // 整个长度为奇数
        if(k % 2 == 1) {
            // 奇数的中位数索引为k/2, 如果序号从1开始的话, 为k/2 + 1
            return (double) findKthInTwoSortedArr(nums1, nums2, m, n, half + 1);
        }
        // 整个长度为偶数
        else {
            return avg(
                // 偶数的左边中位数索引为k/2-1, 如果序号从1开始的话, 为k/2
                findKthInTwoSortedArr(nums1, nums2, m, n, half),

                // 偶数的右边中位数索引为k/2, 如果序号从1开始的话, 为k/2 + 1
                findKthInTwoSortedArr(nums1, nums2, m, n, half + 1)
            );
        }
    }

    private double avg(int left, int right) {
        return (double) (left + right) / 2;
    }

    // 从两个顺序数组中, 寻找第k小的数值, 其中序号从1开始
    private int findKthInTwoSortedArr(int[] nums1, int[] nums2, int m, int n, int k) {
        // nums1和nums2每轮的起始索引, 开头已经被抛弃的个数, 本轮nums1和nums2比较的最右边的索引
        int i1 = 0, i2 = 0, decnt = 0, r1, r2;
        while(true) {
            // 减去开头被抛弃的个数
            k-= decnt;

            // 出口1：i1走到了尽头, 那么就从nums2中找
            if(i1 == m) {
                return nums2[k - 1 + i2];
            }
            // 出口2：i2走到了尽头, 那么就从nums1中找
            if(i2 == n) {
                return nums1[k - 1 + i1];
            }
            // 出口3：如果nums1和nums2都不为空, 但k已经为1了
            // 那么就不用进行+k/2的区间比较了, 直接比较第一个数, 返回较小者即可
            if(k == 1) {
                return Math.min(nums1[i1], nums2[i2]);
            }

            // 如果nums1和nums2都不为空, 那么取各自一半长度进行比较, 最右边的索引为k/2-1, 以决定丢弃哪一半
            // 丢弃的理由是：当k不是, 由于是一半, 小者的那个一半, 肯定是不满足要求的一半, 则可以全部丢弃
            r1 = Math.min(m, k/2 + i1) - 1;
            r2 = Math.min(n, k/2 + i2) - 1;
            // 开始比较, 相等的情况照样抛弃小者
            
            // r1小, 则抛弃[0, r1], 然后继续看下一段, 从r1+1开始
            if(nums1[r1] <= nums2[r2]) {
                decnt = r1 - i1 + 1;
                i1 = r1 + 1;
            } 
            // r2小, 则抛弃[0, r2], 然后继续看下一段nums2
            else {
                decnt = r2 - i2 + 1;
                i2 = r2 + 1;
            }
        }
    }
}
```

##### 4）中位数定义 + 数组左右区间划分 | O（ log[ min(m, n) ] ）

- **思路**：
  1. 根据中位数定义有，将一个集合划分为两个长度相等的子集，其中的一个子集的元素，总是大于另外一个子集的元素。所以，就有了划分数组左右区间的思路。
  2. 由于这里是两个顺序数组，所以，可以在 A#i、B#j 位置进行划分，使得 A[-oo, i - 1] 为 A 的左区间，A[i, +oo] 为 A 的右区间，B[-oo, j - 1] 为 B 的左区间，B[j, +oo] 为 B 的右区间。
  3. 此时，如果能有 A[i - 1] <= B[j] 且 B[j - 1] <= A[i] 的话，说明 A#i、B#j 的位置划分正确，需要再往右二分，以尽可能划分更大的区间，否则，则需要往左二分，以往前寻找更小的可行解，这就是本题二分的应用所在。
  4. 最后，如果二分结束，说明找到了最大的左区间，即左区间的元素个数，最接近右区间的个数（奇数时，左 size == 右 size - 1，偶数时，左 size == 右 size），此时，如果为奇数，则 lmax 就是解，否则 lmax 和 rmin 取平均值才是解。
- **结论**：时间，1 ms，100%，空间，42.4 mb，26.23%，时间上，由于实现对 nums1 和 nums2 数组进行了交换，使得长度 m <= n，且是在 m 上进行区间二分，所以时间复杂度为 O（log m），即 O（log [m, n] ），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        if(nums1 == null || nums2 == null) {
            return 0;
        }

        // 交换短的数组作为nums1, 长的数组作为nums2
        if(nums1.length > nums2.length) {
            int[] tmp = nums2;
            nums2 = nums1;
            nums1 = tmp;
        }

        int m = nums1.length, n = nums2.length, len = m + n;
        int l1 = 0, r1 = m, mid1, mid2, lmax1, lmax2, rmin1, rmin2, lmax = 0, rmin = 0;

        // 开始二分划分nums1, 这里的l1和r1, 实质上是在枚举所有的间隙情况, 初始时为0左边的间隙和m左边的间隙
        while(l1 <= r1) {
            // 这里的mid, 实质上是[-OO, mid - 1]为左区间, [mid, +OO]为右区间
            mid1 = l1 + (r1 - l1) / 2;

            // 长度i+长度j=左区间长度, m-i+n-j=右区间长度, 由于是在求中位数, 其含义是划分左右两个等长区间
            // 对于偶数情况, i+j=m-i+n-j, 对于奇数情况, i+j=m-i+n-j+1
            // => 可推出：i + j = (m + n + 1) / 2, 这里是根据i求出j
            mid2 = (len + 1) / 2 - mid1;

            // 求左区间的最大值, 如果mid大于0, 说明存在左区间, 则最大值等于i-1的值, 否则为-OO
            lmax1 = mid1 - 1 > -1? nums1[mid1 - 1] : Integer.MIN_VALUE;
            lmax2 = mid2 - 1 > -1? nums2[mid2 - 1] : Integer.MIN_VALUE;

            // 求右区间的最小值, 如果mid小于m, 说明存在右区间, 则最小值等于i的值, 否则为+OO
            rmin1 = mid1 < m? nums1[mid1] : Integer.MAX_VALUE;
            rmin2 = mid2 < n? nums2[mid2] : Integer.MAX_VALUE;

            // 经过证明, 只需要保证A[i - 1] <= B[j], 即可保证B[j - 1] <= A[i]
            // 因此, 如果A[i - 1] <= B[j], 说明本次区间划分有效
            // 则记好左区间的最大和右区间的最小值, 然后l继续向右枚举间隙即可
            if(lmax1 <= rmin2) {
                lmax = Math.max(lmax1, lmax2);
                rmin = Math.min(rmin1, rmin2);
                l1 = mid1 + 1;
            } 
            // 否则, 如果A[i - 1] > B[j], 说明本次区间划分无效
            // 则无需记录本次的划分, 直接r继续向左枚举间隙即可
            else {
                r1 = mid1 - 1;
            }
        }

        // 由于mid划分的是区间, 实质上是[-OO, mid - 1]为左区间, [mid, +OO]为右区间
        // 所以, 在奇数时, mid-1为lmax, mid本身为rmin, 如果像偶数那样直接求平均的话, 那么就有问题了
        // 因为此时应该为lmax才对
        return len % 2 == 1? lmax : (double) (lmax + rmin) / 2;
    }
}
```

#### 29. 两数相除 | medium

##### 1）二分查找 + 快速乘 | O（ [ logc ]^2 ）

- **思路**：
  1. 由于不能用任何乘法、除法、模法，也只能用 int 存储，还需要提高程序速度，这就不能单存的遍历扣减，还需要做好多细节工作。
  2. 第一个细节就是，预处理，对那些特殊情况，以及会引起 int 溢出的条件进行判断，比如 0、Integer.MIN_VALUE，具体的逻辑看程序。
  3. 第二个细节就是，取反处理，由于会输入即可能是负数、也可能是正数，所以需要取反，化成一种情况处理，而对负数取反，可能会导致 -2^32 取反得到 2^32，造成 int 溢出，所以这里做了反方向的取反，也就是对正数取反，统一用负数进行处理。
  4. 第三个细节就是，非除实现，这类似于《69. x 的平方根》的思路，都是求最逼近 y * z >= x 中 z 的峰值作为答案，所以也是使用二分查找，但需要注意 int 溢出，所以用了 l + ((r - l ) >> 1) 的方式求 mid 中点。以及由于是求下限的二分，所以二分条件用了 l <= r、l = mid + 1、r = mid - 1。还有就是 mid 的终止判断，如果发现 mid 已经最大了，就没必要继续二分了，此时退出循环，做最后的结果处理就好。
  5. 第四个细节就是，结果处理，由于是对数字进行实现非除，所以 res 是个正数，此时还要结合以前的符号位，以决定是否取反，这个符号位可以在《第 2 步取反处理》的时候进行判断，通过两次对 isNegativeRes 的取反运算，即可得出最终结果是否会为负数的结论，非常的巧妙~
- **结论**：时间，1 ms，70.07%，空间，38.7 mb，51.13%，时间上，主方法进行二分，需要花费 O（logC），这里 C 为 Integer.MAX_VALUE，而主方法调用的 `checkYmultiZLtgtX()` 检查方法，最多也需要花费 O（logC），所以整体时间复杂度为 O（ [ logc ]^2 ），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int divide(int dividend, int divisor) {
        // 0不能作为除数
        if(divisor == 0) {
            throw new RuntimeException("0不能作为被除数!");
        }
        // 0除以任何数都位0
        if(dividend == 0) {
            return 0;
        }

        // 被除数等于下限
        if(dividend == Integer.MIN_VALUE) {
            // -2^32 / 1 = -2^32 => 简化运算
            if(divisor == 1) {
                return Integer.MIN_VALUE;
            } 
            // -2^32 / -1 => 防止int的2^32溢出
            else if(divisor == -1) {
                return Integer.MAX_VALUE;
            }
        }

        // 除数等于下限
        if(divisor == Integer.MIN_VALUE) {
            // -2^32 / -2^32 = 1 => 简化运算
            if(dividend == Integer.MIN_VALUE) {
                return 1;
            } 
            // x / -2^32 => 0, 因为上限、下限都超不过2^32
            else {
                return 0;
            }
        }

        // 先对x、y求相反数, 都往负的方向取, 可以保证不会溢出, 因此负数的下限较高
        boolean isNegativeRes = false;
        if(dividend > 0) {
            isNegativeRes = !isNegativeRes;
            dividend = -dividend;
        }
        if(divisor > 0) {
            isNegativeRes = !isNegativeRes;
            divisor = -divisor;
        }

        // x / y, 当x、y都为负数时, z必定不为0, 所以l从1开始, 而不是0
        // 但由于最终结果取的是下限, 所以res有可能为0, 
        // 对于任意一正数z, 当z * y >= x 满足时, 则必定有(z + 1) * y < x
        // 这说明了z存在极大值(峰值), 而要求x / y, 上述公式 z * y >= x, 由于y是负数
        // 所以会转换为 z <= x / y, 所以求出Z的峰值就是最终最接近x / y的答案了
        int l = 1, r = Integer.MAX_VALUE, mid, res = 0;
        while(l <= r) {
            // 防止溢出, 代替(r + 1) >> 1
            mid = l + ( (r - l) >> 1 );

            // 检查 y * z >= x, 是的话, 则看是否右区域存在更大的z
            if(checkYmultiZLtgtX(divisor, mid, dividend)) {
                res = mid;

                // 如果mid已经最大了, 就不用再二分了
                if(mid == Integer.MAX_VALUE) {
                    break;
                }
                
                l = mid + 1;
            } 
            // 不是的话, 则看往回看, 是否出现在之前的左区域
            else {
                r = mid - 1;
            }
        }

        return isNegativeRes? -res : res;
    }

    // 检查 y * z >= x, 但不能用乘法, 只能用加法, 此时可以用快速乘算法, 也就是快速幂的加号版
    private boolean checkYmultiZLtgtX(int y, int z, int x) {
        int res = 0, factor = y;
        while(z > 0) {
            // 二进制位为1的, 则累加到结果中
            if((z & 1) == 1) {
                // 累加前先判断res + factor >= x, 注意, 要用减法防止溢出
                // 但由于res自己累加过程中也可能溢出, 所以这里做了反方向的判断
                if(res < x - factor) {
                    return false;
                }
                
                res += factor;
            }

            // 最后一步不要做factor判断, 因为z次的y都累加过了, 这次判断属于是(z+1)次的判断了
            // 由于y=factor是负数, 所以(z+1)次的factor累加, 肯定比x小, 所以跳过这次校验即可
            if(z != 1) {
                // 累加前先判断factor + factor >= x, 注意, 要用减法防止溢出
                if(factor < x - factor) {
                    return false;
                }

                // 最后factor累加一次
                factor += factor;
            }

            // 然后z右移一位
            z >>= 1;
        }

        return true;
    }
}
```

#### 33. 搜索旋转排序数组 | medium

##### 1）二分查找 | O（logn）

- **思路**：
  1. 可以发现，将数组从中间分开左右两部分，一定是有一部分顺序的，所以，可以确定哪个部分有序，进而确定如何改变二分查找的上下界即可。
  2. 如果 mid 比 nums[0] 小，说明 nums[0] 是换过去的，即 [mid, n-1] 有序，则可以在这部分上进行二分。
  3. 如果mid 大于等于 nums[0]，说明nums[0] 没有交换过，即 [0, mid] 是有序的，则可以在这部分上进行二分。
  4. 周而复始，最终找到 target 则返回其下标，否则返回 -1，符合考核区间元素角度的二分模板。
- **结论**：时间，0 ms，100%，空间，41.5 mb，5.02%，时间上，二分要么丢弃左半部分，要么丢弃右半部分，所以时间复杂度为 O（logn），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int search(int[] nums, int target) {
        int n = nums.length, member0 = nums[0], membern = nums[n - 1];
        if(n == 0) {
            return -1;
        }
        if(n == 1) {
            return member0 == target? 0 : -1;
        }

        // 开始二分
        int l = 0, r = n - 1, mid;
        while(l <= r) {
            mid = l + (r - l) / 2;
            if(nums[mid] == target) {
                return mid;
            }

            // 从mid一刀切开, 肯定是一半有序的, 一半无序的
            // 如果mid比nums[0]小, 说明nums[0]是换过去的, [mid, n-1]有序
            if(nums[mid] < member0) {
                // [mid, n-1]有序, 则在这部分进行二分
                if(nums[mid] < target && target <= membern) {
                    l = mid + 1;
                } 
                // 否则, 在[0, mid-1]上进行二分
                else {
                    r = mid - 1;
                }
            }
            // 如果mid大于等于nums[0], 说明nums[0]没有交换过, 即[0, mid]是有序的
            else {
                // [0, mid]有序, 则在这部分进行二分
                if(member0 <= target && target < nums[mid]) {
                    r = mid - 1;
                } 
                // 否则, 在[mid+1, n-1]上进行二分
                else {
                    l = mid + 1;
                }
            }
        }

        return -1;
    }
}
```

#### 34. 在排序数组中查找元素的第一个和最后一个位置 | medium

##### 1）两次二分 | O（2 * logn）

- **思路**：
  1. 这道题是完全符合 l + 1 != r 的代码模板的，所以，需要研究其左区间和右区间的划分方式。
  2. 当在寻找起始索引时：
    - 1）isBlue == true，说明为左区间，条件是：< target 的左区间。
    - 2）isBlue == false，说明为右区间，条件是：>= target 的右区间。
    - 3）所以，起始索引结果是，右区间开始的索引，也就是 r，不过要做越界和是否为 target 的判断。
  3. 当在寻找结束索引时：
    - 1）isBlue == true，说明为左区间，条件是：>= target 的左区间。
    - 2）isBlue == false，说明为右区间，条件是：< target 的右区间。
    - 3）所以，结束索引结果是，左区间结束的索引，也就是 l，不过要做越界和是否为 target 的判断。
  4. 最后，在主方法中，分别获取起始索引和结束索引即可。
- **结论**：时间，0 ms，100%，空间，44.9 mb，5.30%，时间上，由于做了两次二分，所以时间复杂度为 O（2 * logn），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        if(nums == null || nums.length == 0) {
            return new int[] {-1, -1};
        }
        return new int[] {
            findStartIndex(nums, target),
            findEndIndex(nums, target)
        };
    }

    // 找起始索引
    private int findStartIndex(int[] nums, int target) {
        int l = -1, r = nums.length, mid;
        while(l + 1 != r) {
            mid = l + (r - l) / 2;
            
            // 非target的左区间
            if(nums[mid] < target) {
                l = mid;
            } 
            // 大于等于target的右区间
            else {
                r = mid;
            }
        }

        // 右区间的开头, 就是起始索引
        return r != nums.length && nums[r] == target? r : -1;
    }

    // 找结束索引
    private int findEndIndex(int[] nums, int target) {
        int l = -1, r = nums.length, mid;
        while(l + 1 != r) {
            mid = l + (r - l) / 2;
            
            // target的左区间
            if(nums[mid] <= target) {
                l = mid;
            } 
            // 大于target的右区间
            else {
                r = mid;
            }
        }

        // 左区间的结尾, 就是结束索引
        return l != -1 && nums[l] == target? l : -1;
    }
}
```

#### 35. 搜索插入位置 | easy

##### 1）二分查找 + 索引角度划分区间 | O（logn）

- **思路**：
  1. 从索引角度划分格子，初始时，l 为 -1，r 为 n，都不为数组中的某个格子，也就是都不是要找的结果。
  2. 然后开始二分，如果判断到 mid 大于等于 target，则将其加入右侧的红色区域，否则加入左侧的蓝色区域。
  3. 最后二分结束，根据题意，要返回的是第一个比 target 大于等于的数字的索引，也就是此时红色区域中开头的索引 r 即可。
- **结论**：时间，0 ms，100%，空间，40.8 mb，78.20%，时间复杂度为 O（logn），额外空间复杂度为 O（1）。

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        // 二分查找, 找到第一个比它大于等于的数字的索引
        int l = -1, r = nums.length, mid;
        while(l + 1 != r) {
            mid = l + (r - l) / 2;
            
            // 红色区域
            if(nums[mid] >= target) {
                r = mid; 
            } 
            // 蓝色区域
            else {
                l = mid;
            }
        }

        // 最后返回红色区域的开始位置
        return r;
    }
}
```

#### 50. Pow(x, n) | medium

##### 1）递归式快速幂 | O（logn）

- **思路**：快速幂原理：
  1. 如果当前 n 为偶数，则等于前一个 power 的平方，相当于 2^64 = (2^32)^2。
  2. 如果当前 n 为奇数，则等于前一个 power 的平方 * 再乘一个 x，相当于 2^65 = (2^32)^2 * 2。
  3. 其中，由于 n 可能会有负数，所以可以先对 n 求相反数，拿到结果后再求倒数就好。
  4. 而由于 n 取了相反数，碰到 -2^32 时取相反数，则可能会导致 int 溢出，所以需要先对 n 换成更长的 long 类型再做操作。
- **结论**：时间，0 ms，100%，空间，40.5 mb，60.32%，时间上，由于不断地对 n 进行求一半，所以时间复杂度为 O（logn），空间上，由于递归栈深度为 logn，所以额外空间复杂度为 O（logn）。

```java
class Solution {
    public double myPow(double x, int n) {
        long ln = n;
        return ln < 0? 1 / doPow(x, -ln) : doPow(x, ln); 
    }

    // n必定为非负数
    private double doPow(double x, long n) {
        if(n == 0) {
            return 1d;
        }

        // 快速乘, 对n/2求power
        double y = doPow(x, n >> 1);

        // 如果当前n为偶数, 则等于前一个power的平方, 相当于2^64=(2^32)^2
        // 如果当前n为奇数, 则等于前一个power的平方 * 再乘一个x, 相当于2^65=(2^32)^2 * 2
        return (n & 1) == 0? y * y : y * y * x;
    }
}
```

##### 2）迭代式快速幂 | O（logn）

- **思路**：

  1. 由于快速乘幂要等到上一次的 power 结果，还要直到当前的 n 是偶数还是奇数，如果要换成迭代式的话，只能从 n 开始进行除一半，但这样又不知道上一次的 power 结果，所以需要研究这些 n 的数据状态。

     ![1654869770771](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1654869770771.png)

  2. 可以看到打上 + 号的地方，就是 n 为奇数时的地方：

     - 1）x^38 -> x^77：由于 77 后面还被平方了 1 次，所以，该过程相当于为最终结果贡献了 x^(2^0 = x^1。
     - 2）x^19 -> x^38：由于 38 后面还被平方了 2 次，所以，该过程相当于为最终结果贡献了 x^2 = x^4。
     - 3）x^9 -> x^19：由于 19 后面还被平方了 3 次，所以，该过程相当于为最终结果贡献了 x^(2^3)  = x^8。
     - 4）而原始的 x^1 -> x^77：由于 1 后面还被平方了 6 次，所以，该过程相当于为最终结果贡献了 x^(2^6)  = x^64。
     - 5）这样累加贡献的 x，即可得： x^77 = x^1 + x^2 + x^3 + x^6。

  3. 而 77 分解为二进制形式则为 `100,1101`：即 6、3、2、0 的位置都为 1，此时，恰恰对应则上面多贡献的地方，因此，x^77 = x^(2^0) + x^(2^2) x^(2^3) +  x^(2^3) + x^(2^6)。

- **结论**：时间，0 ms，100%，空间，41 mb，5.12%，时间上，由于需要遍历 n 中所有的二进制位，所以时间复杂度为 O（logn），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public double myPow(double x, int n) {
        long ln = n;
        return ln < 0? 1 / doPow(x, -ln) : doPow(x, ln); 
    }

    // n必定为非负数
    private double doPow(double x, long n) {
        if(n == 0) {
            return 1d;
        }

        double res = 1d, factor = x;
        while(n > 0) {
            // 只取二进制位为1的位, 进行参与结果计算
            if((n & 1) == 1) {
                res *= factor;
            }

            // n往右移1位
            n >>= 1;
            
            // 然后乘数因子进行一次平方
            factor *= factor;
        }

        return res;
    }
}
```

#### 69. x 的平方根 | easy

##### 1）Math.sqrt() 法 | O（1）

- **思路**：直接使用 Math.sqrt() 内置函数，再向下取整得出答案。
- **结论**：时间，0 ms，100%，空间，38.8 mb，42.51%，时间复杂度为 O（1），额外空间复杂度为 O（1），但由于不能使用内置函数，所以这种做法是不通过的。

```java
class Solution {
    public int mySqrt(int x) {
        return (int) Math.sqrt(x);
    }
}
```

##### 2）公式转换法 | O（1）

- **思路**：不能直接使用 Math.sqrt()  函数，那么就用数学公式来转换，即：

  ![1654862180285](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1654862180285.png)

-  **结论**：时间，0 ms，100%，空间，38.6 mb，57.63%，时间复杂度为 O（1），额外空间复杂度为 O（1），但由于不能使用内置函数，所以这种做法也是不通过的。

```java
class Solution {
    public int mySqrt(int x) {
        if(x == 0) {
            return 0;
        }

        // 向下取整可能导致差了10^ -11
        int maybeRes = (int) Math.exp(0.5 * Math.log(x));

        // 所以还需要判断(i+1)^2是否更加接近x, 是的话则返回i+1, 否则取i就可以了
        return (long) (maybeRes + 1) * (maybeRes + 1) <= x? maybeRes + 1 : maybeRes;
    }
}
```

##### 3）二分查找法 | O（logn）

- **思路**：二分逐步尝试 mid^2 逼近 x，不断地大于、小于来回，如果 mid^2 大了，那么应该尝试更小的 mid^2，取左半部分，如果 mid^2 小了，那么应该尝试更大的 mid^2，取右半部分，而由于最终结果取的是下限，所以需要对二分的临界点做相应的修改。
- **结论**：时间，1 ms，94.55%，空间，38.7 mb，50.30%，时间上，由于二分要么丢弃左半部分，要么丢弃右半部分，所以时间复杂度为 O（logn），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int mySqrt(int x) {
        int l = 0, r = x, mid = (r + l) >>> 1;
        long tmp;

        // l==r继续比较, 是因为要尝试最后一种情况的mid^2是否接近x
        while(l <= r) {
            tmp = (long) mid * mid;

            // 如果mid^2等于x, 那么直接返回mid
            if(tmp == x) {
                return mid;
            }
            // 如果mid^2大于x, 那么要尝试更小的mid^2
            // 这里的mid-1是为了保证l=r的mid^2还大时, 能够返回更小的r, 因为2.7777取下限2
            else if(tmp > x) {
                r = mid - 1;
            } 
            // 如果mid^2小于x, 那么要尝试更大的mid^2
            // 这里的mid+1是为了保证l=r的mid^2还小时, 能够返回更小的r, 因为2.7777取下限2
            else {
                l = mid + 1;
            }
            
            // 继续二分
            mid = (r + l) >>> 1;
        }

        // 返回更小的r, 以保证取了下限
        return r;
    }
}
```

##### 4）牛顿迭代法 | O（logn）

- **思路**：

  1. 牛顿迭代法，是一种可以用来快速求解函数零点的方法，其本质是借助泰勒级数，从初始值开始快速向零点逼近。

     ![1654864226001](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1654864226001.png)

  2. 其中，迭代的结束条件为，相邻两次的迭代结果的差值，小于一个极小的非负数则认为非常接近零点，即方程 0 = x^2 - c 的解了，这里极小值用的是 10^( -7 )。

- **结论**：时间，1 ms，94.55%，空间，38.6 mb，53.70%，时间上，由于牛顿迭代法是平方收敛的，所以时间复杂度为 O（logn），但比二分查找的 O（logn）收敛更快，空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int mySqrt(int x) {
        if(x == 0) {
            return 0;
        }

        double c = x, x0 = x, x1;
        while(true) {
            x1 = 0.5 * (x0 + c / x0);
            if(Math.abs(x1 - x0) < 1e-7) {
                break;
            }
            x0 = x1;
        }
        
        return (int) x1;
    }
}
```

#### 74. 搜索二维矩阵 | medium

##### 1）二叉搜索树查找 | O（log [m * n]）

- **思路**：把矩阵斜着看成一棵二叉搜索树，从(0, 0)出发，如果 target 比同行末尾大，那么查找下一行，否则往右边查找。
- **结论**：时间，0 ms，100%，空间，41.4 mb，7.22%，时间上，由于最多需要遍历 log（m * n），即斜着遍历，所以时间复杂度为 O（log [m * n]），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        // 二叉搜索树查找, 从(0, 0)出发, 如果比同行末尾大, 那么查找下一行, 否则查找右边
        int m = matrix.length, n = matrix[0].length, i = 0, j = 0;
        while(i < m && j < n) {
            if(target == matrix[i][j]) {
                return true;
            } else if(target > matrix[i][n - 1]) {
                i++;
            } else {
                j++;
            }
        }
        return false;
    }
}
```

##### 2）首列上二分 + 指定行上二分 | O（log [m * n]）

- **思路**：
  1. 首先在首列上二分，找到最后一个小于等于 target 的行下标，然后再在该行上二分，查找是否存在等于 target 的元素。
  2. 其中要注意的是，要对首列上二分返回的行下标进行讨论，如果越界了，说明首列上的值都大于 target，即 target 肯定不存在，所以直接返回 false 即可，而如果 target 就落在该行上的 0 位置，那么直接返回 true 即可，否则只能去该行上二分了。
- **结论**：时间，0 ms，100%，空间，41.1 mb，46.40%，时间上，由于最多需要做两次二分，所以时间复杂度为 O（log m + logn）= O（log [m * n]），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int m = matrix.length, n = matrix[0].length;

        // 找到第一列上, 最后一个小于等于target的行下标
        int rowidx = binarySearchLastLtRow(matrix, target, m, n);
        if(rowidx == -1) {
            return false;
        }
        if(matrix[rowidx][0] == target) {
            return true;
        }

        // 在指定行上, 二分查找是否存在等于target的元素
        return binarySearchAtRow(matrix, target, m, n, rowidx);
    }

    // 在指定行上, 二分查找是否存在等于target的元素
    private boolean binarySearchAtRow(int[][] matrix, int target, int m, int n, int rowidx) {
        int l = -1, r = n, mid;
        while(l + 1 != r) {
            mid = l + (r - l) / 2;
            if(matrix[rowidx][mid] == target) {
                return true;
            }
            // 蓝色区域
            else if(matrix[rowidx][mid] < target) {
                l = mid;
            }
            // 红色区域
            else {
                r = mid;
            }
        }
        return false;
    }

    // 找到第一列上, 最后一个小于等于target的行下标
    private int binarySearchLastLtRow(int[][] matrix, int target, int m, int n) {
        int l = -1, r = m, mid;
        while(l + 1 != r) {
            mid = l + (r - l) / 2;
            // 蓝色区域
            if(matrix[mid][0] <= target) {
                l = mid;
            }
            // 红色区域
            else {
                r = mid;
            }
        }
        return l;
    }
}
```

##### 3）整个数组上二分 | O（log [m * n]）

- **思路**：由于矩阵行有序、列也有序，且列头比上一行尾大，所以可以把整个矩阵。看作一个单调递增的一维数组，然后查找 target 则等同于在该数组上进行二分即可。
- **结论**：时间，0 ms，100%，空间，41.6 mb，5.21%，时间上，由于最多需要遍历 log（m * n），所以时间复杂度为 O（log [m * n]），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int m = matrix.length, n = matrix[0].length, len = m * n;
        int l = -1, r = len, mid, i, j;
        while(l + 1 != r) {
            mid = l + (r - l) / 2;
            i = mid / n;
            j = mid % n;
            if(matrix[i][j] == target) {
                return true;
            } else if(matrix[i][j] < target) {
                l = mid;
            } else {
                r = mid;
            }
        }
        return false;
    }
}
```

#### 81. 搜索旋转排序数组 II

##### 1）变种二分查找 | O（n）

- **思路**：
  1. 思路参考《33. 搜索旋转排序数组》，但不同的地方在于，当 nums[mid] == nums[0] && nums[mid] == nums[n-1] 时，会出现 nums[mid] 等于nums[0]，却二分了左边无序部分的情况，此时只能对整个二分区间进行收窄，最坏情况下降低到 O（n）的复杂度。
  2. 还有要注意的是，由于整个二分区间的收窄，原本与 nums[0] 和 nums[n - 1] 的左右端点的比较，现在应该改为与 nums[l] 和 nums[r] 的比较，否则还是会产生二分不对的情况。
- **结论**：时间，0 ms，100%，空间，41.1 mb，67.35%，时间上，由于极限情况下，收窄的二分区间是整个数组范围，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public boolean search(int[] nums, int target) {
        int n = nums.length;
        if(n == 0) {
            return false;
        }
        if(n == 1) {
            return nums[0] == target;
        }

        // 开始二分
        int l = 0, r = n - 1, mid;
        while(l <= r) {
            mid = l + (r - l) / 2;
            if(nums[mid] == target) {
                return true;
            }
            // 当nums[mid]==nums[0] && nums[mid]==nums[n-1]时
            // 会出现nums[mid]等于nums[0], 却二分了左边无序部分的情况
            // 所以, 此时需要对整个二分区间进行收窄
            if(nums[l] == nums[mid] && nums[mid] == nums[r]) {
                l++;
                r--;
            }
            // 而只要nums[mid]不同时等于nums[0]和nums[n-1], 那么还是可以进行正常二分的
            // 从mid一刀切开, 肯定是一半有序的, 一半无序的
            // 如果mid比nums[0]小, 说明nums[0]是换过去的, [mid, n-1]有序
            else if(nums[mid] < nums[l]) {
                // [mid, n-1]有序, 则在这部分进行二分
                if(nums[mid] < target && target <= nums[r]) {
                    l = mid + 1;
                } 
                // 否则, 在[0, mid-1]上进行二分
                else {
                    r = mid - 1;
                }
            }
            // 如果mid大于等于nums[0], 说明nums[0]没有交换过, 即[0, mid]是有序的
            else {
                // [0, mid]有序, 则在这部分进行二分
                if(nums[l] <= target && target < nums[mid]) {
                    r = mid - 1;
                } 
                // 否则, 在[mid+1, n-1]上进行二分
                else {
                    l = mid + 1;
                }
            }
        }

        return false;
    }
}
```

#### 162. 寻找峰值 | medium

##### 1）极值法 | O（n）

- **思路**：从左到右遍历数组，碰到前一个大于后一个的，则认为前一个是极大值，返回前者的索引。
- **结论**：时间，0ms，100%，空间，40.7mb，62.79%，时间上，最坏情况下需要遍历 n 长的数组，所以时间复杂度为 O（n），空间上，没有使用额外变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int findPeakElement(int[] nums) {
        if(nums == null || nums.length < 2) {
            return 0;
        }

        for(int i = 1; i < nums.length; i++) {
            if(nums[i - 1] > nums[i]) {
                return i - 1;
            }
        }

        return nums.length - 1;
    }
}
```

##### 2）二分查找法 | O（logn）

- **思路**：
  1. 如果大的在右边，则二分继续往右找。
  2. 如果大的在左边，则二分继续往左找。
  3. 如果 mid 局部最大, 则返回 mid 索引。
- **结论**：时间，0ms，100%，空间，40.6 mb，88.77%，时间上，由于每次都二分，丢弃一半的元素，所以时间复杂度为 O（logn），空间上，由于采用了递归，栈深度最大为 O（logn），所以额外空间复杂度为 O（logn）。

```java
class Solution {
    public int findPeakElement(int[] nums) {
        if(nums == null || nums.length < 2) {
            return 0;
        }

        return f(nums, 0, nums.length - 1);
    }

    private int f(int[] nums, int l, int r) {
        if(l >= r) {
            return r;
        }

        int mid = l + ((r - l) / 2);

        // 如果大的在右边, 则二分继续往右找
        if(mid + 1 < nums.length && nums[mid] < nums[mid + 1]) {
            return f(nums, mid + 1, r);
        } 
        // 如果大的在左边, 则二分继续往左找
        else if(mid - 1 > -1 && nums[mid] < nums[mid - 1]) {
            return f(nums, l, mid - 1);
        } 
        // 如果mid局部最大, 则返回mid
        else {
            return mid;
        }
    }
}
```

#### 240. 搜索二维矩阵 II | medium

##### 1）递归式二分 | O（logn）

- **思路**：把二维矩阵是以 matrix[0, n-1] 为根节点的一棵二叉搜索树，然后以二分查找的方式查找搜索树即可。
- **结论**：时间，5ms，96.74%，空间，47.4mb，5.00%，效率非常高了，就这样吧~

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        if(matrix == null || matrix.length == 0 || matrix[0].length == 0) {
            return false;
        }
        return searchMatrix(matrix, target, matrix.length, matrix[0].length, 0, matrix[0].length-1);
    }

    private boolean searchMatrix(int[][] matrix, int target, int m, int n, int i, int j) {
        if(i < 0 || i >= m || j < 0 || j >= n) {
            return false;
        }

        // 等于
        if(target == matrix[i][j]) {
            return true;
        } 
        // 小于
        else if(target < matrix[i][j]) {
            return searchMatrix(matrix, target, m, n, i, j-1);
        } 
        // 大于
        else {
            return searchMatrix(matrix, target, m, n, i+1, j);
        }
    }
}
```

#### 287. 寻找重复数 | medium

##### 1）暴力解法 | O（n^2）

- **思路**：双重循环遍历，碰到第一个值相同的元素则返回。
- **结论**：执行超时，时间上，由于是双重循环遍历，所以时间复杂度为 O（n^2），空间上，由于只用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int findDuplicate(int[] nums) {
        if(nums == null || nums.length <= 1) {
            return -1;
        }

        for(int i = 0; i < nums.length; i++) {
            for(int j = i + 1; j < nums.length; j++) {
                if(nums[i] == nums[j]) {
                    return nums[i];
                }
            }
        }

        return -1;
    }
}
```

##### 2）哈希表法 | O（n）

- **思路**：从头到尾遍历数组，然后判断当前元素是否已经存在哈希表中，如果存在则代表重复了，返回即可，否则把元素加入哈希表中。
- **结论**：时间，19ms，44.22%，空间，57.9mb，5.01%，时间上，由于只遍历一次数组，所以时间复杂度为 O（n），空间上，由于使用了一个哈希表，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public int findDuplicate(int[] nums) {
        if(nums == null || nums.length <= 1) {
            return -1;
        }

        Set<Integer> set = new HashSet<>();
        for(int i = 0; i < nums.length; i++) {
            if(set.contains(nums[i])) {
                return nums[i];
            } else {
                set.add(nums[i]);
            }
        }

        return -1;
    }
}
```

##### 3）二分查找法 | O（n * logn）

- **思路**：
  1. 看到二分查找的标签，首先要去想研究数组的单调性。
  2. 根据本题题意，分析可得，在值为 1,2,...n 的数组中，如果统计小于等于 i 值的个数，由于有重复值的存在，则会导致重复值及其以后的统计小于等于的个数都会比当前 i 值还大，即存在着单调性。
  3. 另外，至于在值为 1,3...,n（重复值 3 之前的 2 缺失数组） 或者 1,2,3,..,n （重复值 3 之后的 4 缺失数组）中，上述结论同样成立，前者相当于 3 占据了 2 的位置而已，对于 1 依然成立；后者相当于 3 占据了 4 的位置而已，相对于 5,...n 依然成立。
  4. 因此，基于这种单调性，可对 count[i] 数组进行二分查找，如果发现 i 处的 count 值 > i 值，那么说明重复值就在当下或者在左前面；否则，如果发现 i 处的 count 值 <= i 值，那么说明重复值在右后面。
- **结论**：时间，25ms，38.09%，空间，58.4mb，5.01%，时间上，由于需要二分 1,2,3,...,数组花费 O（logn），每步二分又要遍历 nums 数组花费 O（n），所以时间复杂度为 O（n * logn），空间上，由于只花费了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int findDuplicate(int[] nums) {
        if(nums == null || nums.length <= 1) {
            return -1;
        }

        // l、r代表的不是原数组的下标, 而是1,2,3...n数值统计数组count[i]的下标
        int res = -1, l = 1, r = nums.length-1, mid, icount;
        while(l <= r) {
            mid = (l + r) >>> 1;
            icount = getCount(nums, mid);
            
            // 如果小于等于mid值的个数, 大于当前mid下标, 说明重复值在左边, 或者本身可能就是重复值
            if(icount > mid) {
                r = mid - 1;
                res = mid;
            } 
            // 如果小于等于mid值的个数, 小于等于当前mid下标, 说明重复值
            else {
                l = mid + 1;
            }
        }

        return res;
    }

    // 统计原生数组中小于等于target的个数
    private int getCount(int[] nums, int target) {
        int count = 0;
        for(int i = 0; i < nums.length; i++) {
            if(nums[i] <= target) {
                count++;
            }
        }
        return count;
    }
}
```

##### 4）数组转链表法 | O（n + a）

- **思路**：

  1. 这个方法很奇妙，由于数组的下标为 0,1,2,...,n-1，值是 1,2,...n，如果把 nums[0] 看作是链头节点，再把 nums[nums[0]] 看作是链头的下一个节点...，则构成一条链的数据结构，如果 0 位置的节点一旦出去就回不来了，但如果是重复元素也不会影响最终结果，因为如果 nums[0] 与 nums[1] 重复，判断 nums[1] 也是一样的。 
  2. 由于数组中存在重复元素，所以这种链的数据结构就存在者链环，因此，完全可以走《链表 - 环形链表2》的快慢指针套路了~

  ![1643716096926](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1643716096926.png)

- **结论**：时间，4ms，95.17%，空间，58.4mb，5.01%，时间上，遍历一次链表 + 再遍历一次链头到入环点的距离 a，时间复杂度为 O（n + a），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int findDuplicate(int[] nums) {
        if(nums == null || nums.length == 0) {
            return -1;
        }

        // 慢指针一次走1步, 快指针一次走2步
        int t1 = nums[0], t2 = nums[0];
        do {
            t1 = nums[t1];
            t2 = nums[nums[t2]];
        } while(t1 != t2);

        // 快慢指针第一次相遇后, 快指针回到链表开头, 然后与慢指针每次走1步, 最终会在入环点相遇
        t2 = nums[0];
        while(t1 != t2) {
            t1 = nums[t1];
            t2 = nums[t2];
        }

        return t1;
    }
}
```

#### 2300. 咒语和药水的成功对数

##### 1）排序 + 二分查找 | O（n * logn + m * logn）

- **思路**：先顺序排序 potions 数组，再查看 potions[i] >= success / spells[i] 有几个，也就是找到第一个大于等于 target 的第一个索引 r，然后用 pn - r 即可得到大于等于的数量，所以，也就是区间大于等于的题，可以使用 l + 1 != r 的二分代码模板。
- **结论**：时间，50 ms，21.21%，空间，60.4 mb，32.18%，时间上，排序花费 O（n * logn），遍历 spells 数组花费 O（m），里面的二分查找花费 O（logn），所以时间复杂度为 O（n * logn + m * logn），空间上，快速排序递归栈深度为 logn，所以额外空间复杂度为 O（logn）。

```java
class Solution {
    public int[] successfulPairs(int[] spells, int[] potions, long success) {
        if(spells == null || spells.length == 0 || potions == null || potions.length == 0) {
            return new int[0];
        }
        
        // 顺序排序potions数组
        Arrays.sort(potions);

        // 统计长度 = pn - r 
        int[] counts = new int[spells.length];
        for(int i = 0; i < spells.length; i++) {
            counts[i] = potions.length - binarySearch(potions, success / (double) spells[i]);
        }

        return counts;
    }

    // 二分查找, 查看potions[i]>=success/spells[i]有几个, 也就是找到第一个大于等于target的第一个索引
    private int binarySearch(int[] potions, double target) {
        int l = -1, r = potions.length, mid;
        while(l + 1 != r) {
            mid = l + (r - l) / 2;
            if(potions[mid] >= target) {
                r = mid;
            } else {
                l = mid;
            }
        }
        return r;
    }
}
```

### 2.6. 算法思想 - 双指针

#### 11. 盛最多水的容器 | medium

##### 1）暴力求解 | O（n^2）

- **思路**：从左到右，一次固定左边，遍历右边，遍历完移动左边到下一个位置，周而复始。
- **缺点**：没有根据数据状态进行优化，有些值明明不可能作为最大容器的还重新计算了一次，导致**超时**。

```java
class Solution {
    public int maxArea(int[] height) {
        if(height == null || height.length == 0 || height.length == 1) {
            return 0;
        }
        
        int max = Integer.MIN_VALUE;
        for(int i = 0; i < height.length - 1; i++) {
            for(int j = i + 1; j < height.length; j++) {
                max = Math.max(max, (j - i) * Math.min(height[i], height[j]));
            }
        }
        return max;   
    }
}
```

##### 2）双向指针 | O（n）

- **思路**：分析业务可知道，容器容量 = 宽度 * 高度，可从宽度或者高度方面着手。
  1. 假设两指针在左右两边，向中间靠近，这样宽度便减小了。
  2. 而如果左比右高，则说明瓶颈在右边，需要移动右指针；否则，如果右比左高，则说明瓶颈在左边，需要移动左指针。
  3. 这里可以使用贪心，让常数项比单纯的双指针法更低一些：如果宽度减小，但高度却没变大，则认为肯定不是答案，只去比较那些宽度变小，但高度变大的数据。
- **优点**：时间，6 ms，12.02%，空间，51.4%，50.63%，时间上，由于只需要遍历一遍数组，所以时间复杂度为 O（n），空间上，由于只是用了有限几个变量，所以额外空间复杂为 O（1）。

```java
class Solution {
    public int maxArea(int[] nums) {
        if(nums == null || nums.length <= 1) {
            return 0;
        }

        int l = 0, r = nums.length - 1,lmax = nums[l], rmax = nums[r];
        int min = Math.min(lmax, rmax), res = (r - l) * min;

        // 先走一步
        if(lmax <= rmax) {
            l++;
        } else {
            r--;
        }

        while(l < r) {
            lmax = Math.max(lmax, nums[l]);
            rmax = Math.max(rmax, nums[r]);
            min = Math.min(lmax, rmax);
            res = Math.max(res, (r - l) * min);

            // 先走一步
            if(lmax <= rmax) {
                l++;
            } else {
                r--;
            }
        }

        return res;
    }
}
```

#### 15. 三数求和 | medium

##### 1）暴力求解 | O（n^3）

- **思路**：3 重循环累加判断是否等于 0， 是的就加入集合，最后去重再返回。
- **缺点**：时间复杂度高，执行**超时**！
- **结论**：面试会挂！不能作为最终答案！

```java
import java.util.LinkedList;
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> res = new LinkedList<>();
        if(nums == null || nums.length < 3) {
            return res;
        }

        for(int i = 0; i < nums.length - 2; i++) {
            for(int j = i + 1; j < nums.length - 1; j++) {
                for(int k = j + 1; k < nums.length; k++) {
                    if(nums[i] + nums[j] + nums[k] == 0) {
                        List<Integer> list = new LinkedList<>();
                        list.add(nums[i]);
                        list.add(nums[j]);
                        list.add(nums[k]);
                        res.add(list);
                    }                       
                }
            }
        }

        // 去重
        distinct(res);
        return res;
    }

    // 去重
    private void distinct(List<List<Integer>> list) {
        HashMap<String, String> hasesMap = new HashMap<>();
        Iterator<List<Integer>> it = list.iterator();
        while(it.hasNext()) {
            List<Integer> elist = it.next();
            sort(elist);
            String key = "" + elist.get(0) + elist.get(1) + elist.get(2);
            if(hasesMap.containsKey(key)) {
                it.remove();
            } else {
                hasesMap.put(key, key);
            }
        }
    }

    private void sort(List<Integer> elist) {
        if(elist.get(0) < elist.get(1)) {
            swap(elist, 0, 1);
        }
        if(elist.get(0) < elist.get(2)) {
            swap(elist, 0, 2);
        }
        if(elist.get(1) < elist.get(2)) {
            swap(elist, 1, 2);
        }
    }

   private void swap(List<Integer> elist, int from, int to) {
       int tmp = elist.get(from);
       elist.set(from, elist.get(to));
       elist.set(to, tmp);
    }
}
```

##### 2）排序 + 外层去重 + 两数之和 + 哈希表 | O（n^2）

- **思路**：
  1. 通过先从小到大排序数组，相同的只取第一个做两数之和生成，避免外层重复。
  2. 由于整体是有序的，所以在做两数之和时，通过顺序 key 做重复判断，重复的将不再添加，避免内层重复。
- **缺点**：耗时严重（259ms，9.81%），因为前后重复运算了相同位置的元素，空间也浪费严重，使用了两个 map。
- **结论**：时间上，259 ms，9.81%，空间，47.8 mb，5.10%，时间上，外层遍历花费 O（n），内层双指针花费 O（n），所以时间复杂度为 O（n^2），空间上，由于使用了 1 张 n 长的哈希表，所以额外空间复杂度为 O（n）。由于里层只是用了哈希表做物理去重，没做逻辑去重，所以时间上和空间上，都还能继续优化~

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        if(nums == null || nums.length < 3) {
            return res;
        }

        // 从小到大排序
        Arrays.sort(nums);

        HashMap<String, String> distinctMap = new HashMap<>();
        for(int i = 0; i < nums.length; i++) {
            // 相同的跳过重复解, 取第一个即可
            if(i > 0 && nums[i-1] == nums[i]) {
                continue;
            }

            twoSum(nums, i + 1, res, nums[i], distinctMap);
        }

        return res;
    }

    private void twoSum(int[] nums, int start, List<List<Integer>> res, int value, HashMap<String, String> distinctMap) {
        if(nums == null || nums.length - start < 2) {
            return;
        }
        
        Integer tmp = null, target =  0 - value;
        HashMap<Integer, Integer> map = new HashMap<>();
        for(int i = start; i < nums.length; i++) {
            tmp = map.get(target - nums[i]);
            if(tmp != null) {
                String key = "" + value + nums[i] + nums[tmp];
                if(!distinctMap.containsKey(key)) {
                    List<Integer> list = new ArrayList<>(3);
                    list.add(value);
                    list.add(nums[i]);
                    list.add(nums[tmp]);
                    res.add(list);
                    distinctMap.put(key, key);
                }
            }
            map.put(nums[i], i);
        }
    }
}
```

##### 3）排序 + 外层去重 + 两数之和 + 双向指针 + 贪心去重 | O（n^2）

- **思路**：
  1. 通过先从小到大排序数组，相同的只取第一个做两数之和生成，避免外层重复。
  2. 由于整体是有序的，所以在做两数之和时，判断完后再判断是否重复，重复则跳过，避免内层重复。
- **优点**：通过双向指针 + 贪心去重，重复的元素只会计算一次，也不需要哈希表判重，大大节省了时间（22ms，51.86%）和空间。
- **结论**：时间上，22 ms，48.93%，空间，46.1 mb，5.10%，时间上，外层遍历花费 O（n），内层双指针花费 O（n），所以时间复杂度为 O（n^2），空间上，由于只是使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        if(nums == null || nums.length == 0) {
            return new ArrayList<>();
        }

        // 先顺序排序
        Arrays.sort(nums);

        // 注意去重~
        List<List<Integer>> res = new ArrayList<>();
        for(int i = 0; i < nums.length; i++) {
            if(i - 1 > -1 && nums[i] == nums[i - 1]) {
                continue;
            }

            // 由于是有序的, 所以从左开始确定第1个数后, 后两个数就不用从0开始判断了
            // 这是为了防止重复, 因为之前的外层遍历就操作过了
            twoSum(nums, res, i, 0 - nums[i]);   
        }

        return res;
    }
    
    private void twoSum(int[] nums, List<List<Integer>> res, int firstIdx, int target) {
        int l = firstIdx + 1, r = nums.length - 1, sum;
        while(l < r) {
            sum = nums[l] + nums[r];

            // 符合的则加入
            if(sum == target) {
                List<Integer> list = new ArrayList<>();
                System.out.println(String.format("firstIdx: %s, l: %s, r: %s", firstIdx, l, r));

                list.add(nums[firstIdx]);
                list.add(nums[l]);
                list.add(nums[r]);
                res.add(list);

                // 内层l也要进行去重判断
                while(l + 1 < nums.length && nums[l + 1] == nums[l]) {
                    l++;
                }

               // 做了去重后, 由于l已经被上面确定过了, 所以需要换一个l进行确定
                l++;

                // 内层r也要进行去重判断: 注意这里是至少要比firstIdx的大, 而不是像外层循环那样比0大
                while(r - 1 > firstIdx && nums[r - 1] == nums[r]) {
                    r--;
                }

                // 做了去重后, 由于r已经被上面确定过了, 所以需要换一个r进行确定
                r--;
            } 
            // 比结果偏小, 则l++
            else if(sum < target) {
                l++;
            }
            // 比结果偏大, 则r--
            else {
                r--;
            }
        }
    }
}
```

#### 16. 最接近的三数之和 | medium

##### 1）暴力解法 | O（n^3）

- **思路**：遍历 i、j、k，三重循环，然后比较出三者之和到 target 的距离时，该和就是答案。
- **结论**：时间，259 ms，8.82%，空间，41 mb，38.49%，时间上，由于需要三重循环，所以时间复杂度为 O（n^3），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int threeSumClosest(int[] nums, int target) {
        if(nums == null || nums.length < 3) {
            return 0;
        }

        int res = Integer.MAX_VALUE, tmp, diff = Integer.MAX_VALUE, difftmp; 
        for(int i = 0; i < nums.length; i++) {
            for(int j = i + 1; j < nums.length; j++) {
                for(int k = j + 1; k < nums.length; k++) {
                    tmp = nums[i] + nums[j] + nums[k];
                    difftmp = Math.abs(tmp - target);
                    if(difftmp < diff) {
                        diff = difftmp;
                        res = tmp;
                    }
                }
            }
        }

        return res;
    }
}
```

##### 2）排序 + 双向指针 O（n^2）

- **思路**：
  1. 类似于三数求和那样，先对数组进行顺序排序，然后依次遍历确定第一个数字，剩余两个数字则利用数组有序性进行判断。
  2. 如果 tmp + 剩余两者之和比 target 要大，说明大了，那么就需要 r-- 进行调小剩余两者之和。
  3. 否则，如果比 target 要小，说明小了，那么就需要 l++ 进行调大剩余两者之和。
  4. 其中，每次 l 或者 r 移动之前，都做一次 res 和 diff 的判断，比之前小的，则更新为 res，这样整个流程结束时，res 就是答案了。
- **结论**：时间，5 ms，97.40%，空间，40.8 mb，68.27%，时间上，由于需要两重循环判断，所以时间复杂度为 O（n^2），空间上，由于快速排序的栈深度为 logn，所以额外空间复杂度为 O（logn）。

```java
class Solution {
    public int threeSumClosest(int[] nums, int target) {
        if(nums == null || nums.length < 3) {
            return 0;
        }

        // 先顺序排序
        Arrays.sort(nums);

        // 再利用顺序性, 做双向指针
        int res = Integer.MAX_VALUE, diff = res, tmp, diffTmp, l, r;
        for(int i = 0; i < nums.length; i++) {
            l = i + 1;
            r = nums.length - 1;

            while(l < r) {
                tmp = nums[l] + nums[r] + nums[i];
                diffTmp = Math.abs(tmp - target);

                if(diffTmp < diff) {
                    diff = diffTmp;
                    res = tmp;
                }

                // 利用顺序性, 做双向指针
                if(tmp == target) {
                    return target;
                } else if(tmp < target) {
                    l++;
                } else {
                    r--;
                }
            }
        }

        return res;
    }
}
```

#### 18. 四数之和 | medium

##### 1）暴力解法 | O（n^4）

- **思路**：先顺序排序，再利用有序性，跳过重复计算的循环，经过 4 次循环，即可计算出这四个数字，之和是否等于 target 了，同时还保证了去重。
- **结论**：时间，395 ms，5.02%，空间，41.7 mb，63.85%，时间上，由于需要遍历 4 次，所以时间复杂度为 O（n^4），空间上，由于快速排序的栈深度为 logn，所以额外空间复杂度为 O（logn）。

```java
class Solution {
    public List<List<Integer>> fourSum(int[] nums, int target) {
        if(nums == null || nums.length < 4) {
            return new ArrayList<>();
        }

        // 先顺序排序
        Arrays.sort(nums);

        List<List<Integer>> res = new ArrayList<>();
        for(int i = 0; i < nums.length; i++) {
            // 外层循环跳过重复i
            if(i - 1 > -1 && nums[i - 1] == nums[i]) {
                continue;
            }
            
            for(int j = i + 1; j < nums.length; j++) {
                // 外层循环跳过重复j
                if(j - 1 > i && nums[j - 1] == nums[j]) {
                    continue;
                }
                
                for(int k = j + 1; k < nums.length; k++) {
                    // 外层循环跳过重复k
                    if(k - 1 > j && nums[k - 1] == nums[k]) {
                        continue;
                    }
                    
                    for(int m = k + 1; m < nums.length; m++) {
                        // 外层循环跳过重复m
                        if(m - 1 > k && nums[m - 1] == nums[m]) {
                            continue;
                        }
                        
                        if(nums[i] + nums[j] + nums[k] + nums[m] == target) {
                            List<Integer> tmp = new ArrayList<>();
                            tmp.add(nums[i]);
                            tmp.add(nums[j]);
                            tmp.add(nums[k]);
                            tmp.add(nums[m]);
                            res.add(tmp);
                        }
                    }
                }
            }
        }

        return res;
    }
}
```

##### 2）排序 + 双向指针 + 贪心去重 | O（n^3）

- **思路**：
  1. 外面两重循环与 1. 暴力解法的一样，其目的是先通过枚举，确认好两个数。
  2. 里面的两个数，则跟《15. 三数之和》的双指针一样，通过 l、r 前后指针，利用有序性，分情况讨论：
     - 1）如果四数之和，等于 target，则加入结果集，然后判断是否有重复。
     - 2）如果四数之和，大于 target，说明离目标偏大，需要 r--，减小 nums[l] + nums[r] 的累加和。
     - 3）如果四数之和，小于 target，说明离目标偏小，需要 l++，增大 nums[l] + nums[r] 的累加和。
- **结论**：时间，13 ms，64.41%，空间，42 mb，16.50%，时间上，由于需要遍历 3 次数组，所以时间复杂度为 O（n^3），空间上，由于快速排序的栈深度为 O（logn），所以额外空间复杂度为 O（logn）。

```java
class Solution {
    public List<List<Integer>> fourSum(int[] nums, int target) {
        if(nums == null || nums.length < 4) {
            return new ArrayList<>();
        }

        // 先顺序排序
        Arrays.sort(nums);
        
        int l, r, tmp;
        List<List<Integer>> res = new ArrayList<>();
        for(int i = 0; i < nums.length - 3; i++) {
            // 外层循环跳过重复i
            if(i - 1 > -1 && nums[i - 1] == nums[i]) {
                continue;
            }

            for(int j = i + 1; j < nums.length - 2; j++) {
                // 外层循环跳过重复j
                if(j - 1 > i && nums[j - 1] == nums[j]) {
                    continue;
                }
                
                l = j + 1;
                r = nums.length - 1;

                while(l < r) {
                    tmp = nums[i] + nums[j] + nums[l] + nums[r];

                    // 等于target则加入结果集, 然后判断是否有重复
                    if(tmp == target) {
                        List<Integer> list = new ArrayList<>();
                        list.add(nums[i]);
                        list.add(nums[j]);
                        list.add(nums[l]);
                        list.add(nums[r]);
                        res.add(list);

                        // 重复的nums[r]不用再计算
                        while(l < r && nums[r - 1] == nums[r]) {
                            r--;
                        }

                        // r--放while的后面, 保证是先跳过重复的数字, 再移到下一个合理的位置
                        r--;
                        
                        // 重复的nums[l]不用再计算
                        while(l < r && nums[l] == nums[l + 1]) {
                            l++;
                        }

                        // l++放while的后面, 保证是先跳过重复的数字, 再移到下一个合理的位置
                        l++;       
                    } 
                    // 大于target, 说明离目标偏大, 需要r--, 减小nums[l]+nums[r]的累加和
                    else if(tmp > target) {
                        r--; 
                    } 
                    // 小于target, 说明离目标偏小, 需要l++, 增大nums[l]+nums[r]的累加和
                    else {
                        l++;
                    }
                }
            }
        }      
        
        return res;
    }
}
```

#### 26. 删除有序数组中的重复项 | easy

##### 1）快慢指针法 | O（n）

- **思路**：快指针控制访问的元素，慢指针控制重复元素的窗口大小，如果重复次数超过了 maxRepat 次，那么就增大 slow 指针，增大窗口大小，以放入其他元素，最后返回窗口大小即可。
- **结论**：时间，0 ms，100%，空间，43.2 mb，28.61%，时间上，由于需要遍历一次数组，所以时间复杂度为 O（n），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        // 最大允许的重复次数
        int maxRepeat = 1;

        // 由于最大能够重复maxRepeat次, 所以slow可以直接从maxRepeat-1下标开始
        // 而由于如果fast==slow会跳过、再+1, 所以fast可以直接从slow+1下标开始
        int n = nums.length, slow = maxRepeat - 1, fast = maxRepeat;

        // fast越界则结束
        while(fast < n) {
            // 确保每个窗口内, 最大重复的次数不超过maxRepeat次
            if(nums[fast] != nums[slow - maxRepeat + 1]) {
                slow++;
                if(fast != slow) {
                    nums[slow] = nums[fast];
                }
            }

            // 继续考察下一个元素
            fast++;
        }

        // 返回结束时, 窗口的长度
        return slow + 1;
    }
}
```

#### 27. 移除元素 | easy

##### 1）快慢指针法 | O（2 * n）

- **思路**：
  1. 同《26. 删除有序数组中的重复项》的快慢指针法，同样为了去重，可以使用一个 right 指针代表下次碰到不是目标数字时，其需要存储的位置。
  2. 如果不等于 val 的数字，那么就把它放到 right 正确的索引上，然后 right、cur 继续往前 +1。
  3. 而如果碰到的是等于 val 的数字，则 right 保持不变，cur 当前指针继续往前推进判断即可。
- **结论**：时间，0 ms，100%，空间，40.1 mb，25.16%，时间上，最坏情况下，即数组中没有一个数字等于 val 时，需要遍历数组 2 次，所以时间复杂度为 O（2 * n），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int removeElement(int[] nums, int val) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        int right = 0, cur = right;
        while(cur < nums.length) {
            if(nums[cur] != val) {
                nums[right++] = nums[cur];
            }
            cur++;
        }

        return right;
    }
}
```

##### 2）前后指针法 | O（n + 1）

- **思路**：类似于荷兰国旗和快速排序的 partition 过程，左边可以经过划分为不等于 val 的部分，右边可以划分为等于 val 的部分，其中，由于要求不等于 val 的数量，所以终止条件为 l <= r，便于计算出 l 所走的步数作为该数量。
- **结论**：时间，0 ms，100%，空间，39.8 mb，61.46%，时间上，由于 l 和 r 终止后再走一步就终止循环了，所以时间复杂度为 O（n + 1），对比 1 的思路，在最坏情况下，能减少一次数组的遍历，空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int removeElement(int[] nums, int val) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        // 类似于荷兰国旗的partition过程
        int l = 0, r = nums.length - 1;
        while(l <= r) {
            // 左边都是不等于val的
            if(nums[l] != val) {
                l++;
            }
            // 右边都是等于val的
            else {
                if(nums[r] != val) {
                    swap(nums, l, r);
                }
                r--;
            }
        }

        return l;
    }

    private void swap(int[] nums, int from, int to) {
        int tmp = nums[to];
        nums[to] = nums[from];
        nums[from] = tmp;
    }
}
```

#### 42. 接雨水 | hard

##### 1）双向指针法 | O（n）

- **思路**：
  1. 利用当前位置与左右边界最小值比较，如果小于最小边界，则认为当前格子有 min - i 的水。
  2. 首先选择左右指针 l 和 r 作为边界，然后从**较小边界**的旁边 i 出发，直到左右边界重合，或者碰到左右边界为止（由于 l 或者 r 每次都往里面缩，而 i 又是比边界更往里面，所以除了特殊数据，并不会出现一开始 i 等于边界的情况）。
  3. 如果 i 比左右两边界都小，则认为边界有效，则用**较小边界的值**结算当前格子的水 min - i，否则每次 i 碰到更大的边界，则替换较小的边界，然后重新选举较小边界的索引，从而得出下一步 i 的位置，周而复始，直到循环结束，得到水的总和即是答案。
- **结论**：时间，1 ms，73.96 %，空间，42.8 mb，5.08 %，时间上，由于只需要遍历一遍数组，所以时间复杂度为 O（n），空间上，由于只是用了有限几个变量，所以额外空间复杂为 O（1）。

```java
class Solution {
    public int trap(int[] nums) {
        if(nums == null || nums.length <= 1) {
            return 0;
        }

        int res = 0, l = 0, r = nums.length - 1;
        int lmax = nums[l], rmax = nums[r], min;

        // 先走一步
        int tmp;
        if(lmax <= rmax) {
            min = lmax;
            l++;
            tmp = l;
        } else {
            min = rmax;
            r--;
            tmp = r;
        }

        // 开始遍历
        while(l < r) {
            res += Math.max(0, min - nums[tmp]);
            lmax = Math.max(lmax, nums[l]);
            rmax = Math.max(rmax, nums[r]);
            
            if(lmax <= rmax) {
                min = lmax;
                l++;
                tmp = l;
            } else {
                min = rmax;
                r--;
                tmp = r;
            }
        }

        return res;
    }
}
```

#### 80. 删除有序数组中的重复项 II | medium

##### 1）快慢指针法 | O（n）

- **思路**：思路参考《26. 删除有序数组中的重复项》，只需要把 maxRepeat 次数改为 2 即可。
- **结论**：时间，0 ms，100%，空间，41.8 mb，24.32%，时间上，由于需要遍历一次数组，所以时间复杂度为 O（n），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        // 最大允许的重复次数
        int maxRepeat = 2;

        // 由于最大能够重复maxRepeat次, 所以slow可以直接从maxRepeat-1下标开始
        // 而由于如果fast==slow会跳过、再+1, 所以fast可以直接从slow+1下标开始
        int n = nums.length, slow = maxRepeat - 1, fast = maxRepeat;

        // fast越界则结束
        while(fast < n) {
            // 确保每个窗口内, 最大重复的次数不超过maxRepeat次
            if(nums[fast] != nums[slow - maxRepeat + 1]) {
                slow++;
                if(fast != slow) {
                    nums[slow] = nums[fast];
                }
            }

            // 继续考察下一个元素
            fast++;
        }

        // 返回结束时, 窗口的长度
        return slow + 1;
    }
}
```

### 2.7. 算法思想 - 极值法

#### 31. 下一个排列 | medium

要求空间复杂度 O（1），即原地计算。

##### 1）极值解法 | O（n * nlogn）

- **思路**：从后往左遍历，交换次小值和次大值，然后对次小值后面的位置进行顺序排序。

  1. 从后往左遍历，保证找到的次小值是最右的次小值，也就是最右的下坡（相同时的最右一个）。
  2. 找到次小值后，再次从后往左遍历，找到最左的次大值，也就是最左的上坡（相同时不论左右）。
  3. 如果找到次小值，那么必定有次大值（因为存在下坡），接着交换次小值和次大值，可以使得序列整体在 i 位置提高一些，但为了保证下一个序列是相邻的，所以只能提高一点，因此，还需要对 i 位置以后的序列，进行顺序排列，以保证最小。
  4. 如果没找到次小值，那么说明这个序列本身是降序排列的，所以，下一个序列应该为升序序列，因此对整个序列进行顺序排列即可。

  ![1641719239061](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1641719239061.png)

- **结论**：时间，0 m，100%，空间，41.6 mb，50.34%，时间上，由于只需遍历一次数组，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public void nextPermutation(int[] nums) {
        if(nums == null || nums.length == 0) {
            return;
        }
        if(nums.length == 1) {
            return;
        }
        if(nums.length == 2) {
            swap(nums, 0, 1);
            return;
        }

        // 先从后往左, 寻找峰值
        int n = nums.length, peak = n - 1;
        while(peak - 1 > -1 && nums[peak - 1] >= nums[peak]) {
            peak--;
        }

        // 如果峰值是第一位, 说明当前序列是最后一组序列, 则反转再返回即可
        if(peak == 0) {
            reverseArr(nums, 0, n - 1);
        } 
        // 如果峰值不是第一位
        else {
            // 那么先取往左的、第一个比峰值小的数
            int i = peak - 1;
            
            // 然后从右取左取、第一个比i大的数, 必定存在! 因为至少存在一个峰值
            int j = n - 1;
            while(j > i && nums[i] >= nums[j]) {
                j--;
            }

            // 然后交换
            swap(nums, i, j);
        
            // 最后, 由于峰值处到后面, 肯定是降序的, 所以可以反转成为顺序的, 以保证右边最小
            reverseArr(nums, peak, n - 1);
        }
    }

    private void reverseArr(int[] nums, int start, int end) {
        while(start < end) {
            swap(nums, start++, end--);
        }
    }

    private void swap(int[] nums, int from, int to) {
        int tmp = nums[to];
        nums[to] = nums[from];
        nums[from] = tmp;
    }
}
```

#### 581. 最短无序连续子数组 | medium

##### 1）暴力解法 | O（n * logn + 2n）

- **思路**：先复制出来一个数组，然后排好序，接着左右指针遍历原数组，如果对比排序数组发现当前值已经位于最终位置时，则 l++ 或者 r--，直到 l 和 r 重逢或者发现位置不合适，然后返回之间的长度，但要注意的是，如果 l 和 r 最终重逢了，说明原数组本身就有序，此时应该返回 0，而不是计算长度的结果 1。
- **结论**：时间，7ms，20.76%，空间，42.7mb，5.01%，时间上，由于复制数组需要 O（n），排序需要 O（n * logn），左右指针移动需要 O（n），所以时间复杂度为 O（n * logn + 2n），空间上，由于额外建立了一个数组 n，排序需要的递归深度 logn，所以额外空间复杂度为 O（n + logn）。

```java
class Solution {
    public int findUnsortedSubarray(int[] nums) {
        if(nums == null || nums.length <= 1) {
            return 0;
        }

        int[] sorteds = new int[nums.length];
        for(int i = 0; i < nums.length; i++) {
            sorteds[i] = nums[i];
        }
        Arrays.sort(sorteds);

        int l = 0, r = nums.length-1;
        while(l < r) {
            if(nums[l] == sorteds[l]) {
                l++;
                if(nums[r] == sorteds[r]) {
                    r--;
                }
                continue;
            }
            if(nums[r] == sorteds[r]) {
                r--;
                if(nums[l] == sorteds[l]) {
                    l++;
                }
                continue;
            }
            break;
        }

        return r == l? 0 : r - l + 1;
    }
}
```

##### 2）不等式法 | O（2n）

- **思路**：研究数据状态发现，最短无序子数组的右边界，必定满足右边界 nums[i] <  nums[i-1]，同理其左边界也必定满足，左边界 nums[j] > nums[j+1]，因此，可以先遍历找出其右边界，再找出其左边界，然后获取其长度即可。
- **结论**：时间，0ms，100%，空间，41.5mb，8.90%，时间上，由于需要遍历两次数组，所以时间复杂度为 O（2n），空间上，由于只有使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int findUnsortedSubarray(int[] nums) {
        if(nums == null || nums.length <= 1) {
            return 0;
        }

        // 右边第一个不符合 r >= l 不等式的下标为右边界
        int lmax = Integer.MIN_VALUE, rres = 0;
        for(int i = 0; i < nums.length; i++) {
            if(nums[i] >= lmax) {
                lmax = nums[i];
            } else {
                rres = i;
            }
        }

        // 左边第一个不符合 l <= r 不等式的下标为左边界
        int rmin = Integer.MAX_VALUE, lres = 0;
        for(int i = nums.length - 1; i > -1; i--) {
            if(nums[i] <= rmin) {
                rmin = nums[i];
            } else {
                lres = i;
            }
        }
        
        return lres >= rres? 0 : rres - lres + 1;
    }
}
```

### 2.8. 算法思想 - 编辑距离

#### 2267. 第 292 场周赛 - 检查是否有合法括号字符串路径 | hard

##### 1）暴力递归 | O（[m * n * (m + n - 1)]^2）

- **思路**：按规则往下、往右遍历二维表，每遍历到一个 '('，则 diff + 1，否则代表为 ')'，则 diff - 1，中间如果判断到 diff < 0、或者剩余的格子数（m - 1 - i）+ (n - 1 - j) 不能够扣完 diff、或者 diff 遍历到最后一个格子时仍未为 0，则返回 false，否则有任意成功路径，则返回 true。
- **结论**：执行超时，时间上，由于 diff 最多等于一半的路径格子数，即等于（(m - 1) + (n - 1) + 1) / 2，且递归中调用了两次 f（i，j，diff）函数，所以可以组合两次，即时间复杂度为 O（[m * n * (m + n - 1)]^2），空间上，同理 f 函数也会组合两次，所以额外空间复杂度为  O（[m * n * (m + n - 1)]^2）。

```java
class Solution {
    public boolean hasValidPath(char[][] grid) {
        if(grid == null || grid.length == 0 || grid[0] == null || grid[0].length == 0) {
            return false;
        }

        int m = grid.length;
        int n = grid[0].length;
        if(((m - 1) + (n - 1) + 1) % 2 != 0) {
            return false;
        }

        return f(grid, m, n, 0, 0, 0);
    }

    // i, j时的字符串是否合法, diff < 0, 表示右括号多, diff > 0, 表示左括号多
    private boolean f(char[][] grid, int m, int n, int i, int j, int diff) {
        diff = grid[i][j] == '('? diff + 1 : diff - 1;
        if(diff < 0 || diff > (m - 1 - i) + (n - 1 - j)) {
            return false;
        }

        // 下
        if(i + 1 < m) {
            if(f(grid, m, n, i + 1, j, diff)) {
                return true;
            }
        }

        // 右
        if(j + 1 < n) {
            if(f(grid, m, n, i, j + 1, diff)) {
                return true;
            }
        }

        return (i == m - 1 && j == n - 1) && diff == 0;
    }
}
```

##### 2）记忆化搜索 |  O（m * n * (m + n - 1)）

- **思路**：在暴力递归的基础上，增加 dp 三维表缓存，如果缓存中存在，则从缓存中获取，否则根据规则设置缓存，再返回结果到上一层。
- **结论**：时间，9 ms，63.13%，空间，59.9 mb，19.32%，时间上，由于 f 函数只需要被组合一次，所以时间复杂度为 O（m * n * (m + n - 1)），空间上，同理，f 函数也只需要被组合一次，所以额外复杂度为 O（ 2 * m * n * (m + n - 1)）。

```java
class Solution {
    public boolean hasValidPath(char[][] grid) {
        if(grid == null || grid.length == 0 || grid[0] == null || grid[0].length == 0) {
            return false;
        }

        int m = grid.length;
        int n = grid[0].length;
        int all = ((m - 1) + (n - 1) + 1);
        if(all % 2 != 0) {
            return false;
        }

        Boolean[][][] dp = new Boolean[m][n][all];
        return f(grid, dp, m, n, 0, 0, 0);
    }

    // i, j时的字符串是否合法, diff < 0, 表示右括号多, diff > 0, 表示左括号多
    private boolean f(char[][] grid, Boolean[][][] dp, int m, int n, int i, int j, int diff) {
        diff = grid[i][j] == '('? diff + 1 : diff - 1;
        if(diff < 0 || diff > (m - 1 - i) + (n - 1 - j)) {
            return false;
        }
        if(dp[i][j][diff] != null) {
            return dp[i][j][diff];
        }

        // 下
        if(i + 1 < m) {
            if(f(grid, dp, m, n, i + 1, j, diff)) {
                dp[i][j][diff] = true;
                return dp[i][j][diff];
            }
        }

        // 右
        if(j + 1 < n) {
            if(f(grid, dp, m, n, i, j + 1, diff)) {
                dp[i][j][diff] = true;
                return dp[i][j][diff];
            }
        }

        dp[i][j][diff] = (i == m - 1 && j == n - 1) && diff == 0;
        return dp[i][j][diff];
    }
}
```

##### 3）严格表结构优化 | O（m * n * (m + n - 1)）

- **思路**：在记忆化搜索的基础上，经过研究表结构的依赖关系，发现是依赖以及上一层 i+1 和 j+1 位置，所以需要从上到下、从后往前初始化 dp 表。
- **结论**：执行超时，时间上也是为  O（m * n * (m + n - 1)），但可能由于很多地方没有记忆化那样跳过去，导致了执行超时，空间上，由于省去了递归栈的使用，所以额外空间复杂度为仅仅为  O（m * n * (m + n - 1)）。

```java
class Solution {
    public boolean hasValidPath(char[][] grid) {
        if(grid == null || grid.length == 0 || grid[0] == null || grid[0].length == 0) {
            return false;
        }

        int m = grid.length;
        int n = grid[0].length;
        int all = ((m - 1) + (n - 1) + 1);
        if(all % 2 != 0) {
            return false;
        }

        boolean[][][] dp = new boolean[m][n][all];
        for(int diff = all - 2; diff > -1; diff--) {
            for(int i = m - 1; i > -1; i--) {
                for(int j = n - 1; j > -1; j--) {
                    diff = grid[i][j] == '('? diff + 1 : diff - 1;
                    if(diff < 0 || diff > (m - 1 - i) + (n - 1 - j)) {
                        continue;
                    }
                    if(dp[i][j][diff]) {
                        continue;
                    }

                    // 下
                    if(i + 1 < m && dp[i + 1][j][diff]) {
                        dp[i][j][diff] = true;
                        continue;
                    }

                    // 右
                    if(j + 1 < n && dp[i][j + 1][diff]) {
                        dp[i][j][diff] = true;
                        continue;
                    }

                    dp[i][j][diff] = (i == m - 1 && j == n - 1) && diff == 0;
                }
            }
        }

        return dp[0][0][0];
    }
}
```

#### 2272. 第 78 双周赛 - 最大波动的子字符串 | hard

##### 1）暴力解法 | O（3 * n^3）

- **思路**：遍历每个子字符串，然后统计各自字符出现的次数，得到最小次数和最大次数，相减得到每个子字符串的波动值，取最大的返回即可。
- **结论**：执行超时，时间上，由于遍历每个子字符串花费 O（n^2），记子字符串的长度为 m，在做子字符串截取时最大花费 O（n），统计最大花费 O（n），取最小、最大次数最多花费 O（n），所以时间复杂度为 O（3 * n^3），空间上，由于使用了一张哈希表，其长度最大为 n，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public int largestVariance(String s) {
        if(s == null || s.length() == 0) {
            return 0;
        }

        int res = 0, min, max;
        Map<Character, Integer> charCountMap = new HashMap<>(); 
        for(int len = 1; len <= s.length(); len++) {
            for(int i = 0; i < len; i++) {
                char[] chars = s.substring(i, len).toCharArray();

                charCountMap.clear();
                for(int j = 0; j < chars.length; j++) {
                    charCountMap.put(chars[j], charCountMap.getOrDefault(chars[j], 0) + 1);
                }

                min = Integer.MAX_VALUE;
                max = Integer.MIN_VALUE;
                for(Integer value : charCountMap.values()) {
                    min = Math.min(min, value);
                    max = Math.max(max, value);
                }

                res = Math.max(res, max - min);
            }
        }

        return res;
    }
}
```

##### 2）暴力递归 | O（26 * 26 * n^2）

- **思路**：

  1. 本题可以看作是《动态规划 - 53. 最大子数组和》的变种，变种的地方在于：可以认为碰到次数多的字符 i，则认为是 1，碰到次数少的字符 j，则认为是 -1，碰到其他字符，则认为是 0，然后累加从左到右的值，则可以得到最大和，也就是编辑距离的最大值，刚好也是本题的波动值。

     ```java
     // 最大子数组核心代码
     int pre = nums[0];
     int max = Integer.MIN_VALUE;
     for(int i = 1; i < nums.length; i++) {
     	 pre = Math.max(pre + nus[end], nums[end]);
     	 max = Math.max(max, pre);
     }
     
     // 本题核心代码
     int max = Integer.MIN_VALUE;
     for(int a = 'a'; a <= 'z'; a++) {
     	for(int b = 'a'; b <= 'z'; b++) {
     		int pre = chars[0] == a? 1 : chars[0] == b? -1 : 0;
     		for(int i = 1; i < chars.length; i++) {
     			if(chars[i] == a) {
     				pre = Math.max(pre + 1, 1);
     			} else if(chars[i] == b) {
     				pre = Math.max(pre - 1, -1);
     			} else {
     				pre = Math.max(pre, 0);
     			}
     			max = Math.max(max, pre);
     		}
     	}
     }
     ```

  2. 但是，上面的核心代码并不能满足要求，原因是没能解决 i 字符多次出现，导致的编辑距离一直为正数的问题，同理 j 字符多次出现，也会出现编辑距离一直为负数的问题。

  3. 解决方法就是，设计一个 diff，表示 i 字符多于 j 字符出现的次数，默认为 0，碰到 i 字符，diff++，碰到 j 字符 diff--，碰到其他字符，则保持不变。

  4. 同时，还设计一个 diffWithJ，表示 i 字符多于 j 字符出现的次数，且必须出现 j 字符，默认为 -s.length()，代表 j 还没出现：

     - 1）碰到 i 字符，diffWithJ++，当全部为 i 时，diffWithJ 为 0，从而解决编辑距一直为正的问题。
     - 2）碰到 j 字符，diffWithJ 等于 diff，当全为 j 时，diffWithJ 等于 -n，不过没有关系，Math.max 并不会把他作为结果。
     - 3）注意，用于采用的是 最大子数组和 的 dp 思路，即依赖 f（end - 1）的值，diff 设置后，会作为结果返回给上层用的，表示的是大数 i 字符多于 j 字符出现的次数，如果 diff 为负数了，还不如取 0，认为从取下一个开始的子串，放弃前面的子串，所以还需要与 0 做个最大值比较，负数则更新为 0，返回给下一个 end + 1的子串判断。

  5. 因此，设计一个 f（end，i，j）函数，代表获取以 end 结尾的、i 作为次数多的字符、j 作为次数少的字符的 diff 和 diffWithJ。

  6. 然后，外层则是通过 26 * 26 的遍历次数，枚举任意的 i、j 字符组合，然后从左到右遍历字符数组，类似于最大子数组和那样，将 end 传入 f 函数，同时还带上 i、j 状态。

  7. 最后，比较每次 f 函数返回的 diffWithJ，取最大值作为最大的子串波动值即可。

- **结论**：执行超时，时间上，枚举 i、j 字符花费 O（26 * 26），dp 花费 O（n^2），所以时间复杂度为 O（26 * 26 * n^2），空间上，由于递归深度最大为 n，所以额外空间复杂度为 O（n）。

```java
class Solution {

    public int largestVariance(String s) {
        if(s == null || s.length() == 0) {
            return 0;
        }
        
        Info info;
        int max = Integer.MIN_VALUE;
        char[] chars = s.toCharArray();

        // 枚举最多字符i和最少字符j
        for(int i = 'a'; i <= 'z'; i++) {
            for(int j = 'a'; j <= 'z'; j++) {
                if(i == j) {
                    continue;
                }

                // 枚举每个子串
                for(int end = 0; end < chars.length; end++) {
                    info = f(chars, end, i, j);
                    
                    // 结算每个子串的波动值
                    max = Math.max(max, info.diffWithJ);
                }
            }
        }

        return max;
    }

    class Info {
        int diff;
        int diffWithJ;

        Info(int diff, int diffWithJ) {
            this.diff = diff;
            this.diffWithJ = diffWithJ;
        }
    }

    private Info f(char[] chars, int end, int i, int j) {
        if(end < 0) {
            return new Info(0, -chars.length);
        }

        Info info = f(chars, end - 1, i, j);
        if(chars[end] == i) {
            info.diff++;

            // 没有j时, diffWithJ就是整个八经的波动值
            // 全为i时, diffWithJ为0
            info.diffWithJ++;
        } else if(chars[end] == j) {
            info.diff--;

            // 有出现j时, diffWithJ完全等于diff, 即本次的波动值
            // 全为j时, diffWithJ为-n, 但没关系, i、j反过来尝试时, 下面的max就为0了 
            info.diffWithJ = info.diff;

            // 全为j时, diff为-n, 此时下一个子串需与0取最大值, 认为不应该从负的diff出发
            info.diff = Math.max(info.diff, 0);
        }

        return info;
    }
}
```

##### 3）记忆化搜索 |  O（26 * 26 * n）

- **思路**：在暴力递归的基础上，增加 dp 三维表，如果缓存中存在，则从缓存中返回，否则先设置结果到缓存，再返回给上一层。
- **结论**：时间，1358 ms，5.14%，空间，123.5 mb，5.14%，时间上，由于存在缓存，避免了多余的递归，但时间复杂度为  O（26 * 26 * n），空间上，由于递归深度最大为 n，三维表缓存为 26 * 26 * n，所以额外空间复杂度为 O（n + 26 * 26 * n）。

```java
class Solution {

    public int largestVariance(String s) {
        if(s == null || s.length() == 0) {
            return 0;
        }
        
        Info info;
        int max = Integer.MIN_VALUE;
        char[] chars = s.toCharArray();
        Info[][][] dp = new Info[chars.length][26][26];

        // 枚举最多字符i和最少字符j
        for(int i = 'a'; i <= 'z'; i++) {
            for(int j = 'a'; j <= 'z'; j++) {
                if(i == j) {
                    continue;
                }

                // 枚举每个子串
                for(int end = 0; end < chars.length; end++) {
                    info = f(chars, dp, end, i, j);
                    
                    // 结算每个子串的波动值
                    max = Math.max(max, info.diffWithJ);
                }
            }
        }

        return max;
    }

    class Info {
        int diff;
        int diffWithJ;

        Info(int diff, int diffWithJ) {
            this.diff = diff;
            this.diffWithJ = diffWithJ;
        }
    }

    private Info f(char[] chars, Info[][][] dp, int end, int i, int j) {
        if(end < 0) {
            return new Info(0, -chars.length);
        }

        int ci = i - 'a', cj = j - 'a';
        if(dp[end][ci][cj] != null) {
            return dp[end][ci][cj];
        }

        Info info = f(chars, dp, end - 1, i, j);
        if(chars[end] == i) {
            info.diff++;

            // 没有j时, diffWithJ就是整个八经的波动值
            // 全为i时, diffWithJ为0
            info.diffWithJ++;
        } else if(chars[end] == j) {
            info.diff--;

            // 有出现j时, diffWithJ完全等于diff, 即本次的波动值
            // 全为j时, diffWithJ为-n, 但没关系, i、j反过来尝试时, 下面的max就为0了 
            info.diffWithJ = info.diff;

            // 全为j时, diff为-n, 此时下一个子串需与0取最大值, 认为不应该从负的diff出发
            info.diff = Math.max(info.diff, 0);
        }

        dp[end][ci][cj] = info;
        return info;
    }
}
```

##### 4）严格表结构优化 | O（26 * 26 * n）

- **思路**：在记忆化搜索的基础上发现，是后面的值依赖前面的值，所以需要从左到右进行初始化，同时，三维表缓存则可以使用 diff 和 diffWithJ 局部变量作为替代，从而省去了递归和 Info 的数据结构。
- **结论**：时间，89 ms，86.88%，空间，41 mb，68.83%，时间上，枚举 i、j 字符花费 O（26 * 26），dp 花费 O（n），所以时间复杂度为 O（26 * 26 * n），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {

    public int largestVariance(String s) {
        if(s == null || s.length() == 0) {
            return 0;
        }

        int max = Integer.MIN_VALUE, diff, diffWithJ;
        char[] chars = s.toCharArray();
        
        // 枚举最多字符i和最少字符j
        for(int i = 'a'; i <= 'z'; i++) {
            for(int j = 'a'; j <= 'z'; j++) {
                if(i == j) {
                    continue;
                }

                diff = 0;
                diffWithJ = -chars.length;
                
                // 枚举每个子串
                for(int end = 0; end < chars.length; end++) {
                    if(chars[end] == i) {
                        diff++;
                        
                        // 没有j时, diffWithJ就是整个八经的波动值
            			// 全为i时, diffWithJ为0
                        diffWithJ++;
                    } else if(chars[end] == j) {
                        diff--;

                   // 有出现j时, diffWithJ完全等于diff, 即本次的波动值
            	   // 全为j时, diffWithJ为-n, 但没关系, i、j反过来尝试时, 下面的max就为0了 
                        diffWithJ = diff;

                   // 全为j时, diff为-n, 此时下一个子串需与0取最大值, 认为不应该从负的diff出发
                        diff = Math.max(diff, 0);
                    } else {
                        continue;
                    }

                    // 结算每个子串的波动值
                    max = Math.max(max, diffWithJ);
                }
            }
        }

        return max;
    }
}
```

##### 5）严格表结构思路优化 | O（26 * n）

- **思路**：
  1. 在严格表结构的基础上，发现需要分别枚举 i 和 j 字符，同时还要遍历 chars 数组做 dp，如果遍历到 chars[k] 时，认为它就是 j 的话，那就可以省去一层 j 字符的外层循环了。
  2. 但如果 chars[k] 作为小数 j 的话，那么局部变量 diff 和 diffWithJ 就失去了 chars[k] 作为大数 i 的枚举情况了，这就需要使用另一个 diff' 和 diffWithJ' 来记录，所以设计了 diffs 和  diffWithJs 二维表，通过控制 ci、cj 作为 m 和 n 的坐标，即可控制是 diff' 还是 diff' 了，同时还满足变量的 dp 累加行为。
  3. 不过要注意的是，这种做法实质上，重叠了 diffs 或者 diffWithJs 数组相同位置的值，相当于 f 函数的 dp 操作，所以，chars 数组的遍历，需要放在枚举字符之前，因为如果放在枚举字符之后，就会失去 f 函数的 dp 特性，导致错误的发生。
- **结论**：时间，19 ms，91.25%，空间，41.1 mb，55.71%，时间上，遍历 chars 数组花费 O（n），枚举 i 字符花费 O（26），所以时间复杂度为 O（26 * n），空间上，由于使用了两张 26 * 26 的二维表，所以额外空间复杂度为 O（26 * 26）。

```java
class Solution {

    public int largestVariance(String s) {
        if(s == null || s.length() == 0) {
            return 0;
        }

        int max = Integer.MIN_VALUE, ci, cj;
        char[] chars = s.toCharArray();
        
        int[][] diffs = new int[26][26];
        int[][] diffWithJs = new int[26][26];
        for(int i = 'a'; i <= 'z'; i++) {
            Arrays.fill(diffWithJs[i - 'a'], -chars.length);
        }

        // 枚举最多字符i和最少字符chars[j], 这里重叠了diffs或者diffWithJs数组相同位置的值, 相当于f(end-1), 所以chars数组的遍历要放在枚举字符之前, 因为如果放在枚举字符之后, 就失去了f(end-1)的动态规划特性了
        for(int j = 0; j < chars.length; j++) {
            for(int i = 'a'; i <= 'z'; i++) {
                if(chars[j] == i) {
                    continue;
                }

                ci = i - 'a';
                cj = chars[j] - 'a';

                // ci => cj
                diffs[ci][cj]++;
                // 没有j时, diffWithJ就是整个八经的波动值
                // 全为i时, diffWithJ为0
                diffWithJs[ci][cj]++;
                
                // cj => ci
                diffs[cj][ci]--;
                // 有出现j时, diffWithJ完全等于diff, 即本次的波动值
                // 全为j时, diffWithJ为-n, 但没关系, i、j反过来尝试时, 下面的max就为0了 
                diffWithJs[cj][ci] = diffs[cj][ci];
                // 全为j时, diff为-n, 此时下一个子串需与0取最大值, 认为不应该从负的diff出发
                diffs[cj][ci] = Math.max(diffs[cj][ci], 0);

                // 结算最大波动值
                max = Math.max(max, Math.max(diffWithJs[ci][cj], diffWithJs[cj][ci]));
            }
        }

        return max;
    }
}
```

### 2.9. 算法思想 - 宏观调度

#### 48. 旋转图像 | medium

##### 1）宏观调度法 | O（n/2 * (n-1)）

- **思路**：
  1. 不再拘泥于每个坐标如何变换，从宏观的角度上看待问题，把问题看作成一次宏观的旋转。
  2. 每次选取当前层的左上角和右下角作为调度基准，比如选取左上角的位置作为交换中心，其他的值都和该值做交换，当前层的值都交换完毕后，则逐步把左上角和右下角往里缩进一层，继续调度，直到左右重合即得到最终结果。
  3. 其中值交换的过程为：每层划分为 4 组，分别为第一行、最后一列、最后一行、第一列，每组数量相等，其旋转方式如下：
     1. 第一行的组: 列下标赋值为当前层的最后一列, 行下标依次+1。
     2. 最后一列的组: 行下标赋值为当前层的最后一行, 列下标依次-1。
     3. 最后一行的组: 列下标赋值为当前层的第一列，行下标依次-1。
     4. 第一列的组: 行下标赋值为当前层的第一行, 列下标依次+1。
- **结论**：时间，0ms，100%，空间，38.5mb，72%，性能非常好，就这样吧~

```java
class Solution {
    public void rotate(int[][] matrix) {
        for(int i = 0; i < matrix.length / 2; i++) {
            f(matrix, i, i, matrix.length - 1 - i, matrix.length - 1 - i);
        }
    }

    private void f(int[][] matrix, int left, int up, int right, int down) {
        if(left > right || up > down) {
            return;
        }

        for(int rounds = 1; rounds <= right - left; rounds++) {
            // 第一行的组: 列下标赋值为当前层的最后一列, 行下标依次+1
            swap(matrix, left, left + rounds, up + rounds, right);

            // 最后一列的组: 行下标赋值为当前层的最后一行, 列下标依次-1
            swap(matrix, left, left + rounds, right, right - rounds);

            // 最后一行的组: 列下标赋值为当前层的第一列，行下标依次-1
            swap(matrix, left, left + rounds, down - rounds, left);

            // 第一列的组: 行下标赋值为当前层的第一行, 列下标依次+1
            swap(matrix, left, left + rounds, left, left + rounds);
        }
    }

    // src交换to
    private void swap(int[][] matrix, int from_row, int from_col, int to_row, int to_col) {
        int tmp = matrix[to_row][to_col];
        matrix[to_row][to_col] = matrix[from_row][from_col];
        matrix[from_row][from_col] = tmp;
    }
}
```

#### 54. 螺旋矩阵 | medium

##### 1）直接模拟法 | O（m * n）

- **思路**：照着题意，从 (0, 0) 位置开始，不断地向右、向下、向左、向上循环访问，即如果碰到越界的、访问过的，则换个方向访问，直到把所有位置都访问完，则循环结束。
- **结论**：时间，0 ms，100%，空间，39.9 mb，5.00%，时间上，由于每个位置只会被访问一遍，所以时间复杂度为 O（m * n），空间上，由于增加了一张 m * n 大的 visiteds 二维表，所以额外空间复杂度为 O（m * n）。

```java
class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        if(matrix == null || matrix.length == 0 || matrix[0] == null || matrix[0].length == 0) {
            return new ArrayList<>();
        }

        int m = matrix.length, n = matrix[0].length;
        boolean[][] visiteds = new boolean[m][n];
        List<Integer> res = new ArrayList<>();

        int i = 0, j = 0;
        visiteds[i][j] = true;
        res.add(matrix[i][j]);

        // 没访问完, 则继续访问
        while(res.size() < m * n) {
            // 访问顺序, 如果是向右访问, 访问不了则走下一个逻辑
            while(f(matrix, m, n, visiteds, res, i, j + 1)) {
                j++;
            }

            // 访问顺序, 如果是向下访问, 访问不了则走下一个逻辑
            while(f(matrix, m, n, visiteds, res, i + 1, j)) {
                i++;
            }

            // 访问顺序, 如果是向左访问, 访问不了则走下一个逻辑
            while(f(matrix, m, n, visiteds, res, i, j - 1)) {
                j--;
            }

            // 访问顺序, 如果是向上访问, 访问不了则走下一个逻辑
            while(f(matrix, m, n, visiteds, res, i - 1, j)) {
                i--;
            }
        }

        return res;
    }

    private boolean f(int[][] matrix, int m, int n, boolean[][] visiteds, List<Integer> res, int i, int j) {
        // 越界的、或者已经访问过的, 则无需再访问
        if(i < 0 || i >= m || j < 0 || j >= n || visiteds[i][j]) {
            return false;
        }

        // 设置已经访问过
        visiteds[i][j] = true;
        res.add(matrix[i][j]);

        return true;
    }
}
```

##### 2）宏观调度法 | O（m * n）

- **思路**：
  1. 不像直接模拟法那样，需要拘泥于每个转弯的细节，这里可以采用宏观调度的思路，即每次选择好左上角和右下角的元素作为起始，然后收集完一圈的元素，然后移动对角元素到内层，相当于不断收缩外层到内层，直到内外层重合，则停止收集。
  2. 其中，要注意的是，由于矩形可能为正方行，导致最内层只有一个元素，以及也有可能只有一行、只有一列的情况，所以每次收集都需要分开讨论：
     - 1）同一行，说明要么只有一行，要么是最后一行，则从左到右收集完整行的元素。
     - 2）同一列，但不同行，说明要么只有一列，要么是最后一列，则从上到下收集完整列的元素。
     - 3）不同行，也不同列，则说明还是在外层，则按规则来收集一层的元素。
- **结论**：时间，0 ms，100%，空间，39.9 mb，5.00%，时间上，由于每个位置只会被访问一遍，所以时间复杂度为 O（m * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        if(matrix == null || matrix.length == 0 || matrix[0] == null || matrix[0].length == 0) {
            return new ArrayList<>();
        }

        List<Integer> res = new ArrayList<>();
        int m = matrix.length, n = matrix[0].length;
        int left = 0, top = 0, right = n - 1, bottom = m - 1;
        while(left <= right && top <= bottom) {
            f(matrix, m, n, res, left, top, right, bottom);

            // 外层向内层收缩
            left++;
            top++;
            right--;
            bottom--;
        }

        return res;
    }

    private void f(int[][] matrix, int m, int n, List<Integer> res, int left, int top, int right, int bottom) {
        // 同一行, 说明要么只有一行, 要么是最后一行, 则从左到右收集完整行的元素
        if(top == bottom) {
            for(int i = left; i <= right; i++) {
                res.add(matrix[top][i]);
            }
        } 
        // 同一列, 但不同行, 说明要么只有一列, 要么是最后一列, 则从上到下收集完整列的元素
        else if(left == right) {
            for(int i = top; i <= bottom; i++) {
                res.add(matrix[i][left]);
            }
        } 
        // 不同行, 也不同列, 则说明还是在外层, 则按规则来收集一层的元素
        else {
            // 收集top行上, left => right的元素, 但不包括right
            for(int i = left; i < right; i++) {
                res.add(matrix[top][i]);
            }

            // 收集right列上, top => bottom的元素, 但不包括bottom
            for(int i = top; i < bottom; i++) {
                res.add(matrix[i][right]);
            }

            // 收集bottom行上, right => left的元素, 但不包括left
            for(int i = right; i > left; i--) {
                res.add(matrix[bottom][i]);
            }

            // 收集left列上, bottom => top的元素, 但不包括top
            for(int i = bottom; i > top; i--) {
                res.add(matrix[i][left]);
            }
        }
    }
}
```

#### 59. 螺旋矩阵 II | medium

##### 1）宏观调度法 | O（n^2）

- **思路**：
  1. 参考《54. 螺旋矩阵》的思路，只不过是把收集的操作，换成设置的操作，此时只需要记录号当前应该设置的数字 cur 即可。
  2. 其中，由于本题规定是正方形，所以最终结果只会是同行、同列，即 f 函数中的前两个 if 逻辑写其中一个即可。
  3. 而这里的解法，是可以同时兼容正方形矩阵，和非正方形矩阵的。
- **结论**：时间，0 ms，100%，空间，39.4 mb，54.06%，时间上，由于 n^2 的每个数字都需要设置，所以时间复杂度为 O（n^2），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int[][] generateMatrix(int n) {
        int[][] res = new int[n][n];
        int left = 0, right = n - 1, top = 0, bottom = n - 1, cur = 1;
        
        while(left <= right && top <= bottom) {
            cur = f(res, cur, left, right, top, bottom);
            left++;
            right--;
            top++;
            bottom--;
        }
        
        return res;
    }

    private int f(int[][] res, int cur, int left, int right, int top, int bottom) {
        // 由于是正方形矩阵, 所以这里可以省略掉同行、或者同列的其中一个, 如果属于同一行, 则从左到右设置
        if(top == bottom) {
            for(int i = left; i <= right; i++) {
                res[top][i] = cur++;
            }
        } 
        // 由于是正方形矩阵, 所以这里可以省略掉同行、或者同列的其中一个, 如果属于同一列, 则从上到下设置
        else if(left == bottom) {
            for(int i = top; i <= bottom; i++) {
                res[i][left] = cur++;
            }
        }
        // 如果不同行、不同列, 则上、右、下、左设置
        else {
            // 上
            for(int i = left; i < right; i++) {
                res[top][i] = cur++;
            }
            // 右
            for(int i = top; i < bottom; i++) {
                res[i][right] = cur++;
            }
            // 下
            for(int i = right; i > left; i--) {
                res[bottom][i] = cur++;
            }
            // 左
            for(int i = bottom; i > top; i--) {
                res[i][left] = cur++;
            }
        }

        return cur;
    }
}
```

### 3.0. 算法思想 - 回溯 | DFS

#### 模板代码

##### 1）组合枚举 | n 选 k 进行组合

```java
class Solution {
    private List<List<Integer>> res;
    private List<Integer> cur;

    public List<List<Integer>> combine(int n, int k) {
        this.res = new ArrayList<>();
        this.cur = new ArrayList<>();

        int[] nums = new int[n];
        for(int i = 0; i < n; i++) {
            nums[i] = i + 1;
        }

        dfs(nums, n, k, 0);
        return this.res;
    }

    private void dfs(int[] nums, int n, int k, int idx) {
        if(cur.size() == k) {
            res.add(new ArrayList<>(cur));
            return;
        }
        if(idx == n) {
            return;
        }
        // 剪枝：如果当前cur元素+后面的所有元素, 长度都不够k个的话, 那么就不用再递归尝试下去了, 直接返回即可
        if(cur.size() + (n - idx) < k) {
            return;
        }

        // 要
        cur.add(nums[idx]);
        dfs(nums, n, k, idx + 1);
        cur.remove(cur.size() - 1);

        // 不要
        dfs(nums, n, k, idx + 1);
    }
}
```

#### 22. 括号生成 | medium

##### 1）前缀树法 | O（2^n * 3 * 2）

- **思路**：

  1. 把 ”（” 和 “）” 一次放入前缀树，然后深度优先遍历，最后过滤掉非法括号串就好。
  2. 过滤非法括号串：初始化一个 count=0，从头遍历到尾，碰到  ”（” 则 count+1，碰到  ”）” 则 count-1，期间如果出现负数，或者最后 count 不为 0，则认为该括号串非法。

  ![1641661558130](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1641661558130.png)

- **结论**：259ms，5.63%，面试应该不好，实在想不出来时才用这个吧。

```java
class Solution {

    // 前缀树节点
    class Node {
        String value;
        Node[] nodes;
    }

    public List<String> generateParenthesis(int n) {
        List<String> res = new ArrayList<>();

        Node head = new Node();
        head.nodes = new Node[2];

        List<Node> firstNodeList = new ArrayList<>();
        firstNodeList.add(head);

        Map<Integer, List<Node>> levelMap = new HashMap<>();
        levelMap.put(0, firstNodeList);

        // 2n层前缀树
        int level = 2 * n;
        for(int i = 0; i < level; i++) {
            if(!levelMap.containsKey(i)) {
                levelMap.put(i+1, new ArrayList<>());
            }

            List<Node> nodeList = new ArrayList<>();
            for(Node node : levelMap.get(i)) {
                Node[] nodes = node.nodes;

                Node left = new Node();
                Node right = new Node();
                
                left.value = "(";
                right.value = ")";

                nodes[0] = left;
                nodes[1] = right;

                nodeList.add(left);
                nodeList.add(right);

                if(i < level - 1) {
                    left.nodes = new Node[2];
                    right.nodes = new Node[2];
                }
            }
            levelMap.put(i+1, nodeList);
        }

        generateString(res, head, "");
        filterInvalidString(res, n);
        return res;
    }

    private void filterInvalidString(List<String> res, int n) {
        Iterator<String> it = res.iterator();
        while(it.hasNext()) {
            int count = 0;
            String str = it.next();
            boolean isRemove = false;
            for(char c : str.toCharArray()) {
                if(count < 0) {
                    it.remove();
                    isRemove = true;
                    break;
                }
                if(c == '(') {
                    count++;
                } else {
                    count--;
                }
            }
            if(count != 0 && !isRemove) {
                it.remove();
            }
        }
    }

    private void generateString(List<String> res, Node head, String cur) {
        if(head.nodes == null) {
            res.add(cur);
            return;
        }
        
        for(int i = 0; i < head.nodes.length; i++) {
            generateString(res, head.nodes[i], cur + head.nodes[i].value);
        }
    }
}
```

##### 2）回溯法 | O（2^[f(n-1)+f(n+2)]）

- **思路**：

  1. 如果左括号数量等于右括号数量，则不可以生成右括号了，接下来就交给子过程去完成剩余部分。
  2. 如果左括号数量不等于右括号数量，则还可以继续生成左括号和右括号：
     1. 如果左括号数量还有，则既可以生成左括号，再生成右括号（顺序不规定）。
     2. 但如果左括号数量没有了，则只能生成右括号来填补了。

- **时间复杂度**：

  1. n=1时，需要递归 2^2 次。
  2. n=2时，需要递归 2^3 次。
  3. n=3时，需要递归 2^5 次。
  4. 因此，n=n 时，需要递归 2^[f(n-1)+f(n+2)] 次。

  ![1641665536662](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1641665536662.png)

- **结论**：1ms，73.83%，面试主要考这个思路！

```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> res = new ArrayList<>();
        if(n == 0) {
            return res;
        }

        f(res, "", n, n);
        return res;
    }

    private void f(List<String> res, String cur, int l, int r) {
        if(l == 0 && r == 0) {
            res.add(cur);
            return;
        }
    
        // 如果左括号数量等于右括号数量, 则不可以生成右括号
        if(l == r) {
            f(res, cur + "(", l-1, r);
        }
        // 否则, 只能即生成左括号又可以生成右括号
        else {
            // 如果左括号还有, 且左边左右平衡了, 则可以继续加左括号
            if(l > 0) {
                f(res, cur + "(", l-1, r);
            }

            // 或者右括号, 但如果左括号没有了, 则只能填写右括号了
            f(res, cur + ")", l, r-1);
        }
    }
}
```

#### 37. 解数独 | hard

##### 1）哈希表 + dfs + 回溯 | O（ m * n + [m * n]^9 ）

- **思路**：
  1. 先根据 board 二维表进行初始化，rowVis 表示按统计数字是否有出现，通过 i 来表示行索引；colVis 表示按列统计数字是否有出现，通过 j 来表示列索引；regionVis 表示按九宫格区域统计数字是否有出现，通过 `[i / 3][j / 3]` 来表示九宫格区域索引；blankList 表示空格的索引列表。
  2. 然后，开始遍历填写空白字符，通过枚举 1~9，尝试是否能填写，如果可以，则继续填写下一个空白字符，否则回溯掉之前填写的数字，重新枚举下一个数字。
  3. 最后，如果所有空白字符都填写过，则直接一层层返回，返回到最顶层时，board 数组所有空白字符都已经被填写过正确的数字了，即已经是正确结果了。
- **结论**：时间，2 ms，90.53%，空间，39.2 mb，20.31%，时间上，根据 board 二维表初始化数据，花费 O（m * n），dfs + 回溯花费 O（ [m * n]^9 ），所以整体时间复杂度为 O（ m * n + [m * n]^9 ），空间上，由于使用了一张 81 长的 rowVis，81 长的 colVis，81 长的 regionVis，81 长的 blankList，dfs 递归栈深度最大为 81，所以额外空间复杂度为 O（324）。

```java
class Solution {
    private boolean[][] rowVis = new boolean[9][9];// 1~9, 从0开始
    private boolean[][] colVis = new boolean[9][9];// 1~9, 从0开始
    private boolean[][][] regionVis = new boolean[3][3][9];// 1~9, 从0开始
    private List<int[]> blankList = new ArrayList<>();

    public void solveSudoku(char[][] board) {
        // 初始化访问数组
        for(int i = 0; i < 9; i++) {
            for(int j = 0; j < 9; j++) {
                char c = board[i][j];
                if(c == '.') {
                    blankList.add(new int[] { i, j });
                }
                else {
                    int idx = c - '0' - 1;
                    rowVis[i][idx] = true;
                    colVis[j][idx] = true;
                    regionVis[i/3][j/3][idx] = true;
                }
            }
        }

        // 开始枚举填写空白格字符
        dfs(board, 0);
    }

    // 枚举填写空白格字符
    private boolean dfs(char[][] board, int blankIdx) {
        if(blankIdx == blankList.size()) {
            return true;
        }

        // 获取空白格索引
        int[] blank = blankList.get(blankIdx);
        int x = blank[0], y = blank[1];
        
        // 枚举1~9
        for(int i = 1; i <= 9; i++) {
            int idx = i - 1;

            // 行、列、九宫格上有该数字, 则不填当前数字
            if(rowVis[x][idx] || colVis[y][idx] || regionVis[x/3][y/3][idx]) {
                continue;
            }

            // 填当前数字, 设置访问结果
            board[x][y] = (char) (i + '0');
            rowVis[x][idx] = colVis[y][idx] = regionVis[x/3][y/3][idx] = true;

            // 继续填写下一个空白格数字
            if(dfs(board, blankIdx + 1)) {
                return true;// 所有空白格都填写完毕, 则直接返回即可
            }

            // 填写失败, 则回溯掉之前填的访问结果
            board[x][y] = '.';
            rowVis[x][idx] = colVis[y][idx] = regionVis[x/3][y/3][idx] = false;
        }

        // 尝试完毕, 还没得到结果的, 则认为尝试失败
        return false;
    }
}
```

#### 39. 组合总和 | medium

##### 1）暴力解法 | >= O（m * n）

- **思路**：定义一个 f（end，rest）函数，它返回以 end 结尾的数组中数字和等于 rest 的所有组合。
  1. 尝试使用 0,1,2,..,n 个 end 值去匹配 rest，如果尝试发现大于了 rest 值，则不用再尝试了，因为再大已经不可能了，强调一下这里求得是**以end结尾**，这里不用再考虑前面的了。
  2. 如果 n 个 end 去匹配，发现没有大于 rest，而是等于 rest，此时说明 n * end 就是其中的一个解，此时把它加入集合中，继续尝试 n+1。
  3. 如果 n 个 end 去匹配，发现小于 rest，说明不够，还需要前面的来凑，此时可以调用 f（end-1，rest-n*num[end]）去获取前面匹配剩余值的所有组合列表。
     1. 如果返回的组合列表为空，说明当前 n * end + 前面，都无法匹配 rest，即 n 不是答案，继续尝试 n+1.
     2. 如果返回的组合列表不为空，说明当前 n * end + 前面，是有解的，因此把他们加入集合中，继续尝试下一个。
  4. 最后尝试完所有尝试后（即 n * end > rest），就不用再试了，返回结果即可。
- **结论**：
  1. 由于需要尝试 m 次，每次尝试最差需要遍历整个数组 n，且还需要包括组装返回结果的时间，因此，整体时间复杂度为 O（m * n）以上，但这是不可避免的，也算是 O（m * n）还好吧（3ms，58.65%）。
  2. 空间浪费严重，38.8mb，24.91%，每次都需要构建 ArrayList 才返回，没能做到复用，面试会挂！

```java
class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        if(candidates == null || candidates.length == 0) {
            return new ArrayList<>();
        }
        return f(candidates, candidates.length - 1, target);
    }

    // 以end结尾时的所有数字组合之和等于rest的所有情况, 都存在res集合中
    private List<List<Integer>> f(int[] nums, int end, int rest) {
        if(rest <= 0 || end < 0) {
            return new ArrayList<>();
        }

        // 开始尝试 => 大于rest则返回, 因为已经是相对最小的了
        int sum;
        List<List<Integer>> res = new ArrayList<>();
        for(int i = 0; (sum = i * nums[end]) <= rest; i++) {
            // n个end等于rest时, 则加入集合
            if(sum == rest) {
                List<Integer> members = new ArrayList<>();
                addMembers(members, nums[end], i);
                res.add(members);
            } else {
                // n个end小于rest时, 则看前面的是否组合成功
                if(sum < rest) {
                    // 如果组合成功, 说明end加前面之和等于rest, 则加入最终结果
                    List<List<Integer>> leftRes = f(nums, end - 1, rest - sum);
                    if(!leftRes.isEmpty()) {
                        for(List<Integer> leftMembers : leftRes) {
                            addMembers(leftMembers, nums[end], i);
                            res.add(leftMembers);
                        }
                    }
                    // 如果组合失败, 则再尝试下一个n+1个end
                }
            }
        }

        // 尝试完毕, 返回结果
        return res;
    }

    private void addMembers(List<Integer> members, int value, int i) {
        for(int j = i; j > 0; j--) {
            members.add(value);
        }
    }
}
```

##### 2）回溯法 | >= O（m * n）

- **思路**：同暴力解法的思路，但好在使用了 ArrayList 根据索引 O（1）remove 的特性，回溯时从后面挨个把当前 i * end 的尝试删掉，以实现一个 ArrayList 重复利用的目的，大大节省了空间。
- **结论**：时间 3m，58.602%，空间 38 mb，99%，可以了，就这样吧。

```java
class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        if(candidates == null || candidates.length == 0) {
            return new ArrayList<>();
        }

        // 回溯剪枝
        List<List<Integer>> res = new ArrayList<>();
        List<Integer> cur = new ArrayList<>();
        f(candidates, res, cur, candidates.length - 1, target);
        return res;
    }

    // f代表尝试以end结尾的数字, 去组合出刚好等于rest的结果
    private void f(int[] nums, List<List<Integer>> res, List<Integer> cur, int end, int rest) {
        if(rest == 0) {
            res.add(new ArrayList<>(cur));
            return;
        }
        if(rest < 0 || end < 0) {
            return;
        }

        // 开始尝试
        for(int i = 0; i * nums[end] <= rest; i++) {
            // 加入尝试
            addMembers(cur, nums[end], i);

            // 子过程去解决剩余的rest
            f(nums, res, cur, end-1, rest - i * nums[end]);
           
            // 回溯时把上面加入的尝试删除
            removeMembers(cur, i);
        }
    }

    private void addMembers(List<Integer> cur, int value, int n) {
        for(int i = n; i > 0; i--) {
            cur.add(value);
        }
    }

    private void removeMembers(List<Integer> cur, int n) {
        for(int i = n; i > 0; i--) {
            cur.remove(cur.size() - 1);
        }
    }
}
```

#### 40. 组合总和 II | medium

##### 1）顺序排序 + 词频数组 + 回溯 + 剪枝 | O（n * logn + n + k^q）

- **思路**：
  1. 先顺序排序，可以在后面进行剪枝，如果当前元素都大于 rest 了，那么后面也不用继续尝试了，直接返回即可。
  2. 再顺序统计词频数组，可以尝试组合 0,1...,n 次重复同类元素，使用数组而不是哈希表的原因是，可以保持排序后的特性。
  3. 最后再回溯统计结果，分为当前元素要还是不要，要的话，则尝试 0~n 次重复当前元素，然后 rest 扣减对应次数的值，不要的话，则直接尝试下一个元素，rest 不变即可。
- **结论**：时间，2 ms，99.81%，空间，41.5 mb，77.26%，时间上，顺序排序花费 O（n * logn），统计词频数组花费 O（n），dfs 回溯 + 剪枝花费 O（k^q），其中 k 为 feq 数组的大小，q 为 feq 数组中元素最大的出现次数，所以整体时间复杂度为 O（n * logn + n + k^q），空间上，快速排序递归栈深度为 logn，cur 数组花费 O（k），词频数组花费 O（k * q），dfs 递归栈深度为 n，所以额外空间复杂度为 O（logn + k + k * q + n）。

```java
class Solution {
    private List<List<Integer>> res = new ArrayList<>();
    private List<Integer> cur = new ArrayList<>();
    private List<int[]> feq = new ArrayList<>();

    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        // 先顺序排序
        Arrays.sort(candidates);

        // 再顺序统计词频数组
        initFeqList(candidates);

        // 回溯统计结果
        dfs(0, target);

        // 返回最终结果
        return res;
    }

    private void dfs(int idx, int rest) {
        if(rest == 0) {
            res.add(new ArrayList<>(cur));
            return;
        }
        if(idx == feq.size()) {
            return;
        }

        // 要当前元素 
        // 获取词频元素
        int[] element = feq.get(idx);

        // 如果当前元素都大于rest了, 那么后面也不用继续尝试了, 直接返回即可
        if(element[0] > rest) {
            return;
        }

        // 同类数字的最大重复次数
        int n = Math.min(rest / element[0], element[1]);

        // 尝试组合0,1...,n次重复同类元素
        for(int i = 1; i <= n; i++) {
            cur.add(element[0]);
            dfs(idx + 1, rest - i * element[0]);
        }

        // 回溯, 回滚掉之前加入的所有重复同类元素
        for(int i = 1; i <= n; i++) {
            cur.remove(cur.size() - 1);
        }

        // 不要当前元素
        dfs(idx + 1, rest);
    }

    // 顺序统计词频数组
    private void initFeqList(int[] candidates) {
        for(int i = 0; i < candidates.length; i++) {
            if(feq.size() == 0) {
                feq.add(new int[] { candidates[i], 1 });
            } else {
                int[] element = feq.get(feq.size() - 1);
                if(candidates[i] == element[0]) {
                    element[1]++;
                } else {
                    feq.add(new int[] { candidates[i], 1 });  
                }
            }
        }
    }
}
```

#### 46. 全排列 | medium

##### 1）回溯法 | O（n * n-1）

- **思路**：
  1. 设计一个 f（end）的函数，代表使用 [0~end]结尾之间所有的数字进行全排列，将得到的组合结果设置到 res 里面。
  2. f（end）函数里，通过索引枚举的方式，从 0 枚举到 num.length - 1，每次枚举都尝试把当前 end 数字放到 i 位置上，设置完成后再把剩余位置交给 end-1 数字来完成填充，而关键的一步是，每次在 end-1 填充完毕后，都需要把当前位置的元素清空掉，以腾出空间继续下一次尝试。
  3. 值得注意的是，如果使用的是 List 实现类的 get（i）/set（i）/remove（i），操作 i 位置元素都会先校验 i 是否越界（即 index >= size），这就需要提前把 size 扩充起来，比如填充一堆 null 进去，即可实现 size 赋初值的目的。
  4. 同理，如果在清空 i 位置元素使用的是 remove（i），则又会在删除元素的同时，把 size--，使得下次 set（i）的时候又报越界异常，因此清空元素使用的是 set（i，null），以达到清空了元素又不会减小 size 的目的。
- **结论**：时间 1ms，80.35%，空间 38.4 mb，88.93%，非常高效，就这样吧~

```java
class Solution {
    public List<List<Integer>> permute(int[] nums) {
        if(nums == null || nums.length == 0) {
            return new ArrayList<>();
        }

        List<Integer> cur = new ArrayList<>(nums.length);
        for(int i = 0; i < nums.length; i++) {
            cur.add(null);
        }

        List<List<Integer>> res = new LinkedList<>();        
        f(nums, res, cur, nums.length - 1);
        return res;
    }

    // f代表使用[0~end]结尾所有数字参与的全排列组合
    private void f(int[] nums, List<List<Integer>> res, List<Integer> cur, int end) {
        if(end < 0) {
            res.add(new ArrayList<>(cur));
            return;
        }

        for(int i = 0; i < nums.length; i++) {
            // 如果i位置已经被占据了, 那么就尝试下一个位置
            if(cur.get(i) != null) {
                continue;
            }

            // 加入尝试
            cur.set(i, nums[end]);

            // 子过程填完剩下的格子
            f(nums, res, cur, end-1);

            // 回溯时把之前的尝试清空掉, 以腾出空间, 方便继续下一个尝试
            cur.set(i, null);
        }
    }
}
```

#### 47. 全排列 II | medium

##### 1）dfs + 回溯 + 同层哈希表判重 + 不同层打标判重 | O（n + n!）

- **思路**：参考《46. 全排列》的思路，即通过为 list 设置 null 坑位来进行 dfs + 回溯，但要这里注意的是，由于 nums 中的数字有重复，所以需要通过 vis 打标数组，来实现不同层的判重，通过每层都设置一个哈希表，来实现同层的判重，以保证最终生成的排列不会重复。
- **结论**：时间，2 ms，43.08%，空间，42.1 mb，55.35%，时间上，设置坑位花费 O（n），dfs 栈深度最大为 n，但由于每次 dfs 过程做了去重，所以分别需要遍历 n,n-1,...1 个数字，所以整体时间复杂度为 O（n + n!），空间上，cur 数组花费 O（n），vis 数组花费 O（1），dfs 花费 O（n），每次 dfs 过程都花费一个 n,n-1,...1 长的哈希表，所以整体额外空间复杂度为 O（n + n!）。

```java
class Solution {
    private List<List<Integer>> res = new ArrayList<>();
    private List<Integer> cur = new ArrayList<>();
    private boolean[] vis = new boolean[8];
    
    public List<List<Integer>> permuteUnique(int[] nums) {
        // 设置坑位
        for(int i = 0; i < nums.length; i++) {
            cur.add(null);
        }

        dfs(nums, 0);
        return res;
    }

    private void dfs(int[] nums, int idx) {
        // 收集结果
        if(idx == nums.length) {
            res.add(new ArrayList<>(cur));
            return;
        }

        // 枚举开头
        Set<Integer> useSet = new HashSet<>();
        for(int i = 0; i < nums.length; i++) {
            // 如果已经设置过, 则跳过
            if(useSet.contains(nums[i]) || vis[i]) {
                continue;
            }

            // 如果没设置过, 则设置到坑位中
            cur.set(idx, nums[i]);
            useSet.add(nums[i]);
            vis[i] = true;

            // 继续添加下一个坑位
            dfs(nums, idx + 1);

            // 回溯掉当前坑位
            cur.set(idx, null);
            vis[i] = false;
        }
    }
}
```

##### 2）dfs + 回溯 + 同层排序判重 + 不同层打标判重 | O（n + n * n!）

- **思路**：与 1 的做法类似，不同的地方在于，同层的判重，不再使用哈希表，而是通过预先排序，然后每次只取第一个同类的数字，从而保证不会重复，但要注意的是，这样还需要加上 `!vis[i - 1]` 的条件，以保证 i - 1 是同层的数字，而不是上一层的数字。
- **结论**：时间，1 ms，99.77%，空间，42.1 mb，51.13%，时间上，设置坑位花费 O（n），排序花费 O（n * logn），dfs 栈深度最大为 n，但由于每次 dfs 过程做了去重，所以分别需要遍历 n,n-1,...1 个数字，所以整体时间复杂度为 O（n + n * logn + n!），空间上，cur 数组花费 O（n），vis 数组花费 O（1），dfs 花费 O（n），所以整体额外空间复杂度为 O（2 * n）。

```java
class Solution {
    private List<List<Integer>> res = new ArrayList<>();
    private List<Integer> cur = new ArrayList<>();
    private boolean[] vis = new boolean[8];
    
    public List<List<Integer>> permuteUnique(int[] nums) {
        // 设置坑位
        for(int i = 0; i < nums.length; i++) {
            cur.add(null);
        }

        // 先顺序排序
        Arrays.sort(nums);

        // 再dfs
        dfs(nums, 0);
        return res;
    }

    private void dfs(int[] nums, int idx) {
        // 收集结果
        if(idx == nums.length) {
            res.add(new ArrayList<>(cur));
            return;
        }

        // 枚举开头
        for(int i = 0; i < nums.length; i++) {
            // 如果上层已经设置过了, 则跳过
            if(vis[i]) {
                continue;
            }
            // 如果同层的只取第一个同类的, 用!vis[i - 1]保证是同层的, 因为被回溯清空掉了, 所以跳过
            if(i - 1 > -1 && nums[i] == nums[i - 1] && !vis[i - 1]) {
                continue;
            }

            // 如果没设置过, 则设置到坑位中
            cur.set(idx, nums[i]);
            vis[i] = true;

            // 继续添加下一个坑位
            dfs(nums, idx + 1);

            // 回溯掉当前坑位
            cur.set(idx, null);
            vis[i] = false;
        }
    }
}
```

#### 51. N 皇后 | hard

##### 1）dfs + 回溯 + 枚举 + 行列斜线判重 | O（n!）

- **思路**：
  1. dfs + 回溯的思路参考《47. 全排列 II》，判重的思路参考《37. 解数独》，具体思路见代码注释，都是回溯的套路模板。
  2. 这里做了一些优化，就是由于是枚举行，所以之前定的 rowVis 数组可以去掉，由于每行必定会有一个皇后，所以只需要记录其列下表即可，即使用一个 colIdxs 数组即可替代之前定的 matrix 二维表。
- **结论**：时间，2 ms，99.80%，空间，41.3 mb，82.99%，时间上，dfs 深度为 n，即枚举了 n 行，由于做了判重，所以每次 dfs 递归过程，分别需要枚举 n-1,n-2...,1 列，所以整体时间复杂度为 O（n!），空间上，colVis 长 n，lslashVis、rslashVis 长 2 * n，colIdxs 长 n，所以整体额外空间复杂度为 O（4 * n）。

```java
class Solution {
    private List<List<String>> res;
    private int[] colIdxs;
    private boolean[] colVis;
    private boolean[] lslashVis;
    private boolean[] rslashVis;
    
    public List<List<String>> solveNQueens(int n) {
        res = new ArrayList<>();

        // 先全部设置为'.'
        colIdxs = new int[n];
        Arrays.fill(colIdxs, -1);

        // 统计列上、左斜线、右斜线上的皇后是否有出现
        colVis = new boolean[n];
        lslashVis = new boolean[2 * n - 1];// row - col + n - 1, 长2 * n - 2 + 1
        rslashVis = new boolean[2 * n - 1];// row + col, 长2 * n - 2 + 1

        // dfs枚举行, 获取统计结果
        dfs(n, 0, 0);
        return res;
    }

    private void dfs(int n, int i, int size) {
        if(size == n) {
            res.add(convert2Strs(n, colIdxs));
            return;
        }

        // 递归中再枚举列
        for(int j = 0; j < n; j++) {
            if(colVis[j] || lslashVis[i - j + n - 1] || rslashVis[i + j]) {
                continue;
            }

            // 如果行、列、斜线都没有放置, 那么就放置皇后
            colVis[j] = true;
            lslashVis[i - j + n - 1] = true;
            rslashVis[i + j] = true;
            colIdxs[i] = j;
            
            // dfs继续枚举下一行
            dfs(n, i + 1, size + 1);

            // 回溯掉之前放置的
            colIdxs[i] = -1;
            colVis[j] = false;
            lslashVis[i - j + n - 1] = false;
            rslashVis[i + j] = false;
        }
    }

    private List<String> convert2Strs(int n, int[] colIdxs) {
        List<String> cur = new ArrayList<>();

        // 遍历每行
        for(int i = 0; i < n; i++) {
            char[] chars = new char[n];
            Arrays.fill(chars, '.');
            chars[colIdxs[i]] = 'Q';
            cur.add(new String(chars));
        }

        return cur;
    }
}
```

##### 2）dfs + 回溯 + 枚举 + 位运算判重 | O（n!）

- **思路**：大体思路同 1 的解法，但不同的是，根据题意，n 最大为 9，colVis 最长为 9，lslashVis 最长为 17，rslashVis 最长为 17，所以可以不采用数组做判重，改用二进制的位运算方式实现，即：
  1. 通过 x & (1 << offset) != 0，看 offset 位是否放置了皇后。
  2. 通过 x |= (1 << offset) ，在 offset 位上放置皇后。
  3. 通过 x &= ~(1 << offset)，取消 offset 位上的皇后。
- **结论**：时间，1 ms，99.80%，空间，41.7 mb，43.56%，时间上，dfs 深度为 n，即枚举了 n 行，由于做了判重，所以每次 dfs 递归过程，分别需要枚举 n-1,n-2...,1 列，所以整体时间复杂度为 O（n!），空间上，colIdxs 长 n，所以额外空间复杂度为 O（n）。

```java
class Solution {
    private List<List<String>> res;
    private int[] colIdxs;
    private int colVis;
    private int lslashVis;// row - col + n - 1, 长2 * n - 2 + 1
    private int rslashVis;// row + col, 长2 * n - 2 + 1
    
    public List<List<String>> solveNQueens(int n) {
        res = new ArrayList<>();

        // 先全部设置为'.'
        colIdxs = new int[n];
        Arrays.fill(colIdxs, -1);

        // dfs枚举行, 获取统计结果
        dfs(n, 0, 0);
        return res;
    }

    private void dfs(int n, int i, int size) {
        if(size == n) {
            res.add(convert2Strs(n, colIdxs));
            return;
        }

        // 递归中再枚举列
        for(int j = 0; j < n; j++) {
            int colMark = 1 << j;
            int lslashMark = 1 << (i - j + n - 1);
            int rslashMark = 1 << (i + j);

            if(
                // 列
                (colVis & colMark) != 0 
                // 左斜线
                || (lslashVis & lslashMark) != 0
                // 右斜线
                || (rslashVis & rslashMark) != 0
            ) {
                continue;
            }

            // 如果行、列、斜线都没有放置, 那么就放置皇后
            colVis |= colMark;
            lslashVis |= lslashMark;
            rslashVis |= rslashMark;
            colIdxs[i] = j;

            // dfs继续枚举下一行
            dfs(n, i + 1, size + 1);

            // 回溯掉之前放置的
            colIdxs[i] = -1;
            colVis &= ~colMark;
            lslashVis &= ~lslashMark;
            rslashVis &= ~rslashMark;
        }
    }

    private List<String> convert2Strs(int n, int[] colIdxs) {
        List<String> cur = new ArrayList<>();

        // 遍历每行
        for(int i = 0; i < n; i++) {
            char[] chars = new char[n];
            Arrays.fill(chars, '.');
            chars[colIdxs[i]] = 'Q';
            cur.add(new String(chars));
        }

        return cur;
    }
}
```

#### 52. N 皇后 II | hard

##### 1）dfs + 回溯 + 枚举 + 行列斜线判重 | O（n!）

- **思路**：在 《51. N 皇后》的基础上，把 res 结果收集，改为 cnt 数量统计即可。
- **结论**：时间，1 ms，79.47%，空间，38.2 mb，65.01%，时间上，dfs 深度为 n，即枚举了 n 行，由于做了判重，所以每次 dfs 递归过程，分别需要枚举 n-1,n-2...,1 列，所以整体时间复杂度为 O（n!），空间上，colVis 长 n，lslashVis、rslashVis 长 2 * n，colIdxs 长 n，所以整体额外空间复杂度为 O（4 * n）。

```java
class Solution {
    private int cnt;
    private int[] colIdxs;
    private boolean[] colVis;
    private boolean[] lslashVis;
    private boolean[] rslashVis;
    
    public int totalNQueens(int n) {
        // 先全部设置为'.'
        colIdxs = new int[n];
        Arrays.fill(colIdxs, -1);

        // 统计列、左斜线、右斜线的皇后是否有出现
        colVis = new boolean[n];
        lslashVis = new boolean[2 * n - 1];// row - col + n - 1, 长2 * n - 2 + 1
        rslashVis = new boolean[2 * n - 1];// row + col, 长2 * n - 2 + 1

        // dfs枚举行, 获取统计结果
        dfs(n, 0, 0);
        return cnt;
    }

    private void dfs(int n, int i, int size) {
        if(size == n) {
            cnt++;
            return;
        }

        // 递归中再枚举列
        for(int j = 0; j < n; j++) {
            if(colVis[j] || lslashVis[i - j + n - 1] || rslashVis[i + j]) {
                continue;
            }

            // 如果行、列、斜线都没有放置, 那么就放置皇后
            colVis[j] = true;
            lslashVis[i - j + n - 1] = true;
            rslashVis[i + j] = true;
            colIdxs[i] = j;
            
            // dfs继续枚举下一行
            dfs(n, i + 1, size + 1);

            // 回溯掉之前放置的
            colIdxs[i] = -1;
            colVis[j] = false;
            lslashVis[i - j + n - 1] = false;
            rslashVis[i + j] = false;
        }
    }

    private List<String> convert2Strs(int n, int[] colIdxs) {
        List<String> cur = new ArrayList<>();

        // 遍历每行
        for(int i = 0; i < n; i++) {
            char[] chars = new char[n];
            Arrays.fill(chars, '.');
            chars[colIdxs[i]] = 'Q';
            cur.add(new String(chars));
        }

        return cur;
    }
}
```

#### 60. 排列序列 | hard

##### 1）暴力回溯 + 不同层打标判重 + 序号跟踪 | O（n + n!）

- **思路**：参考《47. 全排列 II》的思路，还是走通用的回溯模板，不同的是，在递归过程中加入 cur 序号标记，在数组遍历结束后，判断 cur 是否达到 k 序号即可。
- **结论**：时间，510 ms，19.61%，空间，39.1 mb，42.71%，时间上，初始化 nums 数组花费 O（n），dfs 栈深度最大为 n，但由于每次 dfs 过程做了去重，所以分别需要遍历 n,n-1,...1 个数字，所以整体时间复杂度为 O（n + n!），空间上，nums 数组花费 O（n），vis 数组花费 O（n），dfs 递归栈花费 O（n），所以整体额外空间复杂度为 O（3 * n）。

```java 
class Solution {
    private int n;
    private int k;
    private int cur;
    private int[] nums;
    private boolean[] vis;
    private StringBuilder sb;

    public String getPermutation(int n, int k) {
        // 组装[1, n]
        int[] nums = new int[n];
        for(int i = 0; i < n; i++) {
            nums[i] = i + 1;
        }

        // 全局变量
        this.n = n;
        this.k = k;
        this.cur = 0;
        this.nums = nums;
        this.vis = new boolean[n + 1];
        this.sb = new StringBuilder();

        // dfs + 回溯
        dfs(0);
        return this.sb.toString();
    }

    private boolean dfs(int idx) {
        // 字符用完, 则看有没有找到第k个
        if(idx == n) {
            return ++cur == k;
        }

        // 从小的开始枚举
        for(int i = 0; i < n; i++) {
            // 这字符在上层用过, 那么就跳过
            if(vis[i]) {
                continue;
            }

            // 没用过则加入
            sb.append(nums[i]);
            vis[i] = true;

            // 继续枚举下一个, 找到了则返回
            if(dfs(idx + 1)) {
                return true;
            }

            // 没找到, 则回溯, 继续尝试下一个
            sb.deleteCharAt(sb.length() - 1);
            vis[i] = false;
        }

        return false;
    }
}
```

##### 2）数学 + 分析规律 | O（2 * n + n^2）

- **思路**：
  1. 通过分析数字排列的规律可知，对于 n 个数字，全排列数为 n!，而任意一个数字开头的全排列数则为 (n - 1)!。
  2. 所以，可以先通过 k / (n - 1)! 获取在 n 个数字时，排第 k 的排列中开头的那个数字，而由于 k 属于 [0, [n - 1]!）中的一个，所以需要先 k--。
  3. 然后，把得到的开头数字追加到结果中，再通过 k % (n - 1)! 计算剩余的 k 排多少，并删除 nums 数组中该位置的数字，更新 n--，周而复始，直到所有位置上的数字都求出来，则返回结果即可。
- **结论**：时间，1 ms，87.53%，空间，39.4 mb，22.54%，时间上，获取 nums 数组花费 O（n），获取 factorials 数组花费 O（n），遍历 + 设置 n 个位置花费 O（n^2），所以整体时间复杂度为 O（2 * n + n^2），空间上，nums 长 n，factorials 长 n，所以额外空间复杂度为 O（2 * n）。

```java
class Solution {
    public String getPermutation(int n, int k) {
        // 获取[1~n]的数字数组
        List<Integer> nums = initNums(n);

         // 获取[1~n]的阶乘函数
        int[] factorials = initFactorials(n);

        // 依次计算第k个序列的第1,2,...,n-1位置的数字
        StringBuilder sb = new StringBuilder();

        // k从0开始, 方便计算属于[0, [n - 1]!)的哪个索引
        k--;

        // (n - 1)!存在
        while(n - 1 >= 0) {
            // 获取(n - 1)!
            int fac = factorials[n - 1];

            // 计算k属于(0, [n - 1]!)的哪个索引, 并更新下一个k的大小
            int idx = k / fac;
            k %= fac;

            // 组装该idx索引下nums对应的数字
            sb.append(nums.get(idx));

            // 组装后删除idx索引对应的数字, 并且扣减一个n的数量
            nums.remove(idx);
            n--;
        }

        return sb.toString();
    }

    // 获取[1~n]的数字数组
    private List<Integer> initNums(int n) {
        List<Integer> nums = new ArrayList<>();
        for(int i = 1; i <= n; i++) {
            nums.add(i);
        }
        return nums;
    }

    // 获取[1~n]的阶乘函数
    private int[] initFactorials(int n) {
        // 先求阶乘函数：小于1的, 默认设置阶乘结果为1
        int[] factorials = new int[n];
        factorials[0] = 1;
        for(int i = 1; i < n; i++) {
            factorials[i] = i * factorials[i - 1];
        }
        return factorials;
    }
}
```

#### 77.  组合 | medium

##### 1）构造元素数组 + 在数组上进行组合枚举 + dfs + 回溯 + 剪枝 | O（n + C[n，k] * k）

- **思路**：参考《47. 全排列 II》的思路，还是走通用的回溯模板，先构造一个元素数组，存放 [1, n] 的元素，再进行 dfs 枚举使用哪些元素，而 dfs 实现则分要还是不要的判断，当满足个数等于 k 时，则加入结果集合中。
- **结论**：时间，1 ms，99.99%，空间，42.9 mb，22.10%，时间上，由于初始化 nums 数组花费 O（n），dfs 存在 C（n，k）种选择方式，表示在 n 个当中选 k 个，而每次 `new ArrayList<>(cur)` 花费 O（k），所以整体时间复杂度为 O（n + C[n，k] * k），空间上，nums 数组长 n，cur 最大长 n，dfs 最大栈深度为 k，所以整体额外空间复杂度为 O（2 * n + k）。

```java
class Solution {
    private List<List<Integer>> res;
    private List<Integer> cur;

    public List<List<Integer>> combine(int n, int k) {
        this.res = new ArrayList<>();
        this.cur = new ArrayList<>();

        int[] nums = new int[n];
        for(int i = 0; i < n; i++) {
            nums[i] = i + 1;
        }

        dfs(nums, n, k, 0);
        return this.res;
    }

    private void dfs(int[] nums, int n, int k, int idx) {
        if(cur.size() == k) {
            res.add(new ArrayList<>(cur));
            return;
        }
        if(idx == n) {
            return;
        }
        // 剪枝：如果当前cur元素+后面的所有元素, 长度都不够k个的话, 那么就不用再递归尝试下去了, 直接返回即可
        if(cur.size() + (n - idx) < k) {
            return;
        }

        // 要
        cur.add(nums[idx]);
        dfs(nums, n, k, idx + 1);
        cur.remove(cur.size() - 1);

        // 不要
        dfs(nums, n, k, idx + 1);
    }
}
```

##### 2）直接进行组合枚举 + dfs + 回溯 + 剪枝 | O（C[n，k] * k）

- **思路**：在 1 的基础上进行优化，去掉元素数组，直接在 [1, n] 上进行枚举即可。
- **结论**：时间，1 ms，99.99%，空间，42.6 mb，63.79%，时间上，由于dfs 存在 C（n，k）种选择方式，表示在 n 个当中选 k 个，而每次 `new ArrayList<>(cur)` 花费 O（k），所以整体时间复杂度为 O（C[n，k] * k），空间上，cur 最大长 n，dfs 最大栈深度为 k，所以整体额外空间复杂度为 O（n + k）。

```java
class Solution {
    private List<List<Integer>> res;
    private List<Integer> cur;

    public List<List<Integer>> combine(int n, int k) {
        this.res = new ArrayList<>();
        this.cur = new ArrayList<>();
        dfs(n, k, 1);
        return this.res;
    }

    private void dfs(int n, int k, int num) {
        if(cur.size() == k) {
            res.add(new ArrayList<>(cur));
            return;
        }
        // 剪枝：如果当前cur元素+后面的所有元素, 长度都不够k个的话, 那么就不用再递归尝试下去了, 直接返回即可
        if(cur.size() + (n - num + 1) < k) {
            return;
        }
        // 由于有了上面的剪枝, 所以这里就不再需要判断是否来到数组末尾了, 因为已经被上面剪掉了
        // if(num == n + 1) {
        //     return;
        // }

        // 要
        cur.add(num);
        dfs(n, k, num + 1);
        cur.remove(cur.size() - 1);

        // 不要
        dfs(n, k, num + 1);
    }
}
```

#### 78. 子集 | medium

##### 1）dfs + 回溯 | O（n * 2^n）

- **思路**：参考《47. 全排列 II》的思路，还是走通用的回溯模板，进行 dfs 枚举使用哪些元素，而 dfs 实现则分要还是不要的判断，但不同的是，这里不是 n 个选 k 个，而是当 n 个数字的状态（选和没选）都确定时，才把 cur 结果加入 res 集合中。
- **结论**：时间，0ms，100%，空间，41.3 mb，78.89%，时间上，由于每个数字状态有 2 种（选和没选），所以组合起来共有 2^n 种，然后每种状态在 `new ArrayList<>(cur)` 需要花费 O（n），所以整体时间复杂度为 O（n * 2^n），空间上，cur 最大长 n，dfs 最大栈深度为 n，所以整体额外空间复杂度为 O（2 * n）。

```java
class Solution {
    private List<List<Integer>> res;
    private List<Integer> cur;

    public List<List<Integer>> subsets(int[] nums) {
        this.res = new ArrayList<>();
        this.cur = new ArrayList<>();

        dfs(nums, nums.length, 0);
        return this.res;
    }

    private void dfs(int[] nums, int n, int idx) {
        if(idx == n) {
            res.add(new ArrayList<>(cur));
            return;
        }

        // 要
        cur.add(nums[idx]);
        dfs(nums, n, idx + 1);
        cur.remove(cur.size() - 1);

         // 不要
        dfs(nums, n, idx + 1);
    }
}
```

##### 2）二进制枚举 | O（n * 2^n）

- **思路**：
  1. 观察结果可以发现，子集数量刚好等于 2^n 个，此时如果用 0表示数字没出现, 1表示数字出现，则可以完美表示子集的状态。
  2. 因此，通过枚举 2^n 个状态，然后根据状态上 1 的位置来反着获取对应的数字，比如 010 就表示索引为 1 的数字应该加入子集中。
- **结论**：时间，1ms，24.94%，41.9 mb，5.11%，时间上，由于枚举了 2^n 个，每次枚举都需要遍历 n 位，所以时间复杂度为 O（n * 2^n），空间上，由于额外使用了一个 cur 数组，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        int n = nums.length, len = 1 << n;
        List<List<Integer>> res = new ArrayList<>(len);
        List<Integer> cur = new ArrayList<>();

        // 一共有2^n个不同序列mask
        for(int mask = 0; mask < len; mask++) {
            cur.clear();

            // 1左移idx位, 取出mask下, 每个位为1的数字
            for(int idx = 0; idx < n; idx++) {
                // 如果mask的idx位置为1, 那么就将nums[idx]加入集合中
                if((mask & (1 << idx)) != 0) {
                    cur.add(nums[idx]);
                }
            }

            // 收集完当前序列, 继续收集下一个序列
            res.add(new ArrayList<>(cur));
        }

        return res;
    }
}
```

#### 79. 单词搜索 | medium

##### 1）回溯 | O（n * m * s）

- **思路**：
  1. 设计一个 f（end，row，col）函数，代表 [0...end] 的字符是否连续存在矩阵中。
  2. f（end，row，col）通过判断上、下、左、右的字符是否存在矩阵中来实现，只有字符相同且还没使用的才算存在。
  3. 要注意的是，由于递归是自上而下的，且要求矩阵中的字符不能重复使用，所以在顶层判断好后需要先设置为已使用，然后调用子过程才不会被重复使用，而这个尝试在子过程判断到不存在时，还需要在回溯时把它清除掉，以方便进行下一个尝试。
- **结论**：时间，60ms，96.47%，空间，36.1mb，95.07%，效率非常高，由于极端情况下，需要在矩阵的每个位置都尝试一次 O（n * m），而每次尝试又需要找出字符 s 中的每个字符  O（s），因此，时间复杂度最多需要 O（n * m * s）。

```java
class Solution {
    public boolean exist(char[][] board, String word) {
        if(word == null || word.length() == 0) {
            return true;
        }
        if(board == null || board.length == 0 || board[0].length == 0) {
            return false;
        }

        boolean[][] used = new boolean[board.length][board[0].length];
        char[] chars = word.toCharArray();

        for(int i = 0; i < board.length; i++) {
            for(int j = 0; j < board[0].length; j++) {
                if(f(board, used, chars, chars.length-1, i, j)) {
                    return true;
                }
            }
        }

        return false;
    }

    // f代表[0...end]的字符是否连续存在矩阵中
    private boolean f(char[][] board, boolean[][] used, char[] chars, int end, int row, int col) {
        if(row < 0 || row >= board.length || col < 0 || col >= board[0].length) {
            return false;
        }
        if(end == 0) {
            if(!used[row][col] && board[row][col] == chars[end]) {
                return true;
            } else {
                return false;
            }
        }

        // 当前位置是否存在
        if(!used[row][col] && board[row][col] == chars[end]) {

            // 先认为已经使用了
            used[row][col] = true;

            // 且子串也存在时
            if(
                // 上
                f(board, used, chars, end-1, row-1, col)
                // 下
                || f(board, used, chars, end-1, row+1, col)
                // 左
                || f(board, used, chars, end-1, row, col-1)
                // 右
                || f(board, used, chars, end-1, row, col+1)) {
                // 则才认为存在
                return true;
            }

            // 如果不存在, 则在回溯时恢复正常
            used[row][col] = false;
        }

        return false;
    }
}
```

#### 90. 子集 II | medium

##### 1）dfs + 回溯 | O（n * logn + n * 2^n）

- **思路**：在《78. 子集》的基础上，增加去重判断，即先顺序排序，dfs 时判断，如果前面一个数与当前数相等, 且没被选择，那么当前数不能选，防止重复。
- **结论**：时间，1 ms，99.66%，空间，41.6 mb，62.67%，时间上，由于排序花费 O（n * logn），且在数字不重复时，每个数字状态最多有 2 种（选和没选），所以组合起来共有 2^n 种，然后每种状态在 `new ArrayList<>(cur)` 需要花费 O（n），所以整体时间复杂度为 O（n * logn + n * 2^n），空间上，快速排序递归栈深度为 logn，cur 最大长 n，dfs 最大栈深度为 n，所以整体额外空间复杂度为 O（logn + 2 * n）。

```java
class Solution {
    private List<List<Integer>> res;
    private List<Integer> cur;

    public List<List<Integer>> subsetsWithDup(int[] nums) {
        this.res = new ArrayList<>();
        this.cur = new ArrayList<>();

        // 先顺序排序
        Arrays.sort(nums);

        // 再dfs + 回溯
        dfs(nums, nums.length, 0, false);
        return this.res;
    }

    private void dfs(int[] nums, int n, int idx, boolean hasPre) {
        if(idx == n) {
            res.add(new ArrayList<>(cur));
            return;
        }

        // 要: 如果前面一个数与当前数相等, 且没被选择, 那么当前数不能选, 防止重复
        // 即只同一类的数字, 选的话都优先选前面的, 包括100,110,111, 而001,011与100和110重复了, 所以不选
        if(!(idx - 1 > -1 && nums[idx - 1] == nums[idx] && !hasPre)) {
            cur.add(nums[idx]);
            dfs(nums, n, idx + 1, true);
            cur.remove(cur.size() - 1);
        }

        // 不要
        dfs(nums, n, idx + 1, false);
    }
}
```

##### 2）二进制枚举 | O（n * 2^n）

- **思路**：同样，在《78. 子集》的基础上，增加去重判断，即先顺序排序，但在判断到 mask 位为 1，加入 cur 前，先判断如果前面一个数与当前数相等，且没被选择，那么当前数不能选，防止重复。
- **结论**：时间，2 ms，12.52%，41.6 mb，52.95%，时间上，由于最多需要枚举 2^n 个，每次枚举都需要遍历 n 位，所以时间复杂度为 O（n * 2^n），空间上，由于额外使用了一个 cur 数组，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public List<List<Integer>> subsetsWithDup(int[] nums) {
        // 先顺序排序
        Arrays.sort(nums);

        int n = nums.length, len = 1 << n;
        List<List<Integer>> res = new ArrayList<>(len);
        List<Integer> cur = new ArrayList<>();

        // 一共有2^n个不同序列mask
        boolean flag;
        for(int mask = 0; mask < len; mask++) {
            flag = true;
            cur.clear();

            // 1左移idx位, 取出mask下, 每个位为1的数字
            for(int idx = 0; idx < n; idx++) {
                // 如果mask的idx位置为1, 那么就将nums[idx]加入集合中
                if((mask & (1 << idx)) != 0) {
                     // 但如果前面一个数与当前数相等, 且没被选择, 那么当前数不能选, 防止重复
                     // 即只同一类的数字, 选的话都优先选前面的, 包括100,110,111, 而001,011与100和110重复了, 所以不选
                    if(idx - 1 > -1 && nums[idx - 1] == nums[idx] && (((mask >> (idx - 1)) & 1) == 0)) {
                        flag = false;
                        break;
                    } else {
                        cur.add(nums[idx]);
                    }
                }
            }

            // 收集完当前序列, 继续收集下一个序列
            if(flag) {
                res.add(new ArrayList<>(cur));
            }   
        }

        return res;
    }
}
```

#### 93. 复原 IP 地址 | medium

##### 1）dfs + 回溯 + 位数枚举 | O（3^4 * 4）

- **思路**：
  1. 参考《47. 全排列 II》的思路，还是走通用的回溯模板，其中，dfs 分使用 1、2、3 个字符情况讨论，符合的则加入 cur，然后继续 dfs 剩余字符。
  2. 当所有字符都刚好用完，且 cur 长度为 4 的时候，才把 cur 结果组装到 res 中，否则直接返回给上层、让上层回溯掉、然后继续下一个尝试。
- **结论**：时间，1 ms，95.18%，空间，41.6 mb，41.31%，时间上，由于 ip 地址一共有 4 段，所有 dfs 最深有 4 层，每段 ip 数字最大有 3 位，所以每层的 dfs 状态有 3 种，组合起来一共有 3^4 种，每种状态最后组装 cur 到 res 时花费 O（4），所以整体时间复杂度为 O（3^4 * 4），空间上，由于 dfs 递归栈最深有 4 层，所以额外空间复杂度为 O（4）。

```java
class Solution {
    private List<String> res;
    private List<String> cur;

    public List<String> restoreIpAddresses(String s) {
        this.res = new ArrayList<>();
        this.cur = new ArrayList<>();
        dfs(s, s.length(), 0);
        return res;
    }

    private void dfs(String s, int n, int idx) {
        if(idx >= n) {
            // 字符已用完, 但还没够4个, 说明分割错了, 直接返回即可
            if(cur.size() != 4) {
                return;
            }

            // 否则说明分割正确, 加入结果集即可
            StringBuilder sb = new StringBuilder();
            for(String str : cur) {
                sb.append(str).append(".");
            }
            res.add(sb.toString().substring(0, sb.length() - 1));
            return;
        }
        // 字符还没用完, 但已经有4个了, 说明分割错了, 直接返回即可
        if(cur.size() == 4) {
            return;
        }

        // 当前位字符
        char c1 = s.charAt(idx);
        StringBuilder sb = new StringBuilder();
        sb.append(c1);

        // 只使用1位z
        cur.add(sb.toString());
        dfs(s, n, idx + 1);
        cur.remove(cur.size() - 1);

        // 使用2位
        if(idx + 1 < n) {
            // c1+c2不合法, 则直接返回
            char c2 = s.charAt(idx + 1);
            if(!isValid(c1, c2)) {
                return;
            }

            sb.append(c2);
            cur.add(sb.toString());
            dfs(s, n, idx + 2);
            cur.remove(cur.size() - 1);

            // 使用3位
            if(idx + 2 < n) {
                // c1+c2+c3不合法, 则直接返回
                char c3 = s.charAt(idx + 2);
                if(!isValid(c1, c2, c3)) {
                    return;
                }

                sb.append(c3);
                cur.add(sb.toString());
                dfs(s, n, idx + 3);
                cur.remove(cur.size() - 1);
            }
        }
    }

    // 判断c1+c2是否合法：[0, 255], 不能有前导0
    private boolean isValid(char c1, char c2) {
        int val1 = c1 - '0';
        int val2 = c2 - '0';
        
        // 有前导0
        if(val1 == 0) {
            return false;
        }

        // c1+c2
        int val = val1 * 10 + val2;
        return val >= 0 && val <= 255;
    }

    // 判断c1+c2+c3是否合法：[0, 255], 不能有前导0
    private boolean isValid(char c1, char c2, char c3) {
        int val1 = c1 - '0';
        int val2 = c2 - '0';
        int val3 = c3 - '0';

        // 有前导0
        if(val1 == 0) {
            return false;
        }

        // c1+c2+c3
        int val = val1 * 100 + val2 * 10 + val3;
        return val >= 0 && val <= 255;
    }
}
```

#### 301. 删除无效的括号 | hard

##### 1）回溯 + 剪枝 | O（n * 2^n）

- **思路**：
  1. 按题意，字符串 s 中可能存在小写字母、'('、')' 三种字符，如果需要获取合法的字符串，那么小写字母可以不用管，跳过就好；其他的则还要判断左括号是否匹配，如果不匹配的，则删除多余的括号。
  2. 这里删除最少的多余的括号，可以走统一的算法，即：
     1. 从左遍历到右，碰到左括号，则左待删除+1。
     2. 碰到右括号，如果左待删除为 0，说明前面没有左括号，则右待删除+1，否则说明匹配，要消耗掉一个左括号和右括号，则左待删除-1，右待删除不变。
     3. 最后，返回左待删除和右待删除，表示最少要删除的左括号数和右括号数。
  3. 然后是判断字符串是否为合法的括号串，这可以参考《动态规划 - 最长有效括号》中的判断方法，即从左到右，遍历到左括号则 count++，遍历到右括号则 count--，并且判断此时 count 是否 0，最后还要判断 count 是否为 0。
  4. 再然后就是递归回溯函数的实现了，由于这里要做的是枚举各个字符串的子序列，然后做括号合法性判断，所以需要根据左待删除和右待删除的数量来进行递归，如果待删除的数量都为 0，说明可以做括号合法性判断了，否则还需要尝试删除 0,1,2,...,n-1 位置上的左括号或者右括号...
  5. 其中，回溯过程中的剪枝过程比较重要，可以跳过很多无用的判断，比如如果前面的括号形状和当前的一样，说明当前括号也是必定要留下的，因为如果当前括号要删除，那么在之前早就被删了，所以跳过尝试即可。
- **结论**：时间，3ms，92.98%，空间，41.7mb，5.00%，时间上，由于 n 个字符的子序列有 2^n 种，对其中一种做合法性校验需要花费 O（n），所以时间复杂度为 O（n * 2^n），空间上，由于需要尝试删除 0,1,2,...n-1 位置上的括号，所以额外空间复杂度取决于递归深度，为 O（n）。

```java
class Solution {
    public List<String> removeInvalidParentheses(String s) {
        if(s == null || s.length() == 0) {
            return new ArrayList<>();
        }

        List<String> res = new LinkedList<>();
        int[] lrRemovingCounts = getLrRemovingCount(s);
        f(s, res, 0, lrRemovingCounts[0], lrRemovingCounts[1]);
        return res;
    }

    private void f(String s, List<String> res, int start, int lremove, int rremove) {
        if(lremove == 0 && rremove == 0) {
            if(isValid(s)) {
                res.add(s);
            }
            return;
        }
        
        for(int i = start; i < s.length(); i++) {
            // 由于删除括号后i保持不变, 所以这里剪枝的意思是, 如果前后两个括号相同, 那么删除前和删除后是一样的, 所以这里只处理第一个出现的括号, 后面重复出现的就不再处理了
            if (i != start && s.charAt(i) == s.charAt(i-1)) {
                continue;
            }
            // 如果剩余字符无法满足需要去掉的左括号和右括号数, 则直接返回即可, 无需继续尝试
            if(lremove + rremove > s.length() - i) {
                return;
            }

            // 尝试去掉一个左括号: 由于是字符串截取, 所以截取后还是从i开始, 而不是i+1
            if(lremove > 0 && s.charAt(i) == '(') {
                f(s.substring(0, i) + s.substring(i+1), res, i, lremove-1, rremove);
            }

            // 尝试去掉一个右括号: 由于是字符串截取, 所以截取后还是从i开始, 而不是i+1
            if(rremove > 0 && s.charAt(i) == ')') {
                f(s.substring(0, i) + s.substring(i+1), res, i, lremove, rremove-1);
            }
        }
    }

    private boolean isValid(String s) {
        int count = 0;

        for(int i = 0; i < s.length(); i++) {
            if(s.charAt(i) == '(') {
                count++;
            } else if(s.charAt(i) == ')') {
                count--;
                if(count < 0) {
                    return false;
                }
            }
        }

        return count == 0;
    }

    private int[] getLrRemovingCount(String s) {
        int lremove = 0, rremove = 0;
        
        for(int i = 0; i < s.length(); i++) {
            if(s.charAt(i) == '(') {
                lremove++;
            } else if(s.charAt(i) == ')') {
                if(lremove == 0) {
                    rremove++;
                } else {
                    lremove--;
                }
            }
        }

        return new int[] {lremove, rremove};
    }   
}
```

#### 394. 字符串解码 | medium

##### 1）递归法 | O（n）

- **思路**：
  1. 设计一个 f（index）函数，代表处理并返回 [index,  ']' 或者结尾]位置解析后的字符串。
  2. f 函数的实现需要从 index 处开始遍历 s 字符串，一共有 5 种情况：
     1. 如果碰到数字字符，则转换为倍数，0 倍表示字符串原样输出，其中需要注意 10+ 倍连续数字的出现。
     2. 如果碰到左括号字符，则把剩余遍历工作交由子过程去处理，处理完后把其返回结果乘上第 1 步得到的倍数，再与当前返回结果做拼接，然后当前递归继续处理子过程处理后剩余的位置。
     3. 如果碰到小写字母，则把小写字母加入当前返回结果中。
     4. 如果碰到右括号，则把当前处理到的位置记录在 Info 变量上，以告诉父过程什么是已经处理过的，然后把当前处理结果返回，以让父过程做字符串拼接。
     5. 如果当前递归遍历完毕，也没有碰到右括号，说明当前为第一次递归函数，则当前处理结果返回即可。
- **结论**：时间，0ms，100%，空间，39.2mb，6.03%，时间上，虽然递归深度可能会很深，但字符串 s 也就只会被遍历 1 遍，所以时间复杂度为 O（n），空间上，额外空间取决于递归深度，其最坏情况为层层嵌套，记最多嵌套 m 层，所以额外空间复杂度为 O（m）。

```java
class Solution {

    class Info {
        int index;
        Info(int index) {
            this.index = index;
        }
    }

    public String decodeString(String s) {
        if(s == null || s.length() == 0) {
            return "";
        }
        return f(s, new Info(0));
    }

    // f代表处理并返回[index, ']'或者结尾]位置解析后的字符串
    private String f(String s, Info info) {
        if(info.index == s.length()) {
            return "";
        }

        int mul = 0;
        char c;
        StringBuilder sb = new StringBuilder();
        for(int i = info.index; i < s.length(); i++) {
            c = s.charAt(i);
            
            // 数字 => 转换为倍数
            if(c >= '0' && c <= '9') {
                mul = mul * 10 + (c - '0');
            }
            // 左括号 => 后面的交给子过程去处理
            else if(c == '[') {
                info.index = i + 1;
                sb.append(
                    process(f(s, info), mul)
                );
                i = info.index;
                mul = 0;
            }
            // 小写字母 => 加入当前递归字符集
            else if(c >= 97 && c <= 122) {
                sb.append(c);
            }
            // 右括号 => 直接返回当前递归收集到的字符串
            else if(c == ']') {
                info.index = i;
                return sb.toString();
            }
        }

        // 遍历完也没遇到右括号, 说明被消化完了, 则直接返回
        return sb.toString();
    }

    private String process(String s, int k) {
        if(s == null || s.length() == 0) {
            return "";
        }
        if(k == 0) {
            return s;
        }

        StringBuilder sb = new StringBuilder();
        while(k > 0) {
            sb.append(s);
            k--;
        }

        return sb.toString();
    }
}
```

##### 2）迭代法 | O（n）

- **思路**：跟递归法的思路一样，迭代法的思路也是从头到尾遍历字符串 s，也共分为 5 种情况：
  1. 如果碰到数字字符，则转换为倍数，0 倍表示字符串原样输出，其中需要注意 10+ 倍连续数字的出现。
  2. 如果碰到左括号字符，则初始化好一个字符集并加入字母栈中，以及把收集到的倍数加入倍数栈中，然后清空当前倍数，以继续收集其他处理集的倍数。
  3. 如果碰到小写字母，则如果字母栈不为空，说明为栈顶处理集的字母，则把该小写字母加入字母栈中；如果字母栈为空，说明为顶层处理集的字母，则把该小写字母加入当前处理集中。
  4. 如果碰到右括号，则说明为栈顶处理集的右括号（字母栈肯定不为空），也就是栈顶处理集到尽头了，则字母栈栈顶和倍数栈栈顶弹出，调用 process 处理函数加工乘上倍数后的字符串；如果弹出后字母栈不为空，说明为栈顶处理集的字母，则把该小写字母加入字母栈中；如果字母栈为空，说明为顶层处理集的字母，则把该小写字母加入当前处理集中。
  5. 如果当前递归遍历完毕，说明顶层处理集处理完毕，则返回处理结果即可。
- **结论**：时间，0ms，100%，39.4mb，5.01%，时间上，由于只需要遍历 1 次字符串 s，所以时间复杂度为 O（n），空间上，由于使用了两个栈，最坏情况下层层嵌套的话，记栈的最大深度为 m，此时的额外空间复杂度为 O（m）。

```java
class Solution {

    public String decodeString(String s) {
        if(s == null || s.length() == 0) {
            return "";
        }

        LinkedList<Integer> mulStack = new LinkedList<>();
        LinkedList<StringBuilder> sbStack = new LinkedList<>();

        int mul = 0;
        char c;
        String tmp;
        StringBuilder sb = new StringBuilder();
        for(int i = 0; i < s.length(); i++) {
            c = s.charAt(i);
            
            // 数字 => 转换为倍数
            if(c >= '0' && c <= '9') {
                mul = mul * 10 + (c - '0');
            }
            // 左括号 => 入栈字母栈和倍数栈
            else if(c == '[') {
                sbStack.push(new StringBuilder());
                mulStack.push(mul);
                mul = 0;
            }
            // 小写字母
            else if(c >= 97 && c <= 122) {
                // 加入栈顶的字符集
                if(!sbStack.isEmpty()) {
                    sbStack.peek().append(c);
                } 
                // 加入当前收集的字符集
                else {
                    sb.append(c);
                }
            }
            // 右括号
            else if(c == ']') {
                if(sbStack.isEmpty() || mulStack.isEmpty()) {
                    continue;
                }

                // 加入栈顶的字符集
                tmp = process(sbStack.poll().toString(), mulStack.poll());
                if(!sbStack.isEmpty() && !mulStack.isEmpty()) {
                    sbStack.peek().append(tmp);
                } 
                // 加入当前收集的字符集
                else {
                    sb.append(tmp);
                }
            }
        }

        // 字符串遍历完毕
        return sb.toString();
    }

    private String process(String s, int k) {
        if(s == null || s.length() == 0) {
            return "";
        }
        if(k == 0) {
            return s;
        }

        StringBuilder sb = new StringBuilder();
        while(k > 0) {
            sb.append(s);
            k--;
        }

        return sb.toString();
    }
}
```

#### 2257. 第 77 双周赛 - 统计网格图中没有被保卫的格子数 | medium

##### 1）感染法 | O（2 * m * n）

- **思路**：遍历 matrix 中每个格子，然后采用感染法，为每个方向的格子染色，如果碰到 W / G，则说明视线被挡住了，那么就退出当前方向的格子染色循环，进行下一个方向的染色，周而复始。
- **结论**：时间，18 ms，87.06%，空间，58.5 mb，81.57%，时间上，由于染色只会在每个格子上"染一次"，花费 O（m * n），以及在最后统计没染色的格子也要花费 O（m * n），所以时间复杂度为 O（2 * m * n），空间上，由于建立了一张 m * n 的 matrix 数组，且递归深度最深也为 m * n，所以额外空间复杂度为 O（2 * m * n）。

```java
class Solution {
    public int countUnguarded(int m, int n, int[][] guards, int[][] walls) {
        if(guards == null || guards.length == 0 || walls == null || walls.length == 0) {
            throw new RuntimeException("参数有误!");
        }
        
        int row, col;
        int[][] matrix = new int[m][n];// 0
        for(int i = 0; i < guards.length; i++) {
            row = guards[i][0];
            col = guards[i][1];
            matrix[row][col] = 1;// G
        }
        for(int i = 0; i < walls.length; i++) {
            row = walls[i][0];
            col = walls[i][1];
            matrix[row][col] = -1;// W
        }
        
        for(int i = 0; i < m; i++) {
            for(int j = 0; j < n; j++) {
                if(matrix[i][j] == 1) {
                    infects(matrix, m, n, i, j);
                }
            }
        }
        
        int count = 0;
        for(int i = 0; i < m; i++) {
            for(int j = 0; j < n; j++) {
                if(matrix[i][j] == 0) {
                    count++;
                }
            }
        }
        
        return count;
    }
    
    private void infects(int[][] matrix, int m, int n, int i, int j) {
        // 上下左右
        for(int k = i - 1; k > -1; k--) {
            // W / G
            if(matrix[k][j] == -1 || matrix[k][j] == 1) {
                break;
            } 
            
            // 标记
            matrix[k][j] = -2;
        }
        for(int k = i + 1; k < m; k++) {
            // W / G
            if(matrix[k][j] == -1 || matrix[k][j] == 1) {
                break;
            } 

            // 标记
            matrix[k][j] = -2;
        }
        for(int k = j - 1; k > -1; k--) {
             // W / G
            if(matrix[i][k] == -1 || matrix[i][k] == 1) {
                break;
            } 

            // 标记
            matrix[i][k] = -2;
        }
        for(int k = j + 1; k < n; k++) {
            // W / G
            if(matrix[i][k] == -1 || matrix[i][k] == 1) {
                break;
            } 

            // 标记
            matrix[i][k] = -2;
        }
    }
}
```

#### 2322. 从树中删除边的最小分数 | hard

##### 1）DFS + 时间戳 + 分割树 | O（m + n + n^2）

- **思路**：

  1. 先根据 edges 边集数组，建立图数组。

  2. 然后，DFS + 时间戳遍历图，建立 int、out 时间戳数组，用来判断两点的是否存在祖先关系，可证明：如果 y 是 x 的子孙节点，那么区间 [ [in[y]、out[y] ] 必然被区间 [ in[x]，out[x] ] 所包含，并且，如果 区间 [ [in[y]、out[y] ] 必然被区间 [ in[x]，out[x] ] 所包含，也可证明出 y 是 x 的子孙节点。所以，in[x] < in[y] <= out[y] <= out[x]，由于 in[y] 恒小于等于 out[y] ，因此，推导式可化简为 in[x] < int[y] <= out[x]。这里之所以用 <=，是因为代码里退出递归时 clk 并不会更新。

  3. 同时，在遍历图的过程中，还累计异或和数组，用来计算分割后的区域异或和。

  4. 最后，枚举 i、j 切割点 y1、y2，分析切割出的三个区域：

     - 1）枚举 y1、y2 切割点，如果 y1 是 y2 的祖先节点，那么分割区域为 0~y1-1、y1~y2-1，y2 三个，这里 -1 的意思不是索引 -1，而是区域往前推 1 块的意思，异或和则分别为 xors[0]^xors[y1]、xors[y1]^xors[y2]、xors[y2]，这里之所以异或，是利用异或的相同异或为 0、与 0 异或为本身的性质：

       ![1656915223854](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1656915223854.png)

     - 2）同理，枚举 y1、y2 切割点，如果 y2 是 y1 的祖先节点，那么分割区域为 0~y2-1、y2~y1-1，y1 三个，异或和则分别为 xors[0]^xors[y2]、xors[y2]^xors[y1]、xors[y1]。

     - 3）还有一种情况就是，y1 既不是 y2 的祖先节点，y2 也不是 y1 的祖先节点，那么分割区域为 0~( y1-1，y2 - 1)、y1、y2：

       ![1656915421248](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1656915421248.png)

- **结论**：时间，26 ms，89.48%，空间，41.6 mb，74.21%，时间上，建立图花费 O（m）、dfs 遍历图花费 O（n），枚举切割点花费 O（n^2），所以时间复杂度为 O（m + n + n^2），空间上，建立图使用一张 m^2 的二维表，ins、outs、xors 共花费了 O（3 * n）的数组，dfs 递归栈深度最大为 O（n），所以额外空间复杂度为 O（m^2 + 4 * n）。

```java
class Solution {

    private List<Integer>[] lists;
    private int[] ins;
    private int[] outs;
    private int[] nums;
    private int[] xors;
    private int clk;

    public int minimumScore(int[] nums, int[][] edges) {
        int n = nums.length;
        lists = new ArrayList[n];
        Arrays.setAll(lists, o -> new ArrayList<>());

        // 建立图
        for(int[] edge : edges) {
            int x = edge[0], y = edge[1];
            lists[x].add(y);
            lists[y].add(x);
        }

        // dfs遍历图, 建立int、out时间戳数组, 用来判断两点的是否存在父子关系
        // 同时, 建立异或和数组, 用来计算分割后的区域异或和
        ins = new int[n];// 进入rt节点的时间戳
        outs = new int[n];// 离开rt节点的时间戳
        xors = new int[n];// 以rt节点为根的整棵树的异或和
        this.nums = nums;// 原始数组
        clk = 0;// 当前时间戳
        dfs(0, -1);

        // 由于n不大, 所以可以枚举任意两点, 判断去掉为根的线段
        int min = Integer.MAX_VALUE;

        // 枚举切割之后的根节点i, 所以不用枚举0, 因为0肯定不被切割, 所以从1开始枚举
        for(int i = 1, x, y, z; i < n; i++) {
            // 枚举切割之后的根节点j, 因为0肯定不被切割, 且在i的基础上进行枚举, 所以从i+1开始
            for(int j = i + 1; j < n; j++) {
                // i是j的祖先节点, 则分为三个区域, 0~i-1, i~j-1, j~n-1
                // 所以三个区域的异或和, 分别为xors[0]^xors[i]、xors[u]^xors[j]、xors[j]
                if(ins[i] < ins[j] && ins[j] <= outs[i]) {
                    x = xors[j];
                    y = xors[i] ^ x;
                    z = xors[0] ^ xors[i];
                }
                // j是i的祖先节点, 则分为三个区域, 0~j-1、j~i-1, i~n-1
                // 所以三个区域的异或和, 分别为xors[0]^xors[j]、xors[j]^xors[i]、xors[i]
                else if(ins[j] < ins[i] && ins[i] <= outs[j]) {
                    x = xors[i];
                    y = xors[j] ^ x;
                    z = xors[0] ^ xors[j];
                }
                // i、j没有祖先关系, 则分为三个区域0~(i-1, j-1), i~n-1, j~n-1
                // 所以三个区域的异或和, 分别为xors[0]^xors[i]^xors[j]、xors[i]、xors[j]
                else {
                    x = xors[i];
                    y = xors[j];
                    z = xors[0] ^ x ^ y;
                }

                // 最小结果 = 最小的(最大的区域异或和 - 最小的区域异或和)
                min = Math.min(min, Math.max(Math.max(x, y), z) - Math.min(Math.min(x, y), z));
                
                // min == 0, 则提前退出, 因为已经是最小的了, 无需继续比较
                if(min == 0) {
                    return 0;
                }
            }
        }

        return min;
    }

    // dfs遍历图, 建立int、out时间戳数组, 用来判断两点的是否存在父子关系
    // 同时, 建立异或和数组, 用来计算分割后的区域异或和
    private void dfs(int rt, int fa) {
        // 节点进入, 当前时间戳+1
        ins[rt] = ++clk;

        // 异或和从根节点开始
        xors[rt] = nums[rt];

        // 遍历孩子节点
        for(int sub : lists[rt]) {
            // 注意, 由于lists图是双向的, 一定要让sub孩子节点不能等于rt自己
            if(sub != fa) {
                dfs(sub, rt);

                // 异或和, 异或行孩子节点的值
                xors[rt] ^= xors[sub];
            }
        }

        // 如果为叶子节点, 那么出节点的时间戳=入节点的时间戳
        // 否则, 入节点的时间戳=dfs前的时间戳, 出节点的时间戳=dfs完的时间戳=出最后一个孩子节点的时间戳
        outs[rt] = clk;
    }
}
```

### 3.1. 算法思想 - BFS

#### 2258. 第 77 双周赛 - 逃离火灾 | hard

##### 1）二分 + BFS | O（log[m * n] * [m * n]）

- **思路**：

  1. 设定一个 canRunAfterWait(int mins) 的函数，代表等待 mins 后，才开始从（0，0）跑起来，也就是在这之前，火势已经蔓延了 mins 次，其中，对于火势，很明显地，应该采用 BFS 进行蔓延。
  2. 而对于人跑来说，用于要尝试每种可能，所以也应该采用 BFS 模型，因为这可以把下次的可能都记录在案，然后 BFS 去尝试，以及获取更多的下下次可能。
  3. 至于 BFS 的思路，就应该想到用队列，这里就是不断地把下次地可能装进队列，然后不断取出，但与树的 BFS 不同，这里取出以后，并不会再装回去，因为是用了一个 fireMatrix 和 manMatrix 的 boolean 值进行记录了，省去了大量的重复 BFS 操作。
  4. 然后就是 BFS 的实现了，这里用的是一个 BFS 因子（上，下，左，右）数组，在做对应的 BFS 数组时，直接遍历这个 BFS 因子，i、j 与因子对应相加，即可得到 BFS 后的 i 和 j 了，对比几次 if 来说，这种写法非常方便。
  5. 最后就是，不断尝试 canRunAfterWait(int mins) 函数了，由于至少有 m * n 可能，所以，如果都尝试一遍，性能肯定不高，但如果理解了尝试的含义，就发现：比如等 3 min 成功，那就尝试等 4 min 甚至更大，但如果等 3 min 失败，那就尝试 2 min 甚至更小，即有尝试一边后，就会放弃另一边的思想，也就是可以用二分来大大减少尝试次数。 

  ![1653665123961](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1653665123961.png)

- **结论**：

  1. 时间，80 ms，16.19%，空间，41.8 mb，99.59%。
  2. 时间上，由于至少有 m * n 种可能，二分计算需要尝试 log（m * n）次，每次最多需要 BFS 遍历完整个二维表，花费 m * n 次遍历，所以时间复杂度为 O（log[m * n] * [m * n]）。
  3. 空间上，由于火势用了 fireQueue 队列和 fireMatrix 二维数组，人跑用了 manQueue 队列和 manMatrix 二维数组，它们最大都是等于 m * n，所以额外空间复杂度为 O（4 * m * n）。

```java
class Solution {

    // BFS因子 => 上下左右
    private static final int[][] BFS_FACTOR = new int[][] {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};

    public int maximumMinutes(int[][] grid) {
        if(grid == null || grid.length == 0 || grid[0] == null || grid[0].length == 0) {
            return -1;
        }

        // lwait保底, 初始为-1, 代表不能走到安全屋
        int lwait = -1, rwait = grid.length * grid[0].length, mwait;
        while(lwait < rwait) {
            // 二分多加1, 防止每次都成功导致的偏左死循环
            mwait = lwait + ((rwait - lwait) >>> 1) + 1;
            
            // 等成功, 则可能还能再等, 继续往前看, 但保底lwait
            if(canRunAfterWait(grid, mwait)) {
                lwait = mwait;
            } 
            // 等失败, 则不能再等, 回头看
            else {
                rwait = mwait - 1;
            }
        }

        // 如果等待了所有格子数-1, 都还能走到安全屋, 则说明可以无限地等下去
        return lwait < grid.length * grid[0].length? lwait : (int) 1e9;
    }

    // 等待mins分钟后再跑
    private boolean canRunAfterWait(int[][] grid, int mins) {
        Queue<int[]> fireQueue = initFireQueue(grid);
        boolean[][] fireMatrix = initFireMatrix(grid);
        
        // 先蔓延mins分钟火势
        while(mins-- > 0) {
            fireQueue = spreadFire(fireQueue, fireMatrix, grid);
        }

        // 等待mins完毕, 开始奔跑
        Queue<int[]> manQueue = initManQueue(grid), tmpQueue;
        boolean[][] manMatrix = initManMatrix(grid);

        // 如果还有下次可能的机会, 则继续遍历
        while(!manQueue.isEmpty()) {
            tmpQueue = manQueue;
            manQueue = new LinkedList<>();

            // tmpQueue也要遍历, 防止多种情况只走了1种, 没走完的情况 
            while(!tmpQueue.isEmpty()) {
                int[] cur = tmpQueue.poll();

                // 如果已经着火, 则认为已经被火烧到了, 上一步BFS失败
                if(fireMatrix[cur[0]][cur[1]]) {
                    continue;
                }

                // 人先走
                for(int[] dfs : BFS_FACTOR) {
                    int i = cur[0] + dfs[0];
                    int j = cur[1] + dfs[1];

                    // 索引非法, 则不再进入
                    if(i < 0 || i >= grid.length || j < 0 || j >= grid[0].length) {
                        continue;
                    }
                    // 碰到墙, 或者已经着火, 或者已经进入过, 则不再进入
                    if(grid[i][j] == 2 || fireMatrix[i][j] || manMatrix[i][j]) {
                        continue;
                    }
                    // 成功走到安全屋: 放在火校验后面, 保证安全屋不会着火
                    if(i == grid.length - 1 && j == grid[0].length - 1) {
                        return true;
                    }

                    manQueue.add(new int[] {i, j});
                    manMatrix[i][j] = true; 
                }

                // 不要再tmpQueue尝试过程中进行火势蔓延, 这从业务上来说是不合理的!
            }

            // tmpQueue所有情况都尝试完成后, 再开始火再蔓延
            fireQueue = spreadFire(fireQueue, fireMatrix, grid);
        }

        // 如果没有下次可能的机会, 还没能到安全屋, 则认为失败
        return false;
    }

    // 火势蔓延一分钟
    private Queue<int[]> spreadFire(Queue<int[]> srcQueue, boolean[][] fireMatrix, int[][] grid) {
        Queue<int[]> targetQueue = new LinkedList<>();
        while(!srcQueue.isEmpty()) {
            int[] cur = srcQueue.poll();
            for(int[] dfs : BFS_FACTOR) {
                int i = cur[0] + dfs[0];
                int j = cur[1] + dfs[1];

                // 索引非法, 则不再蔓延
                if(i < 0 || i >= grid.length || j < 0 || j >= grid[0].length) {
                    continue;
                }
                // 碰到墙, 或者已经蔓延过, 则不再蔓延
                if(grid[i][j] == 2 || fireMatrix[i][j]) {
                    continue;
                }

                targetQueue.add(new int[] {i, j});
                fireMatrix[i][j] = true; 
            }
        }
        return targetQueue;
    }

    // 初始化人访问队列
    private Queue<int[]> initManQueue(int[][] grid) {
        Queue<int[]> manQueue = new LinkedList<>();
        manQueue.offer(new int[] {0, 0});
        return manQueue;
    }

    // 初始化人访问索引
    private boolean[][] initManMatrix(int[][] grid) {
        boolean[][] manMatrix = new boolean[grid.length][grid[0].length];
        manMatrix[0][0] = true;
        return manMatrix;
    }

    // 初始化火格子队列
    private Queue<int[]> initFireQueue(int[][] grid) {
        Queue<int[]> fireQueue = new LinkedList<>();
        for(int i = 0; i < grid.length; i++) {
            for(int j = 0; j < grid[0].length; j++) {
                if(grid[i][j] == 1) {
                    fireQueue.offer(new int[] {i, j});
                }
            }
        }
        return fireQueue;
    }

    // 初始化火格子索引
    private boolean[][] initFireMatrix(int[][] grid) {
        boolean[][] fireMatrix = new boolean[grid.length][grid[0].length];
        for(int i = 0; i < grid.length; i++) {
            for(int j = 0; j < grid[0].length; j++) {
                if(grid[i][j] == 1) {
                    fireMatrix[i][j] = true; 
                }
            }
        }
        return fireMatrix;
    }
}
```

#### 2290. 第 295 周赛 - 到达角落需要移除障碍物的最小数目 | hard

##### 1）BFS + 贪心 + 优先级队列 | O（m * n * 4 * logn）

- **思路**：
  1. 把普通格子看作是权重 0，把墙格子看作是权重 1，然后求左上角到右下角的最小路径，得出的最小和就是最终结果了，即移除墙格子的最少数量。
  2. 这里求最小路径，优先取最小权重小的格子，即墙少的路径，进行 BFS 寻路，所以用到了优先级队列。
  3. 然后，BFS 访问过程则是，通过一个 vis 数组，控制访问过的不会被重复访问，以保证起到类似泛洪的作用，然后新格子累加当前格子的值，即路径和累加，再把新格子加入优先级队列排序，以便开启下一轮 BFS。
- **结论**：时间，163 ms，49.74%，空间，81.4 mb，12.17%，时间上，需要遍历整个矩阵的元素，花费 O（m * n），每次遍历过程，最多会进入 4 次优先级队列，共花费 O（4 * logn），所以时间复杂度为 O（m * n * 4 * logn），空间上，由于使用了最多长 m + n 的优先级队列，所以额外空间复杂度为 O（m + n）。

```java
class Solution {
    
    private static final int[][] BFS_FACTOR = new int[][] { {-1, 0}, {1, 0}, {0, -1}, {0, 1} };

    public int minimumObstacles(int[][] grid) {
        if(grid == null || grid.length == 0 || grid[0] == null || grid[0].length == 0) {
            return 0;
        }

        int m = grid.length, n = grid[0].length;
        boolean[][] vis = new boolean[m][n];
        Queue<int[]> queue = new PriorityQueue<>((o1, o2) -> {
            return grid[o1[0]][o1[1]] - grid[o2[0]][o2[1]];
        });
        
        queue.offer(new int[] {0, 0});
        vis[0][0] = true;

        // bfs
        while(!queue.isEmpty()) {
            int[] cur = queue.poll();
            int x = cur[0];
            int y = cur[1];

            for(int[] bfs : BFS_FACTOR) {
                int xn = x + bfs[0];
                int yn = y + bfs[1];
                if(xn < 0 || xn >= m || yn < 0 || yn >= n || vis[xn][yn]) {
                    continue;
                }

                vis[xn][yn] = true;
                
                // dp
                grid[xn][yn] = grid[x][y] + grid[xn][yn];

                // 注意! 因为是以grid作为优先级队列的排序条件, 所以, 必须是dp更新了grid的值以后, 再入队, 提前入队会导致拿着旧值去排序, 产生错误!
                queue.offer(new int[] {xn, yn});
            }
        }

        return grid[m - 1][n - 1];
    }
}
```

##### 2）BFS + 贪心 + 双端队列 | O（m * n）

- **思路**：在 1 的基础上，进行分析，每次 BFS 过程，需要优先级队列进行排序的，都是碰到了 0 的新给子，所以，这里在碰到为 0 的新格子，则直接从队头加入节点，避免 logn 的排序，否则从队尾加入节点，代表格子优先级最低，最后才处理。
- **结论**：时间，49 ms，98.03%，空间，80.1 mb，13.83%，时间上，需要遍历整个矩阵的元素，花费 O（m * n），所以时间复杂度为 O（m * n），空间上，由于使用了最多长 m + n 的双端队列，所以额外空间复杂度为 O（m + n）。

```java
class Solution {
    
    private static final int[][] BFS_FACTOR = new int[][] { {-1, 0}, {1, 0}, {0, -1}, {0, 1} };

    public int minimumObstacles(int[][] grid) {
        if(grid == null || grid.length == 0 || grid[0] == null || grid[0].length == 0) {
            return 0;
        }


        int m = grid.length, n = grid[0].length;
        boolean[][] vis = new boolean[m][n];
        Deque<int[]> deque = new LinkedList<>();// 这里改用双端队列, 优化堆排序的logn消耗!

        deque.offer(new int[] {0, 0});
        vis[0][0] = true;

        // bfs
        while(!deque.isEmpty()) {
            int[] cur = deque.poll();
            int x = cur[0];
            int y = cur[1];

            for(int[] bfs : BFS_FACTOR) {
                int xn = x + bfs[0];
                int yn = y + bfs[1];
                if(xn < 0 || xn >= m || yn < 0 || yn >= n || vis[xn][yn]) {
                    continue;
                }

                vis[xn][yn] = true;
                
                // 是0的话, 则认为最小路径基准没变, 需要从队头进入, 优先BFS
                if(grid[xn][yn] == 0) {
                    deque.offerFirst(new int[] {xn, yn});
                } 
                // 非0的话, 就是1了, 这时路径权重变大, 需要从队列尾部进入, 最后才BFS
                else {
                    // 由于这里用的是普通队列, 没涉及排序, 所以这里offer在dp前或者后调用都无所谓!
                    deque.offerLast(new int[] {xn, yn});
                }
                
                // dp
                grid[xn][yn] = grid[x][y] + grid[xn][yn];
            }
        }

        return grid[m - 1][n - 1];
    }
}
```

### 3.2. 算法思想 - KMP

#### 28. 实现 strStr() | easy

##### 1）双指针模拟法 | O（n * m）

- **思路**：
  1. 确定好边界条件，为空的、为空串的、匹配串比原串长的，以及原串最多需要遍历的次数。
  2. 然后开始遍历原串，如果字符与匹配串第一个字符不同，则跳过。
  3. 如果字符与匹配串第一个字符相同，则开始匹配，此时遍历匹配串，看是否每个字符都与原串 i 往后的字符相同，是的话则认为匹配，否额认为不匹配。
- **结论**：时间，0 ms，100%，空间，39.2 mb，80.30%，时间上，最多需要遍历 n 次原串，每次遍历过程中又遍历 m 次匹配串，所以时间复杂度为 O（n * m），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int strStr(String haystack, String needle) {
        if(haystack == null || needle == null) {
            return -1;
        }

        int hlen = haystack.length();
        int nlen = needle.length();
        if(nlen == 0) {
            return 0;
        }
        if(hlen == 0 || nlen > hlen) {
            return -1;
        }

        char[] charsH = haystack.toCharArray();
        char[] charsN = needle.toCharArray();

        int j, k;
        for(int i = 0; i < hlen - nlen + 1; i++) {
            if(charsH[i] != charsN[0]) {
                continue;
            }
            
            j = 0;
            k = i;
            while(j < nlen && k < hlen && charsH[k] == charsN[j]) {
                j++;
                k++;
            }
            if(j == nlen) {
                return k - nlen;
            }
        }

        return -1;
    }
}
```

##### 2）KMP 算法 | O（2 * m + 2 * n）

- **思路**：
  1. 经典的 KMP 算法实现，先求 nextArr 数组，然后根据 nextArr 数组加速，主串从左往右的匹配过程。
  2. nextArr 数组的初始化过程在于，判断每个 charsN[i - 1] 与 chars[cn] 是否相同，相同的话，则在 cn 上一次最长相同前缀后缀的长度上 + 1，得到当前 nextArr[i] 的值。如果不相同，那么就回跳 nextArr[cn]，或者如果回跳不了 cn 的话，那么就设置 nextArr[i] 为 0。
  3. 加速主串的匹配过程则是，判断 hi 和 ni 的字符是否相同，相同的话则继续比较，不同的话，就看 ni 能否继续会跳，如果可以，就回跳到上一次最长相同前缀后缀的长度 nextArr[ni] 上。如果不可以，则只能移动 hi，从头开始匹配与  ni 匹配，相当于放弃了加速匹配的过程。
- **结论**：时间，0 ms，100%，空间，39.6 mb，48%，时间上，初始化 nextArr 数组的过程，最多花费 O（2 * m），可通过 i 与 i - cn 的增减过程判断出来，加速主串的匹配过程，最多花费 O（2 * n），可通过 hi 与 hi - ni 的增减过程判断出来 ，所以时间复杂度为 O（2 * m + 2 * n），空间上，由于使用了一个 m 长的 nextArr 数组，所以额外空间复杂度为 O（m）。

```java
class Solution {
    public int strStr(String haystack, String needle) {
        if(haystack == null || needle == null) {
            return -1;
        }

        int hlen = haystack.length();
        int nlen = needle.length();
        if(nlen == 0) {
            return 0;
        }
        if(hlen == 0 || nlen > hlen) {
            return -1;
        }

        char[] charsH = haystack.toCharArray();
        char[] charsN = needle.toCharArray();

        int hi = 0, ni = 0;
        int[] nextArr = getNextArr(charsN);
        
        // O(2 * n)
        while(hi < charsH.length && ni < charsN.length) {
            // 一路匹配, 都各自+1
            if(charsH[hi] == charsN[ni]) {
                hi++;
                ni++;
            }
            // ni == 0, 表示charsN已经无法减少了, 则选择下一个hi, 与charsN从头开始匹配
            else if(ni == 0) {
                hi++;
            } 
            // 如果charsN还可以减少, 那么就来到上一个匹配的地方, charsH保持i不变
            else {
                ni = nextArr[ni];
            }
        }

        // ni越界, 则说明已经匹配出charsN了, 而如果hi越界, 但ni没越界, 则说明遍历了整个charsH串, 都没办法匹配出charsN来, 则返回-1
        return ni == charsN.length? hi - ni : -1;
    }

    // 类似于小kmp算法
    private int[] getNextArr(char[] chars) {
        if(chars.length == 1) {
            return new int[] { -1 };
        }

        // 人为规定
        int[] nextArr = new int[chars.length];
        nextArr[0] = -1;
        nextArr[1] = 0;

        // i表示要计算的nextArr下标
        int i = 2;

        // cn既表示之前匹配过程中, 已出现的最长相同前缀后缀的长度是多少, 
        // cn也表示chars[i-1]要和chars[cn]相比较的地方
        int cn = 0;

        // O(2 * m)
        while(i < chars.length) {
            // 如果与cn的字符比较相同的话, 则next[2]=之前的最长相同前缀后缀的长度+1, 相当于跟之前连续匹配的意思
            if(chars[i - 1] == chars[cn]) {
                nextArr[i] = cn + 1;

                // 最长相同前缀后缀的长度+1, 同时也表示下一个要比较的字符+1
                cn++;

                // 继续更新下一个next[i]的信息
                i++;
            } 
            // 如果与cn的字符比较不同, 且之前存在过最长相同前缀后缀, 那么看当时next[cn]的最长相同前缀后缀, 然后定位cn到那里, 根据next[cn]往前跳
            else if(cn > 0) {
                cn = nextArr[cn];
            } 
            // 如果与cn的字符比较不同, 且之前不存在相同前缀后缀, 说明到目前位置都没有出现相同的前缀后缀, 无法根据next[cn]往前跳, 那么更新当前next[i]信息为0
            else {
                nextArr[i] = 0;
                i++;
            }
        }

        return nextArr;
    }
}
```

#### 459. 重复的子字符串

##### 1）暴力枚举 | O（n^2）

- **思路**：核心点在于，不断地看当前匹配串，是否与前一个匹配串一致。
- **结论**：时间，4 ms，97.52%，空间，41.6 mb，76.14%，时间上，由于确定子串花费 O（n），匹配串判断花费 O（n），所以整体时间复杂度为 O（n^2），空间上，由于使用了一张 n 长的 chars 数组，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public boolean repeatedSubstringPattern(String s) {
        char[] chars = s.toCharArray();
        int n = chars.length;
        if(n <= 1) {
            return false;
        }
        
        // 确定[0, i)字符串作为子串, 根据题目背景, 只需要确定以i-1作为尾的子串即可, 以x开头的子串就无需判断了, 因为已经判断过了或者肯定不符合
        int len;
        boolean isMatch;
        for(int i = 1; i < n; i++) {
            len = i - 1 + 1;
            isMatch = true;
            
            // 如果整个长度不是len的整数倍, 则肯定不会完全匹配
            if(n % len != 0) {
                continue;
            }

            // 确定[i, n)字符串作为匹配串
            for(int j = i; j < n; j++) {
                // 看当前匹配串是否与前一个匹配串一致
                if(chars[j] != chars[j - len]) {
                    isMatch = false;
                    break;
                }
            }

            // 如果完全匹配, 则返回true
            if(isMatch) {
                return true;
            }
        }

        return false;
    }
}
```

##### 2）模拟 + 移位 + contains 匹配 | O（2 * n + 4 * n^2）

- 思路：
  1. 原串为：abab => 右移位 1 次：baba => 再右移位 1 次：abab，如果能移回原串，则说明原串能够由其中一个子串重复构成。
  2. 而如果模拟这多次移位 + 匹配最多 len - 1 次，会导致复杂度上升很多，为了避免这种无用的环绕，可以先把字符串复制一倍，形成环字符串，即 `abab + abab = abababab` 。
  3. 其中 `baba` 、`abab` 就包含在 `abababab` 的后面子串中，所以，判断能否由其中一个子串重复构成，则只需要看 `abababab` 中是否存在 `abab` 子串即可。
- 结论：时间，68 ms，34.03%，空间，41.6 mb，85.28%，时间上，由于字符串截取花费 O（2 * n），contains 花费 O（4 * n^2），所以整体时间复杂度为 O（2 * n + 4 * n^2），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public boolean repeatedSubstringPattern(String s) {
        return (s + s).substring(1, s.length() * 2 - 1).contains(s);
    }
}
```

##### 3）模拟 + 移位 + KMP 匹配 | O（8 * n）

- **思路**：基于 2 的思路，但直接使用 contains 时间复杂度为 O（n^2），所以可以使用 KMP 算法进行优化。
- **结论**：
  1. 时间，7 ms，82.42%，空间，42.2 mb，10.09%。
  2. 时间上，时间上，由于字符串截取花费 O（2 * n），KMP 中，基于 needle 匹配串初始化 nextArr 数组的过程，最多花费 O（2 * n），基于 haystack 主串可通过 i 与 i - cn 的增减过程判断出来，加速主串的匹配过程，最多花费 O（2 * 2 * n），可通过 hi 与 hi - ni 的增减过程判断出来 ，所以整体时间复杂度为 O（2 * n + 2 * n + 2 * 2 * n） = O（8 * n）。
  3. 空间上，由于使用了一个 2 * n 长的 nextArr 数组，所以额外空间复杂度为 O（2 * n）。

```java
class Solution {
    public boolean repeatedSubstringPattern(String s) {
        String str = (s + s).substring(1, s.length() * 2 - 1);
        return kmp(str, s) != -1;
    }

    // 找出原串haystack中, 第一次出现needle匹配串的索引
    public int kmp(String haystack, String needle) {
        if(haystack == null || needle == null) {
            return -1;
        }

        int hlen = haystack.length();
        int nlen = needle.length();
        if(nlen == 0) {
            return 0;
        }
        if(hlen == 0 || nlen > hlen) {
            return -1;
        }

        char[] charsH = haystack.toCharArray();
        char[] charsN = needle.toCharArray();

        int hi = 0, ni = 0;
        int[] nextArr = getNextArr(charsN);
        
        // O(2 * n)
        while(hi < charsH.length && ni < charsN.length) {
            // 一路匹配, 都各自+1
            if(charsH[hi] == charsN[ni]) {
                hi++;
                ni++;
            }
            // ni == 0, 表示charsN已经无法减少了, 则选择下一个hi, 与charsN从头开始匹配
            else if(ni == 0) {
                hi++;
            } 
            // 如果charsN还可以减少, 那么就来到上一个匹配的地方, charsH保持i不变
            else {
                ni = nextArr[ni];
            }
        }

        // ni越界, 则说明已经匹配出charsN了, 而如果hi越界, 但ni没越界, 则说明遍历了整个charsH串, 都没办法匹配出charsN来, 则返回-1
        return ni == charsN.length? hi - ni : -1;
    }

    // 类似于小kmp算法
    private int[] getNextArr(char[] chars) {
        if(chars.length == 1) {
            return new int[] { -1 };
        }

        // 人为规定
        int[] nextArr = new int[chars.length];
        nextArr[0] = -1;
        nextArr[1] = 0;

        // i表示要计算的nextArr下标
        int i = 2;

        // cn既表示之前匹配过程中, 已出现的最长相同前缀后缀的长度是多少, 
        // cn也表示chars[i-1]要和chars[cn]相比较的地方
        int cn = 0;

        // O(2 * m)
        while(i < chars.length) {
            // 如果与cn的字符比较相同的话, 则next[2]=之前的最长相同前缀后缀的长度+1, 相当于跟之前连续匹配的意思
            if(chars[i - 1] == chars[cn]) {
                nextArr[i] = cn + 1;

                // 最长相同前缀后缀的长度+1, 同时也表示下一个要比较的字符+1
                cn++;

                // 继续更新下一个next[i]的信息
                i++;
            } 
            // 如果与cn的字符比较不同, 且之前存在过最长相同前缀后缀, 那么看当时next[cn]的最长相同前缀后缀, 然后定位cn到那里, 根据next[cn]往前跳
            else if(cn > 0) {
                cn = nextArr[cn];
            } 
            // 如果与cn的字符比较不同, 且之前不存在相同前缀后缀, 说明到目前位置都没有出现相同的前缀后缀, 无法根据next[cn]往前跳, 那么更新当前next[i]信息为0
            else {
                nextArr[i] = 0;
                i++;
            }
        }

        return nextArr;
    }
}
```

### 3.3. 算法思想 - 马拉车

#### 5. 最长回文子串 | medium

##### 1）暴力求解 | O（n^3）

- **思路**：尝试每一个子串， 并找出最大的回文子串。
- **缺点**：
  1. O（n^3），耗时的地方在于，第一轮判断过的地方，第二轮又重新判断了，而且用缓存法还超时了！
  2. 用了个对象来状态变量，可优化成直接使用变量。
- **结论**：放弃该方法，面试会挂（1211ms，0%）！

```java
class Solution {

    class MaxInfo {
        int start;
        int len;

        MaxInfo(int start, int len) {
            this.start = start;
            this.len = len;
        }
    }

    public String longestPalindrome(String s) {
        if(s == null) {
            return null;
        }
        
        MaxInfo maxInfo = new MaxInfo(0, 1);
        char[] chars = s.toCharArray();
        for(int i = 0; i < chars.length; i++) {
            for(int j = 1; j <= chars.length - i; j++) {
                if(isPalindrome(chars, i, i + j - 1) && j > maxInfo.len) {
                    maxInfo.start = i;
                    maxInfo.len = j;
                }
            }
        }

        return s.substring(maxInfo.start, maxInfo.start + maxInfo.len);
    }

    private boolean isPalindrome(char[] chars, int start, int end) {
        if(start == end) {
            return true;
        }
        while(start < end) {
            if(chars[start++] != chars[end--]) {
                return false;
            }
        }
        return true;
    }
}
```

##### 2）马拉车算法 Manacher | O（n）

- **概念**：

  1. **回文半径 r**：指 i 从自身开始算作 1，向外每成功回文一次，则 r + 1。
  2. **回文直径 d**：指 i 向外回文范围内，囊括的字符数。
  3. **回文半径数组 arr**：指从左到右遍历过程中，搜集到的回文半径数组。
  4. **最大回文右边界 R**：指从左到右遍历过程中，回文边界出现得最远的索引位置。
  5. **最大回文右边界中心点 c**：指 R 更新过程中，对应的中心点出现的索引位置。

- **结论**：

  1. **右 r（i） > R**：R 和  c 暴力往右扩。

  2. **右 r（i）<= R**：此时，在 d（c）范围内，出现了 i 的对称点 i' 以及 c 的左边界 L。

     1. **L < 左 r（i'） < c < 右 r（i‘） < R**：说明 i 与 i' 字符数组成逆序关系，r（i） == r（i'），此时不用遍历，可以直接计算得到。

        - **证明**：X != Y，Y == Z，X == L => Z != L。

          ![1641481121818](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1641481121818.png)

     2. **左 r（i’）< L <  c**：r（i）= R - i，此时不用遍历，可以直接计算得到。

        - **证明**：X == Y，Y == Z，X != P => Z != P。

          ![1641481747755](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1641481747755.png)

     3. **左 r（i'）== L < c**：r（i）部分在 R 内，这段不用遍历，可以直接计算得到，但实际有多远不知道，因此，需要 r（i） 初始值等于 R - i，从 R 位置开始暴力扩。

        - **证明**：就像是 （2）情况 i' 的镜像。

```java
class Solution {
    public String longestPalindrome(String s) {
        if(s == null) {
            return null;
        }
        if("".equals(s) ||  s.length() == 1) {
            return s;
        }

        char[] chars = getManacherChars(s);
        int[] rArr = new int[chars.length];// 回文半径数组
        int R = -1, c = -1;// 最大回文右边界、最大回文右边界中心点
        int maxIndex = -1, max = Integer.MIN_VALUE;// 最大回文半径, 最大回文半径索引
        
        for(int i = 0; i < chars.length; i++) {
            // i' = 2 * c - i, R - i 为 i 到最大回文右边界的距离
            // i < R, 代表 i 在 c 的回文半径内, 为了囊口 r(i) > L、r(i) < L 两种情况, 使用了Math.min结合在了一起, 否则则初始化 rArr[i]
            rArr[i] = i < R? Math.min(rArr[2 * c - i], R - i) : 1;

            // r(i) > R 则暴力扩、r(i) = L 则初始值等于 R - i，从 R 位置开始暴力扩
            while(i + rArr[i] < chars.length && i - rArr[i] > -1) {
                if(chars[i + rArr[i]] == chars[i - rArr[i]]) {
                    rArr[i]++;
                } else {
                    break;
                }
            }

            // 暴力扩出 R 范围，则同时更新 R 和 c 的值
            if(i + rArr[i] > R) {
                R = i + rArr[i];
                c = i;
            }

            // 获取最大回文右边界、最大回文右边界对应的索引
            if(rArr[i] > max) {
                maxIndex = i;
                max = rArr[i];
            }
        }

        return getSubString(chars, maxIndex, max);
    }

    private String getSubString(char[] chars, int maxIndex, int max) {
        StringBuilder sb = new StringBuilder();

        // 偶数位为'#'
        for(int i = maxIndex - (max - 1); i < maxIndex + (max - 1); i++) {
            if((i & 1) == 1) {
                sb.append(chars[i]);
            }
        }

        return sb.toString();
    }

    // 获取马拉车字符数组
    private char[] getManacherChars(String s) {
        char[] chars = s.toCharArray();
        char[] res = new char[chars.length * 2 + 1];
        
        // 偶数位添加'#'
        int index = 0;
        for(int i = 0; i < res.length; i++) {
            res[i] = (i & 1) == 0? '#' : chars[index++];
        }

        return res;
    }
}
```

##### 3）中心点向外扩散 | O（n^2）

- **思路**：启发于马拉车算法，但好处在于实现简单，利用中心向外扩散 + 奇数位置为正常字符的原理进行实现。
- **缺点**：由于中心向外扩时，没研究 i 与 i' 的回文半径关系，导致每个字符都需要重新从中心向外扩散一次，时间复杂度（21ms，87%）比马拉车（7ms，96%）高了不少。
- **结论**：面试写的步骤可以循序渐进，从暴力求解 -> 中心点向外扩散 -> 马拉车优化。

```java
class Solution {
    
    public String longestPalindrome(String s) {
        if(s == null) {
            return null;
        }
        if("".equals(s) ||  s.length() == 1) {
            return s;
        }

        int maxIndex = -1, max = Integer.MIN_VALUE;
        char[] chars = getManacherChars(s);
        for(int i = 0; i < chars.length; i++) {
            for(int len = 1; i - len > -1 && i + len < chars.length; len++) {
                if(chars[i-len] != chars[i+len]) {
                    break;
                } else if(len + 1 > max) {
                    max = len + 1;
                    maxIndex = i;
                }
            }
        }

        return getSubString(chars, maxIndex, max);
    }
}
```

#### 9. 回文数 | easy

##### 1）双向指针法 | O（n）

- **思路**：整数 x 转换为 chars 数组，一前一后遍历数组，碰到不同的字符则返回 false，否则返回 true。
- **结论**：
  1. 时间，5 ms，70.84%，空间，41.3 mb，6.79%。
  2. 时间上，由于遍历一次 chars 数组，所以时间复杂度为 O（n），n 为 x 十进制的位数。
  3. 空间上，由于使用了一个 n 长的 chars 数组，所以额外空间复杂度为 O（n）。
  4. 注意，题目要求不能用字符串，所以这样做是错的！

```java
class Solution {
    public boolean isPalindrome(int x) {
        String s = String.valueOf(x);
        char[] chars = s.toCharArray();
        
        int i = 0, j = chars.length - 1;
        while(i < j) {
            if(chars[i++] != chars[j--]) {
                return false;
            }
        }

        return true;
    }
}
```

##### 2）整数反转法 | O（n）

- **思路**：借鉴 《模拟 - 7. 整数反转》的思路，对 x 进行反转，然后判断是否与没反转的相等即可，注意，x 要保留一份原来的副本，以作判断。
- **结论**：时间，5 ms，70.48%，空间，40.9 mb，37.02%，时间上，由于遍历一次 x 十进制的长度，所以时间复杂度为 O（n），n 为 x 十进制的位数，空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public boolean isPalindrome(int x) {
        if(x < 0) {
            return false;
        }

        int y = 0, z = x, adigit;
        while(z != 0) {
            adigit = z % 10;
            z /= 10;
            if(y < Integer.MIN_VALUE / 10 || y > Integer.MAX_VALUE / 10) {
                return false;
            }
            y = y * 10 + adigit;
        }

        return y == x;
    }
}
```

##### 3）反转一半法 | O（n / 2）

- **思路**：
  1. 经过分析，发现 2 中的反转，并不需要完全反转，只反转一半即可，即如果碰到 y 比 x 还大了，那么说明已经反转过一半的十进制位数了。
  2. 虽然这样还可以避免了 int 类型的精度溢出问题，但需要判断两种情况，一种是偶数个，比如 1221，这时，只需要判断 y 和 x 是否相等即可，还有就是奇数个，比如 121，由于 y 比 x 大，即 y = 12 > x = 1，此时需要判断的是 y / 10 是否和 x 相等，因为这样可以移除掉最后一位的 2。
- **结论**：时间，4 ms，100%，空间，40.5 mb，79.66%，时间上，由于只需要遍历一半的十进制位数，所以时间复杂度为 O（n / 2），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public boolean isPalindrome(int x) {
        if(x < 0 || (x != 0 && x % 10 == 0)) {
            return false;
        }

        int y = 0;
        while(y < x) {
            y = y * 10 + (x % 10);
            x /= 10;
        }

        // 偶数个：1221, 奇数个：121
        return y == x || (y / 10) == x;
    }
}
```

#### 125. 验证回文串 | easy

##### 1）双指针 + 暴力转换 | O（n）

- **思路**：通过 `Character.isDigit()` 和 `Character.isLetter()` 方法，来判断字符是否符合情况，通过统一把字母转换为大写字母，来不区分大小写地，比较两个字符是否相同。
- **结论**：时间，5 ms，28.25%，空间，42.9 mb，5.00%，时间上，由于只需要遍历一次字符串，所以时间复杂度为 O（n），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public boolean isPalindrome(String s) {
        if(s == null || s.length() == 0) {
            return true;
        }

        char[] chars = s.toCharArray();
        
        int l = 0, r = chars.length - 1;
        while(l < r) {
            if(!Character.isDigit(chars[l]) && !Character.isLetter(chars[l])) {
                l++;
                continue;
            }
            if(!Character.isDigit(chars[r]) && !Character.isLetter(chars[r])) {
                r--;
                continue;
            }
            if(!String.valueOf(chars[l++]).toUpperCase().equals(
                    String.valueOf(chars[r--]).toUpperCase()
                )) {
                return false;
            }
        }

        return true;
    }
}
```

##### 2）双指针 + ASCII 码判同 | O（n）

- **思路**：思路都是一样的，不过是通过了 ASCII 码来判断是否为数字、小写、大写字母，然后通过 `chars[i] - 'A' + 'a'` 来统一转换为小写字母，再通过小写字母的 ASCII 码是否相等（A' = 65，'Z' = 65 - 1 + 26 = 90，'a' = 97，'z' = 97 - 1 + 26 = 122，'0' = 48，'9' = 48 + 9 = 57），来判断字符是否相同。
- **结论**：时间，1 ms，100%，空间，41.4 mb，56.09%，时间上，时间上，由于只需要遍历一次字符串，所以时间复杂度为 O（n），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public boolean isPalindrome(String s) {
        if(s == null || s.length() == 0) {
            return true;
        }

        char[] chars = s.toCharArray();
        
        int l = 0, r = chars.length - 1, lint, rint;
        while(l < r) {
            lint = chars[l];
            rint = chars[r];

            // 小写、数字则不变, 大写则转换为小写, 其他则跳过
            if(!isValid(lint)) {
                if(!isUpperCase(lint)) {
                    l++;
                    continue;
                } else {
                    lint = (lint - 'A') + 'a';
                }
            }

            // 小写、数字则不变, 大写则转换为小写, 其他则跳过
            if(!isValid(rint)) {
                if(!isUpperCase(rint)) {
                    r--;
                    continue;
                } else {
                    rint = (rint - 'A') + 'a';
                }
            }

            // 判断是否回文
            if(lint != rint) {
                return false;
            }

            l++;
            r--;
        }

        return true;
    }

    private boolean isValid(int charint) {
        if((charint >= '0' && charint <= '9') || (charint >= 'a' && charint <= 'z')) {
            return true;
        }
        return false;
    }

    private boolean isUpperCase(int charint) {
        if(charint >= 'A' && charint <= 'Z') {
            return true;
        }
        return false;
    }
}
```

#### 647. 回文子串 | medium

##### 1）暴力解法 | O（n^3）

- **思路**：枚举每个子串，然后判断是否为回文串，是的话则 count++，统计好后在最后返回即可。
- **结论**：时间，95ms，9.10%，空间，39.2mb，12.73%，时间上，枚举子串需要 O（n^2），判断回文需要 O（n），所以总的时间复杂度为 O（n^3），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int countSubstrings(String s) {
        if(s == null || s.length() == 0) {
            return 0;
        }

        int count = 0;
        char[] chars = s.toCharArray();
        for(int i = 0; i < chars.length; i++) {
            for(int j = i; j < chars.length; j++) {
                if(isPalindrome(chars, i, j)) {
                    count++;
                }
            }
        }

        return count;
    }

    private boolean isPalindrome(char[] chars, int start, int end) {
        if(start == end) {
            return true;
        }
        while(start < end) {
            if(chars[start++] != chars[end--]) {
                return false;
            }
        }
        return true;
    }
}
```

##### 2）暴力递归 | O（n^3）

- **思路**：与暴力解法思路一样，都是枚举所有子串，然后判断子串是否为回文子串，但不同的是，这里判断回文子串使用的递归的方式判断，也就是还存在优化的可能性。
- **结论**：时间，340ms，6.19%，空间，39.4mb，11.62%，时间上，枚举所有子串需要花费 O（n^2），判断子串是否为回文需要 O（n），所以时间复杂度为 O（n^3），空间上，由于判断子串是否为回文最大递归深度为 logn，所以额外空间复杂度为 O（logn）。

```java
class Solution {
    public int countSubstrings(String s) {
        if(s == null || s.length() == 0) {
            return 0;
        }

        int count = 0;
        char[] chars = s.toCharArray();
        for(int i = 0; i < chars.length; i++) {
            for(int j = i; j < chars.length; j++) {
                if(f(chars, i, j)) {
                    count++;
                }
            }
        }

        return count;
    }

    private boolean f(char[] chars, int i, int j) {
        if(i >= j) {
            return true;
        }
        return chars[i] == chars[j] && f(chars, i+1, j-1);
    }
}
```

##### 3）记忆化搜索 | O（n^2）

- **思路**：在暴力递归的基础上，增加了 dp 一维表缓存，如果缓存中存在，则从缓存中获取，否则先把结果设置到缓存中再返回。
- **结论**：时间，18ms，10.75%，空间，46.9mb，5.01%，时间上，由于增加了缓存，使得每次二维表内的元素只会被处理一次，所以时间复杂度为 O（n^2），空间上，由于判断子串是否为回文最大递归深度为 logn，且使用了一张 dp 二维表，所以额外空间复杂度为 O（logn + n^2）。

```java
class Solution {

    private Boolean[][] dp;

    public int countSubstrings(String s) {
        if(s == null || s.length() == 0) {
            return 0;
        }

        char[] chars = s.toCharArray();
        dp = new Boolean[chars.length][chars.length];

        int count = 0;
        for(int i = 0; i < chars.length; i++) {
            for(int j = i; j < chars.length; j++) {
                if(f(chars, i, j)) {
                    count++;
                }
            }
        }

        return count;
    }

    private boolean f(char[] chars, int i, int j) {
        if(i >= j) {
            dp[i][j] = true;
            return dp[i][j];
        }
        if(dp[i][j] != null) {
            return dp[i][j];
        }

        dp[i][j] = chars[i] == chars[j] && f(chars, i+1, j-1);
        return dp[i][j];
    }
}
```

##### 4）严格表结构优化 | O（n^2 + n^2）

- **思路**：在记忆化搜索的基础上，经过研究值依赖关系，发现一个值依赖它的左下一个值，所以这是一个从下往上、从左往右的 dp 模型，因此，需要先初始化好左下半部分，然后再设置右上半部分。
- **结论**：时间，6ms，51.77%，空间，41mb，8.96%，时间上，由于初始化 dp 花费了 O（n^2），枚举并判断每个子串是否为回文串花费了 O（n^2），所以时间复杂度为 O（n^2 + n^2），空间上，由于使用了一张 dp 二维表，所以额外空间复杂度为 O（n^2）。

```java
class Solution {
    public int countSubstrings(String s) {
        if(s == null || s.length() == 0) {
            return 0;
        }

        char[] chars = s.toCharArray();
        boolean[][] dp = new boolean[chars.length][chars.length];

        // 初始化左下半部分
        for(int i = 0; i < dp.length; i++) {
            for(int j = 0; j <= i; j++) {
                dp[i][j] = true;
            }
        }

        // 从下往上、从左往右初始化右上半部分
        for(int i = dp.length-2; i > -1; i--) {
            for(int j = i + 1; j < dp[0].length; j++) {
                dp[i][j] = chars[i] == chars[j] && dp[i+1][j-1];
            }
        }

        int count = 0;
        for(int i = 0; i < chars.length; i++) {
            for(int j = i; j < chars.length; j++) {
                if(dp[i][j]) {
                    count++;
                }
            }
        }

        return count;
    }
}
```

##### 5）中心点向外扩散 | O（n^2）

- **思路**：同暴力解法的思路，都是通过枚举所有子串，然后判断是否为回文串，不过这里的思路是根据中心点的位置来枚举，分为奇数时的中心点位置（索引本身位置）和偶数时的中心点位置（索引本身位置 + 索引前一个位置），然后判断回文串也是使用中心点扩散法来判断~
- **结论**：时间，1ms，99.88%，空间，39.3mb，12.87%，时间上，由于需要枚举两次所有中心点位置的子串 O（2 * n），每次判断子串是否回文需要 O（n），所以时间复杂度为 O（2 * n^2），空间上，由于只使用有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int countSubstrings(String s) {
        if(s == null || s.length() == 0) {
            return 0;
        }

        int n = s.length();
        char[] chars = s.toCharArray();

        // 枚举所有奇数中心点
        int count = 0, wl, wr;
        for(int i = 0; i < n; i++) {
            wl = wr = i;
            while(wl > -1 && wr < n && chars[wl] == chars[wr]) {
                count++;
                wl--;
                wr++;
            }
        }

        // 枚举所有偶数中心点
        for(int i = 1; i < n; i++) {
            wl = i - 1;
            wr = i;
            while(wl > -1 && wr < n && chars[wl] == chars[wr]) {
                count++;
                wl--;
                wr++;
            }
        }

        return count;
    }
}
```

##### 6）马拉车算法 Manacher | O（n）

- **思路**：马拉车算法可以在 O（n）时间内求得马拉车字符数组中，每个位置 i 的回文半径 r，而 r 的一半就正是原本字符数组中《中心向外扩散》的最远距离，也就是枚举每个 i 为中心向外扩散时的 count，因此，累加每个 r/2 则得到 最终答案，其中马拉车算法的证明见《最长回文子串 - 马拉车算法 Manacher》。
- **结论**：时间，1ms，99.88%，空间，39.6mb，10.72%，时间上，由于回文半径只会增加不会减少，也就是生成回文半径时，每个字符只会被访问一次，要么用于计算出来，要么用于暴力扩，而马拉车字符数组中增加的 '#' 个数为 n+1 个，所以遍历马拉车字符数组的时间复杂度为 O（2n +1），空间上，由于构建了马拉车字符数组 O（2n + 1），以及回文半径数组 O（2n +1），所以额外空间复杂度为 O（4n +2）。

```java
class Solution {
    public int countSubstrings(String s) {
        if(s == null || s.length() == 0) {
            return 0;
        }

        char[] chars = getManacherChars(s);
        int[] rArr = new int[chars.length];// 回文半径数组
        int R = -1, c = -1;// 最大回文右边界、最大回文右边界中心点
        int count = 0;
        
        for(int i = 0; i < chars.length; i++) {
            // i' = 2 * c - i, R - i 为 i 到最大回文右边界的距离
            // i < R, 代表 i 在 c 的回文半径内, 为了囊口 r(i) > L、r(i) < L 两种情况, 使用了Math.min结合在了一起, 否则则初始化 rArr[i]
            rArr[i] = i < R? Math.min(rArr[2 * c - i], R - i) : 1;

            // r(i) > R 则暴力扩、r(i) = L 则初始值等于 R - i，从 R 位置开始暴力扩
            while(i + rArr[i] < chars.length && i - rArr[i] > -1) {
                if(chars[i + rArr[i]] == chars[i - rArr[i]]) {
                    rArr[i]++;
                } else {
                    break;
                }
            }

            // 暴力扩出 R 范围，则同时更新 R 和 c 的值
            if(i + rArr[i] > R) {
                R = i + rArr[i];
                c = i;
            }

            // 除2是为了得到真实的回文个数: 因为有一半的'#'不是原来的子串
            count += rArr[i] / 2;
        }

        return count;
    }

    // 获取马拉车字符数组
    private char[] getManacherChars(String s) {
        char[] chars = s.toCharArray();
        char[] res = new char[chars.length * 2 + 1];
        
        // 偶数位添加'#'
        int index = 0;
        for(int i = 0; i < res.length; i++) {
            res[i] = (i & 1) == 0? '#' : chars[index++];
        }

        return res;
    }
}
```

### 3.4. 算法思想 - 斐波那契数列

#### 70. 爬楼梯 | 递推 dp | easy

##### 1）暴力递归 | Master 公式

- **思路**：如果每步只可以爬 1 阶 或者 2 阶，要爬上第 n 阶楼梯，则可以认为从 n-1 阶爬 1 阶爬上来的，或者从 n-2 阶爬 2 阶爬上来的，所以总和为 f（n-1）+ f（n-2）。
- **结论**：执行超时，是由于 f（n-1）和 f（n-2）有很多重复的递归操作，这可以用记忆化搜索来优化。

```java
class Solution {
    public int climbStairs(int n) {
        if(n == 0 || n == 1) {
            return 1;
        }
        if(n == 2) {
            return 2;
        }
        return climbStairs(n-1) + climbStairs(n-2);
    }
}
```

##### 2）记忆化搜索 | O（n）

- **思路**：
  1. 基于暴力递归的基础上，增加了 dp 一维表缓存，如果缓存中存在则从缓存获取，否则返回前先设置结果到缓存再返回。
  2. 并且，发现 n=0,1,2 是不需要缓存的，可以直接返回，此时 dp 缓存也就可以减少前 3 个格子，以节省空间的使用。
- **结论**：时间，0ms，100%，34.9mb，92.24%，效率非常高，应该是最优解了。

```java
class Solution {
    public int climbStairs(int n) {
        if(n == 0 || n == 1) {
            return 1;
        }
        if(n == 2) {
            return 2;
        }
        
        int[] dp = new int[n-2];
        Arrays.fill(dp, -1);
        return f(dp, n);
    }

    private int f(int[] dp, int n) {
        if(n < 0) {
            return 0;
        }
        if(n == 0 || n == 1) {
            return 1;
        }
        if(n == 2) {
            return 2;
        }
        if(dp[n-3] != -1) {
            return dp[n-3];
        }

        dp[n-3] = f(dp, n-1) + f(dp, n-2);
        return dp[n-3];
    }
}
```

#### 2266. 第 292 场周赛 - 统计打字方案数 | medium

##### 1）暴力解法 | O（4 * n）

- **思路**：
  1. 首先，可以把相邻的、相同的字母，进行分组。
  2. 然后，对于每组相同的字母来说，就类似于 70. 爬楼梯 的斐波那契数列：
     - 1）对于 3 种字母的数字来说，组合数 f（n）= 从 f（n - 1）跳上来 + 从 f（n - 2）跳上来 + 从 f（n - 3）跳上来。
     - 2）对于 4 种字母的数字来说，组合数 f（n）= 从 f（n - 1）跳上来 + 从 f（n - 2）跳上来 + 从 f（n - 3）跳上来 + 从 f（n - 4）跳上来。
  3. 最后，总组合数 = 组数 * 每组的组合数。
  4. 其中，在计算每组组合数时，要注意溢出，在计算总组合数时，也要注意溢出，防止溢出的方法有：
     - 1）相加取模公式：`(a + b) % p = (a % p + b % p) % p`。
     - 2）相乘取模公式：`(a * b) % p = (a % p * b % p) % p`。
     - 3）改用 long 类型存储，防止即使取模后相加、或者相乘还是溢出的问题。
- **结论**：
  1. 时间，24 ms，58.07%，空间，62.9 mb，5.03%。
  2. 时间上，最坏情况下，所有的数字都是一样的，且是四种字母的数字（7 或者 9），这时等于 f4（dp4，n），就完全相当于爬楼梯问题了，由于加入了 dp4 的记忆化数组，所以时间复杂度为 O（4 * n）。
  3. 空间上，由于使用了一个 dp3 和一个 dp4 数组，额外空间复杂度为 O（m + k）， m 为 dp3 数组的长度，k 为 dp4 数组的长度。

```java
class Solution {

    private static final int MOD = (int) 1e9+7;

    public int countTexts(String pressedKeys) {
        if(pressedKeys == null || pressedKeys.length() == 0) {
            return 0;
        }
        
        List<Long> dp3 = initDp3();
        List<Long> dp4 = initDp4();

        long res = 1;
        int n;
        char[] chars = pressedKeys.toCharArray();
        for(int i = 0; i < chars.length; i++) {
            n = 1;
            while(i + 1 < chars.length && chars[i] == chars[i + 1]) {
                i++;
                n++;
            }

            // (a * b) % p = (a % p * b % p) % p
            res = chars[i] != '7' && chars[i] != '9'? 
                ( (res % MOD) * (f3(dp3, n) % MOD) ) % MOD
                :
                ( (res % MOD) * (f4(dp4, n) % MOD) ) % MOD;
        }

        return (int) res;
    }

    // 3个字母的dp
    private long f3(List<Long> dp, int n) {
        if(n <= dp.size() - 1) {
            return dp.get(n);
        }

        // (a + b) % p = (a % p + b % p) % p
        long res = (
            f3(dp, n - 1) % MOD
            + f3(dp, n - 2) % MOD
            + f3(dp, n - 3) % MOD
        ) % MOD;

        dp.add(res);
        return res;
    }

    // 4个字母的dp
    private long f4(List<Long> dp, int n) {
        if(n <= dp.size() - 1) {
            return dp.get(n);
        }

        // (a + b) % p = (a % p + b % p) % p
        long res = (
            f4(dp, n - 1) % MOD
            + f4(dp, n - 2) % MOD
            + f4(dp, n - 3) % MOD
            + f4(dp, n - 4) % MOD
        ) % MOD;

        dp.add(res);
        return res;
    }

    // 初始化3个字母的dp
    private List<Long> initDp3() {
        List<Long> dp = new ArrayList<>();
        dp.add(-1L);
        dp.add(1L);
        dp.add(2L);
        dp.add(4L);
        return dp;
    }

    // 初始化4个字母的dp
    private List<Long> initDp4() {
        List<Long> dp = new ArrayList<>();
        dp.add(-1L);
        dp.add(1L);
        dp.add(2L);
        dp.add(4L);
        dp.add(8L);
        return dp;
    }
}
```

##### 2）暴力递归 | O（n^4）

- **思路**：
  1. 构建一个 f（end）函数，代表以 end 结尾的数字，所拥有的组合数。
  2. 首先，默认认为与前面的数字不同，相当于只有一个场景，即（当前数字）构成一个字母，则组合数 = 前面的组合数。
  3. 然后，如果与前面的数字相同，相当于多了一个场景，即（当前数字，前面的数字）构成一个字母，则组合数 = 前面的组合数 + （当前数字，前面的数字）的组合数。
  4. 接着，如果与前前面数字相同，相当于多了一个场景，即（当前数字，前面的数字，前前面的数字）构成一个字母，则组合数 = 前面的组合数 + （当前数字，前面的数字）的组合数 + （当前数字，前面的数字，前前面的数字）的组合数。
  5. 最后，当当前数字为 7 或者 9，且与前前前面的数字相同，相当于多了一个场景，即（当前数字，前面的数字，前前面的数字，前前前面的数字）构成一个字母，则组合数 = 前面的组合数 + （当前数字，前面的数字）的组合数 + （当前数字，前面的数字，前前面的数字）+（当前数字，前面的数字，前前面的数字，前前前面的数字）的组合数。
  6. 其中，在累加时时，要注意溢出，防止溢出的方法有：
     - 1）相加取模公式：`(a + b) % p = (a % p + b % p) % p`。
     - 2）改用 long 类型存储，防止即使取模后相加还是溢出的问题。
- **结论**：执行超时，时间上，由于最坏情况下，可能会调用 4 次 f（end）函数，即 4 次 f（end）函数的组合，所以时间复杂度为 O（n^4），空间上，由于递归深度最大为 4 次 f（end）函数的组合，所以额外空间复杂度为 O（n^4）。

```java
class Solution {

    private static final int MOD = (int) 1e9+7;

    public int countTexts(String pressedKeys) {
        if(pressedKeys == null || pressedKeys.length() == 0) {
            return 0;
        }
        
        char[] chars = pressedKeys.toCharArray();
        return (int) f(chars, chars.length - 1);
    }

    // 以end结尾的数字, 所拥有的组合数
    private long f(char[] chars, int end) {
        // 小于0, 则认为当前别人调的也算一种方案
        if(end <= 0) {
            return 1;
        }
        if(end == 1) {
            return chars[1] == chars[0]? 2 : 1;
        }
        if(end == 2) {
            long res = f(chars, 1);
            if(chars[2] == chars[1]) {
                res += f(chars, 0);
                if(chars[2] == chars[0]) {
                    res += f(chars, -1);
                }
            }
            return res;
        }

        // 1、默认认为与前面的数字不同, 则组合数 = 前面的组合数
        long res = f(chars, end - 1) % MOD;

        // 2、与前面的数字相同, 则组合数 = 前面的组合数 + (当前数字, 前面的数字)的组合数
        if(chars[end] == chars[end - 1]) {
            res += f(chars, end - 2) % MOD;

            // 3、与前前面数字相同, 则组合数 = 前面的组合数 + (当前数字, 前面的数字)的组合数 + (当前数字, 前面的数字, 前前面的数字)的组合数
            if(chars[end] == chars[end - 2]) {
                res += f(chars, end - 3) % MOD;

                // 4、当为7或者9时, 且与前前前面的数字相同, 则组合数 = 前面的组合数 + (当前数字, 前面的数字)的组合数 + (当前数字, 前面的数字, 前前面的数字)的组合数 + (当前数字, 前面的数字, 前前面的数字, 前前前面的数字)的组合数
                if((chars[end] == '7' || chars[end] == '9') && chars[end] == chars[end - 3]) {
                    res += f(chars, end - 4) % MOD;
                }
            }
        }

        return res % MOD;
    }
}
```

##### 3）记忆化搜索 | O（n）

- **思路**：在暴力递归的基础上，增加 dp 一维数组缓存，如果缓存中存在，则返回缓存中的数据，否则分情况讨论并设置结果到缓存后，才返回给上一层。
- **结论**：时间，48 ms，29.42%，空间，57.9 mb，5.74%，时间上，由于加入了缓存，使得 4 次的 f（end） 组合，只需要执行一次即可，所以时间复杂度为 O（n），同理，空间上，递归深度最大也只有 O（n），所以额外空间复杂度为 O（n）。

```java
class Solution {

    private static final int MOD = (int) 1e9+7;

    public int countTexts(String pressedKeys) {
        if(pressedKeys == null || pressedKeys.length() == 0) {
            return 0;
        }
        
        char[] chars = pressedKeys.toCharArray();
        long[] dp = new long[chars.length];
        Arrays.fill(dp, -1);

        return (int) f(chars, dp, chars.length - 1);
    }

    // 以end结尾的数字, 所拥有的组合数
    private long f(char[] chars, long[] dp, int end) {
        // 小于0, 则认为当前别人调的也算一种方案
        if(end <= 0) {
            return 1;
        }
        if(dp[end] != -1) {
            return dp[end];
        }
        if(end == 1) {
            dp[end] = chars[1] == chars[0]? 2 : 1;
            return dp[end];
        }
        if(end == 2) {
            long res = f(chars, dp, 1);
            if(chars[2] == chars[1]) {
                res += f(chars, dp, 0);
                if(chars[2] == chars[0]) {
                    res += f(chars, dp, -1);
                }
            }

            dp[end] = res;
            return dp[end];
        }

        // 1、默认认为与前面的数字不同, 则组合数 = 前面的组合数
        long res = f(chars, dp, end - 1) % MOD;

        // 2、与前面的数字相同, 则组合数 = 前面的组合数 + (当前数字, 前面的数字)的组合数
        if(chars[end] == chars[end - 1]) {
            res += f(chars, dp, end - 2) % MOD;

            // 3、与前前面数字相同, 则组合数 = 前面的组合数 + (当前数字, 前面的数字)的组合数 + (当前数字, 前面的数字, 前前面的数字)的组合数
            if(chars[end] == chars[end - 2]) {
                res += f(chars, dp, end - 3) % MOD;

                // 4、当为7或者9时, 且与前前前面的数字相同, 则组合数 = 前面的组合数 + (当前数字, 前面的数字)的组合数 + (当前数字, 前面的数字, 前前面的数字)的组合数 + (当前数字, 前面的数字, 前前面的数字, 前前前面的数字)的组合数
                if((chars[end] == '7' || chars[end] == '9') && chars[end] == chars[end - 3]) {
                    res += f(chars, dp, end - 4) % MOD;
                }
            }
        }

        dp[end] = res % MOD;
        return dp[end];
    }
}
```

##### 4）严格表结构优化 | O（n）

- **思路**：在记忆化搜索的基础上，经过研究表结构的依赖关系，发现是后一个依赖前一个的值，所以是从左到右初始化 dp 一维数组，最后再返回 dp[n - 1] 的值即可。
- **结论**：时间，23 ms，63.81%，空间，41.9 mb，77.57%，时间上，由于只需要遍历一次 chars 数组，所以时间复杂度为 O（n），空间上，由于使用了 dp 一维数组，所以额外空间复杂度为 O（n）。

```java
class Solution {

    private static final int MOD = (int) 1e9+7;

    public int countTexts(String pressedKeys) {
        if(pressedKeys == null || pressedKeys.length() == 0) {
            return 0;
        }
        if(pressedKeys.length() == 1) {
            return 1;
        }
        
        char[] chars = pressedKeys.toCharArray();
        long[] dp = new long[chars.length];

        dp[0] = 1;
        dp[1] = chars[1] == chars[0]? 2 : 1;
        
        long res;
        for(int end = 2; end < chars.length; end++) {
            // 1、默认认为与前面的数字不同, 则组合数 = 前面的组合数
            res = end - 1 > -1? dp[end - 1] % MOD : 1;

            // 2、与前面的数字相同, 则组合数 = 前面的组合数 + (当前数字, 前面的数字)的组合数
            if(chars[end] == chars[end - 1]) {
                res += end - 2 > -1? dp[end - 2] % MOD : 1;

                // 3、与前前面数字相同, 则组合数 = 前面的组合数 + (当前数字, 前面的数字)的组合数 + (当前数字, 前面的数字, 前前面的数字)的组合数
                if(chars[end] == chars[end - 2]) {
                    res += end - 3 > -1? dp[end - 3] % MOD : 1;

                    // 4、当为7或者9时, 且与前前前面的数字相同, 则组合数 = 前面的组合数 + (当前数字, 前面的数字)的组合数 + (当前数字, 前面的数字, 前前面的数字)的组合数 + (当前数字, 前面的数字, 前前面的数字, 前前前面的数字)的组合数
                    if((chars[end] == '7' || chars[end] == '9') && end >= 3 && chars[end] == chars[end - 3]) {
                        res += end - 4 > -1 ? dp[end - 4] % MOD : 1;
                    }
                }
            }

            dp[end] = res % MOD;
        }

        return (int) dp[chars.length - 1];
    }
}
```

#### 2320. 第 299 场周赛 - 统计放置房子的方式数 | 线性 dp | medium

##### 1）斐波纳契数列 + 线性 dp | O（n）

- **思路**：

  1. 通过打表，尝试提交每一个低阶的 n，以获得对应的正确结果，得到以下信息：

     | n    | 1    | 2    | 3    | 4    | 5    |
     | ---- | ---- | ---- | ---- | ---- | ---- |
     | 值   | 2^2  | 3^2  | 5^2  | 8^2  | 13^2 |

  2. 可见，值呈现斐波纳契数列平方的规律显示，因此可以肯定的是，结果是运用斐波那契数列的 dp 过程，最后平方取模即可。

  3. 而如果从状态转移的角度，去思考问题的话，可以得到的结论是，由于街道两旁互不影响，所以可以先单独考虑一侧的结果。

  4. 对于一侧的放置方案，由于房屋不能放在相邻的位置，所以第 i 位置的放置方案有，要么 i 位置不放，此时 dp[i] = dp[i - 1] ，要么 i 位置放，此时 dp[i] = dp[i - 2]，因此，dp[i] = dp[i - 1] + dp[i  - 2]，与爬楼梯和斐波那契数列思路一样。

  5. 最后，由于需要考虑两侧街道的组合数，所以最终结果等于 dp[i]^2，其中要注意转换为 long 类型再取模保证不会 int 溢出。

- **结论**：时间，6 ms，85.28%，空间，40.9 mb，66.20%，时间上，由于只需要遍历一次 n 块地块，所以时间复杂度为 O（n），空间上，由于使用了一张 n + 1 长的 dp 一维表，所以额外空间复杂度为 O（n + 1）。

```java
class Solution {

    public static final int MOD = (int) 1e9 + 7;
    
    public int countHousePlacements(int n) {
        if(n == 1) {
            return 4;
        }
        if(n == 2) {
            return 9;
        }
        
        int[] dp = new int[n + 1];
        dp[1] = 2;
        dp[2] = 3;
        for(int i = 3; i < n + 1; i++) {
            dp[i] = (dp[i - 1] + dp[i - 2]) % MOD;
        }
        
        return (int) (((long) dp[n] * dp[n]) % MOD);
    }
}
```

### 3.5. 算法思想 - 动态规划

#### 10. 正则表达式匹配 | 线性 dp | hard

##### 1）暴力递归 | O（s * p * s）

- **思路**：从左到右尝试模型，比的是 s 目标串和 p匹配串从某位置开始，从左到右是否能够**一直**保持匹配。
- **可能性讨论**：
  1. **p[i + 1] 不为 ***：p[i + 1] 越界、p[i + 1] != '*' 都算。
     1. 先看 p[i] 是否匹配，即是否等于 s[i] 或者 '.'。
     2. 如果匹配，则 s 和 p **一起**从左到右尝试，看剩下 f(s+1,p+1) 是否继续匹配。
     3. 如果不匹配，则返回 false，表示不匹配。
  2. **p[i + 1] 为 ***：p[i + 1] == '*'。
     1. 先看 p[i] 是否匹配，即是否等于 s[i] 或者 '.'。
     2. 如果匹配，则 s **自己**从左到到右尝试，依次匹配0,1,2,... 次 p[i]，这种尝试形态上为 f(s+n, p+2)，n 循环变化的。
     3. 如果不匹配，则看 f(s, p+2) ，代表 p 丢弃了 p[i] 与 * 后，剩余的字符是否还能和 s 继续匹配。
- **缺点**：属于尝试的原型，时间复杂度高（9ms，18%），出现算过的位置又重新递归算的地方。

```java
class Solution {
    public boolean isMatch(String s, String p) {
        if((s == null && p == null) || (s == "" && p == "")) {
            return true;
        }
        if((s == null && p != null) || (s != null && p == null) || p == "") {
            return false;
        }
        
        char[] schars = s.toCharArray();
        char[] pchars = p.toCharArray();

        return isValid(pchars) && f(schars, pchars, 0, 0);
    }

    // 用于保证不会出现 **
    private boolean isValid(char[] pchars) {
        for(int i = 0; i < pchars.length; i++) {
            if(pchars[i] == '*') {
                if(i == 0) {
                    return false;
                }
                if(i + 1 < pchars.length && pchars[i + 1] == '*') {
                    return false;
                }
            }
        }

        return true;
    }

    // si、ei：代表s[si...]一直匹配p[pi...]
    // 一定要保证ei不是*, 即保证无后效性, 即前面的讨论情况留到后面来讨论, 这是不对的
    private boolean f(char[] schars, char[] pchars, int si, int pi) {
        // pi越界, 代表表达式串没了, 这时结果取决于s串是否还有
        if(pi == pchars.length) {
            return si == schars.length;
        }

        // 如果pi没有后续字符, 相当于pi+1不是*, 此时结果取决于si与pi一一匹配
        if(pi + 1 == pchars.length || pchars[pi + 1] != '*') {
            // 如果si提前越界了, 说明字符串不匹配
            if(si == schars.length) {
                return false;
            } 
            // 如果si还没越界, 说明还有得判断, 此时结果看pi是否匹配 + 后面是否匹配得上
            else if(pchars[pi] == '.' || pchars[pi] == schars[si]) {
                return f(schars, pchars, si + 1, pi + 1);// 同时保证了pi不为*
            } else {
                return false;
            }
        } 

        // 如果pi有后续字符, 且为 *, 此时需要做*号的匹配判断
        while (si != schars.length && (pchars[pi] == '.' || pchars[pi] == schars[si])) {
            // 如果一路匹配, 则看后面的是否能够继续匹配
            if(f(schars, pchars, si, pi + 2)) {// 同时保证了pi不为*
                // 如果一路匹配同时后面也匹配, 则说明确实匹配
                return true;
            }
            
            si++;
        }

        // 如果s一路匹配到尾了, 则看p是否还有剩余字符
        // 或者pi可能在0,1,2...任何位置不匹配, 则看丢弃掉n*然后后面是否能够从原si位置匹配s成功
        return f(schars, pchars, si, pi + 2);// 同时保证了pi不为*
    }
}
```

##### 2）记忆化搜索优化 | <= O（s * p * s）

- **思路**：保持着和暴力递归一样的尝试，但通过缓存的方式，减少了一些多余的递归尝试，可以在常数项上降低一点。
- **缺点**：力扣测试用例不多，效果不明显（13ms，16%）。

```java
class Solution {

    private Boolean[][] dp;

    public boolean isMatch(String s, String p) {
        if((s == null && p == null) || (s == "" && p == "")) {
            return true;
        }
        if((s == null && p != null) || (s != null && p == null) || p == "") {
            return false;
        }
        
        char[] schars = s.toCharArray();
        char[] pchars = p.toCharArray();

        // 记忆化搜索优化: si => row, pi => col
        dp = new Boolean[schars.length + 1][pchars.length + 1];
        return isValid(pchars) && f(schars, pchars, 0, 0);
    }

    // 用于保证不会出现 **
    private boolean isValid(char[] pchars) {
        for(int i = 0; i < pchars.length; i++) {
            if(pchars[i] == '*') {
                if(i == 0) {
                    return false;
                }
                if(i + 1 < pchars.length && pchars[i + 1] == '*') {
                    return false;
                }
            }
        }

        return true;
    }

    // si、ei：代表s[si...]一直匹配p[pi...]
    // 一定要保证ei不是*, 即保证无后效性, 即前面的讨论情况留到后面来讨论, 这是不对的
    private boolean f(char[] schars, char[] pchars, int si, int pi) {
        if(dp[si][pi] != null) {
            return dp[si][pi];// 记忆化搜索优化: si => row, pi => col
        }

        // pi越界, 代表表达式串没了, 这时结果取决于s串是否还有
        if(pi == pchars.length) {
            dp[si][pi] = si == schars.length;
            return dp[si][pi];// 记忆化搜索优化: si => row, pi => col
        }

        // 如果pi没有后续字符, 相当于pi+1不是*, 此时结果取决于si与pi一一匹配
        if(pi + 1 == pchars.length || pchars[pi + 1] != '*') {
            // 如果si提前越界了, 说明字符串不匹配
            if(si == schars.length) {
                dp[si][pi] = false;
                return dp[si][pi];// 记忆化搜索优化: si => row, pi => col
            } 
            // 如果si还没越界, 说明还有得判断, 此时结果看pi是否匹配 + 后面是否匹配得上
            else if(pchars[pi] == '.' || pchars[pi] == schars[si]) {
                dp[si][pi] = f(schars, pchars, si + 1, pi + 1);// 同时保证了pi不为*
                return dp[si][pi];// 记忆化搜索优化: si => row, pi => col
            } else {
                dp[si][pi] = false;
                return dp[si][pi];// 记忆化搜索优化: si => row, pi => col
            }
        } 

        // 如果pi有后续字符, 且为 *, 此时需要做*号的匹配判断
        while (si != schars.length && (pchars[pi] == '.' || pchars[pi] == schars[si])) {
            // 如果一路匹配, 则看后面的是否能够继续匹配
            if(f(schars, pchars, si, pi + 2)) {// 同时保证了pi不为*
                // 如果一路匹配同时后面也匹配, 则说明确实匹配
                dp[si][pi] = true;
                return dp[si][pi];// 记忆化搜索优化: si => row, pi => col
            }
            
            si++;
        }

        // 如果s一路匹配到尾了, 则看p是否还有剩余字符
        // 或者pi可能在0,1,2...任何位置不匹配, 则看丢弃掉n*然后后面是否能够从原si位置匹配s成功
        dp[si][pi] = f(schars, pchars, si, pi + 2);// 同时保证了pi不为*
        return dp[si][pi];// 记忆化搜索优化: si => row, pi => col
    }
}
```

##### 3）严格表结构优化 | O ( s * p）

- **思路**：

  1. 纸上绘制 S、P 两个维度（因为递归中只有两个**变量**），整理 base case 值、分析 dp 位置值依赖关系。
  2. 这道题的 base case 值不明显，需要根据值依赖关系提前倒推出来。
  3. 值依赖关系为 x = 右下角的值、右边第二列值的枚举。
  4. 因此，目标要求的值为第一行第一列格子的值，可以通过从右往左、从下往上求出来并返回。

  ![1641576066088](D:\MyData\yaocs2\AppData\Roaming\Typora\typora-user-images\1641576066088.png)

- **优点**：dp 是对记忆化搜索的一种优化手段，通过绘图得出每个 dp 位置的值依赖计算，从而省去了递归的开销。

- **缺点**：在碰到 * 时会做了一层枚举计算依赖值，应该属于最痛的地方（3ms，38%），后期看看能否通过进行**斜率优化**。

```java
class Solution {

    public boolean isMatch(String s, String p) {
        if((s == null && p == null) || (s == "" && p == "")) {
            return true;
        }
        if((s == null && p != null) || (s != null && p == null) || p == "") {
            return false;
        }
        
        char[] schars = s.toCharArray();
        char[] pchars = p.toCharArray();
        if(!isValid(pchars)) {// 注释掉这步，虽然能过（2ms，83.33%），但逻辑上是不对的
            return false;
        }

        int S = schars.length + 1, P = pchars.length + 1;
        boolean[][] dp = new boolean[S][P];

        // base case 值整理
        dp[S - 2][P - 2] = schars[S - 2] == pchars[P - 2] || pchars[P - 2] == '.';
        dp[S - 1][P - 1] = true;

        // 最后一行 base case 必须为 a*a*a*...的格式, 否则为 true
        for(int pi = P - 3; pi > -1; pi -= 2) {
            if(pchars[pi + 1] == '*') {
                dp[S - 1][pi] = true;
            } else {
                break;
            }
        }

        // 动态规划 dp => 从右往左, 从下往上
        for(int si = S - 2; si > -1; si--) {
            for(int pi = P - 3; pi > - 1; pi--) {
                if(pchars[pi + 1] != '*') {
                    if(pchars[pi] == '.' || pchars[pi] == schars[si]) {
                        dp[si][pi] = dp[si + 1][pi + 1];
                    }
                } else {
                    int i = si;
                    while (i != S - 1 && (pchars[pi] == '.' || pchars[pi] == schars[i])) {
                        if(dp[i][pi + 2]) {
                            dp[si][pi] = true;
                            break;
                        }
                        i++;
                    }

                    dp[si][pi] = dp[i][pi + 2];
                }
            }
        }

        return dp[0][0];
    }

    // 用于保证不会出现 **
    private boolean isValid(char[] pchars) {
        for(int i = 0; i < pchars.length; i++) {
            if(pchars[i] == '*') {
                if(i == 0) {
                    return false;
                }
                if(i + 1 < pchars.length && pchars[i + 1] == '*') {
                    return false;
                }
            }
        }

        return true;
    }
}
```

#### 32. 最长有效括号 | 线性 dp | hard

##### 1）二维暴力递归 | O（n^3）

- **思路**：f（start，end）代表从 [start，end] 内最大的有效括号子串长度。
  1. 每次先判断 [start，end] 是否为有效括号串，如果是则直接返回。
  2. 如果整个 [start，end] 不是有效括号串，则尝试看看 [start，end - 1] 和 [start + 1，end] 哪个才是最长的有效括号串，最后把最大值返回即可。
- **缺点**：时间复杂度高，由于 [start，end - 1] 和 [start + 1，end] 的尝试都要花费 O（n^2），因为要看 n-1,n-2,n-3...1 次，且每次看都要花费 O（n），所以一共花费 O（n^3）。
- **结论**：执行超时，面试会挂，考的应该是动态规划！

```java
class Solution {
    public int longestValidParentheses(String s) {
        if(s == null || s.length() < 2) {
            return 0;
        }

        char[] chars = s.toCharArray();
        return f(chars, 0, chars.length - 1);
    }

    // 以start开头、以end结尾最大的括号子串
    private int f(char[] chars, int start, int end) {
        if(end - start < 1) {
            return 0;// base case
        }
        if(isValid(chars, start, end)) {
            return end - start + 1;// base case
        }
        return Math.max(f(chars, start, end - 1), f(chars, start + 1, end));
    }

    // 判断从start开头到end结尾是否为合格的括号串
    private boolean isValid(char[] chars, int start, int end) {
        int count = 0;
        for(int i = start; i <= end; i++) {
            if(chars[i] == '(') {
                count++;
            } else {
                count--;
            }
            if(count < 0) {
                return false;
            }
        }
        return count == 0;
    }
}
```

##### 2）二维记忆化搜索优化 | <= O（n^3）

- **思路**：在暴力递归的基础上，增加 dp 二维数组缓存，每次获取前从缓存中获取，每次返回前先塞到缓存中。
- **结论**：**超过内存限制**，面试会挂，应该考的是动态规划，还要继续优化！

```java
class Solution {

    private int[][] dp;

    public int longestValidParentheses(String s) {
        if(s == null || s.length() < 2) {
            return 0;
        }

        char[] chars = s.toCharArray();
        dp = new int[chars.length][chars.length];
        for(int i = 0; i < chars.length; i++) {
            for(int j = 0; j < chars.length; j++) {
                dp[i][j] = -1;
            }
        }

        return f(chars, 0, chars.length - 1);
    }

    // 以start开头、以end结尾最大的括号子串
    private int f(char[] chars, int start, int end) {
        if(dp[start][end] != -1) {
            return dp[start][end];
        }
        if(end - start < 1) {
            dp[start][end] = 0;
            return dp[start][end];// base case
        }     
        if(isValid(chars, start, end)) {
            dp[start][end] = end - start + 1;
            return dp[start][end];// base case
        }
        
        dp[start][end] = Math.max(f(chars, start, end - 1), f(chars, start + 1, end));
        return dp[start][end];
    }

    // 判断从start开头到end结尾是否为合格的括号串
    private boolean isValid(char[] chars, int start, int end) {
        int count = 0;
        for(int i = start; i <= end; i++) {
            if(chars[i] == '(') {
                count++;
            } else {
                count--;
            }
            if(count < 0) {
                return false;
            }
        }
        return count == 0;
    }
}
```

##### 3）二维严格表结构优化 | <= O（n^3）

- **思路**：
  1. 根据对记忆化搜索缓存表的观察可得知，f(x，y) 依赖于 f(x，y-1) 和 f（x+1，y），其中 x 代表二维表的高度，y 代表二维表的宽度，也就是依赖于左边和下边的值。
  2. 因此，二维表的初始化顺序为：从下往上（x 减小方向），从左往右（y 增大方向）。
- **结论**：**超过内存限制**，面试会挂，应该考的是**动态规划的空间压缩技巧**，还要继续优化！

```java
class Solution {

    public int longestValidParentheses(String s) {
        if(s == null || s.length() < 2) {
            return 0;
        }

        int[][] dp = new int[s.length()][s.length()];

        // 从下往上, 从左往右
        for(int i = s.length() - 1; i > -1; i--) {
            for(int j = 0; j < s.length(); j++) {
                if(j - i < 1) {
                    continue;
                }
                if(isValid(s, i, j)) {
                    dp[i][j] = j - i + 1;
                    continue;
                }
                
                dp[i][j] = Math.max(dp[i][j-1], dp[i+1][j]);
            }
        }

        return dp[0][s.length()-1];
    }

    // 判断从start开头到end结尾是否为合格的括号串
    private boolean isValid(String s, int start, int end) {
        int count = 0;
        for(int i = start; i <= end; i++) {
            if(s.charAt(i) == '(') {
                count++;
            } else {
                count--;
            }
            if(count < 0) {
                return false;
            }
        }
        return count == 0;
    }
}
```

##### 4）二维表空间压缩优化 | <= O（n^3）

- **思路**：
  1. 观察二维表的依赖行为可发现，一个正常的格子只会依赖它的左边和下边，由于是从下往上、从左往右的初始化顺序，所以可以压缩从下往上方向的维度空间。
  2. 压缩方法为，在下次取下格子值的时候，获取当前数组同位置的老值即可，计算出新值后在塞会该位置，然后网上走一层，这样下次再去下格子值时，此时的位置已经算式老值了。
  3. 同时优化 isValid（）函数，在为非偶数，或者左不为 ‘（’，或者右不为 ')' 时，直接返回false，不用在遍历判断了。
- **结论**：
  1. 内存得到保障后，结果又**超出了时间的限制**，面试会挂，还要继续优化！
  2. 摆脱不了 isValid（）函数的话，就永远降不到 O（n）！

```java
class Solution {

    public int longestValidParentheses(String s) {
        if(s == null || s.length() < 2) {
            return 0;
        }

        // 压缩start的空间
        int[] dp = new int[s.length()];

        // 从下往上, 从左往右
        for(int i = s.length() - 1; i > -1; i--) {
            for(int j = 0; j < s.length(); j++) {
                if(j - i < 1) {
                    continue;
                }
    
                if(isValid(s, i, j)) {
                    dp[j] = j - i + 1;
                    continue;
                }
                
                dp[j] = Math.max(dp[j-1], dp[j]);
            }
        }

        return dp[s.length()-1];
    }

    // 判断从start开头到end结尾是否为合格的括号串
    private boolean isValid(String s, int start, int end) {
        if(end - start + 1 % 2 == 1) {
            return false;
        }
        if(s.charAt(start) != '(') {
            return false;
        }
        if(s.charAt(end) != ')') {
            return false;
        }

        int count = 0;
        for(int i = start; i <= end; i++) {
            if(count < 0) {
                return false;
            }
            if(s.charAt(i) == '(') {
                count++;
            } else {
                count--;
            }
        }
        return count == 0;
    }
}
```

##### 5）一维暴力递归 | O（n^2）

- **思路**：f（）函数定义为：以 end 字符结尾的子序列，它的最大括号匹配长度为多少，这样就可以将上述二维的递归降到一维，且摆脱了 isValid（）函数的依赖，时间降到了 O（n）级别。
  1. 如果 end 位置越界，或者剩余的字符根本不够 2 个，说明当前子序列肯定是非法的，返回 0 代表非法子序列。
  2. 如果 end 位置为 '('，说明当前子序列肯定也是非法的，返回 0 代表非法子序列。
  3. 如果 end 位置为 ')'，说明当前子序列有可能是合法的，此时还要看前面的字符是否匹配。
     1. 如果 end-1 字符为 '('，说明 end-1 与 end 匹配，此时最大的匹配长度就等于 f（end-1）+ 2。
     2. 如果 end-1 字符为 ')'，说明 end-1 并不与 end 匹配，此时就还要往前看是否有匹配的字符。
        1. 通过计算 f（end-1）得到 end-1 的最大匹配长度，然后用当前下标 end-1-f（end-1），即可得到远处能够匹配当前 end 的字符索引 l。
        2. 如果 l 合法，且为 '('，说明 l 匹配了 end，也就是 end 同时囊括了 l 和 end-1 子序列的最大匹配长度，此时最大的匹配长度为 2 + f（end-1）+ f（l-1）。
        3. 如果 l 不合法，或者为 ')'，说明 end 肯定不是个合法的子序列，因为即使中间的 l+1~end-1 都匹配了，如果没有 l 或者为 ‘)’，那么 end 处的 ')' 就没有 '(' 匹配了，所以返回 0。
- **结论**：暴力递归重复判断了很多次（18ms，5.01%），可以上记忆化搜索来优化。

```java
class Solution {
    public int longestValidParentheses(String s) {
        if(s == null || s.length() < 2) {
            return 0;
        }

        int max = Integer.MIN_VALUE;
        for(int i = 0; i < s.length(); i++) {
            max = Math.max(max, f(s, i));
        }

        return max;
    }

    // 返回以end字符结尾的子串的最大匹配长度
    private int f(String s, int end) {
        // 1) 如果end位置不够或者越界, 说明本子串为非法的括号串, 返回0
        if(end < 1 || end >= s.length()) {
            return 0;// base case
        }
        // 2) 如果end位置为'(', 说明本子串为非法的括号串, 返回0
        if(s.charAt(end) == '(') {
            return 0;// base case
        }

        // 3) end位置为')'
        // 3.1) end-1位置(必存在)为'(', 则匹配end位置的')', 此时最大匹配长度还要看end-2位置的
        if(s.charAt(end - 1) == '(') {
            if(end - 2 > -1) {
                return 2 + f(s, end - 2);
            } else {
                return 2;
            }
        } 
        
        // 3.2) end-1位置为')', 说明不匹配end位置的')', 此时还要继续往前找左括号
        // 最前的左括号位置可能等于 = 当前位置 - 1 - (end-1)的最大匹配长度
        int prelen = f(s, end - 1);
        int l = end - 1 - prelen;

        // 远处存在'(', 则当前的最大匹配长度为: 2 + prelen + f(s, l-1);
        if(l > -1 && s.charAt(l) == '(') {
            return 2 + prelen + f(s, l - 1);
        } else {
            return 0;
        }
    }
}
```

##### 6）一维记忆化搜索优化 | <= O（n^2）

- **思路**：在一维暴力递归的基础上，增加 dp 一维数组作为缓存，大大减少了不必要的递归。
- **结论**：还是用到了递归（2ms，54.78%），可以分析表依赖，继续优化为动态规划~

```java
class Solution {

    private int[] dp;

    public int longestValidParentheses(String s) {
        if(s == null || s.length() < 2) {
            return 0;
        }

        // 记忆化搜索优化
        dp = new int[s.length()];
        Arrays.fill(dp, -1);

        int max = Integer.MIN_VALUE;
        for(int i = 0; i < s.length(); i++) {
            max = Math.max(max, f(s, i));
        }

        return max;
    }

    // 返回以end字符结尾的子串的最大匹配长度
    private int f(String s, int end) {
        // 1) 如果end位置不够或者越界, 说明本子串为非法的括号串, 返回0
        if(end < 1 || end >= s.length()) {   
            return 0;// 越界校验
        }
        if(dp[end] != -1) {
            return dp[end];
        }
        
        // 2) 如果end位置为'(', 说明本子串为非法的括号串, 返回0
        if(s.charAt(end) == '(') {
            dp[end] = 0;
            return dp[end];// base case
        }

        // 3) end位置为')'
        // 3.1) end-1位置(必存在)为'(', 则匹配end位置的')', 此时最大匹配长度还要看end-2位置的
        if(s.charAt(end - 1) == '(') {
            if(end - 2 > -1) {
                dp[end] = 2 + f(s, end - 2);
                return dp[end];
            } else {
                dp[end] = 2;
                return dp[end];
            }
        } 
        
        // 3.2) end-1位置为')', 说明不匹配end位置的')', 此时还要继续往前找左括号
        // 最前的左括号位置可能等于 = 当前位置 - 1 - (end-1)的最大匹配长度
        int prelen = f(s, end - 1);
        int l = end - 1 - prelen;

        // 远处存在'(', 则当前的最大匹配长度为: 2 + prelen + f(s, l-1);
        if(l > -1 && s.charAt(l) == '(') {
            dp[end] = 2 + prelen + f(s, l - 1);
            return dp[end];
        } else {
            dp[end] = 0;
            return dp[end];
        }
    }
}
```

##### 7）一维严格表结构优化 | O（n）

- **思路**：
  1. 在基于一维记忆化搜索进行了优化，通过分析表中的值依赖关系，替换掉递归。
  2. 由于一个正常的格子，依赖于左边第一个，或者左边两个，所以 dp 表需要从左到右进行初始化。
- **结论**：效率非常高（1ms，100%），根据依赖关系，直接在表上就可以完成值设置，面试必过！

```java
class Solution {

    public int longestValidParentheses(String s) {
        if(s == null || s.length() < 2) {
            return 0;
        }

        int[] dp = new int[s.length()];

        // 1) 如果end位置不够或者越界, 说明本子串为非法的括号串, 返回0
        for(int end = 1; end < s.length(); end++) {
            // 2) 如果end位置为'(', 说明本子串为非法的括号串, 返回0
            if(s.charAt(end) == '(') {
                dp[end] = 0;
                continue;
            }

            // 3) end位置为')'
            // 3.1) end-1位置(必存在)为'(', 则匹配end位置的')', 此时最大匹配长度还要看end-2位置的
            if(s.charAt(end - 1) == '(') {
                if(end - 2 > -1) {
                    dp[end] = 2 + dp[end - 2];
                    continue;
                } else {
                    dp[end] = 2;
                    continue;
                }
            } 
            
            // 3.2) end-1位置为')', 说明不匹配end位置的')', 此时还要继续往前找左括号
            // 最前的左括号位置可能等于 = 当前位置 - 1 - (end-1)的最大匹配长度
            int prelen = dp[end - 1];
            int l = end - 1 - prelen;

            // 远处存在'(', 则当前的最大匹配长度为: 2 + prelen + f(s, l-1);
            if(l > -1 && s.charAt(l) == '(') {
                if(l - 1 > -1) {
                    dp[end] = 2 + prelen + dp[l-1];
                } else {
                    dp[end] = 2 + prelen;   
                }
            } else {
                dp[end] = 0;
            }
        }

        int max = Integer.MIN_VALUE;
        for(int i = 0; i < s.length(); i++) {
            max = Math.max(max, dp[i]);
        }

        return max;
    }
}
```

#### 44. 通配符匹配 | 线性 dp | hard

##### 1）线性 dp + 递推 | O（pn + sn * pn）

- **思路**：思路见代码注释。
- **结论**：时间，13 ms，88.46%，空间，42.1 mb，15.01%，时间上，初始化 dp 花费 O（pn），状态转移花费 O（sn * pn），所以整体时间复杂度为 O（pn + sn * pn），空间上，由于使用了一张 sn * pn 的二维表，所以额外空间复杂度为 O（sn * pn）。

```java
class Solution {
    public boolean isMatch(String s, String p) {
        // 定义状态：dp[i][j]表示, s的前i个字符, 和p的前j个字符, 是否完全匹配
        char[] charsS = s.toCharArray();
        char[] charsP = p.toCharArray();
        int sn = charsS.length, pn = charsP.length;
        boolean[][] dp = new boolean[sn + 1][pn + 1];

        // 初始化状态：
        // dp[0][0], 表示p的空模式串完全匹配s空串, 肯定为true
        // dp[i][0]表示, p的空模式串是否完全匹配s串, 此时肯定为false, 默认, 无需显示赋值
        // dp[0][j]表示, p的前j个字符是否匹配空串, 因此也就只有为*号时, 才等于true
        dp[0][0] = true;
        for(int j = 1; j <= pn; j++) {
            if(charsP[j - 1] == '*') {
                dp[0][j] = true;
            } else {
                // 如果当前不为*, 那么后面也不用判断了, 直接为false即可
                break;
            }
        }

        // 状态转移：从上到下、从左到右进行转移
        for(int i = 1; i <= sn; i++) {
            for(int j = 1; j <= pn; j++) {
                char sc = charsS[i - 1];
                char pc = charsP[j - 1];
                
    // 当p#j为'?'时, s#i字符不限, 但s#前i-1个字符, 都必须和p#前j-1个字符, 完全匹配, 状态才为true
                if(pc == '?') {
                    dp[i][j] = dp[i - 1][j - 1];
                }
    // 当p#j为'*'时, 需要分为不用'*', 即看作是空串, 以及用'*', 即看作是任意长度的任意字符串, 这两种情况 
    // 1) 当不用'*', 即看作是空串, 此时, p#前j-1个字符, 必须和s#前i个字符, 完全匹配, 状态才为true
    // 2) 当用'*', 此时则看p#前j个字符, 能否和s#前i-1个字符匹配, 能的话'*'就代表任意长度的任意字符串了
    // 比如s=abcdef, p=abc*, 如果p#abc*和s#abc匹配, 那么*就能够代表def了
    // 而在判断p#abc*和s#abc时, 又可以把*看作是空串, 即p#前j-1个完全匹配s#前i个, 所以为true  
                else if(pc == '*') {
                    dp[i][j] = dp[i][j - 1] || dp[i - 1][j];
                }
    // 当p#j为字母时, s#i必须等于p#j, 且s#前i-1个字符, 都必须和p#前j-1个字符, 完全匹配, 状态才为true
                else {
                    dp[i][j] = sc == pc && dp[i - 1][j - 1];
                } 
            }
        }

        // 获取结果：dp[i][j]表示, s的前i个字符, 和p的前j个字符, 是否完全匹配, 所以返回dp[sn][pn]即可
        return dp[sn][pn];
    }
}
```

#### 45. 跳跃游戏 II | 线性 dp | medium

##### 1）线性 dp + 暴力枚举 | O（n^2）

- **思路**：状态转移思路是，枚举前 i - 1 个起跳点，看能否跳到目前的 i 位置，如果可以，则取跳跃数的最小值即可。
- **结论**：时间，223 ms，7.14%，空间，41.8 mb，58.89%，时间上，由于需要设置 dp 数组，且每次设置都需要枚举前面 i - 1 个元素，所以时间复杂度为 O（n^2），空间上，由于使用了一张 n 长的 dp 数组，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public int jump(int[] nums) {
        // 定义状态：dp[i]表示跳到i位置, 所需要的最少跳跃次数
        int n = nums.length;
        int[] dp = new int[n];

        // 初始化状态：dp[0]为0
        dp[0] = 0;

        // 状态转移：dp[1] = dp[0] + 1
        // dp[2] = Math.min(nums[0] >= 2? dp[0] + 1 : max, nums[1] >= 1? dp[1] + 1 : max)
        // dp[n] = Math.min( for(nums[j] >= i - i? dp[j] + 1 : max) )
        int min;
        for(int i = 1; i < n; i++) {
            min = Integer.MAX_VALUE;
            for(int j = i - 1; j > -1; j--) {
                if(nums[j] >= i - j) {
                    min = Math.min(min, dp[j] + 1);
                }
            }
            dp[i] = min;
        }

        // 获取结果：dp[n - 1]
        return dp[n - 1];
    }
}
```

##### 2）贪心 + 逆向倒推 + 暴力枚举 | O（n^2）

- **思路**：见代码注释。
- **结论**：时间，11 ms，28.29%，空间，42.1 mb，17.31%，时间上，由于最坏情况下，起跳点每次都是在目的点的前一个，导致需要枚举 n 次目的点、n 次起跳点，所以时间复杂度为 O（n^2），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int jump(int[] nums) {
        int pos = nums.length - 1;
        int steps = 0;
        
        // 从目的点开始倒推, 直到回到原点
        while(pos > 0) {
            // 从头枚举起跳点, 保证能够跳到pos的起跳点是最远的
            for(int i = 0; i < pos; i++) {
                // 找到最远的起跳点后, 则交换i为下一轮的目的点, 继续叠加步数
                if(nums[i] >= pos - i) {
                    pos = i;
                    steps++;
                    break;
                }
            }
            // 根据题意, 不存在跳不到pos的情况
        }

        return steps;
    }
}
```

##### 3）贪心 + 正向递推 + 区间枚举 | O（n）

- **思路**：见代码注释。
- **结论**：时间，1 ms，99.01%，空间，42.2 mb，10.01%，时间上，由于只需要遍历一次数组，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int jump(int[] nums) {
        // 初始时, end边界和furthest最远能跳到的点, 都为0, 表示在远点
        int n = nums.length, steps = 0, end = 0, furthest = 0;

        // i=n-2时, end边界肯定>=n-1, 所以无需处理n-1的点, 避免边界=n-1的重复计算问题
        for(int i = 0; i < n - 1; i++) {
            // 每次都更新最远能跳到的点
            furthest = Math.max(furthest, i + nums[i]);

            // 如果i来到end边界了, 说明本段距离, 所有起跳点都枚举完毕, 贪心取最远的一个起跳点即可
            if(i == end) {
                end = furthest;
                steps++;
            }
        }

        return steps;
    }
}
```

#### 53. 最大子数组和 | 线性 dp| easy

##### 1）暴力递归1 | O（n^2）

- **思路**：
  1. 设计一个 f（end）函数，代表以 end 结尾的最大子数组的和是多少。
  2. 如果要计算的是以 end 为结尾的子数组之和，那么分为以 end 为结尾，长度分别为 0,1,2...i 的子数组，然后分别求他们的累加和，即得到的以 end 结尾子数组的最大累加和。
  3. 然后再与 end-1 结尾子数组的最大累加和比较，把较大者返回，即得到以 end 结尾的最大子数组和了。
- **结论**：
  1. 执行超时，每次以 end 结尾的 f（end）调用，都会重新算 0~end-1 的和，这是多余的计算，需要被优化掉。
  2. 经过加 dp 缓存优化，发现依然超时，经验告诉我，这应该是一个**通过不了**的尝试，请换一种方向思考。

```java
class Solution {
    public int maxSubArray(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }
        return f(nums, nums.length - 1);
    }

    // f代表以end结尾的最大子数组的值
    private int f(int[] nums, int end) {
        if(end < 0) {
            return 0;
        }
        if(end == 0) {
            return nums[end];
        }

        int max = Integer.MIN_VALUE, sum = 0;
        for(int i = 0; i < end + 1; i++) {
            sum += nums[end - i];
            max = Math.max(max, sum);
        }

        return Math.max(max, f(nums, end - 1));
    }
}
```

##### 2）暴力递归2 | O（n^2）

- **思路**：
  1. 还是沿用暴力递归1 的 f（end）函数，代表以 end 结尾的最大子数组的和是多少。
  2. 但不同的在于，暴力递归1 中 f 函数求最大和是通过遍历的方式实现的，而当前方法只是比对前一个过程的最大和，最后才遍历每个 f 函数。
- **结论**：还是执行超时，还是 O（n^2）的解，因为最后遍历每个 f 函数，其实有很多 f 是重新算了，这个可以通过 dp 缓存数组来优化。

```java
class Solution {

    public int maxSubArray(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        int max = Integer.MIN_VALUE;
        for(int i = 0; i < nums.length; i++) {
            max = Math.max(max, f(nums, i));
        }
        
        return max;
    }

    // f代表以end结尾的最大子数组的值
    private int f(int[] nums, int end) {
        if(end < 0) {
            return 0;
        }
        if(end == 0) {
            return nums[end];
        }
        return Math.max(nums[end], f(nums, end - 1) + nums[end]);
    }
}
```

##### 3）记忆化搜索优化 | <= O（n^2）

- **思路**：在暴力递归2 的基础上，增加了 dp 缓存数组，每次取数先取缓存数组中取，每次返回前先设置缓存数组。
- **结论**：时间，5ms，5.58%，空间，47.2mb，80.43%，时间还是很慢，可能是由于很多无效递归导致，还不如直接遍历数组来的高效~

```java
class Solution {

    private int[] dp;

    public int maxSubArray(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        dp = new int[nums.length];
        Arrays.fill(dp, -1);

        int max = Integer.MIN_VALUE;
        for(int i = 0; i < nums.length; i++) {
            max = Math.max(max, f(nums, i));
        }
        
        return max;
    }

    // f代表以end结尾的最大子数组的值
    private int f(int[] nums, int end) {
        if(end < 0) {
            return 0;
        }
        if(dp[end] != -1) {
            return dp[end];
        }
        if(end == 0) {
            dp[end] = nums[end];
            return dp[end];
        }
        dp[end] = Math.max(nums[end], f(nums, end - 1) + nums[end]);
        return dp[end];
    }
}
```

##### 4）严格表结构优化 | O（2n）

- **思路**：在记忆化搜索的基础上，把原本递归获取值，改成了从一维数组中获取值，大大提高了效率。
- **结论**：时间，2ms，46.16%，空间，47.3mb，77.50%，虽然提升了不少效率，但是还是需要遍历两次，可以再合并优化一下。

```java
class Solution {

    public int maxSubArray(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        int[] dp = new int[nums.length];
        dp[0] = nums[0];
        for(int end = 1; end < nums.length; end++) {
            dp[end] = Math.max(nums[end], dp[end-1] + nums[end]);
        }

        int max = Integer.MIN_VALUE;
        for(int i = 0; i < nums.length; i++) {
            max = Math.max(max, dp[i]);
        }
        
        return max;
    }
}
```

##### 5）合并后优化 | O（n）

- **思路**：在严格表结构的基础上，合并了两次的遍历。
- **结论**：时间，2ms，46.16%，空间，47.2mb，80.22%，可见**同一指数的时间复杂度差距是不大的**，但由于减少了一次遍历，能**稍微减少空间的使用**。

```java
class Solution {

    public int maxSubArray(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        int[] dp = new int[nums.length];
        dp[0] = nums[0];

        int max = nums[0];
        for(int end = 1; end < nums.length; end++) {
            dp[end] = Math.max(nums[end], dp[end-1] + nums[end]);
            max = Math.max(max, dp[end]);
        }

        return max;
    }
}
```

##### 6）贪心 | O（n）

- **思路**：利用暴力递归2 的思路，即 nums[end], f(nums, end - 1) + nums[end] 取最大值，代表要么取当前数字，要么取之前的数组+当前数字，但与其不同的地方在于，一次 f 就能调完的了，就根本不用在外层又进行一套 max 比较了。
- **结论**：时间，1ms，100%，48.8mb，6.73%，由于只遍历一次，所以对比暴力递归2 省了一层遍历。

```java
class Solution {
    public int maxSubArray(int[] nums) {
       if(nums == null || nums.length == 0) {
            return 0;
        }

        int max = nums[0], cur = nums[0];
        for(int i = 1; i < nums.length; i++) {
            cur = Math.max(cur + nums[i], nums[i]);
            max = Math.max(max, cur);
        }

        return max;
    }
}
```

#### 55. 跳跃游戏 | 线性 dp | medium

##### 1）暴力递归 | O（n^2）

- **思路**：
  1. 设计一个 f（end）函数，代表从 0 跳到 end 能否完成。
  2. f（end）的实现通过 end 再往前 0,1,2... 个长度进行判断，比如，如果 end-1 位置的值大于 1，且 f（end-1）为 true，那么说明 0~end-1位置可以到达，且 end-1~end 也可以到达，即 f（end）可以到达。
- **结论**：执行超时，因为重复递归了很多次，可以继续优化。

```java
class Solution {
    public boolean canJump(int[] nums) {
        if(nums == null || nums.length <= 1) {
            return true;
        }
        if(nums[0] == 0) {
            return false;
        }

        return f(nums, nums.length - 1);
    }

    // f代表从0跳到end能否完成
    private boolean f(int[] nums, int end) {
        if(end <= 1) {
            return true;
        }

        for(int len = 1; len <= end; len++) {
            if(nums[end - len] >= len) {
                if(f(nums, end - len)) {
                    return true;
                }
            }
        }

        return false;
    }
}
```

##### 2）记忆化搜索优化 | <= O（n^2）

- **思路**：延续暴力递归的思路，增加了 dp 缓存表，获取前先从缓存中获取，返回前先设置到缓存中。
- **结论**：时间，4ms，21%，空间，40.4mb，16%，效率提升了不少，看还能不能用动态规划优化。

```java
class Solution {

    private Boolean[] dp;

    public boolean canJump(int[] nums) {
        if(nums == null || nums.length <= 1) {
            return true;
        }
        if(nums[0] == 0) {
            return false;
        }

        dp = new Boolean[nums.length];
        return f(nums, nums.length - 1);
    }

    // f代表从0跳到end能否完成
    private boolean f(int[] nums, int end) {
        if(dp[end] != null) {
            return dp[end];
        }
        if(end <= 1) {
            return true;
        }

        for(int len = 1; len <= end; len++) {
            if(nums[end - len] >= len) {
                if(f(nums, end - len)) {
                    dp[end] = true;
                    return dp[end];
                }
            }
        }

        dp[end] = false;
        return dp[end];
    }
}
```

##### 3）严格表结构优化 | O（n^2）

- **思路**：经过研究值依赖，可以发现，右边的值依赖左边，也就是需要从左到右的初始化。
- **结论**：时间，37ms，16.27%，39.9mb，20%，这是一道记忆化搜索效率由于严格表结构的题，可是为什么？

```java
class Solution {
    public boolean canJump(int[] nums) {
        if(nums == null || nums.length <= 1) {
            return true;
        }
        if(nums[0] == 0) {
            return false;
        }

        boolean[] dp = new boolean[nums.length];
        dp[0] = dp[1] = true;
        
        for(int end = 2; end < nums.length; end++) {
            for(int len = 1; len <= end; len++) {
                if(nums[end - len] >= len && dp[end - len]) {
                    dp[end] = true;
                    break;
                }
            }
        }

        return dp[nums.length-1];
    }
}
```

##### 4）贪心1 | O（n）

- **思路**：每个位置都记录自身索引 + 自身值的和，代表能够跳到最远的索引位置，如果遍历过程中，发现某个索引 i 已经超过了当前最大能够到达的索引，则返回 false，否则返回 true。
- **结论**：时间，2ms，95.03%，空间，39.9mb，22.14%，效率非常高，贪心不难写，难在想出来，如果用动态规划写出来发现效率不高，那么可以试着想想用贪心实现了。

```java
class Solution {
    public boolean canJump(int[] nums) {
        if(nums == null || nums.length <= 1) {
            return true;
        }
        if(nums[0] == 0) {
            return false;
        }

        int max = nums[0];
        for(int i = 1; i < nums.length; i++) {
            if(i > max) {
                return false;
            }
            max = Math.max(i + nums[i], max);
        }

        return true;
    }
}
```

##### 5）贪心2 | O（n）

- **思路**：类似于贪心1，比较的也是索引的位置，但贪心2研究的是从最后节点出发，看前一个节点能不能到达上一个节点，如果可以则继续更新，否则就不更新，最后通过判断上一个节点是否为 0 索引位置，即得知是否完成从 0~end 的跳跃了。
- **结论**：时间，1ms，100%，40mb，12.21%，效率和贪心1差不多，都是 O（n）。

```java
class Solution {
    public boolean canJump(int[] nums) {
        if(nums == null || nums.length <= 1) {
            return true;
        }
        if(nums[0] == 0) {
            return false;
        }

        int last = nums.length - 1;
        for(int i = nums.length - 2; i > -1; i--) {
            if(i + nums[i] >= last) {
                last = i;
            }
        }
        
        return last == 0;
    }
}
```

#### 62. 不同路径 | 线性 dp | medium

##### 1）线性 dp | O（m * n）

- **思路**：见代码注释。
- **结论**：时间，0 ms，100%，空间，38.1 mb，68.69%，时间上，由于需要遍历 m * n 个点，所以时间复杂度为 O（m * n），空间上，由于使用了一张 m * n 的二维表，所以额外空间复杂度为 O（m * n）。

```java
class Solution {
    public int uniquePaths(int m, int n) {
        // 定义状态: (i, j)代表i行j列到达终点(m-1,n-1)的位置
        int[][] dp = new int[m][n];

        // 初始状态：初始时(m-1,n-1)为1
        dp[m - 1][n - 1] = 1;

        // 状态转移：从下到上, 从右往左转移
        for(int i = m - 1; i > -1; i--) {
            for(int j = n - 1; j > - 1; j--) {
                if(i == m - 1 && j == n - 1) {
                    continue;
                }

                // 看下边的点, 能否到终点
                if(i + 1 < m) {
                    dp[i][j] += dp[i + 1][j];
                }

                // 看右边的点, 能否到终点
                if(j + 1 < n) {
                    dp[i][j] += dp[i][j + 1];
                }
            }
        }

        // 获取结果
        return dp[0][0];
    }
}
```

#### 63. 不同路径 II | 线性 dp | medium

##### 1）线性 dp | O（m * n）

- **思路**：见代码注释，与《62. 不同路径》思路类似，不过要注意石头的判断，包括终点和过程点。
- **结论**：时间，0 ms，100%，空间，39.6 mb，53.82%，时间上，由于需要遍历 m * n 个点，所以时间复杂度为 O（m * n），空间上，由于使用了一张 m * n 的二维表，所以额外空间复杂度为 O（m * n）。

```java
class Solution {
    public int uniquePathsWithObstacles(int[][] grid) {
        int m = grid.length, n = grid[0].length;

        // 定义状态: (i, j)代表i行j列到达终点(m-1,n-1)的位置
        int[][] dp = new int[m][n];

        // 初始状态：初始时(m-1,n-1)为1, 还需要看是否有石头
        dp[m - 1][n - 1] = grid[m - 1][n - 1] == 0? 1 : 0;

        // 状态转移：从下到上, 从右往左转移
        for(int i = m - 1; i > -1; i--) {
            for(int j = n - 1; j > - 1; j--) {
                if(i == m - 1 && j == n - 1) {
                    continue;
                }
                if(grid[i][j] == 1) {
                    continue;
                }

                // 看下边的点, 能否到终点
                if(i + 1 < m) {
                    dp[i][j] += dp[i + 1][j];
                }

                // 看右边的点, 能否到终点
                if(j + 1 < n) {
                    dp[i][j] += dp[i][j + 1];
                }
            }
        }

        // 获取结果
        return dp[0][0];
    }
}
```

#### 64. 最小路径和 | 最短路径 | medium

##### 1）暴力递归 | O（m * n）

- **思路**：
  1. 设计一个 f（x，y）函数，代表从 [0,0] 走到 [x,y] 需要的最小数字总和。
  2. f 实现通过尝试获取左和获取右两边，来对比出最小的路径和，再加上自身的值，就等于来到当前格子的最小数字总和了。
- **结论**：执行超时，还需要继续优化。

```java
class Solution {
    public int minPathSum(int[][] grid) {
        if(grid == null) {
            return 0;
        }
        return f(grid, grid[0].length-1, grid.length-1);
    }

    // f代表从[0,0]走到[x,y]需要的最小数字总和
    private int f(int[][] grid, int x, int y) {
        if(x < 0 || y < 0) {
            return Integer.MAX_VALUE;
        }
        // 防止0位还继续往左或者往上走
        if(x == 0 && y == 0) {
            return grid[y][x];
        }

        return grid[y][x] + Math.min(f(grid, x-1, y), f(grid, x, y-1));
    }
}
```

##### 2）记忆化搜索 | O（m * n）

- **思路**：基于暴力递归的思路，建立一个 dp 二维数组缓存，如果缓存中存在则从缓存中获取，否则返回前先设置结果到缓存再返回。
- **结论**：时间，2ms，96.47%，41.3mb，20.92%，效率非常高，不过还可以优化，因为其中有很多 f 递归是被重复调用了的。

```java
class Solution {
    public int minPathSum(int[][] grid) {
        if(grid == null) {
            return 0;
        }
        
        int[][] dp = new int[grid.length][grid[0].length];
        for(int i = 0; i < dp.length; i++) {
            for(int j = 0; j < dp[i].length; j++) {
                dp[i][j] = -1;
            }
        }

        return f(grid, dp, grid[0].length-1, grid.length-1);
    }

    // f代表从[0,0]走到[x,y]需要的最小数字总和
    private int f(int[][] grid, int[][] dp, int x, int y) {
        if(x < 0 || y < 0) {
            return Integer.MAX_VALUE;
        }
        // 防止0位还继续往左或者往上走
        if(x == 0 && y == 0) {
            return grid[y][x];
        }
        if(dp[y][x] != -1) {
            return dp[y][x];
        }

        dp[y][x] = grid[y][x] + Math.min(f(grid, dp, x-1, y), f(grid, dp, x, y-1));
        return dp[y][x];
    }
}
```

##### 3）严格表结构优化 | O（m * n）

- **思路**：基于记忆化搜索的思路，研究了二维表的值依赖，发现每个值只依赖它的左边和上边，因此需要从左到右的初始化。
- **结论**：时间，2ms，96.47%，空间，40.8mb，93.74%，空间上还可以用一维表继续优化。

```java
class Solution {
    public int minPathSum(int[][] grid) {
        if(grid == null) {
            return 0;
        }
        
        int[][] dp = new int[grid.length][grid[0].length];
        dp[0][0] = grid[0][0];

        for(int y = 0; y < dp.length; y++) {
            for(int x = 0; x < dp[y].length; x++) {
                if(x - 1 > -1 && y - 1 > -1) {
                    dp[y][x] = grid[y][x] + Math.min(dp[y][x-1], dp[y-1][x]);
                } else if(x - 1 > -1){
                    dp[y][x] = grid[y][x] + dp[y][x-1];
                } else if(y - 1 > -1){
                    dp[y][x] = grid[y][x] + dp[y-1][x];
                }
            }
        }

        return dp[grid.length-1][grid[0].length-1];
    }
}
```

##### 4）空间压缩优化 | O（m * n）

- **思路**：基于严格表结构的思路，经过二维表值依赖的研究得知，每个值只依赖它的左边和上边，左边可以通过数组中上一个索引获取，而上边可以通过没赋值前的值获取，也就是完全可以把一个二维数组替换成一个一维数组，以减少空间的使用。
- **结论**：时间，2ms，96.47%，空间，41.2mb，37.59%，可能是用例不够多，导致优化出的效果不明显，但这已经是动态规划的最优解了。

```java
class Solution {
    public int minPathSum(int[][] grid) {
        if(grid == null) {
            return 0;
        }
        
        int[] dp = new int[grid[0].length];
        dp[0] = grid[0][0];

        for(int y = 0; y < grid.length; y++) {
            for(int x = 0; x < grid[0].length; x++) {
                if(x - 1 > -1 && y - 1 > -1) {
                    dp[x] = grid[y][x] + Math.min(dp[x-1], dp[x]);
                } else if(x - 1 > -1){
                    dp[x] = grid[y][x] + dp[x-1];
                } else if(y - 1 > -1){
                    dp[x] = grid[y][x] + dp[x];
                }
            }
        }

        return dp[grid[0].length-1];
    }
}
```

#### 72. 编辑距离 | 线性 dp | hard

##### 1）暴力递归 | O（n^2）

- **思路**：
  1. 设计一个 f（end1，end2）函数，代表从 [0...end1] 匹配 [0...end2] 需要的最少操作数。
  2. f（end1，end2）函数的实现，通过比较 word1 和 word2 的末尾字符来判断，如果字符相同，则认为匹配，此时最少操作数取决于 f（end1-1，end2-1）子过程的最少操作数。
  3. 如果字符不相同，则认为本次需要操作，即操作数 + 1，不过总的最少操作数还是要取决于子过程的最少操作数：
     1. 如果尝试了新增操作，那么 word1 就新增了一个匹配的字符，用于匹配 word2 的末尾字符，此时 word2 也少了一个字符，所以子过程的最少操作数为 f（end1，end2-1）。
     2. 如果尝试了删除操作，那么 word1 就删除了一个末尾字符，即 word1 少了一个字符，此时子过程的最少操作数为 f（end1-1，end2）。
     3. 如果尝试了替换操作，那么 word1 就需要替换一个末尾字符，即 word1 和 word2 的字符数都不变，此时子过程的最少操作为 f（end1-1，end2-1）。
     4. 最后只要返回最少操作数的那个尝试，再 +1 即等于当前 end1 和 end2 匹配的最少操作数。
- **结论**：执行超时，由于可能多次调用同一个参数的 f 递归，所以还有优化的空间。

```java
class Solution {
    public int minDistance(String word1, String word2) {
        if(word1 == null && word2 == null) {
            return 0;
        } else if(word1 == null) {
            return word2.length();
        } else if(word2 == null) {
            return word1.length();
        }

        return f(word1, word2, word1.length()-1, word2.length()-1);
    }

    // f代表从[0...end1]匹配[0...end2]需要的最少操作数
    private int f(String word1, String word2, int end1, int end2) {
        // 同时没有那么就不需要操作
        if(end1 < 0 && end2 < 0) {
            return 0;
        }
        // 如果匹配串没有, 但原串还有, 那么就需要插入多少
        else if(end1 < 0) {
            return end2 + 1;
        }
        // 如果原串没有, 但匹配串还有, 那么就需要删除多少
        else if(end2 < 0) {
            return end1 + 1;
        }
        // 否则其他情况, 则按递归去匹配
        // 如果end1等于end2, 那么则无需操作
        if(word1.charAt(end1) == word2.charAt(end2)) {
            return f(word1, word2, end1-1, end2-1);
        }
        // 否则枚举增删改的操作
        return 1 + Math.min(
            // 增
            f(word1, word2, end1, end2 - 1), 
            Math.min(
                // 删
                f(word1, word2, end1-1, end2), 
                // 改
                f(word1, word2, end1-1, end2-1)
            )
        );
    }
}
```

##### 2）记忆化搜索 | O（n）

- **思路**：基于暴力递归的思路，增加了 dp 二维表缓存，如果缓存中存在则从缓存中获取，否则返回前先把结果设置到缓存中再返回。
- **结论**：时间，3ms，99.64%，空间，38.7mb，5.03%，多次调用了无用的递归，还可以继续优化。

```java
class Solution {
    public int minDistance(String word1, String word2) {
        if(word1 == null && word2 == null) {
            return 0;
        } else if(word1 == null) {
            return word2.length();
        } else if(word2 == null) {
            return word1.length();
        }

        int[][] dp = new int[word1.length()][word2.length()];
        for(int i = 0; i < dp.length; i++) {
            for(int j = 0; j < dp[i].length; j++) {
                dp[i][j] = -1;
            }
        }

        return f(dp, word1, word2, word1.length()-1, word2.length()-1);
    }

    // f代表从[0...end1]匹配[0...end2]需要的最少操作数
    private int f(int[][] dp, String word1, String word2, int end1, int end2) {
        // 同时没有那么就不需要操作
        if(end1 < 0 && end2 < 0) {
            return 0;
        }
        // 如果匹配串没有, 但原串还有, 那么就需要插入多少
        else if(end1 < 0) {
            return end2 + 1;
        }
        // 如果原串没有, 但匹配串还有, 那么就需要删除多少
        else if(end2 < 0) {
            return end1 + 1;
        }
        if(dp[end1][end2] != -1) {
            return dp[end1][end2];
        }

        // 否则其他情况, 则按递归去匹配
        // 如果end1等于end2, 那么则无需操作
        if(word1.charAt(end1) == word2.charAt(end2)) {
            dp[end1][end2] = f(dp, word1, word2, end1-1, end2-1);
            return dp[end1][end2];
        }

        // 否则枚举增删改的操作
        dp[end1][end2] = 1 + Math.min(
            // 增
            f(dp, word1, word2, end1, end2 - 1), 
            Math.min(
                // 删
                f(dp, word1, word2, end1-1, end2), 
                // 改
                f(dp, word1, word2, end1-1, end2-1)
            )
        );
        return dp[end1][end2];
    }
}
```

##### 3）严格表结构优化 | O（n * m）

- **思路**：基于记忆化搜索的思路，把递归改成了二维表的动态规划，减少了空间的使用。
- **结论**：时间，6ms，20.34%，38.4mb，74.76%，改成了动态规划后发现，空间使用减少了，但时间复杂度变成了 O（n * m），所以二维表的动态规划并不是最优解。

```java
class Solution {
    public int minDistance(String word1, String word2) {
        if(word1 == null && word2 == null) {
            return 0;
        } else if(word1 == null) {
            return word2.length();
        } else if(word2 == null) {
            return word1.length();
        } else if(word1.length() == 0) {
            return word2.length();
        } else if(word2.length() == 0) {
            return word1.length();
        }

        int[][] dp = new int[word1.length()][word2.length()];
        for(int end1 = 0; end1 < dp.length; end1++) {
            for(int end2 = 0; end2 < dp[end1].length; end2++) {
                // 否则其他情况, 则按递归去匹配
                // 如果end1等于end2, 那么则无需操作
                if(word1.charAt(end1) == word2.charAt(end2)) {
                    dp[end1][end2] = getValue(dp, end1-1, end2-1);
                    continue;
                }

                // 否则枚举增删改的操作
                dp[end1][end2] = 1 + Math.min(
                    // 增
                    getValue(dp, end1, end2-1),
                    Math.min(
                        // 删
                        getValue(dp, end1-1, end2),
                        // 改
                        getValue(dp, end1-1, end2-1)
                    )
                );
            }
        }

        return dp[word1.length()-1][word2.length()-1];
    }

    private int getValue(int[][] dp, int end1, int end2) {
        int predictRes = predict(end1, end2);
        return predictRes != -1? predictRes : dp[end1][end2];
    }

    // 根据下标预测出操作数
    private int predict(int end1, int end2) {
        // 同时没有那么就不需要操作
        if(end1 < 0 && end2 < 0) {
            return 0;
        }
        // 如果匹配串没有, 但原串还有, 那么就需要插入多少
        else if(end1 < 0) {
            return end2 + 1;
        }
        // 如果原串没有, 但匹配串还有, 那么就需要删除多少
        else if(end2 < 0) {
            return end1 + 1;
        }
        else {
            return -1;
        }
    }
}
```

#### 87. 扰乱字符串 | 区间 dp | hard

##### 1）区间 dp + 枚举切割点 | O（n^2 + n^4）

- **思路**：

  1. 根据题意，扰乱字符串的关系具有对称性，即如果 s1 是 s2 的扰乱字符串，那么 s2 也是 s1 的扰乱字符串，可以称 s1 和 s2 是和谐的。
  2. 所以，状态转移可以通过枚举切割点，然后对切割后的结果是否交换顺序，进而分为两种情况：
     - 1）一种是，l（s1）和 r（s1）没有发生交换，那么要想 s2 与 s1 和谐，s2 的切割点需要落在相同的位置，使得 s2 = l（s2）+ r（s2），有 l（s1）与 l（s2）和谐，且 r（s1）与 r（s2）和谐。
     - 2）另外一种是，l（s1）和 r（s1）发生了交换，那么要想 s2 与 s1 和谐，s2 的切割点需要落在相反的位置，使得 s2 = l（s2）+ r（s2），有 r（s1）与 l（s2）和谐，且 l（s1）与 r（s2）和谐。

  ![1658828283689](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1658828283689.png)

- **结论**：时间，6 ms，77.76%，空间，41.7 mb，53.40%，时间上，由于初始化抓状态花费 O（n^2），状态转移花费 O（n^4），所以整体时间复杂度为 O（n^2 + n^4），空间上，由于使用了 n 长的 chars1 和 chars2，以及一张 n^3 长的 dp 三微表，所以额外空间复杂度为 O（2 * n + n^3）。

```java
class Solution {

    public boolean isScramble(String s1, String s2) {
        int n = s1.length();

        // 定义状态：长度为0时, 默认为false
        // 其余的表示, s1从i1开始、s2从i2开始、其长度均为len时, 是否和谐
        boolean[][][] dp = new boolean[n][n][n + 1];

        // 初始化状态：只有一个字符时, 则看字符是否相等
        char[] chars1 = s1.toCharArray();
        char[] chars2 = s2.toCharArray();
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < n; j++) {
                dp[i][j][1] = chars1[i] == chars2[j];
            }
        }

        // 状态转移：从下到上转移
        for(int len = 2; len <= n; len++) {
            // 枚举s1的开头, 至少n-i+1>=len, 即还剩len个字符, 所以i<=n-len+1, 而i从0开始, 所以i<n-len+1
            for(int i = 0; i < n - len + 1; i++) {
                // 枚举s2的开头, 至少n-j+1>=len, 即还剩len个字符, 所以j<=n-len+1, 而j从0开始, 所以j<n-len+1
                for(int j = 0; j < n - len + 1; j++) {
                    // 如果字符串不同、各字符出现的次数相等, 那么还是有机会和谐的, 不过要进行扰动判断
                    // 枚举s1的切点, [1, n-1], 表示在i索引左侧进行切割
                    for(int k = 1; k < len; k++) {
                        // 如果不交换切割后的字串, 则看原来顺序的[0, k]是否和谐, 以及[k+1, n-k]是否和谐
                        // 即：l(s1)和谐l(s2), r(s1)和谐r(s2), 那么s1=l(s1)+r(s1)和谐s2=l(s2)+r(s2)
                        if(dp[i][j][k] && dp[i + k][j + k][len - k]) {
                            dp[i][j][len] = true;
                            break;
                        }

                        // 如果交换切割后的字符串, 则看交换后的[0, k]是否和谐, 以及[k+1, n-k]是否和谐
                        // 即：l(s1)和谐r(s2), r(s1)和谐l(s2), 那么s1=l(s1)+r(s1)和谐s2=l(s2)+r(s2)
                        if(dp[i][j + (len - k)][k] && dp[i + k][j][len - k]) {
                            dp[i][j][len] = true;
                            break;
                        }
                    }
                }
            }
        }

        // 获取结果
        return dp[0][0][n];
    }
}
```

#### 91. 解码方法 | 线性 dp | medium

##### 1）线性 dp | O（n）

- **思路**：见代码注释。
- **时间**：时间，0 ms，100.00%，空间，39.7 mb，55.41%，时间上，由于只需要遍历一次 s 字符串，所以时间复杂度为 O（n），空间上，由于使用了一张 n 长的 dp 数组，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public int numDecodings(String s) {
        // 定义状态：dp代表以[0, i-1]的字符串的方案数
        int n = s.length();
        int[] dp = new int[n + 1];

        // 初始化状态：i=0, 代表空串的方案数, 因为-1在s中找不到字符
        dp[0] = 1;

        // 状态转移：从左到右转移, 从s的第一个字符0开始判断
        for(int i = 1; i <= n; i++) {
            // 如果当前位不为0, 则可以从i-1转移过来
            int cur = s.charAt(i - 1) - '0';
            if(cur != 0) {
                dp[i] += dp[i - 1];
            }

            // 如果前一位不为0, 且组合起来小于26, 则可以从i-2转移过来
            if(i - 2 > -1) {
                int pre = s.charAt(i - 2) - '0';
                if(pre != 0 && 10 * pre + cur <= 26) {
                    dp[i] += dp[i - 2];
                }
            }
        }

        // 获取结果：i=n, 代表[0, n-1]作为字符串的方案数
        return dp[n];
    }
}
```

##### 2）线性 dp + 空间压缩 | O（n）

- **思路**：基于 1 的解法，由于当前的方案数，只依赖于 i - 1 和 i - 2 的方案数，所以可以把 dp 数组，替换成 curCnt 代表当前方案数、preCnt 代表 i - 1的方案数，prepreCnt 代表 i - 2 的方案数，然后每计算完一个 i 的方案数后，更新 curCnt、preCnt、prepreCnt 的值即可。
- **结论**：时间，0 ms，100%，空间，40 mb，18.34%，时间上，由于只需要遍历一次 s 字符串，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int numDecodings(String s) {
        // 定义状态：dp代表以[0, i-1]的字符串的方案数
        int n = s.length();
        int[] dp = new int[n + 1];

        // 初始化状态：i=0, 代表空串的方案数, 因为-1在s中找不到字符
        dp[0] = 1;

        // 状态转移：从左到右转移, 从s的第一个字符0开始判断
        int prepreCnt = 0, preCnt = 1, curCnt = 0;
        for(int i = 1; i <= n; i++) {
            curCnt = 0;

            // 如果当前位不为0, 则可以从i-1转移过来
            int cur = s.charAt(i - 1) - '0';
            if(cur != 0) {
                curCnt += preCnt;
            }

            // 如果前一位不为0, 且组合起来小于26, 则可以从i-2转移过来
            if(i - 2 > -1) {
                int pre = s.charAt(i - 2) - '0';
                if(pre != 0 && 10 * pre + cur <= 26) {
                    curCnt += prepreCnt;
                }
            }

            prepreCnt = preCnt;
            preCnt = curCnt;
        }

        // 获取结果：i=n, 代表[0, n-1]作为字符串的方案数
        return curCnt;
    }
}
```

#### 97. 交错字符串 | 线性 dp| medium

##### 1）线性 dp | O（n1 * n2）

- **思路**：见代码注释。
- **结论**：时间，7 ms，13.95%，空间，39.5 mb，72.09%，时间上，由于状态转移需要花费 O（n1 * n2），所以时间复杂度为 O（n1 * n2），空间上，由于使用了一张 n1 * n2 长的 dp 二维表，所以额外空间复杂度为 O（n1 * n2）。

```java
class Solution {
    public boolean isInterleave(String s1, String s2, String s3) {
        int n1 = s1.length(), n2 = s2.length(), n3 = s3.length();
        if(n1 + n2 != n3) {
            return false;
        }

        // 定义状态：dp[i][j]表示, s1的前i个字符+s2的前j个字符, 是否和s3的前i+j个字符相等
        boolean[][] dp = new boolean[n1 + 1][n2 + 1];

        // 初始化状态：dp[0][0]表示, s1==s2==s3==''
        dp[0][0] = true;

        // 状态转移：从上到下、从左到右进行转移
        // dp[i][0]表示, s1前i个字符, s2='', 是否等于s3前i个字符
        // dp[0][j]表示, s2前j个字符, s1='', 是否等于s3前j个字符
        // 这里把dp[i][0]、dp[0][j]放到了状态转移里面, 所以每次都需要控制i和j不能同时为0, 防止更新了初始值
        for(int i = 0; i <= n1; i++) {
            for(int j = 0; j <= n2; j++) {
                // dp[i][j]最后一步, 可以由s1匹配出来, 即s1第i个字符等于s3的第i+j-1个字符 => s1[i-1]==s3[i+j-1], 以及s1的前i-1个字符+s2的前j个字符, 和s3的前i+j-1个字符相等 => dp[i-1][j]
                if(i - 1 > -1) {
                    dp[i][j] |= s1.charAt(i - 1) == s3.charAt(i + j - 1) && dp[i - 1][j];
                }
                
                // dp[i][j]最后一步, 也可以由s2匹配出来, 即s2第j个字符等于s3的第i+j-1个字符 => s2[j-1]==s3[i+j-1], 以及s2的前j-1个字符+s1的前i个字符, 和s3的前i+j-1个字符相等 => dp[i][j-1]
                if(j - 1 > -1) {
                    dp[i][j] |= s2.charAt(j - 1) == s3.charAt(i + j - 1) && dp[i][j - 1];
                }
            }
        }
        
        // 获取结果：dp[n1][n2]表示, 获取s1前n1个字符+s2前n2个字符, 是否和s3的前n1+n2个字符相等
        return dp[n1][n2];
    }
}
```

##### 2）线性 dp + 空间压缩 | O（n1 * n2）

- **思路**：
  1. 经过表结构依赖分析发现，当前 i 行依赖 i - 1行 j 列的结果，当前 j 列依赖 i 行 j - 1列的结果，所以把 dp 二维表替换为一维表，减少了一共 n1 行的存储。
  2. 但要从上到下、从左到右进行转移，来保证 s1 判断用到 dp 数组时，可以看作是 i - 1 行的结果，经过s1 判断更新 dp 数组，相当于更新了当前 i 行的结果，再提供 i 行的结果给 s2 判断使用，使得 s2 判断时用到的 dp[j - 1] 都是当前行更新后的，j - 1从左更新过的最新数据。
  3. 其中要注意的是，由于此时 i、j 的状态判断。都复用了 dp[j]，所以不能像 1 的解法那样，简单地通过 `|=` 来合并两个状态的判断结果，而是需要手动改用 `= ? || ?` 的方式来合并判断结果。
- **结论**：时间，6 ms，24.11%，空间，39.4 mb，82.87%，时间上，由于状态转移需要花费 O（n1 * n2），所以时间复杂度为 O（n1 * n2），空间上，由于使用了一张 n2 长的 dp 一维表，所以额外空间复杂度为 O（n2）。

```java
class Solution {
    public boolean isInterleave(String s1, String s2, String s3) {
        int n1 = s1.length(), n2 = s2.length(), n3 = s3.length();
        if(n1 + n2 != n3) {
            return false;
        }

        // 空间压缩：dp[j]表示s2的前j个字符, 是否与s3的前i+j个字符相等, 其中i使用for循环的变量来表示
        boolean[] dp = new boolean[n2 + 1];
        dp[0] = true;

        // 状态转移：从上到下、从左到右进行转移
        for(int i = 0; i <= n1; i++) {
            for(int j = 0; j <= n2; j++) {
                // i=0时, 第一行不会用上, 所以初始时dp[j]表示的就是, s1为空串时的结果
                if(i - 1 > -1) {
                    // 这里把当前dp数组, 看作成i-1行, 利用完了dp[j]后, 更新为dp[j], 代表设置了第i行的结果
                    dp[j] = s1.charAt(i - 1) == s3.charAt(i + j - 1) && dp[j];
                }
                // 由于这里要读取的是, 当前行的第j-1列, 所以需要放到i更新了当前行之后
                if(j - 1 > -1) {
                    // 由于这里i、j的状态判断, 都复用了dp[j], 所以不能像以前那样通过|=来合并两个状态的结果
                    // 而是手动改用 = ? || ? 的方式来合并结果
                    dp[j] = dp[j] || (s2.charAt(j - 1) == s3.charAt(i + j - 1) && dp[j - 1]);
                }
            }
        }

        return dp[n2];
    }
}
```

#### 115. 不同的子序列 | 线性 dp | hard

##### 1）线性 dp | O（m * n）

- **思路**：见代码注释。
- **结论**：时间，5 ms，95.31%，空间，48.5 mb，41.77%，时间上，由于状态转移花费了 O（m * n），所以时间复杂度为 O（m * n），空间上，由于使用了一张 m 长的 schars、n 长的 tchars、m * n 长的 dp 数组，所以额外空间复杂度为 O（m + n + m * n）。

```java
class Solution {
    public int numDistinct(String s, String t) {
        char[] schars = s.toCharArray();
        char[] tchars = t.toCharArray();
        int m = schars.length, n = tchars.length;
        if(m < n) {
            return 0;
        }

        // 定义状态：dp[i][j]表示, s[i:]往后的字符串, 包含t[j:]往后的子序列的方案数
        int[][] dp = new int[m + 1][n + 1];

        // 初始化状态：j=n时, 表示s包含空串的方案数, 为1; i=m时, 表示空串包含t子序列的方案数, 为0
        for(int i = 0; i <= m; i++) {
            dp[i][n] = 1;
        }
        // 省略dp[m][j] = 0;

        // 状态转移：s[i]==t[j], 那么可以选i、也可以不选i; s[i]!=t[j], 那么只能不选[i]
        // 从下到上、从右到左, 进行状态转移
        for(int i = m - 1; i > -1; i--) {
            for(int j = n - 1; j > -1; j--) {
                if(schars[i] == tchars[j]) {
                    dp[i][j] =
                        // 选i 
                        dp[i + 1][j + 1]
                        // 不选i
                        + dp[i + 1][j];
                }
                else {
                    // 不选i
                    dp[i][j] = dp[i + 1][j];
                }
            }
        }

        // 获取结果：dp[0][0]表示, s完整串包含t完整子序列的方案数
        return dp[0][0];
    }
}
```

#### 118. 杨辉三角 | 线性 dp | easy

##### 1）公式递推 + 线性 dp | O（ [ n^2 + n ] / 2 ）

- **思路**：见杨辉三角性质以及代码注释：

  ![1660640133638](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660640133638.png)

- **结论**：时间，0 ms，100%，空间，39.4 mb，23.76%，时间上，由于第 1 行有 1 个、第 2 行有 2 个 ... 第 n 行有 n 个位置，即一共需要设置 1 + 2 + ... + n = (n + 1) * n / 2 = [ n^2 + n ] / 2 个位置，所以整体时间复杂度为 O（ [ n^2 + n ] / 2 ），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public List<List<Integer>> generate(int n) {
        List<List<Integer>> res = new ArrayList<>();
        
        // 从第0行开始, 存在n-1列
        List<Integer> cur, pre;
        for(int i = 0; i < n; i++) {
            cur = new ArrayList<>();
            res.add(cur);

            // 从第0列开始, 第i行存在i+1列
            for(int j = 0; j < i + 1; j++) {
                // 每行的第0列和最后一列, 规定为1
                if(j == 0 || j == i) {
                    cur.add(1);
                }
                // 每行的中间列: (i, j) = (i-1, j-1) + (i-1, j)
                else {
                    pre = res.get(i - 1);
                    cur.add(pre.get(j - 1) + pre.get(j));
                }
            }
        }
        
        return res;  
    }
}
```

#### 119. 杨辉三角 II | 线性 dp | easy

##### 1）公式递推 + 线性 dp | O（ [ n^2 + n ] / 2 ）

- **思路**：参考《118. 杨辉三角》，不过这里使用 pre，把二维表优化成了一维表。
- **结论**：时间，1 ms，77.25%，空间，39.4 mb，18.75%，时间上，由于第 1 行有 1 个、第 2 行有 2 个 ... 第 n 行有 n 个位置，即一共需要设置 1 + 2 + ... + n = (n + 1) * n / 2 = [ n^2 + n ] / 2 个位置，所以整体时间复杂度为 O（ [ n^2 + n ] / 2 ），空间上，由于使用了 1 + 2 + ... + n-1 = n * (n - 1) / 2 = [ n^2 - n ] / 2，所以额外空间复杂度为 O（ [ n^2 - n ] / 2 ）。

```java
class Solution {
    public List<Integer> getRow(int n) {   
        List<Integer> pre = new ArrayList<>();
        pre.add(1);

        // 第1行
        if(n == 0) {
            return pre;
        }

        // 从第1行开始, 存在n列
        List<Integer> cur;
        for(int i = 1; i <= n; i++) {
            cur = new ArrayList<>();

            // 从第0列开始, 第i行存在i+1列
            for(int j = 0; j < i + 1; j++) {
                // 每行的第0列和最后一列, 规定为1
                if(j == 0 || j == i) {
                    cur.add(1);
                }
                // 每行的中间列: (i, j) = (i-1, j-1) + (i-1, j)
                else {
                    cur.add(pre.get(j - 1) + pre.get(j));
                }
            }

            pre = cur;
        }
        
        return pre;  
    }
}
```

##### 2）公式递推 + 线性 dp + 空间压缩 | O（ [ n^2 + n ] / 2 ）

- **思路**：在 1 的基础上，研究表依赖关系可知道，当前值依赖于上一行的前一个值以及当前值，所以右往左进行转移，从而把 pre 进一步压缩。
- **结论**：时间，1 ms，77.25%，空间，39.4 mb，23.19%，时间上，由于第 1 行有 1 个、第 2 行有 2 个 ... 第 n 行有 n 个位置，即一共需要设置 1 + 2 + ... + n = (n + 1) * n / 2 = [ n^2 + n ] / 2 个位置，所以整体时间复杂度为 O（ [ n^2 + n ] / 2 ），空间上，由于最多需要 n + 1 个位置，所以额外空间复杂度为 O（n + 1）。

```java
class Solution {
    public List<Integer> getRow(int n) {   
        List<Integer> cur = new ArrayList<>();
        cur.add(1);

        // 从第1行开始, 存在n列
        for(int i = 1; i <= n; i++) {
            // 补充最后一个1
            cur.add(1);

            // 由于依赖前一行的j-1, 所以需要从右往左进行转移, 即从第i-1列到1列, 跳过每行的第0列和最后一列
            for(int j = i - 1; j > 0; j--) {
                // 每行的中间列: (i, j) = (i-1, j-1) + (i-1, j)
                cur.set(j, cur.get(j - 1) + cur.get(j));
            }
        }
        
        return cur;  
    }
}
```

##### 3）数学 + 公式推导 + 线性 dp | O（n）

![1660643773320](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660643773320.png)

![1660643673823](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660643673823.png)

- **思路**：推导上述公式可知，n 行的 m 值依赖于 m - 1 的值 * (n - m + 1) / m。
- **结论**：时间，0 ms，100%，空间，39 mb，76.69%，时间上，由于只需要遍历一遍 n，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public List<Integer> getRow(int n) {
        List<Integer> dp = new ArrayList<>();
        dp.add(1);
        
        // 从左到右转移, dp[m] = dp[m-1] * [ (n - m + 1) / m ]
        for(int m = 1; m <= n; m++) {
            // 整数相乘再除, 保证精度没丢失
            dp.add((int) divide(multiply(dp.get(m - 1), n - m + 1), m));
        }

        return dp;
    }

    // multiplier被乘数, factor乘数因子
    private long multiply(long multiplier, long factor) {
        return multiplier * factor;
    }

    // dividend被除数, divisor除数
    private double divide(double dividend, double divisor) {
        return dividend / divisor;
    }
}
```

#### 120. 三角形最小路径和 | 最短路径 | medium

##### 1）LCP | O（ [ n^2 + n ] / 2 + n ）

- **思路**：LCP 套路，见代码注释。
- **结论**：时间，3 ms，77.03%，空间，40.9 mb，75.67%，时间上，由于分别需要初始化 1、2、...、n = (n + 1) * n / 2 = [ n^2 + n ] / 2 个位置的 dp 值，以及最后需要遍历 n 次取出最后一行的最小值，花费 O（n），所以整体时间复杂度为 O（ [ n^2 + n ] / 2 + n ），空间上，由于使用了一张 n * n 长的二维表，所以额外空间复杂度为 O（n^2）。

```java
class Solution {
    public int minimumTotal(List<List<Integer>> list) {
        // 定义状态：dp[i][j]表示, 从(0,0)走到(i,j)经过的最小路径和
        int n = list.size();
        int[][] dp = new int[n][n];

        // 初始化状态：dp[0][0]表示, 从(0,0)走到(0,0)经过的最小路径和, 为list.get(0).get(0)
        dp[0][0] = list.get(0).get(0);

        // 状态转移：尝试从左上、正上走过来, 求出最小值
        // 从上到下、从左到右进行转移
        int preSize, curSize, cur;
        for(int i = 1; i < n; i++) {
            preSize = i;
            curSize = i + 1;

            for(int j = 0; j < curSize; j++) {
                cur = list.get(i).get(j);

                // 同时存在左上和正上
                if(j - 1 > -1 && j < preSize) {
                    dp[i][j] = Math.min(dp[i - 1][j - 1], dp[i - 1][j]) + cur;
                }
                // 只存在左上
                else if(j - 1 > -1) {
                    dp[i][j] = dp[i - 1][j - 1] + cur;
                }
                // 只存在正上
                else {
                    dp[i][j] = dp[i - 1][j] + cur;
                }
            }    
        }

        // 获取结果：对dp[n-1]一行, 求最小值
        int min = Integer.MAX_VALUE;
        for(int j = 0; j < n; j++) {
            min = Math.min(min, dp[n - 1][j]);
        }

        return min;
    }
}
```

##### 2）LCP + 空间压缩 | O（[ n^2 + n ] / 2 + n ）

- **思路**：经过研究发现，由于值只依赖于上一层的，左上和正上，即与当前行的值无关，所以可以省略掉行维度的数组，在初始化当前行的 dp 值时，认为 dp[j] 就是上一行的值。
- **结论**：时间，3 ms，77.03%，空间，41.5 mb，5.40%，时间上，由于分别需要初始化 1、2、...、n = (n + 1) * n / 2 = [ n^2 + n ] / 2 个位置的 dp 值，以及最后需要遍历 n 次取出最后一行的最小值，花费 O（n），所以整体时间复杂度为 O（ [ n^2 + n ] / 2 + n ），空间上，由于使用了一张 n 长的一维表，所以额外空间复杂度为 O（n）。

```java
class Solution {
    public int minimumTotal(List<List<Integer>> list) {
        // 定义状态：dp[i][j]表示, 从(0,0)走到(i,j)经过的最小路径和
        // 空间压缩：省略掉行维度的数组, 因此dp[j]表示从(上一行, 0)到(目前行, j)经过的最小路径和 
        int n = list.size();
        int[] dp = new int[n];

        // 初始化状态：dp[0]表示, 从(0,0)走到(0,0)经过的最小路径和, 为list.get(0).get(0)
        dp[0] = list.get(0).get(0);

        // 状态转移：尝试从左上、正上走过来, 求出最小值
        // 由于依赖左上和正上的值, 所以当前行需要从右往左进行转移
        int preSize, curSize, cur;
        for(int i = 1; i < n; i++) {
            preSize = i;
            curSize = i + 1;

            for(int j = curSize - 1; j > -1; j--) {
                cur = list.get(i).get(j);

                // 同时存在左上和正上
                if(j - 1 > -1 && j < preSize) {
                    dp[j] = Math.min(dp[j - 1], dp[j]) + cur;
                }
                // 只存在左上
                else if(j - 1 > -1) {
                    dp[j] = dp[j - 1] + cur;
                }
                // 只存在正上
                else {
                    dp[j] = dp[j] + cur;
                }
            }
        }

        // 获取结果：对当前行即最后一行, 求最小值
        int min = Integer.MAX_VALUE;
        for(int j = 0; j < n; j++) {
            min = Math.min(min, dp[j]);
        }

        return min;
    }
}
```

#### 121. 买卖股票的最佳时机 | 线性 dp | easy

##### 1）贪心 + 极值法 | O（n）

- **思路**：
  1. 从左到右遍历，每次找到极小值则记录起来，用于与后面的股票价格相比，得出的最大利润则是答案。
  2. 其中，如果遇到极小值是比之前的更小，此时算利润的话是为负数，肯定不是最大值，所以只更新掉替换之前极小值即可，无需计算利润。
- **结论**：时间，2ms，88.95%，空间，51mb，90.14%，时间上，由于只需要遍历 1 次 n，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int maxProfit(int[] prices) {
        if(prices == null || prices.length <= 1) {
            return 0;
        }

        int max = 0, minPrice = prices[0];
        for(int i = 1; i < prices.length; i++) {
            if(prices[i] == minPrice) {
                continue;
            } else if(prices[i] < minPrice) {
                minPrice = prices[i];
            } else {
                max = Math.max(max, prices[i] - minPrice);
            }
        }

        return max;
    }
}
```

##### 2）线性 dp | O（n）

- **思路**：见代码注释。
- **结论**：时间，4 ms，25.59%，空间，50.1 mb，98.91%，时间上，由于只需要遍历一次数组，所以时间复杂度为 O（n），空间上，由于使用了一张 2 * n 长的二维表，所以额外空间复杂度为 O（2 * n）。

```java
class Solution {
    public int maxProfit(int[] prices) {
        int n = prices.length;

        // 定义状态：dp[i][j]表示, 第j天进行i类型操作, 能得到的最大收入
        // 其中, i=1表示买入, i=0表示卖出
        int[][] dp = new int[2][n]; 

        // 初始化状态：dp[1][n-1]表示最后一天买入, dp[0][n-1]表示最后一天卖出
        dp[0][n - 1] = prices[n - 1];

        // 状态转移：第i天时, 买入={当天买入, 以后买入}, 卖出={当天卖出, 以后卖出}
        // 从右往左进行状态转移
        for(int i = n - 2; i > -1; i--) {
            // 买入
            dp[1][i] = Math.max(
                // 当天买入, 以后卖出
                -prices[i] + dp[0][i + 1],
                // 以后买入
                dp[1][i + 1]
            );

            // 卖出
            dp[0][i] = Math.max(
                // 当天卖出, 终止交易
                prices[i],
                // 以后卖出
                dp[0][i + 1]
            );
        }

        // 获取结果：dp[1][0]表示, 第0天时买入, 得到的最大收入
        return dp[1][0];
    }
}
```

##### 3）线性 dp + 空间压缩 | O（n）

- **思路**：见代码注释。
- **结论**：时间，2 ms，57.52%，空间，57.3 mb，68.41%，时间上，由于状态转移只需要遍历 1 次 n，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int maxProfit(int[] prices) {
        int n = prices.length;

        // 定义状态：
        // 1）buy表示, 买入能得到的收入
        // 2）sell表示, 卖出能得到的收入, 如果此前没有买入, 则认为是当天买入+卖出=0
        int buy, sell;

        // 初始化状态：第0天时, buy=-prices[0], sell=0
        buy = -prices[0];
        sell = 0;

        // 状态转移：第i天时,
        // 1）buy= max{当天买入的贷款, 以后再买入}
        // 2）sell=max{上一次买入的贷款 + 今天卖出得到的收入, 以后再卖出}
        for(int i = 1; i < n; i++) {
            // buy叠加0表示：之前买卖0次, 今天尝试买入
            buy = Math.max(-prices[i], buy);

            // sell叠加buy表示：
            // 1）之前买入, 今天尝试卖出
            // 2）之前没买入, 今天尝试卖出 => 相当于max{今天买入+卖出=0, 今天买入+以后卖出}, 做max不影响结果
            sell = Math.max(buy + prices[i], sell);
        }

        // 获取结果：最终sell表示, 卖出时得到的最大利润
        return sell;
    }
}
```

#### 122. 买卖股票的最佳时机 II | 线性 dp | medium

##### 1）贪心 + 极值法 | O（n）

- **思路**：只要当天比前一天的价格要高，那么就买入前一天的股票，并在当前天卖出。
- **局限**：这种做法对比现实，是拥有这超前的预知能力，买到了过去的股票，所以总能保持不亏，达到最大的利润。
- **结论**：时间，1ms，82.25%，空间，41.3mb，40.60%，时间上，由于只需要遍历一次数组，所以时间复杂度为 O（n），空间上，由于只是用了额外的几个有限变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int maxProfit(int[] prices) {
        if(prices == null || prices.length == 0) {
            return 0;
        }

        int profit = 0;
        for(int i = 1; i < prices.length; i++) {
            if(prices[i] > prices[i - 1]) {
                profit += prices[i] - prices[i - 1];
            }
        }

        return profit;
    }
}
```

##### 2）线性 dp | O（n）

- **思路**：见代码注释。
- **结论**：时间，2 ms，34.01%，空间，41.1 mb，87.81%，时间上，由于只需要遍历一次数组，所以时间复杂度为 O（n），空间上，由于使用了一张 2 * n 长的二维表，所以额外空间复杂度为 O（2 * n）。

```java
class Solution {
    public int maxProfit(int[] prices) {
        int n = prices.length;

        // 定义状态：dp[i][j]表示, 第j天进行i类型操作, 产生的最大收入
        // 其中, i=1代表买, i=0代表卖
        int[][] dp = new int[2][n];

        // 初始化状态：dp[1][n-1]代表最后一天买, dp[0][n-1]代表最后一天卖
        dp[0][n - 1] = prices[n - 1];
         
        // 状态转移：第j天时, 买入=max{当天买入, 以后买入}, 卖出=max{当天卖出, 以后卖出}
        // 从右往左进行状态转移
        for(int i = n - 2; i > -1; i--) {
            // 买入
            dp[1][i] = Math.max(
                // 当天买入
                -prices[i] + dp[0][i + 1],
                // 以后买入
                dp[1][i + 1]
            );

            // 卖出
            dp[0][i] = Math.max(
                // 当天卖出
                prices[i] + dp[1][i + 1],
                // 以后卖出
                dp[0][i + 1]
            );
        }

        // 获取结果：dp[1][0]表示, 第0天进行买入能够得到的最大收入
        return dp[1][0];
    }
}
```

##### 3）线性 dp + 空间压缩 | O（n）

- **思路**：见代码注释。
- **结论**：时间，2 ms，34.01%，空间，41.2 mb，76.56%，时间上，由于状态转移只需要遍历 1 次 n，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int maxProfit(int[] prices) {
        int n = prices.length;

        // 定义状态：
        // 1）buy表示, 买入能得到的收入
        // 2）sell表示, 卖出能得到的收入, 如果此前没有买入, 则认为是当天买入+卖出=0
        int buy, sell;

        // 初始化状态：第0天时, buy=-prices[0], sell=0
        buy = -prices[0];
        sell = 0;

        // 状态转移：第i天时,
        // 1）buy= max{之前买卖的利润 + 当天买入的贷款, 以后再买入}
        // 2）sell=max{之前买卖的利润 + 上一次买入的贷款 + 今天卖出得到的收入, 以后再卖出}
        for(int i = 1; i < n; i++) {
            // buy叠加sell表示：
            // 1）之前买卖0次, 今天尝试买入
            // 2）之前买卖1次, 今天尝试买入 => 相当于max{之前买卖+今天买入, 之前买卖+以后买入}, 做max不影响结果 
            buy = Math.max(sell - prices[i], buy);

            // sell叠加buy表示：
            // 1）之前买入, 今天尝试卖出
            // 2）之前没买入, 今天尝试卖出 => 相当于max{今天买入+卖出=0, 今天买入+以后卖出}, 做max不影响结果
            sell = Math.max(buy + prices[i], sell);
        }

        // 获取结果：最终sell表示, 卖出时得到的最大利润
        return sell;
    }
}
```

#### 123. 买卖股票的最佳时机 III | 线性 dp | hard

##### 1）线性 dp | O（n）

- **思路**：见代码注释。
- **结论**：时间，4 ms，64.54%，空间，55.3 mb，58.82%，时间上，由于状态转移只需要遍历一次 n，所以时间复杂度为 O（n），空间上，由于使用了一张 2 * 2 * n 的三维表，所以时间复杂度为 O（4 * n）。

```java
class Solution {
    public int maxProfit(int[] prices) {
        int n = prices.length;

        // 定义状态：dp[i][j][k]表示, 在第k天进行第i次j类型操作, 产生的最大收入
        // 其中, i=0代表第1次, i=1代表第2次, j=1代表买, j=0代表卖
        int[][][] dp = new int[2][2][n];

        // 初始化状态：
        // dp[0][0]代表第1次卖、dp[0][1]代表第1次买、dp[1][0]代表第2次卖、dp[1][1]代表第2次买
        // dp[0][0][n-1]代表最后一天进行第1次卖、dp[0][1][n-1]代表最后一天进行第1次买
        // dp[1][0][n-1]代表最后一天进行第2次卖、dp[1][1][n-1]代表最后一天进行第2次买
        dp[0][0][n - 1] = dp[1][0][n - 1] = prices[n - 1];

        // 状态转移：第k天时, 进行第1次买、第2次买、第1次卖、第2次卖
        // 从下到上进行状态转移
        for(int k = n - 2; k > -1; k--) {
            // 第1次买入
            dp[0][1][k] = Math.max(
                // 当天买入, 以后第1次卖出
                -prices[k] + dp[0][0][k + 1],
                // 以后第1次买入
                dp[0][1][k + 1]
            );

            // 第2次买入
            dp[1][1][k] = Math.max(
                // 当天买入, 以后第2次卖出
                -prices[k] + dp[1][0][k + 1],
                // 以后第2次买入
                dp[1][1][k + 1]
            );

            // 第1次卖出
            dp[0][0][k] = Math.max(
                // 当天卖出, 以后第2次买入
                prices[k] + dp[1][1][k + 1],
                // 以后第1次卖出
                dp[0][0][k + 1]
            );

            // 第2次卖出
            dp[1][0][k] = Math.max(
                // 当天卖出, 以后不能交易了
                prices[k],
                // 以后第2次卖出
                dp[1][0][k + 1]
            );
        }

        // 获取结果：dp[0][1][0]代表, 在第0天进行第1次买入操作, 产生的最大收入
        return dp[0][1][0];
    }
}
```

##### 2）线性 dp + 空间压缩 | O（n）

- **思路**：见代码注释。
- **结论**：时间，1 ms，100%，空间，57.5 mb，33.59%，时间上，由于状态转移只需要遍历 1 次 n，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int maxProfit(int[] prices) {
        int n = prices.length;

        // 定义状态：
        // 1) buy1 表示, 第1次买入能得到的收入
        // 2) sell1表示, 第1次卖出能得到的收入, 如果此前没有买入, 则认为是当天买入+卖出=0
        // 3) buy2 表示, 第2次买入能得到的收入, 如果此前没有第1次, 则认为是当天买入+卖出+买入
        // 4) sell2表示, 第2次卖出能得到的收入, 如果此前没有第1次, 则认为是当天买入+卖出+买入+卖出=0
        int buy1, sell1, buy2, sell2;

        // 初始化状态：第0天时, buy1=buy2=-prices[0], sell1=sell2=0
        buy1 = buy2 = -prices[0];
        sell1 = sell2 = 0;

        // 状态转移：第i天时,
        // 1）buy1= max{当天买入的贷款, 以后再买入}
        // 2）sell1=max{之前买入的贷款 + 当天卖出得到的收入, 以后再卖出}
        // 3）buy2= max{之前买卖的利润 + 当天买入的贷款, 以后再买入}
        // 4）sell2=max{之前买卖的利润 + 上一次买入的贷款 + 当天卖出得到的收入, 以后再卖出}
        for(int i = 1; i < n; i++) {
            // buy1叠加0表示：之前买卖0次, 今天尝试买入
            buy1 = Math.max(-prices[i], buy1);

            // sell1叠加buy1表示：
            // 1）之前买入, 今天尝试卖出
            // 2）之前没买入, 今天尝试卖出 => 相当于max{今天买入+卖出=0, 今天买入+以后卖出}, 做max不影响结果
            sell1 = Math.max(buy1 + prices[i], sell1);

            // buy2叠加sell2表示：
            // 1）之前买卖1次, 今天尝试买入
            // 2）之前买卖0次, 今天尝试买入 => 相当于max{之前买卖+今天买入, 之前买卖+以后买入}, 做max不影响结果 
            buy2 = Math.max(sell1 - prices[i], buy2);

            // sell2叠加buy2表示：
            // 1）之前买卖+买入, 今天尝试卖出
            // 2）之前买卖+没买入, 今天尝试卖出 => 相当于max{之前买卖+今天买入+卖出=0, 之前买卖+今天买入+以后卖出}, 做max不影响结果
            sell2 = Math.max(buy2 + prices[i], sell2);
        }

        // 获取结果：取卖1次和卖2次的最大值, 但由于如果卖1次能达到最大, 那么卖2次也可以达到最大(相当于出现一次当天买+卖的情况), 所以直接取卖2次的值即可
        return sell2;
    }
}
```

#### 188. 买卖股票的最佳时机 IV | 线性 dp | hard

##### 1）线性 dp | O（in + n * in）

- **思路**：见代码注释。
- **结论**：时间，1 ms，99.30%，空间，41.4 mb，29.42%，时间上，由于初始化状态花费 O（in），状态转移花费 O（n * in），所以整体时间复杂度为 O（in + n * in），空间上，由于使用了一张 in * 2 * n 的三维表，所以额外空间复杂度为 O（in * 2 * n）。

```java
class Solution {
    public int maxProfit(int in, int[] prices) {
        int n = prices.length;
        if(in == 0 || n == 0) {
            return 0;
        }

        // 定义状态：dp[i][j][k]表示, 在第k天进行第i次j类型操作, 产生的最大收入
        // 其中, i=0代表第1次, i=1代表第2次, j=1代表买, j=0代表卖
        int[][][] dp = new int[in][2][n];

        // 初始化状态：
        // dp[0][0]代表第1次卖、dp[0][1]代表第1次买、dp[in-1][0]代表最后一次卖、dp[n-1][1]代表最后一次买
        // dp[i][0][n-1]代表最后一天进行第i次卖
        for(int i = 0; i < in; i++) {
            dp[i][0][n - 1] = prices[n - 1];
        }
        
        // 状态转移：第k天时, 进行第i次买、第i次买
        // 从下到上进行状态转移
        for(int k = n - 2; k > -1; k--) {
            // 从右到左进行状态转移
            for(int i = 0; i < in; i++) {
                // 第i次买入
                dp[i][1][k] = Math.max(
                    // 当天买入, 以后第i次卖出
                    -prices[k] + dp[i][0][k + 1],
                    // 以后第i次买入
                    dp[i][1][k + 1]
                );

                // 第i次卖出
                dp[i][0][k] = Math.max(
                    // 当天卖出, 以后第i+1次买入
                    prices[k] + (i + 1 < in? dp[i + 1][1][k + 1] : 0),
                    // 以后第i次卖出
                    dp[i][0][k + 1]
                );
            }
        }

        // 获取结果：dp[0][1][0]代表, 在第0天进行第1次买入操作, 产生的最大收入
        return dp[0][1][0];
    }
}
```

##### 2）线性 dp + 空间压缩 | O（in + n * in）

- **思路**：见代码注释。
- **结论**：时间，1 ms，99.30%，空间，39.6 mb，78.36%，时间上，由于初始化状态花费 O（in），状态转移花费 O（n * in），所以整体时间复杂度为 O（in + n * in），空间上，由于使用了 2 张 in 的数组，所以额外空间复杂度为 O（2 * in）。

```java
class Solution {
    public int maxProfit(int in, int[] prices) {
        int n = prices.length;
        if(in == 0 || n == 0) {
            return 0;
        }
        
        // 定义状态：
        // 1) buys[i] 表示, 第i次买入能得到的收入, 如果第i-1次为0, 则认为现在是当天买入+卖出+买入
        // 2) sells[i]表示, 第i次卖出能得到的收入, 如果第i-1次为0, 则认为现在是当天买入+卖出+买入+卖出=0
        int[] buys = new int[in], sells = new int[in];

        // 初始化状态：第0天时, buys[i]=-prices[0], sells[i]=0
        for(int i = 0; i < in; i++) {
            buys[i] = -prices[0];
        }

        // 状态转移：第j天时,
        // 1）buy[i]= max{之前买卖的利润 + 当天买入的贷款, 以后再买入}
        // 2）sell[i]=max{之前买卖的利润 + 上一次买入的贷款 + 当天卖出得到的收入, 以后再卖出}
        for(int j = 1; j < n; j++) {
            // 第1次买入卖出
            // buys[0]叠加0表示：之前买卖0次, 今天尝试买入
            buys[0] = Math.max(-prices[j], buys[0]);

            // sell[0]叠加buy[0]表示：
            // 1）之前买入, 今天尝试卖出
            // 2）之前没买入, 今天尝试卖出 => 相当于max{今天买入+卖出=0, 今天买入+以后卖出}, 做max不影响结果
            sells[0] = Math.max(buys[0] + prices[j], sells[0]);

            // 第2~n次买入卖出
            for(int i = 1; i < in; i++) {
                // buy[i]叠加sell[i - 1]表示：
                // 1）之前买卖1次, 今天尝试买入
                // 2）之前买卖0次, 今天尝试买入 => 相当于max{之前买卖+今天买入, 之前买卖+以后买入}, 做max不影响结果 
                buys[i] = Math.max(sells[i - 1] - prices[j], buys[i]);

                // sell[i]叠加buy[i]表示：
                // 1）之前买卖+买入, 今天尝试卖出
                // 2）之前买卖+没买入, 今天尝试卖出 => 相当于max{之前买卖+今天买入+卖出=0, 之前买卖+今天买入+以后卖出}, 做max不影响结果
                sells[i] = Math.max(buys[i] + prices[j], sells[i]);
            }
        }

        // 获取结果：由于如果卖i-1次能达到最大, 那么卖i次也可以达到最大(相当于出现一次当天买+卖的情况), 所以直接取卖i次的值即可
        return sells[in - 1];
    }
}
```

#### 309. 最佳买卖股票时机含冷冻期 | 线性 dp | medium

##### 1）线性 dp | O（n）

- **思路**：见代码注释。
- **结论**：时间，0 ms，100.01%，空间，40 mb，15.45%，时间上，由于只需要遍历一次数组，所以时间复杂度为 O（n），空间上，由于使用了一张 2 * n 长的二维表，所以额外空间复杂度为 O（2 * n）。

```java
class Solution {
    public int maxProfit(int[] prices) {
        int n = prices.length;
        if(n == 1) {
            return 0;
        }

        // 定义状态：dp[i][j]表示, 第j天时进行i类型操作, 能得到的最大收入
        // 其中, i=1表示买入操作, i=0表示卖出操作
        int[][] dp = new int[2][n];

        // 初始化状态：
        // 1）dp[1][n-1]表示最后一天买入, dp[0][n-1]表示最后一天卖出
        // 2）dp[1][n-2]表示倒数第二天买入, dp[0][n-2]表示倒数第二天卖出, 
        dp[0][n - 1] = prices[n - 1];
        dp[1][n - 2] = Math.max(0, -prices[n - 2] + dp[0][n - 1]);
        dp[0][n - 2] = Math.max(prices[n - 2], dp[0][n - 1]);

        // 状态转移：第i天时, 买入={当天买入, 以后买入}, 卖出={当天卖出, 以后卖出}
        // 从右往左进行状态转移
        for(int i = n - 3; i > -1; i--) {
            // 买入
            dp[1][i] = Math.max(
                // 当天买入, 以后卖出 
                -prices[i] + dp[0][i + 1],
                // 以后买入
                dp[1][i + 1]
            );

            // 卖出
            dp[0][i] = Math.max(
                // 当天卖出, 后天或以后买入
                prices[i] + dp[1][i + 2],
                // 以后卖出
                dp[0][i + 1]
            );
        }

        // 获取结果：dp[1][0]表示, 第0天时买入, 得到的最大收入
        return dp[1][0];
    }
}
```

##### 2）线性 dp + 空间压缩 | O（n）

- **思路**：见代码注释。
- **结论**：时间，1 ms，100%，空间，57.5 mb，33.59%，时间上，由于状态转移只需要遍历 1 次 n，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int maxProfit(int[] prices) {
        int n = prices.length;

        // 定义状态：
        // 1）buy表示, 买入能得到的收入
        // 2）sell表示, 卖出能得到的收入, 如果此前没有买入, 则认为是当天买入+卖出=0
        // 3）unlock表示, 非冷冻期, 只有此时才能买入, 出现卖出后, unlock=卖出前的收入
        int buy, sell, unlock;

        // 初始化状态：第0天时, buy=-prices[0], sell=0, unlock=0
        buy = -prices[0];
        sell = unlock = 0;

        // 状态转移：第i天时, 
        // 1）buy= max{上次没卖出时的利润 +  + 当天买入的贷款, 以后再买入}
        // 2）sell=max{之前买卖的利润 + 上一次买入的贷款 + 今天卖出得到的收入, 以后再卖出}
        for(int i = 1; i < n; i++) {
            // buy叠加unlock表示：非冷冻期中买入股票, 能得到的最大收入
            buy = Math.max(unlock - prices[i], buy);

            // 由于当天可能卖出, 所以先备份下卖出前的利润, 以便用于后面分析, 由于现在没卖出, 导致后面买入的利润
            unlock = sell;

            // sell叠加buy表示：
            // 1）之前买入, 今天尝试卖出
            // 2）之前没买入, 今天尝试卖出 => 相当于max{今天买入+卖出=0, 今天买入+以后卖出}, 做max不影响结果
            sell = Math.max(buy + prices[i], sell);
        }

        // 获取结果：
        // 1）如果unlock是n-1被赋值, 说明n-1发生过卖出, 由于能卖则卖, 后面又没有价格影响, 所以肯定是sell大
        // 2）如果unlock是n-2被赋值, 说明n-2发生过卖出, 此时有可能n-2大于n-1的利润, 但由于n-1轮sell#max判断, 对n-2不卖的情况做了结算赋值给了sell, 所以最后sell肯定也是最大的
        // 因此, 直接返回sell即可, 不用比较unlock和sell的最大值
        return sell;
    }
}
```

#### 714. 买卖股票的最佳时机含手续费 | 线性 dp | medium

##### 1）线性 dp | O（n）

- **思路**：见代码注释。
- **结论**：时间，8 ms，51.90%，空间，48.7 mb，86.40%，时间上，由于只需要遍历一次数组，所以时间复杂度为 O（n），空间上，由于使用了一张 2 * n 长的二维表，所以额外空间复杂度为 O（2 * n）。

```java
class Solution {
    public int maxProfit(int[] prices, int fee) {
        int n = prices.length;

        // 定义状态：dp[i][j]表示, 第j天进行i类型操作, 产生的最大收入
        // 其中, i=1代表买, i=0代表卖
        int[][] dp = new int[2][n];

        // 初始化状态：dp[1][n-1]代表最后一天买, dp[0][n-1]代表最后一天卖, 但要扣去手续费
        dp[0][n - 1] = prices[n - 1] - fee;
         
        // 状态转移：第j天时, 买入=max{当天买入, 以后买入}, 卖出=max{当天卖出, 以后卖出}
        // 从右往左进行状态转移
        for(int i = n - 2; i > -1; i--) {
            // 买入
            dp[1][i] = Math.max(
                // 当天买入
                -prices[i] + dp[0][i + 1],
                // 以后买入
                dp[1][i + 1]
            );

            // 卖出
            dp[0][i] = Math.max(
                // 当天卖出, 但要扣取手续费
                prices[i] - fee + dp[1][i + 1],
                // 以后卖出
                dp[0][i + 1]
            );
        }

        // 获取结果：dp[1][0]表示, 第0天进行买入能够得到的最大收入
        return dp[1][0];
    }
}
```

##### 2）线性 dp + 空间压缩 | O（n）

- **思路**：见代码注释。
- **结论**：时间，4 ms，87.00%，空间，49.8 mb，20.11%，时间上，由于状态转移只需要遍历 1 次 n，所以时间复杂度为 O（n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int maxProfit(int[] prices, int fee) {
        int n = prices.length;

        // 定义状态：
        // 1）buy表示, 买入能得到的收入
        // 2）sell表示, 卖出能得到的收入, 如果此前没有买入, 则认为是当天买入+卖出=0
        int buy, sell;

        // 初始化状态：第0天时, buy=-prices[0], sell=0, 第一天没做任何卖出, 所以没有收入也不用扣手续费
        buy = -prices[0];
        sell = 0;

        // 状态转移：第i天时,
        // 1）buy= max{之前买卖的利润 + 当天买入的贷款, 以后再买入}
        // 2）sell=max{之前买卖的利润 + 上一次买入的贷款 + 今天卖出得到的收入 - 手续费, 以后再卖出}
        for(int i = 1; i < n; i++) {
            // buy叠加sell表示：
            // 1）之前买卖0次, 今天尝试买入
            // 2）之前买卖1次, 今天尝试买入 => 相当于max{之前买卖+今天买入, 之前买卖+以后买入}, 做max不影响结果 
            buy = Math.max(sell - prices[i], buy);

            // sell叠加buy表示：
            // 1）之前买入, 今天尝试卖出
            // 2）之前没买入, 今天尝试卖出 => 相当于max{今天买入+卖出=0, 今天买入+以后卖出}, 做max不影响结果
            sell = Math.max(buy + prices[i] - fee, sell);
        }

        // 获取结果：最终sell表示, 卖出时得到的最大利润
        return sell;
    }
}
```

#### 139. 单词拆分 | 完全背包 + 组合| medium

##### 1）暴力递归 | O（n^2）

- **思路**：
  1. 设计一个 f（end）函数，代表 [0...end] 字符串能否从map中拼接出来。
  2. f（end）的实现通过遍历每个 end,end-1,end-2... 字符是否在字典中 + 再往前的剩余字符串能否被拼接出来，来得到当前结果。
- **结论**：执行超时，由于没作任何处理，导致出现了很多重复的递归，浪费了计算时间。

```java
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        if(s == null || s.length() == 0) {
            return true;
        }
        if(wordDict == null || wordDict.isEmpty()) {
            return false;
        }
        
        Map<String, Integer> map = new HashMap<>();
        for(int i = 0; i < wordDict.size(); i++) {
            map.put(wordDict.get(i), i);
        }

        return f(s, map, s.length()-1);
    }

    // f代表[0...end]字符串能否从map中拼接出来
    private boolean f(String s, Map<String, Integer> map, int end) {
        if(end < 0) {
            return false;
        }

        // 先判断以自身为字符串是否存在map中
        if(map.containsKey(s.substring(0, end+1))) {
            return true;
        }

        // 再判断当前字符+剩余字符串
        for(int i = end; i > -1; i--) {
            if(map.containsKey(s.substring(i, end+1)) && f(s, map, i-1)) {
                return true;
            }
        }

        return false;
    }
}
```

##### 2）记忆化搜索 | O（n）

- **思路**：沿用暴力递归的思路，增加了 dp 一维数组缓存，如果在缓存中存在则从获取中获取，否则先设置结果到缓存中再返回。
- **结论**：时间，6ms，73.83%，空间，38.5mb，62.13%，效率提升了不少，看还能不能用动态规划来优化下。

```java
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        if(s == null || s.length() == 0) {
            return true;
        }
        if(wordDict == null || wordDict.isEmpty()) {
            return false;
        }
        
        Map<String, Integer> map = new HashMap<>();
        for(int i = 0; i < wordDict.size(); i++) {
            map.put(wordDict.get(i), i);
        }

        Boolean[] dp = new Boolean[s.length()];
        return f(s, map, dp, s.length()-1);
    }

    // f代表[0...end]字符串能否从map中拼接出来
    private boolean f(String s, Map<String, Integer> map, Boolean[] dp, int end) {
        if(end < 0) {
            return false;
        }
        if(dp[end] != null) {
            return dp[end];
        }

        // 先判断以自身为字符串是否存在map中
        if(map.containsKey(s.substring(0, end+1))) {
            dp[end] = true;
            return dp[end];
        }

        // 再判断当前字符+剩余字符串
        for(int i = end; i > -1; i--) {
            if(map.containsKey(s.substring(i, end+1)) && f(s, map, dp, i-1)) {
                dp[end] = true;
                return dp[end];
            }
        }

        dp[end] = false;
        return dp[end];
    }
}
```

##### 3）严格表结构优化 | O（n）

- **思路**：通过把记忆化搜索的递归行为，优化成从 dp 数组取数的动态规划，免去了递归的调用。
- **结论**：
  1. 时间，4ms，81.34%，空间，38.3%，73.40%，效率提升很多了，就这样吧~
  2. 同时还发现，如果要想严格表结构改的比记忆化搜索的效率还好，那么就要尽量把逻辑卸载暴力递归中的递归函数中，这样才能把递归函数调用优化成从 dp 表中取数的动态规划，不然效率好像提升不大，甚至还降低！

```java
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        if(s == null || s.length() == 0) {
            return true;
        }
        if(wordDict == null || wordDict.isEmpty()) {
            return false;
        }
        
        Map<String, Integer> map = new HashMap<>();
        for(int i = 0; i < wordDict.size(); i++) {
            map.put(wordDict.get(i), i);
        }

        boolean[] dp = new boolean[s.length()];
        dp[0] = map.containsKey(s.substring(0, 1));

        int end = 1;
        while(end < s.length()) {
            // 先判断以自身为字符串是否存在map中
            if(map.containsKey(s.substring(0, end+1))) {
                dp[end] = true;
            }
            else {
                // 再判断当前字符+剩余字符串
                for(int i = end; i > -1; i--) {
                    if(i == 0) {
                        dp[end] = map.containsKey(s.substring(i, end+1));
                         break;
                    }
                    else {
                        if(map.containsKey(s.substring(i, end+1)) && dp[i-1]) {
                            dp[end] = true;
                            break;
                        }
                    }
                }
            }

            end++;
        }

        return dp[s.length()-1];
    }
}
```

#### 152. 乘积最大子数组 | 线性 dp | medium

##### 1）暴力递归 | O（n^2）

- **思路**：
  1. 看到子数组就想到以 end 结尾子数组，不过这题求的是最大乘积，与最大和不同的是，乘积有正负之分，还需要研究当前位的正负状态。
  2. 如果当前位是正数，那么最大乘积应该等于 max{当前值，当前值 * 前面的最大值}，其中这个最大值可能是最小的负数和最大的正数。
  3. 如果当前位是负数，那么最大乘积应该等于 max{当前值，当前值 * 前面的最小值}，其中这个最小值可能是最大的负数和最小的正数。
  4. 最后循环遍历每个下标，对比每个以 end 结尾的子数组的最大乘积，取最大的一个并返回。
- **结论**：执行超时，由于循环遍历调用递归时，执行了很多重复的递归操作，所以还需要继续优化~

```java
class Solution {

    // 子数组的最大乘积与最小乘积
    class Finfo {
        int max;
        int min;

        Finfo(int max, int min) {
            this.max = max;
            this.min = min;
        }
    }

    public int maxProduct(int[] nums) {
        if(nums == null || nums.length == 0) {
            return Integer.MIN_VALUE;
        }

        int max = Integer.MIN_VALUE;
        for(int i = 0; i < nums.length; i++) {
            max = Math.max(max, f(nums, i).max);
        }

        return max;
    }

    // f代表以end结尾的子数组的最大乘积与最小乘积
    private Finfo f(int[] nums, int end) {
        if(end == 0) {
            return new Finfo(nums[end], nums[end]);
        }

        Finfo finfo = f(nums, end-1);
        if(nums[end] < 0) {
            return new Finfo(
                Math.max(nums[end], nums[end] * finfo.min), 
                Math.min(nums[end], nums[end] * finfo.max)
            );
        } else {
            return new Finfo(
                Math.max(nums[end], nums[end] * finfo.max), 
                Math.min(nums[end], nums[end] * finfo.min)
            );
        }
    }
}
```

##### 2）记忆化搜索 | O（n）

- **思路**：沿用暴力递归的思路，增加了 dp 一维表缓存，如果缓存中存在，则直接从缓存中获取，否则则把结果先存入缓存再返回。
- **结论**：时间，1ms，97%，空间，39.7mb，5.15%，效率非常高了，看优化成动态规划的形式有没有性能上的提升~

```java
class Solution {

    // 子数组的最大乘积与最小乘积
    class Finfo {
        int max;
        int min;

        Finfo(int max, int min) {
            this.max = max;
            this.min = min;
        }
    }

    private Finfo[] dp;

    public int maxProduct(int[] nums) {
        if(nums == null || nums.length == 0) {
            return Integer.MIN_VALUE;
        }

        dp = new Finfo[nums.length];

        int max = Integer.MIN_VALUE;
        for(int i = 0; i < nums.length; i++) {
            max = Math.max(max, f(nums, i).max);
        }

        return max;
    }

    // f代表以end结尾的子数组的最大乘积与最小乘积
    private Finfo f(int[] nums, int end) {
        if(end == 0) {
            dp[end] = new Finfo(nums[end], nums[end]);
            return dp[end];
        }
        if(dp[end] != null) {
            return dp[end];
        }

        Finfo finfo = f(nums, end-1);
        if(nums[end] < 0) {
            dp[end] = new Finfo(
                Math.max(nums[end], nums[end] * finfo.min), 
                Math.min(nums[end], nums[end] * finfo.max)
            );
            return dp[end];
        } else {
            dp[end] = new Finfo(
                Math.max(nums[end], nums[end] * finfo.max), 
                Math.min(nums[end], nums[end] * finfo.min)
            );
            return dp[end];
        }
    }
}
```

##### 3）严格表结构优化 | O（n）

- **思路**：在记忆化搜索的基础上，研究了值关系依赖，发现是后面的值依赖前面的值，所以这是一个从左到右的模型，则先初始化 dp[0]，然后从左到右填充 dp 数组，以替代过多的递归行为。
- **结论**：
  1. 时间，1ms，97%，空间，39.7mb，5.15%，可见优化的效果并不明显，就这样吧~
  2. 不过有了新的心得，凡是记忆化搜索改动态规划，无论主函数怎么调的递归，都一定要**先填充好 dp 数组**，再用主函数的调用方法去调，这样是为了分离两个遍历，控制时间复杂度与记忆化搜索一致，不然很可能搞出比记忆化搜索大的时间复杂度，那样就得不偿失了~

```java
class Solution {

    // 子数组的最大乘积与最小乘积
    class Finfo {
        int max;
        int min;

        Finfo(int max, int min) {
            this.max = max;
            this.min = min;
        }
    }

    public int maxProduct(int[] nums) {
        if(nums == null || nums.length == 0) {
            return Integer.MIN_VALUE;
        }

        Finfo[] dp = new Finfo[nums.length];
        dp[0] = new Finfo(nums[0], nums[0]);
        for(int end = 1; end < nums.length; end++) {
            if(nums[end] < 0) {
                dp[end] = new Finfo(
                    Math.max(nums[end], nums[end] * dp[end-1].min), 
                    Math.min(nums[end], nums[end] * dp[end-1].max)
                );
            } else {
                dp[end] = new Finfo(
                    Math.max(nums[end], nums[end] * dp[end-1].max), 
                    Math.min(nums[end], nums[end] * dp[end-1].min)
                );
            }
        }

        int max = Integer.MIN_VALUE;
        for(int end = 0; end < nums.length; end++) {
            max = Math.max(max, dp[end].max);
        }

        return max;
    }
}
```

#### 198. 打家劫舍 | 线性 dp | medium

##### 1）暴力递归 | O（n^2）

- **思路**：
  1. 设计一个 f（end）函数，代表必须以 end 结尾的方案，最多能偷多少钱。
  2. 其中，f 函数的实现分为 3 种情况：
     1. 不偷当前这家的钱，看偷前 1 家的最大结果。
     2. 只投当前这家的钱。
     3. 既偷当前这家的钱，又继续看偷前前 1 家的最大结果。
  3. 三种情况取最大值返回即可作为当前 f（end）函数的返回值。
  4. 而最终的最大结果，还要从 0,1,2...length-1 不断尝试，比较出最大结果并返回。
- **结论**：执行超时，由于存在超多重复的递归，所以还需要继续优化~

```java
class Solution {
    public int rob(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        int max = 0;
        for(int i = 0; i < nums.length; i++) {
            max = Math.max(max, f(nums, i));
        }

        return max;
    }
    
    // f代表偷取以end为结尾最多能偷多少钱
    private int f(int[] nums, int end) {
        if(end < 0) {
            return 0;
        }
        if(end == 0) {
            return nums[0];
        }
        if(end == 1) {
            return Math.max(nums[0], nums[1]);
        }

        return Math.max(
            // 不偷这家, 看偷前1家的最大值
            f(nums, end-1),
            Math.max(
                // 只偷这家
                nums[end],
                // 偷这家+看偷前前1家的最大值 
                nums[end] + f(nums, end-2)
            )
        );
    }
}
```

##### 2）记忆化搜索 | O（2n）

- **思路**：沿用暴力递归的思路，增加了 dp 一维表缓存，如果缓存中存在则直接从缓存中获取，否则先把结果设置到缓存中再返回。
- **结论**：时间，0ms，100%，空间，36mb，5.50%，由于存在很多多余的递归，占据了很多压栈的空间，所以还需要优化成动态规划，以减少额外空间复杂度。

```java
class Solution {

    private int[] dp;

    public int rob(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        dp = new int[nums.length];
        Arrays.fill(dp, -1);

        int max = 0;
        for(int i = 0; i < nums.length; i++) {
            max = Math.max(max, f(nums, i));
        }

        return max;
    }
    
    // f代表偷取以end为结尾最多能偷多少钱
    private int f(int[] nums, int end) {
        if(end < 0) {
            return 0;
        }
        if(dp[end] != -1) {
            return dp[end];
        }
        if(end == 0) {
            dp[0] = nums[0];
            return dp[0];
        }
        if(end == 1) {
            dp[1] = Math.max(nums[0], nums[1]);
            return dp[1];
        }

        dp[end] = Math.max(
            // 不偷这家, 看偷前1家的最大值
            f(nums, end-1),
            Math.max(
                // 只偷这家
                nums[end],
                // 偷这家+看偷前前1家的最大值 
                nums[end] + f(nums, end-2)
            )
        );
        
        return dp[end];
    }
}
```

##### 3）严格表结构优化 | O（2n）

- **思路**：在记忆化搜索的基础上，经过研究值依赖关系，可知，一个值依赖于它的前一个和前前一个，所以这是一个从左到右的尝试模型，需要先初始化好 dp[0] 和 dp[1]，然后从左到右去给 dp 数组赋值即可。
- **结论**：时间，0ms，100%，空间，36.1mb，5.50%，经过动态规划把递归优化成了 dp 数组取值，优化掉了递归的压栈空间，但应该是测试用例不多，所以优化效果并不明显~

```java
class Solution {

    public int rob(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }
        if(nums.length == 1) {
            return nums[0];
        }
        
        int[] dp = new int[nums.length];
        dp[0] = nums[0];
        dp[1] = Math.max(nums[0], nums[1]);
        for(int end = 2; end < nums.length; end++) {
            dp[end] = Math.max(
                // 不偷这家, 看偷前1家的最大值
                dp[end-1],
                Math.max(
                    // 只偷这家
                    nums[end],
                    // 偷这家+看偷前前1家的最大值 
                    nums[end] + dp[end-2]
                )
            );
        }

        int max = 0;
        for(int end = 0; end < nums.length; end++) {
            max = Math.max(max, dp[end]);
        }

        return max;
    }
}
```

#### 279. 完全平方数 | 完全背包 + 最值 | medium

##### 1）暴力递归 | O（[√n] ^ m）

- **思路**：
  1. 设计一个 f（n）函数，代表获取数字 n 完全平方数的最少个数。
  2. 其中，f（n）函数通过枚举 0,1,2...√n，然后再递归调用 f（n - i）得到子过程完全平方数的最少个数，再 +1 得到当前尝试 i 完全平方数的最少个数。
  3. 当前所有尝试 i 都尝试完毕后，取最小的尝试结果即是当前 n 完全平方数的最少个数。 
- **结论**：执行超时，由于 f（n）时需要遍历 √n 次，每次遍历又需要分别调用子过程 f（n-i^2），子过程又要遍历 √n-i^2 次...，记 n-i^2 = a,b,c...,z，那么每次遍历时间花费平均为 √a * √b... * √z，因此时间复杂度为 O（√n * √a * √b * ... * √z） = O（[√n] ^ m），空间上，每次的递归深度记为 m，所以额外空间复杂度为 O（m * √n）。

```java
class Solution {
    public int numSquares(int n) {
        return f(n);
    }
    
    private int f(int n) {
        if(n <= 1) {
            return 1;
        }
        if(n == 2) {
            return 2;
        }

        int min = Integer.MAX_VALUE;
        for(int i = 1; (i * i) <= n; i++) {
            if(n-(i*i) > 0) {
                min = Math.min(min, 1 + f(n-(i*i)));
            } else {
                min = 1;
            }
        }

        return min;
    }
}
```

##### 2）记忆化搜索 | O（m * √n）

- **思路**：基于暴力递归的思路，增加了 dp 一维表缓存，如果缓存中存在则从缓存中获取，否则先设置结果到缓存中再返回。
- **结论**：时间，66ms，15.82%，41.1mb，5.25%，时间上，记需要遍历 √n 次，平均每次遍历需要尝试 m 个完全平方数，因此时间复杂度为 O（m * √n），空间上，每次的递归深度记为 m，所以额外空间复杂度为 O（m * √n）。

```java
class Solution {

    private int[] dp;

    public int numSquares(int n) {
        if(n <= 1) {
            return 1;
        }
        if(n == 2) {
            return 2;
        }

        dp = new int[n + 1];
        Arrays.fill(dp, -1);
        dp[0] = dp[1] = 1;
        dp[2] = 2;

        return f(n);
    }

    private int f(int n) {
        if(dp[n] != -1) {
            return dp[n];
        }

        int min = Integer.MAX_VALUE;
        for(int i = 1; (i * i) <= n; i++) {
            if(n-(i*i) > 0) {
                min = Math.min(min, 1 + f(n-(i*i)));
            } else {
                min = 1;
            }
        }

        dp[n] = min;
        return dp[n];
    }
}
```

##### 3）严格表结构优化 | O（n * √n）

- **思路**：在记忆化搜索的基础上，经过研究值依赖关系，发现是后面一个值依赖前面每个枚举值，所以可以枚举初始化好 0,1,...n-1，然后获取 n 的值返回即可。
- **结论**：
  1. 时间，23ms，80.99%，空间，40.5mb，7.83%，时间上，由于需要枚举前面 n-1 个值，每次枚举又需要遍历 √n 次，所以时间复杂度为 O（n * √n），空间上，由于使用了一个 n+1 长度的 dp 数组，所以额外空间复杂度为 O（n+1）。
  2. 本题得到一新的心得，如果记忆化搜索改动态规划时，发现递归函数中有枚举行为，则可以先模拟出递归深度有多少、f（n）一共依赖多少个前面的值，然后遍历这些值，再把递归函数中的逻辑拷到遍历里面即可。

```java
class Solution {

    public int numSquares(int n) {
        if(n <= 1) {
            return 1;
        }
        if(n == 2) {
            return 2;
        }

        int[] dp = new int[n + 1];
        dp[0] = dp[1] = 1;
        dp[2] = 2;

        for(int j = 3; j <= n; j++) {
            int min = Integer.MAX_VALUE;
            for(int i = 1; (i * i) <= j; i++) {
                if(j-(i*i) > 0) {
                    min = Math.min(min, 1 + dp[j-(i*i)]);
                } else {
                    min = 1;
                }
            }
            dp[j] = min;
        }

        return dp[n];
    }
}
```

##### 4）四平方和定理推论 | O（√n）

- **思路**：
  1. 四平方和定理：任意一个正整数，都可以被表示为至多 4 个整数的平方和。
  2. 四平方和定理推论：当前仅当 n = 4^k * （8m + 7）时，n 只能被表示为 4 个正整数的平方和，否则至多表示为 3 个。
     1. 当答案为 1 时，则 n 需要是完美的平方数，即 n = a^2。
     2. 当答案为 2 时，则 n 需要为平方和，即 n = a^2 + b^2。
     3. 当答案为 3 时，n 为其他情况，可排除其他情况后返回。
- **结论**：时间，0ms，100%，空间，38.8mb，11.54%，时间上，在最坏的情况下，当答案为 3 时，需要遍历完答案 2 的所有情况，所以时间复杂度为 O（√n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int numSquares(int n) {
        if(isPerfectSquares(n)) {
            return 1;
        }
        if(is4k8m7(n)) {
            return 4;
        }

        // 既不是完美平方数, 又不是4^k * (8m + 7), 则枚举判断是否为n=a^2+b^2
        int j;
        for(int i = 1; (i * i) <= n; i++) {
            j = n - i * i;
            if(isPerfectSquares(j)) {
                return 2;
            }
        }

        // 如果还不是n=a^2+b^2, 则只能是剩下的情况了
        return 3;
    }

    // 判断n是否为完美平方数: n=a^2
    private boolean isPerfectSquares(int n) {
        int a = (int) Math.sqrt(n);
        return n == a * a;
    }

    // 判断n是否等于4^k * (8m + 7) <= 四平方和定理推论
    private boolean is4k8m7(int n) {
        while(n % 4 == 0) {
            n /= 4;
        }
        return n % 8 == 7; 
    }
}
```

#### 300. 最长递增子序列 | 线性 dp | medium

##### 1）暴力递归 | O（n^2）

- **思路**：
  1. 设计一个 f（end）函数，代表获取以 end 结尾的最大递增子序列的长度。
  2. f（end）函数的实现通过当前结尾与枚举前面的最大子序列（即子过程 f（end-i））结合，得出当前的最大递增子序列的长度。
  3. 然后，由于 nums 数组中的最大递增子序列，可能没落在以最后一个字符结尾的子序列中，所以主函数还要枚举各个字符结尾的子序列，最后得出最大的答案并返回即可。
- **结论**：执行超时，时间上，由于每个位置可能被访问 n 次，一共有 n 个位置，所以时间复杂度为 O（n^2），空间上，额外空间复杂度取决于递归的最大深度 n,n-1,...,1 层，所以额外空间复杂度为 O（n^2）。

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        // 尝试选择以每个位置作为结尾的递增子序列
        int max = 0;
        for(int i = 0; i < nums.length; i++) {
            max = Math.max(max, f(nums, i));
        }

        return max;
    }

    // f代表获取以end结尾的最长递增子序列长度
    private int f(int[] nums, int end) {
        if(end < 0) {
            return 0;
        }
        if(end == 0) {
            return 1;
        }

        // 只选当前位置时为1
        int max = 1;
        
        // 尝试选择前面的递增子序列
        for(int i = 0; i < end; i++) {
            if(nums[end] > nums[i]) {
                max = Math.max(max, 1 + f(nums, i));
            }
        }

        return max;
    }
}
```

##### 2）记忆化搜索 | O（n^2）

- **思路**：在暴力递归的基础上，增加了 dp 一维表缓存，如果缓存中存在则从缓存中获取，否则先把结果设置到缓存中再返回。
- **结论**：时间，94ms，5.55%，空间，40.8mb，5.00%，时间上，每个元素只需要遍历 n 次，一共 n 个元素，所以空间复杂度为 O（n^2），空间上，额外空间复杂度取决于递归的最大深度 n,n-1,...,1 层，所以额外空间复杂度为 O（n^2）。

```java
class Solution {

    private int dp;

    public int lengthOfLIS(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        dp = new int[nums.length];
        Arrays.fill(dp, -1);

        // 尝试选择以每个位置作为结尾的递增子序列
        int max = 0;
        for(int i = 0; i < nums.length; i++) {
            max = Math.max(max, f(nums, i));
        }

        return max;
    }

    // f代表获取以end结尾的最长递增子序列长度
    private int f(int[] nums, int end) {
        if(end < 0) {
            return 0;
        }
        if(dp[end] != -1) {
            return dp[end];
        }
        if(end == 0) {
            dp[end] = 1;
            return dp[end];
        }

        // 只选当前位置时为1
        int max = 1;
        
        // 尝试选择前面的递增子序列
        for(int i = 0; i < end; i++) {
            if(nums[end] > nums[i]) {
                max = Math.max(max, 1 + f(nums, i));
            }
        }

        dp[end] = max;
        return dp[end];
    }
}
```

##### 3）严格表结构优化 | O（n^2）

- **思路**：在记忆化搜索的基础上，经过研究值依赖关系可知，每个值都依赖于前面的 n-1,n-2,...1,0 的值，有枚举的行为，因此需要先在主函数中枚举并初始化好每个 dp 数组的值，然后再比较出最大的递增子序列长度。
- **结论**：时间，55ms，71.50%，空间，40.8mb，5.00%，时间上，需要先循环遍历 2 次初始化好 dp 数组，然后在遍历一次 dp 数组，取得最大递增子序列长度，所以时间复杂度为 O（n^2 + n），空间上，由于使用了一个长度为 n 的 dp 一维数组，所以额外空间复杂度为 O（n）。

```java
class Solution {

    public int lengthOfLIS(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        int[] dp = new int[nums.length];
        dp[0] = 1;

        // f代表获取以end结尾的最长递增子序列长度
        int max;
        for(int end = 1; end < nums.length; end++) {
            // 只选当前位置时为1
            max = 1;

            // 尝试选择前面的递增子序列
            for(int i = 0; i < end; i++) {
                if(nums[end] > nums[i]) {
                    max = Math.max(max, 1 + dp[i]);
                }
            }

            dp[end] = max;
        }

        // 尝试选择以每个位置作为结尾的递增子序列
        max = 0;
        for(int i = 0; i < nums.length; i++) {
            max = Math.max(max, dp[i]);
        }

        return max;
    }
}
```

##### 4）二分查找与枚举优化 | O（n * logn）

- **思路**：在严格表结构的基础上，看能不能优化掉动态规划的枚举行为，其中，优化枚举行为的方式有两种，一是观察值依赖的枚举关系，把枚举行为优化成取值行为，从而把 O（i）的复杂度优化成 O（1）；二是由题目的特定条件决定，本题正是经过组合出单调性，通过二分查找来优化掉枚举行为，从而把 O（i）的复杂度优化成 O（logn）。
- **结论**：时间，2ms，99.70%，空间，41mb，5.00%，时间上，初始化 dp 数组需要花费 O（n * logn），遍历并判断 dp 数组最大值花费 O（n），所以时间复杂为 O（n * logn + n），空间上，由于使用了一个长度为 n 的 dp 一维数组，所以额外空间复杂度为 O（n）。

```java
class Solution {

    public int lengthOfLIS(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        int[] dp = new int[nums.length];
        dp[0] = 1;

        // 长度为i+1递增子序列的最小结尾: 从小到大排序
        int el = 0, er = 0;
        int[] ends = new int[nums.length];
        ends[0] = nums[0];

        // f代表获取以end结尾的最长递增子序列长度
        for(int end = 1; end < nums.length; end++) {
            dp[end] = processDp(ends, el, er, nums[end]);

            // 有效区+1
            if(dp[end] == er + 2) {
                er++;
            }
        }

        // 尝试选择以每个位置作为结尾的递增子序列
        int max = 0;
        for(int i = 0; i < nums.length; i++) {
            max = Math.max(max, dp[i]);
        }

        return max;
    }

    // 长度为i+1递增子序列的最小结尾: 从小到大排序
    private int processDp(int[] ends, int el, int er, int value) {
        // er+1必合法
        if(value > ends[er]) {
            ends[er+1] = value;
            return er + 2;
        } 
        // 二分查找最右的最小结尾
        else {
            int mid, resi = 0;
            while(el <= er) {
                mid = (el + er) >>> 1;
                
                // 右
                if(value > ends[mid]) {
                    el = mid + 1;
                } 
                // 左边, 也可能就为答案
                else if(value < ends[mid]) {
                    er = mid - 1;
                    resi = mid;
                } 
                // 就是当前位置
                else {
                    resi = mid;
                    break;
                }
            }

            ends[resi] = value;
            return resi + 1;
        }
    }
}
```

#### 312. 戳气球 | 区间 dp | hard

##### 1）暴力递归 | >= O（n^3）

- **思路**：
  1. 设计一个 f（l，r）函数，代表打爆 [l, r] 范围内的气球，所能带来的最大收益。
  2. 其中，f 函数通过尝试最后打爆第 0,1,...,n-1 个气球，分别计算各自的收益，来比较得到 [l, r] 范围内的最大收益。
- **结论**：执行超时，时间上，由于是二维参数， f 函数实现需要依赖 l+1 和 r-1， 所以需要花费 O（n^2）来铺满所有的二维参数，且每次递归还需要花费 O（n）来从头遍历 nums 数组，因此时间复杂度至少为 O（n^3），空间上，最大递归深度  O（n^2），同时还是使用了一个 help 辅助数组 O（n+2），所以额外空间复杂度为 O（n^2 + n+2）。

```java
class Solution {
    public int maxCoins(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        int n = nums.length;
        int[] help = new int[n+2];
        help[0] = help[n+1] = 1;
        for(int i = 1; i < n+1; i++) {
            help[i] = nums[i-1];
        }

        return f(help, 1, n);
    }

    // f代表[l,r]范围内的气球被打爆能带来的最大收益
    // 潜台词: 打爆[l,r]时, l-1和r+1肯定没有被打爆
    private int f(int[] help, int l, int r) {
        if(l > r) {
            return 0;
        }
        if(l == r) {
            return help[l-1] * help[l] * help[r+1];
        }

        int max = Math.max(
            // 尝试最后才打爆l上的气球: 之前打爆后面的分数 + 当前分数
            f(help, l+1, r) + help[l-1] * help[l] * help[r+1],
            // 尝试最后才打爆r上的气球: 之前打爆前面的分数 + 当前分数
            f(help, l, r-1) + help[l-1] * help[r] * help[r+1]
        );

        // 尝试最后才打爆中间位置上的气球: 之前打爆前面的分数 + 之前打爆后面的分数 + 当前分数 
        for(int i = l + 1; i < r; i++) {
            max = Math.max(
                max,
                f(help, l, i-1) + f(help, i+1, r) + help[l-1] * help[i] * help[r+1]
            );
        }

        return max;
    }
}
```

##### 2）记忆化搜索 | O（n^3）

- **思路**：在暴力递归的基础上，增加了一张 dp 二维表缓存数组，如果缓存中存在则先从缓存中获取，否则先设置结果到缓存中再返回。
- **结论**：时间，104ms，20%，空间，40.6mb，7%，时间上，由于是二维参数， f 函数实现需要依赖 l+1 和 r-1， 所以需要花费 O（n^2）来铺满所有的二维参数，且每次递归还需要花费 O（n）来从头遍历 nums 数组，同时因为有缓存数组的存在，因此时间复杂度为 O（n^3），空间上，二维表 dp 数组  O（n^2），同时还是使用了一个 help 辅助数组 O（n+2），所以额外空间复杂度为 O（n^2 + n+2）。

```java
class Solution {

    private int[][] dp;

    public int maxCoins(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        int n = nums.length;
        int[] help = new int[n+2];
        help[0] = help[n+1] = 1;
        for(int i = 1; i < n+1; i++) {
            help[i] = nums[i-1];
        }

        dp = new int[n+1][n+1];
        for(int i = 0; i < dp.length; i++) {
            Arrays.fill(dp[i], -1);
        }

        return f(help, 1, n);
    }

    // f代表[l,r]范围内的气球被打爆能带来的最大收益
    // 潜台词: 打爆[l,r]时, l-1和r+1肯定没有被打爆
    private int f(int[] help, int l, int r) {
        if(l > r) {
            return 0;
        }
        if(dp[l][r] != -1) {
            return dp[l][r];
        }
        if(l == r) {
            dp[l][r] = help[l-1] * help[l] * help[r+1];
            return dp[l][r];
        }

        int max = Math.max(
            // 尝试最后才打爆l上的气球: 之前打爆后面的分数 + 当前分数
            f(help, l+1, r) + help[l-1] * help[l] * help[r+1],
            // 尝试最后才打爆r上的气球: 之前打爆前面的分数 + 当前分数
            f(help, l, r-1) + help[l-1] * help[r] * help[r+1]
        );

        // 尝试最后才打爆中间位置上的气球: 之前打爆前面的分数 + 之前打爆后面的分数 + 当前分数 
        for(int i = l + 1; i < r; i++) {
            max = Math.max(
                max,
                f(help, l, i-1) + f(help, i+1, r) + help[l-1] * help[i] * help[r+1]
            );
        }

        dp[l][r] = max;
        return max;
    }
}
```

##### 3）严格表结构优化 | O（n^3）

- **思路**：在记忆化搜索的基础上，经过研究值依赖关系，发现一个值依赖它左边和下边的值，且左半部分是没用的部分，默认设置为 0，所以这是一个从下往上、从左往右初始化 dp 数组的模型。
- **结论**：时间，27ms，100%，空间，40.6mb，6.81%，时间上，时间复杂度等于初始化 dp 所花费的时间 O（n^3），空间上，使用了一张二维表 dp 数组以及一张 help 辅助数组，所以额外空间复杂度为 O（n^2 + n+2）。

```java
class Solution {

    public int maxCoins(int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        int n = nums.length;
        int[] help = new int[n+2];
        help[0] = help[n+1] = 1;
        for(int i = 1; i < n+1; i++) {
            help[i] = nums[i-1];
        }
        
        // 初始化对角线
        int[][] dp = new int[n+1][n+1];
        for(int i = 1; i < dp.length; i++) {
            dp[i][i] = help[i-1] * help[i] * help[i+1];
        }
        // 从下往上、从左往右进行初始化: 其中k范围取[1...n], j范围取左半部分
        int max;
        for(int k = dp.length - 2; k > 0; k--) {
            for(int j = k + 1; j < dp[k].length; j++) {
                max = Math.max(
                    // 尝试最后才打爆l上的气球: 之前打爆后面的分数 + 当前分数
                    dp[k+1][j] + help[k-1] * help[k] * help[j+1],
                    // 尝试最后才打爆r上的气球: 之前打爆前面的分数 + 当前分数
                    dp[k][j-1] + help[k-1] * help[j] * help[j+1]
                );

            // 尝试最后才打爆中间位置上的气球: 之前打爆前面的分数 + 之前打爆后面的分数 + 当前分数 
                for(int i = k + 1; i < j; i++) {
                    max = Math.max(
                        max,
                        dp[k][i-1] + dp[i+1][j] + help[k-1] * help[i] * help[j+1]
                    );
                }

                dp[k][j] = max;
            }
        }

        return dp[1][n];
    }
}
```

#### 322. 零钱兑换 | 完全背包 + 最值 | medium

##### 1）暴力递归1 | >= O（s^2 * n）

- **思路**：
  1. 设计一个 f（index，rest）函数，代表使用 [0, index] 处的硬币，来凑成 rest 金额的最少硬币数。
  2. f 函数的实现通过枚举 index 处的硬币使用 0,1,2,...n 个 + 剩余的金额使用子过程去枚举 index-1,index-2,..., 0 处的硬币，从而得到每种情况的最少硬币数，然后再比较得出最少的硬币数作为当前参数的最少硬币数。
- **结论**：执行超时，时间上，记金额为 s，数组长度为 n，最坏情况下，递归函数首先需要遍历 s 次，然后递归深度需要覆盖两个参数，需要遍历 s * n 二维矩阵，所以时间复杂度至少为 O（s^2 * n），空间上，额外空间复杂度取决于递归深度，为 O（s * n）。

```java
class Solution {

    public int coinChange(int[] coins, int amount) {
        if(coins == null || coins.length == 0) {
            return -1;
        }
        return f(coins, coins.length-1, amount);
    }

    // f代表使用[0, index]处的硬币, 凑成rest金额的最少硬币数
    private int f(int[] coins, int index, int rest) {
        int min = Integer.MAX_VALUE, count, cur;
        for(int i = 0; i * coins[index] <= rest; i++) {
            // 使用i个当前硬币
            cur = rest - i * coins[index];

            // 只使用当前硬币凑成rest
            if(cur == 0) {
                min = Math.min(min, i);
            }
            // 没凑完, 如果还有的凑, 则剩余的则交给前面的硬币去凑
            else if(index - 1 > -1) {
                count = f(coins, index - 1, cur);

                // 如果前面的凑不成, 则跳过; 如果凑成的, 则比较最小硬币数
                if(count != -1) {
                    min = Math.min(min, i + count);
                }
            }
        }

        // 如果没凑成, 则返回-1, 否则返回能凑成的最少硬币数
        return min == Integer.MAX_VALUE? -1 : min;
    }
}
```

##### 2）记忆化搜索1 | O（s^2 * n）

- **思路**：在暴力递归 1 的基础上，增加了一张 dp 二维表缓存，如果缓存中存在则从缓存中获取，否则先把结果设置到缓存中再返回。
- **结论**：执行超时，时间上，由于增加了缓存，缓存设置一次后将不再处理，所以时间复杂度等于暴力递归 1 的下限 O（s^2 * n），空间上，由于在暴力递归1 的基础上增加了一张 dp 二维表缓存，所以额外空间复杂度为 O（2s * n）。

```java
class Solution {

    private int[][] dp;

    public int coinChange(int[] coins, int amount) {
        if(coins == null || coins.length == 0) {
            return -1;
        }

        dp = new int[coins.length][amount+1];
        for(int i = 0; i < dp.length; i++) {
            Arrays.fill(dp[i], -1);
        }

        return f(coins, coins.length-1, amount);
    }

    // f代表使用[0, index]处的硬币, 凑成rest金额的最少硬币数
    private int f(int[] coins, int index, int rest) {
        if(dp[index][rest] != -1) {
            return dp[index][rest];
        }

        int min = Integer.MAX_VALUE, count, cur;
        for(int i = 0; i * coins[index] <= rest; i++) {
            // 使用i个当前硬币
            cur = rest - i * coins[index];

            // 只使用当前硬币凑成rest
            if(cur == 0) {
                min = Math.min(min, i);
            }
            // 没凑完, 如果还有的凑, 则剩余的则交给前面的硬币去凑
            else if(index - 1 > -1) {
                count = f(coins, index - 1, cur);

                // 如果前面的凑不成, 则跳过; 如果凑成的, 则比较最小硬币数
                if(count != -1) {
                    min = Math.min(min, i + count);
                }
            }
        }

        // 如果没凑成, 则返回-1, 否则返回能凑成的最少硬币数
        dp[index][rest] = min == Integer.MAX_VALUE? -1 : min;
        return dp[index][rest];
    }
}
```

##### 3）严格表结构优化1 | O（s^2 * n）

- **思路**：在记忆化搜索 1 的基础上，经过研究值依赖关系，发现一个值只依赖与它前面 -1 的值，所以这是从上往下初始化 dp 的模型，其中 index=0 处的初始化值，等于是否能够使用 0 处硬币凑成各种 rest 金额的数量，凑不成则为 -1。
- **结论**：时间，535ms，5.01%，空间，5.02%，时间上，时间复杂度取决于初始化 dp，需要花费 O（n * s * s），空间上，由于使用了一张 n*（s+1）的二维表，所以额外空间复杂度为 O（s * n）。

```java
class Solution {

    public int coinChange(int[] coins, int amount) {
        if(coins == null || coins.length == 0) {
            return -1;
        }
        if(amount == 0) {
            return 0;
        }

        int[][] dp = new int[coins.length][amount+1];
        for(int i = 0; i < dp[0].length; i++) {
            dp[0][i] = i % coins[0] == 0? i / coins[0] : -1;
        }

        int min, count, cur;
        for(int index = 1; index < dp.length; index++) {
            for(int rest = 0; rest < dp[0].length; rest++) {
                min = Integer.MAX_VALUE;
                for(int i = 0; i * coins[index] <= rest; i++) {
                    // 使用i个当前硬币
                    cur = rest - i * coins[index];

                    // 只使用当前硬币凑成rest
                    if(cur == 0) {
                        min = Math.min(min, i);
                    }
                    // 没凑完, 如果还有的凑, 则剩余的则交给前面的硬币去凑
                    else if(index - 1 > -1) {
                        count = dp[index-1][cur];

                        // 如果前面的凑不成, 则跳过; 如果凑成的, 则比较最小硬币数
                        if(count != -1) {
                            min = Math.min(min, i + count);
                        }
                    }
                }

                // 如果没凑成, 则返回-1, 否则返回能凑成的最少硬币数
                dp[index][rest] = min == Integer.MAX_VALUE? -1 : min;
            }
        }

        return dp[coins.length-1][amount];
    }
}
```

##### 4）暴力递归2 | >= O（s * n）

- **思路**：
  1. 由于暴力递归 1 思路的时间复杂度太高，所以需要考虑过其他思路。
  2. 设计一个 f（rest）函数，代表使用 coins 数组中所有的硬币，来凑成 rest 金额的最小硬币数。
  3. f 函数通过遍历每个硬币，一次递归中只使用 1 个 coin 硬币，而不是尝试 n 个，然后再把剩余的 rest 交由子过程去尝试所有硬币，同时也是每个子过程只尝试 1 个 coin硬币...周而复始，从而把两个参数降低为 1个参数的递归。
- **结论**：执行超时，时间上，由于每次递归要遍历 n 长的 coins 数组，然后最多要递归 s 次，所以时间复杂度为 O（s * n），空间上，额外空间复杂度取决于递归深度，为 O（s）。

```java
class Solution {

    public int coinChange(int[] coins, int amount) {
        if(coins == null || coins.length == 0) {
            return -1;
        }
        return f(coins, amount);
    }

    // f代表使用coins数组中所有的硬币, 凑成amount金额的最小硬币数
    private int f(int[] coins, int rest) {
        if(rest == 0) {
            return 0;
        }

        int min = Integer.MAX_VALUE, count, cur;
        for(int coin : coins) {        
            // 只使用1个coin就凑成了rest
            cur = rest - coin;
            if(cur == 0) {
                min = 1;
            }
            // 组合使用coin
            else if(cur > 0) {
                count = f(coins, cur);

                // 凑成rest, 则对比最小硬币数
                if(count != -1) {
                    min = Math.min(min, count+1);
                }
            }
        }

        return min == Integer.MAX_VALUE? -1 : min;
    }
}
```

##### 5）记忆化搜索2 | O（s * n）

- **思路**：在暴力递归2 的基础上，增加了 dp 一维表缓存，如果缓存中存在则从缓存中获取，否则先把结果设置到缓存中再返回。
- **结论**：执行超时，时间上，由于增加了缓存，同参数的递归只会处理 1 次，所以时间复杂度等于暴力递归 2 的时间复杂度，为 O（s * n），空间上，由于在暴力递归2 的基础上增加了一张 dp 一维表缓存数组，所以额外空间复杂度为 O（2s）。

```java
class Solution {

    private int[] dp;

    public int coinChange(int[] coins, int amount) {
        if(coins == null || coins.length == 0) {
            return -1;
        }

        dp = new int[amount+1];
        Arrays.fill(dp, -1);

        return f(coins, amount);
    }

    // f代表使用coins数组中所有的硬币, 凑成amount金额的最小硬币数
    private int f(int[] coins, int rest) {
        if(dp[rest] != -1) {
            return dp[rest];
        }
        if(rest == 0) {
            dp[rest] = 0;
            return dp[rest];
        }

        int min = Integer.MAX_VALUE, count, cur;
        for(int coin : coins) {        
            // 只使用1个coin就凑成了rest
            cur = rest - coin;
            if(cur == 0) {
                min = 1;
            }
            // 组合使用coin
            else if(cur > 0) {
                count = f(coins, cur);

                // 凑成rest, 则对比最小硬币数
                if(count != -1) {
                    min = Math.min(min, count+1);
                }
            }
        }

        dp[rest] = min == Integer.MAX_VALUE? -1 : min;
        return dp[rest];
    }
}
```

##### 6）严格表结构优化2 | O（S * n）

- **思路**：在记忆化搜索2 的基础上，经过研究值依赖关系，发现一个依赖它 rest-coin 的值，且没有负数的时候，也就是 rest 最小也只是 0 而已，所以只需初始化 dp[0]，然后从左到右初始化 dp 数组即可。
- **结论**：时间，13ms，56.77%，空间，41.1mb，5.02%，时间上，时间复杂度取决于初始化 dp 数组的花费 O（s * n），空间上，由于使用了一张 dp 一维表数组，所以额外空间复杂度为 O（s）。

```java
class Solution {
    
    public int coinChange(int[] coins, int amount) {
        if(coins == null || coins.length == 0) {
            return -1;
        }

        int[] dp = new int[amount+1];
        dp[0] = 0;
        
        int min, count, cur;
        for(int rest = 1; rest <= amount; rest++) {
            min = Integer.MAX_VALUE;
            for(int coin : coins) {        
                // 只使用1个coin就凑成了rest
                cur = rest - coin;
                if(cur == 0) {
                    min = 1;
                }
                // 组合使用coin
                else if(cur > 0) {
                    count = dp[cur];

                    // 凑成rest, 则对比最小硬币数
                    if(count != -1) {
                        min = Math.min(min, count+1);
                    }
                }
            }

            dp[rest] = min == Integer.MAX_VALUE? -1 : min;
        }

        return dp[amount];
    }
}
```

#### 416. 分割等和子集 | 01 背包 + 是否存在 | medium

##### 1）暴力递归 | O（n^2）

- **思路**：
  1. 递归函数调用前，先获取原数组总和，如果总和为奇数，那么肯定不能分割等和子集，直接返回 false 即可。
  2. 然后设计一个 f（start，cur）函数，代表 [start...结尾] 能否扣减 cur 为 0。
  3. f 函数实现分为 2 种情况：一是，扣减当前 start 上的值，此时需要把扣减后的和传给子过程继续判断；二是，不扣减当前 start 上的值，此时只需要把 cur 原封不动的传给子过程继续判断即可。
- **结论**：执行超时，时间上，由于扣和不扣当前值，调用了两次递归函数到数组末尾，深度最大为 n，组合起来一共有 n^2 种递归行为，所以时间复杂度为 O（n^2），空间上，额外空间复杂度取决于递归深度，为 O（n^2）。

```java
class Solution {
    public boolean canPartition(int[] nums) {
        if(nums == null || nums.length == 0) {
            return true;
        }

        // 校验总和是否为奇数
        int sum = 0;
        for(int i = 0; i < nums.length; i++) {
            sum += nums[i];
        }
        if(sum % 2 != 0) {
            return false;
        }

        return f(nums, 0, sum / 2);
    }

    // f代表判断[start...结尾]能否扣减cur为0
    private boolean f(int[] nums, int start, int cur) {
        if(cur < 0) {
            return false;
        }
        if(start == nums.length) {
            return cur == 0;
        }
        // 扣减当前值
        return f(nums, start+1, cur-nums[start]) || 
                // 不扣减当前值
               f(nums, start+1, cur);
    }
}
```

##### 2）记忆化搜索 | <= O（n^2）

- **思路**：在暴力递归的基础上，增加了 dp 二维表缓存，如果缓存中存在则从缓存中获取，否则先把结果设置到缓存中再返回。
- **结论**：时间，5ms，99.17%，54.6mb，5.02%，时间上，由于有缓存的存在，同样参数的递归行为只会处理一次，所以时间复杂度会比暴力递归的少，即 <= O（n^2），空间上，递归深度最大为 O（n），且还使用了一张 dp 二维表 O（n * t/2），所以额外空间复杂度为 O（n + n* t/2）。

```java
class Solution {

    private Boolean[][] dp;

    public boolean canPartition(int[] nums) {
        if(nums == null || nums.length == 0) {
            return true;
        }

        // 校验总和是否为奇数
        int sum = 0;
        for(int i = 0; i < nums.length; i++) {
            sum += nums[i];
        }
        if(sum % 2 != 0) {
            return false;
        }

        dp = new Boolean[nums.length + 1][sum/2 + 1];
        return f(nums, 0, sum / 2);
    }

    // f代表判断[start...结尾]能否扣减cur为0
    private boolean f(int[] nums, int start, int cur) {
        if(cur < 0) {
            return false;
        }
        if(start == nums.length) {
            return cur == 0;
        }
        if(dp[start][cur] != null) {
            return dp[start][cur];
        }

        // 扣减当前值
        dp[start][cur] = f(nums, start+1, cur-nums[start]) || 
                         // 不扣减当前值
                         f(nums, start+1, cur);

        return dp[start][cur];
    }
}
```

##### 3）严格表结构优化 | O（n * t/2）

- **思路**：在记忆化搜索的基础上，经过研究值依赖关系，发现一个值依赖它下方和左方的值，所以需要初始化好最左边的一列和最下边的一行，然后从下往上、从左往右构建 dp 数组。
- **结论**：时间，31ms，43.18%，42.2mb，11.25%，时间上，记忆化搜索中 cur - nums[start] 执行的都是关键的递归行为，而动态规划中，由于是从下往上、从左往右依次执行，执行了很多多余的参数，相对地就比记忆化搜索的更慢了，所以时间复杂度为 O（n * t/2），空间上，由于使用了一张 dp 二维表数组，所以额外空间复杂度为 O（n * t/2）。

```java
class Solution {

    public boolean canPartition(int[] nums) {
        if(nums == null || nums.length == 0) {
            return true;
        }

        // 校验总和是否为奇数
        int sum = 0;
        for(int i = 0; i < nums.length; i++) {
            sum += nums[i];
        }
        if(sum % 2 != 0) {
            return false;
        }

        // 初始化第一列为true
        boolean[][] dp = new boolean[nums.length + 1][sum/2 + 1];
        for(int i = 0; i < dp.length; i++) {
            dp[i][0] = true;
        }
        // 从下往上, 从左往右初始化
        int rest;
        for(int start = dp.length-2; start > -1; start--) {
            for(int cur = 1; cur < dp[0].length; cur++) {
                // 扣减当前值 || 不扣减当前值
                rest = cur - nums[start];
                dp[start][cur] = (rest < 0? false : dp[start+1][rest]) || dp[start+1][cur];
            }
        }

        return dp[0][dp[0].length-1];
    }
}
```

#### 494. 目标和 | 01 背包 + 元素组合 | medium

##### 1）暴力递归 | O（n^2）

- **思路**：
  1. 设计一个 f（idx，rest）函数，代表获取 [idx, 结尾] 数字能够组合等于 rest 的不同表达式数量。
  2. f 函数的实现分为 2 种情况：
     1. 如果 idx 已越界，则看 rest 是否为 0 了，是则返回 1，代表这一路的组合是一种满足条件的表达式，如果还没为 0，代表已经没有元素可以凑 0 了，即这一路的组合不满足条件。
     2. 如果 idx 没越界，则当前 idx 的符号尝试使用 + 号，以及尝试使用 - 号，返回结果取两者的和。
- **结论**：时间，526ms，11.74%，空间，39.2mb，10.58%，时间上，由于需要尝试两种符号 + 号 和 - 号，递归深度最大为 n，所以一共需要 n*n 次递归，时间复杂度为 O（n^2），空间上，由于需要执行 n^2 次递归行为，所以额外空间复杂度为 O（n^2）。

```java
class Solution {

    public int findTargetSumWays(int[] nums, int target) {
        if(nums == null || nums.length == 0) {
            return 0;
        }
        return f(nums, 0, target);
    }

    // f代表获取[idx, 结尾]数字能够组合等于rest的不同表达式数量
    private int f(int[] nums, int idx, int rest) {
        if(idx == nums.length) {
            return rest == 0? 1 : 0;
        }
        return f(nums, idx+1, rest-nums[idx]) + f(nums, idx+1, rest+nums[idx]);
    }
}
```

##### 2）记忆化搜索 | <= O（n^2）

- **思路**：在暴力递归的基础上，增加了一张 dp 二维表缓存，如果缓存中存在则从缓存中获取，否则先把结果设置到缓存中再返回。
- **结论**：时间，7ms，52.70%，空间，41.4mb，5.05%，时间上，由于增加了 dp 缓存，相同参数的递归行为只会被处理一次，也就是时间复杂度会比暴力递归的少，即 <= O（n^2），空间上，由于使用了一张长为 n+1, 宽为 m+1 的表，m 为 target + 所有nums[i] 的总和，所以空间复杂度为 O（n * m）。

```java
class Solution {

    private Integer[][] dp;
    private int offset;

    public int findTargetSumWays(int[] nums, int target) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        int min = target, max = target;
        for(int i = 0; i < nums.length; i++) {
            min -= nums[i];
            max += nums[i];
        }

        offset = 0 - min;
        max += offset;
        dp = new Integer[nums.length+1][max+1];

        return f(nums, 0, target + offset);
    }

    // f代表获取[idx, 结尾]数字能够组合等于rest的不同表达式数量
    private int f(int[] nums, int idx, int rest) {
        if(dp[idx][rest] != null) {
            return dp[idx][rest];
        }
        if(idx == nums.length) {
            dp[idx][rest] = rest == offset? 1 : 0;
            return dp[idx][rest];
        }

        dp[idx][rest] = f(nums, idx+1, rest-nums[idx]) + f(nums, idx+1, rest+nums[idx]);
        return dp[idx][rest];
    }
}
```

##### 3）严格表结构优化 | O（n * m）

- **思路**：在记忆化搜索的基础上，经过研究值依赖关系，发现一个值依赖它下方的值，所以这是从下到上，从左到右的初始化模型，因此，首先初始化好最后一行，再从下往上、从左往右初始化 dp 数组。
- **结论**：时间，7ms，52.70%，空间，41.6mb，5.01%，时间上，记 m 为 target + 所有nums[i] 的总和 ，初始化 dp 数组需要遍历 n-1 行，最坏情况下 num[i]=0 时，需要遍历 m 列，因此，最坏的时间复杂度为 O（n * m），空间上，由于使用了一张  n * m 的二维表，所以额外空间复杂度为 O（n * m）。

```java
class Solution {

    public int findTargetSumWays(int[] nums, int target) {
        if(nums == null || nums.length == 0) {
            return 0;
        }

        int min = target, max = target;
        for(int i = 0; i < nums.length; i++) {
            min -= nums[i];
            max += nums[i];
        }

        int offset = 0 - min;
        max += offset;
        int[][] dp = new int[nums.length+1][max+1];

        // 初始化最后一行
        for(int j = 0; j < dp[0].length; j++) {
            dp[dp.length-1][j] = j == offset? 1 : 0;
        }
        // 从下往上、从左往右初始化dp数组
        for(int idx = dp.length-2; idx > -1; idx--) {
            for(int rest = nums[idx]; rest < dp[0].length - nums[idx]; rest++) {
                dp[idx][rest] = dp[idx+1][rest-nums[idx]] + dp[idx+1][rest+nums[idx]];
            }
        }

        return dp[0][target+offset];
    }
}
```

#### 1049. 最后一块石头的重量 II | 01 背包 + 最值 | medium

##### 1）严格表结构优化 | O（2 * n + m * n）

- **思路**：

  1. 记石头的总重量为 sum，ki = -1 的石头重量之和为 neg，表示选 A 的石头的重量，其余 ki = 1 的石头重量之和为 sum - neg，表示选 B 的石头的重量，则有：

     ```java
     i=0
     ∑   ki * stones[i] = (sum − neg) − neg = sum − 2 * neg
     n−1 
     ```

  2. 即如果对每块石头都做了 ki 操作，那么操作过后，剩余总重量为 sum - 2 * neg，如果堆 A 重量 < 总重量 / 2, 堆 B 重量 > 总重量 / 2，假设撞击的是 x 重，那么最终答案就为 sum - 2 * x。

  3. 因此，想要使最后一块石头的重量尽可能小，neg 就需要尽可能大，但最大不能大于 sum / 2，也就相当可以看作是背包容量为 sum / 2，物品和重量都为 stone[i] 的 01 背包问题，然后看能否凑出多少价值出来，走套路模板即可。

- **结论**：时间，2 ms，97.10%，空间，39.1 mb，37.90%，时间上，统计 sum 花费 O（n），求每个 dp 的值花费 O（m * n），遍历比较获取最小的 j 花费 O（n），所以整体时间复杂度为 O（2 * n + m * n），空间上，由于需要一张 m * n 长的 dp 二维表，所以额外空间复杂度为 O（m * n）。 

```java
class Solution {
    public int lastStoneWeightII(int[] stones) {
        // 把石头分为堆A和堆B, 如果堆A重量=总重量/2, 堆B重量=总重量/2, 那么最终结果为0
        // 而如果堆A重量<总重量/2, 堆B重量>总重量/2, 那么就可以假设撞击的是x重, 则最终答案就是sum - 2 * x
        // 相当于01背包问题, 求大小为sum/2的背包, 最多能放入多少重量的石头?
        int sum = 0;
        for(int i = 0; i < stones.length; i++) {
            sum += stones[i];
        }

        // 设计状态: dp[i][j]表示前i块石头能否凑出j的重量来, 0表示没有放入石头, n表示放入了n块石头 
        int target = sum / 2;
        int m = stones.length + 1, n = target + 1;
        boolean[][] dp = new boolean[m][n];

        // 初始化状态: i=0表示没有放入石头, j=0凑出0重量来
        // 第一行, 表示没有石头, 肯定凑不出j重量来 => false
        // 第一列, 表示石头虽然有m个, 但也肯定凑不出0重量来 => false
        dp[0][0] = true;

        // 状态转移: 由于依赖前一行的值, 所以从上到下初始化
        // 1) 如果第i-1个石头比重量j要大, 那么只能放前[0, i-2]的石头
        // 2) 如果第i-1个石头比重量j要小, 那么可以选择只放前[0, i-2]的石头, 也可以放[0, i-1]的, 取任一方案
        // 遍历[0, i-1]一批石头
        for(int i = 1; i < m; i++) {
            for(int j = 0; j < n; j++) {
                // stones[i - 1], 表示这一批当中的最后一个石头
                if(stones[i - 1] > j) {
                    dp[i][j] = dp[i - 1][j];
                }
                // stones[i - 1] <= j
                else {
                    dp[i][j] = dp[i - 1][j] || dp[i - 1][j - stones[i - 1]];
                }
            }
        }
        
        // 求出结果: 求出i=m-1, 即放入前[0, i-1]个石头时, 重量j的最大值x, 以让sum - 2 * x得出最小值
        for(int j = n - 1; j > -1; j--) {
            if(dp[m - 1][j]) {
                return sum - 2 * j;
            }
        }
        
        return 0;
    }
}
```

##### 2）空间压缩优化 | O（2 * n + m * n）

- **思路**：经过研究 1 的表结构依赖，发现是下一层依赖上一层每一列的值，所以可以使用滚动数组，进行空间压缩优化，但要注意的是，由于依赖每一列的值，所以本层滚动更新数组时，后面的值没计算完，不能先更新前面的值，否则就相当于是读到了 dp[i] 的值，而不是 dp[i- 1] 的值了，因此，需要从右往左进行滚动更新。
- **结论**：时间，1 ms，99.97%，空间，38.6 mb，90.78%，时间上，统计 sum 花费 O（n），求每个 dp 的值花费 O（m * n），遍历比较获取最小的 j 花费 O（n），所以整体时间复杂度为 O（2 * n + m * n），空间上，由于需要一张 n 长的 dp 一维表，所以额外空间复杂度为 O（n）。 

```java
class Solution {
    public int lastStoneWeightII(int[] stones) {
        int sum = 0;
        for(int weight : stones) {
            sum += weight;
        }

        // 设计状态: dp[i][j]表示前i块的石头能否凑出j的重量来, 0表示没有放入石头, n表示放入了n块石头 
        int n = sum / 2 + 1;
        boolean[] dp = new boolean[n];
        dp[0] = true;

        // 遍历[0, i-1]一批石头的重量
        for(int weight : stones) {
            for(int j = n - 1; j > -1; j--) {
                if(weight <= j) {
                    dp[j] = dp[j] || dp[j - weight];
                }
            }
        }
        
        // 求出结果: 求出i=m-1, 即放入前[0, i-1]个石头时, 重量j的最大值x, 以让sum - 2 * x得出最小值
        for(int j = n - 1; j > -1; j--) {
            if(dp[j]) {
                return sum - 2 * j;
            }
        }
        
        return 0;
    }
}
```

#### 1723. 完成所有工作的最短时间 | 状压 dp | hard

##### 1）二分查找 + 回溯 + 剪枝 | O（n * logn + n + log[L + R] * k^n）

- **思路**：
  1. 一个思路是，使用 dfs + 回溯的方式，枚举所有的工人尝试工作情况，符合条件的返回 true，不符合条件的回溯掉，继续尝试下一个，如果尝试完毕，还有工作没完成的，则返回 false。
  2. 然后，这种思路尝试枚举的下界为最大工作时间，相当于每人只需要负责一个任务，上界为所有工作的时间和，相当于一个人要负责完所有工作，所以外层就需要尝试 [下界，上界] 次，而由于当能够确定中点 mid 能完成任务时，那么 [mid + 1，上界] 也必定能满足，此时后面的就不用继续尝试了，相当于丢弃了一半，这就可以使用二分查找。
  3. 这里的二分查找，使用的是 l + 1 != r 模板，做法是先确定 l、r 上下界，然后确定 isBlue 区域的划分条件，最后再按题意返回对应的下标，由于是要返回最小的能够完成所有工作的方案，所以这里把 isBlue 划分为右区域，最后返回的是最左的 r 即可。
  4. 其中，dfs + 回溯的思路是，优先让前面的工人尝试大的任务，完成得了的则加入当前的工作时间 wktimes 数组中，再尝试下一个任务，同样也是优先让前面的工作尝试大的任务。
  5. 当工人发现自己完成不了当前任务时，则交给下一个工人来完成，当下一个任务整体都完成不了时，则返回 false，上层接收到递归函数返回的 false，则需要进行回溯，把当前工作时间从 wktimes 抽取出来，再丢给下一个工人尝试。
  6. 但在下一个工人尝试之前，需要先判断一下回溯后的当前工人工作情况，如果没有任何工作，则说明下一个工人肯定也完成不了，则直接退出尝试循环，直接返回 false 即可。
- **结论**：时间，2 ms，92.88%，空间，38.9 mb，63.66%，时间上，从大到小排序划分 O（n * logn + n），二分划分 O（log [L + R]），每次二分都进行 dfs + 回溯，每次 dfs 都需要遍历 k 次，共尝试 n 次深度搜索， 花费 O（k^n），所以时间复杂度为 O（n * logn + n + log[L + R] * k^n），空间上，快速排序花费 O（logn）递归栈，dfs 递归栈花费 n，同时 wktimes 数组花费 k，所以额外空间复杂度为 O（logn + n + k）。

```java
class Solution {
    public int minimumTimeRequired(int[] jobs, int k) {
        // 工作从大到小尝试
        sortIntArray(jobs);
        
        // 下界=最大工作时间, 相当于没人只需要负责一个任务
        // 上界=所有工作的时间和, 相当于一个人要负责完所有工作
        int n = jobs.length, l = jobs[0] - 1, r = Arrays.stream(jobs).sum() + 1, mid;

        // 二分尝试每一个limit, 直到遇到最小的为止
        // mid下界=l下界+r下界=(jobs[0]-1+jobs[0]+1)/2=jobs[0]
	    // mid上界=l上界+r上界=(sum+1-2+sum+1)/2=sum
        while(l + 1 != r) {
            mid = l + (r - l) / 2;
            
            // dfs尝试mid方案成功, 则划分为蓝色区域
            if(dfs(jobs, new int[k], mid, 0)) {
                r = mid;
            } 
            // 方案不成功, 则划分为红色区域
            else {
                l = mid;
            }
        }

        // 返回最小的蓝色区域
        return r;
    }


    private boolean dfs(int[] jobs, int[] wktimes, int limit, int idx) {
        // 如果所有工作都完成了, 则返回true
        if(idx == jobs.length) {
            return true;
        }

        // 如果还有工作没做完, 则工人顺序分配
        int curtime = jobs[idx];
        for(int i = 0; i < wktimes.length; i++) {
            // 当前工人还可以继续分配
            if(wktimes[i] + curtime <= limit) {
                wktimes[i] += curtime;

                // 下一个工作继续走工人顺序分配, 都分配成功, 则说明整个方案成功, 不用继续尝试了
                if(dfs(jobs, wktimes, limit, idx + 1)) {
                    return true;
                }

                // 下一个工作分配失败, 说明当前分配不合理, 则回溯扣减当前工作, 再尝试把工作分配给下一个工人
                wktimes[i] -= curtime;

                // 剪枝, 如果当前工人回溯后的工作时间为0, 由于程序隐藏规则约定, 不能跳过前一个去分配下一个
                // 即代表方案分配地不对, 则下一个也不用尝试了, 直接返回false即可
                if (wktimes[i] == 0) {
                    break;
                }
            }
        }

        // 如果所有工人都尝试完成, 还没能完成所有工作, 则认为没有方案可行
        return false;
    }

    // 从大到小排序int数组
    private void sortIntArray(int[] arr) {
        // 先从小到大排序, 因为int数组不支持comparator比较器, 泛型不对
        Arrays.sort(arr);

        // 再双指针从大到小排序
        int l = 0, r = arr.length - 1;
        while(l < r) {
            swap(arr, l++, r--);
        }
    }

    private void swap(int[] arr, int from, int to) {
        int tmp = arr[to];
        arr[to] = arr[from];
        arr[from] = tmp;
    }
}
```

##### 2）状压 dp | O（3^n）

- **思路**：

  1. 状态压缩，就是使用某种方法，以最小代价来表示某种状态，通常是用一串 01 数字，即二进制数，来表示各个点的状态。如果状态压缩的对象的点状态只有两种，那就可以用 0 和 1 表示，如果有三种，也可以用三进制来表示。
  2. 而状态压缩 dp，简称状压 dp，也就是使用状态压缩的方式，来实现 dp，解决的问题最经典就是，旅行商问题：给定一系列城市和每对城市的距离，求解访问每一座城市一次并回到起始城市的最短回路。其中，就可以用 0 来表示没访问过，1 表示访问过，`dp[i][s]` 则表示走到 i 个点，已经走过的地方为 s 时所走过的最短路。
  3. 本题同样也是采用了状压 dp 的思路，采用二进制来枚举每个工作是否被分配，1 表示工作被分配，0 表示工作没有被分配，从而在枚举 [0, 2^n] 个分配方案时，每一个数字都代表一种工作被分配的状态，然后再分情况讨论处理这些状态即可。
  4. 所以，设 `dp[i][j]` 表示， 给前 [0, i) 人分配工作，分配情况为 j，要完成所有工作所需的最短时长，为了求出最短的时长，那么就需要枚举情况 j 下所有的子集 x，通过第 i 人在当前子集下所有的工作量 sums[x]，和前 [0, i - 1) 个人在 x 对于 j 的补集 `dp[i - 1][x对于j的补集]` 下的最短工作时长中，取最大值，作为某个子集 x 下的最大工作时长，然后对所有子集的最大工作时长求最小值，得到前 [0, i) 人在分配情况为 j 时，的最短最大工作时长，并设置到 `dp[i][j]` ... 周而复始，最后返回 `dp[k - 1][2^n - 1]` 表示前 [0, k) 人在情况 2^n 下的最短最大工作时长。
  5. 其中，sums 数组的计算，是通过枚举 2^n 个方案，然后在每个 i 方案下分别计算的，通过 `int x = Integer.numberOfTrailingZeros(i)` 得到在 i 方案下，从低位开始算起的第一个被分配的工作索引，通过 `y = i - (1 << x)` 得到除去这一次新加入的工作外，上次所有工作量之和在 sums 中的索引，也就是 sums[y] 表示上一次的所有工作量之和，通过 `sums[i] = sums[y] + sums[x]`，即当前 i 方案的所有工作量之和 = 新加入的工作量 + 上一次、缺少本次新加入工作时的所有工作量之和。

- **结论**：

  1. 时间，115 ms，38.47%，空间，40.7 mb，23.06%。

  2. 时间上，初始化 sums 数组花费 O（2^n），初始化 dp[0] 花费 O（2^n），遍历 k 花费 O（k），k 循环里面有 O（n * 2^n）个状态，经过二项式定理：

     ![1655728043405](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1655728043405.png)

     可证得时间复杂度为 O（3^n），证明略，不会...

     ![1655728096683](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1655728096683.png)

  3. 空间上，由于使用了一张 2^n 长的 sums 数组，以及一张 k * 2^n 长的 dp 二维表，所以额外空间复杂度为 O（2^n + k * 2^n）。

```java 
class Solution {
    public int minimumTimeRequired(int[] jobs, int k) {
        int n = jobs.length, bitlen = 1 << n;
        
        // 枚举bitlen种分配方案, 初始化好sums数组, sums[i]表示完成i方案分配下, 所有工作量之和
        int[] sums = new int[bitlen];
        for(int i = 1; i < bitlen; i++) {
            // i：表示方案的枚举位置, 以求出在i方案下, 每个i的所有工作量之和
            // x：numberOfTrailingZeros求的是最后一个1后面出现0的个数, 
            //    -> 表示i方案下最低1出现索引位置, 也就是从右往左第一个分配了工作的位置
            // y：方案i 减去 第一个分配工作的索引位置, 用来得到去掉这次新加入的工作外, 上一次所有工作量之和
            // 因此, sums[i] = sums[y] + jobs[x] 
            // => 本次方案i工作量之和 = 上一次之和 + 新加入的工作时长(二进制时从低位加入, 数组则从0开始数起)
            int x = Integer.numberOfTrailingZeros(i), y = i - (1 << x);
            sums[i] = sums[y] + jobs[x];

            // 第6种情况详解：
            // 110
            //-010
            // 100

            // jobs_idx[0, 1, 2] => jobs[3, 2, 3]
            // 0: 000 => sums[0] = 0
            // 1：001 => x：0, y：1 - 2^0 = 0 = 000 => sums[001] = sums[000] + jobs[000] => sums[1] = sums[0] + jobs[0] = 3
            // 2：010 => x：1, y：2 - 2^1 = 0 = 000 => sums[010] = sums[000] + jobs[001] => sums[2] = sums[0] + jobs[1] = 2
            // 3：011 => x：0, y：3 - 2^0 = 2 = 010 => sums[011] = sums[010] + jobs[000] => sums[3] = sums[2] + jobs[0] = 5
            // 4：100 => x：2, y：4 - 2^2 = 0 = 000 => sums[100] = sums[000] + jobs[010] => sums[4] = sums[0] + jobs[2] = 3
            // 5：101 => x：0, y：5 - 2^0 = 4 = 100 => sums[101] = sums[100] + jobs[000] => sums[5] = sums[4] + jobs[0] = 6
            // 6：110 => x：1, y：6 - 2^1 = 4 = 100 => sums[110] = sums[100] + jobs[001] => sums[6] = sumss[4] + jobs[1] = 5
            // 7：111 => x：0, y：7 - 2^0 = 6 = 110 => sums[111] = sums[110] + jobs[000] => sums[7] = sums[6] + jobs[0] = 8
        }

        // 初始化dp数组, dp[i][j]表示给前i人分配工作, 分配情况为j, 要完成所有工作所需的最短时长
        int[][] dp = new int[k][bitlen];
        for(int j = 0; j < bitlen; j++) {
            // dp[0][i]表示给前[0]人, 分配情况为j, 所需最短时长, 刚好等于sums[j]
            dp[0][j] = sums[j];
        }

        // 状态转移, 从前[1, k]人, 分配情况[0, bitlen]更新dp数组
        int min;
        for(int i = 1; i < k; i++) {
            for(int j = 0; j < bitlen; j++) {
                min = Integer.MAX_VALUE;

                // 从大往小, 枚举每一个方案子集, j & (x - 1)可以枚举到下一个更小的子集
                // 比如j=3=011, 每次循环分别是011,011&010=010,011&001=001, 即一共有[011,010,001]三种子集
                for(int x = j; x != 0; x = j & (x - 1)) {
                    // 再比较每个子集的最大工作时长的最小值
                    min = Math.min(
                        min,
                        // j-x：表示x对于j的补集, 比如j=110,x=010, 那么j-x=110-010=100, 即最高位的1
                        // dp[i-1][j-x]：表示给前i-1人, 分配j-x子集方案的工作, 最小工作时长
                        // sums[x]：表示第i人, 在x子集下, 所有工作量之和
                        // 由于[0, i]下, 如果第i人完成x, 那么前[0, i-1]则完成的是j-x, 所以Math.max表示:
                        // 1) 给前i人分配了x子集方案, 最短工作时长=前i-1人、在j-x的最短工作时长
                        // 2) 第i人、x子集方案下的所有工作量之和
                        // 两者取最大值, 就是当前子集x方案下的最大工作时长
                        Math.max(dp[i - 1][j - x], sums[x])
                    );
                }

                // 在每种子集都遍历完毕后, 得出j方案下所有子集的最大工作时长的最小值
                // 以作为[0, i]、j方案下的所有子集的最长工作时长的最小值
                dp[i][j] = min;
            }
        }

        // 最后, 返回前k-1人, 在2^n-1方案集合下的最小值是多少
        return dp[k - 1][bitlen - 1];
    }
}

```

#### 2304. 第 297 周赛 - 网格中的最小路径代价 | 线性 dp | medium

##### 1）暴力递归 | O（m * n^3）

- **思路**：
  1. 设计一个 f（row，col）函数，代表获取（row, col）点，到最后一行任意一个目标点，的最小路径代价。
  2. 那么主方法要想获取第一行到最后一行的最小路径，只需要遍历第一行的每一个 f（0，col），然后取最小值，即可得到最小路径。
  3. 而 f（row，col）函数的实现，则是同样也是用当前点，取尝试到下一行每一列到最后一行的最小路径，取最小值作为其返回值 = 当前点的值 + 下一行每一列到最后一行的最小路径，即 = 当前点的值 + 到下一行最小点的代价 + （下一行最小点到下下一行的最小路径）... 周而复始，直到最后一行，则可以直接返回最后一行点的值，停止递归。
- **结论**：执行超时，时间上，主方法遍历花费 O（n），f 函数有两维 O（m * n）且函数内又遍历了 n 次列，即花费 O（m * n^2），所以时间复杂度为 O（m * n^3），空间上，递归栈最大深度为 m * n，所以额外空间复杂度为 O（m * n）。

```java
class Solution {
    public int minPathCost(int[][] grid, int[][] moveCost) {
        if(grid == null || grid[0] == null || moveCost == null || moveCost[0] == null) {
            return 0;
        }

        // 尝试到下一行的每一列, 以获取第一行到最后一行的最小路径
        int minCost = Integer.MAX_VALUE;
        for(int i = 0; i < grid[0].length; i++) {
            minCost = Math.min(minCost, f(grid, moveCost, 0, i));
        }

        return minCost;
    }

    // f代表获取(row, col)点, 到最后一行任意一个目标点, 的最小路径代价
    private int f(int[][] grid, int[][] moveCost, int row, int col) {
        // 最后一行到最后一行的代价为0
        if(row == grid.length - 1) {
            return grid[row][col];
        }

        // 如果为非最后一行, 那么尝试到下一行的每一列, 以获取本行到最后一行的最小路径
        int minCost = Integer.MAX_VALUE;
        for(int i = 0; i < grid[0].length; i++) {
            minCost = Math.min(
                minCost, 
                grid[row][col] + moveCost[grid[row][col]][i] + f(grid, moveCost, row + 1, i)
            );
        }

        return minCost;        
    }
}
```

##### 2）记忆化搜索| O（m * n^2）

- **思路**：在暴力递归的基础上，增加了 dp 二维表缓存，如果缓存中存在，则从缓存中获取，否则先计算结果，设置结果到缓存再返回给上一层。
- **结论**：
  1. 时间，10 ms，12.15%，空间，64.7 mb，44.56%。
  2. 时间上，对比暴力递归，由于增加了 dp 缓存，所以下层的再调用下下一层的 f 函数时，上一层的其他列在相同参数下，可能已经调用过了，即存在缓存，此时只需要从缓存中获取即可，所以，dp 缓存表中的元素最多只会被设置一次，相当于减少了一次 n 列的遍历，因此，时间复杂度为 O（m * n^2）。
  3. 空间上，递归栈最大深度为 m * n，dp 二维表花费 O（m * n），所以额外空间复杂度为 O（2 * m * n）。

```java
class Solution {
    public int minPathCost(int[][] grid, int[][] moveCost) {
        if(grid == null || grid[0] == null || moveCost == null || moveCost[0] == null) {
            return 0;
        }

        int[][] dp = new int[grid.length][grid[0].length];
        for(int i = 0; i < grid.length; i++) {
            Arrays.fill(dp[i], -1);
        }

        // 尝试到下一行的每一列, 以获取第一行到最后一行的最小路径
        int minCost = Integer.MAX_VALUE;
        for(int i = 0; i < grid[0].length; i++) {
            minCost = Math.min(minCost, f(grid, moveCost, dp, 0, i));
        }

        return minCost;
    }

    // f代表获取(row, col)点, 到最后一行任意一个目标点, 的最小路径代价
    private int f(int[][] grid, int[][] moveCost, int[][] dp, int row, int col) {
        // 最后一行到最后一行的代价为0
        if(row == grid.length - 1) {
            dp[row][col] = grid[row][col];
            return dp[row][col];
        }
        if(dp[row][col] != -1) {
            return dp[row][col];
        }

        // 如果为非最后一行, 那么尝试到下一行的每一列, 以获取本行到最后一行的最小路径
        int minCost = Integer.MAX_VALUE;
        for(int i = 0; i < grid[0].length; i++) {
            minCost = Math.min(
                minCost, 
                grid[row][col] + moveCost[grid[row][col]][i] + f(grid, moveCost, dp, row + 1, i)
            );
        }

        dp[row][col] = minCost;
        return dp[row][col];        
    }
}
```

##### 3）严格表结构优化 | O（m * n^2）

- **思路**：在记忆化搜索的基础上，经过研究表依赖关系，发现每一行依赖它的下一行的每一列的值，所以需要从下往上进行初始化 dp 数组，最后返回第一行的最小 dp 值即可。
- **结论**：时间，5 ms，96.75%，空间，64.9 mb，30.68%，时间上，对比记忆化搜索，减少了递归栈的调用，但总的时间复杂度还是 O（m * n^2），空间上，由于减少了递归栈的调用，所以减少了 m * n 长递归栈的空间使用，即只使用了一张 m * n 长的 dp 二维表，所以额外空间复杂度为 O（m * n）。

```java
class Solution {
    public int minPathCost(int[][] grid, int[][] moveCost) {
        if(grid == null || grid[0] == null || moveCost == null || moveCost[0] == null) {
            return 0;
        }

        // 初始化最后一行
        int m = grid.length, n = grid[0].length;
        int[][] dp = new int[m][n];
        for(int i = 0; i < n; i++) {
            dp[m - 1][i] = grid[m - 1][i];
        }

        int minCost;
        for(int row = m - 2; row > -1; row--) {
            for(int col = 0; col < n; col++) {
                minCost = Integer.MAX_VALUE;

                // 如果为非最后一行, 那么尝试到下一行的每一列, 以获取本行到最后一行的最小路径
                for(int i = 0; i < grid[0].length; i++) {
                    minCost = Math.min(
                        minCost, 
                        grid[row][col] + moveCost[grid[row][col]][i] + dp[row + 1][i]
                    );
                }

                dp[row][col] = minCost;
            }
        }

        // 尝试到下一行的每一列, 以获取第一行到最后一行的最小路径
        minCost = Integer.MAX_VALUE;
        for(int i = 0; i < grid[0].length; i++) {
            minCost = Math.min(minCost, dp[0][i]);
        }

        return minCost;
    }
}
```

#### 2305. 第 297 周赛 - 公平分发饼干 | 状压 dp | medium

##### 1）二分查找 + 回溯 + 剪枝 | O（n * logn + n + log[L + R] * k^n）

- **思路**：见《1723 . 完成所有工作的最短时间》。
- **结论**：时间，1 ms，89.22%，空间，39.1 mb，31.47%，时间复杂度和额外空间复杂度，证明见《1723 . 完成所有工作的最短时间》。

```java
class Solution {
    public int distributeCookies(int[] cookies, int k) {
        // 从大到下排序cookies数组
        sortIntArray(cookies);

        // 上界=饼干的最大值, 下界饼干总和
        int l = cookies[0] - 1, r = Arrays.stream(cookies).sum() + 1, mid;
        while(l + 1 != r) {
            mid = l + (r - l) / 2;

            // 如果分配成功, 则设置为蓝色区域
            if(dfs(cookies, new int[k], mid, 0)) {
                r = mid;
            } 
            // 如果分配失败, 则设置为红色区域
            else {
                l = mid;
            }
        }

        // 最后返回最小的红色区域下标r
        return r;
    }

    private boolean dfs(int[] cookies, int[] obtains, int limit, int idx) {
        // 如果饼干全部分发完, 则返回true
        if(idx == cookies.length) {
            return true;
        }

        // 如果饼干还没分发完, 则挨个尝试分发
        int cur = cookies[idx];
        for(int i = 0; i < obtains.length; i++) {
            // 如果i孩子还能继续被分饼干, 则继续分给他
            if(obtains[i] + cur <= limit) {
                obtains[i] += cur;

                // 然后继续尝试分下一块饼干, 如果能全部分配成功, 则认为该方案通过
                if(dfs(cookies, obtains, limit, idx + 1)) {
                    return true;
                } 

                // 如果分配没成功, 说明该方案没通过, 则回溯扣减掉之前分的cur, 尝试把cur分给下一个孩子
                obtains[i] -= cur;

                // 如果回溯后发现当前孩子没分到任何饼干, 说明方案错误, 必须是从左到右分配, 不能跳过
                if(obtains[i] == 0) {
                    break;
                }
            }
        }
        
        return false;
    }

    // 从大到下排序int数组
    private void sortIntArray(int[] arr) {
        // 从小到大排序
        Arrays.sort(arr);

        // 再从大到下排序, 因为Arrays.sort不支持int类型
        int l = 0, r = arr.length - 1;
        while(l < r) {
            swap(arr, l++, r--);
        }
    }

    private void swap(int[] arr, int from, int to) {
        int tmp = arr[to];
        arr[to] = arr[from];
        arr[from] = tmp;
    }
}
```

##### 2）状压 dp | O（3^n）

- **思路**：同《1723 . 完成所有工作的最短时间》。
- **结论**：时间，3 ms，70.97%，空间，39 mb，42.30%，时间复杂度和额外空间复杂度，证明见《1723 . 完成所有工作的最短时间》。

```java
class Solution {
    public int distributeCookies(int[] cookies, int k) {
        int n = cookies.length, bitlen = 1 << n;

        // 枚举bitlen种分配方案, 初始化好sums数组, sums[i]表示完成i方案分配下, 所有饼干不公平程度
        int[] sums = new int[bitlen];
        for(int i = 1; i < bitlen; i++) {
            // i：表示方案的枚举位置, 以求出在i方案下, 每个i的所有饼干的不公平程度
            // x：numberOfTrailingZeros求的是最后一个1后面出现0的个数, 
            //    -> 表示i方案下最低1出现索引位置, 也就是从右往左第一个分配了饼干的位置
            // y：方案i 减去 第一个分配饼干的索引位置, 用来得到去掉这次新加入的饼干外, 上一次所有饼干的不公平程度
            // 因此, sums[i] = sums[y] + cookies[x] 
            // => 本次方案i饼干的不公平程度 = 上一次的不公平程度 + 新加入的饼干数量(二进制时从低位加入, 数组则从0开始数起)
            int x = Integer.numberOfTrailingZeros(i), y = i - (1 << x);
            sums[i] = sums[y] + cookies[x];

            // 第6种情况详解：
            // 110
            //-010
            // 100

            // cookies_idx[0, 1, 2] => cookies[3, 2, 3]
            // 0: 000 => sums[0] = 0
            // 1：001 => x：0, y：1 - 2^0 = 0 = 000 => sums[001] = sums[000] + cookies[000] => sums[1] = sums[0] + cookies[0] = 3
            // 2：010 => x：1, y：2 - 2^1 = 0 = 000 => sums[010] = sums[000] + cookies[001] => sums[2] = sums[0] + cookies[1] = 2
            // 3：011 => x：0, y：3 - 2^0 = 2 = 010 => sums[011] = sums[010] + cookies[000] => sums[3] = sums[2] + cookies[0] = 5
            // 4：100 => x：2, y：4 - 2^2 = 0 = 000 => sums[100] = sums[000] + cookies[010] => sums[4] = sums[0] + cookies[2] = 3
            // 5：101 => x：0, y：5 - 2^0 = 4 = 100 => sums[101] = sums[100] + cookies[000] => sums[5] = sums[4] + cookies[0] = 6
            // 6：110 => x：1, y：6 - 2^1 = 4 = 100 => sums[110] = sums[100] + cookies[001] => sums[6] = sumss[4] + cookies[1] = 5
            // 7：111 => x：0, y：7 - 2^0 = 6 = 110 => sums[111] = sums[110] + cookies[000] => sums[7] = sums[6] + cookies[0] = 8
        }

        // 初始化dp数组, dp[i][j]表示给前i人分配饼干, 分配情况为j, 要完成所有分配所需的最小不公平程度
        int[][] dp = new int[k][bitlen];
        for(int j = 0; j < bitlen; j++) {
            dp[0][j] = sums[j];
        }

        // 状态转移, 从前[1, k]人, 分配情况[0, bitlen]更新dp数组
        int min;
        for(int i = 1; i < k; i++) {
            for(int j = 0; j < bitlen; j++) {
                min = Integer.MAX_VALUE;

                // 从大往小, 枚举每一个方案子集, j & (x - 1)可以枚举到下一个更小的子集
                // 比如j=3=011, 每次循环分别是011,011&010=010,011&001=001, 即一共有[011,010,001]三种子集
                for(int x = j; x != 0; x = j & (x - 1)) {
                    // 再比较每个子集的最大不公平程度的最小值
                    min = Math.min(
                        min,
                        Math.max(
                            // j-x：表示x对于j的补集, 比如j=110,x=010, 那么j-x=110-010=100, 即最高位的1
                            // dp[i-1][j-x]：表示给前i-1人, 分配j-x子集方案的饼干, 最小不公平程度
                            // sums[x]：表示第i人, 在x子集下的不公平程度
                            // 由于[0, i]下, 如果第i人完成x, 那么前[0, i-1]则完成的是j-x, 所以Math.max表示:
                            // 1) 给前i人分配了x子集方案, 最小不公平程度=前i-1人、在j-x的最小不公平程度
                            // 2) 第i人、x子集方案下的不公平程度
                            // 两者取最大值, 就是当前子集x方案下的最大不公平程度
                            dp[i - 1][j - x],
                            sums[x]
                        )
                    );
                }

                // 在每种子集都遍历完毕后, 得出j方案下所有子集的最大不公平程度的最小值
                // 以作为[0, i]、j方案下的所有子集的最大不公平程度的最小值
                dp[i][j] = min;
            }
        }

        // 最后, 返回前k-1人, 在2^n-1方案集合下的最小值是多少
        return dp[k - 1][bitlen - 1];
    }
}
```

#### 2310. 第 298 周赛 - 卖木头块 | 线性 dp | hard

线性 dp，线性动态规划，是指在线性结构上进行状态转移，本题在行上和列上，同时枚举切点，属于线性 dp。

##### 1）暴力递归 | O（ [m + n]^[m * n] ）

- **思路**：
  1. 先创建一张二维矩阵，来缓存 prices 提供的不符合二维规则的价格数据。
  2. 然后，由于木块在价格确定、大小确定的情况下，其最大收益也是确定的，所以，设计一个 f（m，n）函数，代表获取 m 高 n 宽木块的最大收益，其中， m、n 需要控制必定在 [ 1, size ] 范围上。
  3. 接着，f 递归函数的实现，则需要分情况讨论：
     - 1）如果 m * n 整块存在题目给的价格，那么先尝试一整块卖。
     - 2）枚举 k  切点 [ 1, m - 1]，尝试横着切，切完之后，留下 k * n 和 (m - k) * n 两半的木块，每次的最大收益就是两半木块的价格之和，而小半木块的价格可以通过递归调用 f 函数求得。
     - 3）枚举 k 切点 [1, n - 1]，尝试竖着切，切完之后，留下 m * k 和 m * (n - k) 两半的木块，每次的最大收益就是两半木块的价格之和，而小半木块的价格可以通过递归调用 f 函数求得。
  4. 最后，题目要求得 m * n 木块的最大价格，则可以直接调用 f（m，n）获得。
- **结论**：执行超时，由于 f 递归函数有两个维度的变量 m 和 n，花费 O（m * n），而每次递归函数过程中，枚举横切点花费 （m），枚举竖切点花费 O（n），所以整体的时间复杂度为 O（ [m + n]^[m * n] ），空间上，由于使用了一张 m * n 的价格缓存表，且递归栈深度为 m * n，所以额外空间复杂度为 O（2 * m * n）。

```java
class Solution {
    public long sellingWood(int m, int n, int[][] prices) {
        // 价格矩阵, 1~m, 0不存储数据
        int[][] matrix = new int[m + 1][n + 1];
        for(int i = 0; i < prices.length; i++) {
            matrix[prices[i][0]][prices[i][1]] = prices[i][2];
        }
        
        return f(matrix, m, n);
    }

    // f代表获取m高n宽的木块的最大收益: m、n必定[1, size]
    private long f(int[][] matrix, int m, int n) {
        // 有价格的话, 尝试整块卖
        long res = matrix[m][n];
        
        // 枚举切点[1, m - 1], 尝试横着切, 切完之后, 留下 k * n 和 (m - k) * n 两半的木块
        for(int k = 1; k < m; k++) {
            res = Math.max(res, f(matrix, k, n) + f(matrix, m - k, n));
        }

        // 枚举切点[1, n - 1], 尝试竖着切, 切完之后, 留下 m * k 和 m * (n - k) 两半的木块
        for(int k = 1; k < n; k++) {
            res = Math.max(res, f(matrix, m, k) + f(matrix, m, n - k));
        }

        return res;
    }
}
```

##### 2）记忆化搜索 | O（ m * n * (m + n) ）

- **思路**：在 1 的基础上，增加一张 m * n 的 dp 缓存，每次调用递归函数时，都先判断缓存中已是否存在结果，如果存在，则直接从缓存中获取，否则先设置到缓存中再返回。
- **结论**：
  1. 时间，144 ms，13.14%，空间，48.9 mb，58.53%。
  2. 时间上，由于 f 递归函数有两个维度的变量 m 和 n，花费 O（m * n），而每次递归函数过程中，枚举横切点花费 （m），枚举竖切点花费 O（n），但重复的 m 和 n 由于存在缓存，所以无需重复执行，因此，整体的时间复杂度为 O（ m * n *  [m + n] ）。
  3. 空间上，由于使用了一张 m * n 的价格缓存表，递归栈深度为 m * n，dp 数组大小为 m * n，所以额外空间复杂度为 O（3 * m * n）。

```java
class Solution {
    public long sellingWood(int m, int n, int[][] prices) {
        // 价格矩阵, 1~m, 0不存储数据
        int[][] matrix = new int[m + 1][n + 1];
        for(int i = 0; i < prices.length; i++) {
            matrix[prices[i][0]][prices[i][1]] = prices[i][2];
        }
        
        long[][] dp = new long[m + 1][n + 1];
        for(int i = 0; i < m + 1; i++) {
            Arrays.fill(dp[i], -1);
        }

        return f(matrix, dp, m, n);
    }

    // f代表获取m高n宽的木块的最大收益: m、n必定[1, size]
    private long f(int[][] matrix, long[][] dp, int m, int n) {
        if(dp[m][n] != -1) {
            return dp[m][n];
        }

        // 有价格的话, 尝试整块卖
        long res = matrix[m][n];
        
        // 枚举切点[1, m - 1], 尝试横着切, 切完之后, 留下 k * n 和 (m - k) * n 两半的木块
        for(int k = 1; k < m; k++) {
            res = Math.max(res, f(matrix, dp, k, n) + f(matrix, dp, m - k, n));
        }

        // 枚举切点[1, n - 1], 尝试竖着切, 切完之后, 留下 m * k 和 m * (n - k) 两半的木块
        for(int k = 1; k < n; k++) {
            res = Math.max(res, f(matrix, dp, m, k) + f(matrix, dp, m, n - k));
        }

        dp[m][n] = res;
        return res;
    }
}
```

##### 3）严格表结构优化 | O（ m * n * (m + n) ）

- **思路**：在 2 的基础上，经过研究表结构依赖发现，一个值同时依赖了 0~m 行 和 0~n 列的值，且初始状态只落在了 matrix 价格数组中，所以无需再对 dp 数组进行额外的初始化，只需要对 m 和 n 遍历一次，分组讨论处理 `dp[i][j]` 的值即可。
- **结论**：时间，46 ms，75.59%，空间，49 mb，45.69%，时间上，遍历了 m 行、n 列、且最里面又遍历了 m + n 次的 k，所以时间复杂度为 O（ m * n * (m + n) ），对比 2 减少了递归函数调用的时间和空间开销，空间上，matrix 价格数组花费 O（m * m），dp 数组花费 O（m * n），所以额外空间复杂度为 O（2 * m * n）。

```java
class Solution {
    public long sellingWood(int m, int n, int[][] prices) {
        // 价格矩阵, 1~m, 0不存储数据
        int[][] matrix = new int[m + 1][n + 1];
        for(int i = 0; i < prices.length; i++) {
            matrix[prices[i][0]][prices[i][1]] = prices[i][2];
        }
        
        // dp代表获取i高j宽的木块的最大收益: i、j必定[1, size]
        long[][] dp = new long[m + 1][n + 1];
        for(int i = 0; i < m + 1; i++) {
            for(int j = 0; j < n + 1; j++) {
                // 有价格的话, 尝试整块卖
                dp[i][j] = matrix[i][j];
                
                // 枚举切点[1, i - 1], 尝试横着切, 切完之后, 留下 k * j 和 (i - k) * j 两半的木块
                for(int k = 1; k < i; k++) {
                    dp[i][j] = Math.max(dp[i][j], dp[k][j] + dp[i - k][j]);
                }

                // 枚举切点[1, j - 1], 尝试竖着切, 切完之后, 留下 i * k 和 i * (j - k) 两半的木块
                for(int k = 1; k < j; k++) {
                    dp[i][j] = Math.max(dp[i][j], dp[i][k] + dp[i][j - k]);
                }
            }
        }

        // 获取m高n宽木块的最大收益
        return dp[m][n];
    }
}
```

##### 4）对称性范围优化 | O（ m * n * [m + n] / 2 ）

- **思路**：
  1. 在 3 的基础上，研究发现，在枚举 k 横着切时，切点 k = 1 和 k = m - 1切出来的效果都是一样，前者切出 1 * n 和 (m - 1）* n 的木块，后者切出（m - 1）* n 和 1 * n 的木块。
  2. 同理，在枚举 k 竖着切时，切点 k = 1 和 k = n - 1切出来的效果也都是一样，前者切出 m * 1 和 m * （n - 1） 的木块，后者切出 m * （n - 1）和 m * 1 的木块。
  3. 因此，切点满足对称性，只需要枚举 [0, k / 2] 即可，减少了一半的 k 遍历过程。
- **结论**：时间，32 ms，86.38%，空间，48.6 mb，82.16%，时间上，遍历了 m 行、n 列、且最里面又遍历了 m / 2 + n / 2  次的 k，所以时间复杂度为 O（ m * n * (m + n) / 2 ），空间上，matrix 价格数组花费 O（m * m），dp 数组花费 O（m * n），所以额外空间复杂度为 O（2 * m * n）。

```java
class Solution {
    public long sellingWood(int m, int n, int[][] prices) {
        // 价格矩阵, 1~m, 0不存储数据
        int[][] matrix = new int[m + 1][n + 1];
        for(int i = 0; i < prices.length; i++) {
            matrix[prices[i][0]][prices[i][1]] = prices[i][2];
        }
        
        // dp代表获取i高j宽的木块的最大收益: i、j必定[1, size]
        long[][] dp = new long[m + 1][n + 1];
        for(int i = 0; i < m + 1; i++) {
            for(int j = 0; j < n + 1; j++) {
                // 有价格的话, 尝试整块卖
                dp[i][j] = matrix[i][j];
                
                // 枚举切点[1, i / 2], 尝试横着切, 切完之后, 留下 k * j 和 (i - k) * j 两半的木块
                for(int k = 1; k < i / 2 + 1; k++) {
                    dp[i][j] = Math.max(dp[i][j], dp[k][j] + dp[i - k][j]);
                }

                // 枚举切点[1, j / 2], 尝试竖着切, 留下 i * k 和 i * (j - k) 两半的木块
                for(int k = 1; k < j / 2 + 1; k++) {
                    dp[i][j] = Math.max(dp[i][j], dp[i][k] + dp[i][j - k]);
                }
            }
        }

        // 获取m高n宽木块的最大收益
        return dp[m][n];
    }
}
```

#### 2318. 第 81 双周赛 - 不同骰子序列的数目 | 线性 dp| hard

##### 1）模拟 + 暴力回溯 | O（6 + 6 * n ^6）

- **思路**：从头开始模拟合法的序列产生过程，即先枚举开头的数字 [1, 6]，然后后面的则走回溯的套路，从互质数组中进行枚举，并通过 preIsValid 函数保证 i 与 i - 2 不同，最后累加回溯过程中得到的结果返回即可。
- **结论**：执行超时，时间上，初始化互质数组花费 O（6），枚举第一个数字花费 O（6），一个回溯过程花费 O（n^6），所以整体的时间复杂度为 O（6 + 6 * n ^6），空间上，由于使用了一张 7 * 7 长度的互质数组，一张 n 长的 tmp 数组，以及回溯栈深度最大为 n，所以额外空间复杂度为 O（49 + 2 * n）。

```java
class Solution {

    // 整理下一个能取得最小公约数为1的数字
    private static final List<int[]> lists = new ArrayList<>(7);
    static {
        lists.add(new int[0]);
        // 1
        lists.add(new int[] { 2, 3, 4, 5, 6 });
        // 2
        lists.add(new int[] { 1, 3, 5});
        // 3
        lists.add(new int[] { 1, 2, 4, 5 });
        // 4
        lists.add(new int[] { 1, 3, 5 });
        // 5
        lists.add(new int[] { 1, 2, 3, 4, 6 });
        // 6
        lists.add(new int[] { 1, 5 });
    }

    public int distinctSequences(int n) {
        // 0位置作废
        List<Integer> tmp = new ArrayList<>(n + 1);
        for(int i = 0; i < n + 1; i++) {
            tmp.add(null);
        }

        int cnt = 0;
        for(int cur = 1; cur <= 6; cur++) {
            tmp.set(1, cur);
            cnt += f(tmp, n, 1, cur);
            tmp.set(1, null);
        }

        return cnt;
    }

    // 开始回溯
    private int f(List<Integer> tmp, int n, int idx, int pre) {
        if(idx == n) {
            return 1;
        }

        // 回溯时, 则枚举下一个能取到的数
        int cnt = 0;
        for(int cur : lists.get(pre)) {
            // 还要符合看i-3的数字是否与自己相同
            if(preIsValid(tmp, idx + 1, cur)) {
                tmp.set(idx + 1, cur);
                cnt += f(tmp, n, idx + 1, cur);
                tmp.set(idx + 1, null);
            }
        }

        return cnt;
    }

    // cur必定大于0
    private boolean preIsValid(List<Integer> tmp, int idx, int cur) {
        if(idx == 1) {
            return true;
        } else if(idx == 2) {
            return tmp.get(idx - 1) != cur;
        } else {
            return tmp.get(idx - 1) != cur && tmp.get(idx - 2) != cur;
        }
    }
}
```

##### 2）线性 dp + 三维互质枚举 | O（78 + 216 * n）

- **思路**：
  1. 定义状态，`dp[n][pre][cur]` 表示 n 长的序列、前一个元素为 pre、当前元素为 cur 时的合法序列个数是多少。其中，这里的思路必须用三维，不然就看不了 i 与 i - 2 以及 i 与 i - 1 是否不同了。并且，dp 数组从 1 开始，0 位置弃用。
  2. 初始化状态，初始化 `dp[2][pre][cur]` 与 dp[1] 互质的状态，由于这里是在互质数组中枚举，所以 pre 和 cur 必不相等。
  3. 状态转移，`dp[n][pre][cur]` = pre != cur && pre != prepre && cur != prepre && 互质，同理，这里也是在互质数组中枚举，所以 pre 与 cur、pre 与 prepe 必不相等且互质，但并不能保证 cur 与 prepre 不同，所以还需要加一个 cur != prepre 不等的条件判断。然后，当前 cur 合法, 则序列个数 = 前 i - 1合法的序列个数的累加和。
  4. 统计结果，最后结果等于 n 长时的各个状态的累加之和。
- **结论**：时间，78 ms，71.29%，空间，50.7 mb，20.30%，时间上，初始化互质数组花费 O（6），初始化 dp 状态花费 O（6 * 6），状态转移花费 O（n * 6 * 6 * 6），统计结果花费 O（6 * 6），所以整体时间复杂度为 O（78 + 216 * n），空间上，由于使用了一张 7 * 7 长度的互质数组，以及一张 (n + 1) * 7 * 7 的三维表，所以额外空间复杂度为 O（49 * [n + 2]）。

```java
class Solution {

    private static final int MOD = (int) 1e9 + 7;

    // 整理下一个能取得最小公约数为1的数字 => 互质数组, 其中Solution虽然多个testCase, 但只会被构建一次
    private static final List<int[]> coprimes = new ArrayList<>(7);
    static {
        coprimes.add(new int[0]);
        // 1
        coprimes.add(new int[] { 2, 3, 4, 5, 6 });
        // 2
        coprimes.add(new int[] { 1, 3, 5 });
        // 3
        coprimes.add(new int[] { 1, 2, 4, 5 });
        // 4
        coprimes.add(new int[] { 1, 3, 5 });
        // 5
        coprimes.add(new int[] { 1, 2, 3, 4, 6 });
        // 6
        coprimes.add(new int[] { 1, 5 });
    }

    public int distinctSequences(int n) {
        if(n == 1) {
            return 6;
        }

        // 定义状态, dp[n][pre][cur]表示n长的序列, 前一个元素为pre, 当前元素为cur时的合法序列个数
        // 必须用三维, 不然看不了i与i-2、i与i-1是否不同, 且从1开始, 0位置弃用
        long[][][] dp = new long[n + 1][7][7];

        // 初始化状态, dp[2][pre][cur]与dp[1]互质
        for(int pre = 1; pre <= 6; pre++) {
            // pre和cur这里必不相等
            for(int cur : coprimes.get(pre)) {
                dp[2][pre][cur] = 1;
            }
        }

        // 状态转移, dp[n][pre][cur] = pre != cur && pre != prepre && cur != prepre && 互质
        for(int len = 3; len <= n; len++) {
            // 枚举i-2的元素
            for(int prepre = 1; prepre <= 6; prepre++) {
                // 枚举互质的i-1的元素, prepre和pre这里必不相等
                for(int pre : coprimes.get(prepre)) {
                    // 枚举互质的i的元素, cur和pre这里必不相等
                    for(int cur : coprimes.get(pre)) {
                        // 还要校验cur和prepre不等
                        if(cur != prepre) {
                            // 当前cur合法, 则序列个数=前i-1合法的序列个数的累加和
                            dp[len][pre][cur] = (dp[len][pre][cur] + dp[len - 1][prepre][pre]) % MOD;
                        }

                    }
                }
            }
        }

        // 统计结果, 最后结果等于n长时的各个值累加之和
        int cnt = 0;
        for(int pre = 1; pre <= 6; pre++) {
            // pre和cur这里必不相等
            for(int cur : coprimes.get(pre)) {
                cnt = (int) ((cnt + dp[n][pre][cur]) % MOD);
            }
        }

        return cnt;
    }
}
```

##### 3）线性 dp + 二维互质枚举 | O（48 + 36 * n）

- **思路**：

  1. 定义状态，`dp[n][cur]` 表示 n 长的序列、当前元素为 cur 时的合法序列个数是多少。其中，这里的思路用的是二维，可以通过 `dp[n - 1][..]` 和 `dp[n - 2][..]` 来判断 i - 1 和 i - 2 的序列个数。并且，dp 数组从 1 开始，0 位置弃用。

  2. 初始化状态，由于需要用到 `dp[n - 2][..]` 的值，所以除了需要初始化 `dp[2][..]` 外，还需要初始化 `dp[1][..]` 的状态，由于都是在互质数组中枚举，所以 pre 和 cur 必不相等。

  3. 状态转移，`dp[n][cur]` = cur != pre && pre != prepre && cur != prepre && 互质，同理，这里也是在互质数组中枚举，所以 pre 与 cur、pre 与 prepe 必不相等且互质。

  4. 虽然这样保证了 `dp[n][cur]` 与 `dp[n - 1][k]` 是互质且不等的，但无法保证 `dp[n][..]` 与 `dp[n - 2][..]`  互质且不等，所以，在转移方程里每次枚举 `dp[n - 1]` 时，都要扣减掉从 `dp[n - 2][cur]` 转移过来的序列个数。

     ![1656506613775](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1656506613775.png)

  5. 但是，这样也还是不对，虽然保证了 `dp[n][cur]`  与 `d[n - 1][..]` 和 `dp[n - 2][..]` 互斥且不等，但却带来了新的问题，如上图，由于 `f[i - 1][1]` 不存在 `dp[i - 3][1]` 转移过来的情况，但  `f[i - 1][1] - f[i - 2][4]` 却多减掉了 `f[i - 3][1]` 转移过来的个数。同理，`f[i - 1][3]` 被多减了 `f[i - 3][3]`、`f[i - 1][5] ` 被多减了 `f[i - 3][5]` 的个数。

  6. 而 `f[i - 2][4]` 就恰恰等于被多扣的个数之和，等于 `f[i - 3][1] + f[i - 3][3] + f[i - 3][5]`，因此，在所有的枚举扣完之后，对于存在 i - 3 元素，即 len > 3 时，还需要加回 `f[i - 2][4]` 的值，保证转移方程的正确。

  7. 最后，统计结果，等于 n 长时的 6 个状态的累加之和，其中，根据同余定理，`(x + mod) % mod = x % mod + mod % mod = x % mod`，通过 ans + mod，可以保证负数转换为正数再取模，且结果不变，保证最终返回的结果为非负数。

- **结论**：时间，26 ms，94.31%，空间，41.6 mb，78.96%，时间上，初始化互质数组花费 O（6），初始化 dp 状态花费 O（6 + 6 * 6），状态转移花费 O（n * 6 * 6），统计结果花费 O（6），所以整体时间复杂度为 O（48 + 36 * n），空间上，由于使用了一张 7 * 7 长度的互质数组，以及一张 (n + 1) * 7 的二维表，所以额外空间复杂度为 O（49 + [n + 1] * 7）。

```java
class Solution {

    private static final int MOD = (int) 1e9 + 7;

    // 整理能让当前互质的上一个数字 => 互质数组, 其中Solution虽然多个testCase, 但只会被构建一次
    // 实质上preCoprimes和coprimes一模一样
    private static final List<int[]> preCoprimes = new ArrayList<>(7);
    static {
        preCoprimes.add(new int[0]);
        // 1
        preCoprimes.add(new int[] { 2, 3, 4, 5, 6 });
        // 2
        preCoprimes.add(new int[] { 1, 3, 5 });
        // 3
        preCoprimes.add(new int[] { 1, 2, 4, 5 });
        // 4
        preCoprimes.add(new int[] { 1, 3, 5 });
        // 5
        preCoprimes.add(new int[] { 1, 2, 3, 4, 6 });
        // 6
        preCoprimes.add(new int[] { 1, 5 });
    }

    public int distinctSequences(int n) {
        if(n == 1) {
            return 6;
        }
        if(n == 2) {
            return 22;
        }

        // 定义状态, dp[n][cur]表示n长的序列, 当前元素为cur时的合法序列个数
        // 这里用了二维, 从1开始, 0位置弃用
        long[][] dp = new long[n + 1][7];

        // 初始化状态, dp[1][cur]全部为1
        for(int cur = 1; cur <= 6; cur++) {
             dp[1][cur] = 1;
        }

        // 初始化状态, dp[2][cur]与dp[1]互质
        for(int cur = 1; cur <= 6; cur++) {
            // pre和cur这里必不相等
            for(int pre : preCoprimes.get(cur)) {
                dp[2][cur] += 1;
            }
        }

        // 状态转移, dp[n][cur] = cur != pre && pre != prepre && cur != prepre && 互质
        for(int len = 3; len <= n; len++) {
            // 枚举i的元素
            for(int cur = 1; cur <= 6; cur++) {
                // 枚举互质的i-1的元素, cur和pre这里必不相等
                for(int pre : preCoprimes.get(cur)) {
                    dp[len][cur] = (dp[len][cur] + dp[len - 1][pre] - dp[len - 2][cur] + MOD) % MOD;
                }

                // 由于每次都扣减dp[n-2][cur], 导致每次都扣多了dp[n-3][pre]的合法数列数
                // 所以所有都扣减完了以后, 还需要加回一次dp[n-3][pre]的累加和, 即刚好等于dp[n-2][cur]
                if(len > 3) {
                    dp[len][cur] = (dp[len][cur] + dp[len - 2][cur] + MOD) % MOD;
                }
            }
        }

        // 统计结果, 最后结果等于n长时的各个值累加之和
        int cnt = 0;
        for(int cur = 1; cur <= 6; cur++) {
            // pre和cur这里必不相等
            // 同余定理, (x + mod) % mod = x % mod + mod % mod = x % mod, 这里+mod可以保证负数转换为正数
            cnt = (int) ((cnt + dp[n][cur] + MOD) % MOD);
        }

        return cnt;
    }
}
```

#### 2321. 第 299 场周赛 - 拼接数组的最大分数 | 线性 dp | hard

##### 1）枚举 + 区间和 | O（4 * n + n^2）

- **思路**：
  1. 先统计好 nums1 的前缀和、后缀和，以及 nums2 的前缀和、后缀和。
  2. 然后，枚举各个子数组，在每次枚举过程中，分别利用前缀和以及后缀和数组，用 O（1）的复杂度，计算出交换之后的 nums1 和 nums2 的元素之和。
  3. 其中，交换区间的前一段元素和 = [0, fi - 1] 的前缀和，交换区间的元素和 = [0, li] 的前缀和 - [0, fi - 1] 的前缀和，交换区间的后一段元素和 = [0, li + 1]的后缀和，所以，交换后的元素和 = 交换区间的前一段元素和 + 交换区间的元素和 + 交换区间的后一段元素和。
  4. 最后，比较返回最大的交换和即可。
- **结论**：执行超时，时间上，计算前缀和以及后缀和花费 O（4 * n），枚举过程花费 O（n^2），所以整体时间复杂度为 O（4 * n + n^2），空间上，由于使用了两张 n 长的前缀和数组，两张 n 长的后缀和数组，所以额外空间复杂度为 O（4 * n）。

```java 
class Solution {
    public int maximumsSplicedArray(int[] nums1, int[] nums2) {
        // nums1.length == nums2.length
        int n = nums1.length;
        
        // 1和2的前缀和
        long[] preSums1 = new long[n];
        preSums1[0] = nums1[0];
        for(int i = 1; i < n; i++) {
            preSums1[i] = preSums1[i - 1] + nums1[i];
        }
        
        long[] preSums2 = new long[n];
        preSums2[0] = nums2[0];
        for(int i = 1; i < n; i++) {
            preSums2[i] = preSums2[i - 1] + nums2[i];
        }
        
        // 1和2的后缀和
        long[] sufSums1 = new long[n];
        sufSums1[n - 1] = nums1[n - 1];
        for(int i = n - 2; i > 0; i--) {
            sufSums1[i] = sufSums1[i + 1] + nums1[i];
        }
        
        long[] sufSums2 = new long[n];
        sufSums2[n - 1] = nums2[n - 1];
        for(int i = n - 2; i > 0; i--) {
            sufSums2[i] = sufSums2[i + 1] + nums2[i];
        }
        
        long max = Long.MIN_VALUE;
        Deque<Integer> deque = new LinkedList<Integer>();
        for(int fi = 0; fi < n; fi++) {
            for(int li = fi; li < n; li++) {
                // 获取sum1交换后的和
                long sum1 = getPreSum(preSums1, fi) + getBtwSum(preSums2, nums2, fi, li) + getSufSum(sufSums1, li);
                // 获取sum2交换后的和
                long sum2 = getPreSum(preSums2, fi) + getBtwSum(preSums1, nums1, fi, li) + getSufSum(sufSums2, li);
                // sum1、sum2取最大值
                max = Math.max(max, Math.max(sum1, sum2));
            }
        }

        return (int) max;
    }

    // 获取[0, fi-1]的前缀和
    private long getPreSum(long[] preSums, int fi) {    
        return fi == 0? 0 : preSums[fi - 1];
    }

    // 获取[fi, li]的和 = [0, li]前缀和 - [0, fi-1]的前缀和
    private long getBtwSum(long[] preSums, int[] sums, int fi, int li) {
        return li == fi? (long) sums[fi] : preSums[li] - getPreSum(preSums, fi);
    }

    // 获取[0, li+1]的后缀和
    private long getSufSum(long[] preSums, int li) {    
        return li == preSums.length - 1? 0 : preSums[li + 1];
    }
}
```

##### 2）最大子序和 + 线性 dp | O（2 * n）

- **思路**：
  1. 在 1 暴力超时后，经过研究状态转移方程发现，转移状态貌似与 nums1 和 nums2 的差有关系，也可通过下面的推导证明出来。
  2. 设 sum1 = nums1 所有元素之和，那么交换 [fi, li] 后，nums1 的整体元素之和 = sum1' - （nums1[fi] + ... + nums1[li]） + （nums2[fi] + ... nums2[li]）= sum1 +（nums2[fi] - nums1[fi]）+ ... + （nums2[li] - nums1[li]），即 sum1' 与 [fi, li] 的所有元素之差的和有关。
  3. 因此，要求最大的 sum1'，就要求出能得到最大元素差之和的 [fi, li] 区间，也就相当于《53. 最大连续子数组和》思路相同，都是线性 dp 的运用。
  4. 由于上面只是得到 sum2 交换到 sum1 的最大和，而最终结果还要看 sum1 交换到 sum2 的最大和，同理也是这么求，最后取两者最大值返回即可。
- **结论**：时间，4 ms，76.43%，空间，52.6 mb，74.27%，时间上，由于遍历两次数组，所以时间复杂度为 O（2 * n），空间上，由于只使用了有限几个变量，所以额外空间复杂度为 O（1）。

```java
class Solution {
    public int maximumsSplicedArray(int[] nums1, int[] nums2) {
        // nums2交换到nums1取nums1的最大和, 与nums1交换到nums2取nums2的最大和, 取最大值
        return Math.max(f(nums1, nums2), f(nums2, nums1));
    }

    private int f(int[] from, int[] to) {
        int pre = 0, maxDiff = Integer.MIN_VALUE, sum = 0;

        for(int i = 0; i < from.length; i++) {
            // 要么与to交换, 与前面的pre构成连续交换子数组
            // 要么不交换, 直接从当前位置开启新的连续交换子数组
            pre = Math.max(pre + to[i] - from[i], 0);

            // 取最大连续交换子数组
            maxDiff = Math.max(maxDiff, pre);

            // 在求最大连续交换子数组过程中, 同时计算所有元素的累加和
            sum += from[i];
        }

        return sum + maxDiff;
    }
}
```

### 3.6. 算法思想 - 互补交换

#### 2306. 第 297 周赛 - 公司命名 | hard

##### 1）前缀分组 + 暴力枚举 | O（ 26 + n + 26 * 26 * m ）

- **思路**：
  1. 先进行预处理，以每个字符串的首字符进行分组，分组的列表存放除了首字符的后缀，得到 firstSuffixs 后缀集合数组。
  2. 然后进行枚举 i、j 字符（看到有限字符的，都可以往枚举的思路上靠），判断 j 字符开头的字符串的后缀集合，是否出现了 i 字符开头的字符串的后缀，如果存在，则代表发生交集。
  3. 发生交集，则说明后缀相同，则 i 组和 j 组都无法交换，因为即使交换了首字母，交换后的字符串重复出现在了 j 组合 i 组，与题目原意相悖，因此这种情况是不可交换的，所以当前枚举的组合数 = 2 * （i 组的集合元素个数  -  交集数）* （j 组的集合元素个数- 交集数）。
  4. 而如果没发生交集，但枚举的字符 i 或者 j 不存在，那么交集自然为 0，且 i 或者 j 组的集合元素个数也为 0，所以结果为 0，不影响最终结果。
  5. 其中，组合数需要 * 2 的原因是，由于枚举 j 总是落后于 i，所以肯定不会出现 [j, i] 的情况，只会出现 [i, j]，但 [j, i] 是符合题意的，所以根据对偶性，需要把结果 * 2 再做累加。
- **结论**：时间，153 ms，77.47%，空间，52.3 mb，50.7%，时间上，初始化 firstSuffixs 数组花费 O（26 + n），枚举 i 、j 花费 O（26 * 26），然后在每个 i、j 枚举过程中判断是否交集，最大花费 O（m），m 是指最大的后缀集合长度，所以整体时间复杂度为 O（26 + n + 26 * 26 * m），空间上，由于使用了一张 26 * m 的数组，所以额外空间复杂度为 O（26 * m）。

```java 
class Solution {
    public long distinctNames(String[] ideas) {
        // 为每个字符初始化一个set集合
        Set<String>[] firstSuffixs = new HashSet[26];
        for(int i = 0; i < 26; i++) {
            firstSuffixs[i] = new HashSet<>();
        }

        // 根据前缀分组, 如果后缀相同的, 则两个组都无法交换, 因为交换首字母会导致重复, 否则可以交换
        // c-[offee], d-[onuts], t-[ime, offee]
        for(String s : ideas) {
            firstSuffixs[s.charAt(0) - 'a'].add(s.substring(1));
        }

        // 枚举每个字符, 累加组合数 = 2 * (A集合大小 - 交集大小) * (B集合大小 - 交集大小)
        // 2 * [(1 * 1) + (0 * 1) + (1 * 2)] = 2 * 3 = 6
        long res = 0;
        for(int i = 0; i < 26; i++) {
            // j < i, 顺序枚举, 可以通过对结果对偶*2, 来减少循环次数, 即只枚举[t, a], 不枚举[a, t]
            for(int j = 0; j < i; j++) {
                int inters = 0; 
                // 判断j中的后缀集合, 是否出现了i的后缀
                for(String suffix : firstSuffixs[i]) {
                    if(firstSuffixs[j].contains(suffix)) {
                        inters++;
                    }
                }
                res += 2 * (firstSuffixs[i].size() - inters) * (firstSuffixs[j].size() - inters);
            }
        }

        return res;
    }
}
```

##### 2）替换首字母 + 互补交换 + 暴力枚举 | O（53 * n）

- **思路**：

  1. 替换首字母的思路是，从数组中找出两个字符串，使得"互换"首字母生成的两个新字符串，在原数组中不存在。而要实现"互换"，暴力解法需要花费 O(n^2)，则会导致超时，所以需要互补交换的思想来降维。
  2. 对于原条件，字符串 s1 和 s2 互换首字母后在原数组不存在，可以拆解为 s1 首字母换成 s2 [0] 后在原数组不存在，且 s2 首字母换成 s1[0] 后在原数组不存在，
  3. 如果把 s2 看作是枚举的字符串，s1 看作是 ideas 数组中的字符串，那么条件 1 需要，对 s1 首字母的行根据枚举的 j 列进行初始化，即下面表格的 [c, d, t] 行，条件 2 则需要，对 s1 首字母的列，即下面表格的 [c, d, t] 列，根据枚举的 i 行进行获取。
  4. 由于第二个 for 循环，是在第一个 for 循环之后发生的，也就是第二次取到的单元格的值，实质上是叠加了一的结果，即【s1 首字母，换成 s2 [ 0 ] 后，在原数组中不存在】&& 【s2 首字母，换成 s1 [ 0 ] 后，在原数组中不存在】同时成立，也就是成功把需要同时考虑 2 个维度的问题，转变为了只需考虑 1 个维度的问题，即得到原始命题的个数，也就是答案，所以累加起来返回即可。

  | i/j  | a    | b    | c    | d    | e    | f    | g    | h    | i    | j    | k    | l    | m    | n    | o    | p    | q    | r    | s    | t    | u    | v    | w    | x    | y    | z    |
  | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
  | a    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | b    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | c    | 1    | 1    | 0    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 0    | 1    | 1    | 1    | 1    | 1    | 1    |
  | d    | 1    | 1    | 1    | 0    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    | 1    |
  | e    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | f    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | g    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | h    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | i    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | j    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | k    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | l    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | m    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | n    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | o    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | p    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | q    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | r    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | s    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | t    | 2    | 2    | 2    | 1    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 2    | 0    | 2    | 2    | 2    | 2    | 2    | 2    |
  | u    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | v    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | w    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | x    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | y    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |
  | z    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    | 0    |

- **结论**：时间，881 ms，27.53%，空间，50.4 mb，80.89%，时间上，缓存 ideas 花费 O（n），第一个 for 循环花费 O（26 * n），第二个 for 循环花费 O（26 * n），所以整体时间复杂度为 O（53 * n），空间上，由于使用了一张 n 长的 set 集合，以及一张 26 * 26 的二维表，所以额外空间复杂度为 O（n + 26 * 26）。

```java
class Solution {
    public long distinctNames(String[] ideas) {
        // 先缓存ideas数组到map中
        Set<String> set = new HashSet<>();
        for(String s : ideas) {
            set.add(s);
        }

        // s2为枚举字符串, s1为ideas数组中的字符串
        // 遍历ideas数组, 枚举26个字符, 如果替换首字母成功, 代表s2[0]交换到s1[0]成功, 则记录[old, j]列
        int[][] cnt = new int[26][26];
        for(String s : ideas) {
            char[] chars = s.toCharArray();
            int oldc_idx = chars[0] - 'a';
            for(int j = 0; j < 26; j++) {
                chars[0] = (char) (j + 'a');
                if(!set.contains(String.valueOf(chars))) {
                    cnt[oldc_idx][j]++;
                }
            }
        }

        // 遍历ideas数组, 枚举26个字符, 如果替换首字母成功, 代表s1[0]交换到s2[0]成功, 则累加[i, old]列 
        long res = 0;
        for(String s : ideas) {
            char[] chars = s.toCharArray();
            int oldc_idx = chars[0] - 'a';
            for(int i = 0; i < 26; i++) {
                chars[0] = (char) (i + 'a');
                if(!set.contains(String.valueOf(chars))) {
                    res += cnt[i][oldc_idx];
                }
            }
        }

        return res;
    }
}
```

