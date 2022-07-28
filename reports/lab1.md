# 多道程序与分时多任务

## 实验报告

### TASK

主要修改如何管理 `TaskInfo`，包括，存储，初始化，修改，获取。

#### `task.rs`

为 `TaskControlBlock` 增加：
- `syscall_times: [u32; MAX_SYSCALL_NUM]`：系统调用次数。
- `start_time: usize`：第一次被调度时刻。

#### `mod.rs`

Initialize：

在 `lazy_static` 中对新增字段进行初始化。

update：

在 `run_first_task` 以及 `run_next_task` 中，判断为第一次调用，更新 `start_time` 为当前时间。

为 `TaskManage` 新增函数 `make_a_syscall(&self, syscall_id: usize)` 为特定的调用类型增加调用次数。

get：

`get_task_status`、`get_task_calltimes`、`get_task_rtime` 分别获得当前程序的状态、调用次数以及运行时间。

调用次数为了**防止所有权转移**，采用了 `clone`。

为 `updata`、`get` 函数创建的外部接口，方便调用。

## SYSCALL

### `mod.rs`

在进入具体调用之前，令 `TaskManage` 为系统调用数增加1。

### `process.rs`

通过 `get` 函数进行更新。

## `timer.rs`

增加获取当前 `ms` 的函数，途中遇到的问题，不能直接采用定义的方式求 `ms`，因为都为整数运算，会产生精读损失，为了与程序中的计算时间保持一致，在 `get_time_us` 上进行再处理。

## 简答题

1. 
2. 深入理解 `trap.S` 中两个函数 `__alltraps` 和 `__restore` 的作用，并回答如下问题:
    - 指向内核栈。（1）第一次进入用户态；（2）当发生异常时，从内核态返回用户态，恢复上下文。
    - `sstatus`：还原 `Trap` 发生之前的转态； `sepc`：`Trap` 处理完成后默认会执行的下一条指令的地址；`sscratch`：用户栈，用于之后与内核栈进行交换。
    - `x2`：栈指针，现在还没有切换，先预存到 `sscratch` 中了，在 60 行再交换进 `sp`；`x4`：除非我们手动出于一些特殊用途使用它，否则一般也不会被用到，所以可以跳过。
    - `sp`：指向当前用户栈；`sscratch`：指向内核栈。
    - `sret`：从内核态返回用户态。
    - `sp`：指向内核栈；`sscratch`：指用户栈。
    - `call trap_handler` 陷入内核。
