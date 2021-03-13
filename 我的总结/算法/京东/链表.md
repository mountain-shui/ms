 ### 链表反转

1、以p2节点为根，把p2节点原本指向p3的next指针反转，指向p1

![v2-5e778b73052bfe4f258cab5736b49401_720w](images\v2-5e778b73052bfe4f258cab5736b49401_720w.jpg)

2、三个临时节点引用p1，p2，p3分别向后移动一位

![v2-651cebb92ef638ef7d3d58be92adee01_720w](images\v2-651cebb92ef638ef7d3d58be92adee01_720w.jpg)

3、重复”1”的工作，以p2节点为根，把p2节点原本指向p3的next指针反转，指向p1

![v2-90df2382bc929526c84470b42eda672f_720w](images\v2-90df2382bc929526c84470b42eda672f_720w.jpg)

4、重复”2”的工作，三个临时节点引用p1，p2，p3分别向后移动一位

![v2-9eb0ead14474e2bf73ea15498fc9a3e3_720w](images\v2-9eb0ead14474e2bf73ea15498fc9a3e3_720w.jpg)

5、继续重复以上的工作，一直到p2为空为止

![v2-33979a48ca94d9eacd0ceeebcb6ca20c_720w](images\v2-33979a48ca94d9eacd0ceeebcb6ca20c_720w.jpg)

6、最后，把head节点的next指向空，成为反转链表的尾节点。并把p1赋值给head，让p1所在节点成为反转链表的头节点

![v2-3babc6ad6247cc74c1bc6a17bfc2a7f9_720w](images\v2-3babc6ad6247cc74c1bc6a17bfc2a7f9_720w.jpg)





###  链表判断环路 

### 1. 遍历链表

将已经遍历过的节点放在一个hash表中，如果一个节点已经存在hash表中，说明有环。
 时间复杂度:O(n)
 空间复杂度:O(n)

### 2. 快慢指针

时间复杂度:O(n)
 空间复杂度:O(1)
 使用两个指针

#### 判断环的存在：

设置两个指针fast和slow，初始值都指向头，slow每次前进1步，fast每次前进2步。
 链表存在环：则fast必定先进入环，而slow后进入环，两个指针必定相遇。(当然，fast先行头到尾部为NULL，则是无环链表)。
 链表不存在环：fast先行头到尾部为NULL，则是无环链表。

#### 寻找环的入口点：

![20180524215151196](images\20180524215151196.jpg)

根据前文的“公理”可得：a+nr+b=2(a+b),  其中r为圆周长，n>=1

化简得：a=nr-b, 即: a=(n-1)r+r-b

这个式子的意义就是，一个慢指针slower1从链表头出发，1个慢指针slower2从b点出发，slower1走到环入口时(路程为a)，slower也刚好走到环入口（路程为(n-1)r+r-b：n-1个整圈加上r-b的路程)



