## Q1
- Prediction / 预测:
```
Time  PID: 0     PID: 1     CPU  IOs
1   RUN:cpu    READY      1
2   RUN:cpu    READY      1
3   RUN:cpu    READY      1
4   RUN:cpu    READY      1
5   RUN:cpu    READY      1
6   DONE       RUN:cpu    1
7   DONE       RUN:cpu    1
8   DONE       RUN:cpu    1
9   DONE       RUN:cpu    1
10   DONE       RUN:cpu    1
```

总时间：10
CPU 利用率：100%
- Reasoning / 理由:
两个进程都只有 CPU 指令，没有 I/O。系统只在进程结束或发起 I/O 时切换。
因此 PID 0 先连续跑完 5 条指令（tick 1-5），PID 1 再连续跑完 5 条（tick 6-10）。
CPU 全程都在工作，利用率 100%。

- Verified result / 验证结果:
Total Time 10, CPU 100%, IO 0%.


- Analysis / 分析:
预测和实际完全一致。两个进程都只有 CPU 指令，系统在进程结束后切换，<br/>所以 PID 0 先跑完 5 tick，PID 1 再跑完 5 tick。CPU 全程忙碌，利用率 100%。

## Q2
- Prediction / 预测:
```
Time  PID: 0     PID: 1     CPU  IOs
1   RUN:cpu    READY      1
2   RUN:cpu    READY      1
3   RUN:cpu    READY      1
4   RUN:cpu    READY      1
5   DONE       RUN:io     1 
6   DONE       BLOCKED          1
7   DONE       BLOCKED          1
8   DONE       BLOCKED          1
9   DONE       BLOCKED          1
10   DONE       BLOCKED          1
11   DONE       RUN:io_done 1
```

总时间：11
CPU 利用率：54.55%
I/O 利用率：54.55%


- Reasoning / 理由:
PID 0 先连续跑完 4 条 CPU 指令（tick 1-4）。
PID 0 结束后切换到 PID 1，PID 1 发起 I/O（tick 5 RUN:io 占用 1 CPU tick），
然后 BLOCKED 5 tick（tick 6-10）等待 I/O 完成，
最后用 1 tick（tick 11）处理 I/O 完成。

- Verified result / 验证结果:
Total Time 11, CPU 54.55%, IO 45.45%.


- Analysis / 分析:
预测的状态表完全正确。IO 利用率我之前预测 54.55%，实际 45.45%。
差异原因：模拟器只统计 BLOCKED 的 5 tick 为 I/O Busy，
不包括 RUN:io 和 RUN:io_done（它们算 CPU Busy）。
所以 IO Busy = 5/11 = 45.45%。


## Q3
- Prediction / 预测:
```
Time  PID: 0     PID: 1     CPU  IOs
1   RUN:io    READY      1    1
2   BLOCKED   RUN:cpu    1    1
3   BLOCKED   RUN:cpu    1    1
4   BLOCKED   RUN:cpu    1    1
5   BLOCKED   RUN:cpu    1    1
6   BLOCKED   DONE           1
7*  RUN:io_done DONE      1

```

总时间：7
CPU 利用率：85.71%
I/O 利用率：85.71%


- Reasoning / 理由:
PID 0 先运行，发起 I/O（tick 1 RUN:io）。之后 PID 0 被切换走，进入 BLOCKED 状态。
系统把 CPU 交给 PID 1，PID 1 在 tick 2-5 连续跑完 4 条 CPU 指令。
tick 6 时 PID 1 已经 DONE，而 PID 0 还在等待 I/O 的最后 1 tick，所以 CPU 空闲。
tick 7（带 *）PID 0 用 1 tick 处理 I/O 完成，然后结束。


- Verified result / 验证结果:
otal Time 7, CPU 85.71%, IO 71.43%.


- Analysis / 分析:
预测的状态表完全正确。IO 利用率我之前预测 85.71%，实际 71.43%。
差异原因同上：I/O Busy 只算 BLOCKED 期间的 5 tick，5/7 = 71.43%。

## Q4
- Prediction / 预测:

```

Time  PID: 0     PID: 1     CPU  IOs
1   RUN:io    READY      1    1
2   BLOCKED   READY          1
3   BLOCKED   READY          1
4   BLOCKED   READY          1
5   BLOCKED   READY          1
6   BLOCKED   READY          1
7*  RUN:io_done READY     1
8   DONE      RUN:cpu    1
9   DONE      RUN:cpu    1
10   DONE      RUN:cpu    1
11   DONE      RUN:cpu    1

```

总时间：11
CPU 利用率：54.55%
I/O 利用率：54.55%


- Reasoning / 理由:
使用 SWITCH_ON_END 后，系统只在当前进程完全结束时才切换。
PID 0 在 tick 1 发起 I/O 后进入 BLOCKED，但系统不切走
所以 tick 2-6 CPU 完全空闲，PID 1 只能 READY 等待。
tick 7 PID 0 处理完 I/O 完成，tick 8 才正式 DONE。
直到这时系统才切换给 PID 1，PID 1 在 tick 8-11 跑完 4 条 CPU 指令。
总时间比 Q3 长了 4 tick，CPU 利用率也降低。



- Verified result / 验证结果:
Total Time 11, CPU 54.55%, IO 45.45%.


- Analysis / 分析:
预测和实际一致。SWITCH_ON_END 让系统等 PID 0 完全结束才切换，
CPU 在 PID 0 阻塞期间（tick 2-6）空闲。IO Busy 同样只算 BLOCKED 的 5 tick。

## Q5
- Prediction / 预测:

```

Time  PID: 0     PID: 1     CPU  IOs
1   RUN:io    READY      1    1
2   BLOCKED   RUN:cpu    1    1
3   BLOCKED   RUN:cpu    1    1
4   BLOCKED   RUN:cpu    1    1
5   BLOCKED   RUN:cpu    1    1
6   BLOCKED   DONE           1
7*  RUN:io_done DONE      1

```

总时间：7
CPU 利用率：85.71%
I/O 利用率：85.71%

- Reasoning / 理由:
SWITCH_ON_IO 是模拟器的默认行为（所以 Q3 和 Q5 结果相同）。
当 PID 0 在 tick 1 发起 I/O 后，系统立即切换给 PID 1。
PID 1 在 tick 2-5 用 CPU 跑完 4 条指令，tick 6 已 DONE，此时 CPU 短暂空闲。
tick 7（带 *）PID 0 完成 I/O 处理，进程结束。
与 Q4 相比，总时间从 11 缩短到 7，CPU 利用率从 54.55% 提升到 85.71%，
说明 SWITCH_ON_IO 能更有效地利用 CPU。


- Verified result / 验证结果:
Total Time 7, CPU 85.71%, IO 71.43%.


- Analysis / 分析:
预测和实际一致。SWITCH_ON_IO 是默认行为（Q3 和 Q5 结果相同），
CPU 在 PID 0 等待时切换给 PID 1。IO Busy 5/7 = 71.43%。


## Q6
- Prediction / 预测:

```

Time  PID: 0      PID: 1     PID: 2     PID: 3     CPU  IOs
1   RUN:io     READY      READY      READY      1    1
2   BLOCKED    RUN:cpu    READY      READY      1    1
3   BLOCKED    RUN:cpu    READY      READY      1    1
4   BLOCKED    RUN:cpu    READY      READY      1    1
5   BLOCKED    RUN:cpu    READY      READY      1    1
6   BLOCKED    RUN:cpu    READY      READY      1    1
7*  BLOCKED    DONE       RUN:cpu    READY      1    1
8   BLOCKED    DONE       RUN:cpu    READY      1    1
9   BLOCKED    DONE       RUN:cpu    READY      1    1
10   BLOCKED    DONE       RUN:cpu    READY      1    1
11   BLOCKED    DONE       RUN:cpu    READY      1    1
12   BLOCKED    DONE       DONE       RUN:cpu    1
13   BLOCKED    DONE       DONE       RUN:cpu    1
14   BLOCKED    DONE       DONE       RUN:cpu    1
15   BLOCKED    DONE       DONE       RUN:cpu    1
16   BLOCKED    DONE       DONE       RUN:cpu    1
17   RUN:io_done DONE      DONE       DONE       1
18   RUN:io     DONE       DONE       DONE       1    1
19   BLOCKED    DONE       DONE       DONE            1
20   BLOCKED    DONE       DONE       DONE            1
21   BLOCKED    DONE       DONE       DONE            1
22   BLOCKED    DONE       DONE       DONE            1
23   BLOCKED    DONE       DONE       DONE            1
24*  RUN:io_done DONE     DONE       DONE       1
25   RUN:io     DONE       DONE       DONE       1    1
26   BLOCKED    DONE       DONE       DONE            1
27   BLOCKED    DONE       DONE       DONE            1
28   BLOCKED    DONE       DONE       DONE            1
29   BLOCKED    DONE       DONE       DONE            1
30   BLOCKED    DONE       DONE       DONE            1
31*  RUN:io_done DONE     DONE       DONE       1

```

总时间：31
CPU 利用率 = 17/31 ≈ 54.84%
I/O 利用率 = 21/31 ≈ 67.74%

- Reasoning / 理由:
PID 0 有 3 组 I/O，PID 1、2、3 各有 5 条 CPU 指令。
使用 SWITCH_ON_IO 和 IO_RUN_LATER：

第一组 I/O（tick 1-17）：
- tick 1：PID 0 发起 I/O。
- tick 2-6：PID 0 BLOCKED，PID 1 跑 5 条 CPU。
- tick 7：I/O 完成（带 *），但 IO_RUN_LATER 让 PID 0 排队。系统切给 PID 2。
-  tick 7-11：PID 2 跑 CPU。
- tick 12-16：PID 3 跑 CPU。
- tick 17：其他进程都 DONE，PID 0 才恢复，执行 RUN:io_done。

第二组 I/O（tick 18-24）：
- 只剩 PID 0 一个进程，tick 18 发起 RUN:io，tick 19-23 BLOCKED，
tick 24 I/O 完成并执行 RUN:io_done。此时无其他进程竞争 CPU，
IO_RUN_LATER 无影响。

第三组 I/O（tick 25-31）：
同第二组，tick 25 RUN:io，tick 26-30 BLOCKED，tick 31 RUN:io_done 结束。


总时间 31。CPU 忙碌 17 tick，利用率 54.84%
I/O 设备在 tick 1-7、18-24、25-31 工作，利用率 67.74%。
- Verified result / 验证结果:
Total Time 31, CPU 67.74%, IO 48.39%.


- Analysis / 分析:
总时间 31 和预测一致，但 CPU Busy 实际为 21 tick（67.74%），
比我预测的 17 tick 多了 4 tick。差异原因：tick 7 的 `*` 行实际上 CPU 仍在工作
处理 PID 0 的 io_done 或其他进程），我在手动推理时漏算了。
IO Busy 15 tick（48.39%）也比我预测的 21 tick 少，
因为 I/O Busy 只算 BLOCKED 的 5 tick，不包括 RUN:io 和 RUN:io_done。


## Q7
- Prediction / 预测:

```

Time  PID: 0      PID: 1     PID: 2     PID: 3     CPU  IOs
1   RUN:io     READY      READY      READY      1    1
2   BLOCKED    RUN:cpu    READY      READY      1    1
3   BLOCKED    RUN:cpu    READY      READY      1    1
4   BLOCKED    RUN:cpu    READY      READY      1    1
5   BLOCKED    RUN:cpu    READY      READY      1    1
6   BLOCKED    RUN:cpu    READY      READY      1    1
7*  BLOCKED    DONE       READY      READY           1
8   RUN:io_done DONE      READY      READY      1
9   RUN:io     DONE       READY      READY      1    1
10   BLOCKED    DONE       RUN:cpu    READY      1    1
11   BLOCKED    DONE       RUN:cpu    READY      1    1
12   BLOCKED    DONE       RUN:cpu    READY      1    1
13   BLOCKED    DONE       RUN:cpu    READY      1    1
14   BLOCKED    DONE       RUN:cpu    READY      1    1
15*  BLOCKED    DONE       DONE       READY           1
16   RUN:io_done DONE      DONE       READY      1
17   RUN:io     DONE       DONE       READY      1    1
18   BLOCKED    DONE       DONE       RUN:cpu    1    1
19   BLOCKED    DONE       DONE       RUN:cpu    1    1
20   BLOCKED    DONE       DONE       RUN:cpu    1    1
21   BLOCKED    DONE       DONE       RUN:cpu    1    1
22   BLOCKED    DONE       DONE       RUN:cpu    1    1
23*  BLOCKED    DONE       DONE       DONE            1
24   RUN:io_done DONE      DONE       DONE       1

```

总时间：24
CPU 利用率：87.5%
I/O 利用率：87.5%


- Reasoning / 理由:
IO_RUN_IMMEDIATE 让 I/O 完成的进程立即恢复运行，而不是排队。
PID 0 的 3 组 I/O 因此几乎无缝衔接（tick 1-8、9-16、17-24），
每组只给其他 CPU 进程留出 5 tick 的空隙。
PID 1 在 tick 2-6 跑，PID 2 在 tick 10-14 跑，PID 3 在 tick 18-22 跑。
总时间从 Q6 的 31 tick 缩短到 24 tick，
CPU 和 I/O 利用率都提升到 87.5%。


- Verified result / 验证结果:
Total Time 21, CPU 100%, IO 71.43%.


- Analysis / 分析:
IO_RUN_IMMEDIATE 让 PID 0 的 I/O 完成后立即恢复，3 组 I/O 几乎无缝衔接。
CPU 在 PID 0 阻塞等待 I/O 时切给 PID 1、2、3，因此 CPU 利用率达到 100%。
IO Busy 15/21 = 71.43%（只算 BLOCKED 的 tick，共 15 tick）。
我预测的总时间是 24，实际是 21，差了 3 tick。
原因：IO_RUN_IMMEDIATE 下 PID 0 恢复得非常快，
PID 1、2、3 分别在 PID 0 的 3 次 BLOCKED 期间跑完 5 tick CPU，
完全"填满"了等待时间，因此没有额外的 CPU 空闲。


## Q8


- Prediction / 预测:

使用种子 1、2、3 各运行 3 种设置，结果如下。

**种子 1（-s 1）**：

- 默认设置：Total Time 15, CPU 53.33%, IO 66.67%
- IO_RUN_IMMEDIATE：Total Time X, CPU X%, IO X%
- SWITCH_ON_END：Total Time X, CPU X%, IO X%

**种子 2（-s 2）**：

（同上格式）


**种子 3（-s 3）**：
（同上格式）


- Reasoning / 理由:
种子决定每条指令是 CPU 还是 I/O。
- 默认设置（SWITCH_ON_IO + IO_RUN_LATER）：I/O 完成后进程排队，CPU 在 I/O 阻塞时切换给其他进程。
- IO_RUN_IMMEDIATE：I/O 完成后进程立即恢复，减少等待时间，通常总时间更短。
- SWITCH_ON_END：只有当前进程完全结束才切换，CPU 空闲时间变长，总时间最长。


- Verified result / 验证结果:
种子 1 默认：Total Time 15, CPU 53.33%, IO 66.67%
种子 1 IMMEDIATE：Total Time 15, CPU 53.33%, IO 66.67%
种子 1 SWITCH_ON_END：Total Time 18, CPU 50.00%, IO 55.56%
种子 2 默认：Total Time 16, CPU 62.50%, IO 87.50%
种子 2 IMMEDIATE：Total Time 16, CPU 62.50%, IO 87.50%
种子 2 SWITCH_ON_END：Total Time 30, CPU 33.33%, IO 66.67%
种子 3 默认：Total Time 18, CPU 50.00%, IO 61.11%
种子 3 IMMEDIATE：Total Time 17, CPU 52.94%, IO 64.71%
种子 3 SWITCH_ON_END：Total Time 24, CPU 37.50%, IO 62.50%


- Analysis / 分析:
种子决定每条指令是 CPU 还是 I/O，因此每个种子的指令序列都不同。
三种调度策略的对比：
- 默认设置（SWITCH_ON_IO + IO_RUN_LATER）：I/O 完成后进程排队等
其他进程跑完，I/O 设备可能空闲，总时间中等。
- IO_RUN_IMMEDIATE：I/O 完成后立即恢复，减少等待时间，总时间通常最短。
种子 3 中默认 18 → IMMEDIATE 17（快 1 tick）。
- SWITCH_ON_END：CPU 在 I/O 等待期间空闲，总时间最长。
种子 2 中默认 16 → SWITCH_ON_END 30（慢了近一倍！）。
整体来看，SWITCH_ON_END 的效果最差，IO_RUN_IMMEDIATE 的效果最好。

