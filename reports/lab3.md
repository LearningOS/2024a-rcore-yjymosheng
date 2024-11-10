# 我实现的功能

我实现了spawn和stride的调度算法,我采取了以下的方式

1. 因为tips提到spawn不必像 fork 一样复制父进程的地址空间。我采取了如同initproc的创建方式，调用了，通过app的名字来创建地址空间。

2. 调度部分在TCB添加了一个字段作为stride容纳后创建任务时初始化pass。通过lazy创建了一个队列作为调度队列
 

# 简答作业
## 问题1

#### stride 算法原理非常简单，但是有一个比较大的问题。例如两个 pass = 10 的进程，使用 8bit 无符号整形储存 stride， p1.stride = 255, p2.stride = 250，在 p2 执行一个时间片后，理论上下一次应该 p1 执行。


1. 实际情况是轮到 p1 执行吗？为什么？

        不是，实际情况是p2继续执行。

        原因： stride由u8进行存储，250+10=260但是u8::max是255,溢出之后变为了4u8,4 < 255，所以由p2进行执行

2. 我们之前要求进程优先级 >= 2 其实就是为了解决这个问题。可以证明， 在不考虑溢出的情况下 , 在进程优先级全部 >= 2 的情况下，如果严格按照算法执行，那么 STRIDE_MAX – STRIDE_MIN <= BigStride / 2 . 为什么？尝试简单说明（不要求严格证明）。

     首先,当priority>=2时，pass 一定小于等于BigStride/2 。那么当严格按照算法执行时，且每个进程的初始stride都为0，则STRIDE_MAX - STRIDE_MIN = pass ,且pass <=  BigStride / 2。 所以STRIDE_MAX – STRIDE_MIN <= BigStride / 2 

3. 已知以上结论，考虑溢出的情况下，可以为 Stride 设计特别的比较器，让 BinaryHeap<Stride> 的 pop 方法能返回真正最小的 Stride。补全下列代码中的 partial_cmp 函数，假设两个 Stride 永远不会相等。


```rust
use core::cmp::Ordering;

struct Stride(u64);

impl PartialOrd for Stride {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
        let BigStride = 255;
        
        let diff = (self.0.wrapping_sub(other.0)) as i64;
        
        if diff.abs() as u64 <= BigStride / 2 {
            if diff < 0 {
                Some(Ordering::Less)
            } else {
                Some(Ordering::Greater)
            }
        } else {
            if self.0 < other.0 {
                Some(Ordering::Greater)
            } else {
                Some(Ordering::Less)
            }
        }
    }
}

impl PartialEq for Stride {
    fn eq(&self, other: &Self) -> bool {
        false
    }
}
```

# 荣誉准则

1. 在完成本次实验的过程（含此前学习的过程）中，我曾分别与 以下各位 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：

        群友，关于问答题的证明题

1. 此外，我也参考了 以下资料 ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：

        RISCV汇编指令

3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。