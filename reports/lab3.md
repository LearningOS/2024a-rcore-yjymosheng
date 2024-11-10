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
